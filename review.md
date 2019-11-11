# Java8并发操作可没有它看起来那么简单

[Java 8 Parallel Operations Are Not As Simple As They Seem](https://www.bruceeckel.com/2016/04/27/java-8-parallel-operations-are-not-as-simple-as-they-seem/)

在探索流和并发流的不确定性，让我们看一个看起来很简单的问题：对一个自增序列的求和。事实证明有很多方法可以做到这一点，我也将冒着时间风险来比较它们--尽量小心谨慎，但我承认当执行时序代码很容易陷入基本的陷阱之内。结果会有一些瑕疵(比如没有预热JVM)，尽管如此它也提供了一些有用的指示。

我将以timeTest()方法开始计时，它获取一个LongSupplier，测量getAsLong()调用的时长，使用checkValue来比较结果并显示结果。它还将结果分配给volatile值，以防被Java引诱到优化了计算。

注意在任何时候，都尽可能严格使用long类型。在最重要的地方缺少long之前我花费了一些时间来找出溢出。

所有关于时间和内存的数字和讨论都涉及我的机器。你的经历可能和我的有所区别。

```java
// Summing.java
import java.util.stream.*;
import java.util.function.*;

public class Summing {
  static volatile long x;
  static void timeTest(String id, long checkValue,
    LongSupplier operation) {
    System.out.print(id + ": ");
    long t = System.currentTimeMillis();
    long result = operation.getAsLong();
    long duration = System.currentTimeMillis() - t;
    if(result == checkValue)
      System.out.println(duration + "ms");
    else
      System.out.format("result: %d\ncheckValue: %d\n",
        result, checkValue);
    x = result; // Prevent optimization
  }
  public static int SZ = 100_000_000;
  // This even works:
  // public static int SZ = 1_000_000_000;
  // Gauss's formula:
  public static final long CHECK =
    (long)SZ * ((long)SZ + 1)/2;
  public static void main(String[] args) {
    System.out.println(CHECK);
    timeTest("Sum Stream", CHECK, () ->
      LongStream.rangeClosed(0, SZ).sum());
    timeTest("Sum Stream Parallel", CHECK, () ->
      LongStream.rangeClosed(0, SZ).parallel().sum());
    timeTest("Sum Iterated", CHECK, () ->
      LongStream.iterate(0, i -> i + 1)
        .limit(SZ+1).sum());
    // Takes longer, runs out of memory above 1_000_000:
    // timeTest("Sum Iterated Parallel", CHECK, () ->
    //   LongStream.iterate(0, i -> i + 1)
    //     .parallel()
    //     .limit(SZ+1).sum());
  }
}
/* Output:
5000000050000000
Sum Stream: 625ms
Sum Stream Parallel: 158ms
Sum Iterated: 2521ms
*/
```

CHECK的值使用卡尔·弗里德里希·高斯在1700年末创造的公式来进行计算，在小学的时候我们也已经学习过。

第一个版本的main()使用简单的生成流的方法然后调用sum()来计算。我们看到使用流的福利就是虽然sz是一亿也没有溢出(我使用了一个小的数字所以程序没有运行太长时间)。使用了基本循环操作的parallel()显著提加快。

如果使用iterate()来生成序列速度会急剧下降，可能是因为每次生成一个数字都会调用lambda。但是如果我们试着并行，比起非并行版本不仅仅是耗时较长，当sz达到百万时就会消耗完内存。当然你在使用range()的时候不会使用iterator()，但是如果你生成一些简单序列你必须使用iterator()。尝试使用parallel()也是很合理，但是会生成奇怪的结果。我们可以对流并行算法进行一些初步的观察：

- 流并行将传入的数据分为多个部分，因此算法可以应用于这些单独的部分
- 数组可以很容易分割，拆分尺寸也可以很好知道
- 链表没有那些属性；拆分链表意味着分为第一个元素和其余元素，也是相当低效
- 无状态的产生者就像数组；上面range()的使用就是无状态的
- 迭代的产生者就像链表；iterator就是迭代的产生着

现在让我们试着给数组填入一些值来解决问题，然后再对数组求和。因为数组是一次性分配，看起来我们不太会遇到垃圾回收时机的问题。

首先我试着将原始的long类型填入数组：

```java
// Summing2.java
import java.util.*;

public class Summing2 {
  static long basicSum(long[] ia) {
    long sum = 0;
    final int sz = ia.length;
    for(int i = 0; i < sz; i++)
      sum += ia[i];
    return sum;
  }
  // Approximate largest value of SZ before
  // running out of memory on my machine:
  public static int SZ = 20_000_000;
  public static final long CHECK =
    (long)SZ * ((long)SZ + 1)/2;
  public static void main(String[] args) {
    System.out.println(CHECK);
    long[] la = new long[SZ+1];
    Arrays.parallelSetAll(la, i -> i);
    Summing.timeTest("Array Stream Sum", CHECK, () ->
      Arrays.stream(la).sum());
    Summing.timeTest("Parallel", CHECK, () ->
      Arrays.stream(la).parallel().sum());
    Summing.timeTest("Basic Sum", CHECK, () ->
      basicSum(la));
    // Destructive summation:
    Summing.timeTest("parallelPrefix", CHECK, () -> {
      Arrays.parallelPrefix(la, Long::sum);
      return la[la.length - 1];
    });
  }
}
/* Output:
200000010000000
Array Stream Sum: 114ms
Parallel: 27ms
Basic Sum: 33ms
parallelPrefix: 49ms
*/
```

第一个限制是内存大小；因为数组要先分配，我们不能比前一个版本创建的更大。并行可以提升速度，甚至比仅仅使用循环的basicSum()还快。比较有趣的是，Arrays.parallelPrefix()没有我们想象的提升那么快。然而，上面的技术在其他条件都非常有用--这也是为什么不能下确定性结论的原因，你必须尝试一下。

最后，考虑下使用包装类Long代替的影响：

```java
// Summing3.java
import java.util.*;

public class Summing3 {
  static long basicSum(Long[] ia) {
    long sum = 0;
    final int sz = ia.length;
    for(int i = 0; i < sz; i++)
      sum += ia[i];
    return sum;
  }
  // Approximate largest value of SZ before
  // running out of memory on my machine:
  public static int SZ = 10_000_000;
  public static final long CHECK =
    (long)SZ * ((long)SZ + 1)/2;
  public static void main(String[] args) {
    System.out.println(CHECK);
    Long[] La = new Long[SZ+1];
    Arrays.parallelSetAll(La, i -> (long)i);
    Summing.timeTest("Long Array Stream Reduce",
      CHECK, () ->
      Arrays.stream(La).reduce(0L, Long::sum));
    Summing.timeTest("Long Basic Sum", CHECK, () ->
      basicSum(La));
    // Destructive summation:
    Summing.timeTest("Long parallelPrefix",CHECK, ()-> {
      Arrays.parallelPrefix(La, Long::sum);
      return La[La.length - 1];
    });
  }
}
/* Output:
50000005000000
Long Array Stream Reduce: 1046ms
Long Basic Sum: 21ms
Long parallelPrefix: 3287ms
*/
```

现在，可用内存的大小大概被削减一半，除了basicSum()简单的通过数组内循环，其他所有的时间都大幅增长。更惊奇的是，Arrays.parallelPrefix()比其他方法消耗更多的时间。

我分开了parallel()版本是因为把它运行在上个程序中会引起漫长的垃圾回收，影响了测试结果：

```java
// Summing4.java
import java.util.*;

public class Summing4 {
  public static void main(String[] args) {
    System.out.println(Summing3.CHECK);
    Long[] La = new Long[Summing3.SZ+1];
    Arrays.parallelSetAll(La, i -> (long)i);
    Summing.timeTest("Long Parallel",
      Summing3.CHECK, () ->
      Arrays.stream(La).parallel().reduce(0L,Long::sum));
  }
}
/* Output:
50000005000000
Long Parallel: 1008ms
*/
```

它比不使用parallel()版本稍稍快一点，但是不显著。

引起时间增长的一个重要的原因是处理器内存缓存。在Summing2.java使用原始的long类型，数组la是连续的内存。处理器可以更容易的预期使用量，然后保持缓存中填满下次需要用到的数组元素。使用缓存相比起使用主内存速度会非常非常快。显然Long parallelPrefix受到了影响，因为它不仅仅读取每次计算的两个元素，而且将结果写会到数组中，每个数组都会为Long提供外部缓存。

在Summing3.java和Summing4.java中，La是一个Long类型数组，它们不是连续数组的数据，但是Long对象的连续数组的引用。尽快数组将会被缓存，但是指向的对象几乎总是没有命中缓存。

这些例子使用了不同的sz值来显示内存限制。下面是sz设置为1000万的最小值的结果。

```command
Sum Stream: 69ms
Sum Stream Parallel: 18ms
Sum Iterated: 277ms
Array Stream Sum: 57ms
Parallel: 14ms
Basic Sum: 16ms
parallelPrefix: 28ms
Long Array Stream Reduce: 1046ms
Long Basic Sum: 21ms
Long parallelPrefix: 3287ms
Long Parallel: 1008ms
```

Java8中各种新的内置的并发工具非常好，我见证了它们被当做神奇的灵丹妙药：“所有你需要做的扔到parallel()会变得更快”。我希望我已经证明了不全是这样，如果盲目的应用内置的并发操作有的时候可能会变得更慢。

对并发性和并行性支持的语言和库似乎是“漏洞百出的抽象”一词的完美候选选项，但我开始怀疑是否真的有什么抽象的东西--在编写这些类型的代码时，你永远不会被任何底层系统和工具所隔离，甚至包括CPU缓存。归根到底，如果你非常小心，你创造的东西在特定的情况下是有用的，但是在不同的情况下它不会起作用。有时候差异是两个机器不同的配置方式，或者是程序的预计负载。这并非特定于Java本身--而是并发和并行编程的特质。

你可能争论纯函数式语言不存在这些约束。事实上，纯函数式语言可以解决大部分问题。但最终，例如你编写的系统有一个队列，如果事情没有得到正确的调整，并且输入速率不是估计不正确就会限制了（节流意味着不同的事物，在不同的情况下具有不同的影响），该队列将被填充并阻止，或者溢出。最后，你必须理解所有的细节，任何问题都会破坏你的系统。这是一种不同的编程方式。

我想知道对此的反馈，如果我有任何明显的失误，请在评论中留言，感谢！
