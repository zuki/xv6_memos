# `exec.c`の関数

## 公開関数

- `static struct linux_binfmt *formats`
  - 登録フォーマットリスト
- `int register_binfmt(struct linux_binfmt * fmt)`
  - フォーマットの登録（成功は0）
- `int unregister_binfmt(struct linux_binfmt * fmt)`
  - フォーマットの削除（成功は0）
- `asmlinkage long sys_uselib(const char * library)`
  - 共有ファイルを使用するためのシステムコール（xv6では実装しない）
- `int copy_strings(int argc,char ** argv, struct linux_binprm *bprm)`
  - 引数/環境変数文字列をユーザメモリからカーネルメモリに（新規ユーザメモリにそのまま置ける形で）コピーする
- `int copy_strings_kernel(int argc,char ** argv, struct linux_binprm *bprm)`
  - argvとその値をカーネルメモリから取得する
- `void put_dirty_page(struct task_struct * tsk, struct page *page, unsigned long address)`
  - アドレス空間にページをマップする（address用のページディスクリプタを作成（pmd, pte, *pte））
- `int setup_arg_pages(struct linux_binprm *bprm)`
  - vm_areaを一つ作成し
  - put_dirty_pageを利用してbprmにセットされているargv, envpをマッピングする
- `struct file *open_exec(const char *name)`
  - パス名nameのfile構造体を取得する
- `int kernel_read(struct file *file, unsigned long offset, char * addr, unsigned long count)`
  - カーネルモーでファイル読み込み
- `int flush_old_exec(struct linux_binprm * bprm)`
  - 親から受け継いだ古いexec情報を開放する
  - make_private_signals(), exec_mmap(), release_old_signals(oldsig), flush_thread(), de_thread(current), flush_signal_handlers(current), flush_old_files(current->files)
- `int prepare_binprm(struct linux_binprm *bprm)`
  - binprm構造体のcapabilityをセットして、バッファにファイルの先頭128バイトを読み込む
- `void compute_creds(struct linux_binprm *bprm)`
  - 古いidとファイルのcapabilitiesから新しいidとcapabilitiesを作成する
- `void remove_arg_zero(struct linux_binprm *bprm)`
  - bprm中のargvを0クリアする
- `int search_binary_handler(struct linux_binprm *bprm,struct pt_regs *regs)`
  - バイナリフォーマットリストのハンドラを順番に実行して成功したら戻る
- `int do_execve(char * filename, char ** argv, char ** envp, struct pt_regs * regs)`
  - execveシステムコールの実行関数
- `void set_binfmt(struct linux_binfmt *new)`
  - カレントプロセスのbinfmtを新しいformatに置き換える
- `int do_coredump(long signr, struct pt_regs * regs)`
  - (xv6では実装しない)

## 非公開関数

- `static int count(char ** argv, int max)`
  - 引数配列や環境変数配列の個数を返す
- `static int exec_mmap(void)`
  - 新規コンテキストを作成して、実行
- `static inline int make_private_signals(void)`
  - プロセス独自シグナルテーブルを保存する（xv6では実装しない）
- `static inline void release_old_signals(struct signal_struct * oldsig)`
  - シグナルテーブルを開放する(xv6では実装しない)
- `static inline void flush_old_files(struct files_struct * files)`
  - close_on_execのファイルを閉じる
- `static inline void de_thread(struct task_struct *tsk)`
  - スレッドグループを解除すする
- `static inline int must_not_trace_exec(struct task_struct * p)`
  - (xv6では実装しない)

# `execve`の処理概要

1. filenameからfile構造体を取得
2. bprm構造体を構成する
3. flename, argv, argcをbprmにコピーする
4. バイナリハンドラリストのハンドラを成功するまで順に実行する

# `load_elf_binary`の処理概要

1. elfヘッダのチェック
2. プログラムヘッダのチェック
3. ファイルを開く（カレントプロセスに登録）
4. インタプリタの有無をチェック（実行ファイルがstatic linkかdyanamic linkか）
   - インタプリタをオープン
   - インタプリタのelfヘッダの読み込み
5. カレントプロセスの不要なデータを削除: flush_old_exec
6. 種別（PT_LOAD）のプログラムヘッダを処理
   - prot (PROT_READ/WRITE/EXEC) の設定
   - flags (MAP_PRIVATE|MAP_DENYWRITE|MAP_EXECUTABLE)の設定
     - staticな実行ファイルは flags + MAP_FIXED
   - dynamicの場合、addrsにbiasをかませる (ELF_ET_DYN_BASE)
   - processのvmaにmapping: elf_map
7. elf_entry = elf.e_entry + load_bias : (staticの場合は, load_bias=0であり、elf_entry=elf.e_entry)
8. インタプリタがある場合、インタプリタの読み込み
   - elf_entry = インタプリタのマッピングアドレス（dynamicの場合は、elf_entryはインタプリタのe_entryをmmapしたアドレス）
   - インタプリタの開放
9. プログラムヘッダの開放
10. インタプリタのclose
11. argv, envp, auxをスタックに設定: create_elf_tables
12. current->mmの設定（xv6にはなし）
13. elf_bss, elf_brkの調整とelf_bssの0クリア
14. elf_entryから実行開始

# dynamic linkしたユーザアプリケーション

```
$ aarch64-linux-gnu-readelf -aW obj/user_dyn/bin/hello
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x560
  Start of program headers:          64 (bytes into file)
  Start of section headers:          6576 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         7
  Size of section headers:           64 (bytes)
  Number of section headers:         24
  Section header string table index: 23

Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        00000000000001c8 0001c8 00001a 00   A  0   0  1
  [ 2] .hash             HASH            00000000000001e8 0001e8 00003c 04   A  4   0  8
  [ 3] .gnu.hash         GNU_HASH        0000000000000228 000228 000028 00   A  4   0  8
  [ 4] .dynsym           DYNSYM          0000000000000250 000250 0000f0 18   A  5   3  8
  [ 5] .dynstr           STRTAB          0000000000000340 000340 000071 00   A  0   0  1
  [ 6] .rela.dyn         RELA            00000000000003b8 0003b8 0000d8 18   A  4   0  8
  [ 7] .rela.plt         RELA            0000000000000490 000490 000048 18  AI  4  17  8
  [ 8] .init             PROGBITS        00000000000004d8 0004d8 000010 00  AX  0   0  4
  [ 9] .plt              PROGBITS        00000000000004f0 0004f0 000050 10  AX  0   0 16
  [10] .text             PROGBITS        0000000000000540 000540 000124 00  AX  0   0  8
  [11] .fini             PROGBITS        0000000000000664 000664 000010 00  AX  0   0  4
  [12] .rodata           PROGBITS        0000000000000678 000678 000015 01 AMS  0   0  8
  [13] .eh_frame         PROGBITS        0000000000000690 000690 00009c 00   A  0   0  8
  [14] .init_array       INIT_ARRAY      0000000000001db8 000db8 000008 08  WA  0   0  8
  [15] .fini_array       FINI_ARRAY      0000000000001dc0 000dc0 000008 08  WA  0   0  8
  [16] .dynamic          DYNAMIC         0000000000001dc8 000dc8 0001d0 10  WA  5   0  8
  [17] .got              PROGBITS        0000000000001f98 000f98 000068 08  WA  0   0  8
  [18] .data             PROGBITS        0000000000002000 001000 000008 00  WA  0   0  8
  [19] .bss              NOBITS          0000000000002008 001008 000008 00  WA  0   0  1
  [20] .comment          PROGBITS        0000000000000000 001008 00002a 01  MS  0   0  1
  [21] .symtab           SYMTAB          0000000000000000 001038 0006f0 18     22  55  8
  [22] .strtab           STRTAB          0000000000000000 001728 0001d4 00      0   0  1
  [23] .shstrtab         STRTAB          0000000000000000 0018fc 0000af 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  p (processor specific)

There are no section groups in this file.

Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  PHDR           0x000040 0x0000000000000040 0x0000000000000040 0x000188 0x000188 R   0x8
  INTERP         0x0001c8 0x00000000000001c8 0x00000000000001c8 0x00001a 0x00001a R   0x1
      [Requesting program interpreter: /lib/ld-musl-aarch64.so.1]
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x00072c 0x00072c R E 0x1000
  LOAD           0x000db8 0x0000000000001db8 0x0000000000001db8 0x000250 0x000258 RW  0x1000
  DYNAMIC        0x000dc8 0x0000000000001dc8 0x0000000000001dc8 0x0001d0 0x0001d0 RW  0x8
  GNU_STACK      0x000000 0x0000000000000000 0x0000000000000000 0x000000 0x000000 RW  0x10
  GNU_RELRO      0x000db8 0x0000000000001db8 0x0000000000001db8 0x000248 0x000248 R   0x1

 Section to Segment mapping:
  Segment Sections...
   00
   01     .interp
   02     .interp .hash .gnu.hash .dynsym .dynstr .rela.dyn .rela.plt .init .plt .text .fini .rodata .eh_frame
   03     .init_array .fini_array .dynamic .got .data .bss
   04     .dynamic
   05
   06     .init_array .fini_array .dynamic .got

Dynamic section at offset 0xdc8 contains 25 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libc.so]
 0x000000000000000c (INIT)               0x4d8
 0x000000000000000d (FINI)               0x664
 0x0000000000000019 (INIT_ARRAY)         0x1db8
 0x000000000000001b (INIT_ARRAYSZ)       8 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x1dc0
 0x000000000000001c (FINI_ARRAYSZ)       8 (bytes)
 0x0000000000000004 (HASH)               0x1e8
 0x000000006ffffef5 (GNU_HASH)           0x228
 0x0000000000000005 (STRTAB)             0x340
 0x0000000000000006 (SYMTAB)             0x250
 0x000000000000000a (STRSZ)              113 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000015 (DEBUG)              0x0
 0x0000000000000003 (PLTGOT)             0x1f98
 0x0000000000000002 (PLTRELSZ)           72 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x490
 0x0000000000000007 (RELA)               0x3b8
 0x0000000000000008 (RELASZ)             216 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000000000001e (FLAGS)              BIND_NOW
 0x000000006ffffffb (FLAGS_1)            Flags: NOW PIE
 0x000000006ffffff9 (RELACOUNT)          6
 0x0000000000000000 (NULL)               0x0

Relocation section '.rela.dyn' at offset 0x3b8 contains 9 entries:
    Offset             Info             Type               Symbol's Value  Symbol's Name + Addend
0000000000001db8  0000000000000403 R_AARCH64_RELATIVE                        660
0000000000001dc0  0000000000000403 R_AARCH64_RELATIVE                        618
0000000000001fd8  0000000000000403 R_AARCH64_RELATIVE                        4d8
0000000000001ff0  0000000000000403 R_AARCH64_RELATIVE                        540
0000000000001ff8  0000000000000403 R_AARCH64_RELATIVE                        664
0000000000002000  0000000000000403 R_AARCH64_RELATIVE                        2000
0000000000001fd0  0000000400000401 R_AARCH64_GLOB_DAT     0000000000000000 __cxa_finalize + 0
0000000000001fe0  0000000500000401 R_AARCH64_GLOB_DAT     0000000000000000 _ITM_registerTMCloneTable + 0
0000000000001fe8  0000000600000401 R_AARCH64_GLOB_DAT     0000000000000000 _ITM_deregisterTMCloneTable + 0

Relocation section '.rela.plt' at offset 0x490 contains 3 entries:
    Offset             Info             Type               Symbol's Value  Symbol's Name + Addend
0000000000001fb0  0000000300000402 R_AARCH64_JUMP_SLOT    0000000000000000 puts + 0
0000000000001fb8  0000000400000402 R_AARCH64_JUMP_SLOT    0000000000000000 __cxa_finalize + 0
0000000000001fc0  0000000700000402 R_AARCH64_JUMP_SLOT    0000000000000000 __libc_start_main + 0

The decoding of unwind sections for machine type AArch64 is not currently supported.

Symbol table '.dynsym' contains 10 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 00000000000004d8     0 SECTION LOCAL  DEFAULT    8
     2: 0000000000002000     0 SECTION LOCAL  DEFAULT   18
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts
     4: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND __cxa_finalize
     5: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
     6: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTable
     7: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main
     8: 00000000000004d8     4 FUNC    GLOBAL DEFAULT    8 _init
     9: 0000000000000664     4 FUNC    GLOBAL DEFAULT   11 _fini

Symbol table '.symtab' contains 74 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 00000000000001c8     0 SECTION LOCAL  DEFAULT    1
     2: 00000000000001e8     0 SECTION LOCAL  DEFAULT    2
     3: 0000000000000228     0 SECTION LOCAL  DEFAULT    3
     4: 0000000000000250     0 SECTION LOCAL  DEFAULT    4
     5: 0000000000000340     0 SECTION LOCAL  DEFAULT    5
     6: 00000000000003b8     0 SECTION LOCAL  DEFAULT    6
     7: 0000000000000490     0 SECTION LOCAL  DEFAULT    7
     8: 00000000000004d8     0 SECTION LOCAL  DEFAULT    8
     9: 00000000000004f0     0 SECTION LOCAL  DEFAULT    9
    10: 0000000000000540     0 SECTION LOCAL  DEFAULT   10
    11: 0000000000000664     0 SECTION LOCAL  DEFAULT   11
    12: 0000000000000678     0 SECTION LOCAL  DEFAULT   12
    13: 0000000000000690     0 SECTION LOCAL  DEFAULT   13
    14: 0000000000001db8     0 SECTION LOCAL  DEFAULT   14
    15: 0000000000001dc0     0 SECTION LOCAL  DEFAULT   15
    16: 0000000000001dc8     0 SECTION LOCAL  DEFAULT   16
    17: 0000000000001f98     0 SECTION LOCAL  DEFAULT   17
    18: 0000000000002000     0 SECTION LOCAL  DEFAULT   18
    19: 0000000000002008     0 SECTION LOCAL  DEFAULT   19
    20: 0000000000000000     0 SECTION LOCAL  DEFAULT   20
    21: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS Scrt1.c
    22: 0000000000000560     0 NOTYPE  LOCAL  DEFAULT   10 $x
    23: 000000000000057c     0 NOTYPE  LOCAL  DEFAULT   10 $x
    24: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /usr/local/musl/lib/crti.o
    25: 00000000000004d8     0 NOTYPE  LOCAL  DEFAULT    8 $x
    26: 0000000000000664     0 NOTYPE  LOCAL  DEFAULT   11 $x
    27: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /usr/local/musl/lib/crtn.o
    28: 00000000000004e0     0 NOTYPE  LOCAL  DEFAULT    8 $x
    29: 000000000000066c     0 NOTYPE  LOCAL  DEFAULT   11 $x
    30: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main.c
    31: 0000000000000678     0 NOTYPE  LOCAL  DEFAULT   12 $d
    32: 0000000000000540     0 NOTYPE  LOCAL  DEFAULT   10 $x
    33: 0000000000000708     0 NOTYPE  LOCAL  DEFAULT   13 $d
    34: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    35: 00000000000005a8     0 NOTYPE  LOCAL  DEFAULT   10 $x
    36: 00000000000005a8     0 FUNC    LOCAL  DEFAULT   10 deregister_tm_clones
    37: 00000000000005d8     0 FUNC    LOCAL  DEFAULT   10 register_tm_clones
    38: 0000000000002000     0 NOTYPE  LOCAL  DEFAULT   18 $d
    39: 0000000000000618     0 FUNC    LOCAL  DEFAULT   10 __do_global_dtors_aux
    40: 0000000000002008     1 OBJECT  LOCAL  DEFAULT   19 completed.9126
    41: 0000000000001dc0     0 NOTYPE  LOCAL  DEFAULT   15 $d
    42: 0000000000001dc0     0 OBJECT  LOCAL  DEFAULT   15 __do_global_dtors_aux_fini_array_entry
    43: 0000000000000660     0 FUNC    LOCAL  DEFAULT   10 frame_dummy
    44: 0000000000001db8     0 NOTYPE  LOCAL  DEFAULT   14 $d
    45: 0000000000001db8     0 OBJECT  LOCAL  DEFAULT   14 __frame_dummy_init_array_entry
    46: 00000000000006a4     0 NOTYPE  LOCAL  DEFAULT   13 $d
    47: 0000000000002008     0 NOTYPE  LOCAL  DEFAULT   19 $d
    48: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    49: 0000000000000728     0 NOTYPE  LOCAL  DEFAULT   13 $d
    50: 0000000000000728     0 OBJECT  LOCAL  DEFAULT   13 __FRAME_END__
    51: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS
    52: 0000000000001dc8     0 OBJECT  LOCAL  DEFAULT  ABS _DYNAMIC
    53: 0000000000001fc8     0 OBJECT  LOCAL  DEFAULT  ABS _GLOBAL_OFFSET_TABLE_
    54: 00000000000004f0     0 NOTYPE  LOCAL  DEFAULT    9 $x
    55: 0000000000002010     0 NOTYPE  GLOBAL DEFAULT   19 _bss_end__
    56: 0000000000002008     0 OBJECT  GLOBAL HIDDEN    18 __TMC_END__
    57: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts
    58: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND __cxa_finalize
    59: 0000000000002008     0 NOTYPE  GLOBAL DEFAULT   19 __bss_start__
    60: 0000000000002000     0 OBJECT  GLOBAL HIDDEN    18 __dso_handle
    61: 00000000000004d8     4 FUNC    GLOBAL DEFAULT    8 _init
    62: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
    63: 0000000000002010     0 NOTYPE  GLOBAL DEFAULT   19 __bss_end__
    64: 0000000000000560     0 FUNC    GLOBAL DEFAULT   10 _start
    65: 000000000000057c    40 FUNC    GLOBAL DEFAULT   10 _start_c
    66: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTable
    67: 0000000000002008     0 NOTYPE  GLOBAL DEFAULT   19 __bss_start
    68: 0000000000000540    32 FUNC    GLOBAL DEFAULT   10 main
    69: 0000000000002010     0 NOTYPE  GLOBAL DEFAULT   19 __end__
    70: 0000000000000664     4 FUNC    GLOBAL DEFAULT   11 _fini
    71: 0000000000002008     0 NOTYPE  GLOBAL DEFAULT   18 _edata
    72: 0000000000002010     0 NOTYPE  GLOBAL DEFAULT   19 _end
    73: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main

Histogram for bucket list length (total of 3 buckets):
 Length  Number     % of total  Coverage
      0  0          (  0.0%)
      1  2          ( 66.7%)     28.6%
      2  0          (  0.0%)     28.6%
      3  0          (  0.0%)     28.6%
      4  0          (  0.0%)     28.6%
      5  1          ( 33.3%)    100.0%

Histogram for `.gnu.hash' bucket list length (total of 2 buckets):
 Length  Number     % of total  Coverage
      0  1          ( 50.0%)
      1  0          (  0.0%)      0.0%
      2  1          ( 50.0%)    100.0%

No version information found in this file.
```

# static linkしたユーザアプリケーション

```
$ aarch64-linux-gnu-readelf -aW obj/user/bin/hellos
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
  Entry point address:               0x400178
  Start of program headers:          64 (bytes into file)
  Start of section headers:          15656 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         4
  Size of section headers:           64 (bytes)
  Number of section headers:         17
  Section header string table index: 16

Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
  [ 1] .init             PROGBITS        0000000000400120 000120 000010 00  AX  0   0  4
  [ 2] .text             PROGBITS        0000000000400130 000130 00122c 00  AX  0   0 16
  [ 3] .fini             PROGBITS        000000000040135c 00135c 000010 00  AX  0   0  4
  [ 4] .rodata           PROGBITS        0000000000401370 001370 00001e 01 AMS  0   0  8
  [ 5] .eh_frame         PROGBITS        0000000000401390 001390 00009c 00   A  0   0  8
  [ 6] .init_array       INIT_ARRAY      0000000000402f48 001f48 000008 08  WA  0   0  8
  [ 7] .fini_array       FINI_ARRAY      0000000000402f50 001f50 000008 08  WA  0   0  8
  [ 8] .data.rel.ro      PROGBITS        0000000000402f58 001f58 000010 00  WA  0   0  8
  [ 9] .got              PROGBITS        0000000000402f68 001f68 000080 08  WA  0   0  8
  [10] .got.plt          PROGBITS        0000000000402fe8 001fe8 000018 08  WA  0   0  8
  [11] .data             PROGBITS        0000000000403000 002000 000100 00  WA  0   0  8
  [12] .bss              NOBITS          0000000000403100 002100 000640 00  WA  0   0  8
  [13] .comment          PROGBITS        0000000000000000 002100 00002a 01  MS  0   0  1
  [14] .symtab           SYMTAB          0000000000000000 002130 001530 18     15 149  8
  [15] .strtab           STRTAB          0000000000000000 003660 00063c 00      0   0  1
  [16] .shstrtab         STRTAB          0000000000000000 003c9c 000086 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  p (processor specific)

There are no section groups in this file.

Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  LOAD           0x000000 0x0000000000400000 0x0000000000400000 0x00142c 0x00142c R E 0x1000
  LOAD           0x001f48 0x0000000000402f48 0x0000000000402f48 0x0001b8 0x0007f8 RW  0x1000
  GNU_STACK      0x000000 0x0000000000000000 0x0000000000000000 0x000000 0x000000 RW  0x10
  GNU_RELRO      0x001f48 0x0000000000402f48 0x0000000000402f48 0x0000b8 0x0000b8 R   0x1

 Section to Segment mapping:
  Segment Sections...
   00     .init .text .fini .rodata .eh_frame
   01     .init_array .fini_array .data.rel.ro .got .got.plt .data .bss
   02
   03     .init_array .fini_array .data.rel.ro .got .got.plt

There is no dynamic section in this file.

There are no relocations in this file.

The decoding of unwind sections for machine type AArch64 is not currently supported.

Symbol table '.symtab' contains 226 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 0000000000400120     0 SECTION LOCAL  DEFAULT    1
     2: 0000000000400130     0 SECTION LOCAL  DEFAULT    2
     3: 000000000040135c     0 SECTION LOCAL  DEFAULT    3
     4: 0000000000401370     0 SECTION LOCAL  DEFAULT    4
     5: 0000000000401390     0 SECTION LOCAL  DEFAULT    5
     6: 0000000000402f48     0 SECTION LOCAL  DEFAULT    6
     7: 0000000000402f50     0 SECTION LOCAL  DEFAULT    7
     8: 0000000000402f58     0 SECTION LOCAL  DEFAULT    8
     9: 0000000000402f68     0 SECTION LOCAL  DEFAULT    9
    10: 0000000000402fe8     0 SECTION LOCAL  DEFAULT   10
    11: 0000000000403000     0 SECTION LOCAL  DEFAULT   11
    12: 0000000000403100     0 SECTION LOCAL  DEFAULT   12
    13: 0000000000000000     0 SECTION LOCAL  DEFAULT   13
    14: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /usr/local/musl/lib/crti.o
    15: 0000000000400120     0 NOTYPE  LOCAL  DEFAULT    1 $x
    16: 000000000040135c     0 NOTYPE  LOCAL  DEFAULT    3 $x
    17: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /usr/local/musl/lib/crtn.o
    18: 0000000000400128     0 NOTYPE  LOCAL  DEFAULT    1 $x
    19: 0000000000401364     0 NOTYPE  LOCAL  DEFAULT    3 $x
    20: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS exit.c
    21: 00000000004004ec     0 NOTYPE  LOCAL  DEFAULT    2 $x
    22: 00000000004004ec     4 FUNC    LOCAL  DEFAULT    2 dummy
    23: 00000000004004f0     0 NOTYPE  LOCAL  DEFAULT    2 $x
    24: 00000000004004f0    56 FUNC    LOCAL  DEFAULT    2 libc_exit_fini
    25: 0000000000400130     0 NOTYPE  LOCAL  DEFAULT    2 $x
    26: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main.c
    27: 0000000000401370     0 NOTYPE  LOCAL  DEFAULT    4 $d
    28: 0000000000400150     0 NOTYPE  LOCAL  DEFAULT    2 $x
    29: 0000000000401408     0 NOTYPE  LOCAL  DEFAULT    5 $d
    30: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS Scrt1.c
    31: 0000000000400178     0 NOTYPE  LOCAL  DEFAULT    2 $x
    32: 0000000000400194     0 NOTYPE  LOCAL  DEFAULT    2 $x
    33: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    34: 00000000004001c0     0 NOTYPE  LOCAL  DEFAULT    2 $x
    35: 00000000004001c0     0 FUNC    LOCAL  DEFAULT    2 deregister_tm_clones
    36: 00000000004001f0     0 FUNC    LOCAL  DEFAULT    2 register_tm_clones
    37: 0000000000403000     0 NOTYPE  LOCAL  DEFAULT   11 $d
    38: 0000000000400230     0 FUNC    LOCAL  DEFAULT    2 __do_global_dtors_aux
    39: 0000000000403100     1 OBJECT  LOCAL  DEFAULT   12 completed.9126
    40: 0000000000402f50     0 NOTYPE  LOCAL  DEFAULT    7 $d
    41: 0000000000402f50     0 OBJECT  LOCAL  DEFAULT    7 __do_global_dtors_aux_fini_array_entry
    42: 0000000000400278     0 FUNC    LOCAL  DEFAULT    2 frame_dummy
    43: 0000000000402f48     0 NOTYPE  LOCAL  DEFAULT    6 $d
    44: 0000000000402f48     0 OBJECT  LOCAL  DEFAULT    6 __frame_dummy_init_array_entry
    45: 00000000004013a4     0 NOTYPE  LOCAL  DEFAULT    5 $d
    46: 0000000000403100     0 NOTYPE  LOCAL  DEFAULT   12 $d
    47: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __libc_start_main.c
    48: 000000000040027c     0 NOTYPE  LOCAL  DEFAULT    2 $x
    49: 000000000040027c     4 FUNC    LOCAL  DEFAULT    2 dummy
    50: 0000000000400280     0 NOTYPE  LOCAL  DEFAULT    2 $x
    51: 0000000000400280     4 FUNC    LOCAL  DEFAULT    2 dummy1
    52: 0000000000400284     0 NOTYPE  LOCAL  DEFAULT    2 $x
    53: 0000000000400430     0 NOTYPE  LOCAL  DEFAULT    2 $x
    54: 0000000000400430    60 FUNC    LOCAL  DEFAULT    2 libc_start_init
    55: 000000000040046c     0 NOTYPE  LOCAL  DEFAULT    2 $x
    56: 000000000040046c    56 FUNC    LOCAL  DEFAULT    2 libc_start_main_stage2
    57: 00000000004004a4     0 NOTYPE  LOCAL  DEFAULT    2 $x
    58: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS defsysinfo.c
    59: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS libc.c
    60: 0000000000403108     0 NOTYPE  LOCAL  DEFAULT   12 $d
    61: 0000000000403110     0 NOTYPE  LOCAL  DEFAULT   12 $d
    62: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS puts.c
    63: 0000000000400528     0 NOTYPE  LOCAL  DEFAULT    2 $x
    64: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS stdout.c
    65: 0000000000403118  1032 OBJECT  LOCAL  DEFAULT   12 buf
    66: 0000000000403118     0 NOTYPE  LOCAL  DEFAULT   12 $d
    67: 0000000000403008     0 NOTYPE  LOCAL  DEFAULT   11 $d
    68: 00000000004030f0     0 NOTYPE  LOCAL  DEFAULT   11 $d
    69: 0000000000402f58     0 NOTYPE  LOCAL  DEFAULT    8 $d
    70: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS memset.lo
    71: 00000000004005e0     0 NOTYPE  LOCAL  DEFAULT    2 $x
    72: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __environ.c
    73: 0000000000403520     0 NOTYPE  LOCAL  DEFAULT   12 $d
    74: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __init_tls.c
    75: 00000000004006e4     0 NOTYPE  LOCAL  DEFAULT    2 $x
    76: 0000000000400764     0 NOTYPE  LOCAL  DEFAULT    2 $x
    77: 0000000000400814     0 NOTYPE  LOCAL  DEFAULT    2 $x
    78: 0000000000400814   428 FUNC    LOCAL  DEFAULT    2 static_init_tls
    79: 0000000000403528     0 NOTYPE  LOCAL  DEFAULT   12 $d
    80: 0000000000403528   336 OBJECT  LOCAL  DEFAULT   12 builtin_tls
    81: 0000000000403678     0 NOTYPE  LOCAL  DEFAULT   12 $d
    82: 0000000000403678    48 OBJECT  LOCAL  DEFAULT   12 main_tls
    83: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS _Exit.c
    84: 00000000004009c0     0 NOTYPE  LOCAL  DEFAULT    2 $x
    85: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __lockfile.c
    86: 00000000004009d8     0 NOTYPE  LOCAL  DEFAULT    2 $x
    87: 0000000000400a88     0 NOTYPE  LOCAL  DEFAULT    2 $x
    88: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __overflow.c
    89: 0000000000400acc     0 NOTYPE  LOCAL  DEFAULT    2 $x
    90: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __stdio_close.c
    91: 0000000000400b44     0 NOTYPE  LOCAL  DEFAULT    2 $x
    92: 0000000000400b44     4 FUNC    LOCAL  DEFAULT    2 dummy
    93: 0000000000400b48     0 NOTYPE  LOCAL  DEFAULT    2 $x
    94: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __stdio_seek.c
    95: 0000000000400b6c     0 NOTYPE  LOCAL  DEFAULT    2 $x
    96: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __stdout_write.c
    97: 0000000000400b74     0 NOTYPE  LOCAL  DEFAULT    2 $x
    98: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __towrite.c
    99: 0000000000400bd0     0 NOTYPE  LOCAL  DEFAULT    2 $x
   100: 0000000000400c18     0 NOTYPE  LOCAL  DEFAULT    2 $x
   101: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS fputs.c
   102: 0000000000400c1c     0 NOTYPE  LOCAL  DEFAULT    2 $x
   103: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS fwrite.c
   104: 0000000000400c5c     0 NOTYPE  LOCAL  DEFAULT    2 $x
   105: 0000000000400d48     0 NOTYPE  LOCAL  DEFAULT    2 $x
   106: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS memcpy.lo
   107: 0000000000400dd0     0 NOTYPE  LOCAL  DEFAULT    2 $x
   108: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS strlen.c
   109: 0000000000400f60     0 NOTYPE  LOCAL  DEFAULT    2 $x
   110: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __set_thread_area.lo
   111: 0000000000400fdc     0 NOTYPE  LOCAL  DEFAULT    2 $x
   112: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS default_attr.c
   113: 00000000004030f8     0 NOTYPE  LOCAL  DEFAULT   11 $d
   114: 00000000004030fc     0 NOTYPE  LOCAL  DEFAULT   11 $d
   115: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS lseek.c
   116: 0000000000400fe8     0 NOTYPE  LOCAL  DEFAULT    2 $x
   117: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS syscall_ret.c
   118: 0000000000401000     0 NOTYPE  LOCAL  DEFAULT    2 $x
   119: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __stdio_exit.c
   120: 0000000000401030     0 NOTYPE  LOCAL  DEFAULT    2 $x
   121: 0000000000401030   112 FUNC    LOCAL  DEFAULT    2 close_file
   122: 00000000004010a0     0 NOTYPE  LOCAL  DEFAULT    2 $x
   123: 00000000004036a8     8 OBJECT  LOCAL  DEFAULT   12 dummy_file
   124: 00000000004036a8     0 NOTYPE  LOCAL  DEFAULT   12 $d
   125: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __stdio_write.c
   126: 00000000004010e8     0 NOTYPE  LOCAL  DEFAULT    2 $x
   127: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS ofl.c
   128: 00000000004011cc     0 NOTYPE  LOCAL  DEFAULT    2 $x
   129: 00000000004011ec     0 NOTYPE  LOCAL  DEFAULT    2 $x
   130: 00000000004036b0     0 NOTYPE  LOCAL  DEFAULT   12 $d
   131: 00000000004036b0     8 OBJECT  LOCAL  DEFAULT   12 ofl_head
   132: 00000000004036b8     0 NOTYPE  LOCAL  DEFAULT   12 $d
   133: 00000000004036b8     4 OBJECT  LOCAL  DEFAULT   12 ofl_lock
   134: 0000000000402f60     0 NOTYPE  LOCAL  DEFAULT    8 $d
   135: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __lock.c
   136: 00000000004011f8     0 NOTYPE  LOCAL  DEFAULT    2 $x
   137: 00000000004012f8     0 NOTYPE  LOCAL  DEFAULT    2 $x
   138: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS __errno_location.c
   139: 0000000000401350     0 NOTYPE  LOCAL  DEFAULT    2 $x
   140: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
   141: 0000000000401428     0 NOTYPE  LOCAL  DEFAULT    5 $d
   142: 0000000000401428     0 OBJECT  LOCAL  DEFAULT    5 __FRAME_END__
   143: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS
   144: 0000000000402f58     0 NOTYPE  LOCAL  DEFAULT    7 __fini_array_end
   145: 0000000000402f50     0 NOTYPE  LOCAL  DEFAULT    7 __fini_array_start
   146: 0000000000402f50     0 NOTYPE  LOCAL  DEFAULT    6 __init_array_end
   147: 0000000000402f68     0 OBJECT  LOCAL  DEFAULT    9 _GLOBAL_OFFSET_TABLE_
   148: 0000000000402f48     0 NOTYPE  LOCAL  DEFAULT    6 __init_array_start
   149: 0000000000403738     4 OBJECT  GLOBAL HIDDEN    12 __thread_list_lock
   150: 00000000004030f0     8 OBJECT  GLOBAL HIDDEN    11 __stdout_used
   151: 0000000000402f58     8 OBJECT  GLOBAL DEFAULT    8 stdout
   152: 0000000000400284   428 FUNC    GLOBAL HIDDEN     2 __init_libc
   153: 0000000000401000    48 FUNC    GLOBAL HIDDEN     2 __syscall_ret
   154: 0000000000400b74    92 FUNC    GLOBAL HIDDEN     2 __stdout_write
   155: 00000000004011ec    12 FUNC    GLOBAL HIDDEN     2 __ofl_unlock
   156: 0000000000403740     0 NOTYPE  GLOBAL DEFAULT   12 _bss_end__
   157: 0000000000400a88    68 FUNC    GLOBAL HIDDEN     2 __unlockfile
   158: 00000000004036c8     8 OBJECT  GLOBAL HIDDEN    12 __hwcap
   159: 00000000004010e8   228 FUNC    GLOBAL HIDDEN     2 __stdio_write
   160: 0000000000400bd0    72 FUNC    GLOBAL HIDDEN     2 __towrite
   161: 0000000000400dd0   396 FUNC    GLOBAL DEFAULT    2 memcpy
   162: 00000000004011cc    32 FUNC    GLOBAL HIDDEN     2 __ofl_lock
   163: 0000000000403100     0 OBJECT  GLOBAL HIDDEN    11 __TMC_END__
   164: 00000000004012f8    88 FUNC    GLOBAL HIDDEN     2 __unlock
   165: 0000000000400528   176 FUNC    GLOBAL DEFAULT    2 puts
   166: 0000000000400acc   120 FUNC    GLOBAL PROTECTED    2 __overflow
   167: 00000000004036d0   104 OBJECT  GLOBAL HIDDEN    12 __libc
   168: 0000000000403100     0 NOTYPE  GLOBAL DEFAULT   12 __bss_start__
   169: 0000000000403000     0 OBJECT  GLOBAL HIDDEN    11 __dso_handle
   170: 0000000000400fdc     0 FUNC    GLOBAL HIDDEN     2 __set_thread_area
   171: 00000000004036a8     8 OBJECT  WEAK   HIDDEN    12 __stdin_used
   172: 0000000000400764   176 FUNC    GLOBAL HIDDEN     2 __copy_tls
   173: 0000000000400b6c     8 FUNC    GLOBAL HIDDEN     2 __stdio_seek
   174: 0000000000403520     8 OBJECT  WEAK   DEFAULT   12 _environ
   175: 00000000004009d8   176 FUNC    GLOBAL HIDDEN     2 __lockfile
   176: 0000000000401350    12 FUNC    WEAK   HIDDEN     2 ___errno_location
   177: 0000000000400fe8    20 FUNC    WEAK   DEFAULT    2 lseek
   178: 0000000000403520     8 OBJECT  GLOBAL DEFAULT   12 __environ
   179: 00000000004009c0    24 FUNC    GLOBAL DEFAULT    2 _Exit
   180: 0000000000400c18     4 FUNC    GLOBAL HIDDEN     2 __towrite_needs_stdio_exit
   181: 0000000000400814   428 FUNC    WEAK   HIDDEN     2 __init_tls
   182: 0000000000400120     0 FUNC    GLOBAL DEFAULT    1 _init
   183: 00000000004004ec     4 FUNC    WEAK   HIDDEN     2 __funcs_on_exit
   184: 0000000000403520     8 OBJECT  WEAK   DEFAULT   12 environ
   185: 0000000000402f60     8 OBJECT  GLOBAL HIDDEN     8 __stdio_ofl_lockptr
   186: 0000000000400c1c    64 FUNC    WEAK   DEFAULT    2 fputs_unlocked
   187: 0000000000403740     0 NOTYPE  GLOBAL DEFAULT   12 __bss_end__
   188: 0000000000403520     8 OBJECT  WEAK   DEFAULT   12 ___environ
   189: 0000000000403108     8 OBJECT  GLOBAL DEFAULT   12 __progname
   190: 0000000000400178     0 FUNC    GLOBAL DEFAULT    2 _start
   191: 0000000000400194    40 FUNC    GLOBAL DEFAULT    2 _start_c
   192: 0000000000403008   232 OBJECT  GLOBAL HIDDEN    11 __stdout_FILE
   193: 0000000000403108     8 OBJECT  WEAK   DEFAULT   12 program_invocation_short_name
   194: 0000000000400430    60 FUNC    WEAK   HIDDEN     2 __libc_start_init
   195: 00000000004006e4   128 FUNC    GLOBAL HIDDEN     2 __init_tp
   196: 0000000000400280     4 FUNC    WEAK   HIDDEN     2 __init_ssp
   197: 0000000000400c5c   236 FUNC    GLOBAL HIDDEN     2 __fwritex
   198: 0000000000403100     0 NOTYPE  GLOBAL DEFAULT   12 __bss_start
   199: 00000000004005e0   260 FUNC    GLOBAL DEFAULT    2 memset
   200: 0000000000400150    40 FUNC    GLOBAL DEFAULT    2 main
   201: 00000000004010a0    72 FUNC    GLOBAL HIDDEN     2 __stdio_exit
   202: 00000000004011f8   256 FUNC    GLOBAL HIDDEN     2 __lock
   203: 0000000000403740     0 NOTYPE  GLOBAL DEFAULT   12 __end__
   204: 0000000000400b44     4 FUNC    WEAK   HIDDEN     2 __aio_close
   205: 0000000000400fe8    20 FUNC    GLOBAL HIDDEN     2 __lseek
   206: 000000000040135c     0 FUNC    GLOBAL DEFAULT    3 _fini
   207: 00000000004004f0    56 FUNC    WEAK   HIDDEN     2 __libc_exit_fini
   208: 0000000000400d48   136 FUNC    WEAK   DEFAULT    2 fwrite_unlocked
   209: 0000000000400d48   136 FUNC    GLOBAL DEFAULT    2 fwrite
   210: 0000000000403100     0 NOTYPE  GLOBAL DEFAULT   11 _edata
   211: 0000000000403740     0 NOTYPE  GLOBAL DEFAULT   12 _end
   212: 0000000000400b48    36 FUNC    GLOBAL HIDDEN     2 __stdio_close
   213: 0000000000401350    12 FUNC    GLOBAL DEFAULT    2 __errno_location
   214: 0000000000400130    28 FUNC    GLOBAL DEFAULT    2 exit
   215: 00000000004036a8     8 OBJECT  WEAK   HIDDEN    12 __stderr_used
   216: 00000000004010a0    72 FUNC    WEAK   HIDDEN     2 __stdio_exit_needed
   217: 00000000004004a4    72 FUNC    GLOBAL DEFAULT    2 __libc_start_main
   218: 0000000000400f60   124 FUNC    GLOBAL DEFAULT    2 strlen
   219: 0000000000400fe8    20 FUNC    WEAK   DEFAULT    2 lseek64
   220: 0000000000403110     8 OBJECT  WEAK   DEFAULT   12 program_invocation_name
   221: 00000000004030fc     4 OBJECT  GLOBAL HIDDEN    11 __default_stacksize
   222: 0000000000400c1c    64 FUNC    GLOBAL DEFAULT    2 fputs
   223: 00000000004030f8     4 OBJECT  GLOBAL HIDDEN    11 __default_guardsize
   224: 00000000004036c0     8 OBJECT  GLOBAL HIDDEN    12 __sysinfo
   225: 0000000000403110     8 OBJECT  GLOBAL DEFAULT   12 __progname_full

