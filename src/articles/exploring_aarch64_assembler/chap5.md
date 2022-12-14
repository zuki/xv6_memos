# [AArch64アセンブラ研究: 第5章](https://thinkingeek.com/2016/11/13/exploring-aarch64-assembler-chapter-5/)

この章ではAArch64でメモリにアクセスする方法について説明します。

## メモリ

ランダムアクセスメモリ、または単にメモリは、あらゆるアーキテクチャに不可欠な
コンポーネントです。メモリは各バイトがアドレスと呼ばれる連続した数字と対に
なった配列であると見なすことができることを思い出してください。AArch64では
アドレスは64ビットの数値です（ただし、すべてのビットがアドレスとして意味を
持つというわけではありません）。

### アドレスの代数

アドレスは数字ですので数字として操作することができます。ただし、すべての
算術演算がアドレスに対して意味を持つわけではありません。上位アドレスから
下位アドレスを引き算することができますが、その結果はアドレスではなく
オフセットです。オフセットはアドレスに加算して別のアドレスを作成できます。
メモリ上の連続するサイズbの要素にアクセスする必要がある場合が多いので
それらのアドレスも連続しています。これはi番目の要素のアドレスを計算するために
A + b*iの形のアドレスを計算することが一般的な操作であることを意味します。

このような一般的なアドレス計算があることは、後述するようにアーキテクチャの命令に
影響を与えています。

## ロードとストア

RISCの伝統を受け継いでいるため、AArch64の命令はメモリを操作しません。
2つの特別な命令であるロードとストアだけがこれができます。この2つの命令は
最も基本的な形としてレジスタとアドレスという2つのオペランドを取ります。
アドレスは後述するアドレッシングモードに基づいて計算されます。ロードは
計算されたアドレスから何バイトかを取り出してレジスタに格納します。ストアは
レジスタから何バイトかを取り出し、アドレスにより識別されるメモリ位置に
置きます。

AArch64はいくつかのロード/ストア命令をサポートしていますが、この章では
`ldr`（ロード）と`str`（ストア）だけを考えます。これらの構文は次のとおりです。

```
ldr Xn, address-mode    //  Xn ← address-modeにより計算されたアドレスにある8バイトのメモリ
ldr Wn, address-mode    //  Wn ← address-modeにより計算されたアドレスにある4バイトのメモリ

str Xn, address-mode    // address-modeにより計算されたアドレスにある8バイトのメモリ ← Xn
str Wn, Address-mode    // address-modeにより計算されたアドレスにある4バイトのメモリ ← Wn
```

AArch64はビッグエンディアンとリトルエンディアンに構成することができます。
これは一般的にはほとんど違いはありませんが、ロード/ストアする8/4バイトがどこの
レジスタに行き、どこから来るかを決めます。ここではリトルエンディアンとします
（最も一般的な構成です）。これは8バイトのロード/ストアを行う場合、最初のバイトが
レジスタの最下位のバイトに対応し、次のバイトが次の最下位のバイトに対応し、と
いった具合になることを意味します。ビッグエンディアンマシンの場合は逆で、最初の
バイトが最上位のバイトに対応します。

## アドレッシングモード

アドレッシングモードとはロード/ストア命令がアクセスするアドレスを計算する
プロセスのことです。AArch64の命令は32ビットでエンコードされますがすでに説明
したようにアドレスは64ビットです。これは即値を使った最も基本的なアドレッシング
モードが利用できないことを意味します。一部のアーキテクチャでは完全なアドレスを
命令でエンコードすることができます。そのようなアーキテクチャでも容量が大きく
なる可能性があるのでプログラムでそれを行うことはほとんどありません。

### ベース

次に考えられる最も単純なアドレッシングモードはまだ説明していない何らかの
メカニズムによりXnレジスタにアドレスがある場合です。この場合はアドレスの
計算にはレジスタの値を使うだけです。このモードはベースレジスタと呼ばれており
その構文は`[Xn]`です。ベースとして使用できるのは64ビットレジスタのみです。

```
ldr W2, [X1]        // W2 ← *X1 (32ビットロード)
ldr X2, [X1]        // X2 ← *X1 (64ビットロード)
```

### ベース + 即値オフセット

アドレスの代数について説明したときに説明したようにアドレスにオフセットを
加えて別のアドレスを作成することができます。構文は`[Xn, #offset]`です。
オフセットの範囲は32ビットと64ビットのアクセスで共に-256から255です。
これより大きなオフセットには制限があり、32ビットでは0から16380の範囲の
4の倍数、64ビットでは0から32760の範囲の8の倍数でなければなりません。

```
ldr W2, [X1, #4]        // W2 ← *(X1 + 4)     [32ビットロード]
ldr W2, [X1, #-4]       // W2 ← *(X1 - 4)     [32ビットロード]
ldr X2, [X1, #240]      // X2 ← *(X1 + 240)   [64ビットロード]
ldr X2, [X1, #400]      // X2 ← *(X1 + 400)   [64ビットロード]
// ldr X2, [X1, #484]   // 無効なオフセット、8の倍数でない!
// ldr X2, [X1, #-400]  // 無効なオフセット、正値でなければならない!
// ldr X2, [x1, #32768] // 無効なオフセット、範囲外!
```

