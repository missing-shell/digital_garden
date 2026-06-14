#哈希 #Leetcode #union-find
### 题目描述
[49. 字母异位词分组](https://leetcode.cn/problems/group-anagrams/)
### 分析
- 当且仅当两个字符串包含的字母相同，两个字符串互为字母异位值
- 寻找每组中的特性：通过==排序==或==计数==得到唯一匹配的字符串，将这个字符串作为对应的键值
- 一个键值对应一组字母异位词
### code
#### 排序
```c++
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> map;
        for (auto& str : strs) {
            string key = str;
            sort(key.begin(), key.end());
            map[key].push_back(str);
        }
        vector<vector<string>> res;
        for (auto& p : map) {
            res.push_back(p.second);
        }
        return res;
    }
};
```
#### 计数
- 由于题目提到仅包含小写字母，所以可以使用数组实现

---
- C++标准库中的`std::unordered_map`要求键类型是可哈希的，并且具有有效的不等运算符。`int tmp[]`数组类型和不符合这些要求，因为它们既没有默认的哈希函数，也没有默认的不等比较操作。
- `std::vector`不是`std::unordered_map`的一个好键类型，因为`std::vector`的默认比较器是基于元素的逐个比较，这会导致非哈希行为，从而使得`unordered_map`无法正常工作。此外，`std::vector`没有重载`std::hash`模板特化，因此不能直接用作哈希容器的键。
---
- `std::array<int,26> count={}`
	std::array<int, 26> count = {0}; 这行代码的作用是声明并初始化一个固定大小的整数数组 count，其大小为26，用于存储字母表中每个字母的计数。这里使用的是C++11引入的std::array容器，它是一个固定大小的序列容器，提供了比传统C风格数组更安全和更方便的操作接口。
---

```c++
class Solution {
public:
    std::vector<std::vector<std::string>> groupAnagrams(std::vector<std::string>& strs) {
        // 定义一个哈希函数，用于处理 array<int, 26> 类型的键
        auto arrayHash = [](const std::array<int, 26>& arr) -> size_t {
            // 使用一个lambda表达式作为哈希函数
            // accumulate 函数将 arr 中的元素转换成一个哈希值
            // 每个元素的哈希值通过 XOR 和左移运算组合起来
            return std::accumulate(arr.begin(), arr.end(), 0u,
                                   [](size_t acc, int num) {
                                       static std::hash<int> fn;
                                       return (acc << 1) ^ fn(num);
                                   });
        };

        // 创建一个 unordered_map，使用自定义的哈希函数
        // 第二个模板参数是哈希函数的类型
        std::unordered_map<std::array<int, 26>, std::vector<std::string>, decltype(arrayHash)> mp(0, arrayHash);

        // 遍历输入的字符串向量
        for (std::string& str : strs) {
            // 初始化一个 array<int, 26> 类型的计数器
            std::array<int, 26> counts{};

            // 遍历当前字符串，统计每个字符的出现次数
            for (char ch : str) {
                counts[ch - 'a']++;
            }

            // 将当前字符串添加到对应的分组中
            mp[counts].emplace_back(str);
        }

        // 创建一个结果向量，用于存储分组后的字符串
        std::vector<std::vector<std::string>> ans;

        // 遍历 unordered_map，将每个分组的字符串添加到结果向量中
        for (const auto& item : mp) {
            ans.emplace_back(item.second);
        }

        // 返回结果向量
        return ans;
    }
};
```