No version information found in this file.
```
# linux 2.4.18の`sys_execve()`を調査

```c
int sys_execve(struct pt_regs regs): arch/i386/kernel/process.c#L77
    int do_execve(char * filename, char ** argv, char ** envp, struct pt_regs * regs): fs/exec.c#L856
        struct file *open_exec(const char *name): #L339
        int prepare_binprm(struct linux_binprm *bprm): #L609
            int kernel_read(struct file *file, unsigned long offset, char * addr, unsigned long count): #L377
        int copy_strings_kernel(int argc,char ** argv, struct linux_binprm *bprm): #L243
        int copy_strings(int argc,char ** argv, struct linux_binprm *bprm): #L183
        int search_binary_handler(struct linux_binprm *bprm,struct pt_regs *regs): #L762
            static int load_elf_binary(struct linux_binprm * bprm, struct pt_regs * regs): fs/binfmt_elf.c#L427
```

## 主要構造体: `include/linux/binfmts.h`

###  `struct linux_binprm`: バイナリをロードする際に使用される引数を保持する構造体

```c
struct linux_binprm {
    char buf[BINPRM_BUF_SIZE];                                  /* 先頭128バイト読み込み用 */
    struct page *page[MAX_ARG_PAGES];                           /* 引数保存 */
    unsigned long p;                                            /* 現在のメモリトップを指す */
    int sh_bang;                                                /* 先頭行がsh_bangか */
    struct file * file;                                         /* バイナリを表すファイル構造体 */
    int e_uid, e_gid;                                           /* カレントプロセスのeuid, egid、set-uid/gid=>ファイルのuid, gid */
    kernel_cap_t cap_inheritable, cap_permitted, cap_effective; /* capability */
    int argc, envc;                                             /* argc: 引数の個数, envc: 環境変数の個数 */
    char * filename;                                            /* バイナリ名 */
    unsigned long loader, exec;                                 /* exec: page内のargvが始まるアドレス */
};
```

### `struct linux_binfmt`: バイナリフォーマットのロードに使用される関数を定義する構造体

```c
struct linux_binfmt {
    struct linux_binfmt * next;
    struct module *module;
    int (*load_binary)(struct linux_binprm *, struct  pt_regs * regs);
    int (*load_shlib)(struct file *);                                           // load_shlibはobsoluteなsys_uselibにしか
    int (*core_dump)(long signr, struct pt_regs * regs, struct file * file);    // 使用されていないので無視して可
    unsigned long min_coredump;    /* minimal dump size */
};
```

## struct linux_binfmtの実例

### elf実行ファイル: binfmt_elf.c

```c
struct linux_binfmt elf_format = {
    NULL, THIS_MODULE, load_elf_binary, load_elf_library, elf_core_dump, ELF_EXEC_PAGESIZE
};
```

### シェバングのあるテキスト実行ファイル: binfmt_script.c

```c
struct linux_binfmt script_format = {
    NULL, THIS_MODULE, load_script, NULL, NULL, 0
};
```

## linux 2.4の主要な構造体


### `struct task_struct`

```c
struct task_struct {
/*
 * 以下のオフセットは別の場所でハードコーディングされているので変更する場合は注意すること
 */
    volatile long state;        // -1 unrunnable, 0 runnable, >0 stopped
    unsigned long flags;        // プロセスごとのフラグ、以下で定義されている
    int sigpending;
    mm_segment_t addr_limit;    // スレッドのアドレス空間: mm_segment_t = unsigned long
                                //   0-0xBFFFFFFF ユーザスレッド用
                                //   0-0xFFFFFFFF カーネルスレッド用
    struct exec_domain *exec_domain;  // include/linux/personality.h
    volatile long need_resched;
    unsigned long ptrace;

    int lock_depth;             // Lock depth

/*
 * 32ビットプラットフォームでは、ここからオフセット32が始まります。
 * schedule() の goodness() ループに必要なすべてのフィールドを
 * 1つのキャッシュラインに収めています。
 */

    long counter;
    long nice;
    unsigned long policy;
    struct mm_struct *mm;       // mmap
    int processor;

/*
 * cpus_runnableは、プロセスがどのCPUでも実行されていない場合は ~0 となる。
 * CPU上で動作している場合は (1 << cpu)となる。このマスクを更新する場合は
 * ランキューロックが必要である。
 * プロセスがCPU上で実行されているか判断するために、このマスクが
 * cpus_allowedとANDされる。
 */
    unsigned long cpus_runnable, cpus_allowed;

/*
 * （'next'ポインタのみがキャッシュラインに収まるが、それで良い。（
 */
    struct list_head run_list;
    unsigned long sleep_time;

    struct task_struct *next_task, *prev_task;
    struct mm_struct *active_mm;
    struct list_head local_pages;
    unsigned int allocation_order, nr_local_pages;

/* Taskの状態 */
    struct linux_binfmt *binfmt;
    int exit_code, exit_signal;
    int pdeath_signal;            // 親が死んだ時に送信されるシグナル

/* ??? */
    unsigned long personality;
    int did_exec:1;
    pid_t pid;
    pid_t pgrp;
    pid_t tty_old_pgrp;
    pid_t session;
    pid_t tgid;
    int leader;                   // セッショングループリーダ用のブール値
/*
 * それぞれ、(オリジナル)親プロセス、親プロセス、末子プロセス、弟プロセス、
 * 兄プロセスへのポインタ。 (p->father は p->p_pptr->pid で置き換えることが
 * できる)
 */
    struct task_struct *p_opptr, *p_pptr, *p_cptr, *p_ysptr, *p_osptr;
    struct list_head thread_group;

/* PIDハッシュテーブルリンケージ */
    struct task_struct *pidhash_next;
    struct task_struct **pidhash_pprev;

    wait_queue_head_t wait_chldexit;    // wait4()用
    struct completion *vfork_done;      // vfork()用
    unsigned long rt_priority;
    unsigned long it_real_value, it_prof_value, it_virt_value;
    unsigned long it_real_incr, it_prof_incr, it_virt_incr;
    struct timer_list real_timer;
    struct tms times;
    unsigned long start_time;
    long per_cpu_utime[NR_CPUS], per_cpu_stime[NR_CPUS];
/* mmフォールトとスワップ情報：mm固有またはスレッド固有である */
    unsigned long min_flt, maj_flt, nswap, cmin_flt, cmaj_flt, cnswap;
    int swappable:1;
/* プロセスクレデンシャル */
    uid_t uid,euid,suid,fsuid;
    gid_t gid,egid,sgid,fsgid;
    int ngroups;
    gid_t    groups[NGROUPS];
    kernel_cap_t   cap_effective, cap_inheritable, cap_permitted;
    int keep_capabilities:1;
    struct user_struct *user;
/* 制約 */
    struct rlimit rlim[RLIM_NLIMITS];
    unsigned short used_math;
    char comm[16];
/* ファイルシステム情報 */
    int link_count, total_link_count;
    struct tty_struct *tty;           // ttyがない場合はNULL
    unsigned int locks;               // 獲得済みのファイルロックの数
/* ipc用のデータ */
    struct sem_undo *semundo;
    struct sem_queue *semsleeping;
/* このタスクのCPU固有の状態 */
    struct thread_struct thread;
/* ファイルシステム情報 */
    struct fs_struct *fs;
/* オープン済みのファイル情報 */
    struct files_struct *files;
/* シグナルハンドラ */
    spinlock_t sigmask_lock;        // シグナルとblockedを保護
    struct signal_struct *sig;

    sigset_t blocked;
    struct sigpending pending;

    unsigned long sas_ss_sp;
    size_t sas_ss_size;
    int (*notifier)(void *priv);
    void *notifier_data;
    sigset_t *notifier_mask;

/* スレッドグループの追跡用 */
    u32 parent_exec_id;
    u32 self_exec_id;
/* (de-)allocationの保護: mm, files, fs, tty */
    spinlock_t alloc_lock;

/* ジャーナリングファイルシステム情報 */
    void *journal_info;
};
```

### `struct mm_struct`

```C
struct mm_struct {
    struct vm_area_struct * mmap;         // VMAリスト
    rb_root_t mm_rb;
    struct vm_area_struct * mmap_cache;   // find_vamの最新結果
    pgd_t * pgd;
    atomic_t mm_users;            // このユーザ空間を持つユーザ数
    atomic_t mm_count;            // この"struct mm_struct"の参照数 (users count as 1)
    int map_count;                // VMAの数
    struct rw_semaphore mmap_sem;
    spinlock_t page_table_lock;   // task page tablesとmm->rssを保護

    struct list_head mmlist;      // アクティブなすべてのmmのリスト
                                  // init_mm.mmlist からグローバルに繋げられ、
                                  // mmlist_lockで保護される

    unsigned long start_code, end_code, start_data, end_data;
    unsigned long start_brk, brk, start_stack;
    unsigned long arg_start, arg_end, env_start, env_end;
    unsigned long rss, total_vm, locked_vm;
    unsigned long def_flags;
    unsigned long cpu_vm_mask;
    unsigned long swap_address;

    unsigned dumpable:1;

    /* アーキテクチャ固有のMMコンテキスト */
    mm_context_t context;
};
```

### `struct files_struct`

```c
struct files_struct {
    atomic_t count;
    rwlock_t file_lock;   // 以下のすべてのメンバを保護。tsk->alloc_lock内でネスト
    int max_fds;
    int max_fdset;
    int next_fd;
    struct file ** fd;    // 現在のfd配列
    fd_set *close_on_exec;
    fd_set *open_fds;
    fd_set close_on_exec_init;
    fd_set open_fds_init;
    struct file * fd_array[NR_OPEN_DEFAULT];
};
```

### `struct user_struct`

```c
struct user_struct {
    atomic_t __count;   /* reference count */
    atomic_t processes; /* How many processes does this user have? */
    atomic_t files;     /* How many open files does this user have? */

    /* Hash table maintenance information */
    struct user_struct *next, **pprev;
    uid_t uid;
};
```

### `struct file`

- `include/linux/fs.h`

```c
struct file {
    struct list_head        f_list;
    struct dentry           *f_dentry;
    struct vfsmount         *f_vfsmnt;
    struct file_operations  *f_op;
    atomic_t                f_count;
    unsigned int            f_flags;
    mode_t                  f_mode;
    loff_t                  f_pos;
    unsigned long           f_reada, f_ramax, f_raend, f_ralen, f_rawin;
    struct fown_struct      f_owner;
    unsigned int            f_uid, f_gid;
    int                     f_error;

    unsigned long           f_version;

    /* needed for tty driver, and maybe others */
    void                    *private_data;

    /* preallocated helper kiobuf to speedup O_DIRECT */
    struct kiobuf           *f_iobuf;
    long                    f_iobuf_lock;
};
```

### `struct file_operations`

```c
struct file_operations {
    struct module *owner;
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char *, size_t, loff_t *);
    int (*readdir) (struct file *, void *, filldir_t);
    unsigned int (*poll) (struct file *, struct poll_table_struct *);
    int (*ioctl) (struct inode *, struct file *, unsigned int, unsigned long);
    int (*mmap) (struct file *, struct vm_area_struct *);
    int (*open) (struct inode *, struct file *);
    int (*flush) (struct file *);
    int (*release) (struct inode *, struct file *);
    int (*fsync) (struct file *, struct dentry *, int datasync);
    int (*fasync) (int, struct file *, int);
    int (*lock) (struct file *, int, struct file_lock *);
    ssize_t (*readv) (struct file *, const struct iovec *, unsigned long, loff_t *);
    ssize_t (*writev) (struct file *, const struct iovec *, unsigned long, loff_t *);
    ssize_t (*sendpage) (struct file *, struct page *, int, size_t, loff_t *, int);
    unsigned long (*get_unmapped_area)(struct file *, unsigned long, unsigned long, unsigned long, unsigned long);
};
```

### `struct vm_area_struct`

- `include/linux/mm.h`

```c
struct vm_area_struct {
    struct mm_struct    *vm_mm;     // 所属するアドレス空間
    unsigned long       vm_start;   // vm_mm内のこの開始アドレス
    unsigned long       vm_end;     // vm_mm内のこの終了アドレスの次のバイト

    /* タスクごとのVMエリアのリンクリスト。アドレス順にソートされている */
    struct vm_area_struct *vm_next;

    pgprot_t            vm_page_prot;   // このVMAのアクセス権限
    unsigned            long vm_flags;  // フラグ

    rb_node_t           vm_rb;          // 赤黒木ノード

    /*
     * アドレス空間とバッキングストアを持つVMエリアについては、
     * address_space->i_mmap{,shared} リストのいずれか、
     * shmエリアについてはアタッシュリスト、それ以外は未使用。
     */
    struct vm_area_struct *vm_next_share;
    struct vm_area_struct **vm_pprev_share;

    /* この構造体を処理する関数ポインタ */
    struct vm_operations_struct     *vm_ops;

    /* バッキングストアに関する情報 */
    unsigned long   vm_pgoff;   // PAGE_SIZE（PAGE_CACHE_SIZEではない）
                                // 単位の(vm_file内の）オフセット
    struct file     *vm_file;   // このVMがマッピングしているファイル（NULL可）
    unsigned long   vm_raend;   // XXX: full readahead infoを置く
    void            *vm_private_data;   // 旧(shared mem) のvm_pte
};
```

### `struct page`

```c
typedef struct page {
    struct list_head list;          /* ->mappingは複数のページリストを持つ */
    struct address_space *mapping;  /* 所属するinode */
    unsigned long index;            /* mapping内のオフセット */
    struct page *next_hash;         /* pagecacheハッシュ表でハッシュバケットを共有する次のページ */
    atomic_t count;                 /* 利用数 */
    unsigned long flags;            /* atomicなフラグ。非同期に更新される可能性のあるもの */
    struct list_head lru;           /* ページアウトリスト。たとえば、active_list。
                                         pagemap_lru_lockで保護される */
    wait_queue_head_t wait;         /* ページはロックされているか？ 先着順 */
    struct page **pprev_hash;       /* *next_hashの補足 */
    struct buffer_head * buffers;   /* bufferはdiskブロックにマップする */
    void *virtual;                  /* カーネル仮想アドレス（kmappedされていない(highmem)場合はNULL */
    struct zone_struct *zone;       /* 所属するメモリゾーン */
} mem_map_t;
```

## ELF_PAGESTART, ELF_PAGEOFFSET, ELF_PAGEALIGN

```
addr    start offs  align down  up
   0 => 0000, 0000, 0000, 0000, 0000
   1 => 0000, 0001, 4096, 0000, 4096
4095 => 0000, 4095, 4096, 0000, 4096
4096 => 4096, 0000, 4096, 4096, 4096
4097 => 4096, 0001, 8192, 4096, 8192
8191 => 4096, 4095, 8192, 4096, 8192
8192 => 8192, 0000, 8192, 8192, 8192
```

## do_execve()処理後のbprm.pageの構成

```
              filename文字列
bprm.exec -> ----------------
              argv[n]文字列
               ...
              argv[1]文字列
              argv[0]文字列
              envp[n]文字列
               ...
              envp[1]文字列
              envp[0]文字列
bprm.p    -> ----------------
```

# share libraryを使った実行ファイル

## プログラムヘッダ

```
Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x0000000000000188 0x0000000000000188  R      0x8
  INTERP         0x00000000000001c8 0x00000000000001c8 0x00000000000001c8
                 0x000000000000001a 0x000000000000001a  R      0x1
      [Requesting program interpreter: /lib/ld-musl-aarch64.so.1]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000724 0x0000000000000724  R E    0x10000
  LOAD           0x0000000000000db8 0x0000000000010db8 0x0000000000010db8
                 0x0000000000000250 0x0000000000000258  RW     0x10000
  DYNAMIC        0x0000000000000dc8 0x0000000000010dc8 0x0000000000010dc8
                 0x00000000000001d0 0x00000000000001d0  RW     0x8
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000000db8 0x0000000000010db8 0x0000000000010db8
                 0x0000000000000248 0x0000000000000248  R      0x1

```

## execve()のauxについて

- AT_PAGESZ以外は絶対必要というわけでもなさそう

```c
/? musl/src/env/__libc_start_main.c */

#define AUX_CNT 38

