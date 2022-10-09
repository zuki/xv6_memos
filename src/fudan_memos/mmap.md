# xv6-lazy-complete-mmap

## `void *mmap(void *addr, int length, int prot, int flags, int fd, int offset)`

- flgasはMAP_ANONYMOUSとMAP_FILEだけ
- vmaはリンクリストで持つ
- 実ページの割当て、マッピングはなし（page-faultで処理）

1. p->sz = p->sz + length
2. mmap_regionを作成してメンバをセット
3. MAP_ANONYMOUS: fd != -1 => エラー
4. MAP_FILE:
   - fd > -1: fdを作成してp->ofile[fd])を複製
   - fd < 0: エラー
5. 作成したmmap_regionをproc->headに組み込む
6. proc->nregions++


```
# mmaptest
mmap_test starting
- open mmap.dur (3) RDONLY
[1] test mmap f
- mmap 3 -> 2PAGE PROT_READ MAP_PRIVATE: addr=0x600000000000
- munmmap 2PAGE: addr=0x600000000000
[1] OK
[2] test mmap private
- mmap 3 -> 2PAGE PROT_READ/WRITE MAP_PRIVATE: addr=0x600000000000
- close 3
- write 2PAGE = Z
- p[1]=Z
- munmmap 2PAGE: addr=0x600000000000
[2] OK
[3] test mmap read-only
- open mmap.dur (3) RDONALY
- mmap 3 -> 3PAGE PROT_READ/WRITE MAP_SHARED: addr=0xffffffffffffffff
- close 3
[3] OK
[4] test mmap read/write
- open mmap.dur (3) RDWR
- mmap 3 -> 3PAGE PROT_READ|WRITE MAP_SHARED: addr=0x600000000000
- close 3
- print 2PAGE = Z
- munmmap 2PAGE: addr=0x600000000000
[4] OK
[5] test mmap dirty
- open mmap.dur (3) RDWR
- close 3
[5] OK
[6] test not-mapped unmap
- munmmap PAGE: addr=0x600000002000
[6] OK
[7] test mmap two files
- open mmap1 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000000000
- close 3
- unlink mmap1
- open mmap2 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000001000
- close 3
- unlink mmap2
mmaptest: mmap_test failed: mmap1 mismatch, pid=7
[7] failed
mmap_test: Total: 7, OK: 6, NG: 1

fork_test starting
wait4: copyout failed
fork_test failed, status=1509949440
# ls
uvm_copy: no memory
fork: failed uvm_copy
dash: 2: Cannot fork
```

# mmap機能を見直し

1. vmasの持ち方を配列からリストに変更
2. page cacheを削除

```
main: [CPU0] Init success.
iinit: ok
init log[1]: ok
proc[1] calls sys_execve
execve: proc[1] '/bin/init' start, uid=0, gid=0
proc[1] calls sys_gettid
proc[1] calls sys_openat
proc[1] calls sys_dup
proc[1] calls sys_dup
proc[1] calls sys_ioctl
proc[1] calls sys_writev
init: starting login
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_clone
proc[6] calls sys_gettid
pf_handler: fault_addr: 0x404000, flt: 3

====== TRAP FRAME DUMP ======
sp:	0x406dc0
tpidr:	0x404630
spsr:	0x60000000
esr:	0x9200004f
elr:	0x4013c4
x0:	0x6
x1:	0x404630
x2:	0x404568
x3:	0x8
x4:	0x406eef
x29:	0x406f20
x30:	0x40139c
====== DUMP END ======

pagefault handler
```

```
FLT_PERMISSION: addr=0x404000
- PTE_FLAGS=0x7c7
```

## fork時の新規プロセス用のマッピングテーブルのコピーでread only acess (COW0 にしているため

- page fault処理でFLT_PERMISSIONの場合の処理でmmaped以外のアドレスの処理を追加

```
# ls
dash: 1: Out of space
```

## `dash/src/memalloc.c#ckmalloc(size_t)`が負値の引数で呼び出されることがあるためだった

- どこからからよびだされているか発見できない
- とりあえず負値の場合、512を設定することでこのエラーは回避できた
- しかし、次のようなエラーが発生。cdするとenvp[]におかしな値が設定されるようだ

```
sys_execve: path=/bin/init, uargv=0x20, uenvp=0x0
init: starting dash
sys_execve: path=/bin/dash, uargv=0x406f68, uenvp=0x406f60  // uenvp[0] = 0
# ls
sys_execve: path=/bin/ls, uargv=0x43c680, uenvp=0x43f208    // uenvp[0] = "x\271C"
                                                            // uenvp[1] = 0xffffffff
bin  dev  etc  home  lib  usr
# cd bin
# ls
sys_execve: path=/bin/ls, uargv=0x43c680, uenvp=0x43f328
fetchstr: addr=0xffffffff, size=0x440000
sys_execve: failed to fetch envs
dash: 3: ls: Bad address
```

```
(gdb) p (char *)uargv[0]
cannot subscript something of type `long unsigned int'
(gdb) p (char *)((char **)uargv)[0]
$10 = 0x43c660 "ls"
(gdb) p (char *)((char **)uargv)[1]
$11 = 0x0
(gdb) p (char *)((char **)uenvp)[0]
$12 = 0x43b990 "x\271C"
(gdb) p (char *)((char **)uenvp)[1]
$13 = 0xffffffff <error: Cannot access memory at address 0xffffffff>
```

## `dash/src/memalloc.c`を修正

- stalloc(sizt_t)`の引数が負数の場合は0を返すように修正したら、今の所、うまく動いているみたいだ
- そもそもなぜunsignedの引数に負値がくるのかは不明

```
diff --git a/dash/src/memalloc.c b/dash/src/memalloc.c
index 60637da..77beb45 100644
--- a/dash/src/memalloc.c
+++ b/dash/src/memalloc.c
@@ -116,6 +116,8 @@ stalloc(size_t nbytes)
        char *p;
        size_t aligned;

+    if ((ssize_t)nbytes < 0) return 0;
+
        aligned = SHELL_ALIGN(nbytes);
        if (aligned > stacknleft) {
                size_t len;
```

## `mmaptest2`

```
# /bin/mmaptest2
[F-01] file backed mapping invalid file descriptor test
sys_mmap: addr=0x0, length=0x64, prot=0x3, flags=0x2, fd=-1, offset=0x0
sys_mmap: addr=0x0, length=0x64, prot=0x3, flags=0x2, fd=10, offset=0x0
sys_mmap: addr=0x0, length=0x64, prot=0x3, flags=0x2, fd=18, offset=0x0
[F-01] ok

[F-02] file backed mapping invalid flags test
sys_mmap: addr=0x0, length=0x64, prot=0x3, flags=0x0, fd=3, offset=0x0
[F-02] ok

[F-03] file backed writeable shared mapping on read only file test
sys_mmap: addr=0x0, length=0xc8, prot=0x3, flags=0x1, fd=3, offset=0x0
[F-03] ok

[F-04] file backed read only shared mapping on read only file test
sys_mmap: addr=0x0, length=0xc8, prot=0x1, flags=0x1, fd=3, offset=0x0
sys_munmap: addr=0x600000000000, length=0xc8
[F-04] ok

[F-05] file backed private mapping permission test
sys_mmap: addr=0x0, length=0xc8, prot=0x1, flags=0x2, fd=3, offset=0x0
sys_mmap: addr=0x0, length=0xc8, prot=0x3, flags=0x2, fd=3, offset=0x0
map_region: remap *pte=0x3a31d747, va=0x0
```

