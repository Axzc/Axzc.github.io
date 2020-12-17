---
title: mysql索引 1 -聚簇索引和非聚簇索引
date: 2020-12-14 19:59:47
tags: mysql  
categories: 笔记
---    
##  聚簇索引和非聚簇索引   

### 聚簇索引 
- 聚簇索引：聚簇索引就是按照表的主键构造一颗B+树，同时非叶子节点存储的是索引指针，叶子节点中存放的就是整张表的行记录数据。一个InnoDB表只能有1个(有且只有)聚簇索引**在InnoD中如果表中没有主键 那么InnoDB会在 一个唯一并且非null的行创建聚簇索引，如果没有这样的行那么会创建一个隐藏的主键，然后对其建立聚簇索引**。目前只有InnoDB支持聚簇索引MyISAM不支持 聚簇索引。在InnoDB 中出了聚簇索引之外的其他索引都叫做辅助索引（二级索引）比如复合索引、前缀索引、唯一索引等，他们的叶子节点保存的不是整行记录的数据也不是对应行的物理地址而是主键值。    
<!--more-->    

![](/img/cluster_index.png)        
### 非聚簇索引
- 非聚簇索引： 将数据存储于索引分开结构，索引结构的叶子节点保存了指向了数据的对应行的物理行(磁盘位置) MyISM使用的是非聚簇索引。  
![](/img/not_cluster_index.png)  
- 辅助索引（二级索引）：二级索引是在InnoDB中出主键索引之外的索引，他们叶子结点保存的是主键值。在使用辅助索引进行查询的时候需要多查一次，如下图：  
![](/img/assist_index.png)  
在使用辅助索引查询时首先会在辅助索引B+树中检索，找到符合条件叶子结点获取他对应的主键，第二步使用主键在聚簇索引B+数中在执行一次检索操作，最终到达叶子结点获取整行数据。这个操作 叫做回表。
### 覆盖索引  
我们上面说过 使用辅助通常需要练功次查询才可以查询到数据，但是如果需要的数据从索引中就能够获得，就不必从表中读取，减少了查询次数。**如果一个索引包含了（或覆盖了）满足查询语句中字段与条件的数据就叫做覆盖索引。不是所有类型的索引都可以成为覆盖索引。覆盖索引必须要存储索引的列，而哈希索引、空间索引和全文索引等都不存储索引列的值，所以MySQL只能使用B-Tree索引做覆盖索引。**      
```sql     
  mysql> show index from actor \G;  -- 查询索引
  *************************** 1. row ***************************
          Table: actor
    Non_unique: 0
      Key_name: PRIMARY  -- 主键（聚簇索引）
  Seq_in_index: 1
    Column_name: actor_id
      Collation: A
    Cardinality: 200
      Sub_part: NULL
        Packed: NULL
          Null: 
    Index_type: BTREE
        Comment: 
  Index_comment: 
  *************************** 2. row ***************************
          Table: actor
    Non_unique: 1
      Key_name: idx_actor_last_name -- 辅助索引
  Seq_in_index: 1
    Column_name: last_name
      Collation: A
    Cardinality: 121
      Sub_part: NULL
        Packed: NULL
          Null: 
    Index_type: BTREE
        Comment: 
  Index_comment: 
  2 rows in set (0.00 sec)
```

可以看下面 两条语句 分别使用了回表和 覆盖索引        
```sql
  mysql> explain select * from actor where last_name like "WOO%" \G;
  *************************** 1. row ***************************
            id: 1
    select_type: SIMPLE
          table: actor
    partitions: NULL
          type: range  -- 在索引 中范围查找
  possible_keys: idx_actor_last_name
            key: idx_actor_last_name  
        key_len: 182
            ref: NULL
          rows: 2
      filtered: 100.00
          Extra: Using index condition  -- 使用了索引但是需要回表
  1 row in set, 1 warning (0.01 sec)

  ERROR: 
  No query specified  
  -- -------------------------------------------------  
  mysql> explain select actor_id from actor where last_name like "WOO%" \G;
  *************************** 1. row ***************************
            id: 1
    select_type: SIMPLE
          table: actor
    partitions: NULL
          type: range
  possible_keys: idx_actor_last_name
            key: idx_actor_last_name
        key_len: 182
            ref: NULL
          rows: 2
      filtered: 100.00
          Extra: Using where; Using index  -- 使用了where条件过滤 和 使用了索引 这里表示直接获取数据没有回表。
  1 row in set, 1 warning (0.00 sec)
```
我们可以发现上面两条语句的区别在于 select * 和 select actor_id. 因为辅助索引idx_actor_last_name的叶子节点保存的数据就是要查询的actor_id（主键值）所以这里并不需要回表操作，而select * 查询的是正行数据 辅助索引中并没有这多数据，就需要使用辅助索引中保存的actor_id 去聚簇索引中查询对应行的索引数据。



