---
title: "AcWing 2019. 拖拉机"
date: 2022-01-05T10:59:26+08:00
draft: false
tags: ["acwing", "中等", "广度优先搜索"]
math: true
---

干了一整天的活，农夫约翰完全忘记了他把拖拉机落在田地中央了。

他的奶牛非常调皮，决定对约翰来场恶作剧。

她们在田地的不同地方放了 N 捆干草，这样一来，约翰想要开走拖拉机就必须先移除一些干草捆。拖拉机的位置以及 N 捆干草的位置都是二维平面上的整数坐标点。

拖拉机的初始位置上没有干草捆。

当约翰驾驶拖拉机时，他只能沿平行于坐标轴的方向（北，南，东和西）移动拖拉机，并且拖拉机必须每次移动整数距离。

例如，驾驶拖拉机先向北移动 2 单位长度，然后向东移动 3 单位长度。

拖拉机无法移动到干草捆占据的位置。

请帮助约翰确定他需要移除的干草捆的最小数量，以便他能够将拖拉机开到二维平面的原点。

<!--more-->

#### 输入格式

第一行包含三个整数：N 以及拖拉机的初始位置 (x,y)。

接下来 N 行，每行包含一个干草捆的位置坐标 (x,y)。

#### 输出格式

输出约翰需要移除的干草捆的最小数量。

#### 数据范围

1≤N≤50000,  
1≤x,y≤1000

#### 输入样例：

```
7 6 3
6 2
5 2
4 3
2 1
7 3
5 4
6 4
```

#### 输出样例：

```
1
```

```cpp
#include<iostream>
#include <queue>
#include<cstring>

using namespace std;

const int N = 1000;

struct point {
    int x;
    int y;
    int step;

    point(int x, int y, int step) : x(x), y(y), step(step) {};
};

int bfs(point start, int field[1010][1010], int max_x, int max_y) {
    auto que = deque<point>();
    que.push_front(start);
    while (!que.empty()) {
        auto p = que.front();
        que.pop_front();
        point nexts[] = {point(p.x + 1, p.y, p.step), point(p.x - 1, p.y, p.step),
                         point(p.x, p.y + 1, p.step), point(p.x, p.y - 1, p.step)
                        };
        for (auto next: nexts) {
            if (next.x == 0 && next.y == 0) {
                return next.step;
            }
            if (0 <= next.x && next.x <= max_x+2 && 0 <= next.y && next.y <= max_y+2 &&
                    field[next.x][next.y] != 2) {
                if (field[next.x][next.y] == 0) {
                    que.push_front(next);
                } else {//field[next.first][next.second]==1
                    next.step++;
                    que.push_back(next);
                }
                field[next.x][next.y] = 2;
            }
        }
    }
    return -1;
}

int main() {
    int n, start_x, start_y;
    int field[N + 10][N + 10];   //记录状态。0=没有 1=墙 2=已经被搜索过

    cin >> n >> start_x >> start_y;
    memset(field, 0, sizeof field);
    int max_x = 0;
    int max_y = 0;
    for (int i = 0; i < n; i++) {
        int x, y;
        cin >> x >> y;
        if (max_x < x) {
            max_x = x;
        }
        if (max_y < y) {
            max_y = y;
        }
        field[x][y] = 1;
    }

    cout << bfs(point(start_x, start_y, 0), field, max_x, max_y);
    return 0;
}
```
