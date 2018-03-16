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
- 事务隔离中,读一致性可以通过两种方式来处理
    - 加锁,串行化事务
    - 不加锁,数据多版本并发控制(MCC或MVCC)