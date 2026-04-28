---
title: "CMU 15-445 Project #2 — B+Tree Index"
date: 2025-09-28
draft: false
categories: ["数据库"]
tags: ["CMU 15-445", "学习笔记"]
---

Project #2 要求在 BusTub 中实现完整的 B+Tree 索引，包括插入、删除、迭代器和并发控制。分两个 Checkpoint。

## Checkpoint #1

### Task 1 — B+Tree Pages

三个页面类：

**Parent Page（基类）**

Internal Page 和 Leaf Page 的公共父类，包含页面类型、LSN、键值对数量、最大容量、父页面 ID 等字段。

**Internal Page（内部页）**

存储有序的 m 个 key 和 m+1 个子指针（page_id）。第一个 key 无效，查找从第二个 key 开始。插入删除时需处理分裂、合并和重分布。

**Leaf Page（叶子页）**

存储有序的 m 个 key 和 m 个 value（64 位 RID，指向实际数据）。同样需要处理分裂、合并和重分布。

每个页面对应 buffer pool 中的一页，操作时需先通过 page_id 从 buffer pool 获取，再做类型转换。

### Task 2 — B+Tree 数据结构

只支持唯一键，重复插入返回 false。需要实现：

- `Insert()` — 插入键值对，页面满时触发分裂
- `GetValue()` — 点查找
- `Delete()` — 删除键值对，页面低于阈值时触发合并或重分布

B+Tree 类使用模板隐藏类型：

```cpp
template <typename KeyType, typename ValueType, typename KeyComparator>
class BPlusTree { ... }
```

键类型为 `GenericKey`，值类型为 64 位 RID，比较器为 `KeyComparator`。

插入和删除都可能导致根页面 ID 变化，需调用 `UpdateRootPageId` 更新。

## Checkpoint #2

### Task 3 — Index Iterator

实现迭代器遍历所有叶子页面的键值对。叶子页通过链表连接，迭代器沿链表顺序遍历。

需实现 C++17 迭代器接口：

- `isEnd()` — 是否到达末尾
- `operator++()` — 移到下一个键值对
- `operator*()` — 返回当前键值对
- `operator==()`、`operator!=()` — 迭代器比较

### Task 4 — Concurrent Index

使用 latch crabbing 技术实现并发安全，禁止全局锁。

核心思路：遍历树时逐层加锁解锁。在 `FindLeafPage` 中实现加锁逻辑，用 `transaction` 参数记录已加锁的页面。`root_page_id` 的并发更新用 `std::mutex` 保护。

## 测试

Checkpoint #1 只需通过插入、查找、删除测试。Checkpoint #2 需要通过所有测试，包括并发和迭代器。项目提供了 `b_plus_tree_printer` 可视化工具辅助调试。
