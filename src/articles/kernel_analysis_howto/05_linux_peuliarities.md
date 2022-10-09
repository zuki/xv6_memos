# 5. Linuxの特殊性

## 5.1 概要

Linuxには他のOSとは異なる次のような特殊性があります。

1. ページネーションのみ
2. ソフト割り込み（Softirq）
3. カーネルスレッド
4. カーネルモジュール
5. `Proc`ディレクトリ

### 柔軟性要素

項目4と5からシステム管理者はシステム構成において大きな柔軟性を持つことが
でき、ユーザーモードからマシンを再起動することなくカーネルの重大なバグや
特定の問題を解決することができます。たとえば、大きなサーバーで何かを変更
する必要があるが再起動はしたくない場合、モジュールを作成して、カーネルが
このモジュールと会話するように設定することができます。

## 5.2 ページネーションのみ

Linuxではタスクを区別するのにセグメンテーションは使用せず、ページ
ネーションを使用します(すべてのタスクにおいてCODEとDATA/STACKの2つの
セグメントしか使用されません）。

また、各タスクが個別のページテーブルセットを使用するため、タスクをまたがる
ページフォルトが発生することはありません。共有ライブラリのように、異なる
タスクが同一のページテーブルを指す場合がありますが、これはメモリ使用量を減らすために必要なことです。共有ライブラリはCODEだけだけであり、データは
すべてタスクのスタックに格納されることを思い出してください。

### Linuxのセグメント

Linuxカーネルには次の4つのセグメントしか存在しません。

1. カーネルコード [0x10]
2. カーネルデータ/スタック [0x18]
3. ユーザコード [0x23]
4. ユーザデータ/スタック [0x2b]

[各項の構文は ''目的 [セグメント]'']

Intelアーキテクチャで使用されるセグメントレジスタは次の通りです。

- CS: コードセグメント用
- DS: データセグメント用
- SS: スタックセグメント用
- ES: 代替セグメント用（たとえば、2つの異なるセグメント間でメモリを
      コピーする際に使用される）

したがって、全てのタスクはコードには0x23を、データ/stackには0x2bを
使用します。

### Linuxのページネーション

Linuxではアーキテクチャによりますが、3レベルのページが使用されます。
インテル版では2レベルのみサポートされています。Linuxはコピーオンライと
機構もサポートしています（詳細は10章を参照してください）。

### タスク間でアドレス競合が起きないのはなぜか？

答えはとても簡単です。タスク間のアドレスの競合はありえないので存在でき
ません。リニアアドレスから物理アドレスへのマッピングは「ページネーション」
によって行われるので、物理ページを一義的に割り当てるだけです。

### メモリのデフラグは必要ですか？

いいえ。ページ割り当ては動的な処理です。ページはタスクが要求した時に
初めて必要になるので、空きメモリのページから順番にページを選びます。
ページを解放したい時はそのページを空きページリストに追加するだけです。

### カーネルページについてはどうでしょうか？

カーネルページには問題があります。カーネルページは動的割り当てが可能
ですが、リニアカーネル空間は物理カーネル空間と等価であるため、それが
連続した領域に割り当てられていることを保証できません。

コードセグメントについては問題はありません。ブートコードはブート時に
割り当てられる（したがって、割り当てるメモリ量は決まっている）し、
モジュールについては、モジュールコードを格納できるメモリ領域を割り
当てるだけでよいからです。

実施に問題になるのはスタックセグメントです。各タスクが何らかのカーネル
スタックページを使用するからです。スタックセグメントは（スタックの定義に
より）連続でなければならないので、各タスクのスタックサイズの最大制限値を
設定する必要があります。この制限値を超えると悪いことが起こります。
カーネルモードプロセスのデータ構造を上書きしてしまうからです。

カーネルの構造は私達を助けてくれます。カーネル関数は決して次のことを
しないからです。

- 再帰処理
- N回以上の相互呼び出し

Nの値とすべてのカーネル関数の静的変数の平均がわかれば、スタックの
制限値を推定することができます。

この問題を試してみるには自分自身を何度も呼び出す関数を持つモジュールを
作ればよいでしょう。一定の回数が経過するとページフォルト例外ハンドラ
（通常は、読み取り専用ページへの書き込み）のためにカーネルモジュールが
ハングします。

## 5.3 Softirq（ソフト割り込み）

IRQが上がった場合、タスクスイッチを後回しにすることでパフォーマンスを
上げることができます。ある種のタスク（TCP/IPパケットの構築のような、
IRQの直後に実行する必要があり、かつ、割り込み時間内に多くのCPUを消費
するようなタスク）はキューに入れられ、スケジューリングされた時に実行
されます（タイムスライスが終了した時点で）。

最近のカーネル (2.4.x) ではsoftirq機構がカーネルスレッドに導入
されました。`ksoftirqd_CPUn`です。ここで、nはカーネルスレッドを実行
しているCPUの番号です（モノプロセッサのシステムでは`ksoftirqd_CPU0`は
PID 3を使います）。

### Softirqの準備

### Softirqの有効化

`cpu_raise_softirq`は`ksoftirqd_CPU0`カーネルスレッドを起床させ、
エンキュージョブに管理させるルーチンです。

```
|cpu_raise_softirq
   |__cpu_raise_softirq
   |wakeup_softirqd
      |wake_up_process
```

- cpu_raise_softirq [kernel/softirq.c]
- __cpu_raise_softirq [include/linux/interrupt.h]
- wakeup_softirq [kernel/softirq.c]
- wake_up_process [kernel/sched.c]

`__cpu_raise_softirq`関数はソフト割り込み保留を記述するベクタの
該当するビットを設定します。

`wakeup_softirq`は`wakeup_process`を使って`ksoftirqd_CPU0`カーネル
スレッドを起床させます。

### Softirgの実行

TODO: softirq機構に関係するデータ構造体を説明する。

カーネルスレッド`ksoftirqd_CPU0`は起床するとエンキュージョブを事項します。
When kernel thread '''' has been woken up, it will execute queued jobs

`ksoftirqd_CPU0`のコード（mainの無限ループ）は次の遠です。

```c
for (;;) {
   if (!softirq_pending(cpu))
      schedule();
      __set_current_state(TASK_RUNNING);
   while (softirq_pending(cpu)) {
      do_softirq();
      if (current->need_resched)
         schedule
   }
   __set_current_state(TASK_INTERRUPTIBLE)
}
```

- ksoftirqd [kernel/softirq.c]

## 5.4 カーネルスレッド

LinuxはモノリシックOSですが少数の「カーネルスレッド」が存在し、
ハウスキーピング処理を行います。

これらのタスクはユーザメモリは使用せず、カーネルメモリを共有します。
また、他のカーネルモードのコードと同様に最高の特権（i386アーキ
テクチャではRING 0）で動作します。

カーネルスレッドは`kernel_thread [arch/i386/kernel/process]`関数に
よって生成され、アセンブリコードから`clone` [arch/i386/kernel/process.c]
システムコール（`fork`に似たシステムコール）を呼び出します。

```c
int kernel_thread(int (*fn)(void *), void * arg, unsigned long flags)
{
        long retval, d0;

        __asm__ __volatile__(
                "movl %%esp,%%esi\n\t"
                "int $0x80\n\t"         /* Linux/i386 システムコール */
                "cmpl %%esp,%%esi\n\t"  /* 子か親か? */
                "je 1f\n\t"             /* 親の場合は - jump */
                /* 引数をeaxにロードし、さらにプッシュ。こうすれば、
                 * 呼び出された関数が-mregparm付きでコンパイルされて
                 * いるか否かが問題にならない。 */
                "movl %4,%%eax\n\t"
                "pushl %%eax\n\t"
                "call *%5\n\t"          /* fnを呼び出す*/
                "movl %3,%0\n\t"        /* exit */
                "int $0x80\n"
                "1:\t"
                :"=&a" (retval), "=&S" (d0)
                :"0" (__NR_clone), "i" (__NR_exit),
                 "r" (arg), "r" (fn),
                 "b" (flags | CLONE_VM)
                : "memory");
        return retval;
}
```

呼び出されると、スワップやUSBイベントなどの非常に遅いリソースを待ち
受ける新しいタスク（通常は2、3などの非常に小さなPID番号）が実行されます。
非常に遅いリソースが使われるのは、そうしないとタスクスイッチのオーバー
ヘッドが発生するからです。

以下は、最も一般的なカーネルスレッドのリストです（`ps x`コマンドによる）。

```
PID      COMMAND
 1        init
 2        keventd
 3        kswapd
 4        kreclaimd
 5        bdflush
 6        kupdated
 7        kacpid
67        khubd
```

`init`カーネルスレッドはブート時に最初に作成されるプロセスであり、
コンソールデーモンやttyデーモン、ネットワークデーモンなど、他の
すべてのユーザモードタスクを（`/etc/inittab`ファイルから`rc`スクリプト
で）呼び出します。

### カーネルスレッドの例: kswapd [mm/vmscan.c].

`kswapd`は`clone() [arch/i386/kernel/process.c]`により作成されます。

初期化ルーチンは次の通り。

```
|do_initcalls
   |kswapd_init
      |kernel_thread
         |syscall fork (in assembler)
```

- do_initcalls [init/main.c]
- kswapd_init [mm/vmscan.c]
- kernel_thread [arch/i386/kernel/process.c]

## 5.5 カーネルモジュール

### 概要

Linuxカーネルモジュールはカーネルモードで動作するコード（例: fs、net、
hwドライバ）で、実行時に追加することができます。

ケジューリングや割り込み管理、あるいはコアネットワークなどのLinuxの
コアはモジュール化できません。

`/lib/modules/KERNEL_VERSION/`配下にシステムにインストールされている
すべてのモジュールがあります。

### モジュールのロードとアンロード

モジュールをロードするには次のコマンドを実行します。

```
insmod MODULE_NAME parameters
```

例: `insmod ne io=0x300 irq=9`

注意: （たとえば、PCIドライバを使用する場合や特定のパラメタを
/etc/conf.modulesファイルにおいた場合などで）カーネルにパラメタを
自動的に検索させたい場合は、`lsmod`の代わりに`modprobe`を使用します。

モジュールをアンロードするには次のコマンドを実行します。

```
rmmod MODULE_NAME
```

### モジュール定義

モジュールは常に以下の関数を含みます。

1. `init_module`関数。insmod（または、modprobe）コマンドにより実行されます。
2. `cleanup_module`関数。rmmodコマンドにより実行されます。

これらの関数がモジュールにない場合、どの関数がモジュールのinitとexitと
して動作するかを指定するために次の2つのマクロを追加する必要があります。


1. module_init(FUNCTION_NAME)
2. module_exit(FUNCTION_NAME)

注意: モジュールは（EXPORT_SYMBOLマクロで）エクスポートされているカーネル
変数を「見る：ことができます。

以下はカーネルに柔軟性を与える便利なトリックです。

```c
// カーネルソース側
void (*foo_function_pointer)(void *);

if (foo_function_pointer)
  (foo_function_pointer)(parameter);


// モジュール側
extern void (*foo_function_pointer)(void *);

void my_function(void *parameter) {
  // My code
}

int init_module() {
  foo_function_pointer = &my_function;
}

int cleanup_module() {
  foo_function_pointer = NULL;
}
```

この簡単なトリックによりカーネルに非常に高い柔軟性を持たせることが
できます。モジュールをロードした時だけ`my_function`ルーチンを実行
させることができるからです。このルーチンはやりたいことをすべてやって
くれます。たとえば、ネットワークからの帯域幅の入力トラフィックを
制御する`rshaper`モジュールはこの方法で動作しています。

モジュール機構全体はモジュールにエクスポートされているグローバル変数の
おかげで可能であることに注意してください。たとえば、head listです（
このリストはすきなだけ拡張することができます）。典型的な例は、fsや汎用
デバイス (char、block、net、telephony)です。新しいモジュールを受け入れる
にはカーネルを準備する必要があります。場合によっては、できるだけ標準に
なるようなインフラストラクチャ（最近作成されたテレフォニーのように）を
作成する必要があります。

## 5.6 Procディレクトリ

Procファイルシステムは`/proc`ディレクトリにあり、カーネルと直接
対話できる特殊なディレクトリです。

Linuxは`proc`ディレクトリを使ってカーネルと直接対話できる機能をサポート
しています。これは多くのケースで必要です。たとえば、プロセスの主なデータ
構造体を見たい場合やあるインタフェースでは`proxy-arp`機能を有効にし、
別のインタフェースでは無効にしたい場合、ISAやPCIなどのバスの状態を
デバッグしてどのカードがインストールされているか、どのI/Oアドレスや
IRQがカードに割り当てられているかを知りたい場合などです。

```
|-- bus
|   |-- pci
|   |   |-- 00
|   |   |   |-- 00.0
|   |   |   |-- 01.0
|   |   |   |-- 07.0
|   |   |   |-- 07.1
|   |   |   |-- 07.2
|   |   |   |-- 07.3
|   |   |   |-- 07.4
|   |   |   |-- 07.5
|   |   |   |-- 09.0
|   |   |   |-- 0a.0
|   |   |   `-- 0f.0
|   |   |-- 01
|   |   |   `-- 00.0
|   |   `-- devices
|   `-- usb
|-- cmdline
|-- cpuinfo
|-- devices
|-- dma
|-- dri
|   `-- 0
|       |-- bufs
|       |-- clients
|       |-- mem
|       |-- name
|       |-- queues
|       |-- vm
|       `-- vma
|-- driver
|-- execdomains
|-- filesystems
|-- fs
|-- ide
|   |-- drivers
|   |-- hda -> ide0/hda
|   |-- hdc -> ide1/hdc
|   |-- ide0
|   |   |-- channel
|   |   |-- config
|   |   |-- hda
|   |   |   |-- cache
|   |   |   |-- capacity
|   |   |   |-- driver
|   |   |   |-- geometry
|   |   |   |-- identify
|   |   |   |-- media
|   |   |   |-- model
|   |   |   |-- settings
|   |   |   |-- smart_thresholds
|   |   |   `-- smart_values
|   |   |-- mate
|   |   `-- model
|   |-- ide1
|   |   |-- channel
|   |   |-- config
|   |   |-- hdc
|   |   |   |-- capacity
|   |   |   |-- driver
|   |   |   |-- identify
|   |   |   |-- media
|   |   |   |-- model
|   |   |   `-- settings
|   |   |-- mate
|   |   `-- model
|   `-- via
|-- interrupts
|-- iomem
|-- ioports
|-- irq
|   |-- 0
|   |-- 1
|   |-- 10
|   |-- 11
|   |-- 12
|   |-- 13
|   |-- 14
|   |-- 15
|   |-- 2
|   |-- 3
|   |-- 4
|   |-- 5
|   |-- 6
|   |-- 7
|   |-- 8
|   |-- 9
|   `-- prof_cpu_mask
|-- kcore
|-- kmsg
|-- ksyms
|-- loadavg
|-- locks
|-- meminfo
|-- misc
|-- modules
|-- mounts
|-- mtrr
|-- net
|   |-- arp
|   |-- dev
|   |-- dev_mcast
|   |-- ip_fwchains
|   |-- ip_fwnames
|   |-- ip_masquerade
|   |-- netlink
|   |-- netstat
|   |-- packet
|   |-- psched
|   |-- raw
|   |-- route
|   |-- rt_acct
|   |-- rt_cache
|   |-- rt_cache_stat
|   |-- snmp
|   |-- sockstat
|   |-- softnet_stat
|   |-- tcp
|   |-- udp
|   |-- unix
|   `-- wireless
|-- partitions
|-- pci
|-- scsi
|   |-- ide-scsi
|   |   `-- 0
|   `-- scsi
|-- self -> 2069
|-- slabinfo
|-- stat
|-- swaps
|-- sys
|   |-- abi
|   |   |-- defhandler_coff
|   |   |-- defhandler_elf
|   |   |-- defhandler_lcall7
|   |   |-- defhandler_libcso
|   |   |-- fake_utsname
|   |   `-- trace
|   |-- debug
|   |-- dev
|   |   |-- cdrom
|   |   |   |-- autoclose
|   |   |   |-- autoeject
|   |   |   |-- check_media
|   |   |   |-- debug
|   |   |   |-- info
|   |   |   `-- lock
|   |   `-- parport
|   |       |-- default
|   |       |   |-- spintime
|   |       |   `-- timeslice
|   |       `-- parport0
|   |           |-- autoprobe
|   |           |-- autoprobe0
|   |           |-- autoprobe1
|   |           |-- autoprobe2
|   |           |-- autoprobe3
|   |           |-- base-addr
|   |           |-- devices
|   |           |   |-- active
|   |           |   `-- lp
|   |           |       `-- timeslice
|   |           |-- dma
|   |           |-- irq
|   |           |-- modes
|   |           `-- spintime
|   |-- fs
|   |   |-- binfmt_misc
|   |   |-- dentry-state
|   |   |-- dir-notify-enable
|   |   |-- dquot-nr
|   |   |-- file-max
|   |   |-- file-nr
|   |   |-- inode-nr
|   |   |-- inode-state
|   |   |-- jbd-debug
|   |   |-- lease-break-time
|   |   |-- leases-enable
|   |   |-- overflowgid
|   |   `-- overflowuid
|   |-- kernel
|   |   |-- acct
|   |   |-- cad_pid
|   |   |-- cap-bound
|   |   |-- core_uses_pid
|   |   |-- ctrl-alt-del
|   |   |-- domainname
|   |   |-- hostname
|   |   |-- modprobe
|   |   |-- msgmax
|   |   |-- msgmnb
|   |   |-- msgmni
|   |   |-- osrelease
|   |   |-- ostype
|   |   |-- overflowgid
|   |   |-- overflowuid
|   |   |-- panic
|   |   |-- printk
|   |   |-- random
|   |   |   |-- boot_id
|   |   |   |-- entropy_avail
|   |   |   |-- poolsize
|   |   |   |-- read_wakeup_threshold
|   |   |   |-- uuid
|   |   |   `-- write_wakeup_threshold
|   |   |-- rtsig-max
|   |   |-- rtsig-nr
|   |   |-- sem
|   |   |-- shmall
|   |   |-- shmmax
|   |   |-- shmmni
|   |   |-- sysrq
|   |   |-- tainted
|   |   |-- threads-max
|   |   `-- version
|   |-- net
|   |   |-- 802
|   |   |-- core
|   |   |   |-- hot_list_length
|   |   |   |-- lo_cong
|   |   |   |-- message_burst
|   |   |   |-- message_cost
|   |   |   |-- mod_cong
|   |   |   |-- netdev_max_backlog
|   |   |   |-- no_cong
|   |   |   |-- no_cong_thresh
|   |   |   |-- optmem_max
|   |   |   |-- rmem_default
|   |   |   |-- rmem_max
|   |   |   |-- wmem_default
|   |   |   `-- wmem_max
|   |   |-- ethernet
|   |   |-- ipv4
|   |   |   |-- conf
|   |   |   |   |-- all
|   |   |   |   |   |-- accept_redirects
|   |   |   |   |   |-- accept_source_route
|   |   |   |   |   |-- arp_filter
|   |   |   |   |   |-- bootp_relay
|   |   |   |   |   |-- forwarding
|   |   |   |   |   |-- log_martians
|   |   |   |   |   |-- mc_forwarding
|   |   |   |   |   |-- proxy_arp
|   |   |   |   |   |-- rp_filter
|   |   |   |   |   |-- secure_redirects
|   |   |   |   |   |-- send_redirects
|   |   |   |   |   |-- shared_media
|   |   |   |   |   `-- tag
|   |   |   |   |-- default
|   |   |   |   |   |-- accept_redirects
|   |   |   |   |   |-- accept_source_route
|   |   |   |   |   |-- arp_filter
|   |   |   |   |   |-- bootp_relay
|   |   |   |   |   |-- forwarding
|   |   |   |   |   |-- log_martians
|   |   |   |   |   |-- mc_forwarding
|   |   |   |   |   |-- proxy_arp
|   |   |   |   |   |-- rp_filter
|   |   |   |   |   |-- secure_redirects
|   |   |   |   |   |-- send_redirects
|   |   |   |   |   |-- shared_media
|   |   |   |   |   `-- tag
|   |   |   |   |-- eth0
|   |   |   |   |   |-- accept_redirects
|   |   |   |   |   |-- accept_source_route
|   |   |   |   |   |-- arp_filter
|   |   |   |   |   |-- bootp_relay
|   |   |   |   |   |-- forwarding
|   |   |   |   |   |-- log_martians
|   |   |   |   |   |-- mc_forwarding
|   |   |   |   |   |-- proxy_arp
|   |   |   |   |   |-- rp_filter
|   |   |   |   |   |-- secure_redirects
|   |   |   |   |   |-- send_redirects
|   |   |   |   |   |-- shared_media
|   |   |   |   |   `-- tag
|   |   |   |   |-- eth1
|   |   |   |   |   |-- accept_redirects
|   |   |   |   |   |-- accept_source_route
|   |   |   |   |   |-- arp_filter
|   |   |   |   |   |-- bootp_relay
|   |   |   |   |   |-- forwarding
|   |   |   |   |   |-- log_martians
|   |   |   |   |   |-- mc_forwarding
|   |   |   |   |   |-- proxy_arp
|   |   |   |   |   |-- rp_filter
|   |   |   |   |   |-- secure_redirects
|   |   |   |   |   |-- send_redirects
|   |   |   |   |   |-- shared_media
|   |   |   |   |   `-- tag
|   |   |   |   `-- lo
|   |   |   |       |-- accept_redirects
|   |   |   |       |-- accept_source_route
|   |   |   |       |-- arp_filter
|   |   |   |       |-- bootp_relay
|   |   |   |       |-- forwarding
|   |   |   |       |-- log_martians
|   |   |   |       |-- mc_forwarding
|   |   |   |       |-- proxy_arp
|   |   |   |       |-- rp_filter
|   |   |   |       |-- secure_redirects
|   |   |   |       |-- send_redirects
|   |   |   |       |-- shared_media
|   |   |   |       `-- tag
|   |   |   |-- icmp_echo_ignore_all
|   |   |   |-- icmp_echo_ignore_broadcasts
|   |   |   |-- icmp_ignore_bogus_error_responses
|   |   |   |-- icmp_ratelimit
|   |   |   |-- icmp_ratemask
|   |   |   |-- inet_peer_gc_maxtime
|   |   |   |-- inet_peer_gc_mintime
|   |   |   |-- inet_peer_maxttl
|   |   |   |-- inet_peer_minttl
|   |   |   |-- inet_peer_threshold
|   |   |   |-- ip_autoconfig
|   |   |   |-- ip_conntrack_max
|   |   |   |-- ip_default_ttl
|   |   |   |-- ip_dynaddr
|   |   |   |-- ip_forward
|   |   |   |-- ip_local_port_range
|   |   |   |-- ip_no_pmtu_disc
|   |   |   |-- ip_nonlocal_bind
|   |   |   |-- ipfrag_high_thresh
|   |   |   |-- ipfrag_low_thresh
|   |   |   |-- ipfrag_time
|   |   |   |-- neigh
|   |   |   |   |-- default
|   |   |   |   |   |-- anycast_delay
|   |   |   |   |   |-- app_solicit
|   |   |   |   |   |-- base_reachable_time
|   |   |   |   |   |-- delay_first_probe_time
|   |   |   |   |   |-- gc_interval
|   |   |   |   |   |-- gc_stale_time
|   |   |   |   |   |-- gc_thresh1
|   |   |   |   |   |-- gc_thresh2
|   |   |   |   |   |-- gc_thresh3
|   |   |   |   |   |-- locktime
|   |   |   |   |   |-- mcast_solicit
|   |   |   |   |   |-- proxy_delay
|   |   |   |   |   |-- proxy_qlen
|   |   |   |   |   |-- retrans_time
|   |   |   |   |   |-- ucast_solicit
|   |   |   |   |   `-- unres_qlen
|   |   |   |   |-- eth0
|   |   |   |   |   |-- anycast_delay
|   |   |   |   |   |-- app_solicit
|   |   |   |   |   |-- base_reachable_time
|   |   |   |   |   |-- delay_first_probe_time
|   |   |   |   |   |-- gc_stale_time
|   |   |   |   |   |-- locktime
|   |   |   |   |   |-- mcast_solicit
|   |   |   |   |   |-- proxy_delay
|   |   |   |   |   |-- proxy_qlen
|   |   |   |   |   |-- retrans_time
|   |   |   |   |   |-- ucast_solicit
|   |   |   |   |   `-- unres_qlen
|   |   |   |   |-- eth1
|   |   |   |   |   |-- anycast_delay
|   |   |   |   |   |-- app_solicit
|   |   |   |   |   |-- base_reachable_time
|   |   |   |   |   |-- delay_first_probe_time
|   |   |   |   |   |-- gc_stale_time
|   |   |   |   |   |-- locktime
|   |   |   |   |   |-- mcast_solicit
|   |   |   |   |   |-- proxy_delay
|   |   |   |   |   |-- proxy_qlen
|   |   |   |   |   |-- retrans_time
|   |   |   |   |   |-- ucast_solicit
|   |   |   |   |   `-- unres_qlen
|   |   |   |   `-- lo
|   |   |   |       |-- anycast_delay
|   |   |   |       |-- app_solicit
|   |   |   |       |-- base_reachable_time
|   |   |   |       |-- delay_first_probe_time
|   |   |   |       |-- gc_stale_time
|   |   |   |       |-- locktime
|   |   |   |       |-- mcast_solicit
|   |   |   |       |-- proxy_delay
|   |   |   |       |-- proxy_qlen
|   |   |   |       |-- retrans_time
|   |   |   |       |-- ucast_solicit
|   |   |   |       `-- unres_qlen
|   |   |   |-- route
|   |   |   |   |-- error_burst
|   |   |   |   |-- error_cost
|   |   |   |   |-- flush
|   |   |   |   |-- gc_elasticity
|   |   |   |   |-- gc_interval
|   |   |   |   |-- gc_min_interval
|   |   |   |   |-- gc_thresh
|   |   |   |   |-- gc_timeout
|   |   |   |   |-- max_delay
|   |   |   |   |-- max_size
|   |   |   |   |-- min_adv_mss
|   |   |   |   |-- min_delay
|   |   |   |   |-- min_pmtu
|   |   |   |   |-- mtu_expires
|   |   |   |   |-- redirect_load
|   |   |   |   |-- redirect_number
|   |   |   |   `-- redirect_silence
|   |   |   |-- tcp_abort_on_overflow
|   |   |   |-- tcp_adv_win_scale
|   |   |   |-- tcp_app_win
|   |   |   |-- tcp_dsack
|   |   |   |-- tcp_ecn
|   |   |   |-- tcp_fack
|   |   |   |-- tcp_fin_timeout
|   |   |   |-- tcp_keepalive_intvl
|   |   |   |-- tcp_keepalive_probes
|   |   |   |-- tcp_keepalive_time
|   |   |   |-- tcp_max_orphans
|   |   |   |-- tcp_max_syn_backlog
|   |   |   |-- tcp_max_tw_buckets
|   |   |   |-- tcp_mem
|   |   |   |-- tcp_orphan_retries
|   |   |   |-- tcp_reordering
|   |   |   |-- tcp_retrans_collapse
|   |   |   |-- tcp_retries1
|   |   |   |-- tcp_retries2
|   |   |   |-- tcp_rfc1337
|   |   |   |-- tcp_rmem
|   |   |   |-- tcp_sack
|   |   |   |-- tcp_stdurg
|   |   |   |-- tcp_syn_retries
|   |   |   |-- tcp_synack_retries
|   |   |   |-- tcp_syncookies
|   |   |   |-- tcp_timestamps
|   |   |   |-- tcp_tw_recycle
|   |   |   |-- tcp_window_scaling
|   |   |   `-- tcp_wmem
|   |   `-- unix
|   |       `-- max_dgram_qlen
|   |-- proc
|   `-- vm
|       |-- bdflush
|       |-- kswapd
|       |-- max-readahead
|       |-- min-readahead
|       |-- overcommit_memory
|       |-- page-cluster
|       `-- pagetable_cache
|-- sysvipc
|   |-- msg
|   |-- sem
|   `-- shm
|-- tty
|   |-- driver
|   |   `-- serial
|   |-- drivers
|   |-- ldisc
|   `-- ldiscs
|-- uptime
`-- version
```

このディレクトリには、PIDをファイル名とするすべてのタスクもあります
（バイナリファイルのパス名や使用メモリなど、すべてのタスク情報に
アクセスできます）。

興味深いのは、カーネルの値（たとえば、タスクやTCP/IPスタックで有効に
なっているネットワークオプションに関する情報）を見ることだけでなく、
その一部は変更もできることです（通常は、/proc/sysディレクトリ配下の
ものです）。

```
/proc/sys/
          acpi
          dev
          debug
          fs
          proc
          net
          vm
          kernel

/proc/sys/kernel
```

以下は変更可能な重要かつよく知られているカーネル値です。

```
overflowgid
overflowuid
random
threads-max   // スレッドの最大数、通常は 16384
sysrq         // カーネルハック: レジスタ値などを見ることができる
sem
msgmnb
msgmni
msgmax
shmmni
shmall
shmmax
rtsig-max
rtsig-nr
modprobe      // modprobeファイルのロケーション
printk
ctrl-alt-del
cap-bound
panic
domainname    // Linuxマシンのドメイン名
hostname      // Linuxマシンのホスト名
version       // カーネルコンパイルに関する日付情報
osrelease     // カーネルバージョン (i.e. 2.4.5)
ostype        // Linux!
```

### /proc/sys/net

これはもっとも有益なprocのサブディレクトであると思われます。これにより
ネットワークカーネル構成の非常に重要な設定を変更できます。

```
core
ipv4
ipv6
unix
ethernet
802
```

### /proc/sys/net/core

このディレクトリ配下には、すべてのネットワークのパケット長である`netdev_max_backlog`（通常は300）などの一般的なネットの設定が
リストされています。この値はパケットを受信する際のネットワーク
帯域幅を制限することができます。Linuxはバッファをフラッシュする
ために（ボトムハーフ機構により）約1000/HZミリ秒のスケジューリング
時間待つ必要があります、

```
  300    *        100             =     30 000
packets     HZ(Timeslice freq)         packets/s

30 000   *       1000             =      30 M
packets     average (Bytes/packet)   throughput Bytes/s
```

より高いスループットにしたい場合は、netdev_max_backlogを次のコマンドで
増加させる必要があります。

```
echo 4000 > /proc/sys/net/core/netdev_max_backlog
```

注意: HZの値に関する警告: （alphaやarm-tboxなどの）アーキテクチャでは
この値は1000であり、平均スループット300MBytes/sを得ることができます。

### /proc/sys/net/ipv4

`ip_forward`はLinuxマシンのipフォワーディングを有効または無効にします。
これは全てのデバイスに共通する設定ですので、選択したデバイス毎に指定
できます。

### /proc/sys/net/ipv4/conf/interface

これは最も有用な`/proc`エントリだと思います。ワイヤレスネットワークの
サポートなど、ネット設定の変更ができるからです（詳しくは
[Wireless-HOWTO](http://www.bertolinux.com/)を参照してください）。

以下はこの設定を使う例です。

- "forwarding": インターフェイスのipフォワーディングを有効にします
- "proxy_arp": プロキシarp機能を有効にします。詳しくは、
  [Linux Documentation Project](http://www.tldp.org/)の Proxy arp HOWTOと
  ワイヤレスネットワークにおけるproxy arpの使用については
  [Wireless-HOWTO](http://www.bertolinux.com/)を参照してください。
- "send_redirects": インターフェイスがICMP_REDIRECTを送信しないように
  します（詳しくはこれも[Wireless-HOWTO](http://www.bertolinux.com/)を
  参照してください）。
