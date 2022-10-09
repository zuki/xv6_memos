# oldmallocを試す

## ビルド

```
$ cd libc
$ rm -rf obj
$ export CROSS_COMPILE=aarch64-linux-gnu-
$ ./configure --target=aarch64 --with-malloc=oldmalloc
$ make
...
/usr/lib/gcc-cross/aarch64-linux-gnu/9/../../../../aarch64-linux-gnu/bin/ld: obj/src/malloc/oldmalloc/malloc.lo: in function `__libc_malloc':
malloc.c:(.text.__libc_malloc+0x0): multiple definition of `__libc_malloc'; obj/src/malloc/lite_malloc.lo:lite_malloc.c:(.text.__libc_malloc+0x0): first defined here
collect2: error: ld returned 1 exit status
make: *** [Makefile:162: lib/libc.so] Error 1

# 下記パッチ適用

$ make
$ ls -l lib
total 3544
-rw-rw-r-- 1 vagrant vagrant    1760 Sep 15 09:43 crt1.o
-rw-rw-r-- 1 vagrant vagrant    1072 Sep 15 09:43 crti.o
-rw-rw-r-- 1 vagrant vagrant    1016 Sep 15 09:43 crtn.o
-rw-rw-r-- 1 vagrant vagrant 2740306 Sep 15 09:48 libc.a
-rw-rw-r-- 1 vagrant vagrant       8 Aug  4 11:07 libcrypt.a
-rwxrwxr-x 1 vagrant vagrant  824640 Sep 15 09:48 libc.so
-rw-rw-r-- 1 vagrant vagrant       8 Aug  4 11:07 libdl.a
-rw-rw-r-- 1 vagrant vagrant       8 Aug  4 11:07 libm.a
-rw-rw-r-- 1 vagrant vagrant       8 Aug  4 11:07 libpthread.a
-rw-rw-r-- 1 vagrant vagrant       8 Aug  4 11:07 libresolv.a
-rw-rw-r-- 1 vagrant vagrant       8 Aug  4 11:07 librt.a
-rw-rw-r-- 1 vagrant vagrant       8 Aug  4 11:07 libutil.a
-rw-rw-r-- 1 vagrant vagrant       8 Aug  4 11:07 libxnet.a
-rw-rw-r-- 1 vagrant vagrant     622 Sep 15 09:48 musl-gcc.specs
-rw-rw-r-- 1 vagrant vagrant    2688 Sep 15 09:43 rcrt1.o
-rw-rw-r-- 1 vagrant vagrant    1760 Sep 15 09:43 Scrt1.o

$ cd ../coreutils-8.32
$ make clean && make
$ cd ..
$ rm -rf obj && make && make qemu
```

## パッチ

