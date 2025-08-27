---
title: C++指针传递、值传递、引用传递
updated: 2025-04-15T16:05:23
created: 2025-04-15T16:03:50
---

在 C++ 中，函数参数传递有三种常见的方式：值传递、引用传递和指针传递。以下分别给出这三种方式的示例：
## 一、值传递（Value Passing）
值传递是将实参的值传递给形参。在这种情况下，函数内对形参的修改不会影响到实参。
示例:
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
<p>15</p></th>
<th><p>#include &lt;iostream&gt;</p>
<p></p>
<p>void swap_value(int a, int b) {</p>
<p>int temp = a;</p>
<p>a = b;</p>
<p>b = temp;</p>
<p>}</p>
<p></p>
<p>int main() {</p>
<p>int x = 10;</p>
<p>int y = 20;</p>
<p>swap_value(x, y);</p>
<p>std::cout &lt;&lt; "x: " &lt;&lt; x &lt;&lt; ", y: " &lt;&lt; y &lt;&lt; std::endl; // 输出：x: 10, y: 20</p>
<p>return 0;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
## 二、引用传递（Reference Passing）
引用传递是将实参的引用传递给形参。在这种情况下，函数内对形参的修改会影响到实参。
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
<p>13</p></th>
<th><p>#include &lt;iostream&gt;</p>
<p>void swap_reference(int &amp;a, int &amp;b) {</p>
<p>int temp = a;</p>
<p>a = b;</p>
<p>b = temp;</p>
<p>}</p>
<p>int main() {</p>
<p>int x = 10;</p>
<p>int y = 20;</p>
<p>swap_reference(x, y);</p>
<p>std::cout &lt;&lt; "x: " &lt;&lt; x &lt;&lt; ", y: " &lt;&lt; y &lt;&lt; std::endl; // 输出：x: 20, y: 10</p>
<p>return 0;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
## 三、指针传递（Pointer Passing）
指针传递是将实参的地址传递给形参。在这种情况下，函数内对形参的修改会影响到实参。
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
<p>13</p></th>
<th><p>#include &lt;iostream&gt;</p>
<p>void swap_pointer(int *a, int *b) {</p>
<p>int temp = *a;</p>
<p>*a = *b;</p>
<p>*b = temp;</p>
<p>}</p>
<p>int main() {</p>
<p>int x = 10;</p>
<p>int y = 20;</p>
<p>swap_pointer(&amp;x, &amp;y);</p>
<p>std::cout &lt;&lt; "x: " &lt;&lt; x &lt;&lt; ", y: " &lt;&lt; y &lt;&lt; std::endl; // 输出：x: 20, y: 10</p>
<p>return 0;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
