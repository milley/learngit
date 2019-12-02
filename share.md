# 实现简单的控制台聊天室

近期完成一个练手小项目，控制台版本的聊天室，有以下功能：

- [ ] 每个客户端可以用使用telnet ip:port的方式连接到服务器上。
- [ ] 新连接需要用用户名和密码登录，如果没有，则需要注册一个。
- [ ] 然后可以选择一个聊天室加入聊天。
- [ ] 管理员有权创建或删除聊天室，普通人员只有加入、退出、查询聊天室的权力。
- [ ] 聊天室需要有人数限制，每个人发出来的话，其它所有的人都要能看得到。

要完成以上功能，首先从分析下需要实现两部分：

- 客户端：登陆/注册/操作聊天室/发起聊天会话等
- 服务端：给客户端所有的功能提供后台服务

由于实现一个最简单的聊天室功能，所以就没有想着引入控制复杂协议的工具(json/xml等等)。协议就自定义一个最简单的文本串，以"|"来分隔不同的操作。数据保存也不引入数据库，直接存放到内存中。

考虑下以上功能，我们可以先将大体的数据结构设计出来。以目前的需求来看，我认为至少需要设计如下几个实体：

| 类 | 作用 |
|---| ----- |
| User | 用户实体类 |
| Room | 房间实体类 |
| ChatRoom | 聊天室类，保存房间ID对应到的当前用户集合、保存所有的已创建房间 |
| LoginService | 登陆服务，保存所有的用户列表 |

客户端和服务端需要相互通讯，由于TCP的可靠性优点，这里使用TCP作为通讯协议。由于开发的环境是在win7上使用C++，因此整个系统设计分为3个模块：

- 公共组件：包括用户、房间、聊天室、登陆服务的操作
- 客户端程序
- 服务端程序

## 1. 实现用户操作的LoginService

作为LoginService，操作的都是和User相关。我们需要先定义User的实体类：

```c++
class User
{
public:
	User(std::string n, std::string p) : name(n), passwd(p), adminFlag(false) {}
	User() : adminFlag(false) {}
	~User() {}

	std::string getName()
	{
		return name;
	}

	void setName(std::string nm)
	{
		name = nm;
	}

	std::string getPasswd()
	{
		return passwd;
	}

	void setPassword(std::string password)
	{
		passwd = password;
	}

	bool getAdminFlag()
	{
		return adminFlag;
	}

	void setAdminFlag(bool flag)
	{
		adminFlag = flag;
	}

	friend std::ostream& operator<<(std::ostream& out, const User& u)
	{
		out << "[Username=" << u.name << "|Password=" << u.passwd << "|AdminFlag=" << (u.adminFlag ? "true" : "false") << "]";
		return out;
	}


private:
	std::string name;
	std::string passwd;
	bool adminFlag;
};
```

LoginService的功能需要实现login验证用户名密码、注册新用户，可以扩展下相应的操作，比如：判断用户是否存在、根据名称查找用户实例、打印用户：

```c++
class LoginService
{
public:
	LoginService();
	~LoginService() {}


	bool login(string name, string passwd)
	{
		for (vector<User>::iterator iter = users.begin(); iter != users.end(); iter++)
		{
			if (iter->getName() == name && iter->getPasswd() == passwd)
			{
				return true;
			}
		}

		return false;
	}

	bool userExists(string name)
	{
		for (vector<User>::iterator iter = users.begin(); iter != users.end(); iter++)
		{
			if (iter->getName() == name)
			{
				return true;
			}
		}

		return false;
	}

	User findUserByName (string name)
	{
		vector<User>::iterator iter = users.begin();
		while (iter != users.end())
		{
			if (iter->getName() == name)
			{
				return *iter;
			}
			iter++;
		}
		return User();
	}

	bool login(User u)
	{
		return login(u.getName(), u.getPasswd());
	}

	void signIn()
	{
		string name, password;
		cout << "Please input user name:";
		cin >> name;
		cout << "Please input password:";
		cin >> password;
		User u(name, password);
		addUser(u);
	}

	void printUsers()
	{
		for (vector<User>::iterator iter = users.begin(); iter != users.end(); iter++)
		{
			cout << *iter << endl;
		}
	}

	void addUser(User u)
	{
		if (!userExists(u.getName()))
		{
			users.push_back(u);
		}
	}

private:
	void loadUser()
	{
		User aaa("aaa", "aaa");
		User bbb("bbb", "bbb");
		User ccc("ccc", "ccc");
		User root("root", "root");
		root.setAdminFlag(true);
		users.push_back(aaa);
		users.push_back(bbb);
		users.push_back(ccc);
		users.push_back(root);
	}
	

private:
	vector<User> users;
};
```

## 2. 实现客户端基本功能

