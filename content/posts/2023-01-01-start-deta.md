---
title: "Deta.shでサクッとウェブアプリをデプロイしてみる"
date: 2023-01-01T00:00:00+09:00
draft: false
author: gild
tags: ["FastAPI", "deta.sh"]
---

## はじめに
　昨年、今までアプリケーションの運用ができるPaaS(Platform as a Service)の一つであるHerokuが無料プランを廃止し、代替手段を探すエンジニア(特に学生)が続出しました。
私もそのうちの一人なわけですが、代替サービスでたまに聞くようになったDeta.shが面白そうだったので、サクッとアプリ作成からデプロイまでやってみようと思います。

## Deta.shとは
Deta.sh([https://www.deta.sh/](https://www.deta.sh/))は完全無料のホスティングサービスの一つで、以下の3つのサービスを提供しています。

- Deta Micros: デプロイサービス
- Deta Base: NoSQLデータベース
- Deta Drive: ファイルストレージ

Deta Microsに関してはPythonとNode.jsで書かれたアプリのデプロイができるようですね。

## 早速アプリを作ってみる
というわけで、早速Deta Microsを使ってアプリをデプロイしていこうと思います。
Detaアカウントは作成済という前提で説明していきます。

### Deta CLIのインストール
Detaアカウントを作成したら、ローカルからDeta Microsへデプロイを行ったりMicro上のプログラムのcron等が出来るDeta CLIをインストールしていきます。

WindowsであればPowershellで、以下のコマンドを入力します。
```shell
iwr https://get.deta.dev/cli.ps1 -useb | iex
```
Mac, Linuxであればターミナルで、以下のコマンドを実行するとインストールされます。
```bash
curl -fsSL https://get.deta.dev/cli.sh | sh
```
インストールが終了したら、自身の持つユーザーアカウントにデプロイを行うために、ターミナルからログインコマンドを入力します。
```bash
deta login
```
コマンド入力後、ブラウザが自動で開きログインを求められます。
detaコマンドがないと言われた場合は、ターミナルの再起動やパスの設定を適宜行ってください。

### アプリケーションを作る
デプロイするためのアプリケーションを作っていきます。
正月ということで、運勢を返すだけのおみくじアプリでも作ってみましょうかね。。。

今回は以下の環境でアプリを作っていきます。
- Python3.9
    - FastAPI
- HTML
    - Jinja2
- CSS
    - Bootstrap5

Deta.shがFastAPIのスポンサーらしく、FastAPIに[チュートリアル](https://fastapi.tiangolo.com/ja/deployment/deta/)が存在しますので参考にしていきます。
最終的なディレクトリ構成は以下の様になりました。
```text
.
├── main.py
├── requirements.txt
└── template
    ├── index.html
    └── result.html
```
まずは作業ディレクトリをつくり、`template`フォルダを作り、その中にトップページの`index.html`とおみくじの結果を表示する`result.html`を作ります。

HTMLは以下のように作成しました。
- index.html: トップページ
```HTML {linenos=true, linenostart=1, name="index.html"}
<!DOCTYPE html>
<html>
    <head>
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
        <title>Deta.sh-Tryout</title>
    </head>
    <body>
        <h1 class="bg-danger text-white"><a href="/" class="text-decoration-none text-white">おみくじ</a></h1>
            <div class="card mx-auto" style="width: 75%; height: 50%;">
            <div class="card-body">
                <form  action="result" method="POST">
                    <button type="submit" style="width: 80%; height: 18rem" class="btn btn-danger mx-auto d-flex justify-content-center"><h2>おみくじを引く！</h2></button>
                </form>
            </div>
            </div>
    </body>
</html>
```
- result.html: 結果ページ
```HTML {linenos=true, linenostart=1, name="result.html"}
<!DOCTYPE html>
<html>
    <head>
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">
        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM" crossorigin="anonymous"></script>
        <title>あなたの運勢</title>
    </head>
    <body>
        <h1 class="bg-danger text-white"><a href="/" class="text-decoration-none text-white">おみくじ</a></h1>
            <div class="card mx-auto" style="width: 75%; height: 50%;">
            <div class="card-body">
                <p>ボタンをもう一度クリックするともう一度実行するよ！</p>
                <form  action="/result" method="POST">
                    <button type="submit" data-html="true" style="width: 80%; height: 18rem" class="btn btn-danger mx-auto d-flex justify-content-center text-nowrap">
                        <h2>{{lack}}</h2>
                    </button>
                </form>
            </div>
            </div>
    </body>
</html>
```
`result.html`の15行目、`{{lack}}`にはFastAPIから渡された文字列が挿入されます。

次に、FastAPIでサーバーを立てるための`main.py`を作業ディレクトリ直下に作成し、以下のようにします。
```Python {linenos=true, linenostart=1, name="main.py"}
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates
import random

# FastAPIの呼び出し
app = FastAPI()
# Jinja2テンプレートを使用するためにテンプレートディレクトリを指定して呼び出し
templates = Jinja2Templates(directory="template")

# Topページのテンプレートを返す
@app.get("/", response_class=HTMLResponse)
async def index(request:Request):
    return templates.TemplateResponse("index.html", {"request":request})

# ボタンが押された際に結果画面に遷移する
@app.post("/result", response_class=HTMLResponse)
async def result(request:Request):
    # おみくじの運勢リストを作成
    lack_list = ["大吉", "吉", "中吉", "小吉", "末吉", "凶", "大凶"]
    # リストの要素をシャッフルする
    random.shuffle(lack_list)
    # シャッフル後に0番目の要素をおみくじの運勢として返す
    lack = lack_list[0]
    return templates.TemplateResponse("result.html", {"request":request, "lack":lack})
```

### Detaにデプロイ
ここまで書けたらDetaにデプロイしてみましょう！  
以下のコマンドを順番に入力していきます。必ずターミナルでは作業ディレクトリを開いた状態にしておいてください。

 1. deta上でインストールするパッケージを指定するために作業ディレクトリ直下に`requirements.txt`を作成する

    今回は`requirements.txt`を以下のようにしました。
    Numpyなどの他のライブラリも、テキストファイル上に記載することでインストールできそうですね。
    ```text {linenos=true, linenostart=1, name="requirements.txt"}
    fastapi == 0.85.1
    jinja2 == 3.1.2
    ```
    このコマンドを打った後、以下のような出力が表示されます。
    ```json {linenos=true, linenostart=1}
    {
        "name": {アプリケーションの名前},
        "id": {Deta MicrosのID},
        "project": {プロジェクトID},
        "runtime": "python3.9",
        "endpoint": {アプリケーションのURL},
        "region": "ap-southeast-1",
        "visor": "disabled",
        "http_auth": "disabled"
    }
    ```
    これが作成したアプリケーションの情報で、`deta details`コマンドでも確認することが出来ます。

    この出力の`endpoint`の欄に記載されているのが、デプロイしたアプリケーションのURLとなります。


2. アプリケーションをDeta上に作成する

    以下の様にコマンドを入力します。
    ```
    deta new
    ```

3. detaのvisorを有効にします

    アプリケーションの作成した後だと、HTTPS通信のためのポート開放が行われておらず、アプリケーションにアクセスしてもエラーを返すことがあります。

    Detaでは、visorと呼ばれる機能を有効化することで、アプリにアクセスできるようにしてくれるようです。
    ```bash
    deta visor enable
    ``` 
     
4. Detaにデプロイする

    ここまで来たらついにデプロイです！

    オンラインダッシュボード上での煩雑な設定とかもなしに3行でデプロイできてしまいます。すごく便利ですね。。。

    捕捉ですが、バグや新機能の実装後もこのコマンドを実行することで最新のアプリケーションをデプロイすることが出来ます。 

5. アプリにアクセスして確認する

    早速、1で確認したURLにアクセスして結果を確認してみましょう！

    私の場合は以下のリンクにデプロイされています。

    [https://y0rcw0.deta.dev](https://y0rcw0.deta.dev/) 

    無事デプロイできていそうです！
    {{<figure src="/posts/2023-01-01/result.png" width="75%" alt="デプロイ結果">}}
    以上がDeta.shを使ってウェブアプリをデプロイする方法です。

    ちなみにDeta.shはウェブダッシュボードでサブドメインを設定することが出来るので、使われていなければ自分のアプリに合ったドメインを設定することも可能です。

    今回作ったアプリケーションは以下のURLからも確認することが出来ます。

    [https://omikuji-n4u.deta.dev](https://omikuji-n4u.deta.dev/)

## おわりに
サクッとと言いつつ、少々長くなってしまった気もしますが、ここまで読んでいただきありがとうございました！

今回紹介したDeta.shは細かい機能不足などはあると思いますが、個人で制作した簡単なアプリを公開するにはかなり便利に感じました。

Deta.shのサイトを読んでいると、今後も無料でサービスを提供するらしいので、もしかしたら個人的に長いお付き合いになるかもしれません(笑)

## おまけ
幣開発チームのリーダーが今回作成したアプリを試してくれたらしいのですが、このようなスクショが送られてきました笑
    {{<figure src="/posts/2023-01-01/omake.png" width="75%" alt="おまけ">}}
ま、まぁこれ以上悪くならないという意味でもあるので。。。(震え)

## 参考
- "Deta Docs"：[https://docs.deta.sh/docs/home/](https://docs.deta.sh/docs/home/)
- "Detaにデプロイ": [https://fastapi.tiangolo.com/ja/deployment/deta/?h=deta](https://fastapi.tiangolo.com/ja/deployment/deta/?h=deta)