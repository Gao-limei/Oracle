# 实验三：创建分区表
## 实验目的

##### 掌握分区表的创建方法，掌握各种分区方式的使用场景。

## 实验内容
##### Oracle有一个开发者角色resource，可以创建表、过程、触发器等对象，但是不能创建视图。本训练要求：

   - ##### 本实验使用3个表空间：USERS,USERS02,USERS03。在表空间中创建两张表：订单表(orders)与订单详表(order_details)。

   - ##### 使用你自己的账号创建本实验的表，表创建在上述3个分区，自定义分区策略。

   - ##### 你需要使用system用户给你自己的账号分配上述分区的使用权限。你需要使用system用户给你的用户分配可以查询执行计划的权限。
  
   - ##### 表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。
   - ##### 写出插入数据的语句和查询数据的语句，并分析语句的执行计划。
  
   - ##### 进行分区与不分区的对比实验。


## 实验步骤

- #### 第1步：首先创建自己的账号limei_user，然后以system身份登录
```SQL
[student@deep02 ~]$sqlplus system/123@localhost/pdborcl
SQL>ALTER USER limei_user QUOTA UNLIMITED ON USERS;
SQL>ALTER USER limei_user QUOTA UNLIMITED ON USERS02;
SQL>ALTER USER limei_user QUOTA UNLIMITED ON USERS03;
SQL>exit
```

- #### 第2步：然后以自己的账号limei_user身份登录,并运行脚本文件test3.sql

```SQL
[student@deep02 ~]$cat test3.sql
[student@deep02 ~]$sqlplus limei_user/123@localhost/pdborcl
SQL>@test3.sql
SQL>exit
```

- #### 第3步：以system用户运行

```SQL
set autotrace on

select * from limei_user.orders where order_date
between to_date('2017-1-1','yyyy-mm-dd') and to_date('2018-6-1','yyyy-mm-dd');

select a.ORDER_ID,a.CUSTOMER_NAME,
b.product_name,b.product_num,b.product_price
from limei_user.orders a,limei_user.order_details b where
a.ORDER_ID=b.order_id and
a.order_date between to_date('2017-1-1','yyyy-mm-dd') and to_date('2018-6-1','yyyy-mm-dd');
```

##### 运行结果如下：
##### 1、以system身份登录
![](https://raw.githubusercontent.com/Gao-limei/pictures/master/1.png)

##### 2、以limei_user身份登录
![](https://raw.githubusercontent.com/Gao-limei/pictures/master/2.png)

##### 3、以system用户运行
![](https://raw.githubusercontent.com/Gao-limei/pictures/master/3.png)

![](https://raw.githubusercontent.com/Gao-limei/pictures/master/4.png)


  #### 在主表orders和从表order_details之间建立引用分区，在limei_user用户中创建两个表：orders（订单表）和order_details（订单详表），两个表通过列order_id建立主外键关联。orders表按范围分区进行存储，order_details使用引用分区进行存储。

 - #### 创建orders表的部分语句是：

```SQL
SQL>CREATE TABLESPACE users02 DATAFILE
'/home/student/你的目录/pdbtest_users02_1.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED，
'/home/student/你的目录/pdbtest_users02_2.dbf' 
  SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED
EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;

SQL>CREATE TABLESPACE users03 DATAFILE
'/home/student/你的目录/pdbtest_users02_1.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED，
'/home/student/你的目录/pdbtest_users02_2.dbf' 
  SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE UNLIMITED
EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;

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
```
##### 运行结果如下：

- #### 创建order_details表的部分语句如下：
```SQL
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
```
##### 运行结果如下：

## 查看数据库的使用情况
#### 以下代码查看表空间的数据库文件，以及每个文件的磁盘占用情况。

```SQL
$ sqlplus system/123@pdborcl

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

##### 运行结果如下：
![](https://raw.githubusercontent.com/Gao-limei/pictures/master/0.png)

- ##### autoextensible是显示表空间中的数据文件是否自动增加。
- ##### MAX_MB是指数据文件的最大容量。

## 实验总结
### 成功掌握了分区表的创建方法，和各种分区方式的使用场景。

### 首先创建了3个表分区：USERS,USERS02,USERS03。用上节课中创建的用户limei_user创建了本实验中需要的两张表：订单表(orders)与订单详表(order_details)，表创建在上述3个分区。然后创建表、插入数据、对表进行联合查询。对比分区与不分区的不同差别。

