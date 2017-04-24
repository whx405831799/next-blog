title: Redis-4.深入HyperLogLog、事务、发布订阅等
date: 2017/02/05 19:58:01
categories:
- 缓存
tags:
- redis
---
# Redis-4.深入HyperLogLog、事务、发布订阅等

## 1.HyperLogLog
 HyperLogLog其实是基数估计算法。作为一种基数统计算法，比如统计一篇莎士比亚的文章中，不同单词出现的个数，如果按照平常我们想到的做法，把里面的单词都存到hashset中，求出容量即可，但是当面对的是海量数据的时候，这得占据多大的内存呢？后来就有了位图法（这里就不说了），但是hyperLogLog提供了比上面效率更高的算法。

顾名思义，Hyper LogLog计数器就是估算Nmax为基数的数据集仅需使用loglog(Nmax)+O(1) bits就可以。如线性计数器的Hyper LogLog计数器允许设计人员指定所需的精度值，在Hyper LogLog的情况下，这是通过定义所需的相对标准差和预期要计数的最大基数。大部分计数器通过一个输入数据流M，并应用一个哈希函数设置h(M)来工作。这将产生一个S = h(M) of {0,1}^∞字符串的可观测结果。通过分割哈希输入流成m个子字符串，并对每个子输入流保持m的值可观测 ，这就是相当一个新Hyper LogLog（一个子m就是一个新的Hyper LogLog）。利用额外的观测值的平均值，产生一个计数器，其精度随着m的增长而提高，这只需要对输入集合中的每个元素执行几步操作就可以完成。其结果是，这个计数器可以仅使用1.5 kb的空间计算精度为2%的十亿个不同的数据元素。与执行 HashSet所需的120 兆字节进行比较，这种算法的效率很明显。这就是传说中的”如何仅用1.5KB内存为十亿对象计数“。（还需多研究，此处很多地方还不是很理解）

以上摘自

[http://blog.csdn.net/Androidlushangderen/article/details/40683763](http://blog.csdn.net/Androidlushangderen/article/details/40683763)

[http://www.cnblogs.com/ysuzhaixuefei/p/4052110.html](http://www.cnblogs.com/ysuzhaixuefei/p/4052110.html)

命令 | 作用 | 例子 | 结果
---|---|---|---
PFADD key element [element ...]| 将所有元素参数添加到 HyperLogLog 数据结构中|PFADD hyperlogtest ashine shitou masha guanyou guaishou|如果至少有个元素被添加返回 1， 否则返回 0
PFCOUNT key [key ...]| 返回给定 HyperLogLog 的基数估算值|PFCOUNT hyperlogtest hyperlogtest1|整数，返回给定 HyperLogLog 的基数值，如果多个 HyperLogLog 则返回基数估值之和
PFMERGE destkey sourcekey [sourcekey ...]|将多个 HyperLogLog 合并为一个 HyperLogLog ，合并后的 HyperLogLog 的基数估算值是通过对所有 给定 HyperLogLog 进行并集计算得出的|PFMERGE hypermain hyperlogtest hyperlogtest1|OK


## 2.事务
事务，和关系型数据库的事务应该是一个道理。
事务四大特性(简称ACID) 
- 1、原子性(Atomicity)：事务中的全部操作在数据库中是不可分割的，要么全部完成，要么均不执行。
- 2、一致性(Consistency)：几个并行执行的事务，其执行结果必须与按某一顺序串行执行的结果相一致。
- 3、隔离性(Isolation)：事务的执行不受其他事务的干扰。
- 4、持久性(Durability)：对于任意已提交事务，系统必须保证该事务对数据库的改变不被丢失，即使数据库出现故障。 

我们主要来看下下面的命令。

命令 | 作用 | 例子 | 结果
---|---|---|---
Multi|用于标记一个事务块的开始。事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 EXEC 命令原子性(atomic)地执行。|MULTI 然后INCR test 然后INCR test 然后EXEC 执行|事务块内所有命令的返回值，按命令执行的先后顺序排列。 当操作被打断时，返回空值 nil 
Exec| 执行所有事务块内的命令|同上|同上一起用
Watch|用于监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断|WATCH test test1|OK
UNWATCH |用于取消 WATCH 命令对所有 key 的监视|UNWATCH|OK
DISCARD |用于取消事务，放弃执行事务块内的所有命令|DISCARD|OK


## 3.发布订阅
Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

Redis 客户端可以订阅任意数量的频道。 


命令 | 作用 | 例子 | 结果
---|---|---|---
SUBSCRIBE channel [channel...]|用于订阅给定的一个或多个频道的信息|SUBSCRIBE musicnews|1) "subscribe" 2) "musicnews" 3) (integer) 1 
PUBLISH channel message| 用于将信息发送到指定的频道|PUBLISH musicnews mayday|命令返回接收到信息的订阅者数量；而在订阅此channel的客户端出现如下结果1) "message" 2) "musicnews" 3) "mayday"
UNSUBSCRIBE channel [channel ...]|用于退订给定的一个或多个频道的信息|UNSUBSCRIBE musicnews|在不同的客户端中有不同的表现
PSUBSCRIBE pattern [pattern ...]|订阅一个或多个符合给定模式的频道。每个模式以 * 作为匹配符，比如 it* 匹配所有以 it 开头的频道( it.news 、 it.blog 、 it.tweets 等等)|PSUBSCRIBE *news|接收到的信息
PUNSUBSCRIBE [pattern [pattern ...]] |用于退订所有给定模式的频道|PUNSUBSCRIBE *news|在不同的客户端中有不同的表现
PUBSUB <subcommand> [argument [argument ...]]|用于退订所有给定模式的频道|PUBSUB CHANNELS(后面不接即打出所有) s*|由活跃频道组成的列表






 


