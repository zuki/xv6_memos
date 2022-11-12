# timer機能を実装

## 実行はするがストールする

```
# timertest 2
       Elapsed   Value Interval
START:    0.01                                  // 実行はするが時間が来ても反応なし
[Cntl-P]
Total Memory: 945 MB, Free Memmory: 940 MB

1 sleep  init
2 runble idle
3 runble idle
4 run    idle
5 run    idle
6 sleep  dash fa: 1
7 run    itimertest fa: 6
```

### `init_timervecs()`をしていないためだった


```
# date
Tue Jun 21 10:31:31 JST 2022
# timertest
       Elapsed   Value Interval
START:    0.01
Main:     2.02    0.00    0.00
ALARM:    2.02    0.00    0.00
That's all folks
# date
Tue Jun 21 10:31:57 JST 2022
# timertest 2 0 1 0
       Elapsed   Value Interval
START:    0.01
Main:     2.02    1.00    1.00
ALARM:    2.02    1.00    1.00
Main:     3.02    1.00    1.00
ALARM:    3.02    1.00    1.00
Main:     4.02    1.00    1.00
ALARM:    4.02    1.00    1.00
That's all folks
#
```

## jiffiesの初期値で処理が変わる

- 初期値を負値にするのは意味があったはずだが、当面ゼロにする
- 最初にdateコマンドを実行すると以下はclock割り込みが行われるようだ

| define                                 |             初期値 | 動作                |
| :------------------------------------- | -----------------: | :------------------ |
| ((uint64_t)-300 * HZ)                  | 0xFFFFFFFFFFFF8AD0 | itimerが動かず      |
| (((uint64_t)(-300 * HZ)) & 0xFFFFFFFF) | 0x00000000FFFF8AD0 | clock割り込みしない |
| 0UL                                    | 0x0000000000000000 | 正常稼働            |


## 変更履歴

```
$ git status
On branch mac
Changes to be committed:
	modified:   README.md
	modified:   inc/linux/time.h
	modified:   inc/list.h
	modified:   inc/proc.h
	modified:   inc/syscall1.h
	modified:   kern/clock.c
	new file:   kern/itimer.c
	modified:   kern/main.c
	modified:   kern/proc.c
	modified:   kern/syscall.c
	modified:   kern/timer.c
	modified:   memos/src/SUMMARY.md
	new file:   memos/src/timer.md
	new file:   usr/src/timertest/main.c
```

## 問題を認識

### Macで実行した場合の結果

```
$ ./timer 2 0 1 0
       Elapsed   Value Interval
START:    0.00
Main:     0.50    1.50    1.00
Main:     1.00    1.00    1.00
Main:     1.51    0.49    1.00
ALARM:    2.01    0.99    1.00
Main:     2.01    0.99    1.00
Main:     2.51    0.49    1.00
ALARM:    3.00    1.00    1.00
Main:     3.01    0.99    1.00
Main:     3.51    0.49    1.00
ALARM:    4.00    1.00    1.00
That's all folks
```

### 相違点

- Macでは0.5秒ごとに表示されるMain行でValue値にexpireするまでの残り時間が設定されている
- XV6ではALARM時にしかMain行が表示されない
  - これについては`clock()`コマンドが常に`-1`を返しているため
  - それだとMain行は一度も出力されないはず
- clock()が-1を返すのはmsulでの実装が`__syscall(SYS_clock_gettime, CLOCK_PROCESS_CPUTIME_ID, &ts)`であり、sys_clock_gettimeはCLOCK_PROCESS_CPUTIME_IDに対応していないので`-EINVAL`を返しているため
- CLOCK_PROCESS_CPUTIME_IDの場合とりあえずCLOCK_REALTIMEとして処理するようにするとMain行の振る舞いは正しくなるががALARMが発行されない