## `mmaptest`

```
# /bin/mmaptest
mmap_test starting
- open mmap.dur (3) RDONLY
[1] test mmap f
sys_mmap: addr=0x0, length=0x2000, prot=0x1, flags=0x2, fd=3, offset=0x0
- mmap 3 -> 2PAGE PROT_READ MAP_PRIVATE: addr=0x600000000000
sys_munmap: addr=0x600000000000, length=0x2000
- munmmap 2PAGE: addr=0x600000000000
[1] OK
[2] test mmap private
sys_mmap: addr=0x0, length=0x2000, prot=0x3, flags=0x2, fd=3, offset=0x0
- mmap 3 -> 2PAGE PROT_READ/WRITE MAP_PRIVATE: addr=0x600000000000
- close 3
- write 2PAGE = Z
map_region: remap *pte=0x60000039f077c7, va=0x600000000000
```

## 12/26現在

### mmaptest2

```
# mmaptest2
[F-01] file backed mapping invalid file descriptor test
 error: Bad file descriptor
 error: Bad file descriptor
 error: Bad file descriptor
[F-01] ok

[F-02] file backed mapping invalid flags test
 error: Invalid argument
[F-02] ok

[F-03] file backed writeable shared mapping on read only file test
 error: Permission denied
[F-03] ok

[F-04] file backed read only shared mapping on read only file test
- mapped addr=0x600000000000
[F-04] ok

[F-05] file backed private mapping permission test
- ret=0x600000000000
- ret2=0x600000001000
[F-05] ok

[F-06] file backed exceed mapping size test
 error: Invalid argument
[F-06] ok

[F-07] file backed exceed mapping count test
[F-07] ok

[F-08] file mmap on empty file test
[F-08] ok

[F-09] file backed private mapping test
file backed private mapping test failed 3: pos: 1, buf: p, ret: a
[F-09] failed

[F-10] file backed shared mapping test
[F-10] ok

[F-11] file backed mapping pagecache coherency test
[F-11] failed: at mmap 2
sys_munmap: addr=0x600000000000, length=0x1064
sys_munmap: addr=0xffffffffffffffff, length=0x1064
[F-11] failed

[F-12] file backed private mapping with fork test
pf_handler: FLT_TRANSLATION: addr: 600000000000
[F-12] ok

[F-13] file backed shared mapping with fork test
pf_handler: FLT_TRANSLATION: addr: 600000000000
[F-13] failed at strcmp parent
sys_munmap: addr=0x600000000000, length=0xc8

[F-14] file backed mapping with offset test
map_region: remap *pte=0x60000039ece747, va=0x600000000000
kern/console.c:284: kernel panic at cpu 0.

[A-01] anonymous private mapping test
map_file_pages: addr=600000000000, length=0x3000, perm=0x600000000000c1
- ret=0x600000000000
pf_handler: fault_addr: 0x600000000000, flt: 3

====== TRAP FRAME DUMP ======
sp:	0x417f10
tpidr:	0x411748
spsr:	0x40000000
esr:	0x9200004f
elr:	0x4036f0
x0:	0x2710
x1:	0x40bd01
x2:	0x600000000000
x3:	0x600000002710
x4:	0x40bd01
x29:	0x417f10
x30:	0x4036d0
====== DUMP END ======

[O-01] munmap only partial size test
map_file_pages: addr=600000000000, length=0x3000, perm=0x600000000000c1
pf_handler: fault_addr: 0x600000000000, flt: 3

====== TRAP FRAME DUMP ======
sp:	0x417f20
tpidr:	0x411748
spsr:	0x0
esr:	0x9200004f
elr:	0x404d78
x0:	0x2710
x1:	0x600000000000
x2:	0x3
x3:	0x22
x4:	0xffffffffffffffff
x29:	0x417f20
x30:	0x404d4c
====== DUMP END ======
```

### mmaptest

```
# mmaptest
mmap_test starting
- open mmap.dur (3) RDONLY
[1] test mmap f
- mmap 3 -> 2PAGE PROT_READ MAP_PRIVATE: addr=0x600000000000
sys_munmap: addr=0x600000000000, length=0x2000
- munmmap 2PAGE: addr=0x600000000000
[1] OK
[2] test mmap private
- mmap 3 -> 2PAGE PROT_READ/WRITE MAP_PRIVATE: addr=0x600000000000
- close 3
- write 2PAGE = Z
- p[1]=A
sys_munmap: addr=0x600000000000, length=0x2000
- munmmap 2PAGE: addr=0x600000000000
[2] OK
[3] test mmap read-only
- open mmap.dur (3) RDONALY
- mmap 3 -> 3PAGE PROT_READ/WRITE MAP_SHARED: addr=0xffffffffffffffff
- close 3
sys_munmap: addr=0xffffffffffffffff, length=0x3000
[3] OK
[4] test mmap read/write
- open mmap.dur (3) RDWR
- mmap 3 -> 3PAGE PROT_READ|WRITE MAP_SHARED: addr=0x600000000000
- close 3
mismatch at 2048, wanted 'A', got 0x0, addr=0x600000000800
[4] failed
[5] test mmap dirty
- open mmap.dur (3) RDWR
expect Z, but got A
[5] failed
[6] test not-mapped unmap
- munmmap PAGE: addr=0x600000002000
[6] OK
[7] test mmap two files
- open mmap1 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000000000
- close 3
- unlink mmap1
- open mmap2 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000001000
- close 3
- unlink mmap2
- munmap PAGE: addr=0x600000000000
- munmap PAGE: addr=0x600000001000
[7] OK
mmap_test: Total: 7, OK: 5, NG: 2
```

```
init: starting dash
fork: size=0x407000
FLT_PERMISSION: addr=0x406000, sz=0x407000
pte=0x3afb47c7, ref=2
FLT_PERMISSION: addr=0x404000, sz=0x407000
pte=0x3afb67c7, ref=2
FLT_PERMISSION: addr=0x406000, sz=0x407000
pte=0x3afb47c7, ref=1
FLT_PERMISSION: addr=0x43d000, sz=0x440000
pte=0x3ab6a707, ref=1
FLT_PERMISSION: addr=0x43d000, sz=0x440000
pte=0x3ab6a707, ref=1
```