[musl commit](https://git.musl-libc.org/cgit/musl/commit/?id=98b9df994c85dcb6a8a5a9099495dd44c7cf2bce)

```diff
diff --git a/src/malloc/oldmalloc/malloc.c b/src/malloc/oldmalloc/malloc.c
index 53f5f959..a5cbdb68 100644
--- a/src/malloc/oldmalloc/malloc.c
+++ b/src/malloc/oldmalloc/malloc.c
@@ -11,7 +11,7 @@
 #include "malloc_impl.h"
 #include "fork_impl.h"

-#define malloc __libc_malloc
+#define malloc __libc_malloc_impl
 #define realloc __libc_realloc
 #define free __libc_free
 ```

# 結果

- デグレッションなし
- cpも正常

```
$ cat > test
aa
aa
bb
cc
cc
$ cp test test2
$ cat test2
aa
aa
bb
cc
cc
$ uniq test
aa
bb
cc
$ uniq -c test
      2 aa
      1 bb
      2 cc
$ echo abc
abc
$ uname -a
xv6  1.0.1 2021-09-15 (oldmalloc) AArch64 GNU/Linux
$ wc test
unexpected interrupt 0 at cpu 0
Synchronous: Unknown:
  ESR_EL1 0x0 ELR_EL1 0x419c40
 SPSR_EL1 0x20000000 FAR_EL1 0x0
	irq_error: irq of type 8 unimplemented.
$ mv test test2
sys_fstat: fd=-100, path=test2, flags=0
sys_fstat: fd=-100, path=test, flags=256
sys_fstatat: flags unimplemented
mv: cannot stat 'test'Total vmas: 0
pagefault_handler: Segmentation Fault 2: 430ff0
- elr: 0x41f4e0, far: 0x430ff0
pagefault handler
$ head -3 test
aa
aa
bb
$ tail -3 test
$ tail test
$ rm test
sys_fstatat: flags unimplemented
```

## `sys_reanameat2`を実装

```
$ echo abc > test
$ ls
console        0 24 0
test           8000 25 4
$ mv test test2
$ ls
console        0 24 0
test2          8000 25 4
$ cat test2
abc
$ rm test2
$ ls
console        0 24 0
$
```

# coreutilsの`ls`と`cat`を使用してみる

- `SYS_getdents64`を実装した。
- ただし、`SYS_getdents64`が想定するディレクトリエントリの組織化方式がxv6とは異なり、
  正確な対応は取れなかった。

```
$ ls
cat	 echo	       init   mkfs	 mv	rm     sort  tee    uniq
console  getdentstest  ls     mmaptest	 mycat	rmdir  sync  touch  wc
cp	 head	       mkdir  mmaptest2  myls	sh     tail  uname
$ myls
.              4000 1 4096
..             4000 1 4096
mmaptest2      8000 2 84920
mkfs           8000 3 47800
mycat          8000 4 40536
init           8000 5 25384
mmaptest       8000 6 58120
sh             8000 7 59264
myls           8000 8 46440
getdentstest   8000 9 41096
cp             8000 10 227168
echo           8000 11 101704
head           8000 12 126512
mkdir          8000 13 129984
mv             8000 14 237608
rm             8000 15 162064
rmdir          8000 16 112576
sort           8000 17 260160
sync           8000 18 111592
tail           8000 19 180216
tee            8000 20 118392
touch          8000 21 188248
uname          8000 22 111464
uniq           8000 23 129112
wc             8000 24 138216
cat            8000 25 117032
ls             8000 26 318088
console        0 27 0
$ echo abc > test
$ mycat test
abc
$ cat test
abc
$ ls -l
sys_openat: cant namei /etc/passwd
sys_openat: cant namei /etc/group
sys_openat: cant namei /etc/passwd
total 0
Total vmas: 0
pagefault_handler: Segmentation Fault 2: 442fe0
- elr: 0x407dfc, far: 0x442fe0
pagefault handler
```

- 前項2は理解不足でxv6でも対応できることがわかり、`SYS_getdents64`の実装を修正

```
$ ls
cat	 echo	       init   mkfs	 mv	rm     sort  tee    uniq
console  getdentstest  ls     mmaptest	 mycat	rmdir  sync  touch  wc
cp	 head	       mkdir  mmaptest2  myls	sh     tail  uname
```

# `ls -l`のpagefaultを修正

- `vm.c#handle_pagefualt()`を修正
- `SYS_clock_gettime()`を実装（常に同じtimeを返す)

