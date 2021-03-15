# MySQL

### SQL优化

1. 写入优化

   大批量写入优化

   PreparedStatement 减少SQL解析

   Multiple Values/Add Batch 减少交互

   Load Data，直接导入

   索引和约束问题

2. 数据更新

   数据的范围更新

   注意GAP Lock 的问题

   导致锁范围扩大

3. 模糊查询

   Like的问题

   前缀匹配

   否则不走索引

   全文检索，solr/ES

4. 连接查询

   连接查询优化

   驱动表的选择问题

   避免笛卡尔积

5. 索引失效

   索引失效情况汇总

   NULL，not，not in，函数等

   减少使用or，可以用union（注意 union all 的区别），以及前面提到的 like

   大数据量下，放弃所有条件组合都走索引的幻想，出门左拐 "全文检索"

   必要时可以使用 force index 来强制查询走某个索引

6. 查询 SQL 到底怎么设计

   查询数据量和查询次数的平衡

   避免不必要的大量重复数据传输

   避免使用临时文件排序或临时表

   分析类需求，可以用汇总表

### 实现主键ID

1. 自增

2. sequence

3. 模拟seq

4. UUID

5. 时间戳/随机数

6. snowflake

   雪花算法简单说一下

   ![雪花算法](https://github.com/chenhj123/chenhj/blob/main/images/snowflake.png)

   **1.第一位** 占用1bit，其值始终是0，没有实际作用。 **2.时间戳** 占用41bit，精确到毫秒，总共可以容纳约69年的时间。 **3.工作机器id** 占用10bit，其中高位5bit是数据中心ID，低位5bit是工作节点ID，做多可以容纳1024个节点。 **4.序列号** 占用12bit，每个节点每毫秒0开始不断累加，最多可以累加到4095，一共可以产生4096个ID。

   SnowFlake算法在同一毫秒内最多可以生成多少个全局唯一ID呢：： **同一毫秒的ID数量 = 1024 X 4096 = 4194304**