客户端要访问服务端，首先需要拿到服务端的ip和port，然后通过socket访问服务端。因此第一步就是在main方法中拿到相应的ip和port，可以通过命令行参数传递进来：

```c++
string ipAndPort = argv[1];
//string ipAndPort = "127.0.0.1:9999";
size_t colonIndex = ipAndPort.find_first_of(":");
string ip = ipAndPort.substr(0, colonIndex);
string port = ipAndPort.substr(colonIndex + 1, ipAndPort.length() - colonIndex);

cout << ip << ":" << port << endl;

telnetClient.init(ip, (unsigned short)std::stoi(port));
```

windows平台调用WSAStartup开启套接字，然后调用socket函数来设置套接字类型并获取套接字句柄，接下来设置协议族、ip、端口调用connect来进行socket连接：

```c++
void TelnetClient::init(string ip, unsigned short port)
{
	if (WSAStartup(MAKEWORD(1, 1), &wsadata) != 0)
	{
		cout << "WSAStartup() error" << endl;
	}

	if ((clientSocket = socket(PF_INET, SOCK_STREAM, 0)) == INVALID_SOCKET)
	{
		cout << "socket() error" << endl;
	}

	serverAddr.sin_family = AF_INET;
	serverAddr.sin_addr.S_un.S_addr = inet_addr(ip.c_str());
	serverAddr.sin_port = htons(port);

	if (connect(clientSocket, (SOCKADDR *)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR)
	{
		cout << "connect() error" << endl;
	}
}
```

init执行成功后客户端和服务端连接成功，接下来弹出菜单来选择操作"[login:1 signIn:2 quit:0]"：

```c++
while (1)
{
	int menuId = 0;
	cout << "Please select menu action[login:1 signIn:2 quit:0]";
	cin >> menuId;
	if (1 == menuId)
	{
		if (telnetClient.login())
		{
			if (telnetClient.getUser().getAdminFlag())
			{
				while (1)
				{
					cout << "Login successful!Please select chatroom operation[join:1 upper menu:2 search:3 create:4 delete:5]";
					int oper = 2;
					cin >> oper;
					if (oper == 2)
					{
						break;
					}
					switch (oper)
					{
					case 1:
						//join
						telnetClient.joinChatRoom();
						break;
					case 3:
						// search
						telnetClient.searchChatRoom();
						break;
					case 4:
						telnetClient.createChatRoom();
						break;
					case 5:
						// delete
						telnetClient.deleteChatRoom();
						break;
					default:
						cout << "Input error" << endl;
						break;
					}
				}
			}
			else
			{
				cout << "Login successful!Please select chatroom operation[join:1 exit:2 search:3]";
				int oper = 2;
				cin >> oper;
				if (oper == 2)
				{
					break;
				}
				switch (oper)
				{
				case 1:
					//join
					telnetClient.joinChatRoom();
					break;
				case 3:
					// search
					telnetClient.searchChatRoom();
					break;
				default:
					cout << "Input error" << endl;
					break;
				}
			}
		}
	}
	else if (2 == menuId)
	{
		telnetClient.signIn();
	}
	else if (0 == menuId)
	{
		break;
	}
	break;
}
```

这种菜单一般采用while(1)来控制菜单选择，如果选择1则进入相应的login逻辑，选择2则进入注册signIn逻辑，选择0则退出。

执行0退出，则需要断开socket连接，通过封装的unInit方法来执行：

```c++
void TelnetClient::unInit()
{
	CloseHandle(hThread1);

	closesocket(clientSocket);
	WSACleanup();
}
```

这里调用的CloseHandle是关闭聊天的多线程句柄。这个句柄的打开在后面讲到。

## 3. 实现服务端基本功能

服务端首先也要建立socket连接，这里同client封装为init方法，首先调用WSAStartup开启套接字，执行socket获取套接字句柄，绑定ip为任意ip，端口固定为9999，调用bind函数来将socket句柄和serverAddr地址相关SOCKADD_IN结构：

```c++
void TelnetServer::init()
{
	if (WSAStartup(MAKEWORD(1, 1), &wsadata) != 0)
	{
		cout << "WSAStartup() error" << endl;
	}

	serverSocket = socket(PF_INET, SOCK_STREAM, 0);
	if (serverSocket == INVALID_SOCKET)
	{
		cout << "socket() error" << endl;
	}

	memset(&serverAddr, 0, sizeof(serverAddr));
	serverAddr.sin_family = AF_INET;
	serverAddr.sin_addr.S_un.S_addr = htonl(INADDR_ANY);
	serverAddr.sin_port = htons(9999);

	if (bind(serverSocket, (SOCKADDR *)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR)
	{
		cout << "bind() error" << endl;
	}
}
```

