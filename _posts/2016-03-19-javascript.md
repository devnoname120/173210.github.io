---
layout: post
title:  "JavaScriptを使った開発で苛ついている話"
date:   2016-03-19 13:14:50 +0900
tags:   JavaScript
---

愚痴だ. 聞いてくれ. いや, 聞かなくてもいいが, 聞いて欲しい.

私は[_Yet Another Tweets Search_](https://github.com/173210/173210.github.io)という,
過去の自分のツイートを検索するChrome拡張を作っている. そこで私はWeb開発の世界の深遠なる闇を覗いているところである.

_Twitter_のAPIを利用するために, _sed_をプリプロセッサとして用いてcredentialsを埋め込み,
_UglifyJS2_でそれを難読化しようとした. 私はいつものように_Makefile_を書き,
何ということもなく普通に動いた.

ここで, ユーザーの_Twitter_ アカウントを知る必要があることに気づいた. _Twitter_に過去のツイートを請求すると,
ZIPファイルがダウンロードできるのだが, 今のところは, その中でもツイートの情報のみを含むCSVをユーザーが取り出して読み込ませることになっている.
ところで, ZIPファイル内にはCSV以外にもユーザーのその時点での情報が含まれるJSONもある.
そこで, _Chrome_拡張自体がZIPファイルを展開し, CSVとJSONの両方を同時に読み込ませることにした.

さて, _Firefox_にはZIPを展開するAPIがあるらしいのだが, _Chrome_にはない. 依存関係をなくすようにこれまでコードを書いていたが,
ついに依存関係ができてしまった. といっても1つだけだ. _JSZip_というのが人気らしいのでそれを使うことにした. (依存: 1個)
__これで終わるはずだった―__

_JSZip_のページに行くと, いろいろなパッケージ管理があるようだった. 単にファイルを直接ダウンロードするか,
_Git_のsubmoduleとしてダウンロードしてから目標のファイルにリンクを貼るぐらいしか能がなかった私には素晴らしいように見えた.
__そんなことはなかった__

そこには3つの選択肢があった. _npm_, _bower_, そして _component_だ. _npm_は簡単だが,
実行環境がセットになっているので今回の用途には向かない. _component_は_bower_ほど知名度はないようである.
_bower_はTwitter社が作ったらしい. __ちょうどいい. _bower_を使おう.__ (依存: 2個)

さて, _bower_で依存関係を取得してみた.

```
$ bower install Stuk/jszip
```

すると驚いたことにjszipのGitリポジトリをクローンした. 無能すぎる. submoduleにしたって同じじゃないか.
まあこんなものかと思いつつ, ではビルドするときにはどうすればよいのかとググった.
すると, _bower_は単体じゃ能無しなので_Grunt_とそのプラグイン使えとのこと. 素直に従う.
なお, _Grunt_関連は_npm_で管理される. なんだ, 結局_npm_使うんじゃないか. (依存: 5個)

```
$ npm install -g grunt grunt-bower-task --save-dev
```

早速_Grunt_を使ってみる. さて, ファイルをコピーしたい. (依存: 6個)

```
$ npm install -g grunt-contrib-copy
```

そうだ. UglifyJS2を忘れていた. これもGrunt用のフロントエンドが必要だ. (依存: 7個)

```
$ npm install -g grunt-contrib-uglify
```

さて, Chrome拡張をパッケージングするかと思ったが, シェルのコマンド呼び出しにも当然追加のパッケージが必要である.
もっとも, パッケージング用のプラグインもあるらしいのだが. (__依存: 8個__)

私はここでやめた.　狂っているとしか思えない. 古き良きmakeを使えばフロントエンドなんぞいらないのに,
_Grunt_はなぜこんなにも依存を要求するのか. こんなものを維持できるとは思えない. 現に,
_grunt-bower-task_はメンテナを募集しており, _npm_は狂ったように古い依存に警告を出す.
もしこの拡張機能がより大きくなったら, 更に将来性に不安のある依存が増えるのではないかと,
今の時点で不安になる.

これは_Grunt_に限った話なのだろうか. いや, そうは思えない. _npm_でこのようなことはもうさんざん経験している.
私はつかれた…