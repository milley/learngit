# Rust自定义Release下的构建配置

最近在看crm的源码，在Rust的Cargo.toml配置中，可能会存在profile.release来定义编译的部分参数。通过release profile可以预定义和可定制不同配置的参数，它允许程序员在编译代码时可以控制更多的选项。每个配置都独立于其他的配置。

Cargo有两个主要的配置选项：dev和release。通过cargo build会调用dev的配置选项，通过cargo build --release会调用release的配置选项。

在没有显示增加任何[profile.*]节点时Cargo会有默认的设置。增加[profile.*]节点会相应的覆盖默认的设置。通过[profile](https://doc.rust-lang.org/cargo/reference/profiles.html)来查看所有的配置信息。

下面列出crm的几种配置：

## opt-level

控制优化等级，更高等级的优化通过牺牲更长的编译时间来产生更快的运行时代码。高等级优化会修改或重新排列编译代码，可能会导致代码调试起来更困难。

允许的选项有：

- 0: 不优化
- 1: 基本优化
- 2: 更多优化
- 3: 全部优化
- "s": 优化二进制大小
- "z": 优化二进制大小，但是会关闭循环向量

建议通过实验不同等级来确定最终合适的优化选项。

## incremental

控制-C incremental flag选项，可以打开或者关闭incremental编译模式。incremental构建导致rustc保存额外信息到磁盘，在编译crate时可以用来复用，可以提升重新编译时间。额外信息保存到target路径中。

允许的选项：

- true: 打开
- false: 关闭

Incremental构建模式仅仅用在workspace成员和"path"依赖项。incremental值可以被全局的CARGO_INCREMENTAL环境变量或者build.incremental配置变量覆盖。

## codegen-units

控制-C codegen-units flag选项，标识一个crate将会分割到几个"code generation units"。更多的code generation units允许crate尽可能的并行处理来减少编译时间，但是会生成慢的代码。

选项值应该大于0。incremental构建默认的选项是256，其他选项默认是16。

## lto

控制-C lto flag选项，控制LLVM的链接时间优化。LTO可以，使用整个程序分析，在链接最耗时的地方生成更高效的代码。

允许的选项有：

- false: 执行"thin local LTO"，仅仅用本地crate自身的"codegen utils"来执行细微的"LTO"。如果codegen utils是1或者opt-level是0则不执行LTO。
- true or "fat": 执行"fat" LTO尝试在依赖树中优化所有的crate。
- "thin": 执行"thin" LTO，有点类似于"fat"，在获得类似于"fat"的执行效率同时大大的减少时间。
- "off": 关闭LTO。

## panic

控制-C panic flag选项，标识使用哪种panic策略。

允许的选项有：

- "unwind": 放松栈上的panic
- "abort": panic后结束流程

如果设置为"unwind"，依赖于目标平台的默认配置。例如，NVPTX平台不支持unwind因此只能是abort。

测试，基准，构建脚本，过程宏都会忽略panic设置。rustc测试会普遍依赖unwind行为。

另外，当使用abort策略然后构建一个test，所有的依赖会强制使用unwind策略进行构建。
