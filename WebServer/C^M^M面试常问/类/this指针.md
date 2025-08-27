---
title: this指针
updated: 2025-05-10T10:22:05
created: 2025-05-10T10:13:29
---

是一个特殊的指针，指向当前对象的实例
每一个对象都通过this指针访问自己的地址
是一个**隐藏**的指针，可以在类的成员函数中使用，可以用来指向调用对象
友元函数没有this指针，友元函数不是类的成员函数
静态成员函数没有this指针
示例：
<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th><p>#include &lt;iostream&gt;</p>
<p>class MyClass {</p>
<p>private:</p>
<p>int value;</p>
<p>public:</p>
<p>void setValue(int value) {</p>
<p>this-&gt;value = value;</p>
<p>}</p>
<p>void printValue() {</p>
<p>std::cout &lt;&lt; "Value: " &lt;&lt; this-&gt;value &lt;&lt; std::endl;</p>
<p>}</p>
<p>};</p>
<p>int main() {</p>
<p>MyClass obj;</p>
<p>obj.setValue(42);</p>
<p>obj.printValue();</p>
<p>return 0;</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
