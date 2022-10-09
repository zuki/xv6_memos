# bash-5.1.8をインストール

```
$ cp MUSL-LIB/musl-gcc.specs .
$ ./configure --host=aarch64-linux-gnu --enable-minimal-config --enable-static-link --disable-nls CFLAGS="-specs /home/vagrant/xv6-fudan/bash-5.1.8/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096"
$ make
rm -f mksyntax
gcc -DPROGRAM='"bash"' -DCONF_HOSTTYPE='"aarch64"' -DCONF_OSTYPE='"linux-gnu"' -DCONF_MACHTYPE='"aarch64-unknown-linux-gnu"' -DCONF_VENDOR='"unknown"' -DLOCALEDIR='"/usr/local/share/locale"' -DPACKAGE='"bash"' -DSHELL -DHAVE_CONFIG_H   -I.  -I. -I./include -I./lib   -g -DCROSS_COMPILING -rdynamic -g -DCROSS_COMPILING -o mksyntax ./mksyntax.c
In file included from ./mksyntax.c:23:
./config.h:334:16: error: duplicate ‘unsigned’
  334 | #define size_t unsigned int
      |                ^~~~~~~~
./config.h:334:25: error: two or more data types in declaration specifiers
  334 | #define size_t unsigned int
      |                         ^~~
./config.h:298:15: error: two or more data types in declaration specifiers
  298 | #define off_t long int
      |               ^~~~
./config.h:298:20: error: two or more data types in declaration specifiers
  298 | #define off_t long int
      |                    ^~~
./config.h:337:17: error: two or more data types in declaration specifiers
  337 | #define ssize_t int
      |                 ^~~
./mksyntax.c: In function ‘main’:
```

### muse-gcc.specsを修正

```
$ make
...
aarch64-linux-gnu-gcc -L./builtins -L./lib/readline -L./lib/readline -L./lib/glob -L./lib/tilde -L./lib/malloc -L./lib/sh  -rdynamic -specs /home/vagrant/xv6-fudan/bash-5.1.8/musl-gcc.specs -std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096   -static -o bash shell.o eval.o y.tab.o general.o make_cmd.o print_cmd.o  dispose_cmd.o execute_cmd.o variables.o copy_cmd.o error.o expr.o flags.o nojobs.o subst.o hashcmd.o hashlib.o mailcheck.o trap.o input.o unwind_prot.o pathexp.o sig.o test.o version.o alias.o array.o arrayfunc.o assoc.o braces.o bracecomp.o bashhist.o bashline.o  list.o stringlib.o locale.o findcmd.o redir.o pcomplete.o pcomplib.o syntax.o xmalloc.o signames.o -lbuiltins -lglob -lsh    -ltilde -lmalloc
ls -l bash
-rwxrwxr-x 1 vagrant vagrant 978336 Nov 23 12:13 bash
aarch64-linux-gnu-size bash
   text       data        bss        dec        hex    filename
 825540      21360      22328     869228      d436c    bash
$ ls -l bash
-rwxrwxr-x 1 vagrant vagrant 978336 Nov 23 12:13 bash
```

## bash大きすぎ

```
Welcome to xv6 2021-09-15 (oldmalloc)  tty

 login: vagrant
Password:
bmap: too big file
```

## bmap, mkfsを二重間接まで拡張

```
Welcome to xv6 2021-09-15 (oldmalloc)  tty

 login: vagrant
Password:
bret: no buffers
```

## params.hを調整

```
 login: vagrant
Password:
unexpected interrupt 0 at cpu 0: sp=4d7f80
Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x0
 SPSR_EL1 0x60000000 FAR_EL1 0x0                // stack overflowか?
    irq_error: irq of type 8 unimplemented.

 login: root
Password:
# ls -l /bin/bash

-rwxr-xr-x 1 root root 978336 Nov 24  2021 /bin/bash
# /bin/bash
unexpected interrupt 0 at cpu 0: sp=4d7f70
Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x0
 SPSR_EL1 0x60000000 FAR_EL1 0x442f00       // 442f00: a9425bf5  ldp x21, x22, [sp, #32]
    irq_error: irq of type 8 unimplemented.
```

