# volatile关键字


volatile是一个非常重要的关键字，虽然看起来很简单，但是想要彻底弄清楚volatile的来龙去脉还是需要具备Java内存模型、CPU缓存模型等知识的，我们尝试一步步地来深入地分析下volatile关键字工作原理。

#### volatile关键字使用
首先我们还是从代码来入手：
```

import java.util.concurrent.TimeUnit;

public class VolatileFoo {
	
	//init_value的最大值
	final static int MAX = 5;
	//init_value的初始值
	static int  init_value = 0;

	public static void main(String[] args) {
		//启动一个Reader线程，当发现local_value和init_value不同时，
		//则输出init_value被修改的信息
		new Thread("Readder"){
			@Override
			public void run() {
				int localValue = init_value;
				while(localValue < MAX) {
					if(init_value != localValue) {
						System.out.println("this init_value is updated to " + init_value);
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
					System.out.println("this init_value will be changed to " + ++localValue);
					//对local_value重新赋值
					init_value = localValue;
					try {
                        //短暂休眠，目的是为了使Reader线程能够来得及输出变化内容
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
上面的程序分别启动了两个线程，一个线程负责对变量进行修改，一个线程负责对变量进行输出，该变量是属于共享资源（数据），在多线程操作的情况下，很有可能会引起数据不一致等线程安全的问题。

运行上面的程序，输出如下：
```
this init_value will be changed to 1
this init_value will be changed to 2
this init_value will be changed to 3
this init_value will be changed to 4
this init_value will be changed to 5
```
从输出信息我们发现，Reader线程没有感知到init_value的变化，我们期望的是在Updater进程更新init_value的值之后，Reader进程能够打印出变化的init_value的值，但结果并不是我们期望的那样。

接下来我们做个小小的改动：
```
static volatile int  init_value = 0;

```
我们使用volatile来修饰这个init_value的值。再次运行修改后的程序，输出结果如下：
```
this init_value will be changed to 1
this init_value is updated to 1
this init_value will be changed to 2
this init_value is updated to 2
this init_value will be changed to 3
this init_value is updated to 3
this init_value will be changed to 4
this init_value is updated to 4
this init_value will be changed to 5
this init_value is updated to 5
```
这个时候Reader线程就能够感受到init_value的值的变化了，并且在条件不满足时就退出了运行。

那么为什么加了个volatile就正常了呢， volatile关键字的作用到底是什么呢？

接下来我们一步一步来拆解问题。

#### CPU缓存模型
要想对volatile有比较深刻的理解，首先我们需要对CPU的缓存模型有一定的认识。

在计算机中，所有的运算操作都是由CPU的寄存器来完成的，CPU指令的执行过程需要设计数据的读取和写入操作，CPU所能访问的所有数据只能是计算机的贮存（通常是指RAM），虽然CPU的发现频率不断得到提升，但受制于制造工艺以及成本的限制，计算机的内存反倒在访问速度上没有多大的突破，因此CPU的处理速度和内存的访问速度之间的差距越拉越大，通常这种差距可以达到上千倍，极端情况下甚至会在上万倍以上。

由于两边速度严重的不对等，通过传统FSB直接内存的访问方式会导致CPU资源受到极大的限制，降低CPU整体的吞吐量，于是就有了CPU和主内存直接增加缓存的设计，现在缓存数量都可以增加到3级了，最靠近CPU的缓存为L1,ranhou依次是L2,L3和主内存，CPU缓存模型图如下所示：

![CPU缓存模型图](../img/cpu_cache.png)

Cache的出现是为了解决CPU直接访问内存效率低下的问题，程序在运行的过程中，会将运算所需要的数据从主内存复制一份到CPU Cache中，这样CPU计算时就可以直接怼CPU Cache中的数据进行读取和写入，当运算结束之后，再将CPU Cache中最新的数据刷新到主内存当中，CPU通过直接访问Cache的方式提到直接访问主内存的方式极大地提高了CPU的吞吐能力，有个CPU Cache之后，整体的CPU和主内存之间的交互的架构大致如下图所示：

![](../img/cpu_cache_framework.png)

#### Java内存模型

Java内存模型指定了Java虚拟机如何与计算机的内存进行工作，理解Java内存模型对于立即并发的原理是非常重要的。

Java的内存迷行决定了一个线程对共享变量的写入合适对线程可一件，Java内存迷行定义了线程和主内存之间的抽象关系，具体如下。


[link](https://www.cnblogs.com/dolphin0520/p/3920373.html)