```
init log[1]: ok
execve: proc[1] '/bin/init' start, uid=0, gid=0
- sz1=0x405000
- sz2=0x407000
init: starting dash
addr=0x406000, pte=0x3afb47c7, ref=2,  (1)
addr=0x404000, pte=0x3afb67c7, ref=2,  (1)
addr=0x406000, pte=0x3afb47c7, ref=2,  (1)
execve: proc[6] '/bin/dash' start, uid=0, gid=0
execve: proc[1] '/bin/dash' start, uid=0, gid=0
- sz1=0x43d000
- sz2=0x43f000
- sz1=0x43d000
- sz2=0x43f000
addr=0x43d000, pte=0x3ab69707, ref=1, pf_handler: FLT_TRANSLATION: addr: 3ab69000
 (2)
pf_handler: FLT_TRANSLATION: addr: 3a725000
```

## 12/29現在

```
iinit: ok
init log[1]: ok
execve: proc[1] '/bin/init' start, uid=0, gid=0
- sz1=0x405000
- sz2=0x407000
init: starting dash
fork: size=0x407000
- uvm_copy end
- copy_mmap_list end
fork: old=1, new=6
addr=0x404000, pte=0x3afb67c7, ref=2,  (1)
addr=0x406000, pte=0x3afb47c7, ref=2,  (1)
execve: proc[6] '/bin/dash' start, uid=0, gid=0
addr=0x406000, pte=0x3afb47c7, ref=2,  (1)
execve: proc[1] '/bin/dash' start, uid=0, gid=0
- sz1=0x43d000
- sz2=0x43f000
- sz1=0x43d000
- sz2=0x43f000
# # ls
fork: size=0x440000
- uvm_copy end
- copy_mmap_list end
fork: old=6, new=7
addr=0x43e000, pte=0x3ab687c7, ref=2,  (1)
unexpected interrupt 0 at cpu 0

====== TRAP FRAME DUMP ======
sp:	0x43ebe0
tpidr:	0x43bde8
spsr:	0x0
esr:	0x2000000
elr:	0x7
x0:	0x0
x1:	0x0
x2:	0x0
x3:	0x8
x4:	0x43f140
x29:	0x43ebe0
x30:	0x7
====== DUMP END ======

Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x7
 SPSR_EL1 0x0 FAR_EL1 0x43ebc0
	irq_error: irq of type 8 unimplemented.
```

## xv6-fudan

```
iinit: ok
init log[1]: ok
execve: proc[1] '/bin/init' start, uid=0, gid=0
- sz1=0x40a000
- sz2=0x40c000
init: starting dash
fork: size=0x40c000
- uvm_copy end
fork: tf->tf: p=0x11, np=0x0
fork: elr_el1: p=0x403174, np=0x403174
fork: old=1, new=6
addr=0x409000, pte=0x3afb17c7, ref=2,  (1)
addr=0x40b000, pte=0x3afaf7c7, ref=2,  (1)
addr=0x40b000, pte=0x3afaf7c7, ref=2,  (1)
addr=0x409000, pte=0x3afb17c7, ref=2,  (1)
pid: 0
child calls dash
execve: proc[6] '/bin/dash' start, uid=0, gid=0
pid: 0
child calls dash
execve: proc[1] '/bin/dash' start, uid=0, gid=0
- sz1=0x43d000
- sz2=0x43f000
- sz1=0x43d000
- sz2=0x43f000
# #
```

## xv6-arm8v8

```
iinit: ok.
initlog: ok.
mknodat: path 'console', major:minor 1:1
init: starting sh.
fork: tf->x0:  p=0x11, np=0x0
fork: elr_el1: p=0x402f78, np=0x402f78
fork: old=1, new=6
pid=0
pid=0 execve(sh, argv)
pid=6
parent wait
$ ls
fork: tf->x0:  p=0x11, np=0x0
fork: elr_el1: p=0x404bc0, np=0x404bc0
fork: old=6, new=7
```

## pagecacheを使うようにした

### mmaptest

#### 実行例1

```
# mmaptest
mmap_testスタート
- open mmap.dur (fd=3) READ ONLY
[1] test mmap f
- mmap fd=3 -> 2ページ分PROT_READ MAP_PRIVATEでマップ: addr=0x600000000000
- munmmap 2PAGE: addr=0x600000000000
[1] OK
[2] test mmap private
- mmap fd=3 -> 2ページ分をPAGE PROT_READ/WRITE, MAP_PRIVATEでマップ: addr=0x600000000000
- close 3
- write 2PAGE = Z
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000000000, flags: 2, prot: 3, fd: 4
pf_handler: ec=36, addr=0x600000000010, flt=1
pf_handler: ec=36, addr=0x600000000010, flt=2
map_region: remap *pte=0x60000039b26043, va=0x600000000000
kern/console.c:284: kernel panic at cpu 0.
```

### 実行例2

```
# mmaptest
mmap_testスタート
- open mmap.dur (fd=3) READ ONLY
[1] test mmap f
- mmap fd=3 -> 2ページ分PROT_READ MAP_PRIVATEでマップ: addr=0x600000000000
- munmmap 2PAGE: addr=0x600000000000
[1] OK
[2] test mmap private
- mmap fd=3 -> 2ページ分をPAGE PROT_READ/WRITE, MAP_PRIVATEでマップ: addr=0x600000000000
- close 3
mismatch at 2048, wanted 'A', got 0x0, addr=0x600000000800
mismatch at 2049, wanted 'A', got 0x0, addr=0x600000000801
mismatch at 2050, wanted 'A', got 0x0, addr=0x600000000802
mismatch at 2051, wanted 'A', got 0x0, addr=0x600000000803
- ret: -4
[2] failed
[3] test mmap read-only
- open mmap.dur (3) RDONALY
- mmap 3 -> 3PAGE PROT_READ/WRITE MAP_SHARED: addr=0xffffffffffffffff
- close 3
[3] OK
[4] test mmap read/write
- open mmap.dur (3) RDWR
- mmap 3 -> 3PAGE PROT_READ|WRITE MAP_SHARED: addr=0x600000000000
- close 3
- print 2PAGE = Z
- munmmap 2PAGE: addr=0x600000000000
[4] OK
[5] test mmap dirty
- open mmap.dur (3) RDWR
- close 3
[5] OK
[6] test not-mapped unmap
- munmmap PAGE: addr=0x600000002000
[6] OK
[7] test mmap two files
- open mmap1 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000000000
- close 3
- unlink mmap1
- open mmap2 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000001000
- close 3
- unlink mmap2
- munmap PAGE: addr=0x600000000000
- munmap PAGE: addr=0x600000001000
[7] OK
mmap_test: Total: 7, OK: 6, NG: 1
```

### mmaptest2

