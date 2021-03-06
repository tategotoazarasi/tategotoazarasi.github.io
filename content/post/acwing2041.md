---
title: "AcWing 2041. 干草堆"
date: 2022-01-03T15:24:38+08:00
draft: false
tags: ["acwing", "简单", "差分"]
math: true
---

贝茜对她最近在农场周围造成的一切恶作剧感到抱歉，她同意帮助农夫约翰把一批新到的干草捆堆起来。

开始时，共有 N 个空干草堆，编号 1∼N。

约翰给贝茜下达了 K 个指令，每条指令的格式为 `A B`，这意味着贝茜要在 A..B 范围内的每个干草堆的顶部添加一个新的干草捆。

例如，如果贝茜收到指令 `10 13`，则她应在干草堆 10,11,12,13 中各添加一个干草捆。

在贝茜完成了所有指令后，约翰想知道 N 个干草堆的中值高度——也就是说，如果干草堆按照高度从小到大排列，位于中间的干草堆的高度。

方便起见，N 一定是奇数，所以中间堆是唯一的。

请帮助贝茜确定约翰问题的答案。

<!--more-->

#### 输入格式

第一行包含 N 和 K。

接下来 K 行，每行包含两个整数 A,B，用来描述一个指令。

#### 输出格式

输出完成所有指令后，N 个干草堆的中值高度。

#### 数据范围

1≤N≤106,  
1≤K≤25000,  
1≤A≤B≤N

#### 输入样例：

```
7 4
5 5
2 4
4 6
3 5
```

#### 输出样例：

```
1
```

#### 样例解释

贝茜完成所有指令后，各堆高度为 0,1,2,3,3,1,0。

将各高度从小到大排序后，得到 0,0,1,1,2,3,3，位于中间的是 1。

```cpp
#include<iostream>
#include<algorithm>
#include<cstring>

using namespace std;

int main() {
    int * haystack = new int[1000010];
    memset(haystack, 0, 1000010 * sizeof * haystack);
    int n, k;
    cin >> n >> k;
    for (int i = 0; i < k; i++) {
        int a, b;
        cin >> a >> b;
        haystack[a]++;
        haystack[b + 1]--;
    }
    for (int i = 1; i <= n; i++) {
        haystack[i] += haystack[i - 1];
    }
    sort(haystack + 1, haystack + n + 1);
    cout << haystack[(n + 1) / 2];
    return 0;
}
```
