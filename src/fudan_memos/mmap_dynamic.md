# mmap版でdynamic linkアプリを動かす

## 2022.03.28現在

- フォルトアドレスに対応するmmap regionなし

```
# hello
pf_handler[7]: fault_addr: 0x5ed000, flt: 2

======= PROC[7]: /bin/hello TRAP FRAME DUMP =========
sp:    0x0000fffffffffd10  tpidr: 0x00000000004342f0
spsr:  0x0000000060000000  esr:   0x000000009200000b
elr:   0x0000c00000064768  x0:    0x0000000000000000
x1:    0x00000000055b36e2  x2:    0x0000000000436000
x3:    0x000000000017b6e4  x4:    0x0000000000000001
x5:    0x0000c000000ae070  x6:    0x0000c000000ae070
x7:    0x0000000000436002  x8:    0x0000000000000000
x9:    0x0000c0000009b431  x10:   0x0000000000000000
x11:   0x0000000000000000  x12:   0x0000000000000000
x13:   0x0000000000000018  x14:   0x0000c000000ae070
x15:   0x000000007984f81a  x16:   0x0000c000000669b0
x17:   0x0000000000000000  x18:   0x0000c0000009b431
x19:   0x0000000004000000  x20:   0x0000000000000000
x21:   0x0000000001e613e0  x22:   0x0000000000000406
x23:   0x0000000000000067  x24:   0x00000000055b36e2
x25:   0x0000000000000000  x26:   0x0000000000434538
x27:   0x0000000000419bfc  x28:   0x0000000000433040
x29:   0x0000fffffffffd70  x30:   0x0000c00000065864
===================== DUMP END =====================

pagefault handler
```

## debug printを入れる

```
start_code=0xc00000000000, end_code=0xc0000000072c
start_data=0xc00000001db8, end_data=0xc00000002008
bss=0xc00000002008, brk=0xc00000002010
entry=0xc0000006454c, bprm->p=0xfffffffffe90

# hello
unexpected interrupt 0 at cpu 0

======= PROC[7]: /bin/hello TRAP FRAME DUMP =========
sp:    0x0000fffffffffd40  tpidr: 0x00000000004342f0
spsr:  0x0000000060000000  esr:   0x0000000002000000
elr:   0x0000000000000001  x0:    0x0000fffffffffe90
x1:    0x0000fffffffffeb8  x2:    0x0000000000000001
x3:    0x00000000055b36e4  x4:    0x0000000000000001

x29:   0x0000fffffffffd60  x30:   0x0000c00000066b10
===================== DUMP END =====================

==== stack proc[7] =========
fffffffffd20: 000000000041bc04 0000000000419bfc 0000c00000066af8 0000000000000000
fffffffffd40: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffd60: 0000000000000000 0000000000000000 0000000000434ba8 0000000000434b88
fffffffffd80: 0000000000428ff8 0000000000434b98 0000000000000000 0000000000000000

==== stack proc[7] =========

同期: 不明
ESR_EL1  (例外原因)     0x0000000002000000
ELR_EL1  (例外LR)       0x0000000000000001
SPSR_EL1 (状態レジスタ) 0x0000000060000000
FAR_EL1  (フォルトaddr) 0x00000000156cdb90

IRQエラー: IRQ種別 (8) は未実装。
```

## program headerとmmap list

