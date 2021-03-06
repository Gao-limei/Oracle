# 实验一：SQL语句的执行计划分析与优化指导
## 实验目的

##### 分析SQL执行计划，执行SQL语句的优化指导。理解分析SQL语句的执行计划的重要作用。

## 实验内容

   - ##### 对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。

   - ##### 首先运行和分析教材中的样例：本训练任务目的是查询两个部门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。

   - ##### 设计自己的查询语句，并作相应的分析，查询语句不能太简单。
## 实验步骤
### 对Oracle12c中的HR人力资源管理系统中的表查询分析：

##### 利用查询语句1查询：

```SQL
set autotrace on

SELECT d.department_name,count(e.job_id)as "部门总人数",
avg(e.salary)as "平均工资"
from hr.departments d,hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT','Sales')
GROUP BY d.department_name;
```

##### 运行结果如下：


![](https://raw.githubusercontent.com/Gao-limei/pictures/master/20210316111045.png)


##### 语句统计信息如下：

![](https://raw.githubusercontent.com/Gao-limei/pictures/master/20210316110655.png)

![](https://raw.githubusercontent.com/Gao-limei/pictures/master/20210316110753.png)

### 教材样例分析

1. 教材样例如下：

   - 查询1：

   ```SQL
   set autotrace on
   
   SELECT d.department_name,count(e.job_id)as "部门总人数",
   avg(e.salary)as "平均工资"
   from hr.departments d,hr.employees e
   where d.department_id = e.department_id
   and d.department_name in ('IT','Sales')
   GROUP BY d.department_name;
   ```

   - 查询2

   ```SQL
   set autotrace on
   
   SELECT d.department_name,count(e.job_id)as "部门总人数",
   avg(e.salary)as "平均工资"
   FROM hr.departments d,hr.employees e
   WHERE d.department_id = e.department_id
   GROUP BY d.department_name
   HAVING d.department_name in ('IT','Sales');
   ```

   

 #### 查询语句1分析：

![](https://raw.githubusercontent.com/Gao-limei/pictures/master/20210316110655.png)

   优化指导有：

![](https://raw.githubusercontent.com/Gao-limei/pictures/master/20210316111803.png)

   

   对查询1进行优化，添加索引如下：

![](https://raw.githubusercontent.com/Gao-limei/pictures/master/20210316112324.png)


   
 #### 查询语句2分析：

![](https://raw.githubusercontent.com/Gao-limei/pictures/master/20210316112539.png)

   ### 查询语句设计

   查询shipping部门的员工编号以及对应的工作编号（已创建索引）。

   ```sql
   set autotrace on
   SELECT d.department_name,e.EMPLOYEE_id,e.job_id
   from hr.departments d,hr.employees e
   where d.department_id = e.department_id
   and d.department_name in ('Shipping')
   GROUP BY department_name,e.EMPLOYEE_id,e.job_id;
   ```

![](https://raw.githubusercontent.com/Gao-limei/pictures/master/20210316112652.png)

## 实验总结
### 在本次实验中分析了SQL的执行计划，执行了SQL语句的优化指导，并理解了分析SQL语句的执行计划的重要作用。

### 本次实验内容首先对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。然后运行和分析教材中的样例，得出结论为两个查询结果是一样的，但效率不相同。最后设计自己的查询语句，并作相应的分析。

### 在本次试验中，我了解到了SQL基本的查询语句操作，和查询的优化，并有了自己新的学习领悟，对SQL语句的使用也有了更进一步的熟悉与理解。