```
$ ls
cat	 echo	       init   mkfs	 mv	rm     sort  tee    uniq
console  getdentstest  ls     mmaptest	 mycat	rmdir  sync  touch  wc
cp	 head	       mkdir  mmaptest2  myls	sh     tail  uname
$ ls -l
sys_openat: cant namei /etc/passwd
sys_openat: cant namei /etc/group
sys_openat: cant namei /etc/passwd
total 0
va: 0x442fe0, p: addr: 0xffff0000008d96d8, size: 0x44a000, kstack: 0xffff00003a37d000
sys_openat: cant namei /etc/localtime
?????????? ? ?       ?      ?            ?                // 最初のファイルが表示されない
?--------- 1 4448256 0      0 Feb 18  1970 console
---------- 1 4448256 0 227168 Feb 18  1970 cp
---------- 1 4448256 0 101704 Feb 18  1970 echo
---------- 1 4448256 0  41096 Feb 18  1970 getdentstest
---------- 1 4448256 0 126512 Feb 18  1970 head
---------- 1 4448256 0  25384 Feb 18  1970 init
---------- 1 4448256 0 318088 Feb 18  1970 ls
---------- 1 4448256 0 129984 Feb 18  1970 mkdir
---------- 1 4448256 0  47800 Feb 18  1970 mkfs
---------- 1 4448256 0  58120 Feb 18  1970 mmaptest
---------- 1       0 0  84920 Feb 18  1970 mmaptest2
---------- 1 4448256 0 237608 Feb 18  1970 mv
---------- 1 4448256 0  40536 Feb 18  1970 mycat
---------- 1 4448256 0  46440 Feb 18  1970 myls
---------- 1 4448256 0 162064 Feb 18  1970 rm
---------- 1 4448256 0 112576 Feb 18  1970 rmdir
---------- 1 4448256 0  59264 Feb 18  1970 sh
---------- 1 4448256 0 260160 Feb 18  1970 sort
---------- 1 4448256 0 111592 Feb 18  1970 sync
---------- 1 4448256 0 180216 Feb 18  1970 tail
---------- 1 4448256 0 118392 Feb 18  1970 tee
---------- 1 4448256 0 188248 Feb 18  1970 touch
---------- 1 4448256 0 111464 Feb 18  1970 uname
---------- 1 4448256 0 129112 Feb 18  1970 uniq
---------- 1 4448256 0 138216 Feb 18  1970 wc
$ ls -al
sys_openat: cant namei /etc/passwd
sys_openat: cant namei /etc/group
sys_openat: cant namei /etc/passwd
total 0
va: 0x442fe0, p: addr: 0xffff0000008d96d8, size: 0x44a000, kstack: 0xffff000039b1b000
sys_openat: cant namei /etc/localtime
?????????? ? ?       ?      ?            ?          // 最初のファイルが表示されない
d--------- 1 4448256 0   4096 Feb 18  1970 ..
---------- 1 4448256 0 117032 Feb 18  1970 cat
?--------- 1 4448256 0      0 Feb 18  1970 console
---------- 1 4448256 0 227168 Feb 18  1970 cp
---------- 1 4448256 0 101704 Feb 18  1970 echo
---------- 1 4448256 0  41096 Feb 18  1970 getdentstest
---------- 1 4448256 0 126512 Feb 18  1970 head
---------- 1 4448256 0  25384 Feb 18  1970 init
---------- 1 4448256 0 318088 Feb 18  1970 ls
---------- 1 4448256 0 129984 Feb 18  1970 mkdir
---------- 1 4448256 0  47800 Feb 18  1970 mkfs
---------- 1 4448256 0  58120 Feb 18  1970 mmaptest
---------- 1 4448256 0  84920 Feb 18  1970 mmaptest2
---------- 1 4448256 0 237608 Feb 18  1970 mv
---------- 1 4448256 0  40536 Feb 18  1970 mycat
---------- 1 4448256 0  46440 Feb 18  1970 myls
---------- 1 4448256 0 162064 Feb 18  1970 rm
---------- 1 4448256 0 112576 Feb 18  1970 rmdir
---------- 1 4448256 0  59264 Feb 18  1970 sh
---------- 1 4448256 0 260160 Feb 18  1970 sort
---------- 1 4448256 0 111592 Feb 18  1970 sync
---------- 1 4448256 0 180216 Feb 18  1970 tail
---------- 1 4448256 0 118392 Feb 18  1970 tee
---------- 1 4448256 0 188248 Feb 18  1970 touch
---------- 1 4448256 0 111464 Feb 18  1970 uname
---------- 1 4448256 0 129112 Feb 18  1970 uniq
---------- 1 4448256 0 138216 Feb 18  1970 wc

$ ls -l
sys_openat: cant namei /etc/passwd
sys_openat: cant namei /etc/group
sys_openat: cant namei /etc/passwd
total 0
print_long_format called                  // ソートしたファイル情報毎にprint_long_format()をcall
sys_openat: cant namei /etc/localtime
?????????? ? ?       ?      ?            ?
print_long_format called
?--------- 1 4448256 0      0 Feb 18  1970 console
print_long_format called
---------- 1 4448256 0 227168 Feb 18  1970 cp

$ ls -l
total 0
call plf: cat                               // f->name: cat
print_long_format called: (null)            // f->name: null  呼び出し中に消えた
?????????? ? ?       ?      ?            ?
call plf: console
print_long_format called: console
?--------- 1 4448256 0      0 Feb 18  1970 console
call plf: cp
print_long_format called: cp
---------- 1 4448256 0 227168 Feb 18  1970 cp

call plf: 0x0x446140                        // struct fileinfo * のポインタを出力
print_long_format called: 0x0
?????????? ? ?       ?      ?            ?
call plf: 0x0x446368
print_long_format called: 0x0x446368
?--------- 1 4448256 0      0 Feb 18  1970 console
```

- `coreutils/src/ls.c`を編集

