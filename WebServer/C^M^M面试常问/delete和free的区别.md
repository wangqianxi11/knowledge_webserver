---
title: delete和free的区别
updated: 2025-08-22T18:39:24
created: 2025-05-20T15:12:30
---

### 1.delete（C++）
**作用**：
- 用于释放由new运算符动态分配的单个对象或数组。
- 会调用对象的**析构函数**（如果存在），确保资源正确释放（如文件句柄、锁等）。
**语法**：
<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th><p>// 释放单个对象</p>
<p>int* ptr = new int(42);</p>
<p>delete ptr; // 调用析构函数（如果是类对象），并释放内存</p>
<p>// 释放数组</p>
<p>int* arr = new int[10];</p>
<p>delete[] arr; // 必须用 delete[] 释放数组</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
**特点**：
- 是C++的运算符，与new配对使用。
- 在释放内存前执行析构函数（避免资源泄漏）。
- 如果误用（如对数组用delete而非delete\[\]），可能导致未定义行为（UB）。

### 2.free（C）
**作用**：
- 用于释放由==<sub></sub>​malloc、\`calloc、​<sub></sub>==realloc分配的内存。
- **不会调用析构函数**，仅直接释放内存。
**语法**：
<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th><p>int* ptr = (int*)malloc(sizeof(int));</p>
<p>free(ptr); // 仅释放内存，无析构函数调用</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
**特点**：
- 是C标准库函数（\<stdlib.h\>）。
- 仅释放内存，不处理对象的清理逻辑（适合纯数据或POD类型）。
- 误用（如重复free或释放非堆内存）会导致程序崩溃或安全漏洞。

**关键区别**
| **特性** | **delete(C++)** | **free(C)** |
|----|----|----|
| **语言** | C++ | C |
| **配对操作** | ==new==/==new\[\]== | ==malloc==/==calloc== |
| **析构函数** | 会调用 | 不会调用 |
| **类型安全** | 是（自动计算大小） | 否（需手动指定大小） |
| **数组释放** | 必须用==delete\[\]== | 直接==free==即可 |
| **错误风险** | 误用可能导致UB | 误用可能导致内存泄漏或崩溃 |
