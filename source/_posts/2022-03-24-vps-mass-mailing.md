---
title: 搬瓦工 VPS 的 Network abuse Mass Mailing 问题
key: vps-mass-mailing
tags:
- Linux
toc: true
---

今天凌晨收到搬瓦工发送的邮件，提示 VPS 服务被停止，控制台显示停止原因为 **Network abuse: Mass Mailing** ，细节信息如下：

> We have detected a large number of outgoing SMTP connections originating from this server. This usually means that the server is sending out spam.

<!--more-->

# 问题分析

可以看出，因为服务器与外界建立了过多SMTP连接，从而搬瓦工认为服务器正在发送大量的垃圾邮件，因此暂停了服务。

下图是搬瓦工在控制台给出的部分信息：

![控制台部分提示信息](/img/vps-mass-mailing/bandwagon-control-pane.png)

对于我目前购买的服务，每次出现滥用网络资源的问题会扣除100分，总分是1000分，如果在一年内扣分达到1000分，vps 服务会一直暂停到该年年底。搬瓦工给出的解决方法是在服务恢复后重新安装操作系统。

在问题出现时，我的 VPS 上部署了两个服务，一个是 Shadowsocks（Python实现），另一个是 V2Ray，都用于科学上网，V2Ray为主，Shadowsocks为辅。V2Ray 使用的协议是 WebSocket + TLS，并通过 Cloudflare 中转流量，具体细节可以参考[这篇博客](https://eveaz.com/1094.html)。

在此之前，我的 VPS 已经平稳运行了接近3年，参考了一些网上的分析，该问题应该不是因为 VPS 被人恶意访问导致的，而大概率是使用科学上网服务的客户端电脑上有恶意软件大量发送邮件，而这些请求经过了 VPS 中转，从而导致 VPS 建立了大量 SMTP 连接。

由于担心直接在原操作系统上重启后再次发生同样的问题，我直接重新安装了操作系统，因此无法通过日志定位这些邮件是经过 Shadowsocks 还是 V2Ray 发出。但我平时几乎不使用Shadowsocks，而且 V2Ray 目前有多人共享，因此大概率是经过 V2Ray 发出的。

# 问题预防

为了防止再次出现此问题，一种可能的方法是让 VPS 禁止 SMTP 的出流量。这可以通过 Linux 中的 iptables 工具实现。关于 iptables 相关介绍和使用方法可以参考 [Arch wiki](https://wiki.archlinux.org/title/iptables) 和 [CentOS wiki](https://wiki.centos.org/zh/HowTos/Network/IPTables)。以下的介绍和命令均基于 CentOS 7.9 发行版。

## iptables 使用

iptables 中的 filter table 默认包含了三种规则链：
- INPUT
	- 所有以主机为目的地的数据包
- OUTPUT
	- 所有源自主机的数据包
- FORWARD
	- 数据包的源和目的地都不是主机，但会经过主机路由

为了禁止 SMTP 的出流量，可以在 OUTPUT 和 FORWARD 中添加相应的规则（经测试发现目前 FORWARD 链中没有数据包经过，但保险起见也同样添加此规则）。

首先查看添加规则前的 iptables 信息：

```bash
[root@vps-host ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

初始的 iptables 中不包含任何规则，所有数据包均同意接收或发送，使用以下命令通过过滤25端口TCP报文的方式禁止 STMP 出站：

```bash
[root@vps-host ~]# iptables -A FORWARD -p tcp --dport 25 -j DROP
[root@vps-host ~]# iptables -A OUTPUT -p tcp --dport 25 -j DROP
```

重新查看添加规则后的 iptables：

```bash
[root@vps-host ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
DROP       tcp  --  anywhere             anywhere             tcp dpt:smtp

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DROP       tcp  --  anywhere             anywhere             tcp dpt:smtp
```

可以看出在 FORWARD 和 OUTPUT 中均已有相应规则。在 VPS 上通过 wget 命令测试连接任意 IP 的 25 端口，然后查看 iptables 的数据统计情况：

```bash
[root@vps-host ~]# wget 1.1.1.1:25
--2022-03-24 21:25:10--  http://1.1.1.1:25/
正在连接 1.1.1.1:25... ^C
[root@vps-host ~]# iptables -L -v
Chain INPUT (policy ACCEPT 2384 packets, 443K bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       tcp  --  any    any     anywhere             anywhere             tcp dpt:smtp

Chain OUTPUT (policy ACCEPT 2311 packets, 413K bytes)
 pkts bytes target     prot opt in     out     source               destination
    2   120 DROP       tcp  --  any    any     anywhere             anywhere             tcp dpt:smtp
[root@vps-host ~]#
```

可以看出在 OUTPUT 中，记录了有 2 个数据包被丢弃，说明规则应用正确。

## 规则持久化

以上命令中定义的规则在重启机器后会失效，为了能够将规则持久化，可以使用 `iptables-save` 和 `iptables-restore` 命令。

`iptables-save` 命令会将 iptables 中的规则打印到标准输出，为了持久化，可以将输入重定向到文件中：

```bash
[root@vps-host ~]# iptables-save > iptables-disable-stmp
```

这里的文件名可以任取。`iptables-restore` 命令默认会从标准输入恢复规则，后续可以使用输入重定向将该文件的规则恢复：

```bash
[root@vps-host ~]# iptables-restore < iptables-disable-stmp
```

为了可以实现开机自行加载规则，一个简单的方法是在 root 用户目录下的 .bashrc 文件中添加该恢复命令：

{% codeblock "/root/.bashrc" %}
# iptables disable SMTP
iptables-restore < /root/iptables-disable-smtp
{% endcodeblock %}

# 总结

VPS 的安全是一个不可忽视的问题，特别是用于科学上网的 VPS，因为一旦出现后不仅没法通过 Google 搜索资料，而且 Github 的访问也会出现问题，如果 IP 被封还无法使用 ssh 连接。平时使用代理时尽量不要使用全局代理，同时分享给多人使用也会提升出问题的风险。

iptables 提供了极其强大的防火墙功能，本文中介绍的内容只是针对该具体问题的解决方法，后续可以考虑添加 INPUT 规则，仅接收符合规则要求的数据包，从而进一步提升安全性。

# 参考资料

[1] https://zhuanlan.zhihu.com/p/26282070
[2] https://glglife.com/index.php/2021/09/25/ban-wa-gong-massmailing-wen-ti-jie-jue/
[3] https://www.cyberciti.biz/faq/how-to-save-iptables-firewall-rules-permanently-on-linux/
