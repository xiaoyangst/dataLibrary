- *boost::asio*：这是核心类和函数所在的地方。重要的类有 *io_service* 和 *streambuf*。类似*read, read_at, read_until*方法，它们的异步方法，它们的写方法和异步写方法等自由函数也在这里
- *boost::asio::ip*：这是网络通信部分所在的地方。重要的类有 *address, endpoint, tcp, udp和 icmp*，重要的自由函数有 *connect* 和 *async_connect*。要注意的是在 *boost::asio::ip::tcp::socket*中间，*socket*只是 *boost::asio::ip::tcp* 类中间的一个*typedef*关键字
- *boost::asio::error*：这个命名空间包含了调用 I/O 例程时返回的错误码
- *boost::asio::ssl*：包含了 SSL 处理类的命名空间
- *boost::asio::local*：这个命名空间包含了 POSIX 特性的类
- *boost::asio::windows*：这个命名空间包含了 Windows 特性的类

## IP地址的处理

由于下面介绍终端端点需要用到关于 IP 地址作为参数传递，先提前讲，因为不是单纯的传递 string 类型的 IP 参数就可以的。

- ip::address(v4_or_v6_address)：这个函数把一个v4或者v6的地址转换成 ip::address
- ip::address:from_string(str)：这个函数根据一个IPv4地址或者一个IPv6地址创建一个地址
- ip::address::to_string() ：这个函数返回这个地址的字符串
- ip::address_v4::broadcast([addr, mask])：这个函数创建了一个广播地址 
- ip::address_v4::any()：这个函数返回一个能表示任意地址的地址
- ip::address_v4::loopback(), ip_address_v6::loopback()：这个函数返回环路地址（为v4/v6协议）
- ip::host_name()：这个函数用 string 数据类型返回当前的主机名

注：不支持传递域名

## 终端端点

所谓终端节点就是**用来通信的端对端的节点**，可以通过ip地址和端口构造，其它的节点可以连接这个终端节点进行通信。不同类型的 socket 有它自己的 endpoint 类，比如 ip::tcp::endpoint、 ip::udp::endpoint 和 ip::icmp::endpoint。

- endpoint()：这是默认构造函数，某些时候可以用来创建UDP/ICMP socket
- endpoint(protocol, port)：这个方法通常用来创建可以接受新连接的服务器端socket
- endpoint(addr, port)：这个方法创建了一个连接到某个地址和端口的端点

```c++
using namespace std;
using namespace boost::asio::ip;

// 客户端可以通过对端地址和端口构造一个endpoint，用这个endpoint和其通信
    string ip = "192.168.1.12";	
    int port = 8080;	
    tcp::endpoint c_ep(address::from_string(ip),port);  

// 服务端只需根据本地地址绑定就可以生成endpoint
    int server_port = 8080;	
    tcp::endpoint s_ep(address_v4::any(), server_port);
```

给定一个端点，可以获得他的地址，端口和IP协议（v4或者v6）：

```c++
cout << c_ep.address().to_string() << ":" << c_ep.port() << "/" << c_ep.protocol() << endl;
```

注：输出协议部分报错，因为 `c_ep.protocol()` 返回的 `tcp::protocol` 类型没有与 `std::cout` 流兼容的 `<<` 运算符重载，所以实际上你只能打印出地址和端口信息

## 创建socket

asio 有三种类型的套接字类：ip::tcp, ip::udp 和 ip::icmp。当然它也是可扩展的，你可以创建自己的socket类，尽管这相当复杂。我们知道在网络通信中有两种 socket 类型，即监听客户端连接的socket，以及用于后续和客户端通信的socket。

下面介绍 socket 相关的函数，这些函数用来连接或绑定socket、断开socket字连接以及查询连接是活动还是非活动的：

- assign(protocol,socket)：这个函数分配了一个原生的 socket 给这个 socket 实例。当处理老（旧）程序时会使用它（也就是说，原生socket已经被建立了）
- open(protocol)：这个函数用给定的 IP 协议（v4 或者 v6）打开一个 socket。你主要在 UDP/ICMP socket，或者服务端 socket 上使用
- bind(endpoint)：这个函数绑定到一个地址
- connect(endpoint)：这个函数用同步的方式连接到一个地址
- async_connect(endpoint)：这个函数用异步的方式连接到一个地址
- is_open()：如果套接字已经打开，这个函数返回 true
- close()：这个函数用来关闭套接字。调用时这个套接字上任何的异步操作都会被立即关闭，同时返回 error::operation_aborted 错误码
- shutdown(type_of_shutdown)：这个函数立即使 send 或者 receive 操作失效，或者两者都失效
- cancel()：这个函数取消套接字上所有的异步操作。这个套接字上任何的异步操作都会立即结束，然后返回 error::operation_aborted 错误码

创建用于接收客户端连接的 socket

```c++
io_context ic;
tcp protocol = tcp::v4();
tcp::acceptor acceptor(ic);
acceptor.open(protocol);
```

创建用于通信的 socket

```c++
io_context ic;
tcp protocol = tcp::v4();
tcp::socket sock(ic);
sock.open(protocol);
```

## 绑定acceptor

```c++
// 服务端需要创建的端点
    int server_port = 8080;
    tcp::endpoint s_ep(address_v4::any(), server_port);

// 创建用于接收客户端连接的 socket 类型，即 acceptor
    io_context ic;
    tcp protocol = tcp::v4();
    tcp::acceptor acceptor(ic);
    acceptor.open(protocol);

// 绑定 端点和acceptor
    acceptor.bind(s_ep);
```

将 acceptor 绑定到 端点，意味着在这个端点上进行客户端连接的接收，即所有连接这个端点的连接都可以被接收到。

## 连接指定的端点

```c++
//  创建端点，记录服务器的信息   
    string ip = "192.168.1.12";
    int port = 8080;
    tcp::endpoint ep(address::from_string(ip), port);

//  连接到服务器
    io_context ic;
    tcp::socket sock(ic, ep.protocol());
    sock.connect(ep);

// 后续就可以通信了
```

作为客户端可以连接服务器指定的端点进行连接，用于后续通信。

## 服务器接收连接

```c++
    const int BACKLOG_SIZE = 30;
    int server_port = 8080;
    tcp::endpoint s_ep(address_v4::any(), server_port);

// 创建用于接收客户端连接的 socket 类型，即 acceptor
    io_context ic;
    tcp protocol = tcp::v4();
    tcp::acceptor acceptor(ic);
    acceptor.open(protocol);

// 绑定、监听、接收客户端连接
    acceptor.bind(s_ep);
    acceptor.listen(BACKLOG_SIZE);

    tcp::socket c_socket(ic);
    acceptor.accept(c_socket);

// 后续就可以和客户端通信了
```

当有客户端连接时，服务器需要接收连接，如何就可以完成这个工作。

---

⭐️内容取自 B 站 UP 恋恋风辰和 mmoaay 的《Boost.Asio C++ 网络编程》，仅从中取出个人以为需要纪录的内容。不追求内容的完整性，却也不会丢失所记内容的逻辑性。如果需要了解细致，建议看原视频或者读原书。
