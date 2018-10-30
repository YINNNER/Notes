[TOC]

# Windows编程

## 线程间通信与同步

### 1. 线程及其创建过程

#### 线程

- 进程是计算机分配资源的单位，线程是运行调度单位。
- 进程中的线程也具有线程控制块，包含内容有所属进程ID，创建和退出时间，线程启动地址等。

#### 创建过程

1. 在进程的地址空间中为线程创建用户态堆栈。
2. 初始化线程硬件上下文。
3. 创建线程对象。
4. 通知内核系统为线程运行准备。
5. 新创建线程handle和线程ID值返回到调用者。
6. 线程进入调度准备执行。

#### 生命期

启动 => 执行任务 => 暂停 => 终止

#### 工作线程的结束

1. 线程正常结束
   - 自动消亡，OS清理
2. 线程非正常结束，被KILL
   - os无法控制，造成系统损失或破坏
3. 控制线程正常终止的方法
   - 低级事件对象ManualResetEvent

#### 线程非正常结束的后果

- 内存无法回收－内存泄漏
- 文件缓冲没写入－文件被破坏
- 文件句柄未回收－被占用
- 共享资源的占用(网络端口，管道，DLL)

#### 创建与启动（CODE）

```csharp
// 线程执行代码的编写
void workThread(){
    
}

// 设定函数名为线程入口
ThreadStart s = new ThreadStart(workThread);

// 线程委托对象（委托的实质是函数指针或叫函数地址）
Thread thread1 = new Thread(s);

// 设定线程优先级等属性
// 线程启动
thread1.Start();
// 线程参数传递
thread1.Start(paraObject);
```

- C#的System.Threading命名空间下的Thread类和ThreadStart类用于完成的线程创建和管理。
- 使用Thread类创建线程时，只需要提供线程入口，线程入口告诉程序让这个线程做什么。通过实例化一个Thread类的对象就可以创建一个线程。创建新的Thread对象时，将创建新的托管线程。Thread类接收一个ThreadStart委托或ParameterizedThreadStart委托的构造函数，该委托包装了调用Start方法时由新线程调用的方法。

##### a. 创建无参数方法的托管线程

```csharp
// 创建线程
Thread thread1=new Thread(new ThreadStart(method));
// 启动线程
thread1.Start();                                                         
// 定义无参方法 
static void method() {                                                
  Console.WriteLine("这是无参的静态方法");
}                    

class ThreadTest
    {
        public void MyThread()
        {
            Console.WriteLine("这是一个实例方法");
        }
 }
ThreadTest test=new ThreadTest();
//创建线程
Thread thread2=new Thread(new ThreadStart(test.MyThread()));
//启动线程
thread2.Start();                                                     


```

##### b. 利用有参的委托ParameterizedThreadStart来创建线程

```csharp
class Program
    {
        static void Main(string[] args)
        {
            //通过ParameterizedThreadStart创建线程
            Thread thread = new Thread(new ParameterizedThreadStart(Thread1));
            //给方法传值
            thread.Start("这是一个有参数的委托");
            Console.ReadKey();
        }
       /// 创建有参的方法，方法里面的参数类型必须是Object类型
       static void Thread1(object obj)
        {
            Console.WriteLine(obj);
        }
    }
```

##### c. 通过匿名委托或Lambda表达式创建线程

```csharp
//通过匿名委托创建
Thread thread1 = new Thread(delegate() {Console.WriteLine("我是通过匿名委托创建的线程"); });
thread1.Start();

//通过Lambda表达式创建
Thread thread2 = new Thread(() => Console.WriteLine("我是通过Lambda表达式创建的委托"));
thread2.Start();
```

#### 前台线程与后台线程

- 前台线程：只有所有的前台线程都结束，应用程序才能结束。默认情况下创建的线程都是前台线程。

- 后台线程：所有的后台线程在应用程序退出时都会自动结束。
  - 通过Thread.IsBackground设置后台线程。
  - 且必须在调用Start方法之前设置线程的类型，否则一旦线程运行，将无法改变其类型
- 一般后台线程用于处理时间较短的任务，如在一个Web服务器中可以利用后台线程来处理客户端发过来的请求信息。
- 而前台线程一般用于处理需要长时间等待的任务，如在Web服务器中的监听客户端请求的程序，或是定时对某些系统资源进行扫描的程序

#### 线程的优先级与调度

- windows中的线程按照优先级进行调度
- 具有最高优先权的线程一直被执行
- 相同优先级的线程 按时间片轮转执行，时间片在windows XP系统中相当于2个时钟周期。
- 当更高优先级的线程就绪时，高优先的线程会抢占执行低优先级的线程。![image-20181030105802180](images/image-20181030105802180.png)

