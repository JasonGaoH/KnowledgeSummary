

[RxJava 是如何实现线程切换的（上）](https://www.jianshu.com/p/3dd582bb10cc)

[RxJava 是如何实现线程切换的（下）](https://www.jianshu.com/p/88aa273d37be)

### RxJava变换操作符map,flatMap,concatMap,buffer

- map：【数据类型转换】将被观察者发送的事件转换为另一种类型的事件
- flatMap：【化解循环嵌套和接口嵌套】将被观察者发送的事件序列进行拆分 & 转换 后合并成一个新的事件序列，最后再进行发送
- concatMap：【有序】与 flatMap 的 区别在于，拆分 & 重新合并生成的事件序列 的顺序与被观察者旧序列生产的顺序一致
- flatMapIterable：相当于对 flatMap 的数据进行了二次扁平化
- buffer：定期从被观察者发送的事件中获取一定数量的事件并放到缓存区中，然后把这些数据集合打包发射

> The map operator creates a new Observable that emits items that have been converted from items emitted by the source Observable. The map operator would allow us, for example, to turn an Observable that emits Storys into an Observable that emits the titles of those Storys.

[link](https://www.jianshu.com/p/c820afafd94b)