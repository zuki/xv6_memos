# Lab4 マルチコアとロック

lab3の課題のいくつかを確認します。

- スタックポインタの16バイトアライメントの問題。stp/ldpのみを使用
- cとアセンブリが相互作用するトラップフレームの最初のメンバーは
  下位アドレス
- トラップフレームではcaller-savedレジスタだけを保存すればよいか
- sp、SP_SEL、SP_EL1の関係
- SP_EL1には直接アクセスできないので、まず、SP_SELを1にしてから
  spの値にアクセスする必要がある
- `call trap(struct trapframe *tf)`のtfはspであり、ユーザスタック
  ポインタ SP_EL0はtrapframeに保持されている

今回の実習では、マルチコアとOSのロック機構について学び、以下の部分を
取り上げます。

- SMPアーキテクチャの定義を理解する
- Raspberry Pi 3におけるマルチコアの初期状態とそれをオンにする方法を
  理解する（演習問題）
- マルチコアに対応するための実装済みモジュールのロック対応（演習問題）

まず、今回の実習のために追加されたコードを、自分の開発ブランチに
取り込みます。次のように、pullした後,rebase/mergeしてください。

```bash
git pull
git rebase origin/lab4
# Or git merge origin/lab4
```

conflictが発生した場合は、gitの指示に従って修正してください。

## 4.1 SMPアーキテクチャ

Raspberry PiやほとんどのノートパソコンはSMP（対称型マルチプロセッシング）
アーキテクチャを採用しており、すべてのCPUが独自のレジスタ（汎用レジスタと
システムレジスタ）を持ちながら、同じメモリとMMIOを共有しています。
すべてのCPUは同じ機能を持っていますが、通常、最初に起動してBIOSの初期化
コードを実行するCPUをBSP（ブートストラッププロセッサ）、それ以外を
AP（アプリケーションプロセッサ）と呼びます。しかし、Raspberry Piは少し
変わっておりGPUから起動します。つまり、CPUの実行が行われる前に、GPUが
BIOSがするようなハードウェアの初期化プロセスを完了するのです。

オペレーティングシステムは、現在どのCPU上で動作しているかを知る必要が
ある場合があるため、ARMv8ではMPIDR_EL1レジスタを使用して各CPUのIDを
提供しています。現在のCPUのIDを取得する方法は`inc/arm.h:cpuid`を
参照してください。