### ベース + レジスタオフセット

即値オフセットが使えるのは便利ですが、オフセットが即値としてエンコードできない
場合やプログラムの実行前に不明な場合があります。このような場合は代わりに
レジスタを使用することができます。

```
ldr W1, [X2, X3]        // W1 ← *(X2 + X3) [32ビットロード]
ldr X1, [X2, X3]        // X1 ← *(X2 + X3) [64ビットロード]
```

シフト演算し`lsl`を使ってオフセットレジスタの値をスケールさせることができます。
この方法では`lsl #n`はオフセットを2<sup>n</sup>倍します。

```
ldr W1, [X2, X3, lsl #3]    // W1 ← *(X2 + (X3 << 3)) [32ビットロード]
                            // これは次と同じです。
                            // W1 ← *(X2 + X3 * 8) [32ビットロード]
ldr X1, [X2, X3, lsl #3]    // X1 ← *(X2 + (X3 << 3)) [64ビットロード]
                            // これは次と同じです。
                            // X1 ← *(X2 + X3 * 8) [64ビットロード]
```

ベースレジスタとは対照的に、オフセットレジスタを32ビットレジスタにすることも
できますが、その場合、32ビット値を64ビット値に拡張する方法を指定することが
必要です。第3章で見た拡張演算子を使わなければなりません。ソースが32ビット値で
あるので`sxtw`か`uxtw`しか使用できません。

```
ldr W1, [X2, W3, sxtw]      // W1 ← *(X2 + ExtendSigned32To64(W3))   [32ビットロード]
ldr X1, [X2, W3, uxtw]      // X1 ← *(X2 + ExtendUnSigned32To64(W3)) [64ビットロード]
```

すでに知っているように拡張演算子はシフトと組み合わせることができます。

```
ldr W1, [X2, X3, sxtw #3]   // W1 ← *(X2 + ExtendSigned32To64(W3 << 3)) [32ビットロード]
```

## インデキシングモード

時に現在読み込んでいる要素だけに基づいてメモリの連続した位置を読み込むことが
あります。最悪の場合でもレジスタの値に算術演算を使ってアドレスを計算することで
ベースインデックスモードを使用することができます。もう少し良い方法として最初の
アドレスをベースとして働くようにレジスタに保存しておいて都度オフセットを計算
する方法があります。後者のアプローチをとる場合、ほとんどの場合でオフセットは
加算や加算＋シフト（2のべき乗による乗算を行う）などの比較的単純な演算で更新
されます。ほとんどの場合、アドレス計算はそのような形になるという事実を利用する
ことができます。このような場合はインデキシングモードを使用するとよいでしょう。

AArch64にはプリインデキシングとポストインデキシングという2種類のインデキシング
モードがあります。プリインデキシングモードでは、ベースレジスタにオフセットを加えて
アドレスを計算し、このアドレスをベースレジスタに書き戻します。ポストインデキシング
モードでは、通常通りベースレジスタを使ってアドレスを計算しますが、メモリアクセスの
終了時にベースレジスタを指定されたオフセットが加算されたアドレス値で更新します。

これら2つのモードはベースレジスタをオフセットで更新するという点でよく似ています。
プリインデキシングモードはアクセスの前に、ポストインデキシングモードはアクセスの
後にオフセットが足される点が異なります。使用できるオフセットは-256～255の範囲で
なければなりません。

### プリインデキシング

プリインデキシングアクセスモードの構文は`[Xn, #offset]!`です。`!`を忘れないで
ください。これを忘れるとインデキシングなしのベース+オフセットを指定したことに
なります。実際にはこらはベース+オフセットと全く同じように動作しますがが、`!`
記号はベースレジスタを更新するという副作用を思い出させてくれます。

```
ldr X1, [X2, #4]!       // X1 ← *(X2 + 4)
                        // X2 ← X2 + 4
```

### ポストインデキシング

構文は`[Xn], #offset`です。プリインデキシングと同じ視覚的な合図が得られるように
#offsetの後に!がある構文であったらと思います。

```
ldr X1, [X2], #4        // X1 ← *X2
                        // X2 ← X2 + 4
```

## リテラルアドレスのロード

グローバル変数や関数などのグローバルオブジェクトは定数アドレスを持っています。
これはリテラルとして読み込むことができることを意味します。しかし、ご存知の通り
AArch64ではリテラルから直接ロードすることはできません。そのため2段階アプローチを
とる必要があります（これはRISCアーキテクチャでは非常に一般的のことです）。まず、
アセンブラにグローバル変数のアドレスを現在の命令の近くに置くように指示する必要が
あります。次に、ロード命令の特殊な形式（ロードイミディエートと呼ばれる）を使って
そのアドレスをレジスタにロードします。

