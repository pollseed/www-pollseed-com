---
title: MySQLのv5.1～v5.7.11へのバージョンアップ対応
date: 2016-03-07 23:19:48
tags: MySQL
categories: Technology
---
## ※注意点
* この記事は、公式のドキュメントを読むことに勝る内容は掲載されていません。
* 公式マニュアルを見ましょう。この記事のコマンドをコピペしても僕の環境でしかうまく動きません。
* 特に今回はデータベースという大事な資産を削除するので本当に気をつけてください。作業者以外責任を取れません。

　おかしな点があれば親切な方教えてください。大したことない人がやっているので間違いがあると思います。

# 実施環境(参考までに)
* CentOS6.7
* MySQL v5.1, MySQL v5.7

# 最初に
今回、バージョンアップ対応中にミスをしてやらかしてしまったので再発しないようにメモを残す

そのミスとは、互換性の確認をしていなかったこと。
本来なら、本番サーバ(v5.1)からデータ以外をダンプし、新サーバ(v5.7)にダンプファイルを流し、新サーバにて挙動を確認する
問題なければ、本番サーバからデータをダンプし、新サーバにダンプファイルを流し、アプリケーションとの接続(CRUD)を試して挙動を確認する。
こうすることで、何かエラーが発生したときの選択肢を一つ切ることができるのだが、新サーバを用意するのがめんどくさくてそのまま移行作業をしてしまった。

# 手順
サービス停止用の知らせを関係者にしておく。コールド状態で行うのでサービスを停止します。
Masterの対応後、Slaveを対応する。今回は、一方向Maste-Slave構成であり、データに差分が出ないやり方なので特別な対応はあまりしていません。

0. 前準備
1. バックアップ対応
2. MySQLサーバの停止
3. 古いMySQLパッケージのアンインストール
4. 新しいMySQLパッケージをインストール
5. 初期設定＆MySQLサーバの起動
6. バックアップデータの復帰
7. 後始末

<a href="http://px.a8.net/svt/ejp?a8mat=2I0Y1E+BZ1UR6+50+2HK0TD" target="_blank">
<img border="0" width="350" height="80" alt="" src="http://www28.a8.net/svt/bgt?aid=151209554724&wid=001&eno=01&mid=s00000000018015042000&mc=1"></a>
<img border="0" width="1" height="1" src="http://www15.a8.net/0.gif?a8mat=2I0Y1E+BZ1UR6+50+2HK0TD" alt="">

## 0. 前準備
まずは、現在バージョンの確認を行う
`mysql --version && ps aux | grep mysql & which mysql mysqld`

Webサーバを停止させデータのCLUDが走らないようにする。ログをモニタリングしながら作業にあたること。

## 1. バックアップ対応
MySQLのコマンドでダンプファイルを作ります
必ず、公式マニュアルを見てから対応しましょう。なぜならコマンドやオプションは、バージョンによって異なる可能性があるからです。

今回は、v5.1とv5.7のマニュアルを見ればよいので二つを目視しつつ作業にあたるのが望ましい。

