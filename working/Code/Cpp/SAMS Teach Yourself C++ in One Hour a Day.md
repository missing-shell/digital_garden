## Sams Teach Yourself C++ in One Hour a Day

## Table of Contents

## Introduction

### Who Should Read This Book

### Conventions Used in This Book

### Sample Code for This Book

## PART I: The Basics

### LESSON 1: Getting Started

#### A Brief History of C++

#### How C++ Evolved

#### Should I Learn C First?

#### Microsoft's Managed Extensions to C++

#### Preparing to Program

#### Your Development Environment

#### The Process of Creating the Program

#### The Development Cycle

#### HELLO.cpp—Your First C++ Program

#### Getting Started with Your Compiler

#### Compile Errors

#### Summary

#### Q&A

#### Workshop

### LESSON 2: The Anatomy of a C++ Program

#### A Simple Program

#### A Brief Look at cout

#### Using the Standard Namespace

#### Commenting Your Programs

#### Functions

#### Summary

#### Q&A

#### Workshop

### LESSON 3: Using Variables, Declaring Constants

#### What Is a Variable?

#### Defining a Variable

#### Determining Memory Consumed by a Variable Type

#### Creating More Than One Variable at a Time

#### Assigning Values to Your Variables

#### Creating Aliases with typedef

#### When to Use short and When to Use long

#### Working with Characters

#### Constants

#### Enumerated Constants

#### Summary

#### Q&A

#### Workshop

### LESSON 4: Managing Arrays and Strings

#### What Is an Array?

#### Multidimensional Arrays

#### char Arrays and Strings

#### Using the strcpy() and strncpy() Methods

#### String Classes

#### Summary

#### Q&A

#### Workshop

### LESSON 5: Working with Expressions, Statements, and Operators

#### Starting with Statements

#### Expressions

#### Working with Operators

#### Combining the Assignment and Mathematical Operators

#### Incrementing and Decrementing

#### Understanding Operator Precedence

#### Nesting Parentheses

#### The Nature of Truth

#### The if Statement

#### Using Braces in Nested if Statements

#### Using the Logical Operators

#### Short Circuit Evaluation

#### Relational Precedence

#### More About Truth and Falsehood

#### The Conditional (Ternary) Operator

#### Summary

#### Q&A

#### Workshop

### LESSON 6: Organizing Code with Functions

#### What Is a Function?

#### Return Values, Parameters, and Arguments

#### Declaring and Defining Functions

#### Execution of Functions

#### Determining Variable Scope

#### Parameters Are Local Variables

#### Considerations for Creating Function Statements

#### More About Function Arguments

#### More About Return Values

#### Default Parameters

#### Overloading Functions

#### Special Topics About Functions

#### How Functions Work—A Peek Under the Hood

#### Summary

#### Q&A

#### Workshop

### LESSON 7: Controlling Program Flow

#### Programming Loops

#### Using while Loops

#### Implementing do...while Loops

#### Using do...while

#### Looping with the for Statement

#### Summing Up Loops

#### Controlling Flow with switch Statements

#### Summary

#### Q&A

#### Workshop

### LESSON 8: Pointers Explained

#### What Is a Pointer?
> A pointer is a variable that holds a `memory address`.

- A pointer that is not initialized is called a `wild pointer` because you have no idea what it is pointing to—and it could be pointing to anything!
#### The Stack and the Free Store (Heap)

#### Another Look at Memory Leaks
- Another way you might inadvertently create a memory leak is by reassigning your pointer `before` deleting the memory to which it points.
#### Creating Objects on the Free Store

#### Deleting Objects from the Free Store

#### Stray, Wild, or Dangling Pointers

#### Using const Pointers

#### Summary

#### Q&A

#### Workshop

