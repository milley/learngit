# 实现简单的控制台聊天室v2

在上一节完成了聊天室的登陆、注册功能，本节主要介绍其他功能的完善。

## 聊天室的增删改查

要聊天当然少不了聊天室的管理功能。这里是将用户分为管理员和普通用户，通过User类中的adminFlag标识来确定是否为管理员。如果是管理员则要提供创建聊天室、删除聊天室的菜单和功能，普通用户只有查找、加入、退出的菜单和功能。

ChatRoom类完成了这些基本操作及辅助操作。数据结构上，选择了std的map和list，分别存储房间号映射到加入的用户列表，房间列表。提供了一系列的操作：

```c++
class ChatRoom
{
public:
	ChatRoom();
	~ChatRoom(){}

	bool createChatRoom(string roomName);
	bool deleteChatRoom(Room room);
	bool deleteChatByRoomName(string roomName);

	list<Room> listChatRooms();
	list<User> listUsersByRoomId(int roomId);
	bool roomExists(string roomName);
	bool roomExists(int roomId);
	Room findRoomByName(string roomName);
	void joinChatRoom(Room room, User u);
	void exitChatRoom(Room room, User u);

	int getConnectCount(int roomId);

	void printRooms();

private:
	map<int, list<User>> roomIdToUsersMap;
	list<Room> rooms;
};
```

具体类的方法实现如下：

```c++
bool ChatRoom::createChatRoom(string roomName)
{
	if (roomExists(roomName) || roomName.length() == 0)
	{
		return false;
	}
	
	int id = 0;
	if (roomIdToUsersMap.size() > 0)
	{
		map<int, list<User> >::reverse_iterator iter = roomIdToUsersMap.rbegin();
		id = iter->first + 1;
	}
	else
	{
		id = 1;
	}

	Room room(id, roomName);
	rooms.push_back(room);
	return true;
}

bool ChatRoom::deleteChatByRoomName(string roomName)
{
	if (!roomExists(roomName))
	{
		return false;
	}
	return deleteChatRoom(findRoomByName(roomName));
}

bool ChatRoom::deleteChatRoom(Room room)
{
	if (!roomExists(room.getRoomName()))
	{
		return false;
	}

	int roomId = room.getRoomId();

	map<int, list<User> >::iterator iter;
	iter = roomIdToUsersMap.find(roomId);
	if (iter != roomIdToUsersMap.end() && iter->first == room.getRoomId())
	{
		cout << "RoomdId = " << roomId << " closing!!!" << endl;
		roomIdToUsersMap.erase(iter);
	}

	list<Room>::iterator it;
	for (it = rooms.begin(); it != rooms.end();)
	{
		if (room.getRoomId() == it->getRoomId())
		{
			it = rooms.erase(it);
		}
		else
		{
			++it;
		}
	}

	return true;
}

list<Room> ChatRoom::listChatRooms()
{
	return rooms;
}

Room ChatRoom::findRoomByName(string roomName)
{
	list<Room>::iterator it;
	for (it = rooms.begin(); it != rooms.end(); ++it)
	{
		if (roomName.compare(it->getRoomName()) == 0)
		{
			return *it;
		}
	}
	return Room();
}

bool ChatRoom::roomExists(string roomName)
{
	list<Room>::iterator it;
	for (it = rooms.begin(); it != rooms.end(); ++it)
	{
		if (roomName == it->getRoomName())
		{
			return true;
		}
	}
	return false;
}

bool ChatRoom::roomExists(int roomId)
{
	list<Room>::iterator it;
	for (it = rooms.begin(); it != rooms.end(); ++it)
	{
		if (roomId == it->getRoomId())
		{
			return true;
		}
	}
	return false;
}

int ChatRoom::getConnectCount(int roomId)
{
	return roomIdToUsersMap[roomId].size();
}

void ChatRoom::joinChatRoom(Room room, User u)
{
	for (list<Room>::iterator iter = rooms.begin(); iter != rooms.end(); iter++)
	{
		if (iter->getRoomId() == room.getRoomId())
		{
			roomIdToUsersMap[iter->getRoomId()].push_back(u);
		}
	}
	
}

void ChatRoom::exitChatRoom(Room room, User u)
{
	for (list<Room>::iterator iter = rooms.begin(); iter != rooms.end(); iter++)
	{
		if (iter->getRoomId() == room.getRoomId())
		{
			list<User> users = roomIdToUsersMap[iter->getRoomId()];
			for (list<User>::iterator uIter = users.begin(); uIter != users.end(); ++uIter)
			{
				if (uIter->getName() == u.getName())
				{
					users.erase(uIter);
				}
			}
		}
	}
}

void ChatRoom::printRooms() {
	for (list<Room>::iterator iter = rooms.begin(); iter != rooms.end(); iter++)
	{
		cout << "RoomId=" << iter->getRoomId() << ", RoomName=" << iter->getRoomName() << endl;
	}
}

list<User> ChatRoom::listUsersByRoomId(int roomId)
{
	if (roomExists(roomId))
	{
		return roomIdToUsersMap[roomId];
	}

	return list<User>();
}
```

