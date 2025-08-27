---
title: C++内存分区
updated: 2025-05-10T13:34:09
created: 2025-04-15T15:59:43
---

一般来说，程序运行时，代码、数据等都存放在不同的内存区域，这些内存区域从逻辑上做了划分，大概以下几个区域：代码区、全局/静态存储区、栈区、堆区和常量区。
在 CSAPP 第九章虚拟内存，就将内存分为堆、bss、data、txt、栈等区域。
高地址
┌─────────────┐
│ 栈(stack) │ ← 局部变量、函数调用等
├─────────────┤
│ ↓ │
│ ↑ │
├─────────────┤
│ 堆(heap) │ ← 动态分配的内存
├─────────────┤
│ .bss │ ← 未初始化的静态/全局变量
├─────────────┤
│ .data │ ← 已初始化的静态/全局变量
├─────────────┤
│ .text │ ← 程序代码
└─────────────┘
低地址

## 代码区（Code Segment）
也就是 .text 段， 代码区存放程序的二进制代码，它是只读的，以防止程序在运行过程中被意外修改。
<table>
<colgroup>
<col style="width: 15%" />
<col style="width: 84%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p></th>
<th><p>#include &lt;iostream&gt;</p>
<p>int main() {</p>
<p>std::cout &lt;&lt; "Hello, World!" &lt;&lt; std::endl;</p>
<p>return 0;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
比如上面这段代码中的 main 函数，编译为二进制后，函数的逻辑就存放在代码区。
当然这段区域也有可能包含一些只读的常数变量，例如字符串常量等。
## 全局/静态存储区（Global/Static Storage）
全局变量和静态变量都存放在全局/静态存储区。
以前在 **C 语言中全局变量又分为初始化的和未初始化的，分别放在上面图中的 .bss 和 .data 段**，但在 C++里面没有这个区分了，他们共同占用同一块内存区，就叫做全局存储区。
这个区域的内存在程序的生命周期几乎都是全局的，举例:
<table>
<colgroup>
<col style="width: 17%" />
<col style="width: 82%" />
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
<th><p>#include &lt;iostream&gt;</p>
<p>int globalVar = 0; // 全局变量</p>
<p>void function() {</p>
<p>static int staticVar = 0; // 静态变量</p>
<p>staticVar++;</p>
<p>std::cout &lt;&lt; staticVar &lt;&lt; std::endl;</p>
<p>}</p>
<p>int main() {</p>
<p>function();</p>
<p>function();</p>
<p>return 0;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
globalVar是一个全局变量，staticVar是一个静态变量，它们都存放在全局/静态存储区。
## 栈区（Stack）
栈区用于存储函数调用时的局部变量、函数参数以及返回地址。
当函数调用完成后，分配给这个函数的栈空间会被释放。例如：
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
<p>9</p></th>
<th><p>#include &lt;iostream&gt;</p>
<p>void function(int a, int b) {</p>
<p>int localVar = a + b;</p>
<p>std::cout &lt;&lt; localVar &lt;&lt; std::endl;</p>
<p>}</p>
<p>int main() {</p>
<p>function(3, 4);</p>
<p>return 0;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
在这个例子中，a、b和localVar都是局部变量，它们存放在栈区。
当 function 函数调用结束后，对应的函数栈所占用的空间(参数 a、b，局部变量 localVar等)都会被回收。
## 堆区（Heap）
堆区是用于动态内存分配的区域，当使用new（C++）或者malloc（C）分配内存时，分配的内存块就位于堆区。
我们需要手动释放这些内存，否则可能导致内存泄漏。例如：
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
<p>7</p></th>
<th><p>#include &lt;iostream&gt;</p>
<p>int main() {</p>
<p>int* dynamicArray = new int[10]; // 动态分配内存</p>
<p>// 使用动态数组...</p>
<p>delete[] dynamicArray; // 释放内存</p>
<p>return 0;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
## 常量区（Constant Storage）：
常量区用于存储常量数据，例如字符串字面量和其他编译时常量。这个区域通常也是只读的。例如：
<table>
<colgroup>
<col style="width: 15%" />
<col style="width: 84%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p></th>
<th><p>#include &lt;iostream&gt;</p>
<p>int main() {</p>
<p>char* c="abc"; // abc在常量区，c在栈上。</p>
<p>return 0;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
总结
上面这些分区比较细节，最基础的要求是我们必须掌握下面这些概念：
1.  代码和数据是分开存储的
2.  堆和栈的不同区别
3.  全局变量和局部变量的存储区别