```
Entry point address:               0x401c50
Start of section headers:          977248 (bytes into file)

LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x00000000000c99e8 0x00000000000c99e8  R E    0x1000
LOAD           0x00000000000ca340 0x00000000004cb340 0x00000000004cb340
                 0x0000000000005370 0x000000000000aaa8  RW     0x1000

0xc99e8 + 0xaaa8 = 0xd4490
```

## mkfsで間接テーブル用の領域をローカルからグローバルに変更（stackを使わない）

結果は同じだった

## 二重間接が正しく実装されているか調査

### big.cを実行

- 16523 = 11 + 128 + 128 * 128 全ブロックを書き込むのは時間がかかりすぎるので267ブロックで終了させる

```
# /bin/big
.........#.........#......                                  // printfはバッファリングするので
wrote 267 sectors                                           // 途中経過が出ない
log_write outside of trans: [1] = 0                         // syscallのどれかでbegin_opがない
kern/console.c:284: kernel panic at cpu 0.
QEMU: Terminated

# ls -l
total 1073

-rwxr-x-wx 1 root root 1093632 Sep 30  2021 big.file        // 書き込み自体は成功
drwxrwxr-x 1 root root    2080 Nov 24  2021 bin
drwxrwxr-x 1 root root      96 Nov 24  2021 dev
drwxrwxr-x 1 root root      64 Nov 24  2021 etc
drwxrwxr-x 1 root root      48 Nov 24  2021 home

# /bin/big                                                  // 小さなファイルに変更すると問題なし
.
wrote 20 sectors
closed
opened for read
read; ok
done; ok
```

### filewrite, filereadにbegin_op, end_opを追加

```
# /bin/big
.........#....
wrote 140 sectors
opend rfd
sector: 138 = 138
sector: 139 = 0           // 二重間接部分が読み込まれない
close rfd
read; err=1, ok
done; ok
```

### dinodeのaddrs[]の配列数の直し忘れだった

```
# /bin/big
.........#....
wrote 140 sectors
opend rfd
sector: 138 = 138
sector: 139 = 139
close rfd
read; err=0, ok
done; ok
```

## 二重間接によるファイル容量増加への対応は証明されたがbashは動かない

```
# bash
unexpected interrupt 0 at cpu 0: sp=4d7f70
Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x0
 SPSR_EL1 0x60000000 FAR_EL1 0x409f00
    irq_error: irq of type 8 unimplemented.
```

## とりあえず凍結

# 再挑戦

```
$ CC=/usr/local/musl/bin/musl-gcc ./configure -host=aarch64-linux-gnu --disable-nls --prefix=$(pwd)/bin
$ make
$ make install
$ cd bin
$ ls
bin  include  lib  share
$ ls bin
bash  bashbug
$ ls lib
bash  pkgconfig
$ ls lib/bash
accept    fdflags  loadables.h   mktemp    push      setpgid   truefalse
basename  finfo    logname       mypid     realpath  sleep     tty
csv       head     Makefile.inc  pathchk   rm        strftime  uname
cut       id       mkdir         print     rmdir     sync      unlink
dirname   ln       mkfifo        printenv  seq       tee       whoami
$ ls lib/pkgconfig/
bash.pc
$ ls include/
bash
$ ls include/bash
alias.h      builtins       error.h      quit.h         variables.h
arrayfunc.h  builtins.h     externs.h    shell.h        version.h
array.h      command.h      general.h    sig.h          xmalloc.h
assoc.h      config-bot.h   hashlib.h    siglist.h      y.tab.h
bashansi.h   config.h       include      signames.h
bashintl.h   config-top.h   jobs.h       subst.h
bashjmp.h    conftypes.h    make_cmd.h   syntax.h
bashtypes.h  dispose_cmd.h  pathnames.h  unwind_prot.h
$ ls share
doc  info  man
$ file bin/bash
bin/bash: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-aarch64.so.1, with debug_info, not stripped
```

## xv6に組み込む

