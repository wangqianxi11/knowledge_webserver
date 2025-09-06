---
title: Log日志
updated: 2025-05-19T22:35:59
created: 2025-04-07T15:58:39
---

单例模式与阻塞队列，实现异步的日志系统。

**一、使用阻塞队列的核心动机**
1.  **实现“异步日志写入”机制**
    - **主线程不直接写日志文件，而是将日志消息放入阻塞队列**
    - **后台日志线程从队列中取出日志消息并写入磁盘**
    - **这样可以避免主线程频繁阻塞在 I/O 操作，提高系统响应速度和吞吐量**
2.  **线程安全的生产者-消费者模型**
    - **阻塞队列天然支持多个线程安全地并发访问**
    - **主线程作为“生产者”，日志线程作为“消费者”，无锁或轻锁实现，简洁稳定**
3.  **流量控制（背压）机制**
    - **当日志写入过快，超过队列容量时，主线程会阻塞等待消费者消费日志**
    - **避免日志过量产生导致内存暴涨，形成自动限流**

**二、与非阻塞队列的对比分析**
| **维度** | **阻塞队列** | **非阻塞队列** |
|----|----|----|
| **性能** | 高并发下可能稍慢，但更稳定 | 高速写入，但需手动处理边界情况 |
| **资源控制** | 有限队列，写满则自动等待（背压） | 无法自动限流，易OOM |
| **线程行为** | 写满时挂起写线程 | 写满时需立即丢弃或重试（复杂） |
| **实现复杂度** | 简洁，常用 STL queue + mutex/cond | 需细致管理 CAS、自旋、失败重试等 |
| **适用场景** | 日志量中等、容忍短暂阻塞 | 对日志实时性要求极高、超低延迟系统 |

## 用生产者-消费者模式维护一个队列，两个线程协作：  
- 队列长度 >10 时生产者阻塞并唤醒消费者  
- 队列长度 =0 时消费者阻塞并唤醒生产者  
请打印线程协作全过程。用C++实现
```c++
#include <iostream>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <chrono>

std::queue<int> queue;
std::mutex mtx;
std::condition_variable cv;
const int MAX_QUEUE_SIZE = 10;
bool done = false;

void producer() {
    int item = 0;
    while (!done) {
        std::this_thread::sleep_for(std::chrono::milliseconds(100));

        std::unique_lock<std::mutex> lock(mtx);
        
        // 队列长度 >10 时生产者阻塞并唤醒消费者
        if (queue.size() > MAX_QUEUE_SIZE) {
            std::cout << "队列已满(" << queue.size() << ">10)，生产者阻塞并唤醒消费者..." << std::endl;
            cv.notify_all();  // 唤醒消费者
            cv.wait(lock, [] { return queue.size() <= MAX_QUEUE_SIZE || done; });
            if (done) break;
        }

        queue.push(item);
        std::cout << "生产者生产了商品: " << item++ << "，队列大小: " << queue.size() << std::endl;

        // 如果队列不为空，唤醒消费者
        if (queue.size() == 1) {
            cv.notify_all();
        }
    }
}

void consumer() {
    while (!done) {
        std::this_thread::sleep_for(std::chrono::milliseconds(150));

        std::unique_lock<std::mutex> lock(mtx);
        
        // 队列长度 =0 时消费者阻塞并唤醒生产者
        if (queue.empty()) {
            std::cout << "队列为空(0)，消费者阻塞并唤醒生产者..." << std::endl;
            cv.notify_all();  // 唤醒生产者
            cv.wait(lock, [] { return !queue.empty() || done; });
            if (done) break;
        }

        int item = queue.front();
        queue.pop();
        std::cout << "消费者消费了商品: " << item << "，队列大小: " << queue.size() << std::endl;

        // 如果队列不再满，唤醒生产者
        if (queue.size() == MAX_QUEUE_SIZE - 1) {
            cv.notify_all();
        }
    }
}

int main() {
    std::thread producer_thread(producer);
    std::thread consumer_thread(consumer);

    std::this_thread::sleep_for(std::chrono::seconds(5));
    done = true;
    cv.notify_all();

    producer_thread.join();
    consumer_thread.join();

    return 0;
}
```