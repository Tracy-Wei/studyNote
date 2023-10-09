### RabbitMQ:

它的并发能力强，性能极好，延迟极低，稳定性和安全性很高，同时还支持集群。RabbitMQ 在分布式系统开发中应用非常广泛，是最受欢迎的开源消息中间件之一。

#### RabbitMQ 基于 AMQP 的开源消息代理软件

1.AMQP 是什么?

AMQP(Advanced Message Queuing Protocol,高级消息队列协议)是进程之间传播异步消息的网络协议

2.AMQP 工作过程

发布者(Publisher)发布消息(Message),经过交换机(Exchange),交换机根据路由规则将收到消息分发给交换机绑定的队列(Queue),最后 AMQP 代理会将消息投递给订阅了此队列的消费者,或者消费者按照需求自行获取.

AMQP 实体后面就是 RabbitMQ

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/8ad63dcb-8d87-4d53-998b-33501266afa8)

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/135c2dfb-e773-42c8-92e2-81e74eca52da)

1. Publisher：消息生产者，就是投递消息的程序，也是一个向交换机发布消息的客户端应用程序。生产者发送的消息一般包含两个部分：消息体和内容标签。
2. Consumer：消息消费者，也就是接收消息的一方。消费者连接到 RabbitMQ 服务器，并订阅到队列上。消费消息时只消费消息体，丢弃标签。
3. Connection ：网络连接，比如一个 TCP 连接，用于连接到具体 Broker。
4. Channel： 信道，AMQP 命令都是在信道中进行的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接，一个 TCP 连接可以用多个信道。客户端可以建立多个 channel，每个 channel 表示一个会话任务。
5. Broker 服务节点：表示消息队列服务器实体。一般情况下一个 Broker 可以看做一个 RabbitMQ 服务器。
6. Exchange：交换器，接受生产者发送的消息，根据路由键将消息路由到绑定的队列上。
   > Exchange 分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct、fanout、topic、headers，由于 headers 交换器和 direct 交换器完全一致，且性能差很多，目前几乎用不到。这里只看 direct、fanout、topic 这三种类型：
   >
   > 1. direct（直连）：消息中的路由键（RoutingKey）如果和 Bingding 中的 bindingKey 完全匹配，交换器就将消息发到对应的队列中。是基于完全匹配、单播的模式。
   > 2. fanout（广播）：把所有发送到 fanout 交换器的消息路由到所有绑定该交换器的队列中，fanout 类型转发消息是最快的。
   > 3. topic（主题）：通过模式匹配的方式对消息进行路由，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。匹配规则：
   >    ① RoutingKey 和 BindingKey 为一个 点号 '.' 分隔的字符串。 比如: stock.usd.nyse；可以放任意的 key 在 routing_key 中，当然最长不能超过 255 bytes。
   >    ② BindingKey 可使用 * 和 # 用于做模糊匹配：*匹配一个单词，#匹配 0 个或者多个单词；
   > 4. headers：不依赖于路由键进行匹配，是根据发送消息内容中的 headers 属性进行匹配，除此之外 headers 交换器?和 direct 交换器完全一致，但性能差很多，目前几乎用不到了。
7. Queue：消息队列，用来存放消息。一个消息可投入一个或多个队列，多个消费者可以订阅同一队列，这时队列中的消息会被平摊（轮询）给多个消费者进行处理。
8. Message：消息，由消息体和标签组成。消息体是不透明的，而标签则由一系列的可选属性组成，这些属性包括 routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。
9. Routing Key： 路由关键字，用于指定这个消息的路由规则，需要与交换器类型和绑定键(Binding Key)联合使用才能最终生效。
10. Binding：绑定，通过绑定将交换器和队列关联起来，一般会指定一个 BindingKey，通过 BindingKey，交换器就知道将消息路由给哪个队列了。
11. Virtual host：虚拟主机，用于逻辑隔离，表示一批独立的交换器、消息队列和相关对象。一个 Virtual host 可以有若干个 Exchange 和 Queue，同一个 Virtual host 不能有同名的 Exchange 或 Queue。最重要的是，其拥有独立的权限系统，可以做到 vhost 范围的用户控制。当然，从 RabbitMQ 的全局角度，vhost 可以作为不同权限隔离的手段。

3.队列

对垒是数据结构中概念.数据存储在一个队列中,数据是由顺序的,先进先出的,后进后出.其中一侧负责进数据,另一侧负责出数据

MQ(消息队列)很多功能都是基于次队列结构实现的,**队列也是解决高并发下排队问题的解决方案**,即使恰巧出现同一时刻向队列添加的两个数据,也会有 CPU 帮助判断,放入到队列后,一定会先后顺序.

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/b4e52cd1-51d0-4544-bea6-bd3552890672)

#### RabbitMQ 简介

