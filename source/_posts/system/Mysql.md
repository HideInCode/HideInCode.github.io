# Mysql

> 伟大滴麦色可友，养活互联网企业的神器。

## 组成结构

### 架构

- server层
  - 连接器：管理链接，权限验证
  - 查询缓存：命中直接返回
  - 分析器：词法分析，语法分析
  - 优化器：执行计划生成，索引选择
  - 执行器：操作引擎，返回结果
- 引擎层：操作引擎，返回结果。

### redo-log

- WAL：Write-Ahead Logging
- crash-safe
- innodb
- 物理日志：在数据页上做的修改
- 循环写

### bin-log

- server层，意味着所有引擎可以用
- 逻辑日志：语句原始逻辑，给ID=2的字段c加1
- 追加写

### 两段式提交

1. redolog进入prepare 
2. 写入binlog
3. 提交事务

## 索引

### B+树

- 为什么选B+树？
  - 选了B+而不用B树，是因为B树在每个节点都存储数据，而B+树只会在叶子节点存储数据，所以B树IO会更频繁；数据库的索引是存在磁盘上的，当数据量大时，就不能全部加载到内存，只能逐一加载。

- 基于B树实现，具有B树的平衡性；
- 

### 索引特性

- 回表
- 索引覆盖
- 最左前缀原则
- 索引下推

## 事务

### 基本性质

1. 原子性(Atomicity)

   事务的所有操作只有成功和失败,操作期间不会去对数据库数据进行增删改操作,只记录操作,只有commit后才会去修改数据库的数据.事务失败就会rollback,数据不会有任何变化.

2. 一致性（Consistency）

   会话的双方要同时进行的DML操作,使得事务执行后数据库从一个一致性状态变到另一个一致性状态.

3. 隔离性（Isolation）

   事务之间是相互独立的两个对象,可以设置不同的隔离级别.

4. 持久性（Durability）

   事务终结的标志,提交后的事务将会永久性改变数据库的数据.

### 隔离级别

1. read uncommitted ->脏读

   > 这种隔离级别下 两个事务A B,A 没有提交数据 ,B 读到了A没有commit的数据(事务A正在操作的数据),这种数据叫做dirty data,这种行为叫做dirty read,一般只在理论上存在;	

2. read committed -> 不可重复度

   > 两个事务A B,A提交数据,B就读到A的数据,这种隔离级别的缺点就是不可重复度,即会话双方一方修改,一方读取,总是读取不到正确数据.
   > 不可重复读的重点是修改.

3. repeatable read -> 幻读

   > 看这名字就知道就是来解决无法重复读的问题的.事务A开始时,不管事务BCD修改数据还是提交事务,都不会改变A读取的数据.
   > 幻读:幻读是指当事务不是独立执行时发生的一种现象.事务A读取与搜索条件相匹配的若干行。
   > 事务B以插入或删除行等方式来修改事务A的结果集，然后再提交。这时候事务A的用户发现表中还存在没有修改的数据行,就像出现了幻觉一样.

4. serializable -> 效率低

   > 不会出现上述问题,但由于进行了锁表操作,所以会降低吞吐量.
   > 解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表.	

### 隔离的原理

多版本并发控制MVCC

## 锁

### 全局锁

- 一般全库逻辑备份时会用到
- FlushTablesWithReadLock：库进入只读，阻塞DML，DDL，未提交的更新事务。故障时会释放锁。
- set global readlonly=true：用于主备库判断，故障时数据库不会释放锁。

### 表锁

- lock tables ... read/write 客户端断开自动释放
- 元数据锁：metadata lock：访问表时自动开启，修改表结构时加MDL锁；
- 给小表安全加字段：
  - kill长事务
  - alter table语句加等待时间，过期放弃。

### 行锁

