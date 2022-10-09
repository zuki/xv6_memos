# mmaptest2

```
[F-05] file backed private mapping permission test
sys_mmap: addr=0x0, size=0xc8, prot=0x1, flags=0x2, fd=3, offset=0x0
- ret=0x0x60000000
sys_mmap: addr=0x0, size=0xc8, prot=0x3, flags=0x2, fd=3, offset=0x0
- ret2=0x0x60001000
pagefault_handler: va=0x60001000
mmap_store_data: addr=0x60001000, size=0xc8, flags=0x2, prot=0x3, offset=0x0
map_pagecache_page_util
- tempsize=0xc8, curoff=0x0, cursize=0xc8, ip-size=0x0
copy_page: ip=0xffff0000008c5f78, offset=, inum=25, dev=1, dest=0xffff00003afb4000, size=0xc8, dest_offset=0
get_page
- alloc new page
- page=0xffff0000008c5510
map_pagecache_page_util
- tempsize=0x1000, curoff=0x1000, cursize=0x0, ip-size=0x0
map_pagecache_page_util
- tempsize=0x1000, curoff=0x0, cursize=0x1000, ip-size=0x0
copy_page: ip=0xffff0000008c5f78, offset=, inum=25, dev=1, dest=0xffff000039f64000, size=0x1000, dest_offset=0
get_page
- readi failed
- page=0xffffffffffffffff
```


```
[F-05] file backed private mapping permission test
sys_mmap: addr=0x0, size=0xc8, prot=0x1, flags=0x2, fd=3, offset=0x0
- ret=0x0x60000000
sys_mmap: addr=0x0, size=0xc8, prot=0x3, flags=0x2, fd=3, offset=0x0
- ret2=0x0x60001000
pagefault_handler: va=0x60001000
mmap_store_data: addr=0x60001000, size=0xc8, flags=0x2, prot=0x3, offset=0x0
- map_pagecache_page_util
  - tempsize=0xc8, curoff=0x0, cursize=0xc8, ip-size=0x0
- copy_page: offset=0, inum=25, dev=1, dest=0xffff00003afb4000, size=0xc8
  - get_page
   - alloc new page: page[0]=0x0
  - page=0xffff0000008c5510
  - map_region: va=0x60001000, temp=0x3afb4000
pagefault_handler: mmap_store_data failed
data abort: insruction 0x4060f4, fault addr 0xffffffffffffffff, dfs=4
```

```c
tatic int map_pagecache_page(struct proc *p, struct file *f, uint64_t vmaaddr,
                              int prot, int offset, uint64_t main_size)
{
    uint64_t size = main_size;
    for (uint64_t cursize = 0; cursize < main_size; cursize += PGSIZE) {
        uint64_t mapsize = PGSIZE > size ? size : PGSIZE;
        if (map_pagecache_page_util(p, f, vmaaddr + cursize, prot, offset + cursize, mapsize) < 0)
            return -1;
        size -= PGSIZE;     // これがバグ。正しくは size -= mapsize。
    }
    return size;
}
```

- バグ発見で[F-05]は正常終了

```
[F-05] file backed private mapping permission test
sys_mmap: addr=0x0, size=0xc8, prot=0x1, flags=0x2, fd=3, offset=0x0
- ret=0x0x60000000
sys_mmap: addr=0x0, size=0xc8, prot=0x3, flags=0x2, fd=3, offset=0x0
- ret2=0x0x60001000
pagefault_handler: va=0x60001000
mmap_store_data: addr=0x60001000, size=0xc8, flags=0x2, prot=0x3, offset=0x0
- map_pagecache_page_util
  - tempsize=0xc8, curoff=0x0, cursize=0xc8, ip-size=0x0
- copy_page: offset=0, inum=25, dev=1, dest=0xffff00003afb4000, size=0xc8
  - get_page
   - alloc new page: page[0]=0x0
  - page=0xffff0000008c5510
  - map_region: va=0x60001000, temp=0x3afb4000
pagefault_handler ok
- set rets[0-20]='a'
sys_munmap: addr=0x60000000, size=0xc8
sys_munmap: addr=0x60001000, size=0xc8
file backed private mapping permission test ok
file_test: ok: 5, nk: 0
```

# F-08:

- 200個の'a'をwriteすると、最初の16文字がnullとなる。17文字以降は'a'が書き込まれる。
- しかし、32個の'a'をwriteした場合は、32個の'a'が書き込まれる。
- 97, 98, 100, 104, 128, 256は最初の16個がnull
- 96は最初の64個がnull
- 88は最初の56個がnull
- 80は最初の48個がnull
- 72は最初の40個がnull
- 64は最初の32個がnull
- 56は最初の24個がnull
- 48は最初の16個がnull
- 40は最初の8個がnull
- 34は最初の2個がnull
- 33は最初の1個がnull
- 16はok

```
inode
04120680: 0200 0000 0000 0100 c800 0000 e302 0000  ................     # nlink=1, size=c8 (200)
04120690: 0000 0000 0000 0000 0000 0000 0000 0000  ................     # addr = 0x0410_0000 + 0x2e3 * 0x1000
041206a0: 0000 0000 0000 0000 0000 0000 0000 0000  ................     #      = 0x043e_3000
041206b0: 0000 0000 0000 0000 0000 0000 0000 0000  ................

data: 200個の場合
043e3000: 0000 0000 0000 0000 0000 0000 0000 0000  ................
043e3010: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e3020: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e3030: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e3040: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e3050: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e3060: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e3070: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e3080: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e3090: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e30a0: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e30b0: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e30c0: 6161 6161 6161 6161 0000 0000 0000 0000  aaaaaaaa........

data: 32個の場合
043e3000: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e3010: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
043e3020: 0000 0000 0000 0000 0000 0000 0000 0000  ................

```

```
[F-08] file mmap on empty file test
sys_mmap: addr=0x0, size=0xc8, prot=0x3, flags=0x1, fd=3, offset=0x0
- addr=0x60000000
ret(0x0x60000000): ................aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
sys_munmap: addr=0x60000000, size=0xc8
filewrite: inum=25, addr=0x60000000, size=0xc8, off=0x0
- update_page: addr=0x60000000, size=0xc8, offset=0x0
  - aligned_offset=0x0
  - fount page 0
    - page_offset=0x0, addr=0x60000000, size=0xc8
file mmap on empty file test failed: at buf[0]

file_test: ok: 0, ng: 1
```

## 200バイトを分割して設定する

- 32バイトずつ7回に分けて200バイト設定したら問題なく設定できる
- 100バイトずつ2回の場合は、先頭16バイトだけが0。
- 最初に0から32バイト設定し、さらに0から200バイトを登録しても、先頭16バイトだけ0.
-

```
[F-08] file mmap on empty file test
sys_mmap: addr=0x0, size=0xc8, prot=0x3, flags=0x1, fd=3, offset=0x0
- addr=0x60000000
ret(0x0x60000000): aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
sys_munmap: addr=0x60000000, size=0xc8
filewrite: inum=25, addr=0x60000000, size=0xc8, off=0x0
- update_page: addr=0x60000000, size=0xc8, offset=0x0
  - aligned_offset=0x0
  - fount page 0
    - page_offset=0x0, addr=0x60000000, size=0xc8
file mmap on empty file test ok

04104000: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
04104010: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
04104020: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
04104030: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
04104040: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
04104050: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
04104060: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
04104070: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
04104080: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
04104090: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
041040a0: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
041040b0: 6161 6161 6161 6161 6161 6161 6161 6161  aaaaaaaaaaaaaaaa
041040c0: 6161 6161 6161 6161 0000 0000 0000 0000  aaaaaaaa........
```

## 最適化オプションの違いによる変化

1. `-O0`: 先頭に１個、nullが

```
ret(0x0x60000000): .aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

2. `-O1`, `-O2`: 全部null

```
ret(0x0x60000000): ........................................................................................................................................................................................................
```

3. `-O3`: これまでのオプション

```
ret(0x0x60000000): ................aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

## `file_empty_file_size_test`

