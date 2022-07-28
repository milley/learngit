# Rust中Copy trait

Rust语言和其他编程语言的最大区别就是引入了所有权规则。所有权规则的定义如下：

- Each value in Rust has an owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

我们简单翻译下：

- 一个值只能被一个变量所拥有，这个变量被称为所有者
- 一个值同一时刻只能有一个所有者
- 当所有者离开作用域，其拥有的值被丢弃

这三条规则核心就是保证单一所有权。第二条规则是从C++借鉴的Move语义。

先看一个简单例子：

```rust
#[derive(Debug)]
struct Foo;

fn main() {
    let x = Foo;
    let y = x;

    // `x` has moved into `y`, and so cannot be used
    println!("{x:?}");
}
```

Foo默认绑定的是move语义，当x赋值给y变量后，x的就会移动到y，再次使用x就会报错：error[E0382]: borrow of moved value: `x`

如果要在赋值后接着使用x，那么就需要实现copy语义：

```rust
// We can derive a `Copy` implementation. `Clone` is also required, as it's
// a supertrait of `Copy`.
#[derive(Debug, Copy, Clone)]
struct Foo;

fn main() {
    let x = Foo;
    let y = x;

    // `x` has moved into `y`, and so cannot be used
    println!("{x:?}");
}
```

有两种方法来实现Copy trait。

第一种就是上面示例使用derive:

```rust
#[derive(Copy, Clone)]
struct MyStruct;
```

第二种可以手动实现Copy和Clone：

```rust
// We can derive a `Copy` implementation. `Clone` is also required, as it's
// a supertrait of `Copy`.
#[derive(Debug, Copy, Clone)]
struct Foo;

use std::fmt;
struct MyStruct;

impl fmt::Debug for MyStruct {
    fn fmt(&self, fmt: &mut fmt::Formatter) -> fmt::Result {
        fmt.debug_struct("MyStruct")
            .finish()
    }
}

impl Copy for MyStruct { }

impl Clone for MyStruct {
    fn clone(&self) -> MyStruct {
        *self
    }
}

fn main() {
    let x = Foo;
    let y = x;

    // `x` has moved into `y`, and so cannot be used
    println!("{x:?}");

    let ms0 = MyStruct;
    let ms1 = ms0;
    println!("{ms0:?}");
}
```

## Copy和Clone的差别

两种方式有个细微的差别：derive策略也会绑定Copy到类型参数中，这并不总是需要这样做。

Copy是隐式发生的，例如上面的赋值y=x。Copy的行为没有被重载，仅仅是简单的位拷贝。Clone是显式的行为，例如x.clone()。Clone实现提供了安全赋值必须的任何指定类型的行为。例如，实现String的Clone trait需要拷贝堆中指向字符串缓冲区。String值的简单拷贝将会拷贝指针，导致双倍空闲量。所以String只实现了Clone而没有实现Copy。

Clone是Copy的父trait，所以实现Copy必须实现Clone。如果类型是Copy因此Clone实现仅仅需要返回*self。

如果struct的构成都实现了Copy，那么这个struct也可以实现Copy。例如：

```rust
#[derive(Copy, Clone)]
struct Point {
   x: i32,
   y: i32,
}
```

```rust
struct PointList {
    points: Vec<Point>,
}
```

作为对比，PointList结构不能实现Copy，因为Vec<T>不能实现。如果尝试实现Copy就会得到一个错误：the trait `Copy` may not be implemented for this type; field `points` does not implement `Copy`

共享引用(&T)可以实现Copy，因此一个类型可以Copy，只有当含有共享引用的类型T不能Copy才可以。下面的struct，可以实现Copy，因为它仅有包含了不能实现Copy的PointList类型：

```rust
#[derive(Copy, Clone)]
struct PointListWrapper<'a> {
    point_list_ref: &'a PointList,
}
```

## 不能使用Copy的场景

有些类型不能安全的拷贝。例如，拷贝&mut T将会创建一个可变引用的别名。拷贝String将会复制管理String缓冲区的所有权，导致重复释放。

普遍来说，任何实现了Drop的类型都不能Copy，因为它除了管理自身的size_of::<T>之外还管理其他资源。

## 使用Copy的场景

通常来说，如果类型可以实现Copy，那么就可以。不过请记住，实现Copy是类型公共API的一部分。如果将来会变成不能拷贝，谨慎起见还是现在就省略掉Copy，来避免破坏性的API修改。

## 常用类型是否实现Copy

可以通过下面代码来判断常用类型是否实现Copy：

```rust
fn is_copy<T: Copy>() {}

fn types_impl_copy_trait() {
    is_copy::<bool>();
    is_copy::<char>();

    // all iXX and uXX, usize/isize, fXX implement Copy trait
    is_copy::<i8>();
    is_copy::<u64>();
    is_copy::<i64>();
    is_copy::<usize>();

    // function (actually a pointer) is Copy
    is_copy::<fn()>();

    // raw pointer is Copy
    is_copy::<*const String>();
    is_copy::<*mut String>();

    // immutable reference is Copy
    is_copy::<&[Vec<u8>]>();
    is_copy::<&String>();

    // array/tuple with values which is Copy is Copy
    is_copy::<[u8; 4]>();
    is_copy::<(&str, &str)>();
}

fn types_not_impl_copy_trait() {
    // unsized or dynamic sized type is not Copy
    is_copy::<str>();
    is_copy::<[u8]>();
    is_copy::<Vec<u8>>();
    is_copy::<String>();

    // mutable reference is not Copy
    is_copy::<&mut String>();

    // array / tuple with values that not Copy is not Copy
    is_copy::<[Vec<u8>; 4]>();
    is_copy::<(String, u32)>();
}

fn main() {
    types_impl_copy_trait();
    types_not_impl_copy_trait();
}
```

types_not_impl_copy_trait方法中都没有实现Copy，因此编译器会打印出每种未实现Copy的具体报错信息。
