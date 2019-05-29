---
title: Rabbitmq学习记录
date: 2019-04-25 15:28:17
tags:
- Rabbitmq
- Python
categories:
- MQ
---

## 优点

- 实现了`AMQP`标准的消息服务器
- 可靠性，使用持久化、传输确认、发布确认等保证
- 灵活的路由
- 集群部署简单
- 支持多种协议，以及多种语言客户端
- 易用的用户界面

...

<!-- more -->

## `MAC`下的安装

采用`Homebrew`进行安装：

```shell 
➜  ~ brew update
➜  ~ brew install rabbitmq
```

通过`brew`安装的文件位于`/usr/local/Cellar/rabbitmq/3.7.14/sbin`中。开启插件：

```shell 
➜  /usr/local/Cellar/rabbitmq/3.7.14 sudo sbin/rabbitmq-plugins enable rabbitmq_management
```

直接运行`rabbitmq-server`会提示找不到命令：

```shell 
➜  ~ rabbitmq-server
zsh: command not found: rabbitmq-server
```

因为使用的是`zsh`，所以需要将路径添加至`.zshrc`中：

```shell 
➜  ~ cat .zshrc
# If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:/usr/local/bin:$PATH
# Path to your oh-my-zsh installation.
export PATH=$PATH:/usr/local/opt/rabbitmq/sbin
...
```

再`source .zshrc`使它生效。这下`rabbit-server`是有用了，但是启动却报错了：

```shell 
2019-04-25 15:18:09.525358
    args: []
    format: "Error when reading /Users/littlemay/.erlang.cookie: eacces"
    label: {error_logger,error_msg}
```

原因是`/Users/littlemay/.erlang.cookie`这个文件没有权限，需要更改文件所有者以及权限：

```shell 
➜  ~ ls -l /Users/littlemay/.erlang.cookie
-r--------  1 root  staff  20  4 25 00:00 /Users/littlemay/.erlang.cookie
➜  ~ sudo chown littlemay:staff /Users/littlemay/.erlang.cookie
➜  ~ sudo chmod u+x /Users/littlemay/.erlang.cookie
```

折腾了半个小时，此时终于可以不使用`sudo`启动了。

可以使用`rabbitmqctl start_app/stop_app`启动/停止`rabbitmq`。网页端的用户界面访问：`localhost:15762`，默认用户名密码都为`guest`。

## 基本概念

- `Message`：消息，`AMQP`中预定了`14`个属性，即后文`pika`中会用到的`BasicProperties`
- `Producer`：消息的生产者，也是一个向交换机发布消息的客户端应用程序
- `Consumer`：消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。
- `Exchange`：交换机，用来接收生产者发送的消息并根据规则将这些消息路由给服务器中的队列
- `Queue`：消息队列，用来保存消息直到发送给消费者。一个消息可投入一个或多个队列。如果一个队列没有被任何消费者队列，那么它将一直在队列里面，等待消费者连接到这个队列将其取走。多个消费者可以订阅同一个队列，此时队列的消息会被**平均分摊**(`round-robin`)给多个消费者处理。
- `Binding`：绑定，将`Exchange`和`Queue`按照路由规则绑定
- `Channel`：信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的`TCP`连接内的虚拟连接，`AMQP` 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁`TCP`都是非常昂贵的开销，所以引入了信道的概念，以复用一条`TCP`连接
- `Virtual Host`：虚拟主机，用作不同用户的权限分离。默认的`vhost`是`/`。
- `Broker`：表示消息队列服务器实体

### `Routing Key`与`Binding Key`

`Binding Key`是在`Exchange`与`Queue`进行`Binding`时使用的路由规则。多个`Queue`允许使用同一个`Binding Key`与`Exchange`进行绑定。

`Routing Key`是当生产者将消息发送给`Exchange`时，指定的路由规则。它会依据`Exchange Type`及`Binding Key`联合使用来决定将消息投放到哪个队列中。

> `Routing Key`的长度限制为`255 bytes`。

### `Exchange Types`

通常使用的有四种：

