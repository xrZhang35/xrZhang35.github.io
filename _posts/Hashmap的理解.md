# Hashmap的理解

##先来看一下Java8的官方doc

>   Hash table based implementation of the `Map` interface. This implementation provides all of the optional map operations, and permits `null` values and the `null` key. (The `HashMap` class is roughly equivalent to `Hashtable`, except that it is unsynchronized and permits nulls.) This class makes no guarantees as to the order of the map; in particular, it does not guarantee that the order will remain constant over time.
>
>   This implementation provides constant-time performance for the basic operations (`get` and `put`), assuming the hash function disperses the elements properly among the buckets. Iteration over collection views requires time proportional to the "capacity" of the `HashMap` instance (the number of buckets) plus its size (the number of key-value mappings). Thus, it's very important not to set the initial capacity too high (or the load factor too low) if iteration performance is important.
>
>   An instance of `HashMap` has two parameters that affect its performance: *initial capacity* and *load factor*. The *capacity* is the number of buckets in the hash table, and the initial capacity is simply the capacity at the time the hash table is created. The *load factor* is a measure of how full the hash table is allowed to get before its capacity is automatically increased. When the number of entries in the hash table exceeds the product of the load factor and the current capacity, the hash table is *rehashed* (that is, internal data structures are rebuilt) so that the hash table has approximately twice the number of buckets.
>
>   As a general rule, the default load factor (.75) offers a good tradeoff between time and space costs. Higher values decrease the space overhead but increase the lookup cost (reflected in most of the operations of the `HashMap` class, including `get` and `put`). The expected number of entries in the map and its load factor should be taken into account when setting its initial capacity, so as to minimize the number of rehash operations. If the initial capacity is greater than the maximum number of entries divided by the load factor, no rehash operations will ever occur.
>
>   If many mappings are to be stored in a `HashMap` instance, creating it with a sufficiently large capacity will allow the mappings to be stored more efficiently than letting it perform automatic rehashing as needed to grow the table. Note that using many keys with the same `hashCode()` is a sure way to slow down performance of any hash table. To ameliorate impact, when keys are [`Comparable`](https://docs.oracle.com/javase/8/docs/api/java/lang/Comparable.html), this class may use comparison order among keys to help break ties.
>
>   **Note that this implementation is not synchronized.** If multiple threads access a hash map concurrently, and at least one of the threads modifies the map structurally, it *must* be synchronized externally. (A structural modification is any operation that adds or deletes one or more mappings; merely changing the value associated with a key that an instance already contains is not a structural modification.) This is typically accomplished by synchronizing on some object that naturally encapsulates the map. If no such object exists, the map should be "wrapped" using the [`Collections.synchronizedMap`](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#synchronizedMap-java.util.Map-) method. This is best done at creation time, to prevent accidental unsynchronized access to the map:
>
>   ```
>      Map m = Collections.synchronizedMap(new HashMap(...));
>   ```
>
>   The iterators returned by all of this class's "collection view methods" are *fail-fast*: if the map is structurally modified at any time after the iterator is created, in any way except through the iterator's own `remove` method, the iterator will throw a [`ConcurrentModificationException`](https://docs.oracle.com/javase/8/docs/api/java/util/ConcurrentModificationException.html). Thus, in the face of concurrent modification, the iterator fails quickly and cleanly, rather than risking arbitrary, non-deterministic behavior at an undetermined time in the future.
>
>   Note that the fail-fast behavior of an iterator cannot be guaranteed as it is, generally speaking, impossible to make any hard guarantees in the presence of unsynchronized concurrent modification. Fail-fast iterators throw `ConcurrentModificationException` on a best-effort basis. Therefore, it would be wrong to write a program that depended on this exception for its correctness: *the fail-fast behavior of iterators should be used only to detect bugs.*
>
>   This class is a member of the [Java Collections Framework](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html).
>
>   https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html

`HashMap`是基于`Hash Table`(注意，不是`HashTable`，而是说哈希表格的意思)来实现一种`Map`接口。`HashMap`提供了几乎所有可供选择的`map`类的方法，并且允许空值和空键(空键只允许有一个)。(`HashMap`和`HashTable`很像，但是`HashMap`不是同步的，但是允许`null`值。)这个类不保证映射的顺序;特别是，它不能保证顺序在一段时间内保持不变。

>   ==这里还是不明白==

------

## 再来看一下美团技术团队给出的文档

https://tech.meituan.com/2016/06/24/java-hashmap.html

### 1.`Map`接口的常用实现类

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/f7fe16a2.png)

-   `HashMap`：根据键的`hashCode`值来定位位置，进行`value`的操作；

​						允许至多一个键为`null`，允许任意多条记录值为`null`

​						非线程安全，可以使用`Collections`的`synchronizedMap`方法或者`ConcurrentHashMap`来解决并发问题。

-   `HashTable`：遗留类，很多功能与`HashMap`重复，不建议使用

​							线程安全，并发性不如引进了分段锁的`ConcurrentHashMap`

​							键、值都不允许为`null`

-   `LinkedHashMap`：是`HashMap`的一个子类，保存了记录的插入顺序
-   `TreeMap`：TreeMap实现SortedMap接口，能够把它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。

​						`key`必须实现`Comparable`接口或在构造`TreeMap`的时候传入`Comparator`比较器

### 2.`HashMap`的内部实现

弄清楚`HashMap`，首先需要知道`HashMap`是什么，其次弄明白`HashMap`能用来做什么。

#### (一).存储结构--字段

`HashMap`是数组 + 链表 + 红黑树

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/e4a19398.png)

##### (1).Node[] table 哈希桶数组

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;    //用来定位数组索引位置
        final K key;
        V value;
        Node<K,V> next;   //链表的下一个node

        Node(int hash, K key, V value, Node<K,V> next) { ... }
        public final K getKey(){ ... }
        public final V getValue() { ... }
        public final String toString() { ... }
        public final int hashCode() { ... }
        public final V setValue(V newValue) { ... }
        public final boolean equals(Object o) { ... }
}
```

`HashMap`就是使用这个table来存储的。

`HashMap`为了解决哈希碰撞，采用了链地址法。如果发生了哈希碰撞，那么`HashMap`的存取效率就会下降一些，所以我们需要减少哈希碰撞的产生。

>   哈希碰撞的根本原因在于hash算法的问题，在后面我们会讲到主要涉及到`key`对象的`hashCode()`方法和`table`的长度，所以我们可以增加数组的长度，可是当数组长度过长的时候
>
>   可以说像上面这样讲是不正确的，原因是在这里我直接就使用了jdk里边的实现方式，但现在我们要探究的就是如何能够有一种比较好的扩容方式，我这里就是在拿答案套问题

如果哈希桶数组很大，即使较差的哈希算法也会比较分散，如果哈希桶数组数组很小，即使好的哈希算法也会出现较多碰撞，所以就需要在空间成本和时间成本之间权衡，其实就是在根据实际情况确定哈希桶数组的大小，并在此基础上设计好的哈希算法减少哈希碰撞。那么通过什么方式来控制`HashMap`使得哈希碰撞的概率又小，哈希桶数组占用空间又少呢？答案就是好的哈希算法和扩容机制。

##### (2).扩容字段

```java
     int threshold;             
     final float loadFactor;   
     int modCount;  
     int size; 
