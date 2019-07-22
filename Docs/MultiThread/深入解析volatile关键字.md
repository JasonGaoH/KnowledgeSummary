# volatile关键字

volatile是一个非常重要的关键字，虽然看起来很简单，但是想要彻底弄清楚volatile的来龙去脉还是需要具备Java内存模型、CPU缓存模型等知识的，我们尝试一步步地来深入地分析下volatile关键字工作原理。

首先我们还是从代码来入手：
```
import java.util.concurrent.TimeUnit;

public class VolatileFoo {
	
	//init_value的最大值
	final static int MAX = 5;
	//init_value的初始值
	static int init_value = 0;

	public static void main(String[] args) {
		//启动一个Reader线程，当发现local_value和init_value不同时，
		//则输出init_value被修改的信息
		new Thread("Readder"){

			@Override
			public void run() {
				int localValue = init_value;
				while(init_value < MAX) {
					if(init_value != localValue) {
						System.out.println("this init_value is updated to" + init_value);
						//对local_value重新赋值
						localValue = init_value;
					}
				}
			}
			
		}.start();
		
		new Thread("Updater") {
			public void run() {
				int localValue = init_value;
				while(localValue < MAX) {
					System.out.println("this init_value will be changed to" + localValue);
					//对local_value重新赋值
					init_value = localValue;
					try {
						TimeUnit.SECONDS.sleep(2);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			};
		}.start();
	}
}

```

[link](https://www.cnblogs.com/dolphin0520/p/3920373.html)