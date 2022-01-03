---
title: "AcWing 4201. 01数"
date: 2021-12-25T21:22:38+08:00
draft: false
tags: ["acwing", "中等", "枚举", "深度优先搜索", "二进制"]
math: true
---

如果一个**正整数**，其各个数位上的数字均满足要么是 0，要么是 1，则称该数字为 01 数。

例如，1 和 10 都是 01 数。

给定一个整数 n。

请你计算，1∼n 中有多少个 01 数。

<!--more-->

#### 输入格式

一行，一个整数 n。

#### 输出格式

一个整数，表示 01 数的数量。

#### 数据范围

前六个测试点满足 1≤n≤100。

所有测试点满足 1≤n≤109。

#### 输入样例：

```
10
```

#### 输出样例：

```
2
```

```cpp
#include<iostream>
#include<cmath>
#include<vector>

using namespace std;

int main(){
    int number;
    cin>>number;
    int copy = number;
    int len = 0;
    while (copy != 0) {
        copy /= 10;
        len++;
    }
    vector<int> arr = vector<int>(len);
    copy = number;
    for (int i = len - 1; i >= 0; i--) {
        arr[i] = copy % 10;
        copy /= 10;
    }

    int sum = static_cast<int>(pow(2, len));
    for (int i = 0; i < len; i++) {
        if (arr[i] > 1) {
            break;
        } else if (arr[i] == 0) {
            sum -= static_cast<int>(pow(2, len - i - 1));
        }
    }
    sum--;
    cout<< sum;
    return 0;
}
```
