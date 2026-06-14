#哈希 #剪枝
### 题目描述
[128. 最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)
### 思路
- 枚举数组中的每一个数$x$，以$x$为起点，不断尝试匹配$x+1$，$x+2$，...后不断更新枚举并更新答案。这种暴力匹配的时间复杂度为$O(n^2)$
#### 哈希表
- 用哈希表存储数组中的数，当查看数组中是否存在该数的时间复杂度优化为$O(1)$，但是最坏情况下仍为$O(n^2)$（即外层枚举$O(n)$，内层暴力匹配$O(n)$次）
- 如果已知有一个 $x,x+1,x+2,⋯,x+y$的连续序列，而我们却重新从 $x+1，x+2$ 或者是 $x+y$ 处开始尝试匹配，那么得到的结果肯定不会优于枚举 $x$为起点的答案，因此我们在外层循环的时候碰到这种情况跳过即可。
- **每次在哈希表中检查是否存在 $x−1$ 即能判断是否需要跳过。**
- 外层循环需要 $O(n)$ 的时间复杂度，只有当一个数是连续序列的第一个数的情况下才会进入内层循环，然后在内层循环中匹配连续序列中的数，因此数组中的每个数只会进入内层循环一次。根据上述分析可知，总时间复杂度为 $O(n)$
- 空间复杂度：$O(n)$  哈希表存储数组中所有的数需要 $O(n)$ 的空间。
```c++
class Solution
{
public:
    int longestConsecutive(vector<int> &nums)
    {
        unordered_set<int> num_set;
        for (const int &num : nums)
        {
            num_set.insert(num);
        }
        int res = 0;
        for (const int &num : num_set)
        {
            if (!num_set.count(num - 1))
            {
                int curNum = num;
                int curLen = 1;
                while (num_set.count(curNum + 1))
                {
                    curNum++;
                    curLen++;
                }
                res = max(res, curLen);
            }
        }
        return res;
    }
};
```
#### 排序+双指针
#双指针 
- 时间复杂度: $O(nlogN)$ 遍历判断是 $O(n)$, 排序是  $O(nlogN)$
- 空间复杂度: $O(1)$ 使用固定的额外空间
- 首先对数组进行排序，连续的数据就会聚集，在遍历中用双指针去重
```c++
class Solution
{
public:
    int longestConsecutive(vector<int> &nums)
    {
        sort(nums.begin(), nums.end());
        int res = 1;
        int cur = 1;
        for (int fast = 1, slow = 0; fast < nums.size(); fast++)
        {
            if (nums[fast] == nums[slow])
            {
                continue;
            }
            else if (nums[slow] + 1 == nums[fast])
            {
                cur++;
                res = max(res, cur);
            }
            else
            {
                cur = 1;
            }
            slow = fast;
        }
        res = min((int)nums.size(), res);
        return res;
    }
};
```
