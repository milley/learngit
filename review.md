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
