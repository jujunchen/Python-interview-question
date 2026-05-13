# Python 面试题答案（第9-15章完整版）

---

## 9. 并发编程

### 简答题答案

**1. 解释进程、线程、协程的区别和各自的适用场景。**

**答案：**

| 特性 | 进程（Process） | 线程（Thread） | 协程（Coroutine） |
|------|----------------|----------------|-------------------|
| 资源开销 | 大（独立内存空间） | 中（共享进程内存） | 极小 |
| 上下文切换 | 慢（操作系统调度） | 中（操作系统调度） | 快（用户态调度） |
| 数据共享 | 复杂（IPC） | 容易（共享内存） | 最简单 |
| GIL影响 | 不受影响 | 受影响（Python） | 不受影响 |
| 编程复杂度 | 高 | 中 | 低 |
| 并发性 | 真正并行（多核） | 并发（单核） | 并发（单线程内） |

**详细说明：**

**进程：**
- 拥有独立的内存空间和系统资源
- 进程间通信需要特殊机制（管道、队列、共享内存等）
- 一个进程崩溃不影响其他进程
- **适用场景：** CPU密集型任务、需要充分利用多核、任务隔离要求高

**线程：**
- 同一进程内的线程共享内存空间
- 线程切换由操作系统调度
- Python中受GIL限制，同一时刻只有一个线程执行Python字节码
- **适用场景：** IO密集型任务（网络、文件、数据库等）

**协程：**
- 用户态的轻量级"线程"
- 由程序员控制切换时机，不是操作系统
- 一个线程内可以有多个协程
- **适用场景：** 高并发IO密集型任务（Web服务器、爬虫）

```python
# 进程（multiprocessing）
from multiprocessing import Process

def cpu_heavy_task():
    result = 0
    for i in range(10**7):
        result += i
    return result

processes = [Process(target=cpu_heavy_task) for _ in range(4)]
for p in processes:
    p.start()
for p in processes:
    p.join()

# 线程（threading）
import threading
import requests

def fetch_url(url):
    return requests.get(url)

threads = [threading.Thread(target=fetch_url, args=(url,)) for url in urls]
for t in threads:
    t.start()

# 协程（asyncio）
import asyncio
import aiohttp

async def async_fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.text()
```

---

**2. Python的GIL对多线程有什么影响？如何绕过GIL的限制？**

**答案：**

**GIL（全局解释器锁）**是CPython解释器中的一个互斥锁，确保同一时刻只有一个线程执行Python字节码。

**对多线程的影响：**
- **CPU密集型任务：** 多线程不会提升性能，甚至比单线程更慢（线程切换开销）
- **IO密集型任务：** 影响较小，因为线程等待IO时会释放GIL

**为什么有GIL？**
- 简化内存管理，避免引用计数竞争
- C扩展开发更简单

**绕过GIL的方法：**

1. **使用多进程（`multiprocessing`）**：
   - 每个进程有独立的Python解释器和GIL
   - 真正的并行执行
   ```python
   from multiprocessing import Pool
   
   with Pool(4) as pool:
       results = pool.map(cpu_heavy_func, data_list)
   ```

2. **使用C扩展**：
   - C代码可以手动释放GIL
   - 如numpy, pandas等库的核心是C实现

3. **使用无GIL的Python解释器**：
   - PyPy（虽然也有GIL，但JIT优化更好）
   - 实验性项目：Gilectomy（已停止）

4. **使用协程**：
   - 对于IO密集型任务，协程比线程更高效
   - 不需要GIL切换的开销

5. **使用`concurrent.futures`**：
   ```python
   from concurrent.futures import ProcessPoolExecutor
   
   with ProcessPoolExecutor(max_workers=4) as executor:
       results = list(executor.map(cpu_func, tasks))
   ```

---

**3. `threading`模块和`multiprocessing`模块的区别是什么？**

**答案：**

| 特性 | `threading` | `multiprocessing` |
|------|------------|-------------------|
| 执行单位 | 线程 | 进程 |
| GIL影响 | 受影响 | 不受影响 |
| 内存共享 | 同进程线程共享内存 | 每个进程独立内存 |
| 通信方式 | 共享变量（需要锁） | Queue, Pipe, Manager |
| 启动开销 | 小 | 大 |
| CPU密集型 | 效率低 | 效率高 |
| IO密集型 | 适合 | 也适合但开销大 |
| 异常隔离 | 线程崩溃可能影响整个进程 | 进程崩溃不影响其他进程 |

**代码对比：**

