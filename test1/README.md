#                                         实验一：分析SQL执行计划，执行SQL语句的优化指导
实验内容：

对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。
首先运行和分析教材中的样例：本训练任务目的是查询两个部门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。
设计自己的查询语句，并作相应的分析，查询语句不能太简单。
教材中的查询语句

### 查询1：

SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
from hr.departments d，hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT'，'Sales')
GROUP BY department_name;
### 实验结果截图：![](https://github.com/rusellwestbook/oracle/blob/master/test1/J%7DL%602X%25WIUT9%5DDQD~WVXENR.png)
### 优化指导截图：![](https://github.com/rusellwestbook/oracle/blob/master/test1/3CTNM~6GPIDIOT847IAC5%40B.png)
### 查询2：
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
FROM hr.departments d，hr.employees e
WHERE d.department_id = e.department_id
GROUP BY department_name
HAVING d.department_name in ('IT'，'Sales');
### 实验结果截图：![](https://github.com/rusellwestbook/oracle/blob/master/test1/3CTNM~6GPIDIOT847IAC5%40B.png)
### 优化指导截图：![](https://github.com/rusellwestbook/oracle/blob/master/test1/R%7BLFDCWCF2HM1TEPF%60O0NXL.png)
#### 以上两个查询第二个查询是最优的，第二个语句通过group by 和having的联合使用使查询语句的逻辑更加紧密。HAVING语句通常与GROUP BY语句联合使用，用来过滤由GROUP BY语句返回的记录集。HAVING语句的存在弥补了WHERE关键字不能与聚合函数联合使用的不足。
#### 我的设计的查询：我的查询语句用到了右外连接，右外连接与左外连接类似，只是右端表中的所有元组都列出，限制左端表的数据必须满足连接条件，而不管右端表中的数据是否满足连接条件，均输出表中的内容。
SELECT d.department_name，count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
FROM hr.departments d right outer join hr.employees e
on d.department_id = e.department_id
and d.department_name in ('IT'，'Sales')
GROUP BY department_name;
### 实验结果截图：![](https://github.com/rusellwestbook/oracle/blob/master/test1/bymyself.png)



