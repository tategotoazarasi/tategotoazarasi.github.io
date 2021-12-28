---
title: "AcWing 608. 差"
date: 2021-12-28T20:24:50+08:00
draft: false
tags: ["acwing", "简单", "顺序结构"]
math: true
---

读取四个整数 A,B,C,D，并计算 (A×B−C×D) 的值。

<!--more-->

#### 输入格式

输入共四行，第一行包含整数 A，第二行包含整数 B，第三行包含整数 C，第四行包含整数 D。

#### 输出格式

输出格式为 `DIFERENCA = X`，其中 X 为 (A×B−C×D)的结果。

#### 数据范围

−10000≤A,B,C,D≤10000

#### 输入样例：

```
5
6
7
8
```

#### 输出样例：

```
DIFERENCA = -26
```

```cpp
#include<iostream>

using namespace std;

int main() {
    int a, b, c, d;
    cin >> a >> b >> c >> d;
    cout << "DIFERENCA = " << a * b - c * d;
}
```
