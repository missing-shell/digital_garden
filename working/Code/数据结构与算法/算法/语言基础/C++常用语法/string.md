## substr
接受两个参数：开始索引和要提取的字符数。注意，如果你的意图是包含结束位置`end`对应的字符，那么 `end + 1 - first` 的用法是正确的；如果`end`应当是截止位置（不包含该位置的字符），则应直接使用 `end - first`。
```c++
res += s.substr(first, end + 1 - first);
```

