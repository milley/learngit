# MFC创建一个简单的聊天程序

在复习多线程内容时，通过MFC编写一个简单的聊天程序。下面列下实现步骤。

## 1. 创建MFC程序界面

首先新建一个MFC对话框程序，通过拖拽2个GroupBox分别为接收数据和发送数据的组合框，2个EDIT_CONTROL分别代表相应的文本，1个IP ADDRESS CONTROL来放入IP信息，和最后一个发送按钮。

<img src="./img/chat_1.png" width="60%">

## 2. 创建并初始化套接字

在创建套接字之前，首先要加载套接字库，MFC提供AfxSocketInit函数，需要在Chat.cpp中的InitInstance中增加如下：

```c++
if (!AfxSocketInit())
{
    AfxMessageBox(_T("加载套接字库失败！"));
    return FALSE;
}
```

在加载完套接字库后，需要给CChatDlg类增加一个SOCKET类型的成员变量：m_socket，然后为CChatDlg类添加一个BOOL类型的成员函数InitSocket用来初始化套接字成员变量：

```c++
BOOL CChatDlg::InitSocket()
{
    // Create socket
    m_socket = socket(AF_INET, SOCK_DGRAM, 0);
    if (INVALID_SOCKET == m_socket)
    {
        MessageBox(_T("套接字创建失败！"), NULL, MB_OK);
        return FALSE;
    }

    SOCKADDR_IN addrSock;
    addrSock.sin_family = AF_INET;
    addrSock.sin_port = htons(6010);
    addrSock.sin_addr.S_un.S_addr = htonl(INADDR_ANY);

    int retval;
    // bind
    retval = bind(m_socket, (SOCKADDR*)&addrSock, sizeof(SOCKADDR));
    if (SOCKET_ERROR == retval)
    {
        closesocket(m_socket);
        MessageBox(_T("套接字绑定失败！"));
        return FALSE;
    }
    return TRUE;
}
```

在上面代码完成后，在对话框程序初始化函数OnInitDialog增加调用：

```c++
InitSocket();
```

## 3. 实现接收端功能

首先在ChatDlg.h中定义一个RECVPARAM结构体，用来保存套接字和句柄：

```c++
struct RECVPARAM
{
    SOCKET sock;
    HWND hwnd;
};
```

定义完成后再CChatDialog的OnInitDialog函数中增加线程的创建：

```c++
RECVPARAM *pRecvParam = new RECVPARAM;
pRecvParam->sock = m_socket;
pRecvParam->hwnd = m_hWnd;
// Create recv thread
HANDLE hThread = CreateThread(NULL, 0, RecvProc, (LPVOID)pRecvParam, 0, NULL);
CloseHandle(hThread);
```

线程创建完成后需要增加一个成员函数RecvProc来接收消息：

```c++
DWORD WINAPI CChatDlg::RecvProc( LPVOID lpParameter )
{
    // Receive main thread socket and hwnd
    SOCKET sock = ((RECVPARAM*)lpParameter)->sock;
    HWND hwnd = ((RECVPARAM*)lpParameter)->hwnd;
    delete lpParameter;

    SOCKADDR_IN addrFrom;
    int len = sizeof(SOCKADDR);

    char recvBuf[2000];
    char tempBuf[3000];

    int retval;
    while (TRUE)
    {
        retval = recvfrom(sock, recvBuf, sizeof(recvBuf), 0, (SOCKADDR*)&addrFrom, &len);
        if (SOCKET_ERROR == retval)
        {
            break;
        }
        sprintf(tempBuf, "%s 说: %s", inet_ntoa(addrFrom.sin_addr), recvBuf);
        ::PostMessage(hwnd, WM_RECVDATA, 0, (LPARAM)tempBuf);
    }

    return 0;
}
```

不要忘了在头文件中药相应的声明：

```c++
static DWORD WINAPI RecvProc(LPVOID lpParameter);
```

完成接收消息的功能后，需要增加消息映射，这里我们用自定义的消息映射方法。先在头文件增加：

```c++
#define WM_RECVDATA WM_USER+1
```

生成的消息映射函数增加：

```c++
afx_msg LRESULT OnRecvData(WPARAM wParam, LPARAM lParam);
DECLARE_MESSAGE_MAP()
```

源文件中调用消息映射函数：

```c++
BEGIN_MESSAGE_MAP(CChatDlg, CDialogEx)
    // ...
    ON_MESSAGE(WM_RECVDATA, OnRecvData)
    // ...
END_MESSAGE_MAP()
```

定义消息相应函数：

```c++
LRESULT CChatDlg::OnRecvData( WPARAM wParam, LPARAM lParam )
{
    CString str((char*)lParam);
    CString strTemp;

    GetDlgItemText(IDC_EDIT_RECV, strTemp);
    str += "\r\n";
    str += strTemp;

    SetDlgItemText(IDC_EDIT_RECV, str);
    return 0;
}
```

## 4. 实现发送功能

发送功能只需要将按钮的事件中增加实现功能：

```c++
void CChatDlg::OnBnClickedBtnSend()
{
    DWORD dwIP;
    ((CIPAddressCtrl*)GetDlgItem(IDC_IPADDRESS1))->GetAddress(dwIP);

    SOCKADDR_IN addrTo;
    addrTo.sin_family = AF_INET;
    addrTo.sin_port = htons(6010);
    addrTo.sin_addr.S_un.S_addr = htonl(dwIP);

    CString strSend;
    GetDlgItemText(IDC_EDIT_SEND, strSend);
    USES_CONVERSION;
    std::string editText(W2A(strSend));

    sendto(m_socket, editText.c_str(), strSend.GetLength() + 1, 0, (SOCKADDR*)&addrTo, sizeof(SOCKADDR));
    SetDlgItemText(IDC_EDIT_SEND, _T(""));
}
```

编译完成后就可以实现简单的聊天功能：

<img src="./img/chat_2.png" width="60%">

## 5. 问题总结

在实现过程中有出现字符接收不全的状况，主要原因就是在发送sendTo函数的第二个参数是用const char*来传递信息，之前使用强制类型转换：

```c++
sendto(m_socket, (LPSTR)(LPCTSTR)strSend, strSend.GetLength() + 1, 0, (SOCKADDR*)&addrTo, sizeof(SOCKADDR));
```

这里把CString强制转换为char *是有问题的，需要通过string的c_str()方法来完成转换。
