---
title: "AcWing 4319. 合适数对"
date: 2022-03-26T20:44:54+08:00
draft: false
tags: ["acwing", "困难", "数论"]
math: true
---

给定一个长度为 n 的正整数数列 \\(a_1,a_2,...,a_n\\) 和一个正整数 k。

请你判断共有多少个数对 (l,r)

同时满足：

- 1≤l<r≤n

- 存在一个整数 x 使得 \\(a_l×a_r=x^k\\)成立

<!--more-->

#### 输入格式

第一行包含两个整数 n,k。

第二行包含 n 个正整数 \\(a_1,a_2,...,a_n\\)。

#### 输出格式

一个整数，表示满足条件的数对的数量。

#### 数据范围

前三个测试点满足 2≤n≤10。

所有测试点满足 \\(2≤n≤10^5，2≤k≤100，1≤ai≤10^5\\)。

#### 输入样例

```
6 3
1 3 9 8 24 1
```

#### 输出样例

```
5
```

## 题解

```cpp
#include<iostream>
#include<vector>
#include<map>

using namespace std;

int main() {
    unsigned int n;
    unsigned int k;
    cin >> n >> k;
    vector<unsigned int> a(n);
    vector<vector<pair<unsigned int, unsigned int>>> factors(n);
    map<vector<pair<unsigned int, unsigned int>>, unsigned int> factor_status_count;
    for(int i = 0; i < n; i++) {
        cin >> a[i];
        for(unsigned int factor = 2; factor * factor <= a[i]; factor++) {
            if((a[i] % factor) != 0u) {
                continue;
            }
            unsigned int count = 0;
            while(a[i] % factor == 0) {
                a[i] /= factor;
                ++count;
            }
            count %= k;
            if(count != 0) {
                factors[i].emplace_back(factor, count);
            }
        }
        if(a[i] != 1) {
            factors[i].emplace_back(a[i], 1);
        }
    }
    unsigned long long ans = 0;
    for(unsigned int i = 0; i < n; i++) {
        vector<pair<unsigned int, unsigned int>> factor_status;
        for(auto [factor, count]: factors[i]) {
            factor_status.emplace_back(factor, k - count);
        }
        ans += factor_status_count[factor_status];
        ++factor_status_count[factors[i]];
    }
    cout << ans;
    return 0;
}
```
