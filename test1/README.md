# 实验一：分析SQL执行计划，执行SQL语句的优化指导



## 实验内容：

- 对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。

- 首先运行和分析教材中的样例：本训练任务目的是查询两个部门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。

- 设计自己的查询语句，并作相应的分析，查询语句不能太简单。



## 实验过程

###  1.运行并分析教材样例
- <b>1.1查询一</b>

```SQL

SELECT d.department_name，count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
from hr.departments d,hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT','Sales')
GROUP BY department_name;
```
- <b>1.2查询二</b>
```SQL
SELECT d.department_name，count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
FROM hr.departments d,hr.employees e
WHERE d.department_id = e.department_id
GROUP BY department_name
HAVING d.department_name in ('IT','Sales');
```
- <b>1.3查询截图：</b>
- 1.3.1查询一截图
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/a.png)
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/b.png)
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/c.png)
- 1.3.2查询二截图
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/g.png)
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/e.png)
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/f.png)


- <b>1.4 查询分析：</b><br>
通过截图可以发现，查询语句一用时0.031s,查询语句2用时0.008s。查询语句二用时更少，更加优化。</br>
第一个查询语句group by分组  计算每一个部门，把所有的组的数据都查询过，只是没有显示。where条件限定了，多计算了除开IT，Sales的分组，所以用时更多。</br>
第二个查询只计算IT和Sales组的数据，所以时间更少，更优化。having和where类似，唯一的差别是where过滤元组，having过滤分组。</br>
- <b>1.5sqldevolper优化指导</b><br>
sqldevloper对两个查询均未给出优化指导

### 2.自我设计


- <b>2.1自己的查询语句</b>

```SQL
SELECT D.DEPARTMENT_NAME,L.CITY,COUNT(*) 
FROM EMPLOYEES E,DEPARTMENTS D,LOCATIONS L 
WHERE E.DEPARTMENT_ID = D.DEPARTMENT_ID AND D.LOCATION_ID = L.LOCATION_ID AND E.SALARY > 1000 
GROUP BY D.DEPARTMENT_NAME,L.CITY 
HAVING COUNT(*) > 2;
```
- <b>2.2自己的查询语句分析</b><br>

查询需求：列出每个部门中工资高于一千的员工数量超过二人的部门，显示部门名字，地区名称，以及总人数。<br>
详细分析：该查询由基本的SELECT、FROM和WHERE三个子句构成基本查询结构，再使用GROUP  BY对查询结果按列进行分组，使相等的分为一组。HAVING子句对分组结果进行选择，仅输出满足条件的组，并和GROUP  BY结合使用。该查询运用了等值连接E.DEPARTMENT_ID = D.DEPARTMENT_ID AND D.LOCATION_ID = L.LOCATION_ID AND E.SALARY > 1000  实现了多个关系的查询，并表示出了工资大于一千，以及表之间对应应该满足的条件，最后由HAVING子句实现人数超过2人的条件完成查询。
- <b>2.3sqldevoper执行结果</b>

![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/x.png)
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/y.png)
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/z.png)


