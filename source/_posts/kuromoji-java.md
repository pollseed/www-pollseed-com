---
title: KuromojiをJavaで使ってみる
date: 2016-03-07 22:27:53
tags: Java
categories: Technology
---
# [Kuromoji](http://www.atilika.com/ja/products/kuromoji.html)
* Javaで書かれているオープンソースの日本語形態素解析エンジン
* Javaで自然言語処理等をするならば非常に有用なライブラリです。

## 機能
* 複合語の分割 テキストを言葉に分割（形態素）
* 品詞のタグ付け 言葉分類の割当（名詞、動詞、助詞、形容詞など）
* 見出し化 活用の動詞や形容詞に辞書の見出しを表示
* 読み方 漢字の読み方を抽出

大学院でも最近自然言語処理を研究テーマとする人が多く、Javaを使うなら候補としては上がってくるものです。というわけで使ってみることにしました。

## 環境
Java 1.8.x
gradle 2.4.x
kuromoji 0.9.0

<a href="http://px.a8.net/svt/ejp?a8mat=2I0Y1E+D7PD4I+3C9Q+60H7L" target="_blank">
<img border="0" width="300" height="250" alt="" src="http://www22.a8.net/svt/bgt?aid=151209554799&wid=001&eno=01&mid=s00000015587001010000&mc=1"></a>
<img border="0" width="1" height="1" src="http://www13.a8.net/0.gif?a8mat=2I0Y1E+D7PD4I+3C9Q+60H7L" alt="">

## サンプルコード

### build.gradle

``` gradle
dependencies {
    compile 'com.atilika.kuromoji:kuromoji-jumandic:0.9.0'
    testCompile group: 'junit', name: 'junit-dep', version: '4.10'
}
```

### KuromojiWrapper.java
このクラスで実際にKuromojiを使っています。
Tokenizerクラスに与えられた引数を元に形態素解析を行っています。
難しい処理はしていません。素直にサンプルソースを書いただけですね。

``` java
package kuromojiWrapper;

import java.util.ArrayList;
import java.util.List;

import com.atilika.kuromoji.jumandic.Token;
import com.atilika.kuromoji.jumandic.Tokenizer;

public class KuromojiWrapper {
    public static final Tokenizer T = new Tokenizer();
    public static final KuromojiWrapper KW = new KuromojiWrapper();

    // 表層形式
    public List<String> surfaces(final String target) {
        final List<String> list = new ArrayList<String>();
        token(target).forEach(v -> list.add(v.getSurface()));
        return list;
    }

    // 品詞
    public List<String> allFeatures(final String target) {
        final List<String> list = new ArrayList<String>();
        token(target).forEach(v -> list.add(v.getAllFeatures()));
        return list;
    }

    // 単語のトークン
    public List<Token> token(final String target) {
        return T.tokenize(target);
    }
}
```

### KuromojiWrapperTest.java
テストクラスです。`KuromojiWrapper`を実際に使ってテストをしています。

``` java
package kuromojiWrapper;

import java.util.LinkedList;
import java.util.Queue;

import junit.framework.Assert;

import org.junit.Test;

import com.atilika.kuromoji.jumandic.Token;

public class KuromojiWrapperTest {
    static KuromojiWrapper KW = new KuromojiWrapper();

    @Test
    public void test_surfaces() {
        final Queue<String> surfacesTrue = new LinkedList<String>() {
            private static final long serialVersionUID = 1L;
            {
                add("ほ");
                add("げ");
                add("ほ");
                add("げ");
            }
        };
        KW.surfaces("ほげほげ").forEach(v -> Assert.assertEquals(v, surfacesTrue.poll()));
    }

    @Test
    public void test_features() {
        final Queue<Token> tokens = new LinkedList<Token>();
        KW.token("ほげほげ").forEach(v -> tokens.add(v));
        KW.allFeatures("ほげほげ").forEach(v -> Assert.assertEquals(v, tokens.poll().getAllFeatures()));
    }
}
```

<a href="http://px.a8.net/svt/ejp?a8mat=2I0Y1E+BZ1UR6+50+2HSDQP" target="_blank">
<img border="0" width="350" height="80" alt="" src="http://www20.a8.net/svt/bgt?aid=151209554724&wid=001&eno=01&mid=s00000000018015081000&mc=1"></a>
<img border="0" width="1" height="1" src="http://www12.a8.net/0.gif?a8mat=2I0Y1E+BZ1UR6+50+2HSDQP" alt="">