- `fanout`：将所有发送到该`Exchange`的消息路由到所有与它绑定的`Queue`中，此时`Binding Key`与`Routing Key`不起作用
- `direct`：将所有发送到该`Exchange`的消息路由到`Binding Key`与`Routing Key`完全匹配的`Queue`中
- `topic`：将所有发送到该`Exchange`的消息路由到`Binding Key`与`Routing Key`相匹配的`Queue`中。采用`.`进行分隔(每分隔开的一段独立字符串成为一个单词)，`*`表示匹配一个单词，`#`表示匹配`0`个或者多个。
- `headers`：不依赖于`Routing Key`与`Bingding Key`的匹配规则路由消息，而是根据发送消息内容中的`headers`属性进行匹配，它与`direct`功能一致，但是性能十分低，一般不使用。

### `Message Acknowledgment`

消费者接收到了消息，`RabbitMQ`并不会立即将消息从队列中丢弃，而是在接收到消费者的`ACK`响应之后，才进行丢弃。

如果`RabbitMQ`没有收到回执并检测到消费者的`RabbitMQ`断开，将会传递给其他消费者直到队列接收到了`ACK`，以此保证每一个消息都能被有效传递。因此，如果忘记回复`ACK`，那么队列中的消息会越来越多，`RabbitMQ server`不会再发送数据给它，可以起到限流的作用。

`Prefetch Count`

允许消费者每次从队列中获取任意数量的消息。针对任务粒度小，执行时间短的消费者，可以设定更大的预读取数。

`RPC`远过程调用

除了异步通信之外，`RabbitMQ`也支持同步调用。实现原理如图：

![RPC调用](/assets/rpc.jpg)

1. 生产者在生产请求消息时，会在请求消息的属性中设置两个`replyTo` 值：一个为`Queue Name`，用于告知消费者将处理完成后的通知消息返回到该队列；另一个为`correlationId`，是请求消息的唯一标示，随着请求消息一同发送给消费者，也会随着响应消息返回给生产者，生产者就能够通过`correlationId`来判定来判定请求是否被成功执行，最终实现请求和响应的一一配对。
2. 生产者只有在接收到响应消息之后才会继续发生下一次请求消息，以此实现同步的效果

## `Python3`实践

首先需要安装`pika`，分别建立消费者和生产者：

**生产者**

```python 
# rabbit_p.py
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))

channel = connection.channel()

channel.queue_declare(queue='hello')

for i in range(20):
    body = '%s Hello rabbitmq' % i
    channel.basic_publish(exchange='',
                          routing_key='hello',
                          body=body)

connection.close()
```

**消费者**

```python 
# rabbit_c1.py
import pika
connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel = connection.channel()
# 为了防止queue不存在，最好消费者和生产者都创建
channel.queue_declare(queue='hello')

def callback(ch, method, properties, body):
    print(body)
# 如果不开启ack，消息会一直显示unacked，每当消费者重启又会重复消费
channel.basic_consume(on_message_callback=callback,
                       queue='hello',
                     auto_ack=True)

channel.start_consuming()
```

> 使用`basic_consume`而不是循环调用`basic_get`

如果使用`exchange=''`，会绑定在`RabbitMQ`默认的`Exchange`下，它绑定了所有的`queue`，`routing key`使用`queue name`。

如果再开启一个消费者，那么他们会轮流处理消息。

### `prefetch_count`

为了防止有的消费者空闲，所以可以设置`prefetch_count`参数(使用`basic_qos`来指定)，当`RabbitMQ`发现有个消费者迟迟没有返回`ACK`，便不会再将消息分配给它，此时需要我们采用主动`ACK`的方式，消费者`1`不进行`ack`，消费者`2`进行`ack`:

```python 
# rabbit_c1.py
...
def callback(ch, method, properties, body):
    print(body)
# basic_qos需要放在basic_consume前才有效
channel.basic_qos(prefetch_count=3)
channel.basic_consume(on_message_callback=callback,
                       queue='hello',
                      )
...
# rabbit_c2.py
...
def callback(ch, method, properties, body):
    print(body)
    # 主动使用ack告知调用方
    ch.basic_ack(delivery_tag=method.delivery_tag)
...

```

