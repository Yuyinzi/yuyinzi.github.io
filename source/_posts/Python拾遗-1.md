---
title: Python拾遗(1) 
date: 2019-04-10 15:46:27
tags: Python 
categories: 
    - Python
---
## 虚拟机
`python`会在模块载入时将源码编译成字节码，然后字节码会被虚拟机在核心函数中解释执行。
虚拟机开始运行时，通过初始化函数完成整个运行环境设置，包括：
- 创建`__builtlin__`模块，持有所有内置类型和函数
- 创建`sys`模块
- 初始化内置`Exception`
- 创建`__main__`模块，准备所需名字空间
- 通过`site.py`将`site-packages`中的第三方扩展库添加到搜索路径列表
- 执行入口`py`文件，执行前将`__main__.__dict__`作为名字空间传递进去
- 程序执行结束后执行清理操作
<!-- more -->
## 类型和对象
每个对象都包含一个标准头，通过头部信息就可以明确知道其具体类型。头部信息由“引用计数”和“类型指针”组成。
```python
>>> import sys
>>> x = 0x1234
>>> sys.getsizeof(x)
24
>>> sys.getrefcount(x)    //作为形参也会增加一次引用计数
2
>>> y = x
>>> sys.getrefcount(x)
3
>>> del y
>>> sys.getrefcount(x)
2
```
所有内置类型对象都能从`types`模块中找到，`int`、`long`、`str`都可以看做是简短别名：
```python
>>> import types
>>> x = 20
>>> type(x) is types.IntType
True
>>> x.__class__
<type 'int'>
>>> x.__class__ is type(x)
True
>>> x.__class__ is int
True
>>> x.__class__ is types.IntType
True
```
## 命名空间
实际上`python`中，名字实际上是字符串对象，和所指向的目标对象在命名空间中构成`{name: object}`关联。它有多种命名空间，比如`globals`(模块命名空间)，`locals`函数堆栈帧命名空间，以及`class`、`instance`等。
```python
>>> globals()
{'__builtins__': <module '__builtin__' (built-in)>, '__package__': None, 'sys': <module 'sys' (built-in)>, 'x': 123, '__name__': '__main__', '__doc__': None, 'types': <module 'types' from '/usr/lib/python2.7/types.pyc'>}
>>> globals()["y"] = "hello"
>>> y
'hello'
```
> 名字的作用仅仅是在某个时刻与名字空间的某个对象进行关联。其本身不包含目标对象的任何信息，只有通过对象头部的类型指针才能获得其具体类型。因此可以在运行期随时将其关联到任何类型对象。

在函数外部时，`locals()`和`globals()`作用完全相同，在函数内部调用时，`locals()`则是获取当前函数堆栈帧的命名空间，其中存储的是函数参数、局部变量等。
```python
>>> import sys
>>> def test(x):
...     y = x + 100
...     print locals()
...     print globals() is locals()
...     frame = sys._getframe(0)     # 获取当前堆栈帧
...     print locals() is frame.f_locals
...     print globals() is frame.f_globals
... 
>>> test(123)
{'y': 223, 'x': 123}
False
True
True
```
在函数内部中调用`globals()`时，总是获取包含该函数定义的模块命名空间，而非调用处的：
```python
# test.py
# -*- coding: utf-8 -*-
a = 1
def test():
    print {k:v for k, v in globals().items() if k != "__builtins__"}

# test1.py
import test

print test.test()

# 结果
{'a': 1, '__file__': '/home/may/study/test.py', '__package__': None, 'test': <function test at 0x7f6fc1119848>, '__name__': 'test', '__doc__': None}
```
可以通过`<module>.__dict__`访问其他模块的命名空间。
```python
>>> import sys
>>> sys.modules[__name__].__dict__ is globals() //当前模块命名空间和globals相同
True
```
然而使用命名空间管理上下文，虽然能带来灵活性，但是相比指针低效很多，因此牺牲了一部分的执行性能。

