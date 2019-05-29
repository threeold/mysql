# mysql基本语法
### 单条记录

SELECT user_id FROM orders where user_id=1 limit 1

### （group gy） （order by）

SELECT user_id,sum(pay_money) as pay_money,count(id) as count_tj  
FROM orders GROUP BY user_id ORDER BY pay_money desc

===============ROUND四舍五入函数=============================

SELECT pay_money,ROUND(pay_money,1)  FROM orders


===============TRUNCATE舍去法===============================

SELECT pay_money,TRUNCATE(pay_money,1)  FROM orders


===============if的用法====================================

SELECT sum(if(pay_money>100,1,0)) as tj_num FROM orders


===============CASE用法====================================


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
    

===============insert ignore into ==========  
查询惟一索引不重复的数据，ignore如果有重复不报错跳过

insert ignore into order_1(user_id,c_time)
SELECT user_id,UNIX_TIMESTAMP(NOW()) as c_time 
from orders where pay_money>0



===============UPDATE 批量更新=================

UPDATE order_1 as p
  INNER JOIN
  (
    SELECT user_id,sum(pay_money) as pay_money from orders GROUP BY user_id
  )as s on  p.user_id=s.user_id
  set p.pay_money=s.pay_money;


=======UNION= 连接两个以上的 SELECT 语句的结果组合到一个结果集合========

SELECT id,login FROM order_0 where id<14600
UNION
SELECT id,login FROM order_1  where id<14600


===============LEFT JOIN================================

关键字从左表（order_0）返回所有的行，即使右表（order_1）中没有匹配。如果右表中没有匹配，则结果为 NULL

SELECT o.id,o.login,o1.login as login1 FROM order_0 as o 
LEFT JOIN order_1  as o1 on o.id=o1.id where o.id <14630

===============RIGHT JOIN ================================

关键字从右表（order_1）返回所有的行，即使左表（order_0）中没有匹配。如果左表中没有匹配，则结果为 NULL。

SELECT o.id,o.login,o1.login as login1 FROM order_0 as o 
RIGHT JOIN order_1  as o1 on o.id=o1.id where o.id <14630


//===============INNER JOIN ================================

INNER JOIN 关键字左表（order_0）、右表（order_1）匹配的行才会返回

SELECT o.id,o.login,o1.login as login1 FROM order_0 as o 
RIGHT JOIN order_1  as o1 on o.id=o1.id where o.id <14630