```
$ cp bin/bin/bash XV6/user/local/bin/
$ cd XV6
$ vi mksd.mk
@@ -30,7 +30,8 @@ BASH := $(BASHSRC)/bash

 BINUTILSDIR := user/local/bin
 BINUTILS := $(BINUTILSDIR)/as $(BINUTILSDIR)/gprof $(BINUTILSDIR)/nm $(BINUTILSDIR)/objcopy \
-       $(BINUTILSDIR)/objdump $(BINUTILSDIR)/readelf $(BINUTILSDIR)/size $(BINUTILSDIR)/strings
+       $(BINUTILSDIR)/objdump $(BINUTILSDIR)/readelf $(BINUTILSDIR)/size $(BINUTILSDIR)/strings \
+       $(BINUTILSDIR)/bash

 SECTOR_SIZE := 512

$ vi user/src/mkfs/main.c
@@ -266,8 +266,8 @@ main(int argc, char *argv[])
     // /usr/loca/bin
     char *localbins[] = { "./user/local/bin/as", "./user/local/bin/nm", "./user/local/bin/objcopy",
                      "./user/local/bin/objdump", "./user/local/bin/readelf", "./user/local/bin/size",
-                     "./user/local/bin/strings", "./user/local/bin/gprof" };
-    copy_file(0, 8, localbins, localbinino, 0, 0, S_IFREG|0755);
+                     "./user/local/bin/strings", "./user/local/bin/gprof", "./user/local/bin/bash" };
+    copy_file(0, 9, localbins, localbinino, 0, 0, S_IFREG|0755);

     // fix size of root inode dir
     rinode(rootino, &din);
$ make
$ make qemu

# bash
 x0: 0xfffffffffff6f8b8
 x1: 0x8e8
 x2: 0xfffffffffff83e08
 x3: 0xfffffffffff83520
 x4: 0xafee0
x30: 0x0
 sp: 0xafce0

data abort: ec=0x24, iss=0x4, dfs=4
insruction: 0x61760
fault addr: 0xfffffffffff83528
```

```
0xafff0: argv[0]=bash
0xaffe0: envp[0]=PWD=/.
0xaffd8: auxv[27]=0
0xaffd0: auxv[26]=0
0xaffc8: auxv[25]=0
0xaffc0: auxv[24]=16
0xaffb8: auxv[23]=4096
0xaffb0: auxv[22]=6
0xaffa8: auxv[21]=100
0xaffa0: auxv[20]=17
0xaff98: auxv[19]=178954304   0xaaa_a040
0xaff90: auxv[18]=3           AT_PHDR = exec_ehdr.e_phoff
0xaff88: auxv[17]=56
0xaff80: auxv[16]=4
0xaff78: auxv[15]=7
0xaff70: auxv[14]=5
0xaff68: auxv[13]=0
0xaff60: auxv[12]=7
0xaff58: auxv[11]=0
0xaff50: auxv[10]=8
0xaff48: auxv[9]=179176596    0xaae_0494
0xaff40: auxv[8]=9            AT_ENTRY = load_bias + exec_ehdr.e_entry
0xaff38: auxv[7]=0
0xaff30: auxv[6]=11
0xaff28: auxv[5]=0
0xaff20: auxv[4]=12
0xaff18: auxv[3]=0
0xaff10: auxv[2]=13
0xaff08: auxv[1]=0
0xaff00: auxv[0]=14
0xaff00: estack[1]=0x0
0xafef8: estack[0]=0xaffe0
0xafef0: ustack[1]=0x0
0xafee8: ustack[0]=0xafff0
0xafee0: argc=1
```

## 該当箇所

