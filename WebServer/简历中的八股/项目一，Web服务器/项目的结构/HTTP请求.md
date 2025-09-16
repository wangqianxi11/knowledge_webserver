---
title: HTTP请求
updated: 2025-05-19T22:40:15
created: 2025-04-07T19:48:37
---

|    |
|-----|
**一、设计思路：正则表达式 + 状态机**
**核心目标**：从客户端接收到的原始 HTTP 报文中提取出：请求行、请求头、请求体，构造成结构化的请求对象。
**1. 正则表达式处理的部分**
适合处理**格式固定、一次性匹配的结构**，比如：
- **请求行（Request Line）**：  
  GET /index.html HTTP/1.1\r\n  
  正则：^(\w+)\s+(\[^\s\]+)\s+HTTP/(\d\\\d)\r\n
- **请求头（Header）**：  
  
  Host: example.com\r\n  
  Content-Length: 123\r\n  

  正则：^(\[\w-\]+):\s\*(.\*)\r\n
你可以使用正则批量提取头部字段为键值对，封装成 std::unordered_map。

**2. 状态机处理的部分**
处理**数据接收非原子性 + 报文不完整的情况**，按状态逐步推进解析过程：
```c++
enum class ParseState {

  REQUEST_LINE,

  HEADERS,

  BODY,

  DONE,

  ERROR

};
```
每次接收到数据后：
- 当前状态下使用正则匹配处理对应内容
- 如果成功匹配 → 切换到下一个状态
- 如果数据不完整 → 等待更多数据（状态不变）
- 如果解析失败 → 设置 ERROR 状态

**✅ 二、完整的解析流程简要**

1\. REQUEST_LINE → 使用正则提取 method, path, version  
1. HEADERS → 使用正则逐行匹配 headers，直到遇到空行  
2. BODY → 根据 Content-Length 读取指定字节数内容  
3. DONE → 解析完成，交由业务逻辑处理

**三、常见边界情况分析**
| **场景** | **问题描述** | **解决方法** |
|----|----|----|
| 请求数据分多次到达 | TCP 粘包/拆包，导致请求行或头部不完整 | 状态机 + buffer 拼接缓存 |
| 客户端发来非法请求 | 如 method 不合法、缺少冒号的头部行 | 正则匹配失败 → 设置 ERROR 状态 |
| 报文头太大 / Content-Length 过大 | 可能是恶意请求，占用大量内存 | 设置头部/体积大小上限，超过直接断开连接 |
| Transfer-Encoding: chunked | 正则无法直接处理分块编码体 | 若支持 chunked，需要引入单独 chunk 状态逻辑 |
| 不支持的 HTTP 方法 | 比如 TRACE, CONNECT 等 | 匹配后判断合法性，非法返回 405 Method Not Allowed |
| 非标准格式的换行符 | \n 而不是 \r\n | 可预处理标准化换行或使用宽松匹配 |

## 初始化函数
```c++
void HttpRequest::Init() {

  // 初始化成员变量

  method_ = path_ = version_ = body_ = ""; // 初始化为空

  state_ = REQUEST_LINE; // 初始化状态机状态为请求行

  header_.clear(); // 清空请求头

  post_.clear(); // 清空请求体

}
```

**检查HTTP连接是否应该保持长连接**
```c++
bool HttpRequest::IsKeepAlive() const {

  if(header_.count("Connection") == 1) { // 检查请求头中是否存在“Connection字段”

    // 验证长连接条件,Connection的值是否为keep-alive以及协议版本是否为1.1

    return header_.find("Connection")->second == "keep-alive" && version_ == "1.1";

  }

  return false;

}
```

