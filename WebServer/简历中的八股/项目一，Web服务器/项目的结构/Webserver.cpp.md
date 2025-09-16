---
title: Webserver.cpp
updated: 2025-05-19T22:52:56
created: 2025-04-01T19:19:53
---

## WebServer构造函数
```c++
WebServer::WebServer(

    int port, int trigMode, int timeoutMS, bool OptLinger, // http连接端口号、触发模式、超时时间、是否优雅关闭

    int sqlPort, const char* sqlUser, const char* sqlPwd, // 数据库端口、数据库用户名、数据库密码

    const char* dbName, int connPoolNum, int threadNum, // 数据库名称、数据库连接池数目、线程池数目

    bool openLog, int logLevel, int logQueSize): // 是否打开日志、日志级别、日志队列大小

    port_(port), openLinger_(OptLinger), timeoutMS_(timeoutMS), isClose_(false), // 参数赋给成员变量

    timer_(new HeapTimer()), threadpool_(new ThreadPool(threadNum)), epoller_(new Epoller()) // 创建定时器、线程池、epoller实例

{

    srcDir_ = getcwd(nullptr, 256);

    strncat(srcDir_, "/resources", 16); // 设置http静态资源的目录

    HttpConn::userCount = 0;

    HttpConn::srcDir = srcDir_;

    SqlConnPool::Instance()->Init("localhost", sqlPort, sqlUser, sqlPwd, dbName, connPoolNum); // 初始化单例模式的数据库连接池

    InitEventMode_(trigMode); // 初始化连接和监听的事件模式(LT/ET)

    if(!InitSocket_()) { isClose_ = true;} // 套接字初始化

    if(openLog) {

        // 初始化日志系统

        Log::Instance()->init(logLevel, "./log", ".log", logQueSize);

        if(isClose_) { 
            LOG_ERROR("========== Server init error!=========="); 
        }

        else {

            LOG_INFO("========== Server init ==========");

            LOG_INFO("Port:%d, OpenLinger: %s", port_, OptLinger? "true":"false");

            LOG_INFO("Listen Mode: %s, OpenConn Mode: %s",

            (listenEvent_ & EPOLLET ? "ET": "LT"),

            (connEvent_ & EPOLLET ? "ET": "LT"));

            LOG_INFO("LogSys level: %d", logLevel);

            LOG_INFO("srcDir: %s", HttpConn::srcDir);

            LOG_INFO("SqlConnPool num: %d, ThreadPool num: %d", connPoolNum, threadNum);

        }

    }

}
```

## 初始化WebServer事件触发模式
```c++
void WebServer::InitEventMode_(int trigMode) {

    listenEvent_ = EPOLLRDHUP; // 监听事件，默认设置，用于检测对端关闭连接

    connEvent_ = EPOLLONESHOT | EPOLLRDHUP; // 连接事件：保证一个socket连接在任意时刻只被一个线程处理|检测对端关闭

    switch (trigMode)

    {

    case 0: // 默认的LT模式

        break;

    case 1:

        connEvent_ |= EPOLLET;

    break;

    case 2:

        listenEvent_ |= EPOLLET;

    break;

    case 3:

        listenEvent_ |= EPOLLET;

        connEvent_ |= EPOLLET;

    break;

    default:

        listenEvent_ |= EPOLLET;

        connEvent_ |= EPOLLET;

    break;

    }

    HttpConn::isET = (connEvent_ & EPOLLET); // 根据连接事件是否有EPOLLET，设置http连接是否使用ET模式

}
```

## 典型的Reactor模式，单线程I/O事件循环和多线程业务处理的混合模式，也称半同步/半异步模式。单线程配合非阻塞**
**I/O可以更高效处理并发连接，将实际业务交给线程池，避免阻塞事件循环和资源隔离与可控。**
```c++
void WebServer::Start() {

    int timeMS = -1; /* epoll wait timeout == -1 无事件将阻塞 */

    if(!isClose_) { 
        LOG_INFO("========== Server start =========="); 
    }

    while(!isClose_) { // 典型的reactor模式，主循环直到isClose_为true结束

        if(timeoutMS_ > 0) {

            timeMS = timer_->GetNextTick(); // 计算下一次定时任务时长

    }

    int eventCnt = epoller_->Wait(timeMS); // 等待I/O事件发生，返回事件数量，epoll_wait函数

    for(int i = 0; i < eventCnt; i++) {

        /* 处理事件 */

        int fd = epoller_->GetEventFd(i); // 获取事件描述符

        uint32_t events = epoller_->GetEvents(i); // 获取事件

        if(fd == listenFd_) {

            DealListen_();

        }

        else if(events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR)) {

        CloseConn_(&users_[fd]);

        }

        else if(events & EPOLLIN) {

            DealRead_(&users_[fd]); // 客户端发来请求，读取并解析

        }

        else if(events & EPOLLOUT) {

            DealWrite_(&users_[fd]); // 服务端发送响应，

        } else {

            LOG_ERROR("Unexpected event");

        }

    }

}

}
```

