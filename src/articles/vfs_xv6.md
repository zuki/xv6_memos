# XV6への仮想ファイルシステムレイヤの実装

[A Virtual Filesystem Layer implementation in the XV6 Operating System / Caio Araújo Neponoceno de Lima](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwifhZPJ14nzAhUSAIgKHcxJBycQFnoECAQQAQ&url=https%3A%2F%2Fmaups.github.io%2Fpapers%2Ftcc_004.pdf&usg=AOvVaw3muz4kRlp7OOUyxbJfHdH7)

## 1. 序論

コンピュータの開発者にとって、オペレーティングシステムの基本的な概念は非常に重要です。
なぜなら、ソフトウェアの大半は、オペレーティングシステムの中でアプリケーションとして
動作するように開発されているからです。このシナリオでは、オペレーティング・システムが
性能面でのボトルネックとなります。オペレーティングシステムは一次メモリやストレージへの
アクセスなど、コンピュータのすべてのリソースを管理するからです。現在、私たちはGNU/Linuxや
BSDなどの最先端の商用OSを利用することができますが、そのソースコードを理解することは困難です。
そこで、MIT CSAIL並列分散オペレーティングシステムグループは、XV6を開発しました。

XV6は、デニス・リッチーとケン・トンプソンが開発したUnix Version 6 (v6)を、最新のx86ベースの
マルチプロセッサ上で動作するように再実装したもので、ANSI Cで記述されています。2006年夏、
MITのオペレーティングシステムコース「6.828: Operating System Engineering」のために開発
されました。XV6のソースコードは、UnixやLinuxなどの最新のオペレーティングシステムとは異なり、
1学期でカバーできるほどシンプルであり、また、Unix系オペレーティングシステムの主要な概念を
カバーすることができます。

XV6は、教育用のOSとしてはユニークな存在ではありません。Tanenbaum (2007) は、自著で取り上げた
概念を適用するために、マイクロカーネルベースのオペレーティングシステムである Minix を開発
しました。その第3版では、主な目的が教育用から信頼性の高い自己修復型マイクロカーネルOSの作成に
シフトし、多くの作業や学術研究が行われているので、OSの現代的な概念を見つけることが可能です。
XV6とMinix 3の最大の違いは設計上のもので、XV6はモノリシックカーネル、Minix 3はマイクロ
カーネルであるためです。また、Minix 3は、XV6とは異なり、コードベースが豊富で、ネットワーク
スタックなどの高度な概念が実装されているため、ソースコードを修正する際には、詳細な調査が
必要となります。

XV6に実装されているファイルシステムは、オリジナルのUnix v6のファイルシステムをよりシンプルに
したものです。しかし、その設計はXV6の他のコードと完全に結合しており、複数のファイルシステムを
同時にマウントするなど、カーネルのこの部分の変更を必要とする作業は困難です。そのため、この
バージョンのXV6に最新のファイルシステム開発技術を導入するのは簡単ではありません。

本研究では、XV6上に複数のファイルシステムを共存させるための仮想ファイルシステム層(VFS)の
実装を紹介し、現代のファイルシステムの設計・実装に影響を与える重要な概念について議論します。
また、概念実証として、Linuxユーザが使用する最も基本的なファイルシステムの一つであるEXT2
（Second Extended Filesystem）の基本的な実装を紹介します。

我々の主な目的は、Unix系環境におけるファイルシステム開発の知識を、シンプルかつ実用的に
文書化して提供することにあります。XV6のシンプルさゆえに、このVFSの実装は、最小限の労力で
新しいファイルシステムをサポートすることを可能にします。

この論文の構成は以下の通りです。第2章では、さまざまなOSで実装されているVFSについて説明します。
第3章では、VFSを実装するために必要な、XV6のオリジナルソースコードの変更点を紹介します。

第4章では、XV6 VFSに対する我々のEXT2実装と、新しいファイルシステムを実装するための戦略を
紹介します。第5章では、我々の実装の操作性を確認するための実験を行い、第6章では、結論と
今後の課題を述べます。

## 2 関連システム

複数のファイルシステムが共存する以前は、ファイルシステムの設計はカーネルと連動していました。
かつては、Unix系OSのすべての実装で使用されるファイルシステムの種類が1つしかなかったため、
このアプローチはうまくいっていました。バークリー大学でBSD Fast File Systemが作られたとき、
新しいファイルシステムをサポートする必要性がようやく現実のものとなりました。これは、Unixが
普及して以来、異なるファイルシステムのレイアウトが初めて登録されたものでした。その後、様々な
ファイルシステムが作成され、その結果、ファイルシステムを操作するための抽象的なレイヤーが
商用カーネルに必須の機能となったのです。この抽象層の例としては、Unix System V Revision 3.0の
File System Switch（FSS）、SunOSのSun VFS/Vnode、LinuxのVFSレイアyどがあります。これらに
ついては後述します。

### 2.1 ファイルシステムスイッチ

カーネル内に複数のファイルシステムが共存する問題を解決するために，Unix System V Revision
3.0 (PATE; BOSCH, 2003)では，ファイルシステムスイッチ(FSS)が導入されました。この作品で
述べられているように「SVR3の主な目標の1つは、複数の異なるファイルシステムタイプが同時に
共存できるフレームワークを提供することでした」。

そのため、マウントを呼び出すたびに、呼び出し側がファイルシステムタイプを指定することが
できました。カーネルのファイルシステム非依存層とファイルシステム依存層の大きな違いは、
インコアのinodeを搭載していることです。FSSで実装されているin-core inodeには、異なる
ファイルシステムタイプ間で抽象化されたフィールド（ユーザやグループのID、ファイルサイズ、
アクセス許可など）に加え、ファイルシステム固有のデータを参照する機能が含まれています
（PATE; BOSCH, 2003）。このようにして、各ファイルシステムタイプは、そのメタデータ/データの
ディスク上の表現が非常に異なっていても、FSSレイヤによって操作することができます。

FSS の設計では、ファイルシステム固有の操作がすべて定義されている fstypsw と呼ばれる
構造体を通じて、ファイルシステムの操作に関する抽象的な動作を実装しています。複数の
ファイルシステムをサポートするため、カーネルはこの構造体のグローバルな配列を維持し、
各エントリは可能なファイルシステムを表します。そして，ファイルにアクセスするための
システムコールが行われたとき，メモリ内のファイルを表すinodeは，適切な操作のリストに
アクセスするために，正しいファイルシステムのインデックスを持つだけでよいのです。
FSSが使用するファイルシステム固有の操作を表1に示します。ご覧のように、このリストの
操作は、Unix 系のシステムコール (mount, umount, statf, ioctl など) およびそのサブ
プロシージャ (fs_namei, fs_access など) に関連しています。

<center><strong>表1: File System Switchの操作関数一覧</strong></center>

| FSS操作関数 | 説明 |
|:------------|:-----|
|fs_init | 各ファイルシステムは、カーネルの初期化時に呼び出される関数を指定することができる。これにより、最初のマウントコールの前にファイルシステムが任意の初期化タスクを実行することができる |
|fs_iread | （パス名解析の際に） inodeを読み込む |
| fs_iput | inodeを開放する |
|fs_iupdat | inodeのタイムスタンプを更新する |
|fs_readi | ファイルからデータを読み込む際に呼び出される |
|fs_writei | ファイルにデータを書き出す際に呼び出される |
|fs_itrunc | ファイルを切り捨てる |
|fs_statf | stat()により要求されたファイル情報を返す |
|fs_namei | パス名検索の際に呼び出される |
|fs_mount | ファイルシステムをマウントする際に呼び出される |
|fs_umount | ファイルシステムをアンマウントする際に呼び出される |
|fs_getinode | パイプ用のファイルを割り当てる |
|fs_openi | デバイスのopen関数を呼び出す |
|fs_closei | デバイスのclose関数を呼び出す |
|fs_update | スーパーブロックをデスクに同期させる |
|fs_statfs | statfs()とustat()で使用される |
|fs_access | アクセス権限を検査する |
|fs_getdents | ディレクトリエントリを読み出す |
|fs_allocmap | デンマンドページング用のブロックリストマップを構築する |
|fs_freemap | デンマンドページング用のブロックリストマップを開放する |
|fs_readmap | ブロックリストマップを使ってページを読み込む |
|fs_setattr | ファイル属性を設定する |
|fs_notify | ファイル属性が変更された際にファイルシステムに通知する |
|fs_fcntl | fcntl()システムコールを処理する |
|fs_fsinfo | ファイルシステム固有の情報を返す |
f|s_ioctl | ioctl()システムコールに呼応して呼び出される |

FSSアーキテクチャは、Unixで複数のファイルシステムをサポートした最初の実用的なシステムです。
この論文で紹介している以下のアーキテクチャは、レイヤ構成やファイルシステム依存の機能に
若干のバリエーションがあるものの、広くFSSアーキテクチャの影響を受けています。

### 2.2 Sun VFS/Vnode アーキテクチャ

このVFSの実装は、Sun MicrosystemのSunOS(KLEIMAN, 1986)用に開発されたもので、以下の4つの目標に
基づいています。

