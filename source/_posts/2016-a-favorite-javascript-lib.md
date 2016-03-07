---
title: 【必見】2016年から人気爆発JavaScriptライブラリ紹介
date: 2016-03-07 22:16:14
tags: JavaScript
categories: Technology
---
タイトルが凄い凄いオーバーな感じですが、早速紹介します。
　JSのライブラリを見ていて面白そうなものを恣意的に使ってみて、少しだけ紹介している記事となっています。濃い内容は書いてないです。
　新しい技術をちょっと動かして遊んでいるだけなので、ガチで使いたい時はドキュメントを読むだけでなくソース読まないと多分使えないですね。

## [Labella.js](http://twitter.github.io/labella.js/)

こちらのライブラリですが「[Twitter, Inc.](https://twitter.com/)」のリポジトリで開発されてリリースされています。となると当然、注目すべきライブラリの一つですよね。ちなみに開発された方は[kristw](https://twitter.com/kristw)という方みたいです。
既にファーストリリースから10日以上経ちます。
このライブラリは、なんとオーバラップさせずにラベルを自動的にインタラクティブに再配置してくれます。こんなこと自分でやろうとしたらものすごく大変ですよね。

### Install
labellaというより、グラフやSVG,図をよく一緒に使うと思うのでd3もpackage.jsonに追加しておきましょう。

``` zsh
npm install labella --save
npm install d3 --save
```

#### CDN
定番どころを読み込んでおく

* [Chart.js](https://cdnjs.cloudflare.com/ajax/libs/Chart.js/1.0.2/Chart.min.js)

### ドキュメント
* [Force](https://github.com/twitter/labella.js/blob/master/docs/Force.md)
* [Node](https://github.com/twitter/labella.js/blob/master/docs/Node.md)

### サンプルコーディング
<del>ちょっとTwitterのサンプルソースだとコンパイルエラーになるので修正します。</del>(既に修正版が出てました。)README.mdの保守って忘れがちですよね。僕も良く忘れちゃいます。

``` js
var nodes = [
  new labella.Node(1, 50),
  new labella.Node(2, 50),
  new labella.Node(3, 50),
  new labella.Node(3, 50),
  new labella.Node(3, 50),
];
var force = new labella.Force();
  force.nodes(nodes)
  .on('end', function(){
  draw(force.nodes());
  })
  .start(100);

var renderer = new labella.Renderer({
  layerGap: 60,
  nodeHeight: 12,
  direction: 'up'
});

// Node情報を元に、rect, pathタグを追加する
function draw(nodes){
  renderer.layout(nodes);

  d3.selectAll('rect.label')
    .data(nodes)
    .enter().append('rect')
    .classed('label', true)
    .attr('x', function(d){ return d.x - d.dx/2; })
    .attr('y', function(d){ return d.y; })
    .attr('width', function(d){ return d.dx; })
    .attr('height', function(d){ return d.dy; });

  d3.selectAll('path.link')
    .data(nodes)
    .enter().append('path')
    .classed('link', true)
    .attr('d', function(d){return renderer.generatePath(d);});
}
```

### スクショ
こんな形で出力されます。
<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-03-22.52.10.png"><img class="alignnone size-medium wp-image-61" src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-03-22.52.10-300x202.png" alt="スクリーンショット 2015-12-03 22.52.10" width="500" height="404" /></a>

<a href="http://px.a8.net/svt/ejp?a8mat=2I0Y1E+D7PD4I+3C9Q+60OXD" target="_blank">
<img border="0" width="320" height="50" alt="" src="http://www20.a8.net/svt/bgt?aid=151209554799&wid=001&eno=01&mid=s00000015587001011000&mc=1"></a>
<img border="0" width="1" height="1" src="http://www10.a8.net/0.gif?a8mat=2I0Y1E+D7PD4I+3C9Q+60OXD" alt="">

## [Matreshka.js Framework](http://matreshka.io/#!home)

　イベントドリブンモデルのJSフレームワーク。最近流行りのReact.jsのようにフロント側のJSフレームワークですので、使い方はシンプルです。
　ちなみにマトリョーシカというのは、ロシア製の人形で、人形が入れ子になって中にはより小さな人形が6層以上入っているようなものを指します。初音ミクとグミのボカロソングでもマトリョーシカっていうのがあるので、馴染みのある単語かと思います。

### Install

<pre>
npm install --save matreshka
</pre>

### Sample
クラス名my-input, my-outputを、キーxに`Hoge Fuga Piyo`と表示させる。中の文字を変更する度に`change:x`により、xが変更したことを感知してログを吐きます。
今回は、xのキーをmy-input, my-outputで同じにしているので、inputを変更すれば、outputも変更されます。

``` js
var app = new Matreshka();
  app.bindNode('x', '.my-input');
  app.x = 'Hoge Fuga Piyo';
  app.bindNode('x', '.my-output', {
  setValue: function(v) {
    this.innerHTML = v;
  }
});
app.on('change:x', function() {
  console.log('x changed to' + this.x);
});
```

### スクショ
inputボックスに入力をすることで、先の説明のようにログを吐きます。

<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-05-3.35.33.png"><img class="alignnone size-medium wp-image-61" src="http://php-aoringo.rhcloud.com/wp-content/uploads/2015/12/スクリーンショット-2015-12-05-3.35.33.png" alt="スクリーンショット 2015-12-03 22.52.10" width="500" height="404" /></a>