```
# timertest
       Elapsed   Value Interval
START:    0.01
Main:     0.53     inf     nan
Main:     1.03    1.00    0.00
Main:     1.53    0.50    0.00
Main:     2.03    0.00    0.00			// ここでストール
QEMU: Terminated
```

## procに現在時を持ち、nano秒単位のjiffiesを設定

```diff
$ git diff inc/proc.h
diff --git a/inc/proc.h b/inc/proc.h
index 0717d68..1abb66a 100644
--- a/inc/proc.h
+++ b/inc/proc.h
@@ -87,6 +87,7 @@ struct proc {
     kernel_cap_t   cap_effective, cap_inheritable, cap_permitted;   // capabilities
     mode_t umask;               // umask

+    uint64_t stime, utime;      // ticks for system and user
     uint64_t it_real_value, it_prof_value, it_virt_value;   /* timer interval value: REAL, PROF, VIRTUAL */
     uint64_t it_real_incr, it_prof_incr, it_virt_incr;      /* timer increment: REAL, PROF, VIRTUAL */
     struct timer_list real_timer;                           /* Real time timer */
$ git diff kern/clock.c
diff --git a/kern/clock.c b/kern/clock.c
index 589f629..78bc40b 100644
--- a/kern/clock.c
+++ b/kern/clock.c
@@ -1,10 +1,12 @@
 #include "clock.h"
+#include "linux/errno.h"
 #include "linux/time.h"
 #include "arm.h"
 #include "base.h"
 #include "irq.h"
 #include "console.h"
 #include "rtc.h"
+#include "proc.h"

 /* Local timer */
 #define TIMER_ROUTE             (LOCAL_BASE + 0x24)
@@ -92,6 +94,7 @@ void
 clock_intr()
 {
     ++jiffies;
+    thisproc()->stime = jiffies * 1000000000 / HZ;
     trace("c: %d", jiffies);
     update_times();
     run_timer_list();
@@ -101,20 +104,39 @@ clock_intr()
 long
 clock_gettime(clockid_t clk_id, struct timespec *tp)
 {
-    // TODO: clk_idによる振り分け
-    tp->tv_nsec = xtime.tv_nsec;
-    tp->tv_sec = xtime.tv_sec;
-
-    trace("tv_sec: %lld, tv_nsec: %lld", tp->tv_sec, tp->tv_nsec);
+    uint64_t ptime;
+
+    switch(clk_id) {
+        default:
+            return -EINVAL;
+        case CLOCK_REALTIME:
+            tp->tv_nsec = xtime.tv_nsec;
+            tp->tv_sec = xtime.tv_sec;
+            break;
+        case CLOCK_PROCESS_CPUTIME_ID:
+            ptime = thisproc()->stime + thisproc()->utime;
+            tp->tv_nsec = ptime % 1000000000;
+            tp->tv_sec  = ptime / 1000000000;
+            break;
+    }
+    debug("clk: %d, tv_sec: %lld, tv_nsec: %lld", clk_id, tp->tv_sec, tp->tv_nsec);
     return 0;
 }

 long clock_settime(clockid_t clk_id, const struct timespec *tp)
 {
-    // TODO: 権限チェック, lock?
-    xtime.tv_nsec = tp->tv_nsec;
-    xtime.tv_sec = tp->tv_sec;
-
+    switch(clk_id) {
+        default:
+            return -EINVAL;
+        case CLOCK_REALTIME:
+            if (capable(CAP_SYS_TIME)) {
+                xtime.tv_nsec = tp->tv_nsec;
+                xtime.tv_sec = tp->tv_sec;
+            } else {
+                return -EPERM;
+            }
+            break;
+    }
     return 0;
 }
```

- かなりずれるが正式な計時はTODO

