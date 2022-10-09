# 6. Linuxのマルチタスク

## 6.1 概要

この章ではLinuxのマルチタスク環境を管理する組みであるデータ構造体を分析します。

### タスクの状態

Linuxのタスクは次のいずれかの状態になることができます（[include/linux.h]による）。

1. TASK_RUNNING: "Ready List"にあることを意味します
2. TASK_INTERRUPTIBLE: シグナルやリソースを待っているタスク（スリーピング）
3. TASK_UNINTERRUPTIBLE: リソースを待っているタスク（スリーピング）。同じ"Wait Queue"にあります
4. TASK_ZOMBIE: 親を持たない子タスク
5. TASK_STOPPED: デバッグ中のタスク

### 状態繊維図

```
       ______________     CPU Available     ______________
      |              |  ---------------->  |              |
      | TASK_RUNNING |                     | Real Running |
      |______________|  <----------------  |______________|
                           CPU Busy
            |   /|\
Waiting for |    | Resource
 Resource   |    | Available
           \|/   |
    ______________________
   |                      |
   | TASK_INTERRUPTIBLE / |
   | TASK-UNINTERRUPTIBLE |
   |______________________|

                     Main Multitasking Flow
```

## 6.2 タイムスライス

### PIT 8253プログラミング

10msごとに（HZの値による）IRQ0が上がるのでマルチタスク環境で利用できます。このシグナルは
PIC 8259（arch 386+の場合）から来ますが、これは1.19318MHzのクロックを持つPIC 8253に接続
されています。

```
    _____         ______        ______
   | CPU |<------| 8259 |------| 8253 |
   |_____| IRQ0  |______|      |___/|\|
                                    |_____ CLK 1.193.180 MHz

// include/asm/param.h より
#ifndef HZ
#define HZ 100
#endif

// include/asm/timex.h より
#define CLOCK_TICK_RATE 1193180                 /* 元になるHZ */

// include/linux/timex.h より
#define LATCH ((CLOCK_TICK_RATE + HZ/2) / HZ)   /* ドライバ用 */

// arch/i386/kernel/i8259.c より
outb_p(0x34,0x43);                              /* binary, mode 2, LSB/MSB, ch 0 */
outb_p(LATCH & 0xff , 0x40);                    /* LSB */
outb(LATCH >> 8 , 0x40);                        /* MSB */
```

そこで、8253（PIT: Programmable Interval Timer）を`HZ=100`（デフォルト）の時は
`LATCH =（1193180/HZ）= 11931.8`とプログラムします。LATCHは周波数除算係数を表します。

`LATCH = 11931.8 `は8253（の出力）に`1193180 / 11931.8 = 100 Hz`の周波数を与えるので
`周期 = 10ms`となります。

それで`タイムスライス＝ 1/HZ`となります。

タイムスライスごとに現在のプロセスの実行を一時的に中断し（タスクスイッチなし）、
ハウスキーピング作業を行い、その後、以前のプロセスに戻ります。

### LinuxのタイマーIRQ ICA

```
Linux Timer IRQ
IRQ 0 [Timer]
 |
\|/
|IRQ0x00_interrupt        //   wrapper IRQ handler
   |SAVE_ALL              ---
      |do_IRQ                |   wrapper routines
         |handle_IRQ_event  ---
            |handler() -> timer_interrupt    // registered IRQ 0 handler
               |do_timer_interrupt
                  |do_timer
                     |jiffies++;
                     |update_process_times
                     |if (--counter <= 0) {  // if time slice ended then
                        |counter = 0;        //   reset counter
                        |need_resched = 1;   //   prepare to reschedule
                     |}
         |do_softirq
         |while (need_resched) {             // if necessary
            |schedule                        //   reschedule
            |handle_softirq
         |}
   |RESTORE_ALL
```

各関数は以下にあります。

- IRQ0x00_interrupt, SAVE_ALL [include/asm/hw_irq.h]
- do_IRQ, handle_IRQ_event [arch/i386/kernel/irq.c]
- timer_interrupt, do_timer_interrupt [arch/i386/kernel/time.c]
- do_timer, update_process_times [kernel/timer.c]
- do_softirq [kernel/soft_irq.c]
- RESTORE_ALL, while loop [arch/i386/kernel/entry.S]

