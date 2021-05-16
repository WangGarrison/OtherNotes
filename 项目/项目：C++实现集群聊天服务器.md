# Json

TCP：面向连接的、可靠的字节流服务

客户端将要发送的数据序列化成字节流

服务器端将字节流反序列化成数据

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210209154143876.png" alt="image-20210209154143876" style="zoom:50%;" />

**Json介绍**

Json是一种轻量级的数据交换格式（也叫数据序列化方式）。Json采用==完全独立于编程语言==的文本格式来存储和表示数据。简洁和清晰的层次结构使得 Json 成为理想的数据交换语言。 易于人阅读和编写，同时也易于机器解析和生成，并有效地提升网络传输效率  （比xml效率高，没有protobuf效率高，但比protobuf简单）

**一个优秀的Json三方库**

JSON for Modern C++ 是一个由德国大牛 nlohmann 编写的在 C++ 下使用的 JSON 库。

具有以下特点

- 直观的语法
- 整个代码由一个头文件组成 json.hpp，没有子项目，没有依赖关系，没有复杂的构建系统，使用
  起来非常方便
- 使用 C++ 11 标准编写
- 使用 json 像使用 STL 容器一样
- STL 和 json 容器之间可以相互转换
- 严谨的测试：所有类都经过严格的单元测试，覆盖了 100％ 的代码，包括所有特殊的行为。此
  外，还检查了 Valgrind 是否有内存泄漏。为了保持高质量，该项目遵循核心基础设施倡议(CII)
  的最佳实践  

**Json序列化和反序列化**

- 序列化：定义Json对象，使用==dump==将Json对象序列化成字符串
- 反序列化：使用==parse==将字符串反序列化成Json对象

![image-20210209182805547](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210209182805547.png)![image-20210209171022881](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210209171022881.png)

# muduo网络库

![image-20210209174112351](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210209174112351.png)

muduo网络库封装了epoll + 线程池，向外暴露用户的连接和断开、用户的可读写事件，提供了两个类

好处：把网络I/O和业务的代码区分开

- TcpServer类：用于编写服务器程序的
- TcpClient类：用于编写客户端程序的

**muduo库的使用代码示例**

```cpp
#include <muduo/net/TcpServer.h>
#include <muduo/net/EventLoop.h>
#include <functional> //绑定器头文件
#include <iostream>
#include <string>
using namespace std;
using namespace muduo;
using namespace muduo::net;
using namespace placeholders;

/*基于muduo网络库开发服务器程序
1. 组合TcpServer对象
2. 创建EventLoop事件循坏对象的指针, 就是你的epoll
3. 明确TcpServer构造函数需要什么参数，输出ChatServer的构造函数
4. 在当前服务器类的构造函数当中，注册处理连接的的回调函数和处理读写事件的回调函数
5. 设置合适的服务端线程数量，muduo库会自己分配I/O线程和工作线程
*/

class ChatServer
{
public:
    ChatServer(EventLoop *loop,               //事件循环，Reator反应堆
               const InetAddress &listenAddr, //IP地址+端口Port
               const string &nameArg)         //给server起个名字
        : _server(loop, listenAddr, nameArg), _loop(loop)
    {
        //给服务器注册用户连接的创建和断开回调
        _server.setConnectionCallback(std::bind(&ChatServer::onConnection, this, _1));

        //给服务器注册用户读写事件回调
        _server.setMessageCallback(std::bind(&ChatServer::onMessage, this, _1, _2, _3));

        //设置服务器端的线程数量, 1个I/O线程，3个工作线程
        _server.setThreadNum(4);
    }

    //开启事件循环
    void start()
    {
        _server.start();
    }

private:
    //专门处理用户的连接创建和断开  底层：epoll  listenfd  accept
    void onConnection(const TcpConnectionPtr &conn) //写成成员方法是因为想访问对象的成员变量
    {
        if (conn->connected())
        {
            cout << conn->peerAddress().toIpPort() << " -> " << conn->localAddress().toIpPort() << "state : online" << endl;
        }
        else
        {
            cout << conn->peerAddress().toIpPort() << " -> " << conn->localAddress().toIpPort() << "state : offline" << endl;
            conn->shutdown();  //close(fd);
            //_loop->quit();  //退出epoll
        }
    }

    //专门处理用户的读写事件
    void onMessage(const TcpConnectionPtr &conn, //连接
                   Buffer *buffer,                  //缓冲区
                   Timestamp time)               //接收到数据的时间信息
    {
        string buf = buffer->retrieveAllAsString();
        cout<<"recv data: "<<buf<<"time: "<<time.toString()<<endl;
        conn->send(buf);
    }
    TcpServer _server; //1
    EventLoop *_loop;  //2 事件循环的指针epoll
};

int main()
{
    EventLoop loop; //epoll
    InetAddress addr("127.0.0.1", 6000);  //回送地址
    ChatServer server(&loop, addr, "ChatServer");

    server.start();  //启动服务器：将listenfd通过epoll_ctl添加到epfd上
    loop.loop();  //epoll_wait以阻塞方式等待新用户连接、已连接用户的读写事件等
    return 0;
}
```

 **编译testmuduo**

