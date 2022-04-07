---
title: "AcWing 785. 快速排序"
date: 2022-04-07T17:04:23+08:00
draft: false
tags: ["acwing", "简单", "快速排序"]
math: true
---

给定你一个长度为 n 的整数数列。

请你使用快速排序对这个数列按照从小到大进行排序。

并将排好序的数列按顺序输出。

<!--more-->

#### 输入格式

输入共两行，第一行包含整数 n。

第二行包含 n 个整数（所有整数均在 1∼109 范围内），表示整个数列。

#### 输出格式

输出共一行，包含 n 个整数，表示排好序的数列。

#### 数据范围

1≤n≤100000

#### 输入样例

```
5
3 1 2 4 5
```

#### 输出样例

```
1 2 3 4 5
```

```cpp
#include <iostream>
#include <vector>

using namespace std;

const int N = 1e6+10;
int vec[N];

void qs(int vec[],int l, int r) {
    if(l >= r) {
        return;
    }
    int x  = vec[l+r>>1];
    int lp = l-1;
    int rp = r+1;
    while(lp < rp) {
        while(vec[++lp] < x);
        while(vec[--rp] > x);
        if(lp < rp) {
            swap(vec[lp], vec[rp]);
        }
    }
    qs(vec, l, rp);
    qs(vec, rp + 1, r);
}

int main() {
    int n;
    scanf("%d", &n);
    for(int i=0; i<n; i++) {
        scanf("%d",&vec[i]);
    }
    qs(vec,0,n-1);
    for(int i=0; i<n; i++) {
        printf("%d ",vec[i]);
    }
    return 0;
}
```
