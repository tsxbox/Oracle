

## 实验4：对象管理
### 1.实验目的：
了解Oracle表和视图的概念，学习使用SQL语句Create Table创建表，学习Select语句插入，修改，删除以及查询数据，学习使用SQL语句创建视图，学习部分存储过程和触发器的使用。

### 2.实验场景：
假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。


### 3.实验过程：
我的用户名为user_tsx
- 分配权限

```sql
 -- QUOTAS
ALTER USER USER_TSX QUOTA UNLIMITED ON USERS;
ALTER USER USER_TSX QUOTA UNLIMITED ON USERS02;
ALTER USER USER_TSX ACCOUNT UNLOCK;

-- ROLES
GRANT "CONNECT" TO USER_TSX WITH ADMIN OPTION;
GRANT "RESOURCE" TO USER_TSX WITH ADMIN OPTION;
ALTER USER USER_TSX DEFAULT ROLE "CONNECT","RESOURCE";

-- SYSTEM PRIVILEGES
GRANT CREATE VIEW TO USER_TSX WITH ADMIN OPTION;
```

- 登录用户USER_TSX，创建部门表语句及其相关语句（DEPARTMENTS 表空间：USERS）。

```sql
CREATE TABLE DEPARTMENTS
(
  DEPARTMENT_ID NUMBER(6, 0) NOT NULL
, DEPARTMENT_NAME VARCHAR2(40 BYTE) NOT NULL
, CONSTRAINT DEPARTMENTS_PK PRIMARY KEY
(
DEPARTMENT_ID
 )
USING INDEX
(
  CREATE UNIQUE INDEX DEPARTMENTS_PK ON DEPARTMENTS (DEPARTMENT_ID ASC)
  NOLOGGING
  TABLESPACE USERS
  PCTFREE 10
  INITRANS 2
  STORAGE
  (
    INITIAL 65536
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
  )
  NOPARALLEL
)
ENABLE
)
NOLOGGING
TABLESPACE USERS
PCTFREE 10
INITRANS 1
STORAGE
(
INITIAL 65536
NEXT 1048576
MINEXTENTS 1
MAXEXTENTS UNLIMITED
BUFFER_POOL DEFAULT
)
NOCOMPRESS NO INMEMORY NOPARALLEL;

Table DEPARTMENTS 已创建。
```


- 创建员工表语句及其相关sql语句（EMPLOYEES 表空间：USERS）。

