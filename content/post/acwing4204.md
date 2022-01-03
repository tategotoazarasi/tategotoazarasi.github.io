---
title: "AcWing 4204. 构造矩阵"
date: 2022-01-01T20:50:10+08:00
draft: false
tags: ["acwing", "中等"]
math: true
---

请你构造一个 n×n 的整数矩阵。要求，矩阵满足下列所有条件：

1. 矩阵中的所有元素的取值范围为 [0,n−1]。
2. 矩阵主对角线上的所有元素都为 0。主对角线是指从左上角到右下角这一斜线方向的对角线。
3. 该矩阵是对称矩阵。对称矩阵是指以主对角线为对称轴，各元素对应相等的矩阵。
4. 同一行上的所有元素两两不同。
5. 同一列上的所有元素两两不同。

<!--more-->

#### 输入格式

一个整数 n。

#### 输出格式

共 n 行，每行 n 个空格隔开的整数，表示整个矩阵。如果方案不唯一，输出任意合理方案均可。

#### 数据范围

前三个测试点满足 2≤n≤10 。
所有测试点满足 2≤n≤1000，n 保证是偶数。

#### 输入样例 1：

```
2
```

#### 输出样例 1：

```
0 1
1 0
```

#### 输入样例 2：

```
4
```

#### 输出样例 2：

```
0 1 3 2
1 0 2 3
3 2 0 1
2 3 1 0
```

```cpp
#include <iostream>

#include<set>

using namespace std;

int main() {
    int n;
    const int N = 1010;
    int g[N][N];
    cin >> n;
    for (int i = 1; i <= n; i++) {
        g[1][i] = i - 1;
        g[i][1] = i - 1;
    }
    for (int i = 2; i < n; i++) {
        for (int j = 2; j < n; j++) {
            if (i == j) {
                g[i][j] = 0; //对角线为0
            } else {
                if (g[i][j - 1] == 0) { //在0后面一格
                    g[i][j] = g[i - 1][j] + 1; //上面一格的值+1
                } else { //否则
                    g[i][j] = g[i][j - 1] + 1; //前面一格的值+1
                }
                if (g[i][j] == n) { //达到上限
                    g[i][j] = 1; //回归到1
                }
            }
        }
    }
    for (int i = 1; i < n; i++) {
        set < int > s;
        for (int j = 1; j < n; j++) {
            s.insert(g[i][j]);
        }
        for (int j = 0; j < n; j++) {
            if (!s.count(j)) {
                g[i][n] = g[n][i] = j; //补最后一列和最后一行的值
            }
        }
    }

    //输出
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            cout << g[i][j];
            if (j != n) {
                cout << " ";
            }
        }
        if (i != n) {
            cout << endl;
        }
    }
    return 0;
}
```