**连接保活机制**
```c++
void WebServer::ExtentTime_(HttpConn* client) {

    // 延长客户端连接超时的方法，客户端有活动时刷新超时计时器

    if(timeoutMS_ > 0) { 
        timer_->adjust(client->GetFd(), timeoutMS_); 
    }

}
```

## 处理读事件
```c++
void WebServer::DealRead_(HttpConn* client) {

    ExtentTime_(client);

    // 提交读任务到线程池，使用bind将Onread_方法和当前对象和客户端连接绑定

    // 交给线程池异步处理，避免主线程阻塞

    threadpool_->AddTask(std::bind(&WebServer::OnRead_, this, client));

}
```

## 处理写事件
```c++
void WebServer::DealWrite_(HttpConn* client) {

    ExtentTime_(client);

    // 提交写任务到线程池，使用bind将Onread_方法和当前对象和客户端连接绑定

    // 交给线程池异步处理，避免主线程阻塞

    threadpool_->AddTask(std::bind(&WebServer::OnWrite_, this, client));

}
```

## 初始化服务器监听套接字，负责创建、配置和启动服务器的监听
```c++
bool WebServer::InitSocket_() {

    int ret;

    struct sockaddr_in addr;

    // 端口号有效验证

    if(port_ > 65535 || port_ < 1024) {

        LOG_ERROR("Port:%d error!", port_);

        return false;

    }

    // 地址结构初始化

    addr.sin_family = AF_INET; // IPv4协议

    addr.sin_addr.s_addr = htonl(INADDR_ANY);

    addr.sin_port = htons(port_); // 设置监听端口

    struct linger optLinger = { 0 };

    if(openLinger_) {

        /* 优雅关闭: 直到所剩数据发送完毕或超时 */

        optLinger.l_onoff = 1;

        optLinger.l_linger = 1;

    }

    listenFd_ = socket(AF_INET, SOCK_STREAM, 0); // 创建IPv4套接字

    if(listenFd_ < 0) {

        LOG_ERROR("Create socket error!", port_);

        return false;

    }

// 设置套接字选项

    ret = setsockopt(listenFd_, SOL_SOCKET, SO_LINGER, &optLinger, sizeof(optLinger));

    if(ret < 0) {

        close(listenFd_);

        LOG_ERROR("Init linger error!", port_);

        return false;

    }

    int optval = 1;

    /* 端口复用 */

    /* 只有最后一个套接字会正常接收数据。 */

    ret = setsockopt(listenFd_, SOL_SOCKET, SO_REUSEADDR, (const void*)&optval, sizeof(int));

    if(ret == -1) {

        LOG_ERROR("set socket setsockopt error !");

        close(listenFd_);

        return false;

    }

    // 绑定

    ret = bind(listenFd_, (struct sockaddr *)&addr, sizeof(addr));

    if(ret < 0) {

        LOG_ERROR("Bind Port:%d error!", port_);

        close(listenFd_);

        return false;

    }

    // 监听

    ret = listen(listenFd_, 6);

        if(ret < 0) {

        LOG_ERROR("Listen port:%d error!", port_);

        close(listenFd_);

        return false;

    }

    // 将套接字注册到epoll实例中，

    ret = epoller_->AddFd(listenFd_, listenEvent_ | EPOLLIN);

    if(ret == 0) {

        LOG_ERROR("Add listen error!");

        close(listenFd_);

        return false;

    }

    // 设置非阻塞模式

    SetFdNonblock(listenFd_);

    LOG_INFO("Server port:%d", port_);

    return true;

}
```

实现的是一个简单的单线程reactor，所有事件处理都在同一个线程中完成，逻辑代码如下所示：
```c++
while (!isClose_) {

epoll_wait(...); // 阻塞等待事件

for (每个事件) {

if (是监听socket) accept();

else if (可读) read(); // 直接在当前线程处理

else if (可写) write(); // 直接在当前线程处理

}

}
```
1、单线程事件循环while(!isClose)
2、直接处理I/O（没有线程池派发）
3、无锁（没有mutex或atomic）
4、串行执行（如果某一行卡住，整个服务器会卡住）
