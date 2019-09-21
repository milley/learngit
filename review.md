# C++中的Tree-Set数据结构

[Tree-Set data structure in C++](https://link.medium.com/doFxDJ9w9Z)

你知道什么是二叉查找树和set在数学中的概念吗？在这篇文章中我将会使用set的属性来构建一个二叉树，自平衡和可以支持各种数据类型(从你拥有的内置对象)，使用C++的超级特性模板。

树是一种复杂的数据结构，大多数在树上执行的函数都需要递归调用。这个练习将会通过递归训练我们的大脑，理解如何构建一个二叉树，AVL树如何自平衡和使用二叉树优先于其他数据结构的不同原因，包括一个简单的复杂性分析。

<img src="./img/TreeSet_1.png" width="60%" >

如果一棵树的所有子树不超过2个我们就把这棵树称作二叉树，因此，一个子节点不存在通常就是指向NULL。一个完全二叉树是指除了最后一级其余的层级都有左右子节点。树的一些相关术语有：Edge(两个节点中间的连接线)，子节点和父节点，根节点(所有节点最高的祖先节点)和叶子节点(没有子节点的节点)。

<img src="./img/TreeSet_2.png" width="60%" >

首先，我为二叉查找树定义了一个类DoublyNode，它意味着所有的节点最多只能有2个子节点，用另外一种说法就是每个节点有2个指针(left:prev或者right:next)和一个数据value<Type>，还有一些成员变量和函数定义。

<img src="./img/TreeSet_2.png" width="60%" >

在DoublyNode类中，头文件"DoublyNode.h"包含了#pragma once和一个模板来使节点灵活的支持不同数据类型template<class Type>，类定义了一个私有的成员变量Type data和两个公共的指针Type(left和right)，指针将会初始化成NULL(C++11中的nullptr)整体的C++11构成"Type *left{nullptr}, *right{rightptr};"。

```c++
#pragma once

template<class Type>
class DoublyNode
{
    private:
        Type key;
    
    public:
        DoublyNode<Type> *left{ nullptr }, *right{ nullptr };

        DoublyNode();
        DoublyNode(const Type& data);

        // getter and setter
        Type& getData();
        void setData(const Type& data);

        // destructor
        ~DoublyNode();
};
```

为什么要使用#pragma once或者#infdef DOUBLYNODE_H #def DOUBLYNODE_H #endif？这是为了在编译文件时只引入一次。所有的成员函数都是public，两个构造函数(一个没有参数一个有Type value参数)，一个setter用来给节点设置值和一个getter用来从节点取值，跟随OOP封装概念。

类DoublyNode和.cpp文件中5-13行是两个构造方法，这里我只用了带参数的定义，当data值是以参数传到节点，right和left指针指向{ nullptr }。

```c++
#include "DoublyNode.h"

// constructors with no parameter calls the parameter node with value 0
template<class Type>
DoublyNode<Type>::DoublyNode() : DoublyNode(nullptr)
{
}
// constructor with parameters
template<class Type>
DoublyNode<Type>::DoublyNode(cont Type& data)
{
    this->key = data;
}
// getters for the value of the node
template<class Type>
Type& DoublyNode<Type>::getData()
{
    return key;
}
// setter value of the node
template<class Type>
void DoublyNode<Type>::setData(const Type& data)
{
    this->key = data;
}
// destructor
template<class Type>
DoublyNode<Type>::~DoublyNode()
{
    // next points to NULL
    this->left, this->right = nullptr;
}
```

在类定义中成员函数都非常简单也容易理解，这些方法是：getters，setters，最后是析构函数将指针指向NULL避免指针出错。

Tree-set实现了二叉查找树(BST)。什么是二叉查找树？BST维护了动态改变数据集合的顺序，和排序数组不同的是它的元素不能高效的插入和删除，BST在很多场景中非常有用是因为它在将元素按顺序初入后可以很高效的进行查找。BST最坏的时间复杂度是O(h)，h代表树的高度。

<img src="./img/TreeSet_3.png" width="60%" >

树的高度是指从根节点到最远叶子节点其中连线的个数。如上图所示，查找，插入，删除的复杂度是O(log n)。

BST有4个属性：每个节点包含了一个值，左子树的节点值都要小于其父节点的值，右子树的节点值都要大于其父节点的值，基本上，重复的值不允许，就像数学定义中的SET{}。

我们定义的树可以在文件TreeSet.h中找到，我们可以先看到预编译声明#pragma once，template<class Type>声明，成员函数和成员变量。开始，成员变量是私有的。

```c++
#pragma once
#include "DoublyNode.h"

template <class Type>
class TreeSet
{
    private:
        // head of the Tree-set, size MEMBER VARIABLES
        DoublyNode<Type>* root{ nullptr };
        unsigned int length{ 0 };

        DoublyNode<Type>* insertRecursively(DoublyNode<Type> *newNode, DoublyNode<Type> *node);
        void printRecursively(DoublyNode<Type> *node);
        // get height of a given node
        int depthRecursively(const DoublyNode<Type> *node);
        // search a value in the tree
        bool searchRecursively(DoublyNode<Type> *node, const Type &data);
        // delete tree
        void deleteTree(const DoublyNode<Type> *node);
        // get balance for factor of node
        int getBalance(const DoublyNode<Type> *subTree);
        // AVL rotation
        DoublyNode<Type>* rightRotate(DoublyNode<Type>* y);
        DoublyNode<Type>* leftRotate(DoublyNode<Type>* x);

    public:
        TreeSet();
        TreeSet(const Type &data);
        TreeSet(DoublyNode<Type>* root);

        void add(const Type data);
        void add(DoublyNode<Type> *node);
        void printSet();

        bool isInSet(const Type& data);
        void remove(const Type& data);
        void clear();
        bool isEmpty();
        int size();
        ~TreeSet();
};
```

在公共成员方法中我们能找到基本数据结构的方法来执行基本操作(insert, delete, search and clear)。私有成员方法是递归方法在树的内部执行。我们为什么需要递归？树就是一个递归的数据结构，这意味着树是由简单的同样的数据结构组成。

第一个函数是插入。方法"void insert(const Type& data)"或"void insert(const DoublyNode<Type> &newNode)"将一个节点插入到树中是共有的，也就是说，这可以从类外面访问，参数可以是Type TreeSet<Type>的值或者已经创建好的节点。内部方法调用私有递归方法(insertRecursively)，这个方法迭代执行整个树结构通过比较data值和其他节点来移动节点直到BST属性分布(小的移到左边，大的移到右边)，直到碰到了叶子节点，然后新的节点添加到叶子节点的子节点。

<img src="./img/TreeSet_4.png" width="60%" >

上面的图，插入40，必须横向比较其他节点的值：1)40比100小，移到左节点，2)40比20大，移到右节点，3)40比30大，30是叶子节点，移到右子节点。注意左边所有的值都小于100。