1. ファイルシステムの実装は、ファイルシステムに依存しない層とファイルシステムに依存する層に
   明確に分けられること。この2つの間のインターフェースを明確に定義すること。
2. 4.2BSD Fast File System (FSS)のようなローカルディスクファイルシステム、MS-DOSのような
   非UNIX系ファイルシステム、NFSのようなステートレスファイルシステム、RFSのようなステート
   フルファイルシステムをサポートすること。
3. NFSやRFSなどのリモートファイルシステムのサーバ側をサポートできること。
4. インターフェイス上のファイルシステム操作は、いくつかの操作をロックに包含させる必要が
   ないようなアトミックなものでなければならない。

FSSとVFS/Vnodeの主な違いは、VFS/Vnodeアーキテクチャは、抽象的なファイルシステム操作のための
インターフェースであるvfsopsと、個々のファイル操作を可能にするインターフェースであるvnopsと
いう2つのメジャーな構造に基づいていることです。図1は、VFS/Vnodeアーキテクチャの図であり、
SunOSのコンポーネント間のハイレベルな相互関係を示しています。見てわかるように、VFS/Vnode層で
あるVFS/VOP/veneer層は、他のカーネルコンポーネントとサポートされるファイルシステムを分離して
います。さらにこの図は、抽象層がディスクレスファイルシステム（ネットワークベースのファイル
システムであるNFS（Network File System）など）を扱えることを示しています。

![SunOS VFS/Vnodeアーキテクチャ](screens/sunos_vfs.png)

<center><strong>図1: SunOS VFS/Vnodeアーキテクチャ</strong></center>

<br/>

このアーキテクチャの目的の1つは、ディスクレスでUnix以外のファイルシステムをサポートすること
なので、インコアinode（vnodeとも呼ばれる）は、先に紹介したFSSインコアinodeのようにすべての
ファイルシステムに共通するデータのみを格納します。この構造体は、ファイルシステム非依存層が
必要とするすべての情報を格納し、ファイルシステム依存層が使用するプライベートデータへの
ポインタを格納しています。

VFS層（vfsopsで表される）は、特定のファイルシステムに関連するすべての操作を格納する責任が
あります。VFS層が格納する操作のセットは、表2に記載されています。

<center><strong>表2: vfsops操作関数一覧</strong></center>

| VFS操作関数 | 説明 |
|:------------|:-----|
| vfs_mount | ファイルシステムを正確にマウントするために呼び出される |
| vfs_unmount | ファイルシステムをアンマウントするために呼び出される |
| vfs_root | このファイルして無のルートvnodeを返す。パス名解析の際に呼び出される |
| vfs_statfs | statfs()システムコールに呼応してファイルシステム固有の情報を返す |
| vfs_sync | ファイルデータとファイルシステム構造データをディスクにフラッシュする。システムクラッシュが生じた際のデータロスを最小限にするファイルシステムの堅牢性を提供する |
| vfs_fid | 指定したvnodeのためのファイルハンドルを作成するためにNFSにより使用される |
| vfs_vget | vfs_fidを呼び出して得られたファイルハンドルをさらなる操作をおこぬためにvonodeに変換するためにNFSにより使用される |

Vnode層（vnopsで表される）は、ファイルに対するすべての操作を格納します。この一連の操作は表3に
記載されています。

<center><strong>表3: vnops操作関数一覧</strong></center>

| VNODE操作関数 | 説明 |
|:------------|:-----|
| vop_open | この関数はハードウェアデバイスを表す名前空間内のファイルであるデバイススペシャルファイルにのみ適用される。直前に呼び出されたvoplookupからvnodeが返された後に呼び出される |
| vop_close | デバイススペシャルファイルにのみ適用される。直前に呼び出されたvoplookupからvnodeが返された後に呼び出される |
| vop_rdwr | ファイルの読み書きをするために呼び出される。I/Oに関する情報はuio構造体に渡される |
| vop_ioctl | デバイスドライバに渡すことができるファイルに対するioctlを呼び出す |
| vop_select | select()を実装する |
| vop_getattr | stat()などのシステムコールに呼応して呼び出される。この関数はvattr構造体を設定する。これはstat構造体を経由して呼び出し元に返すことができる。|
| vop_setattr | これもvattr構造体を使用する。呼び出し元がファイルサイズやモード、ユーザID、グループID、タイムスタンプなど、様々なファイル属性を設定することを可能にする。 |
| vop_access | 呼び出し元がファイルの読み込み、書き込み、実行の各権限をチェックすることを可能にする。この関数に渡されるcred構造体は呼び出し元のクレデンシャルを保持している |
| vop_lookup | namei()実装の一部を置き換える関数である。ディレクトリvnodeと要素名を受け取り、ディレクトリ要素のvnodeを返す |
| vop_create | 指定されたディレクトリvnodeに新しいファイルを作成する。ファイル属性はvattr構造体で渡される |
| vop_remove | ディレクトリエントリを削除する |
| vop_link | link()システムコールを実装する |
| vop_rename | rename()システムコールを実装する |
| vop_mkdir | mkdir()システムコールを実装する |
| vop_rmdir | rmdir()システムコールを実装する |
| vop_readdir | 指定されたディレクトリvnodeからディレクトリエントリを読み込む。getdents()システムコールに呼応して呼び出される |
| vop_symlink | symlink()システムコールを実装する |
| vop_readlink | シンボリックリンクのコンテンツを読み込む |
| vop_fsync | メモリ上の更新されたファイルデータをディスクにフラッシュする。fsync()システムコールに呼応して呼びされる |
| vop_inactive | カーネルのファイルシステム非依存レイヤがvnodeに保持していたデータを開放する際に呼び出される。ファイルシステムはvnodeを開放する事が可能になる。 |
| vop_bmap | デマンドページングで使用され、仮想メモリ（VM）サブシステムが論理ファイルオフセットを物理ディスクオフセットにマッピングできるようにする |
| vop_strategy | VMおよびバッファキャッシュ層がvop_bmap()の呼び出しに続いて、ファイルのブロックをメモリに読み込む際に使用される |
| vop_bread | 指定されたvnodeから論理ブロックを読み込み、データを参照するバッファキャッシュからバッファを返す |
| vop_brelse | vop_bread関数で返されたバッファを開放する |

XV6のVFSアーキテクチャは、そのシンプルさと、Unix以外のファイルシステムやディスクレスファイル
システムに対応する柔軟性から、この実装に強い影響を受けています。


### 2.3 Linux VFSレイヤ

Linux VFSの実装は、移植性と性能という複雑な要件を満たすことを主な理由として、設計面で
最も成功したものの一つです。データ構造のファミリーは、抽象的なファイルモデルを表して
います。また、Linuxの実装では、ファイルシステムに依存しない層が必要とするデータのための
汎用構造体の設計を踏襲し、ファイルシステムに依存する情報へのデータやポインタを維持
しています。

Linux VFSの主要な構造体は以下の4つです。

- _superblock_: この構造体は、マウントされたファイルシステムのスーパーブロックを
  格納し、グローバルファイルシステムのメタデータを含む。
- _inode_: ファイルとそのメタデータ(アクセス許可など)を格納する構造体である。
- _dentry_: ディレクトリエントリを表す構造体で，ファイルを構成する1つの要素である。
- _file_: この構造体は実行中のプロセスに付随するオープンファイルを表す．

提示された各構造体には，ファイルシステム依存関数を格納した操作グループへの
ポインタがあります。これらの構造体はファイルシステム依存関数を実行する必要がある際に
カーネルから呼び出されるメソッドを記述しています。

- *super_operations*: write_inode()など、カーネルが特定のファイルシステム上で呼び出す
  メソッドが含まれる。
- *inode_operations*: create()やmkdir()など、ファイルシステムのinodeを操作するメソッドが
  含まれる。
- *dentry_operations*: d_compare()など、特定のディレクトリエントリを操作するメソッドが
  含まれる。
- *file_operations*: open()やclose()など、オープンされたファイルを操作するための
  操作が含まれる。

これらの構造体とその操作は、これまでのVFSに比べてはるかに複雑です。なお、Linux VFSが
どのような抽象情報をカバーしているかについては、Robert Love氏の著作(LOVE, 2010)を読む
ことをお勧めします。

Linux VFSの大きな利点は、世界最大のオープンソースプロジェクトであることから、常に進化
していることです。そのため、このVFSの実装では、ページキャッシュの利用など、他では
実装されていない機能もあります。

## 3 XV6 VFS の実装

本節では、XV6におけるVFSの実装を紹介します。図2にXV6のVFSアーキテクチャを示しますが
SunOSのVFS/Vnodeアーキテクチャの影響を受けていることがわかります。これは、本研究の主な
貢献であり、後述するほとんどすべての内容は、ここで紹介したコンセプトと設計を参照して
います。

![xv6 VFSアーキテクチャ](screens/xv6_vfs.png)

