# TCP/IP详解

TCP/IP协议簇是一个四层协议体系系统,自底而上分别是`数据链路层` `网络层` `传输层` `应用层`,
每一层通过各自的协议来实现各自不同的功能,并且上层的协议使用下层协议提供的服务

网络层 | 使用的协议 | 描述
------------- | -------- | -------------------------------
数据链路层 | ARP,RARP | 物理传输媒介
网络层 | IP, ICMP | 内核空间
传输层 | TCP,UDP | 
应用层 | HTTP,DNS | 用户空间

## TCP/IP协议簇

### 数据链路层

数据链路层实现了网卡接口的网络驱动程序,以处理数据在物理媒介上(如：以太网,令牌环)的传输. 不同的物理网络拥有不同的物理电器特性,网络驱动程序隐藏了这些细节,未上层协议提供一个统一的对接接口.

数据链路层常用的协议有`ARP`,`RARP`.他们实现了IP地址和机器物理地址(通常是MAC地址, 802.1无线网络,以太网都使用MAC地址)之间的相互转换

## IP协议

IP协议是TCP/IP协议簇的动力，它为上层协议提供无状态、无连接、不可靠的服务

## TCP协议

传输层的协议有两个`TCP`协议和`UDP`协议。TCP相对UDP协议的特点是面向连接、字节流和可靠传输

使用TCP通信的双方必选先建立连接，然后才能开始数据的读写

TCP协议的连接是一对一，所以基于广播和多播（目标是多个主机地址）的应用程序不能使用TCP服务，无连接协议UDP更适合广播和多播