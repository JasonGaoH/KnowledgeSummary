为什么说二分查找的时间复杂度是O(log n)

这个问题本质上一个数学题。

先举个例子。
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search.png)

给定一个长度为16的有序数组，如上图所示。

现在我们要查找在这个数组上查找13这个数。

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_1.jpeg)

首先我们将(length / 2 )拿到中间的元素,这个作为一个基准

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_2.jpeg)

我们发现13是小于16的，所以需要对左边的数组再一半对比。

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_3.jpeg)

重复再尝试去查找

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_4.jpeg)

最后拿到13这个数。

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_5.jpeg)

这里从长度为16的数组里面查找元素，最多需要查找4次才能查找到。

简单可以得到下面的公式。

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_6.png)


假设我们有n个有序元素，对n个元素进行查找的时候，时间复杂度为k，这样就可以有下面的公式。

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_7.png)

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_8.png)

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_9.png)

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_10.png)

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/binary_search_11.png)