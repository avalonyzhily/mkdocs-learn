### Feign和Ribbon的关系:

1. Ribbon是Netflix提供的开源组件，用于客户端负载控制；Feign对Ribbon进行了封装，采用声明式的方式来构建服务接口。

2. 与通过spring-cloud与eureka集成后，配置项中使用Ribbon的超时、重试等配置有效，但是裸Ribbon时，无效，原因目前不明。

3. 注意！以上说明基于SpringCloud.Dalston.SR4版本，早期版本可能需要手动引入Spring-Retry依赖。



### Feign与Hystrix：

1. 各自有各自的超时时间。
2. 对于首次请求时间超过了Hystrix的超时时间，则会直接熔断触发fallback方法，完成请求响应。
3. 对于Feign来说，根据Ribbon的设置，超时后先对当前服务实例进行重试，再次失败后，切换服务实例重试，所有重试的实例都失败后，触发Hystrix中的fallback方法，完成请求响应。
4. 注意！当Hystrix的超时时间设置过短，导致Feign的重试未完成或根本没有重试时，会直接触发fallback，但是并不会阻断Feign的行为，只不过后续重试拿到的结果不会再响应，也不会触发fallback。


