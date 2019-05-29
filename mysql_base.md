# MYSQL基本特征及优化

## 事务的基本特征
#### 原子性（atomicity）:
     一个事务必须视为一个不可分割的最小工作单元，整个事务中的所有操作要么全部提交成功，要么全部失败回滚，
     对于一个事务来说，不可能只执行其中的一部分操作，这就是事务的原子性
     
#### 一致性（consistency）:
	数据库总数从一个一致性的状态转换到另一个一致性的状态

#### 隔离性（isolation）:
	一个事务所做的修改在最终提交以前，对其他事务是不可见的

#### 持久性（durability）:
	一旦事务提交，则其所做的修改就会永久保存到数据库中。此时即使系统崩溃，修改的数据也不会丢失。



## 事务的隔离级别 (Mysql的默认隔离级别是Repeatable read)
#### 读未提交(Read uncommitted):
	一个事务可以读取另一个未提交事务的数据，最低级别，任何情况都无法保证

#### 读已提交(Read committed):
	一个事务要等另一个事务提交后才能读取数据，可避免脏读的发生

#### 可重复读(Repeatable read):
	就是在开始读取数据（事务开启）时，不再允许修改操作，可避免脏读、不可重复读的发生

#### 串行(Serializable):
	是最高的事务隔离级别，在该级别下，事务串行化顺序执行，可以避免脏读、不可重复读与幻读。
	但是这种事务隔离级别效率低下，比较耗数据库性能，一般不使用


## 脏读、不可重复读、幻读的解释

#### 脏读：
	事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据；
#### 不可重复读：
	事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，
	导致事务A多次读取同一数据时，结果因此本事务先后两次读到的数据结果会不一致；
#### 幻读：
	幻读解决了不重复读，保证了同一个事务里，查询的结果都是事务开始时的状态（一致性）


## 表结构的优化

#### 数据表类型
	MyIASM、InnoDB、HEAP、ISAM、MERGE、DBD、Gemeni(一般只知道MyIASM、InnoDB即可)

#### innodb引擎的4大特性
	插入缓冲(insert buffer)、二次写(double write)、自适应哈希索引(ahi)、预读(read ahead)


#### InnoDB引擎的行锁是基于索引

### Mysql中的myisam与innodb的区别
	InooDB支持事务，而MyISAM不支持事务；

	InnoDB支持行级锁，而MyISAM支持表级锁；

	InnoDB支持MVCC，而MyISAM不支持；

	InnoDB支持外键，而MyISAM不支持；

	InnoDB不支持全文索引，而MyISAM支持；

	InnoDB不能通过直接拷贝表文件的方法拷贝表到另外一台机器， myisam 支持；

	InnoDB表支持多种行格式， myisam 不支持；

	InnoDB是索引组织表， myisam 是堆表；


#### myisam与innodb 执行select count(*) myisam更快，因为myisam内部维护了一个计数器，可以直接调取。

### sql语句优化
	避免select *，将需要查找的字段列出来；

	使用连接（join）来代替子查询；

	拆分大的delete或insert语句；

	使用limit对查询结果的记录进行限定；

	不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，
	否则系统将可能无法正确使用索引尽量避免在where 子句中对字段进行 null 值判断，
	否则将导致引擎放弃使用索引而进行全表扫描；

	尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描；

	尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描；

### 表结构优化
	ID 所有建表的时候设置主键；

	选择正确的存储引擎 ;

	使用可存下数据的最小的数据类型，整型 < date,time < char,varchar < blob；

	使用简单的数据类型，整型比字符处理开销更小，因为字符串的比较更复杂。如，int类型存储时间类型，bigint类型转ip函数；

	使用合理的字段属性长度，固定长度的表会更快。

	使用enum、char而不是varchar；

	尽可能使用not null定义字段(给空字段设置默认值)；

	尽量少用text;

	给频繁使用和查询的字段建立合适的索引；

### mysql常用的函数:
	sum(和)、count（计数几条） 、avg（平均值）、min（最小值）、max（最大值）

## 常用的命令 Explain、describe、show、truncate

	Explain使用方法：EXPLAIN SELECT user_id from orders where user_id=1
	查看explain.md  
	
	DESCRIBE：查看表结构 DESCRIBE orders;

	show：查看表结构 show columns from  orders;

