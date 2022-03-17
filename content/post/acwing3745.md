---
title: "AcWing 3745. 牛的学术圈 I"
date: 2022-03-17T16:37:13+08:00
draft: false
tags: ["acwing", "简单", "枚举", "贪心", "双指针", "前缀和"]
math: true
---

由于对计算机科学的热爱，以及有朝一日成为 「Bessie 博士」的诱惑，奶牛 Bessie 开始攻读计算机科学博士学位。

经过一段时间的学术研究，她已经发表了 N 篇论文，并且她的第 i 篇论文得到了来自其他研究文献的 ci 次引用。

Bessie 听说学术成就可以用 h 指数来衡量。

h 指数等于使得研究员有至少 h 篇引用次数不少于 h 的论文的最大整数 h。

例如，如果一名研究员有 4 篇论文，引用次数分别为 (1,100,2,3)，则 h 指数为 2，然而若引用次数为 (1,100,3,3) 则 h 指数将会是 3。

为了提升她的 h 指数，Bessie 计划写一篇综述，并引用一些她曾经写过的论文。

由于页数限制，她至多可以在这篇综述中引用 L 篇论文，并且她只能引用每篇她的论文至多一次。

请帮助 Bessie 求出在写完这篇综述后她可以达到的最大 h 指数。

注意 Bessie 的导师可能会告知她纯粹为了提升 h 指数而写综述存在违反学术道德的嫌疑；我们不建议其他学者模仿 Bessie 的行为。

<!--more-->

#### 输入格式

输入的第一行包含 N 和 L。

第二行包含 N 个空格分隔的整数 \\(c_1,…,c_N\\)。

#### 输出格式

输出写完综述后 Bessie 可以达到的最大 h 指数。

#### 数据范围

\\(1≤N≤10^5\\),  
\\(0≤ci≤10^5\\),  
\\(0≤L≤10^5\\)

#### 输入样例1

```
4 0
1 100 2 3
```

#### 输出样例1

```
2
```

#### 样例1解释

Bessie 不能引用任何她曾经写过的论文。上文中提到，(1,100,2,3) 的 h 指数为 2。

#### 输入样例2

```
4 1
1 100 2 3
```

#### 输出样例2

```
3
```

如果 Bessie 引用她的第三篇论文，引用数会变为 (1,100,3,3)。上文中提到，这一引用数的 h 指数为 3。

## 题解

Let’s suppose that L=0; then we just want to compute the h−index of Bessie’s papers with no additional citations. To find this, we can sort the citation counts from largest to smallest, and find the largest h such that the first h papers all have at least h citations. This can be done in one pass through the sorted counts.

Now let’s consider how to use L extra citations to increase the h−index. Since the survey cannot cite any one paper multiple times, it’s not possible to increase the h−index by more than 1: the (h+1)−st most cited paper before the survey had at most h citations, so afterwards it cannot have more than h+1 citations.

But it’s not always possible to increase the h-index by one. When is it possible? Well, out of the top h papers before the survey, some k of them have exactly h citations (and the rest have more than h). We need to cite each of these k papers. Additionally, we need to cite the (h+1)-st most cited paper. So it’s necessary that L≥k+1. Finally, if the (h+1)-st most cited paper has less than h citations, then we cannot hope to increase the h-index. Conversely, if it has h citations (it cannot have more than h), and if L≥k+1, then the h-index can be increased to h+1.

```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
#include <algorithm>

using namespace std;

int main() {
    int n, l;
    cin >> n >> l;
    vector<int> c(n);
    unordered_map<int, int> count;
    for(int i = 0; i < n; i++) {
        cin >> c[i];
        count[c[i]]++;
    }
    sort(c.rbegin(), c.rend());
    if(*c.begin() == 0) {
        if(l != 0) {
            *c.begin() = 1;
            l--;
        } else {
            cout << 0;
            return 0;
        }
    }
    int h = 0;
    while(h < n && c[h] >= h + 1) {
        h++;
    }
    if(l == 0) {
        cout << h;
        return 0;
    }
    int k = 0;
    for(int i = h - 1; i >= 0 && c[i] == h; i--) {
        k++;
    }
    if(l >= k + 1 && c[h] >= h) {
        cout << h + 1;
    } else {
        cout << h;
    }
    return 0;
}
```
