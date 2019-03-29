写这篇文章的目的，一个是为了了解ARouter的原理，另外一个就是看看ARouter使用了哪些有意思的技术，哪些技术可以应用到我们自己的项目中。

ARouter本质上是个路由框架，常用在组件化架构中，满足解耦的组件之间的通信、页面路由、数据传递等。它所支持的Router类型从下面这个类就可以看出：

```
enum RouteType {
    ACTIVITY(0, "android.app.Activity"),
    SERVICE(1, "android.app.Service"),
    PROVIDER(2, "com.alibaba.android.arouter.facade.template.IProvider"),
    CONTENT_PROVIDER(-1, "android.app.ContentProvider"),
    BOARDCAST(-1, ""),
    METHOD(-1, ""),
    FRAGMENT(-1, "android.app.Fragment"),
    UNKNOWN(-1, "Unknown route type");
}
```

### 页面路由

大家使用ARouter，最常用的就是页面路由的功能。研究页面如何路由时，先提几个问题，可以更有针对性的看代码：

1. 启动页面时用的是字符串，字符串和对应页面有什么关系？
2. 页面的路由表是什么东西？
3. 页面路由表怎么初始化的？
4. 如何拦截跳转过程，处理登陆、埋点等逻辑？
6. 为什么加入分组(group)的概念，有什么好处？
7. 启动Activity可以通过startActivityForResult获取回调，那启动一个View、Fragment或Service，如何回调？

衍生问题：

1. 如何调试自定义注解处理器里的代码？
2. 如何跨进程启动页面？
3. 如何在项目实战中，优雅的使用页面URL，以及传参？

##### 1.启动页面时用的是字符串，字符串和对应页面是什么关系？

每个页面都用自定义注解Route进行注解，其中启动页面的字符串就是注解的参数path。也就是利用注解给页面起了个别名。启动页面时，因为模块间解耦的关系，不能直接显示启动Activity，所以ARouter使用字符串别名启动Activity。

##### 2.页面的路由表是什么东西？

页面路由表指一个Map，这个Map建立了页面的字符串别名和Activity的Class信息。启动页面时，根据传入的页面别名，从Map里拿到页面Class，再startActivity启动目标页面。

##### 3.如何初始化页面路由表？

页面路由表在项目中就是指Warehouse#routes这个Map。
ARouter和其他页面路由库（比如ActivityRouter）一样使用了APT的技术。编译期，根据页面的注解信息，建立页面URL（即Route path中的参数）和页面的Class信息的映射关系。
相关实现都在自定义的AbstractProcessor类里，即`RouteProcessor`。
APT生成的类，可以参考下面的`ARouter$$Group$$test`。

```
public class ARouter$$Group$$test implements IRouteGroup {
  @Override
  public void loadInto(Map<String, RouteMeta> atlas) {
    atlas.put("/test/activity1", RouteMeta.build(RouteType.ACTIVITY, Test1Activity.class));
  }
}

```
即初始化了一个映射页面URL和页面信息的Map。

路由表初始化入口：

```
class LogisticsCenter {
	public synchronized static void completion(Postcard postcard) {
		// 调用编译期生成的类，初始化页面路由表
		iGroupInstance.loadInto(Warehouse.routes);
	}
}

class Warehouse {
	// 页面路由表
	// Key为Activity的注解path，比如Test1Activity的key就是"/test/activity1"
	// Value是存储Activity信息的Module，比如Test1Activity.class
	static Map<String, RouteMeta> routes = new HashMap<>();
}
```


路由其他页面时，都会执行ARouter#navigation()，看下实现。

页面跳转：

```
class _ARouter {

private Object _navigation(final Context context, final Postcard postcard, final int requestCode, final NavigationCallback callback) {
	...

    switch (postcard.getType()) {
        case ACTIVITY:
            // Build intent
            final Intent intent = new Intent(currentContext, postcard.getDestination());
            intent.putExtras(postcard.getExtras());

            // Set flags.
            ...
            
            // Set Actions
            ...

            // Navigation in main looper.
            new Handler(Looper.getMainLooper()).post(new Runnable() {
                @Override
                public void run() {
                    if (requestCode > 0) {  // Need start for result
                        ActivityCompat.startActivityForResult((Activity) currentContext, intent, requestCode, postcard.getOptionsBundle());
                    } else {
                        ActivityCompat.startActivity(currentContext, intent, postcard.getOptionsBundle());
                    }

                    ...

                    if (null != callback) { // Navigation over.
                        callback.onArrival(postcard);
                    }
                }
            });
            break;
        case PROVIDER:
            return postcard.getProvider();
        case BOARDCAST:
        case CONTENT_PROVIDER:
        case FRAGMENT:
            Class fragmentMeta = postcard.getDestination();
            try {
                Object instance = fragmentMeta.getConstructor().newInstance();
                if (instance instanceof Fragment) {
                    ((Fragment) instance).setArguments(postcard.getExtras());
                } else if (instance instanceof android.support.v4.app.Fragment) {
                    ((android.support.v4.app.Fragment) instance).setArguments(postcard.getExtras());
                }

                return instance;
            } catch (Exception ex) {
                ...
            }
        case METHOD:
        case SERVICE:
        default:
            return null;
    }

    return null;
}

}
```

