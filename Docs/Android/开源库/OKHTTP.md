# OkHttp

### 为什么要使用OkHttp
* OkHttp 提供了对最新的 HTTP 协议版本 HTTP/2 和 SPDY 的支持，这 使得对同一个主机发出的所有请求都可以共享相同的套接字连接。
* 如果 HTTP/2 和 SPDY 不可用，OkHttp 会使用连接池来复用连接以提高效率。
* OkHttp 提供了对 GZIP 的默认支持来降低传输内容的大小。
* OkHttp 也提供了对 HTTP 响应的缓存机制，可以避免不必要的网络请
求。
* 当网络出现问题时，OkHttp 会自动重试一个主机的多个 IP 地址。

### OkHttp有哪些用法

GET、POST 请求、上传文件、上传表单等等。

### OkHttp核心实现原理是什么

OkHttp 内部的请求流程:使用 OkHttp 会在请求的时候初始化一个 Call 的实例， 然后执行它的 execute()方法或 enqueue()方法，内部最后都会执行到 getResponseWithInterceptorChain()方法，这个方法里面通过拦截器组成的责任链，依次经过用户自定义普通拦截器、重试拦截器、桥接拦截器、缓存拦截器、
连接拦截器和用户自定义网络拦截器以及访问服务器拦截器等拦截处理过程，来 获取到一个响应并交给用户。其中，除了 OKHttp 的内部请求流程这点之外，缓存和连接这两部分内容也是两个很重要的点。

* Interceptors:用户自定义拦截器
* retryAndFollowUpInterceptor:负责失败重试以及重定向
* BridgeInterceptor:请求时，对必要的 Header 进行一些添加，接收响应 时，移除必要的 Header
* CacheInterceptor: 负责读取缓存直接返回(根据请求的信息和缓存的响 应的信息来判断是否存在缓存可用)、更新缓存
* ConnectInterceptor:负责和服务器建立连接
* NetworkInterceptors:用户定义网络拦截器
* CallServerInterceptor:负责向服务器发送请求数据、从服务器读取响应数据

### ConnectionPool
1. 判断连接是否可用，不可用则从 ConnectionPool 获取连接，ConnectionPool
无连接，创建新连接，握手，放入 ConnectionPool。
2. 它是一个 Deque，add 添加 Connection，使用线程池负责定时清理缓存。 
3. 使用连接复用省去了进行 TCP 和 TLS 握手的一个过程

### 从这个库中学到什么有价值的或者说可借鉴的设计思想

使用责任链模式实现拦截器的分层设计，每一个拦截器对应一个功能，充分实现 了功能解耦，易维护。
### 手写拦截器

### 如何理解OkHttp
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