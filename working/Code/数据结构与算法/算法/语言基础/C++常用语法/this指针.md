C++中的`this`指针是隐式传递给每个非静态成员函数的第一个参数。`this`指针是一个指向调用成员函数的对象的指针，它允许成员函数访问对象的成员变量和调用其他成员函数。虽然程序员在编写代码时不需要显式声明`this`作为参数，但编译器会在生成机器代码时自动将其作为第一个参数传递。

例如，考虑以下类的成员函数：
cpp

```c++
class MyClass {
public:
    int myVar;
    void myFunction() {
        // 在这里，'this'指针指向调用myFunction的对象
        // 可以通过'->'操作符访问成员变量
        std::cout << this->myVar << std::endl;
    }
};
```

在上面的例子中，即使代码中没有显式声明`this`，在`myFunction`内部，`this`仍然可用，它是一个指向`MyClass`实例的指针，可以用来访问`myVar`成员。当调用`myFunction`时，编译器会自动将对象的地址作为`this`传递给函数。