```
00000000004015c8 <file_empty_file_size_test>:
  4015c8:	a9ae7bfd 	stp	x29, x30, [sp, #-288]!
  4015cc:	b0000040 	adrp	x0, 40a000 <__fixunstfsi+0x68>
  4015d0:	913b2000 	add	x0, x0, #0xec8  // x0 = "\n[F-08]..."
  4015d4:	910003fd 	mov	x29, sp
  4015d8:	a90363f7 	stp	x23, x24, [sp, #48]
  4015dc:	d0000077 	adrp	x23, 40f000 <__FRAME_END__+0xfd4> // x23 = 40f000
  4015e0:	f9478ae1 	ldr	x1, [x23, #3856]  // 0x406758 = [0xf0ff10 = 40f000 + 0xf10]
  4015e4:	a90153f3 	stp	x19, x20, [sp, #16]
  4015e8:	f94792f3 	ldr	x19, [x23, #3872] // 0xfff1000400000236 = [40ff20]
  4015ec:	f9400022 	ldr	x2, [x1]  // x2 = 0xaa0003f3a9be53f3
  4015f0:	f9008fe2 	str	x2, [sp, #280]
  4015f4:	d2800002 	mov	x2, #0x0                   	// #0
  4015f8:	a9046bf9 	stp	x25, x26, [sp, #64]
  4015fc:	b000005a 	adrp	x26, 40a000 <__fixunstfsi+0x68>
  401600:	d63f0260 	blr	x19
  401604:	f947eae2 	ldr	x2, [x23, #4048]            // x2 = 0x4042bc[48d80]
  401608:	913bc340 	add	x0, x26, #0xef0             // "file1"
  40160c:	52800841 	mov	w1, #0x42                  	// #66 (O_CREAT|O_RDWR)
  401610:	d63f0040 	blr	x2                          // open("file1", mode)
  401614:	3100041f 	cmn	w0, #0x1                    // if (fd == -1)
  401618:	54000d80 	b.eq	4017c8 <file_empty_file_size_test+0x200>  // b.none
  40161c:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  401620:	2a0003e4 	mov	w4, w0                      // fd
  401624:	2a0003f9 	mov	w25, w0                     // w25 = fd
  401628:	d2800005 	mov	x5, #0x0                   	// #0
  40162c:	f947a0c6 	ldr	x6, [x6, #3904]             // mmap
  401630:	52800023 	mov	w3, #0x1                   	// #1 = MAP_SHARED
  401634:	52800062 	mov	w2, #0x3                   	// #3 = PROT_READ|WRITE
  401638:	d2801901 	mov	x1, #0xc8                  	// #200
  40163c:	d2800000 	mov	x0, #0x0                   	// #0
  401640:	d63f00c0 	blr	x6                          // calll mmap()
  401644:	aa0003f8 	mov	x24, x0                     // x24 = ret
  401648:	b100041f 	cmn	x0, #0x1                    // if (ret == -1)
  40164c:	54000f00 	b.eq	40182c <file_empty_file_size_test+0x264>  // b.none
  401650:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4> // x6 = 0x40f00
  401654:	a9025bf5 	stp	x21, x22, [sp, #32]
  401658:	d2801902 	mov	x2, #0xc8                  	// #200
  40165c:	f947d0c3 	ldr	x3, [x6, #4000]             // x3 = 0x4_0000_0000_002a = [0x40faa0]
  401660:	52800c21 	mov	w1, #0x61                  	// #97 = 'a'
  401664:	aa1803f3 	mov	x19, x24                    // x19 = ret
  401668:	91032314 	add	x20, x24, #0xc8             // x20 = ret[200]
  40166c:	d63f0060 	blr	x3
  401670:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  401674:	aa1803e1 	mov	x1, x24                     // x1 = ret
  401678:	b0000040 	adrp	x0, 40a000 <__fixunstfsi+0x68>
  40167c:	913d6000 	add	x0, x0, #0xf58              // x0 = 0x40af58
  401680:	f94780c2 	ldr	x2, [x6, #3840]             // x2 = 0 = [0x40ff00]
  401684:	d63f0040 	blr	x2
  401688:	f9477ef5 	ldr	x21, [x23, #3832]           // x21 = 0xfff10004000002c2 = [0x40fef8]
  40168c:	aa1503f6 	mov	x22, x21
  401690:	39400260 	ldrb	w0, [x19]                 // w0 = [ret]
  401694:	91000673 	add	x19, x19, #0x1              // ret = ret + 1
  401698:	51008001 	sub	w1, w0, #0x20               // w1 = c - 0x20
  40169c:	12001c21 	and	w1, w1, #0xff               // w1 = w1 & 0xff
  4016a0:	7101783f 	cmp	w1, #0x5e                   // c == 0x5e
  4016a4:	54000888 	b.hi	4017b4 <file_empty_file_size_test+0x1ec>  // b.pmore
  4016a8:	d63f02c0 	blr	x22
  4016ac:	eb13029f 	cmp	x20, x19
  4016b0:	54ffff01 	b.ne	401690 <file_empty_file_size_test+0xc8>  // b.any
  4016b4:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  4016b8:	52800140 	mov	w0, #0xa                   	// #10
  4016bc:	f9477cc1 	ldr	x1, [x6, #3832]
  4016c0:	d63f0020 	blr	x1
  4016c4:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  4016c8:	aa1803e0 	mov	x0, x24
  4016cc:	d2801901 	mov	x1, #0xc8                  	// #200
  4016d0:	f94784c2 	ldr	x2, [x6, #3848]
  4016d4:	d63f0040 	blr	x2                          // munmpa(ret, 200)
  4016d8:	3100041f 	cmn	w0, #0x1
  4016dc:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  4016e0:	54000a00 	b.eq	401820 <file_empty_file_size_test+0x258>  // b.none
  4016e4:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  4016e8:	2a1903e0 	mov	w0, w25
  4016ec:	f947ecc1 	ldr	x1, [x6, #4056]
  4016f0:	d63f0020 	blr	x1
  4016f4:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  4016f8:	913bc340 	add	x0, x26, #0xef0
  4016fc:	52800001 	mov	w1, #0x0                   	// #0
  401700:	f947e8c2 	ldr	x2, [x6, #4048]
  401704:	d63f0040 	blr	x2
  401708:	2a0003f4 	mov	w20, w0
  40170c:	3100041f 	cmn	w0, #0x1
  401710:	540006c0 	b.eq	4017e8 <file_empty_file_size_test+0x220>  // b.none
  401714:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  401718:	910143f3 	add	x19, sp, #0x50
  40171c:	aa1303e1 	mov	x1, x19
  401720:	d2801902 	mov	x2, #0xc8                  	// #200
  401724:	f947bcc3 	ldr	x3, [x6, #3960]
  401728:	d63f0060 	blr	x3
  40172c:	7103201f 	cmp	w0, #0xc8
  401730:	aa1303e2 	mov	x2, x19
  401734:	52800001 	mov	w1, #0x0                   	// #0
  401738:	540000c0 	b.eq	401750 <file_empty_file_size_test+0x188>  // b.none
  40173c:	14000036 	b	401814 <file_empty_file_size_test+0x24c>
  401740:	11000421 	add	w1, w1, #0x1
  401744:	91000442 	add	x2, x2, #0x1
  401748:	7103203f 	cmp	w1, #0xc8
  40174c:	54000760 	b.eq	401838 <file_empty_file_size_test+0x270>  // b.none
  401750:	39400040 	ldrb	w0, [x2]
  401754:	7101841f 	cmp	w0, #0x61
  401758:	54ffff40 	b.eq	401740 <file_empty_file_size_test+0x178>  // b.none
  40175c:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  401760:	b0000040 	adrp	x0, 40a000 <__fixunstfsi+0x68>
  401764:	913f0000 	add	x0, x0, #0xfc0
  401768:	f94780c2 	ldr	x2, [x6, #3840]
  40176c:	d63f0040 	blr	x2
  401770:	f0000062 	adrp	x2, 410000 <__dso_handle>
  401774:	a9425bf5 	ldp	x21, x22, [sp, #32]
  401778:	b9411440 	ldr	w0, [x2, #276]
  40177c:	11000400 	add	w0, w0, #0x1
  401780:	b9011440 	str	w0, [x2, #276]
  401784:	f9478af7 	ldr	x23, [x23, #3856]
  401788:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  40178c:	f9408fe0 	ldr	x0, [sp, #280]
  401790:	f94002e1 	ldr	x1, [x23]
  401794:	eb010000 	subs	x0, x0, x1
  401798:	d2800001 	mov	x1, #0x0                   	// #0
  40179c:	540006e1 	b.ne	401878 <file_empty_file_size_test+0x2b0>  // b.any
  4017a0:	a94153f3 	ldp	x19, x20, [sp, #16]
  4017a4:	a94363f7 	ldp	x23, x24, [sp, #48]
  4017a8:	a9446bf9 	ldp	x25, x26, [sp, #64]
  4017ac:	a8d27bfd 	ldp	x29, x30, [sp], #288
  4017b0:	d65f03c0 	ret
  4017b4:	528005c0 	mov	w0, #0x2e                  	// #46
  4017b8:	d63f02a0 	blr	x21
  4017bc:	eb13029f 	cmp	x20, x19
  4017c0:	54fff681 	b.ne	401690 <file_empty_file_size_test+0xc8>  // b.any
  4017c4:	17ffffbc 	b	4016b4 <file_empty_file_size_test+0xec>
  4017c8:	b0000040 	adrp	x0, 40a000 <__fixunstfsi+0x68>
  4017cc:	913be000 	add	x0, x0, #0xef8
  4017d0:	d63f0260 	blr	x19
  4017d4:	f0000061 	adrp	x1, 410000 <__dso_handle>
  4017d8:	b9411420 	ldr	w0, [x1, #276]
  4017dc:	11000400 	add	w0, w0, #0x1
  4017e0:	b9011420 	str	w0, [x1, #276]
  4017e4:	17ffffe8 	b	401784 <file_empty_file_size_test+0x1bc>
  4017e8:	b0000040 	adrp	x0, 40a000 <__fixunstfsi+0x68>
  4017ec:	913be000 	add	x0, x0, #0xef8
  4017f0:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  4017f4:	f94790c1 	ldr	x1, [x6, #3872]
  4017f8:	d63f0020 	blr	x1
  4017fc:	f0000061 	adrp	x1, 410000 <__dso_handle>
  401800:	a9425bf5 	ldp	x21, x22, [sp, #32]
  401804:	b9411420 	ldr	w0, [x1, #276]
  401808:	11000400 	add	w0, w0, #0x1
  40180c:	b9011420 	str	w0, [x1, #276]
  401810:	17ffffdd 	b	401784 <file_empty_file_size_test+0x1bc>
  401814:	b0000040 	adrp	x0, 40a000 <__fixunstfsi+0x68>
  401818:	913e4000 	add	x0, x0, #0xf90
  40181c:	17fffff5 	b	4017f0 <file_empty_file_size_test+0x228>
  401820:	b0000040 	adrp	x0, 40a000 <__fixunstfsi+0x68>
  401824:	913da000 	add	x0, x0, #0xf68
  401828:	17fffff3 	b	4017f4 <file_empty_file_size_test+0x22c>
  40182c:	b0000040 	adrp	x0, 40a000 <__fixunstfsi+0x68>
  401830:	913ca000 	add	x0, x0, #0xf28
  401834:	17ffffe7 	b	4017d0 <file_empty_file_size_test+0x208>
  401838:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  40183c:	b0000040 	adrp	x0, 40a000 <__fixunstfsi+0x68>
  401840:	913fe000 	add	x0, x0, #0xff8
  401844:	f94790c1 	ldr	x1, [x6, #3872]
  401848:	d63f0020 	blr	x1
  40184c:	d0000066 	adrp	x6, 40f000 <__FRAME_END__+0xfd4>
  401850:	2a1403e0 	mov	w0, w20
  401854:	f947ecc1 	ldr	x1, [x6, #4056]
  401858:	d63f0020 	blr	x1
  40185c:	f0000060 	adrp	x0, 410000 <__dso_handle>
  401860:	91045000 	add	x0, x0, #0x114
  401864:	a9425bf5 	ldp	x21, x22, [sp, #32]
  401868:	b9400401 	ldr	w1, [x0, #4]
  40186c:	11000421 	add	w1, w1, #0x1
  401870:	b9000401 	str	w1, [x0, #4]
  401874:	17ffffc4 	b	401784 <file_empty_file_size_test+0x1bc>
  401878:	f947a4c0 	ldr	x0, [x6, #3912]
  40187c:	a9025bf5 	stp	x21, x22, [sp, #32]
  401880:	d63f0000 	blr	x0
  401884:	d503201f 	nop
```

