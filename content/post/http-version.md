+++
author = "W1ndy"
title = "HTTP的演变及各版本区别"
date = "2020-08-04"
description = "HTTP的演变及各版本区别"
categories = ["怎么做?"]
tags = [
    "http tcp udp quic"
]
+++

<h4 style="color: #23D18B"> HTTP的演变 </h4>

HTTP(Hypertext transfer protocol)，即超文本传输协议，是由欧洲核子研究委员会CERN的英国工程师Tim Berners Lee发明的。最开始主要用于传递html封装过的数据。下面是HTTP的各个版本，以及对应的年限。

| 年限   | 版本     |
| ------ | -------- |
| 1991年 | HTTP 0.9 |
| 1996年 | HTTP 1.0 |
| 1997年 | HTTP 1.1 |
| 2015年 | HTTP 2.0 |
| 2019年 | HTTP 3.0 |

其中HTTP 1.1的版本是迄今为止流通最为广泛的版本。初始的[RFC 2068](https://tools.ietf.org/html/rfc2068)在1997年发布，又在1999年被[RFC 2616](https://tools.ietf.org/html/rfc2616)取代，而到了2014年，再一次被[RFC 7230](https://tools.ietf.org/html/rfc7230)/[7231](https://tools.ietf.org/html/rfc7231)/[7232](https://tools.ietf.org/html/rfc7232)......一直到[7235](https://tools.ietf.org/html/rfc7235)取代。2.0版本的发布，极大的优化了1.1的安全性和性能。3.0则是继续优化2.0，并用UDP协议取代了TCP协议。

<h4 style="color: #23D18B"> HTTP 0.9/1.0 </h4>

这两个版本最为简单。0.9版本没有请求头，只支持GET请求。1.0版本新增了HTTP版本号，有了请求头，增加了状态码，`Content-Type`可以传输文件。但是1.0版本有一个很大的性能问题，每当有一个请求时，就会新建立一次TCP连接，而且是串行的，所以响应速度很慢。

<h4 style="color: #23D18B"> HTTP 1.1 </h4>

相比于1.0，1.1对性能进行了优化，并且新增了一些东西，使协议更安全:

* 支持`pipeline`网络传输，可以在没收到响应时，继续发送请求，减少响应时间。
* 可以设置`Keepalive`，使TCP可以复用，减少了每次请求TCP都需要握手的时间。也叫HTTP的长连接。
* Header中新增了Language, Encoding, Type, Host......其中新增的Host能让服务器知道你要请求哪一个网站。
* 新增了`OPTIONS`方法，主要为了解决跨域资源共享，即CORS。
* 新增了cache control机制，使用缓存，减少等待时间，提升性能。
* 新增了分块传输编码(Chunked transfer encoding)。通常HTTP发送请求，数据都是整个发送的，`Content-Length`消息头用来表示数据的长度。而分块传输则将数据分成一系列的块，所以发送的时候不需要事先知道数据的大小，知道发送完，收到服务端的EOF标识请求结束。它的好处是允许服务器维持[HTTP持久连接](https://zh.wikipedia.org/wiki/HTTP持久连接),不需要缓冲内容计算数据长度，有利于一边压缩一边发送数据，而不是先完成压缩，得到压缩后的数据大小。

此时，HTTP 1.1协议已经支持了Websocket模型，TCP的长连接，分块传输编码，另外也加大了安全性，如使用了[TLS协议](https://zh.wikipedia.org/wiki/傳輸層安全性協定)。

<h4 style="color: #23D18B"> HTTP 2.0 </h4>

HTTP/2.0解决了请求的并行需求。另外在HTTP/1.1版本，传输数据时，消耗了CPU用来压缩数据，传输的成本比较大，HTTP/2.0解决了这个问题。和HTTP/1.1不同，2.0的协议是一个二进制协议，提升了传输效率。常见了二进制协议有RPC,thrift，TCP。

为什么二进制协议能提高传输效率？

1. 根据协议约定，省去了参数名占用的字节，缩减了数据大小。
2. 将数据类型的数据打包至相应范围内的二进制，节省空间。4bytes能表示32位的文本数据，但文本数据需要32bytes。
3. 因为事先约定，变相起到了数据加密的作用，如果第三方不知道数据协议，就没办法截取相应字节获取数据。

HTTP/2可以在一个TCP链接中并发请求多个HTTP请求，移除了HTTP/1.1的串行请求。并允许服务端在客户端放缓存，如你请求了X，X依赖于Y，我可以把Y和X一起返回给客户端，将Y放在你的本地缓存中。这个是HTTP/2.0的官方组织维护的[各语言实现](https://github.com/http2/http2-spec/wiki/Implementations)，可以看看。

<h4 style="color: #23D18B"> HTTP 3.0 </h4>

HTTP/1.1和HTTP/2.0在传输数据时，如果遇到丢包的问题，需要等待数据的回传，会导致阻塞。一旦发生了丢包，就会block剩下的所有请求。因为TCP协议是安全的传输协议，对数据的丢失，是必须要回传的。而UDP协议则不用，于是HTTP/3.0将UDP协议作为HTTP底层的协议。紧接着Google公司发布了[QUIC协议](https://zh.wikipedia.org/wiki/快速UDP网络连接)，避免了网络阻塞，使网页的传输速度加快。QUIC继承了UDP的优点，即不管顺序，不管丢包，传输速度快。也解决了UDP的不足，即数据丢包。QUIC有自己的一套数据重传和阻塞控制协议。这里要提一提TCP的BBR拥塞控制算法。BBR对网络进行建模，使其可以用媲美宽带的速度传输数据，速度比之前的TCP快2700倍，而损失仅为1%。

我们知道TCP需要3次握手，TLS也需要3次，而QUIC协议将这6次握手合并成了3次，所以QUIC是一个在UDP上的伪TCP+TLS+HTTP/2.0的多路复用协议。

QUIC协议目前还在发展中，等到他成熟，将是一个功能强大，颇为完美的协议，我们拭目以待。