```
$ aarch64-linux-gnu-readelf -lW obj/user_dyn/bin/hello
Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  INTERP         0x0001c8 0x00000000000001c8 0x00000000000001c8 0x00001a 0x00001a R   0x1
      [Requesting program interpreter: /lib/ld-musl-aarch64.so.1]
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x00072c 0x00072c R E 0x1000
  LOAD           0x000db8 0x0000000000001db8 0x0000000000001db8 0x000250 0x000258 RW  0x1000

$ aarch64-linux-gnu-readelf -lW user/lib/ld-musl-aarch64.so.1

Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x098ac4 0x098ac4 R E 0x10000
  LOAD           0x098b90 0x00000000000a8b90 0x00000000000a8b90 0x00089a 0x003568 RW  0x10000

elf_map (application): addr=0x0,            mapped=0xc00000000000, length=0x72c   (filesz=0x72c,   offset=0x0)
elf_map (application): addr=0xc00000001db8, mapped=0xc00000001000, length=0x1008  (filesz=0x250,   offset=0xdb8)
elf_map (interpreter): addr=0x0,            mapped=0xc00000003000, length=0x98ac4 (filesz=0x98ac4, offset=0x0)
elf_map (interpreter): addr=0xc000000abb90, mapped=0xc000000ab000, length=0x142a  (filesz=0x89a,   offset=0xb90)

== PRINT mmap_region[7] (binfmt_elf): head=0xffff00003afe0c30, nregions=6 ==
 - region[1]: addr=0xc00000000000, length=0x1000,  prot=0x5, flags=0x801, f=35,  offset=0x0
 - region[2]: addr=0xc00000001000, length=0x2000,  prot=0x3, flags=0x12,  f=35,  offset=0x0         // length?
 - region[3]: addr=0xc00000003000, length=0x99000, prot=0x5, flags=0x801, f=148, offset=0x0
 - region[4]: addr=0xc000000ab000, length=0x2000,  prot=0x3, flags=0x12,  f=148, offset=0x98000     // length?
 - region[5]: addr=0xc000000ad000, length=0x3000,  prot=0x3, flags=0x32,  f=0,   offset=0x0
 - region[6]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32,  f=0,   offset=0x0
```

# "load_elf_binary()`実行時のデバッグ

