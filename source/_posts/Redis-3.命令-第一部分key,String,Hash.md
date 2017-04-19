title: Redis-3.命令之key,String,Hash
date: 2017/01/15 19:10:01
categories:
- JAVA服务端-redis
tags:
- redis
---
# Redis命令-第一部分key,String,Hash
## 1.连接redis

```
$ redis-cli -h host -p port -a password
eg：redis-cli -h 127.0.0.1 -p 6379 -a redis 
如果没密码 直接redis-cli
```

## 2.Redis的键命令

命令 | 作用 | 例子 | 结果
---|---|---|---
SET key | 插入键 | SET mykey whx| ok
GET key | 插入键 | GET mykey whx| whx
DEL key | 删除键 | DEL mykey| (integer) 1
DUMP key  | 返回存储在指定键的值的序列化版本 | DUMP mykey| 省略
EXISTS key | 查询键是否存在 | EXISTS mykey| (integer) 1
EXPIRE key seconds | 指定键的过期时间，秒 | EXPIRE mykey 10| (integer) 1
EXPIREAT key timestamp  | 指定的键过期时间，时间为Unix时间戳格式 | EXPIREAT mykey 1484539499| (integer) 1
PEXPIRE key milliseconds | 指定键的过期时间，毫秒 | PEXPIRE mykey  10000| (integer) 1
PEXPIREAT key milliseconds-timestamp  | 设置键在Unix时间戳指定为毫秒到期 | PEXPIREAT key 1484556630000| (integer) 1
KEYS pattern  | 查找与指定模式匹配的所有键 | KEYS my*| mykey
MOVE key db   | 移动键到另一个数据库(select 0 就是选择0库) | MOVE mykey 1| (integer) 1
PERSIST key   |命令用于移除给定 key 的过期时间，使得 key 永不过期 | PERSIST mykey| 当过期时间移除成功时，返回 1 。 如果 key 不存在或 key 没有设置过期时间，返回 0 。
TTL keys  | TTL 命令以秒为单位返回 key 的剩余过期时间| TTL mykey| 当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以毫秒为单位，返回 key 的剩余生存时间。
PTTL key | 以毫秒为单位返回 key 的剩余过期时间 | PTTL mykey| 当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以毫秒为单位，返回 key 的剩余生存时间。
RANDOMKEY  | 从当前数据库中随机返回一个 key | RANDOMKEY| mykey
RENAME key newkey  | 修改键的名字 | RENAME mykey newmykey| ok
RENAMENX key newkey   | 用于在新的 key 不存在时修改 key 的名称 | RENAMEX mykey newmykey| ok
TYPE key   | 用于返回 key 所储存的值的类型 | TYPE mykey| String

## 3.Redis的String命令

