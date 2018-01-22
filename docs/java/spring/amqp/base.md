# Spring-AMQP 要点
PS：以下各组件都是Java代码中对AMQP规范中各组件的映射,所以定义了组件的实例,并不一定就在MQ中实际创建,可能还需要行为触发

另外,spring-amqp中只是定义了通用的代理模型,这些都是抽象的,实际的实现则由具体的中间件来完成,当前继承的是spring-rabbit
## exchanges, queues and bindings 的概念
- exchanges 是消息生产者投消息的地方
- queues 是消息消费者取消息的地方
- bindings 是获取exchanges 到指定的queues的路由(route)的地方

## exchange route_key queue bindings
四种模式:direct(路由), topic(主题) ,fanout(广播),headers
- direct:routeKey的完全匹配(即生产方指定的routeKey和消费方的routeKey完全一样),一个队列可以监听多个routeKey(同一个queue多次bind不同的routeKey),多个队列也可以监听一个routeKey(多个queue绑定同一个routeKey)
- fanout:忽略routeKey的匹配,将消息发送到所有bind的队列中去(不论是否显式指定routeKey)
- topic:主题模式的routeKey可以使用正则模式进行匹配,如果仅使用*或#,则类似fanout接受所有的消息;如果明确指定routeKey字符串,则类似direct。
>注意,同一条消息的对于同一个queue,即使消息routeKey匹配到2个监听的routeKey,也只会发送一次,不会重复。
- headers:Exchange不处理routeKey(路由键),而是通过消息中的headers属性进行匹配,在绑定exchange和queue时,设置一组键值对，
- queue和exchange通过binding进行关联,这是channel的一个queue_bind方法,提供exchange的名字和队列的名字,以及routeKey,即可进行绑定。
- routeKey用于指定消息通过exchange(一般是默认)发送到哪个queue,不显式的指定则与queue的名字相同,这个前提是exchange与queue进行了绑定,因为默认的exchange是默认存在，且与所有队列绑定的,因此不用显式的声明和绑定;如果使用了没有绑定queue的exchange,则发送的消息会丢失;routeKey的匹配机制与exchange的模式有关系。
- routeKey的格式(topic模式),以"."号分割字符,其中有2个特殊字符——"*"代表任意一个单词,"#"代表任意的0个或多个单词。
- 队列声明时exclusive参数为True,代表是排他队列,当该队列对应的Consumer关闭时,排他队列会删除
- 如果使用exchange向多个queue发送消息,则可以不用再生产侧进行queue的声明,生产方只需要定义exchange,在消费侧要进行exchange、queue的声明以及与exchange的绑定,这样就能一次消息发送到多个队列了。
- Spring-amqp提供了BindingBuilder工具类可以方便的创建binding;Spring-amqp中binding的实例定义后,还需要使用才能触发绑定行为;
### Message properties
AMQP 预定义了14个属性。它们中的绝大多很少会用到。以下几个是平时用的比较多的：
- delivery_mode: 持久化一个Message（通过设定值为2）。其他任意值都是非持久化。
- content_type: 描述mime-type 的encoding。比如设置为JSON编码：设置该property为application/json。
- reply_to: 一般用来指明用于回调的queue（Commonly used to name a callback queue）。
- correlation_id: 在请求中关联处理RPC响应（correlate RPC responses with requests）。

### 异步接受消息
- 实现MessageListener进行消息监听,
- 基于AMQP协议的可以实现ChannelAwareMessageListener进行消息监听,并提供了扩展的参数Channel进行额外的操作,
- 使用MessageListenerAdapter适配器,则可以严格的分离应用逻辑和消息处理的api。

### 消息转换
- 消息监听器中的消息转换分两步：
    - 第一步是常规的消息转换(用的比较多),
    - 第二步是方法参数的转换(但是通常不会用)
- 消息的转换通常不推荐使用java的序列化,而是推荐使用JSON对象来进行转换,这样就保证了语言的无关性(Spring-amqp则推荐使用```Jackson2JsonMessageConverter```来转换消息,用于替换默认的```SimpleMessageConverter ```)
    - ```Jackson2JsonMessageConverter```可以配置自定义的```ClassMapper```,但是也配置了默认的```DefaultClassMapper```,```DefaultClassMapper```通过```MessageProperties```中的类型信息来完成类型转化,但是如果其中没有类型信息,但是你知道该消息的预期类型,则可以配置```DefaultClassMapper```的```defaultType```属性。

