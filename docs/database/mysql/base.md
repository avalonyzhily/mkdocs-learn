### mysql的启动

- 通常用 mysqld_safe 支持很多参数来监控MySQL服务器的运行,部分系统使用mysql.server运行也是调用这个指令脚本
- 一般不会直接使用mysqld指令直接运行

### mysql引擎类型

- 常见的 myisam innodb
    - myisam是最简单的引擎,不支持事务,但是读写速度快,主要依靠3个后缀的文件来实现存储(frm MYD MYI)
    - innodb是常用的引擎,支持事务,支持外键,支持自增列,读写性能相比前者慢一些,有共享表空间和多表空间两种存储方式,但是表结构都是放在.frm后缀的文件中
- 还有memory merge rdb等
    - memory是基于内存的,
    - merge是多张myisam引擎的表的集合

### mysql的索引

- mysql有唯一索引(BTree,hash等) 全文索引 空间索引几种类型
- myisam和innodb均使用BTree索引(索引的数据结构),部分版本支持全文索引(fulltext),myisam支持空间索引(spatial)
- mysql也可以使用前缀索引,即对索引字段的前N个字符创建索引
- 索引的设计是有原则的
    - 搜索的索引列(即where关键字后的列,连接子句中指定的列)
    - 使用唯一索引
    - 使用短索引
    - 利用最左前缀(即使用N列索引中最左边的列集来匹配,成为最左前缀)
    - 不要过度索引
    - innodb有默认的记录保存顺序,有明确的主键时,按主键排序,其次是唯一索引,最后是自建的内部列,访问速度以主键和内部列最快,因此最好是明确指定主键。
    - BTree索引相比Hash索引支持的过滤条件的形式更多。

### mysql事务控制和锁操作

- DDL语句无法回滚,隐式提交
- mysql5.0.3以上开始支持分布式事务
    - XA分布式事务,采用2PC,存在一些缺陷
    - 处于prepare阶段的事务,如果遇到数据库异常,恢复后事务仍存在,但是事务提交不会记录到binlog,影响之后的容灾和数据恢复
    - 如果有分支事务失败,会自动回滚,但是如果该分支已经到达prepare阶段,则其他分支事务可能已经提交,其他事务不会回滚导致事务不完整
    - prepare状态的分支事务不会记录到binlog,因此是无法通过binlog恢复的

### mysq的SQL MODE

- 严格模式可以提供比较好的数据校验功能,譬如TRADITIONAL、STRICT_TRANS_TABLES
- 多种模式可以灵活组合，组合后的模式可以更好地满足应用程序的需求。尤其在数据迁移中，SQL Mode 的使用更为重要

### mysql sql 优化

- 通过show status查看各类sql的执行频率
    - 有Com_xxx,提供xxx对应语句的执行次数,每执行一次累加1
    - 针对innodb引擎有Innodb_rows_xxx,用来统计xxx对应的语句影响的行数
    - Connections 代表视图连接mysql服务器的次数
    - Uptime 代表服务器工作时间
    - Slow_queries 代表慢查询的次数
- 通过日志输出慢sql到日志文件,这个需要启动时增加选项,还可以通过show processlist命令来实时查看mysql当前情况。
- 当定位了低效sql之后,就可以使用执行计划来分析sql了,使用explain或desc命令

### mysql 索引

- 多列索引,必须要有最左列索引字段出现,才会使用索引
- like查询,只能后模糊(%在后),前模糊(%在前)不使用索引
- 大文本搜索使用全文索引,不要使用like '%...%'
- 使用索引比全表扫描更慢,则不适用索引
- or分割的条件,前面不包含索引列,就不会使用索引
- 索引列是字符串,那么条件值必须要以字符串形式传递,否则不适用索引(譬如 294 和 ‘294’)
- 通过show status 查询Handler_read_xxx的结果,可以获得索引的使用情况

### 简单使用的mysql优化

