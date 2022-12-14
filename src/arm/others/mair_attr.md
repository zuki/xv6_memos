# MAIR_EL1の属性

訳注: MAIRはページ属性をまとめてインデックス化し、3ビットで複雑な属性を設定
できるようにするための機構である。

## `Attr<n>、bits[8n+7:8n]、fro n = 7 to 0`

Longデスクリプタ形式の変換テーブルのAttrIndx[2:0]エントリに設定する
メモリ属性エンコーディングである。AttrIndx[2:0]にはAttr<n>の<n>の値を
指定する。

## Attrは以下のようにエンコードする。

| Attr | 意味 |
|:-----|:-----|
|0b0000dd00 |デバイスメモリ。デバイスメモリの種類については'dd'のエンコーディングを参照。|
|0b0000dd01 |FEAT_XSが実装されている場合は、XS属性が0に設定されたデバイスメモリ。デバイスメモリのタイプについては'dd'のエンコーディングを参照してください。それ以外の場合は、UNPREDICTABLE。|
|0b0000dd1x |UNPREDICTABLE.|
|0booooiiii, (oooo != 0000 and iiii != 0000) | ノーマルメモリ。ノーマルメモリのタイプについては、'oooo'と'iiii'のエンコーディングを参照してください。|
|0b01000000 | FEAT_XSが実装されている場合は、XS属性が0に設定されたノーマルなインナーキャッシュ不能、アウターキャッシュ不能なメモリ。それ以外の場合はUNPREDICTABLE。|
|0b10100000 |FEAT_XSが実装されている場合は、ノーマルなインナーライトスルーキャッシュ可能、アウターライトスルーキャッシュ可能、非リードアロケート、非ライトアロケート、XS属性が0の非トランジェントメモリ。|
|0b11110000 |FEAT_MTE2が実装されている場合は、タグ付けされたノーマルなインナーライトバック、アウターライトバック、リードアロケート、ライトアロケート、非トランジェントなメモリ。それ以外の場合は、UNPREDICTABLE。|
|0bxxxx0000, (xxxx != 0000, xxxx != 0100, xxxx != 1010, xxxx != 1111) |UNPREDICTABLE.|

### 'dd'は以下のようにエンコードする。

|dd |意味|
|:--|:---|
|0b00 |デバイス-nGnRnE メモリ|
|0b01 |デバイス-nGnRE メモリ|
|0b10 |デバイス-nGRE メモリ|
|0b11 |デバイス-GRE メモリ|

### 'oooo'は以下のようにエンコードする。

|'oooo' |意味|
|:------|:---|
|0b0000 |Attrのエンコーディング参照|
|0b00RW, RW not 0b00 |ノーマルメモリ、アウターライトスルートランジェント|
|0b0100 |ノーマルメモリ、アウターキャッシュ不能|
|0b01RW, RW not 0b00 |ノーマルメモリ，アウターライトバックトランジェント|
|0b10RW |ノーマルメモリ，アウターライトスルー 非トランジェント|
|0b11RW |ノーマルメモリ，アウターライトバック 非トランジェント|

R = アウターリードアロケートポリシー、W = アウターライトアロケートポリシー。

### 'iiii'は以下のようにエンコードする。

|'iiii' |意味|
|:-----|:----|
|0b0000 |Attrのエンコーディング参照|
|0b00RW, RW not 0b00 |ノーマルメモリ、インナーライトスルークランジェント|
|0b0100 |ノーマルメモリ、インナーキャッシュ不能|
|0b01RW, RW not 0b00 |ノーマルメモリ，インナーライトバックトランジェント|
|0b10RW |ノーマルメモリ，インナー ライトスルー 非トランジェント|
|0b11RW |ノーマルメモリ，インナーライトバック 非トランジェント|

R = Inner Read-Allocate Policy, W = Inner Write-Allocate Policy.

### 'oooo'と'iiii'フィールドのRビットおよびWビットの意味は次のとおり。

|R/W |意味|
|:---|:---|
|0b0 |割り当てなし|
|0b1 |割り当てあり|

