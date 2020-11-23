- [OKHttp](#OKHttp)
  - [OKHttp请求流程](#OKHttp请求流程)
    - [新建OKHttpClient客户端](#新建OKHttpClient客户端)
    - [同步请求流程](#同步请求流程)
    - [异步请求流程](#异步请求流程)
  - [网络请求缓存处理](#网络请求缓存处理之CacheInterceptor)
  - [连接池](#ConnectInterceptor之连接池)
- [Retrofit](#Retrofit)
  - [基本使用流程](#基本使用流程)
  - [Retrofit构建过程](#Retrofit构建过程)
    - [Retrofit核心对象解析](#Retrofit核心对象解析)
    - [Builder内部构造](#Builder内部构造)
    - [添加baseUrl](#添加baseUrl)
    - [添加GsonConverterFactory](#添加GsonConverterFactory)
    - [build过程](#build过程)
  - [创建网络请求接口实例过程](#创建网络请求接口实例过程)
  - [创建网络请求接口类实例并执行请求过程](#创建网络请求接口类实例并执行请求过程)
  - [Retrofit源码流程图](#Retrofit源码流程图)
- [Glide](#Glide)
  - [基本使用流程](#基本使用流程-1)
  - [GlideApp.with(context)源码详解](#GlideAppwithcontext源码详解)
  - [load(url)源码详解](#loadurl源码详解)
  - [into(iv)源码详解](#intoiv源码详解)
  - [完整Glide加载流程图](#完整Glide加载流程图)
- [GreenDao](#GreenDao)
  - [基本使用流程](#基本使用流程-2)
  - [GreenDao使用流程分析](#GreenDao使用流程分析)
    - [创建数据库帮助类对象DaoMaster.DevOpenHelper](#创建数据库帮助类对象DaoMasterDevOpenHelper)
    - [创建DaoMaster对象](#创建DaoMaster对象)
    - [创建DaoSession对象](#创建DaoSession对象)
    - [插入源码分析](#插入源码分析)
    - [查询源码分析](#查询源码分析)
  - [GreenDao是如何与ReactiveX结合？](#GreenDao是如何与ReactiveX结合？)
- [RxJava](#RxJava)
  - [RxJava是什么？](#RxJava是什么？)
  - [RxJava的订阅流程](#RxJava的订阅流程)
    - [创建被观察者过程](#创建被观察者过程)
    - [订阅过程](#订阅过程)
  - [RxJava的线程切换](#RxJava的线程切换)
- [LeakCanary](#LeakCanary)
  - [原理概述](#原理概述)
  - [简单示例](#简单示例)
  - [源码分析](#源码分析)
  - [LeakCanary运作流程](#LeakCanary运作流程)
- [ButterKnife](#ButterKnife)
  - [简单示例](#简单示例-1)
  - [源码分析](#源码分析-1)
    - [模板代码解析](#模板代码解析)
    - [ButterKnife 是怎样实现代码注入的](#ButterKnife-是怎样实现代码注入的)
    - [ButterKnife是如何在编译时生成代码的？](#ButterKnife是如何在编译时生成代码的？)
- [Dagger 2](#Dagger-2)
  - [预备知识](#预备知识)
    - [@Inject](#@Inject)
    - [@Module](#@Module)
    - [@Singleton](#@Singleton)
    - [@Providers](#@Providers)
    - [@Component](#@Component)
    - [@Scope](#@Scope)
    - [@Qualifier](#@Qualifier)
    - [dependencies](#dependencies)
    - [@SubComponent](#@SubComponent)
  - [简单示例](#简单示例-2)
  - [源码分析](#源码分析-2)
- [EventBus](#EventBus)
  - [简单示例](#简单示例-3)
  - [源码分析](#源码分析-3)

# OKHttp

## OKHttp请求流程

OKHttp内部的大致请求流程图如下所示：

![image](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/976154d56c224d96b33b1ca424e935a2~tplv-k3u1fbpfcp-zoom-1.image)

如下为使用OKHttp进行Get请求的步骤：

```
//1.新建OKHttpClient客户端
OkHttpClient client = new OkHttpClient();
//新建一个Request对象
Request request = new Request.Builder()
        .url(url)
        .build();
//2.Response为OKHttp中的响应
Response response = client.newCall(request).execute();
```

### 新建OKHttpClient客户端

```
OkHttpClient client = new OkHttpClient();

public OkHttpClient() {
    this(new Builder());
}

OkHttpClient(Builder builder) {
    ....
}
```

可以看到，OkHttpClient使用了建造者模式，Builder里面的可配置参数如下：

```
public static final class Builder {
    Dispatcher dispatcher;// 分发器
    @Nullable Proxy proxy;
    List<Protocol> protocols;
    List<ConnectionSpec> connectionSpecs;// 传输层版本和连接协议
    final List<Interceptor> interceptors = new ArrayList<>();// 拦截器
    final List<Interceptor> networkInterceptors = new ArrayList<>();
    EventListener.Factory eventListenerFactory;
    ProxySelector proxySelector;
    CookieJar cookieJar;
    @Nullable Cache cache;
    @Nullable InternalCache internalCache;// 内部缓存
    SocketFactory socketFactory;
    @Nullable SSLSocketFactory sslSocketFactory;// 安全套接层socket 工厂，用于HTTPS
    @Nullable CertificateChainCleaner certificateChainCleaner;// 验证确认响应证书 适用 HTTPS 请求连接的主机名。
    HostnameVerifier hostnameVerifier;// 验证确认响应证书 适用 HTTPS 请求连接的主机名。  
    CertificatePinner certificatePinner;// 证书锁定，使用CertificatePinner来约束哪些认证机构被信任。
    Authenticator proxyAuthenticator;// 代理身份验证
    Authenticator authenticator;// 身份验证
    ConnectionPool connectionPool;// 连接池
    Dns dns;
    boolean followSslRedirects; // 安全套接层重定向
    boolean followRedirects;// 本地重定向
    boolean retryOnConnectionFailure;// 重试连接失败
    int callTimeout;
    int connectTimeout;
    int readTimeout;
    int writeTimeout;
    int pingInterval;

    // 这里是默认配置的构建参数
    public Builder() {
        dispatcher = new Dispatcher();
        protocols = DEFAULT_PROTOCOLS;
        connectionSpecs = DEFAULT_CONNECTION_SPECS;
        ...
    }

    // 这里传入自己配置的构建参数
    Builder(OkHttpClient okHttpClient) {
        this.dispatcher = okHttpClient.dispatcher;
        this.proxy = okHttpClient.proxy;
        this.protocols = okHttpClient.protocols;
        this.connectionSpecs = okHttpClient.connectionSpecs;
        this.interceptors.addAll(okHttpClient.interceptors);
        this.networkInterceptors.addAll(okHttpClient.networkInterceptors);
        ...
    }
```

### 同步请求流程

```
Response response = client.newCall(request).execute();

/**
* Prepares the {@code request} to be executed at   some point in the future.
*/
@Override public Call newCall(Request request) {
    return RealCall.newRealCall(this, request, false /* for web socket */);
}

// RealCall为真正的请求执行者
static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    // Safely publish the Call instance to the EventListener.
    RealCall call = new RealCall(client, originalRequest, forWebSocket);
    call.eventListener = client.eventListenerFactory().create(call);
    return call;
}

@Override public Response execute() throws IOException {
    synchronized (this) {
        // 每个Call只能执行一次
        if (executed) throw new IllegalStateException("Already Executed");
        executed = true;
    }
    captureCallStackTrace();
    timeout.enter();
    eventListener.callStart(this);
    try {
        // 通知dispatcher已经进入执行状态
        client.dispatcher().executed(this);
        // 通过一系列的拦截器请求处理和响应处理得到最终的返回结果
        Response result = getResponseWithInterceptorChain();
        if (result == null) throw new IOException("Canceled");
        return result;
    } catch (IOException e) {
        e = timeoutExit(e);
        eventListener.callFailed(this, e);
        throw e;
    } finally {
        // 通知 dispatcher 自己已经执行完毕
        client.dispatcher().finished(this);
    }
}

Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    // 在配置 OkHttpClient 时设置的 interceptors；
    interceptors.addAll(client.interceptors());
    // 负责失败重试以及重定向
    interceptors.add(retryAndFollowUpInterceptor);
    // 请求时，对必要的Header进行一些添加，接收响应时，移除必要的Header
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    // 负责读取缓存直接返回、更新缓存
    interceptors.add(new CacheInterceptor(client.internalCache()));
    // 负责和服务器建立连接
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
        // 配置 OkHttpClient 时设置的 networkInterceptors
        interceptors.addAll(client.networkInterceptors());
    }
    // 负责向服务器发送请求数据、从服务器读取响应数据
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
        originalRequest, this, eventListener, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    // 使用责任链模式开启链式调用
    return chain.proceed(originalRequest);
}

// StreamAllocation 对象，它相当于一个管理类，维护了服务器连接、并发流
// 和请求之间的关系，该类还会初始化一个 Socket 连接对象，获取输入/输出流对象。
public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec,
  RealConnection connection) throws IOException {
    ...

    // Call the next interceptor in the chain.
    // 实例化下一个拦截器对应的RealIterceptorChain对象
    RealInterceptorChain next = new RealInterceptorChain(interceptors, streamAllocation, httpCodec,
        connection, index + 1, request, call, eventListener, connectTimeout, readTimeout,
        writeTimeout);
    // 得到当前的拦截器
    Interceptor interceptor = interceptors.get(index);
    // 调用当前拦截器的intercept()方法，并将下一个拦截器的RealIterceptorChain对象传递下去,最后得到响应
    Response response = interceptor.intercept(next);

    ...
    
    return response;
}
```

### 异步请求流程

```
Request request = new Request.Builder()
    .url("http://publicobject.com/helloworld.txt")
    .build();

client.newCall(request).enqueue(new Callback() {
    @Override 
    public void onFailure(Call call, IOException e) {
      e.printStackTrace();
    }

    @Override 
    public void onResponse(Call call, Response response) throws IOException {
        ...
    }
    
void enqueue(AsyncCall call) {
    synchronized (this) {
        readyAsyncCalls.add(call);
    }
    promoteAndExecute();
}

// 正在准备中的异步请求队列
private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();

// 运行中的异步请求
private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();

// 同步请求
private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();

// Promotes eligible calls from {@link #readyAsyncCalls} to {@link #runningAsyncCalls} and runs
// them on the executor service. Must not be called with synchronization because executing calls
// can call into user code.
private boolean promoteAndExecute() {
    assert (!Thread.holdsLock(this));

    List<AsyncCall> executableCalls = new ArrayList<>();
    boolean isRunning;
    synchronized (this) {
      for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
        AsyncCall asyncCall = i.next();

        // 如果其中的runningAsynCalls不满，且call占用的host小于最大数量，则将call加入到runningAsyncCalls中执行，
        // 同时利用线程池执行call；否者将call加入到readyAsyncCalls中。
        if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
        if (runningCallsForHost(asyncCall) >= maxRequestsPerHost) continue; // Host max capacity.

        i.remove();
        executableCalls.add(asyncCall);
        runningAsyncCalls.add(asyncCall);
      }
      isRunning = runningCallsCount() > 0;
    }

    for (int i = 0, size = executableCalls.size(); i < size; i++) {
      AsyncCall asyncCall = executableCalls.get(i);
      asyncCall.executeOn(executorService());
    }

    return isRunning;
}
```

最后，我们在看看AsynCall的代码。

```
final class AsyncCall extends NamedRunnable {
    private final Callback responseCallback;

    AsyncCall(Callback responseCallback) {
      super("OkHttp %s", redactedUrl());
      this.responseCallback = responseCallback;
    }

    String host() {
      return originalRequest.url().host();
    }

    Request request() {
      return originalRequest;
    }

    RealCall get() {
      return RealCall.this;
    }

    /**
     * Attempt to enqueue this async call on {@code    executorService}. This will attempt to clean up
     * if the executor has been shut down by reporting    the call as failed.
     */
    void executeOn(ExecutorService executorService) {
      assert (!Thread.holdsLock(client.dispatcher()));
      boolean success = false;
      try {
        executorService.execute(this);
        success = true;
      } catch (RejectedExecutionException e) {
        InterruptedIOException ioException = new InterruptedIOException("executor rejected");
        ioException.initCause(e);
        eventListener.callFailed(RealCall.this, ioException);
        responseCallback.onFailure(RealCall.this, ioException);
      } finally {
        if (!success) {
          client.dispatcher().finished(this); // This call is no longer running!
        }
      }
    }

    @Override protected void execute() {
      boolean signalledCallback = false;
      timeout.enter();
      try {
        // 跟同步执行一样，最后都会调用到这里
        Response response = getResponseWithInterceptorChain();
        if (retryAndFollowUpInterceptor.isCanceled()) {
          signalledCallback = true;
          responseCallback.onFailure(RealCall.this, new   IOException("Canceled"));
        } else {
          signalledCallback = true;
          responseCallback.onResponse(RealCall.this,   response);
        }
      } catch (IOException e) {
        e = timeoutExit(e);
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure   for " + toLoggableString(), e);
        } else {
          eventListener.callFailed(RealCall.this, e);
          responseCallback.onFailure(RealCall.this, e);
        }
      } finally {
        client.dispatcher().finished(this);
      }
    }
}
```

从上面的源码可以知道，拦截链的处理OKHttp帮我们默认做了五步拦截处理，其中RetryAndFollowUpInterceptor、BridgeInterceptor、CallServerInterceptor内部的源码很简洁易懂，此处不再多说。

## 网络请求缓存处理之CacheInterceptor

```
@Override public Response intercept(Chain chain) throws IOException {
    // 根据request得到cache中缓存的response
    Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;

    long now = System.currentTimeMillis();

    // request判断缓存的策略，是否要使用了网络，缓存或两者都使用
    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(),     cacheCandidate).get();
    Request networkRequest = strategy.networkRequest;
    Response cacheResponse = strategy.cacheResponse;

    if (cache != null) {
      cache.trackResponse(strategy);
    }

    if (cacheCandidate != null && cacheResponse == null) {
      closeQuietly(cacheCandidate.body()); // The cache   candidate wasn't applicable. Close it.
    }

    // If we're forbidden from using the network and the cache is insufficient, fail.
    if (networkRequest == null && cacheResponse == null) {
      return new Response.Builder()
          .request(chain.request())
          .protocol(Protocol.HTTP_1_1)
          .code(504)
          .message("Unsatisfiable Request (only-if-cached)")
          .body(Util.EMPTY_RESPONSE)
          .sentRequestAtMillis(-1L)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();
    }

    // If we don't need the network, we're done.
    if (networkRequest == null) {
      return cacheResponse.newBuilder()
          .cacheResponse(stripBody(cacheResponse))
          .build();
    }

    Response networkResponse = null;
    try {
        // 调用下一个拦截器，决定从网络上来得到response
        networkResponse = chain.proceed(networkRequest);
    } finally {
        // If we're crashing on I/O or otherwise,   don't leak the cache body.
        if (networkResponse == null && cacheCandidate != null) {
          closeQuietly(cacheCandidate.body());
        }
    }

    // If we have a cache response too, then we're doing a conditional get.
    // 如果本地已经存在cacheResponse，那么让它和网络得到的networkResponse做比较，决定是否来更新缓存的cacheResponse
    if (cacheResponse != null) {
        if (networkResponse.code() == HTTP_NOT_MODIFIED)   {
          Response response = cacheResponse.newBuilder()
                  .headers(combine(cacheResponse.headers(), networkResponse.headers()))
                  .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
                  .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
                  .cacheResponse(stripBody(cacheResponse))
                  .networkResponse(stripBody(networkResponse))
              .build();
          networkResponse.body().close();
    
          // Update the cache after combining headers but before stripping the
          // Content-Encoding header (as performed by initContentStream()).
          cache.trackConditionalCacheHit();
          cache.update(cacheResponse, response);
          return response;
        } else {
          closeQuietly(cacheResponse.body());
        }
    }

    Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();

    if (cache != null) {
      if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response,   networkRequest)) {
        // Offer this request to the cache.
        // 缓存未经缓存过的response
        CacheRequest cacheRequest = cache.put(response);
        return cacheWritingResponse(cacheRequest, response);
      }

      if (HttpMethod.invalidatesCache(networkRequest.method())) {
        try {
          cache.remove(networkRequest);
        } catch (IOException ignored) {
          // The cache cannot be written.
        }
      }
    }

    return response;
}
```

缓存拦截器会根据请求的信息和缓存的响应的信息来判断是否存在缓存可用，如果有可以使用的缓存，那么就返回该缓存给用户，否则就继续使用责任链模式来从服务器中获取响应。当获取到响应的时候，又会把响应缓存到磁盘上面。

## ConnectInterceptor之连接池

```
@Override public Response intercept(Chain chain) throws IOException {
    RealInterceptorChain realChain = (RealInterceptorChain) chain;
    Request request = realChain.request();
    StreamAllocation streamAllocation = realChain.streamAllocation();

    // We need the network to satisfy this request.     Possibly for validating a conditional GET.
    boolean doExtensiveHealthChecks = !request.method().equals("GET");
    // HttpCodec是对 HTTP 协议操作的抽象，有两个实现：Http1Codec和Http2Codec，顾名思义，它们分别对应 HTTP/1.1 和 HTTP/2 版本的实现。在这个方法的内部实现连接池的复用处理
    HttpCodec httpCodec = streamAllocation.newStream(client, chain,     doExtensiveHealthChecks);
    RealConnection connection = streamAllocation.connection();

    return realChain.proceed(request, streamAllocation, httpCodec, connection);
}



// Returns a connection to host a new stream. This // prefers the existing connection if it exists,
// then the pool, finally building a new connection.
// 调用 streamAllocation 的 newStream() 方法的时候，最终会经过一系列
// 的判断到达 StreamAllocation 中的 findConnection() 方法
private RealConnection findConnection(int   connectTimeout, int readTimeout, int writeTimeout,
    int pingIntervalMillis, boolean connectionRetryEnabled) throws IOException {
      ...
    
      // Attempt to use an already-allocated connection. We need to be careful here because our
      // already-allocated connection may have been restricted from creating new streams.
      // 尝试使用已分配的连接，已经分配的连接可能已经被限制创建新的流
      releasedConnection = this.connection;
      // 释放当前连接的资源，如果该连接已经被限制创建新的流，就返回一个Socket以关闭连接
      toClose = releaseIfNoNewStreams();
      if (this.connection != null) {
        // We had an already-allocated connection and it's good.
        result = this.connection;
        releasedConnection = null;
      }
      if (!reportedAcquired) {
        // If the connection was never reported acquired, don't report it as released!
        // 如果该连接从未被标记为获得，不要标记为发布状态，reportedAcquired 通过 acquire()   方法修改
        releasedConnection = null;
      }
    
      if (result == null) {
        // Attempt to get a connection from the pool.
        // 尝试供连接池中获取一个连接
        Internal.instance.get(connectionPool, address, this, null);
        if (connection != null) {
          foundPooledConnection = true;
          result = connection;
        } else {
          selectedRoute = route;
        }
      }
    }
    // 关闭连接
    closeQuietly(toClose);
    
    if (releasedConnection != null) {
      eventListener.connectionReleased(call, releasedConnection);
    }
    if (foundPooledConnection) {
      eventListener.connectionAcquired(call, result);
    }
    if (result != null) {
      // If we found an already-allocated or pooled connection, we're done.
      // 如果已经从连接池中获取到了一个连接，就将其返回
      return result;
    }
    
    // If we need a route selection, make one. This   is a blocking operation.
    boolean newRouteSelection = false;
    if (selectedRoute == null && (routeSelection == null || !routeSelection.hasNext())) {
      newRouteSelection = true;
      routeSelection = routeSelector.next();
    }
    
    synchronized (connectionPool) {
      if (canceled) throw new IOException("Canceled");
    
      if (newRouteSelection) {
        // Now that we have a set of IP addresses,   make another attempt at getting a   connection from
        // the pool. This could match due to   connection coalescing.
         // 根据一系列的 IP地址从连接池中获取一个链接
        List<Route> routes = routeSelection.getAll();
        for (int i = 0, size = routes.size(); i < size;i++) {
          Route route = routes.get(i);
          // 从连接池中获取一个连接
          Internal.instance.get(connectionPool, address, this, route);
          if (connection != null) {
            foundPooledConnection = true;
            result = connection;
            this.route = route;
            break;
          }
        }
      }
    
      if (!foundPooledConnection) {
        if (selectedRoute == null) {
          selectedRoute = routeSelection.next();
        }
    
        // Create a connection and assign it to this allocation immediately. This makes it   possible
        // for an asynchronous cancel() to interrupt the handshake we're about to do.
        // 在连接池中如果没有该连接，则创建一个新的连接，并将其分配，这样我们就可以在握手之前进行终端
        route = selectedRoute;
        refusedStreamCount = 0;
        result = new RealConnection(connectionPool, selectedRoute);
        acquire(result, false);
      }
    }
    // If we found a pooled connection on the 2nd time around, we're done.
    if (foundPooledConnection) {
    // 如果我们在第二次的时候发现了一个池连接，那么我们就将其返回
      eventListener.connectionAcquired(call, result);
      return result;
    }

    // Do TCP + TLS handshakes. This is a blocking     operation.
     // 进行 TCP 和 TLS 握手
    result.connect(connectTimeout, readTimeout, writeTimeout, pingIntervalMillis,
      connectionRetryEnabled, call, eventListener);
    routeDatabase().connected(result.route());

    Socket socket = null;
    synchronized (connectionPool) {
      reportedAcquired = true;

      // Pool the connection.
      // 将该连接放进连接池中
      Internal.instance.put(connectionPool, result);

      // If another multiplexed connection to the same   address was created concurrently, then
      // release this connection and acquire that one.
      // 如果同时创建了另一个到同一地址的多路复用连接，释放这个连接并获取那个连接
      if (result.isMultiplexed()) {
        socket = Internal.instance.deduplicate(connectionPool, address, this);
        result = connection;
      }
    }
    closeQuietly(socket);

    eventListener.connectionAcquired(call, result);
    return result;
}
```

从以上的源码分析可知：

- 判断当前的连接是否可以使用：流是否已经被关闭，并且已经被限制创建新的流；
- 如果当前的连接无法使用，就从连接池中获取一个连接；
- 连接池中也没有发现可用的连接，创建一个新的连接，并进行握手，然后将其放到连接池中。

在从连接池中获取一个连接的时候，使用了 Internal 的 get() 方法。Internal 有一个静态的实例，会在 OkHttpClient 的静态代码快中被初始化。我们会在 Internal 的 get() 中调用连接池的 get() 方法来得到一个连接。并且，从中我们明白了连接复用的一个好处就是省去了进行 TCP 和 TLS 握手的一个过程。因为建立连接本身也是需要消耗一些时间的，连接被复用之后可以提升我们网络访问的效率。

接下来详细分析下ConnectionPool是如何实现连接管理的。

OkHttp 的缓存管理分成两个步骤，一边当我们创建了一个新的连接的时候，我们要把它放进缓存里面；另一边，我们还要来对缓存进行清理。在 ConnectionPool 中，当我们向连接池中缓存一个连接的时候，只要调用双端队列的 add() 方法，将其加入到双端队列即可，而清理连接缓存的操作则交给线程池来定时执行。

```
private final Deque<RealConnection> connections = new ArrayDeque<>();

void put(RealConnection connection) {
assert (Thread.holdsLock(this));
    if (!cleanupRunning) {
      cleanupRunning = true;
      // 使用线程池执行清理任务
      executor.execute(cleanupRunnable);
    }
    // 将新建的连接插入到双端队列中
    connections.add(connection);
}

 private final Runnable cleanupRunnable = new Runnable() {
@Override public void run() {
    while (true) {
        // 内部调用 cleanup() 方法来清理无效的连接
        long waitNanos = cleanup(System.nanoTime());
        if (waitNanos == -1) return;
        if (waitNanos > 0) {
          long waitMillis = waitNanos / 1000000L;
          waitNanos -= (waitMillis * 1000000L);
          synchronized (ConnectionPool.this) {
            try {
              ConnectionPool.this.wait(waitMillis, (int) waitNanos);
            } catch (InterruptedException ignored) {
            }
          }
        }
    }
};

long cleanup(long now) {
    int inUseConnectionCount = 0;
    int idleConnectionCount = 0;
    RealConnection longestIdleConnection = null;
    long longestIdleDurationNs = Long.MIN_VALUE;

    // Find either a connection to evict, or the time that the next eviction is due.
    synchronized (this) {
        // 遍历所有的连接
        for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
          RealConnection connection = i.next();
    
          // If the connection is in use, keep     searching.
          // 遍历所有的连接
          if (pruneAndGetAllocationCount(connection, now) > 0) {
            inUseConnectionCount++;
            continue;
          }
    
          idleConnectionCount++;
    
          // If the connection is ready to be     evicted,     we're done.
          // 如果找到了一个可以被清理的连接，会尝试去寻找闲置时间最久的连接来释放
          long idleDurationNs = now - connection.idleAtNanos;
          if (idleDurationNs > longestIdleDurationNs) {
            longestIdleDurationNs = idleDurationNs;
            longestIdleConnection = connection;
          }
        }
    
        // maxIdleConnections 表示最大允许的闲置的连接的数量,keepAliveDurationNs表示连接允许存活的最长的时间。
        // 默认空闲连接最大数目为5个，keepalive 时间最长为5分钟。
        if (longestIdleDurationNs >= this.keepAliveDurationNs
            || idleConnectionCount > this.maxIdleConnections) {
          // We've found a connection to evict. Remove it from the list, then close it     below (outside
          // of the synchronized block).
          // 该连接的时长超出了最大的活跃时长或者闲置的连接数量超出了最大允许的范围，直接移除
          connections.remove(longestIdleConnection);
        } else if (idleConnectionCount > 0) {
          // A connection will be ready to evict soon.
          // 闲置的连接的数量大于0，停顿指定的时间（等会儿会将其清理掉，现在还不是时候）
          return keepAliveDurationNs - longestIdleDurationNs;
        } else if (inUseConnectionCount > 0) {
          // All connections are in use. It'll be at least the keep alive duration 'til we run again.
          // 所有的连接都在使用中，5分钟后再清理
          return keepAliveDurationNs;
        } else {
          // No connections, idle or in use.
           // 没有连接
          cleanupRunning = false;
          return -1;
      }
}
```

从以上的源码分析可知，首先会对缓存中的连接进行遍历，以寻找一个闲置时间最长的连接，然后根据该连接的闲置时长和最大允许的连接数量等参数来决定是否应该清理该连接。同时注意上面的方法的返回值是一个时间，如果闲置时间最长的连接仍然需要一段时间才能被清理的时候，会返回这段时间的时间差，然后会在这段时间之后再次对连接池进行清理。



> 经过上面对OKHttp内部工作机制的一系列分析，相信你已经对OKHttp已经有了一个比较深入的了解了。首先，我们会在请求的时候初始化一个Call的实例，然后执行它的execute()方法或enqueue()方法，内部最后都会执行到getResponseWithInterceptorChain()方法，这个方法里面通过拦截器组成的责任链，依次经过用户自定义普通拦截器、重试拦截器、桥接拦截器、缓存拦截器、连接拦截器和用户自定义网络拦截器以及访问服务器拦截器等拦截处理过程，来获取到一个响应并交给用户。
>
> 其中，除了OKHttp的内部请求流程这点之外，缓存和连接这两部分内容也是两个很重要的点，相信经过讲解，大家对这三部分重点内容已经有了自己的理解。

# Retrofit

## 基本使用流程

### 定义HTTP API，用于描述请求

```
public interface GitHubService {

     @GET("users/{user}/repos")
     Call<List<Repo>> listRepos(@Path("user") String user);
}
```

### 创建Retrofit并生成API的实现

> （**注意：**方法上面的注解表示请求的接口部分，返回类型是请求的返回值类型，方法的参数即是请求的参数）

```
// 1.Retrofit构建过程
Retrofit retrofit = new Retrofit.Builder()
.baseUrl("https://api.github.com/")
.build();

// 2.创建网络请求接口类实例过程
GitHubService service = retrofit.create(GitHubService.class);
```

### 调用API方法，生成Call，执行请求

```
// 3.生成并执行请求过程
Call<List<Repo>> repos = service.listRepos("octocat");
repos.execute() or repos.enqueue()
```

Retrofit的基本使用流程很简洁，但是简洁并不代表简单，Retrofit为了实现这种简洁的使用流程，内部使用了优秀的架构设计和大量的设计模式，在分析过Retrofit最新版的源码和大量优秀的Retrofit源码分析文章后发现，要想真正理解Retrofit内部的核心源码流程和设计思想，首先，需要对这九大设计模式有一定的了解，如下：

```
1.Retrofit构建过程 
建造者模式、工厂方法模式

2.创建网络请求接口实例过程
外观模式、代理模式、单例模式、策略模式、装饰模式（建造者模式）

3.生成并执行请求过程
适配器模式（代理模式、装饰模式）
```

其次，需要对OKHttp源码有一定的了解。让我们按以上流程去深入Retrofit源码内部，领悟它带给我们的**设计之美**。

## Retrofit构建过程

### Retrofit核心对象解析

首先Retrofit中有一个全局变量非常关键，在V2.5之前的版本，使用的是LinkedHashMap()，它是一个网络请求配置对象，是由网络请求接口中方法注解进行解析后得到的。

```
public final class Retrofit {

    // 网络请求配置对象，存储网络请求相关的配置，如网络请求的方法、数据转换器、网络请求适配器、网络请求工厂、基地址等
    private final Map<Method, ServiceMethod<?>> serviceMethodCache = new ConcurrentHashMap<>();
```

Retrofit使用了建造者模式通过内部类Builder类建立一个Retrofit实例，如下：

```
public static final class Builder {

    // 平台类型对象（Platform -> Android)
    private final Platform platform;
    // 网络请求工厂，默认使用OkHttpCall（工厂方法模式）
    private @Nullable okhttp3.Call.Factory callFactory;
    // 网络请求的url地址
    private @Nullable HttpUrl baseUrl;
    // 数据转换器工厂的集合
    private final List<Converter.Factory> converterFactories = new ArrayList<>();
    // 网络请求适配器工厂的集合，默认是ExecutorCallAdapterFactory
    private final List<CallAdapter.Factory> callAdapterFactories = new ArrayList<>();
    // 回调方法执行器，在 Android 上默认是封装了 handler 的 MainThreadExecutor, 默认作用是：切换线程（子线程 -> 主线程）
    private @Nullable Executor callbackExecutor;
    // 一个开关，为true则会缓存创建的ServiceMethod
    private boolean validateEagerly;
```

### Builder内部构造

下面看看Builder内部构造做了什么。

```
public static final class Builder {

    ...
    
    Builder(Platform platform) {
        this.platform = platform;
    }

    
    public Builder() {
        this(Platform.get());
    }
    
    ...
    
}


class Platform {

    private static final Platform PLATFORM = findPlatform();
    
    static Platform get() {
      return PLATFORM;
    }
    
    private static Platform findPlatform() {
      try {
        // 使用JVM加载类的方式判断是否是Android平台
        Class.forName("android.os.Build");
        if (Build.VERSION.SDK_INT != 0) {
          return new Android();
        }
      } catch (ClassNotFoundException ignored) {
      }
      try {
        // 同时支持Java平台
        Class.forName("java.util.Optional");
        return new Java8();
      } catch (ClassNotFoundException ignored) {
      }
      return new Platform();
    }

static class Android extends Platform {

    ...

    
    @Override public Executor defaultCallbackExecutor() {
        //切换线程（子线程 -> 主线程）
        return new MainThreadExecutor();
    }

    // 创建默认的网络请求适配器工厂，如果是Android7.0或Java8上，则使
    // 用了并发包中的CompletableFuture保证了回调的同步
    // 在Retrofit中提供了四种CallAdapterFactory(策略模式)：
    // ExecutorCallAdapterFactory（默认）、GuavaCallAdapterFactory、
    // va8CallAdapterFactory、RxJavaCallAdapterFactory
    @Override List<? extends CallAdapter.Factory> defaultCallAdapterFactories(
        @Nullable Executor callbackExecutor) {
      if (callbackExecutor == null) throw new AssertionError();
      ExecutorCallAdapterFactory executorFactory = new   ExecutorCallAdapterFactory(callbackExecutor);
      return Build.VERSION.SDK_INT >= 24
        ? asList(CompletableFutureCallAdapterFactory.INSTANCE, executorFactory)
        : singletonList(executorFactory);
    }
    
    ...

    @Override List<? extends Converter.Factory> defaultConverterFactories() {
      return Build.VERSION.SDK_INT >= 24
          ? singletonList(OptionalConverterFactory.INSTANCE)
          : Collections.<Converter.Factory>emptyList();
    }

    ...
    
    static class MainThreadExecutor implements Executor {
    
        // 获取Android 主线程的Handler 
        private final Handler handler = new Handler(Looper.getMainLooper());

        @Override public void execute(Runnable r) {
        
            // 在UI线程对网络请求返回数据处理
            handler.post(r);
        }
    }
}
```

可以看到，在Builder内部构造时设置了默认Platform、callAdapterFactories和callbackExecutor。

### 添加baseUrl

很简单，就是将String类型的url转换为OkHttp的HttpUrl过程如下：

```
/**
 * Set the API base URL.
 *
 * @see #baseUrl(HttpUrl)
 */
public Builder baseUrl(String baseUrl) {
    checkNotNull(baseUrl, "baseUrl == null");
    return baseUrl(HttpUrl.get(baseUrl));
}

public Builder baseUrl(HttpUrl baseUrl) {
    checkNotNull(baseUrl, "baseUrl == null");
    List<String> pathSegments = baseUrl.pathSegments();
    if (!"".equals(pathSegments.get(pathSegments.size() - 1))) {
      throw new IllegalArgumentException("baseUrl must end in /: " + baseUrl);
    }
    this.baseUrl = baseUrl;
    return this;
}
```

### 添加GsonConverterFactory

首先，看到GsonConverterFactory.creat()的源码。

```
public final class GsonConverterFactory extends Converter.Factory {
 
    public static GsonConverterFactory create() {
        return create(new Gson());
    }
    
    
    public static GsonConverterFactory create(Gson gson) {
        if (gson == null) throw new NullPointerException("gson ==   null");
        return new GsonConverterFactory(gson);
    }
    
    private final Gson gson;
    
    // 创建了一个含有Gson对象实例的GsonConverterFactory
    private GsonConverterFactory(Gson gson) {
        this.gson = gson;
    }

```

然后，看看addConverterFactory()方法内部。

```
public Builder addConverterFactory(Converter.Factory factory) {
    converterFactories.add(checkNotNull(factory, "factory null"));
    return this;
}
```

可知，这一步是将一个含有Gson对象实例的GsonConverterFactory放入到了数据转换器工厂converterFactories里。

### build过程

```
public Retrofit build() {

    if (baseUrl == null) {
      throw new IllegalStateException("Base URL required.");
    }
    
    okhttp3.Call.Factory callFactory = this.callFactory;
    if (callFactory == null) {
        // 默认使用okhttp
         callFactory = new OkHttpClient();
    }
    
    Executor callbackExecutor = this.callbackExecutor;
    if (callbackExecutor == null) {
        // Android默认的callbackExecutor
        callbackExecutor = platform.defaultCallbackExecutor();
    }
    
    // Make a defensive copy of the adapters and add the defaultCall adapter.
    List<CallAdapter.Factory> callAdapterFactories = new ArrayList<>(this.callAdapterFactories);
    // 添加默认适配器工厂在集合尾部
    callAdapterFactories.addAll(platform.defaultCallAdapterFactorisca  llbackExecutor));
    
    // Make a defensive copy of the converters.
    List<Converter.Factory> converterFactories = new ArrayList<>(
        1 + this.converterFactories.size() + platform.defaultConverterFactoriesSize());
    // Add the built-in converter factory first. This prevents overriding its behavior but also
    // ensures correct behavior when using converters thatconsumeall types.
    converterFactories.add(new BuiltInConverters());
    converterFactories.addAll(this.converterFactories);
    converterFactories.addAll(platform.defaultConverterFactories();
    
    return new Retrofit(callFactory, baseUrl, unmodifiableList(converterFactories),
        unmodifiableList(callAdapterFactories), callbackExecutor, validateEagerly);
        
}
```

可以看到，最终我们在Builder类中看到的6大核心对象都已经配置到Retrofit对象中了。

## 创建网络请求接口实例过程

retrofit.create()使用了外观模式和代理模式创建了网络请求的接口实例，我们分析下create方法。

```
public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
        // 判断是否需要提前缓存ServiceMethod对象
        eagerlyValidateMethods(service);
    }
    
    // 使用动态代理拿到请求接口所有注解配置后，创建网络请求接口实例
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new  Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();
          private final Object[] emptyArgs = new Object[0];

          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            return loadServiceMethod(method).invoke(args != null ? args : emptyArgs);
          }
    });
 }

private void eagerlyValidateMethods(Class<?> service) {

  Platform platform = Platform.get();
  for (Method method : service.getDeclaredMethods()) {
    if (!platform.isDefaultMethod(method)) {
      loadServiceMethod(method);
    }
  }
}
```

继续看看loadServiceMethod的内部流程

```
ServiceMethod<?> loadServiceMethod(Method method) {

    ServiceMethod<?> result = serviceMethodCache.get(method);
    if (result != null) return result;

    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
            // 解析注解配置得到了ServiceMethod
            result = ServiceMethod.parseAnnotations(this, method);
            // 可以看到，最终加入到ConcurrentHashMap缓存中
            serviceMethodCache.put(method, result);
      }
    }
    return result;
}


abstract class ServiceMethod<T> {
  static <T> ServiceMethod<T> parseAnnotations(Retrofit retrofit, Method   method) {
        // 通过RequestFactory解析注解配置（工厂模式、内部使用了建造者模式）
        RequestFactory requestFactory = RequestFactory.parseAnnotations(retrofit, method);
    
        Type returnType = method.getGenericReturnType();
        if (Utils.hasUnresolvableType(returnType)) {
          throw methodError(method,
              "Method return type must not include a type variable or wildcard: %s", returnType);
        }
        if (returnType == void.class) {
          throw methodError(method, "Service methods cannot return void.");
        }
    
        // 最终是通过HttpServiceMethod构建的请求方法
        return HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory);
    }

    abstract T invoke(Object[] args);
}
```

### 请求构造核心流程

根据RequestFactory#Builder构造方法和parseAnnotations方法的源码，可知的它的作用就是用来解析注解配置的。

```
Builder(Retrofit retrofit, Method method) {
    this.retrofit = retrofit;
    this.method = method;
    // 获取网络请求接口方法里的注释
    this.methodAnnotations = method.getAnnotations();
    // 获取网络请求接口方法里的参数类型       
    this.parameterTypes = method.getGenericParameterTypes();
    // 获取网络请求接口方法里的注解内容    
    this.parameterAnnotationsArray = method.getParameterAnnotations();
}
```

接着看HttpServiceMethod.parseAnnotations()的内部流程。

```
static <ResponseT, ReturnT> HttpServiceMethod<ResponseT, ReturnT> parseAnnotations(
      Retrofit retrofit, Method method, RequestFactory requestFactory) {
        
    //1.根据网络请求接口方法的返回值和注解类型，
    // 从Retrofit对象中获取对应的网络请求适配器
    CallAdapter<ResponseT, ReturnT> callAdapter = createCallAdapter(retrofit,method);
    
    // 得到响应类型
    Type responseType = callAdapter.responseType();
    
    ...

    //2.根据网络请求接口方法的返回值和注解类型从Retrofit对象中获取对应的数据转换器 
    Converter<ResponseBody, ResponseT>responseConverter =
        createResponseConverter(retrofit,method, responseType);

    okhttp3.Call.Factory callFactory = retrofit.callFactory;
    
    return newHttpServiceMethod<>(requestFactory, callFactory, callAdapter,responseConverter);
}
```

#### createCallAdapter(retrofit, method)

```
private static <ResponseT, ReturnT> CallAdapter<ResponseT, ReturnT>     createCallAdapter(
      Retrofit retrofit, Method method) {
      
    // 获取网络请求接口里方法的返回值类型
    Type returnType = method.getGenericReturnType();
    
    // 获取网络请求接口接口里的注解
    Annotation[] annotations = method.getAnnotations();
    try {
      //noinspection unchecked
      return (CallAdapter<ResponseT, ReturnT>)  retrofit.callAdapter(returnType, annotations);
    } catch (RuntimeException e) { // Wide exception range because factories are user code.
      throw methodError(method, e, "Unable to create call adapter for %s", returnType);
    }
}
  
public CallAdapter<?, ?> callAdapter(Type returnType, Annotation[] annotations) {
    return nextCallAdapter(null, returnType, annotations);
}

public CallAdapter<?, ?> nextCallAdapter(@Nullable CallAdapter.Factory skipPast, Type returnType,
  Annotation[] annotations) {
    ...

    int start = callAdapterFactories.indexOf(skipPast) + 1;
    // 遍历 CallAdapter.Factory 集合寻找合适的工厂
    for (int i = start, count = callAdapterFactories.size(); i <count; i++) {
        CallAdapter<?, ?> adapter = callAdapterFactories.get(i).get(returnType, annotations, this);
        if (adapter != null) {
          return adapter;
        }
    }
}
```

#### createResponseConverter(Retrofit retrofit, Method method, Type responseType)

```
 private static <ResponseT> Converter<ResponseBody, ResponseT>  createResponseConverter(
     Retrofit retrofit, Method method, Type responseType) {
   Annotation[] annotations = method.getAnnotations();
   try {
     return retrofit.responseBodyConverter(responseType,annotations);
   } catch (RuntimeException e) { // Wide exception range because    factories are user code.
     throw methodError(method, e, "Unable to create converter for%s",   responseType);
   }
}

public <T> Converter<ResponseBody, T> responseBodyConverter(Type type, Annotation[] annotations) {
    return nextResponseBodyConverter(null, type, annotations);
}

public <T> Converter<ResponseBody, T> nextResponseBodyConverter(
  @Nullable Converter.Factory skipPast, Type type, Annotation[] annotations) {
...

int start = converterFactories.indexOf(skipPast) + 1;
// 遍历 Converter.Factory 集合并寻找合适的工厂, 这里是GsonResponseBodyConverter
for (int i = start, count = converterFactories.size(); i < count; i++) {
  Converter<ResponseBody, ?> converter =
      converterFactories.get(i).responseBodyConverter(type, annotations, this);
  if (converter != null) {
    //noinspection unchecked
    return (Converter<ResponseBody, T>) converter;
  }
}
```

#### 执行HttpServiceMethod的invoke方法

```
@Override ReturnT invoke(Object[] args) {
    return callAdapter.adapt(
        new OkHttpCall<>(requestFactory, args, callFactory, responseConverter));
}
```

最终在adapt中创建了一个ExecutorCallbackCall对象，它是一个装饰者，而在它内部真正去执行网络请求的还是OkHttpCall。

## 创建网络请求接口类实例并执行请求过程

### service.listRepos()

```
1、Call<List<Repo>> repos = service.listRepos("octocat");
```

service对象是动态代理对象Proxy.newProxyInstance()，当调用getCall()时会被 它拦截，然后调用自身的InvocationHandler#invoke()，得到最终的Call对象。

### 同步执行流程 repos.execute()

```
@Override public Response<T> execute() throws IOException {
    okhttp3.Call call;

    synchronized (this) {
      if (executed) throw new IllegalStateException("Already executed.");
      executed = true;

      if (creationFailure != null) {
        if (creationFailure instanceof IOException) {
          throw (IOException) creationFailure;
        } else if (creationFailure instanceof RuntimeException) {
          throw (RuntimeException) creationFailure;
        } else {
          throw (Error) creationFailure;
        }
      }

      call = rawCall;
      if (call == null) {
        try {
          // 创建一个OkHttp的Request对象请求
          call = rawCall = createRawCall();
        } catch (IOException | RuntimeException | Error e) {
          throwIfFatal(e); //  Do not assign a fatal error to     creationFailure.
          creationFailure = e;
          throw e;
        }
      }
    }

    if (canceled) {
      call.cancel();
    }

    // 调用OkHttpCall的execute()发送网络请求（同步），
    // 并解析网络请求返回的数据
    return parseResponse(call.execute());
}


private okhttp3.Call createRawCall() throws IOException {
    // 创建 一个okhttp3.Request
    okhttp3.Call call =
    callFactory.newCall(requestFactory.create(args));
    if (call == null) {
      throw new NullPointerException("Call.Factory returned null.");
    }
    return call;
}


Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException {
    ResponseBody rawBody = rawResponse.body(); 
    
    // Remove the body's source (the only stateful object) so we can   pass the response along.
    rawResponse = rawResponse.newBuilder()
        .body(new NoContentResponseBody(rawBody.contentType(), rawBody.contentLength()))
        .build();    
    
    // 根据响应返回的状态码进行处理    
    int code = rawResponse.code();
    if (code < 200 || code >= 300) {
      try {
        // Buffer the entire body to avoid future I/O.
        ResponseBody bufferedBody = Utils.buffer(rawBody);
        return Response.error(bufferedBody, rawResponse);
      } finally {
        rawBody.close();
      }
    }    
    if (code == 204 || code == 205) {
      rawBody.close();
      return Response.success(null, rawResponse);
    }    
    
    
    ExceptionCatchingResponseBody catchingBody = new ExceptionCatchingResponseBody(rawBody);
    try {
      // 将响应体转为Java对象
      T body = responseConverter.convert(catchingBody);
      
      return Response.success(body, rawResponse);
    } catch (RuntimeException e) {
      // If the underlying source threw an exception, propagate that     rather than indicating it was
      // a runtime exception.
      catchingBody.throwIfCaught();
      throw e;
    }
}
```

### 异步请求流程 reponse.enqueque

```
@Override 
public void enqueue(final Callback<T> callback) {

    // 使用静态代理 delegate进行异步请求 
    delegate.enqueue(new Callback<T>() {

      @Override 
      public void onResponse(Call<T> call, finalResponse<T>response) {
        // 线程切换，在主线程显示结果
        callbackExecutor.execute(new Runnable() {
            @Override 
             public void run() {
            if (delegate.isCanceled()) {
              callback.onFailure(ExecutorCallbackCall.this, newIOException("Canceled"));
            } else {
              callback.onResponse(ExecutorCallbackCall.this,respons);
            }
          }
        });
      }
      @Override 
      public void onFailure(Call<T> call, final Throwable t) {
        callbackExecutor.execute(new Runnable() {
          @Override public void run() {
            callback.onFailure(ExecutorCallbackCall.this, t);
          }
        });
      }
    });
}
```

看看 delegate.enqueue 内部流程。

```
@Override 
public void enqueue(final Callback<T> callback) {
   
    okhttp3.Call call;
    Throwable failure;

    synchronized (this) {
      if (executed) throw new IllegalStateException("Already executed.");
      executed = true;

      call = rawCall;
      failure = creationFailure;
      if (call == null && failure == null) {
        try {
          // 创建OkHttp的Request对象，再封装成OkHttp.call
          // 方法同发送同步请求，此处上面已分析
          call = rawCall = createRawCall(); 
        } catch (Throwable t) {
          failure = creationFailure = t;
        }
      }

@Override public void enqueue(final Callback<T> callback) {
  checkNotNull(callback, "callback == null");

  okhttp3.Call call;
  Throwable failure;

  ...

  call.enqueue(new okhttp3.Callback() {
    @Override public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse) {
      Response<T> response;
      try {
        // 此处上面已分析
        response = parseResponse(rawResponse);
      } catch (Throwable e) {
        throwIfFatal(e);
        callFailure(e);
        return;
      }

      try {
        callback.onResponse(OkHttpCall.this, response);
      } catch (Throwable t) {
        t.printStackTrace();
      }
    }

    @Override public void onFailure(okhttp3.Call call, IOException e) {
      callFailure(e);
    }

    private void callFailure(Throwable e) {
      try {
        callback.onFailure(OkHttpCall.this, e);
      } catch (Throwable t) {
        t.printStackTrace();
      }
    }
  });
}
```

## Retrofit源码流程图

建议大家自己主动配合着Retrofit最新版的源码一步步去彻底地认识它，只有这样，你才能看到它真实的内心，附上一张Retrofit源码流程图，要注意的是，这是V2.5之前版本的流程，但是，在看完上面的源码分析后，我们知道，主体流程是没有变化的。

![image](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ab58d6242cc4fae96cd73eb198e325a~tplv-k3u1fbpfcp-zoom-1.image)



> 从本质上来说，Retrofit虽然只是一个RESTful 的HTTP 网络请求框架的封装库。但是，它内部通过 大量的设计模式 封装了 OkHttp，让使用者感到它非常简洁、易懂。它内部主要是用动态代理的方式，动态将网络请求接口的注解解析成HTTP请求，最后执行请求的过程。

# Glide

## 基本使用流程

Glide最基本的使用流程就是下面这行代码，其它所有扩展的额外功能都是以其建造者链式调用的基础上增加的。

```
GlideApp.with(context).load(url).into(iv);
```

其中的GlideApp是注解处理器自动生成的，要使用GlideApp，必须先配置应用的AppGlideModule模块，里面可以为空配置，也可以根据实际情况添加指定配置。

```
@GlideModule
public class MyAppGlideModule extends AppGlideModule {

    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        // 实际使用中根据情况可以添加如下配置
        <!--builder.setDefaultRequestOptions(new RequestOptions().format(DecodeFormat.PREFER_RGB_565));-->
        <!--int memoryCacheSizeBytes = 1024 * 1024 * 20;-->
        <!--builder.setMemoryCache(new LruResourceCache(memoryCacheSizeBytes));-->
        <!--int bitmapPoolSizeBytes = 1024 * 1024 * 30;-->
        <!--builder.setBitmapPool(new LruBitmapPool(bitmapPoolSizeBytes));-->
        <!--int diskCacheSizeBytes = 1024 * 1024 * 100;-->
        <!--builder.setDiskCache(new InternalCacheDiskCacheFactory(context, diskCacheSizeBytes));-->
    }
}
```

接下来，本文将针对Glide的最新源码版本V4.8.0对Glide加载网络图片的流程进行详细地分析与讲解，力争做到让读者朋友们知其然也知其所以然。

## GlideApp.with(context)源码详解

首先，用这份Glide框架图让我们对Glide的总体框架有一个初步的了解。

![image](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2478933d44534c2fabb6466049b9f3b7~tplv-k3u1fbpfcp-zoom-1.image)

从GlideApp.with这行代码开始，内部主线执行流程如下。

### GlideApp#with

```
return (GlideRequests) Glide.with(context);
```

### Glide#with

```
return getRetriever(context).get(context);

return Glide.get(context).getRequestManagerRetriever();

// 外部使用了双重检锁的同步方式确保同一时刻只执一次Glide的初始化
checkAndInitializeGlide(context);

initializeGlide(context);

// 最终执行到Glide的另一个重载方法
initializeGlide(context, new GlideBuilder());

@SuppressWarnings("deprecation")
  private static void initializeGlide(@NonNull Context   context, @NonNull GlideBuilder builder) {
    Context applicationContext =     context.getApplicationContext();
    // 1、获取前面应用中带注解的GlideModule
    GeneratedAppGlideModule annotationGeneratedModule =     getAnnotationGeneratedGlideModules();
    // 2、如果GlideModule为空或者可配置manifest里面的标志为true，则获取manifest里面
    // 配置的GlideModule模块（manifestModules）。
    List<com.bumptech.glide.module.GlideModule>     manifestModules = Collections.emptyList();
    if (annotationGeneratedModule == null ||     annotationGeneratedModule.isManifestParsingEnabled(    )) {
      manifestModules = new   ManifestParser(applicationContext).parse();
    }

    ...

    RequestManagerRetriever.RequestManagerFactory     factory =
        annotationGeneratedModule != null
            ? annotationGeneratedModule.getRequestManag    erFactory() : null;
    builder.setRequestManagerFactory(factory);
    for (com.bumptech.glide.module.GlideModule module :     manifestModules) {
      module.applyOptions(applicationContext, builder);
    }
    if (annotationGeneratedModule != null) {
      annotationGeneratedModule.applyOptions(applicatio  nContext, builder);
    }
    // 3、初始化各种配置信息
    Glide glide = builder.build(applicationContext);
    // 4、把manifestModules以及annotationGeneratedModule里面的配置信息放到builder
    // 里面（applyOptions）替换glide默认组件（registerComponents）
    for (com.bumptech.glide.module.GlideModule module :     manifestModules) {
      module.registerComponents(applicationContext,   glide, glide.registry);
    }
    if (annotationGeneratedModule != null) {
      annotationGeneratedModule.registerComponents(appl  icationContext, glide, glide.registry);
    }
    applicationContext.registerComponentCallbacks(glide    );
    Glide.glide = glide;
}
```

### GlideBuilder#build

```
@NonNull
  Glide build(@NonNull Context context) {
    // 创建请求图片线程池sourceExecutor
    if (sourceExecutor == null) {
      sourceExecutor =   GlideExecutor.newSourceExecutor();
    }

    // 创建硬盘缓存线程池diskCacheExecutor
    if (diskCacheExecutor == null) {
      diskCacheExecutor =   GlideExecutor.newDiskCacheExecutor();
    }

    // 创建动画线程池animationExecutor
    if (animationExecutor == null) {
      animationExecutor =   GlideExecutor.newAnimationExecutor();
    }

    if (memorySizeCalculator == null) {
      memorySizeCalculator = new   MemorySizeCalculator.Builder(context).build();
    }

    if (connectivityMonitorFactory == null) {
      connectivityMonitorFactory = new   DefaultConnectivityMonitorFactory();
    }

    if (bitmapPool == null) {
      // 依据设备的屏幕密度和尺寸设置各种pool的size
      int size =   memorySizeCalculator.getBitmapPoolSize();
      if (size > 0) {
        // 创建图片线程池LruBitmapPool，缓存所有被释放的bitmap
        // 缓存策略在API大于19时，为SizeConfigStrategy，小于为AttributeStrategy。
        // 其中SizeConfigStrategy是以bitmap的size和config为key，value为bitmap的HashMap
        bitmapPool = new LruBitmapPool(size);
      } else {
        bitmapPool = new BitmapPoolAdapter();
      }
    }

    // 创建对象数组缓存池LruArrayPool，默认4M
    if (arrayPool == null) {
      arrayPool = new   LruArrayPool(memorySizeCalculator.getArrayPoolSiz  eInBytes());
    }

    // 创建LruResourceCache，内存缓存
    if (memoryCache == null) {
      memoryCache = new   LruResourceCache(memorySizeCalculator.getMemoryCa  cheSize());
    }

    if (diskCacheFactory == null) {
      diskCacheFactory = new   InternalCacheDiskCacheFactory(context);
    }

    // 创建任务和资源管理引擎（线程池，内存缓存和硬盘缓存对象）
    if (engine == null) {
      engine =
          new Engine(
              memoryCache,
              diskCacheFactory,
              diskCacheExecutor,
              sourceExecutor,
              GlideExecutor.newUnlimitedSourceExecutor(  ),
              GlideExecutor.newAnimationExecutor(),
              isActiveResourceRetentionAllowed);
    }
    
    RequestManagerRetriever requestManagerRetriever =
    new RequestManagerRetriever(requestManagerFactory);

    return new Glide(
        context,
        engine,
        memoryCache,
        bitmapPool,
        arrayPool,
        requestManagerRetriever,
        connectivityMonitorFactory,
        logLevel,
        defaultRequestOptions.lock(),
        defaultTransitionOptions);
}
```

### Glide#Glide构造方法

```
Glide(...) {
    ...
    // 注册管理任务执行对象的类(Registry)
    // Registry是一个工厂，而其中所有注册的对象都是一个工厂员工，当任务分发时，
    // 根据当前任务的性质，分发给相应员工进行处理
    registry = new Registry();
    
    ...
    
    // 这里大概有60余次的append或register员工组件（解析器、编解码器、工厂类、转码类等等组件）
    registry
    .append(ByteBuffer.class, new ByteBufferEncoder())
    .append(InputStream.class, new StreamEncoder(arrayPool))
    
    // 根据给定子类产出对应类型的target（BitmapImageViewTarget / DrawableImageViewTarget)
    ImageViewTargetFactory imageViewTargetFactory = new ImageViewTargetFactory();
    
    glideContext =
        new GlideContext(
            context,
            arrayPool,
            registry,
            imageViewTargetFactory,
            defaultRequestOptions,
            defaultTransitionOptions,
            engine,
            logLevel);
}
```

### RequestManagerRetriever#get

```
@NonNull
public RequestManager get(@NonNull Context context) {
  if (context == null) {
    throw new IllegalArgumentException("You cannot start a load on a null Context");
  } else if (Util.isOnMainThread() && !(context instanceof Application)) {
    // 如果当前线程是主线程且context不是Application走相应的get重载方法
    if (context instanceof FragmentActivity) {
      return get((FragmentActivity) context);
    } else if (context instanceof Activity) {
      return get((Activity) context);
    } else if (context instanceof ContextWrapper) {
      return get(((ContextWrapper) context).getBaseContext());
    }
  }

  // 否则直接将请求与ApplicationLifecycle关联
  return getApplicationManager(context);
}
```

这里总结一下，对于当前传入的context是application或当前线程是子线程时，请求的生命周期和ApplicationLifecycle关联，否则，context是FragmentActivity或Fragment时，在当前组件添加一个SupportFragment（SupportRequestManagerFragment），context是Activity时，在当前组件添加一个Fragment(RequestManagerFragment)。

### GlideApp#with小结

1. 初始化各式各样的配置信息（包括缓存，请求线程池，大小，图片格式等等）以及glide对象。

2. 将glide请求和application/SupportFragment/Fragment的生命周期绑定在一块。

### with方法的执行流程

![image](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a307a034fea446a486c4f988849751a0~tplv-k3u1fbpfcp-zoom-1.image)

## load(url)源码详解

### GlideRequest(RequestManager)#load

```
return (GlideRequest<Drawable>) super.load(string);

return asDrawable().load(string);

// 1、asDrawable部分
return (GlideRequest<Drawable>) super.asDrawable();

return as(Drawable.class);

// 最终返回了一个GlideRequest（RequestManager的子类）
return new GlideRequest<>(glide, this, resourceClass, context);

// 2、load部分
return (GlideRequest<TranscodeType>) super.load(string);

return loadGeneric(string);

@NonNull
private RequestBuilder<TranscodeType> loadGeneric(@Nullable Object model) {
    // model则为设置的url
    this.model = model;
    // 记录url已设置
    isModelSet = true;
    return this;
}
```

可以看到，load这部分的源码很简单，就是给GlideRequest（RequestManager）设置了要请求的mode（url），并记录了url已设置的状态。

### load方法的执行流程

![image](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33114794d81a421ea4345806826919b7~tplv-k3u1fbpfcp-zoom-1.image)

## into(iv)源码详解

真正复杂的地方要开始了。

### RequestBuilder.into

```
 @NonNull
public ViewTarget<ImageView, TranscodeType>   into(@NonNull ImageView view) {
  Util.assertMainThread();
  Preconditions.checkNotNull(view);

  RequestOptions requestOptions =     this.requestOptions;
  if (!requestOptions.isTransformationSet()
      && requestOptions.isTransformationAllowed()
      && view.getScaleType() != null) {
    // Clone in this method so that if we use this   RequestBuilder to load into a View and then
    // into a different target, we don't retain the   transformation applied based on the previous
    // View's scale type.
    switch (view.getScaleType()) {
      // 这个RequestOptions里保存了要设置的scaleType，Glide自身封装了CenterCrop、CenterInside、
      // FitCenter、CenterInside四种规格。
      case CENTER_CROP:
        requestOptions =   requestOptions.clone().optionalCenterCrop();
        break;
      case CENTER_INSIDE:
        requestOptions =   requestOptions.clone().optionalCenterInside()  ;
        break;
      case FIT_CENTER:
      case FIT_START:
      case FIT_END:
        requestOptions =   requestOptions.clone().optionalFitCenter();
        break;
      case FIT_XY:
        requestOptions =   requestOptions.clone().optionalCenterInside()  ;
        break;
      case CENTER:
      case MATRIX:
      default:
        // Do nothing.
    }
  }

  // 注意，这个transcodeClass是指的drawable或bitmap
  return into(
      glideContext.buildImageViewTarget(view,     transcodeClass),
      /*targetListener=*/ null,
      requestOptions);
}
```

### GlideContext#buildImageViewTarget

```
return imageViewTargetFactory.buildTarget(imageView, transcodeClass);
```

### ImageViewTargetFactory#buildTarget

```
@NonNull
@SuppressWarnings("unchecked")
public <Z> ViewTarget<ImageView, Z>   buildTarget(@NonNull ImageView view,
    @NonNull Class<Z> clazz) {
  // 返回展示Bimtap/Drawable资源的目标对象
  if (Bitmap.class.equals(clazz)) {
    return (ViewTarget<ImageView, Z>) new   BitmapImageViewTarget(view);
  } else if (Drawable.class.isAssignableFrom(clazz))     {
    return (ViewTarget<ImageView, Z>) new   DrawableImageViewTarget(view);
  } else {
    throw new IllegalArgumentException(
        "Unhandled class: " + clazz + ", try   .as*(Class).transcode(ResourceTranscoder)");
  }
}
```

可以看到，Glide内部只维护了两种target，一种是BitmapImageViewTarget，另一种则是DrawableImageViewTarget，接下来继续深入。

### RequestBuilder#into

```
private <Y extends Target<TranscodeType>> Y into(
      @NonNull Y target,
      @Nullable RequestListener<TranscodeType>   targetListener,
      @NonNull RequestOptions options) {
    Util.assertMainThread();
    Preconditions.checkNotNull(target);
    if (!isModelSet) {
      throw new IllegalArgumentException("You must call   #load() before calling #into()");
    }

    options = options.autoClone();
    // 分析1.建立请求
    Request request = buildRequest(target,     targetListener, options);

    Request previous = target.getRequest();
    if (request.isEquivalentTo(previous)
        && !isSkipMemoryCacheWithCompletePreviousReques    t(options, previous)) {
      request.recycle();
      // If the request is completed, beginning again   will ensure the result is re-delivered,
      // triggering RequestListeners and Targets. If   the request is failed, beginning again will
      // restart the request, giving it another chance   to complete. If the request is already
      // running, we can let it continue running   without interruption.
      if (!Preconditions.checkNotNull(previous).isRunni  ng()) {
        // Use the previous request rather than the new     one to allow for optimizations like skipping
        // setting placeholders, tracking and     un-tracking Targets, and obtaining View     dimensions
        // that are done in the individual Request.
        previous.begin();
      }
      return target;
    }
    
    requestManager.clear(target);
    target.setRequest(request);
    // 分析2.真正追踪请求的地方
    requestManager.track(target, request);

    return target;
}

// 分析1
private Request buildRequest(
      Target<TranscodeType> target,
      @Nullable RequestListener<TranscodeType>   targetListener,
      RequestOptions requestOptions) {
    return buildRequestRecursive(
        target,
        targetListener,
        /*parentCoordinator=*/ null,
        transitionOptions,
        requestOptions.getPriority(),
        requestOptions.getOverrideWidth(),
        requestOptions.getOverrideHeight(),
        requestOptions);
}

// 分析1
private Request buildRequestRecursive(
      Target<TranscodeType> target,
      @Nullable RequestListener<TranscodeType>   targetListener,
      @Nullable RequestCoordinator parentCoordinator,
      TransitionOptions<?, ? super TranscodeType>   transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      RequestOptions requestOptions) {

    // Build the ErrorRequestCoordinator first if     necessary so we can update parentCoordinator.
    ErrorRequestCoordinator errorRequestCoordinator =     null;
    if (errorBuilder != null) {
      // 创建errorRequestCoordinator（异常处理对象）
      errorRequestCoordinator = new   ErrorRequestCoordinator(parentCoordinator);
      parentCoordinator = errorRequestCoordinator;
    }

    // 递归建立缩略图请求
    Request mainRequest =
        buildThumbnailRequestRecursive(
            target,
            targetListener,
            parentCoordinator,
            transitionOptions,
            priority,
            overrideWidth,
            overrideHeight,
            requestOptions);

    if (errorRequestCoordinator == null) {
      return mainRequest;
    }

    ...
    
    Request errorRequest =     errorBuilder.buildRequestRecursive(
        target,
        targetListener,
        errorRequestCoordinator,
        errorBuilder.transitionOptions,
        errorBuilder.requestOptions.getPriority(),
        errorOverrideWidth,
        errorOverrideHeight,
        errorBuilder.requestOptions);
    errorRequestCoordinator.setRequests(mainRequest,     errorRequest);
    return errorRequestCoordinator;
}

// 分析1
private Request buildThumbnailRequestRecursive(
      Target<TranscodeType> target,
      RequestListener<TranscodeType> targetListener,
      @Nullable RequestCoordinator parentCoordinator,
      TransitionOptions<?, ? super TranscodeType> transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight,
      RequestOptions requestOptions) {
    if (thumbnailBuilder != null) {
      // Recursive case: contains a potentially recursive thumbnail request builder.
      
      ...

      ThumbnailRequestCoordinator coordinator = new ThumbnailRequestCoordinator(parentCoordinator);
      // 获取一个正常请求对象
      Request fullRequest =
          obtainRequest(
              target,
              targetListener,
              requestOptions,
              coordinator,
              transitionOptions,
              priority,
              overrideWidth,
              overrideHeight);
      isThumbnailBuilt = true;
      // Recursively generate thumbnail requests.
      // 使用递归的方式建立一个缩略图请求对象
      Request thumbRequest =
          thumbnailBuilder.buildRequestRecursive(
              target,
              targetListener,
              coordinator,
              thumbTransitionOptions,
              thumbPriority,
              thumbOverrideWidth,
              thumbOverrideHeight,
              thumbnailBuilder.requestOptions);
      isThumbnailBuilt = false;
      // coordinator（ThumbnailRequestCoordinator）是作为两者的协调者，
      // 能够同时加载缩略图和正常的图的请求
      coordinator.setRequests(fullRequest, thumbRequest);
      return coordinator;
    } else if (thumbSizeMultiplier != null) {
      // Base case: thumbnail multiplier generates a thumbnail request, but cannot recurse.
      // 当设置了缩略的比例thumbSizeMultiplier(0 ~  1)时，
      // 不需要递归建立缩略图请求
      ThumbnailRequestCoordinator coordinator = new ThumbnailRequestCoordinator(parentCoordinator);
      Request fullRequest =
          obtainRequest(
              target,
              targetListener,
              requestOptions,
              coordinator,
              transitionOptions,
              priority,
              overrideWidth,
              overrideHeight);
      RequestOptions thumbnailOptions = requestOptions.clone()
          .sizeMultiplier(thumbSizeMultiplier);

      Request thumbnailRequest =
          obtainRequest(
              target,
              targetListener,
              thumbnailOptions,
              coordinator,
              transitionOptions,
              getThumbnailPriority(priority),
              overrideWidth,
              overrideHeight);

      coordinator.setRequests(fullRequest, thumbnailRequest);
      return coordinator;
    } else {
      // Base case: no thumbnail.
      // 没有缩略图请求时，直接获取一个正常图请求
      return obtainRequest(
          target,
          targetListener,
          requestOptions,
          parentCoordinator,
          transitionOptions,
          priority,
          overrideWidth,
          overrideHeight);
    }
}

private Request obtainRequest(
      Target<TranscodeType> target,
      RequestListener<TranscodeType> targetListener,
      RequestOptions requestOptions,
      RequestCoordinator requestCoordinator,
      TransitionOptions<?, ? super TranscodeType>   transitionOptions,
      Priority priority,
      int overrideWidth,
      int overrideHeight) {
    // 最终实际返回的是一个SingleRequest对象（将制定的资源加载进对应的Target
    return SingleRequest.obtain(
        context,
        glideContext,
        model,
        transcodeClass,
        requestOptions,
        overrideWidth,
        overrideHeight,
        priority,
        target,
        targetListener,
        requestListeners,
        requestCoordinator,
        glideContext.getEngine(),
        transitionOptions.getTransitionFactory());
}
```

从上源码分析可知，我们在分析1处的buildRequest()方法里建立了请求，且最多可同时进行缩略图和正常图的请求，最后，调用了requestManager.track(target, request)方法，接着看看track里面做了什么。

### RequestManager#track

```
// 分析2
void track(@NonNull Target<?> target, @NonNull Request request) {
    // 加入一个target目标集合(Set)
    targetTracker.track(target);
    
    requestTracker.runRequest(request);
}
```

### RequestTracker#runRequest

```
/**
* Starts tracking the given request.
*/
// 分析2
public void runRequest(@NonNull Request request) {
    requests.add(request);
    if (!isPaused) {
      // 如果不是暂停状态则开始请求
      request.begin();
    } else {
      request.clear();
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        Log.v(TAG, "Paused, delaying request");
      }
      // 否则清空请求，加入延迟请求队列（为了对这些请求维持一个强引用，使用了ArrayList实现）
      pendingRequests.add(request);
    }
}
```

### SingleRequest#begin

```
// 分析2
@Override
public void begin() {
  
  ...
  
  if (model == null) {
  
    ...
    // model（url）为空，回调加载失败
    onLoadFailed(new GlideException("Received null   model"), logLevel);
    return;
  }

  if (status == Status.RUNNING) {
    throw new IllegalArgumentException("Cannot   restart a running request");
  }

 
  if (status == Status.COMPLETE) {
    onResourceReady(resource,   DataSource.MEMORY_CACHE);
    return;
  }

  status = Status.WAITING_FOR_SIZE;
  if (Util.isValidDimensions(overrideWidth, overrideHeight)) {
    // 当使用override() API为图片指定了一个固定的宽高时直接执行onSizeReady，
    // 最终的核心处理位于onSizeReady
    onSizeReady(overrideWidth, overrideHeight);
  } else {
    // 根据imageView的宽高算出图片的宽高，最终也会走到onSizeReady
    target.getSize(this);
  }

  if ((status == Status.RUNNING || status ==     Status.WAITING_FOR_SIZE)
      && canNotifyStatusChanged()) {
    // 预先加载设置的缩略图
    target.onLoadStarted(getPlaceholderDrawable());
  }
  if (IS_VERBOSE_LOGGABLE) {
    logV("finished run method in " +   LogTime.getElapsedMillis(startTime));
  }
}
```

从requestManager.track(target, request)开始，最终会执行到SingleRequest#begin()方法的onSizeReady，可以猜到（因为后面只做了预加载缩略图的处理），真正的请求就是从这里开始的，咱们进去一探究竟~

### SingleRequest#onSizeReady

```
// 分析2
@Override
public void onSizeReady(int width, int height) {
  stateVerifier.throwIfRecycled();
  
  ...
  
  status = Status.RUNNING;

  float sizeMultiplier =     requestOptions.getSizeMultiplier();
  this.width = maybeApplySizeMultiplier(width,     sizeMultiplier);
  this.height = maybeApplySizeMultiplier(height,     sizeMultiplier);

  ...
  
  // 根据给定的配置进行加载，engine是一个负责加载、管理活跃和缓存资源的引擎类
  loadStatus = engine.load(
      glideContext,
      model,
      requestOptions.getSignature(),
      this.width,
      this.height,
      requestOptions.getResourceClass(),
      transcodeClass,
      priority,
      requestOptions.getDiskCacheStrategy(),
      requestOptions.getTransformations(),
      requestOptions.isTransformationRequired(),
      requestOptions.isScaleOnlyOrNoTransform(),
      requestOptions.getOptions(),
      requestOptions.isMemoryCacheable(),
      requestOptions.getUseUnlimitedSourceGeneratorsP    ool(),
      requestOptions.getUseAnimationPool(),
      requestOptions.getOnlyRetrieveFromCache(),
      this);

  ...
}
```

终于看到Engine类了，感觉距离成功不远了，继续~

### Engine#load

```
public <R> LoadStatus load(
    GlideContext glideContext,
    Object model,
    Key signature,
    int width,
    int height,
    Class<?> resourceClass,
    Class<R> transcodeClass,
    Priority priority,
    DiskCacheStrategy diskCacheStrategy,
    Map<Class<?>, Transformation<?>> transformations,
    boolean isTransformationRequired,
    boolean isScaleOnlyOrNoTransform,
    Options options,
    boolean isMemoryCacheable,
    boolean useUnlimitedSourceExecutorPool,
    boolean useAnimationPool,
    boolean onlyRetrieveFromCache,
    ResourceCallback cb) {
  
  ...

  // 先从弱引用中查找，如果有的话回调onResourceReady并直接返回
  EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
  if (active != null) {
    cb.onResourceReady(active,   DataSource.MEMORY_CACHE);
    if (VERBOSE_IS_LOGGABLE) {
      logWithTimeAndKey("Loaded resource from active     resources", startTime, key);
    }
    return null;
  }

  // 没有再从内存中查找,有的话会取出并放到ActiveResources（内部维护的弱引用缓存map）里面
  EngineResource<?> cached = loadFromCache(key,     isMemoryCacheable);
  if (cached != null) {
    cb.onResourceReady(cached,   DataSource.MEMORY_CACHE);
    if (VERBOSE_IS_LOGGABLE) {
      logWithTimeAndKey("Loaded resource from cache",     startTime, key);
    }
    return null;
  }

  EngineJob<?> current = jobs.get(key,     onlyRetrieveFromCache);
  if (current != null) {
    current.addCallback(cb);
    if (VERBOSE_IS_LOGGABLE) {
      logWithTimeAndKey("Added to existing load",     startTime, key);
    }
    return new LoadStatus(cb, current);
  }

  // 如果内存中没有，则创建engineJob（decodejob的回调类，管理下载过程以及状态）
  EngineJob<R> engineJob =
      engineJobFactory.build(
          key,
          isMemoryCacheable,
          useUnlimitedSourceExecutorPool,
          useAnimationPool,
          onlyRetrieveFromCache);

  // 创建解析工作对象
  DecodeJob<R> decodeJob =
      decodeJobFactory.build(
          glideContext,
          model,
          key,
          signature,
          width,
          height,
          resourceClass,
          transcodeClass,
          priority,
          diskCacheStrategy,
          transformations,
          isTransformationRequired,
          isScaleOnlyOrNoTransform,
          onlyRetrieveFromCache,
          options,
          engineJob);

  // 放在Jobs内部维护的HashMap中
  jobs.put(key, engineJob);

  // 关注点8 后面分析会用到
  // 注册ResourceCallback接口
  engineJob.addCallback(cb);
  // 内部开启线程去请求
  engineJob.start(decodeJob);

  if (VERBOSE_IS_LOGGABLE) {
    logWithTimeAndKey("Started new load", startTime,   key);
  }
  return new LoadStatus(cb, engineJob);
}

public void start(DecodeJob<R> decodeJob) {
    this.decodeJob = decodeJob;
    // willDecodeFromCache方法内部根据不同的阶段stage，如果是RESOURCE_CACHE/DATA_CACHE则返回true，使用diskCacheExecutor，否则调用getActiveSourceExecutor，内部会根据相应的条件返回sourceUnlimitedExecutor/animationExecutor/sourceExecutor
    GlideExecutor executor =   
    decodeJob.willDecodeFromCache()
        ? diskCacheExecutor
        : getActiveSourceExecutor();
    executor.execute(decodeJob);
}
```

可以看到，最终Engine(引擎)类内部会执行到自身的start方法，它会根据不同的配置采用不同的线程池使用diskCacheExecutor/sourceUnlimitedExecutor/animationExecutor/sourceExecutor来执行最终的解码任务decodeJob。

### DecodeJob#run

```
runWrapped();

private void runWrapped() {
    switch (runReason) {
      case INITIALIZE:
        stage = getNextStage(Stage.INITIALIZE);
        // 关注点1
        currentGenerator = getNextGenerator();
        // 关注点2 内部会调用相应Generator的startNext()
        runGenerators();
        break;
      case SWITCH_TO_SOURCE_SERVICE:
        runGenerators();
        break;
      case DECODE_DATA:
        // 关注点3 将获取的数据解码成对应的资源
        decodeFromRetrievedData();
        break;
      default:
        throw new IllegalStateException("Unrecognized     run reason: " + runReason);
    }
}

// 关注点1，完整情况下，会异步依次生成这里的ResourceCacheGenerator、DataCacheGenerator和SourceGenerator对象，并在之后执行其中的startNext()
private DataFetcherGenerator getNextGenerator() {
    switch (stage) {
      case RESOURCE_CACHE:
        return new ResourceCacheGenerator(decodeHelper, this);
      case DATA_CACHE:
        return new DataCacheGenerator(decodeHelper, this);
      case SOURCE:
        return new SourceGenerator(decodeHelper, this);
      case FINISHED:
        return null;
      default:
        throw new IllegalStateException("Unrecognized     stage: " + stage);
    }
}
```

### SourceGenerator#startNext

```
// 关注点2
@Override
public boolean startNext() {
  // dataToCache数据不为空的话缓存到硬盘（第一执行该方法是不会调用的）
  if (dataToCache != null) {
    Object data = dataToCache;
    dataToCache = null;
    cacheData(data);
  }

  if (sourceCacheGenerator != null &&     sourceCacheGenerator.startNext()) {
    return true;
  }
  sourceCacheGenerator = null;

  loadData = null;
  boolean started = false;
  while (!started && hasNextModelLoader()) {
    // 关注点4 getLoadData()方法内部会在modelLoaders里面找到ModelLoder对象
    // （每个Generator对应一个ModelLoader），
    // 并使用modelLoader.buildLoadData方法返回一个loadData列表
    loadData =   helper.getLoadData().get(loadDataListIndex++);
    if (loadData != null
        && (helper.getDiskCacheStrategy().isDataCache  able(loadData.fetcher.getDataSource())
        || helper.hasLoadPath(loadData.fetcher.getDat  aClass()))) {
      started = true;
      // 关注点6 通过loadData对象的fetcher对象（有关注点3的分析可知其实现类为HttpUrlFetcher）的
      // loadData方法来获取图片数据
      loadData.fetcher.loadData(helper.getPriority(),     this);
    }
  }
  return started;
}
```

### DecodeHelper#getLoadData

```
List<LoadData<?>> getLoadData() {
    if (!isLoadDataSet) {
      isLoadDataSet = true;
      loadData.clear();
      List<ModelLoader<Object, ?>> modelLoaders =   glideContext.getRegistry().getModelLoaders(model)  ;
      //noinspection ForLoopReplaceableByForEach to   improve perf
      for (int i = 0, size = modelLoaders.size(); i <   size; i++) {
        ModelLoader<Object, ?> modelLoader =     modelLoaders.get(i);
        // 注意：这里最终是通过HttpGlideUrlLoader的buildLoadData获取到实际的loadData对象
        LoadData<?> current =
            modelLoader.buildLoadData(model, width,     height, options);
        if (current != null) {
          loadData.add(current);
        }
      }
    }
    return loadData;
}
```

### HttpGlideUrlLoader#buildLoadData

```
@Override
public LoadData<InputStream> buildLoadData(@NonNull   GlideUrl model, int width, int height,
    @NonNull Options options) {
  // GlideUrls memoize parsed URLs so caching them     saves a few object instantiations and time
  // spent parsing urls.
  GlideUrl url = model;
  if (modelCache != null) {
    url = modelCache.get(model, 0, 0);
    if (url == null) {
      // 关注点5
      modelCache.put(model, 0, 0, model);
      url = model;
    }
  }
  int timeout = options.get(TIMEOUT);
  // 注意，这里创建了一个DataFetcher的实现类HttpUrlFetcher
  return new LoadData<>(url, new HttpUrlFetcher(url,     timeout));
}

// 关注点5
public void put(A model, int width, int height, B value) {
    ModelKey<A> key = ModelKey.get(model, width,     height);
    // 最终是通过LruCache来缓存对应的值，key是一个ModelKey对象（由model、width、height三个属性组成）
    cache.put(key, value);
}
```

从这里的分析，我们明白了HttpUrlFetcher实际上就是最终的请求执行者，而且，我们知道了Glide会使用LruCache来对解析后的url来进行缓存，以便后续可以省去解析url的时间。

### HttpUrlFetcher#loadData

```
@Override
public void loadData(@NonNull Priority priority,
    @NonNull DataCallback<? super InputStream>   callback) {
  long startTime = LogTime.getLogTime();
  try {
    // 关注点6
    // loadDataWithRedirects内部是通过HttpURLConnection网络请求数据
    InputStream result =   loadDataWithRedirects(glideUrl.toURL(), 0, null,   glideUrl.getHeaders());
    // 请求成功回调onDataReady()
    callback.onDataReady(result);
  } catch (IOException e) {
    if (Log.isLoggable(TAG, Log.DEBUG)) {
      Log.d(TAG, "Failed to load data for url", e);
    }
    callback.onLoadFailed(e);
  } finally {
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
      Log.v(TAG, "Finished http url fetcher fetch in     " + LogTime.getElapsedMillis(startTime));
    }
  }
}

private InputStream loadDataWithRedirects(URL url, int redirects, URL lastUrl,
  Map<String, String> headers) throws IOException {
    
    ...

    urlConnection.connect();
    // Set the stream so that it's closed in cleanup to avoid resource leaks. See #2352.
    stream = urlConnection.getInputStream();
    if (isCancelled) {
      return null;
    }
    final int statusCode = urlConnection.getResponseCode();
    // 只要是2xx形式的状态码则判断为成功
    if (isHttpOk(statusCode)) {
      // 从urlConnection中获取资源流
      return getStreamForSuccessfulRequest(urlConnection);
    } else if (isHttpRedirect(statusCode)) {
    
      ...
      
      // 重定向请求
      return loadDataWithRedirects(redirectUrl, redirects + 1, url,   headers);
    } else if (statusCode == INVALID_STATUS_CODE) {
      throw new HttpException(statusCode);
    } else {
      throw new HttpException(urlConnection.getResponseMessage(),   statusCode);
    }
}

private InputStream getStreamForSuccessfulRequest(HttpURLConnection urlConnection)
  throws IOException {
    if (TextUtils.isEmpty(urlConnection.getContentEncoding())) {
      int contentLength = urlConnection.getContentLength();
      stream = ContentLengthInputStream.obtain(urlConnection.getInputStr  eam(), contentLength);
    } else {
      if (Log.isLoggable(TAG, Log.DEBUG)) {
        Log.d(TAG, "Got non empty content encoding: " +     urlConnection.getContentEncoding());
      }
      stream = urlConnection.getInputStream();
    }
    return stream;
}
```

在HttpUrlFetcher#loadData方法的loadDataWithRedirects里面，Glide通过原生的HttpURLConnection进行请求后，并调用getStreamForSuccessfulRequest()方法获取到了最终的图片流。

### DecodeJob#run

在我们通过HtttpUrlFetcher的loadData()方法请求得到对应的流之后，我们还必须对流进行处理得到最终我们想要的资源。这里我们回到第10步DecodeJob#run方法的关注点3处，这行代码将会对流进行解码。

```
decodeFromRetrievedData();
```

接下来，继续看看他内部的处理。

```
private void decodeFromRetrievedData() {
    if (Log.isLoggable(TAG, Log.VERBOSE)) {
      logWithTimeAndKey("Retrieved data", startFetchTime,
          "data: " + currentData
              + ", cache key: " + currentSourceKey
              + ", fetcher: " + currentFetcher);
    }
    Resource<R> resource = null;
    try {
      //  核心代码 
      // 从数据中解码得到资源
      resource = decodeFromData(currentFetcher, currentData,   currentDataSource);
    } catch (GlideException e) {
      e.setLoggingDetails(currentAttemptingKey, currentDataSource);
      throwables.add(e);
    }
    if (resource != null) {
      // 关注点8 
      // 编码和发布最终得到的Resource<Bitmap>对象
      notifyEncodeAndRelease(resource, currentDataSource);
    } else {
      runGenerators();
    }
}

 private <Data> Resource<R> decodeFromData(DataFetcher<?> fetcher, Data data,
  DataSource dataSource) throws GlideException {
    try {
      if (data == null) {
        return null;
      }
      long startTime = LogTime.getLogTime();
      // 核心代码
      // 进一步包装了解码方法
      Resource<R> result = decodeFromFetcher(data, dataSource);
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        logWithTimeAndKey("Decoded result " + result, startTime);
      }
      return result;
    } finally {
      fetcher.cleanup();
    }
}

@SuppressWarnings("unchecked")
private <Data> Resource<R> decodeFromFetcher(Data data, DataSource dataSource)
  throws GlideException {
    LoadPath<Data, ?, R> path = decodeHelper.getLoadPath((Class<Data>) data.getClass());
    // 核心代码
    // 将解码任务分发给LoadPath
    return runLoadPath(data, dataSource, path);
}

private <Data, ResourceType> Resource<R> runLoadPath(Data data, DataSource dataSource,
  LoadPath<Data, ResourceType, R> path) throws GlideException {
    Options options = getOptionsWithHardwareConfig(dataSource);
    // 将数据进一步包装
    DataRewinder<Data> rewinder =     glideContext.getRegistry().getRewinder(data);
    try {
      // ResourceType in DecodeCallback below is required for   compilation to work with gradle.
      // 核心代码
      // 将解码任务分发给LoadPath
      return path.load(
          rewinder, options, width, height, new   DecodeCallback<ResourceType>(dataSource));
    } finally {
      rewinder.cleanup();
    }
}
```

### LoadPath#load

```
public Resource<Transcode> load(DataRewinder<Data> rewinder, @NonNull Options options, int width,
  int height, DecodePath.DecodeCallback<ResourceType> decodeCallback) throws GlideException {
List<Throwable> throwables = Preconditions.checkNotNull(listPool.acquire());
try {
  // 核心代码
  return loadWithExceptionList(rewinder, options, width, height, decodeCallback, throwables);
} finally {
  listPool.release(throwables);
}
```

}

```
private Resource<Transcode> loadWithExceptionList(DataRewinder<Data> rewinder,
      @NonNull Options options,
      int width, int height, DecodePath.DecodeCallback<ResourceType>   decodeCallback,
      List<Throwable> exceptions) throws GlideException {
    Resource<Transcode> result = null;
    //noinspection ForLoopReplaceableByForEach to improve perf
    for (int i = 0, size = decodePaths.size(); i < size; i++) {
      DecodePath<Data, ResourceType, Transcode> path =   decodePaths.get(i);
      try {
        // 核心代码
        // 将解码任务又进一步分发给DecodePath的decode方法去解码
        result = path.decode(rewinder, width, height, options,     decodeCallback);
      } catch (GlideException e) {
        exceptions.add(e);
      }
      if (result != null) {
        break;
      }
    }

    if (result == null) {
      throw new GlideException(failureMessage, new   ArrayList<>(exceptions));
    }

    return result;
}
```

### DecodePath#decode

```
public Resource<Transcode> decode(DataRewinder<DataType> rewinder,     int width, int height,
      @NonNull Options options, DecodeCallback<ResourceType> callback)   throws GlideException {
    // 核心代码
    // 继续调用DecodePath的decodeResource方法去解析出数据
    Resource<ResourceType> decoded = decodeResource(rewinder, width,     height, options);
    Resource<ResourceType> transformed =     callback.onResourceDecoded(decoded);
    return transcoder.transcode(transformed, options);
}

@NonNull
private Resource<ResourceType> decodeResource(DataRewinder<DataType>   rewinder, int width,
    int height, @NonNull Options options) throws GlideException {
  List<Throwable> exceptions =     Preconditions.checkNotNull(listPool.acquire());
  try {
    // 核心代码
    return decodeResourceWithList(rewinder, width, height, options,   exceptions);
  } finally {
    listPool.release(exceptions);
  }
}

@NonNull
private Resource<ResourceType>   decodeResourceWithList(DataRewinder<DataType> rewinder, int width,
    int height, @NonNull Options options, List<Throwable> exceptions)   throws GlideException {
  Resource<ResourceType> result = null;
  //noinspection ForLoopReplaceableByForEach to improve perf
  for (int i = 0, size = decoders.size(); i < size; i++) {
    ResourceDecoder<DataType, ResourceType> decoder = decoders.get(i);
    try {
      DataType data = rewinder.rewindAndGet();
      if (decoder.handles(data, options)) {
        // 获取包装的数据
        data = rewinder.rewindAndGet();
        // 核心代码 
        // 根据DataType和ResourceType的类型分发给不同的解码器Decoder
        result = decoder.decode(data, width, height, options);
      }
    } catch (IOException | RuntimeException | OutOfMemoryError e) {
      if (Log.isLoggable(TAG, Log.VERBOSE)) {
        Log.v(TAG, "Failed to decode data for " + decoder, e);
      }
      exceptions.add(e);
    }

    if (result != null) {
      break;
    }
  }

  if (result == null) {
    throw new GlideException(failureMessage, new   ArrayList<>(exceptions));
  }
  return result;
}
```

可以看到，经过一连串的嵌套调用，最终执行到了decoder.decode()这行代码，decode是一个ResourceDecoder<DataType, ResourceType>接口（资源解码器），根据不同的DataType和ResourceType它会有不同的实现类，这里的实现类是ByteBufferBitmapDecoder，接下来让我们来看看这个解码器内部的解码流程。

### ByteBufferBitmapDecoder#decode

```
/**
 * Decodes {@link android.graphics.Bitmap Bitmaps} from {@link    java.nio.ByteBuffer ByteBuffers}.
 */
public class ByteBufferBitmapDecoder implements     ResourceDecoder<ByteBuffer, Bitmap> {
  
  ...

  @Override
  public Resource<Bitmap> decode(@NonNull ByteBuffer source, int width,   int height,
      @NonNull Options options)
      throws IOException {
    InputStream is = ByteBufferUtil.toStream(source);
    // 核心代码
    return downsampler.decode(is, width, height, options);
  }
}
```

可以看到，最终是使用了一个downsampler，它是一个压缩器，主要是对流进行解码，压缩，圆角等处理。

### DownSampler#decode

```
public Resource<Bitmap> decode(InputStream is, int outWidth, int outHeight,
  Options options) throws IOException {
    return decode(is, outWidth, outHeight, options, EMPTY_CALLBACKS);
}

 @SuppressWarnings({"resource", "deprecation"})
public Resource<Bitmap> decode(InputStream is, int requestedWidth, int requestedHeight,
      Options options, DecodeCallbacks callbacks) throws IOException {
    Preconditions.checkArgument(is.markSupported(), "You must provide an     InputStream that supports"
        + " mark()");

    ...

    try {
      // 核心代码
      Bitmap result = decodeFromWrappedStreams(is, bitmapFactoryOptions,
          downsampleStrategy, decodeFormat, isHardwareConfigAllowed,   requestedWidth,
          requestedHeight, fixBitmapToRequestedDimensions, callbacks);
      // 关注点7   
      // 解码得到Bitmap对象后，包装成BitmapResource对象返回，
      // 通过内部的get方法得到Resource<Bitmap>对象
      return BitmapResource.obtain(result, bitmapPool);
    } finally {
      releaseOptions(bitmapFactoryOptions);
      byteArrayPool.put(bytesForOptions);
    }
}

private Bitmap decodeFromWrappedStreams(InputStream is,
      BitmapFactory.Options options, DownsampleStrategy downsampleStrategy,
      DecodeFormat decodeFormat, boolean isHardwareConfigAllowed, int requestedWidth,
      int requestedHeight, boolean fixBitmapToRequestedDimensions,
      DecodeCallbacks callbacks) throws IOException {
    
    // 省去计算压缩比例等一系列非核心逻辑
    ...
    
    // 核心代码
    Bitmap downsampled = decodeStream(is, options, callbacks, bitmapPool);
    callbacks.onDecodeComplete(bitmapPool, downsampled);

    ...

    // Bimtap旋转处理
    ...
    
    return rotated;
}

private static Bitmap decodeStream(InputStream is,     BitmapFactory.Options options,
      DecodeCallbacks callbacks, BitmapPool bitmapPool) throws   IOException {
    
    ...
    
    TransformationUtils.getBitmapDrawableLock().lock();
    try {
      // 核心代码
      result = BitmapFactory.decodeStream(is, null, options);
    } catch (IllegalArgumentException e) {
      ...
    } finally {
      TransformationUtils.getBitmapDrawableLock().unlock();
    }

    if (options.inJustDecodeBounds) {
      is.reset();
    }
    return result;
}
```

从以上源码流程我们知道，最后是在DownSampler的decodeStream()方法中使用了BitmapFactory.decodeStream()来得到Bitmap对象。然后，我们来分析下图片时如何显示的，我们回到步骤19的DownSampler#decode方法，看到关注点7，这里是将Bitmap包装成BitmapResource对象返回，通过内部的get方法可以得到Resource对象，再回到步骤15的DecodeJob#run方法，这是使用了notifyEncodeAndRelease()方法对Resource对象进行了发布。

### DecodeJob#notifyEncodeAndRelease

```
private void notifyEncodeAndRelease(Resource<R> resource, DataSource     dataSource) {
 
    ...

    notifyComplete(result, dataSource);

    ...
    
}

private void notifyComplete(Resource<R> resource, DataSource     dataSource) {
    setNotifiedOrThrow();
    callback.onResourceReady(resource, dataSource);
}
```

从以上EngineJob的源码可知，它实现了DecodeJob.CallBack这个接口。

```
class EngineJob<R> implements DecodeJob.Callback<R>,
    Poolable {
    ...
}
```

### EngineJob#onResourceReady

```
@Override
public void onResourceReady(Resource<R> resource, DataSource   dataSource) {
  this.resource = resource;
  this.dataSource = dataSource;
  MAIN_THREAD_HANDLER.obtainMessage(MSG_COMPLETE, this).sendToTarget();
}

private static class MainThreadCallback implements Handler.Callback{

    ...

    @Override
    public boolean handleMessage(Message message) {
      EngineJob<?> job = (EngineJob<?>) message.obj;
      switch (message.what) {
        case MSG_COMPLETE:
          // 核心代码
          job.handleResultOnMainThread();
          break;
        ...
      }
      return true;
    }
}
```

从以上源码可知，通过主线程Handler对象进行切换线程，然后在主线程调用了handleResultOnMainThread这个方法。

```
@Synthetic
void handleResultOnMainThread() {
  ...

  //noinspection ForLoopReplaceableByForEach to improve perf
  for (int i = 0, size = cbs.size(); i < size; i++) {
    ResourceCallback cb = cbs.get(i);
    if (!isInIgnoredCallbacks(cb)) {
      engineResource.acquire();
      cb.onResourceReady(engineResource, dataSource);
    }
  }
 
  ...
}
```

这里又通过一个循环调用了所有ResourceCallback的方法，让我们回到步骤9处Engine#load方法的关注点8这行代码，这里对ResourceCallback进行了注册，在步骤8出SingleRequest#onSizeReady方法里的engine.load中，我们看到最后一个参数，传入的是this，可以明白，engineJob.addCallback(cb)这里的cb的实现类就是SingleRequest。接下来，让我们看看SingleRequest的onResourceReady方法。

### SingleRequest#onResourceReady

```
/**
 * A callback method that should never be invoked directly.
 */
@SuppressWarnings("unchecked")
@Override
public void onResourceReady(Resource<?> resource, DataSource   dataSource) {
  ...
  
  // 从Resource<Bitmap>中得到Bitmap对象
  Object received = resource.get();
  
  ...
  
  onResourceReady((Resource<R>) resource, (R) received, dataSource);
}

private void onResourceReady(Resource<R> resource, R resultDataSource dataSource) {

    ...

    try {
      ...

      if (!anyListenerHandledUpdatingTarget) {
        Transition<? super R> animation =
            animationFactory.build(dataSource, isFirstResource);
        // 核心代码
        target.onResourceReady(result, animation);
      }
    } finally {
      isCallingCallbacks = false;
    }

    notifyLoadSuccess();
}
```

在SingleRequest#onResourceReady方法中又调用了target.onResourceReady(result, animation)方法，这里的target其实就是我们在into方法中建立的那个BitmapImageViewTarget，看到BitmapImageViewTarget类，我们并没有发现onResourceReady方法，但是我们从它的子类ImageViewTarget中发现了onResourceReady方法，从这里继续往下看。

### ImageViewTarget#onResourceReady

```
public abstract class ImageViewTarget<Z> extends ViewTarget<ImageView, Z>
implements Transition.ViewAdapter {

    ...

    @Override
    public void onResourceReady(@NonNull Z resource, @Nullable       Transition<? super Z> transition) {
      if (transition == null || !transition.transition(resource, this))   {
        // 核心代码
        setResourceInternal(resource);
      } else {
        maybeUpdateAnimatable(resource);
      }
    }
 
    ...
    
    private void setResourceInternal(@Nullable Z resource) {
        // Order matters here. Set the resource first to make sure that the         Drawable has a valid and
        // non-null Callback before starting it.
        // 核心代码
        setResource(resource);
        maybeUpdateAnimatable(resource);
    }
    
    // 核心代码
    protected abstract void setResource(@Nullable Z resource);
}
```

这里我们在回到BitmapImageViewTarget的setResource方法中，终于看到Bitmap被设置到了当前的imageView上了。

```
public class BitmapImageViewTarget extends ImageViewTarget<Bitmap> {

    ...
    
    
    @Override
    protected void setResource(Bitmap resource) {
      view.setImageBitmap(resource);
    }
}
```

到这里，我们的分析就结束了，从以上的分析可知，Glide将大部分的逻辑处理都放在了最后一个into方法中，里面经过了20多个分析步骤才将请求图片流、解码出图片，到最终设置到对应的imageView上。

## 完整Glide加载流程图

![img](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9da2924eeab4ef79249b5836fd916da~tplv-k3u1fbpfcp-zoom-1.image)



> 可以看到，Glide最核心的逻辑都聚集在into()方法中，它里面的设计精巧而复杂，这部分的源码分析非常耗时，但是，如果你真真正正地去一步步去深入其中，你也许在Android进阶之路上将会有顿悟的感觉。

# GreenDao

## 基本使用流程

### 导入GreenDao的代码生成插件和库

```
// 项目下的build.gradle
buildscript {
    ...
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.0'
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.1' 
    }
}

// app模块下的build.gradle
apply plugin: 'com.android.application'
apply plugin: 'org.greenrobot.greendao'

...

dependencies {
    ...
    compile 'org.greenrobot:greendao:3.2.0' 
}
```

### 创建一个实体类，这里为HistoryData

```
@Entity
public class HistoryData {

    @Id(autoincrement = true)
    private Long id;

    private long date;

    private String data;
}
```

### 选择ReBuild Project，HistoryData会被自动添加Set/get方法，并生成整个项目的DaoMaster、DaoSession类，以及与该实体HistoryData对应的HistoryDataDao。

![image](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75ea30aea6034e939918d0da2b43c9d9~tplv-k3u1fbpfcp-zoom-1.image)

```
@Entity
public class HistoryData {

    @Id(autoincrement = true)
    private Long id;

    private long date;

    private String data;

    @Generated(hash = 1371145256)
    public HistoryData(Long id, long date, String data) {
        this.id = id;
        this.date = date;
        this.data = data;
    }

    @Generated(hash = 422767273)
    public HistoryData() {
    }

    public Long getId() {
        return this.id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public long getDate() {
        return this.date;
    }

    public void setDate(long date) {
        this.date = date;
    }

    public String getData() {
        return this.data;
    }

    public void setData(String data) {
        this.data = data;
    }
}
```

这里点明一下这几个类的作用：

- DaoMaster：所有Dao类的主人，负责整个库的运行，内部的静态抽象子类DevOpenHelper继承并重写了Android的SqliteOpenHelper。
- DaoSession：作为一个会话层的角色，用于生成相应的Dao对象、Dao对象的注册，操作Dao的具体对象。
- xxDao（HistoryDataDao）：生成的Dao对象，用于进行具体的数据库操作。

### 获取并使用相应的Dao对象进行增删改查操作

```
DaoMaster.DevOpenHelper devOpenHelper = new DaoMaster.DevOpenHelper(this, Constants.DB_NAME);
SQLiteDatabase database = devOpenHelper.getWritableDatabase();
DaoMaster daoMaster = new DaoMaster(database);
mDaoSession = daoMaster.newSession();
HistoryDataDao historyDataDao = daoSession.getHistoryDataDao();

// 省略创建historyData的代码
...

// 增
historyDataDao.insert(historyData);

// 删
historyDataDao.delete(historyData);

// 改
historyDataDao.update(historyData);

// 查
List<HistoryData> historyDataList = historyDataDao.loadAll();
```

本节将会以上述使用流程来对GreenDao的源码进行逐步分析，最后会分析下GreenDao中一些优秀的特性，让大家对GreenDao的理解有更一步的加深。

## GreenDao使用流程分析

### 创建数据库帮助类对象DaoMaster.DevOpenHelper

```
DaoMaster.DevOpenHelper devOpenHelper = new DaoMaster.DevOpenHelper(this, Constants.DB_NAME);
```

创建GreenDao内部实现的数据库帮助类对象devOpenHelper，核心源码如下：

```
public class DaoMaster extends AbstractDaoMaster {

    ...

    public static abstract class OpenHelper extends DatabaseOpenHelper {
    
    ...
    
         @Override
        public void onCreate(Database db) {
            Log.i("greenDAO", "Creating tables for schema version " + SCHEMA_VERSION);
            createAllTables(db, false);
        }
    }
    
    public static class DevOpenHelper extends OpenHelper {
    
        ...
        
        @Override
        public void onUpgrade(Database db, int oldVersion, int newVersion) {
            Log.i("greenDAO", "Upgrading schema from version " + oldVersion + " to " + newVersion + " by dropping all tables");
            dropAllTables(db, true);
            onCreate(db);
        }
    }
}
```

DevOpenHelper自身实现了更新的逻辑，这里是弃置了所有的表，并且调用了OpenHelper实现的onCreate方法用于创建所有的表，其中DevOpenHelper继承于OpenHelper，而OpenHelper自身又继承于DatabaseOpenHelper，那么，这个DatabaseOpenHelper这个类的作用是什么呢？

```
public abstract class DatabaseOpenHelper extends SQLiteOpenHelper {

    ...
    
    // 关注点1
    public Database getWritableDb() {
        return wrap(getWritableDatabase());
    }
    
    public Database getReadableDb() {
        return wrap(getReadableDatabase());
    }   
    
    protected Database wrap(SQLiteDatabase sqLiteDatabase) {
        return new StandardDatabase(sqLiteDatabase);
    }
    
    ...
    
    // 关注点2
    public Database getEncryptedWritableDb(String password) {
        EncryptedHelper encryptedHelper = checkEncryptedHelper();
        return encryptedHelper.wrap(encryptedHelper.getWritableDatabase(password));
    }
    
    public Database getEncryptedReadableDb(String password) {
        EncryptedHelper encryptedHelper = checkEncryptedHelper();
        return encryptedHelper.wrap(encryptedHelper.getReadableDatabase(password));
    }
    
    ...
    
    private class EncryptedHelper extends net.sqlcipher.database.SQLiteOpenHelper {
    
        ...
    
    
        protected Database wrap(net.sqlcipher.database.SQLiteDatabase     sqLiteDatabase) {
            return new EncryptedDatabase(sqLiteDatabase);
        }
    }
```

其实，DatabaseOpenHelper也是实现了SQLiteOpenHelper的一个帮助类，它内部可以获取到两种不同的数据库类型，一种是标准型的数据库**StandardDatabase**，另一种是加密型的数据库**EncryptedDatabase**，从以上源码可知，它们内部都通过wrap这样一个包装的方法，返回了对应的数据库类型，我们大致看一下StandardDatabase和EncryptedDatabase的内部实现。

```
public class StandardDatabase implements Database {

    // 这里的SQLiteDatabase是android.database.sqlite.SQLiteDatabase包下的
    private final SQLiteDatabase delegate;

    public StandardDatabase(SQLiteDatabase delegate) {
        this.delegate = delegate;
    }

    @Override
    public Cursor rawQuery(String sql, String[] selectionArgs) {
        return delegate.rawQuery(sql, selectionArgs);
    }

    @Override
    public void execSQL(String sql) throws SQLException {
        delegate.execSQL(sql);
    }

    ...
}

public class EncryptedDatabaseStatement implements DatabaseStatement     {

    // 这里的SQLiteStatement是net.sqlcipher.database.SQLiteStatement包下的
    private final SQLiteStatement delegate;

    public EncryptedDatabaseStatement(SQLiteStatement delegate) {
        this.delegate = delegate;
    }

    @Override
    public void execute() {
        delegate.execute();
    }
    
    ...
}
```

StandardDatabase和EncryptedDatabase这两个类内部都使用了**代理模式**给相同的接口添加了不同的具体实现，StandardDatabase自然是使用的Android包下的SQLiteDatabase，而EncryptedDatabaseStatement为了实现加密数据库的功能，则使用了一个叫做**sqlcipher**的数据库加密三方库，**如果你项目下的数据库需要保存比较重要的数据，则可以使用getEncryptedWritableDb方法来代替getdWritableDb方法对数据库进行加密，这样，我们之后的数据库操作则会以代理模式的形式间接地使用sqlcipher提供的API去操作数据库**。

### 创建DaoMaster对象

```
SQLiteDatabase database = devOpenHelper.getWritableDatabase();
DaoMaster daoMaster = new DaoMaster(database);
```

首先，DaoMaster作为所有Dao对象的主人，它内部肯定是需要一个SQLiteDatabase对象的，因此，先由DaoMaster的帮助类对象devOpenHelper的getWritableDatabase方法得到一个标准的数据库类对象database，再由此创建一个DaoMaster对象。

```
public class DaoMaster extends AbstractDaoMaster {

    ...

    public DaoMaster(SQLiteDatabase db) {
        this(new StandardDatabase(db));
    }

    public DaoMaster(Database db) {
        super(db, SCHEMA_VERSION);
        registerDaoClass(HistoryDataDao.class);
    }
    
    ...
}
```

在DaoMaster的构造方法中，它首先执行了super(db, SCHEMA_VERSION)方法，即它的父类AbstractDaoMaster的构造方法。

```
public abstract class AbstractDaoMaster {

    ...

    public AbstractDaoMaster(Database db, int schemaVersion) {
        this.db = db;
        this.schemaVersion = schemaVersion;

        daoConfigMap = new HashMap<Class<? extends AbstractDao<?, ?>>, DaoConfig>();
    }
    
    protected void registerDaoClass(Class<? extends AbstractDao<?, ?>> daoClass) {
        DaoConfig daoConfig = new DaoConfig(db, daoClass);
        daoConfigMap.put(daoClass, daoConfig);
    }
    
    ...
}
```

在AbstractDaoMaster对象的构造方法中，除了记录当前的数据库对象db和版本schemaVersion之外，还创建了一个类型为**HashMap<Class>, DaoConfig>()的daoConfigMap对象用于保存每一个DAO对应的数据配置对象DaoConfig，并且Daoconfig对象存储了对应的Dao对象所必需的数据**。最后，在DaoMaster的构造方法中使用了registerDaoClass(HistoryDataDao.class)方法将HistoryDataDao类对象进行了注册，实际上，就是为HistoryDataDao这个Dao对象创建了相应的DaoConfig对象并将它放入daoConfigMap对象中保存起来。

### 创建DaoSession对象

```
mDaoSession = daoMaster.newSession();
```

在DaoMaster对象中使用了newSession方法新建了一个DaoSession对象。

```
public DaoSession newSession() {
    return new DaoSession(db, IdentityScopeType.Session, daoConfigMap);
}
```

在DaoSeesion的构造方法中，又做了哪些事情呢？

```
public class DaoSession extends AbstractDaoSession {

    ...

    public DaoSession(Database db, IdentityScopeType type, Map<Class<?     extends AbstractDao<?, ?>>, DaoConfig>
            daoConfigMap) {
        super(db);

        historyDataDaoConfig = daoConfigMap.get(HistoryDataDao.class).clone();
        historyDataDaoConfig.initIdentityScope(type);

        historyDataDao = new HistoryDataDao(historyDataDaoConfig, this);

        registerDao(HistoryData.class, historyDataDao);
    }
    
    ...
}
```

首先，调用了父类AbstractDaoSession的构造方法。

```
public class AbstractDaoSession {

    ...

    public AbstractDaoSession(Database db) {
        this.db = db;
        this.entityToDao = new HashMap<Class<?>, AbstractDao<?, ?>>();
    }
    
    protected <T> void registerDao(Class<T> entityClass, AbstractDao<T, ?> dao) {
        entityToDao.put(entityClass, dao);
    }
    
    ...
}
```

在AbstractDaoSession构造方法里面**创建了一个实体与Dao对象的映射集合**。接下来，在DaoSession的构造方法中还做了2件事：

1. **创建每一个Dao对应的DaoConfig对象**，这里是historyDataDaoConfig，**并且根据IdentityScopeType的类型初始化创建一个相应的IdentityScope**，根据type的不同，它有两种类型，分别是**IdentityScopeObject**和**IdentityScopeLong**，它的作用是根据主键缓存对应的实体数据。当主键是数字类型的时候，如long/Long、int/Integer、short/Short、byte/Byte，则使用IdentityScopeLong缓存实体数据，当主键不是数字类型的时候，则使用IdentityScopeObject缓存实体数据。

2. **根据DaoSession对象和每一个Dao对应的DaoConfig对象，创建与之对应的historyDataDao对象**，由于这个项目只创建了一个实体类HistoryData，因此这里只有一个Dao对象historyDataDao，然后就是注册Dao对象，其实就是将实体和对应的Dao对象放入entityToDao这个映射集合中保存起来了。

### 插入源码分析

```
HistoryDataDao historyDataDao = daoSession.getHistoryDataDao();

// 增
historyDataDao.insert(historyData);
```

这里首先在会话层DaoSession中获取了我们要操作的Dao对象HistoryDataDao，然后插入了一个我们预先创建好的historyData实体对象。其中HistoryDataDao继承了AbstractDao<HistoryData, Long> 。

```
public class HistoryDataDao extends AbstractDao<HistoryData, Long> {
    ...
}
```

那么，这个AbstractDao是干什么的呢？

```
public abstract class AbstractDao<T, K> {

    ...
    
    public List<T> loadAll() {
        Cursor cursor = db.rawQuery(statements.getSelectAll(), null);
        return loadAllAndCloseCursor(cursor);
    }
    
    ...
    
    public long insert(T entity) {
        return executeInsert(entity, statements.getInsertStatement(),     true);
    }
    
    ...
    
    public void delete(T entity) {
        assertSinglePk();
        K key = getKeyVerified(entity);
        deleteByKey(key);
    }
    
    ...

}
```

看到这里，根据程序员优秀的直觉，大家应该能猜到，AbstractDao是所有Dao对象的基类，它实现了实体数据的操作如增删改查。我们接着分析insert是如何实现的，在AbstractDao的insert方法中又调用了executeInsert这个方法。在这个方法中，第二个参里的statements是一个**TableStatements**对象，它是在AbstractDao初始化构造器时从DaoConfig对象中取出来的，是一个**根据指定的表格创建SQL语句的一个帮助类**。使用statements.getInsertStatement()则是获取了一个插入的语句。而第三个参数则是判断是否是主键的标志。

```
public class TableStatements {

    ...

    public DatabaseStatement getInsertStatement() {
        if (insertStatement == null) {
            String sql = SqlUtils.createSqlInsert("INSERT INTO ", tablename, allColumns);
            DatabaseStatement newInsertStatement = db.compileStatement(sql);
            ...
        }
        return insertStatement;
    }

    ...
}
```

在TableStatements的getInsertStatement方法中，主要做了两件事：

1. **使用SqlUtils创建了插入的sql语句**。

2. **根据不同的数据库类型（标准数据库或加密数据库）将sql语句编译成当前数据库对应的语句**。

我们继续往下分析executeInsert的执行流程。

```
private long executeInsert(T entity, DatabaseStatement stmt, boolean setKeyAndAttach) {
    long rowId;
    if (db.isDbLockedByCurrentThread()) {
        rowId = insertInsideTx(entity, stmt);
    } else {
        db.beginTransaction();
        try {
            rowId = insertInsideTx(entity, stmt);
            db.setTransactionSuccessful();
        } finally {
            db.endTransaction();
        }
    }
    if (setKeyAndAttach) {
        updateKeyAfterInsertAndAttach(entity, rowId, true);
    }
    return rowId;
}
```

这里首先是判断数据库是否被当前线程锁定，如果是，则直接插入数据，否则为了避免死锁，则开启一个数据库事务，再进行插入数据的操作。最后如果设置了主键，则在插入数据之后更新主键的值并将对应的实体缓存到相应的identityScope中，这一块的代码流程如下所示：

```
protected void updateKeyAfterInsertAndAttach(T entity, long rowId, boolean lock) {
    if (rowId != -1) {
        K key = updateKeyAfterInsert(entity, rowId);
        attachEntity(key, entity, lock);
    } else {
       ...
    }
}

protected final void attachEntity(K key, T entity, boolean lock) {
    attachEntity(entity);
    if (identityScope != null && key != null) {
        if (lock) {
            identityScope.put(key, entity);
        } else {
            identityScope.putNoLock(key, entity);
        }
    }
}
```

接着，我们还是继续追踪主线流程，在executeInsert这个方法中调用了insertInsideTx进行数据的插入。

```
private long insertInsideTx(T entity, DatabaseStatement stmt) {
    synchronized (stmt) {
        if (isStandardSQLite) {
            SQLiteStatement rawStmt = (SQLiteStatement) stmt.getRawStatement();
            bindValues(rawStmt, entity);
            return rawStmt.executeInsert();
        } else {
            bindValues(stmt, entity);
            return stmt.executeInsert();
        }
    }
}
```

为了防止并发，这里使用了悲观锁保证了数据的一致性，在AbstractDao这个类中，大量使用了这种锁保证了它的线程安全性。接着，如果当前是标准数据库，则直接获取stmt这个DatabaseStatement类对应的原始语句进行实体字段属性的绑定和最后的执行插入操作。如果是加密数据库，则直接使用当前的加密数据库所属的插入语句进行实体字段属性的绑定和执行最后的插入操作。其中bindValues这个方法对应的实现类就是我们的HistoryDataDao类。

```
public class HistoryDataDao extends AbstractDao<HistoryData, Long> {

    ...

    @Override
    protected final void bindValues(DatabaseStatement stmt, HistoryData     entity) {
        stmt.clearBindings();

        Long id = entity.getId();
        if (id != null) {
            stmt.bindLong(1, id);
        }
        stmt.bindLong(2, entity.getDate());

        String data = entity.getData();
        if (data != null) {
            stmt.bindString(3, data);
        }
    }
    
    @Override
    protected final void bindValues(SQLiteStatement stmt, HistoryData     entity) {
        stmt.clearBindings();

        Long id = entity.getId();
        if (id != null) {
            stmt.bindLong(1, id);
        }
        stmt.bindLong(2, entity.getDate());

        String data = entity.getData();
        if (data != null) {
            stmt.bindString(3, data);
        }
    }

    ...
}
```

可以看到，这里对HistoryData的所有字段使用对应的数据库语句进行了绑定操作。这里最后再提及一下，**如果当前数据库是加密型时，则会使用最开始提及的DatabaseStatement的加密实现类EncryptedDatabaseStatement应用代理模式去使用sqlcipher这个加密型数据库的insert方法**。

### 查询源码分析

经过对插入源码的分析，相信大家对GreenDao内部的机制已经有了一些自己的理解，由于删除和更新内部的流程比较简单，且与插入源码有异曲同工之妙，这里就不再赘述了。最后再分析下查询的源码，查询的流程调用链较长，所以将它的核心流程源码直接给出。

```
List<HistoryData> historyDataList = historyDataDao.loadAll();

public List<T> loadAll() {
    Cursor cursor = db.rawQuery(statements.getSelectAll(), null);
    return loadAllAndCloseCursor(cursor);
}

protected List<T> loadAllAndCloseCursor(Cursor cursor) {
    try {
        return loadAllFromCursor(cursor);
    } finally {
        cursor.close();
    }
}

protected List<T> loadAllFromCursor(Cursor cursor) {
    int count = cursor.getCount();
    ...
    boolean useFastCursor = false;
    if (cursor instanceof CrossProcessCursor) {
        window = ((CrossProcessCursor) cursor).getWindow();
        if (window != null) {  
            if (window.getNumRows() == count) {
                cursor = new FastCursor(window);
                useFastCursor = true;
            } else {
              ...
            }
        }
    }

    if (cursor.moveToFirst()) {
        ...
        try {
            if (!useFastCursor && window != null && identityScope != null) {
                loadAllUnlockOnWindowBounds(cursor, window, list);
            } else {
                do {
                    list.add(loadCurrent(cursor, 0, false));
                } while (cursor.moveToNext());
            }
        } finally {
            ...
        }
    }
    return list;
}
```

最终，loadAll方法将会调用到loadAllFromCursor这个方法，首先，如果**当前的游标cursor是跨进程的cursor**，并且cursor的行数没有偏差的话，则使用一个加快版的**FastCursor**对象进行游标遍历。接着，不管是执行loadAllUnlockOnWindowBounds这个方法还是直接加载当前的数据列表list.add(loadCurrent(cursor, 0, false))，最后都会调用到这行list.add(loadCurrent(cursor, 0, false))代码，很明显，loadCurrent方法就是加载数据的方法。

```
final protected T loadCurrent(Cursor cursor, int offset, boolean lock) {
    if (identityScopeLong != null) {
        ...
        T entity = lock ? identityScopeLong.get2(key) : identityScopeLong.get2NoLock(key);
        if (entity != null) {
            return entity;
        } else {
            entity = readEntity(cursor, offset);
            attachEntity(entity);
            if (lock) {
                identityScopeLong.put2(key, entity);
            } else {
                identityScopeLong.put2NoLock(key, entity);
            }
            return entity;
        }
    } else if (identityScope != null) {
        ...
        T entity = lock ? identityScope.get(key) : identityScope.getNoLock(key);
        if (entity != null) {
            return entity;
        } else {
            entity = readEntity(cursor, offset);
            attachEntity(key, entity, lock);
            return entity;
        }
    } else {
        ...
        T entity = readEntity(cursor, offset);
        attachEntity(entity);
        return entity;
    }
}
```

#### loadCurrent方法内部的执行策略

**首先，如果有实体数据缓存identityScopeLong/identityScope，则先从缓存中取，如果缓存中没有，会使用该实体对应的Dao对象，这里的是HistoryDataDao，它在内部根据游标取出的数据新建了一个新的HistoryData实体对象返回。**

```
@Override
public HistoryData readEntity(Cursor cursor, int offset) {
    HistoryData entity = new HistoryData( //
        cursor.isNull(offset + 0) ? null : cursor.getLong(offset + 0), // id
        cursor.getLong(offset + 1), // date
        cursor.isNull(offset + 2) ? null : cursor.getString(offset + 2) // data
    );
    return entity;
}
```

**最后，如果是非identityScopeLong缓存类型，即是属于identityScope的情况下，则还会在identityScope中将上面获得的数据进行缓存。如果没有实体数据缓存的话，则直接调用readEntity组装数据返回即可。**

注意：对于GreenDao缓存的特性，可能会出现没有拿到最新数据的bug，因此，如果遇到这种情况，可以使用DaoSession的clear方法删除缓存。

## GreenDao是如何与ReactiveX结合？

首先，看下与rx结合的使用流程：

```
RxDao<HistoryData, Long> xxDao = daoSession.getHistoryDataDao().rx();
xxDao.insert(historyData)
        .observerOn(AndroidSchedulers.mainThread())
        .subscribe(new Action1<HistoryData>() {
            @Override
            public void call(HistoryData entity) {
                // insert success
            }
        });
```

在AbstractDao对象的.rx()方法中，创建了一个默认执行在io线程的rxDao对象。

```
@Experimental
public RxDao<T, K> rx() {
    if (rxDao == null) {
        rxDao = new RxDao<>(this, Schedulers.io());
    }
    return rxDao;
}
```

接着分析rxDao的insert方法。

```
@Experimental
public Observable<T> insert(final T entity) {
    return wrap(new Callable<T>() {
        @Override
        public T call() throws Exception {
            dao.insert(entity);
            return entity;
        }
    });
}
```

起实质作用的就是这个wrap方法了，在这个方法里面主要是调用了RxUtils.fromCallable(callable)这个方法。

```
@Internal
class RxBase {

    ...

    protected <R> Observable<R> wrap(Callable<R> callable) {
        return wrap(RxUtils.fromCallable(callable));
    }

    protected <R> Observable<R> wrap(Observable<R> observable) {
        if (scheduler != null) {
            return observable.subscribeOn(scheduler);
        } else {
            return observable;
        }
    }

    ...
}
```

在RxUtils的fromCallable这个方法内部，其实就是**使用defer这个延迟操作符来进行被观察者事件的发送，主要目的就是为了确保Observable被订阅后才执行**。最后，如果调度器scheduler存在的话，将通过外部的wrap方法将执行环境调度到io线程。

```
@Internal
class RxUtils {

    @Internal
    static <T> Observable<T> fromCallable(final Callable<T> callable) {
        return Observable.defer(new Func0<Observable<T>>() {

            @Override
            public Observable<T> call() {
                T result;
                try {
                    result = callable.call();
                } catch (Exception e) {
                    return Observable.error(e);
                }
                return Observable.just(result);
            }
        });
    }
}
```



> 在分析完GreenDao的核心源码之后发现，GreenDao作为最好的数据库框架之一，是有一定道理的。
>
> **首先，它通过使用自身的插件配套相应的freemarker模板生成所需的静态代码，避免了反射等消耗性能的操作。**
>
> **其次，它内部提供了实体数据的映射缓存机制，能够进一步加快查询速度。对于不同数据库对应的SQL语句，也使用了不同的DataBaseStatement实现类结合代理模式进行了封装，屏蔽了数据库操作等繁琐的细节。**
>
> **最后，它使用了sqlcipher提供了加密数据库的功能，在一定程度确保了安全性，同时，结合RxJava，我们便能更简洁地实现异步的数据库操作**。

# RxJava

## RxJava到底是什么？

RxJava是基于Java虚拟机上的响应式扩展库，它通过**使用可观察的序列将异步和基于事件的程序组合起来**。 与此同时，它**扩展了观察者模式来支持数据/事件序列**，并且添加了操作符，这些**操作符允许你声明性地组合序列**，同时抽象出要关注的问题：比如低级线程、同步、线程安全和并发数据结构等。

从RxJava的官方定义来看，我们如果要想真正地理解RxJava，就必须对它以下两个部分进行深入的分析：

1. **订阅流程**

2. **线程切换**

当然，RxJava操作符的源码也是很不错的学习资源，特别是FlatMap、Zip等操作符的源码，有很多可以借鉴的地方，但是它们内部的实现比较复杂。

## RxJava的订阅流程

首先给出RxJava消息订阅的例子：

```
Observable.create(newObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String>emitter) throws Exception {
        emitter.onNext("1");
        emitter.onNext("2");
        emitter.onNext("3");
        emitter.onComplete();
    }
}).subscribe(new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "onSubscribe");
    }
    @Override
    public void onNext(String s) {
        Log.d(TAG, "onNext : " + s);
    }
    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "onError : " + e.toString());
    }
    @Override
    public void onComplete() {
        Log.d(TAG, "onComplete");
    }
});
```

可以看到，这里首先创建了一个被观察者，然后创建一个观察者订阅了这个被观察者，因此下面分两个部分对RxJava的订阅流程进行分析：

1. **创建被观察者过程**

2. **订阅过程**

### 创建被观察者过程

首先，上面使用了Observable类的create()方法创建了一个被观察者，看看里面做了什么。

#### Observable#create()

```
// 省略一些检测性的注解
public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
    ObjectHelper.requireNonNull(source, "source is null");
    return RxJavaPlugins.onAssembly(new ObservableCreate<T>(source));
}
```

在Observable的create()里面实际上是创建了一个新的ObservableCreate对象，同时，把我们定义好的ObservableOnSubscribe对象传入了ObservableCreate对象中，最后调用了RxJavaPlugins.onAssembly()方法。接下来看看这个ObservableCreate是干什么的。

#### ObservableCreate

```
public final class ObservableCreate<T> extends Observable<T> {

    final ObservableOnSubscribe<T> source;

    public ObservableCreate(ObservableOnSubscribe<T> source) {
        this.source = source;
    }
    
    ...
}
```

这里仅仅是把ObservableOnSubscribe这个对象保存在ObservableCreate中了。然后看看RxJavaPlugins.onAssembly()这个方法的处理。

#### RxJavaPlugins#onAssembly()

```
public static <T> Observable<T> onAssembly(@NonNull Observable<T> source) {

    // 应用hook函数的一些处理，一般用到不到
    ...
    return source;
}
```

最终仅仅是把我们的ObservableCreate给返回了。

#### 创建被观察者过程小结

从以上分析可知，Observable.create()方法仅仅是**先将我们自定义的ObservableOnSubscribe对象重新包装成了一个ObservableCreate对象**。

### 订阅过程

接着，看看Observable.subscribe()的订阅过程是如何实现的。

#### Observable#subscribe()

```
public final void subscribe(Observer<? super T> observer) {
    ...
    
    // 1
    observer = RxJavaPlugins.onSubscribe(this,observer);
    
    ...
    
    // 2
    subscribeActual(observer);
    
    ...
}
```

在注释1处，在Observable的subscribe()方法内部首先调用了RxJavaPlugins的onSubscribe()方法。

#### RxJavaPlugins#onSubscribe()

```
public static <T> Observer<? super T> onSubscribe(@NonNull Observable<T> source, @NonNull Observer<? super T> observer) {

    // 应用hook函数的一些处理，一般用到不到
    ...
    
    return observer;
}
```

除去hook应用的逻辑，这里仅仅是将observer返回了。接着来分析下注释2处的subscribeActual()方法，

#### Observable#subscribeActual()

```
protected abstract void subscribeActual(Observer<? super T> observer);
```

这是一个抽象的方法，很明显，它对应的具体实现类就是我们在第一步创建的ObservableCreate类，接下来看到ObservableCreate的subscribeActual()方法。

#### ObservableCreate#subscribeActual()

```
@Override
protected void subscribeActual(Observer<? super T> observer) {
    // 1
    CreateEmitter<T> parent = new CreateEmitter<T>(observer);
    // 2
    observer.onSubscribe(parent);

    try {
        // 3
        source.subscribe(parent);
    } catch (Throwable ex) {
        Exceptions.throwIfFatal(ex);
        parent.onError(ex);
    }
}
```

在注释1处，首先新创建了一个CreateEmitter对象，同时传入了我们自定义的observer对象进去。

##### CreateEmitter

```
static final class CreateEmitter<T>
extends AtomicReference<Disposable>
implements ObservableEmitter<T>, Disposable {

    ...
    
    final Observer<? super T> observer;

    CreateEmitter(Observer<? super T> observer) {
        this.observer = observer;
    }
    
    ...
}
```

从上面可以看出，**CreateEmitter通过继承了Java并发包中的原子引用类AtomicReference保证了事件流切断状态Dispose的一致性**（这里不理解的话，看到后面讲解Dispose的时候就明白了），并**实现了ObservableEmitter接口和Disposable接口**，接着我们分析下注释2处的observer.onSubscribe(parent)，这个onSubscribe回调的含义其实就是**告诉观察者已经成功订阅了被观察者**。再看到注释3处的source.subscribe(parent)这行代码，这里的source其实是ObservableOnSubscribe对象，我们看到ObservableOnSubscribe的subscribe()方法。

##### ObservableOnSubscribe#subscribe()

```
Observable observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public voidsubscribe(ObservableEmitter<String> emitter) throws Exception {
        emitter.onNext("1");
        emitter.onNext("2");
        emitter.onNext("3");
        emitter.onComplete();
    }
});
```

这里面使用到了ObservableEmitter的onNext()方法将事件流发送出去，最后调用了onComplete()方法完成了订阅过程。ObservableEmitter是一个抽象类，实现类就是我们传入的CreateEmitter对象，接下来我们看看CreateEmitter的onNext()方法和onComplete()方法的处理。

##### CreateEmitter#onNext() && CreateEmitter#onComplete()

```
static final class CreateEmitter<T>
extends AtomicReference<Disposable>
implements ObservableEmitter<T>, Disposable {

...

@Override
public void onNext(T t) {
    ...
    
    if (!isDisposed()) {
        //调用观察者的onNext()
        observer.onNext(t);
    }
}

@Override
public void onComplete() {
    if (!isDisposed()) {
        try {
            observer.onComplete();
        } finally {
            dispose();
        }
    }
}


...

}
```

在CreateEmitter的onNext和onComplete方法中首先都要经过一个**isDisposed**的判断，作用就是看**当前的事件流是否被切断（废弃）掉了**，默认是不切断的，如果想要切断，可以调用Disposable的dispose()方法将此状态设置为切断（废弃）状态。继续看看这个isDisposed内部的处理。

##### ObservableEmitter#isDisposed()

```
@Override
public boolean isDisposed() {
    return DisposableHelper.isDisposed(get());
}
```

注意到这里通过get()方法首先从ObservableEmitter的AtomicReference中拿到了保存的Disposable状态。然后交给了DisposableHelper进行判断处理。接下来看看DisposableHelper的处理。

##### DisposableHelper#isDisposed() && DisposableHelper#set()

```
public enum DisposableHelper implements Disposable {

    DISPOSED;

    public static boolean isDisposed(Disposable d) {
        // 1
        return d == DISPOSED;
    }
    
    public static boolean set(AtomicReference<Disposable> field, Disposable d) {
        for (;;) {
            Disposable current = field.get();
            if (current == DISPOSED) {
                if (d != null) {
                    d.dispose();
                }
                return false;
            }
            // 2
            if (field.compareAndSet(current, d)) {
                if (current != null) {
                    current.dispose();
                }
                return true;
            }
        }
    }
    
    ...
    
    public static boolean dispose(AtomicReference<Disposable> field) {
        Disposable current = field.get();
        Disposable d = DISPOSED;
        if (current != d) {
            // ...
            current = field.getAndSet(d);
            if (current != d) {
                if (current != null) {
                    current.dispose();
                }
                return true;
            }
        }
        return false;
    }
    
    ...
}
```

DisposableHelper是一个枚举类，内部只有一个值即DISPOSED, 从上面的分析可知它就是用来**标记事件流被切断（废弃）状态的**。先看到注释2和注释3处的代码**field.compareAndSet(current, d)和field.getAndSet(d)**，这里使用了**原子引用AtomicReference内部包装的[CAS](https://www.jianshu.com/p/ab2c8fce878b)方法处理了标志Disposable的并发读写问题**。最后看到注释3处，将我们传入的CreateEmitter这个原子引用类保存的Dispable状态和DisposableHelper内部的DISPOSED进行比较，如果相等，就证明数据流被切断了。为了更进一步理解Disposed的作用，再来看看CreateEmitter中剩余的关键方法。

##### CreateEmitter

```
@Override
public void onNext(T t) {
    ...
    // 1
    if (!isDisposed()) {
        observer.onNext(t);
    }
}

@Override
public void onError(Throwable t) {
    if (!tryOnError(t)) {
        // 2
        RxJavaPlugins.onError(t);
    }
}

@Override
public boolean tryOnError(Throwable t) {
    ...
    // 3
    if (!isDisposed()) {
        try {
            observer.onError(t);
        } finally {
            // 4
            dispose();
        }
        return true;
    }
    return false;
}

@Override
public void onComplete() {
    // 5
    if (!isDisposed()) {
        try {
            observer.onComplete();
        } finally {
            // 6
            dispose();
        }
    }
}
```

在注释1、3、5处，onNext()和onError()、onComplete()方法首先都会判断事件流是否被切断，如果事件流此时被切断了，那么onNext()和onComplete()则会退出方法体，不做处理，**onError()则会执行到RxJavaPlugins.onError(t)这句代码，内部会直接抛出异常，导致崩溃**。如果事件流没有被切断，那么在onError()和onComplete()内部最终会调用到注释4、6处的这句dispose()代码，将事件流进行切断，由此可知，**onError()和onComplete()只能调用一个，如果先执行的是onComplete()，再调用onError()的话就会导致异常崩溃**。

## RxJava的线程切换

首先给出RxJava线程切换的例子：

```
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public voidsubscribe(ObservableEmitter<String>emitter) throws Exception {
        emitter.onNext("1");
        emitter.onNext("2");
        emitter.onNext("3");
        emitter.onComplete();
    }
}) 
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<String>() {
        @Override
        public void onSubscribe(Disposable d) {
            Log.d(TAG, "onSubscribe");
        }
        @Override
        public void onNext(String s) {
            Log.d(TAG, "onNext : " + s);
        }
        @Override
        public void onError(Throwable e) {
            Log.d(TAG, "onError : " +e.toString());
        }
        @Override
        public void onComplete() {
            Log.d(TAG, "onComplete");
        }
});
```

可以看到，RxJava的线程切换主要**分为subscribeOn()和observeOn()方法**，首先，来分析下subscribeOn()方法。

### subscribeOn(Schedulers.io())

在Schedulers.io()方法中，我们需要先传入一个Scheduler调度类，这里是传入了一个调度到io子线程的调度类，我们看看这个Schedulers.io()方法内部是怎么构造这个调度器的。

### Schedulers#io()

```
static final Scheduler IO;

...

public static Scheduler io() {
    // 1
    return RxJavaPlugins.onIoScheduler(IO);
}

static {
    ...

    // 2
    IO = RxJavaPlugins.initIoScheduler(new IOTask());
}

static final class IOTask implements Callable<Scheduler> {
    @Override
    public Scheduler call() throws Exception {
        // 3
        return IoHolder.DEFAULT;
    }
}

static final class IoHolder {
    // 4
    static final Scheduler DEFAULT = new IoScheduler();
}
```

Schedulers这个类的代码很多，这里我只拿出有关Schedulers.io这个方法涉及的逻辑代码进行讲解。首先，在注释1处，同前面分析的订阅流程的处理一样，只是一个处理hook的逻辑，最终返回的还是传入的这个IO对象。再看到注释2处，**在Schedulers的静态代码块中将IO对象进行了初始化，其实质就是新建了一个IOTask的静态内部类**，在IOTask的call方法中，也就是注释3处，可以了解到使用了静态内部类的方式把创建的IOScheduler对象给返回出去了。绕了这么大圈子，**Schedulers.io方法其实质就是返回了一个IOScheduler对象**。

### Observable#subscribeOn()

```
  public final Observable<T> subscribeOn(Scheduler scheduler) {
    ...
    
    return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
}
```

在subscribeOn()方法里面，又将ObservableCreate包装成了一个ObservableSubscribeOn对象。我们关注到ObservableSubscribeOn类。

### ObservableSubscribeOn

```
public final class ObservableSubscribeOn<T> extends AbstractObservableWithUpstream<T, T> {
    final Scheduler scheduler;

    public ObservableSubscribeOn(ObservableSource<T> source, Scheduler scheduler) {
        // 1
        super(source);
        this.scheduler = scheduler;
    }

    @Override
    public void subscribeActual(final Observer<? super T> observer) {
        // 2
        final SubscribeOnObserver<T> parent = new SubscribeOnObserver<T>(observer);
        
        // 3
        observer.onSubscribe(parent);
        
        // 4
        parent.setDisposable(scheduler.scheduleDirect(new SubscribeTask(parent)));
    }

...
}
```

首先，在注释1处，将传进来的source和scheduler保存起来。接着，等到实际订阅的时候，就会执行到这个subscribeActual方法，在注释2处，将我们自定义的Observer包装成了一个SubscribeOnObserver对象。在注释3处，通知观察者订阅了被观察者。在注释4处，内部先创建了一个SubscribeTask对象，来看看它的实现。

### ObservableSubscribeOn#SubscribeTask

```
final class SubscribeTask implements Runnable {
    private final SubscribeOnObserver<T> parent;

    SubscribeTask(SubscribeOnObserver<T> parent) {
        this.parent = parent;
    }

    @Override
    public void run() {
        source.subscribe(parent);
    }
}
```

SubscribeTask是ObservableSubscribeOn的内部类，它实质上就是一个任务类，在它的run方法中会执行到source.subscribe(parent)的订阅方法，**这个source其实就是我们在ObservableSubscribeOn构造方法中传进来的ObservableCreate对象**。接下来看看scheduler.scheduleDirect()内部的处理。

### Scheduler#scheduleDirect()

```
public Disposable scheduleDirect(@NonNull Runnable run) {
    return scheduleDirect(run, 0L, TimeUnit.NANOSECONDS);
}

public Disposable scheduleDirect(@NonNull Runnable run, long delay, @NonNull TimeUnit unit) {

    // 1
    final Worker w = createWorker();

    // 2
    final Runnable decoratedRun = RxJavaPlugins.onSchedule(run);

    // 3
    DisposeTask task = new DisposeTask(decoratedRun, w);

    // 4
    w.schedule(task, delay, unit);

    return task;
}
```

这里最后会执行到上面这个scheduleDirect()重载方法。首先，在注释1处，会调用createWorker()方法创建一个工作者对象Worker，它是一个抽象类，这里的实现类就是IoScheduler，下面看看IoScheduler类的createWorker()方法。

#### IOScheduler#createWorker()

```
final AtomicReference<CachedWorkerPool> pool;

...

public IoScheduler(ThreadFactory threadFactory) {
    this.threadFactory = threadFactory;
    this.pool = new AtomicReference<CachedWorkerPool>(NONE);
    start();
}

...

@Override
public Worker createWorker() {
    // 1
    return new EventLoopWorker(pool.get());
}

static final class EventLoopWorker extends Scheduler.Worker {
    ...

    EventLoopWorker(CachedWorkerPool pool) {
        this.pool = pool;
        this.tasks = new CompositeDisposable();
        // 2
        this.threadWorker = pool.get();
    }
    
}
```

首先，在注释1处调用了pool.get()这个方法，**pool是一个CachedWorkerPool类型的原子引用对象**，它的作用就是**用于缓存工作者对象Worker的**。然后，将得到的CachedWorkerPool传入新创建的EventLoopWorker对象中。重点关注一下注释2处，这里将CachedWorkerPool缓存的threadWorker对象保存起来了。

下面继续分析3.6处代码段的注释2处的代码，这里又是一个关于hook的封装处理，最终还是返回的当前的Runnable对象。在注释3处新建了一个切断任务DisposeTask将decoratedRun和w对象包装了起来。最后在注释4处调用了工作者的schedule()方法。下面来分析下它内部的处理。

#### IoScheduler#schedule()

```
@Override
public Disposable schedule(@NonNull Runnableaction, long delayTime, @NonNull TimeUnit unit){
    ...
    
    return threadWorker.scheduleActual(action,delayTime, unit, tasks);
}
```

内部调用了threadWorker的scheduleActual()方法，实际上是调用到了父类NewThreadWorker的scheduleActual()方法，继续看看NewThreadWorker的scheduleActual()方法中做的事情。

#### NewThreadWorker#scheduleActual()

```
public NewThreadWorker(ThreadFactory threadFactory) {
    executor = SchedulerPoolFactory.create(threadFactory);
}


@NonNull
public ScheduledRunnable scheduleActual(final Runnable run, long delayTime, @NonNull TimeUnit unit, @Nullable DisposableContainer parent) {
    Runnable decoratedRun = RxJavaPlugins.onSchedule(run);

    // 1
    ScheduledRunnable sr = new ScheduledRunnable(decoratedRun, parent);
    
   
    if (parent != null) {
        if (!parent.add(sr)) {
            return sr;
        }
    }

    Future<?> f;
    try {
        // 2
        if (delayTime <= 0) {
            // 3
            f = executor.submit((Callable<Object>)sr);
        } else {
            // 4
            f = executor.schedule((Callable<Object>)sr, delayTime, unit);
        }
        sr.setFuture(f);
    } catch (RejectedExecutionException ex) {
        if (parent != null) {
            parent.remove(sr);
        }
        RxJavaPlugins.onError(ex);
    }

    return sr;
}
```

在NewThreadWorker的scheduleActual()方法的内部，在注释1处首先会新建一个ScheduledRunnable对象，将Runnable对象和parent包装起来了，**这里parent是一个DisposableContainer对象，它实际的实现类是CompositeDisposable类，它是一个保存所有事件流是否被切断状态的容器，其内部的实现是使用了RxJava自己定义的一个简单的OpenHashSet类进行存储**。最后注释2处，判断是否设置了延迟时间，如果设置了，则调用线程池的submit()方法立即进行线程切换，否则，调用schedule()方法进行延时执行线程切换。

### 为什么多次执行subscribeOn()，只有第一次有效？

从上面的分析，可以很容易了解到**被观察者被订阅时是从最外面的一层（ObservableSubscribeOn）通知到里面的一层（ObservableOnSubscribe）**，当连续执行了到多次subscribeOn()的时候，其实就是先执行倒数第一次的subscribeOn()方法，直到最后一次执行的subscribeOn()方法，这样肯定会覆盖前面的线程切换。

### observeOn(AndroidSchedulers.mainThread())

```
public final Observable<T> observeOn(Scheduler scheduler) {
    return observeOn(scheduler, false, bufferSize());
}

public final Observable<T> observeOn(Scheduler scheduler, boolean delayError, int bufferSize) {
    ....
    
    return RxJavaPlugins.onAssembly(new ObservableObserveOn<T>(this, scheduler, delayError, bufferSize));
}
```

可以看到，observeOn()方法内部最终也是返回了一个ObservableObserveOn对象，直接来看看ObservableObserveOn的subscribeActual()方法。

### ObservableObserveOn#subscribeActual()

```
@Override
protected void subscribeActual(Observer<? super T> observer) {
    // 1
    if (scheduler instanceof TrampolineScheduler) {
        // 2
        source.subscribe(observer);
    } else {
        // 3
        Scheduler.Worker w = scheduler.createWorker();
        // 4
        source.subscribe(new ObserveOnObserver<T>(observer, w, delayError, bufferSize));
    }
}
```

首先，在注释1处，判断指定的调度器是不是TrampolineScheduler，这是一个不进行线程切换，立即执行当前代码的调度器。如果是，则会直接调用ObservableSubscribeOn的subscribe()方法，如果不是，则会在注释3处创建一个工作者对象。然后，在注释4处创建一个新的ObserveOnObserver将SubscribeOnobserver对象包装起来，并传入ObservableSubscribeOn的subscribe()方法进行订阅。接下来看看ObserveOnObserver类的重点方法。

### ObserveOnObserver

```
@Override
public void onNext(T t) {
    ...
    if (sourceMode != QueueDisposable.ASYNC) {
        // 1
        queue.offer(t);
    }
    schedule();
}

@Override
public void onError(Throwable t) {
    ...
    schedule();
}

@Override
public void onComplete() {
    ...
    schedule();
}

```

去除非主线逻辑的代码，在ObserveOnObserver的onNext()和onError()、onComplete()方法中最后都会调用到schedule()方法。接着看schedule()方法，其中**onNext()还会把消息存放到队列中**。

### ObserveOnObserver#schedule()

```
void schedule() {
    if (getAndIncrement() == 0) {
        worker.schedule(this);
    }
}
```

这里使用了worker进行调度ObserveOnObserver这个实现了Runnable的任务。worker就是在AndroidSchedulers.mainThread()中创建的，内部其实就是**使用Handler进行线程切换的**，此处不再赘述了。接着看ObserveOnObserver的run()方法。

### ObserveOnObserver#run()

```
@Override
public void run() {
    // 1
    if (outputFused) {
        drainFused();
    } else {
        // 2
        drainNormal();
    }
}
```

在注释1处会**先判断outputFused这个标志位，它表示事件流是否被融化掉，默认是false，所以，最后会执行到drainNormal()方法**。接着看看drainNormal()方法内部的处理。

### ObserveOnObserver#drainNormal()

```
void drainNormal() {
    int missed = 1;
    
    final SimpleQueue<T> q = queue;
    
    // 1
    final Observer<? super T> a = downstream;
    
    ...
    
    // 2
    v = q.poll();
    
    ...
    // 3
    a.onNext(v);
    
    ...
}
```

在注释1处，这里的downstream实际上是从外面传进来的SubscribeOnObserver对象。在注释2处将队列中的消息取出来，接着在注释3处调用了SubscribeOnObserver的onNext方法。**最终，会从我们包装类的最外层一直调用到最里面的我们自定义的Observer中的onNext()方法，所以，在observeOn()方法下面的链式代码都会执行到它所指定的线程中，噢，原来如此**。



> 很多人使用RxJava也已经挺长时间了，但是一直没有去深入去了解过它的内部实现原理，**如今细细品尝，的确是酣畅淋漓**。

# LeakCanary

## 原理概述

查看Leakcanary官方的github仓库，最重要的便是对**Leakcanary是如何起作用的**（即原理）这一问题进行了阐述，把它翻译成了易于理解的文字，主要分为如下7个步骤：

1. RefWatcher.watch()创建了一个KeyedWeakReference用于去观察对象。

2. 然后，在后台线程中，它会检测引用是否被清除了，并且是否没有触发GC。

3. 如果引用仍然没有被清除，那么它将会把堆栈信息保存在文件系统中的.hprof文件里。

4. HeapAnalyzerService被开启在一个独立的进程中，并且HeapAnalyzer使用了HAHA开源库解析了指定时刻的堆栈快照文件heap dump。

5. 从heap dump中，HeapAnalyzer根据一个独特的引用key找到了KeyedWeakReference，并且定位了泄露的引用。

6. HeapAnalyzer为了确定是否有泄露，计算了到GC Roots的最短强引用路径，然后建立了导致泄露的链式引用。

7. 这个结果被传回到app进程中的DisplayLeakService，然后一个泄露通知便展现出来了。

官方的原理简单来解释就是这样的：**在一个Activity执行完onDestroy()之后，将它放入WeakReference中，然后将这个WeakReference类型的Activity对象与ReferenceQueque关联。这时再从ReferenceQueque中查看是否有没有该对象，如果没有，执行gc，再次查看，还是没有的话则判断发生内存泄露了。最后用HAHA这个开源库去分析dump之后的heap内存。**

## 简单示例

下面这段是Leakcanary官方仓库的示例代码：

首先在你项目app下的build.gradle中配置:

```
dependencies {
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:1.6.2'
  releaseImplementation   'com.squareup.leakcanary:leakcanary-android-no-op:1.6.2'
  // 可选，如果你使用支持库的fragments的话
  debugImplementation   'com.squareup.leakcanary:leakcanary-support-fragment:1.6.2'
}
```

然后在你的Application中配置:

```
public class WanAndroidApp extends Application {

    private RefWatcher refWatcher;
    
    public static RefWatcher getRefWatcher(Context context) {
        WanAndroidApp application = (WanAndroidApp)     context.getApplicationContext();
        return application.refWatcher;
    }

    @Override public void onCreate() {
      super.onCreate();
      if (LeakCanary.isInAnalyzerProcess(this)) {
        // 1
        return;
      }
      // 2
      refWatcher = LeakCanary.install(this);
    }
}
```

在注释1处，会首先判断当前进程是否是Leakcanary专门用于分析heap内存的而创建的那个进程，即HeapAnalyzerService所在的进程，如果是的话，则不进行Application中的初始化功能。如果是当前应用所处的主进程的话，则会执行注释2处的LeakCanary.install(this)进行LeakCanary的安装。只需这样简单的几行代码，我们就可以在应用中检测是否产生了内存泄露了。当然，这样使用只会检测Activity和标准Fragment是否发生内存泄漏，如果要检测V4包的Fragment在执行完onDestroy()之后是否发生内存泄露的话，则需要在Fragment的onDestroy()方法中加上如下两行代码去监视当前的Fragment：

```
RefWatcher refWatcher = WanAndroidApp.getRefWatcher(_mActivity);
refWatcher.watch(this);
```

上面的**RefWatcher其实就是一个引用观察者对象，是用于监测当前实例对象的引用状态的**。从以上的分析可以了解到，核心代码就是LeakCanary.install(this)这行代码，接下来，就从这里出发将LeakCanary一步一步进行拆解。

## 源码分析

### LeakCanary#install()

```
public static @NonNull RefWatcher install(@NonNull Application application) {
  return refWatcher(application).listenerServiceClass(DisplayLeakService.class)
      .excludedRefs(AndroidExcludedRefs.createAppDefaults().build())
      .buildAndInstall();
}
```

在install()方法中的处理，可以分解为如下四步：

1. **refWatcher(application)**

2. **链式调用listenerServiceClass(DisplayLeakService.class)**

3. **链式调用excludedRefs(AndroidExcludedRefs.createAppDefaults().build())**

4. **链式调用buildAndInstall()**

首先，我们来看下第一步，这里调用了LeakCanary类的refWatcher方法，如下所示：

```
public static @NonNull AndroidRefWatcherBuilder refWatcher(@NonNull Context context) {
  return new AndroidRefWatcherBuilder(context);
}
```

然后新建了一个AndroidRefWatcherBuilder对象，再看看AndroidRefWatcherBuilder这个类。

### AndroidRefWatcherBuilder

```
/** A {@link RefWatcherBuilder} with appropriate Android defaults. */
public final class AndroidRefWatcherBuilder extends RefWatcherBuilder<AndroidRefWatcherBuilder> {

...

    AndroidRefWatcherBuilder(@NonNull Context context) {
        this.context = context.getApplicationContext();
    }

...
}
```

在AndroidRefWatcherBuilder的构造方法中仅仅是将外部传入的applicationContext对象保存起来了。**AndroidRefWatcherBuilder是一个适配Android平台的引用观察者构造器对象，它继承了RefWatcherBuilder，RefWatcherBuilder是一个负责建立引用观察者RefWatcher实例的基类构造器**。继续看看RefWatcherBuilder这个类。

### RefWatcherBuilder

```
public class RefWatcherBuilder<T extends RefWatcherBuilder<T>> {

    ...
    
    public RefWatcherBuilder() {
        heapDumpBuilder = new HeapDump.Builder();
    }

    ...
}
```

在RefWatcher的基类构造器RefWatcherBuilder的构造方法中新建了一个HeapDump的构造器对象。其中**HeapDump就是一个保存heap dump信息的数据结构**。

接着来分析下install()方法中的链式调用的listenerServiceClass(DisplayLeakService.class)这部分逻辑。

### AndroidRefWatcherBuilder#listenerServiceClass()

```
public @NonNull AndroidRefWatcherBuilder listenerServiceClass(
  @NonNull Class<? extends AbstractAnalysisResultService> listenerServiceClass) {
    return heapDumpListener(new ServiceHeapDumpListener(context, listenerServiceClass));
}
```

在这里，传入了一个DisplayLeakService的Class对象，它的作用是展示泄露分析的结果日志，然后会展示一个用于跳转到显示泄露界面DisplayLeakActivity的通知。在listenerServiceClass()这个方法中新建了一个ServiceHeapDumpListener对象，下面看看它内部的操作。

### ServiceHeapDumpListener

```
public final class ServiceHeapDumpListener implements HeapDump.Listener {

    ...
    
    public ServiceHeapDumpListener(@NonNull final Context context,
        @NonNull final Class<? extends AbstractAnalysisResultService> listenerServiceClass) {
      this.listenerServiceClass = checkNotNull(listenerServiceClass, "listenerServiceClass");
      this.context = checkNotNull(context, "context").getApplicationContext();
    }
    
    ...
}
```

可以看到这里仅仅是在ServiceHeapDumpListener中保存了DisplayLeakService的Class对象和application对象。它的作用就是接收一个heap dump去分析。

然后我们继续看install()方法链式调用.excludedRefs(AndroidExcludedRefs.createAppDefaults().build())的这部分代码。先看AndroidExcludedRefs.createAppDefaults()。

### AndroidExcludedRefs#createAppDefaults()

```
public enum AndroidExcludedRefs {

    ...

    public static @NonNull ExcludedRefs.Builder createAppDefaults() {
      return createBuilder(EnumSet.allOf(AndroidExcludedRefs.class));
    }
    
    public static @NonNull ExcludedRefs.Builder createBuilder(EnumSet<AndroidExcludedRefs> refs) {
      ExcludedRefs.Builder excluded = ExcludedRefs.builder();
      for (AndroidExcludedRefs ref : refs) {
        if (ref.applies) {
          ref.add(excluded);
          ((ExcludedRefs.BuilderWithParams) excluded).named(ref.name());
        }
      }
      return excluded;
    }
    
    ...
}
```

先来说下**AndroidExcludedRefs**这个类，它是一个enum类，它**声明了Android SDK和厂商定制的SDK中存在的内存泄露的case**，根据AndroidExcludedRefs这个类的类名就可看出这些case**都会被Leakcanary的监测过滤掉**。目前这个版本是有**46种**这样的**case**被包含在内，后续可能会一直增加。然后EnumSet.allOf(AndroidExcludedRefs.class)这个方法将会返回一个包含AndroidExcludedRefs元素类型的EnumSet。Enum是一个抽象类，在这里具体的实现类是**通用正规型的RegularEnumSet，如果Enum里面的元素个数大于64，则会使用存储大数据量的JumboEnumSet**。最后，在createBuilder这个方法里面构建了一个排除引用的建造器excluded，将各式各样的case分门别类地保存起来再返回出去。

最后看到链式调用的最后一步buildAndInstall()。

### AndroidRefWatcherBuilder#buildAndInstall()

```
private boolean watchActivities = true;
private boolean watchFragments = true;

public @NonNull RefWatcher buildAndInstall() {
    // 1
    if (LeakCanaryInternals.installedRefWatcher != null) {
      throw new UnsupportedOperationException("buildAndInstall() should only be called once.");
    }
    
    // 2
    RefWatcher refWatcher = build();
    if (refWatcher != DISABLED) {
      // 3
      LeakCanaryInternals.setEnabledAsync(context, DisplayLeakActivity.class, true);
      if (watchActivities) {
        // 4
        ActivityRefWatcher.install(context, refWatcher);
      }
      if (watchFragments) {
        // 5
        FragmentRefWatcher.Helper.install(context, refWatcher);
      }
    }
    // 6
    LeakCanaryInternals.installedRefWatcher = refWatcher;
    return refWatcher;
}
```

首先，在注释1处，会判断LeakCanaryInternals.installedRefWatcher是否已经被赋值，如果被赋值了，则会抛出异常，警告 buildAndInstall()这个方法应该仅仅只调用一次，在此方法结束时，即在注释6处，该LeakCanaryInternals.installedRefWatcher才会被赋值。再来看注释2处，调用了AndroidRefWatcherBuilder其基类RefWatcherBuilder的build()方法，看看它是如何建造的。

### RefWatcherBuilder#build()

```
public final RefWatcher build() {
    if (isDisabled()) {
      return RefWatcher.DISABLED;
    }

    if (heapDumpBuilder.excludedRefs == null) {
      heapDumpBuilder.excludedRefs(defaultExcludedRefs());
    }

    HeapDump.Listener heapDumpListener = this.heapDumpListener;
    if (heapDumpListener == null) {
      heapDumpListener = defaultHeapDumpListener();
    }

    DebuggerControl debuggerControl = this.debuggerControl;
    if (debuggerControl == null) {
      debuggerControl = defaultDebuggerControl();
    }

    HeapDumper heapDumper = this.heapDumper;
    if (heapDumper == null) {
      heapDumper = defaultHeapDumper();
    }

    WatchExecutor watchExecutor = this.watchExecutor;
    if (watchExecutor == null) {
      watchExecutor = defaultWatchExecutor();
    }

    GcTrigger gcTrigger = this.gcTrigger;
    if (gcTrigger == null) {
      gcTrigger = defaultGcTrigger();
    }

    if (heapDumpBuilder.reachabilityInspectorClasses == null) {
      heapDumpBuilder.reachabilityInspectorClasses(defa  ultReachabilityInspectorClasses());
    }

    return new RefWatcher(watchExecutor, debuggerControl, gcTrigger, heapDumper, heapDumpListener,
        heapDumpBuilder);
}
```

可以看到，**RefWatcherBuilder包含了以下7个组成部分：**

1. **excludedRefs : 记录可以被忽略的泄漏路径**。

2. **heapDumpListener : 转储堆信息到hprof文件，并在解析完 hprof 文件后进行回调，最后通知 DisplayLeakService 弹出泄漏提醒**。

3. debuggerControl : 判断是否处于调试模式，调试模式中不会进行内存泄漏检测。为什么呢？因为**在调试过程中可能会保留上一个引用从而导致错误信息上报**。

4. **heapDumper : 堆信息转储者，负责dump 内存泄漏处的 heap 信息到 hprof 文件**。

5. **watchExecutor : 线程控制器，在 onDestroy() 之后并且在主线程空闲时执行内存泄漏检测**。

6. **gcTrigger : 用于 GC，watchExecutor 首次检测到可能的内存泄漏，会主动进行 GC，GC 之后会再检测一次，仍然泄漏的判定为内存泄漏，最后根据heapDump信息生成相应的泄漏引用链**。

7. **reachabilityInspectorClasses : 用于要进行可达性检测的类列表。**

最后，会使用建造者模式将这些组成部分构建成一个新的RefWatcher并将其返回。

继续看回到AndroidRefWatcherBuilder的注释3处的 LeakCanaryInternals.setEnabledAsync(context, DisplayLeakActivity.class, true)这行代码。

### LeakCanaryInternals#setEnabledAsync()

```
public static void setEnabledAsync(Context context, final Class<?> componentClass,
final boolean enabled) {
  final Context appContext = context.getApplicationContext();
  AsyncTask.THREAD_POOL_EXECUTOR.execute(new Runnable() {
    @Override public void run() {
      setEnabledBlocking(appContext, componentClass, enabled);
    }
  });
}
```

在这里直接使用了**AsyncTask内部自带的THREAD_POOL_EXECUTOR线程池**进行阻塞式地显示DisplayLeakActivity。

然后再继续看AndroidRefWatcherBuilder的注释4处的代码。

### ActivityRefWatcher#install()

```
public static void install(@NonNull Context context, @NonNull RefWatcher refWatcher) {
    Application application = (Application) context.getApplicationContext();
    // 1
    ActivityRefWatcher activityRefWatcher = new ActivityRefWatcher(application, refWatcher);
    
    // 2
    application.registerActivityLifecycleCallbacks(activityRefWatcher.lifecycleCallbacks);
}
```

可以看到，在注释1处创建一个自己的activityRefWatcher实例，并在注释2处调用了application的registerActivityLifecycleCallbacks()方法，这样就能够监听activity对应的生命周期事件了。继续看看activityRefWatcher.lifecycleCallbacks里面的操作。

```
private final Application.ActivityLifecycleCallbacks lifecycleCallbacks =
    new ActivityLifecycleCallbacksAdapter() {
      @Override public void onActivityDestroyed(Activity activity) {
          refWatcher.watch(activity);
      }
};

public abstract class ActivityLifecycleCallbacksAdapter
implements Application.ActivityLifecycleCallbacks {

}
```

很明显，这里**实现并重写了Application的ActivityLifecycleCallbacks的onActivityDestroyed()方法，这样便能在所有Activity执行完onDestroyed()方法之后调用 refWatcher.watch(activity)这行代码进行内存泄漏的检测了**。

再看到注释5处的FragmentRefWatcher.Helper.install(context, refWatcher)这行代码，

### FragmentRefWatcher.Helper#install()

```
public interface FragmentRefWatcher {

    void watchFragments(Activity activity);

    final class Helper {
    
      private static final String SUPPORT_FRAGMENT_REF_WATCHER_CLASS_NAME =
          "com.squareup.leakcanary.internal.SupportFragmentRefWatcher";
    
      public static void install(Context context, RefWatcher refWatcher) {
        List<FragmentRefWatcher> fragmentRefWatchers = new ArrayList<>();
    
        // 1
        if (SDK_INT >= O) {
          fragmentRefWatchers.add(new AndroidOFragmentRefWatcher(refWatcher));
        }
    
        // 2
        try {
          Class<?> fragmentRefWatcherClass = Class.forName(SUPPORT_FRAGMENT_REF_WATCHER_CLASS_NAME);
          Constructor<?> constructor =
              fragmentRefWatcherClass.getDeclaredConstructor(RefWatcher.class);
          FragmentRefWatcher supportFragmentRefWatcher   =
              (FragmentRefWatcher) constructor.newInstance(refWatcher);
          fragmentRefWatchers.add(supportFragmentRefWatcher);
        } catch (Exception ignored) {
        }
    
        if (fragmentRefWatchers.size() == 0) {
          return;
        }
    
        Helper helper = new Helper(fragmentRefWatchers);
    
        // 3
        Application application = (Application) context.getApplicationContext();
        application.registerActivityLifecycleCallbacks(helper.activityLifecycleCallbacks);
      }
      
    ...
}
```

这里面的逻辑很简单，首先在注释1处将Android标准的Fragment的RefWatcher类，即AndroidOfFragmentRefWatcher添加到新创建的fragmentRefWatchers中。在注释2处**使用反射将leakcanary-support-fragment包下面的SupportFragmentRefWatcher添加进来，如果你在app的build.gradle下没有添加下面这行引用的话，则会拿不到此类，即LeakCanary只会检测Activity和标准Fragment这两种情况**。

```
debugImplementation   'com.squareup.leakcanary:leakcanary-support-fragment:1.6.2'
```

继续看到注释3处helper.activityLifecycleCallbacks里面的代码。

```
private final Application.ActivityLifecycleCallbacks activityLifecycleCallbacks =
    new ActivityLifecycleCallbacksAdapter() {
      @Override public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
        for (FragmentRefWatcher watcher : fragmentRefWatchers) {
            watcher.watchFragments(activity);
        }
    }
};
```

可以看到，在Activity执行完onActivityCreated()方法之后，会调用指定watcher的watchFragments()方法，注意，这里的watcher可能有两种，但不管是哪一种，都会使用当前传入的activity获取到对应的FragmentManager/SupportFragmentManager对象，调用它的registerFragmentLifecycleCallbacks()方法，在对应的onDestroyView()和onDestoryed()方法执行完后，分别使用refWatcher.watch(view)和refWatcher.watch(fragment)进行内存泄漏的检测，代码如下所示。

```
@Override public void onFragmentViewDestroyed(FragmentManager fm, Fragment fragment) {
    View view = fragment.getView();
    if (view != null) {
        refWatcher.watch(view);
    }
}

@Override
public void onFragmentDestroyed(FragmentManagerfm, Fragment fragment) {
    refWatcher.watch(fragment);
}
```

注意，下面到真正关键的地方了，接下来分析refWatcher.watch()这行代码。

### RefWatcher#watch()

```
public void watch(Object watchedReference, String referenceName) {
    if (this == DISABLED) {
      return;
    }
    checkNotNull(watchedReference, "watchedReference");
    checkNotNull(referenceName, "referenceName");
    final long watchStartNanoTime = System.nanoTime();
    // 1
    String key = UUID.randomUUID().toString();
    // 2
    retainedKeys.add(key);
    // 3
    final KeyedWeakReference reference =
        new KeyedWeakReference(watchedReference, key, referenceName, queue);

    // 4
    ensureGoneAsync(watchStartNanoTime, reference);
}
```

注意到在注释1处**使用随机的UUID保证了每个检测对象对应 key 的唯一性**。在注释2处将生成的key添加到类型为CopyOnWriteArraySet的Set集合中。在注释3处新建了一个自定义的弱引用KeyedWeakReference，看看它内部的实现。

### KeyedWeakReference

```
final class KeyedWeakReference extends WeakReference<Object> {
    public final String key;
    public final String name;
    
    KeyedWeakReference(Object referent, String key, String name,
        ReferenceQueue<Object> referenceQueue) {
      // 1
      super(checkNotNull(referent, "referent"), checkNotNull(referenceQueue, "referenceQueue"));
      this.key = checkNotNull(key, "key");
      this.name = checkNotNull(name, "name");
    }
}
```

可以看到，**在KeyedWeakReference内部，使用了key和name标识了一个被检测的WeakReference对象**。在注释1处，**将弱引用和引用队列 ReferenceQueue 关联起来，如果弱引用reference持有的对象被GC回收，JVM就会把这个弱引用加入到与之关联的引用队列referenceQueue中。即 KeyedWeakReference 持有的 Activity 对象如果被GC回收，该对象就会加入到引用队列 referenceQueue 中**。

接着回到RefWatcher.watch()里注释4处的ensureGoneAsync()方法。

### RefWatcher#ensureGoneAsync()

```
private void ensureGoneAsync(final long watchStartNanoTime, final KeyedWeakReference reference) {
    // 1
    watchExecutor.execute(new Retryable() {
        @Override public Retryable.Result run() {
            // 2
            return ensureGone(reference watchStartNanoTime);
        }
    });
}
```

在ensureGoneAsync()方法中，在注释1处使用 watchExecutor 执行了注释2处的 ensureGone 方法，watchExecutor 是 AndroidWatchExecutor 的实例。

下面看看watchExecutor内部的逻辑。

### AndroidWatchExecutor

```
public final class AndroidWatchExecutor implements WatchExecutor {

    ...
    
    public AndroidWatchExecutor(long initialDelayMillis)     {
      mainHandler = new Handler(Looper.getMainLooper());
      HandlerThread handlerThread = new HandlerThread(LEAK_CANARY_THREAD_NAME);
      handlerThread.start();
      // 1
      backgroundHandler = new Handler(handlerThread.getLooper());
      this.initialDelayMillis = initialDelayMillis;
      maxBackoffFactor = Long.MAX_VALUE / initialDelayMillis;
    }
    
    @Override public void execute(@NonNull Retryable retryable) {
      // 2
      if (Looper.getMainLooper().getThread() == Thread.currentThread()) {
        waitForIdle(retryable, 0);
      } else {
        postWaitForIdle(retryable, 0);
      }
    }
    
    ...
}
```

在注释1处**AndroidWatchExecutor的构造方法**中，注意到这里**使用HandlerThread的looper新建了一个backgroundHandler**，后面会用到。在注释2处，会判断当前线程是否是主线程，如果是，则直接调用waitForIdle()方法，如果不是，则调用postWaitForIdle()，来看看这个方法。

```
private void postWaitForIdle(final Retryable retryable, final int failedAttempts) {
  mainHandler.post(new Runnable() {
    @Override public void run() {
      waitForIdle(retryable, failedAttempts);
    }
  });
}
```

很清晰，这里使用了在构造方法中用主线程looper构造的mainHandler进行post，那么waitForIdle()最终也会在主线程执行。接着看看waitForIdle()的实现。

```
private void waitForIdle(final Retryable retryable,     final int failedAttempts) {
  Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
    @Override public boolean queueIdle() {
      postToBackgroundWithDelay(retryable, failedAttempts);
      return false;
    }
  });
}
```

这里**MessageQueue.IdleHandler()回调方法的作用是当 looper 空闲的时候，会回调 queueIdle 方法，利用这个机制我们可以实现第三方库的延迟初始化**，然后执行内部的postToBackgroundWithDelay()方法。接下来看看它的实现。

```
private void postToBackgroundWithDelay(final Retryable retryable, final int failedAttempts) {
  long exponentialBackoffFactor = (long) Math.min(Math.pow(2, failedAttempts),     maxBackoffFactor);
  // 1
  long delayMillis = initialDelayMillis * exponentialBackoffFactor;
  // 2
  backgroundHandler.postDelayed(new Runnable() {
    @Override public void run() {
      // 3
      Retryable.Result result = retryable.run();
      // 4
      if (result == RETRY) {
        postWaitForIdle(retryable, failedAttempts +   1);
      }
    }
  }, delayMillis);
}
```

先看到注释4处，可以明白，postToBackgroundWithDelay()是一个递归方法，如果result 一直等于RETRY的话，则会一直执行postWaitForIdle()方法。在回到注释1处，这里initialDelayMillis 的默认值是 5s，因此delayMillis就是5s。在注释2处，使用了在构造方法中用HandlerThread的looper新建的backgroundHandler进行异步延时执行retryable的run()方法。这个run()方法里执行的就是RefWatcher的ensureGoneAsync()方法中注释2处的ensureGone()这行代码，继续看它内部的逻辑。

### RefWatcher#ensureGone()

```
Retryable.Result ensureGone(final KeyedWeakReference reference, final long watchStartNanoTime) {
    long gcStartNanoTime = System.nanoTime();
    long watchDurationMs = NANOSECONDS.toMillis(gcStartNanoTime -     watchStartNanoTime);

    // 1
    removeWeaklyReachableReferences();

    // 2
    if (debuggerControl.isDebuggerAttached()) {
      // The debugger can create false leaks.
      return RETRY;
    }
    
    // 3
    if (gone(reference)) {
      return DONE;
    }
    
    // 4
    gcTrigger.runGc();
    removeWeaklyReachableReferences();
    
    // 5
    if (!gone(reference)) {
      long startDumpHeap = System.nanoTime();
      long gcDurationMs = NANOSECONDS.toMillis(startDumpHeap - gcStartNanoTime);

      File heapDumpFile = heapDumper.dumpHeap();
      if (heapDumpFile == RETRY_LATER) {
        // Could not dump the heap.
        return RETRY;
      }
      
      long heapDumpDurationMs = NANOSECONDS.toMillis(System.nanoTime() - startDumpHeap);

      HeapDump heapDump = heapDumpBuilder.heapDumpFile(heapDumpFile).referenceKey(reference.key)
          .referenceName(reference.name)
          .watchDurationMs(watchDurationMs)
          .gcDurationMs(gcDurationMs)
          .heapDumpDurationMs(heapDumpDurationMs)
          .build();

      heapdumpListener.analyze(heapDump);
    }
    return DONE;
}
```

在注释1处，执行了removeWeaklyReachableReferences()这个方法，接下来分析下它的含义。

```
private void removeWeaklyReachableReferences() {
    KeyedWeakReference ref;
    while ((ref = (KeyedWeakReference) queue.poll()) != null) {
        retainedKeys.remove(ref.key);
    }
}
```

这里使用了while循环遍历 ReferenceQueue ，并从 retainedKeys中移除对应的Reference。

再看到注释2处，**当Android设备处于debug状态时，会直接返回RETRY进行延时重试检测的操作**。在注释3处，看看gone(reference)这个方法的逻辑。

```
private boolean gone(KeyedWeakReference reference) {
    return !retainedKeys.contains(reference.key);
}
```

这里会**判断 retainedKeys 集合中是否还含有 reference，若没有，证明已经被回收了，若含有，可能已经发生内存泄露（或Gc还没有执行回收）**。前面的分析中我们知道了 **reference 被回收的时候，会被加进 referenceQueue 里面，然后我们会调用removeWeaklyReachableReferences()遍历 referenceQueue 移除掉 retainedKeys 里面的 refrence**。

接着看到注释4处，执行了gcTrigger的runGc()方法进行垃圾回收，然后使用了removeWeaklyReachableReferences()方法移除已经被回收的引用。这里再深入地分析下runGc()的实现。

```
GcTrigger DEFAULT = new GcTrigger() {
    @Override public void runGc() {
      // Code taken from AOSP FinalizationTest:
      // https://android.googlesource.com/platform/libc  ore/+/master/support/src/test/java/libcore/
      // java/lang/ref/FinalizationTester.java
      // System.gc() does not garbage collect every   time. Runtime.gc() is
      // more likely to perform a gc.
      Runtime.getRuntime().gc();
      enqueueReferences();
      System.runFinalization();
    }

    private void enqueueReferences() {
      // Hack. We don't have a programmatic way to wait   for the reference queue daemon to move
      // references to the appropriate queues.
      try {
        Thread.sleep(100);
      } catch (InterruptedException e) {
        throw new AssertionError();
      }
    }
};
```

这里并没有使用System.gc()方法进行回收，因为**system.gc()并不会每次都执行**。而是**从AOSP中拷贝一段GC回收的代码，从而相比System.gc()更能够保证垃圾回收的工作**。

最后分析下注释5处的代码处理。首先会判断activity是否被回收，如果还没有被回收，则证明发生内存泄露，进行if判断里面的操作。在里面先调用堆信息转储者heapDumper的dumpHeap()生成相应的 hprof 文件。这里的heapDumper是一个HeapDumper接口，具体的实现是AndroidHeapDumper。我们分析下AndroidHeapDumper的dumpHeap()方法是如何生成hprof文件的。

```
public File dumpHeap() {
    File heapDumpFile = leakDirectoryProvider.newHeapDumpFile();

    if (heapDumpFile == RETRY_LATER) {
        return RETRY_LATER;
    }

    ...
    
    try {
      Debug.dumpHprofData(heapDumpFile.getAbsolutePath());
      ...
      
      return heapDumpFile;
    } catch (Exception e) {
      ...
      // Abort heap dump
      return RETRY_LATER;
    }
}
```

这里的核心操作就是**调用了Android SDK的API Debug.dumpHprofData() 来生成 hprof 文件**。

如果这个文件等于RETRY_LATER则表示生成失败，直接返回RETRY进行延时重试检测的操作。如果不等于的话，则表示生成成功，最后会**执行heapdumpListener的analyze()对新创建的HeapDump对象进行泄漏分析**。由前面对AndroidRefWatcherBuilder的listenerServiceClass()的分析可知，heapdumpListener的实现 就是ServiceHeapDumpListener，接着看到ServiceHeapDumpListener的analyze方法。

### ServiceHeapDumpListener#analyze()

```
@Override public void analyze(@NonNull HeapDump heapDump) {
    checkNotNull(heapDump, "heapDump");
    HeapAnalyzerService.runAnalysis(context, heapDump, listenerServiceClass);
}
```

可以看到，这里**执行了HeapAnalyzerService的runAnalysis()方法，为了避免降低app进程的性能或占用内存，这里将HeapAnalyzerService设置在了一个独立的进程中**。接着继续分析runAnalysis()方法里面的处理。

```
public final class HeapAnalyzerService extends ForegroundService
implements AnalyzerProgressListener {

    ...

    public static void runAnalysis(Context context, HeapDump heapDump,
    Class<? extends AbstractAnalysisResultService> listenerServiceClass) {
        ...
        
        ContextCompat.startForegroundService(context, intent);
    }

    ...
    
    @Override protected void onHandleIntentInForeground(@Nullable Intent intent) {
        ...

        // 1
        HeapAnalyzer heapAnalyzer =
            new HeapAnalyzer(heapDump.excludedRefs, this, heapDump.reachabilityInspectorClasses);

        // 2
        AnalysisResult result = heapAnalyzer.checkForLeak(heapDump.heapDumpFile, heapDump.referenceKey,
        heapDump.computeRetainedHeapSize);
        
        // 3
        AbstractAnalysisResultService.sendResultToListener(this, listenerClassName, heapDump, result);
    }
        ...
}
```

这里的HeapAnalyzerService实质是一个类型为IntentService的ForegroundService，执行startForegroundService()之后，会回调onHandleIntentInForeground()方法。注释1处，首先会新建一个**HeapAnalyzer**对象，顾名思义，它就是**根据RefWatcher生成的heap dumps信息来分析被怀疑的泄漏是否是真的**。在注释2处，然后会**调用它的checkForLeak()方法去使用haha库解析 hprof文件**，如下所示：

```
public @NonNull AnalysisResult checkForLeak(@NonNull File heapDumpFile,
  @NonNull String referenceKey,
  boolean computeRetainedSize) {
    ...
    
    try {
    listener.onProgressUpdate(READING_HEAP_DUMP_FILE);
    // 1
    HprofBuffer buffer = new MemoryMappedFileBuffer(heapDumpFile);
  
    // 2
    HprofParser parser = new HprofParser(buffer);
    listener.onProgressUpdate(PARSING_HEAP_DUMP);
    Snapshot snapshot = parser.parse();
  
    listener.onProgressUpdate(DEDUPLICATING_GC_ROOTS);
    // 3
    deduplicateGcRoots(snapshot);
    listener.onProgressUpdate(FINDING_LEAKING_REF);
  
    // 4
    Instance leakingRef = findLeakingReference(referenceKey, snapshot);

    // 5
    if (leakingRef == null) {
        return noLeak(since(analysisStartNanoTime));
    }
  
    // 6
    return findLeakTrace(analysisStartNanoTime, snapshot, leakingRef, computeRetainedSize);
    } catch (Throwable e) {
    return failure(e, since(analysisStartNanoTime));
    }
}
```

在注释1处，会新建一个**内存映射缓存文件buffer**。在注释2处，会**使用buffer新建一个HprofParser解析器去解析出对应的引用内存快照文件snapshot**。在注释3处，**为了减少在Android 6.0版本中重复GCRoots带来的内存压力的影响，使用deduplicateGcRoots()删除了gcRoots中重复的根对象RootObj**。在注释4处，**调用了findLeakingReference()方法将传入的referenceKey和snapshot对象里面所有类实例的字段值对应的keyCandidate进行比较，如果没有相等的，则表示没有发生内存泄漏**，直接调用注释5处的代码返回一个没有泄漏的分析结果AnalysisResult对象。**如果找到了相等的，则表示发生了内存泄漏**，执行注释6处的代码findLeakTrace()方法返回一个有泄漏分析结果的AnalysisResult对象。

最后，来分析下HeapAnalyzerService中注释3处的AbstractAnalysisResultService.sendResultToListener()方法，很明显，这里AbstractAnalysisResultService的实现类就是我们刚开始分析的用于展示泄漏路径信息的DisplayLeakService对象。在里面直接**创建一个由PendingIntent构建的泄漏通知用于供用户点击去展示详细的泄漏界面DisplayLeakActivity**。核心代码如下所示：

```
public class DisplayLeakService extends AbstractAnalysisResultService {

    @Override
    protected final void onHeapAnalyzed(@NonNull AnalyzedHeap analyzedHeap) {
    
        ...
        
        boolean resultSaved = false;
        boolean shouldSaveResult = result.leakFound || result.failure != null;
        if (shouldSaveResult) {
            heapDump = renameHeapdump(heapDump);
            // 1
            resultSaved = saveResult(heapDump, result);
        }
        
        if (!shouldSaveResult) {
            ...
            showNotification(null, contentTitle, contentText);
        } else if (resultSaved) {
            ...
            // 2
            PendingIntent pendingIntent =
                DisplayLeakActivity.createPendingIntent(this, heapDump.referenceKey);

            ...
            
            showNotification(pendingIntent, contentTitle, contentText);
        } else {
             onAnalysisResultFailure(getString(R.string.leak_canary_could_not_save_text));
        }
    
    ...
}

@Override protected final void onAnalysisResultFailure(String failureMessage) {
    super.onAnalysisResultFailure(failureMessage);
    String failureTitle = getString(R.string.leak_canary_result_failure_title);
    showNotification(null, failureTitle, failureMessage);
}
```

可以看到，只要当分析的堆信息文件保存成功之后，即在注释1处返回的resultSaved为true时，才会执行注释2处的逻辑，即创建一个供用户点击跳转到DisplayLeakActivity的延时通知。

## LeakCanary运作流程

![image](https:////p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0363331db824e7ab342e4bd74702a93~tplv-k3u1fbpfcp-zoom-1.image)



> 性能优化一直是Android中进阶和深入的方向之一，而内存泄漏一直是性能优化中比较重要的一部分，Android Studio自身提供了MAT等工具去分析内存泄漏，但是分析起来比较耗时耗力，因而才诞生了LeakCanary，它的使用非常简单，但是经过对它的深入分析之后，才发现，**简单的API后面往往藏着许多复杂的逻辑处理，尝试去领悟它们，你可能会发现不一样的世界**。

# ButterKnife

## 简单示例

首先看一下ButterKnife的基本使用，如下所示：

```
public class CollectFragment extends BaseRootFragment<CollectPresenter> implements CollectContract.View {

    @BindView(R.id.normal_view)
    SmartRefreshLayout mRefreshLayout;
    @BindView(R.id.collect_recycler_view)
    RecyclerView mRecyclerView;
    @BindView(R.id.collect_floating_action_btn)
    FloatingActionButton mFloatingActionButton;
    
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(getLayoutId(), container, false);
        unBinder = ButterKnife.bind(this, view);
        initView();
        return view;
    }
    
    @OnClick({R.id.collect_floating_action_btn})
    void onClick(View view) {
        switch (view.getId()) {
            case R.id.collect_floating_action_btn:
                mRecyclerView.smoothScrollToPosition(0);
                break;
            default:
                break;
        }
    }
    
    
    @Override
    public void onDestroyView() {
        super.onDestroyView();
        if (unBinder != null && unBinder != Unbinder.EMPTY) {
            unBinder.unbind();
            unBinder = null;
        }
    }
```

可以看到，我们使用了@BindView()替代了findViewById()方法，然后使用了@OnClick替代了setOnClickListener()方法。ButterKnife的初期版本是通过使用注解+反射这样的运行时解析的方式实现上述功能的，后面，为了改善性能，便使用了**注解+APT编译时解析技术并从中生成配套模板代码的方式**来实现。

在开始分析之前，可能有同学对APT不是很了解，这里普及一下，APT是Annotation Processing Tool的缩写，即注解处理工具。它的使用步骤通常为如下三个步骤：

1. **首先，声明注解的生命周期为CLASS，即@Retention(CLASS)**。

2. **然后，通过继承AbstractProcessor自定义一个注解处理器**。

3. **最后，在编译的时候，编译器会扫描所有带有你要处理的注解的类，最后再调用AbstractProcessor的process方法，对注解进行处理**。

下面，正式来解剖一下ButterKnife的心脏。

## 源码分析

### 模板代码解析

首先，在编写好上述的示例代码之后，调用 gradle build 命令，在app/build/generated/source/apt下将可以找到APT为我们生产的配套模板代码CollectFragment_ViewBinding，如下所示：

```
public class CollectFragment_ViewBinding implements Unbinder {
    private CollectFragment target;
    
    private View view2131230812;
    
    @UiThread
    public CollectFragment_ViewBinding(final CollectFragment target, View source) {
      this.target = target;
    
      View view;
      // 1
      target.mRefreshLayout = Utils.findRequiredViewAsType(source, R.id.normal_view, "field 'mRefreshLayout'", SmartRefreshLayout.class);
      target.mRecyclerView = Utils.findRequiredViewAsType(source, R.id.collect_recycler_view, "field 'mRecyclerView'", RecyclerView.class);
      view = Utils.findRequiredView(source, R.id.collect_floating_action_btn, "field 'mFloatingActionButton' and method 'onClick'");
      target.mFloatingActionButton = Utils.castView(view, R.id.collect_floating_action_btn, "field 'mFloatingActionButton'", FloatingActionButton.class);
      view2131230812 = view;
      // 2
      view.setOnClickListener(new DebouncingOnClickListener() {
        @Override
        public void doClick(View p0) {
          target.onClick(p0);
        }
      });
    }
    
    @Override
    @CallSuper
    public void unbind() {
      CollectFragment target = this.target;
      if (target == null) throw newIllegalStateException("Bindings already     cleared.");
      this.target = null;
    
      target.mRefreshLayout = null;
      target.mRecyclerView = null;
      target.mFloatingActionButton = null;
    
      view2131230812.setOnClickListener(null);
      view2131230812 = null;
    }
}
```

生成的配套模板CollectFragment_ViewBinding中，在注释1处，使用了ButterKnife内部的工具类Utils的findRequiredViewAsType()方法来寻找控件。在注释2处，使用了view的setOnClickListener()方法来添加了一个去抖动的DebouncingOnClickListener，这样便可以防止重复点击，在重写的doClick()方法内部，直接调用了CollectFragment的onClick方法。最后，再深入看下Utils的findRequiredViewAsType()方法内部的实现。

```
public static <T> T findRequiredViewAsType(View source, @IdRes int id, String who,
  Class<T> cls) {
    // 1
    View view = findRequiredView(source, id, who);
    // 2
    return castView(view, id, who, cls);
}

public static View findRequiredView(View source, @IdRes int id, String who) {
    View view = source.findViewById(id);
    if (view != null) {
        return view;
    }
    
    ...
}

public static <T> T castView(View view, @IdRes int id, String who, Class<T> cls) {
    try {
        return cls.cast(view);
    } catch (ClassCastException e) {
        ...
    }
}
```

在注释1处，**最终也是通过View的findViewById()方法找到相应的控件**，在注释2处，**通过相应Class对象的cast方法强转成对应的控件类型**。

### ButterKnife 是怎样实现代码注入的

接下来，为了使用这套模板代码，我们必须调用ButterKnife的bind()方法实现代码注入，即自动帮我们执行重复繁琐的findViewById和setOnClicklistener操作。下面我们来分析下bind()方法是如何实现注入的。

```
@NonNull @UiThread
public static Unbinder bind(@NonNull Object target, @NonNull View source) {
    return createBinding(target, source);
}
```

在bind()方法中调用了createBinding()，

```
@NonNull @UiThread
public static Unbinder bind(@NonNull Object target, @NonNull View source) {
    Class<?> targetClass = target.getClass();
    // 1
    Constructor<? extends Unbinder> constructor = findBindingConstructorForClass(targetClass);

    if (constructor == null) {
        return Unbinder.EMPTY;
    }

    
    try {
        // 2
        return constructor.newInstance(target, source);
    // 3
    } catch (IllegalAccessException e) {
    ...
}
```

首先，在注释1处，通过 findBindingConstructorForClass() 方法从 Class 中查找 constructor，这里constructor即上文生成的CollectFragment_ViewBinding类。然后，在注释2处，**利用反射来新建 constructor 对象**。最后，如果新建 constructor 对象失败，则会在注释3后面捕获一系列对应的异常进行自定义异常抛出处理。

下面，来详细分析下 findBindingConstructorForClass() 方法的实现逻辑。

```
@VisibleForTesting
static final Map<Class<?>, Constructor<? extends Unbinder>> BINDINGS = new LinkedHashMap<>();

@Nullable @CheckResult @UiThread
private static Constructor<? extends Unbinder> findBindingConstructorForClass(Class<?> cls) {
    // 1
    Constructor<? extends Unbinder> bindingCtor = BINDINGS.get(cls);
    if (bindingCtor != null || BINDINGS.containsKey(cls)) {
        return bindingCtor;
    }
    
    // 2
    String clsName = cls.getName();
    if (clsName.startsWith("android.") || clsName.startsWith("java.")
    || clsName.startsWith("androidx.")) {
        return null;
    }
    
    try {
        // 3
        Class<?> bindingClass = cls.getClassLoader().loadClass(clsName + "_ViewBinding");
        bindingCtor = (Constructor<? extends Unbinder>) bindingClass.getConstructor(cls, View.class);
    } catch (ClassNotFoundException e) {
        // 4
        bindingCtor = findBindingConstructorForClass(cls.getSuperclass());
    } catch (NoSuchMethodException e) {
        throw new RuntimeException("Unable to find binding constructor for " + clsName, e);
    }
    
    // 5
    BINDINGS.put(cls, bindingCtor);
    return bindingCtor;
}
```

这里，我把多余的log代码删除并把代码格式优化了一下，可以看到，findBindingConstructorForClass() 这个方法中的逻辑瞬间清晰不少，这里**建议以后大家自己在分析源码的时候可以进行这样的优化重整**，会带来不少好处。

重新看到 findBindingConstructorForClass() 方法，在注释1处，我们首先从缓存BINDINGS中获取CollectFragment类对象对应的模块类CollectFragment_ViewBinding的构造器对象，这里的BINDINGS是一个LinkedHashMap对象，它保存了上述两者的映射关系。在注释2处，如果是 android，androidx，java 原生的文件，不进行处理。在注释3处，先**通过CollectFragment类对象的类加载器加载出对应的模块类CollectFragment_ViewBinding的类对象**，再通过自身的getConstructor()方法获得相应的构造对象。如果在步骤3中加载不出对应的模板类对象，则会在注释4处使用类似递归的方法重新执行findBindingConstructorForClass()方法。最后，如果找到了bindingCtor模板构造对象，则将它保存在BINDINGS这个LinkedHashMap对象中。

**这里总结一下findBindingConstructorForClass()方法的处理：**

1. **首先从缓存BINDINGS中获取CollectFragment类对象对应的模块类CollectFragment_ViewBinding的构造器对象，获取不到，则继续执行下面的操作**。

2. **如果不是android，androidx，java 原生的文件，再进行后面的处理**。

3. **通过CollectFragment类对象的类加载器加载出对应的模块类CollectFragment_ViewBinding的类对象，再通过自身的getConstructor()方法获得相应的构造对象，如果获取不到，会抛出异常，在异常的处理中，我们会从当前 class 文件的父类中再去查找。如果找到了，最后会将bindingCtor对象缓存进在BINDINGS对象中**。

### ButterKnife是如何在编译时生成代码的？

在编译的时候，ButterKnife会通过自定义的注解处理器ButterKnifeProcessor的process方法，对编译器扫描到的要处理的类中的注解进行处理，然后，**通过javapoet这个库来动态生成绑定事件或者控件的模板代码**，最后在运行的时候，直接调用bind方法完成绑定即可。

首先，先来分析下ButterKnifeProcessor的重写的入口方法init()。

```
@Override public synchronized void init(ProcessingEnvironment env) {
    super.init(env);

    String sdk = env.getOptions().get(OPTION_SDK_INT);
    if (sdk != null) {
        try {
            this.sdk = Integer.parseInt(sdk);
        } catch (NumberFormatException e) {
           ...
        }
    }

    typeUtils = env.getTypeUtils();
    filer = env.getFiler();
    ...
}
```

可以看到，**ProcessingEnviroment对象提供了两大工具类 typeUtils和filer。typeUtils的作用是用来处理TypeMirror，而Filer则是用来创建生成辅助文件**。

接着，再来看看被重写的getSupportedAnnotationTypes()方法，这个方法的作用主要是用于指定ButterknifeProcessor注册了哪些注解的。

```
@Override public Set<String> getSupportedAnnotationTypes() {
    Set<String> types = new LinkedHashSet<>();
    for (Class<? extends Annotation> annotation : getSupportedAnnotations()) {
    types.add(annotation.getCanonicalName());
    }
    return types;
}
```

这里面首先创建了一个LinkedHashSet对象，然后将getSupportedAnnotations()方法返回的支持注解集合进行遍历一一并添加到types中返回。

接着看下getSupportedAnnotations()方法，

```
private Set<Class<? extends Annotation>> getSupportedAnnotations() {
    Set<Class<? extends Annotation>> annotations = new LinkedHashSet<>();

    annotations.add(BindAnim.class);
    annotations.add(BindArray.class);
    annotations.add(BindBitmap.class);
    annotations.add(BindBool.class);
    annotations.add(BindColor.class);
    annotations.add(BindDimen.class);
    annotations.add(BindDrawable.class);
    annotations.add(BindFloat.class);
    annotations.add(BindFont.class);
    annotations.add(BindInt.class);
    annotations.add(BindString.class);
    annotations.add(BindView.class);
    annotations.add(BindViews.class);
    annotations.addAll(LISTENERS);

    return annotations;
}
```

可以看到，这里注册了一系列的Bindxxx注解类和监听列表LISTENERS，接着看一下LISTENERS中包含的监听方法：

```
private static final List<Class<? extends Annotation>> LISTENERS = Arrays.asList(
    OnCheckedChanged.class, 
    OnClick.class, 
    OnEditorAction.class, 
    OnFocusChange.class, 
    OnItemClick.class, 
    OnItemLongClick.class, 
    OnItemSelected.class, 
    OnLongClick.class, 
    OnPageChange.class, 
    OnTextChanged.class, 
    OnTouch.class 
);
```

最后，来分析下整个ButterKnifeProcessor中最关键的方法process()。

```
@Override public boolean process(Set<? extends TypeElement> elements, RoundEnvironment env) {
    // 1
    Map<TypeElement, BindingSet> bindingMap = findAndParseTargets(env);

    for (Map.Entry<TypeElement, BindingSet> entry : bindingMap.entrySet()) {
        TypeElement typeElement = entry.getKey();
        BindingSet binding = entry.getValue();

        // 2
        JavaFile javaFile = binding.brewJava(sdk, debuggable);
        try {
            javaFile.writeTo(filer);
        } catch (IOException e) {
           ...
        }
    }

    return false;
}
```

首先，在注释1处通过**findAndParseTargets()方法**，知名见义，它应该就是**找到并解析注解目标的关键方法**了，继续看看它内部的处理：

```
private Map<TypeElement, BindingSet> findAndParseTargets(RoundEnvironment env) {
    Map<TypeElement, BindingSet.Builder> builderMap = new LinkedHashMap<>();
    Set<TypeElement> erasedTargetNames = new LinkedHashSet<>();

    // 1、一系列处理每一个@Bindxxx元素的for循环代码块
    ...

    // Process each @BindView element.
    for (Element element : env.getElementsAnnotatedWith(BindView.class)) {
        try {
        // 2
        parseBindView(element, builderMap, erasedTargetNames);
        } catch (Exception e) {
            logParsingError(element, BindView.class, e);
        }
    }

    // Process each @BindViews element.
    ...

    // Process each annotation that corresponds to a listener.
    for (Class<? extends Annotation> listener : LISTENERS) {
        findAndParseListener(env, listener, builderMap, erasedTargetNames);
    }

    // 2
    Deque<Map.Entry<TypeElement, BindingSet.Builder>> entries =
        new ArrayDeque<>(builderMap.entrySet());
    Map<TypeElement, BindingSet> bindingMap = new LinkedHashMap<>();
    while (!entries.isEmpty()) {
        Map.Entry<TypeElement, BindingSet.Builder> entry = entries.removeFirst();

        TypeElement type = entry.getKey();
        BindingSet.Builder builder = entry.getValue();

        TypeElement parentType = findParentType(type, erasedTargetNames);
        if (parentType == null) {
            bindingMap.put(type, builder.build());
        } else {
            BindingSet parentBinding = bindingMap.get(parentType);
            if (parentBinding != null) {
                builder.setParent(parentBinding);
                bindingMap.put(type, builder.build());
            } else {
            entries.addLast(entry);
            }
        }
    }
    return bindingMap;
}
```

findAndParseTargets()方法的代码非常多，这里尽可能做了精简。首先，在注释1处，**扫描并处理所有具有@Bindxxx注解和符合LISTENERS监听方法集合的代码，然后在每一个@Bindxxx对应的for循环代码中的parseBindxxx()或findAndParseListener()方法中将解析出的信息放入builderMap这个LinkedHashMap对象中**，其中builderMap是一个key为TypeElement，value为BindingSet.Builder的映射集合，这个 BindSet 是指**的一个类型请求的所有绑定的集合**。在注释3处，首先使用上面的builderMap对象去构建了一个entries对象，它是一个双向队列，能实现两端存取的操作。接着，又新建了一个key为TypeElement，value为BindingSet的LinkedHashMap对象，最后使用了一个while循环从entries的第一个元素开始，这里会判断当前元素类型是否有父类，如果没有，直接构建builder放入bindingMap中，如果有，则将parentBinding添加到BindingSet.Builder这个建造者对象中，然后再创建BindingSet再添加到bindingMap中。

接着，分析下注释2处parseBindView是如何对每一个@BindView注解的元素进行处理。

```
private void parseBindView(Element element, Map<TypeElement, BindingSet.Builder> builderMap,
  Set<TypeElement> erasedTargetNames) {
    TypeElement enclosingElement = (TypeElement) element.getEnclosingElement();

    // 1、首先验证生成的常见代码限制
    ...

    // 2、验证目标类型是否继承自View。
    ...
    
    // 3
    int id = element.getAnnotation(BindView.class).value();
    BindingSet.Builder builder = builderMap.get(enclosingElement);
    Id resourceId = elementToId(element, BindView.class, id);
    if (builder != null) {
        String existingBindingName = builder.findExistingBindingName(resourceId);
        if (existingBindingName != null) {
            ...
            return;
        }
    } else {
        // 4
        builder = getOrCreateBindingBuilder(builderMap, enclosingElement);
    }

    String name = simpleName.toString();
    TypeName type = TypeName.get(elementType);
    boolean required = isFieldRequired(element);

    // 5
    builder.addField(resourceId, new     FieldViewBinding(name, type, required));

    // Add the type-erased version to the valid binding targets set.
    erasedTargetNames.add(enclosingElement);
}
```

首先，在注释1、2处均是一些验证处理操作，如果不符合则会return。然后，看到注释3处，这里获取了BindView要绑定的View的id，然后先从builderMap中获取BindingSet.Builder对象，如果存在，直接return。如果不存在，则会在注释4处的 getOrCreateBindingBuilder()方法生成一个。看一下getOrCreateBindingBuilder()方法:

```
private BindingSet.Builder getOrCreateBindingBuilder(
  Map<TypeElement, BindingSet.Builder> builderMap, TypeElement enclosingElement) {
    BindingSet.Builder builder = builderMap.get(enclosingElement);
    if (builder == null) {
        builder = BindingSet.newBuilder(enclosingElement);
        builderMap.put(enclosingElement, builder);
    }
    return builder;
}
```

可以看到，这里会再次从buildMap中获取BindingSet.Builder对象，如果没有则直接调用BindingSet的newBuilder()方法新建一个BindingSet.Builder对象并保存在builderMap中，然后，再将新建的builder对象返回。

回到parseBindView()方法的注释5处，这里根据view的信息生成一个FieldViewBinding，最后添加到上边生成的builder对象中。

最后，再回到我们的process()方法中，现在**所有的绑定的集合数据都放在了bindingMap对象中，这里使用for循环取出每一个BindingSet对象，调用它的brewJava()方法**，看看它内部的处理：

```
JavaFile brewJava(int sdk, boolean debuggable) {
    TypeSpec bindingConfiguration = createType(sdk, debuggable);
    return JavaFile.builder(bindingClassName.packageName(), bindingConfiguration)
    .addFileComment("Generated code from Butter Knife. Do not modify!")
    .build();
}

private TypeSpec createType(int sdk, boolean debuggable) {
    TypeSpec.Builder result = TypeSpec.classBuilder(bindingClassName.simpleName())
    .addModifiers(PUBLIC);
    if (isFinal) {
        result.addModifiers(FINAL);
    }

    if (parentBinding != null) {
        result.superclass(parentBinding.bindingClassName);
    } else {
        result.addSuperinterface(UNBINDER);
    }

    if (hasTargetField()) {
        result.addField(targetTypeName, "target", PRIVATE);
    }

    if (isView) {
        result.addMethod(createBindingConstructorForView());
    } else if (isActivity) {
        result.addMethod(createBindingConstructorForActivity());
    } else if (isDialog) {
        result.addMethod(createBindingConstructorForDialog());
    }
    if (!constructorNeedsView()) {
        // Add a delegating constructor with a target type + view signature for reflective use.
        result.addMethod(createBindingViewDelegateConstructor());
    }
    result.addMethod(createBindingConstructor(sdk, debuggable));

    if (hasViewBindings() || parentBinding == null) {
        result.addMethod(createBindingUnbindMethod(result));
    }

    return result.build();
}
```

在createType()方法里面使用了java中的javapoet技术生成了一个bindingConfiguration对象，很显然，它里面**保存了所有的绑定配置信息。然后，通过javapoet的builder构造器将上面得到的bindingConfiguration对象构建生成一个JavaFile对象，最终，通过javaFile.writeTo(filer)生成了java源文件**。



> 从上面的源码分析来看，ButterKnife的执行流程总体可以分为如下两步：

1. **在编译的时候扫描注解，并通过自定义的ButterKnifeProcessor做相应的处理解析得到bindingMap对象，最后，调用 javapoet 库生成java模板代码**。

2. **当我们调用 ButterKnife的bind()方法的时候，它会根据类的全限定类型，找到相应的模板代码，并在其中完成 findViewById 和 setOnClick ，setOnLongClick 等操作**。

# Dagger 2

## 预备知识

### @Inject

告诉dagger这个字段或类需要依赖注入，然后在需要依赖的地方使用这个注解，dagger会自动生成这个构造器的实例。

#### 获取所需依赖：

- 全局变量注入
- 方法注入

#### 提供所需实例：

- 构造器注入（如果有多个构造函数，只能注解一个，否则编译报错）

### @Module

类注解，表示此类的方法是提供依赖的，它告诉dagger在哪可以找到依赖。用于不能用@Inject提供依赖的地方，如第三方库提供的类，基本数据类型等不能修改源码的情况。

注意：**Dagger2会优先在@Module注解的类上查找依赖，没有的情况才会去查询类的@Inject构造方法**

### @Singleton

声明这是一个单例，**在确保只有一个Component并且不再重新build()之后，对象只会被初始化一次，之后的每次都会被注入相同的对象**，它就是一个内置的作用域。

对于@Singleton，大家可能会产生一些误解，这里详细阐述下：

- Singleton容易给人造成一种误解就是用Singleton注解后在整个Java代码中都是单例，但**实际上他和Scope一样，只是在同一个Component是单例**。也就是说，如果重新调用了component的build（）方法，即使使用了Singleton注解了，但仍然获取的是不同的对象。
- 它表明了**@Singleton注解只是声明了这是一个单例，为的只是提高代码可读性，其实真正控制对象生命周期的还是Component**。同理，自定义的@ActivityScope 、@ApplicationScope也仅仅是一个声明的作用，**真正控制对象生命周期的还是Component**。

### @Providers

只在@Module中使用，用于提供构造好的实例。一般与@Singleton搭配，用单例方法的形式对外提供依赖,是一种替代@Inject注解构造方法的方式。

注意：

- 使用了@Providers的方法应使用provide作为前缀，使用了@Module的类应使用Module作为后缀。
- **如果@Providers方法或@Inject构造方法有参数，要保证它能够被dagger获取到，比如通过其它@Providers方法或者@Inject注解构造器的形式得到**。

### @Component

**@Component作为Dagger2的容器总管，它拥有着@Inject与@Module的所有依赖。同时，它也是一枚注射器，用于获取所需依赖和提供所需依赖的桥梁**。这里的桥梁即指@Inject和@Module（或@Inject构造方法）之间的桥梁。定义时需要列出响应的Module组成，此外，还可以使用dependencies继承父Component。

#### Component与Module的区别：

Component既是注射器也是一个容器总管，而module则是作为容器总管Component的子容器，实质是一个用于提供依赖的模块。

### @Scope

注解作用域，通过自定义注解**限定对象作用范围，增强可读性**。

@Scope有两种常用的使用场景：

- **模拟Singleton代表全局单例，与Component生命周期关联**。
- **模拟局部单例，如登录到退出登录期间**。

### @Qualifier

限定符，利用它**定义注解类以用于区分类的不同实例**。例如：2个方法返回不同的Person对象，比如说小明和小华，为了区分，使用@Qualifier定义的注解类。

### dependencies

使用它表示ChildComponent依赖于FatherComponent，如下所示：

```
@Component(modules = ChildModule.class, dependencies = FatherComponent.class)
public interface ChildComponent {
    ...
}
```

### @SubComponent

表示是一个子@Component，它能**将应用的不同部分封装起来，用来替代@Dependencies**。

## 简单示例

### 首先，创建一个BaseActivityComponent的Subcomponent：

```
@Subcomponent(modules = {AndroidInjectionModule.class})
public interface BaseActivityComponent extends AndroidInjector<BaseActivity> {

    @Subcomponent.Builder
    abstract class BaseBuilder extends AndroidInjector.Builder<BaseActivity>{
    }
}
```

这里必须要注解成@Subcomponent.Builder表示是顶级@Subcomponent的内部类。AndroidInjector.Builder的泛型指定了BaseActivity，即表示每一个继承于BaseActivity的Activity都继承于同一个子组件（BaseActivityComponent）。

### 然后，创建一个将会导入Subcomponent的公有Module。

```
// 1
@Module(subcomponents = {BaseActivityComponent.class})
public abstract class AbstractAllActivityModule {

    @ContributesAndroidInjector(modules = MainActivityModule.class)
    abstract MainActivity contributesMainActivityInjector();

    @ContributesAndroidInjector(modules = SplashActivityModule.class)
    abstract SplashActivity contributesSplashActivityInjector();
    
    // 一系列的对应Activity的contributesxxxActivityInjector
    ...
    
}
```

在注释1处用subcomponents来表示开放全部依赖给AbstractAllActivityModule，使用Subcomponent的重要原因是它将应用的不同部分封装起来了。**@AppComponent负责维护共享的数据和对象，而不同处则由各自的@Subcomponent维护**。

### 接着，配置项目的Application。

```
public class WanAndroidApp extends Application implements HasActivityInjector {

    // 3
    @Inject
    DispatchingAndroidInjector<Activity> mAndroidInjector;

    private static volatile AppComponent appComponent;
    
    @Override
    public void onCreate() {
        super.onCreate();
        
        ...
        // 1
        appComponent = DaggerAppComponent.builder()
            .build();
        // 2
        appComponent.inject(this);
        
        ...
        
    }
    
    ...
    
    // 4
    @Override
    public AndroidInjector<Activity> activityInjector() {
        return mAndroidInjector;
    }
}
```

首先，在注释1处，使用AppModule模块和httpModule模块构建出AppComponent的实现类DaggerAppComponent。这里看一下AppComponent的配置代码：

```
@Singleton
@Component(modules = {AndroidInjectionModule.class,
        AndroidSupportInjectionModule.class,
        AbstractAllActivityModule.class,
        AbstractAllFragmentModule.class,
        AbstractAllDialogFragmentModule.class}
    )
public interface AppComponent {

    /**
     * 注入WanAndroidApp实例
     *
     * @param wanAndroidApp WanAndroidApp
     */
    void inject(WanAndroidApp wanAndroidApp);
    
    ...
    
}
```

可以看到，AppComponent依赖了AndroidInjectionModule模块，它包含了一些基础配置的绑定设置，如activityInjectorFactories、fragmentInjectorFactories等等，而AndroidSupportInjectionModule模块显然就是多了一个supportFragmentInjectorFactories的绑定设置，activityInjectorFactories的内容如所示：

```
@Beta
@Module
public abstract class AndroidInjectionModule {
    @Multibinds
    abstract Map<Class<? extends Activity>, AndroidInjector.Factory<? extends Activity>>
        activityInjectorFactories();
    
    @Multibinds
    abstract Map<Class<? extends Fragment>, AndroidInjector.Factory<? extends Fragment>>
        fragmentInjectorFactories();
    
    ...

}
```

接着，下面依赖的AbstractAllActivityModule、 AbstractAllFragmentModule、AbstractAllDialogFragmentModule则是为项目的所有Activity、Fragment、DialogFragment提供的统一基类抽象Module，这里看下AbstractAllActivityModule的配置：

```
@Module(subcomponents = {BaseActivityComponent.class})
public abstract class AbstractAllActivityModule {

    @ContributesAndroidInjector(modules = MainActivityModule.class)
    abstract MainActivity contributesMainActivityInjector();

    @ContributesAndroidInjector(modules = SplashActivityModule.class)
    abstract SplashActivity contributesSplashActivityInjector();
    
    ...
    
}
```

可以看到，项目下的所有xxxActiviity都有对应的contributesxxxActivityInjector()方法提供实例注入。并且，注意到AbstractAllActivityModule这个模块依赖的 subcomponents为BaseActivityComponent，前面说过了，每一个继承于BaseActivity的Activity都继承于BaseActivityComponent这一个subcomponents。同理，AbstractAllFragmentModule与AbstractAllDialogFragmentModule也是类似的实现模式，如下所示：

```
// 1
@Module(c = BaseFragmentComponent.class)
public abstract class AbstractAllFragmentModule {

    @ContributesAndroidInjector(modules = CollectFragmentModule.class)
    abstract CollectFragment contributesCollectFragmentInject();

    @ContributesAndroidInjector(modules = KnowledgeFragmentModule.class)
    abstract KnowledgeHierarchyFragment contributesKnowledgeHierarchyFragmentInject();
    
    ...
    
}


// 2
@Module(subcomponents = BaseDialogFragmentComponent.class)
public abstract class AbstractAllDialogFragmentModule {

    @ContributesAndroidInjector(modules = SearchDialogFragmentModule.class)
    abstract SearchDialogFragment contributesSearchDialogFragmentInject();

    @ContributesAndroidInjector(modules = UsageDialogFragmentModule.class)
    abstract UsageDialogFragment contributesUsageDialogFragmentInject();

}
```

注意到注释1和注释2处的代码，AbstractAllFragmentModule和AbstractAllDialogFragmentModule的subcomponents为BaseFragmentComponent、BaseDialogFragmentComponent，很显然，同AbstractAllActivityModule的子组件BaseActivityComponent一样，它们都是作为一个通用的子组件。

然后，回到我们配置项目下的Application下面的注释2处的代码，在这里使用了第一步Dagger为我们构建的DaggerAppComponent对象将当期的Application实例注入了进去，交给了Dagger这个依赖大管家去管理。最终，**Dagger2内部创建的mAndroidInjector对象会在注释3处的地方进行实例赋值。在注释4处，实现HasActivityInjector接口，重写activityInjector()方法，将我们上面得到的mAndroidInjector对象返回**。这里的mAndroidInjector是一个类型为DispatchingAndroidInjector的对象，可以这样理解它：它能够执行Android框架下的核心成员如Activity、Fragment的成员注入，在我们项目下的Application中将DispatchingAndroidInjector的泛型指定为Activity就说明它承担起了所有Activity成员依赖的注入。那么，如何指定某一个Activity能被纳入DispatchingAndroidInjector这个所有Activity的依赖总管的口袋中呢？接着看使用步骤4。

### 最后，将目标Activity纳入Activity依赖分配总管DispatchingAndroidInjector的囊中。

很简单，只需在目标Activity的onCreate()方法前的super.onCreate(savedInstanceState)前配置一行代码 AndroidInjection.inject(this)，如下所示：

```
public abstract class BaseActivity<T extends AbstractPresenter> extends AbstractSimpleActivity implements
    AbstractView {

    ...
    @Inject
    protected T mPresenter;
    
    
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        AndroidInjection.inject(this);
        super.onCreate(savedInstanceState);
    }

    ...
    
}
```

这里使用了@Inject表明了需要注入mPresenter实例，然后，我们需要在具体的Presenter类的构造方法上使用@Inject提供基于当前构造方法的mPresenter实例，如下所示：

```
public class MainPresenter extends BasePresenter<MainContract.View> implements MainContract.Presenter {

    ...

    @Inject
    MainPresenter(DataManager dataManager) {
        super(dataManager);
        this.mDataManager = dataManager;
    }

    ...
    
}
```

从上面的使用流程中，有三个关键的核心实现是我们需要了解的，如下所示：

- 1、appComponent = DaggerAppComponent.builder().build()这句代码如何构建出DaggerAPPComponent的？
- 2、appComponent.inject(this)是如何将mAndroidInjector实例赋值给当前的Application的？
- 3、在目标Activity下的AndroidInjection.inject(this)这句代码是如何将当前Activity对象纳入依赖分配总管DispatchingAndroidInjector囊中的呢？

下面我们逐个地来探索其中的奥妙~

## 源码分析

### DaggerAppComponent.builder().build()是如何构建出DaggerAPPComponent的？

首先，看到DaggerAppComponent的builder()方法：

```
public static Builder builder() {
    return new Builder();
}
```

里面直接返回了一个新建的Builder静态内部类对象，看看它的构造方法中做了什么：

```
public static final class Builder {

    private Builder() {}
    
    ...
    
}
```

看来，Builder的默认构造方法什么也没有做，那么，真正的实现肯定在Builder对象的build()方法中，接着看到build()方法。

```
public static final class Builder {

    ...
 
    public AppComponent build() {
         return new DaggerAppComponent(this);
    }

    ...

}
```

在Builder的build()方法中直接返回了新建的DaggerAppComponent对象。下面，看看DaggerAppComponent的构造方法:

```
private DaggerAppComponent(Builder builder) {
    initialize(builder);
}
```

在DaggerAppComponent的构造方法中调用了initialize方法，顾名思义，它就是真正初始化项目全局依赖配置的地方了，下面，来看看它内部的实现：

```
private void initialize(final Builder builder) {
    // 1
    this.mainActivitySubcomponentBuilderProvider =
        new Provider<
            AbstractAllActivityModule_ContributesMainActivityInjector.MainActivitySubcomponent
                .Builder>() {
        @Override
        public AbstractAllActivityModule_ContributesMainActivityInjector.MainActivitySubcomponent
                .Builder
            get() {
                // 2
                return new MainActivitySubcomponentBuilder();
            }
        };

    // 一系列xxxActivitySubcomponentBuilderProvider的创建赋值代码块
    ...

}
```

在注释1处，新建了一个mainActivit的子组件构造器实例提供者Provider。在注释2处，使用匿名内部类的方式重写了该Provider的get()方法，返回一个新创建好的MainActivitySubcomponentBuilder对象。很显然，它就是负责创建管理MAinActivity中所需依赖的Subcomponent建造者。接下来重点来分析下MainActivitySubcomponentBuilder这个类的作用。

```
// 1
private final class MainActivitySubcomponentBuilder
  extends AbstractAllActivityModule_ContributesMainActivityInjector.MainActivitySubcomponent
      .Builder {
    private MainActivity seedInstance;
    
    @Override
    public AbstractAllActivityModule_ContributesMainActivityInjector.MainActivitySubcomponent
        build() {
      if (seedInstance == null) {
        throw new IllegalStateException(MainActivity.class.getCanonicalName() + " must be set");
      }
      // 2
      return new MainActivitySubcomponentImpl(this);
    }
    
    @Override
    public void seedInstance(MainActivity arg0) {
      // 3
      this.seedInstance = Preconditions.checkNotNull(arg0);
    }
}
```

首先，在注释1处，MainActivitySubcomponentBuilder继承了AbstractAllActivityModule_ContributesMainActivityInjector内部的子组件MainActivitySubcomponent的内部的子组件建造者类Builder，如下所示：

```
@Subcomponent(modules = MainActivityModule.class)
public interface MainActivitySubcomponent extends AndroidInjector<MainActivity> {
    @Subcomponent.Builder
    abstract class Builder extends
    AndroidInjector.Builder<MainActivity> {}
}
```

可以看到，这个子组件建造者Builder又继承了AndroidInjector的抽象内部类Builder，那么，这个AndroidInjector到底是什么呢？

顾名思义，**AndroidInjector**是一个Android注射器，它**为每一个具体的子类型，即核心Android类型Activity和Fragment执行成员注入。**

接下来分析下AndroidInjector的内部实现，源码如下所示：

```
public interface AndroidInjector<T> {

    void inject(T instance);
    
    // 1
    interface Factory<T> {
        AndroidInjector<T> create(T instance);
    }
    
    // 2
    abstract class Builder<T> implements AndroidInjector.Factory<T> {
        @Override
        public final AndroidInjector<T> create(T instance) {
            seedInstance(instance);
            return build();
        }
    
        @BindsInstance
        public abstract void seedInstance(T instance);
    
        public abstract AndroidInjector<T> build();
    }
}
```

在注释1处，使用了抽象工厂模式，用来创建一个具体的Activity或Fragment类型的AndroidInjector实例。注释2处，Builder实现了AndroidInjector.Factory，它是一种Subcomponent.Builder的通用实现模式，在重写的create()方法中，进行了实例保存seedInstance()和具体Android核心类型的构建。

接着，我们回到MainActivitySubcomponentBuilder类，可以看到，它实现了AndroidInjector.Builder的seedInstance()和build()方法。在注释3处首先播种了MainActivity的实例，然后 在注释2处新建了一个MainActivitySubcomponentImpl对象返回。我们看看MainActivitySubcomponentImpl这个类是如何将mPresenter依赖注入的，相关源码如下：

```
private final class MainActivitySubcomponentImpl
    implements AbstractAllActivityModule_ContributesMainActivityInjector
    .MainActivitySubcomponent {
      
    private MainPresenter getMainPresenter() {
        // 2
        return MainPresenter_Factory.newMainPresenter(
        DaggerAppComponent.this.provideDataManagerProvider.get());
    }

    @Override
    public void inject(MainActivity arg0) {
        // 1
        injectMainActivity(arg0);
    }

    private MainActivity injectMainActivity(MainActivity instance) {
        // 3
        BaseActivity_MembersInjector
        .injectMPresenter(instance, getMainPresenter());
        return instance;
    }
```

在注释1处，MainActivitySubcomponentImpl实现了AndroidInjector接口的inject()方法，**在injectMainActivity()首先调用getMainPresenter()方法从MainPresenter_Factory工厂类中新建了一个MainPresenter对象**。我们看看MainPresenter的newMainPresenter()方法：

```
public static MainPresenter newMainPresenter(DataManager dataManager) {
    return new MainPresenter(dataManager);
}
```

这里直接新建了一个MainPresenter。然后我们回到MainActivitySubcomponentImpl类的注释3处，继续调用了**BaseActivity_MembersInjector的injectMPresenter()方法**，顾名思义，可以猜到，它是BaseActivity的成员注射器，继续看看injectMPresenter()内部：

```
public static <T extends AbstractPresenter> void injectMPresenter(
  BaseActivity<T> instance, T mPresenter) {
    instance.mPresenter = mPresenter;
}
```

可以看到，这里直接将需要的mPresenter实例赋值给了BaseActivity的mPresenter，当然，这里其实是指的BaseActivity的子类MainActivity，其它的xxxActivity的依赖管理机制都是如此。

### appComponent.inject(this)是如何将mAndroidInjector实例赋值给当前的Application的？

我们继续查看appComponent的inject()方法：

```
@Override
public void inject(WanAndroidApp wanAndroidApp) {
  injectWanAndroidApp(wanAndroidApp);
}
```

在inject()方法里调用了injectWanAndroidApp()，继续查看injectWanAndroidApp()方法：

```
private WanAndroidApp injectWanAndroidApp(WanAndroidApp instance) {
    WanAndroidApp_MembersInjector.injectMAndroidInjector(
        instance,
        getDispatchingAndroidInjectorOfActivity());
    return instance;
}
```

首先，执行getDispatchingAndroidInjectorOfActivity()方法得到了一个Activity类型的DispatchingAndroidInjector对象，继续查看getDispatchingAndroidInjectorOfActivity()方法：

```
private DispatchingAndroidInjector<Activity> getDispatchingAndroidInjectorOfActivity() {
    return DispatchingAndroidInjector_Factory.newDispatchingAndroidInjector(
    getMapOfClassOfAndProviderOfFactoryOf());
}
```

在getDispatchingAndroidInjectorOfActivity()方法里面，首先调用了getMapOfClassOfAndProviderOfFactoryOf()方法，我们看到这个方法：

```
private Map<Class<? extends Activity>, Provider<AndroidInjector.Factory<? extends Activity>>>
  getMapOfClassOfAndProviderOfFactoryOf() {
    return MapBuilder
        .<Class<? extends Activity>, Provider<AndroidInjector.Factory<? extends Activity>>>
        newMapBuilder(8)
        .put(MainActivity.class, (Provider) mainActivitySubcomponentBuilderProvider)
        .put(SplashActivity.class, (Provider) splashActivitySubcomponentBuilderProvider)
        .put(ArticleDetailActivity.class,
            (Provider) articleDetailActivitySubcomponentBuilderProvider)
        .put(KnowledgeHierarchyDetailActivity.class,
            (Provider) knowledgeHierarchyDetailActivitySubcomponentBuilderProvider)
        .put(LoginActivity.class, (Provider) loginActivitySubcomponentBuilderProvider)
        .put(RegisterActivity.class, (Provider) registerActivitySubcomponentBuilderProvider)
        .put(AboutUsActivity.class, (Provider) aboutUsActivitySubcomponentBuilderProvider)
        .put(SearchListActivity.class, (Provider) searchListActivitySubcomponentBuilderProvider)
        .build();
}
```

可以看到，这里新建了一个建造者模式实现的MapBuilder，并且同时制定了固定容量为8，将项目下使用了AndroidInjection.inject(mActivity)方法的8个Activity对应的xxxActivitySubcomponentBuilderProvider保存起来。

我们再回到getDispatchingAndroidInjectorOfActivity()方法，这里将上面得到的Map容器传入了DispatchingAndroidInjector_Factory的newDispatchingAndroidInjector()方法中，这里应该就是新建DispatchingAndroidInjector的地方了。我们点进去看看：

```
public static <T> DispatchingAndroidInjector<T> newDispatchingAndroidInjector(
  Map<Class<? extends T>, Provider<AndroidInjector.Factory<? extends T>>> injectorFactories) {
    return new DispatchingAndroidInjector<T>(injectorFactories);
}
```

在这里，果然新建了一个DispatchingAndroidInjector对象。继续看看DispatchingAndroidInjector的构造方法：

```
@Inject
DispatchingAndroidInjector(
  Map<Class<? extends T>, Provider<AndroidInjector.Factory<? extends T>>> injectorFactories) {
    this.injectorFactories = injectorFactories;
}
```

这里仅仅是将传进来的Map容器保存起来了。

我们再回到WanAndroidApp_MembersInjector的injectMAndroidInjector()方法，将上面得到的DispatchingAndroidInjector实例传入，继续查看injectMAndroidInjector()这个方法：

```
public static void injectMAndroidInjector(
  WanAndroidApp instance, DispatchingAndroidInjector<Activity> mAndroidInjector) {
    instance.mAndroidInjector = mAndroidInjector;
}
```

可以看到，最后在WanAndroidApp_MembersInjector的injectMAndroidInjector()方法中，直接将新建好的DispatchingAndroidInjector实例赋值给了WanAndroidApp的mAndroidInjector。

### 在目标Activity下的AndroidInjection.inject(this)这句代码是如何将当前Activity对象纳入依赖分配总管DispatchingAndroidInjector囊中的呢？

首先，我们看到AndroidInjection.inject(this)这个方法：

```
public static void inject(Activity activity) {
    checkNotNull(activity, "activity");
    
    // 1
    Application application = activity.getApplication();
    if (!(application instanceof HasActivityInjector)) {
    throw new RuntimeException(
        String.format(
            "%s does not implement %s",
            application.getClass().getCanonicalName(), 
            HasActivityInjector.class.getCanonicalName()));
    }

    // 2
    AndroidInjector<Activity> activityInjector =
        ((HasActivityInjector) application).activityInjector();
        
    checkNotNull(activityInjector, "%s.activityInjector() returned null", application.getClass());

    // 3
    activityInjector.inject(activity);
```

}

在注释1处，会先判断当前的application是否实现了HasActivityInjector这个接口，如果没有，则抛出RuntimeException。如果有，会继续在注释2处调用application的activityInjector()方法得到DispatchingAndroidInjector实例。最后，在注释3处，会将当前的activity实例传入activityInjector的inject()方法中。我们继续查看inject()方法：

```
@Override
public void inject(T instance) {
    boolean wasInjected = maybeInject(instance);
    if (!wasInjected) {
        throw new IllegalArgumentException(errorMessageSuggestions(instance));
    }
}
```

**DispatchingAndroidInjector的inject()方法，它的作用就是给传入的instance实例执行成员注入**。具体在这个案例中，其实就是负责将创建好的Presenter实例赋值给BaseActivity对象 的mPresenter全局变量。在inject()方法中，又调用了maybeInject()方法，我们继续查看它：

```
@CanIgnoreReturnValue
public boolean maybeInject(T instance) {
    // 1
    Provider<AndroidInjector.Factory<? extends T>> factoryProvider =
    injectorFactories.get(instance.getClass());
    if (factoryProvider == null) {
    return false;
    }

    @SuppressWarnings("unchecked")
    // 2
    AndroidInjector.Factory<T> factory = (AndroidInjector.Factory<T>) factoryProvider.get();
    try {
        // 3
        AndroidInjector<T> injector =
            checkNotNull(
                factory.create(instance), "%s.create(I) should not return null.", factory.getClass());
        // 4
        injector.inject(instance);
        return true;
    } catch (ClassCastException e) {
        ...
    }
}
```

在注释1处，我们从injectorFactories（前面得到的Map容器）中根据当前Activity实例拿到了factoryProvider对象，这里我们具体一点，看到MainActivity对应的factoryProvider，也就是我们研究的第一个问题中的mainActivitySubcomponentBuilderProvider：

```
private void initialize(final Builder builder) {
    this.mainActivitySubcomponentBuilderProvider =
        new Provider<
            AbstractAllActivityModule_ContributesMainActivityInjector.MainActivitySubcomponent
            .Builder>() {
        @Override
        public AbstractAllActivityModule_ContributesMainActivityInjector.MainActivitySubcomponent
                .Builder
            get() {
                return new MainActivitySubcomponentBuilder();
            }
        };

    ...
    
}
```

在maybeInject()方法的注释2处，调用了mainActivitySubcomponentBuilderProvider的get()方法得到了一个新建的MainActivitySubcomponentBuilder对象。在注释3处执行了它的create方法，create()方法的具体实现在AndroidInjector的内部类Builder中：

```
abstract class Builder<T> implements AndroidInjector.Factory<T> {
    @Override
    public final AndroidInjector<T> create(T instance) {
        seedInstance(instance);
        return build();
    }
```

看到这里，我相信看过第一个问题的同学已经明白后面是怎么回事了。在create()方法中，我们首先MainActivitySubcomponentBuilder的seedInstance()将MainActivity实例注入，然后再调用它的build()方法新建了一个MainActivitySubcomponentImpl实例返回。

最后，在注释4处，执行了MainActivitySubcomponentImpl的inject()方法：

```
private final class MainActivitySubcomponentImpl
    implements AbstractAllActivityModule_ContributesMainActivityInjector
    .MainActivitySubcomponent {
      
    private MainPresenter getMainPresenter() {
        // 2
        return MainPresenter_Factory.newMainPresenter(
        DaggerAppComponent.this.provideDataManagerProvider.get());
    }

    @Override
    public void inject(MainActivity arg0) {
        // 1
        injectMainActivity(arg0);
    }

    private MainActivity injectMainActivity(MainActivity instance) {
        // 3
        BaseActivity_MembersInjector
        .injectMPresenter(instance, getMainPresenter());
        return instance;
    }
```

这里的逻辑已经在问题一的最后部分详细讲解了，最后，会在注释3处调用BaseActivity_MembersInjector的injectMPresenter()方法：

```
public static <T extends AbstractPresenter> void injectMPresenter(
  BaseActivity<T> instance, T mPresenter) {
    instance.mPresenter = mPresenter;
}
```

这样，就将mPresenter对象赋值给了当前Activity对象的mPresenter全局变量中了。至此，Dagger.Android的核心源码分析就结束了。



> 相比于ButterKnife，Dagger是一个**锋利的全局依赖注入管理框架**，它主要用来**管理对象的依赖关系和生命周期**，当项目越来越大时，类之间的调用层次会越来越深，并且有些类是Activity或Fragment，有些是单例，而且它们的生命周期不一致，所以创建所需对象时需要处理的各个对象的依赖关系和生命周期时的任务会很繁重。因此，使用Dagger会大大减轻这方面的工作量。虽然它的学习成本比较高，而且需要写一定的模板类，但是，**对于越大的项目来说，Dagger越值得被需要**。

# EventBus

## 简单示例

### 首先，定义要传递的事件实体

```
public class CollectEvent { ... }
```

### 准备订阅者：声明并注解你的订阅方法

```
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessageEvent(CollectEvent event) {
    LogHelper.d("OK");
}
```

### 在2中，也就是订阅中所在的类中，注册和解注册你的订阅者

```
@Override
public void onStart() {
    super.onStart();
    EventBus.getDefault().register(this);
}

@Override
public void onStop() {
    super.onStop();
    EventBus.getDefault().unregister(this);
}
```

### 发送事件

```
EventBus.getDefault().post(new CollectEvent());
```

在正式讲解之前需要对一些基础性的概念进行详细的讲解。众所周知，EventBus没出现之前，那时候的开发者一般是使用Android四大组件中的广播进行组件间的消息传递，那么我们**为什么要使用事件总线机制来替代广播呢**？

主要是因为：

- 广播：耗时、容易被捕获（不安全）。
- 事件总线：更节省资源、更高效，能将信息传递给原生以外的各种对象。

那么，话又说回来了，**事件总线又是什么呢？**

如下图所示，事件总线机制通过记录对象、使用观察者模式来通知对象各种事件。（当然，你也可以发送基本数据类型如 int，String 等作为一个事件）

![image](https://user-gold-cdn.xitu.io/2020/3/6/170ada0907b50b75?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

对于**事件总线EventBus**而言，它的**优缺点**又是如何？这里简单总结下：

- 优点：开销小，代码更优雅、简洁，解耦发送者和接收者，可动态设置事件处理线程和优先级。
- 缺点：每个事件必须自定义一个事件类，增加了维护成本。

EventBus是基于观察者模式扩展而来的，我们先了解一下观察者模式是什么？

观察者模式又可称为**发布 - 订阅模式**，它定义了对象间的一种1对多的依赖关系，每当这个对象的状态改变时，其它的对象都会接收到通知并被自动更新。

观察者模式有以下角色：

- 抽象被观察者：将所有已注册的观察者对象保存在一个集合中。
- 具体被观察者：当内部状态发生变化时，将会通知所有已注册的观察者。
- 抽象观察者：定义了一个更新接口，当被观察者状态改变时更新自己。
- 具体观察者：实现抽象观察者的更新接口。

这里给出一个简单的示例来让大家更深一步理解观察者模式的思想：

1、首先，创建抽象观察者

```
public interface observer {
    
    public void update(String message);
}
```

2、接着，创建具体观察者

```
public class WeXinUser implements observer {
    private String name;
    
    public WeXinUser(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String message) {
        ...
    }
}
```

3、然后，创建抽象被观察者

```
public interface observable {
    
    public void addWeXinUser(WeXinUser weXinUser);
    
    public void removeWeXinUser(WeXinUser weXinUser);
    
    public void notify(String message);
}
```

4、最后，创建具体被观察者

```
public class Subscription implements observable {
    private List<WeXinUser> mUserList = new ArrayList();
    
    @Override
    public void addWeXinUser(WeXinUser weXinUser) {
        mUserList.add(weXinUser);
    }
    
    @Override
    public void removeWeXinUser(WeXinUser weXinUser) {
        mUserList.remove(weXinUser);
    }
    
    @Override
    public void notify(String message) {
        for(WeXinUser weXinUser : mUserList) {
            weXinUser.update(message);
        }
    }
}
```

在具体使用时，我们便可以这样使用，如下所示：

```
Subscription subscription = new Subscription();

WeXinUser hongYang = new WeXinUser("HongYang");
WeXinUser rengYuGang = new WeXinUser("RengYuGang");
WeXinUser liuWangShu = new WeXinUser("LiuWangShu");

subscription.addWeiXinUser(hongYang);
subscription.addWeiXinUser(rengYuGang);
subscription.addWeiXinUser(liuWangShu);
subscription.notify("New article coming");
```

在这里，hongYang、rengYuGang、liuWangShu等大神都订阅了我的微信公众号，每当我的公众号发表文章时（subscription.notify())，他们就会接收到最新的文章信息（weXinUser.update()）。（ps：当然，这一切都是YY~）

当然，EventBus的观察者模式和一般的观察者模式不同，它使用了**扩展的观察者模式对事件进行订阅和分发，其实这里的扩展就是指的使用了EventBus来作为中介者，抽离了许多职责**，如下是它的官方原理图：

![image](https://user-gold-cdn.xitu.io/2020/3/6/170ada0907c082ed?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

在得知了EventBus的原理之后，我们注意到，每次我们在register之后，都必须进行一次unregister，这是为什么呢？

**因为register是强引用，它会让对象无法得到内存回收，导致内存泄露。所以必须在unregister方法中释放对象所占的内存**。

有些同学可能之前使用的是EventBus2.x的版本，那么它又与EventBus3.x的版本有哪些区别呢？

1. EventBus2.x使用的是**运行时注解，它采用了反射的方式对整个注册的类的所有方法进行扫描来完成注册，因而会对性能有一定影响**。

2. EventBus3.x使用的是**编译时注解，Java文件会编译成.class文件，再对class文件进行打包等一系列处理。在编译成.class文件时，EventBus会使用EventBusAnnotationProcessor注解处理器读取@Subscribe()注解并解析、处理其中的信息，然后生成Java类来保存所有订阅者的订阅信息。这样就创建出了对文件或类的索引关系，并将其编入到apk中**。

3. 从EventBus3.0开始**使用了对象池缓存减少了创建对象的开销**。

除了EventBus，其实现在比较流行的事件总线还有RxBus，那么，它与EventBus相比又如何呢？

1. **RxJava的Observable有onError、onComplete等状态回调**。

2. **Rxjava使用组合而非嵌套的方式，避免了回调地狱**。

3. **Rxjava的线程调度设计的更加优秀，更简单易用**。

4. **Rxjava可使用多种操作符来进行链式调用来实现复杂的逻辑**。

5. **Rxjava的信息效率高于EventBus2.x，低于EventBus3.x**。

在了解了EventBus和RxBus的区别之后，那么，对待新项目的事件总线选型时，我们该如何考量？

很简单，**如果项目中使用了RxJava，则使用RxBus，否则使用EventBus3.x**。

## 源码分析

接下来将按以下顺序来进行EventBus的源码分析：

1. 订阅者：EventBus.getDefault().register(this)；

2. 发布者：EventBus.getDefault().post(new CollectEvent())；

3. 订阅者：EventBus.getDefault().unregister(this)。

### EventBus.getDefault().register(this)

首先，从获取EventBus实例的方法getDefault()开始分析：

```
public static EventBus getDefault() {
    if (defaultInstance == null) {
        synchronized (EventBus.class) {
            if (defaultInstance == null) {
                defaultInstance = new EventBus();
            }
        }
    }
    return defaultInstance;
}
```

在getDefault()中使用了双重校验并加锁的单例模式来创建EventBus实例。

接着，看到EventBus的默认构造方法中做了什么:

```
private static final EventBusBuilder DEFAULT_BUILDER = new EventBusBuilder();

public EventBus() {
    this(DEFAULT_BUILDER);
}
```

在EventBus的默认构造方法中又调用了它的另一个有参构造方法，将一个类型为EventBusBuilder的DEFAULT_BUILDER对象传递进去了。这里的EventBusBuilder很明显是一个EventBus的建造器，以便于EventBus能够添加自定义的参数和安装一个自定义的默认EventBus实例。

再看一下EventBusBuilder的构造方法：

```
public class EventBusBuilder {

    ...

    EventBusBuilder() {
    }
    
    ...
    
}
```

EventBusBuilder的构造方法中什么也没有做，那继续查看EventBus的这个有参构造方法：

```
private final Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType;
private final Map<Object, List<Class<?>>> typesBySubscriber;
private final Map<Class<?>, Object> stickyEvents;

EventBus(EventBusBuilder builder) {
    ...
    
    // 1
    subscriptionsByEventType = new HashMap<>();
    
    // 2
    typesBySubscriber = new HashMap<>();
    
    // 3
    stickyEvents = new ConcurrentHashMap<>();
    
    // 4
    mainThreadSupport = builder.getMainThreadSupport();
    mainThreadPoster = mainThreadSupport != null ? mainThreadSupport.createPoster(this) : null;
    backgroundPoster = new BackgroundPoster(this);
    asyncPoster = new AsyncPoster(this);
    
    ...
    
    // 5
    subscriberMethodFinder = new SubscriberMethodFinder(builder.subscriberInfoIndexes,
            builder.strictMethodVerification, builder.ignoreGeneratedIndex);
   
    // 从builder取中一些列订阅相关信息进行赋值
    ...
   
    // 6
    executorService = builder.executorService;
}
```

在注释1处，创建了一个subscriptionsByEventType对象，可以看到它是一个类型为HashMap的subscriptionsByEventType对象，并且其key为 Event 类型，value为 Subscription链表。这里的Subscription是一个订阅信息对象，它里面保存了两个重要的字段，一个是类型为 Object 的 subscriber，该字段即为注册的对象（在 Android 中时通常是 Activity对象）；另一个是 类型为SubscriberMethod 的 subscriberMethod，它就是被@Subscribe注解的那个订阅方法，里面保存了一个重要的字段eventType，它是 Class<?> 类型的，代表了 Event 的类型。在注释2处，新建了一个类型为 Map 的typesBySubscriber对象，它的key为subscriber对象，value为subscriber对象中所有的 Event 类型链表，日常使用中仅用于判断某个对象是否注册过。在注释3处新建了一个类型为ConcurrentHashMap的stickyEvents对象，它是专用于粘性事件处理的一个字段，key为事件的Class对象，value为当前的事件。可能有的同学不了解sticky event，这里解释下：

- 我们都知道**普通事件是先注册，然后发送事件才能收到；而粘性事件，在发送事件之后再订阅该事件也能收到。并且，粘性事件会保存在内存中，每次进入都会去内存中查找获取最新的粘性事件，除非你手动解除注册**。

在注释4处，新建了三个不同类型的事件发送器，这里总结下：

- mainThreadPoster：主线程事件发送器，通过它的mainThreadPoster.enqueue(subscription, event)方法可以将订阅信息和对应的事件进行入队，然后通过 handler 去发送一个消息，在 handler 的 handleMessage 中去执行方法。
- backgroundPoster：后台事件发送器，通过它的enqueue() 将方法加入到后台的一个队列，最后通过线程池去执行，注意，它在 Executor的execute()方法 上添加了 synchronized关键字 并设立 了控制标记flag，保证任一时间只且仅能有一个任务会被线程池执行。
- asyncPoster：实现逻辑类似于backgroundPoster，不同于backgroundPoster的保证任一时间只且仅能有一个任务会被线程池执行的特性，asyncPoster则是异步运行的，可以同时接收多个任务。

我们再回到注释5这行代码，这里新建了一个subscriberMethodFinder对象，这是从EventBus中抽离出的订阅方法查询的一个对象，在优秀的源码中，我们经常能看到**组合优于继承**的这种实现思想。在注释6处，从builder中取出了一个默认的线程池对象，它由**Executors的newCachedThreadPool()\**方法创建，它是一个\**有则用、无则创建、无数量上限**的线程池。

分析完这些核心的字段之后，后面的讲解就比较轻松了，接着查看EventBus的regist()方法：

```
public void register(Object subscriber) {
    Class<?> subscriberClass = subscriber.getClass();
    
    // 1
    List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
    synchronized (this) {
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            // 2
            subscribe(subscriber, subscriberMethod);
        }
    }
}
```

在注释1处，根据当前注册类获取 subscriberMethods这个订阅方法列表 。在注释2处，使用了增强for循环令subsciber对象 对 subscriberMethods 中每个 SubscriberMethod 进行订阅。

接着查看SubscriberMethodFinder的findSubscriberMethods()方法：

```
List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
    // 1
    List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
    if (subscriberMethods != null) {
        return subscriberMethods;
    }

    // 2
    if (ignoreGeneratedIndex) {
        subscriberMethods = findUsingReflection(subscriberClass);
    } else {
        subscriberMethods = findUsingInfo(subscriberClass);
    }
    if (subscriberMethods.isEmpty()) {
        throw new EventBusException("Subscriber " + subscriberClass
                + " and its super classes have no public methods with the @Subscribe annotation");
    } else {
        METHOD_CACHE.put(subscriberClass, subscriberMethods);
        return subscriberMethods;
    }
}
```

在注释1处，如果缓存中有subscriberClass对象对应 的订阅方法列表，则直接返回。注释2处，先详细说说这个**ignoreGeneratedIndex**字段， 它用来**判断是否使用生成的 APT 代码去优化寻找接收事件的过程，如果开启了的话，那么将会通过 subscriberInfoIndexes 来快速得到接收事件方法的相关信息**。如果我们没有在项目中接入 EventBus 的 APT，那么可以将 ignoreGeneratedIndex 字段设为 false 以提高性能。这里ignoreGeneratedIndex 默认为false，所以会执行findUsingInfo()方法，后面生成 subscriberMethods 成功的话会加入到缓存中，失败的话会 抛出异常。

接着查看SubscriberMethodFinder的findUsingInfo()方法：

```
private List<SubscriberMethod> findUsingInfo(Class<?> subscriberClass) {
    // 1
    FindState findState = prepareFindState();
    findState.initForSubscriber(subscriberClass);
    // 2
    while (findState.clazz != null) {
        findState.subscriberInfo = getSubscriberInfo(findState);
        if (findState.subscriberInfo != null) {
            SubscriberMethod[] array = findState.subscriberInfo.getSubscriberMethods();
            for (SubscriberMethod subscriberMethod: array) {
                if (findState.checkAdd(subscriberMethod.method, subscriberMethod.eventType)) {
                    findState.subscriberMethods.add(subscriberMethod);
                }
            }
        } else {
             // 3
             findUsingReflectionInSingleClass(findState);
        }
        findState.moveToSuperclass();
    }
    // 4
    return getMethodsAndRelease(findState);
}
```

在注释1处，调用了SubscriberMethodFinder的prepareFindState()方法创建了一个新的 FindState 类，来看看这个方法：

```
private static final FindState[] FIND_STATE_POOL = new FindState[POOL_SIZE];
private FindState prepareFindState() {
    // 1
    synchronized(FIND_STATE_POOL) {
        for (int i = 0; i < POOL_SIZE; i++) {
            FindState state = FIND_STATE_POOL[i];
            if (state != null) {
                FIND_STATE_POOL[i] = null;
                return state;
            }
        }
    }
    // 2
    return new FindState();
}
```

在注释1处，会先从 FIND_STATE_POOL 即 FindState 池中取出可用的 FindState（这里的POOL_SIZE为4），如果没有的话，则通过注释2处的代码直接新建 一个新的 FindState 对象。

接着来分析下FindState这个类：

```
static class FindState {
    ....
    void initForSubscriber(Class<?> subscriberClass) {
        this.subscriberClass = clazz = subscriberClass;
        skipSuperClasses = false;
        subscriberInfo = null;
    }
    ...
}
```

它是 SubscriberMethodFinder 的内部类，这个方法主要做一个初始化、回收对象等工作。

接着回到SubscriberMethodFinder的注释2处的SubscriberMethodFinder()方法：

```
private SubscriberInfo getSubscriberInfo(FindState findState) {
    if (findState.subscriberInfo != null && findState.subscriberInfo.getSuperSubscriberInfo() != null) {
        SubscriberInfo superclassInfo = findState.subscriberInfo.getSuperSubscriberInfo();
        if (findState.clazz == superclassInfo.getSubscriberClass()) {
            return superclassInfo;
        }
    }
    if (subscriberInfoIndexes != null) {
        for (SubscriberInfoIndex index: subscriberInfoIndexes) {
            SubscriberInfo info = index.getSubscriberInfo(findState.clazz);
            if (info != null) {
                return info;
            }
        }
    }
    return null;
}
```

在前面初始化的时候，findState的subscriberInfo和subscriberInfoIndexes 这两个字段为空，所以这里直接返回 null。

接着查看注释3处的findUsingReflectionInSingleClass()方法：

```
private void findUsingReflectionInSingleClass(FindState findState) {
    Method[] methods;
    try {
        // This is faster than getMethods, especially when subscribers are fat classes like Activities
        methods = findState.clazz.getDeclaredMethods();
    } catch (Throwable th) {
        methods = findState.clazz.getMethods();
        findState.skipSuperClasses = true;
    }
    for (Method method: methods) {
        int modifiers = method.getModifiers();
        if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
            Class<?> [] parameterTypes = method.getParameterTypes();
            if (parameterTypes.length == 1) {
                Subscribe subscribeAnnotation = method.getAnnotation(Subscribe.class);
                if (subscribeAnnotation != null) {
                    // 重点
                    Class<?> eventType = parameterTypes[0];
                    if (findState.checkAdd(method, eventType)) {
                        ThreadMode threadMode = subscribeAnnotation.threadMode();
                        findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode, subscribeAnnotation.priority(),  subscribeAnnotation.sticky()));
                    }
                }
            } else if (strictMethodVerification &&     method.isAnnotationPresent(Subscribe.class)) {
            String methodName = method.getDeclaringClass().getName() + "." + method.getName();
            throw new EventBusException("@Subscribe method " + methodName + "must have exactly 1 parameter but has " + parameterTypes.length);
            }
        } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
            String methodName = method.getDeclaringClass().getName() + "." + method.getName();
            throw new EventBusException(methodName + " is a illegal @Subscribe method: must be public, non-static, and non-abstract");
        }
    }
}
```

这个方法很长，大概做的事情是：

1. **通过反射的方式获取订阅者类中的所有声明方法，然后在这些方法里面寻找以 @Subscribe作为注解的方法进行处理**。

2. **在经过经过一轮检查，看看 findState.subscriberMethods是否存在，如果没有，将方法名，threadMode，优先级，是否为 sticky 方法等信息封装到 SubscriberMethod 对象中，最后添加到 subscriberMethods 列表中**。

最后，继续查看注释4处的getMethodsAndRelease()方法：

```
private List<SubscriberMethod> getMethodsAndRelease(FindState findState) {
    // 1
    List<SubscriberMethod> subscriberMethods = new ArrayList<>(findState.subscriberMethods);
    // 2
    findState.recycle();
    // 3
    synchronized(FIND_STATE_POOL) {
        for (int i = 0; i < POOL_SIZE; i++) {
            if (FIND_STATE_POOL[i] == null) {
                FIND_STATE_POOL[i] = findState;
                break;
            }
        }
    }
    // 4
    return subscriberMethods;
}
```

在这里，首先在注释1处，从findState中取出了保存的subscriberMethods。在注释2处，将findState里的保存的所有对象进行回收。在注释3处，把findState存储在 FindState 池中方便下一次使用，以提高性能。最后，在注释4处，返回subscriberMethods。接着，**在EventBus的 register() 方法的最后会调用 subscribe 方法**：

```
public void register(Object subscriber) {
    Class<?> subscriberClass = subscriber.getClass();
    List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
    synchronized (this) {
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod);
        }
    }
}
```

继续看看这个subscribe()方法做的事情：

```
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
    Class<?> eventType = subscriberMethod.eventType;
    Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
    
    // 1
    CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    if (subscriptions == null) {
        subscriptions = new CopyOnWriteArrayList <> ();
        subscriptionsByEventType.put(eventType, subscriptions);
    } else {
        if (subscriptions.contains(newSubscription)) {
            throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event " + eventType);
        }
    }
    int size = subscriptions.size();
    
    // 2
    for (int i = 0; i <= size; i++) {
        if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
            subscriptions.add(i, newSubscription);
            break;
        }
    }
    
    // 3
    List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
    if (subscribedEvents == null) {
        subscribedEvents = new ArrayList<>();
        typesBySubscriber.put(subscriber, subscribedEvents);
    }
    subscribedEvents.add(eventType);
    // 4
    if (subscriberMethod.sticky) {
        if (eventInheritance) {
            Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
            for (Map.Entry<Class<?>, Object> entry : entries) {
                Class<?> candidateEventType = entry.getKey();
                if(eventType.isAssignableFrom(candidateEventType)) {
                Object stickyEvent = entry.getValue();
                    checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                }
            }
        } else {
            Object stickyEvent = stickyEvents.get(eventType);
            checkPostStickyEventToSubscription(newSubscription, stickyEvent);
        }
    }
}
```

首先，在注释1处，会根据 subscriberMethod的eventType，在 subscriptionsByEventType 去查找一个 CopyOnWriteArrayList ，如果没有则创建一个新的 CopyOnWriteArrayList，然后将这个 CopyOnWriteArrayList 放入 subscriptionsByEventType 中。在注释2处，**添加 newSubscription对象，它是一个 Subscription 类，里面包含着 subscriber 和 subscriberMethod 等信息，并且这里有一个优先级的判断，说明它是按照优先级添加的。优先级越高，会插到在当前 List 靠前面的位置**。在注释3处，对typesBySubscriber 进行添加，这主要是在EventBus的isRegister()方法中去使用的，目的是用来判断这个 Subscriber对象 是否已被注册过。最后，在注释4处，会判断是否是 sticky事件。如果是sticky事件的话，会调用 checkPostStickyEventToSubscription() 方法。

接着查看这个checkPostStickyEventToSubscription()方法：

```
private void checkPostStickyEventToSubscription(Subscription newSubscription, Object stickyEvent) {
    if (stickyEvent != null) {
        postToSubscription(newSubscription, stickyEvent, isMainThread());
    }
}
```

可以看到最终是**调用了postToSubscription()这个方法来进行粘性事件的发送**，对于粘性事件的处理，最后再分析，接下来看看事件是如何post的。

### EventBus.getDefault().post(new CollectEvent())

```
public void post(Object event) {
    // 1
    PostingThreadState postingState = currentPostingThreadState.get();
    List <Object> eventQueue = postingState.eventQueue;
    eventQueue.add(event);
    // 2
    if (!postingState.isPosting) {
        postingState.isMainThread = isMainThread();
        postingState.isPosting = true;
        if (postingState.canceled) {
            throw new EventBusException("Internal error. Abort state was not reset");
        }
        try {
            while (!eventQueue.isEmpty()) {
                postSingleEvent(eventQueue.remove(0), postingState);
            }
        } finally {
            postingState.isPosting = false;
            postingState.isMainThread = false;
        }
    }
}
```

注释1处，这里的currentPostingThreadState 是一个 ThreadLocal 类型的对象，里面存储了 PostingThreadState，而 PostingThreadState 中包含了一个 eventQueue 和其他一些标志位，相关的源码如下：

```
private final ThreadLocal <PostingThreadState> currentPostingThreadState = new ThreadLocal <PostingThreadState> () {
@Override
protected PostingThreadState initialValue() {
    return new PostingThreadState();
}
};

final static class PostingThreadState {
    final List <Object> eventQueue = new ArrayList<>();
    boolean isPosting;
    boolean isMainThread;
    Subscription subscription;
    Object event;
    boolean canceled;
}
```

接着把传入的 event，保存到了当前线程中的一个变量 PostingThreadState 的 eventQueue 中。在注释2处，最后调用了 postSingleEvent() 方法，我们继续查看这个方法：

```
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
    Class<?> eventClass = event.getClass();
    boolean subscriptionFound = false;
    // 1
    if (eventInheritance) {
        // 2
        List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
        int countTypes = eventTypes.size();
        for (int h = 0; h < countTypes; h++) {
            Class<?> clazz = eventTypes.get(h);
            subscriptionFound |=
            // 3
            postSingleEventForEventType(event, postingState, clazz);
        }
    } else {
        subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
    }
    if (!subscriptionFound) {
        ...
    }
}
```

首先，在注释1处，首先取出 Event 的 class 类型，接着**会对 eventInheritance 标志位 判断，它默认为true，如果设为 true 的话，它会在发射事件的时候判断是否需要发射父类事件，设为 false，能够提高一些性能**。接着，在注释2处，会调用lookupAllEventTypes() 方法，它的作用就是取出 Event 及其父类和接口的 class 列表，当然重复取的话会影响性能，所以它也做了一个 eventTypesCache 的缓存，这样就不用重复调用 getSuperclass() 方法。最后，在注释3处会调用postSingleEventForEventType()方法，看下这个方法：

```
private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class <?> eventClass) {
    CopyOnWriteArrayList <Subscription> subscriptions;
    synchronized(this) {
        subscriptions = subscriptionsByEventType.get(eventClass);
    }
    if (subscriptions != null && !subscriptions.isEmpty()) {
        for (Subscription subscription: subscriptions) {
            postingState.event = event;
            postingState.subscription = subscription;
            boolean aborted = false;
            try {
                postToSubscription(subscription, event, postingState.isMainThread);
                aborted = postingState.canceled;
            } finally {
                postingState.event = null;
                postingState.subscription = null;
                postingState.canceled = false;
            }
            if (aborted) {
                break;
            }
        }
        return true;
    }
    return false;
}
```

可以看到，这里直接根据 Event 类型从 subscriptionsByEventType 中取出对应的 subscriptions对象，最后调用了 postToSubscription() 方法。

这个时候再看看这个postToSubscription()方法：

```
private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
    switch (subscription.subscriberMethod.threadMode) {
        case POSTING:
            invokeSubscriber(subscription, event);
            break;
        case MAIN:
            if (isMainThread) {
                invokeSubscriber(subscription, event);
            } else {
                mainThreadPoster.enqueue(subscription, event);
            }
            break;
        case MAIN_ORDERED:
            if (mainThreadPoster != null) {
                mainThreadPoster.enqueue(subscription, event);
            } else {
                invokeSubscriber(subscription, event);
            }
            break;
        case BACKGROUND:
            if (isMainThread) {
                backgroundPoster.enqueue(subscription, event);
            } else {
                invokeSubscriber(subscription, event);
            }
            break;
        case ASYNC:
            asyncPoster.enqueue(subscription, event);
            break;
        default:
            throw new IllegalStateException("Unknow thread mode: " + subscription.subscriberMethod.threadMode);
    }
}
```

从上面可以看出，这里通过threadMode 来判断在哪个线程中去执行方法：

1. POSTING：执行 invokeSubscriber() 方法，内部**直接采用反射调用**。

2. MAIN：**首先去判断当前是否在 UI 线程，如果是的话则直接反射调用，否则调用mainThreadPoster的enqueue()方法，即把当前的方法加入到队列之中，然后通过 handler 去发送一个消息，在 handler 的 handleMessage 中去执行方法**。

3. MAIN_ORDERED：**与MAIN类似，不过是确保是顺序执行的**。

4. BACKGROUND：**判断当前是否在 UI 线程，如果不是的话则直接反射调用，是的话通过backgroundPoster的enqueue()方法 将方法加入到后台的一个队列，最后通过线程池去执行。注意，backgroundPoster在 Executor的execute()方法 上添加了 synchronized关键字 并设立 了控制标记flag，保证任一时间只且仅能有一个任务会被线程池执行**。

5. ASYNC：**逻辑实现类似于BACKGROUND，将任务加入到后台的一个队列，最终由Eventbus 中的一个线程池去调用，这里的线程池与 BACKGROUND 逻辑中的线程池用的是同一个，即使用Executors的newCachedThreadPool()方法创建的线程池，它是一个有则用、无则创建、无数量上限的线程池。不同于backgroundPoster的保证任一时间只且仅能有一个任务会被线程池执行的特性，这里asyncPoster则是异步运行的，可以同时接收多个任务**。

分析完EventBus的post()方法值，接着看看它的unregister()。

### EventBus.getDefault().unregister(this)

它的核心源码如下所示：

```
public synchronized void unregister(Object subscriber) {
    List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
    if (subscribedTypes != null) {
        for (Class<?> eventType : subscribedTypes) {
            //1
            unsubscribeByEventType(subscriber, eventType);
        }
        // 2
        typesBySubscriber.remove(subscriber);
    }
}
```

首先，在注释1处，**unsubscribeByEventType() 方法中对 subscriptionsByEventType 移除了该 subscriber 的所有订阅信息**。最后，在注释2处，**移除了注册对象和其对应的所有 Event 事件链表**。

最后，再来分析下EventBus中对粘性事件的处理。

### EventBus.getDefault.postSticky(new CollectEvent())

如果想要发射 sticky 事件需要通过 EventBus的postSticky() 方法，内部源码如下所示：

```
public void postSticky(Object event) {
    synchronized (stickyEvents) {
        // 1
        stickyEvents.put(event.getClass(), event);
    }
    // 2
    post(event);
}
```

在注释1处，先将该事件放入 stickyEvents 中，接着在注释2处使用post()发送事件。前面我们在分析register()方法的最后部分时，其中有关粘性事件的源码如下：

```
if (subscriberMethod.sticky) {
    Object stickyEvent = stickyEvents.get(eventType);
    if (stickyEvent != null) {
        postToSubscription(newSubscription, stickyEvent, isMainThread());
    }
}
```

可以看到，在这里**会判断当前事件是否是 sticky 事件，如果是，则从 stickyEvents 中拿出该事件并执行 postToSubscription() 方法**。



> EventBus 的源码在Android主流三方库源码分析系列中可以说是除了ButterKnife之外，算是比较简单的了。但是，它其中的一些思想和设计是值得借鉴的。比如**它使用 FindState 复用池来复用 FindState 对象，在各处使用了 synchronized 关键字进行代码块同步的一些优化操作**。其中上面分析了这么多，**EventBus最核心的逻辑就是利用了 subscriptionsByEventType 这个重要的列表，将订阅对象，即接收事件的方法存储在这个列表，发布事件的时候在列表中查询出相对应的方法并执行**。
