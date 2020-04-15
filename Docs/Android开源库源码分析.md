- [LeakCanary](#leakcanary)
  - [初始化注册](#%e5%88%9d%e5%a7%8b%e5%8c%96%e6%b3%a8%e5%86%8c)
  - [引用泄漏观察](#%e5%bc%95%e7%94%a8%e6%b3%84%e6%bc%8f%e8%a7%82%e5%af%9f)
  - [Dump Heap](#dump-heap)
- [EventBus](#eventbus)
  - [自定义注解](#%e8%87%aa%e5%ae%9a%e4%b9%89%e6%b3%a8%e8%a7%a3)
  - [注册订阅者](#%e6%b3%a8%e5%86%8c%e8%ae%a2%e9%98%85%e8%80%85)
  - [发送事件](#%e5%8f%91%e9%80%81%e4%ba%8b%e4%bb%b6)
# LeakCanary
![](http://ww1.sinaimg.cn/large/006dXScfly1fj22w7flt4j30z00mrtc0.jpg)

## 初始化注册
在清单文件中注册了一个 ContentProvider 用于在应用启动时初始化代码：  

``leakcanary-leaksentry/*/AndroidManifest.xml``
```xml
···
    <application>
        <provider
            android:name="leakcanary.internal.LeakSentryInstaller"
            android:authorities="${applicationId}.leak-sentry-installer"
            android:exported="false"/>
    </application>
···
```

在 LeakSentryInstaller 生命周期 ``onCreate()`` 方法中完成初始化步骤：

``LeakSentryInstaller.kt``
```kotlin
internal class LeakSentryInstaller : ContentProvider() {

    override fun onCreate(): Boolean {
        CanaryLog.logger = DefaultCanaryLog()
        val application = context!!.applicationContext as Application
        InternalLeakSentry.install(application)
        return true
    }
···
```

然后分别注册 Activity/Fragment 的监听：

``InternalLeakSentry.kt``
```kotlin
···
    fun install(application: Application) {
        CanaryLog.d("Installing LeakSentry")
        checkMainThread()
        if (this::application.isInitialized) {
        return
        }
        InternalLeakSentry.application = application

        val configProvider = { LeakSentry.config }
        ActivityDestroyWatcher.install(
            application, refWatcher, configProvider
        )
        FragmentDestroyWatcher.install(
            application, refWatcher, configProvider
        )
        listener.onLeakSentryInstalled(application)
    }
···
```
``ActivityDestroyWatcher.kt``
```kotlin
···
    fun install(application: Application,refWatcher: RefWatcher,configProvider: () -> Config
        ) {
            val activityDestroyWatcher = ActivityDestroyWatcher(refWatcher, configProvider)
            application.registerActivityLifecycleCallbacks(activityDestroyWatcher.lifecycleCallbacks)
        }
    }
···
```
``AndroidOFragmentDestroyWatcher.kt``
```kotlin
···
    override fun watchFragments(activity: Activity) {
        val fragmentManager = activity.fragmentManager
        fragmentManager.registerFragmentLifecycleCallbacks(fragmentLifecycleCallbacks, true)
    }
···
```
``AndroidXFragmentDestroyWatcher.kt``
```kotlin
···
    override fun watchFragments(activity: Activity) {
        if (activity is FragmentActivity) {
        val supportFragmentManager = activity.supportFragmentManager
        supportFragmentManager.registerFragmentLifecycleCallbacks(fragmentLifecycleCallbacks, true)
        }
    }
···
```

## 引用泄漏观察
``RefWatcher.kt``
```kotlin
···
    @Synchronized fun watch(watchedInstance: Any, name: String) {
        if (!isEnabled()) {
            return
        }
        removeWeaklyReachableInstances()
        val key = UUID.randomUUID().toString()
        val watchUptimeMillis = clock.uptimeMillis()
        val reference = KeyedWeakReference(watchedInstance, key, name, watchUptimeMillis, queue)
        CanaryLog.d(
            "Watching %s with key %s",
            ((if (watchedInstance is Class<*>) watchedInstance.toString() else "instance of ${watchedInstance.javaClass.name}") + if (name.isNotEmpty()) " named $name" else ""), key
        )

        watchedInstances[key] = reference
        checkRetainedExecutor.execute {
            moveToRetained(key)
        }
    }

    @Synchronized private fun moveToRetained(key: String) {
        removeWeaklyReachableInstances()
        val retainedRef = watchedInstances[key]
        if (retainedRef != null) {
            retainedRef.retainedUptimeMillis = clock.uptimeMillis()
            onInstanceRetained()
        }
    }
···
```

``InternalLeakCanary.kt``
```kotlin
···
    override fun onReferenceRetained() {
        if (this::heapDumpTrigger.isInitialized) {
            heapDumpTrigger.onReferenceRetained()
        }
    }
···
```

## Dump Heap
发现泄漏之后，获取 Heamp Dump 相关文件：

``AndroidHeapDumper.kt``
```kotlin
···
    override fun dumpHeap(): File? {
        val heapDumpFile = leakDirectoryProvider.newHeapDumpFile() ?: return null
        ···
        return try {
            Debug.dumpHprofData(heapDumpFile.absolutePath)
            if (heapDumpFile.length() == 0L) {
                CanaryLog.d("Dumped heap file is 0 byte length")
                null
            } else {
                heapDumpFile
            }
        } catch (e: Exception) {
            CanaryLog.d(e, "Could not dump heap")
            // Abort heap dump
            null
        } finally {
            cancelToast(toast)
            notificationManager.cancel(R.id.leak_canary_notification_dumping_heap)
        }
    }
···
```

``HeapDumpTrigger.kt``
```kotlin
···
    private fun checkRetainedInstances(reason: String) {
        ···
        val heapDumpFile = heapDumper.dumpHeap()
        ···
        lastDisplayedRetainedInstanceCount = 0
        refWatcher.removeInstancesWatchedBeforeHeapDump(heapDumpUptimeMillis)

        HeapAnalyzerService.runAnalysis(application, heapDumpFile)
    }
···
```

启动一个 HeapAnalyzerService 来分析 heapDumpFile：

``HeapAnalyzerService.kt``
```kotlin
···
    override fun onHandleIntentInForeground(intent: Intent?) {
        ···
        val heapAnalyzer = HeapAnalyzer(this)
        val config = LeakCanary.config

        val heapAnalysis =
        heapAnalyzer.checkForLeaks(
            heapDumpFile, config.referenceMatchers, config.computeRetainedHeapSize, config.objectInspectors,
            if (config.useExperimentalLeakFinders) config.objectInspectors else listOf(
                AndroidObjectInspectors.KEYED_WEAK_REFERENCE
            )
        )

        config.analysisResultListener(application, heapAnalysis)
    }
···
```

>Heap Dump 之后，可以查看以下内容：
>- 应用分配了哪些类型的对象，以及每种对象的数量。
>- 每个对象使用多少内存。
>- 代码中保存对每个对象的引用。
>- 分配对象的调用堆栈。（调用堆栈当前仅在使用Android 7.1及以下时有效。）

# EventBus
## 自定义注解
- 申明注解类
```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface Subscribe {
    // 线程模式
    ThreadMode threadMode() default ThreadMode.POSTING;

    // 是否为粘性事件
    boolean sticky() default false;

    // 事件的优先级
    int priority() default 0;
}
```

- 注册订阅事件
```java
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true)
public void onEventMainThreadP1(IntTestEvent event) {
    handleEvent(1, event);
}
```

## 注册订阅者
```java
EventBus.getDefault().register(object);
```

``Eventbus.java``
```java
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

- 通过反射查找订阅者类里的订阅事件，添加到 ``METHOD_CACHE`` 中
  
``SubscriberMethodFinder.java``
```java
private static final Map<Class<?>, List<SubscriberMethod>> METHOD_CACHE = new ConcurrentHashMap<>();
···

List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
    List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
    if (subscriberMethods != null) {
        return subscriberMethods;
    }

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
···

private void findUsingReflectionInSingleClass(FindState findState) {
    Method[] methods;
    try {
        // This is faster than getMethods, especially when subscribers are fat classes like Activities
        methods = findState.clazz.getDeclaredMethods();
    } catch (Throwable th) {
        // Workaround for java.lang.NoClassDefFoundError, see https://github.com/greenrobot/EventBus/issues/149
        methods = findState.clazz.getMethods();
        findState.skipSuperClasses = true;
    }
    for (Method method : methods) {
        int modifiers = method.getModifiers();
        if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
            Class<?>[] parameterTypes = method.getParameterTypes();
            if (parameterTypes.length == 1) {
                Subscribe subscribeAnnotation = method.getAnnotation(Subscribe.class);
                if (subscribeAnnotation != null) {
                    Class<?> eventType = parameterTypes[0];
                    if (findState.checkAdd(method, eventType)) {
                        ThreadMode threadMode = subscribeAnnotation.threadMode();
                        findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode,
                                subscribeAnnotation.priority(), subscribeAnnotation.sticky()));
                    }
                }
            } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
                String methodName = method.getDeclaringClass().getName() + "." + method.getName();
                throw new EventBusException("@Subscribe method " + methodName +
                        "must have exactly 1 parameter but has " + parameterTypes.length);
            }
        } else if (strictMethodVerification && method.isAnnotationPresent(Subscribe.class)) {
            String methodName = method.getDeclaringClass().getName() + "." + method.getName();
            throw new EventBusException(methodName +
                    " is a illegal @Subscribe method: must be public, non-static, and non-abstract");
        }
    }
}
```

## 发送事件
```java
EventBus.getDefault().post(object);
```

- 根据事件类型获取到对应的订阅者信息

``Subscription.java``
```java
final class Subscription {
    final Object subscriber;
    final SubscriberMethod subscriberMethod;
    ···
}
```

``EventBus.java``
```java
private final Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType;
···

private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
    CopyOnWriteArrayList<Subscription> subscriptions;
    synchronized (this) {
        subscriptions = subscriptionsByEventType.get(eventClass);
    }
    if (subscriptions != null && !subscriptions.isEmpty()) {
        for (Subscription subscription : subscriptions) {
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
···
```

- 根据注册已获得的 ``Method`` 对象调用相关注册方法

``EventBus.java``

```java
void invokeSubscriber(Subscription subscription, Object event) {
    try {
        subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
    } catch (InvocationTargetException e) {
        handleSubscriberException(subscription, event, e.getCause());
    } catch (IllegalAccessException e) {
        throw new IllegalStateException("Unexpected exception", e);
    }
}
```

<!-- # Glide
![](https://raw.githubusercontent.com/JsonChao/Awesome-Third-Library-Source-Analysis/master/ScreenShots/Glide%E6%A1%86%E6%9E%B6%E5%9B%BE.jpg) -->
