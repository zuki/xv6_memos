# 1. 基本

## 3.1 カーネルとはなにか?

カーネルはコンピュータシステムの「核」であり、ユーザーがコンピュータ資源を
共有するための「ソフトウェア」である。

カーネルはOS（Operating System）のメインソフトウェアと考えることができ、
グラフィックス管理が含まれることもあります。

たとえば、Linuxでは（他のUnix系OSと同様）XWindow環境はLinuxカーネルには
含まれません。XWindowは（ユーザモードI/Oを使用してビデオカードデバイスに
アクセスする）グラフィックの操作しか管理しないからです。

これに対し、Windows環境（Win9x、WinME、WinNT、Win2K、WinXPなど）は、グラフィック
環境とカーネルが混在しています。

## 3.2 ユーザモードとカーネルモードの違いはなにか?

### 概要

一昔前、コンピュータが部屋ほどの大きさだった頃、ユーザはアプリケーションを
実行するのに大変苦労し、時にはアプリケーションがコンピュータをクラッシュ
させることもありました。

## 動作モード

アプリケーションが何度もクラッシュするのを防ぐために、新しいOSでは2種類の
動作モードが設定されました。

1. カーネルモード: マシンが重要なデータ構造を操作し、ハードウェア（IN/OUTまたは
   メモリマップド）やメモリに直接アクセスし、IRQ、DMAなどを使用して動作します。
2. ユーザモード: ユーザがアプリケーションを実行できます。

```
               |          Applications           /|\
               |         ______________           |
               |         | User Mode  |           |
               |         ______________           |
               |               |                  |
Implementation |        _______ _______           |   Abstraction
    Detail     |        | Kernel Mode |           |
               |        _______________           |
               |               |                  |
               |               |                  |
               |               |                  |
              \|/          Hardware               |
```

カーネルモードはユーザモードのアプリケーションがシステムやその機能にダメージを
与えるのを「防ぎ」ます。

最近のマイクロプロセッサは少なくとも2つの異なる状態をハードウェアで実装しています。
たとえば、Intelでは4つの状態がPL（特権レベル）を決定します。0,1,2,3の状態を使用する
ことができ、カーネルモードは0を使用します。

Unix OSでは2つの特権レベルしか必要としないので評価基準としてこのようなパラダイムを
使用します。

## 3.3 ユーザモードとカーネルモードの切り替え

### いつ切り替えるのか?

2つのモードがあることを理解したら、次はそれらをいつ切り替えるのかを知る必要があります。

通常、切り替えのポイントは2つあります。

- システムコールの呼び出し時: システムコールを呼び出す際、タスクは自発的にカーネル
  モードに存在するコードを呼び出します。
- IRQ（または礼儀）が上がった時: IRQが上がると、IRQハンドラ（または例外ハンドラ）が
  呼びだされ、それが終わると何事もなかったかのように割り込まれたタスクに制御が戻ります。

### システムコール

システムコールはカーネルモードにあるOSルーチンを管理する特別な関数のようなものです。

システムコールは次のような場合に呼び出すことができます。

- I/Oドライバやファイルにアクセスする場合（読み書きなど）
- 特権情報にアクセスにする費用がある場合（pid、スケジューラポリシー情報などの変更など）
- 実行コンテキストを変更する必要がある場合（他のアプリケーションのフォークや実行など）
- 特定のコマンドを実行する必要がある場合（''chdir'', ''kill", ''brk'', ''signal''など)

```
                                 |                |
                         ------->| System Call i  | (デバイスのアクセス)
|                |       |       |  [sys_read()]  |
| ...            |       |       |                |
| system_call(i) |--------       |                |
|   [read()]     |               |                |
| ...            |               |                |
| system_call(j) |--------       |                |
|   [get_pid()]  |       |       |                |
| ...            |       ------->| System Call j  | (カーネルデータ構造体へのアクセス)
|                |               |  [sys_getpid()]|
                                 |                |

    USER MODE                        KERNEL MODE


                        Unix System Calls Working
```

