# ```#include<mutex>```
`<mutex>` 头文件是 C++11 引入的，它包含了用于互斥锁（mutex）的类和函数。<b>互斥锁是一种同步机制，用于防止多个线程同时访问共享资源。</b>

互斥锁（Mutex）是一个用于控制对共享资源访问的同步原语。当一个线程需要访问共享资源时，它会尝试锁定互斥锁。如果互斥锁已经被其他线程锁定，请求线程将被阻塞，直到互斥锁被释放。

## 基本语法
在 C++ 中，<mutex> 头文件提供了以下主要类：

- **std::mutex**：基本的互斥锁。
- **std::recursive_mutex**：递归互斥锁，允许同一个线程多次锁定。
- **std::timed_mutex**：具有超时功能的互斥锁。
- **std::recursive_timed_mutex**：具有超时功能的递归互斥锁。

## 使用```std::mutex```
mutex的简单实例
```c++
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx; // 全局互斥锁
int shared_resource = 0;

void increment() {
    for (int i = 0; i < 10000; ++i) {
        mtx.lock(); // 锁定互斥锁
        ++shared_resource;
        mtx.unlock(); // 解锁互斥锁
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "Final value of shared_resource: " << shared_resource << std::endl;
    return 0;
}
```
输出结果为：
```c++
Final value of shared_resource: 20000
```

## 使用```std::recursive_mutex```
递归互斥锁允许同一个线程多次锁定同一个互斥锁。
```c++
#include <iostream>
#include <thread>
#include <mutex>

std::recursive_mutex rmtx; // 创建一个递归 mutex 对象
int shared_resource = 0; // 共享资源

// 递归函数
void recursive_increment(int count) {
    if (count <= 0) return;

    std::lock_guard<std::recursive_mutex> lock(rmtx); // 上锁，确保线程安全
    ++shared_resource;
    std::cout << "Incremented shared_resource to " << shared_resource << " (count = " << count << ")" << std::endl;

    // 递归调用
    recursive_increment(count - 1);
}

int main() {
    std::thread t1(recursive_increment, 3); // 线程 t1 执行 recursive_increment(3)
    std::thread t2(recursive_increment, 3); // 线程 t2 执行 recursive_increment(3)
    
    t1.join(); // 等待线程 t1 完成
    t2.join(); // 等待线程 t2 完成

    std::cout << "Final value of shared_resource: " << shared_resource << std::endl;

    return 0;
}
```
输出结果：
```c++
Incremented shared_resource to 1 (count = 3)
Incremented shared_resource to 2 (count = 2)
Incremented shared_resource to 3 (count = 1)
Incremented shared_resource to 4 (count = 0)
Incremented shared_resource to 5 (count = 3)
Incremented shared_resource to 6 (count = 2)
Incremented shared_resource to 7 (count = 1)
Incremented shared_resource to 8 (count = 0)
Final value of shared_resource: 8
```

## ```std::lock_guard```
一种自动管理 std::mutex 锁的封装器，使用 RAII 风格，确保在作用域结束时自动释放锁。
```c++
#include <mutex>

std::mutex mtx;

void function() {
    std::lock_guard<std::mutex> lock(mtx);
    // 访问共享资源
}
```

## ```std::unique_lock```
提供比 std::lock_guard 更灵活的锁管理，可以手动释放和重新获得锁，还支持定时锁定。
```c++
#include <mutex>
#include <chrono>

std::mutex mtx;

void function() {
    std::unique_lock<std::mutex> lock(mtx);
    // 访问共享资源
    
    // 可以手动释放锁
    lock.unlock();
    
    // 可以重新获得锁
    lock.lock();
    
    // 可以进行定时锁定
    if (lock.try_lock_for(std::chrono::seconds(1))) {
        // 成功获得锁
    }
}
```