```c
    case long_format:                         // FIXME: for文の中でi==0の場合、引数が0x0で渡る
      set_normal_color ();                    // [0]だけ2回出力する
      print_long_format (sorted_file[0]);
      DIRED_PUTCHAR ('\n');
      for (i = 0; i < cwd_n_used; i++)
        {
          set_normal_color ();
          print_long_format (sorted_file[i]);
          DIRED_PUTCHAR ('\n');
        }
      break;
    }

static void
print_long_format (const struct fileinfo *f)
{
  printf("plf: 0x%p\n", f);
  if (!f) {
      printf("f is null\n");
      return;
  }
```

```
$ ls -l
total 0
plf: 0x446140                                 // printfで出力するとfは設定されているが
f is null                                     // if(!f) ではnull判定されている

plf: 0x0x446140
---------- 1 4448256 0 117032 Feb 19  1970 cat
```

- デバッグ出力を削除

```
$ ls -al
total 0

d--------- 1       0 0   4096 Feb 18  1970 .
d--------- 1 4448256 0   4096 Feb 18  1970 ..
---------- 1 4448256 0 117032 Feb 18  1970 cat
?--------- 1 4448256 0      0 Feb 18  1970 console
---------- 1 4448256 0 227168 Feb 18  1970 cp
```

# `sort`コマンドを動かす

```
$ sort test
syscall1: pid=10, x0=0xe, x1=0x0, x2=0x436ac0, x3=0x8 x4=0x436ac0
Unexpected syscall #134
```

1. `SYS_rt_sigaction`を実装

```
$ sort test
syscall1: pid=10, x0=0xffffffffffffff9c, x1=0x436fe0, x2=0x4, x3=0x200 x4=0x200
Unexpected syscall #439
```

2. `SYS_faccessat2`を実装

```
$ sort test
syscall1: pid=8, x0=0x0, x1=0x80, x2=0x436a88, x3=0x41a7a0 x4=0x3c3c3c3c3c3c3c3c
Unexpected syscall #123
```

3. ` SYS_sched_getaffinity`を実装

```
$ sort test
syscall1: pid=8, x0=0x0, x1=0x2, x2=0x0, x3=0x436c20 x4=0x436ad0
Unexpected syscall #261
```

4. `SYS_prlimit64`を実装

```
$ sort test
syscall1: pid=9, x0=0x4367f0, x1=0x417808, x2=0x0, x3=0x436c20 x4=0x436ad0
Unexpected syscall #179
```

現状、ここまで。最初の方で有効な値を返さないため、次々と新たなシステムコールを要求して
くる。179番が`SYS_sysinfo`で`/proc/sysinto`に必要な情報を提供する必要がある。sortで
なぜそこまで必要かと思うが、おそらくワークメモリや並列処理の可否を判断するのではないかと
思われる。とりあえず、ここでペンディングとした。

```
$ sort test
called sys_rt_sigprocmask, pid=6
called sys_rt_sigprocmask, pid=6
called sys_clone, pid=6
called sys_gettid, pid=9
called sys_rt_sigprocmask, pid=9
called sys_rt_sigprocmask, pid=9
called sys_exec, pid=9
called sys_rt_sigprocmask, pid=6
called sys_rt_sigprocmask, pid=6
called sys_wait4, pid=6
called sys_gettid, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigprocmask, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_rt_sigaction, pid=9
called sys_brk, pid=9
called sys_brk, pid=9
called sys_faccessat2, pid=9
called sys_sched_getaffinity, pid=9
called sys_sched_getaffinity, pid=9
called sys_openat, pid=9
called sys_fcntl, pid=9
called sys_fadvise64, pid=9
called sys_fstat, pid=9
called sys_prlimit64, pid=9
called sys_prlimit64, pid=9
called sys_prlimit64, pid=9
called (null), pid=9
syscall: pid=9, x0=0x4367f0, x1=0x417808, x2=0x0, x3=0x436c20 x4=0x436ad0
Unexpected syscall #179
```

# `wc`コマンドを動かす

```
$ wc test
unexpected interrupt 0 at cpu 0
Synchronous: Unknown:
  ESR_EL1 0x0 ELR_EL1 0x419c40
 SPSR_EL1 0x20000000 FAR_EL1 0x0
	irq_error: irq of type 8 unimplemented.
```

ソースを見ると、並列処理をする気満々のように思える。このエラーの発生箇所はまだ不明。

## スタックオーバーフローだった

- 関数内で大きなバッファを使用していた(16KB)。これらをグローバル変数にしたら動いた。
- スタックを広げる方法を考えた方が健全だろう。