![image-20210210122211269](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210122211269.png)

先写`-lmuduo_net`，因为muduo_base也依赖了muduo_net

或者在配置文件里加上这三个命令，就不用每次都写了，如下：

![image-20210210125300675](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210125300675.png)

然后ctrl + shift + b点击即可生成可执行文件

![image-20210210125517655](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210125517655.png)

**测试代码**

![image-20210210122452230](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210122452230.png)

![image-20210210123612525](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210123612525.png)

# CMake

g++一般使用方式：

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210132341815.png" alt="image-20210210132341815" style="zoom: 50%;" />

CMake使用简单方便，可以跨平台，构建项目编译环境。尤其比直接写Makefile简单（在构建大型工程编译
时，需要写大量的文件依赖关系），可以通过简单的CMake生成负责的Makefile文件。  

CMake要使用首先写配置文件`CMakeLists.txt`，配置的东西对应于g++的6个栏

![image-20210210134950606](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210134950606.png)

配置好CMakeLists.txt后执行cmake命令，最后再执行make命令

- cmake会去指定路径下找CMakeLists.txt
- make会去找Makefile

![image-20210210135907745](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210135907745.png)

```shell
# 设置可执行文件最终存储的路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin) 
#PROHECT_SOURCE_DIR是工程的根目录，也就是~/Desktop/project/chat_server/testmuduo
```

# MySQL

用户数据可以是一两万级别（集群3台服务器可以达到5~6万），百万级别就要涉及到其他数据库的技术了，

**创建user表**

user表：存用户的详细信息：id，name，password，state

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210164221144.png" alt="image-20210210164221144" style="zoom: 50%;" />

```mysql
create table user(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE, 
    password VARCHAR(50) NOT NULL, 
    state ENUM('online', 'offline') DEFAULT 'offline');
```

**创建friend表**

friend表：存用户id和该用户对应的好友的id

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210164313406.png" alt="image-20210210164313406" style="zoom:50%;" />

```mysql
create table friend(
	userid INT NOT NULL,
	friendid INT NOT NULL,
	PRIMARY KEY(userid, friendid));
```

**创建allgroup表**

allgroup表：存储群组的基本信息，群主id、群名、群简介

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210164748243.png" alt="image-20210210164748243" style="zoom:50%;" />

```mysql
create table allgroup(
	id INT PRIMARY KEY AUTO_INCREMENT,
	groupname VARCHAR(50) NOT NULL UNIQUE,
	groupdesc VARCHAR(200) DEFAULT '');
```

**创建groupuser表**

groupuser表：存一个群的群员信息，群员id、群员角色（群主，普通成员）

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210165056328.png" alt="image-20210210165056328" style="zoom:50%;" />

```mysql
create table groupuser(
	groupid int not null,
	userid int not null,
	grouprole enum('creator', 'normal') default 'normal',
	primary key(groupid, userid));
```

**创建offlinemessage表**

offlinemessage表：存储用户的离线消息，id: message

注意：该表里userid不能设置为primary key，因为用户可能收到一个用户的多条离线消息，如果设置为主键那就只能收到对方用户的一条离线消息，如果对方再想发送离线消息，就会插入该表失败

