# Lab6 ドライバとlibc

[TOC]

## 6.1 実験内容の紹介

### 6.1.1 実験の目的

この実験では、簡単なsdカードドライバを実装し、MBRパーティション形式を理解し、
libcとオペレーティングシステムのインターフェースについて学習します。

### 6.1.2 準備作業

まず、今回の実習のために追加されたコードを、以下のように開発ブランチに
取り込みます。

```bash
git pull
git rebase origin/lab6
# Or git merge origin/lab6
```

conflictが発生した場合は、gitの指示に従って修正してください。

- SDカードイメージ作成用の`mtools`をインストールしてください（Ubuntuでは
  `sudo apt install -y mtools`を実行）。
- `make init`を実行して、libcを初期化します（1回だけ）。
- その後、`make`を実行すれば普通にカーネルがコンパイルされますが、最初にlibcを
  コンパイルするときには時間がかかります。どうしても待てない場合には、`make -j10`
  コマンドで並列コンパイルをすることができます。

## 6.2 I/Oフレームワーク

ここでは、シンプルなキューベースの非同期I/Oフレームワークについて抽象的に
説明します。

6.2.1 非同期Read/Write

デバイスドライバはデバイスの読み書きを実装する必要がある場合が多いですが、
I/Oは遅いので、カレントCPUをブロックしないように非同期で実装します。

- Read: Read要求とターゲットアドレスをデバイスに送信し、デバイスからの
  Read完了割り込みが発生するまで待機し、デバイスから返されたデータを読み込みます。
- write: デバイスにWrite要求を送り、ターゲットアドレスと書き込むデータを指定し、
  デバイスからのWrite完了割り込みを待ちます。

もちろん、繰り返し割り込みが発生しないように、割り込み後に対応するフラグを
クリアすることや、エラー割り込みが発生した場合には、現在の動作を再実行する
ことなど、細かな配慮が必要です。

### 6.2.2 リクエストキュー

複数のプロセスがI/Oを利用する場合にはリクエストキューが必要です。 プロデューサで
あるプロセスは、リクエストバッファをキューに入れ、ドライバが起こすまでブロック
（スリープ）します。 コンシューマであるI/Oドライバは、毎回、キューの先頭にある
バッファを取り出して解析し、デバイスに対して指定の読み書き操作を発行し、それが
終わったら対応するプロセスを起こします。

### 6.2.3 演習問題1

