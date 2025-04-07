---
title: "Web攻击之 SQL注入"
description:
date: "2024-04-07T14:48:30+08:00"
slug: "01Webe))XSS"
image: ""
license: false
hidden: false
comments: false
draft: true
tags: ["网络攻击", "SQL注入"]
categories: ["网络攻击", "SQL注入"]
# weight: 1 # You can add weight to some posts to override the default sorting (date descending)

---

## SQL 注入是什么？

> SQL 注入（SQL Injection）是一种攻击方式，攻击者通过“拼接用户输入”和数据库语句，让服务器执行恶意 SQL 命令，进而**读取、修改、删除数据，甚至控制数据库主机**。

## SQL 注入是怎么产生的？

> 本质：**用户输入参与了 SQL 拼接，且没有做安全处理。**

比如你写了这样的代码：

```sql
SELECT * FROM users WHERE username = 'admin' AND password = '123456';
```

用户在登录框输入：

- 用户名：`admin' --`
- 密码：（随便填）

结果 SQL 变成了：

```sql
SELECT * FROM users WHERE username = 'admin' --' AND password = 'xxx';
```

`--` 是 SQL 中的注释符，后面的部分被忽略，攻击者绕过了密码验证，**成功登录后台**！

## SQL 注入能造成什么危害？

| 危害类别       | 具体描述                                               |
|----------------|--------------------------------------------------------|
| 登录绕过       | 修改 SQL 语句逻辑，**无需密码直接登录账号**               |
| 数据泄露       | `UNION SELECT` 等方式读取敏感数据（账号、密码、银行卡）   |
| 数据篡改       | 执行 `UPDATE`、`DELETE` 操作，**删库跑路**                 |
| 写入后门       | 某些情况下配合文件写入，**写入 Webshell**，反弹命令行     |
| 拓展攻击       | 通过数据库获取服务器权限，进一步横向移动攻击其他系统     |

## 如何防御 SQL 注入？（核心方法）

### 使用 **预编译语句（Prepared Statement）**

> **千万别直接拼接字符串！**
>
> **千万别直接拼接字符串！**
>
> **千万别直接拼接字符串！**

**错误写法（易被注入）：**

```python
sql = "SELECT * FROM users WHERE username = '" + user + "'"
```

**正确写法（参数绑定，防注入）：**

```python
cursor.execute("SELECT * FROM users WHERE username = %s", (user,))
```

各种语言通用写法推荐使用：

| 语言     | 推荐库                    |
|----------|---------------------------|
| PHP      | PDO + bindParam           |
| Python   | `cursor.execute(sql, param)` |
| Java     | `PreparedStatement`       |
| Node.js  | `mysql.format()`、ORM     |

### 输入校验 + 白名单限制

- 只允许合法字符（如用户名只能是字母、数字）
- 限制字段长度、格式（如手机号必须 11 位）

### 最小权限原则

- 数据库账号只给必要权限
- 不要用 `root` 账户连接数据库！

### 错误信息不暴露

- SQL 报错不能直接显示给用户（容易泄露表名、字段名）
- 使用统一的错误提示页面

### 使用 ORM 框架（但要小心）

ORM（如 Django ORM、Hibernate）默认防注入，但：

- 仍可能因原生 SQL 被误用而注入
- ORM 的 `extra()` / `raw()` / `execute()` 等接口使用时要小心

### WAF/IPS 网络层防御

- 使用 Web 应用防火墙拦截注入关键词
- 配合日志报警、验证码限制等方式增强防御深度

## 总结一句话

> **SQL 注入之所以可怕，是因为开发者信了用户的“输入”，数据库却把它当“命令”。**
