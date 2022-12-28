---
title: "Hugoでポートフォリオサイトを作成する際の備忘録"
date: 2022-12-28T15:20:37+09:00
draft: false
author: 23akei
tags: ["Hugo", "GitHub Actions"]
---

## Introduction

　この記事は筆者が１年放置したポートフォリオサイトの更新に際して，でHugoでポートフォリオサイトを作る際の手順をまとめておこうと思い，執筆するものです．

## What’s/Why Hugo?

### What’s?

> Hugo is a static HTML and CSS website generator written in [Go](https://go.dev/). It is optimized for speed, ease of use, and configurability. Hugo takes a directory with content and templates and renders them into a full HTML website.
>

*（[hugo README.md](https://github.com/gohugoio/hugo/blob/master/README.md)より）*

　HugoはGolang（プログラミング言語Go）によって記述された静的ウェブサイトジェネレーターです．HTML+CSSで構成された静的サイトを生成するのに便利です．

### Why?

筆者は今までに何度かHugoでブログや静的サイトを構築したことがあります．他の静的サイトジェネレーターの利用経験が無いため比較はできないのですが，Hugoには以下の良い点があると思います．

1. Markdownで記事が書けるという点がとても楽で良い．必要に応じてHTMLをMarkdown内に埋め込むことができる．
2. [https://themes.gohugo.io/](https://themes.gohugo.io/)等で公開されているthemeを利用することで，いい感じのデザインのページを簡単に生成できる．

## Hugoのインストール

　LinuxやMacOSの環境の方はパッケージマネージャーを利用してインストールするのが楽でいいと思います．例えばUbuntu等apt環境だとこんな感じ．

```bash
$ sudo apt install hugo
```

　Windows用ソフトウェアでよくあるインストーラパッケージは用意されておらず，Chocolatey等のパッケージマネージャーを利用する必要があったり，Dockerのイメージを利用する必要があります．個人的には面倒な気がするのでWSL上のLinuxにインストールして利用するのがいいと思います．

　オープンソースなのでgoの開発環境が有る方は自前でビルドするのもいいかもしれません．

　注意すべき点として，Hugoにはstandardとextendedの２つのエディションがあることが挙げられます．一部のThemeはextendedの機能を要求するため，基本的にはextendedをインストールするのがよいでしょう．以下に示す参考ページに具体的なインストール方法が示されていますが，パッケージマネージャーによってextendedのパッケージ名が異なるため注意してください．

参考：[https://gohugo.io/installation/](https://gohugo.io/installation/)

## はじめの１コマンド

`hugo new site` コマンドで新しいプロジェクトが生えます．

```bash
$ hugo new site myportfolio
```

例えば上記コマンドでカレントディレクトリの `myportfolio` 以下にhugoのプロジェクトが生えます．こんな感じに．

```bash
$ cd myportfolio/
$ ls
archetypes  assets  config.toml  content  data  layouts  public  static  themes
```

デフォルトのconfig.tomlはこんな感じです．こんな感じなんだ～（小並感）．

```toml
baseURL = 'http://example.org/'
languageCode = 'en-us'
title = 'My New Hugo Site'
```

参考：[https://gohugo.io/commands/hugo_new_site/](https://gohugo.io/commands/hugo_new_site/)

gitでプロジェクトのソースを管理してる人は `git init` しておくと良いと思います．

## テーマを適用する

　今回私は[Hugo Profile](https://themes.gohugo.io/themes/hugo-profile/) を使います．画像少なめのポートフォリオサイト用Themeということで選びました．適当に選んだのでこの後変えるかも．（ちなみに前回Hugoでポートフォリオサイト作った時もこんな気持ちで，その一年後が今です．本当はThemeの入れ替えだけならわざわざプロジェクトを作り直す必要は無いのですが，記事のためにやりなおしています．）

　テーマの使い方は各テーマのReadmeとかに書いてあるのでそれぞれ参照してください．以下，この記事は[Hugo Profile](https://themes.gohugo.io/themes/hugo-profile/)を使った説明になります．

とりあえず，submoduleとして `themes` 内にテーマを持ってきます．

```bash
myportfolio:$ git submodule add git@github.com:gurusabarish/hugo-profile.git themes/hugo-profile
```

## テーマを編集する

　Hugoのポートフォリオテーマははポートフォリオ機能だけなら，configファイルだけで完結してしまうことが多く（筆者個人の感想．n_themes=2），今回のThemeもそうでした．ですので，Hugoを紹介しているサイトとかでよくある `hugo new` コマンドを用いた新規記事の作成の説明は省きます．

　ここでHugo ProfileのReadmeを読むと `hugo new site` の際に `-f=yaml` オプションをつけろと書いてありますね．デフォルトではconfigファイルはTOMLですが， yamlも指定できるらしい．Hugoのプロジェクトを立ち上げるときには，まず最初に利用するテーマを決めて，そのReadmeのUsageを確認することをおすすめします．

　今回私はすでにオプションを指定せずに`hugo new site` を実行して `config.toml`を作ってしまいましたが， `config.toml` を削除して `config.yaml` をテーマの `exampleSite` 内から持ってきたら解決しました．

　yamlファイルを適当に編集して「とりあえず出せそうな感じになったな」と思ったら内容が激薄になっていました．


{{<figure src="/posts/2022-12-28/portfoliosite.png" alt="screenshot_of_my_portfolio_site">}}

　ローカルでのテストは `hugo server` コマンドでできます． `localhost:1313` でテストサーバーにアクセスできます．実行中にファイルを更新すると自動でリビルドが走って更新されるので便利です．

　デプロイしたサイトはこちらです．[https://23akei.github.io/myportfolio/](https://23akei.github.io/myportfolio/)

　以上です．実際のデプロイやCI/CDについては別記事で． *Thank you for reading!*
