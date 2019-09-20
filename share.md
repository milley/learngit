# Windows进程间通信

最近看了下Windows进程间通信的内容，感觉和之前看过的Unix进程间通讯还挺像的。这里记录下Windows进程间通信的几种方式。

## 1. 剪贴板

剪贴板是我们平常常用的功能，在之前做过的B/S开发中也有用到过剪贴板的功能。在MFC中往剪贴板写入数据其实是比较方便的，打开剪贴板主要就是用了CWnd类的OpenClipBoard函数来实现，在打开后调用SetClipboardData来设置剪贴板内容，最后调用CloseClipboard来关闭剪贴板。同样，从剪贴板读取内容也是先打开剪贴板，之后先判断剪贴板是否为激活状态，调用IsClipboardFormatAvailable判断即可，通过调用GetClipboardData来从剪贴板获取数据。

主要的功能可以参考下面的示例：

```c++
void CClipboardDlg::OnBnClickedBtnSend()
{
	if (OpenClipboard())
	{
		CString str;
		HANDLE hClip;
		char *pBuf;
		EmptyClipboard();
		GetDlgItemText(IDC_EDIT_SEND, str);
		hClip = GlobalAlloc(GMEM_MOVEABLE, str.GetLength() + 1);
		pBuf = (char*)GlobalLock(hClip);
		strcpy(pBuf, str);
		GlobalUnlock(hClip);
		SetClipboardData(CF_TEXT, hClip);
		CloseClipboard();
	}
}


void CClipboardDlg::OnBnClickedBtnRecv()
{
	if (OpenClipboard())
	{
		if (IsClipboardFormatAvailable(CF_TEXT))
		{
			HANDLE hClip;
			char *pBuf;
			hClip = GetClipboardData(CF_TEXT);
			pBuf = (char*)GlobalLock(hClip);
			GlobalUnlock(hClip);
			SetDlgItemText(IDC_EDIT_RECV, pBuf);
		}
		CloseClipboard();
	}
}
```

## 2. 匿名管道

匿名管道是一个未命名的单向管道，通常用来在一个父进程和一个子进程中间传递数据。所以匿名管道适合于本地机器上父子进程间的通讯。

匿名管道实现中，父进程可以作为服务端，子进程作为客户端。

### 2.1 服务端实现步骤

首先，要创建匿名管道来进行通信。通过调用CreatePipe来创建匿名管道，前两个参数分别传入读句柄和写句柄。创建成功后需要创建子进程，可以通过CreateProcess来实现。

实现代码：

```c++
void CParentView::OnPipeCreate()
{
	// TODO: Add your command handler code here
	SECURITY_ATTRIBUTES sa;
	sa.bInheritHandle = TRUE;
	sa.lpSecurityDescriptor = NULL;
	sa.nLength = sizeof(SECURITY_ATTRIBUTES);
	if (!CreatePipe(&hRead, &hWrite, &sa, 0))
	{
		MessageBox(_T("Create pipe failed!"));
		return;
	}

	STARTUPINFO sui;
	PROCESS_INFORMATION pi;
	ZeroMemory(&sui, sizeof(STARTUPINFO));
	sui.cb = sizeof(STARTUPINFO);
	sui.dwFlags = STARTF_USESTDHANDLES;
	sui.hStdInput = hRead;
	sui.hStdOutput = hWrite;
	sui.hStdError = GetStdHandle(STD_ERROR_HANDLE);

	if (!CreateProcess("Child.exe", NULL, NULL, NULL,
		TRUE, 0, NULL, NULL, &sui, &pi))
	{
		CloseHandle(hRead);
		CloseHandle(hWrite);
		hRead = NULL;
		hWrite = NULL;
		MessageBox(_T("Create process failed!"));
		return;
	}
	else
	{
		CloseHandle(pi.hProcess);
		CloseHandle(pi.hThread);
	}
}
```

需要注意的是父进程在打开子进程应用程序时，需要传入正确的应用程序路径。

服务端读取数据通过调用ReadFile，写数据调用WriteFile来完成。

```c++
void CParentView::OnPipeRead()
{
	// TODO: Add your command handler code here
	char buf[100];
	DWORD dwRead;
	if (!ReadFile(hRead, buf, 100, &dwRead, NULL))
	{
		MessageBox(_T("Read data failed!"));
		return;
	}
	MessageBox(_T(buf));
}


void CParentView::OnPipeWrite()
{
	// TODO: Add your command handler code here
	char buf[] = "http://www.microsoft.com";
	DWORD dwWrite;
	if (!WriteFile(hWrite, buf, strlen(buf) + 1, &dwWrite, NULL))
	{
		MessageBox(_T("Write data failed!"));
		return;
	}
}
```

### 2.2 客户端实现步骤

客户端在初始化时需要先获取读、写的父进程匿名管道句柄，在MFC中客户端通过定义虚函数OnInitialUpdate来实现初始化获取父进程的读、写句柄：

```c++
void CChildView::OnInitialUpdate()
{
	CView::OnInitialUpdate();

	hRead = GetStdHandle(STD_INPUT_HANDLE);
	hWrite = GetStdHandle(STD_OUTPUT_HANDLE);
}

```

在获取成功后，可以和服务端类似读写数据：

