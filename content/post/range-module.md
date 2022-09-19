---
title: "LeetCode 715. Range 模块"
date: 2022-09-19T21:11:46+08:00
draft: false
tags: ["leetcode", "困难", "设计", "线段树", "有序集合"]
math: true
---

Range 模块是跟踪数字范围的模块。设计一个数据结构来跟踪表示为 **半开区间**的范围并查询它们。

**半开区间** `[left, right)`  表示所有  `left <= x < right`  的实数 `x` 。

实现 `RangeModule` 类:

- `RangeModule()`  初始化数据结构的对象。
- `void addRange(int left, int right)` 添加 **半开区间** `[left, right)`，跟踪该区间中的每个实数。添加与当前跟踪的数字部分重叠的区间时，应当添加在区间  `[left, right)`  中尚未跟踪的任何数字到该区间中。
- `boolean queryRange(int left, int right)`  只有在当前正在跟踪区间  `[left, right)`  中的每一个实数时，才返回`true` ，否则返回 `false` 。
- `void removeRange(int left, int right)`  停止跟踪
  **半开区间** `[left, right)`  中当前正在跟踪的每个实数。

<!--more-->

**示例 1：**

    输入
    ["RangeModule", "addRange", "removeRange", "queryRange", "queryRange", "queryRange"]
    [[], [10, 20], [14, 16], [10, 14], [13, 15], [16, 17]]
    输出
    [null, null, null, true, false, true]

    解释
    RangeModule rangeModule = new RangeModule();
    rangeModule.addRange(10, 20);
    rangeModule.removeRange(14, 16);
    rangeModule.queryRange(10, 14); 返回 true （区间 [10, 14) 中的每个数都正在被跟踪）
    rangeModule.queryRange(13, 15); 返回 false（未跟踪区间 [13, 15) 中像 14, 14.03, 14.17 这样的数字）
    rangeModule.queryRange(16, 17); 返回 true （尽管执行了删除操作，区间 [16, 17) 中的数字 16 仍然会被跟踪）

**提示：**

- \\(1 <= left < right <= 10^9\\)
- 在单个测试用例中，对  `addRange` 、  `queryRange`  和 `removeRange` 的调用总数不超过  \\(10^4\\)  次

## 题解

### 方法一：有序集合 / 有序映射

**思路与算法**

我们可以使用有序集合或者有序映射来实时维护所有的区间。在任意一次 \\(\text{addRange}\\) 或 \\(\text{removeRange}\\) 操作后，我们需要保证有序集合中的区间两两不能合并成一个更大的连续区间。也就是说：如果当前有序集合中有 \\(k\\) 个区间 \\([l_1, r_1), [l_2, r_2), \cdots, [l_k, r_k)\\)，那么需要保证：

$$
l_1 < r_1 < l_2 < r_2 < \cdots < l_k < r_k
$$

成立。这样一来 \\(\text{queryRange}\\) 操作就会变得非常方便：对于 \\(\text{queryRange(left, right)}\\) 而言，我们只需要判断是否存在一个区间 \\([l_i, r_i)\\)，满足 \\(l_i \leq \textit{left} < \textit{right} \leq r_i\\) 即可。

接下来我们详细讲解如何处理 \\(\text{addRange}\\) 或 \\(\text{removeRange}\\) 和操作。对于 \\(\text{addRange(left, right)}\\) 操作，我们首先在有序集合上进行二分，找出最后一个满足 \\(l_i \leq \textit{left}\\) 的区间 \\([l_i, r_i)\\)，那么会有如下的四种情况：

- 如果不存在这样的区间，那么我们可以忽略这一步；

- 如果 \\(l_i \leq \textit{left} < \textit{right} \leq r_i\\)，即 \\([l_i, r_i)\\) 完全包含待添加的区间，那么我们不需要进行任何操作，可以直接返回；

- 如果 \\(l_i \leq \textit{left} \leq r_i < \textit{right}\\)，我们需要删除区间 \\([l_i, r_i)\\)，并把 \\(\textit{left}\\) 置为 \\(l_i\\)。此时 \\([\textit{left}, \textit{right})\\) 就表示待添加区间与 \\([l_i, r_i)\\) 的并集；

- 如果 \\(l_i < r_i < \textit{left} < \textit{right}\\)，那么我们也可以忽略这一步。

随后，我们遍历 \\([l_i, r_i)\\) 之后的区间（如果前面不存在满足要求的 \\([l_i, r_i)\\)，那么就从头开始遍历），这些区间 \\([l_j, r_j)\\) 都满足 \\(l_j > \textit{left}\\)，那么只要 \\(l_j \leq \textit{right}\\)，\\([l_j, r_j)\\) 就可以与 \\([\textit{left}, \textit{right})\\) 合并成一个更大的连续区间。当遍历到 \\(l_j > \textit{right}\\) 时，根据集合的有序性，之后的所有区间都不会和 \\([\textit{left}, \textit{right})\\) 有交集，就可以结束遍历。

