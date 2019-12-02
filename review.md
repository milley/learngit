# C++中的虚基类

[Virtual base class in C++](https://www.geeksforgeeks.org/virtual-base-class-in-c/)

在使用多继承的时候，虚基类用在防止给定类在多层继承中多实例。

**需要的虚基类：**

考虑下我们有一个类A。这个类被两个类B和C继承。这两个类又被一个新的类D多继承，如下图所示：

<img src="./img/chat_1.png" width="60%">

跟上面我们看到的一样，B和C都从A继承那么A的数据和成员/函数继承两次。一个通过B另外一个通过C。当A类中的任何数据/函数被D访问，就会引发模棱两可的问题到底哪个数据/函数被调用？一个继承自B另外一个继承自C。这就会导致编译错误。

```c++
#include <iostream>
using namespace std;

class A
{
public:
    void show()
    {
        cout << "Hello from A" << endl;
    }
};

class B : public A
{

};

class C : public A
{

};

class D : public B, public C
{

};

int main()
{
    D obj;
    obj.show();     // error: request for member 'show' is ambiguous
}
```

**编译报错：**

```command
prog.cpp: In function 'int main()':
prog.cpp:29:9: error: request for member 'show' is ambiguous
  object.show();
         ^
prog.cpp:8:8: note: candidates are: void A::show()
   void show()
        ^
prog.cpp:8:8: note:                 void A::show()
```

**如何解决这个问题？**

当基类A同时被B和C继承导致的模棱两可的问题，可以通过定义成虚基类解决：

```c++
class B : virtual public A
{

};

class C : virtual public A
{

};
```

注意：virtual关键字也可以放到public后。这样A就编程了虚基类，数据/成员函数将是同一份拷贝到B和C。

虚基类提供了一种节省空间和避免因多继承引发的模棱两可的问题。当一个基类被定义为虚基类，它可以多次成为间接基类，而不重复数据成员。其数据成员的单个副本由使用虚继承的基类共享。
