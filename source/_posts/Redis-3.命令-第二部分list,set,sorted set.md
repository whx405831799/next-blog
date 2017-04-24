title: Redis-3.命令之list,set,sorted set
date: 2017/01/15 19:10:01
categories:
- 缓存
tags:
- redis
---
# Redis命令-第二部分list,set,sorted set

## 5.Redis的list命令
命令 | 作用 | 例子 | 结果
---|---|---|---
LPUSH KEY_NAME VALUE1.. VALUEN| Redis Lpush 命令将一个或多个值插入到列表头部。 如果 key 不存在，一个空列表会被创建并执行 LPUSH 操作。 当 key 存在但不是列表类型时，返回一个错误 | LPUSH newlist wukong sanzang bajie| (integer) 3
LPUSHX KEY_NAME VALUE1.. VALUEN | 将一个或多个值插入到已存在的列表头部，列表不存在时操作无效 |LPUSHX listnew wukong| (integer) 0
RPUSH KEY_NAME VALUE1..VALUEN | 用于将一个或多个值插入到列表的尾部(最右边)。如果列表不存在，一个空列表会被创建并执行 RPUSH 操作。 当列表存在但不是列表类型时，返回一个错误| RPUSH newlist bailongma shashen| (integer) 5
RPUSHX KEY_NAME VALUE1..VALUEN| 将一个或多个值插入到已存在的列表尾部(最右边)。如果列表不存在，操作无效 | RPUSHX listnew wukong| (integer) 0
LPOP KEY_NAME | 移除并返回列表的第一个元素 | LPOP newlist| "bajie"
BLPOP LIST1 LIST2 .. LISTN TIMEOUT|移出并获取列表的第一个元素，如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止| BLPOP maydaylist newlist 10| 1) "maydaylist"2) "masha"注意你写多个list,按顺序下来，哪个先弹出及返回，并不是所有list都弹出一个。
RPOP KEY_NAME |移除并返回列表的最后一个元素| RPOP newlist| 同lpop
BRPOP LIST1 LIST2 .. LISTN TIMEOUT  | 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止 | BRPOP maydayli newlist 10| 同上
RPOPLPUSH SOURCE_KEY_NAME DESTINATION_KEY_NAME| 移除列表的最后一个元素，并将该元素添加到另一个列表并返回 | RPOPLPUSH newlist xiyoulist| "shashen"
BRPOPLPUSH LIST1 ANOTHER_LIST TIMEOUT |从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止| BRPOPLPUSH newlist xiyoulist 10|"bailongma"
LINDEX KEY_NAME INDEX_POSITION |通过索引获取列表中的元素。你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推| LINDEX newlist -1| "wukong"
LINSERT KEY_NAME BEFORE/AFTER EXISTING_VALUE NEW_VALUE| 在列表的元素前或者后插入元素。 当指定元素不存在于列表中时，不执行任何操作。 当列表不存在时，被视为空列表，不执行任何操作。 如果 key 不是列表类型，返回一个错误| LINSERT newlist BEFORE bailongma beforetudi| (integer) 6
LLEN KEY_NAME| 用于返回列表的长度。 如果列表 key 不存在，则 key 被解释为一个空列表，返回 0 。 如果 key 不是列表类型，返回一个错误| LLEN newlist| (integer) 6
LREM KEY_NAME COUNT VALUE| 根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素。count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值 | LREM newlist -2 wukong|(integer) 2 
LSET KEY_NAME INDEX VALUE | 通过索引来设置元素的值。当索引参数超出范围，或对一个空列表进行 LSET 时，返回一个错误 |LSET newlist 0 newwukong| ok
LTRIM KEY_NAME START STOP| 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。下标 0 表示列表的第一个元素，以 1 表示列表的第二个元素，以此类推。 你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推|LTRIM newlist 1 -1| ok

## 6.Redis的set命令