### LESSON 9: Exploiting References
- [ ] What references are
- [ ] How references differ from pointers
- [ ] How to create references and use them
- [ ] What the limitations of references are
- [ ] How to pass values and objects into and out of functions by reference
#### What Is a Reference?
A reference is an alias;
References can have any legal variable name, but some programmers prefer to prefix reference names with the letter r.
#### Using the Address-Of Operator (&) on References
C++ gives you no way to access the address of the reference itself because it is not meaningful as it would be if you were using a pointer or other variable.
References are initialized when created, and they always act as a synonym for their target, even when the address-of operator is applied.

> Normally, when you use a reference, you do not use the address-of operator. You simply use the reference as you would use the target variable.

#### Null Pointers and Null References
the ability to reference specific addresses is valuable and required. For this reason, most compilers support a null or numeric initialization of a reference without much complaint, crashing only if you try to use the object in some way when that reference would be invalid.
#### Passing Function Arguments by Reference

#### Returning Multiple Values

#### Passing by Reference for Efficiency
References themselves can never be reassigned to refer to another object, and so they are always constant.If the keyword const is applied to a reference, it is to make constant the object referred to.
#### Knowing When to Use References Versus Pointers
References cannot be reassigned, however. If you need to point first to one object and then to another, you must use a pointer. References cannot be null, so if any chance exists that the object in question might be null, you must not use a reference. You must use a pointer.

```c++
int *pInt = new int; 
if (pInt != NULL) {
	int &rInt = *pInt;
}
```
a pointer to int, pInt, is declared and initialized with the memory returned by the operator new. The address in pInt is tested, and if it is not null, pInt is dereferenced. The result of dereferencing an int variable is an int object, and rInt is initialized to refer to that object. Thus, rInt becomes an alias to the int returned by the operator new.

DON’T try to reassign a reference to a different variable.
#### Mixing References and Pointers

#### Returning Out-of-Scope Object References
DON’T pass by reference if the item referred to might go out of scope.
DON’T lose track of when and where memory is allocated so that you can be certain it is also freed.
#### Summary

#### Q&A
Q Why have references if pointers can do everything references can?
A References are easier to use and to understand. The indirection is hidden, and no need exists to repeatedly dereference the variable.
Q Why have pointers if references are easier?
A References cannot be null, and they cannot be reassigned. Pointers offer greater flexibility but are slightly more difficult to use.
Q Why would you ever return by value from a function?
A If the object being returned is local, you must return by value or you will be returning a reference to a nonexistent object.
Q Given the danger in returning by reference, why not always return by value?
A Far greater efficiency is achieved in returning by reference. Memory is saved and the program runs faster.
#### Workshop

## PART II: Fundamentals of Object-Oriented Programming and C++
What conditional compilation is and how to manage it n How to use the preprocessor in finding bugs n How to manipulate individual bits and use them as flags n What the next steps are in learning to use C++ effectively
### LESSON 10: Classes and Objects

#### Is C++ Object-Oriented?

#### Creating New Types

#### Introducing Classes and Members

#### Accessing Class Members

#### Private Versus Public Access
All members of a class are private, by default.
In effect, by leaving these members as private, you’ve said to the compiler, “I’ll access itsAge, itsWeight, and Meow() only from within member functions of the Cat class.”
#### Implementing Class Methods

#### Adding Constructors and Destructors
The constructor can take parameters as needed, but it cannot have a return value—not even void. The constructor is a class method with the same name as the class itself.

A destructor always has the name of the class, preceded by a tilde (~). Destructors take no arguments and have no return value.

In the event that the constructor takes no parameters at all (that is, that it is a default constructor), you leave off the parentheses and write
#### Including const Member Functions
If you declare a class method const, you are promising that the method won’t change the value of any of the members of the class.
To declare a class method constant, put the keyword const after the parentheses enclosing any parameters but before the semicolon ending the method declaration.
#### Where to Put Class Declarations and Method Definitions

#### Inline Implementation
put the definition of a function into the declaration of the class, which automatically makes that function inline.
#### Classes with Other Classes as Member Data
> What is the difference between declaring and defining?

