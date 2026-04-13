---
title: "#p2 解释"
date: 2025-09-28
draft: false
categories: ["数据库"]
tags: ["CMU 15-445", "学习笔记"]
---

日期：2025-09-28

## 总览

你需要在 Bustub 数据库系统中实现 B+Tree 索引结构。B+Tree 是一种平衡树，支持高效的有序查找和随机访问。你要实现其插入、删除、查找、分裂和合并等核心逻辑。项目分为两个阶段（Checkpoint）。

## Checkpoint #1（第一阶段）

### Task 1 – B+Tree Pages（B+Tree 页面类实现）

你需要实现三个页面类，分别用于存储 B+Tree 的数据：

- **B+Tree Parent Page（父页面类）**

- 这是 Internal Page（内部页）和 Leaf Page（叶子页）的父类，包含两者共有的信息。

- 字段包括：页面类型、日志序列号、键值对数量、最大键值对数量、父页面ID、当前页面ID。

- 只允许修改 b_plus_tree_page.h 和 b_plus_tree_page.cpp。

- **B+Tree Internal Page（内部页面类）**

- 只存储有序的 m 个键和 m+1 个子指针（页面ID），不存储实际数据。

- 第一个键无效，查找时从第二个键开始。

- 插入和删除时需处理分裂、合并和重分布。

- 只允许修改 b_plus_tree_internal_page.h 和 b_plus_tree_internal_page.cpp。

- **B+Tree Leaf Page（叶子页面类）**

- 存储有序的 m 个键和 m 个值（值为 64 位 RID，指向实际数据）。

- 插入和删除时需处理分裂、合并和重分布。

- 只允许修改 b_plus_tree_leaf_page.h 和 b_plus_tree_leaf_page.cpp。

**注意：**每个页面都对应 buffer pool 中的一页，读写时需先通过页面ID从 buffer pool 获取，再进行类型转换和操作。

### Task 2 – B+Tree Data Structure（B+Tree 数据结构实现）

- 只支持唯一键（插入重复键时返回 false，不插入）。

- 实现插入（`Insert()`）、点查找（`GetValue()`）、删除（`Delete()`）。

- 插入时如果页面满了要分裂，删除后页面低于阈值要合并或重分布。

- 插入/删除可能导致根页面ID变化，需调用 `UpdateRootPageId` 更新。

- B+Tree 类需使用模板隐藏键值类型和比较器：

```
  template 
  class BPlusTree { ... }
```

- 键类型为 `GenericKey`，值类型为 64 位 RID，比较器为 `KeyComparator`。

## Checkpoint #2（第二阶段）

### Task 3 – Index Iterator（索引迭代器实现）

- 实现一个迭代器，能高效遍历所有叶子页面上的键值对。

- 叶子页面需组织成单链表，迭代器通过链表遍历。

- 需实现 C++17 标准的迭代器接口，包括：

- `isEnd()`：判断是否到达末尾

- `operator++()`：移动到下一个键值对

- `operator*()`：返回当前键值对

- `operator==()` 和 `operator!=()`：判断迭代器是否相等

- 只允许修改 index_iterator.h 和 index_iterator.cpp。

### Task 4 – Concurrent Index（并发索引实现）

- 让 B+Tree 支持多线程并发操作，采用 latch crabbing 技术（即在遍历树时逐步加锁/解锁页面）。

- 不允许用全局锁保护整个数据结构，必须细粒度加锁。

- 需正确处理插入、查找、删除时的加锁/解锁顺序，避免死锁和性能瓶颈。

- 相关辅助类和方法已提供，如 `rwlatch.h`（读写锁）、`page.h`（页面加锁/解锁）。

- 需用 `transaction` 参数记录加锁页面和删除页面，建议在 `FindLeafPage` 方法中实现加锁逻辑。

- 需用 `std::mutex` 保护 `root_page_id` 的并发更新。

## 测试与提交

- 提供了测试工具和可视化工具（如 `b_plus_tree_printer`），可帮助你调试和验证实现。

- Checkpoint #1 只需通过插入、查找、删除相关测试，不要求并发和迭代器功能。

- Checkpoint #2 需通过所有相关测试，包括并发和迭代器。

- 按要求打包并提交指定文件，支持多次提交和即时反馈。