# Python中sort和sorted

对于列表来说，可以使用list.sort()方法对当前列表进行排序。sorted()函数是对所有可迭代的对象进行排序。

区别：

list中的sort()函数对已经存在的列表进行操作，内建函数sorted()方法返回的是一个新的list，而不是在原来的基础上进行的操作。

```python3
>>> num = [3,2,1]
>>> num.sort()
>>> print(num)
[1, 2, 3]
>>> num = [3, 2, 1]
>>> sorted(num)
[1, 2, 3]
>>> num
[3, 2, 1]
```
