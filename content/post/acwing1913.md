---
title: "AcWing 1913. 公平摄影"
date: 2022-01-20T12:57:54+08:00
draft: false
tags: ["acwing", "简单", "枚举", "前缀和", "哈希表"]
math: true
---

农夫约翰的 N 头奶牛站在一维长围栏的不同位置。

第 i 头牛位于位置 xi，其所属品种为 bi（根西岛牛或荷斯坦牛）。

所有奶牛的位置各不相同。

约翰想给一段连续区间内的奶牛拍摄一张照片，用来在乡村集市上展览。

但是我们希望他所有品种的奶牛都能在照片中得到公平的展示。

因此，他希望确保无论照片中出些哪些品种的奶牛，每种品种的奶牛在照片中的数量都必须相等。

例如，一张照片中只包含荷斯坦牛是可以的，包含荷斯坦牛和根西岛牛各 27 头也没问题，但是包含 10 头荷斯坦牛和 9 头根西岛牛则不可以。

请确定，约翰可以拍下的满足以上条件的照片的最大尺寸。

照片的尺寸是指照片中奶牛最大和最小位置之间的差。

约翰最终可能只拍下一头奶牛，这种情况下，照片尺寸为 0。

<!--more-->

#### 输入格式

第一行包含整数 N。

接下来 N 行，每行包含一个整数 xi 和一个字符 bi（H 表示荷斯坦牛，G 表示根西岛牛）。

#### 输出格式

输出照片的最大尺寸。

#### 数据范围

1≤N≤105,

0≤xi≤109

#### 输入样例：

```
6
4 G
10 H
7 G
16 G
1 G
3 H
```

#### 输出样例：

```
7
```

#### 样例解释

共 6 头牛，从左到右排列顺序为 G,H,G,G,H,G。

最佳摄影方案是拍中间四头奶牛，恰好荷斯坦牛和根西岛牛各两头。

```cpp
#include<iostream>
#include <vector>
#include <map>
#include <unordered_map>
#include<algorithm>
#include<set>

using namespace std;

int main() {
    unsigned int n;
    cin >> n;
    map<unsigned int, int> cows               = map<unsigned int, int>();
    vector<unsigned int> indexes              = vector<unsigned int>();
    unordered_map<int, set<unsigned int>> sum = unordered_map<int, set<unsigned int>>();
    for(unsigned int i = 0; i < n; i++) {
        unsigned int x;
        char b;
        cin >> x >> b;
        indexes.push_back(x);
        if(b == 'G') {
            cows.insert(pair(x, 1));
        } else {
            cows.insert(pair(x, -1));
        }
    }
    sort(indexes.begin(), indexes.end());

    unsigned int g_count = 0;
    unsigned int h_count = 0;
    unsigned int g_max   = 0;
    unsigned int h_max   = 0;
    int g_start          = -1;
    int h_start          = -1;
    int count            = 0;
    for(auto &cow: cows) {
        if(cow.second == 1) {//G
            h_count = 0;
            h_start = -1;
            if(g_start == -1) {
                g_start = cow.first;
            } else {
                g_count = cow.first - g_start;
            }
            g_max = max(g_max, g_count);
        } else {//H
            g_count = 0;
            g_start = -1;
            if(h_start == -1) {
                h_start = cow.first;
            } else {
                h_count = cow.first - h_start;
            }
            h_max = max(h_max, h_count);
        }
        count += cow.second;
        cow.second = count;
        if(sum.count(cow.second) == 0) {
            set<unsigned int> s = set<unsigned int>();
            s.insert(cow.first);
            sum.insert(pair(cow.second, s));
        } else {
            sum[cow.second].insert(cow.first);
        }
    }
    unsigned int maximum = 0;
    for(const auto &i: sum) {
        if(i.first == 0) {
            maximum = max(maximum, *i.second.rbegin() - indexes[0]);
        } else if(i.second.size() >= 2) {
            maximum = max(maximum, *i.second.rbegin() - *(upper_bound(indexes.begin(), indexes.end(), *i.second.begin())));
        }
    }
    cout << max(maximum, max(g_max, h_max));
    return 0;
}
```