## 服务端增加聊天功能和聊天室管理相关功能

在完成了ChatRoom类的基本功能后，就可以完成相应的服务端的功能了。服务端目前需要增加两块内容：

1. 聊天室管理功能
2. 登陆进去聊天室后聊天功能

第一点相对比较容易实现，在上一节介绍过的routeInfo逻辑中，给客户端暴露相应的聊天室管理的数据就可以了，大的逻辑上几乎没什么变动：

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
	else if (telenetResult.action.compare("Chat") == 0)
	{
		resp = getCurrentTime() + " " + telenetResult.user + " said:\n" + telenetResult.msg + "\n";
		cout << resp;
	}
	else if (telenetResult.action.compare("CreateRoom") == 0)
	{
		if (chatRoom.createChatRoom(telenetResult.msg))
		{
			resp = "CreateRoom|0|0";
		}
		else
		{
			resp = "CreateRoom|1|0";
		}
	}
	else if (telenetResult.action.compare("DeleteRoom") == 0)
	{
		if (chatRoom.deleteChatByRoomName(telenetResult.msg))
		{
			resp = "DeleteRoom|0|0";
		}
		else
		{
			resp = "DeleteRoom|1|0";
		}
	}
	else if (telenetResult.action.compare("SearchRoom") == 0)
	{
		if (telenetResult.msg.length() == 0)
		{
			// list all
			list<Room> rooms = chatRoom.listChatRooms();
			if (rooms.size() > 0)
			{
				resp = "SearchRoom|0|";
				for (list<Room>::iterator iter = rooms.begin();
					iter != rooms.end(); iter++)
				{
					resp += iter->getRoomName() + "\t";
				}
			}
			else
			{
				resp = "SearchRoom|1|0";
			}
		}
		else
		{
			if (chatRoom.roomExists(telenetResult.msg))
			{
				Room room = chatRoom.findRoomByName(telenetResult.msg);
				resp = "SearchRoom|0|" + room.getRoomName();
			}
			else
			{
				resp = "SearchRoom|1|0";
			}
		}
	}
	else if (telenetResult.action.compare("JoinRoom") == 0)
	{
		string userName = telenetResult.user;
		string roomName = telenetResult.msg;

		if (!loginService.userExists(userName) ||
			!chatRoom.roomExists(roomName))
		{
			resp = "JoinRoom|1|0";
		}
		else
		{
			if (chatRoom.getConnectCount(chatRoom.findRoomByName(roomName).getRoomId()) >= MAXCONN)
			{
				resp = "JoinRoom|1|2";
			}
			else
			{
				chatRoom.joinChatRoom(chatRoom.findRoomByName(roomName), loginService.findUserByName(userName));
				resp = "JoinRoom|0|" + userName + string(" join successful!");

				addSession(userName, socket);
			}
			
		}
	}
	else if (telenetResult.action.compare("ExitRoom") == 0)
	{
		string userName = telenetResult.user;
		string roomName = telenetResult.msg;
		if (!loginService.userExists(userName) ||
			!chatRoom.roomExists(roomName))
		{
			resp = "ExitRoom|1|0";
		}
		else
		{
			chatRoom.exitChatRoom(chatRoom.findRoomByName(roomName), loginService.findUserByName(userName));
			clearSession(userName);
			resp = "ExitRoom|0|" + userName + string(" exit successful!");
		}
	}

	
	//send(socket, resp.c_str(), resp.length(), 0);
	if (telenetResult.action.compare("Chat") == 0)
	{
		ChatResult chatResult;
		getChatResult(telenetResult.user, chatResult);
		
		string userName = chatResult.userName;
		string roomName = chatResult.roomName;

		sendMsg(roomName, resp);
	}
	else
	{
		send(socket, resp.c_str(), resp.length(), 0);
	}
}
```

上面传参是CreateRoom、DeleteRoom、SearchRoom、JoinRoom、ExitRoom都是和聊天室管理相关的功能。需要注意的是在加入房间和退出房间，多了session的操作。这里session就是存储用户名和socket句柄的映射关系。

```c++
void TelnetServer::addSession(string userName, SOCKET socket)
{
	session[userName] = socket;
}

