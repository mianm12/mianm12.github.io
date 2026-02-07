---
title: Day 01 - 数组 Part 01
date: 2026-02-04T11:56:24+08:00
draft: false
description: 数组基础与三道经典数组题：二分查找、原地移除元素（暴力/快慢指针）、有序数组平方（暴力/双指针），含 C++ 实现与复杂度分析。
summary: 本文梳理数组的核心特性（0 索引、内存连续、不可删除只能覆盖），并讲解三道 LeetCode 数组题：二分查找、移除元素、以及有序数组平方。包含暴力与优化思路（快慢指针/双指针）、C++ 代码示例及时间与空间复杂度总结。
tags:
  - 数组
  - 二分查找
  - 双指针
  - cpp
categories:
  - 数据结构
  - 算法
  - 刷题
series: [代码随想录]
series_order: 1
lastmod: 2026-02-04T15:44:00+08:00
---

{{< katex >}}

数组是**相同类型数据**的集合

- 下标从 0 开始
- 内存空间地址是连续的
- 数组元素无法删除只能覆盖

## [二分查找](https://leetcode.cn/problems/binary-search/)

- 要求**有序数组**
- 存在重复元素时返回下标可能不唯一

```cpp
class Solution
{
public:
    int search(vector<int> &nums, int target)
    {
        int left, right;
        left = 0;
        right = nums.size() - 1;

        while (left <= right) // traget 在闭区间[left, right]中
        {
            int mid = (left + right) / 2;
            int current = nums[mid];

            if (current > target)
            { // traget 在左半边
                right = mid - 1;
            }
            else if (current < target)
            { // traget 在右半边
                left = mid + 1;
            }
            else
            { // current == traget
                return mid;
            }
        }

        return -1;
    }
};
```

复杂度：

- 时间复杂度：\(O(\log(n))\)
- 空间复杂度：\(O(1)\)

## [移除元素](https://leetcode.cn/problems/remove-element/)

给出一个数组 nums 和一个值 val，需要**原地**移除所有数值等于 val 的元素，并返回移除后数组的新长度。

### 暴力解法

使用两层 for 循环，遇到目标元素就将后续元素移上来

```cpp
class Solution
{
public:
    int removeElement(vector<int>& nums, int val) {
        int n = nums.size(); // 当前待处理的数组长度
        int k = 0; // 新数组长度

        for (int i = 0; i < n; i++) {
            if (nums[i] == val) { // 发现目标元素将后续元素前移
                for (int j = i; j < n - 1; j++) {
                    nums[j] = nums[j + 1];
                }
                i--; // 后续元素前移，i 也要向前移动一位
                n--; // 移除一个元素，数组大小-1
            } else {
                k++;
            }
        }

        return k;
    }
};
```

复杂度：

- 时间复杂度：\(O(\log(n^2))\)
- 空间复杂度：\(O(1)\)

### 快慢指针

通过快指针和慢指针在一个 for 循环中完成两个 for 循环的工作

- 快指针：遍历所有元素，寻找新数组的元素
- 慢指针：指向新数组中末尾待更新元素的位置

```cpp
class Solution
{
public:
    int removeElement(vector<int>& nums, int val) {
        int fast, slow = 0;
        int n = nums.size();
        
        for (fast = 0; fast < n; fast++) {
            if (nums[fast] != val) { // 需要保留不等于 `val` 的元素
                if (fast != slow)    // 只有双指针指向不同元素时才移动
                    nums[slow] = nums[fast];
                slow++; // 更新慢指针，指向下一个需要被赋值的位置
            }
        }

        return slow;
    }
};
```

复杂度：

- 时间复杂度：\(O(n)\)
- 空间复杂度：\(O(1)\)

## [有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/)

给你一个按**非递减顺序**排序的整数数组 nums，返回**每个数字的平方**组成的新数组，要求也按**非递减顺序**排序。

### 暴力解法

每个数平方后再排序

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        for (int i = 0; i < nums.size(); i++) {
            nums[i] = pow(nums[i], 2);
        }
        sort(nums.begin(), nums.end()); // 快排

        return nums;
    }
};
```

复杂度：

- 时间复杂度：\(O(n+n\log(n))\)
- 空间复杂度：\(O(1)\)

### 双指针

对于平方后数组，其实是有规律的。负数平方后就是正数，那么数组平方的最大值就在数组的两端

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        int n, left, right;
        vector<int> result(nums.size(), 0);
        
        n = nums.size() - 1;
        left = 0;
        right = nums.size() - 1;
        
        while(left <= right) { // 采用闭区间[left, right]
            int left_square = pow(nums[left], 2);
            int right_square = pow(nums[right], 2);
            
            if (left_square > right_square)
            {
                result[n] = left_square;
                left++;
            }
            else
            {
                result[n] = right_square;
                right--;
            }
            n--;
        }
        
        return result;
    }
};
```

复杂度：

- 时间复杂度：\(O(n)\)
- 空间复杂度：\(O(n)\)
