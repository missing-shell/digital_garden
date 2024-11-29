typedef是一种有趣的声明形式：它为一种类型引入新的名字，而不是为变量分配空间。在某些方面，typedef*类似于宏文本替换*——它并没有引入新类型，而是为现有类型取个新名字，但它们之间存在一个关键性的区别，容我稍后解释。

### 语法

对于现有类型：`typedef 现有类型 新别名;`

对于函数指针：`typedef void (*新别名)(int);`

对于结构体：

```c
//在定义结构体的同时创建别名
typedef struct StructName {
    // 成员定义
} TypedefName;

//先定义结构体，然后创建别名
struct StructName {
    // 成员定义
};

typedef struct StructName TypedefName;
```

### 与`#define`区别
- **typedef** 仅限于为类型定义符号名称，**#define**不仅可以为类型定义别名，也能为*数值*定义别名，比如您可以定义 1 为 ONE。
- **typedef** 是由**编译器执行解释**的，**#define** 语句是由预编译器进行处理的，在调试时不可见。类似于
- 在连续几个变量的声明中，用typedef定义的类型能够保证声明中所有的变量*均为同一种类型*，而用#define定义的类型则无法保证。

`# define  int_ptr  int`

`*int_ptr  chalk, cheese;`经过宏扩展，第二行变为：`int* chalk, cheese;`这使得chalk和cheese成为不同的类型，就好象是辣椒酱与细香葱的区别：`chalk是一个指向int的指针`，==而cheese则是一个int==
