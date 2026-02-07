---
title: Day 02 - 数组 Part 02
date: 2026-02-04T16:46:00+08:00
draft: false
description: 数组进阶：滑动窗口求最小子数组、螺旋矩阵模拟、区间和与前缀和优化，以及二维前缀和解决土地划分最小差问题（含 C++ 代码与复杂度）。
summary: 本篇涵盖数组相关的几类经典技巧：用双指针/滑动窗口解决「长度最小的子数组」，通过循环不变式模拟生成「螺旋矩阵 II」，并从区间和暴力查询引出前缀和（含一维/二维前缀和）。最后用二维前缀和计算不同切分方式下的区域和，求开发商购买土地问题的最小差。
tags:
  - 数组
  - 滑动窗口
  - 双指针
  - 前缀和
  - cpp
categories:
  - 数据结构
  - 算法
  - 刷题
series:
  - 代码随想录
series_order: 2
lastmod: 2026-02-07T12:07:15+08:00
---

{{< katex >}}

## [长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)

给定一个含有 n 个正整数的数组和一个正整数 s ，找出该数组中满足其和 ≥ s 的长度最小的 连续 子数组，并返回其长度。如果不存在符合条件的子数组，返回 0。

示例：

- 输入：s = 7, nums = [2,3,1,2,4,3]
- 输出：2
- 解释：子数组 [4,3] 是该条件下的长度最小的子数组。

提示：

- 1 <= target <= 10
- 1 <= nums.length <= 10
- 1 <= nums[i] <= 10

### 暴力解法

依旧是 for 循环暴力求解
> [!WARNING]
> 暴力解法已经超出时间限制

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int min_length = INT_MAX;

        for (int i = 0; i < nums.size(); i++)
        {
            int sum = 0, length = 0;

            while (sum < target && i + length < nums.size())
            {
                sum += nums[i + length];
                length++;
            }

            if (sum >= target && length < min_length)
            {
                min_length = length;
            }
        }

        return min_length == INT_MAX ? 0 : min_length;
    }
};
```

复杂度：

- 时间复杂度：\(O(n^2)\)
- 空间复杂度：\(O(1)\)

### 双指针法

利用双指针构建滑动窗口，动态选取子系列的范围，从而得到结果

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int left = 0, right = 0;
        int sum = 0, minSubLength = INT_MAX;

        for (right = 0; right < nums.size(); right++)
        {
            sum += nums[right];

            while (sum >= target)
            {
                int subLength = right - left + 1; // 计算子序列长度
                minSubLength = (subLength < minSubLength) ? subLength : minSubLength;
                sum -= nums[left++]; // 变更滑动窗口起始位置
            }
        }

        return minSubLength == INT_MAX ? 0 : minSubLength;
        
    }
};
```

复杂度：

- 时间复杂度：\(O(n)\)
- 空间复杂度：\(O(1)\)

## [螺旋矩阵 II](https://leetcode.cn/problems/spiral-matrix-ii/)

给定一个正整数 n，生成一个包含 1 到 n^2 所有元素，且元素按顺时针顺序螺旋排列的正方形矩阵。

示例:

- 输入：3
- 输出：

```
[ 
    [ 1, 2, 3 ], 
    [ 8, 9, 4 ], 
    [ 7, 6, 5 ] 
]
```

本题没有什么算法，就是模拟过程，考验对于代码的掌握能力。依照循环不变的逻辑，由外向内根据”右 -> 下 -> 左 -> 上“的规则构建螺旋矩阵：

- 从左到右构建上面的边
- 从上到下构建右边的边
- 从右到左构建下面的边
- 从下到上构建左面的边
![5x5 Spiral Matrix](attachments/5x5_Spiral_Matrix.png)

```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> matrix(n, vector<int>(n, 0)); // 螺旋矩阵
        int loop = n / 2;                                 // 螺旋矩阵层数
        int side = n;                                     // 每层的边长
        int i = 0, j = 0;                                 // 循环的起始位置
        int count = 1;                                    // 起始值，1～n^2

        while (loop--)
        {
            // 从左到右构建上边
            for (size_t offset = 0; offset < side - 1; offset++)
            {
                matrix[i][j + offset] = count++;
            }
            j += side - 1;
            // 从上到下构建右边
            for (size_t offset = 0; offset < side - 1; offset++)
            {
                matrix[i + offset][j] = count++;
            }
            i += side - 1;
            // 从右到左构建下边
            for (size_t offset = 0; offset < side - 1; offset++)
            {
                matrix[i][j - offset] = count++;
            }
            j -= side - 1;
            // 从下到上构建左边
            for (size_t offset = 0; offset < side - 1; offset++)
            {
                matrix[i - offset][j] = count++;
            }
            i -= side - 1;

            // 更新下一层循环的起始位置和边长
            i++;
            j++;
            side -= 2;
        }

        if (n % 2) // 当 n 是奇数时，要单独对中心的位置赋值
        {
            matrix[n / 2][n / 2] = count;
        }

        return matrix;
    }
};
```

