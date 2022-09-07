# Rust过程宏3

这一章节，我们会使用syn和quote两个crate来完成一个简单的命令行工具的编写，其中使用过程宏来完成主要的逻辑。

第一步先引入toml配置，除了上面两个crate之外，还用了proc-macro2来使过程宏更易用。

```toml
[lib]
proc-macro = true

[dependencies]
proc-macro2 = "1"
syn = { version = "1", features = ["extra-traits"] }
quote = "1"
```

引入了相应的crate后，我们需要定义一个proc_macro_derive宏，用来生成Builder的trait。这个trait提供了derive方法，可以将一个输入的TokenStream转换成输出的TokenStream。通过使用parse_macro_input宏可以生成一个AST，然后我们自己将AST通过我们自己定义的BuilderContext来处理我们需要用到的信息。

lib.rs代码如下：

```rust
mod builder;

use builder::BuilderContext;
use proc_macro::TokenStream;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(Builder)]
pub fn derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let context = BuilderContext::new(input);
    context.generate().into()
}
```

BuilderContext是我们定义的struct，其中包含了name和fields两个成员。创建BuilderContext对象时，name可以使用input.ident来获取，而fields使用了rust中的if let语法，非常巧妙的使用syn的一系列结构获取到了能匹配到的input.data。

generate方法将BuildContext对象分别按照builder_name, optionized_fields, methods, assigns来分别处理生成相应的代码，最后通过quote宏将生成的代码组织成一套完整的代码。

```rust
use proc_macro2::{Ident, TokenStream};
use quote::quote;
use std::iter::Map;
use syn::{
    punctuated::{Iter, Punctuated},
    token::Comma,
    Data, DataStruct, DeriveInput, Field, Fields, FieldsNamed,
};

type TokenStreamIter<'a> = Map<Iter<'a, Field>, fn(&'a Field) -> TokenStream>;

pub struct BuilderContext {
    name: Ident,
    fields: Punctuated<Field, Comma>,
}

impl BuilderContext {
    pub fn new(input: DeriveInput) -> Self {
        let name = input.ident;
        let fields = if let Data::Struct(DataStruct {
            fields: Fields::Named(FieldsNamed { named, .. }),
            ..
        }) = input.data
        {
            named
        } else {
            panic!("Unsupported data type");
        };

        Self { name, fields }
    }

    pub fn generate(&self) -> TokenStream {
        let name = &self.name;
        // builder name: {}Builder, e.g. CommandBuilder
        let builder_name = Ident::new(&format!("{}Builder", name), name.span());
        // optional fields, e.g. executable: String -> executable: Option<String>
        let optionized_fields = self.gen_optionized_fields();
        // methods: fn executable(mut self, v: impl Into<String>) -> Self { self.executable = Some(v); self }
        // Command::builder().executable("hello").args(vec![]).finish()
        let methods = self.gen_methods();
        // assign Builder fields back to original struct fields
        // field_name: self.#field_name.take().ok_or(" xxx need to be set!")
        let assigns = self.gen_assigns();

        quote! {
            /// Builder structure
            #[derive(Debug, Default)]
            struct #builder_name {
                #(#optionized_fields,)*
            }

            impl #builder_name {
                #(#methods)*
                pub fn finish(mut self) -> Result<#name, &'static str> {
                    Ok(#name {
                        #(#assigns,)*
                    })
                }
            }



            impl #name {
                fn builder() -> #builder_name {
                    Default::default()
                }
            }
        }
    }

    fn gen_optionized_fields(&self) -> TokenStreamIter {
        self.fields.iter().map(|f| {
            let ty = &f.ty;
            let name = &f.ident;
            quote! { #name: std::option::Option<#ty> }
        })
    }

    fn gen_methods(&self) -> TokenStreamIter {
        self.fields.iter().map(|f| {
            let ty = &f.ty;
            let name = &f.ident;
            quote! {
                pub fn #name(mut self, v: impl Into<#ty>) -> Self {
                    self.#name = Some(v.into());
                    self
                }
            }
        })
    }

    fn gen_assigns(&self) -> TokenStreamIter {
        self.fields.iter().map(|f| {
            let name = &f.ident;
            quote! {
                #name: self.#name.take().ok_or(concat!(stringify!(#name), " needs to be set!"))?
            }
        })
    }
}
```

通过100行左右的代码，我们就可以实现通过定义#[derive(Builder)]自动生成相应的代码块来使用。同样，创建examples/command.rs来使用：

```rust
use builder::Builder;

#[allow(dead_code)]
#[derive(Debug, Builder)]
struct Command {
    executable: String,
    args: Vec<String>,
    env: Vec<String>,
    current_dir: String,
}

fn main() {
    let command = Command::builder()
        .executable("find")
        .args(vec!["-c".into(), "-vvv".into()])
        .env(vec![])
        .current_dir("/Users/tchen/arena/rust/proc_macros")
        .finish()
        .unwrap();

    println!("{:?}", command);
}
```

最红成功解析到参数：

```bash
$ cargo run --example command
    Blocking waiting for file lock on build directory
   Compiling builder v0.1.0 (E:\SourceCode\RustProject\RustPractice\macro_demo\builder)
    Finished dev [unoptimized + debuginfo] target(s) in 1m 12s
     Running `E:\SourceCode\RustProject\RustPractice\target\debug\examples\command.exe`
Command { executable: "find", args: ["-c", "-vvv"], env: [], current_dir: "/Users/tchen/arena/rust/proc_macros" }
```
