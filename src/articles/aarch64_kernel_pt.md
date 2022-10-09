# [AArch64 kernel Page Tables](https://wenboshen.org/posts/2018-09-09-page-table.html)

## ページテーブルフォーマット


[VMSAv8-64 translation table format descriptors](https://armv8-ref.codingbelief.com/en/chapter_d4/d43_1_vmsav8-64_translation_table_descriptor_formats.html)に
よれば、レベル0, 1, 2, 3の変換テーブルデスクリプタは次の通り。

![図D4-16](screens/pt_fig_d4_16.png)

![図D4-17](screens/pt_fig_d4_17.png)

AArch64では、1つのページテーブルエントリは64ビットです。上の図から、Linuxカーネルのページテーブルは
最大48ビットまで使用できることがわかります。AArch64 Linuxのメモリレイアウトについてはlinuxのカーネル
ソースの[Documentation/arm64/memory.txt](../../../source/linux/Documentation/arm64/memory.txt)
に次のように記載されています。

```
AArch64 Linux memory layout with 4KB pages + 3 levels:

Start			End			Size		Use
-----------------------------------------------------------------------
0000000000000000	0000007fffffffff	 512GB		user
ffffff8000000000	ffffffffffffffff	 512GB		kernel


AArch64 Linux memory layout with 4KB pages + 4 levels:

Start			End			Size		Use
-----------------------------------------------------------------------
0000000000000000	0000ffffffffffff	 256TB		user
ffff000000000000	ffffffffffffffff	 256TB		kernel


AArch64 Linux memory layout with 64KB pages + 2 levels:

Start			End			Size		Use
-----------------------------------------------------------------------
0000000000000000	000003ffffffffff	   4TB		user
fffffc0000000000	ffffffffffffffff	   4TB		kernel


AArch64 Linux memory layout with 64KB pages + 3 levels:

Start			End			Size		Use
-----------------------------------------------------------------------
0000000000000000	0000ffffffffffff	 256TB		user
ffff000000000000	ffffffffffffffff	 256TB		kernel
```

以下では、4KBページ＋3レベルを使用しています。各ページテーブルエントリは64ビットです。
ページフレームのサイズは4KBなので、各ページフレームには512エントリが含まれます。
3レベルの場合、合計で9+9+9+12=39ビットが使用されます。各レベル1のPGTエントリは2^30=1GBの
メモリをインデックスし、レベル2は2MBのメモリをインデックスし、レベル3は4KBのメモリを
インデックスします。

## カーネルページテーブルのダンプ

Linuxカーネルはカーネルページテーブルのダンプをサポートしています。arm64カーネルのコミットは
[arm64: add support to dump the kernel page tables](https://patchwork.kernel.org/patch/5323931/)
です。ソースコードは arch/arm64/mm/dump.c と arch/arm64/mm/ptdump_debugfs.c にあります。

```
// For Android common kernel v4.9.126
CONFIG_ARM64_PTDUMP=y

// For Android common kernel v4.14.68
CONFIG_ARM64_PTDUMP_CORE=y
CONFIG_ARM64_PTDUMP_DEBUGFS=y

mount -t debugfs nodev /sys/kernel/debug
cat /sys/kernel/debug/kernel_page_tables
```

# [D4.3.3 VMSAv8-64変換テーブルフォーマットデスクリプタのメモリ属性フィールド](https://armv8-ref.codingbelief.com/en/chapter_d4/d43_3_memory_attribute_fields_in_the_vmsav8-64_translation_table_formats_descriptors.html)

![ページデスクリプタの属性フィールド](screens/pt_fig_attrib.png)
