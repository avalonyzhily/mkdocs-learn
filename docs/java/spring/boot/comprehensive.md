### Spring boot加载配置的顺序(优先级由高到低):
1. 在命令行中输入的参数;
2. SPRING_APPLICATION_JSON中的属性。即以JSON格式配置在系统环境变量中的内容。
3. java:comp/env中的JNDI属性;
4. java的系统属性,可以通过System.getProperties()获取;
5. 操作系统的环境变量
6. 通过random.*配置的随机属性
7. 当前应用jar之外,针对不同profile环境的配置文件;
8. 当前应用jar之内,针对不同profile环境的配置文件;
9. 当前应用jar之外,默认的application.properties/yml的配置文件;
10. 当前应用jar之内,默认的application.properties/yml的配置文件;
11. 在@Configuration注解修改的类中，通过@PropertySource注解定义的属性
12. 应用默认属性，使用SpringApplication.setDefaultProperties 定义的内容