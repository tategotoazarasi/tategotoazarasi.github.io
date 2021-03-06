---
title: "AcWing 1996. 打乱字母"
date: 2022-01-09T18:39:18+08:00
draft: false
tags: ["acwing", "简单", "贪心", "二分"]
math: true
---

农夫约翰将按字典序排列的 N 头奶牛的名字列表贴在了牛棚的门上。

每个奶牛的名字都由一个长度介于 1 到 20 之间的由小写字母构成的唯一字符串表示。

麻烦制造者贝茜将列表中的奶牛名字重新排序打乱了列表。

此外，她还对每头奶牛的名字中的字母顺序进行了重新排列（也可能保持不变）。

给定修改过后的列表，请帮助约翰确定列表中的每个名字可能出现在原始列表中的最低和最高位置。

<!--more-->

#### 输入格式

第一行包含整数 N。

接下来 N 行，按照修改过后列表的顺序，给出了修改过后的奶牛的名字。

#### 输出格式

共 N 行，第 i 行输出给定的第 i 个字符串在原始列表中可能的最低和最高位置。

#### 数据范围

1≤N≤50000

#### 输入样例

```
4
essieb
a
xzy
elsie
```

#### 输出样例

```
2 3
1 1
4 4
2 3
```

#### 样例解释

无论如何，字符串 “a” 必然出现在原始列表中第一个，类似的，字符串 “xzy” 必然出现在原始列表中的最后一个。而字符串 “essieb” 和 “elsie” 都有可能位于原始列表的第 2 位或第 3 位，这取决于它们的原始字母顺序。例如，”bessie” 和 “elsie” 分别位于 2,3 位，”sisbee” 和 “ilees” 分别位于 3,2 位。

```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

bool cmp(char x, char y) {
    return x>y;
}

int main() {
    int n;
    cin >> n;
    auto names = vector<string>();
    names.resize(n);
    auto min_names = vector<string>();
    auto max_names = vector<string>();
    min_names.resize(n);
    max_names.resize(n);
    for(int i = 0; i < n; i++) {
        cin >> names[i];
        auto min_name_const_c_str = names[i].c_str();
        auto max_name_const_c_str = names[i].c_str();
        char *min_name_c_str = new char[names[i].length() + 1];
        char *max_name_c_str = new char[names[i].length() + 1];
        strcpy(min_name_c_str, min_name_const_c_str);
        strcpy(max_name_c_str, max_name_const_c_str);
        sort(min_name_c_str, min_name_c_str + names[i].length() * sizeof(char));
        sort(max_name_c_str, max_name_c_str + names[i].length() * sizeof(char), cmp);
        auto min_name = string(min_name_c_str);
        auto max_name = string(max_name_c_str);
        //sort(min_name.begin(), min_name.end());
        //sort(max_name.begin(), max_name.begin(), less<>());
        min_names[i] = min_name;
        max_names[i] = max_name;
        delete[]min_name_c_str;
        delete[]max_name_c_str;
    }
    auto min_names_sorted = vector<string>(min_names);
    auto max_names_sorted = vector<string>(max_names);
    sort(min_names_sorted.begin(), min_names_sorted.end());
    sort(max_names_sorted.begin(), max_names_sorted.end());
    for(int i = 0; i < n; i++) {
        auto min_name = min_names[i];
        auto max_name = max_names[i];
        cout << lower_bound(max_names_sorted.begin(), max_names_sorted.end(), min_name) - max_names_sorted.begin() +
             1
             << " "
             << upper_bound(min_names_sorted.begin(), min_names_sorted.end(), max_name) - min_names_sorted.begin()
             << endl;
    }
    return 0;
}
```
