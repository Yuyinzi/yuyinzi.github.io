---
title: Python线程/进程
date: 2019-04-03 17:02:50
tags: Python
categories:
    - Python
---

[TOC]
## `GIL`
`GIL`:`global interpreter lock`全局锁
限制：使得`python`中一个线程对应于`C`语言中的一个线程。同一时刻只有一个线程运行在一个`CPU`上执行字节码，无法将多线程运行到多`CPU`上。
释放：根据字节码执行行数或者时间片来释放锁，或者遇到`io`操作时会主动释放。
显示字节码：`from dis import dis`

## 区别
- 耗`CPU`操作(例如运算)采用多进程更加具有优势(`GIL`使得多线程没有办法在不同的`CPU`上运行)
- 耗`IO`操作采用多线程更具有优势(当碰见`IO`操作时，会主动释放`GIL`)

<!-- more-->

## `ThreadLocal`
使用场景：多线程环境下，如果使用全局变量，需要加锁。但是如果每个线程都需要拥有自己的私有数据时，即这个数据对其他线程不可见。可以使用`ThreadLocal`变量，它本身是一个全局变量，但是每个线程却可以利用它来保存属于自己的私有数据。
```python
import threading

global_data = threading.local()

def show():
    print threading.current_thread().getName(), global_data.num

def thread_cal():
    global_data.num = 0
    for _ in xrange(1000):
        global_data.num += 1
    show()

threads = []
for i in range(10):
    threads.append(threading.Thread(target=thread_cal))
    threads[i].start()
```

## `join()`和`setDaemon()`
`join`：主线程启动若干个子线程后，可以继续执行主线程的代码，也可以等待所有子线程执行完毕后继续执行主线程。如果子线程调用了`join`方法，则表明**主线程必须等子线程执行完了才可以往下继续执行**。因此，如果需要所有的子线程都能再主线程结束前被执行完毕，则必须为每一个子线程都注册`join`。
`setDaemon()`：设置此线程是否被主线程守护回收。需要在`start`方法前调用，设为`True`时，当主线程结束之后，会回收此子线程。默认是`False`，即主线程结束时不会回收子线程。

## 创建线程
### `Threading`模块
```python
# -*- coding: utf-8 -*-
import time
import threading 

def get_detail_html(url):
    print("get detail html started")
    time.sleep(2)
    print("get detail html end")


def get_detail_url(url):
    print("get detail url started")
    time.sleep(2)
    print("get detail url end")


if __name__ == "__main__":
    thread1 = threading.Thread(target=get_detail_html, args=("",))
    thread2 = threading.Thread(target=get_detail_url, args=("",))
    start_time = time.time()
    thread1.start()
    thread2.start()
    print("last time: {}".format(time.time() - start_time))
```
### 派生`Thread`的子类，并创建实例
> 需要重载`run`方法
```python
class GetDetailHtml(threading.Thread):
    def __init__(self, name):
        super().__init__(name=name)

    def run(self):
        print("get detail html started")
        time.sleep(2)
        print("get detail html end")


class GetDetailUrl(threading.Thread):
    def __init__(self, name):
        super().__init__(name=name)

    def run(self):
        print("get detail url started")
        time.sleep(2)
        print("get detail url end")



if __name__ == "__main__":
    thread1 = GetDetailHtml("get_detail_html")
    thread2 = GetDetailUrl("get_detail_url")
    start_time = time.time()
    thread1.start()
    thread2.start()
    print("last time: {}".format(time.time() - start_time))
```

## `Queue`
使用队列，可以进行阻塞操作。它是基于`deque`开发的，因而是线程安全的。
初始化：
```python
import Queue
msg_deque = Queue.Queue()
```
放入队列：
```python
msg_deque.put(data)
```
取出队列，没有时间默认阻塞等待：
```python
msg_deque.get(block=True)
```
`msg_deque.task_done()`：每次从`queue`中`get`一个数据之后，当处理好相关问题，最后调用该方法，以提示`msg_deque.join()`是否停止阻塞。让线程向前执行或者退出：
```python
import queue

q = queue.Queue(2)
q.put(15)
q.put(59)

q.get()
q.task_done()  # get取完队列中的一个值后，使用task_done方法告诉队列，我已经取出了一个值并处理完毕
q.get()
q.task_done()

q.join()
```
> 如果没有`task_done`，那么会一直阻塞在`join`处

队列类型：
- 先进先出队列：`queue.Queue`
- 后进先出队列：`queue.LifoQueue`
- 优先级队列：`queue.PriorityQueue`
- 双向队列：`queue.deque`

