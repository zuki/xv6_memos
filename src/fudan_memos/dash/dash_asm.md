# dash

```
$ readelf -lw dash
Elf file type is EXEC (Executable file)
Entry point 0x4004a0
There are 4 program headers, starting at offset 64

Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  LOAD           0x000000 0x0000000000400000 0x0000000000400000 0x02fb70 0x02fb70 R E 0x1000
  LOAD           0x030940 0x0000000000431940 0x0000000000431940 0x0019e0 0x0037c0 RW  0x1000
  GNU_STACK      0x000000 0x0000000000000000 0x0000000000000000 0x000000 0x000000 RW  0x10
  GNU_RELRO      0x030940 0x0000000000431940 0x0000000000431940 0x0016c0 0x0016c0 R   0x1

 Section to Segment mapping:
  Segment Sections...
   00     .init .text .fini .rodata .eh_frame
   01     .init_array .fini_array .data.rel.ro .got .got.plt .data .bss
   02
   03     .init_array .fini_array .data.rel.ro .got .got.plt

$ readelf -SW dash
Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
  [ 1] .init             PROGBITS        0000000000400120 000120 000010 00  AX  0   0  4
  [ 2] .text             PROGBITS        0000000000400130 000130 028df0 00  AX  0   0 16
  [ 3] .fini             PROGBITS        0000000000428f20 028f20 000010 00  AX  0   0  4
  [ 4] .rodata           PROGBITS        0000000000428f30 028f30 002e8c 00   A  0   0 16
  [ 5] .eh_frame         PROGBITS        000000000042bdc0 02bdc0 003db0 00   A  0   0  8

  [ 6] .init_array       INIT_ARRAY      0000000000431940 030940 000008 08  WA  0   0  8
  [ 7] .fini_array       FINI_ARRAY      0000000000431948 030948 000008 08  WA  0   0  8
  [ 8] .data.rel.ro      PROGBITS        0000000000431950 030950 000d20 00  WA  0   0 16
  [ 9] .got              PROGBITS        0000000000432670 031670 000978 08  WA  0   0  8
  [10] .got.plt          PROGBITS        0000000000432fe8 031fe8 000018 08  WA  0   0  8
  [11] .data             PROGBITS        0000000000433000 032000 000320 00  WA  0   0 16
  [12] .bss              NOBITS          0000000000433320 032320 001de0 00  WA  0   0 16
  [13] .comment          PROGBITS        0000000000000000 032320 00002a 01  MS  0   0  1
  [14] .symtab           SYMTAB          0000000000000000 032350 009db0 18     15 1029  8
  [15] .strtab           STRTAB          0000000000000000 03c100 0029fe 00      0   0  1
  [16] .shstrtab         STRTAB          0000000000000000 03eafe 000086 00      0   0  1
```

# ELR_EL1 0x419f70

```
0000000000419f5c <wait4>:
  419f5c:       d2802088        mov     x8, #0x104                      // #260
  419f60:       93407c00        sxtw    x0, w0
  419f64:       93407c42        sxtw    x2, w2
  419f68:       f81f0ffe        str     x30, [sp, #-16]!
  419f6c:       d4000001        svc     #0x0
  419f70:       93407c00        sxtw    x0, w0                         // ELR_EL1 (wait4がsleepから起床した際に呼ばれる)
  419f74:       97ffffe3        bl      419f00 <__syscall_ret>
  419f78:       f84107fe        ldr     x30, [sp], #16
  419f7c:       d65f03c0        ret
```

## `FAR_EL1 0x40a640`

```
x29: 0000fffffffffa60
x30: 000000000040a964
x19: 000000000000000a
x20: 0000000000434ba8
x21: 0000000000436100
x22: 0000000000432000
x23: 0000000000000001
x24: 0000000000434b50
x25: 0000fffffffffd68
x26: 0000000000000000
sp : 000000000040b718

```