FEAT_XSが実装されている場合、ステージ1のInner Write-Back Cacheable、Outer Write-Back
CacheableのメモリタイプはXS属性が0に設定されます。

ウォームリセット時には、このフィールドはアーキテクチャー上のUNKNOWN値にリセットされます。

## xv6での使用例

```
#define MT_DEVICE_nGnRnE_FLAGS  0x00    // nGnRnEデバイスメモリとして使用する
#define MT_NORMAL_FLAGS         0xFF    // アウターかつインターライトバック非トランジェントのノーマルメモリであり、RWともに割り当てあり
```

# デバイス固有のメモリ属性であるGREについて

出典: [AArch64 Device Memory Attributes](https://www.linkedin.com/pulse/aarch64-device-memory-attributes-om-narasimhan)

## Gather(G)/non-Gather(nG)

デバイスメモリのブロックに対してGather属性が設定されている場合、以下のことを意味します。

- 同一のメモリロケーションに対する同一タイプの複数のメモリアクセス（複数のリード
  または複数のライト）を1つのトランザクションにまとめることができる。
- ブロック内の異なるメモリロケーションに対する同じタイプの複数のメモリアクセスを、
  インターコネクト上で1つのメモリトランザクションにまとめることができる。

これらの動作は、メモリロケーションの順序付けとコヒーレンシのルールに従う場合にのみ
許可されます。矛盾が生じた場合は、順序付けとコヒーレンシのルールが優先されます。

これらの動作が最も顕著に現れるのは、エンドポイントの位置（メモリ）でのメモリアクセス
数が、実行中のエグゼクティブが生成するアクセス数よりも少ないことです。補足すると、
ギャザービットがクリアされているメモリ領域では、エンドポイントの位置でのメモリアクセス
数は、実行中のエクゼクティブが生成するメモリアクセス数と全く同じになります。

### いくつかの重要なポイント

- メモリバリア(DMB/DSBやそれに類するもの)を越えたギャザリングは許されません。(まあ、
  許可されるためのいくつかの条件がありますが、それは後の議論のために置いておきます)。
- メモリフェンス（ロード・アクワイア、ストア・リリース）を越えたギャザリングは許可
  されていません。
- ターゲットメモリが非ギャザーの場合、そのメモリからの読み出しは、キャッシュや
  メモリシステムのそのアドレスのターゲットエンドポイントの一部ではないバッファからは
  できません。
- 言い換えれば、リードはターゲットエンドポイントに到達しなければならず、リザルトは
  ターゲットエンドポイントから結果を返さなければならない。
- ARMアーキテクチャでは、Gather属性のPE観察可能な動作を定義しています。実装は、PEが
  観察できない方法でギャザーを実行することは自由です。
- つまり、Gather属性のセマンティクスでは、ターゲットエンドポイントがどのようにメモリ
  アクセスを実装するかは定義されていません。

デバイス内部の様々なレジスタにマップされ、MMIOインタフェースを通じて公開されるデバイス
メモリは、一般的に非ギャザーとして帰属します。そのようなレジスタの例を以下に示します。

- 制御レジスタ。異なるビットが異なるデバイスの側面や動作を制御します。このような
  レジスタへの書き込みを組み合わせると、予期せぬ動作を引き起こすことがあります。
- 副作用のある読み取り専用レジスタ。多くの場合、ステータス・レジスタはクリア・オン・
  リード（CoR）レジスタとして実装されます。このようなレジスタのリードアクセスは、
  個別に行う必要があり、組み合わせてはいけません。

## Reorder(R)/non-Reorder(nR)

非Reordering属性を持つすべてのデバイスメモリについて、サイズSIZE（デバイスによって
定義される）のメモリに到達するメモリ・アクセスの順序は、PE内のエグゼクティブによって
生成される順序と同じでなければなりません。アクセスはプログラムの順序で表示されなければ
なりません。

注: SIZEは、一般的にDMB命令で操作されるメモリサイズと同じと定義されています。

非Reorder属性が推奨されるメモリの例は、1つ以上のPCIe BARを介してホストMMIOに外部
マッピングされたPCIeデバイスの内部レジスタ空間です。

Reorder属性が推奨されるメモリの例は、1つまたは複数のPCIe BARを介してホストMMIOに
外部からマッピングされているPCIeデバイスの内部RAMメモリです。このような場合、
実装者は並べ替えられたトランザクションが予期せぬ動作を引き起こさないようにする
必要があることに注意してください。

実装では、Reordering属性が設定されたアクセスを、非Reordering属性と一致する要件と
一致する方法で実行することが許可されています。しかし、その傍証は真ではない。

Reorderをサポートしない実装では、以下の間のアクセスについて満たすべき追加の
順序付け要件はありません。

- 非順序付け属性を持つ領域と順序付け属性を持つ領域。
- 非順序付け属性を持つ領域と通常のメモリ。
- 非順序付け属性を持つ領域と他のペリフェラル（他のペリフェラルで定義されたアクセスサイズ）。

## Early Write Acknowledgement (E/nE)

PEが書き込みの確認をエンドポイントから行うことを要求するエンドポイントのメモリに
対して、非Early Write Acknowledgement属性は以下のことを保証します。

- 書き込み操作のエンドポイントのみが、Write Acknowledgementを返すことができます。
- 中間エージェントはWrite Acknowledgementを返すことができません。

つまり、DMB命令は、エンドポイントのメモリへの書き込みがエンドポイントに到達し、確認されて初めて完了するということです。

### 注意

- この属性は、確認応答を発信すべきエンドポイントを定義するものであり、確認応答を
  出す順序を制御するものではありません。
- PCI/PCIe コンフィギュレーションスペースは、nE属性の良い例です。PCIeデバイス内の
  メモリ領域で、ポストされたメモリの書き込みを期待するものは、E 属性の良い例です。

## デバイスのメモリと命令フェッチ

ソフトウェアは、CPUにデバイスメモリから命令をフェッチさせてはいけません。そのような
フェッチは、未定義の動作を引き起こす可能性があります。しかし、そのようなフェッチが
許される状況もあります。

- デバイスメモリの特定の領域が現在の例外レベルで実行不可能とマークされておらず、
  アドレス変換が有効な場合、その領域を投機的な命令アクセスに使用することができます。
- デバイスメモリ内の特定の領域が現在の例外レベルに対して実行不可能とマークされておらず、
  分岐命令によってPCがメモリのそのような領域を指すようになった場合、実装は以下のことが
  可能です。
- パーミッション・フォールトを生成する。
- 指定された領域を通常のキャッシュ不可能なメモリとして扱う。

# ARM64ノーマルメモリの属性について

出典: [AArch64 Normal Memory Attributes](https://medium.com/@om.nara/arm64-normal-memory-attributes-6086012fa0e3)

この記事では、AArch64 Normal Memory Attributesの一部について説明します。システムメモリと
ノーマルメモリの説明については、[この記事](https://medium.com/@om.nara/arm64-system-memory-fbf71dce37ee)を参照してください。
デバイスメモリの属性については、全勝を参照してください。

メモリ領域の多くの特性（メモリタイプ、キャッシュ可能性、共有可能性...など）は、そのメモリ領域に
関連する属性によって制御されます。前回の記事では、デバイスメモリに関連する属性について説明しました。
今回の記事では、それを補完するために、通常のメモリについて同様に説明します。

通常のメモリは一般に複数のページに分割され、ページテーブルをたどり歩くように実装されたプロセッサの
ハードウェア回路を介してアクセスされます。このアクセスは、ページテーブルエントリに関連付けられた
属性によって制御され、規制されます。

## テーブルデスクリプタ

この記事では、さまざまなページテーブルエントリのフォーマットについて説明します。簡単に参照
できるように、ここではそれを再掲します。

ページテーブルのテーブルデスクリプタエントリは、ビット`[1:0] == 0b11`であり，レベル3にあることは
ありません。

### テーブルデスクリプタフォーマット

![テーブルデスクリプタフォーマット](screens/fig_pt_descriptor.png)

### テーブルデスクリプタの属性

Stage2のテーブルデスクリプタには属性フィールドがないことに注意してください。次の表は、
Stage1 Tableエントリーに関連する属性を説明しています。

![テーブルデスクリプタの属性](screens/fig_pt_descriptor_attr.png)

### NSTable

このビットは、ディスクリプタ内のテーブル識別子が、セキュアな状態からのアクセスにおいて、
セキュアなPA空間（NSTable == 0）にあるか、ノンセキュア（NSTable == 1）メモリにあるかを示します。

`NSTable == 1`の場合、以降の検索でページやブロックの記述子のNSビットの値は無視され、参照される
ブロックやページはNon-Secureメモリにあることになります。また、検索されたテーブルのNSTableの
値は無視され、そのようなテーブルのエントリはNon-Secureメモリを参照します。さらに、セキュアな
状態でフェッチされたエントリは、`nG == 1` (non-global) として扱われます。

注：NSTableビットの効果は、翻訳テーブルウォークの後続のすべてのルックアップに適用されます。

## ブロック/ページ記述子

この記事では、さまざまなページテーブルエントリのフォーマットについて説明しています。簡単に
参照できるように、ここではそれを再掲しています。

ページテーブルのブロック記述子エントリは、ビット`[1:0] == 0b01`です。

ページテーブルのページ記述子エントリは、`bit[1:0] == 0b11`であり，レベルは3でなければ
なりません。

### ページ記述子のフォーマット

![ページ記述子のフォーマット](screens/fig_page_descriptor.png)

### ブロック記述子のフォーマット

![ブロック記述子のフォーマット](screens/fig_block_descriptor.png)

ページテーブルエントリ（ブロック記述子、ページ記述子、テーブル記述子）のフィールドの意味は
使用している翻訳レジームによって異なります。翻訳レジームとメモリトランスレーションのステージに
ついては、この記事を参照してください。

### ブロック/ページ記述子の属性

![ブロック/ページ記述子の属性](screens/fig_page_descriptor_attr.png)

### UXN

- ビット54は、Stage1が2つのVAレンジをサポートするあらゆる変換レジームのStage1テーブルエントリに
  対してのみUXNとして定義されます。
- このフィールドはEL0のコード実行だけに適用されます。
- 0の場合、コードの実行はEL0に対して許可されます。

### XN

- ビット54は、Stage1が1つのVA範囲しかサポートできない翻訳レジームに対してXNとして定義されます。
- このフィールドは、翻訳が適用されるすべての例外レベルに適用されます。
- 0であればコードの実行が許可され、そうでなければ実行は許可されません。

### PXN

- ビット53は、Stage1が2つのVA範囲をサポートする任意の翻訳レジームのStage1テーブルエントリに
  対してのみPXNとして定義されます。単一のVAレンジしあサポートしていないTRでは、このビットは
  RES0であり、ハードウェアによって無視されます。
- このフィールドは、EL0コード実行よりも高い例外レベルのみに適用されます。
- 0の場合、コードの実行が許可されます。

### Contiguous

Stage2の最初のルックアップにのみ適用され、すべての属性と許可のセットが同じであるため、
単一のTLBエントリにキャッシュできることをハードウェアに示唆します。

### DBM

このビットは、メモリのページまたはセクションの内容が変更されていることを示します (ダーティステート)。
このビットは、アクセスフラグのハードウェア管理が有効になっている場合、オプションでハードウェアが
制御できます。TCR_ELxレジスタのフィールドHD (TCR_ELx.HD) を設定することで、このビットのハードウェア
管理を有効にすることができます。

これは、ハードウェア実装でサポートされるオプション機能です。この機能がサポートされていない場合、
TCR_Elx.HDはRES0（ハードウェアが値を無視）となります。

### nG

ビット11は、Stage1が2つのVA範囲をサポートする任意の変換レジームのStage1テーブルエントリに対してのみ、
nGとして定義されます。単一のVA範囲しかサポートしていないTRでは、このビットはRES0であり、
ハードウェアによって無視されます。

### AF

- このビットは、ARMv8.0ではソフトウェアで管理されていますが、拡張機能であるARMv8.1-TTHMが実装
  されている場合は、オプションでハードウェアで管理することができます。
- AFが0に設定されてから初めてページやブロックにアクセスされたときに表示されます。
- AF == 0 の記述子エントリが TLB に読み込まれようとするたびに、アクセスフォルトが発生します。
  SWはその記述子にはAF=1を設定する必要があります。

### SH[1:0]

- 通常のキャッシュ可能なメモリにのみ適用されます（AttrInx[2:0]で決定）。
- Stage1とStage2の意味は以下の通りです　（0b01は予約）。
  - 0b00 -- 共有不可
  - 0b10 -- Outer 共有可能
  - 0b11 -- Inner 共有可能

### AP[2:1] - データアクセス許可

- stage1のトランスレーション(複数のELにまたがるTR)の場合。
  - 0b00：EL0からはアクセス不可、全ての上位階層からはRead/Writeアクセス可能。
  - 0b01：EL0からのRead/Write、全ての上位階層からのRead/Writeアクセス。
  - 0b10: EL0からのアクセス不可、全ての上位階層からのRead onlyアクセス。
  - 0b11：EL0からのRead onlyアクセス、上位のすべてのレベルからのRead onlyアクセス。
- Stage1のトランスレーション(単一のElsのTR)では、AP[1]はRES1です。
  - 0b0：Read/Write
  - 0b1: Read only.
- stage2のトランスレーションでは、S2APフィールドがアクセス許可に影響し、AP[2:1]と
  組み合わせて使用されます。

### NS - 非セキュア・アクセス・コントロール

- 0b1: SecureおよびNon-Secureの実行状態でアクセスが許可されます。
- 0b0：Secure の実行状態でのみアクセスが許可されます。

### MemAttr[3:0] - MAIR_ELxレジスタ内のメモリ属性インデックス

- デバイスメモリとノーマルメモリを区別することができる。
- 通常のメモリでは、キャッシュ可能性(Writeback/Writethrough)、共有可能性(Inner/outer)、
  アロケート(Read/Write)のポリシーヒントなどが含まれています。

## 書き込み可能なメモリからの実行を防ぐ

AArch64では、拡張セキュリティ対策として、レジスタフィールドSCTLR_ELx.WXNで、すべての
例外レベルと変換レジームで書き込み可能なメモリからの実行を防止できます。

- EL0以上のELに適用されるTRでは、対応するSCTLR_Elx.WXN == 1のときに
- アドレス変換のstage1でEL0から書き込み可能な全ての領域がXN == 1として扱われます。
- アドレス変換のステージ1でEL0から書き込み可能な全ての領域はPXN == 1として扱われます。
- 1つの例外レベルのみに適用されるTRでは、対応するSCTLR_Elx.WXN == 1のとき、アドレス変換の
  stage1から書き込み可能なすべての領域はUXN == 1として扱われます。

# AArch64のアクセスパーミッション

出典: [ARM Cortex-A Series Programmer's Guide for ARMv8-A](https://developer.arm.com/documentation/den0024/a/BABCEADG)

アクセスパーミッションは、トランスレーションテーブルのエントリによって制御されます。
アクセスパーミッションは、ある領域が読み取り可能か、書き込み可能か、あるいはその
両方かを制御するもので、表12.4に示すように、非特権アクセスのためのEL0と、特権アクセスの
ためのEL1、EL2、EL3で個別に設定することができます。

### 表12.4. アクセス権限

| AP | 非特権(EL0) | 特権(EL1/2/3)|
|:---|:------------|:-------------|
| 00 | アクセス不可 | 読込みと書込み|
| 01 | 読み取りと書き込み | 読み取りと書き込み|
| 10 | アクセス不可 | Read-only|
| 11 | リードオンリー | リードオンリー|

オペレーティングシステムのカーネルは実行レベルEL1で動作します。カーネルはカーネル
自身やEL0で動作するアプリケーションが使用する変換テーブルのマッピングを定義します。
カーネルは自身のコードとアプリケーションに対して異なるパーミッションを指定するため、
非特権と特権のアクセスパーミッションを区別する必要があります。実行レベルEL2で動作
するハイパーバイザーやEL3で実行するセキュアモニタは、自分たちが使用するための翻訳
スキームを持っているだけなので、アクセス許可に特権と非特権を分ける必要はありません。

もう一つのアクセス許可の種類として、実行可能属性があります。ブロックは*実行可能*
または*実行不可能*（Execute Never (XN)）としてマークすることができます。*UXN* (Unprivileged Execute Never) と*PXN* (Privileged Execute Never) という属性を別々に
設定することができ、これを使って、例えば、アプリケーションコードがカーネル特権で
実行されたり、非特権状態でカーネルコードを実行しようとしたりするのを防ぐことが
できます。これらの属性を設定すると、プロセッサがメモリロケーションに対して投機的な
命令フェッチを実行することができなくなり、投機的な命令フェッチが、例えばFIFO
（First in, First out）ページ交換キューなど、そのようなアクセスによって妨害される
可能性のあるロケーションに誤ってアクセスすることがなくなります。したがって、
デバイス領域は常にExecute Neverとしてマークされていなければなりません。

### 図 12.18 デバイス領域

![fig_12.18](screens/fig_12_18.png)

SCTLRレジスタ内の以下のビットを使用して、書き込み可能な領域をExecute Neverとして
扱うようにプロセッサを設定することができます。

- SCTLR_EL1.WXN. EL0で書き込み可能な領域は、EL0とEL1でXNとして扱われます。EL1で
  書き込み可能な領域は、EL1でXNとして扱われます。
- SCTLR_EL2と3.WXN. ELnで書き込み可能な領域は、ELnではXNとして扱われます。
- SCTLR.UWXN. EL0で書き込み可能な領域は、EL1ではXNとして扱われます。AArch32のみの
  機能です。

SCTLR_ELnのビットは、TLBエントリにキャッシュすることができます。そのため、SCTLRの
ビットを変更しても、すでにTLBにあるエントリには影響しない場合があります。これらの
ビットを変更する場合は、TLBの無効化とISBシーケンスが必要です。ISBバリアについては、
[バリア](https://developer.arm.com/documentation/den0024/a/Memory-Ordering/Barriers?lang=en)を参照してください。

# ステージ2トランスレーション

出典: [Learn the architecture: AArch64 Virtualization](https://developer.arm.com/documentation/102142/0100/Stage-2-translation)

## ステージ2トランスレーションとは何か？

ステージ2トランスレーションとは、ハイパバイザーが仮想マシン（VM）のメモリの表示を制御することです。
具体的には、ハイパバイザーが、VMがアクセスできるメモリマップされたシステムリソースと、それらの
リソースがVMのアドレス空間のどこに表示されるかを制御することを可能にします。

このメモリアクセスの制御機能は、アイソレーションやサンドボックス化に重要です。ステージ2変換は
VMが自分に割り当てられたリソースのみを見ることができ、他のVMやハイパーバイザに割り当てられた
リソースを見ることができないようにするために使用できます。

メモリアドレス変換の場合、ステージ2変換は第2段階の変換となります。これをサポートするためには、
以下のように、ステージ2テーブルと呼ばれる新しい翻訳テーブルが必要となります。

![stage 2 translation](screens/fig_stage2_translation.png)

オペレーティングシステム（OS）は、仮想アドレス空間から物理アドレス空間と思われる場所に
マッピングする一連の変換テーブルを制御します。しかし、このプロセスでは、実際の物理アドレス
空間への2回目の変換が行われます。この2回目の変換は、ハイパーバイザーによって制御されます。

OSが制御する変換を「ステージ1変換」、ハイパーバイザーが制御する変換を「ステージ2変換」と
呼びます。OSが物理メモリだと思っているアドレス空間をIPA（Intermediate Physical Address）空間と
呼びます。

注：アドレス変換の仕組みについては[メモリ管理](https://developer.arm.com/architectures/learn-the-architecture/memory-management)をご参照ください。

ステージ2で使用する変換テーブルのフォーマットは、ステージ1で使用するものと非常によく似ています。
しかし、ステージ2では一部の属性の扱いが異なり、「タイプ」「ノーマル」「デバイス」は、MAIR_ELx
レジスタを経由せず、テーブルエントリに直接エンコードされます。
