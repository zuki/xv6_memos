@(#)TOUR	8.1 (Berkeley) 5/31/93

**注記** -- 本書は`ash`とともに配布されたオリジナルTOUR文書であり、シェルの現状を表しているわけではありません。シェルがどのように構成されているかを知る上で有用な情報であるため提供しますが、状況は変わっていることに注意してください。最新のシェルは依然として開発中です。

<center>

# Ashをめぐるツアー

## Copyright 1989 by Kenneth Almquist.
</center>


**ディレクトリ**:  サブディレクトリ`bltin`にはスタンドアロンでコンパイルできるコマンドが含まれています。その他のコードはashディレクトリにあります。

**ソースコードジェネレータ**:  ファイル名が`"mk"`で始まるファイルはソースコードを生成するプログラムです。その全リストは次のとおりです。

        program         intput files        generates
        -------         ------------        ---------
        mkbuiltins      builtins            builtins.h builtins.c
        mkinit          *.c                 init.c
        mknodes         nodetypes           nodes.h nodes.c
        mksignames          -               signames.h signames.c
        mksyntax            -               syntax.h syntax.c
        mktokens            -               token.h
        bltin/mkexpr    unary_op binary_op  operators.h operators.c

これは間違いなく多すぎます。`mkinit`はすべてのCソースファイルから次のようなエントリを検索します。

        INIT {
              x = 1;    /* executed during initialization */
        }

        RESET {
              x = 2;    /* executed when the shell does a longjmp
                           back to the main command loop */
        }

mkinitは特定のイベントが発生した際の処理ルーチンとしてこのコードを取り出します、その意図は、どのモジュールが明示的に初期化/リセットされる必要があるかという情報をモジュール自体の中に分離することによりモジュール性を向上させることにあります。

mkinitは`init.c`ファイルに宣言を配置するためのいくつかの構成要素を認識します。

```
        INCLUDE "file.h"
```
はファイルをインクルードします。

ストレージクラス`MKINIT`は宣言をinit.cファイルで利用可能にします。

```
        MKINIT int funcnest;    /* depth of function calls */
```

次のように行にMKINITを単独でおくと構造体や共用体を宣言できます。

```
        MKINIT
        struct redirtab {
              short renamed[10];
        };
```

プリプロセッサの`#define`文は特別な操作を行わなくてもinit.cにコピーされます。

**インデント**: ashのソースは6スペースでインデントされています。私が耳にしているインデントに関する唯一の研究は最適なインデントは4から6スペースの範囲であると結論づけています。私は広く使われている8スペースからそれほど大きくはずれていないので6スペースを使っています。6スペースがどうしても嫌なら（同梱されている）adjindプログラムを使って他のインデントに変えてください。

**例外**: 例外を扱うコードは`exceptions.c`（現在は`error.c`）にあります。C言語には例外処理がないのでsetjmpとlongjmpを使って実装しています。グローバル変数`exception`は例外のタイプを保持します。errorを呼び出すことでEXERRORが発生します。EXINTは割り込みです。（現在は、`exraise(EXERROR)`、`exraise(EXINT)`で発生）

**割り込み**: 対話型シェルでは割り込みはEXINT例外を発生させ、メインコマンドループに処理が戻ります（例外: ユーザーが`trap`コマンドを使用して割り込みをトラップするとEXINTは発生しません）。（`exception.h`（現在は`error.h`）で定義されている）INTOFFマクロとINTONマクロは割り込みができないクリティカルセクションを提供します。INTOFFの実行からINTONの実行までの間、割り込み信号は後で配送するために保持されます。INTOFFとINTONはネストすることができます。

**MEMALLOC.C**: `memalloc.c`はメモリ不足の際にエラーを呼び出す`malloc`と`realloc`を定義しています。また、スタック指向のメモリ割り当てスキームも定義しています。スタックからの割り当てはおそらくmallocによる割り当てよりも効率的ですが、それより、例外が発生した際にスタックポインタの復元するだけでその時点で使用中であったメモリを解放することができるという大きな利点があります。スタックはブロックの連結リストを使用して実装されています。

**STPUTC**: スタックが連続であれば、文字列の長さを事前に知らなくてもスタックに文字列を格納するのは簡単です。

```c
        p = stackptr;
        *p++ = c;       /* repeated as many times as needed */
        stackptr = p;
```

（`memalloc.h`で定義されている）次の3つのマクロはこれらの操作を行いますが終端から実行するとスタックが大きくなります。

```c
        STARTSTACKSTR(p);
        STPUTC(c, p);   /* repeated as many times as needed */
        grabstackstr(p);
```

ではコードを最初から見て行きましょう。

**MAIN.C**: メインルーチンは初期化処理を行い、必要であればユーザのプロファイルを実行し、`cmdloop`を呼び出します。cmdloopはコマンドの解析と実行を繰り返します。

**OPTIONS.C**: このファイルにはオプション処理コードが含まれています。シェルが起動されると`main`から呼び出されてシェルの引数を解析します。また、set組み込み関数も含まれています。オプション`-i`と`-j`(後者はジョブコントロールをオンにします)はシグナル処理の変更を要求します。`setjobctl`（jobs.cにある）と`setinteractive`（trap.cにある）はこれらのオプションの変更を処理するために呼び出されます。

**パース**: パーサのコードはすべて`parser.c`にあります。再帰下降パーサが使用されています。構文テーブル（`mksyntax`で生成）は字句解析の際に文字を分類するために使用されます。構文テーブルには通常用、一重引用符内用、二重引用符内用の3つのテーブルがあります。これらのテーブルはマシン依存です。テーブルは文字変数によりインデックス化されますが、charの範囲はマシンによって異なるためです。

**パーサ出力**: パーサの出力はノードのツリーで構成されます。様々なノードタイプがファイル`node-types`で定義されています。

NARGタイプのノードは単語とヒアドキュメントの内容を表現するために使用される。a初期バージョンのashではヒアドキュメントの内容を一時ファイルに保存していたが、ヒアドキュメントの内容をメモリに保存すると一般にパフォーマンスが大幅に向上します。ヒアドキュメント用に一時ファイルを使用するオプションがあれば小さなマシンには役に立ち、素晴らしかったでしょう。しかし、一時ファイルを削除するタイミングを追跡するコードは複雑なので私はそのバグをすべて修正することはできませんでした（AT&TはBourneシェルを10年以上保守していますが、私の知る限り、いまだに曖昧なケースで一時ファイルを正しく扱えるようにはなっていません)。

