# 为什么Golang是我最喜欢的编程语言

[Why Go is my favorite programming language](https://michael.stapelberg.ch/posts/2017-08-19-golang_favorite/#getting-started)

我努力尊重每个人的个人喜好，所以我经常避开辩论哪个是最好的编程语言、文本编辑器或操作系统。

然而，最近最近有很多次被问到为什么大量使用Go以及原因，所以这是一个连续的文章来填补特别是我个人的一些拙见:-).

## 我的背景

我使用C和Perl做了一些看起来还比较体面的项目。我写过Python，Ruby，C++，CHICKEN Scheme，Emacs Lisp，Rust和Java(仅仅是Android)。我了解一些Lua，PHP，Erlang和Haskell。在早些时间，我用Delphi开发了不少项目。

我在2009年简短的看过Go，当时是第一版Release。我正式使用Go是在2012年Go 1.0已经有了Release版本，实践了[Go 1 兼容性保障](https://golang.org/doc/go1compat)。现在仍然有我2012年写的代码跑在生产环境上，几乎没有改动过。

### 1. 清晰

#### 格式化

Go代码，约定俗成，使用gofmt进行格式化。格式化代码不是新鲜的场景，但是相反的，gofmt恰恰支持一种规范的格式。

将所有代码按一种格式格式化有助于更好的阅读代码；会感觉到代码更熟悉。这不仅对阅读Go编译器或者标准库有利，同时当参与大型项目--开源项目或者大公司项目。

此外，自动格式化在代码走查能节省大量时间，因为它消除了评审前整个代码的分歧。现在你可以让你的持续集成系统验证gofmt不产生差异。

更有趣的是，我的编辑器在保存文件自动调用gofmt，改变了我写代码的方式。我曾经尝试匹配格式化程序将执行什么，然后改正我的错误。如今，我尽快来表达我的想法和新人gofmt能将它变的漂亮。

#### 高质量代码

我使用大量的标准库，如下。

迄今我读过所有的标准库，代码质量都非常高。

一个例子是[image/jpeg](https://golang.org/pkg/image/jpeg/)包：我不知道JPEG如何工作，但是它很容易的从[Wikipedia JPEG article](https://en.wikipedia.org/wiki/JPEG)和[image/jpeg](https://golang.org/pkg/image/jpeg/)重新捡起来。如果包中有些许注释，我会说它是一个教学的实现。

#### 见解

我非常赞同Go社区的一些见解，例如：

- 默认情况下变量名应该简短，从它的声明更多的名字被使用，更进一步应该变得更具有描述性。
- 保持依赖树更小：[a little copying is better than a little dependency](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=9m28s)。
- 引入抽象层是有成本的。Go代码通常非常清晰，代价是有时候会有一些重复代码。
- 更多查看[CodeReviewComments](https://github.com/golang/go/wiki/CodeReviewComments)和[Go Proverbs](https://go-proverbs.github.io/)。

#### 更少的关键字和抽象层

Go只有仅仅25个关键字，所有的都可以容易的记在脑子里。

同样[内置函数](https://golang.org/pkg/builtin/)和[类型](https://golang.org/ref/spec#Types)。

在我的经验中，更少的抽象层和上下文的语言可以容易上手和更快的感到舒适。

当我们讨论这些，我很惊讶如何读[Go specification](https://golang.org/ref/spec)。它看起来更像是给目标程序员(相比起标准库提交者)。

### 2. 速度

#### 快速反馈低延迟

我喜欢快速反馈：我很赏识加载快速的网站，我喜欢流畅不滞后的用户界面，在以后任何一天相比起选择强大的工具我都会选择快速的工具。大型网站的调查结果证实，这都是很多行为共享的。

Go编译器的作者尊重我低延迟的意愿：编译速度对他们很重要，在是否会减慢编译速度的优化他们会仔细权衡。

我的一个朋友以前从来没使用过Go。在使用go get安装了[RobustIRC](https://robustirc.net/)后，他认为Go肯定是解释性语言然后我去给他们纠正：不，Go编译器就是这么快。

大多数的Go工具都不含有异常，例如gofmt和goimports是如此之快。

#### 最大化的资源利用

批处理应用(而不是交互性应用)，它们充分利用现有资源比低延迟更重要。利用所有可用的IPOS、网络带宽和计算，Go程序可以很容易的配置和使用。例如，我写了一个[filling a 1 Gbps link](https://people.debian.org/~stapelberg/2014/01/17/debmirror-rackspace.html)，然后优化[debiman](https://github.com/Debian/debiman/)利用所有可用资源，减少运行时间数小时。

### 3. 强大的标准库

Go标准库提供了有效通用的通讯协议和数据存储格式/机制，例如TCP/IP，HTTP，JPEG，SQL等等。

Go的标准库是我看过的所有标准库里面最好的。我认为它有良好的组织，干净，短小，更全面：我经常发现只使用标准库就可以编写出功能强大的应用，最多增加一两个额外的包。

特定域的数据结构和算法没有包含在标准库之中，例如[golang.org/x/net/html](https://godoc.org/golang.org/x/net/html)。golang.org/x命名空间提供了即将要加入到标准库中的新的类库。Go 1版本的兼容性保证排除了任何重大的更改，显然是值得的。最显著的例子就是golang.org/x/crypto/ssh，必须破坏现有的代码至[establish a more secure default](https://github.com/golang/crypto/commit/e4e2799dd7aab89f583e1d898300d96367750991)。

### 4. 工具

下载，编译，安装和升级Go包，我使用go get工具。

所有我使用的代码库使用内置的测试机制。这个结果不仅仅是简单、快速的测试，而且也生成可读的报告。

  每当程序使用比预期多的资源，我就会发动pprof。查看[golang.org blog post about pprof](https://blog.golang.org/profiling-go-programs)的介绍，或者[my blog post about optimizing Debian Code Search](https://people.debian.org/~stapelberg/2014/12/23/code-search-taming-the-latency-tail.html)。在导入pprof包后，你可以不用重新编译或重启来配置你的服务。

  交叉编译在设置了GOARCH环境变量后变得非常容易，例如GOARCH=arm64可以生成树莓派3目标程序。注意，工具也会跨平台工作。例如，我可以在amd64电脑配置gokrazy：go tool pprof ~/go/bin/linux_arm64/dhcp <http://gokrazy:3112/debug/pprof/heap>。

  godoc通过HTTP可以显示成超文本格式文档。<godoc.org>是一个公共的实例，但是我一般在离线或者没有发布包的情况下使用本地实例。

  请注意，这些是语言标准工具。使用C，完成以上每个都是一项壮举。使用Go，我们认为是理所当然的。

## 开始

  希望我能够说清楚我为什么喜欢用Go。

  如果你有兴趣使用Go，查看[the beginner’s resources](https://github.com/gopheracademy/gopher/blob/1cdbcd9fc3ba58efd628d4a6a552befc8e3912be/bot/bot.go#L516)我们指出了如何加入Go开发者的slack频道。查看<https://golang.org/help/>。

## 警告

当然，没有编程工具是完全没有问题的。通过这个文章解释了我为什么喜欢Go语言，它的重点是积极地。我将会提到几个问题：

- 如果你使用的包没有稳定的API，你可能还需要一种特定的，可以正常工作的版本。你最好的选择是dep工具，在写这篇文章的时候还不是语言内置的工具。
- 习惯于Go代码不一定能转换为高性能的机器码，和运行时最小消耗。在极少数情况下我发现性能缺失，我成功的用cgo或者汇编解决了。如果你的场景是强实时应用或及其性能关键的代码，你的经历可能会有所不同。
- 我说到过Go标准库是我看到的所有标准库中最好的，但不是说没有一点问题。一个例子就是当通过旧的标准库包修改Go代码就会[complicated handling of comments](https://golang.org/issues/20744)。
