---
title: extern作用
updated: 2025-04-29T16:24:52
created: 2025-04-15T15:51:02
---

一般而言，C++全局变量的作用范围仅限于当前的文件，但同时C++也支持分离式编译，允许将程序分割为若干个文件被独立编译。
于是就需要在文件间共享变量数据，这里extern就发挥了作用。
extern用于指示变量或函数的定义在另一个源文件中，并在当前源文件中声明。 说明该符号具有外部链接(external linkage)属性。
也就是告诉编译器: 这个符号在别处定义了，你先编译，到时候链接器会去别的地方找这个符号定义的地址。
# 1. 符号的声明与定义
首先明白 C/C++ 中变量的声明和定义是两个不同的概念。 声明是指告诉编译器某个符号的存在，在程序变量表中记录类型和名字，而定义则是指为该符号分配内存空间或实现其代码逻辑。
凡是没有带extern的声明同时也都是定义。 而对函数而言，带有{}是定义，否则是声明。如果想声明一个变量而非定义它，就在变量名前添加关键字extern，且不要显式的初始化变量。
## 1.1 变量的声明与定义
<table>
<colgroup>
<col style="width: 27%" />
<col style="width: 72%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p></th>
<th><p>// 声明</p>
<p>extern int global_var;</p>
<p></p>
<p>// 定义</p>
<p>int global_var = 42;</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
在上面的示例中，global_var变量的声明使用extern关键字告诉编译器它的定义在其他源文件中，而定义则是为变量分配内存空间并初始化为 42。
1.2 函数的声明和定义：
<table>
<colgroup>
<col style="width: 26%" />
<col style="width: 73%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p></th>
<th><p>// 声明</p>
<p>int sum(int a, int b);</p>
<p>// 定义</p>
<p>int sum(int a, int b) {</p>
<p>return a + b;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
在上面的示例中，sum函数的声明告诉编译器该函数的存在及其参数和返回值类型，而定义则是实现函数的代码逻辑。

### extern_c的作用
主要用于解决C++和C语言混合编程时的名称修饰和链接问题，
1.  禁止C++的名称修饰
2.  确保C和C++之间的正确链接