```
(gdb) p/x elf_bss
$26 = 0xc000000ad000
(gdb) p/x last_bss
$28 = 0xc000000af0f8
(gdb) x/16xg elf_bss
0xc000000ad000:	0x0000000000000000	0x0000000000000000
0xc000000ad010:	0x0000000000000000	0x0000000000000000
...
(gdb) x/16xg
...
0xc000000affe0:	0x0000000000000000	0x0000000000000000
0xc000000afff0:	0x0000000000000000	0x0000000000000000

(gdb) p/x *interp_load_addr
$32 = 0xc00000003000
(gdb) p/x elf_entry
$37 = 0xc0000006454c

(gdb) p/x bprm->p
$38 = 0xfffffffffea0
(gdb) x/16gx bprm->p
0xfffffffffea0:	0x0000000000000001	0x0000ffffffffffe7
0xfffffffffeb0:	0x0000000000000000	0x0000000000000000
0xfffffffffec0:	0x0000000000000010	0x0000000000000887
0xfffffffffed0:	0x0000000000000006	0x0000000000001000
0xfffffffffee0:	0x0000000000000011	0x0000000000000064
0xfffffffffef0:	0x0000000000000003	0x0000c00000000040
0xffffffffff00:	0x0000000000000004	0x0000000000000038
0xffffffffff10:	0x0000000000000005	0x0000000000000007
(gdb)
0xffffffffff20:	0x0000000000000007	0x0000c00000003000
0xffffffffff30:	0x0000000000000008	0x0000000000000000
0xffffffffff40:	0x0000000000000009	0x0000c00000000560
0xffffffffff50:	0x000000000000000b	0x0000000000000000
0xffffffffff60:	0x000000000000000c	0x0000000000000000
0xffffffffff70:	0x000000000000000d	0x0000000000000000
0xffffffffff80:	0x000000000000000e	0x0000000000000000
0xffffffffff90:	0x0000000000000017	0x0000000000000000
(gdb)
0xffffffffffa0:	0x0000000000000033	0x0000000000001207
0xffffffffffb0:	0x000000000000000f	0x0000ffffffffffdf
0xffffffffffc0:	0x0000000000000000	0x0000000000000000
0xffffffffffd0:	0x0000000000000000	0x6100000000000000
0xffffffffffe0:	0x6800343668637261	0x69622f006f6c6c65
0xfffffffffff0:	0x006f6c6c65682f6e	0x0000000000000000

(gdb) p/x elf_bss
$39 = 0xc00000002008
(gdb) p/x elf_brk
$40 = 0xc00000002010
(gdb) x/16gx elf_bss
0xc00000002008:	0x0000000000000000	0x0000000000000000
0xc00000002018:	0x0000000000000000	0x0000000000000000

(gdb) p/x *p
$45 = {
  sz = 0xc00000003000,
  pgdir = 0xffff00003aee8000,
  kstack = 0xffff00003aeec000,
  state = 0x4,
  pid = 0x8,
  pgid = 0x8,
  sid = 0x1,
  uid = 0x0,
  euid = 0x0,
  suid = 0x0,
  fsuid = 0x0,
  gid = 0x0,
  egid = 0x0,
  sgid = 0x0,
  fsgid = 0x0,
  groups = {0x0 <repeats 32 times>},
  ngroups = 0x0,
  cap_effective = 0xfffffeff,
  cap_inheritable = 0xfffffeff,
  cap_permitted = 0xfffffeff,
  parent = 0xffff000000119e20,
  group_leader = 0xffff00000011a5d8,
  it_real_value = 0x0,
  it_prof_value = 0x0,
  it_virt_value = 0x0,
  it_real_incr = 0x0,
  it_prof_incr = 0x0,
  it_virt_incr = 0x0,
  real_timer = {
    list = {
      next = 0x0,
      prev = 0x0
    },
    expires = 0x0,
    data = 0x0,
    function = 0x0
  },
  tf = 0xffff00003aeeced0,
  context = 0xffff00003aeec1c0,
  chan = 0x0,
  killed = 0x0,
  xstate = 0x0,
  fdflag = 0x400,
  umask = 0x2,
  name = {0x2f, 0x62, 0x69, 0x6e, 0x2f, 0x68, 0x65, 0x6c, 0x6c, 0x6f, 0x0,
    0x0, 0x0, 0x0, 0x0, 0x0},
  nregions = 0x6,
  head = 0xffff00003afe0c30,
  mmap_lock = 0x0,
  ofile = {0xffff000000110078, 0xffff000000110078, 0xffff000000110078,
    0x0 <repeats 61 times>},
  cwd = 0xffff0000000dffd8,
  signal = {
    mask = 0x0,
    pending = 0x0,
    actions = {{
        sa_handler = 0x0,
        sa_mask = 0x0,
        sa_flags = 0x0,
        sa_restorer = 0x0
      }, ...}
  },
  oldtf = 0x0,
  brk_start = 0xc00000003000
}
(gdb) p/x *p->tf
$44 = {
  q0 = 0x0,
  tpidr_el0 = 0x4342f0,
  esr_el1 = 0x56000000,
  sp_el0 = 0xfffffffffea0,
  spsr_el1 = 0x80000000,
  elr_el1 = 0xc0000006454c,
  x0 = 0x1,
  x1 = 0xfffffffffea8,
  x2 = 0x0,
  ...
  x29 = 0xfffffffffa70,
  x30 = 0x404c54
}
```

# 実行ファイルにMMAPBASE, インタプリタにELF_ET_DYN_BASEの下駄を履かせた

