---
title: 利用nodejs实现简单直播
date: 2021-03-17 13:30:03
tag: []
categories: [node]
---

## 项目准备

- [node-media-server](https://github.com/illuspas/Node-Media-Server)
- [flv.js](https://github.com/bilibili/flv.js)

## 服务端

新建app.js 文件 `npm init -y`初始化项目后，安装node-media-server

> ```shell
> npm insatll node-media-server --save
> ```

app.js 内容如下

```js
const nodeMediaServer = require("node-media-server");

const config = {
  rtmp: {
    port: 1935,
    chunk_size: 60000,
    gop_cache: true,
    ping: 60,
    ping_timeout: 30,
  },
  http: {
    port: 8000,
    allow_origin: "*",
  },
};

var nms = new nodeMediaServer(config);
nms.run();
```

使用 `node app.js` 启动项目后即可进行推流了，我使用的是OBS

{% asset_img 1.png%}

## 前端

使用flv.js 播放流文件

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>直播</title>
  </head>
  <body>
    <script src="https://cdn.bootcdn.net/ajax/libs/flv.js/1.5.0/flv.min.js"></script>
    <video id="videoElement" width="100%" controls></video>
    <script>
      if (flvjs.isSupported()) {
        var videoElement = document.getElementById("videoElement");
        var flvPlayer = flvjs.createPlayer({
          type: "flv",
          url: "http://localhost:8000/live/1234.flv", //可以使用ws协议 ws://localhost:8000/live/1234.flv
          // 1234 为串流密钥
        });
        flvPlayer.attachMediaElement(videoElement);
        flvPlayer.load();
        flvPlayer.play();
      }
    </script>
  </body>
</html>
```

{% asset_img 2.png%}

