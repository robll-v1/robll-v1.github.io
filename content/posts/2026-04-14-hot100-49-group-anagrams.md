---
title: "Hot100 49. 字母异位词分组"
date: 2026-04-14
draft: false
categories: ["算法"]
tags: ["LeetCode", "Hot100", "哈希表", "字符串", "排序"]
---

日期：2026-04-14

题目链接：<https://leetcode.cn/problems/group-anagrams/?envType=study-plan-v2&envId=top-100-liked>

## 核心思路

这题的关键不是直接比较两个字符串，而是先想清楚：

同一组字母异位词，虽然原字符串顺序不同，但排序之后一定完全一样。

所以可以把“排序后的字符串”当成分组标识。

## 为什么这样做

比如：

- `eat` 排序后是 `aet`
- `tea` 排序后是 `aet`
- `ate` 排序后是 `aet`

它们排序后都一样，所以应该放进同一组。

于是可以用哈希表做分组：

- `key`：排序后的字符串
- `value`：这一组对应的原字符串数组

## 解题步骤

1. 定义一个哈希表，键是排序后的字符串，值是该组的原字符串列表。
2. 遍历 `strs` 中的每个字符串 `s`。
3. 复制一份得到 `t`，对 `t` 排序。
4. 把原字符串 `s` 放进 `mp[t]` 对应的数组中。
5. 遍历哈希表，把所有 `value` 放进结果数组中返回。

## 为什么放的是 `s` 不是 `t`

`t` 只是用来分组的，不是最终答案。

题目要求返回的是原始字符串的分组结果，而不是排序后的字符串。

如果把 `t` 放进去，像 `eat`、`tea`、`ate` 最后就都会变成 `aet`，不符合题意。

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

- 设字符串个数为 `n`
- 每个字符串平均长度为 `k`

则：

- 时间复杂度：`O(n * k log k)`
- 空间复杂度：`O(n * k)`

时间主要花在每个字符串的排序上。

## 容易错的点

1. 哈希表的 `key` 不是原字符串，而是排序后的字符串。
2. 放进分组里的必须是原字符串 `s`，不是排序后的 `t`。
3. 遍历 `unordered_map` 时，范围 `for` 取到的是对象，用 `.` 访问成员，不是 `->`。
4. 只读遍历时，优先用 `const auto&`，可以避免不必要的拷贝。

## 一句话记忆

异位词排序后一定相同，所以用排序后的字符串当哈希表 key 来分组。
