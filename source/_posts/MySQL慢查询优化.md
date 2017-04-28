title: MySQL慢查询优化
date: 2017/04/15 00:26:22
categories:
- 数据库
tags:
- mysql
- 慢查询
---
[toc]
## 1.慢查询日志
### 1.1 开启慢查询日志 
慢查询日志开启：

在配置文件my.cnf或my.ini中在[mysqld]一行下面加入两个配置参数

slow-query-log-file=/data/mysqldata/slow-query.log   #慢查询日志路径        

long_query_time=2 #超过两秒才记录

log-queries-not-using-indexes   #记录没使用索引的查询

### 1.2 使用mysqldumpslow来分析日志
- 首先要安装perl，设置好环境变量
- 在mysql的bin目录执行perl mysqldumpslow.pl --help 可以查看功能

```
mysqldumpslow的命令参数列举如下：
--help    输出帮助信息
-v           输出详细信息 
-d          调试
-s          按照什么排序，默认是'at'，显示顺序为倒序
              al: 平均锁表时间
ar: 平均结果行数
                at: 平均查询时间
                 c: 次数
                 l: 锁表时间
                 r: 总结果行数
                 t: 总查询时间  
 -r          正序排序，即从小到大排序
-t NUM       限制显示的条数
-a           显示出数字和字符串，默认数字为 N 字符串为 'S'
-g PATTERN   过滤字符串，后接正则表达式，如'10$' 以10为结尾的条件
例子：
/usr/local/MySQL/bin/mysqldumpslow -s t -a -t 3   slow.txt
根据总查询时间排序，只列出前3条
/usr/local/mysql/bin/mysqldumpslow -r -s c -a -t 3 -g 'hello'   slow.txt
搜索包括关键字 hello的结果，并按照次数正序排序前3条
```

- 在mysql的bin目录执行perl mysqldumpslow.pl ../slow-query.log便可以分析slowsql日志。


## 2.使用EXPLAIN分析查询语句
### 2.1 EXPLAIN字段分析
![image](http://ohoyqlwj0.bkt.clouddn.com/explaintest.png)
- select_type:select类型。
- 
```
simple:表示简单的select,没有使用union和子查询。
primary:在有子查询的情况下，最外面的查询就是primary。
union:union语句的第二个是此类型。
dependent union：union语句的第二个是此类型。取决于外面的select语句。
union result: union的结果。
...
```

- table:显示这数据属于哪张表
- type:这是最重要的字段之一，显示查询使用了什么类型。从最好到最差的连接类型为system、const、eq_reg、ref、range、index和all

```
1.system:表仅有一行，这是const类型的特列，平时不会出现。
eg:无。
2.const：表最多有一个匹配行，const用于比较primary key 或者unique索引。因为只匹配一行数据，所以很快。记住一定是用到primary key 或者unique，并且只检索出两条数据的 情况下才会是const。
eg:EXPLAIN SELECT * FROM coupon_allot WHERE id = 701060421
3.eq_ref：MySQL手册是这样说的:"对于每个来自于前面的表的行组合，从该表中读取一行。这可能是最好的联接类型，除了const类型。它用在一个索引的所有部分被联接使用并且索引是UNIQUE或PRIMARY KEY"。eq_ref可以用于使用=比较带索引的列。只有比较的两个字段索引是primary时，才会有eq_ref出现，如果都是unique，则是ref类型。
eg:EXPLAIN SELECT * FROM  coupon_allot ,entity_allot  WHERE coupon_allot.id = entity_allot.id 
4.ref：访问索引,返回某个值的数据。(可以返回多行) 通常使用=时发生。下面status加了索引。
eg:EXPLAIN SELECT * FROM coupon_allot where `status` = 1
5.range：这个连接类型使用索引返回一个范围中的行，比如使用>或<查找东西，并且该字段上建有索引时发生的情况。(注:不一定好于index)。下面的uid是有索引的。
eg:EXPLAIN SELECT * FROM coupon_allot WHERE `uid` IN ('pt1461662954011-7533681','pt1461550427342-7533619')
6.index：该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）
7.all：全表扫描，应该尽量避免。
```
- possible_keys:显示可能应用在这张表中的索引。如果为空，没有可能的索引。
- key:实际使用的索引。如果为NULL，则没有使用索引。MYSQL很少会选择优化不充分的索引，这种情况可以在SELECT语句中使用USE INDEX（index）来强制使用一个索引或者用IGNORE INDEX（index）来强制忽略索引。
- key_len:使用的索引长度。在不丢失精确性的情况下，长度越短越好。
- ref:ref列显示使用哪个列或常数与key一起从表中选择行。
- rows:显示MYSQL执行查询的行数，数值越大越不好，说明索引没用好。
- Extra:显示MYSQL如何解析查询的额外信息。
```
1.using index:只用到索引,可以避免访问表。
2.using where：使用到where来过滤数据,不是所有的where clause都要显示using where。如以=方式访问索引.
3.using tmporary：用到临时表。
4.using filesort：用到额外排序。 (当使用order by ,而没用到索引时,就会使用额外的排序)
5.range checked for eache record(index map:N)：没有好的索引.
```

## 3.profiling分析
profiling命令可以得到更准确的SQL执行消耗系统资源。
### 3.1 使用profiling步骤
- 开启profiling

```
SET profiling=1;
```
- 执行sql,然后执行profile命令查找queryID

```
SHOW PROFILES
```
![image](http://ohoyqlwj0.bkt.clouddn.com/showprofile.png)

- 执行分析具体queryID的命令

```
SHOW PROFILE FOR QUERY 41
```
![image](http://ohoyqlwj0.bkt.clouddn.com/showprofileforquery.png)

- 分析结束后，关闭profiling

```
 set profiling=0
```
### 3.2 其他常用命令

```
SHOW PROFILE cpu FOR QUERY 41;查看cpu消耗
SHOW PROFILE MEMORY FOR QUERY 41;查看内存消耗
SHOW PROFILE block io,cpu FOR QUERY 41;查看io及cpu的消耗
SHOW PROFILE ALL FOR QUERY 41;查看所有性能指标
```






