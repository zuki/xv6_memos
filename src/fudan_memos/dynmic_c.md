```
struct dso {
#if DL_FDPIC
	struct fdpic_loadmap *loadmap;
#else
	unsigned char *base;                    // 0
#endif
	char *name;                             // 8
	size_t *dynv;                           // 16
	struct dso *next, *prev;                // 24

	Phdr *phdr;                             // 40
	int phnum;                              // 48   (4 pad)
	size_t phentsize;                       // 56
	Sym *syms;                              // 64
	Elf_Symndx *hashtab;                    // 72
	uint32_t *ghashtab;                     // 80
	int16_t *versym;                        // 88
	char *strings;                          // 96
	struct dso *syms_next, *lazy_next;      // 104
	size_t *lazy, lazy_cnt;                 // 120
	unsigned char *map;                     // 136
	size_t map_len;                         // 144
	dev_t dev;                              // 152
	ino_t ino;                              // 160
	char relocated;                         // 168
	char constructed;                       // 169
	char kernel_mapped;                     // 170
	char mark;
	char bfs_built;
	char runtime_loaded;
	struct dso **deps, *needed_by;
	size_t ndeps_direct;
	size_t next_dep;
	pthread_t ctor_visitor;
	char *rpath_orig, *rpath;
	struct tls_module tls;
	size_t tls_id;
	size_t relro_start, relro_end;
	uintptr_t *new_dtv;
	unsigned char *new_tls;
	struct td_index *td_index;
	struct dso *fini_next;
	char *shortname;
#if DL_FDPIC
	unsigned char *base;
#else
	struct fdpic_loadmap *loadmap;
#endif
	struct funcdesc {
		void *addr;
		size_t *got;
	} *funcdescs;
	size_t *got;
	char buf[];
};
```

```
typedef struct {
  Elf64_Word	p_type;         // 0
  Elf64_Word	p_flags;        // 4
  Elf64_Off	    p_offset;       // 8
  Elf64_Addr	p_vaddr;        // 16
  Elf64_Addr	p_paddr;        // 24
  Elf64_Xword	p_filesz;       // 32
  Elf64_Xword	p_memsz;        // 40
  Elf64_Xword	p_align;        // 48
} Elf64_Phdr;
```

