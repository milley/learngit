# HOOK

在Windows应用程序中点击鼠标左键，操作系统会感知到这个事件然后产生鼠标左键按键消息，通过消息队列，应用程序通过调用GetMessage函数取出消息，然后调用DispatchMessage函数将这条消息调度给操作系统，操作系统会调用在设计窗口类时指定的应用程序窗口过程对这个消息进行处理。

在有的时候需要对某些消息进行屏蔽，这里就可以使用到HOOK技术。HOOK程序在实现时，可以通过setWindowsHookEx函数来安装一个钩子过程。setWindowsHookEx实际是一个宏，在非UNICODE编码定义的是SetWindowsHookExA函数。函数定义如下：

```c++
HHOOK SetWindowsHookExA(
  int       idHook,
  HOOKPROC  lpfn,
  HINSTANCE hmod,
  DWORD     dwThreadId
);
```

其中idHook是表示hook的类型。

|Value|Meaning|
| --- | ----- |
|WH_CALLWNDPROC 4|Installs a hook procedure that monitors messages before the system sends them to the destination window procedure. For more information, see the CallWndProc hook procedure.|
|WH_CALLWNDPROCRET 12|Installs a hook procedure that monitors messages after they have been processed by the destination window procedure. For more information, see the CallWndRetProc hook procedure.|
|WH_CBT 5|Installs a hook procedure that receives notifications useful to a CBT application. For more information, see the CBTProc hook procedure.|
|WH_DEBUG 9|Installs a hook procedure useful for debugging other hook procedures. For more information, see the DebugProc hook procedure.|
|WH_FOREGROUNDIDLE 11|Installs a hook procedure that will be called when the application's foreground thread is about to become idle. This hook is useful for performing low priority tasks during idle time. For more information, see the ForegroundIdleProc hook procedure.|
|WH_GETMESSAGE 3|Installs a hook procedure that monitors messages posted to a message queue. For more information, see the GetMsgProc hook procedure.|
|WH_JOURNALPLAYBACK 1|Installs a hook procedure that posts messages previously recorded by a WH_JOURNALRECORD hook procedure. For more information, see the JournalPlaybackProc hook procedure.|
|WH_JOURNALRECORD 0|Installs a hook procedure that records input messages posted to the system message queue. This hook is useful for recording macros. For more information, see the JournalRecordProc hook procedure.|
|WH_KEYBOARD 2|Installs a hook procedure that monitors keystroke messages. For more information, see the KeyboardProc hook procedure.|
|WH_KEYBOARD_LL 13|Installs a hook procedure that monitors low-level keyboard input events. For more information, see the LowLevelKeyboardProc hook procedure.|
|WH_MOUSE 7|Installs a hook procedure that monitors mouse messages. For more information, see the MouseProc hook procedure.|
|WH_MOUSE_LL 14|Installs a hook procedure that monitors low-level mouse input events. For more information, see the LowLevelMouseProc hook procedure.|
|WH_MSGFILTER -1|Installs a hook procedure that monitors messages generated as a result of an input event in a dialog box, message box, menu, or scroll bar. For more information, see the MessageProc hook procedure.|
|WH_SHELL 10|Installs a hook procedure that receives notifications useful to shell applications. For more information, see the ShellProc hook procedure.|
|WH_SYSMSGFILTER 6|Installs a hook procedure that monitors messages generated as a result of an input event in a dialog box, message box, menu, or scroll bar. The hook procedure monitors these messages for all applications in the same desktop as the calling thread. For more information, see the SysMsgProc hook procedure.|

lpfn表示指向钩子程序的指针。如果dwThreadId为0，或者指定了一个其他进程创建的线程标识符，那么参数lpfn必须指向一个位于DLL动态库中的钩子过程。否则，lpfn指向当前进程相关的代码中定义的一个钩子过程。

hmod指定lpfn指向的钩子过程所在的DLL句柄。如果参数dwThreadId指定的线程由当前进程创建，并且相应的钩子过程定义于当前进程相关的代码中，那么必须将参数hmod设置为NULL。

dwThreadId指定与钩子过程相关的线程标识。如果其值为0，那么安装的钩子过程将于桌面上运行的所有线程有关。

如果调用成功，返回值就是所安装的钩子过程的句柄。

## 进程内钩子

新建一个基于对话框的MFC工程，这里用屏蔽鼠标左键来举例。首先安装一个鼠标钩子：

```c++
HHOOK g_hMouse = NULL;
LRESULT CALLBACK MouseProc(int nCode, WPARAM wParam, LPARAM lParam)
{
	return 1;
}
```

在对话框初始化函数中调用：

