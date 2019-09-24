# Windows动态链接库

动态链接库在我们平时使用的软件中几乎随处可见，动态链接库可以使软件的模块化更加灵活，有了动态链接库可以使得程序调用可以跨语言，同时也可以节省磁盘空间或者内存。动态链接库加载可以分为隐式加载和显式加载。

## Win32 DLL的创建

使用VC++新建一个Win32 Project，Application Type选择DLL，这里创建一个Dll1.cpp为例说明：

```c++
_declspec(dllexport) int add(int a, int b)
{
	return a + b;
}

_declspec(dllexport) int subtract(int a, int b)
{
	return a - b;
}
```

编译完成后，通过使用VC自带的dumpbin命令行工具可以看出导出函数：

```cmd
Dumpbin -exports dll1.dll
```

### 隐式链接方式

创建一个基于对话框的MFC程序，工程取名为DllTest1，对话框增加两个按钮分别为Add和Subtract。

#### 1. 使用extern声明外部函数

在调用add和subtract之前需要先声明函数是外部定义的：

```c++
extern int add(int a, int b);
extern int subtract(int a, int b);
```

在按钮的消息函数实现中调用add和subtract函数：

```c++
void CDllTestDlg::OnBnClickedBtnAdd()
{
	// TODO: Add your control notification handler code here
	CString str;
	str.Format("5+3=%d", add(5, 3));
	MessageBox(_T(str));
}


void CDllTestDlg::OnBnClickedBtnSubtract()
{
	// TODO: Add your control notification handler code here
	CString str;
	str.Format("5-3=%d", subtract(5, 3));
	MessageBox(_T(str));
}
```

在链接时要指定动态库的名称，设置路径：Configuration Properties->Linker->Input->Additional Dependencies

增加lib名称：..\Debug\Dll1.lib;

重新编译后点击Add按钮和Subtract按钮可以看到运算结果正常展示。

#### 2. 优化函数导出C++类或者成员函数

为了让Dll1这个工程可以导出给别的工程也可以自己使用，增加Dll1.h头文件，这里增加一个类Point用来表示坐标：

```c++
#ifdef DLL1_API
#else
#define DLL1_API _declspec(dllimport)
#endif

DLL1_API int add(int a, int b);
DLL1_API int subtract(int a, int b);

class DLL1_API Point
{
public:
	void /*DLL1_API*/ output(int x, int y);
	void test();
};#ifdef DLL1_API
#else
#define DLL1_API extern "C" _declspec(dllimport)
#endif

DLL1_API int add(int a, int b);
DLL1_API int subtract(int a, int b);
```

通过在Dll1.cpp中声明宏来动态判断头文件中方法是否导出：

```c++
#ifdef DLL1_API
#else
#define DLL1_API _declspec(dllimport)
#endif

DLL1_API int add(int a, int b);
DLL1_API int subtract(int a, int b);

class DLL1_API Point
{
public:
	void /*DLL1_API*/ output(int x, int y);
	void test();
};
```

重新编译Dll1工程后，在DllTest1中增加一个按钮Output，用来调用Point类的output函数显示坐标：

```c++
void CDllTestDlg::OnBnClickedBtnOutput()
{
	Point pt;
	pt.output(5, 3);
}
```

这样就可以成功调用output来输出坐标，同样add和subtract也可以正常调用。这里演示的是导出整个类，也可以导出output方法。

#### 3. 解决名字改变问题

因为C++编译对导出函数的名字进行了改编，而且不同编译器改编规则不同就会导致导出名字不一样，客户端调用就会无法使用。这样就可以通过在导出的时候增加extern "C"来限定导出函数的名称不发生改变。

```c++
//h

#ifdef DLL1_API
#else
#define DLL1_API extern "C" _declspec(dllimport)
#endif

DLL1_API int add(int a, int b);
DLL1_API int subtract(int a, int b);

//cpp

#define DLL1_API extern "C" _declspec(dllexport)

int add(int a, int b)
{
	return a + b;
}

int subtract(int a, int b)
{
	return a - b;
}
```

这样可以解决名字变化的问题，但是无法支持导出类的成员函数，因此只能用于导出全局函数这种情况。

但是，如果导出函数采用标准调用约定，在声明函数时添加_stdcall关键字：

```c++
DLL1_API int _stdcall add(int a, int b);
DLL1_API int _stdcall subtract(int a, int b);

int _stdcall add(int a, int b)
{
    return a + b;
}

int _stdcall subtract(int a, int b)
{
    return a - b;
}
```

这样编译完成后，使用dumpbin命令查看，函数名还是发生变化了。这种情况下，可以通过模块定义文件(DEF)的方式来解决名字改编问题。

### 显示链接方式

新建一个动态库工程Dll2，添加Dll2.cpp

```c++
int _stdcall add(int a, int b)
{
	return a + b;
}

int subtract(int a, int b)
{
	return a - b;
}
```

添加模块定义文件，Dll2.def:

```def
LIBRARY Dll2

EXPORTS
add
subtract
```

修改调用的消息映射函数：

```c++
void CDllTest2Dlg::OnBtnAdd()
{
	HINSTANCE hInst;
	hInst = LoadLibrary("Dll2.dll");
	//typedef int (*ADDPROC)(int a, int b);	// define function pointer type
	typedef int (_stdcall *ADDPROC)(int a, int b);
	ADDPROC Add = (ADDPROC)GetProcAddress(hInst, "add");	// get export function
	if (!Add)
	{
		MessageBox(_T("获取函数地址失败！"));
		return;
	}
	CString str;
	str.Format("5+3=%d", Add(5, 3));
	MessageBox(str);
}
```

当DLL中导出函数采用的是标准调用约定时，访问该DLL的客户端程序也需要采用该约定来访问相应的导出函数。