<center><strong>図 2: XV6 VFS アーキテクチャ</strong></center>

 ### 3.1 XV6 VFSの主な構造体

 XV6のVFSは、2.2節で紹介したSunOSのVFS/Vnode(KLEIMAN, 1986)のアーキテクチャに強く影響を
 受けて設計されました。ファイルシステムに依存しない情報を扱うために、`inode`、`superblock`、
 `filesystem_type`という3つの主要な構造体があります。これらの構造体の実装の詳細は以下の
 通りです。

#### 3.1.1 inode

inode構造体は、ファイル(通常のファイルやディレクトリなど)を表現する役割を担っています。
inode構造体を操作するファイルシステム関連のシステムコールは、使用されているファイル
システムのタイプを知りません。inode構造体をリスト3.1に示します。

<center><strong>リスト 3.1: inode構造体</strong></center>

```c
// in-memory copy of an inode
struct inode {
    uint dev;                           // マイナーデバイス番号
    uint inum;                          // inode番号
    int ref;                            // 参照カウンタ
    int flags;                          // I_BUSY, I_VALID
    struct filesystem_type *fs_t;
    struct inode_operations *iops;      // 特定のinode操作関数
    void *i_private;                    // ファイルシステム固有情報

    short type;                         // ファイルタイプ
    short major;                        // メジャーデバイス番号(T_DEVのみ)
    short minor;                        // マイナーデバイス番号(T_DEVのみ)
    short nlink;                        // ファイルシステムのけるinode似た対するリンク数
    uint size;                          // ファイルサイズ（バイト単位）
};
```

表3.1の各変数の意味は表4にあります。

<center><strong>表 4: inode変数の説明</strong></center>

| inodeフィールド | 説明 |
|:------------|:-----|
| dev | マイナーデバイス番号を格納する。Linuxなどの商用OSではこのinodeが格納されているブロックデバイスを表す構造体を格納するのが賢明である。XV6では1種類のブロックデバイス(IDEハードディスク)しかサポートする予定がなく、ブロックデバイス構造体を持たないので、マイナーデバイス番号を格納することにした |
| inum | inodeを管理するすべての操作関数で使用されるinodeのID番号を格納する。inodeキャッシュで多用される  |
| ref | このinodeの使用を追跡する。この値が0の場合、inodeキャッシュは新しいinodeの情報を格納するためにこのスペースを再利用する |
| flags | VFSのアルゴリズムが内部的に使用するフラグを格納する |
| fs_t | ファイルシステム固有の高速アクセス操作へのポインタ |
| iops | このinodeで使用されるファイルシステム固有の操作関数へのポインタ  |
| i_private | ファイルシステム固有のデータを格納する。通常はファイルシステム固有のinode表現（例: EXT2ファイルシステムの場合は "struct ext2_inode*"）を指す。このデータはファイルシステム固有関数で使用される (inodeにデータを書き込むための`int ext2_writei(struct inode *ip, char *src, uint off, uint n)`など) |
| type | T_DIR、T_FILE、T_DEV、T_MOUNTのいずれかのファイルタイプを格納する |
| major | ファイルタイプがT_DEVの場合、デバイスのメジャー番号を格納する  |
| minor | ファイルタイプがT_DEVの場合、デバイスのマイナー番号を格納する  |
| nlink | このinodeを指しているリンクの数  |
| size | ファイルサイズ  |

#### 3.1.2 スーパーブロック

スーパーブロックは、ファイルシステムの総サイズ、利用可能なストレージ量、ブロック
サイズなどのファイルシステムのメタデータ情報を格納する役割を持つ構造体です。
ファイルシステム関連のシステムコールは、ファイルシステムに依存しない構造の抽象的な
表現として、スーパーブロックを操作します。XV6のスーパーブロックは、フィールド数の
多いLinuxのスーパーブロック実装とは異なり、リスト3.2に示すように、inodeやブロックの
割り当てといったファイルシステムの基本的な操作に必要な情報のみを格納しています。

<center><strong>リスト 3.2: superblock構造体</strong></center>

```c
struct superblock {
    int major;                      // このスーパーブロックが格納されているブロックデバイスのデバイスメジャー番号
    int minor;                      // このスーパーブロックが格納されているブロックデバイスのデバイスマイナー番号
    uint blocksize;                 // このスーパーブロックのブロックサイズ
    void *fs_info;                  // ファイルシステム固有の情報
    unsigned char s_bocksize_bits;
    int flags;                      // 利用法をマッピングするためのスーパーブロックフラグ
};
```

表3.2の各変数の意味は表5にあります。

<center><strong>表 5: superblock変数の説明</strong></center>

| inodeフィールド | 説明 |
|:------------|:-----|
| major | ブロックデバイスのメジャーID。このスーパーブロックが格納されているブロックデバイスへのアクセスに使用するドライバが使用を正しくマッピングするために、カーネルが内部的に使用する |
| minor | ブロックデバイスのマイナーID。このスーパーブロックが格納されているブロックデバイスへのアクセスに使用するドライバが使用を正しくマッピングするために、カーネルが内部的に使用する |
| blocksize | このファイルシステムのブロックサイズをカーネルに通知するために使用される |
| fs_info | ファイルシステム固有情報を格納するための汎用ポインタ |
| s_blocksize_bits | このファイルシステムのブロックサイズをカーネルに通知するために使用されるが、ビット演算で使用される |
| flags | カーネルの内部制御のためのフラグを格納する |


#### 3.1.3 ファイルシステムタイプ

XV6のVFSレイヤは、サポートしているすべてのファイルシステムを含む登録リストを保持しています。
これを整理してカーネルに格納するために、名前、inode操作、グローバル操作など、ファイル
システムに関する重要なデータを格納する役割を持つ`filesystem_type`という構造体があります。
この構造体は`src/vfs.h`で定義されており、リスト3.3に示します。

<center><strong>リスト 3.3: filesystem_type構造体</strong></center>

```c
struct filesystem_type {
    char *name;
    struct vfs_oprations *ops;
    struct inode_operations *iops;
    struct list_head fs_list;
};
```

登録されたァイルシステムを指し示すリストがあります。このリストを操作するために、
カーネルが内部的に使用する2つのヘルパー関数があり、どちらもリスト3.4に示されています。

<center><strong>リスト 3.4: filesystem_type用のヘルパー関数</strong></center>

```c
int register_fs(struct filesystem_type *fs);
struct filesystem_type *getfs(const char *fs_name);
```

register_fs()は、新しいファイルシステムをカーネル内部のファイルシステムリストに
インストールするために使用されます。getfs()は、fs_nameという名前のファイルシステム
タイプを取得するために使用されます（リスト3.10のmountシステムコールを参照）。

## 3.2 ファイルシステム固有の操作

XV6のVFS実装では、vfs_operationsとinode_operationsという2つの構造体を用いて、
実装されたファイルシステムとカーネルコードの間のインタフェースを提供しています。

### 3.2.1 vfs_operations

この構造体には、ファイルシステム全体に影響を与える操作が格納されており、ほぼすべての
操作の後に、ファイルシステムの状態が変更されます。この構造体は、関数ポインタのリストで
あり、カーネルがファイルシステムの操作を十分な抽象度で使用するための主要な
コンポーネントの1つです。この構造体は，3.1.3節のfilesystem_typeや3.1.1節のinodeなど，
ファイルシステムの一般的な情報を管理する構造体に格納されています。

この構造体に格納されている操作は、リスト3.5に示され、表6に詳細が示されています。

<center><strong>リスト 3.5: vfs_operations構造体</strong></center>

```c
struct vfs_operations {
    int             (*fs_init)(void);
    int             (*mount)(struct inode *devi, struct inode *ip);
    int             (*unmount)(struct inode *);
    struct inode *  (*getroot)(int, int);
    void            (*readsb)(int dev, struct superblock *sb);
    struct inode *  (*ialloc)(uint dev, short type);
    uint            (*balloc)(uint dev);
    void            (*bzero)(int dev, int bno);
    void            (*bfree)(int dev, uint b);
    void            (*brelse)(struct buf *b);
    void            (*bwrite)(struct buf *b);
    struct buf *    (*bread)(uint dev, uint blockno);
    int             (*namecmp)(const char *s, const char *t);
};
```

<center><strong>表 6: vfs_operations変数の説明</strong></center>

| vfs_operationsフィールド | 説明 |
|:------------|:-----|


### 3.2.2 inode_operations

この構造体は、ファイルシステムコードとカーネルが実行するinode操作との間の主要な
接着剤です。この構造体はファイルシステムに依存するinode情報を操作し、成功または
失敗のメッセージを抽象的なカーネルコード（通常はファイルシステム関連のシステム
コール）に返します。inode_operationsには，ファイルの読み書き，フォルダからの
ディレクトリエントリの読み込み，ディレクトリ検索などの操作が格納されます。この
構造体をリスト3.6に示し、その操作の詳細を表7に示します。

<center><strong>リスト 3.6: inode_operations構造体</strong></center>

