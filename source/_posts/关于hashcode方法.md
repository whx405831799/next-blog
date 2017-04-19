title: 关于hashcode方法
date: 2017/02/08 21:38:03
categories:
- JAVA服务端
tags:
- hashCode
---

# 一.总述
> 下面内容主要学习两点。第一点是hashCode方法的相关介绍；第二点是为什么java中修改了equals就要修改hashCode方法。

# 二.hashCode方法
## 1.数据结构-哈希表
　　哈希表是一种根据关键码值(Key value)而直接进行访问的数据结构，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。

　　哈希表可以提供快速的查找操作，插入操作。所以优点就是快，易用；缺点就是基于数组的，量大后性能会下降严重。

## 2.hashCode方法
hashCode方法的作用是为了配合基于散列的集合运行，这些散列集合包括HashSet、HashMap及HashTable等。

为什么这些散列集合会用到hashcode呢？当集合中不允许重复元素的时候。插入对象的时候肯定要判断集合中是否已存在此对象，用equals方法显然是效率有问题的。所有就得用到hashCode方法了。

简单来说就是：==当集合要添加新的对象时，先调用这个对象的hashCode方法，得到对应的hashcode值，实际上在HashMap的具体实现中会用一个table保存已经存进去的对象的hashcode值，如果table中没有该hashcode值，它就可以直接存进去，不用再进行任何比较了；如果存在该hashcode值，就调用它的equals方法与新元素进行比较，相同的话就不存了，不相同就散列其它的地址，所以这里存在一个冲突解决的问题，这样一来实际调用equals方法的次数就大大降低了==。

换一种方式说，Java里hashCode方法就是按某些规则将与对象有关联的信息（例如对象的字段或存储地址）映射成一个数值，次值称作为散列值。可以看看这段代码，此代码是java.util.HashMap的中put方法的实现

```
public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
    
     final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```
HashMap的put方法是用来向HashMap中添加新的元素，看以上代码克制，会先调用hashCode方法得到该元素的hashCode值，然后对比table中是否存在此hashCode值，如果存在则调用equals方法重新确定是否存在该元素（以为不同对象的hashcode值有可能相同），如果存在此值，则更新value，否则将新的元素添加到HashMap中。所以，hashCode方法存在的一个意义是是为了减少equals方法的调用次数，从而提高程序执行的效率。

#### ==总的来说，如果hashcode值相等，对象不一定相等，还需要equals；如果equals相当，hashcode值一定相等；如果hashcode不相等，对象一定不等。==

## 3.重写equals的同时，一定要重写hashCode方法
为何这么说？我们看一段代码。

```
public class User {
    private String name;
    private String idCard;

    public User(String name, String idCard) {
        super();
        this.name = name;
        this.idCard = idCard;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getIdCard() {
        return idCard;
    }
    public void setIdCard(String idCard) {
        this.idCard = idCard;
    }
    @Override
    public boolean equals(Object obj) {
        return this.getName().equals(((User) obj).getName()) && this.getIdCard().equals(((User)obj).getIdCard());
    }
}


public class HashCodeAndEqualsTest {
    public static void main(String[] args) {
        User user1 = new User("mayday", "55555");
        User user2 = new User("mayday", "55555");
        System.out.println("user1的hashcode为：" + user1.hashCode());
        System.out.println("user1的hashcode为：" + user2.hashCode());
        System.out.println("user1和user2是否equals？" + user1.equals(user2));
        Map map = new HashMap<>();
        map.put(user1,"test");
        System.out.println("通过equals为true的user2取得的值为" + map.get(user2));
    }
}

console：
user1的hashcode为：1876189785
user1的hashcode为：1619327594
user1和user2是否equals？true
通过equals为true的user2取得的值为null

```
这里只重写了equals方法。本来是想map.get(user2)打印出test,但是输出的是“null”。原因就是重写equals方法的同时忘记重写hashCode方法。

可以看看HashMap的get方法代码。

```
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}

final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }

    int hash = (key == null) ? 0 : hash(key);
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

如果想上述代码输出结果为test，只需要重写hashCode方法，让equals方法和hashCode方法始终在逻辑上保持一致。

比如重写hashCode方法。

```
@Override
public int hashCode() {
    // TODO Auto-generated method stub
    return idcard.hashCode()*19+name.hashCode();
}
```

还有一点就是，在重写hashCode方法和equals方法时，千万不要依赖易变的字段。。
再强调一下，如果hashcode值相等，对象不一定相等，还需要equals；如果equals相当，hashcode值一定相等；如果hashcode不相等，对象一定不等。
