# OKHTTP

1. 利用 okhttp 实现基本的网络访问功能，包括基本的数据请求，表单提交，文件上传，文件断点下载，https的设置等等。

2. 深入研究 okhttp 源码，熟悉 okhttp 中的调用过程，拦截器原理，缓存原理以及其中涉及的设计模式，并可以自定义拦截器实现特殊的功能，如日志打印等等。

3. 在研究 okhttp 缓存原理之前，得首先熟悉 http 缓存的相关字段以及在设置 https 时，也要全面复习 https 的相关原理。
  
4. 怎么实现多路复用的（这个主要从https实现多路复用的原理上谈，用了二进制分帧，那OKHttp其实就是按照分帧来读取的，具体可以看相关源码。）

### 问题
1. 线程池如何优化的
2. 缓存怎么做的


* Interceptors
* 责任链设计模式


```
RealCall.java

使用责任链模式，处理一系列拦截器

Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);
    return chain.proceed(originalRequest);
  }



RealInterceptorChain.java

  public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
      Connection connection) throws IOException {
    if (index >= interceptors.size()) throw new AssertionError();

    calls++;

    ... //安全检查

    // Call the next interceptor in the chain.
    RealInterceptorChain next = new RealInterceptorChain(
        interceptors, streamAllocation, httpCodec, connection, index + 1, request);
    Interceptor interceptor = interceptors.get(index);
    Response response = interceptor.intercept(next);

    ... //安全检查

    return response;
  }
```
[你猜一个TCP连接上面能发多少个HTTP请求](https://zhuanlan.zhihu.com/p/61423830)
[OkHttp相关](https://juejin.im/post/5d450c3e6fb9a06af92b863e?utm_source=gold_browser_extension)

[拆OkHttp](https://blog.piasy.com/2016/07/11/Understand-OkHttp/index.html)

[OkHttp里的设计模式](https://www.jianshu.com/p/5cd6775cbb51​)
[OkHttp的源码过程](https://juejin.im/post/5a704ed05188255a8817f4c9)
[OkHttp的系列文章](https://www.jianshu.com/p/82f74db14a18​)