---
title: STL
updated: 2025-05-20T07:19:43
created: 2025-04-15T14:48:05
---

它是C++标准库的重要组成部分，不仅是一个可复用的组件库也是一个包含了数据结构与算法的软件架构，它拥有六大组件分别是：仿函数，算法，迭代器，空间配置器，容器，配接器

**STL容器本身不是线程安全的**。
对于vector，即使写方（生产者）是单线程写入，但是并发读的时候，由于潜在的内存重新申请和对象复制问题，会导致读方（消费者）的迭代器失效。此外，多个并发写也会失效。

**一、加锁**
mutex互斥锁，性能较差。
对于多读少写的场景可以使用读写锁（共享独占锁）

**二、固定大小**
避免动态扩容来做到lock-free

**关联容器的线程安全问题**
vector是顺序容器，map、unordered_map叫关联类容器

**1. 容器（Containers）**
封装各种数据结构，用于存储和组织数据。
**常用容器分类：**
| **序列式容器（有序、线性）** | **关联式容器（基于红黑树）** | **无序容器（基于哈希表）** |
|----|----|----|
| vector | set, map | unordered_set, unordered_map |
| list | multiset, multimap | unordered_multiset etc. |
| deque |  |  |
| array (C++11) |  |  |
| forward_list (C++11) |  |  |
**2. 算法（Algorithms）**
STL 提供了 **60+ 常用算法函数**，如查找、排序、复制、变换、组合、计数等。
**示例：**
```c++
std::sort(vec.begin(), vec.end());

std::find(vec.begin(), vec.end(), 42);

std::count_if(vec.begin(), vec.end(), [](int x){ return x > 0; });
```
**3. 迭代器（Iterators）**
迭代器是 STL 容器与算法之间的桥梁。每种容器都定义了自己的迭代器类型。
**常见类型：**
| **类型**               | **示例用途**               |
|------------------------|----------------------------|
| input_iterator         | 只读，如 istream           |
| output_iterator        | 只写，如 ostream           |
| forward_iterator       | 单向遍历 list              |
| bidirectional_iterator | 双向，如 list              |
| random_access_iterator | 随机访问，如 vector, array |

**4. 仿函数（Functors）**
本质是**重载了 operator() 的类**，也称为“函数对象”。可以将其作为自定义逻辑传给算法。
```c++

struct Greater {

bool operator()(int a, int b) const {

    return a > b;

    }

};

std::sort(vec.begin(), vec.end(), Greater());
```
**5. 适配器（Adaptors）**
适配器是对容器、函数或迭代器的功能扩展或封装。
**容器适配器：**
- stack（基于 deque）
- queue
- priority_queue
**迭代器适配器：**
- reverse_iterator
- insert_iterator（如 back_inserter）
**函数适配器：**
- std::bind, std::function, std::not1
  
**6. 内存分配器（Allocators）**
控制容器如何申请、管理和释放内存。默认是 std::allocator，也可以自定义。
```c++
std::vector<int, MyCustomAllocator<int>> v;
```