```c
struct inode_operations {
    struct inode *  (*dirlookup)(struct inode *dp, char *name, uint *off);
    void            (*iupdate)(struct inode *ip);
    void            (*itrunc)(struct inode *ip);
    void            (*cleanup)(struct inode *ip);
    uint            (*bmap)(struct inode *ip, uint bn);
    void            (*ilock)(struct inode *ip);
    void            (*iunlock)(struct inode *ip);
    void            (*stati)(struct inode *ip, struct stat *st);
    int             (*readi)(struct inode *ip, char *dst, uint off, uint n);
    int             (*writei)(struct inode *ip, char *src, uint off, uint n);
    int             (*dirlink)(struct inode *dp, char *name, uint inum, uint type);
    int             (*unlink)(struct inode *dp, uint off);
    int             (*isdirempty)(struct inode *dp);
};
```


<center><strong>表 7: inode_operations変数の説明</strong></center>

| vfs_operationsフィールド | 説明 |
|:------------|:-----|


## 3.3 mountシステムコール

ファイルシステムレイヤは、プロセスがファイルにアクセスするためのインターフェースです。
この抽象化により、アプリケーション開発者は、アプリケーションデータがブロックデバイスに
どのように格納されるかを考える必要も、知る必要もないという強力な機能を実現します。

XV6は、Unix系OSとして、ファイルアクセスをファイルシステムレイヤで実現しており、当初は
1つのブロックデバイスしかサポートしていませんでした。複数のブロックデバイスをアタッチ
できることはVFSの機能ではありませんが、これがないとVFSの使い勝手が悪くなります。
そこで、複数のブロックデバイスをサポートするために、マウントシステムコールのプロト
タイプを実装する必要がありました。

マウントとは、新しいブロックデバイスをファイルシステムレイヤにアタッチする操作です。
これにより、アプリケーション開発者は、あるブロックデバイスから別のブロックデバイスへ
データを操作するための高い抽象度を得ることができます。XV6のマウントシステムコールは
以下のように定義されています。

```c
    int mount(char *special_device_file, char *mount_point_directory, char *filesystem_type_type)
```

ここで、`special_device_file`はマウントされるディスクを指定するファイル、
`mount_point_directory`は新しいファイルシステムがマウントされるディレクトリ、
`filesystem_type`はマウントされるファイルシステムの有効でサポートされているタイプです。
今回の実装とは異なりますが、商用のオペレーティングシステムでは、マウント操作に
オプションを渡すためのインターフェースが提供されています。例えば、読み取り専用や
リカバリなしのフラグなどです。

マウント操作の例として、EXT2ファイルシステムを搭載した/dev/hdcというブロックデバイスが
あり、それを/mntディレクトリにマウントしたいとします。そのためには、次のように
呼び出します。

```c
    mount("/dev/hdc", "/mnt", "ext2")
```

mountコールを実行すると/mntを通して/dev/hdcファイルシステムツリーにアクセスすることが
できます。図3は、マウント操作後のファイルシステムレイヤの様子を示しています。

![fig. 3: ファイルシステムツリー](screens/fig3_fs_tree.png)

<center><strong>図 3: /dev/hdc デバイスを?mntにマウントしたファイルシステムツリー</strong></center>

### 3.3.1 マウントテーブル

オペレーティングシステムがマウント操作をサポートする場合、パスの変換でマウント
ポイントを越えなければならないケースがあるため、データ構造とパス名をinodeに
変換するメソッド（リスト3.13のnamex関数など）の変更が必要になります。

マウントテーブルは、マウントされたファイルシステムに関する情報を格納する役割を
持っています。XV6での実装はBach (1986)の研究に基づいており、マウントテーブルを
2つの主要な構造体で表現しています。各テーブルエントリを表すmntentryとテーブル
自体を表すグローバル構造体mtableです。mntentryとmtableをそれぞれリスト3.7と3.8に
示します。

<center><strong>リスト 3.7: mntentry構造体</strong></center>

```c
// Mount Table Entry
sturct mntentry {
    struct inode *m_inode;
    struct inode *m_rtinode;
    void *pdata;
    int dev;
    int flag;
};
```

<center><strong>表 3.8: mtable構造体</strong></center>

```c
// Mount Table structure
struct {
    struct spinlock lock;
    struct mntentry mpoint[MOUNTSIZE];
} mtable;
```

マウントテーブルの各エントリには、表8に示す情報が格納されています。

<center><strong>表 8: mntentry変数の説明</strong></center>

| mntentryフィールド | 説明 |
|:------------|:-----|
| m_inode | マウントポイントであるinodeへのポインタ（図3のルートファイルシステムの"/mnt") |
| m_rtinode | マウントされたファイルシステムのルートであるinodeへのポインタ（図3の"/dev/hdc"ファイルシステムの"/"） |
| pdata | エントリのプライベートデータへのポインタ（通常はスーパーブロック） |
| dev | ブロックデバイス識別子 |
| flags | カーネルの内部操作用のフラグ |


グローバルマウントテーブル表現では、サイズMOUNTSIZEのマウントエントリの配列と、
同時アクセスを制御するためにカーネルが内部的に使用するスピンロックが格納されます。
このテーブルを操作するために、リスト3.9のようにいくつかのユーティリティー関数が
定義されています。

<center><strong>表 3.9: マウントテーブルのためのヘルパ関数一覧</strong></center>

```c
struct inode *mtablertinode(struct inode *ip);
struct inode *mtablemntinode(struct inode *ip);
int isinoderoot(struct inode *ip);
void mountinit(void);
```

関数`mtablertinode()`は、ipがマウントポイントになっているマウントされたファイル
システムのルートinodeを返します。関数`mtablemntinode()`は、ルートinode ipが
マウントされているマウントポイントのinodeを返します。この2つの関数はリスト3.13に
示されているnamex関数の修正された実装で使用されています。

mountシステムコールは`src/sysfile.c`に実装されています。このシステムコールの主な
アイデアをリスト3.10に示しますが、読みやすさを向上させるためにエラーチェックを
省略しています。

<center><strong>リスト 3.10: mountシステムコールの簡略版</strong></center>

```c
int sys_mount(void) {
    char *dev; char *path; char *fstype;
    struct inode *ip, *evi;
    // Handle syscall arguments
    if (argstr(0, &devf) < 0 ||
        argstr(1, &path) < 0 ||
        argsr(2, &fstype) < 0) {
        return -1;
    }
    // Get inodes
    if ((ip = namei(path)) == 0 ||
        (devi = namei(devf)) == 0) {
        return -1;
    }

    struct filesystem_type *fs_t = getfs(fstype);
    // Open the device and check if everything is ok.
    bdev_open(devi);
    // Add this to a list of filesystem type
    putvfsonlist(devi->major, devi->minor, fs_t);
    // Call specific fs mount operation.
    fs_t->ops->mount(devi, ip);
    // Turn the current ip into a mount point
    ip->type = T_MOUNT;
    return 0;
}
```

2行目から14行目までは、システムコールの引数を解析してローカル変数の設定を行い、
マウント操作を行うために必要な情報を取得しています。16行目では、fstypeがカーネルで
サポートされているかどうかをチェックしています。18行目では、マウントされるデバイスを
オープンし、エラーなくハードウェアにアクセスできるかどうかをチェックしています。
20行目では、デバイスの識別子（メジャー番号とマイナー番号）とファイルシステムタイプを
示す、このマウント操作の登録があります。22行目では、VFSレイヤーを使用して、ファイル
システム固有のマウント操作を呼び出しています。この操作では、デバイスのスーパーブロックを
読み取り、マウントテーブルのマウントエントリを要求します。操作を完了するために、
マウントポイントを表すinodeがT_MOUNTとしてマークされます。

ファイルシステム固有のマウント操作は、ファイルシステムの種類によって変わります。
マウントシステムコールの理解を助けるために、S5と名付けたXV6のデフォルトファイル
システムに対するファイルシステム固有のマウント操作の簡単なバージョンをリスト3.11に
示します。完全な実装は`src/s5.c`にあります（付録B参照）。

<center><strong>リスト 3.11: S5のmount操作ハンドラ</strong></center>

```c
int s5_mount(struct inode *devi, struct inode *ip) {
    struct mntentry *mp;
    s5_ops.readsb(devi->minor, &sb[devi->minor]);
    struct inode *devrtip = s5_ops.getroot(devi->major, devi->minor);
    for (mp = &mtable.mpoint[0]; mp < &mtable.moint[MOOUTSIZE]; mp++) {
        // This slot is avilable
        if (mp->flag == 0) {
fount_slot:
            mp->dev = devi->minor;
            mp->m_inode = ip;
            mp->pdata = &sb[devi->minor];
            mp->flag |= M_USED;
            mp->m_rtinode = devrtip;
            initlog(devi->minor);
            return 0;
        } else {
            // The disk is already mounted
            if (mp->dev == devi->minor) {
                return -1;
            }
            if (ip->dev == mp->m_inode->dev &&
                ip->inum == mp->m_inode->inum)
                goto found_slot;
        }
    }
    return -1;
}
```