void __init_libc(char **envp, char *pn)
{
    size_t i, *auxv, aux[AUX_CNT] = { 0 };
    __environ = envp;
    for (i=0; envp[i]; i++);
    libc.auxv = auxv = (void *)(envp+i+1);
    for (i=0; auxv[i]; i+=2) if (auxv[i]<AUX_CNT) aux[auxv[i]] = auxv[i+1];
    __hwcap = aux[AT_HWCAP];
    if (aux[AT_SYSINFO]) __sysinfo = aux[AT_SYSINFO];
    libc.page_size = aux[AT_PAGESZ];

    if (!pn) pn = (void*)aux[AT_EXECFN];
    if (!pn) pn = "";
    __progname = __progname_full = pn;
    for (i=0; pn[i]; i++) if (pn[i]=='/') __progname = pn+i+1;

    __init_tls(aux);
    __init_ssp((void *)aux[AT_RANDOM]);

    if (aux[AT_UID]==aux[AT_EUID] && aux[AT_GID]==aux[AT_EGID]
        && !aux[AT_SECURE]) return;

/* musl/src/internal/vdso.c */
void *__vdsosym(const char *vername, const char *name)
{
        size_t i;
        for (i=0; libc.auxv[i] != AT_SYSINFO_EHDR; i+=2)
                if (!libc.auxv[i]) return 0;
        if (!libc.auxv[i+1]) return 0;
```

# dynamic linkした実行ファイルのreadelf

```
$ aarch64-linux-gnu-gcc -specs /usr/local/musl/lib/musl-gcc.specs -Wall -Wstrict-prototypes -Wno-trigraphs -O2 -fpic -fpie -fplt -fomit-frame-pointer -fno-strict-aliasing -fno-common hello.c
$ aarch64-linux-gnu-readelf -W -a a.out
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x55c
  Start of program headers:          64 (bytes into file)
  Start of section headers:          6576 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         7
  Size of section headers:           64 (bytes)
  Number of section headers:         24
  Section header string table index: 23

Section Headers:
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            0000000000000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        00000000000001c8 0001c8 00001a 00   A  0   0  1
  [ 2] .hash             HASH            00000000000001e8 0001e8 00003c 04   A  4   0  8
  [ 3] .gnu.hash         GNU_HASH        0000000000000228 000228 000028 00   A  4   0  8
  [ 4] .dynsym           DYNSYM          0000000000000250 000250 0000f0 18   A  5   3  8
  [ 5] .dynstr           STRTAB          0000000000000340 000340 000071 00   A  0   0  1
  [ 6] .rela.dyn         RELA            00000000000003b8 0003b8 0000d8 18   A  4   0  8
  [ 7] .rela.plt         RELA            0000000000000490 000490 000048 18  AI  4  17  8
  [ 8] .init             PROGBITS        00000000000004d8 0004d8 000010 00  AX  0   0  4
  [ 9] .plt              PROGBITS        00000000000004f0 0004f0 000050 10  AX  0   0 16
  [10] .text             PROGBITS        0000000000000540 000540 00011c 00  AX  0   0  8
  [11] .fini             PROGBITS        000000000000065c 00065c 000010 00  AX  0   0  4
  [12] .rodata           PROGBITS        0000000000000670 000670 00000d 01 AMS  0   0  8
  [13] .eh_frame         PROGBITS        0000000000000680 000680 000098 00   A  0   0  8
  [14] .init_array       INIT_ARRAY      0000000000010db8 000db8 000008 08  WA  0   0  8
  [15] .fini_array       FINI_ARRAY      0000000000010dc0 000dc0 000008 08  WA  0   0  8
  [16] .dynamic          DYNAMIC         0000000000010dc8 000dc8 0001d0 10  WA  5   0  8
  [17] .got              PROGBITS        0000000000010f98 000f98 000068 08  WA  0   0  8
  [18] .data             PROGBITS        0000000000011000 001000 000008 00  WA  0   0  8
  [19] .bss              NOBITS          0000000000011008 001008 000008 00  WA  0   0  1
  [20] .comment          PROGBITS        0000000000000000 001008 00002a 01  MS  0   0  1
  [21] .symtab           SYMTAB          0000000000000000 001038 0006f0 18     22  55  8
  [22] .strtab           STRTAB          0000000000000000 001728 0001d5 00      0   0  1
  [23] .shstrtab         STRTAB          0000000000000000 0018fd 0000af 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  p (processor specific)

There are no section groups in this file.

Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  PHDR           0x000040 0x0000000000000040 0x0000000000000040 0x000188 0x000188 R   0x8
  INTERP         0x0001c8 0x00000000000001c8 0x00000000000001c8 0x00001a 0x00001a R   0x1
      [Requesting program interpreter: /lib/ld-musl-aarch64.so.1]
  LOAD           0x000000 0x0000000000000000 0x0000000000000000 0x000718 0x000718 R E 0x10000
  LOAD           0x000db8 0x0000000000010db8 0x0000000000010db8 0x000250 0x000258 RW  0x10000
  DYNAMIC        0x000dc8 0x0000000000010dc8 0x0000000000010dc8 0x0001d0 0x0001d0 RW  0x8
  GNU_STACK      0x000000 0x0000000000000000 0x0000000000000000 0x000000 0x000000 RW  0x10
  GNU_RELRO      0x000db8 0x0000000000010db8 0x0000000000010db8 0x000248 0x000248 R   0x1

 Section to Segment mapping:
  Segment Sections...
   00
   01     .interp
   02     .interp .hash .gnu.hash .dynsym .dynstr .rela.dyn .rela.plt .init .plt .text .fini .rodata .eh_frame
   03     .init_array .fini_array .dynamic .got .data .bss
   04     .dynamic
   05
   06     .init_array .fini_array .dynamic .got

Dynamic section at offset 0xdc8 contains 25 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libc.so]
 0x000000000000000c (INIT)               0x4d8
 0x000000000000000d (FINI)               0x65c
 0x0000000000000019 (INIT_ARRAY)         0x10db8
 0x000000000000001b (INIT_ARRAYSZ)       8 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x10dc0
 0x000000000000001c (FINI_ARRAYSZ)       8 (bytes)
 0x0000000000000004 (HASH)               0x1e8
 0x000000006ffffef5 (GNU_HASH)           0x228
 0x0000000000000005 (STRTAB)             0x340
 0x0000000000000006 (SYMTAB)             0x250
 0x000000000000000a (STRSZ)              113 (bytes)
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000015 (DEBUG)              0x0
 0x0000000000000003 (PLTGOT)             0x10f98
 0x0000000000000002 (PLTRELSZ)           72 (bytes)
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000017 (JMPREL)             0x490
 0x0000000000000007 (RELA)               0x3b8
 0x0000000000000008 (RELASZ)             216 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000000000001e (FLAGS)              BIND_NOW
 0x000000006ffffffb (FLAGS_1)            Flags: NOW PIE
 0x000000006ffffff9 (RELACOUNT)          6
 0x0000000000000000 (NULL)               0x0

Relocation section '.rela.dyn' at offset 0x3b8 contains 9 entries:
    Offset             Info             Type               Symbol's Value  Symbol's Name + Addend
0000000000010db8  0000000000000403 R_AARCH64_RELATIVE                        658
0000000000010dc0  0000000000000403 R_AARCH64_RELATIVE                        610
0000000000010fd8  0000000000000403 R_AARCH64_RELATIVE                        4d8
0000000000010ff0  0000000000000403 R_AARCH64_RELATIVE                        540
0000000000010ff8  0000000000000403 R_AARCH64_RELATIVE                        65c
0000000000011000  0000000000000403 R_AARCH64_RELATIVE                        11000
0000000000010fd0  0000000400000401 R_AARCH64_GLOB_DAT     0000000000000000 __cxa_finalize + 0
0000000000010fe0  0000000500000401 R_AARCH64_GLOB_DAT     0000000000000000 _ITM_registerTMCloneTable + 0
0000000000010fe8  0000000600000401 R_AARCH64_GLOB_DAT     0000000000000000 _ITM_deregisterTMCloneTable + 0

Relocation section '.rela.plt' at offset 0x490 contains 3 entries:
    Offset             Info             Type               Symbol's Value  Symbol's Name + Addend
0000000000010fb0  0000000300000402 R_AARCH64_JUMP_SLOT    0000000000000000 puts + 0
0000000000010fb8  0000000400000402 R_AARCH64_JUMP_SLOT    0000000000000000 __cxa_finalize + 0
0000000000010fc0  0000000700000402 R_AARCH64_JUMP_SLOT    0000000000000000 __libc_start_main + 0

The decoding of unwind sections for machine type AArch64 is not currently supported.

Symbol table '.dynsym' contains 10 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 00000000000004d8     0 SECTION LOCAL  DEFAULT    8
     2: 0000000000011000     0 SECTION LOCAL  DEFAULT   18
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts
     4: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND __cxa_finalize
     5: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
     6: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTable
     7: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main
     8: 00000000000004d8     4 FUNC    GLOBAL DEFAULT    8 _init
     9: 000000000000065c     4 FUNC    GLOBAL DEFAULT   11 _fini

Symbol table '.symtab' contains 74 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 00000000000001c8     0 SECTION LOCAL  DEFAULT    1
     2: 00000000000001e8     0 SECTION LOCAL  DEFAULT    2
     3: 0000000000000228     0 SECTION LOCAL  DEFAULT    3
     4: 0000000000000250     0 SECTION LOCAL  DEFAULT    4
     5: 0000000000000340     0 SECTION LOCAL  DEFAULT    5
     6: 00000000000003b8     0 SECTION LOCAL  DEFAULT    6
     7: 0000000000000490     0 SECTION LOCAL  DEFAULT    7
     8: 00000000000004d8     0 SECTION LOCAL  DEFAULT    8
     9: 00000000000004f0     0 SECTION LOCAL  DEFAULT    9
    10: 0000000000000540     0 SECTION LOCAL  DEFAULT   10
    11: 000000000000065c     0 SECTION LOCAL  DEFAULT   11
    12: 0000000000000670     0 SECTION LOCAL  DEFAULT   12
    13: 0000000000000680     0 SECTION LOCAL  DEFAULT   13
    14: 0000000000010db8     0 SECTION LOCAL  DEFAULT   14
    15: 0000000000010dc0     0 SECTION LOCAL  DEFAULT   15
    16: 0000000000010dc8     0 SECTION LOCAL  DEFAULT   16
    17: 0000000000010f98     0 SECTION LOCAL  DEFAULT   17
    18: 0000000000011000     0 SECTION LOCAL  DEFAULT   18
    19: 0000000000011008     0 SECTION LOCAL  DEFAULT   19
    20: 0000000000000000     0 SECTION LOCAL  DEFAULT   20
    21: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS Scrt1.c
    22: 000000000000055c     0 NOTYPE  LOCAL  DEFAULT   10 $x
    23: 0000000000000578     0 NOTYPE  LOCAL  DEFAULT   10 $x
    24: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /usr/local/musl/lib/crti.o
    25: 00000000000004d8     0 NOTYPE  LOCAL  DEFAULT    8 $x
    26: 000000000000065c     0 NOTYPE  LOCAL  DEFAULT   11 $x
    27: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS /usr/local/musl/lib/crtn.o
    28: 00000000000004e0     0 NOTYPE  LOCAL  DEFAULT    8 $x
    29: 0000000000000664     0 NOTYPE  LOCAL  DEFAULT   11 $x
    30: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS hello.c
    31: 0000000000000670     0 NOTYPE  LOCAL  DEFAULT   12 $d
    32: 0000000000000540     0 NOTYPE  LOCAL  DEFAULT   10 $x
    33: 00000000000006f8     0 NOTYPE  LOCAL  DEFAULT   13 $d
    34: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    35: 00000000000005a0     0 NOTYPE  LOCAL  DEFAULT   10 $x
    36: 00000000000005a0     0 FUNC    LOCAL  DEFAULT   10 deregister_tm_clones
    37: 00000000000005d0     0 FUNC    LOCAL  DEFAULT   10 register_tm_clones
    38: 0000000000011000     0 NOTYPE  LOCAL  DEFAULT   18 $d
    39: 0000000000000610     0 FUNC    LOCAL  DEFAULT   10 __do_global_dtors_aux
    40: 0000000000011008     1 OBJECT  LOCAL  DEFAULT   19 completed.9126
    41: 0000000000010dc0     0 NOTYPE  LOCAL  DEFAULT   15 $d
    42: 0000000000010dc0     0 OBJECT  LOCAL  DEFAULT   15 __do_global_dtors_aux_fini_array_entry
    43: 0000000000000658     0 FUNC    LOCAL  DEFAULT   10 frame_dummy
    44: 0000000000010db8     0 NOTYPE  LOCAL  DEFAULT   14 $d
    45: 0000000000010db8     0 OBJECT  LOCAL  DEFAULT   14 __frame_dummy_init_array_entry
    46: 0000000000000694     0 NOTYPE  LOCAL  DEFAULT   13 $d
    47: 0000000000011008     0 NOTYPE  LOCAL  DEFAULT   19 $d
    48: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    49: 0000000000000714     0 NOTYPE  LOCAL  DEFAULT   13 $d
    50: 0000000000000714     0 OBJECT  LOCAL  DEFAULT   13 __FRAME_END__
    51: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS
    52: 0000000000010dc8     0 OBJECT  LOCAL  DEFAULT  ABS _DYNAMIC
    53: 0000000000010fc8     0 OBJECT  LOCAL  DEFAULT  ABS _GLOBAL_OFFSET_TABLE_
    54: 00000000000004f0     0 NOTYPE  LOCAL  DEFAULT    9 $x
    55: 0000000000011010     0 NOTYPE  GLOBAL DEFAULT   19 _bss_end__
    56: 0000000000011008     0 OBJECT  GLOBAL HIDDEN    18 __TMC_END__
    57: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND puts
    58: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND __cxa_finalize
    59: 0000000000011008     0 NOTYPE  GLOBAL DEFAULT   19 __bss_start__
    60: 0000000000011000     0 OBJECT  GLOBAL HIDDEN    18 __dso_handle
    61: 00000000000004d8     4 FUNC    GLOBAL DEFAULT    8 _init
    62: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMCloneTable
    63: 0000000000011010     0 NOTYPE  GLOBAL DEFAULT   19 __bss_end__
    64: 000000000000055c     0 FUNC    GLOBAL DEFAULT   10 _start
    65: 0000000000000578    40 FUNC    GLOBAL DEFAULT   10 _start_c
    66: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterTMCloneTable
    67: 0000000000011008     0 NOTYPE  GLOBAL DEFAULT   19 __bss_start
    68: 0000000000000540    28 FUNC    GLOBAL DEFAULT   10 main
    69: 0000000000011010     0 NOTYPE  GLOBAL DEFAULT   19 __end__
    70: 000000000000065c     4 FUNC    GLOBAL DEFAULT   11 _fini
    71: 0000000000011008     0 NOTYPE  GLOBAL DEFAULT   18 _edata
    72: 0000000000011010     0 NOTYPE  GLOBAL DEFAULT   19 _end
    73: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main

Histogram for bucket list length (total of 3 buckets):
 Length  Number     % of total  Coverage
      0  0          (  0.0%)
      1  2          ( 66.7%)     28.6%
      2  0          (  0.0%)     28.6%
      3  0          (  0.0%)     28.6%
      4  0          (  0.0%)     28.6%
      5  1          ( 33.3%)    100.0%

Histogram for `.gnu.hash' bucket list length (total of 2 buckets):
 Length  Number     % of total  Coverage
      0  1          ( 50.0%)
      1  0          (  0.0%)      0.0%
      2  1          ( 50.0%)    100.0%

No version information found in this file.
```

# static linkのgettyでエラー発生

```
execve: proc[1] '/bin/init' start, uid=0, gid=0
execve: proc[6] '/bin/getty' start, uid=0, gid=0
syscall: pid=6, x0=0x0, x1=0x0, x2=0x0, x3=0x0 x4=0x0
Unexpected syscall #73              // sys_ppoll だが、これはttyname(0)がエラーで
                                    // 呼び出されたpause()からよびだされているので
                                    // 問題は別にあり
trapret () at kern/trapasm.S:88
88	    eret
(gdb)
warning: Breakpoint address adjusted from 0xeb03009ff9448cc3 to 0x3009ff9448cc3.
```

## argint(int, uint64_t *)の引数の型を間違っていた。

- `int fd`と定義して`argint(0, (uint64_t *)&fd)`と呼び出すように変更したが、これだとfdがマイナスの場合、うまく設定できないようだ。
- `uint64_t fd64`を定義して`argint(0, &fd64); fd = (int)fd64;`とする元の方法に戻した。
- とりあえず、これでgettyも動くようになったが、後でもう一度考える。

# dynamic linkしたアプリを動かす

```
# /bin/hello
execve: Unable to load interpreter
-dash: 2: /bin/hello: Operation not permitted

# /bin/hello
- elf_entry: ffffffffffffffff
execve: Unable to load interpreter
-dash: 1: /bin/hello: not found
# ls -li /bin/hello
140 -rwxr-xr-x 1 root root 8112 Dec  5  2021 /bin/hello
# ls -li /lib
144 -rwxr-xr-x 1 root root 828816 Dec  5  2021 ld-musl-aarch64.so.1
# /bin/hellos
hello static world!
```

## `load_elf_interpreter`が呼び出された際のgdb

- intrtpreterが読み込まれていない

```
Thread 1 hit Breakpoint 1, load_elf_interp (pgdir=0xffff000039b07000,
    ehdr=0xffff00003a7939c8, interp=0xffff0000000dcfc0 <icache+24>,
    interp_load_addr=0xffff00003a793980) at kern/exec.c:36
36	    uint64_t load_addr = 0;
(gdb) p *interp
$1 = {
  dev = 1,
  inum = 1,

(gdb) p *ehdr
$2 = {
  e_ident = "\001\000.", '\000' <repeats 12 times>,
  e_type = 0,
  e_machine = 0,
  e_version = 0,
  e_entry = 0,
  e_phoff = 0,
  e_shoff = 0,
  e_flags = 0,
  e_ehsize = 0,
  e_phentsize = 0,
  e_phnum = 0,
  e_shentsize = 0,
  e_shnum = 0,
  e_shstrndx = 0
}
```

## interp_pathの設定に間違いあり

```
# /bin/hello
- elf_entry: 61614
uvm_alloc: newsz 0x15555000 is too big
execve: bad
dash: 1: /bin/hello: Invalid argument

# /bin/hello
- uload: 0xaaaa000, off=0x0, len=0x72c, sz=0xaaaa72c
- uload: 0xaaabdb8, off=0xdb8, len=0x250, sz=0xaaac010
load_interp: ualloc: old=0x0, new=0x98c04
- uload: addr=0x0, off=0x0, len=0x98c04
load_interp: ualloc: old=0x98c04, new=0xad0f8
- uload: addr=0xa9b90, off=0x99b90, len=0x89a
- elf_entry: 0x61614
0xaaaeff0: argv[0]=/bin/hello
0xaaaefe0: envp[0]=PWD=/.
0xaaaefe0: auxv[28]=178974688
0xaaaefd8: auxv[27]=0
0xaaaefd0: auxv[26]=0
0xaaaefc8: auxv[25]=0
0xaaaefc0: auxv[24]=16
0xaaaefb8: auxv[23]=4096
0xaaaefb0: auxv[22]=6
0xaaaefa8: auxv[21]=100
0xaaaefa0: auxv[20]=17
0xaaaef98: auxv[19]=64
0xaaaef90: auxv[18]=3
0xaaaef88: auxv[17]=56
0xaaaef80: auxv[16]=4
0xaaaef78: auxv[15]=7
0xaaaef70: auxv[14]=5
0xaaaef68: auxv[13]=0
0xaaaef60: auxv[12]=7
0xaaaef58: auxv[11]=0
0xaaaef50: auxv[10]=8
0xaaaef48: auxv[9]=178955616
0xaaaef40: auxv[8]=9
0xaaaef38: auxv[7]=0
0xaaaef30: auxv[6]=11
0xaaaef28: auxv[5]=0
0xaaaef20: auxv[4]=12
0xaaaef18: auxv[3]=0
0xaaaef10: auxv[2]=13
0xaaaef08: auxv[1]=0
0xaaaef00: auxv[0]=14
0xaaaef00: estack[1]=0x0
0xaaaeef8: estack[0]=0xaaaefe0
0xaaaeef0: ustack[1]=0x0
0xaaaeee8: ustack[0]=0xaaaeff0
0xaaaeee0: argc=1
unexpected interrupt 0 at cpu 0: sp=aaaeee0
Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x0
 SPSR_EL1 0x60000000 FAR_EL1 0x434d40
	irq_error: irq of type 8 unimplemented.
```

## stackoverflowの質問と回答

**EC=0（一桁見間違えた）だったのでこれは無関係だが記録しておく**

- [AArch64 ESR Trapped WFI or WFE instruction execution](https://stackoverflow.com/questions/64502103/aarch64-esr-trapped-wfi-or-wfe-instruction-execution)
- [linuxのパッチ](https://patchwork.kernel.org/project/linux-arm-kernel/patch/20180807093326.5090-1-marc.zyngier@arm.com/)

```
$ aarch64-linux-gnu-readelf -l obj/user_dyn/bin/hello

Elf file type is DYN (Shared object file)
Entry point 0x560
There are 7 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x0000000000000188 0x0000000000000188  R      0x8
  INTERP         0x00000000000001c8 0x00000000000001c8 0x00000000000001c8
                 0x000000000000001a 0x000000000000001a  R      0x1
      [Requesting program interpreter: /lib/ld-musl-aarch64.so.1]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x000000000000072c 0x000000000000072c  R E    0x1000
  LOAD           0x0000000000000db8 0x0000000000001db8 0x0000000000001db8
                 0x0000000000000250 0x0000000000000258  RW     0x1000
```

```
$ aarch64-linux-gnu-readelf -l /usr/local/musl/lib/libc.so

Elf file type is DYN (Shared object file)
Entry point 0x61614
There are 7 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000098c04 0x0000000000098c04  R E    0x10000
  LOAD           0x0000000000099b90 0x00000000000a9b90 0x00000000000a9b90
                 0x000000000000089a 0x0000000000003568  RW     0x10000
```

```
00061610: bed5 fe17 1d00 80d2 1e00 80d2 e003 0091  ................
00061620: 4102 0090 2120 3891 1fec 7c92 0100 0014  A...! 8...|.....
00061630: e403 00aa ff03 08d1 0284 40f8 4204 0011  ..........@.B...
00061640: 427c 4093 0378 62f8 4204 0091 c3ff ffb5  B|@..xb.B.......
00061650: e503 0091 020c 028b 0000 80d2 bf78 20f8  .............x .
00061660: 0004 0091 1f80 00f1 a1ff ff54 4300 40f9  ...........TC.@.
00061670: 8304 00b5 e503 0491 bf78 23f8 6304 0091  .........x#.c...
00061680: 7f80 00f1 a1ff ff54 e003 01aa 0200 40f9  .......T......@.
```

```
(gdb) x/128xb 0
0x0:	0x7f	0x45	0x4c	0x46	0x02	0x01	0x01	0x00
0x8:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0x10:	0x03	0x00	0xb7	0x00	0x01	0x00	0x00	0x00
0x18:	0x14	0x16	0x06	0x00	0x00	0x00	0x00	0x00
0x20:	0x40	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0x28:	0xd0	0x9f	0x0c	0x00	0x00	0x00	0x00	0x00
0x30:	0x00	0x00	0x00	0x00	0x40	0x00	0x38	0x00
0x38:	0x07	0x00	0x40	0x00	0x17	0x00	0x16	0x00

(gdb) x/128xb 0x61610
0x61610:	0xbe	0xd5	0xfe	0x17	0x1d	0x00	0x80	0xd2
0x61618:	0x1e	0x00	0x80	0xd2	0xe0	0x03	0x00	0x91
0x61620:	0x41	0x02	0x00	0x90	0x21	0x20	0x38	0x91
0x61628:	0x1f	0xec	0x7c	0x92	0x01	0x00	0x00	0x14
0x61630:	0xe4	0x03	0x00	0xaa	0xff	0x03	0x08	0xd1
0x61638:	0x02	0x84	0x40	0xf8	0x42	0x04	0x00	0x11
0x61640:	0x42	0x7c	0x40	0x93	0x03	0x78	0x62	0xf8
0x61648:	0x42	0x04	0x00	0x91	0xc3	0xff	0xff	0xb5
0x61650:	0xe5	0x03	0x00	0x91	0x02	0x0c	0x02	0x8b
0x61658:	0x00	0x00	0x80	0xd2	0xbf	0x78	0x20	0xf8
0x61660:	0x00	0x04	0x00	0x91	0x1f	0x80	0x00	0xf1
0x61668:	0xa1	0xff	0xff	0x54	0x43	0x00	0x40	0xf9
0x61670:	0x83	0x04	0x00	0xb5	0xe5	0x03	0x04	0x91
0x61678:	0xbf	0x78	0x23	0xf8	0x63	0x04	0x00	0x91
0x61680:	0x7f	0x80	0x00	0xf1	0xa1	0xff	0xff	0x54
0x61688:	0xe0	0x03	0x01	0xaa	0x02	0x00	0x40	0xf9

(gdb) x/8i 0x61614
=> 0x61614:	mov	x29, #0x0                   	// #0
   0x61618:	mov	x30, #0x0                   	// #0
   0x6161c:	mov	x0, sp
   0x61620:	adrp	x1, 0xa9000
   0x61624:	add	x1, x1, #0xe08
   0x61628:	and	sp, x0, #0xfffffffffffffff0
   0x6162c:	b	0x61630
   0x61630:	mov	x4, x0

0000000000061614 <_dlstart>:
   61614:       d280001d        mov     x29, #0x0                       // #0
   61618:       d280001e        mov     x30, #0x0                       // #0
   6161c:       910003e0        mov     x0, sp
   61620:       90000241        adrp    x1, a9000 <__GNU_EH_FRAME_HDR+0x10858>
   61624:       91382021        add     x1, x1, #0xe08
   61628:       927cec1f        and     sp, x0, #0xfffffffffffffff0
   6162c:       14000001        b       61630 <_dlstart_c>

```

# 今日の知見

- linuxの`struct pt_regs`はxv6の`struct trapframe`に該当する


## syscallが実行されるまで

### 1. 割り込みが発生。割り込みベクタから割り込みハンドラを呼び出す: vectors.S

```arm
// in vectors.S
#define ventry .align 7; b alltraps     // alltraps()を呼び出す

.globl vectors
.align 11
vectors:                                // 割り込みベクタ定義
el1_sp0:
    ...
el0_aarch64:
    /* Lower EL using AArch64 */
    ventry          // Synchronous (8)  // ventry -> alltraps
    ventry          // IRQ (9)
```

### 2. 割り込みハンドラ: trapasm.S

```arm
.global alltraps
alltraps:                               // alltraps()定義
    stp     x29, x30, [sp, #-16]!       // struct trapframeを作成
    ...
    stp     x1, x2, [sp, #-16]!
    mrs     x4, tpidr_el0
    mrs     x5, esr_el1
    stp     x4, x5, [sp, #-16]!
    str     q0, [sp, #-16]!

    add     x0, sp, #0x0
    bl      trap                // trap(trap framse)を呼び出す
```

### 3. 割り込みの振り分け: trap.c

```c
void trap(struct trapframe *tf)
{
    int ec = resr() >> EC_SHIFT, iss = resr() & ISS_MASK;   // mrs $resr, esr_el1
    switch (ec) {
    case EC_UNKNOWN:
        interrupt(tf);
        break;
    case EC_SVC64:
        if (iss == 0) {
            tf->x0 = syscall(tf);       // syscall処理関数を呼び出す
        }
```

### 4. システムコール処理: syscall.c

```c
int syscall(struct trapframe *tf)
{
    struct proc *p = thisproc();
    p->tf = tf;
    uint64_t sysno = tf->x8;
    tf->x0 = syscalls[sysno]();         // システムコール処理関数を呼び出す
```

### 5. syscall引数の処理: syscall.c

```c
int argint(int n, uint64_t *ip)
{
    if (n > 7) {
        panic("argint: too many system call parameters\n");
    }

    struct proc *proc = thiscpu->proc;
    *ip = *(&proc->tf->x0 + n);             // 引数nにあたる値を`struct trampfreme`から取得
    return 0;
}
```

## mkfsを修正

```
# hello
- uload: 0xaaaa000, off=0x0, len=0x72c, sz=0xaaaa72c
- uload: 0xaaabdb8, off=0xdb8, len=0x250, sz=0xaaac010
load_interp: ualloc: old=0x0, new=0x98c04
- uload: addr=0x0, off=0x0, len=0x98c04
load_interp: ualloc: old=0x98c04, new=0xad0f8
- uload: addr=0xa9b90, off=0x99b90, len=0x89a
- elf_entry: 0x61614
0xafff0: argv[0]=hello
0xaffe0: envp[0]=PWD=/.
0xaffd8: auxv[27]=0
0xaffd0: auxv[26]=0             // AT_NULL
0xaffc8: auxv[25]=0
0xaffc0: auxv[24]=16            // AT_HWCAP
0xaffb8: auxv[23]=4096
0xaffb0: auxv[22]=6             // AT_PAGESZ
0xaffa8: auxv[21]=100
0xaffa0: auxv[20]=17            // AT_CLKTCK
0xaff98: auxv[19]=64
0xaff90: auxv[18]=3             // AT_PHDR  = file_ehdr.e_phoff
0xaff88: auxv[17]=56
0xaff80: auxv[16]=4             // AT_PHENT = sizeof(Elf64_Phdr)
0xaff78: auxv[15]=7
0xaff70: auxv[14]=5             // AT_PHNUM = file_ehdr.e_phnum
0xaff68: auxv[13]=0
0xaff60: auxv[12]=7             // AT_BASE = interp_load_addr (EP=0x61614)
0xaff58: auxv[11]=0
0xaff50: auxv[10]=8             // AT_FLAGS
0xaff48: auxv[9]=178955616      // 0xaaaa560
0xaff40: auxv[8]=9              // AT_ENTRY - load_bias + file_ehdr.e_entry = 0x560
0xaff38: auxv[7]=0
0xaff30: auxv[6]=11             // AT_UID
0xaff28: auxv[5]=0
0xaff20: auxv[4]=12             // AT_EUID
0xaff18: auxv[3]=0
0xaff10: auxv[2]=13             // AT_GID
0xaff08: auxv[1]=0
0xaff00: auxv[0]=14             // AT_EGID
0xaff00: estack[1]=0x0
0xafef8: estack[0]=0xaffe0
0xafef0: ustack[1]=0x0
0xafee8: ustack[0]=0xafff0
0xafee0: argc=1
musl libc (aarch64)
Version 1.2.2-git-50-gb76f37fd
Dynamic Program Loader
Usage: hello [options] [--] pathname [args]
# /lib/ld-musl-aarch64.so.1 /bin/hello
- uload: 0x0, off=0x0, len=0x98c04, sz=0x98c04
- uload: 0xa9b90, off=0x99b90, len=0x89a, sz=0xad0f8
0xaffe0: argv[0]=/lib/ld-musl-aarch64.so.1
0xaffd0: argv[1]=/bin/hello
0xaffc0: envp[0]=PWD=/.
0xaffb8: auxv[3]=0
0xaffb0: auxv[2]=4096
0xaffa8: auxv[1]=6
0xaffa0: auxv[0]=0
0xaffa0: estack[1]=0x0
0xaff98: estack[0]=0xaffc0
0xaff90: ustack[2]=0x0
0xaff88: ustack[1]=0xaffd0
0xaff80: ustack[0]=0xaffe0
0xaff78: argc=2
data abort: ec=0x24, iss=0x6, insruction 0x61830, fault addr 0x1ce0cd54, dfs=6
kern/console.c:284: kernel panic at cpu 0.
```

## auxv[]の設定ミスを修正

```
# /bin/hello
- uload: 0xaaaa000, off=0x0, len=0x72c, sz=0xaaaa72c
- uload: 0xaaabdb8, off=0xdb8, len=0x250, sz=0xaaac010
load_interp: ualloc: old=0x0, new=0x98c04
- uload: addr=0x0, off=0x0, len=0x98c04
load_interp: ualloc: old=0x98c04, new=0xad0f8
- uload: addr=0xa9b90, off=0x99b90, len=0x89a
- elf_entry: 0x61614
0xafff0: argv[0]=/bin/hello
0xaffe0: envp[0]=PWD=/.
0xaffd8: auxv[27]=0
0xaffd0: auxv[26]=0
0xaffc8: auxv[25]=0
0xaffc0: auxv[24]=16
0xaffb8: auxv[23]=4096
0xaffb0: auxv[22]=6
0xaffa8: auxv[21]=100
0xaffa0: auxv[20]=17
0xaff98: auxv[19]=178954304
0xaff90: auxv[18]=3
0xaff88: auxv[17]=56
0xaff80: auxv[16]=4
0xaff78: auxv[15]=7
0xaff70: auxv[14]=5
0xaff68: auxv[13]=0
0xaff60: auxv[12]=7
0xaff58: auxv[11]=0
0xaff50: auxv[10]=8
0xaff48: auxv[9]=178955616
0xaff40: auxv[8]=9
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
syscall: pid=7, x0=0xaaab000, x1=0x1000, x2=0x1, x3=0xaaacfff x4=0x1fff
Unexpected syscall #226         // SYS_mprotect
```

## sys_mprotectを実装

```
sys_mprotect: start=0xaaab000, len=4096, prot=1
- failed find_vma: start=0xaaab000
Error relocating hello: RELRO protection failed: Bad address
```

- リロケーションでmmapを想定しているが、mprotect()を実装しなければこのエラーはなくなりそう

```c
// libc/ldso/dynlink.c#reloc_all(struct dso *p)

if (head != &ldso && p->relro_start != p->relro_end
 && mprotect(laddr(p, p->relro_start), p->relro_end-p->relro_start, PROT_READ)
 && errno != ENOSYS) {
    error("Error relocating %s: RELRO protection failed: %m", p->name);
    if (runtime) longjmp(*rtld_fail, 1);
}
```

## sys_mprotect()をENOSYSを返すだけの関数にした

```
# hello
- uload: 0xaaaa000, off=0x0, len=0x72c, sz=0xaaaa72c
- uload: 0xaaabdb8, off=0xdb8, len=0x250, sz=0xaaac010
load_interp: ualloc: old=0x0, new=0x98c04
- uload: addr=0x0, off=0x0, len=0x98c04
load_interp: ualloc: old=0x98c04, new=0xad0f8
- uload: addr=0xa9b90, off=0x99b90, len=0x89a
- elf_entry: 0x61614
0xafff0: argv[0]=hello
0xaffe0: envp[0]=PWD=/.
0xaffd8: auxv[27]=0
0xaffd0: auxv[26]=0
0xaffc8: auxv[25]=0
0xaffc0: auxv[24]=16
0xaffb8: auxv[23]=4096
0xaffb0: auxv[22]=6
0xaffa8: auxv[21]=100
0xaffa0: auxv[20]=17
0xaff98: auxv[19]=178954304
0xaff90: auxv[18]=3
0xaff88: auxv[17]=56
0xaff80: auxv[16]=4
0xaff78: auxv[15]=7
0xaff70: auxv[14]=5
0xaff68: auxv[13]=0
0xaff60: auxv[12]=7
0xaff58: auxv[11]=0
0xaff50: auxv[10]=8
0xaff48: auxv[9]=178955616
0xaff40: auxv[8]=9
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
hello dynamic world!                // 正常に動いた
```

### デバッグプリントを削除

```
# hello
hello dynamic world!
# hellos
hello static world!
```

# ldを直接実行した場合

```
# /lib/ld-musl-aarch64.so.1 /bin/hello
data abort: ec=0x24, iss=0x6, insruction 0x61830, fault addr 0x1ce0cd54, dfs=6
```

- mmapアドレスの範囲外か
- page of demandが処理されていないか

## mmapアドレスの定義(memlayout.h)

```
#define MMAPBASE 0x60000000           /* Base of mmap address */
#define MMAPTOP  0x80000000           /* Top of mmap address (512MB) */
```

# 動かなくなった

```
# hello
 x0: 0x98571
 x1: 0x55b36e2
 x2: 0x464c457f
 x3: 0x55b36e4
 x4: 0x1
 x5: 0xac070
x30: 0x6299c
 sp: 0xafd50

data abort: ec=0x24, iss=0x6, dfs=6
insruction: 0x61830
fault addr: 0x156cdb90
```

## CFLAGSに`-z max-page-size=4096`を追加したら直った

```
# hello
hello dynamic world!
```

## しかし、bashはまだエラー

- カーネルメモリにアクセスしている

```
# bash

====== TRAP FRAME DUMP ======
sp:	0xafce0
tpidr:	0x432df8
spsr:	0x60000000
esr:	0x92000004
elr:	0x61760
x0:	0xfffffffffff6f8b8
x1:	0x8e8
x2:	0xfffffffffff83e08
x3:	0xfffffffffff83520
x4:	0xafee0
x29:	0x0
x30:	0x0
====== DUMP END ======

data abort: ec=0x24, iss=0x4, dfs=4
insruction: 0x61760
fault addr: 0xfffffffffff83528          // 負値だとすると -0x7cad8 = -510680
```

## readelfでエラー

```
# readelf -h /usr/local/bin/bash
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x36494
  Start of program headers:          64 (bytes into file)
  Start of section headers:          5557448 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         7
  Size of section headers:           64 (bytes)
  Number of section headers:         32
  Section header string table index: 31

syscall: pid=10, x0=0x53f000, x1=0x41c000, x2=0x4, x3=0x0 x4=0x4ff00c
Unexpected syscall #233
```

## sys_madviseを実装

```# readelf -h /usr/local/bin/bash
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x36494
  Start of program headers:          64 (bytes into file)
  Start of section headers:          5557448 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         7
  Size of section headers:           64 (bytes)
  Number of section headers:         32
  Section header string table index: 31
sys_madvise: addr=53f000, length=4308992, advice=4
#
```

# ubuntuで実行した場合のauxv

```
$ LD_SHOW_AUXV=1 /lib64/ld-linux-x86-64.so.2 ./hello_ubuntu
AT_SYSINFO_EHDR:      0x7fffdb35b000
AT_HWCAP:             178bfbff
AT_PAGESZ:            4096
AT_CLKTCK:            100
AT_PHDR:              0x7ff6548ae040
AT_PHENT:             56
AT_PHNUM:             11
AT_BASE:              0x0
AT_FLAGS:             0x0
AT_ENTRY:             0x7ff6548af100
AT_UID:               1000
AT_EUID:              1000
AT_GID:               1000
AT_EGID:              1000
AT_SECURE:            0
AT_RANDOM:            0x7fffdb2d73a9
AT_HWCAP2:            0x0
AT_EXECFN:            /lib64/ld-linux-x86-64.so.2
AT_PLATFORM:          x86_64
hello world!
```

動かない場合

```
iinit: ok
init log[1]: ok
- argv[0] = '/bin/init'
argv[1] = '(null)'
envp[0] = '(null)'
- path: /bin/init
execve: proc[1] '/bin/init' start, uid=0, gid=0
0x406ff0: argv[0]=/bin/init
0x406fe8: auxv[3]=0x0
0x406fe0: auxv[2]=0x1000
0x406fd8: auxv[1]=0x6
0x406fd0: auxv[0]=0x0
0x406fd0: estack[0]=0x0
0x406fc8: ustack[1]=0x0
0x406fc0: ustack[0]=0x406ff0
0x406fb8: argc=1
(exec)
init: starting login
init: starting login
```

動いた場合

```
iinit: ok
init log[1]: ok
sys_execve: path=/bin/init, uargv=0x20, uenvp=0x0
- argv[0] = '/bin/init'
argv[1] = '(null)'
envp[0] = '(null)'
execve: proc[1] '/bin/init' start, uid=0, gid=0
0x406ff0: argv[0]=/bin/init
0x406fe8: auxv[3]=0x0
0x406fe0: auxv[2]=0x1000
0x406fd8: auxv[1]=0x6
0x406fd0: auxv[0]=0x0
0x406fd0: estack[0]=0x0
0x406fc8: ustack[1]=0x0
0x406fc0: ustack[0]=0x406ff0
0x406fb8: argc=1
(exec)
init: starting login
sys_execve: path=/bin/dash, uargv=0x406f68, uenvp=0x406f60
- argv[0] = 'dash'
argv[1] = '(null)'
envp[0] = '(null)'
execve: proc[6] '/bin/dash' start, uid=0, gid=0
0x435ff0: argv[0]=dash
0x435fe8: auxv[3]=0x0
0x435fe0: auxv[2]=0x1000
0x435fd8: auxv[1]=0x6
0x435fd0: auxv[0]=0x0
0x435fd0: estack[0]=0x0
0x435fc8: ustack[1]=0x0
0x435fc0: ustack[0]=0x435ff0
0x435fb8: argc=1
(exec)
# ls
sys_execve: path=/bin/ls, uargv=0x433690, uenvp=0x4336a0
- argv[0] = 'ls'
argv[1] = '(null)'
- envp[0] = 'PWD=/.'
envp[1] = '(null)'
execve: proc[7] '/bin/ls' start, uid=0, gid=0
0x443ff0: argv[0]=ls
0x443fe0: envp[0]=PWD=/.
0x443fd8: auxv[3]=0x0
0x443fd0: auxv[2]=0x1000
0x443fc8: auxv[1]=0x6
0x443fc0: auxv[0]=0x0
0x443fc0: estack[1]=0x0
0x443fb8: estack[0]=0x443fe0
0x443fb0: ustack[1]=0x0
0x443fa8: ustack[0]=0x443ff0
0x443fa0: argc=1
(exec)
bin  dev  etc  home  lib  usr
```

# mmapで4バイト超のMMAPBASEを指定するとmapped addressが0になるエラーが発覚

- musl側では8バイト対応していたが、syscall関数の返り値をintとしていたためだった
- 返り値をlongに変更した
- 4バイト超の問題は直ったが、mmaptest2で下記のtestがfailする
  - F-06, F-15, F-17, F-18, F-19, A-05, A-08, A-10, A-12, A-13, A-14, O-03, O-04, O-05

```
# mmaptest2
sizeof char* = 8, unsigned long = 8
[F-01] file backed mapping invalid file descriptor test
[F-01] ok

[F-02] file backed mapping invalid flags test
mmap: invalid flags
[F-02] ok

[F-03] file backed writeable shared mapping on read only file test
mmap: file not writable
[F-03] ok

[F-04] file backed read only shared mapping on read only file test
- mapped addr=0x600000000000
[F-04] ok

[F-05] file backed private mapping permission test
[F-05] ok

[F-06] file backed exceed mapping size test
[F-06] failed                                           // <= NG

[F-07] file backed exceed mapping count test
mmap: vmas exhausted
[F-07] ok

[F-08] file mmap on empty file test
[F-08] ok

[F-09] file backed private mapping test
[F-09] ok

[F-10] file backed shared mapping test
[F-09] ok

[F-11] file backed mapping pagecache coherency test
[F-11] ok

[F-12] file backed private mapping with fork test
[F-12] ok

[F-13] file backed shared mapping with fork test
[F-13] ok

[F-14] file backed mapping with offset test
[F-14] ok

[F-15] file backed valid provided address test
[F-15]: ok

[F-16] file backed invalid provided address test
mmap: invalid addr
[F-16] ok

[F-17] file backed overlapping provided address test
munmap: not exit addr: 0x0
[F-17] failed at second munmap                          // <= NG

[F-18] file backed intermediate provided address test
check_vma_possible: vmas[1].addr=0x600000003000 >= mmap_addr=0x600000001000
[F-18] failed at third mmap: ret3 = 0x0                 // <= NG

[F-19] file backed intermediate provided address not possible test
check_vma_possible: vmas[1].addr=0x600000003000 >= mmap_addr=0x600000001000
munmap: not exit addr: 0x0
[F-19]  ok

[F-20] File exceed file size test
[F-20] ok

[F-21] file mapping on write only file
mmap: file not readable
[F-21] ok

[A-01] anonymous private mapping test
- ret=0x0x600000000000
[A-01] ok

[A-02] anonymous shared mapping test
munmap: not exit addr: 0xffff0000000f9930
munmap: not exit addr: 0xffff0000000f9930
munmap: not exit addr: 0xffff0000000f9930
[A-02] failed at ret[i]=144, i=1                        // <= NG

[A-03] anonymous private mapping with fork test
uvm_copy: no memory
fork: failed uvm_copy
[A-03] ok

[A-04] anonymous shared mapping with multiple forks test
uvm_copy: no memory
fork: failed uvm_copy
[A-04] at fork 1                                        // <= NG

[A-05] anonymous private & shared mapping together with fork test
uvm_copy: no memory
fork: failed uvm_copy
[A-05] failed at strcmp share                           // <= NG

[A-06] anonymous missing flags test
mmap: invalid flags
[A-06] ok

[A-07] anonymous exceed mapping count test
mmap: vmas exhausted
[A-07] failed 28 total mappings                         // <= NG

[A-08] anonymous exceed mapping size test
find_vma_addr: exeed MMAPTOP
mmap: vma not available
[A-08] ok

[A-09] anonymous zero size mapping test
mmap: invalid size or offset
[A-09] ok

[A-10] anonymous valid provided address test
check_vma_possible: vmas[1].addr=0xffff0000000f9930 >= mmap_addr=0x600000001000
munmap: not exit addr: 0x0
[A-10] failed at munmap                                 // <= NG

[A-11] anonymous invalid provided address test
mmap: invalid addr
[A-11] ok

[A-12] anonymous overlapping provided address test
check_vma_possible: vmas[1].addr=0xffff0000000f9930 >= mmap_addr=0x600000001000
check_vma_possible: vmas[1].addr=0xffff0000000f9930 >= mmap_addr=0x600000001000
munmap: not exit addr: 0x0
[A-12] failed at first munmap                           // <= NG
munmap: not exit addr: 0x0

[A-13] anonymous intermediate provided address test
check_vma_possible: vmas[1].addr=0x600000001000 >= mmap_addr=0x600000001000
[A-13] failed at third mmap                             // <= NG

[A-14] anonymous intermediate provided address not possible test
check_vma_possible: vmas[1].addr=0x600000001000 >= mmap_addr=0x600000001000
munmap: not exit addr: 0x0
[A-14] ok

[O-01] munmap only partial size test
map_anon_page: memory exhausted
pagefault_handler: mmap_store_data failed       // loop
```

## MMAPREGIONを見直し

- テストは通るようになった
- なぜかmmaptest2実行後にローカルエコーが無効になる
  - `stty echo`で戻るので、深追いはしない
- mmaptestはほとんどパスしない上に異常終了する
  - mmap_test: Total: 7, OK: 2, NG: 5
  - fork_test: mismatch at 6144, wanted zero, got 0x41, addr=0x600000001800

```
# mmaptest2
[F-01] file backed mapping invalid file descriptor test
[F-01] ok

[F-02] file backed mapping invalid flags test
[F-02] ok

[F-03] file backed writeable shared mapping on read only file test
[F-03] ok

[F-04] file backed read only shared mapping on read only file test
[F-04] ok

[F-05] file backed private mapping permission test
[F-05] ok

[F-06] file backed exceed mapping size test
[F-06] ok

[F-07] file backed exceed mapping count test
[F-07] ok

[F-08] file mmap on empty file test
[F-08] ok

[F-09] file backed private mapping test
[F-09] ok

[F-10] file backed shared mapping test
[F-09] ok

[F-11] file backed mapping pagecache coherency test
[F-11] ok

[F-12] file backed private mapping with fork test
[F-12] ok

[F-13] file backed shared mapping with fork test
[F-13] ok

[F-14] file backed mapping with offset test
[F-14] ok

[F-15] file backed valid provided address test
[F-15] ok

[F-16] file backed invalid provided address test
[F-16] ok

[F-17] file backed overlapping provided address test
[F-17] ok

[F-18] file backed intermediate provided address test
[F-18] ok

[F-19] file backed intermediate provided address not possible test
[F-19]  ok

[F-20] File exceed file size test
[F-20] ok

[F-21] file mapping on write only file
[F-21] ok

[A-01] anonymous private mapping test
- ret=0x0x600000000000
[A-01] ok

[A-02] anonymous shared mapping test
[A-02] ok

[A-03] anonymous private mapping with fork test
[A-03] ok

[A-04] anonymous shared mapping with multiple forks test
[A-04] ok

[A-05] anonymous private & shared mapping together with fork test
wait4: copyout failed
[A-05] ok

[A-06] anonymous missing flags test
[A-06] ok

[A-07] anonymous exceed mapping count test
[A-07] ok

[A-08] anonymous exceed mapping size test
[A-08] ok

[A-09] anonymous zero size mapping test
[A-09] ok

[A-10] anonymous valid provided address test
[A-10] ok

[A-11] anonymous invalid provided address test
[A-11] ok

[A-12] anonymous overlapping provided address test
[A-12] ok

[A-13] anonymous intermediate provided address test
[A-13] ok

[A-14] anonymous intermediate provided address not possible test
[A-14] ok

[O-01] munmap only partial size test
[O-01] ok

[O-02] write on read only mapping test
[O-02] ok

[O-03] none permission on mapping test
[O-03] ok

[O-04] mmap valid address map fixed flag test
[O-04] ok

[O-05] mmap invalid address map fixed flag test
[O-05] test ok

file_test:  ok: 21, ng: 0
anon_test:  ok: 13, ng: 0
other_test: ok: 5, ng: 0
```

## syscallの返り値をlongに、vma.offsetをoff_tに変えたがbashのエラーは変わらず

```
# bash

====== TRAP FRAME DUMP ======
sp:	0xafce0
tpidr:	0x432df8
spsr:	0x60000000
esr:	0x92000004
elr:	0x61760
x0:	0xfffffffffff6f8b8
x1:	0x8e8
x2:	0xfffffffffff83e08
x3:	0xfffffffffff83520
x4:	0xafee0
x29:	0x0
x30:	0x0
====== DUMP END ======

data abort: ec=0x24, iss=0x4, dfs=4
insruction: 0x61760
fault addr: 0xfffffffffff83528
```

```
# /lib/ld-musl-aarch64.so.1 /bin/hello
unexpected interrupt 0 at cpu 0

====== TRAP FRAME DUMP ======
sp:	0xaff78
tpidr:	0xabf40
spsr:	0x20000000
esr:	0x2000000
elr:	0x0
x0:	0x0
x1:	0x0
x2:	0x1
x3:	0x279d785
x4:	0x1
x29:	0xafe40
x30:	0x6466c
====== DUMP END ======

Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x0
 SPSR_EL1 0x20000000 FAR_EL1 0x9e75e14
	irq_error: irq of type 8 unimplemented.
```

# raspi版Ubuntuでの実行結果

```sh
$ LD_SHOW_AUXV=1 ls -al
AT_SYSINFO_EHDR:      0xffffb0df4000
AT_??? (0x33):        0x1270              // AT_MINSIGSTKSZ
AT_HWCAP:             887
AT_PAGESZ:            4096
AT_CLKTCK:            100
AT_PHDR:              0xaaaad944e040
AT_PHENT:             56
AT_PHNUM:             9
AT_BASE:              0xffffb0dc4000
AT_FLAGS:             0x0
AT_ENTRY:             0xaaaad9453940
AT_UID:               1000
AT_EUID:              1000
AT_GID:               1000
AT_EGID:              1000
AT_SECURE:            0
AT_RANDOM:            0xfffff24afc18
AT_HWCAP2:            0x0
AT_EXECFN:            /usr/bin/ls
AT_PLATFORM:          aarch64
total 36
drwxr-xr-x 4 ubuntu ubuntu 4096 Jan 13 16:54 .
drwxr-xr-x 4 root   root   4096 Jan 13 17:25 ..
-rw------- 1 ubuntu ubuntu  803 Jan 17 17:34 .bash_history
...
$ ps
    PID TTY          TIME CMD
   2614 pts/0    00:00:00 bash
   2631 pts/0    00:00:00 ps
$ cat /proc/2614/maps
addres start-end          perm offset   maj:min inode                    permのp=private, s=shared
aaaaccb2b000-aaaaccc46000 r-xp 00000000 b3:02 1559                       /usr/bin/bash
aaaaccc56000-aaaaccc5b000 r--p 0011b000 b3:02 1559                       /usr/bin/bash
aaaaccc5b000-aaaaccc64000 rw-p 00120000 b3:02 1559                       /usr/bin/bash
aaaaccc64000-aaaaccc6d000 rw-p 00000000 00:00 0                          [bss]
aaaad6833000-aaaad69a2000 rw-p 00000000 00:00 0                          [heap]
ffff8b8ee000-ffff8b8f9000 r-xp 00000000 b3:02 3202                       /usr/lib/aarch64-linux-gnu/libnss_files-2.31.so
ffff8b8f9000-ffff8b909000 ---p 0000b000 b3:02 3202                       /usr/lib/aarch64-linux-gnu/libnss_files-2.31.so
ffff8b909000-ffff8b90a000 r--p 0000b000 b3:02 3202                       /usr/lib/aarch64-linux-gnu/libnss_files-2.31.so
ffff8b90a000-ffff8b90b000 rw-p 0000c000 b3:02 3202                       /usr/lib/aarch64-linux-gnu/libnss_files-2.31.so
ffff8b90b000-ffff8b911000 rw-p 00000000 00:00 0
ffff8b917000-ffff8b949000 r--p 00000000 b3:02 8150                       /usr/lib/locale/C.UTF-8/LC_CTYPE
ffff8b949000-ffff8babc000 r--p 00000000 b3:02 8149                       /usr/lib/locale/C.UTF-8/LC_COLLATE
ffff8babc000-ffff8bac3000 r--s 00000000 b3:02 2806                       /usr/lib/aarch64-linux-gnu/gconv/gconv-modules.cache
ffff8bac3000-ffff8bda9000 r--p 00000000 b3:02 8161                       /usr/lib/locale/locale-archive
ffff8bda9000-ffff8bf03000 r-xp 00000000 b3:02 2921                       /usr/lib/aarch64-linux-gnu/libc-2.31.so
ffff8bf03000-ffff8bf13000 ---p 0015a000 b3:02 2921                       /usr/lib/aarch64-linux-gnu/libc-2.31.so
ffff8bf13000-ffff8bf16000 r--p 0015a000 b3:02 2921                       /usr/lib/aarch64-linux-gnu/libc-2.31.so
ffff8bf16000-ffff8bf19000 rw-p 0015d000 b3:02 2921                       /usr/lib/aarch64-linux-gnu/libc-2.31.so
ffff8bf19000-ffff8bf1c000 rw-p 00000000 00:00 0
ffff8bf1c000-ffff8bf1f000 r-xp 00000000 b3:02 2960                       /usr/lib/aarch64-linux-gnu/libdl-2.31.so
ffff8bf1f000-ffff8bf2e000 ---p 00003000 b3:02 2960                       /usr/lib/aarch64-linux-gnu/libdl-2.31.so
ffff8bf2e000-ffff8bf2f000 r--p 00002000 b3:02 2960                       /usr/lib/aarch64-linux-gnu/libdl-2.31.so
ffff8bf2f000-ffff8bf30000 rw-p 00003000 b3:02 2960                       /usr/lib/aarch64-linux-gnu/libdl-2.31.so
ffff8bf30000-ffff8bf59000 r-xp 00000000 b3:02 3335                       /usr/lib/aarch64-linux-gnu/libtinfo.so.6.2
ffff8bf59000-ffff8bf69000 ---p 00029000 b3:02 3335                       /usr/lib/aarch64-linux-gnu/libtinfo.so.6.2
ffff8bf69000-ffff8bf6d000 r--p 00029000 b3:02 3335                       /usr/lib/aarch64-linux-gnu/libtinfo.so.6.2
ffff8bf6d000-ffff8bf6e000 rw-p 0002d000 b3:02 3335                       /usr/lib/aarch64-linux-gnu/libtinfo.so.6.2
ffff8bf6e000-ffff8bf8f000 r-xp 00000000 b3:02 2842                       /usr/lib/aarch64-linux-gnu/ld-2.31.so
ffff8bf8f000-ffff8bf90000 r--p 00000000 b3:02 8157                       /usr/lib/locale/C.UTF-8/LC_NUMERIC
ffff8bf90000-ffff8bf91000 r--p 00000000 b3:02 8160                       /usr/lib/locale/C.UTF-8/LC_TIME
ffff8bf91000-ffff8bf92000 r--p 00000000 b3:02 8155                       /usr/lib/locale/C.UTF-8/LC_MONETARY
ffff8bf92000-ffff8bf93000 r--p 00000000 b3:02 8154                       /usr/lib/locale/C.UTF-8/LC_MESSAGES/SYS_LC_MESSAGES
ffff8bf93000-ffff8bf97000 rw-p 00000000 00:00 0
ffff8bf97000-ffff8bf98000 r--p 00000000 b3:02 8158                       /usr/lib/locale/C.UTF-8/LC_PAPER
ffff8bf98000-ffff8bf99000 r--p 00000000 b3:02 8156                       /usr/lib/locale/C.UTF-8/LC_NAME
ffff8bf99000-ffff8bf9a000 r--p 00000000 b3:02 8148                       /usr/lib/locale/C.UTF-8/LC_ADDRESS
ffff8bf9a000-ffff8bf9b000 r--p 00000000 b3:02 8159                       /usr/lib/locale/C.UTF-8/LC_TELEPHONE
ffff8bf9b000-ffff8bf9c000 r--p 00000000 b3:02 8152                       /usr/lib/locale/C.UTF-8/LC_MEASUREMENT
ffff8bf9c000-ffff8bf9d000 r--p 00000000 b3:02 8151                       /usr/lib/locale/C.UTF-8/LC_IDENTIFICATION
ffff8bf9d000-ffff8bf9e000 r--p 00000000 00:00 0                          [vvar]
ffff8bf9e000-ffff8bf9f000 r-xp 00000000 00:00 0                          [vdso]
ffff8bf9f000-ffff8bfa0000 r--p 00021000 b3:02 2842                       /usr/lib/aarch64-linux-gnu/ld-2.31.so
ffff8bfa0000-ffff8bfa2000 rw-p 00022000 b3:02 2842                       /usr/lib/aarch64-linux-gnu/ld-2.31.so
ffffc92f3000-ffffc9314000 rw-p 00000000 00:00 0                          [stack]
```


# i386の`get_user`

```c
char ** argv

if (get_user(p, argv))
```

- include/asm-i386/uaccess.h

```c
#define __get_user_x(size,ret,x,ptr) \
    __asm__ __volatile__("call __get_user_" #size \
        :"=a" (ret),"=d" (x) \
        :"0" (ptr))

#define get_user(x,ptr)                         \
({  int __ret_gu,__val_gu;                      \
    switch(sizeof (*(ptr))) {                   \
    case 1:  __get_user_x(1,__ret_gu,__val_gu,ptr); break;      \
    case 2:  __get_user_x(2,__ret_gu,__val_gu,ptr); break;      \
    case 4:  __get_user_x(4,__ret_gu,__val_gu,ptr); break;      \
    default: __get_user_x(X,__ret_gu,__val_gu,ptr); break;      \
    }                               \
    (x) = (__typeof__(*(ptr)))__val_gu;             \
    __ret_gu;                           \
})
```

- arch/i386/lib/getuser.S

```asm
.globl __get_user_4
__get_user_4:
    addl $3,%eax              // eax = ptr + 3
    movl %esp,%edx            // edx = sp
    jc bad_get_user
    andl $0xffffe000,%edx     //
    cmpl addr_limit(%edx),%eax
    jae bad_get_user
3:  movl -3(%eax),%edx
    xorl %eax,%eax            // eax = 0
    ret

bad_get_user:
    xorl %edx,%edx          // edx = 0
    movl $-14,%eax          // eax = -EFAULT
    ret
```

```
# ls -al
execve: path=/bin/ls, argv=0xffff00003abd3d30, envp=0xffff00003abd3b30
```

# bprm.filenameが見られなくなる

`copy_strings()`内の`fileread(bprm->file, bprm->buf, BINPRM_BUF_SIZE);`の前後

```
(gdb) p *bprm
$1 = {
  buf = '\000' <repeats 127 times>,
  page = {0x0 <repeats 32 times>},
  p = 131064,
  sh_bang = 0,
  file = 0xffff00000010e050 <ftable+24>,
  fd = 0,
  e_uid = 0,
  e_gid = 0,
  cap_inheritable = 4294967295,
  cap_permitted = 4294967295,
  cap_effective = 4294967295,
  argc = 1,
  envc = 0,
  filename = 0x14 "/bin/init",
  loader = 0,
  exec = 0
}

(gdb) n
237	    cprintf("prepare_bprm9: filename=%s\n", bprm->filename);
(gdb) p *bprm
$3 = {
  buf = "\177ELF\002\001\001\000\000\000\000\000\000\000\000\000\002\000\267\000\001\000\000\000h\002@\000\000\000\000\000@\000\000\000\000\000\000\000\060_\000\000\000\000\000\000\000\000\000\000@\000\070\000\004\000@\000\021\000\020\000\001\000\000\000\005", '\000' <repeats 13 times>, "@\000\000\000\000\000\000\000@\000\000\000\000\000t\"\000\000\000\000\000\000t\"\000\000\000\000\000\000\000\020\000\000\000\000\000\000\001\000\000\000\006\000\000",
  page = {0x0 <repeats 32 times>},
  p = 131064,
  sh_bang = 0,
  file = 0xffff00000010e050 <ftable+24>,
  fd = 0,
  e_uid = 0,
  e_gid = 0,
  cap_inheritable = 4294967295,
  cap_permitted = 4294967295,
  cap_effective = 4294967295,
  argc = 1,
  envc = 0,
  filename = 0x14 <error: Cannot access memory at address 0x14>,
  loader = 0,
  exec = 0
}

addr: path=/bin/init, path=0x14, argc=1, envc=0, argv=/bin/init (0xffff00003affed30), envp=(null) (0xffff00003affeb30), bprm=0xffff00003affe8f8
```

## `filename = 0x14 <error: Cannot access memory at address 0x14>`の原因

sys_execve()でpath, argv[], envp[]を設定しているが、pathのアドレス、argv[], envp[]に
設定される文字列のアドレスはユーザ空間にあるため。カーネルでメモリを確保してコピーした
後使用するようにした。（使い終わったらkmfreeする）

## 割り込みが置きまくって最終的にpanic

```
execve: proc[1] '/bin/init' start, uid=0, gid=0
setup_arg_pages: stack_base=0xfffffffe0000
- put_dirty_page: address=0xfffffffff000
-- pa=0x3afbb000
pf_handler: ec=37, addr=0x3afbb000, flt=1
- FLT_TRANSLATION (pte=0): fault_addr: 0x3afbb000, pte=0xffff00003afb8dd8, *pte=0x0
pf_handler: ec=37, addr=0x0, flt=1
- FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0xffff00003afb7000, *pte=0x0
pf_handler: ec=37, addr=0x0, flt=1
- FLT_TRANSLATION (pte): fault_addr: 0, pte=0xffff00003afb7000, *pte=0x43
pf_handler: ec=37, addr=0x0, flt=2
 PRINT mmap_region: pid=1, nregions=1
 - region[1]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x132, fd=-1, offset=0x0
pf_handler: fault_addr: 0x0, flt: 2

====== TRAP FRAME DUMP ======
sp:	0x1000
tpidr:	0x0
spsr:	0x0
esr:	0x56000000
elr:	0x10
x0:	0x14
x1:	0x20
x2:	0x0
x3:	0x0
x4:	0x0
x29:	0x0
x30:	0x0
====== DUMP END ======

pagefault handler
```

## `setup_arg_pages()`のi=31の時だけ page != 0 となり、`put_dirty_page()`の`memmove((void *)pa, page->page, PGSIZE)`で、paにメモリが背とされておらずエラーになっている

```
(gdb) p/x pa
$7 = 0x3afbb000
(gdb) p/x *pa
Cannot access memory at address 0x3afbb000
(gdb) p/x address
$8 = 0xfffffffff000
```

```c
for (int i = 0; i < MAX_ARG_PAGES; i++) {
        struct page *page = bprm->page[i];
        if (page) {
            bprm->page[i] = NULL;
            if (page->page)
                //kmfree(page->page);
                put_dirty_page(page, stack_base);
        }
        stack_base += PGSIZE;
    }
```

```
(gdb) p/x page
$39 = 0xffff00003afe0fa0
(gdb) p i
$40 = 31
(gdb) p/x page->page
$41 = 0xffff00003afde000
```

## `setup_arg_pages()`のmmapにおける`map_anon_page()`

```
perm = 0x200000000007c7

map_anon_page: map addr=0xfffffffe0000, page=0x3afdd000
map_anon_page: map addr=0xfffffffe1000, page=0x3afd9000
map_anon_page: map addr=0xfffffffe2000, page=0x3afd8000
map_anon_page: map addr=0xfffffffe3000, page=0x3afd7000
map_anon_page: map addr=0xfffffffe4000, page=0x3afd6000
map_anon_page: map addr=0xfffffffe5000, page=0x3afd5000
map_anon_page: map addr=0xfffffffe6000, page=0x3afd4000
map_anon_page: map addr=0xfffffffe7000, page=0x3afd3000
map_anon_page: map addr=0xfffffffe8000, page=0x3afd2000
map_anon_page: map addr=0xfffffffe9000, page=0x3afd1000
map_anon_page: map addr=0xfffffffea000, page=0x3afd0000
map_anon_page: map addr=0xfffffffeb000, page=0x3afcf000
map_anon_page: map addr=0xfffffffec000, page=0x3afce000
map_anon_page: map addr=0xfffffffed000, page=0x3afcd000
map_anon_page: map addr=0xfffffffee000, page=0x3afcc000
map_anon_page: map addr=0xfffffffef000, page=0x3afcb000
map_anon_page: map addr=0xffffffff0000, page=0x3afca000
map_anon_page: map addr=0xffffffff1000, page=0x3afc9000
map_anon_page: map addr=0xffffffff2000, page=0x3afc8000
map_anon_page: map addr=0xffffffff3000, page=0x3afc7000
map_anon_page: map addr=0xffffffff4000, page=0x3afc6000
map_anon_page: map addr=0xffffffff5000, page=0x3afc5000
map_anon_page: map addr=0xffffffff6000, page=0x3afc4000
map_anon_page: map addr=0xffffffff7000, page=0x3afc3000
map_anon_page: map addr=0xffffffff8000, page=0x3afc2000
map_anon_page: map addr=0xffffffff9000, page=0x3afc1000
map_anon_page: map addr=0xffffffffa000, page=0x3afc0000
map_anon_page: map addr=0xffffffffb000, page=0x3afbf000
map_anon_page: map addr=0xffffffffc000, page=0x3afbe000
map_anon_page: map addr=0xffffffffd000, page=0x3afbd000
map_anon_page: map addr=0xffffffffe000, page=0x3afbc000
map_anon_page: map addr=0xfffffffff000, page=0x3afbb000
setup_arg_pages: stack_base=0xfffffffe0000
- stack_base=0x1000000000000

```

## do_execve()からのbprm関係のcallstack

```
fileopen(path)
pgdir_init()
prepare_binprm(&bprm)   # bprm->bufは実行ファイル
copy_strings(filename)
copy_strings(envp)
copy_strings(argv)
                        # <= (1)
search_binary_handler(&bprm)
  load_elf_binary(&bprm) # dynamic linkの場合bprm->bufはインタプリタ
  flush_old_exec()
    free_mmap_list(p)
    flush_signal_handlers(p)
  setup_arg_pages(bprm)
    put_dirty_page()
  set_brk(bss, brk)
  elf_map(vaddr)
  load_elf_interp()   # dynamic link only
  compute_creds
  create_elf_tables()
  set_brk()
  pdzero()
  uvm_switch(p)
```

## (1)の時点でのbprm

```
(gdb) p bprm
$1 = {
  buf = "\177ELF\002\001\001\000\000\000\000\000\000\000\000\000\002\000\267\000\001\000\000\000h\002@\000\000\000\000\000@\000\000\000\000\000\000\000\060_\000\000\000\000\000\000\000\000\000\000@\000\070\000\004\000@\000\021\000\020\000\001\000\000\000\005", '\000' <repeats 13 times>, "@\000\000\000\000\000\000\000@\000\000\000\000\000t\"\000\000\000\000\000\000t\"\000\000\000\000\000\000\000\020\000\000\000\000\000\000\001\000\000\000\006\000\000",
  page = {0x0 <repeats 31 times>, 0xffff00003afe0fa0},
  p = 131044,   # 0x1ffe4
  sh_bang = 0,
  file = 0xffff00000010f050 <ftable+24>,
  fd = 0,
  e_uid = 0,
  e_gid = 0,
  cap_inheritable = 4294967295,
  cap_permitted = 4294967295,
  cap_effective = 4294967295,
  argc = 1,
  envc = 0,
  filename = 0xffff00003afe0fd0 "/bin/init",
  loader = 0,
  exec = 131054    # 0x1ffee
}
(gdb) p/x bprm.page[0]
$11 = 0x0
(gdb) p/x bprm.page[31]
$12 = 0xffff00003afe0fa0
(gdb) p/x bprm.page[31]->page
$13 = 0xffff00003afde000
(gdb) x/16bx bprm.page[31]->page
0xffff00003afde000:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0xffff00003afde008:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
```

## copy_strings(1, &bprm.filename, &bprm)

```
(gdb) s
copy_strings (argc=1, argv=0xffff00003affea88, bprm=0xffff00003affe8d0)
    at kern/exec.c:120
(gdb) p/x argv
$9 = 0xffff00003affea88
(gdb) p/x *argv
$10 = 0xffff00003afe0fd0
(gdb) p *argv
$11 = 0xffff00003afe0fd0 "/bin/init"
(gdb) p argv[0]
$12 = 0xffff00003afe0fd0 "/bin/init"
(gdb) p str
$13 = 0xffff00003afe0fd0 "/bin/init"
(gdb) p strlen(str)
$14 = 9
(gdb) p bprm->p
$15 = 131064                  # 0x1fff8 = stack_base
(gdb) p pos
$16 = 131055                  # 0x1ffef
(gdb) p bprm->p
$17 = 131055
(gdb) p offset
$18 = 4079                    # 0xfef
(gdb) p i
$19 = 31
(gdb) p/x page
$20 = 0x0
(gdb) n
149	                page = (struct page *)kmalloc(sizeof(struct page));
(gdb)
150	                bprm->page[i] = page;
(gdb) p/x page
$21 = 0xffff00003afe0fa0
(gdb)
153	                page->page = kalloc();
(gdb) p/x page->page
$22 = 0xffff00003afde000
(gdb) p/x kaddr
$23 = 0xffff00003afde000
(gdb)
165	            bytes_to_copy = PGSIZE - offset;
(gdb)
166	            if (bytes_to_copy > len) {
(gdb) p bytes_to_copy
$26 = 17
(gdb) p len
$27 = 9
(gdb) p bytes_to_copy
$26 = 9
(gdb)
171	            memmove(kaddr + offset, str, bytes_to_copy);
(gdb) p bprm.page[31]->page
$51 = 0xffff00003afde000 ""
(gdb) x/17bx kaddr+offset
0xffff00003afdefef:	0x2f	0x62	0x69	0x6e	0x2f	0x69	0x6e	0x69  # /bin/ini
0xffff00003afdeff7:	0x74	0x00	0x00	0x00	0x00	0x00	0x00	0x00  # t
0xffff00003afdefff:	0x00
```

## copy_strings(bprm.argc, argv, &bprm)呼び出し後

filenameとargv[0]の後ろのnullがない

```
(gdb) x/32bx bprm.page[31]->page+0xfe6
0xffff00003afdefe6:	0x2f	0x62	0x69	0x6e	0x2f	0x69	0x6e	0x69  # /bin/ini
0xffff00003afdefee:	0x74	0x2f	0x62	0x69	0x6e	0x2f	0x69	0x6e  # t/bin/in
0xffff00003afdeff6:	0x69	0x74	0x00	0x00	0x00	0x00	0x00	0x00  # it
0xffff00003afdeffe:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
```

## 修正: copy_stringsまではうまく動いている

```
(gdb) x/32bx bprm.page[31]->page+0xfe4
0xffff00003afdefe4:	0x2f	0x62	0x69	0x6e	0x2f	0x69	0x6e	0x69  # /bin/ini    <= argv[0]
0xffff00003afdefec:	0x74	0x00	0x2f	0x62	0x69	0x6e	0x2f	0x69  # t /bin/i    <= filename
0xffff00003afdeff4:	0x6e	0x69	0x74	0x00	0x00	0x00	0x00	0x00  # nit
0xffff00003afdeffc:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
```

## load_elf_binary()のelf_map()を2回終わったところまでは動いている

```
execve: proc[1] '/bin/init' start, uid=0, gid=0
mmap: addr=0xfffffffe0000, length=131072, prot=0x7, flags=0x132, fd=-1, offset=0x0
, addr2=0xfffffffe0000, addr9=0xfffffffe0000
setup_arg_pages: stack_base=0xfffffffe0000
call put_dirty_page: i=31
- put_dirty_page: address=0xfffffffff000
-- pa=0x3afbb000, page=0xffff00003afde000
pf_handler: ec=37, addr=0x3afbb000, flt=1
pf_handler: ec=37, addr=0x0, flt=1
- FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
- FLT_TRANSLATION (pte=0): fault_addr: 0x3afbb000, pte=0x0, *pte=0x0
-- move end
- stack_base=0x1000000000000
setup_arg_pages end
mmap: addr=0x400000, length=8820, prot=0x5, flags=0x1812, fd=0, offset=0x0
, addr2=0x400000, addr3=0x600000400000, node=0xffff00003afe0e70
addr: 600000400000, node_addr: fffffffe0000
, addr9=0x600000400000
- mapped_addr=0x600000400000
mmap: addr=0x403000, length=4360, prot=0x3, flags=0x1812, fd=0, offset=0x2000
, addr2=0x403000, addr3=0x600000403000, node=0xffff00003afe0e30
addr: 600000403000, node_addr: 600000400000
, addr9=0x600000403000
- mapped_addr=0x600000403000      # <= 2回目のelf_map()の結果

pf_handler: ec=37, addr=0xffffffffffdc, flt=3
pf_handler: fault_addr: 0xfffffffff000, flt: 3

====== TRAP FRAME DUMP ======
sp:	0x1000
tpidr:	0x0
spsr:	0x0
esr:	0x56000000
elr:	0x10
x0:	0x14
x1:	0x20
x2:	0x0
x3:	0x0
x4:	0x0
x29:	0x0
x30:	0x0
====== DUMP END ======
```

## init実行まで進んだがエラー

```
  Entry point address:               0x400268

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x0000000000002274 0x0000000000002274  R E    0x1000
  LOAD           0x0000000000002eb0 0x0000000000403eb0 0x0000000000403eb0
                 0x0000000000000258 0x00000000000008e0  RW     0x1000
 ```

```
entry=0x400268, bss=0x404108, brk=0x404790    # bss = 0x403eb0 + 0x258
start_code=0x400000, end_code=0x402274        # brk = 0x403eb0 + 0x8e0
start_data=0x403eb0, end_data=0x404108
bprm->p=0x0
== UVM_SWITCH ==
trap: CPU 0 unexpected irq: 0x20, iss: 0x7, sp: 0.
Synchronous: Instruction abort, lower EL:
  ESR_EL1 0x82000007 ELR_EL1 0x400268
 SPSR_EL1 0x0 FAR_EL1 0x400268
	irq_error: irq of type 8 unimplemented.
```

## create_elf_tables()に渡されるpの値がおかしい

```
(gdb) x/16bx argv
0xfffffffffec8:	0xe4	0xff	0xff	0xff	0xff	0xff	0x00	0x00
0xfffffffffed0:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
(gdb) x/16bx *argv
0xffffffffffe4:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0xffffffffffec:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
(gdb) p/x p
$19 = 0xffffffffffe4
(gdb) n
138	        if (!len || len > PAGE_SIZE * MAX_ARG_PAGES)
(gdb) p len
$20 = 0
```

```
(gdb) p u_platform
$1 = 0xffffffffffdc ""
(gdb) x/16bx u_platform
0xffffffffffdc:	0x00	0x61	0x72	0x63	0x68	0x36	0x34	0x00  # .arch64.
0xffffffffffe4:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
(gdb) p k_platform
$2 = 0xffff0000000a44a0 "aarch64"
(gdb) p u_platform+1
$3 = 0xffffffffffdd "arch64"
(gdb) x/48gx 0xfffffffffe80
0xfffffffffe80:	0x0000000000000000	0x0000000000000000
0xfffffffffe90:	0x0000000000000000	0x0000000000000000
0xfffffffffea0:	0x0000000000000000	0x0000000000000000
0xfffffffffeb0:	0x0000000000000000	0x0000000000000000
0xfffffffffec0:	0x0000000000000001	0x0000ffffffffffe4
0xfffffffffed0:	0x0000000000000000	0x0000000000000000
0xfffffffffee0:	0x0000000000000010	0x0000000000000377
0xfffffffffef0:	0x0000000000000006	0x0000000000001000
0xffffffffff00:	0x0000000000000011	0x0000000000000064
0xffffffffff10:	0x0000000000000003	0x0000000000400040
0xffffffffff20:	0x0000000000000004	0x0000000000000038
0xffffffffff30:	0x0000000000000005	0x0000000000000004
0xffffffffff40:	0x0000000000000007	0x0000000000000000
0xffffffffff50:	0x0000000000000008	0x0000000000000000
0xffffffffff60:	0x0000000000000009	0x0000000000400268
0xffffffffff70:	0x000000000000000b	0x0000000000000000
0xffffffffff80:	0x000000000000000c	0x0000000000000000
0xffffffffff90:	0x000000000000000d	0x0000000000000000
0xffffffffffa0:	0x000000000000000e	0x0000000000000000
0xffffffffffb0:	0x000000000000000f	0x0000ffffffffffdc
0xffffffffffc0:	0x0000000000000000	0x0000000000000000
0xffffffffffd0:	0x0000000000000000	0x6372610000000000
0xffffffffffe0:	0x0000000000343668	0x0000000000000000
0xfffffffffff0:	0x0000000000000000	0x0000000000000000
```

## raspi linuxでのstatic link program

```
$ cat /proc/19255/maps
00400000-00477000 r-xp 00000000 b3:02 636540                             /home/dspace/work/c/hello
00487000-0048a000 r--p 00077000 b3:02 636540                             /home/dspace/work/c/hello
0048a000-0048c000 rw-p 0007a000 b3:02 636540                             /home/dspace/work/c/hello
0048c000-0048e000 rw-p 00000000 00:00 0
3665b000-3667d000 rw-p 00000000 00:00 0                                  [heap]
ffffb1dd3000-ffffb1dd4000 r--p 00000000 00:00 0                          [vvar]
ffffb1dd4000-ffffb1dd5000 r-xp 00000000 00:00 0                          [vdso]
ffffeb815000-ffffeb836000 rw-p 00000000 00:00 0                          [stack]
```

## MMAPBASEを廃止

```
execve: proc[1] '/bin/init' start, uid=0, gid=0
mmap: addr=0xfffffffe0000, length=131072, prot=0x7, flags=0x132, fd=-1, offset=0x0
, addr2=0xfffffffe0000, addr9=0xfffffffe0000
setup_arg_pages: stack_base=0xfffffffe0000
call put_dirty_page: i=31
- put_dirty_page: address=0xfffffffff000
-- pa=0x3afbb000, page=0xffff00003afde000
pf_handler: ec=37, addr=0x3afbb000, flt=1
pf_handler: ec=37, addr=0x0, flt=1
- FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
- FLT_TRANSLATION (pte=0): fault_addr: 0x3afbb000, pte=0x0, *pte=0x0
-- move end
- stack_base=0x1000000000000
setup_arg_pages end
mmap: addr=0x400000, length=8820, prot=0x5, flags=0x1812, fd=0, offset=0x0
, addr2=0x400000, node=0xffff00003afe0e70
addr: 400000, node_addr: fffffffe0000
, addr9=0x400000
- mapped_addr=0x400000
mmap: addr=0x403000, length=4360, prot=0x3, flags=0x1812, fd=0, offset=0x2000
, addr2=0x403000, node=0xffff00003afe0e30
addr: 403000, node_addr: 400000
, addr9=0x403000
- mapped_addr=0x403000
pf_handler: ec=37, addr=0xffffffffffdc, flt=3
pf_handler: ec=37, addr=0x404108, flt=3
entry=0x400268, bss=0x404108, brk=0x404790
start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bprm->p=0x0
 PRINT mmap_region: pid=1, nregions=3
 - region[1]: addr=0x400000, length=0x2274, prot=0x5, flags=0x1812, fd=1, offset=0x0
 - region[2]: addr=0x403000, length=0x1108, prot=0x3, flags=0x1812, fd=2, offset=0x2000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x132, fd=-1, offset=0x0
== UVM_SWITCH ==
pf_handler: ec=36, addr=0xffffffffffffffe0, flt=1
- FLT_TRANSLATION (pte): fault_addr: 0xfffffffffffff000, pte=0xffff00003afdaff8, *pte=0x2000003af90747
```

```
execve: proc[1] '/bin/init' start, uid=0, gid=0
mmap: addr=0xfffffffe0000, length=131072, prot=0x7, flags=0x132, fd=-1, offset=0x0
, addr2=0xfffffffe0000, addr9=0xfffffffe0000
setup_arg_pages: stack_base=0xfffffffe0000
call put_dirty_page: i=31
- put_dirty_page: address=0xfffffffff000
-- pa=0x3afbb000, *pte=0x2000003afbb7c7, page=0xffff00003afde000                # このaddress=0xfffffffff000のpaと
pf_handler: ec=37, addr=0x3afbb000, flt=1
pf_handler: ec=37, addr=0x0, flt=1
- FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
- FLT_TRANSLATION (pte=0): fault_addr: 0x3afbb000, pte=0x0, *pte=0x0
-- move end
- stack_base=0x1000000000000
setup_arg_pages end
mmap: addr=0x400000, length=8820, prot=0x5, flags=0x1812, fd=0, offset=0x0
, addr2=0x400000, node=0xffff00003afe0e70
addr: 400000, node_addr: fffffffe0000
, addr9=0x400000
- mapped_addr=0x400000
mmap: addr=0x403000, length=4360, prot=0x3, flags=0x1812, fd=0, offset=0x2000
, addr2=0x403000, node=0xffff00003afe0e30
addr: 403000, node_addr: 400000
, addr9=0x403000
- mapped_addr=0x403000
pf_handler: ec=37, addr=0xffffffffffdc, flt=3
get_physical_page: addr=0xffffffffffe4, pa=0x3af90000, *pte=0x2000003af90747      # このpaが違う
// get_physical_page: addr=0xfffffffff000, pa=0x3af90000, *pte=0x2000003af90747   # addrを強制的に変えてもpaは同じ
pf_handler: ec=37, addr=0x3af90fe4, flt=1
- FLT_TRANSLATION (pte): fault_addr: 0x3af90000, pte=0xffff00003afb7c80, *pte=0x43
pf_handler: ec=37, addr=0x3af90fe5, flt=2
 PRINT mmap_region: pid=1, nregions=3
 - region[1]: addr=0x400000, length=0x2274, prot=0x5, flags=0x1812, fd=3, offset=0x0
 - region[2]: addr=0x403000, length=0x1108, prot=0x3, flags=0x1812, fd=4, offset=0x2000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x132, fd=-1, offset=0x0
pf_handler: fault_addr: 0x3af90000, flt: 2

====== TRAP FRAME DUMP ======
sp:	0x1000
tpidr:	0x0
spsr:	0x0
esr:	0x56000000
elr:	0x10
x0:	0x14
x1:	0x20
x2:	0x0
x3:	0x0
x4:	0x0
x29:	0x0
x30:	0x0
====== DUMP END ======
```

## put_dirty_pages()を廃止

- すでにpageはanonymous mmap時にページが作成されているので、
  そこにコピーすれば良いのでmap_region()ではなく、単にmemmove()
  すればよかった。
- initが動き出したが、エラー発生

```
execve: proc[1] '/bin/init' start, uid=0, gid=0
mmap: addr=0xfffffffe0000, length=131072, prot=0x7, flags=0x132, fd=-1, offset=0x0
, addr2=0xfffffffe0000, addr9=0xfffffffe0000
setup_arg_pages: stack_base=0xfffffffe0000
call put_dirty_page: i=31
pf_handler: ec=37, addr=0xfffffffff000, flt=3
pf_handler: ec=37, addr=0x0, flt=1
- FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
- stack_base=0x1000000000000
setup_arg_pages end
mmap: addr=0x400000, length=8820, prot=0x5, flags=0x1812, fd=0, offset=0x0
, addr2=0x400000, node=0xffff00003afe0e70
addr: 400000, node_addr: fffffffe0000
, addr9=0x400000
- mapped_addr=0x400000
mmap: addr=0x403000, length=4360, prot=0x3, flags=0x1812, fd=0, offset=0x2000
, addr2=0x403000, node=0xffff00003afe0e30
addr: 403000, node_addr: 400000
, addr9=0x403000
- mapped_addr=0x403000
pf_handler: ec=37, addr=0x404108, flt=3
entry=0x400268, bss=0x404108, brk=0x404790
start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bprm->p=0xfffffffffec0
 PRINT mmap_region: pid=1, nregions=3
 - region[1]: addr=0x400000, length=0x2274, prot=0x5, flags=0x1812, fd=3, offset=0x0
 - region[2]: addr=0x403000, length=0x1108, prot=0x3, flags=0x1812, fd=4, offset=0x2000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x132, fd=-1, offset=0x0
== UVM_SWITCH ==
init: starting dash                       # <= dashがinitが動き、dashの起動でエラー
pf_handler: ec=37, addr=0x4021a0, flt=3
pf_handler: fault_addr: 0x402000, flt: 3

====== TRAP FRAME DUMP ======
sp:	0xfffffffffd70
tpidr:	0x404630
spsr:	0x60000000
esr:	0x56000000
elr:	0x400938
x0:	0x0
x1:	0x4021a0
x2:	0xfffffffffdb0
x3:	0x8
x4:	0xfffffffffdff
x29:	0xfffffffffe30
x30:	0x400784
====== DUMP END ======
```

## init実行時にfork()で設定したp->headが消えている

```
init log[1]: ok
proc[1] calls sys_execve
pf_handler: proc[1], head=0xffff00003afe0e70, ec=37, addr=0xfffffffff000, flt=3
pf_handler: proc[1], head=0xffff00003afe0e70, ec=37, addr=0x0, flt=1
pf_handler: proc[1], head=0xffff00003afe0e30, ec=37, addr=0x404108, flt=3
== UVM_SWITCH ==
proc[1] calls sys_gettid
proc[1] calls sys_openat
proc[1] calls sys_dup
proc[1] calls sys_dup
proc[1] calls sys_ioctl
proc[1] calls sys_writev
init: starting dash
proc[1] calls sys_rt_sigprocmask
pf_handler: proc[1], head=0xffff00003afe0e30, ec=37, addr=0x4021a0, flt=3
proc[1] calls sys_rt_sigprocmask
pf_handler: proc[1], head=0xffff00003afe0e30, ec=37, addr=0x402198, flt=3
proc[1] calls sys_clone
map: addr=0x400000, pa=0x3afde000
...
map: addr=0x404000, pa=0x3af90000
map: addr=0xfffffffe0000, pa=0x3afba000
...
map: addr=0xfffffffff000, pa=0x3af9b000
fork: np[6]: nregions=3, head=0xffff00003afe0db0      # <= headあり
proc[6] calls sys_gettid
proc[6] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_wait4
proc[6] calls sys_rt_sigprocmask
proc[6] calls sys_execve
sys_execve: p->head=0xffff00003afe0db0
pf_handler: proc[6], head=0x0, ec=37, addr=0xfffffffff000, flt=3  # <= headなし
pf_handler: fault_addr: 0xfffffffff000, flt: 3

====== TRAP FRAME DUMP ======
sp:	0xfffffffffe20
tpidr:	0x404630
spsr:	0x60000000
esr:	0x56000000
elr:	0x40074c
x0:	0x402168
x1:	0xfffffffffe78
x2:	0xfffffffffe70
x3:	0x400740
x4:	0xfffffffffdff
x29:	0xfffffffffe30
x30:	0x400234
====== DUMP END ======
```

## create_elf_tables()実行後のsp

- 正しい

```
(gdb) n
160	    return sp;
(gdb) p/x sp
$1 = 0xfffffffffec0
(gdb) x/-40gx 0x1000000000000
0xfffffffffec0:	0x0000000000000001	0x0000ffffffffffe4  // argc, argv[0]
0xfffffffffed0:	0x0000000000000000	0x0000000000000000  // arg[], envp[]
0xfffffffffee0:	0x0000000000000010	0x0000000000000377  // AT_HWCAP
0xfffffffffef0:	0x0000000000000006	0x0000000000001000  // AT_PAGESZ
0xffffffffff00:	0x0000000000000011	0x0000000000000064  // AT_CLKTCK
0xffffffffff10:	0x0000000000000003	0x0000000000400040  // AT_PHDR
0xffffffffff20:	0x0000000000000004	0x0000000000000038  // AT_PHENT
0xffffffffff30:	0x0000000000000005	0x0000000000000004  // AT_PHNUM
0xffffffffff40:	0x0000000000000007	0x0000000000000000  // AT_BASE
0xffffffffff50:	0x0000000000000008	0x0000000000000000  // AT_FLAGS
0xffffffffff60:	0x0000000000000009	0x0000000000400268  // AT_ENTRY
0xffffffffff70:	0x000000000000000b	0x0000000000000000  // AT_UID
0xffffffffff80:	0x000000000000000c	0x0000000000000000  // AT_EUID
0xffffffffff90:	0x000000000000000d	0x0000000000000000  // AT_GDI
0xffffffffffa0:	0x000000000000000e	0x0000000000000000  // AT_EGID
0xffffffffffb0:	0x000000000000000f	0x0000ffffffffffdc  // AT_PLATFORM
0xffffffffffc0:	0x0000000000000000	0x0000000000000000  // AT_NULL
0xffffffffffd0:	0x0000000000000000	0x6372616100000000  // fdc -> aarch64     : u_platform
0xffffffffffe0:	0x6e69622f00343668	0x622f0074696e692f  // fe4 -> /bin/init   : sthb[0]
0xfffffffffff0:	0x0074696e692f6e69	0x0000000000000000  // fee -> /bin/init   : filename

(gdb) x/2s 0x0000ffffffffffe4
0xffffffffffe4:	"/bin/init"
0xffffffffffee:	"/bin/init"
(gdb) x/s 0x0000ffffffffffdc
0xffffffffffdc:	"aarch64"

(gdb) x/48bc 0xffffffffffd0
0xffffffffffd0:	........
0xffffffffffd8:	....aarc
0xffffffffffe0:	h64./bin
0xffffffffffe8:	/init./b
0xfffffffffff0:	in/init.
0xfffffffffff8:	........
```

## 1/26 開始時点の状況

- dash実行時にp->headがない状態でアクセスしてエラー

```
proc[1] calls sys_execve
sys_execve: path=/bin/init, uargv=0x20, uenvp=0x0
- argv[0] = '/bin/init' (0xffff00003afe0ff0)
- argv[1] = '(null)' (0x0)
- envp[0] = '(null)'
- argc=1, envc=0
execve: proc[1] '/bin/init' start, uid=0, gid=0
binfmt_elf1: p[1], head=0x0
binfmt_elf2: p[1], head=0x0
binfmt_elf3: p[1], head=0xffff00003afe0e70
== UVM_SWITCH ==
execve4: p[1], head=0xffff00003afe0e30
proc[1] calls sys_gettid
proc[1] calls sys_openat
proc[1] calls sys_dup
proc[1] calls sys_dup
proc[1] calls sys_ioctl
proc[1] calls sys_writev
init: starting dash
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_clone
fork: np[6]: nregions=3, head=0xffff00003afe0db0
proc[6] calls sys_gettid
proc[6] calls sys_rt_sigprocmask
proc[6] calls sys_rt_sigprocmask
proc[6] calls sys_execve
sys_execve: path=/bin/dash, uargv=0xfffffffffe78, uenvp=0xfffffffffe70
- argv[0] = 'dash' (0xffff00003afe0d10)
- argv[1] = '(null)' (0x0)
- envp[0] = '(null)'
- argc=1, envc=0
execve: proc[6] '/bin/dash' start, uid=0, gid=0
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_wait4
binfmt_elf1: p[6], head=0x0
binfmt_elf2: p[6], head=0x0
pf_handler: fault_addr: 0xfffffffff000, flt: 3

====== TRAP FRAME DUMP ======
sp:	0xfffffffffe20
tpidr:	0x404630
spsr:	0x60000000
esr:	0x56000000
elr:	0x40074c
x0:	0x402168
x1:	0xfffffffffe78
x2:	0xfffffffffe70
x3:	0x400740
x4:	0xfffffffffdff
x29:	0xfffffffffe30
x30:	0x400234
====== DUMP END ======
```

- fork時に新プロセス用にpgdirを作成し、これをexecveでも使用するようにした

```
execve: proc[1] '/bin/init' start, argc=1, envc=0
execve1: p[1], head=0x0
execve2: p[1], head=0x0
execve3: p[1], head=0x0
binfmt_elf1: p[1], head=0x0
binfmt_elf2: p[1], head=0x0
binfmt_elf3: p[1], head=0xffff00003afe0e70
== UVM_SWITCH ==
execve4: p[1], head=0xffff00003afe0e30
init: starting dash
fork: copy_mmap_listfork: np[6]: nregions=3, head=0xffff00003afe0db0
execve: proc[6] '/bin/dash' start, argc=1, envc=0
execve1: p[6], head=0xffff00003afe0db0
execve2: p[6], head=0xffff00003afe0db0
execve3: p[6], head=0xffff00003afe0db0
binfmt_elf1: p[6], head=0xffff00003afe0db0
binfmt_elf2: p[6], head=0x0
binfmt_elf3: p[6], head=0xffff00003afe0db0
== UVM_SWITCH ==
execve4: p[6], head=0xffff00003afe0d70
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=3
 - region[1]: addr=0x400000, length=0x29180, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x439000, length=0x2230, prot=0x3, flags=0x1812, fd=6, offset=0x29000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x132, fd=-1, offset=0x0
pf_handler: fault_addr: 0x43c000, flt: 2

====== TRAP FRAME DUMP ======
sp:	0xfffffffffd30
tpidr:	0x404630
spsr:	0x80000000
esr:	0x9200004b
elr:	0x4135d0
x0:	0x0
x1:	0x43cb80
x2:	0xffffffffffffffe0
x3:	0xfffffffffd80
x4:	0xfffffffffeb0
x29:	0x0
x30:	0x4135a4
====== DUMP END ======
```

- bss用のregionが設定されていない

```
data_end = 0x0000000000439f90 + 0x00000000000012a0 = 0x43b230     // <= region[2]
bss_end  = 0x0000000000439f90 + 0x0000000000002c68 = 0x43cbf8     // <= 設定なし
```

```
$ aarch64-linux-gnu-readelf -l dash/src/dash

Elf file type is EXEC (Executable file)
Entry point 0x40032c
There are 4 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x0000000000029180 0x0000000000029180  R E    0x10000
  LOAD           0x0000000000029f90 0x0000000000439f90 0x0000000000439f90
                 0x00000000000012a0 0x0000000000002c68  RW     0x10000
```

- set_brkのfind_map_regionの返す値がおかしい

```
(gdb) p/x start
$7 = 0x43c000
(gdb) p/x end
$8 = 0x43d000
(gdb) n
62	    region = find_mmap_region((void *)start);
(gdb)
65	    mapped_addr = (uint64_t)mmap((void *)ELF_PAGEALIGN((uint64_t)region->addr + end), end - start, prot, flags, -1, 0);   // このmmapを実行でエラー
(gdb) p/x region->addr
Cannot access memory at address 0xffffffffffffffff
```

- check_mmap_regionの条件に引っかかる。条件式が間違っている

```
(gdb)
140	        if (cursor->next && cursor->addr <= addr && addr + length > cursor->next->addr)
(gdb)
141	            return 0;
(gdb) p/cursor->next
A syntax error in expression, near `->next'.
(gdb) p/x cursor->next
$5 = 0xffff00003afe0d30
(gdb) p/x cursor->addr
$6 = 0x400000
(gdb) p/x addr
$7 = 0x43c000
(gdb) p/x addr + length
$8 = 0x43d000
(gdb) p/x cursor->next->addr
$9 = 0x439000
```

- 条件を修正

```
init: starting dash
fork: copy_mmap_listfork: np[6]: nregions=3, head=0xffff00003afe0db0
execve: proc[6] '/bin/dash' start, argc=1, envc=0
entry=0x40032c, bss=0x43b230, brk=0x43cbf8
start_code=0x400000, end_code=0x429180
start_data=0x439f90, end_data=0x43b230
bprm->p=0xfffffffffed0
p[6]: sz=0x43cbf8, kstack=0xffff00003affa000

====== TRAP FRAME DUMP ======
sp:	0xfffffffffed0
tpidr:	0x404630
spsr:	0x60000000
esr:	0x56000000
elr:	0x40032c
x0:	0x1
x1:	0xfffffffffed8
x2:	0xfffffffffe70
x3:	0x400740
x4:	0xfffffffffdff
x29:	0xfffffffffe30
x30:	0x400234
====== DUMP END ======

pf_handler: proc[6], head=0xffff00003afe0d70, ec=36, addr=0xfffffffdfff0, flt=2
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=4
 - region[1]: addr=0x400000, length=0x29180, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x439000, length=0x2230, prot=0x3, flags=0x1812, fd=6, offset=0x29000
 - region[3]: addr=0x43c000, length=0x1000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x132, fd=-1, offset=0x0
pf_handler: fault_addr: 0xfffffffdf000, flt: 2

====== TRAP FRAME DUMP ======
sp:	0xfffffffe0150
tpidr:	0x43bde8
spsr:	0x20000000
esr:	0x9200004b
elr:	0x4164b4
x0:	0x0
x1:	0x0
x2:	0x422b68
x3:	0xfffffffe0180
x4:	0xfffffffe03c0
x29:	0xfffffffe0150
x30:	0x40fa64
====== DUMP END ======
```

- stackは0xfffffffe0000からTOPまで0x20000バイト確保
- sp = 0xfffffffe0150 - 0x160 (352) = 0xfffffffdfff0
- stack overflow している
- dash実行時のスタックポイントは sp = bprm->p = 0xfffffffffed0 を設定している
- エラー時のspは0xfffffffe0150で0x1fd80 (130,432) バイト使用されている
- MAX_ARG_PAGESを64, 96に変更するとspがそれぞれ0xfffffffc0070, 0xfffffffa00f0となり
  stackを確保したアドレス近くにspが設定されていると思われる。なお、128にすると別のエラーとなる。

```
00000000004164b4 <vsnprintf>:
  4164b4:       a9aa53f3        stp     x19, x20, [sp, #-352]!
  4164b8:       aa0103f4        mov     x20, x1
```

- MAX_ARG_PAGES=128の場合

```
execve: proc[1] '/bin/init' start, argc=1, envc=0
entry=0x400268, bss=0x404108, brk=0x404790
start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bprm->p=0xfffffffffec0
p[1]: sz=0x404790, kstack=0xffff00003affe000
kfree: invalid v=ffff4240017e8000               // V2P(v) >= PHYSTOP
```

## uvm_switch()の後にtrapfram_dump

```
bprm->p=0xfffffffffed0

====== TRAP FRAME DUMP ======
sp:	0xfffffffffed0
tpidr:	0x404630
spsr:	0x60000000
esr:	0x56000000
elr:	0x40032c
x0:	0x1
x1:	0xfffffffffed8
x2:	0xfffffffffe70
x3:	0x400740
x4:	0xfffffffffdff
x29:	0xfffffffffe30
x30:	0x400234
====== DUMP END ======
```

- branch no_mmap_exec

```
init log[1]: ok

====== TRAP FRAME DUMP ======
sp:	0x406fb8
tpidr:	0x0
spsr:	0x0
esr:	0x56000000
elr:	0x400268
x0:	0x1
x1:	0x406fc0
x2:	0x0
x3:	0x0
x4:	0x0
x29:	0x0
x30:	0x0
====== DUMP END ======

init: starting dash

====== TRAP FRAME DUMP ======
sp:	0x43efb8
tpidr:	0x404630
spsr:	0x60000000
esr:	0x56000000
elr:	0x40032c
x0:	0x1
x1:	0x43efc0
x2:	0x406f60
x3:	0x400740
x4:	0x406eef
x29:	0x406f20
x30:	0x400234
====== DUMP END ======

# ls

====== TRAP FRAME DUMP ======
sp:	0x443fa0
tpidr:	0x43bde8
spsr:	0x80000000
esr:	0x56000000
elr:	0x401cec
x0:	0x1
x1:	0x443fa8
x2:	0x43c690
x3:	0xfefefefefeff726b
x4:	0x423bf1
x29:	0x43eb50
x30:	0x4039bc
====== DUMP END ======

bin  dev  etc  home  lib  usr
```

## raspi linuxでの実行

```
$ LD_SHOW_AUXV=1 dash (dynamic)
AT_SYSINFO_EHDR:      0xffff898dd000
AT_??? (0x33): 0x1270
AT_HWCAP:             887
AT_PAGESZ:            4096
AT_CLKTCK:            100
AT_PHDR:              0xaaaabb3f8040
AT_PHENT:             56
AT_PHNUM:             9
AT_BASE:              0xffff898ad000
AT_FLAGS:             0x0
AT_ENTRY:             0xaaaabb3fd2d8
AT_UID:               1002
AT_EUID:              1002
AT_GID:               1002
AT_EGID:              1002
AT_SECURE:            0
AT_RANDOM:            0xffffefd8cfc8
AT_HWCAP2:            0x0
AT_EXECFN:            /usr/bin/dash
AT_PLATFORM:          aarch64
$ ps
    PID TTY          TIME CMD
   1998 pts/0    00:00:00 bash
   2116 pts/0    00:00:00 dash
   2117 pts/0    00:00:00 ps
$ cat /proc/2116/maps
00400000-0042a000 r-xp 00000000 b3:02 636547                             /home/dspace/work/c/dash
00439000-0043c000 rw-p 00029000 b3:02 636547                             /home/dspace/work/c/dash
0043c000-0043d000 rw-p 00000000 00:00 0
3b49e000-3b49f000 rw-p 00000000 00:00 0                                  [heap]
ffffb1d66000-ffffb1d67000 r--p 00000000 00:00 0                          [vvar]
ffffb1d67000-ffffb1d68000 r-xp 00000000 00:00 0                          [vdso]
ffffd9d3f000-ffffd9d60000 rw-p 00000000 00:00 0                          [stack]
$ sudo xxd /proc/2116/auxv
00000000: 2100 0000 0000 0000 0070 d6b1 ffff 0000  !........p......     // AT_SYSINFO_EHDR  = 0xffffb1d67000
00000010: 3300 0000 0000 0000 7012 0000 0000 0000  3.......p.......     // AT_MINSIGSTKSZ   = 0x1207  = 4615
00000020: 1000 0000 0000 0000 8708 0000 0000 0000  ................     // AT_HWCAP         = 0x0887
00000030: 0600 0000 0000 0000 0010 0000 0000 0000  ................     // AT_PAGESZ        = 0x1000  = 4096
00000040: 1100 0000 0000 0000 6400 0000 0000 0000  ........d.......     // AT_CLKTCK        = 0x64    = 100
00000050: 0300 0000 0000 0000 4000 4000 0000 0000  ........@.@.....     // AT_PHDR          = 0x400040
00000060: 0400 0000 0000 0000 3800 0000 0000 0000  ........8.......     // AT_PHENT         = 0x38    = 56
00000070: 0500 0000 0000 0000 0400 0000 0000 0000  ................     // AT_PHNUM         = 0x4
00000080: 0700 0000 0000 0000 0000 0000 0000 0000  ................     // AT_BASE          = 0x0
00000090: 0800 0000 0000 0000 0000 0000 0000 0000  ................     // AT_FLAGS         = 0x0
000000a0: 0900 0000 0000 0000 2c03 4000 0000 0000  ........,.@.....     // AT_ENTRY         = 0x40032c
000000b0: 0b00 0000 0000 0000 ea03 0000 0000 0000  ................     // AT_UID           = 0x03ea  = 1002
000000c0: 0c00 0000 0000 0000 ea03 0000 0000 0000  ................     // AT_EUID          = 0x03ea  = 1002
000000d0: 0d00 0000 0000 0000 ea03 0000 0000 0000  ................     // AT_GID           = 0x03ea  = 1002
000000e0: 0e00 0000 0000 0000 ea03 0000 0000 0000  ................     // AT_EGID          = 0x03ea  = 1002
000000f0: 1700 0000 0000 0000 0000 0000 0000 0000  ................     // AT_SECURE        = 0x
00000100: 1900 0000 0000 0000 28f5 d5d9 ffff 0000  ........(.......     // AT_RANDOM        = 0xffffd9d5f528
00000110: 1a00 0000 0000 0000 0000 0000 0000 0000  ................     // AT_HWCAP2        = 0x0
00000120: 1f00 0000 0000 0000 f1ff d5d9 ffff 0000  ................     // AT_EXECFN        = 0xffffd9d5fff1
00000130: 0f00 0000 0000 0000 38f5 d5d9 ffff 0000  ........8.......     // AT_PLATFORM      = 0xffffd9d5f538
00000140: 0000 0000 0000 0000 0000 0000 0000 0000  ................     // AT_NULL          = 0x
/proc/2116/task/2116$ sudo cat stack
[<0>] __switch_to+0x144/0x1a0
[<0>] wait_woken+0x5c/0xb8
[<0>] n_tty_read+0x698/0xaa0
[<0>] tty_read+0x94/0x120
[<0>] __vfs_read+0x4c/0x90
[<0>] vfs_read+0xd4/0x1a0
[<0>] ksys_read+0x7c/0x108
[<0>] __arm64_sys_read+0x28/0x38
[<0>] el0_svc_common.constprop.0+0x84/0x230
[<0>] el0_svc_handler+0x38/0xa0
[<0>] el0_svc+0x10/0x2d4
```

## dash (static)

```
$ ps
    PID TTY          TIME CMD
   2000 pts/0    00:00:00 bash
   2224 pts/0    00:00:00 strace
   2227 pts/0    00:00:00 dash
   2228 pts/0    00:00:00 ps
$ cd /proc/2227
$ cat maps
00400000-0042a000 r-xp 00000000 b3:02 636547                             /home/dspace/work/c/dash
00439000-0043c000 rw-p 00029000 b3:02 636547                             /home/dspace/work/c/dash
0043c000-0043d000 rw-p 00000000 00:00 0
2464d000-2464e000 rw-p 00000000 00:00 0                                  [heap]
ffffaed72000-ffffaed73000 r--p 00000000 00:00 0                          [vvar]
ffffaed73000-ffffaed74000 r-xp 00000000 00:00 0                          [vdso]
ffffd925b000-ffffd927c000 rw-p 00000000 00:00 0                          [stack]
$ xxd auxv
00000000: 2100 0000 0000 0000 0030 d7ae ffff 0000  !........0......
00000010: 3300 0000 0000 0000 7012 0000 0000 0000  3.......p.......
00000020: 1000 0000 0000 0000 8708 0000 0000 0000  ................
00000030: 0600 0000 0000 0000 0010 0000 0000 0000  ................
00000040: 1100 0000 0000 0000 6400 0000 0000 0000  ........d.......
00000050: 0300 0000 0000 0000 4000 4000 0000 0000  ........@.@.....
00000060: 0400 0000 0000 0000 3800 0000 0000 0000  ........8.......
00000070: 0500 0000 0000 0000 0400 0000 0000 0000  ................
00000080: 0700 0000 0000 0000 0000 0000 0000 0000  ................
00000090: 0800 0000 0000 0000 0000 0000 0000 0000  ................
000000a0: 0900 0000 0000 0000 2c03 4000 0000 0000  ........,.@.....
000000b0: 0b00 0000 0000 0000 ea03 0000 0000 0000  ................
000000c0: 0c00 0000 0000 0000 ea03 0000 0000 0000  ................
000000d0: 0d00 0000 0000 0000 ea03 0000 0000 0000  ................
000000e0: 0e00 0000 0000 0000 ea03 0000 0000 0000  ................
000000f0: 1700 0000 0000 0000 0000 0000 0000 0000  ................
00000100: 1900 0000 0000 0000 b8b5 27d9 ffff 0000  ..........'.....
00000110: 1a00 0000 0000 0000 0000 0000 0000 0000  ................
00000120: 1f00 0000 0000 0000 f1bf 27d9 ffff 0000  ..........'.....
00000130: 0f00 0000 0000 0000 c8b5 27d9 ffff 0000  ..........'.....
00000140: 0000 0000 0000 0000 0000 0000 0000 0000  ................
$ cat stack
cat: stack: Permission denied
dspace@ubuntu:/proc/2227$ sudo cat stack
[<0>] __switch_to+0x144/0x1a0
[<0>] wait_woken+0x5c/0xb8
[<0>] n_tty_read+0x698/0xaa0
[<0>] tty_read+0x94/0x120
[<0>] __vfs_read+0x4c/0x90
[<0>] vfs_read+0xd4/0x1a0
[<0>] ksys_read+0x7c/0x108
[<0>] __arm64_sys_read+0x28/0x38
[<0>] el0_svc_common.constprop.0+0x84/0x230
[<0>] el0_svc_handler+0x38/0xa0
[<0>] el0_svc+0x10/0x2d4
$ cat stat
2227 (dash) S 2224 2227 2000 34816 2227 4194304 42 165 0 0 0 0 1 2 20 0 1 0 55406 335872 2 18446744073709551615 4194304 4362624 281474325001088 0 0 0 0 2637828 65538 1 0 0 17 3 0 0 0 0 0 4431760 4436528 610586624 281474325002088 281474325002095 281474325002095 281474325004273 0
$ cat statm
82 2 0 42 0 38 0
$ cat status
Name:	dash
Umask:	0002
State:	S (sleeping)
Tgid:	2227
Ngid:	0
Pid:	2227
PPid:	2224
TracerPid:	2224
Uid:	1002	1002	1002	1002
Gid:	1002	1002	1002	1002
FDSize:	64
Groups:	27 1002
NStgid:	2227
NSpid:	2227
NSpgid:	2227
NSsid:	2000
VmPeak:	     328 kB
VmSize:	     328 kB
VmLck:	       0 kB
VmPin:	       0 kB
VmHWM:	       8 kB
VmRSS:	       8 kB
RssAnon:	       8 kB
RssFile:	       0 kB
RssShmem:	       0 kB
VmData:	      20 kB
VmStk:	     132 kB
VmExe:	     168 kB
VmLib:	       4 kB
VmPTE:	      28 kB
VmSwap:	       0 kB
CoreDumping:	0
THP_enabled:	0
Threads:	1
SigQ:	0/3244
SigPnd:	0000000000000000
ShdPnd:	0000000000000000
SigBlk:	0000000000000000
SigIgn:	0000000000284004
SigCgt:	0000000000010002
CapInh:	0000000000000000
CapPrm:	0000000000000000
CapEff:	0000000000000000
CapBnd:	0000003fffffffff
CapAmb:	0000000000000000
NoNewPrivs:	0
Seccomp:	0
Speculation_Store_Bypass:	unknown
Cpus_allowed:	f
Cpus_allowed_list:	0-3
Mems_allowed:	1
Mems_allowed_list:	0
voluntary_ctxt_switches:	106
nonvoluntary_ctxt_switches:	1
$ ls -l fd
total 0
lrwx------ 1 dspace dspace 64 Jan 27 14:30 0 -> /dev/pts/0
lrwx------ 1 dspace dspace 64 Jan 27 14:30 1 -> /dev/pts/0
lrwx------ 1 dspace dspace 64 Jan 27 14:30 10 -> /dev/tty
lrwx------ 1 dspace dspace 64 Jan 27 14:30 2 -> /dev/pts/0
$ ls -l fdinfo
total 0
-r-------- 1 dspace dspace 0 Jan 27 14:31 0
-r-------- 1 dspace dspace 0 Jan 27 14:31 1
-r-------- 1 dspace dspace 0 Jan 27 14:31 10
-r-------- 1 dspace dspace 0 Jan 27 14:31 2
$ cat fdinfo/10
pos:	0
flags:	02400002
mnt_id:	25
```

## 1/28 開始地点

```
execve: proc[1] '/bin/init' start, argc=1, envc=0
entry=0x400268, bss=0x404108, brk=0x404790
start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bprm->p=0xfffffffffea0
p[1]: sz=0x404790, kstack=0xffff00003affe000
 PRINT mmap_region: pid=1, head=0xffff00003afe0e30, nregions=3
 - region[1]: addr=0x400000, length=0x2274, prot=0x5, flags=0x1812, fd=3, offset=0x0
 - region[2]: addr=0x403000, length=0x1108, prot=0x3, flags=0x1812, fd=4, offset=0x2000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
init: starting dash
fork: copy_mmap_listfork: np[6]: nregions=3, head=0xffff00003afe0db0
execve: proc[6] '/bin/dash' start, argc=1, envc=0
entry=0x40032c, bss=0x43b230, brk=0x43cbf8
start_code=0x400000, end_code=0x429180
start_data=0x439f90, end_data=0x43b230
bprm->p=0xfffffffffeb0
p[6]: sz=0x43cbf8, kstack=0xffff00003affa000
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=4
 - region[1]: addr=0x400000, length=0x29180, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x439000, length=0x2230, prot=0x3, flags=0x1812, fd=6, offset=0x29000
 - region[3]: addr=0x43c000, length=0x1000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
pf_handler: proc[6], head=0xffff00003afe0d70, nregions=4, ec=36, addr=0xfffffffdfff0, flt=2
pf_handler: fault_addr: 0xfffffffdf000, flt: 2

====== TRAP FRAME DUMP ======
sp:	0xfffffffe0020
tpidr:	0x43bde8
spsr:	0x80000000
esr:	0x9200004b
elr:	0x41efc8
x0:	0x0
x1:	0x1
x2:	0xfffffffe0468
x3:	0xfffffffe0468
x4:	0xfffffffe0051
x29:	0xfffffffe0550
x30:	0x41c538
====== DUMP END ======
```

## mmap.cでlengthを切り上げ

```
execve: proc[1] '/bin/init' start, argc=1, envc=0
entry=0x400268, bss=0x404108, brk=0x404790
start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bprm->p=0xfffffffffea0
p[1]: sz=0x404790, kstack=0xffff00003affe000
 PRINT mmap_region: pid=1, head=0xffff00003afe0e30, nregions=3
 - region[1]: addr=0x400000, length=0x3000, prot=0x5, flags=0x1812, fd=3, offset=0x0
 - region[2]: addr=0x403000, length=0x2000, prot=0x3, flags=0x1812, fd=4, offset=0x2000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
pf_handler: map_region failed
pf_handler: fault_addr: 0x4000000000000, flt: 1

====== TRAP FRAME DUMP ======
sp:	0xfffffffffca0
tpidr:	0x0
spsr:	0x20000000
esr:	0x92000044
elr:	0x401158
x0:	0x0
x1:	0x404720
x2:	0xffffffffffffff44
x3:	0x22
x4:	0xffffffffffffffff
x29:	0x0
x30:	0x4012b4
====== DUMP END ======
```

## mmap()におけるlengthはmappedアドレスを決める際はそのまま使い、最後にページ境界に切り上げて保存する。

```
execve: proc[1] '/bin/init' start, argc=1, envc=0
pf_handler: fault_addr=0xfffffffff000, flt=3
pf_handler: fault_addr=0x0, flt=1
entry=0x400268, bss=0x404108, brk=0x404790
start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bprm->p=0xfffffffffea0
p[1]: sz=0x404790, kstack=0xffff00003affe000
 PRINT mmap_region: pid=1, head=0xffff00003afe0e30, nregions=3
 - region[1]: addr=0x400000, length=0x3000, prot=0x5, flags=0x1812, fd=3, offset=0x0
 - region[2]: addr=0x403000, length=0x2000, prot=0x3, flags=0x1812, fd=4, offset=0x2000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
pf_handler: fault_addr=0x404560, flt=3
init: starting dash
pf_handler: fault_addr=0x4021a0, flt=3
pf_handler: fault_addr=0x402198, flt=3
fork: copy_mmap_listfork: np[6]: nregions=3, head=0xffff00003afe0db0
pf_handler: fault_addr=0x404588, flt=3
pf_handler: fault_addr=0xfffffffffca0, flt=3
execve: proc[6] '/bin/dash' start, argc=1, envc=0
pf_handler: fault_addr=0xfffffffff000, flt=3
pf_handler: fault_addr=0x0, flt=1
set_brk: addr=0x43c000, length=0x1000, mapped=0x43c000
entry=0x40032c, bss=0x43b230, brk=0x43cbf8
start_code=0x400000, end_code=0x429180
start_data=0x439f90, end_data=0x43b230
bprm->p=0xfffffffffeb0
p[6]: sz=0x43cbf8, kstack=0xffff00003affa000
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=4
 - region[1]: addr=0x400000, length=0x2a000, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x439000, length=0x3000, prot=0x3, flags=0x1812, fd=6, offset=0x29000
 - region[3]: addr=0x43c000, length=0x1000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
pf_handler: fault_addr=0x43b5b8, flt=3
pf_handler: fault_addr=0x43cb88, flt=3
pf_handler: fault_addr=0xfffffffdfff0, flt=1
pf_handler: fault_addr=0xfffffffdfff0, flt=2
pf_handler: proc[6], head=0xffff00003afe0d70, nregions=4, ec=36, addr=0xfffffffdfff0, flt=2
pf_handler: fault_addr: 0xfffffffdf000, flt: 2

====== TRAP FRAME DUMP ======
sp:	0xfffffffe0020
tpidr:	0x43bde8
spsr:	0x80000000
esr:	0x9200004b
elr:	0x41efc8
x0:	0x0
x1:	0x1
x2:	0xfffffffe0468
x3:	0xfffffffe0468
x4:	0xfffffffe0051
x29:	0xfffffffe0550
x30:	0x41c538
====== DUMP END ======
```

## muslのmmalocで問題がある模様

- 2回のsys_brkの呼び出しの引数がいずれも0
- 次はsys_mmapを呼び出している（この場合もlengthが0）
- これは`libc/src/malloc/oldmalloc/malloc.c`の`__expand_heap(size_t *pn)`関数が
  呼び出しているものと思われる。

```
proc[6] calls sys_getppid
proc[6] calls sys_brk
sys_brk: n=0x0, sz=0x43cbf8
- param = 0 return: 0x43cbf8
proc[6] calls sys_brk
sys_brk: n=0x0, sz=0x43cbf8
- param = 0 and return: 0x43cbf8
proc[6] calls sys_mmap
sys_mmap: addr=0x0, length=0x0, prot=0x3, flags=0x22, fd=-1, offset=0x0
map_region: remap *pte=0x2000003af4c747, va=0x0
```

### mmapを使わない実装 (branch: dynamic) では

- 1回目のsys_brkで現在のbrkを取得し
- 2回めのsys_brkで新しいbrk値を指定している

```
proc[6] calls sys_getppid
proc[6] calls sys_brk
sys_brk: n=0x0, sz=0x436000
n=0 return 0x436000
proc[6] calls sys_brk
sys_brk: n=0x437000, sz=0x436000
- return 0x437000
proc[6] calls sys_getcwd
```

- roundup(elf_brk, PGSIZE)をp->szに設定するようにした
- sys_mmapでlength=0の場合、length=PGSIZEとした場合は次のような結果になる
- fault_addr=0xfffffffffffffff0は永遠に繰り返される（このアドレスはnull地が設定されている）
- このアドレスにアクセスすること自体がおかしい

```
execve: proc[6] '/bin/dash' start, argc=1, envc=0
entry=0x40032c, bss=0x43b230, brk=0x43cbf8
start_code=0x400000, end_code=0x429180
start_data=0x439f90, end_data=0x43b230
bprm->p=0xfffffffffeb0
p[6]: sz=0x43d000, kstack=0xffff00003affa000
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=4
 - region[1]: addr=0x400000, length=0x2a000, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x439000, length=0x3000, prot=0x3, flags=0x1812, fd=6, offset=0x29000
 - region[3]: addr=0x43c000, length=0x1000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
proc[6] calls sys_gettid
proc[6] calls sys_getpid
proc[6] calls sys_rt_sigprocmask
proc[6] calls sys_rt_sigaction
proc[6] calls sys_geteuid
proc[6] calls sys_getppid
proc[6] calls sys_brk
sys_brk: n=0x0, sz=0x43d000
- n=0 return 0x43d000
proc[6] calls sys_brk
sys_brk: n=0x0, sz=0x43d000
- n=0 return 0x43d000
proc[6] calls sys_mmap
sys_mmap: addr=0x0, length=0x0, prot=0x3, flags=0x22, fd=-1, offset=0x0
- addr=0x400000, node->addr=0x400000, node->next->addr=0x439000
- addr=0x42a000, node->addr=0x439000, node->next->addr=0x43c000
- return 0x42a000
Trap fault_addr=0xfffffffffffffff0 5 times
pf_handler: fault_addr: 0xfffffffffffffff0, flt: 1

====== TRAP FRAME DUMP ======
sp:	0xfffffffffc00
tpidr:	0x43bde8
spsr:	0x80000000
esr:	0x92000044
elr:	0x41482c
x0:	0x0
x1:	0x43b000
x2:	0x1
x3:	0x22
x4:	0xffffffffffffffff
x29:	0xfffffffffca0
x30:	0x41488c
====== DUMP END ======
```

- sys_mmapでlength=0の場合、エラーを返すと以下が120回以上繰り返された挙げ句、

```
proc[6] calls sys_brk
sys_brk: n=0x0, sz=0x43d000
- n=0 return 0x43d000
proc[6] calls sys_brk
sys_brk: n=0x0, sz=0x43d000
- n=0 return 0x43d000
proc[6] calls sys_mmap
sys_mmap: addr=0x0, length=0x0, prot=0x3, flags=0x22, fd=-1, offset=0x0
```

- なぜかmmallocは諦められ、いつものエラーとなった。

```
pf_handler: fault_addr: 0xfffffffdf000, flt: 2

====== TRAP FRAME DUMP ======
sp:	0xfffffffe0020
tpidr:	0x43bde8
spsr:	0x80000000
esr:	0x9200004b
elr:	0x41efc8
x0:	0x0
x1:	0x1
x2:	0xfffffffe0468
x3:	0xfffffffe0468
x4:	0xfffffffe0051
x29:	0xfffffffe0550
x30:	0x41c538
====== DUMP END ======

```

## `growproc()`と`sys_brk()`をmmapで書き換え

- 結果は変わらず

```
proc[1] calls sys_execve
execve: proc[1] '/bin/init' start, argc=1, envc=0
entry=0x400268, bss=0x404108, brk=0x404790
start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bprm->p=0xfffffffffea0
p[1]: sz=0x405000, kstack=0xffff00003affe000
 PRINT mmap_region: pid=1, head=0xffff00003afe0e30, nregions=3
 - region[1]: addr=0x400000, length=0x3000, prot=0x5, flags=0x1812, fd=3, offset=0x0
 - region[2]: addr=0x403000, length=0x2000, prot=0x3, flags=0x1812, fd=4, offset=0x2000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
proc[1] calls sys_gettid
proc[1] calls sys_openat
proc[1] calls sys_dup
proc[1] calls sys_dup
proc[1] calls sys_ioctl
proc[1] calls sys_writev
init: starting dash
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_clone
fork: copy_mmap_listfork: np[6]: nregions=3, head=0xffff00003afe0db0
proc[6] calls sys_gettid
proc[6] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_wait4
proc[6] calls sys_rt_sigprocmask
proc[6] calls sys_execve
execve: proc[6] '/bin/dash' start, argc=1, envc=0
entry=0x40032c, bss=0x43b230, brk=0x43cbf8
start_code=0x400000, end_code=0x429180
start_data=0x439f90, end_data=0x43b230
bprm->p=0xfffffffffeb0
p[6]: sz=0x43d000, kstack=0xffff00003affa000
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=4
 - region[1]: addr=0x400000, length=0x2a000, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x439000, length=0x3000, prot=0x3, flags=0x1812, fd=6, offset=0x29000
 - region[3]: addr=0x43c000, length=0x1000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
proc[6] calls sys_gettid
proc[6] calls sys_getpid
proc[6] calls sys_rt_sigprocmask
proc[6] calls sys_rt_sigaction
proc[6] calls sys_geteuid
proc[6] calls sys_getppid
proc[6] calls sys_brk
sys_brk: n=0x0, sz=0x43d000
- invalid n 0 return 0x43d000
proc[6] calls sys_brk
sys_brk: n=0x0, sz=0x43d000
- invalid n 0 return 0x43d000
proc[6] calls sys_mmap
sys_mmap: addr=0x0, length=0x0, prot=0x3, flags=0x22, fd=-1, offset=0x0
- addr=0x400000, node->addr=0x400000, node->next->addr=0x439000
- addr=0x42a000, node->addr=0x439000, node->next->addr=0x43c000
- return 0x42a000
Trap fault_addr=0xfffffffffffffff0 5 times
pf_handler: fault_addr: 0xfffffffffffffff0, flt: 1

====== TRAP FRAME DUMP ======
sp:	0xfffffffffc00
tpidr:	0x43bde8
spsr:	0x80000000
esr:	0x92000044
elr:	0x41482c
x0:	0x0
x1:	0x43b000
x2:	0x1
x3:	0x22
x4:	0xffffffffffffffff
x29:	0xfffffffffca0
x30:	0x41488c
====== DUMP END ======
```

# muslのmallocをmallocngに変更

- 2回目のsys_brkで1回目に返したアドレスを指定するようになった
- しかし、2回目のsys_brkで返した値は使われず、sys_mmapがp->baseで呼ばれ、
  これはsys_brkでmmapしているのでremapのエラーとなった。

```
$ cd source/musl
$ make clean
$ CROSS_COMPILE=aarch64-linux-gnu- ./configure --target=aarch64                           // mallocng
$ CROSS_COMPILE=aarch64-linux-gnu- ./configure --target=aarch64 --with-malloc=oldmalloc   // oldmalloc
$ make
$ sudo make install

$ cd coreutils
$ make clean
$ CC=/usr/local/musl/bin/musl-gcc ./configure --host=aarch64-linux-gnu CFLAGS="-std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096"
$ make

$ cd dash
$ make clean
$ CC=/usr/local/musl/bin/musl-gcc ./configure --host=aarch64-linux-gnu --enable-static CFLAGS="-std=gnu99 -O3 -MMD -MP -static -fno-plt -fno-pic -fpie -z max-page-size=4096 -Isrc"
$ make

$ cd xv6
$ rm -rf obj
$ make
$ make qemu
proc[6] calls sys_brk
sys_brk: n=0x0, sz=0x435000
- invalid n 0 return 0x435000
proc[6] calls sys_brk
sys_brk: n=0x437000, sz=0x435000
- ok return 0x437000
proc[6] calls sys_mmap
sys_mmap: addr=0x435000, length=0x1000, prot=0x0, flags=0x32, fd=-1, offset=0x0   // PROT_NONE
map_region: remap *pte=0x6000003af137c7, va=0x435000
```

## sys_brkをp->szの修正だけでgrwoprocしないように修正

- 2page要求して、最初のページはアクション禁止とし、2page目から使用するようだ。

```
sys_brk: n=0x0, sz=0x435000
- invalid n 0 return 0x435000
proc[6] calls sys_brk
sys_brk: n=0x437000, sz=0x435000
- ok return 0x437000
proc[6] calls sys_mmap
sys_mmap: addr=0x435000, length=0x1000, prot=0x0, flags=0x32, fd=-1, offset=0x0   // PROT_NONE
- return 0x435000
pf_handler: fault_addr: 0x436000, flt: 2                                          // 2page目でfault
```

## sys_mprotectのstart=0, len=0

- muslでPAGESIZEが未定義でmprotectのaddrが正しく設定されていなかった

```
proc[6] calls sys_brk
proc[6] calls sys_brk
proc[6] calls sys_mmap
sys_mmap: addr=0x0, length=0x2000, prot=0x0, flags=0x22, fd=-1, offset=0x0
- addr=0x400000, node->addr=0x400000, node->next->addr=0x431000
- addr=0x430000, node->addr=0x431000, node->next->addr=0x434000
- addr=0x434000, node->addr=0x434000, node->next->addr=0xfffffffe0000
- addr=0x435000, node->addr=0xfffffffe0000, node->next->addr=0x0
- return 0x435000
proc[6] calls sys_mprotect
sys_mprotect: start=0x0, len=0, prot=3        // start, lenがいずれも0
```

## arch/aarch64/arch/aarch64/bits/alltypes.h.inにPGSIZEの定義を追加

```
p=0x436000, pagesize=4096, unprot=1
proc[6] calls sys_writev
__mprotect: addr=0x436000, len=1000, PAGE_SIZE=4096, start=436000, end=437000
proc[6] calls sys_mprotect
sys_mprotect: start=0x436000, len=1000, prot=3      // start, lenが設定されている
munmap: pid=6, addr=0x435000, length=0x2000
```

## munmapを修正

```
execve: proc[6] '/bin/dash' start, argc=1, envc=0
 PRINT mmap_region: pid=6, head=0xffff00003afe0db0, nregions=3
 - region[1]: addr=0x400000, length=0x3000, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x403000, length=0x2000, prot=0x3, flags=0x1812, fd=6, offset=0x2000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=3
 - region[1]: addr=0x403000, length=0x2000, prot=0x3, flags=0x1812, fd=6, offset=0x2000
 - region[2]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=2
 - region[1]: addr=0x403000, length=0x2000, prot=0x3, flags=0x1812, fd=6, offset=0x2000
 - region[2]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
 PRINT mmap_region: pid=6, head=0xffff00003afe0d30, nregions=2
 - region[1]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
 PRINT mmap_region: pid=6, head=0xffff00003afe0d30, nregions=1
 - region[1]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
 PRINT mmap_region: pid=6, head=0x0, nregions=1
entry=0x4004a0, bss=0x433320, brk=0x434e80
start_code=0x400000, end_code=0x4300e0
start_data=0x431940, end_data=0x433320
bprm->p=0xfffffffffeb0
p[6]: sz=0x435000, kstack=0xffff00003affa000
proc[6] calls sys_gettid
proc[6] calls sys_getpid
proc[6] calls sys_rt_sigprocmask
proc[6] calls sys_rt_sigaction
proc[6] calls sys_geteuid
proc[6] calls sys_getppid
proc[6] calls sys_brk
proc[6] calls sys_brk
proc[6] calls sys_mmap
sys_mmap: addr=0x0, length=0x2000, prot=0x0, flags=0x22, fd=-1, offset=0x0
- return 0x435000
proc[6] calls sys_ioctl
proc[6] calls sys_writev
p=0x436000, pagesize=4096, unprot=1
proc[6] calls sys_writev
__mprotect: addr=0x436000, len=1000, PAGE_SIZE=4096, start=436000, end=437000
proc[6] calls sys_mprotect
sys_mprotect: start=0x436000, len=1000, prot=3
munmap: pid=6, addr=0x435000, length=0x2000
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=5
 - region[1]: addr=0x400000, length=0x31000, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x1812, fd=6, offset=0x30000
 - region[3]: addr=0x434000, length=0x1000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[4]: addr=0x435000, length=0x2000, prot=0x0, flags=0x22, fd=-1, offset=0x0
 - region[5]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=5
 - region[1]: addr=0x400000, length=0x31000, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x1812, fd=6, offset=0x30000
 - region[3]: addr=0x434000, length=0x1000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=4
 - region[1]: addr=0x400000, length=0x31000, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x1812, fd=6, offset=0x30000
 - region[3]: addr=0x434000, length=0x1000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
proc[6] calls sys_writev
p=0x436000, pagesize=4096, unprot=1
proc[6] calls sys_writev
__mprotect: addr=0x436000, len=1000, PAGE_SIZE=4096, start=436000, end=437000
proc[6] calls sys_mprotect
sys_mprotect: start=0x436000, len=1000, prot=3
- ok
proc[6] calls sys_mmap
sys_mmap: addr=0x0, length=0x1000, prot=0x3, flags=0x22, fd=-1, offset=0x0
- return 0x437000
proc[6] calls sys_write
sh: 0:                           // ここでストール
```

## munmap, mprotectのバグを修正

```
proc[1] calls sys_execve
execve: proc[1] '/bin/init' start, argc=1, envc=0
- mmap: addr=0xfffffffe0000, length=131072, prot=0x7, flags=0x32, fd=-1, offset=0x0
- mmap: addr=0x400000, length=8820, prot=0x5, flags=0x1812, fd=0, offset=0x0
- mmap: addr=0x403000, length=4360, prot=0x3, flags=0x1812, fd=0, offset=0x2000
entry=0x400268, bss=0x404108, brk=0x404790
start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bprm->p=0xfffffffffea0
p[1]: sz=0x405000, kstack=0xffff00003affe000
proc[1] calls sys_gettid
proc[1] calls sys_openat
proc[1] calls sys_dup
proc[1] calls sys_dup
proc[1] calls sys_ioctl
proc[1] calls sys_writev
init: starting dash
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_clone
fork: copy_mmap_listfork: np[6]: nregions=3, head=0xffff00003afe0db0
proc[6] calls sys_gettid
proc[6] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_wait4
proc[6] calls sys_rt_sigprocmask
proc[6] calls sys_execve
execve: proc[6] '/bin/dash' start, argc=1, envc=0
- mmap: addr=0xfffffffe0000, length=131072, prot=0x7, flags=0x32, fd=-1, offset=0x0
- mmap: addr=0x400000, length=196440, prot=0x5, flags=0x1812, fd=7, offset=0x0
- mmap: addr=0x431000, length=8752, prot=0x3, flags=0x1812, fd=7, offset=0x30000
- mmap: addr=0x434000, length=4096, prot=0x3, flags=0x32, fd=-1, offset=0x0
entry=0x4004a0, bss=0x433230, brk=0x434988
start_code=0x400000, end_code=0x42ff58
start_data=0x431940, end_data=0x433230
bprm->p=0xfffffffffeb0
p[6]: sz=0x435000, kstack=0xffff00003affa000
proc[6] calls sys_gettid
proc[6] calls sys_getpid
proc[6] calls sys_rt_sigprocmask
proc[6] calls sys_rt_sigaction
proc[6] calls sys_geteuid
proc[6] calls sys_getppid
proc[6] calls sys_brk
proc[6] calls sys_brk
proc[6] calls sys_mmap
sys_mmap: addr=0x0, length=0x2000, prot=0x0, flags=0x22, fd=-1, offset=0x0
- mmap: addr=0x0, length=8192, prot=0x0, flags=0x22, fd=-1, offset=0x0
- return 0x435000
proc[6] calls sys_mprotect
sys_mprotect: start=0x436000, len=0x1000, prot=3
- munmap: addr=0x435000, length=0x2000
- mmap: addr=0x435000, length=4096, prot=0x0, flags=0x32, fd=-1, offset=0x0
- mmap: addr=0x436000, length=4096, prot=0x3, flags=0x32, fd=-1, offset=0x0
- len=0
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=6
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x1812, fd=6, offset=0x30000
 - region[3]: addr=0x434000, length=0x1000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[4]: addr=0x435000, length=0x1000, prot=0x0, flags=0x32, fd=-1, offset=0x0
 - region[5]: addr=0x436000, length=0x1000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[6]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
- ok
proc[6] calls sys_mmap
sys_mmap: addr=0x0, length=0x1000, prot=0x3, flags=0x22, fd=-1, offset=0x0
- mmap: addr=0x0, length=4096, prot=0x3, flags=0x22, fd=-1, offset=0x0
- return 0x430000
proc[6] calls sys_write
sh: 0:
```

## init.cでenvp[]に"DEBUG=1"を追加したら

```
execve: proc[6] '/bin/dash' start, argc=1, envc=1
- mmap: addr=0xfffffffe0000, length=131072, prot=0x7, flags=0x32, fd=-1, offset=0x0
pf_handler: fault_addr=0xfffffffff000, flt=3
pf_handler: fault_addr=0x0, flt=1
- mmap: addr=0x400000, length=196440, prot=0x5, flags=0x1812, fd=7, offset=0x0
- mmap: addr=0x431000, length=8752, prot=0x3, flags=0x1812, fd=7, offset=0x30000
- mmap: addr=0x434000, length=4096, prot=0x3, flags=0x32, fd=-1, offset=0x0
entry=0x4004a0, bss=0x433230, brk=0x434988
start_code=0x400000, end_code=0x42ff58
start_data=0x431940, end_data=0x433230
bprm->p=0x0
p[6]: sz=0x435000, kstack=0xffff00003affa000
pf_handler: fault_addr=0x402130, flt=3
pf_handler: fault_addr=0xffffffffffffffe0, flt=1
pf_handler: fault_addr=0xffffffffffffffe0, flt=1
pf_handler: fault_addr=0xffffffffffffffe0, flt=1
pf_handler: fault_addr=0xffffffffffffffe0, flt=1
Trap fault_addr=0xffffffffffffffe0 5 times
pf_handler: fault_addr: 0xffffffffffffffe0, flt: 1

====== TRAP FRAME DUMP ======
sp:	0x0
tpidr:	0x404630
spsr:	0x60000000
esr:	0x92000044
elr:	0x419b44
x0:	0x0
x1:	0xffff0000000ded20
x2:	0x8
x3:	0x400120
x4:	0x4292a8
x29:	0x0
x30:	0x0
====== DUMP END ======
```

## create_elf_tablesにバグあり

- lenにゼロ終端の1を足す

```
(gdb) p/x sp
$5 = 0xfffffffffe90
(gdb) x/-46gx 0x1000000000000
0xfffffffffe90:	0x0000000000000001	0x0000ffffffffffe1
0xfffffffffea0:	0x0000000000000000	0x0000ffffffffffe6
0xfffffffffeb0:	0x0000000000000000	0x0000000000000010
0xfffffffffec0:	0x0000000000000887	0x0000000000000006
0xfffffffffed0:	0x0000000000001000	0x0000000000000011
0xfffffffffee0:	0x0000000000000064	0x0000000000000003
0xfffffffffef0:	0x0000000000400040	0x0000000000000004
0xffffffffff00:	0x0000000000000038	0x0000000000000005
0xffffffffff10:	0x0000000000000004	0x0000000000000007
0xffffffffff20:	0x0000000000000000	0x0000000000000008
0xffffffffff30:	0x0000000000000000	0x0000000000000009
0xffffffffff40:	0x00000000004004a0	0x000000000000000b
0xffffffffff50:	0x0000000000000000	0x000000000000000c
0xffffffffff60:	0x0000000000000000	0x000000000000000d
0xffffffffff70:	0x0000000000000000	0x000000000000000e
0xffffffffff80:	0x0000000000000000	0x0000000000000017
0xffffffffff90:	0x0000000000000000	0x0000000000000033
0xffffffffffa0:	0x0000000000001207	0x000000000000000f
0xffffffffffb0:	0x0000ffffffffffd9	0x0000000000000000
0xffffffffffc0:	0x0000000000000000	0x0000000000000000
0xffffffffffd0:	0x0000000000000000	0x3436686372616100
0xffffffffffe0:	0x4544006873616400	0x622f00313d475542
0xfffffffffff0:	0x00687361642f6e69	0x0000000000000000
(gdb) x/s 0x0000ffffffffffd9
0xffffffffffd9:	"aarch64"
(gdb) x/48bc 0xffffffffffd0
0xffffffffffd0:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
0xffffffffffd8:	0 '\000'	97 'a'	97 'a'	114 'r'	99 'c'	104 'h'	54 '6'	52 '4'
0xffffffffffe0:	0 '\000'	100 'd'	97 'a'	115 's'	104 'h'	0 '\000'	68 'D'	69 'E'
0xffffffffffe8:	66 'B'	85 'U'	71 'G'	61 '='	49 '1'	0 '\000'	47 '/'	98 'b'
0xfffffffffff0:	105 'i'	110 'n'	47 '/'	100 'd'	97 'a'	115 's'	104 'h'	0 '\000'
0xfffffffffff8:	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'	0 '\000'
(gdb) x/s 0x0000ffffffffffe6
0xffffffffffe6:	"DEBUG=1"
(gdb) x/s 0x0000ffffffffffe1
0xffffffffffe1:	"dash"
```

## kmfreeでおかしな値を指定している

```
bprm->p=0xfffffffffe90
p[6]: sz=0x435000, kstack=0xffff00003affa000
- kmfree: filename=0xffff00003afe0cd0
- kmfree: envp[0]=0x402140
pf_handler: fault_addr=0x402130, flt=3
- kmfree: argv[0]=0xffff00003afe0d10

Thread 1 hit Breakpoint 1, kmfree (ap=0x402140) at kern/kmalloc.c:30
30	    bp = (Header*)ap - 1;
```

## sys_execveにバグがあり、envpのアドレスが間違っていた

```
p[6]: sz=0x435000, kstack=0xffff00003affa000
- kmfree: filename=0xffff00003afe0cd0
- kmfree: envp[0]=0xffff00003afe0cf0
- kmfree: argv[0]=0xffff00003afe0d10
```

## しかし、もとのエラーに戻っただけ

``
sys_mmap: addr=0x0, length=0x1000, prot=0x3, flags=0x22, fd=-1, offset=0x0
- mmap: addr=0x0, length=0x1000, prot=0x3, flags=0x22, fd=-1, offset=0x0
- return 0x430000
pf_handler: fault_addr=0x430008, flt=3
proc[6] calls sys_write
sh: 0:
```

## dashのmain()でargvをデバッグ出力

- `printf("argv[0]=%s\n", argv[0])`

```
dash: argv[0]=dash                    // argv[0]は正しく渡っている
```

## dashのinit()のPPIDを設定している分のckmallocに問題がある

- 一度、oldmallocに戻してみる

## dashは動いた

- echoなどのdash組み込みも動く
- lsなどのcoreutilsはエラー
- helloもエラー
- hellosは以下のエラー

```
# /bin/hellos
fork: copy_mmap_listfork: np[7]: nregions=5, head=0xffff00003afe0f50
execve: proc[7] '/bin/hellos' start, argc=1, envc=1
entry=0x400178, bss=0x403100, brk=0x403740
start_code=0x400000, end_code=0x40142c
start_data=0x402f48, end_data=0x403100
bprm->p=0xfffffffffe90
p[7]: sz=0x404000, kstack=0xffff00003a6a1000
hello static world!
fileclose: ref 0
```

## fileopen時にfiledupしていなかった

```
execve: proc[7] '/bin/hellos' start, argc=1, envc=1
fileclose: free_mmap_list f=0xffff00000010f0a8
fileclose: free_mmap_list f=0xffff00000010f0a8
fileclose: flush_old_exec f[10]=0xffff00000010f0d8
entry=0x400178, bss=0x403100, brk=0x403740
start_code=0x400000, end_code=0x40142c
start_data=0x402f48, end_data=0x403100
bprm->p=0xfffffffffe90
p[7]: sz=0x404000, kstack=0xffff00003a6a1000
fileclose: search_binary_handler f=0xffff00000010f108
fileclose: exec elf_file1 f=0xffff00000010f108
hello static world!
unexpected interrupt 0 at cpu 0

====== TRAP FRAME DUMP ======
sp:	0xfffffffffc20
tpidr:	0x4342f0
spsr:	0x60000000
esr:	0x2000000
elr:	0x414000
x0:	0x0
x1:	0x413fe8
x2:	0x434508
x3:	0x0
x4:	0x432000
x29:	0xfffffffffc20
x30:	0x4046cc
====== DUMP END ======

Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x414000
 SPSR_EL1 0x60000000 FAR_EL1 0x403520
	irq_error: irq of type 8 unimplemented.
```

## 初期状態に戻したらdashがエラーで動かない

- `(0x0 - 16)`をアドレスとしてアクセスしている模様

```
fork: copy_mmap_listexecve: proc[6] '/bin/dash' start, argc=1, envc=1
entry=0x4004a0, bss=0x432230, brk=0x433c08
bprm->p=0xfffffffffe90
p[6]: sz=0x434000, kstack=0xffff00003affa000
sys_brk: n=0x0, sz=0x434000
- invalid n 0 return 0x434000
sys_brk: n=0x435000, sz=0x434000
- ok return 0x435000
 PRINT mmap_region: pid=6, head=0xffff00003afe0d70, nregions=5
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x1812, fd=5, offset=0x0
 - region[2]: addr=0x430000, length=0x3000, prot=0x3, flags=0x1812, fd=6, offset=0x2f000
 - region[3]: addr=0x433000, length=0x1000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[4]: addr=0x434000, length=0x435000, prot=0x3, flags=0x32, fd=-1, offset=0x0
 - region[5]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, fd=-1, offset=0x0
Trap fault_addr=0xfffffffffffffff0 5 times
pf_handler: fault_addr: 0xfffffffffffffff0, flt: 1

====== TRAP FRAME DUMP ======
sp:	0xfffffffffba0
tpidr:	0x432df8
spsr:	0x80000000
esr:	0x92000044
elr:	0x41ab5c
x0:	0x0
x1:	0x432000
x2:	0x1
x3:	0x435000
x4:	0xfffffffffc38
x29:	0xfffffffffc40
x30:	0x41a76c
====== DUMP END ======
```

## muslのmalloc.cでsys_brk, sys_mmapの後にデバッグ出力を行うと上のエラーは起きない

- memoryアクセスで書き込んだデータがすぐに読めないためと思われるが、書き込み後すぐに
  読めるようにする方法は今のところわからない。

## mmaptestを実行していたところ以下のエラーで一切動かなくなった

- 直前まではmmapのテストが全クリアし、forkのテストがエラーになるので修正していた

```
# mmaptest
unexpected interrupt 0 at cpu 0

====== TRAP FRAME DUMP ======
sp:	0xfffffffffb00
tpidr:	0x4342f0
spsr:	0x0
esr:	0x2000000
elr:	0xfffffffffb00
x0:	0x0
x1:	0x0
x2:	0x0
x3:	0x8
x4:	0x436140
x29:	0xfffffffffb00
x30:	0xfffffffffb00
====== DUMP END ======

Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0xfffffffffb00
 SPSR_EL1 0x0 FAR_EL1 0xfffffffffae0
	irq_error: irq of type 8 unimplemented.
```

## 絶対パスで指定すると動いたのでmmatestのstack設定状態を出力

```
(gdb) p/x sp
$1 = 0xfffffffffe80
(gdb) x/-48gx 0x1000000000000
0xfffffffffe80:	0x0000000000000001	0x0000ffffffffffd5
0xfffffffffe90:	0x0000000000000000	0x0000ffffffffffe3
0xfffffffffea0:	0x0000000000000000	0x0000000000000010
0xfffffffffeb0:	0x0000000000000887	0x0000000000000006
0xfffffffffec0:	0x0000000000001000	0x0000000000000011
0xfffffffffed0:	0x0000000000000064	0x0000000000000003
0xfffffffffee0:	0x0000000000400040	0x0000000000000004
0xfffffffffef0:	0x0000000000000038	0x0000000000000005
0xffffffffff00:	0x0000000000000004	0x0000000000000007
0xffffffffff10:	0x0000000000000000	0x0000000000000008
0xffffffffff20:	0x0000000000000000	0x0000000000000009
0xffffffffff30:	0x0000000000400180	0x000000000000000b
0xffffffffff40:	0x0000000000000000	0x000000000000000c
0xffffffffff50:	0x0000000000000000	0x000000000000000d
0xffffffffff60:	0x0000000000000000	0x000000000000000e
0xffffffffff70:	0x0000000000000000	0x0000000000000017
0xffffffffff80:	0x0000000000000000	0x0000000000000033
0xffffffffff90:	0x0000000000001207	0x000000000000000f
0xffffffffffa0:	0x0000ffffffffffcd	0x0000000000000000
0xffffffffffb0:	0x0000000000000000	0x0000000000000000
0xffffffffffc0:	0x0000000000000000	0x7261610000000000
0xffffffffffd0:	0x69622f0034366863	0x657470616d6d2f6e
0xffffffffffe0:	0x2f3d445750007473	0x6d2f6e69622f002e
0xfffffffffff0:	0x007473657470616d	0x0000000000000000
(gdb) x/s 0x0000ffffffffffd5
0xffffffffffd5:	"/bin/mmaptest"
(gdb) x/s 0x0000ffffffffffe3
0xffffffffffe3:	"PWD=/."
(gdb) x/s 0x0000ffffffffffcd
0xffffffffffcd:	"aarch64"
(gdb) x/s 0x0000ffffffffffef
0xffffffffffef:	"mmaptest"
```

## trap frame dumpに全レジスタとプロセスID, プロセス名を出力

```
fork_test starting
sys_mmap: addr=0x0, length=0x2000, prot=0x1, flags=0x1, fd=9, offset=0x0
sys_mmap: addr=0x0, length=0x2000, prot=0x1, flags=0x1, fd=9, offset=0x0
p1[PGSIZE]=A
p2[PGSIZE]=A
sys_munmap: addr=0x40d000, length=0x1000
- munmap: addr=0x40d000, length=0x1000
- found: addr=0x40d000
unexpected interrupt 0 at cpu 0

======= PROC[7]: mmaptest TRAP FRAME DUMP =========
sp:    0x0000fffffffff9f0  tpidr: 0xffff00003a6a1e00
spsr:  0x0000000080000000  esr:   0x0000000002000000
elr:   0x0000000000405000  x0:    0x000000000040b530
x1:    0x000000000000000c  x2:    0x000000000040b010
x3:    0x000000000040b010  x4:    0x0000fffffffffba8
x5:    0x0000000000000000  x6:    0x0000000000407ff4
x7:    0x000000007fffffff  x8:    0x000000000000000a
x9:    0x0000000000000001  x10:   0x00000000ffffffff
x11:   0x414d20444145525f  x12:   0x20747365745f6b72
x13:   0x676e697472617473  x14:   0x0000000000000000
x15:   0x52505f50414d202c  x16:   0xa781e34554415649
x17:   0x83e38383e39e83e3  x18:   0x0000000000000000
x19:   0x000000000040b010  x20:   0x000000000000000c
x21:   0x000000000000000c  x22:   0x0000000000407fe8
x23:   0x0000fffffffffb88  x24:   0x000000000000000c
x25:   0x0000fffffffffb2f  x26:   0x0000fffffffffba8
x27:   0x00000000000017ff  x28:   0x0000000000407fe8
x29:   0x0000fffffffffdd0  x30:   0x00000000004037a8
===================== DUMP END =====================

Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x405000
 SPSR_EL1 0x80000000 FAR_EL1 0xfffffffffc60
	irq_error: irq of type 8 unimplemented.
```

## growproc, mremapを書き換え

- すぐにエラーになる場合と、実行できる場合がある

```
$ make qemu

execve: proc[1] '/bin/init' start, argc=1, envc=0
entry=0x400268, bss=0x404108, brk=0x404790
bprm->p=0xfffffffffea0
p[1]: sz=0x405000, kstack=0xffff00003affe000
init: starting dash
execve: proc[6] '/bin/dash' start, argc=1, envc=0
entry=0x4004a0, bss=0x433320, brk=0x435100
bprm->p=0xfffffffffeb0
p[6]: sz=0x436000, kstack=0xffff00003affa000
__expand_head: *pn=96, n=4096
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
- brk1=0x436000
sys_brk: n=0x437000, sz=0x436000
- ok return 0x437000
- brk2=0x437000, ret=0x436000
expand_heap: n=4096, p=0x0x436000, end=0x0
# /bin/mmaptest
unexpected interrupt 0 at cpu 0

======= PROC[7]: dash TRAP FRAME DUMP =========
sp:    0x0000fffffffffb20  tpidr: 0x00000000004342f0
spsr:  0x0000000000000000  esr:   0x0000000002000000
elr:   0x0000fffffffffb20  x0:    0x0000000000000000
x1:    0x0000000000000000  x2:    0x0000000000000000
x3:    0x0000000000000008  x4:    0x0000000000436100
x5:    0x0000000000433704  x6:    0x00000000000003b0
x7:    0x0000000000000027  x8:    0x00000000000000dc
x9:    0x0000000000000000  x10:   0x0000000000000000
x11:   0x0000fffffffffcb0  x12:   0x0000000000001030
x13:   0x3d6e702a203a6461  x14:   0x0000000000000001
x15:   0x0000000000000000  x16:   0x000000000041ee70
x17:   0x0000000000000000  x18:   0x0000000000000000
x19:   0x0000000000432000  x20:   0x0000000000434b80
x21:   0x0000000000000000  x22:   0x0000000000432000
x23:   0x0000000000000001  x24:   0x0000000000434b50
x25:   0x0000fffffffffd68  x26:   0x0000000000000000
x27:   0x0000000000000001  x28:   0x0000000000000000
x29:   0x0000fffffffffb20  x30:   0x0000fffffffffb20
===================== DUMP END =====================

Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0xfffffffffb20
 SPSR_EL1 0x0 FAR_EL1 0xfffffffffb00
	irq_error: irq of type 8 unimplemented.

$ make qemu

execve: proc[1] '/bin/init' start, argc=1, envc=0
entry=0x400268, bss=0x404108, brk=0x404790
bprm->p=0xfffffffffea0
p[1]: sz=0x405000, kstack=0xffff00003affe000
init: starting dash
execve: proc[6] '/bin/dash' start, argc=1, envc=0
entry=0x4004a0, bss=0x433320, brk=0x435100
bprm->p=0xfffffffffeb0
p[6]: sz=0x436000, kstack=0xffff00003affa000
__expand_head: *pn=96, n=4096
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
- brk1=0x436000
sys_brk: n=0x437000, sz=0x436000
- ok return 0x437000
- brk2=0x437000, ret=0x436000
expand_heap: n=4096, p=0x0x436000, end=0x0
# /bin/mmaptest
execve: proc[7] '/bin/mmaptest' start, argc=1, envc=1
entry=0x400180, bss=0x40b110, brk=0x40c798
bprm->p=0xfffffffffe80
p[7]: sz=0x40d000, kstack=0xffff00003a6a1000
mmap_testスタート
- open mmap.dur (fd=9) READ ONLY
[1] test mmap f
sys_mmap: addr=0x0, length=0x2000, prot=0x1, flags=0x2, fd=9, offset=0x0
- mmap fd=9 -> 2ページ分PROT_READ MAP_PRIVATEでマップ: addr=0x40d000
sys_munmap: addr=0x40d000, length=0x2000
- munmap: addr=0x40d000, length=0x2000
- found: addr=0x40d000
- munmmap 2PAGE: addr=0x40d000
[1] OK
[2] test mmap private
sys_mmap: addr=0x0, length=0x2000, prot=0x3, flags=0x2, fd=9, offset=0x0
- mmap fd=9 -> 2ページ分をPAGE PROT_READ/WRITE, MAP_PRIVATEでマップ: addr=0x40d000
- closed 9
- write 2PAGE = Z
- p[1]=A
sys_munmap: addr=0x40d000, length=0x2000
- munmap: addr=0x40d000, length=0x2000
- found: addr=0x40d000
- munmmap 2PAGE: addr=0x40d000
[2] OK
[3] test mmap read-only
- open mmap.dur (9) RDONALY
sys_mmap: addr=0x0, length=0x3000, prot=0x3, flags=0x1, fd=9, offset=0x0
- mmap 9 -> 3PAGE PROT_READ/WRITE MAP_SHARED: addr=0xffffffffffffffff
- close 9
[3] OK
[4] test mmap read/write
- open mmap.dur (9) RDWR
sys_mmap: addr=0x0, length=0x3000, prot=0x3, flags=0x1, fd=9, offset=0x0
- mmap 9 -> 3PAGE PROT_READ|WRITE MAP_SHARED: addr=0x40d000
- close 9
- print 2PAGE = Z
sys_munmap: addr=0x40d000, length=0x2000
- munmap: addr=0x40d000, length=0x2000
- found: addr=0x40d000
- munmmap 2PAGE: addr=0x40d000
[4] OK
[5] test mmap dirty
- open mmap.dur (9) RDWR
- close 9
[5] OK
[6] test not-mapped unmap
sys_munmap: addr=0x40f000, length=0x1000
- munmap: addr=0x40f000, length=0x1000
- found: addr=0x40f000
- munmmap PAGE: addr=0x40f000
[6] OK
[7] test mmap two files
- open mmap1 (9) RDWR/CREAT
sys_mmap: addr=0x0, length=0x1000, prot=0x1, flags=0x2, fd=9, offset=0x0
- mmap 9 -> PAGE PROT_READ MAP_PRIVATE: addr=0x40d000
- close 9
- unlink mmap1
- open mmap2 (9) RDWR/CREAT
sys_mmap: addr=0x0, length=0x1000, prot=0x1, flags=0x2, fd=9, offset=0x0
- mmap 9 -> PAGE PROT_READ MAP_PRIVATE: addr=0x40e000
- close 9
- unlink mmap2
sys_munmap: addr=0x40d000, length=0x1000
- munmap: addr=0x40d000, length=0x1000
- found: addr=0x40d000
- munmap PAGE: addr=0x40d000
sys_munmap: addr=0x40e000, length=0x1000
- munmap: addr=0x40e000, length=0x1000
- found: addr=0x40e000
- munmap PAGE: addr=0x40e000
[7] OK
mmap_test: Total: 7, OK: 7, NG: 0

fork_test starting
sys_mmap: addr=0x0, length=0x2000, prot=0x1, flags=0x1, fd=9, offset=0x0
sys_mmap: addr=0x0, length=0x2000, prot=0x1, flags=0x1, fd=9, offset=0x0
p1[PGSIZE]=A
p2[PGSIZE]=A
sys_munmap: addr=0x40d000, length=0x1000
- munmap: addr=0x40d000, length=0x1000
- found: addr=0x40d000
unexpected interrupt 0 at cpu 0

======= PROC[7]: mmaptest TRAP FRAME DUMP =========
sp:    0x0000fffffffff9f0  tpidr: 0x000000000040b638
spsr:  0x0000000080000000  esr:   0x0000000002000000
elr:   0x0000000000405000  x0:    0x000000000040b530
x1:    0x000000000000000c  x2:    0x000000000040b010
x3:    0x000000000040b010  x4:    0x0000fffffffffba8
x5:    0x0000000000000000  x6:    0x0000000000407ff4
x7:    0x000000007fffffff  x8:    0x000000000000000a
x9:    0x0000000000000001  x10:   0x00000000ffffffff
x11:   0x414d20444145525f  x12:   0x20747365745f6b72
x13:   0x676e697472617473  x14:   0x0000000000000000
x15:   0x52505f50414d202c  x16:   0xa781e34554415649
x17:   0x83e38383e39e83e3  x18:   0x0000000000000000
x19:   0x000000000040b010  x20:   0x000000000000000c
x21:   0x000000000000000c  x22:   0x0000000000407fe8
x23:   0x0000fffffffffb88  x24:   0x000000000000000c
x25:   0x0000fffffffffb2f  x26:   0x0000fffffffffba8
x27:   0x00000000000017ff  x28:   0x0000000000407fe8
x29:   0x0000fffffffffdd0  x30:   0x00000000004037a8
===================== DUMP END =====================

Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x405000
 SPSR_EL1 0x80000000 FAR_EL1 0xfffffffffc60
	irq_error: irq of type 8 unimplemented.
```

```
execve: proc[6] '/bin/dash' start, argc=1, envc=0
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_rt_sigprocmask
proc[1] calls sys_wait4
entry=0x4004a0, bss=0x433320, brk=0x435100
bprm->p=0xfffffffffeb0
p[6]: sz=0x436000, kstack=0xffff00003affa000
proc[6] calls sys_gettid
proc[6] calls sys_getpid
proc[6] calls sys_rt_sigprocmask
proc[6] calls sys_rt_sigaction
proc[6] calls sys_geteuid
proc[6] calls sys_getppid
proc[6] calls sys_ioctl
proc[6] calls sys_writev
__expand_head: *pn=96, n=4096
proc[6] calls sys_brk
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
proc[6] calls sys_writev
- brk1=0x436000
proc[6] calls sys_brk
sys_brk: n=0x437000, sz=0x436000
- ok return 0x437000
proc[6] calls sys_writev
- brk2=0x437000, ret=0x436000
proc[6] calls sys_writev
expand_heap: n=4096, p=0x0x436000, end=0x0
proc[6] calls sys_getcwd
proc[6] calls sys_ioctl
proc[6] calls sys_ioctl
proc[6] calls sys_rt_sigaction
proc[6] calls sys_rt_sigaction
proc[6] calls sys_rt_sigaction
proc[6] calls sys_rt_sigaction
proc[6] calls sys_rt_sigaction
proc[6] calls sys_rt_sigaction
proc[6] calls sys_openat
proc[6] calls sys_fcntl
proc[6] calls sys_close
proc[6] calls sys_fcntl
proc[6] calls sys_ioctl
proc[6] calls sys_getpgid
proc[6] calls sys_rt_sigaction
proc[6] calls sys_rt_sigaction
proc[6] calls sys_rt_sigaction
proc[6] calls sys_rt_sigaction
proc[6] calls sys_rt_sigaction
proc[6] calls sys_rt_sigaction
proc[6] calls sys_setpgid
proc[6] calls sys_ioctl
proc[6] calls sys_write
# proc[6] calls sys_read
```

```
cmdloop(1) called
call setstackmark
call jobctl
chkmail
parsecmd
unexpected interrupt 0 at cpu 0

======= PROC[1]: init TRAP FRAME DUMP =========
sp:    0x0000fffffffffe00  tpidr: 0x0000000000404630
spsr:  0x0000000020000000  esr:   0x0000000002000000
elr:   0x0000000000400e4c  x0:    0x0000000000000006
x1:    0x0000000000000000  x2:    0x0000000000000000
x3:    0x0000000000000000  x4:    0x0000000000000000
x5:    0x0000000000000000  x6:    0x0000000000000000
x7:    0x6420676e69747261  x8:    0x0000000000000104
x9:    0x0000000000000000  x10:   0x0000000000400040
x11:   0x000000006474e551  x12:   0x7472617473203a74
x13:   0x6873616420676e69  x14:   0x0000000000000000
x15:   0x0000000000000000  x16:   0x000000000040055c
x17:   0x0000000000000000  x18:   0x0000000000000000
x19:   0x0000000000000006  x20:   0x0000000000400988
x21:   0x0000000000402178  x22:   0x0000000000403000
x23:   0x0000000000402000  x24:   0x00000000004008bc
x25:   0x0000000000000000  x26:   0x0000000000000000
x27:   0x0000000000000000  x28:   0x0000000000000000
x29:   0x0000fffffffffe10  x30:   0x00000000004008f0
===================== DUMP END =====================

Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x400e4c
 SPSR_EL1 0x20000000 FAR_EL1 0x436010     // FAR_EL1: 番兵wのアドレス
	irq_error: irq of type 8 unimplemented.
```

```
execve: proc[6] '/bin/dash' start, argc=1, envc=0
entry=0x4004a0, bss=0x433320, brk=0x435100
bprm->p=0xfffffffffeb0
p[6]: sz=0x436000, kstack=0xffff00003affa000
__expand_head: *pn=96, n=4096
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
- brk1=0x436000
sys_brk: n=0x437000, sz=0x436000
- ok return 0x437000
- brk2=0x437000, ret=0x436000
expand_heap: n=4096, p=0x0x436000, end=0x0
- n=4064, p=0x0x436020
- w->psize=1, w->psize=0
- end=0x0x437000
- w->psize=4065, w->psize=1
- w->psize=1, w->psize=4065
unexpected interrupt 0 at cpu 0

struct chunk {
	size_t psize, csize;
	struct chunk *next, *prev;
};


- w->psize=1, w->psize=0                                                        // 個別にprintfするとw->psize = 1だが
- w->psize=4065, w->psize=1
- w->psize=1, w->psize=4065

436000: 0000000000000000 0000000000000000 0000000000000000 0000000000000fe1     // 個別にprintfせず、後でまとめてprintfすると
436fe0: 0000000000000000 0000000000000000 0000000000000fe1 0000000000000001     // w->psize = 1 になっていない

___expand_head: *pn=96, n=4096
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
- brk1=0x436000
sys_brk: n=0x437000, sz=0x436000
- ok return 0x437000
- brk2=0x437000, ret=0x436000
pf_handler: fault_addr=0x436010, flt=3
436000: 0000000000000000 0000000000000000 0000000000000000 0000000000000000     // if (p != end) {}後。セットされていない
436fe0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
- end=0x0x437000
436000: 0000000000000000 0000000000000000 0000000000000000 0000000000000fe1     // return前。セットされていない
436fe0: 0000000000000000 0000000000000000 0000000000000fe1 0000000000000001

pf_handler: fault_addr=0x436010, flt=3                                          // w->psize = 0 | C_INUSEでtrap発生
- w->psize=1, w->psize=0                                                        // trap後にprintf
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000000     // セットされている
436fe0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
- end=0x0x437000
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000fe1     // セットされている
436fe0: 0000000000000000 0000000000000000 0000000000000fe1 0000000000000001

```

```
execve: proc[1] '/bin/init' start, argc=1, envc=0

pf_handler: fault_addr=0x74696e6933, esr=0x96000005, ec=0x25, iss=0x5, dfs=5, flt=1         // カーネルDABORT2, 翻訳 1 (read)
mmap: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=1476395425, offset=0x0
- call mmap_load_pages: addr=0xfffffffe0000, length=0x20000, ref=1

pf_handler: fault_addr=0xfffffffff000, esr=0x9600004f, ec=0x25, iss=0x4f, dfs=15, flt=3     // カーネルDABORT2, パーミション 3
mmap: addr=0x400000, length=0x2274, prot=0x5, flags=0x812, f=22, offset=0x0
- call mmap_load_pages: addr=0x400000, length=0x2274, ref=1
mmap: addr=0x403000, length=0x1108, prot=0x3, flags=0x12, f=22, offset=0x2000
- call mmap_load_pages: addr=0x403000, length=0x1108, ref=1

pf_handler: fault_addr=0x404108, esr=0x9600004f, ec=0x25, iss=0x4f, dfs=15, flt=3           // ユーザDABORT, パーミション 3

start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bss=0x404108, brk=0x404790
entry=0x400268, bprm->p=0xfffffffffea0

init: starting dash
pf_handler: fault_addr=0x4021a0, esr=0x9600004f, ec=0x25, iss=0x4f, dfs=15, flt=3           // カーネルDABORT2, パーミション 3
pf_handler: fault_addr=0x402198, esr=0x9600004f, ec=0x25, iss=0x4f, dfs=15, flt=3           // カーネルDABORT2, パーミション 3
pf_handler: fault_addr=0x404588, esr=0x9200004f, ec=**0x24**, iss=0x4f, dfs=15, flt=3       // ユーザDABORT, パーミション 3
pf_handler: fault_addr=0xfffffffffca0, esr=0x9200004f, ec=**0x24**, iss=0x4f, dfs=15, flt=3 // ユーザDABORT, パーミション 3

execve: proc[6] '/bin/dash' start, argc=1, envc=0

pf_handler: fault_addr=0x18, esr=0x96000006, ec=0x25, iss=0x6, dfs=6, flt=1                 // カーネルDABORT2, 翻訳 2 (read)
mmap: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
- call mmap_load_pages: addr=0xfffffffe0000, length=0x20000, ref=1

pf_handler: fault_addr=0xfffffffff000, esr=0x9600004f, ec=0x25, iss=0x4f, dfs=15, flt=3     // カーネルDABORT2, パーミション 3
mmap: addr=0x400000, length=0x2fc70, prot=0x5, flags=0x812, f=144, offset=0x0
- call mmap_load_pages: addr=0x400000, length=0x2fc70, ref=1
mmap: addr=0x431000, length=0x2320, prot=0x3, flags=0x12, f=144, offset=0x30000
- call mmap_load_pages: addr=0x431000, length=0x2320, ref=1
mmap: addr=0x434000, length=0x2000, prot=0x3, flags=0x32, f=0, offset=0x0
- call mmap_load_pages: addr=0x434000, length=0x2000, ref=1

pf_handler: fault_addr=0x433320, esr=0x9600004f, ec=0x25, iss=0x4f, dfs=15, flt=3           // カーネルDABORT2, パーミション 3

start_code=0x400000, end_code=0x42fc70
start_data=0x431940, end_data=0x433320
bss=0x433320, brk=0x435100
entry=0x4004a0, bprm->p=0xfffffffffeb0

pf_handler: fault_addr=0x435090, esr=0x9200004f, ec=**0x24**, iss=0x4f, dfs=15, flt=3       // ユーザDABORT, パーミション 3

__expand_head: *pn=96, n=4096
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
- brk1=0x436000

sys_brk: n=0x437000, sz=0x436000
mmap: addr=0x436000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
- call mmap_load_pages: addr=0x436000, length=0x1000, ref=1
- ok return 0x437000
- brk2=0x437000, ret=0x436000

expand_heap: n=4096, p=0x0x436000, end=0x0

pf_handler: fault_addr=0x436010, esr=0x9200004f, ec=**0x24**, iss=0x4f, dfs=15, flt=3       // ユーザDABORT, パーミション 3

- w->psize=1, w->psize=0
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000000
436fe0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
- end=0x0x437000
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000fe1
436fe0: 0000000000000000 0000000000000000 0000000000000fe1 0000000000000001
#
```

## delete_mmap_node()でmemset(node, 0, sizeof(struct mmap_region))あり

```
init: starting dash
execve: proc[6] '/bin/dash' start, argc=1, envc=0
== PRINT mmap_region[6] (delete node before): head=0xffff00003afe0d70, nregions=3 ==
 - region[1]: addr=0x400000, length=0x3000, prot=0x5, flags=0x812, f=22, offset=0x0
 - region[2]: addr=0x403000, length=0x2000, prot=0x3, flags=0x12, f=22, offset=0x2000
- FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (delete node after): head=0xffff00003afe0d20, nregions=3 ==
 - region[1]: addr=0x403000, length=0x2000, prot=0x3, flags=0x12, f=22, offset=0x2000
 - region[2]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
mmap: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
- call mmap_load_pages: addr=0xfffffffe0000, length=0x20000, ref=1
map_region: remap *pte=0x2000003af87747, va=0xfffffffe0000
kern/console.c:303: kernel panic at cpu 0.
```

## なし

```
init: starting dash
execve: proc[6] '/bin/dash' start, argc=1, envc=0
== PRINT mmap_region[6] (delete node before): head=0xffff00003afe0d70, nregions=3 ==
 - region[1]: addr=0x400000, length=0x3000, prot=0x5, flags=0x812, f=22, offset=0x0
 - region[2]: addr=0x403000, length=0x2000, prot=0x3, flags=0x12, f=22, offset=0x2000
- FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (delete node after): head=0xffff00003afe0d20, nregions=3 ==
 - region[1]: addr=0x403000, length=0x2000, prot=0x3, flags=0x12, f=22, offset=0x2000
 - region[2]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (delete node before): head=0xffff00003afe0d20, nregions=2 ==
 - region[1]: addr=0x403000, length=0x2000, prot=0x3, flags=0x12, f=22, offset=0x2000
 - region[2]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (delete node after): head=0xffff00003afe0cd0, nregions=2 ==
 - region[1]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (delete node before): head=0xffff00003afe0cd0, nregions=1 ==
 - region[1]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (delete node after): head=0x0, nregions=1 ==
mmap: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
- call mmap_load_pages: addr=0xfffffffe0000, length=0x20000, ref=1
mmap: addr=0x400000, length=0x2fc70, prot=0x5, flags=0x812, f=144, offset=0x0
- call mmap_load_pages: addr=0x400000, length=0x2fc70, ref=1
mmap: addr=0x431000, length=0x2320, prot=0x3, flags=0x12, f=144, offset=0x30000
- call mmap_load_pages: addr=0x431000, length=0x2320, ref=1
mmap: addr=0x434000, length=0x2000, prot=0x3, flags=0x32, f=0, offset=0x0
- call mmap_load_pages: addr=0x434000, length=0x2000, ref=1
start_code=0x400000, end_code=0x42fc70
start_data=0x431940, end_data=0x433320
bss=0x433320, brk=0x435100
entry=0x4004a0, bprm->p=0xfffffffffeb0
__expand_head: *pn=96, n=4096
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
- brk1=0x436000
sys_brk: n=0x437000, sz=0x436000
mmap: addr=0x436000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
- call mmap_load_pages: addr=0x436000, length=0x1000, ref=1
- ok return 0x437000
- brk2=0x437000, ret=0x436000
expand_heap: n=4096, p=0x0x436000, end=0x0
- w->psize=1, w->psize=0
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000000
436fe0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
- end=0x0x437000
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000fe1
436fe0: 0000000000000000 0000000000000000 0000000000000fe1 0000000000000001
#
```

## stackを出力した

```
execve: proc[6] '/bin/dash' start, argc=1, envc=0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d70, nregions=1 ==
- FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
 - region[1]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=2 ==
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=3 ==
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=4 ==
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0x434000, length=0x2000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
start_code=0x400000, end_code=0x42fc70
start_data=0x431940, end_data=0x433320
bss=0x433320, brk=0x435100
entry=0x4004a0, bprm->p=0xfffffffffeb0
__expand_head: *pn=96, n=4096
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
- brk1=0x436000
sys_brk: n=0x437000, sz=0x436000
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=5 ==
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0x434000, length=0x2000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0x436000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[5]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
- ok return 0x437000
- brk2=0x437000, ret=0x436000
expand_heap: n=4096, p=0x0x436000, end=0x0
- w->psize=1, w->psize=0
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000000
436fe0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
- end=0x0x437000
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000fe1
436fe0: 0000000000000000 0000000000000000 0000000000000fe1 0000000000000001
# /bin/myls
sys_wait4: pid: -1, wstatus: fffffffffb1c, options: 0x2, rusage: 0
wait4: pid=6, wpid=2
- FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0                                 // fault_addr: 0x0 はおかしい?
execve: proc[7] '/bin/myls' start, argc=1, envc=1
== PRINT mmap_region[7] (mmap): head=0xffff00003afe0c80, nregions=1 ==
 - region[1]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[7] (mmap): head=0xffff00003afe0c30, nregions=2 ==
 - region[1]: addr=0x400000, length=0x8000, prot=0x5, flags=0x812, f=30, offset=0x0
 - region[2]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[7] (mmap): head=0xffff00003afe0c30, nregions=3 ==
 - region[1]: addr=0x400000, length=0x8000, prot=0x5, flags=0x812, f=30, offset=0x0
 - region[2]: addr=0x408000, length=0x2000, prot=0x3, flags=0x12, f=30, offset=0x7000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[7] (mmap): head=0xffff00003afe0c30, nregions=4 ==
 - region[1]: addr=0x400000, length=0x8000, prot=0x5, flags=0x812, f=30, offset=0x0
 - region[2]: addr=0x408000, length=0x2000, prot=0x3, flags=0x12, f=30, offset=0x7000
 - region[3]: addr=0x40a000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
start_code=0x400000, end_code=0x4075d8
start_data=0x408ec8, end_data=0x4091f0
bss=0x4091f0, brk=0x40a868
entry=0x4001a8, bprm->p=0xfffffffffe90
drwxrwxr-x   1       4096 .
drwxrwxr-x   1       4096 ..
drwxrwxr-x   2       8448 bin
drwxrwxr-x   3        384 dev
drwxrwxr-x   8        320 etc
drwxrwxrwx   9        192 lib
drwxrwxr-x  10        192 home
drwxrwxr-x  12        192 usr
sys_exit: proc[7], status=0
wait4: exit proc 7, status=0
unexpected interrupt 0 at cpu 0

======= PROC[6]: /bin/dash TRAP FRAME DUMP =========
sp:    0x0000fffffffffa90  tpidr: 0x00000000004342f0
spsr:  0x0000000020000000  esr:   0x0000000002000000
elr:   0x0000000000419f70  x0:    0x0000000000000007
x1:    0x0000fffffffffb1c  x2:    0x0000000000000002
x3:    0x0000000000000000  x4:    0x0000000000432000
x5:    0x00000000004361ca  x6:    0x6c796d2f6e69622f
x7:    0x00736c796d2f6e69  x8:    0x0000000000000104
x9:    0x0000000000429000  x10:   0x0000000000000000
x11:   0x0000000000000000  x12:   0x0000000000429418
x13:   0x00000000004293f8  x14:   0x0000000000429410
x15:   0x0000000000429720  x16:   0x0000000000429400
x17:   0x0000000000000022  x18:   0x0000fffffffffa90
x19:   0x0000000000432000  x20:   0x0000fffffffffb20
x21:   0x0000000000000001  x22:   0x0000000000419f48
x23:   0x0000000000000001  x24:   0x0000000000000002
x25:   0x0000000000419be4  x26:   0x0000fffffffffb1c
x27:   0x0000fffffffffba0  x28:   0x0000000000000001
x29:   0x0000fffffffffaa0  x30:   0x000000000040b718
===================== DUMP END =====================

==== stack proc[6] =========
fffffffffa90: 000000000040b718 0000000000000000 0000fffffffffc60 000000000040c8dc
fffffffffab0: 0000000000436100 0000000000432000 0000000000000001 0000000000432000
fffffffffad0: 0000000000000001 0000000000434b50 0000fffffffffd68 0000000000000000
fffffffffaf0: 0000000000000001 0000000000000000 000000000040d584 0000000000436100
fffffffffb10: 0000fffffffffb20 000000000040c820 0000fffffffffc90 00000000004044d4
fffffffffb30: 000000000042a1c5 0000000000434b80 0000000000000000 0000000000405534
fffffffffb50: 0000fffffffffc90 0000000000436100 0000000000434b50 0000000000434b98
fffffffffb70: 000000000042a1c5 ffffffff00432000 0000fffffffffd68 0000000000000000       // ffffffff00432000 <= kernel mem ?
fffffffffb90: 0000fffffffffd68 0000000000000000 0000fffffffffb10 0000000000413f08
fffffffffbb0: 0000000000432000 0000fffffffffb10 0000000000000000 0000000000000000
fffffffffbd0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffbf0: 0000000000000000 000000000042a1c5 000000000040d514 0000000000000000
fffffffffc10: 0000fffffffffc30 000000000040973c 0000000000433000 000000000040976c
fffffffffc30: 0000fffffffffc90 00000000004043a8 0000000000000000 0000000000434b80
fffffffffc50: 0000000000000018 00114b8f3798ec48 0000fffffffffc90 0000000000404660
fffffffffc70: 000000000042a1c5 0000000000434b80 0000000000000000 00114b8f3798ec48
fffffffffc90: 0000fffffffffd90 0000000000402f64 0000000000414740 0000000000432000
fffffffffcb0: 0000000000000002 0000000000000000 0000fffffffffdd0 0000000000432000
fffffffffcd0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
fffffffffcf0: 0000000000432000 0000000000433000 0000fffffffffd78 ffffffff00000000       // ffffffff00000000
fffffffffd10: 0000000000000000 0000000000434b70 0000000000434950 0000000000000000
fffffffffd30: 0000000000000000 0000000000434b98 0000000000434b90 0000000000000000
fffffffffd50: 0000000000434b50 0000000000434b80 0000000000434b80 0000000000000000
fffffffffd70: 0000fffffffffd68 0000000000000000 00000000ffffffff 00114b8f3798ec48       // 00114b8f3798ec48 <= invalid mem ?
fffffffffd90: 0000fffffffffdf0 0000000000400454 0000000000432000 0000fffffffffe60
fffffffffdb0: 0000000000000000 000000000040d7f0 0000000000429798 00000000004003bc
fffffffffdd0: 0000000000434b18 0000000000434b70 00000000000001a8 00114b8f3798ec48       // 00114b8f3798ec48
fffffffffdf0: 0000000000000000 0000000000419b40 0000000000000001 0000fffffffffeb8
fffffffffe10: 0000000000400150 0000fffffffffec8 0000000000402000 0000000000000000
fffffffffe30: 0000000100000000 0000fffffffffeb8 0000000400000000 0000000000434b18
fffffffffe50: 0000000000434b20 00000000000001f8 0000000000434b18 0000000000434b20
fffffffffe70: 00000000000001f8 00114b8f3798ec48 0000000000000000 0000000000400988       // 00114b8f3798ec48
fffffffffe90: 0000000000402178 0000000000403000 0000000000000000 0000000000000000
fffffffffeb0: 0000000000000001 0000ffffffffffe9 0000000000000000 0000000000000000       // bprm->p = 0xfffffffffeb0
fffffffffed0: 0000000000000010 0000000000000887 0000000000000006 0000000000001000
fffffffffef0: 0000000000000011 0000000000000064 0000000000000003 0000000000400040
ffffffffff10: 0000000000000004 0000000000000038 0000000000000005 0000000000000004
ffffffffff30: 0000000000000007 0000000000000000 0000000000000008 0000000000000000
ffffffffff50: 0000000000000009 00000000004004a0 000000000000000b 0000000000000000
ffffffffff70: 000000000000000c 0000000000000000 000000000000000d 0000000000000000
ffffffffff90: 000000000000000e 0000000000000000 0000000000000017 0000000000000000
ffffffffffb0: 0000000000000033 0000000000001207 000000000000000f 0000ffffffffffe1
ffffffffffd0: 0000000000000000 0000000000000000 3436686372616100 622f006873616400
fffffffffff0: 00687361642f6e69 0000000000000000
==== stack proc[6] =========

Synchronous: Unknown:
  ESR_EL1 0x2000000 ELR_EL1 0x419f70
 SPSR_EL1 0x20000000 FAR_EL1 0x40a640
	irq_error: irq of type 8 unimplemented.
```

## `FAR_EL1 0x40a640`はonint()関数に該当する

- SIGINTを受信した際にtrap.cのonsig(int signo)から呼び出される
- SIGCHLD


## dashのsigaction呼び出し

```
sys_rt_sigaction: signum: 2, act: not, oldact: set
sys_rt_sigaction: signum: 2, act: set, oldact: not
sys_rt_sigaction: signum: 3, act: not, oldact: set
sys_rt_sigaction: signum: 3, act: set, oldact: not
sys_rt_sigaction: signum: 15, act: not, oldact: set
sys_rt_sigaction: signum: 15, act: set, oldact: not
sys_rt_sigaction: signum: 20, act: not, oldact: set
sys_rt_sigaction: signum: 20, act: set, oldact: not
sys_rt_sigaction: signum: 22, act: not, oldact: set
sys_rt_sigaction: signum: 22, act: set, oldact: not
sys_rt_sigaction: signum: 21, act: not, oldact: set
sys_rt_sigaction: signum: 21, act: set, oldact: not
```

```
execve: proc[6] '/bin/dash' start, argc=1, envc=0
start_code=0x400000, end_code=0x42fc70
start_data=0x431940, end_data=0x433320
bss=0x433320, brk=0x435100
entry=0x4004a0, bprm->p=0xfffffffffeb0
Breakpoint 1 at 0xffff00000009b1e4: file kern/sysproc.c, line 155.
(gdb) c
Continuing.

// sys_rt_sigaction: signum: 17, act: handler=0x4141a0, flags=0x4000000, restorer=0x423b0c, mask=0xfffffffc7fffffff
// sigaction: signum: 17, handler: 0x4141a0, mask: 0xfffffffc7fffffff, flags=0x4000000

$1 = 17 (SIGCHLD)
$2 = {
  handler = 0x4141a0, (dash: onsig())
  flags = 0x4000000,
  restorer = 0x423b0c,
  mask = {0x7fffffff, 0xfffffffc}
}
0xfffffffffb60:	0xa0	0x41	0x41	0x00	0x00	0x00	0x00	0x00
0xfffffffffb68:	0x00	0x00	0x00	0x04	0x00	0x00	0x00	0x00
0xfffffffffb70:	0x0c	0x3b	0x42	0x00	0x00	0x00	0x00	0x00
0xfffffffffb78:	0xff	0xff	0xff	0x7f	0xfc	0xff	0xff	0xff
(gdb) c
Continuing.

// sys_rt_sigaction: signum: 2, act: handler=0x4141a0, flags=0x4000000, restorer=0x423b0c, mask=0xfffffffc7fffffff
// sigaction: signum: 2, handler: 0x4141a0, mask: 0xfffffffc7fffffff, flags=0x4000000

$3 = 2 (SIGINT)
$4 = {
  handler = 0x4141a0, (dash: onsig())
  flags = 0x4000000,
  restorer = 0x423b0c,
  mask = {0x7fffffff, 0xfffffffc}
}
0xfffffffffc50:	0xa0	0x41	0x41	0x00	0x00	0x00	0x00	0x00
0xfffffffffc58:	0x00	0x00	0x00	0x04	0x00	0x00	0x00	0x00
0xfffffffffc60:	0x0c	0x3b	0x42	0x00	0x00	0x00	0x00	0x00
0xfffffffffc68:	0xff	0xff	0xff	0x7f	0xfc	0xff	0xff	0xff
(gdb) c
Continuing.

// sys_rt_sigaction: signum: 3, act: handler=0x1, flags=0x4000000, restorer=0x423b0c, mask=0xfffffffc7fffffff
// sigaction: signum: 3, handler: 0x1, mask: 0xfffffffc7fffffff, flags=0x4000000

$5 = 3 (SIGQUIT)
$6 = {
  handler = 0x1, (SIG_IGN)
  flags = 0x4000000,
  restorer = 0x423b0c,
  mask = {0x7fffffff, 0xfffffffc}
}
0xfffffffffc50:	0x01	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0xfffffffffc58:	0x00	0x00	0x00	0x04	0x00	0x00	0x00	0x00
0xfffffffffc60:	0x0c	0x3b	0x42	0x00	0x00	0x00	0x00	0x00
0xfffffffffc68:	0xff	0xff	0xff	0x7f	0xfc	0xff	0xff	0xff
(gdb) c
Continuing.

// sys_rt_sigaction: signum: 15, act: handler=0x1, flags=0x4000000, restorer=0x423b0c, mask=0xfffffffc7fffffff
// sigaction: signum: 15, handler: 0x1, mask: 0xfffffffc7fffffff, flags=0x4000000

$7 = 15 (SIGTERM)
$8 = {
  handler = 0x1, (SIG_IGN)
  flags = 0x4000000,
  restorer = 0x423b0c,
  mask = {0x7fffffff, 0xfffffffc}
}
0xfffffffffc50:	0x01	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0xfffffffffc58:	0x00	0x00	0x00	0x04	0x00	0x00	0x00	0x00
0xfffffffffc60:	0x0c	0x3b	0x42	0x00	0x00	0x00	0x00	0x00
0xfffffffffc68:	0xff	0xff	0xff	0x7f	0xfc	0xff	0xff	0xff
(gdb) c
Continuing.

// sys_rt_sigaction: signum: 20, act: handler=0x1, flags=0x4000000, restorer=0x423b0c, mask=0xfffffffc7fffffff
// sigaction: signum: 20, handler: 0x1, mask: 0xfffffffc7fffffff, flags=0x4000000

$9 = 20 (SIGTSTP)
$10 = {
  handler = 0x1, (SIG_IGN)
  flags = 0x4000000,
  restorer = 0x423b0c,
  mask = {0x7fffffff, 0xfffffffc}
}
0xfffffffffbf0:	0x01	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0xfffffffffbf8:	0x00	0x00	0x00	0x04	0x00	0x00	0x00	0x00
0xfffffffffc00:	0x0c	0x3b	0x42	0x00	0x00	0x00	0x00	0x00
0xfffffffffc08:	0xff	0xff	0xff	0x7f	0xfc	0xff	0xff	0xff
(gdb) c
Continuing.

// sys_rt_sigaction: signum: 22, act: handler=0x1, flags=0x4000000, restorer=0x423b0c, mask=0xfffffffc7fffffff
// sigaction: signum: 22, handler: 0x1, mask: 0xfffffffc7fffffff, flags=0x4000000

$11 = 22 (SIGTTOU)
$12 = {
  handler = 0x1, (SIG_IGN)
  flags = 0x4000000,
  restorer = 0x423b0c,
  mask = {0x7fffffff, 0xfffffffc}
}
0xfffffffffbf0:	0x01	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0xfffffffffbf8:	0x00	0x00	0x00	0x04	0x00	0x00	0x00	0x00
0xfffffffffc00:	0x0c	0x3b	0x42	0x00	0x00	0x00	0x00	0x00
0xfffffffffc08:	0xff	0xff	0xff	0x7f	0xfc	0xff	0xff	0xff
(gdb) c
Continuing.

// sys_rt_sigaction: signum: 21, act: handler=0x0, flags=0x4000000, restorer=0x423b0c, mask=0xfffffffc7fffffff
// sigaction: signum: 21, handler: 0x0, mask: 0xfffffffc7fffffff, flags=0x4000000

$13 = 21 (SIGTTIN)
$14 = {
  handler = 0x0, (SIG_DFL)
  flags = 0x4000000,
  restorer = 0x423b0c, (dash:__restorer(): call SYS_rt_sigreturn)
  mask = {0x7fffffff, 0xfffffffc}
}
0xfffffffffbf0:	0x00	0x00	0x00	0x00	0x00	0x00	0x00	0x00
0xfffffffffbf8:	0x00	0x00	0x00	0x04	0x00	0x00	0x00	0x00
0xfffffffffc00:	0x0c	0x3b	0x42	0x00	0x00	0x00	0x00	0x00
0xfffffffffc08:	0xff	0xff	0xff	0x7f	0xfc	0xff	0xff	0xff
(gdb) c
Continuing.
Remote connection closed
```

## sigaction, sigprocmaskを修正

```
execve: proc[1] '/bin/init' start, argc=1, envc=0
start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bss=0x404108, brk=0x404790
entry=0x400268, bprm->p=0xfffffffffea0
init: starting dash
sigprocmask[1]: how=0 oldmask=0x0 set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
sigprocmask[1]: how=0 oldmask=0xfffffffc7fffffff set=0xffffffffffffffff newmask=0xffffffffffffffff
sigprocmask[6]: how=2 oldmask=0xffffffffffffffff set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
sigprocmask[1]: how=2 oldmask=0xffffffffffffffff set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
sigprocmask[1]: how=2 oldmask=0xfffffffc7fffffff set=0x0 newmask=0x0
sys_wait4: pid: -1, wstatus: 0, options: 0x0, rusage: 0
wait4: pid=1, wpid=0
sigprocmask[6]: how=2 oldmask=0xfffffffc7fffffff set=0x0 newmask=0x0
execve: proc[6] '/bin/dash' start, argc=1, envc=0
- FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
start_code=0x400000, end_code=0x42fc70
start_data=0x431940, end_data=0x433320
bss=0x433320, brk=0x435100
entry=0x4004a0, bprm->p=0xfffffffffeb0
sigprocmask[6]: how=1 oldmask=0x0 set=0x300000000 newmask=0x0
sigaction[6]: signum=17 handler=0x4141a0 mask=0xfffffffc7fffffff flags=0x4000000
__expand_head: *pn=96, n=4096
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
- brk1=0x436000
sys_brk: n=0x437000, sz=0x436000
- ok return 0x437000
- brk2=0x437000, ret=0x436000
expand_heap: n=4096, p=0x0x436000, end=0x0
- w->psize=1, w->psize=0
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000000
436fe0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
- end=0x0x437000
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000fe1
436fe0: 0000000000000000 0000000000000000 0000000000000fe1 0000000000000001
sigaction[6]: signum=2 handler=0x4141a0 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=3 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=15 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=20 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=22 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=21 handler=0x0 mask=0xfffffffc7fffffff flags=0x4000000
# /bin/myls
sigprocmask[6]: how=2 oldmask=0xfffffffc7ffbfeff set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
sigprocmask[6]: how=2 oldmask=0xfffffffc7fffffff set=0x0 newmask=0x0
sys_wait4: pid: -1, wstatus: fffffffffb1c, options: 0x2, rusage: 0
wait4: pid=6, wpid=2
- FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
sigprocmask[7]: how=2 oldmask=0xfffffffc7fffffff set=0x0 newmask=0x0
execve: proc[7] '/bin/myls' start, argc=1, envc=1
- FLT_TRANSLATION (pte): fault_addr: 0x423000, pte=0xffff00003af0d118, *pte=0x43
pf_handler: fault_addr: 0x423000, flt: 2

======= PROC[7]: /bin/dash TRAP FRAME DUMP =========
sp:    0x0000fffffffffa60  tpidr: 0x00000000004342f0
spsr:  0x0000000060000000  esr:   0x0000000056000000
elr:   0x000000000041bdf8  x0:    0x0000000000434b70
x1:    0x0000000000434b98  x2:    0x0000000000434ba8
x3:    0x0000000000432000  x4:    0x6b786c2e6d68612e
x5:    0x0000000000433ab8  x6:    0x2f2f2f2f2f2f2f2f
x7:    0x0000000000000027  x8:    0x00000000000000dd
x9:    0x0000000000000000  x10:   0x0000000000000000
x11:   0x0000fffffffffcb0  x12:   0x0000000000001030
x13:   0x3d6e702a203a6461  x14:   0x0000000000000001
x15:   0x0000000000000000  x16:   0x000000000040d5a8
x17:   0x0000000000000000  x18:   0x0000000000000000
x19:   0x0000000000434b70  x20:   0x0000000000434b98
x21:   0x00000000004291e0  x22:   0x0000000000434ba8
x23:   0x000000000041bdec  x24:   0x0000000000419be4
x25:   0x0000000000434ba8  x26:   0x0000000000000000
x27:   0x0000000000000001  x28:   0x0000000000000000
x29:   0x0000fffffffffa70  x30:   0x0000000000404c54
===================== DUMP END =====================
```

## 少し変更していたら

```
execve: proc[1] '/bin/init' start, argc=1, envc=0
start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bss=0x404108, brk=0x404790
entry=0x400268, bprm->p=0xfffffffffea0
init: starting dash
sigprocmask[1]: how=0 oldmask=0x0 set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
sigprocmask[1]: how=0 oldmask=0xfffffffc7fffffff set=0xffffffffffffffff newmask=0xffffffffffffffff
sigprocmask[1]: how=2 oldmask=0xffffffffffffffff set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
sigprocmask[6]: how=2 oldmask=0xffffffffffffffff set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
sigprocmask[1]: how=2 oldmask=0xfffffffc7fffffff set=0x0 newmask=0x0
sigprocmask[6]: how=2 oldmask=0xfffffffc7fffffff set=0x0 newmask=0x0
sys_wait4: pid: -1, wstatus: 0, options: 0x0, rusage: 0
wait4: pid=1, wpid=0
execve: proc[6] '/bin/dash' start, argc=1, envc=0
- [6] FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
start_code=0x400000, end_code=0x42fc70
start_data=0x431940, end_data=0x433320
bss=0x433320, brk=0x435100
entry=0x4004a0, bprm->p=0xfffffffffeb0
sigprocmask[6]: how=1 oldmask=0x0 set=0x300000000 newmask=0x0
sigaction[6]: signum=17 handler=0x4141a0 mask=0xfffffffc7fffffff flags=0x4000000
__expand_head: *pn=96, n=4096
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
- brk1=0x436000
sys_brk: n=0x437000, sz=0x436000
- ok return 0x437000
- brk2=0x437000, ret=0x436000
expand_heap: n=4096, p=0x0x436000, end=0x0
- w->psize=1, w->psize=0
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000000
436fe0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
- end=0x0x437000
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000fe1
436fe0: 0000000000000000 0000000000000000 0000000000000fe1 0000000000000001
sigaction[6]: signum=2 handler=0x4141a0 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=3 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=15 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=20 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=22 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=21 handler=0x0 mask=0xfffffffc7fffffff flags=0x4000000
# /bin/myls
sigprocmask[6]: how=2 oldmask=0xfffffffc7ffbfeff set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
sigprocmask[6]: how=2 oldmask=0xfffffffc7fffffff set=0x0 newmask=0x0
sys_wait4: pid: -1, wstatus: fffffffffb1c, options: 0x2, rusage: 0
wait4: pid=6, wpid=2
trap: CPU 0 unexpected irq: 0x20, iss: 0xf, sp: fffffffffb20.
Synchronous: Instruction abort, lower EL:
  ESR_EL1 0x8200000f ELR_EL1 0x432000
 SPSR_EL1 0x0 FAR_EL1 0x432000
	irq_error: irq of type 8 unimplemented.
kern/console.c:303: kernel panic at cpu 0.
```

## signal関係を修正

```
proc[1] > sys_execve
execve: proc[1] '/bin/init' start, argc=1, envc=0
== PRINT mmap_region[1] (mmap): head=0xffff00003afe0e60, nregions=1 ==
- [1] FLT_TRANSLATION (pte=0): fault_addr: 0x74696e6000, pte=0x0, *pte=0x580001a158000180
 - region[1]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=1476395425, offset=0x0
== PRINT mmap_region[1] (mmap): head=0xffff00003afe0e10, nregions=2 ==
 - region[1]: addr=0x400000, length=0x3000, prot=0x5, flags=0x812, f=22, offset=0x0
 - region[2]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[1] (mmap): head=0xffff00003afe0e10, nregions=3 ==
 - region[1]: addr=0x400000, length=0x3000, prot=0x5, flags=0x812, f=22, offset=0x0
 - region[2]: addr=0x403000, length=0x2000, prot=0x3, flags=0x12, f=22, offset=0x2000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
start_code=0x400000, end_code=0x402274
start_data=0x403eb0, end_data=0x404108
bss=0x404108, brk=0x404790
entry=0x400268, bprm->p=0xfffffffffea0
proc[1] > sys_gettid
proc[1] > sys_openat
proc[1] > sys_dup
proc[1] >> sys_dup
proc[1] > sys_ioctl
proc[1] > sys_writev
init: starting dash
proc[1] > sys_rt_sigprocmask
sigprocmask[1]: how=0 oldmask=0x0 set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
proc[1] >> sys_rt_sigprocmask
sigprocmask[1]: how=0 oldmask=0xfffffffc7fffffff set=0xffffffffffffffff newmask=0xffffffffffffffff
proc[1] > sys_clone
proc[6] > sys_gettid
proc[6] > sys_rt_sigprocmask
sigprocmask[6]: how=2 oldmask=0xffffffffffffffff set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
proc[1] >> sys_rt_sigprocmask
sigprocmask[1]: how=2 oldmask=0xffffffffffffffff set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
proc[1] >> sys_rt_sigprocmask
sigprocmask[1]: how=2 oldmask=0xfffffffc7fffffff set=0x0 newmask=0x0
proc[1] > sys_wait4
sys_wait4: pid: -1, wstatus: 0, options: 0x0, rusage: 0
wait4: pid=1, wpid=0
proc[6] > sys_rt_sigprocmask
sigprocmask[6]: how=2 oldmask=0xfffffffc7fffffff set=0x0 newmask=0x0
proc[6] > sys_execve
execve: proc[6] '/bin/dash' start, argc=1, envc=0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d70, nregions=1 ==
- [6] FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
 - region[1]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=2 ==
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=3 ==
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=4 ==
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0x434000, length=0x2000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
start_code=0x400000, end_code=0x42fc70
start_data=0x431940, end_data=0x433320
bss=0x433320, brk=0x435100
entry=0x4004a0, bprm->p=0xfffffffffeb0
proc[6] > sys_gettid
proc[6] > sys_getpid
proc[6] > sys_rt_sigprocmask
sigprocmask[6]: how=1 oldmask=0x0 set=0x300000000 newmask=0x0
proc[6] > sys_rt_sigaction
sigaction[6]: signum=17 handler=0x4141a0 mask=0xfffffffc7fffffff flags=0x4000000
proc[6] > sys_geteuid
proc[6] > sys_getppid
proc[6] > sys_ioctl
proc[6] > sys_writev
__expand_head: *pn=96, n=4096
proc[6] > sys_brk
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
proc[6] > sys_writev
- brk1=0x436000
proc[6] > sys_brk
sys_brk: n=0x437000, sz=0x436000
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=5 ==
 - region[1]: addr=0x400000, length=0x30000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0x434000, length=0x2000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0x436000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[5]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
- ok return 0x437000
proc[6] > sys_writev
- brk2=0x437000, ret=0x436000
proc[6] >> sys_writev
expand_heap: n=4096, p=0x0x436000, end=0x0
proc[6] >> sys_writev
- w->psize=1, w->psize=0
...
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000000
436fe0: 0000000000000000 0000000000000000 0000000000000000 0000000000000000
- end=0x0x437000
436000: 0000000000000000 0000000000000000 0000000000000001 0000000000000fe1
436fe0: 0000000000000000 0000000000000000 0000000000000fe1 0000000000000001
proc[6] > sys_getcwd
proc[6] > sys_ioctl
proc[6] >> sys_ioctl
proc[6] > sys_rt_sigaction
proc[6] >> sys_rt_sigaction
sigaction[6]: signum=2 handler=0x4141a0 mask=0xfffffffc7fffffff flags=0x4000000
proc[6] >> sys_rt_sigaction
...
sigaction[6]: signum=3 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=15 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
proc[6] > sys_openat
proc[6] > sys_fcntl
proc[6] > sys_close
proc[6] > sys_fcntl
proc[6] > sys_ioctl
proc[6] > sys_getpgid
proc[6] > sys_rt_sigaction
proc[6] >> sys_rt_sigaction
sigaction[6]: signum=20 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
proc[6] >> sys_rt_sigaction
...
sigaction[6]: signum=22 handler=0x1 mask=0xfffffffc7fffffff flags=0x4000000
sigaction[6]: signum=21 handler=0x0 mask=0xfffffffc7fffffff flags=0x4000000
proc[6] > sys_setpgid
proc[6] > sys_ioctl
proc[6] > sys_write
proc[6] > sys_read
# /bin/myls
proc[6] > sys_rt_sigprocmask
sigprocmask[6]: how=2 oldmask=0xfffffffc7ffbfeff set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
proc[6] > sys_clone
proc[6] > sys_rt_sigprocmask
sigprocmask[6]: how=2 oldmask=0xfffffffc7fffffff set=0x0 newmask=0x0
proc[6] > sys_setpgid
proc[6] > sys_wait4
sys_wait4: pid: -1, wstatus: fffffffffb1c, options: 0x2, rusage: 0
wait4: pid=6, wpid=2
proc[7] >> sys_wait4
sys_wait4: pid: -1, wstatus: fffffffffb1c, options: 0x2, rusage: 0
wait4: pid=7, wpid=2
- [7] FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
- [7] FLT_TRANSLATION (pte=0): fault_addr: 0xffffffff00434000, pte=0x0, *pte=0x0
proc[7] > sys_write
proc[7] > sys_read
# /bin/myls
proc[7] > sys_rt_sigprocmask
sigprocmask[7]: how=2 oldmask=0xfffffffc7fffffff set=0xfffffffc7fffffff newmask=0xfffffffc7fffffff
proc[7] > sys_clone
proc[7] > sys_rt_sigprocmask
sigprocmask[7]: how=2 oldmask=0xfffffffc7fffffff set=0x0 newmask=0x0
proc[7] > sys_setpgid
proc[7] > sys_wait4
sys_wait4: pid: -1, wstatus: fffffffffb1c, options: 0x2, rusage: 0
wait4: pid=7, wpid=2
- [8] FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, *pte=0x0
proc[8] > sys_setpgid
proc[8] > sys_ioctl         // ここでストール
```

## mallocngに再度変更

- mmap()でMAP_FIXEDの場合を修正
- malloc関係は動くようになった
- `sh: 0:`という謎のエラーでハングアップ

```
execve: proc[6] '/bin/dash' start, argc=1, envc=0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d70, nregions=1 ==
 - region[1]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
- [6] FLT_TRANSLATION (pte=0): fault_addr: 0x0, pte=0x0, addr=0x0, flags=0x0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=2 ==
 - region[1]: addr=0x400000, length=0x31000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=3 ==
 - region[1]: addr=0x400000, length=0x31000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=4 ==
 - region[1]: addr=0x400000, length=0x31000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0x434000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
start_code=0x400000, end_code=0x4301b0
start_data=0x431940, end_data=0x433320
bss=0x433320, brk=0x434e80
entry=0x4004a0, bprm->p=0xfffffffffeb0
sigprocmask[6]: how=1 oldmask=0x0 set=0x300000000 newmask=0x0
sigaction[6]: signum=17 handler=0x4141a0 mask=0xfffffffc7fffffff flags=0x4000000
sys_brk: n=0x0, sz=0x435000
- invalid n 0 return 0x435000
brk1: 0x435000
new: 0x437000
sys_brk: n=0x437000, sz=0x435000
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=5 ==
 - region[1]: addr=0x400000, length=0x31000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0x434000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0x435000, length=0x2000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[5]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
- ok return 0x437000
brk2 == new & call mmap: addr=0x435000, length=0x1000
sys_mmap: addr=0x435000, length=0x1000, prot=0x0, flags=0x32, f=0, offset=0x0
munmap: length before=0x1000, after=0x1000
- munmap: addr=0x435000, length=0x1000
- found: addr=0x435000
- new region: addr=0x436000, length=0x1000
munmap ok
== PRINT mmap_region[6] (munmap): head=0xffff00003afe0d20, nregions=5 ==
 - region[1]: addr=0x400000, length=0x31000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0x434000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0x436000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[5]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=6 ==
 - region[1]: addr=0x400000, length=0x31000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0x434000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0x435000, length=0x1000, prot=0x0, flags=0x32, f=0, offset=0x0
 - region[5]: addr=0x436000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[6]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
- return 0x435000
alloc_group: call mmap: addr=0, length=0x1000
sys_mmap: addr=0x0, length=0x1000, prot=0x3, flags=0x22, f=0, offset=0x0
== PRINT mmap_region[6] (mmap): head=0xffff00003afe0d20, nregions=7 ==
 - region[1]: addr=0x400000, length=0x31000, prot=0x5, flags=0x812, f=144, offset=0x0
 - region[2]: addr=0x431000, length=0x3000, prot=0x3, flags=0x12, f=144, offset=0x30000
 - region[3]: addr=0x434000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[4]: addr=0x435000, length=0x1000, prot=0x0, flags=0x32, f=0, offset=0x0
 - region[5]: addr=0x436000, length=0x1000, prot=0x3, flags=0x32, f=0, offset=0x0
 - region[6]: addr=0x437000, length=0x1000, prot=0x3, flags=0x22, f=0, offset=0x0
 - region[7]: addr=0xfffffffe0000, length=0x20000, prot=0x7, flags=0x32, f=0, offset=0x0
- return 0x437000
sh: 0:
```

## `sh: 0`はdash:error.cのexvwarning2()関数で出力されている

- これはエラー出力だが、なぜエラーになっているかがわからないのでoldmallocに戻す

## stack用のmmapのpaがおかしい

- dash[6]がmyls[7]からのexitコードを収めるための物理ページがすでに削除されたはずのfork時にdashからmylsにコピーされたものになっている

```
iinit: ok
init log[1]: ok
execve: proc[1] '/bin/init' start, argc=1, envc=0
setup_arg call: addr=0xfffffffe0000, length=0x20000
- map_region: pgdir=0xffff00003affd000, va=0xfffffffff000, pte=0xffff00003afdbff8, pa=0x3afbc000    //  (1) setup_arg_pages p[1]: pa=0x3afbc000
mmaped: pgdir=0xffff00003affd000, addr=0xfffffffff000, pa=0x3afbc000
cmp[1]: addr=0xfffffffff000, pa=0x3af9c000                                                          //  (2) FLT_PERMISSION  p[1]: pa=0x3af9c000
init: starting dash
- map_region: pgdir=0xffff00003af93000, va=0xfffffffff000, pte=0xffff00003af8dff8, pa=0x3af9c000    //  (3) fork 1 to 6     p[6]: pa=0x3af9c000
cml: parent[1] pgdir=0xffff00003affd000, pa=0x3af9c000
cml: child [6] pgdir=0xffff00003af93000, pa=0x3af9c000
fork: old=1, new=6
cmp[6]: addr=0xfffffffff000, pa=0x3af6b000                                                          //  (4) FLT_PERMISSION  p[6]: pa=0x3af6b000
execve: proc[6] '/bin/dash' start, argc=1, envc=0
sys_wait4: current: 1, pid: -1, wstatus: 0x0, options: 0x0, rusage: 0x0
- wait4: current=1, current->pgdir=0xffff00003affd000, pid=-1
setup_arg call: addr=0xfffffffe0000, length=0x20000
- map_region: pgdir=0xffff00003af93000, va=0xfffffffff000, pte=0xffff00003af8dff8, pa=0x3af8a000    //  (5) setup_arg_pages p[6]: pa=0x3af8a000
mmaped: pgdir=0xffff00003af93000, addr=0xfffffffff000, pa=0x3af8a000
cmp[6]: addr=0xfffffffff000, pa=0x3af4f000                                                          //  (6) FLT_PERMISSION  p[6]: pa=0x3af4f000
# /bin/myls
- map_region: pgdir=0xffff00003af10000, va=0xfffffffff000, pte=0xffff00003af0aff8, pa=0x3af4f000    //  (7) fork 6 to 7     p[7]: pa=0x3af4f000
cml: parent[6] pgdir=0xffff00003af93000, pa=0x3af4f000
cml: child [7] pgdir=0xffff00003af10000, pa=0x3af4f000
fork: old=6, new=7
cmp[7]: addr=0xfffffffff000, pa=0x3aeea000                                                          //  (8) FLT_PERMISSION  p[7]: pa=0x3aeea000
musl call wait4: pid=-1, status=0xfffffffffb1c, dest=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
- addr=0xfffffffff000, pa=0x3af4f000                                                                //  (9) wstatus         p[6]: pa=0x3af4f000
- wait4: current=6, current->pgdir=0xffff00003af93000, pid=-1
execve: proc[7] '/bin/myls' start, argc=1, envc=1
setup_arg call: addr=0xfffffffe0000, length=0x20000
- map_region: pgdir=0xffff00003af10000, va=0xfffffffff000, pte=0xffff00003af0aff8, pa=0x3af09000    // (10) setup_arg_pages p[7]: pa=0x3af09000
mmaped: pgdir=0xffff00003af10000, addr=0xfffffffff000, pa=0x3af09000
cmp[7]: addr=0xfffffffff000, pa=0x3af37000                                                          // (11) FLT_PERMISSION  p[7]: pa=0x3af37000

sys_exit: proc[7], status=123
copyout: pgdir=ffff00003af93000, va=0xfffffffffb1c, src=0xffff00000011a734, len=0x4
- uva_to_ka: pgdir=0xffff00003af93000, va=0xfffffffff000, ka=0x3af4f000                             // (11) copyout         p[6]: ka=0x3af4f000
- - return 7, status=123
- wait4 return 7, status=123
```

## trap: translation: *, access: #, permission: $
```
init log[1]: ok
execve: proc[1] '/bin/init' start, argc=1, envc=0
setup_arg call: addr=0xfffffffe0000, length=0x20000
- map_region: pgdir=0xffff00003affd000, va=0xfffffffff000, pte=0xffff00003afdbff8, pa=0x3afbc000
mmaped: pgdir=0xffff00003affd000, addr=0xfffffffff000, pa=0x3afbc000
$ cmp[1]: addr=0xfffffffff000, pa=0x3af9c000
$ init: starting dash
- map_region: pgdir=0xffff00003af93000, va=0xfffffffff000, pte=0xffff00003af8dff8, pa=0x3af9c000
cml: parent[1] pgdir=0xffff00003affd000, pa=0x3af9c000
cml: child [6] pgdir=0xffff00003af93000, pa=0x3af9c000
fork: old=1, new=6
$$ cmp[6]: addr=0xfffffffff000, pa=0x3af6b000
sys_wait4[1]: pid: -1, wstatus: 0x0, options: 0x0, rusage: 0x0
- - wait4: current=1, current->pgdir=0xffff00003affd000, pid=-1
execve: proc[6] '/bin/dash' start, argc=1, envc=0
setup_arg call: addr=0xfffffffe0000, length=0x20000
- map_region: pgdir=0xffff00003af93000, va=0xfffffffff000, pte=0xffff00003af8dff8, pa=0x3af8a000
mmaped: pgdir=0xffff00003af93000, addr=0xfffffffff000, pa=0x3af8a000
$ cmp[6]: addr=0xfffffffff000, pa=0x3af4f000
*$$ __expand_head: *pn=96, n=4096
sys_brk: n=0x0, sz=0x436000
- invalid n 0 return 0x436000
- brk1=0x436000
sys_brk: n=0x437000, sz=0x436000
- ok return 0x437000
- brk2=0x437000, ret=0x436000
$ # /bin/myls
- map_region: pgdir=0xffff00003af10000, va=0xfffffffff000, pte=0xffff00003af0aff8, pa=0x3af4f000
cml: parent[6] pgdir=0xffff00003af93000, pa=0x3af4f000
cml: child [7] pgdir=0xffff00003af10000, pa=0x3af4f000
fork: old=6, new=7
$ cmp[7]: addr=0xfffffffff000, pa=0x3aeea000
musl call wait4: pid=-1, status=0xfffffffffb1c, dest=0
sys_wait4[6]: pid: -1, wstatus: 0xfffffffffb1c, options: 0x2, rusage: 0x0
- addr=0xfffffffff000, pa=0x3af4f000
- - wait4: current=6, current->pgdir=0xffff00003af93000, pid=-1
$* $$execve: proc[7] '/bin/myls' start, argc=1, envc=1
setup_arg call: addr=0xfffffffe0000, length=0x20000
- map_region: pgdir=0xffff00003af10000, va=0xfffffffff000, pte=0xffff00003af0aff8, pa=0x3af09000
mmaped: pgdir=0xffff00003af10000, addr=0xfffffffff000, pa=0x3af09000
$ cmp[7]: addr=0xfffffffff000, pa=0x3af37000
$$ drwxrwxr-x   1       4096 .
drwxrwxr-x   1       4096 ..
drwxrwxr-x   2       8448 bin
drwxrwxr-x   3        384 dev
drwxrwxr-x   8        320 etc
drwxrwxrwx   9        192 lib
drwxrwxr-x  10        192 home
drwxrwxr-x  12        192 usr
sys_exit: proc[7], status=123
copyout: pgdir=ffff00003af93000, va=0xfffffffffb1c, src=0xffff00000011b734, len=0x4
- uva_to_ka: pgdir=0xffff00003af93000, va=0xfffffffff000, ka=0x3af4f000
- - return 7, status=123
- wait4 return 7, status=123
```

## region->ref_countでcopy_mmapを調整中

- usertop(=stacktop = mmaptop)を1<44に変更

```
init log[1]: ok
execve: proc[1] '/bin/init' start, argc=1, envc=0
setup_arg call: addr=0xffffffe0000, length=0x20000
- mmap[1]: region->addr=0xffffffe0000, ref=0 => 0
- mmap load_pages[1]: region->addr=0xffffffe0000, ref=0 => 1
- mmap[1]: region->addr=0x400000, ref=0 => 0
- mmap load_pages[1]: region->addr=0x400000, ref=0 => 1
- mmap[1]: region->addr=0x403000, ref=0 => 0
- mmap load_pages[1]: region->addr=0x403000, ref=0 => 1 // ↓ fault_addrがカーネルアドレス
FLT_TRANSLATION[1]: fault: oaddr=0xffffffffffffffe0, alignedr=0xfffffffffffff000  // fault_addr = -32
FLT_TRANSLATION[1]: cursor->addr=0x400000, ref=1
FLT_TRANSLATION[1]: cursor->addr=0x403000, ref=1
FLT_TRANSLATION[1]: cursor->addr=0xffffffe0000, ref=1
pf_handler[1]: fault_addr: 0xfffffffffffff000, flt: 1
```
