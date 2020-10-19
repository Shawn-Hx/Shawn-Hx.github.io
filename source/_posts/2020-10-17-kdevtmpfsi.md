---
title: kdevtmpfsi 进程占用 CPU 问题
key: kdevtmpfsi
tags:
- Arch
- Linux
toc: true
---

今天用实验室电脑（Arch Linux 系统）训练模型时发现训练一个 epoch 的时间比往常要慢得多。使用`top`指令发现有名为 **kdevtmpfsi** 的进程 CPU 占用率达到 500%，且直接使用 `kill -9 pid` 杀死后还会重复出现。

```shell
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND    
 619970 huangxi+  20   0 3025392   2.3g   2712 S 582.4  14.8  47:36.78 kdevtmpfsi
```

<!--more-->

## 问题排查

根据网上资料[1]显示，*kdevtmpfsi* 进程为挖矿程序。

通过 `ps -ef | grep kdevtmpfsi` 可以获知该进程的 pid，然而在用 `kill -9 pid` 杀死进程后不久又会重新出现，因此必然是有另一个程序会不断重新启动该进程。

### 查看定时任务

根据分析，有可能是恶意程序在 Linux 中注册了定时任务。查看 [Cron Arch wiki](https://wiki.archlinux.org/index.php/cron) 得知，Arch 中默认使用 [systemd/Timers](https://wiki.archlinux.org/index.php/Systemd/Timers) 管理定时任务。

通过 `systemctl list-timers` 查看所有启动的定时任务，输出如下：

```shell
NEXT                        LEFT     LAST                        PASSED       UNIT                         ACTIVATES                     
Sun 2020-10-18 00:00:00 CST 6h left  Sat 2020-10-17 00:00:46 CST 17h ago      man-db.timer                 man-db.service                
Sun 2020-10-18 00:00:00 CST 6h left  Sat 2020-10-17 00:00:46 CST 17h ago      shadow.timer                 shadow.service                
Sun 2020-10-18 14:03:53 CST 20h left Sat 2020-10-17 14:03:53 CST 3h 44min ago systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service

3 timers listed.
Pass --all to see loaded but inactive timers, too.
```

并没有发现异常的定时任务。

### 守护进程

由此可以怀疑是有守护进程在不断重启 *kdevtmpfsi* 进程。使用 `systemctl status pid` （pid 为 kdevtmpfsi 进程id）查看后发现， */var/tmp/kinsing* 与 */tmp/kdevtmpfsi* 进程处于同一 CGroup 中。

分别使用  `kill -9` 杀死这两个进程并删除系统中对应的文件，之后 *kdevtmpfsi* 进程就不会再次出现了。

## 原因分析

从网络上资料来看，基本上都是服务器遭受了该进程的影响，而我的环境是实验室里的PC机，并且并没有打开 sshd 服务。

之后分析发现，被攻击的原因很可能是我在电脑上开了一晚上的 Flink 本地集群。在[3]的其中一个回答中也表示是在启动了 Flink 集群后遇到了这个问题。Flink 集群启动后默认会在 8081 端口部署 web UI，通过 web UI 可以提交用户自定义 Job 在集群中执行。因此攻击者应该是通过 Flink web UI 暴露的端口实施了攻击。

通过查看 Flink standalonesession 的 log，发现的确在凌晨有记录一些异常信息：

{% codeblock "flink-huangxiao-standalonesession-0-huangxiao-lab.log" lang:java >folded %}
2020-10-17 00:09:40,372 WARN  org.apache.flink.runtime.dispatcher.DispatcherRestEndpoint   [] - Unhandled exception
java.io.IOException: Connection reset by peer
	at sun.nio.ch.FileDispatcherImpl.read0(Native Method) ~[?:?]
	at sun.nio.ch.SocketDispatcher.read(SocketDispatcher.java:39) ~[?:?]
	at sun.nio.ch.IOUtil.readIntoNativeBuffer(IOUtil.java:276) ~[?:?]
	at sun.nio.ch.IOUtil.read(IOUtil.java:233) ~[?:?]
	at sun.nio.ch.IOUtil.read(IOUtil.java:223) ~[?:?]
	at sun.nio.ch.SocketChannelImpl.read(SocketChannelImpl.java:358) ~[?:?]
	at org.apache.flink.shaded.netty4.io.netty.buffer.PooledByteBuf.setBytes(PooledByteBuf.java:247) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.buffer.AbstractByteBuf.writeBytes(AbstractByteBuf.java:1140) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.socket.nio.NioSocketChannel.doReadBytes(NioSocketChannel.java:347) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:148) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:697) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:632) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:549) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:511) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:918) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at java.lang.Thread.run(Thread.java:834) [?:?]
2020-10-17 00:09:41,896 ERROR org.apache.flink.runtime.rest.handler.legacy.files.StaticFileServerHandler [] - Caught exception
java.io.IOException: Connection reset by peer
	at sun.nio.ch.FileDispatcherImpl.read0(Native Method) ~[?:?]
	at sun.nio.ch.SocketDispatcher.read(SocketDispatcher.java:39) ~[?:?]
	at sun.nio.ch.IOUtil.readIntoNativeBuffer(IOUtil.java:276) ~[?:?]
	at sun.nio.ch.IOUtil.read(IOUtil.java:233) ~[?:?]
	at sun.nio.ch.IOUtil.read(IOUtil.java:223) ~[?:?]
	at sun.nio.ch.SocketChannelImpl.read(SocketChannelImpl.java:358) ~[?:?]
	at org.apache.flink.shaded.netty4.io.netty.buffer.PooledByteBuf.setBytes(PooledByteBuf.java:247) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.buffer.AbstractByteBuf.writeBytes(AbstractByteBuf.java:1140) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.socket.nio.NioSocketChannel.doReadBytes(NioSocketChannel.java:347) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:148) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:697) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:632) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:549) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:511) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:918) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at java.lang.Thread.run(Thread.java:834) [?:?]