可以发现，消费者`1`只处理了三条数据(并且网页端显示一直是`unacked`)，其他的数据都由消费者`2`处理了。接着此时关闭消费者`1`，会发现这三条数据又被分配给消费者`2`所处理了。

### `Exchange`

**fanout**类型：

需要在消费者和生产者上面都加上对`exchange`的声明，两个消费者分别订阅了`hello1`与`hello2`队列，这两个队列都连向`hello_fanout`这个`exchange`：

```python 
# rabbit_c1.py
...
channel.exchange_declare(exchange='hello_fanout', exchange_type='fanout')
channel.queue_declare(queue='hello1')
channel.queue_bind(exchange='hello_fanout', queue='hello1', routing_key='hello.1')
...
# rabbit_c2.py 
...
channel.exchange_declare(exchange='hello_fanout', exchange_type='fanout')
channel.queue_declare(queue='hello2')
channel.queue_bind(exchange='hello_fanout', queue='hello1', routing_key='hello.2')
...
# rabbit_p.py
channel.exchange_declare(exchange='hello_fanout', exchange_type='fanout')
channel.queue_declare(queue='hello1')
channel.queue_declare(queue='hello2')

for i in range(20):
    body = '%s Hello rabbitmq' % i
    channel.basic_publish(exchange='hello_fanout',
                          routing_key='world',
                          body=body)

```

在`fanout`模式下，`routing_key`即使不匹配也没有关系。消费者`1`和消费者`2`的输出结果一样。

**direct**

将消费者修改为：

```python 
# rabbit_c1.py
...
# 修改exchange的type为direct
channel.exchange_declare(exchange='hello_direct', exchange_type='direct')
channel.queue_declare(queue='hello1')
# 指定路由规则
channel.queue_bind(exchange='hello_direct', queue='hello1', routing_key='hello.1')
...
channel.basic_consume(on_message_callback=callback,
                       queue='hello1',
                      )

...

```

生产者：

```python 
# rabbit_p.y
...
channel.exchange_declare(exchange='hello', exchange_type='direct')
channel.queue_declare(queue='hello1')
channel.queue_declare(queue='hello2')

for i in range(20):
    body = '%s Hello rabbitmq' % i
    if i < 10:
        channel.basic_publish(exchange='hello',
                              routing_key='hello.1',
                              body=body)
    else:
        channel.basic_publish(exchange='hello',
                              routing_key='hello.2',
                              body=body)
...

```

此时`i<10`的消息会由消费者`1`处理。

**topic**

`Binding Key`可以存在`*`与`#`，但是生产者的`Routing Key`应当是精确的。修改消费者与生产者分别为：

```python 
#rabbit_c1.py
channel.exchange_declare(exchange='hello_topic', exchange_type='topic')
channel.queue_declare(queue='hello1')
channel.queue_bind(exchange='hello_topic', queue='hello1', routing_key='hello.*')
...
#rabbit_c2.py 绑定了两种规则
channel.queue_bind(exchange='hello_topic', queue='hello2', routing_key='hello.*')
channel.queue_bind(exchange='hello_topic', queue='hello2', routing_key='*.hello.*')

#rabbit_p.py
...
channel.exchange_declare(exchange='hello_topic', exchange_type='topic')
for i in range(20):
    body = '%s Hello rabbitmq' % i
    if 10 < i < 20:
        channel.basic_publish(exchange='hello_topic',
                              routing_key='hello.2',
                              body=body)
    else:
        channel.basic_publish(exchange='hello_topic',
                              routing_key='2.hello.2',
                              body=body)
...


```

一个队列可以使用多种路由规则与同一`exchange`绑定。此时消费者`2`可以收到全部消息，而消费者`1`只能收到`11-19`之间的消息。

同样的，一个队列可以与多个`exchange`绑定，让消费者`2`订阅的队列同时与`hello_topic`和`hello_direct`相连：