```c++
void CChildView::OnPipeRead()
{
	// TODO: Add your command handler code here
	char buf[100];
	DWORD dwRead;
	if (!ReadFile(hRead, buf, 100, &dwRead, NULL))
	{
		MessageBox(_T("Read data failed...!"));
		return;
	}
	MessageBox(buf);
}


void CChildView::OnPipeWrite()
{
	// TODO: Add your command handler code here
	char buf[] = "Pipe test routine";
	DWORD dwWrite;
	if (!WriteFile(hWrite, buf, strlen(buf) + 1, &dwWrite, NULL))
	{
		MessageBox(_T("Write data failed...!"));
		return;
	}
}
```

## 3. 命名管道

命名管道可以通过网络来进行进程间的通信，它屏蔽了网络底层的细节。需要注意的是命名管道只能在服务端创建，客户端只能读写数据。创建命名管道的服务器只能在Windows NT和Windows 2000之后的版本才可以，客户端没有要求。

### 3.1 服务端实现步骤

创建命名管道通过调用CreateNamedPipe来进行创建，创建成功后需要创建时间对象，调用CreateEventA来进行创建。然后通过调用ConnectNamedPipe来连接命名管道，最后调用WaitForSingleObject来等待命名管道进行数据通信，最后关闭。

```c++
void CNamedPipeSrvView::OnPipeCreate()
{
	// TODO: Add your command handler code here
	hPipe = CreateNamedPipe("\\\\.\\pipe\\MyPipe",
		PIPE_ACCESS_DUPLEX | FILE_FLAG_OVERLAPPED,
		0, 1, 1024, 1024, 0, NULL);
	if (INVALID_HANDLE_VALUE == hPipe){
		MessageBox(_T("创建命名管道失败"));
		hPipe = NULL;
		return;
	}

	// 创建匿名的人工重置对象
	HANDLE hEvent;
	hEvent = CreateEvent(NULL, TRUE, FALSE, NULL);
	if (!hEvent)
	{
		MessageBox(_T("创建事件对象失败!"));
		CloseHandle(hPipe);
		hPipe = NULL;
		return;
	}

	OVERLAPPED ovlap;
	ZeroMemory(&ovlap, sizeof(OVERLAPPED));
	ovlap.hEvent = hEvent;
	if (!ConnectNamedPipe(hPipe, &ovlap))
	{
		if (ERROR_IO_PENDING != GetLastError())
		{
			MessageBox(_T("等待客户端连接失败！"));
			CloseHandle(hPipe);
			CloseHandle(hEvent);
			hPipe = NULL;
			return;
		}
	}
	if (WAIT_FAILED == WaitForSingleObject(hEvent, INFINITE))
	{
		MessageBox(_T("等待对象失败！"));
		CloseHandle(hPipe);
		CloseHandle(hEvent);
		hPipe = NULL;
		return;
	}
	CloseHandle(hEvent);
}
```

创建完成后，服务端可以通过命名管道读写数据，实现方式和匿名管道一样。

### 3.2 客户端实现步骤

客户端首先要连接命名管道，通过调用WaitNamedPipe函数来完成，然后使用CreateFile函数开始创建命名管道文件。

```c++
void CNamedPipeCltView::OnPipeConnect()
{
	// TODO: Add your command handler code here
	if (!WaitNamedPipe("\\\\.\\pipe\\MyPipe", NMPWAIT_WAIT_FOREVER))
	{
		MessageBox(_T("当前没有可利用的命名管道实例!"));
		return;
	}
	hPipe = CreateFile("\\\\.\\pipe\\MyPipe", GENERIC_READ | GENERIC_WRITE,
		0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
	if (INVALID_HANDLE_VALUE == hPipe)
	{
		MessageBox(_T("打开命名管道失败！"));
		hPipe = NULL;
		return;
	}
}
```

客户端的读写命名管道可以服用前面匿名管道的实现。

## 4. 邮槽

邮槽是基于广播通信体系设计的，我理解应该是类似UDP协议的实现，可以一对多进行广播。邮槽的服务端和客户端实现比较简单。

示例代码：

```c++
void CMailslotSrvView::OnMailslotRecv()
{
	// TODO: Add your command handler code here
	HANDLE hMailslot;
	hMailslot = CreateMailslot("\\\\.\\mailslot\\MyMailslot", 0,
		MAILSLOT_WAIT_FOREVER, NULL);
	if (INVALID_HANDLE_VALUE == hMailslot)
	{
		MessageBox(_T("创建邮槽失败！"));
		return;
	}
	char buf[100];
	DWORD dwRead;
	if (!ReadFile(hMailslot, buf, 100, &dwRead, NULL))
	{
		MessageBox(_T("读取数据失败！"));
		CloseHandle(hMailslot);
		return;
	}
	MessageBox(buf);
	CloseHandle(hMailslot);
}

void CMailslotCltView::OnMailslotSend()
{
	// TODO: Add your command handler code here
	HANDLE hMailslot;
	hMailslot = CreateFile("\\\\.\\mailslot\\MyMailslot", GENERIC_WRITE,
		FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);
	if (INVALID_HANDLE_VALUE == hMailslot)
	{
		MessageBox(_T("打开邮槽失败！"));
		return;
	}
	char buf[] = "http://www.zte.com.cn";
	DWORD dwWrite;
	if (!WriteFile(hMailslot, buf, strlen(buf) + 1, &dwWrite, NULL))
	{
		MessageBox(_T("写入数据失败！"));
		CloseHandle(hMailslot);
		return;
	}
	CloseHandle(hMailslot);
}
```