- innodb特有
- 死锁与死锁检测：
  - 循环依赖导致
  - 等到超时：innodb_lock_wait_timeout
  - 主动检测：innodb_deadlock_detect=on，缺点影响性能。
  - 减小主动检测的性能影响：
    - 控制并发度，一行只能十个线程，但是客户端过多还是不行。
    - 设计层面优化，把记录拆分多条，然后算多个记录的和。

## 常见表结构设计

### 表设计三大范式

> 范式只是一种参考，实际生产要考虑索引问题、联表查询效率问题。

1. 列不可分

   - 一列不可有两种属性
   - 比如地址可分为省、市、区三个字段

2. 在范式1的基础上，所有的非主键字段要**完全依赖**主键，不是依赖主键的某一部分（针对联合主键）

   - 出现了**多对多**关系
   - 例如一张表存学生和课程
     - 新增一个学生，该学生还没有选课，因此就不能新增。新增老师同理。

   - 一个表只描述一件事情

3. 在范式2基础上，所有非主键字段和主键字段**没有传递依赖**

   - 出现**一对多**关系
   - 例如一张表存老师和学生
     - 修改老师职称时要改很多数据（修改一影响多）
     - 没人选该老师课程时，该老师职称记录会被删除。（多没了代表了一也没了，不合理）
     - 新老师还没指定职称，那么职称不知如何保存。
   - 一对多出现时考虑：一和多分成两张表，多表存一表主键，

## SQL命令使用常识

- count：count(字段)<count(id)<count(1)约等count(*)
- order by：
  - sort_buffer_size：排序内存大小，如果太小就利用临时文件排序。
  - rowid排序：如果要返回字段过多，只取出排序字段。
  - 如果内存够，就用内存排序。
- join:
  - 正确选择驱动表，驱动表走全表扫描，被驱动表走数索引。
  - 使用join语句，性能比强行拆成多个单表执行SQL语句的性能好。
  - 如果使用join，需要使用小表做驱动。

## 常见疑难杂症

- 普通索引和唯一索引怎么选？

  - 读：区别不大

  - 写：
    - change buffer：缓存更新操作，唯一索引不能用，只有普通索引可以用。如果更新后马上查询，不要用这个。redolog节省随机写磁盘的io消耗，changebuffer节省对应读的io消耗。
    - 唯一索引：判断冲突
    - 普通索引：更新记录在changebuffer

- 怎么给字符串加索引？

- mysql为什么会抖一下？
  - 内存脏页：内存数据和磁盘数据不一样时，内存页就是脏页。

- 为什么表数据删除一半，表文件大小不变？
  - delete结果不会变，alter table才会改变。

- 为什么逻辑相同的语句，性能缺差异巨大？

  - 条件字段函数操作

  - 隐式类型转换

  - 隐式字符编码转换

- 查一行也很慢？

  - 查询长时间不返回：锁表了，通过show processlist查看状态。
  - 等DML锁：show processlist查看wait for table metadata lock
  - 等flush

- 幻读有什么问题吗？

- 鸩止渴的提高性能？

  - 先处理占着链接但是不工作的线程

  - 减少链接过程的消耗

  - 慢查询性能问题
    - 索引没有设计好
    - 语句没写好
  - QPS突增问题

- 如何保证数据不丢？

  - binglog写入机制

  - redolog写入机制

- 保证主备一致？

- 涉及到binlog可选
  - statement
  - row
  - mixed

- 怎么保证高可用？

  - 主备延迟

    - 备库机器差劲

    - 备库压力大
    - 大事务

  - 可靠性优先策略

- 备库为什么会延迟几个小时？

  1. 5.5版本并行复制策略
     1. 按表分发
     2. 按行分发

  1. 5.7并行复制策略

  1. 5.7.22并行复制策略

- 主库出问题，从库怎么办？

  1. 基于位点的主备切换

  1. 基于GTID的主备切换

- 读写分离有哪些坑？