- 定期分析表 ANALYZE TABLE table_name
- 定义优化表 OPTIMIZE TABLE table_name
- SQL的优化
    - 插入语句的优化,innodb引擎可以在批量插入时,先按主键排序好,关闭唯一性校验(SET UNIQUE_CHECKS=0),关闭自动提交等(SET AUTOCOMMIT=0)。
    - myisam引擎则可以通过DISABLE KEYS 和 ENABLE KEYS 用来打开或者关闭 MyISAM 表非唯一索引的更新,提高速度;或者增加bulk_insert_buffer_size变量值
    - 通用的使用一次插多值的方式,如果时不同客户端则需要额外使用INSERT DELAYED。
    - 使用文本装载一个表时,使用LOAD DATA INFILE,比普通的INSERT快20倍
- 优化GROUP BY, 由于mysql会默认针对GROUP BY的字段进行排序,如果没有显式指定ORDER BY的字段,通过 ORDER BY NULL避免排序消耗。
- 优化ORDER BY, where和order by 使用了相同的索引列,order by的顺序与索引顺序相同(这里指多列索引)，同为升序或降序时,会使用索引达到优化效果
    - 因此使用不同关键字索引排序时,不会走索引。
- 使用表连接来替换子查询,优化嵌套查询
- or条件的优化,必须所有字段都是索引列,才会走索引;而且复合索引无效。mysql的or条件查询使用索引时,实际上是单独每个条件进行查询后,再进行union操作
- 使用SQL提示,即在sql加入认为的提示,达到优化效果
    - USE INDEX: select * from table_name use index(index_name) where xxxx
    - IGNORE INDEX: select * from table_name ignore index(index_name) where xxxx
    - FORCE INDEX: select * from table_name force index(index_name) where xxxx ps:强制使用索引时,即使效率不是最好,也会使用索引
- 优化数据类型
- 拆分表提高访问效率

### mysql 锁

- 表锁 开销小,加锁快,不会死锁,但是粒度大,冲突概率高,并发性能低
- 行锁 开销大,加锁慢,会出现死锁,粒度最小,冲突概率低,并发性能最高
- 页面锁 折中
- 各种引擎对锁的支持不一样
    - myisam只支持表锁
        - 通过show status检查table_locks_waited(等待获取的锁数量)和table_locks_immediate(立即可获取的锁数量)状态变量,来获取系统表锁的情况
        - myisam读锁只阻塞写请求,写锁同时阻塞读请求和写请求
        - SELECT操作给涉及的表加上读锁,UPDATE DELETE INSERT给涉及的表加上写锁
        - LOCK TABLE显式加锁
        - 锁请求上,写锁会比读锁优先请求到;可以通过myisam得锁调度,配置一些参数来控制读锁和写锁得优先级,也可以设置一些阈值(譬如max_write_lock_count),来再特定情况下调整读写锁各自得优先级
    - innodb还支持行锁
        - 通过show status检查InnoDB_row_lock_xxx等一系列状态变量,来获取行锁的情况(表锁则同上)
        - 还可以设置监视器
        - 两种行锁
            - 共享锁 阻止其他事务获得排他写锁
            - 排他锁 阻止其他事务获得共享读锁和排他写锁
        - 两种意向锁
            - 意向共享锁 加共享锁前需要获取该锁
            - 意向排他锁 加排他锁前需要获取该锁
        - 加锁
            - 意向锁都是自动加的,用户不需要干预
            - 共享锁获取,...LOCK IN SHARE MODE
            - 排他锁获取,...FOR UPDATE
    - 间隙锁的理解,对于条件范围内不存在的记录也会加锁,因此叫间隙锁,这是为了满足mysql基于sql的binlog日志在恢复和复制时保证数据正确的需要,同时也可以有效的防止幻读
    - myisam一次性获得所需要的锁,因此不会死锁,但是容易出现锁冲突;innodb中,非单sql组成的事务，锁是逐步获取的,因此容易出现死锁
- 事务隔离中,读一致性可以通过两种方式来处理
    - 加锁,串行化事务
    - 不加锁,数据多版本并发控制(MCC或MVCC)

