# Lab 1 ブート

この実習では、カーネルを正式に起動します。ARM v8アーキテクチャ、Raspberry Pi 3のブートプロセス、
リンカスクリプトについて学んだ後、カーネルのCランタイム環境を提供し、Cコードに飛び込みます。

## 1.1 ARM v8

ARMはMIPSと同じ縮小命令アーキテクチャ（RISC）であり、複雑な命令アーキテクチャ（X86などのCISC）に
比べて、よりシンプルな命令とアクセスモデル（LOAD/STORE）を提供します。 たとえば、X86のINCはメモリ
上の値に直接1を加えますが、ARMでは、レジスタに値を読み込む`LOAD`、レジスタに1を加える`ADD`、
レジスタをメモリに戻す`STORE`が必要です。 CISCの命令は強力ですが、RISCの簡単な命令操作はクロック
サイクルを短縮し、ハードウェアの実行を高速化することができます。

ARMアーキテクチャについての簡単な紹介はStanford大学の*Phase1: ARM and a Leg (Subphase A-C)*
[^Stanford] にあります。 *A Guide to ARM64 / AArch64 Assembly on Linux with Shellcodes and
Cryptography*[^Assembly]はARMアーキテクチャに関するコンテンツも充実しています。

[^Stanford]: https://cs140e.sergio.bz/assignments/3-spawn/
[^Assembly]: https://modexp.wordpress.com/2018/10/30/arm64-assembly/

## 1.2 Raspberry Pi 3BCM2837

armアーキテクチャだけでなく、Raspberry Pi 3ボード（BCM2837ベース）のハードウェア情報についても
知る必要がありますが、今回の実験では以下の2点だけを知っていれば十分です。

- どうやって起動するのか。つまり、Raspberry Piはどのようにカーネルをロードするのか。
- 使用可能な物理メモリ（ユーザプロセスへのメモリの割り当てなど）とデバイス関連のMMIO
  （Memory-mapped IO）とはなにか。

### 1.2.1 ブートプロセス

qemuのRaspberry Pi 3エミュレーションでは、単にkernel8.imgを0x80000のメモリアドレスに
コピーして、0x80000にジャンプして実行を開始しています。