虽然该表没有主键，但是InnoDB存储引擎会给该表添加一个主键

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210165354430.png" alt="image-20210210165354430" style="zoom:50%;" />

```mysql
create table offlinemessage(
	userid INT NOT NULL,
	message VARCHAR(500) NOT NULL);
```

**返回用户好友列表**

- friend表：存用户id和该用户对应的好友的id
- user表：存用户的详细信息：id，name，password，state

联合friend表和userid表 查userid的好友的详细信息

![image-20210221104511297](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210221104511297.png)

通过b.friendid = a.id把两张表连接起来，在连接起来的表里筛选b.userid=userid，这样就查出来了用户1的好友2的详细信息

![image-20210221105457790](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210221105457790.png)

![image-20210221110114461](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210221110114461.png)

**查询用户所在群组的成员列表**

- groupuser表：存一个群的群员信息，群员id、群员角色（群主，普通成员）
- allgroup表：存储群组的基本信息，群主id、群名、群简介

1、先根据userid在groupuser表中查询出该用户所属的群组信息

2、再根据群组信息，查询属于该群组的所有用户的userid，并且和user表进行多表联合查询，查出用户的详细信息

![image-20210221113009787](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210221113009787.png)

![image-20210221113129540](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210221113129540.png)![image-20210221113437564](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210221113437564.png)

![image-20210221113241449](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210221113241449.png)

![image-20210221113742006](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210221113742006.png)

**mysql_insert_id**

mysql_insert_id()返回给定的 link_identifier中上一步 INSERT 查询中产生的 AUTO_INCREMENT 的 ID 号。如果没有指定 link_identifier，则使用上一个打开的连接。

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210221111058363.png" alt="image-20210221111058363" style="zoom:50%;" />





# 主要业务流程梳理

**注册业务流程**

客户端通过`telnet 127.0.0.1 6000`建立连接之后，向服务器发送字符串`{"msgid":2,"name":"wang","password":"111111"}`

服务器端正处于事件循环，监听到读写事件的发生，通过chatsever构造函数里的消息回调，调动onMessage函数。onMessage方法读取数据，反序列化成json对象，通过js["msgid"] 查询map表，获取业务处理器handler，注册msgid对应的处理器是reg方法，reg调动底层usermodel里的insert，将用户名密码插入数据库，将插入成功与否返回给reg，reg方法根据结果给客户端回馈response


**离线消息业务流程**

首先是有一张offlinemessage表，对应该表的在offlinemessagemodel文件里有存，删，查操作

一对一聊天业务上：用户发送消息时查到对方用户下线，将消息用insert插入offlinemessage表里。另一个用户登录成功以后，先去查询有没有离线消息，将离线消息推去给该用户后，清除离线消息表里的消息

**完全解耦网络模块的代码和业务模块的代码**

怎么把网络模块收到的消息派发到业务服务模块？让网络模块的代码和业务模块的代码完全解耦？

达到的目的：完全解耦网络模块的代码和业务模块的代码

例：通过js["msgid"] 获取=》业务处理器handler=》conn js time

答：创建一个ChatService模块，里面有一个map表，把msgid与对应的处理方法绑定之后存储起来，每次通过msgid去回调处理方法即可

# 远程登录

**VMware虚拟机ubuntu配置文件**

![image-20210216190359685](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210216190359685.png)

**腾讯轻量服务器配置文件**

![image-20210216203247529](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210216203247529.png)

**cmd登录**

ssh lighthouse@82.156.28.34

**远程发送文件**

直接打开cmd命令窗口就行

上传到服务器：

```shell
scp -r 本机文件路径 用户名@服务器ip:服务器要存储的路径

scp -r F://send//json.hpp lighthouse@82.156.28.34:/home/lighthouse/json
```

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210216205006551.png" alt="image-20210216205006551" style="zoom:50%;" />

![image-20210216204934815](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210216204934815.png)

从服务器下载到本地：（仍然在本地cmd窗口执行）

```shell
scp -r 用户名@服务器ip:服务器文件路径 本地存储路径
```