### mysql server 参数优化

- table_cache 数据库用户打开表的缓存数量,与 max_connections 有关
- myisam
    - key_buffer_size 索引块（Index Blocks）缓存的大小;默认的不能删除,但是mysql5.1以后可以设置多个,所有线程共享
- innodb
    - innodb_buffer_pool_size 表数据和索引数据的最大内存缓冲区大小,越高需要的磁盘I/O越少,但是内存占用大
    - innodb_flush_log_at_trx_commit 控制缓冲区中的数据写入到日志文件以及日志文件数据刷新到磁盘的操作时机,默认值为1,最安全
        - 值为 0 日志缓冲每秒一次地被写到日志文件，并且对日志文件做向磁盘刷新的操作，但是在一个事务提交不做任何操作(最不安全,数据库崩溃时会丢失最多1秒的数据)
        - 值为 1 在每个事务提交时，日志缓冲被写到日志文件，并且对日志文件做向磁盘刷新的操作,最安全,但是性能损失大
        - 值为 2 在每个事务提交时，日志缓冲被写到日志文件，但不对日志文件做向磁盘刷新的操作，对日志文件每秒向磁盘做一次刷新操作(只要操作系统不崩溃,不会损失数据)
    - innodb_lock_wait_timeout 锁等待的超时时间设置,针对表锁导致的死锁不能自动检测的情况,设置一个等待超时,默认50秒
    - innodb_support_xa mysql分布式事务支持的开关
- 磁盘的分布I/O优化也可以是myisam引擎收益,但是对于innodb引擎则会起反作用,这时应该选择使用Raw Device裸设备

### mysql 日志

- mysql的错误日志可以用--log-error[=file_name]来指定文件名和存储位置,默认在DATADIR目录下(命名为主机名.err),通过查看这个文件来获取错误日志
- 二进制日志记录的是DML和DDL语句,但是没有查询语句,语句以事件的形式存储
    - 可以用--log-bin[=file_name]来指定文件名和存储位置,默认在DATADIR目录下(命名为主机名-bin),通过查看这个文件来获取二进制日志
    - 二进制日志文件不能直接读取,而是需要使用mysql的工具——mysqlbinlog log-file
    - 删除日志指令
        - RESET MASTER 删除所有日志
        - PURGE MASTER LOGS TO ' mysql-bin.****** '命令，该命令将删除******编号之前的所有日志。
        - 执行PURGE MASTER LOGS BEFORE 'yyyy-mm-dd hh24:mi:ss'命令，该命令将删除日期为yyyy-mm-dd hh24:mi:ss之前产生的所有日志
        - 设置参数--expire_logs_days=#，此参数的含义是设置日志的过期天数，过了指定的天数后日志将会被自动删除
- mysql有独立的查询日志设置--log[=file_name]来指定文件名和存储位置,默认在DATADIR目录下(命名为主机名.log)
- mysql还有独立的慢查询日志--log-slow-queries[=file_name]来指定文件名和存储位置,默认在DATADIR目录下(命名为主机名-slow.log)
    - 慢查询的阈值key是long_query_time

### mysql备份和恢复

- 逻辑备份和恢复
    - mysqldump备份指令
        - mysqldump [opt] db_name [tables] > xxx.sql //备份指定数据库或其中的一些表到文件
        - mysqldump [opt] ---database [DB1,DB2,....] > xxx.sql //备份指定数据库到文件
        - mysqldump [opt] --all--database > xxx.sql //备份所有数据库
    - mysql恢复
        - 完全恢复,可以直接只用备份文件恢复，还可以使用mysqlbinlog恢复自备份以来的binlog
        - 可以基于时间点来回复
        - 基于位置恢复,精度高于基于时间点,因为同一时间点可能有多个sql语句
- 物理备份和恢复
    - 物理备份比逻辑备份更快,基于文件的cp
    - 冷备份 停掉mysql服务，cp数据文件进行备份和恢复
    - 热备份 不同的引擎不一样的操作
        - myisam 
            - mysqlhotcopy工具
            - flush table for read 手动给所有表加读锁,然后cp数据文件
        - innodb 
            - ibbackup 收费工具,可免费1个月