- A declaration introduces a name of something but does not allocate memory. A definition allocates memory.
- With a few exceptions, all declarations are also definitions. The most important exceptions are the declaration of a global function (a prototype) and the declaration of a class (usually in a header file).
#### Exploring Structures
In C++, a struct is the same as a class, except that its members are public by default and that it inherits publicly, by default.
#### Summary

#### Q&A

#### Workshop

### LESSON 11: Implementing Inheritance

#### What Is Inheritance?

#### Private Versus Protected
Protected data members and functions are fully visible to derived classes, but are otherwise private.

#### Inheritance with Constructors and Destructors
After you override any overloaded method, all the other overrides of that method are hidden. If you want them not to be hidden, you must override them all.
#### Overriding Base Class Functions
DON’T hide a base class function by changing the function signature.
DON’T forget that const is a part of the signature.
DON’T forget that the return type is not part of the signature.
#### Virtual Methods
the virtual function magic operates only on pointers and references. Passing an object by value does not enable the virtual functions to be invoked.

It is legal and common to pass a pointer to a derived object when a pointer to a base object is expected.
If any function in your class is virtual, the destructor should be as well.

##### Virtual Copy Constructors
needs to be able to pass in a pointer to a base object and have a copy of the correct derived object that is created. A common solution to this problem is to create a Clone() method in the base class and to make it virtual. The Clone() method creates a new object copy of the current class and returns that object.
#### Private Inheritance

#### Summary

#### Q&A

#### Workshop

### LESSON 12: Polymorphism
- [ ] What multiple inheritance is and how to use it
- [ ] What virtual inheritance is and when to use it
- [ ] What abstract classes are and when to use them
- [ ] What pure virtual functions are
#### Problems with Single Inheritance
DO move functionality up the inheritance hierarchy when it is conceptually cohesive with the meaning of the ancestor class.
DO avoid performing actions based on the runtime type of the object—use virtual methods, templates, and multiple inheritance.
DON’T clutter ancestor classes with capabilities that are only added to support a need for polymorphism in descendant classes.
DON’T cast pointers to base objects down to derived objects.
#### Multiple Inheritance
Declaring Classes for Virtual Inheritance To ensure that derived classes have only one instance of common base classes, declare the intermediate classes to inherit virtually from the base class.

DO use multiple inheritance when a new class needs functions and features from more than one base class.
DO use virtual inheritance when more than one derived classes have only one instance of the shared base class.
DO initialize the shared base class from the most derived class when using virtual base classes.
DON’T use multiple inheritance when single inheritance will do.
#### Abstract Data Types
In an abstract class, the interface represents a concept (such as shape) rather than a specific object (such as circle). In C++, an abstract class is always the base class to other classes, and it is not valid to make an instance of an abstract class.

A virtual function is made pure by initializing it with zero, as in `virtual void Draw() = 0;`

Any class with one or more pure virtual functions is an abstract class, and it becomes illegal to instantiate.

Typically, the pure virtual functions in an abstract base class are never implemented.

DO use abstract classes to provide common description of capabilities provided in a number of related classes.
DO make pure virtual any function that must be overridden.
DON’T try to instantiate an object of abstract classes.
#### Summary

#### Q&A

#### Workshop

### LESSON 13: Operator Types and Operator Overloading
- [ ] Using the operator keyword
- [ ] Unary and binary operators
- [ ] Conversion operators
- [ ] Operators that cannot be redefined
#### What Are Operators in C++?

