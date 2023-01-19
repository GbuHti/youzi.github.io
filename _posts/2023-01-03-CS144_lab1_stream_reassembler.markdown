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
- 数据结果有两大类：
   - 线性结构：相邻元素使用的内存空间是连续的，如数组。一经初始化，占用的内存空间就固定了。
   - 非线性结构：相邻元素使用的内存空间是不连续的，如链表。初始化之后，可动态调整调整内存空间大小。
- 内存空间和它们的状态变量(intervral.first, intervarl.second)封装在一个元素里面，方便管理。
  - 但是每个元素只能看到自己的区间，它不知道自己扩展后，会不会延伸到后面的区间里去。 后面的区间有可能也会执行相同的动作。
- 直接开辟一个capacity的内存空间来存放未拼接的子字符串。 
  - 如何控制为拼接的字符串占用的空间为ByteStream::remaining_capacity()? --> 明明有空间，却不使用，有毛病。
  - 每个元素除了记录自己的区间，还记录一些关于这个内存空间的整体信息，比如起始序号，和结尾序号。 
  - 区间插入操作完成后，字符串必然在其中一个区间内，找到这个区间，把字符串填进去。
2. 模块设计思路
这是leetcode上的一道题，要求在一系列区间(用二元组表示)中，插入新的区间。这个题目的场景与现在要解决的问题场景非常契合。下图显示了这道题的解题思路。剩下要解决的是，stream-reassembler的存储
空间里存储的是实际的内容，而非简洁的二元组，需要考虑接下来是用二元组数组来间接管理存储区间，还是直接操作操作二元组(类似贪吃蛇，遇到一个吃一下，不断增长自身长度）。

<img src="./pic/lab1_design.png" width="400"/>

3. 按照已经给出的初始化方式
    ```c++
    StreamReassembler::StreamReassembler(const size_t capacity) : _output(capacity), _capacity(capacity) {}
    ```
    在遇到下图所示的情况时，应该把新到达的不能放入到ByteStream的字符串放在哪里？
    ![img.png](img.png)

    - 维护一个string数组，数组中所有字符串长度之和不能超过 ByteStream::remaining_capacity()。这种方法使用额外的空间来存储字符串，最坏情况下使用双倍的空间。
    - ByteStream的读操作之后，需要更新firstUnread. 更新完firstUnassembled之后，需要写入ByteStream.
    - 插入区间之后，需要更新intervals，以及 ByteStream.
    - 总的来说，这是一种内存与内存的管理分离的架构。写操作先影响状态量，进而影响存储空间；读操作先影响存储空间，后影响状态量。 
    - 可是用已经发生变化的 intervals，如何去管理内存空间？ --> 在发生变化的同时，及时调整内存空间, 边收边调的策略。