##### RabbitMQ 介绍

RabbitMQ 是由 Erlang 语言编写的基于 AMQP 的消息中间件.而消息中间件作为分布式系统重要组件之一,可以解决应用耦合,异步消息,流量削锋等问题.

解决应用耦合

1.  不使用 MQ 时：
    应用程序 A 向应用程序 B 发送消息时,A 必须知道 B 的相关消息,当 B 的位置等信息改变时,A 中数据也需要跟随修改.这个时候称为 A 和 B 是耦合的.

    **耦合**：
    ![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/452cf099-f6d4-492d-a0f6-3db0b009b7e6)

2.  使用 MQ 解决耦合：
    应用程序 A 发送消息只需要向 MQ 发送,不需要管应用程序 B.
    应用程序 B 只需监听 MQ 中消息,有消息取出即可,不需要管应用程序 A.
    此时认为应用程序 A 和应用程序 B 是解耦,它们两个没有直接联系了.

    **解耦**：
    ![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/12912fc2-dfff-45eb-b583-82cc5f02e5d1)

##### RabbitMQ 适用场景

1. 两大核心特性

RabbitMQ 两大核心特性:异步消息,队列

异步消息:只要异步消息就不阻塞线程,减少了主线程执行时间.所有需要这种效果场景都可以使用 MQ.

队列:进入队列的数据一定后先后之分.只要应用程序要对内容分先后的场景都可以使用 MQ.

2. 具体场景

- 排队算法

使用队列特性.把数据发送给 MQ,进入到队列就有了排队的效果

- 秒杀活动

使用队列特性.例如:抢红包,限时秒杀,直播卖货时抢商品.使用了 MQ 按照顺序一个一个操作,当商品库存操作到 0 个时,秒杀结束

- 消息分发

在程序中同时向多个其他程序发送消息.应用了 AMQP 中交换机,实现消息分发

- 异步处理

利用 MQ 异步消息特性.大大提升主线程效率

- 数据同步

利用异步特性.我们电商中使用 RabbitMQ 绝大多数的事情即使在实现数据同步

- 处理耗时任务

利用异步特性.可以把程序中耗时任务(例如:发送邮件,发送验证码)交给 MQ 去处理,减少当前项目的耗时时间.

- 流量削峰

在互联网项目中,可能会出现某一段时间范围内,访问流量骤增的情况(双 11,品牌促销,10 点抢购),如果使用使用监控工具,会发现这段时间访问出现顶峰.使用 MQ 可以把这些访问分摊到多个项目中,把流量分摊,祛除了顶峰效果,这就是流量削峰.

利用 RabbitMQ 中交换机实现的

##### RabbitMQ 执行原理文字解释:

客户端应用程序向 RabbitMQ 发送消息 Message,在 Message 会包含路由键 Routing Key,交换器 Exchange 接收到消息 Message 后会根据交换器类型 ExchangeType 解决把消息如何发送给绑定的队列 Queue 中,如果交换器类型是 Direct 这个消息只放入到路由键对应的队列中,如果是 topic 交换器消息放入到 routing key 匹配的多个对列中,如果是 fanout 交换器消息会放入到所有绑定到交换器的对列中.等放入到队列中之后 RabbitMQ 的事情就结束了.剩下的事情是由 Consumer 进行完成,Consumer 一直在监听对列,当对列里面有消息就会把消息取出,取出后根据程序的逻辑对消息进行处理.以上这些就是 RabbitMQ 的运行原理.

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/1a8c8aa9-28e1-4245-ae6d-4f3e9551a60c)

注意：由于 RabbitMQ 是采用 erlang 语言开发的，所以必须有 erlang 环境才可以运行

官网： http://www.rabbitmq.com/
官方教程：http://www.rabbitmq.com/getstarted.html

##### [RabbitMQ 提供了 6 种模式][1]：

1. 简单模式：一对一模式，只有一个生产者，一个队列，一个消费者，是最简单的模式。
2. 工作队列模式：一对多模式，一个消息生产者，一个消息队列，多个消费者。
3. Publish/Subscribe 发布与订阅模式：无选择接收消息，一个消息生产者，一个交换器，多个消息队列，多个消费者。
4. Routing 路由模式：在发布/订阅模式的基础上，有选择的接收消息，也就是通过 routing 路由进行匹配条件是否满足接收消息。
5. Topics 主题模式：同样是在发布/订阅模式的基础上，根据主题匹配进行筛选是否接收消息，比第四类更灵活（也是最常用的一种）。
6. RPC 远程调用模式（远程调用，不太算 MQ；不作介绍）

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/326903ed-9b9b-48d9-898f-6d376c46639a)

##### -> [使用参考][2]

[1]: https://www.rabbitmq.com/getstarted.html
[2]: https://blog.csdn.net/wulex/article/details/123319032