```sql
CREATE TABLE EMPLOYEES
(
EMPLOYEE_ID NUMBER(6, 0) NOT NULL
, NAME VARCHAR2(40 BYTE) NOT NULL
, EMAIL VARCHAR2(40 BYTE)
, PHONE_NUMBER VARCHAR2(40 BYTE)
, HIRE_DATE DATE NOT NULL
, SALARY NUMBER(8, 2)
, MANAGER_ID NUMBER(6, 0)
, DEPARTMENT_ID NUMBER(6, 0)
, PHOTO BLOB
, CONSTRAINT EMPLOYEES_PK PRIMARY KEY
(
EMPLOYEE_ID
)
USING INDEX
(
  CREATE UNIQUE INDEX EMPLOYEES_PK ON EMPLOYEES (EMPLOYEE_ID ASC)
  NOLOGGING
  TABLESPACE USERS
  PCTFREE 10
  INITRANS 2
  STORAGE
  (
    INITIAL 65536
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
  )
  NOPARALLEL
)
ENABLE
)
NOLOGGING
TABLESPACE USERS
PCTFREE 10
INITRANS 1
STORAGE
(
INITIAL 65536
NEXT 1048576
MINEXTENTS 1
MAXEXTENTS UNLIMITED
BUFFER_POOL DEFAULT
)
NOCOMPRESS
NO INMEMORY
NOPARALLEL
LOB (PHOTO) STORE AS SYS_LOB0000092017C00009$$
(
ENABLE STORAGE IN ROW
CHUNK 8192
NOCACHE
NOLOGGING
TABLESPACE USERS
STORAGE
(
INITIAL 106496
NEXT 1048576
MINEXTENTS 1
MAXEXTENTS UNLIMITED
BUFFER_POOL DEFAULT
)
);

Table EMPLOYEES 已创建。

CREATE INDEX EMPLOYEES_INDEX1_NAME ON EMPLOYEES (NAME ASC)
NOLOGGING
TABLESPACE USERS
PCTFREE 10
INITRANS 2
STORAGE
(
INITIAL 65536
NEXT 1048576
MINEXTENTS 1
MAXEXTENTS UNLIMITED
BUFFER_POOL DEFAULT
)

NOPARALLEL;
Index EMPLOYEES_INDEX1_NAME 已创建。


ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_FK1 FOREIGN KEY
(
DEPARTMENT_ID
)
REFERENCES DEPARTMENTS
(
DEPARTMENT_ID
)
ENABLE;
Table EMPLOYEES已变更。

ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_FK2 FOREIGN KEY
(
MANAGER_ID
)
REFERENCES EMPLOYEES
(
EMPLOYEE_ID
)
ON DELETE SET NULL ENABLE;

Table EMPLOYEES已变更。

ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_CHK1 CHECK
(SALARY>0)
ENABLE;

Table EMPLOYEES已变更。

ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_CHK2 CHECK
(EMPLOYEE_ID<>MANAGER_ID)
ENABLE;

Table EMPLOYEES已变更。

ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_EMPLOYEE_MANAGER_ID CHECK
(MANAGER_ID<>EMPLOYEE_ID)
ENABLE;

Table EMPLOYEES已变更。

ALTER TABLE EMPLOYEES
ADD CONSTRAINT EMPLOYEES_SALARY CHECK
(SALARY>0)
ENABLE;

Table EMPLOYEES已变更。
```

- 登录user_tsx用户，创建产品表语句及其相关sql语句（PRODUCTS 表空间：USERS）
```sql
CREATE TABLE PRODUCTS
(
PRODUCT_NAME VARCHAR2(40 BYTE) NOT NULL
, PRODUCT_TYPE VARCHAR2(40 BYTE) NOT NULL
, CONSTRAINT PRODUCTS_PK PRIMARY KEY
(
PRODUCT_NAME
)
ENABLE
)
LOGGING
TABLESPACE "USERS"
PCTFREE 10
INITRANS 1
STORAGE
(
INITIAL 65536
NEXT 1048576
MINEXTENTS 1
MAXEXTENTS 2147483645
BUFFER_POOL DEFAULT
);

Table PRODUCTS 已创建。

ALTER TABLE PRODUCTS
ADD CONSTRAINT PRODUCTS_CHK1 CHECK
(PRODUCT_TYPE IN ('耗材', '手机', '电脑'))
ENABLE;
Table PRODUCTS已变更。
```