```
$ cat > test
aa
aa
bb
cc
cc
$ wc test
 5  5 15 test
$ wc -l test
5 test
$ wc -w test
5 test
$ wc -c test
15 test
```


# xv6の`wc`コマンドを移植

```
$ mkdir -p user/src/mywc
$ cp [XV-6]/wc.c user/src/mywc/main.c
$ vi user/src/mywc/main.c
$ make && make qemu

$ mywc test
5 5 15 test
```

# `mkdir`, `cd`

- いつのまにか、絶対パス、相対パスによるコマンド実行ができるようになっていた。

```
$ mkdir dir1
sys_mkdirat: ignores mode
$ cd dir1
$ ls ..
execve: 'ls' fail
exec ls failed
$ /ls ..
cat	 echo	       ls	 mmaptest2  mywc   sort  test	   uniq
console  getdentstest  mkdir	 mv	    rm	   sync  timetest  wc
cp	 head	       mkfs	 mycat	    rmdir  tail  touch
dir1	 init	       mmaptest  myls	    sh	   tee	 uname
$ /ls -a .
.  ..
$ /myls ..
.              4000 1 4096
..             4000 1 4096
timetest       8000 2 40728
...
test           8000 30 15
dir1           4000 31 32
$ cd ..
$ ls
cat	 echo	       ls	 mmaptest2  mywc   sort  test	   uniq
console  getdentstest  mkdir	 mv	    rm	   sync  timetest  wc
cp	 head	       mkfs	 mycat	    rmdir  tail  touch
dir1	 init	       mmaptest  myls	    sh	   tee	 uname

$ cd dir1
$ ../ls .
$ ../ls -a .
.  ..
$ ../ls ..
cat	 echo	       ls	 mmaptest2  mywc   sort  test	   uniq
console  getdentstest  mkdir	 mv	    rm	   sync  timetest  wc
cp	 head	       mkfs	 mycat	    rmdir  tail  touch
dir1	 init	       mmaptest  myls	    sh	   tee	 uname
$ ../myls .
.              4000 31 32
..             4000 1 4096
```

# `pwd`

```
$ pwd
syscall1: pid=7, x0=0x418ba0, x1=0x1000, x2=0x0, x3=0x419b90 x4=0x3c687d776e6e717d
Unexpected syscall #17
```

- `sys_getcwd`を実装

```
$ mkdir dir1
sys_mkdirat: ignores mode
$ cd dir1
$ /pwd
sys_pwd: cwd=30
sys_pwd: pdp=1
- buf: /
/
$ /mkdir dir2
sys_mkdirat: ignores mode
$ cd dir2
$ /pwd
sys_pwd: cwd=31
sys_pwd: pdp=30
sys_pwd: pdp=30
...                                 // 無限ループ
$ cd dir1
$ cd dir2
$ /ls -ai
31 .  30 ..
$ cd ..
$ /ls -ai
30 .   1 ..  31 dir2
$ cd ..
$ ls -ai
 1 .	    30 dir1	     27 ls	   15 mv     16 rm     20 tail	 25 wc
 1 ..	    12 echo	     14 mkdir	    4 mycat  17 rmdir  21 tee
26 cat	    10 getdentstest   3 mkfs	    8 myls    7 sh     22 touch
29 console  13 head	      6 mmaptest    9 mywc   18 sort   23 uname
11 cp	     5 init	      2 mmaptest2  28 pwd    19 sync   24 uniq

$ cd dir1
$ /ls -ai
30 .   1 ..  31 dir2
$ /pwd
sys_pwd: cwd=30
dirlookup: dp_inum=30, name=.., type=1
direntlookup: dp_inum=1, dp_type=1, inum=30,
/dir1
$ cd dir2
$ /ls -ai
31 .  30 ..  32 dir21
$ /pwd
sys_pwd: cwd=31
dirlookup: dp_inum=31, name=.., type=1
direntlookup: dp_inum=30, dp_type=1, inum=31,
dirlookup: dp_inum=30, name=.., type=1
direntlookup: dp_inum=1, dp_type=1, inum=30,
- buf: 2rid/1rid/, i: 4, pos: 9
- fub: /dir1/dir2
/dir1/dir2
$ cd dir21
$ /ls -ai
32 .  31 ..
$ /pwd
dirlookup: dp_inum=1, name=pwd, type=1
sys_pwd: cwd=32
dirlookup: dp_inum=32, name=.., type=1
direntlookup: dp_inum=31, dp_type=1, inum=32,
dirlookup: dp_inum=31, name=.., type=1
direntlookup: dp_inum=30, dp_type=1, inum=30,
dirlookup: dp_inum=30, name=.., type=1
direntlookup: dp_inum=1, dp_type=1, inum=30,
- buf: 12rid/./1rid/, i: 4, pos: 12
- fub: /dir1/./dir21
/dir1/./dir21

$ mkdir dir1
sys_mkdirat: ignores mode
$ cd dir1
$ /mkdir dir2
sys_mkdirat: ignores mode
$ /pwd
- cwd: 30, dp: 1
- buf: 1rid/, i: 4, pos: 4
- fub: /dir1
/dir1
$ cd dir2
$ /pwd
- cwd: 31, dp: 30
- cwd: 30, dp: 1
- buf: 2rid/1rid/, i: 4, pos: 9
- fub: /dir1/dir2
/dir1/dir2
$ /mkdir dir3
sys_mkdirat: ignores mode
$ cd dir3
$ /pwd
- cwd: 32, dp: 31
- cwd: 30, dp: 30
- cwd: 30, dp: 1
- buf: 3rid/./1rid/, i: 4, pos: 11
- fub: /dir1/./dir3
/dir1/./dir3
```

