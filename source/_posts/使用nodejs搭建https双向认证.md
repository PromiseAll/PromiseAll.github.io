---
title: 使用nodejs搭建https双向认证
tag: []
categories: [node]
date: 2021-04-09 13:56:55
---

研究 HTTPS 的双向认证实现与原理，踩了不少坑，终于整个流程都跑通了,把一些心得，特别是容易踩坑的地方记录下来

### 原理

双向认证，顾名思义，客户端和服务器端都需要验证对方的身份，在建立 Https 连接的过程中，握手的流程比单向认证多了几步。单向认证的过程，客户端从服务器端下载服务器端公钥证书进行验证，然后建立安全通信通道。双向通信流程，客户端除了需要从服务器端下载服务器的公钥证书进行验证外，还需要把客户端的公钥证书上传到服务器端给服务器端进行验证，等双方都认证通过了，才开始建立安全通信通道进行数据传输

### 单向认证流程

{% asset_img 1.webp%}

- 客户端发起建立 HTTPS 连接请求，将 SSL 协议版本的信息发送给服务器端
- 服务器端将本机的公钥证书（server.crt）发送给客户端
- 客户端读取公钥证书(server.crt)，取出了服务端公钥
- 客户端生成一个随机数（密钥 R），用刚才得到的服务器公钥去加密这个随机数形成密文，发送给服务端
- 服务端用自己的私钥(server.key)去解密这个密文，得到了密钥 R
- 服务端和客户端在后续通讯过程中就使用这个密钥 R 进行通信了

### 双向认证流程

{% asset_img 2.webp%}

- 客户端发起建立 HTTPS 连接请求，将 SSL 协议版本的信息发送给服务端
- 服务器端将本机的公钥证书(server.crt)发送给客户端
- 客户端读取公钥证书(server.crt)，取出了服务端公钥
- 客户端将客户端公钥证书(client.crt)发送给服务器端
- 服务器端使用根证书(root.crt)解密客户端公钥证书，拿到客户端公钥
- 客户端发送自己支持的加密方案给服务器端
- 服务器端根据自己和客户端的能力，选择一个双方都能接受的加密方案，使用客户端的公钥加密后，发送给客户端
- 客户端使用自己的私钥解密加密方案，生成一个随机数 R，使用服务器公钥加密后传给服务器端
- 服务端用自己的私钥去解密这个密文，得到了密钥 R
- 服务端和客户端在后续通讯过程中就使用这个密钥 R 进行通信了

### 生成证书

如果要把整个双向认证的流程跑通，最终需要六个证书

- 服务器端公钥证书：server.crt
- 服务器端私钥文件：server.key
- 根证书：root.crt
- 客户端公钥证书：client.crt
- 客户端私钥文件：client.key
- 客户端集成证书（包括公钥和私钥，用于浏览器访问场景）：client.p12

{% asset_img 3.png%}

生成这一些列证书之前，我们需要先生成一个 CA 根证书，然后由这个 CA 根证书颁发服务器公钥证书和客户端公钥证书，为了验证根证书颁发与验证客户端证书这个逻辑，我们使用根证书办法两套不同的客户端证书，然后同时用两个客户端证书来发送请求，看服务器端是否都能识别

{% asset_img 4.png%}

我们可以全程使用 openssl 来生成一些列的自签名证书，自签名证书没有听过证书机构的认证，很多浏览器会认为不安全，但我们用来实验是足够的。需要在本机安装了 openssl 后才能继续本章的实验

### 生成自签名根证书

> ```bash
> （1）创建根证书私钥：
> openssl genrsa -out root.key 1024
>
> （2）创建根证书请求文件：
> openssl req -new -out root.csr -key root.key
> 后续参数请自行填写，下面是一个例子：
> Country Name (2 letter code) [XX]:cn
> State or Province Name (full name) []:bj
> Locality Name (eg, city) [Default City]:bj
> Organization Name (eg, company) [Default Company Ltd]:alibaba
> Organizational Unit Name (eg, section) []:test
> Common Name (eg, your name or your servers hostname) []:root
> Email Address []:a.alibaba.com
> A challenge password []:
> An optional company name []:
>
> （3）创建根证书：
> openssl x509 -req -in root.csr -out root.crt -signkey root.key -CAcreateserial -days 3650
> ```

在创建证书请求文件的时候需要注意三点，下面生成服务器请求文件和客户端请求文件均要注意这三点：