```
# mmaptest2
[F-01] 不正なfdを指定した場合のテスト
 error: Bad file descriptor
 error: Bad file descriptor
 error: Bad file descriptor
[F-01] ok

[F-02] 不正なフラグを指定した場合のテスト
 error: Invalid argument
[F-02] ok

[F-03] readonlyファイルにPROT_WRITEな共有マッピングをした場合のテスト
 error: Permission denied
[F-03] ok

[F-04] readonlyファイルにPROT_READのみの共有マッピングをした場合のテスト
- mapped addr=0x600000000000
[F-04] ok

[F-05] PROT指定の異なるプライベートマッピンのテスト
- ret=0x600000000000
- ret2=0x600000001000
pf_handler: ec=36, addr=0x600000001000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000001000, flags: 2, prot: 3, fd: 5
pf_handler: ec=36, addr=0x600000001004, flt=1
pf_handler: ec=36, addr=0x600000001004, flt=2
map_region: remap *pte=0x600000392c7043, va=0x600000001000
kern/console.c:284: kernel panic at cpu 0.
```

## `get_perm()`を修正

```
# mmaptest
mmap_testスタート
- open mmap.dur (fd=3) READ ONLY
[1] test mmap f
- mmap fd=3 -> 2ページ分PROT_READ MAP_PRIVATEでマップ: addr=0x600000000000
- munmmap 2PAGE: addr=0x600000000000
[1] OK
[2] test mmap private
- mmap fd=3 -> 2ページ分をPAGE PROT_READ/WRITE, MAP_PRIVATEでマップ: addr=0x600000000000
- close 3
mismatch at 2048, wanted 'A', got 0x0, addr=0x600000000800                      // READ エラー
- ret: -1
[2] failed
[3] test mmap read-only
- open mmap.dur (3) RDONALY
- mmap 3 -> 3PAGE PROT_READ/WRITE MAP_SHARED: addr=0xffffffffffffffff
- close 3
[3] OK
[4] test mmap read/write
- open mmap.dur (3) RDWR
- mmap 3 -> 3PAGE PROT_READ|WRITE MAP_SHARED: addr=0x600000000000
- close 3
- print 2PAGE = Z
- munmmap 2PAGE: addr=0x600000000000
[4] OK
[5] test mmap dirty
- open mmap.dur (3) RDWR
- close 3
[5] OK
[6] test not-mapped unmap
- munmmap PAGE: addr=0x600000002000
[6] OK
[7] test mmap two files
- open mmap1 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000000000
- close 3
- unlink mmap1
- open mmap2 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000001000
- close 3
- unlink mmap2
- munmap PAGE: addr=0x600000000000
- munmap PAGE: addr=0x600000001000
[7] OK
mmap_test: Total: 7, OK: 6, NG: 1

## 再実行

# mmaptest2
[F-01] 不正なfdを指定した場合のテスト
 error: Bad file descriptor
 error: Bad file descriptor
 error: Bad file descriptor
[F-01] ok

[F-02] 不正なフラグを指定した場合のテスト
 error: Invalid argument
[F-02] ok

[F-03] readonlyファイルにPROT_WRITEな共有マッピングをした場合のテスト
 error: Permission denied
[F-03] ok

[F-04] readonlyファイルにPROT_READのみの共有マッピングをした場合のテスト
- mapped addr=0x600000000000
[F-04] ok

[F-05] PROT指定の異なるプライベートマッピンのテスト
- ret=0x600000000000
- ret2=0x600000001000
pf_handler: ec=36, addr=0x600000001000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000001000, flags: 2, prot: 3, fd: 5
copy_mmap_pages: addr=600000001000, length=0xc8, perm=0x60000000000747
- start=600000001000, pte=ffff000039b1f008, *pte=0x60000039b1d747
[F-05] ok

[F-06] MMAPTOPを超えるサイズをマッピングした場合のテスト
 error: Invalid argument
[F-06] ok

[F-07] 連続したマッピングを行うテスト
[F-07] ok

[F-08] 空のファイルを共有マッピングした場合のテスト
[F-08] ok

[F-09] ファイルが背後にあるプライベートマッピング
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000000000, flags: 2, prot: 3, fd: 5
copy_mmap_pages: addr=600000000000, length=0x3e8, perm=0x60000000000747
- start=600000000000, pte=ffff000039b1f000, *pte=0x60000039aee747
file backed private mapping test failed 3: pos: 0, buf: p, ret: a               //
[F-09] failed

[F-10] ファイルが背後にある共有マッピングのテスト
[F-10] ok

[F-11] file backed mapping pagecache coherency test
copy_page: get_page failed                                                      //
[F-11] failed: at mmap 2
[F-11] failed

[F-12] file backed private mapping with fork test
pf_handler: ec=36, addr=0x600000000000, flt=1
pf_handler: FLT_TRANSLATION (pte=0): addr: 600000000000
[F-12] ok

[F-13] file backed shared mapping with fork test
pf_handler: ec=36, addr=0x600000000000, flt=1
pf_handler: FLT_TRANSLATION (pte=0): addr: 600000000000
[F-13] failed at strcmp parent: ret2[0]=a, buf[0]=a                             //

[F-14] オフセットを指定したプライベートマッピングのテスト
map_region: remap *pte=0x60000039aee747, va=0x600000000000                      //
kern/console.c:284: kernel panic at cpu 0.

## 再実行

# mmaptest
mmap_testスタート
- open mmap.dur (fd=3) READ ONLY
[1] test mmap f
- mmap fd=3 -> 2ページ分PROT_READ MAP_PRIVATEでマップ: addr=0x600000000000
- munmmap 2PAGE: addr=0x600000000000
[1] OK
[2] test mmap private
- mmap fd=3 -> 2ページ分をPAGE PROT_READ/WRITE, MAP_PRIVATEでマップ: addr=0x600000000000
- close 3
- write 2PAGE = Z                                                               // READエラーはなくなっている
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000000000, flags: 2, prot: 3, fd: 4
copy_mmap_pages: addr=600000000000, length=0x2000, perm=0x60000000000747
- start=600000000000, pte=ffff000039b28000, *pte=0x60000039b26747
- start=600000001000, pte=ffff000039b28008, *pte=0x60000039b25747
- p[1]=A
- munmmap 2PAGE: addr=0x600000000000
[2] OK
[3] test mmap read-only
- open mmap.dur (3) RDONALY
- mmap 3 -> 3PAGE PROT_READ/WRITE MAP_SHARED: addr=0xffffffffffffffff
- close 3
[3] OK
[4] test mmap read/write
- open mmap.dur (3) RDWR
- mmap 3 -> 3PAGE PROT_READ|WRITE MAP_SHARED: addr=0x600000000000
- close 3
- print 2PAGE = Z
- munmmap 2PAGE: addr=0x600000000000
[4] OK
[5] test mmap dirty
- open mmap.dur (3) RDWR
- close 3
[5] OK
[6] test not-mapped unmap
- munmmap PAGE: addr=0x600000002000
[6] OK
[7] test mmap two files
- open mmap1 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000000000
- close 3
- unlink mmap1
- open mmap2 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000001000
- close 3
- unlink mmap2
- munmap PAGE: addr=0x600000000000
- munmap PAGE: addr=0x600000001000
[7] OK
mmap_test: Total: 7, OK: 7, NG: 0

fork_test starting
p1[PGSIZE]=A
p2[PGSIZE]=A
pf_handler: ec=36, addr=0x600000000000, flt=1
pf_handler: FLT_TRANSLATION (pte=0): addr: 600000000000
mismatch at 0, wanted 'A', got 0x0, addr=0x600000000000                         //
- ret: -1
wait4: copyout failed
fork_test failed, status=1509949440

# mmaptest2
[F-01] 不正なfdを指定した場合のテスト
 error: Bad file descriptor
 error: Bad file descriptor
 error: Bad file descriptor
[F-01] ok

[F-02] 不正なフラグを指定した場合のテスト
 error: Invalid argument
[F-02] ok

[F-03] readonlyファイルにPROT_WRITEな共有マッピングをした場合のテスト
 error: Permission denied
[F-03] ok

[F-04] readonlyファイルにPROT_READのみの共有マッピングをした場合のテスト
- mapped addr=0x600000000000
[F-04] ok

[F-05] PROT指定の異なるプライベートマッピンのテスト
- ret=0x600000000000
- ret2=0x600000001000
pf_handler: ec=36, addr=0x600000001000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000001000, flags: 2, prot: 3, fd: 5
copy_mmap_pages: addr=600000001000, length=0xc8, perm=0x60000000000747
- start=600000001000, pte=ffff000038ea9008, *pte=0x60000038ea7747
[F-05] ok

[F-06] MMAPTOPを超えるサイズをマッピングした場合のテスト
 error: Invalid argument
[F-06] ok

[F-07] 連続したマッピングを行うテスト
[F-07] ok

[F-08] 空のファイルを共有マッピングした場合のテスト
[F-08] ok

[F-09] ファイルが背後にあるプライベートマッピング
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000000000, flags: 2, prot: 3, fd: 5
copy_mmap_pages: addr=600000000000, length=0x3e8, perm=0x60000000000747
- start=600000000000, pte=ffff000038ea9000, *pte=0x60000038e78747
file backed private mapping test failed 3: pos: 0, buf: p, ret: a               // エラー再現
[F-09] failed

[F-10] ファイルが背後にある共有マッピングのテスト
[F-10] ok

[F-11] file backed mapping pagecache coherency test
[F-11] failed at strcmp                                                         //
[F-11] failed

[F-12] file backed private mapping with fork test
pf_handler: ec=36, addr=0x600000000000, flt=1
pf_handler: FLT_TRANSLATION (pte=0): addr: 600000000000
[F-12] ok

[F-13] file backed shared mapping with fork test
pf_handler: ec=36, addr=0x600000000000, flt=1
pf_handler: FLT_TRANSLATION (pte=0): addr: 600000000000
[F-13] failed at strcmp parent: ret2[0]=a, buf[0]=a                             //

[F-14] オフセットを指定したプライベートマッピングのテスト
map_region: remap *pte=0x60000038e78747, va=0x600000000000                      //
kern/console.c:284: kernel panic at cpu 0.
```

