# 实验2：用户管理 - 掌握管理角色、权根、用户的能力，并在用户之间共享对象。

## 实验内容：
Oracle有一个开发者角色resource，可以创建表、过程、触发器等对象，但是不能创建视图。本训练要求：
- 在pdborcl插接式数据中创建一个新的本地角色con_res_view，该角色包含connect和resource角色，同时也包含CREATE VIEW权限，这样任何拥有con_res_view的用户就同时拥有这三种权限。
- 创建角色之后，再创建用户new_user，给用户分配表空间，设置限额为50M，授予con_res_view角色。
- 最后测试：用新用户new_user连接数据库、创建表，插入数据，创建视图，查询表和视图的数据。

## 实验步骤

###  1.第1步：以system登录到pdborcl，创建角色con_tsx_view和用户user_tsx，并授权和分配空间：

```sql
$ sqlplus system/123@pdborcl
SQL> CREATE ROLE con_tsx_view;
Role created.
SQL> GRANT connect,resource,CREATE VIEW TO con_tsx_view;
Grant succeeded.
SQL> CREATE USER user_tsx IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;
User created.
SQL> ALTER USER new_user QUOTA 50M ON users;
User altered.
SQL> GRANT con_res_view TO new_user;
Grant succeeded.
SQL> exit
```
![IMAGE](https://github.com/tsxbox/Oracle/blob/master/test2/one.png)

###  2.第2步：新用户user-tsx连接到 pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。
```sql
$ sqlplus /123@pdborcl
SQL> show user;
USER is "USER_TSX"
SQL> CREATE TABLE mytable (id number,name varchar(50));
Table created.
SQL> INSERT INTO mytable(id,name)VALUES(1,'zhang');
1 row created.
SQL> INSERT INTO mytable(id,name)VALUES (2,'wang');
1 row created.
SQL> CREATE VIEW myview AS SELECT name FROM mytable;
View created.
SQL> SELECT * FROM myview;
NAME
--------------------------------------------------
zhang
wang
SQL> GRANT SELECT ON myview TO hr;
Grant succeeded.
SQL>exit
```

![IMAGE](https://github.com/tsxbox/Oracle/blob/master/test2/two.png)

###  2.第3步：用户hr连接到pdborcl，查询user_tsx授予它的视图myview
```sql
$ sqlplus hr/123@pdborcl
SQL> SELECT * FROM user_tsx.myview;
NAME
--------------------------------------------------
zhang
wang
SQL> exit
```
![IMAGE](https://github.com/tsxbox/Oracle/blob/master/test2/three.png)
