### 概述
#栈 #队列
- [71. 简化路径](https://leetcode.cn/problems/simplify-path/)
#### 题目分析
> 根据题意可知返回的**规范路径**为：
- 始终以斜杠`/`开头
- 两个目录名之间使用`/`分隔
- 最后一个目录名后不能以`/`结尾（除仅有`/`目录这种情况
- 最终路径不包含`.`或`..`
#### 解题思路
- 删除掉多余的斜杠`/`，将每一级目录保存到队列或栈中
- 遍历队列，遇到`..`时返回上一级（如果存在），遇到`.`则将其移除
- 生成新`path`字符串，在每级目录前加上`/`

> 删除多余`/`
- 通过`start`和`end`指针找到每一级目录中的内容保存到队列
- `start`为合法的指向第一个不为`/`的位置，`end`为遇到`/`前的第一个位置
> 遇到`..`返回上一级
- 首先通过判断队列中是否为空，如果为空，则忽略`..`
- 回退到上一级
### Code
#### deque
```c++
class Solution {
public:
    string simplifyPath(string path) {
        deque<string> dirs;
        int n = path.size();
        int start = 0;
        int end;
        string dir;
        while (start < n) {
            while (start < n &&
                   path[start] == '/') { // 找到目录开头（第一个不为`/`
                start++;
            }
            if (start >= n) {
                break;
            }
            end = start;
            while (end < n && path[end] != '/') // end指向第一次出现`/`的位置
            {
                end++;
            }
            dir = path.substr(start, end - start); // 等价于(end-1)-start+1
            if (dir == ".." && !dirs.empty()) {
                dirs.pop_back();
            } else if (dir != ".." && dir != ".") {
                dirs.emplace_back(
                    dir); // 如果目录名不为.或..，那么就加入队尾，组成路径
            }
            start = end;
        }
        if (dirs.empty()) // 队列为空，由于路径以'/'开头，因此返回一个'/'
        {
            return "/";
        }
        string new_path;
        while (!dirs.empty()) {
            new_path += "/" + dirs.front();
            dirs.pop_front();
        }
        return new_path;
    }
};
```
#### stack
- 与使用`deque`的区别在于最终字符串的拼接
- `deque`可以从队头弹出，`stack`只能弹出栈顶元素，不过可以利用`string`的插入（每次将新字符串插入到原始字符串开头）
```c++
class Solution {
public:
    string simplifyPath(string path) {
        stack<string> dirs;
        int start = 0;
        int end;
        int n = path.size();
        string dir;
        while (start < n) {
            while (start < n && path[start] == '/') {
                start++;
            }
            if (start >= n) {
                break;
            }
            end = start;
            while (end < n && path[end] != '/') {
                end++;
            }
            dir = path.substr(start, (end - 1) - start + 1);
            if(dir==".."&&!dirs.empty())
            {
                dirs.pop();
            }
            else if(dir!=".."&&dir!=".")
            {
                dirs.push(dir);
            }
            start=end;
        }
        if(dirs.empty())
        {
            return "/";
        }
        string new_path;
        while(!dirs.empty())
        {
            new_path="/"+dirs.top()+new_path; //等价于`new_path.insert(0,"/"+dirs.top());`
            dirs.pop();
        }
        return new_path;
    }
};
```
