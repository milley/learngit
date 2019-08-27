# Windows网络编程

之前看过UNIX网络编程，最近因工作原因需要重新回顾下VC++的相关知识。手头放了一本很多年前放的VC++的书，正好拿来翻了翻关于Windows网络编程的部分，记录下以便后面还用得上。

## TCP应用程序

基于TCP的socket服务端程序步骤如下：

1. 创建socket
2. 绑定socket到本地地址和端口上
3. socket设置为监听模式
4. accept等待客户端过来接收请求
5. send/recv进行和客户端通信
6. 返回，等待下一个客户端请求
7. 关闭

基于TCP的socket客户端步骤如下：

1. 创建socket
2. 向服务端发送连接connect
3. send/recv和服务端通信
4. 关闭

### 主要函数

文档：

- <https://docs.microsoft.com/en-us/windows/win32/api/winsock2>

#### WSAStartup函数

Windows下加载套接字库和进行套接字库的版本协商。

```c++
int WSAStartup(
  _In_  WORD      wVersionRequested,
  _Out_ LPWSADATA lpWSAData
);
```

```help
Parameters
wVersionRequested [in]
The highest version of Windows Sockets specification that the caller can use. The high-order byte specifies the minor version number; the low-order byte specifies the major version number.

lpWSAData [out]
A pointer to the WSADATA data structure that is to receive details of the Windows Sockets implementation.

Return value
If successful, the WSAStartup function returns zero. Otherwise, it returns one of the error codes listed below.
```

#### socket函数

```c++
SOCKET WSAAPI socket(
  int af,
  int type,
  int protocol
);
```

socket函数来创建一个socket连接。

#### bind函数

```c++
int WSAAPI bind(
  SOCKET         s,
  const sockaddr *name,
  int            namelen
);
```

bind函数将本地地址绑定到socket上。

#### inet_addr函数

```c++
unsigned long WSAAPI inet_addr(
  const char *cp
);
```

inet_addr将字符串转换成IPv4的地址。

#### inet_ntoa函数

```c++
char *WSAAPI inet_ntoa(
  in_addr in
);
```

和inet_addr相反的操作，将IPv4的类型转换到字符串。

#### listen函数

```c++
int WSAAPI listen(
  SOCKET s,
  int    backlog
);
```

监听socket连接。

#### accept函数

```c++
SOCKET WSAAPI accept(
  SOCKET   s,
  sockaddr *addr,
  int      *addrlen
);
```

accept函数接受客户端发送的连接请求。

#### send函数

```c++
int WSAAPI send(
  SOCKET     s,
  const char *buf,
  int        len,
  int        flags
);
```

通过socket发送数据。

#### recv函数

```c++
int WSAAPI recv(
  SOCKET s,
  char   *buf,
  int    len,
  int    flags
);
```

从已连接的socket接收数据。

#### connect函数

```c++
int WSAAPI connect(
  SOCKET         s,
  const sockaddr *name,
  int            namelen
);
```

与特定的套接字建立连接。

#### recvfrom

```c++
int WSAAPI recvfrom(
  SOCKET   s,
  char     *buf,
  int      len,
  int      flags,
  sockaddr *from,
  int      *fromlen
);
```

接收一个数据报信息并保存源地址。

#### sendto函数

```c++
int WSAAPI sendto(
  SOCKET         s,
  const char     *buf,
  int            len,
  int            flags,
  const sockaddr *to,
  int            tolen
);
```

给指定的连接发送信息。

#### htons和htonl函数

```c++
u_short WSAAPI htons(
  u_short hostshort
);

u_long WSAAPI htonl(
  u_long hostlong
);
```

hton将u_short类型的值从主机字节顺序转换为网络字节序。

htonl将u_long类型的值从主机字节顺序转换为网络字节序。

### TCP服务端程序

