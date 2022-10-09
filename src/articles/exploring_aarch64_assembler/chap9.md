# [AArch64アセンブラ研究: 第9章](https://thinkingeek.com/2017/11/05/exploring-aarch64-assembler-chapter-9/)

第6章では条件分岐を説明し、より高度な制御構造を実装するために使用できる
ことをコメントし終わりました。この章ではそのうちのいくつかを見ていきます。

## ifステートメント

この時点でif文をどのように実装するかは明らかでしょう。次の形の構造が
あるとします。

```
IF (E) THEN
    S1
ELSE
    S2
END IF
```

アセンブリでは次のようなものになるでしょう。

```
/* 条件Eを評価してフラグを更新する */
b<inverse-condition> else_label
    S1
    b end_if
else_label:
    S2
end_fi:
```

次のような別の方法でも実装できます。

```
/* 条件Eを評価してフラグを更新する */
b<condition> then_label
    S2
    b end_if
then_label:
    S1
end_if:
```

## 条件をどのように評価するか？

多くのプログラミング言語では条件は明示的にboolean型（C++のbool、Fortranの
LOGICALなど）で与えれるか、限定された値集合の整数型（Cの0か1など）で
与えられるかのいずれかです。通常の整数から条件式への変換が暗黙のうちに
定義されているプログラム言語もあります。たとえば、C言語では整数型または
ポインタ型の式`e`を条件式で使用した場合は条件式`e != 0`（0または1の値を
与えます）を評価したことに相当します。C++でも同じ変換が定義されていますが、
結果の型は`bool`であり`true`か`false`になります。Fortranはこれらの変換を
持たないのでオペランドが常に必要です。

`e1 ⊕ e2`の形の式（ここで`⊕`は `<`, `>`, `≤`, `≥`などの関係演算子）では
`cmp`命令に続いて、関連する条件を指定した`b.cond`を実行することで実現する
ことができます。

条件式の評価は言語仕様の影響を受けることがあります。条件付きで評価される
論理演算子（短絡的評価ともいいます）を実装しているプログラミング言語も
あります。これらの演算子は非厳格化され、論理演算子として数学的定義よりも
if文に近くなっています。論理演算子の定義を厳密に行っているプログラミング
言語もあります。これはすべてのオペランドを評価してから論理演算を適用する
ものです。CとC++は短絡的評価を採用しており、`e1 && e2`のような式は`e1`が
真でなければ`e2`を評価することはありません。同様に、`e1 || e2`のような
式では`e1`が偽でなければ`e2`は評価されません（これらの演算子の定義に
"if"が使われているので「条件付き評価」になっています）。これは次のような
C/C++のコード

```c
if (e1 && e2)
    S1;
else
    S2;
```

は実際は次のように実装されていることを意味します。

```c
if (e1)
{
    if (e2)
        S1
    else
        S2
}
else
    S2
```

もちろんこれをそのまま行うのは馬鹿げているので、実際には次のような形に
なります（C/C++ではこれもあまり良いものではありませんが）。

```c
if (e1)
{
    if (e2)
        S1
    else
        goto else_block;
}
else
{
    else_block:
        S2;
}
```

これをさらに単純化してアセンブリに近づけることができます（以下、
わかりやすくするために冗長なC/C++のラベルをいくつか追加しています）。

```c
if_start:
    check_e1: if (e1) goto check_e2;
    goto else_block;
    check_e2: if (e2) goto then_clock;
    goto else_block;
then_block:
    S1;
    goto if_end;
else_block:
    S2;
if_end:
```

C/C++の論理演算はいつもこのように実装する必要があるのでしょうか？そうでは
なく、第2オペランドを無条件に評価することと、条件付きで評価することが
異なる可能性があることを確信できない場合だけです。たとえば、次のような
ケースです。

```c
int a;
...
if ((5 < a) && a < 25)
{
}
```

両オペランドを評価しても害はないので、分岐を避け、実際に次のコードで
あったかのように評価することができます（ビット単位操作を意味する1つの
&は常に2つのオペランドを評価することに注意してください）。

```c
if ((s < a) & (a < 25))
{
}
```

しかし、次のようなケース

```c
if (p != 0 && ((10 / p) > 5))
{
}
```

では、`p == 0`の時に`((10 / p) > 5)`を評価したくはありません。
同じ戦略は`||`にも使われ、時には`|`として評価できる場合もあります。

次の表は、いくつかの例を示しています。これらは提案ですので状況によっては
もっと巧妙なシーケンスを使用することができるでしょう。

<table>

<thead>
<tr>
<th>評価する条件 (C/C++)</th>
<th>命令シーケンス</th>
</tr>
</thead>

<tbody>
<tr>
<td><code>e1&nbsp;&lt;&nbsp;e2</code></td>
<td><p>直訳</p>

