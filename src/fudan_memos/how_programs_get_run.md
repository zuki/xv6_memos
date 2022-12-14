# 1. [How programs get run](https://lwn.net/Articles/630727/)

この記事は、カーネルがどのようにプログラムを実行するのか、すなわち、ユーザプログラムが
execve()システムコールを呼び出した時にその裏側で何が起こっているのかを解説する2本
立ての記事の1本目です。私は最近新しいシステムコール`execveat()`を実装しました。
それはexecve()のバリエーションであり、他の`*at()`システムコールと同様に、呼び出し元が
ファイル記述子とパスの組み合わせで実行プログラムを指定できるようにしたものです。
（これにより、Capsicumのようなサンドボックス環境で重要な、`/proc`ファイルシステムへの
アクセスに依存しない`fexecve()`ライブラリ関数の実装が可能になります)。

その過程で、既存のexecve()の実装を調査しました。2本の記事はその機能の詳細を紹介
するものです。今回は、さまざまなプログラム形式を可能にする、カーネルがプログラム呼び
出しに使用する一般的なメカニズムに焦点を当てます。2番目の記事では、ELFバイナリの
実行の詳細に焦点を当てます。

## ユーザ空間から眺める

 カーネルに潜る前に、まずユーザ空間でのプログラム実行の挙動を探ることにします
 （この挙動については、"The Linux Programming Interface”の第27章にも良い記述が
 あります）。バージョン3.18までのLinuxでは、新しいプログラムを呼び出すシステム
 コールは`execve()`だけです。そのプロトタイプは以下のようになっています。

```c
    int execve(const char *filename, char *const argv[], char *const envp[]);
```

