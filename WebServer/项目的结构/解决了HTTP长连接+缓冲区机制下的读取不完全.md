---
title: 解决了HTTP长连接+缓冲区机制下的读取不完全
updated: 2025-04-15T22:13:19
created: 2025-04-15T22:09:17
---

经常出现在基于Reactor模型+Epoll+自定义缓冲区的高性能服务器中。

问题本质是：
- 页面发起 multipart/form-data 上传请求（POST 请求，带 body）
- socket 读取时，只读取到了头部（header），读取不到 body
- **重启服务器后却能正常读取完整数据**
**1. 数据还未到达 recv 缓冲区**

你调用了 read 或 readv 后只读取到了请求头，但实际上请求体尚未完全到达内核缓冲区。

这在 Chrome 或其他浏览器发送大 body 时非常常见 —— 数据是**分段发送**的，尤其是上传文件。

因此你必须：

**支持“读不完整”的场景，等待更多数据到来后再继续解析。**

**2. 你只读了一次，没有持续等待**

若你用 epoll（比如 EPOLLIN）触发后只读了一次 socket，而没有循环检查是否还有更多数据（比如 readableBytes() \< content-length 就直接返回），则也会导致**读取中断**。

**3. 重启服务器后能正常工作，是因为浏览器“重发了请求”，而这次发的很快，body 一次性就到了**

所以重启后之所以能读取完整 body，是因为：
- 新连接 = 全新的 socket
- 浏览器重发请求 = 数据可能直接以更大块形式到达
- 所以第一次读取就包含了完整头+体，才会 **“误以为问题解决了”**
# 解决方法：
## 【1】读取时要支持“未读完”的情况
你的读取逻辑应该像这样：

<table>
<colgroup>
<col style="width: 18%" />
<col style="width: 81%" />
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
<th><p>ssize_t len = read(fd, buffer, ...);</p>
<p></p>
<p>if (len == 0) {</p>
<p>// 客户端关闭连接</p>
<p>}</p>
<p>else if (len &lt; 0 &amp;&amp; errno == EAGAIN) {</p>
<p>// 数据还没准备好，下次继续</p>
<p>}</p>
<p>else {</p>
<p>// 读取成功，将数据 append 到 buff 中</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
## 【2】解析逻辑要“状态机化”，支持头部和 body 分阶段解析
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
<p>33</p></th>
<th><p>enum PARSE_STATE {</p>
<p>REQUEST_LINE,</p>
<p>HEADERS,</p>
<p>BODY,</p>
<p>FINISH</p>
<p>};</p>
<p></p>
<p>bool HttpRequest::parse(Buffer&amp; buff, int&amp; fd) {</p>
<p>while (buff.ReadableBytes() &amp;&amp; state_ != FINISH) {</p>
<p>if (state_ != BODY) {</p>
<p>// 每次读取一行数据</p>
<p>const char* lineEnd = searchCRLF(...);</p>
<p>std::string line(buff.Peek(), lineEnd);</p>
<p></p>
<p>if (state_ == HEADERS &amp;&amp; line.empty()) {</p>
<p>state_ = BODY;</p>
<p>}</p>
<p>}</p>
<p></p>
<p>if (state_ == BODY) {</p>
<p>if (buff.ReadableBytes() &gt;= content_length) {</p>
<p>body_ = std::string(buff.Peek(), content_length);</p>
<p>buff.Retrieve(content_length);</p>
<p>state_ = FINISH;</p>
<p>} else {</p>
<p>// 数据不够，返回 false，等待 epoll 再次触发</p>
<p>return false;</p>
<p>}</p>
<p>}</p>
<p>}</p>
<p></p>
<p>return true;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

