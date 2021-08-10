---
title: 屏幕适配方案
tag: []
categories: [html,css]
date: 2021-07-16 10:28:23
---

### 使用 css

```css
* {
  margin: 0;
  padding: 0;
}
:root {
  font-size: calc(100vw / 750);
}
body {
  font-size: 16px;
}
```

以设计图750为例,1px = 1rem