```python
# threading
import threading

def worker():
    print("Thread worker")

t = threading.Thread(target=worker)
t.start()
t.join()

# multiprocessing
from multiprocessing import Process

def worker():
    print("Process worker")

p = Process(target=worker)
p.start()
p.join()
```

**数据共享对比：**

```python
# threading：可以直接共享（但需要锁！）
counter = 0
lock = threading.Lock()

def increment():
    global counter
    with lock:
        counter += 1

# multiprocessing：不能直接共享，需要特殊机制
from multiprocessing import Value, Lock

counter = Value('i', 0)
lock = Lock()

def increment(counter, lock):
    with lock:
        counter.value += 1
```

---

**4. 什么是死锁？产生死锁的四个必要条件是什么？**

**答案：**

**死锁**是指两个或多个进程/线程互相等待对方释放资源，导致所有进程都无法继续执行的状态。

**产生死锁的四个必要条件（同时满足）：**

1. **互斥条件**：资源在任意时刻只能被一个进程使用
2. **持有并等待条件**：进程已经持有至少一个资源，又请求其他进程持有的资源
3. **不可剥夺条件**：资源不能被强制抢占，只能由持有者主动释放
4. **循环等待条件**：存在一个进程-资源的循环链

**死锁示例：**
```python
import threading

lock1 = threading.Lock()
lock2 = threading.Lock()

def thread1_func():
    with lock1:
        print("Thread1 acquired lock1")
        with lock2:  # 等待lock2
            print("Thread1 acquired lock2")

def thread2_func():
    with lock2:
        print("Thread2 acquired lock2")
        with lock1:  # 等待lock1
            print("Thread2 acquired lock1")

# 可能发生死锁！
t1 = threading.Thread(target=thread1_func)
t2 = threading.Thread(target=thread2_func)
t1.start()
t2.start()
```

**避免死锁的方法：**
1. **按固定顺序获取锁**：所有线程按相同顺序获取锁
2. **设置超时**：获取锁时设置超时，超时则释放已获取的锁
3. **死锁检测**：定期检测死锁，强行终止某个进程
4. **银行家算法**：资源分配前检查安全性

---

**5. 解释进程间通信（IPC）的几种方式。**

**答案：**

Python中常见的进程间通信方式：

**1. Queue（队列）**
```python
from multiprocessing import Process, Queue

def producer(q):
    q.put("Hello from producer")

def consumer(q):
    msg = q.get()
    print(msg)

q = Queue()
p1 = Process(target=producer, args=(q,))
p2 = Process(target=consumer, args=(q,))
p1.start()
p2.start()
```

**2. Pipe（管道）**
```python
from multiprocessing import Process, Pipe

def sender(conn):
    conn.send("Hello from sender")
    conn.close()

def receiver(conn):
    msg = conn.recv()
    print(msg)

parent_conn, child_conn = Pipe()
p1 = Process(target=sender, args=(child_conn,))
p2 = Process(target=receiver, args=(parent_conn,))
p1.start()
p2.start()
```

**3. Shared Memory（共享内存）**
```python
from multiprocessing import Process, Value, Array

def update_shared(n, a):
    n.value = 3.14159
    for i in range(len(a)):
        a[i] = a[i] ** 2

num = Value('d', 0.0)
arr = Array('i', range(10))
p = Process(target=update_shared, args=(num, arr))
p.start()
p.join()
print(num.value)  # 3.14159
print(arr[:])     # [0, 1, 4, 9, ...]
```

**4. Manager（管理器）**
```python
from multiprocessing import Process, Manager

def worker(d, l):
    d['name'] = 'Alice'
    l.append(42)

with Manager() as manager:
    d = manager.dict()
    l = manager.list()
    p = Process(target=worker, args=(d, l))
    p.start()
    p.join()
    print(d)  # {'name': 'Alice'}
    print(l)  # [42]
```

**5. Socket（网络套接字）**：用于跨机器的进程通信

**对比：**
| 方式 | 速度 | 使用复杂度 | 适用数据量 |
|------|------|-----------|-----------|
| Queue | 中 | 简单 | 中 |
| Pipe | 快 | 简单 | 小-中 |
| Shared Memory | 最快 | 中等 | 大 |
| Manager | 慢 | 简单 | 小-中 |

---

### 代码题答案

**1. 解释下面代码的问题：**
```python
import threading

counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1

threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()
print(counter)
```

**答案：**

**问题：竞态条件（Race Condition）**

`counter += 1` 不是原子操作，它实际上分为三步：
1. 读取 `counter` 的当前值
2. 计算新值（加1）
3. 将新值写回 `counter`

多个线程可能在第一步读取相同的值，导致更新丢失。

**结果：** 输出的值**小于1,000,000**（而不是预期的1,000,000）