NARG構造体のテキストフィールドは単語のテキストを指しています。テキストは通常文字と`parser.h`で定義されている特殊コードで構成されます。特殊コードは次の通りです。

```
        CTLVAR              Variable substitution
        CTLENDVAR           End of variable substitution
        CTLBACKQ            Command substitution
        CTLESC              Escape next character
```

変数置換は以下の要素を含みます。

```
        CTLVAR type name '=' [ alternative-text CTLENDVAR ]
```

typeフィールドは置換種別を指定する1文字です。可能なタイプは次の通りです。

```
        VSNORMAL            $var
        VSMINUS             ${var-text}
        VSMINUS|VSNUL       ${var:-text}
        VSPLUS              ${var+text}
        VSPLUS|VSNUL        ${var:+text}
        VSQUESTION          ${var?text}
        VSQUESTION|VSNUL    ${var:?text}
        VSASSIGN            ${var=text}
        VSASSIGN|VSNUL      ${var=text}
```

次は変数名で等号で終端します。タイプがVSNORMAL以外の場合は、置換するテキストフィールドが続き、CTLENDVARバイトで終了します。

バッククォート内のコマンドは解析され、リンクリストに格納されます。文字列におけるこれらのコマンドの位置はCTLBACKQ文字で示されます。

CTLESC文字は次の文字をエスケープするので、入力に上記のCTL文字が現れた場合もそのまま通過させることができます。CTLESCは'*', '?', '[', '!'をエスケープするためにも使用されます。これらはユーザが引用されるため、ファイル名の生成に使用すべきでありません。

