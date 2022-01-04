---
title: "AcWing 2060. 奶牛选美"
date: 2022-01-03T15:24:38+08:00
draft: false
tags: ["acwing", "泛洪", "广度优先搜索", "深度优先搜索"]
math: true
---

听说最近两斑点的奶牛最受欢迎，约翰立即购进了一批两斑点牛。

不幸的是，时尚潮流往往变化很快，当前最受欢迎的牛变成了一斑点牛。

约翰希望通过给每头奶牛涂色，使得它们身上的两个斑点能够合为一个斑点，让它们能够更加时尚。

<!--more-->

牛皮可用一个 N×M 的字符矩阵来表示，如下所示：

```
................
..XXXX....XXX...
...XXXX....XX...
.XXXX......XXX..
........XXXXX...
.........XXX....
```

其中，X 表示斑点部分。如果两个 X 在垂直或水平方向上相邻（对角相邻不算在内），则它们属于同一个斑点，由此看出上图中恰好有两个斑点。

约翰牛群里**所有的牛都有两个斑点**。

约翰希望通过使用油漆给奶牛尽可能少的区域内涂色，将两个斑点合为一个。

在上面的例子中，他只需要给三个 . 区域内涂色即可（新涂色区域用 ∗ 表示）：

```
................
..XXXX....XXX...
...XXXX*...XX...
.XXXX..**..XXX..
........XXXXX...
.........XXX....
```

请帮助约翰确定，为了使两个斑点合为一个，他需要涂色区域的最少数量。

#### 输入格式

第一行包含两个整数 N 和 M。

接下来 N 行，每行包含一个长度为 M 的由 X 和 . 构成的字符串，用来表示描述牛皮图案的字符矩阵。

#### 输出格式

输出需要涂色区域的最少数量。

#### 数据范围

1≤N,M≤50

#### 输入样例：

```
6 16
................
..XXXX....XXX...
...XXXX....XX...
.XXXX......XXX..
........XXXXX...
.........XXX....
```

#### 输出样例：

```
3
```

```cpp
#include<iostream>

#include<unordered_set>

#include<queue>

using namespace std;

struct point {
    int x;
    int y;
    point() =
        default;
    point(int x, int y): x(x), y(y) {};
};

struct pointhash {
    size_t operator()(const point & p) const {
        return p.x * 50 + p.y;
    }
};

struct pointequal {
    bool operator()(const point & p1,
        const point & p2) const {
        return p1.x == p2.x && p1.y == p2.y;
    }
};

void flood(point first, bool occupy[51][51], unordered_set < point, pointhash, pointequal > * edge, char cowhide[51][51], int n, int m) {
    auto que = queue < point > ();
    auto eq = pointequal();
    que.push(first);
    while (!que.empty()) {
        auto p = que.front();
        if (!eq(p, first) && occupy[p.x][p.y]) {
            que.pop();
            continue;
        }
        occupy[p.x][p.y] = true;
        point nexts[] = {
            point(p.x + 1, p.y),
            point(p.x - 1, p.y),
            point(p.x, p.y + 1),
            point(p.x, p.y - 1)
        };
        for (auto next: nexts) {
            if (0 <= next.x && next.x <= n && 0 <= next.y && next.y <= m && !occupy[next.x][next.y]) {
                if (cowhide[next.x][next.y] == 'X') {
                    que.push(next);
                } else {
                    edge -> insert(p);
                }
            }
        }
        que.pop();
    }
}

int main() {
    char cowhide[51][51] {};
    bool occupy[51][51] {};
    auto edge = unordered_set < point, pointhash, pointequal > ();
    int n, m;
    cin >> n >> m;
    bool flag = true;
    point first;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cin >> cowhide[i][j];
            if (flag && cowhide[i][j] == 'X') {
                occupy[i][j] = true;
                first = point(i, j);
                flag = false;
            } else {
                occupy[i][j] = false;
            }
        }
    }

    flood(first, occupy, & edge, cowhide, n, m);

    int count = 0;
    auto nextedge = unordered_set < point, pointhash, pointequal > ();
    while (true) {
        for (auto p: edge) {
            point nexts[] = {
                point(p.x + 1, p.y),
                point(p.x - 1, p.y),
                point(p.x, p.y + 1),
                point(p.x, p.y - 1)
            };
            for (auto next: nexts) {
                if (0 <= next.x && next.x <= n && 0 <= next.y && next.y <= m && !occupy[next.x][next.y]) {
                    if (cowhide[next.x][next.y] == 'X') {
                        cout << count;
                        return 0;
                    }
                    cowhide[next.x][next.y] = 'X';
                    occupy[next.x][next.y] = true;
                    nextedge.insert(next);
                }
            }
        }
        count++;
        edge = nextedge;
        nextedge = unordered_set < point, pointhash, pointequal > ();
    }
    return 0;
}
```