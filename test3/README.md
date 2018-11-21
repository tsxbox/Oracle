# 实验3：创建分区表

## 实验目的：

掌握分区表的创建方法，掌握各种分区方式的使用场景。

## 实验内容：
- 本实验使用3个表空间：USERS,USERS02,USERS03。在表空间中创建两张表：订单表(orders)与订单详表(order_details)。
- 使用**你自己的账号创建本实验的表**，表创建在上述3个分区，自定义分区策略。
- 你需要使用system用户给你自己的账号分配上述分区的使用权限。你需要使用system用户给你的用户分配可以查询执行计划的权限。
- 表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。
- 写出插入数据的语句和查询数据的语句，并分析语句的执行计划。
- 进行分区与不分区的对比实验。

## 实验步骤
### 1、创建orders订单表
-   SQL语句：
```sql
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
TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 
'NLS_CALENDAR=GREGORIAN')) 
NOLOGGING 
TABLESPACE USERS03 
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
);
```
### 2、创建订单详表(order_details)
-   SQL语句：
```sql
CREATE TABLE order_details   
(  
id NUMBER(10, 0) NOT NULL   
, order_id NUMBER(10, 0) NOT NULL  
, product_id VARCHAR2(40 BYTE) NOT NULL   
, product_num NUMBER(8, 2) NOT NULL   
, product_price NUMBER(8, 2) NOT NULL   
, CONSTRAINT order_details_fk1 FOREIGN KEY  (order_id)  
REFERENCES orders(order_id)  
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
NOCOMPRESS NO INMEMORY,   
PARTITION PARTITION_BEFORE_2017   
NOLOGGING   
TABLESPACE USERS02  
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
NOCOMPRESS NO INMEMORY,  
PARTITION PARTITION_BEFORE_2018   
NOLOGGING   
TABLESPACE USERS03  
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
);  
```
### 3、分配权限

![IMAGE](https://github.com/tsxbox/Oracle/blob/master/test3/quanxianone.png)
![IMAGE](https://github.com/tsxbox/Oracle/blob/master/test3/quanxiantwo.png)
### 4、插入数据、联合查询
--------
##### 向两个表中各自插入10000条数据：
BEGIN
       FOR i IN 1..3000 LOOP
       insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(i,'黎明',12345,to_date('2017-12-14','yyyy-mm-dd'),i,i);
       END LOOP;
       COMMIT;
       FOR i IN 3001..6000 LOOP
       insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(i,'李晓',12345,to_date('2015-02-14','yyyy-mm-dd'),i,i);
       END LOOP;
       COMMIT;
       FOR i IN 6001..10000 LOOP
       insert into ORDERS(ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT) VALUES(i,'张浩',12345,to_date('2016-6-14','yyyy-mm-dd'),i,i);
       END LOOP;
       COMMIT;
       FOR j IN 1..3000 LOOP
       insert into order_details(ID,ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) VALUES(j,j,'j',10,100);
       END LOOP;
       COMMIT;
       FOR j IN 3001..6000 LOOP
       insert into order_details(ID,ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) VALUES(j,j,'j',20,100);
       END LOOP;
       COMMIT;
       FOR j IN 6001..10000 LOOP
       insert into order_details(ID,ORDER_ID,PRODUCT_ID,PRODUCT_NUM,PRODUCT_PRICE) VALUES(j,j,'j',30,100);
       END LOOP;
       COMMIT;     
END; 
    
 ##### 联合查询语句：
    SELECT
    *
    FROM orders,order_details  Where orders.order_id = order_details.order_id AND
    orders.order_date>=to_date('2016-02-14','yyyy-mm-dd')
--------  

## 实验分析
两张表均有上万条数据，从表ORDER_DETAILS跟主表ORDERS建立了主外键，orders表按照时间分成三个表空间，通过分区和不分区实验结果对比，分区表查 询的资源占比明显高出很多，查询速度快了不少。 通过分区， 查询时就不用扫描整张表，而是一块区域一块区域的去查找，这样就会快不少。

## 查看数据库的使用情况
$ sqlplus system/123@pdborcl
```sql
SQL>SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';

SQL>SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
 free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
 where  a.tablespace_name = b.tablespace_name;
```
- autoextensible是显示表空间中的数据文件是否自动增加。
- MAX_MB是指数据文件的最大容量。

