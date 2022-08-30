# Rust过程宏2

通过学习陈天老师的[Rust 过程宏 (第一弹)](https://b23.tv/UINce2o)，记录下如何使用json配置来生成相应的struct代码。

首先，创建文件fixtures/person1.json，描述了Person结构体：

```json
{
    "title": "Person",
    "type": "object",
    "properties": {
        "firstName": {
            "type": "string",
            "description": "first name"
        },
        "lastName": {
            "type": "string",
            "description": "last name"
        }
    }
}
```

针对输入的json，我们把其描述为Schema结构，输出结构描述为St结构，其中包含了Fd列表，创建文件src/json_schema.rs：

```rust
/// output structure
pub struct St {
    /// structure name
    name: String,
    /// a list of structure fields
    fields: Vec<Fd>,
}

pub struct Fd {
    name: String,
    ty: String,
}

impl St {
    pub fn new(name: impl Into<String>, fields: Vec<Fd>) -> Self {
        Self {
            name: name.into(),
            fields,
        }
    }
}

impl Fd {
    pub fn new(name: impl Into<String>, ty: impl Into<String>) -> Self {
        Self {
            name: name.into(),
            ty: ty.into(),
        }
    }
}
```

Schema实现中，into_vec_st函数将其转换为Vec<St>的结构，在转换内部又递归调用了process_type来处理名称和类型:

```rust
impl Schema {
    pub fn into_vec_st(&self) -> Vec<St> {
        let mut structs = vec![];
        match self.ty.as_str() {
            "object" => {
                let fields: Vec<_> = self.properties.as_ref().unwrap()
                    .iter()
                    .map(|(k, v)| process_type(&mut structs, k.as_str(), v))
                    .collect();
                structs.push(St::new(p(self.title.as_ref().unwrap()), fields));
                structs
            },
            _ => panic!("Not supported yet"),
        }
    }
}

fn process_type(structs: &mut Vec<St>, k: &str, v: &Schema) -> Fd {
    let name = n(k);
    match v.ty.as_str() {
        "object" => {
            // need to create a new St, field type is the St name
            let sts = v.into_vec_st();
            structs.extend(sts);
            Fd::new(name, gen_name(v.title.as_deref(), k))
        },
        "integer" => Fd::new(name, "i64"),
        "float" => Fd::new(name, "f64"),
        "string" => Fd::new(name, "String"),
        v => panic!("Unsupported schema type: {}", v),
    }
}

// pascal case
fn p(s: &str) -> String {
    AsPascalCase(s).to_string()
}

// snake case
fn n(s: &str) -> String {
    AsSnakeCase(s).to_string()
}

// generate name
fn gen_name(first: Option<&str>, second: &str) -> String {
    p(first.unwrap_or(second))
}
```

其中p和n都是字符串辅助函数，调用了heck这个crate。gen_name则是生成name字段。

通过以下2个unit test来验证是否正确：

```rust
#[test]
fn schema_should_be_converted_to_st() {
    let schema: Schema = serde_json::from_str(PERSON1).unwrap();
    let mut structs = schema.into_vec_st();
    assert_eq!(structs.len(), 1);
    let st = structs.pop().unwrap();
    assert_eq!(st.name, "Person");
    assert_eq!(st.fields.len(), 2);
    let mut names = st.fields.iter().map(|f| f.name.clone()).collect::<Vec<_>>();
    names.sort();
    assert_eq!(&names[..], &["first_name", "last_name"]);
    assert_eq!(st.fields[0].ty, "String");
    assert_eq!(st.fields[1].ty, "String");
}

#[test]
fn schema_with_nested_should_be_converted_to_st() {
    let schema: Schema = serde_json::from_str(PERSON2).unwrap();
    let structs = schema.into_vec_st();
    assert_eq!(structs.len(), 2);
}
```

下一步，需要找到一个类似模板引擎来构造我们的struct生成代码。这里使用了askama这个crate，通过介绍得知此模板引擎基于Jinja来处理。可以通过模板在编译时来生成Rust代码。

```rust
use serde::{Serialize, Deserialize};

use std::{fs, collections::HashMap};
use heck::{AsPascalCase, AsSnakeCase};
use askama::Template;
use anyhow::{Result, anyhow};
use proc_macro::TokenStream;
use litrs::Literal;

pub fn get_string_literal(input: TokenStream) -> Result<String> {
    input.into_iter().next()
        .and_then(|v| Literal::try_from(v).ok())
        .and_then(|v| match v {
            Literal::String(s) => Some(s.value().to_string()),
            _ => None,
        })
        .ok_or_else(|| anyhow!("Only string literal are allowed"))
}

#[derive(Template)]
#[template(path = "code.j2")]
pub struct StructsTemplate {
    structs: Vec<St>,
}

impl StructsTemplate {
    fn try_new(filename: &str) -> Result<Self> {
        println!("{}", filename);
        let content = fs::read_to_string(filename)?;
        println!("{}", content);
        
        let schema: Schema = serde_json::from_str(&content)?;
        Ok(Self{
            structs: schema.into_vec_st(),
        })
    }

    pub fn render(filename: &str) -> Result<String> {
        let template = Self::try_new(filename)?;
        Ok(template.render()?)
    }
}
```

定义了StructsTemplate这个结构体，并且实现了render函数，通过传入一个字符串切片来生成代码。

通过下面的单元测试验证正确性：

Person2.json内容：

```json
{
    "title": "Person",
    "type": "object",
    "properties": {
        "firstName": {
            "type": "string",
            "description": "first name"
        },
        "lastName": {
            "type": "string",
            "description": "last name"
        },
        "skill": {
            "type": "object",
            "title": "skill",
            "properties": {
                "name": {
                    "type": "string"
                }
            }
        }
    }
}
```

code.j2内容：

```j2
{% for st in structs %}
#[derive(Debug, Default, serde::Serialize, serde::Deserialize)]
pub struct {{ st.name }} {
    {% for field in st.fields %}
        pub {{ field.name }}: {{ field.ty }},
    {% endfor %}
}
{% endfor %}
```

单元测试代码：

```rust
#[test]
fn schema_render_should_work() {
    let result = StructsTemplate::render("fixtures/person2.json").unwrap();
    println!("{:#?}", result);
}
```

lib.rs增加如下内容：

```rust
mod json_schema;

use proc_macro::TokenStream;
use json_schema::{StructsTemplate, get_string_literal};

#[proc_macro]
pub fn generate(input: TokenStream) -> TokenStream {
    let filename = get_string_literal(input).unwrap();
    //println!("filename: {}", filename);
    let result = StructsTemplate::render(&filename).unwrap();
    //println!("result: {}", result);
    result.parse().unwrap()
}
```

最后，创建examples/generate.rs来使用我们的生成代码：

```rust
pub mod generated {
    use basic::generate;
    generate!("macro_demo/basic/fixtures/person.json");
}

use generated::*;

use serde::{Serialize, Deserialize};
use std::collections::HashMap;

#[derive(Debug, Default, Serialize, Deserialize)]
pub struct Schema {
    title: Option<String>,
    #[serde(rename = "type")]
    ty: String,
    properties: Option<HashMap<String, Schema>>,
}

fn main() {
    // let data: Schema = serde_json::from_str(include_str!("../fixtures/person.json")).unwrap();
    // println!("{:?}", data);

    let person = Person {
        first_name: "Tyr".into(),
        last_name: "Chen".into(),
        skill: Skill {
            name: "Rust".into(),
        },
    };

    println!("{:?}", person);
}
```

运行generate.rs：

```command
$ cargo run --example generate
    Finished dev [unoptimized + debuginfo] target(s) in 0.23s
     Running `E:\SourceCode\RustProject\RustPractice\target\debug\examples\generate.exe`
Person { first_name: "Tyr", last_name: "Chen", skill: Skill { name: "Rust" } }
```

说明已经可以正常生成我们所需的Person代码。
