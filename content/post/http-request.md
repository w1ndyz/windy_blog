+++
author = "W1ndy"
title = "一个HTTP请求的整个流程"
date = "2020-07-31"
description = "http请求"
categories = ["怎么做?"]
tags = [
    "http tcp/ip udp dns"
]
+++

<h3 style="color: #23D18B"> 问题 </h3>

当发出一个HTTP请求时，具体的请求过程是怎么样的呢？

假如我们访问了`https://www.google.com`,它会经历什么样的历程呢？

<h3 style="color: #23D18B"> 解答 </h3>

我们知道TCP/IP协议。依靠遵循TCP协议的两台主机，可以在不可靠的网络传输中得到可靠的数据传输能力。而HTTP的五层模型可以很好的理解计算机网络，他们分别是应用层，传输层，网络层，数据链路层和物理层。

![](https://raw.githubusercontent.com/w1ndyz/windy-img/master/img/http-layer.png)

常见的应用层协议: HTTP, SMTP, FTP, TELNET, SNMP

常见的传输层协议: TCP, UDP

常见的网络层协议: IP, ICMP, ARP

<h4 style="color: #23D18B">  HTTP请求 </h4>

HTTP是应用层的协议。它配合TCP/IP使用。

一个HTTP请求报文分为请求报文和响应报文。这两种报文都由三个部分组成: request line(请求开始行), header(头部), data(数据)。

>请求行由：方法、[空格]、URL、[空格]、HTTP版本组成

常见方法包括:GET、POST、PUT、DELETE、OPTIONS......

>响应开始行由: HTTP版本、[空格]、状态码组成

常见状态码都是3位数字

| 状态码 | 说明                            |
| ------ | ------------------------------- |
| 1xx    | 表示通知，如请求收到或正在处理  |
| 2xx    | 表示成功，如200                 |
| 3xx    | 表示重定向，如304               |
| 4xx    | 表示客户端问题，如路由不存在404 |
| 5xx    | 表示服务器问题，如503           |

应用层是最接近我们用户的一层。应用层就是用户主机上的一个个进程。当用户在浏览器中输入google的地址时，浏览器会使用DNS协议返回google的物理IP地址，接着应用层会创建一个TCP socket，将这个请求封装成一个http的数据包，并将这个数据包放入socket中。

https是什么？

简单讲就是http的安全版本。基于SSL(Secure Socket Layer), 在http的基础上通过传输加密和身份认证保证了传输过程中的安全性。

<h4 style="color: #23D18B"> DNS </h4>

DNS是一个应用层的协议，它选择的传输层协议大部分是UDP协议，但是也有少量的TCP协议，在需要数据传输准确性和稳定性时。因为是UDP协议，所以域名解析过程很迅速。在DNS服务器上存储了域名和域名所对应的IP地址。

常见的顶级域DNS服务器有.com, .gov, .org, .net, .edu......

本地的DNS服务器往往是具有缓存的功能的，通常会保存两天内的访问记录，大部分时候你是感觉不到域名解析的过程的，因为解析时会从缓存中读取。

![](https://raw.githubusercontent.com/w1ndyz/windy-img/master/img/dns-layer.png)

<h4 style="color: #23D18B"> TCP/UDP </h4>

socket分为TCP/UDP这两种协议的套接字，而这两种协议便处于传输层。往往应用层会有很多请求，所以传出层会有很多的socket，他们通过协议头中的IP，端口号进行标识，用来知道传输层收到的响应请求对应哪一个socket。

这里有对[TCP](http://xxx.com/post/tcp-why)的详细理解，这里是对[UDP](http://xxx.com/post/udp-header)的详细理解，就不一一赘述了。

<h4 style="color: #23D18B"> IP </h4>

IP协议是处于网络层的协议，解决的是两台不处于同一子网的主机相互通信的问题。

IPv4的地址长度为4个8字节，即32bit。而IPv6的地址长度为原来的4倍，即128bit。它为了解决IPv4地址耗尽的问题，这里就不细说了。

<h4 style="color: #23D18B"> MAC地址 </h4>

在网络层的数据通信问题解决后，需要在链路层将IP数据包封装成以太网帧。基本结构([802.3](https://zh.wikipedia.org/wiki/以太网帧格式)))包含了:前导码, 帧开始符, MAC目标地址，MAC源地址，类型长度以及数据。其中MAC地址是硬件的唯一标识，全世界任意一台主机的MAC地址都是不同的。

所以之前有人问，都有IP地址了，为什么还需要MAC地址呢？

原因是IP在别人不用的时候可以共享出去，但是MAC地址是唯一的。以太网采取了`广播`的方式向任意主机发送报文。如果一个请求如果不是自己主机发出的，获得相应的报文是会被抛弃掉的。

<h4 style="color: #23D18B"> 总结 </h4>

以上就是一个http请求的整个过程，浏览器就好比一个客户端，当你输入URL时，浏览器会先去请求DNS服务器，获取域名对应的IP地址，再通过IP地址找到对应的服务器，并建立TCP/UDP连接，等待浏览器发送完HTTP请求包后，服务器开始处理请求(request)，并返回响应包(response)。客户端收到后，渲染返回主体(body)，等收到全部内容后断开TCP连接。