##### 4. 如何优化数据传递？

这个问题什么意思呢？实际项目中使用ARouter做页面路由，比如demo中启动kotlin测试页面：

```
ARouter.getInstance()
        .build("/kotlin/test")
        .withString("name", "老王")
        .withInt("age", 23)
        .navigation();
```
这种做法会有两个问题：

1）会出现较多hardcode，包括页面path及参数，容易犯拼写错误<br>
2）启动一个页面时，必须预先了解目标页面的path，及支持哪些参数，然后再去启动

当然，有同学会说，path及参数这些字符串，可以用常量事先声明在基础组件层，即各个业务Module都依赖的公共层。这种做法虽然可行，但是仍避免不了大量的常量出现。同样，问题2也得不到解决。

因为我们在使用过程中遇到上述问题，于是对ARouter做了修改，提升开发效率同时还能避免犯低级错误。

改动后，如果启动一个目标页面时，大致的做法是下面这样的：

```
ARouter.getInstance()
        .build(new KotlinPage("老王", 23))
        .navigation();
```

要是使用Kotlin，结合方法参数的可选特性，及IDE的提示，是不是开发效率更高，简直闭着眼睛就可以启动目标页面。

这种改进的好处有两点：
1.不需要在公共层声明path和param这些常量，只需要声明一个数据类即可
2.启动目标页面时，不需要过多关心目标页面的实现，只需要和对应数据类打交道即可，做到目标页面一定程度开闭原则上的保护。

实现方式简单介绍下：
。。。

具体代码可以参考：
。。。

### 模块间通信

同样，研究代码之前提几个问题：

1. 怎么做到跨模块API调用？
2. 如何通过控制反转来做组件解耦？

##### Q1.怎么做到跨模块API调用？
模块对外提供支撑的接口类，需要实现IProvider接口（比如demo中的HelloService）。接口实现类（HelloServiceImpl）使用Route注解。
其实原理和页面路由一样，就是在编译时，创建模块API接口的路由表。使用时，通过URL，在接口路由表中映射到模块接口实现类，进行跨模块API调用。

以demo中的HelloService为例，

```
接口类：
public interface HelloService extends IProvider {
    void sayHello(String name);
}

实现类，使用Route注解，方便APT创建接口路由表：
@Route(path = "/service/hello")
public class HelloServiceImpl implements HelloService {
    Context mContext;

    @Override
    public void sayHello(String name) {
        ...
    }

    /**
     * Do your init work in this method, it well be call when processor has been load.
     *
     * @param context ctx
     */
    @Override
    public void init(Context context) {
        mContext = context;
        ...
    }
}

APT创建的文件，初始化接口路由表：
public class ARouter$$Group$$service implements IRouteGroup {
  @Override
  public void loadInto(Map<String, RouteMeta> atlas) {
    atlas.put("/service/hello", RouteMeta.build(RouteType.PROVIDER, HelloServiceImpl.class));
  }
}


其他组件通过接口调用Hello组件：

HelloService helloService = (HelloService) ARouter.getInstance().build("/service/hello").navigation();
helloService.sayHello("mike");

```
模块接口类可以沉淀到公共层，实现类还在自己的Module中。这样可以做到接口和实现分离，只需通过接口提供服务，做到模块间解耦。

不过这种做法，接口和组件间通信的数据结构都要下沉到公共层，导致公共层中心化问题越来越凸显，有没有一种去中心化的通信方式呢？

Q2.如何通过控制反转来做组件解耦？
答案在下面一节。

