---
title: "Demo"
description:
date: "2022-03-10T11:54:01+08:00"
slug: "demo"
image: "cover.jpg"
license: false
hidden: false
comments: false
draft: true
tags:
categories:
weight: 1 # You can add weight to some posts to override the default sorting (date descending)
---

## 图片库，Image Gallery

使用 Markdown 创建精美的交互式图片库

Hugo theme Stack支持使用 Markdown 创建交互式图片库。它由 [PhotoSwipe](https://photoswipe.com/) 提供支持，语法灵感来自 [Typlog](https://typlog.com/) 。

### 语法

```markdown
![Image 1](1.jpg) ![Image 2](2.jpg)
```

### 结果

![Image 1](1.jpg) ![Image 2](2.jpg)

> 图片来自 [Unsplash](https://unsplash.com/) 平台 [mymind](https://unsplash.com/@mymind) 、[Luke Chesser](https://unsplash.com/@lukechesser)

## 数学排版, Math Typesetting

Stack 内置支持数学排版，使用 [KaTeX](https://katex.org/)。

**默认情况下，全站未启用**，但可通过在 front matter 添加 `math: true` 为单个帖子启用。或通过在 `config.toml` 的 `params.article` 部分添加 `math = true` 全站启用。

### 内联数学

这是一个内联数学表达式：$\varphi = \dfrac{1+\sqrt5}{2}= 1.6180339887…$

```markdown
$\varphi = \dfrac{1+\sqrt5}{2}= 1.6180339887…$
```

### 块数学

$$
    \varphi = 1+\frac{1} {1+\frac{1} {1+\frac{1} {1+\cdots} } }
$$

```markdown
$$
    \varphi = 1+\frac{1} {1+\frac{1} {1+\frac{1} {1+\cdots} } }
$$
```

$$
    f(x) = \int_{-\infty}^\infty\hat f(\xi)\,e^{2 \pi i \xi x}\,d\xi
$$

```markdown
$$
    f(x) = \int_{-\infty}^\infty\hat f(\xi)\,e^{2 \pi i \xi x}\,d\xi
$$
```

## 短代码，Shortcodes

更多详情，请查阅 [文档](https://stack.jimmycai.com/writing/shortcodes).

### Bilibili

{{< bilibili "BV1d4411N7zD" >}}

### 腾讯视频

{{< tencent "g0014r3khdw" >}}

### YouTube

{{< youtube "0qwALOOvUik" >}}

### 自定义视频文件

{{< video "<https://www.w3schools.com/tags/movie.mp4>" >}}

### Gist

{{< gist CaiJimmy e2751a943de10b2a5b3a8a6c2120cb86 >}}

### GitLab

{{< gitlab 2589724 >}}

### 引语

{{< quote author="A famous person" source="The book they wrote" url="<https://en.wikipedia.org/wiki/Book">}}>
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
{{< /quote >}}