1. 主备延迟导致的读写分离
   1. 强制走主库
   2. sleep
   3. 判断主备无延迟
      1. 通过seconds_behind_master
      2. 位点比对
      3. GTID
   4. semi-sync半同步复制

- 判断数据库是不是出问题？

  1. select 1

  1. 查表判断

  1. 更新判断

  1. 内部统计

- 误删数据怎么办？

  1. 删除行

  1. 删除库/表

- 为什么有些命令kill不掉？

  1. kill query + 线程id

  1. kill connection + 线程id

  1. 关于客户端的误解：表多就慢

- 查大量数据会不会打爆内存？

  1. 全表扫描对server层影响：先装net_buffer，满了就输出，边读边发。

  1. 全表扫描对innodb的影响

- 为什么临时表可以重名？

  1. create temporary table 创建临时表

  1. 创建的临时表只能被当前session访问

  1. 可以与普通表重名，优先访问临时表，show tables不显示临时表。

- 何时会用到临时表？

  1. union执行流程

  1. group by

- innodb虽然厉害，那么memory还用不用？

  1. innodb索引组织表
     1. 有序存放
     2. 有空位也会写入新位置
     3. 数据位置变化只要修改主键索引
     4. 通过主键就一次，普通索引要两次

  1. memory堆组织表
     1. 按照写入顺序存放
        1. 有空位就插入
        2. 位置变化要修改所有索引
        3. 查找时一视同仁

- 怎么快速复制一张表？

  1. sqldump方法：生成包含insert语句的方法，可以在where参数里面加过滤，不能join

  1. 导出csv

  1. 物理拷贝：最快、必须全表拷贝、需要到机器上拷贝，在用户登录数据库的场景下无法使用，都得是innodb

- grant之后要跟着flushprivileges吗？

- 要不要分区分表？

- 自增ID用完怎么办？

  1. 表定义自增id：用完会冲突，可以增加范围biginit unsigned。

     1. innodb系统自增row_id：
        1. 不显示指定主键时，系统隐式自带。
        2. 用完就会重零开始覆盖

     1. xid
        1. 作用于事务：这是redo log和binglog配合使用的id
        2. 不要在binlog重复

  1. innodb trx_id：每次随着mysql重启会被保存起来。

  1. thread_id

## 练习

**初始化SQL**

```mysql
drop table if exists dept;
drop table if exists salgrade;
drop table if exists emp; 
create table dept(		
    deptno int(10) primary key,		
    dname varchar(14),		
    loc varchar(13)		
);		
create table salgrade(		
    grade int(11),		
    losal int(11),		
    hisal int(11)		
);		
create table emp(		
    empno int(4) primary key,		
    ename varchar(10),		
    job varchar(9),		
    mgr int(4),		
    hiredate date,		
    sal double(7,2),		
    comm double(7,2),		
    deptno int(2)		
);		
insert into dept(deptno,dname,loc) values(10,'ACCOUNTING','NEW YORK');
insert into dept(deptno,dname,loc) values(20,'RESEARCHING','DALLAS');
insert into dept(deptno,dname,loc) values(30,'SALES','CHICAGO');
insert into dept(deptno,dname,loc) values(40,'OPERATIONS','BOSTON'); 

insert into salgrade(grade,losal,hisal) values(1,700,1200);
insert into salgrade(grade,losal,hisal) values(2,1201,1400);
insert into salgrade(grade,losal,hisal) values(3,1401,2000);
insert into salgrade(grade,losal,hisal) values(4,2001,3000);
insert into salgrade(grade,losal,hisal) values(5,3001,5000); 

insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7369,'SIMITH','CLERK',7902,'1980-12-17',800,null,20);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7499,'ALLEN','SALESMAN',7698,'1981-02-20',1600,300,30);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7521,'WARD','SALESMAN',7698,'1981-02-22',1250,500,30);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7566,'JONES','MANAGER',7839,'1981-04-02',2975,null,20);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7654,'MARTIN','SALESMAN',7698,'1981-09-28',1250,1400,30);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7698,'BLAKE','MANAGER',7839,'1981-05-01',2850,null,30);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7782,'CLARK','MANAGER',7839,'1981-06-09',2450,null,10);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7788,'SCOTT','ANALYST',7566,'1987-04-19',3000,null,20);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7839,'KING','PRESIDENT',null,'1981-11-17',5000,null,10);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7844,'TURNER','SALESMAN',7698,'1981-09-08',1500,null,30);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7876,'ADAMS','CLERK',7788,'1987-05-23',1100,null,20);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7900,'JAMES','CLERK',7698,'1981-12-03',950,null,30);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7902,'FORD','ANALYST',7566,'1981-12-03',3000,null,20);
insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno)	values(7934,'MILLER','CLERK',7782,'1982-01-23',1300,null,10);	
```



