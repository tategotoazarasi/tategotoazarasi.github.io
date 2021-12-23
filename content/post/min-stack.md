---
title: "LeetCode 155. 最小栈"
date: 2021-12-23T15:25:43+08:00
draft: false
tags: ["leetcode", "简单", "栈", "设计"]
math: true
---

设计一个支持 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。

- `push(x)` —— 将元素 x 推入栈中。
- `pop()` —— 删除栈顶的元素。
- `top()` —— 获取栈顶元素。
- `getMin()` —— 检索栈中的最小元素。

<!--more-->

**示例:**

**输入：**
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

**输出：**
[null,null,null,null,-3,null,0,-2]

**解释：**
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin(); --> 返回 -3.
minStack.pop();
minStack.top(); --> 返回 0.
minStack.getMin(); --> 返回 -2.

**提示：**

- `pop`、`top` 和 `getMin` 操作总是在 **非空栈** 上调用。
- 考虑到堆栈中的每个节点都有一个最小值。 (Credits to @aakarshmadhavan)

```java
class MinStack {
    private final Stack<Integer> stack;
    private Integer min;
    private int minCount;

    public MinStack() {
        this.min = null;
        this.stack = new Stack<>();
    }

    public void push(int val) {
        stack.push(val);
        if (min == null) {
            min = val;
            minCount = 1;
        } else if (min > val) {
            min = val;
            minCount = 1;
        } else if (min == val) {
            minCount++;
        }
    }

    public void pop() {
        int p = stack.pop();
        if (stack.empty()) {
            min = null;
            minCount = 0;
            return;
        }
        if (p == min) {
            minCount--;
        }
        if (minCount == 0) {
            min = stack.peek();
            minCount = 0;
            for (int i : stack) {
                if (min > i) {
                    min = i;
                    minCount = 1;
                } else if (min == i) {
                    minCount++;
                }
            }
        }
    }

    public int top() {
        return stack.peek();
    }

    public int getMin() {
        return min;
    }
}
```