```python 
# rabbit_c2.py
channel.exchange_declare(exchange='hello_topic', exchange_type='topic')
channel.exchange_declare(exchange='hello_direct', exchange_type='direct')
channel.queue_declare(queue='hello2')
channel.queue_bind(exchange='hello_direct', queue='hello2', routing_key='hello.2')
channel.queue_bind(exchange='hello_topic', queue='hello2', routing_key='hello.*')

# rabbit_p.py

channel.exchange_declare(exchange='hello_topic', exchange_type='topic')
channel.exchange_declare(exchange='hello_direct', exchange_type='direct')

for i in range(20):
    body = '%s Hello rabbitmq' % i
    if 10 < i < 20:
        channel.basic_publish(exchange='hello_topic',
                              routing_key='hello.1',
                              body=body)
    else:
        channel.basic_publish(exchange='hello_direct',
                              routing_key='hello.2',
                              body=body)

```

此时，消费者`2`可以收到全部消息。

### `exclusive`

前面测试的时候，如果不重启`RabbitServer`，如果没有删除交换机或者队列的话，以前的路由规则仍然存在，经常会出现匪夷所思的问题。这是因为持久化的问题，留到下节介绍。

实际上可以使用临时队列的方式进行处理，`exclusive`参数指明这个队列为排他性队列，即只有自己可以访问，此时生产者不能再进行生命，当连接断掉，这个`queue`会被删除(这点与`auto_delete`参数功能一致)：

```python 
# rabbit_c3.py
import pika
connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel = connection.channel()
channel.exchange_declare(exchange='hello_direct', exchange_type='direct')
# 如果queue为空，会自动创建一个名字唯一的队列，想要连接断开自动删除，可以使用exclusive，也可以使用auto_delete
# 队列名采用r.method.queue获取
r = channel.queue_declare(queue='', exclusive=True)
#r = channel.queue_declare(queue='', auto_delete=True)
channel.queue_bind(exchange='hello_direct', queue=r.method.queue, routing_key='hello.2')

def callback(ch, method, properties, body):
    print(body)
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(on_message_callback=callback,
                       queue=r.method.queue)

channel.start_consuming()

```

这样可以实现一个动态创建队列的消费者。

### 持久化

当将`RabbitMQ`停掉，会发现不论是`exchange`还是`queue`的所有信息都丢失了，因此如果需要恢复`exchange`和`queue`，需要在声明交换机和队列时就提供持久化参数(`durable=True`)：

```python 
# rabbit_c1.py
channel.exchange_declare(exchange='hello_topic', exchange_type='topic', durable=True)
channel.queue_declare(queue='hello1', durable=True)

```

需要注意的是，如果消费者声明了`durable=True`，那么生产者在再次声明时，也必须将其声明为`durable=True`，否则会报错。或者也可以删除后再次声明：

```python 
channel.queue_delete(queue='hello1')
channel.exchange_delete(exchange='hello_topic')

```



但是想要使得队列里的消息不丢失，需要额外在`basic_publish`时也指定消息的参数(`delivery_mode=2`)：

```python 
# rabbit_p.py
channel.basic_publish(exchange='hello_topic',
                      routing_key='hello.1',
                      properties=pika.BasicProperties(delivery_mode=2),
                      body=body)

```

这样从交换机、队列层面保证了消息只要进入了，就不会丢失，但是如果在投递的过程中丢失，比如消息并未到达交换机或者没有对应的队列(消息会被丢弃)，此时应当使用`RabbitMQ`提供的`confirm mode`。

以上测试是在最新的`pika==1.0.1`之下，但是这个版本下`channel.basic_publish`并没有返回了，所以切回`pika=0.13.1`下测试，接下来的环境都为`Python=3.7 pika=0.13.1`：

```python 
# rabbit_p.py
import pika
connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel = connection.channel()
channel.exchange_declare(exchange='test', exchange_type='fanout')
channel.queue_declare(queue="test", durable=True, exclusive=False, auto_delete=False)

# 开启确认
channel.confirm_delivery()
res = channel.basic_publish(exchange='test',
                         routing_key='test',
                         body='Hello World!',
                         properties=pika.BasicProperties(content_type='text/plain',
                                                         delivery_mode=2),
                            mandatory=True)

```

