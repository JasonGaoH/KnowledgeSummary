# 红黑树

在讲红黑树数之前，需要需要先熟悉以下几个概念：

## 二叉树
二叉树指的是每个节点最多只能有两个字数的有序树。通常左边的子树称为``左子树``,右边的子树称为``右子树``。这里说的有序树强调的是二叉树的左子树和右子树的次序不能随意颠倒。

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_tree_node.png)

代码定义：
```java
class Node {
    T data;
    Node left;
    Node right;
}
```

## 排序二叉树

排序二叉树是一种特殊结构的二叉树，我们可以对树中所有节点进行排序和检索。
>性质
* 若它的左子树不空，则左子树上所有节点的值均小于它的根节点的值；
* 若她的右子树不空，则右子树上所有节点的值均大于它的根节点的值；
* 具有递归性，排序二叉树的左子树、右子树也是排序二叉树。

![排序二叉树](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_tree.png)

### 排序二叉树退化成链表
对于排序二叉树来说，为了保证二叉树是有序的，也就是它的左子树上所有节点的值小于根节点的值，右子树上所有节点的值大于根节点的值，在往树中添加节点或者删除节点的时候，需要对树的结构进行调整，这个时候可能会使排序二叉树退化成链表。

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_tree_v1.png)

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_link_tree.png)

如上面两图，正常情况下，排序二叉树是左边图这样的，但是，在某些情况下，排序二叉树就变成了右边这样了，就变成了普通的链表结构。
> 注意：这里说的某些情况表示的是当插入的一系列数据本身就是有序的，要么是从小到大排序的，要么是从大大小排序，那么最后得到的排序二叉树将变成链表。如上面图所示，这个排序二叉树变成了只有右节点的普通链表。

正常情况下的排序二叉树检索效率类似于二分查找，二分查找的时间复杂度为O(log n)，但是如果排序二叉树退化成链表结构，那么检索效率就变成了线性的O(n)的，这样相对于O(log n)来说，检索效率肯定是要差不少的。

> 思考，二分查找和正常的排序二叉树的时间复杂度都是O(log n)，那么为什么是O(log n)？

为了解决这个问题，于是就有了``平衡二叉树``。

## 平衡二叉树
平衡二叉数又被称为AVL树，AVL树的名字来源于它的发明作者G.M. Adelson-Velsky 和 E.M. Landis，取自两人名字的首字母。

官方定义：它或者是一颗空树，或者具有以下性质的排序二叉树：它的左子树和右子树的深度之差(平衡因子)的绝对值不超过1，且它的左子树和右子树都是一颗平衡二叉树。

两个条件：
* 平衡二叉树必须是排序二叉树，也就是说平衡二叉树他的左子树所有节点的值必须小于根节点的值，它的右子树上所有节点的值必须大于它的根节点的值。
* 左子树和右子树的深度之差的绝对值不超过1。

## 红黑树

> 为什么会有红黑树
其实红黑树和上面的平衡二叉树类似，本质上都是为了解决排序二叉树在极端情况下退化成链表导致检索效率大大降低的问题，红黑树最早是由Rudolf Bayer于1972年发明的。

红黑树首先肯定是一个排序二叉树，它在每个节点上增加了一个存储位来表示节点的颜色，可以是RED或BLACK

Java中实现红黑树大概结构图如下所示：

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/java_tree_map.jpg)

### 红黑树的特性
- 性质1：每个节点要么是红色，要么是黑色。
- 性质2：根节点永远是黑色的。
- 性质3：所有的叶子节点都是空节点（即null），并且是黑色的。
- 性质4：每个红色节点的两个子节点都是黑色。（从每个叶子到根的路径上不会有两个连续的红色节点。）
- 性质5：从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点。

注意：针对上面的5种性质，我们简单理解下，对于性质1和性质2，相当于是对红黑树每个节点的约束，根节点是黑色，其他的节点要么是红色，要么是黑色。

对于性质3中指定红黑树的每个叶子节点都是空节点，而且叶子节点都是黑色，但Java实现的红黑树会使用null来代表空节点，因此我们在遍历Java里的红黑树的时候会看不到叶子节点，而看到的是每个叶子节点都是红色的，这一点需要注意。

对于性质5，这里我们需要注意的是，这里的描述是从任一节点，从任一节点到它的子树的每个叶子节点黑色节点的数量都是相同的，这个数量被称为这个节点的黑高。

如果我们从根节点出发到每个叶子节点的路径都包含相同数量的黑色节点，这个黑色节点的数量被称为树的黑色高度。树的黑色高度和节点的黑色高度是不一样的，这里要注意区分。

接着我们来看下面这张图，
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree.png)

这个树的黑色高度为3，从根节点到叶子节点的最短路径长度是2，该路径上全是黑色节点，包括叶子节点，从根节点到叶子节点最长路径为4，每个黑色节点之间会插入红色节点。
通过上面的性质4和性质5，其实上保证了没有任何一条路径会比其他路径长出两倍，所以基本上能保证红黑树是平衡的。

