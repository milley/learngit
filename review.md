# 嵌套类

[nested classes](https://en.cppreference.com/w/cpp/language/nested_types)

一个class/struct或者union定义在另外一个类中被称作嵌套类。下面定义了一些嵌套类。

## 阐述

嵌套类的名字存在于一个封闭的类作用范围中，嵌套类的成员函数在检查嵌套类的范围之后，在封闭类的作用于内查找类名称。
和封闭类的其他成员一样，嵌套类可以访问封闭类的任意名称(私有、保护等等)，但是它是独立的，不能访问封闭类的this指针。

- 在嵌套类中仅仅可以定义类型名，静态成员，和枚举类 (until C++11)
- 在嵌套类中可以使用封闭类的所以有成员，依据[usual usage rules](https://en.cppreference.com/w/cpp/language/data_members#Usage)

```cpp
int x,y; // globals
class enclose // enclosing class
{
    // note: private members
    int x;
    static int s;
public:
    struct inner // nested class
    {
        void f(int i)
        {
            x = i; // Error: can't write to non-static enclose::x without instance            
            int a = sizeof x; // Error until C++11,
                              // OK in C++11: operand of sizeof is unevaluated,
                              // this use of the non-static enclose::x is allowed.
            s = i;   // OK: can assign to the static enclose::s            
            ::x = i; // OK: can assign to global x
            y = i;   // OK: can assign to global y
        }
 
        void g(enclose* p, int i)
        {
            p->x = i; // OK: assign to enclose::x
        }
    };
};
```

在嵌套类中定义友元函数没有访问封闭类的特殊权限，即使从定义在嵌套类中的函数包体中查找，也可以找到封闭类的私有成员。

在命名空间作用域内封闭函数之外定义的成员需要指定嵌套类：

```cpp
struct enclose
{
    struct inner
    {
        static int x;
        void f(int i);
    };
};
 
int enclose::inner::x = 1;       // definition
void enclose::inner::f(int i) {} // definition
```

嵌套类也可以前置声明后再定义，和在类做作用域内定义相同：

```cpp
class enclose
{
    class nested1;    // forward declaration
    class nested2;    // forward declaration
    class nested1 {}; // definition of nested class
};
 
class enclose::nested2 { }; // definition of nested class
```

嵌套类定义遵循[member access](https://en.cppreference.com/w/cpp/language/access)规则，私有变量不能在封闭类之外使用，尽管可以操作类对象。

```cpp
class enclose
{
    struct nested // private member
    {
        void g() {}
    };
public:
    static nested f() { return nested{}; }
};
 
int main()
{
    //enclose::nested n1 = enclose::f(); // error: 'nested' is private
 
    enclose::f().g();       // OK: does not name 'nested'
    auto n2 = enclose::f(); // OK: does not name 'nested'
    n2.g();
}
```
