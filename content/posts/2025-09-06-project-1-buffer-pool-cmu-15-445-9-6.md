---
title: "CMU 15-445 Project #1 — Buffer Pool"
date: 2025-09-06
draft: false
categories: ["数据库"]
tags: ["CMU 15-445", "Buffer Pool"]
---

Project #1 的目标是实现一个可供测试的缓冲池（Buffer Pool），分为三个 Task。

## Task #1 — Extendible Hash Table

可扩展哈希表是一种防止哈希冲突的方案，核心是目录-桶（Directory-Bucket）机制。

**关键概念：**

- **全局深度（Global Depth）**：决定目录的大小（`2^depth` 个槽位）
- **局部深度（Local Depth）**：决定一个桶被多少个目录项指向
- **分裂机制**：桶满时局部深度 +1，分裂为两个桶；当局部深度 == 全局深度时，触发目录扩展

需要实现的接口：

```cpp
// Hash Table
auto Find(const K &key, V &value) -> bool override;
void Insert(const K &key, const V &value) override;
auto Remove(const K &key) -> bool override;

// Bucket
auto Bucket::Find(const K &key, V &value) -> bool;
auto Bucket::Remove(const K &key) -> bool;
auto Bucket::Insert(const K &key, const V &value) -> bool;
```

Insert 是最复杂的操作，流程如下：

1. 写锁保护目录
2. 计算 key 的目录索引，获取对应桶
3. 桶未满则直接插入返回
4. 桶满且局部深度 == 全局深度时，目录翻倍扩展
5. 分裂桶，重新分配键值对到新旧桶

## Task #2 — LRU-K Replacement Policy

LRU-K 是对经典 LRU 的扩展，记录每个页面最近的 K 次访问时间来决定淘汰顺序。

### 核心思想

- **经典 LRU**：只看最近一次访问，容易被偶然访问污染
- **LRU-K**：看第 K 次最近访问，能区分偶然访问和频繁访问

### 淘汰规则

1. 只有标记为「可淘汰」的页面才参与
2. 计算回溯 K 距离 = 当前时间 - 第 K 次最近访问时间
3. 淘汰回溯距离最大的页面
4. 访问次数不足 K 次的页面，回溯距离视为无穷大（优先淘汰）
5. 多个无穷大时，按经典 LRU 规则淘汰

### 举例（K=2）

| 页面 | 访问历史 | 回溯 2 距离（当前时间 30） |
|------|----------|--------------------------|
| A | 10, 20 | 30 - 10 = 20 |
| B | 15 | 无穷大（不足 2 次） |
| C | 5, 25 | 30 - 5 = 25 |

B 优先被淘汰。

### 实现结构

我在头文件中额外定义了三个工具类：

- **Entry**：记录每个帧的访问历史、可淘汰状态、时间戳等
- **TempPool**：管理访问次数不足 K 次的 Entry，链表 + 哈希表
- **CachePool**：管理已达到 K 次访问的 Entry，按时间排序的 set + 哈希表

简单函数加了 `inline` 提升效率。整个 BusTub 系统禁止拷贝和移动。

## Task #3 — Buffer Pool Manager

缓冲池管理器的工作原理：写入时优先使用空闲列表（free list）中的纯净页；空闲列表为空时启动 LRU-K 淘汰机制；淘汰脏页时必须先写回磁盘。

这个 Task 细节比前两个多，但思路相对直接。重点说一下 `FetchPgImp`：

```cpp
auto BufferPoolManager::FetchPgImp(page_id_t page_id) -> Page* {
    // 1. 通过 hash 表查找，如果页面已在内存中直接返回
    //    - LRU-K 记录访问
    //    - SetEvictable(false)
    //    - pin_count++

    // 2. 如果不在内存中，从空闲列表或淘汰中获取 frame
    //    - 淘汰脏页时先写回磁盘
    //    - 从磁盘读入目标页
    //    - pin_count = 1
    //    - LRU-K 记录访问
    //    - SetEvictable(false)
}
```

注意：无论页面是否已在 pool 中，都需要 record access、设置不可淘汰、更新 pin count。这个在文档里没有明确写，容易踩坑。

参考了 [AntiO2 的博客](https://blog.csdn.net/AntiO2/article/details/128554356) 中的部分思路。
