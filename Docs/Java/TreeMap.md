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
其实红黑树和上面的平衡二叉树类似，本质上都是为了解决排序二叉树在极端情况下退化成链表导致检索效率大大降低的问题。

红黑树虽然本质上是一棵二叉查找树，但它在二叉查找树的基础上增加了着色和相关的性质使得红黑树相对平衡，从而保证了红黑树的查找、插入、删除的时间复杂度最坏为O(log n)

红黑树的特性是为了让红黑树保持相对平衡，从而提升树的检索效率。

红黑树的时间复杂度为什么是时间负责度为O(log n)？

红黑树为什么会有红色节点和黑色节点，这么设计是为了什么？

[What does the time complexity O(log n) actually mean?](https://hackernoon.com/what-does-the-time-complexity-o-log-n-actually-mean-45f94bb5bfbf)

[Java提高篇--TreeMap](https://blog.csdn.net/chenssy/article/details/26668941)

[浅谈算法和数据结构--平衡查找树之红黑树](https://www.cnblogs.com/yangecnu/p/Introduce-Red-Black-Tree.html)

[TreeMap源码解析](https://www.jianshu.com/p/fc5e16b5c674)

[红黑树深入剖析及Java实现](https://tech.meituan.com/2016/12/02/redblack-tree.html)

[深入理解数据库索引原理-B+树](https://cloud.tencent.com/developer/article/1194116)

[平衡二叉树、B树、B+树、B*树 理解其中一种你就都明白了](https://zhuanlan.zhihu.com/p/27700617)