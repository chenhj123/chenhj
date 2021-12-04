## group by 优化

创建表

```sql
create table group_optimize(id int primary key, a int, b int, index(a));
delimiter ;;
create procedure groupdata()
begin
  declare i int;

  set i=1;
  while(i<=1000)do
    insert into group_optimize values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call groupdata();
```

然后执行下面语句

```sql
select id%10 as m, count(*) as c from group_optimize group by m;
```

这个语句的逻辑是把表里的数据，按照 id%10 进行分组统计，并按照 m 的结果排序后输出，explain 结果如下：

| id   | select_type | table          | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                        |
| ---- | ----------- | -------------- | ---------- | ----- | ------------- | ---- | ------- | ---- | ---- | -------- | -------------------------------------------- |
| 1    | SIMPLE      | group_optimize |            | Index | PRIMARY,a,z   | a    | 5       |      | 1000 | 100      | Using index; Using temporary; Using filesort |

在 Extra 字段里面，我们可以看到 3 个信息：

- Using index，表示这个语句使用了覆盖索引，选择了索引 a，这里不需要回表
- Using temporary，表示使用了临时表
- Using filesort，表示需要排序

这个语句的执行流程是这样的：

1. 创建内存临时表，表里有两个字段 m 和 c，主键是 m
2. 扫描表 group_optimize 的索引 a，依次取出叶子节点上的 id 值，计算 id%10 的结果，记为 x
   1. 如果临时表中没有主键为 x 的行，就插入一个纪录 (X,1)
   2. 如果表中有主键为 x 的行，就将 x 这一行的 c 值加 1
3. 遍历完成后，再根据字段 m 做排序，得到的结果集返回给客户端

### Group by 优化方法 -- 索引

可以看到，不论是使用内存临时表还是磁盘临时表，group by 逻辑都需要构造一个带唯一索引的表，执行代价都是比较高的

group by 的语义逻辑，是统计不同的值出现的个数。但是，由于每一行的 id%100 的结果是无序的，所以我们就需要有一个临时表，来记录并统计结果。

如果扫描过程中可以保证出现的数据是有序的，是不是就简单了呢

在 MySQL 5.7 版本支持了 generated column 机制，用来实现列数据的关联更新。

```sql
alter table group_optimize add column z int generated always as(id % 100), add index(z);
```

然后查询 sql 语句可以改成

```sql
select z, count(*) as c from group_optimize group by z;
```

优化后的 explain 结果如下

| id   | select_type | table          | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra        |
| ---- | ----------- | -------------- | ---------- | ----- | ------------- | ---- | ------- | ---- | ---- | -------- | ------------ |
| 1    | SIMPLE      | group_optimize |            | Index | z             | z    | 5       |      | 1000 | 100      | Using index; |

从 Extra 字段可以看到，这个语句的执行不再需要临时表，也不需要排序了。

### group by 优化方法 -- 直接排序

如果碰上不适合创建索引的场景，我们还是要老老实实做排序的。那么，这时候的 group by 要怎么优化呢？

一个 group by 语句中需要放到临时表上的数据量特别大，却还是要按照“先放到内存临时表，插入一部分数据后，发现内存临时表不够用了再转成磁盘临时表”，看上去就有点傻。

在 group by 语句中加入 SQL_BIG_RESULT 这个提示（hint），就可以告诉优化器：这个语句涉及的数据量很大，请直接用磁盘临时表。

```sql
select SQL_BIG_RESULT id%100 as m, count(*) as c from group_optimize group by m;
```

执行流程就是这样的：

1. 初始化 sort_buffer，确定放入一个整型字段，记为 m
2. 扫描表 group_optimize 的索引 a，依次取出里面的 id 值, 将 id%100 的值存入 sort_buffer 中
3. 扫描完成后，对 sort_buffer 的字段 m 做排序（如果 sort_buffer 内存不够用，就会利用磁盘临时文件辅助排序）
4. 排序完成后，就得到了一个有序数组

优化后的 explain 结果如下

| id   | select_type | table          | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra                        |
| ---- | ----------- | -------------- | ---------- | ----- | ------------- | ---- | ------- | ---- | ---- | -------- | ---------------------------- |
| 1    | SIMPLE      | group_optimize |            | Index | z             | z    | 5       |      | 1000 | 100      | Using index;  Using filesort |

