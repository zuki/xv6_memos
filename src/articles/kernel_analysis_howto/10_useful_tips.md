# 10.  役に立つヒント

## 10.1 スタックとヒープ

### 概要

ここでは「スタック」と「ヒープ」がメモリ上でどのように割り当てられるかを見ていきます。

### メモリ割り当て

```
FF..        |                 | <-- スタックの底
       /|\  |                 |   |
 higher |   |                 |   |   スタックの成長方法
 values |   |                 |  \|/
            |                 |
XX..        |                 | <-- スタックのトップ [スタックポインタ]
            |                 |
            |                 |
            |                 |
00..        |_________________| <-- スタックの終わり[スタックセグメンチ]

                   スタック
```

メモリアドレスは`00..`値から開始（ここはスタックセグメントの開始地点でもある）し、
`FF..`値の方向に成長します。

`XX..`はスタックポインタの実際の値です。

スタックは関数において次の用途で使用されれます。

1. グローバル変数
2. ローカル変数
3. 返り値

たとえば、古典的な次の関数において

```c
int foo_function (parameter_1, parameter_2, ..., parameter_n) {
    variable_1 declaration;
    variable_2 declaration;
      ..
    variable_n declaration;

    // 関数本体
    dynamic variable_1 declaration;
    dynamic variable_2 declaration;
      ..
    dynamic variable_n declaration;

    // コードはデータ/スタックセグメントではなくコードセグメントに置かれます

    return (ret-type) value;  // レジスタに置かれることもあります。
                              // たとえば、i386ではeaxレジスタが使用されます。
}
```

メモリは次のようになります。

```
          |                       |
          | 1. parameter_1 pushed | \
    S     | 2. parameter_2 pushed |  | Before
    T     | ...................   |  | the calling
    A     | n. parameter_n pushed | /
    C     | ** Return address **  | -- Calling
    K     | 1. local variable_1   | \
          | 2. local variable_2   |  | After
          | .................     |  | the calling
          | n. local variable_n   | /
          |                       |
         ...                     ...   Free
         ...                     ...   stack
          |                       |
    H     | n. dynamic variable_n | \
    E     | ...................   |  | Allocated by
    A     | 2. dynamic variable_2 |  | malloc & kmalloc
    P     | 1. dynamic variable_1 | /
          |_______________________|

            スタックの通常の使われ方
```

注意: 変数の格納順はハードウェアアーキテクチャにより異なる場合があります。

## 10.2 アプリケーションとプロセス

### 基本的な定義

次の2つの概念を区別する必要があります。

- アプリケーション: 実行したい有益なコードです。
- プロセス: アプリケーションのメモリ上の「イメージ」です（使用するメモリ戦略や
  セグメンテーション、ページネーションによります）。

プロセスはタスクやスレッドと呼ばれることもあります。

## 10.3 ロック

### 概要

次の2種類のロックがあります。

1. CPU内 (intraCPU)
2. CPU間 (interCPU)

## 10.4 コピーオンライト

コピーオンライトはメモリの使用を減らすための機構です。メモリが実際に使用されるまで
メモリの割当を先送りします。

たとえば、タスクが"fork()"システムコールを（他のタスクを作成するために）呼び出した際、
依然として親と同じメモリページをリードオンリーモードで使用します。タスクがそのページに
書き込みを行うと例外が発生してページがコピーされ"rw"(読み書きモード)としてマークされます。


1-) ページXはタスクParentとタスクChildで共有される。
```
 Task Parent
 |         | RO Access  ______
 |         |---------->|Page X|
 |_________|           |______|
                          /|\
                           |
 Task Child                |
 |         | RO Access     |
 |         |----------------
 |_________|
```

2-) Write要求
```
 Task Parent
 |         | RO Access  ______
 |         |---------->|Page X|    writeの試み
 |_________|           |______|
                          /|\
                           |
 Task Child                |
 |         | RO Access     |
 |         |----------------
 |_________|
```

3-) 最終的な構成: タスクParentとタスクChildは個別のページコピー、XとYを持つ。
```
 Task Parent
 |         | RW Access  ______
 |         |---------->|Page X|
 |_________|           |______|


 Task Child
 |         | RW Access  ______
 |         |---------->|Page Y|
 |_________|           |______|
```
