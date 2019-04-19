List代表一种线性表的数据结构，ArrayList则是一种顺序存储的线性表。ArrayList底层采用数组来保存每个集合的元素，LinkedList则是一种链式存储的线性表。其本质上就是一个双向链表，但它不仅实现了List接口，还是想了Deque接口。也就是说LinkedList既可以当成双向链表使用，也可以当成队列使用，还可以当成栈来适用于（Deque代表双端队列，即具有队列的特征，也具有栈的特征）。

在[从源码角度分析ArrayList和Vector的区别](./从源码角度分析ArrayList和Vector的区别.md)中已经分析过，ArrayList底层采用一个elementData数组来保存所有集合的元素，因此ArrayList在插入元素时需要完成下面两件事情。

- 保证ArrayList底层封装的数组长度大于集合元素的个数；
- 将插入位置之后的所有数组元素“整体搬家”，向后移动一“格”。

反过来，当删除ArrayList集合中指定的元素时，程序也需要“整体搬家”，而且还需要将被删除索引处的数组元素置为null。下面是ArrayList集合的remove(int idnex)方法的源码。

```java
     public E remove(int index) {
         //如果index是大于或者等于size，抛出异常
        rangeCheck(index);

        modCount++;
        //保存索引处的元素
        E oldValue = elementData(index);
        //计算需要“整体搬家”的元素个数
        int numMoved = size - index - 1;
        //当numMoved大于0时，开始搬家
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        //释放被删除元素，以便GC回收该元素  
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```
 
 从上面的代码来看，对于ArrayList而言，当程序向ArrayList中添加、删除集合元素时，ArrayList底层都需要对数组进行“整体搬家”，因此性能比较差。

 但如果程序调用get(int index)方法来取出ArrayList集合中的元素时，性能和数据几乎相同--非常快。

```java
     public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```
  
LinkedList本质上是一个双向列表，因此它使用如下内部类来保存每个集合元素。
```java
    private static class Node<E> {
        //集合元素   
        E item;
        //保存指向下一个链表节点的引用
        Node<E> next;
        //保存指向上一个节点的引用
        Node<E> prev;
        //构造方法
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

由于LinkedList采用双向链表来保存集合元素，因此它在添加集合元素的时候，只需要对链表进行如下图所示的操作即可添加一个新节点。

