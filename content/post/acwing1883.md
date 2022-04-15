---
title: "AcWing 1883. 删减"
date: 2022-04-15T13:43:00+08:00
draft: false
tags: ["acwing", "简单", "字符串"]
math: true
---

农夫约翰把杂志上所有的文章摘抄了下来并把它变成了一个长度不超过 106 的字符串 S。

他希望从 S 中删除子串 T。

农夫约翰在 S 中从头开始寻找子串 T，一旦找到，就将它从 S 中删除，然后再次**从头开始**寻找（而不是接着往下找）。

他重复这个操作直到 S 中没有子串 T 为止。

注意，删除一个子串 T 可能会导致一个新的子串 T 的出现。

请帮助约翰完成这些操作并输出最后的 S。

<!--more-->

#### 输入格式

第一行包含字符串 S。

第二行包含字符串 T。

#### 输出格式

输出操作完成后的 S。

保证最终字符串 S 不为空。

#### 数据范围

S 的长度不超过 106，T 的长度不超过 100 且不超过 S 的长度，字符串中只出现小写字母。

#### 输入样例

```
whatthemomooofun
moo
```

#### 输出样例

```
whatthefun
```

```cpp
#include <iostream>
#include <cstring>

using namespace std;

string s,t,res;

int main() {
    cin >> s >> t;
    int slen = s.length(),tlen = t.length();
    //一边添加一边判断
    for(int i = 0; i < slen; i++) {
        res += s[i];
        //当添加到长度大于等于t字符串时，开始比较
        //每次从末尾向前匹配
        while(res.length() >= tlen && res.substr(res.length()-tlen,tlen) == t) {
            //存在t字符串，则删除
            res.erase(res.begin()+res.length()-tlen,res.end());
        }
    }
    cout << res << endl;
    return 0;
}
```