#### 线程状态

1. 初始化--线程处于创始中。
2. 就绪--等待由CPU执行。
3. 待命--只能由一个线程处于待命状态，离执行状态最近。
4. 运行--在CPU的当前时间片内执行。
5. 等待--线程同步需要等待，
6. 接转--准备执行，但是它的内核堆栈不在内存，需要内存页面调入，调入后进入就绪状态。
7. 终止--线程执行完。

> 通过ThreadState可以检测线程是处于Unstarted、Sleeping、Running 等等状态，它比 IsAlive 属性能提供更多的特定信息。

#### 多线程

1. CPU运行速度太快，硬件处理速度跟不上，所以操作系统进行分时间片管理。这样，从宏观角度来说是多线程并发的，因为CPU速度太快，察觉不到，看起来是同一时刻执行了不同的操作。但是从微观角度来讲，同一时刻只能有一个线程在处理。
2. 目前电脑都是多核多CPU的，一个CPU在同一时刻只能运行一个线程，但是多个CPU在同一时刻就可以运行多个线程


- 多线程的优点：
  - 线程机制使可以同时完成多个任务；可以使程序的响应速度更快；可以让占用大量处理时间的任务或当前没有进行处理的任务定期将处理时间让给别的任务；可以随时停止任务；可以设置每个任务的优先级以优化程序性能。
  - 程序具有异步执行能力以充分发挥机器计算能力，程序还可以利用其他计算机的处理能力。
  - 合理的线程分工使得数据计算与用户交互得到均衡。

- 线程的并发：
  - 机器采用时间片轮转算法轮流执行线程，形成并发执行。
  - 线程可以很好平衡程序响应与数据处理，还能通过网络利用其它处理机资源。
  - 线程同步控制非常复杂，调试困难。

#### 线程应用场合

1. 网络通信程序。
2. 与Web服务器和数据库操作。
3. 执行占用大量时间的操作。
4. 有不同优先级的任务。
5. 用户响应效能与数据运算均衡。

#### 线程缺点

- 线程上下文信息消耗计算机资源。
- 线程上下文切换过程，线程会带来资源特殊要求和潜在冲突。如果线程过多，系统管理线程的负担会加大，则其中大多数线程都不会产生明显的进度。
- 线程控制代码非常复杂，并可能产生许多bug。
- 线程的非正常终结会造成资源浪费影响系统的运行性能。

### 2. 线程跨域访问

- 界面中的控件（textBox1等）是由主线程创建的，thread线程是另外创建的一个线程，在.NET上执行的是托管代码，C#强制要求这些代码必须是线程安全的，即不允许跨线程访问Windows窗体的控件；
- 在遵守.NET安全标准的前提下，实现从一个线程成功地访问另一个线程创建的空间，要使用C#的方法回调机制。
- C#的方法回调机制，也是建立在委托基础上的，下面给出它的典型实现过程。


```csharp
// 1.定义回调
 private delegate void DoSomeCallBack(Type para); 
// 2.声明回调 
DoSomeCallBack doSomaCallBack;

// 定义回调方法
void DoSomeMethod(Type para){ 
}

// 3.初始化回调方法
doSomeCallBack=new DoSomeCallBack(DoSomeMethod);
或doSomeCallBack = DoSomeMethod;

// 4.触发对象操作
控件 obj.Invoke(doSomeCallBack,arg);
或控件 obj.Dispatcher.Invoke(doSomeCallBack,arg);

```

###  3. 线程同步

![image-20181030110905459](images/image-20181030110905459.png)

1. 同步方法执行是有序的，异步方法执行是无序的。
2. 异步方法无序包括启动无序和结束无序。
   1. 启动无序是因为同一时刻向操作系统申请线程，操作系统收到申请以后，返回执行的顺序是无序的，所以启动是无序的。
   2. 结束无序是因为虽然线程执行的是同样的操作，但是每个线程的耗时是不同的，所以结束的时候不一定是先启动的线程就先结束。
3. 同步方法由于主线程忙于计算，所以会卡住界面。
4. 异步方法由于主线程执行完了，其他计算任务交给子线程去执行，所以不会卡住界面，用户体验性好。
5. 同步方法由于只有一个线程在计算，所以执行速度慢。
6. 异步方法由多个线程并发运算，所以执行速度快，但并不是线性增长的（资源可能不够）。多线程也不是越多越好，只有多个独立的任务同时运行，才能加快速度。

#### 解决线程的异步无序问题

- 使用回调来解决异步线程的无序问题，在BeginInvoke的参数中指定回调函数。

