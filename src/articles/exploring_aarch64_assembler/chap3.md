# [AArch64アセンブラ研究: 第3章](https://thinkingeek.com/2016/10/23/exploring-aarch64-assembler-chapter-3/)

前章では、命令はレジスタオペランドと即値オペランドを持つことを確認しました。
また、32ビットと64ビットのレジスタは混在できないも説明しました。今回は
レジスタオペランドについてもう少し詳しく説明します。

## レジスタオペランドに対応する演算子

レジスタを第2ソースオペランドにとる命令の多くは、そのソースレジスタの値に
何らかの演算を施すこともできます。これはより少ない命令で計算密度を高める
方法として、また、オペランドの一方に変換などの一般的な演算を施す方法として
利用することができます。

2種類の演算子であるシフト演算子と拡張演算子を区別することができます。

### シフト演算子

AArch64には4つのシフト演算子LSL、LSR、ASR、RORがあります。その構文は以下の
通りです。

```
reg, LSL, #amount
reg, LSR, #amount
reg, ASR, #amount
reg, ROR, #amount
```

ここで`reg`は64ビットレジスタ`Xn`または32ビットレジスタ`Wn`を、`amount`は
使用するレジスタに依存する数値で32ビットレジスタでは0〜31、64ビットレジスタでは
0〜63の範囲です。

演算子`LSL`はregの値を左に論理シフトします（ただし、regの内容は変更しません）。
nビットを左にシフトするということは元の値から最下位ビットに0をn個挿入し、
最上位ビットをn個破棄することを意味します。nビットの左シフトは2<sup>n</sup>の
乗算に相当します。

```
add     r1, r2, r3, LSL #4      // r1 ← r2 + (r3 << 4)
                                // これは次に等しい
                                // r1 ← r2 + r3 * 16
add     r0, r0, r0, LSL #2      // r0 ← r0 + (r0 << 2)
                                // これは次に等しい
                                // r0 ← r0 + r0 * 4
                                // これはオーバーフローが無いと仮定すれば次に等しい
                                // r0 ← r0 * 5
```

演算子`LSR`は論理右シフトを行います。この演算とLSLはニコイチの演算ですが
最上位nビットに0が挿入され、最下位nビットが破棄されます。符号なし算術としては
2<sup>n</sup>による除算に相当します。

演算子`ASR`は算術右シフトを行います。これはLSRと似ていますが最上位ビットn個に
0が挿入されるのではなく、最上位ビットがn回複製されて最上位ビットn個に挿入され
ます。LSRと同様に最下位nビットは破棄されます。レジスタの最上位ビットが0の場合、
ASRはLSRと等価です。このシフト演算子は符号ビット（2進数として解釈した場合の
レジスタの最上位ビット）を伝播するため、2の補数に対して有用であり、負数の
2<sup>n</sup>による除算にも使用できます。2の補数による負数に対するLSRは除算の
目的としては意味を持ちません。

演算子`ROR`はレジスタの右ローテートを行います。これは暗号によく使われるもので
他のシフトオペランドに比べると使用が少ないです。ローテートはLSRと似ていますが
ビットを落として0を挿入するのではなく、LSRでは落とすはずの最下位ビットを最上位
ビットとして挿入します。ローテートには左ローテートはありません。右ローテートで
代用できるからです。それには全ビット数から左に回転させたいビット数を引いた数
だけ右にローテートすればよいのです。

AArch64でRORシフト演算子を使用できるのは一部の命令（主に論理命令）だけです。

```
mov     w2, #0x1234             // w2 ← 0x1234
mov     w1, wzr                 // w1 ← 0
orr     w0, w1, w2, ROR #4      // w0 ← BitwiseOR(w1, RotateRight(w2, 4))
                                // w0 = 0x40000123
orr     w0, w1, w2, ROR #28     // w0 ← BitwiseOR(w1, RotateRight(w2, 32-4))
                                // これは実質上 rotateLeft(w2, 4)に等しい
                                // w0 = 0x12340
```

### 拡張演算子

拡張演算子の主な目的はレジスタにある狭い値を演算に必要なビット数に合わせて
広げることです。拡張演算子は<i>k</i>xt<i>w</i>という形です。ここで、<i>k</i>は
広げたい整数の種類、<i>w</i>は元の狭い値のビット幅です。前者については、整数の
種類は`U`（符号なし）または`S`（符号あり、すなわち2の補数）です。後者については、
ビット幅は`B`、`H`、`W`のいずれかであり、それぞれバイト（レジスタの最下位8ビット）、
ハーフワード（レジスタの最下位16ビット）、ワード（レジスタの最下位32ビット）を
意味します。

つまり、拡張演算子は`uxtb`, `sxtb`, `uxth`,` sxth`,` uxtw`, `sxtw`のいずれかです。

これらの演算子が存在するのはソース値の範囲を小さいビット幅から大きいビット幅に
上げなければならない場合があるからです。そのようなケースはこの後の章で数多く目に
することになるでしょう。たとえば、32ビットのレジスタを64ビットのレジスタに足す
必要がある場合があります。両方のレジスタが2の補数整数を表していれば次のように
なります。

```
add     x0, x1, w2, sxtw        // x0 ← x1 + ExtendSigned32To64(w2)
```

これらの拡張演算子を使用する際には何らかの文脈を考慮する必要があります。
たとえば、以下の2つの命令は若干意味が異なります。

```
add     x0, x1, w2, sxtb        // x0 ← x1 + ExtendSigned8To64(w2)
add     w0, w1, w2, sxtb        // w0 ← w1 + ExtendSigned8To32(w2)
```

どちらの場合もw2の最下位8ビットを拡張しますが、前者は64ビットに、後者は
32ビットに拡張しています。

#### 拡張とシフト

拡張演算子の後に量を指定することで、値を拡張し他あとに1、2、3、4ビット左に
シフトすることができます。たとえば

```
mov     x0, #0                  // x0 ← 0
mov     x1, #0x1234             // x1 ← 0x1234
add     x2, x0, x1 sxtw #1      // x2 ← x0 + (ExtendSigned16To64(x1) << 1)
                                // x2 = 0x2468
add     x2, x0, x1 sxtw #2      // x2 ← x0 + (ExtendSinged16To64(x1) << 2)
                                // x2 = 0x48d0
add     x2, x0. x1 sxtw #3      // x2 ← x0 + (ExtendSinged16To64(x1) << 3)
                                // x2 = 0x91a0
add     x2, x0. x1 sxtw #4      // x2 ← x0 + (ExtendSinged16To64(x1) << 4)
                                // x2 = 0x12340
```

現時点では少し奇妙で恣意的に見えるかもしれませんが、後の章ではこれが実際に
多くのケースで有用であることがわかるでしょう。

今日はここまでです。
