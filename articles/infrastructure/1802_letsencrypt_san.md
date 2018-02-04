---
title: " Let's Encryptでマルチドメイン(SANs)証明書を発行する"
author: nishio-dens
categories: インフラ
tags: Let's Encrypt, Nginx
draft: false
published_at: 2018-02-05 00:30
---

[前回](/articles/infrastructure-1802_letsencrypt) はLet's Encryptを使い単一ドメインのHTTPS化を行いました。

サブドメインの異なる複数サイトを保有している場合は、1枚ずつ証明書を発行するか、ワイルドカード証明書を発行するか、SANs証明書を利用する必要があります。

<!-- more -->

# SANs証明書とは

SANs は Subject Alternative Names の略で、この証明書を利用すると複数ドメインに対応した1枚の証明書が発行できます。

例えば、

- http://example.com
- http://www.example.com
- http://api.example.com

という3つのドメインを管理している場合、通常は3枚証明書を発行しますが、SANs証明書を使えば1つにまとめることができます。


# Let's EncryptでSANs証明書を発行する

Nginxの設定は以下のようにして、全てのドメインでLet's Encryptにトークンを返せるようにしておきます。

```
server {
  listen 80;
  server_name example.com www.example.com api.example.com;

  location ^~ /.well-known/acme-challenge/ {
    default_type "text/plain";
    root         /var/www/letsencrypt-challenge;
  }
}
```

certbot-auto コマンドを利用して証明書を発行します。
単純に -d オプションの後に複数ドメインを指定するだけです。

```
./certbot-auto certonly \
      --no-self-upgrade \
      -n \
      --webroot \
      --agree-tos \
      --email your-email-address@examle.com \
      -w /var/www/letsencrypt-challenge/ \
      -d example.com
      -d www.example.com
      -d api.example.com
```

Let's encryptから

- http://example.com/.well-known/acme-challenge/{{some-token}}
- http://www.example.com/.well-known/acme-challenge/{{some-token}}
- http://api.example.com/.well-known/acme-challenge/{{some-token}}

宛にアクセスがきます。Let's Encryptからのアクセスを正しく返せればSANs証明書が /etc/letsencrypt/live/example.com/ 配下に置かれます。

densan-labs.net (2018-02-05現在) の場合、Let's Encrypt で以下のSANs証明書を発行しています。
Chromeなどのブラウザで証明書をみて、DNS名が複数存在していればSANs証明書が発行できています。

![SANs証明書](/images/infrastructure/1802_letsencrypt.png)
