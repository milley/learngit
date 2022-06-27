# 使用Rust爬取web

[Web Scraping with Rust](https://www.scrapingbee.com/blog/web-scraping-rust/)

比如说你想从一个网站获取一些信息，比如股票价格，最新的广告，或者最新的文章列表。最简单的方法就是连接到API。如果网站提供了免费的API，你可以获取到你想要的信息。

如果没有，常常还有一个方法，网络爬虫。

作为代替官方的资源，你可以使用机器人来爬取网站的内容然后解析出你想要的事情。

这个文章将介绍如何使用rust编程语言来实现web爬虫。你将会使用两个库，reqwest和scraper，从IMDb爬取TOP100的电影。

## 在Rust中实现网络爬虫

你将会使用Rust来实现一个完整功能的网络爬虫。你爬取的目标是IMDb，一个电影、电视剧等媒体数据库。

最后，你可以用Rust程序完成任何时候爬取TOP100级别的电影。

这个教程需要你首先安装Rust和Cargo。如果没有请参考官方文档。

## 创建工程并且添加依赖

开始之前，需要创建一个基本的Rust工程和添加需要用到的依赖。最好的方式是使用Cargo。

为Rust程序创建一个工程，使用：

```rust
cargo new web_scraper
```

下一步，添加依赖。这个工程将要用到的reqwest和scraper。

用你最喜欢的代码编辑器打开web_scraper工程然后打开cargo.toml。最后部分添加库：

```toml
[dependencies]

reqwest = {version = "0.11", features = ["blocking"]}
scraper = "0.12.0"
```

然后可以移到src/main.rs来创建web爬虫了。

## 获取网站HTML

爬取页面通常包含获取页面的HTML代码然后解析出你需要找的信息。因此，你需要在Rust程序中获取到IMDb页面。要做这些，首先要了解浏览器的工作原理，因为它将是获取页面最常用的手段。

在浏览器中显示一个页面，客户端需要发送一个HTTP请求给服务器，然后会返回web页面的源代码。浏览器随后渲染这个页面。

HTTP有一些不同类型的请求，例如GET和POST。使用Rust程序获取IMDb网站的页面的代码，需要模拟浏览器发送HTTP GET请求给IMDb。

在Rust中可以使用reqwest。此库通常可以提供HTTP客户端的特性。它可以像浏览器一样提供打开页面，登录，保存cookie等一系列操作。

请求一个web页面，可以使用reqwest::blocking::get方法：

```rust
fn main() {
    let response = reqwest::blocking::get(
        "https://www.imdb.com/search/title/?groups=top_100&sort=user_rating,desc&count=100",
    )
    .unwrap()
    .text()
    .unwrap();
}
```

response将会包含了所有的HTML代码。

## 从HTML中提取信息

爬虫工程最难的部分通常是从HTML文档中获取你需要的信息。为了达成这个目的，通常使用的库是scraper。它可以将HTML解析成树结构。可以使用CSS选择器来查询你感兴趣的元素。

第一步需要使用库解析整个HTML文档：

```rust
let document = scraper::Html::parse_document(&response);
```

下一步，查找和选择你需要的信息。为此你需要检查网站的代码，从中查找一系列CSS选择器然后选择唯一的条目。

最简单的方法是使用你的浏览器。找到你需要的元素，然后点击检查元素来获取元素。

就IMDb来说，你需要的元素是电影的名称。当你检查元素，你将会看到这样的标签：

```html
<a href="/title/tt0111161/?ref_=adv_li_tt">The Shawshank Redemption</a>
```

不幸的是，这个标签不是唯一的。因为页面中有很多<a>标签，如果爬取所有的将不是明智的行为，大部分也不是想要的。作为替代，可以查找唯一的电影标题标签然后导航到<a>标签中的那个标签。

在这个情况下，可以选择lister-item-header：

```html
<h3 class="lister-item-header">
    <span class="lister-item-index unbold text-primary">1.</span>
    <a href="/title/tt0111161/?ref_=adv_li_tt">The Shawshank Redemption</a>
    <span class="lister-item-year text-muted unbold">(1994)</span>
</h3>
```

现在你需要使用scraper::Selector::parse方法来创建一个查询。

你可以指定h3.lister-item-header>a选择器。换句话说，它找到<a>标签含有<h3>标签并且是lister-item-header样式。

使用如下的查询：

```rust
let title_selector = scraper::Selector::parse("h3.lister-item-header>a").unwrap();
```

现在你可以接收这个查询返回的文档。获取实际的电影标题来替代HTML元素，你可以映射每个HTML元素到内置的HTML：

```rust
let titles = document.select(&title_selector).map(|x| x.inner_html());
```

titles现在是获取到所有TOP100标题的迭代器。

现在你想做的是打印这些名字。为此，首先使用1-100压缩你的标题列表。然后调用for_each方法打印迭代器中每个元素。

```rust
titles
    .zip(1..101)
    .for_each(|(item, number)| println!("{}. {}", number, item));
```

现在你的爬虫就完成了。这里是完整的代码：

```rust
fn main() {
    let response = reqwest::blocking::get(
        "https://www.imdb.com/search/title/?groups=top_100&sort=user_rating,desc&count=100",
    )
    .unwrap()
    .text()
    .unwrap();

    let document = scraper::Html::parse_document(&response);

    let title_selector = scraper::Selector::parse("h3.lister-item-header>a").unwrap();

    let titles = document.select(&title_selector).map(|x| x.inner_html());

    titles
        .zip(1..101)
        .for_each(|(item, number)| println!("{}. {}", number, item));
}
```

如果你保存文件然后用cargo运行你将会得到TOP100的电影列表：

```txt
1. The Shawshank Redemption
2. The Godfather
3. The Dark Knight
4. The Lord of the Rings: The Return of the King
5. Schindler's List
6. The Godfather: Part II
7. 12 Angry Men
8. Pulp Fiction
9. Inception
10. The Lord of the Rings: The Two Towers
...
```

## 总结

这篇指南，使用Rust创建了一个简单的网络爬虫。Rust不是一个大众化的脚本语言，但是正如你所见，它可以非常容易的完整任务。

这只是一个入门网络爬虫的指南。有非常多的提升这个爬虫的方法，取决于你想做什么。

下面是一些可以尝试练习的选项：

- 将数据解析到自定义结构：你可以创建一个Rust结构保存电影数据。你将会更容易的在后面打印数据等。
- 将数据保存到文件中：使用文件存储到本地代替打印到控制台中
- 创建一个客户端记录IMDb账号：你可能希望在你解析之前IMDb显示你喜欢的电影。例如IMDb展示你所在国家最受欢迎的电影名称。如果这是个议题，你需要配置你的IMDb然后创建web爬虫登陆和爬取你的喜好。

然而，有的时候使用CSS选择器并不是一个很好的选择。你可能需要一个更高级的方案来模拟请求。你可以使用thirtyfour，Rust的UI测试框架，提供了更有效的爬虫功能。