## 区间和

给定一个整数数组 Array，请计算该数组在每个指定区间内元素的总和。

### 暴力解法

直接根据给定的区间，把每个区间的数累加一遍。这种解法一旦进行大量的数据查询就会耗费大量的时间

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
    int n, a, b;

    cin >> n;
    vector<int> vec(n); // 数组

    for (int i = 0; i < n; i++)
        cin >> vec[i];

    while (cin >> a >> b)
    {
        int sum = 0;

        // 累加区间 a 到 b 的和
        for (int i = a; i <= b; i++)
            sum += vec[i];

        cout << sum << endl;
    }
}
```

复杂度：

- 时间复杂度：\(O(n+nm)\)
- 空间复杂度：\(O(1)\)

## [前缀和](https://kamacoder.com/problempage.php?pid=1070)

在多次查询中，可以发现我们计算了许多重复的子数组，如果能重复利用计算过的子数组之和，就能大大降低计算中查询区间的次数。

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main()
{
    int n, a, b;
    if (!(cin >> n))
        return 0;

    // 前缀和: preVec[i] = sum of vec[0..i-1]
    vector<int> preVec(n + 1, 0);

    for (int i = 0; i < n; i++)
    {
        int x;
        cin >> x;
        preVec[i + 1] = preVec[i] + x; // 计算前缀和
    }

    while (cin >> a >> b)
    {
        int sum = preVec[b + 1] - preVec[a];
        cout << sum << endl;
    }

    return 0;
}
```

复杂度：

- 时间复杂度：\(O(n+m)\)
- 空间复杂度：\(O(1)\)

## [开发商购买土地](https://kamacoder.com/problempage.php?pid=1044)

在一个城市区域内，被划分成了 n * m 个连续的区块，每个区块都拥有不同的权值，代表着其土地价值。目前，有两家开发公司，A 公司和 B 公司，希望购买这个城市区域的土地。
现在，需要将这个城市区域的所有区块分配给 A 公司和 B 公司。
然而，由于城市规划的限制，只允许将区域按横向或纵向划分成两个子区域，而且每个子区域都必须包含一个或多个区块。 为了确保公平竞争，你需要找到一种分配方式，使得 A 公司和 B 公司各自的子区域内的土地总价值之差最小。
注意：区块不可再分。

示例：

- 输入：（第一行输入两个正整数，代表 n 和 m。 接下来的 n 行，每行输出 m 个正整数。）

```
3 3
1 2 3
2 1 3
1 2 3
```

- 输出：0（输出一个整数，代表两个子区域内土地总价值之间的最小差距。）

提示：
如果将区域按照如下方式划分：

```
1 2 | 3
2 1 | 3
1 2 | 3 
```

两个子区域内土地总价值之间的最小差距可以达到 0。

### 思路

依旧采用前缀和的思路，在构建过程中计算出前缀区间和，再利用前缀和求分割区间

```cpp
#include <iostream>
#include <vector>
#include <climits>

using namespace std;

int main()
{
    int n, m;
    int minDiff = INT_MAX;
    cin >> n >> m;

    vector<vector<int>> preVec(n + 1, vector<int>(m + 1, 0)); // preVec[i][j] = sum of vec[0...i-1][0...j-1]
    for (size_t i = 0; i < n; i++)
    {
        for (size_t j = 0; j < m; j++)
        {
            int num;
            cin >> num;

            // 构建前缀和
            preVec[i + 1][j + 1] = preVec[i + 1][j] + preVec[i][j + 1] - preVec[i][j] + num;
        }
    }

    // 横向分割
    for (size_t i = 0; i < n; i++)
    {
        int diff = abs(preVec[n][m] - 2 * preVec[i + 1][m]);
        minDiff = diff < minDiff ? diff : minDiff;
    }

    // 纵向分割
    for (size_t i = 0; i < m; i++)
    {
        int diff = abs(preVec[n][m] - 2 * preVec[n][i + 1]);
        minDiff = diff < minDiff ? diff : minDiff;
    }

    cout << minDiff << endl;
    return 0;
}
```

复杂度：

- 时间复杂度：\(O(nm)\)
- 空间复杂度：\(O(1)\)
