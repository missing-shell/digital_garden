## 标签
#双指针  #哈希表 

## 双指针解法
- 重点思路应该是`排序`，`双指针`，`去重`
- 对于每一个指针所指向的数都应该`去重`

```c++
vector<vector<int>> threeSum(vector<int> &nums)
    {
        int len = nums.size();
        sort(nums.begin(), nums.end());
        vector<vector<int>> res;
        for (int k = 0; k < len - 2; k++)
        {
            if (nums[k] > 0)
            {
                break;
            }
            if (k > 0 && nums[k] == nums[k - 1])
            {
                continue;//去重
            }
            int i = k + 1, j = len - 1;
            while (i < j)
            {
                int sum = nums[i] + nums[j] + nums[k];
                if (sum < 0)
                {
                    i++;
                    while (i < j && nums[i] == nums[i - 1])
                    {
                        i++; // 去重}
                    }
                }
                else if (sum > 0)
                {
                    j--;
                    while (i < j && nums[j] == nums[j + 1])
                        j--; // 去重
                }
                else
                {
                    res.push_back({nums[k], nums[i], nums[j]});
                    i++;
                    j--;
                    while (i < j && nums[i] == nums[i - 1])
                        i++; // 去重
                    while (i < j && nums[j] == nums[j + 1])
                        j--; // 去重
                }
            }
        }
        return res;
    }
```

