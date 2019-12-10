# Java传值和传引用

[Java Pass By Value and Pass By Reference](https://javapapers.com/core-java/java-pass-by-value-and-pass-by-reference/)

Java是按值传递。在Java中不存在通过引用传递。这个Java教程带你了解传值和传引用，然后使用例子探索Java使用传值。最重要的是，我们需要使用“传值”和“传引用”两个术语来明确我们的意思。有些人说在Java中，原值是按值传递，对象时按引用传递。这不正确。

## 按值传递

让我们理解什么是按值传递。实参表达了传递到一个方法中就会派生一个重新计算的值。因此这个值就会存储在一块内存中它也就变成了调用方法的形式参数。这个机制称作按值传递Java也使用它。

## 按引用传递

在传引用中，形式参数仅仅是实参的别名。它指向了真实的参数。所有的变更到形参都会反射到实参，反之亦然。

## Java语言这样指定

在Java语言规格说明书 8.4.1 形参章节这样说，

> “当方法或者构造函数被调用(§15.12)，在执行方法或者构造函数体之前，实参表达式的值重新初始化创建参数变量，每个声明的类型。”

这明确了实参表达式重新计算作为形参然后会按值传递到方法体中。当对象作为参数传入，对象本身不会作为参数传递给调用方法。在内部，该对象的引用作为值传递，并成为方法中的形参。

Java使用JVM栈内存来创建新的对象，这些对象是形参。这些新创建的对象范围在方法执行的边界之内。一旦方法执行完成，就可以回收此内存。

## 测试传值 vs 传引用

我们可以运行一个简单的交换测试来检查传值vs传引用。让我们传入两个参数然后再方法中交换它们，最后检查是否实参被交换了。如果实参被交换说明是传引用反之是传值。

```java
public class Swap {
    public static void main(String args[]) {
        Animal a1 = new Animal("Lion");
        Animal a2 = new Animal("Crocodile");
        System.out.println("Before Swap:- a1:" + a1 + "; a2:" + a2);
        swap(a1, a2);
        System.out.println("Before Swap:- a1:" + a1 + "; a2:" + a2);
    }

    public static void swap(Animal animal1, Animal animal2) {
        Animal temp = new Animal("");
        temp = animal1;
        animal1 = animal2;
        animal2 = temp;
    }
}

class Animal {
    String name;

    public Animal(String name) {
        this.name = name;
    }

    public String toString() {
        return name;
    }
}
```

### 输出结果1:

```console
Before Swap:- a1:Lion; a2:Crocodile
After Swap:- a1:Lion; a2:Crocodile
```

对象没有被交换因为Java是按值传递。

## 在C++中交换

在C++中同样的代码将会交换对象因为它使用按引用传递。

```c++
void swap(Type& arg1, Type& arg2) {
    Type temp = arg1;
    arg1 = arg2;
    arg2 = temp;
}
```

## Java中可以传引用吗？

所有的事情都简单为什么这个主题这么热门？让我们通过下面的代码看下通过改变方法中传入对象的属性，反射到真实的参数上。

```java
public class Swap {
    public static void main(String args[]) {
		Animal a = new Animal("Lion");

		System.out.println("Before Modify: " + a);
		modify(a);
		System.out.println("After Modify: " + a);
	}

	public static void modify(Animal animal) {
		animal.setName("Tiger");
	}
}

class Animal {
	String name;

	public Animal(String name) {
		this.name = name;
	}

	public String toString() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}
```

### 输出结果2:

```console
Before Modify: Lion
After Modify: Tiger
```

如果参数按值传递那么我们如何修改传入的参数属性？这是因为Java按值传递对象引用。当一个对象被传递给方法，实际上对象的引用被传递了。形参是映射到实参的引用上。

## Java通过值来传引用

下面的示例图阐明了变量遵循按值传递的方法范围。考虑到数字100，200，600和700作为指针指向内存单元，这仅仅是逻辑上为了帮助你来理解。

<img src="./img/roadmap.png" width="100%" >

在图1中，有两个对象分别有变量名a1，a2分别引用了100，200的内存位置。

<img src="./img/roadmap.png" width="100%" >

在图2中，有两个对象formal-arg1，formal-arg2分别引用了600，700的内存位置。这里是重点，600，700的内容位置引用了另外的100和200的位置。所以调用方法获得了传参的引用。使用引用，它只能操作实参对象的内容。但是它不能改变a1指向100和a2指向200这个事实，这就是当我们交换对象我们试着去做的事情。

值得注意的是“引用是作为值的副本”到新的变量，并且它被作为形参提供给调用的方法。它没有得到a1变量，这个变量是在实参范围。这是传值和传引用的关键区别。
