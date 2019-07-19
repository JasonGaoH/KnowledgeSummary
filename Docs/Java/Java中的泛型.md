泛型，主要是为了参数化类型，或者说可以将类型当做参数传递给一个类或者方法，泛型最主要功能是因为它是一种约束。

什么叫参数化类型呢？

那么为什么说泛型是一种约束呢？

```
public class Cache {
	Object value;
	public Object getValue() {
		return value;
	}
	public void setValue(Object value) {
		this.value = value;
	}
}
```

假设 Cache 能够存取任何类型的值，于是，我们可以这样使用它。

```
Cache cache = new Cache();
cache.setValue(134);
int value = (int) cache.getValue();
cache.setValue("hello");
String value1 = (String) cache.getValue();
```

使用的方法也很简单，只要我们做正确的强制转换就好了。

但是，泛型却给我们带来了不一样的编程体验。

```
public class Cache<T> {
	T value;

	public Object getValue() {
		return value;
	}

	public void setValue(T value) {
		this.value = value;
	}
}
```
这就是泛型，它将 value 这个属性的类型也参数化了，这就是所谓的参数化类型。再看它的使用方法。
```
Cache<String> cache1 = new Cache<String>();
cache1.setValue("123");
String value2 = cache1.getValue();
		
Cache<Integer> cache2 = new Cache<Integer>();
cache2.setValue(456);
int value3 = cache2.getValue();

```
这样来看，
- 在与普通的 Object 代替一切类型这样简单粗暴而言，泛型使得数据的类别可以像参数一样由外部传递进来。它提供了一种扩展能力。它更符合面向抽象开发的软件编程宗旨。
- 当具体的类型确定后，泛型又提供了一种类型检测的机制，只有相匹配的数据才能正常的赋值，否则编译器就不通过。所以说，它是一种类型安全检测机制，一定程度上提高了软件的安全性防止出现低级的失误。
- 泛型提高了程序代码的可读性，不必要等到运行的时候才去强制转换，在定义或者实例化阶段，因为 Cache<String>这个类型显化的效果，程序员能够一目了然猜测出代码要操作的数据类型。