此时由于没有消费者绑定名为`test`的`exchange`，因此会返回`False`。

> 每一个版本的`pika`函数参数都有很大不同T_T

`confirm mode`的确认结果表示，一条`persisting`的消息投向`durable`队列成功，并且成功写到磁盘。

### `basic_reject`

与`basic_ack`相对应的是`basic_reject`，它可以拒绝一条消息，如果要拒绝多条消息，使用`basic_nack`：

```python 
# rabbit_c1.py
def callback(ch, method, properties, body):
    global count
    if count % 2 == 0:
        ch.basic_reject(delivery_tag=method.delivery_tag)
    else:
        print(body)
        ch.basic_ack(delivery_tag=method.delivery_tag)
    count += 1

```

默认情况下，`requeue=True`，表示消息被拒绝后可以由其他消费者处理。

### `RPC`调用

首先定义生产者端：

```python 
import pika
import uuid

conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
ch = conn.channel()
ch.exchange_declare(exchange='rpc_p', exchange_type='fanout')
ch.exchange_declare(exchange='rpc_r', exchange_type='direct')
ch.queue_declare(queue='rpc_p')
ch.queue_declare(queue='rpc_r')
ch.queue_bind(exchange='rpc_p', queue='rpc_p')
ch.queue_bind(exchange='rpc_r', queue='rpc_r')


def callback(channel, method, properties, body):
    print(body, properties.correlation_id)
    channel.basic_ack(delivery_tag=method.delivery_tag)

ch.basic_consume(callback, queue='rpc_r')
correlation_id = uuid.uuid4().hex
print(correlation_id)
ch.basic_publish(exchange='rpc_p',
                 routing_key='',
                 body='1,2,3,4',
                 properties=pika.BasicProperties(
                     reply_to='rpc_r',
                     correlation_id=correlation_id
                 ))
ch.start_consuming()

```

生产者指定自己发出消息的交换机为`rpc_p`，需要接受回复使用的交换机为`rpc_r`，并分别绑定队列`rpc_p`和`rpc_r`，在发布消息时，指定`reply_to`参数以及标识这条消息的唯一`id`。

消费者：

```python 
import pika

conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
ch = conn.channel()
ch.exchange_declare(exchange='rpc_r', exchange_type='direct')
ch.exchange_declare(exchange='rpc_p', exchange_type='fanout')
ch.queue_declare(queue='rpc_p')
ch.queue_bind(exchange='rpc_p', queue='rpc_p')

def callback(channel, method, properties, body):
    print(body)
    body = body.decode('utf-8')
    s = sum(int(x) for x in body.split(','))
    channel.basic_publish(exchange='rpc_r',
                          routing_key=properties.reply_to,
                          body=str(s),
                          properties=pika.BasicProperties(
                              correlation_id=properties.correlation_id
                          ))
    channel.basic_ack(delivery_tag=method.delivery_tag)

ch.basic_consume(callback, queue='rpc_p')
ch.start_consuming()

```

消费者只需要一直监听来自队列`rpc_p`的消息，并且在接收到消息后将确认信息发送至`reply_to`所指向的队列，同时指明自己所收到的是哪条消息。

输出结果：

```python 
# 生产者
7bfd8db775144308945b679ed4c5a2ed
b'10' 7bfd8db775144308945b679ed4c5a2ed
# 消费者
b'1,2,3,4'

```



`References`：

[RabbitMQ 使用参考](<https://www.zouyesheng.com/rabbitmq.html#toc11>)

[消息队列之 RabbitMQ](<https://www.jianshu.com/p/79ca08116d57>)

[RabbitMq的整理 exchange、route、queue关系](<https://my.oschina.net/wy08/blog/186202>)

[一篇全面透彻的RabbitMQ指南！](<https://juejin.im/entry/599e5e3b5188252437799049>)



