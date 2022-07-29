# Rust值的借用

在Rust中，如果不希望值的所有权被转移，也没有实现Copy trait而无法使用Copy语义，就可以使用Borrow语义，也就是所谓的借用。Borrow通过引用语法(&或者&mut)来实现。

## 只读引用

本地变量、函数参数和临时存储单元在函数结束后都会释放掉。所以在函数内返回本地变量引用会直接报错：returns a reference to data owned by the current function

```rust
fn main() {
    fn get_dangling_reference() -> &'static i32 {
        let x = 0;
        &x
    }
}
```

返回函数内容器的迭代器也会报错：returns a reference to data owned by the current function

```rust
fn main() {
use std::slice::Iter;
    fn get_dangling_iterator<'a>() -> Iter<'a, i32> {
        let v = vec![1, 2, 3];
        v.iter()
    }
}
```

以上都是因为在函数结束后会释放掉内部变量，因此在main函数中就无法使用。如果要想在main中使用，可以返回所有权的值：

```rust
fn main() {
    use std::vec::IntoIter;
    
    fn get_integer() -> i32 {
        let x = 0;
        x
    }
    
    fn get_owned_iterator() -> IntoIter<i32> {
        let v = vec![1, 2, 3];
        v.into_iter()
    }
}
```

如果在堆变量中引用栈上的变量，那么要确保栈变量的生命周期比堆变量更长，要不然也会被编译器拦截。例如：

```rust
fn main() {
    let mut data: Vec<&u32> = Vec::new();
    push_local_ref(&mut data);
    println!("data: {:?}", data);
}

fn push_local_ref(data: &mut Vec<&u32>) {
    let v = 42;
    data.push(&v)
}
```

这个例子中data添加了局部变量v的借用，然而push_local_ref执行结束后v变量生命周期已经结束，因此违反了：同一个作用域，同一时刻，一个值只能有一个所有者。

## 可变引用

```rust
fn main() {
    let mut data = vec![1, 2, 3];
    for item in data.iter_mut() {
        data.push(*item + 1);
    }
}
```

在一个容器中，循环遍历容器同时又添加新的元素，这样很容易引起死循环等问题。rust编译器会阻止此情况发生：cannot borrow `data` as mutable more than once at a time

```rust
fn main() {
    fn bar(x: &mut i32) {}
    fn foo(a: &mut i32) {
        let y = &a; // a is borrowed as immutable.
        bar(a); // error: cannot borrow `*a` as mutable because `a` is also borrowed
                //        as immutable
        println!("{}", y);
    }
}
```

上面例子中，a是可变引用，在foo中借用给不可变变量y，然后再将a传入给bar函数中可变引用参数，编译器就会报错。如果把bar调用和不可变借用两行互换位置，则可以正常编译：

```rust
fn main() {
    fn bar(x: &mut i32) {}
    fn foo(a: &mut i32) {
        bar(a);
        let y = &a; // ok!
        println!("{}", y);
    }
}
```

下面的例子同理，也是无法编译：

```rust
fn main() {
    let mut data = vec![1, 2, 3];
    let data1 = vec![&data[0]];
    println!("data[0]: {:p}", &data[0]);

    for i in 0..100 {
        data.push(i);
    }

    println!("data[0]: {:p}", &data[0]);
    println!("boxed: {:p}", &data1);
}
```

这里有潜在内存不安全操作：如果继续添加元素，堆上的数据预留的空间不够了，就会重新分配一片足够大的内存，把之前的值拷过来，然后释放旧的内存。这样会让data1中保存的&data[0]引用失效，导致内存安全问题。
