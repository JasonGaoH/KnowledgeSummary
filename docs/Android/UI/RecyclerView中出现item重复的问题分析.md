### 问题背景
前不久我们项目中由用户反馈说遇到笔记重复的问题，而且不只一次遇到类似的反馈。

这种重复笔记总是出现的feed流的中间位置，如下示意图所示：

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/0QP04e.jpg)

这个图画的有点丑，凑合看，意思大概就是这样的。

接下来，我就得追踪下这个问题了，开始时我几乎就一口咬定是接口返回的有问题，由于前几次后端没有日志，好像之前的反馈就那么过去了，直到后面又出现一次重复笔记的问题，这次是公司内部员工出现的，于是后端也通过这个抓到了相应的日志，发现返回的笔记的确没有重复的，这下跑不掉了，就是前端的问题。

### 问题排查
于是，我又重新梳理了下代码流程，发现有一处比较有嫌疑：

```java
        ...
        if (...) {
            mItems[0] = noteItem
        } else {
            mItems.add(0, noteItem)
        }
        mAdapter.items = mItems
        mAdapter.notifyItemChanged(0)
        ...
```

鉴于是公司项目，我就省略掉业务逻辑了，这里的代码按照开发者的意图是当RecyclerView第一个item如果已经是noteItem这种类型的时候，我们就将这个位置item替换成最新的，如果这个位置的item不是noteItem这个数据的话，我们需要手动把它添加到第一个位置去，到这里实际上都没有什么问题。

但是，当我看到``mAdapter.notifyItemChanged(0)``这个方法，直觉告诉我这里好像有点问题，当上面的逻辑走到else这里的时候，会往list里add一个新的item，但是这时候调用的刷新方法却是notifyItemChanged(0)。

这个notifyItemChanged明显是刷新某个item的方法，即当这个item里的数据有变化时，调用这个方法去刷新这个item区域的UI，但是如果我们在adapter中add了一个新的item，再调用这个方法明显是不行的，这里是导致重复的原因嘛，我其实也不太确定。

### 问题复现
于是我写了一个Demo试了下。

```java
class RecyclerViewActivity : AppCompatActivity() {

    private val mDataList = ArrayList<Any>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_recycler_view)

        val layoutManager = LinearLayoutManager(this)
        layoutManager.orientation = RecyclerView.VERTICAL
        recyclerView.layoutManager = layoutManager

        val customAdapter =  CustomAdapter(mDataList)

        recyclerView.adapter = customAdapter

        for (i in 0..5) {
            mDataList.add("text: $i")
        }

        recyclerView.adapter?.notifyDataSetChanged()
    }

    //刷新方法
    fun refresh(view: View) {
        mDataList.add(0, "add item")
        recyclerView.adapter?.notifyItemChanged(0)
    }

}
```
构造一个普通的feed列表，每次点击刷新按钮，就会调用刷新方法，调用刷新方法的时候往index为0的位置再add一个item，然后再调用notifyItemChanged(0)方法，上下滑动后，发现数据是重复了。

下面放个gif图展示下效果。

![](https://raw.githubusercontent.com/JasonGaoH/Images/master/gifhome_480x1040_4s.gif)

从gif图中可以看到，点击刷新按钮，添加了”add item“,往下滑动后，出现了两个”text：5“的item，这个就是重复的item。

### 问题原因
我们看到notifyItemChanged的文档说明：
![](https://raw.githubusercontent.com/JasonGaoH/Images/master/NotifyItemChanged.jpg)

> notifyItemChanged(int position)
This is an item change event, not a structural change event.

RecyclerView中有两种不同数据改变事件，一种叫(item changes)项目改变，另一种叫(structual changes)结构改变。项目改变指的是某个单个item的数据发生变化，这个时候没有位置的改变。而structual changes则是有位置的变化发生，主要是数据源的变化会导致RecyclerView item位置发生变化。

我们这个问题是我们往数据源前面加了一个item，这个时候应该需要调用具有structual changes 的方法来刷新，而不是采用notifyItemChanged来刷新，因为notifyItemChanged是一个item changes 的方法。