# 集群服务器之间的通信设计  

**如何解决跨服务器通信问题？**

当ChatServer集群部署多台服务器以后，当登录在不同服务器上的用户进行通信时，该怎么设计！如下
设计好吗？

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219113523003.png" alt="image-20210219113523003" style="zoom: 33%;" />

上面的设计，让各个ChatServer服务器互相之间直接建立TCP连接进行通信，相当于在服务器网络之间进行广播。这样的设计使得各个服务器之间耦合度太高，不利于系统扩展，并且会占用系统大量的socket资源，各服务器之间的带宽压力很大，不能够节省资源给更多的客户端提供服务，因此绝对不是一个好的设计。  

集群部署的服务器之间进行通信，最好的方式就是**引入中间件消息队列**，解耦各个服务器，使整个系统松耦合，提高服务器的响应能力，节省服务器的带宽资源，如下图所示：  

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219113613626.png" alt="image-20210219113613626" style="zoom:33%;" />

在集群分布式环境中，经常使用的中间件消息队列有ActiveMQ、RabbitMQ、Kafka等，都是应用场景广泛并且性能很好的消息队列，供集群服务器之间，分布式服务之间进行消息通信。限于我们的项目业务类型并不是非常复杂，对并发请求量也没有太高的要求（本项目并发量5~6万），因此我们的中间件消息队列选型的是-**基于发布-订阅模式的redis。**  

本项目用到通信的地方：一对一聊天，群聊

一对一聊天：client1与client2进行聊天，client1发现当前服务器_userConnMap表里没有client2的连接，那么就去数据库里看client2是否在线，若在线说明clinet2登录在其他服务器，若不在线则存为clinet2的离线消息

**基于发布-订阅模式的redis** 

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219114929283.png" alt="image-20210219114929283" style="zoom: 50%;" />

**发布: publish	订阅: subscribe	通知: notify**

每一台服务器都会与消息队列进行连接，当client1在ChatServer1登录成功，ChatServer1就会向消息队列订阅一下Client1，这样，当有人给client1发布消息，ChatServer1就能收到消息队列发布的消息。

现在比如：client1要与client2聊天，client2的连接没在ChatServer1上并且client2在线，说明clinet2在其他服务器上，client2所在的机器肯定**订阅**了client2的消息事件。client1发送消息后，ChatServer1向消息队列**publish**一个chat_json消息，是给client2的，消息队列就会**notify**订阅了client2的服务器。这样ChatServer2就知道了有人要与client2聊天，ChatServer2就在自己的_userConnMap中找到client2通信用的链接，转发出去

这样，消息队列开发好后，各个服务器要做的就是：向消息队列==订阅==自己的client，有客户要聊天，服务器就向消息队列==发布==消息去通过消息队列中间件找到对端客户的链接，消息队列去主动==通知==订阅了对端client的服务器

这就是一个典型的观察者模式应用场景：消息队列相当于观察者，观察到事件的发生，就把消息推给对该事件感兴趣的监听者（各个ChatServer）

# 负载均衡器

负载均衡：Load Balance，将负载（工作任务）进行平衡、分摊到多个操作单元上进行运行，负载均衡器就是完成负载均衡的机器

服务器集群：将很多服务器集中起来一起进行同一种服务，在客户端看来就像是只有一个服务器，一个服务器能处理2万左右的用户，3台集群就能处理5~6万

**集群服务器为什么要引入负载均衡器？**

答：单台服务器受限于硬件资源，其性能是有上限的，当单台服务器不能满足应用场景的并发需求量时，就
需要考虑部署多个服务器共同处理客户端的并发请求，但是客户端怎么知道去连接具体哪台服务器呢？
此时就需要一台负载均衡器，通过预设的负载算法，指导客户端连接服务器  

为了提高客户端并发聊天在线数量，去横向扩展ChatServer服务器的数量，服务器多了，就要引入负载均衡器

**负载均衡器需要解决的事情/负载均衡在本项目的意义（图中圆圈）**