void TelnetServer::clearSession(string userName)
{
	for (map<string, SOCKET>::iterator iter = session.begin();
		iter != session.end(); iter++)
	{
		if (iter->first.compare(userName) == 0)
		{
			session.erase(iter);
		}
	}
}

bool TelnetServer::sessionLive(string userName)
{
	for (map<string, SOCKET>::iterator iter = session.begin();
		iter != session.end(); iter++)
	{
		if (iter->first.compare(userName) == 0)
		{
			return true;
		}
	}
	return false;
}
```

在有了session的一系列操作，就剩下聊天功能了。聊天功能因为需要广播到加入到当前聊天室的每一个用户，因此收到发送来的聊天信息后解析用户名和聊天室名称，调用sendMsg来完成消息的广播。sendMsg就是通过房间名称取到所有房间内的用户，再通过用户名取到活动的socket句柄，发送到相应socket句柄：

```c++
void TelnetServer::sendMsg(string roomName, string resp)
{
	list<User> users = chatRoom.listUsersByRoomId(chatRoom.findRoomByName(roomName).getRoomId());
	for (list<User>::iterator iter = users.begin(); iter != users.end(); iter++)
	{
		for (map<string, SOCKET>::iterator iSer = session.begin();
			iSer != session.end(); iSer++)
		{
			if (sessionLive(iter->getName()) && iter->getName() == iSer->first)
			{
				send(session[iter->getName()], resp.c_str(), resp.length(), 0);
			}
		}
	}
}
```

## 客户端增加聊天功能和聊天室管理相关功能

客户端对于聊天室的增删改查相对比较容易，就是发送请求解析请求一系列操作：

```c++
void TelnetClient::createChatRoom()
{
	cout << "Input room name:";
	string roomName;
	cin >> roomName;

	netService(clientSocket, "CreateRoom", user.getName(), roomName);
	cout << "Received create chatroom info:" << message << endl;

	getMsgResult(msgResult);
	if (msgResult.action.compare("CreateRoom") == 0)
	{
		if (msgResult.flag.compare("0") == 0)
		{
			cout << roomName << " create successful!" << endl;
		}
		else
		{
			cout << roomName << " create failed!" << endl;
		}
	}
	
}

