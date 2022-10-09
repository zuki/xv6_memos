# mmap版でコマンドが異常終了する

## エラー時のレジスタダンプとスタックダンプ

```
======= PROC[8]: /bin/dash TRAP FRAME DUMP =========
sp:    0x0000fffffffffb20  tpidr: 0x00000000004342f0
spsr:  0x0000000000000000  esr:   0x0000000002000000
elr:   0x0000fffffffffb20  x0:    0x0000000000000000
x1:    0x0000000000000000  x2:    0x0000000000000000
x3:    0x0000000000000008  x4:    0x0000000000435068
x5:    0x0000000000434bc8  x6:    0x2f2f2f2f2f2f2f2f
x7:    0x0000000000432000  x8:    0x00000000000000dc
...
x29:   0x0000fffffffffb20  x30:   0x0000fffffffffb20        // x30がspと同じ -> 明らかにおかしい
===================== DUMP END =====================

==== stack proc[8] =========
fffffffffb00: 0000000000432000 000000000041bd98 0000fffffffffb20 000000000040c820
fffffffffb20: 0000fffffffffc90 00000000004044d4 0000000000429ff5 0000000000434b98
```

## fork()内でのsp0のダンプ

```
(gdb) p/x sp
$1 = 0xfffffffffb10
(gdb) x/16gx (sp - 16)
0xfffffffffb00:	0x0000000000000000	0x0000000000432000
0xfffffffffb10:	0x000000000040c708	0x00114b8f3798ec48
0xfffffffffb20:	0x0000fffffffffc90	0x00000000004044d4
```


- gdbでステップ実行したり、デバッグプリントを入れると動く場合が多い
- 問題が発生するのは`dash/src/jobc.s#vforkcexec()#L74`で子プロセスでx30が復元できないのが原因と思われる

  ```
  pid = vfork();        // 子プロセスのこの行でエラー

  000000000041bd80 <vfork>:
  41bd80:       d2801b88        mov     x8, #0xdc                       // #220
  41bd84:       d2800220        mov     x0, #0x11                       // #17
  41bd88:       d2800001        mov     x1, #0x0                        // #0
  41bd8c:       f81f0ffe        str     x30, [sp, #-16]!                // x30 = 0x000000000040c708
  41bd90:       d4000001        svc     #0x0
  41bd94:       97fff861        bl      419f18 <__syscall_ret>
  41bd98:       f84107fe        ldr     x30, [sp], #16                  // <= このx30が正しく戻らない
  41bd9c:       d65f03c0        ret
  ```
void print_mmap_list(struct proc *, const char *);
## vforkexec()のコンパイル結果

```c
struct job *vforkexec(union node *n, char **argv, const char *path, int idx)
{
	struct job *jp;
	int pid;

	jp = makejob(n, 1);

	sigblockall(NULL);
	vforked++;

	pid = vfork();

	if (!pid) {
		forkchild(jp, n, FORK_FG);
		sigclearmask();
		shellexec(argv, path, idx);
		/* NOTREACHED */
	}

	vforked = 0;
	sigclearmask();
	forkparent(jp, n, FORK_FG, pid);

	return jp;
}
```

`432660 d _GLOBAL_OFFSET_TABLE_`

```
  [ 4] .rodata           PROGBITS        0000000000428db0 028db0 002e7c 00   A  0   0 16
  [ 5] .eh_frame         PROGBITS        000000000042bc30 02bc30 003db0 00   A  0   0  8
  [ 6] .init_array       INIT_ARRAY      0000000000431930 030930 000008 08  WA  0   0  8
  [ 7] .fini_array       FINI_ARRAY      0000000000431938 030938 000008 08  WA  0   0  8
  [ 8] .data.rel.ro      PROGBITS        0000000000431940 030940 000d20 00  WA  0   0 16
  [ 9] .got              PROGBITS        0000000000432660 031660 000980 08  WA  0   0  8
  [10] .got.plt          PROGBITS        0000000000432fe8 031fe8 000018 08  WA  0   0  8
  [11] .data             PROGBITS        0000000000433000 032000 000320 00  WA  0   0 16
  [12] .bss              NOBITS          0000000000433320 032320 001de0 00  WA  0   0 16
```

`[x19, #xx] = 0x31000 + toHex(xx) => xxdのアドレス`

