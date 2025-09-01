---
title: this指针
updated: 2025-05-10T10:22:05
created: 2025-05-10T10:13:29
---

### this指针
是一个特殊的指针，指向当前对象的实例
每一个对象都通过this指针访问自己的地址

是一个**隐藏**的指针，可以在类的成员函数中使用，可以用来指向调用对象

`友元函数没有this指针，友元函数不是类的成员函数`

静态成员函数没有this指针

示例：
```c++
#include <iostream>

class MyClass {

private:

int value;

public:

void setValue(int value) {

this->value = value;

}

void printValue() {

std::cout << "Value: " << this->value << std::endl;

}

};

int main() {

MyClass obj;

obj.setValue(42);

obj.printValue();

return 0;

}
```