```
0000aec0: 7465 7374 206f 6b00 0a5b 462d 3038 5d20  test ok..[F-08]
0000aed0: 6669 6c65 206d 6d61 7020 6f6e 2065 6d70  file mmap on emp
0000aee0: 7479 2066 696c 6520 7465 7374 0000 0000  ty file test....
0000aef0: 6669 6c65 3100 0000 6669 6c65 206d 6d61  file1...file mma
0000af00: 7020 6f6e 2065 6d70 7479 2066 696c 6520  p on empty file
0000af10: 7465 7374 2066 6169 6c65 643a 2061 7420  test failed: at
0000af20: 6f70 656e 0000 0000 6669 6c65 206d 6d61  open....file mma
0000af30: 7020 6f6e 2065 6d70 7479 2066 696c 6520  p on empty file
0000af40: 7465 7374 2066 6169 6c65 643a 2061 7420  test failed: at
0000af50: 6d6d 6170 0000 0000 7265 7428 3078 2570  mmap....ret(0x%p
0000af60: 293a 2000 0000 0000 6669 6c65 206d 6d61  ): .....file mma
0000af70: 7020 6f6e 2065 6d70 7479 2066 696c 6520  p on empty file
0000af80: 7465 7374 2066 6169 6c65 6400 0000 0000  test failed.....
0000af90: 6669 6c65 206d 6d61 7020 6f6e 2065 6d70  file mmap on emp
0000afa0: 7479 2066 696c 6520 7465 7374 2066 6169  ty file test fai
0000afb0: 6c65 643a 2061 7420 7265 6164 0000 0000  led: at read....
0000afc0: 6669 6c65 206d 6d61 7020 6f6e 2065 6d70  file mmap on emp
0000afd0: 7479 2066 696c 6520 7465 7374 2066 6169  ty file test fai
0000afe0: 6c65 643a 2061 7420 6275 665b 2564 5d0a  led: at buf[%d].
0000aff0: 0000 0000 0000 0000 6669 6c65 206d 6d61  ........file mma
0000b000: 7020 6f6e 2065 6d70 7479 2066 696c 6520  p on empty file
0000b010: 7465 7374 206f 6b00 0a66 696c 655f 7465  test ok..file_te
```

## F-11, F-14など、mmap()呼び出し時にoffsetが0でない場合、sys_mmapが呼ばれない

- libcの方でoffsetが4KBアラインに制限されていた。
- コードは以下の通りだが、`UNIT=4096`で`OFF_MASK=0xfff`となるので、offは4KBアラインで
  なければならない。
- man(2) mmapには"offset must be a multiple of the page size as returned by sysconf
  (_SC_PAGE_SIZE)"とあるので、これはテストコードの間違い。

```
#define UNIT SYSCALL_MMAP2_UNIT
#define OFF_MASK ((-0x2000ULL << (8*sizeof(syscall_arg_t)-1)) | (UNIT-1))

void *__mmap(void *start, size_t len, int prot, int flags, int fd, off_t off)
{
        long ret;
        if (off & OFF_MASK) {
                errno = EINVAL;
                return MAP_FAILED;
        }
```

# `file_tests()`の21テストがすべてok

```
file_test: ok: 21, ng: 0
```

## 本日の知見

1. システムコールを呼ぶ前にlibcではねられる場合がある
2. スタック領域不足でpagefaultが生じる（これ用にハンドラを要修正）

# すべてOK

