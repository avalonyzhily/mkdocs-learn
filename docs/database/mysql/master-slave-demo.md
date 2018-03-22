### mysql的主从复制

- 基于binlog,保证主动库版本一致性
- 主服务器配置
  - 主服务器上创建复制使用账户,授予 REPLICATION SLAVE ,并设置可以使用从服务器的ip地址进行访问
      ```
      GRANT REPLICATION SLAVE ON *.* TO 'USER'@'IP' IDENTIFIED BY 'XXX'
      ```
  - 修改主服务器的配置文件my.cnf,开启binlog,设置server-id,然后重启生效
      ```
      [mysqld]
      log-bin = /xxx/xxx/xxx-bin.log #binlog保存文件
      server-id = xx #服务器唯一id值
      ```
  - 主服务器设置读锁有效,确保没有数据库操作,保证一致性快照的获得
      ```
      mysql>flush tables with read lock
      ```
  - 获取主服务器上二进制日志名(File字段)和偏移量(Position字段),让从库启动后从这个点开始复制
    ```
      mysql>show master status
    ```
  - 主服务器备份,方便直接把主服务器当前数据复制到从服务器,直接cp数据文件最简单,如下
    ```
    tar -cvf data.tar data
    ```
  - 备份完毕,主服务器可以释放读锁,恢复写操作
    ```
    mysql>unlock tables
    ```
  - 将主数据库的一致性备份恢复到从数据库上。如果是使用.tar 打包的文件包，只需要解开到相应的目录即可
- 从服务器配置
  - 修改从服务器配置my.cnf,增加server-id,必须唯一,修改方式同主服务器
  - 在从服务器上，使用--skip-slave-start 选项启动从数据库，这样不会立即启动从数据库服务上的复制进程，方便我们对从数据库的服务进行进一步的配置
    ```
    ./bin/mysqld_safe --skip-slave-start &
    ```
  - 对从数据库服务器做相应设置，指定复制使用的用户，主数据库服务器的 IP、端口以及开始执行复制的日志文件和位置等，具体语法如下
    ```
    mysql> CHANGE MASTER TO
    -> MASTER_HOST='master_host_name',
    -> MASTER_USER='replication_user_name',
    -> MASTER_PASSWORD='replication_password',
    -> MASTER_LOG_FILE='recorded_log_file_name', //这里填写前面从主服务器查询得到的日志文件名
    -> MASTER_LOG_POS=recorded_log_position;    //这里填写前面从主服务器查询得到的日志偏移量
    ```
  - 在从服务器上，启动 slave 线程
    ```
    mysql>start slave
    ```
  - 从服务器上上执行 show processlist 命令可以查看是否连接到master
  - 也可以在主服务器上做更新操作,查看从服务器是否同步到

### 从服务器复制启动的一些其他重要选项

- 出去上面CHANGE MASTER TO操作的一些参数外,还有几个常用的启动选项
- log-slave-updates 跟--log-bin参数搭配使用,开启从服务器的写binlog的配置,用于从服务器也作为其他服务器的主服务器的情况
- master-connect-retry 用于丢失主服务器连接时,重试的时间间隔,默认60秒
- read-only 开参数限制了从服务器的更新操作只能由超级用户完成,限制应用对服务器的错误更新操作
- replicate-do-db、replicate-do-table、replicate-ignore-db、replicate-ignore-table、replicate-wild-ignore-table(可以用通配符指定表名) 或 replicate-wild-do-table(可以用通配符指定表名) 这些启动参数用于指定从服务器复制指定的数据库或表
- slave-skip-errors 这个启动参数可以配置多个错误码,用于在复制过程中出现异常的情况,一般来说从服务器会停止复制不再同步,等待人工处理,使用这个参数跳过特定的错误码,则可以保证复制继续进行,但是可能会造成从库的数据不完整,如果从库是主库的备份就不应该使用

### 主从日常维护

- show slave status 可以查看从库状态,主要关心“Slave_IO_Running”和“Slave_SQL_Running”这两个进程状态是否是“yes”，这两个进程的含义分别如下。
  - Slave_IO_Running：此进程负责从服务器（Slave）从主服务器（Master）上读取 BINLOG日志，并写入从服务器上的中继日志中。
  - Slave_SQL_Running：此进程负责读取并且执行中继日志中的 BINLOG 日志。只要其中有一个进程的状态是 no，则表示复制进程停止，错误原因可以从“Last_Errno”字段的值中看到。
- 主从服务器同步维护,强制同步
  - 在主库负载较低时,添加读锁阻塞更新操作```FLUSH TABLES WITH READ LOCK```
  - 查看主库的状态获取二进制日志文件名和偏移量```show master status```
  - 从库使用```select MASTER_POS_WAIT('recorded_log_file_name','recorded_log_position')```,阻塞从库直到达到指定的日志文件和偏移量,返回0则同步完成,返回-1则是超时
  - 主库```unlock tables```恢复更新操作
- 从库复制出错处理
  - 首先判断是否因为表结构不同,如果是则通知从库同步stop slave,然后修改表结构,再运行start slave
  - 如果不是,则确认手动更新是不是安全,然后跳过更新失败的语句,指令为```SET GLOBAL SQL_SLAVE_SKIP_COUNTER=X```,其中 X 的取值为 1 或者 2
    - 来自主服务器的更新语句不使用 AUTO_INCREMENT 或 LAST_INSERT_ID()，n 值应为 1
    - 否则，值应为 2。原因是使用AUTO_INCREMENT 或 LAST_INSERT_ID()的语句需要从二进制日志中取两个事件。
- log event entry exceeded max_allowed_packet 说明含有大文本记录无法进行网络传输,需要增加主动服务器上的max_allowed_packe参数的大小,默认是1MB
  ```
  mysql> show variables like 'max_allowed_packet' //查看参数值
  mysql> SET @@global.max_allowed_packet=xxx //临时设置参数值
  //在my.cnf文件中设置max_allowed_packet = xxx则是永久性修改,保证下次启动参数继续有效
  ```
- 多主一从复制时的自增长变量冲突问题,这时应该对不同主库的自增策略进行间隔配置,防止重复

### 主从切换

- stop slave IO_THREAD 停止从库复制进程(Slave_IO_Running),使得从服务器暂时不写中继日志，也就是最后执行的SQL就是当前中继日志中最后一个SQL。
- 通过show processlist 查看从库状态,state提示Has read all relay log,则表示已经执行relay log的全部更新
- 对需要切为主库的从库使用stop slave停止从服务,使用reset master重置成主库
  - 这里要特别注意该从库一定是要开启了log-bin选项的数据库,这样才能在切换成主库后提供对应的binlog
  - 同时要注意,不能开启log-slave-updates,否则会将已经执行过的binlog重复传输到其他从库,造成错误
- 其他从库使用stop slave停止从服务,然后使用change master to 指令来修改主库的配置为新主库的配置(可以仅修改ip,即MASTER_HOST = 'xxx'),然后start slave开启从服务

### mysql多主一从的说明

- 主库配置与前面基本一致
- 从库则需要开启多实例(服务的多实例,数据文件可以用一份,也可以每个实例一份)
- 从库的每个实例对应一个主库,配置方法跟一主的情况一样
- 通过my.cnf配置多实例信息或多个配置文件启动实例

### mysql NDB

- 不支持真正的事务
- 有高可用和写负载均衡
- 需要配置有管理节点  数据节点 sql节点 
- 没有记录配置简明的配置方式
- NDB不是INNODB的集群。