```
# hello
== PRINT mmap_region[8] (fork): head=0xffff00003afe0f20, nregions=5 ==
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x811, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0x434000, length=0x2000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0x436000, length=0x1000, prot=0x3, flags=0x8032, f=0, offset=0x0
 - region[5]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
0x0000ffffffffffc0: AT_NULL              = 0x0000000000000000
0x0000ffffffffffb0: AT_PLATFORM          = 0x0000ffffffffffd8
0x0000fffffffffec0: AT_HWCAP             = 0x0000000000000887
0x0000fffffffffed0: AT_PAGESZ            = 0x0000000000001000
0x0000fffffffffee0: AT_CLKTCK            = 0x0000000000000064
0x0000fffffffffef0: AT_PHDR              = 0x0000600000000040
0x0000ffffffffff00: AT_PHENT             = 0x0000000000000038
0x0000ffffffffff10: AT_PHNUM             = 0x0000000000000007
0x0000ffffffffff20: AT_BASE              = 0x0000c00000000000
0x0000ffffffffff30: AT_FLAGS             = 0x0000000000000000
0x0000ffffffffff40: AT_ENTRY             = 0x0000600000000560
0x0000ffffffffff50: AT_UID               = 0x0000000000000000
0x0000ffffffffff60: AT_EUID              = 0x0000000000000000
0x0000ffffffffff70: AT_GID               = 0x0000000000000000
0x0000ffffffffff80: AT_EGID              = 0x0000000000000000
0x0000ffffffffff90: AT_SECURE            = 0x0000000000000000
0x0000ffffffffffa0: AT_MINSIGSTKS        = 0x0000000000001207
== PRINT mmap_region[8] (binfmt_elf): head=0xffff00003afe0c30, nregions=6 ==
 - region[1]: addr=0x600000000000, length=0x1000, prot=0x5, flags=0x801, f=35, offset=0x0
 - region[2]: addr=0x600000001000, length=0x2000, prot=0x3, flags=0x12, f=35, offset=0x0
 - region[3]: addr=0xc00000000000, length=0x99000, prot=0x5, flags=0x801, f=148, offset=0x0
 - region[4]: addr=0xc000000a8000, length=0x2000, prot=0x3, flags=0x12, f=148, offset=0x98000
 - region[5]: addr=0xc000000aa000, length=0x3000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[6]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
unexpected interrupt 0 at cpu 0

x27:   0x0000000000419bfc  x28:   0x0000000000433040
x29:   0x0000fffffffffd60  x30:   0x0000c00000063b10
===================== DUMP END =====================

==== stack proc[7] =========
fffffffffd20: 000000000041bc04 0000000000419bfc 0000c00000063af8 0000000000000000

同期: 不明
ESR_EL1  (例外原因)     0x0000000002000000
ELR_EL1  (例外LR)       0x0000000000000001
SPSR_EL1 (状態レジスタ) 0x0000000060000000
FAR_EL1  (フォルトaddr) 0x00000000156cdb90

# /lib/ld-musl-aarch64.so.1 /bin/hello
== PRINT mmap_region[8] (fork): head=0xffff00003afe0f20, nregions=5 ==
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x811, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0x434000, length=0x2000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0x436000, length=0x1000, prot=0x3, flags=0x8032, f=0, offset=0x0
 - region[5]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
0x0000ffffffffff90: AT_NULL              = 0x0000000000000000
0x0000ffffffffff80: AT_PLATFORM          = 0x0000ffffffffffaa
0x0000fffffffffe90: AT_HWCAP             = 0x0000000000000887
0x0000fffffffffea0: AT_PAGESZ            = 0x0000000000001000
0x0000fffffffffeb0: AT_CLKTCK            = 0x0000000000000064
0x0000fffffffffec0: AT_PHDR              = 0x0000600000000040
0x0000fffffffffed0: AT_PHENT             = 0x0000000000000038
0x0000fffffffffee0: AT_PHNUM             = 0x0000000000000007
0x0000fffffffffef0: AT_BASE              = 0x0000000000000000
0x0000ffffffffff00: AT_FLAGS             = 0x0000000000000000
0x0000ffffffffff10: AT_ENTRY             = 0x000060000006154c
0x0000ffffffffff20: AT_UID               = 0x0000000000000000
0x0000ffffffffff30: AT_EUID              = 0x0000000000000000
0x0000ffffffffff40: AT_GID               = 0x0000000000000000
0x0000ffffffffff50: AT_EGID              = 0x0000000000000000
0x0000ffffffffff60: AT_SECURE            = 0x0000000000000000
0x0000ffffffffff70: AT_MINSIGSTKS        = 0x0000000000001207
== PRINT mmap_region[8] (binfmt_elf): head=0xffff00003afe0c30, nregions=4 ==
 - region[1]: addr=0x600000000000, length=0x99000, prot=0x5, flags=0x801, f=148, offset=0x0
 - region[2]: addr=0x6000000a8000, length=0x2000, prot=0x3, flags=0x12, f=148, offset=0x98000
 - region[3]: addr=0x6000000aa000, length=0x3000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
unexpected interrupt 0 at cpu 0

x27:   0x0000000000000001  x28:   0x0000000000000000
x29:   0x0000fffffffffd30  x30:   0x0000600000063b10
===================== DUMP END =====================

==== stack proc[8] =========
fffffffffcf0: 000000000041bc04 0000000000419bfc 0000600000063af8 0000000000000000

同期: 不明
ESR_EL1  (例外原因)     0x0000000002000000
ELR_EL1  (例外LR)       0x0000000000000001
SPSR_EL1 (状態レジスタ) 0x0000000060000000
FAR_EL1  (フォルトaddr) 0x00000000156cdb90

IRQエラー: IRQ種別 (8) は未実装。

$ aarch64-linux-gnu-readelf -h obj/user_dyn/bin/hello
  Entry point address:               0x560

$ aarch64-linux-gnu-readelf -lW obj/user_dyn/bin/hello
Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  PHDR           0x000040 0x0000000000000040 0x0000000000000040 0x000188 0x000188 R   0x8
  INTERP         0x0001c8 0x00000000000001c8 0x00000000000001c8 0x00001a 0x00001a R   0x1
      [Requesting program interpreter: /lib/ld-musl-aarch64.so.1]
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x00072c 0x00072c R E 0x1000
  LOAD           0x000db8 0x0000000000001db8 0x0000000000001db8 0x000250 0x000258 RW  0x1000

$ aarch64-linux-gnu-readelf -h /usr/local/musl/lib/libc.so
  Entry point address:               0x6154c

$ aarch64-linux-gnu-readelf -lW /usr/local/musl/lib/libc.so
Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x098ac4 0x098ac4 R E 0x10000
  LOAD           0x098b90 0x00000000000a8b90 0x00000000000a8b90 0x00089a 0x003568 RW  0x10000
  DYNAMIC        0x098e08 0x00000000000a8e08 0x00000000000a8e08 0x000150 0x000150 RW  0x8
```

