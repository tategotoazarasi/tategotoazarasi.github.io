---
title: "AcWing 2014. 岛"
date: 2022-01-06T10:51:11+08:00
draft: false
tags: ["acwing", "简单"]
math: true
---

每当下雨时，农夫约翰的田地总是被洪水淹没。

由于田地不是完全水平的，所以一些地方充满水后，留下了许多被水隔开的“岛”。

约翰的田地被描述为由 N 个连续高度值 H1,…,HN 指定的一维场景。

假设该场景被无限高的围墙包围着，请考虑暴雨期间发生的情况：

最低处首先被水覆盖，形成一些不连贯的岛，随着水位的不断上升，这些岛最终都会被覆盖。

一旦水位等于一块田地的高度，那块田地就被认为位于水下。

![fig_islands.png](https://tategotoazarasi.github.io/images/19_8c986a0cb5-fig_islands.png)

上图显示了一个示例：在左图中，我们只加入了刚好超过 1 单位的水，此时剩下 4 个岛（最大岛屿剩余数量），而在右图中，我们共加入了 7 单位的水，此时仅剩下 2 个岛。

请计算，暴风雨期间我们能在某个时间点看到的最大岛屿数量。

水会一直上升到所有田地都在水下。

<!--more-->

#### 输入格式

第一行包含整数 N。

接下来 N 行，每行包含一个整数表示 Hi。

#### 输出格式

输出暴风雨期间我们能在某个时间点看到的最大岛屿数量。

#### 数据范围

1≤N≤105,  
1≤Hi≤109

#### 输入样例：

```
8
3
5
2
3
1
4
2
3
```

#### 输出样例：

```
4
```

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <set>

using namespace std;

int main() {
    int n;
    cin >> n;
    auto above = new bool[n];
    auto heights = set<unsigned int>();
    auto m = map<unsigned int, vector<unsigned int>>();
    for (int i = 0; i < n; i++) {
        int height;
        cin >> height;
        heights.insert(height);
        above[i] = true; //露出水面
        if (m.count(height) == 0) {
            m[height] = vector<unsigned int>();
        }
        m[height].push_back(i);
    }
    int count = 1;
    int max = count;
    for (auto height: heights) {
        for (auto index: m[height]) {
            above[index] = false;
            if (0 < index && index + 1 < n) {
                if (above[index - 1] && above[index + 1]) {
                    count++;
                } else if (!above[index - 1] && !above[index + 1]) {
                    count--;
                }
            } else if ((index == 0 && !above[1]) || (index == n - 1 && !above[n - 2])) {
                count--;
            }
        }
        if (max < count) {
            max = count;
        }
    }
    cout << max;
    return 0;
}
```