システムコールはユーザモードが低レベル資源（ハードウェア）と話すのに使用できる
ほとんど唯一のインタフェースです。この唯一の例外はプロセスが''ioperm''システムコールを
使用する場合です。この場合、ユーザモードプロセスは直接デバイスにアクセスできます
（IRQは使用できません）。

注意: すべての''C'' 関数がシステムコールというわけではありません。その一部だけです。

以下はLinuxカーネル 2.4.17におけるシステムコールの一覧です（[ arch/i386/kernel/entry.S ]）。

```
    .long SYMBOL_NAME(sys_ni_syscall)       /* 0  -  old "setup()" system call*/
    .long SYMBOL_NAME(sys_exit)
    .long SYMBOL_NAME(sys_fork)
    .long SYMBOL_NAME(sys_read)
    .long SYMBOL_NAME(sys_write)
    .long SYMBOL_NAME(sys_open)             /* 5 */
    .long SYMBOL_NAME(sys_close)
    .long SYMBOL_NAME(sys_waitpid)
    .long SYMBOL_NAME(sys_creat)
    .long SYMBOL_NAME(sys_link)
    .long SYMBOL_NAME(sys_unlink)           /* 10 */
    .long SYMBOL_NAME(sys_execve)
    .long SYMBOL_NAME(sys_chdir)
    .long SYMBOL_NAME(sys_time)
    .long SYMBOL_NAME(sys_mknod)
    .long SYMBOL_NAME(sys_chmod)            /* 15 */
    .long SYMBOL_NAME(sys_lchown16)
    .long SYMBOL_NAME(sys_ni_syscall)                   /* old break syscall holder */
    .long SYMBOL_NAME(sys_stat)
    .long SYMBOL_NAME(sys_lseek)
    .long SYMBOL_NAME(sys_getpid)           /* 20 */
    .long SYMBOL_NAME(sys_mount)
    .long SYMBOL_NAME(sys_oldumount)
    .long SYMBOL_NAME(sys_setuid16)
    .long SYMBOL_NAME(sys_getuid16)
    .long SYMBOL_NAME(sys_stime)            /* 25 */
    .long SYMBOL_NAME(sys_ptrace)
    .long SYMBOL_NAME(sys_alarm)
    .long SYMBOL_NAME(sys_fstat)
    .long SYMBOL_NAME(sys_pause)
    .long SYMBOL_NAME(sys_utime)            /* 30 */
    .long SYMBOL_NAME(sys_ni_syscall)                   /* old stty syscall holder */
    .long SYMBOL_NAME(sys_ni_syscall)                   /* old gtty syscall holder */
    .long SYMBOL_NAME(sys_access)
    .long SYMBOL_NAME(sys_nice)
    .long SYMBOL_NAME(sys_ni_syscall)       /* 35 */    /* old ftime syscall holder */
    .long SYMBOL_NAME(sys_sync)
    .long SYMBOL_NAME(sys_kill)
    .long SYMBOL_NAME(sys_rename)
    .long SYMBOL_NAME(sys_mkdir)
    .long SYMBOL_NAME(sys_rmdir)            /* 40 */
    .long SYMBOL_NAME(sys_dup)
    .long SYMBOL_NAME(sys_pipe)
    .long SYMBOL_NAME(sys_times)
    .long SYMBOL_NAME(sys_ni_syscall)                   /* old prof syscall holder */
    .long SYMBOL_NAME(sys_brk)              /* 45 */
    .long SYMBOL_NAME(sys_setgid16)
    .long SYMBOL_NAME(sys_getgid16)
    .long SYMBOL_NAME(sys_signal)
    .long SYMBOL_NAME(sys_geteuid16)
    .long SYMBOL_NAME(sys_getegid16)        /* 50 */
    .long SYMBOL_NAME(sys_acct)
    .long SYMBOL_NAME(sys_umount)                       /* recycled never used phys() */
    .long SYMBOL_NAME(sys_ni_syscall)                   /* old lock syscall holder */
    .long SYMBOL_NAME(sys_ioctl)
    .long SYMBOL_NAME(sys_fcntl)            /* 55 */
    .long SYMBOL_NAME(sys_ni_syscall)                   /* old mpx syscall holder */
    .long SYMBOL_NAME(sys_setpgid)
    .long SYMBOL_NAME(sys_ni_syscall)                   /* old ulimit syscall holder */
    .long SYMBOL_NAME(sys_olduname)
    .long SYMBOL_NAME(sys_umask)            /* 60 */
    .long SYMBOL_NAME(sys_chroot)
    .long SYMBOL_NAME(sys_ustat)
    .long SYMBOL_NAME(sys_dup2)
    .long SYMBOL_NAME(sys_getppid)
    .long SYMBOL_NAME(sys_getpgrp)          /* 65 */
    .long SYMBOL_NAME(sys_setsid)
    .long SYMBOL_NAME(sys_sigaction)
    .long SYMBOL_NAME(sys_sgetmask)
    .long SYMBOL_NAME(sys_ssetmask)
    .long SYMBOL_NAME(sys_setreuid16)       /* 70 */
    .long SYMBOL_NAME(sys_setregid16)
    .long SYMBOL_NAME(sys_sigsuspend)
    .long SYMBOL_NAME(sys_sigpending)
    .long SYMBOL_NAME(sys_sethostname)
    .long SYMBOL_NAME(sys_setrlimit)        /* 75 */
    .long SYMBOL_NAME(sys_old_getrlimit)
    .long SYMBOL_NAME(sys_getrusage)
    .long SYMBOL_NAME(sys_gettimeofday)
    .long SYMBOL_NAME(sys_settimeofday)
    .long SYMBOL_NAME(sys_getgroups16)      /* 80 */
    .long SYMBOL_NAME(sys_setgroups16)
    .long SYMBOL_NAME(old_select)
    .long SYMBOL_NAME(sys_symlink)
    .long SYMBOL_NAME(sys_lstat)
    .long SYMBOL_NAME(sys_readlink)         /* 85 */
    .long SYMBOL_NAME(sys_uselib)
    .long SYMBOL_NAME(sys_swapon)
    .long SYMBOL_NAME(sys_reboot)
    .long SYMBOL_NAME(old_readdir)
    .long SYMBOL_NAME(old_mmap)             /* 90 */
    .long SYMBOL_NAME(sys_munmap)
    .long SYMBOL_NAME(sys_truncate)
    .long SYMBOL_NAME(sys_ftruncate)
    .long SYMBOL_NAME(sys_fchmod)
    .long SYMBOL_NAME(sys_fchown16)         /* 95 */
    .long SYMBOL_NAME(sys_getpriority)
    .long SYMBOL_NAME(sys_setpriority)
    .long SYMBOL_NAME(sys_ni_syscall)                   /* old profil syscall holder */
    .long SYMBOL_NAME(sys_statfs)
    .long SYMBOL_NAME(sys_fstatfs)          /* 100 */
    .long SYMBOL_NAME(sys_ioperm)
    .long SYMBOL_NAME(sys_socketcall)
    .long SYMBOL_NAME(sys_syslog)
    .long SYMBOL_NAME(sys_setitimer)
    .long SYMBOL_NAME(sys_getitimer)        /* 105 */
    .long SYMBOL_NAME(sys_newstat)
    .long SYMBOL_NAME(sys_newlstat)
    .long SYMBOL_NAME(sys_newfstat)
    .long SYMBOL_NAME(sys_uname)
    .long SYMBOL_NAME(sys_iopl)             /* 110 */
    .long SYMBOL_NAME(sys_vhangup)
    .long SYMBOL_NAME(sys_ni_syscall)                   /* old "idle" system call */
    .long SYMBOL_NAME(sys_vm86old)
    .long SYMBOL_NAME(sys_wait4)
    .long SYMBOL_NAME(sys_swapoff)          /* 115 */
    .long SYMBOL_NAME(sys_sysinfo)
    .long SYMBOL_NAME(sys_ipc)
    .long SYMBOL_NAME(sys_fsync)
    .long SYMBOL_NAME(sys_sigreturn)
    .long SYMBOL_NAME(sys_clone)            /* 120 */
    .long SYMBOL_NAME(sys_setdomainname)
    .long SYMBOL_NAME(sys_newuname)
    .long SYMBOL_NAME(sys_modify_ldt)
    .long SYMBOL_NAME(sys_adjtimex)
    .long SYMBOL_NAME(sys_mprotect)         /* 125 */
    .long SYMBOL_NAME(sys_sigprocmask)
    .long SYMBOL_NAME(sys_create_module)
    .long SYMBOL_NAME(sys_init_module)
    .long SYMBOL_NAME(sys_delete_module)
    .long SYMBOL_NAME(sys_get_kernel_syms)  /* 130 */
    .long SYMBOL_NAME(sys_quotactl)
    .long SYMBOL_NAME(sys_getpgid)
    .long SYMBOL_NAME(sys_fchdir)
    .long SYMBOL_NAME(sys_bdflush)
    .long SYMBOL_NAME(sys_sysfs)            /* 135 */
    .long SYMBOL_NAME(sys_personality)
    .long SYMBOL_NAME(sys_ni_syscall)       /* for afs_syscall */
    .long SYMBOL_NAME(sys_setfsuid16)
    .long SYMBOL_NAME(sys_setfsgid16)
    .long SYMBOL_NAME(sys_llseek)           /* 140 */
    .long SYMBOL_NAME(sys_getdents)
    .long SYMBOL_NAME(sys_select)
    .long SYMBOL_NAME(sys_flock)
    .long SYMBOL_NAME(sys_msync)
    .long SYMBOL_NAME(sys_readv)            /* 145 */
    .long SYMBOL_NAME(sys_writev)
    .long SYMBOL_NAME(sys_getsid)
    .long SYMBOL_NAME(sys_fdatasync)
    .long SYMBOL_NAME(sys_sysctl)
    .long SYMBOL_NAME(sys_mlock)            /* 150 */
    .long SYMBOL_NAME(sys_munlock)
    .long SYMBOL_NAME(sys_mlockall)
    .long SYMBOL_NAME(sys_munlockall)
    .long SYMBOL_NAME(sys_sched_setparam)
    .long SYMBOL_NAME(sys_sched_getparam)   /* 155 */
    .long SYMBOL_NAME(sys_sched_setscheduler)
    .long SYMBOL_NAME(sys_sched_getscheduler)
    .long SYMBOL_NAME(sys_sched_yield)
    .long SYMBOL_NAME(sys_sched_get_priority_max)
    .long SYMBOL_NAME(sys_sched_get_priority_min)  /* 160 */
    .long SYMBOL_NAME(sys_sched_rr_get_interval)
    .long SYMBOL_NAME(sys_nanosleep)
    .long SYMBOL_NAME(sys_mremap)
    .long SYMBOL_NAME(sys_setresuid16)
    .long SYMBOL_NAME(sys_getresuid16)      /* 165 */
    .long SYMBOL_NAME(sys_vm86)
    .long SYMBOL_NAME(sys_query_module)
    .long SYMBOL_NAME(sys_poll)
    .long SYMBOL_NAME(sys_nfsservctl)
    .long SYMBOL_NAME(sys_setresgid16)      /* 170 */
    .long SYMBOL_NAME(sys_getresgid16)
    .long SYMBOL_NAME(sys_prctl)
    .long SYMBOL_NAME(sys_rt_sigreturn)
    .long SYMBOL_NAME(sys_rt_sigaction)
    .long SYMBOL_NAME(sys_rt_sigprocmask)   /* 175 */
    .long SYMBOL_NAME(sys_rt_sigpending)
    .long SYMBOL_NAME(sys_rt_sigtimedwait)
    .long SYMBOL_NAME(sys_rt_sigqueueinfo)
    .long SYMBOL_NAME(sys_rt_sigsuspend)
    .long SYMBOL_NAME(sys_pread)            /* 180 */
    .long SYMBOL_NAME(sys_pwrite)
    .long SYMBOL_NAME(sys_chown16)
    .long SYMBOL_NAME(sys_getcwd)
    .long SYMBOL_NAME(sys_capget)
    .long SYMBOL_NAME(sys_capset)           /* 185 */
    .long SYMBOL_NAME(sys_sigaltstack)
    .long SYMBOL_NAME(sys_sendfile)
    .long SYMBOL_NAME(sys_ni_syscall)               /* streams1 */
    .long SYMBOL_NAME(sys_ni_syscall)               /* streams2 */
    .long SYMBOL_NAME(sys_vfork)            /* 190 */
    .long SYMBOL_NAME(sys_getrlimit)
    .long SYMBOL_NAME(sys_mmap2)
    .long SYMBOL_NAME(sys_truncate64)
    .long SYMBOL_NAME(sys_ftruncate64)
    .long SYMBOL_NAME(sys_stat64)           /* 195 */
    .long SYMBOL_NAME(sys_lstat64)
    .long SYMBOL_NAME(sys_fstat64)
    .long SYMBOL_NAME(sys_lchown)
    .long SYMBOL_NAME(sys_getuid)
    .long SYMBOL_NAME(sys_getgid)           /* 200 */
    .long SYMBOL_NAME(sys_geteuid)
    .long SYMBOL_NAME(sys_getegid)
    .long SYMBOL_NAME(sys_setreuid)
    .long SYMBOL_NAME(sys_setregid)
    .long SYMBOL_NAME(sys_getgroups)        /* 205 */
    .long SYMBOL_NAME(sys_setgroups)
    .long SYMBOL_NAME(sys_fchown)
    .long SYMBOL_NAME(sys_setresuid)
    .long SYMBOL_NAME(sys_getresuid)
    .long SYMBOL_NAME(sys_setresgid)        /* 210 */
    .long SYMBOL_NAME(sys_getresgid)
    .long SYMBOL_NAME(sys_chown)
    .long SYMBOL_NAME(sys_setuid)
    .long SYMBOL_NAME(sys_setgid)
    .long SYMBOL_NAME(sys_setfsuid)         /* 215 */
    .long SYMBOL_NAME(sys_setfsgid)
    .long SYMBOL_NAME(sys_pivot_root)
    .long SYMBOL_NAME(sys_mincore)
    .long SYMBOL_NAME(sys_madvise)
    .long SYMBOL_NAME(sys_getdents64)       /* 220 */
    .long SYMBOL_NAME(sys_fcntl64)
    .long SYMBOL_NAME(sys_ni_syscall)       /* reserved for TUX */
    .long SYMBOL_NAME(sys_ni_syscall)       /* Reserved for Security */
    .long SYMBOL_NAME(sys_gettid)
    .long SYMBOL_NAME(sys_readahead)        /* 225 */
```