> 其实红黑树并不是真正的平衡二叉树，它只能保证大致是平衡的，因为红黑树的高度不会无限增高，在实际应用用，红黑树的统计性能要高于平衡二叉树，但极端性能略差。

红黑树的性质4和性质5就是对树的平衡进行约束，从树的黑色高度出发，我们可以得出一个结论：
``对于给定的黑色高度为N的红黑树，从根到叶子节点的最短路径长度为N-1,最长路径为2*（N-1）。``

### 红黑树的插入
为什么说新加入到红黑树中的节点为红色节点

从性质5中知道，当前红黑树中从根节点到每个叶子节点的黑色节点数量是一样的，此时假如新的黑色节点的话，必然破坏规则，但加入红色节点却不一定，除非其父节点就是红色节点，因此加入红色节点，破坏规则的可能性小一些。

给定下面这样一个红黑树：
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_insert_1.png)

当我们插入值为66的节点的时候，示意图如下：

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_insert_2.png)

很明显，这个时候结构依然遵循着上述5大特性，无需启动自动平衡机制调整节点平衡状态。

如果再向里面插入值为51的节点呢，这个时候红黑树变成了这样。
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_insert_3.png)

这样的结构实际上是不满足性质4的，红色两个子节点必须是黑色的，而这里49这个红色节点现在有个51的黑色节点。

这个时候我们需要调整这个树的结构来保证红黑树的平衡。

红黑树保证平衡的手段一个是变色一个是旋转，我们现在在上面这个场景下先来讲讲变色。

* 首先解决结构不遵循性质4这一点（红色节点相连，节点49-51），需将节点49改为黑色
* 此时我们发现又违反了性质5（56-45-49-51-XX路径中黑色节点超过了其他路径），那么我们将节点45改为红色节点
* 这里又违反了性质4（红色节点相连，节点56-45-43），那么我们将节点56和节点43改为黑色节点
* 但是我们发现此时又违反了性质5（60-56-XX路径的黑色节点比60-68-XX的黑色节点多），因此我们需要调整节点68为黑色
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_insert_4.png)

最终调整的树为：
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_insert_5.png)

但并不是什么时候都那么幸运，可以直接通过变色就达成目的，大多数时候还需要通过旋转来解决。

接下来我们来讲讲如何使用旋转操作来保持树的平衡。

现在我们要在下面这颗红黑树上加入节点65。
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_insert_2.png)

插入节点65后进行以下步骤
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_insert_6.png)

这个时候，你会发现对于节点64无论是红色节点还是黑色节点，都会违反规则5，路径中的黑色节点始终无法达成一致，这个时候仅通过【变色】已经无法达成目的。我们需要通过旋转操作，当然【旋转】操作一般还需要搭配【变色】操作。

旋转包括【左旋】和【右旋】

#### 左旋
逆时针旋转两个节点，让一个节点被其右子节点取代，而该节点成为右子节点的左子节点

首先断开节点PL与右子节点G的关系，同时将其右子节点的引用指向节点C2；然后断开节点G与左子节点C2的关系，同时将G的左子节点的应用指向节点PL
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_rotate_left.png)

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_rotate_left.gif)

#### 右旋
顺时针旋转两个节点，让一个节点被其左子节点取代，而该节点成为左子节点的右子节点

首先断开节点G与左子节点PL的关系，同时将其左子节点的引用指向节点C2；然后断开节点PL与右子节点C2的关系，同时将PL的右子节点的应用指向节点G。
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_rotate_right.png)

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_rotate_right.gif)

无法通过变色而进行旋转的场景分为以下四种：

* 左左节点旋转

这种情况下，父节点和插入的节点都是左节点，如下图这种情况下，我们要插入节点65。
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_rotate_1.png)

以祖父节点【右旋】，搭配【变色】

操作步骤如下：
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_rotate_2.png)

* 左右节点旋转

这种情况下，父节点是左节点，插入的节点是右节点，还是在上面旋转图中，我们要插入节点67。

先父节点【左旋】，然后祖父节点【右旋】，搭配【变色】

操作步骤如下：
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_rotate_3.png)

* 右左节点旋转
这种情况下，父节点是右节点，插入的节点是左节点，如下图这种情况，我们要插入节点68。
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_rotate_4.png)

按照规则，步骤如下:
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_rotate_5.png)

* 右右节点旋转
这种情况下，父节点和插入的节点都是右节点，还是在上面的旋转原始中，我们要插入节点70。

以祖父节点【左旋】，搭配【变色】
操作步骤如下：
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/red_black_tree_rotate_6.png)

### 红黑树的删除
由于篇幅原因，这里先不详细介绍在删除红黑树的节点后是如何做树的平衡的，但删除节点和插入节点后的本质上是没有什么太大的区别。

接下来我们简单讲下红黑树的java实现TreeMap。

### TreeMap

```java
// TreeMap中使用Entry来描述每个节点
 static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;
        Entry<K,V> right;
        Entry<K,V> parent;
        boolean color = BLACK;
        ...
 }
```

