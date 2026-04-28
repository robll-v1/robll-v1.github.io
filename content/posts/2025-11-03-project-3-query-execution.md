---
title: "CMU 15-445 Project #3 — Query Execution"
date: 2025-11-03
draft: false
categories: ["数据库"]
tags: ["CMU 15-445", "Query Execution"]
---

Project #3 实现查询执行器，整体比 P2 轻松不少 — 头文件已经提供了大量辅助函数，实际编码量远小于 P2。

## Task #1 — Access Method Executors

四个基础执行器：

- **SeqScan**：顺序遍历表中所有 tuple
- **Insert**：插入 tuple 到表，同时更新索引
- **Delete**：标记删除 tuple，同时更新索引
- **IndexScan**：通过索引定位 tuple，避免全表扫描

实现上注意初始化细节和指针的正确调用，辅助函数会处理大部分底层逻辑。

## Task #2 — Aggregation and Join Executors

**Aggregation**

分组聚合，按 group-by 键分组后对每组执行聚合函数。

**NestedLoopJoin**

双层循环遍历左表和右表，找到满足条件的 tuple 对。效率较低，适合小表。

**NestedIndexJoin**

外层遍历左表，内层通过索引查找右表匹配行。比 NestedLoopJoin 快很多，前提是右表有索引。

这些在 Andy 的课上都讲过原理，实现时对照讲义即可。

## Task #3 — Sort + Limit + Top-N

**Sort**

自定义比较器，子查询器遍历 Next 把所有 tuple 收集到 vector，然后 `std::sort`。

**Limit**

获取限制条数，每次 Next 计数，达到上限后停止输出。

**Top-N**

动态维护一个大小为 N 的堆。自定义比较器，遍历所有 tuple 维护堆即可 — 本质就是 LeetCode 上经典的「第 K 大元素」问题的实际应用。

## 总结

P3 的设计很好，每个 Task 都有明确的目标，头文件的辅助函数大幅降低了实现难度。做完之后对查询执行的火山模型（Volcano Model）有了更直观的理解。
