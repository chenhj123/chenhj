##### 首先看这个 DDL[data definition language]

```
CREATE TABLE `t_terminal_log_old`  (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `terminal_id` int(11) NOT NULL COMMENT '终端ID',
  `log_type` int(11) NULL DEFAULT NULL COMMENT '1-鉴权,2-购彩,3-兑奖,4-返奖',
  `log_info` text CHARACTER SET utf8 COLLATE utf8_general_ci NULL COMMENT '日志详情',
  `date_added` datetime(0) NULL DEFAULT NULL COMMENT '发生时间',
  `device_log_id` bigint(20) NULL DEFAULT NULL COMMENT '设备日志ID',
  `terminal_date` bigint(20) NULL DEFAULT NULL COMMENT '终端时间',
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE INDEX `UNIQUE_TERMINAL_ID_DEVICE_LOG_ID`(`terminal_id`, `device_log_id`) USING BTREE,
  INDEX `INDEX_DATE_ADDED`(`date_added`) USING BTREE,
  INDEX `INDEX_TERMINAL_ID`(`terminal_id`) USING BTREE,
  INDEX `INDEX_TERMINAL_DATE`(`terminal_date`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = '终端日志';
```

问题：

1. UNIQUE_TERMINAL_ID_DEVICE_LOG_ID 索引 和 INDEX_TERMINAL_ID 索引重复

2. utf8 和 utf8mb4 需要规范化

解答：

1. 索引最左原则，比如 UNIQUE_TERMINAL_ID_DEVICE_LOG_ID 索引， (terminal_id) 和 (terminal_id、device_log_id)都可以走索引

2. 统一规范成 utf8 或者 utf8mb4

##### 接着看这个 DML[data manipulation language]

```
EXPLAIN SELECT * FROM t_terminal_log_old WHERE terminal_id=106218 AND log_type=8 ORDER BY id DESC LIMIT 10
```

执行结果：

![](C:\Users\chj15\AppData\Roaming\marktext\images\2022-11-12-15-06-46-image.png)

问题：

1. 为什么走的是主键索引呢，根据什么走的主键索引，怎样解决问题

解答：

1. 因为存在ORDER BY id 数据库解析器错误以为走主键索引是最快的，因为当数据量特别大的时候，用索引去排序是比较快的，那我们猜想要怎样证明呢？

```
EXPLAIN SELECT * FROM t_terminal_log_old WHERE terminal_id=106218 AND log_type=8 LIMIT 10        
```

    就是把 id 去掉，就可以走正确的索引

2. 用 force index 强制选择一个索引

```
EXPLAIN SELECT * FROM t_terminal_log_old FORCE INDEX (UNIQUE_TERMINAL_ID_DEVICE_LOG_ID) WHERE terminal_id=106218 AND log_type=8 ORDER BY id DESC LIMIT 10
```

    3. 在有些场景下，我们可以新建一个更合适的索引，来提供给优化器做选择，或删掉误用的索引。

##### utf8 和 utf8mb4 规范问题

```
CREATE TABLE `tradelog` (
  `id` int(11) NOT NULL,
  `tradeid` varchar(32) DEFAULT NULL,
  `operator` int(11) DEFAULT NULL,
  `t_modified` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `tradeid` (`tradeid`),
  KEY `t_modified` (`t_modified`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `trade_detail` (
  `id` int(11) NOT NULL,
  `tradeid` varchar(32) DEFAULT NULL,
  `trade_step` int(11) DEFAULT NULL, /*操作步骤*/
  `step_info` varchar(32) DEFAULT NULL, /*步骤信息*/
  PRIMARY KEY (`id`),
  KEY `tradeid` (`tradeid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into tradelog values(1, 'aaaaaaaa', 1000, now());
insert into tradelog values(2, 'aaaaaaab', 1000, now());
insert into tradelog values(3, 'aaaaaaac', 1000, now());