```
$ mmaptest2
[F-01] file backed mapping invalid file descriptor test
sys_mmap: invalid fd
sys_mmap: invalid fd
sys_mmap: invalid fd
file backed mapping invalid file descriptor test ok

[F-02] file backed mapping invalid flags test
mmap: invalid flags
sys_mmap: mapped=0xffffffffffffffff <= addr=0x0, size=0x64, prot=0x3, flags=0x0, fd=3, offset=0x0
file backed mapping invalid flags test ok

[F-03] file backed writeable shared mapping on read only file test
mmap: file not writable
sys_mmap: mapped=0xffffffffffffffff <= addr=0x0, size=0xc8, prot=0x3, flags=0x1, fd=3, offset=0x0
file backed writeable shared mapping on read only file test ok

[F-04] file backed read only shared mapping on read only file test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0xc8, prot=0x1, flags=0x1, fd=3, offset=0x0
sys_munmap: addr=0x60000000, size=0xc8
file backed read only shared mapping on read only file test ok

[F-05] file backed private mapping permission test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0xc8, prot=0x1, flags=0x2, fd=3, offset=0x0
- ret=0x0x60000000
sys_mmap: mapped=0x60001000 <= addr=0x0, size=0xc8, prot=0x3, flags=0x2, fd=3, offset=0x0
- ret2=0x0x60001000
   - alloc new cached page [0]
- set rets[0-20]='a'
sys_munmap: addr=0x60000000, size=0xc8
sys_munmap: addr=0x60001000, size=0xc8
file backed private mapping permission test ok

[F-06] file backed exceed mapping size test
find_vma_addr: exeed MMAPTOP
mmap: vma not available
sys_mmap: mapped=0xffffffffffffffff <= addr=0x0, size=0x25800000, prot=0x3, flags=0x22, fd=3, offset=0x0
file backed exceed mapping size test ok

[F-07] file backed exceed mapping count test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60002000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60004000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60006000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60008000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6000a000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6000c000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6000e000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60010000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60012000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60014000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60016000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60018000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6001a000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6001c000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6001e000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60020000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60022000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60024000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60026000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60028000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6002a000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6002c000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6002e000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60030000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60032000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60034000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60036000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x60038000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6003a000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6003c000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_mmap: mapped=0x6003e000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
mmap: vmas exhausted
sys_mmap: mapped=0xffffffffffffffff <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=3, offset=0x0
sys_munmap: addr=0x60000000, size=0x13e8
sys_munmap: addr=0x60002000, size=0x13e8
sys_munmap: addr=0x60004000, size=0x13e8
sys_munmap: addr=0x60006000, size=0x13e8
sys_munmap: addr=0x60008000, size=0x13e8
sys_munmap: addr=0x6000a000, size=0x13e8
sys_munmap: addr=0x6000c000, size=0x13e8
sys_munmap: addr=0x6000e000, size=0x13e8
sys_munmap: addr=0x60010000, size=0x13e8
sys_munmap: addr=0x60012000, size=0x13e8
sys_munmap: addr=0x60014000, size=0x13e8
sys_munmap: addr=0x60016000, size=0x13e8
sys_munmap: addr=0x60018000, size=0x13e8
sys_munmap: addr=0x6001a000, size=0x13e8
sys_munmap: addr=0x6001c000, size=0x13e8
sys_munmap: addr=0x6001e000, size=0x13e8
sys_munmap: addr=0x60020000, size=0x13e8
sys_munmap: addr=0x60022000, size=0x13e8
sys_munmap: addr=0x60024000, size=0x13e8
sys_munmap: addr=0x60026000, size=0x13e8
sys_munmap: addr=0x60028000, size=0x13e8
sys_munmap: addr=0x6002a000, size=0x13e8
sys_munmap: addr=0x6002c000, size=0x13e8
sys_munmap: addr=0x6002e000, size=0x13e8
sys_munmap: addr=0x60030000, size=0x13e8
sys_munmap: addr=0x60032000, size=0x13e8
sys_munmap: addr=0x60034000, size=0x13e8
sys_munmap: addr=0x60036000, size=0x13e8
sys_munmap: addr=0x60038000, size=0x13e8
sys_munmap: addr=0x6003a000, size=0x13e8
sys_munmap: addr=0x6003c000, size=0x13e8
sys_munmap: addr=0x6003e000, size=0x13e8
file backed exceed mapping count test ok

[F-08] file mmap on empty file test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x20, prot=0x3, flags=0x1, fd=3, offset=0x0
   - alloc new cached page [1]
ret(0x0x60000000): aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
sys_munmap: addr=0x60000000, size=0x20
filewrite: inum=26, addr=0x60000000, size=0x20, off=0x0
- update_page: addr=0x60000000, size=0x20, offset=0x0
  - aligned_offset=0x0
  - found page 1
    - addr=0x60000000, page_offset=0x0, size=0x20
file mmap on empty file test ok

[F-09] file backed private mapping test
filewrite: inum=25, addr=0x411728, size=0x3e8, off=0x0
- update_page: addr=0x411728, size=0x3e8, offset=0x0
  - aligned_offset=0x0
  - found page 0
    - addr=0x411728, page_offset=0x0, size=0x3e8
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x3e8, prot=0x3, flags=0x2, fd=3, offset=0x0
  - found page 0
sys_munmap: addr=0x60000000, size=0x3e8
file backed private mapping test ok

[F-10] file backed shared mapping test
filewrite: inum=25, addr=0x411728, size=0x3e8, off=0x0
- update_page: addr=0x411728, size=0x3e8, offset=0x0
  - aligned_offset=0x0
  - found page 0
    - addr=0x411728, page_offset=0x0, size=0x3e8
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x3e8, prot=0x3, flags=0x1, fd=3, offset=0x0
  - found page 0
sys_munmap: addr=0x60000000, size=0x3e8
filewrite: inum=25, addr=0x60000000, size=0x3e8, off=0x0
- update_page: addr=0x60000000, size=0x3e8, offset=0x0
  - aligned_offset=0x0
  - found page 0
    - addr=0x60000000, page_offset=0x0, size=0x3e8
file backed shared mapping test ok

[F-11] file backed mapping pagecache coherency test
filewrite: inum=25, addr=0x411728, size=0x10c8, off=0x0
- update_page: addr=0x411728, size=0x10c8, offset=0x0
  - aligned_offset=0x0
  - found page 0
    - addr=0x411728, page_offset=0x0, size=0x10c8
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x64, prot=0x3, flags=0x2, fd=3, offset=0x0
filewrite: inum=25, addr=0x417ea0, size=0x64, off=0x64
- update_page: addr=0x417ea0, size=0x64, offset=0x64
  - aligned_offset=0x0
  - found page 0
    - addr=0x417ea0, page_offset=0x64, size=0x64
sys_mmap: mapped=0x60001000 <= addr=0x0, size=0x1064, prot=0x3, flags=0x2, fd=3, offset=0x1000
  - found page 0
  - found page 0
sys_munmap: addr=0x60000000, size=0x64
sys_munmap: addr=0x60001000, size=0x1064
file backed mapping pagecache coherency test ok

[F-12] file backed private mapping with fork test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0xc8, prot=0x3, flags=0x2, fd=3, offset=0x0
  - found page 0
  - found page 0
sys_munmap: addr=0x60000000, size=0xc8
file backed private mapping with fork test ok

[F-13] file backed shared mapping with fork test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0xc8, prot=0x3, flags=0x1, fd=4, offset=0x0
  - found page 0
sys_munmap: addr=0x60000000, size=0xc8
filewrite: inum=25, addr=0x60000000, size=0xc8, off=0x0
- update_page: addr=0x60000000, size=0xc8, offset=0x0
  - aligned_offset=0x0
  - found page 0
    - addr=0x60000000, page_offset=0x0, size=0xc8
file backed shared mapping with fork test ok

[F-14] file backed mapping with offset test
filewrite: inum=25, addr=0x411728, size=0x10c8, off=0x0
- update_page: addr=0x411728, size=0x10c8, offset=0x0
  - aligned_offset=0x0
  - found page 0
    - addr=0x411728, page_offset=0x0, size=0x10c8
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x10c8, prot=0x3, flags=0x2, fd=4, offset=0x1000
  - found page 0
  - found page 0
sys_munmap: addr=0x60000000, size=0x10c8
file backed mapping with offset test ok

[F-15] file backed valid provided address test
sys_mmap: mapped=0x60001000 <= addr=0x60001000, size=0xc8, prot=0x3, flags=0x2, fd=4, offset=0x0
sys_munmap: addr=0x60001000, size=0xc8
file backed valid provided address test ok

[F-16] file backed invalid provided address test
mmap: invalid addr
sys_mmap: mapped=0xffffffffffffffff <= addr=0x50001000, size=0xc8, prot=0x3, flags=0x2, fd=4, offset=0x0
file backed invalid provided address test ok

[F-17] file backed overlapping provided address test
sys_mmap: mapped=0x60001000 <= addr=0x60001000, size=0x2710, prot=0x3, flags=0x2, fd=4, offset=0x0
check_vma_possible: total_vmas=1, vmas exhausted
sys_mmap: mapped=0x60000000 <= addr=0x60001000, size=0xc8, prot=0x3, flags=0x2, fd=4, offset=0x0
sys_munmap: addr=0x60001000, size=0x2710
sys_munmap: addr=0x60000000, size=0xc8
file backed overlapping provided address test ok

[F-18] file backed intermediate provided address test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x3e8, prot=0x3, flags=0x2, fd=4, offset=0x0
sys_mmap: mapped=0x60003000 <= addr=0x60003000, size=0xc8, prot=0x3, flags=0x2, fd=4, offset=0x0
check_vma_possible: total_vmas=2, vmas exhausted
sys_mmap: mapped=0x60001000 <= addr=0x60000100, size=0x3e8, prot=0x3, flags=0x2, fd=4, offset=0x0
sys_munmap: addr=0x60000000, size=0x3e8
sys_munmap: addr=0x60003000, size=0xc8
sys_munmap: addr=0x60001000, size=0x3e8
file backed intermediate provided address test ok

[F-19] file backed intermediate provided address not possible test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x3e8, prot=0x3, flags=0x2, fd=4, offset=0x0
sys_mmap: mapped=0x60003000 <= addr=0x60003000, size=0xc8, prot=0x3, flags=0x2, fd=4, offset=0x0
check_vma_possible: total_vmas=2, vmas exhausted
sys_mmap: mapped=0x60004000 <= addr=0x60000100, size=0x2710, prot=0x3, flags=0x2, fd=4, offset=0x0
sys_munmap: addr=0x60000000, size=0x3e8
sys_munmap: addr=0x60003000, size=0xc8
sys_munmap: addr=0x60004000, size=0x2710
file backed intermediate provided address not possible test ok

[F-20] File exceed file size test
filewrite: inum=25, addr=0x411728, size=0x8ee, off=0x0
- update_page: addr=0x411728, size=0x8ee, offset=0x0
  - aligned_offset=0x0
  - found page 0
    - addr=0x411728, page_offset=0x0, size=0x8ee
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x1f40, prot=0x3, flags=0x1, fd=4, offset=0x0
  - found page 0
  - found page 0
  - found page 0
sys_munmap: addr=0x60000000, size=0x1f40
filewrite: inum=25, addr=0x60000000, size=0x1f40, off=0x0
- update_page: addr=0x60000000, size=0x1f40, offset=0x0
  - aligned_offset=0x0
  - found page 0
    - addr=0x60000000, page_offset=0x0, size=0x1f40
File exceed file size test ok

[F-21] file mapping on write only file
mmap: file not readable
sys_mmap: mapped=0xffffffffffffffff <= addr=0x0, size=0x7d0, prot=0x3, flags=0x1, fd=5, offset=0x0
file mapping on write only file ok

[A-01] anonymous private mapping test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x2710, prot=0x3, flags=0x22, fd=-1, offset=0x0
- ret=0x0x60000000
sys_munmap: addr=0x60000000, size=0x2710
anonymous private mapping test ok

[A-02] anonymous shared mapping test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x2710, prot=0x3, flags=0x21, fd=-1, offset=0x0
sys_munmap: addr=0x60000000, size=0x2710
anonymous shared mapping test ok

[A-03] anonymous private mapping with fork test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0xc8, prot=0x3, flags=0x22, fd=-1, offset=0x0
anonymous private mapping with fork test ok
sys_munmap: addr=0x60000000, size=0xc8

[A-04] anonymous shared mapping with multiple forks test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x3e8, prot=0x3, flags=0x21, fd=-1, offset=0x0
sys_munmap: addr=0x60000000, size=0x3e8
anonymous shared mapping with multiple forks test ok

[A-05] anonymous private & shared mapping together with fork test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0xc8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60001000 <= addr=0x0, size=0xc8, prot=0x3, flags=0x21, fd=-1, offset=0x0
sys_munmap: addr=0x60000000, size=0xc8
sys_munmap: addr=0x60001000, size=0xc8
anonymous private & shared mapping together with fork test ok

[A-06] anonymous missing flags test
mmap: invalid flags
sys_mmap: mapped=0xffffffffffffffff <= addr=0x0, size=0x2710, prot=0x3, flags=0x20, fd=-1, offset=0x0
anonymous missing flags test ok

[A-07] anonymous exceed mapping count test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60002000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60004000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60006000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60008000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6000a000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6000c000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6000e000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60010000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60012000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60014000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60016000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60018000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6001a000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6001c000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6001e000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60020000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60022000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60024000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60026000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60028000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6002a000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6002c000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6002e000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60030000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60032000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60034000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60036000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60038000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6003a000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6003c000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x6003e000 <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap: vmas exhausted
sys_mmap: mapped=0xffffffffffffffff <= addr=0x0, size=0x13e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_munmap: addr=0x60000000, size=0x13e8
sys_munmap: addr=0x60002000, size=0x13e8
sys_munmap: addr=0x60004000, size=0x13e8
sys_munmap: addr=0x60006000, size=0x13e8
sys_munmap: addr=0x60008000, size=0x13e8
sys_munmap: addr=0x6000a000, size=0x13e8
sys_munmap: addr=0x6000c000, size=0x13e8
sys_munmap: addr=0x6000e000, size=0x13e8
sys_munmap: addr=0x60010000, size=0x13e8
sys_munmap: addr=0x60012000, size=0x13e8
sys_munmap: addr=0x60014000, size=0x13e8
sys_munmap: addr=0x60016000, size=0x13e8
sys_munmap: addr=0x60018000, size=0x13e8
sys_munmap: addr=0x6001a000, size=0x13e8
sys_munmap: addr=0x6001c000, size=0x13e8
sys_munmap: addr=0x6001e000, size=0x13e8
sys_munmap: addr=0x60020000, size=0x13e8
sys_munmap: addr=0x60022000, size=0x13e8
sys_munmap: addr=0x60024000, size=0x13e8
sys_munmap: addr=0x60026000, size=0x13e8
sys_munmap: addr=0x60028000, size=0x13e8
sys_munmap: addr=0x6002a000, size=0x13e8
sys_munmap: addr=0x6002c000, size=0x13e8
sys_munmap: addr=0x6002e000, size=0x13e8
sys_munmap: addr=0x60030000, size=0x13e8
sys_munmap: addr=0x60032000, size=0x13e8
sys_munmap: addr=0x60034000, size=0x13e8
sys_munmap: addr=0x60036000, size=0x13e8
sys_munmap: addr=0x60038000, size=0x13e8
sys_munmap: addr=0x6003a000, size=0x13e8
sys_munmap: addr=0x6003c000, size=0x13e8
sys_munmap: addr=0x6003e000, size=0x13e8
anonymous exceed mapping count test ok

[A-08] anonymous exceed mapping size test
find_vma_addr: exeed MMAPTOP
mmap: vma not available
sys_mmap: mapped=0xffffffffffffffff <= addr=0x0, size=0x25800000, prot=0x3, flags=0x22, fd=-1, offset=0x0
anonymous exceed mapping size test ok

[A-09] anonymous zero size mapping test
mmap: invalid size or offset
sys_mmap: mapped=0xffffffffffffffff <= addr=0x0, size=0x0, prot=0x3, flags=0x22, fd=-1, offset=0x0
anonymous zero size mapping test ok

[A-10] anonymous valid provided address test
sys_mmap: mapped=0x60001000 <= addr=0x60001000, size=0xc8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_munmap: addr=0x60001000, size=0xc8
anonymous valid provided address test ok

[A-11] anonymous invalid provided address test
mmap: invalid addr
sys_mmap: mapped=0xffffffffffffffff <= addr=0x50001000, size=0xc8, prot=0x3, flags=0x22, fd=-1, offset=0x0
anonymous invalid provided address test ok

[A-12] anonymous overlapping provided address test
sys_mmap: mapped=0x60001000 <= addr=0x60001000, size=0x2710, prot=0x3, flags=0x22, fd=-1, offset=0x0
check_vma_possible: total_vmas=1, vmas exhausted
sys_mmap: mapped=0x60000000 <= addr=0x60001000, size=0xc8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_munmap: addr=0x60001000, size=0x2710
sys_munmap: addr=0x60000000, size=0xc8
anonymous overlapping provided address test ok

[A-13] anonymous intermediate provided address test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x3e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60003000 <= addr=0x60003000, size=0xc8, prot=0x3, flags=0x22, fd=-1, offset=0x0
check_vma_possible: total_vmas=2, vmas exhausted
sys_mmap: mapped=0x60001000 <= addr=0x60000100, size=0x3e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_munmap: addr=0x60000000, size=0x3e8
sys_munmap: addr=0x60003000, size=0xc8
sys_munmap: addr=0x60001000, size=0x3e8
anonymous intermediate provided address test ok

[A-14] anonymous intermediate provided address not possible test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x3e8, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_mmap: mapped=0x60003000 <= addr=0x60003000, size=0xc8, prot=0x3, flags=0x22, fd=-1, offset=0x0
check_vma_possible: total_vmas=2, vmas exhausted
sys_mmap: mapped=0x60004000 <= addr=0x60000100, size=0x2710, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_munmap: addr=0x60000000, size=0x3e8
sys_munmap: addr=0x60003000, size=0xc8
sys_munmap: addr=0x60004000, size=0x2710
anonymous intermediate provided address not possible test ok

[O-01] munmap only partial size test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x2710, prot=0x3, flags=0x22, fd=-1, offset=0x0
sys_munmap: addr=0x60000000, size=0x5
sys_munmap: addr=0x60001000, size=0x2710
munmap only partial size test ok

[O-02] write on read only mapping test
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x2710, prot=0x1, flags=0x22, fd=-1, offset=0x0
write on read only mapping test ok

[O-03] none permission on mapping test
sys_mmap: mapped=0x70003000 <= addr=0x70003000, size=0x2710, prot=0x0, flags=0x22, fd=-1, offset=0x0
ret=0x0x70003000
none permission on mapping test ok

[O-04] mmap valid address map fixed flag test
sys_mmap: mapped=0x60001000 <= addr=0x60001000, size=0xc8, prot=0x3, flags=0x32, fd=-1, offset=0x0
sys_munmap: addr=0x60001000, size=0xc8
mmap valid address map fixed flag test ok

[O-05] mmap invalid address map fixed flag test
mmap: invalid addr=0x50001000
sys_mmap: mapped=0xffffffffffffffff <= addr=0x50001000, size=0xc8, prot=0x3, flags=0x32, fd=-1, offset=0x0
mmap: invalid addr=0x60000100
sys_mmap: mapped=0xffffffffffffffff <= addr=0x60000100, size=0xc8, prot=0x3, flags=0x32, fd=-1, offset=0x0
sys_mmap: mapped=0x60000000 <= addr=0x60000000, size=0xc8, prot=0x3, flags=0x32, fd=-1, offset=0x0
check_vma_possible: total_vmas=1, vmas exhausted
mmap: invalid 2 addr=0x60000000
sys_mmap: mapped=0xffffffffffffffff <= addr=0x60000000, size=0xc8, prot=0x3, flags=0x32, fd=-1, offset=0x0
sys_munmap: addr=0x60000000, size=0xc8
mmap invalid address map fixed flag test ok

file_test:  ok: 21, ng: 0
anon_test:  ok: 13, ng: 0
other_test: ok: 5, ng: 0
```

