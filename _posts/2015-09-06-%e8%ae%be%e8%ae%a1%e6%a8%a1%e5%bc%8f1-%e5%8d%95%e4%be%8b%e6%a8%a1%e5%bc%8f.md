---
id: 198
title: '设计模式(1)&#8211;单例模式'
date: 2015-09-06T00:21:44+00:00
author: Tango
layout: post
guid: http://wtango.com/?p=198
permalink: '/%e8%ae%be%e8%ae%a1%e6%a8%a1%e5%bc%8f1-%e5%8d%95%e4%be%8b%e6%a8%a1%e5%bc%8f/'
tags:
  - 设计模式
---
- [模式适用性](#模式适用性)
- [模式结构](#模式结构)
- [实现](#实现)

单例模式(Singleton)是软件开发中最常用的设计模式，也是比较简单的设计模式。
  
对于一些类来说，我们只需要一个实例，有时候这样的特性很有作用。比如一个王国，只有一个国王，那么国王就可以看作为单例模式，任何时候全国臣民都只听这个国王发布号令，不能有两个国王发布号令。
  
**简单说来，单例模式（也叫单件模式）的作用就是保证在整个应用程序的生命周期中，任何一个时刻，单例类的实例都只存在一个（当然也可以不存在）。**

<!--more-->

## 模式适用性

  * 当类只能有一个实例而且客户可以从一个众所周知的访问点访问它时。
  * 当这个唯一实例应该是通过子类化可扩展的,并且客户应该无需更改代码就能使用一个扩展的实例时。

## 模式结构

![](../wp-content/uploads/2015/09/singleton.png)


## 实现

```c
class Singleton{

public:
   static Singleton* Instance();

private:
   static Singleton* _instance;
   Singleton();
};

Singleton* Singleton::_instance = 0;

Singleton* Singleton::Instance(){

   if(_instance == 0) {
      _instance = new Singleton();
   }

   return _instance;

}
```

Singleton的构造函数是私有的，对于使用者只能通过Instance静态函数获得这个单例。