- 表的导出
    - SELECT .. FROM TABLE .. INTO OUTFILE 'FILE_NAME' [opt]
    - mysqldump target_dir dbname tablename [opt] 导出数据为文本
- 表的导入
    - LOAD DATA [LOCAL] INFILE 'filename' INTO TABLE tablename [opt]
    - mysqlimport [--LOCAL] dbname order_tab.txt [option]
- PS: 凡是mysql提供的命令行工具,都需要提供数据库连接信息,比如 -u -p这样的,其他的数据库指令则是在数据库提供的命令行操作界面直接使用

### mysqk权限和安全

- 权限系统的两阶段
    - 对连接用户进行身份认证,合法的通过,不合法的拒绝
    - 对合法用户,赋予对应的权限,权限内可以对数据库进行操作
    - mysql使用ip(连接方的ip,也可以用域名)和用户名联合认证用户,联合唯一视为同一个用户
- 权限信息存储在mysql这个数据库的user、host和db三个表中
    - 其中user表最重要,分为用户列 权限列 安全列 资源列,db表次之
    - 合法用户取得权限顺序 user->db->tables_priv->column_priv
    - 权限范围依次递减,全局权限覆盖局部权限
- mysql账户的创建可以通过GRANT指令或直接操作授权表,但是推荐使用GRANT指令,不容易出错,GRANT语法如下
    - GRANT priv_type [ ( column_list ) ] [, priv_type [ (column_list)]]... ON [object_type] {tblname | * | *.* |db_name.*} TO user [IDENTIFIED BY [PASSWORD] 'password'] [, user [IDENTIFIED BY [PASSWORD] 'password']... [WITH GRANT OPTION]
    - object_type = TABLE | FUNCTION | PROCEDURE
    - priv_type = 对应user表里面的 'xxx_priv' 列的前缀
- 查看权限
    - show grants for user@host
- 修改权限
    - GRANT语法同上,不存在的账号则新增,存在则修改
    - REVOKE语法同GRANT,回收权限,但是不能删除用户
    - 直接更改权限表
- 修改密码
    - mysqladmin -u username -h hostname password 'xxx'
    - grant usage on *.* to 'jeffrey'@'%' identified by 'xxxx'
    - set password = password('xxx') //修改自己的。
- 删除账号
    - DROP USER user[,user...]

### mysql 安全问题

- 控制系统账号和权限
- 避免root用户启动mysql,数据目录设置为mysql用户,用mysql用户启动数据库,可以防止有FILE权限(数据库的权限)的用户使用root通过sql创建文件。
- 防止dns欺骗,host指定域名时,可能会因为域名对应的ip地址被修改,而出现问题。
- 数据库安全
    - 删除匿名帐号,安装时默认创建的空帐户
    - 给root账户设置密码
    - 使用安全密码
    - 只授予账号必要的权限
    - root账号之外的用户都不应该拥有user表的存取权限
    - 不要把FILE 、PROCESS  或 SUPER 权限授予管理员以外的账号
    - LOAD DATA LOCAL  带来的安全问题,--local-infile=0 选项启动 mysqld 从服务器端禁用所有 LOAD DATA LOCAL命令。
        - 可以任意加载本地的有访问权限的文件
        - web客户端的用户会通过该功能访问到web服务器上的文件,因为此时mysql服务器的客户端是web服务器
    -  DROP TABLE不会收回以前关于该表的授权,因此再次创建的同名表,以前的用户会自动赋予该表的权限,造成安全问题
    -  使用SSL安全连接,最好给用户加上访问IP限制 
    -  REVOKE  命令的漏洞,单个数据库多次赋予权限会自动合并,但是多个数据库(特别是有重复数据库时),多次赋予权限,会被认为时单独的一组不会合并,因此使用REVOKE回收时会出现遗漏
    - 其他安全设置选项


