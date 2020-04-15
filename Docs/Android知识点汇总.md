- [Activity](#activity)
  - [生命周期](#%e7%94%9f%e5%91%bd%e5%91%a8%e6%9c%9f)
  - [启动模式](#%e5%90%af%e5%8a%a8%e6%a8%a1%e5%bc%8f)
  - [启动过程](#%e5%90%af%e5%8a%a8%e8%bf%87%e7%a8%8b)
- [Fragment](#fragment)
  - [特点](#%e7%89%b9%e7%82%b9)
  - [生命周期](#%e7%94%9f%e5%91%bd%e5%91%a8%e6%9c%9f-1)
  - [与Activity通信](#%e4%b8%8eactivity%e9%80%9a%e4%bf%a1)
- [Service](#service)
  - [启动过程](#%e5%90%af%e5%8a%a8%e8%bf%87%e7%a8%8b-1)
  - [绑定过程](#%e7%bb%91%e5%ae%9a%e8%bf%87%e7%a8%8b)
  - [生命周期](#%e7%94%9f%e5%91%bd%e5%91%a8%e6%9c%9f-2)
  - [启用前台服务](#%e5%90%af%e7%94%a8%e5%89%8d%e5%8f%b0%e6%9c%8d%e5%8a%a1)
- [BroadcastReceiver](#broadcastreceiver)
  - [注册过程](#%e6%b3%a8%e5%86%8c%e8%bf%87%e7%a8%8b)
- [ContentProvider](#contentprovider)
  - [基本使用](#%e5%9f%ba%e6%9c%ac%e4%bd%bf%e7%94%a8)
- [数据存储](#%e6%95%b0%e6%8d%ae%e5%ad%98%e5%82%a8)
- [View](#view)
  - [MeasureSpec](#measurespec)
  - [MotionEvent](#motionevent)
  - [VelocityTracker](#velocitytracker)
  - [GestureDetector](#gesturedetector)
  - [Scroller](#scroller)
  - [View 的滑动](#view-%e7%9a%84%e6%bb%91%e5%8a%a8)
  - [View 的事件分发](#view-%e7%9a%84%e4%ba%8b%e4%bb%b6%e5%88%86%e5%8f%91)
  - [在 Activity 中获取某个 View 的宽高](#%e5%9c%a8-activity-%e4%b8%ad%e8%8e%b7%e5%8f%96%e6%9f%90%e4%b8%aa-view-%e7%9a%84%e5%ae%bd%e9%ab%98)
  - [Draw 的基本流程](#draw-%e7%9a%84%e5%9f%ba%e6%9c%ac%e6%b5%81%e7%a8%8b)
  - [自定义 View](#%e8%87%aa%e5%ae%9a%e4%b9%89-view)
- [进程](#%e8%bf%9b%e7%a8%8b)
  - [进程生命周期](#%e8%bf%9b%e7%a8%8b%e7%94%9f%e5%91%bd%e5%91%a8%e6%9c%9f)
  - [多进程](#%e5%a4%9a%e8%bf%9b%e7%a8%8b)
  - [进程存活](#%e8%bf%9b%e7%a8%8b%e5%ad%98%e6%b4%bb)
    - [OOM_ADJ](#oomadj)
    - [进程被杀情况](#%e8%bf%9b%e7%a8%8b%e8%a2%ab%e6%9d%80%e6%83%85%e5%86%b5)
    - [进程保活方案](#%e8%bf%9b%e7%a8%8b%e4%bf%9d%e6%b4%bb%e6%96%b9%e6%a1%88)
- [Parcelable 接口](#parcelable-%e6%8e%a5%e5%8f%a3)
  - [使用示例](#%e4%bd%bf%e7%94%a8%e7%a4%ba%e4%be%8b)
  - [方法说明](#%e6%96%b9%e6%b3%95%e8%af%b4%e6%98%8e)
  - [Parcelable 与 Serializable 对比](#parcelable-%e4%b8%8e-serializable-%e5%af%b9%e6%af%94)
- [IPC](#ipc)
  - [IPC方式](#ipc%e6%96%b9%e5%bc%8f)
  - [Binder](#binder)
  - [AIDL 通信](#aidl-%e9%80%9a%e4%bf%a1)
  - [Messenger](#messenger)
- [Window / WindowManager](#window--windowmanager)
  - [Window 概念与分类](#window-%e6%a6%82%e5%bf%b5%e4%b8%8e%e5%88%86%e7%b1%bb)
  - [Window 的内部机制](#window-%e7%9a%84%e5%86%85%e9%83%a8%e6%9c%ba%e5%88%b6)
  - [Window 的创建过程](#window-%e7%9a%84%e5%88%9b%e5%bb%ba%e8%bf%87%e7%a8%8b)
    - [Activity 的 Window 创建过程](#activity-%e7%9a%84-window-%e5%88%9b%e5%bb%ba%e8%bf%87%e7%a8%8b)
    - [Dialog 的 Window 创建过程](#dialog-%e7%9a%84-window-%e5%88%9b%e5%bb%ba%e8%bf%87%e7%a8%8b)
    - [Toast 的 Window 创建过程](#toast-%e7%9a%84-window-%e5%88%9b%e5%bb%ba%e8%bf%87%e7%a8%8b)
- [Bitmap](#bitmap)
  - [配置信息与压缩方式](#%e9%85%8d%e7%bd%ae%e4%bf%a1%e6%81%af%e4%b8%8e%e5%8e%8b%e7%bc%a9%e6%96%b9%e5%bc%8f)
  - [常用操作](#%e5%b8%b8%e7%94%a8%e6%93%8d%e4%bd%9c)
    - [裁剪、缩放、旋转、移动](#%e8%a3%81%e5%89%aa%e7%bc%a9%e6%94%be%e6%97%8b%e8%bd%ac%e7%a7%bb%e5%8a%a8)
    - [Bitmap与Drawable转换](#bitmap%e4%b8%8edrawable%e8%bd%ac%e6%8d%a2)
    - [保存与释放](#%e4%bf%9d%e5%ad%98%e4%b8%8e%e9%87%8a%e6%94%be)
    - [图片压缩](#%e5%9b%be%e7%89%87%e5%8e%8b%e7%bc%a9)
  - [BitmapFactory](#bitmapfactory)
    - [Bitmap创建流程](#bitmap%e5%88%9b%e5%bb%ba%e6%b5%81%e7%a8%8b)
    - [Option类](#option%e7%b1%bb)
    - [基本使用](#%e5%9f%ba%e6%9c%ac%e4%bd%bf%e7%94%a8-1)
  - [内存回收](#%e5%86%85%e5%ad%98%e5%9b%9e%e6%94%b6)
- [屏幕适配](#%e5%b1%8f%e5%b9%95%e9%80%82%e9%85%8d)
  - [单位](#%e5%8d%95%e4%bd%8d)
  - [头条适配方案](#%e5%a4%b4%e6%9d%a1%e9%80%82%e9%85%8d%e6%96%b9%e6%a1%88)
  - [刘海屏适配](#%e5%88%98%e6%b5%b7%e5%b1%8f%e9%80%82%e9%85%8d)
- [Context](#context)
- [SharedPreferences](#sharedpreferences)
  - [获取方式](#%e8%8e%b7%e5%8f%96%e6%96%b9%e5%bc%8f)
    - [getPreferences](#getpreferences)
    - [getDefaultSharedPreferences](#getdefaultsharedpreferences)
    - [getSharedPreferences](#getsharedpreferences)
  - [架构](#%e6%9e%b6%e6%9e%84)
  - [apply / commit](#apply--commit)
  - [注意](#%e6%b3%a8%e6%84%8f)
- [消息机制](#%e6%b6%88%e6%81%af%e6%9c%ba%e5%88%b6)
  - [Handler 机制](#handler-%e6%9c%ba%e5%88%b6)
  - [工作原理](#%e5%b7%a5%e4%bd%9c%e5%8e%9f%e7%90%86)
    - [ThreadLocal](#threadlocal)
    - [MessageQueue](#messagequeue)
    - [Looper](#looper)
    - [Handler](#handler)
- [线程异步](#%e7%ba%bf%e7%a8%8b%e5%bc%82%e6%ad%a5)
  - [AsyncTask](#asynctask)
    - [基本使用](#%e5%9f%ba%e6%9c%ac%e4%bd%bf%e7%94%a8-2)
    - [工作原理](#%e5%b7%a5%e4%bd%9c%e5%8e%9f%e7%90%86-1)
  - [HandlerThread](#handlerthread)
  - [IntentService](#intentservice)
  - [线程池](#%e7%ba%bf%e7%a8%8b%e6%b1%a0)
- [RecyclerView 优化](#recyclerview-%e4%bc%98%e5%8c%96)
- [Webview](#webview)
  - [基本使用](#%e5%9f%ba%e6%9c%ac%e4%bd%bf%e7%94%a8-3)
    - [WebView](#webview-1)
    - [WebSettings](#websettings)
    - [WebViewClient](#webviewclient)
    - [WebChromeClient](#webchromeclient)
  - [Webview 加载优化](#webview-%e5%8a%a0%e8%bd%bd%e4%bc%98%e5%8c%96)
  - [内存泄漏](#%e5%86%85%e5%ad%98%e6%b3%84%e6%bc%8f)
# Activity
## 生命周期  
![](http://gityuan.com/images/lifecycle/activity.png)

- Activity A 启动另一个Activity B，回调如下:  
Activity A 的onPause() → Activity B的onCreate() → onStart() → onResume() → Activity A的onStop()；如果B是透明主题又或则是个DialogActivity，则不会回调A的onStop；

- 使用onSaveInstanceState（）保存简单，轻量级的UI状态
```java
lateinit var textView: TextView
var gameState: String? = null

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    gameState = savedInstanceState?.getString(GAME_STATE_KEY)
    setContentView(R.layout.activity_main)
    textView = findViewById(R.id.text_view)
}

override fun onRestoreInstanceState(savedInstanceState: Bundle?) {
    textView.text = savedInstanceState?.getString(TEXT_VIEW_KEY)
}

override fun onSaveInstanceState(outState: Bundle?) {
    outState?.run {
        putString(GAME_STATE_KEY, gameState)
        putString(TEXT_VIEW_KEY, textView.text.toString())
    }
    super.onSaveInstanceState(outState)
}
```
## 启动模式
| LaunchMode | 说明                      
|----------|-----|
| standard | 系统在启动它的任务中创建 activity 的新实例 |
| singleTop | 如果activity的实例已存在于当前任务的顶部，则系统通过调用其onNewIntent()，否则会创建新实例 |
| singleTask | 系统创建新 task 并在 task 的根目录下实例化 activity。但如果 activity 的实例已存在于单独的任务中，则调用其 onNewIntent() 方法，其上面的实例会被移除栈。一次只能存在一个 activity 实例 |
| singleInstance | 相同 singleTask，activity始终是其task的唯一成员; 任何由此开始的activity 都在一个单独的 task 中打开 |
&nbsp;
<!-- | 使用Intent标志 | 说明                      
|----------|-----|
| FLAG_ACTIVITY_NEW_TASK | 同 singleTask |
| FLAG_ACTIVITY_SINGLE_TOP | 同 singleTop |
| FLAG_ACTIVITY_CLEAR_TOP | 如果正在启动的 activity 已在当前 task中 运行，则不会启动该activity 的新实例，而是销毁其上的 activity，并调用其 onNewIntent() | -->

## 启动过程
![](https://img-blog.csdn.net/20180427173504903)

``ActivityThread.java``
```java
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    ...
    ActivityInfo aInfo = r.activityInfo;
    if (r.packageInfo == null) {
        //step 1: 创建LoadedApk对象
        r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,
                Context.CONTEXT_INCLUDE_CODE);
    }
    ... //component初始化过程

    java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
    //step 2: 创建Activity对象
    Activity activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
    ...

    //step 3: 创建Application对象
    Application app = r.packageInfo.makeApplication(false, mInstrumentation);

    if (activity != null) {
        //step 4: 创建ContextImpl对象
        Context appContext = createBaseContextForActivity(r, activity);
        CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
        Configuration config = new Configuration(mCompatConfiguration);
        //step5: 将Application/ContextImpl都attach到Activity对象
        activity.attach(appContext, this, getInstrumentation(), r.token,
                r.ident, app, r.intent, r.activityInfo, title, r.parent,
                r.embeddedID, r.lastNonConfigurationInstances, config,
                r.referrer, r.voiceInteractor);

        ...
        int theme = r.activityInfo.getThemeResource();
        if (theme != 0) {
            activity.setTheme(theme);
        }

        activity.mCalled = false;
        if (r.isPersistable()) {
            //step 6: 执行回调onCreate
            mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
        } else {
            mInstrumentation.callActivityOnCreate(activity, r.state);
        }

        r.activity = activity;
        r.stopped = true;
        if (!r.activity.mFinished) {
            activity.performStart(); //执行回调onStart
            r.stopped = false;
        }
        if (!r.activity.mFinished) {
            //执行回调onRestoreInstanceState
            if (r.isPersistable()) {
                if (r.state != null || r.persistentState != null) {
                    mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state,
                            r.persistentState);
                }
            } else if (r.state != null) {
                mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state);
            }
        }
        ...
        r.paused = true;
        mActivities.put(r.token, r);
    }

    return activity;
}

```

# Fragment
## 特点
- Fragment 解决 Activity 间的切换不流畅，轻量切换
- 可以从 startActivityForResult 中接收到返回结果，但是View不能
- 只能在 Activity 保存其状态（用户离开 Activity）之前使用 commit() 提交事务。如果您试图在该时间点后提交，则会引发异常。 这是因为如需恢复 Activity，则提交后的状态可能会丢失。 对于丢失提交无关紧要的情况，请使用 commitAllowingStateLoss()。

## 生命周期  
![](https://developer.android.google.cn/images/fragment_lifecycle.png)![](https://developer.android.google.cn/images/activity_fragment_lifecycle.png)  

## 与Activity通信
执行此操作的一个好方法是，在片段内定义一个回调接口，并要求宿主 Activity 实现它。
```java
public static class FragmentA extends ListFragment {
    ...
    // Container Activity must implement this interface
    public interface OnArticleSelectedListener {
        public void onArticleSelected(Uri articleUri);
    }
    ...
}

public static class FragmentA extends ListFragment {
    OnArticleSelectedListener mListener;
    ...
    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        try {
            mListener = (OnArticleSelectedListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString());
        }
    }
    ...
}
```

# Service
Service 分为两种工作状态，一种是启动状态，主要用于执行后台计算；另一种是绑定状态，主要用于其他组件和 Service 的交互。

## 启动过程
![](http://gityuan.com/images/android-service/am/Seq_start_service.png)

``ActivityThread.java``
```java
@UnsupportedAppUsage
private void handleCreateService(CreateServiceData data) {
    ···
    LoadedApk packageInfo = getPackageInfoNoCheck(
            data.info.applicationInfo, data.compatInfo);
    Service service = null;
    try {
        java.lang.ClassLoader cl = packageInfo.getClassLoader();
        service = packageInfo.getAppFactory()
                .instantiateService(cl, data.info.name, data.intent);
    } 
    ···

    try {
        if (localLOGV) Slog.v(TAG, "Creating service " + data.info.name);

        ContextImpl context = ContextImpl.createAppContext(this, packageInfo);
        context.setOuterContext(service);

        Application app = packageInfo.makeApplication(false, mInstrumentation);
        service.attach(context, this, data.info.name, data.token, app,
                ActivityManager.getService());
        service.onCreate();
        mServices.put(data.token, service);
        try {
            ActivityManager.getService().serviceDoneExecuting(
                    data.token, SERVICE_DONE_EXECUTING_ANON, 0, 0);
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    } 
    ··· 
}
```

## 绑定过程
![](http://gityuan.com/images/ams/bind_service.jpg)

``ActivityThread.java``
```java
private void handleBindService(BindServiceData data) {
    Service s = mServices.get(data.token);
    ···
    if (s != null) {
        try {
            data.intent.setExtrasClassLoader(s.getClassLoader());
            data.intent.prepareToEnterProcess();
            try {
                if (!data.rebind) {
                    IBinder binder = s.onBind(data.intent);
                    ActivityManager.getService().publishService(
                            data.token, data.intent, binder);
                } else {
                    s.onRebind(data.intent);
                    ActivityManager.getService().serviceDoneExecuting(
                            data.token, SERVICE_DONE_EXECUTING_ANON, 0, 0);
                }
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
        } 
        ···
    }
}
```

## 生命周期
![](https://upload-images.jianshu.io/upload_images/944365-cf5c1a9d2dddaaca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/456/format/webp)
| 值 | 说明 |
|-----|-----|
| START_NOT_STICKY | 如果系统在 onStartCommand() 返回后终止服务，则除非有挂起 Intent 要传递，否则系统不会重建服务。这是最安全的选项，可以避免在不必要时以及应用能够轻松重启所有未完成的作业时运行服务 |
| START_STICKY | 如果系统在 onStartCommand() 返回后终止服务，则会重建服务并调用 onStartCommand()，但不会重新传递最后一个 Intent。相反，除非有挂起 Intent 要启动服务（在这种情况下，将传递这些 Intent ），否则系统会通过空 Intent 调用 onStartCommand()。这适用于不执行命令、但无限期运行并等待作业的媒体播放器（或类似服务 |
| START_REDELIVER_INTENT | 如果系统在 onStartCommand() 返回后终止服务，则会重建服务，并通过传递给服务的最后一个 Intent 调用 onStartCommand()。任何挂起 Intent 均依次传递。这适用于主动执行应该立即恢复的作业（例如下载文件）的服务 |

## 启用前台服务
```java
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
```
```java
Notification notification = new Notification(icon, text, System.currentTimeMillis());
Intent notificationIntent = new Intent(this, ExampleActivity.class);
PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, notificationIntent, 0);
notification.setLatestEventInfo(this, title, mmessage, pendingIntent);
startForeground(ONGOING_NOTIFICATION_ID, notification);
```

# BroadcastReceiver
target 26 之后，无法在 AndroidManifest 显示声明大部分广播，除了一部分必要的广播，如：
- ACTION_BOOT_COMPLETED
- ACTION_TIME_SET
- ACTION_LOCALE_CHANGED
```java
LocalBroadcastManager.getInstance(MainActivity.this).registerReceiver(receiver, filter);
```

## 注册过程
![](http://gityuan.com/images/ams/send_broadcast.jpg)


# ContentProvider
ContentProvider 管理对结构化数据集的访问。它们封装数据，并提供用于定义数据安全性的机制。 内容提供程序是连接一个进程中的数据与另一个进程中运行的代码的标准界面。

ContentProvider 无法被用户感知，对于一个 ContentProvider 组件来说，它的内部需要实现增删该查这四种操作，它的内部维持着一份数据集合，这个数据集合既可以是数据库实现，也可以是其他任何类型，如 List 和 Map，内部的 insert、delete、update、query 方法需要处理好线程同步，因为这几个方法是在 Binder 线程池中被调用的。

ContentProvider 通过 Binder 向其他组件乃至其他应用提供数据。当 ContentProvider 所在的进程启动时，ContentProvider 会同时启动并发布到 AMS 中，需要注意的是，这个时候 ContentProvider 的 onCreate 要先于 Application 的 onCreate 而执行。

## 基本使用
```java
// Queries the user dictionary and returns results
mCursor = getContentResolver().query(
    UserDictionary.Words.CONTENT_URI,   // The content URI of the words table
    mProjection,                        // The columns to return for each row
    mSelectionClause                    // Selection criteria
    mSelectionArgs,                     // Selection criteria
    mSortOrder);                        // The sort order for the returned rows
```

```java
public class Installer extends ContentProvider {

    @Override
    public boolean onCreate() {
        return true;
    }

    @Nullable
    @Override
    public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection, @Nullable String[] selectionArgs, @Nullable String sortOrder) {
        return null;
    }

    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        return null;
    }

    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        return null;
    }

    @Override
    public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }

    @Override
    public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }
}
```

> ContentProvider 和 sql 在实现上有什么区别?
>- ContentProvider 屏蔽了数据存储的细节，内部实现透明化，用户只需关心 uri 即可(是否匹配)
>- ContentProvider 能实现不同 app 的数据共享，sql 只能是自己程序才能访问
>- Contentprovider 还能增删本地的文件,xml等信息
 
# 数据存储
| 存储方式 | 说明 |
|-----|-----|
| SharedPreferences | 在键值对中存储私有原始数据 |
| 内部存储 | 在设备内存中存储私有数据 |
| 外部存储 | 在共享的外部存储中存储公共数据 |
| SQLite 数据库 | 在私有数据库中存储结构化数据 |

# View
![](https://user-gold-cdn.xitu.io/2019/6/12/16b4a8a388f3a91a?imageslim)
ViewRoot 对应于 ViewRootImpl 类，它是连接 WindowManager 和 DecorView 的纽带，View 的三大流程均是通过 ViewRoot 来完成的。在 ActivityThread 中，当 Activity 对象被创建完毕后，会将 DecorView 添加到 Window 中，同时会创建 ViewRootImpl 对象，并将 ViewRootImpl 对象和 DecorView 建立关联

View 的整个绘制流程可以分为以下三个阶段：
- measure: 判断是否需要重新计算 View 的大小，需要的话则计算
- layout: 判断是否需要重新计算 View 的位置，需要的话则计算
- draw: 判断是否需要重新绘制 View，需要的话则重绘制

![](https://img-blog.csdn.net/20180510164327114)

## MeasureSpec
MeasureSpec表示的是一个32位的整形值，它的高2位表示测量模式SpecMode，低30位表示某种测量模式下的规格大小SpecSize。MeasureSpec 是 View 类的一个静态内部类，用来说明应该如何测量这个 View

| Mode | 说明 |
|-----|-----|
| UNSPECIFIED | 不指定测量模式, 父视图没有限制子视图的大小，子视图可以是想要的任何尺寸，通常用于系统内部，应用开发中很少用到。 |
| EXACTLY | 精确测量模式，视图宽高指定为 match_parent 或具体数值时生效，表示父视图已经决定了子视图的精确大小，这种模式下 View 的测量值就是 SpecSize 的值|
| AT_MOST | 最大值测量模式，当视图的宽高指定为 wrap_content 时生效，此时子视图的尺寸可以是不超过父视图允许的最大尺寸的任何尺寸 |

对于 DecorView 而言，它的MeasureSpec 由窗口尺寸和其自身的 LayoutParams 共同决定；对于普通的 View，它的 MeasureSpec 由父视图的 MeasureSpec 和其自身的 LayoutParams 共同决定

| childLayoutParams/parentSpecMode | EXACTLY | AT_MOST 
|--|--|--
| dp/px | EXACTLY(childSize) | EXACTLY(childSize)
| match_parent | EXACTLY(childSize) | AT_MOST(parentSize)
| wrap_content | AT_MOST(parentSize) | AT_MOST(parentSize)


直接继承 View 的控件需要重写 onMeasure 方法并设置 wrap_content 时的自身大小，因为 View 在布局中使用 wrap_content，那么它的 specMode 是 AT_MOST 模式，在这种模式下，它的宽/高等于父容器当前剩余的空间大小，就相当于使用 match_parent。这解决方式如下：
```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
    int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
    int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);
    // 在 wrap_content 的情况下指定内部宽/高(mWidth 和 mHeight`)
    if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST) {
        setMeasuredDimension(mWidth, mHeight);
    } else if (widthSpecMode == MeasureSpec.AT_MOST) {
        setMeasureDimension(mWidth, heightSpecSize);
    } else if (heightSpecMode == MeasureSpec.AT_MOST) {
        setMeasureDimension(widthSpecSize, mHeight);
    }
}
```

## MotionEvent
| 事件 | 说明 
|-----|------
| ACTION_DOWN | 手指刚接触到屏幕 
| ACTION_MOVE | 手指在屏幕上移动
| ACTION_UP | 手机从屏幕上松开的一瞬间
| ACTION_CANCEL | 触摸事件取消

点击屏幕后松开，事件序列为 DOWN -> UP，点击屏幕滑动松开，事件序列为 DOWN -> MOVE -> ...> MOVE -> UP。

``getX/getY`` 返回相对于当前View左上角的坐标，`getRawX/getRawY` 返回相对于屏幕左上角的坐标

TouchSlop是系统所能识别出的被认为滑动的最小距离，不同设备值可能不相同，可通过 ``ViewConfiguration.get(getContext()).getScaledTouchSlop()`` 获取。

## VelocityTracker
**VelocityTracker** 可用于追踪手指在滑动中的速度：
```java
view.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        VelocityTracker velocityTracker = VelocityTracker.obtain();
        velocityTracker.addMovement(event);
        velocityTracker.computeCurrentVelocity(1000);
        int xVelocity = (int) velocityTracker.getXVelocity();
        int yVelocity = (int) velocityTracker.getYVelocity();
        velocityTracker.clear();
        velocityTracker.recycle();
        return false;
    }
});
```

## GestureDetector
**GestureDetector** 辅助检测用户的单击、滑动、长按、双击等行为：
```java
final GestureDetector mGestureDetector = new GestureDetector(this, new GestureDetector.OnGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) { return false; }

    @Override
    public void onShowPress(MotionEvent e) { }

    @Override
    public boolean onSingleTapUp(MotionEvent e) { return false; }

    @Override
    public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) { return false; }

    @Override
    public void onLongPress(MotionEvent e) { }

    @Override
    public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) { return false; }
});
mGestureDetector.setOnDoubleTapListener(new OnDoubleTapListener() {
    @Override
    public boolean onSingleTapConfirmed(MotionEvent e) { return false; }

    @Override
    public boolean onDoubleTap(MotionEvent e) { return false; }

    @Override
    public boolean onDoubleTapEvent(MotionEvent e) { return false; }
});
// 解决长按屏幕后无法拖动的问题
mGestureDetector.setIsLongpressEnabled(false);
imageView.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        return mGestureDetector.onTouchEvent(event);
    }
});
```
如果是监听滑动相关，建议在 ``onTouchEvent`` 中实现，如果要监听双击，那么就使用 ``GestureDectector``。

## Scroller
弹性滑动对象，用于实现 View 的弹性滑动，**Scroller** 本身无法让 View 弹性滑动，需要和 View 的 ``computeScroll`` 方法配合使用。``startScroll`` 方法是无法让 View 滑动的，``invalidate`` 会导致 View 重绘，重回后会在 ``draw`` 方法中又会去调用 ``computeScroll`` 方法，``computeScroll`` 方法又会去向 Scroller 获取当前的 scrollX 和 scrollY，然后通过 ``scrollTo`` 方法实现滑动，接着又调用 ``postInvalidate`` 方法如此反复。
```java
Scroller mScroller = new Scroller(mContext);

private void smoothScrollTo(int destX) {
    int scrollX = getScrollX();
    int delta = destX - scrollX;
    // 1000ms 内滑向 destX，效果就是慢慢滑动
    mScroller.startScroll(scrollX, 0 , delta, 0, 1000);
    invalidate();
}

@Override
public void computeScroll() {
    if (mScroller.computeScrollOffset()) {
        scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
        postInvalidate();
    }
}
```

## View 的滑动
- ``scrollTo/scrollBy``  
适合对 View 内容的滑动。``scrollBy`` 实际上也是调用了 ``scrollTo`` 方法：
```java
public void scrollTo(int x, int y) {
    if (mScrollX != x || mScrollY != y) {
        int oldX = mScrollX;
        int oldY = mScrollY;
        mScrollX = x;
        mScrollY = y;
        invalidateParentCaches();
        onScrollChanged(mScrollX, mScrollY, oldX, oldY);
        if (!awakenScrollBars()) {
            postInvalidateOnAnimation();
        }
    }
}

public void scrollBy(int x, int y) {
    scrollTo(mScrollX + x, mScrollY + y);
}
```
mScrollX的值等于 View 的左边缘和 View 内容左边缘在水平方向的距离，mScrollY的值等于 View 上边缘和 View 内容上边缘在竖直方向的距离。``scrollTo`` 和 ``scrollBy`` 只能改变 View 内容的位置而不能改变 View 在布局中的位置。

- 使用动画  
操作简单，主要适用于没有交互的 View 和实现复杂的动画效果。
- 改变布局参数
操作稍微复杂，适用于有交互的 View.
```java
ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams) view.getLayoutParams();
params.width += 100;
params.leftMargin += 100;
view.requestLayout();
//或者 view.setLayoutParams(params);
```

## View 的事件分发
点击事件达到顶级 View(一般是一个 ViewGroup)，会调用 ViewGroup 的 dispatchTouchEvent 方法，如果顶级 ViewGroup 拦截事件即 onInterceptTouchEvent 返回 true，则事件由 ViewGroup 处理，这时如果 ViewGroup 的 mOnTouchListener 被设置，则 onTouch 会被调用，否则 onTouchEvent 会被调用。也就是说如果都提供的话，onTouch 会屏蔽掉 onTouchEvent。在 onTouchEvent 中，如果设置了 mOnClickListenser，则 onClick 会被调用。如果顶级 ViewGroup 不拦截事件，则事件会传递给它所在的点击事件链上的子 View，这时子 View 的 dispatchTouchEvent 会被调用。如此循环。

![](https://user-gold-cdn.xitu.io/2019/7/19/16c08654e36be140?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![](https://user-gold-cdn.xitu.io/2019/7/19/16c086493dc70018?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- ViewGroup 默认不拦截任何事件。ViewGroup 的 onInterceptTouchEvent 方法默认返回 false。

- View 没有 onInterceptTouchEvent 方法，一旦有点击事件传递给它，onTouchEvent 方法就会被调用。

- View 在可点击状态下，onTouchEvent 默认会消耗事件。

- ACTION_DOWN 被拦截了，onInterceptTouchEvent 方法执行一次后，就会留下记号（mFirstTouchTarget == null）那么往后的 ACTION_MOVE 和 ACTION_UP 都会拦截。`


## 在 Activity 中获取某个 View 的宽高
- Activity/View#onWindowFocusChanged
```
// 此时View已经初始化完毕
// 当Activity的窗口得到焦点和失去焦点时均会被调用一次
// 如果频繁地进行onResume和onPause，那么onWindowFocusChanged也会被频繁地调用
public void onWindowFocusChanged(boolean hasFocus) {
    super.onWindowFocusChanged(hasFocus);
    if (hasFocus) {
        int width = view.getMeasureWidth();
        int height = view.getMeasuredHeight();
    }
}
```
- view.post(runnable)
```
// 通过post可以将一个runnable投递到消息队列的尾部，// 然后等待Looper调用次runnable的时候，View也已经初
// 始化好了
protected void onStart() {
    super.onStart();
    view.post(new Runnable() {

        @Override
        public void run() {
            int width = view.getMeasuredWidth();
            int height = view.getMeasuredHeight();
        }
    });
}
```
- ViewTreeObserver
```java
// 当View树的状态发生改变或者View树内部的View的可见// 性发生改变时，onGlobalLayout方法将被回调
protected void onStart() {
    super.onStart();

    ViewTreeObserver observer = view.getViewTreeObserver();
    observer.addOnGlobalLayoutListener(new OnGlobalLayoutListener() {

        @SuppressWarnings("deprecation")
        @Override
        public void onGlobalLayout() {
            view.getViewTreeObserver().removeGlobalOnLayoutListener(this);
            int width = view.getMeasuredWidth();
            int height = view.getMeasuredHeight();
        }
    });
}
```

## Draw 的基本流程
```java
// 绘制基本上可以分为六个步骤
public void draw(Canvas canvas) {
    ...
    // 步骤一：绘制View的背景
    drawBackground(canvas);
    ...
    // 步骤二：如果需要的话，保持canvas的图层，为fading做准备
    saveCount = canvas.getSaveCount();
    ...
    canvas.saveLayer(left, top, right, top + length, null, flags);
    ...
    // 步骤三：绘制View的内容
    onDraw(canvas);
    ...
    // 步骤四：绘制View的子View
    dispatchDraw(canvas);
    ...
    // 步骤五：如果需要的话，绘制View的fading边缘并恢复图层
    canvas.drawRect(left, top, right, top + length, p);
    ...
    canvas.restoreToCount(saveCount);
    ...
    // 步骤六：绘制View的装饰(例如滚动条等等)
    onDrawForeground(canvas)
}
```

## 自定义 View
- 继承 View 重写 ``onDraw`` 方法
  
主要用于实现一些不规则的效果，静态或者动态地显示一些不规则的图形，即重写 ``onDraw`` 方法。采用这种方式需要自己支持 wrap_content，并且 padding 也需要自己处理。

- 继承 ViewGroup 派生特殊的 Layout
  
主要用于实现自定义布局，采用这种方式需要合适地处理 ViewGroup 的测量、布局两个过程，并同时处理子元素的测量和布局过程。

- 继承特定的 View
  
用于扩张某种已有的View的功能

- 继承特定的 ViewGroup
  
用于扩张某种已有的ViewGroup的功能

# 进程
当某个应用组件启动且该应用没有运行其他任何组件时，Android 系统会使用单个执行线程为应用启动新的 Linux 进程。默认情况下，同一应用的所有组件在相同的进程和线程（称为“主”线程）中运行。

各类组件元素的清单文件条目``<activity>``、``<service>``、``<receiver>`` 和 ``<provider>``—均支持 android:process 属性，此属性可以指定该组件应在哪个进程运行。

## 进程生命周期
**1、前台进程**
- 托管用户正在交互的 Activity（已调用 Activity 的 ``onResume()`` 方法）
- 托管某个 Service，后者绑定到用户正在交互的 Activity
- 托管正在“前台”运行的 Service（服务已调用 ``startForeground()``）
- 托管正执行一个生命周期回调的 Service（``onCreate()``、``onStart()`` 或 ``onDestroy()``）
- 托管正执行其 ``onReceive()`` 方法的 BroadcastReceiver

**2、可见进程**  
- 托管不在前台、但仍对用户可见的 Activity（已调用其 ``onPause()`` 方法）。例如，如果 re前台 Activity 启动了一个对话框，允许在其后显示上一 Activity，则有可能会发生这种情况。
- 托管绑定到可见（或前台）Activity 的 Service

**3、服务进程**  
- 正在运行已使用 startService() 方法启动的服务且不属于上述两个更高类别进程的进程。

**4、后台进程**
- 包含目前对用户不可见的 Activity 的进程（已调用 Activity 的 ``onStop()`` 方法）。通常会有很多后台进程在运行，因此它们会保存在 LRU （最近最少使用）列表中，以确保包含用户最近查看的 Activity 的进程最后一个被终止。

**5、空进程**
- 不含任何活动应用组件的进程。保留这种进程的的唯一目的是用作缓存，以缩短下次在其中运行组件所需的启动时间。 为使总体系统资源在进程缓存和底层内核缓存之间保持平衡，系统往往会终止这些进程。\

## 多进程
如果注册的四大组件中的任意一个组件时用到了多进程，运行该组件时，都会创建一个新的 Application 对象。对于多进程重复创建 Application 这种情况，只需要在该类中对当前进程加以判断即可。
```java
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        Log.d("MyApplication", getProcessName(android.os.Process.myPid()));
        super.onCreate();
    }

    /**
     * 根据进程 ID 获取进程名
     * @param pid 进程id
     * @return 进程名
     */
    public  String getProcessName(int pid){
        ActivityManager am = (ActivityManager)getSystemService(Context.ACTIVITY_SERVICE);
        List<ActivityManager.RunningAppProcessInfo> processInfoList = am.getRunningAppProcesses();
        if (processInfoList == null) {
            return null;
        }
        for (ActivityManager.RunningAppProcessInfo processInfo : processInfoList) {
            if (processInfo.pid == pid) {
                return processInfo.processName;
            }
        }
        return null;
    }
}
```

>一般来说，使用多进程会造成以下几个方面的问题：
>- 静态成员和单例模式完全失效
>- 线程同步机制完全失效
>- SharedPreferences 的可靠性下降
>- Application 会多次创建

## 进程存活
### OOM_ADJ
| ADJ级别 | 取值 | 解释
|-----|-----|------
| UNKNOWN_ADJ | 16 | 一般指将要会缓存进程，无法获取确定值
| CACHED_APP_MAX_ADJ | 15 | 不可见进程的adj最大值
| CACHED_APP_MIN_ADJ | 9 | 不可见进程的adj最小值
| SERVICE_B_AD | 8 | B List 中的 Service（较老的、使用可能性更小）
| PREVIOUS_APP_ADJ | 7 | 上一个App的进程(往往通过按返回键)
| HOME_APP_ADJ | 6 | Home进程
| SERVICE_ADJ | 5 | 服务进程(Service process)
| HEAVY_WEIGHT_APP_ADJ | 4 | 后台的重量级进程，system/rootdir/init.rc 文件中设置
| BACKUP_APP_ADJ | 3 | 备份进程
| PERCEPTIBLE_APP_ADJ | 2 | 可感知进程，比如后台音乐播放
| VISIBLE_APP_ADJ | 1 | 可见进程(Visible process)
| FOREGROUND_APP_ADJ | 0 | 前台进程（Foreground process)
| PERSISTENT_SERVICE_ADJ | -11 | 关联着系统或persistent进程
| PERSISTENT_PROC_ADJ | -12 | 系统 persistent 进程，比如telephony
| SYSTEM_ADJ | -16 | 系统进程
| NATIVE_ADJ | -17 | native进程（不被系统管理）

### 进程被杀情况
![](https://pic3.zhimg.com/80/18b6bfb1bf54433619a7122c3a8e606e_hd.png)

### 进程保活方案
- 开启一个像素的 Activity
- 使用前台服务
- 多进程相互唤醒
- JobSheduler 唤醒
- 粘性服务 & 与系统服务捆绑

# Parcelable 接口
只要实现了 Parcelable 接口，一个类的对象就可以实现序列化并可以通过 Intent 和 Binder 传递。

## 使用示例
```java
import android.os.Parcel;
import android.os.Parcelable;

public class User implements Parcelable {
    
    private int userId;

    protected User(Parcel in) {
        userId = in.readInt();
    }

    public static final Creator<User> CREATOR = new Creator<User>() {
        @Override
        public User createFromParcel(Parcel in) {
            return new User(in);
        }

        @Override
        public User[] newArray(int size) {
            return new User[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(userId);
    }

    public int getUserId() {
        return userId;
    }
}
```

## 方法说明
Parcel 内部包装了可序列化的数据，可以在 Binder 中自由传输。序列化功能由 ``writeToParcel`` 方法完成，最终是通过 Parcel 中的一系列 write 方法完成。反序列化功能由 CREATOR 来完成，通过 Parcel 的一系列 read 方法来完成反序列化过程。

| 方法 | 功能
|--|--
| createFromParcel(Parcel in) | 从序列化后的对象中创建原始对象
| newArray(int size) | 创建指定长度的原始对象数组
| User(Parcel in) | 从序列化后的对象中创建原始对象
| writeToParcel(Parcel dest, int flags) | 将当前对象写入序列化结构中，其中 flags 标识有两种值：0 或者 1。为 1 时标识当前对象需要作为返回值返回，不能立即释放资源，几乎所有情况都为 0
| describeContents | 返回当前对象的内容描述。如果含有文件描述符，返回 1，否则返回 0，几乎所有情况都返回 0 |

## Parcelable 与 Serializable 对比
- Serializable 使用 I/O 读写存储在硬盘上，而 Parcelable 是直接在内存中读写
- Serializable 会使用反射，序列化和反序列化过程需要大量 I/O 操作， Parcelable 自已实现封送和解封（marshalled &unmarshalled）操作不需要用反射，数据也存放在 Native 内存中，效率要快很多

# IPC
IPC 即 Inter-Process Communication (进程间通信)。Android 基于 Linux，而 Linux 出于安全考虑，不同进程间不能之间操作对方的数据，这叫做“进程隔离”。
> 在 Linux 系统中，虚拟内存机制为每个进程分配了线性连续的内存空间，操作系统将这种虚拟内存空间映射到物理内存空间，每个进程有自己的虚拟内存空间，进而不能操作其他进程的内存空间，只有操作系统才有权限操作物理内存空间。 进程隔离保证了每个进程的内存安全。

## IPC方式
| 名称 | 优点 | 缺点 | 适用场景
|----|-----|----|----
| Bundle | 简单易用 | 只能传输 Bundle 支持的数据类型 | 四大组件间的进程间通信
| 文件共享 | 简单易用 | 不适合高并发场景，并且无法做到进程间即时通信|无并发访问情形，交换简单的数据实时性不高的场景
| AIDL |功能强大，支持一对多并发通信，支持实时通信 | 使用稍复杂，需要处理好线程同步 | 一对多通信且有 RPC 需求
| Messenger | 功能一般，支持一对多串行通信，支持实时通信|不能很处理高并发清醒，不支持 RPC，数据通过 Message 进行传输，因此只能传输 Bundle 支持的数据类型|低并发的一对多即时通信，无RPC需求，或者无需返回结果的RPC需求
| ContentProvider | 在数据源访问方面功能强大，支持一对多并发数据共享，可通过 Call 方法扩展其他操作|可以理解为受约束的 AIDL，主要提供数据源的 CRUD 操作 | 一对多的进程间数据共享
| Socket | 可以通过网络传输字节流，支持一对多并发实时通信 | 实现细节稍微有点烦琐，不支持直接的RPC | 网络数据交换

## Binder
Binder 是 Android 中的一个类，实现了 IBinder 接口。从 IPC 角度来说，Binder 是 Android 中的一种扩进程通信方方式。从 Android 应用层来说，Binder 是客户端和服务器端进行通信的媒介，当 bindService 的时候，服务端会返回一个包含了服务端业务调用的 Binder 对象。

Binder 相较于传统 IPC 来说更适合于Android系统，具体原因的包括如下三点：
- Binder 本身是 C/S 架构的，这一点更符合 Android 系统的架构
- 性能上更有优势：管道，消息队列，Socket 的通讯都需要两次数据拷贝，而 Binder 只需要一次。要知道，对于系统底层的 IPC 形式，少一次数据拷贝，对整体性能的影响是非常之大的
- 安全性更好：传统 IPC 形式，无法得到对方的身份标识（UID/GID)，而在使用 Binder IPC 时，这些身份标示是跟随调用过程而自动传递的。Server 端很容易就可以知道 Client 端的身份，非常便于做安全检查

示例：

- **新建AIDL接口文件**
  
``RemoteService.aidl``
```java
package com.example.mystudyapplication3;

interface IRemoteService {

    int getUserId();

}
```

系统会自动生成 ``IRemoteService.java``:

```java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 */
package com.example.mystudyapplication3;
// Declare any non-default types here with import statements
//import com.example.mystudyapplication3.IUserBean;

public interface IRemoteService extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.example.mystudyapplication3.IRemoteService {
        private static final java.lang.String DESCRIPTOR = "com.example.mystudyapplication3.IRemoteService";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.example.mystudyapplication3.IRemoteService interface,
         * generating a proxy if needed.
         */
        public static com.example.mystudyapplication3.IRemoteService asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.example.mystudyapplication3.IRemoteService))) {
                return ((com.example.mystudyapplication3.IRemoteService) iin);
            }
            return new com.example.mystudyapplication3.IRemoteService.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            java.lang.String descriptor = DESCRIPTOR;
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_getUserId: {
                    data.enforceInterface(descriptor);
                    int _result = this.getUserId();
                    reply.writeNoException();
                    reply.writeInt(_result);
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        private static class Proxy implements com.example.mystudyapplication3.IRemoteService {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public int getUserId() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                int _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getUserId, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.readInt();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
        }

        static final int TRANSACTION_getUserId = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    }

    public int getUserId() throws android.os.RemoteException;
}
```

| 方法 | 含义
|--|--
| DESCRIPTOR | Binder 的唯一标识，一般用当前的 Binder 的类名表示
| asInterface(IBinder obj) | 将服务端的 Binder 对象成客户端所需的 AIDL 接口类型对象，这种转换过程是区分进程的，如果位于同一进程，返回的就是 Stub 对象本身，否则返回的是系统封装后的 Stub.proxy 对象。
| asBinder | 用于返回当前 Binder 对象
| onTransact | 运行在服务端中的 Binder 线程池中，远程请求会通过系统底层封装后交由此方法来处理

| 定向 tag | 含义
|--|--
| in | 数据只能由客户端流向服务端，服务端将会收到客户端对象的完整数据，客户端对象不会因为服务端对传参的修改而发生变动。
| out | 数据只能由服务端流向客户端，服务端将会收到客户端对象，该对象不为空，但是它里面的字段为空，但是在服务端对该对象作任何修改之后客户端的传参对象都会同步改动。
| inout | 服务端将会接收到客户端传来对象的完整信息，并且客户端将会同步服务端对该对象的任何变动。

## AIDL 通信
Android Interface Definition Language

使用示例：
- **新建AIDL接口文件**
```java
// RemoteService.aidl
package com.example.mystudyapplication3;

interface IRemoteService {

    int getUserId();

}
```
- **创建远程服务**
```java
public class RemoteService extends Service {

    private int mId = -1;

    private Binder binder = new IRemoteService.Stub() {

        @Override
        public int getUserId() throws RemoteException {
            return mId;
        }
    };

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        mId = 1256;
        return binder;
    }
}
```
- **声明远程服务**
```java
<service
    android:name=".RemoteService"
    android:process=":aidl" />
```
- **绑定远程服务**
```java
public class MainActivity extends AppCompatActivity {

    public static final String TAG = "wzq";

    IRemoteService iRemoteService;
    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            iRemoteService = IRemoteService.Stub.asInterface(service);
            try {
                Log.d(TAG, String.valueOf(iRemoteService.getUserId()));
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            iRemoteService = null;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        bindService(new Intent(MainActivity.this, RemoteService.class), mConnection, Context.BIND_AUTO_CREATE);
    }
}
```

## Messenger
Messenger可以在不同进程中传递 Message 对象，在Message中放入我们需要传递的数据，就可以轻松地实现数据的进程间传递了。Messenger 是一种轻量级的 IPC 方案，底层实现是 AIDL。

# Window / WindowManager
## Window 概念与分类
Window 是一个抽象类，它的具体实现是 PhoneWindow。WindowManager 是外界访问 Window 的入口，Window 的具体实现位于 WindowManagerService 中，WindowManager 和 WindowManagerService 的交互是一个 IPC 过程。Android 中所有的视图都是通过 Window 来呈现，因此 Window 实际是 View 的直接管理者。

| Window 类型 | 说明 | 层级
|--|--|--
| Application Window | 对应着一个 Activity | 1~99
| Sub Window | 不能单独存在，只能附属在父 Window 中，如 Dialog 等 | 1000~1999
| System Window | 需要权限声明，如 Toast 和 系统状态栏等 | 2000~2999

## Window 的内部机制
Window 是一个抽象的概念，每一个 Window 对应着一个 View 和一个 ViewRootImpl。Window 实际是不存在的，它是以 View 的形式存在。对 Window 的访问必须通过 WindowManager，WindowManager 的实现类是 WindowManagerImpl：

``WindowManagerImpl.java``
```java
@Override
public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow);
}

@Override
public void updateViewLayout(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.updateViewLayout(view, params);
}

@Override
public void removeView(View view) {
    mGlobal.removeView(view, false);
}
```

WindowManagerImpl 没有直接实现 Window 的三大操作，而是全部交给 WindowManagerGlobal 处理，WindowManagerGlobal 以工厂的形式向外提供自己的实例：

``WindowManagerGlobal.java``
```java
// 添加
public void addView(View view, ViewGroup.LayoutParams params,
        Display display, Window parentWindow) {
    ···
    // 子 Window 的话需要调整一些布局参数
    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
    if (parentWindow != null) {
        parentWindow.adjustLayoutParamsForSubWindow(wparams);
    } else {
        ···
    }
    ViewRootImpl root;
    View panelParentView = null;
    synchronized (mLock) {
        // 新建一个 ViewRootImpl，并通过其 setView 来更新界面完成 Window 的添加过程
        ···
        root = new ViewRootImpl(view.getContext(), display);
        view.setLayoutParams(wparams);
        mViews.add(view);
        mRoots.add(root);
        mParams.add(wparams);
        // do this last because it fires off messages to start doing things
        try {
            root.setView(view, wparams, panelParentView);
        } catch (RuntimeException e) {
            // BadTokenException or InvalidDisplayException, clean up.
            if (index >= 0) {
                removeViewLocked(index, true);
            }
            throw e;
        }
    }
}

// 删除
@UnsupportedAppUsage
public void removeView(View view, boolean immediate) {
    ···
    synchronized (mLock) {
        int index = findViewLocked(view, true);
        View curView = mRoots.get(index).getView();
        removeViewLocked(index, immediate);
        ···
    }
}

private void removeViewLocked(int index, boolean immediate) {
    ViewRootImpl root = mRoots.get(index);
    View view = root.getView();
    if (view != null) {
        InputMethodManager imm = InputMethodManager.getInstance();
        if (imm != null) {
            imm.windowDismissed(mViews.get(index).getWindowToken());
        }
    }
    boolean deferred = root.die(immediate);
    if (view != null) {
        view.assignParent(null);
        if (deferred) {
            mDyingViews.add(view);
        }
    }
}

// 更新
public void updateViewLayout(View view, ViewGroup.LayoutParams params) {
    ···
    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams)params;
    view.setLayoutParams(wparams);
    synchronized (mLock) {
        int index = findViewLocked(view, true);
        ViewRootImpl root = mRoots.get(index);
        mParams.remove(index);
        mParams.add(index, wparams);
        root.setLayoutParams(wparams, false);
    }
}
```

在 ViewRootImpl 中最终会通过 WindowSession 来完成 Window 的添加、更新、删除工作，mWindowSession 的类型是 IWindowSession，是一个 Binder 对象，真正地实现类是 Session，是一个 IPC 过程。

## Window 的创建过程
### Activity 的 Window 创建过程
在 Activity 的创建过程中，最终会由 ActivityThread 的 performLaunchActivity() 来完成整个启动过程，该方法内部会通过类加载器创建 Activity 的实例对象，并调用 attach 方法关联一系列上下文环境变量。在 Activity 的 attach 方法里，系统会创建所属的 Window 对象并设置回调接口，然后在 Activity 的 setContentView 方法中将视图附属在 Window 上：

``Activity.java``
```java
final void attach(Context context, ActivityThread aThread,
        Instrumentation instr, IBinder token, int ident,
        Application application, Intent intent, ActivityInfo info,
        CharSequence title, Activity parent, String id,
        NonConfigurationInstances lastNonConfigurationInstances,
        Configuration config, String referrer, IVoiceInteractor voiceInteractor,
        Window window, ActivityConfigCallback activityConfigCallback) {
    attachBaseContext(context);

    mFragments.attachHost(null /*parent*/);

    mWindow = new PhoneWindow(this, window, activityConfigCallback);
    mWindow.setWindowControllerCallback(this);
    mWindow.setCallback(this);
    mWindow.setOnWindowDismissedCallback(this);
    mWindow.getLayoutInflater().setPrivateFactory(this);
    if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
        mWindow.setSoftInputMode(info.softInputMode);
    }
    if (info.uiOptions != 0) {
        mWindow.setUiOptions(info.uiOptions);
    }
    ···
}
···

public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID);
    initWindowDecorActionBar();
}

```

``PhoneWindow.java``
```java
@Override
public void setContentView(int layoutResID) {
    if (mContentParent == null) { // 如果没有 DecorView，就创建
        installDecor();
    } else {
        mContentParent.removeAllViews();
    }
    mLayoutInflater.inflate(layoutResID, mContentParent);
    final Callback cb = getCallback();
    if (cb != null && !isDestroyed()) {
        // 回调 Activity 的 onContentChanged 方法通知 Activity 视图已经发生改变
        cb.onContentChanged();
    }
}
```

这个时候 DecorView 还没有被 WindowManager 正式添加。在 ActivityThread 的 handleResumeActivity 方法中，首先会调用 Activity 的 onResume 方法，接着调用 Activity 的 makeVisible()，完成 DecorView 的添加和显示过程：

``Activity.java``
```java
void makeVisible() {
    if (!mWindowAdded) {
        ViewManager wm = getWindowManager();
        wm.addView(mDecor, getWindow().getAttributes());
        mWindowAdded = true;
    }
    mDecor.setVisibility(View.VISIBLE);
}
```

### Dialog 的 Window 创建过程
Dialog 的 Window 的创建过程和 Activity 类似，创建同样是通过 PolicyManager 的 makeNewWindow 方法完成的，创建后的对象实际就是 PhoneWindow。当 Dialog 被关闭时，会通过 WindowManager 来移除 DecorView：mWindowManager.removeViewImmediate(mDecor)。

``Dialog.java``
```java
Dialog(@NonNull Context context, @StyleRes int themeResId, boolean      createContextThemeWrapper) {
    ···
    mWindowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);

    final Window w = new PhoneWindow(mContext);
    mWindow = w;
    w.setCallback(this);
    w.setOnWindowDismissedCallback(this);
    w.setOnWindowSwipeDismissedCallback(() -> {
        if (mCancelable) {
            cancel();
        }
    });
    w.setWindowManager(mWindowManager, null, null);
    w.setGravity(Gravity.CENTER);

    mListenersHandler = new ListenersHandler(this);
}
```

普通 Dialog 必须采用 Activity 的 Context，采用 Application 的 Context 就会报错，是因为应用 token 所导致，应用 token 一般只有 Activity 拥有。系统 Window 比较特殊，不需要 token。

### Toast 的 Window 创建过程
Toast 属于系统 Window ，由于其具有定时取消功能，所以系统采用了 Handler。Toast 的内部有两类 IPC 过程，第一类是 Toast 访问 NotificationManagerService，第二类是 NotificationManagerService 回调 Toast 里的 TN 接口。

Toast 内部的视图由两种方式，一种是系统默认的样式，另一种是 setView 指定一个自定义 View，它们都对应 Toast 的一个内部成员 mNextView。

``Toast.java``
```java
public void show() {
    if (mNextView == null) {
        throw new RuntimeException("setView must have been called");
    }

    INotificationManager service = getService();
    String pkg = mContext.getOpPackageName();
    TN tn = mTN;
    tn.mNextView = mNextView;

    try {
        service.enqueueToast(pkg, tn, mDuration);
    } catch (RemoteException e) {
        // Empty
    }
}
···

public void cancel() {
    mTN.cancel();
}

```

``NotificationManagerService.java``
```java
private void showNextToastLocked() {
    ToastRecord record = mToastQueue.get(0);
    while (record != null) {
        if (DBG) Slog.d(TAG, "Show pkg=" + record.pkg + " callback=" + record.callback);
        try {
            record.callback.show();
            scheduleTimeoutLocked(record, false);
            return;
        } catch (RemoteException e) {
            Slog.w(TAG, "Object died trying to show notification " + record.callback
                    + " in package " + record.pkg);
            // remove it from the list and let the process die
            int index = mToastQueue.indexOf(record);
            if (index >= 0) {
                mToastQueue.remove(index);
            }
            keepProcessAliveLocked(record.pid);
            if (mToastQueue.size() > 0) {
                record = mToastQueue.get(0);
            } else {
                record = null;
            }
        }
    }
}

···
private void scheduleTimeoutLocked(ToastRecord r, boolean immediate)
{
    Message m = Message.obtain(mHandler, MESSAGE_TIMEOUT, r);
    long delay = immediate ? 0 : (r.duration == Toast.LENGTH_LONG ? LONG_DELAY : SHORT_DELAY);
    mHandler.removeCallbacksAndMessages(r);
    mHandler.sendMessageDelayed(m, delay);
}
```

# Bitmap
![](https://upload-images.jianshu.io/upload_images/2618044-cd996dd172cce293.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

## 配置信息与压缩方式
**Bitmap 中有两个内部枚举类：**
- Config 是用来设置颜色配置信息
- CompressFormat 是用来设置压缩方式

| Config | 单位像素所占字节数 | 解析 
|-------|-------|------
| Bitmap.Config.ALPHA_8 | 1 | 颜色信息只由透明度组成，占8位 
| Bitmap.Config.ARGB_4444 | 2 |颜色信息由rgba四部分组成，每个部分都占4位，总共占16位 
| Bitmap.Config.ARGB_8888 | 4 |颜色信息由rgba四部分组成，每个部分都占8位，总共占32位。是Bitmap默认的颜色配置信息，也是最占空间的一种配置
| Bitmap.Config.RGB_565 | 2 | 颜色信息由rgb三部分组成，R占5位，G占6位，B占5位，总共占16位
| RGBA_F16 | 8 | Android 8.0 新增（更丰富的色彩表现HDR）
| HARDWARE | Special | Android 8.0 新增 （Bitmap直接存储在graphic memory）

> 通常我们优化 Bitmap 时，当需要做性能优化或者防止 OOM，我们通常会使用 Bitmap.Config.RGB_565 这个配置，因为 Bitmap.Config.ALPHA_8 只有透明度，显示一般图片没有意义，Bitmap.Config.ARGB_4444 显示图片不清楚， Bitmap.Config.ARGB_8888 占用内存最多。

| CompressFormat | 解析 
|-------|-------
| Bitmap.CompressFormat.JPEG | 表示以 JPEG 压缩算法进行图像压缩，压缩后的格式可以是 ``.jpg`` 或者 ``.jpeg``，是一种有损压缩 |
| Bitmap.CompressFormat.PNG | 颜色信息由 rgba 四部分组成，每个部分都占 4 位，总共占 16 位 |
| Bitmap.Config.ARGB_8888 | 颜色信息由 rgba 四部分组成，每个部分都占 8 位，总共占 32 位。是 Bitmap 默认的颜色配置信息，也是最占空间的一种配置
| Bitmap.Config.RGB_565 | 颜色信息由 rgb 三部分组成，R 占 5 位，G 占 6 位，B 占 5 位，总共占 16 位

## 常用操作
### 裁剪、缩放、旋转、移动
```java
Matrix matrix = new Matrix();  
// 缩放 
matrix.postScale(0.8f, 0.9f);  
// 左旋，参数为正则向右旋
matrix.postRotate(-45);  
// 平移, 在上一次修改的基础上进行再次修改 set 每次操作都是最新的 会覆盖上次的操作
matrix.postTranslate(100, 80);
// 裁剪并执行以上操作
Bitmap bitmap = Bitmap.createBitmap(source, 0, 0, source.getWidth(), source.getHeight(), matrix, true);
````
> 虽然Matrix还可以调用postSkew方法进行倾斜操作，但是却不可以在此时创建Bitmap时使用。

### Bitmap与Drawable转换
```java
// Drawable -> Bitmap
public static Bitmap drawableToBitmap(Drawable drawable) {
    Bitmap bitmap = Bitmap.createBitmap(drawable.getIntrinsicWidth(), drawable.getIntrinsicHeight(), drawable.getOpacity() != PixelFormat.OPAQUE ? Bitmap.Config.ARGB_8888 : Bitmap.Config.RGB_565);
    Canvas canvas = new Canvas(bitmap);
    drawable.setBounds(0, 0, drawable.getIntrinsicWidth(), drawable.getIntrinsicHeight();
    drawable.draw(canvas);
    return bitmap;
}

// Bitmap -> Drawable
public static Drawable bitmapToDrawable(Resources resources, Bitmap bm) {
    Drawable drawable = new BitmapDrawable(resources, bm);
    return drawable;
}
```

### 保存与释放
```java
Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.test);
File file = new File(getFilesDir(),"test.jpg");
if(file.exists()){
    file.delete();
}
try {
    FileOutputStream outputStream=new FileOutputStream(file);
    bitmap.compress(Bitmap.CompressFormat.JPEG,90,outputStream);
    outputStream.flush();
    outputStream.close();
} catch (FileNotFoundException e) {
    e.printStackTrace();
} catch (IOException e) {
    e.printStackTrace();
}
//释放bitmap的资源，这是一个不可逆转的操作
bitmap.recycle();
```

### 图片压缩
```java
public static Bitmap compressImage(Bitmap image) {
    if (image == null) {
        return null;
    }
    ByteArrayOutputStream baos = null;
    try {
        baos = new ByteArrayOutputStream();
        image.compress(Bitmap.CompressFormat.JPEG, 100, baos);
        byte[] bytes = baos.toByteArray();
        ByteArrayInputStream isBm = new ByteArrayInputStream(bytes);
        Bitmap bitmap = BitmapFactory.decodeStream(isBm);
        return bitmap;
    } catch (OutOfMemoryError e) {
        e.printStackTrace();
    } finally {
        try {
            if (baos != null) {
                baos.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    return null;
}
```

## BitmapFactory
### Bitmap创建流程
![](https://upload-images.jianshu.io/upload_images/2618044-9c2046ca5054da05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)


### Option类
| 常用方法 | 说明
|-----|------
| boolean inJustDecodeBounds | 如果设置为true，不获取图片，不分配内存，但会返回图片的高度宽度信息
| int inSampleSize | 图片缩放的倍数
| int outWidth | 获取图片的宽度值
| int outHeight | 获取图片的高度值
| int inDensity | 用于位图的像素压缩比
| int inTargetDensity | 用于目标位图的像素压缩比（要生成的位图）
| byte[] inTempStorage | 创建临时文件，将图片存储
| boolean inScaled | 设置为true时进行图片压缩，从inDensity到inTargetDensity
| boolean inDither | 如果为true,解码器尝试抖动解码
| Bitmap.Config inPreferredConfig | 设置解码器这个值是设置色彩模式，默认值是ARGB_8888，在这个模式下，一个像素点占用4bytes空间，一般对透明度不做要求的话，一般采用RGB_565模式，这个模式下一个像素点占用2bytes
| String outMimeType | 设置解码图像
| boolean inPurgeable | 当存储Pixel的内存空间在系统内存不足时是否可以被回收
| boolean inInputShareable | inPurgeable为true情况下才生效，是否可以共享一个InputStream
| boolean inPreferQualityOverSpeed | 为true则优先保证Bitmap质量其次是解码速度
| boolean inMutable | 配置Bitmap是否可以更改，比如：在Bitmap上隔几个像素加一条线段
| int inScreenDensity | 当前屏幕的像素密度

### 基本使用
```java
try {
    FileInputStream fis = new FileInputStream(filePath);
    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    // 设置inJustDecodeBounds为true后，再使用decodeFile()等方法，并不会真正的分配空间，即解码出来的Bitmap为null，但是可计算出原始图片的宽度和高度，即options.outWidth和options.outHeight
    BitmapFactory.decodeFileDescriptor(fis.getFD(), null, options);
    float srcWidth = options.outWidth;
    float srcHeight = options.outHeight;
    int inSampleSize = 1;

    if (srcHeight > height || srcWidth > width) {
        if (srcWidth > srcHeight) {
            inSampleSize = Math.round(srcHeight / height);
        } else {
            inSampleSize = Math.round(srcWidth / width);
        }
    }

    options.inJustDecodeBounds = false;
    options.inSampleSize = inSampleSize;

    return BitmapFactory.decodeFileDescriptor(fis.getFD(), null, options);
} catch (Exception e) {
    e.printStackTrace();
}
```

## 内存回收
```java
if(bitmap != null && !bitmap.isRecycled()){ 
    // 回收并且置为null
    bitmap.recycle(); 
    bitmap = null; 
} 
```
Bitmap 类的构造方法都是私有的，所以开发者不能直接 new 出一个 Bitmap 对象，只能通过 BitmapFactory 类的各种静态方法来实例化一个 Bitmap。仔细查看 BitmapFactory 的源代码可以看到，生成 Bitmap 对象最终都是通过 JNI 调用方式实现的。所以，加载 Bitmap 到内存里以后，是包含两部分内存区域的。简单的说，一部分是Java 部分的，一部分是 C 部分的。这个 Bitmap 对象是由 Java 部分分配的，不用的时候系统就会自动回收了，但是那个对应的 C 可用的内存区域，虚拟机是不能直接回收的，这个只能调用底层的功能释放。所以需要调用 recycle() 方法来释放 C 部分的内存。从 Bitmap 类的源代码也可以看到，recycle() 方法里也的确是调用了 JNI 方法了的。

# 屏幕适配
## 单位
- dpi
每英寸像素数(dot per inch)  

- dp  
密度无关像素 - 一种基于屏幕物理密度的抽象单元。 这些单位相对于 160 dpi 的屏幕，因此一个 dp 是 160 dpi 屏幕上的一个 px。 dp 与像素的比率将随着屏幕密度而变化，但不一定成正比。为不同设备的 UI 元素的实际大小提供了一致性。

- sp  
与比例无关的像素 - 这与 dp 单位类似，但它也可以通过用户的字体大小首选项进行缩放。建议在指定字体大小时使用此单位，以便根据屏幕密度和用户偏好调整它们。
```
dpi = px / inch

density = dpi / 160

dp = px / density
```
## 头条适配方案
```java
private static void setCustomDensity(@NonNull Activity activity, @NonNull final Application application) {
    final DisplayMetrics appDisplayMetrics = application.getResources().getDisplayMetrics();
    if (sNoncompatDensity == 0) {
        sNoncompatDensity = appDisplayMetrics.density;
        sNoncompatScaledDensity = appDisplayMetrics.scaledDensity;
        // 监听字体切换
        application.registerComponentCallbacks(new ComponentCallbacks() {
            @Override
            public void onConfigurationChanged(Configuration newConfig) {
                if (newConfig != null && newConfig.fontScale > 0) {
                    sNoncompatScaledDensity = application.getResources().getDisplayMetrics().scaledDensity;
                }
            }

            @Override
            public void onLowMemory() {

            }
        });
    }
    
    // 适配后的dpi将统一为360dpi
    final float targetDensity = appDisplayMetrics.widthPixels / 360;
    final float targetScaledDensity = targetDensity * (sNoncompatScaledDensity / sNoncompatDensity);
    final int targetDensityDpi = (int)(160 * targetDensity);

    appDisplayMetrics.density = targetDensity;
    appDisplayMetrics.scaledDensity = targetScaledDensity;
    appDisplayMetrics.densityDpi = targetDensityDpi;

    final DisplayMetrics activityDisplayMetrics = activity.getResources().getDisplayMetrics();
    activityDisplayMetrics.density = targetDensity;
    activityDisplayMetrics.scaledDensity = targetScaledDensity;
    activityDisplayMetrics.densityDpi = targetDensityDpi
}
```

## 刘海屏适配
- Android P 刘海屏适配方案

Android P 支持最新的全面屏以及为摄像头和扬声器预留空间的凹口屏幕。通过全新的 DisplayCutout 类，可以确定非功能区域的位置和形状，这些区域不应显示内容。要确定这些凹口屏幕区域是否存在及其位置，使用 getDisplayCutout() 函数。

| DisplayCutout 类方法 | 说明
|--|--
| getBoundingRects() | 返回Rects的列表，每个Rects都是显示屏上非功能区域的边界矩形
| getSafeInsetLeft () | 返回安全区域距离屏幕左边的距离，单位是px
| getSafeInsetRight () | 返回安全区域距离屏幕右边的距离，单位是px
| getSafeInsetTop () | 返回安全区域距离屏幕顶部的距离，单位是px
| getSafeInsetBottom() | 返回安全区域距离屏幕底部的距离，单位是px

Android P 中 WindowManager.LayoutParams 新增了一个布局参数属性 layoutInDisplayCutoutMode：

| 模式 | 模式说明
|--|--
| LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT | 只有当DisplayCutout完全包含在系统栏中时，才允许窗口延伸到DisplayCutout区域。 否则，窗口布局不与DisplayCutout区域重叠。
| LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER | 该窗口决不允许与DisplayCutout区域重叠。
| LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES | 该窗口始终允许延伸到屏幕短边上的DisplayCutout区域。

- Android P 之前的刘海屏适配

不同厂商的刘海屏适配方案不尽相同，需分别查阅各自的开发者文档。

# Context
Context 本身是一个抽象类，是对一系列系统服务接口的封装，包括：内部资源、包、类加载、I/O操作、权限、主线程、IPC 和组件启动等操作的管理。ContextImpl, Activity, Service, Application 这些都是 Context 的直接或间接子类, 关系如下:

![](http://gityuan.com/images/context/context.jpg)

ContextWrapper是代理Context的实现，简单地将其所有调用委托给另一个Context（mBase）。

Application、Activity、Service通过``attach() ``调用父类ContextWrapper的``attachBaseContext()``, 从而设置父类成员变量 mBase 为 ContextImpl 对象, ContextWrapper 的核心工作都是交给 mBase(ContextImpl) 来完成，这样可以子类化 Context 以修改行为而无需更改原始 Context。

# SharedPreferences
SharedPreferences 采用key-value（键值对）形式, 主要用于轻量级的数据存储, 尤其适合保存应用的配置参数, 但不建议使用 SharedPreferences 来存储大规模的数据, 可能会降低性能.

SharedPreferences采用xml文件格式来保存数据, 该文件所在目录位于 ``/data/data/<package name>/shared_prefs``，如：
```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
   <string name="blog">https://github.com/JasonWu1111/Android-Review</string>
</map>
```

从Android N开始, 创建的 SP 文件模式, 不允许 ``MODE_WORLD_READABLE`` 和 ``MODE_WORLD_WRITEABLE`` 模块, 否则会直接抛出异常 SecurityException。 ``MODE_MULTI_PROCESS`` 这种多进程的方式也是 Google 不推荐的方式, 后续同样会不再支持。

当设置 MODE_MULTI_PROCESS 模式, 则每次 getSharedPreferences 过程, 会检查 SP 文件上次修改时间和文件大小, 一旦所有修改则会重新从磁盘加载文件。

## 获取方式
### getPreferences
Activity.getPreferences(mode): 以当前 Activity 的类名作为 SP 的文件名. 即 xxxActivity.xml
``Activity.java``
```java
public SharedPreferences getPreferences(int mode) {
    return getSharedPreferences(getLocalClassName(), mode);
}
```

### getDefaultSharedPreferences
PreferenceManager.getDefaultSharedPreferences(Context): 以包名加上 _preferences 作为文件名, 以 MODE_PRIVATE 模式创建 SP 文件. 即 packgeName_preferences.xml.
```java
public static SharedPreferences getDefaultSharedPreferences(Context context) {
    return context.getSharedPreferences(getDefaultSharedPreferencesName(context),
           getDefaultSharedPreferencesMode());
}
```

### getSharedPreferences
直接调用 Context.getSharedPreferences(name, mode)，所有的方法最终都是调用到如下方法：
```java
class ContextImpl extends Context {
    private ArrayMap<String, File> mSharedPrefsPaths;

    public SharedPreferences getSharedPreferences(String name, int mode) {
        File file;
        synchronized (ContextImpl.class) {
            if (mSharedPrefsPaths == null) {
                mSharedPrefsPaths = new ArrayMap<>();
            }
            //先从mSharedPrefsPaths查询是否存在相应文件
            file = mSharedPrefsPaths.get(name);
            if (file == null) {
                //如果文件不存在, 则创建新的文件 
                file = getSharedPreferencesPath(name);
                mSharedPrefsPaths.put(name, file);
            }
        }
 
        return getSharedPreferences(file, mode);
    }
}
```

## 架构
![](http://gityuan.com/images/sp/shared_preference.jpg)

SharedPreferences 与 Editor 只是两个接口. SharedPreferencesImpl 和 EditorImpl 分别实现了对应接口。另外, ContextImpl 记录着 SharedPreferences 的重要数据。

``putxxx()`` 操作把数据写入到EditorImpl.mModified；

``apply()/commit()`` 操作先调用 commitToMemory(), 将数据同步到 SharedPreferencesImpl 的 mMap, 并保存到 MemoryCommitResult 的 mapToWriteToDisk，再调用 enqueueDiskWrite(), 写入到磁盘文件; 先之前把原有数据保存到 .bak 为后缀的文件,用于在写磁盘的过程出现任何异常可恢复数据;

``getxxx()`` 操作从 SharedPreferencesImpl.mMap 读取数据.

## apply / commit
- apply 没有返回值, commit 有返回值能知道修改是否提交成功  
- apply 是将修改提交到内存，再异步提交到磁盘文件，而 commit 是同步的提交到磁盘文件
- 多并发的提交 commit 时，需等待正在处理的 commit 数据更新到磁盘文件后才会继续往下执行，从而降低效率; 而 apply 只是原子更新到内存，后调用 apply 函数会直接覆盖前面内存数据，从一定程度上提高很多效率。

## 注意
- 强烈建议不要在 sp 里面存储特别大的 key/value，有助于减少卡顿 / anr
- 不要高频地使用 apply，尽可能地批量提交
- 不要使用 MODE_MULTI_PROCESS
- 高频写操作的 key 与高频读操作的 key 可以适当地拆分文件，由于减少同步锁竞争
- 不要连续多次 edit()，应该获取一次获取 edit()，然后多次执行 putxxx()，减少内存波动

# 消息机制
## Handler 机制
Handler 有两个主要用途：（1）安排 Message 和 runnables 在将来的某个时刻执行; （2）将要在不同于自己的线程上执行的操作排入队列。(在多个线程并发更新UI的同时保证线程安全。)

Android 规定访问 UI 只能在主线程中进行，因为 Android 的 UI 控件不是线程安全的，多线程并发访问会导致 UI 控件处于不可预期的状态。为什么系统不对 UI 控件的访问加上锁机制？缺点有两个：加锁会让 UI 访问的逻辑变得复杂；其次锁机制会降低 UI 访问的效率。如果子线程访问 UI，那么程序就会抛出异常。ViewRootImpl 对UI操作做了验证，这个验证工作是由 ViewRootImpl的 ``checkThread`` 方法完成：

``ViewRootImpl.java``
```java
void checkThread() {
    if (mThread != Thread.currentThread()) {
        throw new CalledFromWrongThreadException(
                "Only the original thread that created a view hierarchy can touch its views.");
    }
}
```

- Message：Handler 接收和处理的消息对象
- MessageQueue：Message 的队列，先进先出，每一个线程最多可以拥有一个
- Looper：消息泵，是 MessageQueue 的管理者，会不断从 MessageQueue 中取出消息，并将消息分给对应的 Handler 处理，每个线程只有一个 Looper。

Handler 创建的时候会采用当前线程的 Looper 来构造消息循环系统，需要注意的是，线程默认是没有 Looper 的，直接使用 Handler 会报错，如果需要使用 Handler 就必须为线程创建 Looper，因为默认的 UI 主线程，也就是 ActivityThread，ActivityThread 被创建的时候就会初始化 Looper，这也是在主线程中默认可以使用 Handler 的原因。

## 工作原理
### ThreadLocal
ThreadLocal 是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，其他线程则无法获取。Looper、ActivityThread 以及 AMS 中都用到了 ThreadLocal。当不同线程访问同一个ThreadLocal 的 get方法，ThreadLocal 内部会从各自的线程中取出一个数组，然后再从数组中根据当前 ThreadLcoal 的索引去查找对应的value值。
``ThreadLocal.java``
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

···
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

### MessageQueue
MessageQueue主要包含两个操作：插入和读取。读取操作本身会伴随着删除操作，插入和读取对应的方法分别是 ``enqueueMessage`` 和 ``next``。MessageQueue 内部实现并不是用的队列，实际上通过一个单链表的数据结构来维护消息列表。next 方法是一个无限循环的方法，如果消息队列中没有消息，那么 next 方法会一直阻塞。当有新消息到来时，next 方法会放回这条消息并将其从单链表中移除。

``MessageQueue.java``
```java
boolean enqueueMessage(Message msg, long when) {
    ···
    synchronized (this) {
        ···
        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
···
Message next() {
    // Return here if the message loop has already quit and been disposed.
    // This can happen if the application tries to restart a looper after quit
    // which is not supported.
    ···
    for (;;) {
        ···
        synchronized (this) {
            // Try to retrieve the next message.  Return if found.
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    // Next message is not ready.  Set a timeout to wake up when it is ready.
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // Got a message.
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    msg.markInUse();
                    return msg;
                }
            } else {
                // No more messages.
                nextPollTimeoutMillis = -1;
            }
            ···
        }

        // Run the idle handlers.
        // We only ever reach this code block during the first iteration.
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mPendingIdleHandlers[i];
            mPendingIdleHandlers[i] = null; // release the reference to the handler

            boolean keep = false;
            try {
                keep = idler.queueIdle();
            } catch (Throwable t) {
                Log.wtf(TAG, "IdleHandler threw exception", t);
            }

            if (!keep) {
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }

        // Reset the idle handler count to 0 so we do not run them again.
        pendingIdleHandlerCount = 0;

        // While calling an idle handler, a new message could have been delivered
        // so go back and look again for a pending message without waiting.
        nextPollTimeoutMillis = 0;
    }
}
```

### Looper
Looper 会不停地从 MessageQueue 中 查看是否有新消息，如果有新消息就会立刻处理，否则会一直阻塞。
``Looper.java``
```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```

可通过 Looper.prepare() 为当前线程创建一个 Looper：
```java
new Thread("Thread#2") {
    @Override
    public void run() {
        Looper.prepare();
        Handler handler = new Handler();
        Looper.loop();
    }
}.start();
```

除了 prepare 方法外，Looper 还提供了 ``prepareMainLooper`` 方法，主要是给 ActivityThread 创建 Looper 使用，本质也是通过 prepare 方法实现的。由于主线程的 Looper 比较特殊，所以 Looper 提供了一个 getMainLooper 方法来获取主线程的 Looper。

Looper 提供了 ``quit`` 和 ``quitSafely`` 来退出一个 Looper，二者的区别是：``quit`` 会直接退出 Looper，而 ``quitSafly`` 只是设定一个退出标记，然后把消息队列中的已有消息处理完毕后才安全地退出。Looper 退出后，通过 Handler 发送的消息会失败，这个时候 Handler 的 send 方法会返回 false。因此在不需要的时候应终止 Looper。

``Looper.java``
```java
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;
    ···
    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }
        ···
        try {
            msg.target.dispatchMessage(msg);
            dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
        ···
        msg.recycleUnchecked();
    }
}
```
loop 方法是一个死循环，唯一跳出循环的方式是 MessageQueue 的 next 方法返回了null。当 Looper 的 quit 方法被调用时，Looper就会调用 MessageQueue 的 quit 或者 qutiSafely 方法来通知消息队列退出，当消息队列被标记为退出状态时，它的 next 方法就会返回 null。loop 方法会调用 MessageQueue 的 next 方法来获取新消息，而 next 是一个阻塞操作，当没有消息时，next 会一直阻塞，导致 loop 方法一直阻塞。Looper 处理这条消息： msg.target.dispatchMessage(msg)，这里的 msg.target 是发送这条消息的 Handler 对象。

### Handler
Handler 的工作主要包含消息的发送和接收的过程。消息的发送可以通过 post/send 的一系列方法实现，post 最终也是通过send来实现的。

![](https://img-blog.csdnimg.cn/20181220142659447)

# 线程异步
应用启动时，系统会为应用创建一个名为“主线程”的执行线程( UI 线程)。 此线程非常重要，因为它负责将事件分派给相应的用户界面小部件，其中包括绘图事件。 此外，它也是应用与 Android UI 工具包组件（来自 ``android.widget`` 和 ``android.view`` 软件包的组件）进行交互的线程。

系统不会为每个组件实例创建单独的线程。运行于同一进程的所有组件均在 UI 线程中实例化，并且对每个组件的系统调用均由该线程进行分派。 因此，响应系统回调的方法（例如，报告用户操作的 onKeyDown() 或生命周期回调方法）始终在进程的 UI 线程中运行。

Android 的单线程模式必须遵守两条规则:
- 不要阻塞 UI 线程
- 不要在 UI 线程之外访问 Android UI 工具包

为解决此问题，Android 提供了几种途径来从其他线程访问 UI 线程:
- ``Activity.runOnUiThread(Runnable)``
- ``View.post(Runnable)``
- ``View.postDelayed(Runnable, long)``

## AsyncTask
AsyncTask 封装了 Thread 和 Handler，并不适合特别耗时的后台任务，对于特别耗时的任务来说，建议使用线程池。

### 基本使用
| 方法 | 说明
|--|--
| onPreExecute() | 异步任务执行前调用，用于做一些准备工作
| doInBackground(Params...params) | 用于执行异步任务，此方法中可以通过 publishProgress 方法来更新任务的进度，publishProgress 会调用 onProgressUpdate 方法
| onProgressUpdate | 在主线程中执行，后台任务的执行进度发生改变时调用
| onPostExecute | 在主线程中执行，在异步任务执行之后 

```java
import android.os.AsyncTask;

public class DownloadTask extends AsyncTask<String, Integer, Boolean> {

    @Override
    protected void onPreExecute() {
        super.onPreExecute();
    }

    @Override
    protected Boolean doInBackground(String... strings) {
        return null;
    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        super.onProgressUpdate(values);
    }

    @Override
    protected void onPostExecute(Boolean aBoolean) {
        super.onPostExecute(aBoolean);
    }
}
```

- 异步任务的实例必须在 UI 线程中创建，即 AsyncTask 对象必须在UI线程中创建。
- execute(Params... params)方法必须在UI线程中调用。
- 不要手动调用 onPreExecute()，doInBackground()，onProgressUpdate()，onPostExecute() 这几个方法。
- 不能在 doInBackground() 中更改UI组件的信息。
- 一个任务实例只能执行一次，如果执行第二次将会抛出异常。
- execute() 方法会让同一个进程中的 AsyncTask 串行执行，如果需要并行，可以调用 executeOnExcutor 方法。

### 工作原理
``AsyncTask.java``
```java
@MainThread
public final AsyncTask<Params, Progress, Result> execute(Params... params) {
    return executeOnExecutor(sDefaultExecutor, params);
}

@MainThread
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
        Params... params) {
    if (mStatus != Status.PENDING) {
        switch (mStatus) {
            case RUNNING:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task is already running.");
            case FINISHED:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task has already been executed "
                        + "(a task can be executed only once)");
        }
    }

    mStatus = Status.RUNNING;

    onPreExecute();

    mWorker.mParams = params;
    exec.execute(mFuture);

    return this;
}
```
sDefaultExecutor 是一个串行的线程池，一个进程中的所有的 AsyncTask 全部在该线程池中执行。AysncTask 中有两个线程池（SerialExecutor 和 THREAD_POOL_EXECUTOR）和一个 Handler（InternalHandler），其中线程池 SerialExecutor 用于任务的排队，THREAD_POOL_EXECUTOR 用于真正地执行任务，InternalHandler 用于将执行环境从线程池切换到主线程。

``AsyncTask.java``
```java
private static Handler getMainHandler() {
    synchronized (AsyncTask.class) {
        if (sHandler == null) {
            sHandler = new InternalHandler(Looper.getMainLooper());
        }
        return sHandler;
    }
}

private static class InternalHandler extends Handler {
    public InternalHandler(Looper looper) {
        super(looper);
    }

    @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
    @Override
    public void handleMessage(Message msg) {
        AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
        switch (msg.what) {
            case MESSAGE_POST_RESULT:
                // There is only one result
                result.mTask.finish(result.mData[0]);
                break;
            case MESSAGE_POST_PROGRESS:
                result.mTask.onProgressUpdate(result.mData);
                break;
        }
    }
}


private Result postResult(Result result) {
    @SuppressWarnings("unchecked")
    Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
            new AsyncTaskResult<Result>(this, result));
    message.sendToTarget();
    return result;
}
```

## HandlerThread
HandlerThread 集成了 Thread，却和普通的 Thread 有显著的不同。普通的 Thread 主要用于在 run 方法中执行一个耗时任务，而 HandlerThread 在内部创建了消息队列，外界需要通过 Handler 的消息方式通知 HanderThread 执行一个具体的任务。

``HandlerThread.java``
```java
@Override
public void run() {
    mTid = Process.myTid();
    Looper.prepare();
    synchronized (this) {
        mLooper = Looper.myLooper();
        notifyAll();
    }
    Process.setThreadPriority(mPriority);
    onLooperPrepared();
    Looper.loop();
    mTid = -1;
}
```

## IntentService
IntentService 可用于执行后台耗时的任务，当任务执行后会自动停止，由于其是 Service 的原因，它的优先级比单纯的线程要高，所以 IntentService 适合执行一些高优先级的后台任务。在实现上，IntentService 封装了 HandlerThread 和 Handler。

``IntentService.java``
```java
@Override
public void onCreate() {
    // TODO: It would be nice to have an option to hold a partial wakelock
    // during processing, and to have a static startService(Context, Intent)
    // method that would launch the service & hand off a wakelock.

    super.onCreate();
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    thread.start();

    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
}
```

IntentService 第一次启动时，会在 onCreatea 方法中创建一个 HandlerThread，然后使用的 Looper 来构造一个 Handler 对象 mServiceHandler，这样通过 mServiceHandler 发送的消息最终都会在 HandlerThread 中执行。每次启动 IntentService，它的 onStartCommand 方法就会调用一次，onStartCommand 中处理每个后台任务的 Intent，onStartCommand 调用了 onStart 方法：

``IntentService.java``
```java
private final class ServiceHandler extends Handler {
    public ServiceHandler(Looper looper) {
        super(looper);
    }

    @Override
    public void handleMessage(Message msg) {
        onHandleIntent((Intent)msg.obj);
        stopSelf(msg.arg1);
    }
}

···

@Override
public void onStart(@Nullable Intent intent, int startId) {
    Message msg = mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    mServiceHandler.sendMessage(msg);
}
```

可以看出，IntentService 仅仅是通过 mServiceHandler 发送了一个消息，这个消息会在 HandlerThread 中被处理。mServiceHandler 收到消息后，会将 Intent 对象传递给 onHandlerIntent 方法中处理，执行结束后，通过 stopSelf(int startId) 来尝试停止服务。（stopSelf() 会立即停止服务，而 stopSelf(int startId) 则会等待所有的消息都处理完毕后才终止服务）。

## 线程池
线程池的优点有以下：
- 重用线程池中的线程，避免因为线程的创建和销毁带来性能开销。
- 能有效控制线程池的最大并发数，避免大量的线程之间因互相抢占系统资源而导致的阻塞现象。
- 能够对线程进行管理，并提供定时执行以及定间隔循环执行等功能。

java 中，ThreadPoolExecutor 是线程池的真正实现：

``ThreadPoolExecutor.java``
```java
/**
    * Creates a new {@code ThreadPoolExecutor} with the given initial
    * parameters.
    *
    * @param corePoolSize 核心线程数
    * @param maximumPoolSize 最大线程数
    * @param keepAliveTime 非核心线程闲置的超时时长
    * @param unit 用于指定 keepAliveTime 参数的时间单位
    * @param 任务队列，通过线程池的 execute 方法提交的 Runnable 对象会存储在这个参数中
    * @param threadFactory 线程工厂，用于创建新线程
    * @param handler 任务队列已满或者是无法成功执行任务时调用
    */
public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler) {
    ···
}
```

| 类型 | 创建方法 | 说明
|--|--|--
| FixedThreadPool | Executors.newFixedThreadPool(int nThreads) | 一种线程数量固定的线程池，只有核心线程并且不会被回收，没有超时机制
| CachedThreadPool | Executors.newCachedThreadPool() | 一种线程数量不定的线程池，只有非核心线程，当线程都处于活动状态时，会创建新线程来处理新任务，否则会利用空闲的线程，超时时长为60s
| ScheduledThreadPool | Executors.newScheduledThreadPool(int corePoolSize) | 核心线程数是固定的，非核心线程数没有限制，非核心线程闲置时立刻回收，主要用于执行定时任务和固定周期的重复任务
| SingleThreadExecutor | Executors.newSingleThreadExecutor() | 只有一个核心线程，确保所有任务在同一线程中按顺序执行

# RecyclerView 优化
- 数据处理和视图加载分离：数据的处理逻辑尽可能放在异步处理，onBindViewHolder 方法中只处理数据填充到视图中。

- 数据优化：分页拉取远端数据，对拉取下来的远端数据进行缓存，提升二次加载速度；对于新增或者删除数据通过 DiffUtil 来进行局部刷新数据，而不是一味地全局刷新数据。

示例
```java
public class AdapterDiffCallback extends DiffUtil.Callback {
    
    private List<String> mOldList;
    private List<String> mNewList;
    
    public AdapterDiffCallback(List<String> oldList, List<String> newList) {
        mOldList = oldList;
        mNewList = newList;
        DiffUtil.DiffResult
    }
    
    @Override
    public int getOldListSize() {
        return mOldList.size();
    }

    @Override
    public int getNewListSize() {
        return mNewList.size();
    }

    @Override
    public boolean areItemsTheSame(int oldItemPosition, int newItemPosition) {
        return mOldList.get(oldItemPosition).getClass().equals(mNewList.get(newItemPosition).getClass());
    }

    @Override
    public boolean areContentsTheSame(int oldItemPosition, int newItemPosition) {
        return mOldList.get(oldItemPosition).equals(mNewList.get(newItemPosition));
    }
}
```
```java
DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(new AdapterDiffCallback(oldList, newList));
diffResult.dispatchUpdatesTo(mAdapter);
```

- 布局优化：减少布局层级，简化 ItemView

- 升级 RecycleView 版本到 25.1.0 及以上使用 Prefetch 功能

- 通过重写 RecyclerView.onViewRecycled(holder) 来回收资源

- 如果 Item 高度是固定的话，可以使用 RecyclerView.setHasFixedSize(true); 来避免 requestLayout 浪费资源

- 对 ItemView 设置监听器，不要对每个 Item 都调用 addXxListener，应该大家公用一个 XxListener，根据 ID 来进行不同的操作，优化了对象的频繁创建带来的资源消耗

- 如果多个 RecycledView 的 Adapter 是一样的，比如嵌套的 RecyclerView 中存在一样的 Adapter，可以通过设置 RecyclerView.setRecycledViewPool(pool)，来共用一个 RecycledViewPool。

# Webview
## 基本使用
### WebView
```java
// 获取当前页面的URL
public String getUrl();
// 获取当前页面的原始URL(重定向后可能当前url不同)
// 就是http headers的Referer参数，loadUrl时为null
public String getOriginalUrl();
// 获取当前页面的标题
public String getTitle();
// 获取当前页面的favicon
public Bitmap getFavicon();
// 获取当前页面的加载进度
public int getProgress();

// 通知WebView内核网络状态
// 用于设置JS属性`window.navigator.isOnline`和产生HTML5事件`online/offline`
public void setNetworkAvailable(boolean networkUp)

// 设置初始缩放比例
public void setInitialScale(int scaleInPercent)；

```

### WebSettings
```java
WebSettings settings = web.getSettings();

// 存储(storage)
// 启用HTML5 DOM storage API，默认值 false
settings.setDomStorageEnabled(true); 
// 启用Web SQL Database API，这个设置会影响同一进程内的所有WebView，默认值 false
// 此API已不推荐使用，参考：https://www.w3.org/TR/webdatabase/
settings.setDatabaseEnabled(true);  
// 启用Application Caches API，必需设置有效的缓存路径才能生效，默认值 false
// 此API已废弃，参考：https://developer.mozilla.org/zh-CN/docs/Web/HTML/Using_the_application_cache
settings.setAppCacheEnabled(true); 
settings.setAppCachePath(context.getCacheDir().getAbsolutePath());

// 定位(location)
settings.setGeolocationEnabled(true);

// 是否保存表单数据
settings.setSaveFormData(true);
// 是否当webview调用requestFocus时为页面的某个元素设置焦点，默认值 true
settings.setNeedInitialFocus(true);  

// 是否支持viewport属性，默认值 false
// 页面通过`<meta name="viewport" ... />`自适应手机屏幕
settings.setUseWideViewPort(true);
// 是否使用overview mode加载页面，默认值 false
// 当页面宽度大于WebView宽度时，缩小使页面宽度等于WebView宽度
settings.setLoadWithOverviewMode(true);
// 布局算法
settings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.NORMAL);

// 是否支持Javascript，默认值false
settings.setJavaScriptEnabled(true); 
// 是否支持多窗口，默认值false
settings.setSupportMultipleWindows(false);
// 是否可用Javascript(window.open)打开窗口，默认值 false
settings.setJavaScriptCanOpenWindowsAutomatically(false);

// 资源访问
settings.setAllowContentAccess(true); // 是否可访问Content Provider的资源，默认值 true
settings.setAllowFileAccess(true);    // 是否可访问本地文件，默认值 true
// 是否允许通过file url加载的Javascript读取本地文件，默认值 false
settings.setAllowFileAccessFromFileURLs(false);  
// 是否允许通过file url加载的Javascript读取全部资源(包括文件,http,https)，默认值 false
settings.setAllowUniversalAccessFromFileURLs(false);

// 资源加载
settings.setLoadsImagesAutomatically(true); // 是否自动加载图片
settings.setBlockNetworkImage(false);       // 禁止加载网络图片
settings.setBlockNetworkLoads(false);       // 禁止加载所有网络资源

// 缩放(zoom)
settings.setSupportZoom(true);          // 是否支持缩放
settings.setBuiltInZoomControls(false); // 是否使用内置缩放机制
settings.setDisplayZoomControls(true);  // 是否显示内置缩放控件

// 默认文本编码，默认值 "UTF-8"
settings.setDefaultTextEncodingName("UTF-8");
settings.setDefaultFontSize(16);        // 默认文字尺寸，默认值16，取值范围1-72
settings.setDefaultFixedFontSize(16);   // 默认等宽字体尺寸，默认值16
settings.setMinimumFontSize(8);         // 最小文字尺寸，默认值 8
settings.setMinimumLogicalFontSize(8);  // 最小文字逻辑尺寸，默认值 8
settings.setTextZoom(100);              // 文字缩放百分比，默认值 100

// 字体
settings.setStandardFontFamily("sans-serif");   // 标准字体，默认值 "sans-serif"
settings.setSerifFontFamily("serif");           // 衬线字体，默认值 "serif"
settings.setSansSerifFontFamily("sans-serif");  // 无衬线字体，默认值 "sans-serif"
settings.setFixedFontFamily("monospace");       // 等宽字体，默认值 "monospace"
settings.setCursiveFontFamily("cursive");       // 手写体(草书)，默认值 "cursive"
settings.setFantasyFontFamily("fantasy");       // 幻想体，默认值 "fantasy"


if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    // 用户是否需要通过手势播放媒体(不会自动播放)，默认值 true
    settings.setMediaPlaybackRequiresUserGesture(true);
}
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    // 5.0以上允许加载http和https混合的页面(5.0以下默认允许，5.0+默认禁止)
    settings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
}
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    // 是否在离开屏幕时光栅化(会增加内存消耗)，默认值 false
    settings.setOffscreenPreRaster(false);
}

if (isNetworkConnected(context)) {
    // 根据cache-control决定是否从网络上取数据
    settings.setCacheMode(WebSettings.LOAD_DEFAULT);
} else {
    // 没网，离线加载，优先加载缓存(即使已经过期)
    settings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);
}

// deprecated
settings.setRenderPriority(WebSettings.RenderPriority.HIGH);
settings.setDatabasePath(context.getDir("database", Context.MODE_PRIVATE).getPath());
settings.setGeolocationDatabasePath(context.getFilesDir().getPath());

```

### WebViewClient
```java
// 拦截页面加载，返回true表示宿主app拦截并处理了该url，否则返回false由当前WebView处理
// 此方法在API24被废弃，不处理POST请求
public boolean shouldOverrideUrlLoading(WebView view, String url) {
    return false;
}

// 拦截页面加载，返回true表示宿主app拦截并处理了该url，否则返回false由当前WebView处理
// 此方法添加于API24，不处理POST请求，可拦截处理子frame的非http请求
@TargetApi(Build.VERSION_CODES.N)
public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
    return shouldOverrideUrlLoading(view, request.getUrl().toString());
}

// 此方法废弃于API21，调用于非UI线程
// 拦截资源请求并返回响应数据，返回null时WebView将继续加载资源
public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
    return null;
}

// 此方法添加于API21，调用于非UI线程
// 拦截资源请求并返回数据，返回null时WebView将继续加载资源
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
public WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request) {
    return shouldInterceptRequest(view, request.getUrl().toString());
}

// 页面(url)开始加载
public void onPageStarted(WebView view, String url, Bitmap favicon) {
}

// 页面(url)完成加载
public void onPageFinished(WebView view, String url) {
}

// 将要加载资源(url)
public void onLoadResource(WebView view, String url) {
}

// 这个回调添加于API23，仅用于主框架的导航
// 通知应用导航到之前页面时，其遗留的WebView内容将不再被绘制。
// 这个回调可以用来决定哪些WebView可见内容能被安全地回收，以确保不显示陈旧的内容
// 它最早被调用，以此保证WebView.onDraw不会绘制任何之前页面的内容，随后绘制背景色或需要加载的新内容。
// 当HTTP响应body已经开始加载并体现在DOM上将在随后的绘制中可见时，这个方法会被调用。
// 这个回调发生在文档加载的早期，因此它的资源(css,和图像)可能不可用。
// 如果需要更细粒度的视图更新，查看 postVisualStateCallback(long, WebView.VisualStateCallback).
// 请注意这上边的所有条件也支持 postVisualStateCallback(long ,WebView.VisualStateCallback)
public void onPageCommitVisible(WebView view, String url) {
}

// 此方法废弃于API23
// 主框架加载资源时出错
public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
}

// 此方法添加于API23
// 加载资源时出错，通常意味着连接不到服务器
// 由于所有资源加载错误都会调用此方法，所以此方法应尽量逻辑简单
@TargetApi(Build.VERSION_CODES.M)
public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
    if (request.isForMainFrame()) {
        onReceivedError(view, error.getErrorCode(), error.getDescription().toString(), request.getUrl().toString());
    }
}

// 此方法添加于API23
// 在加载资源(iframe,image,js,css,ajax...)时收到了 HTTP 错误(状态码>=400)
public void onReceivedHttpError(WebView view, WebResourceRequest request, WebResourceResponse errorResponse) {
}


// 是否重新提交表单，默认不重发
public void onFormResubmission(WebView view, Message dontResend, Message resend) {
    dontResend.sendToTarget();
}

// 通知应用可以将当前的url存储在数据库中，意味着当前的访问url已经生效并被记录在内核当中。
// 此方法在网页加载过程中只会被调用一次，网页前进后退并不会回调这个函数。
public void doUpdateVisitedHistory(WebView view, String url, boolean isReload) {
}

// 加载资源时发生了一个SSL错误，应用必需响应(继续请求或取消请求)
// 处理决策可能被缓存用于后续的请求，默认行为是取消请求
public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
    handler.cancel();
}

// 此方法添加于API21，在UI线程被调用
// 处理SSL客户端证书请求，必要的话可显示一个UI来提供KEY。
// 有三种响应方式：proceed()/cancel()/ignore()，默认行为是取消请求
// 如果调用proceed()或cancel()，Webview 将在内存中保存响应结果且对相同的"host:port"不会再次调用 onReceivedClientCertRequest
// 多数情况下，可通过KeyChain.choosePrivateKeyAlias启动一个Activity供用户选择合适的私钥
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
public void onReceivedClientCertRequest(WebView view, ClientCertRequest request) {
    request.cancel();
}

// 处理HTTP认证请求，默认行为是取消请求
public void onReceivedHttpAuthRequest(WebView view, HttpAuthHandler handler, String host, String realm) {
    handler.cancel();
}

// 通知应用有个已授权账号自动登陆了
public void onReceivedLoginRequest(WebView view, String realm, String account, String args) {
}
// 给应用一个机会处理按键事件
// 如果返回true，WebView不处理该事件，否则WebView会一直处理，默认返回false
public boolean shouldOverrideKeyEvent(WebView view, KeyEvent event) {
    return false;
}

// 处理未被WebView消费的按键事件
// WebView总是消费按键事件，除非是系统按键或shouldOverrideKeyEvent返回true
// 此方法在按键事件分派时被异步调用
public void onUnhandledKeyEvent(WebView view, KeyEvent event) {
    super.onUnhandledKeyEvent(view, event);
}

// 通知应用页面缩放系数变化
public void onScaleChanged(WebView view, float oldScale, float newScale) {
} 

```

### WebChromeClient
```java
// 获得所有访问历史项目的列表，用于链接着色。
public void getVisitedHistory(ValueCallback<String[]> callback) {
}

// <video /> 控件在未播放时，会展示为一张海报图，HTML中可通过它的'poster'属性来指定。
// 如果未指定'poster'属性，则通过此方法提供一个默认的海报图。
public Bitmap getDefaultVideoPoster() {
    return null;
}

// 当全屏的视频正在缓冲时，此方法返回一个占位视图(比如旋转的菊花)。
public View getVideoLoadingProgressView() {
    return null;
}

// 接收当前页面的加载进度
public void onProgressChanged(WebView view, int newProgress) {
}

// 接收文档标题
public void onReceivedTitle(WebView view, String title) {
}

// 接收图标(favicon)
public void onReceivedIcon(WebView view, Bitmap icon) {
}

// Android中处理Touch Icon的方案
// http://droidyue.com/blog/2015/01/18/deal-with-touch-icon-in-android/index.html
public void onReceivedTouchIconUrl(WebView view, String url, boolean precomposed) {
}

// 通知应用当前页进入了全屏模式，此时应用必须显示一个包含网页内容的自定义View
public void onShowCustomView(View view, CustomViewCallback callback) {
}

// 通知应用当前页退出了全屏模式，此时应用必须隐藏之前显示的自定义View
public void onHideCustomView() {
}


// 显示一个alert对话框
public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
    return false;
}

// 显示一个confirm对话框
public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
    return false;
}

// 显示一个prompt对话框
public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
    return false;
}

// 显示一个对话框让用户选择是否离开当前页面
public boolean onJsBeforeUnload(WebView view, String url, String message, JsResult result) {
    return false;
}


// 指定源的网页内容在没有设置权限状态下尝试使用地理位置API。
// 从API24开始，此方法只为安全的源(https)调用，非安全的源会被自动拒绝
public void onGeolocationPermissionsShowPrompt(String origin, GeolocationPermissions.Callback callback) {
}

// 当前一个调用 onGeolocationPermissionsShowPrompt() 取消时，隐藏相关的UI。
public void onGeolocationPermissionsHidePrompt() {
}

// 通知应用打开新窗口
public boolean onCreateWindow(WebView view, boolean isDialog, boolean isUserGesture, Message resultMsg) {
    return false;
}

// 通知应用关闭窗口
public void onCloseWindow(WebView window) {
}

// 请求获取取焦点
public void onRequestFocus(WebView view) {
}

// 通知应用网页内容申请访问指定资源的权限(该权限未被授权或拒绝)
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
public void onPermissionRequest(PermissionRequest request) {
    request.deny();
}

// 通知应用权限的申请被取消，隐藏相关的UI。
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
public void onPermissionRequestCanceled(PermissionRequest request) {
}

// 为'<input type="file" />'显示文件选择器，返回false使用默认处理
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
    return false;
}

// 接收JavaScript控制台消息
public boolean onConsoleMessage(ConsoleMessage consoleMessage) {
    return false;
} 

```

## Webview 加载优化
- 使用本地资源替代

可以 将一些资源文件放在本地的 asset s目录, 然后重 写WebViewClient 的 ``shouldInterceptRequest`` 方法，对访问地址进行拦截，当 url 地址命中本地配置的url时，使用本地资源替代，否则就使用网络上的资源。

```java
mWebview.setWebViewClient(new WebViewClient() {   
     // 设置不用系统浏览器打开,
    @Override    
    public boolean shouldOverrideUrlLoading(WebView view, String url) {      
        view.loadUrl(url);     
        return true;    
    }   
         
    @Override    
    public WebResourceResponse shouldInterceptRequest(WebView view, String url) {      // 如果命中本地资源, 使用本地资源替代      
        if (mDataHelper.hasLocalResource(url)){         
             WebResourceResponse response = mDataHelper.getReplacedWebResourceResponse(getApplicationContext(), url);          
            if (response != null) {              
                return response;
            }      
        }      
        return super.shouldInterceptRequest(view, url);    
    }   
    
    @TargetApi(VERSION_CODES.LOLLIPOP)@Override    
    public WebResourceResponse shouldInterceptRequest(WebView view,WebResourceRequest request) {      
        String url = request.getUrl().toString();      
        if (mDataHelper.hasLocalResource(url)) {         
            WebResourceResponse response =  mDataHelper.getReplacedWebResourceResponse(getApplicationContext(), url);          
            if (response != null) {              
                return response;          
            }      
        }      
        return super.shouldInterceptRequest(view, request);    
    }
}); 
```

- WebView初始化慢，可以在初始化同时先请求数据，让后端和网络不要闲着。
  
- 后端处理慢，可以让服务器分trunk输出，在后端计算的同时前端也加载网络静态资源。

- 脚本执行慢，就让脚本在最后运行，不阻塞页面解析。

- 同时，合理的预加载、预缓存可以让加载速度的瓶颈更小。

- WebView初始化慢，就随时初始化好一个WebView待用。

- DNS和链接慢，想办法复用客户端使用的域名和链接。

- 脚本执行慢，可以把框架代码拆分出来，在请求页面之前就执行好。
  
![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2017/9a2f8beb.png)

## 内存泄漏
直接 new WebView 并传入 application context 代替在 XML 里面声明以防止 activity 引用被滥用，能解决90+%的 WebView 内存泄漏。

```java
vWeb =  new WebView(getContext().getApplicationContext());
container.addView(vWeb);
```

销毁 WebView

```java
if (vWeb != null) {
    vWeb.setWebViewClient(null);
    vWeb.setWebChromeClient(null);
    vWeb.loadDataWithBaseURL(null, "", "text/html", "utf-8", null);
    vWeb.clearHistory();

    ((ViewGroup) vWeb.getParent()).removeView(vWeb);
    vWeb.destroy();
    vWeb = null;
} 
```
