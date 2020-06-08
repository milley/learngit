# 什么是拷贝和转换

[What is the copy-and-swap idiom?](https://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom/3279550#3279550)

## 概述

### 什么时候我们需要拷贝和转换？

任何管理资源的类(一个包装类，例如智能指针)需要实现[The Big Three](https://stackoverflow.com/questions/4172722/what-is-the-rule-of-three)。拷贝构造和析构的目的和实现都很简单直接，拷贝赋值操作大概有一些微妙和困难。该怎么做呢？有什么陷阱需要避免呢？

拷贝和转换术语就是解决方案，并且优美的解决了赋值操作在两种情况下：避免代码重复和提供一个强健的异常保证。

### 怎么工作

从概念上讲，使用拷贝构造函数来创建一个本地的拷贝数据，然后将拷贝数据用swap函数交换新旧数据。临时的拷贝则进行析构，旧的数据则被释放。我们则留下了新数据的一份拷贝。

为了使用拷贝和交换，我们需要三件事情：一个拷贝构造，一个析构（同时也是所有包装类的基类，因此可以完成任意的），和一个swap函数。

一个swap函数是一个不抛出异常的函数，可以交换两个类的对象，成员和成员。我们可能尝试使用std::swap来代替我们自己的实现，但是这是不可能的；std::swap使用了拷贝构造和拷贝赋值操作在它的实现中，因此我们最后试着依据它们定义赋值操作。

（不仅仅是这些，不合格的调用swap将会使用我们自己的swap操作，跳过不需要的构造和析构在我们的类中，std::swap是必须的）

## 深度的解释

### 目的

让我们考虑下具体的情况。我们希望管理，在一个无效的类，一个动态的数组。我们使用构造函数，拷贝构造和析构：

```cpp
#include <algorithm>  // std::copy
#include <cstddef>  // std::size_t

class dumb_array {
 public:
  // (default) constructor
  dumb_array(std::size_t size = 0)
    : mSize(size), mArray(mSize ? new int[mSize]() : nullptr) {}

  // copy-constructor
  dumb_array(const dumb_array& other)
    : mSize(other.mSize), mArray(mSize ? new int[mSize] : nullptr) {
    // note that this is non-throwing, because of the data
    // types being used; more attention to detail with regards
    // to exceptions must be given in a more general case, hower
    std::copy(other.mArray, other.mArray + mSize, mArray);
  }
  
  // destructor
  ~dumb_array() {
    delete[] mArray;
  }

 private:
  std::size_t mSize;
  int* mArray;
}
```

这个类可以成功管理数组，但是需要operator=才能工作正确。

### 一个失败的方案

这是一个幼稚的实现大概看起来是这样：

```cpp
// the hard part
dumb_array& operator=(const dumb_array& other) {
  if (this != &other) {  // (1)
    // get rid of the old data...
    delete[] mArray;  // (2)
    mArray = nullptr;  // (2) *(see footnote for rationale)

    // ...and put in the new
    mSize = other.mSize;  // (3)
    mArray = mSize ? new int[mSize] : nullptr;  // (3)
    std::copy(other.mArray, other.mArray + mSize, mArray);  // (3)
  }

  return *this;
}
```

可以说我们完成了；现在可以管理数组，没有内存泄漏。但是忍受了三个问题，在代码中相继的标记为(n)。

1. 第一个是测试自身赋值。这个检查有两个目的：这是一个简单的方式阻止我们从不必要的代码运行自身赋值，它也从微妙的bug保护我们（比如拷贝的时候尝试删除数组）。但是在其他情况仅仅充当降低程序噪音；自身赋值几乎不会发生，所以大多数这样的时间都是浪费。如果不用刻意工作是更好的。
2. 第二点仅仅提供了基本的异常保证。如果new int[mSize]失败，*this将会被改变。（也就是说，size是错误的，data也已经失效）作为强健的异常保证，将需要类似这样的：

```cpp
dumb_array& operator=(const dumb_array& other) {
  if (this != &other) {  // (1)
    // get the new data ready before we replace the old
    std::size_t newSize = other.mSize;
    int* newArray = newSize ? new int[newSize]() : nullptr;  // (3)
    std::copy(other.mArray, other.mArray + newSize, newArray);  // (3)

    // replace the old data(all are non-throwing)
    delete[] mArray;
    mSize = newSize;
    mArray = newArray;
  }

  return *this;
}
```

3. 代码被扩大了。将会导致第三个问题：代码重复。我们的赋值操作出现在每个需要写的地方，这是一个糟糕的事情。

在我们的案例中，核心就仅仅两行（分配和拷贝），但是有更多复杂的资源操作在这个代码中使得很麻烦。我们正确不要重复我们的代码。

（有人可能会怀疑：如果这个代码需要管理一个正确的资源，我们的类如何管理更多的？这看起来像是一个无效的方案，实际上确实需要避免try/catch操作，这是一个无问题的。因为一个类需要管理仅仅一个资源！）

### 一个成功的方案

就像上面提及的，拷贝和转换操作可以解决这些问题。但是现在，我们有所有需要的除了一样：一个swap函数。虽然三个法则成功的要求我们要有拷贝构造，赋值和析构，实际上被称作“三个大的和半个”：任何时候你的类管理资源都会提供有意义的swap函数。

我们需要增加swap函数到我们的类中，就像这样：

```cpp
class dumb_array {
 public:
  // ...

  friend void swap(dumb_array& first, dumb_array& second) {
    // enable ADL(not necessary in our case, but good practice)
    using std::swap;

    // by swapping the numbers of two objects,
    // the two objects are effectively swapped
    swap(first.mSize, second.mSize);
    swap(first.mArray, second.mArray);
  }

  // ...
};
```

（[这里](https://stackoverflow.com/questions/5695548/public-friend-swap-member-function)解释了为什么用public friend swap.）现在不仅仅是可以交换dmb_array，交换普通的是更有效率；它不仅仅是交换指针和大小，同时也分配和拷贝了整个数组。除了功能性和效率的红利，我们现在可以实现拷贝和转换操作。

不需要再进一步，我们的赋值操作是这样：

```cpp
dumb_array& operator=(dumb_array other) {  // (1)
  swap(*this, other);  // (2)
  return *this;
}
```

就这样，三个问题一下子就被巧妙的解决了。

### 它为什么工作？

我们首先注意到一个重要的选择：参数类型是值类型。虽然人们可以很容易的做以下事情（实际上，许多朴素的实现了这些）：

```cpp
dumb_array& operator=(const dumb_array& other) {
  dumb_array temp(other);
  swap(*this, temp);
  return *this;
}
```

我们失去了一个重要的优化时机。不仅仅这样，C++11选择的临界点，稍后就会讨论。（一般的，一个显著的指南如下：如果你想在函数中复制一些东西，让编译器通过参数列表完成它）

不管怎样，这个获得资源的方法是消除重复代码的关键因素：我们通过拷贝构造来获得拷贝，从来不需要重复任何一点。现在拷贝已经有了，我们准备交换它。

观察到上面整个函数新的数据已经分配、拷贝和等待使用。这就是异常给我们在强有力的保证：如果拷贝失败了就不会进入到函数中，因此也不会有改变*this状态的可能。（在我们手动保证异常之前，编译器会自动完成；多么完美）

这一点上我们已经大功告成，因为swap是不抛出异常。我们将当前数据和副本数据交换，安全的修改状态，旧的数据将会放到临时变量。在函数返回时旧的变量将会释放。（在参数作用于结束析构函数会被调用）

因为方案没有重复代码，我们不能在这个操作中介绍bug。注意这意味着我们摆脱自身赋值的检查，允许单独统一实现的operator=操作。（额外的，我不需要在非自身赋值有性能浪费。）

到底什么是拷贝和交换。

### C++11是什么状况？

下一个版本的C++，C++11，其中一项很重要的变化是如何管理资源：三个规则现在变为四个规则（和半个）。为什么？因为我们不仅仅需要拷贝构造我们的资源，还需要移动构造。

幸运的，这非常容易：

```cpp
class dumb_array
{
public:
    // ...

    // move constructor
    dumb_array(dumb_array&& other) noexcept
        : dumb_array() // initialize via default constructor, C++11 only
    {
        swap(*this, other);
    }

    // ...
};
```

怎么回事？重调移动构造的目的：从类的另外一个实例获取资源，将其置于可转让且可销毁的状态。

因此我们要做的就很简单：通过默认构造初始化（C++11新特性），然后用other交换；我们知道一个默认的构造的实例可以安全的分配和析构，所以我们知道other也可以这样，在交换后。

（注意有些编译器不支持委托构造；既然这样，我们需要手动默认构造类。这是不幸的但是不影响大局）

### 它们如何工作？

这将是我们需要改变的地方，因此如何工作？永远记住我们将参数作为值而非引用使用：

```cpp
dumb_array& operator=(dumb_array other);  // (1)
```

现在如果other通过右值初始化，它将是移动构造。非常不错。以同样的方式，C++03让我们重新使用值类型的复制构造函数，C++11将在适当时自动选取移动构造函数。（当然，正如之前链接文章中提到，复制/移动该值可以简单的完全省略）

所以复制和交换属于结束了。

## 脚注

为什么将mArray设置成null？因为如果有任何代码特性在操作中抛出，dumb_array的析构也会被调用；如果发生的时候没有设置为null，我们将试图释放内存！我们通过设置为null来避免，释放null无操作。

其他宣称我们需要给我们的类型指定std::swap，在自由函数swap旁边提供一个类内置swap，等等。但是这不是必须的：在适用swap的时候通过不适当的调用，我们的函数通过[ADL](http://en.wikipedia.org/wiki/Argument-dependent_name_lookup)将会找到。一个函数将执行。

原因很简单：一旦你有资源，在需要的时候你应该交换或者移动它(C++11)在任何时候。通过在参数列表中创建副本可以最大化优化。

移动构造函数需要指定noexcept，否则某些代码（例如：std::vector逻辑修改大小）将使用复制构造函数，即使移动构造有意义。当然，仅仅在代码内部没有引发异常的情况下，才可以标记为noexcept。
