# [AArch64アセンブラ研究: 第2章](https://thinkingeek.com/2016/10/08/exploring-aarch64-assembler-chapter-2/)

このシリーズの第1回目では初めての非常に簡単なプログラムを作成しました。
この章では引き続きAArch64についてもう少し学びます。

## レジスタ

コンピュータは2進数のデータしか扱えないので、プログラムはマシンコードと呼ばれる
ものに符号化されます。しかし、マシンコードを書くのは非常に大変なので代わりに
アセンブリ言語が使われます。アセンブリ言語では命令（とそのオペランド）と
プログラムのデータを指定することができます。命令はコンピュータに何をすべきかを
指示します（したがって、命令には意味があります）。

CPUはプログラムを実行するコンピュータの一部です。AArch64アーキテクチャを実装した
CPUの命令はCPUにあるデータでしか動作しません。このデータがある場所をレジスタと
呼びます。レジスタにないデータを操作するためには、まずレジスタにデータをロード
する必要があります。通常、データはメモリからロードされますが、周辺機器からロード
することもできます。また、外界との通信手段としてレジスタからメモリや周辺機器に
データを取り出すこともできます。

AArch64には31本の汎用レジスタがあります。どのようなデータでも格納できるので
汎用レジスタと呼ばれています。一般には整数かアドレス（アドレスについては後の
章で説明します）しか保持できませんが、64ビットに符号化できるものなら何でも
レジスタに格納することができます。この31本のレジスタはx0, x1, ..., x30と呼ばれ
ます。なぜ、より自然な2のべき乗値の32でなく、31なのかと思われるかもしれません。
その理由は、x31であるべきものがxzrと呼ばれるゼロレジスタを意味するからです。
これは非常に特殊なレジスタで使用用途も限られています。後ほど、このレジスタの
使い方の例については後で見てみます。一般にすべてのレジスタは任意の目的に使用
できますが、その使用についてはいくつか慣習があることを後の章で確認します。

AArch64アーキテクチャではさらに多くのレジスタを定義していますが、それらは
より具体的な目的を持っており、後の章で明らかにします。

64ビット幅のレジスタがあれば作業には十分ですが、これはすべての演算が64ビット
領域で行われることを意味します。多くの場合はそれほど多くのビットは必要なく、
実際、ほとんどのプログラムは32ビットデータ（あるいはそれ以下）で十分です。
32ビット処理を実現するために、xnレジスタの下位32ビットをwnという名前でアクセス
することが可能になっています。たとえば、レジスタx6の下位32ビットはw6です。
上位32ビットに名前を付けることはできません。レジスタxzrにはwzrという32ビットの
対応する名前があります。

これが、第1章のプログラムが次のようになっていた理由です。

```
mov w0, #2      // w0 ← 2
```

C言語ではmainの戻り値はint値です。技術的に言うと、Cはint値の具体的なビット幅を
規定していませんが（表現できなければならない値の最小限の範囲を述べているだけ）、
経済的な理由から（intがCで最も使用されている型であることから）、AArch64やx86-64
などのほとんどすべての64ビット環境ではintを32ビット整数型としています。

## レジスタのデータを操作する

AArch64のほとんどすべての命令は3つのオペランドを持ちます。デスティネーション
レジスタと2つのソースレジスタです。たとえば、レジスタw3とw4を足した結果を
レジスタw5に格納することができます。

```
add     w5, w3, w4      // w5 ← w3 + w4
```

同様に、

```
add     x5, x3, x4      / x5 ← x3 + x4
```

ただし、一般にwnレジスタとxnレジスタを同時に一つの演算に指定することはできない
ことに注意してください。次の命令

```
add     w5, w3, x4
```

は、有効な代替手段を示唆するメッセージが表示され、失敗します。

```
add.s:6: Error: operand mismatch -- `add w5,w3,x4'
add.s:6: Info:    did you mean this?
add.s:6: Info:    	add w5,w3,w4
add.s:6: Info:    other valid variant(s):
add.s:6: Info:    	add x5,x3,x4
```

### ゼロレジスタ

ゼロレジスタxzr（または、wzr）は、ソースレジスタとしてのみ有効です。これは
実レジスタを表すものではなく「オペランドの値としてここに0を仮定する」という
ことを示すだけのものです。

### MOVE

上で述べた、1つのデスティネーションレジスタと2つのソースレジスタという
スキーマにはいくつかの例外があります。特に目立つのは`mov`命令です。この命令は
ソースレジスタを1つしか取りません。

```
mov     w0, w1          // w0 ← w1
```

なお、これは利便的な命令であり、他の命令で実装することも可能です。その一つの
方法としてソースレジスタに0を加算することが考えられます。上のmovは次のようにも
書くことができるでしょう。

```
add     w0, w1, wzr     // w0 ← w1 + 0
```

実際、AArch64では`mov`は`orr`という命令で実装されています。これは第1ソース
オペランドとしてwzrを使用してビット単位のor演算を行う命令です。

### 即値

命令のソースオペランドがレジスタに限定されていたとしたら、レジスタに初期値を
ロードすることができません。即値と呼ばれるオペランドを許可している命令があるのは
そのためです。即値とは命令自体に符号化されている整数のことです。これはすべての
値を即値としてエンコードできるわけでは無いことを意味しますが幸いなことに多くの
値が即値でエンコードできます。即値の許容範囲は命令によって異なりますが、多くの
命令では`[-4096, 4095]`（すなわち、12bit）の範囲です。また、使用する符号化の
関係でこの範囲に2<sup>12</sup>（4096）を乗じた値も即値として使用できます。
たとえば、12288 と 16384 は即値として使用できます（ただし、この間の数値は使用
できません）。即値はアセンブリ構文では戦闘時として`#`を付けて表します。

```
mov     w0, #2          // w0 ← 2
mov     w1, #-2         // w1 ← -2
```

次は許されません。

```
add     w0, #1, w1      // ERROR: second operand should be an integer register
add     w0, #1, #2      // ERROR: second operand should be an integer register.
                        // This case is actually better expressed as
                        //    mov w0, #3
```

### ディスティネーションとしての32ビットレジスタ

命令のディスティネーションレジスタが32ビットレジスタである場合、上位32ビットには
0がセットされます。上位32ビットは保存されません。


## つまらない例

この時点ではまだ大したことはできませんが、次のプログラムで少し遊ぶことができます。
このプログラムの引数の数はプログラム開始時にw0に格納されます。この数字に1を足した
ものを返すことにしましょう。

```
.text
.global main

main:
    add     w0, w0, #1       // w0 ← w0 + 1
    ret

$ aarch64-linux-gnu-gcc -c test.s
$ aarch64-linux-gnu-gcc -o test test.o
$ ./test ; echo $?
2
$ ./test foo ; echo $?
3
$ ./test foo bar ; echo $?
4
```

やったー。なぜ最初のケースが1ではなく2を返すのかと思うかもしれませんが、UNIXでは
Cプログラムのmain関数は常に実行されるプログラムの名前を最初のパラメータとして
受け取るからです。

今日はここまでです。
