+++
author = "W1ndy"
title = "Go设计模式-单例"
date = "2021-12-11"
description = "golang singleton 单例"
categories = ["设计模式"]
tags = [
    "design pattern"
]
+++

最近打算集中一段时间复习一下设计模式，今天复习的第一个就是`单例模式`

当我们聊到`单例模式`的时候，我们可以知道它一般有两种实现方式，即`懒汉模式`和`饿汉模式`。

<h2 style="color: #23D18B"> 懒汉模式 </h2>

```go
import "sync"

type Singleton struct{}

var (
	singleton *Singleton
  once = &sync.Once{}
)

func Get() *Singleton {
  if singleton == nil {
    once.Do(func(){
      singleton = &Singleton{}
    })
  }
  return singleton
}

```

<h2 style="color: #23D18B"> 饿汉模式 </h2>

```go
type Singleton struct{}

var singleton *Singleton

func	init() {
  singleton = &Singleton
}

func Get() *Singleton {
  return singleton
}
```

`饿汉模式`与`懒汉模式`的区别在于：

* `init`函数是在package首次被加载时执行，若`singleton`一直没有被使用，则浪费了内存，也延长了程序的加载时长
* `sync.Once`是在使用时再执行，在并发的场景下也是线程安全的

顺带，我们来看一下`sync.Once`的底层实现:

```go
package sync

import (
	"sync/atomic"
)

type Once struct {
  done uint32
  m Mutex
}

func (o *Once) Do(f func()) {
  if atomic.LoadUint32(&o.done) == 0 {
    o.doSlow(f)
  }
}

func (o *Once) doSlow(f func()) {
  o.m.Lock()
  defer o.m.Unlock()
  
  if o.done == 0 {
    defer atomic.StoreUint32(&o.done, 1)
    f()
  }
}
```

在调用Do()的时候，加载是否被执行`done`，如果没有则调用doSlow(),在加锁后重新判断done的值是否有变化。那么为什么会判断两次呢？原因是:

1. 如果有多个`goroutine`调用这个代码，其中一个获取到了锁，另外的就不会去调用f()。但是f()是否执行完是位置的
2. 所以为了保证先执行f()，再设置标志位`done`，f()一定被执行，需要对`done`做原子操作

