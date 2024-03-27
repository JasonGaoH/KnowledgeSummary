# Retrofit

通过动态代理，将@Get/@Post等IO相关的注解方法，代理成OkHttpCall，内部实现网络请求。

Retrofit CreateApi实现原理

Retrofit 如何实现文件（或图片上传）接口是如何定义的
>大意要回答multipart/form-data 文件上传表单中
它会将表单的数据处理为一条消息，以标签为单元，用分隔符分开。既可以上传键值对，也可以上传文件。当上传的字段是文件时，会有Content-Type来表名文件类型；content-disposition，用来说明字段的一些信息；
由于有boundary隔离，所以multipart/form-data既可以上传文件，也可以上传键值对，它采用了键值对的方式，所以可以上传多个文件。


Retrofit+Okhttp+Rxjava在华为的好多手机会OOM是由线程数溢出引起如何解决

在Android7.0及以上的华为手机（EmotionUI_5.0及以上）的手机产生OOM，这些手机的线程数限制都很小(应该是华为rom特意修改的limits)，每个进程只允许最大同时开500个线程。解决方法尽量不要重复创建retrofit 和Okhttp的实例，手动设置okhttp 最大空闲连接数量，并减少keep-alive时间
如
ConnectionPool pool = new ConnectionPool(5, 10000, TimeUnit.MILLISECONDS);

OkHttpClient client = new OkHttpClient.Builder()
.connectionPool(pool)
.build();

RxJava+Retrofit进行网络请求; Header里如何添加通用参数， Response里如何解析，只返回业务层result，不需要每个Bean都有code ?

问题一：
Header里如何添加通用参数，如果是自己封装的代码，可以在封装Request中可以解决，也可以增加拦截器，通过拦截器去做。

问题二：
Retrofit中通过addConverterFactory来自定义一个Convert类，在convert方法里面做转换。


[拆轮子系列：拆 Retrofit](https://blog.piasy.com/2016/06/25/Understand-Retrofit/index.html)