- 登录user_tsx用户，创建订单表语句及其相关sql语句（ORDERS 分区表：USERS,USERS02）
```
    --  DDL for Table ORDER_ID_TEMP
    CREATE GLOBAL TEMPORARY TABLE "ORDER_ID_TEMP"
    (	"ORDER_ID" NUMBER(10,0) NOT NULL ENABLE,
	 CONSTRAINT "ORDER_ID_TEMP_PK" PRIMARY KEY ("ORDER_ID") ENABLE
    ) ON COMMIT DELETE ROWS ;

    COMMENT ON TABLE "ORDER_ID_TEMP"  IS '用于触发器存储临时ORDER_ID';
    
    Comment 已创建。
    
    --  DDL for Table ORDERS
    --------------------------------------------------------
    CREATE TABLE ORDERS
    (
    ORDER_ID NUMBER(10, 0) NOT NULL
    , CUSTOMER_NAME VARCHAR2(40 BYTE) NOT NULL
    , CUSTOMER_TEL VARCHAR2(40 BYTE) NOT NULL
    , ORDER_DATE DATE NOT NULL
    , EMPLOYEE_ID NUMBER(6, 0) NOT NULL
    , DISCOUNT NUMBER(8, 2) DEFAULT 0
    , TRADE_RECEIVABLE NUMBER(8, 2) DEFAULT 0
    )
    TABLESPACE USERS
    PCTFREE 10
    INITRANS 1
    STORAGE
    (
    BUFFER_POOL DEFAULT
    )
    NOCOMPRESS
    NOPARALLEL
    PARTITION BY RANGE (ORDER_DATE)
    (
    PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
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
    , PARTITION PARTITION_BEFORE_2017 VALUES LESS THAN (TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
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
    );
    
    Table ORDERS 已创建。

    --创建本地分区索引ORDERS_INDEX_DATE：
    CREATE INDEX ORDERS_INDEX_DATE ON ORDERS (ORDER_DATE ASC)
    LOCAL
    (
    PARTITION PARTITION_BEFORE_2016
    TABLESPACE USERS
    PCTFREE 10
    INITRANS 2
    STORAGE
    (
      INITIAL 8388608
      NEXT 1048576
      MINEXTENTS 1
      MAXEXTENTS UNLIMITED
      BUFFER_POOL DEFAULT
    )
    NOCOMPRESS
    , PARTITION PARTITION_BEFORE_2017
    TABLESPACE USERS02
    PCTFREE 10
    INITRANS 2
    STORAGE
    (
      INITIAL 8388608
      NEXT 1048576
      MINEXTENTS 1
      MAXEXTENTS UNLIMITED
      BUFFER_POOL DEFAULT
    )
    NOCOMPRESS
    )
    STORAGE
    (
    BUFFER_POOL DEFAULT
    )
    NOPARALLEL;
    
    Index ORDERS_INDEX_DATE 已创建。

    CREATE INDEX ORDERS_INDEX_CUSTOMER_NAME ON ORDERS (CUSTOMER_NAME ASC)
    NOLOGGING
    TABLESPACE USERS
    PCTFREE 10
    INITRANS 2
    STORAGE
    (
    INITIAL 65536
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
    )
    NOPARALLEL;
    
    Index ORDERS_INDEX_CUSTOMER_NAME 已创建。

    CREATE UNIQUE INDEX ORDERS_PK ON ORDERS (ORDER_ID ASC)
    GLOBAL PARTITION BY HASH (ORDER_ID)
    (
    PARTITION INDEX_PARTITION1 TABLESPACE USERS
    NOCOMPRESS
    , PARTITION INDEX_PARTITION2 TABLESPACE USERS02
    NOCOMPRESS
    )
    NOLOGGING
    TABLESPACE USERS
    PCTFREE 10
    INITRANS 2
    STORAGE
    (
    INITIAL 65536
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
    )
    NOPARALLEL;
    
    INDEX ORDERS_PK 已创建。

    ALTER TABLE ORDERS
    ADD CONSTRAINT ORDERS_PK PRIMARY KEY
    (
    ORDER_ID
    )
    USING INDEX ORDERS_PK
    ENABLE;
    
    Table ORDERS已变更。

    ALTER TABLE ORDERS
    ADD CONSTRAINT ORDERS_FK1 FOREIGN KEY
    (
    EMPLOYEE_ID
    )
    REFERENCES EMPLOYEES
    (
    EMPLOYEE_ID
    )
    ENABLE;
    
    Table ORDERS已变更。
 ```
    
 -（3）登录user_tsx用户，创建产品表语句及其相关sql语句（PRODUCTS 表空间：USERS）。
    CREATE TABLE PRODUCTS
    (
    PRODUCT_NAME VARCHAR2(40 BYTE) NOT NULL
    , PRODUCT_TYPE VARCHAR2(40 BYTE) NOT NULL
    , CONSTRAINT PRODUCTS_PK PRIMARY KEY
    (
    PRODUCT_NAME
    )
    ENABLE
    )
    LOGGING
    TABLESPACE "USERS"
    PCTFREE 10
    INITRANS 1
    STORAGE
    (
    INITIAL 65536
    NEXT 1048576
    MINEXTENTS 1
    MAXEXTENTS 2147483645
    BUFFER_POOL DEFAULT
    );
    
    Table PRODUCTS 已创建。

    ALTER TABLE PRODUCTS
    ADD CONSTRAINT PRODUCTS_CHK1 CHECK
    (PRODUCT_TYPE IN ('耗材', '手机', '电脑'))
    ENABLE;
    
    Table PRODUCTS已变更。

