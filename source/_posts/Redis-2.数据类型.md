title: Redis-2.数据类型
date: 2017/01/03 19:06:06
categories:
- 缓存
tags:
- redis
---

# 数据类型及常用命令
## Strings-字符串类型
Redis的字符串是字节序列。在Redis中字符串是二进制安全的，这意味着他们有一个已知的长度，是没有任何特殊字符终止决定的，所以可以存储任何东西，最大长度可达512兆。

> 例子

```
redis 127.0.0.1:6379> SET name "helloworld"
OK
redis 127.0.0.1:6379> GET name
"helloworld"
```

在上面的例子使用Redis命令set和get，helloworld即为字符串类型。

注：字符串值可以存储最大512兆字节的长度。



## Hashes-哈希值型
Redis的哈希键值对的集合。 Redis的哈希值是字符串字段和字符串值之间的映射，所以它们被用来表示对象。

> 例子

```
127.0.0.1:6379> hmset user:1 name ashine age 15 address shenzhen
OK
127.0.0.1:6379> hgetall user:1
1) "whx"
2) "17"
3) "name"
4) "ashine"
5) "age"
6) "15"
7) "address"
8) "shenzhen"
127.0.0.1:6379> hget user:1 name
"ashine"
```


## Lists-列表
Redis的列表是简单的字符串列表，排序插入顺序。可以添加元素到Redis列表的头部或尾部。

> 例子

```
127.0.0.1:6379> lpush maydaylist ashine shitou
(integer) 2
127.0.0.1:6379> lpush maydaylist guanyou
(integer) 3
127.0.0.1:6379> lpush maydaylist guaishou
(integer) 4
127.0.0.1:6379> lpush maydaylist masha
(integer) 5
127.0.0.1:6379> lrange maydaylist 0 10
1) "masha"
2) "guaishou"
3) "guanyou"
4) "shitou"
5) "ashine"
```

列表的最大长度也很大，可以支持40亿。


## Sets-无序集合
set是集合，它是string类型的无序集合。
set是通过hash table实现的，添加，删除和查找的复杂度都是0（1）。
对集合我们可以取并集、交集、差集。

> 例子

```
127.0.0.1:6379> sadd maydayset ashine
(integer) 1
127.0.0.1:6379> sadd maydayset ashine
(integer) 0
127.0.0.1:6379> sadd maydayset shitou
(integer) 1
127.0.0.1:6379> sadd maydayset masha
(integer) 1
127.0.0.1:6379> smembers maydayset
1) "masha"
2) "ashine"
3) "shitou"
127.0.0.1:6379> smembers maydayset
1) "masha"
2) "ashine"
3) "shitou"
127.0.0.1:6379> smembers maydayset
1) "masha"
2) "ashine"
3) "shitou"
```

注意：在上面的例子中ashine加了两次，但由于唯一性只加一次。

# sorted set有序集合
sortes set是set的一个升级版本，它在set的基础上增加了一个顺序属性，
这一属性在添加修改元素的时候可以指定，每次指定后，zset会自动重新按新的值调整顺序。
可以理解为有两列的MySQL表，一列存value，一列存顺序。

> 例子

```
127.0.0.1:6379> zadd sortmaydayset 0 ashine
(integer) 1
127.0.0.1:6379> zadd sortmaydayset 2 shitou
(integer) 1
127.0.0.1:6379> zadd sortmaydayset 1  guanyou
(integer) 1
127.0.0.1:6379> zadd sortmaydayset 1  ashine
(integer) 0
127.0.0.1:6379> zrangebyscore sortmaydayset 0 10
1) "ashine"
2) "guanyou"
3) "shitou"

```

注意：在上面的例子中ashine加了两次，但由于唯一性只加一次。














参考链接命令：[http://www.redis.cn/commands.html](http://note.youdao.com/)

参考链接数据类型：[http://www.redis.cn/topics/data-types-intro.html](http://note.youdao.com/)
