#区间合并 #双指针
### 题目描述
- 链接：[57. 插入区间](https://leetcode.cn/problems/insert-interval/)
### 分析
- 首先合并两个集合，现在就变成了[合并区间](https://leetcode.cn/problems/merge-intervals/)
- 根据区间开始时间对`intervals`进行排序，然后遍历每个区间，如果当前区间与下一个区间重叠，则更新当前区间的结束时间；如果不重叠，则将当前区间添加到结果向量`res`中，并将下一个区间设为新的当前区间。
### code
#### 合并集合正确方式
- 将`newInterval`作为一个整体添加到`intervals`向量的末尾，而不是逐个元素添加。
```c++
intervals.push_back(newInterval);
```

#### 完整代码
```c++
vector<vector<int>> insert(vector<vector<int>> &intervals, vector<int> &newInterval)
    {
        // Correctly add the newInterval as a whole instead of individual elements.
        intervals.push_back(newInterval);

        // Sort the entire list of intervals based on the start time.
        sort(intervals.begin(), intervals.end());

        vector<vector<int>> res; // Result vector to hold the merged intervals.

        // Initialize the current interval with the first interval in the sorted list.
        vector<int> cur = intervals[0];

        for (size_t i = 1; i < intervals.size(); i++)
        { 
	        // Use size_t for index to avoid signed/unsigned mismatch.
            if (cur[1] >= intervals[i][0])
            {                                          // Check for overlap.
                cur[1] = max(cur[1], intervals[i][1]); // Merge intervals by updating the end time.
            }
            else
            {
                res.push_back(cur); // No overlap, add the current interval to the result.
                cur = intervals[i]; // Set the current interval to the next one.
            }
        }

        // Add the last interval to the result after the loop.
        res.push_back(cur);

        return res;
    }
```