<figure class="highlight"><pre><code class="language-asm" data-lang="asm">cmp e1, e2
b.lt true_case
   </code></pre></figure>

   <p><code>!(e1 &lt; e2)</code>を評価したい場合
   <small>（これは<code>false_case</code>が<code>true_case</code>の場合、<code>e1 &gt;= e2</code>に等しい）</small>
   </p>

<figure class="highlight"><pre><code class="language-asm" data-lang="asm">cmp e1, e2
b.ge false_case
   </code></pre></figure>

   <p>オペランドを逆にできる<code>e2 &gt; e1</code>場合</p>

<figure class="highlight"><pre><code class="language-asm" data-lang="asm">cmp e2, e1
b.gt true_case</code></pre></figure>

<p>オペランドを逆にできる<code>!(e2 &gt; e1)場合</code>.
<small>（これは<code>false_case</code>が<code>true_case</code>であれば<code>e1 &lt;= e2</code>に等しい）</small>
</p>

<figure class="highlight"><pre><code class="language-asm" data-lang="asm">cmp e2, e1
b.le false_case</code></pre></figure>

</td>
</tr>

<tr>
<td><code>e1&nbsp;&amp;&amp;&nbsp;e2</code></td>
<td>
<p>直訳</p>

<figure class="highlight"><pre><code class="language-asm" data-lang="asm">cmp e1, #1
b.ne false_case
check_e2:
  cmp e2, #1
  b.ne false_case
true_case:
  ...
false_case:
</code></pre></figure>

<p>短絡評価なし（すわなち、<code>e1 &amp; e2</code>）</p>

<figure class="highlight"><pre><code class="language-asm" data-lang="asm">tst e1, e2       // bitwise-and(e1, e2)の結果で
                 // フラグを更新
b.eq false_case
true_case:
   ...</code></pre></figure>

<p><code>tst</code>命令は2つのレジスタオペランドのビット単位ANDを計算し、その結果に基づいてフラグを更新する。結果自体は破棄される。
</p>
</td>
</tr>

<tr>
<td><code>e1&nbsp;||&nbsp;e2</code></td>
<td>

<figure class="highlight"><pre><code class="language-asm" data-lang="asm">cmp e1, #0
b.eq false_case
check_e2:
  cmp e2, #0
  b.eq false_case
true_case:
  ...</code></pre></figure>


<p>短絡評価なし（すなわち、<code>e1 | e2</code>）</p>

<figure class="highlight"><pre><code class="language-asm" data-lang="asm">orr x1, e1, e2 // x1 = bitwise-or(e1, e2)
cmp x1, #0
b.eq false_case:
true_case:
  ...</code></pre></figure>

</td>
</tr>


<tr>
<td>
<code>!e</code>
</td>
<td>
<p><code>e</code>はすでにブーリアンの時はほとんど稀なケースだが、0と比較するだけである。</p>

<figure class="highlight"><pre><code class="language-asm" data-lang="asm">cmp e1, #0
b.ne false_case
true_case:
  ...</code></pre></figure>

<p>
しかし、通常、<code>!</code>は逆の条件を比較するだけで実装できる。たとえば、<code>!(e1 &lt; e2)</code>は<code>e1 &gt;= e2</code>として評価できる。
</p>
<p>
!(e1 &amp;&amp; e2) のような場合は、 <code>!e1 || !e2</code>のようにド・モルガンの法則を使用できる。これは<code>!((e11 &lt; e12) &amp;&amp; (e21 &lt; e22))</code>のような場合、特に有用である。ここで ! は式の内部まで伝播し<code>(e11 &gt;= e12) || (e21 &gt;= e22)</code>となる。
</p>
</td>
</tr>

</tbody>
</table>

## 条件付き代入

C/C++でよく使われるイディオムとして次のようなものがあります。

```c
if (a)
    x = e;
```

Fortranではこの場合の省略記法があります。

```fortran
IF (A) X = E;
```

より一般的にC/C++では

```c
if (a)
    x = e1;
else
    x = e1;
```

これはC/C++ではさらにコンパクトに書くことができます。

```c
x = a ? e1 : e2;
```

多くの場合、e1とe2について、一般的なケースを次のように書き直すことは
正しい。

```c
x = e2;
if (a)
    x = e1;
```

あるいは、逆の形で

```c
x = e1;
if (!a)
    x = e2;
```

これらのどちらかの形でコードを書くことができ、そしてこのようなケースは
比較的一般的なので、AArch64はそのための特定のサポートを提供しています。

`csel rdest, rsource1, rsource2, cond`という命令は、条件condが真であれば
デスティネーションレジスタrdestの値をrsource1に、そうでなければrsource2に
設定します。cselは"conditional selection"の略です。

たとえば、Cによる次のような典型的なmax演算

```c
int m, a, b;
...
m = a > b ? a : b;
```

は、cselを使って次のように実装できます。

```
cmp x0, x1              // x0とx1を比較
csel x2, x0, x1, gt     // x2 ← if x0 > x1
                        //      then       x0
                        //      otherwise  x1
```

分岐がないので便利です。
