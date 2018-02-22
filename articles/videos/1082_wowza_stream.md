---
title: "Wowza Streaming Engine stream名に制限をかける方法"
author: nishio-dens
categories: 動画配信
tags: Wowza, Java, Video Streaming
draft: false
published_at: 2018-02-22 19:00
---

Wowzaでライブ動画を配信する際、配信者はストリーム名を指定します。
このストリーム名はデフォルトではなんでも指定可能ですが、場合によっては制限を掛けたいものです。

今回はストリーム名に制限をかけるモジュールのコードを紹介します。

<!-- more -->

ストリーム名に制限をかけるには、Wowzaのカスタムモジュールを作成する必要があります。
カスタムモジュールの作り方は、[こちら](https://dev.densan-labs.net/articles/videos-1802_hello_wowza_extension)を参考にしてください。

以下コードで配信者が新規接続してきたときにフックをかけられます。

```java
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

public class WowzaLimitation extends ModuleBase {

    // publishは配信要求が来たときに呼ばれる
    public void publish(IClient client, RequestFunction function, AMFDataList params) {
        // ストリーム名を取得する
        String streamName = getParamString(params, PARAM1);
        getLogger().info("Streams from client: " + streamName);

        if (streamName.equals("testtest")) {
            // stream名が testtest だったら配信許可
            // 接続可能なstream名を別システムから取得するなどの連携が考えられる
            invokePrevious(client, function, params);
        } else {
            // stream名がtesttest以外だったら拒否(切断)
            getLogger().info("You cannot use this stream name " + streamName);
            sendClientOnStatusError(client, "NetStream.Publish.BadName", "You cannot use this stream.");
            client.setShutdownClient(true);
        }
    }

    // connection release要求が来たときに呼ばれる
    public void releaseStream(IClient client, RequestFunction function, AMFDataList params) {
      // publishと同様なので省略 ...
    }
}
```

例えばストリーム名を特殊なhash値などにして、hash値を知らない人は配信できないようにすることが可能です。

ストリーム名は視聴者側も知り得る情報です。場合によっては配信者が指定するストリーム名と視聴者が指定するストリーム名を別のものにしたいものです。
以下のようにパラメータを書き換えることで実現可能です。

```java
package net.densanlabs.net;

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

public class WowzaLimitation extends ModuleBase {

    public void publish(IClient client, RequestFunction function, AMFDataList params) {
        String streamName = getParamString(params, PARAM1);
        getLogger().info("Streams from client: " + streamName);

        if (streamName.equals("testtest")) {
            // stream名が testtest だったら配信許可
            // さらにストリーム名を published に書き換えて配信
            params.set(PARAM1, "published");
            invokePrevious(client, function, params);
        } else {
            getLogger().info("You cannot use this stream name " + streamName);
            sendClientOnStatusError(client, "NetStream.Publish.BadName", "You cannot use this stream.");
            client.setShutdownClient(true);
        }
    }

```
