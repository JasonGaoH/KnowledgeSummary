#### LinkedHashMap

1. LinkedHashMap继承于HashMap。

2. 它使用了一个双向链表来存储Map中的Entry顺序关系。

3. 这种顺序有两种，一种是LRU顺序，一种是插入顺序。这可以由其构造函数public LinkedHashMap(int initialCapacity,float loadFactor, boolean accessOrder)指定。

4. access-order为true，是LRU顺序，可作为LRU cache技术方案，具备最近最少使用原则。(Map节点默认存储顺序是inserting-order)

5. access-order原理：每次访问(get/put)节点，被访问节点都会被放到链表首部，当节点数量超过容量，会移除尾部的节点；<br>

LinkedHashMap:

```
public V get(Object key) {
        LinkedHashMapEntry<K,V> e = (LinkedHashMapEntry<K,V>)getEntry(key);
        if (e == null)
            return null;
        e.recordAccess(this);
        return e.value;
    }
```

LinkedHashMapEntry:

``` 
void recordAccess(HashMap<K,V> m) {
            LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
            if (lm.accessOrder) {
                lm.modCount++;
                remove();
                addBefore(lm.header);
            }
        }
        
/**
  * Removes this entry from the linked list.
  */
 private void remove() {
     before.after = after;
     after.before = before;
 }
 
 /**
  * Inserts this entry before the specified existing entry in the list.
  */
 private void addBefore(LinkedHashMapEntry<K,V> existingEntry) {
     after  = existingEntry;
     before = existingEntry.before;
     before.after = this;
     after.before = this;
 }
```

2.内含双向链表； <br>

```
@Override
    void init() {
        header = new LinkedHashMapEntry<>(-1, null, null, null);
        header.before = header.after = header;
    }    
```