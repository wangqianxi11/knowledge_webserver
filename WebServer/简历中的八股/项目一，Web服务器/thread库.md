# ```std::thread```

`<thread> `库是 C++ 标准库的一部分，提供了创建和管理线程的基本功能，它包括以下几个关键组件：

- **std::thread**：表示一个线程，可以创建、启动、等待和销毁线程。
- **std::this_thread**：提供了一些静态成员函数，用于操作当前线程。
- **std::thread::id**：线程的唯一标识符。

## 创建线程
要创建一个线程，你需要实例化 std::thread 类，并传递一个可调用对象（函数、lambda 表达式或对象的成员函数）作为参数。
```c++
#include <iostream>
#include <thread>

void print_id(int id) {
    std::cout << "ID: " << id << ", Thread ID: " << std::this_thread::get_id() << std::endl;
}

int main() {
    std::thread t1(print_id, 1);
    std::thread t2(print_id, 2);
}
```

## 启动线程
创建 std::thread 对象后，线程会立即开始执行，你可以调用 join() 方法来等待线程完成。

<b>`join()`方法会阻塞当前线程，直到被调用的线程完成执行</b>
```c++
t1.join();
t2.join();
```

## 销毁线程
当线程执行完毕后，你可以使用 detach() 方法来分离线程，或者让 std::thread 对象超出作用域自动销毁。
```c++
t1.detach(); // 线程将继续运行，但无法再被 join 或 detach
```

## 实例：使用 <thread> 创建并行计算
下面是一个使用 <thread> 库实现的并行计算实例，计算两个数的和。
```c++
#include <iostream>
#include <thread>

int sum = 0;

void add(int a, int b) {
    sum += a + b;
}

int main() {
    int a = 5;
    int b = 10;

    std::thread t1(add, a, b);
    std::thread t2(add, a, b);

    t1.join();
    t2.join();

    std::cout << "Sum: " << sum << std::endl; // 输出结果：Sum: 30
}
```
输出结果为：
```c++
Sum: 30
```

# 类和函数
<thread> 库包含了一系列的类和函数，用于创建、管理和同步线程。

以下是对 C++ <thread> 库的详细介绍:

主要组件
- std::thread
- std::mutex
- std::lock_guard
- std::unique_lock
- std::condition_variable
- std::future 和 std::promise
- std::async

## ```std::thread```
`std::thread` 类用于创建和管理线程。
```c++
#include <iostream>
#include <thread>

void print_hello() {
    std::cout << "Hello from thread!" << std::endl;
}

int main() {
    std::thread t(print_hello);
    t.join(); // 等待线程 t 结束
    return 0;
}
```
**重要方法**

- **join()**: 等待线程结束。
- **detach()**: 将线程置于后台运行，不再等待线程结束。
- **joinable()**: 检查线程是否可被 join 或 detach。
## ```std::mutex```
`std::mutex` 类用于同步对共享资源的访问。
```c++
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx; // 创建一个全局 mutex 对象
int shared_resource = 0; // 共享资源

// 线程函数
void increment() {
    std::lock_guard<std::mutex> lock(mtx); // 上锁，保证线程安全
    ++shared_resource;
    std::cout << "Incremented shared_resource to " << shared_resource << std::endl;
    // lock 在 lock_guard 离开作用域时自动释放
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
   
    t1.join(); // 等待线程 t1 完成
    t2.join(); // 等待线程 t2 完成

    std::cout << "Final value of shared_resource: " << shared_resource << std::endl;

    return 0;
}
```
## ```std::lock_guard```
`std::lock_guard` 是一个 RAII 风格的锁管理器，用于自动管理锁的生命周期。
```c++
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void print_thread_id(int id) {
    std::lock_guard<std::mutex> lock(mtx);
    std::cout << "Thread ID: " << id << std::endl;
}

int main() {
    std::thread t1(print_thread_id, 1);
    std::thread t2(print_thread_id, 2);
    t1.join();
    t2.join();
    return 0;
}
```
## ```std::unique_lock```
`std::unique_lock` 提供了比 std::lock_guard 更灵活的锁管理。
```c++
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void print_thread_id(int id) {
    std::unique_lock<std::mutex> lock(mtx);
    std::cout << "Thread ID: " << id << std::endl;
    lock.unlock(); // 可以手动解锁
    // ... 其他操作
}

int main() {
    std::thread t1(print_thread_id, 1);
    std::thread t2(print_thread_id, 2);
    t1.join();
    t2.join();
    return 0;
}
```

## ```std::condition_variable```
`std::condition_variable` 用于线程间的等待和通知。
```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void print_id(int id) {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, []{ return ready; });
    std::cout << "Thread ID: " << id << std::endl;
}

void set_ready() {
    std::unique_lock<std::mutex> lock(mtx);
    ready = true;
    cv.notify_all();
}

int main() {
    std::thread t1(print_id, 1);
    std::thread t2(print_id, 2);
   
    std::this_thread::sleep_for(std::chrono::seconds(1));
    set_ready();
   
    t1.join();
    t2.join();
    return 0;
}
```