# サブプロセスとそれに渡すスイッチの指定

[Using the GNU Compiler Collection (GCC): 3.20 Specifying Subprocesses and the Switches to Pass to Them](https://gcc.gnu.org/onlinedocs/gcc/Spec-Files.html)の翻訳

gccはドライバプログラムです。GCCは、コンパイル、アセンブル、リンクなどの作業を
行うために、一連のプログラムを起動して仕事を行います。GCCは、コマンドラインの
パラメータを解釈して、どのプログラムを起動すべきか、またどのコマンドライン
オプションをコマンドラインに置くべきかを判断します。この動作は、spec文字列に
よって制御されます。ほとんどの場合、GCCが呼び出すことができるプログラムごとに
1つのspec文字列がありますが、いくつかのプログラムでは、その動作を制御するために
複数のspec文字列があります。GCCに組み込まれているspec文字列は、-specs=コマンド
ライン・スイッチを使ってspecファイルを指定することで上書きすることができます。

specファイルとは、spec文字列を構成するためのプレーンテキストのファイルです。
specファイルは、空白行で区切られた一連の指示文で構成されています。ディレクティブの
種類は、その行の最初の空白以外の文字によって決定され、以下のいずれかになります。

<dl>
    <dt>%command</dt>
    <dd>specファイルプロセッサにコマンドを発行します。ここで表示できるコマンドは次の通りです。
        <dl>
            <dt>%include &lt;file&gt;</dt>
            <dd>ファイルを検索し、そのテキストをスペックファイルの現在の位置に挿入します。</dd>
            <dt>%include_noerr &lt;file&gt;</dt>
            <dd>`%include`と同じですが、includeファイルが見つからなくてもエラーメッセージを生成しません。</dd>
            <dt>%rename old_name new_name</dt>
            <dd>spec文字列`old_name`を`new_name`にリネームします。</dd>
        </dl>
    </dd>
    <dt>*[spec_name]:</dt>
    <dd>コンパイラに指定されたスペック文字列の作成、オーバーライド、削除を指示します。
    この指示の後、次の指示または空行までのすべての行が、spec 文字列のテキストとみなされます。
    この結果、空の文字列になった場合は、specが削除されます。(そうでなければ、specが存在して
    いなければ、新しいspecが作成され、specが存在している場合はその内容がこのディレクティブの
    テキストで上書きされます。ただし、テキストの最初の文字が '+' である場合は、そのテキストが
    specに追加されます。</dd>
    <dt>[suffix]:</dt>
    <dd>新たな ‘[suffix] spec’ ペアを作成します。このディレクティブの後、次のディレクティブ
    または空行までのすべての行が、指定された接尾辞のspec文字列とみなされます。コンパイラは
    指定された接尾辞を持つ入力ファイルに出会うと、そのファイルをどのようにコンパイルするかを
    決定するために、spec文字列を処理します。例えば、以下のようになります。<br/>

        .ZZ:
        z-compile -input %i

これは、名前が'.ZZ'で終わる入力ファイルはすべてプログラム'z-compile'に渡され、コマンド
    ラインスイッチ'-input'で起動され、'%i'の置換を実行した結果が渡されることを意味します。
    (以下を参照)。

スペック文字列を指定する代わりに、サフィックスディレクティブの後に続くテキストを以下の
いずれかにすることができます。
        <dl>
            <dt>@language</dt>
            <dd>接尾辞が既知の言語のエイリアスであることを示します。これは、GCCの-x
            コマンドラインスイッチを使って、言語を明示的に指定するのと同じです。
            たとえば、以下のようになります。

                .ZZ:
                @c++

これは、.ZZファイルがC++のソースファイルであることを示します。</dd>
            <dt>#name</dt>
            <dd>次のようなエラーメッセージを表示します

                name compiler not installed on this system.

</dd>
        </dl>
GCCには、すでに広範な接尾辞のリストが組み込まれています。このディレクティブは、接尾辞の
リストの最後にエントリを追加します。リストは最後から逆に検索されるので、このテクニックを
使って従来のエントリを上書きすることが可能です。

    </dd>
</dl>

GCCには、以下のようなスペック文字列が組み込まれています。スペックファイルはこれらの文字列を
上書きしたり、独自の文字列を作成することができます。個々のターゲットは、このリストに独自の
仕様文字列を追加することもできることに注意してください。

```
asm          アセンブラに渡すオプション
asm_final    アセンブラポストプロセッサに渡すオプション
cpp          Cプリプロセッサに渡すオプション
cc1          Cコンパイラに渡すオプション
cc1plus      C++コンパイラ渡すオプション
endfile      リンクの末尾にインクルードするオブジェクトファイル
link         リンカに渡すオプション
lib          リンカへのコマンドラインでインクルードするライブラリ
libgcc       リンカにどのGCCサポートライブラリを渡すかを決定する
linker       リンカの名前を設定する
predefines   Cプリプロセッサに渡されるDefine
signed_char  デフォルトでcharがsinedであることをCPPにわたすためのDefine
startfile    リンクの最初にインクルードするオブジェクトファイル
```

以下に、スペックファイルの小さな例を示します。

```
%rename lib                 old_lib

*lib:
--start-group -lgcc -lc -leval1 --end-group %(old_lib)
```

この例では、'lib'という名前のspecを'old_lib'にリネームし、以前の'lib'の定義を新しい定義で
上書きしています。新しい定義では、古い定義のテキストを含める前に、いくつかの追加のコマンド
ライン・オプションが追加されています。

spec文字列は、対応するプログラムに渡されるコマンドラインオプションのリストです。さらに、
スペック文字列には、変数テキストを置換したり、コマンドラインにテキストを条件付きで挿入
したりするために、'%'でプレフィックスされたシーケンスを含めることができます。これらの
構造を利用して、非常に複雑なコマンドラインを生成することができます。

以下は、仕様文字列に定義されているすべての'%'シーケンスの一覧です。これらのシーケンスを
展開した結果の周囲にはスペースは自動的には生成されないことに注意してください。したがって、
これらを連結したり、1つの引数で定数テキストと組み合わせたりすることができます。

<dl>
    <dt>%%</dt>
    <dd>1個の'%'がプログラム名または引数に置換されます</dd>
    <dt>%"</dt>
    <dd>カラ引数に置換されます</dd>
    <dt>%i</dt>
    <dd>処理中の入力ファイル名に置換されます</dd>
    <dt>%b</dt>
    <dd>処理中の入力ファイルに関連する出力用のbasenameに置換されます。これは多くの場合、
    最後のピリオドまでの（そしてそれを含まない）部分文字列であり、ディレクトリは含みませんが、
    %wがアクティブでない限り、補助出力用のbasenameに展開されます。明示的な出力名や補助出力の
    命名法を制御する他の様々なオプションの影響を受ける可能性があります。</dd>
    <dt>%B</dt>
    <dd>これは'%b'と同じですが、ファイルの接尾辞（最後のピリオドの後のテキスト）を
    含みます。%wがない場合、ダンプ出力用のbasenameに展開されます。</dd>
    <dt>%d</dt>
    <dd>GCCが正常に終了した場合に、そのファイルが削除されるように、'%d'を含む、または続く
    引数を一時的なファイル名としてマークします。'%g'とは異なり、これは引数にテキストを
    追加しません。</dd>
    <dt>%gsuffix</dt>
    <dd>suffixを接尾辞として持ち、コンパイルごとに1回選択されるファイル名に置換され、
    引数を'%d'と同じようにマークします。denial-of-service攻撃にさらされる可能性を減らす
    ために、今ではファイル名は以前に選ばれたファイル名がわかっていても、予測が難しい方法で
    選ばれます。たとえば、'%g.s ... %g.o ... %g.s' は、'ccUVUUAU.s ccXYAXZ12.o ccUVUUAU.s' の
    ようになります。接尾辞は、正規表現 '[.A-Za-z]*' または特別な文字列 '%O' にマッチしますが、
    これは '%O' が前処理された場合とまったく同じように扱われます。以前は、'%g' は、付加された
    接尾辞に関係なく、コンパイルごとに選択されたファイル名で単純に置換されていたため（つまり、
    通常のテキストと同様に扱われていたので）、そのような攻撃が成功する可能性が高くなって
    いました。</dd>
    <dt>%usuffix</dt>
    <dd>'%g'と似ていますが、コンパイルごとに1度ではなく、出現するたびに新しい一時ファイル名を
    生成します。</dd>
    <dt>%Usuffix</dt>
    <dd>'%usuffix'で生成された最後のファイル名に置換されます。そのような最後のファイル名が
    ない場合は新しいファイル名を生成します。'%usuffix'がない場合、これは'%gsuffix'と同じですが、
    同じ接尾辞空間を共有していないため、'%g.s ... %U.s ... %g.s ... %U.s'では、'%g.s'と'%U.s'で
    2つの異なるファイル名が生成されます。従来、'%U'は、付加された接尾辞に関係なく、前の'%u'で
    選ばれたファイル名で単純に置換されていました。</dd>
    <dt>%jsuffix</dt>
    <dd></dd>
    <dt>%|suffix<br/>%msuffix</dt>
    <dd></dd>
    <dt>%.SUFFIX</dt>
    <dd></dd>
    <dt>%w</dt>
    <dd></dd>
    <dt>%V</dt>
    <dd></dd>
    <dt>%o</dt>
    <dd></dd>
    <dt>%O</dt>
    <dd></dd>
    <dt>%I</dt>
    <dd></dd>
    <dt>%s</dt>
    <dd>カレント引数は何らかのライブラリまたはスタートアップファイルの名前です。標準的な
    ディレクトリのリストからそのファイルを検索し、見つかったフルネームで置換されます。
    カレントワーキングディレクトリがスキャンされるディレクトリのリストに含まれます。</dd>
    <dt>%T</dt>
    <dd></dd>
    <dt>%estr</dt>
    <dd></dd>
    <dt>%nstr</dt>
    <dd></dd>
    <dt>%(name)</dt>
    <dd></dd>
    <dt>%x{option}</dt>
    <dd></dd>
    <dt>%X</dt>
    <dd></dd>
    <dt>%Y</dt>
    <dd></dd>
    <dt>%Z</dt>
    <dd></dd>
    <dt>%M</dt>
    <dd></dd>
    <dt>%R</dt>
    <dd></dd>
    <dt>%a</dt>
    <dd></dd>
    <dt>％A</dt>
    <dd></dd>
    <dt>%l</dt>
    <dd></dd>
    <dt>%D</dt>
    <dd>GCCがスタートアップファイルを含んでいると考える各ディレクトリに対して、-Lオプションを
    ダンプします。ターゲットがmultilibsをサポートしている場合は、現在のmultilibsディレクトリが
    これらの各パスの前に追加されます。</dd>
    <dt>%L</dt>
    <dd></dd>
    <dt>%G</dt>
    <dd></dd>
    <dt>%S</dt>
    <dd></dd>
    <dt>%E</dt>
    <dd></dd>
    <dt>%C</dt>
    <dd></dd>
    <dt>%1</dt>
    <dd></dd>
    <dt>%2</dt>
    <dd></dd>
    <dt>%*</dt>
    <dd></dd>
    <dt>%&lt;S</dt>
    <dd></dd>
    <dt>%&lt;S+</dt>
    <dd></dd>
    <dt>%&gt;S</dt>
    <dd></dd>
    <dt>%:function(args)</dt>
    <dd></dd>
    <dt>%{S}</dt>
    <dd>GCCに-Sスイッチが指定されている場合、そのスイッチに置換されます。
    そのスイッチが指定されていない場合は、何にも置換されません。このオプションを
    指定する際には、先頭のダッシュは省略され、置換が実行されると自動的に挿入される
    ことに注意してください。したがって、スペック文字列'%{foo}'は、コマンドライン
    オプション'-foo'にマッチし、コマンドラインオプション'-foo'を出力します。</dd>
    <dt>%W{S}</dt>
    <dd></dd>
    <dt>%@{S}</dt>
    <dd></dd>
    <dt>%{S*}</dt>
    <dd></dd>
    <dt>%{S*&T*}</dt>
    <dd></dd>
    <dt>%{S:X}</dt>
    <dd>GCCに-Sスイッチが与えられた場合、Xに置換されます</dd>
    <dt>%{!S:X}</dt>
    <dd>GCCに-Sスイッチが与えられていない場合、Xに置換されます</dd>
    <dt>%{S*:X}</dt>
    <dd></dd>
    <dt>%{.S:X}</dt>
    <dd></dd>
    <dt>%{!.S:X}</dt>
    <dd></dd>
    <dt>%{,S:X}</dt>
    <dd></dd>
    <dt>%{S|P:X}</dt>
    <dd></dd>
    <dt>%{%:function(args):X}</dt>
    <dd></dd>
    <dt>%{S:X; T:Y; :D}</dt>
    <dd></dd>
    <dt></dt>
    <dd></dd>
</dl>

‘%{S}’, ‘%{S:X}’ などの構成でテキストSにマッチするスイッチは、バックスラッシュを使用して、
それに続く文字の特別な意味を無視することができます。例えば、'%{std=iso9899\:1999:X}'は、
-std=iso9899:1999オプションが指定されている場合、Xに置き換えられます。

 ‘%{S:X}’などの条件付きテキストXには、ネストした他の'%'構造やスペース、あるいは改行が
 含まれることがあります。これらは、上述のように通常通りに処理されます。Xの最後のホワイト
 スペースは無視されます。また、これらの構文のコロンの左側には、'.'または'*'と対応する単語の
 間を除いて、どこにでもホワイトスペースを入れることができます。

-O、-f、-m、-W の各スイッチは、これらの構成要素の中で特別に扱われます。-Oの別の値や
-f, -m, -Wスイッチの否定形がコマンドラインの後方に存在する場合、前方のスイッチの値は
無視されます。ただし、{S*}（ここでSは1文字）は例外で、すべての一致するオプションを通過させます。

述語テキストの先頭にある文字'|'は、-pipeが指定されている場合に限り、コマンドが次のコマンドに
パイプされるべきであることを示すために使用されます。

どのスイッチが引数を取り、どのスイッチが取らないかは、GCCに組み込まれています。(これを一般化して、
各コンパイラの仕様でどのスイッチが引数を取るかを指定できるようにすると便利だと思うかもしれません。
しかし、これは一貫した方法ではできません。GCCは、どのスイッチが引数を取るかを知らないと、どの
入力ファイルが指定されているかを判断することさえできませんし、どのコンパイラを実行するかを
指示するために、どの入力ファイルをコンパイルするかを知らなければなりません）。

また、GCCは、-lで始まる引数がコンパイラの出力ファイルとして扱われ、他の出力ファイルの中の適切な
位置でリンカに渡されることを暗黙のうちに知っています。


# `aarch64-linux-gnu-gcc`のデフォルトspecs

```
$ aarch64-linux-gnu-gcc -dumpspecs

*asm:
%{mbig-endian:-EB} %{mlittle-endian:-EL} %{march=*:-march=%*} %(asm_cpu_spec)%{mabi=*:-mabi=%*}

*asm_debug:
%{g*:%{%:debug-level-gt(0):--gdwarf2}} %{fdebug-prefix-map=*:--debug-prefix-map %*}

*asm_final:
%{gsplit-dwarf:
       objcopy --extract-dwo 	 %{c:%{o*:%*}%{!o*:%b%O}}%{!c:%U%O} 	 %{c:%{o*:%:replace-extension(%{o*:%*} .dwo)}%{!o*:%b.dwo}}%{!c:%b.dwo}
       objcopy --strip-dwo 	 %{c:%{o*:%*}%{!o*:%b%O}}%{!c:%U%O}     }

*asm_options:
%{-target-help:%:print-asm-header()} %{v} %{w:-W} %{I*}  %{gz|gz=zlib:--compress-debug-sections=zlib} %{gz=none:--compress-debug-sections=none} %{gz=zlib-gnu:--compress-debug-sections=zlib-gnu} %a %Y %{c:%W{o*}%{!o*:-o %w%b%O}}%{!c:-o %d%w%u%O}

*invoke_as:
%{!fwpa*:   %{fcompare-debug=*|fdump-final-insns=*:%:compare-debug-dump-opt()}   %{!S:-o %|.s |
 as %(asm_options) %m.s %A }  }

*cpp:
%{pthread:-D_REENTRANT}

*cpp_options:
%(cpp_unique_options) %1 %{m*} %{std*&ansi&trigraphs} %{W*&pedantic*} %{w} %{f*} %{g*:%{%:debug-level-gt(0):%{g*} %{!fno-working-directory:-fworking-directory}}} %{O*} %{undef} %{save-temps*:-fpch-preprocess} %(distro_defaults)

*cpp_debug_options:
%{d*}

*cpp_unique_options:
%{!Q:-quiet} %{nostdinc*} %{C} %{CC} %{v} %@{I*&F*} %{P} %I %{MD:-MD %{!o:%b.d}%{o*:%.d%*}} %{MMD:-MMD %{!o:%b.d}%{o*:%.d%*}} %{M} %{MM} %{MF*} %{MG} %{MP} %{MQ*} %{MT*} %{!E:%{!M:%{!MM:%{!MT:%{!MQ:%{MD|MMD:%{o*:-MQ %*}}}}}}} %{remap} %{g3|ggdb3|gstabs3|gxcoff3|gvms3:-dD} %{!iplugindir*:%{fplugin*:%:find-plugindir()}} %{H} %C %{D*&U*&A*} %{i*} %Z %i %{E|M|MM:%W{o*}}

*trad_capable_cpp:
cc1 -E %{traditional|traditional-cpp:-traditional-cpp}

*cc1:
%{profile:-p}%{%:sanitize(address):-funwind-tables}

*cc1_options:
%{pg:%{fomit-frame-pointer:%e-pg and -fomit-frame-pointer are incompatible}} %{!iplugindir*:%{fplugin*:%:find-plugindir()}} %1 %{!Q:-quiet} %{!dumpbase:-dumpbase %B} %{d*} %{m*} %{aux-info*} %{fcompare-debug-second:%:compare-debug-auxbase-opt(%b)}  %{!fcompare-debug-second:%{c|S:%{o*:-auxbase-strip %*}%{!o*:-auxbase %b}}}%{!c:%{!S:-auxbase %b}}  %{g*} %{O*} %{W*&pedantic*} %{w} %{std*&ansi&trigraphs} %{v:-version} %{pg:-p} %{p} %{f*} %{undef} %{Qn:-fno-ident} %{Qy:} %{-help:--help} %{-target-help:--target-help} %{-version:--version} %{-help=*:--help=%*} %{!fsyntax-only:%{S:%W{o*}%{!o*:-o %b.s}}} %{fsyntax-only:-o %j} %{-param*} %{coverage:-fprofile-arcs -ftest-coverage} %{fprofile-arcs|fprofile-generate*|coverage:   %{!fprofile-update=single:     %{pthread:-fprofile-update=prefer-atomic}}}

*cc1plus:


*link_gcc_c_sequence:
%{static|static-pie:--start-group} %G %{!nolibc:%L}    %{static|static-pie:--end-group}%{!static:%{!static-pie:%G}}

*distro_defaults:
%{!fno-asynchronous-unwind-tables:-fasynchronous-unwind-tables} %{!fno-stack-protector:%{!fstack-protector-all:%{!ffreestanding:%{!nostdlib:%{!fstack-protector:-fstack-protector-strong}}}}} %{!Wformat:%{!Wformat=2:%{!Wformat=0:%{!Wall:-Wformat} %{!Wno-format-security:-Wformat-security}}}} %{!fno-stack-clash-protection:-fstack-clash-protection}

*link_ssp:
%{fstack-protector|fstack-protector-all|fstack-protector-strong|fstack-protector-explicit:}

*endfile:
%{Ofast|ffast-math|funsafe-math-optimizations:crtfastmath.o%s} %{fvtable-verify=none:%s;      fvtable-verify=preinit:vtv_end_preinit.o%s;      fvtable-verify=std:vtv_end.o%s}    %{static:crtend.o%s;      shared|static-pie|!no-pie:crtendS.o%s;      :crtend.o%s} crtn.o%s

*link:
%{!r:--build-id} %{!static|static-pie:--eh-frame-hdr} %{h*}		   --hash-style=gnu				   --as-needed				   %{static:-Bstatic}				   %{shared:-shared}		   %{symbolic:-Bsymbolic}			   %{!static:%{!static-pie:	     %{rdynamic:-export-dynamic}		     %{!shared:-dynamic-linker %{muclibc:/lib/ld-uClibc.so.0;:%{mbionic:/system/bin/linker;:%{mmusl:/lib/ld-musl-aarch64%{mbig-endian:_be}%{mabi=ilp32:_ilp32}.so.1;:/lib/ld-linux-aarch64%{mbig-endian:_be}%{mabi=ilp32:_ilp32}.so.1}}}}}}    %{static-pie:-Bstatic -pie --no-dynamic-linker -z text}    -X						   %{mbig-endian:-EB} %{mlittle-endian:-EL}        -maarch64linux%{mabi=ilp32:32}%{mbig-endian:b} %{mfix-cortex-a53-835769:--fix-cortex-a53-835769} %{!mno-fix-cortex-a53-843419:--fix-cortex-a53-843419}

*lib:
%{pthread:-lpthread} %{shared:-lc}    %{!shared:%{profile:-lc_p}%{!profile:-lc}}

*link_gomp:


*libgcc:
%{static|static-libgcc|static-pie:-lgcc -lgcc_eh}%{!static:%{!static-libgcc:%{!static-pie:%{!shared-libgcc:-lgcc --push-state --as-needed -lgcc_s --pop-state}%{shared-libgcc:-lgcc_s%{!shared: -lgcc}}}}}

*startfile:
%{shared:;      pg|p|profile:%{static-pie:grcrt1.o%s;:gcrt1.o%s};      static:crt1.o%s;      static-pie:rcrt1.o%s;      !no-pie:Scrt1.o%s;      :crt1.o%s} crti.o%s    %{static:crtbeginT.o%s;      shared|static-pie|!no-pie:crtbeginS.o%s;      :crtbegin.o%s}    %{fvtable-verify=none:%s;      fvtable-verify=preinit:vtv_start_preinit.o%s;      fvtable-verify=std:vtv_start.o%s}

*cross_compile:
1

*version:
9.3.0

*multilib:
. !mabi=lp64;lp64:../lib:aarch64-linux-gnu mabi=lp64;

*multilib_defaults:
mabi=lp64

*multilib_extra:


*multilib_matches:
mabi=lp64 mabi=lp64;

*multilib_exclusions:


*multilib_options:
mabi=lp64

*multilib_reuse:


*linker:
collect2

*linker_plugin_file:


*lto_wrapper:


*lto_gcc:


*post_link:


*link_libgcc:
%D

*md_exec_prefix:


*md_startfile_prefix:


*md_startfile_prefix_1:


*startfile_prefix_spec:


*sysroot_spec:
--sysroot=%R

*sysroot_suffix_spec:


*sysroot_hdrs_suffix_spec:


*self_spec:


*asm_cpu_spec:
 %{mcpu=*:-march=%:rewrite_mcpu(%{mcpu=*:%*})}

*link_command:
%{!fsyntax-only:%{!c:%{!M:%{!MM:%{!E:%{!S:    %(linker) %{!fno-use-linker-plugin:%{!fno-lto:     -plugin %(linker_plugin_file)     -plugin-opt=%(lto_wrapper)     -plugin-opt=-fresolution=%u.res     %{flinker-output=*:-plugin-opt=-linker-output-known}     %{!nostdlib:%{!nodefaultlibs:%:pass-through-libs(%(link_gcc_c_sequence))}}     }}%{flto|flto=*:%<fcompare-debug*}     %{flto} %{fno-lto} %{flto=*} %l %{static|shared|r:;!no-pie:-pie -z now} %{fuse-ld=*:-fuse-ld=%*}  %{gz|gz=zlib:--compress-debug-sections=zlib} %{gz=none:--compress-debug-sections=none} %{gz=zlib-gnu:--compress-debug-sections=zlib-gnu}  -z relro %X %{o*} %{e*} %{N} %{n} %{r}    %{s} %{t} %{u*} %{z} %{Z} %{!nostdlib:%{!r:%{!nostartfiles:%S}}}     %{static|no-pie|static-pie:} %@{L*} %(mfwrap) %(link_libgcc) %{fvtable-verify=none:} %{fvtable-verify=std:   %e-fvtable-verify=std is not supported in this configuration} %{fvtable-verify=preinit:   %e-fvtable-verify=preinit is not supported in this configuration} %{!nostdlib:%{!r:%{!nodefaultlibs:%{%:sanitize(address):%{!shared:libasan_preinit%O%s} %{static-libasan:%{!shared:-Bstatic --whole-archive -lasan --no-whole-archive -Bdynamic}}%{!static-libasan:%{!fuse-ld=gold:--push-state} --no-as-needed -lasan %{fuse-ld=gold:--as-needed;:--pop-state}}}     %{%:sanitize(thread):%{!shared:libtsan_preinit%O%s} %{static-libtsan:%{!shared:-Bstatic --whole-archive -ltsan --no-whole-archive -Bdynamic}}%{!static-libtsan:%{!fuse-ld=gold:--push-state} --no-as-needed -ltsan %{fuse-ld=gold:--as-needed;:--pop-state}}}     %{%:sanitize(leak):%{!shared:liblsan_preinit%O%s} %{static-liblsan:%{!shared:-Bstatic --whole-archive -llsan --no-whole-archive -Bdynamic}}%{!static-liblsan:%{!fuse-ld=gold:--push-state} --no-as-needed -llsan %{fuse-ld=gold:--as-needed;:--pop-state}}}}}} %o      %{fopenacc|fopenmp|%:gt(%{ftree-parallelize-loops=*:%*} 1):	%:include(libgomp.spec)%(link_gomp)}    %{fgnu-tm:%:include(libitm.spec)%(link_itm)}    %(mflib)  %{fsplit-stack: --wrap=pthread_create}    %{fprofile-arcs|fprofile-generate*|coverage:-lgcov} %{!nostdlib:%{!r:%{!nodefaultlibs:%{%:sanitize(address): %{static-libasan|static:%:include(libsanitizer.spec)%(link_libasan)}    %{static:%ecannot specify -static with -fsanitize=address}}    %{%:sanitize(thread): %{static-libtsan|static:%:include(libsanitizer.spec)%(link_libtsan)}    %{static:%ecannot specify -static with -fsanitize=thread}}    %{%:sanitize(undefined):%{static-libubsan:-Bstatic} %{!static-libubsan:--push-state --no-as-needed} -lubsan  %{static-libubsan:-Bdynamic} %{!static-libubsan:--pop-state} %{static-libubsan|static:%:include(libsanitizer.spec)%(link_libubsan)}}    %{%:sanitize(leak): %{static-liblsan|static:%:include(libsanitizer.spec)%(link_liblsan)}}}}}     %{!nostdlib:%{!r:%{!nodefaultlibs:%(link_ssp) %(link_gcc_c_sequence)}}}    %{!nostdlib:%{!r:%{!nostartfiles:%E}}} %{T*}
%(post_link) }}}}}}
```