### IRQイベント

IRQが上がると、実行中のタスクは中断されIRQハンドラが実行されます。

IRQハンドラの処理が終了すると、何事もなかったかのように、中断された時点に
制御が戻ります。

```
              Running Task
             |-----------|          (3)
NORMAL       |   |       | [break execution] IRQ Handler
EXECUTION (1)|   |       |     ------------->|---------|
             |  \|/      |     |             |  does   |
 IRQ (2)---->| ..        |----->             |  some   |
             |   |       |<-----             |  work   |
BACK TO      |   |       |     |             |  ..(4). |
NORMAL    (6)|  \|/      |     <-------------|_________|
EXECUTION    |___________|  [return to code]
                                    (5)
               USER MODE                     KERNEL MODE

         IRQイベントにより引き起こされるUser->Kernelモード間の移行
```

以下の番号付きステップは上手のイベントシーケンスを参照しています。

1. プロセスが実行中
2. タスクの実行中にIRQが上がる
3. タスクは中断され「割り込みハンドラ」が呼び出される。
4. 「割り込みハンドラ」のコードが実行される。
5. （何もなかったかのように）ユーザモードタスクに制御が戻る。
6. プロセスは通常実行に戻る。

特に注目すべきはタイマIRQであり、TIMER ms毎に割り込みが発生して
以下を管理します。

