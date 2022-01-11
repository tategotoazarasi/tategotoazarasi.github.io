---
title: "AcWing 1978. 奶牛过马路"
date: 2022-01-11T16:08:30+08:00
draft: false
tags: ["acwing", "简单", "排序", "前缀最值"]
math: true
---

每天，农夫约翰的 N 头奶牛都会穿过农场中间的马路。

考虑约翰的农场在二维平面的地图，马路沿水平方向延伸，马路的一侧由直线 y=0 描述，另一侧由直线 y=1 描述。

奶牛 i 从马路一侧的位置 (ai,0) 沿直线过马路到达另一侧的位置 (bi,1)。

所有 ai 互不相同，所有 bi 互不相同。

尽管他的奶牛们行动敏捷，他还是担心行动路径交叉的两头奶牛在过马路时发生碰撞。

约翰认为，如果一头奶牛的行动路径没有跟其他任何奶牛的行动路径相交，则该奶牛是安全的。

请帮助约翰计算安全奶牛的数量。

<!--more-->

#### 输入格式

第一行包含整数 N。

接下来 N 行，每行包含两个整数 ai,bi，用来描述一头牛的行动路径。

#### 输出格式

输出安全奶牛的数量。

#### 数据范围

1≤N≤105,  
−106≤ai,bi≤106

#### 输入样例：

```
4
-3 4
7 8
10 16
3 9
```

#### 输出样例：

```
2
```

#### 样例解释

第一头牛和第三头牛的行动路线不与其他奶牛的路线相交。

第二头牛和第四头牛的行动路线相交。

#### 题解

把所有点对按照 a 的值从小到大排序，对于第 i 个点，只要它之前所有点 b 的最大值小于 bi ，并且它之后的所有点的 b 的最小值大于 bi 就说明该线路是安全的。

用前缀 maximum 数组和后缀 minimum 数组优化时间复杂度。

```cpp
#include<iostream>
#include<cstring>
#include<algorithm>

using namespace std;

struct path {
    int a;
    int b;
    path(): a(0), b(0) {};
    path(int a, int b): a(a), b(b) {};
    bool operator<(const path &p) const {
        return this->a < p.a;
    }
};

int main() {
    int n;
    cin >> n;
    int *maximum = new int[n];
    int *minimum = new int[n];
    memset(maximum, 0, n * sizeof(int));
    memset(minimum, 0, n * sizeof(int));
    path *paths = new path[n];
    for(int i = 0; i < n; i++) {
        int a, b;
        cin >> a >> b;
        paths[i] = path(a, b);
    }
    sort(paths, paths + n);
    maximum[0]     = paths[0].b;    //i以前b的最大值
    minimum[n - 1] = paths[n - 1].b;//i以后b的最小值
    for(int i = 1; i < n; i++) {
        maximum[i] = max(maximum[i - 1], paths[i].b);
    }
    for(int i = n - 2; i >= 0; i--) {
        minimum[i] = min(minimum[i + 1], paths[i].b);
    }
    int ans = n;
    for(int i = 0; i < n; i++) {
        if(maximum[i] > paths[i].b || minimum[i] < paths[i].b) {
            ans--;
        }
    }
    cout << ans;
    delete[] paths;
    delete[] minimum;
    delete[] maximum;
    return 0;
}
```