绑定成功后，这里将服务端的核心逻辑都封装为server方法，主要是监听套接字，通过调用select函数来进行多路复用，通过调用FD_ISSET来判断socket句柄是否在select的socket set中。如果是父进程则调用accept准备接收，如果是子进程则调用recv函数接收客户端发来的消息：

```c++
void TelnetServer::server()
{
	listen(serverSocket, MAXCONN);
	cout << "Server start successful!" << endl;

	FD_ZERO(&reads);
	FD_SET(serverSocket, &reads);

	while (1)
	{
		cpyReads = reads;
		timeout.tv_sec = 5;
		timeout.tv_usec = 5000;

		if ((fdnum = select(0, &cpyReads, 0, 0, &timeout)) == SOCKET_ERROR)
		{
			break;
		}
		if (fdnum == 0)
		{
			continue;
		}
		for (unsigned i = 0; i < reads.fd_count; i++)
		{
			if (FD_ISSET(reads.fd_array[i], &cpyReads))
			{
				if (reads.fd_array[i] == serverSocket)
				{
					szClientAddr = sizeof(clientAddr);
					clientSocket = accept(serverSocket, (SOCKADDR *)&clientAddr, &szClientAddr);
					if (clientSocket == INVALID_SOCKET)
					{
						cout << "accept() error" << endl;
					}
					FD_SET(clientSocket, &reads);
					cout << "Connecting client is:" << clientSocket << endl;
				}
				else
				{
					memset(message, 0, bufsize);
					str_len = recv(reads.fd_array[i], message, bufsize - 1, 0);
					cout << "<<<<" << message << endl;

					if (str_len <= 0)
					{
						FD_CLR(reads.fd_array[i], &reads);
						closesocket(cpyReads.fd_array[i]);
						cout << "Closing client is:" << cpyReads.fd_array[i] << endl;
					}
					else
					{
						if (trim(string(message)).length() > 0)
						{
							cout << "Receive:" << message << endl;
							routeInfo(reads.fd_array[i], message, str_len);
						}
					}
				}
			}
		}
	}
	unInit();
}
```

routeInfo是服务端接入业务层的入口，根据传入的message解析成不同的业务动作：

```c++
void TelnetServer::getTelenetResult(string message, TelenetResult &telenetResult)
{
	size_t firstSplitIndex = message.find(SPLITTER);
	size_t secondSplitIndex = message.find(SPLITTER, firstSplitIndex + 1);

	string action = message.substr(0, firstSplitIndex);
	string username = message.substr(firstSplitIndex + 1, secondSplitIndex - firstSplitIndex - 1);
	string msg = message.substr(secondSplitIndex + 1, message.length() - secondSplitIndex - 1);


	telenetResult.action = action;
	telenetResult.user = username;
	telenetResult.msg = msg;
}
```

如果action是"Login"则处理相关的登陆逻辑，如果action是"SignIn"则处理相关的注册逻辑：

```c++
void TelnetServer::routeInfo(SOCKET socket, string message, int len)
{
	string resp;
	getTelenetResult(message, telenetResult);
	cout << "action=" << telenetResult.action << endl;
	cout << "username=" << telenetResult.user << endl;
	cout << "info=" << telenetResult.msg << endl;

	if (telenetResult.action.compare("Login") == 0)
	{
		if (!loginService.userExists(telenetResult.user))
		{
			cout << "Username = " << telenetResult.user << " doesn't exists, Please SignIn!" << endl;
			resp = "Login|2|0";
		}
		else
		{
			User user = User(telenetResult.user, telenetResult.msg);

			if (login(user))
			{
				resp = "Login|0|";
				if (loginService.findUserByName(user.getName()).getAdminFlag())
				{
					resp = resp + "1";
				}
				else
				{
					resp = resp + "0";
				}
			}
			else
			{
				resp = "Login|1|0";
			}
		}
	}
    else if (telenetResult.action.compare("SignIn") == 0)
	{
		User user = User(telenetResult.user, telenetResult.msg);
		if (loginService.userExists(telenetResult.user))
		{
			resp = "SignIn|2|0";
		}
		else
		{
			if (telenetResult.user.length() == 0 || telenetResult.msg.length() == 0 ||
				telenetResult.user.find("|", 0) != std::string::npos ||
				telenetResult.msg.find("|", 0) != std::string::npos)
			{
				loginService.addUser(user);
				resp = "SignIn|0|0";
			}
			else
			{
				resp = "SignIn|1|0";
			}
		}
	}
    //...
}
```

在server方法的最后调用unInit来释放连接：

```c++
void TelnetServer::unInit()
{
	closesocket(clientSocket);
	closesocket(serverSocket);
	WSACleanup();
}
```

## 4. 本章节总结

本章节主要实现了用户登陆、注册相关的LoginService业务逻辑和网络编程相应的基础框架，可以实现用户的登陆、注册操作。后期再详述剩余的功能。