実際のRaspberry Piは、GPU ROM（BIOSに似ています）から起動した後、SDカードの最初の
パーティション（FAT32パーティションでなければなりません）を読み、bootcode.bin、start.elf、
カーネルイメージの順に読み込んで実行し、カーネルイメージをメモリに直接ロードしています。
bootcode.binとstart.elfは、オープンソースではない公式の[Boot Loader](https://github.com/raspberrypi/firmware/tree/master/boot)で
あると解釈できますが、いくつかの設定（カーネルイメージの名前やメモリ内の位置など）を行う
ことができます。詳細は[Boot Folder](https://www.raspberrypi.org/documentation/configuration/boot_folder.md)
を参照してください。

### 1.2.2 メモリレイアウト

| ARMの物理アドレス        | 説明                      |
| ------------------------ | -------------------------- |
| [0x0, 3F000000)          | Free memory                |
| [0x3F000000, 0x40000000) | MMIO                       |
| [0x40000000, 0x40020000) | ARM timer, IRQs, mailboxes |

今回使用する必要のあるアドレスは上のとおりです。詳細については[BCM2837](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bcm2837/README.md)を
参照してください。

## 1.3 ELFとリンカスクリプト

リンクプロセスについて詳しくない場合は、最初にCSAPPの第7章リンクを確認してください。

### 1.3.1 リンカスクリプト

リンカスクリプト（kern/linker.ld）を使用して、次の2つの問題を解決します。

1. `b main`（mainという名前の関数にジャンプする）のような命令は、コンパイラにとって
   どのようにコンパイルされるべきでしょうか？ 当然のことながら、対応するマシンコードを
   生成するには、main関数のアドレスを知る必要があります。これらのアドレスを制御する
   ためにリンカースクリプトが使われています。
2. Raspberry Piのブートプロセスをおさらいすると、カーネルがロードされるとkernel8.imgが
   物理アドレス0x80000に直接コピーされ、0x80000から実行されます。つまり、kernel8.imgの
   先頭は、カーネルの「エントリ」（kern/entry.S）です。言い換えれば、entry.Sをカーネル
   イメージの先頭に置くにはobjcopyでELFヘッダを削除する必要があります。

以下では、使用した構文のみを紹介します。詳細は[Linker Scripts](https://sourceware.org/binutils/docs/ld/Scripts.html)を
参照してください。

- `.`はロケーションカウンタと呼ばれ、コードがメモリ上のどの位置で実際に実行されるかを
  表します。たとえば、`. = 0xFFFF000000080000;`は、カーネルに伝える最初の命令は0xFFFF000000080000であることを示します。ただし、実際には物理アドレス0x80000から
  実行を開始します（なぜそのように動くのでしょうか）。ページテーブルを開いた後、
  ほとんどのアドレスは高位アドレス（仮想アドレス）にジャンプするからです。
- テキストセグメントの最初の行は`KEEP(*(.text.boot))`です。これは、kern/entry.Sで
  作成した`.text.boot`セグメントをテキストセグメントの先頭に置くことを意味します。
  `KEEP`は、リンカにこのセグメントを常に保持するように指示します。
- `PROVIDE(etext = .)`は、シンボル`etext`を宣言し（ただし、特定のメモリ空間を
  割り当てるわけではないので、Cコードで値を割り当てることはできません）、その
  アドレスを現在のロケーションカウンタに設定します。これによりアセンブリまたは
  Cコードでカーネルセグメント（text/data/bss）のアドレス範囲を取得することが
  できます。これらのシンボルは、BSSを初期化する際に必要です。

### 1.3.2 ELF形式

`make`を実行すると、obj/kernel8.hdrが作成され、ELFヘッダを見ることができます。
SYMBOL TABLEを表示する際、Linuxでは`sort <obj/kernel8.hdr`が利用でき、アドレスを
ソートした結果を見ることができます。

仮想アドレス（VMA）とロードアドレス（LMA）の区別に注意してください。

- VMAは実行時のアドレスで、コード内のアドレスのコンパイルに影響します。リンカ
  スクリプトの中での`.`はVMAに影響を与えるために使用されます。
- LMAとはロードアドレスのことで、これはELFヘッダにのみ記載されており、ローダに
  このelfファイルをメモリ上のどこにコピーする必要があるかを簡単に伝えます。
  Raspberry Piでは、ELFヘッダを削除したobj/kernel8.imgを読み込むのでLMAを気に
  する必要はありません。

## 1.4 コードコメント

### 1.4.1 例外レベルの調整

ここではEL0とEL1（x86のring3とring0に似ている）のみを使用し、各々ユーザと
カーネルの状態を表します。 Raspberry Pi 3のハードウェアはEL2で起動しますが、
qemuではおそらくEL3で起動することになります。そこで、kern/entry.Sでは、まず
現在のELを判断し、EL1にステップダウンする必要があります。

### 1.4.2 初期ページテーブル

OSのカーネルコードは通常、上位アドレスに配置されています（なぜそのように設計
されているのか？)ので、アドレスの上位16ビットに基づいてページテーブルを自動的に
切り替えるというArm v8のMMUの仕組みを考慮して、カーネルを仮想アドレス
[0xFFFF'0000'0000, 0xFFFF'FFFF'FFFF]に置き、残りの仮想アドレス[0x0 ~ 0x0000'. FFFF'FFFF'FFFF]はユーザコードに割り当てます。

実装を容易にするため、kern/kpgdir.cに直接ページテーブルをハードコーディングし、
2つの1GBブロックのメモリ[0, 1GB)と[KERNBASE, KERNBASE+1GB)を次のようにどちらも
[0, 1GB)の物理アドレスにマッピングしました。KERNBASEは0xFFFF'0000' 0000'0000です。

| 仮想アドレス                                   | 物理アドレス       |
| ---------------------------------------------- | ------------------ |
| [0x0, 0x4000'0000)                             | [0x0, 0x4000'0000) |
| [0xFFFF'0000'0000'0000, 0xFFFF'0000'4000'0000) | [0x0, 0x4000'0000) |

この表の最初のマッピング（恒等マッピング）はなぜ必要なのか。 第1の理由は、
MMUを起動してもPCはローアドレスのままなので、このマッピングがないとMMU起動後の
最初の命令がエラーになるからです。第2の理由は、物理メモリへの直接アクセスを
容易にするためです。

さらに実際には、[KERNBASE+1GB, KERNBASE+2GB)を[1GB, 2GB)にマッピングしていますが、
これは後でクロック割り込みを設定する際に使用します。 ページテーブルの設定方法の
詳細は、この後のラボで説明します。

### 1.4.3 Cコードへのアクセス

ページテーブルを作成したら、スタックポインタを`_start`、すなわち、
[0xFFFF'0000'0000, 0xFFFF'0000'0008'0000)にポイントして、初期のカーネル
スタックを確保した後、高位アドレス(kern/main.c:main)にあるCコードに
ジャンプすることができます。main関数は戻りません。

### 1.4.3 練習問題1

初期化していないグローバル変数とローカルスタティック変数は、通常、コンパイラに
よってBSSセグメントに配置されますが、コンパイラは実行ファイルのサイズを小さく
するためにこのセグメントをELFファイルに組み込みません。そのため、これを手動で
ゼロに初期化する必要があります。これを行うよう、kern/main.cのコードを完成させて
ください。kern/linker.ldに基づいて、BSSセグメントがどこから始まってどこで終わる
のかを知る必要があるかもしれません。

### 1.4.4 練習問題2

コンソールの初期化をし、"hello, world\n"を出力するようkern/main.cのコードを完成
させてください。inc/console.h、kern/console.cを読む必要があるかもしれません。