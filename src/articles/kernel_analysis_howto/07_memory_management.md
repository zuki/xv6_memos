# 7. Linuxメモリ管理

## 7.1 概要

Linuxはセグメンテーションとページネーションを使用します。

### セグメント

Linuxは4つのセグメントしか使用していません。

- [0xC000 0000] (3 GB)から[0xFFFF FFFF] (4 GB)までのカーネル空間として使用する
  2つのセグメント（codeとdata/stack)
- [0] (0 GB)から[0xBFFF FFFF] (3 GB)までのユーザ空間として使用する2つの
  セグメント（codeとdata/stack)

```
                               __
   4 GB--->|                |    |
           |     Kernel     |    |  カーネル空間 (Code + Data/Stack)
           |                |  __|
   3 GB--->|----------------|  __
           |                |    |
           |                |    |
   2 GB--->|                |    |
           |     Tasks      |    |  ユーザ空間 (Code + Data/Stack)
           |                |    |
   1 GB--->|                |    |
           |                |    |
           |________________|  __|
 0x00000000
          カーネル/ユーザリニアアドレス
```

## 7.2 i386固有の実装

Linuxは3レベルのページングを使用するページネーションを実装していますが、i386
アーキテクチャでは実際にはそのうち2レベルしか使用していません。

```
   ------------------------------------------------------------------
   L    I    N    E    A    R         A    D    D    R    E    S    S
   ------------------------------------------------------------------
        \___/                 \___/                     \_____/

     PD offset              PF offset                 Frame offset
     [10 bits]              [10 bits]                 [12 bits]
          |                     |                          |
          |                     |     -----------          |
          |                     |     |  Value  |----------|---------
          |     |         |     |     |---------|   /|\    |        |
          |     |         |     |     |         |    |     |        |
          |     |         |     |     |         |    | Frame offset |
          |     |         |     |     |         |   \|/             |
          |     |         |     |     |---------|<------            |
          |     |         |     |     |         |      |            |
          |     |         |     |     |         |      | x 4096     |
          |     |         |  PF offset|_________|-------            |
          |     |         |       /|\ |         |                   |
      PD offset |_________|-----   |  |         |          _________|
            /|\ |         |    |   |  |         |          |
             |  |         |    |  \|/ |         |         \|/
 _____       |  |         |    ------>|_________|       物理アドレス
|     |     \|/ |         |    x 4096 |         |
| CR3 |-------->|         |           |         |
|_____|         | ....... |           | ....... |
                |         |           |         |

               ページディレクトリ          ページファイル

                       Linux i386 Paging
```


## 7.3 メモリマッピング

Linuxではアクセス制御をページネーションだけで管理しているので異なるタスクが同じ
セグメントアドレスを持つことがあります。しかし、各タスクは異なるCR3（ディレクトリ
ページアドレスを格納するレジスタ）を持っているので異なるページエントリを指すことに
なります。

ユーザモードではタスクは3GBの制限（`0xC0 00 00 00`）を越えることができないので
最初の768個のページディレクトリエントリだけが意味を持ちます（768*4MB = 3GB）。

タスクが（システムコールまたはIRQによって）カーネルモードになると残りの256個のページ
ディレクトリエントリが重要になり、それらはすべてのタスクで同じページファイルを
指します（カーネルと同じです）。

カーネル（そしてカーネルだけ）のリニア空間はカーネルの物理空間に等しい
ことに注意してください、したがって、次のようになります。

```
            ________________ _____
           |Other KernelData|___  |  |                |
           |----------------|   | |__|                |
           |     Kernel     |\  |____|   Real Other   |
  3 GB --->|----------------| \      |   Kernel Data  |
           |                |\ \     |                |
           |              __|_\_\____|__   Real       |
           |      Tasks     |  \ \   |     Tasks      |
           |              __|___\_\__|__   Space      |
           |                |    \ \ |                |
           |                |     \ \|----------------|
           |                |      \ |Real KernelSpace|
           |________________|       \|________________|

           Logical Addresses          Physical Addresses
```

リニアカーネル空間は3GB下に変換された物理カーネル空間に対応します（実際、ページテーブルは
{"00000000", "00000001"}のようなものであり、仮想化は行わず、リニアから取得した物理アドレス
をそのまま報告します）。