1. 每个部门薪水最高的人员名称

   ```mysql
   SELECT
   	e.ename NAME,
   	t.maxsal,
   	d.dname deptname 
   FROM
   	emp e
   	JOIN ( SELECT deptno, max( sal ) maxsal FROM emp GROUP BY deptno ) t ON t.deptno = e.deptno 
   	AND t.maxsal = e.sal
   	JOIN dept d ON t.deptno = d.deptno;
   ```

2. 薪水在部门平均水平以上的人

   ```mysql
   select 
   	e.ename name,e.deptno,e.sal
   from 
   	emp e join (select deptno,avg(sal) avgsal from emp group by deptno) t 
   on e.sal > t.avgsal and t.deptno=e.deptno;
   ```

3. 取得部门中的平均薪水等级(所有人)

   ```mysql
   select 
   	t.deptno deptno,s.grade grade 
   from 
   	(select deptno, avg(sal) avgsal from emp group by deptno) t join salgrade s on t.avgsal between losal and hisal;
   ```

4. 取得部门中所有人的薪水等级,并对部门薪水等级求平均

   ```mysql
   select t.dept, avg(t.grade) avg_grade from (select e.deptno dept,e.ename name,e.sal salary,s.grade grade from emp e join salgrade s on e.sal between s.losal and hisal) t group by t.dept;
   ```

   也可以

   ```mysql
   select e.deptno dept,avg(s.grade) from emp e join salgrade s on e.sal between s.losal and hisal group by e.deptno;
   ```

4. 不用聚合函数max,取得最高薪水,使用两种方法解决
   1. `select ename,sal from emp order by sal desc limit 1;`
   2. `select sal from emp where sal not in (select distinct a.sal sal_a from emp a join emp b on a.sal < b.sal);`

5. 取得平均薪水最高的部门的部门编号(2种方案)

   > 可能有相等数据,取第一个可能会错误,所以要根据deptno找到重复的max

   1. ```mysql
      select 
      	deptno,avg(sal) as avgsal
      from 
      	emp 
      group by 
      	deptno
      having 
      	avg(sal) = (select avg(sal) avgsal from emp group by deptno order by avgsal desc limit 1);
      ```

   2. 聚合函数max

      ```mysql
      select 
      	deptno,avg(sal) avgsal
      from 
      	emp
      group by 
      	deptno
      having
      	avg(sal) = (select max(t.avgsal) from (select deptno,avg(sal) avgsal from emp group by deptno ) t);
      ```

6. 取得平均薪水最高的部门的部门名称

   ```mysql
   select 
   	d.dname,m.avgsal
   from
   	dept d
   join 
   	(select 
   	deptno,avg(sal) avgsal
   from 
   	emp
   group by 
   	deptno
   having
   	avg(sal) = (select max(t.avgsal) from (select deptno,avg(sal) avgsal from emp group by deptno ) t)) m
   	on m.deptno=d.deptno;
   ```

   

7. 求平均薪水的等级最高的部门名称

