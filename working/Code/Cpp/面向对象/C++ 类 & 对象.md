### 概念
- **类**用于指定对象的形式，是一种用户自定义的数据类型，它是一种封装了数据和函数的组合。类中的数据称为成员变量，函数称为成员函数。类可以被看作是一种模板，可以用来创建具有相同属性和行为的多个对象。
### C++类定义

定义一个类需要使用关键字 `class`，然后指定类的名称，并类的主体是包含在一对花括号中，主体包含类的成员变量和成员函数。

- 定义一个类，本质上是定义一个数据类型的蓝图，它定义了类的对象包括了什么，以及可以在这个对象上执行哪些操作。
```c++
class classname  /*关键词：类名*/
{
	Access specifiers:  /*访问修饰符*/
		Date members/variables;  /*变量*/
		Member functions(){};    /*方法*/
};  /*使用分号结束一个类*/
```
### 访问
#### public
- 在类对象作用域内，公共成员在类的外部是可访问的，类的对象的公共数据成员可以使用直接成员访问运算符 . 来访问。
#### private/protect
- 私有的成员和受保护的成员不能使用直接成员访问运算符 (.) 来直接访问
### 类 & 对象详解

|概念|描述|
|---|---|
|[类成员函数](https://www.runoob.com/cplusplus/cpp-class-member-functions.html "C++ 类成员函数")|类的成员函数是指那些把定义和原型写在类定义内部的函数，就像类定义中的其他变量一样。|
|[类访问修饰符](https://www.runoob.com/cplusplus/cpp-class-access-modifiers.html "C++ 类访问修饰符")|类成员可以被定义为 public、private 或 protected。默认情况下是定义为 private。|
|[构造函数 & 析构函数](https://www.runoob.com/cplusplus/cpp-constructor-destructor.html "C++ 构造函数 & 析构函数")|类的构造函数是一种特殊的函数，在创建一个新的对象时调用。类的析构函数也是一种特殊的函数，在删除所创建的对象时调用。|
|[C++ 拷贝构造函数](https://www.runoob.com/cplusplus/cpp-copy-constructor.html "C++ 拷贝构造函数")|拷贝构造函数，是一种特殊的构造函数，它在创建对象时，是使用同一类中之前创建的对象来初始化新创建的对象。|
|[C++ 友元函数](https://www.runoob.com/cplusplus/cpp-friend-functions.html "C++ 友元函数")|**友元函数**可以访问类的 private 和 protected 成员。|
|[C++ 内联函数](https://www.runoob.com/cplusplus/cpp-inline-functions.html "C++ 内联函数")|通过内联函数，编译器试图在调用函数的地方扩展函数体中的代码。|
|[C++ 中的 this 指针](https://www.runoob.com/cplusplus/cpp-this-pointer.html "C++ 中的 this 指针")|每个对象都有一个特殊的指针 **this**，它指向对象本身。|
|[C++ 中指向类的指针](https://www.runoob.com/cplusplus/cpp-pointer-to-class.html "C++ 中指向类的指针")|指向类的指针方式如同指向结构的指针。实际上，类可以看成是一个带有函数的结构。|
|[C++ 类的静态成员](https://www.runoob.com/cplusplus/cpp-static-members.html "C++ 类的静态成员")|类的数据成员和函数成员都可以被声明为静态的。|
