# 实验一：分析SQL执行计划，执行SQL语句的优化指导



## 实验内容：

- 对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。

- 首先运行和分析教材中的样例：本训练任务目的是查询两个部门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。

- 设计自己的查询语句，并作相应的分析，查询语句不能太简单。



## 实验过程



- <b>1.1查询一</b>

```SQL

SELECT d.department_name，count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
from hr.departments d,hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT','Sales')
GROUP BY department_name;
```
- -1.2查询二-
```SQL
SELECT d.department_name，count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
FROM hr.departments d,hr.employees e
WHERE d.department_id = e.department_id
GROUP BY department_name
HAVING d.department_name in ('IT','Sales');
```
- -1.3查询截图：-
- 1.3.1查询一截图
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/a.png)
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/b.png)
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/c.png)
- 1.3.2查询二截图
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/g.png)
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/e.png)
![IMAGE](https://raw.githubusercontent.com/tsxbox/Oracle/master/f.png)


- -1.4 查询分析：-<br>
通过截图可以发现，查询语句一用时0.031s,查询语句2用时0.008s。查询语句二用时更少，更加优化。</br>
第一个查询语句group by分组  计算每一个部门，把所有的组的数据都查询过，只是没有显示。where条件限定了，多计算了除开IT，Sales的分组，所以用时更多。</br>
第二个查询只计算IT和Sales组的数据，所以时间更少，更优化。having和where类似，唯一的差别是where过滤行，having过滤组。</br>
- 1.5 -sqldevolper优化指导-<br>
sqldevloper对两个查询均未给出优化指导

- -2.1自己的查询语句-

- -2.2自己的查询语句分析-
- -2.3sqldevoper执行结果-



