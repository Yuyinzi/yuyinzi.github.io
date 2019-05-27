---
title: 简单实现LRU
date: 2019-05-27 22:15:37
tags:
- Python
categories：
- 算法
---

## 要求

设计和实现一个`LRU`(最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 `get`和 写入数据`save` ，这两种操作的时间复杂度都为`O(1)`。

<!--more-->

`get`: 如果`key`存在于缓存中，则获取密钥的值（总是正数），否则返回 -1。 

`save(key, value)`：如果`key`不存在，则写入其数据值，如果存在，覆盖其原来数据值。当缓存容量达到上限时，它应该在写入新数据之前删除最近最少使用的数据值，从而为新的数据值留出空间。

`Example`:

```python 
LRUCache cache = new LRUCache(2);

cache.save(1, 1);
cache.save(2, 2);
cache.get(1);       // 返回  1
cache.save(3, 3);    // 该操作会使得密钥 2 作废
cache.get(2);       // 返回 -1 (未找到)
cache.save(4, 4);    // 该操作会使得密钥 1 作废
cache.get(1);       // 返回 -1 (未找到)
cache.get(3);       // 返回  3
cache.get(4);       // 返回  4
```



## 实现

要进行插入和读取的复杂度都为`O(1)`，可以使用双端链表以及字典来实现。因此需要自定义一个双端链表节点，以及双端链表，与字典一起构成个`LRU`。

每当插入新值/修改值/获得值时，都需要将节点插入到链表头。当链表超过自定义大小时，将尾端节点删除。

下面是一个简单实现(没有进行异常处理)：

```python 
# -*- coding: utf-8 -*-
# author: May
# Time: 2019-05-27 18:01
class Node(object):
    def __init__(self, key, value):
        self.key = key
        self.value = value
        self.pre = None
        self.nxt = None


class DoubleLinklist(object):
    def __init__(self):
        self.head = Node(0, 0)
        self.tail = Node(0, 0)
        self.head.nxt = self.tail
        self.tail.pre = self.head
        self.size = 0

        
class LRU(object):
    def __init__(self, size):
        self.link = DoubleLinklist()
        self.hash_map = {}
        self.size = size

    def add_to_head(self, node):
        f_node = self.link.head.nxt
        # 如果第一个节点是自己，不移动，否则会使自己指向自己
        if node != f_node:
            self.link.head.nxt, node.pre = node, self.link.head
            node.nxt, f_node.pre = f_node, node

    def remove_node(self, node):
        p_node, n_node = node.pre, node.nxt
        p_node.nxt, n_node.pre = n_node, p_node


    def save(self, key, value):
        # 节点存在，则直接更改值，放置于链表头
        if key in self.hash_map:
            node = self.hash_map[key]
            node.value = value

            self.add_to_head(node)

        else:
            # 否则创建一个新节点，放置于链表头
            node = Node(key, value)
            self.add_to_head(node)
            self.hash_map[key] = node
            self.link.size += 1

            # 当前链表大小大于指定大小时，才需要删除最久未使用的节点
            if self.link.size > self.size:
                t_node = self.link.tail.pre
                self.remove_node(t_node)
                del self.hash_map[t_node.key]
                del t_node
                self.link.size -= 1
        return 'SUCCESS'

    def get(self, key):
        if key in self.hash_map:
            node = self.hash_map[key]
            self.add_to_head(node)
            return node.value
        else:
            return -1

    def delete(self, key):
        if key in self.hash_map:
            node = self.hash_map[key]
            self.remove_node(node)
            self.link.size -= 1
            del self.hash_map[key]
            del node
            return 'SUCCESS'
        else:
            return -1


cache = LRU(3)

print(cache.save(1, 1))
# SUCCESS
print(cache.save(2, 2))
# SUCCESS
print(cache.save(1, 3))
# SUCCESS
print(cache.get(1))
# 3
print(cache.save(3, 3))
# SUCCESS
print(cache.get(2))
# 2
print(cache.save(4, 4))
# SUCCESS
print(cache.get(1))
# -1
print(cache.get(3))
# 3
print(cache.get(4))
# 4
print(cache.delete(4))
# SUCCESS
print(cache.get(4))
# -1
print(cache.delete(1))
# -1
print(cache.delete(3))
# SUCCESS
print(cache.get(3))
# -1
```

`References:`

[LRU cache](https://juejin.im/post/5cab4ae46fb9a0688d2e24b4#heading-6)

