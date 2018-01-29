### logstash的配置
- 有两种类型的配置文件
    - pipeline configuration files 用来配置logstash的执行逻辑过程 包含了input filter output等各个部分的执行逻辑,对于Linux系统下(deb和rpm),logstash只会去读取/etc/logstash/conf.d这个目录下以conf作为后缀的文件,其他的都会忽略
        - input 收集数据阶段的配置,可以配置各种插件来收集数据(beats,file,redis等),可以配置多种方式
        - filter 过滤器是对收集进来的数据进行转换,过滤等一系列操作的配置,可以配置多个处理流程
        - output 将处理后的数据输入,也可以配置多个输出端
    - settings files 用来配置logstash本身的一些设置,startup.options是Linux下的特有配置,方便创建多实例的logstash服务,该文件不是只读文件,但是修改之后需要重启logstash来更新配置
        - ```queue.max_events```和```queue.max_bytes```同时设置时,logstash优先使用先满足的条件。