SDカードドライバのリクエストキューの実装については`inc/buf.h`を参照してください。
各リクエストはbufであり、すべてのリクエストは一つのキューに入れられます。 キューの
実装は手書きでも構いませんが、再利用しやすいように別のヘッダファイルに展開することを
勧めます。詳細は[list.h](https://github.com/torvalds/linux/blob/master/include/linux/list.h)を
参照してください。

## 6.3 ブロックデバイスドライバ

ブロックデバイスでは、次の用語を使います。

- ブロックサイズ: 各ブロックの大きさで、通常は512バイトです。
- LBA（[Logical block addressing](https://wiki.osdev.org/LBA): 0から始まるブロックの
  インデックスです。

他のカーネルでは、非同期ブロックデバイスドライバは通常、3つの関数、すなわち、
初期化関数`sd_init`、デバイスRead/Write要求関数`sd_rw`、デバイス割り込み処理関数
`sd_intr`で構成されています。デバイスの初期化はハードウェア依存の部分が多いので、
今回は省略して、読み書き関数から説明します。

### 6.3.1 Read/Write要求

これは、他のカーネルモジュール（ファイルシステムなど）がデバイスを利用するための
インタフェースで、バッファを受け取り、6.2.1と同様の手順を実行し、待機プロセスの
sleep/wakeupを実行します。

### 6.3.2 Sleep

sleepとwakeupは、相互排他ロックの実装です。条件変数ついて、これまで学んだことを
確認してください。これはオペレーティングシステムからユーザープログラムに提供される
[同期プリミティブ](https://en.wikipedia.org/wiki/Event_synchronization_primitive)で
あり、libcではfutexというシステムコールに依存しています（`libc/src/thread/pthread_cond_*.c`を参照。ただし、カーネルではこのシステムコールはまだ実装されて
いません）。sleep関数とwakeup関数でプロセスのスリープ待ちやウェイクアップを行います。

この2つの操作は条件変数である[wait](https://linux.die.net/man/3/pthread_cond_wait)と
[signal](https://linux.die.net/man/3/pthread_cond_signal) の操作に似ています。
sleepの関数宣言は `void sleep(void *chan, struct spinlock *lk)` ですが、最初の引数は
待つべきリソースのアドレスを参照しています（なぜこのような設計になっているかというと、リソースがメモリ上にある限り、そのアドレスを常に固有の識別子として使うことができる
からです）、第2引数はロック（sleepが呼ばれたときからプロセスのsleepがアトミックで
あることを保証するため）です。wakeup関数についても同じです。

### 6.3.3 デバイス割り込み

デバイス割り込みが到着したということは、バッファキューの先頭にあるリクエストが
実行されたことを意味するので、キューの先頭にあるリクエストの種類に応じて処理を
行います。

- Read要求の場合、正当な割り込みはデバイスからデータを読み込んだことを意味し、
  MMIOからバッファのデータフィールドに読み込むことができます。
- Write要求の場合、正当な割り込みは、デバイスにデータが書き込まれたことを意味します。

その後、割り込みフラグをクリアし、次のバッファのリード/ライト要求（もしあれば）を
続行します。

### 6.3.4 演習問題2

`kern/proc.c`の`sleep`関数と`wakeup`関数を完成させ、その設計を簡単に説明してください。

### 6.3.5 演習問題3

`kern/sd.c`の`sd_init`、`sd_intr`、`sd_rw`の各関数を完成させ、適切な場所で`sd_init`
と`sd_test`を呼び出して、SDカードの初期化を完了させ、テストに合格させてください。

## 6.4 ブートディスクの作成

最近のOSは通常、ハードディスクのイメージとして配布されていますが、私たちのOSも
例外ではありません。 しかし、イメージを作成する際には、Raspberry Piのルールに従う
よう注意する必要があります。つまり、最初のパーティションはブートパーティションであり、
ファイルシステムはFAT32でなければなりません。残りのパーティションは自由に割り当てる
ことができます。簡単のために、[マスタブートレコード, MBR](https://en.wikipedia.org/wiki/Master_boot_record)を使用し、最初のパーティションと2番目のパーティションは
約64MBとして、2番目のパーティションはルートディレクトリが配置したファイルシステムと
します（この部分は次のラボで完成させます）。つまり、SDカード上のレイアウトは次の
ようになります。`mksd.mk`を参照してください。

```
 512B            FAT32       Your fancy fs
+-----+-----+--------------+----------------+
| MBR | ... | boot partion | root partition |
+-----+-----+--------------+----------------+
 \   1MB   / \    64MB    / \     63MB     /
  +-------+   +----------+   +------------+
```

### 6.4.1 MBR

MBRはデバイスの最初の512バイトに配置され、様々なフォーマットがあります。
類似していますが、共通のフォーマットを以下に示します。

| アドレス | 説明                            | サイズ（バイト） |
| ------- | ---------------------------------------- | ------------ |
| 0x0     | Bootstrap code area and disk information | 446          |
| 0x1BE   | Partition entry 1                        | 16           |
| 0x1CE   | Partition entry 2                        | 16           |
| 0x1DE   | Partition entry 3                        | 16           |
| 0x1EE   | Partition entry 4                        | 16           |
| 0x1FE   | 0x55                                     | 1            |
| 0x1FF   | 0xAA                                     | 1            |

ここでは2番めのパーティションの情報だけが必要です。つまり、上の表のパーティション
エントリ2の情報です。この16Bには、パーティションの開始LBAやサイズ（合計ブロック数）
などのパーティション情報が含まれています。

| オフセット（バイト） | フィールド長（バイト） | 説明                           |
| -------------- | -------------------- | ---------------------------------------|
| ...            | ...                  | ...                                    |
| 0x8            | 4                    | パーティションの最初の絶対セクタのLBA  |
| 0xC            | 4                    | パーティションのセクタ数               |

### 6.4.2 ブートパーティション

これは、Raspberry Pi関連のパーティションであり、mtoolsを使用して`boot/`ディレクトリ
配下に、公式のファームウェアと設定ファイル`config.txt`、カーネルイメージ
`obj/kernel8.img`をコピーします。

### 6.4.3 ルートパーティション

ファイルシステムを置く場所です。（次回のラボで）ファイルシステムの設計と構築を
行いますが、コマンドラインツールである`user/src/mkfs`を参照してください。

- `Makefile`: `user/Makefile`を呼び出し、`user/src/XXX`ディレクトリ内のすべての
  ファイルをクロスコンパイラで個別にコンパイルし、実行ファイル`user/bin/XXを
  作成します。
- `mksd.mk`: `user/src/mksd`を開発マシンのコンパイラでコンパイルして`obj/mksd`とし、
  `obj/mksd obj/fs.img user/bin/A user/bin/B ...`を実行します。これにより、
  `user/bin/A`、`user/bin/B`などのファイルを含む2番めのパーティションイメージ
  `obj/fs.img`を作成します。
- `user/src/mksd`の現在の実装は、すべてのファイルを連続して`fs.img`に書き込むと
  いう、かなり素朴なものです。次回のラボでは、自分で設計したファイルシステムの
  フォーマットに合わせて修正する必要があります。

### 6.4.4 演習問題4

`sd_init`でMBRを解析して、2番めのパーティションの開始ブロックのLBAと
パーティションサイズを取得し、以降の使用に役立ててください。



## 6.5 libc

Cライブラリを書きたくなかったので、完全なCライブラリである[musl](https://musl.libc.org/)を
[git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules)として
移植して使用しました。 対応するシステムコールを実装するだけで、本当に使えるOSを
実現することができます。

## 6.5.1 プログラムエントリ

カーネル内でユーザログラムを実行する際、PCの値はどのように初期化するのか。つまり、
ユーザプログラムの実行を開始するエントリポイントはどこに指定されているのか。この
情報は、コンパイラによってELFヘッダにコンパイルされます。このヘッダのデフォルトは、
GCCの_startシンボルですが、リンカスクリプトの[ENTRY](https://sourceware.org/binutils/docs/ld/Entry-Point.html)
フィールドでカスタマイズすることができます。カーネルローダはこのELFヘッダを解析する
ことでエントリポイントを知ることができます。

### 6.5.2 crt

Cランタイムはメイン関数の前後に実行されるコードスニペットであり、gccがデフォルトで
実行ファイルにリンクします（例：libc/lib/crt1.o）。これはコンパイラに関連する
ものですが、他の言語も独自のランタイムを持っている場合があります。例えば、C++は、
例外処理やメモリ管理のためのランタイムサポート（グローバルコンストラクタと
デストラクタ）を必要とします。mainの前に実行されるランタイムは、以下のファイルを
順番に見ていく必要があります。

- libc/arch/aarch64/crt_arch.h
- libc/crt/crt1.c
- libc/src/env/__libc_start_main.c

このコードには、BSSセグメントのクリアや[TLS](https://en.wikipedia.org/wiki/Thread-local_storage)の
初期化などの初期化コードが含まれて
います。完全な起動プロセスには複数のシステムコールの実装が必要なので、まずは
テスト用にcrt1.cを修正します。

## 6.5.3 システムコール

カーネルと C ライブラリはシステムコールによって相互作用します。 muslのシステム
コールの実装は`libc/arch/aarch64/syscall_arch.h`にあります。 カーネルについては
以下の情報が必要です。

- syscall番号：libc/obj/include/bits/syscall.h にあります。
- syscall関数宣言：libc/include/unistd.h にありますが、スケジューリング関連のものは別の
  場所にあります。

printfを使用するには、対応するシステムコールが何かを知る必要があります。すべてが
ファイルであるUnixシステムでは、printfは実際にはstdoutファイルに文字列を書き込む
だけです（文字列中の%dフォーマットなど、libcが多くの仕事をしていることに注意して
ください）。libc/src/stdioディレクトリの中をstdoutで検索してください。

## 実行結果

### `mtools`のインストール

```
$ sudo apt install mtools
Reading package lists... Done
Building dependency tree
Reading state information... Done
mtools is already the newest version (4.0.24-1).
mtools set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```

### libcの初期化

```
$ make init
find: ‘obj/user/bin’: No such file or directory
git submodule update --init --recursive
Submodule 'libc' (git://git.musl-libc.org/musl) registered for path 'libc'
Cloning into '/home/vagrant/xv6-armv8-lab/libc'...
Submodule path 'libc': checked out '821083ac7b54eaa040d5a8ddc67c6206a175e0ca'
(cd libc && export CROSS_COMPILE=aarch64-linux-gnu- && ./configure --target=)
checking for C compiler... aarch64-linux-gnu-gcc
checking whether C compiler works... yes
checking whether compiler accepts -Werror=unknown-warning-option... no
checking whether compiler accepts -Werror=unused-command-line-argument... no
checking whether compiler accepts -Werror=ignored-optimization-argument... no
checking whether linker accepts -Werror=unknown-warning-option... no
checking whether linker accepts -Werror=unused-command-line-argument... no
checking for C compiler family... gcc
checking for toolchain wrapper to build... gcc
checking target system type... aarch64-linux-gnu
checking whether compiler accepts -std=c99... yes
checking whether compiler accepts -nostdinc... yes
checking whether compiler accepts -ffreestanding... yes
checking whether compiler accepts -fexcess-precision=standard... yes
checking whether compiler accepts -frounding-math... yes
checking whether compiler needs attribute((may_alias)) suppression... no
checking whether compiler accepts -Wa,--noexecstack... yes
checking whether compiler accepts -fno-stack-protector... yes
checking whether compiler accepts -fno-tree-loop-distribute-patterns... yes
checking whether we should preprocess assembly to add debugging information... no
checking for optimization settings... using defaults
checking whether compiler accepts -Os... yes
components to be optimized for speed: internal malloc string
checking whether compiler accepts -pipe... yes
checking whether compiler accepts -fomit-frame-pointer... yes
checking whether compiler accepts -fno-unwind-tables... yes
checking whether compiler accepts -fno-asynchronous-unwind-tables... yes
checking whether compiler accepts -ffunction-sections... yes
checking whether compiler accepts -fdata-sections... yes
checking whether compiler accepts -Wno-pointer-to-int-cast... yes
checking whether compiler accepts -Werror=implicit-function-declaration... yes
checking whether compiler accepts -Werror=implicit-int... yes
checking whether compiler accepts -Werror=pointer-sign... yes
checking whether compiler accepts -Werror=pointer-arith... yes
checking whether compiler accepts -Werror=int-conversion... yes
checking whether compiler accepts -Werror=incompatible-pointer-types... yes
checking whether compiler accepts -Werror=discarded-qualifiers... yes
checking whether compiler accepts -Werror=discarded-array-qualifiers... yes
checking whether compiler accepts -Waddress... yes
checking whether compiler accepts -Warray-bounds... yes
checking whether compiler accepts -Wchar-subscripts... yes
checking whether compiler accepts -Wduplicate-decl-specifier... yes
checking whether compiler accepts -Winit-self... yes
checking whether compiler accepts -Wreturn-type... yes
checking whether compiler accepts -Wsequence-point... yes
checking whether compiler accepts -Wstrict-aliasing... yes
checking whether compiler accepts -Wunused-function... yes
checking whether compiler accepts -Wunused-label... yes
checking whether compiler accepts -Wunused-variable... yes
checking preprocessor condition __PIC__... true
checking whether linker accepts -Wl,--sort-section,alignment... yes
checking whether linker accepts -Wl,--sort-common... yes
checking whether linker accepts -Wl,--gc-sections... yes
checking whether linker accepts -Wl,--hash-style=both... yes
checking whether linker accepts -Wl,--no-undefined... yes
checking whether linker accepts -Wl,--exclude-libs=ALL... yes
checking whether linker accepts -Wl,--dynamic-list=./dynamic.list... yes
checking whether linker accepts -lgcc... yes
checking whether linker accepts -lgcc_eh... yes
using compiler runtime libraries: -lgcc -lgcc_eh
checking preprocessor condition __AARCH64EB__... false
checking whether compiler's long double definition matches float.h... yes
checking preprocessor condition __FAST_MATH__... false
creating config.mak... done
```

### `make`

```
$ make
find: ‘obj/user/bin’: No such file or directory
make -C user
make[1]: Entering directory '/home/vagrant/xv6-armv8-lab/user'
make -C ../libc
make[2]: Entering directory '/home/vagrant/xv6-armv8-lab/libc'
mkdir -p lib
mkdir -p obj
mkdir -p obj/crt
mkdir -p obj/crt/aarch64
mkdir -p obj/include
mkdir -p obj/include/bits
mkdir -p obj/ldso
mkdir -p obj/src/aio
mkdir -p obj/src/complex
mkdir -p obj/src/conf
mkdir -p obj/src/crypt
mkdir -p obj/src/ctype
mkdir -p obj/src/dirent
mkdir -p obj/src/env
mkdir -p obj/src/errno
mkdir -p obj/src/exit
mkdir -p obj/src/fcntl
mkdir -p obj/src/fenv
mkdir -p obj/src/fenv/aarch64
mkdir -p obj/src/internal
mkdir -p obj/src/ipc
mkdir -p obj/src/ldso
mkdir -p obj/src/ldso/aarch64
mkdir -p obj/src/legacy
mkdir -p obj/src/linux
mkdir -p obj/src/locale
mkdir -p obj/src/malloc
mkdir -p obj/src/malloc/mallocng
mkdir -p obj/src/math
mkdir -p obj/src/math/aarch64
mkdir -p obj/src/misc
mkdir -p obj/src/mman
mkdir -p obj/src/mq
mkdir -p obj/src/multibyte
mkdir -p obj/src/network
mkdir -p obj/src/passwd
mkdir -p obj/src/prng
mkdir -p obj/src/process
mkdir -p obj/src/regex
mkdir -p obj/src/sched
mkdir -p obj/src/search
mkdir -p obj/src/select
mkdir -p obj/src/setjmp/aarch64
mkdir -p obj/src/signal
mkdir -p obj/src/signal/aarch64
mkdir -p obj/src/stat
mkdir -p obj/src/stdio
mkdir -p obj/src/stdlib
mkdir -p obj/src/string
mkdir -p obj/src/string/aarch64
mkdir -p obj/src/temp
mkdir -p obj/src/termios
mkdir -p obj/src/thread
mkdir -p obj/src/thread/aarch64
mkdir -p obj/src/time
mkdir -p obj/src/unistd
sed -f ./tools/mkalltypes.sed ./arch/aarch64/bits/alltypes.h.in ./include/alltypes.h.in > obj/include/bits/alltypes.h
cp arch/aarch64/bits/syscall.h.in obj/include/bits/syscall.h
sed -n -e s/__NR_/SYS_/p < arch/aarch64/bits/syscall.h.in >> obj/include/bits/syscall.h
aarch64-linux-gnu-gcc -std=c99 -nostdinc -ffreestanding -fexcess-precision=standard -frounding-math -Wa,--noexecstack -D_XOPEN_SOURCE=700 -I./arch/aarch64 -I./arch/generic -Iobj/src/internal -I./src/include -I./src/internal -Iobj/include -I./include  -Os -pipe -fomit-frame-pointer -fno-unwind-tables -fno-asynchronous-unwind-tables -ffunction-sections -fdata-sections -Wno-pointer-to-int-cast -Werror=implicit-function-declaration -Werror=implicit-int -Werror=pointer-sign -Werror=pointer-arith -Werror=int-conversion -Werror=incompatible-pointer-types -Werror=discarded-qualifiers -Werror=discarded-array-qualifiers -Waddress -Warray-bounds -Wchar-subscripts -Wduplicate-decl-specifier -Winit-self -Wreturn-type -Wsequence-point -Wstrict-aliasing -Wunused-function -Wunused-label -Wunused-variable  -fPIC -fno-stack-protector -DCRT -c -o obj/crt/Scrt1.o crt/Scrt1.c
cp obj/crt/Scrt1.o lib/Scrt1.o
...
aarch64-linux-gnu-ar rc lib/libm.a
rm -f lib/librt.a
aarch64-linux-gnu-ar rc lib/librt.a
rm -f lib/libpthread.a
aarch64-linux-gnu-ar rc lib/libpthread.a
rm -f lib/libcrypt.a
aarch64-linux-gnu-ar rc lib/libcrypt.a
rm -f lib/libutil.a
aarch64-linux-gnu-ar rc lib/libutil.a
rm -f lib/libxnet.a
aarch64-linux-gnu-ar rc lib/libxnet.a
rm -f lib/libresolv.a
aarch64-linux-gnu-ar rc lib/libresolv.a
rm -f lib/libdl.a
aarch64-linux-gnu-ar rc lib/libdl.a
sh tools/musl-gcc.specs.sh "/usr/local/musl/include" "/usr/local/musl/lib" "/lib/ld-musl-aarch64.so.1" > lib/musl-gcc.specs
printf '#!/bin/sh\nexec "${REALGCC:-aarch64-linux-gnu-gcc}" "$@" -specs "%s/musl-gcc.specs"\n' "/usr/local/musl/lib" > obj/musl-gcc
chmod +x obj/musl-gcc
make[2]: Leaving directory '/home/vagrant/xv6-armv8-lab/libc'
mkdir -p ../obj/user/
# Replace "/usr/local/musl" to "../libc"
sed -e "s/\/usr\/local\/musl/..\/libc/g" ../libc/lib/musl-gcc.specs > ../obj/user/musl-gcc.specs
make ../obj/user/bin/mkfs ../obj/user/bin/sh
make[2]: Entering directory '/home/vagrant/xv6-armv8-lab/user'
mkdir -p ../obj/user/src/mkfs/
aarch64-linux-gnu-gcc -specs ../obj/user/musl-gcc.specs -std=c99 -O3 -MMD -MP -static -I../libc/obj/include/ -c -o ../obj/user/src/mkfs/main.c.o src/mkfs/main.c
mkdir -p ../obj/user/bin/
aarch64-linux-gnu-gcc -specs ../obj/user/musl-gcc.specs -std=c99 -O3 -MMD -MP -static -I../libc/obj/include/ -o ../obj/user/bin/mkfs ../obj/user/src/mkfs/main.c.o
aarch64-linux-gnu-objdump -S -d ../obj/user/bin/mkfs > ../obj/user/src/mkfs/mkfs.asm
aarch64-linux-gnu-objdump -x ../obj/user/bin/mkfs > ../obj/user/src/mkfs/mkfs.hdr
mkdir -p ../obj/user/src/sh/
aarch64-linux-gnu-gcc -specs ../obj/user/musl-gcc.specs -std=c99 -O3 -MMD -MP -static -I../libc/obj/include/ -c -o ../obj/user/src/sh/main.c.o src/sh/main.c
mkdir -p ../obj/user/bin/
aarch64-linux-gnu-gcc -specs ../obj/user/musl-gcc.specs -std=c99 -O3 -MMD -MP -static -I../libc/obj/include/ -o ../obj/user/bin/sh ../obj/user/src/sh/main.c.o
aarch64-linux-gnu-objdump -S -d ../obj/user/bin/sh > ../obj/user/src/sh/sh.asm
aarch64-linux-gnu-objdump -x ../obj/user/bin/sh > ../obj/user/src/sh/sh.hdr
rm ../obj/user/src/mkfs/main.c.o ../obj/user/src/sh/main.c.o
make[2]: Leaving directory '/home/vagrant/xv6-armv8-lab/user'
make[1]: Leaving directory '/home/vagrant/xv6-armv8-lab/user'
make obj/sd.img
make[1]: Entering directory '/home/vagrant/xv6-armv8-lab'
+ cc kern/vm.c
+ cc kern/trap.c
+ cc kern/spinlock.c
+ cc kern/console.c
+ cc kern/sleeplock.c
+ cc kern/proc.c
+ cc kern/bio.c
+ cc kern/syscall.c
+ cc kern/sysfile.c
+ cc kern/kalloc.c
+ cc kern/mbox.c
+ cc kern/uart.c
+ cc kern/sysproc.c
+ cc kern/clock.c
+ cc kern/timer.c
+ cc kern/main.c
+ cc kern/sd.c
+ ld obj/kernel8.elf
+ objdump obj/kernel8.elf
+ objcopy obj/kernel8.img
dd if=/dev/zero of=obj/boot.img seek=$((128*1024 - 1)) bs=512 count=1
1+0 records in
1+0 records out
512 bytes copied, 8.5489e-05 s, 6.0 MB/s
# -F 32 specify FAT32
# -s 1 specify one sector per cluster so that we can create a smaller one
mkfs.vfat -F 32 -s 1 obj/boot.img
mkfs.fat 4.1 (2017-01-24)
# Copy files into boot partition
mcopy -i obj/boot.img obj/kernel8.img ::kernel8.img;  mcopy -i obj/boot.img boot/LICENCE.broadcom ::LICENCE.broadcom;  mcopy -i obj/boot.img boot/bootcode.bin ::bootcode.bin;  mcopy -i obj/boot.img boot/config.txt ::config.txt;  mcopy -i obj/boot.img boot/fixup.dat ::fixup.dat;  mcopy -i obj/boot.img boot/start.elf ::start.elf;
echo obj/user/bin/mkfs obj/user/bin/sh
obj/user/bin/mkfs obj/user/bin/sh
cc user/src/mkfs/main.c -o obj/mkfs
./obj/mkfs obj/fs.img obj/user/bin/mkfs obj/user/bin/sh
mkfs: hello, world 4
arg[0] = "./obj/mkfs"
arg[1] = "obj/fs.img"
arg[2] = "obj/user/bin/mkfs"
arg[3] = "obj/user/bin/sh"
dd if=/dev/zero of=obj/sd.img seek=$((256*1024 - 1)) bs=512 count=1
1+0 records in
1+0 records out
512 bytes copied, 7.9065e-05 s, 6.5 MB/s
printf "                                                                \
  2048, $((128*1024*512/1024))K, c,\n      \
  $((2048+128*1024)), $(($((256*1024-$((2048+128*1024))))*512/1024))K, L,\n          \
" | sfdisk obj/sd.img
Checking that no-one is using this disk right now ... OK

Disk obj/sd.img: 128 MiB, 134217728 bytes, 262144 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Created a new DOS disklabel with disk identifier 0x1f5fb71a.
obj/sd.img1: Created a new partition 1 of type 'W95 FAT32 (LBA)' and of size 64 MiB.
obj/sd.img2: Created a new partition 2 of type 'Linux' and of size 63 MiB.
obj/sd.img3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0x1f5fb71a

Device      Boot  Start    End Sectors Size Id Type
obj/sd.img1        2048 133119  131072  64M  c W95 FAT32 (LBA)
obj/sd.img2      133120 262143  129024  63M 83 Linux

The partition table has been altered.
Syncing disks.
dd if=obj/boot.img of=obj/sd.img seek=2048 conv=notrunc
131072+0 records in
131072+0 records out
67108864 bytes (67 MB, 64 MiB) copied, 0.346607 s, 194 MB/s
dd if=obj/fs.img of=obj/sd.img seek=$((2048+128*1024)) conv=notrunc
137+1 records in
137+1 records out
70464 bytes (70 kB, 69 KiB) copied, 0.000427871 s, 165 MB/s
make[1]: Leaving directory '/home/vagrant/xv6-armv8-lab'
```

### `make qemu`

```
$ make qemu
qemu-system-aarch64 -M raspi3 -nographic -serial null -serial mon:stdio -drive file=obj/sd.img,if=sd,format=raw -kernel obj/kernel8.img
console_init: success.
main: [CPU 0] init started.
alloc_init: success.
proc_init: success.
irq_init: success.
timer_init: success at CPU 0.
proc_alloc: proc 1 success.
user_init: proc 1 (initproc) success.
- mbox write: 0x7fd28
- mbox read: 0x7fd28
- clock rate: 50000000
- SD base clock rate from mailbox: 50000000
- Reset the card.
- Divisor selected = 104, shift count = 6
- EMMC: Set clock, status 0x1ff0000 CONTROL1: 0xe6807
- Send IX_GO_IDLE_STATE command.
- Send command response: 0
- EMMC: Sending ACMD41 SEND_OP_COND status 1ff0000
- Divisor selected = 2, shift count = 0
- EMMC: Set clock, status 0x1ff0000 CONTROL1: 0xe0207
- EMMC: SD Card Type 2 SC 128Mb UHS-I 0 mfr 170 'XY:QEMU!' r0.1 2/2006, #deadbeef RCA 4567
sd_init: Partition 1: 00 20 21 00 0c 49 01 08 00 08 00 00 00 00 02 00
- Status: 0
- CHS address of first absolute sector: head=32, sector=33, cylinder=0
- Partition type: 12
- CHS address of last absolute sector: head=73, sector=1, cylinder=8
- LBA of first absolute sector: 0x800
- Number of sectors: 131072
sd_init: Partition 2: 00 49 02 08 83 51 01 10 00 08 02 00 00 f8 01 00
- Status: 0
- CHS address of first absolute sector: head=73, sector=2, cylinder=8
- Partition type: 131
- CHS address of last absolute sector: head=81, sector=1, cylinder=16
- LBA of first absolute sector: 0x20800
- Number of sectors: 129024
sd_init: Partition 3: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
- Status: 0
- CHS address of first absolute sector: head=0, sector=0, cylinder=0
- Partition type: 0
- CHS address of last absolute sector: head=0, sector=0, cylinder=0
- LBA of first absolute sector: 0x0
- Number of sectors: 0
sd_init: Partition 4: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
- Status: 0
- CHS address of first absolute sector: head=0, sector=0, cylinder=0
- Partition type: 0
- CHS address of last absolute sector: head=0, sector=0, cylinder=0
- LBA of first absolute sector: 0x0
- Number of sectors: 0
sd_init: Boot signature: 55 aa
sd_init: success.
sd_test: begin nblocks 2048
sd_test: sd check rw...
sd_test: read 1048576 B (1 MB), t: 30025429 cycles, speed: 2.0 MB/s
sd_test: write 1048576 B (1 MB), t: 24431822 cycles, speed: 2.5 MB/s
main: [CPU 0] init success.
scheduler: switch to proc 1 at CPU 0.
timer: CPU 0 timer.
yield: proc 1 gives up CPU 0.
scheduler: switch to proc 1 at CPU 0.
main: [CPU 3] init started.
clock: CPU 0 clock.
timer_init: success at CPU 3.
main: [CPU 3] init success.
syscall: proc 1 calls syscall 0.
sys_exec: executing /init with parameters: /init
syscall: proc 1 calls syscall 1.
sys_exit: in exit.
main: [CPU 1] init started.
timer_init: success at CPU 1.
main: [CPU 1] init success.
main: [CPU 2] init started.
timer_init: success at CPU 2.
main: [CPU 2] init success.
```
