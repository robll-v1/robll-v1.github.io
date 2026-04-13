---
title: "Project #3 – Query Execution"
date: 2025-11-03
draft: false
categories: ["数据库"]
tags: ["CMU 15-445", "Query Execution"]
---

日期：2025-11-03

历经半个多月的编写，终于也是把p3写完了，目前p3给我的感受比p2要好很多（因为p2的整个大文件的构建实在是非常折磨，对于我这种没有什么编码经验的c++新手来说非常困难，以至于出现了对于一个功能看半天的情况）。

下面是对于p3的一些自己踩坑的经历和实际体验：

p3主要是计划查询的构建，445的官方文档给出了明确的构建计划，在整个p3的计划中，我们只需要完成简单的读写查询就好了，主要分为：

- **Task #1: Access Method Executors**

- **Task #2: Aggregation and Join Executors**

- **Task #3: Sort + Limit Executors and Top-N Optimization**

这三个任务，由于这些文件的头文件都给出了已经实现好的辅助函数，实际上p3的任务量远远小于p2

t1：

seqscan、insert、delete、indexscan，非常好理解，遍历扫描，插入，删除，索引查找扫描，内部只要注意函数初始化的细节，还有一些指针的调用问题，辅助函数会帮我们解决问题。

t2：

Aggregation、NestedLoopJoin、NestedIndexJoin

分组，遍历连接，利用索引连接，其实在lecture中都讲过，其中遍历查找的效率较慢，需要先遍历完所有的左表和右表，寻找到合适的条件直接塞进去，而索引查找就可以利用索引查找右表的内容，这样就比loop快很多。

t3：
sort、limit、topn

sort：非常简单，自定义一个比较器，子查询器遍历next，另外开一个vector，然后sort一下就好（把比较器塞进去）

limit：获取需要限制输出的语句条数，next（），然后需要限制的语句num++就好了

topn：如果你力扣刷的多一点，简直是非常自然而然的就能想到，动态维护一个列表，按照自定义排序，完完全全就是一个MaxHeap/MinHeap的实际应用，直接自定义比较器，塞到实现好的heap里面，完成。

整个p3做下来的进度也比p2快了不少，等笔者把几个小问题修一下就通关，马不停蹄开下一个pj