詳細については[ARM Cortex-A Series Programmer's Guide for ARMv8-A](https://cs140e.sergio.bz/docs/ARMv8-A-Programmer-Guide.pdf)の
第14章を参照してください。


## 4.2 APの初期状態

qemuは起動すると[armstub8.S](https://github.com/raspberrypi/tools/blob/master/armstubs/armstub8.S)の
コンパイル済みバイナリを物理アドレス0に直接コピーします。すると、
すべてのCPUがPCを0にして起動します（ただし、一部のCPUはarmstub8.Sにより
停止させられ、1つCPUだけが0x80000にジャンプします。詳細は後で説明します）。

実際のRaspberry Pi 3では`armstub8.S`をコンパイルした実行コードが
start.elfまたはbootcode.binに格納されています。ボードをインストール
したら、これら2つのファイルをSDカードの最初のパーティション（FAT32で
ある必要があります）にコピーする必要があります。GPUはそこから
armstub8.Sのバイナリコードを抽出し、それをそのまま物理アドレス0に
コピーします。すると、すべてのCPUがPCを0にして起動します。

armstub8.Sの疑似コードは以下の通りです。つまり、cpuidが0のCPU（BSPに
相当）は0x80000に直接ジャンプし、cpuidが0でないCPU（APに相当）は、
それぞれのエントリ値が0以外になるまで無限ループします。メモリ上の各
CPUのエントリ値は`0xd8 + cpuid()`です。

```c
extern int cpuid();

void boot_kernel(void (*entry)())
{
    entry();
}

void secondary_spin()
{
   	void *entry;
    while (entry = *(void **)(0xd8 + (cpuid() << 3)) == 0)
        asm volatile("wfe");
    boot_kernel(entry);
}

void main()
{
    if (cpuid() == 0) boot_kernel(0x80000);
    else secondary_spin();
}
```

## 4.3 APの電源を入れる

APの電源の入れ方ですが、BSPは`kern/entry.S:_start`でAPのエントリ値を
変更し、`sev`命令でAPを起床させます。

コードの再利用性を高めるために、エントリの値を`kern/start.S:mp_start`に
変更しています。これは、各CPUが異なるスタックポインタを持つことを除いて、
すべてのCPUが[Lab1の起動](lab1.md)と同様のプロセスの実行を並行して開始
することを意味します。

今回の実習から、APの電源を入れた後のコードはすべてのCPUで並列に実行
されますので、**グローバル変数を変更したり読み出したりする際には必ず
ロックをかけてください**。

### 4.3.1 演習問題1

マルチコアのブートプロセスを完全に把握するために、`kern/entry.S`で
各CPUの初期状態にどのような変更が加えられているかを簡単に説明して
ください。少なくとも、PC、スタックポインタ、ページテーブルの説明を
含めてください。

## 4.4 同期とロック

通常、SMPアーキテクチャではメモリ内の特定のアドレスにアクセスするには
排他的アクセスが必要です。ARMv8ではそのような操作のために、リード
ミューテックスLDXRやライトミューテックスSTXRなどの簡単なアトミック命令が
用意されていますが、これらの命令は、inner or outer sharable, inner and outer write-back, read and write allocate, non-transientな通常メモリ
でしか使用できません。ここでは`inc/mmu.h:PTE_NORMAL`と
`inc/mmu.h:TCR_VALUE`を設定することでこれらのアトミックな命令を適切に
使用できるようにしました。

最もシンプルなロックであるスピンロックは、GCCが提供するatomicマクロを
使用して`kern/spinlock.c`で実装されており、対応するアセンブリコードは
`obj/kernel8.asm`に出力されています。 もちろん、rwlock、mcs、mutexなど
のより複雑なロックの実装を試みることもできます。

### 4.4.1 アトミック操作

一般的なアトミック操作の擬似コードを以下に示します。

Test-and-set (atomic exchange)

```c
int test_and_set(int *p, int new)
{
    int old = *p;
    *p = new;
    return old;
}
```

Fetch-and-add (FAA)

```c
int fetch_and_add(int *p, int inc)
{
    int old = *p;
    *p += inc;
    return old;
}
```

Compare-and-swap (CAS)

````c
int compare_and_swap(int *p, int cmp, int new)
{
    int old = *p;
    if (old == cmp)
        *p = new;
    return old;
}
````

### 4.4.2 スピンロック

```c
struct spinlock {
    int locked;
};
void spin_lock(struct spinlock *lk)
{
    while (test_and_set(lk->locked, 1)) ;
}
void spin_unlock(struct spinlock *lk)
{
    test_and_set(lk->locked, 0);
}
```

### 4.4.3 チケットロック

```c
struct ticketlock {
    int ticket, turn;
};
void ticket_lock(struct ticketlock *lk)
{
    int t = fetch_and_add(lk->ticket, 1);
    while (lk->turn != t) ;
}
void ticket_unlock(struct ticketlock *lk)
{
    lk->turn++;
}
```

チケットロックで待機しているCPUは、常に同じ変数lk->turnにアクセスして
います。ロックが解除されてlk->turnが変更されると、待機しているすべての
CPUのキャッシュを変更する必要があり、CPU数が多い場合にはパフォーマンスが
低下します。

### 4.4.4 MCSロック

これはキューベースのロックで、マルチコアのキャッシュに適しており、
test-and-setとcompare-and-swapの効率を向上させます。以下の実装は
[CS140-Synchronization2](http://www.scs.stanford.edu/20wi-cs140/notes/synchronization2.pdf)を参照しています。

```c
struct mcslock {
    struct mcslock *next;
    int locked;
};
void mcs_lock(struct mcslock *lk, struct mcslock *i)
{
    i->next = i->locked = 0;
    struct mcslock *pre = test_and_set(&lk->next, i);
    if (pre) {
        i->locked = 1;
        pre->next = i;
    }
    while (i->locked) ;
}
void mcs_unlock(struct mcslock *lk, struct mcslock *i)
{
    if (i->next == 0)
        if (compare_and_swap(&lk->next, i, 0) == i)
            return;
    while (i->next == 0) ;
    i->next->locked = 0;
}
```

### 4.4.5 ReadWriteロック

読み取り優先のReadWriteロックは2つのスピンロック（または、
スリープをサポートするmutex）で実装されています。

```c
struct rwlock {
    struct spinlock r, w; /* Or mutex. */
    int cnt; /* Number of readers. */
};
void read_lock(struct rwlock *lk)
{
    acquire(&lk->r);
    if (lk->cnt++ == 0)
        acquire(&lk->w);
    release(&lk->r);
}
void read_unlock(struct rwlock *lk)
{
    acquire(&lk->r);
    if (--lk->cnt == 0)
        release(&lk-w);
    release(&lk->r);
}
void write_lock(struct rwlock *lk)
{
    acquire(&lk->w);
}
void write_unlock(struct rwlock *lk)
{
    release(&lk->w);
}
```

### 4.4.6 セマフォ

セマフォ（Semaphore）は、限られた数のユーザにしか対応できない
共有リソースの制御に適しています。このリソースを最大cnt個を占有
することができます（cntは以下で定義されます）。cntが1の場合が
Mutexです。

```c
struct semaphore {
    /* cnt < 0の場合、-cntは待機者の数を示します */
    int cnt;
    struct spinlock lk;
};
void sem_init(struct semaphore *s, int cnt)
{
    s->cnt = cnt;
}
void sem_wait(struct semaphore *s)
{
    acquire(&s->lk);
    if (--s->cnt < 0)
        sleep(s, &s->lk); /* Sleep on s. */
    release(&s->lk);
}
void sem_signal(struct semaphore *s)
{
    acquire(&s->lk);
    if (s->cnt++ < 0)
        wakeup1(s); /* Wake up one process sleeping on s. */
    release(&s->lk);
}
```

### 4.4.7 演習問題2

`kern/spinlock.c`を読み、割り込みをオフにしないと、
`kern/spinlock.c`で何か問題がありますか。もしあるなら、どのように
変更すれば良いですか。

### 4.4.8 演習問題3

すべてのCPUは並行して`kern/main.c:main`に入ります。これらの初期化関数の
いくつかは一度しか呼ばれないことに注意して、これらを確実にするための
あなたの判断と根拠を簡単に説明し、そのためのロックを`kern/main.c`に
追加してください。

## 実行結果

```
$ make
+ cc kern/spinlock.c
+ cc kern/console.c
+ as kern/vectors.S
+ cc kern/kalloc.c
In file included from kern/kalloc.c:10:
inc/types.h:37: warning: "offsetof" redefined
   37 | #define offsetof(type, member)  ((size_t) (&((type*)0)->member))
      |
In file included from inc/string.h:5,
                 from kern/kalloc.c:9:
/usr/lib/gcc-cross/aarch64-linux-gnu/9/include/stddef.h:406: note: this is the location of the previous definition
  406 | #define offsetof(TYPE, MEMBER) __builtin_offsetof (TYPE, MEMBER)
      |
+ as kern/entry.S
+ cc kern/main.c
+ ld obj/kernel8.elf
+ objdump obj/kernel8.elf
+ objcopy obj/kernel8.img
```

```
$ make qemu
qemu-system-aarch64 -M raspi3 -nographic -serial null -serial mon:stdio -kernel obj/kernel8.img
CPU 0: Init started.
Allocator: Init success.
check_map_region: passed!
check_vm_free: passed!
check_free_list: passed!
- irq init
CPU 0: Init success.
CPU 1: Init started.
- irq init
CPU 1: Init success.
CPU 2: Init started.
- irq init
CPU 2: Init success.
CPU 3: Init started.
- irq init
CPU 3: Init success.
```