命令 | 作用 | 例子 | 结果
---|---|---|---
SET key | 设置给定 key 的值。如果 key 已经存储其他值， SET 就覆写旧值，且无视类型 | SET mykey whx| ok
GET key | 获取指定 key 的值。如果 key 不存在，返回 nil 。如果key 储存的值不是字符串类型，返回一个错误 | GET mykey whx| whx
GETRANGE KEY_NAME start end | 获取存储在指定 key 中字符串的子字符串。字符串的截取范围由 start 和 end决定 | GETRANGE mykey 0 1| wh
GETSET KEY_NAME VALUE  | 设置指定 key 的值，并返回 key 旧的值 | GETSET mykey 666| whx
GETBIT KEY_NAME OFFSET | 命令用于对 key 所储存的字符串值，获取指定偏移量上的位(bit) |GETBIT mykey 5| 1
MGET KEY1 KEY2 .. KEYN | 命令返回所有(一个或多个)给定 key 的值 | MGET mykey mykey1 mykey2| 1) "666" 2) "888" 3) (nil)
SETBIT KEY_NAME OFFSET  | 用于对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit) | 同get| 同get
SETEX KEY_NAME TIMEOUT VALUE | 为指定的 key 设置值及其过期时间。如果 key 已经存在， SETEX 命令将会替换旧的值。 | SETEX mykey 20 new666| ok
PSETEX KEY_NAME TIMEOUT VALUE | 以毫秒为单位设置 key 的生存时间。如果 key 已经存在， SETEX 命令将会替换旧的值。 | PSETEX mykey 1000 666| ok
SETNX KEY_NAME VALUE  |  命令在指定的 key 不存在时，为 key 设置指定的值,设置成功，返回 1 。 设置失败，返回 0 | SETNX mykey 666| (integer) 0
SETRANGE KEY_NAME OFFSET VALUE  | 指定的字符串覆盖给定 key 所储存的字符串值，覆盖的位置从偏移量 offset 开始 | SETRANGE mykey 1 whx| 5whx(只会返回数字，这里是get拿到的)
STRLEN KEY_NAME   | 用于获取指定 key 所储存的字符串值的长度。当 key 储存的不是字符串值时，返回一个错误 |STRLEN mykey| (integer) 4
MSET key1 value1 key2 value2 .. keyN valueN   |用于同时设置一个或多个 key-value | MSET key1 "Hello" key2 "World"| ok
MSETNX key1 value1 key2 value2 .. keyN valueN   |用于所有给定 key 都不存在时，同时设置一个或多个 key-value,所有 key 都成功设置，返回 1 。 如果所有给定 key 都设置失败(至少有一个 key 已经存在)，那么返回 0 | MSETNX key1 "Hello" key2 "World"| 0
INCR KEY_NAME | Redis Incr 命令将 key 中储存的数字值增一。如果 key 不存在，那么 key的值会先被初始化为 0 ，然后再执行 INCR 操作。如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。本操作的值限制在 64 位(bit)有符号数字表示之内| incr num| 2
INCRBY KEY_NAME INCR_AMOUNT  | 命令将 key 中储存的数字加上指定的增量值,如果 key 不存在，那么 key的值会先被初始化为 0 ，然后再执行 INCRBY 命令 | INCRBY num 10| 12
INCRBYFLOAT KEY_NAME INCR_AMOUNT | 为 key 中所储存的值加上指定的浮点数增量值,如果 key 不存在，那么 INCRBYFLOAT 会先将 key 的值设为 0 ，再执行加法操作 | INCRBYFLOAT num 10.01| 22.01
DECR KEY_NAME   | Redis Decr 命令将 key 中储存的数字值减一 |incr num1| 2
DECRBY KEY_NAME DECREMENT_AMOUNT   | 命令将 key 中储存的数字减去上指定的值 | DECRBY num1 10| -8
APPEND KEY_NAME NEW_VALUE| 命令用于为指定的 key 追加值,如果 key 不存在， APPEND 就简单地将给定 key 设为 value | APPEND mykey 888| 返回数字
 

## 2.Hash的键命令

命令 | 作用 | 例子 | 结果
---|---|---|---
HSET KEY_NAME FIELD VALUE | 用于为哈希表中的字段赋值 |HSET ashine name whx| (integer) 1
HMSET KEY_NAME FIELD1 VALUE1 ...FIELDN VALUEN | 用于同时将多个 field-value (字段-值)对设置到哈希表中 | HMSET ashine name wuhanxiao age 25 sex man| ok
HSETNX KEY_NAME FIELD VALUE | 用于为哈希表中不存在的的字段赋值|HSETNX ashine name whx1| (integer) 0
HGET KEY_NAME FIELD_NAME  | 用于返回哈希表中指定字段的值 | HGET ashine name| whx
HMGET KEY_NAME FIELD1...FIELDN  | 用于返回哈希表中，一个或多个给定字段的值 | HMGET ashine name age| 1) whx 2)25
HGETALL KEY_NAME | 用于返回哈希表中，所有的字段和值,在返回值里，紧跟每个字段名(field name)之后是字段的值(value)，所以返回值的长度是哈希表大小的两倍 | HGETALL ashine| 省略
HEXISTS KEY_NAME FIELD_NAME  | 用于查看哈希表的指定字段是否存在 | HEXSITS ashine name| 如果哈希表含有给定字段，返回 1 。 如果哈希表不含有给定字段，或 key 不存在，返回 0 
HDEL KEY_NAME FIELD1.. FIELDN  | 用于删除哈希表 key 中的一个或多个指定字段，不存在的字段将被忽略 | HDEL ashine name age| (integer) 2
HINCRBY KEY_NAME FIELD_NAME INCR_BY_NUMBER  | 用于为哈希表中的字段值加上指定增量值,如果type不对，会返回错误 | HINCRBY ashine age 12| (integer)15
HINCRBYFLOAT KEY_NAME FIELD_NAME INCR_BY_NUMBER   | 用于为哈希表中的字段值加上指定浮点数增量值 | HINCRBYFLOAT ashine age 1.5| 16.5
HKEYS KEY_NAME | 用于获取哈希表中的所有字段名| HKEYS ashine| 字段的list
HLEN KEY_NAME|命令用于获取哈希表中字段的数量 | HLEN ashine|(integer) 4
HVALS KEY_NAME |返回哈希表所有字段的值| HVALS ashine|字段值的list


 


