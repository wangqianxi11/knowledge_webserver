---
title: HTTP响应
updated: 2025-04-08T10:07:40
created: 2025-04-08T09:39:30
---

HTTP响应
2025年4月8日
9:39

**HTTP响应的主要逻辑，根据resources中是否存在响应资源赋予不同的响应状态码，并将响应状态行、响应头部、正文写入到buff中。这个响应函数只能响应html。**
<table>
<colgroup>
<col style="width: 11%" />
<col style="width: 88%" />
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
<p>29</p></th>
<th><p>void HttpResponse::MakeResponse(Buffer&amp; buff) {</p>
<p>/*</p>
<p>根据请求的文件路径检查资源状态，将响应状态行、头部、正文按顺序写入到buff中</p>
<p>写入buff的内容如下：</p>
<p>HTTP/1.1 200 OK\r\n</p>
<p>Content-Type: text/html\r\n</p>
<p>Content-Length: 1024\r\n</p>
<p>Connection: keep-alive\r\n</p>
<p>\r\n</p>
<p>&lt;html&gt;...&lt;/html&gt; （或文件内容）</p>
<p>*/</p>
<p>/* 判断请求的资源文件 */</p>
<p>if(stat((srcDir_ + path_).data(), &amp;mmFileStat_) &lt; 0 || S_ISDIR(mmFileStat_.st_mode)) {</p>
<p>// stat()获取文件信息，不存在返回404</p>
<p>code_ = 404;</p>
<p>}</p>
<p>else if(!(mmFileStat_.st_mode &amp; S_IROTH)) {</p>
<p>// 检查读写权限，返回403</p>
<p>code_ = 403;</p>
<p>}</p>
<p>else if(code_ == -1) {</p>
<p>// 正常返回200</p>
<p>code_ = 200;</p>
<p>}</p>
<p>ErrorHtml_();</p>
<p>AddStateLine_(buff);</p>
<p>AddHeader_(buff);</p>
<p>AddContent_(buff);</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

