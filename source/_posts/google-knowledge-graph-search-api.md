---
title: Google Knowledge Graph Search APIを使ってみた
date: 2016-03-07 22:58:06
tags: JavaScript
category: Technology
---
この記事は[Web API Advent calendar](http://qiita.com/advent-calendar/2015/web_api) 2015 の 20 日目です。

# 今回は２つの章に分かれています。
それぞれ必要な環境が違いますので注意してください。

## 1. Google Knowledge Graph Search APIを使うまで
[ドキュメント](https://developers.google.com/knowledge-graph/)はここ
このAPIを使うためには、Googleアカウントが必要です。

## 2. Node.jsで実装してみる
npm, Node.jsの環境が必要です。

### 以下バージョン
* npm v3.3.12
* Node.js v5.2.0
* Express v4.13.1
* Jade v1.11.0

## 1. Google Knowledge Graph Search API
これは何かと言うと、Googleの[ナレッジグラフ](https://www.google.com/intl/bn/insidesearch/features/search/knowledge.html)と言うものを検索するためのAPIになります。

昔[Freebase API](https://developers.google.com/freebase/)というものがありましたが、それは非推奨になっており、新しくは[Google Knowledge Graph Search API](https://developers.google.com/knowledge-graph/)を使うという話になってるみたいですね。

### Request
``` bash
https://kgsearch.googleapis.com/v1/entities:search
```

### Method
1系のメソッドは[以下](https://developers.google.com/knowledge-graph/reference/rest/v1/)になります

|パラメータ名|型|詳細|
|:--:|:--:|:--:|
|query|string|ナレッジグラフで検索するクエリ|
|ids|string|ナレッジグラフで検索するID|
|languages|string|言語コード|
|types|string|型指定|
|indent|boolean|JSON型ならtrue|
|prefix|boolean|部分一致用のプレフィクス|
|limit|number|リミット数|

### 手順

#### 1. Googleアプリにログイン
アプリを作成します

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.07.59.png" rel="attachment wp-att-222"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.07.59-300x84.png" alt="スクリーンショット 2015-12-19 23.07.59" width="600" height="168" class="alignnone size-medium wp-image-222" /></a>

#### 2. Google Knowledge Graph Search APIを有効にします
<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.14.10.png" rel="attachment wp-att-223"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.14.10-300x82.png" alt="スクリーンショット 2015-12-19 23.14.10" width="600" height="164" class="alignnone size-medium wp-image-223" /></a>

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.15.54.png" rel="attachment wp-att-224"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.15.54-300x130.png" alt="スクリーンショット 2015-12-19 23.15.54" width="600" height="260" class="alignnone size-medium wp-image-224" /></a>

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.16.40.png" rel="attachment wp-att-225"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.16.40-300x140.png" alt="スクリーンショット 2015-12-19 23.16.40" width="600" height="280" class="alignnone size-medium wp-image-225" /></a>

#### 3. APIキーを生成します。
<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.27.01.png" rel="attachment wp-att-227"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.27.01-300x96.png" alt="スクリーンショット 2015-12-19 23.27.01" width="600" height="192" class="alignnone size-medium wp-image-227" /></a>

#### 4. 以下のURLをブラウザから叩くか`curl`コマンドで見てみましょう
試しに、チューリング賞で有名なケン・トンプソンを調べてみましょうか。{API_KEY}には3. で生成したものを入れてください。

``` zsh
https://kgsearch.googleapis.com/v1/entities:search?query=Kenneth+Lane+Thompson&key={API_KEY}&limit=3&indent=True
```

こんな感じで出力されます。
<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.34.05.png" rel="attachment wp-att-229"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-19-23.34.05-1024x589.png" alt="スクリーンショット 2015-12-19 23.34.05" width="660" height="380" class="alignnone size-large wp-image-229" /></a>

<a href="http://px.a8.net/svt/ejp?a8mat=2I0Y1E+BZ1UR6+50+2HHNXT" target="_blank">
<img border="0" width="300" height="250" alt="" src="http://www23.a8.net/svt/bgt?aid=151209554724&wid=001&eno=01&mid=s00000000018015031000&mc=1"></a>
<img border="0" width="1" height="1" src="http://www14.a8.net/0.gif?a8mat=2I0Y1E+BZ1UR6+50+2HHNXT" alt="">

## 2. コーディング
Node.jsでコーディングしてみましょうか。
やることは、検索した人の名前と画像を表示するという単純なものにします。

### 1. Expressを使ってプロジェクトを作ります

``` zsh
// ジェネレータをインストール
npm install express-generator -g

// Expressを構築
express hack-api
cd hack-api
npm install
npm start
```

#### フロント側を作成
API_KEYには任意の値を入れてください

``` html
extends layout

block content
  h1= title
  p Welcome to #{title}

  img

  script(type='text/javascript').
    var service_url = 'https://kgsearch.googleapis.com/v1/entities:search';
    var params = {
      'query': 'Kenneth+Lane+Thompson',
      'limit': 3,
      'indent': true,
      'key': 'API_KEY'
    };
    $.getJSON(service_url + '?callback=?', params, function(response) {
      $.each(response.itemListElement, function(i, element) {
        $('<div>', {text:element['result']['name']}).appendTo(document.body);
        $('img:eq(0)').attr("src", element['result']['image']['contentUrl']);
      });
    });
```

#### 3000ポートにアクセスする
サーバサイドやその他に関しては初期のままで構いません

``` zsh
http://localhost:3000/
```

##### 以下のように出力されればOKです

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-20-0.22.53.png" rel="attachment wp-att-234"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-20-0.22.53-300x251.png" alt="スクリーンショット 2015-12-20 0.22.53" width="600" height="502" class="alignnone size-medium wp-image-234" /></a>

※ 画像は、https://en.wikipedia.org/wiki/Ken_Thompson を参照しています。

### GitHub
[GitHub](https://github.com/pollseed/Google-Knowledge-Graph-Search-api)に今回使ったソースを上げてあります。なお、API_KEYがハードコードされてますが、既に消去してますのでこのキーは変えてください。

## 以上
非常に簡単にAPIを使うことを確認しました。
是非やってみてください。

※当記事で問題がある箇所があれば仰ってください。削除します。

<a href="http://px.a8.net/svt/ejp?a8mat=2I0Y1E+BZ1UR6+50+2I0B8H" target="_blank">
<img border="0" width="468" height="60" alt="" src="http://www29.a8.net/svt/bgt?aid=151209554724&wid=001&eno=01&mid=s00000000018015118000&mc=1"></a>
<img border="0" width="1" height="1" src="http://www11.a8.net/0.gif?a8mat=2I0Y1E+BZ1UR6+50+2I0B8H" alt="">