void TelnetClient::deleteChatRoom()
{
	cout << "Input room name:";
	string roomName;
	cin >> roomName;

	netService(clientSocket, "DeleteRoom", user.getName(), roomName);
	cout << "Received delete chatroom info:" << message << endl;

	getMsgResult(msgResult);
	if (msgResult.action.compare("DeleteRoom") == 0)
	{
		if (msgResult.flag.compare("0") == 0)
		{
			cout << roomName << " delete successful!" << endl;
		}
		else
		{
			cout << roomName << " delete failed!" << endl;
		}
	}
	
}

void TelnetClient::searchChatRoom()
{
	cout << "Input room name(press enter list all):";
	string roomName;
	cin >> roomName;

	netService(clientSocket, "SearchRoom", user.getName(), roomName);
	cout << "Received search chatroom info:" << message << endl;
	getMsgResult(msgResult);

	if (msgResult.action.compare("SearchRoom") == 0)
	{
		if (msgResult.flag.compare("0") == 0)
		{
			cout << acctResult.adminFlag << endl;
		}
		else
		{
			cout << roomName << " search failed!" << endl;
		}
	}
}

void TelnetClient::joinChatRoom()
{
	cout << "Please input room name:";
	string roomName;
	cin >> roomName;

	netService(clientSocket, "JoinRoom", user.getName(), roomName);
	cout << "Received search chatroom info:" << message << endl;
	getMsgResult(msgResult);
	if (msgResult.action.compare("JoinRoom") == 0)
	{
		if (msgResult.flag.compare("0") == 0)
		{
			cout << roomName << " join successful!" << endl;
			server(roomName);
		}
		else
		{
			if (msgResult.action.compare("1") == 0 &&
				msgResult.msg.compare("2") == 0)
			{
				cout << roomName << " has max connections!" << endl;
			}
			else
			{
				cout << roomName << " join failed!" << endl;
			}
		}
	}
}

void TelnetClient::exitChatRoom(string userName, string roomName)
{
	netService(clientSocket, "ExitRoom", user.getName(), roomName);
	cout << "Received exit chatroom info:" << message << endl;
	getMsgResult(msgResult);
	if (msgResult.action.compare("ExitRoom") == 0)
	{
		if (msgResult.flag.compare("0") == 0)
		{
			cout << roomName << " exit successful!" << endl;
		}
		else
		{
			cout << roomName << " exit failed!" << endl;
		}
	}
}
```

最核心的地方在于如何能接收服务端广播过来的消息并展示到客户端。由于前面所有的客户端代码都是同步阻塞的，因此服务端广播来的消息在没有刷新缓冲区是没法展示到客户端上的。如果要实时显示，我想使用多线程是最容易的做法，这里就加入线程来专门处理服务端广播的消息：

```c++
// TelnetClient.h
DWORD WINAPI Fun1Proc(LPVOID lpParameter);
```

```c++
// TelnetClient.cpp
DWORD WINAPI Fun1Proc(LPVOID lpParameter)
{
	while (TRUE)
	{
		char mesg[1024];
		memset(mesg, 0, sizeof(mesg));
		recv((SOCKET)lpParameter, mesg, bufsize, 0);
		if (strlen(mesg) <= 0)
		{
			break;
		}
		
		cout << mesg << endl;
	}
	return 0;
}

void TelnetClient::server(string roomName)
{
	hThread1 = CreateThread(NULL, 0, Fun1Proc, (LPVOID)clientSocket, 0, NULL);

	while (1)
	{
		cout << "请输入文字(exit()退出)：";
		cin >> message;
		if (strncmp(message, "exit()", strlen("exit()")) == 0)
		{
			break;
		}
		netService(clientSocket, "Chat", user.getName() + string("=>") + roomName, message);
		cout << message << endl;
	}
}
```

## 总结

以上就完成了一个简单的聊天室功能，没有使用xml/json等协议转换工具，没有使用数据库来存储信息。只是演示了如何完成一个聊天室的基本功能。
