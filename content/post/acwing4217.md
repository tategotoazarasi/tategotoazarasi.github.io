---
title: "AcWing 4217. 机器人移动"
date: 2022-01-29T20:41:54+08:00
draft: false
tags: ["acwing", "困难", "二分"]
math: true
---

在一个无限大的二维平面上有一个机器人。

初始时，机器人位于点 (0,0)。

机器人可以执行四种行动指令：

- `U` — 从 (x,y)移动到 (x,y+1)；

- `D` — 从 (x,y)移动到 (x,y−1)；

- `L` — 从 (x,y)移动到 (x−1,y)；

- `R` — 从 (x,y)移动到 (x+1,y)。

给定一个长度为 n 的指令序列，指令编号 1∼n，机器人将按顺序依次执行序列中的每个行动指令。

我们希望机器人最终抵达目标地点 (a,b)。

为了达成这一目的，我们可能需要对指令序列进行修改。

每次修改可以选择其中一个指令，并将其**替换**为四种指令之一。

注意，只能对序列中的指令进行替换，不得随意删除指令或添加额外指令。

不妨设经过修改的指令中，编号最小的指令编号为 minID，编号最大的指令编号为 maxID。

我们定义修改成本为 maxID−minID+1。

例如，将 `RRRRRRR` 修改为 `RLRRLRL`，则编号为 2,5,7 的指令经过了修改，修改成本为 7−2+1=6。

请你计算，为了使得机器人能够最终抵达目标点 (a,b)，所需花费的最小修改成本。

如果不需要对序列进行修改，则成本为 0。

<!--more-->

#### 输入格式

第一行包含整数 n。

第二行包含一个长度为 n 的字符串，表示指令序列，字符串中只包含 `U`，`D`，`L`，`R`。

第三行包含两个整数 a,b，表示机器人的目标位置为 (a,b)。

#### 输出格式

输出一个整数，表示最小修改成本。

如果无论如何修改，机器人都无法抵达目标位置，则输出 −1。

#### 数据范围

前四个测试点满足 1≤n≤10。

所有测试点满足 1≤n≤2×105，−109≤x,y≤109。

#### 输入样例 1：

```
5
RURUU
-2 3
```

#### 输出样例 1：

```
3
```

#### 输入样例 2：

```
4
RULR
1 1
```

#### 输出样例 2：

```
0
```

#### 输入样例 3：

```
3
UUU
100 100
```

#### 输出样例 3：

```
-1
```

```cpp
#include<iostream>
#include<vector>

using namespace std;

vector<char> ops;
vector<pair<int, int>> forth;
vector<pair<int, int>> back;
unsigned int n;
int a;
int b;

bool check(int len) {
    for(int i = 0; i + len < n + 1; i++) {
        const auto s    = forth[i];
        const auto e    = back[i + len];
        const auto dist = abs(s.first - e.first) + abs(s.second - e.second);
        if(len - dist >= 0 && (len - dist) % 2 == 0) {
            return true;
        }
    }
    return false;
}

int main() {
    cin >> n;
    ops   = vector<char>();
    forth = vector<pair<int, int>>();
    back    = vector<pair<int, int>>();
    ops.resize(n);
    forth.emplace_back(0, 0);
    for(int i = 0; i < n; i++) {
        cin >> ops[i];
        switch(ops[i]) {
        case 'U': {
            forth.emplace_back(forth.back().first, forth.back().second + 1);
            break;
        }
        case 'D': {
            forth.emplace_back(forth.back().first, forth.back().second - 1);
            break;
        }
        case 'L': {
            forth.emplace_back(forth.back().first - 1, forth.back().second);
            break;
        }
        case 'R': {
            forth.emplace_back(forth.back().first + 1, forth.back().second);
            break;
        }
        }
    }
    cin >> a >> b;
    if(forth.back().first == a && forth.back().second == b) {
        cout << 0;
        return 0;
    }
    if(n< (abs(a) + abs(b)) || (n - (abs(a) + abs(b))) % 2 != 0) {
        cout << -1;
        return 0;
    }
    back.emplace_back(a, b);
    for(int i = n - 1; i >= 0; i--) {
        switch(ops[i]) {
        case 'U': {
            back.emplace_back(back.back().first, back.back().second - 1);
            break;
        }
        case 'D': {
            back.emplace_back(back.back().first, back.back().second + 1);
            break;
        }
        case 'L': {
            back.emplace_back(back.back().first + 1, back.back().second);
            break;
        }
        case 'R': {
            back.emplace_back(back.back().first - 1, back.back().second);
            break;
        }
        }
    }
    back  = vector(back.rbegin(), back.rend());
    int l = 1;
    int r = n;
    while(l < r) {
        const int mid = (l + r) / 2;
        if(check(mid)) {
            r = mid;
        } else {
            l = mid + 1;
        }
    }
    cout << l;
    return 0;
}
```
