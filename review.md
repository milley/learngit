# 深拷贝vs浅拷贝

[Shallow Copy and Deep Copy in C++](https://www.geeksforgeeks.org/shallow-copy-and-deep-copy-in-c/)

通常，创建对象的拷贝是指创建一个完整的副本，和原始对象有同样的字面值，数据类型和资源。

为了创建对象副本，依靠对象持有动态内存资源，我们需要执行深拷贝或者浅拷贝。通常来说，如果对象中的变量可以动态的分配，需要优先使用深拷贝而不是浅拷贝。

## 浅拷贝

浅拷贝中，对象通过简单拷贝原始对象中变量的值来完成创建。如果对象中的变量没有定义在堆内存中，这样是没有问题的。如果有的变量是动态的由堆段分配，拷贝对象变量则会引用同一块内存区域。
这将会导致不唯一的运行时错误，悬垂指针。自从两个对象引用到同一块内存区域，改变一个对象的变量则会反映到另一个对象。因为需要创建对象的副本，浅拷贝无法满足这个目的。

```cpp
#include <iostream>
using namespace std;

// Box Class
class Box {
private:
    int length;
    int breadth;
    int height;

public:
    void set_dimensions(int l, int b, int h) {
        length = l;
        breagth = b;
        height = h;
    }

    void show_data() {
        cout << " Length = " << length
             << "\n Breadth = " << breadth
             << "\n Height = " << height
             << endl;
    }
};

int main() {
    Box B1, B3;

    B1.set_dimensions(14, 12, 16);
    B1.show_data();

    Box B2 = B1;
    B2.show_data();

    B3 = B1;
    B3.show_data();

    return 0;
}
```

输出结果：

```txt
Length = 14
 Breadth = 12
 Height = 16
 Length = 14
 Breadth = 12
 Height = 16
 Length = 14
 Breadth = 12
 Height = 16
```

## 深拷贝

在深拷贝中，对象拷贝所有的变量，同时也会分配类似的内存资源和同样的值。为了执行深拷贝，我们需要显示定义拷贝构造函数和必要的内存分配。同样需要在其他构造中定义变量的动态内存分配。

```cpp
#include <iostream>
using namespace std;

// Box Class
class Box {
private:
    int length;
    int *breadth;
    int height;

public:
    Box() {
        breath = new int;
    }

    Box(Box& box) {
        length = box.length;
        breadth = new int;
        *breadth = *(box.breadth);
        height = box.height;
    }

    ~Box() {
        delete breadth;
    }

    void set_dimensions(int l, int b, int h) {
        length = l;
        *breagth = b;
        height = h;
    }

    void show_data() {
        cout << " Length = " << length
             << "\n Breadth = " << *breadth
             << "\n Height = " << height
             << endl;
    }
};

int main() {
    // Object of class first
    Box first;
 
    // Set the dimensions
    first.set_dimension(12, 14, 16);
 
    // Display the dimensions
    first.show_data();
 
    // When the data will be copied then
    // all the resources will also get
    // allocated to the new object
    Box second = first;
 
    // Display the dimensions
    second.show_data();
 
    return 0;
}
```

输出结果：

```txt
Length = 12
 Breadth = 14
 Height = 16
 Length = 12
 Breadth = 14
 Height = 16
```

下面表格对比浅拷贝和深拷贝：

| Shallow Copy | Deep Copy |
| ------------ | --------- |
| 当创建一个对象拷贝，使用拷贝所有成员的数据，称作浅拷贝 | 当通过复制另外一个对象的数据以及驻留在该对象内外的内存资源来创建对象时，称作深拷贝 |
| 浅拷贝一个对象会拷贝所有成员变量的值 | 深拷贝是执行我们实现的拷贝构造函数 |
| 浅拷贝中，两个对象不是独立的 | 拷贝所有字段，然后通过字段拷贝动态分配内存指针 |
| 也会创建动态分配对象的副本 | 如果没有正确实现深拷贝则会指向原始的，产生性能极差的后果 |
