# LruCache

#### 相关知识点
* 最近最少使用原则
* LinkedHashMap

#### 最近最少使用原则
近期一段时间（简单理解为容器容量），最少使用的排在最后，最多使用的排在最前。当容器负载达到上限时，首先移除排在最后的节点。

#### LinkedHashMap
  因为LruCache中Lru算法的实现就是通过LinkedHashMap来实现的。<br><br>
  LinkedHashMap继承于HashMap，它使用了一个双向链表来存储Map中的Entry顺序关系，这种顺序有两种，一种是LRU顺序，一种是插入顺序，这可以由其构造函数public LinkedHashMap(int initialCapacity,float loadFactor, boolean accessOrder)指定。<br><br>
  所以，对于get、put、remove等操作，LinkedHashMap除了要做HashMap做的事情，还做些调整Entry顺序链表的工作。LruCache中将LinkedHashMap的顺序设置为LRU顺序来实现LRU缓存，每次调用get(也就是从内存缓存中取图片)，则将该对象移到链表的尾端。调用put插入新的对象也是存储在链表头部，这样当内存缓存达到设定的最大值时，将链表头部的尾端（最少用到的）移除。<br><br>
  
<b>LruCache:</b>
    
```
package android.support.v4.util;

import java.util.LinkedHashMap;
import java.util.Map;


public class LruCache<K, V> {
    private final LinkedHashMap<K, V> map; 
        
    /** Size of this cache in units. Not necessarily the number of elements. */
    private int size;    //当前cache的大小
    private int maxSize; //cache最大大小

    private int putCount;       //put的次数
    private int createCount;    //create的次数
    private int evictionCount;  //回收的次数
    private int hitCount;       //命中的次数
    private int missCount;      //未命中次数

    /**
     * @param maxSize for caches that do not override {@link #sizeOf}, this is
     *     the maximum number of entries in the cache. For all other caches,
     *     this is the maximum sum of the sizes of the entries in this cache.
     */
    public LruCache(int maxSize) {
        if (maxSize <= 0) {
            throw new IllegalArgumentException("maxSize <= 0");
        }
        this.maxSize = maxSize;
        //将LinkedHashMap的accessOrder设置为true来实现LRU
        this.map = new LinkedHashMap<K, V>(0, 0.75f, true);  
    }

    /**
     * Returns the value for {@code key} if it exists in the cache or can be
     * created by {@code #create}. If a value was returned, it is moved to the
     * head of the queue. This returns null if a value is not cached and cannot
     * be created.
     * 通过key获取相应的item，或者创建返回相应的item。相应的item会移动到队列的尾部，
     * 如果item的value没有被cache或者不能被创建，则返回null。
     */
    public final V get(K key) {
        if (key == null) {
            throw new NullPointerException("key == null");
        }

        V mapValue;
        
        // 对Map的get()操作同步
        synchronized (this) {
            mapValue = map.get(key);
            if (mapValue != null) {
                //mapValue不为空表示命中，hitCount+1并返回mapValue对象
                hitCount++;
                return mapValue;
            }
            missCount++;  //未命中
        }

        /*
         * Attempt to create a value. This may take a long time, and the map
         * may be different when create() returns. If a conflicting value was
         * added to the map while create() was working, we leave that value in
         * the map and release the created value.
         * 如果未命中，则试图创建一个对象，这里create方法返回null,并没有实现创建对象的方法
         * 如果需要事项创建对象的方法可以重写create方法。因为图片缓存时内存缓存没有命中会去
         * 文件缓存中去取或者从网络下载，所以并不需要创建。
         */
        V createdValue = create(key);
        if (createdValue == null) {
            return null;
        }
        //假如创建了新的对象，则继续往下执行
        synchronized (this) {
            createCount++;  
            //将createdValue加入到map中，并且将原来键为key的对象保存到mapValue
            mapValue = map.put(key, createdValue);   
            if (mapValue != null) {
                // There was a conflict so undo that last put
                //如果mapValue不为空，则撤销上一步的put操作。
                map.put(key, mapValue);
            } else {
                //加入新创建的对象之后需要重新计算size大小
                size += safeSizeOf(key, createdValue);
            }
        }

        if (mapValue != null) {
            entryRemoved(false, key, createdValue, mapValue);
            return mapValue;
        } else {
            //每次新加入对象都需要调用trimToSize方法看是否需要回收
            trimToSize(maxSize);
            return createdValue;
        }
    }

    /**
     * Caches {@code value} for {@code key}. The value is moved to the head of
     * the queue.
     *
     * @return the previous value mapped by {@code key}.
     */
    public final V put(K key, V value) {
        if (key == null || value == null) {
            throw new NullPointerException("key == null || value == null");
        }

        V previous;
        synchronized (this) {
            putCount++;
            size += safeSizeOf(key, value);  //size加上预put对象的大小
            previous = map.put(key, value);
            if (previous != null) {
                //如果之前存在键为key的对象，则size应该减去原来对象的大小
                size -= safeSizeOf(key, previous);
            }
        }

        if (previous != null) {
            entryRemoved(false, key, previous, value);
        }
        //每次新加入对象都需要调用trimToSize方法看是否需要回收
        trimToSize(maxSize);
        return previous;
    }

    /**
     * @param maxSize the maximum size of the cache before returning. May be -1
     *     to evict even 0-sized elements.
     * 此方法根据maxSize来调整内存cache的大小，如果maxSize传入-1，则清空缓存中的所有对象
     */
    private void trimToSize(int maxSize) {
        while (true) {
            K key;
            V value;
            synchronized (this) {
                if (size < 0 || (map.isEmpty() && size != 0)) {
                    throw new IllegalStateException(getClass().getName()
                            + ".sizeOf() is reporting inconsistent results!");
                }
                //如果当前size小于maxSize或者map没有任何对象,则结束循环
                if (size <= maxSize || map.isEmpty()) {
                    break;
                }
                //移除链表头部的元素，并进入下一次循环
                Map.Entry<K, V> toEvict = map.entrySet().iterator().next();
                key = toEvict.getKey();
                value = toEvict.getValue();
                map.remove(key);
                size -= safeSizeOf(key, value);
                evictionCount++;  //回收次数+1
            }

            entryRemoved(true, key, value, null);
        }
    }

    /**
     * Removes the entry for {@code key} if it exists.
     *
     * @return the previous value mapped by {@code key}.
     * 从内存缓存中根据key值移除某个对象并返回该对象
     */
    public final V remove(K key) {
        if (key == null) {
            throw new NullPointerException("key == null");
        }

        V previous;
        synchronized (this) {
            previous = map.remove(key);
            if (previous != null) {
                size -= safeSizeOf(key, previous);
            }
        }

        if (previous != null) {
            entryRemoved(false, key, previous, null);
        }

        return previous;
    }

    /**
     * Called for entries that have been evicted or removed. This method is
     * invoked when a value is evicted to make space, removed by a call to
     * {@link #remove}, or replaced by a call to {@link #put}. The default
     * implementation does nothing.
     *
     * <p>The method is called without synchronization: other threads may
     * access the cache while this method is executing.
     *
     * @param evicted true if the entry is being removed to make space, false
     *     if the removal was caused by a {@link #put} or {@link #remove}.
     * @param newValue the new value for {@code key}, if it exists. If non-null,
     *     this removal was caused by a {@link #put}. Otherwise it was caused by
     *     an eviction or a {@link #remove}.
     */
    protected void entryRemoved(boolean evicted, K key, V oldValue, V newValue) {}

    /**
     * Called after a cache miss to compute a value for the corresponding key.
     * Returns the computed value or null if no value can be computed. The
     * default implementation returns null.
     *
     * <p>The method is called without synchronization: other threads may
     * access the cache while this method is executing.
     *
     * <p>If a value for {@code key} exists in the cache when this method
     * returns, the created value will be released with {@link #entryRemoved}
     * and discarded. This can occur when multiple threads request the same key
     * at the same time (causing multiple values to be created), or when one
     * thread calls {@link #put} while another is creating a value for the same
     * key.
     */
    protected V create(K key) {
        return null;
    }

    private int safeSizeOf(K key, V value) {
        int result = sizeOf(key, value);
        if (result < 0) {
            throw new IllegalStateException("Negative size: " + key + "=" + value);
        }
        return result;
    }

    /**
     * Returns the size of the entry for {@code key} and {@code value} in
     * user-defined units.  The default implementation returns 1 so that size
     * is the number of entries and max size is the maximum number of entries.
     *
     * <p>An entry's size must not change while it is in the cache.
     * 用来计算单个对象的大小，这里默认返回1，一般需要重写该方法来计算对象的大小
     * xUtils中创建LruMemoryCache时就重写了sizeOf方法来计算bitmap的大小
     * mMemoryCache = new LruMemoryCache<MemoryCacheKey, Bitmap>(globalConfig.getMemoryCacheSize()) {
     *       @Override
     *       protected int sizeOf(MemoryCacheKey key, Bitmap bitmap) {
     *           if (bitmap == null) return 0;
     *           return bitmap.getRowBytes() * bitmap.getHeight();
     *       }
     *   };
     *
     */
    protected int sizeOf(K key, V value) {
        return 1;
    }

    /**
     * Clear the cache, calling {@link #entryRemoved} on each removed entry.
     * 清空内存缓存
     */
    public final void evictAll() {
        trimToSize(-1); // -1 will evict 0-sized elements
    }

    /**
     * For caches that do not override {@link #sizeOf}, this returns the number
     * of entries in the cache. For all other caches, this returns the sum of
     * the sizes of the entries in this cache.
     */
    public synchronized final int size() {
        return size;
    }

    /**
     * For caches that do not override {@link #sizeOf}, this returns the maximum
     * number of entries in the cache. For all other caches, this returns the
     * maximum sum of the sizes of the entries in this cache.
     */
    public synchronized final int maxSize() {
        return maxSize;
    }

    /**
     * Returns the number of times {@link #get} returned a value.
     */
    public synchronized final int hitCount() {
        return hitCount;
    }

    /**
     * Returns the number of times {@link #get} returned null or required a new
     * value to be created.
     */
    public synchronized final int missCount() {
        return missCount;
    }

    /**
     * Returns the number of times {@link #create(Object)} returned a value.
     */
    public synchronized final int createCount() {
        return createCount;
    }

    /**
     * Returns the number of times {@link #put} was called.
     */
    public synchronized final int putCount() {
        return putCount;
    }

    /**
     * Returns the number of values that have been evicted.
     */
    public synchronized final int evictionCount() {
        return evictionCount;
    }

    /**
     * Returns a copy of the current contents of the cache, ordered from least
     * recently accessed to most recently accessed.
     */
    public synchronized final Map<K, V> snapshot() {
        return new LinkedHashMap<K, V>(map);
    }

    @Override public synchronized final String toString() {
        int accesses = hitCount + missCount;
        int hitPercent = accesses != 0 ? (100 * hitCount / accesses) : 0;
        return String.format("LruCache[maxSize=%d,hits=%d,misses=%d,hitRate=%d%%]",
                maxSize, hitCount, missCount, hitPercent);
    }
}

```