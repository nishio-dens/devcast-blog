---
title: "無料SSL証明書 Let's EncryptでサーバをHTTPS化する(Nginx + webrootプラグイン利用)"
author: nishio-dens
categories: インフラ
tags: Let's Encrypt, Nginx
draft: false
published_at: 2018-02-04 21:30
---

WebサーバをHTTPS化するにはSSL証明書が必要です。
SSL証明書発行会社を利用すれば証明書は取得できますが、基本的には有料です。
しかし、[Let's encrypt](https://letsencrypt.org/) を利用すれば、無料でかつ簡単にSSL証明書を発行することができます。

今回はNginxで稼働しているサーバをLet's Encryptを利用してHTTPS化する方法を紹介します。

<!-- more -->

# サーバの構成

以下構成のWebサーバをHTTPS化することとします。

- ドメインは example.com とする
  - ドメインは自分が保有しているものに読み替えてください
  - ドメインを保持していないと証明書は発行できません
- ドメインに紐づくWebサーバは**1台**
  - 複数台存在する場合には別途構成を考える必要があります
- サーバではNginxが稼働しており、かつ**HTTP**でアクセスできる
  - 今回はcertbotのHTTP-01を利用して証明書を発行するため、HTTPでサーバにアクセスできる必要があります

# Let's Encrypt SSL証明書発行のフロー

[Certbot](https://github.com/certbot/certbot)と呼ばれるツールを利用して証明書を発行します。

今回はhttp-01 と呼ばれる方法で証明書を取得します。
Certbot + http-01 を使った場合、以下フローでドメインを所有しているかを確認し、証明書を取得します。

1. ユーザは example.com の証明書取得のためのワンタイムトークンの取得をLet's Encryptにリクエストする
2. Let's Encryptはワンタイムトークンを返す
3. ユーザはワンタイムトークンを http://example.com/.well-known/acme-challenge/something_token に配置する
  - something_token はランダムなトークンとなります
4. ユーザはLet's Encryptにドメインチャレンジを要求する
5. Let's Encryptが http://example.com/.well-known/acme-challenge/something_token にアクセスする。トークンが正しければ証明書を発行する
6. 発行した証明書がサーバの /etc/letsencrypt/like/example.com/ 以下に配置される


# Certbotのインストール

Certbotを取得するためにはgitが必要です。CentOSを利用している場合は、

```
sudo yum install -y git
```

でgitをインストールしておく必要があります。

次に適当なディレクトリ(homeなど)で

```
git clone https://github.com/certbot/certbot
```

コマンドを実行し、certbotを取得します。取得後、

```
cd ./certbot
./certbot-auto
```

を実行することで、certbotに必要なライブラリをインストールしてくれます。

# Nginxの設定

Let's Encryptで証明書を発行するためには、ワンタイムトークンを .well-known/acme-challenge に配置する必要があります。
nginx で以下のような設定をして、.well-known/acme-challenge にアクセスできるようにしておきましょう。

ワンタイムトークンは /var/www/letsencrypt-challenge ディレクトリに置くこととします。

```
server {
  listen 80;
  server_name example.com;

  location ^~ /.well-known/acme-challenge/ {
    default_type "text/plain";
    root         /var/www/letsencrypt-challenge;
  }
}
```

# 証明書の発行

先程インストールしたcertbot-auto コマンドを利用して証明書を発行します。

```
./certbot-auto certonly \
      --no-self-upgrade \
      -n \
      --webroot \
      --agree-tos \
      --email your-email-address@examle.com \
      -w /var/www/letsencrypt-challenge/ \
      -d example.com
```

コマンド引数の意味は以下のとおりです。

* --no-self-upgrade
  * letsencrypt-autoプログラムの自動アップデートを行わないようにします
* -n
  * インタラクティブな入力を求めないようにします
* --webroot
  * Webサーバのドキュメントルートディレクトリに認証用ファイルを置いて証明書取得をします
* --agree-tos
  * 利用規約同意画面をスキップします
* --email
  * 自身のメールアドレスを指定してください。証明書の有効期限切れ間近にLet's Encryptが連絡してくれます
* -w [DIR]
  * トークンを置くドキュメントルートを指定します
* -d [DOMAIN]
  * 証明書を申請するドメイン名を指定します

これで正常に動作すれば、/etc/letsencrypt/live/example.com/ 以下に証明書が配置されています。

nginx に以下のような設定をして、証明書を利用するようにします。

```
server {
  listen 443;
  server_name example.com;
  ssl on;
  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

  location / {
    root /var/www/something/;
    index index.html index.htm;
  }
}
```

https://example.com でアクセスしてみて、証明書エラーがでなければ成功です。

# 証明書の更新

Let's Encryptで発行した証明書は、有効期限が**90日**と短いものになっています。
余裕を持って期限30日前までには更新しておくようにしましょう。

更新コマンドは以下のとおりです。

```
./certbot/certbot-auto certonly \
      --force-renewal \
      -n \
      --webroot \
      --agree-tos \
      --email your-email-address@example.com \
      -w /var/www/letsencrypt-challenge/ \
      -d example.com
```

--force-renewal は有効期限に関係なく強制的に証明書を発行しなおすオプションです。
あまり頻繁に更新するとLet's Encryptに負荷がかかってしまったり制限にひっかかったりするため、
cronなどで定期的に実行する場合はこのオプションは付けないようにするか、更新の頻度を月1回程度にするようにしましょう。

また、更新後は nginx の reload をするのを忘れないようにしましょう。

```
sudo systemctl reload nginx
```


# Let's Encrypt証明書取得時の注意点

Let's Encryptの証明書には取得数制限があります。
1週間に**20リクエスト**が上限となります。

テストで何度も証明書取得をリクエストすると、あっという間に上限に達してしまいます。
何度もリクエストして、証明書が取得できないということになりかねないのでご注意ください。

その他にも細かな制限がありますので、詳しくは [https://letsencrypt.org/docs/rate-limits/](https://letsencrypt.org/docs/rate-limits/) をご確認ください。