1. 求各个部门平均薪水的等级(结果只有部门名称.平均薪水,等级三个字段)

   ```mysql
   select t.avgsal,s.grade,t.dname from 
   (select d.dname dname,avg(e.sal) avgsal from emp e join dept d on e.deptno=d.deptno group by dname) t
   join salgrade s
   on t.avgsal between s.losal and s.hisal;
   ```

   

2. 获取最高等级值

   ```mysql
   select max(s.grade) from 
   (select deptno,avg(sal) avgsal from emp group by deptno) t
   join salgrade s
   on t.avgsal between s.losal and s.hisal;
   ```

   

3. 联合上两张表 给出avgsal,dname,grade,满足平均值为最大值

   ```mysql
   select t.avgsal,s.grade,t.dname from 
   (select d.dname dname,avg(e.sal) avgsal from emp e join dept d on e.deptno=d.deptno group by dname) t
   join salgrade s
   on t.avgsal between s.losal and s.hisal
   where
   	s.grade=(select max(s.grade) from 
   (select deptno,avg(sal) avgsal from emp group by deptno) t
   join salgrade s
   on t.avgsal between s.losal and s.hisal);
   ```

   

8. 取得比**普通员工**(员工代码没有在mgr字段上出现的)的**最高薪水**还要高的姓名

   1. 先找到普通员工(注意not in (不能有null))
      `select * from emp where empno  not in (select distinct mgr from emp where mgr is not null);`

   			2. 找出普通员工最高薪水
   
      `select max(sal) from emp where empno not in (select distinct mgr from emp where mgr is not null);`
      
   3. 找出薪水高于1600
   
      ```mysql
      select ename ,sal from emp 
      where sal > (select max(sal) from emp where empno not in (select distinct mgr from emp where mgr is not null));
      ```

​			补充学习: `select ename,sal,(case job when 'manager' then sal * 0.8 when 'salesman' then sal*1.5 end) newsal from emp;`

9. 取得薪水最高的前五

10. 取得薪水最高的第六名到第十名

11. 取得最后入职的5名员工

12. 取得每个薪水等级有多少个员工	

    ```mysql
    select e.ename, e.sal, s.grade from emp e join salgrade s on e.sal between s.losal and s.hisal;
    select t.sgrade ,count(*) from (select e.ename, e.sal, s.grade sgrade from emp e join salgrade s on e.sal between s.losal and s.hisal) t group by t.sgrade;
    select count(*), s.grade 
    from emp e join salgrade s on e.sal between s.losal and s.hisal
    group by s.grade;
    ```

13. 列出所有员工和领导的名字
     列出所有的领导,not in 领导就是员工(自连接)

    ```mysql
     select distinct a.empno,a.ename,b.mgr from emp a join emp b on a.empno =b.mgr;-- 所有的领导
     select ename from emp where ename not in (select distinct a.ename from emp a join emp b on a.empno =b.mgr);
    ```

14. 找出入职日期早于其直接领导的员工编号,姓名,部门名称

    ```mysql
    		SELECT
    			t.adno,
    			t.aname,
    			d.dname 
    		FROM
    			(
                    SELECT
                    a.empno eno,
                    a.ename aname,
                    a.hiredate ahdate,
                    a.deptno adno,
                    b.hiredate bhdate 
                    FROM
                    emp a
                    LEFT JOIN emp b ON a.mgr = b.empno 
                ) t
    			JOIN dept d ON d.deptno = t.adno 
    		WHERE
    			t.ahdate < t.bhdate
    ```

    ```mysql
    -- 老司机做法选出员工直接老板和各自的入职日期,通过where筛选
    	select a.empno aempno, a.ename aename,d.dname ddname, b.empno bempno, b.ename bename
    	from emp a join emp b on a.mgr = b.empno join dept d on d.deptno = a.deptno	
    	where a.hiredate<b.hiredate;		
    ```



