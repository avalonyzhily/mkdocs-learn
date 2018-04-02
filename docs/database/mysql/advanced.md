### 引擎选择的因素考虑

- 事务:
  - 考虑事务的,请选择InnoDB
  - 不考虑事务,且只有查询和新增操作的,可以选择MyISAM
- 备份
  - 需要在线热备份的,请选择InnoDB
- 崩溃恢复 InnoDB相比MyISAM发生崩溃的概率小,恢复速度也快。
- 特有特性
  - 聚簇索引 InnoDB
  - 地理空间搜索 myisam
- 转换表的引擎
  - 最直接的方法是使用 ALTER TABLE tablename ENGINE = enginename, 但是会有一个问题,执行时间长,因为mysql会创建新的表,然后赋值数据,可能消耗掉系统所有的I/O能力
  - 通过导入导出,使用mysqldump工具
  - 创建与查询,创建表,然后使用INSERT INTO ... SELECT