[Java HashMap工作原理及实现](https://yikun.github.io/2015/04/01/Java-HashMap%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%8F%8A%E5%AE%9E%E7%8E%B0/)

https://www.zhihu.com/question/20733617

https://blog.csdn.net/u012961566/article/details/72963157


* HashMap实现原理和如何解决散列碰撞，HashMap底层为什么是线程不安全的
* java8对hashmap的优化
* hashmap和hashset区别，hash怎么散列的
* HashMap的rehash扩容是怎么操作的

在官方文档中的描述：
> Hash table based implementation of the Map interface. This implementation provides all of the optional map operations, and permits null values and the null key. (The HashMap class is roughly equivalent to Hashtable, except that it is unsynchronized and permits nulls.) This class makes no guarantees as to the order of the map; in particular, it does not guarantee that the order will remain constant over time.

> 基于Map接口的哈希表的实现。此实现提供所有可选的映射操作，并允许空值和空键。（hashmap类大致等同于hashtable，只是它不同步并且允许空值。）这个类不保证映射的顺序；特别是，它不保证顺序会随着时间的推移而保持不变。

### 两个重要的参数

在HashMap中有两个很重要的参数，容量(Capacity)和负载因子(Load factor)

> - Initial capacity The capacity is the number of buckets in the hash table, The initial capacity is simply the capacity at the time the hash table is created.
> - Load factor The load factor is a measure of how full the hash table is allowed to get before its capacity is automatically increased.

简单的说，Capacity就是buckets的数目，Load factor就是buckets填满程度的最大比例。如果对迭代性能要求很高的话不要把capacity设置过大，也不要把load factor设置过小。当bucket填充的数目（即hashmap中元素的个数）大于capacity*load factor时就需要调整buckets的数目为当前的2倍。

首先，我们来一起看看 HashMap 内部的结构，它可以看作是数组(Node[] table)和链表结 合组成的复合结构，数组被分为一个个桶(bucket)，通过哈希值决定了键值对在这个数组的 寻址;哈希值相同的键值对，则以链表形式存储，你可以参考下面的示意图。这里需要注意的 是，如果链表大小超过阈值(TREEIFY_THRESHOLD, 8)，图中的链表就会被改造为树形结构。

![](../img/Buckets.jpg)

### put函数的实现
接着来看 put 方法实现:

put函数大致的思路为：

1. 对key的hashCode()做hash，然后再计算index;
2. 如果没碰撞直接放到bucket里；
3. 如果碰撞了，以链表的形式存在buckets后；
4. 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树；
5. 如果节点已经存在就替换old value(保证key的唯一性)
6. 如果bucket满了(超过load factor*current capacity)，就要resize。

```java
public V put(K key, V value) {
    // 对key的hashCode()做hash
    return putVal(hash(key), key, value, false, true);
}
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // tab为空则创建
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 计算index，并对null做处理
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 节点存在
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 该链为树
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 该链为链表
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
        // 写入
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 超过load factor*current capacity，resize
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

从 putVal 方法最初的几行，我们就可以发现几个有意思的地方:
- 如果表格是 null，resize 方法会负责初始化它，这从 tab = resize() 可以看出。
- resize 方法兼顾两个职责，创建初始存储表格，或者在容量不满足需求的时候，进行扩容 (resize)。
- 在放置新的键值对的过程中，如果发生下面条件，就会发生扩容。

```java
    if (++size > threshold)
    resize();
```
- 具体键值对在哈希表中的位置(数组 index)取决于下面的位运算:

```java
    i = (n - 1) & hash
```

仔细观察哈希值的源头，我们会发现，它并不是 key 本身的 hashCode，而是来自于 HashMap 内部的另外一个 hash 方法。注意，为什么这里需要将高位数据移位到低位进行异或运算呢?这是因为有些数据计算出的哈希值差异主要在高位，而 HashMap 里的哈希寻址是忽 略容量以上的高位的，那么这种处理就可以有效避免类似情况下的哈希碰撞。

```java
    static final int hash(Object kye) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>>16;
    }