## エラー以外のコメントを外した

- ただし、[F-08]はi=32以下でないとエラーになる。

```
$ mmaptest2
[F-01] file backed mapping invalid file descriptor test
sys_mmap: invalid fd
sys_mmap: invalid fd
sys_mmap: invalid fd
file backed mapping invalid file descriptor test ok

[F-02] file backed mapping invalid flags test
mmap: invalid flags
file backed mapping invalid flags test ok

[F-03] file backed writeable shared mapping on read only file test
mmap: file not writable
file backed writeable shared mapping on read only file test ok

[F-04] file backed read only shared mapping on read only file test
file backed read only shared mapping on read only file test ok

[F-05] file backed private mapping permission test
file backed private mapping permission test ok

[F-06] file backed exceed mapping size test
find_vma_addr: exeed MMAPTOP
mmap: vma not available
file backed exceed mapping size test ok

[F-07] file backed exceed mapping count test
mmap: vmas exhausted
file backed exceed mapping count test ok

[F-08] file mmap on empty file test
file mmap on empty file test ok

[F-09] file backed private mapping test
file backed private mapping test ok

[F-10] file backed shared mapping test
file backed shared mapping test ok

[F-11] file backed mapping pagecache coherency test
file backed mapping pagecache coherency test ok

[F-12] file backed private mapping with fork test
file backed private mapping with fork test ok

[F-13] file backed shared mapping with fork test
file backed shared mapping with fork test ok

[F-14] file backed mapping with offset test
file backed mapping with offset test ok

[F-15] file backed valid provided address test
file backed valid provided address test ok

[F-16] file backed invalid provided address test
mmap: invalid addr
file backed invalid provided address test ok

[F-17] file backed overlapping provided address test
file backed overlapping provided address test ok

[F-18] file backed intermediate provided address test
file backed intermediate provided address test ok

[F-19] file backed intermediate provided address not possible test
file backed intermediate provided address not possible test ok

[F-20] File exceed file size test
File exceed file size test ok

[F-21] file mapping on write only file
mmap: file not readable
file mapping on write only file ok

[A-01] anonymous private mapping test
- ret=0x0x60000000
anonymous private mapping test ok

[A-02] anonymous shared mapping test
anonymous shared mapping test ok

[A-03] anonymous private mapping with fork test
anonymous private mapping with fork test ok

[A-04] anonymous shared mapping with multiple forks test
anonymous shared mapping with multiple forks test ok

[A-05] anonymous private & shared mapping together with fork test
anonymous private & shared mapping together with fork test ok

[A-06] anonymous missing flags test
mmap: invalid flags
anonymous missing flags test ok

[A-07] anonymous exceed mapping count test
mmap: vmas exhausted
anonymous exceed mapping count test ok

[A-08] anonymous exceed mapping size test
find_vma_addr: exeed MMAPTOP
mmap: vma not available
anonymous exceed mapping size test ok

[A-09] anonymous zero size mapping test
mmap: invalid size or offset
anonymous zero size mapping test ok

[A-10] anonymous valid provided address test
anonymous valid provided address test ok

[A-11] anonymous invalid provided address test
mmap: invalid addr
anonymous invalid provided address test ok

[A-12] anonymous overlapping provided address test
anonymous overlapping provided address test ok

[A-13] anonymous intermediate provided address test
anonymous intermediate provided address test ok

[A-14] anonymous intermediate provided address not possible test
anonymous intermediate provided address not possible test ok

[O-01] munmap only partial size test
munmap only partial size test ok

[O-02] write on read only mapping test
write on read only mapping test ok

[O-03] none permission on mapping test
none permission on mapping test ok

[O-04] mmap valid address map fixed flag test
mmap valid address map fixed flag test ok

[O-05] mmap invalid address map fixed flag test
mmap: invalid addr=0x50001000
mmap: invalid addr=0x60000100
mmap: invalid 2 addr=0x60000000
mmap invalid address map fixed flag test ok

file_test:  ok: 21, ng: 0
anon_test:  ok: 13, ng: 0
other_test: ok: 5, ng: 0
```