s5_mountで行われるほとんどすべての操作は、他のファイルシステムと共有することが
できます。3行目では、src/s5.cでグローバル化されているs5_ops構造体を介してスーパー
ブロックが読み込まれます。4行目では、マウントされるデバイスのルートinodeを読み込み
ます。5行目から25行目までのループでは、マウントテーブル上の空のエントリを検索し、
見つかった場合にはmntentryを設定します。14行目は、このファイルシステムのログ
システムを初期化します。また、この実装では、18行目に見られるように、1つの
デバイスを2回マウントすることはできません。最後に、21行目では、マウントポイントが
すでにエントリになっているかどうかをチェックし、マウントされる新しいデバイスを
指すように更新します。

3.3節で説明したケースを例示するために、図4にInode TableとMount Tableの関係を示します。
XV6では、リスト3.9に示したmtablemntinode()関数とmtablertinode()関数のサポートにより、
この図を実装しています。

![fig 4](screens/xv6_vfs_fig4.png)

<center><strong>図 4: `inode table`と`mount table`の関係（Bach (1986)による）</strong></center>

### XV6の変更

このマウント操作を実現するためには、XV6のコードを変更する必要がありました。最初の
変更点は、IDEドライバのコードです。このコードは2つのIDEディスクをサポートするように
ハードコーディングされており、2つのスロットはすでにブートディスクとXV6のルート
ファイルシステムとして使用されていたからです。スレーブバスを使用するようにドライバを
変更し(TECHNOLOGY, 1993)、XV6に4台のIDEデバイスを取り付けることができるように
しました。また、`namex`関数と`iget`関数を変更して、マウントポイントが交差する
パス変換に対応しました。

更新された`iget`関数は、要求された`inode`がマウントポイントであるか（すなわち、
そのタイプがT_MOUNTであるか）チェックします。それが真であれば、この`inode`の
マウントテーブルエントリを見つけ、`mtablertinode()`を使ってマウントされたファイル
システムのルート`inode`を取得し、それを要求された`inode`として返します。この
アルゴリズムは、マウントポイントを横断するパス変換が、マウントポイントから
マウントされたファイルシステムへ正しくたどることを保証します。リスト3.12は
変更されたの`iget()`関数を示しています。

<center><strong>リスト 3.12: iget() - 交差するマウントポイントをもつパス変換をサポートした変換後の関数</strong></center>

```c
struct inode *iget(uint dev, uint inum,
    int (*fill_inode)(struct inode *)) {
    struct inode *ip, *empty;
    struct filesystem_type *fs_t;
    acquire(&icache.lock);
    empty = 0;

    // inodeはキャッシュされているか？
    for (ip = &icache.inode[0]; ip < &icache.inode[NINODE]; ip++) {
        if (ip->ref > 0 && uo->dev == dev && ip->inum == inum) {
            // カレントinodeはマウントポイントか?
            if (ip->type == T_MOUNT) {
                struct inode *rinode = mtablertinode(ip);
                if (inode == 0) {
                    panic("Invalid inode on mount table");
                }
                rinode->ref++;
                release(&icache.lock);
                return rinode;
            }
            ip->ref++;;
            release(&icache.lock);
            return ip;
        }
        if (empty == 0 && ip->ref == 0)     // 空きスロットを記憶
            empty = ip;
    }

    // inodeキャッシュエントリをリサイクル
    if (empty == 0)
        panic("iget: n inodes");

    fs_t = getvfsentry(IDEMAJOR, dev)->fs_t;
    ip = empty;
    ip->dev = dev;
    ip->inum = inum;
    ip->ref = 1;
    ip->flags = 0;
    ip->fs_t = fs_t;
    ip->iops ~ fs_t->iops;
    release(&icache.lock);
    if (!fillinode(ip)) {
        panic("Error on fill inode");
    }
    return ip;
}
```

しかし、この`iget()`の実装では、逆方向（マウントされたデバイスからマウントポイントの
方向）のパス変換は処理できません。これは、ファイルシステムツリーをさかのぼったときに
起こります（つまり、パスに「...」がある場合）。この問題に対処するためには、リスト
3.13のように`namex`関数を変更する必要があります。

<center><strong>リスト 3.13: namex() - 交差するマウントポイントをもつパス変換をサポートした変換後の関数</strong></center>

```c
static struct inode *
namex(char *path, int nameiparent, char *name) {
    struct inode *ip, *next, *ir;
    if (*path == '/')
        ip = rootfs->fs_t->ops->getroot(IDEMAJOR< ROOTDEV);
    else
        ip = idup(proc->cwd);

    while ((psth = skipelem(path, name)) != 0) {
        ip->inode->ilock(ip);
        if (ip->type != T_DIR) {
            iunlockput(ip);
            return 0;
        }
        if (nameiparent && *path == '\0') {
            // 1レベル前に停止
            ip->iops->iunlock(ip);
            return ip;
        }
component_search:
        if ((next = ip->iosp->dirlookup(ip, name, 0)) == 0) {
            iunlockput(ip);
            return 0;
        }
        ir = next->fs_t->ops->getroot(IDEMAJO, next->dev);
        if (next->inum == ir->inum && isinoderoot(ip) &&
            (strncmp(name, "..", 2) == 0)) {
            struct inode *mntinode = mtablemntinode(ip);
            iunlockput(ip);
            ip = mntinde;
            ip->iops->ilock(ip);
            ip-ref++;
            goto cmponet_search;
        }
        iunlockput(ip);
        ip = next;
    }
    if (nameiprent) {
        iput(ip);
        return 0;
    }
    return ip;
}
```

`namex`関数でマウントポイントの横断をサポートするための修正は、20行目から35行目に
かけて行われました。この関数は、到達したinodeがルートinodeであるかどうか、また、
直前のパスコンポーネントが".."であるかをチェックします。これが真であれば、それは
マウントされたファイルシステムのルートinodeであり、そこから".."ディレクトリ
エントリを探すために、マウントテーブルからマウントポイントinodeを見つけなければ
なりません（29行目）。これが、マウントポイントがフォルダであることを要求する主な
理由です。すべてのフォルダには少なくとも"."と".."のディレクトリエントリが含まれて
いるからです。この変更により、マウントされたファイルシステムからマウントポイントを
横切るパス変換が正しく実行されるようになります。

### ブロックデバイスファイルシステムのマッピング

カーネル操作の中には、操作しているブロックデバイスのファイルシステムタイプを認識
しなければならないものがあります（リスト3.12を参照。inodeを正しく設定するために
この情報が必要です）。XV6 VFSでは、マウントテーブルからこの情報を探す無駄を省く
ために、vfsのリストを格納する`vfsmlist`というオブジェクトを用意しています。
この構造体は、`src/vfs.h`で定義されており、リスト3.14に示します。

<center><strong>リスト 3.14: struct vfs</strong></center>

```c
struct vfs {
    int major;
    int minor;
    int flag;
    struct filesystem_type *fs_t;
    struct list_head fs_next;       // vfsで次にマウントされているfs
};
```

この構造体により、メジャーとマイナーの識別子を通じて、デバイスをファイル
システムタイプにマッピングすることができます。このマッピングはハッシュを使って
実装すべきであるということが重要ですが、この作業ではパフォーマンスは主な関心事
ではないので、代わりにリンクリストを使用しました。

<center><strong>リスト 3.15: vfs用のヘルパ関数一覧</strong></center>

```c
struct vfs *getvfsentry(int major, int minor);
int putvfsonlist(int major, int minor, struct filesystem_type *fs_t);
```

XV6 VFSでは、このリストの使用を抽象化するために、デバイスのvfsリファレンスを
取得するための`getvfsentry()`とデバイスとそのファイルシステムタイプを
リンクするための`putvfsonlist()`という2つのヘルパ関数を用意しました。

# 4. XV6上で新しいファイルシステムを実装する

本章では、XV6 VFSを用いて新しいファイルシステムを実装するために必要な手順を
説明します。この手順を説明するために、基本的なバージョンのEXT2 (CARD; TS'O;
TWEEDIE, 2010)を使用します。本章で紹介するすべてのリストは、`src/ext2.c`と
`src/ext2.h`に記載されています（付録C参照）。実装について説明する前に、EXT2の
コンセプトの概要を説明します。EXT2の設計と実装についての完全なドキュメントは
公式ドキュメント（POIRIER, 2011）をお読みください。

## 4.1 EXT2 ファイルシステム

EXT2は、Rémy Card、Theodore Ts'o、Stephen Tweedieの3人が、Extended Filesystemの
代替として実装したブロックベースのファイルシステムで、古い内部構造を維持しながらも
新しい機能を提供しています。1993年1月にLinuxカーネルの一部としてリリースされ、
その後、さまざまな主要Linuxディストリビューションで標準ファイルシステムとして
使用されました。EXT4やEXT3の構造は、EXT2の内部構造の影響を強く受けています。
最近のファイルシステムとは異なり、EXT2はジャーナリングやジャーナルチェックサム、
エクステントなどの最適化機能はサポートしていません。しかし、そのシンプルさゆえに、
ファイルシステムの開発者にとっては良いスタート地点となっています。

### 4.1.1 EXT2のディスク構成