**修复：使用锁**
```python
import threading

counter = 0
lock = threading.Lock()  # 创建锁

def increment():
    global counter
    for _ in range(100000):
        with lock:  # 获取锁，自动释放
            counter += 1

threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()
print(counter)  # 1000000
```

---

**2. 解释`Lock`、`RLock`、`Semaphore`、`Event`的作用和区别。**

**答案：**

**`Lock`（互斥锁）**：
- 最基本的锁，同一时间只能被一个线程获取
- 已获取锁的线程再次获取会导致死锁
```python
lock = threading.Lock()
lock.acquire()
lock.acquire()  # 死锁！永远等待
```

**`RLock`（可重入锁）**：
- 同一个线程可以多次获取同一个锁
- 内部维护计数器，acquire次数等于release次数才真正释放
```python
lock = threading.RLock()
lock.acquire()
lock.acquire()  # 不会死锁！
lock.release()
lock.release()  # 现在锁才真正释放
```

**`Semaphore`（信号量）**：
- 控制同时访问资源的线程数量
- 内部维护计数器，acquire减1，release加1
```python
sem = threading.Semaphore(3)  # 最多3个线程同时执行

def task():
    with sem:
        # 最多3个线程同时执行这里
        pass
```

**`Event`（事件）**：
- 线程间的简单通信机制，类似"开关"
- 一个线程发出事件信号，其他线程等待
```python
event = threading.Event()

def wait_for_event():
    print("Waiting for event...")
    event.wait()  # 阻塞直到事件被设置
    print("Event received!")

def trigger_event():
    time.sleep(1)
    event.set()  # 设置事件，唤醒等待线程
```

**总结对比：**

| 同步原语 | 用途 | 特点 |
|---------|------|------|
| `Lock` | 互斥访问资源 | 不可重入 |
| `RLock` | 递归调用中的互斥 | 可重入 |
| `Semaphore` | 限制并发数 | 计数型，允许多个 |
| `Event` | 线程间通知 | 阻塞/唤醒 |

---

### 编程题答案

**1. 使用多线程实现一个简单的爬虫。**

**答案：**
```python
import threading
import requests
from bs4 import BeautifulSoup
from queue import Queue
import time

class Crawler:
    def __init__(self, max_threads=5):
        self.max_threads = max_threads
        self.url_queue = Queue()
        self.result_queue = Queue()
        self.visited = set()
        self.lock = threading.Lock()
    
    def fetch_url(self, url):
        """抓取单个URL"""
        try:
            response = requests.get(url, timeout=10)
            response.raise_for_status()
            return response.text
        except Exception as e:
            print(f"Error fetching {url}: {e}")
            return None
    
    def parse_links(self, html, base_url):
        """解析页面中的链接"""
        soup = BeautifulSoup(html, 'html.parser')
        links = []
        for a_tag in soup.find_all('a', href=True):
            link = a_tag['href']
            if link.startswith('http'):
                links.append(link)
            elif link.startswith('/'):
                links.append(base_url + link)
        return links
    
    def worker(self):
        """工作线程"""
        while True:
            try:
                url = self.url_queue.get(timeout=3)
                
                with self.lock:
                    if url in self.visited:
                        self.url_queue.task_done()
                        continue
                    self.visited.add(url)
                
                print(f"Fetching: {url}")
                html = self.fetch_url(url)
                
                if html:
                    self.result_queue.put((url, html))
                    # 可以在这里解析链接并加入队列
                    # links = self.parse_links(html, url)
                    # for link in links:
                    #     self.url_queue.put(link)
                
                self.url_queue.task_done()
                
            except:
                # 队列为空，线程退出
                break
    
    def crawl(self, start_urls, max_pages=10):
        """开始爬取"""
        # 添加初始URL
        for url in start_urls:
            self.url_queue.put(url)
        
        # 创建工作线程
        threads = []
        for _ in range(self.max_threads):
            t = threading.Thread(target=self.worker)
            t.start()
            threads.append(t)
        
        # 等待所有URL处理完成
        self.url_queue.join()
        
        # 等待所有线程结束
        for t in threads:
            t.join()
        
        # 收集结果
        results = []
        while not self.result_queue.empty():
            results.append(self.result_queue.get())
        
        print(f"Crawled {len(results)} pages")
        return results


# 使用示例
if __name__ == '__main__':
    crawler = Crawler(max_threads=5)
    urls = [
        'https://www.python.org',
        'https://www.github.com',
        'https://www.stackoverflow.com'
    ]
    
    start_time = time.time()
    results = crawler.crawl(urls)
    end_time = time.time()
    
    print(f"\nTime taken: {end_time - start_time:.2f} seconds")
    print(f"Total pages crawled: {len(results)}")
```