## 内存管理
``python`采用内存池减少操作系统内存分配和回收操作，小于等于`256`字节对象，将直接从内存池中获取存储空间。根据需要，虚拟机每次从操作系统申请一块`256kb`的内存(`areana`)，并且根据系统页划分为多个`pool`。每个`pool`分割成以`8`为倍数的`block`，这是内存池的最小单位。
大于`256`字节的对象，直接使用`malloc`在堆上分配内存。
当`areana`总容量超出`64MB`时，不再请求新的`arena`，而是直接在堆上为对象分配内存。完全空闲的`arena`会被释放交还给操作系统。

### 引用传递
对象总是按引用传递，即通过复制指针来实现多个名字指向同一对象。因为`arena`是在堆上分配的，所以无论何种类型何种大小的对象，都存储在堆上。因此，没有值类型和引用类型的区分。
不可变类型：
```
int, long, str, tuple, frozenset
```
### 深拷贝与浅拷贝
> 浅拷贝是对引用的拷贝(**仅复制对象自身，而不会递归复制其成员**)，深拷贝是对对象资源的拷贝。

需要注意的是：
- 不可变类型当修改时会创建新的对象，产生新的地址
- 切片操作`[:]`，使用工厂函数`list/dict/set`以及`copy()`都会产生浅拷贝的效果

**赋值**操作：
```python
In [57]: will = ["will", 28, ["python", "c#", "javascript"]]

In [58]: wilber = will

In [59]: wilber is will
Out[59]: True

In [60]: print(id(will),id(wilber))
140110220425032 140110220425032

In [61]: print([id(element) for element in will])
[140110445106880, 10912032, 140110220811400]

In [62]: print([id(element) for element in wilber])
[140110445106880, 10912032, 140110220811400]

In [63]: will[0] = 'wilber'

In [64]: print(id(will),id(wilber))
140110220425032 140110220425032

In [65]: print([id(element) for element in will])
[140110221716032, 10912032, 140110220811400]

In [66]: print([id(element) for element in wilber])
[140110221716032, 10912032, 140110220811400]
```
> 赋值操作相当于将两个指针都指向了同一个地址

**浅拷贝**：
```python
In [67]: import copy

In [68]: wilber = copy.copy(will)
# 浅拷贝会创建一个新的对象
In [69]: print(id(will),id(wilber))
140110220425032 140110470159432
# 对于对象中的元素，只会使用原始元素的内存地址
In [70]: print([id(element) for element in will])
[140110221716032, 10912032, 140110220811400]

In [71]: print([id(element) for element in wilber])
[140110221716032, 10912032, 140110220811400]

In [72]: will[0] = 'will'

In [73]: will[2].append('css')
# 对不可变对象的修改会创建新的对象，对可变对象的修改会影响新的对象
In [74]: will, wilber
Out[74]: 
(['will', 28, ['python', 'c#', 'javascript', 'css']],
 ['wilber', 28, ['python', 'c#', 'javascript', 'css']])

In [75]: print([id(element) for element in will])
[140110445106880, 10912032, 140110220811400]

In [76]: print([id(element) for element in wilber])
[140110221716032, 10912032, 140110220811400]
```
**深拷贝**：
```python
In [77]: wilber = copy.deepcopy(will)

In [78]: print(id(will),id(wilber))
140110220425032 140110423481608
# 即使是对象中的可变对象，也会重新生成一份
In [79]: print([id(element) for element in will])
[140110445106880, 10912032, 140110220811400]

In [80]: print([id(element) for element in wilber])
[140110445106880, 10912032, 140110220285128]

In [81]: will[0] = 'wilber'

In [82]: will[2].append('java')

In [83]: will, wilber
Out[83]: 
(['wilber', 28, ['python', 'c#', 'javascript', 'css', 'java']],
 ['will', 28, ['python', 'c#', 'javascript', 'css']])

In [84]: print([id(element) for element in will])
[140110221716032, 10912032, 140110220811400]

