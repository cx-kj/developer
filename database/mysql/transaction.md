# MySQL 事务
```
1.什么是事务
2.为什么要用事务
3.什么是 mvcc
4.什么是脏读
5.什么是幻读
6.什么是不可重复读
7.事务特性
8.事务隔离级别
```

## 1.什么是事务
```
事务是数据执行过程中的一个逻辑单元，由一组有限的数据库（SQL）操作构成。
通俗地讲，事务就是一组原子性的 SQL 操作组成，事务内的操作要么完全执行，要么这些操作完全不执行，不出存在中间状态。
```

## 2.为什么要用事务
```
为了处理高度复杂数据的原子性、一致性、隔离性的问题
```

## 3.什么是 mvcc（Multi Version Concurrency Control的简称)，代表多版本并发控制）
```
mvcc是多版本控制，解决事务隔离级别并发问题
MVCC可以认为是行级锁的一个变种，它可以在很多情况下避免加锁操作，因此开销更低。MVCC的实现大都都实现了非阻塞的读操作，写操作也只锁定必要的行。
InnoDB的MVCC实现，是通过保存数据在某个时间点的快照来实现的。一个事务，不管其执行多长时间，其内部看到的数据是一致的。
也就是事务在执行的过程中不会相互影响。下面我们简述一下MVCC在InnoDB中的实现。
InnoDB的MVCC，通过在每行记录后面保存两个隐藏的列来实现：一个保存了行的创建时间，一个保存行的过期时间（删除时间），
当然，这里的时间并不是时间戳，而是系统版本号，每开始一个新的事务，系统版本号就会递增。在RR隔离级别下，MVCC的操作如下：
select：
    InnoDB只查找版本早于（包含等于）当前事务版本的数据行。可以确保事务读取的行，要么是事务开始前就已存在，或者事务自身插入或修改的记录。
    行的删除版本要么未定义，要么大于当前事务版本号。可以确保事务读取的行，在事务开始之前未删除。
insert：将新插入的行保存当前版本号为行版本号。
delete：将删除的行保存当前版本号为删除标识。
update：为insert和delete操作的组合，insert的行保存当前版本号为行版本号，delete则保存当前版本号到原来的行作为删除标识。

```

## 4.什么是脏读
```
一个事务读取到了其他事务未提交的数据
通俗地讲：事务A读取了事务未提交更新的数据,然后B回滚操作,那么A读取到的数据是脏数据
```

## 5.什么是幻读
```
指当事务在读取某个范围的的记录时，另外一个事务又在该范围内插入新的记录，当事务再去去取相同范围的记录时，读取到另外事务插入的新记录（幻影行）
通俗地讲：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级,但是系统管理员B就在这个时候插入了一条具体分数的记录,当系统管理员A改结束后发现还有一条记录没有改过来,就好像发生了幻觉一样,这就叫幻读
```

## 6.什么是不可重复读
```
一个事务开始到提交之前，只能"看见"已提交的事务所做的修改。
通俗的讲：事务 A 多次读取同一数据,事务 B 在事务A多次读取的过程中,对数据作了更新并提交,导致事务A多次读取同一数据时,结果不一致
一个事务开始读取到了其他事务已提交的数据
```

## 7.事务特性
```
1.原子性（Atomicity）
2.一致性(Consistency)
3.隔离性(Isolation)
4.持久性(Durability)
```

### 7.1.原子性（Atomicity）
```
事务中所有操作要么全部提交成功,要么全部失败回滚.对事务来说,不可能只执行其中的一部分
```

### 7.2.一致性(Consistency)
```
是指数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态
也就是说A、B两人在转账钱的总和是2,000，转账后两人的总和也必须是 2,000. 不会因为这次转账事务破坏这个状态.
如果帐户A上的钱减少了,而帐户B上的钱却没有增加,那么我们认为此时数据处于不一致的状态
```

