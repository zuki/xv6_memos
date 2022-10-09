# Raspberry PiファームウェアWikiへようこそ

このWikiはRaspberry Piのすべてのモデルに搭載されているVideo GPUで
使用されているファームウェアを対象としています。

ビデオコアのファームウェアはクローズドソースであるので、このリポジトリ
にはソースコードが公開されていないことに注意してください。コード
セクションにはコンパイル済みのバイナリしかありません。

## 概要

このWikiではARMで動作するカーネル（または「ベアメタル」）コードの開発者の
視点からRaspberry Piのファームウェアのインターフェースについて説明します。

## メールボックス

メールボックスはARMとGPU上で動作するVideoCoreファームウェアとの間の
主要な通信手段です。利用可能なメールボックスの一覧は[こちら](mailbox.md)を参照してください。