---
title: 【初心者必見】ConoHaのVPSを使って一からセットアップしてみる
date: 2016-03-07 23:17:21
tags: Linux
category: Technology
---
今回、AWSだけではなく、国内のVPSも契約してみることにしました。その際に参考にした情報を残しておきます。

## はじめに
僕が初めてサーバ構築をした時、自宅鯖にLinuxとして新しくリリースされたUbuntuを入れてサーバ環境を整えました。当時は、技術も知識もあまりなく、結局完成まで1週間ぐらいかかりました。確かBINDの設定でどはまりして、大分死にそうになったような気がします。もうあまり覚えていません。Oreillyの本があればもっとスムーズだったかもしれませんが

## きっかけ
* Webサービスや簡単にクラウド上のLinux環境で遊ぶ時にVPSが必要になった
* AWSだけではなく、国内のVPSを使ってみようとオモタ

## 候補
* さくら
* ConoHa
* ServersMan

この3つが候補になったのは、お手軽さと値段・信頼性である。あまりにも不具合が多い会社は怖いので選ぶ気になれない。

今回は、ConoHaにしました。

紹介URLです。もし気が向いたら使ってみてください。こちらのホームページの読者の方は、このリンクから登録するとConoHaの1,000円分のクーポンがもらえます。
[クーポン利用の方はこちらをクリックして登録してください](https://www.conoha.jp/referral/?token=fmmHSnIh9GzQWYsEX7XR9H5snPtr8LfEykCnLDz0msp9hKBV5oc-GQN)

## OS
CentOS release 6.7 (Final) : 7系のブリッジサーバの構築で手間取った経験を思い出し、6系にした。今はまだ得意な方で良いと思います。僕はまだ7系のコマンドに不慣れなので、自宅鯖ぐらいしか冒険できないチキンです。

<a href="http://px.a8.net/svt/ejp?a8mat=2I0Y1E+BZ1UR6+50+2HMT4H" target="_blank">
<img border="0" width="300" height="250" alt="" src="http://www21.a8.net/svt/bgt?aid=151209554724&wid=001&eno=01&mid=s00000000018015055000&mc=1"></a>
<img border="0" width="1" height="1" src="http://www18.a8.net/0.gif?a8mat=2I0Y1E+BZ1UR6+50+2HMT4H" alt="">

## 最初にやっておきたいこと
めんどくさいけど、やってないと後で困るので

一応、rootのパスワードを20文字以上にする、かつ英数字を入れる。記号もいれると尚良い。ただ、忘れないように、忘れたらAHOなので注意

### ユーザ
``` zsh
$ adduser pollseed
$ passwd pollseed
$ gpasswd -a pollseed wheel

# vi /etc/sudoers で 以下のコメントアウト外す
%wheel  ALL=(ALL)       ALL
```

### SSH周り
``` zsh
$ mkdir /home/pollseed/.ssh
$ chown pollseed:pollseed /home/pollseed/.ssh
$ chmod 700 /home/pollseed/.ssh/
$ cp ~/.ssh/authorized_keys /home/pollseed/.ssh/
$ chown pollseed:pollseed authorized_keys

# vi /etc/ssh/sshd_config の内容を以下に変える
Port 22以外
PermitRootLogin no
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
PasswordAuthentication no

# /etc/sysconfig/iptables のポートを変更しておく
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22番以外 -j ACCEPT

$ service iptables restart
$ service sshd restart

$ ssh -i .ssh/id_rsa pollseed@XX.XX.XX.XX -p XX
$ sudo hostname conoha

$ mkdir -p ~/.vim/bundle
$ git clone git://github.com/Shougo/neobundle.vim ~/.vim/bundle/neobundle.vim
# .vimrcを作る

$ exit
```

やってることは単純で、以下の話。他にもごちゃごちゃやってはいるが、とりあえず最低限これぐらいはやっておきたい

* 指定ユーザ（Root禁止）
* パスワードログイン禁止（秘密鍵ログインのみ許可）
* 22番ポートでのログイン禁止

ちなみにIptablesの設定は[こちら](http://oxynotes.com/?p=6361)がわかりやすいと思います。僕もよく見ます。

### Package
簡単に
``` zsh
yum update -y
su pollseed
sudo yum git zsh gcc zlib zlib-devel openssl-devel sqlite-devel gcc-c++ glibc-headers libyaml-devel readline readline-devel zlib-devel redhat-lsb
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# nginx
sudo rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
sudo yum install -y nginx
```

※注意点
当たり前の話ですが、忘れがちなこととして、サーバのメモリが小さいとGccのコンパイルはかなり遅いです。

参考までに、1GでRubyのコンパイルをしてみた時のCPUの消費具合です
<a href="http://php-aoringo.rhcloud.com/wp-content/uploads/2016/02/スクリーンショット-2016-02-13-23.39.44.png" rel="attachment wp-att-555"><img src="http://php-aoringo.rhcloud.com/wp-content/uploads/2016/02/スクリーンショット-2016-02-13-23.39.44-1024x391.png" alt="スクリーンショット 2016-02-13 23.39.44" width="840" height="321" class="alignnone size-large wp-image-555" /></a>

という話ですね
