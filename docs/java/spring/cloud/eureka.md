### spring cloud eureka注意事项：
- 作为client一定要配置spring.application.name,否则会影响其他消费端获取本服务(spring中会使用VirtualHostName对应eureka instanceInfo的vipAddress,这一值由spring.application.name确定)

### spring cloud eureka 组件对应的配置类：
- eureka server---org.springframework.cloud.netflix.eureka.server.EurekaServerConfigBean
- eureka instance---org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean
- eureka client---org.springframework.cloud.netflix.eureka.EurekaClientConfigBean

### eureka server 存储注册进来的服务的方式：
- 使用了一个ConcurrentHashMap来存放,分为两层
- 第一层的key是服务名,即instanceInfo里面的appName属性
- 第二层的key是服务的实例名,即instanceInfo里面的instanceId属性

### eureka client 注册的过程
- 判断是否需要注册,启动注册的定时任务和续约的定时任务
- 判断是否需要获取服务,启动获取服务的定时任务

### spring cloud eureka的健康检查机制
- 客户端本身的心跳包机制
- 还可以依赖actuator包提供的healthcheck端点,即默认的健康检查url