put方法。
```java
    public V put(K key, V value) {
        //先以t保存链表的root节点
        Entry<K,V> t = root;
        //如果t=null,表明是一个空链表，即该TreeMap里没有任何Entry作为root
        if (t == null) {
            compare(key, key); // type (and possibly null) check
            //将新的key-value创建一个Entry，并将该Entry作为root
            root = new Entry<>(key, value, null);
            size = 1;
            //记录修改次数加1
            modCount++;
            return null;
        }
        int cmp;
        Entry<K,V> parent;
        // split comparator and comparable paths
        Comparator<? super K> cpr = comparator;
        //如果比较器cpr不为null，即表明采用定制排序
        if (cpr != null) {
            do {
                //使用parent上次循环后的t所引用的Entry
                parent = t;
                 //将新插入的key和t的key进行比较
                cmp = cpr.compare(key, t.key);
                //如果新插入的key小于t的key，t等于t的左边节点
                if (cmp < 0)
                    t = t.left;
                //如果新插入的key大于t的key，t等于t的右边节点    
                else if (cmp > 0)
                    t = t.right;
                else
                //如果两个key相等，新value覆盖原有的value，并返回原有的value
                    return t.setValue(value);
            } while (t != null);
        }
        else {
            if (key == null)
                throw new NullPointerException();
            @SuppressWarnings("unchecked")
                Comparable<? super K> k = (Comparable<? super K>) key;
            do {
                parent = t;
                cmp = k.compareTo(t.key);
                if (cmp < 0)
                    t = t.left;
                else if (cmp > 0)
                    t = t.right;
                else
                    return t.setValue(value);
            } while (t != null);
        }
        //将新插入的节点作为parent节点的子节点
        Entry<K,V> e = new Entry<>(key, value, parent);
        //如果新插入key小于parent的key,则e作为parent的左子节点
        if (cmp < 0)
            parent.left = e;
        //如果新插入key小于parent的key，则e作为parent的右子节点
        else
            parent.right = e;
        //修复红黑树
        fixAfterInsertion(e);
        size++;
        modCount++;
        return null;
    }
```

```java
//插入节点后修复红黑树
private void fixAfterInsertion(Entry<K,V> x) {
    x.color = RED;

    //直到x节点的父节点不是根，且x的父节点不是红色
    while (x != null && x != root && x.parent.color == RED) {
        //如果x的父节点是其父节点的左子节点
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            //获取x的父节点的兄弟节点
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            //如果x的父节点的兄弟节点是红色
            if (colorOf(y) == RED) {     
                //将x的父节点设置为黑色
                setColor(parentOf(x), BLACK);
                //将x的父节点的兄弟节点设置为黑色
                setColor(y, BLACK);
                //将x的父节点的父节点设为红色
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            }
            //如果x的父节点的兄弟节点是黑色
            else {   
                //如果x是其父节点的右子节点
                if (x == rightOf(parentOf(x))) {
                    //将x的父节点设为x
                    x = parentOf(x);
                    //右旋转
                    rotateLeft(x);
                }
                //把x的父节点设置为黑色
                setColor(parentOf(x), BLACK);
                //把x的父节点父节点设为红色
                setColor(parentOf(parentOf(x)), RED);
                rotateRight(parentOf(parentOf(x)));
            }
        }
        //如果x的父节点是其父节点的右子节点
        else {
            //获取x的父节点的兄弟节点
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            //如果x的父节点的兄弟节点是红色
            if (colorOf(y) == RED) {
                //将x的父节点设置为黑色
                setColor(parentOf(x), BLACK);
                //将x的父节点的兄弟节点设为黑色
                setColor(y, BLACK);
                //将X的父节点的父节点（G）设置红色
                setColor(parentOf(parentOf(x)), RED);
                //将x设为x的父节点的节点
                x = parentOf(parentOf(x));
            }
            //如果x的父节点的兄弟节点是黑色
            else {
                //如果x是其父节点的左子节点
                if (x == leftOf(parentOf(x))) {
                    //将x的父节点设为x
                    x = parentOf(x);
                    //右旋转
                    rotateRight(x);
                }
                //将x的父节点设为黑色
                setColor(parentOf(x), BLACK);
                //把x的父节点的父节点设为红色
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    //将根节点G强制设置为黑色
    root.color = BLACK;
}
```


[What does the time complexity O(log n) actually mean?](https://hackernoon.com/what-does-the-time-complexity-o-log-n-actually-mean-45f94bb5bfbf)

[Java提高篇--TreeMap](https://blog.csdn.net/chenssy/article/details/26668941)

[关于红黑树(R-B tree)原理，看这篇如何](https://www.cnblogs.com/LiaHon/p/11203229.html)

[浅谈算法和数据结构--平衡查找树之红黑树](https://www.cnblogs.com/yangecnu/p/Introduce-Red-Black-Tree.html)

[TreeMap源码解析](https://www.jianshu.com/p/fc5e16b5c674)

[红黑树深入剖析及Java实现](https://tech.meituan.com/2016/12/02/redblack-tree.html)

[深入理解数据库索引原理-B+树](https://cloud.tencent.com/developer/article/1194116)

[平衡二叉树、B树、B+树、B*树 理解其中一种你就都明白了](https://zhuanlan.zhihu.com/p/27700617)