1. アラーム
2. システムカウンタとタスクカウンタ（スケジューラがプロセスを停止する時間を
   決めたり、統計用に使用される）
3. TIMESLICE時間後に起床させる機構に基づいたマルチタスク

## 3.4 マルチタスク

### 機構

現代OSのキーポイントは「タスク」です。タスクとはCPUやメモリなどのすべての資源を
他のタスクと共有しながらメモリ上で動作するアプリケーションです。

この「資源の共有」は「マルチタスク機構」により管理されています。マルチタスク機構は
「タイムスライス」時間ごとにあるタスクから別のタスクに切り替えます。ユーザーは
すべての資源を独占しているかのように「錯覚」します。また、シングルユーザのシナリオも
想像できます。そこではユーザーは同時に多くのタスクを実行しているかのように「錯覚」
することができます。

このマルチタスクを実現するために、タスクは「状態」変数を使用します。状態は次の
いずれかです。

1. READY: 実行準備完了
2. BLOCKED: 資源を待機中

タスクの状態は対応するリスト（READYリストとBLOCKEDリスト）への登録の有無で管理
されます。

### タスクスイッチ

あるタスクから別のタスクに移ることを「タスクスイッチ」と呼びます。多くのコンピュータ
にはこの操作を自動的に行うハードウェア命令があります。タスクスイッチは以下のような
ケースで発生します。

