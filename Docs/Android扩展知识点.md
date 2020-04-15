- [ART](#art)
  - [ART 功能](#art-%e5%8a%9f%e8%83%bd)
    - [预先 (AOT) 编译](#%e9%a2%84%e5%85%88-aot-%e7%bc%96%e8%af%91)
    - [垃圾回收优化](#%e5%9e%83%e5%9c%be%e5%9b%9e%e6%94%b6%e4%bc%98%e5%8c%96)
    - [开发和调试方面的优化](#%e5%bc%80%e5%8f%91%e5%92%8c%e8%b0%83%e8%af%95%e6%96%b9%e9%9d%a2%e7%9a%84%e4%bc%98%e5%8c%96)
  - [ART GC](#art-gc)
- [Apk 包体优化](#apk-%e5%8c%85%e4%bd%93%e4%bc%98%e5%8c%96)
  - [Apk 组成结构](#apk-%e7%bb%84%e6%88%90%e7%bb%93%e6%9e%84)
  - [整体优化](#%e6%95%b4%e4%bd%93%e4%bc%98%e5%8c%96)
  - [资源优化](#%e8%b5%84%e6%ba%90%e4%bc%98%e5%8c%96)
  - [代码优化](#%e4%bb%a3%e7%a0%81%e4%bc%98%e5%8c%96)
  - [.arsc文件优化](#arsc%e6%96%87%e4%bb%b6%e4%bc%98%e5%8c%96)
  - [lib目录优化](#lib%e7%9b%ae%e5%bd%95%e4%bc%98%e5%8c%96)
- [Hook](#hook)
  - [基本流程](#%e5%9f%ba%e6%9c%ac%e6%b5%81%e7%a8%8b)
  - [使用示例](#%e4%bd%bf%e7%94%a8%e7%a4%ba%e4%be%8b)
- [Proguard](#proguard)
  - [公共模板](#%e5%85%ac%e5%85%b1%e6%a8%a1%e6%9d%bf)
  - [常用的自定义混淆规则](#%e5%b8%b8%e7%94%a8%e7%9a%84%e8%87%aa%e5%ae%9a%e4%b9%89%e6%b7%b7%e6%b7%86%e8%a7%84%e5%88%99)
  - [aar中增加独立的混淆配置](#aar%e4%b8%ad%e5%a2%9e%e5%8a%a0%e7%8b%ac%e7%ab%8b%e7%9a%84%e6%b7%b7%e6%b7%86%e9%85%8d%e7%bd%ae)
  - [检查混淆和追踪异常](#%e6%a3%80%e6%9f%a5%e6%b7%b7%e6%b7%86%e5%92%8c%e8%bf%bd%e8%b8%aa%e5%bc%82%e5%b8%b8)
- [架构](#%e6%9e%b6%e6%9e%84)
  - [MVC](#mvc)
  - [MVP](#mvp)
  - [MVVM](#mvvm)
- [Jetpack](#jetpack)
  - [架构](#%e6%9e%b6%e6%9e%84-1)
  - [使用示例](#%e4%bd%bf%e7%94%a8%e7%a4%ba%e4%be%8b-1)
- [NDK 开发](#ndk-%e5%bc%80%e5%8f%91)
  - [JNI 基础](#jni-%e5%9f%ba%e7%a1%80)
    - [数据类型](#%e6%95%b0%e6%8d%ae%e7%b1%bb%e5%9e%8b)
    - [String 字符串函数操作](#string-%e5%ad%97%e7%ac%a6%e4%b8%b2%e5%87%bd%e6%95%b0%e6%93%8d%e4%bd%9c)
    - [常用 JNI 访问 Java 对象方法](#%e5%b8%b8%e7%94%a8-jni-%e8%ae%bf%e9%97%ae-java-%e5%af%b9%e8%b1%a1%e6%96%b9%e6%b3%95)
  - [NDK 开发](#ndk-%e5%bc%80%e5%8f%91-1)
    - [基础开发流程](#%e5%9f%ba%e7%a1%80%e5%bc%80%e5%8f%91%e6%b5%81%e7%a8%8b)
    - [System.loadLibrary()](#systemloadlibrary)
  - [CMake 构建 NDK 项目](#cmake-%e6%9e%84%e5%bb%ba-ndk-%e9%a1%b9%e7%9b%ae)
  - [常用的 Android NDK 原生 API](#%e5%b8%b8%e7%94%a8%e7%9a%84-android-ndk-%e5%8e%9f%e7%94%9f-api)
- [类加载器](#%e7%b1%bb%e5%8a%a0%e8%bd%bd%e5%99%a8)
  - [双亲委托模式](#%e5%8f%8c%e4%ba%b2%e5%a7%94%e6%89%98%e6%a8%a1%e5%bc%8f)
  - [DexPathList](#dexpathlist)
# ART
ART 代表 Android Runtime，其处理应用程序执行的方式完全不同于 Dalvik，Dalvik 是依靠一个 Just-In-Time (JIT) 编译器去解释字节码。开发者编译后的应用代码需要通过一个解释器在用户的设备上运行，这一机制并不高效，但让应用能更容易在不同硬件和架构上运 行。ART 则完全改变了这套做法，在应用安装时就预编译字节码到机器语言，这一机制叫 Ahead-Of-Time (AOT）编译。在移除解释代码这一过程后，应用程序执行将更有效率，启动更快。

## ART 功能
### 预先 (AOT) 编译
ART 引入了预先编译机制，可提高应用的性能。ART 还具有比 Dalvik 更严格的安装时验证。在安装时，ART 使用设备自带的 dex2oat 工具来编译应用。该实用工具接受 DEX 文件作为输入，并为目标设备生成经过编译的应用可执行文件。该工具应能够顺利编译所有有效的 DEX 文件。

### 垃圾回收优化
垃圾回收 (GC) 可能有损于应用性能，从而导致显示不稳定、界面响应速度缓慢以及其他问题。ART 通过以下几种方式对垃圾回收做了优化：
- 只有一次（而非两次）GC 暂停
- 在 GC 保持暂停状态期间并行处理
- 在清理最近分配的短时对象这种特殊情况中，回收器的总 GC 时间更短
- 优化了垃圾回收的工效，能够更加及时地进行并行垃圾回收，这使得 GC_FOR_ALLOC 事件在典型用例中极为罕见
- 压缩 GC 以减少后台内存使用和碎片

### 开发和调试方面的优化
- 支持采样分析器
  
一直以来，开发者都使用 Traceview 工具（用于跟踪应用执行情况）作为分析器。虽然 Traceview 可提供有用的信息，但每次方法调用产生的开销会导致 Dalvik 分析结果出现偏差，而且使用该工具明显会影响运行时性能

ART 添加了对没有这些限制的专用采样分析器的支持，因而可更准确地了解应用执行情况，而不会明显减慢速度。KitKat 版本为 Dalvik 的 Traceview 添加了采样支持。


- 支持更多调试功能

ART 支持许多新的调试选项，特别是与监控和垃圾回收相关的功能。例如，查看堆栈跟踪中保留了哪些锁，然后跳转到持有锁的线程；询问指定类的当前活动的实例数、请求查看实例，以及查看使对象保持有效状态的参考；过滤特定实例的事件（如断点）等。

- 优化了异常和崩溃报告中的诊断详细信息
  
当发生运行时异常时，ART 会为您提供尽可能多的上下文和详细信息。ART 会提供 ``java.lang.ClassCastException``、``java.lang.ClassNotFoundException`` 和 ``java.lang.NullPointerException`` 的更多异常详细信息（较高版本的 Dalvik 会提供 ``java.lang.ArrayIndexOutOfBoundsException`` 和 ``java.lang.ArrayStoreException`` 的更多异常详细信息，这些信息现在包括数组大小和越界偏移量；ART 也提供这类信息）。

## ART GC
ART 有多个不同的 GC 方案，这些方案包括运行不同垃圾回收器。默认方案是 CMS（并发标记清除）方案，主要使用粘性 CMS 和部分 CMS。粘性 CMS 是 ART 的不移动分代垃圾回收器。它仅扫描堆中自上次 GC 后修改的部分，并且只能回收自上次 GC 后分配的对象。除 CMS 方案外，当应用将进程状态更改为察觉不到卡顿的进程状态（例如，后台或缓存）时，ART 将执行堆压缩。

除了新的垃圾回收器之外，ART 还引入了一种基于位图的新内存分配程序，称为 RosAlloc（插槽运行分配器）。此新分配器具有分片锁，当分配规模较小时可添加线程的本地缓冲区，因而性能优于 DlMalloc。

与 Dalvik 相比，ART CMS 垃圾回收计划在很多方面都有一定的改善：

- 与 Dalvik 相比，暂停次数从 2 次减少到 1 次。Dalvik 的第一次暂停主要是为了进行根标记，即在 ART 中进行并发标记，让线程标记自己的根，然后马上恢复运行。

- 与 Dalvik 类似，ART GC 在清除过程开始之前也会暂停 1 次。两者在这方面的主要差异在于：在此暂停期间，某些 Dalvik 环节在 ART 中并发进行。这些环节包括 java.lang.ref.Reference 处理、系统弱清除（例如，jni 弱全局等）、重新标记非线程根和卡片预清理。在 ART 暂停期间仍进行的阶段包括扫描脏卡片以及重新标记线程根，这些操作有助于缩短暂停时间。

- 相对于 Dalvik，ART GC 改进的最后一个方面是粘性 CMS 回收器增加了 GC 吞吐量。不同于普通的分代 GC，粘性 CMS 不移动。系统会将年轻对象保存在一个分配堆栈（基本上是 java.lang.Object 数组）中，而非为其设置一个专属区域。这样可以避免移动所需的对象以维持低暂停次数，但缺点是容易在堆栈中加入大量复杂对象图像而使堆栈变长。

ART GC 与 Dalvik 的另一个主要区别在于 ART GC 引入了移动垃圾回收器。使用移动 GC 的目的在于通过堆压缩来减少后台应用使用的内存。目前，触发堆压缩的事件是 ActivityManager 进程状态的改变。当应用转到后台运行时，它会通知 ART 已进入不再“感知”卡顿的进程状态。此时 ART 会进行一些操作（例如，压缩和监视器压缩），从而导致应用线程长时间暂停。目前正在使用的两个移动 GC 是同构空间压缩和半空间压缩。

- 半空间压缩将对象在两个紧密排列的碰撞指针空间之间进行移动。这种移动 GC 适用于小内存设备，因为它可以比同构空间压缩稍微多节省一点内存。额外节省出的空间主要来自紧密排列的对象，这样可以避免 RosAlloc/DlMalloc 分配器占用开销。由于 CMS 仍在前台使用，且不能从碰撞指针空间中进行收集，因此当应用在前台使用时，半空间还要再进行一次转换。这种情况并不理想，因为它可能引起较长时间的暂停。

- 同构空间压缩通过将对象从一个 RosAlloc 空间复制到另一个 RosAlloc 空间来实现。这有助于通过减少堆碎片来减少内存使用量。这是目前非低内存设备的默认压缩模式。相比半空间压缩，同构空间压缩的主要优势在于应用从后台切换到前台时无需进行堆转换。

# Apk 包体优化
## Apk 组成结构
| 文件/文件夹 | 作用/功能
|--|--
| res | 包含所有没有被编译到 .arsc 里面的资源文件
| lib | 引用库的文件夹
| assets | assets文件夹相比于 res 文件夹，还有可能放字体文件、预置数据和web页面等,通过 AssetManager 访问
| META_INF | 存放的是签名信息，用来保证 apk 包的完整性和系统的安全。在生成一个APK的时候，会对所有的打包文件做一个校验计算，并把结果放在该目录下面
| classes.dex | 包含编译后的应用程序源码转化成的dex字节码。APK 里面，可能会存在多个 dex 文件
| resources.arsc | 一些资源和标识符被编译和写入这个文件
| Androidmanifest.xml | 编译时，应用程序的 AndroidManifest.xml 被转化成二进制格式

## 整体优化
- 分离应用的独立模块，以插件的形式加载
- 解压APK，重新用 7zip 进行压缩
- 用 apksigner 签名工具 替代 java 提供的 jarsigner 签名工具

## 资源优化 
- 可以只用一套资源图片，一般采用 xhdpi 下的资源图片
- 通过扫描文件的 MD5 值，找出名字不同，内容相同的图片并删除
- 通过 Lint 工具扫描工程资源，移除无用资源
- 通过 Gradle 参数配置 shrinkResources=true
- 对 png 图片压缩
- 图片资源考虑采用 WebP 格式
- 避免使用帧动画，可使用 Lottie 动画库
- 优先考虑能否用 shape 代码、.9 图、svg 矢量图、VectorDrawable 类来替换传统的图片

## 代码优化
- 启用混淆以移除无用代码
- 剔除 R 文件
- 用注解替代枚举

## .arsc文件优化 
- 移除未使用的备用资源来优化 .arsc 文件
```groovy
android {
    defaultConfig {
        ...
        resConfigs "zh", "zh_CN", "zh_HK", "en"
    }
}
```

## lib目录优化
- 只提供对主流架构的支持，比如 arm，对于 mips 和 x86 架构可以考虑不提供支持
```groovy
android {
    defaultConfig {
        ...
        ndk {
            abiFilters  "armeabi-v7a"
        }
    }
}
```

# Hook
## 基本流程
1、根据需求确定 要 hook 的对象  
2、寻找要hook的对象的持有者，拿到要 hook 的对象  
3、定义“要 hook 的对象”的代理类，并且创建该类的对象  
4、使用上一步创建出来的对象，替换掉要 hook 的对象

## 使用示例
```java
/**
* hook的核心代码
* 这个方法的唯一目的：用自己的点击事件，替换掉 View 原来的点击事件
*
* @param view hook的范围仅限于这个view
*/
@SuppressLint({"DiscouragedPrivateApi", "PrivateApi"})
public static void hook(Context context, final View view) {//
    try {
        // 反射执行View类的getListenerInfo()方法，拿到v的mListenerInfo对象，这个对象就是点击事件的持有者
        Method method = View.class.getDeclaredMethod("getListenerInfo");
        method.setAccessible(true);//由于getListenerInfo()方法并不是public的，所以要加这个代码来保证访问权限
        Object mListenerInfo = method.invoke(view);//这里拿到的就是mListenerInfo对象，也就是点击事件的持有者

        // 要从这里面拿到当前的点击事件对象
        Class<?> listenerInfoClz = Class.forName("android.view.View$ListenerInfo");// 这是内部类的表示方法
        Field field = listenerInfoClz.getDeclaredField("mOnClickListener");
        final View.OnClickListener onClickListenerInstance = (View.OnClickListener) field.get(mListenerInfo);//取得真实的mOnClickListener对象

        // 2. 创建我们自己的点击事件代理类
        //   方式1：自己创建代理类
        //   ProxyOnClickListener proxyOnClickListener = new ProxyOnClickListener(onClickListenerInstance);
        //   方式2：由于View.OnClickListener是一个接口，所以可以直接用动态代理模式
        // Proxy.newProxyInstance的3个参数依次分别是：
        // 本地的类加载器;
        // 代理类的对象所继承的接口（用Class数组表示，支持多个接口）
        // 代理类的实际逻辑，封装在new出来的InvocationHandler内
        Object proxyOnClickListener = Proxy.newProxyInstance(context.getClass().getClassLoader(), new Class[]{View.OnClickListener.class}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                Log.d("HookSetOnClickListener", "点击事件被hook到了");//加入自己的逻辑
                return method.invoke(onClickListenerInstance, args);//执行被代理的对象的逻辑
            }
        });
        // 3. 用我们自己的点击事件代理类，设置到"持有者"中
        field.set(mListenerInfo, proxyOnClickListener);
    } catch (Exception e) {
        e.printStackTrace();
    }
}

// 自定义代理类
static class ProxyOnClickListener implements View.OnClickListener {
    View.OnClickListener oriLis;

    public ProxyOnClickListener(View.OnClickListener oriLis) {
        this.oriLis = oriLis;
    }

    @Override
    public void onClick(View v) {
        Log.d("HookSetOnClickListener", "点击事件被hook到了");
        if (oriLis != null) {
            oriLis.onClick(v);
        }
    }
}
```

# Proguard
Proguard 具有以下三个功能：
- 压缩（Shrink）: 检测和删除没有使用的类，字段，方法和特性
- 优化（Optimize） : 分析和优化Java字节码
- 混淆（Obfuscate）: 使用简短的无意义的名称，对类，字段和方法进行重命名

## 公共模板
```
#############################################
#
# 对于一些基本指令的添加
#
#############################################
# 代码混淆压缩比，在 0~7 之间，默认为 5，一般不做修改
-optimizationpasses 5

# 混合时不使用大小写混合，混合后的类名为小写
-dontusemixedcaseclassnames

# 指定不去忽略非公共库的类
-dontskipnonpubliclibraryclasses

# 这句话能够使我们的项目混淆后产生映射文件
# 包含有类名->混淆后类名的映射关系
-verbose

# 指定不去忽略非公共库的类成员
-dontskipnonpubliclibraryclassmembers

# 不做预校验，preverify 是 proguard 的四个步骤之一，Android 不需要 preverify，去掉这一步能够加快混淆速度。
-dontpreverify

# 保留 Annotation 不混淆
-keepattributes *Annotation*,InnerClasses

# 避免混淆泛型
-keepattributes Signature

# 抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable

# 指定混淆是采用的算法，后面的参数是一个过滤器
# 这个过滤器是谷歌推荐的算法，一般不做更改
-optimizations !code/simplification/cast,!field/*,!class/merging/*


#############################################
#
# Android开发中一些需要保留的公共部分
#
#############################################

# 保留我们使用的四大组件，自定义的 Application 等等这些类不被混淆
# 因为这些子类都有可能被外部调用
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Appliction
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class * extends android.view.View
-keep public class com.android.vending.licensing.ILicensingService


# 保留 support 下的所有类及其内部类
-keep class android.support.** { *; }

# 保留继承的
-keep public class * extends android.support.v4.**
-keep public class * extends android.support.v7.**
-keep public class * extends android.support.annotation.**

# 保留 R 下面的资源
-keep class **.R$* { *; }

# 保留本地 native 方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保留在 Activity 中的方法参数是view的方法，
# 这样以来我们在 layout 中写的 onClick 就不会被影响
-keepclassmembers class * extends android.app.Activity {
    public void *(android.view.View);
}

# 保留枚举类不被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保留我们自定义控件（继承自 View）不被混淆
-keep public class * extends android.view.View {
    *** get*();
    void set*(***);
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
}

# 保留 Parcelable 序列化类不被混淆
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

# 保留 Serializable 序列化的类不被混淆
-keepnames class * implements java.io.Serializable
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    !static !transient <fields>;
    !private <fields>;
    !private <methods>;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}

# 对于带有回调函数的 onXXEvent、**On*Listener 的，不能被混淆
-keepclassmembers class * {
    void *(**On*Event);
    void *(**On*Listener);
}

# webView 处理，项目中没有使用到 webView 忽略即可
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
    public *;
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
    public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.webViewClient {
    public void *(android.webkit.webView, java.lang.String);
}

# js
-keepattributes JavascriptInterface
-keep class android.webkit.JavascriptInterface { *; }
-keepclassmembers class * {
    @android.webkit.JavascriptInterface <methods>;
}

# @Keep
-keep,allowobfuscation @interface android.support.annotation.Keep
-keep @android.support.annotation.Keep class *
-keepclassmembers class * {
    @android.support.annotation.Keep *;
}
```

##  常用的自定义混淆规则
```xml
# 通配符*，匹配任意长度字符，但不含包名分隔符(.)
# 通配符**，匹配任意长度字符，并且包含包名分隔符(.)

# 不混淆某个类
-keep public class com.jasonwu.demo.Test { *; }

# 不混淆某个包所有的类
-keep class com.jasonwu.demo.test.** { *; }

# 不混淆某个类的子类
-keep public class * com.jasonwu.demo.Test { *; }

# 不混淆所有类名中包含了 ``model`` 的类及其成员
-keep public class **.*model*.** {*;}

# 不混淆某个接口的实现
-keep class * implements com.jasonwu.demo.TestInterface { *; }

# 不混淆某个类的构造方法
-keepclassmembers class com.jasonwu.demo.Test { 
  public <init>(); 
}

# 不混淆某个类的特定的方法
-keepclassmembers class com.jasonwu.demo.Test { 
  public void test(java.lang.String); 
}
```


## aar中增加独立的混淆配置
``build.gralde``
```gradle
android {
    ···
    defaultConfig {
        ···
        consumerProguardFile 'proguard-rules.pro'
    }
    ···
}
```

## 检查混淆和追踪异常
开启 Proguard 功能，则每次构建时 ProGuard 都会输出下列文件：

- dump.txt  
说明 APK 中所有类文件的内部结构。

- mapping.txt  
提供原始与混淆过的类、方法和字段名称之间的转换。

- seeds.txt  
列出未进行混淆的类和成员。

- usage.txt  
列出从 APK 移除的代码。

这些文件保存在 /build/outputs/mapping/release/ 中。我们可以查看 seeds.txt 里面是否是我们需要保留的，以及 usage.txt 里查看是否有误删除的代码。 mapping.txt 文件很重要，由于我们的部分代码是经过重命名的，如果该部分出现 bug，对应的异常堆栈信息里的类或成员也是经过重命名的，难以定位问题。我们可以用 retrace 脚本（在 Windows 上为 retrace.bat；在 Mac/Linux 上为 retrace.sh）。它位于 /tools/proguard/ 目录中。该脚本利用 mapping.txt 文件和你的异常堆栈文件生成没有经过混淆的异常堆栈文件,这样就可以看清是哪里出问题了。使用 retrace 工具的语法如下：

```shell
retrace.bat|retrace.sh [-verbose] mapping.txt [<stacktrace_file>]
```

# 架构
## MVC
![](https://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Wy5xbLTp6JMMdouZiavFxyYCwLhGyLdicyLzgUDKFTZVt1OgU6iaSx2IUwnygzmQzW7Renaa8hmQ62cQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在 Android 中，三者的关系如下：

![](https://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Wy5xbLTp6JMMdouZiavFxyYCicNvEVMO9vDgukUR29Z1DCacZJwmmH1EEb7gUOZmDxolWexP01O8jfg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

由于在 Android 中 xml 布局的功能性太弱，所以 Activity 承担了绝大部分的工作，所以在 Android 中 mvc 更像：

![](https://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Wy5xbLTp6JMMdouZiavFxyYCOq89MLQX4UM3dgBTQfU72desHb1XbOWRQZINnXOCCdZCuicUiaTHhtEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

总结：
- 具有一定的分层，model 解耦，controller 和 view 并没有解耦
- controller 和 view 在 Android 中无法做到彻底分离，Controller 变得臃肿不堪
- 易于理解、开发速度快、可维护性高

## MVP
![](https://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Wy5xbLTp6JMMdouZiavFxyYCLVgibsuVQFguBI8FBdZibLNfpvbpd6njkdGWdyR2UL6TzMOhKHFqLC0Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

通过引入接口 BaseView，让相应的视图组件如 Activity，Fragment去实现 BaseView，把业务逻辑放在 presenter 层中，弱化 Model 只有跟 view 相关的操作都由 View 层去完成。

总结：
- 彻底解决了 MVC 中 View 和 Controller 傻傻分不清楚的问题
- 但是随着业务逻辑的增加，一个页面可能会非常复杂，UI 的改变是非常多，会有非常多的 case，这样就会造成 View 的接口会很庞大
- 更容易单元测试

## MVVM
![](https://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Wy5xbLTp6JMMdouZiavFxyYCMygIDD6xo5djkq6Y3jZo53sT2A4kKNaz8JEVRwmUnTmcAwJm0pZVWg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在 MVP 中 View 和 Presenter 要相互持有，方便调用对方，而在 MVP 中 View 和 ViewModel 通过 Binding 进行关联，他们之前的关联处理通过  DataBinding 完成。

总结：
- 很好的解决了 MVC 和 MVP 的问题
- 视图状态较多，ViewModel 的构建和维护的成本都会比较高
- 但是由于数据和视图的双向绑定，导致出现问题时不太好定位来源

# Jetpack
## 架构
![](https://developer.android.google.cn/topic/libraries/architecture/images/final-architecture.png)

## 使用示例
``build.gradle``
```groovy
android {
    ···
    dataBinding {
        enabled = true
    }
}
dependencies {
    ···
    implementation "androidx.fragment:fragment-ktx:$rootProject.fragmentVersion"
    implementation "androidx.lifecycle:lifecycle-extensions:$rootProject.lifecycleVersion"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:$rootProject.lifecycleVersion"
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$rootProject.lifecycleVersion"
}
```

``fragment_plant_detail.xml``
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="viewModel"
            type="com.google.samples.apps.sunflower.viewmodels.PlantDetailViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            ···
            android:text="@{viewModel.plant.name}"/>

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```


``PlantDetailFragment.kt``
```kotlin
class PlantDetailFragment : Fragment() {

    private val args: PlantDetailFragmentArgs by navArgs()
    private lateinit var shareText: String

    private val plantDetailViewModel: PlantDetailViewModel by viewModels {
        InjectorUtils.providePlantDetailViewModelFactory(requireActivity(), args.plantId)
    }

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        val binding = DataBindingUtil.inflate<FragmentPlantDetailBinding>(
                inflater, R.layout.fragment_plant_detail, container, false).apply {
            viewModel = plantDetailViewModel
            lifecycleOwner = this@PlantDetailFragment
        }

        plantDetailViewModel.plant.observe(this) { plant ->
            // 更新相关 UI
        }

        return binding.root
    }
}
```

``Plant.kt``
```kotlin
data class Plant (
    val name: String
)
```

``PlantDetailViewModel.kt``
```kotlin
class PlantDetailViewModel(
    plantRepository: PlantRepository,
    private val plantId: String
) : ViewModel() {

    val plant: LiveData<Plant>

    override fun onCleared() {
        super.onCleared()
        viewModelScope.cancel()
    }

    init {
        plant = plantRepository.getPlant(plantId)
    }
}
```

``PlantDetailViewModelFactory.kt``
```kotlin
class PlantDetailViewModelFactory(
    private val plantRepository: PlantRepository,
    private val plantId: String
) : ViewModelProvider.NewInstanceFactory() {

    @Suppress("UNCHECKED_CAST")
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return PlantDetailViewModel(plantRepository, plantId) as T
    }
}
```

``InjectorUtils.kt``
```kotlin
object InjectorUtils {
    private fun getPlantRepository(context: Context): PlantRepository {
        ···
    }

    fun providePlantDetailViewModelFactory(
        context: Context,
        plantId: String
    ): PlantDetailViewModelFactory {
        return PlantDetailViewModelFactory(getPlantRepository(context), plantId)
    }
}
```

# NDK 开发
> NDK 全称是 Native Development Kit，是一组可以让你在 Android 应用中编写实现 C/C++ 的工具，可以在项目用自己写源代码构建，也可以利用现有的预构建库。

使用 NDK 的使用目的有：
- 从设备获取更好的性能以用于计算密集型应用，例如游戏或物理模拟  
- 重复使用自己或其他开发者的 C/C++ 库，便利于跨平台。  
- NDK 集成了譬如 OpenSL、Vulkan 等 API 规范的特定实现，以实现在 java 层无法做到的功能如提升音频性能等  
- 增加反编译难度

## JNI 基础
### 数据类型
- 基本数据类型
  
| Java 类型 | Native 类型 | 符号属性 | 字长
|--|--|--|--
| boolean | jboolean | 无符号 | 8位
| byte | jbyte | 无符号 | 8位
| char | jchar | 无符号 | 16位
| short | jshort | 有符号 | 16位
| int | jnit | 有符号 | 32位
| long | jlong | 有符号 | 64位
| float | jfloat | 有符号 | 32位
| double | jdouble | 有符号 | 64位

- 引用数据类型

| Java 引用类型	| Native 类型 | Java 引用类型 | Native 类型
|--|--|--|--
| All objects | jobject | char[] | jcharArray
| java.lang.Class | jclass | short[] | jshortArray
| java.lang.String | jstring | int[] | jintArray
| Object[] | jobjectArray | long[] | jlongArray
| boolean[] | jbooleanArray | float[] | jfloatArray
| byte[] | jbyteArray | double[] | jdoubleArray
| java.lang.Throwable | jthrowable	

### String 字符串函数操作
| JNI 函数 | 描述
|--|--
| GetStringChars / ReleaseStringChars | 获得或释放一个指向 Unicode 编码的字符串的指针（指 C/C++ 字符串）
| GetStringUTFChars / ReleaseStringUTFChars | 获得或释放一个指向 UTF-8 编码的字符串的指针（指 C/C++ 字符串）
| GetStringLength | 返回 Unicode 编码的字符串的长度
| getStringUTFLength | 返回 UTF-8 编码的字符串的长度
| NewString | 将 Unicode 编码的 C/C++ 字符串转换为 Java 字符串
| NewStringUTF | 将 UTF-8 编码的 C/C++ 字符串转换为 Java 字符串
| GetStringCritical / ReleaseStringCritical | 获得或释放一个指向字符串内容的指针(指 Java 字符串)
| GetStringRegion | 获取或者设置 Unicode 编码的字符串的指定范围的内容
| GetStringUTFRegion | 获取或者设置 UTF-8 编码的字符串的指定范围的内容

### 常用 JNI 访问 Java 对象方法
``MyJob.java``
```java
package com.example.myjniproject;

public class MyJob {

    public static String JOB_STRING = "my_job";
    private int jobId;

    public MyJob(int jobId) {
        this.jobId = jobId;
    }

    public int getJobId() {
        return jobId;
    }
}
```
``native-lib.cpp``
```c++
#include <jni.h>

extern "C"
JNIEXPORT jint JNICALL
Java_com_example_myjniproject_MainActivity_getJobId(JNIEnv *env, jobject thiz, jobject job) {

    // 根据实力获取 class 对象
    jclass jobClz = env->GetObjectClass(job);
    // 根据类名获取 class 对象
    jclass jobClz = env->FindClass("com/example/myjniproject/MyJob");

    // 获取属性 id
    jfieldID fieldId = env->GetFieldID(jobClz, "jobId", "I");
    // 获取静态属性 id
    jfieldID sFieldId = env->GetStaticFieldID(jobClz, "JOB_STRING", "Ljava/lang/String;");

    // 获取方法 id
    jmethodID methodId = env->GetMethodID(jobClz, "getJobId", "()I");
    // 获取构造方法 id
    jmethodID  initMethodId = env->GetMethodID(jobClz, "<init>", "(I)V");

    // 根据对象属性 id 获取该属性值
    jint id = env->GetIntField(job, fieldId);
    // 根据对象方法 id 调用该方法
    jint id = env->CallIntMethod(job, methodId);

    // 创建新的对象
    jobject newJob = env->NewObject(jobClz, initMethodId, 10);

    return id;
}
```

## NDK 开发
### 基础开发流程
- 在 java 中声明 native 方法
```java
public class MainActivity extends AppCompatActivity {

    // Used to load the 'native-lib' library on application startup.
    static {
        System.loadLibrary("native-lib");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Log.d("MainActivity", stringFromJNI());
    }

    private native String stringFromJNI();
}
```

- 在 ``app/src/main`` 目录下新建 cpp 目录，新建相关 cpp 文件，实现相关方法（AS 可用快捷键快速生成）

``native-lib.cpp``
```
#include <jni.h>

extern "C" JNIEXPORT jstring JNICALL
Java_com_example_myjniproject_MainActivity_stringFromJNI(
        JNIEnv *env,
        jobject /* this */) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```

>- 函数名的格式遵循遵循如下规则：Java_包名_类名_方法名。
>- extern "C" 指定采用 C 语言的命名风格来编译，否则由于 C 与 C++ 风格不同，导致链接时无法找到具体的函数
>- JNIEnv*：表示一个指向 JNI 环境的指针，可以通过他来访问 JNI 提供的接口方法
>- jobject：表示 java 对象中的 this
>- JNIEXPORT 和 JNICALL：JNI 所定义的宏，可以在 jni.h 头文件中查找到

- 通过 CMake 或者 ndk-build 构建动态库

### System.loadLibrary()
``java/lang/System.java``:
```java
@CallerSensitive
public static void load(String filename) {
    Runtime.getRuntime().load0(Reflection.getCallerClass(), filename);
}
```

- 调用 ``Runtime`` 相关 native 方法

``java/lang/Runtime.java``:
```java
private static native String nativeLoad(String filename, ClassLoader loader, Class<?> caller);
```

- native 方法的实现如下：

``dalvik/vm/native/java_lang_Runtime.cpp``:
```cpp
static void Dalvik_java_lang_Runtime_nativeLoad(const u4* args,
    JValue* pResult)
{
    ···
    bool success;

    assert(fileNameObj != NULL);
    // 将 Java 的 library path String 转换到 native 的 String
    fileName = dvmCreateCstrFromString(fileNameObj);

    success = dvmLoadNativeCode(fileName, classLoader, &reason);
    if (!success) {
        const char* msg = (reason != NULL) ? reason : "unknown failure";
        result = dvmCreateStringFromCstr(msg);
        dvmReleaseTrackedAlloc((Object*) result, NULL);
    }
    ···
}
```

- ``dvmLoadNativeCode`` 函数实现如下：

``dalvik/vm/Native.cpp``
```cpp
bool dvmLoadNativeCode(const char* pathName, Object* classLoader,
        char** detail)
{
    SharedLib* pEntry;
    void* handle;
    ···
    *detail = NULL;

    // 如果已经加载过了，则直接返回 true
    pEntry = findSharedLibEntry(pathName);
    if (pEntry != NULL) {
        if (pEntry->classLoader != classLoader) {
            ···
            return false;
        }
        ···
        if (!checkOnLoadResult(pEntry))
            return false;
        return true;
    }

    Thread* self = dvmThreadSelf();
    ThreadStatus oldStatus = dvmChangeStatus(self, THREAD_VMWAIT);
    // 把.so mmap 到进程空间，并把 func 等相关信息填充到 soinfo 中
    handle = dlopen(pathName, RTLD_LAZY);
    dvmChangeStatus(self, oldStatus);
    ···
    // 创建一个新的 entry
    SharedLib* pNewEntry;
    pNewEntry = (SharedLib*) calloc(1, sizeof(SharedLib));
    pNewEntry->pathName = strdup(pathName);
    pNewEntry->handle = handle;
    pNewEntry->classLoader = classLoader;
    dvmInitMutex(&pNewEntry->onLoadLock);
    pthread_cond_init(&pNewEntry->onLoadCond, NULL);
    pNewEntry->onLoadThreadId = self->threadId;

    // 尝试添加到列表中
    SharedLib* pActualEntry = addSharedLibEntry(pNewEntry);

    if (pNewEntry != pActualEntry) {
        ···
        freeSharedLibEntry(pNewEntry);
        return checkOnLoadResult(pActualEntry);
    } else {
        ···
        bool result = true;
        void* vonLoad;
        int version;
        // 调用该 so 库的 JNI_OnLoad 方法
        vonLoad = dlsym(handle, "JNI_OnLoad");
        if (vonLoad == NULL) {
            ···
        } else {
            // 调用 JNI_Onload 方法，重写类加载器。
            OnLoadFunc func = (OnLoadFunc)vonLoad;
            Object* prevOverride = self->classLoaderOverride;

            self->classLoaderOverride = classLoader;
            oldStatus = dvmChangeStatus(self, THREAD_NATIVE);
            ···
            version = (*func)(gDvmJni.jniVm, NULL);
            dvmChangeStatus(self, oldStatus);
            self->classLoaderOverride = prevOverride;

            if (version != JNI_VERSION_1_2 && version != JNI_VERSION_1_4 &&
                version != JNI_VERSION_1_6)
            {
                ···
                result = false;
            } else {
                ···
            }
        }

        if (result)
            pNewEntry->onLoadResult = kOnLoadOkay;
        else
            pNewEntry->onLoadResult = kOnLoadFailed;

        pNewEntry->onLoadThreadId = 0;

        // 释放锁资源 
        dvmLockMutex(&pNewEntry->onLoadLock);
        pthread_cond_broadcast(&pNewEntry->onLoadCond);
        dvmUnlockMutex(&pNewEntry->onLoadLock);
        return result;
    }
}
```

<!-- ### native 方法调用原理
- 虚拟机调用一个方法时，发现如果这是一个 native 方法，则使用 Method 对象中的nativeFunc 函数指针对象调用。

``dalvik2/vm/interp/Stack.cpp``:
```cpp
Object* dvmInvokeMethod(Object* obj, const Method* method,
    ArrayObject* argList, ArrayObject* params, ClassObject* returnType,
    bool noAccessCheck)
{
    ···
    if (dvmIsNativeMethod(method)) {
        TRACE_METHOD_ENTER(self, method);
        (*method->nativeFunc)((u4*)self->interpSave.curFrame, &retval, method, self);
        TRACE_METHOD_EXIT(self, method);
    } else {
        dvmInterpret(self, method, &retval);
    }
    ···
}
``` -->

## CMake 构建 NDK 项目
> CMake 是一个开源的跨平台工具系列，旨在构建，测试和打包软件，从 Android Studio 2.2 开始，Android Sudio 默认地使用 CMake 与 Gradle 搭配使用来构建原生库。

启动方式只需要在 ``app/build.gradle`` 中添加相关：
```groovy
android {
    ···
    defaultConfig {
        ···
        externalNativeBuild {
            cmake {
                cppFlags ""
            }
        }

        ndk {
            abiFilters 'arm64-v8a', 'armeabi-v7a'
        }
    }
    ···
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
}
```

然后在对应目录新建一个 ``CMakeLists.txt`` 文件：
```txt
# 定义了所需 CMake 的最低版本
cmake_minimum_required(VERSION 3.4.1)

# add_library() 命令用来添加库
# native-lib 对应着生成的库的名字
# SHARED 代表为分享库
# src/main/cpp/native-lib.cpp 则是指明了源文件的路径。
add_library( # Sets the name of the library.
        native-lib

        # Sets the library as a shared library.
        SHARED

        # Provides a relative path to your source file(s).
        src/main/cpp/native-lib.cpp)

# find_library 命令添加到 CMake 构建脚本中以定位 NDK 库，并将其路径存储为一个变量。
# 可以使用此变量在构建脚本的其他部分引用 NDK 库
find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)

# 预构建的 NDK 库已经存在于 Android 平台上，因此，无需再构建或将其打包到 APK 中。
# 由于 NDK 库已经是 CMake 搜索路径的一部分，只需要向 CMake 提供希望使用的库的名称，并将其关联到自己的原生库中

# 要将预构建库关联到自己的原生库
target_link_libraries( # Specifies the target library.
        native-lib

        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})
···
```
- [CMake 命令详细信息文档](https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html)

## 常用的 Android NDK 原生 API
| 支持 NDK 的 API 级别 | 关键原生 API | 包括
|--|--|--
| 3 | Java 原生接口 | 	#include <jni.h>
| 3 | Android 日志记录 API	| #include <android/log.h>
| 5 | OpenGL ES 2.0 | #include <GLES2/gl2.h><br>#include <GLES2/gl2ext.h>
| 8 | Android 位图 API | #include <android/bitmap.h>
| 9 | OpenSL ES | #include <SLES/OpenSLES.h><br>#include <SLES/OpenSLES_Platform.h><br>#include <SLES/OpenSLES_Android.h><br>#include <SLES/OpenSLES_AndroidConfiguration.h>
| 9 | 原生应用 API | #include <android/rect.h><br>#include <android/window.h><br>#include<android/native_activity.h><br>···
| 18 | OpenGL ES 3.0 | #include <GLES3/gl3.h><br>#include <GLES3/gl3ext.h>
| 21 | 原生媒体 API | #include <media/NdkMediaCodec.h><br>#include <media/NdkMediaCrypto.h><br>···
| 24 | 原生相机 API | #include <camera/NdkCameraCaptureSession.h><br>#include <camera/NdkCameraDevice.h><br>···
| ···

# 类加载器
![](https://img-blog.csdn.net/20161021101447117?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## 双亲委托模式
某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载。

因为这样可以避免重复加载，当父亲已经加载了该类的时候，就没有必要子 ClassLoader 再加载一次。如果不使用这种委托模式，那我们就可以随时使用自定义的类来动态替代一些核心的类，存在非常大的安全隐患。

## DexPathList
DexClassLoader 重载了 ``findClass`` 方法，在加载类时会调用其内部的 DexPathList 去加载。DexPathList 是在构造 DexClassLoader 时生成的，其内部包含了 DexFile。

``DexPathList.java``
```java
···
public Class findClass(String name) {
    for (Element element : dexElements) {
        DexFile dex = element.dexFile;
        if (dex != null) {
            Class clazz = dex.loadClassBinaryName(name, definingContext);
            if (clazz != null) {
                return clazz;
            }
        }
    }
    return null;
}
···
```

