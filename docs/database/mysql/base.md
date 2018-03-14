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
- 