```csharp
// 定义一个回调
AsyncCallback callback = p =>
{
    Console.WriteLine($"到这里计算已经完成了。{Thread.CurrentThread.ManagedThreadId.ToString("00")}。");
    update($"到这里计算已经完成了。" + 	Thread.CurrentThread.ManagedThreadId.ToString("00") + "。");
};

// 异步调用回调
for (int i = 0; i < 5; i++)
{
    string name = string.Format($"btnSync_Click_{i}");
    asyncResult = action.BeginInvoke(name, callback, null);
}

```

###  4. 线程间同步模式

![image-20181030111954071](images/image-20181030111954071.png)

- 工作线程可以很容易用SendMessage来发消息。
- 窗体线程可以发送ManualResetEvent事件给工作线程 。

#### WaitHandle类继承关系

![image-20181030112202435](images/image-20181030112202435.png)

#### 线程接受消息

- 工作线程没有消息队列，无法用窗体模式，线程因此显得是一个笨听众，接收方法是线程主动循环检查一些变量，但不能使用"忙检"，因为太耗CPU，要使用`ManualResetEvent.WaitOne`这样的方法，以最低的代价耗费cpu资源。

#### 工作线程响应前打发时间的两种方式

1. 采用IsOut + Sleep打发时间的方式。
2. ManualResetEvent.WaitOne打发时间的方式。
   - 事件对象可实现并发执行中的前趋控制。当线程调用Wait方法时，如果等待对象状态没有激活，则调用线程暂停。对象被激活则线程继续执行。

#### 低级事件对象

- 事件对象声明
  - `public static ManualResetEvent User_Terminate_listen;`
- 全局静态使得线程与窗体可以访问
- `User_Terminate_listen.WaitOne();` 
- 代表最小的信息量(1bit)



`ManualResetEvent`：

- Set设置为有效
- Reset设置为无效



![image-20181030113444989](images/image-20181030113444989.png)

#### 工作线程运行逻辑

![image-20181030113811786](images/image-20181030113811786.png)

1. WaitOne方法等待**当前事件**(信号)有效。
2. WaitAny方法等待**事件(信号)数组中==任一==事件有效**，对应**或**关系实现同步。
3. WaitAll方法等待**事件（信号）数组中==所有==事件有效**，对应**与**关系实现同步。

> 500为TimeOut时间，超时时间。

##### ManualResetEvent.WaitOne要点

1. 一是时间效果上阻止线程继续。
2. 二是在获得信号状态后返回值为true，而超时未获得返回值为false。
3. 三是获得信号状态将不再继续未等待完的时间。
4. 它的使命是完成信号传递，至于功能是启动操作还是终止操作，程序得到的状态都是信号的true。
5. ManualResetEvent比AutoResetEvent要可靠，它可将信号传给多个线程，而线程会重置AutoResetEvent的状态，即中断信号的传递。

> 即，让线程等待一个事件，规定时间内等到了就返回true，没等到就返回false。

##### 具有与关系的同步方式

- ManualResetEvent.WaitAll方法：当所有事件状态同时激活时

##### 具体或关系的同步方式

- ManualResetEvent.WaitAny方法：当任一事件状态激活时

### 5. 线程同步与死锁

- 多个线程对共享资源访问会造成冲突。为了避免冲突，必须对共享资源进行同步或控制对共享资源的访问。如果在相同或不同的应用程序域中未能正确地使访问同步，则会导致出现一些问题，这些问题包括死锁和争用条件等，其中死锁是指两个线程都停止响应，并且都在等待对方完成；争用条件是指由于意外地出现对两个事件的执行时间的临界依赖性而发生反常的结果。

#### 需要同步的资源

1. 系统资源（如通信端口）。
2. 多个进程所共享的资源（如文件句柄）。
3. 由多个线程访问的单个应用程序域的资源（如全局、静态和实例字段）。

#### 同步控制类

![image-20181030114625986](images/image-20181030114625986.png)

#### 互斥量Mutex

##### 介绍

- 最多只能有一个线程可以获取并拥有它，它适合不同线程对同一共享资源互斥访问的应用场合，例如对全局变量，同一个文件或者是数据库同一个对象的访问等，设置程序先获得互斥量再访问共享资源。

##### 使用

- 互斥量的创建
- WaitOne方法
- ReleaseMutex方法



- 线程可调用多次的 WaitOne 方法重复对其所有，使用 ReleaseMutex 方法释放对互斥量所属权，而每一个成功的 WaitOne 方法对应一次 ReleaseMutex。
- WaitOne方法不仅等待互斥量的状态，还使线程拥有它。
- 互斥量最好不要使用WaitAny,WaitAll方法
- 线程运行终止 mutex 被放弃，互斥量仍可被其它线程取得所属权，但会获得 AbandonedMutexException 异常。

#### ManualResetEvent的使用

![image-20181030115133374](images/image-20181030115133374.png)