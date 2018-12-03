# 实验三 创建分区表
## 实验目的 
掌握分区表的创建方法，掌握各种分区方式的使用场景。
## 实验内容
本实验使用3个表空间：USERS,USERS02,USERS03。在表空间中创建两张表：订单表(orders)与订单详表(order_details)。
使用你自己的账号创建本实验的表，表创建在上述3个分区，自定义分区策略。
你需要使用system用户给你自己的账号分配上述分区的使用权限。你需要使用system用户给你的用户分配可以查询执行计划的权限。
表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。
写出插入数据的语句和查询数据的语句，并分析语句的执行计划。
进行分区与不分区的对比实验。
### 实验步骤
#### 1，登录自己的账号 new_userrx
<pre>[oracle@deep02 ~]$ sqlplus new_userrx/123@pdborcl

SQL*Plus: Release 12.1.0.2.0 Production on 星期三 11月 7 09:38:22 2018

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

#### 2，创建orders表
SQL> CREATE TABLE orders 
(
 order_id NUMBER(10, 0) NOT NULL 
 , customer_name VARCHAR2(40 BYTE) NOT NULL 
 , customer_tel VARCHAR2(40 BYTE) NOT NULL 
 , order_date DATE NOT NULL 
 , employee_id NUMBER(6, 0) NOT NULL 
 , discount NUMBER(8, 2) DEFAULT 0 
 , trade_receivable NUMBER(8, 2) DEFAULT 0 
) 
TABLESPACE USERS 
PCTFREE 10 INITRANS 1 
STORAGE (   BUFFER_POOL DEFAULT ) 
NOCOMPRESS NOPARALLEL 
PARTITION BY RANGE (order_date) 
(
 PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (
 TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 
 'NLS_CALENDAR=GREGORIAN')) 
 NOLOGGING 
 TABLESPACE USERS 
 PCTFREE 10 
 INITRANS 1 
 STORAGE 
( 
 INITIAL 8388608 
 NEXT 1048576 
 MINEXTENTS 1 
 MAXEXTENTS UNLIMITED 
 BUFFER_POOL DEFAULT 
) 
NOCOMPRESS NO INMEMORY  
, PARTITION PARTITION_BEFORE_2017 VALUES LESS THAN (
TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 
'NLS_CALENDAR=GREGORIAN')) 
NOLOGGING 
TABLESPACE USERS02 
...
);

#### 创建orders_details表
SQL> CREATE TABLE order_details 
(
id NUMBER(10, 0) NOT NULL 
, order_id NUMBER(10, 0) NOT NULL
, product_id VARCHAR2(40 BYTE) NOT NULL 
, product_num NUMBER(8, 2) NOT NULL 
, product_price NUMBER(8, 2) NOT NULL 
, CONSTRAINT order_details_fk1 FOREIGN KEY  (order_id)
REFERENCES orders  (  order_id   )
ENABLE 
) 
TABLESPACE USERS 
PCTFREE 10 INITRANS 1 
STORAGE (   BUFFER_POOL DEFAULT ) 
NOCOMPRESS NOPARALLEL
PARTITION BY REFERENCE (order_details_fk1)
(
PARTITION PARTITION_BEFORE_2016 
NOLOGGING 
TABLESPACE USERS --必须指定表空间,否则会将分区存储在用户的默认表空间中
...
) 
NOCOMPRESS NO INMEMORY, 
PARTITION PARTITION_BEFORE_2017 
NOLOGGING 
TABLESPACE USERS02
...
) 
NOCOMPRESS NO INMEMORY  
);

#### 3，用system用户授权自己的账号访问USERS02,USERS03表空间。
SQL*Plus: Release 12.1.0.2.0 Production on 星期三 11月 7 09:40:22 2018

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

上次成功登录时间: 星期三 11月 07 2018 08:44:05 +08:00
连接到:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options
SQL> ALTER USER new_userrx QUOTA 50M ON users02;
用户已更改。
SQL> ALTER USER new_userrx QUOTA 50M ON users03;
用户已更改。
SQL> exit
#### 4，给用户分配可查询的权限的语句。
grant SELECT on "new_userrx"."ORDERS" to "new_userrx" ;
grant SELECT on "new_userrx"."ORDER_DETAILS" to "new_userrx" ;

#### 5，向表中插入一万条数据
向orders表中插入一万条数据。
begin
for i in 1..10000
loop
 insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(i ,'rx','151xxxxxxxx',to_date('2017-02-14','yyyy-mm-dd'),1,2);
end loop;
commit;
end;

#### ,6，向orders_details表中插入一万条数据。
begin
for i in 1..10000
loop
insert into order_details(ID,ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) VALUES(i,i,'01-01',i,1000);
end loop;
commit;
end;
#### 7，联合查询语句。
SELECT * FROM orders,order_details  Where orders.order_id = order_details.order_id AND customer_tel like '%151%'
</pre>
