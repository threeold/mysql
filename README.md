# mysql基本语法
### 1、单条记录
	SELECT user_id FROM orders where user_id=1 limit 1

### 2、group gy、order by用法
	SELECT user_id,sum(pay_money) as pay_money,count(id) as count_tj 
	FROM orders 
	GROUP BY user_id 
	HAVING count_tj>2 
	ORDER BY pay_money desc

### 3、ROUND四舍五入函数
	SELECT pay_money,ROUND(pay_money,1)  FROM orders
	
### 4、TRUNCATE舍去法
	SELECT pay_money,TRUNCATE(pay_money,1)  FROM orders

### 5、if的用法

	SELECT sum(if(pay_money>100,1,0)) as tj_num FROM orders


### 6、CASE用法
	SELECT
	    CASE
		WHEN pay_money>=0 and pay_money<100 THEN '100'
		WHEN pay_money>=100 and pay_money<200 THEN '200'
		ELSE '300'
		END AS test 
	FROM orders
    

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
	
#### mysql查询时给查出来的数据设置一个自增的序号
     
      set @i=0;
      SELECT (@i:=@i+1) as "别名"， FROM order_0, (SELECT @i:=0) AS order_0;
      
      
## 存储过程

#### in 变量 int
      in 调用 传参变量，参数类型int
#### out 变量 int
      out 返回，参数变量，参数类型int
      
#### declare 声明变量
      declare todaytime int;##定义 todaytime是int
      set todaytime=UNIX_TIMESTAMP(curdate());##给todaytime赋值
      
#### 调用或执行存储过程
      CALL GetOrderLastDay(1,@lasttime);
      SELECT @lasttime; ##查看值
      ##goodsid 接收到的值 是1 
      ##@last_day接收到的是 o_p返回的值
      
      
### 存储过程-1    
      CREATE DEFINER=`threeold`@`%` PROCEDURE `GetOrderLastDay`(in goodsid int,out o_p int)
      BEGIN
	      #执行到哪一天
	      SELECT statisics_day into o_p from order_day where goods_id=goodsid ORDER BY statisics_day desc limit 1;
      END
      
      
###  存储过程-2

      CREATE DEFINER=`threeold`@`%` PROCEDURE `SetOrderDay`(in goodsid int,in day_time int,in rate decimal(20,6),in price       decimal(20,2),in days int,in operation_time int)
     BEGIN
	declare todaytime int;##今天时间
        declare today_d int;##今天几号
	set todaytime=UNIX_TIMESTAMP(curdate());
	set today_d=FROM_UNIXTIME(day_time+86400,'%d');
  	CALL GetOrderLastDay(goodsid,@lasttime); ##调用“存储过程-1”，最后一次执行时间
	if (@lasttime is NULL or day_time>@lasttime) and day_time<todaytime THEN
            ###读不到@lasttime或@lasttime小于 day_time，就执行这边的代码
	end if;
	if today_d="1" THEN  ##每月一号要执行的
    	    ###每月一号要执行这边的代码
  	end if;
    END
      
      
      