源码分析：
```python
def put(self, item, block=True, timeout=None):
   # self.not_full是一个条件变量，如果size满足的话，直接进行put操作(append)，并且释放锁给get
   # 否则等待not_full(由get中的notify触发)
    with self.not_full:
        if self.maxsize > 0:
            if not block:
                if self._qsize() >= self.maxsize:
                    raise Full
            elif timeout is None:
                while self._qsize() >= self.maxsize:
                    self.not_full.wait()
            elif timeout < 0:
                raise ValueError("'timeout' must be a non-negative number")
            else:
                endtime = time() + timeout
                while self._qsize() >= self.maxsize:
                    remaining = endtime - time()
                    if remaining <= 0.0:
                        raise Full
                    self.not_full.wait(remaining)
        self._put(item)
        self.unfinished_tasks += 1
        self.not_empty.notify()
```
`get`同理，如果`size`为空，则等待`not_empty`，否则通知`put`可进行操作：
```python
def get(self, block=True, timeout=None):
    with self.not_empty:
        if not block:
            if not self._qsize():
                raise Empty
        elif timeout is None:
            while not self._qsize():
                self.not_empty.wait()
        elif timeout < 0:
            raise ValueError("'timeout' must be a non-negative number")
        else:
            endtime = time() + timeout
            while not self._qsize():
                remaining = endtime - time()
                if remaining <= 0.0:
                    raise Empty
                self.not_empty.wait(remaining)
        item = self._get()
        self.not_full.notify()
        return item
```

## 同步机制
> 除了`Lock`之外，还存在其他的锁

**`RLock`**：可重入锁，允许在**同一线程**中被调用多次，但是`acquire`与`release`的数量应当相等。
**`Condition`**：条件同步机制，一个线程等待特定条件，另一个线程发出特定条件满足的信号。具有`wait`和`notify`两种方法。`Condition`实际上有两把锁，当使用`wait`方法时，会释放最外层的锁，并分配一把锁，加至`__waiters__`这个队列里，只有当调用`notify`方法时，才会从`__waiters`方法中释放锁。
**`Semaphore`**：通过计数器限制可同时运行的线程数量。`acquire()`递减计数器，`release()`则是增加计数器。
```python
# -*- coding: utf-8 -*-
import threading

class Client(threading.Thread):
    def __init__(self, cond):
        super().__init__(name='client')
        self.cond = cond
    def run(self):
        with self.cond:
            self.cond.wait()
            print("client saying")
            self.cond.notify()


class Server(threading.Thread):
    def __init__(self, cond):
        super().__init__(name='server')
        self.cond = cond
    def run(self):
        with self.cond:
            print("server saying")
            self.cond.notify()
            self.cond.wait()


if __name__ == "__main__":
    cond = threading.Condition()
    server = Server(cond)
    client = Client(cond)
    client.start()
    server.start()
```
`condition`源码分析：
```python
def wait(self, timeout=None):
    if not self._is_owned():
       raise RuntimeError("cannot wait on un-acquired lock")
   # 获得一个内部锁
   waiter = _allocate_lock()
   waiter.acquire()
   self._waiters.append(waiter)
   # 释放condition锁
   saved_state = self._release_save()
   gotit = False
   try:    # restore state no matter what (e.g., KeyboardInterrupt)
       if timeout is None:
           # 等待notify释放锁才能再次进入
           waiter.acquire()
           gotit = True
       else:
           if timeout > 0:
               gotit = waiter.acquire(True, timeout)
           else:
               gotit = waiter.acquire(False)
       return gotit
   finally:
      # 恢复condition锁
       self._acquire_restore(saved_state)
       if not gotit:
           try:
               self._waiters.remove(waiter)
           except ValueError:
               pass
```
**`Semaphore`**：主要用于控制获取锁的数量
```python
# -*- coding: utf-8 -*-
import threading


class HtmlSpider(threading.Thread):
    def __init__(self, url, sem):
        super().__init__()
        self.url = url
        self.sem = sem

    def run(self):
        import time
        time.sleep(2)
        print('success!')
        self.sem.release()

class UrlProducer(threading.Thread):
    def __init__(self, sem):
        super().__init__()
        self.sem = sem

    def run(self):
        for i in range(20):
            self.sem.acquire()
            html_thread = HtmlSpider('http://www.baidu.com', self.sem)
            html_thread.start()


if __name__ == '__main__':
    sem = threading.Semaphore(3)
    url_procuder = UrlProducer(sem)
    url_procuder.start()
```
**`Event`**：一个线程通知事件，其他线程等待事件。内置了一个初始为`False`的标志，当调用`set()`时为`True`，调用`clear()`时重置为`False`。包括一下方法：
- `isSet()`：当内置标志为`True`时返回`True`
- `set()`：将内置标志位置为`True`
- `clear()`：将标志设为`False`
- `wait[timeout]`：如果标志位为`True`将立即返回，否则阻塞线程至等待阻塞状态，等待其他线程调用`set()`。当`timeout`不为空时，如果发生了超时，则返回值为`False`，否则返回状态为`True`。

