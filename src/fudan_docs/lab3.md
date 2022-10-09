# Lab3 割り込みと例外

今回の実習ではオペレーティングシステムの割り込みについて学習し、
次の項目を取り上げます。

1. 割り込みプロセスを理解する
2. 割り込み設定を理解する
3. トラップフレームを設計し、トラップフレームの保存と復元を行う（演習）
4. 割り込み処理関数を理解する

まず、今回の実習で追加されたコードを自分の開発ブランチに取り込んで
ください。すなわち、次のように pullして、rebase/mergeしてください。

```bash
git pull
git rebase origin/lab3
# Or git merge origin/lab3
```

conflictが発生した場合は、gitの指示に従ってしょりしてください。

## 3.1 割り込みプロセス

割り込みの理解を深めるためには、割り込み時にCPUが実際に何をしているのかを
知る必要があります。ARMv8アーキテクチャで割り込みを行う場合、CPUはまず
**すべての割り込みをオフにします**（DAIFレジスタの値を確認すると、DAIFは
すべてマスクされていることがわかります。もちろん手動でオンにすることも
できますが、実装上、現時点ではカーネルステート割り込みをサポートして
いません）。それから、割り込みベクタテーブルへのジャンプ、割り込み情報の
保存、スタックスイッチなどを行います。 具体的には

- 割り込みは、割り込み後自動的にオフになります。つまり、割り込みはすべて
  ユーザモードで発生します。すなわち、**EL0からEL1への割り込み**のみが
  発生します。
- 割り込みタイプに応じて、割り込みベクタテーブルの該当位置にジャンプします。
- 割り込み前のPSTATEがSPSR_EL1に、PCがELR_EL1に、割り込み原因がESR_EL1に
  保存されます。
- スタックポインタspをSP_EL0からSP_EL1に変更します。これはSPSEL_EL1の値が
  1になることも意味します。

## 3.2 割り込みの設定

### 3.2.1 割り込みベクタテーブル

`kern/main.c`では、ARMv8の割り込みベクタテーブル`kern/vectors.S`を
`lvbar`でロードしています。割り込みの機能は、ハードウェアが
割り込み信号を受信すると、その割り込みタイプ（同期/IRQ/FIQ/システム
エラー）と現在のPSTATE（EL0/ EL1, AArch32/AArch64）を保存し、割り込み
原因をESR_EL1に保存し、割り込み発生時のPCをELR_EL1に保存してから、別の
アドレスにジャンプします。そのアドレスを記述したテーブルは「割り込み
ベクタテーブル」と呼ばれます。

