---
title: TCP套接字编程
updated: 2025-05-19T23:05:14
created: 2025-03-09T14:47:55
---

### TCP 套接字编程的流程

建立连接（三次握手：发送/接收网络数据）--\>关闭连接（四次挥手）

**为什么需要三次握手机制：**

通信双方能够成功通信的前提是双方建立连接，双方都能收和发。三次握手实际上就是一个`测试能不能成功建立连接的过程`。

第一次握手测试了客户端能否发，第二次测试服务端能否收和发，第三次测试客户端能否收。

**四次挥手使计算机释放不再使用的资源**

### 对于 TCP server：

> socket(); 创建一个套接字

> bind(); 把一个套接字与网络地址相绑定，如果不绑定，内核会自动指定一个地址

> listen(); “监听模式”

> accept(); 接收客户端连接请求，多次调用 accept 可以与不同的客户端建立连接

read/recv/recvform or write/send/sendto

close/shutdown

### TCP client:

socket();

bind();

connect(); 发起连接请求

read/recv/recvform or write/send/sendto

close/shutdown

### socket 具体的 API 函数解析：

#### （1）socket：

**socket：创建一个套接字**

```c++
# include<sys/types.h>
# include<sys/sockets.h>

int socket(int domain, int type, int protocol);
/*

@domain: 指定域或者协议族

socket接口不仅仅局限于TCP/IP，还包括Bluetooth，本地通信，....

每一种通信下面都有许多的协议：

AF_INET，IPv4协议族

AF_INET6，IPv6协议族

@type: 指定套接字的类型：

SOCK_STREAM：流式套接字 --\>TCP

SOCK_DGRAM：数据报套接字--\>UDP

SOCK_RAM：原始套接字

@protocol: 指定应用层协议，可以指定为0（不知名的私有应用协议）

@return：成功，返回一个套接字文件描述符(\>2)

失败，-1，同时设置errno
*/
```

#### （2）网络地址结构体

socket 接口不仅仅用于 IPv4，也可以用于 IPv6.....，不同的协议的地址是不一样的。

通用网络地址结构体：

```c++
struct sockaddr

{

sa_family_t sa_family; //指定协议族

char sa_data[14]; //包含套接字中的目的地址和端口信息

}

// 缺陷是把IP地址和端口地址混在一起保存在sa_data中

IPv4专用地址结构体：

struct sockaddr_in

{

sa_family_t sin_family; // 指定协议族AF_INET

in_port sin_port; // 指定端口号

struct in_addr sin_addr; // 指定IP地址

unsigned char sin_zero\[8\]; // 填充8个字节，无实际意义

};

struct in_addr

{

in_addrt_t s_addr; //unsigned int

}

// 这个结构体把IP地址和端口号分开保存，

// sockaddr常用于bind,connect等函数的参数
```

#### （3）IP 地址转换函数

IP 地址是以**点分十进制形式存在的，实际上转换成一个无符号的 32bit 的数据**

```c++
# include<sys/socket.h>
# include<netinet.h>
# include<arpa/inet.h>

// 将点十进制的字符串转为IP

int inet_aton(const char*cp, struct in_addr *inp);

/*

@cp: 指向要转换的点分十进制的字符串 -\>"192.168.31.8"

@inp: 指向一个IP结构体，用来保存转换后的IP的地址

@return：成功返回0，失败-1
*/
```

#### （4） 整数在主机字节序与网络字节序之间的转换函数

htonl，htons，nthol，ntohs--> convert values between host and network byte order

```c++
# include<arpa/inet.h>

uint32_t htonl(uint32_t hostlong);

uint16_t htons(uint16_t hostshort);

uint32_t ntohl(uint32_t netlong);

uint16_t ntohs(uint16_t netshort);
```

#### （5）bind

**将一个网络地址绑定到一个 socket 上**

```c++
# include<sys/types.h>
# include<sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

/*

@sockfd: 要绑定地址的套接字描述符

@addr: 通用的网络地址结构体指针

@addrlen: 指向地址结构体长度

@return: 成功0，失败-1
*/
```

#### （6）listen：让套接字进入“监听模式”

```c++
# include<sys/types.h>;
# include<sys/socket.h>

int listen(int sockfd, int backlog);
/*
@sockfd: 进入监听模式的套接字
@backlog: 同时能够处理连接请求的数目
@return: 成功0，失败-1
*/
```

#### （7）accept：用于 TCP server 接收来自客户端的连接请求

```c++
# include<sys/types.h>;
# include<sys/socket.h>;

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
/*
@sockfd: 套接字
@addr: 网络地址结构体指针，用来保存客户端的地址信息
@addrlen: 保存客户端地址结构体的长度
@return: 成功：与该客户端的连接套接字描述符，后续与该客户端通信都是这个套接字描述符
失败：-1
*/
```

#### （8）connect：客户端主动向服务器发起连接请求

```c++
# include<sys/types.h>
# include<sys/socket.h>

int connect(int sockfd, const struct sockaddr*addr, socklen_t addrlen);
/*
@sockfd: 套接字
@addr: 服务器的地址
@addrlen: 地址结构体的长度
@return: 成功0，失败-1
*/
```

#### （9）往套接字上面发送数据

write/send/sendto 三个函数，TCP 都可以用，UDP 只能 sendto

```c++
# include<sys/types.h>
# include<sys/socket.h>

ssize_t send(int sockfd, const void *buf, size_t len, int flags);
/*
@sockfd: 套接字
@buf: 即将要发送的数据
@len: 发送数据的长度
@flags: 一般为0
@return: 成功：返回实际发送的字节数
失败：-1
*/

ssize_t sendto(int sockfd, const void *buf, size_t len, int flag,
const struct sockaddr *dest_addr, socklen_t addrlen);
/*
@dest_addr: 指定接收方的地址
*/
```

#### （10） 从套接字上接收数据

read/recv/recvfrom

```c++
# include<sys/types.h>
# include<sys/socket.h>

ssize_t recv(int sockfd, void* buf, size_t len, int flags);
/*
@sockfd: 套接字
@buf: 接收的数据
@len: 接收数据的长度
@flags: 一般为0
@return: 成功：接收数据的长度
失败： -1
*/
```

#### （11）关闭套接字

close/shutdown

```c++
# include<sys/socket.h>

int shutdown(int sockfd, int how);
/*
@sockfd: 套接字
@how:
SHUT_RD: 关闭读
SHUT_WR: 关闭写
SHUT_RSWR: 关闭读写 ;
*/

int close(sockfd);
@return: 成功：0
失败：-1
```
