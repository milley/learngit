# Python排序容器

[Sorted Containers](https://grantjenks.com/docs/sortedcontainers/#quickstart)仅Python编写，和C扩展同样高性能，使用Apache2的许可证。

Python的标准库除了需要一个排序类型的容器，其他已经足够优秀了。很多人会证明，如果没有它也可以走的很远，但是当前你需要一个排序的列表，排序的字典，或者排序的set，你可能面对这一系列不同的实现，大多数使用C扩展且没有完善的文档和基准测试。

用Python，我们可以做的更好。我们可以使用纯Python来完成！

```python
>>> from sortedcontainers import SortedList
>>> sl = SortedList(['e', 'a', 'c', 'd', 'b'])
>>> sl
SortedList(['a', 'b', 'c', 'd', 'e'])
>>> sl *= 10_000_000
>>> sl.count('c')
10000000
>>> sl[-3:]
['e', 'e', 'e']
>>> from sortedcontainers import SortedDict
>>> sd = SortedDict({'c': 3, 'a': 1, 'b': 2})
>>> sd
SortedDict({'a': 1, 'b': 2, 'c': 3})
>>> sd.popitem(index=-1)
('c', 3)
>>> from sortedcontainers import SortedSet
>>> ss = SortedSet('abracadabra')
>>> ss
SortedSet(['a', 'b', 'c', 'd', 'r'])
>>> ss.bisect_left('c')
2
```

上面显示的所有操作都比以线性时间更快的运行。上面演示需要将近1G的内存才能运行。当排序列表乘以10_000_000，每个'a'到'e'存储了100_000_000引用。每个引用在排序容器中需要8字节。因为它是指向每个指针对象，因此很难再改变。它比典型的二叉树实现减少了66%的开销（例如Red-Black Tree, AVL-Tree, AA-Tree, Splay-Tree, Treap），因为每个节点都需要存储两个指针指向子节点。

排序容器从Python排序集合中获取所有能力，让你使用Python开发更加容易。不需要安装C编译器或者预编译和分发自定义扩展。性能是一项特性，包括测试覆盖率100%和减少数小时的压力。

## SortedList

SortedList的值必须是可以比较的。如果已经存储到排序列表中，最终的值顺序不可以修改。

### 插入方法

- SortedList.add()
- SortedList.update()
- SortedList.__add__()
- SortedList.__iadd__()
- SortedList.__mul__()
- SortedList.__imul__()

### 删除方法

- SortedList.clear()
- SortedList.discard()
- SortedList.remove()
- SortedList.pop()
- SortedList.__delitem__()

### 查找方法

- SortedList.bisect_left()
- SortedList.bisect_right()
- SortedList.count()
- SortedList.index()
- SortedList.__contains__()
- SortedList.__getitem__()

### 迭代方法

- SortedList.irange()
- SortedList.islice()
- SortedList.__iter__()
- SortedList.__reversed__()

### 混合方法

- SortedList.copy()
- SortedList.__len__()
- SortedList.__repr__()
- SortedList._check()
- SortedList._reset()

## SortedDict

排序字典是排序的多个映射。排序字典的键顺序维护。排序字典的设计比较简单：从字典继承，将键以顺序存储和维护。

### 可变映射方法

- SortedDict.__getitem__() (inherited from dict)
- SortedDict.__setitem__()
- SortedDict.__delitem__()
- SortedDict.__iter__()
- SortedDict.__len__() (inherited from dict)

### 插入方法

- SortedDict.setdefault()
- SortedDict.update()

### 删除方法

- SortedDict.clear()
- SortedDict.pop()
- SortedDict.popitem()

### 查找方法

- SortedDict.__contains__() (inherited from dict)
- SortedDict.get() (inherited from dict)
- SortedDict.peekitem()

### 视图方法

- SortedDict.keys()
- SortedDict.items()
- SortedDict.values()

### 混合方法

- SortedDict.copy()
- SortedDict.fromkeys()
- SortedDict.__reversed__()
- SortedDict.__eq__() (inherited from dict)
- SortedDict.__ne__() (inherited from dict)
- SortedDict.__repr__()
- SortedDict._check()

### 可用的排序列表方法（适用于键）

- SortedList.bisect_left()
- SortedList.bisect_right()
- SortedList.count()
- SortedList.index()
- SortedList.irange()
- SortedList.islice()
- SortedList._reset()

### 如果使用了键函数，则提供其他排序列表方法

- SortedKeyList.bisect_key_left()
- SortedKeyList.bisect_key_right()
- SortedKeyList.irange_key()

## SortedSet

排序集合的值以顺序维护。排序集合的值必须可以比较和哈希。当值被存储在排序集合中，值的顺序和哈希值都不能改变。

### 可变集合方法

- SortedSet.__contains__()
- SortedSet.__iter__()
- SortedSet.__len__()
- SortedSet.add()
- SortedSet.discard()

### 顺序方法

- SortedSet.__getitem__()
- SortedSet.__delitem__()
- SortedSet.__reversed__()

### 删除方法

- SortedSet.clear()
- SortedSet.pop()
- SortedSet.remove()

### 集合操作方法

- SortedSet.difference()
- SortedSet.difference_update()
- SortedSet.intersection()
- SortedSet.intersection_update()
- SortedSet.symmetric_difference()
- SortedSet.symmetric_difference_update()
- SortedSet.union()
- SortedSet.update()

### 混合方法

- SortedSet.copy()
- SortedSet.count()
- SortedSet.__repr__()
- SortedSet._check()

### 可用的列表排序方法

- SortedList.bisect_left()
- SortedList.bisect_right()
- SortedList.index()
- SortedList.irange()
- SortedList.islice()
- SortedList._reset()

### 如果使用了键函数，则提供其他排序列表方法

- SortedKeyList.bisect_key_left()
- SortedKeyList.bisect_key_right()
- SortedKeyList.irange_key()
