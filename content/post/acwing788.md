---
title: "AcWing 788. 逆序对的数量"
date: 2022-04-07T17:04:13+08:00
draft: false
tags: ["acwing", "简单", "归并排序"]
math: true
---

给定一个长度为 n 的整数数列，请你计算数列中的逆序对的数量。

逆序对的定义如下：对于数列的第 i 个和第 j 个元素，如果满足 i<j 且 a[i]>a[j]，则其为一个逆序对；否则不是。

<!--more-->

#### 输入格式

第一行包含整数 n，表示数列的长度。

第二行包含 n 个整数，表示整个数列。

#### 输出格式

输出一个整数，表示逆序对的个数。

#### 数据范围

1≤n≤100000，  
数列中的元素的取值范围 [1,109]。

#### 输入样例

```
6
2 3 4 5 6 1
```

#### 输出样例

```
5
```

```cpp
#include <iostream>

using namespace std;

void ms(int arr[],int l,int r,unsigned long long* ans) {
    if(l>=r) {
        return;
    }
    int m = l+r>>1;
    ms(arr,l,m,ans);
    ms(arr,m+1,r,ans);
    int i = l;
    int j = m+1;
    int p = 0;
    int tmp[100001];
    while(i<=m && j<=r) {
        if(arr[i]<=arr[j]) {
            tmp[p++] = arr[i++];
        } else {
            tmp[p++] = arr[j++];
            (*ans) += (m - i + 1);
        }
    }
    while(i<=m) {
        tmp[p++] = arr[i++];
    }
    while(j<=r) {
        tmp[p++] = arr[j++];
    }
    for(int i=0; i<p; i++) {
        arr[l+i] = tmp[i];
    }
}

int main()
{
    unsigned long long ans = 0;
    int n;
    int arr[100001];
    scanf("%d", &n);
    for(int i=0; i<n; i++) {
        scanf("%d", &arr[i]);
    }
    ms(arr,0,n-1,&ans);
    printf("%lld",ans);
}
```