```
00000000000619c0 <kernel_mapped_dso>:
   619c0:	f0000242 	adrp	x2, ac000 <debug+0x8>
   619c4:	b0000245 	adrp	x5, aa000 <malloc+0x92e40>
   619c8:	b980300a 	ldrsw	x10, [x0, #48]              // x10 = cnt
   619cc:	529caa4d 	mov	w13, #0xe552                	// w13 = 0xe552
   619d0:	b945804c 	ldr	w12, [x2, #1408]                // [ac580] => runtime
   619d4:	90000262 	adrp	x2, ad000 <std_name+0x6>
   619d8:	b94410a9 	ldr	w9, [x5, #1040]                 // [aa410] -> __default_stacksize
   619dc:	529caa2e 	mov	w14, #0xe551                	// w13 = 0xe551
   619e0:	f9404444 	ldr	x4, [x2, #136]                  // [ad088] => ?
   619e4:	aa0503e8 	mov	x8, x5                          // x8 = aa000 ?
   619e8:	f9400006 	ldr	x6, [x0]                        // x6 = p->base
   619ec:	cb0403e2 	neg	x2, x4                          // x2 = ~PAGE_SIZE, x4 = PAGESIZE
   619f0:	f9401401 	ldr	x1, [x0, #40]                   // x1 = ph
   619f4:	5280000b 	mov	w11, #0x0                   	// #0
   619f8:	d2800007 	mov	x7, #0x0                   	    // #0       x7 = max_addr
   619fc:	92800003 	mov	x3, #0xffffffffffffffff    	    // #-1      x3 = min_addr
   61a00:	72ac8e8d 	movk	w13, #0x6474, lsl #16       // w13 |= 0x64760000 => w13 = PT_GNU_RELRO(0x6474e552)
   61a04:	72ac8e8e 	movk	w14, #0x6474, lsl #16       // w14 |= 0x64760000 => w14 = PT_GNU_STACK(0x6474e551)
   61a08:	d2a0100f 	mov	x15, #0x800000              	// #8388608
L3: // for ( ; XX ; )
   61a0c:	d100054a 	sub	x10, x10, #0x1                  // x10 = cnt, cnt--
   61a10:	b100055f 	cmn	x10, #0x1                       // cnt == -1
   61a14:	540001a1 	b.ne	61a48 <kernel_mapped_dso+0x88>  // b.any =>L1
   61a18:	3400004b 	cbz	w11, 61a20 <kernel_mapped_dso+0x60> // if (w11 == 0) => L6
   61a1c:	b9041109 	str	w9, [x8, #1040]
L6: // for()() XX
   61a20:	d1000484 	sub	x4, x4, #0x1                // x4 = PAGE_SIZE - 1
   61a24:	8a030043 	and	x3, x2, x3                  // min_addr(x3) &= ~PAGE_SIZE(x2)
   61a28:	8b070084 	add	x4, x4, x7                  // max_addr(x4) = max_addr(x7) + PAGE_SIZE + 1 (x4)
   61a2c:	8b0300c6 	add	x6, x6, x3                  // map(x6) = base(x6) + min_addr(x3)
   61a30:	8a040042 	and	x2, x2, x4                  // max_addr(x2) = ~PAGE_SIZE(x2) & max_addr(x4)
   61a34:	52800021 	mov	w1, #0x1                   	// #1
   61a38:	cb030042 	sub	x2, x2, x3                  // max_len(x2) = max_addr(x2) - min_addr(x3)
   61a3c:	a9088806 	stp	x6, x2, [x0, #136]          // x6 => p->map, x2 => p->map_len
   61a40:	3902a801 	strb	w1, [x0, #170]          // p->kernel_mapped = 1
   61a44:	d65f03c0 	ret
L1: // for() { XX }
   61a48:	b9400025 	ldr	w5, [x1]                    // x5 = ph->p_type
   61a4c:	710008bf 	cmp	w5, #0x2                    // if (ph_p_type(w5) == PT_DYNAMIC(0x2))
   61a50:	540000e1 	b.ne	61a6c <kernel_mapped_dso+0xac>  // b.any    => L2
   61a54:	f9400825 	ldr	x5, [x1, #16]               // x5 = ph->p_vaddr
   61a58:	8b0500c5 	add	x5, x6, x5                  // p->dynv = p->base(x6) + ph->p_vaddr(x5)
   61a5c:	f9000805 	str	x5, [x0, #16]               // x5 => p->dynv
L5: // for (; ; XX)
   61a60:	f9401c05 	ldr	x5, [x0, #56]               // x5 = phentsize
   61a64:	8b050021 	add	x1, x1, x5                  // x1 = ph
   61a68:	17ffffe9 	b	61a0c <kernel_mapped_dso+0x4c>      // => L3
L2:
   61a6c:	6b0d00bf 	cmp	w5, w13                     // if (ph->p_type(w5) == PT_GNU_RELRO(w13))
   61a70:	54000121 	b.ne	61a94 <kernel_mapped_dso+0xd4>  // b.any    => L4
   61a74:	f9400825 	ldr	x5, [x1, #16]
   61a78:	8a0200b0 	and	x16, x5, x2
   61a7c:	f9009010 	str	x16, [x0, #288]
   61a80:	f9401430 	ldr	x16, [x1, #40]
   61a84:	8b1000a5 	add	x5, x5, x16
   61a88:	8a0200a5 	and	x5, x5, x2
   61a8c:	f9009405 	str	x5, [x0, #296]
   61a90:	17fffff4 	b	61a60 <kernel_mapped_dso+0xa0>
L4:                                                     // if (ph_p_type(w5) == PT_GNU_STACK(w14))
   61a94:	6b0e00bf 	cmp	w5, w14
   61a98:	54000121 	b.ne	61abc <kernel_mapped_dso+0xfc>  // b.any    => L7
   61a9c:	35fffe2c 	cbnz	w12, 61a60 <kernel_mapped_dso+0xa0> // if w12 != 0 => L5
   61aa0:	f9401425 	ldr	x5, [x1, #40]
   61aa4:	eb2940bf 	cmp	x5, w9, uxtw
   61aa8:	54fffdc9 	b.ls	61a60 <kernel_mapped_dso+0xa0>  // b.plast  => L5
   61aac:	f16000bf 	cmp	x5, #0x800, lsl #12
   61ab0:	5280002b 	mov	w11, #0x1                   	// #1
   61ab4:	9a8f90a9 	csel	x9, x5, x15, ls  // ls = plast
   61ab8:	17ffffea 	b	61a60 <kernel_mapped_dso+0xa0>      // => L5
L7:
   61abc:	710004bf 	cmp	w5, #0x1                    // if (ph_p_type(w5) == PT_LOAD(0x1))
   61ac0:	54fffd01 	b.ne	61a60 <kernel_mapped_dso+0xa0>  // continue: b.any => L5

   61ac4:	f9400825 	ldr	x5, [x1, #16]               // x5 <= ph->p_vaddr
   61ac8:	f9401430 	ldr	x16, [x1, #40]              // x16 <= ph->memsz
   61acc:	eb05007f 	cmp	x3, x5                      // if (min_addr(x3) - p_vaddr(x5))
   61ad0:	9a859063 	csel	x3, x3, x5, ls  // ls = plast   min_addr(x3) = p_vaddr
   61ad4:	8b1000a5 	add	x5, x5, x16                 // p_vaddr(x5) += p_memsz
   61ad8:	eb0500ff 	cmp	x7, x5                      // if (ph->p_vaddr+ph->p_memsz(x5) > max_addr(x7))
   61adc:	9a8520e7 	csel	x7, x7, x5, cs  // cs = hs, nlast max_addr(x7) = ph->p_vaddr+ph->p_memsz(x5)
   61ae0:	17ffffe0 	b	61a60 <kernel_mapped_dso+0xa0>      // => L5
```
