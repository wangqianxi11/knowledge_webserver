# ```condition_variable```
`condition_variable`是用于线程间同步的一种高级工具，它比使用低级同步原语（如互斥锁和条件变量）更加安全和方便。

condition_variable是一个类模板，用于在多线程环境中实现线程间的同步。它允许一个或多个线程等待某个条件变为真，而其他线程可以唤醒这些等待的线程。

## 语法
以下是`condition_variable`的基本语法：
```c++
#include <condition_variable>

void notify_one() {
    // 唤醒一个等待的线程
}

void notify_all() {
    // 唤醒所有等待的线程
}

template <class Mutex>
class condition_variable {
public:
    condition_variable();
    ~condition_variable();

    void wait(unique_lock<mutex>& lock);
    void wait_for(unique_lock<mutex>& lock, chrono::duration<Rep, Period> const& rel_time);
    void wait_until(unique_lock<mutex>& lock, chrono::time_point<Clock, Duration> const& abs_time);

    void notify_one() noexcept;
    void notify_all() noexcept;
};
```

## 实例
下面是一个使用`condition_variable`的简单示例，展示了生产者-消费者问题的基本实现。
```c++
#include <iostream>
#include <condition_variable>
#include <mutex>
#include <queue>
#include <thread>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> product;

void producer(int id) {
    for (int i = 0; i < 5; ++i) {
        std::unique_lock<std::mutex> lck(mtx);
        product.push(id * 100 + i);
        std::cout << "Producer " << id << " produced " << product.back() << std::endl;
        cv.notify_one();
        lck.unlock();
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

void consumer(int id) {
    while (true) {
        std::unique_lock<std::mutex> lck(mtx);
        cv.wait(lck, []{ return !product.empty(); });
        if (!product.empty()) {
            int prod = product.front();
            product.pop();
            std::cout << "Consumer " << id << " consumed " << prod << std::endl;
        }
        lck.unlock();
    }
}

int main() {
    std::thread producers[2];
    std::thread consumers[2];

    for (int i = 0; i < 2; ++i) {
        producers[i] = std::thread(producer, i + 1);
    }

    for (int i = 0; i < 2; ++i) {
        consumers[i] = std::thread(consumer, i + 1);
    }

    for (int i = 0; i < 2; ++i) {
        producers[i].join();
        consumers[i].join();
    }

    return 0;
}
```
这个程序的输出可能会有所不同，因为它依赖于线程调度。但是，你会看到生产者生产产品，然后消费者消费这些产品。输出将类似于：

Producer 1 produced 100
Producer 2 produced 200
Consumer 1 consumed 100
Producer 1 produced 101
Consumer 2 consumed 200
Producer 2 produced 201
Consumer 1 consumed 101