### 7.3.隔离性(Isolation)
```
指一个事务的执行不能被其他事务干扰,即一个事务内部的操作及使用的数据对并发的其他事务是隔离的.
因此,每个事务都感觉不到系统中有其它事务在并发地执行,不同的事务之间彼此没有任何干扰.
比如A转出100但事务没有确认提交,这时候银行人员对其账号查询时,看到的应该还是1,000而不是900
```

### 7.4.持久性(Durability)
```
一旦事务提交,其所做的修改就会永久保存到数据库中.此时,即使系统崩溃,修改的数据也不会丢失
```

## 8.事务隔离级别
> 事务每一种隔离级别都规定了一个事务中所作的修改,哪些在事务内和事务之间是可见的,哪些是不可见的.
  较低的隔离级别通常可以执行更高的并发,系统开销也更低.
```
1.未提交读（READ UNCOMMITTED）
2.提交读（READ COMMITTED）
3.可重复读（REPEATABLE READ）
4.可串行化（SERIALIZABLE）

# 示例表
CREATE TABLE `account` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(32) NOT NULL DEFAULT '' COMMENT '帐号',
  `balance` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '金额',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
INSERT INTO `account`(`id`, `name`, `balance`) VALUES (1, 'A', 100);
INSERT INTO `account`(`id`, `name`, `balance`) VALUES (2, 'B', 200);
```

### 8.1.未提交读（READ UNCOMMITTED）
```
事务中的修改,即使没有提交,对其它事务也都是可见的.
通俗的说: 所有事务都可以看到其他未提交事务的执行结果
可能发生脏读:即事务可以读取未提交的数据.(实际中很少使用)

示例:
- 事务A和事务B，事务A未提交的数据，事务B可以读取到
- 这里读取到的数据叫做“脏数据”
- 这种隔离级别最低，这种级别一般是在理论上存在，数据库隔离级别一般都高于该级别
```

|ID|事务A|事务B|
|---:|:---|:---|
| 1 | SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED; # 更改隔离级别 | SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED; # 更改隔离级别 |
| 2 | SELECT @@tx_isolation; #查询隔离级别 | SELECT @@tx_isolation; #查询隔离级别 |
| 3 | select * from account; | select * from account; |
| 4 | start transaction; |  |
| 5 |  | start transaction; |
| 6 | UPDATE `account` SET `balance` = 101 WHERE `name` = 'A'; |  |
| 7 | INSERT INTO `account`(`id`, `name`, `balance`) VALUES (null, 'A2', 120); |  |
| 8 |  | select * from account; # 事务B能看事务A所有的操作，即使事务A未提交 |
| 9 | rollback; | commit; |
| 10 |  |  |

```
mysql> select * from account; # 事务B能看事务A所有的操作，即使事务A未提交 
+----+------+---------+
| id | name | balance |
+----+------+---------+
|  1 | A    |     101 |
|  2 | B    |     200 |
|  3 | A2   |     120 |
+----+------+---------+
```

### 8.2.提交读（READ COMMITTED）
```
一个事务开始时,只能看见已经提交的事务所做的修改.这个级别有时候也叫做不可重复读,因为它不保证事务重新读的时候能读到相同的数据.
因为在每次数据读完之后其他事务可以修改刚才读到的数据.
考虑下面这种情况：A事务读取一个数据项var=1,然后B事务更新该数据var=2并提交,A事务接着又重新读了该数据,结果var=2,
第二次读到的数据和第一次的不相同.大多数数据库系统的默认隔离级别是 READ COMMITTED(但是MySQL不是)

示例:
- 事务A和事务B，事务A提交的数据，事务B才能读取到
- 这种隔离级别高于读未提交
- 换句话说，对方事务提交之后的数据，我当前事务才能读取到
- 这种级别可以避免“脏数据”
- 这种隔离级别会导致“不可重复读取”
- Oracle默认隔离级别
```

