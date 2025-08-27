---
title: Threadpool类分析
updated: 2025-04-06T17:18:39
created: 2025-04-06T16:59:14
---

#  工作流程：
1、线程池创建时启动多少工作线程
2、工作线程等待任务或关闭信号
3、通过AddTask添加的任务会被工作线程取出执行
4、析构时通知所有线程停止工作

**结构体pool，封装了线程池所需的所有共享数据**
<table>
<colgroup>
<col style="width: 11%" />
<col style="width: 88%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p></th>
<th><p>struct Pool {</p>
<p>std::mutex mtx; // 互斥锁</p>
<p>std::condition_variable cond; // 条件变量，用于线程间的任务通知机制</p>
<p>bool isClosed; // 标志位，线程池是否关闭</p>
<p>std::queue&lt;std::function&lt;void()&gt;&gt; tasks; // 任务队列</p>
<p>};</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**构造函数实现了一个线程池：**
- **创建指定数量的工作线程**
- **每个线程执行一个无限循环：**
**1、检查任务队列**

**2、有任务取出执行**

**3、无任务则等待变量通知**

**4、线程池关闭则退出循环**
- **使用detach()式工作线程和主线程分离**

<table>
<colgroup>
<col style="width: 11%" />
<col style="width: 88%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p>
<p>7</p>
<p>8</p>
<p>9</p>
<p>10</p>
<p>11</p>
<p>12</p>
<p>13</p>
<p>14</p>
<p>15</p>
<p>16</p>
<p>17</p>
<p>18</p></th>
<th><p>explicit ThreadPool(size_t threadCount = 8): pool_(std::make_shared&lt;Pool&gt;()) {</p>
<p>for(size_t i = 0; i &lt; threadCount; i++) {</p>
<p>std::thread([pool = pool_] {</p>
<p>std::unique_lock&lt;std::mutex&gt; locker(pool-&gt;mtx); // 获取互斥锁</p>
<p>while(true) {</p>
<p>if(!pool-&gt;tasks.empty()) { // 如果任务队列非空</p>
<p>auto task = std::move(pool-&gt;tasks.front()); // 取出任务</p>
<p>pool-&gt;tasks.pop(); // 移除任务</p>
<p>locker.unlock(); // 释放锁，允许其他线程访问队列</p>
<p>task(); // 执行任务</p>
<p>locker.lock(); // 重新获取锁</p>
<p>}</p>
<p>else if(pool-&gt;isClosed) break; // 如果线程池关闭，退出循环</p>
<p>else pool-&gt;cond.wait(locker); // 否则，等待新任务</p>
<p>}</p>
<p>}).detach(); // 线程分离，主线程不会等待工作线程结束，如果主线程退出，工作线程仍会继续运行（造成资源泄露）</p>
<p>}</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
**提供默认构造和移动构造，移动构造允许线程池对象资源转移**
<table>
<colgroup>
<col style="width: 18%" />
<col style="width: 81%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p></th>
<th><p>ThreadPool() = default;</p>
<p>ThreadPool(ThreadPool&amp;&amp;) = default;</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**析构函数，**
<table>
<colgroup>
<col style="width: 11%" />
<col style="width: 88%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p>
<p>7</p>
<p>8</p>
<p>9</p></th>
<th><p>~ThreadPool() {</p>
<p>if(static_cast&lt;bool&gt;(pool_)) {</p>
<p>{</p>
<p>std::lock_guard&lt;std::mutex&gt; locker(pool_-&gt;mtx); //</p>
<p>pool_-&gt;isClosed = true; // 设置关闭标志</p>
<p>}</p>
<p>pool_-&gt;cond.notify_all(); // 通知所有等待的线程，完成当前任务后自行退出</p>
<p>}</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

**向任务队列添加新任务**
<table>
<colgroup>
<col style="width: 10%" />
<col style="width: 89%" />
</colgroup>
<thead>
<tr class="header">
<th><p>1</p>
<p>2</p>
<p>3</p>
<p>4</p>
<p>5</p>
<p>6</p>
<p>7</p>
<p>8</p></th>
<th><p>template&lt;class F&gt;</p>
<p>void AddTask(F&amp;&amp; task) {</p>
<p>{</p>
<p>std::lock_guard&lt;std::mutex&gt; locker(pool_-&gt;mtx); // 加锁，保护任务队列安全</p>
<p>pool_-&gt;tasks.emplace(std::forward&lt;F&gt;(task)); // 构造任务队列，保证完美转发，不多拷贝</p>
<p>}</p>
<p>pool_-&gt;cond.notify_one(); // 唤醒一个等待线程来处理</p>
<p>}</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>
