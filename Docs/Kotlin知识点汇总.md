- [语法](#语法)
  - [对象](#对象)
  - [类](#类)
  - [继承](#继承)
  - [变量](#变量)
  - [常量](#常量)
  - [静态常量](#静态常量)
  - [定义方法](#定义方法)
  - [重载方法](#重载方法)
  - [基本数据类型](#基本数据类型)
  - [比较类型](#比较类型)
  - [转换符](#转换符)
  - [字符串比较](#字符串比较)
  - [数组](#数组)
  - [循环](#循环)
  - [角标循环](#角标循环)
  - [高级循环](#高级循环)
  - [判断器](#判断器)
  - [构造函数](#构造函数)
  - [类创建](#类创建)
  - [私有化 set 方法](#私有化-set-方法)
  - [私有化 get 方法](#私有化-get-方法)
  - [枚举](#枚举)
  - [接口](#接口)
  - [匿名内部类](#匿名内部类)
  - [内部类](#内部类)
  - [内部类访问外部类同名变量](#内部类访问外部类同名变量)
  - [抽象类](#抽象类)
  - [静态变量和方法](#静态变量和方法)
  - [可变参数](#可变参数)
  - [泛型](#泛型)
  - [构造代码块](#构造代码块)
  - [静态代码块](#静态代码块)
  - [方法代码块](#方法代码块)
  - [可见修饰符](#可见修饰符)
  - [无需 findViewById](#无需-findViewById)
  - [Lambda](#Lambda)
  - [函数变量](#函数变量)
  - [空安全](#空安全)
  - [方法支持添加默认参数](#方法支持添加默认参数)
  - [类方法扩展](#类方法扩展)
  - [运算符重载](#运算符重载)
  - [扩展函数](#扩展函数)
    - [let 函数](#let-函数)
    - [with 函数](#with-函数)
    - [run 函数](#run-函数)
    - [apply 函数](#apply-函数)
    - [also 函数](#also-函数)
    - [总结](#总结)
  - [协程](#协程)

# 语法
随着 Kotlin 越来越火爆，学习 Kotlin 已经成为我们的必经之路

多余的话就不说了，代码是最好的老师

## 对象
> Java 的写法
```java
MainActivity.this
```

> Kotlin 的写法
```kotlin
this@MainActivity
```

## 类
> Java 的写法
```java
MainActivity.class
```

> Kotlin 的写法
```kotlin
MainActivity::class.java
```

## 继承
> Java 的写法
```java
public class MainActivity extends AppCompatActivity {
    
}
```

> Kotlin 的写法（在 Kotlin 中被继承类必须被 open 关键字修饰）
```kotlin
class MainActivity : AppCompatActivity() {
    
}
```

## 变量
> Java 的写法
```java
Intent intent = new Intent();
```

> Kotlin 的写法
```kotlin
var intent = Intent()
```

## 常量
> Java 的写法
```java
final String text = "";
```

> Kotlin 的写法
```kotlin
val text = ""
```

## 静态常量
> Java 的写法
```java
public class MainActivity extends AppCompatActivity {

    static final String text = "";
}
```

> Kotlin 的写法（需要注意的是要把静态变量定义在类上方）
```kotlin
const val text = ""

class MainActivity : AppCompatActivity() {

}
```

## 定义方法
> Java 的写法
```java
public void test(String message) {

}
```

> Kotlin 的写法（Unit 跟 void 一样效果）
```kotlin
fun test(message : String) : Unit {

}
// 在 Kotlin 可以省略 Unit 这种返回值
fun test(message : String) {

}
```

## 重载方法
> Java 的写法
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }
}
```

> Kotlin 的写法
```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }
}
```

## 基本数据类型
> Java 的写法
```java
int i = 1;
long l = 2;
boolean b = true;
float f = 0;
double d = 0;
char c = 'A';
String s = "text";
```

> Kotlin 的写法
```kotlin
var i : Int = 1
var l : Long = 2
var b : Boolean = true
var f : Float = 0F
var d : Double = 0.0
var c : Char = 'A'
var s : String = "text"
// 更简洁点可以这样，自动推倒类型
var i = 1
var l = 2
var b = true
var f = 0F
var d = 0.0
var c = 'A'
var s = "text"
```

## 比较类型
> Java 的写法
```java
if ("" instanceof String) {

}
```

> Kotlin 的写法
```kotlin
if ("" is String) {

}
```

## 转换符
> Java 的写法
```java
int number = 100;
System.out.println(String.format("商品数量有%d", number));
```

> Kotlin 的写法
```kotlin
var number = 100
println("商品数量有${number}")
// 换种简洁的写法
var number = 100
println("商品数量有$number")
// 如果不想字符串被转义可以使用\$
var number = 100
println("商品数量有\$number")
```

## 字符串比较
> Java 的写法
```java
String s1 = "text";
String s2 = "text";
if (s1.equals(s2)) {
    
}
```

> Kotlin 的写法（Kotlin 对字符串比较的写法进行优化了，其他类型对象对比还是要用 equals 方法）
```kotlin
var s1 = "text"
var s2 = "text"
if (s1 == s2) {

}
```

## 数组
> Java 的写法
```java
int[] array1 = {1, 2, 3};
float[] array2 = {1f, 2f, 3f};
String[] array3 = {"1", "2", "3"};
```

>Kotlin 的写法
```kotlin
val array1 = intArrayOf(1, 2, 3)
val array2 = floatArrayOf(1f, 2f, 3f)
val array3 = arrayListOf("1", "2", "3")
```

## 循环
> Java 的写法
```java
String[] array = {"1", "2", "3"};

for (int i = 0; i < array.length; i++) {
    System.out.println(array[i]);
}
```

> Kotlin 的写法
```kotlin
val array = arrayListOf("1", "2", "3")

for (i in array.indices) {
    println(array[i])
}
```

## 角标循环
> Java 的写法
```java
String[] array = {"1", "2", "3"};

for (int i = 1; i < array.length; i++) {
    System.out.println(array[i]);
}
```

> Kotlin 的写法（这种写法在 Kotlin 中称之为区间）
```kotlin
val array = arrayListOf("1", "2", "3")

for (i in IntRange(1, array.size - 1)) {
    println(array[i])
}
// 换种更简洁的写法
val array = arrayListOf("1", "2", "3")

for (i in 1..array.size - 1) {
    println(array[i])
}
// 编译器提示要我们换种写法
val array = arrayListOf("1", "2", "3")

for (i in 1 until array.size) {
    println(array[i])
}
```

## 高级循环
> Java 的写法
```java
String[] array = {"1", "2", "3"};

for (String text : array) {
    System.out.println(text);
}
```

> Kotlin 的写法
```kotlin
val array = arrayListOf("1", "2", "3")

for (text in array) {
    println(text)
}
```

## 判断器
> Java 的写法
```java
int count = 1;

switch (count) {
    case 0:
        System.out.println(count);
        break;
    case 1:
    case 2:
        System.out.println(count);
        break;
    default:
        System.out.println(count);
        break;
}
```

> Kotlin 的写法
```kotlin
var count = 1

when (count) {
    0 -> {
        println(count)
    }
    in 1..2 -> {
        println(count)
    }
    else -> {
        println(count)
    }
}
var count = 1

// 换种更简洁的写法
when (count) {
    0 -> println(count)
    in 1..2 -> println(count)
    else -> println(count)
}
```

## 构造函数
> Java 的写法
```java
public class MyView extends View {

    public MyView(Context context) {
        this(context, null);
    }

    public MyView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MyView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
}
```

> Kotlin 的写法
```kotlin
class MyView : View {

    constructor(context : Context) : this(context, null) {

    }

    constructor(context : Context, attrs : AttributeSet?) : this(context, attrs, 0) {

    }

    constructor(context : Context, attrs : AttributeSet?, defStyleAttr : Int) : super(context, attrs, defStyleAttr) {

    }
}
// 换种更简洁的写法
class MyView : View {

    constructor(context : Context) : this(context, null)

    constructor(context : Context, attrs : AttributeSet?) : this(context, attrs, 0)

    constructor(context : Context, attrs : AttributeSet?, defStyleAttr : Int) : super(context, attrs, defStyleAttr)
}
// 只有一种构造函数的还可以这样写
class MyView(context: Context?) : View(context) {

}
```

## 类创建
> Java 的写法
```java
public class Person {

    String name;
    int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
Person person = new Person("CurvedBowZhang", 100);
person.setName("ZJX");
person.setAge(50);
System.out.println("name: " + person.getName() + ", age: " + person.getAge());
```

> Kotlin 的写法（如果不想暴露成员变量的set方法，可以将 var 改成 val )
```kotlin
class Person {

    var name : String? = null
    get() = field
    set(value) {field = value}

    var age : Int = 0
    get() = field
    set(value) {field = value}
}
// 换种更简洁的写法
class Person(var name : String, var age : Int)
var person = Person("CurvedBowZhang", 100)
person.name = "ZJX"
person.age = 50
println("name: {$person.name}, age: {$person.age}")
```

## 私有化 set 方法
> Java 的写法
```java
public class Person {

    String name;
    int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    private void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    private void setAge(int age) {
        this.age = age;
    }
}
```

> Kotlin 的写法
```kotlin
class Person {

    var name : String? = null
    private set

    var age : Int = 0
    private set
}
```

## 私有化 get 方法
> Java 的写法
```java
public class Person {

    String name;
    int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    private String getName() {
        return name;
    }

    private void setName(String name) {
        this.name = name;
    }

    private int getAge() {
        return age;
    }

    private void setAge(int age) {
        this.age = age;
    }
}
```

> Kotlin 的写法
```kotlin
class Person {

    private var name : String? = null

    private var age : Int = 0
}
```

## 枚举
> Java 的写法
```java
enum Sex {

    MAN(true), WOMAN(false);

    Sex(boolean isMan) {}
}
```

> Kotlin 的写法
```kotlin
enum class Sex (var isMan: Boolean) {

    MAN(true), WOMAN(false)
}
```

## 接口
> Java 的写法
```java
public interface Callback {
    void onSuccess();
    void onFail();
}
```

> Kotlin 的写法（Kotlin接口方法里面是可以自己实现，这里就不再演示了）
```kotlin
interface Callback {
    fun onSuccess()
    fun onFail()
}
```

## 匿名内部类
> Java 的写法
```java
new Callback() {

    @Override
    public void onSuccess() {
        
    }

    @Override
    public void onFail() {

    }
};
```

> Kotlin 的写法
```kotlin
object:Callback {

    override fun onSuccess() {
        
    }

    override fun onFail() {
        
    }
}
```

## 内部类
> Java 的写法
```java
public class MainActivity extends AppCompatActivity {

    public class MyTask {

    }
}
```

> Kotlin 的写法
```kotlin
class MainActivity : AppCompatActivity() {

    inner class MyTask {
        
    }
}
```

## 内部类访问外部类同名变量
> Java 的写法
```java
String name = "CurvedBowZhang";

public class MyTask {

    String name = "ZJX";

    public void show() {
        System.out.println(name + "---" + MainActivity.this.name);
    }
}
```

> Kotlin 的写法
```kotlin
var name = "CurvedBowZhang"

inner class MyTask {

    var name = "ZJX"

    fun show() {
        println(name + "---" + this@MainActivity.name)
    }
}
```

## 抽象类
> Java 的写法
```java
public abstract class BaseActivity extends AppCompatActivity implements Runnable {

    abstract void init();
}
```

> Kotlin 的写法
```kotlin
abstract class BaseActivity : AppCompatActivity(), Runnable {

    abstract fun init()
}
```

## 静态变量和方法
> Java 的写法
```
public class ToastUtils {

    public static Toast sToast;

    public static void show() {
        sToast.show();
    }
}
```

> Kotlin 的写法（在 Kotlin 将这种方式称之为伴生对象）
```kotlin
companion object ToastUtils {

    var sToast : Toast? = null

    fun show() {
        sToast!!.show()
    }
}
```

## 可变参数
> Java 的写法
```java
public int add(int... array) {
    int count = 0;
    for (int i : array) {
        count += i;
    }
    return count;
}
```

> Kotlin 的写法
```kotlin
fun add(vararg array: Int) : Int {
    var count = 0
    //for (i in array) {
    //    count += i
    //}
    array.forEach {
        count += it
    }
    return count
}
```

## 泛型
> Java 的写法
```java
public class Bean<T extends String> {

    T data;
    public Bean(T t) {
        this.data = t;
    }
}
Bean<String> bean = new Bean<>("666666");
```

> Kotlin 的写法
```kotlin
class Bean<T : Comparable<String>>(t: T) {
    var data = t
}
var bean = Bean<String>("666666")
// 换种更简洁的写法
var bean = Bean("666666")
```

## 构造代码块
> Java 的写法
```java
public class MainActivity extends AppCompatActivity {

    int number;

    {
        number = 1;
    }
}
```

> Kotlin 的写法
```kotlin
class MainActivity : AppCompatActivity() {

    var number = 0

    init {
        number = 1
    }
}
```

## 静态代码块
> Java 的写法
```java
public class MainActivity extends AppCompatActivity {

    static int number;

    static {
        number = 1;
    }
}
```

> Kotlin 的写法
```kotlin
class MainActivity : AppCompatActivity() {

    companion object {
        
        var number = 0
        
        init {
            number = 1
        }
    }
}
```

## 方法代码块
> Java 的写法
```java
void test(){
    {
        int a = 1;
    }
}
```

> Kotlin 的写法
```kotlin
fun test() {
    run {
        var a =1
    }
}
```

## 可见修饰符
> Java 的写法（默认为 default）

| 修饰符 | 作用 |
|:--------:| :--------:|
| public | 所有类可见 |
| protected | 子类可见 |
| default | 同一包下的类可见 |
| private | 仅对自己类可见 |

> Kotlin 的写法（默认为 public）

| 修饰符 | 作用 |
|:--------:| :--------:|
| public | 所有类可见 |
| internal | 同 Module 下的类可见 |
| protected | 子类可见 |
| private | 仅对自己类可见 |






## 无需 findViewById
> 在布局中定义
```kotlin
<TextView
    android:id="@+id/tv_content"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Hello World!" />
```

> 直接设置 TextView 的文本
```kotlin
tv_content.text = "改变文本"
```

## Lambda
> Lambda 表达式虽然在 Java JDK 已经加上了，但是没有普及开来，现在搭配 Kotlin 是一个不错的选择
```kotlin
tv_content.setOnClickListener(View.OnClickListener(

    fun(v : View) {
        v.visibility = View.GONE
    }
))
```

> 现在可以用 Lambda 表达式进行简化
```kotlin
tv_content.setOnClickListener { v -> v.visibility = View.GONE }
```

## 函数变量
> 在 Kotlin 语法中函数是可以作为变量进行传递的
```kotlin
var result = fun(number1 : Int, number2 : Int) : Int {
    return number1 + number2
}
```

> 使用这个函数变量
```kotlin
println(result(1, 2))
```

## 空安全
> 在 Java 不用强制我们处理空对象，所以常常会导致 NullPointerException 空指针出现，现在 Kotlin 对空对象进行了限定，必须在编译时处理对象是否为空的情况，不然会编译不通过

> 在对象不可空的情况下，可以直接使用这个对象
```kotlin
fun getText() : String {
    return "text"
}
```

```kotlin
val text = getText()
print(text.length)
```

> 在对象可空的情况下，必须要判断对象是否为空
```kotlin
fun getText() : String? {
    return null
}
```

```kotlin
val text = getText()
if (text != null) {
    print(text.length)
}
```

```kotlin
// 如果不想判断是否为空，可以直接这样，如果 text 对象为空，则会报空指针异常，一般情况下不推荐这样使用
val text = getText()
print(text!!.length)
```

```kotlin
// 还有一种更好的处理方式，如果 text 对象为空则不会报错，但是 text.length 的结果会等于 null
val text = getText()
print(text?.length)
```

## 方法支持添加默认参数
> 在 Java 上，我们可能会为了扩展某个方法而进行多次重载
```java
public void toast(String text) {
    toast(this, text, Toast.LENGTH_SHORT);
}

public void toast(Context context, String text) {
    toast(context, text, Toast.LENGTH_SHORT);
}

public void toast(Context context, String text, int time) {
    Toast.makeText(context, text, time).show();
}
```

```java
toast("弹个吐司");
toast(this, "弹个吐司");
toast(this, "弹个吐司", Toast.LENGTH_LONG);
```

> 但是在 Kotlin 上面，我们无需进行重载，可以直接在方法上面直接定义参数的默认值
```kotlin
fun toast(context : Context = this, text : String, time : Int = Toast.LENGTH_SHORT) {
    Toast.makeText(context, text, time).show()
}
```

```kotlin
toast(text = "弹个吐司")
toast(this, "弹个吐司")
toast(this, "弹个吐司", Toast.LENGTH_LONG)
```

## 类方法扩展
> 可以在不用继承的情况下对扩展原有类的方法，例如对 String 类进行扩展方法
```kotlin
fun String.handle() : String {
    return this + "CurvedBowZhang"
}
```

```kotlin
// 需要注意，handle 方法在哪个类中被定义，这种扩展只能在那个类里面才能使用
print("ZJX = ".handle())
```

```kotlin
ZJX = CurvedBowZhang
```

## 运算符重载
> 在 Kotlin 中使用运算符最终也会调用对象对应的方法，我们可以通过重写这些方法使得这个对象支持运算符，这里不再演示代码

| 运算符 | 调用方法 |
|:--------:| :--------:|
| +a | a.unaryPlus() |
| -a | a.unaryMinus() |
| !a | a.not() |

| 运算符 | 调用方法 |
|:--------:| :--------:|
| a++ | a.inc() |
| a-- | a.dec() |

| 运算符 | 调用方法 |
|:--------:| :--------:|
| a + b | a.plus(b) |
| a - b | a.minus(b) |
| a * b | a.times(b) |
| a / b | a.div(b) |
| a % b | a.rem(b), a.mod(b) (deprecated) |
| a..b | a.rangeTo(b) |

| 运算符 | 调用方法 |
|:--------:| :--------:|
| a in b | b.contains(a) |
| a !in b | !b.contains(a) |

| 运算符 | 调用方法 |
|:--------:| :--------:|
| a[i] | a.get(i) |
| a[i, j] | a.get(i, j) |
| a[i_1, ..., i_n] | a.get(i_1, ..., i_n) |
| a[i] = b | a.set(i, b) |
| a[i, j] = b | a.set(i, j, b) |
| a[i_1, ..., i_n] = b | a.set(i_1, ..., i_n, b) |

| 运算符 | 调用方法 |
|:--------:| :--------:|
| a() | a.invoke() |
| a(i) | a.invoke(i) |
| a(i, j) | a.invoke(i, j) |
| a(i_1, ..., i_n) | a.invoke(i_1, ..., i_n) |

| 运算符 | 调用方法 |
|:--------:| :--------:|
| a += b | a.plusAssign(b) |
| a -= b | a.minusAssign(b) |
| a *= b | a.timesAssign(b) |
| a /= b | a.divAssign(b) |
| a %= b | a.remAssign(b), a.modAssign(b) (deprecated) |

| 运算符 | 调用方法 |
|:--------:| :--------:|
| a == b | a?.equals(b) ?: (b === null) |
| a != b | !(a?.equals(b) ?: (b === null)) |

| 运算符 | 调用方法 |
|:--------:| :--------:|
| a > b | a.compareTo(b) > 0 |
| a < b | a.compareTo(b) < 0 |
| a >= b | a.compareTo(b) >= 0 |
| a <= b | a.compareTo(b) <= 0 |


## 扩展函数
> 扩展函数是 Kotlin 用于简化一些代码的书写产生的，其中有 let、with、run、apply、also 五个函数

### let 函数

> 在函数块内可以通过 it 指代该对象。返回值为函数块的最后一行或指定return表达式

> 一般写法
```kotlin
fun main() {
    val text = "CurvedBowZhang"
    println(text.length)
    val result = 1000
    println(result)
}
```

> let 写法
```kotlin
fun main() {
    val result = "CurvedBowZhang".let {
        println(it.length)
        1000
    }
    println(result)
}
```

> 最常用的场景就是使用let函数处理需要针对一个可null的对象统一做判空处理
```kotlin
mVideoPlayer?.setVideoView(activity.course_video_view)
mVideoPlayer?.setControllerView(activity.course_video_controller_view)
mVideoPlayer?.setCurtainView(activity.course_video_curtain_view)
```

```kotlin
mVideoPlayer?.let {
   it.setVideoView(activity.course_video_view)
   it.setControllerView(activity.course_video_controller_view)
   it.setCurtainView(activity.course_video_curtain_view)
}
```

> 又或者是需要去明确一个变量所处特定的作用域范围内可以使用

### with 函数

> 前面的几个函数使用方式略有不同，因为它不是以扩展的形式存在的。它是将某对象作为函数的参数，在函数块内可以通过 this 指代该对象。返回值为函数块的最后一行或指定return表达式

> 定义 Person 类
```kotlin
class Person(var name : String, var age : Int)
```

> 一般写法
```kotlin
fun main() {
    var person = Person("CurvedBowZhang", 100)
    println(person.name + person.age)
    var result = 1000
    println(result)
}
```

> with 写法
```kotlin
fun main() {
    var result = with(Person("CurvedBowZhang", 100)) {
        println(name + age)
        1000
    }
    println(result)
}
```

> 适用于调用同一个类的多个方法时，可以省去类名重复，直接调用类的方法即可，经常用于Android中RecyclerView中onBinderViewHolder中，数据model的属性映射到UI上
```kotlin
override fun onBindViewHolder(holder: ViewHolder, position: Int){
    val item = getItem(position)?: return
    holder.nameView.text = "姓名：${item.name}"
    holder.ageView.text = "年龄：${item.age}"
}
```

```kotlin
override fun onBindViewHolder(holder: ViewHolder, position: Int){
    val item = getItem(position)?: return
    with(item){
        holder.nameView.text = "姓名：$name"
        holder.ageView.text = "年龄：$age"
    }
}
```

### run 函数

> 实际上可以说是let和with两个函数的结合体，run函数只接收一个lambda函数为参数，以闭包形式返回，返回值为最后一行的值或者指定的return的表达式

> 一般写法
```kotlin
var person = Person("CurvedBowZhang", 100)
println(person.name + "+" + person.age)
var result = 1000
println(result)
```

> run 写法
```kotlin
var person = Person("CurvedBowZhang", 100)
var result = person.run {
    println("$name + $age")
    1000
}
println(result)
```

> 适用于let,with函数任何场景。因为run函数是let,with两个函数结合体，准确来说它弥补了let函数在函数体内必须使用it参数替代对象，在run函数中可以像with函数一样可以省略，直接访问实例的公有属性和方法，另一方面它弥补了with函数传入对象判空问题，在run函数中可以像let函数一样做判空处理，这里还是借助 onBindViewHolder 案例进行简化
```kotlin
override fun onBindViewHolder(holder: ViewHolder, position: Int){
    val item = getItem(position)?: return
    holder.nameView.text = "姓名：${item.name}"
    holder.ageView.text = "年龄：${item.age}"
}
```

```kotlin
override fun onBindViewHolder(holder: ViewHolder, position: Int){
    val item = getItem(position)?: return
    item?.run {
        holder.nameView.text = "姓名：$name"
        holder.ageView.text = "年龄：$age"
    }
}
```

### apply 函数

> 从结构上来看apply函数和run函数很像，唯一不同点就是它们各自返回的值不一样，run函数是以闭包形式返回最后一行代码的值，而apply函数的返回的是传入对象的本身

> 一般写法
```kotlin
val person = Person("CurvedBowZhang", 100)
person.name = "ZJX"
person.age = 50
```

> apply 写法
```kotlin
val person = Person("CurvedBowZhang", 100).apply {
    name = "ZJX"
    age = 50
}
```

> 整体作用功能和run函数很像，唯一不同点就是它返回的值是对象本身，而run函数是一个闭包形式返回，返回的是最后一行的值。正是基于这一点差异它的适用场景稍微与run函数有点不一样。apply一般用于一个对象实例初始化的时候，需要对对象中的属性进行赋值。或者动态inflate出一个XML的View的时候需要给View绑定数据也会用到，这种情景非常常见。特别是在我们开发中会有一些数据model向View model转化实例化的过程中需要用到
```kotlin
mRootView = View.inflate(activity, R.layout.example_view, null)
mRootView.tv_cancel.paint.isFakeBoldText = true
mRootView.tv_confirm.paint.isFakeBoldText = true
mRootView.seek_bar.max = 10
mRootView.seek_bar.progress = 0
```

> 使用 apply 函数后的代码是这样的
```kotlin
mRootView = View.inflate(activity, R.layout.example_view, null).apply {
   tv_cancel.paint.isFakeBoldText = true
   tv_confirm.paint.isFakeBoldText = true
   seek_bar.max = 10
   seek_bar.progress = 0
}
```

> 多层级判空问题
```kotlin
if (mSectionMetaData == null || mSectionMetaData.questionnaire == null || mSectionMetaData.section == null) {
    return;
}
if (mSectionMetaData.questionnaire.userProject != null) {
    renderAnalysis();
    return;
}
if (mSectionMetaData.section != null && !mSectionMetaData.section.sectionArticles.isEmpty()) {
    fetchQuestionData();
    return;
}
```

> kotlin的apply函数优化
```kotlin
mSectionMetaData?.apply {

    //mSectionMetaData不为空的时候操作mSectionMetaData

}?.questionnaire?.apply {

    //questionnaire不为空的时候操作questionnaire

}?.section?.apply {

    //section不为空的时候操作section

}?.sectionArticle?.apply {

    //sectionArticle不为空的时候操作sectionArticle

}
```

### also 函数

> also函数的结构实际上和let很像唯一的区别就是返回值的不一样，let是以闭包的形式返回，返回函数体内最后一行的值，如果最后一行为空就返回一个Unit类型的默认值。而also函数返回的则是传入对象的本身
```kotlin
fun main() {
    val result = "CurvedBowZhang".let {
        println(it.length)
        1000
    }
    println(result) // 打印：1000
}
```

```kotlin
fun main() {
    val result = "CurvedBowZhang".also {
        println(it.length)
    }
    println(result) // 打印：CurvedBowZhang
}
```

> 适用于let函数的任何场景，also函数和let很像，只是唯一的不同点就是let函数最后的返回值是最后一行的返回值而also函数的返回值是返回当前的这个对象。一般可用于多个扩展函数链式调用

### 总结

> 通过以上几种函数的介绍，可以很方便优化kotlin中代码编写，整体看起来几个函数的作用很相似，但是各自又存在着不同。使用的场景有相同的地方比如run函数就是let和with的结合体

## 协程
> 子任务协作运行，优雅的处理异步问题解决方案

> 协程实际上就是极大程度的复用线程，通过让线程满载运行，达到最大程度的利用CPU，进而提升应用性能

> 在当前 app module 中配置环境和依赖（因为现在协程在 Kotlin 中是实验性的）
```kotlin
kotlin {
    experimental {
        coroutines 'enable'
    }
}

dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:0.20'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:0.20'
}
```

> 协程的三种启动方式
```kotlin
runBlocking:T     

launch:Job

async/await:Deferred
```

- runBlocking

> runBlocking 的中文翻译：运行阻塞。说太多没用，直接用代码测试一下
```kotlin
println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
println("测试开始")
runBlocking {
    println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
    println("测试延迟开始")
    delay(20000) // 因为 Activity 最长响应时间为 15 秒
    println("测试延迟结束")
}
println("测试结束")
```

```kotlin
17:02:08.686 System.out: 测试是否为主线程true
17:02:08.686 System.out: 测试开始
17:02:08.688 System.out: 测试是否为主线程true
17:02:08.688 System.out: 测试延迟开始
17:02:28.692 System.out: 测试延迟结束
17:02:28.693 System.out: 测试结束
```

> runBlocking 运行在主线程，过程中 App 出现过无响应提示，由此可见 runBlocking 和它的名称一样，真的会阻塞当前的线程，只有等 runBlocking 里面的代码执行完了才会执行 runBlocking 外面的代码

- launch

> launch 的中文翻译：启动。甭管这是啥，直接用代码测试
```kotlin
println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
println("测试开始")
launch {
    println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
    println("测试延迟开始")
    delay(20000)
    println("测试延迟结束")
}
println("测试结束")
```

```kotlin
17:19:17.190 System.out: 测试是否为主线程true
17:19:17.190 System.out: 测试开始
17:19:17.202 System.out: 测试结束
17:19:17.203 System.out: 测试是否为主线程false
17:19:17.203 System.out: 测试延迟开始
17:19:37.223 System.out: 测试延迟结束
```

- async

> async 的中文翻译：异步。还是老套路，直接上代码

> 测试的时候是主线程，但是到了 launch 中就会变成子线程，这种效果类似 new Thread()，有木有？和 runBlocking 最不同的是 launch 没有执行顺序这个概念
```kotlin
println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
println("测试开始")
async {
    println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
    println("测试延迟开始")
    delay(20000)
    println("测试延迟结束")
}
println("测试结束")
```

```kotlin
17:29:00.694 System.out: 测试是否为主线程true
17:29:00.694 System.out: 测试开始
17:29:00.697 System.out: 测试结束
17:29:00.697 System.out: 测试是否为主线程false
17:29:00.697 System.out: 测试延迟开始
17:29:20.707 System.out: 测试延迟结束
```

> 这结果不是跟 launch 一样么？那么这两个到底有什么区别呢？，让我们先看一段测试代码
```kotlin
println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
println("测试开始")
val async = async {
    println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
    println("测试延迟开始")
    delay(20000)
    println("测试延迟结束")
    return@async "666666"
}
println("测试结束")

runBlocking {
    println("测试返回值：" + async.await())
}
```

```kotlin
17:50:57.117 System.out: 测试是否为主线程true
17:50:57.117 System.out: 测试开始
17:50:57.120 System.out: 测试结束
17:50:57.120 System.out: 测试是否为主线程false
17:50:57.120 System.out: 测试延迟开始
17:51:17.131 System.out: 测试延迟结束
17:51:17.133 System.out: 测试返回值：666666
```

> 看到这里你是否懂了，async 和 launch 还是有区别的，async 可以有返回值，通过它的 await 方法进行获取，需要注意的是这个方法只能在协程的操作符中才能调用

- 线程调度

>啥？协程有类似 RxJava 线程调度？先用 launch 试验一下
```kotlin
println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
println("测试开始")
launch(CommonPool) { // 同学们，敲重点
    println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
    println("测试延迟开始")
    delay(20000)
    println("测试延迟结束")
}
println("测试结束")
```

```kotlin
18:00:23.243 System.out: 测试是否为主线程true
18:00:23.244 System.out: 测试开始
18:00:23.246 System.out: 测试结束
18:00:23.246 System.out: 测试是否为主线程false
18:00:23.247 System.out: 测试延迟开始
18:00:43.256 System.out: 测试延迟结束
```

> Q：这个跟刚刚的代码有什么不一样吗？

> A：当然不一样，假如一个网络请求框架维护了一个线程池，一个图片加载框架也维护了一个线程池.......，你会发现其实这样不好的地方在于，这些线程池里面的线程没有被重复利用，于是乎协程主动维护了一个公共的线程池 CommonPool，很好的解决了这个问题

> Q：还有刚刚不是说能线程调度吗？为什么还是在子线程运行？

> A：因为我刚刚只用了 CommonPool 这个关键字，我再介绍另一个关键字 UI，光听名字就知道是啥了
```kotlin
println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
println("测试开始")
launch(UI) {
    println("测试是否为主线程" + (Thread.currentThread() == Looper.getMainLooper().thread))
    println("测试延迟开始")
    delay(20000)
    println("测试延迟结束")
}
println("测试结束")
```

```kotlin
18:07:20.181 System.out: 测试是否为主线程true
18:07:20.181 System.out: 测试开始
18:07:20.186 System.out: 测试结束
18:07:20.192 System.out: 测试是否为主线程true
18:07:20.192 System.out: 测试延迟开始
18:07:40.214 System.out: 测试延迟结束
```