```c++
#include <WinSock2.h>
#include <stdio.h>

void main()
{
    WORD wVersionRequested;
    WSADATA wsaData;
    int err;

    wVersionRequested = MAKEWORD(1, 1);

    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0) {
        return;
    }

    if (LOBYTE(wsaData.wVersion) != 1 ||
        HIBYTE(wsaData.wVersion) != 1) {
        WSACleanup();
        return;
    }

    SOCKET sockSrv = socket(AF_INET, SOCK_STREAM, 0);

    SOCKADDR_IN addrSrv;
    addrSrv.sin_addr.S_un.S_addr = htonl(INADDR_ANY);
    addrSrv.sin_family = AF_INET;
    addrSrv.sin_port = htons(6000);

    bind(sockSrv, (SOCKADDR *)&addrSrv, sizeof(SOCKADDR));
    listen(sockSrv, 5);

    SOCKADDR_IN addrClient;
    int len = sizeof(SOCKADDR);

    while (1) {
        SOCKET sockConn = accept(sockSrv, (SOCKADDR *)&addrClient, &len);
        char sendBuf[100];
        sprintf(sendBuf, "Welcome %s to http://www.zte.com.cn", inet_ntoa(addrClient.sin_addr));

        send(sockConn, sendBuf, strlen(sendBuf) + 1, 0);
        char recvBuf[100];
        recv(sockConn, recvBuf, 100, 0);
        printf("%s\n", recvBuf);

        closesocket(sockConn);
    }
}
```

### TCP客户端程序

```c++
#include <WinSock2.h>
#include <stdio.h>

void main()
{
    WORD wVersionRequested;
    WSADATA wsaData;
    int err;

    wVersionRequested = MAKEWORD(1, 1);

    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0) {
        return;
    }

    if (LOBYTE(wsaData.wVersion) != 1 ||
        HIBYTE(wsaData.wVersion) != 1) {
        WSACleanup();
        return;
    }

    SOCKET sockClient = socket(AF_INET, SOCK_STREAM, 0);

    SOCKADDR_IN addrSrv;
    addrSrv.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
    addrSrv.sin_family = AF_INET;
    addrSrv.sin_port = htons(6000);

    connect(sockClient, (SOCKADDR *)&addrSrv, sizeof(SOCKADDR));

    char recvBuf[100];
    recv(sockClient, recvBuf, 100, 0);
    printf("%s\n", recvBuf);

    send(sockClient, "This is lisi", strlen("This is lisi") + 1, 0);

    closesocket(sockClient);
    WSACleanup();
}
```

## UDP应用程序

### UDP服务端程序

```c++
#include <WinSock2.h>
#include <stdio.h>

void main()
{
    WORD wVersionRequested;
    WSADATA wsaData;
    int err;

    wVersionRequested = MAKEWORD(1, 1);

    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0) {
        return;
    }

    if (LOBYTE(wsaData.wVersion) != 1 ||
        HIBYTE(wsaData.wVersion) != 1) {
            WSACleanup();
            return;
    }

    SOCKET sockSrv = socket(AF_INET, SOCK_DGRAM, 0);
    SOCKADDR_IN    addrSrv;
    addrSrv.sin_addr.S_un.S_addr = htonl(INADDR_ANY);
    addrSrv.sin_family = AF_INET;
    addrSrv.sin_port = htons(7000);

    bind(sockSrv, (SOCKADDR *)&addrSrv, sizeof(SOCKADDR));


    SOCKADDR_IN addrClient;
    int len = sizeof(SOCKADDR);
    char recvBuf[100];

    //while (1) {
    printf("======\n");
    memset(recvBuf, 0x00, sizeof(recvBuf));
    recvfrom(sockSrv, recvBuf, 100, 0, (SOCKADDR *)&addrClient, &len);
    printf("======\n");
    printf("%s\n", recvBuf);
    //}

    closesocket(sockSrv);
    WSACleanup();
}
```

### UDP客户端程序

```c++
#include <WinSock2.h>
#include <stdio.h>

void main()
{
    WORD wVersionRequested;
    WSADATA wsaData;
    int err;

    wVersionRequested = MAKEWORD(1, 1);
    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0) {
        return;
    }

    if (LOBYTE(wsaData.wVersion) != 1 ||
        HIBYTE(wsaData.wVersion) != 1) {
            WSACleanup();
            return;
    }

    SOCKET sockClient = socket(AF_INET, SOCK_DGRAM, 0);
    SOCKADDR_IN    addrSrv;
    addrSrv.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
    addrSrv.sin_family = AF_INET;
    addrSrv.sin_port = htons(7000);

    sendto(sockClient, "Hello", strlen("Hello") + 1, 0,
        (SOCKADDR *)&addrSrv, sizeof(SOCKADDR));
    closesocket(sockClient);
    WSACleanup();
}
```