1. タイムスライスが終了した場合: "Ready for execution"状態のタスクをスケジュール
   して、アクセス権を与える必要があります。
2. タスクがデバイス待ちの場合: 新しいタスクをスケジュールして切り替える必要が
   あります(*)。

* 他の作業を行わず、デバイスを待っている時に発生する"Busy Form Waiting"を防ぐために
  別のタスクをスケジュールします。

タスクスイッチは"Schedule"エンティティによって管理されます。

```
Timer    |           |
 IRQ     |           |                            Schedule
  |      |           |                     ________________________
  |----->|   Task 1  |<------------------>|(1)Chooses a Ready Task |
  |      |           |                    |(2)Task Switching       |
  |      |___________|                    |________________________|
  |      |           |                               /|\
  |      |           |                                |
  |      |           |                                |
  |      |           |                                |
  |      |           |                                |
  |----->|   Task 2  |<-------------------------------|
  |      |           |                                |
  |      |___________|                                |
  .      .     .     .                                .
  .      .     .     .                                .
  .      .     .     .                                .
  |      |           |                                |
  |      |           |                                |
  ------>|   Task N  |<--------------------------------
         |           |
         |___________|

            タイムスライスに基づくタスクスイッチ
 ```

Linuxの一般的なタイムスライはおよそ10msです。



