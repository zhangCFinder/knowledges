[TOC]
## 什么是indexcondition pushdown(ICP)?


1. 在数据库中pushdown表示某些操作“下推”，也就是某些操作提前执行，在生成执行计划时某些操作下推可以大大提升效率。

2. 为什么叫下推，因为优化器在生成的计划叫做执行计划树，操作从叶子节点开始往根上执行，下推就意味着提前执行。

3. 举个最简单的例子，某些投影操作下推可以大大减小在执行过程中的数据量，而这里说的index condition pushdown也是这一种类似的下推操作。也就是将利用索引判断的这一步提前执行

示例：在customer_id, value两列建立联合索引

*  **在没有ICP之前它是这样执行的**
  
  1. 从索引idx_custid_value索引里面取出下一条customer_id<4的记录，然后利用主键字段读取整个行

  2. 然后对这个完整的行利用value=290这个进行判断看是否符合条件。

  3. 从1开始重复这个过程

 

*  **有了ICP之后则是这样执行的**
  
  1. 从索引idx_custid_value索引里面取出下一条customer_id<4的记录，然后利用这个索引的其他字段(即value列)进行判断，如果条件成立，执行第2步，否则第3步
  
  2. 在上一步中筛选出来符合条件的才会利用主键索引找到这个完整行。
  
  3.  从1开始重复这个过程

**也就是说将判断条件提前（或者称作下推），这对在customer_id上有很多符合条件的行，而value上却很具有选择性的这种查询是很大的改进。**

explain命令示例：


* mysql5.5(没有实现 index conditionpushdown)


| id | select_type | table  | type | possible_keys | key  |key_len | ref  | rows | Extra |
| --- | --- |--- |--- |--- |--- |--- |--- |--- |--- |
|  1| SIMPLE | orders | range |idx_custid_value  | idx_custid_value |5  | NULL | 1 | **Using where** |

* mysql5.6(实现了 index conditionpushdown)

|id|select_type|table |type|possible_keys|key|key_len|ref |rows|Extra|
| --- | --- |--- |--- |--- |--- |--- |--- |--- |--- |
| 1|SIMPLE|orders|range|idx_custid_value |idx_custid_value|5|NULL|1|**Using indexcondition**|

我觉得ICP的核心在于：
* **它并不是影响优化器选择哪个索引，而是在于怎么利用这个索引。**
* 在上面的两个执行计划可以看出无论是否ICP，两个都是利用idx_custid_value索引的customer_id列(key_len=5,而customer_id为int，占4个字节，还有1个字节用来判断是否null)
* 两者使用的索引完全相同，只是处理方式不同。

总结：


需要index condition pushdown 的query通常索引的字段出现where子句里面都是范围查询。比如：

```sql
select * from tb where tb.key_part1 < x and tb.key_part2 = y       

select * from tb where tb.key_part1 = x andtb.key_part2 like '%yyyy%'

select * from tb where tb.key_part1 > x and tb.key_part1 < y and tb.key_part1 > xx and tb.key_part2 < yy

```
但是需要注意的是：

1. 如果索引的第一个字段的查询就是没有边界的比如 key_part1 like '%xxx%'，那么不要说ICP，就连索引都会没法利用。

2. 如果select的字段全部在索引里面，那么就是直接的index scan了，没有必要什么ICP

3. 没有做性能对比测试，因为现在的mysql 5.6都还只是开发版，等有GA版本再测不迟

