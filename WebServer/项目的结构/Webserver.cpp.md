---
title: Webserver.cpp
updated: 2025-05-19T22:52:56
created: 2025-04-01T19:19:53
---

**WebServer构造函数**
<table>
<colgroup>
<col style="width: 6%" />
<col style="width: 93%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p>
<p>7</p>
<p>8</p>
<p>9</p>
<p>10</p>
<p>11</p>
<p>12</p>
<p>13</p>
<p>14</p>
<p>15</p>
<p>16</p>
<p>17</p>
<p>18</p>
<p>19</p>
<p>20</p>
<p>21</p>
<p>22</p>
<p>23</p>
<p>24</p>
<p>25</p>
<p>26</p>
<p>27</p>
<p>28</p>
<p>29</p>
<p>30</p>
<p>31</p>
<p>32</p></th>
<th><p>WebServer::WebServer(</p>
<p>int port, int trigMode, int timeoutMS, bool OptLinger, // http连接端口号、触发模式、超时时间、是否优雅关闭</p>
<p>int sqlPort, const char* sqlUser, const char* sqlPwd, // 数据库端口、数据库用户名、数据库密码</p>
<p>const char* dbName, int connPoolNum, int threadNum, // 数据库名称、数据库连接池数目、线程池数目</p>
<p>bool openLog, int logLevel, int logQueSize): // 是否打开日志、日志级别、日志队列大小</p>
<p>port_(port), openLinger_(OptLinger), timeoutMS_(timeoutMS), isClose_(false), // 参数赋给成员变量</p>
<p>timer_(new HeapTimer()), threadpool_(new ThreadPool(threadNum)), epoller_(new Epoller()) // 创建定时器、线程池、epoller实例</p>
<p>{</p>
<p>srcDir_ = getcwd(nullptr, 256);</p>
<p>strncat(srcDir_, "/resources", 16); // 设置http静态资源的目录</p>
<p>HttpConn::userCount = 0;</p>
<p>HttpConn::srcDir = srcDir_;</p>
<p>SqlConnPool::Instance()-&gt;Init("localhost", sqlPort, sqlUser, sqlPwd, dbName, connPoolNum); // 初始化单例模式的数据库连接池</p>
<p>InitEventMode_(trigMode); // 初始化连接和监听的事件模式(LT/ET)</p>
<p>if(!InitSocket_()) { isClose_ = true;} // 套接字初始化</p>
<p></p>
<p>if(openLog) {</p>
<p>// 初始化日志系统</p>
<p>Log::Instance()-&gt;init(logLevel, "./log", ".log", logQueSize);</p>
<p>if(isClose_) { LOG_ERROR("========== Server init error!=========="); }</p>
<p>else {</p>
<p>LOG_INFO("========== Server init ==========");</p>
<p>LOG_INFO("Port:%d, OpenLinger: %s", port_, OptLinger? "true":"false");</p>
<p>LOG_INFO("Listen Mode: %s, OpenConn Mode: %s",</p>
<p>(listenEvent_ &amp; EPOLLET ? "ET": "LT"),</p>
<p>(connEvent_ &amp; EPOLLET ? "ET": "LT"));</p>
<p>LOG_INFO("LogSys level: %d", logLevel);</p>
<p>LOG_INFO("srcDir: %s", HttpConn::srcDir);</p>
<p>LOG_INFO("SqlConnPool num: %d, ThreadPool num: %d", connPoolNum, threadNum);</p>
<p>}</p>
<p>}</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**初始化WebServer事件触发模式**
<table>
<colgroup>
<col style="width: 7%" />
<col style="width: 92%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p>
<p>7</p>
<p>8</p>
<p>9</p>
<p>10</p>
<p>11</p>
<p>12</p>
<p>13</p>
<p>14</p>
<p>15</p>
<p>16</p>
<p>17</p>
<p>18</p>
<p>19</p>
<p>20</p>
<p>21</p>
<p>22</p>
<p>23</p>
<p>24</p></th>
<th><p>void WebServer::InitEventMode_(int trigMode) {</p>
<p>listenEvent_ = EPOLLRDHUP; // 监听事件，默认设置，用于检测对端关闭连接</p>
<p>connEvent_ = EPOLLONESHOT | EPOLLRDHUP; // 连接事件：保证一个socket连接在任意时刻只被一个线程处理|检测对端关闭</p>
<p>switch (trigMode)</p>
<p>{</p>
<p>case 0: // 默认的LT模式</p>
<p>break;</p>
<p>case 1:</p>
<p>connEvent_ |= EPOLLET;</p>
<p>break;</p>
<p>case 2:</p>
<p>listenEvent_ |= EPOLLET;</p>
<p>break;</p>
<p>case 3:</p>
<p>listenEvent_ |= EPOLLET;</p>
<p>connEvent_ |= EPOLLET;</p>
<p>break;</p>
<p>default:</p>
<p>listenEvent_ |= EPOLLET;</p>
<p>connEvent_ |= EPOLLET;</p>
<p>break;</p>
<p>}</p>
<p>HttpConn::isET = (connEvent_ &amp; EPOLLET); // 根据连接事件是否有EPOLLET，设置http连接是否使用ET模式</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**典型的Reactor模式，单线程I/O事件循环和多线程业务处理的混合模式，也称半同步/半异步模式。单线程配合非阻塞**
**I/O可以更高效处理并发连接，将实际业务交给线程池，避免阻塞事件循环和资源隔离与可控。**
<table>
<colgroup>
<col style="width: 8%" />
<col style="width: 91%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p>
<p>7</p>
<p>8</p>
<p>9</p>
<p>10</p>
<p>11</p>
<p>12</p>
<p>13</p>
<p>14</p>
<p>15</p>
<p>16</p>
<p>17</p>
<p>18</p>
<p>19</p>
<p>20</p>
<p>21</p>
<p>22</p>
<p>23</p>
<p>24</p>
<p>25</p>
<p>26</p>
<p>27</p>
<p>28</p>
<p>29</p>
<p>30</p></th>
<th><p>void WebServer::Start() {</p>
<p>int timeMS = -1; /* epoll wait timeout == -1 无事件将阻塞 */</p>
<p>if(!isClose_) { LOG_INFO("========== Server start =========="); }</p>
<p>while(!isClose_) { // 典型的reactor模式，主循环直到isClose_为true结束</p>
<p>if(timeoutMS_ &gt; 0) {</p>
<p>timeMS = timer_-&gt;GetNextTick(); // 计算下一次定时任务时长</p>
<p>}</p>
<p>int eventCnt = epoller_-&gt;Wait(timeMS); // 等待I/O事件发生，返回事件数量，epoll_wait函数</p>
<p>for(int i = 0; i &lt; eventCnt; i++) {</p>
<p>/* 处理事件 */</p>
<p>int fd = epoller_-&gt;GetEventFd(i); // 获取事件描述符</p>
<p>uint32_t events = epoller_-&gt;GetEvents(i); // 获取事件</p>
<p>if(fd == listenFd_) {</p>
<p>DealListen_();</p>
<p>}</p>
<p>else if(events &amp; (EPOLLRDHUP | EPOLLHUP | EPOLLERR)) {</p>
<p>CloseConn_(&amp;users_[fd]);</p>
<p>}</p>
<p>else if(events &amp; EPOLLIN) {</p>
<p>DealRead_(&amp;users_[fd]); // 客户端发来请求，读取并解析</p>
<p>}</p>
<p>else if(events &amp; EPOLLOUT) {</p>
<p></p>
<p>DealWrite_(&amp;users_[fd]); // 服务端发送响应，</p>
<p>} else {</p>
<p>LOG_ERROR("Unexpected event");</p>
<p>}</p>
<p>}</p>
<p>}</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**连接保活机制**
<table>
<colgroup>
<col style="width: 9%" />
<col style="width: 90%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p></th>
<th><p>void WebServer::ExtentTime_(HttpConn* client) {</p>
<p>// 延长客户端连接超时的方法，客户端有活动时刷新超时计时器</p>
<p>if(timeoutMS_ &gt; 0) { timer_-&gt;adjust(client-&gt;GetFd(), timeoutMS_); }</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**处理读事件**
<table>
<colgroup>
<col style="width: 8%" />
<col style="width: 91%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p></th>
<th><p>void WebServer::DealRead_(HttpConn* client) {</p>
<p>ExtentTime_(client);</p>
<p>// 提交读任务到线程池，使用bind将Onread_方法和当前对象和客户端连接绑定</p>
<p>// 交给线程池异步处理，避免主线程阻塞</p>
<p>threadpool_-&gt;AddTask(std::bind(&amp;WebServer::OnRead_, this, client));</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**处理写事件**
<table>
<colgroup>
<col style="width: 7%" />
<col style="width: 92%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p></th>
<th><p>void WebServer::DealWrite_(HttpConn* client) {</p>
<p>ExtentTime_(client);</p>
<p>// 提交写任务到线程池，使用bind将Onread_方法和当前对象和客户端连接绑定</p>
<p>// 交给线程池异步处理，避免主线程阻塞</p>
<p>threadpool_-&gt;AddTask(std::bind(&amp;WebServer::OnWrite_, this, client));</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**初始化服务器监听套接字，负责创建、配置和启动服务器的监听**
<table>
<colgroup>
<col style="width: 8%" />
<col style="width: 91%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p>
<p>7</p>
<p>8</p>
<p>9</p>
<p>10</p>
<p>11</p>
<p>12</p>
<p>13</p>
<p>14</p>
<p>15</p>
<p>16</p>
<p>17</p>
<p>18</p>
<p>19</p>
<p>20</p>
<p>21</p>
<p>22</p>
<p>23</p>
<p>24</p>
<p>25</p>
<p>26</p>
<p>27</p>
<p>28</p>
<p>29</p>
<p>30</p>
<p>31</p>
<p>32</p>
<p>33</p>
<p>34</p>
<p>35</p>
<p>36</p>
<p>37</p>
<p>38</p>
<p>39</p>
<p>40</p>
<p>41</p>
<p>42</p>
<p>43</p>
<p>44</p>
<p>45</p>
<p>46</p>
<p>47</p>
<p>48</p>
<p>49</p>
<p>50</p>
<p>51</p>
<p>52</p>
<p>53</p>
<p>54</p>
<p>55</p>
<p>56</p>
<p>57</p>
<p>58</p>
<p>59</p>
<p>60</p>
<p>61</p>
<p>62</p>
<p>63</p>
<p>64</p>
<p>65</p>
<p>66</p>
<p>67</p>
<p>68</p>
<p>69</p>
<p>70</p></th>
<th><p>bool WebServer::InitSocket_() {</p>
<p>int ret;</p>
<p>struct sockaddr_in addr;</p>
<p>// 端口号有效验证</p>
<p>if(port_ &gt; 65535 || port_ &lt; 1024) {</p>
<p>LOG_ERROR("Port:%d error!", port_);</p>
<p>return false;</p>
<p>}</p>
<p>// 地址结构初始化</p>
<p>addr.sin_family = AF_INET; // IPv4协议</p>
<p>addr.sin_addr.s_addr = htonl(INADDR_ANY);</p>
<p>addr.sin_port = htons(port_); // 设置监听端口</p>
<p>struct linger optLinger = { 0 };</p>
<p>if(openLinger_) {</p>
<p>/* 优雅关闭: 直到所剩数据发送完毕或超时 */</p>
<p>optLinger.l_onoff = 1;</p>
<p>optLinger.l_linger = 1;</p>
<p>}</p>
<p></p>
<p>listenFd_ = socket(AF_INET, SOCK_STREAM, 0); // 创建IPv4套接字</p>
<p>if(listenFd_ &lt; 0) {</p>
<p>LOG_ERROR("Create socket error!", port_);</p>
<p>return false;</p>
<p>}</p>
<p></p>
<p>// 设置套接字选项</p>
<p>ret = setsockopt(listenFd_, SOL_SOCKET, SO_LINGER, &amp;optLinger, sizeof(optLinger));</p>
<p>if(ret &lt; 0) {</p>
<p>close(listenFd_);</p>
<p>LOG_ERROR("Init linger error!", port_);</p>
<p>return false;</p>
<p>}</p>
<p></p>
<p>int optval = 1;</p>
<p>/* 端口复用 */</p>
<p>/* 只有最后一个套接字会正常接收数据。 */</p>
<p>ret = setsockopt(listenFd_, SOL_SOCKET, SO_REUSEADDR, (const void*)&amp;optval, sizeof(int));</p>
<p>if(ret == -1) {</p>
<p>LOG_ERROR("set socket setsockopt error !");</p>
<p>close(listenFd_);</p>
<p>return false;</p>
<p>}</p>
<p></p>
<p>// 绑定</p>
<p>ret = bind(listenFd_, (struct sockaddr *)&amp;addr, sizeof(addr));</p>
<p>if(ret &lt; 0) {</p>
<p>LOG_ERROR("Bind Port:%d error!", port_);</p>
<p>close(listenFd_);</p>
<p>return false;</p>
<p>}</p>
<p></p>
<p>// 监听</p>
<p>ret = listen(listenFd_, 6);</p>
<p>if(ret &lt; 0) {</p>
<p>LOG_ERROR("Listen port:%d error!", port_);</p>
<p>close(listenFd_);</p>
<p>return false;</p>
<p>}</p>
<p>// 将套接字注册到epoll实例中，</p>
<p>ret = epoller_-&gt;AddFd(listenFd_, listenEvent_ | EPOLLIN);</p>
<p>if(ret == 0) {</p>
<p>LOG_ERROR("Add listen error!");</p>
<p>close(listenFd_);</p>
<p>return false;</p>
<p>}</p>
<p>// 设置非阻塞模式</p>
<p>SetFdNonblock(listenFd_);</p>
<p>LOG_INFO("Server port:%d", port_);</p>
<p>return true;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

实现的是一个简单的单线程reactor，所有事件处理都在同一个线程中完成，逻辑代码如下所示：
<table>
<colgroup>
<col style="width: 13%" />
<col style="width: 86%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p>
<p>7</p>
<p>8</p></th>
<th><p>while (!isClose_) {</p>
<p>epoll_wait(...); // 阻塞等待事件</p>
<p>for (每个事件) {</p>
<p>if (是监听socket) accept();</p>
<p>else if (可读) read(); // 直接在当前线程处理</p>
<p>else if (可写) write(); // 直接在当前线程处理</p>
<p>}</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
1、单线程事件循环while(!isClose)
2、直接处理I/O（没有线程池派发）
3、无锁（没有mutex或atomic）
4、串行执行（如果某一行卡住，整个服务器会卡住）