CTLESC文字は正しく扱うことが特に難しいことが分かっています。変数やコマンド置換の対象とならないヒアドキュメントの場合、パーサはそもそもCTLESC文字を挿入しません（そのため、テキストフィールドの内容は何も処理することなく書き込むことができます)。その他のヒアドキュメント、分割やファイル名生成の対象とならない単語は、変数とコマンドの置換の段階でCTLESC文字が削除されます。分割とファイル名生成の対象となる単語は、ファイル名生成の段階でCTLESC文字が削除されます。

**実行**: コマンド実行は次のファイルにより処理されます。

        eval.c     トップレベル関数
        redir.c    入出力のリダイレクトを処理するコード
        jobs.c     forkとwait, ジョブ制御を処理するコード
        exec.c     パス検索と実際のシステムコールを行うコード
        expand.c   引数を評価するコード
        var.c      変数シンボルテーブルを管理する。expand.cから呼ばれる。

**EVAL.C**: Evaltreeは再帰的に解析木を実行します。終了ステータスはグローバル変数exitstatusで返されます。バッククォート中のコマンドの評価には代替エントリのevalbackcmdが呼び出されます。コマンドがビルトインの場合は結果をメモリに保存し，そうでなければコマンドを実行する子プロセスをフォークし，子プロセスの標準出力をパイプに接続します。

**JOBS.C**: プロセスを作るには，makejobを呼んでジョブ構造体を作成し（このジョブ構造体を引数に）forkshellを呼び出してプロセスを作成します。Waitforjobはジョブが完了するのを待ちます。これらのルーチンはジョブ制御が定義されていればプロセスグループの面倒を見ます。

**REDIR.C**: Ashは、子プロセスをフォークすることなく、ファイル記述子のリダイレクトとリストアが可能です。これは、元のファイル記述子を複製することで実現されます。redirtab構造体にはどこでファイル記述子が複製されたのかが記録されています。

**EXEC.C**: find_command関数はコマンドの場所を特定し、ハッシュテーブルにまだない場合はそのコマンドを追加します。第3引数にはコマンドが見つからなかった場合にエラーメッセージを表示するかを指定します(パイプラインが設定されている場合、フォークを行う前にパイプライン内のすべてのコマンドに対してfind_commandが呼び出されます。これによりこれらのコマンドが親プロセスのハッシュテーブルに入れられます。ただし、コマンドのハッシュ化をできるだけ透過的にするためにこの時点でのエラーは黙って無視し、後でコマンドが見つからなかった場合にのみエラーメッセージを表示するようにしています)。

shellexec関数はexecシステムコールへのインタフェースです。

**EXPAND.C**: 引数は3段階で処理されます。（argstr関数で実行される）第1段階では変数とコマンド置換を行います。第2段階（ifsbreakup）では単語分割を行い、題3段階（expandmeta）ではファイル名生成を行います。"/u"ディレクトリがシミュレートするのであれば"/u/username"がユーザのホームディレクトリに置き換えられる際に"didudir"というフラグが設定されます。これはcdコマンドに、"/u"ディレクトリがシンボリックリンクを用いて実装されているかのようにディレクトリ名を出力すべきであることを伝えます。

**VAR.C**: 変数はハッシュテーブルに格納されます。今後、拡張可能なハッシュに切り替えるべきだと思います。変数名は（"name=value"の形式で）値と同じ文字列に格納されますので、コマンドの環境変数を作るために文字列をコピーする必要はありません。シェルが内部で参照する変数は事前にアロケートされるので、シェルは検索することなくこれらの変数の値を参照することができます。

プログラムを実行するとeval.cのコードは、重複を取り除く最も簡単な方法として（"PATH=xxx command"のような）コマンドの前にある環境変数を変数表に貼り付け、"environment"を呼び出して環境変数の値を取得します。これにより2つのことが導かれます。まず、PATHへの代入がコマンドの前にある場合、代入前のPATHの値を記録し、shellexecに渡さなければなりません。次に、プログラムがシェルプロシージャであることが判明した場合、コマンドの前にあった環境変数の文字列を変換表から取り出し、mallocから取得した文字列に置き換える必要があります。前者はスタックが空になると自動的に開放されるからです（memalloc.cに関するエントリを参照）。

