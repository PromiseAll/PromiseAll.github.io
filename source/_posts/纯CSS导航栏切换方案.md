---
title: 纯CSS导航栏切换方案
tag: []
categories: [html]
date: 2021-04-01 09:00:30
---

如何通过点击某个元素，控制其他元素,不用 Javascript，使用纯 CSS 方案，实现类似下图的导航栏切换
{% asset_img 1.gif %}

### :target 伪类选择器

首先，我们要解决的问题是如何接收点击事件，这里第一种方法我们采用 :target 伪类接收

> :target 是 CSS3 新增的一个伪类，可用于选取当前活动的目标元素。当然 URL 末尾带有锚名称 #，就可以指向文档内某个具体的元素，这个被链接的元素就是目标元素(target element)，它需要一个 id 去匹配文档中的 target

假设我们的 `HTML` 代码如下

```html
<ul class="nav">
  <li>列表1</li>
  <li>列表2</li>
</ul>
<div>列表1内容:123456</div>
<div>列表2内容:abcdefgkijkl</div>
```

由于要使用 `:target`，需要 HTML 锚点，以及锚点对应的 HTML 片段。所以上面的结构要变成

```html
<ul class="nav">
  <li><a href="#content1">列表1</a></li>
  <li><a href="#content2">列表2</a></li>
</ul>
<div id="content1">列表1内容:123456</div>
<div id="content2">列表2内容:abcdefgkijkl</div>
```

这样，上面 <a href="#content1"> 中的锚点 #content1 就对应了列表 1 <div id="content1"> ，锚点 2 与之相同对应列表 2，接下来，我们就可以使用 :target 接受到点击事件，并操作对应的 DOM 了

```css
#content1,
#content2 {
  display: none;
}

#content1:target,
#content2:target {
  display: block;
}
```

上面的 CSS 代码，一开始页面中的 #content1 与 #content2 都是隐藏的，当点击列表 1 触发 href="#content1" 时，页面的 URL 会发生变化

- 由 www.example.com 变成 www.example.com#content1
- 接下来会触发 #content1:target{ } 这条 CSS 规则，#content1 元素由 `display:none` 变成 `display:block` ，点击列表 2 亦是如此

如此即达到了 Tab 切换，当然除了 content1 content2 的切换，我们的 li 元素样式也要不断变化，这个时候，就需要我们在 DOM 结构布局的时候多留心，在 #content1:target 触发的时候可以同时去修改 li 的样式

在上面 HTML 的基础上，再修改一下，变成如下结构

```html
<div id="content1">列表1内容:123456</div>
<div id="content2">列表2内容:abcdefgkijkl</div>
<ul class="nav">
  <li><a href="#content1">列表1</a></li>
  <li><a href="#content2">列表2</a></li>
</ul>
```

仔细对比一下与上面结构的异同，这里我只是将 ul 从两个 content 上面挪到了下面，为什么要这样做呢

> E~F{ cssRules } ，CSS3 兄弟选择符(E~F) ，选择 E 元素后面的所有兄弟元素 F。
>
> 注意这里，最重要的一句话是 E~F 只能选择 E 元素 **之后** 的 F 元素，所以顺序就显得很重要了

在这样调换位置之后，通过兄弟选择符 ~ 可以操作整个 `.nav` 的样式

```css
#content1:target ~ .nav li {
  // 改变li元素的背景色和字体颜色
  &:first-child {
    background: #ff7300;
    color: #fff;
  }
}

#content2:target ~ .nav li {
  // 改变li元素的背景色和字体颜色
  &:last-child {
    background: #ff7300;
    color: #fff;
  }
}
```

上面的 CSS 规则中，我们使用 ~ 选择符，在 #content1:target 和 #content2:target 触发的时候分别去控制两个导航 li 元素的样式，至此两个问题，1. 如何接收点击事件 与 2. 如何操作相关 DOM 都已经解决，剩下的是一些小样式的修补工作

### <input type="radio"> && <label for="">

上面的方法通过添加 <a> 标签添加页面锚点的方式接收点击事件。
这里还有一种方式能够接收到点击事件，就是拥有 checked 属性的表单元素， <input type="radio"> 或者 <input type="checkbox"> 。
