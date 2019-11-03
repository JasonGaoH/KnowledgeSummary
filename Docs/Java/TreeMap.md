# 红黑树

在讲红黑树数之前，需要需要先熟悉以下几个概念：
## 二叉树
二叉树指的是每个节点最多只能有两个字数的有序树。通常左边的子树称为``左子树``,右边的子树称为``右子树``。这里说的有序树强调的是二叉树的左子树和右子树的次序不能随意颠倒。



## 排序二叉树

排序二叉树

平衡二叉树

红黑树

时间负责度为O(log n),那么这个O(log n)到底是什么意思呢，举个例子，你肯定知道二分查找，二分查找算法的时间复杂度就是O(log n)。

因为一棵由n个结点随机构造的二叉查找树的高度为lgn，所以顺理成章，二叉查找树的一般操作的执行时间为O(lgn)。但二叉查找树若退化成了一棵具有n个结点的线性链后，则这些操作最坏情况运行时间为O(n)。

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