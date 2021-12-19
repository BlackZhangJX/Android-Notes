- [头文件](#头文件)
- [数据类型](#数据类型)
- [typedef](#typedef)
- [类型限定符](#类型限定符)
- [定义常量](#定义常量)
- [存储类](#存储类)
- [引用 vs 指针](#引用-vs-指针)
- [struct vs class](#struct-vs-class)
- [成员函数](#成员函数)
- [析构函数](#析构函数)
- [拷贝构造函数](#拷贝构造函数)
- [friend 友元](#friend-友元)
- [inline 内联函数](#inline-内联函数)
- [继承类型](#继承类型)
- [运算符重载](#运算符重载)
- [动态内存](#动态内存)
- [命名空间](#命名空间)
- [预处理器](#预处理器)
  - [#include](#include)
  - [#define](#define)
  - [条件编译](#条件编译)
  - [预定义宏](#预定义宏)
- [信号](#信号)
- [线程](#线程)
- [强制类型转换](#强制类型转换)
  - [const_cast](#const_cast)
  - [static_cast](#static_cast)
  - [dynamic_cast](#dynamic_cast)
  - [reinterupt_cast](#reinterupt_cast)
- [智能指针](#智能指针)
  - [unique_ptr](#unique_ptr)
  - [shared_ptr](#shared_ptr)
  - [weak_ptr](#weak_ptr)
- [内存空间](#内存空间)

# 头文件
**.h** 文件中能包含：
- 类成员数据的声明，但不能赋值
- 类静态数据成员的定义和赋值，但不建议
- 类的成员函数的声明
- 非类成员函数的声明
- 常数的定义：如：constint a=5;
- 静态函数的定义
- 类的内联函数的定义

不能包含：
- 所有非静态变量（不是类的数据成员）的声明
- 默认命名空间声明不要放在头文件，using namespace std; 等应放在 .cpp 中，在 .h 文件中使用 std::string

# 数据类型

| 类型 | 位 |
|--|--|
|char|1 个字节 |
|int |4 个字节 |
|short int|2 个字节 |
|long int|8 个字节 |
|float|4 个字节 |
|double|8 个字节 |
|long double|16 个字节 |
|wchar_t|2 或 4 个字节 |

# typedef
使用 typedef 为一个已有的类型取一个新的名字
```cpp
typedef type newname; 
```

# 类型限定符
|限定符|含义 |
|--|-- |
|const	|const 类型的对象在程序执行期间不能被修改改变。 |
|volatile	|修饰符 volatile 告诉编译器不需要优化volatile声明的变量，让程序可以直接从内存中读取变量。对于一般的变量编译器会对变量进行优化，将内存中的变量值放在寄存器中以加快读写效率。 |
|restrict	|由 restrict 修饰的指针是唯一一种访问它所指向的对象的方式。只有 C99 增加了新的类型限定符 restrict。 |

# 定义常量
```cpp
// 使用 #define 预处理器
#define LENGTH 10

// const 关键字
const int WIDTH  = 5;
```

# 存储类
|存储类 |含义 |
|--|-- |
|auto| 声明变量时根据初始化表达式自动推断该变量的类型 |
|register| 用于定义存储在寄存器中而不是 RAM 中的局部变量，用于需要快速访问的变量，不能对它应用一元的 '&' 运算符（因为它没有内存位置） |
|static | 用在类数据成员上时，会导致仅有一个该成员的副本被类的所有对象共享；修饰全局变量时，会使变量的作用域限制在声明它的文件内 |
|extern | 全局变量的引用，全局变量对所有的程序文件都是可见 |
|mutable | |
|thread_local | 仅可在它在其上创建的线程上访问，变量在创建线程时创建，并在销毁线程时销毁，每个线程都有其自己的变量副本 |

# 引用 vs 指针
- 不存在空引用。引用必须连接到一块合法的内存。
- 一旦引用被初始化为一个对象，就不能被指向到另一个对象。指针可以在任何时候指向到另一个对象。
- 引用必须在创建时被初始化。指针可以在任何时间被初始化。

# struct vs class
class 和 struct 定义类的唯一区别是默认的反问权限，struct 默认是 public，class 默认是 private。

# 成员函数
成员函数可以在类的外部使用范围解析运算符 :: 定义该函数
```cpp
double Box::getVolume(void)
{
    return length * breadth * height;
}
```

# 析构函数
析构函数是类的一种特殊的成员函数，它会在每次删除所创建的对象时执行
```cpp
class Line
{
   public:
      ~Line();  // 这是析构函数声明
 
   private:
      double length;
};
 
Line::~Line(void)
{
    cout << "Object is being deleted" << endl;
}
```

# 拷贝构造函数
拷贝构造函数是一种特殊的构造函数，它在创建对象时，是使用同一类中之前创建的对象来初始化新创建的对象。
```cpp
class Line
{
   public:
      Line(int len);             // 简单的构造函数
      Line(const Line &obj);      // 拷贝构造函数
 
   private:
      int *ptr;
};

Line::Line(const Line &obj)
{
    cout << "调用拷贝构造函数并为指针 ptr 分配内存" << endl;
    ptr = new int;
    *ptr = *obj.ptr; // 拷贝值
}
```

# friend 友元
类的友元函数是定义在类外部，但有权访问类的所有私有（private）成员和保护（protected）成员。尽管友元函数的原型有在类的定义中出现过，但是友元函数并不是成员函数。

友元也可以是一个类，该类被称为友元类类。
```cpp
class Box
{
private:
   double width;
public:
   friend void printWidth(Box box);
   void setWidth(double wid);
};

// 请注意：printWidth() 不是任何类的成员函数
void printWidth(Box box)
{
   /* 因为 printWidth() 是 Box 的友元，它可以直接访问该类的任何成员 */
   cout << "Width of box : " << box.width <<endl;
}
```

# inline 内联函数
如果一个函数是内联的，那么在编译时，编译器会把该函数的代码副本放置在每个调用该函数的地方。对内联函数进行任何修改，都需要重新编译函数的所有客户端，因为编译器需要重新更换一次所有的代码，否则将会继续使用旧的函数。

# 继承类型
```cpp
class Rectangle: public Shape 
{
    ...
}
```
|继承类型 | 说明
|--|--
|public |基类的公有成员也是派生类的公有成员，基类的保护成员也是派生类的保护成员，基类的私有成员不能直接被派生类访问
|protected |基类的公有和保护成员将成为派生类的保护成员。
|private |基类的公有和保护成员将成为派生类的私有成员。

# 运算符重载
```cpp
class Box
{
public:
   double length;      // 长度
   double breadth;     // 宽度
   
   // 重载 + 运算符，用于把两个 Box 对象相加
   Box operator+(const Box& b)
   {
      Box box;
      box.length = this->length + b.length;
      box.breadth = this->breadth + b.breadth;
      return box;
   }
};
```

# 动态内存
new 运算符为给定类型的变量在运行时分配堆内的内存，这会返回所分配的空间地址。如果不再需要动态分配的内存空间，可以使用 delete 运算符，删除之前由 new 运算符分配的内存。
```cpp
double* pvalue = new double
···
delete pvalue; 
```

# 命名空间
命名空间的定义使用关键字 namespace，后跟命名空间的名称：
```cpp
namespace namespace_name {
   // 代码声明
}
```

为了调用带有命名空间的函数或变量，需要在前面加上命名空间的名称：
```cpp
name::code;  // code 可以是变量或函数
```

使用 using namespace 指令，这样在使用命名空间时就可以不用在前面加上命名空间的名称。这个指令会告诉编译器，后续的代码将使用指定的命名空间中的名称：
```cpp
using namespace std;
```

# 预处理器
## #include
**include** 是一个来自 C 语言的宏命令，它在编译器进行编译之前，即在预编译的时候就会起作用。#include 的作用是把它后面所写的那个文件的内容一字不改地包含到当前的文件中来。值得一提的是，它本身是没有其它任何作用与副功能的，它的作用就是把每一个它出现的地方，替换成它后面所写的那个文件的内容。

- 系统自带的头文件用尖括号括起来，这样编译器会在系统文件目录下查找。
- 用户自定义的文件用双引号括起来，编译器首先会在用户目录下查找，然后在到 C++ 安装目录（比如 VC 中可以指定和修改库文件查找路径，Unix 和 Linux 中可以通过环境变量来设定）中查找，最后在系统文件中查找。

## #define
\#define 预处理指令用于创建符号常量。该符号常量通常称为宏，指令的一般形式是：
```cpp
#define macro-name replacement-text 
```

- 示例
```cpp
#define PI 3.14159
#define MIN(a,b) (a<b ? a : b)
```

## 条件编译
```cpp
#define DEBUG

#ifdef DEBUG
   cerr <<"Trace: Inside main function" << endl;
#endif
```

## 预定义宏
|宏|描述 |
|--|-- |
|\_\_LINE__	|程序编译时包含当前行号 |
|\_\_FILE__	|程序编译时包含当前文件名 |
|\_\_DATE__	|包含一个形式为 month/day/year 的字符串，它表示把源文件转换为目标代码的日期 |
|\_\_TIME__	|包含一个形式为 hour:minute:second 的字符串，它表示程序被编译的时 |

# 信号
|信号	|描述 |
|--|-- |
|SIGABRT	|程序的异常终止，如调用 abort |
|SIGFPE	|错误的算术运算，比如除以零或导致溢出的操作 |
|SIGILL	|检测非法指令 |
|SIGINT	|程序终止(interrupt)信号 |
|SIGSEGV	|非法访问内存 |
|SIGTERM	|发送到程序的终止请求 |

C++ 信号处理库提供了 signal 函数，用来捕获突发事件，raise 函数来生成信号：
```cpp
#include <iostream>
#include <csignal>
#include <unistd.h>
 
using namespace std;
 
void signalHandler( int signum )
{
    cout << "Interrupt signal (" << signum << ") received.\n";
    exit(signum);  
}
 
int main ()
{
    int i = 0;
    // 注册信号 SIGINT 和信号处理程序
    signal(SIGINT, signalHandler);  
    raise(SIGINT);
    return 0;
}
```

# 线程
c++ 11 之后有了标准的线程库
```cpp
#include <iostream>
#include <thread>

std::thread::id main_thread_id = std::this_thread::get_id();

void hello()  
{
    std::cout << "Hello Concurrent World\n";
    if (main_thread_id == std::this_thread::get_id())
        std::cout << "This is the main thread.\n";
    else
        std::cout << "This is not the main thread.\n";
}

void pause_thread(int n) {
    std::this_thread::sleep_for(std::chrono::seconds(n));
    std::cout << "pause of " << n << " seconds ended\n";
}

int main() {
    std::thread t(hello);
    std::cout << t.hardware_concurrency() << std::endl;//可以并发执行多少个(不准确)
    std::cout << "native_handle " << t.native_handle() << std::endl;//可以并发执行多少个(不准确)
    t.join();
    std::thread a(hello);
    a.detach();
    std::thread threads[5];                         // 默认构造线程

    std::cout << "Spawning 5 threads...\n";
    for (int i = 0; i < 5; ++i)
        threads[i] = std::thread(pause_thread, i + 1);   // move-assign threads
    std::cout << "Done spawning threads. Now waiting for them to join:\n";
    for (auto &thread : threads)
        thread.join();
    std::cout << "All threads joined!\n";
}
```

# 强制类型转换
## const_cast
const_cast 用于去除对象的 const 或 volatile 属性
```cpp
void Func(double& d) { ... }  

void ConstCast()  
{  
   const double pi = 3.14;  
   Func(const_cast<double&>(pi)); // No error.  
}
```

## static_cast
static_cast 强制转换只会在编译时检查，但没有运行时类型检查来保证转换的安全性。同时，static_cast 也不能去掉 expression 的 const、volitale、或者 __unaligned 属性。

其主要应用场景有：
- 用于类层次结构中基类（父类）和派生类（子类）之间指针或引用的转换。进行上行转换（把派生类的指针或引用转换成基类表示）是安全的；进行下行转换（把基类指针或引用转换成派生类表示）时，由于没有动态类型检查，所以是不安全的。
- 用于基本数据类型之间的转换，如把 int 转换成char，把 int 转换成enum。这种转换的安全性也要开发人员来保证。
- 把 void 转换成目标类型的空指针.
- 把任何类型的表达式转换成 void 类型。
```cpp
Sub sub;
Base *base_ptr = static_cast<Base*>(&sub);  
```

## dynamic_cast
dynamic_cast 运算符的主要用途：将基类的指针或引用安全地转换成派生类的指针或引用。并用派生类的指针或引用调用非虚函数。

如果是基类指针或引用调用的是虚函数无需转换就能在运行时调用派生类的虚函数。

前提条件：当我们将 dynamic_cast 用于某种类型的指针或引用时，只有该类型至少含有虚函数时(最简单是基类析构函数为虚函数)，才能进行这种转换。否则，编译器会报错。

在指针类型中，基类指针所指对象为基类类型，在这种情况下 dynamic_cast 在运行时做检查，转换失败，返回结果为 0；

在引用类型中，并不存在空引用，所以引用的 dynamic_cast 检测失败时会抛出一个 bad_cast 异常。
```cpp
Base * base = new Base;
if (Derived *der = dynamic_cast<Derived*>(base))
{
    cout << "转换成功" <<endl;
    der->Show();
}
else 
{
    cout << "转换失败" <<endl;
}
```

## reinterupt_cast
reinterpret_cast 用来处理无关类型转换，通常为操作数的位模式提供较低层次的重新解释。

推荐使用在：
- 从指针类型到一个足够大的整数类型
- 从整数类型或者枚举类型到指针类型
- 从一个指向函数的指针到另一个不同类型的指向函数的指针
- 从一个指向对象的指针到另一个不同类型的指向对象的指针
- 从一个指向类函数成员的指针到另一个指向不同类型的函数成员的指针
- 从一个指向类数据成员的指针到另一个指向不同类型的数据成员的指针

错误的使用 reinterpret_cast 很容易导致程序的不安全，只有将转换后的类型值转换回到其原始类型，这样才是正确使用 reinterpret_cast 方式。

# 智能指针
## unique_ptr
由 unique_ptr 管理的内存，只能被一个对象持有，所以，unique_ptr不支持复制和赋值。想要把一个 unique_ptr 的内存交给另外一个 unique_ptr 对象管理。只能使用 std::move 转移当前对象的所有权。转移之后，当前对象不再持有此内存，新的对象将获得专属所有权。
```cpp
auto w = std::make_unique<Widget>();
auto w2 = std::move(w); // w2 获得内存所有权，w 此时等于 nullptr
```

unique_ptr在默认情况下和裸指针的大小是一样的，所以内存上没有任何的额外消耗，性能是最优的。

## shared_ptr
多个智能指针可以共享同一个对象。shared_ptr 内部是利用引用计数来实现内存的自动管理，每当复制一个 shared_ptr，引用计数会 +1。当一个 shared_ptr 离开作用域时，引用计数会 -1。当引用计数为0的时候，则 delete 内存。

```cpp
auto w = std::make_shared<Widget>();
```

shared_ptr的内存占用是裸指针的两倍。因为除了要管理一个裸指针外，还要维护一个引用计数。

## weak_ptr
允许你共享但不拥有某对象，一旦最末一个拥有该对象的智能指针失去了所有权，任何 weak_ptr 都会自动成空。

# 内存空间
|区|秒速 |
|--|-- |
|堆|操作系统维护的一块动态分配内存，malloc 在堆上分配的内存块，使用 free 释放内存 |
|栈|由编译器自动分配释放，存放函数的参数值，局部变量的值等 |
|自由存储区|C++ 中通过 new 与 delete 动态分配和释放对象的抽象概念 |
|全局区（静态区）|全局变量和静态变量分配在此一块内存中 |
|常量存储区|存储常量字符串, 程序结束后由系统释放 |
