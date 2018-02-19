---
title: "Wowza Streaming Engineのカスタマイズ モジュール作成編"
author: nishio-dens
categories: 動画配信
tags: Wowza, Java, Video Streaming
draft: false
published_at: 2018-02-19 19:00
---

[前回](https://dev.densan-labs.net/articles/videos-1802_wowza_ide)はWowza IDEの導入を行い、Wowza Streaming Engineのカスタマイズの準備ができました。
今回は新規配信を受け付けたら単にログを出すだけのカスタムモジュールを作成し、Wowzaに組み込む方法を紹介します。

<!-- more -->

# 新規プロジェクトを作成する

WowzaIDEをインストールしたEclipseを起動し、メニューの File -> New -> Otherを選択します。
今回はModuleを一つ作成するだけなので、Wowza Streaming Engine Module Classを選択します。

![Image](/images/videos/0219/001.png)

次にプロジェクトの設定を行います。
この時、Wowza Streaming Engineをインストールしたパスを指定する必要があります。

Macの場合、デフォルト設定ですと ``/Library/WowzaStreamingEngine`` にEngineがあります。

![Image](/images/videos/0219/002.png)

package名とクラス名を指定します。
EventMethodはフックしたいイベントを指定します。後から追加可能ですので、適当なものをチェックしておきます。

![Image](/images/videos/0219/003.png)

Finishを押すと新規プロジェクトが作成されます。

# ログ出力だけのサンプルアプリを作成する

配信の接続/切断があった場合にログを出力するだけのExtensionを作成します。
以下のコードで接続/切断時のログ出力が可能です。

```Java
package net.densanlabs;

import com.wowza.wms.application.*;
import com.wowza.wms.amf.*;
import com.wowza.wms.client.*;
import com.wowza.wms.module.*;
import com.wowza.wms.request.*;
import com.wowza.wms.stream.*;
import com.wowza.wms.rtp.model.*;
import com.wowza.wms.httpstreamer.model.*;
import com.wowza.wms.httpstreamer.cupertinostreaming.httpstreamer.*;
import com.wowza.wms.httpstreamer.smoothstreaming.httpstreamer.*;

public class HelloWowzaCustomize extends ModuleBase {

    public void onAppStart(IApplicationInstance appInstance) {
        String fullname = appInstance.getApplication().getName() + "/" + appInstance.getName();
        getLogger().info("アプリが起動しました: " + fullname);
    }

    public void onAppStop(IApplicationInstance appInstance) {
        String fullname = appInstance.getApplication().getName() + "/" + appInstance.getName();
        getLogger().info("アプリが停止しました: " + fullname);
    }

    // 新しい配信接続があった場合に呼ばれる
    public void onConnect(IClient client, RequestFunction function, AMFDataList params) {
        getLogger().info("新しいClientから接続がありました: " + client.getClientId());
    }

    // 配信接続を受け付けた場合に呼ばれる
    public void onConnectAccept(IClient client) {
        getLogger().info("Clientからの接続を受け付けました: " + client.getClientId());
    }

    public void onConnectReject(IClient client) {
        getLogger().info("Clientからの接続を拒否しました: " + client.getClientId());
    }

    // 配信切断された場合に呼ばれる
    public void onDisconnect(IClient client) {
        getLogger().info("Clientが切断しました: " + client.getClientId());
    }

}
```

コードを書いたらビルドを実施します。
すると、``/Library/WowzaStreamingEngine/lib/`` 配下にjarファイルが作成されていると思います。

![Image](/images/videos/0219/004.png)

作成したjarファイルを読み込む設定を、Wowza Streaming Engine Manager上で行います。

Wowza Streaming Engine Manager の Application 設定からModulesを選択し、Editボタンを押します。

![Image](/images/videos/0219/006.png)

Add Module というボタンが表示されるので、さらにそのボタンを押します。

![Image](/images/videos/0219/007.png)

モーダルが立ち上がるので以下入力します。

- Name: HelloWowza
  - これはなんでもよいです。わかりやすい名前をつけましょう
- Description
  - これもわかりやすい名前をつけましょう
- Fully Qualified Class Name
  - 作成したクラス名を入力します。今回はpackageが net.densanlabs で class名が HelloWowzaCustomize として作成したので、net.densanlabs.HelloWowzaCustom と指定します

追加すると以下のようになります。Saveをします。

![Image](/images/videos/0219/008.png)

保存後、新しいプラグインを反映するには、Wowza Streaming Engineを再起動する必要があります。
Restart Now というダイアログが以下のように表示されているので、ボタンをおします。

![Image](/images/videos/0219/009.png)

または、Server 画面にRestartというボタンがあるので、そこで再起動も可能です。

![Image](/images/videos/0219/005.png)

再起動後にWowza経由で配信を行うと、以下のようにログが出力されていることがわかります。
ログは ``/Library/WowzaStreamingEngine/logs/wowzastreamingengine_access.log`` にて確認可能です。

![Image](/images/videos/0219/010.png)


なお、Module設定は ``/Library/WowzaStreamingEngine/conf/{application名}/Application.xml`` に保存されています。
こちらを直接書き換えてWowza Streaming Engineを再起動してもモジュールが読み込まれます。

![Image](/images/videos/0219/011.png)
