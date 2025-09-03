---
title: C++指针传递、值传递、引用传递
updated: 2025-04-15T16:05:23
created: 2025-04-15T16:03:50
---

在 C++ 中，函数参数传递有三种常见的方式：<b>值传递、引用传递和指针传递。</b>以下分别给出这三种方式的示例：
### 一、值传递（Value Passing）
值传递是将实参的值传递给形参。在这种情况下，<strong style="color:red">函数内对形参的修改不会影响到实参。</strong>
示例:
```c++
#include <iostream>

void swap_value(int a, int b) {

    int temp = a;

    a = b;

    b = temp;

}

int main() {

    int x = 10;

    int y = 20;

    swap_value(x, y);

    std::cout << "x: " << x << ", y: " << y <<  std::endl; // 输出：x: 10, y: 20

    return 0;

}
```
### 二、引用传递（Reference Passing）
引用传递是将实参的引用传递给形参。在这种情况下，<b>函数内对形参的修改会影响到实参。</b>
```c++
#include <iostream>

void swap_reference(int &a, int &b) {

    int temp = a;

    a = b;

    b = temp;

}

int main() {

    int x = 10;

    int y = 20;

    swap_reference(x, y);

    std::cout << "x: " << x << ", y: " << y << std::endl; // 输出：x: 20, y: 10

    return 0;

}
```
### 三、指针传递（Pointer Passing）
指针传递是将实参的地址传递给形参。在这种情况下，函数内对形参的修改会影响到实参。
```c++
#include <iostream>

void swap_pointer(int *a, int *b) {

    int temp = *a;

    *a = *b;

    *b = temp;

}

int main() {

    int x = 10;

    int y = 20;

    swap_pointer(&x, &y);

    std::cout << "x: " << x << ", y: " << y << std::endl; // 输出：x: 20, y: 10

    return 0;

}
```

### 指针传递和引用传递的区别
特性	|引用传递 (Pass by Reference)|	指针传递 (Pass by Pointer)|
|-------|---------------|-------------------|
语法形式|	void func(int &x)|	void func(int *x)
调用方式	|func(a) （直接传递变量）|	func(&a) （传递变量地址）
是否可为空|	不能。引用必须初始化并始终指向一个有效对象。|	可以。指针可以为 nullptr。
是否可重定向	|不能。一旦初始化，就不能再指向其他变量。|	可以。可以随时改变指向的地址。
操作方式	|像操作普通变量一样（x = 10）。|	需要解引用操作符 *（*x = 10）。
安全性	|更安全。不存在空引用和野引用的问题（只要初始化正确）。|	相对不安全。可能遇到空指针、野指针，需手动检查。
内存地址	|语法上隐藏了地址的概念，更抽象。|	明确地操作内存地址，更底层。
“看起来”像|	对象的别名。它就是那个变量本身。	|对象的地址。你需要通过地址找到那个变量。
