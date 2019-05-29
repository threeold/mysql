# mysql基本语法
### 1、单条记录

SELECT user_id FROM orders where user_id=1 limit 1

### 2、group gy、order by用法

SELECT user_id,sum(pay_money) as pay_money,count(id) as count_tj  
FROM orders GROUP BY user_id HAVING count_tj>2 ORDER BY pay_money desc

### 3、ROUND四舍五入函数

SELECT pay_money,ROUND(pay_money,1)  FROM orders


### 4、TRUNCATE舍去法

SELECT pay_money,TRUNCATE(pay_money,1)  FROM orders


### 5、if的用法

SELECT sum(if(pay_money>100,1,0)) as tj_num FROM orders


### 6、CASE用法


SELECT
CASE
WHEN pay_money>=0 and pay_money<100 THEN
    '100'
WHEN pay_money>=100 and pay_money<200 THEN
    '200'
ELSE
    '300'
END AS test 
FROM
    orders
    

### 7、insert ignore into
查询惟一索引不重复的数据，ignore如果有重复不报错跳过

insert ignore into order_1(user_id,c_time)
SELECT user_id,UNIX_TIMESTAMP(NOW()) as c_time 
from orders where pay_money>0



### 8、UPDATE 批量更新

UPDATE order_1 as p
  INNER JOIN
  (
    SELECT user_id,sum(pay_money) as pay_money from orders GROUP BY user_id
  )as s on  p.user_id=s.user_id
  set p.pay_money=s.pay_money;


### 9、UNION= 连接两个以上的 SELECT 语句的结果组合到一个结果集合

SELECT id,login FROM order_0 where id<14600
UNION
SELECT id,login FROM order_1  where id<14600


### 10、LEFT JOIN

关键字从左表（order_0）返回所有的行，即使右表（order_1）中没有匹配。如果右表中没有匹配，则结果为 NULL

SELECT o.id,o.login,o1.login as login1 FROM order_0 as o 
LEFT JOIN order_1  as o1 on o.id=o1.id where o.id <14630

### 11、RIGHT JOIN

关键字从右表（order_1）返回所有的行，即使左表（order_0）中没有匹配。如果左表中没有匹配，则结果为 NULL。

SELECT o.id,o.login,o1.login as login1 FROM order_0 as o 
RIGHT JOIN order_1  as o1 on o.id=o1.id where o.id <14630

### 12、INNER JOIN 

INNER JOIN 关键字左表（order_0）、右表（order_1）匹配的行才会返回

SELECT o.id,o.login,o1.login as login1 FROM order_0 as o 
RIGHT JOIN order_1  as o1 on o.id=o1.id where o.id <14630


### 13、需要本表跟本表关联查询出来的数据

SELECT o.id,o.login FROM order_0 as o
INNER JOIN(

   SELECT id,login FROM order_0  where id=14600
   
) as b on  o.id=b.id

# MYSQL基本特征

## 事务的基本特征
#### 原子性（atomicity）:
     一个事务必须视为一个不可分割的最小工作单元，整个事务中的所有操作要么全部提交成功，要么全部失败回滚，对于一个事务来说，不可能只执行其中的一部分操作，这就是事务的原子性
     
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
是最高的事务隔离级别，在该级别下，事务串行化顺序执行，可以避免脏读、不可重复读与幻读。但是这种事务隔离级别效率低下，比较耗数据库性能，一般不使用


## 脏读、不可重复读、幻读的解释

#### 脏读：
事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据；
#### 不可重复读：
事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果因此本事务先后两次读到的数据结果会不一致；
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

不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引尽量避免在where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描；

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

### Explain使用使用方法 
使用方法，在select语句前加上explain就可以了：

如：EXPLAIN SELECT user_id FROM accounts_wallet_order where user_id=1 

#### EXPLAIN列的解释：

##### table
显示这一行的数据是关于哪张表的

##### type
这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为const、eq_reg、ref、range、indexhe和ALL

##### possible_keys
显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句

##### key
实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。这种情况下，可以在SELECT语句中使用USE INDEX（indexname）来强制使用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引

##### key_len
使用的索引的长度。在不损失精确性的情况下，长度越短越好

##### ref
显示索引的哪一列被使用了，如果可能的话，是一个常数

##### rows
MYSQL认为必须检查的用来返回请求数据的行数

##### Extra
	关于MYSQL如何解析查询的额外信息。将在表4.3中讨论，但这里可以看到的坏的例子是Using temporary和Using filesort，意思MYSQL根本不能使用索引，结果是检索会很慢


### extra列返回的描述的意义

##### Distinct
	一旦MYSQL找到了与行相联合匹配的行，就不再搜索了

#####  Not exists
	MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不再搜索了

##### Range checked for each
	没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。

##### Record（index map:#）
    这是使用索引的最慢的连接之一	

##### Using filesort
	看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行

##### Using index
	列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候

##### Using temporary
	看到这个的时候，查询需要优化了。这里MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上

##### Where used
	使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题