#### Unary Operators
The typical definition of a unary operator implemented as a global function or a static member function is
```cpp
return_type operator operator_type (parameter_type) 
{
	// ... implementation 
}
```
A unary operator that is the member of a class is defined as
```cpp
return_type operator operator_type () 
{ 
	// ... implementation 
}
```
##### Programming a Unary Increment/Decrement Operator
A Calendar Class That Handles Day, Month and Year, and Allows Increments in Days
```cpp
#include <iostream>

class CDate {
private:
    int m_nDay; // Range: 1 - 30 (assuming all months have 30 days)
    int m_nMonth; // Range: 1 - 12
    int m_nYear;

    void AddDays(int nDaysToAdd) {
        m_nDay += nDaysToAdd;

        if (m_nDay > 30) {
            AddMonths(m_nDay / 30);
            m_nDay %= 30; // rollover 30th -> 1st
        }
    }

    void AddMonths(int nMonthsToAdd) {
        m_nMonth += nMonthsToAdd;

        if (m_nMonth > 12) {
            AddYears(m_nMonth / 12);
            m_nMonth %= 12; // rollover Dec -> Jan
        }
    }

    void AddYears(int nYearsToAdd) {
        m_nYear += nYearsToAdd;
    }

public:
    // Constructor that initializes the object to a day, month, and year
    CDate(int nDay, int nMonth, int nYear) : m_nDay(nDay), m_nMonth(nMonth), m_nYear(nYear) {}

    // Unary increment operator (prefix)
    CDate& operator++() {
        AddDays(1);
        return *this;
    }

    // Postfix operator: differs from prefix in return-type and parameters
    CDate operator++(int) {
        // Create a copy of the current object, before incrementing day
        CDate mReturnDate(m_nDay, m_nMonth, m_nYear);
        AddDays(1);
        // Return the state before increment was performed
        return mReturnDate;
    }

    void DisplayDate() {
        std::cout << m_nDay << " / " << m_nMonth << " / " << m_nYear;
    }
};
```
##### Programming Conversion Operators
C++ allows you to achieve this by defining custom unary operators. These typically follow the syntax:
```cpp
operator conversion_type();
```

Using Conversion Operators to Convert a CDate into an Integer
```cpp
#include <iostream>

class CDate {
private:
    int m_nDay; // Range: 1 - 30 (assuming all months have 30 days)
    int m_nMonth; // Range: 1 - 12
    int m_nYear;

public:
    // Constructor that initializes the object to a day, month, and year
    CDate(int nDay, int nMonth, int nYear) : m_nDay(nDay), m_nMonth(nMonth), m_nYear(nYear) {}

    // Convert date object into an integer.
    operator int() {
        return ((m_nYear * 10000) + (m_nMonth * 100) + m_nDay);
    }

    void DisplayDate() {
        std::cout << m_nDay << " / " << m_nMonth << " / " << m_nYear;
    }
};

int main() {
    // Instantiate and initialize a date object to 25 May 2008
    CDate mDate(25, 6, 2008);

    std::cout << "The date object is initialized to: ";
    // Display initial date
    mDate.DisplayDate();
    std::cout << std::endl;

    // Get the integer equivalent of the date
    int nDate = mDate;

    std::cout << "The integer equivalent of the date is: " << nDate;
    return 0;
}
```
#### Binary Operators
- The definition of a binary operator implemented as a global function or a static member function is as follows:
```cpp
return_type operator_type (parameter1, parameter2);
```

