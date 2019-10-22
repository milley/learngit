# 艰难的理解Java中泛型转换

[Understand Casting Generics in Java by Eating Broken Glass](https://link.medium.com/jPg9SPTtZ0)

<img src="./img/generics-1.png" width="100%" >

Java中的类型转换看起来直截了当：一个Dog是Animal的一种类型，因此当你需要一个Animal时，你可以将Dog向上转换成Animal。此外，如果你有一个Animal实际上你知道它是一个Dog，你可以将Animal向下转换成Dog。

```java
interface Trainer<T> {
    void train(T pupil);
    T getPupil();
}
```

当你使用泛型事情就会变得更复杂。如果你有一个Trainer<Dog>，你可以用Trainer<Animal>来替换它吗？如果替换成Trainer<Schnauser>呢？

在这篇文章，我将开发一个简单直接的比喻来帮助你理解Java中的泛型转换，和如何解决一些公共转换(除了认识到公共转换不能工作)。

## 比喻

最上面的图片解释了我的比喻。简单来说，你可以供应任何你想要的东西给一个黑洞，但是你不能把碎玻璃喂给人。

那个比喻可以用下面的代码明确下，将在下面解释。

```java
public void feedStuffToOtherStuff(
    Consumer<Food> human,
    Consumer<Matter> blackhole,
    Supplier<Food> pantry,
    Supplier<Matter> boxOfBrokenGlass
) {
    feed(human, pantry);    // OK
    feed(blackhole, boxOfBrokenGlass);  // OK
    feed(blackhole, (Supplier<Matter>)pantry);  // Should be OK, but Java doesn't know that
    feed(human, boxOfBrokenGlass);  // HFS No! That's not ok!(and Java agrees)
}
```

让我们看一看比喻中浮现的东西：

1. 一个人消费了食物
2. 一个食品柜是食物的供应方
3. 食物是一种物质
4. 黑洞是物质的消费者
5. 一盒破碎的玻璃也是物质的供应方

因此我们有两个供应者和两个消费者。上面的片段试着从每个类型的供应者供给给消费者。让我们分解下：

1. 食品柜中的东西供应给人。没问题，人消费食物，食品柜供给食物
2. 一盒碎玻璃供应给黑洞，没问题，黑洞可以消费任何物质，玻璃也是物质的一种
3. 食品柜供应给黑洞。这看起来对着：食品柜中有食物，食物是物质的一种，黑洞可以消费它。但是Java将会抱怨这个；在完成这篇文章之前我会理解为什么
4. 将碎玻璃供应给人。这就是比喻展示最直接的部分：很显然这不可能，幸亏Java也不允许。我们将会理解为什么不。

## 不兼容类型错误

让我们再仔细的看下case3：尝试将食品柜的东西供应给黑洞。

我们定义的黑洞是一个物质的消费者，我们的食品柜是食物的供应者。因为食物也是物质，每一片食物都是从食品柜取出来，事实上，也是一片物质，所以食品柜既是食物的供应者也是物质的供应者。

但是Java不能那么理解。当你试着使用Supplier<Food>代替Supplier<Matter>，你将会从编译器得到一个类型不兼容的错误。

这同样也发生(幸好)在case4：试着将碎玻璃供应给人。人是食物的消费者，尽快食物是物质的一种类型，也不是说人可以消费任何物质。因此，你不能使用Consume<Food>来代替Consume<Matter>。

## 挖掘

名字"supplier"和"consumer"不是附带的：它们代表两种不同的利用泛型的方式。看一下它们的接口：

```java
interface Consume<T> {
    void consume(T input);
}

interface Supplier<T> {
    G get();
}
```

两个接口都是用一个类型参数，但是它们使用是有区别的：Consumer有一个带参数的方法；Supplier的方法没有带任何参数。

就拿Consumer来说，它可以接收任何类型为T的参数，包含任何子类型是T：黑洞可以消费任何物质，包括食物和坏的玻璃。

```java
Food food;
Matter brokenGlass;
blackhole.consume(food);    // OK
blackhole.consume(brokenGlass); // OK
```

供应者的情况就完全不同了。Supplier的接口描述了get方法返回一个T类型的值：这可以是一个类型是T类的实例，或者任何为T的子类型。因此，大多数指定为T的类型的可以做(在编译的时候)。换句话说，Supplier<Parent>不是Supplier<Child>的子类。

```java
// Supplier<Matter> gives us Matter
Matter brokenGlass = boxOfBrokenGlass.get();

// Nope! We can't assume that the Matter returned is actually Food.
Food edibleGlass = boxOfBrokenGlass.get();
```

上面的例子向下转换了类型参数，从T转到T的子类(例如从Parent转到Child)。相反的当你向上转换参数类型，从T转换到T的超类：食物供应者可以，假设的，当成是物质的供应者，但是一个食物消费者不能当成任何物质的消费者。因此，一个Consumer<Parent>不是Consumer<Child>的子类型。

一般情况下，一个泛型类型即可以是供应者也可以是消费者，意味着我们获取的通过类型转换(向上或向下)获取的类型不能既是子类又是超类：它们是无关类型不能被转换，这也是为什么Java编译器给出你“不合法类型”的错误。

## 通过委托来转换接口

所以Java编译器不允许我们跨类型转换时因为没有通用的方式做这个事情。但是在泛型值消费或者只供应的情况下，有可能实现接口生成一个新的对象。

例如，我们知道消费者的类型T可以消费任何T的子类型。去生成我们的“转换对象”，我创建了接口的实例委托给原型，就像这样：

```java
class CaseConsumer<P, C extends P> implements Consumer<C> {
    private final Consumer<P> from;

    CastConsumer(Consumer<P> from) {
        this.from = from;
    }

    @Override
    public void comsume(C input) {
        from.consume(input);
    }
}
```

供应者也是以这种方式工作，把子类型规则(C)和超类型(P)对调下：

```java
class CastSupplier<C extends P, P> implements Supplier<P> {
    private final Supplier<C> from;

    CastSupplier(Supplier<C> from) {
        this.from = from;
    }

    @Override
    public P get() {
        return from.get();
    }
}
```

上面的例子阐述了我们的目的；接口可以被实现成匿名类，单一方法接口也可以用lambda来实现：

```java
static <P, C extends P> Consumer<C> cast (Consumer<P> from) {
    return from::consume;
}

static <p, C extends P> Supplier<P> cast (Supplier<C> from) {
    return from::get;
}
```

如果你依旧觉得模糊，关于为什么Java编译器不能支持参数类型自动转换，试着实现这个Trainer接口的类型就像这样：

```java
interface Trainer<T> {
    void train(T pipil);
    T getPupil();

    static <P, C extends P> Trainer<P> castUp(Trainer<C> from) {
        // ???
    }

    static <C extends P, P> Trainer<C> castDown(Trainer<P> from) {
        // ???
    }
}
```

## 结论

Java编译器不允许通过参数类型转换泛型类型是因为目标类型，一般来说既不是子类也不是超类。也就是说，你可以通过创建一个新的对象来委托原型：对于消费类型参数，你可以使用这个技术来创建一个消费者子类；对于供应者，你可以创建一个供应者子类。一个物质消费者也可以消费食物；一个食物供应者必须是物质供应者。:w