In [85]: print([id(element) for element in wilber])
[140110445106880, 10912032, 140110220285128]
```
#### 引用计数
当引用计数为`0`时，将立即回收该对象内存，要么将对应的`block`标记为空闲，要么返还给操作系统，在回收时会调用`__del__`方法：
```python
>>> class User(object):
...     def __del__(self):
...             print "dead"
... 
>>> a = User()
>>> sys.getrefcount(a)
2
>>> del a
dead
```
> 某些内置类型，比如小整数，因为缓存的缘故，计数永远不会为0，直到进程结束才由虚拟机清理函数释放。`python`还支持弱引用，允许在不增加引用计数，不妨碍对象回收的情况下间接引用对象。但是`list`、`dict`弱引用会引发异常。

### 弱引用
引用计数无法回收循环引用的对象，因此在对象群组内部使用弱引用有时能避免出现引用循环。它与强引用相对，是指不能确保其引用的对象不会被垃圾回收器回收的引用。一个对象若只被弱引用所引用，则可能在任何时候被回收。**弱引用的主要作用就是减少循环时引用，减少内存中不必要的对象存在的数量。**
创建弱引用可以通过`weakref`模块的`ref(obj[,callback])`来创建弱引用，`obj`是想弱引用的对象，`callback`是一个可选的函数。当没有引用导致`python`要销毁这个对象时，调用`callback`，`callback`要求单个参数(弱引用的对象)。
```python
>>> import sys
>>> import weakref
>>> class Man:
...     pass
... 
>>> o = Man()
>>> sys.getrefcount(o)
2
>>> r = weakref.ref(o)
>>> sys.getrefcount(o)
2
>>> r
<weakref at 0x7f14e2558a48; to 'instance' at 0x7f14e24dc830>
>>> o1 = r()
>>> o1 is o
True
>>> sys.getrefcount(o)
3
>>> o = None
>>> o1 = None
>>> sys.getrefcount(o)
2548
>>> r
<weakref at 0x7f14e2558a48; dead>
```
### 垃圾回收
除了引用计数，还有专门处理循环引用的`GC`。一般能引发循环引用问题的，都是容器类对象，例如`list`、`set`、`object`。当不存在循环引用时，理论上可以禁用`GC`。
```python
>>> import gc
>>> gc.disable()
```
```python
>>> class User(object): pass
... 
>>> def callback(r): print r, "dead"
... 
>>> gc.disable()
>>> a = User(); wa = weakref.ref(a, callback)
>>> b = User(); wb = weakref.ref(b, callback)
>>> a.b = b; b.a = a;
>>> del a; del b;
>>> wa(), wb()
(<__main__.User object at 0x7f14e24e1190>, <__main__.User object at 0x7f14e24e11d0>)
>>> gc.enable()
>>> gc.isenabled()
True
>>> gc.collect()
<weakref at 0x7f14e2558b50; dead> dead
<weakref at 0x7f14e2558ba8; dead> dead
11
```
`GC`将要回收的对象分成`3`级：
```python
>>> import gc
>>> gc.get_threshold()
(700, 10, 10)
>>> gc.get_count()
(263, 9, 0)
```
> 包含`__del__`方法的循环引用对象，永远不会被`GC`回收，直至进程终止。

```python
>>> class User(object):
...     def __del__(self): pass
... 
>>> def callback(r): print r, "dead!"
... 
>>> gc.set_debug(gc.DEBUG_STATS | gc.DEBUG_LEAK)   # 输出详细的回收状态信息
>>> gc.isenabled()
True
>>> a = User(); wa = weakref.ref(a, callback)
>>> b = User(); wb = weakref.ref(b, callback)
>>> a.b = b; b.a = a
>>> del a; del b
>>> gc.collect()                                
gc: collecting generation 2...
gc: objects in each generation: 10 0 3637
gc: uncollectable <User 0x7f14e24e1210>            # a
gc: uncollectable <User 0x7f14e24e1250>            # b
gc: uncollectable <dict 0x7f14e2550b40>            # a.__dict__
gc: uncollectable <dict 0x7f14e2550910>            # b.__dict__
gc: done, 4 unreachable, 4 uncollectable, 0.0023s elapsed.
4
```
## 编译
要运行`python`程序，必须将源码编译成字节码。通常情况下，编译器会将源码转化成字节码后保存在`pyc`文件中。编译发生在模块载入那一刻。
- 载入`pyc`流程：
    - 核对文件`Magic`标记(由`Python`版本号计算得来)
    - 检查时间戳和源码文件修改时间是否相同，以确定是否需要重新编译
    - 载入模块

- 如果没有`pyc`，需要先进行编译
    - 对源码进行`AST`分析
    - 将分析结果编译成`PyCodeObject`
    - 将`Magic`、源码文件修改时间、`PyCodeObject`保存到`pyc`文件中
    - 载入模块

`PyCodeObject`包含了代码对象的完整信息。内部成员嵌入到`co_consts`列表中。
