---
title: socket套接字
updated: 2025-03-09T14:18:59
created: 2025-03-09T14:03:39
---

socket 是一个编程接口，作用是用来实现网络上不同主机的应用程序进行<strong style="color:red">双向通信</strong>。

套接字被当成是一种特殊的文件描述符，使用套接字实现网络通信可以使用 read/write，如：客户端可以用 write 发送网络数据，服务端可以使用 read 接受数据

要通过互联网进行通信，至少需要`一对套接字`，一个运行再客户端（Client socket)，一个运行在服务端（server socket）

socket 分成三种类型：<br>
<strong>（1）流式套接字（SOCK_STREAM) </strong>

用于提供面向连接、可靠的数据传输服务

主要针对于**传输层协议为 TCP 协议的应用**

<strong>（2）数据报套接字（SOCK_DGRAM)</strong>

提供一种无连接的服务（不能保证数据传输的可靠性）

<strong>（3）原始套接字（SOCK_RAM)</strong>

可以直接跳过传输层读取没有处理的 IP 数据包
