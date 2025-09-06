---
title: Http连接
updated: 2025-06-14T14:48:49
created: 2025-04-07T18:51:32
---

Http连接的**构造函数和析构函数**
<table>
<colgroup>
<col style="width: 24%" />
<col style="width: 75%" />
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
<p>11</p></th>
<th><p>// 构造函数，初始化成员变量</p>
<p>HttpConn::HttpConn() {</p>
<p>fd_ = -1;</p>
<p>addr_ = { 0 };</p>
<p>isClose_ = true;</p>
<p>};</p>
<p></p>
<p>// 析构函数，关闭连接</p>
<p>HttpConn::~HttpConn() {</p>
<p>Close();</p>
<p>};</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**初始化函数，初始化一个HTTP连接对象**
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
<p>9</p></th>
<th><p>void HttpConn::init(int fd, const sockaddr_in&amp; addr) {</p>
<p>userCount++; // 增加当前用户连接数</p>
<p>addr_ = addr; // 传入的客户端地址保存在成员变量</p>
<p>fd_ = fd; // 存储传入的文件描述符（socket）</p>
<p>writeBuff_.RetrieveAll();</p>
<p>readBuff_.RetrieveAll(); // 清空读写缓冲区</p>
<p>isClose_ = false; // 设置关闭连接标志</p>
<p>LOG_INFO("Client[%d](%s:%d) in, userCount:%d", fd_, GetIP(), GetPort(), (int)userCount);</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**关闭连接函数**
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
<p>10</p></th>
<th><p>void HttpConn::Close() {</p>
<p>// 关闭连接</p>
<p>response_.UnmapFile(); // 停止映射</p>
<p>if(isClose_ == false){</p>
<p>isClose_ = true; // 设置关闭标志</p>
<p>userCount--; // 用户连接数-1</p>
<p>close(fd_); // 关闭socket</p>
<p>LOG_INFO("Client[%d](%s:%d) quit, UserCount:%d", fd_, GetIP(), GetPort(), (int)userCount);</p>
<p>}</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**从客户端socket读取数据到读缓冲区，并处理阻塞与非阻塞模式下的读取逻辑**
<table>
<colgroup>
<col style="width: 10%" />
<col style="width: 89%" />
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
<p>12</p></th>
<th><p>ssize_t HttpConn::read(int* saveErrno) {</p>
<p>ssize_t len = -1;</p>
<p>// ET模式下循环读取，确保一次性读完可读数据</p>
<p>// LT模式下，只读取一次</p>
<p>do {</p>
<p>len = readBuff_.ReadFd(fd_, saveErrno); // 从socket读取数据到缓冲区</p>
<p>if (len &lt;= 0) {</p>
<p>break;</p>
<p>}</p>
<p>} while (isET); // ET模式，循环读取，直到没有数据可读</p>
<p>return len;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**向客户端socket写数据，并处理ET/LT模式**
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
<th><p>ssize_t HttpConn::write(int* saveErrno) {</p>
<p>ssize_t len = -1;</p>
<p>// do while</p>
<p>// ET模式，循环写入，直到数据发送完毕或错误</p>
<p>// LT模式，一次写入</p>
<p>do {</p>
<p>len = writev(fd_, iov_, iovCnt_); // 一次性写入多个缓冲区,iov_[0]存储HTTP头部,iov_[1]存储文件数据</p>
<p>if(len &lt;= 0) {</p>
<p>*saveErrno = errno;</p>
<p>break;</p>
<p>} //</p>
<p>if(iov_[0].iov_len + iov_[1].iov_len == 0) { break; } /* 传输结束 */</p>
<p>else if(static_cast&lt;size_t&gt;(len) &gt; iov_[0].iov_len) {</p>
<p>// 如果写入的字节数&gt;iov_[0]，调整iov_[1]的指针和长度，</p>
<p>iov_[1].iov_base = (uint8_t*) iov_[1].iov_base + (len - iov_[0].iov_len);</p>
<p>iov_[1].iov_len -= (len - iov_[0].iov_len);</p>
<p>if(iov_[0].iov_len) {</p>
<p>writeBuff_.RetrieveAll();</p>
<p>iov_[0].iov_len = 0;</p>
<p>} // 全部发送，清空写缓冲区</p>
<p>}</p>
<p>else {</p>
<p>// 只调整iov_[0]的指针和剩余长度</p>
<p>iov_[0].iov_base = (uint8_t*)iov_[0].iov_base + len;</p>
<p>iov_[0].iov_len -= len;</p>
<p>writeBuff_.Retrieve(len); // 从写缓冲区移除已发送的数据</p>
<p>}</p>
<p>} while(isET || ToWriteBytes() &gt; 10240);</p>
<p>return len;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**HTTP连接处理的核心逻辑，负责解析请求、生成响应、准备发送数据**

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th><p>HttpConn::PROCESS_STATE HttpConn::process() {</p>
<p>int fd = GetFd(); // 获取客户端socket，fd</p>
<p>isJsonResponse = false;</p>
<p>// request_.Init(); // HTTP请求初始化</p>
<p>if(readBuff_.ReadableBytes() &lt;= 0) {</p>
<p>// 连接异常或空请求</p>
<p>LOG_INFO("连接异常或空请求");</p>
<p>return ERROR;</p>
<p>}</p>
<p>// 解析HTTP请求</p>
<p>HttpRequest::PARSE_STATE result = request_.parse(readBuff_, fd);</p>
<p></p>
<p>if(result == HttpRequest::PARSE_STATE::AGAIN) {</p>
<p>// 数据不够，继续监听 EPOLLIN</p>
<p>return AGAIN;</p>
<p>}</p>
<p>if(result == HttpRequest::PARSE_STATE::ERROR) {</p>
<p>response_.Init(srcDir, request_.path(), request_.body(), request_.header(), false, 400);</p>
<p>return FINISH;</p>
<p>}</p>
<p>RouteRequest(); // 新增函数：分发逻辑处理</p>
<p>// 构造http响应</p>
<p>response_.MakeResponse(writeBuff_,isJsonResponse);</p>
<p>/* 响应头 */</p>
<p>iov_[0].iov_base = const_cast&lt;char*&gt;(writeBuff_.Peek());</p>
<p>iov_[0].iov_len = writeBuff_.ReadableBytes(); // 响应头和长度</p>
<p>iovCnt_ = 1; // 默认只有响应头</p>
<p></p>
<p>/* 文件 */</p>
<p>if(response_.FileLen() &gt; 0 &amp;&amp; response_.File()) {</p>
<p>iov_[1].iov_base = response_.File();</p>
<p>iov_[1].iov_len = response_.FileLen();</p>
<p>iovCnt_ = 2; // 如果有文件，则同时发送响应头和文件</p>
<p>}</p>
<p>LOG_INFO("filesize:%d, %d to %d", response_.FileLen() , iovCnt_, ToWriteBytes());</p>
<p>// 当前请求处理完后，准备下一次请求，清空状态</p>
<p>request_.Init();</p>
<p>return FINISH;</p>
<p>}</p>
<p></p>
<p>void HttpConn::RouteRequest() {</p>
<p>const auto&amp; method = request_.method();</p>
<p>const auto&amp; path = request_.path();</p>
<p>cout&lt;&lt;"method:"&lt;&lt;method.c_str()&lt;&lt;endl;</p>
<p>cout&lt;&lt;"path:"&lt;&lt;path.c_str()&lt;&lt;endl;</p>
<p>if (method == "GET") {</p>
<p>if (path.find("/showlist") != std::string::npos) {</p>
<p>if (!ExtractLoginFromCookie()) {</p>
<p>response_.Init(srcDir, request_.path(), request_.body(), request_.header(), false, 403);</p>
<p>response_.SetJsonResponse("请先登录后再查看文件列表", 403);</p>
<p>isJsonResponse = true;</p>
<p>return;</p>
<p>}</p>
<p>std::string jsonStr = GetSQLFileListJson(); // 已登录，安全查询</p>
<p>response_.SetJsonResponse(jsonStr, 200); // 返回 JSON 列表</p>
<p>isJsonResponse = true;</p>
<p>isJsonResponse = true;</p>
<p>}else if (path.find("/logout") != std::string::npos) {</p>
<p>HandleLogout(); // 登出入口</p>
<p>isJsonResponse = false;</p>
<p>}</p>
<p>else {</p>
<p>response_.Init(srcDir, request_.path(), request_.body(), request_.header(), request_.IsKeepAlive(), 200);</p>
<p>}</p>
<p>} else if (method == "POST") {</p>
<p>if (path.find("/login") == 0 || path.find("/register") == 0) {</p>
<p>HandleUserAuth(); // 设置 path_ 和 code_</p>
<p>isJsonResponse = false;</p>
<p>} else if (path.find("/upload") == 0) {</p>
<p></p>
<p>if (!ExtractLoginFromCookie()) {</p>
<p>// 未登录，直接返回 403 Forbidden</p>
<p>response_.Init(srcDir, request_.path(), request_.body(), request_.header(), false, 403);</p>
<p>response_.SetJsonResponse("请先登录后再上传文件",403);</p>
<p></p>
<p>isJsonResponse = true;</p>
<p>return;</p>
<p>}</p>
<p>HandleUpload(); // 处理上传</p>
<p>isJsonResponse = true;</p>
<p>}</p>
<p>} else if (method == "DELETE" &amp;&amp; path.find("/delete") == 0) {</p>
<p>HandleDelete(); // 设置删除路径</p>
<p>isJsonResponse = true;</p>
<p>} else {</p>
<p>response_.Init(srcDir, request_.path(), request_.body(), request_.header(), false, 400);</p>
<p>}</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

