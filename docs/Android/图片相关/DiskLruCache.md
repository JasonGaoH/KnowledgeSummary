# DiskLruCache

基本原理：

将图片保存在磁盘中，维护一个journal文件用来记录用户对文件的4种操作记录。初始化和任何一种行为都会刷新该journal文件，并且根据该journal文件，将缓存文件记录到LinkedHashMap。和LruCache一样，使用了LinkedHashMap，作为LRU算法的数据基础。

操作行为有4种：

* DIRTY: 含义如其名，意味着一条脏数据。当调用edit()时，就会向journal文件写入一条DIRTY记录，表示我们正准备写入一条缓存数据，但不知结果如何。

* CLEAN: 表示数据是干净的，可以使用了。当调用commit()，会向journal文件写入一条CLEAN记录，表示缓冲数据可用，可以READ。

* READ: 当调用get()，会向journal文件写入一条READ记录，表示读取一条缓存数据。

* REMOVE: 当调用remove()，会向journal文件写入一条REMOVE记录，表示删除一条缓存数据。

journal文件：

```
libcore.io.DiskLruCache		// DiskLruCache的标记，固定常量
1							// DiskLruCache版本号
100							// 应用程序版本号
2							// open()方法中传入的，通常情况下为1

CLEAN 3400330d1dfc7f3f7f4b8d4d803dfcf6 832 21054
DIRTY 335c4c6028171cfddfbaae1a9c313c52
CLEAN 335c4c6028171cfddfbaae1a9c313c52 3934 2342
REMOVE 335c4c6028171cfddfbaae1a9c313c52
DIRTY 1ab96a171faeeee38496d8b330771a7a
CLEAN 1ab96a171faeeee38496d8b330771a7a 1600 234
READ 335c4c6028171cfddfbaae1a9c313c52
READ 3400330d1dfc7f3f7f4b8d4d803dfcf6
     
```



问：

1. 即便get()时，从LinkedHashMap中获得了缓存文件的名字(即hash后的key)，从文件列表中拿到文件不还是要逐个遍历么，定位名字是否匹配的文件么？

A: 读文件时，根据key，从LinkedHashMap中获取Entry，判断Entry是否存在，且readable。满足条件后，根据DiskLruCache内部保存的存放缓存文件的directory，结合key，对应缓存文件的绝对路径就有了，即directory + key。所以获取对应缓存文件也就简单了，即

```
new File(directory, key)
```