举个例子：
```python
# -*- coding: utf-8 -*-
import threading
import time

event = threading.Event()

def func():
    # 等待事件，进入等待阻塞状态
    print '%s wait for event...' % threading.currentThread().getName()
    event.wait()

    # 收到事件后进入运行状态
    print '%s recv event.' % threading.currentThread().getName()

t1 = threading.Thread(target=func)
t2 = threading.Thread(target=func)
t1.start()
t2.start()

time.sleep(2)

# 发送事件通知
print 'MainThread set event.'
event.set()
```
> 在实际中，通常为每个线程准备一个独立的`Event`，而不是多个线程共享。应用场景，当用户成功上线了，才开启接收消息的线程。
## `concurrent.futures`
###  线程池
使用标准库的`from concurrent.futures`下的`ThreadPoolExecutor`。
主线程可以获取某一个线程的状态或者某一个任务的状态(`done`)，以及返回值(`result`)。
```python
from concurrent.futures import ThreadPoolExecutor
import time
def return_future_result(message):
    time.sleep(2)
    return message
pool = ThreadPoolExecutor(max_workers=2)  # 创建一个最大可容纳2个task的线程池
future1 = pool.submit(return_future_result, ("hello"))  # 往线程池里面加入一个task
future2 = pool.submit(return_future_result, ("world"))  # 往线程池里面加入一个task
print(future1.done())  # 判断task1是否结束
time.sleep(3)
print(future2.done())  # 判断task2是否结束
print(future1.result())  # 查看task1返回的结果
print(future2.result())  # 查看task2返回的结果
```
`ThreadPoolExecutor(pool_count)`中`pool_count`代表创建线程的数量，会返回一个该线程池的执行者对象，这个对象的`submit()`方法和`map()`方法，能够使用线程池中的线程来执行我们指定的方法，并且返回一个`Future`对象。并且当线程结束之后会自动归还，如果线程全部被占用，则会阻塞。
如果需要添加参数：
```python 
def return_future_result(message, n):
    time.sleep(n)
    return message
pool = ThreadPoolExecutor(max_workers=2)  # 创建一个最大可容纳2个task的线程池
future1 = pool.submit(return_future_result, "hello", 2)  # 往线程池里面加入一个task
future2 = pool.submit(return_future_result, "world", 1)  # 往线程池里面加入一个task
```

通过`as_completed()`方法获取已经完成的任务的结果：
```python
for future in as_completed(all_tasks):
    print(future.result())
```
或者使用`map`：
```python
for data in pool.map(get_url, urls):
    print(data)
```
> 与`as_completed`不同的是，`map`返回的不是`future`类型，同时输出顺序也与`urls`一致。


主要有以下方法：
- `submit`：立即返回
- `result`：获取执行结果
- `cancel`：取消`task`(未开始执行的)，成功返回`True`，失败返回`False`
- `as_completed`：是一个生成器，先完成的`task`先返回，返回的是`future`类型
- `map`：根据传入的顺序进行返回，返回的直接是`future.result()`
```python
# -*- coding: utf-8 -*-
from concurrent.futures import ThreadPoolExecutor, as_completed, wait
import time

def get_url(times):
    time.sleep(times)
    print("get page {} success".format(times))
    return times


executor = ThreadPoolExecutor(max_workers=2)
# task1 = executor.submit(get_url, (3))
# task2 = executor.submit(get_url, (2))

# print(task1.done())
# print(task2.cancel())

urls = [3, 2, 4]
all_task = [executor.submit(get_url, (url)) for url in urls]
wait(all_task)
print("all task done")
# for future in as_completed(all_task):
#     data = future.result()
#     print("get page {}".format(data))

# for future in executor.map(get_url, urls):
#     print("get page {}".format(future))
```
> 上面演示了两种方法进行多线程下载的方式，一种是利用`as_completed()`，另一种是利用`ThreadPoolExecutor.map()`。
> 通过`ThreadPoolExecutor.submit()`方法返回的是**`futures`对象**，作为`as_completed`的参数，因而可以调用其`result`方法，并且不会发生阻塞(因为`as_completed()`实际上是一个生成器)。
> 而针对后者，返回值是一个迭代器，通过其`__next__`方法调用各个`futures`的`result`方法，因此得到的是**`futures`的结果**。
> 如果需要提交任务的函数是一样的，就可以简化成`map`。但是加入提交的任务函数不一样的，或者执行的过程之可能出现异常，就要用到`submit`。

