---
layout: post
title:  "CS144_lab1_assembler"
date:   2023-01-03 14:53:11 +0800
categories: jekyll update
---
## 1 实验目的
完成一个模块——stream_reassembler，这个模块的功能是，子字符串在到达接收端后，stream_reassembler能够把这些子字符串拼接成完整的
长字符串。子字符串到达时，携带的额外信息包括一个序号，用来表示子字符串第一个字符在完整字符串中的序号，以及一个标志位，用来标志该字符串
的最后一个字符是否是EOF。

### 设计
[firstUnread, firstUnassembled) 代表重新拼接好的存储在ByteStream中的未被读取的字符串；

[firstUnassembled, firstUnacceptable) 代表被re-assembler接收到的放在辅助存储空间内的字符串, 注意，在这一段
内存空间中，存在这多个字符串，它们彼此之间是不连续的；新到达的字符串如果不能立马被放入[firstUnrad, firstUnassembled), 需要考虑
如何将它和[firstUnassembled, firstUnacceptable)中离散的字符串进行合并。

其中的变量满足约束条件： firstUnacceptable - firstUnread == capcacity

<img alt="img.png" src="pic/re-assemble_memory_state.png"/>

在实际实现的时候，这几个变量需要由re-assembler来维护。

#### case 1
下面研究如何将一个序号为seq, 长度为len的字符串插入到ByteStream中去。
```mermaid
graph LR
    A{seq <= firstUnassembled}
    A -- yes --> B{seq + len < firstUnassembled}
        B -- yes --> C["丢弃"]
        B -- no --> D{"seq + len < firstUnacceptable"}
            D -- yes --> E["接受[firstUnassembled, seq + len]"] --> F["更新fisrtUnassembled"] 
                F --> I{"seq+len是否落在了其它子字符串区间内"}
                    I --yes--> J["接收其它子字符串区间"] --> K["更新firstUnassembled"] --> M
                    I --no--> L["当前字符串处理结束"]
            D -- no --> G["接收[firstUnassembled, firstUnacceptable)"] --> F 
                F --> M["合并字符串区间"]
    A --no--> M
```

### 难点
#### 1 维护[firstUnassembled, firstUnacceptable)之间的离散字符串区间。
1. 数据结构的要求
给定一个序列号区间，在该区间内有若干子区间，子区间之间彼此不连续。新的区间加入进来后，判断和其它子区间的关系，有完全重叠、部分重叠、完全不重叠三种关系。
2. 模块设计思路
![](./pic/lab1_design.png)