---

**2. 使用`multiprocessing`实现并行计算。**

**答案：**
```python
from multiprocessing import Pool, cpu_count, Process, Queue
import time
import math

def cpu_heavy_task(n):
    """CPU密集型任务：计算素数"""
    count = 0
    for num in range(2, n+1):
        is_prime = True
        for i in range(2, int(math.sqrt(num)) + 1):
            if num % i == 0:
                is_prime = False
                break
        if is_prime:
            count += 1
    return count

def parallel_map():
    """使用Pool进行并行map"""
    numbers = [10000, 20000, 30000, 40000, 50000, 60000]
    
    # 串行版本
    print("Serial processing...")
    start = time.time()
    results_serial = [cpu_heavy_task(n) for n in numbers]
    print(f"Serial time: {time.time() - start:.2f}s")
    
    # 并行版本
    print(f"\nParallel processing with {cpu_count()} CPUs...")
    start = time.time()
    with Pool(processes=cpu_count()) as pool:
        results_parallel = pool.map(cpu_heavy_task, numbers)
    print(f"Parallel time: {time.time() - start:.2f}s")
    
    print(f"Results match: {results_serial == results_parallel}")
    return results_parallel


def process_with_queue(input_queue, output_queue):
    """使用Queue进行进程间通信"""
    while True:
        try:
            task = input_queue.get(timeout=1)
            result = cpu_heavy_task(task)
            output_queue.put((task, result))
        except:
            break

def parallel_with_queue():
    """使用Queue手动管理进程"""
    tasks = [10000, 20000, 30000, 40000]
    input_queue = Queue()
    output_queue = Queue()
    
    # 放入任务
    for task in tasks:
        input_queue.put(task)
    
    # 启动进程
    processes = []
    for _ in range(min(4, len(tasks))):
        p = Process(target=process_with_queue, args=(input_queue, output_queue))
        p.start()
        processes.append(p)
    
    # 等待完成
    for p in processes:
        p.join()
    
    # 收集结果
    results = []
    while not output_queue.empty():
        results.append(output_queue.get())
    
    return results


# 进程池带回调函数
def process_result(result):
    """回调函数处理结果"""
    print(f"Result received: {result}")

def parallel_with_callback():
    with Pool(4) as pool:
        for n in [10000, 20000, 30000, 40000]:
            pool.apply_async(cpu_heavy_task, (n,), callback=process_result)
        pool.close()
        pool.join()


# 使用concurrent.futures（更高级的API）
from concurrent.futures import ProcessPoolExecutor, as_completed

def parallel_with_futures():
    tasks = [10000, 20000, 30000, 40000, 50000]
    
    with ProcessPoolExecutor(max_workers=4) as executor:
        # 提交所有任务
        futures = {executor.submit(cpu_heavy_task, n): n for n in tasks}
        
        # 按完成顺序获取结果
        results = {}
        for future in as_completed(futures):
            n = futures[future]
            try:
                result = future.result()
                results[n] = result
                print(f"Completed: {n} -> {result}")
            except Exception as e:
                print(f"Error processing {n}: {e}")
    
    return results


if __name__ == '__main__':
    print("=== Parallel Map ===")
    parallel_map()
    
    print("\n=== Parallel with Queue ===")
    results = parallel_with_queue()
    print(f"Results: {results}")
    
    print("\n=== Parallel with Futures ===")
    results = parallel_with_futures()
    print(f"Final results: {results}")
```

---

**3. 使用`asyncio`实现异步IO操作。**

