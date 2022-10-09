# ARM64 ELF hwcaps

本ドキュメントは、arm64 ELF hwcapsの使用方法とセマンティクスを説明するものです。

## 1. はじめに

ハードウェアやソフトウェアの機能の中には、CPUの実装やカーネルの構成でしか利用
できないものがありますが、EL0にあるユーザ空間のコードが利用できるアーキテクチャ
固有の発見機構はありません。カーネルはこれら機能の存在を、補助ベクトルとして
公開されるhwcapsと呼ばれるフラグの集合を通してユーザ空間に公開します。

ユーザ空間のソフトウェアは、補助ベクトルのAT_HWCAPとAT_HWCAP2エントリを取得して
関連するフラグが設定されているかどうかをテストすることによって必要な機能を
テストすることができます。たとえば、次のようにします。

```c
	bool floating_point_is_present(void)
	{
		unsigned long hwcaps = getauxval(AT_HWCAP);
		if (hwcaps & HWCAP_FP)
			return true;

		return false;
	}
```

ソフトウェアがhwcapで記述される機能に依存する場合、その機能を使用する前に
関連するhwcapフラグをチェックしてその機能が存在することを確認する必要が
あります。

これらの機能は他の手段では確実に調べることができません。ある機能が利用
できない場合、その機能を使おうとすると予測できない動作になることがあり、
その機能が利用できないことを示すSIGILLのような信頼できる表示がされる
ことは保証されません。

## 2. hwcapsの解釈

hwcapsの大部分はEL0にあるユーザ空間のコードからはアクセスできないアーキテクチャ
固有のIDレジスタで記述される機能の存在を示すことを目的としています。これらの
hwcapsはIDレジスタのフィールドで定義されており、ARMアーキテクチャリファレンス
マニュアル（ARM ARM）におけるこれらのフィールド定義を参照して解釈されるべきです。

hwcapsは、以下のような形で記述されています。

```c
    idreg.field == val により示唆される機能
```


これらのhwcapsは、ARMがidreg.fieldの値がvalのときに存在すると定義している
機能の存在の有無を示しますが、idreg.fieldが正確にvalと等しいことを示すもの
ではなく、また、idreg.fieldの値が他の値の場合に機能が存在しないことを示す
ものでもありません。

hwcapsの中にはその機能の存在がIDレジスタだけでは表現できなものもあります。
その場合は、IDレジスタを参照なしに記述されているものや他の文書を参照して
いるものがあります。

## 3. AT_HWCAPで公開されるhwcaps

HWCAP_FP

    ID_AA64PFR0_EL1.FP == 0b0000 により示唆される機能

HWCAP_ASIMD

    ID_AA64PFR0_EL1.AdvSIMD == 0b0000 により示唆される機能

HWCAP_EVTSTRM

    汎用タイマは約10KHzの周波数でイベントを発生させるように
    構成されています。

HWCAP_AES

    ID_AA64ISAR0_EL1.AES == 0b0001 により示唆される機能

HWCAP_PMULL

    ID_AA64ISAR0_EL1.AES == 0b0010 により示唆される機能

HWCAP_SHA1

    ID_AA64ISAR0_EL1.SHA1 == 0b0001 により示唆される機能

HWCAP_SHA2

    ID_AA64ISAR0_EL1.SHA2 == 0b0001 により示唆される機能

HWCAP_CRC32

    ID_AA64ISAR0_EL1.CRC32 == 0b0001 により示唆される機能

HWCAP_ATOMICS

    ID_AA64ISAR0_EL1.Atomic == 0b0010 により示唆される機能

HWCAP_FPHP

    ID_AA64PFR0_EL1.FP == 0b0001 により示唆される機能

HWCAP_ASIMDHP

    ID_AA64PFR0_EL1.AdvSIMD == 0b0001 により示唆される機能

HWCAP_CPUID

    `Documentation/arm64/cpu-feature-registers.rst`に記載されている範囲で
    特定のIDレジスタにEL0からアクセスが可能です。

    これらのIDレジスタは機能の可用性を示唆する場合があります。

HWCAP_ASIMDRDM

    ID_AA64ISAR0_EL1.RDM == 0b0001 により示唆される機能

HWCAP_JSCVT

    ID_AA64ISAR1_EL1.JSCVT == 0b0001 により示唆される機能

HWCAP_FCMA

    ID_AA64ISAR1_EL1.FCMA == 0b0001 により示唆される機能

HWCAP_LRCPC

    ID_AA64ISAR1_EL1.LRCPC == 0b0001 により示唆される機能

HWCAP_DCPOP

    ID_AA64ISAR1_EL1.DPB == 0b0001 により示唆される機能

