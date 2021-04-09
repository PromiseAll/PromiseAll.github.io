---
title: vue实现v-down指令下载文件
tag: []
categories: [vue]
date: 2021-03-22 09:25:50
---

前段时间在项目中遇到文件需要下载，a标签发现在下载jpg、txt文件时，并不会直接下载，而是会在浏览器中打开文件，即使给a标签添加了download属性，也无济于事

```html
<a href="static/img/1.jpg" download="2.jpg">下载</a>
```

### 实现v-down（支持Jpg、Txt、Exact、Word、Pdf）

Vue页面中使用指令v-down

```vue
<span v-down="item.Url">(附件)</span>
```

具体实现

```js
let baseDownloadUrl = "https://xxxxx.cn"; // 域名
// 添加自定义v-down指令
Vue.directive("down", {
  inserted: (el, binding) => {
    el.style.cssText = "cursor: pointer;color:red;";
    el.addEventListener("click", () => {
      console.log(binding.value);
      let link = document.createElement("a"); // 创建a标签
      link.style.display = "none";
      link.href = baseDownloadUrl + binding.value; // 设置下载地址
      link.setAttribute("download", ""); // 添加downLoad属性
      document.body.appendChild(link);
      link.click();
    });
  },
});

```

当下载文件中存在jpg、txt等浏览器可以直接预览的文件时，上面的方法就会出现问题，即使加了download仍然会在浏览器中打开下载文件,我们可以将下载地址借助Blob转换成二进制,然后作为a标签的href属性，配合download属性，实现下载

```js
Vue.directive("down", {
  inserted: (el, binding) => {
    el.style.cssText = "cursor: pointer;color:write;";
    el.addEventListener("click", () => {
      console.log(binding.value);
      let link = document.createElement("a");
      let url = baseDownloadUrl + binding.value;
      // 这里是将url转成blob地址，
      fetch(url)
        .then((res) => res.blob())
        .then((blob) => {
          // 将链接地址字符内容转变成blob地址
          link.href = URL.createObjectURL(blob);
          console.log(link.href);
          link.download = "";
          document.body.appendChild(link);
          link.click();
        });
    });
  },
});

```

上面的这种方法，将地址进行了转化成blob，从而实现对jpg、txt 文件的下载

### 使用axios下载

```js
this.$axios({
  method: 'get',
  url: '/task-common-api/api/v1/task/whitelist/template',
  responseType: "blob"
})
.then(function (response) {
  console.log(response);
  if(response.data.type == "application/octet-stream") {
    const filename = response.headers["content-disposition"];
    const blob = new Blob([response.data]);
    var downloadElement = document.createElement("a");
    var href = window.URL.createObjectURL(blob);
    downloadElement.href = href;
    downloadElement.download = filename.split("filename=")[1];
    document.body.appendChild(downloadElement);
    downloadElement.click();
    document.body.removeChild(downloadElement);
    window.URL.revokeObjectURL(href);
  }
})
.catch(function (error) {
  console.log(error);
});
```