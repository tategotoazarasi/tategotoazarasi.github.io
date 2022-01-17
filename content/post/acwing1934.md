---
title: "AcWing 1934. 贝茜放慢脚步"
date: 2022-01-17T15:51:30+08:00
draft: false
tags: ["acwing", "简单", "二路归并"]
math: true
---

奶牛贝茜正在参加冬季哞林匹克运动会的越野滑雪比赛。

她以每秒 1 米的速度出发。

但是，随着时间的推移，她变得越来越疲倦，她开始放慢脚步。

每次放慢脚步，贝茜的速度都会降低：减速一次后，她以每秒 1/2 米的速度移动，减速两次后，则以每秒 1/3 米的速度移动，依此类推。

你将被告知何时何地贝茜会减速。

当减速信息格式为：

```
T 17
```

意味着，贝茜在某个时间点减速，本例表示比赛开始第 17 秒贝茜减速。

当减速信息格式为：

```
D 10
```

意味着，贝茜在某个地点减速，本例表示在行进 10 米处减速。

给定 N 个减速信息，请计算贝茜滑完一千米需要多少秒。

将你的答案四舍五入到最接近的整数（ 0.5 向上舍入为 1）。

#### 输入格式

第一行包含整数 N。

接下来 N 行，每行描述一个减速信息，格式为 `T x` 或 `D x`。

无论哪种情况，x 都是一个整数，保证所有减速都在贝茜滑完一千米前发生。

可能同时发生多次减速，那么这会使得贝茜的速度一下子变慢很多。

所有减速信息不一定按顺序给出。

#### 输出格式

输出贝茜滑完一千米所需的总时间。

#### 数据范围

1≤N≤10000

#### 输入样例：

```
2
T 30
D 10
```

#### 输出样例：

```
2970
```

#### 样例解释

贝茜以每秒 1 米的速度跑完前 10 米，耗时 10 秒。

然后她减速到每秒 1/2 米，接下来的 10 米要花 20 秒。

然后她在第 30 秒时，再次减速到每秒 1/3 米。

滑完剩下的 980 米需要 980×3=2940 秒。

因此，总时间是 10+20+2940=2970 秒。

```cpp
#include<iostream>
#include<algorithm>
#include<vector>
#include<cmath>

using namespace std;

int main() {
    int n;
    cin >> n;
    vector<int> t = vector<int>();
    vector<int> d = vector<int>();
    for(int i = 0; i < n; i++) {
        char op;
        int x;
        cin >> op >> x;
        if(op == 'T') {
            t.push_back(x);
        } else {
            d.push_back(x);
        }
    }
    sort(t.begin(), t.end());
    sort(d.begin(), d.end());
    double current_d     = 0;
    double current_t     = 0;
    double decelerations = 1;
    auto it_d            = d.begin();
    auto it_t            = t.begin();
    while(true) {
        int next_d = 1000;
        if(it_d != d.end()) {
            next_d = *it_d;
        }
        auto t_for_next_d = (next_d - current_d) * decelerations;//到下一个D的时间
        bool next_is_d    = true;
        if(it_t != t.end()) {
            next_is_d = t_for_next_d < *it_t - current_t;
        }
        if(next_is_d) {//先到D
            current_d = next_d;
            ++it_d;
            current_t += t_for_next_d;
        } else if(it_t != t.end()) {//先到T
            current_d += (*it_t - current_t) * (1.0 / decelerations);
            current_t = *it_t;
            ++it_t;
        }
        decelerations++;
        if(it_d == d.end() && it_t == t.end()) {
            current_t += (1000 - current_d) * decelerations;
            break;
        }
    }
    cout << lround(current_t);
    return 0;
}
```