- `iput()`, `idup()`を適切に行うことで

```
$ mkdir dir1
sys_mkdirat: ignores mode
$ cd dir1
$ /pwd
- cwd: 30, dp: 1
- buf: 1rid/, i: 4, pos: 4
- fub: /dir1
/dir1
$ /mkdir dir2
sys_mkdirat: ignores mode
$ cd dir2
$ /pwd
- cwd: 31, dp: 30
- cwd: 30, dp: 1
- buf: 2rid/1rid/, i: 4, pos: 9
- fub: /dir1/dir2
/dir1/dir2
$ /mkdir dir3
sys_mkdirat: ignores mode
$ cd dir3
$ /pwd
- cwd: 32, dp: 31
- cwd: 31, dp: 30
- cwd: 30, dp: 1
- buf: 3rid/2rid/1rid/, i: 4, pos: 14
- fub: /dir1/dir2/dir3
/dir1/dir2/dir3
$ cd ..
$ /pwd
- cwd: 31, dp: 30
- cwd: 30, dp: 1
- buf: 2rid/1rid/, i: 4, pos: 9
- fub: /dir1/dir2
/dir1/dir2
$ cd ..
$ /pwd
- cwd: 30, dp: 1
- buf: 1rid/, i: 4, pos: 4
- fub: /dir1
/dir1
$ cd ..
$ /pwd
- cwd: 1, dp: 1
- buf: ./, i: 1, pos: 1
- fub: /.
/.
```

# `rmdir`, `mkdir -p`

- `rmdir`, `rmdir -p`は問題なし
- `mkdir -p`はシステムコールが必要

```
$ ls
cat	 echo	       ls	 mmaptest2  mywc   sh	 tee	wc
console  getdentstest  mkdir	 mv	    pwd    sort  touch
cp	 head	       mkfs	 mycat	    rm	   sync  uname
dir1	 init	       mmaptest  myls	    rmdir  tail  uniq
$ cd dir1/dir2
$ /pwd
/dir1/dir2
$ /ls
dir3
$ /rmdir dir3
$ /ls -a
.  ..
$ cd ../..
$ ls -a dir1/dir2
.  ..
$ rmdir -p dir1/dir2
$ ls
cat	 echo	       init   mkfs	 mv	mywc  rmdir  sync  touch  wc
console  getdentstest  ls     mmaptest	 mycat	pwd   sh     tail  uname
cp	 head	       mkdir  mmaptest2  myls	rm    sort   tee   uniq
$ mkdir -p dir1/dir2
syscall1: pid=23, x0=0x0, x1=0x4180e0, x2=0x4193a8, x3=0x0 x4=0x0
Unexpected syscall #166
kern/console.c:250: kernel panic at cpu 0.
```

- `sys_umask`を実装

```
$ ls
cat	 echo	       init   mkfs	 mv	mywc  rmdir  sync  touch  wc
console  getdentstest  ls     mmaptest	 mycat	pwd   sh     tail  uname
cp	 head	       mkdir  mmaptest2  myls	rm    sort   tee   uniq
$ mkdir -p dir1/dir2
$ ls -a dir1/dir2
.  ..
$ cd dir1
$ /pwd
/dir1
$ cd dir2
$ /pwd
/dir1/dir2
```
