# [AArch64アセンブラ研究: 第1章](https://thinkingeek.com/2016/10/08/exploring-aarch64-assembler-chapter1/)

 AArch64はARM社が2011年に発表したARMv8アーキテクチャを構成するある新しい64ビット
 モードです。スマートフォンやサーバへの搭載が進んでいます。そこで、このアーキ
 テクチャのアセンブラについてもう少し勉強してみる良い機会だと思います。

## ハードウェアの入手

ARMv6/ARMv7アーキテクチャを搭載したシングルボードコンピュータは現在では容易に
入手することができます。その代表的なものはRaspberry Piです。

これに対して、ARMv8の64ビットモードに対応したシングルボードコンピュータはあまり
一般的ではありませんが、最近徐々に普及してきています。たとえば、Pine64、ODROID-C2、
Dragonboard 410cなどです。どれでも構いませんが、一般的には使用するSystem on Chipに
よって異なります。

    注: Raspberry Pi 3はARMv8アーキテクチャの64ビットモードを実装したCPU
    （Cortex-A53）を搭載しており、技術的には64ビットシステムを実行することが
    できます。しかし、Raspberry Foundationが提供しているソフトウェアシステム
    （Raspbian）は32ビット用のみであり、64ビットシステムに対応する公式な計画は
    ありません。

    更新: SUSEは、Raspberry Pi 3で動作可能な64ビット版のOpenSUSEディストリ
    ビューションを提供しています。ArchにもRPi3にインストールできる64-bit
    ディストリビューションがあります。

## 代替ソフトウェア

ハードウェアがないとAArch64で遊べないということでしょうか？いいえ、そうでは
ありません。クロスツールチェーンとユーザモードのQEMUを使えば多くのことが可能です。

### Ubuntu 16.04の例

QEMUとAArch64用のクロスツールチェーンをインストールするだけです。

```sh
$ sudo apt-get install qemu-user gcc-aarch64-linux-gnu
```

ではC言語で書かれた"Hello world"を実行できるかテストしましょう。以下の内容の
hello.cファイルを作成してください。

```c
#include <stdio.h>

int main(int argc, char *argv[])
{
    printf("Hello AArch64!\n");
    return 0;
}

次に、先にインストールしたAArch64用のクロスコンパイラでコンパイルします
（-staticフラグが重要です）。

```sh
$ aarch64-linux-gcc-gcc -static -o hello hello.c
```

AArch64バイナリであるかチェックします。

```sh
$ file hello
hello: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, for GNU/Linux 3.7.0, BuildID[sha1]=97c2bc66dbe4393aab9e4885df8e223a6baa235a, not stripped
```

実行すると何か紛らわしいエラーが出て失敗するはずです。

```sh
$ ./hello
-bash: ./hello: No such file or directory
```

しかし、先にインストールしたAArch64用のQEMUを使って実行することができます。

```sh
$ qemu-aarch64 ./hello
Hello AArch64!
```

    注: このオプションを使用する際は常に`qemu-aarch64`を使ってプログラムを実行
    することを忘れないでください。

## はじめてAArch64アセンブラプログラム

エラーコード2を返すだけの非常に簡単なプログラムを書いてみましょう。

```
// first.s
.text

.global

main:
    mov     w0, #2
    ret
```

アセンブルします。

```sh
$ aarch64-linux-gcc-gnu-as -c first.s
```

リンクします。便宜上、gccを使います。

```sh
$ aarch64-linux-gcc-gnu-gcc -static -o first first.o
```

実行してリターン値をチェックします。

```sh
$ ./first               # あるいは qemu-aarch64 ./first
$ echo $?
2
```

やった！

コードを1行ずつ見ていきましょう。

```
1 // first.s
2 .text
```

 1行目はこの例で使用するファイル名を記載した単なるコメントです。行の中で`//`に
 続くテキストはコメントであり無視されます。2行目はアセンブラディレクティブであり
 「これからプログラムの命令が来るぞ」という意味です。これはアセンブラファイルの
 中でデータも表現できるからです（データは`.data`ディレクティブの後に来ます）。

```
4 .globl main
```

これもアセンブラディレクティブであり`main`はグローバルシンボルであることを意味
します。これは、最終的なプログラムを構築する際にこのファイルにはC言語ライブラリが
プログラムを開始する際に必要とするグローバルな`main`シンボルがあることを意味します。

```
6 main:
7     mov w0, #2     // w0 ← 2
8     ret            // リターン
```

これはプログラムのエントリポイントです。6行目自体は`main`というシンボルのラベルに
過ぎません（先ほどグローバルシンボルと言ったものです）。7行目と8行目は2つの命令です。
最初の命令はレジスタw0に2をセットしているだけです（レジスタとは何かについては
次の章で説明します）。2番目はmainから復帰し、事実上プログラムを終了しています。

プログラムを終了する際、レジスタw0の内容はプログラムのエラーコードを決定する
ために使用されます。これが上で`echo $?`が`2`を表示する理由です。

## 参考文献

AArch64命令セットに関する文書は["ARM® Architecture Reference Manual ARMv8, for ARMv8-A architecture profile"](https://developer.arm.com/docs/ddi0487/a/arm-architecture-reference-manual-armv8-for-armv8-a-architecture-profile)にあります。
