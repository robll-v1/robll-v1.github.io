---
title: "Hot100 49. 字母异位词分组"
date: 2026-04-14
draft: false
categories: ["算法"]
tags: ["LeetCode", "Hot100", "哈希表", "字符串", "排序"]
---

题目链接：<https://leetcode.cn/problems/group-anagrams/>

## 核心思路

同一组异位词排序后一定完全相同。用排序后的字符串作为哈希表的 key 来分组。

例如 `eat`、`tea`、`ate` 排序后都是 `aet`，归入同一组。

## 代码

```cpp
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> mp;

        for (const auto& s : strs) {
            string t = s;
            sort(t.begin(), t.end());
            mp[t].push_back(s);
        }

        vector<vector<string>> ans;
        for (const auto& item : mp) {
            ans.push_back(item.second);
        }

        return ans;
    }
};
```

## 复杂度

设字符串个数为 n，平均长度为 k：

- 时间：O(n * k log k)，主要花在排序
- 空间：O(n * k)

## 易错点

1. 哈希表 key 是排序后的字符串，不是原字符串
2. 放入分组的必须是原字符串 `s`，不是排序后的 `t`
3. 只读遍历用 `const auto&` 避免不必要的拷贝