```
======= PROC[8]: /bin/hello TRAP FRAME DUMP =========
sp:    0x0000fffffffffd40  tpidr: 0x00000000004342f0
spsr:  0x0000000060000000  esr:   0x0000000002000000
elr:   0x0000000000000001  x0:    0x0000fffffffffe98
x1:    0x0000fffffffffec0  x2:    0x0000000000000001
x3:    0x00000000055b36e4  x4:    0x0000000000000001
x5:    0x0000000000000000  x6:    0x0000c000000ab070
x7:    0x0000000000000002  x8:    0x0000000000000000
x9:    0x0000c00000098431  x10:   0x0000000000000000
x11:   0x0000000000000000  x12:   0x0000000000000000
x13:   0x0000000000000018  x14:   0x0000000000000000
x15:   0x000000007984f81a  x16:   0x0000c000000639b0
x17:   0x0000000000000022  x18:   0x0000c00000098431
x19:   0x0000fffffffffec0  x20:   0x0000c000000ab000
x21:   0x0000fffffffffe98  x22:   0x0000c000000ab070
x23:   0x000000000041bc04  x24:   0x0000000000419bfc
x25:   0x0000000000434b98  x26:   0x0000000000434538
x27:   0x0000000000419bfc  x28:   0x0000000000433040
x29:   0x0000fffffffffd60  x30:   0x0000c00000063b10      // x30 は/usr/local/musl/lib/libc.soの__dls2()の中
===================== DUMP END =====================      // __dls2()はlibc/ldso/dynlink.cで定義

==== stack proc[8] =========
fffffffffd20: 000000000041bc04 0000000000419bfc 0000c00000063af8 0000000000000000
fffffffffd40: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffd60: 0000000000000000 0000000000000000 0000000000434ba8 0000000000434b88
fffffffffd80: 0000000000428ff8 0000000000434b98 0000000000000000 0000000000000000
fffffffffda0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffdc0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffde0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffe00: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffe20: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffe40: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffe60: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffe80: 0000000000000000 0000000000000000 0000000000000000 0000000000000001
fffffffffea0: 0000ffffffffffe0 0000000000000000 0000ffffffffffe6 0000000000000000
fffffffffec0: 0000000000000010 0000000000000887 0000000000000006 0000000000001000
fffffffffee0: 0000000000000011 0000000000000064 0000000000000003 0000600000000040
ffffffffff00: 0000000000000004 0000000000000038 0000000000000005 0000000000000007
ffffffffff20: 0000000000000007 0000c00000000000 0000000000000008 0000000000000000
ffffffffff40: 0000000000000009 0000600000000560 000000000000000b 0000000000000000
ffffffffff60: 000000000000000c 0000000000000000 000000000000000d 0000000000000000
ffffffffff80: 000000000000000e 0000000000000000 0000000000000017 0000000000000000
ffffffffffa0: 0000000000000033 0000000000001207 000000000000000f 0000ffffffffffd8
ffffffffffc0: 0000000000000000 0000000000000000 0000000000000000 0034366863726161
ffffffffffe0: 5750006f6c6c6568 69622f002e2f3d44 006f6c6c65682f6e 0000000000000000

==== stack proc[8] =========

同期: 不明
ESR_EL1  (例外原因)     0x0000000002000000
ELR_EL1  (例外LR)       0x0000000000000001
SPSR_EL1 (状態レジスタ) 0x0000000060000000
FAR_EL1  (フォルトaddr) 0x00000000156cdb90      // これは変わらず

IRQエラー: IRQ種別 (8) は未実装。
```