### 如何实现依赖注入
在回答这个问题之前，对依赖注入和控制反转不熟悉的同学可以看看下面这边文章。<br>
[依赖注入和控制反转](https://hningoba.github.io/2018/08/12/%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5%E5%92%8C%E6%8E%A7%E5%88%B6%E5%8F%8D%E8%BD%AC/#more)

wiki中也对如何通过依赖注入进行解耦给了代码实例：

```
public class Test {
    @Autowired
    HelloService helloService;

    @Autowired(name = "/service/hello")
    HelloService helloService2;

    HelloService helloService3;

    HelloService helloService4;

    public Test() {
		ARouter.getInstance().inject(this);
    }

    public void testService() {
	 // 1. (推荐)使用依赖注入的方式发现服务,通过注解标注字段,即可使用，无需主动获取
	 // Autowired注解中标注name之后，将会使用byName的方式注入对应的字段，不设置name属性，会默认使用byType的方式发现服务(当同一接口有多个实现的时候，必须使用byName的方式发现服务)
	helloService.sayHello("Vergil");
	helloService2.sayHello("Vergil");

	// 2. 使用依赖查找的方式发现服务，主动去发现服务并使用，下面两种方式分别是byName和byType
	helloService3 = ARouter.getInstance().navigation(HelloService.class);
	helloService4 = (HelloService) ARouter.getInstance().build("/service/hello").navigation();
	helloService3.sayHello("Vergil");
	helloService4.sayHello("Vergil");
    }
}
```

关键的下面这行代码：

```
ARouter.getInstance().inject(this);
```
这行代码的工作内容就是在把Test的几个依赖（helloService ~ helloService4）在依赖注入器中初始化，然后注入给Test，从而实现解耦。且Test对HelloService的依赖也都是基于接口，更符合依赖反转原则。

可以结合《依赖注入和控制反转》和上面的实例代码体会下依赖注入和依赖查找的区别。即依赖注入比较被动，跟依赖注入器打声招呼后，所有工作都交给注入器了。依赖查找更加主动，在需要的时候通过调用框架提供的方法来获取对象。

看下inject()工作流程：

```
class Test1Activity {
	@Autowired
	String url;

	@Autowired
	HelloService helloService;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		...
		// 开始注入
		ARouter.getInstance().inject(this);
	
		// 使用服务
		helloService.sayHello("Hello moto.");
		...
	}
}
```

```
class _ARouter {
	// 开始注入
	static void inject(Object thiz) {
		// 获取AutowiredService的实现，过程和页面路由表一样，用APT做解耦策略
		AutowiredService autowiredService = ((AutowiredService) ARouter.getInstance().build("/arouter/service/autowired").navigation());
	    if (null != autowiredService) {
			// 开始装配
	        autowiredService.autowire(thiz);
	    }
	}
}

@Route(path = "/arouter/service/autowired")
public class AutowiredServiceImpl implements AutowiredService {
	@Override
	public void autowire(Object instance) {
		...
		ISyringe autowiredHelper = classCache.get(className);
		// 这里的autowiredHelper就是下面的Test1Activity$$ARouter$$Autowired
		autowiredHelper.inject(instance);
		...
	}
}
```

APT创建的文件：

```
/**
 * DO NOT EDIT THIS FILE!!! IT WAS GENERATED BY AROUTER. */
public class Test1Activity$$ARouter$$Autowired implements ISyringe {
  private SerializationService serializationService;

  @Override
  public void inject(Object target) {
		serializationService = ARouter.getInstance().navigation(SerializationService.class);
		Test1Activity substitute = (Test1Activity)target;
		// field 赋值
		substitute.url = substitute.getIntent().getStringExtra("url");
		// 依赖注入
		substitute.helloService = ARouter.getInstance().navigation(HelloService.class);
	}
}
```

所以，核心点是在于``Test1Activity$$ARouter$$Autowired``，这也是为什么``Test1Activity``中的helloService并没有看到初始化就能直接使用。聪明的你是不是发现这块做法有点[butterknife](https://github.com/JakeWharton/butterknife)的味道。



### 支持InstantRun和MultiDex


### 支持拦截器，面向切面编程


### Gradle插件实现路由表的自动加载

初始化ARouter时，会执行下面的方法。

```
class LogisticsCenter {
	private static void loadRouterMap() {
	    registerByPlugin = false;
	    //auto generate register code by gradle plugin: arouter-auto-register
	    // looks like below:
	    // registerRouteRoot(new ARouter..Root..modulejava());
	    // registerRouteRoot(new ARouter..Root..modulekotlin());
	}
}
```

实际项目中我们可以写gradle插件做很多辅助工作，比如ARouter用来生产代码初始化路由表，也可以代码检查、代码修改、以及上传maven等各种task。

