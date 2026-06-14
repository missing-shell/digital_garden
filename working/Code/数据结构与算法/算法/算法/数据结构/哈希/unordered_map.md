在算法题中，`std::unordered_map`是一个非常强大的工具，尤其在处理与键值对相关的数据时。以下是使用`std::unordered_map`时常用的一些方法和技巧：

- 底层由哈希表实现

### 1. 插入元素
```c++
// 插入键值对
unordered_map<int, int> myMap;
myMap[key] = value; // 如果key不存在，则插入；如果存在，则更新value

// 或者使用insert方法
myMap.insert({key, value});
```
### 2. 查找元素
```c++
// 检查键是否存在
if (myMap.find(key) != myMap.end()) {
    // 键存在
}

// 直接访问，若键不存在则会插入默认构造的value
int val = myMap[key]; 

// 安全访问，使用at()，如果键不存在会抛出out_of_range异常
int val = myMap.at(key);
```
### 3. 更新元素
```c++
// 直接使用[]操作符更新
myMap[key] = newValue;
```
### 4. 删除元素
```c++
// 删除键值对
myMap.erase(key);
```

### 5. 统计元素
```c++
// 检查键是否存在
bool exists = myMap.count(key) > 0;

// 获取元素数量
size_t size = myMap.size();
```
### 6. 遍历元素
```c++
// 遍历所有键值对
for (const auto& pair : myMap) {
    int key = pair.first;
    int value = pair.second;
}

// 或者使用迭代器
for (auto it = myMap.begin(); it != myMap.end(); ++it) {
    int key = it->first;
    int value = it->second;
}
```
### 7. 清空容器
```c++
// 清空所有元素
myMap.clear();
```
### 8. 初始容量和负载因子
```c++
// 设置初始容量
unordered_map<int, int> myMap(100); // 初始容量为100

// 设置最大负载因子
myMap.max_load_factor(0.5); // 默认是1
```
### 9. 交换容器
```c++
// 与另一个unordered_map交换内容
unordered_map<int, int> anotherMap;
myMap.swap(anotherMap);
```
### 10. 使用`emplace`插入元素
```c++
// 使用放置式构造插入元素，避免不必要的拷贝
myMap.emplace(std::piecewise_construct, std::forward_as_tuple(key), std::forward_as_tuple(value));
```
### 11. 使用`operator[]`与`emplace`结合避免重复插入
```c++
// 避免重复插入，仅当键不存在时插入
auto& ref = myMap.emplace(key, defaultValue).first->second;
```
### 12. 使用`std::make_unique`和`std::unique_ptr`存储动态分配的对象
```c++
// 存储动态分配的对象
unordered_map<int, std::unique_ptr<int>> myMap;
myMap[key] = std::make_unique<int>(value);
```
这些方法涵盖了`std::unordered_map`的大部分常见用途，能够帮助你在算法题中高效地管理和操作数据。