```
  000000000040c6a0 <vforkexec>:     // (union node *n, char **argv, const char *path, int idx)
  40c6a0:       a9a97bfd        stp     x29, x30, [sp, #-368]!          // 46 * 8
  40c6a4:       aa0103e5        mov     x5, x1                          // v5 = **argv
  40c6a8:       52800021        mov     w1, #0x1                        // w1 = #1
  40c6ac:       910003fd        mov     x29, sp
  40c6b0:       a90153f3        stp     x19, x20, [sp, #16]
  40c6b4:       d0000133        adrp    x19, 432000 <signal_names+0x100>        // 0x432000 => 0x302e0
  40c6b8:       f943d264        ldr     x4, [x19, #1952]                // x4 = [0x4327a0] => 0x434d20 (bss)
  40c6bc:       f9002be2        str     x2, [sp, #80]
  40c6c0:       f90013f5        str     x21, [sp, #32]
  40c6c4:       f9400082        ldr     x2, [x4]                        //
  40c6c8:       f900b7e2        str     x2, [sp, #360]
  40c6cc:       d2800002        mov     x2, #0x0                        // #0
  40c6d0:       a90417e0        stp     x0, x5, [sp, #64]
  40c6d4:       b9005fe3        str     w3, [sp, #92]
  40c6d8:       97ffff0a        bl      40c300 <makejob>                // jp = makejob(n, 1)
  40c6dc:       f9001fe0        str     x0, [sp, #56]                   // [sp, 56] = jp
  40c6e0:       f9464261        ldr     x1, [x19, #3200]                // x1 = 0x414d78
  40c6e4:       d2800000        mov     x0, #0x0                        // #0
  40c6e8:       d63f0020        blr     x1                              // sigblockall(0)
  40c6ec:       f943c673        ldr     x19, [x19, #1928]
  40c6f0:       b9400260        ldr     w0, [x19]                       // [0x432788] = vfork
  40c6f4:       11000400        add     w0, w0, #0x1
  40c6f8:       b9000260        str     w0, [x19]
  40c6fc:       d0000133        adrp    x19, 432000 <signal_names+0x100>
  40c700:       f946c260        ldr     x0, [x19, #3456]                // x0 = 0x41bd80 = <vfrok>
  40c704:       d63f0000        blr     x0                              // blr <vfork>
  40c708:       34000a80        cbz     w0, 40c858 <vforkexec+0x1b8>    // <= sp0ベースの[0xfffffffffb18]
  40c70c:       d0000133        adrp    x19, 432000 <signal_names+0x100>
  40c710:       9103a3f5        add     x21, sp, #0xe8
  40c714:       2a0003f4        mov     w20, w0
  40c718:       aa1503e0        mov     x0, x21
  40c71c:       f943c673        ldr     x19, [x19, #1928]
  40c720:       b900027f        str     wzr, [x19]
  40c724:       d0000133        adrp    x19, 432000 <signal_names+0x100>
  40c728:       f943a261        ldr     x1, [x19, #1856]
  40c72c:       d63f0020        blr     x1                              //  <sigemptyset> x1 = 0x41c138
  40c730:       f947ea63        ldr     x3, [x19, #4048]
  40c734:       aa1503e1        mov     x1, x21                         // x1 = 0xfffffffffc08
  40c738:       d2800002        mov     x2, #0x0                        // #0
  40c73c:       52800040        mov     w0, #0x2                        // #2
  40c740:       d63f0060        blr     x3                              // <sigprocmask> x3 = 0x41c1a8
  40c744:       37f80794        tbnz    w20, #31, 40c834 <vforkexec+0x194>
  40c748:       f9401fe2        ldr     x2, [sp, #56]
  40c74c:       b4000402        cbz     x2, 40c7cc <vforkexec+0x12c>
  40c750:       39407c41        ldrb    w1, [x2, #31]
  40c754:       79403840        ldrh    w0, [x2, #28]
  40c758:       36080161        tbz     w1, #1, 40c784 <vforkexec+0xe4>
  40c75c:       2a1403e1        mov     w1, w20
  40c760:       34000060        cbz     w0, 40c76c <vforkexec+0xcc>
  40c764:       f9400840        ldr     x0, [x2, #16]
  40c768:       b9400001        ldr     w1, [x0]
  40c76c:       d0000133        adrp    x19, 432000 <signal_names+0x100>
  40c770:       2a1403e0        mov     w0, w20
  40c774:       f944ba62        ldr     x2, [x19, #2416]
  40c778:       d63f0040        blr     x2                              // <setpgid> x2 = 0x420654
  40c77c:       f9401fe0        ldr     x0, [sp, #56]
  40c780:       79403800        ldrh    w0, [x0, #28]
  40c784:       f9401fe3        ldr     x3, [sp, #56]
  40c788:       d0000133        adrp    x19, 432000 <signal_names+0x100>
  40c78c:       d37c3c01        ubfiz   x1, x0, #4, #16
  40c790:       11000400        add     w0, w0, #0x1
  40c794:       f9479a65        ldr     x5, [x19, #3888]
  40c798:       79003860        strh    w0, [x3, #28]
  40c79c:       f9400862        ldr     x2, [x3, #16]
  40c7a0:       12800004        mov     w4, #0xffffffff                 // #-1
  40c7a4:       b94000a0        ldr     w0, [x5]
  40c7a8:       8b010055        add     x21, x2, x1
  40c7ac:       7100001f        cmp     w0, #0x0
  40c7b0:       b8216854        str     w20, [x2, x1]
  40c7b4:       f946ca63        ldr     x3, [x19, #3472]
  40c7b8:       b90006a4        str     w4, [x21, #4]
  40c7bc:       f94023e0        ldr     x0, [sp, #64]
  40c7c0:       f90006a3        str     x3, [x21, #8]
  40c7c4:       fa401804        ccmp    x0, #0x0, #0x4, ne  // ne = any
  40c7c8:       540001a1        b.ne    40c7fc <vforkexec+0x15c>  // b.any
  40c7cc:       d0000133        adrp    x19, 432000 <signal_names+0x100>
  40c7d0:       f943d273        ldr     x19, [x19, #1952]
  40c7d4:       f940b7e0        ldr     x0, [sp, #360]
  40c7d8:       f9400261        ldr     x1, [x19]
  40c7dc:       eb010000        subs    x0, x0, x1
  40c7e0:       d2800001        mov     x1, #0x0                        // #0
  40c7e4:       54000221        b.ne    40c828 <vforkexec+0x188>  // b.any
  40c7e8:       a94153f3        ldp     x19, x20, [sp, #16]
  40c7ec:       f94013f5        ldr     x21, [sp, #32]
  40c7f0:       f9401fe0        ldr     x0, [sp, #56]
  40c7f4:       a8d77bfd        ldp     x29, x30, [sp], #368
  40c7f8:       d65f03c0        ret
  40c7fc:       f9479e73        ldr     x19, [x19, #3896]
  40c800:       f0000121        adrp    x1, 433000 <__dso_handle>
  40c804:       f9400262        ldr     x2, [x19]
  40c808:       f9025022        str     x2, [x1, #1184]
  40c80c:       97fffaa9        bl      40b2b0 <cmdtxt>
  40c810:       f9400260        ldr     x0, [x19]
  40c814:       d0000133        adrp    x19, 432000 <signal_names+0x100>
  40c818:       f9453e61        ldr     x1, [x19, #2680]
  40c81c:       d63f0020        blr     x1                        // <savestr> x1 = 0x40d570
  40c820:       f90006a0        str     x0, [x21, #8]             // <= エラー時のspベースの[0xfffffffffb18]
  40c824:       17ffffea        b       40c7cc <vforkexec+0x12c>
  ```

