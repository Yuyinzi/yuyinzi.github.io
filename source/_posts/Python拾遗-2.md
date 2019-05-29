---
title: Python拾遗(2)
date: 2019-04-10 16:03:16
tags: Python 
categories:
    - Python
---
包括`Python`中的常用数据类型。
<!-- more -->
## `int`
在`64`位平台上，`int`类型是`64`位整数：
- 从堆上按需申请名为`PyIntBlcok`的缓存区域存储整数对象
- 使用固定数组缓存`[-5， 257]`之间的小数字，只需计算下标就能获得指针
- `PyIntBlock`内存不会返还给操作系统，直至进程结束

```python
>>> a = 15
>>> b = 15
>>> a is b
True
>>> a = 1000
>>> b = 1000
>>> a is b
False
>>> a = 257
>>> b = 257
>>> a is b
False
```
根据第三点，如果使用`range`创建巨大的数字列表，这就需要很大内存空间。但是换成`xrange`，每次迭代之后数字对象被回收，其占用内存空间出来并被复用，内存就不会暴涨。(仅指`python2`)。
当超出`int`限制时，会自动转换成为`long`。

## `float`
在`python2`中，`/`默认返回整数，而`python3`中默认返回浮点数。如果需要准确控制运算精度，有效位数和`round`的结果，使用`Decimal`代替。(但是建议往`Decimal`传入字符串型的浮点数 -- [为什么你需要少看垃圾博客以及如何在Python里精确地四舍五入](https://zhuanlan.zhihu.com/p/60952919)
```python
>>> from decimal import Decimal, ROUND_UP, ROUND_DOWN
>>> float('0.1') * 3 == float('0.3')
False
>>> Decimal('0.1') * 3 == Decimal('0.3')
True
```
## `string`
### 不太熟的方法
- `splitlines()`: 按行分割
- `find()`: 查找
- `lstrip()/rstrip()/strip()`: 剔除前后空格
- `replace()`：替换字符
- `expandtabs()`：将`tab`替换成空格
- `ljust()/rjust()/center()/zfill()`：填充
### 字符编码
在计算机内存中，统一使用`Unicode`编码，当需要保存到硬盘或者需要传输的时候，转换为`UTF-8`编码。
`py3`：
>包括`bytes`和`str`。字节的实例包含`8`个原生比特值，字符串实例是由`Unicode`字符组成

`py2`：
>包括`str`和`unicode`。字符串代表原生的`8`比特，而`unicode`由`Unicode`字符组成。

因此代码中，最核心的部分应该使用`Unicode`字符类型，即`py3`中使用`str`，`py2`中使用`unicode`。这两种字符类型没有相关联的*二进制编码*(原生的`8`个比特值)，因此如果要将`Unicode`转换为二进制数据，应该使用`encode`方法。而反过来，如果要*二进制编码*转化为`Unicode`字符，必须使用`decode`方法。
`py3`中的写法：
```python
def to_str(bytes_or_str):
    if isinstance(bytes_or_str, bytes):
        value = bytes_or_str.decode('utf-8')
    else:
        value = bytes_or_str
    return value

def to_bytes(bytes_or_str):
    if isinstance(bytes_or_str, str):
        value = bytes_or_str.encode('utf-8')
    else:
        value = bytes_or_str
    return value
```
`py2`中的写法：
```python
def to_unicode(unicode_or_str):
    if isinstance(unicode_or_str, str):
        value = unicode_or_str.decode('utf-8')
    else:
        value = unicode_or_str
    return value

def to_str(unicode_or_str):
    if isinstance(unicode_or_str, unicode):
        value = unicode_or_str.encode('utf-8')
    else:
        value = unicode_or_str
    return value
```
需要注意的是两大陷阱：
- 在`py2`中，当一个`str`类型仅仅包含7个比特的`ASCII`码字符的时候，`unicode`和`str`实例看起来是一致的。因此可以采用：
    - `+`运算符合并`str`和`unicode`
    - 可以使用等价或者不等价运算符来比较`str`和`unicode`实例
    - 可以使用`unicode`来替换像`%s`这种字符串中的格式化占位符
> 然而在`py3`中，`bytes`和`str`的实例是不可能等价的。

- 在`py3`中，涉及到文件的处理操作，例如采用内置的`open`函数，会默认以`utf8`编码，而在`py2`中默认采用二进制的形式编码。举个例子，在`py3`的情况下，会报错：
```python
def open('/tmp/random.bin','w') as f:
    f.write(os.urandom(10))

>>>
TypeError: must be str,not bytes
```
> 这是因为`py3`中对于`open`函数新增了名为`encoding`的参数，并且此参数默认值为`utf-8`。因此，其对文件的源的期望是包含了`unicode`字符串的`str`实例，而不是包含了二进制的`bytes`文件。因此，可以采用`py2`与`py3`中都通用的方法：
```python
with open('/tmp/random.bin','wb') as f:
    f.write(os.urandom(10))
```
**画重点**
- `py3`中，`bytes`是包含`8`个比特位的序列，`str`是包含`unicode`的字符串，它们不能同时出现在操作符`>`或者`+`中。
- `py2`中，`str`是包含`8`个比特位的序列，而`unicode`是包含`unicode`的字符串，二者可以同时出现在只包含7个比特的`ASCII`码的运算中。
- 使用工具函数来确保程序输入的数据是程序预期的类型，例如上面的`to_str`之类的函数。
- 总是使用`wb`和`rb`模式来处理文件。

### `string.ascii_letters/digits`
使用`string`类的`ascii_letters`或者`digits`可以获得大小写字符，以及数字，避免自己写循环生成：
```python
>>> import string
>>> string.ascii_letters
'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
>>> type(string.ascii_letters)
<class 'str'>
>>> string.digits
'0123456789'
```
#### 池化
使用`intern()`可以把运行期动态生成的字符串池化：
```python
>>> s = "".join(['a', 'b', 'c'])
>>> s is "abc"
False
>>> intern(s) is "abc"
True
```
##  `dict`
- `popitem()`
>随机返回并删除字典中的一对键值

- `setdefault(key, default=None)`
> 如果字典中包含有给定键，则返回键对应的值，否则返回为该键设置的值。

- `fromkeys(it, [initial])`
> 返回以可迭代对象`it`里的元素为键值的字典，如果有`initial`参数，就把它作为这些键对应的值，默认是`None`
```python
In [104]: info = {}.fromkeys(['name', 'blog'])

In [105]: info
Out[105]: {'blog': None, 'name': None}

In [106]: info = {}.fromkeys(['name', 'blog'], ['angel', 'jay'])

In [107]: info
Out[107]: {'blog': ['angel', 'jay'], 'name': ['angel', 'jay']}

In [110]: info = {}.fromkeys(['name', 'blog', 'test'], 'angel')

In [111]: info
Out[111]: {'blog': 'angel', 'name': 'angel', 'test': 'angel'}
```
- `update`
> 使用指定键值对更新字典

可以采用`key=value`的形式：
```python
In [112]: info.update(blog='jay')

In [113]: info
Out[113]: {'blog': 'jay', 'name': 'angel', 'test': 'angel'}
```
也可以采用`{key:value}`的形式：
```python
In [114]: info.update({'test':'album'})

In [115]: info
Out[115]: {'blog': 'jay', 'name': 'angel', 'test': 'album'}
```
也可以使用`tuple`的形式：
```python
In [119]: info.update((('name', 'unkown'),('test', 'secret')))

In [120]: info
Out[120]: {'blog': 'jay', 'name': 'unkown', 'test': 'secret'}
```
对于大字典，调用`keys()`、`values()`、`items()`会同样构造巨大的列表，因此可以使用迭代器(`iterkeys()`、`itervalues()`、`iteritems()`)减少内存开销。
```python
>>> d = {"a":1, "b":2}
>>> d.iterkeys()
<dictionary-keyiterator object at 0x7fde6e70b368>
>>> d.itervalues()
<dictionary-valueiterator object at 0x7fde6e70b3c0>
>>> d.iteritems()
<dictionary-itemiterator object at 0x7fde6e70b368>
>>> for k, v in d.iteritems():
...     print k, v
... 
a 1
b 2
```
### 视图
判断两个字典间的差异，除了使用`Counter`，也可以使用视图。
```python
>>> d1 = dict(a=1, b=2)
>>> d2 = dict(b=2, c=3)
>>> d1 & d2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for &: 'dict' and 'dict'
>>> v1 = d1.viewitems()
>>> v2 = d2.viewitems()
>>> v1 & v2
set([('b', 2)])
>>> v1 | v2
set([('a', 1), ('b', 2), ('c', 3)])
>>> v1 - v2
set([('a', 1)])
>>> v1 ^ v2
set([('a', 1), ('c', 3)])
```
视图会和字典同步变更：
```python
>>> d = {"a": 1}
>>> v = d.viewitems()
>>> v
dict_items([('a', 1)])
>>> d["b"] = 2
>>> v
dict_items([('a', 1), ('b', 2)])
>>> del d["a"]
>>> v
dict_items([('b', 2)])
```
### `collections.defaultdict`
在实例化一个`defaultdict`的时候，需要给构造方法提供一个**可调用对象**或者**无参数函数**，这个可调用对象会在`__getitem__`(例如`dd`是个`defaultdict`，当`dd[k]`时会调用此方法)碰到找不到的键的时候被调用，让`__getitem__`返回某种默认值。如果在创建`defaultdict`的时候没有指定可调用对象，查询不存在的键会触发`KeyError`(这是由于`__missing__`方法的缘故)。
使用一个类型初始化，当访问键值不存在时，默认值为该类型实例。
```python
>>> dd = defaultdict(list)
>>> dd['foo']
[]
>>> cc = defaultdict(dict)
>>> cc['foo']
{}
```
也可以使用不带参数的可调用函数进行初始化，当键值没有指定的时候，可以采用该函数进行初始化
```python
>>> import collections
>>> def default_factory():
...     return 'default value'
... 
>>> d = collections.defaultdict(default_factory, foo='bar')
>>> print('d:', d)
d: defaultdict(<function default_factory at 0x7f2c39c04378>, {'foo': 'bar'})
>>> print(d['foo'])
bar
>>> print(d['bar'])
default value
```
```python
>>> def zero():
...     return 0
... 
>>> ff = defaultdict(zero)
>>> ff['foo']
0
```
简化为：
```python
>>> ee = defaultdict(lambda : 0)
>>> ee['foo']
0
```
### `OrderedDict`
字典是哈希表，默认迭代是无序的，如果需要元素按照添加顺序输出结果，可以使用`OrderedDict`。
```python 
>>> from collections import OrderedDict
>>> od = OrderedDict()
>>> od["a"] = 1
>>> od["c"] = 3
>>> od["b"] = 2
>>> for k, v in od.items(): print k, v
... 
a 1
c 3
b 2
>>> od.popitem()
('b', 2)
>>> od.popitem()
('c', 3)
>>> od.popitem()
('a', 1)
```
> 因为字典是有序的，因此当使用`popitem`时，不再是随机弹出

## `set`
`set`方法的`pop`也是随机弹出的。集合和字典的主键都必须是可哈希类型对象。
```python
>>> a = frozenset('abc')
>>> a
frozenset(['a', 'c', 'b'])
>>> a.add('d')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'frozenset' object has no attribute 'add'
>>> b = set('abc')
>>> b.add('d')
>>> b
set(['a', 'c', 'b', 'd'])
```
如果需要将自定义类型放入集合中，需要保证`hash`和`equal`的结果都相同才行：
```python
# -*- coding: utf-8 -*-
class User(object):

    def __init__(self, name):
        self.name = name

    def __hash__(self):
        return hash(self.name)

    def __eq__(self, other):
        cls = type(self)
        if not other or not isinstance(other, cls):
            return False
        return self.name == other.name


s = set()
s.add(User('tom'))
s.add(User('tom'))
print s
```

##  `list`
对于频繁增删元素的大型列表，应该考虑使用链表或者迭代器创建列表对象的方法(`itertools`)。某些时候，可以考虑用数组(`array`)代替列表，但是它只能放指定的数据类型：
```python
>>> import array
>>> a = array.array("l", range(10)) # 创建一个long类型的数组
>>> a
array('l', [0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> a.tolist()   #转化为list
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> a = array.array("c")
>>> a.fromstring("abc")
>>> a
array('c', 'abc')
>>> a.fromlist(list("def"))
>>> a
array('c', 'abcdef')
>>> a.extend(array.array("c", "xyz"))
>>> a
array('c', 'abcdefxyz')
```
### 向有序列表插入元素
```python
>>> l = ["a", "d", "c", "e"]
>>> l.sort()
>>> bisect.insort(l, "b")
>>> l
['a', 'b', 'c', 'd', 'e']
```

## `Tuple`
在对`tuple`进行比较的时候，`Python`会优先比较元组中下标为`0`的元素，然后依次递增。有个很神奇的例子：
```python
>>> values = [1,5,3,9,7,4,2,8,6]
>>> group = [7, 9]
>>> def sort_priority(values, group):
...     def helper(x):
...         if x in group:
...             return (0, x)
...         return (1, x)
...     values.sort(key=helper)
...     
>>> sort_priority(values, group)
>>> print(values)
[7, 9, 1, 2, 3, 4, 5, 6, 8]
```
### `namedtuple`
需要两个参数，一个是类名，另一个是类的各个字段的名字。后者可以是由数个字符串组成的可迭代对象，或者是由空格分隔开的字段名组成的字符串。
> 继承自`tuple`的子类，创建的对象拥有可以访问的属性，但是仍然是只读的。
```python
>>> from collections import namedtuple
>>> TPoint = namedtuple('TPoint', ['x', 'y'])
>>> p = TPoint(x=10, y=10)
>>> p.x, p.y
(10, 10)
```
> 也可以使用`TPoint = namedtuple('TPoint', 'x y')`这种格式

- 将数据变为`namedtuple`类：
```python
>>> TPoint = namedtuple('TPoint', ['x', 'y'])
>>> t = [11 , 22]
>>> p = TPoint._make(t)
>>> p
TPoint(x=11, y=22)
>>> p.x
11
```
- 如果要进行更新，需要调用方法`_replace`：
```python
>>> p
TPoint(x=10, y=10)
>>> p.y = 33
Traceback (most recent call last):
  File "<input>", line 1, in <module>
AttributeError: can't set attribute
>>> p._replace(y=33)
TPoint(x=10, y=33)
```
- 将字典数据转换成`namedtuple`：
```python
>>> p = {'x': 44, 'y': 55}
>>> p = TPoint(**p)
>>> p
TPoint(x=44, y=55)
```
