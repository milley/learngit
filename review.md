# Rust参考-宏

[Macros](https://doc.rust-lang.org/nightly/reference/macros.html)

使用自定义扩展rust的功能性和语法被称作宏。它有指定的名称，通过前后一致的语法来调用：some_extension!(...)。

定义新的宏有两个方法：

- 在高层级定义新的语法，声明式方式
- 定义类函数宏，自定义推导，使用函数自定义属性来操作输入标记

## 宏调用

```macro
Syntax
MacroInvocation :
   SimplePath ! DelimTokenTree

DelimTokenTree :
      ( TokenTree* )
   | [ TokenTree* ]
   | { TokenTree* }

TokenTree :
   Tokenexcept delimiters | DelimTokenTree

MacroInvocationSemi :
      SimplePath ! ( TokenTree* ) ;
   | SimplePath ! [ TokenTree* ] ;
   | SimplePath ! { TokenTree* }
```

宏调用在编译器展开后将调用结果使用宏的结果来代替。宏调用通过下面方式：

- 表达式和语句(Expressions and statements)
- 模式(Patterns)
- 类型(Types)
- 包含附属条目(Items including associated items)
- macro_rules转换器(macro_rules transcribers)
- 外部块(External blocks)

当使用条目或者语句时，在结尾需要分号的地方，当不使用括号就会使用MacroInvocationSemi。在宏调用之前或者macro_rules定义之前都不允许修饰符可见。

```rust
#![allow(unused)]
fn main() {
    // Used as an expression.
    let x = vec![1, 2, 3];

    // Used as a statement.
    println!("Hello!");

    // Used in a pattern.
    macro_rules! pat {
        ($i:ident) => (Some($i))
    }

    if let pat!(x) = Some(1) {
        assert_eq!(x, 1);
    }

    // Used in type.
    macro_rules! Tuple {
        { $A:ty, $B:ty } => { ($A, $B) };
    }

    type N2 = Tuple!(i32, i32);

    // Used in an item.
    use std::cell::RefCell;
    thread_local!(static FOO: RefCell<u32> = RefCell::new(1));

    // Used as an associated item.
    macro_rules! const_maker {
        ($t:ty, $v:tt) => { const CONST: $t = $v; };
    }
    trait T {
        const_maker!{i32, 7}
    }

    // Macro calls within macros.
    macro_rules! example {
        () => { println!("Macro call in a macro!") };
    }
    // Outer macro `example` is expanded, then inner macro `println` is expanded.
    example!();
}
```

## 宏示例

```macro
Syntax
MacroRulesDefinition :
   macro_rules ! IDENTIFIER MacroRulesDef

MacroRulesDef :
      ( MacroRules ) ;
   | [ MacroRules ] ;
   | { MacroRules }

MacroRules :
   MacroRule ( ; MacroRule )* ;?

MacroRule :
   MacroMatcher => MacroTranscriber

MacroMatcher :
      ( MacroMatch* )
   | [ MacroMatch* ]
   | { MacroMatch* }

MacroMatch :
      Tokenexcept $ and delimiters
   | MacroMatcher
   | $ ( IDENTIFIER_OR_KEYWORD except crate | RAW_IDENTIFIER | _ ) : MacroFragSpec
   | $ ( MacroMatch+ ) MacroRepSep? MacroRepOp

MacroFragSpec :
      block | expr | ident | item | lifetime | literal
   | meta | pat | pat_param | path | stmt | tt | ty | vis

MacroRepSep :
   Tokenexcept delimiters and MacroRepOp

MacroRepOp :
   * | + | ?

MacroTranscriber :
   DelimTokenTree
```

macro_rules允许用户通过声明的方法定义语法扩展。我们称这些扩展为”示例宏“或者“简单宏”。

每个示例宏都有名字，有一个或者多个规则。每个规则有两部分：一个匹配，描述匹配的语法，和一个转换器，描述了替换成功匹配调用的语法。匹配和转换器两部分都必须用分隔符包围起来。宏可以展开为表达式，语句，条目（包括traits,impls和外部条目），类型和匹配。

### 转换

当宏被调用，宏展开看起来使用名字调用，然后挨个尝试每种规则。它尝试转换第一个成功的匹配，如果结果是错误，后面的匹配则不会进行。当匹配过程中，不允许预先匹配；如果编译器在调用时不能明确决定如何解析宏调用哪一个标记，就会出现错误。在下面例子中，编译器无法通过标识符知道接下来的标记是否是)，尽管允许它明确的解析调用：

