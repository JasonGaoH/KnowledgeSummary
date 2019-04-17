## ArrayList和Vector的区别

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