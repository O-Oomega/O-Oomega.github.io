---
title: 关于NextJs的字体优化特性
date: 2024-1-1 09:00:00
categories:
- Idea
tags:
- 突发奇想
---

今天在写Next的项目时，在`app/layout`下看到这样一行代码：

```
import {Inter} from "next/font/google"
const inter = Inter({subsets:["latin"]});
```

于是我就去了解了下这段代码的作用：

-  `import {Inter} from "next/font/google"`

这个代码的作用是**引入了Google Fonts中的Inter字体**;

- `const inter = Inter({subsets:["latin"]});`

这个代码的作用是：
1. **创建了一个Inter字体实例**
2. **指定了使用的字符集为 `latin`**

那么，什么叫“指定了使用的字符集为 `latin`”呢？

这里需要了解一些背景知识。某些字体为了支持不同的语言和符号，通常包含多个字符集。
如果我们不指定字符集，通常网页加载的字体文件会包含所有字符集。而包含多个字符集的字体文件通常体积更大，这会影响到页面的加载时间，特别是在网络连接速度较慢的情况下。

了解了这些后回到项目代码。因为这个项目是个文字内容只有英文的网页。若我们要使用`Inter`字体，在实例化`Inter`字体时，可以特别声明，我们只需要`latin`字符集，用于支持英文。

这样可以缩小字体体积，减少不必要的资源加载，优化网页性能。