```

-   `loadFactor`：负载因子，默认为0.75

    默认的负载因子0.75是对空间和时间效率的一个平衡选择，建议大家不要修改，除非在时间和空间比较特殊的情况下，如果内存空间很多而又对时间效率要求很高，可以降低负载因子Load factor的值；相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子loadFactor的值，这个值可以大于1。

    >   为什么是0.75，可以看这篇文章
    >
    >   https://www.jianshu.com/p/effb601f2c48

-   `threshold`：`length` * `loadFactor`

-   `size`：`HashMap`中实际存在的键值对的数量

-   `modCount`：用于`FailFast`机制，接下来会在专门的文章中讲

这里存在一个问题，即使负载因子和Hash算法设计的再合理，也免不了会出现拉链过长的情况，一旦出现拉链过长，则会严重影响HashMap的性能。于是，在JDK1.8版本中，对数据结构做了进一步的优化，引入了红黑树。而当链表长度太长（默认超过8）时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高HashMap的性能，其中会用到红黑树的插入、删除、查找等算法。

>   也就是说当出现问题的时候，我们要考察到底是什么问题，超过threshold和loadFactor了吗，还是讲拉链过长了，这两种情况是不一样的。

#### (二).功能实现--方法

##### 1.确定哈希桶数组索引位置

定位到哈希桶数组会利用到`hash()`方法，其分为三步：

-   取`key`的`hashCode`值
-   高位运算
-   取模运算

在JDK1.8的实现中，优化了高位运算的算法，通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在数组table的length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/45205ec2.png)

##### 2.分析`HashMap`的`put`方法

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/d669d29c.png)

```java
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

##### 3.扩容机制