注:

1. 関数`IRQ0x00_interrupt`は（他の`IRQ0xXY_interrupt`と同様）。IDT（割込み記述子テーブル、
   リアルモード割込みベクタテーブルと同様、詳細は11章を参照）により直接ポイントされており、
   プロセッサに来るすべての割り込みは`IRQ0x#NR_interrupt`ルーチン（#NRは割込み番号）により
   管理されます。これを「ラッパーIRQハンドラ」と呼ばれます。
2. wrapperルーチンが`do_IRQ`や`handle_IRQ_event` [arch/i386/kernel/irq.c] などと同じように
   実行されます。
3. この後、制御は`request_irq`[arch/i386/kernel/irq.c]で事前に登録された（`handler()`に
   ポイントされた)公式のIRQルーチン、今回の場合は`timer_interrupt` [arch/i386/kernel/time.c] に
   渡されます。
4. `timer_interrupt` [arch/i386/kernel/time.c] ルーチンが実行され、それが終了すると
    制御はアセンブラルーチン [arch/i386/kernel/entry.S] に戻されます。

説明

マルチタスクを管理するためにLinuxは（他のすべてのUnixと同様）は`counter`変数を使って
タスクがどれだけCPUを使ったかを記録しています。そのため、IRQ0が発生するたびにカウンタは
デクリメントされ（ポイント4）、0になったらタスクを切り替えてタイムシェアリングを管理する
必要があります（ポイント4で`need_resched`変数には1がセットされ、ポイント5でアセンブラ
ルーチンが`need_resched`を制御して、必要に応じて`schedule`を呼び出します [kernel/sched.c])．

## 6.3 スケジューラ

スケジューラは指定された時間に実行されるべきタスクを選択するコードです。

実行タスクを変更する必要がある時はいつでも候補を選択することができます。以下は'`schedule`
[kernel/sched.c]関数です。

```
|schedule
   |do_softirq             // manages post-IRQ work
   |for each task
      |calculate counter
   |prepare_to__switch     // does anything
   |switch_mm              // change Memory context (change CR3 value)
   |switch_to (assembler)
      |SAVE ESP
      |RESTORE future_ESP
      |SAVE EIP
      |push future_EIP *** push parameter as we did a call
         |jmp __switch_to (it does some TSS work)
         |__switch_to()
          ..
         |ret *** ret from call using future_EIP in place of call address
      new_task
```

## 6.4 ボトムハーフ、タスクキュー、タスクレット

### 概要

古典的なUnixではIRQが（デバイスから）来るとUnixは「タスクスイッチ」を行い、デバイスを
要求したタスクに問い合わせをします。

Linuxではパフォーマンスを向上させるために緊急ではない作業を後回しにすることで高速な
イベントをうまく管理しています。

この機能はカーネル1.xから「ボトムハーフ」(BH) によって管理されています。irqハンドラは
スケジューリングされると後で実行するようにボトムハーフに「マーク」します。

最新のカーネルではBHよりもダイナミックな「タスクキュー」と、マルチプロセッサ環境を
管理する「タスクレット」があります。

BHのスキーマは次のとおりです。

1. 宣言
2. マーク
3. 実行

### 宣言

```c
#define DECLARE_TASK_QUEUE(q) LIST_HEAD(q)
#define LIST_HEAD(name) \
   struct list_head name = LIST_HEAD_INIT(name)
struct list_head {
   struct list_head *next, *prev;
};
#define LIST_HEAD_INIT(name) { &(name), &(name) }
```

- ''DECLARE_TASK_QUEUE''[include/linux/tqueue.h, include/linux/list.h]

`DECLARE_TASK_QUEUE(q)`マクロはタスクキューを管理する"q"という名前の構造体を宣言する
のに使用されます。

### マーク

以下は"mark_bh" [include/linux/interrupt.h] 関数のICAです。

```
|mark_bh(NUMBER)
   |tasklet_hi_schedule(bh_task_vec + NUMBER)
      |insert into tasklet_hi_vec
         |__cpu_raise_softirq(HI_SOFTIRQ)
            |soft_active |= (1 << HI_SOFTIRQ)
```
- ''mark_bh''[include/linux/interrupt.h]