15. 列出部门名称和这些部门员工信息,同时列出没有员工的部门
    `select d.dname,e.* from emp e right join dept d on e.deptno=d.deptno order by d.dname asc;`

16. 列出至少有5个员工的所有部门(什么这里select后可以有groupby没有的东西)
    `select dept.*,emp.deptno,count(*) from emp join dept on emp.deptno = dept.deptno group by dept.deptno having count(*) >=5;`

17. 列出薪资比simith高的员工的个人信息
    `select * from emp where sal > (select sal from emp where ename='simith');`

18. 列出所有clerk职位的员工姓名,部门名称,部门人数

    ```mysql
    select e.ename ename,d.dname ddname, t.countall from emp e join dept d on d.deptno = e.deptno join (select e.deptno, count(*) countall from  emp e  group by e.deptno) t on t.deptno = d.deptno
    where job='clerk' ;
    select e.deptno, count(*) countall from  emp e group by e.deptno;
    ```

    ​		

19. 列出最低薪资大于1500的各工种及从事此工作的全部员工人数

    ```mysql
    select job,min(sal) minsal,count(*) from emp group by job having minsal >1500;
    ```

20. 在不知道部门编号的情况下列出sales部门的员工姓名
     这题又理解错误,emp的deptno是知道的,只有dept表中deptno不知道

```mysql
 select deptno from dept where dname='sales';
 select ename from emp where deptno = ( select deptno from dept where dname='sales');
```



21. 列出薪资高于公司平均薪资的所有员工,所在部门,上级领导,员工的工资等级

    ```mysql
    -- 注意用全连接找出king的信息
    SELECT
    	e.mgr,
    	e.ename,
    	e.deptno,
    	e2.ename boss,
    	d.dname,
    	s.grade 
    FROM
    	emp e
    	LEFT JOIN emp e2 ON e.mgr = e2.empno
    	JOIN dept d ON d.deptno = e.deptno
    	JOIN salgrade s ON e.sal BETWEEN s.losal 
    	AND s.hisal 
    WHERE
    	e.sal > ( SELECT avg( sal ) FROM emp );
    ```

    

22. 列出与scott从事相同工作的所有员工及其部门名称

    ```mysql
    SELECT
    	job 
    FROM
    	emp 
    WHERE
    	ename = 'scott';
    SELECT
    	e.ename,
    	d.dname,
    	e.job 
    FROM
    	emp e
    	JOIN dept d ON e.deptno = d.deptno 
    WHERE
    	job = ( SELECT job FROM emp WHERE ename = 'scott' );
    ```

    

23. 列出薪资等于部门30中员工的薪资的其他员工的姓名和薪金

    ```mysql
    SELECT DISTINCT
    	sal 
    FROM
    	emp 
    WHERE
    	deptno = 30;
    	
    SELECT
    	ename,
    	sal 
    FROM
    	emp 
    WHERE
    	( sal IN ( SELECT DISTINCT sal FROM emp WHERE deptno = 30 ) ) 	AND deptno <> 30;
    ```

    

24. 列出薪资高于部门30中员工的薪资的其他员工的姓名和薪金

```mysql
SELECT
	ename,
	sal 
FROM
	emp 
WHERE
	sal > ( SELECT max( t.sal ) FROM ( SELECT DISTINCT sal FROM emp WHERE deptno = 30 ) t );
```

25. 列出在每个部门工作的员工数量,平均工资和平均服务期限(工龄)

```mysql
-- 所有、每个这类关键字，一般需要全连接
SELECT
	e.deptno,
	count( e.ename ) totalmember,
	IFNULL(avg( sal ) ,0) avgsal,
	IFNULL(avg(	( ( TO_DAYS( now( ) ) - TO_DAYS( hiredate ) ) / 356 )),0) year
FROM
	emp e
	RIGHT JOIN dept d ON e.deptno = d.deptno 
GROUP BY
	deptno;
```