割り込みベクタテーブルのアドレスはVBAR_EL1で設定することができます
（仮想アドレスであることに注意してください）。テーブルの各項目は1つの
割り込みタイプのエントリであり、128バイトのサイズで、合計16のエントリが
あります。 前節で述べたように、私たちはEL0からEL1への割り込みしかサポート
していませんし、コアはAArch64しかサポートしていませんので、ここでは
以下の2つのエントリにのみ注目します。残りのエントリについては[ARM Cortex-A Series Programmer's Guide for ARMv8-A](https://cs140e.sergio.bz/docs/ARMv8-A-Programmer-Guide.pdf) の第10.4章を参照してください。

| アドレス       | 例外タイプ     | 説明                                  | 利用例               |
| -------------- | -------------- | ------------------------------------- | ------------------- |
| VBAR_EL1+0x400 | 同期    | AArch64の下位ELからの割り込み | システムコール         |
| VBAR_EL1+0x480 | IRQ/vIRQ       | AArch64の下位ELからの割り込み | UART/timer/clock/sd |

`kern/vectors.S`では、この2つを除くすべての割り込みは`kern/trap.c:irq_error`
を実行してエラーを報告しています。

### 3.2.2 割り込みのルーティング

マルチコアの場合、割り込みが発生した際にハードウェアはどのCPUに割り込みを
かけるべきでしょうか。これをハードウェアに伝える必要があります。
Raspberry Piは特殊ですべての割り込みはまずGPUに送られ、その後GPUにより
特定のCPUに割り当てられます（詳細は[BCM2836](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2836/QA7_rev3.4.pdf)を
参照してください）。ここでは、次のタイプの割り込みのみを使用します。
実装を容易にするために、すべてのグローバル割り込みを最初のコア（CPU0）に
ルーティングします。

- UART入力: qemuのキーボード入力。CPU0にルーティングされます。
- ブローバルクロック（clock）: 固定周期のクロックです。実際には、システム
  時刻の記録やシステムの監視などに使用されます。CPU0にルーティングされます。
- ローカルクロック（timer）: 各CPUは各々クロック割り込みを持っており、
  合計4つの割り込みがあります。周波数はCPUの周波数に応じて変化し、プロセスの
  タイムスライスを記録するために使用されます。4つの割り込みは各々4つのCPUに
  ルーティングされます。
- SDカードデバイス: 後でファイルシステムの章で追加されます。CPU0に
  ルーティングされます。

これらの割り込みの初期化と処理関数は各々`kern/uart.c`と`kern/clock.c`、
`kern/timer.c`で実装されています。

## 3.3 トラップフレーム

トラップフレーム構造体の役割は、割り込み前のすべてのレジスタと必要な
割り込み情報を保持することです。今回の実習では、31本の汎用レジスタ
（x0~x30）とシステムレジスタのELR_EL1、SPSR_EL1、SP_EL0の値に注目する
必要があります。

### 3.3.1 演習1

あなたが実装したOSでは、割り込み時にCPUがどのような動作をするのかを
簡単に説明してください。

### 3.3.2 演習2

`inc/trap.h`に独自のトラップフレームを設計し、なぜそのように設計した
のかを簡単に説明してください。

### 3.3.3 演習3

`kern/trapasm.S`のコードを完成させて、トラップフレームの構築、復旧を
行ってください。

## 3.4 割り込み処理

すべての正当な割り込みは`kern/vectors.S`から`kern/trapasm.S`に送られ、
`kern/trap.c:trap`関数（割り込み処理関数）が呼び出されます。 ここでは、
割り込みの原因に応じて、対応するハンドラ関数が呼び出されます。

### 3.4.1 割り込みテスト

割り込みが正しく実装されているかどうかをテストしたい場合は，カーネルの
割り込みを手動でオンにし（カーネル状態の割り込みをサポートしたい場合を
除き，必ず後でオフにすることを忘れないでください）、次のように変更して，
timer/clock/uartの割り込みが正しく行われるかどうかを確認する必要があります。

1. `kern/vectors.S`にある`verror(5)`を`ventry`に変更してください。
   この項目は、SP_EL1スタックポインタを使用したEL1のIRQ割り込みを表して
   います。`kern/entry.S`の`msr spsel, #1`命令は、スタックポインタを
   カーネルスタックであるSP_EL1に切り替えます。その後、ユーザプロセスは
   SP_EL0を使用していることに注意してください。
2. `inc/arm.h`にある`sti`関数は、`kern/main.c:main`関数の最後に呼び出され、
   割り込みを有効にします。

通常、qemuにはタイマ/クロックの出力があり、入力文字も表示されています。

## 3.5 参考資料

実習中は[ARM Cortex-A Series Programmer's Guide for ARMv8-A](https://cs140e.sergio.bz/docs/ARMv8-A-Programmer-Guide.pdf)の
第9章と第10章に注目する必要があります。

- 第9章では、ARMv8アーキテクチャにおけるアセンブリとCのやりとりを標準化
  しています。たとえば、各レジスタの役割（関数の戻りアドレスとして使われる
  のはどれか、caller/calleeが保存するレジスタはどれか）、スタックフレームの
  構造などです。この知識はトラップフレームを設計する際に活用されます。
- 第10章では、割り込みについて書かれており、10.5ではtrapasm.Sと同じような
  簡単なリファレンスコードがあります。

## 実行結果

### 1. 変更なし（割り込み無効）

```
$ make
+ cc kern/vm.c
In file included from kern/vm.c:10:
inc/types.h:37: warning: "offsetof" redefined
   37 | #define offsetof(type, member)  ((size_t) (&((type*)0)->member))
      |
In file included from inc/string.h:5,
                 from kern/vm.c:9:
/usr/lib/gcc-cross/aarch64-linux-gnu/9/include/stddef.h:406: note: this is the location of the previous definition
  406 | #define offsetof(TYPE, MEMBER) __builtin_offsetof (TYPE, MEMBER)
      |
+ cc kern/trap.c
+ cc kern/console.c
+ as kern/vectors.S
+ as kern/trapasm.S
+ cc kern/kalloc.c
In file included from kern/kalloc.c:9:
inc/types.h:37: warning: "offsetof" redefined
   37 | #define offsetof(type, member)  ((size_t) (&((type*)0)->member))
      |
In file included from inc/string.h:5,
                 from kern/kalloc.c:8:
/usr/lib/gcc-cross/aarch64-linux-gnu/9/include/stddef.h:406: note: this is the location of the previous definition
  406 | #define offsetof(TYPE, MEMBER) __builtin_offsetof (TYPE, MEMBER)
      |
+ cc kern/uart.c
kern/uart.c: In function ‘uart_intr’:
kern/uart.c:22:1: warning: control reaches end of non-void function [-Wreturn-type]
   22 | }
      | ^
+ as kern/entry.S
+ cc kern/clock.c
+ cc kern/timer.c
kern/timer.c: In function ‘timer’:
kern/timer.c:29:5: warning: implicit declaration of function ‘cprintf’ [-Wimplicit-function-declaration]
   29 |     cprintf("cpu %d timer.\n", cpuid());
      |     ^~~~~~~
+ cc kern/kpgdir.c
+ cc kern/main.c
+ ld obj/kernel8.elf
+ objdump obj/kernel8.elf
+ objcopy obj/kernel8.img

$ make qemu
qemu-system-aarch64 -M raspi3 -nographic -serial null -serial mon:stdio -kernel obj/kernel8.img
Allocator: Init success.
check_map_region: passed!
check_vm_free: passed!
check_free_list: passed!
- irq init
```

### 2. 割り込みを有効に

```
diff --git a/kern/main.c b/kern/main.c
index c975aae..0bf216a 100644
--- a/kern/main.c
+++ b/kern/main.c
@@ -30,7 +30,7 @@ main()
     lvbar(vectors);
     timer_init();

-    // sti();
+    sti();

     while (1) {}
 }


$ make qemu
qemu-system-aarch64 -M raspi3 -nographic -serial null -serial mon:stdio -kernel obj/kernel8.img
Allocator: Init success.
check_map_region: passed!
check_vm_free: passed!
check_free_list: passed!
- irq init
- irq_error
irq of type 5 unimplemented.
kern/console.c:115: kernel panic.
```

### 3. 割り込みハンドラを変更

```
diff --git a/kern/vectors.S b/kern/vectors.S
index 678d042..e5acf58 100644
--- a/kern/vectors.S
+++ b/kern/vectors.S
@@ -16,8 +16,8 @@ el1_sp0:

 el1_spx:
     verror(4)
-    verror(5)
-    # ventry
+    #verror(5)
+    ventry
     verror(6)
     verror(7)

$ make qemu
qemu-system-aarch64 -M raspi3 -nographic -serial null -serial mon:stdio -kernel obj/kernel8.img
Allocator: Init success.
check_map_region: passed!
check_vm_free: passed!
check_free_list: passed!
- irq init
cpu 0 timer.
cpu 0 timer.
cpu 0 timer.
cpu 0 clock.
cpu 0 timer.
cpu 0 timer.
cpu 0 timer.
cpu 0 clock.
cpu 0 timer.
cpu 0 timer.
cpu 0 timer.
cpu 0 clock.
cpu 0 timer.
make: *** [Makefile:79: qemu] Killed    # kill -9 qemu
```
