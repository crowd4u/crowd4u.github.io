# crowd4u.github.io

## 執筆者ガイド

### おことわり

ここでは正確な説明をするために、[mustache 構文](https://mustache.github.io/mustache.1.html)を用いて説明する部分があります。

### 環境構築

1. Hugo の入手

[公式サイト](https://gohugo.io/installation/)を参照してください。

1. リポジトリのクローン

適当なディレクトリに移動し、以下 2 行のいずれかを実行します:
``` sh
git clone https://github.com/crowd4u/crowd4u.github.io.git
git clone git@github.com:crowd4u/crowd4u.github.io.git
```

2. submodule の入手

クローンしてきたリポジトリのディレクトリへ移動したら、以下のコマンドを順に実行してください:

```sh
git submodule init
git submodule update
```

### 記事を執筆する

1. 著者情報ファイルの作成(初回のみ)

初めて記事を書く場合は `data/authors/eniehack.toml` といったように、`{{author_name}}.toml`を作成してください。
内容は以下の通りです:

``` toml
name = "Nakaya" # 表示される名前
id = "eniehack" # URL に使用される文字列。「この著者が書いた記事一覧」ページの URL に用いられる。
```

2. 原稿ファイルの作成

まず、以下のコマンドを実行してください。

``` sh
hugo new posts/{{ title }}.md
```

そして、5行目の`author` には、最初作った toml のファイル名（ディレクトリ名などは記載する必要はありません）から拡張子を抜いた文字列を記述します（例: eniehack.toml であれば eniehack）。

3. 原稿のプレビューを確認する

以下コマンドを実行します:

``` sh
hugo server -D
```

4. 原稿を提出する

原稿を作成し、プレビューに問題がなければ、branch を切って commit し、pull request を投げてください。

注意すべき点としては、Hugo には draft という状態を各記事が持っていることです。
デフォルトでは draft な状態に置かれているので、公開する直前になったら記事ファイルの 4 行目にある`draft`を`false`にしてください。

