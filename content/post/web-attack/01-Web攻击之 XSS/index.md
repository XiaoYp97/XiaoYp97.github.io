---
title: "Web 攻击之 XSS"
description:
date: "2024-04-01T14:48:30+08:00"
slug: "web-attack-xss"
image: ""
license: false
hidden: false
comments: false
draft: false
tags: ["网络攻击", "XSS"]
categories: ["网络攻击", "XSS"]
# weight: 1 # You can add weight to some posts to override the default sorting (date descending)
---
## XSS 是什么？

XSS (Cross-Site Scripting) 是一种网站安全漏洞，其原理是放入恶意脚本，让用户的浏览器执行网站未筛查的 JS 脚本，导致数据被窃取、表单被作弊、页面被篡改等。

> XSS 不只是技术漏洞，更是信任的破坏者!
>
> XSS 看起来是弹个窗，实际上是把钥匙偷了、钱抽了、后面还跟你说是你自愿的。

## XSS 类型

### **反射型 XSS（Reflected XSS）**

- **过程**：攻击者构造一个恶意链接，把 `<script>` 放在 URL 里 → 受害者点击链接 → 服务端把参数原样“反射”回网页 → 浏览器执行脚本。
- **关键点**：攻击代码不在服务端持久保存，靠“钓鱼链接”触发。
- **常见场景**：搜索框、跳转链接、错误提示信息。

🧠 类比记忆：像“回音墙”，你说什么它就回什么。

---

### **存储型 XSS（Stored XSS）**

- **过程**：攻击者提交脚本（比如发评论）→ 服务端保存到数据库 → 其他用户访问这条评论 → 浏览器执行脚本。
- **关键点**：脚本 **被存储下来**，访问页面时自动触发，不需要特意点击链接。
- **常见场景**：评论区、论坛帖子、个人资料页、客服聊天记录等。

🧠 类比记忆：像病毒藏在快递里，每个人收件就会感染。

---

### **DOM 型 XSS（DOM-based XSS）**

- **过程**：浏览器 JS 读取 URL 参数、Hash、Cookie 等 → 动态写入页面（innerHTML、document.write） → 没有做转义 → 执行脚本。
- **关键点**：**完全不经过服务端**，漏洞存在于前端 JS 的处理逻辑中。
- **常见场景**：单页应用（SPA）、前端框架中常见。

🧠 类比记忆：像是浏览器自己给自己挖坑，自己跳进去。

### 一个例子，三种写法对比

以攻击者想弹窗 `alert(1)` 为例：

- **反射型**：

  ```javascript
  http://site.com/search?q=<script>alert(1)</script>
  ```

- **存储型**：

  ```html
  <input name="comment" value="<script>alert(1)</script>">
  ```

  然后这个评论被存进数据库，别人一打开就触发。
- **DOM 型**（JS 代码）：

  ```javascript
  // JS 代码直接取 URL 参数插到页面
  document.getElementById('output').innerHTML = location.hash;
  // 访问： http://site.com/#<script>alert(1)</script>
  ```

### 核心区别

| 类型       | 恶意脚本位置  | 触发方式        | 是否存储 | 示例                                |
| ---------- | ------------- | --------------- | -------- | ----------------------------------- |
| 反射型 XSS | URL 参数中    | 点击恶意链接    | ❌ 否    | 搜索“\<script>alert(1)\</script>” |
| 存储型 XSS | 数据库 / 日志 | 打开启用页面    | ✅ 是    | 评论区中的 script                   |
| DOM 型 XSS | 前端 DOM 操作 | JS 代码动态触发 | ❌ 否    | JS 把 location.hash 写入 HTML       |

## XSS 的害处

### 【对个人】

- 窃取 Cookie/会话凭证：偷取登录状态，偷取账号
- 伪造页面：伪认证/伪支付码
- 强制操作：切换场景合成 CSRF
- 推光性弹窗/跳转：骗屏/色情/转账

### 【对系统】

- 数据大量被偷取
- 网站内容被乱篡改
- 添加 iframe 或动态下载构成木马

### 【对业务】

- 用户信任层层坏坏
- 规范风险（如 GDPR、网安法）
- 被搜索引擎/安全网站拦截

## 防御

### 输出编码（最核心！）

> 原则：**“信用户输入，死得很惨；编码输出，活得很稳。”**

根据输出位置做对应编码（不只是替换 `< >` 这么简单）：

| 输出位置      | 编码方式                 | 示例                        |
| ------------- | ------------------------ | --------------------------- |
| HTML 元素内容 | `htmlEncode()`         | `<div>${userInput}</div>` |
| HTML 属性值   | `attributeEncode()`    | `<a href="${userInput}">` |
| JS 中的变量   | `jsEncode()`           | `var msg = '${input}';`   |
| URL 中的参数  | `encodeURIComponent()` | `location.href = ...`     |

> ✍️ 记住一句话：**不要原样输出用户输入，必须根据上下文编码！**

### 输入过滤（辅助防线）

虽然不能完全防止 XSS，但能大幅降低攻击难度。

- 去除标签：如 `<script>`, `<iframe>` 等。
- 黑名单法有限，推荐使用白名单法：只允许部分 HTML 标签。
  - 推荐库：`DOMPurify`（前端） / `OWASP Java HTML Sanitizer`（后端）

### HTTP 安全头部设置

**Content-Security-Policy (CSP):**

定义允许加载的资源类型、域名，阻止恶意脚本执行。

```http
Content-Security-Policy: default-src 'self'; script-src 'self'
```

作用：

- 禁止内联 `<script>` 执行
- 阻止第三方 JS 加载
- 检测异常行为

### HttpOnly + Secure Cookie

**设置 Cookie 为 HttpOnly:**

防止 JS 读取 Cookie（攻击者窃取失败）。

```http
Set-Cookie: session=xxx; HttpOnly; Secure
```

### 前端框架防护机制（但别完全依赖）

- Vue、React 等默认会对数据做 HTML 转义
- **但要注意**：
  - `v-html`（Vue）/ `dangerouslySetInnerHTML`（React） 这些接口绕过了防护！
  - 千万不要对用户数据使用这些 API！

### 其他补充措施

- 表单提交加验证码：减缓自动化脚本攻击
- 日志记录 + 异常监控：配合 WAF 实时监控异常行为
- 使用 Web 安全网关 / 云 WAF：防护基础设施级别的 XSS 尝试

### 一句话总结

> **XSS 防不住的根本原因，不是框架不行，是开发者在瞎传字符串。**
>
> 所以防御的关键在于：**凡是用户输入的，都不要相信；凡是输出前的，都要处理。**