## HTTP请求的解析逻辑，使用状态机逐段处理请求数据**
```c++
bool HttpRequest::parse(Buffer& buff, int& fd) {

  const char CRLF[] = "\r\n";

  if (buff.ReadableBytes() <= 0) {

    return false;

  }

  // 分阶段解析

  while (buff.ReadableBytes() && state_ != FINISH) {

  // 状态机非BODY状态

  if (state_ != BODY) {

  // 非 BODY 状态：按行解析（请求行、请求头）

  const char* lineEnd = search(buff.Peek(), buff.BeginWriteConst(), CRLF, CRLF + 2);

  std::string line(buff.Peek(), lineEnd); // 逐行读取,以\r\n为分隔符逐行解析

  switch (state_) {

  case REQUEST_LINE: // 请求行，包括URL，协议版本,失败则直接返回

  if (!ParseRequestLine_(line)) return false;

  ParsePath_(); // 规范化path_

  break;

  case HEADERS: // 请求头

  ParseHeader_(line); // 存储键值对函数

  if (buff.ReadableBytes() <= 2) { // 空行 \r\n 表示头部结束

    state_ = BODY; // 切换到BODY状态

  }

  break;

  default:

  break;

  }

  if (lineEnd == buff.BeginWriteConst()) break;

  buff.RetrieveUntil(lineEnd + 2); // 移动指针到下一行

  } else {

  // BODY 状态：完整读取 body

  if (header_.count("Content-Length")) {

    // 根据Content-Length读取指定长度的Body数据，

    size_t content_length = std::stoul(header_["Content-Length"]);

    if (buff.ReadableBytes() < content_length) { // 长度不足，报错

    LOG_INFO("Incomplete body: expected %zu, got %zu",

    content_length, buff.ReadableBytes());

    return false;

  }

  body_.assign(buff.Peek(), content_length); // 重新申请body_大小

  buff.Retrieve(content_length);

  } else if (header_.count("Transfer-Encoding") &&

  header_["Transfer-Encoding"] == "chunked") {

  // 处理分块数据

  if (!ParseChunkedBody_(buff)) return false;

  } else {

    // 默认情况，读取剩余所有数据作为body

    body_.assign(buff.Peek(), buff.BeginWriteConst());

    buff.RetrieveAll();

  }

  // POST数据处理

  ParsePost_(fd);

  state_ = FINISH;

  }

  }

  LOG_INFO("[%s], [%s], [%s]", method_.c_str(), path_.c_str(), version_.c_str());

  return true;

}
```