EXT2のディスク構成は、BSDファイルシステム(MCKUSICK et al., 1984)のレイアウトを
強く意識しています。EXT2は、従来のファイルシステムとは異なり、物理的にブロック
単位で分割されており、ディレクトリやファイルなどの関連するデータを物理的に近くに
配置することができるため、シーケンシャルアクセスを向上させることができます。
EXT2ファイルシステムの物理構造を図5に示します。

![fig 5](screens/xv6_vfs_fig5.png)

<center><strong>図 5: EXT2 ファイルシステムアーキテクチャ</strong></center>

最初の1024バイトにはブートセクタがあります。その後、N個のブロックグループが続き、
各ブロックグループは1～8KBのブロックに分かれています。ブロックグループ、inode、
ブロックの数は、パーティションサイズとブロックサイズに応じて異なります。これらの
パラメータは、`mkfs.ext2`ユーティリティーを使ってファイルシステムをデバイスに
インストールする際に設定できます。

最初のブロックグループには、スーパーブロックやファイルシステムディスクリプタ
（ブロックグループディスクリプタテーブルなど）などの重要なファイルシステム制御
情報のコピーのほか、図6に示すように、ブロックビットマップ、inodeビットマップ、
inodeテーブルの一部、データブロックなど、ファイルシステム自体の一部が含まれて
います。

![fig 6](screens/xv6_vfs_fig6.png)

<center><strong>図 6: 最初のEXT2ブロックグループのレイアウト</strong></center>

表9は、20MBのEXT2ファイルシステム（ブロックサイズ1KB、ブロックグループサイズ8MB）の
レイアウトです。ご覧のとおり、スーパーブロックとファイルシステムディスクリプタの
バックアップがあります。これらのバックアップは、破損した場合に元のデータを復元
することを可能にするため、信頼性が向上します。

<center><strong>表 9: 1 KiBのブロックサイズを使用した20MBのサンプルExt２ファイルシステム（Pointer(2011)による）</strong></center>

| ブロックオフセット | 長さ | 説明 |
|:-------------------|:-----|:-----|
| バイト 0 | 512 バイト | bootレコード（あれば）  |
| バイト 512 | 512 バイト | 追加のbootレコード（あれば） |
| -- ブロックグループ 1 | ブロック 1 - 8192 --  |
| バイト 1024 | 1024 バイト | スーパーブロック |
| ブロック 2 | 1 ブロック | ファイルシステムディスクリプタテーブル  |
| ブロック 3 | 1 ブロック | ブロックビットマップ  |
| ブロック 4 | 1 ブロック | inodeビットマップ  |
| ブロック 5 | 214 ブロック | inodeテーブル  |
| ブロック 219 | 7974 ブロック | データブロック  |
| -- ブロックグループ 2  | ブロック 8193 - 16384 --
| ブロック 8193 | 1 ブロック | スーパーブロック バックアップ |
| ブロック 8194 | 1 ブロック | ファイルシステムディスクリプタテーブル バックアップ  |
| ブロック 8195 | 1 ブロック | ブロックビットマップ |
| ブロック 8196 | 1 ブロック | inodeビットマップ  |
| ブロック 8197 | 214 ブロック | inodeテーブル  |
| ブロック 8408 | 7974 ブロック | データブロック  |
| -- ブロックグループ 3 | ブロック 16385 - 24576 --  |
| ブロック 16385 | 1 ブロック | ブロックビットマップ |
| ブロック 16386 | 1 ブロック | inodeビットマップ  |
| ブロック 16387 | 214 ブロック | inodeテーブル  |
| ブロック 16601 | 3879 ブロック | データブロック  |

ディスクのレイアウトは、ブロックサイズ、グループごとのブロック数、グループごとの
inodeがわかっていれば予測可能です。これらの情報はスーパーブロック構造体にあり、
EXT2の実装ではこれらの値を使ってinodeテーブル上のinodeエントリの正しいオフセットを
計算したり、特定のデータブロックを探したりしています。

### EXT2の重要な構造体

すべてのファイルシステムは、ある一定の抽象度でデータを操作できるようにするために、
内部の特定の構造を表現する必要があります。これらの構造体は、ブロックデバイスから
生データにアクセスするために使用され、誤った操作を行うとファイルシステムを破壊して
しまう可能性があるため、文書化されたレイアウトに厳密に従わなければなりません。

EXT2の主な構造体には、EXT2のスーパーブロックを操作する`ext2_superblock`、EXT2の
inodeを操作する`ext2_inode`、EXT2のディレクトリエントリを操作する`ext2_dir_entry_2`、
EXT2のブロックグループを操作する`ext2_block_group_desc`があります。

#### `struct ext2_superblock`

構造体`ext2_superblock`は、EXT2の物理的なスーパーブロックを管理するために
使用されます。保存されている情報をリスト4.1に示します。その完全なドキュメントは
EXT2 documentation (POIRIER, 2011)にあります。

<center><strong>リスト 4.1: struct ext2_superblock</strong></center>

```c
struct ext2_superblock {
    uint32 sinodes_count;               /* inode数 */
    uint32 s_blocks_count;              /* ブロック数 */
    uint32 s_r_blocks_count;            /* 予約ブロック数 */
    uint32 s_free_blocks_count;         /* 利用可能なブロック数 */
    uint32 s_free_inodes_count;         /* 利用可能なinode数 */
    uint32 s_first_data_block;          /* 最初のデータブロック */
    uint32 s_log_block_size;            /* ブロックサイズ */
    uint32 s_log_frag_size;             /* フラグメントサイズ */
    uint32 s_blocks_per_group;          /* グループあたりのブロック数 */
    uint32 s_frags_per_group;           /* グループあたりのフラグメント数 */
    uint32 s_inodes_per_group;          /* グループあたりのinode数 */
    uint32 s_mtime;                     /* マウント時間 */
    uint32 s_wtime;                     /* 書き込み時間 */
    uint16 s_mnt_count;                 /* マウント数 */
    uint16 s_max_mnt_count;             /* 最大マウント数 */
    uint16 s_magic;                     /* マジック識別子 */
    uint16 s_state;                     /* ファイルシステムの状態 */
    uint16 s_errors;                    /* エラー検知の場合の措置 */
    uint16 s_minor_rev_level;           /* マイナーリビジョンレベル */
    uint32 s_lastcheck;                 /* 最後にチェックした時間 */
    uint32 s_checkinterval;             /* チェック間隔の最大時間 */
    uint32 s_creator_os;                /* OS */
    uint32 s_rev_level;                 /* リビジョンレベル */
    uint16 s_def_resuid;                /* 予約ブロックのデフォルトuid */
    uint16 s_def_resgid;                /* 予約ブロックのデフォルトgid */
    uint32 s_first_ino;                 /* 予約済みでない最初のinode */
    uint16 s_inode_size;                /* inode構造体のサイズ */
    uint16 s_block_group_nr;            /* このスーパーブロックのブロックグループ番号 */
    uint32 s_feature_compat;            /* 互換機能セット */
    uint32 s_feature_incompat;          /* 非互換機能セット */
    uint32 s_feature_ro_compat;         /* 読み込みのみの互換機能セット */
    uint8  s_uuid[16];                  /* ボリュームの128ビットuuid */
    char   s_volume_name[16];           /* ボリューム名 */
    char   s_last_mounted[64];          /* 最後にマウントされたディレクトリ */
    uint32 s_algorithm_usage_bitmap;    /* 圧縮用 */
    uint8  s_prealloc_blocks;           /* 事前割当を試みるブロック番号 */
    uint8  s_prealloc_dir_blocks;       /* ディクト利用に事前割当する番号 */
    uint16 s_padding1;
    uint8  s_journal_uuid[16];          /* ジャーナルスーバープロックのuuid */
    uint32 s_journal_inum;              /* ジャーナルファイルのinode番号 */
    uint32 s_journal_dev;               /* ジャーナルファイルのデバイス番号 */
    uint32 s_last_orphan;               /* 削除用inodeリストの先頭 */
    uint32 s_hashseed[4];               /* HTREEハッシュシード */
    uint8  s_def_haash_version;         /* 使用するデフォルトハッシュバージョン */
    uint8  s_reserved_char_pad;
    uint16 s_reserved_word_pad;
    uint32 s_default_mount_opts;
    uint32 s_first_meta_bg;             /* 最初のメタブロックブロックループ */
    uint32 s_reserved[190];             /* ブロック終わりのパディング */
};
```

#### 4.1.2.2 `struct ext2_inode`

ext2_inode構造体は、ファイルシステムに格納されているすべてのディレクトリ、通常
ファイル、シンボリックリンク、特殊ファイルを追跡します。ファイルの位置、サイズ、
タイプ、アクセス権などが保存されます。ファイル名はディレクトリエントリに含まれて
いるため、inode自体には保存されません。リスト4.2にこの構造体を示します。

<center><strong>リスト 4.2: struct ext2_inode</strong></center>

