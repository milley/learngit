# 类型和测试

[Types and Tests](https://blog.cleancoder.com/uncle-bob/2019/06/08/TestsAndTypes.html)

Mark Seeman (@ploeh)和我在twitter有一个有趣的争论。它从我发的这个推特开始，后来持续成了一系列的推特时间线：

> 这里不能忽略。无论你使用的静态还是动态类型，你都应该通过执行测试来证明正确性。静态类型不能省略那些测试，因为它们都是基于习惯和经验的。Uncle Bob Martin (@unclebobmartin) June 4, 2019

和往常一样，在社交网络上，一些人发布粗鲁的、侮辱的评论。那些人根本不重要。Mark，在另一边，用尊敬、实质性的回复，接下来可以看到整线。它是这样开始的：

> 我恭恭敬敬的不同意这个观点。一些类型系统允许null引用。在这些类型系统中，你必须编写测试才能证明正在验证的系统没有被null输入互相影响。
>
> 另一种类型系统(例如Haskell)不存在null。编写等价的测试会没有意义。--Mark Seemann (@ploeh) June 4, 2019

接着一场火热的争论开始了。接下来的各种线索你将会看到集中的、有教育意义的辩论。然而，我只想聚焦在Mark的回复上。他在tweet上发了一篇2018年他写的[博客](https://blog.ploeh.dk/2018/07/09/typing-and-testing-problem-23/)。我鼓励你去读一下。你将会学习到一些静态类型、测试和Haskell。你还将会学习到如何使用过去发布的反驳意见来和其他人争论。;-)

Mark在他的blog指出的问题是一个简单函数：rndselect(n,list)。它随机的从输入列表中选择n个元素返回一个列表。他使用Haskell来实现这个对策，使用类型，QuickCheck和事后测试。

这激起了我的兴趣。不同的使用一个动态函数式编程语言例如Clojure，用严格的TDD规则，过程和结果是什么样子。

让我们看一下。

我们用一个退化的测试开始。如果输入列表是空我们返回一个空列表，或者请求的元素个数是0。

```Clojure
(deftest random-element-selection
  (testing "degenerate case"
   (is (= [] (random-elements 0 [])))
   (is (= [] (random-elements 0 [1])))
   (is (= [] (random-elements 1 [])))
   ))

(defn random-elements [n xs]
  [])
```

早期在Mark的博客他记录了[免费定律](https://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf)的推论，就是他不需要测试结果列表中的元素来自源列表中的元素。另一方面，由于我没有使用静态类型系统，如果我的测试部假设结果集中的元素来自源列表，那么我的测试就没意义了。

这是一个琐碎的情况。让我们从列表中取一个元素：

```Clojure
(testing "trivial cases"
  (is (= [1] (random-elements 1 [1]))))

(defn random-elements [n xs]
  (if (or (< n 1) (empty? xs))
    []
    [(first xs)]))
```

注意我在遵循逐步提升复杂度的规则。比起来担心整个程序的随机片段问题，我首先聚焦在测试描述外围的问题。

我们称作：“不要去找金子”。尽可能的原理算法核心，逐步的提升测试难度。首先处理退化的、琐碎的、好管理的任务。

在那个指导下，下一个复杂点就是担心重复。所以我们可以测试下从单个元素列表中取更多的元素。

```Clojure
(testing "repetitive case"
    (is (= [1 1] (random-elements 2 [1]))))
	
(defn generate-indices [n]
  (repeat n 0))

(defn random-elements [n xs]
  (if (or (< n 1) (empty? xs))
    []
    (map #(nth xs %) (generate-indices n))))
```

你们中的一部分人可能会觉得我比起自身会领先一些。我用nth调用代替用first调用来保持。如果没有测试，我为什么要修改它？

无奈！

下一个要考虑的复杂事情就是那些索引。现在他们都是0。要确保除0之外其他的索引也能正常工作。测试这个强迫我需要mock函数来生成索引；这也会强迫我修改一点设计。所以我将会解散generate-indices函数然后从外部mock一个单独的索引。

陌生的with-bindings调用临时的替换index函数的实现，以便每次都返回下表1。奇怪的^:dynamic属性是必须的，在Clojure中如果你想模拟(重新绑定)一个函数。

```Clojure
(testing "singular random case"
  (with-bindings {#'index (fn [] 1)}
    (is (= [2] (random-elements 1 [1 2])))))
	
(testing "repeated random case"
    (with-bindings {#'index (fn [] 1)}
      (is (= [2 2] (random-elements 2 [1 2])))))

(defn ^:dynamic index []
  0)

(defn generate-indices [n]
  (repeatedly n index))

(defn random-elements [n xs]
  (if (or (< n 1) (empty? xs))
    []
    (map #(nth xs %) (generate-indices n))))
```

让我们检查rand-int已经被调用。这样做我们可以确保从[0 10 20 30]10个随机元素的和会大于0小于300。

```Clojure
(testing "random selection"
  (let [ns (random-elements 10 [0 10 20 30])
        sum (reduce + ns)]
    (is (< 0 sum 300))))
	
(defn ^:dynamic index [limit]
  (rand-int limit))

(defn generate-indices [n limit]
  (repeatedly n (partial index limit)))

(defn random-elements [n xs]
  (if (or (< n 1) (empty? xs))
    []
    (map #(nth xs %) (generate-indices n (count xs)))))
```

这个测试失败的概率大概是百万分之一。如果我认为有必要的话，可以适当提高。如果我想消除这些可能性，我可以写一个间谍来确保随机函数被正确地调用。但是我不认为这些都值得。

到目前为止我在测试中只使用了整数。大部分的测试并不关心元素类型。所以为什么不更改这些特定的测试以确保正确处理许多不同的类型呢。

```Clojure
(testing "trivial cases"
  (is (= [1] (random-elements 1 [1]))))

(testing "repetitive case"
  (is (= [:x :x] (random-elements 2 [:x]))))

(testing "singular random case"
  (with-bindings {#'index (fn [_] 1)}
    (is (= ['b'] (random-elements 1 ['a' 'b'])))))

(testing "repeated random case"
  (with-bindings {#'index (fn [_] 1)}
    (is (= ["two" "two"] (random-elements 2 ["one" "two"])))))
```

通过那些，我想我们已经做了。

在我的测试中有8个断言。然而，大部分的断言都是在TDD进程中被增加的。现在它们可以工作，有多少断言是真正需要的？可能仅仅是退化的情况和随机的最终测试。让我们调用那四个断言。处于文档目的我把所有的都留在这里，但是它们并不是绝对需要的。

无效参数怎么样？如果有人这样调用(random-elements -23 nil)又怎么样？我需要为它们写测试用例吗？

该函数已经通过返回一个空的列表来处理任何一个小于1的负数。这没有经过测试，但是代码很清晰。在nil的情况下，将抛出一个异常。这对我来说是可以的。这是一个动态语言。当你没有处理好类型时，会有异常。

这有危险吗？当然，只有当部分系统未经测试就完成编写。如果你从一个你编写过测试的模块来调用，使用与TDD规章一样有效的方法，你将不会传入nil或者负数，或者其他形式的无效参数。因此我不需要担心这些。就像你说的。

## 症结

这就是我和Mark争论的症结。我声称所需的测试数量只是描述系统的正确行为所需的测试。如果系统的每个元素的行为都是正确的，则系统的任何元素都不会将无效的元素传递给其他参数。无效的状态将无法表示。

当然没有一套测试可以证明系统是正确的。Dijkstra说最好是：“测试显示存在，而不是没有bug”。不过，我们至少要尝试下。因此，我们通过一系列全面的测试来阐述实际正确性。

我断言，无论系统是静态语言还是动态语言编写的，都需要一组测试来表明实际正确。

Mark断言，当你使用动态语言你需要更多的测试，因为静态类型检查使得无效状态无法呈现，因此仔细考虑后剔除它们。如果你有一个不能是nil的类型那么你就不需要编写测试来检查nil。

我的回答是，即使使用动态类型语言，我也不需要编写没有的测试，因为我知道永远不会通过。如果我知道那些状态永远无法表达，那我就不需要状态无法表达。

## 无论如何...

回到随机元素：

如果你比较我和Mark写的两个函数(如果你懂得Haskell，我不懂)我想你将会看到两个实现非常相似。然而，我们采用的测试策略完全不同。

我写的测试表明我几乎关心函数的期望行为。测试使用TDD格式一次完成一小步，使得向前一直发展。

但是Mark，更关心的是如何使用泛型类型来表达正确的因素。他绝大多数思维过程被正确代表性问题的类型结构给消耗了。然后他基于属性的QuickCheck样式测试和事后测试来验证正确行为。

哪个更好？

> 巴里克·奥巴马说过：“回答那个具体问题要在我的付费级别上”

我们中哪些人的测试结果较少？在这两种情况下，我认为四个基本断言是合适的。然而，作为过程的一部分，我谢了8个断言，Mark写了两个基于属性的QuickCheck测试，以及三个时候回归测试。这是否等于5 vs 8？或者，这些QuickCheck测试是否更多？我不知道，你来决定吧。
