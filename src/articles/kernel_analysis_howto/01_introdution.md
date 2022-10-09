# 1. はじめに

## 1.1 はじめに

このHOWTOは、Linuxカーネルの各部がどのように動作するのか、どのようなメイン
関数とデータ構造が使用されているのか、どのように「車輪が回る」のかを定義
しようとするものです。この文書の最新版は http://www.bertolinux.com にあります。
この文書をより良くするための提案があれば次の私のアドレスまで送ってください:
berto@bertolinux.com。この文書で使用されているコードは、このHOWTOを書いている
時点の最新の安定版カーネルバージョンであるLinuxカーネルバージョン2.4.xのものです。

## 1.2 著作権

Copyright (C) 2000,2001,2002 Roberto Arcomano. This document is free; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 2 of the License, or (at your option) any later version. This document is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details. You can get a copy of the GNU GPL here

## 1.3 翻訳

この文章を翻訳したい場合は自由にすることができます。ただし、次の事を行う必要が
あるでしょう。

- この文書の他の版が既にあなたの国のLDPに存在しないかチェックする。
- 「はじめに」の（「はじめに」「著作権」「翻訳」「クレジット」を含む）すべての節を
  保持する。

Warning! You don't have to translate TXT or HTML file, you have to modify LYX file, so that it is possible to convert it all other formats (TXT, HTML, RIFF, etc.): to do that you can use "LyX" application you download from http://www.lyx.org.

No need to ask me to translate! You just have to let me know (if you want) about your translation.

Thank you for your translation!

## 1.4 クレジット

私の文書をすばやく公開およびアップロードしていただいたLinux Documentation Projectに
感謝いたします。

Klaas de Waalの提案に感謝いたします。
