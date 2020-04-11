## MySQL

### [数据类型](./mysql_data_type.md)

### 索引

注意：  
按索引顺序读取数据的速度通常要比顺序全表扫描慢，因为没扫描一条记录都要回表查询一次对应的行

#### 类型

- B+ tree 索引

  - 适用查找

    - 全值匹配

    - 范围匹配

    - 匹配前缀

  - 限制

    - 非最左列开始的查找，无法使用索引

    - 不能跳过索引的列

    - 某列范围查询时，其右列都无法使用索引
      如果范围查找数量有限，可以通过多个等于条件代替范围条件

    - 查询的列必须是独立的，不能是表达式一部分，或是函数的参数
      例子：  
      select … where id + 1 = 5  

      select … where to_day(date) <= to_day(current_date)

- 哈希索引

  - 限制

    - 不能避免读取行
      哈希索引只包含哈希值和行指针，不存储字段，所以不能使用索引中的值避免读取行

    - 无法排序
      哈希索引数据并不是按照索引值顺序存储的，所以无法用于排序

    - 不支持部分索引列匹配
      因为哈希索引始终是使用索引列的全部内容来计算哈希值的。例如：在数据列（A，B）上建立哈希索引，查询只有数据列A时，无法使用该索引

    - 只支持等值比较查询

    - 哈希冲突时，必须遍历链表

- 全文索引

#### 聚簇索引

聚簇索引不是一种索引类型，而是一种数据存储方式。将数据行存放在索引的叶子页中，即在同一结构里存放B-tree 索引和数据行

#### 覆盖索引

### 事务