# `cp`のエラーは直らず

- なぜ0x60003000-0x60024014を確保して0x60024040にアクセスするのか?

```
$ uniq test
mmap: invalid addr=0x41d000
aa
bb
cc
$ cp test test2
mmap: invalid addr=0x430000
tdo: file=test2, force=0, err=2, errno=2
Total vmas: 3
Virtual mapping address: 0x60000000	size: 0x1000	shared: 0
Virtual mapping address: 0x60001000	size: 0x2000	shared: 0
Virtual mapping address: 0x60003000	size: 0x21014	shared: 0
pagefault_handler: Segmentation Fault 2: 60024040
pagefault handler
`````````

```
$ cp test test2
mmap: invalid addr=0x430000
sys_mmap: mapped=0xffffffffffffffff <= addr=0x430000, size=0x1000, prot=0x0, flags=0x32, fd=-1, offset=0x0
mmap1: mappded: 0x0xffffffffffffffff <- addr: 0x0x430000, size: 0x1000, PROT_NONE, MAP_ANON|MAP_PRIVATE|MAP_FIXED
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x1000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60000000 <- addr: 0, size: 0x1000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000000, aidx: 1, lidx: 1, scls: 23
enframe: slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000010, aidx: 1, lidx: 1, scls: 19
enframe: slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000020, aidx: 1, lidx: 1, scls: 15
enframe: slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
m: amask: 0x1f, fmask: 0x0, mem: 0x0x60000030, aidx: 4, lidx: 4, scls: 5
enframe: slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
sys_mmap: mapped=0x60001000 <= addr=0x0, size=0x2000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60001000 <- addr: 0, size: 0x2000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x1, fmask: 0x6, mem: 0x0x60001000, aidx: 0, lidx: 2, scls: 25
enframe: slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
sys_mmap: mapped=0x60003000 <= addr=0x0, size=0x42d2fc, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap4: mapped: 0x0x60003000 <- addr: 0, size: 0x42d2fc, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
g: addr: 0x0x60003000, maplen: 0x42e
enframe: slack: 208, off: 3, p: 0x0x60003010, end: 0x0x60430ffc
Total vmas: 3
Virtual mapping address: 0x60000000	size: 0x1000	shared: 0
Virtual mapping address: 0x60001000	size: 0x2000	shared: 0
Virtual mapping address: 0x60003000	size: 0x42d2fc	shared: 0
pagefault_handler: Segmentation Fault 2: 60430328
- elr: 0x418dc8, far: 0x60430328
pagefault handler
```

## libcでのmmap呼び出し時のサイズをROUNDUP

- ページフォルトにはならないが、別のエラーに。これはだめなので戻す。

```
$ cp test test2
mmap: invalid addr=0x430000
sys_mmap: mapped=0xffffffffffffffff <= addr=0x430000, size=0x1000, prot=0x0, flags=0x32, fd=-1, offset=0x0
mmap1: mappded: 0x0xffffffffffffffff <- addr: 0x0x430000, size: 0x1000, PROT_NONE, MAP_ANON|MAP_PRIVATE|MAP_FIXED
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x1000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60000000 <- addr: 0, size: 0x1000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000000, aidx: 1, lidx: 1, scls: 23
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000010, aidx: 1, lidx: 1, scls: 19
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000020, aidx: 1, lidx: 1, scls: 15
m: amask: 0x1f, fmask: 0x0, mem: 0x0x60000030, aidx: 4, lidx: 4, scls: 5
sys_mmap: mapped=0x60001000 <= addr=0x0, size=0x2000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60001000 <- addr: 0, size: 0x2000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x1, fmask: 0x6, mem: 0x0x60001000, aidx: 0, lidx: 2, scls: 25
sys_mmap: mapped=0x60003000 <= addr=0x0, size=0x22000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap4: mapped: 0x0x60003000 <- addr: 0, size: 0x22000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
g: addr: 0x0x60003000, maplen: 0x22
data abort: insruction 0x418d88, fault addr 0xfffffffffffffffd, dfs=4
```

## `MAP_ANON`の場合、`sys_mmap`でsizeをroundup

```
$ cp test test2
mmap: invalid addr=0x430000
sys_mmap: mapped=0xffffffffffffffff <= addr=0x430000, size=0x1000, prot=0x0, flags=0x32, fd=-1, offset=0x0
mmap1: mappded: 0x0xffffffffffffffff <- addr: 0x0x430000, size: 0x1000, PROT_NONE, MAP_ANON|MAP_PRIVATE|MAP_FIXED
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x1000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60000000 <- addr: 0, size: 0x1000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000000, aidx: 1, lidx: 1, scls: 23
enframe: slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000010, aidx: 1, lidx: 1, scls: 19
enframe: slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000020, aidx: 1, lidx: 1, scls: 15
enframe: slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
m: amask: 0x1f, fmask: 0x0, mem: 0x0x60000030, aidx: 4, lidx: 4, scls: 5
enframe: slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
sys_mmap: mapped=0x60001000 <= addr=0x0, size=0x2000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60001000 <- addr: 0, size: 0x2000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x1, fmask: 0x6, mem: 0x0x60001000, aidx: 0, lidx: 2, scls: 25
enframe: slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
sys_mmap: mapped=0x60003000 <= addr=0x0, size=0x42e000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap4: mapped: 0x0x60003000 <- addr: 0, size: 0x42d2fc, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
g: addr: 0x0x60003000, maplen: 0x42e
enframe: slack: 208, off: 3, p: 0x0x60003010, end: 0x0x60430ffc
cp: error reading 'test'            // 新しいエラー: coreutils/src/copy.cで出力。read()の結果がマイナス
Total vmas: 3
Virtual mapping address: 0x60000000	size: 0x1000	shared: 0
Virtual mapping address: 0x60001000	size: 0x2000	shared: 0
Virtual mapping address: 0x60003000	size: 0x42e000	shared: 0
pagefault_handler: Segmentation Fault 2: 42ef20
- elr: 0x41c784, far: 0x42ef20      // elrはprintf_coreの先頭アドレス, farは不明（cpコマンドのサイズより大きい）
pagefault handler
```

## `argptr()`がVMAに対応していなかったので修正

- testが読めるようになったが別のエラー発生