## `proc.c#wait4()`を修正

- mmaptestが全部成功

```
# mmaptest
mmap_testスタート
- open mmap.dur (fd=3) READ ONLY
[1] test mmap f
- mmap fd=3 -> 2ページ分PROT_READ MAP_PRIVATEでマップ: addr=0x600000000000
- munmmap 2PAGE: addr=0x600000000000
[1] OK
[2] test mmap private
- mmap fd=3 -> 2ページ分をPAGE PROT_READ/WRITE, MAP_PRIVATEでマップ: addr=0x600000000000
- closed 3
- write 2PAGE = Z
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000000000, flags: 2, prot: 3, fd: 4
- p[1]=A
- munmmap 2PAGE: addr=0x600000000000
[2] OK
[3] test mmap read-only
- open mmap.dur (3) RDONALY
- mmap 3 -> 3PAGE PROT_READ/WRITE MAP_SHARED: addr=0xffffffffffffffff
- close 3
[3] OK
[4] test mmap read/write
- open mmap.dur (3) RDWR
- mmap 3 -> 3PAGE PROT_READ|WRITE MAP_SHARED: addr=0x600000000000
- close 3
- print 2PAGE = Z
- munmmap 2PAGE: addr=0x600000000000
[4] OK
[5] test mmap dirty
- open mmap.dur (3) RDWR
- close 3
[5] OK
[6] test not-mapped unmap
- munmmap PAGE: addr=0x600000002000
[6] OK
[7] test mmap two files
- open mmap1 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000000000
- close 3
- unlink mmap1
- open mmap2 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000001000
- close 3
- unlink mmap2
- munmap PAGE: addr=0x600000000000
- munmap PAGE: addr=0x600000001000
[7] OK
mmap_test: Total: 7, OK: 7, NG: 0

fork_test starting
p1[PGSIZE]=A
p2[PGSIZE]=A
fork_test OK
mmaptest: all tests succeeded
```

- mmaptest2はまだ問題あり

```
# mmaptest2
[F-01] 不正なfdを指定した場合のテスト
 error: Bad file descriptor
 error: Bad file descriptor
 error: Bad file descriptor
[F-01] ok

[F-02] 不正なフラグを指定した場合のテスト
 error: Invalid argument
[F-02] ok

[F-03] readonlyファイルにPROT_WRITEな共有マッピングをした場合のテスト
 error: Permission denied
[F-03] ok

[F-04] readonlyファイルにPROT_READのみの共有マッピングをした場合のテスト
- mapped addr=0x600000000000
[F-04] ok

[F-05] PROT指定の異なるプライベートマッピンのテスト
- ret=0x600000000000
- ret2=0x600000001000
pf_handler: ec=36, addr=0x600000001000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000001000, flags: 2, prot: 3, fd: 5
[F-05] ok

[F-06] MMAPTOPを超えるサイズをマッピングした場合のテスト
 error: Invalid argument
[F-06] ok

[F-07] 連続したマッピングを行うテスト
[F-07] ok

[F-08] 空のファイルを共有マッピングした場合のテスト
[F-08] ok

[F-09] ファイルが背後にあるプライベートマッピング
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000000000, flags: 2, prot: 3, fd: 5
file backed private mapping test failed 3: pos: 0, buf: p, ret: a
[F-09] failed

[F-10] ファイルが背後にある共有マッピングのテスト
[F-10] ok

[F-11] file backed mapping pagecache coherency test
[F-11] failed at strcmp
[F-11] failed

[F-12] file backed private mapping with fork test
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: fault_addr: 0x600000000000, flt: 3

====== TRAP FRAME DUMP ======
sp:	0x417e30
tpidr:	0x411748
spsr:	0x20000000
esr:	0x9200004f
elr:	0x4078a4
x0:	0x600000000000
x1:	0x6e
x2:	0x32
x3:	0x407850
x4:	0x600000000032
x29:	0x417e30
x30:	0x402470
====== DUMP END ======

pagefault handler
```

