# [AArch64アセンブラ研究: 第4章](https://thinkingeek.com/2016/10/23/exploring-aarch64-assembler-chapter-4/)

この章では何らかの計算を可能にする命令を見ていきます。

## 算術演算命令

コンピュータはよくできた電卓に過ぎませんから（あるいは、電卓は小さなコンピュータに
過ぎませんから）基本的な計算ができなければなりません。ここでは整数演算に限定して
説明します。後の章で他の種類の数字をどのように操作できるかについて説明します。

### 加算と減算

加算と減算は`add`命令と`sub`命令で行うことができます。これらの命令はかなり柔軟で
いろいろな形式が使えます。

```
add Rdest, Rsource1, #immediate         // Rdest ← Rsource1 + immediate
add Rdest, Rsource1, Rsource2           // Rdest ← Rsource1 + Rsource2
add Xdest, Xsource1, Xsource2, shiftop  // Xdest ← Xsource1 + shiftop(Xsource2)
add Xdest, Xsource1, Xsource2, extop    // Xdest ← Xsource1 + extop(Xsource2)
add Wdest, Wsource1, Wsource2, shiftop  // Wdest ← Wsource1 + shiftop(Wsource2)
add Wdest, Wsource1, Wsource2, extop    // Wdest ← Wsource1 + extop(Wsource2)
add Xdest, Xsource1, Wsource2, extop    // Xdest ← Xsource1 + extop(Wsource2)
```

上記の形式において、`Rx`は`Xx`または`Wx`を意味し（ただし、同一命令内で混在
させない）、`shiftop`と`extop`は3章で説明したシフト演算子と拡張演算子を意味
します。ここでは`shiftop`に`ROR`は含まれません。`add`で示した形式はすべて
`sub`にも使用できます。

### 乗算と除算

加算と減算に比べて乗算と除算は難しい演算です。これを行う複数の命令が用意されて
います。

乗算の性質上、32/64ビットの2つの値を掛け算した数学的な結果を完全に符号化する
には64/128ビットが必要になる場合があります。このようなことが起こらないことが
わかっている場合（つまり、結果の値が32/64ビットで符号化できる場合）、または、
桁あふれを気にしない場合は`mul`命令を使用することができます。この場合、余分な
ビットがあれば削除されます。

```
mul Rdest, Rsource1, Rsource2       // Rdest ← Rsource1 * Rsource2
                                    // ただし、 オーバフローに注意
```

余分なビットを気にするのであれば、使える命令はたくさんあります。32ビットの乗算では
`umull`と`smull`を使うことができます。後者は2の補数の数値の乗算で符号ビットを
正しく処理するるために必要です。

```
umull Xdest, Wsource1, Wsource2     // Xdest ← Wsource1 * Wsource2
smull Xdest, Wsource1, Wsource2     // Xdest ← Wsource1 * Wsource2
                                    // Wsource1とWsource2の値が2の歩数の場合
```

あまり一般的ではありませんが2つの64ビットレジスタを乗算し128ビットの結果の
上位64ビットを気にする場合は`umulh`と`smulh`を使用します。この場合、2つの
64ビットレジスタ（下の例ではXlowerとXupperと名付けました）が必要です。

```
mul   Xlower, Xousrce1, Xsource2    // Xlower ← Lower64Bits(Xsource1 * Xsource2)
smulh Xupper, Xousrce1, Xousrce2    // Xupper ← Upper64Bits(Xsource1 * Xsource2)
```

除算は`udiv`と`sdiv`の2つの命令だけで済むのでもう少し簡単です。ここでも後者は
2の補数で符号化された整数用です。

```
udiv Rdest, Rsource1, Rsource2      // Rdest ← Rsource1 / Rsource2
sdiv Rdest, Rsource1, Rsource2      // Rdest ← Rsource1 / Rsource2
                                    // Rsource1とRsource2の値が2の歩数の場合
```

これら2つの命令は0の方向に丸めた除算の商を計算します。

## ビット演算命令

ビット演算命令はレジスタの各ビットを直接操作するものであり、レジスタの
エンコーディングは考慮されません。

`mvn`命令はオペランドに対するビット単位否定演算を行います。

```
mvn Rdest, Rsource      // Rdest ← ~Rsource
```

ビット演算命令の多くはソースレジスタを2つ使用します。基本的な命令は`and`、
`orr`、`eor`（排他的論理和）です。各々ビット単位の論理積、論理和、排他的論理和を
実行します。

```
and Rdest, Rsource1, #immediate         // Rdest ← Rsource1 & immediate
and Rdest, Rsource1, Rsource2           // Rdest ← Rsource1 & Rsource2
and Xdest, Xsource1, Xsource2, shiftop  // Xdest ← Xsource1 & shiftop(Xsource2)
and Wdest, Wsource1, Wsource2, shiftop  // Wdest ← Wsource1 & shiftop(Wsource2)
```

`orr`と`eor`にもこれらと同じ形式があります。ビット演算の`shitop`には`ROR`も
含まれます。

`mvn`と3つの論理演算命令`and`, `orr`, `eor`をそれぞれ組み合わせた命令である
`bic` (bit clear), `orn` (or not), `eon` (exclusive or not) があります。この場合、
第2オペランドにはまずNOT演算が適用されます。これらは少しだけ限定された形に
なっています。

```
orn Rdest, Rsource1, Rsource2           // Rdest ← Rsource1 | ~Rsource2
orn Xdest, Xsource1, Xsource2, shiftop  // Xdest ← Xsource1 | ~shiftop(Xsource2)
orn Wdest, Wsource1, Wsource2, shiftop  // Wdest ← Wsource1 | ~shiftop(Wsource2)
```
`bic`と`eon`にも同じ形式があります。

もっとユースケースが限られた命令がありますが今回は省略します。

今回はここまでにします。