## dynamicブランチ

```
# hello
0xaffd8: auxv[27]=0x0
0xaffd0: auxv[26]=0x0
0xaffc8: auxv[25]=0x0
0xaffc0: auxv[24]=0x10
0xaffb8: auxv[23]=0x1000
0xaffb0: auxv[22]=0x6
0xaffa8: auxv[21]=0x64
0xaffa0: auxv[20]=0x11
0xaff98: auxv[19]=0xaaaaaaaaa040
0xaff90: auxv[18]=0x3
0xaff88: auxv[17]=0x38
0xaff80: auxv[16]=0x4
0xaff78: auxv[15]=0x7
0xaff70: auxv[14]=0x5
0xaff68: auxv[13]=0x0
0xaff60: auxv[12]=0x7
0xaff58: auxv[11]=0x0
0xaff50: auxv[10]=0x8
0xaff48: auxv[9]=0xaaaaaaaaa560
0xaff40: auxv[8]=0x9
0xaff38: auxv[7]=0x0
0xaff30: auxv[6]=0xb
0xaff28: auxv[5]=0x0
0xaff20: auxv[4]=0xc
0xaff18: auxv[3]=0x0
0xaff10: auxv[2]=0xd
0xaff08: auxv[1]=0x0
0xaff00: auxv[0]=0xe
hello dynamic world!
```

# linux

