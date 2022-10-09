# Coreutilsをコンパイル

## `configure && make`

- makeでエラー

```
$ ./configure --host=aarch64-linux-gnu CFLAGS="-I../libc/include/ -I../libc/include/sys -I../libc/obj/include/bits" LFLAGS=-L../libc/lib 2>&1 | tee compile.log

$ make 2>&1 | tee compile.log
...
src/ls.c: In function 'print_dir':
src/ls.c:3026:24: error: 'SYS_getdents' undeclared (first use in this function); did you mean 'SYS_getdents64'?
 3026 |           if (syscall (SYS_getdents, dirfd (dirp), NULL, 0) == -1
      |                        ^~~~~~~~~~~~
      |                        SYS_getdents64
```

## `SYS_getdents64`に修正

- makeはできたが、ライブラリがstatic linkされていない

```
$ file src/pwd
pwd: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, BuildID[sha1]=bb4ad46452c0caf78ecda98613cea5452f89c4ff, for GNU/Linux 3.7.0, with debug_info, not stripped
```

## `LDFLAGS`指定でconfigure失敗

- 実はこれまで`LDFLAGS`を`LFLAGS`と間違って指定していた。その場合、`configure`は実行されていたが、これを正すと`configure`で以下のエラーが発生しmakeできなくなった。

```
checking for aarch64-linux-gnu-gcc... aarch64-linux-gnu-gcc
checking whether the C compiler works... no
configure: error: in `/home/vagrant/xv6-fudan/coreutils-8.32':
configure: error: C compiler cannot create executables
```

## `configure`の`LDFLAG`を削除し、`CFLAGS`を`user/Makefile`に合わせた

- testでエラーとなったが、問題はなさそう。
- static linkされたバイナリが作成された

```
$ ./configure --host=aarch64-linux-gnu CFLAGS="-specs /home/vagrant/xv6-fudan/obj/user/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096 -I/home/vagrant/xv6-fudan/libc/obj/include/ -I/home/vagrant/xv6-fudan/libc/arch/aarch64/ -I/home/vagrant/xv6-fudan/libc/arch/generic/"

$ make

make[4]: Entering directory '/home/vagrant/xv6-fudan/coreutils-8.32/gnulib-tests'
  CC       bench-md5.o
In file included from ../lib/md5.h:23,
                 from bench-md5.c:21:
../lib/stdio.h:43:15: fatal error: stdio.h: No such file or directory
   43 | #include_next <stdio.h>
      |               ^~~~~~~~~
compilation terminated.
make[4]: *** [Makefile:6552: bench-md5.o] Error 1

$ file src/pwd
pwd: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, not stripped

$ aarch64-linux-gnu-readelf -e pwd
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x400914
  Start of program headers:          64 (bytes into file)
  Start of section headers:          125280 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         4
  Size of section headers:           64 (bytes)
  Number of section headers:         17
  Section header string table index: 16

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .init             PROGBITS         0000000000400120  00000120
       0000000000000010  0000000000000000  AX       0     0     4
  [ 2] .text             PROGBITS         0000000000400130  00000130
       0000000000012970  0000000000000000  AX       0     0     16
  [ 3] .fini             PROGBITS         0000000000412aa0  00012aa0
       0000000000000010  0000000000000000  AX       0     0     4
  [ 4] .rodata           PROGBITS         0000000000412ab0  00012ab0
       0000000000001d54  0000000000000000   A       0     0     16
  [ 5] .eh_frame         PROGBITS         0000000000414808  00014808
       0000000000001638  0000000000000000   A       0     0     8
  [ 6] .init_array       INIT_ARRAY       0000000000417a60  00016a60
       0000000000000008  0000000000000008  WA       0     0     8
  [ 7] .fini_array       FINI_ARRAY       0000000000417a68  00016a68
       0000000000000008  0000000000000008  WA       0     0     8
  [ 8] .data.rel.ro      PROGBITS         0000000000417a70  00016a70
       0000000000000188  0000000000000000  WA       0     0     16
  [ 9] .got              PROGBITS         0000000000417bf8  00016bf8
       00000000000003e8  0000000000000008  WA       0     0     8
  [10] .got.plt          PROGBITS         0000000000417fe8  00016fe8
       0000000000000018  0000000000000008  WA       0     0     8
  [11] .data             PROGBITS         0000000000418000  00017000
       00000000000002d8  0000000000000000  WA       0     0     8
  [12] .bss              NOBITS           00000000004182e0  000172d8
       0000000000000ea8  0000000000000000  WA       0     0     16
  [13] .comment          PROGBITS         0000000000000000  000172d8
       000000000000002a  0000000000000001  MS       0     0     1
  [14] .symtab           SYMTAB           0000000000000000  00017308
       0000000000005c70  0000000000000018          15   644     8
  [15] .strtab           STRTAB           0000000000000000  0001cf78
       000000000000195c  0000000000000000           0     0     1
  [16] .shstrtab         STRTAB           0000000000000000  0001e8d4
       0000000000000086  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  p (processor specific)

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x0000000000015e40 0x0000000000015e40  R E    0x1000
  LOAD           0x0000000000016a60 0x0000000000417a60 0x0000000000417a60
                 0x0000000000000878 0x0000000000001728  RW     0x1000
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000016a60 0x0000000000417a60 0x0000000000417a60
                 0x00000000000005a0 0x00000000000005a0  R      0x1

 Section to Segment mapping:
  Segment Sections...
   00     .init .text .fini .rodata .eh_frame
   01     .init_array .fini_array .data.rel.ro .got .got.plt .data .bss
   02
   03     .init_array .fini_array .data.rel.ro .got .got.plt
```

## 実行するとエラー

```
$ ls
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46440
sh             8000 6 59264
pwd            8000 7 126368
console        0 8 0
$ pwd
data abort: insruction 0x405e70, fault addr 0x41abc0
kern/console.c:250: kernel panic at cpu 0.
```

- `4096`しかないスタックから`4448`引いている。

```
$ aarch64-linux-gnu-objdump -d -S coreutils-8.32/src/pwd

