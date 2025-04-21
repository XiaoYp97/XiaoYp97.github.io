---
title: "Iptables"
description:
date: "2023-12-28T21:15:49+08:00"
slug: "iptables"
image: "iptables.webp"
license: false
hidden: false
comments: false
draft: false
tags: ["Linux", "防火墙"]
categories: ["Linux", "防火墙"]
---
## **iptables 是什么**

iptables 是运行在用户空间的应用软件，通过控制Linux内核netfilter模块，来管理网络数据包的处理和转发。iptables只支持处理ipv4数据包，对于ipv6数据包，则需要ip6tables。

## **iptables命令介绍**

```bash
# iptables --help
iptables v1.4.21

Usage: iptables -[ACD] chain rule-specification [options]
       iptables -I chain [rulenum] rule-specification [options]
       iptables -R chain rulenum rule-specification [options]
       iptables -D chain rulenum [options]
       iptables -[LS] [chain [rulenum]] [options]
       iptables -[FZ] [chain] [options]
       iptables -[NX] chain
       iptables -E old-chain-name new-chain-name
       iptables -P chain target [options]
       iptables -h (print this help information)
```

## **iptables基本命令**

```bash
--append  -A chain                      添加一个规则到链的末尾
--check   -C chain                      检查某一条链是否存在
--delete  -D chain                      删除匹配的链
--delete  -D chain rulenum              删除指定链的某一条规则
--insert  -I chain [rulenum]            根据给出的规则序号向所选链中插入一条或更多规则。所以，如果规则序号为1，
                                        规则会被插入链的头部。这也是不指定规则序号时的默认方式。

--replace -R chain rulenum              修改指定链中的某一条规则
--list    -L [chain [rulenum]]          列出指定链中的规则
--list-rules -S [chain [rulenum]]       打印出指定链中的规则
--flush   -F [chain]                    删除指定链中的规则
--zero    -Z [chain [rulenum]]          把指定链，或者表中的所有链上的所有计数器清零
--new     -N chain                      创建一条用户自定义链
--delete-chain   -X [chain]             删除一条用户自定义链
--policy  -P chain target               该表某条链的策略
--rename-chain  -E old-chain new-chain  修改链的名称(只有用户自定义链的名称可以被修改）
```

### **iptables选项参数**

```bash
[!] --protocol  -p proto                规则或者包检查的协议。指定协议可以是tcp、udp、icmp中的一个或全部，也可以是数值，代表这些协议中的某一个。
                                        当然也可以使用在/etc/protocols中定义的协议名。在协议名前加'!'表示相反的规则。数字0相当于all。Protocol
                                        all会匹配所有协议，而且这是缺省的选项。在和check命令结合时，all可以不被使用

[!] --source    -s address[/mask][...]  指定源地址，可以是主机名、网络名或IP地址。mask说明可以是网络掩码或清楚的数字。标志--src是这个选项的简写。

[!] --destination -d address[/mask][...] 指定目标地址。标志--dst是这个选项的简写

--jump -j target                        执行指定的动作

--goto -g chain                         跳转到指定的链

--match  -m match                       扩展匹配

--numeric     -n                        以数字的形式显示IP地址和端口

[!] --in-interface -i input name[+]     匹配由指定网络接口进入的数据包
[!] --out-interface -o output name[+]   由指定接口发出的数据包

[!] --fragment  -f                      这意味着在分片的包中，规则只询问第二及以后的片
--exact       -x                        扩展数字。显示包和字节计数器的精确值，代替用K、M、G表示的约数。这个选项仅能用于-L选项

--line-numbers                          当列表显示规则时，在每个规则前面加上行号，与该规则在链中的位置相对应。
```

#### **1. 针对 tcp 的扩展**

当 `--protocol tcp`被指定，且其他匹配的扩展未被指定时，这些扩展被装载。它提供以下选项：

