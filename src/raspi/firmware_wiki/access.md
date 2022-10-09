# メールボックスのアクセス

## ドラフト

このページはこのテーマについて私より詳しい人が完成させるためのスタブです。

## 概要

メールボックスはARMとVideoCoreの間の通信を容易にします。このページでは
メールボックスにアクセスするための手順を説明します。利用可能なメール
ボックスの一覧は[こちら](mailbox.md)を参照してください。

## 一般的な手順

メールボックスから読み出す手順。

1. エンプティフラグが外れるまでステータスレジスタを読みます。
2. Readレジスタからデータを読みます。
3. 下位4ビットが希望するチャンネル番号と一致しない場合は1から繰り返します。
4. 上位28ビットが返されたデータです。

## メールボックスに書き込む手順。

1. フルフラグが外れるまでステータスレジスタを読みます。
2. データ（上位28ビットにシフト）とチャンネル（下位4ビット）を組み合わせて
   Writeレジスタに書き込みます。

## データとしてのアドレス

**属性タグメールボックスチャンネルを除き**、メールボックスメッセージの
データとしてメモリアドレスを渡す場合、アドレスはVCから見たバスアドレスで
なければなりません。このアドレスはL2キャッシュが有効か否かで変わります。
L2キャッシュが有効な場合、物理メモリはVC MMUにより0x40000000から始まる
ようにマッピングされます。L2キャッシュが無効な場合、物理メモリはVC MMUに
より0xC0000000から始まるようにマッピングされます。返されるアドレスは
（メールボックスレスポンスのデータ部として返されたものも、渡したバッファに
書き込まれたものも）VC MMUによってマップされたアドレスになります。属性タグ
メールボックスチャネルを使用した場合は例外で、物理アドレス（MMUを有効に
する前にARMから見たアドレス）を送受信する必要があります。

たとえば、（ARM MMU を有効にせずに）0x00010000 にあるメモリにフレーム
バッファ記述構造体を作成し、config.txtを変更せずL2キャッシュを無効にして
いない場合、チャネル1にこのバッファ構造体のアドレスを送信するにはメール
ボックスに0x40010001 (0x40000000 | 0x00010000 | 0x1) を送信することに
なります。この構造体は0x40000000から始まるフレームバッファアドレス（たとえば、0x4D385000）を含むように更新されるので、対応するARM物理アドレス（たとえば、0x0D385000）を使用して書き込むことになります。

## メモリバリアとデータキャッシュの無効化/フラッシュ

メールボックスのアクセス時には、メモリバリアやデータキャッシュの無効化/
フラッシュが必要な場合があります。これに関する詳細はその詳細を知っている
人がこのページ（またはそれを必要とする特定のメールボックス/チャネルの
ページ）に追加する必要があります。

以下の命令は[ARMのマニュアル](http://infocenter.arm.com/help/topic/com.arm.doc.ddi0360f/I1014942.html)
から引用したものです。以下の例ではr3の代わりに任意の不要なレジスタを
使用することができます。

```
mov r3, #0				        # 読み込みRegisterは呼び出し前に0クリアする必要がある
mcr p15, 0, r3, C7, C6, 0		# 全データキャッシュを無効にする
mcr p15, 0, r3, c7, c10, 0		# 全データキャッシュをクリーンにする
mcr p15, 0, r3, c7, c14, 0		# 全データキャッシュをクリーンにして無効にする
mcr p15, 0, r3, c7, c10, 4		# データ同期バリア
mcr p15, 0, r3, c7, c10, 5		# データメモリバリア
```

次の手続きは物理デバイスのアクセスで時々使用されます。

```
MemoryBarrier:
	mcr p15, 0, r3, c7, c5, 0	# 命令キャッシュを無効にする
	mcr p15, 0, r3, c7, c5, 6	# BTBを無効にする
	mcr p15, 0, r3, c7, c10, 4	# ライトバッファをフラッシュ
	mcr p15, 0, r3, c7, c5, 4	# フラッシュをプリフェッチ
	mov pc, lr					# 復帰
```

## サンプルコード

次のコードは未検証であり、簡潔さや効率よりも明快性さを重視して
書かれていますが参考のため提供します。

```c++
#define MAIL_BASE 0xB880	// メールボックスRegisterのベースアドレス

// このビットはメールボックスに書き込む余地がなくなった時に
// 状態Registerにセットされる
#define MAIL_FULL 0x80000000
// このビットはメールボックスから読み出すものがなにもない時に
// 状態Registerにセットされる
#define MAIL_EMPTY 0x40000000

uint32 ReadMailbox(byte channel)
{
	// 指定のチャンネルから何かを受信するまでループする
	for (;;)
	{
		while ((ReadMemMappedReg<uint32>(MAIL_BASE, ReadStatusOffset) & MAIL_EMPTY) != 0)
		{
			// データを待つ
		}
		// データを読む
		uint32 data = ReadMemMappedReg<uint32>(MAIL_BASE, ReadOffset);
		byte readChannel = data & 0xF;
		data >>= 4;
		// 指定のチャンネルならすぐに返す
		if (readChannel == channel)
			return data;
	}
}

void WriteMailbox(byte channel, uint32 data)
{
	while ((ReadMemMappedReg<uint32>(MAIL_BASE, WriteStatusOffset) & MAIL_FULL) != 0)
	{
		// スペースが空くのを待つ
	}
	// 指定のチャンネルに値を書く
	WriteMemMappedReg(MAIL_BASE, WriteOffset, (data << 4) | channel);
}

#define MAPPED_REGISTERS_BASE 0x20000000

template<class T>
static T ReadMemMappedReg(size_t BaseAddress, size_t Offset)
{
	return *reinterpret_cast<const T *>(MAPPED_REGISTERS_BASE + BaseAddress + Offset);
}

template<class T>
static void WriteMemMappedReg(size_t BaseAddress, size_t Offset, T Data)
{
	*reinterpret_cast<T *>(MAPPED_REGISTERS_BASE + BaseAddress + Offset) = Data;
}
```
