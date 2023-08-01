---
layout: post
title:  "CS144_lab2_tcp_receiver"
date:   2023-07-09 14:53:11 +0800
categories: jekyll update
---
## 1 概念
以TCP segment 格式为中心，理清各种含义
![](pic/tcp format.png)
### 序号相关
方向：发端 --> 收端
以单词`cat`为例.

ISN:Initial Sequence Number.  
SYN:beginning-of-stream. SYN等于ISN。
FIN:end-of-stream
SYN和FIN分别占用一个sequence number, 用于确认整个数据流是否被完整地接收了。

| 序号类型| ISN | c | a | t | FIN |
|----|----|----|----|----|----|
|seqno (32 bit)| 2^32 - 2|  2^32 - 1| 0 | 1 | 2 |
|abs seqno (64 bit)| 0| 1 | 2 | 3 | 4 |
|stream index (64 bit)| | 0| 1 | 2 | 3|
### 窗口
方向：发端 <-- 收端
收端告知发端自己剩余的存储空间，即`first unacceptable - first unassembled`。
发端在收到了窗口大小后，便知道可以发送的字节的范围——这是实现流控的基础。
### 应答
方向：发端 <-- 收端
ackno：acknowledge number. 收端窗口的开始，也可以说是未被收端收到的字节流的第一个字节的序号。

## 实验
### 如何理解`SYN received, so ackno available`?
ackno表明receiver想要接收的下一个字节的序号。
SYN收到之后，就代表传输正式开始了。
所以说ackno available.

### 实验出错点
- TCPReceiver中的fin标志位应该根据ByteStream的实际情况来，而非根据报文来确定。
- segment_receiver()函数中应该穿入stream index. 要注意在一个报文中SYN 与 payload同时
存在时，stream index 的确定。
- SYN、FIN各占一个sequence number位置，要注意一个报文中SYN与FIN同时存在时，ackno要把这两个位置都考虑进来。 