```
# date
Tue Jun 21 10:31:26 JST 2022
# timertest
       Elapsed   Value Interval
START:    0.00
Main:     0.49    1.53    0.00
Main:     1.04    0.98    0.00
Main:     1.62    0.40    0.00
ALARM:    2.02    0.00    0.00
That's all folks
# timertest 2 0 1 0
       Elapsed   Value Interval
START:    0.00
Main:     0.51    1.50    1.00
Main:     1.15    0.86    1.00
Main:     1.71    0.30    1.00
ALARM:    2.01    1.00    1.00
Main:     2.24    0.77    1.00
Main:     2.77    0.24    1.00
ALARM:    3.01    1.00    1.00
Main:     3.45    0.56    1.00
Main:     3.96    0.05    1.00
ALARM:    4.01    1.00    1.00
That's all folks
# date
Tue Jun 21 10:32:25 JST 2022
```

## CLOCK_PROCESS_CPUTIME_ID 対応を少し真面目に実装

- process->stime, utimeをtimer()割り込み時に増分
- user_modeを判別
- ptimeがTick単位だったのをナノ秒に変換
- QEMUと実機でtimerfreq()の返す値が違った

```diff
dspace@mini:~/xv6-raspi/mac-rpi-os$ git diff
diff --git a/inc/irq.h b/inc/irq.h
index 0f789e3..19ee9db 100644
--- a/inc/irq.h
+++ b/inc/irq.h
@@ -32,6 +32,6 @@ void irq_init();
 void irq_enable(int);
 void irq_disable(int);
 void irq_register(int, void (*)());
-void irq_handler();
+void irq_handler(int);

 #endif
diff --git a/kern/clock.c b/kern/clock.c
index 673e4c9..53f1f58 100644
--- a/kern/clock.c
+++ b/kern/clock.c
@@ -94,7 +94,7 @@ void
 clock_intr()
 {
     ++jiffies;
-    thisproc()->stime = jiffies * 1000000000 / HZ;
+    //thisproc()->stime = jiffies * 1000000000 / HZ;
     trace("c: %d", jiffies);
     update_times();
     run_timer_list();
@@ -114,7 +114,8 @@ clock_gettime(clockid_t clk_id, struct timespec *tp)
             tp->tv_sec = xtime.tv_sec;
             break;
         case CLOCK_PROCESS_CPUTIME_ID:
-            ptime = thisproc()->stime + thisproc()->utime;
+            struct proc *p = thisproc();
+            ptime = (p->stime + p->utime) * TICK_NSEC;
             tp->tv_nsec = ptime % 1000000000;
             tp->tv_sec  = ptime / 1000000000;
             break;
diff --git a/kern/irq.c b/kern/irq.c
index fbcd76c..00737b6 100644
--- a/kern/irq.c
+++ b/kern/irq.c
@@ -235,14 +235,14 @@ handle1(int i)
 }

 void
-irq_handler()
+irq_handler(int user_mode)
 {
     int nack = 0;
 #ifndef USE_GIC
     int src = get32(IRQ_SRC_CORE(cpuid()));
     assert(!(src & ~(IRQ_SRC_CNTPNSIRQ | IRQ_SRC_GPU | IRQ_SRC_TIMER)));
     if (src & IRQ_SRC_CNTPNSIRQ) {
-        timer_intr();
+        timer_intr(user_mode);
         nack++;
     }
     if (src & IRQ_SRC_TIMER) {
diff --git a/kern/timer.c b/kern/timer.c
index 0ae618b..c4710a0 100644
--- a/kern/timer.c
+++ b/kern/timer.c
@@ -7,6 +7,7 @@
 #include "proc.h"
 #include "list.h"
 #include "spinlock.h"
+#include "rtc.h"

 /* Core Timer */
 #define CORE_TIMER_CTRL(i)      (LOCAL_BASE + 0x40 + 4*(i))
@@ -15,13 +16,26 @@
 static uint64_t dt;
 static uint64_t cnt;

+static update_proc_time(int user_mode)
+{
+    struct proc *p = thisproc();
+    if (user_mode)
+        p->utime++;
+    else
+        p->stime++;
+}
+
 void
 timer_init()
 {
-    dt = timerfreq() / HZ;       // 10 ms
+#ifdef USING_RASPI
+    dt = timerfreq() / HZ;       // 10 ms = 19.2 * 10^6 / 100
+#else
+    dt = 62500000UL / HZ;       // QEMUはtimerfreq()で得られる値が実機と違う
+#endif
+    info("timerfreq = 0x%llx", timerfreq());
     asm volatile ("msr cntp_ctl_el0, %[x]"::[x] "r"(1));    // Physical Timer enable
     asm volatile ("msr cntp_tval_el0, %[x]"::[x] "r"(dt));  // Set counter of physica timer
-// asm volatile ("msr cntp_cval_el0, %[x]"::[x] "r"(ct));   // Set compare time
     put32(CORE_TIMER_CTRL(cpuid()), CORE_TIMER_ENABLE);     // core timer enable
 #ifdef USE_GIC
     irq_enable(IRQ_LOCAL_CNTPNS);
@@ -40,10 +54,15 @@ timer_reset()
  * which is determined by cpu clock (may be tuned for power saving).
  */
 void
-timer_intr()
+timer_intr(int user_mode)
 {
-    trace("t: %d", ++cnt);
+#if 0
+    if (cpuid() == 0 && ++cnt % 100) {
+        info("cnt=%lld, jif=%lld", cnt, jiffies);
+    }
+#endif
     timer_reset();
+    update_proc_time(user_mode);
     // プリエンプション
     yield();
 }
diff --git a/kern/trap.c b/kern/trap.c
index cdb6fdd..dc180bb 100644
--- a/kern/trap.c
+++ b/kern/trap.c
@@ -18,6 +18,12 @@ extern long syscall1(struct trapframe *);
 static long pf_handler(int dfs, uint64_t far);
 void trap_error(uint64_t type);

+#define PSR_MODE_EL0t   0x00000000
+#define PSR_MODE_MASK   0x0000000F
+
+#define user_mode(tf) \
+    (((int)((tf)->spsr) & PSR_MODE_MASK) == PSR_MODE_EL0t)
+
 void
 trap_init()
 {
@@ -42,7 +48,7 @@ trap(struct trapframe *tf)
         if (il) {
             debug("IL bit on");
         } else {
-            irq_handler();
+            irq_handler(user_mode(tf));
         }
         check_pending_signal();
         break;
```

