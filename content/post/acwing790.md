---
title: "AcWing 790. 数的三次方根"
date: 2022-04-07T17:04:33+08:00
draft: false
tags: ["acwing", "简单", "二分"]
math: true
---

给定一个浮点数 n，求它的三次方根。

<!--more-->

#### 输入格式

共一行，包含一个浮点数 n。

#### 输出格式

共一行，包含一个浮点数，表示问题的解。

注意，结果保留 6 位小数。

#### 数据范围

−10000≤n≤10000

#### 输入样例

```
1000.00
```

#### 输出样例

```
10.000000
```

```cpp
#include <iostream>
#include <iomanip>

using namespace std;

int main() {
    double n;
    cin>>n;
    bool positive = n>=0;
    if(n<0) {
        n = -n;
    }
    double l = 0;
    double r = max(1.0,n);
    double ans = 0;
    while(r-l>=1e-7) {
        double m = (l+r)/2;
        double mmm = m*m*m;
        if(mmm<n) {
            l = m;
        } else if(mmm>n) {
            r = m;
        } else {
            break;
        }
    }
    ans = (l+r)/2;
    if(!positive) {
        cout<<'-';
    }
    cout<<fixed<<setprecision(6)<<ans;
    return 0;
}
```