insert into trade_detail values(1, 'aaaaaaaa', 1, 'add');
insert into trade_detail values(2, 'aaaaaaaa', 2, 'update');
insert into trade_detail values(3, 'aaaaaaaa', 3, 'commit');
insert into trade_detail values(4, 'aaaaaaab', 1, 'add');
insert into trade_detail values(5, 'aaaaaaab', 2, 'update');
insert into trade_detail values(6, 'aaaaaaab', 3, 'update again');
insert into trade_detail values(7, 'aaaaaaab', 4, 'commit');
insert into trade_detail values(8, 'aaaaaaac', 1, 'add');
insert into trade_detail values(9, 'aaaaaaac', 2, 'update');
insert into trade_detail values(10, 'aaaaaaac', 3, 'update again');
insert into trade_detail values(11, 'aaaaaaac', 4, 'commit');
```

```
EXPLAIN SELECT d.* FROM tradelog l, trade_detail d WHERE d.tradeid=l.tradeid AND l.id=2
```

执行结果：

![](C:\Users\chj15\AppData\Roaming\marktext\images\2022-11-12-15-17-06-image.png)

1. 第一行显示优化器会先在交易记录表tradelog上查到id=2的行，这个步骤用上了主键索引，row=1表示只扫描一行

2. 第二行key=NULL，表示没有用上交易详情表trade_detail上的tradeid索引，进行全盘描扫

在这个执行计划里，是从tradelog表中取tradeid字段，再去trade_detail表里查询匹配字段



详情步骤图

![](C:\Users\chj15\AppData\Roaming\marktext\images\2022-11-12-15-17-44-image.png)

1. 根据id在tradelog表里找到L2这一行

2. 是从L2中取出tradeid字段的值

3. 根据tradeid值到trade_detail表中查找条件匹配的行

explain的结果key=NULL表示是通过遍历全表去匹配的，那不符合我们的逻辑，那你一般百度，也只会跟你说 字符集不一致，我们去深究一下

我们可以把上面的sql，拆分为两条简单sql

```
select * from tradelog where id=2
select * from trade_detail  where CONVERT(traideid USING utf8mb4)='aaaaaaab'
```

根据表，我们知道第一行查出来的 traideid 的字符集是 utf8mb4

第二行sql，trade_detail 的字符集是 utf8，MySQL内部的操作，就是先把utf8转utf8mb4再做比较

这个很好理解，utf8mb4是utf8的超集，类似程序设计，在做类型转换的时候，为了避免数据在转换过程中由于截断导致数据错误，一般都是，按数据长度增加的方向进行转换的

因此，在执行上面这个语句的时候，数据表里的字段一个一个进行转换



再实战一条，分析这条sql的explain

```
select l.operator from tradelog l , trade_detail d where d.tradeid=l.tradeid and d.id=4;
```



如何优化：

1. 改字符集，utf8 改成 utf8mb4
2. 如下

```
select d.* from tradelog l , trade_detail d where d.tradeid=CONVERT(l.tradeid USING utf8) and l.id=2; 
```



###### 简单 explain 中的 type

ALL, index, range, ref, eq_ref, const, system, NULL（从左到右，性能从差到好）

1. ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行

```
EXPLAIN SELECT * FROM t_site
```

2. index: Full Index Scan，index与ALL区别为index类型只遍历索引树

```
SELECT `code` FROM t_site
```

3. range:只检索给定范围的行，使用一个索引来选择行

```
EXPLAIN SELECT * FROM t_site WHERE id BETWEEN 1 AND 10
```

4. ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

```
EXPLAIN SELECT * FROM t_site WHERE city_id=440100
```

5. eq_ref: 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件

```
EXPLAIN SELECT * FROM t_terminal_info ti LEFT JOIN t_site s ON ti.site_id=s.id
```

6. const、system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量,system是const类型的特例，当查询的表只有一行的情况下，使用system

```
EXPLAIN SELECT * FROM t_site WHERE id=1
```

7. NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

```
SELECT MAX(id) FROM t_site
```