```
$ aarch64-linux-gnu-objdump -d user/lib/ld-musl-aarch64.so.1
0000000000061630 <_dlstart_c>:
for (i=argc+1; argv[i]; i++);
   61630:       aa0003e4        mov     x4, x0          // x4 = sp (引数1)
   61634:       d10803ff        sub     sp, sp, #0x200  // sp -= 0x200 (512) 64個分
   61638:       f8408402        ldr     x2, [x0], #8    // x2 = argc, x0 += 8
   6163c:       11000442        add     w2, w2, #0x1    // argc += 1
   61640:       93407c42        sxtw    x2, w2          // x2 = i = w2の符号拡張
   61644:       f8627803        ldr     x3, [x0, x2, lsl #3] // x3 = argv[i]
   61648:       91000442        add     x2, x2, #0x1    // i++
   6164c:       b5ffffc3        cbnz    x3, 61644 <L02>

   61650:       910003e5        mov     x5, sp
   61654:       8b020c02        add     x2, x0, x2, lsl #3
   61658:       d2800000        mov     x0, #0x0                        // #0
L01:
   6165c:       f82078bf        str     xzr, [x5, x0, lsl #3]
   61660:       91000400        add     x0, x0, #0x1
L02:
   61664:       f100801f        cmp     x0, #0x20
   61668:       54ffffa1        b.ne    6165c <L01>  // b.any
L03:
   6166c:       f9400043        ldr     x3, [x2]
   61670:       b5000483        cbnz    x3, 61700 <L10>

   61674:       910403e5        add     x5, sp, #0x100
L04:
   61678:       f82378bf        str     xzr, [x5, x3, lsl #3]
   6167c:       91000463        add     x3, x3, #0x1
   61680:       f100807f        cmp     x3, #0x20
   61684:       54ffffa1        b.ne    61678 <L04>  // b.any

   61688:       aa0103e0        mov     x0, x1        // x0 = dynv
L05:
   6168c:       f9400002        ldr     x2, [x0]      // x2 = dynv[0]
   61690:       b5000442        cbnz    x2, 61718 <L11>
   61694:       f9401fe0        ldr     x0, [sp, #56]
   61698:       b5000140        cbnz    x0, 616c0 <L07>
   6169c:       a9418fe2        ldp     x2, x3, [sp, #24]
   616a0:       f94017e0        ldr     x0, [sp, #40]
L06:
   616a4:       b40000e0        cbz     x0, 616c0 <L07>
   616a8:       b9400045        ldr     w5, [x2]
   616ac:       d1000400        sub     x0, x0, #0x1
   616b0:       710008bf        cmp     w5, #0x2
   616b4:       540003e1        b.ne    61730 <L13>  // b.any
   616b8:       f9400842        ldr     x2, [x2, #16]
   616bc:       cb020020        sub     x0, x1, x2
L07:
   616c0:       a95887e2        ldp     x2, x1, [sp, #392]
   616c4:       8b020002        add     x2, x0, x2
   616c8:       8b010042        add     x2, x2, x1
L08:
   616cc:       cb010045        sub     x5, x2, x1
   616d0:       b5000341        cbnz    x1, 61738 <L14>
   616d4:       a95387e2        ldp     x2, x1, [sp, #312]
   616d8:       8b020002        add     x2, x0, x2
   616dc:       8b010042        add     x2, x2, x1
L09:
   616e0:       cb010043        sub     x3, x2, x1
   616e4:       b50003e1        cbnz    x1, 61760 <ERROR_L16> // ここで71760へb
   616e8:       b0000241        adrp    x1, aa000 <malloc+0x92e40>
   616ec:       f941ac22        ldr     x2, [x1, #856]
   616f0:       aa0403e1        mov     x1, x4
   616f4:       910803ff        add     sp, sp, #0x200
   616f8:       aa0203f0        mov     x16, x2
   616fc:       d61f0200        br      x16
L10:
   61700:       f1007c7f        cmp     x3, #0x1f
   61704:       54000068        b.hi    61710 <_dlstart_c+0xe0>  // b.pmore
   61708:       f9400440        ldr     x0, [x2, #8]
   6170c:       f82378a0        str     x0, [x5, x3, lsl #3]
   61710:       91004042        add     x2, x2, #0x10
   61714:       17ffffd6        b       6166c <L03>
L11:
   61718:       f1007c5f        cmp     x2, #0x1f
   6171c:       54000068        b.hi    61728 <L12>  // b.pmore
   61720:       f9400403        ldr     x3, [x0, #8]          // dynv[i*1]
   61724:       f82278a3        str     x3, [x5, x2, lsl #3]  // dynv[dynv[i]] = x3
L12:
   61728:       91004000        add     x0, x0, #0x10
   6172c:       17ffffd8        b       6168c <L05>
L13:
   61730:       8b030042        add     x2, x2, x3
   61734:       17ffffdc        b       616a4 <L06>
L14:
   61738:       f94004a3        ldr     x3, [x5, #8]
   6173c:       92407863        and     x3, x3, #0x7fffffff
   61740:       f1100c7f        cmp     x3, #0x403
   61744:       540000a1        b.ne    61758 <L15>  // b.any
   61748:       f94000a5        ldr     x5, [x5]
   6174c:       f8656803        ldr     x3, [x0, x5]
   61750:       8b000063        add     x3, x3, x0
   61754:       f8256803        str     x3, [x0, x5]
L15:
   61758:       d1004021        sub     x1, x1, #0x10
   6175c:       17ffffdc        b       616cc <L08>
ERROR_L16:
   61760:       f9400465        ldr     x5, [x3, #8]              // ここでエラー
   61764:       924078a5        and     x5, x5, #0x7fffffff
   61768:       f1100cbf        cmp     x5, #0x403
   6176c:       540000a1        b.ne    61780 <L17>  // b.any
   61770:       f9400065        ldr     x5, [x3]
   61774:       f9400863        ldr     x3, [x3, #16]
   61778:       8b000063        add     x3, x3, x0
   6177c:       f82068a3        str     x3, [x5, x0]
L17:
   61780:       d1006021        sub     x1, x1, #0x18
   61784:       17ffffd7        b       616e0 <L09>
```