ほとんどの例では次のようになります。

```
ldr Xn, addr_of_var                 // Xn ← &var
...
addr_of_var: .dword variable        // varのアドレスをここにしたいことを
                                    // アセンブラに知らせる（計算は不要）
```

一旦変数のアドレスをレジスタにロードしたら、移行はロード命令で必要なバイト数を
ロードすることができます。

```
ldr Xm, [Xn]                        // Xm ← *Xn    [64バイトロード]
ldr Wm, [Xn]                        // Wm ← *Xn    [32バイトロード]
```

### 32ビットアドレスを使う

64ビットアドレスを使うのは正しいのですが少し無駄があります。というのも
私たちのプログラムではコードとデータのすべてのアドレスをエンコードするのに
32ビット以上は必要ないことが多いからです。グローバル変数のアドレスの上位
32ビットは常に0です。そのため32ビットのアドレスを使用したいと思うでしょう。

```
ldr Wn, addr_of_var                 // Wn ← &var
...
addr_of_var: .dword variable        // varのアドレスをここにしたいことを
                                    // アセンブラに知らせる（計算は不要）
                                    // ここでは32ビットアドレス
```

32ビットの値をレジスタに書き込むと上位32ビットはクリアされることを思い出して
ください。したがって、このldr命令後には{Xn}を使うことでロードもストアも問題
なく行うことができます。

## グローバル変数

今日のトピックの例としていくつかのグローバル変数のロードとストアを行います。
このプログラムは何の役にも立ちません。

グローバル変数は`.data`セクションで定義します。そのためには単純にその初期値を
定義するだけです。32ビット変数を定義したい場合は`.word`ディレクティブを使用
します。64ビット変数を定義したい場合は`.dword`ディレクティブを使用します。

```
// globalvar.s
.data

.balign 8                           // 8バイトアライン
.byte   1
global_var64:   .dword 0x1234       // 64ビット値の0x1234
// alternativley:   .word 0x1234, 0x0

.balign 4                           // 4バイトアライン
.byte   1
global_var32:   .word 0x5678        // 32ビット値の0x5678
```

 LinuxのAArch64ではほとんどのメモリアクセスにアライメントを要求していません。
 しかし、アライメントされていればハードウェアでより速く実行することができます。
 そこで、各変数をデータの大きさ（バイト単位）に揃えるために`.balign`
 ディレクティブを使用しています。

これで変数をロードすることができます。この例ではそれぞれに1を足します。

```
.text

.global main
main:
    ldr X0, address_ofglobal_var64      // X0 ← &global_var64
    ldr X1, [X0]                        // X1 ← *X0
    add X1, X1, #1                      // X1 ← X1 + 1
    str X1, [X0]                        // *X0 ← X1

    ldr X0, address_of_global_var32     // X0 ← &global_var32
    ldr W1, [X0]                        // W1 ← *X0
    add W1, W1, #1                      // W1 ← W1 + 1
    str W1, [X0]                        // *X0 ← W1

    mov W0, #0                          // W0 ← 0
    ret                                 // プログラムを終了
address_of_global_var64: .dword global_var64
address_of_global_var32: .dword global_var32
```

### 32ビットアドレスを使う

上で述べたように64ビットのアドレスを変数に格納することは、通常、少し無駄が
あります。以下は32ビットのアドレスを使用するために必要な変更点です。

```
.text

.global main
main:
    ldr X0, address_ofglobal_var64      // X0 ← &global_var64
    ldr X1, [X0]                        // X1 ← *X0
    add X1, X1, #1                      // X1 ← X1 + 1
    str X1, [X0]                        // *X0 ← X1

    ldr X0, address_of_global_var32     // X0 ← &global_var32
    ldr W1, [X0]                        // W1 ← *X0
    add W1, W1, #1                      // W1 ← W1 + 1
    str W1, [X0]                        // *X0 ← W1

    mov W0, #0                          // W0 ← 0
    ret                                 // プログラムを終了
address_of_global_var64: .word global_var64     // ここで.wordを使っていることに注目
address_of_global_var32: .word global_var32     // ここで.wordを使っていることに注目
```

    注意: なお、これを使う場合は、最終的なリンクを行う際に`-static`フラグを
    使用する必要があります。これはメモリに直接ロードされる静的ファイルを作成
    します。デフォルトではリンカは動的ファイルを作成し、実行時に動的リンカに
    よって読み込まれます。動的リンカは2<sup>32</sup>を超えるアドレスに
    プログラムをロードするのでこれらのアドレスは無効なものとなります。
    `.dword`を使用すると静的リンカは動的リンカ用の注釈を発行し動的リンカが
    実行時に64ビットアドレスを固定できるようにします。

グローバル変数のアドレスを取得する方法は他にこれよりも少し良い方法もありますが、
今のところこれで十分です。後の章でこの方法を再検討するかもしれません。

今日はここまでです。