```
$ cp test test2
mmap: invalid addr=0x431000
sys_mmap: mapped=0xffffffffffffffff <= addr=0x431000, size=0x1000, prot=0x0, flags=0x32, fd=-1, offset=0x0
mmap1: mappded: 0x0xffffffffffffffff <- addr: 0x0x431000, size: 0x1000, PROT_NONE, MAP_ANON|MAP_PRIVATE|MAP_FIXED
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x1000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60000000 <- addr: 0, size: 0x1000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000000, aidx: 1, lidx: 1, scls: 23
enframe: slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000010, aidx: 1, lidx: 1, scls: 19
enframe: slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000020, aidx: 1, lidx: 1, scls: 15
enframe: slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
m: amask: 0x1f, fmask: 0x0, mem: 0x0x60000030, aidx: 4, lidx: 4, scls: 5
enframe: slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
sys_mmap: mapped=0x60001000 <= addr=0x0, size=0x2000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60001000 <- addr: 0, size: 0x2000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x1, fmask: 0x6, mem: 0x0x60001000, aidx: 0, lidx: 2, scls: 25
enframe: slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
sys_mmap: mapped=0x60003000 <= addr=0x0, size=0x22000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap4: mapped: 0x0x60003000 <- addr: 0, size: 0x21014, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
pagefault_handler: va=0x60003000
pagefault_handler: alloc: 0x60003000
g: addr: 0x0x60003000, maplen: 0x22
enframe: slack: 254, off: 3, p: 0x0x60003010, end: 0x0x60024ffc
pagefault_handler: va=0x60024040
pagefault_handler: alloc: 0x60024000
copy: fd: 3, buf: 0x0x60004000, count: 131072
mm: 0x60004000 <= 0xffff000000901ee0; 0xf
Synchronous: Data abort, same EL, Translation fault at level 3:
  ESR_EL1 0x96000047 ELR_EL1 0xffff0000000827c8
 SPSR_EL1 0x200003c5 FAR_EL1 0x60004000
	irq_error: irq of type 4 unimplemented.
```

## `irq 4`をventryで処理するよう変更

```
$ cp test test2
mmap: invalid addr=0x431000
sys_mmap: mapped=0xffffffffffffffff <= addr=0x431000, size=0x1000, prot=0x0, flags=0x32, fd=-1, offset=0x0
mmap1: mappded: 0x0xffffffffffffffff <- addr: 0x0x431000, size: 0x1000, PROT_NONE, MAP_ANON|MAP_PRIVATE|MAP_FIXED
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x1000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60000000 <- addr: 0, size: 0x1000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
pagefault_handler: va=0x60000008
pagefault_handler: alloc: 0x60000000
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000000, aidx: 1, lidx: 1, scls: 23
enframe: slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000010, aidx: 1, lidx: 1, scls: 19
enframe: slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000020, aidx: 1, lidx: 1, scls: 15
enframe: slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
m: amask: 0x1f, fmask: 0x0, mem: 0x0x60000030, aidx: 4, lidx: 4, scls: 5
enframe: slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
sys_mmap: mapped=0x60001000 <= addr=0x0, size=0x2000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60001000 <- addr: 0, size: 0x2000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
pagefault_handler: va=0x60001008
pagefault_handler: alloc: 0x60001000
m: amask: 0x1, fmask: 0x6, mem: 0x0x60001000, aidx: 0, lidx: 2, scls: 25
enframe: slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
sys_mmap: mapped=0x60003000 <= addr=0x0, size=0x22000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap4: mapped: 0x0x60003000 <- addr: 0, size: 0x21014, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
pagefault_handler: va=0x60003000
pagefault_handler: alloc: 0x60003000
g: addr: 0x0x60003000, maplen: 0x22
enframe: slack: 254, off: 3, p: 0x0x60003010, end: 0x0x60024ffc
pagefault_handler: va=0x60024040
pagefault_handler: alloc: 0x60024000
copy: fd: 3, buf: 0x0x60004000, count: 131072
trap: CPU 0 unexpected irq: 0x25, iss: 0x47.
Synchronous: Unknown:
  ESR_EL1 0x0 ELR_EL1 0xffff000000082768
 SPSR_EL1 0x200003c5 FAR_EL1 0x60004000
	irq_error: irq of type 8 unimplemented.
```

## `ESR_EL1/EC=0x25`も処理するよう変更して`cp`コマンド成功

```
$ cp test test2
mmap: invalid addr=0x431000
sys_mmap: mapped=0xffffffffffffffff <= addr=0x431000, size=0x1000, prot=0x0, flags=0x32, fd=-1, offset=0x0
mmap1: mappded: 0x0xffffffffffffffff <- addr: 0x0x431000, size: 0x1000, PROT_NONE, MAP_ANON|MAP_PRIVATE|MAP_FIXED
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x1000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60000000 <- addr: 0, size: 0x1000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
pagefault_handler: va=0x60000008
pagefault_handler: alloc: 0x60000000
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000000, aidx: 1, lidx: 1, scls: 23
enframe: slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000010, aidx: 1, lidx: 1, scls: 19
enframe: slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000020, aidx: 1, lidx: 1, scls: 15
enframe: slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
m: amask: 0x1f, fmask: 0x0, mem: 0x0x60000030, aidx: 4, lidx: 4, scls: 5
enframe: slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
sys_mmap: mapped=0x60001000 <= addr=0x0, size=0x2000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60001000 <- addr: 0, size: 0x2000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
pagefault_handler: va=0x60001008
pagefault_handler: alloc: 0x60001000
m: amask: 0x1, fmask: 0x6, mem: 0x0x60001000, aidx: 0, lidx: 2, scls: 25
enframe: slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
sys_mmap: mapped=0x60003000 <= addr=0x0, size=0x22000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap4: mapped: 0x0x60003000 <- addr: 0, size: 0x21014, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
pagefault_handler: va=0x60003000
pagefault_handler: alloc: 0x60003000
g: addr: 0x0x60003000, maplen: 0x22
enframe: slack: 254, off: 3, p: 0x0x60003010, end: 0x0x60024ffc
pagefault_handler: va=0x60024040
pagefault_handler: alloc: 0x60024000
copy: fd: 3, buf: 0x0x60004000, count: 131072
pagefault_handler: va=0x60004000
pagefault_handler: alloc: 0x60004000
copy: fd: 3, buf: 0x0x60004000, count: 131072
sys_munmap: addr=0x60003000, size=0x22000
$ ls
...
test           8000 25 15
test2          8000 26 15
```

## libcのデバッグコメントを外したらエラー再現

- enframeとg構造体をデバッグプリントするとエラーとならない。他の組み合わせはエラー。


### mmap関係とenfram, (m, g)構造体をデバッグプリント => cp成功（上で成功したケース）

```
$ cp test test2
mmap1: mappded: 0x0xffffffffffffffff <- addr: 0x0x430000, size: 0x1000, PROT_NONE, MAP_ANON|MAP_PRIVATE|MAP_FIXED
mmap3: mapped: 0x0x60000000 <- addr: 0, size: 0x1000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000000, aidx: 1, lidx: 1, scls: 23
enframe: stride: 0x7f0, slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000010, aidx: 1, lidx: 1, scls: 19
enframe: stride: 0x3f0, slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000020, aidx: 1, lidx: 1, scls: 15
enframe: stride: 0x1f0, slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
m: amask: 0x1f, fmask: 0x0, mem: 0x0x60000030, aidx: 4, lidx: 4, scls: 5
enframe: stride: 0x60, slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
mmap3: mapped: 0x0x60001000 <- addr: 0, size: 0x2000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x1, fmask: 0x6, mem: 0x0x60001000, aidx: 0, lidx: 2, scls: 25
enframe: stride: 0xaa0, slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
mmap4: mapped: 0x0x60003000 <- addr: 0, size: 0x42d2fc, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
g: addr: 0x0x60003000, maplen: 0x42e
enframe: stride: 0x42dff0, slack: 208, off: 3, p: 0x0x60003010, end: 0x0x60430ffc
```
### `enframe()`をコメントアウト => エラー

```
$ cp test test2
mmap1: mappded: 0x0xffffffffffffffff <- addr: 0x0x430000, size: 0x1000, PROT_NONE, MAP_ANON|MAP_PRIVATE|MAP_FIXED
mmap3: mapped: 0x0x60000000 <- addr: 0, size: 0x1000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000000, aidx: 1, lidx: 1, scls: 23
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000010, aidx: 1, lidx: 1, scls: 19
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000020, aidx: 1, lidx: 1, scls: 15
m: amask: 0x1f, fmask: 0x0, mem: 0x0x60000030, aidx: 4, lidx: 4, scls: 5
mmap3: mapped: 0x0x60001000 <- addr: 0, size: 0x2000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
m: amask: 0x1, fmask: 0x6, mem: 0x0x60001000, aidx: 0, lidx: 2, scls: 25
mmap4: mapped: 0x0x60003000 <- addr: 0, size: 0x21014, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
g: addr: 0x0x60003000, maplen: 0x22
data abort: insruction 0x418d88, fault addr 0xfffffffffffffffd, dfs=4
```

### `enframe()`だけデバッグプリント => 別のエラー

