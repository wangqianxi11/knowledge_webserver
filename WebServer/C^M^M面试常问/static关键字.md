---
title: static关键字
updated: 2025-08-22T16:28:41
created: 2025-04-15T15:55:54
---

## 核心思想是控制标识符的“可见性”和“生命周期”，使其对特定的范围（文件、函数、类）而不是单个对象绑定。
## 1. static 修饰全局变量
static **修饰全局变量**可以**将变量的作用域限定在当前文件中**，使得其他文件无法访问该变量。 同时，static 修饰的全局变量**在程序启动时被初始化**（可以简单理解为在执行 main 函数之前，会执行一个全局的初始化函数，在那里会执行全局变量的初始化），生命周期和程序一样长。
<table>
<colgroup>
<col style="width: 16%" />
<col style="width: 83%" />
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
<th><p>// a.cpp 文件</p>
<p>static int a = 10; // static 修饰全局变量</p>
<p>int main() {</p>
<p>a++; // 合法，可以在当前文件中访问 a</p>
<p>return 0;</p>
<p>}</p>
<p></p>
<p>// b.cpp 文件</p>
<p>extern int a; // 声明 a</p>
<p>void foo() {</p>
<p>a++; // 非法，会报链接错误，其他文件无法访问 a</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
关于这里为什么会是非法、报链接错误，可以看看这篇文章：[extern 的作用-从链接角度理解](https://csguide.cn/cpp/basics/extern.html)
了解一下 extern 和 链接方面的知识。
## 2. static 修饰局部变量
static **修饰局部变量可以使得变量在函数调用结束后不会被销毁**，而是一直存在于内存中，下次调用该函数时可以继续使用。
同时，由于 static 修饰的局部变量的作用域仅限于函数内部，所以其他函数无法访问该变量。
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
<p>5</p>
<p>6</p>
<p>7</p>
<p>8</p>
<p>9</p>
<p>10</p>
<p>11</p></th>
<th><p>void foo() {</p>
<p>static int count = 0; // static 修饰局部变量</p>
<p>count++;</p>
<p>cout &lt;&lt; count &lt;&lt; endl;</p>
<p>}</p>
<p>int main() {</p>
<p>foo(); // 输出 1</p>
<p>foo(); // 输出 2</p>
<p>foo(); // 输出 3</p>
<p>return 0;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
## 3. static 修饰函数
static 修饰函数可以将函数的作用域限定在当前文件中，使得其他文件无法访问该函数。
同时，由于 static 修饰的函数只能在当前文件中被调用，因此可以避免命名冲突和代码重复定义。
<table>
<colgroup>
<col style="width: 12%" />
<col style="width: 87%" />
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
<th><p>// a.cpp 文件</p>
<p>static void foo() { // static 修饰函数</p>
<p>cout &lt;&lt; "Hello, world!" &lt;&lt; endl;</p>
<p>}</p>
<p>int main() {</p>
<p>foo(); // 合法，可以在当前文件中调用 foo 函数</p>
<p>return 0;</p>
<p>}</p>
<p>// b.cpp 文件</p>
<p>extern void foo(); // 声明 foo</p>
<p>void bar() {</p>
<p>foo(); // 非法，会报链接错误，找不到 foo 函数，其他文件无法调用 foo 函数</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
## 4. static 修饰类成员变量和函数
static 修饰类成员变量和函数可以使得它们在所有类对象中共享，且不需要创建对象就可以直接访问。
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
<p>5</p>
<p>6</p>
<p>7</p>
<p>8</p>
<p>9</p>
<p>10</p></th>
<th><p>class MyClass {</p>
<p>public:</p>
<p>static int count; // static 修饰类成员变量</p>
<p>static void foo() { // static 修饰类成员函数</p>
<p>cout &lt;&lt; count &lt;&lt; endl;</p>
<p>}</p>
<p>};</p>
<p>// 访问：</p>
<p>MyClass::count;</p>
<p>MyClass::foo();</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

| **上下文** | **static的作用** | 关键点 |
|----|----|----|
| 全局变量/函数 | 将链接性从外部改为内部，限制其只在当前文件内可见 | 信息隐藏，避免命名冲突 |
| 局部变量 | 将变量的存储期从自动改为静态，使其在程序生命周期内存在 | 保持函数调用间的状态 |
| 类成员变量 | 变量属于类，被所有对象共享 | 共享数据，需在类外定义 |
| 类成员函数 | 函数属于类，没有this指针，只能访问静态成员 | 提供不依赖于对象实例的操作 |