[推荐文章](https://draveness.me/mysql-transaction)

#### ACID

- 原子性 atomicity

- 一致性 consistency

- 隔离性 isolation

- 持久性 durability

#### 隔离级别

- 未提交读 read uncommitted
  能读取到其他事物未提交的变更。

  - 脏读  ----  读取到未提交的数据

- 提交读 read committed
一次事务只能读到其他事务提交后的变更。  
读取不加锁，写入，修改，删除则加锁

- 不可重复读
同一事务中，两次同样的查询，会得到不同的结果

- 可重复读 repeatable read
  一次事务中多次读取记录的结果是一直的  

  mysql 的默认隔离级别

  - 幻读
    幻读 ：某事务读取某范围内的记录，另一事务又在该范围内插入了新的记录，之前的事务再读取该范围的记录，会产生幻行

- 串行化，无并发 serializable
  对读取到每一行数据都加锁

#### 控制语句

- begin

- commit

- rollback

- savepoint

- release savepoint

- rollback to

- set transaction

- 

#### 死锁

InnoDB 将持有最少行级排它锁的事务进行回滚

#### 自动提交

如果不是显示地开始一个事务，每个查询都被当做一个事务执行提交操作

#### 事务日志

##### undo日志
记录事务开始前的状态，用于事务失败时的回滚操作

##### redo日志
redo日志记录事务执行后的状态，用来恢复未写入data file的已成功事务更新的数据

#### 多版本并发控制 MVCC

只在提交读和可重复读两个隔离级别下工作

- 行版本号
  创建行的事务系统版本号

- 删除版本号
  删除行的事务系统版本号

### 高级特性

#### 分区表

- 策略

	- 全量扫描数据，不使用索引

	- 索引数据，分离热点

#### 视图

#### 外键约束

### 存储引擎

#### InnoDB

- B+ tree索引结构

- 锁  [推荐文章](https://i6448038.github.io/2019/02/23/mysql-lock/)

  - 行锁
  - 间隙锁
    防止幻读

- 支持事务

- 聚簇索引

  

#### MyISAM

- 表锁

	- 读，共享锁

	- 写，排他锁

- 非事务

- 全文索引

- 前缀压缩索引

### 基准测试

### 性能剖析

### 设置优化

### 操作系统和硬件优化

### 复制

### 可扩展

### 高可用性

### 应用层优化

### 备份与恢复

### SQL

#### select

- select distinct

#### where

- like

  - 通配符

    - % ：替代 0 个或多个字符

    - _   ：替代一个字符

    - [charlist] ：字符列中的任何单一字符

    - [^charlist]  或  [!charlist] ：不在字符列中的任何单一字符

- in   / not in

- between  / not between


#### and & or

#### order by

#### insert into

- insert into select

#### update

#### delete

#### as

- 别名

#### create

- create table

	- constraints 约束

		- not null

		- unique

		- primary key

		- foreign key

		- check

		- default

	- auto_increment

- create index

- create database

#### drop

- drop database

- drop table

- drop index
  ALTER TABLE table_name DROP INDEX index_name

- drop column
  ALTER TABLE Persons  
  DROP COLUMN DateOfBirth

#### join

- left join

- right join

- inner join

- outer join

#### union

- union all



### 查询性能优化

#### 为何不用NUll

- NOT IN子查询包含null
  NOT IN子查询在有NULL值的情况下返回永远为空结果，查询容易出错

- 索引有null
  单列索引不存null值，复合索引不存全为null的值，如果列允许为null，可能会得到“不符合预期”的结果集  

  如果name允许为null，索引不存储null值，结果集中不会包含这些记录。所以，请使用not null约束以及默认值。

- 对 null 做算术运算的结果都是 null

- CONCAT() 有 null
  如果在两个字段进行拼接：比如题号+分数，首先要各字段进行非null判断，否则只要任意一个字段为空都会造成拼接的结果为null。

- count 不包含该null
  如果有 Null column 存在的情况下，count(Null column)需要格外注意，null 值不会参与统计。

- Null 列需要更多的存储空间
  Null 列需要更多的存储空间：需要一个额外字节作为判断是否为 NULL 的标志位

#### explain

- id
  SELECT 查询的标识符. 每个 SELECT 都会自动分配一个唯一的标识符

- select_type
  SELECT 查询的类型.

  - SIMPLE
    表示此查询不包含 UNION 查询或子查询

  - PRIMARY
    表示此查询是最外层的查询

  - UNION
    表示此查询是 UNION 的第二或随后的查询

  - DEPENDENT UNION
    UNION 中的第二个或后面的查询语句, 取决于外面的查询

  - UNION RESULT
    UNION 的结果

  - SUBQUERY
    子查询中的第一个 SELECT

  - DEPENDENT SUBQUERY
    子查询中的第一个 SELECT, 取决于外面的查询. 即子查询依赖于外层查询的结果

- table
  查询的是哪个表

  - system
    表中只有一条数据. 这个类型是特殊的 const 类型.

  - const
    针对主键或唯一索引的等值查询扫描, 最多只返回一行数据. const 查询速度非常快, 因为它仅仅读取一次即可.  
    例如下面的这个查询, 它使用了主键索引, 因此 type 就是 const 类型的.

  - eq_ref
    此类型通常出现在多表的 join 查询, 表示对于前表的每一个结果, 都只能匹配到后表的一行结果. 并且查询的比较操作通常是 =, 查询效率较高

  - ref
    此类型通常出现在多表的 join 查询, 针对于非唯一或非主键索引, 或者是使用了 最左前缀 规则索引的查询. 

  - range
    表示使用索引范围查询, 通过索引字段范围获取表中部分数据记录. 这个类型通常出现在 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN() 操作中.  

    当 type 是 range 时, 那么 EXPLAIN 输出的 ref 字段为 NULL, 并且 key_len 字段是此次查询中使用到的索引的最长的那个.

  - index_merge

  - index
    表示全索引扫描(full index scan), 和 ALL 类型类似, 只不过 ALL 类型是全表扫描, 而 index 类型则仅仅扫描所有的索引, 而不扫描数据.  

    index 类型通常出现在: 所要查询的数据直接在索引树中就可以获取到, 而不需要扫描数据. 当是这种情况时, Extra 字段 会显示 Using index

  - ALL
    表示全表扫描

- partitions
  匹配的分区

- type
  类型

- possible_keys
  此次查询中可能选用的索引

- key
  此次查询中确切使用到的索引.

- key_len
  表示查询优化器使用了索引的字节数. 这个字段可以评估组合索引是否完全被使用, 或只有最左部分字段被使用到.

- ref
  哪个字段或常数与 key 一起被使用

- rows
  显示此查询一共扫描了多少行. 这个是一个估计值.

- filtered
  表示此查询条件所过滤的数据的百分比

- extra
  额外的信息

  - Using filesort
    当 Extra 中有 Using filesort 时, 表示 MySQL 需额外的排序操作, 不能通过索引顺序达到排序效果.

  - Using index
    "覆盖索引扫描", 表示查询在索引树中就可查找所需数据, 不用扫描表数据文件

  - Using temporary
    查询有使用临时表, 一般出现于排序, 分组和多表 join 的情况, 查询效率不高, 建议优化.

#### 查询优化

- limit 大数偏移，查询优化
  用覆盖索引 优化

#### 索引优化

#### 库表结构优化

---

### 常见问题

##### B+ tree 和 B tree 区别， 为什么要用 B+ tree

##### 索引如何实现，建索引需要考虑什么，如何优化慢查询

##### 事务隔离级别，幻读和脏读区别

##### InnoDB  和 MyISAM 区别

##### 事务如何实现，MVCC ， 以及用到哪些锁







