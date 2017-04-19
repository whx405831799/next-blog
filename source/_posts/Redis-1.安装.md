title: Redis-1.安装
date: 2017/01/03 19:06:06
categories:
- JAVA服务端-redis
tags:
- redis
---

# redis是啥
redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。
Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。

# linux安装redis
安装redis必须已经安装了gcc,如果没安装gcc 就使用命令 yum install -y gcc

```
cd /usr/share
wget http://download.redis.io/releases/redis-3.2.6.tar.gz
tar xvf redis-3.2.6.tar.gz
cd redis-3.2.6/
make(如果报错，请使用make MALLOC=libc)
make test(如果报tcl错请安装 yum install -y tcl)
```
如果make test报错，[exception]: Executing test client: NOREPLICAS Not enough good slaves to write..
则修改配置文件。

```
vim tests/integration/replication-2.tcl
- after 1000
+ after 10000
```
接下来继续安装

```
make install
```
启动、进入、关闭redis

```
redis-server redis.conf ##启动redis
redis-cli ##进入redis客户端(无密码状态)  |  redis-cli -a yourpassword ##进入redis客户端(有密码状态)
redis-cli shutdown ##停止redis服务
```
也可以指定端口启动和结束

```
redis-server --port 6379
redis-cli -p 6380 shutdown
```
设置密码
修改redis.conf

#requirepass foobared去掉注释，foobared改为自己的密码







