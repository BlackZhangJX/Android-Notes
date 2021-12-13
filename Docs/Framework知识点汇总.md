# Handler

## Handler机制实现原理(一)宏观理论分析与Message源码分析

### Message:

定义：

```java
public` `final` `class` `Message ``implements` `Parcelable
```

Message类是个final类，就是说不能被继承，同时Message类实现了Parcelable接口，我们知道android提供了一种新的类型：Parcel。**本类被用作封装数据的容器，是链表结构，**有个属性next和sPool，这两个变量是不同的，具体什么不同看下文。

文档描述：

```java
Defines a message containing a description and arbitrary data object that can be sent to a {@link Handler}.  This object contains two extra int fields and an
extra object field that allow you to not do allocations in many cases. 
```

定义一个包含任意类型的描述数据对象，此对象可以发送给Handler。对象包含两个额外的int字段和一个额外的对象字段，这样可以使得在很多情况下不用做分配工作。尽管Message的构造器是公开的，但是获取Message对象的最好方法是调用Message.obtain()或者Handler.obtainMessage(), 这样是从一个可回收对象池中获取Message对象。

#### 看一下全局变量：有好多存数据的对象。

```java
public int what;
public int arg1;
public int arg2;
public Object obj;
public Messenger replyTo;
/*package*/ int flags;
/*package*/ long when;
 
/*package*/ Bundle data;
 
/*package*/ Handler target;
 
/*package*/ Runnable callback;
 
// sometimes we store linked lists of these things
/*package*/ Message next;
 
private static final Object sPoolSync = new Object();
private static Message sPool;
private static int sPoolSize = 0;
 
private static final int MAX_POOL_SIZE = 50;
 
private static boolean gCheckRecycle = true;
```

1. what：用户定义消息代码以便收件人可以识别这是哪一个Message。每个Handler用它自己的名称空间为消息代码,所以您不需要担心你的Handler与其他handler冲突。
2. arg1、arg2：如果只是想向message内放一些整数值，可以使用arg1和arg2来代替setData方法。
3. obj：发送给接收器的任意对象。当使用Message对象在线程间传递消息时，如果它包含一个Parcelable的结构类（不是由应用程序实现的类），此字段必须为非空（non-null）。其他的数据传输则使用setData(Bundle)方法。注意Parcelable对象是从FROYO版本以后才开始支持的。
4. replyTo：指明此message发送到何处的可选Messenger对象。具体的使用方法由发送者和接受者决定。
5. FLAG_IN_USE：判断Message是否在使用（ default 包内可见）
6. FLAG_ASYNCHRONOUS：如果设置message是异步的。
7. FLAGS_TO_CLEAR_ON_COPY_FROM：明确在copyFrom方法
8. 其他参数都比较简单，不详述

#### Obtain方法：

　

```java
//从全局池中返回一个新的Message实例。在大多数情况下这样可以避免分配新的对象。
//是一个静态方法
public static Message obtain() {
        synchronized (sPoolSync) {
            if (sPool != null) {
                Message m = sPool;
                sPool = m.next;
                m.next = null;
                m.flags = 0; // clear in-use flag
                sPoolSize--;
                return m;
            }
        }
        return new Message();
    }
```

　　在看它一系列的重载方法：

```java
/**
     * Same as {@link #obtain()}, but copies the values of an existing
     * message (including its target) into the new one.
     * @param orig Original message to copy.
     * @return A Message object from the global pool.
     */
public static Message obtain(Message orig) {
        Message m = obtain();
        m.what = orig.what;
        m.arg1 = orig.arg1;
        m.arg2 = orig.arg2;
        m.obj = orig.obj;
        m.replyTo = orig.replyTo;
        m.sendingUid = orig.sendingUid;
        if (orig.data != null) {
            m.data = new Bundle(orig.data);
        }
        m.target = orig.target;
        m.callback = orig.callback;
 
        return m;
    }
 /**
     设置target
     */
public static Message obtain(Handler h) {
        Message m = obtain();
        m.target = h;
 
        return m;
    }
 /**
     * Same as {@link #obtain(Handler)}, but assigns a callback Runnable on
     * the Message that is returned.
     * @param h  Handler to assign to the returned Message object's <em>target</em> member.
     * @param callback Runnable that will execute when the message is handled.
     * @return A Message object from the global pool.
     */
    public static Message obtain(Handler h, Runnable callback) {
        Message m = obtain();
        m.target = h;
        m.callback = callback;
 
        return m;
    }
/**
     * Same as {@link #obtain()}, but sets the values of the <em>target</em>, <em>what</em>,
     * <em>arg1</em>, <em>arg2</em>, and <em>obj</em> members.
      。。。。
     * @param obj  The <em>obj</em> value to set.
     * @return  A Message object from the global pool.
     */
    public static Message obtain(Handler h, int what,
            int arg1, int arg2, Object obj) {
        Message m = obtain();
        m.target = h;
        m.what = what;
        m.arg1 = arg1;
        m.arg2 = arg2;
        m.obj = obj;
 
        return m;
    }
```

还有几个没列举出来，都是先调用obtain()方法，然后把获取的Message实例加上各种参数。代码一目了然。。。

#### recycle():回收当前message到全局池

　

```java
/**
     * Return a Message instance to the global pool.
     * <p>
     * You MUST NOT touch the Message after calling this function because it has
     * effectively been freed.  It is an error to recycle a message that is currently
     * enqueued or that is in the process of being delivered to a Handler.
     * </p>
     */
    public void recycle() {
        if (isInUse()) {
            if (gCheckRecycle) {
                throw new IllegalStateException("This message cannot be recycled because it "
                        + "is still in use.");
            }
            return;
        }
        recycleUnchecked();
    }
 
    /**
     * Recycles a Message that may be in-use.
     * Used internally by the MessageQueue and Looper when disposing of queued Messages.
     */
    void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = -1;
        when = 0;
        target = null;
        callback = null;
        data = null;
 
        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
```

向全局池中返回一个Message实例。一定不能在调用此函数后再使用Message——它实际上已经被释放。

getWhen：

```java
/**
    * Return the targeted delivery time of this message, in milliseconds.
    */
   public long getWhen() {
       return when;
   }
```

返回此消息的传输时间，以毫秒为单位。

 

 setTarget，getTarget：

```java
//设置handler和返回handler
public void setTarget(Handler target) {
        this.target = target;
    }
    /**
     * Retrieve the a {@link android.os.Handler Handler} implementation that
     * will receive this message. The object must implement
     * {@link android.os.Handler#handleMessage(android.os.Message)
     * Handler.handleMessage()}. Each Handler has its own name-space for
     * message codes, so you do not need to
     * worry about yours conflicting with other handlers.
     */
    public Handler getTarget() {
        return target;
    }
```

获取将接收此消息的Handler对象。此对象必须要实现Handler.handleMessage()方法。每个handler各自包含自己的消息代码，所以不用担心自定义的消息跟其他handlers有冲突。

### setData：

 设置一个可以是任何类型值的bundle。

```java
/**
     * Sets a Bundle of arbitrary data values. Use arg1 and arg2 members
     * as a lower cost way to send a few simple integer values, if you can.
     * @see #getData()
     * @see #peekData()
     */
    public void setData(Bundle data) {
        this.data = data;
    }
```

　　getData，peekData

```java
public Bundle getData() {
        if (data == null) {
            data = new Bundle();
        }
         
        return data;
    }
public Bundle peekData() {
        return data;
}
```

### 发送消息的一些方法：

```java
/**向Handler发送此消息，getTarget()方法可以获取此Handler。如果这个字段没有设置会抛出个空指针异常。
     * Sends this Message to the Handler specified by {@link #getTarget}.
     * Throws a null pointer exception if this field has not been set.
     */
    public void sendToTarget() {
        target.sendMessage(this);
    }　　
```

#### 构造方法：

```java
/** Constructor (but the preferred way to get a Message is to call {@link #obtain() Message.obtain()}).
    */
    public Message() {
    }
//推荐使用Message.obtain()
```

#### writeToParcel：

```Java
public void writeToParcel(Parcel dest, int flags) {
        if (callback != null) {
            throw new RuntimeException(
                "Can't marshal callbacks across processes.");
        }
        dest.writeInt(what);
        dest.writeInt(arg1);
        dest.writeInt(arg2);
        if (obj != null) {
            try {
                Parcelable p = (Parcelable)obj;
                dest.writeInt(1);
                dest.writeParcelable(p, flags);
            } catch (ClassCastException e) {
                throw new RuntimeException(
                    "Can't marshal non-Parcelable objects across processes.");
            }
        } else {
            dest.writeInt(0);
        }
        dest.writeLong(when);
        dest.writeBundle(data);
        Messenger.writeMessengerOrNullToParcel(replyTo, dest);
        dest.writeInt(sendingUid);
    }
```

　将类的数据写入外部提供的Parcel中和从Parcel中读取数据。

## Handler机制实现原理（二）MessageQueue的源码分析

> 看源码有一段时间了，越来越能从代码中感觉到工程师们满满的激情，无论是基础Java语法还是高级的语言特性都被发挥的淋漓尽致，写的恰到好处。分析源码的过程，何尝不是与大神们进行灵魂沟的过程。

MessageQueue属于低层类且依附于Looper，Looper外其他类不应该单独创建它，如果想使用MessageQueue可以从Looper类中得到它。

### 消息队列存储原理

再上一章Message源码分析中我们知道了Message内部维持了一个链表缓存池来避免重复创建Message对象造成的额外消耗，以静态属性`private static Message sPool`作为缓存池链表头，`Message next;`作为链表的next指针。

有意思的是Message对象中`next`指针的不止用于链表缓存池，在MessageQueue中也采用同样的方法存储消息对象：



```java
public final class MessageQueue {
    ......
    Message mMessages;
    ......
}
```

上面代码中的`mMessages`就是MessageQueue用来维持消息队列的链表头，至于它是如何存储的，后面再说。

### 使用JNI实现的native方法

MessageQueue的源码调用了多个的C/C++方法，这类方法使用前都会用关键字`native`声明一下。

这些方法所属的底层C++代码创建了属于native层自己的`NativeMessageQueue`和`NativeLooper`消息模型。它们对Java层作用其实就是**控制线程是否阻塞**。

当我们想要从MessageQueue中取出消息时，碰巧队列是空的或即将取出的消息还没到被处理时间，那么我们就需要将线程阻塞掉等待队列中有消息时再取出。

下面就是MessageQueue中的`native`方法：



```java
    // 初始化
    private native static long nativeInit();
    // 注销
    private native static void nativeDestroy(long ptr);
    // 让线程阻塞timeoutMillis毫秒
    private native void nativePollOnce(long ptr, int timeoutMillis); 
    // 立刻唤醒线程
    private native static void nativeWake(long ptr);
    // 线程是否处于阻塞状态
    private native static boolean nativeIsPolling(long ptr);
    // 设置文件描述符
    private native static void nativeSetFileDescriptorEvents(long ptr, int fd, int events);
```

**文件描述符与ePoll指令**

消息队列里控制线程阻塞状态的`native`代码本质是用Linux指令`ePoll`完成的，在这之前需要先了解一点，Linux内核依靠“文件描述符（file descriptor）”来完成所有的文件读写访问操作，它更像是一个文件的索引值。而我们用到的`ePoll`指令就是用来监听文件描述符是否有可I/O操作的。*（这都是一些Linux相关知识）*

### 创建与销毁

上面说了MessageQueue是依附于Looper的，所以本节分析的创建与销毁方法其实都是给Looper调用的，MessageQueue只提供了一个带参的构造方法来创建对象：



```java
    // 当前MessageQueue是否可以退出
    private final boolean mQuitAllowed;

    // native层中NativeMessageQueue队列指针的地址
    // mPtr等于0时表示退出队列
    private long mPtr;

    // native层代码，创建native层中NativeMessageQueue
    private native static long nativeInit();

    // 构造方法
    MessageQueue(boolean quitAllowed) {
        mQuitAllowed = quitAllowed;
        // 执行native层方法
        mPtr = nativeInit();
    }
```

构造方法中`boolean quitAllowed`参数的意思是当前这个MessageQueue是否可以手动退出，为什么要控制能否手动退出呢？这里先说一个结论：Android 系统中要求UI线程不可手动退出，而其他Worker线程则**全部都是**可以的。*（具体的操作在Looper和UI线程中）*

**那么退出是什么意思呢？**

退出就是当前这个MessageQueue停止服务，将队列中已存在的所有消息全部清空，看看源码中退出方法都做了什么：



```java
    // 是否已经退出了
    private boolean mQuitting;

    // native方法，退出队列
    private native static void nativeDestroy(long ptr);


    // 退出队列
    void quit(boolean safe) {
        
        // 如果不是可以手动退出的，抛出异常
        if (!mQuitAllowed) {
            throw new IllegalStateException("Main thread not allowed to quit.");
        }

        synchronized (this) {
            // 如果已经退出了直接结束方法
            if (mQuitting) {
                return;
            }
            // 标记为已退出状态
            mQuitting = true;

            // 两种清除队列中消息的方法
            if (safe) {
                removeAllFutureMessagesLocked();
            } else {
                removeAllMessagesLocked();
            }

            // 注销
            nativeWake(mPtr);
        }
    }
```

方法`quit(boolean safe)`中的参数`safe`决定了到底执行哪种清除消息的方法：

- `removeAllMessagesLocked()`，简单暴力直接清除掉队列中所有的消息。
- `removeAllFutureMessagesLocked()`，清除掉可能还没有被处理的消息。

`removeAllMessagesLocked()`方法的逻辑很简单，从队列头中取消息，有一个算一个，全部拿出来回收掉。：



```csharp
    private void removeAllMessagesLocked() {
        Message p = mMessages;
        while (p != null) {
            Message n = p.next;
            p.recycleUnchecked();
            p = n;
        }
        mMessages = null;
    }
```

然而，`removeAllFutureMessagesLocked()`方法的逻辑稍微多一点：



```csharp
    private void removeAllFutureMessagesLocked() {
    
        // 获取当前系统时间
        final long now = SystemClock.uptimeMillis();
        Message p = mMessages;
        if (p != null) {
            // 判断当前消息对象的预处理时间是否晚于当前时间
            if (p.when > now) {
                // 如果当前消息对象的预处理时间晚于当前时间直接全部暴力清除
                removeAllMessagesLocked();
            } else {
                Message n;
                // 如果当前消息对象的预处理时间并不晚于当前时间
                // 说明有可能这个消息正在被分发处理
                // 那么就跳过这个消息往后找晚于当前时间的消息
                for (;;) {
                    n = p.next;
                    if (n == null) {
                        return;
                    }
                    if (n.when > now) {
                        // 如果找到了晚于当前时间的消息结束循环
                        break;
                    }
                    p = n;
                }
                p.next = null;
                do {
                    // n就是那个晚于当前时间的消息
                    // 从n开始之后的消息全部回收
                    p = n;
                    n = p.next;
                    p.recycleUnchecked();
                } while (n != null);
            }
        }
    }
```

这么看来这个方法名字起的还挺靠谱的，很好的解释了是要删除还没有被处理的消息。

### 消息入队管理enqueueMessage()方法



```csharp
    // 消息入队
    // 参数when就是此消息应该被处理的时间
    boolean enqueueMessage(Message msg, long when) {
        // 如果此消息的target也就是宿主handler是空的抛异常
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }

        // 如果此消息是in-use状态抛异常，in-use的消息不可拿来使用
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        //上锁
        synchronized (this) {

            // 如果当前MessageQueue已经退出了抛异常并释放掉此消息
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }
            
            // 将消息标记为in-use状态
            msg.markInUse();
            // 设置应该被处理的时间
            msg.when = when;
            // 拿到队列头
            Message p = mMessages;


            // 是否需要唤醒线程
            boolean needWake;

            // p等于空说明队列是空的
            // when等于0表示强制把此消息插入队列头部，最先处理
            // when小于队列头的when说明此消息应该被处理的时间比队列中第一个要处理的时间还早
            // 以上情况满足任意一种直接将消息插入队列头部
            if (p == null || when == 0 || when < p.when) {
                // 将消息插入队列头部
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                
                // 线程已经被阻塞&&消息存在宿主Handler&&消息是异步的
                needWake = mBlocked && p.target == null && msg.isAsynchronous();

            
                //如果上述条件都不满足就要按照消息应该被处理的时间插入队列中    
                Message prev;
                for (;;) {
                    // 两根相邻的引用一前一后从队列头开始依次向后移动
                    prev = p;
                    p = p.next;
                    // 如果队列到尾部了或者找到了处理时间早于自身的消息就结束循环
                    if (p == null || when < p.when) {
                        break;
                    }

                    // 如果入队的消息是异步的而排在它前面的消息有异步的就不需要唤醒
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                // 将新消息插在这一前一后两个引用中间，完成入队
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // 判断是否需要唤醒线程
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```

总结一下消息入队的逻辑大致分为如下几步：

1. 检查消息合法性，包括宿主`target`是否为空，是否为in-use状态，队列是否还存活。
2. 如果满足条件【队列为空、when等于0、此消息应被处理的时间比队列中第一个要处理的时间还早】中的任意一个直接将此消息插在队列头部最先被处理。
3. 如果以上三个条件均不满足，那么就从头遍历队列根据被处理时间找到它的位置。

### 同步消息拦截器

除了`enqueueMessage()`方法可以向队列中添加消息外，还有一个`postSyncBarrier()`方法也可以向队列添加消息，但它不是添加普通的消息，我们将它添加的特殊Message称为**同步消息拦截器**。

顾名思义，该拦截器只会影响同步消息。复习一下上节中分析到的东西，我们默认发送的消息都是同步的，只有某个Message被调用了`setAsynchronous(true)`后才是异步消息。同步消息受队列限制依次有序的等待处理，异步消息也不受限制。

消息拦截器与普通消息的差异在于拦截器的`target`是空的，正常我们通过`enqueueMessage()`方法入队的消息由于限制`target`是不能为空的。



```csharp
    // 标识拦截器的token
    private int mNextBarrierToken;

    private int postSyncBarrier(long when) {

        synchronized (this) {
            // 得到拦截器token
            final int token = mNextBarrierToken++;
            // 实例化一个消息对象
            final Message msg = Message.obtain();
            // 将对象设置为in-use状态
            msg.markInUse();
            // 设置时间
            msg.when = when;
            // 将token存于消息的常用属性arg1中
            msg.arg1 = token;

            Message prev = null;
            Message p = mMessages;

            // 如果when不等于0就在队列中按时间找到它的位置
            if (when != 0) {
                while (p != null && p.when <= when) {
                    prev = p;
                    p = p.next;
                }
            }
            // 如果prev不等于空就把拦截器插入
            // 如果prev等于空直接插入队列头部
            if (prev != null) { // invariant: p == prev.next
                msg.next = p;
                prev.next = msg;
            } else {
                msg.next = p;
                mMessages = msg;
            }
          
            // 拦截器入队成功，返回对应token
            return token;
        }
    }
```

总体来说添加拦截器的方法跟正常消息入队差不多，值得一提的就是Message的`target`是空的，然后`arg1`保存着拦截器的唯一标识`token`。

`token`的作用是找到对应的拦截器删除，看看删除拦截器的方法。



```java
    public void removeSyncBarrier(int token) {
        synchronized (this) {
            Message prev = null;
            Message p = mMessages;
            // 遍历队列找到指定拦截器
            // 查找条件：target为空，arg1等于指定token值
            while (p != null && (p.target != null || p.arg1 != token)) {
                prev = p;
                p = p.next;
            }
            // 如果p等于空说明没找到
            if (p == null) {
                throw new IllegalStateException("The specified message queue synchronization "
                        + " barrier token has not been posted or has already been removed.");
            }

            // 是否需要唤醒线程
            final boolean needWake;

            // 在队列中移除掉拦截器
            if (prev != null) {
                prev.next = p.next;
                // 如果prev不等于空说明拦截器前面还有别的消息，就不需要唤醒
                needWake = false;
            } else {
                mMessages = p.next;
                // 拦截器在队列头部，移除它之后如果队列空了或者它的下一个消息是个正常消息就需要唤醒
                needWake = mMessages == null || mMessages.target != null;
            }

            // 回收
            p.recycleUnchecked();

            // 判断是否需要唤醒
            if (needWake && !mQuitting) {
                nativeWake(mPtr);
            }
        }
    }
```

### 队列空闲处理器IdleHandler

由于在从队列中取出消息时队里可能是空的，这时候就会阻塞线程等待消息到来。每次队列中没有消息而进入的阻塞状态，我们叫它为“空闲状态”。

讲道理实际使用中队列空闲状态的情况还是很常见的，为了更好的利用资源，也为了更好的掌握线程的状态，开发人员就设计了这么一个“队列空闲处理器IdleHandler”。

`IdleHandler`是MessageQueue类下的一个子接口，只包含了一个方法：



```csharp
    public static interface IdleHandler {
        /**
         * 当线程的MessageQueue等待更多消息时会调用该方法。
         *
         * 返回值：true代表只执行一次，false代表会一直执行它
         */
        boolean queueIdle();
    }
```

MessageQueue为我们提供了添加和删除IdleHandler的方法：



```java
   //使用一个ArrayList存储IdleHandler
   private final ArrayList<IdleHandler> mIdleHandlers = new ArrayList<IdleHandler>();

   // 添加一个IdleHandler
   public void addIdleHandler(@NonNull IdleHandler handler) {
        if (handler == null) {
            throw new NullPointerException("Can't add a null IdleHandler");
        }
        synchronized (this) {
            mIdleHandlers.add(handler);
        }
    }

    // 删除一个IdleHandler
    public void removeIdleHandler(@NonNull IdleHandler handler) {
        synchronized (this) {
            mIdleHandlers.remove(handler);
        }
    }
```

### 消息出队管理next()方法

`next()`方法很长，先大致看一下源码：



```java
    private IdleHandler[] mPendingIdleHandlers;

    Message next() {

        // mPtr是从native方法中得到的NativeMessageQueue地址
       // 如果mPtr等于0说明队列不存在或被清除掉了
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        // 待处理的IdleHandler数量，因为代表数量，所以只有第一次初始化时为-1
        int pendingIdleHandlerCount = -1;


        // 线程将被阻塞的时间
        // -1：一直阻塞
        // 0：不阻塞
        // >0:阻塞nextPollTimeoutMillis 毫秒
        int nextPollTimeoutMillis = 0;


        // 开始死循环，下面的代码都是在循环中，贼长！
        for (;;) {

            // 如果nextPollTimeoutMillis 不等于0说明要阻塞线程了
            if (nextPollTimeoutMillis != 0) {
                // 为即将长时间阻塞做准备把该释放的对象都释放了
                Binder.flushPendingCommands();
            }

            // 阻塞线程操作
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // 得到当前时间
                final long now = SystemClock.uptimeMillis();

                Message prevMsg = null;
                Message msg = mMessages;

                // 判断队列头是不是同步拦截器
                if (msg != null && msg.target == null) {
                    // 如果是拦截器就向后找一个异步消息
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
    
                // 判断队列是否有可以取出的消息
                if (msg != null) {

                    if (now < msg.when) {
                        // 如果待取出的消息还没有到应该被处理的时间就让线程阻塞到应该被处理的时间
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // 直接就能取出消息，所以不用阻塞线程
                        mBlocked = false;

                        将消息从队列中剥离出来
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        // 让消息脱离队列
                        msg.next = null;

                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        
                        // 设置为in-use状态
                        msg.markInUse();
                        // 返回取出的消息，结束循环，结束next()方法
                        return msg;
                    }
                } else {
                    // 队列中没有可取出的消息，nextPollTimeoutMillis 等于-1让线程一直阻塞
                    nextPollTimeoutMillis = -1;
                }

                // 如果队列已经退出了直接注销和结束方法
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // IdleHandler初始化为-1，所以在本循环中该条件成立的次数 <= 1
                if (pendingIdleHandlerCount < 0 && (mMessages == null || now < mMessages.when)) {
                    // 得到IdleHandler的数量
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }

                // pendingIdleHandlerCount 小于或等于0说明既没有合适的消息也没有合适的闲时处理
                if (pendingIdleHandlerCount <= 0) {
                    // 直接进入下次循环阻塞线程
                    mBlocked = true;
                    continue;
                }

                // 代码执行到此处就说明线程中有待处理的IdleHandler
                // 那么就从IdleHandler集合列表中取出待处理的IdleHandler
                if (mPendingIdleHandlers == null) {
                    // 初始化待处理IdleHandler数组，最小长度为4
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
    
                // 从IdleHandler集合中获取待处理的IdleHandler
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }
          
            // ==========到此处同步代码块已经结束==========


            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                // 取出一个IdleHandler
                final IdleHandler idler = mPendingIdleHandlers[i];
                // 释放掉引用
                mPendingIdleHandlers[i] = null; 

                // IdleHandler的执行模式，true=执行一次，false=总是执行
                boolean keep = false;

                try {
                    // 执行IdleHandler的queueIdle()代码，得到执行模式
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                // 通过执行模式判断是否需要移除掉对应的IdleHandler
                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // 处理完了所有IdleHandler把数量清0
            pendingIdleHandlerCount = 0;

            // 因为执行了IdleHandler的代码块，有可能已经有新的消息入队了
            // 所以到这里就不阻塞线程，直接去查看有没有新消息
            nextPollTimeoutMillis = 0;
        }
    }
```

消息出队的核心代码的逻辑都在一个庞大的死循环`for(;;)`中，其流程如下：

0，循环开始。

1，根据nextPollTimeoutMillis值阻塞线程，初始值为0：不阻塞线程。

2，将【待取出消息指针】指向队列头。

3，如果队列头是同步拦截器的话就将【待取出消息指针】指向队列头后面最近的一个异步消息。

4，如果【待取出消息指针】不可用（msg == null）说明队列中没有可取出的消息，让nextPollTimeoutMillis 等于-1让线程一直阻塞，等待新消息到来时唤醒它。

5，如果【待取出消息指针】可用（msg != null）再判断一下消息的待处理时间。

- 如果消息的待处理时间大于当前时间（now < msg.when）说明当前消息还没到要处理的时间，让线程阻塞到消息待处理的指定时间。
- 如果消息的待处理时间小于当前时间（now > msg.when）就直接从队列中取出消息返回给调用处。（此处会直接结束整个循环，结束next()方法。）

6，如果队列已经退出了直接结束next()方法。

7，如果是第一次循环就初始化IdleHandler数量的局部变量pendingIdleHandlerCount 。

8，如果IdleHandler数量小于等于0说明没有合适的IdleHandler，直接进入下次循环阻塞线程。（此处会直接结束本次循环。）

9，初始化IdleHandler数组，里面保存着本地待处理的IdleHandler。

10，遍历IdleHandler数组，执行对应的queueIdle()方法。

11，执行完所有IdleHandler之后，将IdleHandler数量清0。

12，因为执行了IdleHandler的代码块，有可能已经有新的消息入队了， 所以让nextPollTimeoutMillis 等于0不阻塞线程，直接去查看有没有新消息。

13，本次循环结束，开始新一轮循环。

### 总结

1. MessageQueue队列消息是有序的，按消息待处理时间依次排序。
2. 同步拦截器可以拦截它之后的所有同步消息，直到这个拦截器被移除。
3. 取出消息时如果没有合适的消息线程会阻塞

## Handler机制实现原理（三）Looper的源码分析

> 刚看源码的时候：“这TM写的是啥？那写的又TM是啥？”
> 研究明白了之后：“奥，原来就这点玩意儿啊，太简单了。”

Looper的职责很单一，就是单纯的从MessageQueue中取出消息分发给消息对应的宿主Handler，因此它的代码不多（300行左右）。

Looper是线程独立的且每个线程只能存在一个Looper。

Looper会根据自己的存活情况来创建和退出属于它自己的MessageQueue。

### 创建与退出Looper

上面的结论中提到了Looper是线程独立的且每个线程只能存在一个Looper。所以构造Looper实例的方法类似于单例模式。隐藏构造方法,对外提供了两个指定的获取实例方法`prepare()`和`prepareMainLooper()`。



```java
    // 应用主线程（UI线程）Looper实例
    private static Looper sMainLooper;

    // Worker线程Looper实例，用ThreadLocal保存的对象都是线程独立的
    static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

    // 与当前Looper对应的消息队列
    final MessageQueue mQueue;

    // 当前Looper所以的线程
    final Thread mThread;

    /**
     * 对外公开初始化方法
     *
     * 在普通线程中初始化Looper调用此方法
     */
    public static void prepare() {
        // 初始化一个可以退出的Looper
        prepare(true);
    }

    /**
     * 对外公开初始化方法
     *
     * 在应用主线程（UI线程）中初始化Looper调用此方法
     */
    public static void prepareMainLooper() {
        
        // 因为是主线程，初始化一个不允许退出的Looper
        prepare(false);

        synchronized (Looper.class) {
            // 如果sMainLooper不等于空说明已经创建过主线程Looper了，不应该重复创建
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }

    /**
     * 内部私有初始化方法
     * @param quitAllowed 是否允许退出Looper
     */
    private static void prepare(boolean quitAllowed) {
        // 每个线程只能有一个Looper
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        // 保存实例
        sThreadLocal.set(new Looper(quitAllowed));
    }

    /**
     * 私有构造方法
     * @param quitAllowed 是否允许退出Looper
     */  
    private Looper(boolean quitAllowed) {
        // 初始化MessageQueue
        mQueue = new MessageQueue(quitAllowed);
        // 得到当前线程实例
        mThread = Thread.currentThread();
    }
```

真正创建Looper实例的构造方法中其实很简单，就是创建了对应的MessageQueue实例，然后得到当前线程，值得注意的是MessageQueue和线程实例都是被`final`关键字修饰的，只能被赋值一次。

对外公开初始化方法`prepareMainLooper()`是为应用主线程（UI线程）准备的，应用刚被创建就会调用该方法，所以我们不该再去调用它。

开发者可以通过调用对外公开初始化方法`prepare()`对自己的worker线程创建Looper，但是要注意只能初始化一次。

调用`Looper.prepare()`方法初始化完成后，可以调用`myLooper()`和`myQueue()`方法得到当前线程对应的实例。



```java
    public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }

    public static @NonNull MessageQueue myQueue() {
        return myLooper().mQueue;
    }
```

**退出Looper**

退出Looper有安全与不安全两种退出方法，其实对应的就是MessageQueue的安全与不安全方法：



```cpp
    public void quit() {
        mQueue.quit(false);
    }

    public void quitSafely() {
        mQueue.quit(true);
    }
```

什么安全退出，什么是不安全退出，在MessageQueue源码中分析过。

### 运行Looper处理消息

调用`Looper.prepare()`方法初始化完成Looper后就可以让Looper去工作了，只需要调用`Looper.loop()`方法即可。



```java
    public static void loop() {
        // 得到当前线程下的Looper
        final Looper me = myLooper();

        // 如果还没初始化过抛异常
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }

         // 得到当前线程下与Looper对应的消息队列
        final MessageQueue queue = me.mQueue;

        // 得到当前线程的唯一标识（uid+pid），作用是下面每次循环都判断一下线程有没有被切换
        // 不知道为什么要调用两次该方法
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        // 进入死循环不断取出消息
        for (;;) {

            // 从队列中取出一个消息，这可能会阻塞线程
            Message msg = queue.next(); 

            // 如果消息是空的，说明队列已经退出了，直接结束循环，结束方法
            if (msg == null) {
                return;
            }

            // 打印日志
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            // 性能分析相关的东西
            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }

            try {
                //尝试将消息分发给宿主（Handler）
                //dispatchMessage为宿主Handler的接收消息方法
                msg.target.dispatchMessage(msg);
            } finally {
                 // 性能分析相关的东西
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }

            //打印日志
            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }


            //得到当前线程的唯一标识
            final long newIdent = Binder.clearCallingIdentity();

            //如果本次循环所在的线程与最开始不一样，打印日志记录
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }
      
            //消息分发完毕，回收消息到缓存池
            msg.recycleUnchecked();
        }
    }
```

### 总结

Looper的功能很简单，核心方法`Looper.loop()`就是不断的从消息队列中取出消息分发给对应的宿主Handler，它与对应MessageQueue息息相关，一起创建，一起退出。

Looper更想强调的是线程的独立性与唯一性，利用`ThreadLocal`保证每个线程只有一个Looper实例的存在。利用静态构造实例方法保证不能重复创建Looper。

`Looper.prepareMainLooper()`是比较特殊的方法，它是给UI线程准备，理论上开发者在任何情况下都不应该调用它。

## Handler机制实现原理（四）handler的源码分析

Handler本身可在多线程之间调用，不管它在哪个线程发送消息，都会回到它被初始化的哪个线程中接收到消息。

### 初始化

Handler有**7个**构造方法，分别对应不同的参数来初始化不同的Handler属性，但是真正完成初始化操作的只有两个构造方法：



```java
    // 是否需要查找潜在的漏洞
    private static final boolean FIND_POTENTIAL_LEAKS = false;

    /**
     * 将Callback接口作为构造方法参数，可以用作接收消息的回调
     * 这样就可以省去自己重写Handler自身的handleMessage方法
     *
     * @param msg 接收到的消息
     * @return 是否需要进一步处理，即调用Handler自身的handleMessage方法
     */
    public interface Callback {
        public boolean handleMessage(Message msg);
    }

    final Looper mLooper;
    final MessageQueue mQueue;
    final Callback mCallback;

    // 发送的消息是否为异步的，默认是false
    final boolean mAsynchronous;

    /**
     * @hide 隐藏的构造方法，外部不可见
     */
    public Handler(Callback callback, boolean async) {

        // 检查是否存在内存泄漏的可能
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        // 得到当前线程的Looper
        mLooper = Looper.myLooper();

        // 如果Looper还没初始化抛出异常
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }

        // 得到当前线程的MessageQueue
        mQueue = mLooper.mQueue;

        
        mCallback = callback;
        mAsynchronous = async;
    }

 
    public Handler(Looper looper, Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```

从上面代码的得知，构造方法初始化的工作就是给`mLooper`,`mQueue`,`mCallback`和`mAsynchronous`这几个关键的属性赋值.`mLooper`和`mQueue`自然就是当前线程下的Looper和MessageQueue了，如果传递了Looper参数就直接赋值，如果没传递就调用`Looper.myLooper();`得到当前线程的Looper。

`mCallback`是Handler内部定义的一个简单接口，其目的是为了**替代传统的接收消息方法**。当然使用`mCallback`的同时并不会影响正常的Handler消息分发。此处解释从后面接收消息时的逻辑就可以看到。

`mAsynchronous`的意思是**该Handler发送的消息是否是异步的**，从前面Message源码的文章中我们知道Message中有一个设置消息是否为异步消息的方法，MessageQueue对异步消息的处理也与同步消息不同。此处如果设置了`mAsynchronous`为`true`，那么这个Handler发送的所有消息就都是异步消息。

在有两个参数的构造方法中我们会发现有一段检查是否存在内存泄漏的代码，为什么会这样呢？在分析完发送消息和接收消息后再说这个。

当然，除了上面两个构造方法外还有其它几个构造方法，但均是调用上面两个方法：



```csharp
   public Handler() {
        this(null, false);
    }

    public Handler(Callback callback) {
        this(callback, false);
    }

    public Handler(Looper looper) {
        this(looper, null, false);
    }

    public Handler(Looper looper, Callback callback) {
        this(looper, callback, false);
    }

    /**
     * @hide 隐藏的构造方法，外部不可见
     */
    public Handler(boolean async) {
        this(null, async);
    }
```

### 发送消息

使用Handler发送消息时我们知道它分为两类：

- `postXXX()`方法切换回原线程。
- `sendMessageXXX()`方法发送消息到原线程。

其实这两种方法本质都是发送一个Message对象到原线程，只不过`PostXXX()`方法是发送了一个只有`Runnable callback`属性的Message对象。

先来看一下`sendMessageXXX()`类的方法：



```java
    // 发送一条普通消息
    public final boolean sendMessage(Message msg){
        return sendMessageDelayed(msg, 0);
    }

    // 发送一条空消息
    public final boolean sendEmptyMessage(int what){
        return sendEmptyMessageDelayed(what, 0);
    }

    // 发送一条空的延时消息
    public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
        Message msg = Message.obtain();
        msg.what = what;
        return sendMessageDelayed(msg, delayMillis);
    }

    // 发送一条空的定时消息
    public final boolean sendEmptyMessageAtTime(int what, long uptimeMillis) {
        Message msg = Message.obtain();
        msg.what = what;
        return sendMessageAtTime(msg, uptimeMillis);
    }

    // 发送一个普通的延时消息
    public final boolean sendMessageDelayed(Message msg, long delayMillis) {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

    // 发送一个普通的定时消息
    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        //得到消息队列
        MessageQueue queue = mQueue;
        // 如果消息队列是空的记录日志然后结束方法
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        // 执行消息入队操作
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```

从上面的代码中发现，不管调用何种发送消息的方法，最后真正调用的都是`sendMessageAtTime()`方法。而真正发送的核心方法也就是入队方法是Handler的`enqueueMessage()`方法。



```cpp
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        // 将消息的宿主设置为当前Handler自身
        msg.target = this;

        //如果Handler被设置成了异步就把消息也设置成异步的
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }

        // 执行消息队列的入队操作
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

看到这里终于明白了为啥Looper和MessageQueue一直在使用Message的`target`属性而我们却从来没有给它赋值过，是Handler在发送消息前自己赋值上去的。

看完了发送消息类的方法在看看切换线程类的方法干了什么：



```java

    private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }

    private static Message getPostMessage(Runnable r, Object token) {
        Message m = Message.obtain();
        m.obj = token;
        m.callback = r;
        return m;
    }

    public final boolean post(Runnable r){
       return  sendMessageDelayed(getPostMessage(r), 0);
    }
    
    public final boolean postAtTime(Runnable r, long uptimeMillis) {
        return sendMessageAtTime(getPostMessage(r), uptimeMillis);
    }
    
    public final boolean postAtTime(Runnable r, Object token, long uptimeMillis){
        return sendMessageAtTime(getPostMessage(r, token), uptimeMillis);
    }
    
    public final boolean postDelayed(Runnable r, long delayMillis){
        return sendMessageDelayed(getPostMessage(r), delayMillis);
    }
    
    public final boolean postAtFrontOfQueue(Runnable r){
        return sendMessageAtFrontOfQueue(getPostMessage(r));
    }
```

上本节开头讲的一样，`postXXX()`类方法就是构造了一个只有`Runnable callback`的Message对象，然后走正常发送消息的方法。唯一有一个特例就是`postAtFrontOfQueue()`方法，它调用了`sendMessageAtFrontOfQueue()`方法是之前发送消息没有用到过的：



```cpp
    public final boolean sendMessageAtFrontOfQueue(Message msg) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, 0);
    }
```

原来这个方法特殊的地方就是在入队的时候时间参数为0，我们在MessageQueue源码知道如果入队消息的时间参数为0那么这个消息会被直接放在队列头。所以，`postAtFrontOfQueue()`方法就是直接在消息队列头部插入了一个消息。

### 接收消息

在Looper 的源码中我们知道每当从MessageQueue中取出一个消息时就会调用这个消息的宿主`target`中分发消息的方法：



```cpp
// Looper分发消息
msg.target.dispatchMessage(msg);
```

而这个宿主`target`也就是我们的Handler，所有Handler接收消息就是在这个`dispatchMessage()`方法中了：



```csharp
    public void dispatchMessage(Message msg) {
        // 如果Message的callback不为空，说明它是一个通过postXXX()方法发送的消息
        if (msg.callback != null) {
            // 直接运行这个callback
            handleCallback(msg);
        } else {
            //如果mCallback 不为空说明Handler设置了Callback接口
            // 先执行接口处理消息的方法
            if (mCallback != null) {
                // 如果callback接口处理完消息返回true说明它将消息拦截
                // 不再执行Handler自身的处理消息方法，直接结束方法
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            // 调用Handler自身处理消息的方法
            handleMessage(msg);
        }
    }

    private static void handleCallback(Message message) {
        // 运行callback，也就是这个Runnable接口
        message.callback.run();
    }

    // Handler自身处理消息的方法，开发者需要重新该方法来实现接收消息
    public void handleMessage(Message msg) {
    }
```

由此可见，如果是单纯的`PostXXX()`方法发送的消息，Handler接收到了之后直接运行Message对象的Runnable接口，不会将它当做一个消息进行处理。

而我们的`mCallback`接口是完全可以替代Handler自身接收消息的方法，因为其高优先处理等级，它甚至可以选择拦截掉Handler自身的接收消息方法。

### 内存泄漏的可能

我们在使用Handler的时候写法一般如下：



```java
    private final Handler handler = new Handler(){

        @Override
        public void handleMessage(Message msg) {

        }
    };
```

由于重写了`handleMessage()`方法相当于生成了一个匿名内部类，也就相当于如下代码：



```java
    private final Handler handler = new MyHandler ();

    class MyHandler extends Handler{

        @Override
        public void handleMessage(Message msg) {
            
        }
    }
```

可是你有没有想过内部类凭什么能够调用外部类的属性和方法呢？答案就是内部类隐式的持有着外部类的引用，编译器在创建内部类时把外部类的引用传入了其中，只不过是你看不到而已。

既然Handler作为内部类持有着外部类（多数情况为Activity）的引用，而Handler对应的一般都是耗时操作。当我们在子线程执行一项耗时操作时，用户退出程序，Activity需要被销毁，而Handler还在持有Activity的引用导致无法回收，就会引发内存泄漏。

**解决方法分为两步**

1. 生成内部类时把内部类声明为静态的。
2. 使用弱引用来持有外部类引用。

静态内部类不会持有外部类的引用，且弱引用不会阻止JVM回收对象。



```java
    static class MyHandler extends Handler{

        WeakReference<Activity> mActivity ;

        public MyHandler(Activity activity){
            mActivity = new WeakReference<Activity>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            Activity activity = mActivity.get();

            if (activity == null){
                return;
            }

            // do something

        }
    }
```

所以，在文章刚开始初始化方法中检查漏洞的代码其实就是检查这个内存泄漏的可能性：



```dart
        // 检查是否存在内存泄漏的可能
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            // 是否为匿名类，内部类以及是否为静态类
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
```

大功告成，完美！

## Handler机制实现原理（五）总结

### Message缓存池

Android 的工程师们充分利用了Java的高级语言特性，即类中持有着一个类自身的属性作为经典数据结构中的链表`next`指针，以静态属性属于类本身的特性实现了链表的表头。这种模式给我了很大的启发，让我这种渣渣每逢想起都会惊讶“还有这种操作？”。

**为什么要有缓存池**

了解完Handler整体机制后我猜测，Message功能十分单一且状态很少，它只是一个具体发送消息的载体，但是使用数量十分庞大，回收用过的Message不仅可以有效的减少重复消耗系统资源且回收它的成本很低，所以何乐而不为呢？

**谁负责回收Message**

我们使用Message时候知道调用`Message.obtain();`方法可以从缓存池中取出一个Message，有存才能有取，我们什么时候回收它呢？从源码中发现，Looper在分发Message给宿主Handler之后，确定了Message已经完成了它的使命直接就会将它回收。所以我们完全不用担心这个，我们发送的每个消息最后都会被回收。

### 真正的阻塞发生在MessageQueue

MessageQueue维持的消息队列也是靠跟Message缓存池同样的原理生成的，每次消息出队时如果没有合适的待取出消息就会阻塞线程等待有合适的消息。

非常奇怪的是，MessageQueue线程的方式不是传统使用java实现的，而是通过JNI调用native层的C++代码实现的，C++代码中也实现了一套Looper+MessageQueue+Handler，阻塞线程的方式是调用Linux的监听文件描述符ePoll实现的。

我的猜测是因为Java代码需要经过JVM的帮助才能跟系统接触，这一过程会消耗性能，而C++代码则直接可以绕过这一个环节。所以，使用C++代码实现线程阻塞可能是性能上的需求。

### 为什么推荐使用Handler实现线程间通信

在没有真正了解Handler的时候以为Google的工程师们在Handler上使用了什么了不起的技术呢，所以才推荐开发者们使用Handler来实现线程间通信。

其实呢？Android是事件型驱动的系统，刚创建一个应用程序的主线程里就会被创建一个Looper来不断接受各种事件，所以说如果我们打开一个程序什么都不操作，这个程序就有可能是阻塞状态的，因为他没有任何事件需要去处理。反之，我们在自己的UI线程里执行一项耗时操作，主线程Looper一直在处理这个任务而无法分身处理其它的事件这时候就有可能ANR了。

所以，不是Handler的技术多牛逼，是主线程用了Handler来通信，你是用别的方法通信有可能会影响主线程Looper的正常工作。



# Binder

## Binder原理（一）学习Binder前必须要了解的知识点

Binder原理是掌握系统底层原理的基石，也是进阶高级工程师的必备知识点，这篇文章不会过多介绍Binder原理，而是讲解学习Binder前需要的掌握的知识点。

### Linux和Android的IPC机制种类

IPC全名为inter-Process Communication，含义为进程间通信，是指两个进程之间进行数据交换的过程。在Android和Linux中都有各自的IPC机制，这里分别来介绍下。

#### Linux中的IPC机制种类

Linux中提供了很多进程间通信机制，主要有管道（pipe）、信号（sinal）、信号量（semophore）、消息队列（Message）、共享内存（Share Memory)、套接字（Socket）等。

**管道**
管道是Linux由Unix那里继承过来的进程间的通信机制，它是Unix早期的一个重要通信机制。管道的主要思想是，在内存中创建一个共享文件，从而使通信双方利用这个共享文件来传递信息。这个共享文件比较特殊，它不属于文件系统并且只存在于内存中。另外还有一点，管道采用的是半双工通信方式的，数据只能在一个方向上流动。
简单的模型如下所示。
![nbXJ2T.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMC8yNC8xNmRmOTZhOWE3YzEwMTM5?x-oss-process=image/format,png)

**信号**
信号是软件层次上对中断机制的一种模拟，是一种异步通信方式，进程不必通过任何操作来等待信号的到达。信号可以在用户空间进程和内核之间直接交互，内核可以利用信号来通知用户空间的进程发生了哪些系统事件。信号不适用于信息交换，比较适用于进程中断控制。
**信号量**
信号量是一个计数器，用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。主要作为进程间以及同一进程内不同线程之间的同步手段。
**消息队列**
消息队列是消息的链表，具有特定的格式，存放在内存中并由消息队列标识符标识，并且允许一个或多个进程向它写入与读取消息。信息会复制两次，因此对于频繁或者信息量大的通信不宜使用消息队列。

**共享内存**
多个进程可以直接读写的一块内存空间，是针对其他通信机制运行效率较低而设计的。
为了在多个进程间交换信息，内核专门留出了一块内存区，可以由需要访问的进程将其映射到自己的私有地址空间。进程就可以直接读写这一块内存而不需要进行数据的拷贝，从而大大的提高效率。

**套接字**
套接字是更为基础的进程间通信机制，与其他方式不同的是，套接字可用于不同机器之间的进程间通信。

#### Android中的IPC机制

Android系统是基于Linux内核的，在Linux内核基础上，又拓展出了一些IPC机制。Android系统除了支持套接字，还支持序列化、Messenger、AIDL、Bundle、文件共享、ContentProvider、Binder等。Binder会在后面介绍，先来了解前面的IPC机制。
**序列化**
序列化指的是Serializable/Parcelable，Serializable是Java提供的一个序列化接口，是一个空接口，为对象提供标准的序列化和反序列化操作。Parcelable接口是Android中的序列化方式，更适合在Android平台上使用，用起来比较麻烦，效率很高。
**Messenger**
Messenger在Android应用开发中的使用频率不高，可以在不同进程中传递Message对象，在Message中加入我们想要传的数据就可以在进程间的进行数据传递了。Messenger是一种轻量级的IPC方案并对AIDL进行了封装。

**AIDL**
AIDL全名为Android interface definition Language，即Android接口定义语言。Messenger是以串行的方式来处理客户端发来的信息，如果有大量的消息发到服务端，服务端仍然一个一个的处理再响应客户端显然是不合适的。另外还有一点，Messenger用来进程间进行数据传递但是却不能满足跨进程的方法调用，这个时候就需要使用AIDL了。

**Bundle**
Bundle实现了Parcelable接口，所以它可以方便的在不同的进程间传输。Acitivity、Service、Receiver都是在Intent中通过Bundle来进行数据传递。

**文件共享**
两个进程通过读写同一个文件来进行数据共享，共享的文件可以是文本、XML、JOSN。文件共享适用于对数据同步要求不高的进程间通信。

**ContentProvider**
ContentProvider为存储和获取数据了提供统一的接口，它可以在不同的应用程序之间共享数据，本身就是适合进程间通信的。ContentProvider底层实现也是Binder，但是使用起来比AIDL要容易许多。系统中很多操作都采用了ContentProvider，例如通讯录，音视频等，这些操作本身就是跨进程进行通信。

### Linux和Binder的IPC通信原理

在讲到Linux的进程通信原理之前，我们需要先了解Liunx中的几个概念。

![njr0qU.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMC8yNC8xNmRmOTZhOTkzODU1MzY2?x-oss-process=image/format,png)

**内核空间和用户空间**
当我们接触到Liunx时，免不了听到两个词，User space（用户空间）和 Kernel space（内核空间），那么它们的含义是什么呢？
为了保护用户进程不能直接操作内核，保证内核的安全，操作系统从逻辑上将虚拟空间划分为用户空间和内核空间。Linux 操作系统将最高的1GB字节供内核使用，称为内核空间，较低的3GB 字节供各进程使用，称为用户空间。

内核空间是Linux内核的运行空间，用户空间是用户程序的运行空间。为了安全，它们是隔离的，即使用户的程序崩溃了，内核也不会受到影响。内核空间的数据是可以进程间共享的，而用户空间则不可以。比如在上图进程A的用户空间是不能和进程B的用户空间共享的。

**进程隔离**
进程隔离指的是，一个进程不能直接操作或者访问另一个进程。也就是进程A不可以直接访问进程B的数据。

**系统调用**
用户空间需要访问内核空间，就需要借助系统调用来实现。系统调用是用户空间访问内核空间的唯一方式，保证了所有的资源访问都是在内核的控制下进行的，避免了用户程序对系统资源的越权访问，提升了系统安全性和稳定性。

进程A和进程B的用户空间可以通过如下系统函数和内核空间进行交互。

- copy_from_user：将用户空间的数据拷贝到内核空间。
- copy_to_user：将内核空间的数据拷贝到用户空间。

**内存映射**
由于应用程序不能直接操作设备硬件地址，所以操作系统提供了一种机制：内存映射，把设备地址映射到进程虚拟内存区。
举个例子，如果用户空间需要读取磁盘的文件，如果不采用内存映射，那么就需要在内核空间建立一个页缓存，页缓存去拷贝磁盘上的文件，然后用户空间拷贝页缓存的文件，这就需要两次拷贝。
采用内存映射，如下图所示。

![nzlnaV.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMC8yNC8xNmRmOTZhOTZjOWFjOGY5?x-oss-process=image/format,png)
由于新建了虚拟内存区域，那么磁盘文件和虚拟内存区域就可以直接映射，少了一次拷贝。

内存映射全名为Memory Map，在Linux中通过系统调用函数mmap来实现内存映射。将用户空间的一块内存区域映射到内核空间。映射关系建立后，用户对这块内存区域的修改可以直接反应到内核空间，反之亦然。内存映射能减少数据拷贝次数，实现用户空间和内核空间的高效互动。

#### Linux的IPC通信原理

了解Liunx中的几个概念后，就可以学习Linux的IPC通信原理了，如下图所示。
![nzaypq.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMC8yNC8xNmRmOTZhOTkzMDVhNTZm?x-oss-process=image/format,png)
内核程序在内核空间分配内存并开辟一块内核缓存区，发送进程通过copy_from_user函数将数据拷贝到到内核空间的缓冲区中。同样的，接收进程在接收数据时在自己的用户空间开辟一块内存缓存区，然后内核程序调用 copy_to_user() 函数将数据从内核缓存区拷贝到接收进程。这样数据发送进程和数据接收进程完成了一次数据传输，也就是一次进程间通信。

Linux的IPC通信原理有两个问题：

1. 一次数据传递需要经历：用户空间 --> 内核缓存区 --> 用户空间，需要2次数据拷贝，这样效率不高。
2. 接收数据的缓存区由数据接收进程提供，但是接收进程并不知道需要多大的空间来存放将要传递过来的数据，因此只能开辟尽可能大的内存空间或者先调用API接收消息头来获取消息体的大小，浪费了空间或者时间。

#### Binder的通信原理

Binder是基于开源的OpenBinder实现的，OpenBinder最早并不是由Google公司开发的，而是Be Inc公司开发的，接着由Palm, Inc.公司负责开发。后来OpenBinder的作者Dianne Hackborn加入了Google公司，并负责Android平台的开发工作，顺便把这项技术也带进了Android。

Binder是基于内存映射来实现的，在前面我们知道内存映射通常是用在有物理介质的文件系统上的，Binder没有物理介质，它使用内存映射是为了跨进程传递数据。
![nzNJUA.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMC8yNC8xNmRmOTZhOTc5NTU4NjEy?x-oss-process=image/format,png)

Binder通信的步骤如下所示。
1.Binder驱动在内核空间创建一个数据接收缓存区。
2.在内核空间开辟一块内核缓存区，建立内核缓存区和数据接收缓存区之间的映射关系，以及数据接收缓存区和接收进程用户空间地址的映射关系。
3.发送方进程通过copy_from_user()函数将数据拷贝 到内核中的内核缓存区，由于内核缓存区和接收进程的用户空间存在内存映射，因此也就相当于把数据发送到了接收进程的用户空间，这样便完成了一次进程间的通信。

整个过程只使用了1次拷贝，不会因为不知道数据的大小而浪费空间或者时间，效率更高。

### 为什么要使用Binder

Android是基于Linux内核的 ，Linux提供了很多IPC机制，而Android却自己设计了Binder来进行通信，主要是因为以下几点。
**性能方面**
性能方面主要影响的因素是拷贝次数，管道、消息队列、Socket的拷贝次书都是两次，性能不是很好，共享内存不需要拷贝，性能最好，Binder的拷贝次书为1次，性能仅次于内存拷贝。
**稳定性方面**
Binder是基于C/S架构的，这个架构通常采用两层结构，在技术上已经很成熟了，稳定性是没有问题的。共享内存没有分层，难以控制，并发同步访问临界资源时，可能还会产生死锁。从稳定性的角度讲，Binder是优于共享内存的。
**安全方面**
Android是一个开源的系统，并且拥有开放性的平台，市场上应用来源很广，因此安全性对于Android 平台而言极其重要。
传统的IPC接收方无法获得对方可靠的进程用户ID/进程ID（UID/PID），无法鉴别对方身份。Android 为每个安装好的APP分配了自己的UID，通过进程的UID来鉴别进程身份。另外，Android系统中的Server端会判断UID/PID是否满足访问权限，而对外只暴露Client端，加强了系统的安全性。
**语言方面**
Linux是基于C语言，C语言是面向过程的，Android应用层和Java Framework是基于Java语言，Java语言是面向对象的。Binder本身符合面向对象的思想，因此作为Android的通信机制更合适不过。

从这四方面来看，Linux提供的大部分IPC机制根本无法和Binder相比较，而共享内存只在性能方面优于Binder，其他方面都劣于Binder，这些就是为什么Android要使用Binder来进行进程间通信，当然系统中并不是所有的进程通信都是采用了Binder，而是根据场景选择最合适的，比如Zygote进程与AMS通信使用的是Socket，Kill Process采用的是信号。

### 为什么要学习Binder?

Binder机制在Android中的地位举足轻重，我们需要掌握的很多原理都和Binder有关：

1. 系统中的各个进程是如何通信的？
2. Android系统启动过程
3. AMS、PMS的原理
4. 四大组件的原理，比如Activity是如何启动的？
5. 插件化原理
6. 系统服务的Client端和Server端是如何通信的？（比如MediaPlayer和MeidaPlayerService)

上面只是列了一小部分，简单来说说，比如系统在启动时，SystemServer进程启动后会创建Binder线程池，目的是通过Binder，使得在SystemServer进程中的服务可以和其他进程进行通信了。再比如我们常说的AMS、PMS都是基于Binder来实现的，拿PMS来说，PMS运行在SystemServer进程，如果它想要和DefaultContainerService通信（是用于检查和复制可移动文件的系统服务），就需要通过Binder，因为DefaultContainerService运行在com.android.defcontainer进程。
还有一个比较常见的C/S架构间通信的问题，Client端的MediaPlayer和Server端的MeidaPlayerService不是运行在一个进程中的，同样需要Binder来实现通信。

可以说Binder机制是掌握系统底层原理的基石。根据Android系统的分层，Binder机制主要分为以下几个部分。

![n5i5PP.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMC8yNC8xNmRmOTZhOTgwOTk0NjVj?x-oss-process=image/format,png)

上图并没有给出Binder机制的具体的细节，而是先给出了一个概念，根据系统的Android系统的分层，我将Binder机制分为了Java Binder、Native Binder、Kernel Binder，实际上Binder的内容非常多，完全可以写一本来介绍，但是对于应用开发来说，并不需要掌握那么多的知识点，因此本系列主要会讲解Java Binder和Native Binder。

## Binder原理（二）ServiceManager中的Binder机制

在上一部分中，我们了解了学习Binder前必须要了解的知识点，其中有一点就是Binder机制的三个部分：Java Binder、Native Binder、Kernel Binder，其中Java Binder和Native Binder都是应用开发需要掌握的。Java Binder是需要借助Native Binder来工作的，因此需要先了解Native Binder，Native Binder架构的原型就是基于Binder通信的C/S架构，因此我们先从它开始入手。源码是基于Android 9.0。

### 基于Binder通信的C/S架构

在Android系统中，Binder进程间的通信的使用是很普遍的，在Android进阶三部曲第一部的最后一章，我讲解了MediaPlayer框架，这个框架基于C/S架构，并采用Binder来进行进程间通信，如下图所示。
![uZgTgJ.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMi5heDF4LmNvbS8yMDE5LzA5LzI1L3VaZ1RnSi5wbmc?x-oss-process=image/format,png)

从图中可以看出，除了常规C/S架构的Client端和Server端，还包括了ServiceManager，它用于管理系统中的服务。
首先Server进程会注册一些Service到ServiceManager中，Client要使用某个Service，则需要先到ServiceManager查询Service的相关信息，然后根据Service的相关信息与Service所在的Server进程建立通信通路，这样Client就可以使用Service了。

### MediaServer的main函数

Client、Server、ServiceManager三者的交互都是基于Binder通信的，那么任意两者的交互都可以说明Binder的通信的原理，可以说Native Binder的原理的核心就是ServiceManager的原理，为了更好的了解ServiceManager，这里拿MediaPlayer框架来举例，它也是学习多媒体时必须要掌握的知识点。

MediaPlayer框架的简单框架图如下所示。
![uMegMj.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMi5heDF4LmNvbS8yMDE5LzA5LzI3L3VNZWdNai5wbmc?x-oss-process=image/format,png)
可以看到，MediaPlayer和MediaPlayerService是通过Binder来进行通信的，MediaPlayer是Client端，MediaPlayerService是Server端，MediaPlayerService是系统多媒体服务的一种，系统多媒体服务是由一个叫做MediaServer的服务进程提供的，它是一个可执行程序，在Android系统启动时，MediaServer也被启动，它的入口函数如下所示。
**frameworks/av/media/mediaserver/main_mediaserver.cpp**

```cpp
int main(int argc __unused, char **argv __unused)
{
    signal(SIGPIPE, SIG_IGN);
    //获取ProcessState实例
    sp<ProcessState> proc(ProcessState::self());//1
    sp<IServiceManager> sm(defaultServiceManager());//2
    ALOGI("ServiceManager: %p", sm.get());
    InitializeIcuOrDie();
    //注册MediaPlayerService
    MediaPlayerService::instantiate();//3
    ResourceManagerService::instantiate();
    registerExtensions();
    //启动Binder线程池
    ProcessState::self()->startThreadPool();
    //当前线程加入到线程池
    IPCThreadState::self()->joinThreadPool();
}
```

注释1处用于获取ProcessState实例，在这一过程中会打开/dev/binder设备，并使用mmap为Binder驱动分配一个虚拟地址空间用来接收数据。
注释2处用来得到一个IServiceManager，通过这个IServiceManager，其他进程就可以和当前的ServiceManager进行交互，这里就用到了Binder通信。
注释3处用来注册MediaPlayerService。
除了注释3处的知识点在下一篇文章进行介绍，注释1和注释2处的内容，本篇文章会分别来进行介绍，先看ProcessState实例。

### 每个进程唯一的ProcessState

ProcessState从名称就可以看出来，用于代表进程的状态，先来查看上一小节的ProcessState的self函数。
**frameworks/native/libs/binder/ProcessState.cpp**

```cpp
sp<ProcessState> ProcessState::self()
{
    Mutex::Autolock _l(gProcessMutex);
    if (gProcess != NULL) {
        return gProcess;
    }
    gProcess = new ProcessState("/dev/binder");//1
    return gProcess;
}
```

这里采用了单例模式，确保每个进程只有一个ProcessState实例。注释1处用于创建一个ProcessState实例，参数为/dev/binder。接着来查看ProcessState的构造函数，代码如下所示。
**frameworks/native/libs/binder/ProcessState.cpp**

```cpp
ProcessState::ProcessState(const char *driver)
    : mDriverName(String8(driver))
    , mDriverFD(open_driver(driver))//1
    , mVMStart(MAP_FAILED)
    , mThreadCountLock(PTHREAD_MUTEX_INITIALIZER)
    , mThreadCountDecrement(PTHREAD_COND_INITIALIZER)
    , mExecutingThreadsCount(0)
    , mMaxThreads(DEFAULT_MAX_BINDER_THREADS)
    , mStarvationStartTimeMs(0)
    , mManagesContexts(false)
    , mBinderContextCheckFunc(NULL)
    , mBinderContextUserData(NULL)
    , mThreadPoolStarted(false)
    , mThreadPoolSeq(1)
{
    if (mDriverFD >= 0) {
        mVMStart = mmap(0, BINDER_VM_SIZE, PROT_READ, MAP_PRIVATE | MAP_NORESERVE, mDriverFD, 0);//2
        if (mVMStart == MAP_FAILED) {
            // *sigh*
            ALOGE("Using %s failed: unable to mmap transaction memory.\n", mDriverName.c_str());
            close(mDriverFD);
            mDriverFD = -1;
            mDriverName.clear();
        }
    }
    LOG_ALWAYS_FATAL_IF(mDriverFD < 0, "Binder driver could not be opened.  Terminating.");
}
```

ProcessState的构造函数中调用了很多函数，需要注意的是注释1处，它用来打开/dev/binder设备。
注释2处的mmap函数，它会在内核虚拟地址空间中申请一块与用户虚拟内存相同大小的内存，然后再申请物理内存，将同一块物理内存分别映射到内核虚拟地址空间和用户虚拟内存空间，实现了内核虚拟地址空间和用户虚拟内存空间的数据同步操作，也就是内存映射。
mmap函数用于对Binder设备进行内存映射，除了它还有open、ioctl函数，来看看它们做了什么。
注释1处的open_driver函数的代码如下所示。
**frameworks/native/libs/binder/ProcessState.cpp**

```cpp
static int open_driver(const char *driver)
{
    int fd = open(driver, O_RDWR | O_CLOEXEC);//1
    if (fd >= 0) {
        ...
        size_t maxThreads = DEFAULT_MAX_BINDER_THREADS;
        result = ioctl(fd, BINDER_SET_MAX_THREADS, &maxThreads);//2
        if (result == -1) {
            ALOGE("Binder ioctl to set max threads failed: %s", strerror(errno));
        }
    } else {
        ALOGW("Opening '%s' failed: %s\n", driver, strerror(errno));
    }
    return fd;
}
```

注释1处用于打开/dev/binder设备并返回文件操作符fd，这样就可以操作内核的Binder驱动了。注释2处的ioctl函数的作用就是和Binder设备进行参数的传递，这里的ioctl函数用于设定binder支持的最大线程数为15（maxThreads的值为15）。最终open_driver函数返回文件操作符fd。

ProcessState就分析倒这里，总的来说它做了以下几个重要的事：
1.打开/dev/binder设备并设定Binder最大的支持线程数。
2.通过mmap为binder分配一块虚拟地址空间，达到内存映射的目的。

### ServiceManager中的Binder机制

回到第一小节的MediaServer的入口函数，在注释2处调用了defaultServiceManager函数。
**frameworks/native/libs/binder/IServiceManager.cpp**

```cpp
sp<IServiceManager> defaultServiceManager()
{
    if (gDefaultServiceManager != NULL) return gDefaultServiceManager;

    {
        AutoMutex _l(gDefaultServiceManagerLock);
        while (gDefaultServiceManager == NULL) {
            gDefaultServiceManager = interface_cast<IServiceManager>(
                ProcessState::self()->getContextObject(NULL));//1
            if (gDefaultServiceManager == NULL)
                sleep(1);
        }
    }

    return gDefaultServiceManager;
}
```

从IServiceManager所在的文件路径就可以知道，ServiceManager中不仅仅使用了Binder通信，它自身也是属于Binder体系的。defaultServiceManager中同样使用了单例，注释1处的interface_cast函数生成了gDefaultServiceManager，其内部调用了ProcessState的getContextObject函数，代码如下所示。
**frameworks/native/libs/binder/ProcessState.cpp**

```cpp
sp<IBinder> ProcessState::getContextObject(const sp<IBinder>& /*caller*/)
{
    return getStrongProxyForHandle(0);
}

sp<IBinder> ProcessState::getStrongProxyForHandle(int32_t handle)
{
    sp<IBinder> result;
    AutoMutex _l(mLock);
    handle_entry* e = lookupHandleLocked(handle);//1
    if (e != NULL) {
        IBinder* b = e->binder;
        if (b == NULL || !e->refs->attemptIncWeak(this)) {
            if (handle == 0) {
                Parcel data;
                status_t status = IPCThreadState::self()->transact(
                        0, IBinder::PING_TRANSACTION, data, NULL, 0);
                if (status == DEAD_OBJECT)
                   return NULL;
            }
            b = BpBinder::create(handle);//2
            e->binder = b;
            if (b) e->refs = b->getWeakRefs();
            result = b;
        } else {
            result.force_set(b);
            e->refs->decWeak(this);
        }
    }

    return result;
}
```

getContextObject函数中直接调用了getStrongProxyForHandle函数，注意它的参数的值为0，那么handle的值就为0，handle是一个资源标识。注释1处查询这个资源标识对应的资源（handle_entry）是否存在，如果不存在就会在注释2处新建BpBinder，并在注释3处赋值给 handle_entry的binder。最终返回的result的值为BpBinder。

#### BpBinder和BBinder

说到BpBinder，不得不提到BBinder，它们是Binder通信的“双子星”，都继承了IBinder。BpBinder是Client端与Server交互的代理类，而BBinder则代表了Server端。BpBinder和BBinder是一一对应的，BpBinder会通过handle来找到对应的BBinder。
我们知道在ServiceManager中创建了BpBinder，通过handle(值为0)可以找到对应的BBinder。
![usIeXQ.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMi5heDF4LmNvbS8yMDE5LzEwLzA1L3VzSWVYUS5wbmc?x-oss-process=image/format,png)

分析完了ProcessState的getContextObject函数，回到interface_cast函数：

```cpp
 gDefaultServiceManager = interface_cast<IServiceManager>(
                ProcessState::self()->getContextObject(NULL));
```

interface_cast具体实现如下所示。
**frameworks/native/libs/binder/include/binder/IInterface.h**

```cpp
template<typename INTERFACE>
inline sp<INTERFACE> interface_cast(const sp<IBinder>& obj)
{
    return INTERFACE::asInterface(obj);
}
```

当前的场景中，INTERFACE的值为IServiceManager，那么替换后代码如下所示。

```cpp
inline sp<IServiceManager> interface_cast(const sp<IBinder>& obj){    return IServiceManager::asInterface(obj);}
```

我们接着来分析IServiceManager。

#### 解密IServiceManager

BpBinder和BBinder负责Binder的通信，而IServiceManager用于处理ServiceManager的业务，IServiceManager是C++代码，因此它的定义在IServiceManager.h中。
**frameworks/native/libs/binder/include/binder/IServiceManager.h**

```cpp
class IServiceManager : public IInterface
{
public:
    DECLARE_META_INTERFACE(ServiceManager)//1
    ...
    //一些操作Service的函数
    virtual sp<IBinder>         getService( const String16& name) const = 0;
    virtual sp<IBinder>         checkService( const String16& name) const = 0;
    virtual status_t addService(const String16& name, const sp<IBinder>& service,
                                bool allowIsolated = false,
                                int dumpsysFlags = DUMP_FLAG_PRIORITY_DEFAULT) = 0;
    virtual Vector<String16> listServices(int dumpsysFlags = DUMP_FLAG_PRIORITY_ALL) = 0;
    enum {
        GET_SERVICE_TRANSACTION = IBinder::FIRST_CALL_TRANSACTION,
        CHECK_SERVICE_TRANSACTION,
        ADD_SERVICE_TRANSACTION,
        LIST_SERVICES_TRANSACTION,
    };
};
```

可以看到IServiceManager继承了IInterface，其内部定义了一些常量和一些操作Service的函数，在注释1处调用了DECLARE_META_INTERFACE宏，它的定义在IInterface.h中。
**frameworks/native/libs/binder/include/binder/IInterface.h**

```cpp
#define DECLARE_META_INTERFACE(INTERFACE)                               \
    static const ::android::String16 descriptor;                        \
    static ::android::sp<I##INTERFACE> asInterface(                     \
            const ::android::sp<::android::IBinder>& obj);              \
    virtual const ::android::String16& getInterfaceDescriptor() const;  \
    I##INTERFACE();                                                     \
    virtual ~I##INTERFACE();   
```

其中INTERFACE的值为ServiceManager，那么经过替换后的代码如下所示。

```cpp
    static const ::android::String16 descriptor;       
    //定义asInterface函数
    static ::android::sp<IServiceManager> asInterface(                    
            const ::android::sp<::android::IBinder>& obj);            
    virtual const ::android::String16& getInterfaceDescriptor() const;  
    //定义IServiceManager构造函数
    IServiceManager();          
    //定义IServiceManager析构函数
    virtual ~IServiceManager();   
```

从DECLARE_META_INTERFACE宏的名称和上面的代码中，可以发现它主要声明了一些函数和一个变量。那么这些函数和变量的实现在哪呢？答案还是在IInterface.h中，叫做IMPLEMENT_META_INTERFACE宏，代码如下所示/
**frameworks/native/libs/binder/include/binder/IInterface.h**

```cpp
#define IMPLEMENT_META_INTERFACE(INTERFACE, NAME)                       \
    const ::android::String16 I##INTERFACE::descriptor(NAME);           \
    const ::android::String16&                                          \
            I##INTERFACE::getInterfaceDescriptor() const {              \
        return I##INTERFACE::descriptor;                                \
    }                                                                   \
    ::android::sp<I##INTERFACE> I##INTERFACE::asInterface(              \
            const ::android::sp<::android::IBinder>& obj)               \
    {                                                                   \
        ::android::sp<I##INTERFACE> intr;                               \
        if (obj != NULL) {                                              \
            intr = static_cast<I##INTERFACE*>(                          \
                obj->queryLocalInterface(                               \
                        I##INTERFACE::descriptor).get());               \
            if (intr == NULL) {                                         \
                intr = new Bp##INTERFACE(obj);                          \
            }                                                           \
        }                                                               \
        return intr;                                                    \
    }                                                                   \
    I##INTERFACE::I##INTERFACE() { }                                    \
    I##INTERFACE::~I##INTERFACE() { }                                   \
```

DECLARE_META_INTERFACE宏和IMPLEMENT_META_INTERFACE宏是配合使用的，很多系统服务都使用了它们，IServiceManager使用IMPLEMENT_META_INTERFACE宏只有一行代码，如下所示。
**frameworks/native/libs/binder/IServiceManager.cpp**

```cpp
IMPLEMENT_META_INTERFACE(ServiceManager, "android.os.IServiceManager");
```

IMPLEMENT_META_INTERFACE宏的INTERFACE值为ServiceManager，NAME值为"android.os.IServiceManager"，进行替换后的代码如下所示。

```cpp
const ::android::String16 IServiceManager::descriptor("android.os.IServiceManager");          
const ::android::String16&                                          
        IServiceManager::getInterfaceDescriptor() const {              
    return IServiceManager::descriptor;                                
} 
 //实现了asInterface函数
::android::sp<IServiceManager> IServiceManager::asInterface(              
        const ::android::sp<::android::IBinder>& obj)               
{                                                                   
    ::android::sp<IServiceManager> intr;                               
    if (obj != NULL) {                                              
        intr = static_cast<IServiceManager>(                          
            obj->queryLocalInterface(                               
                    IServiceManager::descriptor).get());               
        if (intr == NULL) {                                         
            intr = new BpServiceManager(obj);//1                        
        }                                                           
    }                                                               
    return intr;                                                    
}                                                                   
IServiceManager::IServiceManager() { }                                    
IServiceManager::~IServiceManager() { }                                   
```

关键的点就在于注释1处，新建了一个BpServiceManager，传入的参数obj的值为BpBinder。看到这里，我们也就明白了，asInterface函数就是用BpBinder为参数创建了BpServiceManager，从而推断出interface_cast函数创建了BpServiceManager，再往上推断，IServiceManager的defaultServiceManager函数返回的就是BpServiceManager。
BpServiceManager有什么作用呢，先从BpServiceManager的构造函数看起。
**frameworks/native/libs/binder/IServiceManager.cpp**

```cpp
class BpServiceManager : public BpInterface<IServiceManager>
{
public:
    explicit BpServiceManager(const sp<IBinder>& impl)
        : BpInterface<IServiceManager>(impl)
    {
    }
...
}
```

impl的值其实就是BpBinder，BpServiceManager的构造函数调用了基类BpInterface的构造函数。
**frameworks/native/libs/binder/include/binder/IInterface.h**

```cpp
template<typename INTERFACE>
class BpInterface : public INTERFACE, public BpRefBase
{
...
};
```

BpInterface继承了BpRefBase，BpRefBase的实现如下所示。
**frameworks/native/libs/binder/Binder.cpp**

```cpp
BpRefBase::BpRefBase(const sp<IBinder>& o)
    : mRemote(o.get()), mRefs(NULL), mState(0)
{
    extendObjectLifetime(OBJECT_LIFETIME_WEAK);

    if (mRemote) {
        mRemote->incStrong(this);           
        mRefs = mRemote->createWeak(this);  
    }
}
```

mRemote是一个IBinder* 指针，它最终的指向为BpBinder，也就是说BpServiceManager的mRemote指向了BpBinder。那么BpServiceManager的作用也就知道了，就是它实现了IServiceManager，并且通过BpBinder来实现通信。

#### IServiceManager家族

可能上面讲的会让你有些头晕，这是因为对各个类的关系不大明确，通过下图也许你就会豁然开朗。
![ufWhRI.png](https://s2.ax1x.com/2019/10/08/ufWhRI.png)

1.BpBinder和BBinder都和通信有关，它们都继承自IBinder。
2.BpServiceManager派生自IServiceManager，它们都和业务有关。
3.BpRefBase包含了mRemote，通过不断的派生，BpServiceManager也同样包含mRemote，它指向了BpBinder，通过BpBinder来实现通信。

### 小结

本篇我们学到了Binder通信的C/S架构，也知道了Native Binder的原理的核心其实就是ServiceManager的原理，为了讲解ServiceManager的原理，我们需要一个框架来举例，那就是MediaPlayer框架。在讲解MediaServer的入口函数时，我们遇到了三个问题，其中前两个问题相关的知识点ProcessState和IServiceManager都讲解到了，下一篇文章会讲解第三个问题，MediaPlayerService是如何注册的。

## Binder原理（三）系统服务的注册过程

在上一部分中，我们学习了ServiceManager中的Binder机制，有一个问题由于篇幅问题没有讲完，那就是MediaPlayerService是如何注册的。通过了解MediaPlayerService是如何注册的，可以得知系统服务的注册过程。

### 从调用链角度说明MediaPlayerService是如何注册的

我们先来看MediaServer的入口函数，代码如下所示。
**frameworks/av/media/mediaserver/main_mediaserver.cpp**

```cpp
int main(int argc __unused, char **argv __unused)
{
    signal(SIGPIPE, SIG_IGN);
    //获取ProcessState实例
    sp<ProcessState> proc(ProcessState::self());
    sp<IServiceManager> sm(defaultServiceManager());
    ALOGI("ServiceManager: %p", sm.get());
    InitializeIcuOrDie();
    //注册MediaPlayerService
    MediaPlayerService::instantiate();//1
    ResourceManagerService::instantiate();
    registerExtensions();
    //启动Binder线程池
    ProcessState::self()->startThreadPool();
    //当前线程加入到线程池
    IPCThreadState::self()->joinThreadPool();
}
```

这段代码中的很多内容都在上一篇文章介绍过了，接着分析注释1处的代码。

**frameworks/av/media/libmediaplayerservice/MediaPlayerService.cpp**

```cpp
void MediaPlayerService::instantiate() {
    defaultServiceManager()->addService(
            String16("media.player"), new MediaPlayerService，());
}
```

defaultServiceManager返回的是BpServiceManager，不清楚的看[Android Binder原理（二）ServiceManager中的Binder机制][1]这篇文章。参数是一个字符串和MediaPlayerService，看起来像是Key/Value的形式来完成注册，接着看addService函数。

**frameworks/native/libs/binder/IServiceManager.cpp**

```cpp
virtual status_t addService(const String16& name, const sp<IBinder>& service,
                               bool allowIsolated, int dumpsysPriority) {
       Parcel data, reply;//数据包
       data.writeInterfaceToken(IServiceManager::getInterfaceDescriptor());
       data.writeString16(name); //name值为"media.player"
       data.writeStrongBinder(service); //service值为MediaPlayerService
       data.writeInt32(allowIsolated ? 1 : 0);
       data.writeInt32(dumpsysPriority);
       status_t err = remote()->transact(ADD_SERVICE_TRANSACTION, data, &reply);//1
       return err == NO_ERROR ? reply.readExceptionCode() : err;
   }
```

data是一个数据包，后面会不断的将数据写入到data中， 注释1处的remote()指的是mRemote，也就是BpBinder。addService函数的作用就是将请求数据打包成data，然后传给BpBinder的transact函数，代码如下所示。
**frameworks/native/libs/binder/BpBinder.cpp**

```cpp
status_t BpBinder::transact(
    uint32_t code, const Parcel& data, Parcel* reply, uint32_t flags)
{
    if (mAlive) {
        status_t status = IPCThreadState::self()->transact(
            mHandle, code, data, reply, flags);
        if (status == DEAD_OBJECT) mAlive = 0;
        return status;
    }

    return DEAD_OBJECT;
}
```

BpBinder将逻辑处理交给IPCThreadState，先来看IPCThreadState::self()干了什么？
**frameworks/native/libs/binder/IPCThreadState.cpp**

```cpp
IPCThreadState* IPCThreadState::self()
{   
    //首次进来gHaveTLS的值为false
    if (gHaveTLS) {
restart:
        const pthread_key_t k = gTLS;//1
        IPCThreadState* st = (IPCThreadState*)pthread_getspecific(k);//2
        if (st) return st;
        return new IPCThreadState;//3
    }
    ...
    pthread_mutex_unlock(&gTLSMutex);
    goto restart;
}
```

注释1处的TLS的全称为Thread local storage，指的是线程本地存储空间，在每个线程中都有TLS，并且线程间不共享。注释2处用于获取TLS中的内容并赋值给IPCThreadState*指针。注释3处会新建一个IPCThreadState，这里可以得知IPCThreadState::self()实际上是为了创建IPCThreadState，它的构造函数如下所示。
**frameworks/native/libs/binder/IPCThreadState.cpp**

```cpp
IPCThreadState::IPCThreadState()
    : mProcess(ProcessState::self()),
      mStrictModePolicy(0),
      mLastTransactionBinderFlags(0)
{
    pthread_setspecific(gTLS, this);//1
    clearCaller();
    mIn.setDataCapacity(256);
    mOut.setDataCapacity(256);
}
```

注释1处的pthread_setspecific函数用于设置TLS，将IPCThreadState::self()获得的TLS和自身传进去。IPCThreadState中还包含mIn、一个mOut，其中mIn用来接收来自Binder驱动的数据，mOut用来存储发往Binder驱动的数据，它们默认大小都为256字节。
知道了IPCThreadState的构造函数，再回来查看IPCThreadState的transact函数。
**frameworks/native/libs/binder/IPCThreadState.cpp**

```cpp
status_t IPCThreadState::transact(int32_t handle,
                                  uint32_t code, const Parcel& data,
                                  Parcel* reply, uint32_t flags)
{
    status_t err;

    flags |= TF_ACCEPT_FDS;
    ...
    err = writeTransactionData(BC_TRANSACTION, flags, handle, code, data, NULL);//1

    if (err != NO_ERROR) {
        if (reply) reply->setError(err);
        return (mLastError = err);
    }

    if ((flags & TF_ONE_WAY) == 0) {
       ...
        if (reply) {
            err = waitForResponse(reply);//2
        } else {
            Parcel fakeReply;
            err = waitForResponse(&fakeReply);
        }
       ...
    } else {
       //不需要等待reply的分支
        err = waitForResponse(NULL, NULL);
    }

    return err;
}
```

调用BpBinder的transact函数实际上就是调用IPCThreadState的transact函数。注释1处的writeTransactionData函数用于传输数据，其中第一个参数BC_TRANSACTION代表向Binder驱动发送命令协议，向Binder设备发送的命令协议都以BC_开头，而Binder驱动返回的命令协议以BR_开头。这个命令协议我们先记住，后面会再次提到他。

现在分别来分析注释1的writeTransactionData函数和注释2处的waitForResponse函数。

#### writeTransactionData函数分析

**frameworks/native/libs/binder/IPCThreadState.cpp**

```cpp
status_t IPCThreadState::writeTransactionData(int32_t cmd, uint32_t binderFlags,
    int32_t handle, uint32_t code, const Parcel& data, status_t* statusBuffer)
{
    binder_transaction_data tr;//1

    tr.target.ptr = 0; 
    tr.target.handle = handle;//2 
    tr.code = code;  //code=ADD_SERVICE_TRANSACTION
    tr.flags = binderFlags;
    tr.cookie = 0;
    tr.sender_pid = 0;
    tr.sender_euid = 0;

    const status_t err = data.errorCheck();//3
    if (err == NO_ERROR) {
        tr.data_size = data.ipcDataSize();
        tr.data.ptr.buffer = data.ipcData();
        tr.offsets_size = data.ipcObjectsCount()*sizeof(binder_size_t);
        tr.data.ptr.offsets = data.ipcObjects();
    } else if (statusBuffer) {
        tr.flags |= TF_STATUS_CODE;
        *statusBuffer = err;
        tr.data_size = sizeof(status_t);
        tr.data.ptr.buffer = reinterpret_cast<uintptr_t>(statusBuffer);
        tr.offsets_size = 0;
        tr.data.ptr.offsets = 0;
    } else {
        return (mLastError = err);
    }

    mOut.writeInt32(cmd);  //cmd=BC_TRANSACTION
    mOut.write(&tr, sizeof(tr));

    return NO_ERROR;
}
```

注释1处的binder_transaction_data结构体(tr结构体）是向Binder驱动通信的数据结构，注释2处将handle传递给target的handle，用于标识目标，这里的handle的值为0，代表了ServiceManager。
注释3处对数据data进行错误检查，如果没有错误就将数据赋值给对应的tr结构体。最后会将BC_TRANSACTION和tr结构体写入到mOut中。
上面代码调用链的时序图如下所示。

![](https://upload-images.jianshu.io/upload_images/11474088-0e4cec13a8f7c169.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### waitForResponse函数分析

接着回过头来查看waitForResponse函数做了什么，waitForResponse函数中的case语句很多，这里截取部分代码。
**frameworks/native/libs/binder/IPCThreadState.cpp**

```cpp
status_t IPCThreadState::waitForResponse(Parcel *reply, status_t *acquireResult)
{
    uint32_t cmd;
    int32_t err;
    while (1) {
        if ((err=talkWithDriver()) < NO_ERROR) break;//1
        err = mIn.errorCheck();
        if (err < NO_ERROR) break;
        if (mIn.dataAvail() == 0) continue;
        cmd = (uint32_t)mIn.readInt32();
        IF_LOG_COMMANDS() {
            alog << "Processing waitForResponse Command: "
                << getReturnString(cmd) << endl;
        }
        switch (cmd) {
        case BR_TRANSACTION_COMPLETE:
            if (!reply && !acquireResult) goto finish;
            break;

        case BR_DEAD_REPLY:
            err = DEAD_OBJECT;
            goto finish;
       ...
        default:
            //处理各种命令协议
            err = executeCommand(cmd);
            if (err != NO_ERROR) goto finish;
            break;
        }
}
finish:
    ...
    return err;
}
```

注释1处的talkWithDriver函数的内部通过ioctl与Binder驱动进行通信，代码如下所示。
**frameworks/native/libs/binder/IPCThreadState.cpp**

```cpp
status_t IPCThreadState::talkWithDriver(bool doReceive)
{
    if (mProcess->mDriverFD <= 0) {
        return -EBADF;
    }
    //和Binder驱动通信的结构体
    binder_write_read bwr; //1
    //mIn是否有可读的数据，接收的数据存储在mIn
    const bool needRead = mIn.dataPosition() >= mIn.dataSize();
    const size_t outAvail = (!doReceive || needRead) ? mOut.dataSize() : 0;
    bwr.write_size = outAvail;
    bwr.write_buffer = (uintptr_t)mOut.data();//2
    //这时doReceive的值为true
    if (doReceive && needRead) {
        bwr.read_size = mIn.dataCapacity();
        bwr.read_buffer = (uintptr_t)mIn.data();//3
    } else {
        bwr.read_size = 0;
        bwr.read_buffer = 0;
    }
   ...
    if ((bwr.write_size == 0) && (bwr.read_size == 0)) return NO_ERROR;
    bwr.write_consumed = 0;
    bwr.read_consumed = 0;
    status_t err;
    do {
        IF_LOG_COMMANDS() {
            alog << "About to read/write, write size = " << mOut.dataSize() << endl;
        }
#if defined(__ANDROID__)
        if (ioctl(mProcess->mDriverFD, BINDER_WRITE_READ, &bwr) >= 0)//4
            err = NO_ERROR;
        else
            err = -errno;
#else
        err = INVALID_OPERATION;
#endif
     ...
    } while (err == -EINTR);
    ...
    return err;
}
```

注释1处的 binder_write_read是和Binder驱动通信的结构体，在注释2和3处将mOut、mIn赋值给binder_write_read的相应字段，最终通过注释4处的ioctl函数和Binder驱动进行通信，这一部分涉及到Kernel Binder的内容
了，就不再详细介绍了，只需要知道在Kernel Binder中会记录服务名和handle，用于后续的服务查询。

#### 小结

从调用链的角度来看，MediaPlayerService是如何注册的貌似并不复杂，因为这里只是简单的介绍了一个调用链分支，可以简单的总结为以下几个步骤：

1. addService函数将数据打包发送给BpBinder来进行处理。
2. BpBinder新建一个IPCThreadState对象，并将通信的任务交给IPCThreadState。
3. IPCThreadState的writeTransactionData函数用于将命令协议和数据写入到mOut中。
4. IPCThreadState的waitForResponse函数主要做了两件事，一件事是通过ioctl函数操作mOut和mIn来与Binder驱动进行数据交互，另一件事是处理各种命令协议。

### 从进程角度说明MediaPlayerService是如何注册的

实际上MediaPlayerService的注册还涉及到了进程，如下图所示。

![Ka0Dx0.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMS8xOS8xNmU4MTFjYmUxMzFiMTdi?x-oss-process=image/format,png)

从图中看出是以C/S架构为基础，addService是在MediaPlayerService进行的，它是Client端，用于请求添加系统服务。而Server端则是指的是ServiceManager，用于完成系统服务的添加。
Client端和Server端分别运行在两个进程中，通过向Binder来进行通信。更详细点描述，就是两端通过向Binder驱动发送命令协议来完成系统服务的添加。这其中命令协议非常多，过程也比较复杂，这里对命令协议进行了简化，只涉及到了四个命令协议，其中
BC_TRANSACTION和BR_TRANSACTION过程是一个完整的事务，BC_REPLY和BR_REPLY是一个完整的事务。
Client端和Server端向Binder驱动发送命令协议以BC开头，而Binder驱动向Client端和Server端返回的命令协议以BR_开头。

步骤如下所示：
1.Client端向Binder驱动发送BC_TRANSACTION命令。
2.Binder驱动接收到请求后生成BR_TRANSACTION命令，唤醒Server端的线程后将BR_TRANSACTION命令发送给ServiceManager。
3.Server端中的服务注册完成后，生成BC_REPLY命令发送给Binder驱动。
4.Binder驱动生成BR_REPLY命令，唤醒Client端的线程后将BR_REPLY命令发送个Client端。

通过这些协议命令来驱动并完成系统服务的注册。

### 总结

本文分别从调用链角度和进程角度来讲解MediaPlayerService是如何注册的，间接的得出了服务是如何注册的
。这两个角度都比较复杂，因此这里分别对这两个角度做了简化，作为应用开发，我们不需要注重太多的过程和细节，只需要了解大概的步骤即可。

## Binder原理（四）ServiceManager的启动过程

在上一部分中，我们以MediaPlayerService为例，讲解了系统服务是如何注册的（addService），既然有注册就势必要有获取，但是在了解获取服务前，我们最好先了解ServiceManager的启动过程，这样更有助于理解系统服务的注册和获取的过程。

另外还有一点需要说明的是，要想了解ServiceManager的启动过程，需要查看Kernel Binder部分的源码，这部分代码在内核源码中，AOSP源码是不包括内核源码的

### ServiceManager的入口函数

ServiceManager是init进程负责启动的，具体是在解析init.rc配置文件时启动的，init进程是在系统启动时启动的，因此ServiceManager亦是如此。

rc文件内部由Android初始化语言编写（Android Init Language）编写的脚本，它主要包含五种类型语句：Action、Commands、Services、Options和Import。
在Android 7.0中对init.rc文件进行了拆分，每个服务一个rc文件。ServiceManager的启动脚本在servicemanager.rc中：
frameworks/native/cmds/servicemanager/servicemanager.rc

```cpp
service servicemanager /system/bin/servicemanager
    class core animation
    user system  //1
    group system readproc
    critical //2
    onrestart restart healthd  
    onrestart restart zygote
    onrestart restart audioserver
    onrestart restart media
    onrestart restart surfaceflinger
    onrestart restart inputflinger
    onrestart restart drm
    onrestart restart cameraserver
    onrestart restart keystore
    onrestart restart gatekeeperd
    writepid /dev/cpuset/system-background/tasks
    shutdown critical
```

service用于通知init进程创建名为servicemanager的进程，这个servicemanager进程执行程序的路径为/system/bin/servicemanager。
注释1的关键字user说明servicemanager是以用户system的身份运行的，注释2处的critical说明servicemanager是系统中的关键服务，关键服务是不会退出的，如果退出了，系统就会重启，当系统重启时就会启动用onrestart关键字修饰的进程，比如zygote、media、surfaceflinger等等。

servicemanager的入口函数在service_manager.c中:
**frameworks/native/cmds/servicemanager/service_manager.c**

```cpp
int main(int argc, char** argv)
{
    struct binder_state *bs;//1
    union selinux_callback cb;
    char *driver;

    if (argc > 1) {
        driver = argv[1];
    } else {
        driver = "/dev/binder";
    }

    bs = binder_open(driver, 128*1024);//2
    ...
    if (binder_become_context_manager(bs)) {//3
        ALOGE("cannot become context manager (%s)\n", strerror(errno));
        return -1;
    }
    ...
    if (getcon(&service_manager_context) != 0) {
        ALOGE("SELinux: Failed to acquire service_manager context. Aborting.\n");
        abort();
    }
    binder_loop(bs, svcmgr_handler);//4

    return 0;
}
```

注释1处的binder_state结构体用来存储binder的三个信息：

```cpp
struct binder_state
{
    int fd; //binder设备的文件描述符
    void *mapped; //binder设备文件映射到进程的地址空间
    size_t mapsize; //内存映射后，系统分配的地址空间的大小，默认为128KB
};
```

main函数主要做了三件事：
1.注释2处调用binder_open函数用于打开binder设备文件，并申请128k字节大小的内存空间。
2.注释3处调用binder_become_context_manager函数，将servicemanager注册成为Binder机制的上下文管理者。
3.注释4处调用binder_loop函数，循环等待和处理client端发来的请求。

现在对这三件事分别进行讲解。

#### 打开binder设备

binder_open函数用于打开binder设备文件，并且将它映射到进程的地址空间，如下所示。

**frameworks/native/cmds/servicemanager/binder.c**

```cpp
struct binder_state *binder_open(const char* driver, size_t mapsize)
{
    struct binder_state *bs;
    struct binder_version vers;

    bs = malloc(sizeof(*bs));
    if (!bs) {
        errno = ENOMEM;
        return NULL;
    }

    bs->fd = open(driver, O_RDWR | O_CLOEXEC);//1
    if (bs->fd < 0) {
        fprintf(stderr,"binder: cannot open %s (%s)\n",
                driver, strerror(errno));
        goto fail_open;
    }
    //获取Binder的version
    if ((ioctl(bs->fd, BINDER_VERSION, &vers) == -1) ||
        (vers.protocol_version != BINDER_CURRENT_PROTOCOL_VERSION)) {//2
        fprintf(stderr,
                "binder: kernel driver version (%d) differs from user space version (%d)\n",
                vers.protocol_version, BINDER_CURRENT_PROTOCOL_VERSION);
        goto fail_open;
    }

    bs->mapsize = mapsize;
    bs->mapped = mmap(NULL, mapsize, PROT_READ, MAP_PRIVATE, bs->fd, 0);//3
    if (bs->mapped == MAP_FAILED) {
        fprintf(stderr,"binder: cannot map device (%s)\n",
                strerror(errno));
        goto fail_map;
    }
    return bs;

fail_map:
    close(bs->fd);
fail_open:
    free(bs);
    return NULL;
}
```

注释1处用于打开binder设备文件，后面会进行分析。
注释2处的ioctl函数用于获取Binder的版本，如果获取不到或者内核空间和用户空间的binder不是同一个版本就会直接goto到fail_open标签，释放binder的内存空间。
注释3处调用mmap函数进行内存映射，通俗来讲就是将binder设备文件映射到进程的地址空间，地址空间的大小为mapsize，也就是128K。映射完毕后会将地址空间的起始地址和大小保存在binder_state结构体中的mapped和mapsize变量中。

这里着重说一下open函数，它会调用Kernel Binder部分的binder_open函数，这部分源码位于内核源码中，这里展示的代码版本为goldfish3.4。

**用户态和内核态**
临时插入一个知识点:用户态和内核态
Intel的X86架构的CPU提供了0到3四个特权级，数字越小，权限越高，Linux操作系统中主要采用了0和3两个特权级，分别对应的就是内核态与用户态。用户态的特权级别低，因此进程在用户态下不经过系统调用是无法主动访问到内核空间中的数据的，这样用户无法随意的进入所有进程共享的内核空间，起到了保护的作用。下面来介绍下什么是用户态和内核态。
当一个进程在执行用户自己的代码时处于用户态，比如open函数，它运行在用户空间，当前的进程处于用户态。
当一个进程因为系统调用进入内核代码中执行时就处于内核态，比如open函数通过系统调用（__open()函数），查找到了open函数在Kernel Binder对应的函数为binder_open，这时binder_open运行在内核空间，当前的进程由用户态切换到内核态。

**kernel/goldfish/drivers/staging/android/binder.c**

```cpp
static int binder_open(struct inode *nodp, struct file *filp)
{   //代表Binder进程
	struct binder_proc *proc;//1
	binder_debug(BINDER_DEBUG_OPEN_CLOSE, "binder_open: %d:%d\n",
		     current->group_leader->pid, current->pid);
    //分配内存空间
	proc = kzalloc(sizeof(*proc), GFP_KERNEL);//2
	if (proc == NULL)
		return -ENOMEM;
	get_task_struct(current);
	proc->tsk = current;
	INIT_LIST_HEAD(&proc->todo);
	init_waitqueue_head(&proc->wait);
	proc->default_priority = task_nice(current);
    //binder同步锁
	binder_lock(__func__);

	binder_stats_created(BINDER_STAT_PROC);
	hlist_add_head(&proc->proc_node, &binder_procs);
	proc->pid = current->group_leader->pid;
	INIT_LIST_HEAD(&proc->delivered_death);
	filp->private_data = proc;//3
    //binder同步锁释放
	binder_unlock(__func__);
	...
	return 0;
}
```

注释1处的binder_proc结构体代表binder进程，用于管理binder的各种信息。注释2处用于为binder_proc分配内存空间。注释3处将binder_proc赋值给file指针的private_data变量，后面的1.2小节会再次提到这个private_data变量。

#### 注册成为Binder机制的上下文管理者

binder_become_context_manager函数用于将servicemanager注册成为Binder机制的上下文管理者，这个管理者在整个系统只有一个，代码如下所示。
**frameworks/native/cmds/servicemanager/binder.c**

```cpp
int binder_become_context_manager(struct binder_state *bs)
{
    return ioctl(bs->fd, BINDER_SET_CONTEXT_MGR, 0);
}
```

ioctl函数会调用Binder驱动的binder_ioctl函数，binder_ioctl函数代码比较多，这里截取BINDER_SET_CONTEXT_MGR的处理部分，代码如下所示。
**kernel/goldfish/drivers/staging/android/binder.c**

```cpp
static long binder_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{
	int ret;
	struct binder_proc *proc = filp->private_data; //1
	struct binder_thread *thread;
	unsigned int size = _IOC_SIZE(cmd);
	void __user *ubuf = (void __user *)arg;
	trace_binder_ioctl(cmd, arg);

	ret = wait_event_interruptible(binder_user_error_wait, binder_stop_on_user_error < 2);
	if (ret)
		goto err_unlocked;

	binder_lock(__func__);
	thread = binder_get_thread(proc);//2
	if (thread == NULL) {
		ret = -ENOMEM;
		goto err;
	}

	switch (cmd) {
    ...
	case BINDER_SET_CONTEXT_MGR:
		if (binder_context_mgr_node != NULL) {//3
			printk(KERN_ERR "binder: BINDER_SET_CONTEXT_MGR already set\n");
			ret = -EBUSY;
			goto err;
		}
		ret = security_binder_set_context_mgr(proc->tsk);
		if (ret < 0)
			goto err;
		if (binder_context_mgr_uid != -1) {//4
			if (binder_context_mgr_uid != current->cred->euid) {//5
				printk(KERN_ERR "binder: BINDER_SET_"
				       "CONTEXT_MGR bad uid %d != %d\n",
				       current->cred->euid,
				       binder_context_mgr_uid);
				ret = -EPERM;
				goto err;
			}
		} else
			binder_context_mgr_uid = current->cred->euid;//6
		binder_context_mgr_node = binder_new_node(proc, NULL, NULL);//7
		if (binder_context_mgr_node == NULL) {
			ret = -ENOMEM;
			goto err;
		}
		binder_context_mgr_node->local_weak_refs++;
		binder_context_mgr_node->local_strong_refs++;
		binder_context_mgr_node->has_strong_ref = 1;
		binder_context_mgr_node->has_weak_ref = 1;
		break;
 ...
err_unlocked:
	trace_binder_ioctl_done(ret);
	return ret;
}
```

注释1处将file指针中的private_data变量赋值给binder_proc，这个private_data变量在binder_open函数中讲过，是一个binder_proc结构体。注释2处的binder_get_thread函数用于获取binder_thread，binder_thread结构体指的是binder线程，binder_get_thread函数内部会从传入的参数binder_proc中查找binder_thread，如果查询到直接返回，如果查询不到会创建一个新的binder_thread并返回。
注释3处的全局变量binder_context_mgr_node代表的是Binder机制的上下文管理者对应的一个Binder对象，如果它不为NULL，说明此前自身已经被注册为Binder的上下文管理者了，Binder的上下文管理者是不能重复注册的，因此会goto到err标签。
注释4处的全局变量binder_context_mgr_uid代表注册了Binder机制上下文管理者的进程的有效用户ID，如果它的值不为-1，说明此前已经有进程注册Binder的上下文管理者了，因此在注释5处判断当前进程的有效用户ID是否等于binder_context_mgr_uid，不等于就goto到err标签。
如果不满足注释4的条件，说明此前没有进程注册Binder机制的上下文管理者，就会在注释6处将当前进程的有效用户ID赋值给全局变量binder_context_mgr_uid，另外还会在注释7处调用binder_new_node函数创建一个Binder对象并赋值给全局变量binder_context_mgr_node。

#### 循环等待和处理client端发来的请求

servicemanager成功注册成为Binder机制的上下文管理者后，servicemanager就是Binder机制的“总管”了，它需要在系统运行期间处理client端的请求，由于client端的请求不确定何时发送，因此需要通过无限循环来实现，实现这一需求的函数就是binder_loop。
**frameworks/native/cmds/servicemanager/binder.c**

```cpp
void binder_loop(struct binder_state *bs, binder_handler func)
{
    int res;
    struct binder_write_read bwr;
    uint32_t readbuf[32];

    bwr.write_size = 0;
    bwr.write_consumed = 0;
    bwr.write_buffer = 0;

    readbuf[0] = BC_ENTER_LOOPER;
    binder_write(bs, readbuf, sizeof(uint32_t));//1

    for (;;) {
        bwr.read_size = sizeof(readbuf);
        bwr.read_consumed = 0;
        bwr.read_buffer = (uintptr_t) readbuf;

        res = ioctl(bs->fd, BINDER_WRITE_READ, &bwr);//2

        if (res < 0) {
            ALOGE("binder_loop: ioctl failed (%s)\n", strerror(errno));
            break;
        }

        res = binder_parse(bs, 0, (uintptr_t) readbuf, bwr.read_consumed, func);//3
        if (res == 0) {
            ALOGE("binder_loop: unexpected reply?!\n");
            break;
        }
        if (res < 0) {
            ALOGE("binder_loop: io error %d %s\n", res, strerror(errno));
            break;
        }
    }
}
```

注释1处将BC_ENTER_LOOPER指令通过binder_write函数写入到Binder驱动中，这样当前线程（ServiceManager的主线程）就成为了一个Binder线程，这样就可以处理进程间的请求了。
在无限循环中不断的调用注释2处的ioctl函数，它不断的使用BINDER_WRITE_READ指令查询Binder驱动中是否有新的请求，如果有就交给注释3处的binder_parse函数处理。如果没有，当前线程就会在Binder驱动中睡眠，等待新的进程间请求。

由于binder_write函数的调用链中涉及到了内核空间和用户空间的交互，因此这里着重讲解下。

**frameworks/native/cmds/servicemanager/binder.c**

```cpp
int binder_write(struct binder_state *bs, void *data, size_t len)
{
    struct binder_write_read bwr;//1
    int res;

    bwr.write_size = len;
    bwr.write_consumed = 0;
    bwr.write_buffer = (uintptr_t) data;//2
    bwr.read_size = 0;
    bwr.read_consumed = 0;
    bwr.read_buffer = 0;
    res = ioctl(bs->fd, BINDER_WRITE_READ, &bwr);//3
    if (res < 0) {
        fprintf(stderr,"binder_write: ioctl failed (%s)\n",
                strerror(errno));
    }
    return res;
}
```

注释1处定义binder_write_read结构体，接下来的代码对bwr进行赋值，其中需要注意的是，注释2处的data的值为BC_ENTER_LOOPER。注释3处的ioctl函数将会bwr中的数据发送给binder驱动，我们已经知道了ioctl函数在Kernel Binder中对应的函数为binder_ioctl，此前分析过这个函数，这里截取BINDER_WRITE_READ命令处理部分。

**kernel/goldfish/drivers/staging/android/binder.c**

```cpp
static long binder_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
{   
    ...
    void __user *ubuf = (void __user *)arg;
    ...
	switch (cmd) {
	case BINDER_WRITE_READ: {
		struct binder_write_read bwr;
		if (size != sizeof(struct binder_write_read)) {
			ret = -EINVAL;
			goto err;
		}
		if (copy_from_user(&bwr, ubuf, sizeof(bwr))) {//1
			ret = -EFAULT;
			goto err;
		}
		binder_debug(BINDER_DEBUG_READ_WRITE,
			     "binder: %d:%d write %ld at %08lx, read %ld at %08lx\n",
			     proc->pid, thread->pid, bwr.write_size, bwr.write_buffer,
			     bwr.read_size, bwr.read_buffer);

		if (bwr.write_size > 0) {//2
			ret = binder_thread_write(proc, thread, (void __user *)bwr.write_buffer, bwr.write_size, &bwr.write_consumed);//3
			trace_binder_write_done(ret);
			if (ret < 0) {
				bwr.read_consumed = 0;
				if (copy_to_user(ubuf, &bwr, sizeof(bwr)))
					ret = -EFAULT;
				goto err;
			}
		}
	    ...
		binder_debug(BINDER_DEBUG_READ_WRITE,
			     "binder: %d:%d wrote %ld of %ld, read return %ld of %ld\n",
			     proc->pid, thread->pid, bwr.write_consumed, bwr.write_size,
			     bwr.read_consumed, bwr.read_size);
		if (copy_to_user(ubuf, &bwr, sizeof(bwr))) {//4
			ret = -EFAULT;
			goto err;
		}
		break;
	}
   ...
	return ret;
}
```

注释1处的copy_from_user函数，在本系列的第一篇**[Android Binder原理（一）学习Binder前必须要了解的知识点](# Android Binder原理（一）学习Binder前必须要了解的知识点)**提过。在这里，它用于将把用户空间数据ubuf拷贝出来保存到内核数据bwr（binder_write_read结构体）中。
注释2处，bwr的输入缓存区有数据时，会调用注释3处的binder_thread_write函数来处理BC_ENTER_LOOPER协议，其内部会将目标线程的状态设置为BINDER_LOOPER_STATE_ENTERED，这样目标线程就是一个Binder线程。
注释4处通过copy_to_user函数将内核空间数据bwr拷贝到用户空间。

### 总结

ServiceManager的启动过程实际上就是分析ServiceManager的入口函数，在入口函数中主要做了三件事，本篇深入到内核源码来对这三件逐一进行分析，由于涉及的函数比较多，这篇文章只介绍了我们需要掌握的，剩余大家可以自行阅读源码，比如binder_thread_write、copy_to_user函数。

## Binder原理（五）系统服务的获取过程

在本系列的此前文章中，以MediaPlayerService为例，讲解了系统服务是如何注册的（addService），既然有注册那肯定也要有获取，本篇文章仍旧以MediaPlayerService为例，来讲解系统服务的获取过程（getService）。文章会分为两个部分进行讲解，分别是客户端MediaPlayerService请求获取服务和服务端ServiceManager处理请求，先来学习第一部分。

### 客户端MediaPlayerService请求获取服务

要想获取MediaPlayerService，需要先调用getMediaPlayerService函数，如下所示。
frameworks/av/media/libmedia/IMediaDeathNotifier.cpp

```cpp
IMediaDeathNotifier::getMediaPlayerService()
{
    ALOGV("getMediaPlayerService");
    Mutex::Autolock _l(sServiceLock);
    if (sMediaPlayerService == 0) {
        sp<IServiceManager> sm = defaultServiceManager();//1
        sp<IBinder> binder;
        do {
            binder = sm->getService(String16("media.player"));//2
            if (binder != 0) {//3
                break;
            }
            ALOGW("Media player service not published, waiting...");
            usleep(500000); //4
        } while (true);

        if (sDeathNotifier == NULL) {
            sDeathNotifier = new DeathNotifier();
        }
        binder->linkToDeath(sDeathNotifier);
        sMediaPlayerService = interface_cast<IMediaPlayerService>(binder);//5
    }
    ALOGE_IF(sMediaPlayerService == 0, "no media player service!?");
    return sMediaPlayerService;
}
```

注释1处的defaultServiceManager返回的是BpServiceManager，注释2处获取名为”media.player”的系统服务（MediaPlayerService），返回的值为BpBinder。由于这个时候MediaPlayerService可能还没有向ServiceManager注册，那么就不能满足注释3的条件，在注释4处休眠0.5s后继续调用getService函数，直到获取服务对应的为止。
注释5处的interface_cast函数用于将BpBinder转换成BpMediaPlayerService，其原理就是通过BpBinder的handle来找到对应的服务，即BpMediaPlayerService。

注释2处的获取服务是本文的重点，BpServiceManager的getService函数如下所示。
**frameworks/native/libs/binder/IServiceManager.cpp::BpServiceManager**

```cpp
virtual sp<IBinder> getService(const String16& name) const
   {
      ...
       int n = 0;
       while (uptimeMillis() < timeout) {
           n++;
           if (isVendorService) {
               ALOGI("Waiting for vendor service %s...", String8(name).string());
               CallStack stack(LOG_TAG);
           } else if (n%10 == 0) {
               ALOGI("Waiting for service %s...", String8(name).string());
           }
           usleep(1000*sleepTime);

           sp<IBinder> svc = checkService(name);//1
           if (svc != NULL) return svc;
       }
       ALOGW("Service %s didn't start. Returning NULL", String8(name).string());
       return NULL;
   }
```

getService函数中主要做的事就是循环的查询服务是否存在，如果不存在就继续查询，查询服务用到了注释1处的checkService函数，代码如下所示。
**frameworks/native/libs/binder/IServiceManager.cpp::BpServiceManager**

```cpp
virtual sp<IBinder> checkService( const String16& name) const
{
    Parcel data, reply;//1
    data.writeInterfaceToken(IServiceManager::getInterfaceDescriptor());
    data.writeString16(name);//2
    remote()->transact(CHECK_SERVICE_TRANSACTION, data, &reply);//3
    return reply.readStrongBinder();
}
```

注释1处的data，看过上一篇文章的同学应该很熟悉，它出现在BpServiceManager的addService函数中，data是一个数据包，后面会不断的将数据写入到data中。注释2处将字符串"media.player"写入到data中。
注释3处的remote()指的是mRemote，也就是BpBinder，BpBinder的transact函数如下所示。

**frameworks/native/libs/binder/BpBinder.cpp**

```cpp
status_t BpBinder::transact(
    uint32_t code, const Parcel& data, Parcel* reply, uint32_t flags)
{
    if (mAlive) {
        status_t status = IPCThreadState::self()->transact(
            mHandle, code, data, reply, flags);
        if (status == DEAD_OBJECT) mAlive = 0;
        return status;
    }

    return DEAD_OBJECT;
}
```

BpBinder将逻辑处理交给IPCThreadState，后面的调用链在=[Android Binder原理（三）系统服务的注册过程](http://liuwangshu.cn/framework/binder/3-addservice.html)中讲过，这里再次简单的过一遍，IPCThreadState::self()会创建创建IPCThreadState，IPCThreadState的transact函数如下所示。
**frameworks/native/libs/binder/IPCThreadState.cpp**

```cpp
status_t IPCThreadState::transact(int32_t handle,
                                  uint32_t code, const Parcel& data,
                                  Parcel* reply, uint32_t flags)
{
    status_t err;

    flags |= TF_ACCEPT_FDS;
    ...
    err = writeTransactionData(BC_TRANSACTION, flags, handle, code, data, NULL);//1

    if (err != NO_ERROR) {
        if (reply) reply->setError(err);
        return (mLastError = err);
    }

    if ((flags & TF_ONE_WAY) == 0) {
       ...
        if (reply) {
            err = waitForResponse(reply);//2
        } else {
            Parcel fakeReply;
            err = waitForResponse(&fakeReply);
        }
       ...
    } else {
       //不需要等待reply的分支
        err = waitForResponse(NULL, NULL);
    }

    return err;
}
```

调用BpBinder的transact函数实际上就是调用IPCThreadState的transact函数。注释1处的writeTransactionData函数用于传输数据，其中第一个参数BC_TRANSACTION代表向Binder驱动发送命令协议。
注释1处的writeTransactionData用于准备发送的数据，其内部会将BC_TRANSACTION和binder_transaction_data结构体写入到mOut中。
接着查看waitForResponse函数做了什么，代码如下所示。
**frameworks/native/libs/binder/IPCThreadState.cpp**

```cpp
status_t IPCThreadState::waitForResponse(Parcel *reply, status_t *acquireResult)
{
    uint32_t cmd;
    int32_t err;
    while (1) {
        if ((err=talkWithDriver()) < NO_ERROR) break;//1
        err = mIn.errorCheck();
        if (err < NO_ERROR) break;
        if (mIn.dataAvail() == 0) continue;
        cmd = (uint32_t)mIn.readInt32();
        IF_LOG_COMMANDS() {
            alog << "Processing waitForResponse Command: "
                << getReturnString(cmd) << endl;
        }
        switch (cmd) {
        case BR_TRANSACTION_COMPLETE:
            if (!reply && !acquireResult) goto finish;
            break;
       ...
        default:
            //处理各种命令协议
            err = executeCommand(cmd);
            if (err != NO_ERROR) goto finish;
            break;
        }
}
finish:
    ...
    return err;
}
```

注释1处的talkWithDriver函数的内部通过ioctl与Binder驱动进行通信，代码如下所示。
**frameworks/native/libs/binder/IPCThreadState.cpp**

```cpp
status_t IPCThreadState::talkWithDriver(bool doReceive)
{
    if (mProcess->mDriverFD <= 0) {
        return -EBADF;
    }
    //和Binder驱动通信的结构体
    binder_write_read bwr; //1
    //mIn是否有可读的数据，接收的数据存储在mIn
    const bool needRead = mIn.dataPosition() >= mIn.dataSize();
    const size_t outAvail = (!doReceive || needRead) ? mOut.dataSize() : 0;
    bwr.write_size = outAvail;
    bwr.write_buffer = (uintptr_t)mOut.data();//2
    //这时doReceive的值为true
    if (doReceive && needRead) {
        bwr.read_size = mIn.dataCapacity();
        bwr.read_buffer = (uintptr_t)mIn.data();//3
    } else {
        bwr.read_size = 0;
        bwr.read_buffer = 0;
    }
   ...
    if ((bwr.write_size == 0) && (bwr.read_size == 0)) return NO_ERROR;
    bwr.write_consumed = 0;
    bwr.read_consumed = 0;
    status_t err;
    do {
        IF_LOG_COMMANDS() {
            alog << "About to read/write, write size = " << mOut.dataSize() << endl;
        }
#if defined(__ANDROID__)
        if (ioctl(mProcess->mDriverFD, BINDER_WRITE_READ, &bwr) >= 0)//4
            err = NO_ERROR;
        else
            err = -errno;
#else
        err = INVALID_OPERATION;
#endif
     ...
    } while (err == -EINTR);
    ...
    return err;
}
```

注释1处的 binder_write_read是和Binder驱动通信的结构体，在注释2和3处将mOut、mIn赋值给binder_write_read的相应字段，最终通过注释4处的ioctl函数和Binder驱动进行通信。这一过程的时序图如下所示。
[![MUr7w9.md.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMS8yNy8xNmVhYjNiMjRiZTViZmU5?x-oss-process=image/format,png)](https://imgchr.com/i/MUr7w9)

这时我们需要再次查看[Android Binder原理（三）系统服务的注册过程](#Android Binder原理（三）系统服务的注册过程)这篇第2小节给出的图。
![Ka0Dx0.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMS8yNy8xNmVhYjNiMjRiYmExZDRh?x-oss-process=image/format,png)

从这张简化的流程图可以看出，我们当前分析的是客户端进程的流程，当MediaPlayerService向Binder驱动发送BC_TRANSACTION命令后，Binder驱动会向ServiceManager发送BR_TRANSACTION命令，接下来我们来查看服务端ServiceManager是如何处理获取服务这一请求的。

### 服务端ServiceManager处理请求

说到服务端ServiceManager处理请求，不得不说到ServiceManager的启动过程，具体的请看[Android Binder原理（四）ServiceManager的启动过程](#Android Binder原理（四）ServiceManager的启动过程) 这篇。
这里简单回顾servicemanager的入口函数，如下所示。

**frameworks/native/cmds/servicemanager/service_manager.c**

```cpp
int main(int argc, char** argv)
{
   ...
    bs = binder_open(driver, 128*1024);
    ...
    if (binder_become_context_manager(bs)) {
        ALOGE("cannot become context manager (%s)\n", strerror(errno));
        return -1;
    }
    ...
    if (getcon(&service_manager_context) != 0) {
        ALOGE("SELinux: Failed to acquire service_manager context. Aborting.\n");
        abort();
    }
    binder_loop(bs, svcmgr_handler);//1
    return 0;
}
```

main函数主要做了三件事，其中最后一件事就是调用binder_loop函数，这里需要注意，它的第二个参数为svcmgr_handler，后面会再次提到svcmgr_handler。
binder_loop函数如下所示。
**frameworks/native/cmds/servicemanager/binder.c**

```cpp
void binder_loop(struct binder_state *bs, binder_handler func)
{
...
    for (;;) {
        bwr.read_size = sizeof(readbuf);
        bwr.read_consumed = 0;
        bwr.read_buffer = (uintptr_t) readbuf;

        res = ioctl(bs->fd, BINDER_WRITE_READ, &bwr);

        if (res < 0) {
            ALOGE("binder_loop: ioctl failed (%s)\n", strerror(errno));
            break;
        }
        res = binder_parse(bs, 0, (uintptr_t) readbuf, bwr.read_consumed, func);
        if (res == 0) {
            ALOGE("binder_loop: unexpected reply?!\n");
            break;
        }
        if (res < 0) {
            ALOGE("binder_loop: io error %d %s\n", res, strerror(errno));
            break;
        }
    }
}
```

在无限循环中不断的调用ioctl函数，它不断的使用BINDER_WRITE_READ指令查询Binder驱动中是否有新的请求，如果有就交给binder_parse函数处理。如果没有，当前线程就会在Binder驱动中睡眠，等待新的进程间通信请求。
binder_parse函数如下所示。
**frameworks/native/cmds/servicemanager/binder.c**

```cpp
int binder_parse(struct binder_state *bs, struct binder_io *bio,
                 uintptr_t ptr, size_t size, binder_handler func)
{
    int r = 1;
    uintptr_t end = ptr + (uintptr_t) size;

    while (ptr < end) {
        uint32_t cmd = *(uint32_t *) ptr;
        ptr += sizeof(uint32_t);
#if TRACE
        fprintf(stderr,"%s:\n", cmd_name(cmd));
#endif
        switch(cmd) {
        ...
        case BR_TRANSACTION: {
            struct binder_transaction_data *txn = (struct binder_transaction_data *) ptr;
            if ((end - ptr) < sizeof(*txn)) {
                ALOGE("parse: txn too small!\n");
                return -1;
            }
            binder_dump_txn(txn);
            if (func) {
                unsigned rdata[256/4];
                struct binder_io msg;
                struct binder_io reply;
                int res;

                bio_init(&reply, rdata, sizeof(rdata), 4);
                bio_init_from_txn(&msg, txn);
                res = func(bs, txn, &msg, &reply);//1
                if (txn->flags & TF_ONE_WAY) {
                    binder_free_buffer(bs, txn->data.ptr.buffer);
                } else {
                    binder_send_reply(bs, &reply, txn->data.ptr.buffer, res);
                }
            }
            ptr += sizeof(*txn);
            break;
        }
        ...
    }

    return r;
}
```

这里截取了BR_TRANSACTION命令的处理部分，注释1出的func通过一路传递指向的是svcmgr_handler，svcmgr_handler函数如下所示。
**frameworks/native/cmds/servicemanager/service_manager.c**

```cpp
int svcmgr_handler(struct binder_state *bs,
                   struct binder_transaction_data *txn,
                   struct binder_io *msg,
                   struct binder_io *reply)
{
    ...
    switch(txn->code) {
    case SVC_MGR_GET_SERVICE:
    case SVC_MGR_CHECK_SERVICE:
        s = bio_get_string16(msg, &len);
        if (s == NULL) {
            return -1;
        }
        handle = do_find_service(s, len, txn->sender_euid, txn->sender_pid);
        if (!handle)
            break;
        bio_put_ref(reply, handle);
        return 0;

   ...
    default:
        ALOGE("unknown code %d\n", txn->code);
        return -1;
    }

    bio_put_uint32(reply, 0);
    return 0;
}
```

当要获取服务时，会调用do_find_service函数，代码如下所示。
**frameworks/native/cmds/servicemanager/service_manager.c**

```cpp
uint32_t do_find_service(const uint16_t *s, size_t len, uid_t uid, pid_t spid)
{
    struct svcinfo *si = find_svc(s, len);//1

    if (!si || !si->handle) {
        return 0;
    }

    if (!si->allow_isolated) {
        uid_t appid = uid % AID_USER;
        if (appid >= AID_ISOLATED_START && appid <= AID_ISOLATED_END) {
            return 0;
        }
    }
    if (!svc_can_find(s, len, spid, uid)) {
        return 0;
    }

    return si->handle;
}
```

注释1处的find_svc函数用于查询服务，返回的svcinfo是一个结构体，其内部包含了服务的handle值，最终会返回服务的handle值。接着来看find_svc函数：
**frameworks/native/cmds/servicemanager/service_manager.c**

```cpp
struct svcinfo *find_svc(const uint16_t *s16, size_t len)
{
    struct svcinfo *si;

    for (si = svclist; si; si = si->next) {
        if ((len == si->len) &&
            !memcmp(s16, si->name, len * sizeof(uint16_t))) {
            return si;
        }
    }
    return NULL;
}
```

系统服务的注册流程中，在Kernel Binder中会调用do_add_service函数，其内部会将包含服务名和handle值的svcinfo保存到svclist列表中。同样的，在获取服务的流程中，find_svc函数中会遍历svclist列表，根据服务名查找对应服务是否已经注册，如果已经注册就会返回对应的svcinfo，如果没有注册就返回NULL。

### 总结

这篇将系统服务的获取过程分为两个部分，代码涉及到了Native Binder和Kernel Binder。在下一篇文章中会继续学习Java Binder相关的内容。

## Binder原理（六）Java Binder的初始化

在[Android Binder原理（一）学习Binder前必须要了解的知识点](#Android Binder原理（一）学习Binder前必须要了解的知识点)这篇中，我根据Android系统的分层，将Binder机制分为了三层：

1. Java Binder (对应Framework层的Binder)
2. Native Binder(对应Native层的Binder)
3. Kernel Binder(对应Kernel层的Binder)

在此前，我一直都在介绍Native Binder和Kernel Binder的内容，它们的架构简单总结为下图。

[![MgRMbF.png](https://s2.ax1x.com/2019/11/19/MgRMbF.png)](https://s2.ax1x.com/2019/11/19/MgRMbF.png)

在[Android Binder原理（二）ServiceManager中的Binder机制](#Android Binder原理（二）ServiceManager中的Binder机制)这篇中，我讲过BpBinder是Client端与Server交互的代理类，而BBinder则代表了Server端，那么上图就可以改为：
[![MgWuRI.png](https://s2.ax1x.com/2019/11/19/MgWuRI.png)](https://s2.ax1x.com/2019/11/19/MgWuRI.png)
从上图可以看到，Native Binder实际是基于C/S架构，Bpinder是Client端，BBinder是Server端，在[Android Binder原理（四）ServiceManager的启动过程](#Android Binder原理（四）ServiceManager的启动过程)这篇中，我们得知Native Binder通过ioctl函数和Binder驱动进行数据交互。
Java Binder是需要借助Native Binder来进行工作的，因此Java Binder在设计上也是一个C/S架构，可以说Java Binder是Native Binder的一个镜像，所以在学习Java Binder前，最好先要学习此前文章讲解的Native Binder的内容。本篇文章先来讲解Java Binder是如何初始化的，即Java Binder的JNI注册。

### Java Binder的JNI注册

Java Binder要想和Native Binder进行通信，需要通过JNI，JNI的注册是在Zygote进程启动过程中注册的，代码如下所示。
**frameworks/base/core/jni/AndroidRuntime.cpp**

```cpp
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote)
{
    ...
    JniInvocation jni_invocation;
    jni_invocation.Init(NULL);
    JNIEnv* env;
    if (startVm(&mJavaVM, &env, zygote) != 0) {//1
        return;
    }
    onVmCreated(env);
    if (startReg(env) < 0) {
        ALOGE("Unable to register all android natives\n");
        return;
    }
   ...
}
```

注释1处用于启动Java虚拟机，注释2处startReg函数用于完成虚拟机的JNI注册，关于AndroidRuntime的start函数的具体分析见[Android系统启动流程（二）解析Zygote进程启动过程](# Android系统启动流程（二）解析Zygote进程启动过程)这篇。
startReg函数如下所示。
**frameworks/base/core/jni/AndroidRuntime.cpp**

```cpp
/*static*/ int AndroidRuntime::startReg(JNIEnv* env)
{
    ATRACE_NAME("RegisterAndroidNatives");
    androidSetCreateThreadFunc((android_create_thread_fn) javaCreateThreadEtc);

    ALOGV("--- registering native functions ---\n");
    env->PushLocalFrame(200);

    if (register_jni_procs(gRegJNI, NELEM(gRegJNI), env) < 0) {//1
        env->PopLocalFrame(NULL);
        return -1;
    }
    env->PopLocalFrame(NULL);
    return 0;
}
```

注释1处的register_jni_procs函数的作用就是循环调用gRegJNI数组的成员所对应的方法，如下所示。

```cpp
static int register_jni_procs(const RegJNIRec array[], size_t count, JNIEnv* env)
{
    for (size_t i = 0; i < count; i++) {
        if (array[i].mProc(env) < 0) {
#ifndef NDEBUG
            ALOGD("----------!!! %s failed to load\n", array[i].mName);
#endif
            return -1;
        }
    }
    return 0;
}
```

gRegJNI数组中有100多个成员变量：

```cpp
static const RegJNIRec gRegJNI[] = {
    REG_JNI(register_com_android_internal_os_RuntimeInit),
    REG_JNI(register_com_android_internal_os_ZygoteInit_nativeZygoteInit),
    REG_JNI(register_android_os_SystemClock),
    ...
    REG_JNI(register_android_os_Binder),//1
   ...
};    
```

其中REG_JNI是一个宏定义：

```cpp
#define REG_JNI(name)      { name }
struct RegJNIRec {
    int (*mProc)(JNIEnv*);
};
```

实际上就是调用参数名所对应的函数。负责Java Binder和Native Binder通信的函数为注释1处的register_android_os_Binder，代码如下所示。
**frameworks/base/core/jni/android_util_Binder.cpp**

```cpp
int register_android_os_Binder(JNIEnv* env)
{   
    //注册Binder类
    if (int_register_android_os_Binder(env) < 0)
        return -1;
    //注册BinderInternal类        
    if (int_register_android_os_BinderInternal(env) < 0)
        return -1;
    //注册BinderProxy类          
    if (int_register_android_os_BinderProxy(env) < 0)
        return -1;
    ...
    return 0;
}
```

register_android_os_Binder函数做了三件事，分别是:
1.注册Binder类
2.注册BinderInternal类
3.注册BinderProxy类

它们是Java Binder关联类的一小部分，它们的关系如下图所示。

[![MTmhzd.png](https://s2.ax1x.com/2019/11/22/MTmhzd.png)](https://s2.ax1x.com/2019/11/22/MTmhzd.png)

- IBinder接口中定义了很多整型的变量，其中定义一个叫做`FLAG_ONEWAY`的整形变量。客户端发起调用时，客户端一般会阻塞，直到服务端返回结果。设置`FLAG_ONEWAY`后，客户端只需要把请求发送到服务端就可以立即返回，而不需要等待服务端的结果，这是一种非阻塞方式。
- Binder和BinderProxy实现了IBinder接口，Binder是服务端的代表，而BinderProxy是客户端的代表。
- BinderInternal只是在Binder框架中被使用，其内部类GcWatcher用于处理和Binder的垃圾回收。
- Parcel是一个数据包装器，它可以在进程间进行传递，Parcel既可以传递基本数据类型也可以传递Binder对象，Binder通信就是通过Parcel来进行客户端与服务端数据交互。Parcel的实现既有Java部分，也有Native部分，具体实现在Native部分中。

下面分别对Binder、BinderInternal这两个类的注册进行分析。

#### Binder类的注册

调用int_register_android_os_Binder函数来完成Binder类的注册，代码如下所示。
**frameworks/base/core/jni/android_util_Binder.cpp**

```cpp
static const JNINativeMethod gBinderMethods[] = {
     /* name, signature, funcPtr */
    { "getCallingPid", "()I", (void*)android_os_Binder_getCallingPid },
    { "getCallingUid", "()I", (void*)android_os_Binder_getCallingUid },
    { "clearCallingIdentity", "()J", (void*)android_os_Binder_clearCallingIdentity },
    { "restoreCallingIdentity", "(J)V", (void*)android_os_Binder_restoreCallingIdentity },
    { "setThreadStrictModePolicy", "(I)V", (void*)android_os_Binder_setThreadStrictModePolicy },
    { "getThreadStrictModePolicy", "()I", (void*)android_os_Binder_getThreadStrictModePolicy },
    { "flushPendingCommands", "()V", (void*)android_os_Binder_flushPendingCommands },
    { "getNativeBBinderHolder", "()J", (void*)android_os_Binder_getNativeBBinderHolder },
    { "getNativeFinalizer", "()J", (void*)android_os_Binder_getNativeFinalizer },
    { "blockUntilThreadAvailable", "()V", (void*)android_os_Binder_blockUntilThreadAvailable }
};
const char* const kBinderPathName = "android/os/Binder";//1
static int int_register_android_os_Binder(JNIEnv* env)
{
    jclass clazz = FindClassOrDie(env, kBinderPathName);//2

    gBinderOffsets.mClass = MakeGlobalRefOrDie(env, clazz);//3
    gBinderOffsets.mExecTransact = GetMethodIDOrDie(env, clazz, "execTransact", "(IJJI)Z");//4
    gBinderOffsets.mObject = GetFieldIDOrDie(env, clazz, "mObject", "J");

    return RegisterMethodsOrDie(
        env, kBinderPathName,
        gBinderMethods, NELEM(gBinderMethods));
}
```

注释1处的kBinderPathName的值为”android/os/Binder”，这是Binder在Java Binder中的全路径名。
注释2处根据这个路径名获取Binder的Class对象，并赋值给jclass类型的变量clazz，clazz是Java层Binder在JNI层的代表。
注释3处通过MakeGlobalRefOrDie函数将本地引用clazz转变为全局引用并赋值给gBinderOffsets.mClass。
注释4处用于找到Java层的Binder的成员方法execTransact并赋值给gBinderOffsets.mExecTransact。
注释5处用于找到Java层的Binder的成员变量mObject并赋值给gBinderOffsets.mObject。
最后一行通过RegisterMethodsOrDie函数注册gBinderMethods中定义的函数，其中gBinderMethods是JNINativeMethod类型的数组，里面存储的是Binder的Native方法（Java层）与JNI层函数的对应关系。

gBinderMethods的定义如下所示。

```cpp
static struct bindernative_offsets_t
{
    jclass mClass;
    jmethodID mExecTransact;
    jfieldID mObject;

} gBinderOffsets;
```

使用gBinderMethods来保存变量和方法有两个原因：
1.为了效率考虑，如果每次调用相关的方法时都需要查询方法和变量，显然效率比较低。
2.这些成员变量和方法都是本地引用，在int int_register_android_os_Binder函数返回时，这些本地引用会被自动释放，因此用gBinderOffsets来保存，以便于后续使用。

对于JNI不大熟悉的同学可以看[Android深入理解JNI（二）类型转换、方法签名和JNIEnv](http://liuwangshu.cn/framework/jni/2-signature-jnienv)这篇文章。

#### BinderInternal类的注册

调用int_register_android_os_BinderInternal函数来完成BinderInternal类的注册，代码如下所示。
**frameworks/base/core/jni/android_util_Binder.cpp**

```cpp
const char* const kBinderInternalPathName = "com/android/internal/os/BinderInternal";
static int int_register_android_os_BinderInternal(JNIEnv* env)
{
    jclass clazz = FindClassOrDie(env, kBinderInternalPathName);

    gBinderInternalOffsets.mClass = MakeGlobalRefOrDie(env, clazz);
    gBinderInternalOffsets.mForceGc = GetStaticMethodIDOrDie(env, clazz, "forceBinderGc", "()V");
    gBinderInternalOffsets.mProxyLimitCallback = GetStaticMethodIDOrDie(env, clazz, "binderProxyLimitCallbackFromNative", "(I)V");

    jclass SparseIntArrayClass = FindClassOrDie(env, "android/util/SparseIntArray");
    gSparseIntArrayOffsets.classObject = MakeGlobalRefOrDie(env, SparseIntArrayClass);
    gSparseIntArrayOffsets.constructor = GetMethodIDOrDie(env, gSparseIntArrayOffsets.classObject,
                                                           "<init>", "()V");
    gSparseIntArrayOffsets.put = GetMethodIDOrDie(env, gSparseIntArrayOffsets.classObject, "put",
                                                   "(II)V");

    BpBinder::setLimitCallback(android_os_BinderInternal_proxyLimitcallback);

    return RegisterMethodsOrDie(
        env, kBinderInternalPathName,
        gBinderInternalMethods, NELEM(gBinderInternalMethods));
}
```

和int_register_android_os_Binder函数的实现类似，主要做了三件事：
1.获取BinderInternal在JNI层的代表clazz。
2.将BinderInternal类中有用的成员变量和方法存储到gBinderInternalOffsets中。
3.注册BinderInternal类的Native方法对应的JNI函数。

还有一个BinderProxy类的注册，它和Binder、BinderInternal的注册过程差不多，这里就不再赘述了，有兴趣的读者可以自行去看源码。

## Binder原理（七）Java Binder中系统服务的注册过程

在[Android Binder原理（三）系统服务的注册过程](#Android Binder原理（三）系统服务的注册过程)这篇文章中，我介绍的是Native Binder中的系统服务的注册过程，这一过程的核心是ServiceManager，而在Java Binder中，也有一个ServiceManager，只不过这个ServiceManager是Java文件。
既然要将系统服务注册到ServiceManager，那么需要选择一个系统服务为例，这里以常见的AMS为例。

### 将AMS注册到ServiceManager

在AMS的setSystemProcess方法中，会调用ServiceManager的addService方法，如下所示。
**frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java**

```cpp
public void setSystemProcess() {
    try {
        ServiceManager.addService(Context.ACTIVITY_SERVICE, this, /* allowIsolated= */ true,
                DUMP_FLAG_PRIORITY_CRITICAL | DUMP_FLAG_PRIORITY_NORMAL | DUMP_FLAG_PROTO);//1
       ....
    } catch (PackageManager.NameNotFoundException e) {
        throw new RuntimeException(
                "Unable to find android system package", e);
    }
 ...
}
```

注释1处的`Context.ACTIVITY_SERVICE`的值为”activity”，作用就是将AMS注册到ServiceManager中。接着来看
ServiceManager的addService方法。
**frameworks/base/core/java/android/os/ServiceManager.java**

```cpp
public static void addService(String name, IBinder service, boolean allowIsolated,
        int dumpPriority) {
    try {
        getIServiceManager().addService(name, service, allowIsolated, dumpPriority);
    } catch (RemoteException e) {
        Log.e(TAG, "error in addService", e);
    }
}
```

主要分析getIServiceManager方法返回的是什么，代码如下所示。
**frameworks/base/core/java/android/os/ServiceManager.java**

```cpp
private static IServiceManager getIServiceManager() {
     if (sServiceManager != null) {
         return sServiceManager;
     }
     sServiceManager = ServiceManagerNative
             .asInterface(Binder.allowBlocking(BinderInternal.getContextObject()));
     return sServiceManager;
 }
```

讲到这里，已经积累了几个点需要分析，分别是：

- BinderInternal.getContextObject()
- ServiceManagerNative.asInterface()
- getIServiceManager().addService()

现在我们来各个击破它们。

#### BinderInternal.getContextObject()

Binder.allowBlocking的作用是将BinderProxy的sWarnOnBlocking值置为false。主要来分析BinderInternal.getContextObject()做了什么，这个方法是一个Native方法，找到它对应的函数：
**frameworks/base/core/jni/android_util_Binder.cpp**

```cpp
static const JNINativeMethod gBinderInternalMethods[] = {
    { "getContextObject", "()Landroid/os/IBinder;", (void*)android_os_BinderInternal_getContextObject },
   ...
};
```

对应的函数为android_os_BinderInternal_getContextObject：
**frameworks/base/core/jni/android_util_Binder.cpp**

```cpp
static jobject android_os_BinderInternal_getContextObject(JNIEnv* env, jobject clazz)
{
    sp<IBinder> b = ProcessState::self()->getContextObject(NULL);//1
    return javaObjectForIBinder(env, b);
}
```

ProcessState::self()的作用是创建ProcessState，注释1处最终返回的是BpBinder，不理解的可以查看[Android Binder原理（二）ServiceManager中的Binder机制](#Android Binder原理（二）ServiceManager中的Binder机制)这篇文章。

BpBinder是Native Binder中的Client端，这说明Java层的ServiceManager需要Native层的BpBinder，但是这个BpBinder在Java层是无法直接使用，那么就需要传入javaObjectForIBinder函数来做处理，其内部会创建一个BinderProxy对象，这样我们得知 BinderInternal.getContextObject()最终得到的是BinderProxy。
在[Android Binder原理（六）Java Binder的初始化](#Android Binder原理（六）Java Binder的初始化)这篇文章我们讲过，BinderProxy是Java Binder的客户端的代表。
需要注意的一点是，这个传入的BpBinder会保存到BinderProxy的成员变量mObject中，后续会再次提到这个点。

#### ServiceManagerNative.asInterface()

说到asInterface方法，在Native Binder中也有一个asInterface函数。在[Android Binder原理（二）ServiceManager中的Binder机制](#Android Binder原理（二）ServiceManager中的Binder机制)这篇文章中讲过IServiceManager的asInterface函数，它的作用是用BpBinder做为参数创建BpServiceManager。那么在Java Binder中的asInterface方法的作用又是什么？
**frameworks/base/core/java/android/os/ServiceManagerNative.java**

```cpp
static public IServiceManager asInterface(IBinder obj)
 {
     if (obj == null) {
         return null;
     }
     IServiceManager in =
         (IServiceManager)obj.queryLocalInterface(descriptor);
     if (in != null) {
         return in;
     }

     return new ServiceManagerProxy(obj);
 }
```

根据1.1小节，我们得知obj的值为BinderProxy，那么asInterface方法的作用就是用BinderProxy作为参数创建ServiceManagerProxy。
BinderProxy和BpBinder分别在Jave Binder和Native Binder作为客户端的代表，BpServiceManager通过BpBinder来实现通信，同样的，ServiceManagerProxy也会将业务的请求交给BinderProxy来处理。
分析到这里，那么：

```cpp
sServiceManager = ServiceManagerNative
        .asInterface(Binder.allowBlocking(BinderInternal.getContextObject()));
```

可以理解为：

```cpp
    sServiceManager = new ServiceManagerProxy（BinderProxy);
}
```

#### getIServiceManager().addService()

根据1.2节的讲解，getIServiceManager()返回的是ServiceManagerProxy，ServiceManagerProxy是ServiceManagerNative的内部类，它实现了IServiceManager接口。

来查看ServiceManagerProxy的addService方法，
**frameworks/base/core/java/android/os/ServiceManagerNative.java::ServiceManagerProxy**

```cpp
public void addService(String name, IBinder service, boolean allowIsolated, int dumpPriority)
        throws RemoteException {
    Parcel data = Parcel.obtain();
    Parcel reply = Parcel.obtain();
    data.writeInterfaceToken(IServiceManager.descriptor);
    data.writeString(name);
    data.writeStrongBinder(service);//1
    data.writeInt(allowIsolated ? 1 : 0);
    data.writeInt(dumpPriority);
    mRemote.transact(ADD_SERVICE_TRANSACTION, data, reply, 0);//2
    reply.recycle();
    data.recycle();
}
```

注释1处的data.writeStrongBinder很关键，后续会进行分析。这里又看到了Parcel，它是一个数据包装器，将请求数据写入到Parcel类型的对象data中，通过注释1处的mRemote.transact发送出去，mRemote实际上是BinderProxy，BinderProxy.transact是native函数，实现的函数如下所示。
**frameworks/base/core/jni/android_util_Binder.cpp**

```cpp
static jboolean android_os_BinderProxy_transact(JNIEnv* env, jobject obj,
        jint code, jobject dataObj, jobject replyObj, jint flags) // throws RemoteException
{
    if (dataObj == NULL) {
        jniThrowNullPointerException(env, NULL);
        return JNI_FALSE;
    }
    Parcel* data = parcelForJavaObject(env, dataObj);//1
    if (data == NULL) {
        return JNI_FALSE;
    }
    Parcel* reply = parcelForJavaObject(env, replyObj);//2
    if (reply == NULL && replyObj != NULL) {
        return JNI_FALSE;
    }
    IBinder* target = getBPNativeData(env, obj)->mObject.get();//3
    if (target == NULL) {
        jniThrowException(env, "java/lang/IllegalStateException", "Binder has been finalized!");
        return JNI_FALSE;
    }
   ...
    status_t err = target->transact(code, *data, reply, flags);//4
    return JNI_FALSE;
}
```

注释1和注释2处，将Java层的Parcel对象转化成为Native层的Parcel对象。在1.1小节中，我们得知BpBinder会保存到BinderProxy的成员变量mObject中，因此在注释3处，从BinderProxy的成员变量mObject中获取BpBinder。最终会在注释4处调用BpBinder的transact函数，向Binder驱动发送数据，可以看出Java Binder是需要Native Binder支持的，最终的目的就是向Binder驱动发送和接收数据。

### 引出JavaBBinder

接着回过头来分析1.3小节遗留下来的data.writeStrongBinder(service)，代码如下所示。
**frameworks/base/core/java/android/os/Parcel.java**

```cpp
public final void writeStrongBinder(IBinder ll) {
      nativeWriteStrongBinder(mNativePtr, val);
  }
```

nativeWriteStrongBinder是Native方法，实现的函数为android_os_Parcel_writeStrongBinder：
**frameworks/base/core/jni/android_os_Parcel.cpp**

```cpp
static void android_os_Parcel_writeStrongBinder(JNIEnv* env, jclass clazz, jlong nativePtr, jobject object)
{
    Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
    if (parcel != NULL) {
        const status_t err = parcel->writeStrongBinder(ibinderForJavaObject(env, object));//1
        if (err != NO_ERROR) {
            signalExceptionForError(env, clazz, err);
        }
    }
}
```

接着查看注释1处ibinderForJavaObject函数：
**frameworks/base/core/jni/android_util_Binder.cpp**

```cpp
sp<IBinder> ibinderForJavaObject(JNIEnv* env, jobject obj)
{
    if (obj == NULL) return NULL;
    if (env->IsInstanceOf(obj, gBinderOffsets.mClass)) {//1
        JavaBBinderHolder* jbh = (JavaBBinderHolder*)
            env->GetLongField(obj, gBinderOffsets.mObject);
        return jbh->get(env, obj);//2
    }
    if (env->IsInstanceOf(obj, gBinderProxyOffsets.mClass)) {
        return getBPNativeData(env, obj)->mObject;
    }

    ALOGW("ibinderForJavaObject: %p is not a Binder object", obj);
    return NULL;
}
```

注释2处，如果obj是Java层的BinderProxy类，则返回BpBinder。
注释1处，如果obj是Java层的Binder类，那么先获取JavaBBinderHolder对象，然后在注释2处调用JavaBBinderHolder的get函数，代码如下所示。
**frameworks/base/core/jni/android_util_Binder.cpp::JavaBBinderHolder**

```cpp
class JavaBBinderHolder
{
public:
    sp<JavaBBinder> get(JNIEnv* env, jobject obj)
    {
        AutoMutex _l(mLock);
        sp<JavaBBinder> b = mBinder.promote();//1
        if (b == NULL) {
            //obj是一个Java层Binder对象
            b = new JavaBBinder(env, obj);//2
            mBinder = b;
            ALOGV("Creating JavaBinder %p (refs %p) for Object %p, weakCount=%" PRId32 "\n",
                 b.get(), b->getWeakRefs(), obj, b->getWeakRefs()->getWeakCount());
        }
        return b;
    }
    sp<JavaBBinder> getExisting()
    {
        AutoMutex _l(mLock);
        return mBinder.promote();
    }
private:
    Mutex           mLock;
    wp<JavaBBinder> mBinder;
};
```

成员变量mBinder是`wp<JavaBBinder>`类型的弱引用，在注释1处得到`sp<JavaBBinder>`类型的强引用b，在注释2处创建JavaBBinder并赋值给b。那么，JavaBBinderHolder的get函数返回的是JavaBBinder。

data.writeStrongBinder(service)在本文中等价于：

```code
data.writeStrongBinder(new JavaBBinder(env，Binder))。
```

讲到这里可以得知ServiceManager.addService()传入的并不是AMS本身，而是JavaBBinder。

### 解析JavaBBinder

接着来分析JavaBBinder，查看它的构造函数：
**frameworks/base/core/jni/android_util_Binder.cpp::JavaBBinderHolder::JavaBBinder**

```cpp
class JavaBBinder : public BBinder
{
public:
    JavaBBinder(JNIEnv* env, jobject /* Java Binder */ c)
        : mVM(jnienv_to_javavm(env)), mObject(env->NewGlobalRef(object))
    {
        ALOGV("Creating JavaBBinder %p\n", this);
        gNumLocalRefsCreated.fetch_add(1, std::memory_order_relaxed);
        gcIfManyNewRefs(env);
    }
...
```

可以发现JavaBBinder继承了BBinder，那么JavaBBinder的作用是什么呢？当Binder驱动得到客户端的请求，紧接着会将响应发送给JavaBBinder，这时会调用JavaBBinder的onTransact函数，代码如下所示。
**frameworks/base/core/jni/android_util_Binder.cpp::JavaBBinderHolder::JavaBBinder**

```cpp
virtual status_t onTransact(
       uint32_t code, const Parcel& data, Parcel* reply, uint32_t flags = 0)
   {                            
       JNIEnv* env = javavm_to_jnienv(mVM);
       ALOGV("onTransact() on %p calling object %p in env %p vm %p\n", this, mObject, env, mVM);
       IPCThreadState* thread_state = IPCThreadState::self();
       const int32_t strict_policy_before = thread_state->getStrictModePolicy();
       jboolean res = env->CallBooleanMethod(mObject, gBinderOffsets.mExecTransact,
           code, reinterpret_cast<jlong>(&data), reinterpret_cast<jlong>(reply), flags);//1

       ...
       return res != JNI_FALSE ? NO_ERROR : UNKNOWN_TRANSACTION;
   }
```

在注释1处会调用Java层Binder的execTransact函数：
**frameworks/base/core/java/android/os/Binder.java**

```cpp
    private boolean execTransact(int code, long dataObj, long replyObj,
            int flags) {
...
        try {
            if (tracingEnabled) {
                Trace.traceBegin(Trace.TRACE_TAG_ALWAYS, getClass().getName() + ":" + code);
            }
            res = onTransact(code, data, reply, flags);//1
        } catch (RemoteException|RuntimeException e) {
           ...
        }
       ...
        return res;
    }
```

关键点是注释1处的onTransact函数，AMS实现了onTransact函数，从而完成业务实现。
从这里可有看出，JavaBBinder并没有实现什么业务，当它接收到请求时，会调用Binder类的execTransact函数，execTransact函数内部又调用了onTransact函数，系统服务会重写onTransact函数来实现自身的业务功能。

### Java Binder架构

Binder架构如下图所示。
[![Qud4yt.png](https://s2.ax1x.com/2019/12/02/Qud4yt.png)](https://s2.ax1x.com/2019/12/02/Qud4yt.png)

Native Binder的部分在此前的文章已经讲过，这里主要来说说Java Binder部分，从图中可以看到：
1.Binder是服务端的代表，JavaBBinder继承BBinder，JavaBBinder通过mObject变量指向Binder。
2.BinderProxy是客户端的代表，ServiceManager的addService等方法会交由ServiceManagerProxy处理。
3.ServiceManagerProxy的成员变量mRemote指向BinderProxy对象，所以ServiceManagerProxy的addService等方法会交由BinderProxy来处理。
4.BinderProxy的成员变量mObject指向BpBinder对象，因此BinderProxy可以通过BpBinder和Binder驱动发送数据。



# Zygote

## Zygote(一)：Android系统的启动过程及Zygote的启动过程

### init进程

作为Android开发者和Android使用者，相信每个人都接触过多种Android设备，不管是哪种品牌、哪种类型的Android设置，在使用之前都要完成开机操作，对于普通用户来说开机只是一个操作过程，但对于开发者有没有想过Android是如何开机的？是如何从断电状态启动到可操作交互的？开发者都听过init进程、孵化器进程已经开发中使用的各种服务，那么它们又是如何启动如何工作的呢？带着这些问题进入本篇文章的主题Androdi系统的启动过程；

- Android系统的启动过程总结

1. 启动电源：按下电源后，程序从固定地方开始加载引导程序到RAM中
2. 引导程序BootLoader：Android系统开始执行引导程序，并同时拉起并运行系统的OS
3. Linux内核启动：当内核启动完成后，首先寻找init.rc配置文件并启动init进程
4. init进程启动：在init进程中完成属性服务的初始、Zygote进程的启动

由上面的程序启动过程知道，程序在执行完引导程序并启动内核后，首先会查找init文件，在init文件中首先执行main（）方法

- init.main（）

```java
......
property_init();//1
......
sigchld_handler_init();//2
......
start_property_service();
......
LoadBootScripts(am, sm);

static void LoadBootScripts(ActionManager& action_manager, ServiceList& service_list) {
    Parser parser = CreateParser(action_manager, service_list);
    parser.ParseConfig("/init.rc"); //3
}
```

在main（）函数中主要执行一下操作：

1. 初始化和启动属性服务
2. 设置进程信号处理
3. 解析init.rc配置文件

#### 属性服务初始化与启动

- 属性服务的初始化

1. 创建非阻塞的Socket
2. 调用listen函数对对属性进行监听
3. 当有数据更新时，init进程会调用handle_property_set_fd函数进行处理

- 处理客户端请求

1. 服务属性接收到客户端请求时调用handle_property_set_fd（）处理数据
2. 根据属性分类处理：普通属性、控制属性

#### 设置进程信号处理

- 僵尸进程：父进程通过Fork创建子进程，当子进程终止之后，如果父进程不知道此时子进程已结束，此时系统中会仍然保存着进程的信息，那么子进程就会成为僵尸进程

1. 僵尸进程危害：系统资源有限，僵尸进程会占用系统资源，当资源耗尽时系统将无法创建新的进程

由僵尸进程的定义知道，出现僵尸进程的原因就是父进程与子进程之间通信中断，signal_handler_init函数就是在父进程中监听子进程的状态，在子进程暂停或终止时会发送SIGCHLD信号，signal_handler_init会接收和处理信号，当接收到子进程终止时及时的释放资源

#### 解析init配置文件

配置文件的解析和处理也是init进程中最主要的部分，安卓中将系统的配置文件保存在init.rc文件中，而Android 8.0之后对init.rc文件浸信会拆分，将每个服务以启动脚本的形式单独存在，然后在init.rc中引入所需要的服务脚本，在启动的时候就可以实现所有服务的启动，这里以接下来要分析的Zygote的启动脚本为例，看看系统是如何定义和处理脚本的

- 启动脚本——init.zygote64.rc

```java
service zygote /system/bin/app_process64 -Xzygote /system/bin --zygote --start-system-server
    class main
    priority -20
    user root
    group root readproc reserved_disk
    socket zygote stream 660 root system
    onrestart write /sys/android_power/request_state wake
    onrestart write /sys/power/state on
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    onrestart restart wificond
    writepid /dev/cpuset/foreground/tasks
```

启动脚本中参数介绍：

1. zygote：创建的进程名称
2. /system/bin/app_process64 ：执行的文件路径
3. class main：表示Zygote的classname为main，后面会根据main查找Zygote服务
4. onrestart：当服务启动时需要重启的服务

上面启动脚本文件的名称为init.zygote64.rc，脚本文件名称表示只支持64系统，不过有的启动过脚本会同时支持32为和64为系统，如init.zygote64_32.rc

```java
service zygote /system/bin/app_process64 -Xzygote /system/bin --zygote --start-system-server --socket-name=zygote
    class main
      ......
    writepid /dev/cpuset/foreground/tasks

service zygote_secondary /system/bin/app_process32 -Xzygote /system/bin --zygote --socket-name=zygote_secondary --enable-lazy-preload
    class main
      .......
    writepid /dev/cpuset/foreground/tasks
```

init.zygote64_32.rc脚本文件中有有两个Zygote服务，一个主模式支持64位的名为zygote进程，另一个辅模式支持32为名为zygote_secondary进程，系统会根据设备的属性决定启动的服务；

- 解析启动脚本

init进程中会使用ServerParse对Service的启动脚本进行解析，最终会针对启动脚本中的每个服务创建对应的实例，然后将所有的对象实例缓存在Service裢表中，在启动服务时就会从此列表中查找对应的服务对象；

### Zygote进程启动

在init.rc文件中引入Zygote的启动脚本，所以在解析init.rc配置文件的服务时，就会将Zygote启动脚本中的服务解析保存在Service的裢表中

```java
import /init.${ro.zygote}.rc
```

- init启动Zygote进程

```java
on nonencrypted
    class_start main
    class_start late_start
```

在解析服务后，会继续init.rc配置文件中的程序，程序执行class_start main，由前面的服务脚本可知classnam为main代表的时Zygote服务，所以此处代表启动Zygote进程，首先会遍历前面保存解析Service的链表，查找classname为main（）的服务，然后执行Service中的start（）方法；

```java
Result<Success> Service::Start() {
pid_t pid = -1;
    if (namespace_flags_) {
        pid = clone(nullptr, nullptr, namespace_flags_ | SIGCHLD, nullptr);
    } else {
        pid = fork(); //1、
    }
    if (pid == 0) { //2、
    if (!ExpandArgsAndExecv(args_)) {
            PLOG(ERROR) << "cannot execve('" << args_[0] << "')";
        }
    }
}

static bool ExpandArgsAndExecv(const std::vector<std::string>& args) {
    return execv(c_strings[0], c_strings.data()) == 0; //3、
}
```

在statr（）方法中，首先判断进程是否已经运行，对未运行的进程通过fork（）创建子进程，创建成功后调用ExpandArgsAndExecv（）方法，在ExpandArgsAndExecv（）中调用执行execv（）后Service进程就被启动并进入Service的main（）方法，Zygote进程对应的程序路径为app_main.cpp,在app_main.cpp的main（）方法中调用runtime.start()启动进程

```java
int main(int argc, char* const argv[]){
        if (strcmp(arg, "--zygote") == 0) { //1
            zygote = true;
            niceName = ZYGOTE_NICE_NAME;
        } else if (strcmp(arg, "--start-system-server") == 0) {
            startSystemServer = true;
        } else if (strcmp(arg, "--application") == 0) {
            application = true;
        } else if (strncmp(arg, "--nice-name=", 12) == 0) {
            niceName.setTo(arg + 12);
        } else if (strncmp(arg, "--", 2) != 0) {
            className.setTo(arg);
            break;
       }
    }
}

if (zygote) {
       runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
   } 
```

在app_main文件的main（）方法中，首先根据进程的名称判断当前是否为Zyote进程，并赋值zygote为true，然后调用runtime.start（）启动进程，注意这里的参数传入的是ZygoteInit类的全路径，这里先猜测下最后是根据全路径反射执行ZygoteInit方法，接着看runtime，这里的runtime指的是AndroidRuntime

- AndroidRuntime.start（）

```java
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote){
 JniInvocation jni_invocation;
    jni_invocation.Init(NULL);
    JNIEnv* env;
    if (startVm(&mJavaVM, &env, zygote) != 0) { //1
        return;
    }
    onVmCreated(env);
    if (startReg(env) < 0) { // 2
        return;
    }

char* slashClassName = toSlashClassName(className != NULL ? className : "");//3
jclass startClass = env->FindClass(slashClassName);// 4
jmethodID startMeth = env->GetStaticMethodID(startClass, "main",
            "([Ljava/lang/String;)V"); // 5
   if (startMeth == NULL) {
            ALOGE("JavaVM unable to find main() in '%s'\n", className);
   } else {
         env->CallStaticVoidMethod(startClass, startMeth, strArray); //6
         if (env->ExceptionCheck())
            threadExitUncaughtException(env);
     }
}
```

在AndroidRuntime的start()方法中，执行了Zygote进程的主要逻辑：

1. 启动Java虚拟机
2. 为Java虚拟机注册JNI方法
3. 通过JNI调用Java层ZygoteInit类中的方法完成进程的启动，此时程序由native进入Java层

- ZygoteInit

```java
public static void main(String argv[]) {
   ZygoteServer zygoteServer = new ZygoteServer();//1
   zygoteServer.registerServerSocketFromEnv(socketName);

   preload(bootTimingsTraceLog);//2

    if (startSystemServer) {
     Runnable r = forkSystemServer(abiList, socketName, zygoteServer);//3
     if (r != null) {
                    r.run();
                    return;
               }
            }
   caller = zygoteServer.runSelectLoop(abiList); //4
}
```

程序进入Java层执行ZygoteInit.main（）方法，在main（）中主要执行：

1. 首先创建并注册Service端的Socket，此Socket用于相应AMS请求创建进程
2. 预加载类和资源
3. 调用forkSystemServer（）启动SystemServer进程
4. 执行zygoteServer.runSelectLoop（）循环等待AMS请求创建新的应用进程

关于ZygoteServer的注册和循环等待AMS创建进程的部分之后在[应用进程的启动过程](https://blog.csdn.net/Alexwll/article/details/100133553)中介绍，这里先来看看startSystemServer（）启动SystemServer进程部分；

#### SystemServer启动过程

SystemServer进程主要用于创建系统服务，如：AMS、WMS、PMS等都由SystemServer启动，由上面知道系统会调用forkSystemServer（）

```java
private static Runnable forkSystemServer(String abiList, String socketName,
       ZygoteServer zygoteServer) {
      String args[] = { //1
            "--setuid=1000",
            "--setgid=1000",
            "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1021,1023,1024,1032,1065,3001,3002,3003,3006,3007,3009,3010",
            "--capabilities=" + capabilities + "," + capabilities,
            "--nice-name=system_server",
            "--runtime-args",
            "--target-sdk-version=" + VMRuntime.SDK_VERSION_CUR_DEVELOPMENT,
            "com.android.server.SystemServer",
        };    
       parsedArgs = new ZygoteConnection.Arguments(args);  //2 
       pid = Zygote.forkSystemServer(  //3
                    parsedArgs.uid, parsedArgs.gid,
                    parsedArgs.gids,
                    parsedArgs.runtimeFlags,
                    null,
                    parsedArgs.permittedCapabilities,
                    parsedArgs.effectiveCapabilities); 
}
      if (pid == 0) { 
            return handleSystemServerProcess(parsedArgs);//4
        }
```

在forkSystemServer（）方法中首先将启动参数封装在数组中，然后使用数组创建ZygoteConnection.Arguments对象，最后调用Zygote.forkSystemServer方法fok SystemServer进程，在forkSystemServer（）中调用nativeForkSystemServer（）方法实现进程创建，fork进程成功后调用handleSystemServerProcess（）处理进程中的工作；

```java
private static Runnable handleSystemServerProcess(ZygoteConnection.Arguments parsedArgs) {
            ClassLoader cl = null;
            if (systemServerClasspath != null) {
               cl = createPathClassLoader(systemServerClasspath, parsedArgs.targetSdkVersion);
               Thread.currentThread().setContextClassLoader(cl);
            }
            return ZygoteInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs, cl);
    }
```

在handleSystemServerProcess（）中首先创建PathClassLoader对象，然后调用ZygoteInit.zygoteInit（）方法

- ZygoteInit.zygoteInit

```java
 public static final Runnable zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader) {
        ZygoteInit.nativeZygoteInit(); //1、
        return RuntimeInit.applicationInit(targetSdkVersion, argv, classLoader);//2、
}
```

在zygoteInit（）中调用nativeZygoteInit（）方法，从名字上看出调用的是native方法，在内部通过JNI方法完成Binder线程池的创建，在方法的最后调用RuntimeInit.applicationInit（）方法传入ClassLoader，applicationInit（）方法相对比较特殊，下面一起看下源码

```java
protected static Runnable applicationInit(int targetSdkVersion, String[] argv,
            ClassLoader classLoader) {
        final Arguments args = new Arguments(argv);
        return findStaticMain(args.startClass, args.startArgs, classLoader);//1
    }

 protected static Runnable findStaticMain(String className, String[] argv,
            ClassLoader classLoader) {
        Class<?> cl;

        try {
            cl = Class.forName(className, true, classLoader);//2
        } catch (ClassNotFoundException ex) {
            throw new RuntimeException(
                    "Missing class when invoking static main " + className,
                    ex);
        }

        Method m;
        try {
            m = cl.getMethod("main", new Class[] { String[].class });//3
        } catch (NoSuchMethodException ex) {
            throw new RuntimeException(
                    "Missing static main on " + className, ex);
        } catch (SecurityException ex) {
            throw new RuntimeException(
                    "Problem getting static main on " + className, ex);
        }

        int modifiers = m.getModifiers();
        if (! (Modifier.isStatic(modifiers) && Modifier.isPublic(modifiers))) {
           throw new RuntimeException(
                    "Main method is not public and static on " + className);
        }
        return new MethodAndArgsCaller(m, argv); //4
    }
```

applicationInit（）中调用了findStaticMain（）方法，findStaticMain（）并没有直接调用SystemServer.main（）方法，而是通过反射获取SystemServer的Class，然后获取main（）方法，并将main方法和参数封装在MethodAndArgsCaller中，这一点Android P中做了修改，之前的版本中将反射main方法封装在异常中抛出，然后捕捉异常执行，Android P返回了MethodAndArgsCaller对象，MethodAndArgsCaller继承实现Runnable，其实在前面ZygoteInit类中，有一段代码如下

```java
if (startSystemServer) {
                Runnable r = forkSystemServer(abiList, socketName, zygoteServer);
                if (r != null) {
                    r.run();
                    return;
                }
            }
```

整个SystemServer继承的启动是从调用forkSystemServer（）开始的，forkSystemServer返回了Runnable对象，这里的Runnable对象就是上面创建的MethodAndArgsCaller对象，然后调用run（）方法执行MethodAndArgsCaller对象；

```java
static class MethodAndArgsCaller implements Runnable {
        private final Method mMethod;
        private final String[] mArgs;
        public MethodAndArgsCaller(Method method, String[] args) {
            mMethod = method;
            mArgs = args;
        }
        public void run() {
            try {
                mMethod.invoke(null, new Object[] { mArgs });
            } catch (IllegalAccessException ex) {
        }
    }
```

在MethodAndArgsCaller中保存了反射获取的Method，这里的Method就是SystemServer.main（）方法，在run方法中调用method.invoke（）执行main方法，反射执行之后程序进入SystemServer.main（）,main中创建SystemServer对象，并执行run（）方法；

```java
public static void main(String[] args) {
        new SystemServer().run();
    }
```

- SystemServer.run（）

```java
mSystemServiceManager = new SystemServiceManager(mSystemContext); //1
// Start services.
        try {
            startBootstrapServices();//2
            startCoreServices();//3
            startOtherServices();//4
            SystemServerInitThreadPool.shutdown();
        } catch (Throwable ex) {
            throw ex;
        } 
```

main（）方法中直接调用SystemServer.run（）方法，在run（）方法中，首先创建系统的SystemServiceManager对象，然后依次调用方法启动引导服务、启动核心服务、启动其他服务

- 启动服务过程

```java
mSystemServiceManager.startService(PowerManagerService.class);

  public SystemService startService(String className) {
        final Class<SystemService> serviceClass;
        try {
            serviceClass = (Class<SystemService>)Class.forName(className);
        } catch (ClassNotFoundException ex) {
        }
        return startService(serviceClass);
    }
```

系统调用SystemServerManager.startService（）传入对应的服务，在startService（）中根据传入的类名加载类文件，然后执行startService(serviceClass)方法，startService中使用加载的Class获取构造函数并创建对象，然后调用startService(service)；

```java
// startService中
Constructor<T> constructor = serviceClass.getConstructor(Context.class);
service = constructor.newInstance(mContext);
startService(service);
return service;

public void startService(@NonNull final SystemService service) {
        mServices.add(service); //1
        try {
            service.onStart();//2
        } catch (RuntimeException ex) {
    }
```

在startService（）中，先将service注册到mServices中，然后调用service.onStart（）方法启动服务；

Android系统指执行到此后会使用ServiceManager启动一系列的服务，这些都是位置Android系统运行的核心功能，之后有时间会逐个分析，本次Android系统的启动过程及Zygote的启动过程就介绍完毕了！

## Zygote（二）：应用进程的启动过程

程序的启动是从进程启动开始的，换句话说只有程序进程启动后，程序才会加载和执行，在AMS启动程序时首先会判断当前进程是否启动，对未启动的进程会发送请求，Zygote在收到请求后创建新的进程；

### Zygote监听客户端请求

由Zygote(一)—Android系统的启动过程知道，系统的启动会执行到ZygoteInit.main()方法；

```java
public static void main(String argv[]) {
   ZygoteServer zygoteServer = new ZygoteServer();//1
   zygoteServer.registerServerSocketFromEnv(socketName);
   caller = zygoteServer.runSelectLoop(abiList); //2
}
```

在main中创建并注册了服务端的Socket，然后执行runSelectLoop（）循环等待AMS的请求创建进程，前一篇文章中跳过了这个部分，本片文章来分析下系统如何实现Socket通信的

- registerServerSocketFromEnv（）

```java
void registerServerSocketFromEnv(String socketName) {
        if (mServerSocket == null) {
            int fileDesc;
            final String fullSocketName = ANDROID_SOCKET_PREFIX + socketName;//1
            try {
                String env = System.getenv(fullSocketName);//2
                fileDesc = Integer.parseInt(env);
            } 
            try {
                FileDescriptor fd = new FileDescriptor();//3
                fd.setInt$(fileDesc);
                mServerSocket = new LocalServerSocket(fd);//4
                mCloseSocketFd = true;
            } catch (IOException ex) {
        }
    }
```

在registerServerSocketFromEnv中（）出入的参数为进程名称，由上一篇文章知道传入的为ztgote，然后使用ANDROID_SOCKET_PREFIX拼接Socket名称，最终的名称为ANDROID_SOCKET_zygote，然后将fullSocketName转换为环境变量的值，注释3处创建文件描述符，然后根据文件描述符fd创建LocalServerSocket对象；

```java
public LocalServerSocket(FileDescriptor fd) throws IOException
    {
        impl = new LocalSocketImpl(fd);
        impl.listen(LISTEN_BACKLOG);
        localAddress = impl.getSockAddress();
    }
```

LocalService使用select监听文件描述符，当文件描述符上出现新内容时会自动触发中断，然后中断处理函数中再次读取文件描述福符上的数据，从而获取信息；

- zygoteServer.runSelectLoop（）

```java
    Runnable runSelectLoop(String abiList) {
        ArrayList<FileDescriptor> fds = new ArrayList<FileDescriptor>();
        ArrayList<ZygoteConnection> peers = new ArrayList<ZygoteConnection>();
177        fds.add(mServerSocket.getFileDescriptor());//1
178        peers.add(null);
179
180        while (true) { //2
181            StructPollfd[] pollFds = new StructPollfd[fds.size()];
182            for (int i = 0; i < pollFds.length; ++i) {
183                pollFds[i] = new StructPollfd();
184                pollFds[i].fd = fds.get(i);
185                pollFds[i].events = (short) POLLIN;
186            }
187            try {
188                Os.poll(pollFds, -1);
189            } catch (ErrnoException ex) {
190                throw new RuntimeException("poll failed", ex);
191            }
192            for (int i = pollFds.length - 1; i >= 0; --i) {
193                if ((pollFds[i].revents & POLLIN) == 0) {
194                    continue;
195                }
197                if (i == 0) { // 3
198                    ZygoteConnection newPeer = acceptCommandPeer(abiList);
199                    peers.add(newPeer);
200                    fds.add(newPeer.getFileDesciptor());
201                } else {
202                    try {
203                        ZygoteConnection connection = peers.get(i);
204                        final Runnable command = connection.processOneCommand(this); // 4
205
206                        if (mIsForkChild) {
213                            return command;
214                        } else {
223                            if (connection.isClosedByPeer()) {
224                                connection.closeSocket();
225                                peers.remove(i);
226                                fds.remove(i);
227                            }
228                        }
229                    } catch (Exception e) {
230                        if (!mIsForkChild) {
241                            ZygoteConnection conn = peers.remove(i);
242                            conn.closeSocket();
244                            fds.remove(i);
245                        } else {
250                            throw e;
251                        }
252                    } finally {
256                        mIsForkChild = false;
257                    }
258                }
259            }
260        }
261    }
```

在runSelectLoop方法中，首先获取注册Socket的文件描述符fd，并将其保存在fds集合中，然后开启循环监听AMS的请求，在注释3处判断i ==0，若i=0表示Socket没有可用的链接需要重连，否则表示接收到AMS的请求信号，调用connection.processOneCommand(this)创建新的进程，创建完成后清除对应的peers集合和fds集合；

### AMS发送创建进程请求

AMS会检查目标程序进程，如果进程未启动则调用startProcessLocked（）方法启动进程

```java
   int uid = app.uid; //1
   ......
if (ArrayUtils.isEmpty(permGids)) { //2
           gids = new int[3];
        } else {
            gids = new int[permGids.length + 3];
            System.arraycopy(permGids, 0, gids, 3, permGids.length);
        }
          gids[0] = UserHandle.getSharedAppGid(UserHandle.getAppId(uid));
          gids[1] = UserHandle.getCacheAppGid(UserHandle.getAppId(uid));
          gids[2] = UserHandle.getUserGid(UserHandle.getUserId(uid));
.....
final String entryPoint = "android.app.ActivityThread"; //3
return startProcessLocked(hostingType, hostingNameStr, entryPoint, app, uid, gids,runtimeFlags, mountExternal, seInfo, requiredAbi, instructionSet, invokeWith,startTime); //4
```

startProcessLocked执行一下逻辑：

1. 获取当前程序的uid
2. 对用户组gids创建并赋值
3. entryPoint赋值为ActivityThread的路径，ActivityThread就是进程启动时要初始化的类
4. 调用startProcessLocked（）启动进程

在startProcessLocked（）中又调用startProcess()，startProcess（）中调用Process.start()方法，start中调用ZygoteProcess.startViaZygote()启动进程，

- ZygoteProcess.startViaZygote()

```java
private Process.ProcessStartResult startViaZygote(final String processClass,
   ......String[] extraArgs)throws ZygoteStartFailedEx {
   ArrayList<String> argsForZygote = new ArrayList<String>();
   argsForZygote.add("--runtime-args");
   argsForZygote.add("--setuid=" + uid);
   argsForZygote.add("--setgid=" + gid);
   argsForZygote.add("--runtime-flags=" + runtimeFlags);

synchronized(mLock) {
return zygoteSendArgsAndGetResult(openZygoteSocketIfNeeded(abi), argsForZygote);
}
```

在startViaZygote中首先创建ArrayList集合，然后添加创建进程的启动参数，最后调用zygoteSendArgsAndGetResult（）执行启动进程，在zygoteSendArgsAndGetResult的第一个参数中首先调用了openZygoteSocketIfNeeded（）方法，它的作用其实就是连接Zygote 中服务端的Socket

```java
private ZygoteState openZygoteSocketIfNeeded(String abi) throws ZygoteStartFailedEx {
        if (primaryZygoteState == null || primaryZygoteState.isClosed()) {
            try {
                primaryZygoteState = ZygoteState.connect(mSocket); //1
            } catch (IOException ioe) {
                throw new ZygoteStartFailedEx("Error connecting to primary zygote", ioe);
            }
            maybeSetApiBlacklistExemptions(primaryZygoteState, false);
           maybeSetHiddenApiAccessLogSampleRate(primaryZygoteState);
        }
        if (primaryZygoteState.matches(abi)) { //2
            return primaryZygoteState;
        }
        if (secondaryZygoteState == null || secondaryZygoteState.isClosed()) {
            try {
                secondaryZygoteState = ZygoteState.connect(mSecondarySocket);//3
            } 
           maybeSetApiBlacklistExemptions(secondaryZygoteState, false);
           maybeSetHiddenApiAccessLogSampleRate(secondaryZygoteState);
        }
        if (secondaryZygoteState.matches(abi)) { //4
            return secondaryZygoteState;
        }
 }
```

在openZygoteSocketIfNeeded（）中ZygoteState.connect（）连接Zygote进程中的Socket，连接成功后返回连接的主模式，用此主模式和传入的abi比较是否匹配，如果匹配则直接返回ZygoteState，否则连接zygote辅模式；

- zygoteSendArgsAndGetResult（）

```java
 private static Process.ProcessStartResult zygoteSendArgsAndGetResult(
281            ZygoteState zygoteState, ArrayList<String> args)
282            throws ZygoteStartFailedEx {
283        try {
286            int sz = args.size();
287            for (int i = 0; i < sz; i++) {
288                if (args.get(i).indexOf('\n') >= 0) {
289                    throw new ZygoteStartFailedEx("embedded newlines not allowed");
290                }
291            }
303            final BufferedWriter writer = zygoteState.writer;
304            final DataInputStream inputStream = zygoteState.inputStream;
306            writer.write(Integer.toString(args.size()));
307            writer.newLine();
309            for (int i = 0; i < sz; i++) {
310                String arg = args.get(i);
311                writer.write(arg);
312                writer.newLine();
313            }
315            writer.flush();
323            result.pid = inputStream.readInt();
324            result.usingWrapper = inputStream.readBoolean();
326            if (result.pid < 0) {
327                throw new ZygoteStartFailedEx("fork() failed");
328            }
329            return result;
330        } catch (IOException ex) {
331            zygoteState.close();
332            throw new ZygoteStartFailedEx(ex);
333        }
334    }
```

在zygoteSendArgsAndGetResult中获取连接Socket返回的ZygoteState，利用ZygoteState内部的BufferedWriter将请求参数写入Zygote进程中；

### Zygote接收信息并创建进程

由第一部分知道，在AMS发送请求后zygote进程会接收请求，并调用ZygoteConnection.processOneCommand()方法处理请求，在ZygoteInit.main（）方法中有以下代码

```java
  final Runnable caller;
  caller = zygoteServer.runSelectLoop(abiList);
  if (caller != null) {
      caller.run();
  }
```

在runSelectLoop（）中接收到AMS请求信息后，然后执行处理并返回Runnable对象并执行run（）方法，

```java
  Runnable processOneCommand(ZygoteServer zygoteServer) {
   String args[];
   Arguments parsedArgs = null;
        FileDescriptor[] descriptors;
        try {
            args = readArgumentList(); //1
            descriptors = mSocket.getAncillaryFileDescriptors();
        } 
         parsedArgs = new Arguments(args); //2
  pid = Zygote.forkAndSpecialize(parsedArgs.uid, parsedArgs.gid, parsedArgs.gids,parsedArgs.runtimeFlags, rlimits, parsedArgs.mountExternal, parsedArgs.seInfo,parsedArgs.niceName, fdsToClose, fdsToIgnore, parsedArgs.startChildZygote,parsedArgs.instructionSet, parsedArgs.appDataDir);
 if (pid == 0) {
                zygoteServer.setForkChild();
                zygoteServer.closeServerSocket();
                IoUtils.closeQuietly(serverPipeFd);
                serverPipeFd = null;
                return handleChildProc(parsedArgs, descriptors, childPipeFd,
                        parsedArgs.startChildZygote); //3
            } else {
                IoUtils.closeQuietly(childPipeFd);
                childPipeFd = null;
                handleParentProc(pid, descriptors, serverPipeFd);
                return null;
            }

  }
```

在processOneCommand（）中首先调用readArgumentList读取参数数组，然后将数组封装在Arguments对象中，调用Zygote.forkAndSpecialize（）方法fork子进程，子进程创建成功后调用handleChildProc（）初始化进程

```java
private Runnable handleChildProc(Arguments parsedArgs, FileDescriptor[] descriptors,FileDescriptor pipeFd, boolean isZygote) {
        if (parsedArgs.niceName != null) {
            Process.setArgV0(parsedArgs.niceName);
        }
            if (!isZygote) {
                return ZygoteInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs,null /* classLoader */);
            } else {
                return ZygoteInit.childZygoteInit(parsedArgs.targetSdkVersion,
                        parsedArgs.remainingArgs, null /* classLoader */);
            }
        }
    }
```

在handleChildProc调用ZygoteInit.zygoteInit（），关于ZygoteInit.zygoteInit（）方法的内容参考[Android进阶知识树——Android系统的启动过程](https://blog.csdn.net/Alexwll/article/details/100133524)，其最终会使用反射调用ActivityThread.main（）方法，程序进入进程初始化，关于ActivityThread中的操作这里不做分析，相信Android开发者应该了解；

### 启动Binder线程池

本篇文章和上一篇[Android进阶知识树——Android系统的启动过程](https://blog.csdn.net/Alexwll/article/details/100133524)中都提到Binder线程池的创建，但都没有详细介绍，这里补充一下，程序在启动进程后会调用ZygoteInit.zygoteInit()方法，zygoteInit中调用本地方法nativeZygoteInit（），在ZygoteInit中声明了nativeZygoteInit方法；

```java
public static final Runnable zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader) {
       RuntimeInit.commonInit();
       ZygoteInit.nativeZygoteInit();//
       return RuntimeInit.applicationInit(targetSdkVersion, argv, classLoader);
    }
    
private static final native void nativeZygoteInit();
```

很明显nativeZygoteInit（）是JNI方法（关于JNI见另一篇文章[Android进阶知识树——JNI和So库开发](https://blog.csdn.net/Alexwll/article/details/99703403)），他在AndroidRuntime中完成方法动态注册，nativeZygoteInit中对应c文件中com_android_internal_os_ZygoteInit_nativeZygoteInit（）方法

```java
int register_com_android_internal_os_ZygoteInit_nativeZygoteInit(JNIEnv* env)
{
    const JNINativeMethod methods[] = {
        { "nativeZygoteInit", "()V",
            (void*) com_android_internal_os_ZygoteInit_nativeZygoteInit },
    };
    return jniRegisterNativeMethods(env, "com/android/internal/os/ZygoteInit",
        methods, NELEM(methods));
}

static void com_android_internal_os_ZygoteInit_nativeZygoteInit(JNIEnv* env, jobject clazz)
{
    gCurRuntime->onZygoteInit();
}
```

在com_android_internal_os_ZygoteInit_nativeZygoteInit（）方法中调用gCurRuntime.onZygoteInit（），这里的gCurRuntime是AppRuntime对象，它继承AndroidRuntime，在AndroidRuntime的构造函数中被初始化，AppRuntime类在app_main中实现

```java
class AppRuntime : public AndroidRuntime
{
virtual void onZygoteInit()    {
       sp<ProcessState> proc = ProcessState::self();
        proc->startThreadPool();
    }
}
```

在onZygoteInit（）中调用ProcessState类的startThreadPool（），startThreadPool（）中调用

```java
void ProcessState::spawnPooledThread(bool isMain){
    if (mThreadPoolStarted) {
        String8 name = makeBinderThreadName();
        sp<Thread> t = new PoolThread(isMain);
        t->run(name.string());
    }
}
```

在spawnPooledThread（）中使用makeBinderThreadName（）生成线程名称，然后创建PoolThread线程并执行线程

```java
class PoolThread : public Thread{
protected:
    virtual bool threadLoop()
    {
        IPCThreadState::self()->joinThreadPool(mIsMain); 
        return false;
    }
    const bool mIsMain;
};
```

PoolThread继承Thread类，启动PoolThread对象就创建了一个新的线程，在PoolThread的threadLoop（）方法中调用IPCThreadState的joinThreadPool（）方法将创建的线程加入Binder线程吃中，那么新创建的应用进程就支持Binder进程通行了；

总结一下整个进程启动的过程：

1. AMS判断当前进程是否启动，对未启动的进程发送请求
2. 首先根据程序的pid设置并赋值用户组gids
3. 将entryPoint赋值为ActivityThread的路径，然后开始执行进程启动
4. 在与zygote交互中，首先根据设置的abi连接zygote进程中的socket，并判断是匹配主模式还是辅模式，连接成功后返回zygoteState对象
5. 使用zygoteState将请求参数写入zygote的Socket中，zygote进程中读取请求的信息保存在数组中
6. 使用参数数组创建Argument对象，并fork出程序进程，从而启动程序进程
7. 进程启动后调用ZygotezInit.zygoteInit（）方法，内部初始化Binder线程池实现进程通信，然后反射获取ActivityThread.main（）方法，完成新进程的初始化



# AMS

## AMS源码分析(一)Activity生命周期管理

AMS(ActivityManagerService)是Activity管理的核心组件，提供了Activity的启动、生命周期管理、栈管理等功能，熟悉AMS会对我们认识Activity的工作原理有很大的帮助。当前比较成熟的插件化技术，也是通过对Activity启动流程中的重要组件（如Instrumentation或主线程Handler等）进行Hook完成的，掌握AMS对我们学习插件化技术也有很大的帮助

AMS中内容很多，对它的分析也不可能面面俱到，我期望从Activity的启动、Activity消息回传（onActivityResult）、Activity栈管理、AMS与WMS和PMS的协同工作方面入手，希望本系列文章完成后可以对AMS有一个更新的认识

### Activity的生命周期

#### 一个Activity从启动到销毁所经历的周期

onCreate -> onStart -> onResume -> onPause -> onStop -> onDestory

这属于Android最基础的知识，就不再赘述了

#### 从一个Activity启动另一个Activity的生命周期

现在有两个Activity,分别是AActivity和BActivity,如果从AActivity跳转到BActivity,那么它们的生命周期会是下面这个样子：

AActivity#onPause -> BActivity#onCreate -> BActivity#onStart -> BActivity#onResume -> AActivity#onStop

如下图:

![img](https:////upload-images.jianshu.io/upload_images/3112838-c98340b7602caebe.image?imageMogr2/auto-orient/strip|imageView2/2/w/248/format/webp)

而从BActivity返回时，它们的生命周期是这样的：

BActivity#onPause -> AActivity#onStart -> AActivity#onResume -> BActivity#onStop -> BActivity#onDestroy

![img](https:////upload-images.jianshu.io/upload_images/3112838-e0000f3c12fd415e.image?imageMogr2/auto-orient/strip|imageView2/2/w/317/format/webp)

为什么是这样呢？下面我们从源码的角度去分析一下

### 源码分析

在做源码分析之前，我们要先理解两个概念

- Android Binder IPC机制
- Hander idleHandler机制

#### Binder

Binder是Android跨进程通信的核心，但不是本文的重点，这里不多做讲解，感兴趣的朋友可以看一下我之前写过的Binder系列文章：[Binder系列](https://www.jianshu.com/p/275bc9a53342)

在本篇文章中，我们只需要知道每当看到类似`mRemote.transact`
 这种调用时，既是跨进程通信的开始

#### IdleHandler

当消息队列空闲时会执行IdelHandler的queueIdle()方法，该方法返回一个boolean值，
 如果为false则执行完毕之后移除这条消息，
 如果为true则保留，等到下次空闲时会再次执行

也就是说，IdleHandler是Handler机制中MessageQueue空闲时才会执行的一种特殊Handler

#### Activity在AMS中的标识

每一个Activity实例在AMS中都会对应一个ActivityRecord,ActivityRecord对象是在Activity启动过程中创建的，每个ActivityRecord中又会有一个ProcessRecord标记Activity所在的进程

而在APP进程中，在Activity启动时，也会创建一个ActivityClientRecord,与AMS中的ActivityRecord遥相呼应。

Activity、ActivityClientRecord、ActivityRecord互相联系的纽带是一个`token`对象，它继承于`IApplicationToken.Stub`,这是一个Binder对象，在Activity、ActivityClientRecord、ActivityRecord中均有一份。

Activity、ActivityClientRecord、ActivityRecord之间的关系图大致如下---注意区分所在进程：

![img](https:////upload-images.jianshu.io/upload_images/3112838-0ff114d27e1a8d37.image?imageMogr2/auto-orient/strip|imageView2/2/w/623/format/webp)

#### 启动过程时序图

黄色为APP进程，蓝色为AMS进程

![img](https:////upload-images.jianshu.io/upload_images/3112838-f98e18517de9cb28.image?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

#### 从AActivity跳转BActivity的生命周期分析

AMS的逻辑太多，本文只针对Activity声明周期相关的代码，其他相关逻辑后面几篇文章继续分析

从Activity开始



```kotlin
public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
                                   @Nullable Bundle options) {
    if (mParent == null) {
        options = transferSpringboardActivityOptions(options);
        Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                        this, mMainThread.getApplicationThread(), mToken, this,
                        intent, requestCode, options);
       
    } else {

    }
}
```

Instrumentation#execStartActivity方法：



```csharp
public ActivityResult execStartActivity(
        Context who, IBinder contextThread, IBinder token, Activity target,
        Intent intent, int requestCode, Bundle options) {
    ...
    try {
        intent.migrateExtraStreamToClipData();
        intent.prepareToLeaveProcess(who);
        int result = ActivityManager.getService()
            .startActivity(whoThread, who.getBasePackageName(), intent,
                    intent.resolveTypeIfNeeded(who.getContentResolver()),
                    token, target != null ? target.mEmbeddedID : null,
                    requestCode, 0, null, options);
        checkStartActivityResult(result, intent);
    } catch (RemoteException e) {
        throw new RuntimeException("Failure from system", e);
    }
    return null;
}
```

重要入参及含义：

- who: 启动源Activity
- contextThread ： 宿主Activity的ApplicationThread
- token：Activity的身份标识 -> Activity#mToken -> ActivityClientRecord#token -> ActivityRecord#appToken -> IApplicationToken.Stub
- target: 哪个activity调用的start方法，因此这个Activity会接收任何的返回结果，如果start调用不是从Activity中发起的则有可能为null
- intent: 启动Activity时的intent
- requestCoode: 启动Activity时的requestCode
- options: 启动Activity的选项

注意token参数，本文将重点追踪它的变化，这个参数非常重要，后面会有很多地方用到

继续走逻辑,在这里进行了IPC调用，从APP进程转入AMS进程执行，`ActivityManager.getService() .startActivity`调用在AMS进程对应的是`ActivityManagerService#startActivity`，我们看一下

ActivityManagerService#startActivity 经过一系列的调用后，会走到 ActivityStarter#startActivityMayWait



```dart
final int startActivityMayWait(IApplicationThread caller, int callingUid,
        int requestRealCallingPid, int requestRealCallingUid,
        String callingPackage, Intent intent, String resolvedType,
        IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
        IBinder resultTo, String resultWho, int requestCode, int startFlags,
        ProfilerInfo profilerInfo, WaitResult outResult,
        Configuration globalConfig, Bundle bOptions, boolean ignoreTargetSecurity, int userId,
        TaskRecord inTask, String reason) {

    ...

    // Collect information about the target of the Intent.
    // zhangyulong 解析Intent中的Activity信息
    ActivityInfo aInfo = mSupervisor.resolveActivity(intent, rInfo, startFlags, profilerInfo);

    ...

        // zhangyulong 声明一个ActivityRecord数组
        final ActivityRecord[] outRecord = new ActivityRecord[1];
        int res = startActivityLocked(caller, intent, ephemeralIntent, resolvedType,
                aInfo, rInfo, voiceSession, voiceInteractor,
                resultTo, resultWho, requestCode, callingPid,
                callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                options, ignoreTargetSecurity, componentSpecified, outRecord, inTask,
                reason);

        Binder.restoreCallingIdentity(origId);

       ...
        return res;
    }
}
```

这里进行的工作是解析Intent信息，并继续调用`startActivityLocked`，注意入参中的`resultTo`就是从App进程传入的token，依旧是透传给`startActivityLocked`



```dart
int startActivityLocked(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
        String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
        IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
        IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
        String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
        ActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
        ActivityRecord[] outActivity, TaskRecord inTask, String reason) {

    if (TextUtils.isEmpty(reason)) {
        throw new IllegalArgumentException("Need to specify a reason.");
    }
    mLastStartReason = reason;
    mLastStartActivityTimeMs = System.currentTimeMillis();
    mLastStartActivityRecord[0] = null;

    mLastStartActivityResult = startActivity(caller, intent, ephemeralIntent, resolvedType,
            aInfo, rInfo, voiceSession, voiceInteractor, resultTo, resultWho, requestCode,
            callingPid, callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
            options, ignoreTargetSecurity, componentSpecified, mLastStartActivityRecord,
            inTask);

    if (outActivity != null) {
        // mLastStartActivityRecord[0] is set in the call to startActivity above.
        outActivity[0] = mLastStartActivityRecord[0];
    }

    // Aborted results are treated as successes externally, but we must track them internally.
    return mLastStartActivityResult != START_ABORTED ? mLastStartActivityResult : START_SUCCESS;
}
```

这个方法也没做太多事情，将参数透传给`startActivity`继续执行



```java
private int startActivity(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
      String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
        IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
        IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
        String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
        ActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
        ActivityRecord[] outActivity, TaskRecord inTask) {
    int err = ActivityManager.START_SUCCESS;
    // Pull the optional Ephemeral Installer-only bundle out of the options early.
    final Bundle verificationBundle
            = options != null ? options.popAppVerificationBundle() : null;

    ProcessRecord callerApp = null;
    if (caller != null) {
        callerApp = mService.getRecordForAppLocked(caller);
        if (callerApp != null) {
            callingPid = callerApp.pid;
            callingUid = callerApp.info.uid;
        } else {
            Slog.w(TAG, "Unable to find app for caller " + caller
                    + " (pid=" + callingPid + ") when starting: "
                    + intent.toString());
            err = ActivityManager.START_PERMISSION_DENIED;
        }
    }

    final int userId = aInfo != null ? UserHandle.getUserId(aInfo.applicationInfo.uid) : 0;

    if (err == ActivityManager.START_SUCCESS) {
        Slog.i(TAG, "START u" + userId + " {" + intent.toShortString(true, true, true, false)
                + "} from uid " + callingUid);
    }

    ActivityRecord sourceRecord = null;
    ActivityRecord resultRecord = null;
    if (resultTo != null) {
        // 通过resultTo在寻找记录在AMS中记录的ActivityRecord，这个ActivityRecord和启动Activity的源Activity对应
        sourceRecord = mSupervisor.isInAnyStackLocked(resultTo);
        if (DEBUG_RESULTS) Slog.v(TAG_RESULTS,
                "Will send result to " + resultTo + " " + sourceRecord);
        if (sourceRecord != null) {
            if (requestCode >= 0 && !sourceRecord.finishing) {
                resultRecord = sourceRecord;
            }
        }
    }

    final int launchFlags = intent.getFlags();

   // 新建一条目标Activity的ActivityRecord，这条ActivityRecord和即将打开的Activity实例对应
    ActivityRecord r = new ActivityRecord(mService, callerApp, callingPid, callingUid,
            callingPackage, intent, resolvedType, aInfo, mService.getGlobalConfiguration(),
            resultRecord, resultWho, requestCode, componentSpecified, voiceSession != null,
            mSupervisor, options, sourceRecord);
    if (outActivity != null) {
        outActivity[0] = r;
    }
    return startActivity(r, sourceRecord, voiceSession, voiceInteractor, startFlags, true,
            options, inTask, outActivity);
}
```

记的我们在分析源码之前所画的ActivityRecord、ActivityClientRecord、Activity之间的关系图么？这里就派上用场了，入参`resultTo`就是启动Activity时传递的token,那么，在AMS中，必然会有一条ActivityRecord与之对应，通过`mSupervisor.isInAnyStackLocked(resultTo)`找到与AActivity对应的ActivityRecord。

这个方法还有一个重要的任务就是新建一条ActivityRecord记录，这条记录和即将打开的Activity，即BActivity对应。完成这些工作后，继续执行`startActivity`。

`startActivity`继续调用了`startActivityUnchecked`,这个方法最重要的功能是对Acitivity栈和Activity启动模式进行处理，篇幅很长，逻辑复杂，但是不是本篇的重点，所以我们后面的文章再着重分析，现在只关注Activity启动流程相关的部分



```java
private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
            IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
            int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
            ActivityRecord[] outActivity) {
      
            ...

    if (mDoResume) {
        final ActivityRecord topTaskActivity =
                mStartActivity.getTask().topRunningActivityLocked();
        if (!mTargetStack.isFocusable()
                || (topTaskActivity != null && topTaskActivity.mTaskOverlay
                && mStartActivity != topTaskActivity)) {
            mTargetStack.ensureActivitiesVisibleLocked(null, 0, !PRESERVE_WINDOWS);
            mWindowManager.executeAppTransition();
        } else {
            if (mTargetStack.isFocusable() && !mSupervisor.isFocusedStack(mTargetStack)) {
                mTargetStack.moveToFront("startActivityUnchecked");
            }
  
            mSupervisor.resumeFocusedStackTopActivityLocked(mTargetStack, mStartActivity,
                    mOptions);
        }
    } else {
        mTargetStack.addRecentActivityLocked(mStartActivity);
    }
    ...
    return START_SUCCESS;
}
```

##### AActivity#onPause

注意看`mSupervisor.resumeFocusedStackTopActivityLocked(mTargetStack, mStartActivity, mOptions)`，这里，mStartActivity是即将启动的ActivityRecord，mTargetStack是在本方法中计算得到的它所在的ActivityStack，继续看`resumeFocusedStackTopActivityLocked`

还记得在开篇时讲到的从AActivity跳转到BActivity的生命周期过程么？不记得没关系，复习一下先：
 AActivity#onPause -> BActivity#onCreate -> BActivity#onStart -> BActivity#onResume -> AActivity#onStop,
 在`resumeFocusedStackTopActivityLocked`这个方法我们将接触到这个过程中的第一个声明周期方法，即`AActivity#onPause`,而且本身这个方法的调用过程也特别复杂，需要重点解析一下，先看一下代码：



```java
private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
    if (!mService.mBooting && !mService.mBooted) {
        // Not ready yet!
        return false;
    }

    // Find the next top-most activity to resume in this stack that is not finishing and is
    // focusable. If it is not focusable, we will fall into the case below to resume the
    // top activity in the next focusable task.
    // 获取栈顶正在聚焦的Activity
    final ActivityRecord next = topRunningActivityLocked(true /* focusableOnly */);

    final boolean hasRunningActivity = next != null;

    // TODO: Maybe this entire condition can get removed?
    if (hasRunningActivity && getDisplay() == null) {
        // zhangyulong 异常情况，有正在运行的activity但getDisplay为空
        return false;
    }

    mStackSupervisor.cancelInitializingActivities();

    // Remember how we'll process this pause/resume situation, and ensure
    // that the state is reset however we wind up proceeding.
    final boolean userLeaving = mStackSupervisor.mUserLeaving;
    mStackSupervisor.mUserLeaving = false;

    if (!hasRunningActivity) {
        // There are no activities left in the stack, let's look somewhere else.
        // zhangyulong  task 退干净了，恢复其他的task
        return resumeTopActivityInNextFocusableStack(prev, options, "noMoreActivities");
    }

    next.delayedResume = false;

    ...
    // Activity栈中有需要Pause操作的Activity
    boolean pausing = mStackSupervisor.pauseBackStacks(userLeaving, next, false);
    if (mResumedActivity != null) {
        // 暂停当前正在显示的Activity
        pausing |= startPausingLocked(userLeaving, false, next, false);
    }
    if (pausing && !resumeWhilePausing) {
        if (next.app != null && next.app.thread != null) {
            mService.updateLruProcessLocked(next.app, true, null);
        }
        if (DEBUG_STACK) mStackSupervisor.validateTopActivitiesLocked();
        return true;
    } 

    ...
        
    ActivityStack lastStack = mStackSupervisor.getLastStack();
    if (next.app != null && next.app.thread != null) {
        ...
    } else {
        // Whoops, need to restart this activity!
        if (!next.hasBeenLaunched) {
            next.hasBeenLaunched = true;
        } else {
            if (SHOW_APP_STARTING_PREVIEW) {
                next.showStartingWindow(null /* prev */, false /* newTask */,
                        false /* taskSwich */);
            }
            if (DEBUG_SWITCH) Slog.v(TAG_SWITCH, "Restarting: " + next);
        }
        if (DEBUG_STATES) Slog.d(TAG_STATES, "resumeTopActivityLocked: Restarting " + next);
        mStackSupervisor.startSpecificActivityLocked(next, true, true);
    }

    if (DEBUG_STACK) mStackSupervisor.validateTopActivitiesLocked();
    return true;
}
```

这个方法会执行两次，第一次进入时，`final ActivityRecord next = topRunningActivityLocked(true /* focusableOnly */);`获取到的是栈顶焦点Activity,在执行该方法之前，我们已经将目标ActivityRecord入栈，因此，这里获取的是要打开的目标ActivityRecord，`mStackSupervisor.pauseBackStacks(userLeaving, next, false);`是判断ActivityStack中是否有需要pause的Activity,如果有，说明还有活动的Activity没有Pause,我们要先执行Pause操作，通过执行`startPausingLocked(userLeaving, false, next, false);`将当前未Pause的Activity进行Pause,在执行完毕后方法直接return了。先进去看一下`startPausingLocked`：



```java
final boolean startPausingLocked(boolean userLeaving, boolean uiSleeping,
        ActivityRecord resuming, boolean pauseImmediately) {

    // 将mResumedActivity赋值prev,并将mResumedActivity置空
    ActivityRecord prev = mResumedActivity;
    mResumedActivity = null;
    mPausingActivity = prev;
    

    if (prev.app != null && prev.app.thread != null) {
        if (DEBUG_PAUSE) Slog.v(TAG_PAUSE, "Enqueueing pending pause: " + prev);
        try {
            EventLog.writeEvent(EventLogTags.AM_PAUSE_ACTIVITY,
                    prev.userId, System.identityHashCode(prev),
                    prev.shortComponentName);
            mService.updateUsageStats(prev, false);
            // 开始进行Pause操作
            prev.app.thread.schedulePauseActivity(prev.appToken, prev.finishing,
                    userLeaving, prev.configChangeFlags, pauseImmediately);
        } catch (Exception e) {
            // Ignore exception, if process died other code will cleanup.
            Slog.w(TAG, "Exception thrown during pause", e);
            mPausingActivity = null;
            mLastPausedActivity = null;
            mLastNoHistoryActivity = null;
        }
    } else {
        mPausingActivity = null;
        mLastPausedActivity = null;
        mLastNoHistoryActivity = null;
    }

    if (mPausingActivity != null) {
        if (!uiSleeping) {
            prev.pauseKeyDispatchingLocked();
        } else if (DEBUG_PAUSE) {
             Slog.v(TAG_PAUSE, "Key dispatch not paused for screen off");
        }

        if (pauseImmediately) {
            completePauseLocked(false, resuming);
            return false;

        } else {
            // 延时500ms处理pause后续操作
            schedulePauseTimeout(prev);
            return true;
        }

    } else {
        if (DEBUG_PAUSE) Slog.v(TAG_PAUSE, "Activity not running, resuming next.");
        if (resuming == null) {
            mStackSupervisor.resumeFocusedStackTopActivityLocked();
        }
        return false;
    }
}
```

`mResumedActivity`就是将被Pause的Activity,通过`prev.app.thread.schedulePauseActivity(prev.appToken, prev.finishing, userLeaving, prev.configChangeFlags, pauseImmediately);`对其进行Pause,这里实际是一个异步操作，因此，在方法的末端，添加了一个500ms的延时消息防止Pause操作timeout,当然，这个延时消息不一定会执行，如果pause操作在500ms以内完成会将这个消息取消。如果熟悉Activity启动流程的话，我们就会知道`schedulePauseActivity`最终一定会执行到`ApplicationThread#handlePauseActivity`,这个ApplicationThread在本文章的场景中是AActivity的ApplicationThread，看下代码：



```java
private void handlePauseActivity(IBinder token, boolean finished,
        boolean userLeaving, int configChanges, boolean dontReport, int seq) {
    ActivityClientRecord r = mActivities.get(token);
    if (DEBUG_ORDER) Slog.d(TAG, "handlePauseActivity " + r + ", seq: " + seq);
    if (!checkAndUpdateLifecycleSeq(seq, r, "pauseActivity")) {
        return;
    }
    if (r != null) {
        //Slog.v(TAG, "userLeaving=" + userLeaving + " handling pause of " + r);
        if (userLeaving) {
            performUserLeavingActivity(r);
        }

        r.activity.mConfigChangeFlags |= configChanges;
        // 真正调用Activity#onPause
        performPauseActivity(token, finished, r.isPreHoneycomb(), "handlePauseActivity");

        // Make sure any pending writes are now committed.
        if (r.isPreHoneycomb()) {
            QueuedWork.waitToFinish();
        }

        // Tell the activity manager we have paused.
        if (!dontReport) {
            try {
                // 通知AMS Activity已完成Pause
                ActivityManager.getService().activityPaused(token);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
        }
        mSomeActivitiesChanged = true;
    }
}
```

其中，`performPauseActivity`最终会调用AActivity的onPause,继续向下执行会通过`ActivityManager.getService().activityPaused(token)`通知AMS进程Activity已完成Pause,



```java
// ActivityManagerService
public final void activityPaused(IBinder token) {
     final long origId = Binder.clearCallingIdentity();
     synchronized(this) {
         ActivityStack stack = ActivityRecord.getStackLocked(token);
         if (stack != null) {
             stack.activityPausedLocked(token, false);
         }
     }
     Binder.restoreCallingIdentity(origId);
 }
```

继续看`ActivityStack#activityPausedLocked`



```java
final void activityPausedLocked(IBinder token, boolean timeout) {

    ...

    final ActivityRecord r = isInStackLocked(token);
    if (r != null) {
        // 取消Pause的延时等待消息消息
        mHandler.removeMessages(PAUSE_TIMEOUT_MSG, r);
        if (mPausingActivity == r) {
            if (DEBUG_STATES) Slog.v(TAG_STATES, "Moving to PAUSED: " + r
                    + (timeout ? " (due to timeout)" : " (pause complete)"));
            mService.mWindowManager.deferSurfaceLayout();
            try {
                // 完成Pause的后续操作
                completePauseLocked(true /* resumeNext */, null /* resumingActivity */);
            } finally {
                mService.mWindowManager.continueSurfaceLayout();
            }
            return;
        } 

        ....
}
```

比较重要的两个操作：

- 取消Pause的延时等待消息消息
- 完成Pause的后续操作
  完成Pause后续操作是在`ActivityStack#completePauseLocked`中



```java
private void completePauseLocked(boolean resumeNext, ActivityRecord resuming) {
   ActivityRecord prev = mPausingActivity;
    ...
    if (prev != null) {
       ...
       // 将已被Pause的Activity加入到mStackSupervisor.mStoppingActivities列表中
        addToStopping(prev, true /* scheduleIdle */, false /* idleDelayed */);
        ...
    }
    if (resumeNext) {
        final ActivityStack topStack = mStackSupervisor.getFocusedStack();
        if (!topStack.shouldSleepOrShutDownActivities()) {
            // Resume栈顶Activity, prev是当前在pause的
            mStackSupervisor.resumeFocusedStackTopActivityLocked(topStack, prev, null);
        } else {
            checkReadyForSleep();
            ActivityRecord top = topStack.topRunningActivityLocked();
            if (top == null || (prev != null && top != prev)) {
                mStackSupervisor.resumeFocusedStackTopActivityLocked();
            }
        }
    }

    ...
}
```

重点关注`mStackSupervisor.resumeFocusedStackTopActivityLocked(topStack, prev, null);`
 兜兜转转，又回到了原点，我们即将第二次调用`ActivityStack#resumeTopActivityInnerLocked`,这次跟上一次又有所不同，因为已经完成活跃Activity的pause，因此，判断是否需要Pause的分支语句就不会执行，继续向下执行到`mStackSupervisor.startSpecificActivityLocked(next, true, true);`，这个方法还有一个重要的事情就是将已被Pause的Activity加入到mStackSupervisor.mStoppingActivities的列表中，这是一个ArrayList<ActivityRecord>，至于什么时候用，后面会讲到

##### BActivity#onCreate

讲到这里，我们只是刚完成了AActivity的Pause,有些朋友肯定已经着急了，那得分析到啥时候啊，快了快了，后面就很快了，加快进度，接着看`ActivityStackSupervisor#startSpecificActivityLocked`



```java
void startSpecificActivityLocked(ActivityRecord r,
     boolean andResume, boolean checkConfig) {
 // 获取目标Activity的进程信息
 ProcessRecord app = mService.getProcessRecordLocked(r.processName,
         r.info.applicationInfo.uid, true);

 r.getStack().setLaunchTime(r);

 if (app != null && app.thread != null) {
     try {
         if ((r.info.flags&ActivityInfo.FLAG_MULTIPROCESS) == 0
                 || !"android".equals(r.info.packageName)) {
             app.addPackage(r.info.packageName, r.info.applicationInfo.versionCode,
                     mService.mProcessStats);
         }
         // 启动Activity
         realStartActivityLocked(r, app, andResume, checkConfig);
         return;
     } catch (RemoteException e) {
         Slog.w(TAG, "Exception when starting activity "
                 + r.intent.getComponent().flattenToShortString(), e);
     }

     // If a dead object exception was thrown -- fall through to
     // restart the application.
 }
 // 如果没有进程信息则启动新的进程
 mService.startProcessLocked(r.processName, r.info.applicationInfo, true, 0,
         "activity", r.intent.getComponent(), false, false, true);
}
```

这个方法的主要工作是给ActivityRecord分配进程，如果没有进程，则启动新的进程,从AActivity跳BActivity是同一个进程，因此会继续执行`realStartActivityLocked(r, app, andResume, checkConfig)`:



```java
final boolean realStartActivityLocked(ActivityRecord r, ProcessRecord app,
        boolean andResume, boolean checkConfig) throws RemoteException {

        ...
            app.thread.scheduleLaunchActivity(new Intent(r.intent), r.appToken,
                    System.identityHashCode(r), r.info,
                   
        ...
    return true;
}
```

方法比较长，掐头去尾只看`app.thread.scheduleLaunchActivity`,从这里开始进入App进程执行，`scheduleLaunchActivity`方法在APP进程对应的是`ApplicationThread#scheduleLaunchActivity`:



```java
public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,
        ActivityInfo info, Configuration curConfig, Configuration overrideConfig,
        CompatibilityInfo compatInfo, String referrer, IVoiceInteractor voiceInteractor,
        int procState, Bundle state, PersistableBundle persistentState,
        List<ResultInfo> pendingResults, List<ReferrerIntent> pendingNewIntents,
        boolean notResumed, boolean isForward, ProfilerInfo profilerInfo) {

    updateProcessState(procState, false);

    ActivityClientRecord r = new ActivityClientRecord();

    r.token = token;
    r.ident = ident;
    r.intent = intent;
    r.referrer = referrer;
    r.voiceInteractor = voiceInteractor;
    r.activityInfo = info;
    r.compatInfo = compatInfo;
    r.state = state;
    r.persistentState = persistentState;

    r.pendingResults = pendingResults;
    r.pendingIntents = pendingNewIntents;

    r.startsNotResumed = notResumed;
    r.isForward = isForward;

    r.profilerInfo = profilerInfo;

    r.overrideConfig = overrideConfig;
    updatePendingConfiguration(curConfig);

    sendMessage(H.LAUNCH_ACTIVITY, r);
}
```

在这里创建了`ActivityClientRecord`,因此ActivityClientRecord和AMS中的ActivityRecord就对应起来了。向主线程发送`H.LAUNCH_ACTIVITY`消息，消息回调后执行handleLaunchActivity:



```csharp
private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent, String reason) {
    // If we are getting ready to gc after going to the background, well
    // we are back active so skip it.
  ...

    Activity a = performLaunchActivity(r, customIntent);

    if (a != null) {
        ....
        handleResumeActivity(r.token, false, r.isForward,
                !r.activity.mFinished && !r.startsNotResumed, r.lastProcessedSeq, reason);
      ...
    }
....
}
```

这里调用performLaunchActivity:



```dart
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    // 创建Context
    ContextImpl appContext = createBaseContextForActivity(r);
    Activity activity = null;
    try {
        // 通过ClassLoader创建Activity实例
        java.lang.ClassLoader cl = appContext.getClassLoader();
        activity = mInstrumentation.newActivity(
                cl, component.getClassName(), r.intent);
        StrictMode.incrementExpectedActivityCount(activity.getClass());
        r.intent.setExtrasClassLoader(cl);
        r.intent.prepareToEnterProcess();
        if (r.state != null) {
            r.state.setClassLoader(cl);
        }
    } catch (Exception e) {
        if (!mInstrumentation.onException(activity, e)) {
            throw new RuntimeException(
                "Unable to instantiate activity " + component
                + ": " + e.toString(), e);
        }
    }

    try {
        Application app = r.packageInfo.makeApplication(false, mInstrumentation);



        if (activity != null) {
            //读取Activity信息
            CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
            Configuration config = new Configuration(mCompatConfiguration);
            if (r.overrideConfig != null) {
                config.updateFrom(r.overrideConfig);
            }
            if (DEBUG_CONFIGURATION) Slog.v(TAG, "Launching activity "
                    + r.activityInfo.name + " with config " + config);
            Window window = null;
            if (r.mPendingRemoveWindow != null && r.mPreserveWindow) {
                window = r.mPendingRemoveWindow;
                r.mPendingRemoveWindow = null;
                r.mPendingRemoveWindowManager = null;
            }
            appContext.setOuterContext(activity);
            // 执行Actiivity attach方法
            activity.attach(appContext, this, getInstrumentation(), r.token,
                    r.ident, app, r.intent, r.activityInfo, title, r.parent,
                    r.embeddedID, r.lastNonConfigurationInstances, config,
                    r.referrer, r.voiceInteractor, window, r.configCallback);

            if (customIntent != null) {
                activity.mIntent = customIntent;
            }
            r.lastNonConfigurationInstances = null;
            checkAndBlockForNetworkAccess();
            activity.mStartedActivity = false;
            int theme = r.activityInfo.getThemeResource();
            if (theme != 0) {
                activity.setTheme(theme);
            }

            activity.mCalled = false;
            // Activity onCreate执行
            if (r.isPersistable()) {
                mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
            } else {
                mInstrumentation.callActivityOnCreate(activity, r.state);
            }
            if (!activity.mCalled) {
                throw new SuperNotCalledException(
                    "Activity " + r.intent.getComponent().toShortString() +
                    " did not call through to super.onCreate()");
            }
            r.activity = activity;
            r.stopped = true;
            if (!r.activity.mFinished) {
                // Activity onStart执行
                activity.performStart();
                r.stopped = false;
            }
            if (!r.activity.mFinished) {
                if (r.isPersistable()) {
                    if (r.state != null || r.persistentState != null) {
                        mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state,
                                r.persistentState);
                    }
                } else if (r.state != null) {
                    mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state);
                }
            }
            
        }
        r.paused = true;

        mActivities.put(r.token, r);

    } catch (SuperNotCalledException e) {
        throw e;

    } catch (Exception e) {
       
    }

    return activity;
}
```

这个方法是Activity真正创建实例且onCreate执行的地方,通过调用Activity#attach方法，ActivityClientRecord的token就和Activity联系起来了

##### BActivity#onStart

onCreate执行完成后，直接调用了`activity.performStart();`去执行onStart，因此我们可以得出结论:在Activity的`onCreate`和`onStart`方法中去处理逻辑，实际没有特别大的区别

##### BActivity#onResume

`performLaunchActivity`调用有返回值，即创建的Activity,如果返回值不为空，则说明创建Activity成功，继续执行`handleResumeActivity`



```java
final void handleResumeActivity(IBinder token,
    boolean clearHide, boolean isForward, boolean reallyResume, int seq, String reason) {
    ActivityClientRecord r = mActivities.get(token);
    if (!checkAndUpdateLifecycleSeq(seq, r, "resumeActivity")) {
        return;
    }
    ...
    // 执行onResume
    r = performResumeActivity(token, clearHide, reason);
    ...

    if (r != null) {
        final Activity a = r.activity;

        // 向messageQueue发送一个Idle消息
        if (!r.onlyLocalRequest) {
            r.nextIdle = mNewActivities;
            mNewActivities = r;
            if (localLOGV) Slog.v(
                TAG, "Scheduling idle handler for " + r);
            Looper.myQueue().addIdleHandler(new Idler());
        }
        r.onlyLocalRequest = false;

        // 通知AMS完成了Resume操作
        if (reallyResume) {
            try {
                ActivityManager.getService().activityResumed(token);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
        }

    } 
    ...
}
```

`performResumeActivity`是Activity的onResume执行的地方

##### AActivity#onStop

在执行完成后向住现成的MessageQueue发送了一个idle消息，这个很重要，具体是做什么的呢，我们进去看一下Idler的实现：



```java
private class Idler implements MessageQueue.IdleHandler {
    @Override
    public final boolean queueIdle() {
        ActivityClientRecord a = mNewActivities;
        boolean stopProfiling = false;
        if (mBoundApplication != null && mProfiler.profileFd != null
                && mProfiler.autoStopProfiler) {
            stopProfiling = true;
        }
        if (a != null) {
            mNewActivities = null;
            IActivityManager am = ActivityManager.getService();
            ActivityClientRecord prev;
            do {
                if (localLOGV) Slog.v(
                    TAG, "Reporting idle of " + a +
                    " finished=" +
                    (a.activity != null && a.activity.mFinished));
                if (a.activity != null && !a.activity.mFinished) {
                    try {
                      // 调用ActivityManagerService#activityIdle
                        am.activityIdle(a.token, a.createdConfig, stopProfiling);
                        a.createdConfig = null;
                    } catch (RemoteException ex) {
                        throw ex.rethrowFromSystemServer();
                    }
                }
                prev = a;
                a = a.nextIdle;
                prev.nextIdle = null;
            } while (a != null);
        }
        if (stopProfiling) {
            mProfiler.stopProfiling();
        }
        ensureJitEnabled();
        return false;
    }
}
```

看`am.activityIdle(a.token, a.createdConfig, stopProfiling)`这里，这种调用在本文中出现过很多次了，我们很容易得出结论：这是一次IPC调用，对应的方法在ActivityManagerService#activityIdle:



```java
@Override
public final void activityIdle(IBinder token, Configuration config, boolean stopProfiling) {
    final long origId = Binder.clearCallingIdentity();
    synchronized (this) {
        ActivityStack stack = ActivityRecord.getStackLocked(token);
        if (stack != null) {
            ActivityRecord r =
                    mStackSupervisor.activityIdleInternalLocked(token, false /* fromTimeout */,
                            false /* processPausingActivities */, config);
            if (stopProfiling) {
                if ((mProfileProc == r.app) && mProfilerInfo != null) {
                    clearProfilerLocked();
                }
            }
        }
    }
    Binder.restoreCallingIdentity(origId);
}
```

做的工作不多，调用了`mStackSupervisor.activityIdleInternalLocked`:



```java
final ActivityRecord activityIdleInternalLocked(final IBinder token, boolean fromTimeout,
            boolean processPausingActivities, Configuration config) {
    if (DEBUG_ALL) Slog.v(TAG, "Activity idle: " + token);
    ....
    // Atomically retrieve all of the other things to do.
    final ArrayList<ActivityRecord> stops = processStoppingActivitiesLocked(r,
            true /* remove */, processPausingActivities);
    NS = stops != null ? stops.size() : 0;
    if ((NF = mFinishingActivities.size()) > 0) {
        finishes = new ArrayList<>(mFinishingActivities);
        mFinishingActivities.clear();
    }

    if (mStartingUsers.size() > 0) {
        startingUsers = new ArrayList<>(mStartingUsers);
        mStartingUsers.clear();
    }
    ...
    // Stop any activities that are scheduled to do so but have been
    // waiting for the next one to start.
    for (int i = 0; i < NS; i++) {
        r = stops.get(i);
        final ActivityStack stack = r.getStack();
        if (stack != null) {
            if (r.finishing) {
                stack.finishCurrentActivityLocked(r, ActivityStack.FINISH_IMMEDIATELY, false);
            } else {
                stack.stopActivityLocked(r);
            }
        }
    }
    ...
    return r;
}
```

`processStoppingActivitiesLocked`做的操作就是寻找需要被Stop的Activity，进去看一下：



```java
final ArrayList<ActivityRecord> processStoppingActivitiesLocked(ActivityRecord idleActivity,
            boolean remove, boolean processPausingActivities) {
    ArrayList<ActivityRecord> stops = null;

    final boolean nowVisible = allResumedActivitiesVisible();
    // 遍历mStoppingActivities列表，选择其中符合Stop条件的Activity
    for (int activityNdx = mStoppingActivities.size() - 1; activityNdx >= 0; --activityNdx) {
        ActivityRecord s = mStoppingActivities.get(activityNdx);
        boolean waitingVisible = mActivitiesWaitingForVisibleActivity.contains(s);
        if (DEBUG_STATES) Slog.v(TAG, "Stopping " + s + ": nowVisible=" + nowVisible
                + " waitingVisible=" + waitingVisible + " finishing=" + s.finishing);
        if (waitingVisible && nowVisible) {
            mActivitiesWaitingForVisibleActivity.remove(s);
            waitingVisible = false;
            if (s.finishing) {
                s.setVisibility(false);
            }
        }
        if (remove) {
            final ActivityStack stack = s.getStack();
            final boolean shouldSleepOrShutDown = stack != null
                    ? stack.shouldSleepOrShutDownActivities()
                    : mService.isSleepingOrShuttingDownLocked();
            if (!waitingVisible || shouldSleepOrShutDown) {
                if (!processPausingActivities && s.state == PAUSING) {
                    // Defer processing pausing activities in this iteration and reschedule
                    // a delayed idle to reprocess it again
                    removeTimeoutsForActivityLocked(idleActivity);
                    scheduleIdleTimeoutLocked(idleActivity);
                    continue;
                }

                if (DEBUG_STATES) Slog.v(TAG, "Ready to stop: " + s);
                if (stops == null) {
                    stops = new ArrayList<>();
                }
                stops.add(s);
                mStoppingActivities.remove(activityNdx);
            }
        }
    }

    return stops;
}
```

之前在执行AActivity的Pause操作后，我们将AActivity加入到了ActivityStackSupervisor类中一个叫mStoppingActivities的列表中，在这里这个列表就派上用场了，我们会遍历这个列表，从中获取符合条件的Activity并组装到一个新的列表中返回。

完成这个操作后，`activityIdleInternalLocked`开始遍历这个返回的列表进Stop操作。

看`stack.stopActivityLocked(r)`,凭我20年Android开发经验练就的火眼金睛，感觉这里就是AActivity#onStop触发的地方：



```java
// ActivityStack
final void stopActivityLocked(ActivityRecord r) {


    if (r.app != null && r.app.thread != null) {
        adjustFocusedActivityStackLocked(r, "stopActivity");
        r.resumeKeyDispatchingLocked();
        try {

            r.app.thread.scheduleStopActivity(r.appToken, r.visible, r.configChangeFlags);
            if (shouldSleepOrShutDownActivities()) {
                r.setSleeping(true);
            }
            Message msg = mHandler.obtainMessage(STOP_TIMEOUT_MSG, r);
            mHandler.sendMessageDelayed(msg, STOP_TIMEOUT);
        } catch (Exception e) {

        }
    }
}
```

果然，在这个方法内部执行了`r.app.thread.scheduleStopActivity`,这个方法就不进去看了，我们只需要知道它最终调用了r的onStop就可以了

至此，从AActivity跳转BActivity所经历的生命周期就全部完成了。

为什么要用IdleHandler?

我猜测是因为google的工程师认为既然BActivity已经启动了，那么主线程Handler的首要任务还是处理B进程内部的事情，至于AActivity，反正都已经Pause了，就抽个空闲时间告诉AMS把它Stop就可以了。

#### 从BActivity返回到AActivity

先回顾一下从BActivity返回AActivity经历的生命周期

BActivity#onPause -> AActivity#onStart -> AActivity#onResume -> BActivity#onStop -> BActivity#onDestroy

先从BActivity的finish方法说起

BActivity的finish方法最终会调用到AMS的finishActivity方法，



```java
// ActivityManagerService
public final boolean finishActivity(IBinder token, int resultCode, Intent resultData,
            int finishTask) {
    // Refuse possible leaked file descriptors
    if (resultData != null && resultData.hasFileDescriptors() == true) {
        throw new IllegalArgumentException("File descriptors passed in Intent");
    }

    synchronized(this) {
        ActivityRecord r = ActivityRecord.isInStackLocked(token);
        if (r == null) {
            return true;
        }
        // Keep track of the root activity of the task before we finish it
        TaskRecord tr = r.getTask();
        // 正在结束的Activity所在的ActivityTask
        ActivityRecord rootR = tr.getRootActivity();
        if (rootR == null) {
            Slog.w(TAG, "Finishing task with all activities already finished");
        }
        
        final long origId = Binder.clearCallingIdentity();
        try {
            boolean res;
            final boolean finishWithRootActivity =
                    finishTask == Activity.FINISH_TASK_WITH_ROOT_ACTIVITY;
            if (finishTask == Activity.FINISH_TASK_WITH_ACTIVITY
                    || (finishWithRootActivity && r == rootR)) {
                ...
            } else {
                res = tr.getStack().requestFinishActivityLocked(token, resultCode,
                        resultData, "app-request", true);
            }
            return res;
        } finally {
            Binder.restoreCallingIdentity(origId);
        }
    }
}
```

入参的含义：

- token ： 准备finish的Activity的Binder token引用
- resultCode：  resultCode  字面意思，在onActivityResult接收到的
- resultData： ResultData  字面意思，在onActivityResult接收得到的
- finishTask： 是否将ActivityStack一起finish
  在这里我们又和token见面了，在前面从AActivity启动BActivity的流程里我们分析过，这个token是BActivity在AMS中的标识。

`ActivityManagerService#finishActivity`经过层层调用，会调用到`ActivityStack#finishActivityLocked`:



```java
final boolean finishActivityLocked(ActivityRecord r, int resultCode, Intent resultData,
            String reason, boolean oomAdj, boolean pauseImmediately) {
   if (r.finishing) {
        Slog.w(TAG, "Duplicate finish request for " + r);
        return false;
    }

    mWindowManager.deferSurfaceLayout();
    try {
        // 将r标记为finishing
        r.makeFinishingLocked();
        ...
        // 将resultCode和resultData设置给resultTo
        finishActivityResultsLocked(r, resultCode, resultData);

        final boolean endTask = index <= 0 && !task.isClearingToReuseTask();
        final int transit = endTask ? TRANSIT_TASK_CLOSE : TRANSIT_ACTIVITY_CLOSE;
        if (mResumedActivity == r) {
            if (DEBUG_VISIBILITY || DEBUG_TRANSITION) Slog.v(TAG_TRANSITION,
                    "Prepare close transition: finishing " + r);
            if (endTask) {
                mService.mTaskChangeNotificationController.notifyTaskRemovalStarted(
                        task.taskId);
            }
            mWindowManager.prepareAppTransition(transit, false);

            // Tell window manager to prepare for this one to be removed.
            r.setVisibility(false);

            if (mPausingActivity == null) {
                if (DEBUG_PAUSE) Slog.v(TAG_PAUSE, "Finish needs to pause: " + r);
                if (DEBUG_USER_LEAVING) Slog.v(TAG_USER_LEAVING,
                        "finish() => pause with userLeaving=false");
                // 将当前处于活跃状态的Activity进行Pause
                startPausingLocked(false, false, null, pauseImmediately);
            }

            if (endTask) {
                mStackSupervisor.removeLockedTaskLocked(task);
            }
        } else if (r.state != ActivityState.PAUSING) {
           ...
        } else {
            if (DEBUG_PAUSE) Slog.v(TAG_PAUSE, "Finish waiting for pause of: " + r);
        }

        return false;
    } finally {
        mWindowManager.continueSurfaceLayout();
    }
}
```

本方法重点事项

1. 将r(即将要finish的Activity)的finishing标记位置为true
2. 将将resultCode和resultData设置给resultTo，这个resultTo是我们在创建r的时候传入的，在AActivity跳BActivity的流程中我们介绍过，本场景中resultTo指AActivity的token
3. 将当前处于活跃状态的Activity进行pause,本场景中活跃状态的Activity就是AActivity

##### BActivity#onPause

其中`startPausingLocked`方法我们在前面已经分析过了，这里就不再赘述了，详细的分析可以看3.5.1章节。通过之前的分析我们知道，startPausingLocked会执行BActvity的onPause回调并最终调用到`ActivityManagerService#activityPaused`方法，这个放我我们之前也做过分析，最终会调用到`ActivityStack#resumeTopActivityInnerLocked`，这个方法的调用和BActivity的启动流程略有不同

##### AActivity#onStart、AActivity#onResume

在调用`ActivityStack#resumeTopActivityInnerLocked`时，因为我们已经完成对BActivity的Pause，因此，获取栈顶Activity 即next时，拿到的会是AActivity，而AActivity的app和thread均不为空，因此，
 判断语句`if (next.app != null && next.app.thread != null)` 成立，它会走入`ActivityStack#resumeTopActivityInnerLocked`如下分支：



```dart
if (next.app != null && next.app.thread != null) {
    if (DEBUG_SWITCH) Slog.v(TAG_SWITCH, "Resume running: " + next
            + " stopped=" + next.stopped + " visible=" + next.visible);
        try {
            // Deliver all pending results.
            ArrayList<ResultInfo> a = next.results;
            if (a != null) {
                final int N = a.size();
                if (!next.finishing && N > 0) {
                    if (DEBUG_RESULTS) Slog.v(TAG_RESULTS,
                            "Delivering results to " + next + ": " + a);
                    next.app.thread.scheduleSendResult(next.appToken, a);
                }
            }

            if (next.newIntents != null) {
                next.app.thread.scheduleNewIntent(
                        next.newIntents, next.appToken, false /* andPause */);
            }

            // Well the app will no longer be stopped.
            // Clear app token stopped state in window manager if needed.
            next.notifyAppResumed(next.stopped);

            EventLog.writeEvent(EventLogTags.AM_RESUME_ACTIVITY, next.userId,
                    System.identityHashCode(next), next.getTask().taskId,
                    next.shortComponentName);

            next.sleeping = false;
            mService.showUnsupportedZoomDialogIfNeededLocked(next);
            mService.showAskCompatModeDialogLocked(next);
            next.app.pendingUiClean = true;
            next.app.forceProcessStateUpTo(mService.mTopProcessState);
            next.clearOptionsLocked();
            // zhangyulong ResumeNextActivity
            next.app.thread.scheduleResumeActivity(next.appToken, next.app.repProcState,
                    mService.isNextTransitionForward(), resumeAnimOptions);

            if (DEBUG_STATES) Slog.d(TAG_STATES, "resumeTopActivityLocked: Resumed "
                    + next);
        } catch (Exception e) {
            ...
            return true;
        }
    }

}
```

注意看`next.app.thread.scheduleResumeActivity`这里，这个方法最终会调用AActivity的onRestart、onStart和onResume,这个部分的调用大家肯定都轻车熟路了，就不跟进去看了

##### BActivity#onStop、BActivity#onDestroy

在BActivity启动流程分析中，我们知道在onResume时，会向主线程MessageQueue发送一个IdleHandler，这个IdleHandler最终会调用`ActivityStackSupervisor#activityIdleInternalLocked`，但这次的调用又跟之前有所不同，因为我们在之前的操作时已经将BActivity置为finishing，因此`if (r.finishing)`判断为true，方法调用会执行



```java
final ActivityRecord activityIdleInternalLocked(final IBinder token, boolean fromTimeout,
            boolean processPausingActivities, Configuration config) {
    if (DEBUG_ALL) Slog.v(TAG, "Activity idle: " + token);


    // Atomically retrieve all of the other things to do.
    final ArrayList<ActivityRecord> stops = processStoppingActivitiesLocked(r,
            true /* remove */, processPausingActivities);
    NS = stops != null ? stops.size() : 0;
    if ((NF = mFinishingActivities.size()) > 0) {
        finishes = new ArrayList<>(mFinishingActivities);
        mFinishingActivities.clear();
    }

    if (mStartingUsers.size() > 0) {
        startingUsers = new ArrayList<>(mStartingUsers);
        mStartingUsers.clear();
    }

    // Stop any activities that are scheduled to do so but have been
    // waiting for the next one to start.
    for (int i = 0; i < NS; i++) {
        r = stops.get(i);
        final ActivityStack stack = r.getStack();
        if (stack != null) {
            if (r.finishing) {
                // stop 且 pause
                stack.finishCurrentActivityLocked(r, ActivityStack.FINISH_IMMEDIATELY, false);
            } else {
                // 仅stop
                stack.stopActivityLocked(r);
            }
        }
    }

    return r;
}
```

方法执行`stack.finishCurrentActivityLocked`后，在APP进程对应执行BActivity的onStop和onDestroy

至此，BActivity返回AActivity的生命周期也执行完毕

## AMS源码分析(二)onActivityResult执行过程

### onActivityResult

onActivityResult机制的使用，属于Android基础知识，我想每一个Android程序员都用过，这里就不多做解释了，看一下使用方式

#### AActivity跳转BAcitivty并从BActivity返回数据



```java
// AActivity
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    activityName = "AActivity";
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_stack_test);
    jumpBtn = findViewById(R.id.jumpBtn);
    jumpBtn.setOnClickListener(v -> {
        BActivity.start(this, 1);
    });
    setActivityName();
}
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    Log.d("zyl", activityName + "      requestCode = " + requestCode + "       resultCode = " + resultCode);
    super.onActivityResult(requestCode, resultCode, data);
}


// BActivity
public static void start(Activity activity, int requestCode) {
   activity.startActivityForResult(new Intent(activity, BActivity.class), requestCode);
}

@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    activityName = "BActivity";
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_stack_test);
}

@Override
public void finish() {
    setResult(102);
    super.finish();
}
```

当BActivity finish时，会将resultCode和requestCode通过AActivity的onActivityResult回传回来

### Intent.FLAG_ACTIVITY_FORWARD_RESULT

这是onActivityResult的另一种表现形式，在日常开发中我们经常会遇到如下场景
 1.我要从Activity A 跳到B再跳到C
 2.我要从Activity C 返回数据给Activity A
 3.这种跳转层级有可能有很多层，比如 A ->B -> C -> D -> E等等

有同学看完肯定会说，这个简单啊，每个Activity都实现onActivityResult，每次跳转都使用startActivityForResult不就行了，这样当然可以，但是代码会稍显冗余。实际上，Android系统已经帮我们想好了这种场景的处理方式了

Intent.FLAG_ACTIVITY_FORWARD_RESULT的作用就是最后一个Activity直接调用第一个Activity的onActivityResult

#### 示例：

以AActivity跳转BActivity跳转CActivity为例：

##### AActivity以startActivityForResult方式打开BActivity



```java
public static void start(Activity activity, int requestCode) {
    Intent intent = new Intent(activity, BActivity.class);
    activity.startActivityForResult(intent, requestCode);
}
```

##### BActivity以普通方式打开CActivity，设置Intent 的Flag Intent.FLAG_ACTIVITY_FORWARD_RESULT



```java
public static void start(Context context) {
    Intent intent = new Intent(context, CActivity.class);
    intent.setFlags(Intent.FLAG_ACTIVITY_FORWARD_RESULT);
    context.startActivity(intent);
}
```

CActivity和BActivity返回后，AActivity会接收到从CActivity传递过来的消息

### 源码解析

#### ActivityResult数据的写入

上篇文章我们在解析Activity的finish过程时其实稍微接触了一些onActivity的执行流程，在Activity的finish过程中，会调用到`ActivityStack#finishActivityResultsLocked`方法



```csharp
private void finishActivityResultsLocked(ActivityRecord r, int resultCode, Intent resultData) {
    // zhangyulong 将结果传递给谁
    ActivityRecord resultTo = r.resultTo;
    if (resultTo != null) {
        if (DEBUG_RESULTS) Slog.v(TAG_RESULTS, "Adding result to " + resultTo
                + " who=" + r.resultWho + " req=" + r.requestCode
                + " res=" + resultCode + " data=" + resultData);
        if (resultTo.userId != r.userId) {
            if (resultData != null) {
                resultData.prepareToLeaveUser(r.userId);
            }
        }
        if (r.info.applicationInfo.uid > 0) {
            mService.grantUriPermissionFromIntentLocked(r.info.applicationInfo.uid,
                    resultTo.packageName, resultData,
                    resultTo.getUriPermissionsLocked(), resultTo.userId);
        }
        // zhangyulong 将ActivityResult结果放在resultTo中保存
        resultTo.addResultLocked(r, r.resultWho, r.requestCode, resultCode,
                                 resultData);
        r.resultTo = null;
    }
    else if (DEBUG_RESULTS) Slog.v(TAG_RESULTS, "No result destination from " + r);
    r.results = null;
    r.pendingResults = null;
    r.newIntents = null;
    r.icicle = null;
}
```

入参含义：

- r：即将finish的Activity
- resultCode: 要传递的resultCode
- resultData：要传递的Intent
  这个方法将result信息封装成一个ResultIfo对象，并保存在列表中

#### ActivityResult数据的传递

在上篇文章中，我们在分析Activity的finish流程时提到过`ActivityStack#resumeTopActivityInnerLocked`方法，这个方法还有一个作用就是向上一个Activity,即`resultTo`传递ActivityResult



```java
private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
    ...
    if (next.app != null && next.app.thread != null) {
        ...
        synchronized(mWindowManager.getWindowManagerLock()) {
            ...
            try {
                // Deliver all pending results.
                ArrayList<ResultInfo> a = next.results;
                if (a != null) {
                    final int N = a.size();
                    if (!next.finishing && N > 0) {
                        if (DEBUG_RESULTS) Slog.v(TAG_RESULTS,
                                "Delivering results to " + next + ": " + a);
                        next.app.thread.scheduleSendResult(next.appToken, a);
                    }
                }

        
                ...
            }
        }

        ...
    } else {
       ....
    }

    if (DEBUG_STACK) mStackSupervisor.validateTopActivitiesLocked();
    return true;
}
```

通过调用`next.app.thread.scheduleSendResult(next.appToken, a)`将ResultInfo发送到App进程中对应的Activity中，并回调onActivityResult方法

#### Intent.FLAG_ACTIVITY_FORWARD_RESULT的实现

上篇文章我们分析过，Activity的启动流程会调用`ActivityStarter#startActivity`,这个方法有对Intent.FLAG_ACTIVITY_FORWARD_RESULT的处理，废话不多说，直接看源码：



```java
private int startActivity(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
       String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
        IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
        IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
        String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
        ActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
        ActivityRecord[] outActivity, TaskRecord inTask) {
    ...

    ActivityRecord sourceRecord = null;
    ActivityRecord resultRecord = null;
    if (resultTo != null) {
        // zhangyulong 获取启动的源activity
        sourceRecord = mSupervisor.isInAnyStackLocked(resultTo);
        if (DEBUG_RESULTS) Slog.v(TAG_RESULTS,
                "Will send result to " + resultTo + " " + sourceRecord);
        if (sourceRecord != null) {
            if (requestCode >= 0 && !sourceRecord.finishing) {
                resultRecord = sourceRecord;
            }
        }
    }
    if ((launchFlags & Intent.FLAG_ACTIVITY_FORWARD_RESULT) != 0 && sourceRecord != null) {
        // 添加Intent.FLAG_ACTIVITY_FORWARD_RESULT启动的Activity不能使用startActivityForResult启动
        if (requestCode >= 0) {
            ActivityOptions.abort(options);
            return ActivityManager.START_FORWARD_AND_REQUEST_CONFLICT;
        }
        // 将源Activity的resultTo赋值给目标Activity
        resultRecord = sourceRecord.resultTo;
        if (resultRecord != null && !resultRecord.isInStackLocked()) {
            resultRecord = null;
        }
        resultWho = sourceRecord.resultWho;
        requestCode = sourceRecord.requestCode;
        sourceRecord.resultTo = null;
        if (resultRecord != null) {
            resultRecord.removeResultsLocked(sourceRecord, resultWho, requestCode);
        }
        if (sourceRecord.launchedFromUid == callingUid) {
            callingPackage = sourceRecord.launchedFromPackage;
        }
    }
    ...
    return startActivity(r, sourceRecord, voiceSession, voiceInteractor, startFlags, true,
            options, inTask, outActivity);
}
```

代码比较简单，主要就是判断Intent中是否带有Intent.FLAG_ACTIVITY_FORWARD_RESULT标签，如果有，则将源Activity的resultTo赋值给目标Activity

如果有多个中间Activity，比如使用这种方式启动ABCDE五个Activity,其中，BCD都是中间Activity，
 那么resultTo的传递过程如下：
 1.A启动B，A作为resultTo传递给B
 2.B启动C，添加Intent.FLAG_ACTIVITY_FORWARD_RESULT，C获取B的ResultTo即A
 3.C启动D，添加Intent.FLAG_ACTIVITY_FORWARD_RESULT，D获取C的ResultTo即A
 4.D启动E，添加Intent.FLAG_ACTIVITY_FORWARD_RESULT，E获取D的ResultTo即A
 5.从E逐级返回，当E返回时，将ResultInfo赋值给A，当返回到A时，触发`next.app.thread.scheduleSendResult(next.appToken, a)`

## AMS源码分析(三)AMS中Activity栈管理详解

Activity栈管理是AMS的另一个重要功能，栈管理又和Activity的启动模式和startActivity时所设置的Flag息息相关，Activity栈管理的主要处理逻辑是在`ActivityStarter#startActivityUnchecked`方法中，本文也会围绕着这个方法进进出出，反复摩擦，直到脑海中都是它的形状。goolge的工程师起名还是很讲究的，为什么要带Unchecked呢? Unchecked-不确定，是因为在执行这个方法时，我要启动哪个Activity还没决定呢，具体为什么，我想看过这篇文章你就明白了。

### Activity栈管理相关类

#### ActivityStackSupervisor

顾名思义，Activity栈的功能提供者和管理者

#### ActivityDisplay

表示一个屏幕，Android支持三种屏幕：主屏幕，外接屏幕（HDMI等），虚拟屏幕（投屏）。一般情况下，即只有主屏幕时，ActivityStackSupervisor与ActivityDisplay都是系统唯一

#### TaskRecord

ActivityTask记录, Task是我们管理Activity栈的重要单元，它的表现形式与逻辑和Activity启动模式息息相关，也是本文重点要分析的

#### ActivityStack

针对ActivityRecord 和 TaskRecord进行管理，记录ActivityRecord的状态和TaskRecord的状态。在Android N之前只有两种ActivityStack:homeStack（launcher和recents Activity）和其他。Android N开始有5种，增加了DockedStack（分屏Activity）、PinnedStack（画中画Activity）、freeformstack(自由模式Activity)，虽然它名字叫ActivityStack，但是跟我们熟知的数据结构中的栈基本上没啥关系，这也是有可能会增加一点理解难度的地方

#### 关系图：

先说一下关系：

- 一个ActivityDisplay包含多个ActivityStack
- 一个ActivityStack包含多个TaskRecord
- 一个TaskRecord包含多个ActivityRecord

![img](https:////upload-images.jianshu.io/upload_images/3112838-9cc3f298a586ec45.png?imageMogr2/auto-orient/strip|imageView2/2/w/1106/format/webp)

### 启动模式

#### standard

标准启动模式，启动Activity时依次向栈顶添加ActivityRecord，返回时依次推出，示例:AActivity打开BActivity打开CActivity

![img](https:////upload-images.jianshu.io/upload_images/3112838-cd8e980ebd343e68.png?imageMogr2/auto-orient/strip|imageView2/2/w/591/format/webp)

这是基本的Activity启动模式，需要注意的点不太多

##### Intent.FLAG_ACTIVITY_CLEAR_TOP

还是刚才的案例，如果依次打开AActivity->BActivity->CActivity->DActivity，此时在DActivity打开AActivity时Intent添加Flag `Intent.FLAG_ACTIVITY_CLEAR_TOP`,系统会从BActivity开始依次将Task中的Activity依次销毁，直到DActivity,因为DActivity处于活跃状态，因此会先执行onPause,在onPause后，会销毁原来的AActivity,然后打开新的AActivity,最后执行DActivity的onStop和onDestory
 流程图如下：

![img](https:////upload-images.jianshu.io/upload_images/3112838-6c3bcffc85f5f084.png?imageMogr2/auto-orient/strip|imageView2/2/w/1189/format/webp)



log:



![img](https:////upload-images.jianshu.io/upload_images/3112838-474e338521ee9f68.png?imageMogr2/auto-orient/strip|imageView2/2/w/871/format/webp)

##### 源码分析

###### 加入TaskRecord

摘抄`ActivityStarter#startActivityUnchecked`部分代码如下：



```kotlin
if (mStartActivity.resultTo == null && mInTask == null && !mAddingToTask
    && (mLaunchFlags & FLAG_ACTIVITY_NEW_TASK) != 0) {
    newTask = true;
    // zhangyulong 使用一个旧的Task 或者新建一个
    result = setTaskFromReuseOrCreateNewTask(
            taskToAffiliate, preferredLaunchStackId, topStack);
} else if (mSourceRecord != null) {
    // zhangyulong 使用源Activity的task
    result = setTaskFromSourceRecord();
} else if (mInTask != null) {
    // zhangyulong 使用启动时传递的task
    result = setTaskFromInTask();
} else {
    // zhangyulong 理论上的可能，不可能走到这里
    setTaskToCurrentTopOrCreateNewTask();
}
if (result != START_SUCCESS) {
    return result;
}
```

因为我们使用标准模式启动，因此，`resultTo`和`mSourceRecord`均不为空，这段逻辑会执行`setTaskFromSourceRecord`：



```cpp
private int setTaskFromSourceRecord() {
       ...
        addOrReparentStartingActivity(sourceTask, "setTaskFromSourceRecord");
        return START_SUCCESS;
    }
addOrReparentStartingActivity最终会执行TaskRecord#addActivityAtIndex:

void addActivityAtIndex(int index, ActivityRecord r) {
        ...
        mActivities.add(index, r);
        ...
    }
```

向对应的ActivityRecord中的mActivities添加本条记录，完成加入TaskRecord的操作

###### standard + Intent.FLAG_ACTIVITY_CLEAR_TOP

回到`ActivityStarter#startActivityUnchecked`,摘抄部分逻辑如下：



```kotlin
if (!mAddingToTask && (mLaunchFlags & FLAG_ACTIVITY_CLEAR_TOP) != 0) {
    // 将ActivityTask目标Actiivty之上的Activity全部清空，返回值top为可以复用的Activity
    ActivityRecord top = sourceTask.performClearTaskLocked(mStartActivity, mLaunchFlags);
    mKeepCurTransition = true;
    // 如果可复用的Activity不为空，直接调用它的onNewIntent方法并将其resume
    if (top != null) {
        ActivityStack.logStartActivity(AM_NEW_INTENT, mStartActivity, top.getTask());
        deliverNewIntent(top);
        mTargetStack.mLastPausedActivity = null;
        if (mDoResume) {
            mSupervisor.resumeFocusedStackTopActivityLocked();
        }
        ActivityOptions.abort(mOptions);
        return START_DELIVERED_TO_TOP;
    }
}
```

重点看`performClearTaskLocked`,这里是将目标Activity顶部元素清空的逻辑



```dart
/***
* newR: 需要启动的新Activity
* launchFlags: 新Activity的启动模式
*/
final ActivityRecord performClearTaskLocked(ActivityRecord newR, int launchFlags) {
    int numActivities = mActivities.size();
    for (int activityNdx = numActivities - 1; activityNdx >= 0; --activityNdx) {
        ActivityRecord r = mActivities.get(activityNdx);
        if (r.finishing) {
            continue;
        }
        // 在目标Task中找到了和新Activity相同的记录
        if (r.realActivity.equals(newR.realActivity)) {
            final ActivityRecord ret = r;
            //将在其之上的Activity全部清除
            for (++activityNdx; activityNdx < numActivities; ++activityNdx) {
                r = mActivities.get(activityNdx);
                if (r.finishing) {
                    continue;
                }
                ActivityOptions opts = r.takeOptionsLocked();
                if (opts != null) {
                    ret.updateOptionsLocked(opts);
                }
                // 执行finishActivityLocked，如果Activity已经stop,会直接执行onDestroy,
                // 如果Activity还在活跃，则会先执行onPause
                if (mStack != null && mStack.finishActivityLocked(
                        r, Activity.RESULT_CANCELED, null, "clear-task-stack", false)) {
                    --activityNdx;
                    --numActivities;
                }
            }

            // ActivityInfo.LAUNCH_MULTIPLE == standrad
            // 如果新Activity的launchMode是standard,且launchFlag没有FLAG_ACTIVITY_SINGLE_TOP，则将之前task
            // 内的activity也结束，以便建立新的
            if (ret.launchMode == ActivityInfo.LAUNCH_MULTIPLE
                    && (launchFlags & Intent.FLAG_ACTIVITY_SINGLE_TOP) == 0
                    && !ActivityStarter.isDocumentLaunchesIntoExisting(launchFlags)) {
                if (!ret.finishing) {
                    if (mStack != null) {
                        mStack.finishActivityLocked(
                                ret, Activity.RESULT_CANCELED, null, "clear-task-top", false);
                    }
                    // 返回空，说明不执行onNewIntent
                    return null;
                }
            }

            return ret;
        }
    }
    return null;
}
```

看完这部分代码，我想FLAG_ACTIVITY_CLEAR_TOP是怎么工作的大家也就明白了。

#### singleTop

singleTop:栈顶唯一，它和standrad的区别在于，如果是standrad模式，在栈顶启动一个相同的Activity，会创建一个新的Activity实例，如果是singleTop模式，在栈顶启动相同的Activity则只会调用原有Activity的onNewIntent,如果原Activity不在栈顶，那么表现形式就和standrad相同

##### 流程图：

###### 原Activity不在栈顶

![img](https:////upload-images.jianshu.io/upload_images/3112838-a44aedbebde4ad4a.png?imageMogr2/auto-orient/strip|imageView2/2/w/767/format/webp)



###### 原Activity在栈顶

![img](https:////upload-images.jianshu.io/upload_images/3112838-0a897cf31cb9aa8b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1103/format/webp)



##### singleTop + Intent.FLAG_ACTIVITY_CLEAR_TOP

现在设想一种场景，AActivitylanchMode为singleTop，在AActivity的基础上依次打开了BActivity、CActivity， 在CActivity再次打开AActivity，但打开时设置Intent属性`Intent.FLAG_ACTIVITY_CLEAR_TOP`,此时的启动流程：

![img](https:////upload-images.jianshu.io/upload_images/3112838-7ba227bafbceb763.png?imageMogr2/auto-orient/strip|imageView2/2/w/922/format/webp)



##### log

###### 原Activity不在栈顶

![img](https:////upload-images.jianshu.io/upload_images/3112838-6a5468b4fbb56192.png?imageMogr2/auto-orient/strip|imageView2/2/w/961/format/webp)



###### 原Activity在栈顶

![img](https:////upload-images.jianshu.io/upload_images/3112838-09cafa3d81155741.png?imageMogr2/auto-orient/strip|imageView2/2/w/1022/format/webp)



###### 原Actiivty在栈顶且设置Intent.FLAG_ACTIVITY_CLEAR_TOP

![img](https:////upload-images.jianshu.io/upload_images/3112838-78c410cfc373debf.png?imageMogr2/auto-orient/strip|imageView2/2/w/821/format/webp)



##### 源码分析

我们又要进入`ActivityStarter#startActivityUnchecked`方法了， startActivityUnchecked：你要对我负责555...

摘抄startActivityUnchecked方法中关于singleTop模式的处理如下：



```kotlin
// 要启动的Activity正好是当前在栈顶的Activity
// 当前聚焦的ActivityStack
final ActivityStack topStack = mSupervisor.mFocusedStack;
// 当前聚焦的ActivityStack中的栈顶Actiivty
final ActivityRecord topFocused = topStack.topActivity();
// 当前栈顶Activity
final ActivityRecord top = topStack.topRunningNonDelayedActivityLocked(mNotTop);
final boolean dontStart = top != null && mStartActivity.resultTo == null
        && top.realActivity.equals(mStartActivity.realActivity)
        && top.userId == mStartActivity.userId
        && top.app != null && top.app.thread != null
        && ((mLaunchFlags & FLAG_ACTIVITY_SINGLE_TOP) != 0
        || mLaunchSingleTop || mLaunchSingleTask);
// dontStart为true说明可以直接复用栈顶Activity
if (dontStart) {
    // For paranoia, make sure we have correctly resumed the top activity.
    topStack.mLastPausedActivity = null;
    if (mDoResume) {
        mSupervisor.resumeFocusedStackTopActivityLocked();
    }
    ActivityOptions.abort(mOptions);
    if ((mStartFlags & START_FLAG_ONLY_IF_NEEDED) != 0) {
        // We don't need to start a new activity, and the client said not to do
        // anything if that is the case, so this is it!
        return START_RETURN_INTENT_TO_CALLER;
    }
    // zhangyulong singleTop  sigleTask onNewIntent 执行
    deliverNewIntent(top);

    return START_DELIVERED_TO_TOP;
}
```

需要注意的是，当我们设置目标Activity launchMode是singleTop时，判断条件用的是mLaunchSingleTop，而(mLaunchFlags & FLAG_ACTIVITY_SINGLE_TOP是指启动时设置的Intent的Flag属性。这两种方式都能达到singleTop的效果。

###### singleTop + Intent.FLAG_ACTIVITY_CLEAR_TOP

这部分的逻辑处理和standard时差不多，还是`performClearTaskLocked`方法，这个方法在3.1.2.2章节已经分析过了，这里摘抄部分逻辑如下：



```kotlin
// ActivityInfo.LAUNCH_MULTIPLE == standrad
// 如果新Activity的launchMode是standard,且launchFlag没有FLAG_ACTIVITY_SINGLE_TOP，则将之前task
// 内的activity也结束，以便建立新的
if (ret.launchMode == ActivityInfo.LAUNCH_MULTIPLE
        && (launchFlags & Intent.FLAG_ACTIVITY_SINGLE_TOP) == 0
        && !ActivityStarter.isDocumentLaunchesIntoExisting(launchFlags)) {
    if (!ret.finishing) {
        if (mStack != null) {
            mStack.finishActivityLocked(
                    ret, Activity.RESULT_CANCELED, null, "clear-task-top", false);
        }
        // 返回空，说明不执行onNewIntent
        return null;
    }
}
```

因为我们的launchMode是FLAG_ACTIVITY_SINGLE_TOP，条件不成立，不会销毁命中的原Activity,但原Actiivty上面的记录均已销户，此时它已经是栈顶Activity了，继续执行会执行到`startActivityUnchecked`方法中3.2.3章节部分，和上面的逻辑就一致了。

#### singleTask

singleTask: 栈内唯一，如果Activity设置了launchMode为singleTask，那么在整个ActivityStack中有且仅有一个实例存在。有些朋友看到它名字叫singleTask，就想当然的认为它是Task内唯一的，我们不要被它的名字骗了。需要注意的一点是，singleTask模式启动是默认clearTop的。

##### 启动流程

###### 场景一

假设AActivity的launchMode为singleTask，AActivity后依次启动BActivity和CActivity, CActivity又启动了AActivity,
 那么这个过程经历的流程如下：



![img](https:////upload-images.jianshu.io/upload_images/3112838-1fa789833f2166c9.png?imageMogr2/auto-orient/strip|imageView2/2/w/901/format/webp)



log:



![img](https:////upload-images.jianshu.io/upload_images/3112838-a4fbd4f21483e22d.png?imageMogr2/auto-orient/strip|imageView2/2/w/896/format/webp)



###### 场景二

 假设AActivity和BActivity都是Standard,CActivity为singleTask,CActiivty再次启动CActivity,这个过程的启动流程：

![img](https:////upload-images.jianshu.io/upload_images/3112838-acca060d14e0b0c0.png?imageMogr2/auto-orient/strip|imageView2/2/w/783/format/webp)



log：



![img](https:////upload-images.jianshu.io/upload_images/3112838-1260cfaa0a9d1eee.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



###### 场景三

 看完前面两种场景，肯定会有同学不服气，你不是说singleTask是栈内唯一么，这两种场景都是task内的处理啊，那不就应该是task内唯一么，你说栈内唯一拿出证据来啊！别着急，证据马上来。

现在假设AActivity是singleTask， BActivity是standard, BActivity打开CActivity时创建新的Task, 然后CActivity再次打开AActivity
 流程如下：
 1.AActivity在其自身所在task启动BActivity
 2.BActivity在启动CActivity时创建新的Task
 3.CActivity启动AActivity时将AActivity所在的Task移到顶部
 4.AActivity将BActivity清除并重新启动

流程图：



![img](https:////upload-images.jianshu.io/upload_images/3112838-fe78ffaab3dbbbb6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1052/format/webp)



看下log是不是这样：



![img](https:////upload-images.jianshu.io/upload_images/3112838-fea64811c8bae7ea.png?imageMogr2/auto-orient/strip|imageView2/2/w/1162/format/webp)



log也验证了这个说法的正确性，从这个案例中我们看出，虽然AB在一个task, C在另一个task，但C启动A的时候，并没有在其自身的task启动，而是操作AB所在的task。因此，singeTask是栈内唯一的。

##### singeTask源码分析

再进入一次`ActivityStarter#startActivityUnchecked`一次，此时`startActivityUnchecked`内心活动：别进了，再进就怀孕了！！

为什么说singleTask自带clearTop属性呢？ 看下`startActivityUnchecked`的这段逻辑：



```dart
// 如果启动Intent设置了FLAG_ACTIVITY_CLEAR_TOP或者目标Activity启动模式为singleInstance或者singleTask，执行以下逻辑
if ((mLaunchFlags & FLAG_ACTIVITY_CLEAR_TOP) != 0
        || isDocumentLaunchesIntoExisting(mLaunchFlags)
        || mLaunchSingleInstance || mLaunchSingleTask) {
    final TaskRecord task = reusedActivity.getTask();
    // 将与新Actiivty在Task内相同的Activity的顶部元素清空
    final ActivityRecord top = task.performClearTaskForReuseLocked(mStartActivity,
            mLaunchFlags);
   ...
}
```

那么task切换到栈顶是在哪里呢，这段逻辑执行完成后，就会执行到Task切换的逻辑了，代码：



```cpp
// 将目标Activity所在Task移动到栈顶
reusedActivity = setTargetStackAndMoveToFrontIfNeeded(reusedActivity);
```

当前面的逻辑完成后，可复用用的Activity就已经在Task顶部，而Task也已经在Stack顶部，完事俱备，只欠东风，继续执行`startActivityUnchecked`会执行到3.2.3的内容，和singleTop的处理是一致的，最终回调了目标Activity的onNewIntent。

#### singleInstance

singleInstance这个启动模式比sigleTask更NB一些，它不光在栈内唯一，而且还独占一个Task，一看就是那种有独立办公室的老板，跟其他打工人完全不是一个等级的。关于singleInstance的栈管理和切换，你可以把它理解成只有一个singleTask的Activity存在的Task就比较好理解了，上面我们也已经分析过了。

### Intent.FLAG_ACTIVITY_NEW_TASK、taskAffinity、新Task的创建

先看几个有意思的案例：
 现在有两个Activity，分别是AActivity和BActivity，我们先设置A和B均为standard，在A启动B时设置`Intent.FLAG_ACTIVITY_NEW_TASK`,跳转时Task会新建么？
 看一下log:

![img](https:////upload-images.jianshu.io/upload_images/3112838-76e4c948605a09a9.png?imageMogr2/auto-orient/strip|imageView2/2/w/767/format/webp)




 从图上我们看到，A和B的TaskId都是305，也就是没有创建新的Task,怎么回事，Intent.FLAG_ACTIVITY_NEW_TASK怎么失效了？



这个时候把B的launchMode设置为singleTask呢？
 看一下log:



![img](https:////upload-images.jianshu.io/upload_images/3112838-c27c170253ec3831.png?imageMogr2/auto-orient/strip|imageView2/2/w/1039/format/webp)



依然没有生效！！

这个时候保持B的launchMode为singleTask, 设置B的taskAffinity为".b"试一下：



![img](https:////upload-images.jianshu.io/upload_images/3112838-6c40dcb6ecf32be1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1032/format/webp)



生效了！！

这个时候把Intent.FLAG_ACTIVITY_NEW_TASK取消，保持B的launchMode为singleTask, 设置B的taskAffinity为".b"试一下：

![img](https:////upload-images.jianshu.io/upload_images/3112838-e0c3f1585d574b24.png?imageMogr2/auto-orient/strip|imageView2/2/w/1061/format/webp)



生效了！

这个时候把Intent.FLAG_ACTIVITY_NEW_TASK取消，修改B的launchMode为standard, 设置B的taskAffinity为".b"试一下：



![img](https:////upload-images.jianshu.io/upload_images/3112838-bd3c6b843d72f901.png?imageMogr2/auto-orient/strip|imageView2/2/w/1025/format/webp)



没生效！

这个时候把B的taskAffinity删掉，设置B为singleInstace试一下：



![img](https:////upload-images.jianshu.io/upload_images/3112838-88f51fe4a9a20a16.png?imageMogr2/auto-orient/strip|imageView2/2/w/1034/format/webp)



又生效了！！

看到这里是不是感觉已经晕了，那到底啥时候生效啥时候失效啊！别着急，看完源码我们再做总结

我们又要进入`ActivityStarter#startActivityUnchecked`方法了， startActivityUnchecked:不挣扎了，已经有你的形状了...

#### Intent.FLAG_ACTIVITY_NEW_TASK的自动设置

startActivityUnchecked前面几行代码执行了一个叫做`computeLaunchingTaskFlags`的方法，这个方法的作用是根据新Activity的launchMode对launchFlag做处理：



```csharp
private void computeLaunchingTaskFlags() {
    ...
    if (mInTask == null) {
        if (mSourceRecord == null) {
            ...
        } else if (mSourceRecord.launchMode == LAUNCH_SINGLE_INSTANCE) {
            // 如果源Activity是singleInstance，则新启动Activity时自动添加launchFlag FLAG_ACTIVITY_NEW_TASK
            mLaunchFlags |= FLAG_ACTIVITY_NEW_TASK;
        } else if (mLaunchSingleInstance || mLaunchSingleTask) {
            // 如果新Activity的launchMode是singleInstace或者singleTask，则新启动Activity时自动添加launchFlag FLAG_ACTIVITY_NEW_TASK
            mLaunchFlags |= FLAG_ACTIVITY_NEW_TASK;
        }
    }
}
```

也就是说，如果一个Actiivty是singleInstacne的，那么不管是别人启动它还是它启动别人，都会自动添加启动flag FLAG_ACTIVITY_NEW_TASK, 如果是singeTask，则只有别人启动它时才会这样设置

#### taskAffinity的识别

上一部分的逻辑执行后，`startActivityUnchecked`会执行`getReusableIntentActivity`方法，这个方法主要是寻找ActivityStack中是否有可复用的Task， 返回值会可复用Task的顶部元素:



```java
private ActivityRecord getReusableIntentActivity() {

    // 设置了launchFlag为FLAG_ACTIVITY_NEW_TASK或者 launchMode为singleInstance或singleTask
    boolean putIntoExistingTask = ((mLaunchFlags & FLAG_ACTIVITY_NEW_TASK) != 0 &&
            (mLaunchFlags & FLAG_ACTIVITY_MULTIPLE_TASK) == 0)
            || mLaunchSingleInstance || mLaunchSingleTask;

    // inTask为null 且requestCode小于0（即resultTo ==  null）
    putIntoExistingTask &= mInTask == null && mStartActivity.resultTo == null;
    ActivityRecord intentActivity = null;
    if (mOptions != null && mOptions.getLaunchTaskId() != -1) {
        final TaskRecord task = mSupervisor.anyTaskForIdLocked(mOptions.getLaunchTaskId());
        intentActivity = task != null ? task.getTopActivity() : null;
    } else if (putIntoExistingTask) {
        if (mLaunchSingleInstance) {
            // 如果launchMode为singleInstance,只要当前状态下Stack中有和要启动的Activity相同的记录，就说明可以复用
           intentActivity = mSupervisor.findActivityLocked(mIntent, mStartActivity.info,
                   mStartActivity.isHomeActivity());
        } else if ((mLaunchFlags & FLAG_ACTIVITY_LAUNCH_ADJACENT) != 0) {
            // 没研究
            intentActivity = mSupervisor.findActivityLocked(mIntent, mStartActivity.info,
                    !mLaunchSingleTask);
        } else {
            // 其他情况查找Stack中是否有适用的Task和可复用的Actiivty
            intentActivity = mSupervisor.findTaskLocked(mStartActivity, mSourceDisplayId);
        }
    }
    return intentActivity;
}
```

`findActivityLocked`逻辑比较简单，就是在整个Stack中遍历Activity作对比，重点看`ActivitySuperVisor#findTaskLocked`,`ActivitySuperVisor#findTaskLocked`中调用了`ActivityStack#findTaskLocked`，看一下重要逻辑：



```csharp
void findTaskLocked(ActivityRecord target, FindTaskResult result) {
    ...
        } else if (!isDocument && !taskIsDocument
                && result.r == null && task.rootAffinity != null) {
            // 如果Task的rootAffinity和新Activity的taskAffinity匹配，则说明有可复用的栈
            // ，task的rootAffinity一般由底部Actiivty决定，不特意设置的话,一般使用包名
            if (task.rootAffinity.equals(target.taskAffinity)) {
                result.r = r;
                result.matchedByRootAffinity = true;
            }
        } else if (DEBUG_TASKS) Slog.d(TAG_TASKS, "Not a match: " + task);
    }
}
```

这里就是匹配taskAffinity的地方。回到`getReusableIntentActivity`方法，说一下它的返回值逻辑：

- 如果launchMode是singleInstance，则判断当前stack中是否有相同Actiivty，如果有则返回对应Actiivty，否则是null
- 如果launchMode是其他，则判断当前stack中是否有可以匹配其affinity的Task，如果有则返回对应Task顶部Activity，否则是null

#### 是否创建新task的识别

如果`getReusableIntentActivity`方法返回值不为null，`startActivityUncheck`后面的逻辑会执行`setTaskFromReuseOrCreateNewTask`方法：



```java
private void setTaskFromIntentActivity(ActivityRecord intentActivity) {
    if ((mLaunchFlags & (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK))
            == (FLAG_ACTIVITY_NEW_TASK | FLAG_ACTIVITY_CLEAR_TASK)) {
        final TaskRecord task = intentActivity.getTask();
        task.performClearTaskLocked();
        // 设置启动新Actiivty时所使用的task
        mReuseTask = task;
        mReuseTask.setIntent(mStartActivity);
    }
    ...
}
```

注意，这个方法执行的必要条件是`getReusableIntentActivity`方法返回值不为null，入参intentActivity即是`getReusableIntentActivity`方法的返回值。
 因此，假如mReuseTask 为null，则启动Actiivty时会创建新的task，否则向mReuseTask 中添加，逻辑如下：



```java
private int setTaskFromReuseOrCreateNewTask(
        TaskRecord taskToAffiliate, int preferredLaunchStackId, ActivityStack topStack) {
    mTargetStack = computeStackFocus(
            mStartActivity, true, mLaunchBounds, mLaunchFlags, mOptions);
    if (mReuseTask == null) {
        // 创建新的Task
        final TaskRecord task = mTargetStack.createTaskRecord(
                mSupervisor.getNextTaskIdForUserLocked(mStartActivity.userId),
                mNewTaskInfo != null ? mNewTaskInfo : mStartActivity.info,
                mNewTaskIntent != null ? mNewTaskIntent : mIntent, mVoiceSession,
                mVoiceInteractor, !mLaunchTaskBehind /* toTop */, mStartActivity.mActivityType);
        // 向新Task中添加
        addOrReparentStartingActivity(task, "setTaskFromReuseOrCreateNewTask - mReuseTask");
        ...
    } else {
        // 向旧task中添加
        addOrReparentStartingActivity(mReuseTask, "setTaskFromReuseOrCreateNewTask");
    }

    ...
    return START_SUCCESS;
}
```

#### 总结

通过上面的代码分析，我们可以总结出在Activity启动过程中创建新Task的条件：

1. standard、singleTop模式   Intent.FLAG_ACTIVITY_NEW_TASK 和taskAffinity必须同时设置
2. sinlgeTask模式   只需设置taskAffinity，Intent.FLAG_ACTIVITY_NEW_TASK 可有可无
3. singeInstance Intent.FLAG_ACTIVITY_NEW_TASK 和taskAffinity均可有可无



# PMS

## 深入PMS源码（一）—— PMS的启动过程和执行流程

### PMS简介

作为Android开发者，或多或少的都接触过Android的framework层架构，这也是开发者从使用Android到了解安卓的过程，framework层的核心功能有AMS、PMS、WMS等，这三个也是系统中最基础的使用，在Android程序启动完成后回启动一系列的核心服务，AMS、PMS、WMS就是在此过程中启动的，之后的系列文章之后会一次介绍他们，本篇主要介绍PMS关于PMS文章共分为3篇，本篇最为首篇也是PMS的主要逻辑部分；

- PMS主要功能

1. 管理设备上安装的所有应用程序，并在系统启动时加载应用程序；
2. 根据请求的Intent匹配到对应的Activity、Provider、Service，提供包含包名和Component的信息对象；
3. 调用需要权限的系统函数时，检查程序是否具备相应权限从而保证系统安全；
4. 提供应用程序的安装、卸载的接口；

- PMS包管理

1. 应用程序层：使用getPackageManager（）获取包的管理对象PackageManager，PMS使用的也是Binder通信，PackageManager是从ServiceManager中获取注册的Binder对象，具体的实现为PackageManagerService，PMS实现对所有程序的安装和加载；
2. PMS服务层：PMS运行在SystemServer进程中，主要使用/system/etc/permissions.xml和/data/system/packages.xml管理包信息；
3. 数据文件管理：PMS负责对系统的配置文件、apk安装文件、apk的数据文件执行管理、读写、创建和删除等功能；
   （1）程序文件：所有系统程序的文件处于/system/app/目录下，第三方程序文件处于/data/app/目录下，在程序安装过程中PMS会将要安装的apk文件复制到/data/app/目录下，以包名命名apk文件并添加“-x”后缀，在文件更新时会修改后缀编号；
   （2）/data/dalvik-cache/ 目录保存了程序中的执行代码，在应用程序运行前PMS会从apk中提取dex文件并保存在该目录下，以便之后能快速运行；
   （3）对framework库文件，PMS会将其中所有的apk、jar文件中提取dex文件，将dex文件保存在/data/dalvik-cache/目录下；
   （4）应用程序所使用的数据文件：数据以键值对保存、数据库保存、File保存所产生的文件都保存带/data/data/xxx/目录下，PMS在程序卸载会删除相应文件；

- /data/system/packages.xml ：系统的配置文件，记录所有的应用程序的包管理信息，PMS根据此文件管理所有程序

1. last-platform-version：记录系统最后一次修改的版本信息；
2. permissions：保存系统中所有的权限信息列表，系统权限以androd开头、自定义权限以包名开头；
3. sigs：签名标签，一个程序只能有一个签名但可以有多个证书，包含count属性表示证书数量；
4. cert：表示签名证书，包含index、key属性，index表示证书的下标；
5. perms：表示一个程序中声明使用的权限列表，存在package标签之下；
6. package：包含一个应用程序的对应关系：
   （1）name：应用程序的包名
   （2）codePath：程序apk文件所在的路径
   （3）nativeLibraryPath：程序中使用的native文件路径，一般指程序包下的lib文件中导入的依赖
   （4）flags：表示应用程序的类型
   （5）it、ut：分别表示程序首次安装的install time、更新时间update time
   （6）userId：表示应用程序在Linux下的用户id
   （7）shareId：表示应用程序所共享的Linux用户Id，与userId互斥
   （8）installer：安装器的名称，在调用PackageManager.installPackage()方法时设置的名称

### PMS的启动过程

在Android系统启动过程中，程序会执行到SystemServer中，然后调用startBootstrapServices()方法启动核心服务，在startBootstrapServices（）方法中完成PMS的启动：

```java
private void startBootstrapServices() {
       mPackageManagerService = PackageManagerService.main(mSystemContext, installer,mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore); // 1、调用main（）创建PMS对象，注册Binder
    
mPackageManager = mSystemContext.getPackageManager(); //2、初始化PackageManager对象
      }
```

1. 调用PackageManagerService.main()方法，在main（）方法中创建PMS的对象，并向ServiceManager注册Binder

```java
public static PackageManagerService main(Context context, Installer installer, boolean factoryTest, boolean onlyCore) {
        // 1、创建PMS对象
        PackageManagerService m = new PackageManagerService(context, installer,factoryTest, onlyCore);
        m.enableSystemUserPackages();
        ServiceManager.addService("package", m); // 2、注册PMS对象到ServiceManager中
        final PackageManagerNative pmn = m.new PackageManagerNative(); // 3、 创建PMN对象
        ServiceManager.addService("package_native", pmn); // 4、注册PMN对象
        return m;
    }

```

1. 调用ContextImpl.getPackageManager（）获取PackageManager对象，getPackageManager（）中使用ActivityThread.getPackageManager()获取前面创建并注册的Binder对象，然后创建ApplicationPackageManager实例

```java
 @Override
public PackageManager getPackageManager() {
  IPackageManager pm = ActivityThread.getPackageManager(); // 3、初始化PackageManagerService的代理Binder对象
     if (pm != null) {
     return (mPackageManager = new ApplicationPackageManager(this, pm)); //创建Packagemanager的实例
     }
  return null;
}
```

1. 程序在获取PMS对象时会调用ActivityThread.getPackageManager()，从ServiceManager中获取Binder，并获取BInder代理对象PMS实例

```java
public static IPackageManager getPackageManager() {
    if (sPackageManager != null) {
        return sPackageManager;
    }
    IBinder b = ServiceManager.getService("package”);  // 获取注册Binder
    sPackageManager = IPackageManager.Stub.asInterface(b); // 获取IPackageManager代理对象，即PMS
    return sPackageManager;
}
```

从上面的3个过程可以得出以下结论：

1. PMS使用Binder通信机制，最终IPackageManager接口的实现类为PackageManagerService类；
2. 系统中获取的PackageManager对象具体实现的子类是ApplicationPackageManager对象；

### PMS构造函数

由上面的分析知道，在系统启动后程序执行PMS的构造函数创建对象，整个系统对程序的管理就从这里开始，先介绍下相关类和属性信息：

- PMS中属性

1. ArrayMap<String, PackageParser.Package> mPackages：在扫描程序文件目录时会将信息保存在Package对象中，然后将所有程序包名极其的package保存在此集合中；
2. Settings mSettings：保存整个系统信息的Setting对象；
3. ActivityIntentResolver mActivities：遍历所有程序的目录，并解析所有的注册清单文件，将提取所有的Intent-filter数据保存在对应的集合中；
4. ActivityIntentResolver mReceivers：同上
5. ServiceIntentResolver mServices：同上
6. ProviderIntentResolver mProviders：同上

- PackageParser：解析apk文件的主要类，执行解析操作；
- PackageParser.Package：PackageParser的内部类，保存apk文件中的解析信息，每个应用程序对应一个Package对象，属性信息如下：

1. String packageName：程序包名
2. String codePath： 软件包的路径
3. ApplicationInfo applicationInfo ：applicationInfo对象
4. final ArrayList permissions：申请权限的集合
5. final ArrayList activities：Activity标签解析的结合
6. final ArrayList receivers：Receiver标签解析的结合
7. final ArrayList providers ：Provider标签解析的结合
8. final ArrayList services ：Service标签解析的结合
9. Bundle mAppMetaData = null：注册清单中设置的信息
10. int mVersionCode：版本Code
11. String mVersionName：版本名称
12. int mCompileSdkVersion：Sdk版本

- Settings：PMS内部主要保存信息的类，主要属性如下：

1. mSettingsFilename：配置系统目录下的package.xml文件

```java
mSystemDir = new File(dataDir, "system"); // 
mSettingsFilename = new File(mSystemDir, "packages.xml"); //获取系统目录下package.xml文件
```

1. mBackupSettingsFilename：配置系统目录下的packages-backup.xml文件，一般在创建和修改package文件前，会先创建packages-backup保存原来信息，在操作读写后会删除此文件；

```java
mBackupSettingsFilename = new File(mSystemDir, "packages-backup.xml");
```

1. mPackageListFilename：配置系统目录下的packages.list文件，保存了所有应用程序列表，每一行对应一个应用程序

```java
mPackageListFilename = new File(mSystemDir, "packages.list”);
如：com.android.hai 10032 1 data/data/com.android.hai ，第一项：应用程序包名、第二项：Linux用户Id、第三项：1表示可以debug，0表示不能debug、第四项：程序数据文件目录
```

1. ArrayMap<String, PackageSetting> mPackages：解析package.xml文件中的每个程序信息保存在PackageSetting对象中，将所有程序的PackageSetting都填充到集合中
2. mDisabledSysPackages：保存那些没有经过正常程序卸载的应用程序列表，按照正常卸载程序时，PMS会自动删除package.xml文件中的信息，使用adb命令或其他方法删除时package文件信息则不会删除，系统启动时PMS会检查package文件，并检查对应的应用程序的文件目录，从而判断是否意外删除，如果意外删除则加入mDisabledSysPackages集合；
3. mUserIds：保存Linux下所有的用户Id列表
4. mPendingPackages：在解析每个PackageSetting时如果是使用sharedId，则将此Setting加入此集合，等相应的share-user标签后再补充Setting
5. mPastSignatures：保存所有的签名文件信息
6. mPermissions：保存所有的权限信息
7. ArraySet mInstallerPackages：已安装的应用软件包

#### PMS的工作过程

- 构造函数

```java
public PackageManagerService(Context context, Installer installer, // PMS的构造函数
        boolean factoryTest, boolean onlyCore) {
sUserManager = new UserManagerService(context, this, // 创建UserManagerService
new UserDataPreparer(mInstaller, mInstallLock, mContext, mOnlyCore), mPackages);
mSettings = new Settings(mPermissionManager.getPermissionSettings(), mPackages); //1、创建保存系统信息的Settings对象

mHandlerThread = new ServiceThread(TAG,Process.THREAD_PRIORITY_BACKGROUND, true );
mHandlerThread.start();
mHandler = new PackageHandler(mHandlerThread.getLooper()); // 2、使用HandlerThread初始化PackageHandler对象

mFirstBoot = !mSettings.readLPw(sUserManager.getUsers(false)); // 3、调用readLPw（）读取并解析配置package.xml
}

final int packageSettingCount = mSettings.mPackages.size();
for (int i = packageSettingCount - 1; i >= 0; i--) {
    PackageSetting ps = mSettings.mPackages.valueAt(i); // 4、遍历所有的程序的packageSettings对象
    if (!isExternal(ps) && (ps.codePath == null || !ps.codePath.exists())
            && mSettings.getDisabledSystemPkgLPr(ps.name) != null) { // 判断文件已经被删除的或程序已被卸载
        mSettings.mPackages.removeAt(i); // 移除对应的PackageSettings对象
        mSettings.enableSystemPackageLPw(ps.name); // 从mDisabledSysPackages集合中也移除此程序的Settings对象
    }
}
File frameworkDir = new File(Environment.getRootDirectory(), "framework”); // 5、获取系统的framework文件
Iterator<PackageSetting> pkgSettingIter = mSettings.mPackages.values().iterator();
while (pkgSettingIter.hasNext()) {
    PackageSetting ps = pkgSettingIter.next();
    if (isSystemApp(ps)) { // 6、保存已经安装的系统的应用程序
        mExistingSystemPackages.add(ps.name);
    }
}
scanDirTracedLI(frameworkDir, // 7、扫描framework文件目录
        mDefParseFlags
        | PackageParser.PARSE_IS_SYSTEM_DIR,
        scanFlags
        | SCAN_NO_DEX
        | SCAN_AS_SYSTEM
        | SCAN_AS_PRIVILEGED,
        0);
final File systemAppDir = new File(Environment.getRootDirectory(), "app");
scanDirTracedLI(systemAppDir, // 8、扫描系统app文件
        mDefParseFlags
        | PackageParser.PARSE_IS_SYSTEM_DIR,
        scanFlags
        | SCAN_AS_SYSTEM,
        0);
。。。。。。扫描各种目录下的文件信息
scanDirTracedLI(sAppInstallDir, 0, scanFlags | SCAN_REQUIRE_KNOWN, 0); //9、扫描安装app目录/data/app/

mSettings.writeLPr(); // 10、写入配置文件
Runtime.getRuntime().gc();
```

在启动程序后，PMS的所有工作基本都在构造函数中执行的，具体的执行过程见上面代码注释，这里列出几点主要的执行步骤：

1. 创建Settings对象，将PMS中的mPackage集合传入，此集合保存所有apk的解析数据
2. 调用readLPw（）方法解析系统配置文件package.xml
3. 调用scanDirTracedLI（）扫描系统app
4. 调用scanDirTracedLI（）扫描/data/app/下安装的第三方啊app
5. 执行mSettings.writeLPr()将扫描后的结果，重新写入配置文件

上面的几个主要过程即可实现PMS对所有安装程序的执行和管理，下面从源码的角度分析下PMS具体的执行细节；

#### 解析配置文件package.xml

- readLPw()

```java
 boolean readLPw(@NonNull List<UserInfo> users) {
        FileInputStream str = null; 
        if (mBackupSettingsFilename.exists()) { // 1、如果package-backUp.xml 文件存在，读取package-back.xml文件
                str = new FileInputStream(mBackupSettingsFilename); // 一般当写入配置时发生意外才会存在此文件
        }
     
str = new FileInputStream(mSettingsFilename); // 2、正常情况下读取package.xml文件信息
XmlPullParser parser = Xml.newPullParser(); // 3、使用PullParser解析xml文件
parser.setInput(str, StandardCharsets.UTF_8.name());

while ((type = parser.next()) != XmlPullParser.END_DOCUMENT
        && (type != XmlPullParser.END_TAG || parser.getDepth() > outerDepth)) { // 4、循环读取xml的节点数据
                    String tagName = parser.getName(); // 读取每个标签名称
3044                if (tagName.equals("package")) {
3045                    readPackageLPw(parser);  // 解析package标签中信息保存在PackageSetting对象中
3046                } else if (tagName.equals("permissions")) {
3047                    mPermissions.readPermissions(parser);  // 解析系统所有的权限信息，保存在PermissionSettings中
3048                } else if (tagName.equals("shared-user")) {
3051                    readSharedUserLPw(parser); // 如果使用share-user，解析此标签
                    }
 // 在解析package标签时，如果程序使用的shareUserId，则将此setting对象加入此集合，等相应共享的程序加载完成后再完善信息
 final int N = mPendingPackages.size();
3172        for (int i = 0; i < N; i++) { //  5、遍历mPendingPackage集合中的PackageSetting对象，逐步完善每个对象
3173            final PackageSetting p = mPendingPackages.get(i);
3174            final int sharedUserId = p.getSharedUserId();
3175            final Object idObj = getUserIdLPr(sharedUserId); // 获取次shareUserId对应的程序
3176            if (idObj instanceof SharedUserSetting) {
3177                final SharedUserSetting sharedUser = (SharedUserSetting) idObj;
3178                p.sharedUser = sharedUser; // 完善p的信息
3179                p.appId = sharedUser.userId;
3180                addPackageSettingLPw(p, sharedUser);  
3181            }
3192        }
3193        mPendingPackages.clear(); // 清除集合
}
}
```

文件在readLPw（）中首先判断mBackupSettingsFilename文件是否存在，前面提到当更新配置文件时，系统会先将package.xml文件重名为backup文件，然后创建package.xml并将mSettings内容写入文件，写入完成之后将back-up为难删除，如果此过程发生意外则系统会保留back-up文件，再此重启PMS时会优先读取backup文件，一般情况会直接读取package.xml，读取信息后使用Xml解析文件中信息，在解析中提取每个标签中的属性，这里需要注意的是package 标签，其中包好应用程序的基本信息，程序回调用readPackageLPw（）解析；

- readPackageLPw（）：解析并获取package标签下的信息，并创建PackageSetting对象保存信息；

```java
 private void readPackageLPw(XmlPullParser parser) throws XmlPullParserException, IOException {
name = parser.getAttributeValue(null, ATTR_NAME); // 1、提前标签下的属性信息，保存在相应的变量中
realName = parser.getAttributeValue(null, "realName");
idStr = parser.getAttributeValue(null, "userId");
uidError = parser.getAttributeValue(null, "uidError");
sharedIdStr = parser.getAttributeValue(null, "sharedUserId");
codePathStr = parser.getAttributeValue(null, "codePath");
.......
version = parser.getAttributeValue(null, "version");
timeStampStr = parser.getAttributeValue(null, "it");
timeStampStr = parser.getAttributeValue(null, "ut");
//2、创建PackageSetting对象保存从package标签解析出的信息，并保存在集合中mPackages集合中
packageSetting = addPackageLPw(name.intern(), realName, new File(codePathStr), 
3836                        new File(resourcePathStr), legacyNativeLibraryPathStr, primaryCpuAbiString,
3837                        secondaryCpuAbiString, cpuAbiOverrideString, userId, versionCode, pkgFlags,
3838                        pkgPrivateFlags, parentPackageName, null /*childPackageNames*/,
3839                        null /*usesStaticLibraries*/, null /*usesStaticLibraryVersions*/);
packageSetting.setTimeStamp(timeStamp);
packageSetting.firstInstallTime = firstInstallTime;
packageSetting.lastUpdateTime = lastUpdateTime;

if (sharedIdStr != null) {//3、如果使用sharedId，则创建packageSetting对象保存在mPending集合中，稍后补充解析信息
3853                if (sharedUserId > 0) {
3854                    packageSetting = new PackageSetting(name.intern(), realName, new File(
3855                            codePathStr), new File(resourcePathStr), legacyNativeLibraryPathStr,
3856                            primaryCpuAbiString, secondaryCpuAbiString, cpuAbiOverrideString,
3857                            versionCode, pkgFlags, pkgPrivateFlags, parentPackageName,
3858                            null /*childPackageNames*/, sharedUserId,
3859                            null /*usesStaticLibraries*/, null /*usesStaticLibraryVersions*/);
3860                    packageSetting.setTimeStamp(timeStamp);
3861                    packageSetting.firstInstallTime = firstInstallTime;
3862                    packageSetting.lastUpdateTime = lastUpdateTime;
3863                    mPendingPackages.add(packageSetting); // 4、将packageSetting保存在mPendingPackages集合中
3873            }
if (installerPackageName != null) {
    mInstallerPackages.add(installerPackageName); // 添加到已安装程序列表集合中
}
```

在readPackageLPw（）中首先从解析器parser中提取所有的标签信息，然后调用addPackageLPw（）方法保存这些属性，最后当程序使用shareId时先暂时将对象的解析 添加到mPendingPackages集合中，等共享的应用执行结束后再补充信息；

- addPackageLPw（）

```java
PackageSetting addPackageLPw(String name,......) {
598        PackageSetting p = mPackages.get(name); //从集合中取出对象
599        if (p != null) {
600            if (p.appId == uid) {
601                return p;
602            }
606        }
607    p = new PackageSetting(name, realName, codePath, resourcePath, // 创建PackageSetting对象
608                legacyNativeLibraryPathString, primaryCpuAbiString, secondaryCpuAbiString,
609                cpuAbiOverrideString, vc, pkgFlags, pkgPrivateFlags, parentPackageName,
610                childPackageNames, 0 /*userId*/, usesStaticLibraries, usesStaticLibraryNames);
611        p.appId = uid;
612        if (addUserIdLPw(uid, p, name)) {
613            mPackages.put(name, p); // 在mpackages中添加保存对象
614            return p;
615        }
616        return null;
617    }
}
```

addPackageLPw（）中直接创建PackageSetting对象，将解析的信息封装起来，然后以程序的name为Key将PackageSetting对象添加的mPackages集合中，那此时mPackages就保存了手机中所有app的应用信息；

#### 扫描安装的应用程序

- scanDirTracedLI()：扫描/data/app/目录文件下的所有apk文件信息，在scanDirTracedLI（）中直接调用scanDirLI（）方法

```java
private void scanDirLI(File scanDir, int parseFlags, int scanFlags, long currentTime) {
final File[] files = scanDir.listFiles(); // 1、获取目录下的文件集合,即所有的apk列表，然后逐个解析
try (ParallelPackageParser parallelPackageParser = new ParallelPackageParser(
        mSeparateProcesses, mOnlyCore, mMetrics, mCacheDir,
        mParallelPackageParserCallback)) { // 2、创建ParallelPackageParser对象
    int fileCount = 0;
    for (File file : files) {
        parallelPackageParser.submit(file, parseFlags); //3、遍历files集合，使用parallelPackageParser提交每个apk文件
        fileCount++;
    }
    for (; fileCount > 0; fileCount--) {
        ParallelPackageParser.ParseResult parseResult = parallelPackageParser.take();//4、取出每个File的扫描结果，执行scan
        scanPackageChildLI(parseResult.pkg, parseFlags, scanFlags,currentTime, null); 
  }
}
```

scanDirLI（）方法执行的逻辑很简单：

1. 遍历文件目录下的所有文件
2. 创建ParallelPackageParser对象，调用submit（）方法提交请求，执行解析每个apk文件
3. 调用parallelPackageParser.take()逐个取出每个解析的结果

到这里我们知道PMS是对/data/app/目录中所有apk文件进行解析，在之前的版本中会直接创建PackageParser对象执行解析，在Androip P版本中引入ParallelPackageParser类，使用线程池和队列执行程序的解析；

- ParallelPackageParser：内部使用线程池和队列执行文件目录中apk扫描解析

```java
private final BlockingQueue<ParseResult> mQueue = new ArrayBlockingQueue<>(QUEUE_CAPACITY); // 1、保存请求结果的队列，阻塞线程
private final ExecutorService mService = ConcurrentUtils.newFixedThreadPool(MAX_THREADS,
        "package-parsing-thread", Process.THREAD_PRIORITY_FOREGROUND); // 2、创建线程池
        
public void submit(File scanFile, int parseFlags) {
    mService.submit(() -> { // 3、线程池提交任务
        ParseResult pr = new ParseResult();
        try {
            PackageParser pp = new PackageParser();
            pp.setSeparateProcesses(mSeparateProcesses);
            pp.setOnlyCoreApps(mOnlyCore);
            pp.setDisplayMetrics(mMetrics);
            pp.setCacheDir(mCacheDir);
            pp.setCallback(mPackageParserCallback);
            pr.scanFile = scanFile;
            pr.pkg = parsePackage(pp, scanFile, parseFlags); // 执行文件的解析扫描，将结果封装在ParseResult中
        }
        try {
            mQueue.put(pr); // 4、将解析的结果添加到队列中
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            mInterruptedInThread = Thread.currentThread().getName();
        }
    });
}
public ParseResult take() {
    try {
        return mQueue.take(); // 从队列中取出解析结果
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        throw new IllegalStateException(e);
    }
}
```

ParallelPackageParser中利用线程池处理并发问题，执行多个apk文件的解析，并使用阻塞队列的方式同步线程的数据，在submit提交的任务run（）方法中，创建了PackageParser对象并调用parser（）方法解析apk，之后将解析的结果封装在ParseResult中，最后添加到mQueue队列中，PMS中在依次调用take（）方法从mQueue队列中获取执行的结果；

- PackageParser.parsePackage（）：解析每个apk文件

```java
public Package parsePackage(File packageFile, int flags, boolean useCaches)
        throws PackageParserException {
    Package parsed = useCaches ? getCachedResult(packageFile, flags) : null; // 1、从缓存中获取解析结果
    if (parsed != null) {
        return parsed;
    }
    if (packageFile.isDirectory()) { // 2、判断是文件还是文件夹，分别执行不同的方法；
        parsed = parseClusterPackage(packageFile, flags); // 执行文件夹解析
    } else {
        parsed = parseMonolithicPackage(packageFile, flags); // 执行单个apk文件解析，单个安装apk文件时执行
    }
    cacheResult(packageFile, flags, parsed); // 3、缓存解析的结果parser
    return parsed;
}
```

ParserPackage.parser（）中开始执行apk文件的解析，对于apk文件执行parseMonolithicPackage（），在执行解析结束后会缓存解析结果；

```java
public Package parseMonolithicPackage(File apkFile, int flags) throws PackageParserException {
    final PackageLite lite = parseMonolithicPackageLite(apkFile, flags);
    final SplitAssetLoader assetLoader = new DefaultSplitAssetLoader(lite, flags); // 1、使用AssetManager加载资源文件
    try {
        final Package pkg = parseBaseApk(apkFile, assetLoader.getBaseAssetManager(), flags); // 执行parseBaseAPk
        pkg.setCodePath(apkFile.getCanonicalPath());
        pkg.setUse32bitAbi(lite.use32bitAbi);
        return pkg;
    } 
}
```

- parseMonolithicPackageLite（）:先解析出apk文件的基本信息

```java
private static PackageLite parseMonolithicPackageLite(File packageFile, int flags) throws PackageParserException {
        final ApkLite baseApk = parseApkLite(packageFile, flags);
        final String packagePath = packageFile.getAbsolutePath();
    
        return new PackageLite(packagePath, baseApk, null, null, null, null, null, null);
    }

//先轻量级解析apk的基本信息
 private static ApkLite parseApkLite(String codePath, XmlPullParser parser, AttributeSet attrs,SigningDetails signingDetails)
            throws IOException, XmlPullParserException, PackageParserException {
        final Pair<String, String> packageSplit = parsePackageSplitNames(parser, attrs);
        for (int i = 0; i < attrs.getAttributeCount(); i++) {
            final String attr = attrs.getAttributeName(i);
            if (attr.equals("installLocation")) {
                installLocation = attrs.getAttributeIntValue(i,
                        PARSE_DEFAULT_INSTALL_LOCATION);
            } else if (attr.equals("versionCode")) {
                versionCode = attrs.getAttributeIntValue(i, 0);
            } else if (attr.equals("versionCodeMajor")) {
                versionCodeMajor = attrs.getAttributeIntValue(i, 0);
            } else if (attr.equals("revisionCode")) {
                revisionCode = attrs.getAttributeIntValue(i, 0);
            } 
            .....
        }
         ......
        return new ApkLite(codePath, packageSplit.first, packageSplit.second, isFeatureSplit,
                configForSplit, usesSplitName, versionCode, versionCodeMajor, revisionCode,
                installLocation, verifiers, signingDetails, coreApp, debuggable,
                multiArch, use32bitAbi, extractNativeLibs, isolatedSplits);
    }
```

在parseMonolithicPackage（）中先调用parseApkLite（）将File先简单的解析以下，这里的解析只是获取注册清单中的基础信息，并将信息保存在ApkLite对象中，然后将ApkLite和文件路径
封装在PackageLite对象中；

- DefaultSplitAssetLoader：内部使用AssestManager加载apk文件的资源，并缓存AssestManager对象信息，主要针对split APK；

- parseBaseApk（）：解析每个apk文件

```java
private Package parseBaseApk(File apkFile, AssetManager assets, int flags)
        throws PackageParserException {// 参数：apk文件File、apk文件路径destCodePath
        final String apkPath = apkFile.getAbsolutePath(); // 1、获取apk文件路径
         mArchiveSourcePath = sourceFile.getPath(); 
399        int cookie = assmgr.findCookieForPath(mArchiveSourcePath); // 2、将apk的路径添加到AssetManager中加载资源
401        parser = assmgr.openXmlResourceParser(cookie, "AndroidManifest.xml”); // 3、解析xml注册清单文件
           Resources res = new Resources(assmgr, metrics, null); // 4、创建Resource对象
           pkg = parseBaseApk(res, parser, flags, errorText); // 解析parser中的信息，保存在Package对象中
pkg.setVolumeUuid(volumeUuid);
pkg.setApplicationVolumeUuid(volumeUuid);
pkg.setBaseCodePath(apkPath);
pkg.setSigningDetails(SigningDetails.UNKNOWN);
return pkg;
}
```

parseBaseApk（）中从apkFile对象中获取apk文件路径，然后使用assmgr加载apk文件中的资源，从文件中读取注册清单文件，然后调用parseBaseApk解析注册清单；

1. parseBaseApk（）：从Parser对象中获取解析的信息，保存在Package对象中

```java
 private Package parseBaseApk(String apkPath,Resources res, XmlResourceParser parser, int flags, String[] outError){
Pair<String, String> packageSplit = parsePackageSplitNames(parser, parser); //1、从parser中解析出“package”设置的包名pkgName
pkgName = packageSplit.first;
splitName = packageSplit.second;
Package pkg = new Package(pkgName); // 2、创建Package对象，保存apk的包名
//从Resource中获取各种信息，并保存在Package的属性中
 pkg.mVersionCode = sa.getInteger(…...
pkg.mVersionName = sa.getNonConfigurationString(…...
 pkg.mSharedUserId = str.intern(…...);
pkg.mSharedUserLabel = sa.getResourceId(…...
pkg.installLocation = sa.getInteger(…...
pkg.applicationInfo.installLocation = pkg.installLocation;
return parseBaseApkCommon(pkg, null, res, parser, flags, outError); // 3、调用parseBaseApkCommon（）继续解析文件
}
```

parseBaseApk中从parser中提取注册清单中的基础信息，并封装保存在Pakage对象中，然后调用parseBaseApkCommon（）方法继续解析清单文件中内容

- parseBaseApkCommon（）

```java
//parseBaseApkCommon（）从Parser对象中解析数据信息
int outerDepth = parser.getDepth(); // 获取parser的深度
while ((type=parser.next()) != parser.END_DOCUMENT. // 1、循环解析parser对象
804               && (type != parser.END_TAG || parser.getDepth() > outerDepth)) {
 if (tagName.equals("application")) {
if (!parseBaseApplication(pkg, res, parser, attrs, flags, outError)) { // 2、解析application标签
825                    return null;
}else if (tagName.equals("permission")) { // 3、解析权限标签
832                if (parsePermission(pkg, res, parser, attrs, outError) == null) {
833                    return null;
834                }
} else if (tagName.equals("uses-feature")) { // 4、解析使用的user-feature标签，并保存在Package的集合中
 FeatureInfo fi = new FeatureInfo();
    …….
pkg.reqFeatures.add(fi);
}else if (tagName.equals("uses-sdk")) { // 解析user-sdk标签
}else if (tagName.equals("supports-screens")) { // 解析support-screens标签
}
}
}
```

parseBaseApkCommon（）中主要负责解析清单文件中的各种标签信息，其中最主要的就是解析标签下的四大组件的信息，在遇到applicaiton标签时直接调用了parseBaseApplication（）执行解析；

- parseBaseApplication（），主要的解析工作

```java
String tagName = parser.getName();
final ApplicationInfo ai = owner.applicationInfo;
ai.theme = sa.getResourceId(
        com.android.internal.R.styleable.AndroidManifestApplication_theme, 0);
ai.descriptionRes = sa.getResourceId(
        com.android.internal.R.styleable.AndroidManifestApplication_description, 0);
ai.maxAspectRatio = sa.getFloat(R.styleable.AndroidManifestApplication_maxAspectRatio, 0);

// 分别解析四大组件，将解析结果保存在Package对应的集合中
1644            if (tagName.equals("activity")) { 
1645                Activity a = parseActivity(owner, res, parser, attrs, flags, outError, false);
1651                owner.activities.add(a);
1653            } else if (tagName.equals("receiver")) {
1654                Activity a = parseActivity(owner, res, parser, attrs, flags, outError, true);
1660                owner.receivers.add(a);
1662            } else if (tagName.equals("service")) {
1663                Service s = parseService(owner, res, parser, attrs, flags, outError);
1669                owner.services.add(s);
1671            } else if (tagName.equals("provider")) {
1672                Provider p = parseProvider(owner, res, parser, attrs, flags, outError);
1678                owner.providers.add(p);
1680            }
```

清单文件解析共分两部分：

1. 解析出application标签下设置的name类名、icon、theme、targetSdk、processName等属性标签并保存在ApplicationInfo对象中
2. 循环解析activity、receiver、service、provider四个标签，并将信息到保存在Package中对应的集合中

下面逐个分析下四大组件是如何解析保存的：

- parserActivity（）：解析Activity和Receiver标签并返回Activity对象封装所有属性；

```java
private Activity parseActivity(Package owner,...)
        throws XmlPullParserException, IOException {
TypedArray sa = res.obtainAttributes(parser, R.styleable.AndroidManifestActivity);
cachedArgs.mActivityArgs.tag = receiver ? "<receiver>" : "<activity>”; // 1、判断为activity或receiver
cachedArgs.mActivityArgs.sa = sa;
cachedArgs.mActivityArgs.flags = flags;
Activity a = new Activity(cachedArgs.mActivityArgs, new ActivityInfo()); // 2、创建Activity实例，并初始化系列属性
a.info.theme = sa.getResourceId(R.styleable.AndroidManifestActivity_theme, 0);
a.info.taskAffinity = buildTaskAffinityName(owner.applicationInfo.packageName,
        owner.applicationInfo.taskAffinity, str, outError);
a.info.launchMode = ...
if (parser.getName().equals("intent-filter")) { // 3、解析intent-filter
    ActivityIntentInfo intent = new ActivityIntentInfo(a);
    if (!parseIntent(res, parser, true /*allowGlobs*/, true /*allowAutoVerify*/,
            intent, outError)) {
        return null;
    }
        a.order = Math.max(intent.getOrder(), a.order);
        a.intents.add(intent); // 将intent设置到Activity
} 
return a;
```

在解析Activity和Receiver标签时，当标签设置intent-filter时则创建一个ActivityIntentInfo对象，并调用parseIntent（）将intent-filter标签下的信息解析到ActivityIntentInfo中，并将ActivityIntentInfo对象保存在a.intents的集合中，简单的说一个intent-filter对应一个ActivityIntentInfo对象，一个Activity和Receiver可以包好多个intent-filter；

1. parseIntent()：解析每个intent-filter标签下的action、name等属性值，并将所有的属性值保存在outInfo对象中，这里的outInfo是ActivityIntentInfo对象；

```java
while ((type = parser.next()) != XmlPullParser.END_DOCUMENT
        && (type != XmlPullParser.END_TAG || parser.getDepth() > outerDepth)) {
    String nodeName = parser.getName(); // 1、获取节点名称
    if (nodeName.equals("action")) { //2、处理action节点
    String value = parser.getAttributeValue( // 3、获取action节点的name属性，并保存在outInfo属性中
        ANDROID_RESOURCES, "name");
    outInfo.addAction(value); //
} else if (nodeName.equals("category")) {
    String value = parser.getAttributeValue( // 4、获取category属性，保存在outInfo属性中
            ANDROID_RESOURCES, "name");
    outInfo.addCategory(value); // 添加到Category集合中
} else if (nodeName.equals("data")) {
sa = res.obtainAttributes(parser,
        com.android.internal.R.styleable.AndroidManifestData); // 解析parser为sa属性
String str = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestData_mimeType, 0);
        outInfo.addDataType(str); // 获取并保存mimeType中
str = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestData_scheme, 0);
    outInfo.addDataScheme(str); // 获取并保存scheme中
String host = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestData_host, 0); // 
String port = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestData_port, 0);
    outInfo.addDataAuthority(host, port); // 获取并保存host、port属性
outInfo.hasDefault = outInfo.hasCategory(Intent.CATEGORY_DEFAULT); // 设置Default属性
}
```

2. ActivityIntentInfo 继承 IntentInfo ，IntentInfo继承IntentFilter属性，所以outInfo保存的数据都保存在IntentFilter中，在IntentFilter中有标签对应的集合，如actions、mDataSchemes等，所以遍历的所有intent-filter中设置的数据都保存在其中；

```java
public final static class ActivityIntentInfo extends IntentInfo {}
public static abstract class IntentInfo extends IntentFilter {}
public final void addAction(String action) {
    if (!mActions.contains(action)) {
        mActions.add(action.intern()); // 在Intent-Filter中保存action
    }
}
public final void addCategory(String category) {
    if (mCategories == null) mCategories = new ArrayList<String>();
    if (!mCategories.contains(category)) {
        mCategories.add(category.intern()); // 
    }
}
public final void addDataScheme(String scheme) {
    if (mDataSchemes == null) mDataSchemes = new ArrayList<String>();
    if (!mDataSchemes.contains(scheme)) {
        mDataSchemes.add(scheme.intern()); // 
    }
}
```

- parserService（）：解析Service标签并返回Service对象，对应service标签下的信息保存在ServiceIntentInfo对象中，ServiceIntentInfo的作用和保存和ActivityIntentInfo一样

```java
TypedArray sa = res.obtainAttributes(parser,
        com.android.internal.R.styleable.AndroidManifestService);
cachedArgs.mServiceArgs.sa = sa;
cachedArgs.mServiceArgs.flags = flags;
Service s = new Service(cachedArgs.mServiceArgs, new ServiceInfo()); // 创建Service对象
if (parser.getName().equals("intent-filter")) { // 解析intent-filter节点
    ServiceIntentInfo intent = new ServiceIntentInfo(s);
    if (!parseIntent(res, parser, true /*allowGlobs*/, false /*allowAutoVerify*/,
            intent, outError)) {
        return null;
    }
    s.order = Math.max(intent.getOrder(), s.order);
    s.intents.add(intent);
} else if (parser.getName().equals("meta-data")) { // 解析meta-data数据，保存在bundle中
    if ((s.metaData=parseMetaData(res, parser, s.metaData,
            outError)) == null) {
        return null;
    }
}
```

- parserProvider（）：解析provider标签并返回provider对象

```java
TypedArray sa = res.obtainAttributes(parser,
        com.android.internal.R.styleable.AndroidManifestProvider);
cachedArgs.mProviderArgs.tag = "<provider>";
cachedArgs.mProviderArgs.sa = sa;
cachedArgs.mProviderArgs.flags = flags;

Provider p = new Provider(cachedArgs.mProviderArgs, new ProviderInfo()); // 创建Provider对象
p.info.exported = sa.getBoolean(
        com.android.internal.R.styleable.AndroidManifestProvider_exported,
        providerExportedDefault);
String permission = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestProvider_permission, 0);
p.info.multiprocess = sa.getBoolean(
        com.android.internal.R.styleable.AndroidManifestProvider_multiprocess,
        false);
p.info.authority = cpname.intern();
if (!parseProviderTags( // 调用parseProvider解析provider下的标签
        res, parser, visibleToEphemeral, p, outError)) {
    return null;
}
```

在解析provider标签中，创建ProviderInfo对象保存设置的属性信息，如export、permission等，然后调用parseProviderTags解析provider标签中使用的其他标签信息

1. parseProviderTags（）：

```java
private boolean parseProviderTags(Resources res, XmlResourceParser parser,
        boolean visibleToEphemeral, Provider outInfo, String[] outError){
    int outerDepth = parser.getDepth();
    int type;
    while ((type=parser.next()) != XmlPullParser.END_DOCUMENT
           && (type != XmlPullParser.END_TAG
                   || parser.getDepth() > outerDepth)) {
if (parser.getName().equals("intent-filter")) {
    ProviderIntentInfo intent = new ProviderIntentInfo(outInfo);
    if (!parseIntent(res, parser, true /*allowGlobs*/, false /*allowAutoVerify*/,
            intent, outError)) {
        return false;
    }
    outInfo.order = Math.max(intent.getOrder(), outInfo.order);
    outInfo.intents.add(intent);
} 
if (parser.getName().equals("meta-data")) {
    if ((outInfo.metaData=parseMetaData(res, parser,
            outInfo.metaData, outError)) == null) {
        return false;
    }
} 
if (parser.getName().equals("grant-uri-permission")) {
String str = sa.getNonConfigurationString(
        com.android.internal.R.styleable.AndroidManifestGrantUriPermission_path, 0);
PatternMatcher   pa = new PatternMatcher(str, PatternMatcher.PATTERN_LITERAL); // 创建PatterMatcher匹配权限
PatternMatcher[] newp = new PatternMatcher[N+1];
newp[N] = pa;
outInfo.info.uriPermissionPatterns = newp; // 保存临时权限数组
}
if (parser.getName().equals("path-permission")) { // 保存路径权限
newp[N] = pa;
outInfo.info.pathPermissions = newp;
}
}
}
```

主要解析如下：

1. 对于intent-filter标签，调用parserIntent（）解析保存在ProviderIntentInfo对象中，并添加到intent集合中
2. 解析设置的meta-data值
3. 解析grant-uri-permission和path-permission等权限的匹配状况保存在Provider对象中的数组中；

到此apk文件已经解析完成，相应的文件和属性都封装在Package对象中，并都保存在PMS属性集合mPackages集合中；

#### 将apk解析数据同步到PMS的属性中

在使用线程池执行所有的apk解析后，所有的解析结果都保存在队列中，系统会循环调用take（）方法取出解析的结果

```java
for (; fileCount > 0; fileCount--) {
                ParallelPackageParser.ParseResult parseResult = parallelPackageParser.take();
                
scanPackageChildLI(parseResult.pkg, parseFlags, scanFlags,
                                    currentTime, null);
                }
```

取出apk文件解析结果后，调用scanPackageChildLI（）扫描获取到的ParseResult中的Pakcage对象，scanPackageChildLI（）中直接调用addForInitLI（）方法

- addForInitLI（）

```java
synchronized (mPackages) {
final PackageSetting installedPkgSetting = mSettings.getPackageLPr(pkg.packageName); // 1、获取从配置文件中读取的Settings

final PackageParser.Package scannedPkg = scanPackageNewLI(pkg, parseFlags, scanFlags | SCAN_UPDATE_SIGNATURE, currentTime, user)； // 调用scanPackageNewLI将pkg中的数据保存到PMS的变量中
}

// scanPackageNewLI（）：创建ScanRequest对象，执行文件扫描修改或更新对应的PackageSetting对象
final ScanRequest request = new ScanRequest(pkg, sharedUserSetting,
        pkgSetting == null ? null : pkgSetting.pkg, pkgSetting, disabledPkgSetting,
        originalPkgSetting, realPkgName, parseFlags, scanFlags,
        (pkg == mPlatformPackage), user);
final ScanResult result = scanPackageOnlyLI(request, mFactoryTest, currentTime); 
if (result.success) {
    commitScanResultsLocked(request, result); // 调用commitScanResultsLocked（）
}
```

commitScanResultsLocked（）中直接调用commitPackageSettings（）调教apk的解析数据

```java
commitPackageSettings(pkg, oldPkg, pkgSetting, user, scanFlags,
(parseFlags & PackageParser.PARSE_CHATTY) != 0 /*chatty*/); // 提交包解析数据
```

- commitPackageSettings（）：将解析得到的Package中的信息保存到PMS内部变量中，并创建程序包所需的各种文件信息

```java
final String pkgName = pkg.packageName;
if (pkg.packageName.equals("android")) { // 1、针对系统包，特殊处理属性的初始化
                   mPlatformPackage = pkg;
2897                pkg.mVersionCode = mSdkVersion;
2898                mAndroidApplication = pkg.applicationInfo;
2899                mResolveActivity.applicationInfo = mAndroidApplication;
2900                mResolveActivity.name = ResolverActivity.class.getName();
。。。。。。
2912                mResolveComponentName = new ComponentName(
2913             mAndroidApplication.packageName, mResolveActivity.name);
}
                int N = pkg.usesLibraries != null ? pkg.usesLibraries.size() : 0;
2950                for (int i=0; i<N; i++) {
2951                String file = mSharedLibraries.get(pkg.usesLibraries.get(i)); 
                    mTmpSharedLibraries[num] = file;   // 遍历所有的user-library标签保存在数组中
2960                num++;
2961                }
            if (!verifySignaturesLP(pkgSetting, pkg)) { // 校验签名文件
  ……. 
}
                int N = pkg.providers.size();
3404            StringBuilder r = null;
3405            int i;
3406            for (i=0; i<N; i++) {
3407            PackageParser.Provider p = pkg.providers.get(i); //遍历提取每个provider
3408                p.info.processName = fixProcessName(pkg.applicationInfo.processName,
3409                        p.info.processName, pkg.applicationInfo.uid);
3410                mProvidersByComponent.put(new ComponentName(p.info.packageName,
3411                        p.info.name), p); // 针对Provider创建ComponentName对象，保存在mProvidersByComponent集合中
3413                if (p.info.authority != null) {
3414                    String names[] = p.info.authority.split(";”); // 获取Provider的权限信息
3415                    p.info.authority = null;
3416                    for (int j = 0; j < names.length; j++) {
3417                        if (j == 1 && p.syncable) {
3425                            p = new PackageParser.Provider(p);
3426                            p.syncable = false;
3427                        }
3428                        if (!mProviders.containsKey(names[j])) {
3429                            mProviders.put(names[j], p); // 将权限和对应的Provider以键值对保存在 mProviders 集合中
3447                    }
3448                }
3457            }
3460            }

N = pkg.services.size();
3463            r = null;
3464            for (i=0; i<N; i++) {
3465                PackageParser.Service s = pkg.services.get(i);
3466                s.info.processName = fixProcessName(pkg.applicationInfo.processName,
3467                        s.info.processName, pkg.applicationInfo.uid);
3468                mServices.addService(s);
N = pkg.receivers.size();
3483            r = null;
3484            for (i=0; i<N; i++) {
3485                PackageParser.Activity a = pkg.receivers.get(i);
3486                a.info.processName = fixProcessName(pkg.applicationInfo.processName,
3487                        a.info.processName, pkg.applicationInfo.uid);
3488                mReceivers.addActivity(a, "receiver");
3497            }

 N = pkg.activities.size();
3503            r = null;
3504            for (i=0; i<N; i++) {
3505                PackageParser.Activity a = pkg.activities.get(i);
3506                a.info.processName = fixProcessName(pkg.applicationInfo.processName,
3507                        a.info.processName, pkg.applicationInfo.uid);
3508                mActivities.addActivity(a, "activity");
}

```

在commitPackageSettings中，主要是将每个apk文件获得的Package对象中保存的四大组件信息分别提取保存在PMS内部对应的属性中，在PMS内部有4个专门储存四大组件的属性：

```java
final ActivityIntentResolver mActivities = new ActivityIntentResolver();
final ActivityIntentResolver mReceivers = new ActivityIntentResolver();
final ServiceIntentResolver mServices = new ServiceIntentResolver();
final ProviderIntentResolver mProviders = new ProviderIntentResolver();
```

- ActivityIntentResolver.addActivity()：处理Activity和Receiver分别保存在各自的ArrayMap中

```java
public final void addActivity(PackageParser.Activity a, String type) {
    mActivities.put(a.getComponentName(), a); // 1、获取内部的Component对象，在Activity会自动创建Component对象
    final int NI = a.intents.size();
    for (int j=0; j<NI; j++) {
        PackageParser.ActivityIntentInfo intent = a.intents.get(j); // 获取Activity中设置的intent
        addFilter(intent); // 添加Intent过滤
    }
}
```

- ServiceIntentResolver.addActivity()：将Package中极细获取的Service对象，保存在ArrayMap中

```java
public final void addService(PackageParser.Service s) {
    mServices.put(s.getComponentName(), s); //保存service
    final int NI = s.intents.size();
    int j;
    for (j=0; j<NI; j++) {
        PackageParser.ServiceIntentInfo intent = s.intents.get(j);
        addFilter(intent);
    }
}
```

- ProviderIntentResolver.addActivity()：将Package中极细获取的Provider对象，保存在ArrayMap中

```java
public final void addProvider(PackageParser.Provider p) {
    if (mProviders.containsKey(p.getComponentName())) {
        return;
    }
    mProviders.put(p.getComponentName(), p); // 保存provider
  
    final int NI = p.intents.size();
    int j;
    for (j = 0; j < NI; j++) {
        PackageParser.ProviderIntentInfo intent = p.intents.get(j);
        addFilter(intent); // 添加查找过滤的intent
    }
}
```

#### 更新配置文件

- mSettings.writeLPr（）：将mPackages中的数据分别写入package.xml和package.list文件中

```java
  void writeLPr() {
        if (mSettingsFilename.exists()) {
            if (!mBackupSettingsFilename.exists()) { // 重命名文件
                if (!mSettingsFilename.renameTo(mBackupSettingsFilename)) {
                    Slog.wtf(PackageManagerService.TAG,
                            "Unable to backup package manager settings, "
                            + " current changes will be lost at reboot");
                    return;
                }
            } else {
                mSettingsFilename.delete(); //删除package文件
            }
        }
FileOutputStream fstr = new FileOutputStream(mSettingsFilename);
BufferedOutputStream str = new BufferedOutputStream(fstr);
 。。。。。。
 for (final PackageSetting pkg : mPackages.values()) {
 writePackageLPr(serializer, pkg); // 写入配置信息
 }
 .......
}
writePackageListLPr(); // 更新package.list文件
```

1. 先判断package.xml和backup.xml文件是否存在，如果两个都存在则删除package.xml
2. 如果backup.xml文件不存在，则将package.xml重命名为backup。xml
3. 创建新的package.xml文件，并将mSetting中的内容写入文件夹
4. 删除backup文件，并重新生成package.list文件

到此PMS的启动过程介绍完毕，简单来说系统在启动会会创建PMS对象，使用PMS对象读取配置文件，然后扫描手机上所有的app程序，并将所有的程序的内容信息都封装在Package对象中，然后将Package集合中信息转换为PMS的属性供系统使用，最后并更新配置文件；

## 深入PMS源码（二）—— APK的安装和卸载源码分析

### 应用程序安装基础

- 单个APK程序安装的过程

1. 把原始的APk文件复制到程序相应的目录文件下，对于第三方app复制到/data/app/目录下
2. 为程序创建相应的数据目录、提取dex文件、修改系统包管理信息

- 程序安装过程

1. 在程序目录下创建以包名称命名的程序apk文件File
2. 在data/data/目录下创建应用程序的数据文件
3. 将程序所有的信息写入到配置文件package.xml文件中

- HandlerParams主要实现子类

1. InstallParams：完成要安装程序的Copy工作
2. MoveParams：实现对已安装程序移动到外部保存目录中

```java
  abstract class HandlerParams { // 抽象类
4676        final static int MAX_RETRIES = 4; // 最多重试4次
4677        int retry = 0;
4678        final boolean startCopy() { // 提供文件Copy方法
4679            try {
4681                retry++;
4682                if (retry > MAX_RETRIES) { // 尝试超过4次报错
4684                    mHandler.sendEmptyMessage(MCS_GIVE_UP); // 超过4次后发送报错信息，回调错误方法
4685                    handleServiceError();
4686                    return false;
4687                } else {
4688                    handleStartCopy(); // 子类需重写，执行文件复制操作
                        res = true;
4691                }
4692            } catch (RemoteException e) {
4694                mHandler.sendEmptyMessage(MCS_RECONNECT);
4695            }
4696            handleReturnCode();
                return res;
4697        }
4699        final void serviceError() {
4701            handleServiceError();
4702            handleReturnCode();
4703        }
4704        abstract void handleStartCopy() throws RemoteException; // 抽象方法
4705        abstract void handleServiceError();
4706        abstract void handleReturnCode();
    }
```

由上面源码知道，HandlerParams是抽象类，内部提供程序文件的复制功能，执行将安装的apk文件复制到程序的制定安装位置，程序调用startCopy（）后开始执行文件赋值，具体赋值调用抽象方法handleStartCopy（），所以具体的功能子类需要重写此方法，在copy发生异常时会发送Handler消息实现重试，最多重试4次后抛出异常；

### PMS中APK安装过程

程序的安装执行到PMS中后，从Packagemanager的installpackage（）方法开始，间接调用到PMS中installStage（）函数，installStage（）中首先创建InstallParams对象，保存要安装的文件URI和其他信息，然后发送INIT_COPY异步消息启动文件复制

```java
public void installStage(….) {
4590        Message msg = mHandler.obtainMessage(INIT_COPY); // 创建INIT_COPY消息
4591        msg.obj = new InstallParams(packageURI, observer, flags,
4592                installerPackageName); // 创建InstallParams对象封装安装包属性信息
4593        mHandler.sendMessage(msg);
4594    }
```

Handler在处理INIT_COPY消息时，根据mBound变量确定是否绑定了MSC服务，未绑定的先调用connectToService（）执行绑定过程，然后将HandlerParams加入mPendingInstalls集合中等待执行，在服务绑定后发送MCS_BOUND消息，对于已经绑定的直接发送MSC_BOUND事件执行任务

```java
                    if (!mBound) { // 1、判断服务是否绑定
457                        if (!connectToService()) { // 2、执行Service绑定
459                            params.serviceError();
460                            return;
461                        } else {
464                            mPendingInstalls.add(idx, params); // 3、添加要执行的任务
465                        }
466                    } else {
467                        mPendingInstalls.add(idx, params); // 3、添加任务
470                        if (idx == 0) {
471                            mHandler.sendEmptyMessage(MCS_BOUND); // 4、就一个事件时，直接发送事件立即执行
472                        }
473                    }
```

在处理MCS_BOUND消息时，使用获取的Binder服务从mPendingInstalls列表中取出安装信息HandlerParams类中，

```java
if (msg.obj != null) {
mContainerService = (IMediaContainerService) msg.obj; // 1、获取对应的服务
}
HandlerParams params = mPendingInstalls.get(0); // 2、取出要安装的参数
if (params != null) {
If（params.startCopy()）{ // 3、执行Copy文件，最终执行到FileInstallArgs.copyApk()
    if (mPendingInstalls.size() > 0) {
        mPendingInstalls.remove(0);  // 删除集合中的数据
    }
}; 
}
```

在Handler处理事件时，从消息msg中获取IMediaContainerService服务对象，并从mPendingInstalls集合中获取要执行的任务，此处获取的是前面创建的InstallParams对象，然后调用params.startCopy()方法执行文件复制，按照HandlerParams的源码程序会调用InstallParams.handleStartCopy()方法；

```java
// InstallParams：实现全新安装时的复制过程
 class InstallParams extends HandlerParams {
      public void handleStartCopy() throws RemoteException {
            mArgs = createInstallArgs(this); // 1、
4834        if (ret == PackageManager.INSTALL_SUCCEEDED) {
4837        ret = mArgs.copyApk(mContainerService, true); // 2、
4838        }
      }
}
```

在InstallParams中主要执行两个过程：

1. 根据Params类型创建InstallerArgs对象保存请求数据
2. 调用InstallArgs.copyApk()方法实现复制

```java
 private InstallArgs createInstallArgs(InstallParams params) {
4931        if (installOnSd(params.flags)) {
4932            return new SdInstallArgs(params);//复制到外部存储时使用
4933        } else {
4934            return new FileInstallArgs(params); // 复制到内部存储时使用，在FileInstallArgs内部调用IMediaContainerService服务执行Copy过程
    }
}
```

InstallArgs本身是一个抽象类，内部包含了很多属性信息，它的实现类主要有两个FileInstallArgs和SdInstallArgs，FileInstallArgs主要实现将文件复制到内部存储，而SdInstallArgs执行复制到外部文件存储，这里会创建FileInstallArgs对象，在FileInstallArgs.copy()中直接调用doCopyApk（）；

- FileInstallArgs.copyApk（）

```java
int doCopyApk(IMediaContainerService imcs, boolean temp) throws RemoteException {
// 1、创建临时文件路径：data/app/vmd.sessionId.tmp 文件目录
final File tempDir = mInstallerService.allocateStageDirLegacy(volumeUuid, isEphemeral);
codeFile = tempDir;
resourceFile = tempDir;
final IParcelFileDescriptorFactory target = new IParcelFileDescriptorFactory.Stub() {
    @Override
    public ParcelFileDescriptor open(String name, int mode) throws RemoteException {
        try {
            final File file = new File(codeFile, name); // 创建apk文件
            final FileDescriptor fd = Os.open(file.getAbsolutePath(),
                    O_RDWR | O_CREAT, 0644);
            Os.chmod(file.getAbsolutePath(), 0644);
            return new ParcelFileDescriptor(fd); //2、以临时File创建ParcelFileDescriptor对象，用于进程通信
        } 
    }
};
// 3、调用IMS方法实现文件Copy，将apk文件复制到temp文件中，然后将tmp文件复制到data/app/下，删除临时tmp文件
ret = imcs.copyPackage(origin.file.getAbsolutePath(), target);
}
```

在doCopyApk（）中，首先在data/app/目录下创建临时目录data/app/vmd.sessionId.tmp 文件，然后实例化IParcelFileDescriptorFactory.Stub()对象，并以临时文件File创建ParcelFileDescriptor类用于Binder通信，最后调用imcs.copyPackage（）执行文件复制，这里imcs实际是DefaultContainerService的对象；

- DefaultContainerService.copyPackage（）

```java
final File packageFile = new File(packagePath); // 1、获取要复制的apk文件
pkg = PackageParser.parsePackageLite(packageFile, 0); // 2、parser程序包
return copyPackageInner(pkg, target); // 3、执行文件复制

private int copyPackageInner(PackageLite pkg, IParcelFileDescriptorFactory target)
        throws IOException, RemoteException {
    copyFile(pkg.baseCodePath, target, "base.apk”); // 1、复制apk文件
    if (!ArrayUtils.isEmpty(pkg.splitNames)) {
        for (int i = 0; i < pkg.splitNames.length; i++) {
            copyFile(pkg.splitCodePaths[i], target, "split_" + pkg.splitNames[i] + ".apk”); // 2、复制拆分的apk
        }
    }
return PackageManager.INSTALL_SUCCEEDED;
}
```

在copyPackage（）方法中，首先根据文件路径获取apk文件，调用PackageParser解析apk文件，这里的解析主要是看apk是否包含split apk，然后调用copyPackageInner执行文件复制工作，在copyPackageInner中调用copyFile（）

```java
private void copyFile(String sourcePath, IParcelFileDescriptorFactory target, String targetName)
        throws IOException, RemoteException {
    InputStream in = null;
    OutputStream out = null;
    try {
        in = new FileInputStream(sourcePath);
        out = new ParcelFileDescriptor.AutoCloseOutputStream(
        target.open(targetName, ParcelFileDescriptor.MODE_READ_WRITE)); // 回调target
        FileUtils.copy(in, out); // 执行文件复制，在tmp/base.apk/
    } finally {
        IoUtils.closeQuietly(out);
        IoUtils.closeQuietly(in);
    }
}
```

在copyFile（）中，首先回调target的open（）方法完成ParcelFileDescriptor的创建，然后调用FileUtils.copy（）方法，将apk文件复制到tmp/base.apk/中，由前面的HandlerParam知道apk复制成功之后，回调InstallParams.handleReturnCode（），handleReturnCode（）中直接调用processPendingInstall（）方法

```java
private void processPendingInstall(final InstallArgs args, final int currentStatus) {
    mHandler.post(new Runnable() { //1、mHandler发送延时任务
        public void run() {
            mHandler.removeCallbacks(this); //移除其它任务
            PackageInstalledInfo res = new PackageInstalledInfo();
            res.setReturnCode(currentStatus);
            res.uid = -1;
            res.pkg = null;
            res.removedInfo = null;
            if (res.returnCode == PackageManager.INSTALL_SUCCEEDED) {
                args.doPreInstall(res.returnCode); //2、执行安装前处理
                synchronized (mInstallLock) {
                    installPackageTracedLI(args, res); //3、执行安装和解析
                }
                args.doPostInstall(res.returnCode, res.uid);//4、执行安装后的扫尾工作
            }
    });
}
```

在processPendingInstall（）中直接发送handler事件执行APK的安装工作，在Handler事件中首先创建PackageInstalledInfo对象，保存安装的状态，在复制成功后执行以下操作：

1. 调用args.doPreInstall（）执行安装apk前准备；
2. 调用installPackageTracedLI(args, res)执行apk的解析和安装工作
3. 调用args.doPostInstall（）执行apk文件安装完成后的工作

- args.doPreInstall(res.returnCode)：如果复制状态不为Success，则清除复制的文件

```java
int doPreInstall(int status) {
    if (status != PackageManager.INSTALL_SUCCEEDED) {//未复制成功清除文件
        cleanUp();
    }
    return status; 
}
```

- installPackageTracedLI（）中直接调用installPackageLI（）

```java
private void installPackageLI(InstallArgs args, PackageInstalledInfo res) {
    final int installFlags = args.installFlags;
    final String installerPackageName = args.installerPackageName;
    final File tmpPackageFile = new File(args.getCodePath());
    PackageParser pp = new PackageParser();
    pkg = pp.parsePackage(tmpPackageFile, parseFlags); // 1、解析apk文件
//检查apk
//校验签名
// 2、/data/app/vmdl18300388.tmp/base.apk，重命名为/data/app/包名-1/base.apk。
if (!args.doRename(res.returnCode, pkg, oldCodePath)) { 
    return;
}
if (replace) {
replacePackageLIF(pkg, parseFlags, scanFlags, args.user,
            installerPackageName, res, args.installReason); // 3、替换apk
} else {
installNewPackageLIF(pkg, parseFlags, scanFlags | SCAN_DELETE_DATA_ON_FAILURES,
        args.user, installerPackageName, volumeUuid, res, args.installReason); // 3、安装新的apk
}
}
```

在installPackageLI（）中主要执行以下操作：

1. 创建PackageParser对象解析复制的临时apk文件；
2. 检查复制的apk文件并校验签名信息等；
3. 将复制的apk文件目录，重命名为以包名为目录的文件
4. 确认是已安装的apk更新还是新安装的app，对于新安装的app执行installNewPackageLIF（）方法

- installNewPackageLIF（）：安装新的apk

```java
PackageParser.Package newPackage = scanPackageTracedLI(pkg, parseFlags, scanFlags, // 重新扫描apk文件
        System.currentTimeMillis(), user);
updateSettingsLI(newPackage, installerPackageName, null, res, user, installReason); // 更新Setting
if (res.returnCode == PackageManager.INSTALL_SUCCEEDED) {
    prepareAppDataAfterInstallLIF(newPackage); // 为安装好的App准备数据
}
```

在installNewPackageLIF（）中首先调用scanPackageTracedLI（）重新扫描apk文件，scanPackageTracedLI内部会调用scanPackageNewLI（）提交解析结果，将解析的结果Package对象添加到PMS的属性中，详细流程参见深入PMS源码（一）—— PMS的启动过程和执行流程，在安装完成后调用updateSettingsLI（）更新mSetting属性，最后调用prepareAppDataAfterInstallLIF（）为新安装的程序准备数据；

```java
scannedPkg = scanPackageNewLI(pkg, parseFlags, scanFlags, currentTime, user);
```

到此程序的安装就完成了，上面主要介绍了Android系统是如何实现安装和文件复制的，在复制完成后，就会向PMS启动过程一样解析和处理apk文件；

### 应用程序的卸载过程

- 应用程序的卸载

```java
packageManager.packageInstaller.uninstall() //卸载程序
```

由深入PMS源码（一）—— PMS的启动过程和执行流程分析知道此处packageManager获取的是ApplicationPackageManager，而packageInstaller是在ApplicationPackageManager内部实例化的PackageInstaller的对象

```java
@Override
    public PackageInstaller getPackageInstaller() {
        synchronized (mLock) {
            if (mInstaller == null) {
                try {
                    mInstaller = new PackageInstaller(mPM.getPackageInstaller(),
                            mContext.getPackageName(), mContext.getUserId());
                }
            }
            return mInstaller;
        }
    }
```

在创建PackageInstaller对象中，调用mPM.getPackageInstaller()调用PMS中方法获取PackageInstallerService对象，并将其封装在PackageInstaller对象中，程序继续调用PackageInstaller.uninstall()中

```java
public void uninstall(@NonNull String packageName, @DeleteFlags int flags,
            @NonNull IntentSender statusReceiver) {
        uninstall(new VersionedPackage(packageName, PackageManager.VERSION_CODE_HIGHEST),
                flags, statusReceiver);
    }
```

uninstall（）中首先根据传入的包名创建VersionedPackage对象，然后继续调用uninstall的重载方法

```java
 @RequiresPermission(anyOf = {
            Manifest.permission.DELETE_PACKAGES,
            Manifest.permission.REQUEST_DELETE_PACKAGES})
    public void uninstall(@NonNull VersionedPackage versionedPackage, @DeleteFlags int flags,
            @NonNull IntentSender statusReceiver) {
        try {
            mInstaller.uninstall(versionedPackage, mInstallerPackageName,
                    flags, statusReceiver, mUserId); //2、
        } catch (RemoteException e) {
            throw e.rethrowFromSystemServer();
        }
    }
```

在uninstall（）中直接调用mInstaller.uninstall()，这里的mInstaller对象就是在创建PackageInstaller时传入的第一个参数，即PMS中实例话的PackageInstallerService对象，PackageInstallerService实现了IPackageInstaller.Stub类，可用于进程间通信

```java
public class PackageInstallerService extends IPackageInstaller.Stub {
 @Override
    public void uninstall(VersionedPackage versionedPackage, String callerPackageName, int flags,IntentSender statusReceiver, int userId) throws RemoteException {
         mPermissionManager.enforceCrossUserPermission(callingUid, userId, true, true, "uninstall"); // 1、检查权限
          final PackageDeleteObserverAdapter adapter = new PackageDeleteObserverAdapter(mContext,
                statusReceiver, versionedPackage.getPackageName(),
                isDeviceOwnerOrAffiliatedProfileOwner, userId);  //2、
                     
                }
                if (mContext.checkCallingOrSelfPermission(android.Manifest.permission.DELETE_PACKAGES)== PackageManager.PERMISSION_GRANTED) {
            mPm.deletePackageVersioned(versionedPackage, adapter.getBinder(), userId, flags);
        } 
}
```

在PackageInstallerService.install（）权限中，主要执行3个流程：

1. 创建PackageDeleteObserverAdapter对象，用于接收响应删除的结果
2. 检查当前程序是否具有删除权限
3. 调用PMS的deletePackageVersioned（）方法执行程序的卸载

从上面分析应用程序真正开始卸载，是从PackageManager的deletePackageAsUser（）函数开始，间接调用PMS的deletePackageVersioned（）方法

```java
@Override
public void deletePackageVersioned(VersionedPackage versionedPackage,
  final IPackageDeleteObserver2 observer, final int userId, final int deleteFlags) {
              mHandler.post(new Runnable() { //发送Handler事件
            public void run() {
                mHandler.removeCallbacks(this); // 移除事件
                int returnCode;
                final PackageSetting ps = mSettings.mPackages.get(internalPackageName);//获取PackageSettings对象
                boolean doDeletePackage = true;
                if (ps != null) {
                    final boolean targetIsInstantApp =
                            ps.getInstantApp(UserHandle.getUserId(callingUid));
                    doDeletePackage = !targetIsInstantApp
                            || canViewInstantApps;
                }
                if (doDeletePackage) {
                    if (!deleteAllUsers) {
                        returnCode = deletePackageX(internalPackageName, versionCode,
                                userId, deleteFlags);
                    } 
               }
                try {
       observer.onPackageDeleted(packageName, returnCode, null);// 删除后回调
                } 
            } 
        }); 
}
```

在deletePackageVersioned（）中发送Post事件执行异步删除操作，在Handler事件中调用deletePackageX（）方法

```java
 int deletePackageX(String packageName, long versionCode, int userId, int deleteFlags) {
  final PackageRemovedInfo info = new PackageRemovedInfo(this);
   if (isPackageDeviceAdmin(packageName, removeUser)) {
            Slog.w(TAG, "Not removing package " + packageName + ": has active device admin");
return PackageManager.DELETE_FAILED_DEVICE_POLICY_MANAGER;
        }
info.origUsers = uninstalledPs.queryInstalledUsers(allUsers, true);
 res = deletePackageLIF(packageName, UserHandle.of(removeUser), true, allUsers,
                        deleteFlags | PackageManager.DELETE_CHATTY, info, true, null);

}
```

在deletePackageX（）中首先判断要卸载的程序是否是admin管理的，如果是则直接return，如果不是则执行deletePackageLIF（）方法继续卸载程序

- deletePackageLIF（）

```java
 p = mPackages.get(packageName); //1、获取保存应用程序的Package对象
if (isSystemApp(p)) {
ret = deleteSystemPackageLI(p, flags, outInfo, writeSettings); //2、如果是系统app
6376        } else {
// 3、处理第三方app卸载
ret = deleteInstalledPackageLIF(p, deleteCodeAndResources, flags, outInfo,
6381                    writeSettings);
6382        }
```

deletePackageLIF（）中根据app类型区分系统app和第三方app，执行不同的卸载程序，对于第三方app执行deleteInstalledPackageLIF（）方法，在deleteInstalledPackageLIF（）中也是直接调用removePackageDataLI（）

- removePackageDataLIF（）

```java
  private void removePackageDataLI(PackageParser.Package p, PackageRemovedInfo outInfo,int flags, boolean writeSettings) {
6183        String packageName = p.packageName;
6187        removePackageLI(p, (flags&REMOVE_CHATTY) != 0); // 1、移除mPackages集合中保存的package对象

if ((flags & PackageManager.DELETE_KEEP_DATA) == 0) {
            final PackageParser.Package resolvedPkg;
                resolvedPkg = new PackageParser.Package(ps.name);
                resolvedPkg.setVolumeUuid(ps.volumeUuid);
            }
            destroyAppDataLIF(resolvedPkg, UserHandle.USER_ALL,
                    StorageManager.FLAG_STORAGE_DE | StorageManager.FLAG_STORAGE_CE); // 2、调用Installer执行清除/data/app/目录下的文件信息
            destroyAppProfilesLIF(resolvedPkg, UserHandle.USER_ALL);
            //3、发送包清理事件
            schedulePackageCleaning(packageName, UserHandle.USER_ALL, true);
        }

6210        synchronized (mPackages) {
6211            if (deletedPs != null) {
6212                if ((flags&PackageManager.DONT_DELETE_DATA) == 0) {
6213                    if (outInfo != null) {
6214                        outInfo.removedUid = mSettings.removePackageLPw(packageName); // 4、移除mSetting中保存的数据
6215                    }
6216                    if (deletedPs != null) {
                                mHandler.post(new Runnable() {
                                    @Override
                                    public void run() {//5、发送事件杀死进程信息
                                        killApplication(deletedPs.name, deletedPs.appId,
                                                KILL_APP_REASON_GIDS_CHANGED);
                                    }
                                });
6221                        }
6222                    }
6223                } 
  if (writeSettings) {
    mSettings.writeLPr(); // 6、写入mSettings信息，更新Package.xml文件
  }
}
```

在removePackageDataLIF（）方法中执行了卸载删除的重要操作，具体如下：

1. 调用removePackageLI（）删除mPackages集合中保存的次apk的解析对象Package，在removePackageLI中会调用cleanPackageDataStructuresLILPw（），删除此PMS中保存的此应用程序中的四大组件信息

```java
  void cleanPackageDataStructuresLILPw(PackageParser.Package pkg, boolean chatty) {
        int N = pkg.providers.size();
         for (i=0; i<N; i++) {
            PackageParser.Provider p = pkg.providers.get(i);
            mProviders.removeProvider(p);
        }
        
        N = pkg.services.size();
        r = null;
        for (i=0; i<N; i++) {
            PackageParser.Service s = pkg.services.get(i);
            mServices.removeService(s);
        }
        }
        ......
}
```

1. 调用destroyAppDataLIF（）方法，使用Installer删除此应用程序/data/app/目录下的安装包信息
2. 调用schedulePackageCleaning（）发送START_CLEANING_PACKAGE事件执行外部使用数据删除
3. 删除mSetting中保存此程序的PackageSetting对象
4. 发送Handler事件调用killApplication（）杀死应用进程
5. 调用mSettings.writeLPr()将mSettings信息写入配置文件package.xml中

- destroyAppDataLIF（）

在destroyAppDataLIF（）中直接调用destroyAppDataLeafLIF（）方法，destroyAppDataLeafLIF中主要调用mInstaller中方法执行删除，删除应用数据之后调用mDexManager通知删除结束；

```java
private void destroyAppDataLeafLIF(PackageParser.Package pkg, int userId, int flags) {
        final PackageSetting ps;
        synchronized (mPackages) {
            ps = mSettings.mPackages.get(pkg.packageName);//获取PackageSetting对象
        }
        for (int realUserId : resolveUserIds(userId)) {
            final long ceDataInode = (ps != null) ? ps.getCeDataInode(realUserId) : 0;
            try { // 调用mInstaller执行文件数据删除
        mInstaller.destroyAppData(pkg.volumeUuid, pkg.packageName, realUserId, flags,
                        ceDataInode);
            } 
        mDexManager.notifyPackageDataDestroyed(pkg.packageName, userId);
        }
    }
```

- schedulePackageCleaning（）：执行mSetting中信息和程序数据文件的删除

```java
void schedulePackageCleaning(String packageName, int userId, boolean andCode) {
        final Message msg = mHandler.obtainMessage(START_CLEANING_PACKAGE,
                userId, andCode ? 1 : 0, packageName); // 创建Message对象
            msg.sendToTarget(); // 发送事件
    }

case START_CLEANING_PACKAGE: {
                    Process.setThreadPriority(Process.THREAD_PRIORITY_DEFAULT);
                    final String packageName = (String)msg.obj;
                    final int userId = msg.arg1;
                    final boolean andCode = msg.arg2 != 0;
                    synchronized (mPackages) {
                            mSettings.addPackageToCleanLPw(
                                    new PackageCleanItem(userId, packageName, andCode));
                                    }          Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
startCleaningPackages();
                } break;
```

在schedulePackageCleaning（）创建Message对象并发送事件，在Handler处理事件时首先调用mSetting将要删除的信息保存在Settings的mPackagesToBeCleaned集合中，然后调用startCleaningPackages（）启动本地服务DefaultContainerService服务，

```java
@Override
    protected void onHandleIntent(Intent intent) {
        if (PackageManager.ACTION_CLEAN_EXTERNAL_STORAGE.equals(intent.getAction())) {
            final IPackageManager pm = IPackageManager.Stub.asInterface(
                    ServiceManager.getService("package")); // 获取PMS对象
            PackageCleanItem item = null;
            try {
                while ((item = pm.nextPackageToClean(item)) != null) {
                    final UserEnvironment userEnv = new UserEnvironment(item.userId);
   //删除/Android/data/xxxx/下文件信息
eraseFiles(userEnv.buildExternalStorageAppDataDirs(item.packageName));
   //删除/Android/media/xxxx/下文件信息
eraseFiles(userEnv.buildExternalStorageAppMediaDirs(item.packageName));
                }
            } 
        }
    }
```

在本地服务的onHandleIntent（）中调用PMS的nextPackageToClean（）获取刚才加入的mSetting中的mPackagesToBeCleaned集合，并遍历集合一次从中取出要卸载删除的数据包，并获取文件路径执行eraseFiles（）删除文件；

```java
void eraseFiles(File path) {
        if (path.isDirectory()) {
            String[] files = path.list();
            if (files != null) {
                for (String file : files) {
                    eraseFiles(new File(path, file)); // 递归删除目录下文件
                }
            }
        }
        path.delete(); // 删除数据
    }
```

到此应用程序的app安装文件和使用中产生的数据文件都已被删除，且配置文件已更新完成，此时会回调deletePackageVersioned（）中的 observer.onPackageDeleted(packageName, returnCode, null)方法通知删除结果，这里回调的是PackageDeleteObserverAdapter类中

```java
@Override
        public void onPackageDeleted(String basePackageName, int returnCode, String msg) {
            if (PackageManager.DELETE_SUCCEEDED == returnCode && mNotification != null) {
                NotificationManager notificationManager = (NotificationManager)
                        mContext.getSystemService(Context.NOTIFICATION_SERVICE);
                notificationManager.notify(basePackageName,
                        SystemMessage.NOTE_PACKAGE_STATE,
                        mNotification);
            }
            final Intent fillIn = new Intent();
            fillIn.putExtra(PackageInstaller.EXTRA_PACKAGE_NAME, mPackageName);
            fillIn.putExtra(PackageInstaller.EXTRA_STATUS,
                    PackageManager.deleteStatusToPublicStatus(returnCode));
            fillIn.putExtra(PackageInstaller.EXTRA_STATUS_MESSAGE,
                    PackageManager.deleteStatusToString(returnCode, msg));
            fillIn.putExtra(PackageInstaller.EXTRA_LEGACY_STATUS, returnCode);
            try {
                mTarget.sendIntent(mContext, 0, fillIn, null, null);
            } catch (SendIntentException ignored) {
            }
        }
```

在onPackageDeleted（）中，首先调用NotificationManager中方法发送notify（）通知安装成功，然后创建Intent对象并回调mTarget.sendIntent（），这里的mTarget对象其实就是卸载最开始时调用uninstall（）方法传入的第二个参数，即IntentSender对象，用于接收删除的结果；

## 深入PMS源码（三）—— PMS中intent-filter的匹配架构

由前面**深入PMS源码（一）—— PMS的启动过程和执行流程和深入PMS源码（二）—— APK的安装和卸载源码分析**两篇文章知道，无论是Android系统启动后执行的PMS启动，还是使用PackageInstaller安装APK的过程，最终都会使用PackageParser扫描相应的apk文件，将扫描提取的信息保在Package对象中，扫描完成后会回调PMS中方法将扫描获取的四大组件信息转换保存在PMS属性中，主要使用ActivityIntentResolver、ServiceIntentResolver、ProviderIntentResolver保存信息，这里会有两个问题：

1. 系统是按照何种方式保存这些属性的对应关系？保存这些属性又是如何使用的？
2. 我们启动Activity只是创建了Intent对象，然后调用Context中的启动方法而已，那系统是如何查找到目标活动的信息的呢？

### PMS保存IntentFilter

下面带着这两个问题，从源码的角度探讨下PMS中的intent-filter的匹配架构，接着PMS的启动过程中分析，对四大组件处理代码如下，这里已对Activity的处理为例分析

```java
N = pkg.activities.size();
3503            r = null;
3504            for (i=0; i<N; i++) {
3505                PackageParser.Activity a = pkg.activities.get(i);
3506                a.info.processName = fixProcessName(pkg.applicationInfo.processName,
3507                        a.info.processName, pkg.applicationInfo.uid);
3508                mActivities.addActivity(a, "activity"); //调用ActivityIntentResolver.addActivity（）
}

// ActivityIntentResolver.addActivity()
public final void addActivity(PackageParser.Activity a, String type) {
    mActivities.put(a.getComponentName(), a); // 1、获取内部的Component对象，在Activity会自动创建Component对象
    final int NI = a.intents.size();
    for (int j=0; j<NI; j++) {
        PackageParser.ActivityIntentInfo intent = a.intents.get(j); // 获取Activity中设置的intent
        addFilter(intent); // 添加Intent过滤
    }
}
```

从解析得到的Package对象中获取每个创建的Activity对象，然后调用ActivityIntentResolver.addActivity（）方法将Activity对象保存在ActivityIntentResolver.mActivities集合中，在程序的最后几行代码中会遍历a.intents的集合，取出每个ActivityIntentInfo信息调用addFilter（）添加Intent过滤信息，这里的ActivityIntentInfo是在PackageParser解析标签中创建的对象并添加到a.intents的集合中，也就是说每个Activity标签下定义的标签都对应一个ActivityIntentInfo对象，且所有的对象都保存在a.intents的集合中，在介绍addFilter（）方法前先介绍下IntentResolver类；

- IntentResolver

1. 作用：IntentResolver主要用于保存程序中设置的和相应的ActivityRecord的对应关系，并在使用时提供查找机制
2. 实现原理：IntentResolver会按照中的每个特性（如action、type）等分别创建HashMap，并在解析时保存相应的对象，在查找时取出要匹配的Component对象的action、type，然后再各自的ArrayMap中获取其保存对象的集合，此时获取到的是多个集合其中包含重复信息，然后对所有的集合取交集并去重，最终得到完全匹配的对象；
3. IntentResolver属性如下：

```java
public abstract class IntentResolver<F extends IntentFilter, R extends Object> {
private final ArraySet<F> mFilters = new ArraySet<F>(); //保存所有的intent对应的匹配对象
private final ArrayMap<String, F[]> mTypeToFilter = new ArrayMap<String, F[]>(); //保存Type 匹配的Intent集合
private final ArrayMap<String, F[]> mBaseTypeToFilter = new ArrayMap<String, F[]>();
private final ArrayMap<String, F[]> mWildTypeToFilter = new ArrayMap<String, F[]>();// 保存所有带“*”通配符类型
private final ArrayMap<String, F[]> mSchemeToFilter = new ArrayMap<String, F[]>(); //已注册所有Uri方案
private final ArrayMap<String, F[]> mActionToFilter = new ArrayMap<String, F[]>();//保存所有指定Action 的Intent集合
private final ArrayMap<String, F[]> mTypedActionToFilter = new ArrayMap<String, F[]>(); // 匹配已注册且指定Mime Type类型Intent
}
```

上面的IntentResolver是一个范型，内部储存者IntentFilter的子类，而ActivityIntentInfo继承IntentInfo，IntentInfo又继承于IntentFilter类，所以此处直接保存的就是ActivityIntentInfo对象，在IntentFilter中保存着设置的多个属性

```java
private int mPriority; // 优先级
private int mOrder; // 层级
private final ArrayList<String> mActions;  // 设置的actions集合
private ArrayList<String> mCategories = null; // 设置categories集合
private ArrayList<String> mDataSchemes = null; // 数据集合
private ArrayList<PatternMatcher> mDataSchemeSpecificParts = null;
private ArrayList<AuthorityEntry> mDataAuthorities = null; // 匹配权限
private ArrayList<PatternMatcher> mDataPaths = null; // 匹配数据path
private ArrayList<String> mDataTypes = null; // 数据类型
```

- addFilter（）：添加匹配意图

添加PMS扫描获得的可匹配的IntentFilter对象，在PMS解析时将每个Activity中设置的标签中信息保存在一个IntentFilter对象中，在IntentFilter中保存着标签下所有的action、type、scheme、categories的集合，在addFilter（）时将这些集合中数据取出作为Key，将包含相同Key的Intent-Filter保存在数组中，然后以Key为键将这些集合存储在Intent-Resolve的ArrayMap中；

```java
public void addFilter(F f) {
    mFilters.add(f); // 在总的Filter中保存数据
    int numS = register_intent_filter(f, f.schemesIterator(),
            mSchemeToFilter, "      Scheme: “); // 判断数据uri，保存在mSchemeToFilter集合中
    int numT = register_mime_types(f, "      Type: “); //判断baseType 和 mWildType 类型
    if (numS == 0 && numT == 0) { 
        register_intent_filter(f, f.actionsIterator(),
                mActionToFilter, "      Action: “); // 判断匹配action对应的
    }
    if (numT != 0) {
        register_intent_filter(f, f.actionsIterator(),
                mTypedActionToFilter, "      TypedAction: “); // 添加mTypedActionToFilter集合
    }
}
```

在addFilter（）中首先将传入的intent-filter对象保存在mFilters中，也就是说mFilters中保存着所有的匹配对象，然后依次调用register_intent_filter（）、register_mime_types（）、register_intent_filter（）、register_intent_filter（）方法分别解析IntentFilter对象中的属性信息，并保存在相应的HashMap中，这里以register_intent_filter（）为例，register_intent_filter中传入的是f.actionsIterator()对象，即获取是actions集合的迭代器

```java
private final int register_intent_filter(F filter, Iterator<String> i, ArrayMap<String, F[]> dest, String prefix) {
    int num = 0;
    while (i.hasNext()) {   // 遍历Iterator
        String name = i.next();   // 取出数据集合DataSchemes下一个数据
        num++;
        addFilter(dest, name, filter);   // 添加到对应的Filter集合中，此处为mSchemeToFilter集合数据
    }
    return num;
}
```

在register_intent_filter中使用迭代器遍历每个action，然后取出每个action标签下设置的name属性，调用addFilter（）方法执行保存，在addFilter（）中传入要保存的相应的ArrayMap对象，如action对应的就是mActionToFilter

```java
private final void addFilter(ArrayMap<String, F[]> map, String name, F filter) {
    F[] array = map.get(name);    // 根据name的名称去除保存数据的数组
    if (array == null) {
        array = newArray(2);    // 创建数组
        map.put(name,  array);    // 保存数据
        array[0] = filter;    // 保存filter
    } else {
        final int N = array.length;
        int i = N;
        while (i > 0 && array[i-1] == null) { // 遍历寻找为null的位置
            i--;
        }
        if (i < N) {
            array[i] = filter;
        } else {
            F[] newa = newArray((N*3)/2); // 原数组长度不够，扩展原数组长度
            System.arraycopy(array, 0, newa, 0, N);
            newa[N] = filter;
            map.put(name, newa);
        }
    }
}
```

addFilter（）的执行逻辑很简单：

1. 首先根据传入的name从相应的集合中获取数组，如果不存在则创建新数组，此时直接保存IntentFilter对象；
2. 如果已经存在数组遍历循转数组中空位置，如果存在将IntentFilter对象保存在数组中；
3. 如果不存在则扩展数组长度，然后保存IntentFilter数据；
4. 最后将保存IntentFilter数据的数组添加到对应的ArrayMap中；

到此程序中注册的四大组件和相应的信息都会保存在IntentResolver类中，在使用时只需要根据设置的条件去查找匹配的对象即可；

### IntentFilter的查找匹配

查询匹配方式：IntentResolver查询时分别从请求的Intent中取出数据（如：action、mime、scheme），然后分别以每个数据变量为Key，取出每个相应的ArrayMao中保存的对应的IntentFilter数组，最后求去这些数组的交集并去重，达到精准匹配的目的，最后返回匹配的所有ResolveInfo集合；

一般在查找是否能匹配意图时，会调用packageManager.queryIntentActivities(intent,flag)查找匹配的IntentResolve，这里的packageManager对象实际调用ApplicationPackageManager对象，程序进入ApplicationPackageManager中，在ApplicationPackageManager.queryIntentActivities()中直接调用了queryIntentActivitiesAsUser（）

- ApplicationPackageManager

```java
@Override
@SuppressWarnings("unchecked")
public List<ResolveInfo> queryIntentActivitiesAsUser(Intent intent,
        int flags, int userId) {
    try {
    ParceledListSlice<ResolveInfo> parceledList = 
    mPM.queryIntentActivities(intent,
                        intent.resolveTypeIfNeeded(mContext.getContentResolver()),
                        flags, userId);2、
        return parceledList.getList();
    } 
}
```

在queryIntentActivitiesAsUser（）中调用PMS中方法查询匹配Intent，PMS.queryIntentActivities()中间接调用queryIntentActivitiesInternal（）方法

- queryIntentActivitiesInternal（）

```java
if (!sUserManager.exists(userId)) return Collections.emptyList(); // 判断userId
final String instantAppPkgName = getInstantAppPackageName(filterCallingUid); // 获取userId对应的进程包名
final String pkgName = intent.getPackage(); // 获取Intent中携带的包名
ComponentName comp = intent.getComponent(); // 获取Component对象
final List<ResolveInfo> list = new ArrayList<ResolveInfo>(1);
final ActivityInfo ai = getActivityInfo(comp, flags, userId); // 首先从PMs中保存的mActivites集合中获取ActivityInfo信息
result = filterIfNotSystemUser(mActivities.queryIntent( // 从mActivities中查询匹配的List<ResolveInfo>
        intent, resolvedType, flags, userId), userId);
```

在queryIntentActivitiesInternal（）方法中：

1. 首先判断当前用于的userId，如果不存在则直接返回空集合；
2. 从请求的Intent中获取包含的Component对象；
3. 如果Component不为空说明此时是显示启动，直接调用首先从PMs中保存的mActivites集合中获取ActivityInfo信息；
4. 对于Component为空的则调用mActivities.queryIntent（）匹配能处理的ActivityInfo对象；

- ActivityIntentResolve.queryIntent（）中直接调用父类IntentResolve.queryIntent（），方法执行到IntentResolve.queryIntent（）中

```java
public List<R> queryIntent(Intent intent, String resolvedType, boolean defaultOnly,
        int userId) {
String scheme = intent.getScheme(); // 获取scheme
ArrayList<R> finalList = new ArrayList<R>();
F[] firstTypeCut = null;   //保存匹配的结果
F[] secondTypeCut = null;
F[] thirdTypeCut = null;
F[] schemeCut = null;
if (scheme != null) {
    schemeCut = mSchemeToFilter.get(scheme); // 1、先根据scheme匹配,获取匹配的数组
}
   final String baseType = resolvedType.substring(0, slashpos);
    if (!baseType.equals("*")) {
        if (resolvedType.length() != slashpos+2
                || resolvedType.charAt(slashpos+1) != '*') {
            firstTypeCut = mTypeToFilter.get(resolvedType); // 根据type匹配
            secondTypeCut = mWildTypeToFilter.get(baseType);
        } else {
            firstTypeCut = mBaseTypeToFilter.get(baseType);
            secondTypeCut = mWildTypeToFilter.get(baseType);
        }
        thirdTypeCut = mWildTypeToFilter.get("*");
    } else if (intent.getAction() != null) {
        firstTypeCut = mTypedActionToFilter.get(intent.getAction());
    }
}
if (resolvedType == null && scheme == null && intent.getAction() != null) {
    firstTypeCut = mActionToFilter.get(intent.getAction()); // 2、根据action匹配
}
if (firstTypeCut != null) {
    buildResolveList(intent, categories, debug, defaultOnly, resolvedType,
            scheme, firstTypeCut, finalList, userId); // 不断进行匹配
}
if (secondTypeCut != null) {
    buildResolveList(intent, categories, debug, defaultOnly, resolvedType,
            scheme, secondTypeCut, finalList, userId);
}
if (thirdTypeCut != null) {
    buildResolveList(intent, categories, debug, defaultOnly, resolvedType,
            scheme, thirdTypeCut, finalList, userId);
}
if (schemeCut != null) {
    buildResolveList(intent, categories, debug, defaultOnly, resolvedType,
            scheme, schemeCut, finalList, userId);
}
sortResults(finalList); // 将数组中元素按照优先级排序
return finalList; // 返回最终匹配的结果集合
```

在queryIntent（）中首先从Intent中取出所有属性信息，如action、scheme等，然后创建四个数组对象，这四个数组主要保存根据Intent中设置的属性查找出对应的数组，之后从每个ArrayMap中匹配返回相应的数组，然后根据四个数数组是否存在数据从而多此执行buildResolveList（）方法，buildResolveList主要是获取四个数组中保存的IntentFilter对象转换为输出的对象，然后执行sortResults（）其中的重复对象此时即可返回最终匹配结果；

```java
private void buildResolveList(Intent intent, FastImmutableArraySet<String> categories,
        boolean debug, boolean defaultOnly, String resolvedType, String scheme,
        F[] src, List<R> dest, int userId) {
final int N = src != null ? src.length : 0;
F filter;
for (i=0; i<N && (filter=src[i]) != null; i++) {
match = filter.match(action, resolvedType, scheme, data, categories, TAG);
if (match >= 0) {
    if (!defaultOnly || filter.hasCategory(Intent.CATEGORY_DEFAULT)) {
        final R oneResult = newResult(filter, match, userId); // 将filter转换为 ResolveInfo 
        if (oneResult != null) {
            dest.add(oneResult); // 保存到dest集合中
        }
    } 
}
}
}
```

在上述的匹配过程之后即可获取到所有的可处理请求的Activity对象，然后将其中的信息设置在Intent之后再执行请求，此时就是像动态启动Activity一样直接启动目标活动；

```java
Intent intent = new Intent(intentToResolve);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
intent.setClassName(ris.get(0).activityInfo.packageName,
ris.get(0).activityInfo.name); 
return intent；
```

到此PMS中关于保存IntentFilter和静态启动过程中的匹配过程介绍完毕了，通过上面的学习相信对PMS如何处理apk文件和系统如何识别要启动的程序或活动有了更深的理解，本篇也是PMS系列的最后一篇，希望整个系列的分析对大家学习有所帮助；



# WMS

## WMS（一）：WMS的诞生

此前我用多篇文章介绍了WindowManager，这个系列我们来介绍WindowManager的管理者WMS，首先我们先来学习WMS是如何产生的。本文源码基于Android 8.0，与Android  7.1.2相比有一个比较直观的变化就是Java FrameWork采用了Lambda表达式。

### WMS概述

WMS是系统的其他服务，无论对于应用开发还是Framework开发都是重点的知识，它的职责有很多，主要有以下几点：

#### 窗口管理

WMS是窗口的管理者，它负责窗口的启动、添加和删除，另外窗口的大小和层级也是由WMS进行管理的。窗口管理的核心成员有DisplayContent、WindowToken和WindowState。

#### 窗口动画

窗口间进行切换时，使用窗口动画可以显得更炫一些，窗口动画由WMS的动画子系统来负责，动画子系统的管理者为WindowAnimator。

#### 输入系统的中转站

通过对窗口的触摸从而产生触摸事件，InputManagerService（IMS）会对触摸事件进行处理，它会寻找一个最合适的窗口来处理触摸反馈信息，WMS是窗口的管理者，因此，WMS“理所应当”的成为了输入系统的中转站。

#### Surface管理

窗口并不具备有绘制的功能，因此每个窗口都需要有一块Surface来供自己绘制。为每个窗口分配Surface是由WMS来完成的。

WMS的职责可以简单总结为下图。



![img](https://s2.ax1x.com/2019/05/28/VexIGd.png)



### WMS的诞生

WMS的知识点非常多，在了解这些知识点前，我们十分有必要知道WMS是如何产生的。WMS是在SyetemServer进程中启动的，不了解SyetemServer进程的可以查看在[Android系统启动流程（三）解析SyetemServer进程启动过程](https://link.juejin.cn?target=http%3A%2F%2Fliuwangshu.cn%2Fframework%2Fbooting%2F3-syetemserver.html)这篇文章。
先来查看SyetemServer的main方法：
**frameworks/base/services/java/com/android/server/SystemServer.java**

```java
public static void main(String[] args) {
       new SystemServer().run();
}
```

main方法中只调用了SystemServer的run方法，如下所示。
**frameworks/base/services/java/com/android/server/SystemServer.java**

```java
private void run() {
       try {
          System.loadLibrary("android_servers");//1
          ...
          mSystemServiceManager = new SystemServiceManager(mSystemContext);//2
          mSystemServiceManager.setRuntimeRestarted(mRuntimeRestart);
          LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);
          // Prepare the thread pool for init tasks that can be parallelized
          SystemServerInitThreadPool.get();
      } finally {
          traceEnd();  // InitBeforeStartServices
      }
      try {
          traceBeginAndSlog("StartServices");
          startBootstrapServices();//3
          startCoreServices();//4
          startOtherServices();//5
          SystemServerInitThreadPool.shutdown();
      } catch (Throwable ex) {
          Slog.e("System", "******************************************");
          Slog.e("System", "************ Failure starting system services", ex);
          throw ex;
      } finally {
          traceEnd();
      }
  ...
  }
```

run方法代码很多，这里截取了关键的部分，在注释1处加载了libandroid_servers.so。在注释2处创建SystemServiceManager，它会对系统的服务进行创建、启动和生命周期管理。接下来的代码会启动系统的各种服务，在注释3中的startBootstrapServices方法中用SystemServiceManager启动了ActivityManagerService、PowerManagerService、PackageManagerService等服务。在注释4处的方法中则启动了BatteryService、UsageStatsService和WebViewUpdateService。注释5处的startOtherServices方法中则启动了CameraService、AlarmManagerService、VrManagerService等服务，这些服务的父类为SystemService。从注释3、4、5的方法名称可以看出，官方把大概80多个系统服务分为了三种类型，分别是引导服务、核心服务和其他服务，其中其他服务为一些非紧要和一些不需要立即启动的服务，WMS就是其他服务的一种。
我们来查看startOtherServices方法是如何启动WMS的：

**frameworks/base/services/java/com/android/server/SystemServer.java**

```Java
 private void startOtherServices() {
 ...
            traceBeginAndSlog("InitWatchdog");
            final Watchdog watchdog = Watchdog.getInstance();//1
            watchdog.init(context, mActivityManagerService);//2
            traceEnd();
            traceBeginAndSlog("StartInputManagerService");
            inputManager = new InputManagerService(context);//3
            traceEnd();
            traceBeginAndSlog("StartWindowManagerService");
            ConcurrentUtils.waitForFutureNoInterrupt(mSensorServiceStart, START_SENSOR_SERVICE);
            mSensorServiceStart = null;
            wm = WindowManagerService.main(context, inputManager,
                    mFactoryTestMode != FactoryTest.FACTORY_TEST_LOW_LEVEL,
                    !mFirstBoot, mOnlyCore, new PhoneWindowManager());//4
            ServiceManager.addService(Context.WINDOW_SERVICE, wm);//5
            ServiceManager.addService(Context.INPUT_SERVICE, inputManager);//6
            traceEnd();   
           ... 
           try {
            wm.displayReady();//7
               } catch (Throwable e) {
            reportWtf("making display ready", e);
              }
           ...
           try {
            wm.systemReady();//8
               } catch (Throwable e) {
            reportWtf("making Window Manager Service ready", e);
              }
            ...      
}
```

startOtherServices方法用于启动其他服务，其他服务大概有70多个，上面的代码只列出了WMS以及和它相关的IMS的启动逻辑，剩余的其他服务的启动逻辑也都大同小异。
在注释1、2处分别得到Watchdog实例并对它进行初始化，Watchdog用来监控系统的一些关键服务的运行状况，后文会再次提到它。在注释3处创建了IMS，并赋值给IMS类型的inputManager对象。注释4处执行了WMS的main方法，其内部会创建WMS，需要注意的是main方法其中一个传入的参数就是注释1处创建的IMS，WMS是输入事件的中转站，其内部包含了IMS引用并不意外。结合上文，我们可以得知WMS的main方法是运行在SystemServer的run方法中，换句话说就是运行在"system_server"线程”中，后面会再次提到"system_server"线程。
注释5和注释6处分别将WMS和IMS注册到ServiceManager中，这样如果某个客户端想要使用WMS，就需要先去ServiceManager中查询信息，然后根据信息与WMS所在的进程建立通信通路，客户端就可以使用WMS了。注释7处用来初始化显示信息，注释8处则用来通知WMS，系统的初始化工作已经完成，其内部调用了WindowManagerPolicy的systemReady方法。
我们来查看注释4处WMS的main方法，如下所示。
**frameworks/base/services/core/java/com/android/server/wm/WindowManagerService .java**

```java
public static WindowManagerService main(final Context context, final InputManagerService im,
           final boolean haveInputMethods, final boolean showBootMsgs, final boolean onlyCore,
           WindowManagerPolicy policy) {
       DisplayThread.getHandler().runWithScissors(() ->//1
               sInstance = new WindowManagerService(context, im, haveInputMethods, showBootMsgs,
                       onlyCore, policy), 0);
       return sInstance;
   }
```

在注释1处调用了DisplayThread的getHandler方法，用来得到DisplayThread的Handler实例。DisplayThread是一个单例的前台线程，这个线程用来处理需要低延时显示的相关操作，并只能由WindowManager、DisplayManager和InputManager实时执行快速操作。注释1处的runWithScissors方法中使用了Java8中的Lambda表达式，它等价于如下代码：

```java
DisplayThread.getHandler().runWithScissors(new Runnable() {
        @Override
        public void run() {
         sInstance = new WindowManagerService(context, im, haveInputMethods, showBootMsgs,
                    onlyCore, policy);//2
        }
    }, 0);
```

在注释2处创建了WMS的实例，这个过程运行在Runnable的run方法中，而Runnable则传入到了DisplayThread对应Handler的runWithScissors方法中，说明WMS的创建是运行在“android.display”线程中。需要注意的是，runWithScissors方法的第二个参数传入的是0，后面会提到。来查看Handler的runWithScissors方法里做了什么：

**frameworks/base/core/java/android/os/Handler.java**

```java
public final boolean runWithScissors(final Runnable r, long timeout) {
       if (r == null) {
           throw new IllegalArgumentException("runnable must not be null");
       }
       if (timeout < 0) {
           throw new IllegalArgumentException("timeout must be non-negative");
       }
       if (Looper.myLooper() == mLooper) {//1
           r.run();
           return true;
       }
       BlockingRunnable br = new BlockingRunnable(r);
       return br.postAndWait(this, timeout);
   }
```

开头对传入的Runnable和timeout进行了判断，如果Runnable为null或者timeout小于0则抛出异常。注释1处根据每个线程只有一个Looper的原理来判断当前的线程（"system_server"线程）是否是Handler所指向的线程（"android.display"线程），如果是则直接执行Runnable的run方法，如果不是则调用BlockingRunnable的postAndWait方法，并将当前线程的Runnable作为参数传进去 ，BlockingRunnable是Handler的内部类，代码如下所示。
**frameworks/base/core/java/android/os/Handler.java**

```java
private static final class BlockingRunnable implements Runnable {
        private final Runnable mTask;
        private boolean mDone;
        public BlockingRunnable(Runnable task) {
            mTask = task;
        }
        @Override
        public void run() {
            try {
                mTask.run();//1
            } finally {
                synchronized (this) {
                    mDone = true;
                    notifyAll();
                }
            }
        }
        public boolean postAndWait(Handler handler, long timeout) {
            if (!handler.post(this)) {//2
                return false;
            }
            synchronized (this) {
                if (timeout > 0) {
                    final long expirationTime = SystemClock.uptimeMillis() + timeout;
                    while (!mDone) {
                        long delay = expirationTime - SystemClock.uptimeMillis();
                        if (delay <= 0) {
                            return false; // timeout
                        }
                        try {
                            wait(delay);
                        } catch (InterruptedException ex) {
                        }
                    }
                } else {
                    while (!mDone) {
                        try {
                            wait();//3
                        } catch (InterruptedException ex) {
                        }
                    }
                }
            }
            return true;
        }
    }
```

注释2处将当前的BlockingRunnable添加到Handler的任务队列中。前面runWithScissors方法的第二个参数为0，因此timeout等于0，这样如果mDone为false的话会一直调用注释3处的wait方法使得当前线程（"system_server"线程）进入等待状态，那么等待的是哪个线程呢？我们往上看，注释1处，执行了传入的Runnable的run方法（运行在"android.display"线程），执行完毕后在finally代码块中将mDone设置为true，并调用notifyAll方法唤醒处于等待状态的线程，这样就不会继续调用注释3处的wait方法。因此得出结论，"system_server"线程线程等待的就是"android.display"线程，一直到"android.display"线程执行完毕再执行"system_server"线程，这是因为"android.display"线程内部执行了WMS的创建，显然WMS的创建优先级更高些。
WMS的创建就讲到这，最后我们来查看WMS的构造方法：

**frameworks/base/services/core/java/com/android/server/wm/WindowManagerService .java**

```java
private WindowManagerService(Context context, InputManagerService inputManager,
         boolean haveInputMethods, boolean showBootMsgs, boolean onlyCore, WindowManagerPolicy policy) {
    ...
    mInputManager = inputManager;//1
    ...
     mDisplayManager = (DisplayManager)context.getSystemService(Context.DISPLAY_SERVICE);
     mDisplays = mDisplayManager.getDisplays();//2
     for (Display display : mDisplays) {
         createDisplayContentLocked(display);//3
     }
    ...
      mActivityManager = ActivityManager.getService();//4
    ...
     mAnimator = new WindowAnimator(this);//5
     mAllowTheaterModeWakeFromLayout = context.getResources().getBoolean(
             com.android.internal.R.bool.config_allowTheaterModeWakeFromWindowLayout);
     LocalServices.addService(WindowManagerInternal.class, new LocalService());
     initPolicy();//6
     // Add ourself to the Watchdog monitors.
     Watchdog.getInstance().addMonitor(this);//7
  ...
 }
```

注释1处用来保存传进来的IMS，这样WMS就持有了IMS的引用。注释2处通过DisplayManager的getDisplays方法得到Display数组（每个显示设备都有一个Display实例），接着遍历Display数组，在注释3处的createDisplayContentLocked方法会将Display封装成DisplayContent，DisplayContent用来描述一快屏幕。
注释4处得到AMS实例，并赋值给mActivityManager ，这样WMS就持有了AMS的引用。注释5处创建了WindowAnimator，它用于管理所有的窗口动画。注释6处初始化了窗口管理策略的接口类WindowManagerPolicy（WMP），它用来定义一个窗口策略所要遵循的通用规范。注释7处将自身也就是WMS通过addMonitor方法添加到Watchdog中，Watchdog用来监控系统的一些关键服务的运行状况（比如传入的WMS的运行状况），这些被监控的服务都会实现Watchdog.Monitor接口。Watchdog每分钟都会对被监控的系统服务进行检查，如果被监控的系统服务出现了死锁，则会杀死Watchdog所在的进程，也就是SystemServer进程。

查看注释6处的initPolicy方法，如下所示。
**frameworks/base/services/core/java/com/android/server/wm/WindowManagerService.java**

```java
private void initPolicy() {
     UiThread.getHandler().runWithScissors(new Runnable() {
         @Override
         public void run() {
             WindowManagerPolicyThread.set(Thread.currentThread(), Looper.myLooper());
             mPolicy.init(mContext, WindowManagerService.this, WindowManagerService.this);//1
         }
     }, 0);
 }
```

initPolicy方法和此前讲的WMS的main方法的实现类似，注释1处执行了WMP的init方法，WMP是一个接口，init方法的具体实现在PhoneWindowManager（PWM）中。PWM的init方法运行在"android.ui"线程中，它的优先级要高于initPolicy方法所在的"android.display"线程，因此"android.display"线程要等PWM的init方法执行完毕后，处于等待状态的"android.display"线程才会被唤醒从而继续执行下面的代码。

在本文中共提到了3个线程，分别是"system_server"、"android.display"和"android.ui"，为了便于理解，下面给出这三个线程之间的关系。



![img](https://s2.ax1x.com/2019/05/28/VexoRA.png)



"system_server"线程中会调用WMS的main方法，main方法中会创建WMS，创建WMS的过程运行在"android.display"线程中，它的优先级更高一些，因此要等创建WMS完毕后才会唤醒处于等待状态的"system_server"线程。
WMS初始化时会执行initPolicy方法，initPolicy方法会调用PWM的init方法，这个init方法运行在"android.ui"线程，并且优先级更高，因此要先执行完PWM的init方法后，才会唤醒处于等待状态的"android.display"线程。
PWM的init方法执行完毕后会接着执行运行在"system_server"线程的代码，比如本文前部分提到WMS的systemReady方法。

## WMS（二）：WMS的重要成员和Window的添加过程

在本系列的上一篇文章中，我们学习了WMS的诞生，WMS被创建后，它的重要的成员有哪些？Window添加过程的WMS部分做了什么呢？这篇文章会给你解答。

### WMS的重要成员

所谓WMS的重要成员是指WMS中的重要的成员变量，如下所示。
**frameworks/base/services/core/java/com/android/server/wm/WindowManagerService.java**

```java
final WindowManagerPolicy mPolicy;
final IActivityManager mActivityManager;
final ActivityManagerInternal mAmInternal;
final AppOpsManager mAppOps;
final DisplaySettings mDisplaySettings;
...
final ArraySet<Session> mSessions = new ArraySet<>();
final WindowHashMap mWindowMap = new WindowHashMap();
final ArrayList<AppWindowToken> mFinishedStarting = new ArrayList<>();
final ArrayList<AppWindowToken> mFinishedEarlyAnim = new ArrayList<>();
final ArrayList<AppWindowToken> mWindowReplacementTimeouts = new ArrayList<>();
final ArrayList<WindowState> mResizingWindows = new ArrayList<>();
final ArrayList<WindowState> mPendingRemove = new ArrayList<>();
WindowState[] mPendingRemoveTmp = new WindowState[20];
final ArrayList<WindowState> mDestroySurface = new ArrayList<>();
final ArrayList<WindowState> mDestroyPreservedSurface = new ArrayList<>();
...
final H mH = new H();
...
final WindowAnimator mAnimator;
...
 final InputManagerService mInputManager
```

这里列出了WMS的部分成员变量，下面分别对它们进行简单的介绍。

 **mPolicy：WindowManagerPolicy**
WindowManagerPolicy（WMP）类型的变量。WindowManagerPolicy是窗口管理策略的接口类，用来定义一个窗口策略所要遵循的通用规范，并提供了WindowManager所有的特定的UI行为。它的具体实现类为PhoneWindowManager，这个实现类在WMS创建时被创建。WMP允许定制窗口层级和特殊窗口类型以及关键的调度和布局。

 **mSessions：ArraySet**
ArraySet类型的变量，元素类型为Session。在[Android解析WindowManager（三）Window的添加过程](https://link.juejin.cn?target=http%3A%2F%2Fliuwangshu.cn%2Fframework%2Fwm%2F3-add-window.html)这篇文章中我提到过Session，它主要用于进程间通信，其他的应用程序进程想要和WMS进程进行通信就需要经过Session，并且每个应用程序进程都会对应一个Session，WMS保存这些Session用来记录所有向WMS提出窗口管理服务的客户端。
**mWindowMap：WindowHashMap**
WindowHashMap类型的变量，WindowHashMap继承了HashMap，它限制了HashMap的key值的类型为IBinder，value值的类型为WindowState。WindowState用于保存窗口的信息，在WMS中它用来描述一个窗口。综上得出结论，mWindowMap就是用来保存WMS中各种窗口的集合。

 **mFinishedStarting：ArrayList**
ArrayList类型的变量，元素类型为AppWindowToken，它是WindowToken的子类。要想理解mFinishedStarting的含义，需要先了解WindowToken是什么。WindowToken主要有两个作用：

- 可以理解为窗口令牌，当应用程序想要向WMS申请新创建一个窗口，则需要向WMS出示有效的WindowToken。AppWindowToken作为WindowToken的子类，主要用来描述应用程序的WindowToken结构，
  应用程序中每个Activity都对应一个AppWindowToken。
- WindowToken会将相同组件（比如Acitivity）的窗口（WindowState）集合在一起，方便管理。

mFinishedStarting就是用于存储已经完成启动的应用程序窗口（比如Acitivity）的AppWindowToken的列表。
除了mFinishedStarting，还有类似的mFinishedEarlyAnim和mWindowReplacementTimeouts，其中mFinishedEarlyAnim存储了已经完成窗口绘制并且不需要展示任何已保存surface的应用程序窗口的AppWindowToken。mWindowReplacementTimeout存储了等待更换的应用程序窗口的AppWindowToken，如果更换不及时，旧窗口就需要被处理。

 **mResizingWindows：ArrayList**
ArrayList类型的变量，元素类型为WindowState。
mResizingWindows是用来存储正在调整大小的窗口的列表。与mResizingWindows类似的还有mPendingRemove、mDestroySurface和mDestroyPreservedSurface等等。其中mPendingRemove是在内存耗尽时设置的，里面存有需要强制删除的窗口。mDestroySurface里面存有需要被Destroy的Surface。mDestroyPreservedSurface里面存有窗口需要保存的等待销毁的Surface，为什么窗口要保存这些Surface？这是因为当窗口经历Surface变化时，窗口需要一直保持旧Surface，直到新Surface的第一帧绘制完成。

 **mAnimator：WindowAnimator**
WindowAnimator类型的变量，用于管理窗口的动画以及特效动画。

 **mH：H**
H类型的变量，系统的Handler类，用于将任务加入到主线程的消息队列中，这样代码逻辑就会在主线程中执行。

 **mInputManager：InputManagerService**
InputManagerService类型的变量，输入系统的管理者。InputManagerService（IMS）会对触摸事件进行处理，它会寻找一个最合适的窗口来处理触摸反馈信息，WMS是窗口的管理者，因此，WMS“理所应当”的成为了输入系统的中转站，WMS包含了IMS的引用不足为怪。

### Window的添加过程（WMS部分）

我们知道Window的操作分为两大部分，一部分是WindowManager处理部分，另一部分是WMS处理部分，如下所示。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2017/10/26/d4cbb185457f36fd6fe1bd58e7470868~tplv-t2oaga2asx-watermark.awebp)


在[Android解析WindowManager（三）Window的添加过程](https://link.juejin.cn?target=http%3A%2F%2Fliuwangshu.cn%2Fframework%2Fwm%2F3-add-window.html)这篇文章中，我讲解了Window的添加过程的WindowManager处理部分，这一篇文章我们接着来学习Window的添加过程的WMS部分。
无论是系统窗口还是Activity，它们的Window的添加过程都会调用WMS的addWindow方法，由于这个方法代码逻辑比较多，这里分为3个部分来阅读。
**frameworks/base/services/core/java/com/android/server/wm/WindowManagerService.java**



#### addWindow方法part1

```java
 public int addWindow(Session session, IWindow client, int seq,
            WindowManager.LayoutParams attrs, int viewVisibility, int displayId,
            Rect outContentInsets, Rect outStableInsets, Rect outOutsets,
            InputChannel outInputChannel) {

        int[] appOp = new int[1];
        int res = mPolicy.checkAddPermission(attrs, appOp);//1
        if (res != WindowManagerGlobal.ADD_OKAY) {
            return res;
        }
        ...
        synchronized(mWindowMap) {
            if (!mDisplayReady) {
                throw new IllegalStateException("Display has not been initialialized");
            }
            final DisplayContent displayContent = mRoot.getDisplayContentOrCreate(displayId);//2
            if (displayContent == null) {
                Slog.w(TAG_WM, "Attempted to add window to a display that does not exist: "
                        + displayId + ".  Aborting.");
                return WindowManagerGlobal.ADD_INVALID_DISPLAY;
            }
            ...
            if (type >= FIRST_SUB_WINDOW && type <= LAST_SUB_WINDOW) {//3
                parentWindow = windowForClientLocked(null, attrs.token, false);//4
                if (parentWindow == null) {
                    Slog.w(TAG_WM, "Attempted to add window with token that is not a window: "
                          + attrs.token + ".  Aborting.");
                    return WindowManagerGlobal.ADD_BAD_SUBWINDOW_TOKEN;
                }
                if (parentWindow.mAttrs.type >= FIRST_SUB_WINDOW
                        && parentWindow.mAttrs.type <= LAST_SUB_WINDOW) {
                    Slog.w(TAG_WM, "Attempted to add window with token that is a sub-window: "
                            + attrs.token + ".  Aborting.");
                    return WindowManagerGlobal.ADD_BAD_SUBWINDOW_TOKEN;
                }
            }
           ...
}
...
}
```

WMS的addWindow返回的是addWindow的各种状态，比如添加Window成功，无效的display等等，这些状态被定义在WindowManagerGlobal中。
注释1处根据Window的属性，调用WMP的checkAddPermission方法来检查权限，具体的实现在PhoneWindowManager的checkAddPermission方法中，如果没有权限则不会执行后续的代码逻辑。注释2处通过displayId来获得窗口要添加到哪个DisplayContent上，如果没有找到DisplayContent，则返回WindowManagerGlobal.ADD_INVALID_DISPLAY这一状态，其中DisplayContent用来描述一块屏幕。注释3处，type代表一个窗口的类型，它的数值介于FIRST_SUB_WINDOW和LAST_SUB_WINDOW之间（1000~1999），这个数值定义在WindowManager中，说明这个窗口是一个子窗口，不了解窗口类型取值范围的请阅读[Android解析WindowManager（二）Window的属性](https://link.juejin.cn?target=http%3A%2F%2Fliuwangshu.cn%2Fframework%2Fwm%2F2-window-property.html)这篇文章。注释4处，attrs.token是IBinder类型的对象，windowForClientLocked方法内部会根据attrs.token作为key值从mWindowMap中得到该子窗口的父窗口。接着对父窗口进行判断，如果父窗口为null或者type的取值范围不正确则会返回错误的状态。

#### addWindow方法part2

```java
...
         AppWindowToken atoken = null;
         final boolean hasParent = parentWindow != null;
         WindowToken token = displayContent.getWindowToken(
                 hasParent ? parentWindow.mAttrs.token : attrs.token);//1
         final int rootType = hasParent ? parentWindow.mAttrs.type : type;//2
         boolean addToastWindowRequiresToken = false;

         if (token == null) {
             if (rootType >= FIRST_APPLICATION_WINDOW && rootType <= LAST_APPLICATION_WINDOW) {
                 Slog.w(TAG_WM, "Attempted to add application window with unknown token "
                       + attrs.token + ".  Aborting.");
                 return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
             }
             if (rootType == TYPE_INPUT_METHOD) {
                 Slog.w(TAG_WM, "Attempted to add input method window with unknown token "
                       + attrs.token + ".  Aborting.");
                 return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
             }
             if (rootType == TYPE_VOICE_INTERACTION) {
                 Slog.w(TAG_WM, "Attempted to add voice interaction window with unknown token "
                       + attrs.token + ".  Aborting.");
                 return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
             }
             if (rootType == TYPE_WALLPAPER) {
                 Slog.w(TAG_WM, "Attempted to add wallpaper window with unknown token "
                       + attrs.token + ".  Aborting.");
                 return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
             }
             ...
             if (type == TYPE_TOAST) {
                 // Apps targeting SDK above N MR1 cannot arbitrary add toast windows.
                 if (doesAddToastWindowRequireToken(attrs.packageName, callingUid,
                         parentWindow)) {
                     Slog.w(TAG_WM, "Attempted to add a toast window with unknown token "
                             + attrs.token + ".  Aborting.");
                     return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
                 }
             }
             final IBinder binder = attrs.token != null ? attrs.token : client.asBinder();
             token = new WindowToken(this, binder, type, false, displayContent,
                     session.mCanAddInternalSystemWindow);//3
         } else if (rootType >= FIRST_APPLICATION_WINDOW && rootType <= LAST_APPLICATION_WINDOW) {//4
             atoken = token.asAppWindowToken();//5
             if (atoken == null) {
                 Slog.w(TAG_WM, "Attempted to add window with non-application token "
                       + token + ".  Aborting.");
                 return WindowManagerGlobal.ADD_NOT_APP_TOKEN;
             } else if (atoken.removed) {
                 Slog.w(TAG_WM, "Attempted to add window with exiting application token "
                       + token + ".  Aborting.");
                 return WindowManagerGlobal.ADD_APP_EXITING;
             }
         } else if (rootType == TYPE_INPUT_METHOD) {
             if (token.windowType != TYPE_INPUT_METHOD) {
                 Slog.w(TAG_WM, "Attempted to add input method window with bad token "
                         + attrs.token + ".  Aborting.");
                   return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
             }
         }
   ...      
```

注释1处通过displayContent的getWindowToken方法来得到WindowToken。注释2处，如果有父窗口就将父窗口的type值赋值给rootType，如果没有将当前窗口的type值赋值给rootType。接下来如果WindowToken为null，则根据rootType或者type的值进行区分判断，如果rootType值等于TYPE_INPUT_METHOD、TYPE_WALLPAPER等值时，则返回状态值WindowManagerGlobal.ADD_BAD_APP_TOKEN，说明rootType值等于TYPE_INPUT_METHOD、TYPE_WALLPAPER等值时是不允许WindowToken为null的。通过多次的条件判断筛选，最后会在注释3处隐式创建WindowToken，这说明当我们添加窗口时是可以不向WMS提供WindowToken的，前提是rootType和type的值不为前面条件判断筛选的值。WindowToken隐式和显式的创建肯定是要加以区分的，注释3处的第4个参数为false就代表这个WindowToken是隐式创建的。接下来的代码逻辑就是WindowToken不为null的情况，根据rootType和type的值进行判断，比如在注释4处判断如果窗口为应用程序窗口，在注释5处会将WindowToken转换为专门针对应用程序窗口的AppWindowToken，然后根据AppWindowToken的值进行后续的判断。

#### addWindow方法part3

```java
 ...
final WindowState win = new WindowState(this, session, client, token, parentWindow,
                  appOp[0], seq, attrs, viewVisibility, session.mUid,
                  session.mCanAddInternalSystemWindow);//1
          if (win.mDeathRecipient == null) {//2
              // Client has apparently died, so there is no reason to
              // continue.
              Slog.w(TAG_WM, "Adding window client " + client.asBinder()
                      + " that is dead, aborting.");
              return WindowManagerGlobal.ADD_APP_EXITING;
          }

          if (win.getDisplayContent() == null) {//3
              Slog.w(TAG_WM, "Adding window to Display that has been removed.");
              return WindowManagerGlobal.ADD_INVALID_DISPLAY;
          }

          mPolicy.adjustWindowParamsLw(win.mAttrs);//4
          win.setShowToOwnerOnlyLocked(mPolicy.checkShowToOwnerOnly(attrs));
          res = mPolicy.prepareAddWindowLw(win, attrs);//5
          ...
          win.attach();
          mWindowMap.put(client.asBinder(), win);//6
          if (win.mAppOp != AppOpsManager.OP_NONE) {
              int startOpResult = mAppOps.startOpNoThrow(win.mAppOp, win.getOwningUid(),
                      win.getOwningPackage());
              if ((startOpResult != AppOpsManager.MODE_ALLOWED) &&
                      (startOpResult != AppOpsManager.MODE_DEFAULT)) {
                  win.setAppOpVisibilityLw(false);
              }
          }

          final AppWindowToken aToken = token.asAppWindowToken();
          if (type == TYPE_APPLICATION_STARTING && aToken != null) {
              aToken.startingWindow = win;
              if (DEBUG_STARTING_WINDOW) Slog.v (TAG_WM, "addWindow: " + aToken
                      + " startingWindow=" + win);
          }

          boolean imMayMove = true;
          win.mToken.addWindow(win);//7
           if (type == TYPE_INPUT_METHOD) {
              win.mGivenInsetsPending = true;
              setInputMethodWindowLocked(win);
              imMayMove = false;
          } else if (type == TYPE_INPUT_METHOD_DIALOG) {
              displayContent.computeImeTarget(true /* updateImeTarget */);
              imMayMove = false;
          } else {
              if (type == TYPE_WALLPAPER) {
                  displayContent.mWallpaperController.clearLastWallpaperTimeoutTime();
                  displayContent.pendingLayoutChanges |= FINISH_LAYOUT_REDO_WALLPAPER;
              } else if ((attrs.flags&FLAG_SHOW_WALLPAPER) != 0) {
                  displayContent.pendingLayoutChanges |= FINISH_LAYOUT_REDO_WALLPAPER;
              } else if (displayContent.mWallpaperController.isBelowWallpaperTarget(win)) {
                  displayContent.pendingLayoutChanges |= FINISH_LAYOUT_REDO_WALLPAPER;
              }
          }
       ...
```

在注释1处创建了WindowState，它存有窗口的所有的状态信息，在WMS中它代表一个窗口。从WindowState传入的参数，可以发现WindowState中包含了WMS、Session、WindowToken、父类的WindowState、LayoutParams等信息。紧接着在注释2和3处分别判断请求添加窗口的客户端是否已经死亡、窗口的DisplayContent是否为null，如果是则不会再执行下面的代码逻辑。注释4处调用了WMP的adjustWindowParamsLw方法，该方法的实现在PhoneWindowManager中，会根据窗口的type对窗口的LayoutParams的一些成员变量进行修改。注释5处调用WMP的prepareAddWindowLw方法，用于准备将窗口添加到系统中。
注释6处将WindowState添加到mWindowMap中。注释7处将WindowState添加到该WindowState对应的WindowToken中(实际是保存在WindowToken的父类WindowContainer中)，这样WindowToken就包含了相同组件的WindowState。

#### addWindow方法总结

addWindow方法分了3个部分来进行讲解，主要就是做了下面4件事：

1. 对所要添加的窗口进行检查，如果窗口不满足一些条件，就不会再执行下面的代码逻辑。
2. WindowToken相关的处理，比如有的窗口类型需要提供WindowToken，没有提供的话就不会执行下面的代码逻辑，有的窗口类型则需要由WMS隐式创建WindowToken。
3. WindowState的创建和相关处理，将WindowToken和WindowState相关联。
4. 创建和配置DisplayContent，完成窗口添加到系统前的准备工作。

### 结语

在本篇文章中我们首先学习了WMS的重要成员，了解这些成员有利于对WMS的进一步分析。接下来我们又学习了Window的添加过程的WMS部分，将addWindow方法分为了3个部分来进行讲解，从addWindow方法我们得知WMS有3个重要的类分别是WindowToken、WindowState和DisplayContent，关于它们会在本系列后续的文章中进行介绍。

## WMS（三）：Window的删除过程

在本系列文章中，我提到过：Window的操作分为两大部分，一部分是WindowManager处理部分，另一部分是WMS处理部分，Window的删除过程也不例外，本篇文章会介绍Window的删除过程，包括了两大处理部分的内容。

### Window的删除过程

和[Android解析WindowManagerService（二）WMS的重要成员和Window的添加过程](https://link.juejin.cn?target=http%3A%2F%2Fliuwangshu.cn%2Fframework%2Fwms%2F2-wms-member.html)这篇文章中Window的创建和更新过程类似，要删除Window需要先调用WindowManagerImpl的removeView方法，removeView方法中又会调用WindowManagerGlobal的removeView方法，我们就从这里开始讲起。为了表述的更易于理解，本文将要删除的Window（View）简称为V。WindowManagerGlobal的removeView方法如下所示。

**frameworks/base/core/java/android/view/WindowManagerGlobal.java**

```java
public void removeView(View view, boolean immediate) {
      if (view == null) {
          throw new IllegalArgumentException("view must not be null");
      }
      synchronized (mLock) {
          int index = findViewLocked(view, true);//1
          View curView = mRoots.get(index).getView();
          removeViewLocked(index, immediate);//2
          if (curView == view) {
              return;
          }
          throw new IllegalStateException("Calling with view " + view
                  + " but the ViewAncestor is attached to " + curView);
      }
  }
```

注释1处找到要V在View列表中的索引，在注释2处调用了removeViewLocked方法并将这个索引传进去，如下所示。 **frameworks/base/core/java/android/view/WindowManagerGlobal.java**

```java
private void removeViewLocked(int index, boolean immediate) {
       ViewRootImpl root = mRoots.get(index);//1
       View view = root.getView();
       if (view != null) {
           InputMethodManager imm = InputMethodManager.getInstance();//2
           if (imm != null) {
               imm.windowDismissed(mViews.get(index).getWindowToken());//3
           }
       }
       boolean deferred = root.die(immediate);//4
       if (view != null) {
           view.assignParent(null);
           if (deferred) {
               mDyingViews.add(view);
           }
       }
   }
```

注释1处根据传入的索引在ViewRootImpl列表中获得V的ViewRootImpl。注释2处得到InputMethodManager实例，如果InputMethodManager实例不为null则在注释3处调用InputMethodManager的windowDismissed方法来结束V的输入法相关的逻辑。注释4处调用ViewRootImpl 的die方法，如下所示。

**frameworks/base/core/java/android/view/ViewRootImpl.java**

```java
boolean die(boolean immediate) {
      //die方法需要立即执行并且此时ViewRootImpl不在执行performTraversals方法
      if (immediate && !mIsInTraversal) {//1
          doDie();//2
          return false;
      }
      if (!mIsDrawing) {
          destroyHardwareRenderer();
      } else {
          Log.e(mTag, "Attempting to destroy the window while drawing!\n" +
                  "  window=" + this + ", title=" + mWindowAttributes.getTitle());
      }
      mHandler.sendEmptyMessage(MSG_DIE);
      return true;
  }
```

注释1处如果immediate为ture（需要立即执行），并且mIsInTraversal值为false则执行注释2处的代码，mIsInTraversal在执行ViewRootImpl的performTraversals方法时会被设置为true，在performTraversals方法执行完时被设置为false，因此注释1处可以理解为die方法需要立即执行并且此时ViewRootImpl不在执行performTraversals方法。注释2处的doDie方法如下所示。
 **frameworks/base/core/java/android/view/ViewRootImpl.java**

```java
void doDie() {
    //检查执行doDie方法的线程的正确性
    checkThread();//1
    if (LOCAL_LOGV) Log.v(mTag, "DIE in " + this + " of " + mSurface);
    synchronized (this) {
        if (mRemoved) {//2
            return;
        }
        mRemoved = true;//3
        if (mAdded) {//4
            dispatchDetachedFromWindow();//5
        }
        if (mAdded && !mFirst) {//6
            destroyHardwareRenderer();
            if (mView != null) {
                int viewVisibility = mView.getVisibility();
                boolean viewVisibilityChanged = mViewVisibility != viewVisibility;
                if (mWindowAttributesChanged || viewVisibilityChanged) {
                    try {
                        if ((relayoutWindow(mWindowAttributes, viewVisibility, false)
                                & WindowManagerGlobal.RELAYOUT_RES_FIRST_TIME) != 0) {
                            mWindowSession.finishDrawing(mWindow);
                        }
                    } catch (RemoteException e) {
                    }
                }
                mSurface.release();
            }
        }
        mAdded = false;
    }
    WindowManagerGlobal.getInstance().doRemoveView(this);//7
}
```

注释1处用于检查执行doDie方法的线程的正确性，注释1的内部会判断执行doDie方法线程是否是创建V的原始线程，如果不是就会抛出异常，这是因为只有创建V的原始线程才能够操作V。注释2到注释3处的代码用于防止doDie方法被重复调用。注释4处V有子View就会调用dispatchDetachedFromWindow方法来销毁View。注释6处如果V有子View并且不是第一次被添加，就会执行后面的代码逻辑。注释7处的WindowManagerGlobal的doRemoveView方法，如下所示。 **frameworks/base/core/java/android/view/WindowManagerGlobal.java**

```java
void doRemoveView(ViewRootImpl root) {
      synchronized (mLock) {
          final int index = mRoots.indexOf(root);//1
          if (index >= 0) {
              mRoots.remove(index);
              mParams.remove(index);
              final View view = mViews.remove(index);
              mDyingViews.remove(view);
          }
      }
      if (ThreadedRenderer.sTrimForeground && ThreadedRenderer.isAvailable()) {
          doTrimForeground();
      }
  }
```

WindowManagerGlobal中维护了和 Window操作相关的三个列表，doRemoveView方法会从这三个列表中清除V对应的元素。注释1处找到V对应的ViewRootImpl在ViewRootImpl列表中的索引，接着根据这个索引从ViewRootImpl列表、布局参数列表和View列表中删除与V对应的元素。 我们接着回到ViewRootImpl的doDie方法，查看注释5处的dispatchDetachedFromWindow方法里做了什么： **frameworks/base/core/java/android/view/ViewRootImpl.java**

```java
void dispatchDetachedFromWindow() {
    ...
      try {
          mWindowSession.remove(mWindow);
      } catch (RemoteException e) {
      }
      ...
  }
```

dispatchDetachedFromWindow方法中主要调用了IWindowSession的remove方法，IWindowSession在Server端的实现为Session，Session的remove方法如下所示。 **frameworks/base/services/core/java/com/android/server/wm/Session.java**

```java
public void remove(IWindow window) {
     mService.removeWindow(this, window);
 }
```

接着查看WMS的removeWindow方法： **frameworks/base/services/core/java/com/android/server/wm/WindowManagerService .java**

```java
void removeWindow(Session session, IWindow client) {
     synchronized(mWindowMap) {
         WindowState win = windowForClientLocked(session, client, false);//1
         if (win == null) {
             return;
         }
         win.removeIfPossible();//2
     }
 }
```

注释1处用于获取Window对应的WindowState，WindowState用于保存窗口的信息，在WMS中它用来描述一个窗口。接着在注释2处调用WindowState的removeIfPossible方法，如下所示。 **frameworks/base/services/core/java/com/android/server/wm/WindowState.java**

```java
Override
void removeIfPossible() {
    super.removeIfPossible();
    removeIfPossible(false /*keepVisibleDeadWindow*/);
}
```

又会调用removeIfPossible方法，如下所示。 **frameworks/base/services/core/java/com/android/server/wm/WindowState.java**

```java
private void removeIfPossible(boolean keepVisibleDeadWindow) {
        ...条件判断过滤，满足其中一个条件就会return，推迟删除操作
	removeImmediately();//1
	if (wasVisible && mService.updateOrientationFromAppTokensLocked(false, displayId)) {
		mService.mH.obtainMessage(SEND_NEW_CONFIGURATION, displayId).sendToTarget();
	}
	mService.updateFocusedWindowLocked(UPDATE_FOCUS_NORMAL, true /*updateInputWindows*/);
	Binder.restoreCallingIdentity(origId);
}
```

removeIfPossible方法和它的名字一样，并不是直接执行删除操作，而是进行多个条件判断过滤，满足其中一个条件就会return，推迟删除操作。比如这时V正在运行一个动画，这时就得推迟删除操作，直到动画完成。通过这些条件判断过滤就会执行注释1处的removeImmediately方法： **frameworks/base/services/core/java/com/android/server/wm/WindowState.java**

```java
@Override
void removeImmediately() {
    super.removeImmediately();
    if (mRemoved) {//1
        if (DEBUG_ADD_REMOVE) Slog.v(TAG_WM,
                "WS.removeImmediately: " + this + " Already removed...");
        return;
    }
    mRemoved = true;//2
    ...
    mPolicy.removeWindowLw(this);//3
    disposeInputChannel();
    mWinAnimator.destroyDeferredSurfaceLocked();
    mWinAnimator.destroySurfaceLocked();
    mSession.windowRemovedLocked();//4
    try {
        mClient.asBinder().unlinkToDeath(mDeathRecipient, 0);
    } catch (RuntimeException e) {          
    }
    mService.postWindowRemoveCleanupLocked(this);//5
}
```

removeImmediately方法如同它的名字一样，用于立即进行删除操作。注释1处的mRemoved为true意味着正在执行删除Window操作，注释1到注释2处之间的代码用于防止重复删除操作。注释3处如果当前要删除的Window是StatusBar或者NavigationBar就会将这个Window从对应的控制器中删除。注释4处会将V对应的Session从WMS的`ArraySet<Session> mSessions`中删除并清除Session对应的SurfaceSession资源（SurfaceSession是SurfaceFlinger的一个连接，通过这个连接可以创建1个或者多个Surface并渲染到屏幕上 ）。注释5处调用了WMS的postWindowRemoveCleanupLocked方法用于对V进行一些集中的清理工作，这里就不在继续深挖下去，有兴趣的同学可以自行查看源码。

Window的删除过程就讲到这里，虽然删除的操作逻辑比较复杂，但是可以简单的总结为以下4点：

1. 检查删除线程的正确性，如果不正确就抛出异常。
2. 从ViewRootImpl列表、布局参数列表和View列表中删除与V对应的元素。
3. 判断是否可以直接执行删除操作，如果不能就推迟删除操作。
4. 执行删除操作，清理和释放与V相关的一切资源。

