# python3中nonlocal

函数内使用nonlocal，函数内部使用就近闭合范围内的非全局变量。

```python3
def myfunc1():
    x = "Bill"
    def myfunc2():
        nonlocal x
        x = "hello"
    myfunc2() 
    return x

print(myfunc1())
```

如果不使用nonlocal，则不会修改函数外部的局部变量。

```python3
def myfunc1():
    x = "Bill"
    def myfunc2():
        x = "hello"
    myfunc2() 
    return x

print(myfunc1())
```