* [http://dev.mysql.com/doc/refman/5.1/en/mysqldump.html](http://dev.mysql.com/doc/refman/5.1/en/mysqldump.html)
* [http://dev.mysql.com/doc/refman/5.7/en/mysqldump.html](http://dev.mysql.com/doc/refman/5.7/en/mysqldump.html)

**簡単に良く使うオプションを紹介しておきます。(公式に全て掲載されているものを情報落ちさせて載せています。)**

#### トランザクションオプションを利用しておくとパフォーマンスが落ちますが、データの信頼性と一貫性を担保できます。
理由もわからずに、オプションなしで発行するのは危険なのでここで整理しておきましょう。
--add-locks LOCK TABLES ステートメントと UNLOCK TABLES ステートメントで各テーブルダンプを囲むオプションで、INSERTの速度を向上させます。
--flush-privileges ダンプ完了後、出力にFLUSH PRIVILEGES ステートメントを追加する。正しいリストアのためには必須のオプションです。
--no-autocommit ダンプするテーブル毎にINSERT ステートメントを SET autocommit = 0 ステートメントと COMMIT ステートメントで囲みます。
--order-by-primary 行を主キーやインデクスでソートしてダンプするので便利だが、操作時間がかなり長くなる。
--single-transaction InnoDB などのトランザクションテーブルの場合にしか使えませんが、アプリケーションをブロックすることなく、START TRANSACTION が発行された時点のデータベースの一貫した状態をダンプします。
ただし、ALTER TABLE、CREATE TABLE、DROP TABLE、RENAME TABLE、TRUNCATE TABLE ステートメントを使用しないようにと書かれているので注意。

#### 次にパフォーマンスオプションですが、
INSERT文を速くするためには、いくつかやり方があると思います。
簡単な方法としては、VALUES文を複数にし、同時に複数行を挿入する、カラムの定義にDEFAULT値を設定するなどですね。
--delayed-insertは非推奨です。
--extended-insert, -e 先ほど行ったVALUES文の複数行挿入はこのオプションです。

#### フィルタリングオプションは
--all-databases, -A 全データベース内のすべてのテーブルをダンプする
--databases, -B 複数のデータベースをダンプする
--ignore-table=db_name.tbl_name 指定されたテーブルをダンプしない
--no-data, -d データが必要ないときに使います

### 今回はただコピーしたいだけなので
[http://dev.mysql.com/doc/refman/5.7/en/mysqldump-copying-database.html](http://dev.mysql.com/doc/refman/5.7/en/mysqldump-copying-database.html)
`mysqldump -u root -p >Dump20150216.sql`

### InnoDBファイルのバックアップ
```
mv /var/lib/mysql/ibdata1 ~/backup/ibdata1
mv /var/lib/mysql/.ibd ~/backup/.ibd
mv /var/lib/mysql/ib_logfile0  ~/backup/ib_logfile0
mv /var/lib/mysql/ib_logfile1  ~/backup/ib_logfile1
mv /etc/my.cnf ~/backup/my.cnf
```

## 2. MySQLサーバの停止
MySQLサーバを止めるのを忘れずに
```
/etc/init.d/mysqld status
/etc/init.d/mysqld stop
```

## 3. 古いMySQLパッケージのアンインストール
https://www.centosblog.com/11-useful-yum-commands-centos-linux/
まず、CentOS6系のマニュアルを見ましょう(eraseとremoveの違いがわからない方はマニュアルを一度読むことを推奨する)
```
# 古いものを全部削除する
yum remove mysql*
rm -rf /bin/mysql
rm -rf /usr/local/mysqld
```

## 4. 新しいMySQLパッケージをインストール
[https://dev.mysql.com/doc/mysql-repo-excerpt/5.7/en/linux-installation-yum-repo.html](https://dev.mysql.com/doc/mysql-repo-excerpt/5.7/en/linux-installation-yum-repo.html)
必ず、マニュアルに書いてある方法を試行しましょう。(以下のコマンドを打つ場合100%こけます)
```
sudo yum localinstall mysql57-community-release-el6-{version-number}.noarch.rpm
yum repolist enabled | grep mysql
sudo yum install mysql-community-server mysql-devel
mysql --version
which mysql mysqld
```

## 5. 初期設定＆MySQLサーバの起動
[https://dev.mysql.com/doc/mysql-repo-excerpt/5.7/en/linux-installation-yum-repo.html
](https://dev.mysql.com/doc/mysql-repo-excerpt/5.7/en/linux-installation-yum-repo.html
)
4と同じURLです。
```
# このコマンドによって、新しくテーブルが作られます。さらに、パスワードもこれによって設定されます。
sudo service mysqld start

sudo service mysqld status && ps aux | grep mysql

sudo grep 'temporary password' /var/log/mysqld.log
mysql -uroot -p # パスワードを入力する
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```
## 6. バックアップデータの復帰
[http://dev.mysql.com/doc/refman/5.7/en/reloading-sql-format-dumps.html](http://dev.mysql.com/doc/refman/5.7/en/reloading-sql-format-dumps.html)
マニュアルの方法を試しましょう。
`mysql -u root -p <Dump20150216.sql`

[https://dev.mysql.com/doc/refman/5.7/en/replication.html](https://dev.mysql.com/doc/refman/5.7/en/replication.html)
スレーブ(Read Only)にも同様に対応し、そのままダンプファイルを流します。(おそらくMasterにSlave用のユーザは存在しているはずなので)
``` zsh
mysql -u root -p
> start slave;
```

## 7. 後始末
ログにおかしなエラーが吐かれていないかは最低限確認する。
余力があれば、新規ユーザを作り、CREATE TABLE, INSERT, UPDATE, DELETE DROP TABLEが発行できるか確認する。
データを確認し、おかしなINSERTがされてないか確認する。パフォーマンスやメモリなどは保証されないやり方なのでそこは今回は無視する。
デフラグ。
アプリケーションを起動させ、プログラムからMySQLサーバに接続し挙動確認を行う。

## もしヤバイ状況になったら
[http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting.html](http://dev.mysql.com/doc/refman/5.7/en/innodb-troubleshooting.html)
下手にぐぐるよりも、トラブルシューティングのマニュアルを読むのが一番手っ取り早いです。
リカバリのやり方が効くかもしれないので。