<img src="./img/TreeSet_5.png" width="60%" >

```c++
// insert data a new node
template<class Type>
void TreeSet<Type>::add(const Type data)
{
    this->add(new DoublyNode<Type>{ data });
}

// insert node
template<class Type>
void TreeSet<Type>::add(DoublyNode<Type> *newNode)
{
    // calling recursive method
    this->root = insertRecursively(newNode, this->root);
}
```

这个过程递归节点，调用方法比较值。查找和插入最坏的时间复杂度就是O(h)其中h是二叉查找树的高度。

<img src="./img/TreeSet_6.png" width="60%" >

```c++
// inserting node
template<class Type>
DoublyNode<Type>* TreeSet<Type>::insertRecursively(DoublyNode<Type>* newNode, DoublyNode<Type>* node)
{
    // when node of the tree is node return new node
    if (node == nullptr)
    {
        node = newNode;
        this->length++;
        return node;
    }
    else if (newNode->getData() < node->getData())
    {
        node->left = insertRecursively(newNode, node->left);
    }
    else if (newNode->getData() > node->getData())
    {
        node->right = insertRecursively(newNode, node->right);
    }
    // ...
}
```

上面这个图我们可以找到递归方法来插入新的节点，基本情况是当当前节点指向NULL，新的节点可以增加，如果没有，就递归调用，使用比较当前节点和新节点的值来确定继续调用左子树还是右子树。

