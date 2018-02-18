---
title: "Wowza Streaming Engineのカスタマイズ WowzaIDE導入編"
author: nishio-dens
categories: 動画配信
tags: Wowza, Java, Video Streaming
draft: false
published_at: 2018-02-18 16:30
---

動画配信をする場合、ストリーミングサーバが必要です。

ストリーミングサーバにはWowzaやRed5、nginx-rtmp-module等がありますが、
今回は安定感がありカスタマイズが容易なWowza Streaming Engineのカスタマイズ方法を紹介します。

<!-- more -->

Wowza Streaming Engine は Javaでカスタマイズすることが可能です。
Wowza自身は手を加えなくても多くの場合はそのまま利用可能なシステムですが、別システムとの連携等が必要な場合は
Streaming Engine自体に手をいれることとなります。

Wowzaを拡張する場合は、WowzaIDEが必要となります。
導入手順は [Wowza公式ガイド](https://www.wowza.com/docs/how-to-extend-wowza-streaming-engine-using-the-wowza-ide) に記載されていますが、今回はこの内容をより詳しく紹介します。

# Wowza Streaming Engineのインストール

Wowzaをカスタマイズする場合、手元のマシンにもWowza Streaming Engineをインストールしておく必要があります。

Wowza Streaming Engineを利用する場合は開発者ライセンスが必要となります。
詳しい導入方法は割愛しますが、ライセンスの導入方法とWowza Streaming Engineのインストール方法は
[Wowza Pricing Downloads](https://www.wowza.com/pricing/installer) をご確認ください。

# WowzaIDEの導入

WowzaIDE は Eclipseのプラグインです。そのためまずはEclipseをインストールする必要があります。

## 手順1: Eclipseのインストール

[Eclipse Downloads](https://eclipse.org/downloads/) からお使いの環境のEclipseをダウンロードし起動してください。

## 手順2: WowzaIDE extensionの導入

EclipseのHelpメニューからInstall New Softwareを押します。

![Install New Software](/images/videos/0218/eclipse_install_new_software.png)

Available Software の Addボタンを押します。

![Add](/images/videos/0218/eclipse_add.png)

Add Repositoryダイアログで、以下入力しOKを押します。

- Name: Wowza
- Location: http://www.wowza.com/wowzaide4/

![Wowza Add](/images/videos/0218/eclipse_wowza_add.png)

Wowza IDE 4 が追加されているので、チェックマークにチェックを入れて、Nextボタンをおします。

![Check](/images/videos/0218/eclipse_wowza_check.png)

Licenseの確認等でてきますので、Acceptして完了します。その後Eclipseを再起動します。

再起動後にメニューバーの New -> Other を押します。

![Add Other](/images/videos/0218/eclipse_add_project_other.png)

Wowza Streaming Engine 関連のクラスが追加できるようになっていれば、WowzaIDEの導入は完了です。

![Wowza Engine](/images/videos/0218/wowza_engine.png)

次回はWowza IDEを使ってカスタムモジュールをWowza Streaming Engineに導入する方法を紹介します。
