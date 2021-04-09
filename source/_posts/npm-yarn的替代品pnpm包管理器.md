---
title: "npm,yarn的替代品pnpm包管理器"
tag: []
categories: [node]
date: 2021-04-08 09:06:01
---

### pnpm

- `快速` 比 npm 和 Yarn 更快
- `高效` 一个版本的软件包只能在磁盘上保存一次
- `确定性` 有一个名为 shrinkwrap.yaml 的锁定文件
- `严格` 一个包只能访问 package.json 中指定的依赖关系
- `无处不在` 适用于 Windows，Linux 和 OS X. 挂钩， node_modules 不再是黑盒子了
- `别名` 安装相同软件包的不同版本或使用不同的名称导入它

{% asset_img 1.png %}

### 安装

```bash
npm install -g pnpm
```

### node_modules

为什么 pnpm 的 node_modules 会不同寻常？ 我们创建两个路径并在其中一个执行 npm add express 然后在另一个中执行 pnpm add express

`npm add express` 的 node_modules 中得到的顶级目录

```
.bin
accepts
array-flatten
body-parser
bytes
content-disposition
cookie-signature
cookie
debug
depd
destroy
ee-first
encodeurl
escape-html
etag
express
```

`pnpm add express` 的 node_modules 中得到的顶级目录

```
.pnpm
.modules.yaml
express
```

所以，所有的依赖去哪了呢？ node_modules 中只有一个叫 .pnpm 的文件夹以及一个叫做 express 的软链，我们只安装了 express，所以它是唯一一个你的应用必须拥有访问权限的包

我们看看在 express 中都有些什么

```
▾ node_modules
  ▸ .pnpm
  ▾ express
    ▸ lib
      History.md
      index.js
      LICENSE
      package.json
      Readme.md
    .modules.yaml
```

express 没有 node_modules? express 的所有依赖都去哪里了？ 诀窍是 `express` 只是一个软链。 当 Node.js 解析依赖的时候，它使用这些依赖的真实位置，所以它不保留软链
