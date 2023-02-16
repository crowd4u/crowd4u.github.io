---
title: "M1とWSL両方で動作するJupyterLabのDockerファイルを作った話"
date: 2023-02-16T16:32:38+09:00
draft: false
author: gild
tag: ["Docker", "Python"]
---

## はじめに
融合知能デザイン研究室でもB3向けのプレゼミが始まり、先生や先輩も環境を再現しやすくするためにDockerでコンテナ立てるかーと思い至りました。

そのため、研究室のPC(M1)と自分のPC(Ubuntu on WSL2)の両方でPython, 特にJupyterLabが動作するようなコンテナを作ったので紹介していきます。

### この記事で扱うもの
- DockerコンテナのPython3.10イメージでJupyterLabをインストールする方法
- Dockerで立ち上げたJupyterLabをブラウザで見る方法

### この記事で扱わないもの
- JupyterLabイメージをはじめから使用してコンテナを立ち上げる方法
- JupyterLabの設定ファイルを最初から適応する方法
- PyTorchなどの機械学習ライブラリやGPUをDockerで動かす方法

## ファイル構成
以下のようなファイル構成で作成を行います。
```
.
│  docker-compose.yml
│  Dockerfile
│  requirements.txt
│
└─app
    └─src
```

## requirements.txt

JupyterLabだけ記述していきます。

これだけならDockerfileに直接書けばいいのですが、実際は他にもライブラリを入れるので。。
```text {linenos=true, linenostart=1, name="requirements.txt"}
jupyterlab
```

## Dockerfile

早速Dockerfileを記述していきます。
```Dockerfile {linenos=true, linenostart=1,name="Dockerfile"}
FROM python:3.10-slim

ENV PYTHONBURRERED=1

WORKDIR /usr/src/app

# python & gcc settings
RUN apt update && apt install -y \
    tzdata \
&& apt-get install -y curl \
&& apt-get install -y gcc \
&&  ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
&&  apt-get clean \
&&  rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

RUN pip install --upgrade pip
RUN pip install --upgrade setuptools

# cargo settings
ENV RUST_HOME /usr/local/lib/rust
ENV RUSTUP_HOME ${RUST_HOME}/rustup
ENV CARGO_HOME ${RUST_HOME}/cargo
RUN mkdir /usr/local/lib/rust && \
    chmod 0755 $RUST_HOME
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs > ${RUST_HOME}/rustup.sh \
    && chmod +x ${RUST_HOME}/rustup.sh \
    && ${RUST_HOME}/rustup.sh -y --default-toolchain nightly --no-modify-path
ENV PATH $PATH:$CARGO_HOME/bin

RUN pip install -r requirements.txt

# 8000番ポートでトークンなしでJupyterLabを起動する
CMD ["jupyter", "lab", "--port", "8000", "--allow-root", "--ip=0.0.0.0", "--LabApp.token=''"]
```

M1やWSLでjupyterlabの起動を行うため、以下の設定を行っています。

- gccのインストール
- cargoのインストール
    - そのためのcurlのインストール
    - cargoのパスを通すための設定

以上の処理を行った後、8000番ポートでjupyterlabを起動しています。

トークンを発行するとブラウザからアクセスが出来なかったので、トークンなしでの起動をしています。

## docker-compose.yml

```yaml {linenos=true, linenostart=1,name="docker-compose.yml"}
version: "3.9"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - 8000:8000
    tty: true
    volumes:
      - ./app:/usr/src/app
```

docker-composeでは先ほど作成したDockerfileを指定して、8000番ポートを使うようにしているだけです。

## 起動
お馴染みのコマンドを入力して起動します。

```bash {linenos=true, linenostart=1}
docker-compose build
docker-compose up -d
```

`localhost:8000`にブラウザでアクセスするとJupyterLabにアクセスできます！

## 参考
[https://c-a-p-engineer.github.io/tech/2022/09/29/docker-rust-install/](https://c-a-p-engineer.github.io/tech/2022/09/29/docker-rust-install/)