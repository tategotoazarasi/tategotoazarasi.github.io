---
title: "AcWing 4200. 简单问题"
date: 2021-12-25T21:19:15+08:00
draft: false
tags: ["acwing", "简单"]
math: true
---

给定 6 个整数 p1,p2,p3,p4,a,b。

请问有多少个整数 x 满足以下全部条件：

1. a≤x≤b
2. x 同时小于 p1,p2,p3,p4

<!--more-->

#### 输入格式

一行 6 个整数 p1,p2,p3,p4,a,b。

#### 输出格式

一个整数，表示满足条件的整数 x 的数量。

#### 数据范围

所有测试点满足 1≤p1,p2,p3,p4≤10000≤a≤b≤31415。保证 p1,p2,p3,p4 两两不同。

#### 输入样例 1：

```
2 7 1 8 2 8
```

#### 输出样例 1：

```
0
```

#### 输入样例 2：

```
20 30 40 50 0 100
```

#### 输出样例 2：

```
20
```

#### 输入样例 3：

```
31 41 59 26 17 43
```

#### 输出样例 3：

```
9
```

```cpp
#include<iostream>

using namespace std;

int main(){
    int p1, p2, p3, p4, a, b;
    cin >> p1 >> p2 >> p3 >> p4 >> a >> b;
    int min = p1;
    min = p2 < min ? p2 : min;
    min = p3 < min ? p3 : min;
    min = p4 < min ? p4 : min;
    int ans = (min - a) < (b - a + 1) ? min - a : b - a + 1;
    cout << (ans < 0 ? 0 : ans);
    return 0;
}
```