- The definition of a binary operator implemented as a class member is
```cpp
return_type operator_type (parameter);
```
The reason the class member version of a binary operator accepts only one parameter is that the second parameter is usually derived from the attributes of the class itself.
##### Overload the + operator
```cpp
#include <iostream>

class CDate {
private:
    int m_nDay; // Range: 1 - 30 (assuming all months have 30 days)
    int m_nMonth; // Range: 1 - 12
    int m_nYear;

    void AddDays(int nDaysToAdd) {
        m_nDay += nDaysToAdd;

        if (m_nDay > 30) {
            AddMonths(m_nDay / 30);
            m_nDay %= 30; // rollover 30th -> 1st
        }
    }

    void AddMonths(int nMonthsToAdd) {
        m_nMonth += nMonthsToAdd;

        if (m_nMonth > 12) {
            AddYears(m_nMonth / 12);
            m_nMonth %= 12; // rollover Dec -> Jan
        }
    }

    void AddYears(int nYearsToAdd) {
        m_nYear += nYearsToAdd;
    }

public:
    // Constructor that initializes the object to a day, month, and year
    CDate(int nDay, int nMonth, int nYear) : m_nDay(nDay), m_nMonth(nMonth), m_nYear(nYear) {}

    // Overload the + operator to add days to a date
    CDate operator+(int nDaysToAdd) {
        CDate newDate(m_nDay, m_nMonth, m_nYear);
        newDate.AddDays(nDaysToAdd);
        return newDate;
    }

    void DisplayDate() {
        std::cout << m_nDay << " / " << m_nMonth << " / " << m_nYear;
    }
};

int main() {
    // Instantiate and initialize a date object to 25 May 2008
    CDate mDate(25, 6, 2008);

    std::cout << "The date object is initialized to: ";
    // Display initial date
    mDate.DisplayDate();
    std::cout << std::endl;

    std::cout << "Date after adding 10 days is: ";
    // Adding 10 days...
    CDate datePlus10(mDate + 10);
    datePlus10.DisplayDate();

    return 0;
}
```
##### Subscript Operators
Operators that allow array-style [] access to a class are called subscript operators. The typical syntax of a subscript operator is
```cpp
return_type& operator [] (subscript_type& subscript);
```

Using the Subscript Operator in Programming a Dynamic Array
```cpp
#include <iostream>

class CMyArray {
private:
    int* m_pnInternalArray;
    int m_nNumElements;

public:
    CMyArray(int nNumElements);
    ~CMyArray();

    // Declare a subscript operator
    int& operator[](int nIndex);
};

// Subscript operator: allows direct access to an element given an index
int& CMyArray::operator[](int nIndex) {
    return m_pnInternalArray[nIndex];
}

CMyArray::CMyArray(int nNumElements) {
    m_pnInternalArray = new int[nNumElements];
    m_nNumElements = nNumElements;
}

CMyArray::~CMyArray() {
    delete[] m_pnInternalArray;
}
```
#### Function operator()

#### Operators That Cannot Be Redefined

#### Summary
DO remember to use the const keyword when defining member operators that don’t need to be changing the object’s attributes.
DO remember that programming operators might take effort for the person programming the class, but makes it easy for the person using it.
DO make it a practice to write const and non-const versions for subscript operators that allow bidirectional access—one for reading and the other for writing.
DO make it a practice to write const and non-const versions for subscript operators that allow bidirectional access—one for reading and the other for writing.
DON’T add more operators than necessary. Doing so is a waste of time if no one uses them.
#### Q&A

#### Workshop

### LESSON 14: Casting Operators
- [ ] The need for casting operators
- [ ] Why traditional C-style casts are not popular with some C++ programmers
- [ ] The four C++ casting operators
- [ ] Why C++ casting operators are not all-time favorites
#### What Is Casting?
Casting is a mechanism by which the programmer can temporarily or permanently change the interpretation of an object by the compiler.
#### The Need for Casting

#### Why C-Style Casts Are Not Popular with Some C++ Programmers

#### The C++ Casting Operators
- `static_cast`
- `dynamic_cast`
- `reinterpret_cast`
- `const_cast`
##### static_cast
static_cast is a mechanism that can be used to convert pointers between related types, and perform explicit type conversions for standard data types that would otherwise happen automatically or implicitly.

As far as pointers go, static_cast implements a basic compile-time check to ensure that the pointer is being cast to a related type.

However, note that static_cast verifies only that the pointer types are related. It does not perform any runtime checks. So, with static_cast, a programmer could still get away with this bug:
```cpp
CBase* pBase = new CDerived (); // construct a CDerived object 
CDerived* pDerived = static_cast<CDerived*>(pBase); // ok! 

// CUnrelated is not related to CBase via any inheritance hierarchy 
CUnrelated* pUnrelated = static_cast<CUnrelated*>(pBase); // Error 
//The cast above is not permitted as types are unrelated

CBase* pBase = new CBase (); 
CDerived* pDerived = static_cast<CDerived*>(pBase); // Still no errors!
```
##### Using dynamic_cast and Runtime Type Identification
Dynamic casting, as the name suggests, is the opposite of static casting and actually executes the cast at runtime—that is, at application execution time.
```cpp
destination_type* pDest = dynamic_cast <class_type*> (pSource);
```
##### reinterpret_cast
reinterpret_cast is the closest a C++ casting operator gets to the C-style cast. It really does allow the programmer to cast one object type to another, regardless of whether or not the types are related.