流程图：
```
|======================= In-process =====================|== Out-of-process ==|

+----------+     +----------+       +--------+     +-----------+    +---------+
|          |  => | Work Ids |    => |        |  => | Call Q    | => |         |
|          |     +----------+       |        |     +-----------+    |         |
|          |     | ...      |       |        |     | ...       |    |         |
|          |     | 6        |       |        |     | 5, call() |    |         |
|          |     | 7        |       |        |     | ...       |    |         |
| Process  |     | ...      |       | Local  |     +-----------+    | Process |
|  Pool    |     +----------+       | Worker |                      |  #1..n  |
| Executor |                        | Thread |                      |         |
|          |     +----------- +     |        |     +-----------+    |         |
|          | <=> | Work Items | <=> |        | <=  | Result Q  | <= |         |
|          |     +------------+     |        |     +-----------+    |         |
|          |     | 6: call()  |     |        |     | ...       |    |         |
|          |     |    future  |     |        |     | 4, result |    |         |
|          |     | ...        |     |        |     | 3, except |    |         |
+----------+     +------------+     +--------+     +-----------+    +---------+
```
- `executor.map`会创建多个`_WorkItem`对象，每个对象都传入了新创建的一个`Future`对象。
- 把每个`_WorkItem`对象然后放进一个叫做`「Work Items」`的`dict`中，键是不同的`「Work Ids」`。
- 创建一个管理`「Work Ids」`队列的线程`「Local worker thread」`，它能做2件事：
    - 从`「Work Ids」`队列中获取Work Id, 通过「Work Items」找到对应的`_WorkItem`。如果这个`Item`被取消了，就从`「Work Items」`里面把它删掉，否则重新打包成一个`_CallItem`放入`「Call Q」`这个队列。`executor`的那些进程会从队列中取`_CallItem`执行，并把结果封装成`_ResultItems`放入`「Result Q」`队列中。
    - 从`「Result Q」`队列中获取`_ResultItems`，然后从`「Work Items」`更新对应的`Future`对象并删掉入口。

## 进程池
`multiprocess`中的方法与多线程的比较类似：
- `apply_async`：非阻塞的，与之对应的是`apply`是阻塞的。所以需要使用`join`，否则在子进程还没开始运行之前主进程就已经完成退出了。
- `close`：关闭`pool`，使其不再接受新的任务
- `terminate`：结束工作进程，不再处理未完成的任务
- `join`：主进程阻塞，等待子进程的退出，`join`方法要在`close`或`terminate`之后使用。

```python
# -*- coding: utf-8 -*-
import multiprocessing
import time

def get_html(n):
    time.sleep(n)
    print("sub_progress success")
    return n


if __name__ == '__main__':
    pool  = multiprocessing.Pool(multiprocessing.cpu_count())
    results = []
    for i in range(4):
        results.append(pool.apply_async(get_html, args=(i,)))
    pool.close()
    pool.join()
    for result in results:
        print(result.get())
```
输出结果：
```python
sub_progress success
sub_progress success
sub_progress success
sub_progress success
0
1
2
3
```

## 进程间通信
`Queue`：不能使用`from queue import Queue`，这种方式只适用于同一进程下不同线程之间的共享，在多进程编程中，每一个进程中都会产生一个相应的副本。应该使用`multiprocess.Queue`。但是，这个`Queue`**不能用于进程池**。线程池中应该使用`Manager().Queue()`。
```python
# -*- coding: utf-8 -*-
import time
from multiprocessing import Process, Queue
from threading import Thread
# from queue import Queue

def producer(queue):
    queue.put("a")
    time.sleep(2)

def consumer(queue):
    time.sleep(2)
    data = queue.get()
    print(data)


if __name__ == '__main__':
    queue = Queue(10)
    my_producer = Process(target=producer, args=(queue,))
    my_consumer = Process(target=consumer, args=(queue,))
    my_consumer.start()
    my_producer.start()
    my_producer.join()
    my_consumer.join()
```
> `Manager()`中定义了很多共享变量。

`Pipe`：只适用于两个进程之间的通信，但是复杂度没有`Queue`高。
```python
# -*- coding: utf-8 -*-
import time
from multiprocessing import Process, Queue, Pool, Manager, Pipe

def producer(pipe):
    pipe.send("a")

def consumer(pipe):
    print(pipe.recv())


if __name__ == '__main__':
    send_pipe, recv_pipe = Pipe()
    my_producer = Process(target=producer, args=(send_pipe, ))
    my_consumer = Process(target=consumer, args=(recv_pipe, ))

    my_producer.start()
    my_consumer.start()
    my_producer.join()
    my_consumer.join()
```
