# ActivityRouter

##### 1. @Router的作用
	
编译时，注解处理器会扫描所有Java源文件。最终会执行自定义注解处理器`RouterProcessor`中的几个关键方法
	 
```
@AutoService(Processor.class)
public class RouterProcessor extends AbstractProcessor {

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {}
    
    @Override
    public Set<String> getSupportedAnnotationTypes() {}
    
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {}
    
}

```
在process()方法中，通过Element、RoundEnvironment等获取到被注解的类的信息，包括类名、注解内容。根据这些信息，借助JavaPoet生成Java文件，此Java文件就是连接注解和使用方的纽带。

PS: 编译时注解处理器不熟悉的，可以参考[Java注解处理器](https://www.race604.com/annotation-processing/)


##### 2. APT工作流程
编译时，系统会调用RouterProcessor#process()，在此方法中做下面相关处理：

1. 获取所有被Router注解的Element，Element是程序的元素，可以是包、类、方法
2. 创建map方法
3. 遍历Element，获取@Routers.value和被注解的Activity(或者是方法)
4. 将Routers注解中的value(代码中的format)和页面(className)绑定，放到数据类Mapping中
5. 程序运行就会初始化Router，执行RouterInit.init()，进而执行RouterMapping_app.map()(这一步是注解处理器添加的初始化代码)，即处理器生成的代码文件。此刻，所有流程串了起来。

```
RouterProcessor.java

private boolean handleRouter(String genClassName, RoundEnvironment roundEnv) {

	// 获取被Router注解的Element
	Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(Router.class);

	// 创建map方法
        MethodSpec.Builder mapMethod = MethodSpec.methodBuilder("map")
                .addModifiers(Modifier.PUBLIC, Modifier.FINAL, Modifier.STATIC)
                .addStatement("java.util.Map<String,String> transfer = null")
                .addStatement("com.github.mzule.activityrouter.router.ExtraTypes extraTypes")
                .addCode("\n");
                
	
    for (String format : router.value()) {
    	
    	// 获取被注解的类名或方法名
        ClassName className;
        Name methodName = null;
        if (element.getKind() == ElementKind.CLASS) {
            className = ClassName.get((TypeElement) element);
        } else if (element.getKind() == ElementKind.METHOD) {
            className = ClassName.get((TypeElement) element.getEnclosingElement());
            methodName = element.getSimpleName();
        } else {
            throw new IllegalArgumentException("unknow type");
        }
        
        ... // 安全检查
        
        // 关键!!! 将@Routers注解中的value(即代码中的format)和页面(className)绑定
        if (element.getKind() == ElementKind.CLASS) {
     mapMethod.addStatement("com.github.mzule.activityrouter.router.Routers.map($S, $T.class, null, extraTypes)", format, className);
        } else {
            mapMethod.addStatement("com.github.mzule.activityrouter.router.Routers.map($S, null, " +
                    "new MethodInvoker() {\n" +
                    "   public void invoke(android.content.Context context, android.os.Bundle bundle, int requestCode) {\n" +
                    "       $T.$N(context, bundle, requestCode);\n" +
                    "   }\n" +
                    "}, " +
                    "extraTypes)", format, className, methodName);
        }
    }
	
}
```

注解处理器生成的文件：

```
RouterMapping_app.java

public final class RouterMapping_app {
  public static final void map() {
    java.util.Map<String,String> transfer = null;
    com.github.mzule.activityrouter.router.ExtraTypes extraTypes;

    transfer = null;
    extraTypes = new com.github.mzule.activityrouter.router.ExtraTypes();
    extraTypes.setTransfer(transfer);
    
    // 最重要！！！注解和页面的绑定
    com.github.mzule.activityrouter.router.Routers.map("home/:homeName", HomeActivity.class, null, extraTypes);

  }
}
```

其中，Routers.map()就是将@Routers注解的value、param和对应的Activity放到mappings中。当使用Routers.open(context, url)启动一个Activity时，会遍历mappings，找到和url对应的Mapping，拿到Mapping中事先存储的目标Activity，再使用Intent启动Activity。

```
Routers.java

static void map(String format, Class<? extends Activity> activity, MethodInvoker method, ExtraTypes extraTypes) {
        mappings.add(new Mapping(format, activity, method, extraTypes));
    }
    
private static boolean doOpen(Context context, Uri uri, Bundle extras, int requestCode) {
        initIfNeed();
        Path path = Path.create(uri);

        for (Mapping mapping : mappings) {

			//找到和目标url匹配的Mapping
            if (mapping.match(path)) {

				// 解析Url中的参数放到Bundle中
                Bundle bundle = mapping.parseExtras(uri);
                appendExtras(uri, extras, bundle);

				// 如果被注解的是方法，直接通过反射调用方法
                if (mapping.getActivity() == null) {
                    mapping.getMethod().invoke(context, bundle, requestCode);
                    return true;
                }
                
				// 如果被注解的是Activity，则通过Intent启动
                Intent intent = new Intent(context, mapping.getActivity());
                intent.putExtras(bundle);
                if (!(context instanceof Activity)) {
                    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                }
                
                ...
                
                context.startActivity(intent);
                
                return true;
            }
        }
        return false;
    }
```

```
Mapping.java

public class Mapping {

    private final String format;
    private final Class<? extends Activity> activity;
    private final MethodInvoker method;
    private final ExtraTypes extraTypes;
    private Path formatPath;

    public Mapping(String format, Class<? extends Activity> activity, MethodInvoker method, ExtraTypes extraTypes) {
        if (format == null) {
            throw new NullPointerException("format can not be null");
        }
        this.format = format;
        this.activity = activity;
        this.method = method;
        this.extraTypes = extraTypes;
        
        if (format.toLowerCase().startsWith("http://") || format.toLowerCase().startsWith("https://")) {
            this.formatPath = Path.create(Uri.parse(format));
        } else {
            this.formatPath = Path.create(Uri.parse("helper://".concat(format)));
        }
    }
}
```


应用示例：

```
Routers.open(LaunchActivity.this, "router://home/somebody?id=123")

可打开如下页面

@Router(value = "home/:homeName", stringParams = "source")
public class HomeActivity extends DumpExtrasActivity {

}

```

##### 问题 3. 有没有好办法可以取代Routers.open()使用字段串拼接Url的方式