-（5）登录new_lyk用户，创建订单详表语句及其相关sql语句（ORDER_DETAILS 分区表：USERS,USERS02）。
```sql
    CREATE TABLE ORDER_DETAILS
    (
    ID NUMBER(10, 0) NOT NULL
    , ORDER_ID NUMBER(10, 0) NOT NULL
    , PRODUCT_NAME VARCHAR2(40 BYTE) NOT NULL
    , PRODUCT_NUM NUMBER(8, 2) NOT NULL
    , PRODUCT_PRICE NUMBER(8, 2) NOT NULL
    , CONSTRAINT ORDER_DETAILS_FK1 FOREIGN KEY
    (
    ORDER_ID
    )
    REFERENCES ORDERS
    (
    ORDER_ID
    )
    ENABLE
    )
    TABLESPACE USERS
    PCTFREE 10
    INITRANS 1
    STORAGE
    (
    BUFFER_POOL DEFAULT
    )
    NOCOMPRESS
    NOPARALLEL
    PARTITION BY REFERENCE (ORDER_DETAILS_FK1)
    (
    PARTITION PARTITION_BEFORE_2016
    NOLOGGING
    TABLESPACE USERS --必须指定表空间，否则会将分区存储在用户的默认表空间中
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
    NOCOMPRESS NO INMEMORY
    )
    ;
    
    Table ORDER_DETAILS 已创建。

    CREATE UNIQUE INDEX ORDER_DETAILS_PK ON ORDER_DETAILS (ID ASC)
    NOLOGGING
    TABLESPACE USERS
    PCTFREE 10
    INITRANS 2
    STORAGE
    (
    INITIAL 65536
     NEXT 1048576
     MINEXTENTS 1
    MAXEXTENTS UNLIMITED
    BUFFER_POOL DEFAULT
    )
    NOPARALLEL;
    
    INDEX ORDER_DETAILS_PK 已创建。

    ALTER TABLE ORDER_DETAILS
    ADD CONSTRAINT ORDER_DETAILS_PK PRIMARY KEY
    (
     ID
    )
    USING INDEX ORDER_DETAILS_PK
    ENABLE;
    
    Table ORDER_DETAILS已变更。

    --这个索引是必须的，可以使整个订单的详单存放在一起
    CREATE INDEX ORDER_DETAILS_ORDER_ID ON ORDER_DETAILS (ORDER_ID)
    GLOBAL PARTITION BY HASH (ORDER_ID)
    (
     PARTITION INDEX_PARTITION1 TABLESPACE USERS
     NOCOMPRESS
    , PARTITION INDEX_PARTITION2 TABLESPACE USERS02
     NOCOMPRESS
    );
    
    Index ORDER_DETAILS_ORDER_ID 已创建。

    ALTER TABLE ORDER_DETAILS
    ADD CONSTRAINT ORDER_DETAILS_PRODUCT_NUM CHECK
    (Product_Num>0)
    ENABLE;
    
    Table ORDER_DETAILS已变更。
```