```
$ cat maps
aaaac3809000-aaaac380a000 r-xp 00000000 b3:02 1647510      /home/dspace/work/aarch64/hello
aaaac3819000-aaaac381a000 r--p 00000000 b3:02 1647510      /home/dspace/work/aarch64/hello
aaaac381a000-aaaac381b000 rw-p 00001000 b3:02 1647510      /home/dspace/work/aarch64/hello
aaaadee2d000-aaaadee4e000 rw-p 00000000 00:00 0            [heap]
ffffaf46a000-ffffaf5c4000 r-xp 00000000 b3:02 6291         /usr/lib/aarch64-linux-gnu/libc-2.31.so
ffffaf5c4000-ffffaf5d4000 ---p 0015a000 b3:02 6291         /usr/lib/aarch64-linux-gnu/libc-2.31.so
ffffaf5d4000-ffffaf5d8000 r--p 0015a000 b3:02 6291         /usr/lib/aarch64-linux-gnu/libc-2.31.so
ffffaf5d8000-ffffaf5da000 rw-p 0015e000 b3:02 6291         /usr/lib/aarch64-linux-gnu/libc-2.31.so
ffffaf5da000-ffffaf5dd000 rw-p 00000000 00:00 0
ffffaf5dd000-ffffaf5fe000 r-xp 00000000 b3:02 6287         /usr/lib/aarch64-linux-gnu/ld-2.31.so
ffffaf603000-ffffaf605000 rw-p 00000000 00:00 0
ffffaf60c000-ffffaf60d000 r--p 00000000 00:00 0            [vvar]
ffffaf60d000-ffffaf60e000 r-xp 00000000 00:00 0            [vdso]
ffffaf60e000-ffffaf60f000 r--p 00021000 b3:02 6287         /usr/lib/aarch64-linux-gnu/ld-2.31.so
ffffaf60f000-ffffaf611000 rw-p 00022000 b3:02 6287         /usr/lib/aarch64-linux-gnu/ld-2.31.so
ffffee34f000-ffffee370000 rw-p 00000000 00:00 0            [stack]

$ LD_SHOW_AUXV=1 ./hello
AT_SYSINFO_EHDR:      0xffffaf60d000
AT_??? (0x33): 0x1270
AT_HWCAP:             887
AT_PAGESZ:            4096
AT_CLKTCK:            100
AT_PHDR:              0xaaaac3809040
AT_PHENT:             56
AT_PHNUM:             9
AT_BASE:              0xffffaf5dd000
AT_FLAGS:             0x0
AT_ENTRY:             0xaaaac38096a0
AT_UID:               1002
AT_EUID:              1002
AT_GID:               1002
AT_EGID:              1002
AT_SECURE:            0
AT_RANDOM:            0xffffee36e928
AT_HWCAP2:            0x0
AT_EXECFN:            ./hello
AT_PLATFORM:          aarch64
hello, world
hello, again

$ readelf -h hello
ELF Header:
  Type:                              DYN (Shared object file)
  Entry point address:               0x6a0
  Start of program headers:          64 (bytes into file)

$ readelf -lW hello
Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  PHDR           0x000040 0x0000000000000040 0x0000000000000040 0x0001f8 0x0001f8 R   0x8
  INTERP         0x000238 0x0000000000000238 0x0000000000000238 0x00001b 0x00001b R   0x1
      [Requesting program interpreter: /lib/ld-linux-aarch64.so.1]
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x0009cc 0x0009cc R E 0x10000
  LOAD           0x000d78 0x0000000000010d78 0x0000000000010d78 0x000298 0x0002a0 RW  0x10000

 Section to Segment mapping:
  Segment Sections...
   00
   01     .interp
   02     .interp .note.gnu.build-id .note.ABI-tag .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt .text .fini .rodata .eh_frame_hdr .eh_frame
   03     .init_array .fini_array .dynamic .got .data .bss

$ readelf -h /lib/ld-linux-aarch64.so.1
ELF Header:
  Type:                              DYN (Shared object file)
  Entry point address:               0x10c0

$ readelf -lW /lib/ld-linux-aarch64.so.1
Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x020768 0x020768 R E 0x10000
  LOAD           0x021590 0x0000000000031590 0x0000000000031590 0x001b08 0x001c70 RW  0x10000

 Section to Segment mapping:
  Segment Sections...
   00     .note.gnu.build-id .hash .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_d .rela.dyn .rela.plt .plt .text .rodata .stapsdt.base .eh_frame_hdr .eh_frame
   01     .data.rel.ro .dynamic .got .got.plt .data .bss
```