```
$ cp test test2
enframe: stride: 0x7f0, slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
enframe: stride: 0x3f0, slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
enframe: stride: 0x1f0, slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
enframe: stride: 0x60, slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
enframe: stride: 0xaa0, slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
enframe: stride: 0x10, slack: -273710, off: 3, p: 0x0x60003010, end: 0x0x6000301c // slack値がおかしい
trap: CPU 0 unexpected irq: 0x3c, iss: 0x3e8.
Synchronous: Unknown:
  ESR_EL1 0x0 ELR_EL1 0x4185d0
 SPSR_EL1 0x80000000 FAR_EL1 0x60004000
```

- `get_meta()`関数に`brk 0x3e8`命令があり。この割り込みを無視するようにtrap.cを修正 => ストール

```
$ cp test test2
enframe: stride: 0x7f0, slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
enframe: stride: 0x3f0, slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
enframe: stride: 0x1f0, slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
enframe: stride: 0x60, slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
enframe: stride: 0xaa0, slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
enframe: stride: 0x10, slack: -273710, off: 3, p: 0x0x60003010, end: 0x0x6000301c   // ここでストール
```

### enframeとm構造体をデバッグプリントする => エラー

```
$ cp test test2
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000000, aidx: 1, lidx: 1, scls: 23
enframe: stride: 0x7f0, slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000010, aidx: 1, lidx: 1, scls: 19
enframe: stride: 0x3f0, slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000020, aidx: 1, lidx: 1, scls: 15
enframe: stride: 0x1f0, slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
m: amask: 0x1f, fmask: 0x0, mem: 0x0x60000030, aidx: 4, lidx: 4, scls: 5
enframe: stride: 0x60, slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
m: amask: 0x1, fmask: 0x6, mem: 0x0x60001000, aidx: 0, lidx: 2, scls: 25
enframe: stride: 0xaa0, slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
enframe: stride: 0x10, slack: -273710, off: 3, p: 0x0x60003010, end: 0x0x6000301c   // エラー
```

### enframeとg構造体をデバッグプリントする => cp成功

```
$ cp test test2
enframe: stride: 0x7f0, slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
enframe: stride: 0x3f0, slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
enframe: stride: 0x1f0, slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
enframe: stride: 0x60, slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
enframe: stride: 0xaa0, slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
g: addr: 0x0x60003000, maplen: 0x42e
enframe: stride: 0x42dff0, slack: 208, off: 3, p: 0x0x60003010, end: 0x0x60430ffc
```

### g構造体だけデバッグプリントする => エラー

```
$ cp test test2
g: addr: 0x0x60003000, maplen: 0x22
data abort: insruction 0x418d88, fault addr 0xfffffffffffffffd, dfs=4
```

# その他のコマンドの現状

```
$ test2
rm: cannot remove 'test2': Operation not permitted
$ mv test2 test3
mv: cannot stat 'test2': Operation not permitted
$ wc test
mmap: invalid addr=0x41f000
sys_mmap: mapped=0xffffffffffffffff <= addr=0x41f000, size=0x1000, prot=0x0, flags=0x32, fd=-1, offset=0x0
mmap1: mappded: 0x0xffffffffffffffff <- addr: 0x0x41f000, size: 0x1000, PROT_NONE, MAP_ANON|MAP_PRIVATE|MAP_FIXED
sys_mmap: mapped=0x60000000 <= addr=0x0, size=0x1000, prot=0x3, flags=0x22, fd=-1, offset=0x0
mmap3: mapped: 0x0x60000000 <- addr: 0, size: 0x1000, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANON
pagefault_handler: va=0x60000008
pagefault_handler: alloc: 0x60000000
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000000, aidx: 1, lidx: 1, scls: 23
enframe: slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000010, aidx: 1, lidx: 1, scls: 19
enframe: slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
m: amask: 0x3, fmask: 0x0, mem: 0x0x60000020, aidx: 1, lidx: 1, scls: 15
enframe: slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
m: amask: 0x7f, fmask: 0x0, mem: 0x0x60000030, aidx: 6, lidx: 6, scls: 3
enframe: slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000007c
enframe: slack: 0, off: 1, p: 0x0x60000220, end: 0x0x6000040c
m: amask: 0x7, fmask: 0x0, mem: 0x0x60000220, aidx: 2, lidx: 2, scls: 9
enframe: slack: 1, off: 1, p: 0x0x60000230, end: 0x0x600002cc
unexpected interrupt 0 at cpu 0
Synchronous: Unknown:
  ESR_EL1 0x0 ELR_EL1 0x41ac40
 SPSR_EL1 0x20000000 FAR_EL1 0x60000008
	irq_error: irq of type 8 unimplemented.
kern/console.c:250: kernel panic at cpu 0.

$ mkdir dir1
sys_mkdirat: ignores mode
$ ls dir1
.              4000 26 32
..             4000 1 4096
$ cd dir1
$ /ls .
pagefault_handler: va=0x814443
Total vmas: 0
pagefault_handler: Segmentation Fault 2: 814443
- elr: 0x40078c, far: 0x814443
pagefault handler
```

# 今日の知見

- デバッグプリントのある無しで結果が変わる（どこかにバグがあるはず）
- libc, coreutilsのコマンドを手短に再構築する方法

  ```
  $ cd libc
  $ touch src/malloc/mallocng/malloc.c    // meta.hを修正した場合は、これを使用してるcソースをtouch
  $ make
  $ cd ../coreutils-8.32
  $ touch src/cp.c src/copy.c             // とりあえず試したいコマンドのソースをtouch
  $ make                                  // 当該コマンドだけ再構築される
  $ cd ..
  $ rm -rf obj
  $ make                                  // objを再構築する場合はmakeとmake qemuは別に行う
  $ make qemu                             // カーネルだけ修正した場合は、makeは不要
  ```

# cpコマンドに誤り発見

```
$ cp test test2
$ ls
test           8000 25 15
test2          8000 26 15           // 長さはおなじになっている
$ cat test2
a                                   // この'a'はどこから? 先頭がNULL
aa
bb
cc
cc
$ cat test
aa
aa
bb
cc
cc
```

```
test (inode)
04120640: 0200 0000 0000 0100 0f00 0000 e702 0000  ................
test (data)
043e7000: 6161 0a61 610a 6262 0a63 630a 6363 0a00  aa.aa.bb.cc.cc..

test2 (inode)
04120680: 0200 0000 0000 0100 0f00 0000 e802 0000  ................
test2 (data)
043e8000: 0061 0a61 610a 6262 0a63 630a 6363 0a00  .a.aa.bb.cc.cc..
```

```
$ cp test test2
enframe: stride: 0x7f0, slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
enframe: stride: 0x3f0, slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
enframe: stride: 0x1f0, slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
enframe: stride: 0x60, slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
enframe: stride: 0xaa0, slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
g: addr: 0x0x60003000, maplen: 0x22
enframe: stride: 0x21ff0, slack: 254, off: 3, p: 0x0x60003010, end: 0x0x60024ffc
copy: fd: 3, buf: 0x0x60004000, count: 131072
- n_read: 15, buf[0-3]: 0x0, 0x61, 0xa, 0x61
copy: fd: 3, buf: 0x0x60004000, count: 131072
```

```
$ cp test test2
fileread: [0-4]: 0x63, 0x70, 0x20, 0x74     // cp t: console input
enframe: stride: 0x7f0, slack: 0, off: 1, p: 0x0x60000010, end: 0x0x600007fc
enframe: stride: 0x3f0, slack: 0, off: 1, p: 0x0x60000020, end: 0x0x6000040c
enframe: stride: 0x1f0, slack: 0, off: 1, p: 0x0x60000030, end: 0x0x6000021c
enframe: stride: 0x60, slack: 0, off: 1, p: 0x0x60000040, end: 0x0x6000009c
enframe: stride: 0xaa0, slack: 38, off: 2, p: 0x0x60001010, end: 0x0x60001aac
cp: copy test to test2
copy_internal: src: test, dst: test2, new: 0
call copy_reg
coppy_reg: from test to test2: 0x0
called sys_openat
sys_openat: dirfd=-100, path:=test, flags=0x20000, mode=0x0
sys_openat: fd=3, readable=1, writable=0, flags=0x20000, off=0
called sys_openat
sys_openat: dirfd=-100, path:=test2, flags=0x200c1, mode=0x0
sys_openat: fd=4, readable=0, writable=1, flags=0x200c1, off=0
g: addr: 0x0x60003000, maplen: 0x22
enframe: stride: 0x21ff0, slack: 254, off: 3, p: 0x0x60003010, end: 0x0x60024ffc
sparse_copy: fd: 3, buf: 0x0x60004000, count: 131072
called sys_read
fileread: [0-4]: 0x0, 0x61, 0xa, 0x61       // 先頭が0x0:  sparce flagsをチェックする
- n_read: 15
- buf[0-3]: 0x0, 0x61, 0xa, 0x61
sparse_copy: fd: 3, buf: 0x0x60004000, count: 131072
called sys_read
fileread: [0-4]: 0x0, 0x61, 0xa, 0x61
- n_read: 0
$
```