命令 | 作用 | 例子 | 结果
---|---|---|---
SADD KEY_NAME VALUE1..VALUEN|将一个或多个成员元素加入到集合中，已经存在于集合的成员元素将被忽略。假如集合 key 不存在，则创建一个只包含添加的元素作成员的集合。当集合 key 不是集合类型时，返回一个错误| SADD xiyouset wukong wujing wuneng sanzang bailongma| (integer) 5
SCARD KEY_NAME| 返回集合中元素的数量|SCARD xiyouset| (integer) 5
SDIFF FIRST_KEY OTHER_KEY1..OTHER_KEYN| 返回给定集合之间的差集(后面的与第一个的差积)。不存在的集合 key 将视为空集| SDIFF maydayset xiyouset newset| 返回了maydayset中wukong以外的值
SDIFFSTORE DESTINATION_KEY KEY1..KEYN| 将给定集合之间的差集存储在指定的集合中。如果指定的集合 key 已存在，则会被覆盖|SDIFFSTORE diffset maydayset xiyouset|(integer) 4
SINTER KEY KEY1..KEYN |返回给定所有给定集合的交集。 不存在的集合 key 被视为空集。 当给定集合当中有一个空集时，结果也为空集|SINTER xiyouset maydayset| "wukong"
SINTERSTORE DESTINATION_KEY KEY KEY1..KEYN|将给定集合之间的交集存储在指定的集合中。如果指定的集合已经存在，则将其覆盖 |SINTERSTORE hahaset xiyouset maydayset|(integer) 1
SUNION KEY KEY1..KEYN |返回给定集合的并集。不存在的集合 key 被视为空集|SUNION xiyouset maydayset| 返回并集
SISMEMBER KEY VALUE |判断成员元素是否是集合的成员| SISMEMBER xiyouset wukong| (integer) 1
SUNIONSTORE DESTINATION KEY KEY1..KEYN| 命令将给定集合的并集存储在指定的集合 destination 中| SUNIONSTORE newhahaset xiyouset maydayset| (integer) 9
SISMEMBER KEY VALUE |判断成员元素是否是集合的成员| SISMEMBER xiyouset wukong| (integer) 1
SRANDMEMBER KEY [count] | 用于返回集合中的一个随机元素。如果 count为正数，且小于集合基数，返回一个包含count个元素的数组，不重复。如果 count 大于等于集合长度，返回整个集合。如果count为负数，返回一个数组，元素可重复，数的长度为 count 的绝对值| SRANDMEMBER xiyouset 3| 随机返回了三个元素
SMOVE SOURCE DESTINATION MEMBER|将指定成员 member元素从source集合移动到destination集合，是原子性操作。source 集合不存在或不包含指定的 member 元素，则不执行操作，返回0；否则，移除元素，并添加到 destination 集合中 | SMOVE xiyouset maydayset wujing| (integer) 1
SPOP KEY | 用于移除并返回集合中的一个随机元素 |SPOP maydayset|"guaishou"
SREM KEY MEMBER1..MEMBERN| 用于移除集合中的一个或多个成员元素，不存在的成员元素会被忽略|SREM maydayset shitou guaishou| (integer) 2
SSCAN KEY [MATCH pattern] [COUNT count] |用于迭代集合键中的元素 |SSCAN maydayset 0 MATCH a* COUNT 3| 1) "1" 2) 1)  "ashine2" 2) "a5" 3) "a3"  这里的游标还不太清楚（当scan命令的游标为0时，服务器开始新的迭代，当服务器返回值为0的游标时，表示迭代结束）


## 7.Redis的sorted set命令

命令 | 作用 | 例子 | 结果
---|---|---|---
ZADD KEY_NAME SCORE1 VALUE1.. SCOREN VALUEN|将一个或多个成员元素及其分数值加入到有序集当中| ZADD sortedkey 1 whx 2 ashine 3 tmac| (integer) 3
ZCARD KEY_NAME| 返回集合中元素的数量|ZCARD sortedkey| (integer) 3
ZCOUNT key min max| 计算有序集合中指定分数区间的成员数量|ZCOUNT sortedkey 1 2| (integer) 2
ZINCRBY key increment member| 对有序集合中指定成员的分数加上增量 increment,可增加负数|ZINCRBY sortedkey -6 tmac|"-3"
ZRANGE key start stop [WITHSCORES]| 返回有序集中，指定区间内的成员，顺序默认按分数从小到大，相同分数按字典顺序，0表示第一个成员，1表示第二个成员；-1表示最后，-2表示倒数第二个 |ZRANGE sortedkey 0 -1| 1) "tmac" 2) "whx" 3) "ashine"
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]|返回有序集合中指定分数区间的成员列表，区间的取值使用闭区间 (小于等于或大于等于)，也可以通过给参数前增加 (符号来使用可选的开区间 (小于或大于)|ZRANGEBYSCORE zset (1 5|省略
ZREVRANGE key start stop [WITHSCORES] |返回有序集中，指定区间内的成员，从大到小|ZREVRANGE sortedkey 0 -1 WITHSCORES|1) "ashine" 2) "2" 3) "whx" 4) "1" 5) "tmac" 6) "-3"
ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]|返回有序集中指定分数区间内的所有的成员|ZREVRANGEBYSCORE sortedkey   2 0| 1) "ashine" 2) "whx"
ZRANK key member| 返回有序集中指定成员的排名。其中有序集成员按分数值递增(从小到大)顺序排列 如果成员不是有序集 key 的成员，返回 nil。| ZRANK sortedkey tmac| (integer) 0
ZREVRANK key member| 返回有序集中成员的排名。其中有序集成员按分数值递减(从大到小)排序，最大位0| ZREVRANK sortedkey whx| (integer) 0
ZREM key member |移除有序集中的一个或多个成员，不存在的成员将被忽略| ZREM sortedkey tmac ashine| (integer) 2
ZREMRANGEBYRANK key start stop | 用于移除有序集中，指定排名(rank)区间内的所有成员
| ZREMRANGEBYRANK sortedkey 1 2| (integer) 2
ZREMRANGEBYSCORE key min max| 用于移除有序集中，指定分数（score）区间内的所有成员 |ZREMRANGEBYSCORE sortedkey 0 1|(integer) 2
 ZSCORE key member| 返回有序集中，成员的分数值。 如果成员元素不是有序集 key 的成员，或 key 不存在，返回 nil |SREM ZSCORE sortedkey whx| "1"
有序列表还有交集，并集及迭代，还有字典遍历相关，需要时再查询。

 



 


