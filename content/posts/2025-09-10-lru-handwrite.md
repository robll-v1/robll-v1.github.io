---
title: "手撕 LRU 缓存"
date: 2025-09-10
draft: false
categories: ["算法"]
tags: ["面试", "LRU", "手撕"]
---

LRU 缓存是面试高频题，核心是哈希表 + 双向链表，实现 O(1) 的 get 和 put。

## 设计思路

- **双向链表**：按使用顺序存储键值对，头部是最近使用的，尾部是最久未使用的
- **哈希表**：通过 key 直接定位到链表节点，O(1) 查找

### get 操作

1. key 不存在：返回 -1
2. key 存在：将节点移到链表头部，返回 value

### put 操作

1. key 不存在：创建新节点插入头部，超出容量则删除尾部节点
2. key 存在：更新 value，将节点移到头部

所有操作都是 O(1)：哈希表查找 O(1)，链表头部插入和尾部删除 O(1)，移动节点 = 删除 + 头部插入。

## 实现

```cpp
struct DLinkedNode {
    int key, value;
    DLinkedNode* prev;
    DLinkedNode* next;
    DLinkedNode(): key(0), value(0), prev(nullptr), next(nullptr) {}
    DLinkedNode(int _key, int _value): key(_key), value(_value), prev(nullptr), next(nullptr) {}
};

class LRUCache {
private:
    unordered_map<int, DLinkedNode*> cache;
    DLinkedNode* head;
    DLinkedNode* tail;
    int size;
    int capacity;

public:
    LRUCache(int _capacity): capacity(_capacity), size(0) {
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head->next = tail;
        tail->prev = head;
    }

    int get(int key) {
        if (!cache.count(key)) {
            return -1;
        }
        DLinkedNode* node = cache[key];
        moveToHead(node);
        return node->value;
    }

    void put(int key, int value) {
        if (!cache.count(key)) {
            DLinkedNode* node = new DLinkedNode(key, value);
            cache[key] = node;
            addToHead(node);
            size++;
            if (size > capacity) {
                DLinkedNode* removed = removeTail();
                cache.erase(removed->key);
                delete removed;
                size--;
            }
        } else {
            DLinkedNode* node = cache[key];
            node->value = value;
            moveToHead(node);
        }
    }

    void addToHead(DLinkedNode* node) {
        node->prev = head;
        node->next = head->next;
        head->next->prev = node;
        head->next = node;
    }

    void removeNode(DLinkedNode* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    void moveToHead(DLinkedNode* node) {
        removeNode(node);
        addToHead(node);
    }

    DLinkedNode* removeTail() {
        DLinkedNode* node = tail->prev;
        removeNode(node);
        return node;
    }
};
```

## 复杂度

- 时间：get O(1)，put O(1)
- 空间：O(capacity)
