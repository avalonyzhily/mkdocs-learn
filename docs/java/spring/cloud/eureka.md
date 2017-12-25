### spring cloud eureka注意事项：
1. 作为client一定要配置spring.application.name,否则会影响其他消费端获取本服务(spring中会使用VirtualHostName对应eureka instanceInfo的vipAddress,这一值由spring.application.name确定)

### spring cloud eureka 组件对应的配置类：
1. eureka server---org.springframework.cloud.netflix.eureka.server.EurekaServerConfigBean
2. eureka instance---org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean
3. eureka client---org.springframework.cloud.netflix.eureka.EurekaClientConfigBean