- 高性能的网络设备、可配置的负载算法：要能承载客户端的请求以及连接服务器，服务器的响应再转发给客户端
- 能够和ChatServer保持心跳，检测ChatServer的故障
- 可以添加新的服务器设备，而不需要重启

![image-20210219110150387](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219110150387.png)

**ChatServer集群后怎么引入负载均衡器？**

# nginx

负载均衡配置

```shell
路径：/usr/local/nginx/conf/nginx.conf
```

<img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219134449593.png" alt="image-20210219134449593" style="zoom:50%;" />

 运行nginx-1.12.2在 这个目录下执行./nginx，nginx-1.12.2配置文件配置了负载均衡

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219132602611.png" alt="image-20210219132602611" style="zoom:50%;" />

直接执行nginx启动的是nginx-1.14.0

```shell
/usr/local/nginx/sbin/nginx   # 启动
/usr/local/nginx/sbin/nginx  -s reload  # 平滑重启
/usr/local/nginx/sbin/nginx  -s stop	# 停止nginx服务
```

![image-20210219133510657](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219133510657.png)

负载均衡功能验证

客户端的请求先发给nginx，nginx按照负载均衡的配置将请求转到相应的服务器上

启动两个服务器、两个客户端

![image-20210219140501817](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219140501817.png)

![image-20210219140521939](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219140521939.png)

# redis

redis首先是一个强大的缓存服务器，比memcache强大很多，不仅仅支持多种数据结构（不像memcache只能存储字符串）如字符串、list列表、set集合、map映射表等结构，还可以支持数据的持久化存储（memcache只支持内存存储），经常被应用到高并发的服务器环境设计之中  

**redis常用命令**

```shell
#启动redis客户端
redis-cli

#设置键值对  set key value
set "abc" "helloworld"

#获取键对应的值
get "abc"
```

**订阅**

```shell
#订阅1号通道的消息, 阻塞
subscribe 1
```

![image-20210219171723237](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219171723237.png)

**发布**

```shell
#向1号频道发送消息
publish 1 "hello"
```

![image-20210219171927839](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219171927839.png)

项目中：用户登录成功以后，服务器ChatServer向redis上根据用户的id去subcribe以用户id号为通道号的channel，当A用户向B用户发送消息时，发现B的链接不在本服务器且B在线，就向redis消息队列中publish消息，格式`publish  B_id  message` ，redis发现B_id通道有事件发生，因为B所在的服务器订阅了B_id对应的通道，所以redis通知notify用户B对应的服务器

**redis.hpp主要架构**

![image-20210219190736218](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210219190736218.png)

# 项目一键编译脚本与github管理

**自动编译脚本**

```shell
#!/bin/bash

set -x # 执行指令后，会先显示该指令及所下的参数。
 
rm -rf `pwd`/build/*
cd `pwd`/build && 
    cmake .. &&
    make
```

给autobuild.sh赋予可执行权限：`chmod  +x  autobuild.sh`

**推到github**

<img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210222154503466.png" alt="image-20210222154503466" style="zoom:50%;" />

![image-20210222154715793](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210222154715793.png)

![image-20210222160527414](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210222160527414.png)

![image-20210222160822742](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210222160822742.png)

注意：空文件夹是推不到git上去的

查看git状态

```shell
git status
```

1、添加到暂存区

```shell
git add .
```

2、提交到本地仓库

```shell
git commit -m "commit chatserver source code"
```

3、push到远程

````
git push
````

# 目录组织结构

![image-20210226222707039](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210226222707039.png)

# ------------------------------------

## 查看已启动的服务

切换root用户，再用`netstat -tanp`

```shell
su
netstat -tanp
```

## 查看g++版本号

```shell
g++ --version
```

## 安装软件

```shell
su
apt-get install nginx #或者apt install nginx
```

## 检测是否安装成功

```shell
which nginx
whereis nginx
nginx --version #或-V
```

## using

1、配合命名空间，对命名空间权限进行管理

```cpp
using namespace std;//释放整个命名空间到当前作用域
using std::cout;    //释放某个变量到当前作用域
```

2、类型重命名：作用等同typedef，但是逻辑上更直观