在遍历完成后，我们还需要将 \\([\textit{left}, \textit{right})\\) 加入有序集合中。

对于 \\(\text{removeRange(left, right)}\\) 操作，我们的处理方法是类似的，首先在有序集合上进行二分，找出最后一个满足 \\(l_i \leq \textit{left}\\) 的区间 \\([l_i, r_i)\\)，那么会有如下的四种情况：

- 如果不存在这样的区间，那么我们可以忽略这一步；

- 如果满足 \\(l_i \leq \textit{left} \leq \textit{right} \leq r_i\\)，即 \\([l_i, r_i)\\) 完全包含待添加的区间，那么 \\(l_i \leq \textit{left} \leq \textit{right} \leq r_i\\) 的删除会导致 \\([l_i, r_i)\\) 变成两个短区间：\\([l_i, \textit{left})\\) 和 \\([\textit{right}, r_i)\\)。如果 \\(\textit{left} = l_i\\)，那么第一个区间为空区间；如果 \\(\textit{right} = r_i\\)，那么第二个区间为空区间。我们将 \\([l_i, r_i)\\) 删除后，向有序集合中添加所有的非空区间，即可直接返回；

- 如果 \\(l_i \leq \textit{left} < r_i < \textit{right}\\)，我们把区间 \\([l_i, r_i)\\) 变成 \\([l_i, \textit{left})\\) 即可。特别地，如果 \\(\textit{left} = l_i\\)，我们可以直接把这个区间删除；

- 如果 \\(l_i < r_i \leq \textit{left} < \textit{right}\\)，那么我们也可以忽略这一步。

随后，我们遍历 \\([l_i, r_i)\\) 之后的区间，这些区间 \\([l_j, r_j)\\) 都满足 \\(l_j > \textit{left}\\)，那么只要 \\(l_j < \textit{right}\\)，\\([l_j, r_j)\\) 中的一部分就会被删除。如果 \\(r_j \leq \textit{right}\\)，那么 \\([l_j, r_j)\\) 会被完全删除；如果 \\(r_j > \textit{right}\\)，那么 \\([l_j, r_j)\\) 会剩下 \\([\textit{right}, r_j)\\)，此时之后的所有区间都不会和 \\([\textit{left}, \textit{right})\\) 有交集，就可以结束遍历。

最后，对于 \\(\text{queryRange(left, right)}\\) 操作，我们同样在有序集合上进行二分，找出最后一个满足 \\(l_i \leq \textit{left}\\) 的区间 \\([l_i, r_i)\\)。如果 \\(l_i \leq \textit{left} < \textit{right} \leq r_i\\)，那么返回 \\(\text{True}\\)，否则返回 \\(\text{False}\\)。

**代码**

```cpp
class RangeModule {
public:
    RangeModule() {}

    void addRange(int left, int right) {
        auto it = intervals.upper_bound(left);
        if (it != intervals.begin()) {
            auto start = prev(it);
            if (start->second >= right) {
                return;
            }
            if (start->second >= left) {
                left = start->first;
                intervals.erase(start);
            }
        }
        while (it != intervals.end() && it->first <= right) {
            right = max(right, it->second);
            it = intervals.erase(it);
        }
        intervals[left] = right;
    }

    bool queryRange(int left, int right) {
        auto it = intervals.upper_bound(left);
        if (it == intervals.begin()) {
            return false;
        }
        it = prev(it);
        return right <= it->second;
    }

    void removeRange(int left, int right) {
        auto it = intervals.upper_bound(left);
        if (it != intervals.begin()) {
            auto start = prev(it);
            if (start->second >= right) {
                int ri = start->second;
                if (start->first == left) {
                    intervals.erase(start);
                }
                else {
                    start->second = left;
                }
                if (right != ri) {
                    intervals[right] = ri;
                }
                return;
            }
            else if (start->second > left) {
                if (start->first == left) {
                    intervals.erase(start);
                }
                else {
                    start->second = left;
                }
            }
        }
        while (it != intervals.end() && it->first < right) {
            if (it->second <= right) {
                it = intervals.erase(it);
            }
            else {
                intervals[right] = it->second;
                intervals.erase(it);
                break;
            }
        }
    }

private:
    map<int, int> intervals;
};
```

**复杂度分析**

- 时间复杂度：对于操作 \\(\text{queryRange}\\)，时间复杂度为 \\(O(\log(a+r))\\)，其中 \\(a\\) 是操作 \\(\text{addRange}\\) 的次数，\\(r\\) 是操作 \\(\text{removeRange}\\) 的次数。对于操作 \\(\text{addRange}\\) 和 \\(\text{removeRange}\\)，时间复杂度为均摊 \\(O(\log(a+r))\\)，这是因为 \\(\text{addRange}\\) 操作最多添加一个区间，\\(\text{removeRange}\\) 最多添加两个区间，每一个添加的区间只会在未来的操作中被移除一次，因此均摊时间复杂度为对有序集合 / 有序映射常数次操作需要的时间，即为 \\(O(\log(a+r))\\)。

