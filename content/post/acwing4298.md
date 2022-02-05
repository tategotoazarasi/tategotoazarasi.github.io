---
title: "AcWing 4298. 搭档"
date: 2022-02-05T21:34:54+08:00
draft: false
tags: ["acwing", "困难", "匈牙利算法"]
math: true
---

有 n 个男孩，每个人的魅力值依次为 a1,a2,…,an.

有 m 个女孩，每个人的魅力值依次为 b1,b2,…,bm。

学校举办舞会，希望选出尽可能多的男女搭档参加。

已知，当一对男女的魅力值相差不超过 1 时，他们才会愿意组成搭档。

请问，最多可以**同时**凑出多少对搭档参加舞会。

注意：

- 不能男男配对或女女配对。
- 每个男生最多只能和一个女生配对，反之亦然。

<!--more-->

#### 输入格式

第一行包含整数 n。

第二行包含 n 个整数 a1,a2,…,an。

第三行包含整数 m。

第四行包含 m 个整数 b1,b2,…,bm。

#### 输出格式

一个整数，表示最大可能对数。

#### 数据范围

前 6 个测试点满足 1≤n,m≤5,

所有测试点满足 1≤n,m≤100，1≤ai,bi≤100。

#### 输入样例 1：

```
4
1 4 6 2
5
5 1 5 7 9
```

#### 输出样例 1：

```
3
```

#### 输入样例 2：

```
4
1 2 3 4
4
10 11 12 13
```

#### 输出样例 2：

```
0
```

#### 输入样例 3：

```
5
1 1 1 1 1
3
1 2 3
```

#### 输出样例 3：

```
2
```

```cpp
#include<iostream>
#include<cstring>

using namespace std;

int n            = 0;      ///< 男孩数量
int m            = 0;      ///< 女孩数量
int *a           = nullptr;///< 男孩魅力值
int *b           = nullptr;///< 女孩魅力值
bool *found      = nullptr;///< 记录女孩是否已经被找到
bool **connected = nullptr;///< 邻接矩阵存图
int *match       = nullptr;///< 记录当前女孩所对应的男孩

bool find(int i) {
    for(int j = 0; j < m; j++) {
        if(connected[i][j] && !found[j]) {
            //可以配对且女孩未被找到
            found[j] = true;
            if(match[j] == -1 || find(match[j])) {
                //女孩没有配对的对象或女孩配对的对象可以找到其他对象去配对
                match[j] = i;//设置女孩j和男孩i配对
                return true;
            }
        }
    }
    return false;
}

int main() {
    cin >> n;
    a = new int[n];
    memset(a, 0, n * sizeof(int));
    for(int i = 0; i < n; i++) {
        cin >> a[i];
    }
    cin >> m;
    b     = new int[m];
    found = new bool[m];
    match = new int[m];
    memset(b, 0, m * sizeof(int));
    memset(found, 0, m * sizeof(bool));
    memset(match, -1, m * sizeof(int));
    for(int i = 0; i < m; i++) {
        cin >> b[i];
    }
    connected = new bool *[n];
    for(int i = 0; i < n; i++) {
        connected[i] = new bool[m];
        for(int j = 0; j < m; j++) {
            if(abs(a[i] - b[j]) <= 1) {
                connected[i][j] = true;
            } else {
                connected[i][j] = false;
            }
        }
    }

    int cnt = 0;
    for(int i = 0; i < n; i++) {
        memset(found, 0, m * sizeof(bool));
        if(find(i)) {
            cnt++;
        }
    }
    cout << cnt;
    for(int i = 0; i < n; i++) {
        delete[] connected[i];
    }
    delete[] connected;
    delete[] a;
    delete[] b;
    delete[] found;
    delete[] match;
    return 0;
}
```