|ID|事务A|事务B|
|---:|:---|:---|
| 1 | SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED; # 更改隔离级别 | SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED; # 更改隔离级别 |
| 2 | SELECT @@tx_isolation; #查询隔离级别 | SELECT @@tx_isolation; #查询隔离级别 |
| 3 | select * from account; | select * from account; |
| 4 | start transaction; |  |
| 5 |  | start transaction; |
| 6 | UPDATE `account` SET `balance` = 101 WHERE `name` = 'A'; |  |
| 7 | INSERT INTO `account`(`id`, `name`, `balance`) VALUES (null, 'A2', 120); |  |
| 8 | select * from account; # 事务能看到自己操作的数据 |  |
| 9 | | select * from account; # 事务B不能看到事务A未提交 |
| 10 | commit; # 事务A提交 |  |
| 11 | | select * from account; # 事务A已经提交数据，事务B能看到事务A提交的数据 |

```


### 8.3.可重复读（REPEATABLE READ）
```
同一个事务多次读取同样记录的结果时一致的。
理论上，可重复读隔离级别还是无法解决另外一个幻读问题。
幻读指当某个事务在读取某个范围内的记录时，另外一个事务又在该范围内插入了新的记录,
当之前的事务再次读取该范围的记录时,会产生幻行.(可重复读是 MySQL 的默认事务隔离级别)
InnoDB和XtraDB引擎通过mvcc解决了幻读问题

示例:
- 事务A和事务B，事务A提交之后的数据，事务B读取不到
- 事务B是可重复读取数据
- 这种隔离级别高于读已提交
- 换句话说，对方提交之后的数据，我还是读取不到
- 这种隔离级别可以避免“不可重复读取”，达到可重复读取
- 比如1点和2点读到数据是同一个
- MySQL默认级别
- 虽然可以达到可重复读取，但是很多人说会导致“幻像读”，实际不会导致幻读（因为使用mvcc）
```

|ID|事务A|事务B|
|---:|:---|:---|
| 1 | SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ; # 更改隔离级别 | SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ; # 更改隔离级别 |
| 2 | SELECT @@tx_isolation; #查询隔离级别 | SELECT @@tx_isolation; #查询隔离级别 |
| 3 | select * from account; | select * from account; |
| 4 | start transaction; |  |
| 5 |  | start transaction; |
| 6 | UPDATE `account` SET `balance` = 101 WHERE `name` = 'A'; |  |
| 7 | INSERT INTO `account`(`id`, `name`, `balance`) VALUES (null, 'A2', 120); |  |
| 8 | select * from account; # 事务能看到自己操作的数据 |  |
| 9 | | select * from account; # 事务B不能看到事务A未提交 |
| 10 | commit; # 事务A提交 |  |
| 11 | | select * from account; # 事务A已经提交数据，事务B不能看到事务A提交的数据 |


### 8.4.可串行化（SERIALIZABLE）
```
最高的事务隔离级别,在该级别下,事务串行化顺序执行,可以避免脏读、不可重复读与幻读.
但是这种事务隔离级别效率低下,比较耗数据库性能(一般不使用)

示例:
- 事务A和事务B，事务A在操作数据库时，事务B只能排队等待
- 这种隔离级别很少使用，吞吐量太低，用户体验差
- 这种级别可以避免“幻像读”，每一次读取的都是数据库中真实存在数据，事务A与事务B串行，而不并发
```

|ID|事务A|事务B|
|---:|:---|:---|
| 1 | SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE; # 更改隔离级别 | SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE; # 更改隔离级别 |
| 2 | SELECT @@tx_isolation; #查询隔离级别 | SELECT @@tx_isolation; #查询隔离级别 |
| 3 | select * from account; | select * from account; |
| 4 | start transaction; |  |
| 5 |  | start transaction; |
| 6 | UPDATE `account` SET `balance` = 101 WHERE `name` = 'A'; |  |
| 7 | INSERT INTO `account`(`id`, `name`, `balance`) VALUES (null, 'A2', 120); |  |
| 8 | select * from account; # 事务能看到自己操作的数据 |  |
| 9 | | select * from account; # 事务A未提交，事务B会等待 |
| 10 | commit; # 事务A提交 |  |
| 11 | | 事务A提交后，事务B的查询才会执行括号内的第9步（select * from account; # 事务A未提交，事务B会等待） |