二、第2步：插入相关数据sql语句及其查询相关数据sql语句。 
---------
##### （1）录入数据语句及其相关语句。
    注：要求至少有1万个订单，每个订单至少有4个详单。至少有两个部门，每个部门至少有1个员工，其中只有一个人没有领导，一个领导至少有一个下属，并且它的下属是另一个人的领导（比如A领导B，B领导C）。
    --------------------------------------------------------
    --  DDL for View VIEW_ORDER_DETAILS
    --------------------------------------------------------
    CREATE OR REPLACE FORCE EDITIONABLE VIEW "VIEW_ORDER_DETAILS" ("ID", "ORDER_ID", "CUSTOMER_NAME", "CUSTOMER_TEL", "ORDER_DATE",     "PRODUCT_TYPE", "PRODUCT_NAME", "PRODUCT_NUM", "PRODUCT_PRICE") AS
     SELECT
    d.ID,
    o.ORDER_ID,
    o.CUSTOMER_NAME,o.CUSTOMER_TEL,o.ORDER_DATE,
    p.PRODUCT_TYPE,
     d.PRODUCT_NAME,
    d.PRODUCT_NUM,
    d.PRODUCT_PRICE
    FROM ORDERS o,ORDER_DETAILS d,PRODUCTS p where d.ORDER_ID=o.ORDER_ID and d.PRODUCT_NAME=p.PRODUCT_NAME;
     /
     
     View "VIEW_ORDER_DETAILS" 已创建。

    --插入DEPARTMENTS，EMPLOYEES数据
    INSERT INTO DEPARTMENTS(DEPARTMENT_ID,DEPARTMENT_NAME) values (1,'总经办');
    INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
     VALUES (1,'李董事长',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,NULL,1);

    INSERT INTO DEPARTMENTS(DEPARTMENT_ID,DEPARTMENT_NAME) values (11,'销售部1');
    INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
    VALUES (11,'张总',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,1,1);
    INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
    VALUES (111,'吴经理',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,11,11);
    INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
    VALUES (112,'白经理',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,11,11);

    INSERT INTO DEPARTMENTS(DEPARTMENT_ID,DEPARTMENT_NAME) values (12,'销售部2');
    INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
    VALUES (12,'王总',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,1,1);
    INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
     VALUES (121,'赵经理',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,12,12);
    INSERT INTO EMPLOYEES(EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID)
    VALUES (122,'刘经理',NULL,NULL,to_date('2010-1-1','yyyy-mm-dd'),50000,12,12);


     insert into products (product_name,product_type) values ('computer1','电脑');
    insert into products (product_name,product_type) values ('computer2','电脑');
    insert into products (product_name,product_type) values ('computer3','电脑');

    insert into products (product_name,product_type) values ('phone1','手机');
    insert into products (product_name,product_type) values ('phone2','手机');
    insert into products (product_name,product_type) values ('phone3','手机');

    insert into products (product_name,product_type) values ('paper1','耗材');
    insert into products (product_name,product_type) values ('paper2','耗材');
    insert into products (product_name,product_type) values ('paper3','耗材');


    --批量插入订单数据，注意ORDERS.TRADE_RECEIVABLE（订单应收款）的自动计算,注意插入数据的速度
    --2千万条记录，插入的时间是：18100秒（约5小时）
    declare
    dt date;
    m number(8,2);
    V_EMPLOYEE_ID NUMBER(6);
    v_order_id number(10);
    v_name varchar2(100);
    v_tel varchar2(100);
    v number(10,2);

     begin
    for i in 1..10000
    loop
      if i mod 2 =0 then
      dt:=to_date('2015-3-2','yyyy-mm-dd')+(i mod 60);
    else
      dt:=to_date('2016-3-2','yyyy-mm-dd')+(i mod 60);
    end if;
    V_EMPLOYEE_ID:=CASE I MOD 6 WHEN 0 THEN 11 WHEN 1 THEN 111 WHEN 2 THEN 112
                                WHEN 3 THEN 12 WHEN 4 THEN 121 ELSE 122 END;
    --插入订单
    v_order_id:=SEQ_ORDER_ID.nextval; --应该将SEQ_ORDER_ID.nextval保存到变量中。
    v_name := 'aa'|| 'aa';
    v_name := 'zhang' || i;
    v_tel := '139888883' || i;
    insert /*+append*/ into ORDERS (ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT)
      values (v_order_id,v_name,v_tel,dt,V_EMPLOYEE_ID,dbms_random.value(100,0));
    --插入订单y一个订单包括4个产品
    v:=dbms_random.value(10000,4000);
    v_name:='computer'|| (i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (SEQ_ORDER_DETAILS_ID.NEXTVAL,v_order_id,v_name,2,v);
    v:=dbms_random.value(1000,50);
    v_name:='paper'|| (i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (SEQ_ORDER_DETAILS_ID.NEXTVAL,v_order_id,v_name,3,v);
    v:=dbms_random.value(9000,2000);
    v_name:='phone'|| (i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (SEQ_ORDER_DETAILS_ID.NEXTVAL,v_order_id,v_name,1,v);
    v:=dbms_random.value(8000,3000);
    v_name:= 'computer'||(i mod 3 + 1);
    insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
      values (SEQ_ORDER_DETAILS_ID.NEXTVAL,v_order_id,v_name,4,v);
    --在触发器关闭的情况下，需要手工计算每个订单的应收金额：
    select sum(PRODUCT_NUM*PRODUCT_PRICE) into m from ORDER_DETAILS where ORDER_ID=v_order_id;
    if m is null then
     m:=0;
    end if;
    UPDATE ORDERS SET TRADE_RECEIVABLE = m - discount WHERE ORDER_ID=v_order_id;
    IF I MOD 1000 =0 THEN
      commit; --每次提交会加快插入数据的速度
    END IF;
    end loop;
     --统计用户的所有表，所需时间很长：2千万行数据，需要1600秒，该语句可选
    --dbms_stats.gather_schema_stats(User,estimate_percent=>100,cascade=> TRUE); --estimate_percent采样行的百分比
    end;
     /
     
     PL/SQL 过程已成功完成

    ALTER TRIGGER "ORDERS_TRIG_ROW_LEVEL" ENABLE;
    --Trigger "ORDERS_TRIG_ROW_LEVEL"已变更。
    
     ALTER TRIGGER "ORDER_DETAILS_SNTNS_TRIG" ENABLE;
     --Trigger "ORDER_DETAILS_SNTNS_TRIG"已变更。
     
     ALTER TRIGGER "ORDER_DETAILS_ROW_TRIG" ENABLE;
     --Trigger "ORDER_DETAILS_ROW_TRIG"已变更。

    --最后动态增加一个PARTITION_BEFORE_2018分区：
    ALTER TABLE ORDERS
    ADD PARTITION PARTITION_BEFORE_2018 VALUES LESS THAN (TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'));
    
    --Table ORDERS已变更
    

    ALTER INDEX ORDERS_INDEX_DATE
    MODIFY PARTITION PARTITION_BEFORE_2018
    NOCOMPRESS;
    
    --Index ORDERS_INDEX_DATE已变更。
    
##### （2）序列的应用语句及其相关语句。
    注：插入ORDERS和ORDER_DETAILS 两个表的数据时，主键ORDERS.ORDER_ID, ORDER_DETAILS.ID的值必须通过序列SEQ_ORDER_ID和SEQ_ORDER_ID取得，不能手工输入一个数字。
    --------------------------------------------------------
    --  DDL for Sequence SEQ_ORDER_ID
    --------------------------------------------------------
    CREATE SEQUENCE  "SEQ_ORDER_ID"  MINVALUE 1 MAXVALUE 9999999999 INCREMENT BY 1 START WITH 1 CACHE 2000 ORDER  NOCYCLE  NOPARTITION ;
    --------------------------------------------------------
    --  DDL for Sequence SEQ_ORDER_DETAILS_ID
    --------------------------------------------------------
    CREATE SEQUENCE  "SEQ_ORDER_DETAILS_ID"  MINVALUE 1 MAXVALUE 9999999999 INCREMENT BY 1 START WITH 1 CACHE 2000 ORDER  NOCYCLE           NOPARTITION ;
    
    --Sequence "SEQ_ORDER_ID" 已创建。


    --Sequence "SEQ_ORDER_DETAILS_ID" 已创建。

    
##### （3）触发器的应用语句及其相关语句。
    注：维护ORDER_DETAILS的数据时（insert,delete,update）要同步更新ORDERS表订单应收货款ORDERS.Trade_Receivable的值。
    
    --创建3个触发器
    --------------------------------------------------------
    --  DDL for Trigger ORDERS_TRIG_ROW_LEVEL
    --------------------------------------------------------
     CREATE OR REPLACE EDITIONABLE TRIGGER "ORDERS_TRIG_ROW_LEVEL"
    BEFORE INSERT OR UPDATE OF DISCOUNT ON "ORDERS"
    FOR EACH ROW --行级触发器
    declare
    m number(8,2);
    BEGIN
    if inserting then
        :new.TRADE_RECEIVABLE := - :new.discount;
    else
      select sum(PRODUCT_NUM*PRODUCT_PRICE) into m from ORDER_DETAILS where ORDER_ID=:old.ORDER_ID;
      if m is null then
        m:=0;
      end if;
      :new.TRADE_RECEIVABLE := m - :new.discount;
    end if;
    END;
    /
    
    Trigger ORDERS_TRIG_ROW_LEVEL 已编译
    
    --批量插入订单数据之前，禁用触发器
    ALTER TRIGGER "ORDERS_TRIG_ROW_LEVEL" DISABLE;
    
    Trigger "ORDERS_TRIG_ROW_LEVEL"已变更。


     --------------------------------------------------------
    --  DDL for Trigger ORDER_DETAILS_ROW_TRIG
    --------------------------------------------------------

    CREATE OR REPLACE EDITIONABLE TRIGGER "ORDER_DETAILS_ROW_TRIG"
    AFTER DELETE OR INSERT OR UPDATE  ON ORDER_DETAILS
    FOR EACH ROW
    BEGIN
    --DBMS_OUTPUT.PUT_LINE(:NEW.ORDER_ID);
    IF :NEW.ORDER_ID IS NOT NULL THEN
     MERGE INTO ORDER_ID_TEMP A
    USING (SELECT 1 FROM DUAL) B
    ON (A.ORDER_ID=:NEW.ORDER_ID)
    WHEN NOT MATCHED THEN
      INSERT (ORDER_ID) VALUES(:NEW.ORDER_ID);
    END IF;
    IF :OLD.ORDER_ID IS NOT NULL THEN
    MERGE INTO ORDER_ID_TEMP A
    USING (SELECT 1 FROM DUAL) B
    ON (A.ORDER_ID=:OLD.ORDER_ID)
    WHEN NOT MATCHED THEN
      INSERT (ORDER_ID) VALUES(:OLD.ORDER_ID);
     END IF;
    END;
     /
     
     Trigger ORDER_DETAILS_ROW_TRIG 已编译
     
     ALTER TRIGGER "ORDER_DETAILS_ROW_TRIG" DISABLE;
     
     Trigger "ORDER_DETAILS_ROW_TRIG"已变更。
     
     --------------------------------------------------------
    --  DDL for Trigger ORDER_DETAILS_SNTNS_TRIG
    --------------------------------------------------------

    CREATE OR REPLACE EDITIONABLE TRIGGER "ORDER_DETAILS_SNTNS_TRIG"
    AFTER DELETE OR INSERT OR UPDATE ON ORDER_DETAILS
    declare
     m number(8,2);
    BEGIN
     FOR R IN (SELECT ORDER_ID FROM ORDER_ID_TEMP)
    LOOP
    --DBMS_OUTPUT.PUT_LINE(R.ORDER_ID);
    select sum(PRODUCT_NUM*PRODUCT_PRICE) into m from ORDER_DETAILS
      where ORDER_ID=R.ORDER_ID;
    if m is null then
      m:=0;
    end if;
    UPDATE ORDERS SET TRADE_RECEIVABLE = m - discount
      WHERE ORDER_ID=R.ORDER_ID;
     END LOOP;
     --delete from ORDER_ID_TEMP; --这句话很重要，否则可能一直不释放空间，后继插入会非常慢。
    END;
    
    Trigger ORDER_DETAILS_SNTNS_TRIG 已编译
    
    /
    ALTER TRIGGER "ORDER_DETAILS_SNTNS_TRIG" DISABLE;
    
    Trigger "ORDER_DETAILS_SNTNS_TRIG"已变更。
    
##### （4）查询数据语句。

###### 1.查询某个员工的信息sql语句及其结果如下图所示。
     SELECT * FROM EMPLOYEES WHERE employee_ID = 11;
   ![image](https://github.com/tsxbox/Oracle/blob/master/test4/1.png)  
###### 2.递归查询某个员工及其所有下属，子下属员工SQL语句及其结果如下图所示。
     WITH A (EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID) AS
     (SELECT EMPLOYEE_ID,NAME,EMAIL,PHONE_NUMBER,HIRE_DATE,SALARY,MANAGER_ID,DEPARTMENT_ID
     FROM employees WHERE employee_ID = 11
     UNION ALL
     SELECT B.EMPLOYEE_ID,B.NAME,B.EMAIL,B.PHONE_NUMBER,B.HIRE_DATE,B.SALARY,B.MANAGER_ID,B.DEPARTMENT_ID
     FROM A, employees B WHERE A.EMPLOYEE_ID = B.MANAGER_ID)
     SELECT * FROM A;
  ![image](https://github.com/tsxbox/Oracle/blob/master/test4/2.png)  
 
###### 3.查询订单表SQL语句及其结果如下图2-4-3所示，并且包括订单的订单应收货款: Trade_Receivable= sum(订单详单表.ProductNum*订单详单表.ProductPrice)- Discount。
    SELECT * FROM orders WHERE order_id=15042;
    SELECT * FROM order_details WHERE order_id=15042;
 ![image](https://github.com/tsxbox/Oracle/blob/master/test4/3.png)  

###### 4.查询订单详表SQL语句及其结果如下图2-4-4所示，要求显示订单的客户名称和客户电话，产品类型用汉字描述。
     SELECT CUSTOMER_NAME as 客户名称,CUSTOMER_TEL as 客户电话,PRODUCT_NAME as 产品类型 FROM order_details,orders WHERE order_details.order_id=orders.order_id And orders.order_id=15042;
  ![image](https://github.com/tsxbox/Oracle/blob/master/test4/4.png)  
   
     
###### 5.查询出所有空订单SQL语句及其结果如下图2-4-5锁死，即没有订单详单的订单。
    SELECT * FROM orders WHERE order_id NOT IN (SELECT order_id FROM order_details);
    或者
    SELECT orders.* FROM orders LEFT JOIN order_details ON orders.order_id = order_details.order_id WHERE order_details.order_id is NULL;
![image](https://github.com/tsxbox/Oracle/blob/master/test4/5.png)   
    
###### 6.查询部门表SQL语句及其结果如下图2-4-6所示，同时显示部门的负责人姓名。
    SELECT departments.department_id,departments.department_name,employees.name FROM departments,employees WHERE employees.manager_id = departments.department_id;
![image](https://github.com/tsxbox/Oracle/blob/master/test4/6.png)  

###### 7.查询部门表SQL语句，统计每个部门的销售总金额。    
    SELECT departments.department_name as 部门名称,SUM(ORDERS.trade_receivable) as 总金额 FROM departments,employees,orders
    WHERE departments.department_id=employees.department_id AND employees.employee_id=orders.employee_id group by departments.department_name;
![image](https://github.com/tsxbox/Oracle/blob/master/test4/7.png)  
   


## 实验总结
   通过本次实验熟悉了SQL语句Create Table创建表，
   学习了Select语句插入，修改，删除以及查询数据，
   以及使用SQL语句创建视图，学习部分存储过程和触发器的使用。