```c
struct ext2_inde {
    uint16 i_mode;                      /* ファイルモード */
    uint16 i_uid;                       /* ユーザuidの下16ビット */
    uint32 i_size;                      /* バイト単位のサイズ */
    uint32 i_atime;                     /* アクセス時間 */
    uint32 i_ctime;                     /* 作成時間 */
    uint32 t_mtime;                     /* 変更時間 */
    uint32 i_dtime;                     /* 削除時間 */
    uint16 i_gid;                       /* グループIDの下16ビット */
    uint16 i_links_count;               /* リンク数 */
    uint32 i_blocks;                    /* ブロック数 */
    uint32 i_flags;                     /* ファイルフラグ */
    union {
        struct {
            uint32 i_i_reserved1;
        } linux1;
        struct {
            uint32 h_i_translator;
        } hurd1;
        struct {
            uint32 m_i_reserved1;
        } masix1;
    } osd1;                             /* OS依存 1 */
    uint32 i_block[EXT2_N_BLOCKS];      /* ブロックへのポインタ */
    uint32 i_generation;                /* ファイルバージョン（NFS用） */
    uint32 i_file_acl;                  /* ファイルACL */
    uint32 i_dir_acl;                   /* ディレクトリACL */
    uint32 i_faddr;                     /* フラグメントアドレス */
    union {
        struct {
            uint8  l_i_frag;            /* フラグメント番号 */
            uint8  l_i_fsize;           /* フラグメントサイズ */
            uint16 l_pad1;
            uint16 i_i_uid_high;        /* 次のフィールドとともに */
            uint16 l_i_gid_high;        /* reserved[0]
            uint32 l_i_reserved2;
        } linux2;
        struct {
            uint8  h_i_frag;            /* フラグメント番号 */
            uint8  h_i_fsize;           /* フラグメントサイズ */
            uint16 h_i_mode_high;
            uint16 h_i_uid_high;
            uint16 h_i_gid_high;
            uint32 h_i_author;
        } hurd2;
        struct {
            uint8  m_i_frag;            /* フラグメントサイズ */
            uint8  m_i_fsize;           /* フラグメントサイズ */
            uint16 m_pad1;
            uint32 m_i_reserved[2];
        } sasix2;
    } osd2;                             /* OS依存 2 */
};
```

ここで重要なのは、たとえEXT2の実装がこの構造体の一部のメンバーを使用して
いなくても、ファイルシステムのメタデータを壊さないようにするために、これらの
メンバーを残しておく必要があるということです。

#### 4.1.2.3 `struct ext2_dir_entry_2`

EXT2のディレクトリエントリはリンクリストに格納されており、各エントリには
inode番号、エントリの全長、名前の長さ、ファイルタイプ、ファイル名が含まれて
います。リスト4.3にこの構造鯛を示します。

<center><strong>リスト 4.3: struct ext2_dir_entry_2</strong></center>

```c
struct ext2_dir_entry_2 {
    uint32 inode;           /* inode番号 */
    uint16 rec_len;         /* ディレクトリエントリ長 */
    uint8  name_len;        /* 名前の長さ */
    uint8  file_type;
    char   name[];          /* ファイル名、最大EXT2_NAME_LEN */
};
```

#### 4.1.2.4 `struct ext2_block_group_decs`

この構造体は、ブロックグループのディスクリプタを格納します。各ブロックグループを管理する
ために、inodeビットマップ、inodeテーブル、ブロックビットマップ、空きブロック数
などの位置情報を提供します。そのインスタンスはスーパーブロックの直後に格納されて
いる配列「ブロックグループディスクリプタテーブル」に格納されています（図6、表9参照）。
リスト4.4にこの構造鯛を示します。

<center><strong>リスト 4.4: struct ext2_block_group_desc</strong></center>

```c
struct ext2_block_group_desc {
    uint32 bg_block_bitmap;             /* ブロックビットマップブロック */
    uint32 bg_inode_bitmap;             /* inodeビットマップブロック */
    uint32 bg_inode_table;              /* inodeテーブルブロック */
    uint16 bg_free_blocks_count;        /* 空きブロック数 */
    uint16 bg_free_inodes_count;        /* 空きinode数 */
    uint16 bg_used_dirs_count;          /* ディレクトリ数 */
    uint16 bg_pad;
    uint32 bg_reserved[3];
};
```

## 4.2 新規ファイルシステムの実装戦略

XV6 VFSでは新規ファイルシステムをカーネルに追加する作業は簡単です。しかし、
この簡単さが、内部のファイルシステムの実装にまで及んでいません。すべての
ファイルシステムの機能を実装してから、検証フェーズに入るのは賢明ではありません。
したがって、ファイルシステムに依存するすべての操作をpanic関数を呼び出す
空の操作として作成するのが良い戦略です。これを行ってから新しいファイル
システムの設定と登録を行い、各システムコール用に1つずつ操作のコーディングを
開始します。

この方法では、コードを部分的にテスト、デバッグすることができ、コードの粒度の
おかげで、より正確にエラーを見つけることができます。

EXT2の実装は本稿の目的ではないため、詳細はLinuxカーネルの実装（POIRIER, 2011）を
参照するか、`src/ext2.c`ファイルを確認してください。

## 4.3 EXT2のvfs_operations

3.2.1節で述べたように、ファイルシステムに依存する汎用操作を指し示す
オブジェクトを作成する必要があります。リスト4.5は、例としてEXT2用の
vfs_operations構造体を示しています。

<center><strong>リスト 4.5: EXT2用のvfs_operationsインスタンス</strong></center>

```c
struct vfs_operations ext2_ops = {
    .fs_init = &ext2fs_init,
    .mount   = &ext2_mount,
    .unmount = &ext2_unmount,
    .getroot = &ext2_getroot,
    .readsb  = &ext2_readsb,
    .ialloc  = &ext2_iallock,
    .balloc  = &ext2_ballock,
    .bzero   = %ext2_bzero,
    .bfree   = %ext2_bfree,
    .brelse  = &ext2_brelse,
    .bwrite  = &ext2_bwrite,
    .bread   = &ext2_bread,
    .namecmp = &ext2_namecmp
};
```

## 4.4 EXT2のinode_operations

3.2.2項で述べたように、ファイルシステムに依存するinode操作を指す
inode_operationsのインスタンスを作成することも必要です。リスト4.6に
EXT2用のinode_operationsの構造体を示します。

<center><strong>リスト 4.6: EXT2用のinode_operationsインスタンス</strong></center>

```c
struct inode_operations ext2_iops = {
    .dirlookup   = &ext2_dirlookup,
    .iupdate     = &ext2_iupdate,
    .itrunc      = &ext2_itrunc,
    .cleanup     = &ext2_cleanup,
    .bmap        = &ext2_bmap,
    .ilock       = &ext2_ilock,
    .iunlock     = &ext2_iunlock,
    .stati       = &ext2_stati,
    .readi       = &ext2_readi,
    .writei      = &ext2_writei,
    .dirlink     = &ext2_dirlink,
    .unlink      = &ext2_unlink,
    .isdireempty = &ext2_isdirempty
};
```

また，inode_operations構造体では`iunlock`, `stati`, `readi`などの汎用的な操作も
定義されていることがわかります。ファイルシステム固有の機能についても同様の命名
規則が用いられています。

## 4.5 filesystem_type構造体の設定と登録

XV6 VFSで新しいファイルシステムをサポートするための最も重要なステップの一つは、
構造体filesystem_typeを作成し、その変数に値を入力することです。ファイルシステムに
特化したコードをグローバルに保持する必要はないため、`src/ext2.c`に実装されています。
リスト4.7にあるように、基本的にはセクション4.3と4.4の構造体をポイントし、
ファイルシステム名を格納します。

<center><strong>リスト 4.7: EXT2用のfilesystem_typeインスタンス</strong></center>

```c
struct filesystem_type ext2fs = {
    .name = "ext2",
    .ops  = %ext2_ops,
    .iops = &ext2_iops
};
```

次に，サポートすべき新しいファイルシステムがあることをカーネルに通知する必要が
あります。これを実現するためには3.1.3節で紹介した関数`register_fs()`を使用します。
私たちのEXT2の実装では、リスト4.8に示すように、ファイルシステムの登録を含むすべての
内部データを初期化するために`initext2fs(void)`という関数を用意しています。

<center><strong>リスト 4.8: initext2fs() - EXT2初期化関数</strong></center>

```c
int initext2fs(void) {
    initlock(&ext2_sb_pool.lock, "ext2_sb_pool");
    return register_fs(&ext2fs);
}
```

Linuxカーネルとは異なり、XV6ではモジュールを動的にロードすることができない
ため、関数`initext2fs()`をカーネルの初期化コードにハードコーディングする必要が
ありました。また、ファイルシステムの初期化を整理するために、リスト4.9のように
`initfss()`という関数を作成しました。

<center><strong>リスト 4.9: ファイルシステムを初期化するためのカーネル関数</strong></center>

```c
static void initfss(void) {
    if (inits5fs() != 0)    // init s5 fs
        panic("S5 not registered");
    if (initext2fs() != 0)  // init ext2 fs
        panic("ext2 not registered");
}
```