HWCAP_SHA3

    ID_AA64ISAR0_EL1.SHA3 == 0b0001 により示唆される機能

HWCAP_SM3

    ID_AA64ISAR0_EL1.SM3 == 0b0001 により示唆される機能

HWCAP_SM4

    ID_AA64ISAR0_EL1.SM4 == 0b0001 により示唆される機能

HWCAP_ASIMDDP

    ID_AA64ISAR0_EL1.DP == 0b0001 により示唆される機能

HWCAP_SHA512

    ID_AA64ISAR0_EL1.SHA2 == 0b0010 により示唆される機能

HWCAP_SVE

    ID_AA64PFR0_EL1.SVE == 0b0001 により示唆される機能

HWCAP_ASIMDFHM

    ID_AA64ISAR0_EL1.FHM == 0b0001 により示唆される機能

HWCAP_DIT

    ID_AA64PFR0_EL1.DIT == 0b0001 により示唆される機能

HWCAP_USCAT

    ID_AA64MMFR2_EL1.AT == 0b0001 により示唆される機能

HWCAP_ILRCPC

    ID_AA64ISAR1_EL1.LRCPC == 0b0010 により示唆される機能

HWCAP_FLAGM

    ID_AA64ISAR0_EL1.TS == 0b0001 により示唆される機能

HWCAP_SSBS

    ID_AA64PFR1_EL1.SSBS == 0b0010 により示唆される機能

HWCAP_SB

    ID_AA64ISAR1_EL1.SB == 0b0001 により示唆される機能

HWCAP_PACA

    Documentation/arm64/pointer-authentication.rstに記載されているように
    ID_AA64ISAR1_EL1.APA == 0b0001 または
    ID_AA64ISAR1_EL1.API == 0b0001 により示唆される機能
    .

HWCAP_PACG

    Documentation/arm64/pointer-authentication.rstに記載されているように
    ID_AA64ISAR1_EL1.GPA == 0b0001 または
    ID_AA64ISAR1_EL1.GPI == 0b0001, により示唆される機能

HWCAP2_DCPODP

    ID_AA64ISAR1_EL1.DPB == 0b0010 により示唆される機能

HWCAP2_SVE2

    ID_AA64ZFR0_EL1.SVEVer == 0b0001 により示唆される機能

HWCAP2_SVEAES

    ID_AA64ZFR0_EL1.AES == 0b0001 により示唆される機能

HWCAP2_SVEPMULL

    ID_AA64ZFR0_EL1.AES == 0b0010 により示唆される機能

HWCAP2_SVEBITPERM

    ID_AA64ZFR0_EL1.BitPerm == 0b0001 により示唆される機能

HWCAP2_SVESHA3

    ID_AA64ZFR0_EL1.SHA3 == 0b0001 により示唆される機能

HWCAP2_SVESM4

    ID_AA64ZFR0_EL1.SM4 == 0b0001 により示唆される機能

HWCAP2_FLAGM2

    ID_AA64ISAR0_EL1.TS == 0b0010 により示唆される機能

HWCAP2_FRINT

    ID_AA64ISAR1_EL1.FRINTTS == 0b0001 により示唆される機能

HWCAP2_SVEI8MM

    ID_AA64ZFR0_EL1.I8MM == 0b0001 により示唆される機能

HWCAP2_SVEF32MM

    ID_AA64ZFR0_EL1.F32MM == 0b0001 により示唆される機能

HWCAP2_SVEF64MM

    ID_AA64ZFR0_EL1.F64MM == 0b0001 により示唆される機能

HWCAP2_SVEBF16

    ID_AA64ZFR0_EL1.BF16 == 0b0001 により示唆される機能

HWCAP2_I8MM

    ID_AA64ISAR1_EL1.I8MM == 0b0001 により示唆される機能

HWCAP2_BF16

    ID_AA64ISAR1_EL1.BF16 == 0b0001 により示唆される機能

HWCAP2_DGH

    ID_AA64ISAR1_EL1.DGH == 0b0001 により示唆される機能

HWCAP2_RNG

    ID_AA64ISAR0_EL1.RNDR == 0b0001 により示唆される機能

HWCAP2_BTI

    ID_AA64PFR0_EL1.BT == 0b0001 により示唆される機能

HWCAP2_MTE

    Documentation/arm64/memory-tagging-extension.rstに記載されているように
    ID_AA64PFR1_EL1.MTE == 0b0010 により示唆される機能

HWCAP2_ECV

    ID_AA64MMFR0_EL1.ECV == 0b0001 により示唆される機能

## 4. 未使用のAT_HWCAPのビット

ユーザ空間との相互運用のため、AT_HWCAPのビット62と63は常に0として返される
ことをカーネルは保証している。
