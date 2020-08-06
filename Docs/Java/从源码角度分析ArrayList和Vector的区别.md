## ArrayList和Vector的区别

> 文章已同步发表于微信公众号JasonGaoH， [ArrayList和Vector的区别](https://mp.weixin.qq.com/s?__biz=MzUyNTE2OTAzMQ==&mid=2247483726&idx=1&sn=0fa987066009e2e8c347a7310f5701fe&chksm=fa2379a6cd54f0b0464108f0d82ddea6de723cc181c0a3d873fd1078985c4ac9d007a7a0395d&token=1938879438&lang=zh_CN#rd)

ArrayList和Vector这两个集合本质上并没有什么太大的不停，他们都实现了List接口，而且底层都是基于Java数组来存储集合元素。

在ArrayList集合类的源代码中也可以看到下面一行：
```java
    transient Object[] elementData; // non-private to simplify nested class access
```

在Vector集合类的源代码中也可以看到类似的一行：
```java
    protected Object[] elementData;
```

从上面的代码中可以看出，ArrayList使用transient修饰了elementData数组，这保证系统序列化ArrayList对象时不会直接序列化elementData数组，而是通过ArrayList提供的writeObject、readObject方法来实现定制序列化；

但对于Vector而言，它没有使用transient修饰elementData数据，而且Vector只提供了一个writeObject方法，并未完全实现订制序列化。

从序列化机制的角度来看，ArrayList的实现比Vector的实现更安全。

除此之外，Vector其实就是ArrayList的线程安全版本，ArrayList和Vector绝大部分方法都是相同的，只是Vector的方法增加了synchronized修饰。

下面先来看ArrayList中的add(int index,E element)方法的源代码。
```java
      public void add(int index, E element) {
        //检查是否下标越界
        rangeCheckForAdd(index);
        //保证ArrayList底层的数组可以保存所有集合元素
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //将elementData数组中的index位置之后的所有元素向后移动一位
        //也就是将elementData数组的index位置的元素空出来
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        //将新元素将入elementData数组的index的位置
        elementData[index] = element;
        size++;
    }
```

再来看Vector中的add(int index,E element)方法的源代码：
```java
     public void add(int index, E element) {
        insertElementAt(element, index);
    };
```
 从上面可以看出，Vector的add(int index,E element)方法其实就是insertElementAt(int index,E element).

 接着来看insertElementAt(int index,E element)的源码。
 ```java
        public synchronized void insertElementAt(E obj, int index) {
        //增加集合的修改次数
        modCount++;
        //如果添加位置大于集合长度，则抛出异常
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        //保证Vector底层的数组可以保存所有集合元素
        ensureCapacityHelper(elementCount + 1);
         //将elementData数组中的index位置之后的所有元素向后移动一位
        //也就是将elementData数组的index位置的元素空出来
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
          //将新元素将入elementData数组的index的位置
        elementData[index] = obj;
        elementCount++;
    }

```

将ArrayList中的add(int index,E element)方法和Vector的insertElementAt(int index,E element)方法进行对比，可以发现Vector的insertElementAt(int index,E element)方法只是多了个synchronized修饰，而且多了一行代码modCount++，这并不代表ArrayList中的add(int index,E element)方法没有这行代码，ArrayList只是将这行代码放在ensureCapacityInternal中完成。

接下来我们看ensureCapacityInternal(int minCapacity)方法的源码：
```java
    private void ensureCapacityInternal(int minCapacity) {
        //当elementData数组为空，如果传进来的minCapacity＜=10时，minCapacity取10，否则取传进来的参数
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        //如果minCapacity大于原数组的长度，则需要扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
   private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //将新容量扩充为原来的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //如果新的newCapacity依然小于minCapacity，直接将minCapacity赋值给newCapacity
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //如果新的newCapacity超过最大的数组长度，则进行更大的扩容 
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        //通过Arrays.copyOf扩充一个新数组，数组的长度为newCapacity
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

```java
     private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
          //如果minCapacity大于原数组的长度，则需要扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //如果capacityIncrement大于0，则新的newCapacity等于旧的oldCapacity加上capacityIncrement，
        //如果不是，则新的newCapacity等于旧的oldCapacity*2，表示扩容两倍
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
         //如果新的newCapacity依然小于minCapacity，直接将minCapacity赋值给newCapacity           
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //如果新的newCapacity超过最大的数组长度，则进行更大的扩容
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        //通过Arrays.copyOf扩充一个新数组，数组的长度为newCapacity    
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

将ArrayList中的ensureCapacityInternal和Vector中的ensureCapacityHelper方法进行对比，可以发现这两个方法几乎完全相同，只是在扩充底层数组的容量时略有区别而已。ArrayList总是将底层数组的容量扩充为原来的1.5倍，但Vector则多了一个选择。

当capacityIncrement大于0时，扩充后的容量等于原来的容量加上这个capacityIncrement的值，如果不是大于0，则扩充为原来容量的2倍。

Vector的ensureCapacityHelper方法在扩充数组容量时多一个选择是因为，创建Vector可以传入一个capacityIncrement参数，如下构造方法：
> Vector(int initialCapacity, int capacityIncrement)：以initialCapacity作为底层数组的初始长度，以capacityIncrement作为扩充数组时的增大步长来创建Vector对象。

但是对于ArrayList而言，它的构造方法最多只能指定一个initialCapacity参数。


### 注意
即使需要在多线程环境下使用List集合，而且需要保证List集合的线程安全，依然可以避免使用Vector，而是考虑将ArrayList包装成线程安全的集合类。Java提供了一个Collections工具类，通过该工具synchronizedList方法可以将一个普瑞的ArrayList包装成线程安全的ArrayList。
  
    