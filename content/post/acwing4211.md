---
title: "AcWing 4211. 序列重排"
date: 2022-01-15T20:42:10+08:00
draft: false
tags: ["acwing", "困难"]
math: true
---

给定一个长度为 n 的整数序列 \\(a_1,a_2,…,a_n\\)。

请你对序列进行重新排序（也可以保持原序列），要求新序列满足每个元素（第 1 个除外）都恰好是前一个元素的两倍或前一个元素的三分之一。

保证输入一定有解。

<!--more-->

#### 输入格式

第一行包含整数 n。

第二行包含 n 个整数 \\(a_1,a_2,…,a_n\\)。

#### 输出格式

一行 n 个整数，表示排序后的序列。

#### 数据范围

前三个测试点满足 2≤n≤10。  
所有测试点满足 \\(2\le n \le 100, 1 \le a_i \le 3 \times 10^{18}\\)。

#### 输入样例 1

```
6
4 8 6 3 12 9
```

#### 输出样例 1

```
9 3 6 12 4 8
```

#### 输入样例 2

```
4
42 28 84 126
```

#### 输出样例 2

```
126 42 84 28
```

#### 输入样例 3

```
2
1000000000000000000 3000000000000000000
```

#### 输出样例 3

```
3000000000000000000 1000000000000000000
```

```cpp
#include<iostream>
#include<vector>
#include<algorithm>

using namespace std;

unsigned int no2(unsigned long long a) {
    unsigned int count = 0;
    while(a % 2 == 0) {
        count++;
        a /= 2;
    }
    return count;
}

unsigned int no3(unsigned long long a) {
    unsigned int count = 0;
    while(a % 3 == 0) {
        count++;
        a /= 3;
    }
    return count;
}

bool cmp(unsigned long long a, unsigned long long b) {
    const unsigned a3 = no3(a);
    const unsigned b3 = no3(b);
    if(a3 == b3) {
        const unsigned a2 = no2(a);
        const unsigned b2 = no2(b);
        return a2 < b2;
    }
    return a3 > b3;
}

int main() {
    unsigned short n;
    cin >> n;
    auto vec = vector<unsigned long long>();
    for(unsigned short i = 0; i < n; i++) {
        unsigned long long a;
        cin >> a;
        vec.push_back(a);
    }
    sort(vec.begin(), vec.end(), cmp);
    for(const auto i: vec) {
        cout << i << " ";
    }
    return 0;
}
```