```bash
--source-port [!] [port[:port]]         源端口或端口范围指定，也可以使用服务名或端口号。如果使用端口范围，若首端口号忽略则默认为0，若尾端口号忽略则
                                        默认为65535。这个选项可以简写为--sport
--destionation-port [!] [port:[port]]   目标端口或端口范围指定。这个选项可以使用--dport别名来代替

--tcp-flags [!] mask comp               匹配指定的TCP标记。第一个参数是我们要检查的标记，一个用逗号分开的列表，第二个参数是用逗号分开的标记表,是必须
                                        被设置的。标记如下：SYN ACK FIN RST URG PSH ALL NONE。例如我们有如下这条命令：
                                                iptables -A FORWARD -p tcp --tcp-flags SYN,ACK,FIN,RST SYN
                                        上面这条命令只匹配那些SYN标志被设置而ACK、FIN和RST标记没有被设置的包

[!] --syn                               只匹配那些设置了SYN位而清除了ACK和FIN位的TCP包。这些包用于TCP连接初始化时发出请求。例如，大量的这种包进入一个
                                        接口发生堵塞时会阻止进入的TCP连接，而出去的TCP连接不会受到影响。这等于：--tcp-flags SYN,RST,ACK SYN

--tcp-option [!] number                 匹配设置了TCP选项的数据包
```

#### **2. 针对 udp 的扩展**

当 `--protocol udp`被指定，且其他匹配的扩展未被指定时，这些扩展被装载。它提供以下选项：

```bash
--source-port [!] [port:[port]]         源端口或端口范围指定
--destionation-port [!] [port:[port]]   目标端口或端口范围指定
```

#### **3. 针对 ICMP 的扩展**

当 `--protocol icmp`被指定，且其他匹配的扩展未被指定时，这些扩展被装载。它提供以下选项：

```bash
--icmp-type [!] typename               这个选项允许指定ICMP类型，可以是一个数值型的ICMP类型，或者是某个由命令iptables -p icmp -h所显示的ICMP类型名
```

#### **4. 针对 mac 的扩展**

```bash
--mac-source [!] address               匹配物理地址。注意它只对来自以太设备并进入PREROUTING、FORWORD和INPUT链的包有效。
```

#### **5. 针对 limit 的扩展**

```bash
--limit rate             最大平均匹配速率：可赋的值有'/second', '/minute', '/hour', or '/day'这样的单位，默认是3/hour
--limit-burst number     待匹配包初始个数的最大值:若前面指定的极限还没达到这个数值,则概数字加1.默认值为5
```

## **iptables targets 介绍**

iptables的 `-j`选项后面对应的是要执行的target。其中有些target具有一些扩展选项，下面我们会一并介绍：

### **1. ACCEPT**

表示接收匹配的数据包

### **2. DROP**

表示丢弃匹配的数据包

### **3. REJECT**

作为对匹配的包的响应，返回一个错误的包：其他情况下与DROP 相同

此目标只适用于INPUT、FORWORD和OUTPUT，和调用这些的用户自定义链。

```bash
--reject-with type     type可以是icmp-net-unreachable、icmp-host-unreachable、icmp-port-nreachable、icmp-proto-unreachable、
                       icmp-net-prohibited或者icmp-host-prohibited，该类型会返回相应的ICMP错误信息（默认是port-unreachable）

--echo-reply           它只能用于指定ICMP ping包的规则中，生成ping的回应

--tcp-reset            可以用于在INPUT链中,或自INPUT链调用的规则，只匹配TCP协议：将回应一个TCP RST包。
```

### **4. REDIRECT**

表示重定向匹配的数据包，只适用于nat表的PREROUTING和OUTPUT，和只调用它们的用户自定义链。它修改包的目标IP地址来发送包到机器自身（本地生成的包被安置为地址127.0.0.1）

```bash
--to-ports [port-port]        指定使用的目的端口或端口范围：不指定的话，目标端口不会被修改。只能用于指定了-p tcp 或 -p udp的规则。
```

## **iptables常用命令**

```bash
# 新增规则# 指定 ip 访问指定端口
iptables -I INPUT -s <ip>  -p tcp -m tcp --dport <port> -j ACCEPT

# 允许所有ip访问指定端口
iptables -I INPUT  -p tcp -m tcp --dport <port> -j ACCEPT

# 查看所有规则
iptables -n -L

# 查看所有规则，带 序号
iptables -n -L --line-number

# 删除规则# 删除 INPUT 的 序号为 1 的规则
iptables -D INPUT 1
```