1. 根证书的 Common Name 填写 root 就可以，所有客户端和服务器端的证书这个字段需要填写域名，一定要注意的是，根证书的这个字段和客户端证书、服务器端证书不能一样
2. 其他所有字段的填写，根证书、服务器端证书、客户端证书需保持一致
3. 最后的密码可以直接回车跳过

经过上面三个命令行，我们最终可以得到一个签名有效期为 10 年的根证书 root.crt，后面我们可以用这个根证书去颁发服务器证书和客户端证书

### 生成自签名服务器端证书

> ```bash
> （1）生成服务器端证书私钥：
> openssl genrsa -out server.key 1024
>
> （2） 生成服务器证书请求文件，过程和注意事项参考根证书，本节不详述：
> openssl req -new -out server.csr -key server.key
>
> （3） 生成服务器端公钥证书
> openssl x509 -req -in server.csr -out server.crt -signkey server.key -CA root.crt -CAkey root.key -CAcreateserial -days 3650
> ```

经过上面的三个命令，我们得到：

- server.key：服务器端的秘钥文件
- server.crt：有效期十年的服务器端公钥证书，使用根证书和服务器端私钥文件一起生成

### 生成自签名客户端证书

> ```bash
> （1）生成客户端证书秘钥：
> openssl genrsa -out client.key 1024
> openssl genrsa -out client2.key 1024
>
> （2） 生成客户端证书请求文件，过程和注意事项参考根证书，本节不详述：
> openssl req -new -out client.csr -key client.key
> openssl req -new -out client2.csr -key client2.key
>
> （3） 生客户端证书
> openssl x509 -req -in client.csr -out client.crt -signkey client.key -CA root.crt -CAkey root.key -CAcreateserial -days 3650
> openssl x509 -req -in client2.csr -out client2.crt -signkey client2.key -CA root.crt -CAkey root.key -CAcreateserial -days 3650
>
>
> （4） 生客户端p12格式证书，需要输入一个密码，选一个好记的，比如123456
> openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12
> openssl pkcs12 -export -clcerts -in client2.crt -inkey client2.key -out client2.p12
> ```

重复使用上面的三个命令，我们得到两套客户端证书：

- client.key/client2.key：客户端的私钥文件
- client.crt/client2.key：有效期十年的客户端证书，使用根证书和客户端私钥一起生成
- client.p12/client2.p12：客户端 p12 格式，这个证书文件包含客户端的公钥和私钥，主要用来给浏览器访问使用

### 服务端配置（app.js）

```js
const https = require("https");
const fs = require("fs");

const options = {
  key: fs.readFileSync("./cert/server.key"),
  cert: fs.readFileSync("./cert/server.crt"),
  // （双向认证）添加可信任的CA证书，用于验证客户端证书
  ca: fs.readFileSync("./cert/root.crt"),
  // passphrase: "146116", 如果证书需要密码则使用密码

  // 使用客户端证书验证
  requestCert: true,
  // 如果没有请求到客户端来自信任CA颁发的证书，拒绝客户端的连接
  rejectUnauthorized: true,
};
const port = 443;
https
  .createServer(options, (req, res) => {
    console.log("server connected", res.connection.authorized ? "authorized" : "unauthorized");
    res.writeHead(200);
    res.end("hello world!\n");
  })
  .listen(port, () => {
    console.log(`running server https://127.0.0.1`);
  });

// 使用客户端连接测试
require("./client");
```

### 客户端配置（client.js）

```js
const https = require("https");
const fs = require("fs");

var options = {
  host: "localhost",
  port: 443,
  path: "/",
  method: "GET",
  // （双向认证）请求使用客户端证书，使服务器信任请求
  key: fs.readFileSync("./cert/client.key"),
  cert: fs.readFileSync("./cert/client.crt"),

  // （单向认证）添加可信任根证书，验证服务端证书 验证是否可信任
  ca: [fs.readFileSync("./cert/root.crt")],

  rejectUnauthorized: true, // 如果验证失败，则断开请求
  agent: false, // 仅为此一个请求创建一个新代理
};
var req = https.request(options, function (res) {
  console.log("server authorize status: " + res.socket.authorized);
  res.on("data", function (d) {
    console.log("响应数据: " + d);
  });
});
req.end();
req.on("error", function (e) {
  console.error(e);
});
```

### 要注意的地方

- 根证书的 Common Name 填写 root 就可以，所有客户端和服务器端的证书这个字段需要填写域名，一定要注意的是，根证书的这个字段和客户端证书、服务器端证书不能一样
- server 端的 ca 需要配置根证书 root.crt，而不是客户端证书,client 端的 ca 需要配置根证书 root.crt，而不是服务端证书
