### 题目描述
- #字符串 #双指针 
- 链接：[151. 反转字符串中的单词](https://leetcode.cn/problems/reverse-words-in-a-string/)
####  法一：分割单词，直接拼接
- 从原始字符串的尾部开始遍历
- 首先跳过字符串末尾的空格
- 使用`fisrt`和`end`双指针分别来标识一个单词的开始和结尾
- `s[i] != ' ' && s[i + 1] == ' '`用于找到`end`指针，并在`i<index`的情况下（不是结果字符串的开头）添加空格`res+=" "`
- `s[i] == ' ' && s[i + 1] != ' '`用于找到`first`指针，`first=i+1`

- *时间复杂度*：$O(n)$ 
- *空间复杂度*：$O(n)$ --额外的结果字符串`string res=""`
#### 法二：首先反转整个字符串，再一次反转每个单词
*思路*：
- 移除多余空格
- 将整个字符串反转
- 将每个单词反转

> 移除多余空格
- 使用双指针移除空格，并使用`resize`重新设置字符串的大小，*时间复杂度*：$O(n)$ 
- 本质同[27. 移除元素](https://leetcode.cn/problems/remove-element/)

> 反转字符串
- 同[344. 反转字符串](https://leetcode.cn/problems/reverse-string/)和[541. 反转字符串 II](https://leetcode.cn/problems/reverse-string-ii/)

> 将每个单词反转
- 使用双指针找到每个单词的开头和结束位置，其余同*反转字符串*

---
- **优势**：不使用辅助空间
- *空间复杂度*：$O(1)$
- *时间复杂度*：$O(n)$ 
### code
#### 直接分割
```c++
class Solution
{
public:
    /**
     * @brief 反转字符串中的每个单词。
     *
     * @param s 输入字符串，包含小写字母和空格。
     * @return std::string 返回反转每个单词后的字符串。
     *
     * 该函数接收一个字符串 s，并返回一个新的字符串，其中每个单词内部的字符顺序被反转，
     * 但单词之间的顺序和空格保持不变。
     */
    string reverseWords(string s)
    {
        string res = ""; // 初始化一个空字符串来存储结果

        // 跳过字符串末尾的空格
        int index = s.size() - 1;
        while (s[index] == ' ')
        {
            index--;
        }

        // 从字符串末尾开始遍历
        for (int i = index, first = i, end = i; i >= 0; i--)
        {
            // 如果当前字符不是空格且下一个字符是空格，则记录该位置为单词的结束位置
            // 并在结果字符串 res 中添加一个空格
            // i<index用于确保在新字符串的开头不会出现空格
            if (i < index && s[i] != ' ' && s[i + 1] == ' ')
            {
                res += " ";
                end = i;
            }

            // 如果当前字符是空格而下一个字符不是空格，则记录该位置为单词的起始位置
            // 并将这个单词添加到结果字符串 res 的前面
            if (i < index && s[i] == ' ' && s[i + 1] != ' ')
            {
                first = i + 1;
                res += s.substr(first, end + 1 - first);
            }

            // 如果到达字符串的开头且第一个字符不是空格，则记录该位置为单词的起始位置
            // 并将这个单词添加到结果字符串 res 的前面
            if (i == 0 && s[i] != ' ')
            {
                first = 0;
                res += s.substr(first, end + 1 - first);
            }
        }

        return res; // 返回反转每个单词后的字符串
    }
};
```
#### 首先反转整个字符串
```c++
class Solution
{
public:
    /**
     * @brief 反转字符串中的指定区间 [start, end] 内的字符。
     *
     * @param s 字符串引用。
     * @param start 区间的起始位置。
     * @param end 区间的结束位置。
     */
    void reverse(string &s, int start, int end)
    {
        while (start < end)
        {
            swap(s[start++], s[end--]);
        }
    }

    /**
     * @brief 移除字符串中的多余空格。
     *
     * @param s 字符串引用。
     */
    void removeExtraSpace(string &s)
    {
        int slow = 0;
        for (int fast = 0; fast < s.size(); fast++)
        {
            if (s[fast] != ' ') // 等价于先移除所有空格
            {
                if (slow != 0)
                {
                    s[slow++] = ' '; // 在单词之间添加空格
                }
                // 双指针移动，更新单词的下标
                // 本质同[27. 移除元素](https://leetcode.cn/problems/remove-element/)
                while (fast < s.size() && s[fast] != ' ')
                {
                    s[slow++] = s[fast++];
                }
            }
        }
        s.resize(slow);
    }

    /**
     * @brief 反转字符串中的每个单词。
     *
     * @param s 输入字符串，包含小写字母和空格。
     * @return std::string 返回反转每个单词后的字符串。
     *
     * 该函数接收一个字符串 s，并返回一个新的字符串，其中每个单词内部的字符顺序被反转，
     * 但单词之间的顺序和空格保持不变。
     */
    string reverseWords(string s)
    {
        removeExtraSpace(s);         // 移除多余的空格
        reverse(s, 0, s.size() - 1); // 反转整个字符串

        int start = 0;
        for (int end = 0; end <= s.size(); ++end)
        {
            if (end == s.size() || s[end] == ' ') // 边界问题-源字符串末尾可能没有多余空格不能遗漏
            {
                reverse(s, start, end - 1); // 反转每个单词
                start = end + 1;
            }
        }

        return s; // 返回反转每个单词后的字符串
    }
};
```