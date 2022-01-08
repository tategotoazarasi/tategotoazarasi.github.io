---
title: "LintCode 295. 交集"
date: 2022-01-08T16:57:28+08:00
draft: false
tags: ["lintcode", "简单", "双指针", "同向双指针"]
math: true
---

## 描述

给定两个排序后的区间序列，序列内每个区间两两互不相交，返回两个序列相交的区间的下标

- 1<=len(a),len(b)<=1e5
- abs(max(a))<=1e9

## 样例

```
输入: a = [[0,3], [7,10]] , b = [[-1,1],[2,8]]
输出: ans = [[0,0],[0,1],[1,1]]
```

```cpp
bool is_intersected(const vector<int> &l, const vector<int> &r) {
    if(l[0] == r[0])
        return true;
    else if(l[0] < r[0])
        return r[0] <= l[1];
    else
        return l[0] <= r[1];
}

vector<vector<int>> Intersection(vector<vector<int>> &a, vector<vector<int>> &b) {
    vector<vector<int>> res;
    if(a.empty() || b.empty())
        return res;
    for(int i = 0, j = 0; i < a.size() && j < b.size();) {
        if(is_intersected(a[i], b[j]))
            res.emplace_back(vector<int>({ i, j }));
        if(a[i][1] == b[j][1]) {
            ++i;
            ++j;
        } else if(a[i][1] > b[j][1])
            ++j;
        else
            ++i;
    }
    return res;
}
```