**BUILTIN COMMANDS**: これらを処理するプロシージャはコード全体の最も適切と思われる場所に散らばっています。これらのプロシージャは名前が"cmd"で終わっているので判別することができます。名前とプロシージャの対応はmkbuiltinsコマンドで作成されるbuiltinsファイルで指定されます。

組み込みコマンドは通常プログラムと同じようにargcとargvを設定して起動されます。組み込みコマンドはその引数を上書きすることができます。組み込み関数はnextoptを呼び出してオプションの解析を行うことができます。これはgetoptのようなものですが、argcとargvは渡してはいけません。組み込み関数はerrorを呼び出すこともできます。この関数は通常シェルを終了させます (または、シェルが対話型の場合はメインコマンドループに戻ります)が、組み込みコマンドから呼ばれた場合は組み込みコマンドを終了ステータス`2`で終了させます。

bltinsディレクトリには独立にコンパイル可能なコマンドが含まれていますが、効率性のためにシェルに組み込むこともできます。このディレクトリにあるmakefileは（呼び出し元がashでなくても実行できるように）これらのプログラムを通常の方法でコンパイルしますが、ashとリンクできるbltinlib.a という名前のライブラリも作成します。ヘッダーファイル bltin.hは、ashとタンドアロン環境の違いのほとんどに対応します。ユーザはメインルーチンを"main"と呼び、#define mainにプログラムがashにリンクされたときに使用されたルーチンの名前を指定します。この#defineはbltin.hがインクルードされる前に現れる必要があります。bltin.hはプログラムがスタンドアロンでコンパイルされるべきである場合、#undef mainするからです。

**CD.C**: このファイルはcdとpwdの組み込みコマンドを定義しています。pwdコマンドは最初に呼び出されたときは/bin/pwdを実行します (すでにユーザが絶対パス名にcdしている場合を除く)がその後はカレントディレクトリを記憶し、 cdコマンドが実行される度にそれを更新するので、その後のpwdコマンドは非常に高速に実行されます。cdコマンドで最も複雑なものはdocdコマンドです。これはシンボリックリンクを実際の名前に解決し、シンボリックリンクをたどった場合、ユーザがどこにたどり着いたかを通知します。

**シグナル**: trap.cはtrapコマンドを実装しています。setsignal関数は、シグナルを受信したときにどのようなアクションを取るべきかを判断し、シグナルのアクションを適切に設定するためにシグナルシステムコールを呼び出します。ユーザがトラップを設定したシグナルをキャッチすると"onsig"関数はフラグを設定します。"dotrap"関数が実際にシグナルを処理するために適切な時点で呼び出されます。割り込みが補足されたがその信号に対するトラップが設定されていない場合はerror.cにあるonint関数が呼び出されます。

**出力**: Ashは独自の出力ルーチンを持っています。3つの出力構造体が割り当てられています。"output"は標準出力、"errout" は標準エラー出力、"memout"はメモリに保存される出力です。memout構造体は組み込みコマンドがバッククォート内に現れた場合に使用され、UNIXオペレーティングシステムを介したI/Oを行わうことなくその出力を集めることを可能にします。変数out1とout2は通常、各々outputとerroutを指していますが、バッククォート内で使用される場合はmemoutを指すように設定されます。

**入力**: 基本的な入力ルーチンはpgetcです。これは現在の入力ファイルから読み込みます。入力ファイルのスタックがあり、現在の入力ファイルはこのスタックの一番上のファイルです。このコードは入力をファイルではなく文字列にすることもできます（これは、-c オプションと"."、eval組み込みコマンド用です）。グローバル変数plinnoはファイルをスタックにプッシュまたはスタックからポップするたびに保存・復元されます。パーサルーチンはこの変数に現在の行番号を格納します。

**デバッグ**: DEBUGがshell.hで定義されている場合、シェルはデバッグ情報を$HOME/traceに書き込みます。これのほとんどは、たとえば、"TRACE(("n=%d0, n)")"のような2重括弧の中にprintf引数のセットを取るTRACEマクロを使って行われます。2重括弧が必要なのはプリプロセッサは可変数の引数を持つ関数を扱えないからです。DEBUGを定義するとquitシグナルを送信した際にシェルがコアダンプを生成するようになります。トレースコードはshow.cにあります。