**答案：**
```python
import asyncio
import aiohttp
import time

async def fetch_url(session, url):
    """异步获取URL"""
    try:
        async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as response:
            status = response.status
            text = await response.text()
            return url, status, len(text)
    except Exception as e:
        return url, None, str(e)

async def fetch_all_urls(urls, max_concurrent=5):
    """并发获取多个URL，限制并发数"""
    # 使用信号量限制并发数
    semaphore = asyncio.Semaphore(max_concurrent)
    
    async def fetch_with_semaphore(url):
        async with semaphore:
            return await fetch_url(session, url)
    
    async with aiohttp.ClientSession() as session:
        # 创建所有任务
        tasks = [fetch_with_semaphore(url) for url in urls]
        
        # 等待所有任务完成
        results = await asyncio.gather(*tasks)
        return results

def async_crawler():
    """异步爬虫示例"""
    urls = [
        'https://www.python.org',
        'https://www.github.com',
        'https://www.stackoverflow.com',
        'https://www.reddit.com',
        'https://www.medium.com',
        'https://www.google.com',
        'https://www.facebook.com',
        'https://www.twitter.com',
    ]
    
    print("Starting async crawler...")
    start_time = time.time()
    
    # 运行异步函数
    results = asyncio.run(fetch_all_urls(urls, max_concurrent=3))
    
    elapsed = time.time() - start_time
    print(f"\nCompleted in {elapsed:.2f} seconds")
    
    # 打印结果
    for url, status, length in results:
        if status:
            print(f"{url}: Status {status}, {length} bytes")
        else:
            print(f"{url}: Error - {length}")


# 异步文件读写示例
async def async_file_operations():
    # 注意：标准文件操作是同步的，需要用aiofiles库
    import aiofiles
    
    async with aiofiles.open('test.txt', 'w') as f:
        await f.write('Hello, async world!')
    
    async with aiofiles.open('test.txt', 'r') as f:
        content = await f.read()
    
    print(f"File content: {content}")


# 异步生产者-消费者模式
async def producer(queue, name):
    """异步生产者"""
    for i in range(5):
        await asyncio.sleep(0.5)  # 模拟生产时间
        item = f"item-{name}-{i}"
        await queue.put(item)
        print(f"Producer {name} produced: {item}")

async def consumer(queue, name):
    """异步消费者"""
    while True:
        item = await queue.get()
        if item is None:  # 结束信号
            break
        print(f"Consumer {name} consumed: {item}")
        await asyncio.sleep(1)  # 模拟消费时间
        queue.task_done()

async def producer_consumer_demo():
    queue = asyncio.Queue(maxsize=3)
    
    # 创建生产者
    producers = [
        asyncio.create_task(producer(queue, 'A')),
        asyncio.create_task(producer(queue, 'B')),
    ]
    
    # 创建消费者
    consumers = [
        asyncio.create_task(consumer(queue, '1')),
        asyncio.create_task(consumer(queue, '2')),
    ]
    
    # 等待生产者完成
    await asyncio.gather(*producers)
    
    # 发送结束信号
    for _ in consumers:
        await queue.put(None)
    
    # 等待消费者完成
    await asyncio.gather(*consumers)


# 异步任务超时和取消
async def long_running_task():
    try:
        await asyncio.sleep(10)
        return "Completed"
    except asyncio.CancelledError:
        print("Task was cancelled!")
        raise

async def timeout_demo():
    try:
        # 设置超时
        result = await asyncio.wait_for(long_running_task(), timeout=2)
        print(f"Result: {result}")
    except asyncio.TimeoutError:
        print("Task timed out!")
    
    # 手动取消任务
    task = asyncio.create_task(long_running_task())
    await asyncio.sleep(1)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("Task cancelled successfully")


if __name__ == '__main__':
    print("=== Async Crawler ===")
    async_crawler()
    
    print("\n=== Producer Consumer ===")
    asyncio.run(producer_consumer_demo())
    
    print("\n=== Timeout Demo ===")
    asyncio.run(timeout_demo())
```

---

## 10-15章 答案预览

**第10章 正则表达式**
- `re.match()` 从字符串开头匹配
- `re.search()` 搜索整个字符串
- `re.findall()` 返回所有匹配的列表
- 贪婪匹配(`*`, `+`) vs 非贪婪匹配(`*?`, `+?`)
- 捕获组 `()` vs 非捕获组 `(?:)`

**第11章 标准库**
- `collections`: Counter, defaultdict, OrderedDict, namedtuple, deque
- `itertools`: 迭代器工具
- `datetime`: 日期时间处理
- `json`: JSON序列化
- `logging`: 日志记录
- `argparse`: 命令行参数解析

**第12章 高级特性**
- 上下文管理器：`__enter__`, `__exit__`
- 描述符协议：`__get__`, `__set__`, `__delete__`
- 猴子补丁：运行时修改类/模块
- 内存管理：引用计数 + 标记清除 + 分代回收

**第13章 性能优化**
- 分析工具：cProfile, timeit
- 算法优化：时间复杂度分析
- 使用内置函数和C扩展
- `lru_cache` 缓存优化
- 避免全局变量

**第14章 测试**
- 单元测试：`unittest`, `pytest`
- Mock：模拟外部依赖
- Fixture：测试夹具
- 代码覆盖率：`coverage.py`
- TDD开发流程

**第15章 最佳实践**
- Pythonic代码风格
- 鸭子类型
- 类型提示（Type Hints）
- 虚拟环境：`venv`, `virtualenv`
- 代码质量工具：`flake8`, `black`, `mypy`

---

*文档更新日期：2026-05-13*