```
void
popfile(void)

  40a4e0:	a9bb7bfd 	stp	x29, x30, [sp, #-80]!
  40a4e4:	910003fd 	mov	x29, sp
  40a4e8:	a90153f3 	stp	x19, x20, [sp, #16]
//	struct parsefile *pf = parsefile;

//	INTOFF;
  40a4ec:	90000154 	adrp	x20, 432000 <signal_names+0xf0> 0x431f10 + 0xf0, data領域 432000 -> ファイルの0x31000
  40a4f0:	f945a681 	ldr	x1, [x20, #2888]  // x1 ~ [0x432000 + 0xb48]: => 434508

  40a4f4:	a90363f7 	stp	x23, x24, [sp, #48]
//	struct parsefile *pf = parsefile;
  40a4f8:	b0000158 	adrp	x24, 433000 <__dso_handle> // x24 (32000) + 0x32 => pf

  40a4fc:	a9046bf9 	stp	x25, x26, [sp, #64]
//	INTOFF;
  40a500:	b9400020 	ldr	w0, [x1]    // w0 = suppressiont
//	struct parsefile *pf = parsefile;
  40a504:	f940131a 	ldr	x26, [x24, #32]     // x26 = pf = 0x434950 (ファイルの0x32020)
//	INTOFF;
  40a508:	11000400 	add	w0, w0, #0x1 // suppressiont++
  40a50c:	b9000020 	str	w0, [x1]
//	if (pf->fd >= 0)
  40a510:	b9400f40 	ldr	w0, [x26, #12]      // w0 = pf->fd
  40a514:	37f80060 	tbnz	w0, #31, 40a520 <popfile+0x40> if (pf0->fd < 0) goto 40a520
//		close(pf->fd);
  40a518:	f947be81 	ldr	x1, [x20, #3960]        // <close> <= 0x420338 = [0x432000 + 0xf78]
  40a51c:	d63f0020 	blr	x1                      // close(pf->fd)
//	if (pf->buf)
  40a520:	f9401340 	ldr	x0, [x26, #32]      // w0 = pf->buf
  40a524:	b4000080 	cbz	x0, 40a534 <popfile+0x54>   // if (pf->buf == 0) goto 40a534
//		ckfree(pf->buf);
  40a528:	90000142 	adrp	x2, 432000 <signal_names+0xf0>
  40a52c:	f947e041 	ldr	x1, [x2, #4032]  // [0x432FC0] => 0x41a098 = <free>
  40a530:	d63f0020 	blr	x1               // free(pf->buf)
//	while (pf->strpush)
  40a534:	f9401740 	ldr	x0, [x26, #40]          // pf->strpush
  40a538:	b40008e0 	cbz	x0, 40a654 <popfile+0x174> // if (pf->strpush == 0) goto 40a654
	INTOFF;
  40a53c:	f945a699 	ldr	x25, [x20, #2888]  // x25 = 0x434508
  40a540:	a9025bf5 	stp	x21, x22, [sp, #32]
////			unalias(sp->ap->name);
  40a544:	f9437a97 	ldr	x23, [x20, #1776]  // x23 = 0x4005a8 = <unalias>
//			ckfree(sp->string);
  40a548:	f947e296 	ldr	x22, [x20, #4032]  // x22 = 0x41a098 = <free>
//			checkkwd |= CHKALIAS;
  40a54c:	f9469295 	ldr	x21, [x20, #3360]  // x21 = 0x434da4
  40a550:	1400001e 	b	40a5c8 <popfile+0xe8>
////	parsefile->nextc = sp->prevstring;
  40a554:	f9401300 	ldr	x0, [x24, #32]      // x0 = pf
//	parsefile->nleft = sp->prevnleft;
  40a558:	b9401263 	ldr	w3, [x19, #16]      // x19 = sp, w3 = sp->prevnleft
//	if (sp != &(parsefile->basestrpush))
  40a55c:	9100c001 	add	x1, x0, #0x30       // x1 = pf->basestrpush
  40a560:	eb01027f 	cmp	x19, x1
//	parsefile->unget = sp->unget;
  40a564:	b9403261 	ldr	w1, [x19, #48]      // w1 = sp->unget
//	parsefile->nextc = sp->prevstring;
  40a568:	f9400662 	ldr	x2, [x19, #8]       // x2 = sp->prevstring
//	parsefile->nleft = sp->prevnleft;
  40a56c:	b9001003 	str	w3, [x0, #16]       // pf->nleft = w3
//	parsefile->nextc = sp->prevstring;
  40a570:	f9000c02 	str	x2, [x0, #24]       // pf->nextc = x2
//	parsefile->unget = sp->unget;
  40a574:	b9007001 	str	w1, [x0, #112]      // sp->basestrrpush->unget = w1
//	memcpy(parsefile->lastc, sp->lastc, sizeof(sp->lastc));
  40a578:	f9401661 	ldr	x1, [x19, #40]      // x1 = s->
  40a57c:	f9003401 	str	x1, [x0, #104]
//	parsefile->strpush = sp->prev;
  40a580:	f9400261 	ldr	x1, [x19]
  40a584:	f9001401 	str	x1, [x0, #40]
//	if (sp != &(parsefile->basestrpush))
  40a588:	540000a0 	b.eq	40a59c <popfile+0xbc>  // b.none
//		ckfree(sp);
  40a58c:	90000142 	adrp	x2, 432000 <signal_names+0xf0>
  40a590:	aa1303e0 	mov	x0, x19
  40a594:	f947e041 	ldr	x1, [x2, #4032]
  40a598:	d63f0020 	blr	x1
//	INTON;
  40a59c:	f945a681 	ldr	x1, [x20, #2888]
  40a5a0:	90000142 	adrp	x2, 432000 <signal_names+0xf0>
  40a5a4:	b9400020 	ldr	w0, [x1]
  40a5a8:	51000400 	sub	w0, w0, #0x1
  40a5ac:	b9000020 	str	w0, [x1]
  40a5b0:	35000080 	cbnz	w0, 40a5c0 <popfile+0xe0>
  40a5b4:	f9434e80 	ldr	x0, [x20, #1688]    // x0 = 0x41f028 = <__stpcpy>
  40a5b8:	b9400000 	ldr	w0, [x0]
  40a5bc:	35000400 	cbnz	w0, 40a63c <popfile+0x15c>  // TODO
//	while (pf->strpush)
  40a5c0:	f9401740 	ldr	x0, [x26, #40]
  40a5c4:	b4000460 	cbz	x0, 40a650 <popfile+0x170>
//	struct strpush *sp = parsefile->strpush;
  40a5c8:	f9401301 	ldr	x1, [x24, #32]
//	INTOFF;
  40a5cc:	b9400320 	ldr	w0, [x25]
  40a5d0:	11000400 	add	w0, w0, #0x1
  40a5d4:	b9000320 	str	w0, [x25]
//	struct strpush *sp = parsefile->strpush;
  40a5d8:	f9401433 	ldr	x19, [x1, #40]      // x19 = pf->strpush
//	if (sp->ap) {
  40a5dc:	f9400e61 	ldr	x1, [x19, #24]      // x1 = pf->strpush->ap
  40a5e0:	b4fffba1 	cbz	x1, 40a554 <popfile+0x74>
//		if (parsefile->nextc[-1] == ' ' ||
  40a5e4:	f9401300 	ldr	x0, [x24, #32]
  40a5e8:	f9400c00 	ldr	x0, [x0, #24]
  40a5ec:	385ff000 	ldurb	w0, [x0, #-1]
  40a5f0:	7100801f 	cmp	w0, #0x20
  40a5f4:	7a491804 	ccmp	w0, #0x9, #0x4, ne  // ne = any
  40a5f8:	54000081 	b.ne	40a608 <popfile+0x128>  // b.any
			checkkwd |= CHKALIAS;
  40a5fc:	b94002a0 	ldr	w0, [x21]
  40a600:	32000000 	orr	w0, w0, #0x1
  40a604:	b90002a0 	str	w0, [x21]
//		if (sp->string != sp->ap->val) {
  40a608:	f9400822 	ldr	x2, [x1, #16]
  40a60c:	f9401260 	ldr	x0, [x19, #32]
  40a610:	eb02001f 	cmp	x0, x2
  40a614:	54000060 	b.eq	40a620 <popfile+0x140>  // b.none
			ckfree(sp->string);
  40a618:	d63f02c0 	blr	x22
  40a61c:	f9400e61 	ldr	x1, [x19, #24]      // x1 = pf->strpush->ap
		sp->ap->flag &= ~ALIASINUSE;
  40a620:	b9401820 	ldr	w0, [x1, #24]
  40a624:	121f7802 	and	w2, w0, #0xfffffffe // w0 = sp->ap->flag & ~ALIASINUSE
  40a628:	b9001822 	str	w2, [x1, #24]       // w2 sp->ap->flag
//		if (sp->ap->flag & ALIASDEAD) {
  40a62c:	360ff940 	tbz	w0, #1, 40a554 <popfile+0x74> // sp->ap->flag & ALIASDEAD
//			unalias(sp->ap->name);
  40a630:	f9400420 	ldr	x0, [x1, #8]        // x0 = sp->ap->name
  40a634:	d63f02e0 	blr	x23                 // x23 = <unalias>
  40a638:	17ffffc7 	b	40a554 <popfile+0x74>
//	INTON;
  40a63c:	f943d840 	ldr	x0, [x2, #1968]     // x0 = 0x402930 = <onint>
  40a640:	d63f0000 	blr	x0                  // onint(void)
	while (pf->strpush)
  40a644:	f9401740 	ldr	x0, [x26, #40]
  40a648:	b5fffc00 	cbnz	x0, 40a5c8 <popfile+0xe8>
  40a64c:	d503201f 	nop
  40a650:	a9425bf5 	ldp	x21, x22, [sp, #32]
//		popstring();
//	parsefile = pf->prev;
//	ckfree(pf);
  40a654:	90000142 	adrp	x2, 432000 <signal_names+0xf0>
//	parsefile = pf->prev;
  40a658:	f9400340 	ldr	x0, [x26]
  40a65c:	f9001300 	str	x0, [x24, #32]
//	ckfree(pf);
  40a660:	f947e041 	ldr	x1, [x2, #4032]
  40a664:	aa1a03e0 	mov	x0, x26
  40a668:	d63f0020 	blr	x1
//	INTON;
  40a66c:	f945a681 	ldr	x1, [x20, #2888]
  40a670:	b9400020 	ldr	w0, [x1]
  40a674:	51000400 	sub	w0, w0, #0x1
  40a678:	b9000020 	str	w0, [x1]
  40a67c:	35000080 	cbnz	w0, 40a68c <popfile+0x1ac>
  40a680:	f9434e94 	ldr	x20, [x20, #1688]
  40a684:	b9400280 	ldr	w0, [x20]
  40a688:	350000c0 	cbnz	w0, 40a6a0 <popfile+0x1c0>

  40a68c:	a94153f3 	ldp	x19, x20, [sp, #16]
  40a690:	a94363f7 	ldp	x23, x24, [sp, #48]
  40a694:	a9446bf9 	ldp	x25, x26, [sp, #64]
  40a698:	a8c57bfd 	ldp	x29, x30, [sp], #80
  40a69c:	d65f03c0 	ret
//	INTON;
  40a6a0:	90000142 	adrp	x2, 432000 <signal_names+0xf0>

  40a6a4:	a94153f3 	ldp	x19, x20, [sp, #16]
  40a6a8:	a94363f7 	ldp	x23, x24, [sp, #48]
  40a6ac:	a9446bf9 	ldp	x25, x26, [sp, #64]
  40a6b0:	a8c57bfd 	ldp	x29, x30, [sp], #80
//	INTON;
  40a6b4:	f943d850 	ldr	x16, [x2, #1968]
  40a6b8:	d61f0200 	br	x16
  40a6bc:	d503201f 	nop
```