## F-09

### i = 40

```
write(u) (0x600000000000): aaaaaaaapp pppppppppp pppppppppp pppppppppp aaaaaaaaaa
write(u) (0x417768):       pppppppppp pppppppppp pppppppppp pppppppppp aaaaaaaaaa
[F-09] failed
```

### i = 8

```
write(u) (0x600000000000): aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
write(u) (0x417768):       aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
[F-09] ok
```

### i = 16

```
write(u) (0x600000000000): pppppppppp ppppppaaaa aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
write(u) (0x417768):       pppppppppp ppppppaaaa aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
[F-09] ok
```

### i = 20

```
write(u) (0x600000000000): aaaapppppp pppppppppp aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
write(u) (0x417768):       pppppppppp pppppppppp aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
[F-09] failed
```

### i = 32

```
write(u) (0x600000000000): pppppppppp pppppppppp pppppppppp ppaaaaaaaaaaaaaaaaaa
write(u) (0x417768):       pppppppppp pppppppppp pppppppppp ppaaaaaaaaaaaaaaaaaa
[F-09] ok
```

### i = 48

```
write(u) (0x600000000000): aaaaaaaaaa aaaaaapppp pppppppppp pppppppppp ppppppppaa
write(u) (0x417768):       pppppppppp pppppppppp pppppppppp pppppppppp ppppppppaa
[F-09] failed
```


### i = 50

```
write(u) (0x600000000000): aaaaaaaaaa aaaaaaaapp pppppppppp pppppppppp pppppppppp aaaaaaaaaa
write(u) (0x417768):       pppppppppp pppppppppp pppppppppp pppppppppp pppppppppp aaaaaaaaaa
[F-09] failed
```

### i = 64

```
write(u) (0x600000000000): aaaaaaaaaa aaaaaaaaaa aaaaaaaaaa aapppppppp pppppppppp pppppppppp ppppaaaaaa
write(u) (0x417768):       pppppppppp pppppppppp pppppppppp pppppppppp pppppppppp pppppppppp ppppaaaaaa
[F-09] failed
```

## mmapを受ける変数を`volatile`にするとどんなiでも正常に設定されるようになった

- F-08も同じ

