---
title: Threadpool类分析
updated: 2025-04-06T17:18:39
created: 2025-04-06T16:59:14
---

# 工作流程：

1、线程池创建时启动多少工作线程<br>
2、工作线程等待任务或关闭信号<br>
3、通过 AddTask 添加的任务会被工作线程取出执行<br>
4、析构时通知所有线程停止工作<br>

### 结构体 pool，封装了线程池所需的所有共享数据

```c++
struct Pool {

    std::mutex mtx; // 互斥锁

    std::condition_variable cond; // 条件变量，用于线程间的任务通知机制

    bool isClosed; // 标志位，线程池是否关闭

    std::queue<std::function<void()>> tasks; // 任务队列

};
```

### 构造函数实现了一个线程池：

- **创建指定数量的工作线程**
- **每个线程执行一个无限循环：**

**1、检查任务队列**

**2、有任务取出执行**

**3、无任务则等待变量通知**

**4、线程池关闭则退出循环**

- **使用 detach()式工作线程和主线程分离**

```c++
explicit ThreadPool(size_t threadCount = 8): pool_(std::make_shared<Pool>()) {

    for(size_t i = 0; i < threadCount; i++) {

    std::thread([pool = pool_] {

    std::unique_lock<std::mutex> locker(pool->mtx); // 获取互斥锁

    while(true) {

        if(!pool->tasks.empty()) { // 如果任务队列非空

            auto task = std::move(pool->tasks.front()); // 取出任务

            pool->tasks.pop(); // 移除任务

            locker.unlock(); // 释放锁，允许其他线程访问队列

            task(); // 执行任务

            locker.lock(); // 重新获取锁

        }

    else if(pool->isClosed) break; // 如果线程池关闭，退出循环

    else pool->cond.wait(locker); // 否则，等待新任务

        }

    }).detach(); // 线程分离，主线程不会等待工作线程结束，如果主线程退出，工作线程仍会继续运行（造成资源泄露）

    }
}
```

### 提供默认构造和移动构造，移动构造允许线程池对象资源转移

```c++
ThreadPool() = default;

ThreadPool(ThreadPool&&) = default;
```

**析构函数，**

```c++
~ThreadPool() {

if(static_cast<bool>(pool_))
{

{

    std::lock_guard<std::mutex> locker(pool_->mtx); //

    pool_->isClosed = true; // 设置关闭标志

}

    pool_->cond.notify_all(); // 通知所有等待的线程，完成当前任务后自行退出

}

}
```

### 向任务队列添加新任务

```c++
template<class F>

void AddTask(F&& task)
{

{

    std::lock_guard<std::mutex> locker(pool_->mtx); // 加锁，保护任务队列安全

    pool_->tasks.emplace(std::forward<F>(task)); // 构造任务队列，保证完美转发，不多拷贝

}

    pool_->cond.notify_one(); // 唤醒一个等待线程来处理

}
```
