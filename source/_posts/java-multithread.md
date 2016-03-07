---
title: Javaのマルチスレッド【クラス変数】と【インスタンス変数】の違い
date: 2016-03-07 23:04:10
tags: Java
categories: Technology
---
# この記事の対象者
* Java初心者プログラマ
* クラス変数とインスタンス変数の違いを説明出来ない方
* staticの付け方がわからない方

# 初めに
* Java・マルチスレッド環境で
* コードレビュー中に、とある新人さんの書いたコードを見ていた時に気になったこと
* クラス変数、インスタンス変数を勘違いしていた

# 環境
* Java 1.8.x

# クラス変数とは
JavaScriptが最近流行っているので、それで言うトコロのグローバル変数のイメージでいいかと思います。
[wikipedia](https://en.wikipedia.org/wiki/Class_variable)を読めば一発でわかりますので詳しいことは省略します。

### Javaの話に戻すと
`static`修飾子をつけるとクラス変数となります。

### 具体的な例ですと
MultiThreadStaticValTestクラスにstatc変数を定義します

``` java
class MultiThreadStaticValTest {
  static int __staticValue = 100;
}
```

MultiThreadStaticValTest_OuterCallクラスから先ほど定義したstatic変数を呼び出すことが可能です。
**要は、この変数は外部から変更されます。**

``` java
class MultiThreadStaticValTest_OuterCall {
  public static void main(String[] args) {
    System.out.println(MultiThreadStaticValTest.__staticValue);
  }
}
```

この変数をただ外部から守るだけなら、以下のように`__staticValue変数`の修飾子のスコープを狭めてあげればよいです。
※privateでも変更することはできますが、通常はそういう使い方はしません。

``` java
class MultiThreadStaticValTest {
  private static int __staticValue = 100;
}
```

### 今回は
マルチスレッドの場合にMultiThreadStaticValTestクラス自身が呼んだ場合に防げないということを説明します
なお、`__staticValue` は変更されうる値である

``` java
public class MultiThreadStaticValTest extends Thread {
    private static final int MAX_CNT = 5;
    private static int __staticValue = 0;

    private String threadName = null;

    /**
     * コンストラクタ.
     *
     * @param v
     *            クラス変数の初期値
     * @param value
     *            実行されるスレッド名
     */
    public MultiThreadStaticValTest(int v, String value) {
        threadName = value;
        __staticValue = v;
    }

    @Override
    public void run() {
        int[] idx = { 0 };
        IntStream.range(0, MAX_CNT).forEach(v -> {
            try {
                sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(threadName + idx[0]++ + " : " + --__staticValue);
        });
    }

    public static void main(String[] args) throws InterruptedException {

        // 1つ目のスレッドで初期値「1」を設定する
        MultiThreadStaticValTest t3 = new MultiThreadStaticValTest(1, "t3 : ");

        // 2つ目のスレッドで初期値「99」を設定する
        MultiThreadStaticValTest t4 = new MultiThreadStaticValTest(99, "t4 : ");
        t3.start();
        t4.start();
    }
}
```

### 出力例

```
t4 : 0 : 98
t3 : 0 : 97
t4 : 1 : 96
t3 : 1 : 95
t3 : 2 : 94
t4 : 2 : 93
...
...
...
```

### 問題点
スレッドを2つ作るも、後に初期化された値がstatic変数に保持され、さらに互いのstatic変数が共存されている点
普通に考えれば当たり前なのですが、このプログラムのようにstatic変数の使い方を間違えると大変なことになるので注意しましょう

<a href="http://px.a8.net/svt/ejp?a8mat=2I0Y1E+BZ1UR6+50+2I0B8H" target="_blank">
<img border="0" width="468" height="60" alt="" src="http://www25.a8.net/svt/bgt?aid=151209554724&wid=001&eno=01&mid=s00000000018015118000&mc=1"></a>
<img border="0" width="1" height="1" src="http://www12.a8.net/0.gif?a8mat=2I0Y1E+BZ1UR6+50+2I0B8H" alt="">

### 対処方ですが
とりあえずただのJavaプログラムであればInstance変数を使うという手段もあります。
※ただし、気を付けて欲しいのはServletを使っている方です。サーブレットはインスタンス変数をマルチスレッド間で共存して使いますので、思わぬ動きをすることがあります。

以下のプログラムを見てください。
※プログラムが汚いのは即興で作っているからで、そこは申し訳ありません

``` java
public class MultiThreadTest extends Thread {
    private static final int MAX_CNT = 3;

    private boolean isStatic = false;

    private int instanceValue = 0;
    private static int staticValue = 0;

    private String threadName = null;

    /**
     * コンストラクタ.
     *
     * @param v
     *            {@code bool} 値によってクラス・インスタンス変数を切り分ける
     * @param value
     *            実行されるスレッド名
     * @param bool
     *            {@code bool} 値によってクラス・インスタンス変数を切り分ける
     */
    public MultiThreadTest(int v, String value, boolean bool) {
        isStatic = bool;
        threadName = value;
        if (isStatic) {
            staticValue = v;
        } else {
            instanceValue = v;
        }
    }

    @Override
    public void run() {
        int[] idx = { 0 };
        IntStream.range(0, MAX_CNT).forEach(v -> {
            try {
                sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (isStatic) {
                System.out.println(threadName + idx[0]++ + " : " + --staticValue);
            } else {
                System.out.println(threadName + idx[0]++ + " : " + --instanceValue);
            }
        });
    }

    public static void main(String[] args) throws InterruptedException {

        // インスタンス変数の場合
        MultiThreadTest t1 = new MultiThreadTest(1, "t1 : ", false);
        MultiThreadTest t2 = new MultiThreadTest(99, "t2 : ", false);
        t1.start();
        t2.start();

        t1.join();
        t2.join();

        // クラス変数の場合
        MultiThreadTest t3 = new MultiThreadTest(1, "t3 : ", true);
        MultiThreadTest t4 = new MultiThreadTest(99, "t4 : ", true);
        t3.start();
        t4.start();
    }
}
```

### 出力例
t1とt2の変数値が共存されていないことがわかると思います。

```
t1 : 0 : 0
t2 : 0 : 98
t1 : 1 : -1
t2 : 1 : 97
t2 : 2 : 96
t1 : 2 : -2
t1 : 3 : -3
t2 : 3 : 95
t1 : 4 : -4
t2 : 4 : 94

t3 : 0 : 97
t4 : 0 : 98
t4 : 1 : 95
t3 : 1 : 96
t4 : 2 : 94
t3 : 2 : 93
t3 : 3 : 92
t4 : 3 : 91
t4 : 4 : 89
t3 : 4 : 90
```

### 変更しない値を使う場合は
定数にすれば問題ないです。

### GitHub
[こちら](https://github.com/pollseed/cure-code/blob/master/src/test/java/test/multithread/MultiThreadTest.java)にプログラムを置きました

# まとめ
* クラス変数はマルチスレッド間で値が共存されるので、それを期待しないのであればクラス変数を避ける
* インスタンス変数、クラス変数ともにサーブレットで使う時は設定を見直すか、そもそも共存されても問題ないのかを確認しましょう。サーブレットの場合通常は、1インスタンスに対してマルチスレッドでアクセスをします

#### 最後に、話はずれますが
　Javaがまだ慣れないという方がいたら  
　こんな参考書を使って見たらいかがでしょうか。

##### Java8も踏まえて学びたい方はこちら

<a rel="nofollow" href="http://www.amazon.co.jp/gp/product/4774166855/ref=as_li_ss_il?ie=UTF8&camp=247&creative=7399&creativeASIN=4774166855&linkCode=as2&tag=slicascript-22"><img border="0" src="http://ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4774166855&Format=_SL250_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=slicascript-22" ></a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=slicascript-22&l=as2&o=9&a=4774166855" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

##### JavaEE7を学びたい方はこちら

<a rel="nofollow" href="http://www.amazon.co.jp/gp/product/4798140929/ref=as_li_ss_il?ie=UTF8&camp=247&creative=7399&creativeASIN=4798140929&linkCode=as2&tag=slicascript-22"><img border="0" src="http://ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4798140929&Format=_SL250_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=slicascript-22" ></a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=slicascript-22&l=as2&o=9&a=4798140929" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
