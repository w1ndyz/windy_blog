+++
author = "W1ndy"
title = "如何处理秒杀"
date = "2020-07-01"
description = "秒杀"
categories = ["怎么做?"]
tags = [
    "秒杀"
]
+++


一般说起秒杀，都会觉得是个比较麻烦，比较有技术含量。我们可以看看如何去处理好秒杀带来的问题。

## 秒杀会带来什么问题
假如像双十一那样，每个商品在某个时刻，同时面对几百万用户的请求，同时下单去抢某一件商品。几百万的请求同时并发会导致网站瞬间就崩溃了。
   * 一方面是用户的请求导致我们的网络带宽不够
   * 另一方面是如果要扛住这几百万请求，我们需要非常多的机器

不仅如此，当所有请求集中在数据库的同一条数据上时，无论如何分库分表，或者采用分布式数据库都没有用，因为面对的是单条的热点数据。

## 一般的秒杀流程
"秒杀"事实上是商家的一种促销策略，以非常低的价格销售某个商品。如：1元卖iphone，仅100台，先到先得，于是来了100万人抢购。

一般秒杀的页面是这样的：
1. 秒杀的前端页面，在还没有开始的时候，有一个置灰的倒计时按钮。
2. 倒计时时间到了后，按钮会激活点亮，用户可以点击按钮下单抢购。
3. 一般来说需要填写一个验证码，防止是机器人。

后台需要面对的情况：
1. 面对前端请求，先要判断抢购时间是否到了。
2. 每次询问时，后端会返回一个校准时间，用来对前端时间的校准。
3. 时间到了后，会返回前端一个可以供来抢购的URL。
4. 前端将URL至于按钮上，以供跳转。
5. 抢到了，就进入支付流程。
6. 如果没货了，则返回秒杀活动结束。

## 秒杀的解决方案
很明显，要让 100 万用户能够在同一时间打开一个页面，这个时候，我们就需要用到 CDN 了。数据中心肯定是扛不住的，所以，我们要引入 CDN。

在 CDN 上，这 100 万个用户就会被几十个甚至上百个 CDN 的边缘结点给分担了，于是就能够扛得住。然后，我们还需要在这些 CDN 结点上做点小文章。

一方面，我们需要把小服务部署到 CDN 结点上去，这样，当前端页面来问开没开始时，这个小服务除了告诉前端开没开始外，它还可以统计下有多少人在线。每个小服务会把当前在线等待秒杀的人数每隔一段时间就回传给我们的数据中心，于是我们就知道全网总共在线的人数有多少。

假设，我们知道有大约 100 万的人在线等着抢，那么，在我们快要开始的时候，由数据中心向各个部署在 CDN 结点上的小服务上传递一个概率值，比如说是 0.02%。

于是，当秒杀开始的时候，这 100 万用户都在点下单按钮，首先他们请求到的是 CDN 上的这些服务，这些小服务按照 0.02% 的量把用户放到后面的数据中心，也就是 1 万个人放过去两个，剩下的 9998 个都直接返回秒杀已结束。

于是，100 万用户被放过了 0.02% 的用户，也就是 200 个左右，而这 200 个人在数据中心抢那 100 个 iPhone，也就是 200 TPS，这个并发量怎么都应该能扛住了。

<span style="color:green">Tips</span>: [什么是CDN?](https://en.wikipedia.org/wiki/Content_delivery_network)

这就是整个“秒杀”的技术细节，是不是有点不敢相信？

## 更多思考
我们可以看到，解决秒杀这种特定业务场景，可以使用 CDN 的边缘结点来扛流量，然后过滤用户请求（限流用户请求），来保护数据中心的系统，这样才让整个秒杀得以顺利进行。

那么，如果我们像双 11 那样，想尽可能多地卖出商品，那么就不像秒杀了。这是要尽可能多地收订单，但又不能超过库存，其中还有大量的银行支付，各大仓库的库存查询和分配，这些都是非常慢的操作。为了保证一致性，还要能够扛得住像双 11 这样的大规模并发访问，那么，应该怎么做呢？

使用秒杀这样的解决方案基本上不太科学了。这个时候就需要认认真真地做高并发的架构和测试了，需要各个系统把自己的性能调整上去，还要小心地做性能规划，更要把分布式的弹力设计做好，最后是要不停地做性能测试，找到整个架构的系统瓶颈，然后不断地做水平扩展，以解决大规模的并发。