## 基于UDP的聊天室

### 聊天室服务端

```c++
#include <WinSock2.h>
#include <stdio.h>

void main()
{
    WORD wVersionRequested;
    WSADATA wsaData;
    int err;

    wVersionRequested = MAKEWORD(1, 1);

    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0) {
        return;
    }

    if (LOBYTE(wsaData.wVersion) != 1 ||
        HIBYTE(wsaData.wVersion) != 1) {
        WSACleanup();
        return;
    }

    SOCKET sockSrv = socket(AF_INET, SOCK_DGRAM, 0);

    SOCKADDR_IN    addrSrv;
    addrSrv.sin_addr.S_un.S_addr = htonl(INADDR_ANY);
    addrSrv.sin_family = AF_INET;
    addrSrv.sin_port = htons(7000);

    bind(sockSrv, (SOCKADDR *)&addrSrv, sizeof(SOCKADDR));

    char recvBuf[100];
    char sendBuf[100];
    char tempBuf[100];

    memset(recvBuf, 0x00, 100);
    memset(sendBuf, 0x00, 100);
    memset(tempBuf, 0x00, 100);

    SOCKADDR_IN addrClient;
    int len = sizeof(SOCKADDR);

    while (1) {
        recvfrom(sockSrv, recvBuf, 100, 0, (SOCKADDR *)&addrClient, &len);
        if ('q' == recvBuf[0]) {
            sendto(sockSrv, "q", strlen("q") + 1, 0, (SOCKADDR *)&addrClient, len);
            printf("Chat end!\n");
            break;
        }
        sprintf(tempBuf, "%s say : %s", inet_ntoa(addrClient.sin_addr), recvBuf);
        printf("%s\n", tempBuf);
        printf("Please input data:\n");
        gets(sendBuf);
        sendto(sockSrv, sendBuf, strlen(sendBuf) + 1, 0, (SOCKADDR *)&addrClient, len);
    }

    closesocket(sockSrv);
    WSACleanup();
}

```

### 聊天室客户端

```c++
#include <WinSock2.h>
#include <stdio.h>

void main()
{
    WORD wVersionRequested;
    WSADATA wsaData;
    int err;

    wVersionRequested = MAKEWORD(1, 1);

    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0)
    {
        return;
    }

    if (LOBYTE(wsaData.wVersion) != 1 ||
        HIBYTE(wsaData.wVersion) != 1)
    {
        WSACleanup();
        return;
    }

    SOCKET sockClient = socket(AF_INET, SOCK_DGRAM, 0);

    SOCKADDR_IN addrSrv;
    addrSrv.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
    addrSrv.sin_family = AF_INET;
    addrSrv.sin_port = htons(7000);

    char recvBuf[100];
    char sendBuf[100];
    char tempBuf[100];

    memset(recvBuf, 0x00, 100);
    memset(sendBuf, 0x00, 100);
    memset(tempBuf, 0x00, 100);

    int len = sizeof(SOCKADDR);

    while (1)
    {
        printf("Please input data:\n");
        gets(sendBuf);
        sendto(sockClient, sendBuf, strlen(sendBuf) + 1, 0,
            (SOCKADDR *)&addrSrv, len);

        recvfrom(sockClient, recvBuf, 100, 0, (SOCKADDR *)&addrSrv, &len);

        if ('q' == recvBuf[0])
        {
            sendto(sockClient, "q", strlen("q") + 1, 0, (SOCKADDR *)&addrSrv, len);
            printf("Chat end!\n");
            break;
        }
        sprintf(tempBuf, "%s say : %s", inet_ntoa(addrSrv.sin_addr), recvBuf);
        printf("%s\n", tempBuf);
    }

    closesocket(sockClient);
    WSACleanup();
}
```