## QEMUの場合は相変わらず大いにずれる

```
# timertest
       Elapsed   Value Interval
START:    0.00
clock=60000
Main:     0.36    1.65    0.00
Main:     0.75    1.27    0.00
Main:     1.08    0.94    0.00
Main:     1.47    0.55    0.00
Main:     1.85    0.17    0.00
ALARM:    2.02    0.00    0.00
That's all folks
# timertest 2 0 1 0
       Elapsed   Value Interval
START:    0.00
clock=50000
Main:     0.33     inf     nan
Main:     0.69    1.32    1.00
Main:     1.06    0.95    1.00
Main:     1.44    0.57    1.00
Main:     1.78    0.23    1.00
ALARM:    2.01    1.00    1.00
Main:     2.17    0.84    1.00
Main:     2.55    0.46    1.00
Main:     2.96    0.05    1.00
ALARM:    3.01    1.00    1.00
Main:     3.36    0.65    1.00
Main:     3.70    0.31    1.00
ALARM:    4.01    1.00    1.00
That's all folks
```

## 実機はほぼ正しい結果となった

```
# timertest
       Elapsed   Value Interval
START:    0.00
clock=10000
Main:     0.49    1.51    0.00
Main:     0.99    1.01    0.00
Main:     1.49    0.51    0.00
Main:     1.98    0.02    0.00
ALARM:    2.00    0.00    0.00
That's all folks
# timertest 2 0 1 0
       Elapsed   Value Interval
START:    0.00
clock=10000
Main:     0.49    1.51    1.00
Main:     0.99    1.01    1.00
Main:     1.49    0.51    1.00
Main:     1.99    0.01    1.00
ALARM:    2.00    1.00    1.00
Main:     2.49    0.51    1.00
Main:     2.99    0.01    1.00
ALARM:    3.00    1.00    1.00
Main:     3.49    0.51    1.00
Main:     3.99    0.01    1.00
ALARM:    4.00    1.00    1.00
That's all folks
```

