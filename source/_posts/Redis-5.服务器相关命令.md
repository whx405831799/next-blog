title: Redis-5.服务器相关命令
date: 2017/02/06 20:38:03
categories:
- 缓存
tags:
- redis
---
# Redis-5.服务器相关命令
## Redis 服务器命令主要是用于管理 redis 服务

命令 | 作用 | 例子 | 结果
---|---|---|---
Info|命令以一种易于理解和阅读的格式，返回关于 Redis 服务器的各种信息和统计数值|Info|省略
DBSIZE| 返回当前数据库的 key 的数量|DBSIZE|(integer) 5
FLUSHDB|用于清空当前数据库中的所有 key|FLUSHDB|OK
FLUSHALL|用于清空整个 Redis 服务器的数据(删除所有数据库的所有 key )|FLUSHALL|OK
SAVE |执行一个同步保存操作，将当前 Redis 实例的所有数据快照(snapshot)以 RDB 文件的形式保存到硬盘|SAVE|OK
ROLE  |查看主从实例所属的角色，角色有master, slave, sentinel|ROLE|所属角色
MONITOR |用于实时打印出 Redis 服务器接收到的命令，调试用|MONITOR |打印出正在执行的语句
LASTSAVE |返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示|LASTSAVE|1486282788
SLOWLOG subcommand [argument]|Redis 用来记录查询执行时间的日志系统|slowlog get:列出所有;slowlog get N:列出最近N条;SLOWLOG LEN 日志数量;SLOWLOG RESET 清空日志|省略
SLAVEOF host port |将当前服务器转变为指定服务器的从属服务器(slave server)|SLAVEOF 127.0.0.1 6379|OK

还有更多命令可以参考[http://www.redis.net.cn/tutorial/3518.html](http://www.redis.net.cn/tutorial/3518.html)







 


