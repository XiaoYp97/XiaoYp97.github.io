---
title: "访问网站缺少样式"
description:
date: "2023-10-23T15:14:25+08:00"
slug: "访问网站缺少样式"
image: "unsafe_link.png"
license: false
hidden: false
comments: false
draft: false
tags: ["随记"]
categories: ["随记"]
---
## 现状

某些网站打开，因为当前网络等其他原因，无法正常访问页面，只能显示网站的基本文字，而没有图片，样式等

![stackoverflow无法显示样式](stackoverflow.png)

## 产生原因

当前网站的样式地址、静态文件地址，无法访问，或者不安全被浏览器拦截

![您的链接不是私密链接](unsafe_link.png)

## 解决办法

1. 打开无法正常访问的网站
2. F12，打开审查，切换到 `Network` tab页，刷新页面，会发现很多地址都是显示红色
   ![F12](not_access.png)
3. 选择任意一个红色的无法访问的地址，双击打开，会发现浏览器出现隐私错误提示
   ![您的链接不是私密链接](unsafe_link.png)
4. 高级，仍然访问，(或者直接在当前页面上键盘输入 `thisisunsafe`)
5. 当前页面会刷新重新访问，并正常访问
6. 这时，继续切换到 该网站 的标签页，刷新，会发现当前页面可以正常访问了
   ![正常访问](success.png)

## 猜测

猜测该类网站无法访问，是因为Chrome不信任这些静态资源地址 `自签名ssl证书`，为了安全起见，直接禁止访问了