二叉查找树的插入方法最坏的查询、插入和删除复杂度是O(n)，当BST非平衡退化成为类似链表的结构。

<img src="./img/TreeSet_7.png" width="60%" >

如上图所示，所有的二叉查找树都退化了因为这不是一个完全二叉树，所有的节点都只有一个孩子节点，为了证明这个，我们选择了上图中第一个，插入5，将会耗费O(h)，高度和节点数一样，意味着插入、删除和查找都是O(n)。

有BST最主要的原因是每种操作复杂度接近O(log n)，为了达到这种目标我们要定义和实现平衡二叉查找树。

<img src="./img/TreeSet_8.png" width="60%" >

平衡二叉查找树的不同之处在于左子树和右子树的高度相差不能大于1.就像我前面描述的，树是一个递归的数据结构平衡二叉查找树的概念适用于每个节点和子节点。平衡二叉查找树保持最小的高度和根节点到每个节点的步数小于log(n)。

<img src="./img/TreeSet_9.png" width="60%" >

最后接近完成我们的数据结构Tree-Set(AVL)，我们需要平衡我们的树来实现AVL树的概念。

一个AVL(Adel'son-Vel'skii and E.M.Landis)树是简单的自平衡二叉树，它需要左子树高度和右子树高度相差不能大于1。

AVL树在插入和删除节点时提供了旋转。这里有两种旋转，左旋和右旋。左旋和右旋都执行递归，使得旋转的节点最终成为根节点。

<img src="./img/TreeSet_10.png" width="60%" >

在计算每个子树的高度和为了比较子树高度然后执行旋转是非常重要的，这个方法depthRecursively(const DoublyNode<Type>* node)递归的计算了给定的节点高度然后比较左边和右边子树，它的基本情况是当前节点时NULL，它返回0.

<img src="./img/TreeSet_11.png" width="60%" >

另外一个比较左边和右边子树高度的方法：depthRecursively(const DoublyNode<Type> *node)。这个比较方法返回不同的高度和它调用getBalance(const DoublyNode<Type> *node)。

旋转操作改变很少的指针因此耗费固定时间，所以AVL插入保持平衡二叉树的复杂度O(log n)。

<img src="./img/TreeSet_12.png" width="60%" >

在改变树的每个操作后确保BST实现了AVL概念，必须执行一些二次平衡。其中包含了四种不同的操作：Left-Left case, Left-Right case, Right-Right case和Right-Left case。

<img src="./img/TreeSet_13.png" width="60%" >

第一种情况，树旋转到右边，在上面的例子中，节点50被分配为新的根节点，它的右指针现在成为100的左指针。

<img src="./img/TreeSet_14.png" width="60%" >

第二种情况和第一种类似，树旋转到左边重新分配根节点移动指针。

<img src="./img/TreeSet_15.png" width="60%" >

第三种情况更复杂一点，但是我们仔细看下就发现，它仅仅是子树左旋，然后子树再右旋得到一个新的根。这个和最后一种情况类似。

<img src="./img/TreeSet_16.png" width="60%" >

上面的几种情况实现起来如下：

<img src="./img/TreeSet_17.png" width="60%" >

下一个操作是查找，这个非常简单，TreeSet类定义了isInSet(const Type &data)调用私有方法searchRecursively(DoublyNode<Type> *node, const Type &data)，这个方法和插入类似但是返回了一个bool变量标识着是否找到。

查找的基本原则是当前节点是NULL，返回false。如果值被找到，返回true然后被分配给其他调用。

<img src="./img/TreeSet_18.png" width="60%" >

注意如果你需要下载代码，所有的方法在不同的位置在TreeSet.cpp中。

<img src="./img/TreeSet_19.png" width="60%" >

[To download the file go to my Github](https://github.com/terselich/TreeSet)
