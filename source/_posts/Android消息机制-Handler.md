---
title: Android消息机制(Handler)
date: 2017-07-12 20:20:05
tags: 通信
---
### 答疑解惑
- 系统为什么不允许在子线程中访问UI？
 
    Android的UI控件不是线程安全的

    增加上锁机制会导致: UI访问逻辑复杂、降低UI访问效率

- 区分线程的数据存储

    ThreadLocal
    
### 工作原理

**MessageQueue**

内部由单链表实现，主要包含两个操作：插入(enqueueMessage)和读取(next)。

**Looper**

从MessageQueue中不停查看是否有新消息，如果有新消息立即处理。

- 系统已经为主线程创建了Looper，可以使用getMainLooper获取
- 其他线程使用Looper.prepare()获取，使用Looper.loop()启动
- loop方法是一个死循环，运行在创建Looper的线程

**Handler**

负责发送和接收消息。可以通过post和send方法发送消息，post方法最终也会走入send的逻辑。

Handler工作过程:

![](1.jpg)

Handler消息处理流程:

![](2.jpg)