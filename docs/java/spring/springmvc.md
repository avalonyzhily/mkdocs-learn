### SpringMVC中的DispatcherServlet在J2EE规范中初始化的默认行为

```DispatcherServlet``` 初始化后,SpringMVC会默认去WEB-INF下寻找名为```[servlet-name]-servlet.xml```的配置文件,该文件内定义了```DispatcherServlet``` 对应上下文(即```WebApplicationContext```)内的bean定义,
这个文件位置也可以通过初始化参数进行修改,即通过```init-param```标签修改```contextConfigLocation```的值

如果没有该配置文件,则默认是Root WebApplicationContext,即只有单一的上下文。