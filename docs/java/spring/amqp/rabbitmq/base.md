### 使用rabbitmq的步骤
生产者端
```
创建连接(connection)
创建channel
创建队列(queue)——不论是生产者还是消费者,都可以创建队列
选择要发送的exchage,并告诉exchange指定的route_key(即queue的名字),以及要发送的消息体
关闭连接
```

消费者端
```
创建连接(connection)
创建channel
创建队列(queue)——不论是生产者还是消费者,都可以创建队列
定义接收到消息的回调方法
开启消息监听
```

### acknowledgment机制
为了保证数据不被丢失，RabbitMQ支持消息确认机制，即acknowledgments。

默认是打开的,可以通过配置关闭

注意！如果没有配置超时机制,RabbitMQ是通过consumer的中断来确认消息是否正确处理;没有正确处理的消息会发送给下一个consumer;但是如果忘记ack,consumer退出后消息会不停重发,导致内存消耗越来越多

### 消息持久化
为了保证在RabbitMQ退出或者crash了数据仍没有丢失，需要将queue和Message都要持久化。

queue的持久化可以在创建队列时声明durables(持久化)属性为true
>注意,已经在MQ中存在的queue,即使再次声明为持久化,其属性也不会改变,只能重新创建新的持久化的队列

Message的持久化可以在发布消息时,增加消息配置delivery_mode(投递模式)属性为2(持久化)

>以上是python pika库中的设置,java或其他语言中可能有出入,但原理是一致的

为了进一步的保证持久化的一致性,还需要额外的用户自己的代码来实现,来保证数据不丢失。

>注意:rabbitmq本身并不直接支持消息持久化到数据库,如有需要则另行编码实现(在生产侧),或选用ActiveMQ这一款消息队列,本身提供了持久化到数据库的支持

### 公平分发 
使用prefetch_count来保证每个consumert同一时间至多只处理多少个消息,除非consumer做了消息确认(ack)

### connection and channel
Connection是真实的tcp连接,连接到rabbitmq server

channel则是tcp连接之上的虚拟连接,目的是为了避免tcp连接打开关闭的开销,同时可以增加数据传输的效率

#### Rabbit RPC这个部分不是很明白是怎么回事！
>以上笔记摘自http://blog.csdn.net/column/details/rabbitmq.html

### virtual host user
virtual host(vhost)相当于数据库    用户相当于账号密码

### spring-rabbit

- Connection

    - 默认提供了```CachingConnectionFactory```工厂类来创建连接,现在的版本可以创建可缓存的连接(默认可以缓存通道),同时可以设置缓存连接的上限,缓存连接模式不支持自动创建MQ的那些组件,同时缓存连接需要考虑设置合适的Executor的连接池,如果想自定义一个```ConnectionFactory```可以从```AbstractConnectionFactory```这个抽象类入手;
    - 创建模式默认是```CHANNEL```,即缓存通道,也可以修改为```CONNECTION```,即缓存连接,注意低版本不一定支持
    - 关于恢复连接的机制(MQ集群,连接地址配置多个),Rabbitmq的客户端默认提供了恢复机制,为了兼容通用模型,Spring-amqp也提供了,但是最好使用Spring-amqp,否则会出现异常;在高版本中,Spring-amqp会默认关闭Rabbitmq本身的恢复机制,除非显式的自己声明Rabbitmq的连接工厂放入```CachingConnectionFactory```来构建实例,通过```RabbitConnectionFactoryBean```创建的Rabbitmq连接工厂实例同样也是关闭Rabbitmq本身的恢复机制的;
    - 通过```RabbitConnectionFactoryBean```可以方便的配置SSL信息,然后提供给```ConnectionFactory```来生成带SSL的连接,其中通过```useSSL```开启SSL模式,通过```sslPropertiesLocation```来加载相关的配置;
    - ```AbstractRoutingConnectionFactory```路由选择连接的机制,只知道是根据lookupKey来选择要使用的连接,通过RabbitTemplate在使用的时候bind连接,用完之后需要unbind,苦于语言原因暂时不是特别懂(看的官方文档)
    - 只有传入```AbstractRoutingConnectionFactory```实例时才会开启路由机制,如果没有有效的lookupKey,或是lookupKey没有对应的连接目标实例,当开启```lenientFallback = true```的回退许可时,会不启动路由,否则会抛出异常;(暂时这么理解)