```c++
g_hMouse = SetWindowsHookEx(WH_MOUSE, MouseProc, NULL, GetCurrentThreadId());
```

以上代码就会屏蔽掉鼠标按键，需要关闭对话框只能通过把鼠标移动到光标上按下空格或者回车，也可以通过ALT+F4来完成退出窗口。

除了鼠标事件，还可以操作键盘事件。这里再增加一个屏蔽ALT+F4按键，但是可以通过按下F2按键来关闭窗口。需要先增加一个全局变量g_hWnd来表示窗口句柄，还有一个键盘钩子句柄。

```c++
HHOOK g_hKeyboard = NULL;
HWND g_hWnd = NULL;
```

在键盘屏蔽钩子中首先判断按下的键是否为ALT+F4的组合键，如果是则屏蔽。如果按下的是F2则调用SendMessage来给窗口发送Close消息用来关闭窗口。

```c++
LRESULT CALLBACK KeyboardProc(int nCode, WPARAM wParam, LPARAM lParam)
{
	if (VK_F4 == wParam && (1 == (lParam >> 29 & 1)))
	{
		return 1;
	}
	if (VK_F2 == wParam)
	{
		::SendMessage(g_hWnd, WM_CLOSE, 0, 0);
		UnhookWindowsHookEx(g_hMouse);
		UnhookWindowsHookEx(g_hKeyboard);
	}
	return CallNextHookEx(g_hKeyboard, nCode, wParam, lParam);
}
```

同样，在窗口初始化函数时调用键盘钩子：

```c++
g_hWnd = m_hWnd;
g_hKeyboard = SetWindowsHookEx(WH_KEYBOARD, KeyboardProc, NULL, GetCurrentThreadId());
```

这样就可以实现上面屏蔽ALT+F4按键，可以通过F2来关闭窗口的功能。

## 全局钩子

根据前面介绍，如果想让安装的钩子过程与所有进程相关，应该将SetWindowHookEx函数的第四个参数设置为0，并将它的第三个参数指定为安装钩子过程的代码所在的DLL的句柄。

首先编写一个安装钩子过程的DLL，新建一个Win32 Dynamic Link Library类型的工程，新建Hook工程并且新建一个源文件Hook.cpp：

```c++
#include <windows.h>

HHOOK g_hMouse = NULL;
HHOOK g_hKeyboard = NULL;
HWND g_hWnd;

LRESULT CALLBACK MouseProc(int nCode, WPARAM wParam, LPARAM lParam)
{
	return 1;
}

LRESULT CALLBACK KeyboardProc(int nCode, WPARAM wParam, LPARAM lParam)
{
	if (VK_F2 == wParam)
	{
		SendMessage(g_hWnd, WM_CLOSE, 0, 0);
		UnhookWindowsHookEx(g_hMouse);
		UnhookWindowsHookEx(g_hKeyboard);
	}
	return 1;
}

void SetHook(HWND hWnd)
{
	g_hWnd = hWnd;
	g_hMouse = SetWindowsHookEx(WH_MOUSE, MouseProc, GetModuleHandle("Hook"), 0);
	g_hKeyboard = SetWindowsHookEx(WH_KEYBOARD, KeyboardProc, GetModuleHandle("Hook"), 0);
}
```

MouseProc和KeyboardProc和前面一样。在SetHook函数中调用鼠标和键盘钩子，其第三个参数都是DLL模块的句柄，第四个参数都是0。再定义一个模块定义文件Hook.def，用来定义导出函数：

```def
LIBRARY Hook
EXPORTS
SetHook @1
```

这样编译完成后会生成一个Hook.dll，可以用客户端程序来调用此动态库用来演示屏蔽按键。新建一个基于对话框的WIN32工程，首先在对话框的头文件中声明SetHook函数：

```c++
_declspec(dllimport) void SetHook(HWND hwnd);
```

在对话框初始化方法中，首先定义了2个变量cxScreen和cyScreen用来设置屏幕的宽度和高度。为了得到宽度和高度又调用了GetSystemMetrics函数，通过SetWindowPos来设置窗口大小，然后调用SetHook来屏蔽鼠标按键和激活F2关闭窗口的功能：

```c++
int cxScreen, cyScreen;
cxScreen = GetSystemMetrics(SM_CXSCREEN);
cyScreen = GetSystemMetrics(SM_CYSCREEN);
SetWindowPos(&wndTopMost, 0, 0, cxScreen, cyScreen, SWP_SHOWWINDOW);
SetHook(m_hWnd);
```

运行程序后就会发现当前对话框会最大化平铺到当前窗口，鼠标按键也已经屏蔽，只有按下F2才会退出当前对话框。
