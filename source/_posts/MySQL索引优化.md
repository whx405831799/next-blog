title: MySQL索引介绍及建议
date: 2017/04/20 20:16:32
categories:
- 数据库
tags:
- mysql
- 索引
---
[toc]
## 1.索引的类型
- 普通索引：最基本的索引类型，没唯一性之类限制。
- 唯一索引unique：和普通索引基本相同，但所有的索引列值保持唯一性。
- 主键索引：是一种唯一索引，但必须指定为primary key。
- 全文索引：全文索引的索引类型为FULLTEXT。全文索引可以在VARCHAR或者TEXT类型的列上创建。但是全文索引对中文的支持很弱，所以基本上做全文检索时不用mysql的全文索引。
- 哈希索引：只有Memory, NDB两种引擎支持，Memory引擎默认支持哈希索引，如果多个hash值相同，出现哈希碰撞，那么索引以链表方式存储。Hash 索引结构的特殊性,其检索效率非常高,索引的检索可以一次定位,不像B-Tree 索引需要从根节点到枝节点,最后才能访问到。但是hash索引存放的是hash值,所以仅支持 < = > 以及 IN 操作,而且碰撞多后，性能会变差。

大多数MySQL索引(PRIMARY KEY、UNIQUE、INDEX和FULLTEXT)使用B树中存储。空间列类型的索引使用R-树，适用用范围查找，很少用。

## 2.索引的方式
- 单列索引:对相关的列使用索引是提高SELECT操作性能的最佳途径之一。
- 多列索引：可以为多个列创建索引，一个索引可以包含15个列。对于某些索引，可以索引列的左前缀。

```
最左前缀原则：
比如建立了多列索引为key(firstname lastname age)，
其实相当于建立了key(firstname lastname)和key(firstname)。
```
eg:建立索引gender-username-weixin多列索引，索引三个字段。
![三个字段乱序都可以使用索引](http://ohoyqlwj0.bkt.clouddn.com/%E4%B8%89%E4%B8%AA%E5%AD%97%E6%AE%B5%E4%BD%BF%E7%94%A8%E7%B4%A2%E5%BC%95.png)

![gender-weixin不会使用索引，符合最左前缀原则](http://ohoyqlwj0.bkt.clouddn.com/gender-weixin-%E4%B8%8D%E4%BD%BF%E7%94%A8%E7%B4%A2%E5%BC%95.png)

经过测试：当where条件句为gender或者gender username或者三个字段全部出现（任意顺序），皆可使用索引。但是出现gender weixin则不会使用索引。

## 3.建立索引的建议
- 越小的数据类型越好：越小的数据类型通常在磁盘、内存和CPU缓存中都需要更少的空间，处理起来更快。
- 简单的数据类型更好：整型数据比起字符，处理开销更小，因为字符串的比较更复杂。
- 尽量避免值出现NULL：应该指定列为NOT NULL，除非你想存储NULL。MySQL对含有空值的列很难进行查询优化，因为它们使得索引、索引的统计信息以及比较运算做起来更加复杂。建议用0、一个特殊的值或者一个空串代替空值。