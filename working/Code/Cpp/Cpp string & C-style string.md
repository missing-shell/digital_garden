#string #`\0`  #`'0'`
```cpp
std::string s = "123"; // 初始化字符串s为"123"
s.push_back(0);        // 在字符串s的末尾添加空字符'\0'，但这不会影响s.length()的返回值
s += "456";           // 将字符串"456"连接到s的末尾，现在s变为"123\0456"

auto c = strlen(s.c_str()); // 使用strlen函数获取字符串s的C风格字符串表示的长度，这将返回3，因为strlen在遇到第一个'\0'时停止计数
auto cxx = s.length();     // 使用string类的length成员函数获取字符串s的长度，这将返回7，因为s实际上包含7个字符：'1', '2', '3', '\0', '4', '5', '6'
```

- `std::string.push_back()`是追加字符，`push_back(0)`是指追加`ASCII`字符`0`，即`NULL`或者说空字符，而在通常说的`C`中，宏定义`NULL`就是`0`。
- `'\0`'就是`ascii`为`0`的，`'0'`是`ascii48`。
### 参考链接
- [std::string](https://www.zhihu.com/question/614285401/answer/3229713754) 