たとえば、IRQハンドラが何らかの処理を「後回し」にしたい場合は"mark_bh(NUMBER)"とします。
ここで、NUMBERは宣言したBHです（前の節を参照）。

### 実行

この呼出は"do_IRQ" [arch/i386/kernel/irq.c] 関数にあります。

```
|do_softirq
   |h->action(h)-> softirq_vec[TASKLET_SOFTIRQ]->action -> tasklet_action
      |tasklet_vec[0].list->func
```

"h->action(h);"は事前にキューイングされている関数です。

## 6.5 非常に低レベルなルーチン

```
set_intr_gate

set_trap_gate

set_task_gate (not used).

(*interrupt)[NR_IRQS](void) = { IRQ0x00_interrupt, IRQ0x01_interrupt, ..}

NR_IRQS = 224 [kernel 2.4.2]
```

## 6.6 タスクスイッチ

### タスクスイッチはいつ行われるか?

Linuxカーネルがあるタスクから別のタスクにどのようにスイッチするかを見てみます。

タスクスイッチは次のように多くの場面で必要になります。

1. タイムスライスが終了した時: 他のタスクにアクセスを与える必要があります。
2. タスクがリソースにアクセスすると決めた時: リソースを待ってスリープするので別の
   タスクを選択する必要があります。
3. タスクがパイプを待つ時: パイプに書き込む他のタスクにアクセスを与える必要があります。


### タスクスイッチ

```
                           TASK SWITCHING TRICK
#define switch_to(prev,next,last) do {                                  \
        asm volatile("pushl %%esi\n\t"                                  \
                     "pushl %%edi\n\t"                                  \
                     "pushl %%ebp\n\t"                                  \
                     "movl %%esp,%0\n\t"        /* save ESP */          \
                     "movl %3,%%esp\n\t"        /* restore ESP */       \
                     "movl $1f,%1\n\t"          /* save EIP */          \
                     "pushl %4\n\t"             /* restore EIP */       \
                     "jmp __switch_to\n"                                \
                     "1:\t"                                             \
                     "popl %%ebp\n\t"                                   \
                     "popl %%edi\n\t"                                   \
                     "popl %%esi\n\t"                                   \
                     :"=m" (prev->thread.esp),"=m" (prev->thread.eip),  \
                      "=b" (last)                                       \
                     :"m" (next->thread.esp),"m" (next->thread.eip),    \
                      "a" (prev), "d" (next),                           \
                      "b" (prev));                                      \
} while (0)
```

トリックは次の箇所です。

1. ''pushl %4'' これはfuture_EIPをスタックに置きます
2. ''jmp __switch_to'' これは ''__switch_to'' 関数を実行しますが、''call'' とは
   反対に、1項でプッシュした値に戻ります（それで新しいタスクになります）

```
      U S E R   M O D E                 K E R N E L     M O D E

 |          |     |          |       |          |     |          |
 |          |     |          | Timer |          |     |          |
 |          |     |  Normal  |  IRQ  |          |     |          |
 |          |     |   Exec   |------>|Timer_Int.|     |          |
 |          |     |     |    |       | ..       |     |          |
 |          |     |    \|/   |       |schedule()|     | Task1 Ret|
 |          |     |          |       |_switch_to|<--  |  Address |
 |__________|     |__________|       |          |  |  |          |
                                     |          |  |S |          |
Task1 Data/Stack   Task1 Code        |          |  |w |          |
                                     |          | T|i |          |
                                     |          | a|t |          |
 |          |     |          |       |          | s|c |          |
 |          |     |          | Timer |          | k|h |          |
 |          |     |  Normal  |  IRQ  |          |  |i |          |
 |          |     |   Exec   |------>|Timer_Int.|  |n |          |
 |          |     |     |    |       | ..       |  |g |          |
 |          |     |    \|/   |       |schedule()|  |  | Task2 Ret|
 |          |     |          |       |_switch_to|<--  |  Address |
 |__________|     |__________|       |__________|     |__________|

Task2 Data/Stack   Task2 Code        Kernel Code  Kernel Data/Stack
```

## 6.7 フォーク

