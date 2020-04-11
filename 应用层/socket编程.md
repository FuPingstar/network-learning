**应用编程接口API就是应用进程的控制权和操作系统控制权相互转换的一个系统调用接口**



1.几种典型的应用编程接口

+ Berkeley Unix操作系统定义了一种API，成为套接字接口（scoket interface）.简称套接字
+ 微软公司在其操作系统中采用了套接字接口API，形成了一个稍有不同的API，并称之为Windows Socket Interface。
+ AT&T为其UNIX系统V定义了一种API，简写为TLI（Transport Layer Interface）



##### Socket API

1. 最初设计
   + 面向BSD UNIX-Berkley
   + 面向TCP/IP协议栈接口
   + 目前，SocketAPI是工业标准，绝大多数的操作系统都支持。是Internet网络应用最典型的API接口。
2. 通信模型
   + 客户/服务器（C/S）