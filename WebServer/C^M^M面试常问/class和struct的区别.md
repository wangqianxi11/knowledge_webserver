---
title: class和struct的区别
updated: 2025-04-15T15:24:08
created: 2025-04-15T11:13:09
---

默认继承权限不同：class默认继承的是private继承，struct默认是public继承。
Class还可用于定义模板参数，但是关键字struct不能同于定义模板参数，C++保留struct关键字，原因是保证与C语言的向下兼容性，为了保证百分百的与C语言中的struct向下兼容，，C++把最基本的对象单元规定为class而不是struct，就是为了避免各种兼容性的限制。

C++ 中的 struct 和 class 基本是通用的，唯有几个细节不同：
- class 中类中的成员默认都是 private 属性的。
- 而在 struct 中结构体中的成员默认都是 public 属性的。
- class 继承默认是 private 继承，而 struct 继承默认是 public 继承。
- class 可以用于定义模板参数，struct 不能用于定义模板参数。