```rust
macro_rules! ambiguity {
    ($($i:ident)* $j:ident) => { };
}

ambiguity!(error); // Error: local ambiguity
```

在匹配和转换中，$标记被用来调用从宏引擎定义的特殊的行为。从字面上看标记不属于这个调用匹配和转换的一部分，除了一种例外。这种例外就是外部分隔符将匹配任意的分隔符配对。因此，比如说，匹配(())将匹配{()}但是不能匹配{{}}。字面上字符$不能匹配和转换。

当转换一个匹配的片段到另一个示例宏，第二个宏中的匹配将会看到一个模糊的片段类型中的AST。第二个宏不能使用字面标记匹配片段中的匹配，仅仅一个片段指定者有同样的类型。标识符，生命周期和tt片段类型是例外，可以用字面标记来匹配。下面阐述了这个解释：

```rust
macro_rules! foo {
    ($l:expr) => { bar!($l); }
// ERROR:               ^^ no rules expected this token in macro call
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

下面描述在tt片段后如何直接匹配：

```rust
// compiles OK
macro_rules! foo {
    ($l:tt) => { bar!($l); }
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

### 元变量

在匹配中，$ name: fragment-specifer 指定类型的Rust语法片段相匹配，然后绑定到元变量$ name。指定允许的片段：

- item: 一个条目
- block: 一个块表达式
- stmt: 不带分号的声明
- pat_param: 一个PatternNoTopAlt 
- pat: 最少PatternNoTopAlt，尽可能多依赖版本
- expr: 一个表达式
- ty: 一个类型
- ident: 一个关键标识或原始标识
- path: 一个TypePath类型目录
- tt: 一个TokenTree（单个标识或多个标识匹配分割(),[],{}）
- meta: 一个Attr，属性上下文
- lifetime: 声明周期标识
- vis: 可为空的Visibility限定词
- Literal: 匹配LiteralExpression
  
在转换中，元变量被简单指定$ name，自从类型片段指定了匹配器。元变量使用匹配元素的语法来替换。元变量关键字 $crate 可以被当前crate用来参考。元变量可以被多次转换。

为了向后兼容的原因，_也是一个表达式，一个单独的下划线不能被expr 片段匹配。但是，当有子表达式出现_可以被expr片段指定。

### 重复

在匹配和转换中，在$(...)内部通过替换标识指定了重复，在重复的操作符后，可选择分隔符。分割标识符可以是任意的标识符，不同于分隔符和重复操作符，一般是;和,比较常见。例如，$( $i:ident ),* 描述了任意数量的标识符用逗号分隔。允许嵌套重复。

重复操作符有：

- \* 指定任意多次重复
- \+ 指定至少一次，任意多次重复
- \? 指定出现0次或者1次的可选片段

自从?指定了最多一次出现后，就不能被用来进行分隔。

重复出现的片段同样可以匹配和转换指定次数的片段，使用分隔符来分隔。元变量匹配每个重复的适当片段。例如，上面的示例$( $i:ident ),*匹配$i到列表中的每个标识符。

在转换过程中，额外限制允许重复因此编译器知道如何正确的展开他们。

1. 一个元变量必须出现在同样数量，类型，重复的转换嵌套顺序直到它匹配。

```txt
所以对于$( $i:ident ),*匹配，转换=> { $i }, => { $( $( $i)* )* }，紧接着=> { $( $i )+ }是非法的。但是 => { $( $i );* }是合法的，使用分号分隔列表替换逗号分隔列表。
```

2. 每个重复的转换至少包含一个元变量来决定需要展开多少次。如果在同一个重复地方出现多个元变量，它必须绑定到同样次数的片段。

```txt
例如( $( $i:ident ),* ; $( $j:ident ),* ) => (( $( ($i,$j) ),* ))必须绑定同样次数的$i片段至$j片段。意味着使用宏调用(a, b, c; d, e, f)是合法的，将会展开为((a,d), (b,e), (c,f))，但是(a, b, c; d, e)是非法的因为不包含同样个数。这需要适用每一层嵌套重复。
```

### Dollar-dollar ($$)

$$展开为单个$。

当元变量表达式经常接受宏展开，他们不能被用在递归宏定义，这也是$$表达式存在的意义，例如$$可以用来解决嵌套宏的不确定性。

下面的例子阐述了由于嵌套宏中存在重复的不确定性导致编译失败：

```rust
macro_rules! foo_error {
    () => {
        macro_rules! bar_error {
            ( $( $any:tt )* ) => { $( $any )* };
            // ^^^^^^^^^^^ error: attempted to repeat an expression containing no syntax variables matched as repeating at this depth
        }
    };
}

foo_error!();
```

下面通过$$来转义$来解决这个问题：

```rust
macro_rules! foo_ok {
    () => {
        macro_rules! bar_ok {
            ( $$( $any:tt )* ) => { $$( $any )* };
        }
    };
}

foo_ok!();
```

深层嵌套会导致这样展开后$$声明会线性增长，开始是$$，然后是$$$$，接着是$$$$$$等等。在每个层级使用其他名称来指定元变量是非常有比必要的。

```rust
$foo          => bar      => bar    // Evaluate foo at level 1
$$foo         => $foo     => bar    // Evaluate foo at level 2
$$$foo        => $bar     => baz    // Evaluate foo at level 1, and use that as a name at level 2
$$$$foo       => $$foo    => $foo   // Evaluate foo at level 3
$$$$$foo      => $$bar    => $bar   // Evaluate foo at level 1, and use that as a name at level 3
$$$$$$foo     => $$$foo   => $bar   // Evaluate foo at level 2, and use that as a name at level 3
$$$$$$$foo    => $$$bar   => $baz   // Evaluate foo at level 1, use that at level 2, and then use *that* at level 3
```

### 作用于，导出和导入

由于历史原因，宏示例的作用域不能像元素一样。宏有两类作用域：文字作用域，基于路径作用域。文字作用域基于代码文件中出现的顺序，甚至多个文件，是默认的作用域。下面会详细介绍。路径作用域工作方式和元素作用域相同。宏的作用域，导出和导入用属性来广泛的控制。

当使用绝对的标识符（不是多个路径的一部分）来调用宏，首先会查找文字作用域。如果没有产生任何结果，就会查找路径作用域。如果宏的名字被指定到路径中，因此它仅仅查找路径作用域。

```rust
use lazy_static::lazy_static; // Path-based import.

macro_rules! lazy_static { // Textual definition.
    (lazy) => {};
}

lazy_static!{lazy} // Textual lookup finds our macro first.
self::lazy_static!{} // Path-based lookup ignores our macro, finds imported one.
```

### 文本作用域

很大程度上文本作用域是基于代码文件中出现的顺序，工作方式和使用let定义局部变量类似，除非它也在模块级别定义了。当使用macro_rules!来定义一个宏，在定义之后进入宏的作用域，直到指定模块关闭，都是它的作用域。这个可以进入到子模块甚至横跨多个文件：

```rust
//// src/lib.rs
mod has_macro {
    // m!{} // Error: m is not in scope.

    macro_rules! m {
        () => {};
    }
    m!{} // OK: appears after declaration of m.

    mod uses_macro;
}

// m!{} // Error: m is not in scope.

//// src/has_macro/uses_macro.rs

m!{} // OK: appears after declaration of m in src/lib.rs
```

多次定义一个宏不会得到错误；后面的声明会覆盖前面的，直到超出作用域。

```rust
macro_rules! m {
    (1) => {};
}

m!(1);

mod inner {
    m!(1);

    macro_rules! m {
        (2) => {};
    }
    // m!(1); // Error: no rule matches '1'
    m!(2);

    macro_rules! m {
        (3) => {};
    }
    m!(3);
}

m!(1);
```

宏可以在函数内声明和使用，类似这样：

```rust
fn foo() {
    // m!(); // Error: m is not in scope.
    macro_rules! m {
        () => {};
    }
    m!();
}


// m!(); // Error: m is not in scope.
```

### macro_use属性

macro_use有两个目的。第一，在模块关闭的时候，可以被用作模块的宏作用域，通过它申请到模块：

```rust
#[macro_use]
mod inner {
    macro_rules! m {
        () => {};
    }
}

m!();
```

第二，它可以被用来从另外一个crate导入宏，通过声明extern crate附加在crate的root模块。使用macro_use预处理导入宏的这个方法非常重要，意味着可以用任何名字来隐藏。在导入语句之前可以使用#[macro_use]来导入，在冲突的情况下，最后一个宏则会胜出。可选的，可以使用MetaListIdents语法来指定一些列的导入宏。当模块中有#[macro_use]是不支持的。

```rust
#[macro_use(lazy_static)] // Or #[macro_use] to import all macros.
extern crate lazy_static;

lazy_static!{}
// self::lazy_static!{} // Error: lazy_static is not defined in `self`
```

使用#[macro_use]导入宏必须使用#[macro_export]导出，如下面描述。

### 局域路径作用域

默认来说，一个宏没有基于路径的作用域。但是，如果它使用了#[macro_export]属性，那么它被定义在crate root范围内就可以正常使用：

```rust
self::m!();
m!(); // OK: Path-based lookup finds m in the current module.

mod inner {
    super::m!();
    crate::m!();
}

mod mac {
    #[macro_export]
    macro_rules! m {
        () => {};
    }
}
```

使用#[macro_export]标记宏常常是pub并且可以被其他crate使用，使用路径或者#[macro_use]如上面所述。

### 保健

默认情况下，宏中引用的标识符都是按原样展开，并查看宏的调用站点。如果宏引用了不在调用范围内的宏或元素，则会导致问题。为了缓解这种情况，$crate元变量可以被用在路径开始，来强制查找出现在crate内的定义。

```rust
//// Definitions in the `helper_macro` crate.
#[macro_export]
macro_rules! helped {
    // () => { helper!() } // This might lead to an error due to 'helper' not being in scope.
    () => { $crate::helper!() }
}

#[macro_export]
macro_rules! helper {
    () => { () }
}

//// Usage in another crate.
// Note that `helper_macro::helper` is not imported!
use helper_macro::helped;

fn unit() {
    helped!();
}
```

注意，$crate属于当前crate，引用非宏元素时，必须使用全部模块名称来指定。

```rust
pub mod inner {
    #[macro_export]
    macro_rules! call_foo {
        () => { $crate::inner::foo() };
    }

    pub fn foo() {}
}
```

此外，尽管$crate在它的crate内允许元素展开，它的使用对可见性没有影响。元素或宏在调用时必须是可见的。下面例子中，因为foo()不是public，我们尝试从crate外调用call_foo!()将会导致错误。

```rust
#[macro_export]
macro_rules! call_foo {
    () => { $crate::foo() };
}

fn foo() {}
```

当宏导出时，#[macro_export]属性可以增加local_inner_macros关键字来自动给包含的每个宏增加$crate::前缀。在Rust2018版本增加$crate基于路径导入之前，这作为一个主要的工具来迁移编写的代码。新代码中这样使用：

```rust
#[macro_export(local_inner_macros)]
macro_rules! helped {
    () => { helper!() } // Automatically converted to $crate::helper!().
}

#[macro_export]
macro_rules! helper {
    () => { () }
}
```

### 遵循既定的模糊限制

宏系统使用的解析器是非常强大的，但它为了防止在当前版本或者未来版本语言中有模糊不清，也是有限制的。通常来说，除了关于模糊扩展，元变量未终结匹配必须紧跟着令牌后，该令牌在匹配后可以安全的使用。

```txt
举例来说，一个宏匹配$i:expr [ , ]在当前rust版本可以使用，当[,]不在是合法表达式的一部分，解析器将是模糊不清的。然而，因为[开始结尾表达式，在表达式之后[不是一个可以安全排除的字符。如果[,]被后面的rust版本接受，这个匹配将会打破工作代码失败或者有歧义。像$i:expr或者$i:expr;的适配器将不合法，但是，因为,和;是合法的表达式分隔符。指定的规则有：
```

- expr和stmt只能被紧跟在下面之一：=>, ,, or ;.
- pat_param只能紧跟在下面之一：=>, ,, =, |, if, or in.
- pat只能紧跟在下面之一：=>, ,, =, if, or in.
- path和ty只能紧跟在下面之一：=>, ,, =, |, ;, :, >, >>, [, {, as, where,或者block变量指定的宏片段
- vis只能被紧跟在下面之一：,一个非原始priv标识，任何以type开始的标记，或者ident,ty,path元变量指定片段
- 所有其他没有限制的片段

当被重复调用，规则适用于所有可能的展开，考虑分隔符。这意味着：

- 如果重复包含了分隔符，那个分隔符必须紧跟在重复的上下文后
- 如果重复可以有多次(*或+)，上下文必须在它们之后
- 重复的上下文必须跟上以前的内容，无论之后发生什么，都必须能跟上重复的内容
- 如果重复能匹配0次(*或?)，无论后面是什么，都要能匹配前面的

# 过程宏

[Procedural Macros](https://doc.rust-lang.org/nightly/reference/procedural-macros.html)

过程宏允许像执行函数一样创建语法扩展。有三种方法使用过程宏：

- 类函数宏 - custom!(...)
- 推导宏 - #[derive(CustomDerive)]
- 属性宏 - #[CustomAttribute]

过程宏允许你在编译器使用rust语法运行代码，无论是消费还是生产语法。你可以想象成过程宏是从一个抽象语法树转换到另外一个抽象语法树。

过程宏必须在crate中定义为proc-macro类型。

```toml
[lib]
proc-macro = true
```

作为函数，它们必须返回语法，恐慌或者无休止的循环。返回语法依赖于各种类型的过程宏的替换或者添加。恐慌是编译器产生并返回一个编译错误。无限循环不会被编译器捕获，将会挂起整个编译器。

过程宏在编译期运行，并且和编译器共享相同的资源。例如，标准输入、输出、错误和编译器访问的相同。类似的，文件访问也是相同。正如这些，过程宏和cargo编译脚本拥有相同的安全性。

过程宏有两种方式来报告错误。第一种是panic，第二种是发送compile_error的宏调用。

## proco_macro trait

过程宏通常被链接到proc_macro crate。proc_macro crate提供了需要写入过程宏的类型使得使用起来更容易。

这个crate主要包含了TokenStream类型。过程宏运行token streams来代替抽象语法树节点，随着时间的推移，编译器和过程宏对目标接口更加稳定。一个token stream大约等同于Vec<TokenTree>，TokenTree大概可以理解成文本标记。例如foo是一个Ident标记，.是一个Punct标记，1.2是一个文本标记。Tokenstream类型，不同于Vec<TokenTree>，可以更廉价的clone。

所有标记都要指定的Span。一个Span不能被修改但是可以批量产生。Span S描述了程序中代码的扩展，主要被用来错误报告。当你不能修改Span自身，可以修改有关的标记，例如通过另外一个标记来获取Span。

## 过程宏事项

过程宏是不健康的。这意味着如果output token stream被简单写入到内联代码中。这意味着被外部元素影响着并且影响外部导入。

宏作者需要非常小心确保他们的宏可以大多数限制的上下文。这需要使用相对路径包含库中的元素，或者确保生成函数可以和其他函数很好区分。

## 类函数过程宏

类函数过程宏可以使用宏调用操作符(!)来调用。

那些宏可以使用proc_macro属性来定义公共函数，签名是(TokenStream) -> TokenStream。输入的TokenStream在调用的分隔符内，输出的TokenStream会替换整个调用。

例如，下面的函数在它的作用域内忽略了输入输出answer：

```rust
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro]
pub fn make_answer(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}
```

我们可以在crate中使用它打印42：

```rust
extern crate proc_macro_examples;
use proc_macro_examples::make_answer;

make_answer!();

fn main() {
    println!("{}", answer());
}
```

类函数过程宏可能在宏调用的任意地方调用，包含声明，表达式，模式匹配，类型表达式，元素位置，通过extern块包裹，继承和实现trait，和trait定义。

## Derive宏

Derive macros定义了derive属性的输入。那些宏可以通过给定的struct,enum,union创建元素。他们也可以定义derive macro helper attributes。

用户派生宏可以使用proc_macro_derive属性和(TokenStream) -> TokenStream来定义。

输入的TokenStream是包含derive属性的元素标记流。输出TokenStream必须是输入流中一系列添加到模块或者块的元素。

下面是派生宏的示例。它没有做任何事情，只是添加了一个函数answer:

```rust
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro_derive(AnswerFn)]
pub fn derive_answer_fn(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}
```

接着使用了派生宏：

```rust
extern crate proc_macro_examples;
use proc_macro_examples::AnswerFn;

#[derive(AnswerFn)]
struct Struct;

fn main() {
    assert_eq!(42, answer());
}
```

## 派生宏属性帮助

派生宏可以在其元素的作用域内添加属性。所说的属性是派生宏的帮助属性。这些属性是惰性的，他们的唯一目的是提供给定义他们的派生宏。意味着，可以被所用宏发现。

定义帮助属性的方式是将属性关键字放在proc_macro_derive宏中用逗号隔开各自的帮助属性名称。

例如下面的派生宏定义了帮助属性helper，最终没有做任何事：

```rust
#[proc_macro_derive(HelperAttr, attributes(helper))]
pub fn derive_helper_attr(_item: TokenStream) -> TokenStream {
    TokenStream::new()
}
```

在struct的派生宏中使用：

```rust
#[derive(HelperAttr)]
struct Struct {
    #[helper] field: ()
}
```

## 属性宏

属性宏定义了外部属性，可以被附加到元素，extern block包括的元素，继承和实现trait，和trait定义。

属性宏使用proc_macro_attribute属性定义为公共函数，签名为(TokenStream, TokenStream) -> TokenStream。第一个TokenStream紧跟着属性名称的分割标识树，不包含外部分割。如果属性被写成空的名称，TokenStream属性就是空的。第二个TokenStream包含属性的剩余的元素。返回的TokenStream替换任意数量元素。

例如，这个属性宏获取输入流然后返回它，有效的成为属性的开始。

```rust
#[proc_macro_attribute]
pub fn return_as_is(_attr: TokenStream, item: TokenStream) -> TokenStream {
    item
}
```

下面示例展示了字符串化TokenStream S。将会在编译器看到输出。输出将出现在"out:"前缀的注释后：

```rust
// my-macro/src/lib.rs

#[proc_macro_attribute]
pub fn show_streams(attr: TokenStream, item: TokenStream) -> TokenStream {
    println!("attr: \"{}\"", attr.to_string());
    println!("item: \"{}\"", item.to_string());
    item
}
```

```rust
// src/lib.rs
extern crate my_macro;

use my_macro::show_streams;

// Example: Basic function
#[show_streams]
fn invoke1() {}
// out: attr: ""
// out: item: "fn invoke1() { }"

// Example: Attribute with input
#[show_streams(bar)]
fn invoke2() {}
// out: attr: "bar"
// out: item: "fn invoke2() {}"

// Example: Multiple tokens in the input
#[show_streams(multiple => tokens)]
fn invoke3() {}
// out: attr: "multiple => tokens"
// out: item: "fn invoke3() {}"

// Example:
#[show_streams { delimiters }]
fn invoke4() {}
// out: attr: "delimiters"
// out: item: "fn invoke4() {}"
```

## 声明宏标记和过程宏标记

声明macro_rules宏和过程宏类似，但是标记有所不同。

macro_rules中的标记树定义如下：

- 分割分组((...), {...}, etc)
- 所有语言支持的操作，包括单字符和多字符(+, +=)，注意不包含单引号'
- 字符串，注意负的(e.g. -1)不是字符串的一部分，是分割操作标识
- 标识符，包括关键字(ident, r#ident, fn)
- 生命周期 ('ident)
- macro_rules中代替元变量(e.g. $my_expr in macro_rules! mac { ($my_expr: expr) => { $my_expr } }，在mac扩展后，会被识别成单个标识树

过程宏中标识树定义如下：

- 分割分组((...), {...}, etc)
- 语言支持的所有标点符号(+, but not +=)，切包含单引号'
- 字符串，注意(e.g. -1)支持整数的一部分，浮点的字符串
- 标识符，包括关键字(ident, r#ident, fn)

考虑到token streams传出或传入给过程宏，这两个定义的不匹配。

注意下面转换会惰性发生，如果标记没被检查是不会发生。

什么时候传入proc-macro：

- 所有多字符标识并入单字符
- '字符标识的生命周期
- 所有元变量用下面的token stream代替：当需要保留解析优先级，这样的token stream可以使用显示的分隔符(Delimiter::None)包装成分割的组(Group)。tt和ident转换从来没有包装到这个组，并且经常被描述为潜在的标识树。

当使用proc触发宏：

- 标点符号在使用的情况下合入多字符操作符
- 单引号标识连接的生命周期
- 当需要保留解析优先级，负的常量被转换到两个标识，可用隐式分隔符将他们分成组

注意，声明宏和过程宏都不支持文档注释 (e.g. /// Doc)。所以他们经常转换成#[doc = r"str"]属性来转换成token stream。