0000000000405e68 <rpl_getcwd>:
  405e68:       d2822c0c        mov     x12, #0x1160                    // #4448
  405e6c:       cb2c63ff        sub     sp, sp, x12
  405e70:       a9007bfd        stp     x29, x30, [sp]
  405e74:       910003fd        mov     x29, sp
  405e78:       a90153f3        stp     x19, x20, [sp, #16]
  405e7c:       d0000093        adrp    x19, 417000 <__FRAME_END__+0x11c4>
```

## 使いそうなコマンドをインストール

```
$ ls
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46440
sh             8000 6 59264
cp             8000 7 227536
echo           8000 8 102080
head           8000 9 130984
mkdir          8000 10 134440
mv             8000 11 242072
rm             8000 12 162440
rmdir          8000 13 117040
sort           8000 14 264608
sync           8000 15 116056
tail           8000 16 180696
tee            8000 17 122864
touch          8000 18 188632
uname          8000 19 115936
uniq           8000 20 133584
wc             8000 21 142680
console        0 22 0
```

## `echo`コマンドはOK

```
$ echo abc
abc
$ echo test
test
$ echo abc > abc.txt
$ cat abc.txt
abc
$
```

## `head`はsyscallが足りない

```
$ ls > ls.txt
$ head ls.txt
syscall1: pid=10, x0=0x3, x1=0xfffffffffffffe69, x8=0x3e    // 0x3e=62: SYS_lseek
Unexpected syscall #62
kern/console.c:250: kernel panic at cpu 0.
```

## `SYS_lseek`を実装

- `head`はOK
- `tail`はsyscall(#222)が足りない
- `wc`もsyscall(#222)が足りない
- `cp`はsyscall(#175)が足りない

```
$ ls > ls.txt
$ head -3 ls.txt
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
$ tail -3 ls.txt
syscall1: pid=12, x0=0x0, x1=0x2000, x8=0xde      // 222: SYS_mmap
Unexpected syscall #222

$ wc ls.txt
syscall1: pid=8, x0=0x0, x1=0x2000, x8=0xde
Unexpected syscall #222

$ cp ls.txt ls2.txt
syscall1: pid=8, x0=0x41eabc, x1=0x402988, x8=0xaf    // 175: SYS_geteuid
Unexpected syscall #175
```

## `SYS_euid`を実装

```
$ cp ls.txt ls2.txt
syscall1: pid=8, x0=0x0, x1=0x2000, x8=0xde
Unexpected syscall #222

$ mkdir test
sys_mkdirat: mode unimplemented
syscall1: pid=8, x0=0x1, x1=0x3, x8=0x19          // 25: SYS_fcntl
Unexpected syscall #25

$ sort ls.txt
syscall1: pid=7, x0=0xe, x1=0x0, x8=0x86          // 134: SYS_rt_sigaction
Unexpected syscall #134

$ touch test.txt
syscall1: pid=7, x0=0x3, x1=0x0, x8=0x18          // 24: SYS_dup3
Unexpected syscall #24

```

# `SYS_mmap`を実装する

## `mmaptest.c`をMITから`user/src/mmaptest/main.c`にダウンロード

```
$ ls
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46440
mmaptest       8000 6 54024
sh             8000 7 59264
cp             8000 8 227536
echo           8000 9 102080
head           8000 10 130984
mkdir          8000 11 134440
mv             8000 12 242072
rm             8000 13 162440
rmdir          8000 14 117040
sort           8000 15 264608
sync           8000 16 116056
tail           8000 17 180696
tee            8000 18 122864
touch          8000 19 188632
uname          8000 20 115936
uniq           8000 21 133584
wc             8000 22 142680
console        0 23 0
$ mmaptest
mmap_test starting
syscall1: pid=8, x0=0xffffffffffffff9c, x1=0x407108, x8=0x23     // SYS_unlinkat
Unexpected syscall #35
```

## `SYS_unlinkat`を実装

```
$ mmaptest
mmap_test starting
test mmap f
syscall1: pid=7, x0=0x0, x1=0x2000, x8=0xde
Unexpected syscall #222
```

## `SYS_mmap, SYS_munmap`を実装

```
$ mmaptest
mmap_test starting
test mmap f
mismatch at 0, wanted 'A', got 0x0
syscall1: pid=7, x0=0x403810, x1=0x40a000, x8=0xac
Unexpected syscall #172
```

## `SYS_pid, SYS_ppid`を実装

```
$ mmaptest
mmap_test starting
test mmap f
mismatch at 0, wanted 'A', got 0x0
mmaptest: mmap_test failed: v1 mismatch (1), pid=7
uvm_unmap: not mapped
```

## `SYS_pid, SYS_ppid`を修正（返り値の型を`int`に変更）

```
$ mmaptest
mmap_test starting
i=0, written=200
i=1, written=400
i=2, written=600
i=3, written=800
i=4, written=a00
i=5, written=c00
i=6, written=e00
i=7, written=1000
i=8, written=1200
i=9, written=1400
i=10, written=1600
i=11, written=1800
makefeile ok
test mmap f
mmap p=0x0x40d000
mismatch at 1, wanted zero, got 0x41, addr=0x40d001
mmaptest: mmap_test failed: v1 mismatch (2), pid=7
uvm_unmap: not mapped
```

## デバッグ出力を追加（ロジックに変更はなし）

```
$ mmaptest
mmap_test starting
i=0, written=200
i=1, written=400
i=2, written=600
i=3, written=800
i=4, written=a00
i=5, written=c00
i=6, written=e00
i=7, written=1000
i=8, written=1200
i=9, written=1400
i=10, written=1600
i=11, written=1800
makefeile ok
test mmap f
mmap p=0x0x40d000
...........................................................................................
~~~~
...............................................,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
~~~
,,
_v1 ok
test mmap f: OK
test mmap private
mismatch at 0, wanted 'A', got 0x0, addr=0x40d000
mmaptest: mmap_test failed: v1 mismatch (1), pid=7
$ ls
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46440
mmaptest       8000 6 54184
sh             8000 7 59264
cp             8000 8 227536
echo           8000 9 102080
head           8000 10 130984
mkdir          8000 11 134440
mv             8000 12 242072
rm             8000 13 162440
rmdir          8000 14 117040
sort           8000 15 264608
sync           8000 16 116056
tail           8000 17 180696
tee            8000 18 122864
touch          8000 19 188632
uname          8000 20 115936
uniq           8000 21 133584
wc             8000 22 142680
console        0 23 0
mmap.dur       8000 24 6144
$
```

## gdb環境で実行

- f, private, read-only, read/write はOK
- dirtyテストが失敗する。
- gdb環境を外すと依然として、privateテストで_v1テストでNG

```
$ mmaptest
mmap_test starting
i=0, written=200
i=1, written=400
i=2, written=600
i=3, written=800
i=4, written=a00
i=5, written=c00
i=6, written=e00
i=7, written=1000
i=8, written=1200
i=9, written=1400
i=10, written=1600
i=11, written=1800
makefeile ok
test mmap f
mmap 1: p=0x0x40d000

_v1 ok
test mmap f: OK
test mmap private
mmap 2: p=0x0x40d000

_v1 ok
test mmap private: OK
test mmap read-only
mmap 3: p=0x0xffffffffffffffff
test mmap read-only: OK
test mmap read/write

_v1 ok
test mmap read/write: OK
test mmap dirty
mmaptest: mmap_test failed: file does not contain modifications, pid=7
$
```
### test mmap f

```
(gdb) p *p
$1 = {sz = 4255744, pgdir = 0xffff00003a39d000,
  kstack = 0xffff00003b3fc000 '\021' <repeats 200 times>..., state = RUNNING,
  pid = 7, parent = 0xffff0000008d1458 <ptable+4984>, tf = 0xffff00003b3fced0,
  context = 0xffff00003b3fcc50, chan = 0x0, killed = 0,
  name = "mmaptest\000\000\000\000\000\000\000", vmas = {{addr = 4247552,
      length = 8192, prot = 1, flags = 2, fd = 3, offset = 0, valid = 0,
      file = 0xffff0000008cef48 <ftable+64>}, {addr = 0, length = 0, prot = 0,
      flags = 0, fd = 0, offset = 0, valid = -1,
      file = 0x0} <repeats 15 times>}, ofile = {
    0xffff0000008cef20 <ftable+24>, 0xffff0000008cef20 <ftable+24>,
    0xffff0000008cef20 <ftable+24>, 0xffff0000008cef48 <ftable+64>,
    0x0 <repeats 12 times>}, cwd = 0xffff0000008c0548 <icache+24>}
```

### test mmap private

```
$2 = {sz = 4255744, pgdir = 0xffff00003a39d000,
  kstack = 0xffff00003b3fc000 '\021' <repeats 200 times>..., state = RUNNING,
  pid = 7, parent = 0xffff0000008d1458 <ptable+4984>, tf = 0xffff00003b3fced0,
  context = 0xffff00003b3fcd70, chan = 0x0, killed = 0,
  name = "mmaptest\000\000\000\000\000\000\000", vmas = {{addr = 4247552,
      length = 8192, prot = 3, flags = 2, fd = 3, offset = 0, valid = 0,
      file = 0xffff0000008cef48 <ftable+64>}, {addr = 0, length = 0, prot = 0,
      flags = 0, fd = 0, offset = 0, valid = -1,
      file = 0x0} <repeats 15 times>}, ofile = {
    0xffff0000008cef20 <ftable+24>, 0xffff0000008cef20 <ftable+24>,
    0xffff0000008cef20 <ftable+24>, 0xffff0000008cef48 <ftable+64>,
    0x0 <repeats 12 times>}, cwd = 0xffff0000008c0548 <icache+24>}
```

### test mmap read-only

```
(gdb) p *p
$3 = {sz = 4259840, pgdir = 0xffff00003a39d000,
  kstack = 0xffff00003b3fc000 '\021' <repeats 200 times>..., state = RUNNING,
  pid = 7, parent = 0xffff0000008d1458 <ptable+4984>, tf = 0xffff00003b3fced0,
  context = 0xffff00003b3fcd70, chan = 0x0, killed = 0,
  name = "mmaptest\000\000\000\000\000\000\000", vmas = {{addr = 4247552,
      length = 12288, prot = 3, flags = 1, fd = 3, offset = 0, valid = 0,
      file = 0xffff0000008cef48 <ftable+64>}, {addr = 0, length = 0, prot = 0,
      flags = 0, fd = 0, offset = 0, valid = -1,
      file = 0x0} <repeats 15 times>}, ofile = {
    0xffff0000008cef20 <ftable+24>, 0xffff0000008cef20 <ftable+24>,
    0xffff0000008cef20 <ftable+24>, 0xffff0000008cef48 <ftable+64>,
    0x0 <repeats 12 times>}, cwd = 0xffff0000008c0548 <icache+24>}
```

## `mmap`, `wait`, `exit`を修正して`mmaptest`クリア

```
$ mmaptest
mmap_test starting
- open mmap.dur (3) RDONLAY
[1] test mmap f
- mmap 3 -> 2PAGE PROT_READ MAP_PRIVATE: addr=0x0x40e000
- p[0]=A
- p[4096]=A
- p[6143]=A
- p[6144]=
- _v1 ok
- munmmap 2PAGE: addr=0x0x40e000
test mmap f: OK
[2] test mmap private
- mmap 3 -> 2PAGE PROT_READ/WRITE MAP_PRIVATE: addr=0x0x40e000
- close 3
- p[0]=A
- p[4096]=A
- p[6143]=A
- p[6144]=
- _v1 ok
- write 2PAGE = Z
- p[1]=Z
- munmmap 2PAGE: addr=0x0x40e000
test mmap private: OK
[3] test mmap read-only
- open mmap.dur (3) RDONALY
- mmap 3 -> 3PAGE PROT_READ/WRITE MAP_SHARED: addr=0x0xffffffffffffffff
- close 3
test mmap read-only: OK
[4] test mmap read/write
- open mmap.dur (3) RDWR
- mmap 3 -> 3PAGE PROT_READ|WRITE MAP_SHARED: addr=0x0x40e000
- close 3
- p[0]=A
- p[4096]=A
- p[6143]=A
- p[6144]=
- _v1 ok
- print 2PAGE = Z
- munmmap 2PAGE: addr=0x0x40e000
test mmap read/write: OK
[5] test mmap dirty
- open mmap.dur (3) RDWR
- close 3
test mmap dirty: OK
[6] test not-mapped unmap
- munmmap PAGE: addr=0x0x410000
test not-mapped unmap: OK
[7] test mmap two files
- open mmap1 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x0x40e000
- close 3
- unlink mmap1
- open mmap2 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x0x40f000
- close 3
- unlink mmap2
- munmap PAGE: addr=0x0x40e000
- munmap PAGE: addr=0x0x40f000
test mmap two files: OK
mmap_test: ALL OK
fork_test starting
- p[0]=A
- p[4096]=A
- p[6143]=A
- p[6144]=
- _v1 ok
- p[0]=A
- p[6143]=A
- p[6144]=
- _v1 ok
- p[0]=A
- p[6143]=A
- p[6144]=
- _v1 ok
fork_test OK
mmaptest: all tests succeeded
```

## `tail`コマンド

```
$ ls > ls.txt
$ head -3 ls.txt
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
$ tail -3 ls.txt
syscall1: pid=11, x0=0x1, x1=0x3, x8=0x19
Unexpected syscall #25
```

# 実機でも動いた

```
$ mmaptest
mmap_test starting
- open mmap.dur (3) RDONLAY
[1] test mmap f
- mmap 3 -> 2PAGE PROT_READ MAP_PRIVATE: addr=0x0x40e000
- p[0]=A
- p[4096]=A
- p[6143]=A
- p[6144]=
- _v1 ok
- munmmap 2PAGE: addr=0x0x40e000
test mmap f: OK
[2] test mmap private
- mmap 3 -> 2PAGE PROT_READ/WRITE MAP_PRIVATE: addr=0x0x40e000
- close 3
- p[0]=A
- p[4096]=A
- p[6143]=A
- p[6144]=
- _v1 ok
- write 2PAGE = Z
- p[1]=Z
- munmmap 2PAGE: addr=0x0x40e000
test mmap private: OK
[3] test mmap read-only
- open mmap.dur (3) RDONALY
- mmap 3 -> 3PAGE PROT_READ/WRITE MAP_SHARED: addr=0x0xffffffffffffffff
- close 3
test mmap read-only: OK
[4] test mmap read/write
- open mmap.dur (3) RDWR
- mmap 3 -> 3PAGE PROT_READ|WRITE MAP_SHARED: addr=0x0x40e000
- close 3
- p[0]=A
- p[4096]=A
- p[6143]=A
- p[6144]=
- _v1 ok
- print 2PAGE = Z
- munmmap 2PAGE: addr=0x0x40e000
test mmap read/write: OK
[5] test mmap dirty
- open mmap.dur (3) RDWR
- close 3
test mmap dirty: OK
[6] test not-mapped unmap
- munmmap PAGE: addr=0x0x410000
test not-mapped unmap: OK
[7] test mmap two files
- open mmap1 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x0x40e000
- close 3
- unlink mmap1
- open mmap2 (3) RDWR/CREAT
- mmap 3 -> PAGE PROT_READ MAP_PRIVATE: addr=0x0x40f000
- close 3
- unlink mmap2
- munmap PAGE: addr=0x0x40e000
- munmap PAGE: addr=0x0x40f000
test mmap two files: OK
mmap_test: ALL OK
fork_test starting
- p[0]=A
- p[4096]=A
- p[6143]=A
- p[6144]=
- _v1 ok
- p[0]=A
- p[6143]=A
- p[6144]=
- _v1 ok
- p[0]=A
- p[6143]=A
- p[6144]=
- _v1 ok
fork_test OK
mmaptest: all tests succeeded
$ ls
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46440
mmaptest       8000 6 54024
sh             8000 7 59264
cp             8000 8 227536
echo           8000 9 102080
head           8000 10 130984
mkdir          8000 11 134440
mv             8000 12 242072
rm             8000 13 162440
rmdir          8000 14 117040
sort           8000 15 264608
sync           8000 16 116056
tail           8000 17 180696
tee            8000 18 122864
touch          8000 19 188632
uname          8000 20 115936
uniq           8000 21 133584
wc             8000 22 142680
console        0 23 0
ls.txt         8000 24 717
$ cat ls.txt
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46440
mmaptest       8000 6 54024
sh             8000 7 59264
cp             8000 8 227536
echo           8000 9 102080
head           8000 10 130984
mkdir          8000 11 134440
mv             8000 12 242072
rm             8000 13 162440
rmdir          8000 14 117040
sort           8000 15 264608
sync           8000 16 116056
tail           8000 17 180696
tee            8000 18 122864
touch          8000 19 188632
uname          8000 20 115936
uniq           8000 21 133584
wc             8000 22 142680
console        0 23 0
ls.txt         8000 24 0
$ head ls.txt
.              4000 1 4096
..             4000 1 4096
mkfs           8000 2 47800
init           8000 3 25384
cat            8000 4 40536
ls             8000 5 46440
mmaptest       8000 6 54024
sh             8000 7 59264
cp             8000 8 227536
echo           8000 9 102080
$ echo abc
abc
$ wc ls.txt
syscall1: pid=12, x0=0x1, x1=0x3, x8=0x19
Unexpected syscall #25
```

## `dup`, `dup3`, `pip2`, `fcntl`を実装

```
$ ls > ls.txt
$ tail -3 ls.txt
tail: memory exhausted
$ cp ls.txt ls2.txt
cp: memory exhausted
$ mkdir test
sys_mkdirat: mode unimplemented
mkdir: cannot create directory ‘test’: Operation not permitted
$ uniq ls.txt
uniq: ls.txt: Out of memory
$ mv ls.txt ls2.txt
syscall1: pid=11, x0=0xffffff9c, x1=0x432fe0, x8=0x114
Unexpected syscall #276
```

# `struct file`に`flags`変数を追加して、`fnctl`の`F_GETFL`, `F_SETFL`コマンドを実装

## シェルの sysout, syserr が表示されない

-  sys_dupが呼ばれない

```
execve: '/init' start                                                 # if (open("console", O_RDWR) < 0) {
sys_openat: dirfd=-100, path:=console, flags=0x20002, mode=0x0          # 0400002 = O_LARGEFILE | O_RDWR
mknodat: path 'console', major:minor 1:1                              # mknode("console", 1, ,1);
sys_openat: dirfd=-100, path:=console, flags=0x20002, mode=0x0        # open("console", O_RDWR)
execve: 'sh' start
sys_openat: dirfd=-100, path:=console, flags=0x20002, mode=0x0        # while ((fd = open("console", O_RDWR)) >= 0) {
sys_openat: dirfd=-100, path:=console, flags=0x20002, mode=0x0        # fd = 1
sys_openat: dirfd=-100, path:=console, flags=0x20002, mode=0x0        # fd = 2
ls > ls.txt                                                           # $は表示されないが、コマンド受け取る
sys_openat: dirfd=-100, path:=ls.txt, flags=0x20041, mode=0x401c60    # 0400101 = O_LARGEFILE | O_CREAT | O_WRONLY
execve: 'ls' start                                                    # ただし、結果も表示されない
sys_openat: dirfd=-100, path:=., flags=0x20000, mode=0x0
cat ls.txt
execve: 'cat' start
sys_openat: dirfd=-100, path:=ls.txt, flags=0x20000, mode=0x0
```

### `syscall.c`にバグ発見

- sys_dupが呼ばれるようになった

```
execve: '/init' start
called sysno=96     # SYS_set_tid_address
called sysno=56     # SYS_openat
sys_openat: dirfd=-100, path:=console, flags=0x20002, mode=0x0
called sysno=33     # SYS_mknodat
mknodat: path 'console', major:minor 1:1
called sysno=56     # SYS_openat
sys_openat: dirfd=-100, path:=console, flags=0x20002, mode=0x0
sys_openat: fd=0, readable=1, writable=0, flags=0x20002, off=0        # writableになっていない
sys_openat: ok fd=0
called sysno=23     # SYS_dup
sys_dup: fd=1
called sysno=23     # SYS_dup
sys_dup: fd=2
called sysno=29     # SYS_ioctl
sys_ioctl: fd=1, req=0x5413, rem=0x406ed8
called sysno=66     # SYS_writev
called sysno=135    # SYS_rt_sigprocmask
called sysno=135    # SYS_rt_sigprocmask
called sysno=220    # SYS_clone
called sysno=178    # SYS_gettid
called sysno=135    # SYS_rt_sigprocmask
called sysno=135    # SYS_rt_sigprocmask
called sysno=221    # SYS_execve
execve: 'sh' start
called sysno=135    # SYS_rt_sigprocmask
called sysno=135    # SYS_rt_sigprocmask
called sysno=260    # SYS_wait4
called sysno=96     # SYS_set_tid_address
called sysno=56     # SYS_openat
sys_openat: dirfd=-100, path:=console, flags=0x20002, mode=0x0
sys_openat: fd=3, readable=1, writable=0, flags=0x20002, off=0
sys_openat: ok fd=3
called sysno=57     # SYS_close
called sysno=66     # SYS_writev
called sysno=63     # SYS_read               # ここで $ が出るはず
```

## syserrが出力されない原因が判明

- `readable`, `writable`を判定するマクロにflagsを設定していないfile変数を渡していた。

ただし、tail, cpなどで`memory exhausted`になる件は依然として未解決

```
$ tail -v ls.txt
sys_brk: n=0, addr=4354048
sys_brk: n=4362240, addr=4354048
tail: memory exhausted
$ tail -3 ls.txt                   # sys_brk()がaddrを返した場合（これまで）
sys_brk: n=0, addr=4354048
sys_brk: n=4362240, addr=4354048
tail: memory exhausted
$ tail -3 ls.txt                   # sys_brk()がnを返した場合（cyanurusに合わせた場合）
sys_brk: n=0, addr=4354048
sys_brk: n=8192, addr=4354048
tail: memory exhausted
$ wc ls.txt
sys_brk: n=0, addr=4321280
sys_brk: n=4329472, addr=4321280
wc: memory exhausted
$ uniq ls.txt
sys_brk: n=0, addr=4313088
sys_brk: n=4321280, addr=4313088
uniq: ls.txt: Out of memory
$ cp ls.txt ls2.txt
sys_brk: n=0, addr=4390912
sys_brk: n=4399104, addr=4390912
cp: memory exhausted
$ mv ls.txt ls2.txt
syscall1: pid=17, x0=0xffffff9c, x1=0x432fe0, x8=0x114
Unexpected syscall #276                                     # SYS_renameat2
$ touch test.txt
syscall1: pid=12, x0=0x0, x1=0x0, x8=0x58
Unexpected syscall #88                                      # SYS_utimensat
$ rm test.txt
sys_brk: n=0, addr=4337664
sys_brk: n=4345856, addr=4337664
rm: memory exhausted
$ echo abc | tee test3.txt                                  # だんまり
$ uname
syscall1: pid=9, x0=0x419df0, x1=0x40a074, x8=0xa0
Unexpected syscall #160                                     # SYS_uname
```

## `sys_utimenst`を実装

- まだファイル構造体にtime変数はもっていないので、ただ0を返す関数を作成

```
$ ls
wc             8000 22 142680
console        0 23 0
$ touch test.txt                   # 空のファイルを作成
$ ls
wc             8000 22 142680
console        0 23 0
test.txt       8000 24 0
$ echo test >> test.txt            # 書き込み
$ cat test.txt
test
$ ls
test.txt       8000 24 5           # 書き込みOK
$ touch test.txt                   # 既存のファイルをtouchしても問題なし
$
```

## `sys_uname`を実装

```
$ uname --help
Usage: uname [OPTION]...
Print certain system information.  With no OPTION, same as -s.

  -a, --all                print all information, in the following order,
                             except omit -p and -i if unknown:
  -s, --kernel-name        print the kernel name
  -n, --nodename           print the network node hostname
  -r, --kernel-release     print the kernel release
  -v, --kernel-version     print the kernel version
  -m, --machine            print the machine hardware name
  -p, --processor          print the processor type (non-portable)
  -i, --hardware-platform  print the hardware platform (non-portable)
  -o, --operating-system   print the operating system
      --help     display this help and exit
      --version  output version information and exit

GNU coreutils online help: <https://www.gnu.org/software/coreutils/>
Report any translation bugs to <https://translationproject.org/team/>
Full documentation <https://www.gnu.org/software/coreutils/uname>
or available locally via: info '(coreutils) uname invocation'
$ uname -a
xv6  1.0 2021-09-03 AArch64 GNU/Linux
$ uname -s
xv6
$ uname -n
                    # nodename: gethostnameで設定する
$ uname -r
1.0
$ uname -v
2021-09-03
$ uname -m
AArch64
$ uname -i
unknown             # UNAME_HARDWARE_PLATFORM or (HAVE_SYSINFO && defined SI_PLATFORM)
$ uname -o
GNU/Linux           # HOST_OPERATING_SYSTEM: coreutilsコンパイル時のホスト名が設定されている
$ uname -p
unknown
```

## (1) `libc/src/malloc/maalocng/malloc.c`

- 以下のmmap呼び出しでfd=-1となっているのが未対応でエラー。

```
if (!ctx.avail_meta_area_count) {
     size_t n = 2UL << ctx.meta_alloc_shift;
     p = mmap(0, n*pagesize, PROT_NONE,
          MAP_PRIVATE|MAP_ANON, -1, 0);
     if (p==MAP_FAILED) return 0;
```

## (2) `src/malloc/mallocng/malloc.c`にバグあり

```diff
$ git diff
diff --git a/src/malloc/mallocng/malloc.c b/src/malloc/mallocng/malloc.c
index d695ab8e..bce9b573 100644
--- a/src/malloc/mallocng/malloc.c
+++ b/src/malloc/mallocng/malloc.c
@@ -180,6 +180,7 @@ static struct meta *alloc_group(int sc, size_t req)
        if (!m) return 0;
        size_t usage = ctx.usage_by_class[sc];
        size_t pagesize = PGSZ;
+       if (!pagesize) pagesize = 4096;     // added
        int active_idx;
        if (sc < 9) {
                while (i<2 && 4*small_cnt_tab[sc][i] > usage)
```

```
$ tail -3 ls.txt
xnmalloc7: 96
malloc: n=96
aclloc_group: sc=6, reg=96
sys_brk: oldaddr=0x428000, n=0x0, newaddr=0x428000
mallocng: brk=4358144
sys_brk: oldaddr=0x428000, n=0x42a000, newaddr=0x42a000
sys_mmap: addr=0x428000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0
mallocng: mmap
alloc_meta ok
alloc_group: alloc_meta m=0x429018
- usage=0, ps=4096, i=2, cnt=4
1aclloc_group: sc=15, reg=460
alloc_meta ok
alloc_group: alloc_meta m=0x429040
- usage=0, ps=40961aclloc_group: sc=19, reg=1004
alloc_meta ok
alloc_group: alloc_meta m=0x429068
- usage=0, ps=40961aclloc_group: sc=23, reg=2028
alloc_meta ok
alloc_group: alloc_meta m=0x429090
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
- usage=0, ps=40961- mmap3 ret=0x42b000
pagefault_handler: va=0x42b008
call map_region: va=0x42b008, size=0x1000, pa=0x3afd4000, perm=0x60000000000041, ret=0
2 malloc_group ok
2 malloc_group ok
2 malloc_group ok
2 malloc_group ok
malloc: sucess p=0x42b040
1abcxmalloc1: 1048
malloc: n=1048
aclloc_group: sc=21, reg=1048
alloc_meta ok
alloc_group: alloc_meta m=0x4290b8
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
- usage=0, ps=40961- mmap3 ret=0x42c000
2 malloc_group ok
malloc: sucess p=0x42c030
xmalloc2: 1048
malloc: n=1048
malloc: sucess p=0x42c570
dghi: ok=1                                        # エラーはなくなったが肝心のtailの表示がない
$ cp ls.txt ls2.txt
cp: failed to access 'ls2.txt': Operation not permitted
$ rm ls.txt
pagefault_handler: va=0x428130
call map_region: va=0x428130, size=0x1000, pa=0x36e40000, perm=0x60000000000041, ret=0
data abort: insruction 0x40f8e0, fault addr 0xfffffffffffffffd
```

# `memory exhausted`はなくなった

- tailの結果が表示されない
- cpは別のエラー
- rmも別のエラー
- wcとuniq, sortは別のsyscallが必要

```
$ tail -3 ls.txt
sys_brk: oldaddr=0x427000, n=0x0, newaddr=0x427000
sys_brk: oldaddr=0x427000, n=0x429000, newaddr=0x429000
sys_mmap: addr=0x427000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0     # L72
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0          # L250
pagefault_handler: va=0x42a008
call map_region: va=0x42a008, size=0x1000, pa=0x3afd4000, perm=0x60000000000041, ret=0
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0          # L250
$ cp ls.txt ls2.txt
sys_brk: oldaddr=0x430000, n=0x0, newaddr=0x430000
sys_brk: oldaddr=0x430000, n=0x432000, newaddr=0x432000
sys_mmap: addr=0x430000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0     #
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
pagefault_handler: va=0x433008
call map_region: va=0x433008, size=0x1000, pa=0x3afd4000, perm=0x60000000000041, ret=0
sys_mmap: addr=0x0, length=8192, prot=0x3, flags=0x22, fd=-1, offset=0
cp: failed to access 'ls2.txt': Operation not permitted
$ rm ls.txt
sys_brk: oldaddr=0x423000, n=0x0, newaddr=0x423000
sys_brk: oldaddr=0x423000, n=0x425000, newaddr=0x425000
sys_mmap: addr=0x423000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
pagefault_handler: va=0x426008
call map_region: va=0x426008, size=0x1000, pa=0x3afd5000, perm=0x60000000000041, ret=0
sys_mmap: addr=0x0, length=16384, prot=0x3, flags=0x22, fd=-1, offset=0
pagefault_handler: va=0x428130
call map_region: va=0x428130, size=0x1000, pa=0x386c9000, perm=0x60000000000041, ret=0
data abort: insruction 0x40f8e0, fault addr 0xfffffffffffffffd
kern/console.c:250: kernel panic at cpu 0.
$ wc ls.txt
sys_brk: oldaddr=0x41f000, n=0x0, newaddr=0x41f000
sys_brk: oldaddr=0x41f000, n=0x421000, newaddr=0x421000
sys_mmap: addr=0x41f000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
pagefault_handler: va=0x422008
call map_region: va=0x422008, size=0x1000, pa=0x3afd4000, perm=0x60000000000041, ret=0
syscall1: pid=8, x0=0x3, x1=0x0, x8=0xdf
Unexpected syscall #223
$ uniq ls.txt
sys_brk: oldaddr=0x41d000, n=0x0, newaddr=0x41d000
sys_brk: oldaddr=0x41d000, n=0x41f000, newaddr=0x41f000
sys_mmap: addr=0x41d000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
pagefault_handler: va=0x420008
call map_region: va=0x420008, size=0x1000, pa=0x3afd4000, perm=0x60000000000041, ret=0
syscall1: pid=8, x0=0x0, x1=0x0, x8=0xdf
Unexpected syscall #223
$ sort ls.txt
syscall1: pid=7, x0=0xe, x1=0x0, x8=0x86
Unexpected syscall #134
```

![mmap(2)](https://linuxjm.osdn.jp/html/LDP_man-pages/man2/mmap.2.html)

- `src/malloc/mallocng/malloc.c`におけるmmap()の使用箇所

```
72:  if (need_guard) mmap((void *)ctx.brk, pagesize, PROT_NONE, MAP_ANON|MAP_PRIVATE|MAP_FIXED, -1, 0);
82:  p = mmap(0, n*pagesize, PROT_NONE, MAP_PRIVATE|MAP_ANON, -1, 0);
250: p = mmap(0, needed, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON, -1, 0);
311: void *p = mmap(0, needed, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON, -1, 0);
```

## sys_mmapを修正

```
$ tail -3 ls.txt
sys_brk: oldaddr=0x427000, n=0x0, newaddr=0x427000
sys_brk: oldaddr=0x427000, n=0x429000, newaddr=0x429000
sys_mmap: addr=0x427000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0
- ret=0x427000
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
- ret=0x42a000
pagefault_handler: va=0x42a008
call map_region: va=0x42a008, size=0x1000, pa=0x3afd4000, perm=0x60000000000041, ret=0
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
- ret=0x42b000
sys_munmap: addr=0x42b000, len=4096
```

## `sys_fadvise64`を実装

```
$ wc ls.txt
sys_brk: oldaddr=0x41f000, n=0x0, newaddr=0x41f000
sys_brk: oldaddr=0x41f000, n=0x421000, newaddr=0x421000
sys_mmap: addr=0x41f000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0
- ret=0x41f000
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
- ret=0x422000
pagefault_handler: va=0x422008
call map_region: va=0x422008, size=0x1000, pa=0x3afd4000, perm=0x60000000000041, ret=0
sys_fadvise64: fd=3, off=0, len=0, advise=0x2
data abort: insruction 0x401264, fault addr 0x2020202020202069

$ cat test.txt
aa
bb
aa
cc
$ uniq test.txt
sys_brk: oldaddr=0x41d000, n=0x41f000, newaddr=0x41f000
sys_mmap: addr=0x41d000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0
- ret=0x41d000
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
- ret=0x420000
pagefault_handler: va=0x420008     # 0x420008から4Kバイト => 2ページ割り当ててmap
call map_region: va=0x420008, size=0x1000, pa=0x3afd4000, perm=0x60000000000041
map_region: va=0x420008, a=0x420000, last=0x421008
map_region: a=0x420000, *pte=0x0
map_region: a=0x421000, *pte=0x0
- ret=0
sys_munmap: addr=0x420000, len=4096     # 割り当てた最初のページだけunmap
uvmunmap: va=0x420000, length=0x1000, free=1
sys_fadvise64: fd=0, off=0, len=0, advise=0x2
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
- ret=0x420000
pagefault_handler: va=0x420008          # 同じアドレス0x420008から4Kバイト => 2ページ割当
call map_region: va=0x420008, size=0x1000, pa=0x3afd4000, perm=0x60000000000041
map_region: va=0x420008, a=0x420000, last=0x421008
map_region: a=0x420000, *pte=0x0
map_region: a=0x421000, *pte=0x6000003afd5747     # 2ページ目はunmapされていないのでremap
map_region: remap *pte=0x6000003afd5747, va=0x421000
kern/console.c:250: kernel panic at cpu 0.
```

## 当面、remapの場合はpanicとせず、上書きにする。
```
$ uniq test
sys_brk: oldaddr=0x41d000, n=0x0, newaddr=0x41d000
sys_brk: oldaddr=0x41d000, n=0x41f000, newaddr=0x41f000
sys_mmap: addr=0x41d000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0
- ret=0x41d000
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
- ret=0x420000
pagefault_handler: va=0x420008
sys_munmap: addr=0x420000, len=4096
uvmunmap: va=0x420000, length=0x1000, free=1
sys_fadvise64: fd=0, off=0, len=0, advise=0x2
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
- ret=0x420000
pagefault_handler: va=0x420008
map_region: remap *pte=0x6000003afd5747, va=0x421000
aa
bb
cc
sys_munmap: addr=0x420000, len=4096
uvmunmap: va=0x420000, length=0x1000, free=1
uvmunmap: va=0x41d000, length=0x1000, free=1

$ cat > test.txt
aa
aa
bb
cc
cc
$ uniq test.txt
sys_fadvise64: fd=0, off=0, len=0, advise=0x2
aa
bb
cc
$ uniq -c test.txt
sys_fadvise64: fd=0, off=0, len=0, advise=0x2
      2 QEMU: Terminated                          # 2を出力したところでスタック
$ uniq test.txt                                   # 既存のファイルをuniqするとスタック

$ cp test.txt test2
cp: failed to access 'test2': Operation not permitted

$ mkdir test
sys_mkdirat: ignores mode
mkdir: cannot create directory ‘test’: Operation not permitted
```

## `remap`は解決

- `pagefault_handler`でアクセスミスしたアドレスをページアラインしてから`map_region`に
  渡すように変更した。

```
$ uniq test
aa
bb
cc
$ uniq -c test
      2                       # -c オプションでスタックする件は変更なし
$ uniq test
aa
bb
cc                            # 既存ファイルのuniqは問題なし
```

## muslの`mmaloc.c`のバグ修正の方法を変えた

- `arch/aarch64/bits/limits.h`を作成して、ここに`#define PAGESIZE 4096`を設定した
- `src/malloc/mallocng/malloc.c`はもとに戻した

```
$ uniq test
aa
bb
cc
$ uniq -c test           # -cオプションが動くようになった
      2 aa
      1 bb
      2 cc
$ uniq --group test      # -groupオプションもOK
aa
aa

bb

cc
cc

$ tail -3 test           # tail
$ cp test test2          # cp
cp: failed to access 'test2': Operation not permitted
```

- エラーは`coreutils-8.32/src/cp.c`の以下の関数で出力
- fileが存在しないので`sys_fstatat`が`-1`を返していた
- `errno=-1`は`EPERM`なのでこのエラーが出力された
- errnoをきちんと設定するべきか?
- とりあえず`-2=ENOENT`を返すようにすればここはクリアした

```c
static bool
target_directory_operand (char const *file, struct stat *st,
                          bool *new_dst, bool forcing)
{
  int err = (stat (file, st) == 0 ? 0 : errno);
  bool is_a_dir = !err && S_ISDIR (st->st_mode);
  if (err)
    {
      if (err == ENOENT)
        *new_dst = true;
      else if (forcing)
        st->st_mode = 0;  /* clear so we don't enter --backup case below.  */
      else
        die (EXIT_FAILURE, err, _("failed to access %s"), quoteaf (file));
    }
  return is_a_dir;
}
```

```
$ cp ls.txt ls2.txt
tdo: file=ls2.txt, force=0, err=1, errno=1        # errno=1: EPERM
cp: failed to access 'ls2.txt': Operation not permitted

$ cp tst test
called sys_fstatat                                # stat ("test", st)
sys_fstat: fd=-100, path=test, flags=0
not exist path                                    # return ENOENT
called sys_ioctl
called sys_writev
tdo: file=test, force=0, err=2, errno=2
called sys_fstatat
sys_fstat: fd=-100, path=tst, flags=0
called sys_fstatat
sys_fstat: fd=-100, path=test, flags=0
not exist path
called sys_openat
called sys_fstat
called sys_openat
called sys_fstat
called sys_fadvise64
called sys_mmap
- invalid va=0x457040, sz=0x457014, sp=0x42f830  # 今度はこのエラーが繰り返し発生。vm.c

$ cp tst test
sys_mmap: addr=0x430000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0
sys_mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0
sys_mmap: addr=0x0, length=8192, prot=0x3, flags=0x22, fd=-1, offset=0
tdo: file=test, force=0, err=2, errno=2
sys_openat: dirfd=-100, path:=tst, flags=0x20000, mode=0x0
sys_openat: dirfd=-100, path:=test, flags=0x200c1, mode=0x0
sys_mmap: addr=0x0, length=135188, prot=0x3, flags=0x22, fd=-1, offset=0
- invalid va=0x457040, sz=0x457014, sp=0x42f830

$ mkdir dir1                       # mkdirが動いた
sys_mkdirat: ignores mode
$ ls
sys_openat: dirfd=-100, path:=., flags=0x20000, mode=0x0
wc             8000 22 142680
console        0 23 0
test           8000 24 4
test2          8000 25 0
dir1           4000 26 32          # 確かに作成されている
$ cd dir1                          # cdでエラーはでない
$ ls                               # 何故か、lsコマンドが実行できない
execve: 'ls' fail
exec ls failed
$ ls .                             # ディレクトリを指定してもlsコマンドはエラー
execve: 'ls' fail
exec ls failed
$ echo abc > test                  # echo コマンドもエラー
sys_openat: dirfd=-100, path:=test, flags=0x20041, mode=0x401c60
execve: 'echo' fail
exec echo failed
$ cd ..                            # cdはエラーなし
$ ls                               # lsでエラー（このエラーは無限ループ）
sys_openat: dirfd=-100, path:=., flags=0x20000, mode=0x0
- invalid va=0x814443, sz=0x40d000, sp=0x40ca40
```

## `#include <errno.h>`として、xv6をmakeすると`fatal error: bits/errno.h: No such file or directory`

- musl 1.1.13から各アーキテクチャの`bits/`配下のファイルで共通のものが`arch/generic`に
  移動した。`errno.h`もこれに含まれる
- `include/errno.h`が`bits/errno.h`をインクルードしているが、`generic`は見ていないので
  （なにか方法があるかもしれないが不明）で表記のエラーとなっている模様。
- 当面、`libc/obj/include/bits/`に`generic/errno.h`をシンボリックリンクすることにした。
- これで`-errno`を返すようにすれば良いようだ（`sys_fstat`の`-ENOENT`は問題なかった。
- muslを再作成するとこのリンクは消えるので注意。`bits/mman.h`も同じ。こちらは空ファイルに
  している（`bits/mman.h`をインクルードしている`include/sys/mman.h`だけで問題ないと思われるため）

## kernソースでlibcのincludeファイルを使わないように変更

- `elf.h`, `errno.h`, `fcnt.h`, `mman.h`, `syscall.h`, `utsname.h`を`inc`にコピー

## `sys_brk()`の返り値

- `man 2 brk`では成功時は0, 失敗時は-1とあるが、このように設定するとsyscall(226)が要求される
- 注記にlinuxでは成功時には新しいsizeを、失敗時には元のsizeを返す実装になっているとあったので
  そのようにした（成功時はこれまで通りで、失敗時を変えた）。失敗時の挙動はまだ未確認。

```
$ uniq txt
aa
bb
cc
$ uniq --group txt
aa
aa

bb

cc
cc
```

## `sys_mmap()`を大編集

1. [xv6-mmap](https://github.com/chaudharirohit2810/xv6-mmap)の`sys_mmap.c`を採用
2. faultアドレスをページアライン（ROUNDDOWN)して実行

```
$ cp test test2
sys_mmap: addr=0x430000, size=0x1000, prot=0x0, flags=0x32, fd=976739872, offset=0x0
sys_mmap: addr=0x0, size=0x1000, prot=0x3, flags=0x22, fd=976739872, offset=0x0
pagefault_handler: va=0x60000008
sys_mmap: addr=0x0, size=0x2000, prot=0x3, flags=0x22, fd=976739872, offset=0x0
pagefault_handler: va=0x60001008
tdo: file=test2, force=0, err=2, errno=2
sys_mmap: addr=0x0, size=0x21014, prot=0x3, flags=0x22, fd=976739904, offset=0x0
pagefault_handler: va=0x60003000
pagefault_handler: va=0x60024040
Total vmas: 3
Virtual mapping address: 0x60000000	size: 0x1000	shared: 0
Virtual mapping address: 0x60001000	size: 0x2000	shared: 0
Virtual mapping address: 0x60003000	size: 0x21014	shared: 0
pagefault_handler: Segmentation Fault 2: 60024040
pagefault handler

$ cp test test2
sys_mmap: addr=0x430000, size=0x1000, prot=0x0, flags=0x32, fd=976739872, offset=0x0
sys_mmap: addr=0x0, size=0x1000, prot=0x3, flags=0x22, fd=976739872, offset=0x0
pagefault_handler: va=0x60000008
sys_mmap: addr=0x0, size=0x2000, prot=0x3, flags=0x22, fd=976739872, offset=0x0
pagefault_handler: va=0x60001008
pagefault_handler: va=0x60001008
tdo: file=test2, force=0, err=2, errno=2
sys_mmap: addr=0x0, size=0x21014, prot=0x3, flags=0x22, fd=976739904, offset=0x0
pagefault_handler: va=0x60003000
pagefault_handler: va=0x60024040
data abort: insruction 0x418db8, fault addr 0xfffffffffffffffd, dfs=4
```

## MAP＿ANONの場合は、memを割り当てる

これはだめだった。

## `xalloc_die.c`

```c
void
xalloc_die (void)
{
  error (exit_failure, 0, "%s", _("memory exhausted"));     // exit_failure = 0x4230a0
  abort ();
}
```

```
000000000040b708 <xalloc_die>:
  40b708:       a9bf7bfd        stp     x29, x30, [sp, #-16]!
  40b70c:       f00000a4        adrp    x4, 422000 <__FRAME_END__+0xcf4>
  40b710:       f0000083        adrp    x3, 41e000 <version_etc_copyright+0x30> // "memory exhausted"
  40b714:       910003fd        mov     x29, sp   //41c760
  40b718:       f9469480        ldr     x0, [x4, #3368]     // x0 =
  40b71c:       91000063        add     x3, x3, #0x0        // x3 = "memory exhausted"
  40b720:       f9476084        ldr     x4, [x4, #3776]     // x4 = 0x40c1c8
  40b724:       d0000082        adrp    x2, 41d000 <_fini+0xcf0> // x2 = 0x41d000
  40b728:       b9400000        ldr     w0, [x0]            // w0 = [0x4230a0]
  40b72c:       911ec042        add     x2, x2, #0x7b0      // x2 = 0x41d7b0 "%S"
  40b730:       52800001        mov     w1, #0x0            // w1 = 0
  40b734:       d63f0080        blr     x4                  // call error(): 0x40c1c8
  40b738:       f00000a4        adrp    x4, 422000 <__FRAME_END__+0xcf4>
  40b73c:       f9464880        ldr     x0, [x4, #3216]     // call abort(): 0x40f22c
  40b740:       d63f0000        blr     x0
  40b744:       d503201f        nop
```

## `libc/include/fcntl.h`, `libc/arch/aarch64/bits/fcntl.h`

```
#define O_SEARCH   O_PATH                    # 8進表記
#define O_EXEC     O_PATH
#define O_TTY_INIT 0
#define O_ACCMODE (03|O_SEARCH)
#define O_RDONLY  00
#define O_WRONLY  01
#define O_RDWR    02

#define O_CREAT        0100
#define O_EXCL         0200
#define O_NOCTTY       0400
#define O_TRUNC       01000
#define O_APPEND      02000
#define O_NONBLOCK    04000
#define O_DSYNC      010000
#define O_SYNC     04010000
#define O_RSYNC    04010000
#define O_DIRECTORY  040000
#define O_NOFOLLOW  0100000
#define O_CLOEXEC  02000000

#define O_ASYNC      020000
#define O_DIRECT    0200000
#define O_LARGEFILE 0400000
#define O_NOATIME  01000000
#define O_PATH    010000000
#define O_TMPFILE 020040000
#define O_NDELAY O_NONBLOCK

#define F_DUPFD  0
#define F_GETFD  1
#define F_SETFD  2
#define F_GETFL  3
#define F_SETFL  4
#define F_GETLK  5
#define F_SETLK  6
#define F_SETLKW 7
#define F_SETOWN 8
#define F_GETOWN 9
#define F_SETSIG 10
#define F_GETSIG 11

#define F_SETOWN_EX 15
#define F_GETOWN_EX 16

#define F_GETOWNER_UIDS 17

#define POSIX_FADV_NORMAL     0
#define POSIX_FADV_RANDOM     1
#define POSIX_FADV_SEQUENTIAL 2
#define POSIX_FADV_WILLNEED   3
#ifndef POSIX_FADV_DONTNEED
#define POSIX_FADV_DONTNEED   4
#define POSIX_FADV_NOREUSE    5
```

## copy on write: page faultによるメモリ割り当て

### page faultを判定するには

```
ESR-EL1
     EC:       bits[31:26] = 0x24
     ISS/DFSC: bits[5:0]   =  9-11 (access flag fault)
                           = 13-15 (permission fault)
                           =  4- 7 (translation fault) if addr < (1 << 48)
```

## mmap関係定数

- `libc/include/sys/mman.h`

```
#define MAP_FAILED ((void *) -1)
#define MAP_SHARED     0x01
#define MAP_PRIVATE    0x02
#define MAP_SHARED_VALIDATE 0x03
#define MAP_TYPE       0x0f
#define MAP_FIXED      0x10
#define MAP_ANON       0x20
#define MAP_ANONYMOUS  MAP_ANON
#define MAP_NORESERVE  0x4000
#define MAP_GROWSDOWN  0x0100
#define MAP_DENYWRITE  0x0800
#define MAP_EXECUTABLE 0x1000
#define MAP_LOCKED     0x2000
#define MAP_POPULATE   0x8000
#define MAP_NONBLOCK   0x10000
#define MAP_STACK      0x20000
#define MAP_HUGETLB    0x40000
#define MAP_SYNC       0x80000
#define MAP_FIXED_NOREPLACE 0x100000
#define MAP_FILE       0

#define PROT_NONE      0
#define PROT_READ      1
#define PROT_WRITE     2
#define PROT_EXEC      4
```

- `include/linux/mm.h`

```
#define VM_NONE     0x00000000
#define VM_READ     0x00000001  /* currently active flags */
#define VM_WRITE    0x00000002
#define VM_EXEC     0x00000004
#define VM_SHARED   0x00000008
```

## `source/linux/mm/mmap.c`より

### 現在の実装におけるマッピングタイプとprotの効果について

このようになるのはx86の限られたページ保護ハードウェアによる。カッコ内が期待される動作である。

```
map_type prot
             PROT_NONE   PROT_READ     PROT_WRITE      PROT_EXEC
MAP_SHARED   r: (no) no  r: (yes) yes  r: (no) yes     r: (no) yes
             w: (no) no  w: (no) no    w: (yes) yes    w: (no) no
             x: (no) no  x: (no) yes   x: (no) yes     x: (yes) yes

MAP_PRIVATE  r: (no) no  r: (yes) yes  r: (no) yes     r: (no) yes
             w: (no) no  w: (no) no    w: (copy) copy  w: (no) no
             x: (no) no  x: (no) yes   x: (no) yes     x: (yes) yes

arm64では, PROT_EXECはMAP_SHARED, MAP_PRIVATEともに次のようになる。
                                                       r: (no) no
                                                       w: (no) no
                                                       x: (yes) yes
```

### arm64

```
#define __P000  PAGE_NONE               // #  0:   NONE
#define __P001  PAGE_READONLY           // #  1:   READ
#define __P010  PAGE_READONLY           //    2:   WRITE
#define __P011  PAGE_READONLY           //    3:   READ/WRITE
#define __P100  PAGE_EXECONLY           // #  4:   EXEC
#define __P101  PAGE_READONLY_EXEC      // #  5:   READ/EXEC
#define __P110  PAGE_READONLY_EXEC      //    6:   WRITE/EXEC
#define __P111  PAGE_READONLY_EXEC      //    7:   READ/WRITE/EXEC

#define __S000  PAGE_NONE               //    8:   SHARE/NONE
#define __S001  PAGE_READONLY           //    9:   SHARE/READ
#define __S010  PAGE_SHARED             // # 10:   SHARE/WRITE
#define __S011  PAGE_SHARED             //   11:   SHARE/READ/WRITE
#define __S100  PAGE_EXECONLY           //   12:   SHARE/EXEC
#define __S101  PAGE_READONLY_EXEC      //   13:   SHARE/READ/EXEC
#define __S110  PAGE_SHARED_EXEC        // # 14:   SHARE/WRITE/EXEC
#define __S111  PAGE_SHARED_EXEC        //   15:   SHARE/REAE/WRITE/EXEC

#define _PAGE_DEFAULT       (PROT_DEFAULT | PTE_ATTRINDX(MT_NORMAL))

#define PAGE_NONE           (_PAGE_DEFAULT & ~PTE_VALID) | PTE_PROT_NONE | PTE_RDONLY | PTE_PXN | PTE_UXN
#define PAGE_READONLY       _PAGE_DEFAULT | PTE_USER | PTE_RDONLY | PTE_NG | PTE_PXN | PTE_UXN
#define PAGE_EXECONLY       _PAGE_DEFAULT | PTE_RDONLY | PTE_NG | PTE_PXN
#define PAGE_READONLY_EXEC  _PAGE_DEFAULT | PTE_USER | PTE_RDONLY | PTE_NG | PTE_PXN
#define PAGE_SHARED         _PAGE_DEFAULT | PTE_USER | PTE_NG | PTE_PXN | PTE_UXN | PTE_WRITE
#define PAGE_SHARED_EXEC    _PAGE_DEFAULT | PTE_USER | PTE_NG | PTE_PXN | PTE_WRITE


#define PTE_TYPE_MASK       (_AT(pteval_t, 3) << 0)
#define PTE_TYPE_FAULT      (_AT(pteval_t, 0) << 0)
#define PTE_TYPE_PAGE       (_AT(pteval_t, 3) << 0)
#define PTE_TABLE_BIT       (_AT(pteval_t, 1) << 1)

#define PTE_USER        (_AT(pteval_t, 1) << 6)     /* AP[1] */
#define PTE_RDONLY      (_AT(pteval_t, 1) << 7)     /* AP[2] */
#define PTE_SHARED      (_AT(pteval_t, 3) << 8)     /* SH[1:0], inner shareable */
#define PTE_AF          (_AT(pteval_t, 1) << 10)    /* Access Flag */
#define PTE_NG          (_AT(pteval_t, 1) << 11)    /* nG */
#define PTE_DBM         (_AT(pteval_t, 1) << 51)    /* Dirty Bit Management */
#define PTE_CONT        (_AT(pteval_t, 1) << 52)    /* Contiguous range */
#define PTE_PXN         (_AT(pteval_t, 1) << 53)    /* Privileged XN */
#define PTE_UXN         (_AT(pteval_t, 1) << 54)    /* User XN */
#define PTE_HYP_XN      (_AT(pteval_t, 1) << 54)    /* HYP XN */
```

## `cyanurus`は次のsyscallを実装している

```
  switch(number) {
    case 1:   syscall_exit();
    case 2:   syscall_fork();
    case 3:   syscall_read();
    case 4:   syscall_write();
    case 5:   syscall_open();
    case 6:   syscall_close();
    case 10:  syscall_unlink();
    case 11:  syscall_execve();
    case 20:  syscall_getpid();
    case 37:  syscall_kill();
    case 39:  syscall_mkdir();
    case 40:  syscall_rmdir();
    case 41:  syscall_dup();
    case 42:  syscall_pipe();
    case 45:  syscall_brk();
    case 54:  syscall_ioctl();
    case 63:  syscall_dup2();
    case 64:  syscall_getppid();
    case 114: syscall_wait4();
    case 119: syscall_sigreturn();
    case 122: syscall_uname();
    case 140: syscall__llseek();
    case 145: syscall_readv();
    case 146: syscall_writev();
    case 174: syscall_rt_sigaction();
    case 175: syscall_rt_sigprocmask();
    case 183: syscall_getcwd();
    case 195: syscall_stat64();
    case 196: syscall_lstat64();
    case 197: syscall_fstat64();
    case 217: syscall_getdents64();
    case 221: syscall_fcntl64();
    case 358: syscall_dup3();
    case 359: syscall_pipe2();

    case 248: // exit_group
    case 270: // fadvise64_64
      syscall_pass(context, 0);
      break;

    case 199: // getuid32
    case 200: // getgid32
    case 201: // geteuid32
    case 202: // getegid32
      syscall_pass(context, 1);
      break;

    default:
      logger_debug("unknown syscall: %d", number);
      syscall_pass(context, -EINVAL);
      break;
  }
```
