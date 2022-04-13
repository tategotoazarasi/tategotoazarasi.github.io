---
title: "LeetCode 380. Insert Delete GetRandom O(1)"
date: 2022-04-13T15:22:28+08:00
draft: false
tags: ["leetcode", "中等", "设计", "数组", "哈希表", "数学", "随机化"]
math: true
---

实现`RandomizedSet` 类：

- `RandomizedSet()` 初始化 `RandomizedSet` 对象
- `bool insert(int val)` 当元素 `val` 不存在时，向集合中插入该项，并返回 `true` ；否则，返回 `false` 。
- `bool remove(int val)` 当元素 `val` 存在时，从集合中移除该项，并返回 `true` ；否则，返回 `false` 。
- `int getRandom()` 随机返回现有集合中的一项（测试用例保证调用此方法时集合中至少存在一个元素）。每个元素应该有 **相同的概率** 被返回。

你必须实现类的所有函数，并满足每个函数的 **平均** 时间复杂度为 `O(1)` 。

<!--more-->

**示例：**

> **输入** > ["RandomizedSet", "insert", "remove", "insert", "getRandom", "remove", "insert", "getRandom"] > [[], [1], [2], [2], [], [1], [2], []]
>
> **输出** > [null, true, false, true, 2, true, false, 2]

**解释**

```java
RandomizedSet randomizedSet = new RandomizedSet();
randomizedSet.insert(1); // 向集合中插入 1 。返回 true 表示 1 被成功地插入。
randomizedSet.remove(2); // 返回 false ，表示集合中不存在 2 。
randomizedSet.insert(2); // 向集合中插入 2 。返回 true 。集合现在包含 [1,2] 。
randomizedSet.getRandom(); // getRandom 应随机返回 1 或 2 。
randomizedSet.remove(1); // 从集合中移除 1 ，返回 true 。集合现在包含 [2] 。
randomizedSet.insert(2); // 2 已在集合中，所以返回 false 。
randomizedSet.getRandom(); // 由于 2 是集合中唯一的数字，getRandom 总是返回 2 。
```

**提示：**

- \\(-2^{31} <= val <= 2^{31} - 1\\)
- \\(最多调用 insert、remove 和 getRandom 函数 2 \* 10^5 次\\)
- 在调用 `getRandom` 方法时，数据结构中 **至少存在一个** 元素。

## 题解

这道题要求实现一个类，满足插入、删除和获取随机元素操作的平均时间复杂度为 \\(O(1)\\)。

变长数组可以在 \\(O(1)\\) 的时间内完成获取随机元素操作，但是由于无法在 \\(O(1)\\) 的时间内判断元素是否存在，因此不能在 \\(O(1)\\) 的时间内完成插入和删除操作。

哈希表可以在 \\(O(1)\\) 的时间内完成插入和删除操作，但是由于无法根据下标定位到特定元素，因此不能在 \\(O(1)\\) 的时间内完成获取随机元素操作。

为了满足插入、删除和获取随机元素操作的时间复杂度都是 \\(O(1)\\)，需要将变长数组和哈希表结合，变长数组中存储元素，哈希表中存储每个元素在变长数组中的下标。

插入操作时，首先判断 \\(\textit{val}\\) 是否在哈希表中，如果已经存在则返回 \\(\text{false}\\)，如果不存在则插入 \\(\textit{val}\\)，操作如下：

1. 在变长数组的末尾添加 \\(\textit{val}\\)；
2. 在添加 \\(\textit{val}\\) 之前的变长数组长度为 \\(\textit{val}\\) 所在下标 \\(\textit{index}\\)，将 \\(\textit{val}\\) 和下标 \\(\textit{index}\\) 存入哈希表；
3. 返回 \\(\text{true}\\)。

删除操作时，首先判断 \\(\textit{val}\\) 是否在哈希表中，如果不存在则返回 \\(\text{false}\\)，如果存在则删除 \\(\textit{val}\\)，操作如下：

1. 从哈希表中获得 \\(\textit{val}\\) 的下标 \\(\textit{index}\\)；
2. 将变长数组的最后一个元素 \\(\textit{last}\\) 移动到下标 \\(\textit{index}\\) 处，在哈希表中将 \\(\textit{last}\\) 的下标更新为 \\(\textit{index}\\)；
3. 在变长数组中删除最后一个元素，在哈希表中删除 \\(\textit{val}\\)
4. 返回 \\(\text{true}\\)。

删除操作的重点在于将变长数组的最后一个元素移动到待删除元素的下标处，然后删除变长数组的最后一个元素。

该操作的时间复杂度是 \\(O(1)\\)，且可以保证在删除操作之后变长数组中的所有元素的下标都连续，方便插入操作和获取随机元素操作。

获取随机元素操作时，由于变长数组中的所有元素的下标都连续，因此随机选取一个下标，返回变长数组中该下标处的元素。

**复杂度分析**

- 时间复杂度：初始化和各项操作的时间复杂度都是 \\(O(1)\\)。

- 空间复杂度：\\(O(n)\\)，其中 \\(n\\) 是集合中的元素个数。

存储元素的数组和哈希表需要 \\(O(n)\\) 的空间。

```cpp
class RandomizedSet {
    unordered_map<int, int> map;
    vector<int> nums;
    default_random_engine generator;
    uniform_int_distribution<int> distribution;
public:
    RandomizedSet() {
        generator    = default_random_engine(time(nullptr));
        distribution = uniform_int_distribution<int>(0, INT_MAX);
    }

    bool insert(int val) {
        if(map.count(val)) {
            return false;
        }
        nums.push_back(val);
        map[val] = nums.size() - 1;
        return true;
    }

    bool remove(int val) {
        if(!map.count(val)) {
            return false;
        }
        int index   = map[val];
        int last    = nums.back();
        nums[index] = last;
        map[last]   = index;
        nums.pop_back();
        map.erase(val);
        return true;
    }

    int getRandom() {
        return nums[distribution(generator) % nums.size()];
    }
};
```