26. 列出所有员工的姓名,部门名称,工资
    `select e.ename,d.dname,e.sal from emp e join dept d on e.deptno=d.deptno;`

27. 列出所有部门的详细信息和人数

    ```mysql
    select d.deptno,d.dnae,d.loc,count(e.ename) from emp e right join dept d on e.deptno=d.deptno group by d.deptno,d.dname,d.loc;
    ```

    

28. 列出各个工作岗位最低工资和符合最低工资的人姓名
    (判空与去重)

    ```mysql
    SELECT
    	t.job,
    	e.ename,
    	t.minsal 
    FROM
    	emp e
    	JOIN ( SELECT job, min( sal ) minsal FROM emp GROUP BY job ) t ON e.job = t.job 
    	AND e.sal = t.minsal 
    ORDER BY
    	t.minsal ASC;
    ```

29. 获取各个部门mgr的最低薪资

```mysql
SELECT
	deptno,
	min( sal ) minsal 
FROM
	emp 
	where
	job='manager'
GROUP BY
	deptno;
```

30. 取出所有员工年薪,由低到高排序

    ```mysql
    SELECT
    	ename,
    	sal * 12+ ifnull( comm, 0 ) yearsal 
    FROM
    	emp 
    ORDER BY
    	yearsal ASC;
    ```

31. 取出员工领导薪水超过3000的员工名称和领导名称

    ```mysql
    SELECT
    	e.ename,
    	e2.ename leader
    FROM
    	emp e JOIN emp e2 ON e.mgr = e2.empno AND e2.sal > 3000;
    ```

    

32. 取出含有s的部门,给出此部门的工资合计,部门人数

    ```mysql
    SELECT
    	d.dname empname,
    	count( e.ename ) totalpersons,
    	SUM( ifnull( e.sal, 0 ) ) sumsal 
    FROM
    	emp e
    	RIGHT JOIN dept d ON e.deptno = d.deptno 
    WHERE
    	d.dname LIKE '%s%' 
    GROUP BY
    	d.dname;
    ```

33. 给任职超过30年的人加薪10%;
    工作时间计算公式(年)
    (( TO_DAYS( now( ) ) - TO_DAYS( hiredate ) ) / 365 ) year

create table emp_bak as select * from emp;

34. 面试一下

**初始化数据**

```mysql
drop table if exists s;
drop table if exists c;
drop table if exists sc; 
create table s(		
    sno int(10) primary key,		
    sname varchar(14)		
);		
create table c(		
    cno int(10) primary key,		
    cname varchar(14),		
    cteacher varchar(14)		
);		
create table sc(		
    sno int(10),  	
    cno int(10),		
    scgrade int(10),		
    primary key(sno,cno)		
);		
insert into s(sno,sname) values(1,'a');
insert into s(sno,sname) values(2,'b');
insert into s(sno,sname) values(3,'c');
insert into s(sno,sname) values(4,'d'); 

insert into c(cno,cname,cteacher) values(1,'java','王老师');
insert into c(cno,cname,cteacher) values(2,'C++','张老师');
insert into c(cno,cname,cteacher) values(3,'C#','李老师');
insert into c(cno,cname,cteacher) values(4,'mysql','周老师');
insert into c(cno,cname,cteacher) values(5,'oracle','黎明'); 

insert into sc(sno,cno,scgrade) values(1,1,50);
insert into sc(sno,cno,scgrade) values(1,2,50);
insert into sc(sno,cno,scgrade) values(1,3,50);
insert into sc(sno,cno,scgrade) values(2,2,80);
insert into sc(sno,cno,scgrade) values(2,3,70);
insert into sc(sno,cno,scgrade) values(2,4,59);
insert into sc(sno,cno,scgrade) values(3,1,60);
insert into sc(sno,cno,scgrade) values(3,2,61);
insert into sc(sno,cno,scgrade) values(3,3,99);
insert into sc(sno,cno,scgrade) values(3,4,100);
insert into sc(sno,cno,scgrade) values(3,5,52);
insert into sc(sno,cno,scgrade) values(4,3,82);
insert into sc(sno,cno,scgrade) values(4,4,99);
insert into sc(sno,cno,scgrade) values(4,5,40);
```