2020-10-17 03:17:44,446 WARN  org.apache.flink.runtime.webmonitor.handlers.JarRunHandler   [] - Configuring the job submission via query parameters is deprecated. Please migrate to submitting a JSON request instead.
2020-10-17 03:17:44,464 INFO  org.apache.flink.client.ClientUtils                          [] - Starting program (detached: true)
2020-10-17 03:17:44,486 ERROR org.apache.flink.runtime.webmonitor.handlers.JarRunHandler   [] - Exception occurred in REST handler: No jobs included in application.
2020-10-17 06:26:13,712 WARN  org.apache.flink.runtime.dispatcher.DispatcherRestEndpoint   [] - Unhandled exception
java.io.IOException: Connection reset by peer
	at sun.nio.ch.FileDispatcherImpl.read0(Native Method) ~[?:?]
	at sun.nio.ch.SocketDispatcher.read(SocketDispatcher.java:39) ~[?:?]
	at sun.nio.ch.IOUtil.readIntoNativeBuffer(IOUtil.java:276) ~[?:?]
	at sun.nio.ch.IOUtil.read(IOUtil.java:233) ~[?:?]
	at sun.nio.ch.IOUtil.read(IOUtil.java:223) ~[?:?]
	at sun.nio.ch.SocketChannelImpl.read(SocketChannelImpl.java:358) ~[?:?]
	at org.apache.flink.shaded.netty4.io.netty.buffer.PooledByteBuf.setBytes(PooledByteBuf.java:247) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.buffer.AbstractByteBuf.writeBytes(AbstractByteBuf.java:1140) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.socket.nio.NioSocketChannel.doReadBytes(NioSocketChannel.java:347) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:148) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:697) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:632) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:549) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:511) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:918) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at java.lang.Thread.run(Thread.java:834) [?:?]
2020-10-17 06:26:15,227 ERROR org.apache.flink.runtime.rest.handler.legacy.files.StaticFileServerHandler [] - Caught exception
java.io.IOException: Connection reset by peer
	at sun.nio.ch.FileDispatcherImpl.read0(Native Method) ~[?:?]
	at sun.nio.ch.SocketDispatcher.read(SocketDispatcher.java:39) ~[?:?]
	at sun.nio.ch.IOUtil.readIntoNativeBuffer(IOUtil.java:276) ~[?:?]
	at sun.nio.ch.IOUtil.read(IOUtil.java:233) ~[?:?]
	at sun.nio.ch.IOUtil.read(IOUtil.java:223) ~[?:?]
	at sun.nio.ch.SocketChannelImpl.read(SocketChannelImpl.java:358) ~[?:?]
	at org.apache.flink.shaded.netty4.io.netty.buffer.PooledByteBuf.setBytes(PooledByteBuf.java:247) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.buffer.AbstractByteBuf.writeBytes(AbstractByteBuf.java:1140) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.socket.nio.NioSocketChannel.doReadBytes(NioSocketChannel.java:347) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:148) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:697) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:632) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:549) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:511) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:918) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at java.lang.Thread.run(Thread.java:834) [?:?]
2020-10-17 16:47:29,283 WARN  org.apache.flink.runtime.dispatcher.DispatcherRestEndpoint   [] - Unhandled exception
java.io.IOException: Connection reset by peer
	at sun.nio.ch.FileDispatcherImpl.read0(Native Method) ~[?:?]
	at sun.nio.ch.SocketDispatcher.read(SocketDispatcher.java:39) ~[?:?]
	at sun.nio.ch.IOUtil.readIntoNativeBuffer(IOUtil.java:276) ~[?:?]
	at sun.nio.ch.IOUtil.read(IOUtil.java:233) ~[?:?]
	at sun.nio.ch.IOUtil.read(IOUtil.java:223) ~[?:?]
	at sun.nio.ch.SocketChannelImpl.read(SocketChannelImpl.java:358) ~[?:?]
	at org.apache.flink.shaded.netty4.io.netty.buffer.PooledByteBuf.setBytes(PooledByteBuf.java:247) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.buffer.AbstractByteBuf.writeBytes(AbstractByteBuf.java:1140) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.socket.nio.NioSocketChannel.doReadBytes(NioSocketChannel.java:347) ~[flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:148) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:697) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:632) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:549) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:511) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:918) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at org.apache.flink.shaded.netty4.io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) [flink-dist_2.11-1.11.2.jar:1.11.2]
	at java.lang.Thread.run(Thread.java:834) [?:?]
{% endcodeblock %}

所以说，不管是在个人 PC 还是服务器上部署 Flink 集群后，都是很有可能受到攻击的，需要提前做好准备和防御的措施。



**参考资料**

[1] https://bbs.huaweicloud.com/blogs/149758
[2] https://blog.csdn.net/u014589116/article/details/103705690
[3] https://stackoverflow.com/questions/60151640/kdevtmpfsi-using-the-entire-cpu
