---
title: "AcWing 4214. 三元组"
date: 2022-01-22T21:32:54+08:00
draft: false
tags: ["acwing", "困难"]
math: true
---

给定两个长度为 n 的整数序列 \\(s_1,s_2,…,s_n\\) 和 \\(c_1,c_2,…,c_n\\)。

请你找到一个三元组 (i,j,k)，满足以下所有条件：

- i<j<k

- \\(s_i<s_j<s_k\\)

- \\(c_i+c_j+c_k\\)尽可能小

输出 \\(c_i+c_j+c_k\\) 的最小可能值。

<!--more-->

#### 输入格式

第一行包含整数 n。

第二行包含 n 个整数 \\(s_1,s_2,…,s_n\\)。

第三行包含 n 个整数 \\(c_1,c_2,…,c_n\\)。

#### 输出格式

如果满足条件的三元组不存在，则输出 −1。

否则，输出 \\(c_i+c_j+c_k\\) 的最小可能值。

#### 数据范围

前 5 个测试点满足 3≤n≤10。

所有测试点满足 \\(3≤n≤3000，1≤si≤10^9，1≤ci≤10^8\\)。

#### 输入样例 1：

```
5
2 4 5 4 10
40 30 20 10 40
```

#### 输出样例 1：

```
90
```

#### 输入样例 2：

```
3
100 101 100
2 4 5
```

#### 输出样例 2：

```
-1
```

#### 输入样例 3：

```
10
1 2 3 4 5 6 7 8 9 10
10 13 11 14 15 12 13 13 18 13
```

#### 输出样例 3：

```
33
```

```cpp
#include<iostream>

using namespace std;

int main() {
    unsigned int max = 300000001;
    unsigned short n;
    cin >> n;
    auto *s = new unsigned int[n];
    auto *c = new unsigned int[n];
    for(unsigned short i = 0; i < n; i++) {
        cin >> s[i];
    }
    for(unsigned short i = 0; i < n; i++) {
        cin >> c[i];
    }
    unsigned int minimum = max;
    for(int j=1; j<n-1; j++) {
        unsigned int min_left = max;
        unsigned int min_right = max;
        for(int i=0; i<j; i++) {
            if(s[i] < s[j]) {
                if(c[i]<min_left) {
                    min_left = c[i];
                }
            }
        }
        for(int k=j+1; k<n; k++) {
            if(s[j] < s[k]) {
                if(c[k] < min_right) {
                    min_right = c[k];
                }
            }
        }
        if(min_left != max && min_right != max && minimum > min_left + c[j] + min_right) {
            minimum = min_left + c[j] + min_right;
        }
    }
    if(minimum != max) {
        cout << minimum;
    } else {
        cout << -1;
    }
    return 0;
}
```