## vforkから戻ったときのgdb sti

```
(gdb)
trapret () at kern/trapasm.S:86
86	    ldp     x29, x30, [sp], #16
(gdb)
trapret () at kern/trapasm.S:88
88	    eret
(gdb) si
0x000000000041bd94 in ?? ()         // bl      419f18 <__syscall_ret>
(gdb)
0x0000000000419f18 in ?? ()         // <__syscall_ret>
(gdb) i r sp
sp             0xfffffffffb10      0xfffffffffb10
...
(gdb)
0x0000000000419f2c in ?? ()         // ret
(gdb)
0x000000000041bd98 in ?? ()         // ldr     x30, [sp], #16
(gdb)
0x000000000041bd9c in ?? ()
```

```
<malloc>
<bin_chunk.part.0>
(gdb)
0x000000000041a548 in ?? ()         // adrp    x1, 433000 <__dso_handle>
(gdb)
0x000000000041a54c in ?? ()         // add     x1, x1, #0x700
(gdb)
0x000000000041a550 in ?? ()         // add     x0, x1, #0x4
(gdb)
0x000000000041a554 in ?? ()         // ldaxr   w0, [x0]

======= PROC[7]: /bin/dash TRAP FRAME DUMP =========
sp:    0x0000fffffffffb20  tpidr: 0x00000000004342f0
spsr:  0x0000000000000000  esr:   0x0000000002000000
elr:   0x0000fffffffffb20  x0:    0x0000000000000000
x1:    0x0000000000000000  x2:    0x0000000000000000
...
x27:   0x0000000000000001  x28:   0x0000000000000000
x29:   0x0000fffffffffb20  x30:   0x0000fffffffffb20
===================== DUMP END =====================
```

## ひどい回避策

fork()の最後に`print_mmap_list(np, "fork");`を実行するとエラーは収まるので当面これで行き、後でまた考える。
