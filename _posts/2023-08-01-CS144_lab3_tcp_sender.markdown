---
layout: post
title:  "CS144_lab3_tcp_sender"
date:   2023-08-01 07:35:11 +0800
categories: jekyll update
---
## 1 实验目的
根据receiver提供的ackNo与window, sender将ByteStream转变为Segment发送出去。
**如果发送出去的Segment没有及时被确认(in-flight)，需要进行重传**。

## 2 实验原理
- ackNo: 代表第一个receiver没有接收到的序号，即下一个receiver想要接收到的序号
- window: 代表receiver还能接收多少数据，即receiver的剩余空间大小

## 3 实验步骤
第一行代码怎么写？写什么？
如果是自己在做设计，划分模块，给出来接口，
- 主干功能是发送segment
- 其次是在主干功能的基础上添加重传机制
所以第一行代码先写发送segment的主干代码。

每个segment的重传次数、超时时间，应该怎么记录？每个segment都要有一个计时器吗？
- 重传是将所有在RTO时间内未被应答的**最老的**segment再次发送出去。就是说周期性地根据ackno判断发送缓存
里的**最老的**的segment是否需要重发。所以重传不是segment粒度的，是TcpSender粒度的。

收到应答之后，部分segment被确认收到，还有一部分新的segment没有被确认，该怎么处理计数器？
- 新的应答被收到之后，如果还有一部分outstanding segment未被确认，则重启计数器。

主干的fill_window()功能和处理ack_received()的功能是不是要分成两个线程来做？
- 目前在测试框架中，在所有动作，比如用户向byteStream写入数据、收到ack、关闭byteStream、超时时间到达，都会调用一次fill_window()

把syn和fin占用sequence number 空间的设计目的是什么？
- 

需不要再增加一个标志位，用于标志发送已经全部完成？
- 本质是没有使用状态机

## 4 总结
### 4.1 实用技巧
- ByteStream向Segment的转换过程中涉及的拷贝和数据搬运是如何实现的？
  - ByteStream内部使用了一个队列，队里内部维护一块malloc出来的内存，大小为capacity。
  提供了常见的访存方法。
- 状态机图和流程图可以帮助理清程序的层次结构和逻辑，把握整体框架
  - https://app.diagrams.net/#G10FHZB1F48J4KblSxwFXHDhQFbZ6H7nce