### 消息的响应
- Spring中提供了一个@SendTo的注解来设置消费者消费消息之后的返回值返回的默认地址
    - @SendTo的值是一个exchange/routeKey的组合,两项都是非必填
    - @SendTo的值支持SpEL表达式(#{...}是变量引用,只在初始化时执行一次)。。。而且支持运行时的表达式(使用 !{....}的形式,用@+bean的名字来访问bean),可从以下几个配置中读取：
        - request - ```the o.s.amqp.core.Message request object.```
        - source - ```the o.s.messaging.Message<?> after conversion.```
        - result - ```the method result.```

### 消息监听器相关
- 注解创建消息监听器不会注册到应用的上下文,但是可以通过```RabbitListenerEndpointRegistry```这个bean来获取;
- ```RabbitListenerEndpointRegistry```的```getListenerContainers()```方法或```getListenerContainer(String id)```分别可以获取所有的监听器或某个id的监听器
- ```RabbitListenerEndpointRegistry```的```getListenerContainerIds()```方法可以获取监听器的id;
- 监听器还可以进行分组

### mandatory 和 immediate
1. mandatory标志位
当mandatory标志位设置为true时，如果exchange根据自身类型和消息routeKey无法找到一个符合条件的queue，那么会调用basic.return方法将消息返还给生产者；当mandatory设为false时，出现上述情形broker会直接将消息扔掉。

2. immediate标志位
当immediate标志位设置为true时，如果exchange在将消息route到queue(s)时发现对应的queue上没有消费者，那么这条消息不会放入队列中。当与消息routeKey关联的所有queue(一个或多个)都没有消费者时，该消息会通过basic.return方法返还给生产者。

总结：
- mandatory标志告诉服务器至少将该消息route到一个队列中，否则将消息返还给生产者;
- immediate标志告诉服务器如果该消息关联的queue上有消费者，则马上将消息投递给它，如果所有queue都没有消费者，直接把消息返还给生产者，不用将消息入队列等待消费者了。

### AMQP还有实现RPC的能力
- client使用```AmqpProxyFactoryBean```
    - 通过```serviceInterface```输入任何bean(使用接口的形式),然后调用获取结果,该接口的实际实现在server端
- sever端使用```AmqpInvokerServiceExporter```
    - 同样使用```serviceInterface```属性来配置bean的接口,同时要通过```service```属性配置接口的实现
通过上面的方式可以完成RPC的实现(像本地方法一样调用远程的方法)

### 临时队列的生成
- 推荐使用```AnonymousQueue```来创建唯一的排他的自动删除的队列,这样比让broker去创建(比如传入""字符串作为队列名)更好,更具有可控性


### amqp事务
- spring-rabbit框架支持自动的事务管理,其中有2个方式将事务语义提供给框架
    - 不论是那种方法,都要注意channelTransacted设置的开启和关闭,决定外部事务是否生效
    - rabbitTemplate,在同步的场景中,只要在对rabbitTemplate的各种api进行调用的方法上添加事务注解,即可启用外部事务,统一管理方法内各种操作的提交和回滚(不论是MQ的还是database的);当出现异常时,接收的消息会返回到broker中去,要发送的消息也不会发送;
    - SimpleMessageListenerContainer,异步的场景中,在构建消息监听器时,传入PlatformTransactionManager接口的实现,同时开启channelTransacted设置,这样就可以通过外部事务进行统一的提交和回滚;;当出现异常时,接收的消息会返回到broker中去,要发送的消息也不会发送。这里要注意,如果channelTransacted设置为false,那么对于MQ来说则不会进行事务操作(会自动ack),但是业务操作会被回滚(譬如数据库写)
    - 这也是1阶段提交的一个最佳实现,是强大的可靠消息模式。
- 对于官网文档苦于语言障碍可能还需要细致正确的理解。