```
 |           |
 |           | Resource    _____________________________
 |   Task 1  |----------->|(1) Enqueue Resource request |
 |           |  Access    |(2)  Mark Task as blocked    |
 |           |            |(3)  Choose a Ready Task     |
 |___________|            |(4)    Task Switching        |
                          |_____________________________|
                                       |
                                       |
 |           |                         |
 |           |                         |
 |   Task 2  |<-------------------------
 |           |
 |           |
 |___________|

     資源待ちに基づくタスクスイッチ
```

## 3.5 マイクロカーネルOS vs モノリシックOS

### 概要

これまでモノリシックOSと呼ばれるものを見てきましたが、もう一つ別の種類のOSがあります。
「マイクロカーネル」OSです。

マイクロカーネルOSでは、ユーザモードプロセスだけでなく、実カーネルマネージャにも
タスクを使用します。Floppy-Task, HDD-Task, Net-Taskなどです。このOSの例としては、
AmoebaやMachがあります。

### マイクロカーネルOSの利点と欠点

利点:

- 各タスクが1種類の処理しか管理しないため、OSのメンテナンスが簡単です。たとえば、
  ネットワークを変更したい場合は、（理論的には構造更新が必要なければ）Net-Taskを
  変更するだけです。

欠点:

- モノリシックOSより性能が悪い。TASK_SWITCHの回数が2倍になる（特定のタスクに入る時に
  1回、出る時にもう1回）からです。

