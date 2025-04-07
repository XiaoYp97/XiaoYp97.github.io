---
title: "Certbot免费申请SSL证书"
description:
date: "2023-12-19T20:15:06+08:00"
slug: "Certbot免费申请SSL证书"
image: "certbot.png"
license: false
hidden: false
comments: false
draft: false
tags: ["证书", "Linux"]
categories: ["证书"]
---

## certbot简介

本质上来说，[certbot](https://github.com/certbot/certbot) 就是一个 ACME client，这也是 [Let’s Encrypt](https://letsencrypt.org/getting-started/) 官网推荐的签发证书的方式，适用于对自己的 domain 具有 shell 访问能力的情况，使用所谓的 ACME 协议来自动化的签发证书，很大程度上简化了证书签发的步骤。

## 安装步骤

以`Ubuntu`为例，安装certbot

```bash
sudo apt-get update
sudo apt-get install certbot python3-certbot-nginx
```

## 为nginx生成证书

```bash
sudo certbot --nginx -d www.yourdomain.com
```

将`www.yourdomain.com`替换为您的实际域名。Certbot将自动配置Nginx以使用生成的证书。

> 证书将在90天后过期，因此我们需要设置自动续期。Certbot包含一个名为certbot renew的命令，用于检查证书是否需要更新，如果需要，则自动更新它们。

### 自动续期

为了设置自动续期，我们将创建一个定时任务（cron job）。

```bash
crontab -e

# 每月1号执行一次
0 0 1 */1 * /usr/bin/certbot renew --quiet
```
