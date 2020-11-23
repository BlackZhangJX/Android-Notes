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
  - [GreenDao是如何与ReactiveX结合？](#GreenDao是如何与ReactiveX结合)
- [RxJava](#RxJava)
  - [RxJava是什么？](#RxJava是什么)
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
    - [ButterKnife是如何在编译时生成代码的？](#ButterKnife是如何在编译时生成代码的)
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
    timeout.enter
