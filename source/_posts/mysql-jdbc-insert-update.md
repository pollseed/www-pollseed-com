---
title: 【MySQL】JDBCでINSERT ... ON DUPLICATE KEY UPDATE時の戻り値
date: 2016-03-07 23:12:23
tags: MySQL
categories: Technology
---
# INSERT ... ON DUPLICATE KEY UPDATE
JDBCから利用すると戻り値に違和感があった

ドキュメントは、[公式サイト](https://dev.mysql.com/doc/refman/5.6/ja/insert-on-duplicate.html)が最も参考になります。

今回は、この構文(`INSERT ... ON DUPLICATE KEYUPDATE`)をJDBCから利用した時の戻り値をまとめておくことにした。

## 気になった箇所
```
ON DUPLICATE KEY UPDATE を使用した場合、

行ごとの影響を受けた行の値は、
その行が新しい行として挿入された場合は 1、
既存の行が更新された場合は 2、
既存の行がその現在の値に設定された場合は 0 です。

mysqld への接続時に
CLIENT_FOUND_ROWS フラグを mysql_real_connect() に指定すると、
既存の行がその現在の値に設定された場合の影響を受けた行の値は
(0 ではなく) 1 になります。
```

## 簡単に実行してみる
挙動を確認することで、MySQLの構文仕様とJDBCの仕様を比較するのが早い

### DDL
id,job_code,store_idをPKとして、
そのキー制約を違反することで更新処理をかけるSQLを実行してその結果を確認することにした。
``` sql
-- DDL
CREATE TABLE `employees` (
   `id` int(11) NOT NULL,
   `fname` varchar(30) DEFAULT NULL,
   `lname` varchar(30) DEFAULT NULL,
   `hired` date NOT NULL DEFAULT '1970-01-01',
   `separated` date NOT NULL DEFAULT '9999-12-31',
   `job_code` int(11) NOT NULL,
   `store_id` int(11) NOT NULL,
   PRIMARY KEY (`id`,`job_code`,`store_id`)
 ) ENGINE=InnoDB DEFAULT CHARSET=utf8
 /*!50100 PARTITION BY RANGE (store_id)
 (PARTITION p0 VALUES LESS THAN (6) ENGINE = InnoDB,
  PARTITION p1 VALUES LESS THAN (11) ENGINE = InnoDB,
  PARTITION p2 VALUES LESS THAN (16) ENGINE = InnoDB,
  PARTITION p3 VALUES LESS THAN (21) ENGINE = InnoDB) */
```

### Wrapper
``` java
package test;

import static org.junit.Assert.assertEquals;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

import mockit.Expectations;
import mockit.integration.junit4.JMockit;

import org.junit.Test;

public class PreparedStatementTest {

    private int executeUpdate(String sql) throws SQLException {
        try (Connection conn =
                DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/test?user=root&password=password");
                PreparedStatement stmt = conn.prepareStatement(sql);) {
            return stmt.executeUpdate();
        } catch (SQLException e) {
            throw e;
        }
    }

    @Test
    public void test() throws SQLException {
        String sql =
                "insert into employees (id,job_code, store_id) values (7, 4, 19) on duplicate key update id = 7, job_code = 4, separated = now();";
        int executeUpdate1 = executeUpdate(sql);
        int executeUpdate2 = executeUpdate(sql);
        int executeUpdate3 = executeUpdate(sql);

        System.out.println("1回目 : " + executeUpdate1);
        System.out.println("2回目 : " + executeUpdate2);
        System.out.println("3回目 : " + executeUpdate3);

        assertEquals(executeUpdate1, 1);
        assertEquals(executeUpdate2, 2);
        assertEquals(executeUpdate3, 0);
    }
}
```

#### コンソール
思った想定値ではなかったようだ。

``` java
// 結果
1回目 : 1
2回目 : 2
3回目 : 1

java.lang.AssertionError: expected:<1> but was:<0>
    at test.PreparedStatementTest.test(PreparedStatementTest.java:44)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at java.lang.reflect.Method.invoke(Method.java:497)
    at org.eclipse.jdt.internal.junit4.runner.JUnit4TestReference.run(JUnit4TestReference.java:50)
    at org.eclipse.jdt.internal.junit.runner.TestExecution.run(TestExecution.java:38)
    at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:459)
    at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.runTests(RemoteTestRunner.java:675)
    at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.run(RemoteTestRunner.java:382)
    at org.eclipse.jdt.internal.junit.runner.RemoteTestRunner.main(RemoteTestRunner.java:192)
```

#### MySQLに直接クエリ投げる
念のため。

``` sql
1   47  16:30:46    insert into employees (id,job_code, store_id) values (7, 4, 19) on duplicate key update id = 7, job_code = 4, separated = now() 0 row(s) affected, 1 warning(s):
 1292 Incorrect date value: '2016-01-06 16:30:46' for column 'separated' at row 1   0.000 sec
```

## 結論

おそらくだが以下のリファレンスの話なのではないかと思う。

```
mysqld への接続時に
CLIENT_FOUND_ROWS フラグを mysql_real_connect() に指定すると、
既存の行がその現在の値に設定された場合の影響を受けた行の値は
(0 ではなく) 1 になります。
```

JDBCの中で`CLIENT_FOUND_ROWS`フラグを`mysql_real_connect()`に指定しているのではなかろうか。

### ここらへんのメソッドがきになる
``` java
checkErrorPacket(Buffer resultPacket)
appendDeadlockStatusInformation(String xOpen, StringBuilder errorBuf)
sqlQueryDirect(StatementImpl callingStatement, String query, String characterEncoding, Buffer queryPacket, int maxRows, int resultSetType, int resultSetConcurrency, boolean streamResults, String catalog, Field[] cachedMetadata)
readAllResults(StatementImpl callingStatement, int maxRows, int resultSetType, int resultSetConcurrency, boolean streamResults, String catalog, Buffer resultPacket, boolean isBinaryEncoded, long preSentColumnCount, Field[] metadataFromCache)
readResultsForQueryOrUpdate(StatementImpl callingStatement, int maxRows, int resultSetType, int resultSetConcurrency, boolean streamResults, String catalog, Buffer resultPacket, boolean isBinaryEncoded, long preSentColumnCount, Field[] metadataFromCache)
buildResultSetWithUpdates(StatementImpl callingStatement, Buffer resultPacket) (updateCountをセットしている)
```

## リファレンス
<a rel="nofollow" href="http://www.amazon.co.jp/gp/product/4774170208/ref=as_li_ss_il?ie=UTF8&camp=247&creative=7399&creativeASIN=4774170208&linkCode=as2&tag=slicascript-22"><img border="0" src="http://ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4774170208&Format=_SL250_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=slicascript-22" ></a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=slicascript-22&l=as2&o=9&a=4774170208" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
<a rel="nofollow" href="http://www.amazon.co.jp/gp/product/4774170208/ref=as_li_ss_tl?ie=UTF8&camp=247&creative=7399&creativeASIN=4774170208&linkCode=as2&tag=slicascript-22">MariaDB&MySQL全機能バイブル</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=slicascript-22&l=as2&o=9&a=4774170208" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

<a rel="nofollow" href="http://www.amazon.co.jp/gp/product/4774171972/ref=as_li_ss_il?ie=UTF8&camp=247&creative=7399&creativeASIN=4774171972&linkCode=as2&tag=slicascript-22"><img border="0" src="http://ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=4774171972&Format=_SL250_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=slicascript-22" ></a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=slicascript-22&l=as2&o=9&a=4774171972" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
<a rel="nofollow" href="http://www.amazon.co.jp/gp/product/4774171972/ref=as_li_ss_tl?ie=UTF8&camp=247&creative=7399&creativeASIN=4774171972&linkCode=as2&tag=slicascript-22">理論から学ぶデータベース実践入門 ~リレーショナルモデルによる効率的なSQL (WEB+DB PRESS plus)</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=slicascript-22&l=as2&o=9&a=4774171972" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