```cpp
typedef std::string TString;   //使用typedef的方式
using Ustring = std::string;   /使用 using typeName_self = stdtypename;

//更直观
typedef void (tFunc*)(void);
using uFunc = void(*)(void);
```

3、继承体系中，改变部分接口的继承权限。

```cpp
class Base
{
    public:
        void func() { cout << "Hello World" << endl;}
}

// private继承会导致 func的可见性为private
// 可使用using，改变访问权限
class Sub : private Base
{
   public：
       using Base::func;
}
```

## bind

bind函数看做一个通用的函数适配器，它接受一个可调用对象callable，生成一个新的可调用对象newCallable。

它可以把原可调用对象callable的某些参数预先绑定到给定的变量中（也叫参数绑定），然后产生一个新的可调用对象newCallable。

**用法一：绑定普通函数**

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210104159385.png" alt="image-20210210104159385" style="zoom:50%;" />

**用法二：普通函数与_1，\_2**

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210104444065.png" alt="image-20210210104444065" style="zoom:50%;" />

**用法三：成员函数**

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210104850965.png" alt="image-20210210104850965" style="zoom:50%;" />

网络编程中， 经常要使用到回调函数。 当底层的网络框架有数据过来时，往往通过回调函数来通知业务层。 这样可以使网络层只专注于 数据的收发， 而不必关心业务

在c语言中， 回调函数的实现往往通过函数指针来实现。 但是在c++中 ， 如果回调函数是一个类的成员函数。这时想把成员函数设置给一个回调函数指针往往是不行的。因为类的成员函数，多了一个隐含的参数this。 所以直接赋值给函数指针肯定会引起编译报错。就要用到bind来达到适配

## 回调

回调：我知道这个事件发生了怎么做，但我不知道具体什么时候发生，所以我先提前把该做的注册好, 等这个事件确实发生了，再来回调这个方法，或者说这个方法响应发生的事件 

简单来说，就是B在完成某件事的时候，自动执行A的相关操作，A、B可以是普通函数，也可以是类

**函数指针实现的回调：**

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210110211073.png" alt="image-20210210110211073" style="zoom:50%;" />

## telnet

常用的远程登录手段，启动telnet就相当于启动了一个客户端

```shell
telnet  IP地址      端口号
telnet  127.0.0.1  6000
```

![image-20210210122452230](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210122452230.png)

退出与服务器的连接

```shell
ctrl + ]
```

退出telnet

```shell
quit
```

![image-20210210123612525](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210123612525.png)

## whereis

Linux whereis命令用于查找文件。

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210210131147370.png" alt="image-20210210131147370" style="zoom: 67%;" />

## 开源项目文件组织形式

- bin：可执行文件
- lib：库文件
- include：头文件
- src：源文件
- build：编译过程中产生的中间文件
- test/example：测试文件
- thirdparty：第三方库文件
- CMakeLlists.txt：CMake的配置文件
- autobuild.sh：shell脚本，一键编译，执行的就是cmake
- README.md：介绍项目的说明性文件

## VSCode终端快捷键

ctrl + j

## 删除所有文件命令

```shell
rm -rf *
# -r: 递归删除
# -f: 即使原档案属性设为只读，亦直接删除，无需逐一确认
```

## ifndef

<img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210211104852690.png" alt="image-20210211104852690" style="zoom:67%;" />

## 编译器如何根据头文件来找到相应的cpp文件？

![image-20210211110138127](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210211110138127.png)

## stl下标引用元素的副作用

如果元素不存在,下标引用不会抛出异常,而是创建一个此下标的元素,整数默认值是0.

所以用下标返回时可以先find一下看有没有

![image-20210213110340860](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210213110340860.png)

## string中+的用法

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210213105737258.png" alt="image-20210213105737258" style="zoom:50%;" />

## ORM对象关系映射

ORM：Object Relational Mapping，对象关系映射

用于实现面向对象编程语言里不同类型系统的数据之间的转换。从效果上说，它其实是创建了一个==可在编程语言里使用的“虚拟对象数据库”==。如今已有很多免费和付费的ORM产品，而有些程序员更倾向于创建自己的ORM工具。

