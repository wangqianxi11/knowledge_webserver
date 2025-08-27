---
title: malloc和new
updated: 2025-08-22T18:37:24
created: 2025-04-15T11:02:08
---

# new和malloc两者的区别
## 属性的区别
new/delete：这两个是C++中的关键字，若要使用，需要编译器支持；
malloc/free：这两个是库函数，若要使用则需要引入相应的头文件才可以正常使用。
## 使用上的区别
malloc：申请空间需要显式填入申请内存的大小；
new：无需显式填入申请的内存大小，new会根据new的类型分配内存。
实例：
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
<p>5</p></th>
<th><p>/** malloc/free用例 **/</p>
<p>int*ma = (int*)malloc(4)；</p>
<p>free(ma)；</p>
<p>/** new/delete用例 **/</p>
<p>int*ne =new int(0);</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
## 内存位置的区别
new：此操作符分配的内存空间是在[自由存储区](https://zhida.zhihu.com/search?content_id=163350214&content_type=Article&match_order=1&q=%E8%87%AA%E7%94%B1%E5%AD%98%E5%82%A8%E5%8C%BA&zhida_source=entity)；
malloc：申请的内存是在[堆空间](https://zhida.zhihu.com/search?content_id=163350214&content_type=Article&match_order=1&q=%E5%A0%86%E7%A9%BA%E9%97%B4&zhida_source=entity)。

C/C++的内存通常分为：**堆、栈、自由存储区、全局/静态存储区、常量存储区**。可能除了自由存储区，其他的内存分布大家应该都比较熟悉。
**堆**是C语言和操作系统的术语，堆是操作系统所维护的一块特殊内存，它提供了动态分配的功能，当运行程序调用malloc()时就会从中分配，调用free()归还内存。

**自由存储区**是C++中动态分配和释放对象的一个概念，通过new分配的内存区域可以称为自由存储区，通过delete释放归还内存。**自由存储区可以是堆、全局/静态存储区等，具体是在哪个区，主要还是要看new的实现以及C++编译器默认new申请的内存是在哪里**。但是基本上，很多C++编译器默认使用堆来实现自由存储，运算符new和delete内部默认是使用malloc和free的方式来被实现，说它在堆上也对，说它在自由存储区上也正确。因为在C++中new和delete符号是可以重载的，我们可以重新实现new的实现代码，可以让其分配的内存位置在静态存储区等。而malloc和free是C里的库函数，无法对其进行重载。
## 返回类型的区别
- new操作符内存分配成功时，**返回的是对象类型的指针**，类型严格与对象匹配，无须进行类型转换，故new是符合[类型安全性](https://zhida.zhihu.com/search?content_id=163350214&content_type=Article&match_order=1&q=%E7%B1%BB%E5%9E%8B%E5%AE%89%E5%85%A8%E6%80%A7&zhida_source=entity)的操作符。
- malloc内存分配成功则是返回void \* ，需要通过强制类型转换将void\*指针转换成我们需要的类型。所以在C++程序中使用new会比malloc安全可靠。
## 分配失败情况的区别
- malloc分配内存失败时返回NULL，我们可以通过判断返回值可以得知是否分配成功；
- new内存分配失败时，会抛出bac_alloc异常，它不会返回NULL，分配失败时如果不捕捉异常，那么程序就会异常退出，我们可以通过[异常捕捉](https://zhida.zhihu.com/search?content_id=163350214&content_type=Article&match_order=1&q=%E5%BC%82%E5%B8%B8%E6%8D%95%E6%8D%89&zhida_source=entity)的方式获取该异常。
## 定义对象系统调度过程的区别
使用new操作符来分配对象内存时会经历三个步骤：
- 调用operator new 函数（对于数组是operator new\[\]）分配一块足够的内存空间（通常底层默认使用malloc实现，除非程序员重载new符号）以便存储特定类型的对象；
- 编译器运行相应的构造函数以构造对象，并为其传入初值。
- 对象构造完成后，返回一个指向该对象的指针。
使用delete操作符来释放对象内存时会经历两个步骤：
- 调用对象的析构函数。
- 编译器调用operator delete(或operator delete\[\])函数释放内存空间（通常底层默认使用free实现，除非程序员重载delete符号）。

## 扩张内存大小的区别
- malloc：使用malloc分配内存后，发现内存不够用，那我们可以通过[realloc](https://zhida.zhihu.com/search?content_id=163350214&content_type=Article&match_order=1&q=realloc&zhida_source=entity)函数来扩张内存大小，realloc会先判断当前申请的内存后面是否还有足够的内存空间进行扩张，如果有足够的空间，那么就会往后面继续申请空间，并返回原来的地址指针；否则realloc会在另外有足够大小的内存申请一块空间，并将当前内存空间里的内容拷贝到新的内存空间里，最后返回新的地址指针。
- new：**new没有扩张内存的机制。**

| 特征 | new/delete | malloc/free |
|----|----|----|
| 分配内存的位置 | 自由存储区 | 堆 |
| 内存分配成功的返回值 | 完整类型指针 | void\* |
| 内存分配失败的返回值 | 默认抛出异常 | 返回NULL |
| 分配内存的大小 | 由编译器根据类型计算得出 | 必须显式指定字节数 |
| 处理数组 | 有处理数组的new版本new\[\] | 需要用户计算数组的大小后进行内存分配 |
| 已分配内存的扩充 | 无法直观地处理 | 使用realloc简单完成 |
| 是否相互调用 | 可以，看具体的operator new/delete实现 | 不可调用new |
| 分配内存时内存不足 | 客户能够指定处理函数或重新制定分配器 | 无法通过用户代码进行处理 |
| 函数重载 | 允许 | 不允许 |
| 构造函数与析构函数 | 调用 | 不调用 |

**