> 有3个表s(学生表),c(课程表),sc(学生选课表)
> s(sno,sname) 代表 (学号,姓名)
> c(cno,cname,cteacher) 代表(课号,课名,教师)
> sc(sno,cno,scgrade)代表 (学号,课号,成绩)

1. 找出没选过"黎明"老师的所有学生姓名
   	1. 肯定要用补集求解,所以先求黎明苏所在表的相关信息
   	select cno,cname from c where cteacher='黎明';
   	2. 选了黎明的人
   	select sno from sc where cno = (select cno from c where cteacher='黎明');
   	 3. 没选黎明的人
   	 select sname from s where sno not in (select sno from sc where cno = (select cno from c where cteacher='黎明'))
   
2. 列出2门以上(含2门) 不及格学生姓名及他的所有成绩的平均成绩
    	1. 先选出2门不及格的学生
    
        `    select sno from sc where scgrade < 60 group by sno having count(*)>=2 ;`
        
        2. 连接s取姓名
        
        `select s.sname from sc join s on sc.sno = s.sno where sc.scgrade < 60 group by s.sname having count(*)>=2 ;`
        
        3. 算出每个学生的平均成绩跟不及格的姓名连接(下面这个做法不行,因为其他成绩已经过滤掉了,只会求不及格成绩的平均线)
      `select s.sname name, avg(sc.scgrade) avgscore from sc join s on sc.sno = s.sno where sc.scgrade < 60 group by s.sname having count(*)>=2 ;`

```mysql
-- 求出所有学生平均成绩
select sc.sno,avg(sc.scgrade) avgscore from sc group  by sc.sno;
select sno,avg(sc.scgrade) avgscore from sc group  by sno;

	SELECT
	t1.NAME NAME,
	t2.avgscore avgscore 
FROM
	(
	SELECT
		s.sname NAME,
		s.sno sno 
	FROM
		sc
		JOIN s ON sc.sno = s.sno 
	WHERE
		sc.scgrade < 60 GROUP BY NAME, sno HAVING count( * ) >= 2 
	) t1
	JOIN ( SELECT sc.sno sno, avg( sc.scgrade ) avgscore FROM sc GROUP BY sc.sno ) t2 ON t1.sno = t2.sno 
GROUP BY
	t1.sno;
```

3. 即学过1号课程又学过2号课程所有学生的姓名
   	1. 学过1号课程的所有学生(sname,cno)
       `select s.sname sname,sc.cno cno from sc join s on s.sno=sc.sno where cno = 1;`
   	2. 学过2号课程的所有学生(sname,cno)
       `select s.sname sname,sc.cno cno from sc join s on s.sno=sc.sno where cno=2;`
   	3. 求交集

```mysql
select t1.sname sname from (select s.sname sname,sc.cno cno from sc 
join s on s.sno=sc.sno where cno = 1) t1 
join (select s.sname sname,sc.cno cno from sc join s on s.sno=sc.sno where cno=2) t2 on t1.sname = t2.sname;
```

**总结**
**作业到此完成,所有的问题都可分成小问题来拼出来,这个过程中除了注意sql的语法,mysql的特殊语法,还要考虑判空,去重,是否要全连接等.**
**查询数据的思路**

1. 我需要哪些字段?这些字段分别在那些表里?这些表靠什么连接?
2. 涉及多表字段(自连接也算) 要先分开查询,查询时要注意判空与去重,以及选用合适的过滤方法和分组函数.
3. 连接时选用内还是外连接,是要用增加查询字段,进行子查询,还是表连接要先后判断一下.

