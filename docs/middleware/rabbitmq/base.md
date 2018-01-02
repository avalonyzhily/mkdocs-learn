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

### 公平分发 
使用prefetch_count来保证每个consumert同一时间至多只处理多少个消息,除非consumer做了消息确认(ack)

### exchanges, queues and bindings 的概念
exchanges 是消息生产者投消息的地方
queues 是消息消费者取消息的地方
bindings 是获取exchanges 到指定的queues的路由(route)的地方

### connection and channel
Connection是真实的tcp连接,连接到rabbitmq server

channel则是tcp连接之上的虚拟连接,目的是为了避免tcp连接打开关闭的开销,同时可以增加数据传输的效率

### exchange route_key queue bindings
三种模式:direct(路由), topic(主题) 和fanout(广播)

direct:routeKey的完全匹配(即生产方指定的routeKey和消费方的routeKey完全一样),一个队列可以监听多个routeKey(同一个queue多次bind不同的routeKey),多个队列也可以监听一个routeKey(多个queue绑定同一个routeKey)

fanout:忽略routeKey的匹配,将消息发送到所有bind的队列中去(不论是否显式指定routeKey)

topic:主题模式的routeKey可以使用正则模式进行匹配,如果仅使用*或#,则类似fanout接受所有的消息;如果明确指定routeKey字符串,则类似direct。
>注意,同一条消息的对于同一个queue,即使消息routeKey匹配到2个监听的routeKey,也只会发送一次,不会重复。

queue和exchange通过binding进行关联,这是channel的一个queue_bind方法,提供exchange的名字和队列的名字,以及routeKey,即可进行绑定。

routeKey用于指定消息通过exchange(一般是默认)发送到哪个queue,不显式的指定则与queue的名字相同,这个前提是exchange与queue进行了绑定,因为默认的exchange是默认存在，且与所有队列绑定的,因此不用显式的声明和绑定;如果使用了没有绑定queue的exchange,则发送的消息会丢失;routeKey的匹配机制与exchange的模式有关系。

routeKey的格式(topic模式),以"."号分割字符,其中有2个特殊字符——"*"代表任意一个单词,"#"代表任意的0个或多个单词。

队列声明时exclusive参数为True,代表是排他队列,当该队列对应的Consumer关闭时,排他队列会删除

如果使用exchange向多个queue发送消息,则可以不用再生产侧进行queue的声明,生产方只需要定义exchange,在消费侧要进行exchange、queue的声明以及与exchange的绑定,这样就能一次消息发送到多个队列了。
