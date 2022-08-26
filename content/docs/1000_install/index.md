---
title: 'インストール'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 1000
summary: Windows環境におけるMittsuのインストール方法を説明します。
---

## Rubyのインストール

まずはRuby本体が必須ですので、Ruby本体をインストールします。

※ インストーラーを使ってレジストリや環境変数を変更したくないという方は、
後述の「簡単な実行環境準備方法」がお勧めです（ただ、7zipの解凍ができる環境が必要ですが）。

インストーラーを使って導入したいという方は、まず[RubyInstaller](https://rubyinstaller.org/ "RubyInstallerへ移動")などから、
ビルド済みバイナリを入手するのが良いでしょう。

Ruby合宿2022では、Ruby 3.X系の最新バージョンである「3.1.2」を使用します。

アーキテクチャは32bit/64bitどちらでも構いませんが、今回は32bit版を使用したいと思います。

インストーラーを使ってインストールしても問題ありませんし、7zip版でインストーラーを使わずに導入しても問題ありません。

最終的に、コマンドプロンプトから「ruby -v」と打ち込んで、

```
ruby 3.1.2p20 (2022-04-12 revision 4491bb740a) [i386-mingw32]
```

のように表示できれば問題ありません。


## RubyGemsでMittsuをインストール

Ruby標準のパッケージ管理ツール「RubyGems」で、MittsuのライブラリをRuby実行環境にインストールします。

```
> gem install mittsu
```

これだけでは、OpenGLでの描画を担ってくれるdllが入らないので、
[こちら](https://www.glfw.org/download.html "GLFW公式へ移動")から、
GLFWのWindows用バイナリをダウンロードしましょう。

この時、インストールされたRubyのアーキテクチャと同じアーキテクチャのものを選ぶ
必要がありますので、今回の場合は「32-bit Windows binaries」を選択します。

そして、得られたアーカイブを展開し、導入済みのRubyのビルド環境に応じた「glfw3.dll」を探します。

今回は、「ruby -v」の結果から32bitのMinGW版Rubyとなりますので、「lib-mingw」
ディレクトリにある「glfw3.dll」が対象となります。

この「glfw3.dll」をパスの通ったディレクトリに配置すれば、導入完了です。

配置先はパスが通ってさえいればどこでも構いませんが、Rubyのbinディレクトリに入れてしまうのが
分かりやすいのではないかと思います。

## 環境変数「MITTSU_LIBGLFW_FILE」の設定

mittsuでは、環境変数「MITTSU_LIBGLFW_FILE」によって「glfw3.dll」の位置を認識します。

※ この環境変数名は将来的に変更されるかもしれません。

ユーザー環境変数／システム環境変数のどちらでも構いませんので、上記環境変数を登録しておいてください。

値は、先ほど「glfw3.dll」を配置した先の絶対パスになります。

例えば、「C:¥ruby\bin」に同dllを配置したのであれば、

```
MITTSU_LIBGLFW_FILE=C:\ruby\bin\glfw3.dll
```

となります。

## 簡単な実行環境準備方法

前述のRubyInstallerのサイトでは、Ruby実行環境一式を7zipによるアーカイブ版としても公開しています。

このアーカイブ版Rubyを使うと、インストーラーや環境変数の設定等無しに、
今回の実行環境を1つのディレクトリ内で完結するように作ることもできます。

※ 当然ながら、大前提として[7zip](https://www.7-zip.org/ "7zip公式へ移動")が事前にインストールされていなければなりませんが。

具体的には、まず任意のドライブの任意のパスに、好きな名前の空ディレクトリを作ります。

例えば、「D:\xxx\yyy\rubycamp」というディレクトリを作ったとしましょう。

この「rubycamp」ディレクトリが、実行環境一式になります。

環境が不要になったら、このディレクトリを丸ごと削除すれば、跡形もなくWindowsから消し去ることが可能です。

では、具体的な中身を作ります。

まず、[こちら](https://rubyinstaller.org/downloads/ "RubyInstaller公式へ移動")から7zip版Rubyをダウンロードします。

今回は「Other Useful Downloads」の「7-ZIP ARCHIVES」から、「Ruby 3.1.2-1 (x86) 」をダウンロードしましょう。

そして、ダウンロードした7zip形式（.7z）のアーカイブを展開し、その内容全てが先ほどの「rubycamp」ディレクトリ直下の「ruby」というディレクトリ下に収まるように配置してください。

具体的には以下のようになります。

```
D:\xxx\yyy
  ├─ rubycamp
  │     ├─ ruby
  │     │     ├─ bin
  │     │     ├─ include
  │     │     ├─ lib
          …
```

ついでに、作成したソースコードをまとめておく「src」フォルダも「rubycamp」直下に作っておくと便利です。

```
D:\xxx\yyy
  ├─ rubycamp
  │     ├─ ruby
  │     │     ├─ bin
  │     │     ├─ include
  │     │     ├─ lib
          …
  ├─ src
```

ここまで出来たら、後は「rubycamp」ディレクトリ直下に「cmd.bat」
（ファイル名部分は任意に変更しても問題ありません）というテキストファイルを作成します。

同テキストファイルをメモ帳等のエディタで開き、以下の内容を書き込んで保存します。

```
@echo off
set MITTSU_LIBGLFW_FILE=%~dp0ruby\bin\glfw3.dll
set PATH=%~dp0ruby\bin;%PATH%

cd %~dp0src
%COMSPEC%
```

この後は、作成した「cmd.bat」をエクスプローラーからダブルクリックすれば、
Ruby実行環境が整ったコマンドプロンプトが開きますので、後は上述の「RubyGemsでのインストール」から
作業を行ってMittsuのインストールとGLFWの導入を行ってください。
環境変数「MITTSU_LIBGLFW_FILE」の設定は、上記cmd.batに既に入っているので実施不要です。
