# vs2010条件断点

局部变量定义如下：

```c++
string tech = "ABC";
int band = 4;
```

如果需要判断tech是"ABC"且band是4，则增加条件断点需注意：

错误用法：

```c++
tech=="ABC" && band==4
```

报错：The breakpoint cannot be set. Unable to evaluate the breakpoint condition: CXX0058: Error: overloaded operator not found

正确用法：

```c++
tech._Bx._Buf[0] == 'A' && band == 4
```

也可以这样比较:

```c++
strcmp(tech._Bx._Buf, "ABC") == 0 && band == 4
```

条件断点支持函数:

```c++
strlen, wcslen, strnlen, wcsnlen, strcmp, wcscmp, _stricmp, _wcsicmp, strncmp, wcsncmp, _strnicmp, _wcsnicmp, strchr, wcschr, strstr, wcsstr.
```