## カウンタタイマー物理タイマーに関するレジスタ: Counter-timer Phyical Timer Registers

### CNTP_CTL_EL0: 制御レジスタ: Control register

ISTATUS, bit [2]: 1: タイマー条件が成立
IMASK,   bit [1]: 1: タイマー割り込みをマスク（割り込みしな）
ENABLE,  bit [0]: 1: タイマーを有効にする

### CNTPCT_EL0: カウントレジスタ: Count register

64-bit 物理カウント値を保持する。

EL0またはEL1からのCNTPCT_EL0の読み出しはアクセス例外とはならず、以下の全てが真であれば、
(PhysicalCountInt<63:0> - CNTPOFF_EL2<63:0>) が返される。

- CNTHCTL_EL2.ECVが1である。
- HCR_EL2.{E2H,TGE}が{1,1}でない。

PhysicalCountInt<63:0>は、CNTPCT_EL0がEL2またはEL3から読み出されたときに返される物理カウントである。

### CNTP_TVAL_EL0: タイマー値レジスタ: TimerValue regist

このレジスタの読み出しは

    CNTP_CTL_EL0.ENABLE が 0 の場合、返される値は UNKNOWN である。
    CNTP_CTL_EL0.ENABLE が 1 の場合、返される値は (CNTP_CVAL_EL0 - CNTPCT_EL0) である。

このレジスタに書き込むと、CNTP_CVAL_EL0が（CNTPCT_EL0 + TimerValue）に設定される。
TimerValueは符号付き32ビット整数として扱われる。

CNTP_CTL_EL0.ENABLEが1の時は、 (CNTPCT_EL0 - CNTP_CVAL_EL0)が0以上の時にタイマー条件が
成立する。つまり、TimerValueは32bitダウンカウンタータイマーと同じように動作する。
タイマーの条件が成立した場合、

- CNTP_CTL_EL0.ISTATUSが1に設定される。
- CNTP_CTL_EL0.IMASKが0であれば、割り込みが発生する。

CNTP_CTL_EL0.ENABLEが0の場合、タイマーの条件は成立しないが、CNTPCT_EL0はカウントし続けるので
TimerValueビューはカウントダウンし続けるように見える。

### CNTP_CVAL_EL0: コンペア値レジスタ: CompareValue register

EL1物理タイマーのコンペア値を保持する。

CNTP_CTL_EL0.ENABLEが1の時、（CNTPCT_EL0 - CompareValue）が0以上の時にタイマーの条件が
成立する。これはCompareValueが64ビットアップカウンタータイマーのように動作することを
意味する。タイマーの条件が成立した場合、

- CNTP_CTL_EL0.ISTATUSが1に設定される。
- CNTP_CTL_EL0.IMASKが0であれば、割り込みが発生する。

CNTP_CTL_EL0.ENABLEが0の場合、タイマー条件は成立しないが、CNTPCT_EL0はカウントを続ける。

このフィールドの値はすべてのカウンタ計算においてゼロ拡張で扱われる。

### CNTFRQ_EL0, Counter-timer Frequency register

クロック周波数。システムカウンターのクロック周波数をHzで示す。

このレジスタはソフトウェアがシステムカウンタの周波数を検出できるように
するために提供されている。システムの初期化の一部としてこの値を
プログラムする必要がある。このレジスタの値は、ハードウェアによって
解釈されることはない。

**注**: Raspi3B+では`boot/armstub8.S`で19,200.000 (仕様書とおり。
ただし、QEMUでは62,500,000となる)に設定されている。
