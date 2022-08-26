# Rust过程宏1

## 声明宏

声明宏对代码模板做简单的替换。下面使用一个实例来实现vector宏的功能：

```rust
#[macro_export]
macro_rules! my_vec {
    // Create a None vector
    () => {
        std::vec::Vec::new()
    };
    // process my_vec![1, 2, 3]
    ($($el:expr),*) => ({
        let mut v = std::vec::Vec::new();
        $(v.push($el);)*
        v
    });
    // process my_vec![0; 10]
    ($el:expr; $n:expr) => {
        std::vec::from_elem($el, $n)
    }
}

fn main() {
    let mut v = my_vec![];
    v.push(1);
    println!("{:?}", v);

    let _v = my_vec!(1, 2, 3, 4);
    let _v = my_vec![1, 2, 3, 4];
    let v = my_vec!{1, 2, 3, 4};
    println!("{:?}", v);

    println!("{:?}", v);

    let v = my_vec![1; 10];
    println!("{:?}", v);
}
```

## 第一个简单的过程宏

Rust中可以使用标准库中的#[proc_macro]来定义过程宏，也可以使用宏属性#[proc_macro_attribute]和用户自定义导出属性#[proc_macro_derive]。

在Cargo.toml文件中，需要使用以下节点：

```toml
[lib]
proc-macro = true
```

定义一个最简单的宏：

```rust
use proc_macro::TokenStream;

#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    println!("{:#?}", input);
    "fn hello() { println!(\"hello world\"); }".parse().unwrap()
}
```

这里通过输出input这个TokenStream来看解析的是什么样，需要新建一个example来调用：

```rust

use basic::sql;

fn main() {
    sql!(select * from table1 where id = 10 and timestamp > 1000 order by timestamp desc limit 10);
}
```

通过命令cargo run --example sql来运行结果：

```txt
TokenStream [
    Ident {
        ident: "select",
        span: #0 bytes(39..45),
    },
    Punct {
        ch: '*',
        spacing: Alone,
        span: #0 bytes(46..47),
    },
    Ident {
        ident: "from",
        span: #0 bytes(48..52),
    },
    Ident {
        ident: "table1",
        span: #0 bytes(53..59),
    },
    Ident {
        ident: "where",
        span: #0 bytes(60..65),
    },
    Ident {
        ident: "id",
        span: #0 bytes(66..68),
    },
    Punct {
        ch: '=',
        spacing: Alone,
        span: #0 bytes(69..70),
    },
    Literal {
        kind: Integer,
        symbol: "10",
        suffix: None,
        span: #0 bytes(71..73),
    },
    Ident {
        ident: "and",
        span: #0 bytes(74..77),
    },
    Ident {
        ident: "timestamp",
        span: #0 bytes(78..87),
    },
    Punct {
        ch: '>',
        spacing: Alone,
        span: #0 bytes(88..89),
    },
    Literal {
        kind: Integer,
        symbol: "1000",
        suffix: None,
        span: #0 bytes(90..94),
    },
    Ident {
        ident: "order",
        span: #0 bytes(95..100),
    },
    Ident {
        ident: "by",
        span: #0 bytes(101..103),
    },
    Ident {
        ident: "timestamp",
        span: #0 bytes(104..113),
    },
    Ident {
        ident: "desc",
        span: #0 bytes(114..118),
    },
    Ident {
        ident: "limit",
        span: #0 bytes(119..124),
    },
    Literal {
        kind: Integer,
        symbol: "10",
        suffix: None,
        span: #0 bytes(125..127),
    },
]
```

可以看出TokenStream被解析成了Ident、Punct、Literal等元素。

写过程宏经常会用到2个Crate：

- [syn](https://docs.rs/syn/1.0.99/syn/)
- [quote](https://docs.rs/quote/1.0.21/quote/)

syn可以解析Rust标记流解析成Rust代码中的语法树。主要有以下几种API：

- Data structures
- Derives
- Parsing
- Location information
- Feature flags

quote!宏将Rust语法树数据结构转换成代码中的标记。

下一节，先不使用syn和quote来实现一个过程宏的基本用法。