私の個人的な意見ですが、マイクロカーネルは（Minixのように）教科書的な例ととしては
良いですが、「最適」ではないので現実的には適していないと思います。Linuxは「カーネル
スレッド」と呼ばれるいくつかのタスクを使って、ちょっとしたマイクロカーネル構造を
実装しています（たとえば、大容量記憶装置からメモリページを取り出すのに使われるkswapd）。
この場合、スワップは非常に遅い仕事なので性能は問題になりません。

## 3.6 ネットワーク

### ISO OSIレベル

ISO-OSI規格では次のレベルでネットワークアーキテクチャを記述しています。

1. 物理レベル（例: PPP, Ethernet）
2. データリンクレベル（例: PPP, Ethernet）
3. ネットワークレベル（例: IP, X.25）
4. トランスポートレベル（例: TCP, UDP）
5. セッションレベル（SSL）
6. プレゼンテーションレベル（FTP binary-ascii coding）
7. アプリケーションレベル（Netscapeなどのアプリケーション）

この内、最初の2つのレベルは通常ハードウェアで実装され、残りのレベルはソフトウェア
（ルータではファームウェア）で実装されます。

多くのプロトコルがOSでは使用されています。その1つがTCP/IP（3−4レベルにまたがる
もっとも重要なプロトコル）です。

### カーネルはなにをするのか?

カーネルはISO-OSIの最初の2レベルについては（アドレス以外は）何も知りません。

RX（受信）では、OSは以下を行います。

1. 低レベルのデバイス（イーサネットカードやモデムなど）とのハンドシェークを管理して
   「フレーム」を受信します。
2. （イーサネットやPPPの）「フレーム」からTCP/IPの「パケット」を構築します。
3. パケットをソケットに変換して（ポート番号を使って）正しいアプリケーションに渡すか、あるいは
4. パケットを適切なキューに転送します。


```
frames         packets              sockets
NIC ---------> Kernel ----------> Application
                  |    packets
                  --------------> Forward
                        - RX -
```

TX（送信）では、OSは以下を行います。

1.  ソケットを変換するか、あるいは
2.  データをTCP/IP「パケット」にキューイングします。
3.  「パケット」を（イーサネットやPPPの）「フレーム」に分割します。
4.  ハードウェアドライバを使って「フレーム」を送信します。

```
sockets       packets                     frames
Application ---------> Kernel ----------> NIC
              packets     /|\
Forward  -------------------
                        - TX -
```

## 3.7 仮想メモリ

### セグメンテーション

セグメンテーションはメモリ割り当ての問題を解決する最初の方法であり、アプリケーションが
メモリ上のどこに配置されるかを気にせずにソースコードをコンパイルできるようにします。
実際、この機能により、アプリケーション開発者は、OSやハードウェアから独立した形で開発を
進めることができます。

```
            |       Stack        |
            |          |         |
            |         \|/        |
            |        Free        |
            |         /|\        |     Segment <---> Process
            |          |         |
            |        Heap        |
            | Data uninitialized |
            |  Data initialized  |
            |       Code         |
            |____________________|

                   Segment
```

セグメントとはアプリケーションの論理的な実体であり、メモリ上のアプリケーションの
イメージであると言えます。

