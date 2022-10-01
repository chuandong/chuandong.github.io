---
title: TCP/IP学习
date: 2022-03-27 15:50:49
description: TCP/IP
tags: TCP/IP
---

### 1. OSI七层参考模型

![image-20220409223051378](C:\Users\11655\AppData\Roaming\Typora\typora-user-images\image-20220409223051378.png)

1. **应用层**

   ​	

2. **表示层**

3. **会话层**

4. **传输控制层**

   TCP、UDP 

   三次握手主要规避网络传输延时而导致服务器开销的问题；

   TCP 面向连接的，可靠的协议：

   ​	连接表示三次握手====》数据传输====》四次分手

   ​	1) Client-->syn=1,Seq=X-->Server

   ​	2) Server-->syn=1,ack=X+1,Seq=Y-->Client

   ​	3) Client-->ack=Y+1,Seq=Z-->Server

   ​	4) 客户端和服务端都开辟资源，连接才算成功

5. **网络层**

6. **链路层**

7. **物理层**

### 2.TCP/IP网络协议栈

**应用层**

​		功能：应用程序，协议有：FTP、Telnet、TFTP、NFS

**传输层**

​		功能：进程到进程，协议有：TCP、UTP

**网络层**

​		功能：主机到主机，协议有：ICMP、IP、IGMP

**链路层**

​		功能：设备到设备，协议有：ARP、RARP

### 3.TCP/IP三次握手和四次挥手

![](E:\CodeProject\Hexo\source\_posts\image\TCP-Header-01.jpg)

​														TCP头格式

注意上图中的四个非常重要的东西：

- **Sequence Number**是包的序号，**用来解决网络包乱序（reordering）问题。**
- **Acknowledgement Number**就是ACK——用于确认收到，**用来解决不丢包的问题**。
- **Window又叫Advertised-Window**，也就是著名的滑动窗口（Sliding Window），**用于解决流控的**。
- **TCP Flag** ，也就是包的类型，**主要是用于操控TCP的状态机的**

<img src="E:\CodeProject\Hexo\source\_posts\image\tcp_open_close.jpg" style="zoom:50%;" />

为什么建链接要3次握手，断链接需要4次挥手？

- **对于建链接的3次握手，**主要是要初始化Sequence Number 的初始值。通信的双方要互相通知对方自己的初始化的Sequence Number（缩写为ISN：Inital Sequence Number）——所以叫SYN，全称Synchronize Sequence Numbers。也就上图中的 x 和 y。这个号要作为以后的数据通信的序号，以保证应用层接收到的数据不会因为网络上的传输的问题而乱序（TCP会用这个序号来拼接数据）。

- **对于4次挥手，**其实你仔细看是2次，因为TCP是全双工的，所以，发送方和接收方都需要Fin和Ack。只不过，有一方是被动的，所以看上去就成了所谓的4次挥手。如果两边同时断连接，那就会就进入到CLOSING状态，然后到达TIME_WAIT状态。下图是双方同时断连接的示意图（你同样可以对照着TCP状态机看）：