**规范化客户端的请求路径**
<table>
<colgroup>
<col style="width: 14%" />
<col style="width: 85%" />
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
<p>17</p></th>
<th><p>void HttpRequest::ParsePath_() {</p>
<p>/*</p>
<p>规范化客户端请求的路径，其中path_根据解析的请求行</p>
<p>*/</p>
<p>if(path_ == "/") {</p>
<p>path_ = "/index.html";</p>
<p>}</p>
<p>else {</p>
<p>for(auto &amp;item: DEFAULT_HTML) {</p>
<p>if(item == path_) {</p>
<p>path_ += ".html";</p>
<p>break;</p>
<p>}</p>
<p>}</p>
<p>}</p>
<p>LOG_INFO("请求行的path:%s",path_);</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**解析请求行**
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
<p>18</p></th>
<th><p>bool HttpRequest::ParseRequestLine_(const string&amp; line) {</p>
<p>/*</p>
<p>解析HTTP请求行，并提取出请求方法(GET/POST）、请求路径(/index.html)、协议版本</p>
<p>如请求原文：GET /index.html HTTP/1.1\r\n</p>
<p>*/</p>
<p>regex patten("^([^ ]*) ([^ ]*) HTTP/([^ ]*)$"); // 正则模式：^和$表示整行匹配，（[^ ]*）表示匹配字段</p>
<p>smatch subMatch;</p>
<p>if(regex_match(line, subMatch, patten)) {</p>
<p>method_ = subMatch[1];</p>
<p>path_ = subMatch[2];</p>
<p>version_ = subMatch[3];</p>
<p>state_ = HEADERS; // 状态机推进，将解析状态从REQUEST_LINE切换到HEADERS</p>
<p>std::cout&lt;&lt;"请求行解析结束"&lt;&lt;endl;</p>
<p>return true;</p>
<p>}</p>
<p>LOG_ERROR("RequestLine Error");</p>
<p>return false;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**解析请求头部**
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
<p>18</p></th>
<th><p>void HttpRequest::ParseHeader_(const string&amp; line) {</p>
<p>/*</p>
<p>解析HTTP请求头部</p>
<p>如请求头：</p>
<p>Host: <a href="http://www.example.com">www.example.com</a></p>
<p>Connection: keep-alive</p>
<p>Content-Type: text/html</p>
<p></p>
<p>*/</p>
<p>regex patten("^([^:]*): ?(.*)$"); // 按照整行匹配，用冒号将其匹配为键值对，冒号前为key，冒号后为value</p>
<p>smatch subMatch;</p>
<p>if(regex_match(line, subMatch, patten)) {</p>
<p>header_[subMatch[1]] = subMatch[2];</p>
<p>}</p>
<p>else {</p>
<p>state_ = BODY; // 状态机推进</p>
<p>}</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**实现了HTTP POST请求的处理逻辑，分为表单数据处理和文件上产处理两大功能**
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
<p>50</p></th>
<th><p>void HttpRequest::ParsePost_(int&amp; fd) {</p>
<p>if(path_ == "/list" &amp;&amp; method_ == "GET") {</p>
<p>// 获取文件列表</p>
<p>std::vector&lt;std::string&gt; files;</p>
<p>getFileList("./resources/images", files);</p>
<p>nlohmann::json jsonResponse = files;</p>
<p>body_ = jsonResponse.dump();</p>
<p>path_ = "filelist.html";</p>
<p>}</p>
<p>if (method_ != "POST" || !header_.count("Content-Type")) return;</p>
<p></p>
<p>const std::string&amp; type = header_["Content-Type"];</p>
<p>std::cout&lt;&lt;type&lt;&lt;endl;</p>
<p>if (type =="application/x-www-form-urlencoded") {</p>
<p>ParseFromUrlencoded_();</p>
<p>std::cout&lt;&lt;post_["username"]&lt;&lt;endl;</p>
<p>std::cout&lt;&lt;post_["password"]&lt;&lt;endl;</p>
<p>// 原来的身份验证逻辑...</p>
<p>if (DEFAULT_HTML_TAG.count(path_)) {</p>
<p>int tag = DEFAULT_HTML_TAG.find(path_)-&gt;second;</p>
<p>LOG_DEBUG("Tag:%d", tag);</p>
<p>if(tag == 0 || tag == 1) {</p>
<p>bool isLogin = (tag == 1);</p>
<p>if(UserVerify(post_["username"], post_["password"], isLogin)) {</p>
<p>std::cout&lt;&lt;"验证成功"&lt;&lt;endl;</p>
<p>path_ = "/welcome.html";</p>
<p>} else {</p>
<p>std::cout&lt;&lt;"验证失败"&lt;&lt;endl;</p>
<p>path_ = "/error.html";</p>
<p>}</p>
<p>}</p>
<p>if(tag == 2){</p>
<p>path_ = "filelist.html";</p>
<p>// 可以在这里添加生成动态html的函数</p>
<p></p>
<p>}</p>
<p>}</p>
<p>}</p>
<p>else if (type.find("multipart/form-data") != std::string::npos) {</p>
<p>// 如果是上传文件</p>
<p>std::cout&lt;&lt;"上传文件"&lt;&lt;endl;</p>
<p>if (ParseMultipartFormData(type, body_)) {</p>
<p>path_ = "filelist.html"; // 上传成功后跳转页面</p>
<p>// 同样在这里生成动态Html</p>
<p>} else {</p>
<p>path_ = "/error.html"; // 失败处理</p>
<p>}</p>
<p>}</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**POST请求中处理表单的函数**
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
<p>43</p></th>
<th><p>void HttpRequest::ParseFromUrlencoded_() {</p>
<p>/*</p>
<p>解析HTTP中POST请求体中的application/x-www-form-urlencoded字段</p>
<p>处理如：username=alice&amp;password=123456</p>
<p>*/</p>
<p>if(body_.size() == 0) { return; }</p>
<p></p>
<p>string key, value;</p>
<p>int num = 0;</p>
<p>int n = body_.size();</p>
<p>int i = 0, j = 0;</p>
<p></p>
<p>for(; i &lt; n; i++) {</p>
<p>char ch = body_[i];</p>
<p>switch (ch) {</p>
<p>case '=': // 找到“=”,表示第一个键结束</p>
<p>key = body_.substr(j, i - j); // 提取第一个键</p>
<p>j = i + 1;</p>
<p>break;</p>
<p>case '+': // "+"是空格的转义</p>
<p>body_[i] = ' ';</p>
<p>break;</p>
<p>case '%': // 解析URL编码字符，“%XX”，这段写法不规范，需要改进</p>
<p>num = ConverHex(body_[i + 1]) * 16 + ConverHex(body_[i + 2]); // 将X转换成对应的16进制数,组合成一个 num = 高4位 * 16 + 低4位</p>
<p>body_[i + 2] = num % 10 + '0';</p>
<p>body_[i + 1] = num / 10 + '0'; // 将%XX 的结果用两位十进制数字覆盖原本位置</p>
<p>i += 2;</p>
<p>break;</p>
<p>case '&amp;': // 找到“&amp;”，代表前面的键值对结束</p>
<p>value = body_.substr(j, i - j);</p>
<p>j = i + 1;</p>
<p>post_[key] = value; // 将键值对保存在post_中</p>
<p>LOG_DEBUG("%s = %s", key.c_str(), value.c_str());</p>
<p>break;</p>
<p>default:</p>
<p>break;</p>
<p>}</p>
<p>}</p>
<p>if(post_.count(key) == 0 &amp;&amp; j &lt; i) {</p>
<p>value = body_.substr(j, i - j);</p>
<p>post_[key] = value;</p>
<p>}</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