引数`filename`は実行するプログラムを指定するもので、引数`argv`と`envp`は新プログラムの
コマンドライン引数と環境変数を指定するNULL終端のリストです。簡単なスケルトン
ドライバプログラム（[`do_execve.c`](https://lwn.net/Articles/630754/#do_execve)）は
引数に"zero", "one", "two"、環境変数に "ENVVAR1=1", "ENVVAR2=2"を与えて、この動作の
様子を調べられるようにしたものです。起動されたプログラムの結果を見るために、
コマンドラインの引数 (argv) と環境変数 (environ) を出力するだけの別の簡単なプログラム
（[`show_info.c`](https://lwn.net/Articles/630754/#show_info)）も使用します。

引数と環境変数は呼び出されたプログラムに渡されます。しかし、呼び出されたバイナリの
argv[0]は`execve()`の呼び出し元で指定された値であることに注意してください。
argv[0]にプログラムの名前を持つことは、少なくともバイナリについては`execve()`自体が
要求したり取り締まったりする規則ではないのです。

```sh
    % ./do_execve ./show_info
    argv[0] = 'zero'
    argv[1] = 'one'
    argv[2] = 'two'
    ENVVAR1=1
    ENVVAR2=2
```

呼び出されるプログラムがバイナリプログラムではなく、スクリプトである場合は少し様子が
変わります。これを調べるために、環境を出力するプログラムに相当するシェルスクリプト
（[`show_info.sh`](https://lwn.net/Articles/630754/#show_info.sh)）を使用します。
execve()を呼び出す元のプログラムと一緒に見てみると、いくつかの違いがあることが
かります。

```sh
    % ./do_execve ./show_info.sh
    $0 = './show_info.sh'
    $1 = 'one'
    $2 = 'two'
    ENVVAR1=1
    ENVVAR2=2
    PWD=/home/drysdale/src/lwn/exec
```

まず、環境変数にカレントディレクトリを示すPWDの値が追加されました。次に、スクリプトに
渡される最初の引数が呼び出し元が指定した"zero"ではなく、スクリプトのファイル名になって
います。さらに実験してみると、PWD環境変数は`/bin/sh`スクリプトインタプリタが追加しており、
引数の変更はカーネルが行っていることがわかりました。

```sh
    % cat ./wrapper
    #!./show_info

    % ./do_execve ./wrapper
    argv[0] = './show_info'
    argv[1] = './wrapper'
    argv[2] = 'one'
    argv[3] = 'two'
    ENVVAR1=1
    ENVVAR2=2
```

具体的には、カーネルは最初の引数("zero")を削除し、2つの引数で置き換えています。
（スクリプトの最初の行から取得した）スクリプトインタプリタのプログラム名と
（スクリプトテキストを保持する）呼び出されたファイルの名前です。スクリプトの最初の
行がインタプリタのコマンドライン引数を含んでいる場合（たとえば、awkは入力をスクリプト
テキストではなくファイル名として扱うためには-fオプションが必要)、3番目の追加引数も
挿入され、すべての追加オプションが保持されます。

```sh
    % cat ./wrapper_args
    #!./show_info -a -b -c

    % ./do_execve ./wrapper_args
    argv[0] = './show_info'
    argv[1] = '-a -b -c'
    argv[2] = './wrapper_args'
    argv[3] = 'one'
    argv[4] = 'two'
    ENVVAR1=1
    ENVVAR2=2
```

ある段階までは、スクリプトをラップするスクリプトを呼び出すなどして、
このポップワン、プッシュツーの引数の変更を繰り返すこともできます。
このような変更のたびにラップスクリプト名がargv[1]にプッシュされます。

```
    argv[0]:  'zero'=>'./wrapper4'=>'./wrapper3'=>'./wrapper2'=>'./wrapper' =>'./show_info'
    argv[1]:  'one'   './wrapper5'  './wrapper4'  './wrapper3'  './wrapper2'  './wrapper'
    argv[2]:  'two'   'one'         './wrapper5'  './wrapper4'  './wrapper3'  './wrapper2'
    argv[3]:          'two'         'one'         './wrapper5'  './wrapper4'  './wrapper3'
    argv[4]:                        'two'         'one'         './wrapper5'  './wrapper4'
    argv[5]:                                      'two'         'one'         './wrapper5'
    argv[6]:                                                    'two'         'one'
    argv[7]:                                                                  'two'
```

しかし、これがいつまでも続くわけではあります。ラッパの階層が大きすぎると
ELOOPで処理が失敗します。

```sh
    % ./do_execve ./wrapper6
    Failed to execute './wrapper6', Too many levels of symbolic links
```

## カーネル: `struct linux_binprm`

では、カーネル空間に移動してexecve()システムコールを実装しているコードを掘り
下げましょう。以前の記事では汎用的なシステムコール機構（とexecve()に必要な
特別な知恵）を調査しました。そのため、`fs_exec.c`にある[`do_execve_common()`関数](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L1430)を
取り上げることができます。この関数のコードの主な目的は、現在のプログラム呼び出し
操作を記述する新しい[`struct linux_binprm`構造体](https://elixir.bootlin.com/linux/v3.18/source/include/linux/binfmts.h#L14)のインスタンスを構築することです。
構造体では

- `file`フィールドは実行されるプログラム用に新たにオープンした`struct file`を設定します。
  これにより、カーネルはファイルの内容を読み取り、そのファイルをどのように処理するかを
  決定することができます。
- `filename`と`interp`フィールドには、プログラムを格納するファイル名を設定します。
  異なる2つのフィールドがなぜ必要なのかは後で説明します。
- `bprm_mm_init()`関数は新プログラムの仮想メモリを管理するための`struct mm_struct`と
  `struct vm_area_struct`データ構造体の割り当てと設定を行います。特に、新プログラムの
  仮想メモリは対象アーキテクチャで可能な限り高いアドレスで終了し、そこからスタックが
  下に向かって成長します。
- `p`フィールドには新プログラムのメモリ空間の終端を指すように設定します。ただし、
  スタックの終端マーカとしてNULLポインタのためのスペースを残します。`p`の値は
  新プログラムのスタックに情報が追加されるたびに（下方に）更新されます。
- `argc`と`envc`フィールドは、引数と環境値の数を設定し、この情報を呼び出し処理の
  後半で新しいプログラムに伝達できるようにします。
- `unsafe`フィールドにはプログラムの実行が安全でないかもしれない理由を保持する
  ビットマスクを設定します。たとえば、プロセスが`ptrace()`でトレースされる場合や
  PR_SET_NO_NEW_PRIVSビットが設定されている場合などです。Linux Security Module
  （LSM）はこの情報を利用してプログラムの実行を拒否することができます。
- `cred`フィールドは別途割り当てられる`struct cred`タイプのオブジェクトであり、
  新プログラム用のクレデンシャル情報を保持します。これらは通常execve()を呼び出した
  プロセスから継承されますが、`setuid/setgid`ビットやその他の複雑な処理を可能に
  するために更新されます。`setuid/setgid`ビットの存在は、セキュリティに悪影響を
  与えるため、一連の互換性機能も禁止します。`per_clear`フィールドは、プロセスの
  パーソナリティのうち、後でクリアされるビットを記録します。
- `security`フィールドは、LSMがLSM固有の情報をlinux_binprmに保存できるようにします。
  LSMには`security_bprm_set_creds()`とLSMフック`bprm_set_creds`の呼び出しにより
  通知されます。このフックのデフォルト実装は新プログラムのLinuxケーパビリティを
  更新して、プログラムファイルのケーパビリティを有効にします。他のLSM実装はこの
  動作を独自のフック実装に組み込んでいます（たとえば、SmackやSELinuxなど）。
- `buf`スクラッチ空間はプログラムファイルの最初の（128バイト）データチャンクで
  満たされます。このデータはバイナリ形式を検出し適切に処理できるようにするために
  使用されます。

これらの設定処理のうち、実行される特定のファイルに依存する部分は、内部関数
[`prepare_binprm()`](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L1253)で
実行されます。この関数は後で別のファイル（たとえばスクリプトインタプリタ）が
実際に実行された場合に再び呼び出されてフィールドを更新する場合があります。

最後に、プログラム呼び出しに関する情報が内部ユーティリティ関数[`copy_strings()`](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L453)と[`copy_strings_kernel()`](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L544)を使用して新プログラムのスタックの先頭にコピーされます。
まず、プログラムのファイル名がスタックにプッシュされ（その位置は`linux_bprm`
インスタンスの`exec`フィールドに保存されます）、その後にすべての環境値とすべての
引数が続きます。この処理が終わるとスタックは次のようになります。

```
    ---------Memory limit---------
    NULL pointer                  <- スタックの終端マーカ
    program_filename string       <- pは最初ここをさしており、データが追加される
    envp[envc-1] string              たびにアドレスは下位方向に更新される
    ...                           ↓
    envp[1] string
    envp[0] string
    argv[argc-1] string
    ...
    argv[1] string
    argv[0] string
```

## バイナリフォーマットハンドラリストの処理: `struct linux_binfmt`

 完全な`linux_binprm`構造体を手に入れたら、プログラム実行の実際の作業は
 [`exec_binprm()`](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L1405)と（より重要な）[`search_binary_handler()`](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L1352)で実行されます。
 このコードはそれぞれが 特定のフォーマットのバイナリプログラムに対する
 ハンドラを提供する[`struct linux_binfmt`](https://elixir.bootlin.com/linux/v3.18/source/include/linux/binfmts.h#L70)オブジェクトのリストを走査して実行
 します。バイナリハンドラはカーネルモジュールで定義される可能性があるため、
 このコードでは各フォーマットに対して[`try_module_get()`](https://elixir.bootlin.com/linux/v3.18/source/kernel/module.c#L945)を呼び出すことで
 実行中に 他のタスクによって関連コードがアンロードされないことを保証しています。

`struct linux_binfmt`ハンドラオブジェクトごとに、`linux_binprm`オブジェクトを
引数に`load_binary()`関数ポインタを呼び出します。ハンドラコードが対象のバイナリ
形式をサポートしている場合、プログラムの実行準備に必要なすべての処理を行い、
成功（`>= 0`）を返します。そうでなければ、ハンドラは失敗コード（`< 0`）を返し、
次のハンドラの処理に移ります。

あるプログラムの実行が別のプログラムの実行に依存している場合があります。
実行可能なスクリプトはその明らかな例であり、これにはスクリプトインタプリタを
呼び出す必要があります。これに対処するため`search_binary_handler()`コードは
`struct linux_binprm`オブジェクトを再利用して再帰的に呼び出すことができます。
ただし、無限再帰を防ぐために再帰の深さは制限されており、先に述べたELOOPエラー
が発生します。

システムのLSMも操作に口を出します。バイナリ形式の反復処理が始まる前に
LSMフック`bprm_check_security`がトリガーされ、LSMが操作を許可するか
判断できるようになっています。その際、先に`linux_binprm.security`フィールドに
格納された状態を使用することができます。

繰り返し処理をしてもプログラムを処理できるフォーマットが見つからなかった場合
（そして、少なくとも最初の4バイトによりプログラムがテキストではなくバイナリで
あるように見える場合）、コードは"binfmt-XXXX"という名前のモジュールをロード
しようと試みます（ここでXXXXはプログラムファイルの3バイト目と4バイト目の16進値です）。
これはバイナリフォーマットハンドラとフォーマットをよりダイナミックに関連付ける
ために（1996年にLinux 1.3.57に追加された）古いメカニズムです。

## バイナリフォーマット

では、標準カーネルで利用できるバイナリ形式にはどのようなものがあるのでしょうか。
`struct linux_binfmt`のインスタンスを登録するコード（[`register_binfmt()`](https://elixir.bootlin.com/linux/v3.18/source/include/linux/binfmts.h?v=3.17#L82)と
[`insert_binfmt()`](https://elixir.bootlin.com/linux/v3.18/source/include/linux/binfmts.h?v=3.17#L87)) を検索するとかなりの数の可能なフォーマットが見つかり、
そのすべてが[`fs/Kconfig.binfmts`](https://elixir.bootlin.com/linux/v3.18/source/fs/Kconfig.binfmt)ファイルで構成・説明されています。

- [`binfmt_script.c`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_script.c#L108): `#!`行で始まるインタプリタスクリプトをサポートします。
- [`binfmt_misc.c`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_misc.c#L729): ランタイム構成にしたがい雑多なバイナリ形式をサポートします。
- [`binfmt_elf.c`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_elf.c#L2200): ELF形式のバイナリをサポートします。
- [`binfmt_aout.c`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_aout.c#L412): 昔のa.out形式のバイナリをサポートします。
- [`binfmt_flat.c`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_flat.c#L945): フラットフォーマットバイナリをサポートします。
- [`binfmt_em86.c`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_em86.c#L102): Alphaマシンで動作するIntel ELFバイナリをサポートします。
- [`binfmt_elf_fdpic.c`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_elf_fdpic.c#L94): ELF FDPICバイナリをサポートします。
- [`binfmt_som.c`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_som.c#L286): SOMフォーマット(HP/UX PA-RISCフォーマット)のバイナリをサポートします。

(その他、アーキテクチャ固有の形式もいくつかサポートしています）。

次のセクションではこれらのうち最も重要な軽視であるインタプリタスクリプトと
任意のフォーマットをサポートする"miscellaneous"メカニズムについて調べます。
次の記事では（通常はすべてのプログラム実行がたどり着く）ELFバイナリ形式に
ついて調べます。

## スクリプトの実行: `binfmt_script.c`

 文字列`#!`で始まるファイル（でexecuteビットがセットされているファイル）は
 スクリプトとして扱われ`fs/binfmt_script.c`ハンドラで処理されます。
 このコードは最初の2バイトをチェックした後、スクリプト呼び出し行の残りを解析し、
 それをインタプリタ名（`#!`以降、最初の空白までのすべて）と引数と思われるもの
 （行末までのすべて、外部の空白を除く）に分割します。

（注意点として、`linux_binprm`構造体の作成時にはプログラムの最初の128バイトしか
取得していません。これはインタプリタ名と引数がこれより長い場合、結果は切り捨て
られることを意味します)。

これらを取得したら、新プログラムのスタックのトップ（すなわち、最下位のアドレス）
からargv[0] を取り除き、次のものをプッシュし、`linux_binprm`オブジェクトのargc値を
変更します。

- プログラム名
- (オプションで)インタプリタ引数をまとめたもの
- インタプリタプログラム名

これらの事実から冒頭で観察したユーザ空間の動作が説明できます。新プログラムの
スタックは、次のように変更されます。

```
    ---------Memory limit---------
    NULL pointer
    program_filename string
    envp[envc-1] string
    ...
    envp[1] string
    envp[0] string
    argv[argc-1] string
    ...
    argv[1] string
    program_filename string
    ( interpreter_args )
    interpreter_filename string
```

 このコードでは`linux_binprm`構造体の`interp`フィールドの値も変更し、
 スクリプトのファイル名ではなく、インタプリタのファイル名を参照するようにします。
 これは`linux_binprm`構造体がなぜ2つの文字列を参照するかを説明します。
 1つ（interp）は現在実行したいプログラムで、もう1つはexecve()コールで元々
 呼び出された名前（filename）です。同様に、`linux_binprm`の`file`フィールドも
 新しいインタプリタプログラムを参照するように更新され、その内容の最初の128バイトが
 bufスクラッチスペースに読み込まれます。

スクリプトハンドラのコードは`search_binary_handler()`に再帰して、スクリプト
インタプリタプログラムに対してこの処理を繰り返します。インタプリタがスクリプトで
ある場合は`interp`の値は再度変更されますがファイル名は変更されません。

## Miscellaneousインタプリタ検出: `binfmt_misc.c`

以前、Linuxカーネルの初期のバージョンでは、バイナリの先頭バイトを含む名前の
カーネルモジュールを探すことで、フォーマットのサポートを動的に追加する荒削りな
方法をサポートしていることを見ました。これは特に便利なものではありません。
数バイトの検索だけでは（fileコマンドが使用する広大な範囲の検出シグネチャと
比較して）非常に限られており、カーネルモジュールを必要とすることは参入障壁を
高くします。

miscellaneousバイナリ形式ハンドラは、新しい形式を扱うより柔軟かつ動的な方法を
可能とし、実行時の構成を（`/proc/sys/fs/binfmt_misc`の下にマウントされた特別な
ファイルシステム経由) で指定可能にします。

- サポート形式の認識方法は、ファイル名の拡張子や特定のオフセットでのマジック値に
  基づきます（スクリプトインタプリタの解析と同様に、このマジック値はプログラム
  ファイルの最初の128バイト以内にある必要があります)。
- 呼び出すインタプリタプログラムは、argv[1]として渡されるプログラムファイル名で
  取得します（スクリプト呼び出しの場合と同様）。

miscellaneous形式ハンドラを使用している良い例はJavaファイルです。（0xCAFEBASE接頭辞に
基づいて）.classファイルを、また（.jar拡張子に基づいて）.jarファイルを検出し、
それらに対して自動的にJVM実行ファイルを呼び出します。miscellaneous構成は引数の
指定を許可しないので、これには関連するコマンドライン引数を提供するラッパスクリプトを
必要とします。miscellaneousハンドラがスクリプトハンドラを起動し、スクリプトハンドラが
JVM実行ファイルのためのELFハンドラを起動することを意味します（そしてそれは
おそらく最終的にはダイナミックリンカのld.soを起動することになります、それはまた
別の話でしょうが）。

内部的には、このフォーマットのためのカーネル実装はすでに説明したスクリプト
プログラムハンドラと同じです。ただし、最初に一致する構成エントリを検索することと
その構成が詳細の一部（argv[0]の削除など）をオプションにするために使用される
ことなどが違います。

スクリプト形式とmiscellaneous形式のためのハンドラは、その特定の形式に
必要なインタプリタプログラムを呼び出すために再帰的に実行されます。この再帰は
ある時点で終了しなければなりません。最近のLinuxシステムでは、ほとんどの場合、
それはELFバイナリプログラム（次回の記事のテーマです）で終わります。

# 2. [ELF binaries](https://lwn.net/Articles/631631/)

前回の記事では、ユーザ空間からexecve()が呼び出され、Linuxカーネルが
プログラムを実行する一般的な仕組みについて説明しました。しかし、
その記事で説明したフォーマットハンドラは内部でsearch_binary_handler()を
呼び出して実行のプロセスを延期していました。この再帰はほとんどの場合
この記事の主題であるELFバイナリプログラムの呼び出しで終了します。

## ELF形式

 ELF（Executable and Linkable Format）形式は、最近のLinuxシステムで
 使用されている主なバイナリ形式であり、[`fs/binfmt_elf.c`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_elf.c)で実装されています。
 カーネルが扱うには少し複雑な形式であり、主関数の[`load_elf_binary()`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_elf.c#L571)は
 400行以上に及び、古いa.out形式をサポートしているコードに比べて4倍以上の
 大きさになっています。

（共有ライブラリやオブジェクトファイルではない）実行プログラムのELFファイルは
必ずファイルの先頭付近（ELFヘッダの後）にプログラムヘッダテーブルを含んでいます。

カーネルは3種類のプログラムヘッダエントリにしか注意を払いません。第1は
PT_LOADセグメントであり、これは新プログラムの実行メモリの領域を記述して
います。これには実行ファイルのコードセクションとデータセクション、さらに
BSSセクションのサイズが含まれます。BSSはゼロで埋められます（したがって、
実行ファイルにはその長さしか保存されていません）。第2はPT_INTERPエントリであり、
プログラム全体を組み立てるのに必要なランタイムリンカを特定します。当面は
静的にリンクされたELFバイナリを想定し、ダイナミックリンクされたバイナリは後で
戻ることにします。最後に、カーネルは（もし存在すれば）PT_GNU_STACKエントリから
1ビットの情報を取得します。これはプログラムのスタックを実行可能にするか否かを
示します。

(この記事では、ELFプログラムをロードするために必要な点にだけ焦点を当て、
フォーマットの詳細をすべて調べることはしません。WikipediaのELFの記事からリンク
されている参考文献や、objdumpツールで実際のバイナリを調べると、より多くの情報を
得ることができます)。

## ELFバイナリの処理

ELFバイナリのロードは`load_elf_binary()`関数で処理されます。まず、ELFヘッダを
調べて対象のファイルが本当にサポートされているELFフォーマットであるかチェック
します。ハンドラはlinux_binprmのbufに読み込まれている最初の128バイトだけでなく
ELFプログラムヘッダのすべてを必要とするのでスクラッチスペースに読み込む必要が
あります。

この関数はプログラムヘッダのエントリをループし、インタプリタ（PT_INTERP）のチェックと、
（PT_GNU_STACK エントリから）プログラムのスタックを実行可能にするかどうかをチェック
します。これらのチェックが終わったら、旧プログラムから引き継いていない新プログラム
の属性を初期化する必要があります。Single UNIX Specification version 3 (SUSv3) の
[実行仕様](http://pubs.opengroup.org/onlinepubs/009695399/functions/exec.html)に必要な動作のほとんどが記述されています（また、The Linux Programming Interface
の表28-4にも関連する属性の優れた要約があります）。

新プログラムをセットアップするプロセスは[`flush_old_exec()`](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L1054)の呼び出しで始まります。
これは旧プログラムを参照しているカーネル内の状態をクリアします。旧プログラムの
すべてのスレッドはkillされ、新プログラムは単一のスレッドで開始します。また、
プロセスのシグナル処理情報は後で安全に変更できるように旧プログラとは共有されません。
旧プログラムの保留中のPOSIXタイマーはすべてクリアされ、（`/proc/pid/exe`で見る
ことができる）プログラムの実行ファイルの場所も更新されます。旧プログラムの仮想
メモリマッピングは解放され、保留中の非同期I/O操作もkillされ、uprobeは解放されます。
最後に、プロセスのpersonalityは更新され、（先に linux_binprmのper_clearフィールドに
記録されたとおり）セキュリティに影響を与える可能性のある機能は削除されます。また、
この関数は、SET_PERSONALITY()マクロを呼び出して、新64ビットプログラム用に
スレッドフラグを適切に設定することも行っています。

関連する[`setup_new_exec()`](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L1097)の呼び出しで新プログラムのためにカーネルの内部状態を
設定します。この関数は新プログラムがコアダンプを生成できるかどうか
（またはptrace()をアタッチさせるかどうか）を決定することから始めます。これらはsetuid
またはsetgidされたプログラムではデフォルトで無効にされます。ダンプ出力は
プログラムファイルが現在のクレデンシャルでは読み込み可能でない場合にも無効になります。
[`__set_task_comm()`](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L1045)の呼び出しは最初に起動されたファイル名のベースネームを現在の
タスクのcommフィールドに設定します。この値はスレッド名として使用され、[`prctl()`](http://man7.org/linux/man-pages/man2/prctl.2.html)の
PR_GET_NAMEとPR_SET_NAME操作によりユーザ空間からアクセスできます。
[`flush_signal_handlers`](https://elixir.bootlin.com/linux/v3.18/source/kernel/signal.c#L484)の呼び出しで新プログラム用のシグナルハンドラを設定します。
SIG_IGN以外のシグナルハンドラにはデフォルトのSIG_DFL値が設定されます（そのため、
無視されていたシグナルは新プログラムに継承されます)。最後に、[`do_close_on_exec()`](https://elixir.bootlin.com/linux/v3.18/source/fs/file.c#L596)
の呼び出しでO_CLOEXECフラグが設定されている旧プログラムのファイル記述子をすべて
閉じます。その以外のファイル記述子は新プログラムに継承されます。

新プログラム用の仮想メモリも設定する必要があります。（スタックオーバーフロー攻撃に
対する防御として）セキュリティを向上させるために、スタックの最上位アドレスは通常
ランダムなオフセットだけ下方に移動させます。最初に呼び出す[`setup_arg_pages()`](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L640)は
カーネルのメモリ追跡構造体を設定し、スタックの新しい位置を調整します。このコードは
プログラムファイル内のすべてのPT_LOADセグメントをループし、それらをプロセスの
アドレス空間にマップし、新プログラムのメモリレイアウトを設定します。そして、
プログラムのBSSセグメントに対応するゼロ詰めのページを設定します。また、仮想動的
共有オブジェクト（vDSO）ページなどの特別な追加ページもマッピングする必要があり、
[`arch_setup_additional_pages()`](https://elixir.bootlin.com/linux/v3.18/source/arch/x86/vdso/vma.c#L202)の呼び出しで処理しています。さらに、後方互換性の
ためにプログラムのアドレス空間のゼロアドレスに空のページをマップすることもできます
（古いSVr4プログラムはNULLポイントを読み込むとSIGSEGVではなくゼロを返すと仮定して
いるからです）。

次に、[`install_exec_creds()`](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L1187)を呼び出して新プログラムのクレデンシャルを設定します。
この関数は（LSM フックである`bprm_committing_creds`と`bprm_committed_creds`を介して)
アクティブなLinux Security Module（LSM）にクレデンシャルの変更を通知し、内部関数
[`commit_creds()`](https://elixir.bootlin.com/linux/v3.18/source/kernel/cred.c#L414)が割り当てを実行します。

新プログラムを実行するための最後の準備は`create_elf_tables()`関数を呼び出して
スタックの残りの部分を（新しいランダムな場所に）設定することです。これについては
以下の別のセクションで説明します。

これで準備はすべて完了し、新プログラムを起動できるようになります。前の記事で、
カーネルのsystem_callエントリポイントでカーネルのメインコードに入る前にユーザ
空間のCPUレジスタをカーネルスタックにプッシュし、システムコールの完了後に
これらのレジスタがリストアされる方法についてを説明しました。保存されるレジスタを
保持するスタックの領域はpt_regs構造体にキャストされ、保存されたユーザ空間の
CPUレジスタは新プログラムの開始に適した値（ゼロ）で上書きされています。
[`start_thread()`](https://elixir.bootlin.com/linux/v3.18/source/arch/x86/kernel/process_64.c#L230)関数の呼び出しは、保存された命令ポインタをプログラム（または
ダイナミックリンカ）のエントリポイントに設定し、保存されていたスタックポインタを
（linux_binprmのpフィールドから）現在のスタックトップに設定します。ハンドラからの
リターンコード0は成功を示し、execve()システムコールはユーザ空間に戻りますが、
そこは完全に別のユーザ空間であり、プロセスのメモリは再マップされており、復元
されたレジスタは新プログラムの実行を開始する値を持っています。

## スタックの設定: 補助ベクトルと環境変数、引数

[`create_elf_tables()`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_elf.c#L149)関数は、新プログラムのスタックに汎用コードによって追加された
引数と環境変数の下にさらに2つの異なる情報を追加します。まず、`arch_align_stack()`を
呼び出して既存のスタック位置を16バイト境界にアライメントします。さらにスタックを
わずか下方にランダムに位置づけることもできます。

第一の情報群はELF補助ベクトルです。これは実行中のプログラムとそれが実行されて
いる環境に関する有用な情報を記述する（id, value）ペアの集合であり、カーネルから
ユーザ空間に伝達されるものです。このベクトルを構築するために、処理コードはまず64ビット
値に適合しない情報をスタックにプッシュする必要があります。x86_64の場合、
プラットフォームケーパビリティ記述子（文字列の"x86_64"）と（ユーザ空間の乱数生成器の
シードとなる）16バイトのランダムデータがこれにあたります。

次に、補助ベクトル用の (id, value) ペアをmm_structのsaved_auxvスペースに配置します。
Michael Kerriskの[LWN記事](https://lwn.net/Articles/519085/)にこのベクトルの内容が記述されているので、ここでは、いくつかの
興味深いエントリにだけ言及します。

- ベクトルの（アーキテクチャ固有の）最初のエントリーは、x86_64ではAT_SYSINFO_EHDRです。
  これは以前の記事で説明したvDSOページの位置を示します。
- AT_PLATFORMは先にプッシュしたプラットフォームケーパビリティ記述子"x86_64"の場所です。
- AT_RANDOMは、先にプッシュしたランダムデータの場所です。
- AT_EXECFNは、引数と環境変数より上に位置するスタックに一番最初にプッシュした
  （そしてその位置をlinux_binprmのexecフィールドに格納した）プログラムファイル名の位置です。
- AT_ENTRYは、テキストセグメントのエントリポイント、すなわちプログラムの実行が開始
  されるべき場所を保持します。

この補助ベクトルを作成した後、コードは新プログラムのスタックの残りの部分を構築します。
必要な領域を計算し、低いアドレスから高いアドレスへとエントリを挿入していきます。

- まず、argcの引数カウントを挿入します。
- 次に、引数ポインタの配列を挿入し、最後にNULLポインタを挿入します。これがmain()の
  argvが最終的に指し示す場所です。
- 次に、環境ポインタの配列を挿入し、最後にNULLポインタを挿入します。これはenvironが
  指し示す場所です。
- 補助ベクトルは、それが参照する追加の値の直下の最上位アドレスに置きます。

これらをまとめると、新プログラムのアドレス空間の先頭は、次の例のような内容に
なります（[このページ](http://www.win.tue.nl/~aeb/linux/hh/stack-layout.html)にも同様の例があります）。

```
    ------------------------------------------------------------- 0x7fff6c845000
     0x7fff6c844ff8: 0x0000000000000000
            _  4fec: './stackdump\0'                      <------+
      env  /   4fe2: 'ENVVAR2=2\0'                               |    <----+
           \_  4fd8: 'ENVVAR1=1\0'                               |   <---+ |
           /   4fd4: 'two\0'                                     |       | |     <----+
     args |    4fd0: 'one\0'                                     |       | |    <---+ |
           \_  4fcb: 'zero\0'                                    |       | |   <--+ | |
               3020: random gap padded to 16B boundary           |       | |      | | |
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -|       | |      | | |
               3019: 'x86_64\0'                        <-+       |       | |      | | |
     auxv      3009: random data: ed99b6...2adcc7        | <-+   |       | |      | | |
     data      3000: zero padding to align stack         |   |   |       | |      | | |
    . . . . . . . . . . . . . . . . . . . . . . . . . . .|. .|. .|       | |      | | |
               2ff0: AT_NULL(0)=0                        |   |   |       | |      | | |
               2fe0: AT_PLATFORM(15)=0x7fff6c843019    --+   |   |       | |      | | |
               2fd0: AT_EXECFN(31)=0x7fff6c844fec      ------|---+       | |      | | |
               2fc0: AT_RANDOM(25)=0x7fff6c843009      ------+           | |      | | |
      ELF      2fb0: AT_SECURE(23)=0                                     | |      | | |
    auxiliary  2fa0: AT_EGID(14)=1000                                    | |      | | |
     vector:   2f90: AT_GID(13)=1000                                     | |      | | |
    (id,val)   2f80: AT_EUID(12)=1000                                    | |      | | |
      pairs    2f70: AT_UID(11)=1000                                     | |      | | |
               2f60: AT_ENTRY(9)=0x4010c0                                | |      | | |
               2f50: AT_FLAGS(8)=0                                       | |      | | |
               2f40: AT_BASE(7)=0x7ff6c1122000                           | |      | | |
               2f30: AT_PHNUM(5)=9                                       | |      | | |
               2f20: AT_PHENT(4)=56                                      | |      | | |
               2f10: AT_PHDR(3)=0x400040                                 | |      | | |
               2f00: AT_CLKTCK(17)=100                                   | |      | | |
               2ef0: AT_PAGESZ(6)=4096                                   | |      | | |
               2ee0: AT_HWCAP(16)=0xbfebfbff                             | |      | | |
               2ed0: AT_SYSINFO_EHDR(33)=0x7fff6c86b000                  | |      | | |
    . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .        | |      | | |
               2ec8: environ[2]=(nil)                                    | |      | | |
               2ec0: environ[1]=0x7fff6c844fe2         ------------------|-+      | | |
               2eb8: environ[0]=0x7fff6c844fd8         ------------------+        | | |
               2eb0: argv[3]=(nil)                                                | | |
               2ea8: argv[2]=0x7fff6c844fd4            ---------------------------|-|-+
               2ea0: argv[1]=0x7fff6c844fd0            ---------------------------|-+
               2e98: argv[0]=0x7fff6c844fcb            ---------------------------+
     0x7fff6c842e90: argc=3
```

スタックレイアウトには2つのランダム化（メモリの先頭位置と、引数値と補助ベクトルの
間のギャップの大きさ）がされていますが、新しく実行するプログラムはスタック上にある
すべての情報がどこにあるのかを把握できることに注意してください。SPレジスタは
スタックのトップ（つまり最下位アドレス）がどこかであるかをプログラムに伝え、
コマンドライン引数はそこからメモリ内の上方に配置され、終了位置を示すNULLポインタを
持っています。次に環境変数が見つかり、これもNULLポインタで終了が示され、補助
ベクトルは次の連続したアドレスに見つかり、AT_NULL IDで終了します。これらの情報の中で
見つかった値は、引数文字列、環境文字列、補助データ値のアドレスを与えるので
ランダムギャップのサイズに関する明示的な情報は必要ありません。

## ダイナミックリンクプログラム

 ここまでは、実行されるプログラムが静的にリンクされていると仮定し、ELFプログラム
 ヘッダのPT_INTERPエントリの存在によって引き起こされる処理をスキップしてきました。
 しかし、ほとんどのプログラムは動的にリンクされています。つまり、必要とされる共有
 ライブラリは実行時に配置され、リンクされなければならないのです。これはランタイム
 リンカ（通常は`/lib64/ld-linux-x86-64.so.2`など）によって実行されます。このリンカの
 身元はPT_INTERPプログラムヘッダエントリによって特定されます。

ランタイムリンカに対応するため、ELFハンドラはまずELFインタプリタファイル名を
スクラッチスペースに読み込み、[`open_exec()`](https://elixir.bootlin.com/linux/v3.18/source/fs/exec.c#L786)で実行ファイルを開きます。ファイルの
最初の128バイトをbprm->bufスクラッチ領域に読み込み、元のプログラムファイルの内容を
置き換え、インタプリタプログラムのELFヘッダにアクセスできるようにします。このため、
インタプリタは他の形式ではなく、ELFバイナリでなければなりません。

プログラムコードを前述のようにメモリにロードした後、ELFハンドラは[`load_elf_interp()`](https://elixir.bootlin.com/linux/v3.18/source/fs/binfmt_elf.c#L395)で
ELFインタプリタプログラムもメモリにロードします。この処理は、元のプログラムをロードする
処理と似ています。コードはELFヘッダのフォーマット情報をチェックし、ELFプログラムヘッダを
読み込み、ファイルからすべてのPT_LOADセグメンを新しいプログラムのメモリにマップし、
インタプリタのBSSセグメントのためのスペースを残します。

プログラムの実行開始アドレスは、プログラム自身のエントリポイントではなく、インタプリタの
エントリポイントに設定されます。execve()システムコールが完了すると、ELFインタプリタの
実行が始まり、プログラムが依存する共有ライブラリを見つけてロードし、プログラムの未定義
シンボルをライブラリの正しい定義に解決するなど、ユーザ空間からのプログラムのリンク要件を
満たすための処理を行います。このリンク処理（これはカーネルよりもはるかに深いELF形式の
理解に依存している）が完了すると、インタプリタは先に記録したAT_ENTRY補助値である
アドレスで新しいプログラムの実行を開始することができます。

## 他のアーキテクチャとの互換性

 前述したように、最新の64ビット（x86_64）Linuxシステムは、通常の32ビットバイナリ
 （x86_32）とx32 ABIプログラム（x86_64の追加レジスタを使用できる）という2種類の
 32ビットバイナリの実行もサポートしています。では、カーネルはどのようにしてこれらの
 バイナリをサポートしているのでしょうか。

これらのフォーマットをサポートする鍵となるファイルは`compat_binfmt_elf.c`です。これは
CONFIG_COMPAT_BINFMT_ELF設定オプションが設定されているとカーネルに取り込まれます。
このファイルはバイナリハンドラを登録するリストには登場していませんでした。それ自体に
ほとんどコードを含んでいないからです。代わりに主たるELFハンドラコードである
`binfmt_elf.c` を（#includeを使って）取り込み、プリプロセッサを使ってさまざまな内部
関数と値を32ビット互換バージョンにリダイレクトしています。これらの変更点以外では
フォーマットハンドラは上記の通常のELFハンドラと同じように動作します。

変更の1つは、ELFファイルのレイアウトを記述する構造体の32ビットバージョンを使用すること
です。同様に、32ビットバイナリ用の適切な定数値を使用し、互換ハンドラが関連する
ELFバイナリタイプのサポートのみを主張するようにします。特に、`elf_check_arch()`の
呼び出しは`compat_elf_check_arch()`に置き換えられ、x86_32または (設定されていれば)
x32のいずれかをチェックされるようになります。

プリプロセッサの変更により、ELFハンドラコードの内部機能の一部もリダイレクトされます。
SET_PERSONALITY()マクロの呼び出しは`set_personality_ia32()`にリダイレクトされ、32ビット
アーキテクチャに関連するスレッドフラグが設定されるようになり、同様に
`arch_setup_additional_pages()`関数は 32 ビットvDSOを設定するバージョンと取り替えられ
ます。さらに重要なことは、`start_thread()`関数が`start_thread_ia32() `に対応する `compat_start_thread()`に置き換えられることです。これにより、内部の`start_thread_common()`
関数の引数が変更され、保存されたセグメントレジスタがx86_64バイナリとは異なる方法で
初期化されるようになります（また、 `ELF_PLAT_INIT()`マクロも一致するように調整されます）。

## エピローグ

 Linuxシステムで実行されるすべてのプログラムは、execve()を通過します。したがって、
 これはカーネル機能の重要な一部であり、詳細に理解する価値があります。カーネルは
 スクリプトや他のマシンコード形式のプログラムをネイティブにサポートしていますが、
 最近のLinuxシステムでのプログラム実行は最終的にはELFバイナリを実行することになります。
 ELFは複雑なフォーマットですが、幸いにもカーネルはその複雑さをほとんど無視できます。
 カーネルは、セグメントをメモリにロードし、ユーザ空間のランタイムリンカプログラムを
 呼び出して実行するプログラムを完全に組み立てる仕事を完了するために必要なだけELFを
 理解すればいいのです。
