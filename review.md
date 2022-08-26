# Python排序字典

[Python | Sort a Dictionary](https://www.geeksforgeeks.org/python-sort-a-dictionary/)

Python，给定一个字典，支持根据键和值来排序。[Python版本>=3.6v]

```sh
Input : test_dict = {“Gfg” : 5, “is” : 7, “Best” : 2}
Output : {‘Best’: 2, ‘Gfg’: 5, ‘is’: 7}, {‘is’: 7, ‘Gfg’: 5, ‘Best’: 2}
Explanation : Sorted by keys, in ascending and reverse order.

Input : test_dict = {“Best” : 2, “for” : 9, “geeks” : 8}
Output : {‘Best’: 2, ‘Gfg’: 5, ‘for’: 9}, {‘for’: 9, ‘geeks’: 8, ‘Best’: 2}
Explanation : Sorted by values, in ascending and reverse order.
```

## 使用键排序

使用sordted可以完成这个，我们使用第一个元素来指定键，字典的元素可以通过items()方法返回，然后传入自定义的lambda函数获取排序后的键。"reverse=True"可以逆序排序。

### Python3

```python3
# Python3 code to demonstrate working of 
# Sort a Dictionary
# Sort by Keys
  
# initializing dictionary
test_dict = {"Gfg" : 5, "is" : 7, "Best" : 2, "for" : 9, "geeks" : 8}
  
# printing original dictionary
print("The original dictionary is : " + str(test_dict))
  
# using items() to get all items 
# lambda function is passed in key to perform sort by key 
res = {key: val for key, val in sorted(test_dict.items(), key = lambda ele: ele[0])}
  
# printing result 
print("Result dictionary sorted by keys : " + str(res)) 
  
# using items() to get all items 
# lambda function is passed in key to perform sort by key 
# adding "reversed = True" for reversed order
res = {key: val for key, val in sorted(test_dict.items(), key = lambda ele: ele[0], reverse = True)}
  
# printing result 
print("Result dictionary sorted by keys ( in reversed order ) : " + str(res)) 
```

### Output

```sh
The original dictionary is : {'Gfg': 5, 'is': 7, 'Best': 2, 'for': 9, 'geeks': 8}
Result dictionary sorted by keys : {'Best': 2, 'Gfg': 5, 'for': 9, 'geeks': 8, 'is': 7}
Result dictionary sorted by keys ( in reversed order ) : {'is': 7, 'geeks': 8, 'for': 9, 'Gfg': 5, 'Best': 2}
```

## 使用值排序

和上面排序类似，不同地方仅仅在抽取的值，通过指定第二个元素来传入给比较器。

### Python3

```python3
# Python3 code to demonstrate working of 
# Sort a Dictionary
# Sort by Values 
  
# initializing dictionary
test_dict = {"Gfg" : 5, "is" : 7, "Best" : 2, "for" : 9, "geeks" : 8}
  
# printing original dictionary
print("The original dictionary is : " + str(test_dict))
  
# using items() to get all items 
# lambda function is passed in key to perform sort by key 
# passing 2nd element of items()
res = {key: val for key, val in sorted(test_dict.items(), key = lambda ele: ele[1])}
  
# printing result 
print("Result dictionary sorted by values : " + str(res)) 
  
# using items() to get all items 
# lambda function is passed in key to perform sort by key 
# passing 2nd element of items()
# adding "reversed = True" for reversed order
res = {key: val for key, val in sorted(test_dict.items(), key = lambda ele: ele[1], reverse = True)}
  
# printing result 
print("Result dictionary sorted by values ( in reversed order ) : " + str(res))
```

### Output

```sh
The original dictionary is : {'Gfg': 5, 'is': 7, 'Best': 2, 'for': 9, 'geeks': 8}
Result dictionary sorted by values : {'Best': 2, 'Gfg': 5, 'is': 7, 'geeks': 8, 'for': 9}
Result dictionary sorted by values ( in reversed order ) : {'for': 9, 'geeks': 8, 'is': 7, 'Gfg': 5, 'Best': 2}
```

# 我是如何学习Rust

[How I went about learning Rust](https://eli.thegreenplace.net/2022/how-i-went-about-learning-rust/)

很长一段时间我开始学习Rust，大概从一年前开始每周花费一些时间来学习它。

本文介绍我开始学习Rust的学习路径和细节，希望这可以帮助到更多的人。你将会注意到这不是一个"24小时学会X"的主题，它会花费很长一段时间并且覆盖了很多不同的素材。YMMV(Your Mileage May Vary)你这样的好处多多。

这个学习旅程通过代码来获取信息。在我的经验中，深入其他工具或学习一门语言都很关键，两者结合起来是最好的。

## 接收信息

我读了Rust指定的书籍，从前到后的顺序读。还有一些额外的数据在写代码章节，但是那不是Rust指定的。

[Programming Rust](https://www.oreilly.com/library/view/programming-rust-2nd/9781492052586/)是我读的第一本关于入门Rust的书。

[The Rust Programming Language](https://doc.rust-lang.org/book/)-我没有完整读了这本书，浏览了其中一些主要内容，实现了书中列出的一些项目。自从在Google排名靠前后，我常常将其作为参考书来使用。我个人觉得，作为参考书，它比Programming Rust更好。

[Rust in Action](https://www.manning.com/books/rust-in-action)

[Rust for Rustaceans](https://rust-for-rustaceans.com/)

除了以上书籍，我也看了John Gjengset在Youtube上的视频。它们非常有趣，在我需要学习高级主题的时候我也会去那里看。

最后，最近一年我也通过blog读了一些Rust博客，常常填满了HN的前端页面或r/rust。

## 写代码

[rustlings](https://github.com/rust-lang/rustlings)读写Rust代码的小练习片段。很不错的理念，虽然不是很全面。在刚开始入门时候是很好的选择。

[Advent of Code](https://adventofcode.com/)我喜欢AOC，2021版本是练习Rust代码很不错的选择。我已经用Rust实现了1-18天的解决方案，大概2000行代码。通过解决AOC问题来学习Rust编程是很好的方式。我很想用2022版本再解决AOC，来保持我的技能更熟练。

[Ray Tracer Challenge](https://pragprog.com/search/?q=the-ray-tracer-challenge)这本书在编程语言中不可不知，我选择Rust来实现了简单的光线跟踪器。另一个实践新语言的方式，总共4000行代码。

[Crafting Interpreters](https://book.douban.com/subject/35548379/)使用Rust实现了大多数的clox编译器和虚拟机，2000行代码。

通过以上这些，我完成了一系列实验，通过写代码了解了Rust的方方面面。这些描述在我的[Blog](https://eli.thegreenplace.net/tag/rust)中，但是大多数还没有。我这里描述的大概有6000行的代码。

以上，这几个月通过学习和练习总共编写了14000行Rust代码。

## 下一步

我不会将自己称作Rust专家，通过过去一年的学习我对编程语言的知识有了更好的理解，我有信心在将来使用Rust来构建生产项目。

总体来说，我喜欢Rust，很高兴将Rust添加到我的工具箱中。它有一些不足，我也有一些不同的观点，在将来会慢慢来的:-)