### 概要

フォークは新しいタスクの作成に使用されます。まず、親タスクから開始し、子タスクに
多くのデータ構造をコピーします。

```
                               |         |
                               | ..      |
         Task Parent           |         |
         |         |           |         |
         |  fork   |---------->|  CREATE |
         |         |          /|   NEW   |
         |_________|         / |   TASK  |
                            /  |         |
             ---           /   |         |
             ---          /    | ..      |
                         /     |         |
         Task Child     /
         |         |   /
         |  fork   |<-/
         |         |
         |_________|

                       Fork SysCall
```

### コピーされないものはなにか

作成されたばかりの新しいタスク (`Task Child`) は親 (`Task Parent`)とほとんど同じ
ですが、次のように少しだけ違いがあります。

1. 明らかに PID
2. ユーザモードで親子を区別するために、子の`fork()`は0を返しますが、親の
   `fork()`は子タスクのPIDを返します。
3. 子のデータページはすべて`READ + EXECUTE`とマークされ、`WRITE`は付きません
   （一方、親は自分のページに対してWRITE権限を持っています）。そのため、書き込み
   要求が来ると「ページフォルト」例外が生成され、これにより新しい個別のページが
   作成されます。この仕組は「コピーオンライト」と呼ばれます（詳細は10章を参照）。

### Fork ICA

```
|sys_fork
   |do_fork
      |alloc_task_struct
         |__get_free_pages
       |p->state = TASK_UNINTERRUPTIBLE
       |copy_flags
       |p->pid = get_pid
       |copy_files
       |copy_fs
       |copy_sighand
       |copy_mm                           // should manage CopyOnWrite (I part)
          |allocate_mm
          |mm_init
             |pgd_alloc -> get_pgd_fast
                |get_pgd_slow
          |dup_mmap
             |copy_page_range
                |ptep_set_wrprotect
                   |clear_bit             // set page to read-only
          |copy_segments                  // For LDT
       |copy_thread
          |childregs->eax = 0
          |p->thread.esp = childregs      // child fork returns 0
          |p->thread.eip = ret_from_fork  // child starts from fork exit
       |retval = p->pid // parent fork returns child pid
       |SET_LINKS // insertion of task into the list pointers
       |nr_threads++ // Global variable
       |wake_up_process(p) // Now we can wake up just created child
       |return retval

               fork ICA
```

- sys_fork [arch/i386/kernel/process.c]
- do_fork [kernel/fork.c]
- alloc_task_struct [include/asm/processor.c]
- __get_free_pages [mm/page_alloc.c]
- get_pid [kernel/fork.c]
- copy_files
- copy_fs
- copy_sighand
- copy_mm
- allocate_mm
- mm_init
- pgd_alloc -> get_pgd_fast [include/asm/pgalloc.h]
- get_pgd_slow
- dup_mmap [kernel/fork.c]
- copy_page_range [mm/memory.c]
- ptep_set_wrprotect [include/asm/pgtable.h]
- clear_bit [include/asm/bitops.h]
- copy_segments [arch/i386/kernel/process.c]
- copy_thread
- SET_LINKS [include/linux/sched.h]
- wake_up_process [kernel/sched.c]

### コピーオンライト

Linuxは次のようにコピーオンライトを実装しています。

1. コピーされたページにはすべてread-onlyとマークする。これによりページに書き込みを
   行うとページフォルトが発生する。
2. ページフォルトハンドラが新しいページを作成する。

```
 | Page
 | Fault
 | Exception
 |
 |
 -----------> |do_page_fault
                 |handle_mm_fault
                    |handle_pte_fault
                       |do_wp_page
                          |alloc_page        // Allocate a new page
                          |break_cow
                             |copy_cow_page  // Copy old page to new one
                             |establish_pte  // reconfig Page Table pointers
                                |set_pte

              Page Fault ICA
```

- do_page_fault [arch/i386/mm/fault.c]
- handle_mm_fault [mm/memory.c]
- handle_pte_fault
- do_wp_page
- alloc_page [include/linux/mm.h]
- break_cow [mm/memory.c]
- copy_cow_page
- establish_pte
- set_pte [include/asm/pgtable-3level.h]
