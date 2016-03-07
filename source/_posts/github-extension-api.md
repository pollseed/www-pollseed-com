---
title: GitHubの拡張APIを作りました
date: 2016-03-07 23:21:39
tags: Sinatra
category: API
---
## リリース
2016年2月20日は、GitHubの拡張APIを作ったのでリリースしました。

使いたい方は、[Twitter Account](https://twitter.com/GEKIKAR90551875)に「GitHubの拡張APIが使いたい」とメッセージを下さい。パスやエンドポイント情報を教えます。(パス等は、変更することがあるので、その場合は[HOME](http://www.pollseed.com/)で公開しますので、事前にでもいいし、事後でもよいので、Twitterでメッセージをくれた方に再度通知します。)

現状(v1)ではBasicAuthで管理しています。今後は、OAuth認証に切り替えると思いますが、なにぶん遊びのサーバで動かしてるのでスペックが高くありません。

v2以降はもう少し使いやすくしようと思います。

<a href="http://px.a8.net/svt/ejp?a8mat=2I0Y1E+BZ1UR6+50+2HHNXT" target="_blank">
<img border="0" width="300" height="250" alt="" src="http://www21.a8.net/svt/bgt?aid=151209554724&wid=001&eno=01&mid=s00000000018015031000&mc=1"></a>
<img border="0" width="1" height="1" src="http://www11.a8.net/0.gif?a8mat=2I0Y1E+BZ1UR6+50+2HHNXT" alt="">

## ドキュメント
[http://docs.githubextensionapi.apiary.io](http://docs.githubextensionapi.apiary.io/#)
APIのリファレンスです。

## できること
最大1ヶ月前のGitHub上のデータをソートして50件まで返します。
単純に自分でデータ取得用のデータマイニング用に作ったのですが、暇だったのでAPIにした感じです。用途が用途なので、機能はしょぼいですね。

ソート条件は、以下の10個です
```
github_id・・・GitHub リポジトリ ID
stargazers_count・・・スター数
forks_count・・・フォーク数
commit_created_at・・・作成日時 New
commit_updated_at・・・更新日時
owner_id・・・開発者ID New
owner_followers・・・開発者のフォロワー数 New
owner_following・・・開発者のフォロー数 New
owner_created_at・・・開発者の登録日時 New
owner_updated_at・・・開発者の更新日時 New
```
Newになっているのが、今回の機能で追加したものです。
今後は、ページャ的な取得、開発期間やユーザのフレンド状況でのソート機能も入れようと思いますが、このあたりは時間があれば開発します。

何か不具合があれば、[GitHub Issue](https://github.com/pollseed/github-extension-api/issues)に優しく追加してくださると助かります。時間を見つけて直します。
いやいや簡単だからこっちで直したぜという凄腕の方がいればプルリクくださると喜びます。言語はRubyで作ってます。

[チャットルーム](https://gitter.im/pollseed/github-extension-api)はこちらです。

## デベロッパ
フォークは[Github Repo github-extension-api](https://github.com/pollseed/github-extension-api)からお願いします。