物理アドレスはページテーブルで管理できるのでカーネル空間とユーザ空間の間で
「アドレスの衝突」は発生しないことに注意してください。

## 7.4 低レベルメモリ割り当て

### ブート初期化

`kmem_cache_init`（ブート時に`start_kernel`[init/main.c]により起動されます)から
開始します。

```
|kmem_cache_init
   |kmem_cache_estimate
```

- kmem_cache_init [mm/slab.c]
- kmem_cache_estimate

`mem_init`（これも`start_kernel`[init/main.c]により起動されます）が続きます。

```
|mem_init
   |free_all_bootmem
      |free_all_bootmem_core
```

- mem_init [arch/i386/mm/init.c]
- free_all_bootmem [mm/bootmem.c]
- free_all_bootmem_core

### 実行時割り当て

Linuxでは、たとえば「コピーオンライト」機構（10章参照）などでメモリを割り当てたい場合、
次の呼び出しを行います。

```
|copy_mm
   |allocate_mm = kmem_cache_alloc
      |__kmem_cache_alloc
         |kmem_cache_alloc_one
            |alloc_new_slab
               |kmem_cache_grow
                  |kmem_getpages
                     |__get_free_pages
                        |alloc_pages
                           |alloc_pages_pgdat
                              |__alloc_pages
                                 |rmqueue
                                 |reclaim_pages
```

これらの関数は各々以下に存在します。

- copy_mm [kernel/fork.c]
- allocate_mm [kernel/fork.c]
- kmem_cache_alloc [mm/slab.c]
- __kmem_cache_alloc
- kmem_cache_alloc_one
- alloc_new_slab
- kmem_cache_grow
- kmem_getpages
- __get_free_pages [mm/page_alloc.c]
- alloc_pages [mm/numa.c]
- alloc_pages_pgdat
- __alloc_pages [mm/page_alloc.c]
- rm_queue
- reclaim_pages [mm/vmscan.c]

TODO: Zoneの理解

## 7.5 スワップ

### 概要

スワップは`kswapd`デーモン（カーネルスレッド）により管理されています。

### kswapd

他のカーネルスレッドと同様にkswapdも起床するまでwaitするメインループを持ちます。

```
|kswapd
   |// 初夏化ルーチン
   |for (;;) { // メインループ
      |do_try_to_free_pages
      |recalculate_vm_stats
      |refill_inactive_scan
      |run_task_queue
      |interruptible_sleep_on_timeout // 新規スワップリクエストが来るまでスリープ
   |}
```

- kswapd [mm/vmscan.c]
- do_try_to_free_pages
- recalculate_vm_stats [mm/swap.c]
- refill_inactive_scan [mm/vmswap.c]
- run_task_queue [kernel/softirq.c]
- interruptible_sleep_on_timeout [kernel/sched.c]

### スワッピングが必要となるのは何時か

スワッピングは物理メモリにないページにアクセスしなければならない時に
必要になります。

この目的を実現するためにLinuxは`kswapd`カーネルスレッドを使用します。
タスクがページフォルト例外を受け取ると以下を実行します。

```
 | ページフォルト例外
 | 以下の3条件がすべて満たされると実行:
 |   a-) ユーザページ
 |   b-) 読み込みまたは書き込みアクセス
 |   c-) ページが存在しない
 |
 |
 -----------> |do_page_fault
                 |handle_mm_fault
                    |pte_alloc
                       |pte_alloc_one
                          |__get_free_page = __get_free_pages
                             |alloc_pages
                                |alloc_pages_pgdat
                                   |__alloc_pages
                                      |wakeup_kswapd // カーネルスレッドkswapdを起こす

                   Page Fault ICA
````

- do_page_fault [arch/i386/mm/fault.c]
- handle_mm_fault [mm/memory.c]
- pte_alloc
- pte_alloc_one [include/asm/pgalloc.h]
- __get_free_page [include/linux/mm.h]
- __get_free_pages [mm/page_alloc.c]
- alloc_pages [mm/numa.c]
- alloc_pages_pgdat
- __alloc_pages
- wakeup_kswapd [mm/vmscan.c]
