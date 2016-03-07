---
title: TA-LibをJavaで使ってみる
date: 2016-03-07 22:24:43
tags: Java
categories: Technology
---
# TA-Lib
　皆さん、TA-Libをご存知でしょうか。
　TA-Libというのは、[こちら](http://ta-lib.org/)のサイトにあるテクニカル分析をするためのライブラリです。
　今回はJavaで書きましたが、他言語もサポートされ、最新バージョンとしては0.4となっています。
　こちらのライブラリは、1999年にOSS化して2007年に最終タグが付けられた状態となっております。
　それでは、早速コードの方を紹介していきます。

## サポート言語

ライブラリの実体は[こちら](http://sourceforge.net/p/ta-lib/code/HEAD/tree/tags/)に置かれています。

* C/C++
* .NET
* Perl
* Python

<a href="http://px.a8.net/svt/ejp?a8mat=2I0Y1E+D7PD4I+3C9Q+60H7L" target="_blank">
<img border="0" width="300" height="250" alt="" src="http://www24.a8.net/svt/bgt?aid=151209554799&wid=001&eno=01&mid=s00000015587001010000&mc=1"></a>
<img border="0" width="1" height="1" src="http://www13.a8.net/0.gif?a8mat=2I0Y1E+D7PD4I+3C9Q+60H7L" alt="">

## サンプルコード

### 環境
* Java 1.8.x
* gradle 2.x

### build.gradle
最終バージョンを依存関係に追加します。

``` gradle
dependencies {
    compile 'com.tictactec:ta-lib:0.4.0'
}
```

### TaLibWrapper.java
main関数に処理を実装しています。

``` java
package taLibWrapper;

import com.tictactec.ta.lib.Core;
import com.tictactec.ta.lib.MInteger;
import com.tictactec.ta.lib.RetCode;

public class TaLibWrapper {
    private static final Core C = new Core();

    private static MInteger __outBegIdx = new MInteger();
    private static MInteger __outNBElement = new MInteger();

    private static class StandardVal {
        private static final int START_IDX = 0;
        private static final int END_IDX = 399;
        private static final double[] IN_REAL = new double[400];
        private static final int OPT_IN_TIME_PERIOD = 30;
        private static final double[] OUT_REAL = new double[400];
    }

    private static class AdvancedVal {
        private static final int OPT_IN_FAST_PERIOD = 100;
        private static final int OPT_IN_SLOW_PERIOD = 100;
        private static final int OPT_IN_SIGNAL_PERIOD = 100;
        private static final int OPT_IN_TIME_PERIOD = 100;
        private static final double[] OUT_MACD = new double[400];
        private static final double[] OUT_MACD_SIGNAL = new double[400];
        private static final double[] OUT_MACD_HIST = new double[400];
    }

    public static void main(String[] args) {
        execute(() -> {

            // 単純移動平均線(SMA)を出す
            return C.sma(
                    StandardVal.START_IDX,
                    StandardVal.END_IDX,
                    StandardVal.IN_REAL,
                    StandardVal.OPT_IN_TIME_PERIOD,
                    __outBegIdx,
                    __outNBElement,
                    StandardVal.OUT_REAL);
        });
        execute(() -> {

            // 相対力指数(RSI)を出す
            return C.rsi(
                    StandardVal.START_IDX,
                    StandardVal.END_IDX,
                    StandardVal.IN_REAL,
                    StandardVal.OPT_IN_TIME_PERIOD,
                    __outBegIdx,
                    __outNBElement,
                    StandardVal.OUT_REAL);
        });
        execute(() -> {

            // 移動平均収束拡散手法(MACD)を出す
            return C.macd(
                    StandardVal.START_IDX,
                    StandardVal.END_IDX,
                    StandardVal.IN_REAL,
                    AdvancedVal.OPT_IN_FAST_PERIOD,
                    AdvancedVal.OPT_IN_SLOW_PERIOD,
                    AdvancedVal.OPT_IN_SIGNAL_PERIOD,
                    __outBegIdx,
                    __outNBElement,
                    AdvancedVal.OUT_MACD,
                    AdvancedVal.OUT_MACD_SIGNAL,
                    AdvancedVal.OUT_MACD_HIST);
        });
        execute(() -> {

            // 移動平均線を出す
            return C.movingAverage(
                    StandardVal.START_IDX,
                    StandardVal.END_IDX,
                    StandardVal.IN_REAL,
                    AdvancedVal.OPT_IN_TIME_PERIOD,
                    MAType.Kama,
                    __outBegIdx,
                    __outNBElement,
                    StandardVal.OUT_REAL);
        });
    }

    // 成功時のみ、出力させる
    private static void execute(TaLib t) {
        if (RetCode.Success == t.run()) {
            System.out.println(__outBegIdx.value);
            System.out.println(__outNBElement.value);
        }
    }

    @FunctionalInterface
    private interface TaLib {
        RetCode run();
    }

    private TaLibWrapper() throws AssertionError {
        new AssertionError("cannot create instance.");
    }
}
```

### 使ってみて

 このままスグに自分のサービスやビジネスに組み込むことはできません。  
 そもそもデータというものは最近はマッシュアップさせるのが定石ですので、こんな算出が簡単にできるライブラリがJavaでできるということを知っているだけでも良いと思います。  
 CSVなどを食わせて連続で処理させるだけなら、サポートも多いし、他のライブラリと組み合わせるのも有りだと思います。  


#### 最後に、話はずれますが
　Javaがまだ慣れないという方がいたら  
　こんな参考書を使って見たらいかがでしょうか。

##### Java8も踏まえて学びたい方はこちら

<a rel="nofollow" href="http://www.amazon.co.jp/gp/product/4774166855/ref=as_li_ss_il?ie=UTF8&camp=247&creative=7399&creativeASIN=4774166855&linkCode=as2&tag=slicascript-22"><img border="0" src="http://ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4774166855&Format=_SL250_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=slicascript-22" ></a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=slicascript-22&l=as2&o=9&a=4774166855" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

##### JavaEE7を学びたい方はこちら

<a rel="nofollow" href="http://www.amazon.co.jp/gp/product/4798140929/ref=as_li_ss_il?ie=UTF8&camp=247&creative=7399&creativeASIN=4798140929&linkCode=as2&tag=slicascript-22"><img border="0" src="http://ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4798140929&Format=_SL250_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=slicascript-22" ></a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=slicascript-22&l=as2&o=9&a=4798140929" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

　[TIOBEの2015年度版](http://www.tiobe.com/index.php/content/paperinfo/tpci/index.html)を見てもわかるように、Javaの人気は今年TOPであり、全体の20%を上回っています。
　現在フリーとして働いている僕の肌感ですが、今年もまたJava案件が多いですし、Javaを得意言語の一つにしておくのはキャリア的にありだと思います。
