---
typora-copy-images-to: img
---

# rabbitMQ笔记
## 相关概念
使用消息队列的原因

不同进程（process）之间传递消息时，两个进程之间耦合程度过高，改动一个进程，引发必须修改另一个进程，为了隔离这两个进程，在两进程间抽离出一层（一个模块），所有两进程之间传递的消息，都必须通过消息队列来传递，单独修改某一个进程，不会影响另一个；

不同进程（process）之间传递消息时，为了实现标准化，将消息的格式规范化了，并且，某一个进程接受的消息太多，一下子无法处理完，并且也有先后顺序，必须对收到的消息进行排队，因此诞生了事实上的消息队列；

rabbitMQ是使用erlang开发的一种基于AMQP协议的消息系统


+ Broker:它提供一种传输服务,它的角色就是维护一条从生产者到消费者的路线，保证数据能按照指定的方式进行传输,
+ Exchange：消息交换机,它指定消息按什么规则,路由到哪个队列。 
+ Queue:消息的载体,每个消息都会被投到一个或多个队列。 
+ Binding:绑定，它的作用就是把exchange和queue按照路由规则绑定起来. 
+ Routing Key:路由关键字,exchange根据这个关键字进行消息投递。 
+ vhost:虚拟主机,一个broker里可以有多个vhost，用作不同用户的权限分离。 
+ Producer:消息生产者,就是投递消息的程序. 
+ Consumer:消息消费者,就是接受消息的程序. 
+ Channel:消息通道,在客户端的每个连接里,可建立多个channel.

![架构](.\img\架构.png)

RabbitMQ队列模型


简单队列:   一个生产者对应一个消费者

work模式:  简单队列只要消息从队列中获取，无论消费者获取到消息后是否成功消费，比如遇到状况：断电，都认为是消息已经成功消费；work模式消费者从队列中获取消息后，服务器会将该消息标记为不可用状态，等待消费者反馈，如果消费这一直没有反馈，则该消息一直处于不可用状态。channel.basicQos(1); 能者多劳，否则平均分配


## 消息分发机制


## 简单队列:   
一个生产者对应一个消费者，简单队列只要消息从队列中获取，无论消费者获取到消息后是否成功消费，比如遇到状况：断电，都认为是消息已经成功消费。

## work模式:  
work模式消费者从队列中获取消息后，服务器会将该消息标记为不可用状态，等待消费者反馈，如果消费这一直没有反馈，则该消息一直处于不可用状态。channel.basicQos(1); 能者多劳，否则平均分配

## 发布订阅模式
一个消费者将消息首先发送到交换器，交换器绑定到多个队列，然后被监听该队列的消费者所接收并消费。

## 交换器类型
+ **fanout**

    fanout类型的Exchange路由规则非常简单，它会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中。生产者发送到Exchange的所有消息都会路由到绑定的Queue，并最终被两个消费者消费。
+ **direct**

    direct类型的Exchange路由规则也很简单，它会把消息路由到那些binding key与routing key完全匹配的Queue中。（在实际使用RabbitMQ的过程中并没有binding key这个参数，只有routing key，为了区分我们把交换机和队列绑定时传的参数叫binding key，把发送消息时带的这个参数叫routing key）
+ **topic**

  前面讲到direct类型的Exchange路由规则是完全匹配binding key与routing key，但这种严格的匹配方式在很多情况下不能满足实际业务需求。topic类型的Exchange在匹配规则上进行了扩展，它与direct类型的Exchage相似，也是将消息路由到binding key与routing key相匹配的Queue中，但direct是完全匹配，而通过topic可以进行模糊匹配