以上の手順で、3.3節で紹介したマウントシステムコールが、EXT2ファイルシステムで
フォーマットされたデバイスをサポートできるようになります。

## 4.6 最後に

今回のEXT2の実装は、Linuxのバージョンをベースにしています。しかし、XV6の
カーネル内部データ操作のために多くの変更が必要となりました。第一の変更点は、
Linuxに存在するバイトエンディアンの互換性を取り除くことでした。XV6はX86
アーキテクチャで動作するように設計されているので、バイトエンディアンを変換
する命令をすべて削除しました。また、XV6では、ファイルシステムのブロック管理に
`buffer_head`構造体を使用しており、ページキャッシュを使用するLinuxの実装とは
異なります。4.3節、4.4節で紹介した構造体や関数は、目的とするファイルシステムの
実装に合わせて実装するできることを知っておくことが重要です。

# XV6 VFSの評価

## 5.1 方法

本章では、XV6におけるVFSの操作性を評価します。EXT2を実装することで，このファイル
システムをサポートしている他のOSとの相互運用性を確認することができます。評価に
使用した環境は、Debian 3.2.65-1とLinuxカーネル3.2.0-4-amd64を搭載した仮想マシンで、
XV6のコードのコンパイルと、mkfs.ext2によるEXT2ファイルシステムの作成に使用した
ものです。XV6はqemu-system-i386でエミュレートされたi386マシンで動作していて
います (実行環境の設定は Appendix A を参照)。

本評価の主な目的は、XV6のVFSが正常に動作し、EXT2のブロックデバイスが完全に動作する
ことを示すことです。そのために、XV6上でディレクトリの作成と読み込み、ディレクトリ
エントリの読み込み、ファイルの書き出しと削除などの操作を行い、その後、Linux上で
ファイルシステムをマウントして、それらの操作が正しく行われたかどうかを確認する
ことにしました。続いて、逆方向を評価します。Linux上でファイルシステムを変更した後、
XV6上でマウントし、変更したファイルシステムに正常にアクセスできるかどうかを確認
しました。LinuxのEXT2実装は、EXT3やEXT4などのファイルシステムのベースとして広く
利用されている安定した商用ファイルシステムであることから、XV6のVFS実装を利用して
その実装を行うことで、本アーキテクチャ設計の妥当性を確認することができます。

## 5.2 実験と結果

XV6のターミナルはスクリプトによる自動化をサポートしていないため，実験は手で行う
必要があります。評価の第一弾として，リスト5.1に示すコマンドを実行し，得られた
結果を図7に示しました。

<center><strong>リスト 5.1: XV6上でEXT2ファイルシステムを変更するコマンドのリスト</strong></center>

```bash
$ mkdir /mnt
$ mount /dev/hdc /mnt ext2
$ mkdir mnt/dir0
$ echo Lorem ipsum > mnt/file0
$ cat mnt/file0
```

![fig. 7: XV6上でEXT2ファイルシステムを変更するコマンドの実行](screens/xv6_vfs_fig7.png)

<center><strong>図 7: XV6上でEXT2ファイルシステムを変更するコマンドの実行</strong></center>

これらのコマンドは簡単そうに見えますが、XV6では多くの処理が行われています。
最初の2つのコマンドは、EXT2ファイルシステムをあるディレクトリにマウントする
ために必要なものです。mountシステムコールを実行するmountプログラムは、マウント
するデバイス、デバイスがマウントされるディレクトリ、このデバイスに含まれる
ファイルシステムのタイプという3つのパラメータを必要とします。その後、この
デバイス上にdir0というディレクトリを作成します。最後に、ファイルの書き込みが
正しく行われるかを確認するために、4行目でfile0に「Lorem ipsum」を書き込み、
5行目でその内容を読み込んで書き込み操作が成功したかどうかを確認しています。

これらの変更がデバイス内で正しく行われていることを確認するために、Linuxで
このファイルシステムをマウントし、リスト5.2と図8に示すように、別の一連の
コマンドを実行しました。

<center><strong>リスト 5.2: Linux上でEXT2ファイルシステムを変更するコマンドのリスト</strong></center>

```bash
$ sudo losetup -f src/ext2.img      // オリジナル: sudo losetup /dev/loop0 src/ext2.img
$ sudo losetup -l                   // -f で空きloopNを使用。-l で使用中のloopNを表示
$ sudo mount /dev/loop0 mnt/
$ sudo cat mnt/file0
$ sudo cp -R /usr/include mnt/
$ sudo cat mnt/include/termio.h
$ sudo umount mnt
$ sudo losetup -d /dev/loopN]       // -l で調べたloopNを指定
```

1行目は、Linuxカーネルのツールを使って、src/ext2.imgにあるファイルシステム
イメージをループデバイスにコピーし、このイメージを仮想ブロックデバイスとして
マウントできるようにしています。2行目は、ファイルシステムイメージを mnt/に
マウントします。3行目は、file0の内容をターミナルに表示します。4行目は、
/usr/include/にある複雑なディレクトリ構造をmnt/にコピーします。5行目と6行目で
EXT2イメージの操作が終了します。

![fig. 8: Linux上でEXT2ファイルシステムを変更するコマンドの実行](screens/xv6_vfs_fig7.png)

<center><strong>図 8: Linux上でEXT2ファイルシステムを変更するコマンドの実行</strong></center>

最後に、Linuxで行った操作が、XV6で正しくロードできるかどうかを確認します。
これは、リスト5.3のコマンドで行い、その結果を図9に示します。

<center><strong>リスト 5.2: XV6上でEXT2ファイルシステムの変更を検証するコマンドのリスト</strong></center>

```bash
$ mount /dev/hdc /mnt ext2
$ cat /mnt/include/termio.h
```

![fig. 9: XV6上でEXT2ファイルシステムの変更を検証するコマンドの実行](screens/xv6_vfs_fig7.png)

<center><strong>図 9: XV6上でEXT2ファイルシステムの変更を検証するコマンドの実行</strong></center>

2行目以降は、Linuxでも同じように出力されているので（図8参照）、
"mnt/include/termio.h "というファイルの内容がターミナルに正しく出力されている
ことが確認できます。なお，catコマンドはシステムコールreadを使用していますが、
これはVFSの2つの主要な操作であるreadi()とdirlookup()を用いて実装されています
（3.2.2項参照。

以上の実験により、XV6上で動作するEXT2ファイルシステムの主要な機能を検証する
ことができました。ファイルシステムの操作には、XV6のシステムコールを用いて実装
された、XV6のネイティブコマンドがそのまま使用されています。これは、XV6のVFS実装が
システムコールのインターフェイスを変更せず、内部機能のみを変更することで、
VFSの実装に求められる抽象化を実現しているからです。また、今回の実験では、XV6の
動作を損なうことなく、新しいファイルシステムを追加することが可能であることを
示しました。

# 6 結論

この作業により、VFSの大きな利点の一つである、オペレーティングシステム間の
ファイルシステムの相互運用性が明らかになりました。複数のファイルシステムとの
互換性は、現代のオペレーティング・システムにおいて非常に重要な機能であり、
VFSの設計と実装は、オペレーティングシステムエンジニアに教えるべきテーマで
あると直感しました。

本研究の主な貢献は、学術的な目的で設計されたオペレーティングシステムに
シンプルでありながら強力なVFSレイヤーを実装して文書化したことであり、これは
オペレーティングシステム開発者のスタート地点として最適です。私たちのEXT2の
実装は、VFSの設計が望ましい抽象化を達成したことを示しています。もう一つの重要な
貢献は、XV6のファイルシステムをVFS層で動作するように移植したことであり、
ルートファイルシステムで実行される全ての操作はVFS層を使用しているため、これも
検証の一部となっています。さらに、我々のXV6 VFSでは、複数のタイプのファイル
システムを同時に使用できることを示しています。

## 6.1 制限事項と今後の課題

今後の課題としては、本稿で紹介したVFSアーキテクチャの実力を検証するために、
非Unix系のファイルシステムの実装が興味深いです。同じことが、procfs、sysfs、
NFSなどのディスクレスファイルシステムにも当てはまります。

さらに、このXV6のVFS実装には抽象度を向上させることができるいくつかの制限が
あります。第1に、改善可能な点は、VFSをエクステントベースのファイルシステムと
互換性を持たせることです。今回の実装では、すべてのファイルシステムがブロック
ベースであることを前提としていますが、これはBTRFS、EXT4、ZFSなどの最新の
ファイルシステムでは当てはまりません。

第2に、ディレクトリエントリを読むための新しいシステムコールを実装する必要が
あります。このシステムコールがないと、lsなどのプログラムをエレガントに実装する
ことができません。

第3に、ブロックI/Oサブシステムのメモリ使用量を改善する必要があります。現在、
ブロックが小さくてもバッファキャッシュで1ブロックあたり4KBを消費しています。

最後に、VFSレイヤーのAccess Control Listパターンをサポートする操作を作ることが
重要です。今回のバージョンでは、XV6にはrootユーザーしかいないため、実装されて
いません。