- 空间复杂度：\\(O(a+r)\\)，即为有序集合 / 有序映射需要使用的空间。

### 方法二：柯朵莉树

#### 名称简介

珂朵莉树（Chtholly Tree），又名老司机树 ODT（Old Driver Tree）。起源自[CF896C](https://codeforces.com/problemset/problem/896/C)。

注意，这种想法的本质是基于数据随机的“颜色段均摊”，而不是一种数据结构，下文介绍的操作是这种想法的具体实现方法。

#### 前置知识

会用 STL 的 set 就行。

#### 核心思想

把值相同的区间合并成一个结点保存在 set 里面。

#### 用处

骗分。只要是有区间赋值操作的数据结构题都可以用来骗分。在数据随机的情况下一般效率较高，但在不保证数据随机的场合下，会被精心构造的特殊数据卡到超时。

如果要保证复杂度正确，必须保证数据随机。详见 [Codeforces 上关于珂朵莉树的复杂度的证明](http://codeforces.com/blog/entry/56135?#comment-398940)。

更详细的严格证明见[珂朵莉树的复杂度分析](https://zhuanlan.zhihu.com/p/102786071)。对于 add，assign 和 sum 操作，用 set 实现的珂朵莉树的复杂度为，而用链表实现的复杂度为 。

#### 正文

首先，结点的保存方式：

```cpp
struct Node_t {
  int l, r;
  mutable int v;

  Node_t(const int &il, const int &ir, const int &iv) : l(il), r(ir), v(iv) {}

  inline bool operator<(const Node_t &o) const { return l < o.l; }
};
```

其中，`int v` 是你自己指定的附加数据。

`mutable` 关键字的含义是什么？

`mutable` 的意思是“可变的”，让我们可以在后面的操作中修改 `v` 的值。在 C++ 中，mutable 是为了突破 const 的限制而设置的。被 mutable 修饰的变量（mutable 只能用于修饰类中的非静态数据成员），将永远处于可变的状态，即使在一个 const 函数中。

这意味着，我们可以直接修改已经插入 `set` 的元素的 `v`值，而不用将该元素取出后重新加入 `set`。

然后，我们定义一个 `set<Node_t> odt;` 来维护这些结点。为简化代码，可以`typedef set<Node_t>::iterator iter`，当然在题目支持 C++11 时也可以使用`auto`。

##### split

`split` 是最核心的操作之一，它用于将原本包含点 x 的区间（设为[l,r]）分裂为两个区间 [l,x) 和 [x,r] 并返回指向后者的迭代器。 参考代码如下：

```cpp
auto split(int x) {
  if (x > n) return odt.end();
  auto it = --odt.upper_bound(Node_t{x, 0, 0});
  if (it->l == x) return it;
  int l = it->l, r = it->r, v = it->v;
  odt.erase(it);
  odt.insert(Node_t(l, x - 1, v));
  return odt.insert(Node_t(x, r, v)).first;
}
```

这个玩意有什么用呢？ 任何对于[l,r]的区间操作，都可以转换成 set 上 [split(l),split(r+1)) 的操作。

##### assign

另外一个重要的操作 `assign` 用于对一段区间进行赋值。 对于 ODT
来说，区间操作只有这个比较特殊，也是保证复杂度的关键。 如果 ODT
里全是长度为 的区间，就成了暴力，但是有了 `assign`，可以使 ODT
的大小下降。 参考代码如下：

```cpp
void assign(int l, int r, int v) {
  auto itr = split(r + 1), itl = split(l);
  odt.erase(itl, itr);
  odt.insert(Node_t(l, r, v));
}
```

##### 其他操作

套模板就好了，参考代码如下：

```cpp
void performance(int l, int r) {
  auto itr = split(r + 1), itl = split(l);
  for (; itl != itr; ++itl) {
    // Perform Operations here
  }
}
```

注：**珂朵莉树在进行求取区间左右端点操作时，必须先 split 右端点，再 split 左端点**。若先 split 左端点，返回的迭代器可能在 split 右端点的时候失效，可能会导致 RE。

#### 习题

- [「SCOI2010」序列操作](https://www.luogu.com.cn/problem/P2572)
- [「SHOI2015」脑洞治疗仪](https://loj.ac/problem/2037)
- [「Luogu 2787」理理思维](https://www.luogu.com.cn/problem/P2787)
- [「Luogu 4979」矿洞：坍塌](https://www.luogu.com.cn/problem/P4979)
