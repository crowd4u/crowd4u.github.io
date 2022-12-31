---
title: "GitHub ActionsではじめてのCI/CD"
date: 2022-12-31T10:00:41+09:00
draft: false
author: 23akei
tags: ["GitHub Actions", "CI/CD", "Hugo"]
---

## Introduction

　GitHub Actions ってよく聞くけどわかってないので使ってみたいと思っていました．また，CI/CDについても同様でした．そこで，ポートフォリオサイトのデプロイを題材にGitHub Actionsに触れてみようと考えました．その過程で得た情報を記事にします．

## GitHub Actionsで出来ること/やりたいこと

　GitHub Actionsとは何か？GitHub Actionsのドキュメントの冒頭が分かりやすく説明してくれていましたのでそのまま引用します．

> GitHub Actions は、ビルド、テスト、デプロイのパイプラインを自動化できる継続的インテグレーションと継続的デリバリー (CI/CD) のプラットフォームです。 リポジトリに対するすべての pull request をビルドしてテストしたり、マージされた pull request を運用環境にデプロイしたりするワークフローを作成できます。
>

*[GitHub Actionsを理解する - GitHub Docs](https://docs.github.com/ja/actions/learn-github-actions/understanding-github-actions#overview)より*

　今回，GitHub Actionsで行いたいことは以下の２つです．

1. Hugoのビルド
2. ビルドしたポートフォリオサイトのデプロイ

　手元にHugoを利用したポートフォリオサイトのプロジェクトがあります．（ポートフォリオサイトの詳細はこちらの記事に：[Hugoでポートフォリオサイトを作成する際の備忘録](https://crowd4u.github.io/posts/2022-12-28-hugo_portfolio/)）

　これをローカルでビルドして，出力されたものをGitHubのリモートレポジトリにpushするだけでGitHub Pagesとして公開することは可能です．しかし，その方法だとサイト更新のたびにコミットログが大量の生成ファイルで埋め尽くされてしまい，見づらくなってしまいます．手元でビルド専用のブランチを切って作業することも可能ですが，面倒なのでこのプロセスを自動化したいと思いました．

　というわけで，Hugoのビルド，デプロイをGitHub Actionsで自動化したいと思います．次節からGitHub Actionsの説明，設定を順に行います．

## Workflowの定義

　WorkflowはGitHub Actionsにおいて実行するジョブの集合で，ブランチのプッシュなどのイベントに応じて発火させることができます．Workflowはレポジトリ内の`.github/workflows` ディレクトリにYAMLファイルで記述します．

　今回は`.github/workflows/hugo-deploy.yml` を作成し， `name` を設定ました．これは任意で，無かったら自動的に決まるそうです．

```yaml
name: hugo-deploy
```

　もう１つ，任意の要素として `run-name` があります．例えとして不適切かもしれませんが， `name` と `run-name` はDockerにおけるイメージ名とコンテナ名に対応する概念です． `run-name` は式を含むことが可能で，動的に設定することが可能みたいです．（コンテキストの話になるのでこの記事では省略）

## Workflow発火条件の定義

　Workflow内の `on` 要素にWorkflowの発火条件を記述できます．今回は `master` ブランチにpushした際に発火させたいので，以下のように記述します．

```yaml
on:
	push:
		branches:
			- master
```

`branches` 以下を省略すれば，全てのブランチにpushされた際に発火させることができます．

また，push時以外にも発火させることができます．（参考：[ワークフローをトリガーするイベント - GitHub Docs](https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows)）

　複数のイベントを条件に発火させることもできるようです．具体的には，複数のイベントが同時に発火した場合，それぞれにWorkflowが実行されるらしいです．

## Jobの定義

　今回実行したい以下の２つの一連の流れをjobとして定義します．

1. Hugoのビルド
2. ビルドしたポートフォリオサイトのデプロイ

上記２つのコマンドについて記述する前にjob全体の定義で設定すべきことがいくつかあります．

```yaml
jobs:
	deploy-hugo:
		runs-on: ubuntu-latest
		steps:
			- uses: actions/checkout@v3
				with:
					submodules: true
					fetch-depatch: 0
```

`jobs` 内に `deploy-hugo` という名前のjobを定義しました．　`runs-on` でjobを実行する仮想マシン（ランナー）の種類を選びます．ubuntu以外にもwindows serverやmacosが選べるみたいです．

`steps` に実行するコマンドを順に記述していきます．

　最初に  `uses: actions/checkout@v3` でランナーにリポジトリのコードをチェックアウトします．ランナー上でリポジトリのコードを使用する際にはこれを必ず実行する必要があります．今回はランナー上でHugoのビルドを行うために必要です．

　今回のレポジトリはHugoのThemeをサブモジュールとして含んでいるので `submodules: true` でサブモジュールもチェックアウトするようにします．

　また， `fetch-depth: 0` を設定することで全てのブランチとタグの全ての履歴をチェックアウトします．*.GitInfoや.Lastmodに必要らしいですが，ここではそれらの説明は省きます．*

## 1. Hugoのビルド

```yaml
			- name: setup hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: latest

			- name: build
        run: hugo --minify
```

　まず，ランナー上にhugoをインストールします. その後, hugoのビルドを実行します．

`name: setup hugo` のステップで `uses: peaceiris/actions-hugo@v2` でhugoのセットアップを行います．

次に，ビルドを実行します． `name: build` のステップで，`run: hugo --minify` でビルドを実行します． `--minify` オプションで出力されるファイルを圧縮します．

## 2. ビルドしたポートフォリオサイトのデプロイ

　今回はGitHub Pagesにデプロイします．便利な既存のActionsを利用します．

```yaml
			- name: deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{secrets.GITHUB_TOKEN}}
          publish_dir: ./public
```

　パラメータとして渡している `github_token` の `secrets.GITHUB_TOKEN` は何かトークンを別で取得して与えなければいけないというものではなく，ランナーによって自動的に生成されるトークンなので設定する必要は無いです．

　`publish_dir` はそのまま公開するディレクトリです．今回は他に特にパラメータを設定していないので，同一レポジトリの `gh-pages` ブランチにランナー上の  `./public` に出力されたビルド結果がコミットされます．

ここまでの `hugo-deploy.yml` の全体を掲載します．

```yaml
name: hugo-deploy

on:
  push:
    branches:
      - master

jobs:
  deploy-hugo:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true
          fetch-depth: 0

      - name: setup hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: latest

      - name: build
        run: hugo --minify

      - name: deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{secrets.GITHUB_TOKEN}}
          publish_dir: ./public
```

## レポジトリ側の設定

　レポジトリ側で `gh-pages` ブランチを公開するように設定する．GithubのレポジトリのSettings→Pagesの”Build and deployment”の”Source”で”Deploy from a branch”を選択し，”Branch”で `gh-pages` を選択して”Save”を押下．

## サイトの微修正

　今回自分はドメイン直下のURLではない位置(`example.com/hoge/` みたいな位置)にデプロイしたため，そのままではページが正常に表示されませんでした．

そこで，Hugoのコンフィグファイルの以下の２点を修正しました．

```yaml
baseURL: https://example.com/hoge # 修正前： https://example.com
canonifyURL: true # 追加
```

`canonifyURL` を設定することで絶対パスを有効化します．

実際にデプロイしたものがこちらになります．[https://23akei.github.io/myportfolio/](https://23akei.github.io/myportfolio/)

そのリポジトリはこちら：[https://github.com/23akei/myportfolio](https://github.com/23akei/myportfolio)

以上です． *Thank you for reading!*

---

参考

[https://gohugo.io/hosting-and-deployment/hosting-on-github/](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

[https://gohugo.io/getting-started/configuration/](https://gohugo.io/getting-started/configuration/)

[https://docs.github.com/ja/pages/getting-started-with-github-pages/creating-a-github-pages-site](https://docs.github.com/ja/pages/getting-started-with-github-pages/creating-a-github-pages-site)

[https://docs.github.com/ja/actions/learn-github-actions/contexts#github-context](https://docs.github.com/ja/actions/learn-github-actions/contexts#github-context)

[https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions](https://docs.github.com/ja/actions/using-workflows/workflow-syntax-for-github-actions)

[https://docs.github.com/ja/actions/learn-github-actions/understanding-github-actions](https://docs.github.com/ja/actions/learn-github-actions/understanding-github-actions)

[https://github.com/marketplace/actions/github-pages-action](https://github.com/marketplace/actions/github-pages-action)

[https://github.com/marketplace/actions/hugo-setup](https://github.com/marketplace/actions/hugo-setup)

[https://zenn.dev/nikaera/articles/hugo-github-actions-for-github-pages](https://zenn.dev/nikaera/articles/hugo-github-actions-for-github-pages)