```
- 我前面提到的链表结构(这里叫 bin)，会在达到一定门限值时，发生树化，我稍后会分析 为什么 HashMap 需要对 bin 进行处理。

可以看到，putVal 方法本身逻辑非常集中，从初始化、扩容到树化，全部都和它有关。

进一步分析一下身兼多职的 resize 方法，很多朋友都反馈经常被面试官追问它的源码设计。

```java
    
final Node<K,V>[] resize() {
    // ...
    else if ((newCap = oldCap << 1) < MAXIMUM_CAPACIY &&
                oldCap >= DEFAULT_INITIAL_CAPAITY)
        newThr = oldThr << 1; // double there
       // ...
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {
        // zero initial threshold signifies using defaultsfults newCap = DEFAULT_INITIAL_CAPAITY;
        newThr = (int)(DEFAULT_LOAD_ATOR* DEFAULT_INITIAL_CAPACITY;
    }
    if (newThr ==0) {
        float ft = (float)newCap * loadFator;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?(int)ft : Intege
    }
    threshold = neThr;
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newap]; table = n;
    // 移动到新的数组结构 e 数组结构 }
```

依据 resize 源码，不考虑极端情况(容量理论最大极限由 MAXIMUM_CAPACITY 指定，数值 为 1<<30，也就是 2 的 30 次方)，我们可以归纳为:
- 门限值等于(负载因子)x(容量)，如果构建 HashMap 的时候没有指定它们，那么就是依 据相应的默认常量值。
- 门限通常是以倍数进行调整 (newThr = oldThr << 1)，我前面提到，根据 putVal 中的逻 辑，当元素个数超过门限大小时，则调整 Map 大小。
- 扩容后，需要将老的数组中的元素重新放置到新的数组，这是扩容的一个主要开销来源。


### 容量、负载因子
前面我们快速梳理了一下 HashMap 从创建到放入键值对的相关逻辑，现在思考一下，为什么 我们需要在乎容量和负载因子呢?

这是因为容量和负载系数决定了可用的桶的数量，空桶太多会浪费空间，如果使用的太满则会严
重影响操作的性能。极端情况下，假设只有一个桶，那么它就退化成了链表，完全不能提供所谓
常数时间存储的性能。
既然容量和负载因子这么重要，我们在实践中应该如何选择呢?

如果能够知道 HashMap 要存取的键值对数量，可以考虑预先设置合适的容量大小。具体数值 我们可以根据扩容发生的条件来做简单预估，根据前面的代码分析，我们知道它需要符合计算条件:

负载因子 * 容量 > 元素数量

所以，预先设置的容量需要满足，大于“预估元素数量 / 负载因子”，同时它是 2 的幂数，结论已经非常清晰了。

而对于负载因子，我建议:
- 如果没有特别需求，不要轻易进行更改，因为 JDK 自身的默认负载因子是非常符合通用场景的需求的。

- 如果确实需要调整，建议不要设置超过 0.75 的数值，因为会显著增加冲突，降低 HashMap 的性能。
- 如果使用太小的负载因子，按照上面的公式，预设容量值也进行调整，否则可能会导致更加
 频繁的扩容，增加无谓的开销，本身访问性能也会受影响。

### 树化改造
 我们前面提到了树化改造，对应逻辑主要在 putVal 和 treeifyBin 中。

 ```java
     final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            // 树化改造逻辑 
        }
    }
```

上面是精简过的 treeifyBin 示意，综合这两个方法，树化改造的逻辑就非常清晰了，可以理解 为，当 bin 的数量大于 TREEIFY_THRESHOLD 时:
- 如果容量小于 MIN_TREEIFY_CAPACITY，只会进行简单的扩容。
- 如果容量大于 MIN_TREEIFY_CAPACITY ，则会进行树化改造。 

那么，为什么 HashMap 要树化呢?

本质上这是个安全问题。因为在元素放置过程中，如果一个对象哈希冲突，都被放置到同一个桶
里，则会形成一个链表，我们知道链表查询是线性的，会严重影响存取的性能。

而在现实世界，构造哈希冲突的数据并不是非常复杂的事情，恶意代码就可以利用这些数据大量 与服务器端交互，导致服务器端 CPU 大量占用，这就构成了哈希碰撞拒绝服务攻击，国内一线 互联网公司就发生过类似攻击事件。
