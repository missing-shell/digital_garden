Java源文件的扩展名是 .java，编译后的字节码文件扩展名是 .class。

表达式 s2 == s1 比较的是两个字符串对象的引用，由于字符串是不可变的，且 s1 和 s2 包含的字面量相同，Java会在**常量池**中**共享**相同的字符串对象，所以结果也是 true。

java.lang 是默认引用的包。

class 是Java的保留关键字，不能用作标识符。

在Java中，发生继承时，子类通过重写（Override）父类的方法实现多态现象，这称为方法重写或简称重写，而不是重载（Overload）。

#### 多态

方法重载是指在同一个类中存在多个同名的方法，但它们的参数列表（参数的类型和/或数量）不同。这是编译时就已经确定的，编译器根据方法签名（方法名和参数列表）来决定调用哪个重载的方法。方法重载是静态多态的一种形式，因为它在编译时就已经决定，但它本身并不等同于静态多态。

静态多态是指在程序编译时就能够确定的多态性。在Java中，静态多态主要通过方法重载和运算符重载实现。由于Java不支持运算符重载，所以静态多态主要是指方法重载。

然而，静态多态这个术语在Java中通常不常使用，更常用的是多态（Polymorphism）这个术语，它涵盖了静态多态和动态多态（也称为运行时多态或动态绑定）。

动态多态（Dynamic Polymorphism）是指在程序运行时，通过虚函数调用（Java中的方法重写）来实现的多态性。在Java中，这是通过方法重写和接口实现的。子类重写父类的方法后，通过父类的引用调用该方法时，实际执行的是子类的实现。这是在运行时绑定的，因为程序需要在执行时确定对象的实际类型。

1. **继承** 是面向对象编程中的一个核心概念，它允许*一个类（称为子类）继承另一个类（称为父类）的属性和方法*。继承的目的是实现代码复用。父类是被继承的类，子类是继承父类的类。继承带来的好处包括**代码复用、层次结构的建立、可以扩展现有类的功能、维护更容易等。**

2. **类的构造方法** 是一种特殊的方法，用于在创建对象时初始化对象的状态。它的特点包括：
   - 与类名相同。
   - 没有返回类型，包括void。
   - 可以有多个构造方法，它们通过参数列表不同来区分。
   - 当创建对象时自动**调用**。

3. **重载** 是指在==同一个类==中可以定义多个同名方法，只要它们的参数列表不同（参数数量或类型不同）。**覆盖**（Override）是*子类重写*父类中具有相同名称和参数列表的方法。**多态** 是指允许不同类的对象对同一消息做出响应，但具体的行为会根据对象的实际类型来确定。重载、覆盖和多态的异同：
   - 重载是编译时多态，覆盖是运行时多态。
   - 重载关注方法的参数，覆盖关注方法的实现。

4. **接口** 是一种==引用==类型，可以包含抽象方法和默认方法，但不能包含实现。接口的特点包括：
   - 可以被任何类实现（implements）。
   - 接口中的方法**默认是public和abstract的**。
   - 接口可以包含常量，不能有变量。
   - 接口可以多实现。接口的作用包括定义规范、实现多继承等。

5. **异常** 是程序运行时发生的错误。抛出异常使用`throw`关键字，捕获异常使用`try-catch`语句，~~处理异常使用`throws`~~。异常处理机制允许程序在发生错误时继续运行，而不是直接崩溃。

6. **类的成员访问修饰符** 包括：
   - `public`：可以被任何其他类访问。
   - `protected`：可以被同一个包中的类和所有子类访问。
   - `private`：只能在定义它的类内部访问。
   - `default`（无修饰符）：可以被同一个包中的类访问。

7. **this** 关键字用于引用当前对象的属性和方法。**super** 关键字用于引用父类中的属性和方法，特别是在子类中重写父类方法时。

8. **实例变量** 是定义在类中，但在方法之外的变量，属于对象的一部分。**局部变量** 是定义在方法内部的变量，只在该方法的作用域内有效。~~static~~

9. **构造方法的执行顺序** 通常是先执行**父类**的构造方法，然后执行子类的构造方法，接着是初始化块，最后是构造方法的主体。

10. **接口** 的作用包括定义一个类必须实现的==规范==，允许类实现多个接口，接口隔离，实现多继承的效果，以及在设计时提供灵活性。
## 4.
### 静态
### 类 
```java
//年终奖合并计算所得税
	 double taxWithoutBonus(double income,int specialLimit) {
	  double tax=0.0;
	  double incomeInTax=income-12*(specialLimit+5000);
	  if(incomeInTax<=36000) {
	   tax=incomeInTax*0.03;
	  }else if(incomeInTax<=144000) {
	   tax=incomeInTax*0.1-2520;
	  }else if(incomeInTax<=300000) {
	   tax=incomeInTax*0.2-16920;
	  }else tax=incomeInTax*0.45-181920;
	  return tax;
	 }
	 //年终奖单独计算所得税
	 double taxBonus(double bonus) {
	  double tax=0.0; 
	  double bonusInTax=bonus/12;
	  if(bonusInTax<=3000) {
	   tax=bonus*0.03;
	  }else if(bonusInTax<=12000) {
	   tax=bonus*0.1-210;
	  }else if(bonusInTax<=25000) {
	   tax=bonus*0.2-1410;
	  }else tax=bonus*0.45-15160;
	  return tax;
	 }
	 //总所得税
	 void computTax(double income,int specialLimit,double bonus) {
	  double taxWithBonus=taxWithoutBonus(income-bonus,specialLimit)
	    +taxBonus(bonus);
	  double taxWithoutBonus=taxWithoutBonus(income,specialLimit);
	  if(taxWithBonus<taxWithoutBonus) {
	   System.out.println("采用奖金单独计算纳税更划算，纳税额为"+taxWithBonus);
	  }else System.out.println("采用奖金合并计算纳税更划算，纳税额为"+taxWithoutBonus);
	 }
```