[stack overflow](https://stackoverflow.com/questions/3207628/why-doesnt-posix-mmap-return-a-volatile-void)

## アドレス指定あり（MAP_FIXEDではない）の場合のアドレス計算を修正

- mmaptestとmmaptest2のF系列のテストを通過
- mmaptestのforkが失敗していた

```
# mmaptest2
[F-01] 不正なfdを指定した場合のテスト
 error: Bad file descriptor
 error: Bad file descriptor
 error: Bad file descriptor
[F-01] ok

[F-02] 不正なフラグを指定した場合のテスト
 error: Invalid argument
[F-02] ok

[F-03] readonlyファイルにPROT_WRITEな共有マッピングをした場合のテスト
 error: Permission denied
[F-03] ok

[F-04] readonlyファイルにPROT_READのみの共有マッピングをした場合のテスト
[F-04] ok

[F-05] PROT指定の異なるプライベートマッピンのテスト
pf_handler: ec=36, addr=0x600000001000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000001000, flags: 2, prot: 3, fd: 5
[F-05] ok

[F-06] MMAPTOPを超えるサイズをマッピングした場合のテスト
 error: Invalid argument
[F-06] ok

[F-07] 連続したマッピングを行うテスト
[F-07] ok

[F-08] 空のファイルを共有マッピングした場合のテスト
[F-08] ok

[F-09] ファイルが背後にあるプライベートマッピング
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000000000, flags: 2, prot: 3, fd: 5
[F-09] ok

[F-10] ファイルが背後にある共有マッピングのテスト
[F-10] ok

[F-11] file backed mapping pagecache coherency test
[F-11] ok

[F-12] file backed private mapping with fork test
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000000000, flags: 2, prot: 3, fd: 6
[F-12] ok

[F-13] file backed shared mapping with fork test
[F-13] ok

[F-14] オフセットを指定したプライベートマッピングのテスト
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000000000, flags: 2, prot: 3, fd: 7
[F-14] ok

[F-15] file backed valid provided address test
[F-15] ok

[F-16] file backed invalid provided address test
mmap: invalid addr=0x5ffffffff000
[F-16] ok

[F-17] 指定されたアドレスが既存のマッピングアドレスと重なる場合
[F-17] ok

[F-18] ２つのマッピングの間のアドレスを指定したマッピングのテスト
[F-18] ok

[F-19] ２つのマッピングの間に不可能なアドレスを指定した場合
[F-19]  ok

[F-20] 共有マッピングでファイル容量より大きなサイズを指定した場合
[F-20] ok

[F-21] write onlyファイルへのREAD/WRITE共有マッピングのテスト
[F-21] ok

# mmaptest
mmap_testスタート
- open mmap.dur (fd=3) READ ONLY
[1] test mmap f
- mmap fd=3 -> 2ページ分PROT_READ MAP_PRIVATEでマップ: addr=0x600000000000
- munmmap 2PAGE: addr=0x600000000000
[1] OK
[2] test mmap private
- mmap fd=3 -> 2ページ分をPAGE PROT_READ/WRITE, MAP_PRIVATEでマップ: addr=0x600000000000
- closed 3
- write 2PAGE = Z
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: FLT_PERMISSION: addr: 600000000000, flags: 2, prot: 3, fd: 4
- p[1]=A
- munmmap 2PAGE: addr=0x600000000000
[2] OK
[3] test mmap read-only
- open mmap.dur (3) RDONALY
- mmap 3 -> 3PAGE PROT_READ/WRITE MAP_SHARED: addr=0xffffffffffffffff
- close 3
[3] OK
[4] test mmap read/write
- open mmap.dur (3) RDWR
- mmap 3 -> 3PAGE PROT_READ|WRITE MAP_SHARED: addr=0x600000000000
- close 3
- print 2PAGE = Z
- munmmap 2PAGE: addr=0x600000000000
[4] OK
[5] test mmap dirty
- open mmap.dur (3) RDWR
- close 3
[5] OK
[6] test not-mapped unmap
- munmmap PAGE: addr=0x600000002000
[6] OK
[7] test mmap two files
- open mmap1 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000000000
- close 3
- unlink mmap1
- open mmap2 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x600000001000
- close 3
- unlink mmap2
- munmap PAGE: addr=0x600000000000
- munmap PAGE: addr=0x600000001000
[7] OK
mmap_test: Total: 7, OK: 7, NG: 0

fork_test starting
p1[PGSIZE]=A
p2[PGSIZE]=A
mismatch at 0, wanted 'A', got 0x0, addr=0x600000000000
- fork parent v1(p1): ret: -1
fork_test OK
mmaptest: all tests succeeded
```

### 子プロセスでp1を半分unmap

```
fork_test starting
p1[PGSIZE]=A
p2[PGSIZE]=A
munmap: addr=0x600000000000, *pte=0x60000039b267c7
mismatch at 0, wanted 'A', got 0x0, addr=0x600000000000
mismatch at 1, wanted 'A', got 0x90, addr=0x600000000001
mismatch at 2, wanted 'A', got 0x70, addr=0x600000000002
mismatch at 3, wanted 'A', got 0x39, addr=0x600000000003
mismatch at 4, wanted 'A', got 0x0, addr=0x600000000004
mismatch at 5, wanted 'A', got 0x0, addr=0x600000000005
mismatch at 6, wanted 'A', got 0xff, addr=0x600000000006
mismatch at 7, wanted 'A', got 0xff, addr=0x600000000007        // FFFF000039709000
- fork parent v1(p1): ret: -8
munmap: addr=0x600000000000, *pte=0x60000039b267c7
munmap: addr=0x600000002000, *pte=0x60000039b257c7
munmap: addr=0x600000003000, *pte=0x60000039b237c7
fork_test OK
mmaptest: all tests succeeded
```

### 子プロセスでp1を全部unmap

```
fork_test starting
p1[PGSIZE]=A
p2[PGSIZE]=A
munmap: addr=0x600000000000, *pte=0x60000039b267c7
munmap: addr=0x600000001000, *pte=0x60000039b247c7
mismatch at 0, wanted 'A', got 0x0, addr=0x600000000000                 // FFFF000039709000
mismatch at 1, wanted 'A', got 0x90, addr=0x600000000001
mismatch at 2, wanted 'A', got 0x70, addr=0x600000000002
mismatch at 3, wanted 'A', got 0x39, addr=0x600000000003
mismatch at 4, wanted 'A', got 0x0, addr=0x600000000004
mismatch at 5, wanted 'A', got 0x0, addr=0x600000000005
mismatch at 6, wanted 'A', got 0xff, addr=0x600000000006
mismatch at 7, wanted 'A', got 0xff, addr=0x600000000007
mismatch at 4096, wanted 'A', got 0x0, addr=0x600000001000              // FFFF000039b26000
mismatch at 4097, wanted 'A', got 0x60, addr=0x600000001001             //  = V2P(PTE_ADDR(*pte))
mismatch at 4098, wanted 'A', got 0xb2, addr=0x600000001002
mismatch at 4099, wanted 'A', got 0x39, addr=0x600000001003
mismatch at 4100, wanted 'A', got 0x0, addr=0x600000001004
mismatch at 4101, wanted 'A', got 0x0, addr=0x600000001005
mismatch at 4102, wanted 'A', got 0xff, addr=0x600000001006
mismatch at 4103, wanted 'A', got 0xff, addr=0x600000001007
- fork parent v1(p1): ret: -16
munmap: addr=0x600000000000, *pte=0x60000039b267c7
munmap: addr=0x600000002000, *pte=0x60000039b257c7
munmap: addr=0x600000003000, *pte=0x60000039b237c7
fork_test OK
mmaptest: all tests succeeded
```

### 子プロセスでp1とp2を全部unmap

```
munmap: addr=0x600000000000, *pte=0x60000039b267c7
munmap: addr=0x600000001000, *pte=0x60000039b247c7
munmap: addr=0x600000002000, *pte=0x60000039b257c7
munmap: addr=0x600000003000, *pte=0x60000039b237c7
mismatch at 0, wanted 'A', got 0x0, addr=0x600000000000
mismatch at 1, wanted 'A', got 0x90, addr=0x600000000001
mismatch at 2, wanted 'A', got 0x70, addr=0x600000000002
mismatch at 3, wanted 'A', got 0x39, addr=0x600000000003
mismatch at 4, wanted 'A', got 0x0, addr=0x600000000004
mismatch at 5, wanted 'A', got 0x0, addr=0x600000000005
mismatch at 6, wanted 'A', got 0xff, addr=0x600000000006
mismatch at 7, wanted 'A', got 0xff, addr=0x600000000007
mismatch at 4096, wanted 'A', got 0x0, addr=0x600000001000
mismatch at 4097, wanted 'A', got 0x60, addr=0x600000001001
mismatch at 4098, wanted 'A', got 0xb2, addr=0x600000001002
mismatch at 4099, wanted 'A', got 0x39, addr=0x600000001003
mismatch at 4100, wanted 'A', got 0x0, addr=0x600000001004
mismatch at 4101, wanted 'A', got 0x0, addr=0x600000001005
mismatch at 4102, wanted 'A', got 0xff, addr=0x600000001006
mismatch at 4103, wanted 'A', got 0xff, addr=0x600000001007
- fork parent v1(p1): ret: -16
mismatch at 0, wanted 'A', got 0x0, addr=0x600000002000
mismatch at 1, wanted 'A', got 0x40, addr=0x600000002001
mismatch at 2, wanted 'A', got 0xb2, addr=0x600000002002
mismatch at 3, wanted 'A', got 0x39, addr=0x600000002003
mismatch at 4, wanted 'A', got 0x0, addr=0x600000002004
mismatch at 5, wanted 'A', got 0x0, addr=0x600000002005
mismatch at 6, wanted 'A', got 0xff, addr=0x600000002006
mismatch at 7, wanted 'A', got 0xff, addr=0x600000002007
mismatch at 4096, wanted 'A', got 0x0, addr=0x600000003000
mismatch at 4097, wanted 'A', got 0x50, addr=0x600000003001
mismatch at 4098, wanted 'A', got 0xb2, addr=0x600000003002
mismatch at 4099, wanted 'A', got 0x39, addr=0x600000003003
mismatch at 4100, wanted 'A', got 0x0, addr=0x600000003004
mismatch at 4101, wanted 'A', got 0x0, addr=0x600000003005
mismatch at 4102, wanted 'A', got 0xff, addr=0x600000003006
mismatch at 4103, wanted 'A', got 0xff, addr=0x600000003007
- fork parent v1(p2): ret: -16
munmap: addr=0x600000000000, *pte=0x60000039b267c7
munmap: addr=0x600000001000, *pte=0x60000039b247c7
munmap: addr=0x600000002000, *pte=0x60000039b257c7
munmap: addr=0x600000003000, *pte=0x60000039b237c7
```

### 子プロセスでp1とp2をunmapしないと成功

```
fork_test starting
p1[PGSIZE]=A
p2[PGSIZE]=A
munmap: addr=0x600000000000, pte=0xffff000039b28000, *pte=0x60000039b267c7
munmap: addr=0x600000001000, pte=0xffff000039b28008, *pte=0x60000039b247c7
munmap: addr=0x600000002000, pte=0xffff000039b28010, *pte=0x60000039b257c7
munmap: addr=0x600000003000, pte=0xffff000039b28018, *pte=0x60000039b237c7
fork_test OK
```

### 書かれているアドレスの内容を出力してみた

- 一つ前のページのアドレスを持っている
- このエラーが生じると次のコマンドがkalloc()で0が返されるエラーとなる
- munmapでkalloc関係がおかしくなるのか？
- 関係があるのかないのか、mmaptest2ではsttyのecho属性が無効になる

```
fork_test starting
ffff00003970b000 -> ffff00003970a000            // 一つ前のページアドレス
ffff00003970a000 -> ffff000039709000            // 一つ前のページアドレス
region: 0xffff00003abd2fc0                      // kmallocによるregionのアドレス

# ls
uvm_copy: no memory
fork: failed uvm_copy
dash: 2: Cannot fork
```

```
# mmaptest

fork_test starting
(1) p1: 41414141414141414141414141414141
(2) p1: 41414141414141414141414141414141
munmap: pid=8, addr=0x600000000000, length=0x1000
- target region=0xffff00003abd2f20, addr=0x600000000000, length=0x2000, flag=0x1, prot=0x1, offset=0
- unmap: pte=0xffff00003970c000, pa=0x3a382000
- new region: addr=0x600000001000, length=0x1000
(2_2) p1: 41414141414141414141414141414141
node: 0xffff00003abd2f20 deleted
prev: 0x0, prev->next: 0x0
node: 0xffff00003abd2ed0 deleted
prev: 0x0, prev->next: 0x0
(3) p1: 00b070390000ffff4141414141414141
(3_2) p1: 41414141414141414141414141414141
mismatch at 0, wanted 'A', got 0x0, addr=0x600000000000
mismatch at 1, wanted 'A', got 0xb0, addr=0x600000000001
mismatch at 2, wanted 'A', got 0x70, addr=0x600000000002
mismatch at 3, wanted 'A', got 0x39, addr=0x600000000003
mismatch at 4, wanted 'A', got 0x0, addr=0x600000000004
mismatch at 5, wanted 'A', got 0x0, addr=0x600000000005
mismatch at 6, wanted 'A', got 0xff, addr=0x600000000006
mismatch at 7, wanted 'A', got 0xff, addr=0x600000000007
- fork parent v1(p1): ret: -8
munmap: pid=7, addr=0x600000000000, length=0x1000
- target region=0xffff00003abd2fc0, addr=0x600000000000, length=0x2000, flag=0x1, prot=0x1, offset=0
- unmap: pte=0xffff000039b28000, pa=0x3a382000
- new region: addr=0x600000001000, length=0x1000
munmap: pid=7, addr=0x600000002000, length=0x2000
- target region=0xffff00003abd2f70, addr=0x600000002000, length=0x2000, flag=0x1, prot=0x1, offset=0
- unmap: pte=0xffff000039b28010, pa=0x39b26000
- unmap: pte=0xffff000039b28018, pa=0x39b25000
node: 0xffff00003abd2f70 deleted
prev: 0xffff00003abd2fc0, prev->next: 0x0
fork_test OK
mmaptest: all tests succeeded
node: 0xffff00003abd2fc0 deleted
prev: 0x0, prev->next: 0x0
```

- syscallを出力してみた

```
# mmaptest
sys_rt_sigprocmask: how: 2, set: 0x7fffffff, oldset: not
sys_rt_sigprocmask: how: 2, set: 0x0, oldset: not
sys_rt_sigprocmask: how: 2, set: 0x0, oldset: not

fork_test starting
p1 = 0x600000000000
p2 = 0x600000002000
(1) p1: 41414141414141414141414141414141
sys_rt_sigprocmask: how: 0, set: 0x7fffffff, oldset: set
sys_rt_sigprocmask: how: 0, set: 0xffffffff, oldset: set
sys_rt_sigprocmask: how: 2, set: 0x0, oldset: not
sys_rt_sigprocmask: how: 2, set: 0x0, oldset: not
sys_rt_sigprocmask: how: 2, set: 0x0, oldset: not
sys_rt_sigprocmask: how: 2, set: 0x0, oldset: not
(2) p1[0-15]:      41414141414141414141414141414141
sys_munmap: addr=0x600000000000, length=0x1000
(2) p1[4096-4111]: 41414141414141414141414141414141
(3) p1: 00b070390000ffff4141414141414141
(3_2) p1: 41414141414141414141414141414141
mismatch at 0, wanted 'A', got 0x0, addr=0x600000000000
mismatch at 1, wanted 'A', got 0xb0, addr=0x600000000001
mismatch at 2, wanted 'A', got 0x70, addr=0x600000000002
mismatch at 3, wanted 'A', got 0x39, addr=0x600000000003
mismatch at 4, wanted 'A', got 0x0, addr=0x600000000004
mismatch at 5, wanted 'A', got 0x0, addr=0x600000000005
mismatch at 6, wanted 'A', got 0xff, addr=0x600000000006
mismatch at 7, wanted 'A', got 0xff, addr=0x600000000007
- fork parent v1(p1): ret: -8
sys_munmap: addr=0x600000000000, length=0x2000
sys_munmap: addr=0x600000002000, length=0x2000
fork_test OK
mmaptest: all tests succeeded
```

- A系列はいきなりエラー

```
[A-01] anonymous private mapping test
- ret=0x600000000000
pf_handler: ec=36, addr=0x600000000000, flt=3
pf_handler: fault_addr: 0x600000000000, flt: 3

====== TRAP FRAME DUMP ======
sp:	0x417f00
tpidr:	0x411748
spsr:	0x40000000
esr:	0x9200004f
elr:	0x403738
x0:	0x2710
x1:	0x40ccf1
x2:	0x600000000000
x3:	0x600000002710
x4:	0x40ccf1
x29:	0x417f00
x30:	0x403718
====== DUMP END ======
```

### mmap_regionがaddr順に並ぶように修正

```
[F-18] ２つのマッピングの間のアドレスを指定したマッピングのテスト
ret1=0x600000000000
ret2=0x600000003000
ret3=0x600000001000
region[1]: address: 0x600000000000, length: 0x3e8
region[2]: address: 0x600000001000, length: 0x3e8
region[3]: address: 0x600000003000, length: 0xc8
[F-18] ok

[F-19] ２つのマッピングの間に不可能なアドレスを指定した場合
ret1=0x600000000000
ret2=0x600000003000
ret3=0x600000004000
region[1]: address: 0x600000000000, length: 0x3e8
region[2]: address: 0x600000003000, length: 0xc8
region[3]: address: 0x600000004000, length: 0x2710
[F-19]  ok
```

## trap.cでanonymousを考慮するよう修正

- mmaptest2のすべてのテストを通過
- 相変わらずテスト後にlocal echoがオフになる
- バックスペースも効かないので無設定のtermioに戻ってるようだ