```c
hidden void _dlstart_c(size_t *sp, size_t *dynv)
{
    size_t i, aux[AUX_CNT], dyn[DYN_CNT];
    size_t *rel, rel_size, base;

    int argc = *sp;
    char **argv = (void *)(sp+1);

    for (i=argc+1; argv[i]; i++);
    size_t *auxv = (void *)(argv+i+1);

    for (i=0; i<AUX_CNT; i++) aux[i] = 0;
    for (i=0; auxv[i]; i+=2) if (auxv[i]<AUX_CNT)
        aux[auxv[i]] = auxv[i+1];

    for (i=0; i<DYN_CNT; i++) dyn[i] = 0;
    for (i=0; dynv[i]; i+=2) if (dynv[i]<DYN_CNT)
        dyn[dynv[i]] = dynv[i+1];

    base = aux[AT_BASE];
    if (!base) {
        size_t phnum = aux[AT_PHNUM];
        size_t phentsize = aux[AT_PHENT];
        Phdr *ph = (void *)aux[AT_PHDR];
        for (i=phnum; i--; ph = (void *)((char *)ph + phentsize)) {
            if (ph->p_type == PT_DYNAMIC) {
                base = (size_t)dynv - ph->p_vaddr;
                break;
            }
        }
    }

    if (NEED_MIPS_GOT_RELOCS) {
        size_t local_cnt = 0;
        size_t *got = (void *)(base + dyn[DT_PLTGOT]);
        for (i=0; dynv[i]; i+=2) if (dynv[i]==DT_MIPS_LOCAL_GOTNO)
            local_cnt = dynv[i+1];
        for (i=0; i<local_cnt; i++) got[i] += base;
    }

    rel = (void *)(base+dyn[DT_REL]);
    rel_size = dyn[DT_RELSZ];
    for (; rel_size; rel+=2, rel_size-=2*sizeof(size_t)) {
        if (!IS_RELATIVE(rel[1], 0)) continue;
        size_t *rel_addr = (void *)(base + rel[0]);
        *rel_addr += base;
    }

    rel = (void *)(base+dyn[DT_RELA]);
    rel_size = dyn[DT_RELASZ];
    for (; rel_size; rel+=3, rel_size-=3*sizeof(size_t)) {
        if (!IS_RELATIVE(rel[1], 0)) continue;
        size_t *rel_addr = (void *)(base + rel[0]);
        *rel_addr = base + rel[2];
    }
#endif

    stage2_func dls2;
    GETFUNCSYM(&dls2, __dls2, base+dyn[DT_PLTGOT]);
    dls2((void *)base, sp);
```