**简单说，ORM 就是通过实例对象的语法，完成关系型数据库的操作的技术，是"对象-关系映射"（Object/Relational Mapping） 的缩写。**

## iterator->second

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210214210600327.png" alt="image-20210214210600327" style="zoom:50%;" />

## map表

count：返回的是被查找元素的个数。如果有，返回1；否则，返回0。注意，map中不存在相同元素，所以返回值只能是1或0。

find()：返回的是被查找元素的位置，没有则返回map.end()。

## enum

**enum自增**

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210215155844312.png" alt="image-20210215155844312" style="zoom:50%;" />

**枚举类型**

在一些实际应用问题中，有些变量的取值被限定在一个有限的范围内。例如一周只有七天，一年只有12个月等，可以把此类变量定义为枚举类型。枚举类型的定义中列举出所有可能的取值，说明为该枚举类型的变量取值不能超过定义的范围。·

枚举类型(enumeration)是 C++ 中的一种派生数据类型，它是由用户定义的若**干枚举常量**的集合。

定义格式：枚举类型的定义格式为：

```cpp
enum <类型名> {<枚举常量表>};
enum color_set1 {RED, BLUE, WHITE, BLACK}; // 定义枚举类型color_set1
enum week {Sun, Mon, Tue, Wed, Thu, Fri, Sat}; // 定义枚举类型week

//枚举常量apple=0,orange=1, banana=1,peach=2,grape=3。
enum fruit_set {apple, orange, banana=1, peach, grape}

//枚举常量Sun,Mon,Tue,Wed,Thu,Fri,Sat的值分别为7、1、2、3、4、5、6。
enum week {Sun=7, Mon=1, Tue, Wed, Thu, Fri, Sat};
```

**枚举变量的使用**

```cpp
enum {Sun,Mon,Tue,Wed,Thu,Fri,Sat} weekday1, weekday2;//weekday1和weekday2就是枚举变量
weekday1 = Sun;//ok
```

枚举类型变量只能取一个值，所以其大小只是一个元素

## find与substr

- find：找指定字符在字符串中的下标
- substr：将字符串从指定位置开始向后跨越len长度生成子串

![image-20210216155248867](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210216155248867.png)

## 找不到mysql/mysql.h

ctrl + shif + p打开C/C++编辑配置，添加这一句

 <img src="img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210218102440931.png" alt="image-20210218102440931" style="zoom:50%;" />

## 不支持C++11

打开主工程目录下的CMakeLists.txt，添加这一句

 ```json
SET( CMAKE_CXX_FLAGS "-std=c++11")
 ```

## c_str()   atoi

- c_str()：string类型转成char*类型，为了让C++中的string能用在char\*的地方

- atoi：把字符串转换成整型数

## 变量名前加下划线

C++变量前面加下划线和不加下划线都不会影响对变量的定义，只是风格问题，更喜欢将==成员变量或者私有成员变量==的前面加上下划线。以表示该变量是某个类的属性。

```cpp
int size;
int getsize()
{
    return size;
}

//但是大多数人都不喜欢，觉得麻烦，就用了如下风格：
int _size;
int size()
{
    return _size;
}
```

## 如何保证代码段只被执行一次？

```cpp
//保证readTask只在客户端启动时候创建一次
//后续用户只要不关闭客户端，登出再登入，用的都是同一个readTask
static int readthreadnum = 0;
if(readthreadnum == 0)
{
    std::thread readTask(readTaskHandler, clientfd);
    readTask.detach(); //分离线程，由操作系统释放任何分配的资源。
    readthreadnum++;
}
```

## 命令行cmake

进入build目录下，执行：`cmake ..`，再执行`make`

```shell
cd /home/lighthouse/project/chatserver/build

cmake ..

make
```

## MVC、model

model：实体对应的类，即数据库表对应的类

![image-20210226212901028](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210226212901028.png)

![image-20210226212939735](img/%E9%A1%B9%E7%9B%AE%EF%BC%9AC++%E5%AE%9E%E7%8E%B0%E9%9B%86%E7%BE%A4%E8%81%8A%E5%A4%A9%E6%9C%8D%E5%8A%A1%E5%99%A8.img/image-20210226212939735.png)