プログラミングではデータがメモリ上のどこに格納されるかは気にすることなく、セグメント
（アプリケーション）内のオフセットしか気にする必要がありません。

以前は、各プロセスにセグメントを割り当て、その逆もまたしかりでした。Linuxではこれは
真実ではありません。Linuxではカーネルとすべてのプロセスに対して4つのセグメントしか
使用しません。

### セグメンテーションの問題

```
                                 ____________________
                          ----->|                    |----->
                          | IN  |     Segment A      | OUT
 ____________________     |     |____________________|
|                    |____|     |                    |
|     Segment B      |          |     Segment B      |
|                    |____      |                    |
|____________________|    |     |____________________|
                          |     |     Segment C      |
                          |     |____________________|
                          ----->|     Segment D      |----->
                            IN  |____________________| OUT

                     Segmentation problem
```

上図ではプロセスA,Dを終了してプロセスBに入りたいのですが、Bには十分なスペースが
あるのに2つに分割できないのでロードできません（メモリアウト）。

この問題が発生するのは、ピュアセグメントは（論理領域であるため）連続領域であり
分割できないからです。

### ページネーション

 ```
             ____________________
            |     Page 1         |
            |____________________|
            |     Page 2         |
            |____________________|
            |      ..            |     Segment <---> Process
            |____________________|
            |     Page n         |
            |____________________|
            |                    |
            |____________________|
            |                    |
            |____________________|

                   Segment
```

ページネーションはメモリを固定長の"n"個の断片に分割します。

プロセスは1つ以上のPageにロードされます。メモリが解放されるとすべてのページが
解放されます (前述したセグメンテーションの問題を参照してください)。

ページネーションはもうひとつの重要な目的である「スワッピング」にも使用されます。
ページが物理メモリに存在しないと例外を生成し、カーネルにストレージメモリに新しい
ページがないか検索させます。この機構によりOSは物理メモリで許容される以上の
アプリケーションをロードすることができるようになります。

### ページネーションの問題

```
             ____________________
   Page   X |     Process Y      |
            |____________________|
            |                    |
            |       WASTE        |
            |       SPACE        |
            |____________________|

              Pagination Problem
```

上の図では、ページネーションポリシーの何が問題なのかがわかります。プロセスYがページXに
ロードされる際、ページのすべてのメモリスペースが割り当てられ、ページ末尾の残りの
スペースが無駄になります。

### セグメンテーションとページネーション

セグメンテーションとページネーションの問題はどのように解決すればよいのでしょうか。
2つのポリシーを使用します。

```
                                  |      ..            |
                                  |____________________|
                            ----->|      Page 1        |
                            |     |____________________|
                            |     |      ..            |
 ____________________       |     |____________________|
|                    |      |---->|      Page 2        |
|      Segment X     |  ----|     |____________________|
|                    |      |     |       ..           |
|____________________|      |     |____________________|
                            |     |       ..           |
                            |     |____________________|
                            |---->|      Page 3        |
                                  |____________________|
                                  |       ..           |
```

セグメントXで識別されるプロセスXは、3つに分割され、それぞれがページにロードされます。

以下の問題がなくなります。

1. セグメンテーションの問題: ページ単位で割り当て、ページ単位で開放するので、
   空き領域を最適な方法で管理できます。
2. ページネーションの問題: 最後のページだけがスペースを浪費しますが、たとえば、
   4096バイト長のような非常に小さなページを使用し（最大で4096*N_Tasksバイトを
   失います）、（2, 3レベルのページングを使用して）ページングを階層的に管理する
   ことができます。

```
                          |         |           |         |
                          |         |   Offset2 |  Value  |
                          |         |        /|\|         |
                  Offset1 |         |-----    | |         |
                      /|\ |         |    |    | |         |
                       |  |         |    |   \|/|         |
                       |  |         |    ------>|         |
                      \|/ |         |           |         |
 Base Paging Address ---->|         |           |         |
                          | ....... |           | ....... |
                          |         |           |         |

                     Hierarchical Paging
```
