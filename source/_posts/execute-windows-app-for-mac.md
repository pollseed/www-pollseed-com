---
title: Mac環境からWindowsアプリケーションを実行する
date: 2016-03-07 23:06:58
tags: Mac
category: Technology
---
## 初めに
今回、少しWindows用のアプリケーションを実行する機会が発生し、ただVagrantで起動するまでもなく簡単に実行したい。
そこで、Windowsアプリケーション実行用のコマンドをインストールしてみました。

## [brew](http://brew.sh/index_ja.html) をまだインストールしていない場合
``` zsh
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## 既にインストールしている場合
### こういうエラーが出た場合

``` zsh
curl: (22) The requested URL returned error: 410 Gone
Error: Failed to download resource "hogehoge"
```

パッケージの在り処が古いままだと、この後行うパッケージインストールでコケるのでbrew自体をupdateしましょう
``` zsh
brew update
```


## [Wine](https://www.winehq.org/)のインストール
Wineは、OSSです。
LinuxなどからWindows用アプリケーションをネイティブ動作させることができます。
```
brew install wine
```

### 使い方
wineコマンドを使ってWindows用のアプリケーションを実行させることができます。
```
wine <__アプリケーションの実行ファイルパス__>
```

<a href="http://px.a8.net/svt/ejp?a8mat=2I0Y1E+BZ1UR6+50+2HHNXT" target="_blank">
<img border="0" width="300" height="250" alt="" src="http://www28.a8.net/svt/bgt?aid=151209554724&wid=001&eno=01&mid=s00000000018015031000&mc=1"></a>
<img border="0" width="1" height="1" src="http://www16.a8.net/0.gif?a8mat=2I0Y1E+BZ1UR6+50+2HHNXT" alt="">

## さくらエディタを使ってみる
参考までにフリーのテキストエディタを使ってみましょう

[ホームページ](http://sourceforge.net/projects/sakura-editor/)にアクセスしてexeファイルをダウンロードしてきます。

「sakura_install2-2-0-1.exe」というファイルがダウンロードされたと思いますので、そのパスをコンソールに打ちましょう

``` zsh
wine /Users/pollseed/Downloads/sakura_install2-2-0-1.exe
```

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.46.31.png" rel="attachment wp-att-325"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.46.31.png" alt="スクリーンショット 2015-12-29 13.46.31" width="754" height="224" class="alignnone size-full wp-image-325" /></a>

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.46.49.png" rel="attachment wp-att-327"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.46.49.png" alt="スクリーンショット 2015-12-29 13.46.49" width="998" height="766" class="alignnone size-full wp-image-327" /></a>

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.47.00.png" rel="attachment wp-att-328"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.47.00.png" alt="スクリーンショット 2015-12-29 13.47.00" width="660" height="508" class="alignnone size-large wp-image-328" /></a>

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.47.18.png" rel="attachment wp-att-329"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.47.18.png" alt="スクリーンショット 2015-12-29 13.47.18" width="660" height="508" class="alignnone size-large wp-image-329" /></a>

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.47.30.png" rel="attachment wp-att-330"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.47.30.png" alt="スクリーンショット 2015-12-29 13.47.30" width="660" height="508" class="alignnone size-large wp-image-330" /></a>

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.47.44.png" rel="attachment wp-att-331"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.47.44.png" alt="スクリーンショット 2015-12-29 13.47.44" width="660" height="507" class="alignnone size-large wp-image-331" /></a>

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.48.02.png" rel="attachment wp-att-332"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.48.02.png" alt="スクリーンショット 2015-12-29 13.48.02" width="660" height="508" class="alignnone size-large wp-image-332" /></a>

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.48.13.png" rel="attachment wp-att-333"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.48.13.png" alt="スクリーンショット 2015-12-29 13.48.13" width="660" height="507" class="alignnone size-large wp-image-333" /></a>

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.48.25.png" rel="attachment wp-att-334"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.48.25.png" alt="スクリーンショット 2015-12-29 13.48.25" width="660" height="507" class="alignnone size-large wp-image-334" /></a>

### 起動する
僕と同じ手順でインストールしていたら以下のパスに居るはずなので、こいつを立ち上げます。

``` zsh
wine .wine/drive_c/Program\ Files/sakura/sakura.exe
```

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.54.18.png" rel="attachment wp-att-335"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-29-13.54.18-1024x604.png" alt="スクリーンショット 2015-12-29 13.54.18" width="660" height="389" class="alignnone size-large wp-image-335" /></a>

簡単なのでﾔｯﾃﾐﾃｸﾀﾞｻｲ