This cast actually forces the compiler to accept situations that static_cast would normally not permit. It finds usage in certain low-level applications (such as drivers, for example) where data needs to be converted to a simple type that the API can accept (for example, some APIs work only with BYTE streams.
```cpp
CSomeClass* pObject = new CSomeClass (); // Need to send the object as a byte-stream... 
unsigned char* pBytes = reinterpret_cast <unsigned char*>(pObject);
```
The cast used in the preceding code has not changed the binary representation of the source object, and has effectively cheated the compiler into allowing the programmer to peek into individual bytes contained by an object of type CSomeClass.

reinterpret_cast changed only the interpretation of the pointer, and did not change the object being pointed to.
##### const_cast
const_cast allows you to turn off the const access modifier to an object.

To makesure we could call to a non-const member which belongs to a third-party library using a const reference or pointer.
```cpp
void DisplayAllData (const CSomeClass& mData) 
{ 
	CSomeClass& refData = const_cast <CSomeClass&>(mData); 
	refData.DisplayMembers(); // Allowed! 
}
```

Don't try to modify the contents of a const-object by casting a pointer / reference to it using const_cast,the result of such an operation is not defined, and definitely not desired.
#### Problems with the C++ Casting Operators
Thus, C++ casting operators other than dynamic_cast are avoidable in modern C++ applications.
#### Summary

#### Q&A

#### Workshop

### LESSON 15: An Introduction to Macros and Templates
- [ ] An introduction to the preprocessor
- [ ] The `#define` keyword and macros
- [ ] An introduction to templates
- [ ] Writing templates for functions and classes
- [ ] The difference between macros and templates
#### The Preprocessor and the Compiler

#### The #define Preprocessor Directive

#### Macro Functions
Macros suffer from four problems in C++
- The first is that they can be confusing if they get large because all macros must be defined on one line. You can extend that line by using the backslash character (\), but larger macros quickly become difficult to manage.
- The second problem is that macros are expanded inline each time they are used. This means that if a macro is used a dozen times, the substitution appears a dozen times in your program, rather than appearing once, as a function call does. On the other hand, they are usually quicker than a function call because the overhead of a function call is avoided.
- The fact that they are expanded inline leads to the third problem, which is that the macro does not appear in the intermediate source code used by the compiler; therefore, it is unavailable in most debuggers. This makes debugging macros tricky.
- The final problem, however, is the biggest: Macros are not type safe. Although it is convenient that absolutely any argument can be used with a macro, this completely undermines the strong typing of C++ and so is anathema to C++ programmers.

DO use CAPITALS for your macro names. This is a pervasive convention, and other programmers will be confused if you don’t.
DO surround all arguments with parentheses in macro functions.
DON’T allow your macros to have side effects. Don’t increment variables or assign values from within a macro.
DON’T use `#define` values when a constant variable will work.
#### An Introduction to Templates
Templates in C++ allow you to define a behavior that you can apply to objects of varying types. This sounds ominously close to what macros let you do (refer to the simple macro MAX that determined the greater of two numbers), save for the fact that macros are type unsafe and templates are type safe.
```cpp
template <parameter list> 
...template declaration..
```

The template declaration is what contains the pattern that you want to implement.

A template declaration can be
- A declaration or definition of a function (as you saw earlier)
- A declaration or definition of a class
- A definition of a member function or a member class of a class template
- A definition of a static data member of a class template
- A definition of a static data member of a class nested within a class template
- A definition of a member template of a class or class template
#### Summary
DO use templates whenever you have a concept that can operate across objects of different classes or across different primitive data types.
DO use the parameters to template functions to narrow their instances to be type safe.
DO specialize template behavior by overriding template functions by type.

#### Q&A

#### Workshop

## PART III: Learning the Standard Template Library (STL)

### LESSON 16: An Introduction to the Standard Template Library

#### STL Containers
Containers are STL classes that are used to store data. STL supplies two types of container classes:
- Sequential containers
- Associative containers

| Container       | Type        | Advantages                                                                                                                                                                                                                                                                              | Disadvantages                                                                                                                                                                                                                                 |
| --------------- | ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `std::vector`   | Sequential  | Quick (constant time) insertion at the end.<br>Array-like access.                                                                                                                                                                                                                       | Resizing can result in performance loss proportional to the number of elements in the container.<br>Insertion only at the end.                                                                                                                |
| `std::deque`    | Sequential  | All advantages of the vector, in addition to allowing for constant-time insertion at the beginning of the container.                                                                                                                                                                    | All disadvantages of the vector are also disadvantages to the deque.<br>Until the vector the deque by specification does not need to feature the `reserve()` function that allows the programmer to save memory space to be used as a vector. |
| `std::list`     | Sequential  | Constant time insertion at the front, middle, or end of the list.<br>Removal of elements from a list is a constant-time activity regardless of the position of the element.<br>Insertion or removal of elements does not invalidate iterators that point to other elements in the list. | Elements cannot be accessed randomly given an index as in an array.<br>Search can be slower because elements are not stored in adjacent memory locations.<br>Search time is proportional to the number of elements in the container.          |
| `std::set`      | Associative | Search is not directly proportionate to the number of elements in the container and hence is often significantly faster than sequential containers.                                                                                                                                     | Insertion of elements is slower than in the vector and its counterparts.                                                                                                                                                                      |
| `std::multiset` | Associative | Advantages similar to that of the `std::set`, to be used when requirements necessitate storage of **nonunique elements** in a sorted container.                                                                                                                                         | Disadvantages similar to that of a `std::set`.                                                                                                                                                                                                |
| `std::map`      | Associative | Key/value pairs container that sorts on the basis of the key.<br>Search is not directly proportionate to the number of elements in the container and hence often significantly faster than sequential containers.                                                                       | Disadvantages similar to that of the `std::set`.                                                                                                                                                                                              |
| `std::multimap` | Associative | Advantages similar to that of the `std::map`, to be selected over `std::map` when requirements necessitate the need of a key/value pairs container that holds **nonunique keys.**                                                                                                       | Disadvantages similar to that of a `std::map`.                                                                                                                                                                                                |
#### STL Iterators
Iterators in STL are template classes that in some ways are generalization of pointers.

Note that operations could as well be STL algorithms that are template functions, Iterators are the bridge that allows these template functions to work with containers, which are template classes, in a consistent and seamless manner.
- **Input iterator**—One that can be dereferenced to reference an object. The object can be in a collection, for instance. Input iterators of the purest kinds guarantee read access only.
- **Output iterator**—One that allows the programmer to write to the collection. Output iterators of the strictest types guarantee write access only.
#### STL Algorithms

#### The Interaction Between Containers and Algorithms Using Iterators

#### Summary

#### Q&A

#### Workshop

### LESSON 17: The STL string Class

#### The Need for String Manipulation Classes
- Increases the stability of the application being programmed by internally managing memory allocation details
- Supplies copy constructor and assignment operators that automatically ensure that member strings get correctly copied
- Provides operators that help in comparisons
-
#### Working with the STL string Class

#### Template-Based Implementation of an STL string
The template declaration of container class basic_string is as follows:
```cpp
template<class _Elem, class _Traits, class _Ax> class basic_string
```

the parameter of utmost importance is the first one: `_Elem`. This is the type collected by the basic_string object. The std::string is therefore the template specialization of basic_string for `_Elem=char`, whereas the wstring is the template specialization of basic_string for `_Elem=wchar_t`.
```cpp
//the STL string class
typedef basic_string<char, char_traits<char>, allocator<char> > string;

//the STL wstring class
typedef basic_string<wchar_t, char_traits<wchar_t>, allocator<wchar_t> > string;
```
#### Summary

#### Q&A

#### Workshop

### LESSON 18: STL Dynamic Array Classes

#### The Characteristics of std::vector

#### Typical Vector Operations

#### Understanding size() and capacity()

#### The STL deque Class

#### Summary

#### Q&A

#### Workshop

### LESSON 19: STL list

#### The Characteristics of a std::list

#### Basic list Operations
The `insert` function returns an iterator that points to the element inserted
#### Reversing and Sorting Elements in a list
list has a special property that iterators pointing to the elements in a list remain valid in spite of rearrangement of the elements or insertion of new elements and so on.
#### Summary

#### Q&A

#### Workshop

### LESSON 20: STL set and multiset

#### An Introduction

#### Basic STL set and multiset Operations

#### Pros and Cons of Using STL set and multiset

#### Summary

#### Q&A

#### Workshop

### LESSON 21: STL map and multimap

#### A Brief Introduction

#### Basic STL map and multimap Operations

#### Supplying a Custom Sort Predicate

#### Summary

#### Q&A

#### Workshop

## PART IV: More STL

### LESSON 22: Understanding Function Objects

#### The Concept of Function Objects and Predicates

#### Typical Applications of Function Objects

#### Summary

#### Q&A

#### Workshop

### LESSON 23: STL Algorithms

#### What Are STL Algorithms?

#### Classification of STL Algorithms

#### Usage of STL Algorithms

#### Summary

#### Q&A

#### Workshop

### LESSON 24: Adaptive Containers: stack and queue

#### The Behavioral Characteristics of Stacks and Queues

#### Using the STL stack Class

#### Using the STL queue Class

#### Using the STL Priority Queue

#### Summary

#### Q&A

#### Workshop

### LESSON 25: Working with Bit Flags Using STL

#### The bitset Class

#### Using std::bitset and Its Members

#### The vector`<bool>`

#### Summary

#### Q&A

#### Workshop

## PART V: Advanced C++ Concepts

### LESSON 26: Understanding Smart Pointers

#### What Are Smart Pointers?

#### How Are Smart Pointers Implemented?

#### Types of Smart Pointers

#### Using the std::auto_ptr

#### Popular Smart Pointer Libraries

#### Summary

#### Q&A

#### Workshop

### LESSON 27: Working with Streams

#### Overview of Streams

#### Streams and Buffers

#### Standard I/O Objects

#### Redirection of the Standard Streams

#### Input Using cin

#### Other Member Functions of cin

#### Outputting with cout

#### Streams Versus the printf() Function

#### File Input and Output

#### Binary Versus Text Files

#### Command-Line Processing

#### Summary

#### Q&A

#### Workshop

### LESSON 28: Exception Handling

#### Bugs, Errors, Mistakes, and Code Rot

#### The Idea Behind Exceptions

#### Placing try Blocks and catch Blocks

#### How Catching Exceptions Work

#### Data in Exceptions and Naming Exception Objects

#### Exceptions and Templates

#### Exceptions Without Errors

#### Bugs and Debugging

#### Summary

#### Q&A

#### Workshop

### LESSON 29: Tapping Further into the Preprocessor

#### The Preprocessor and the Compiler

#### The #define Preprocessor Directive

#### Inclusion and Inclusion Guards

#### String Manipulation

#### Predefined Macros

#### The assert() Macro

#### Bit Twiddling

#### Programming Style

#### Next Steps in Your C++ Development

#### Summary

#### Q&A

#### Workshop

## Appendixes

### APPENDIX A: Working with Numbers: Binary and Hexadecimal

#### Using Other Bases

#### Converting to Different Bases

#### Hexadecimal

### APPENDIX B: C++ Keywords

### APPENDIX C: Operator Precedence

### APPENDIX D: Answers
