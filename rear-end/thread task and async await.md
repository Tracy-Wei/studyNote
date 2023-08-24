## Task 和 Thread 的区别：

thread 是单核多线程，task 是多核多线程。

### Thread

.NET Framework 在 System.Threading 下提供线程的相关类。**一个线程是一组执行指令**。

### Task

.NET Framework 提供了 Threading.Task 类，允许创建任务和异步运行它们。Task 有 Wait、ContinueWith、Cancel 等操作，有返回值。

### Thread 与 Task 的区别

1. Thread 类主要用于实现线程的创建以及执行。（同步）
2. Task 类表示以异步方式执行的单个操作。（异步）
3. Task 是基于 Thread 的，是比较高层级的封装，Task 最终还是需要 Thread 来执行
4. Task 默认使用后台线程执行，Thread 默认使用前台线程
5. Task 可以有返回值，Thread 没有返回值
6. Task 可以执行后续操作，Thread 不能执行后续操作
7. Task 可取消任务执行，Thread 不行
8. 异常传播：Thread 在父方法上获取不到异常，而 Task 可以。

代码示例：

```javascript
namespace myC;

public class Class1
{

    static void Main(string[] args)
    {
        // Thread代码示例：3秒后结束
        Thread thread = new Thread(obg => { Thread.Sleep(3000); });
        thread.Start();

        // Task代码示例：瞬间结束
        Task<int> task = new Task<int>(() =>
        {
            Thread.Sleep(3000);
            return 1;
        });
        task.Start();
    }
}
```

### async/await

async 用来修饰方法，表明这个方法是异步的，声明的方法的返回类型必须为：void，Task 或 Task<TResult>。

await 必须用来修饰 Task 或 Task<TResult>，而且只能出现在已经用 async 关键字修饰的异步方法中。通常情况下，async/await 成对出现才有意义。

```javascript
namespace myC;

public class Class1
{
    static async Task<int> GetStrLengthAsync()
    {
        Console.WriteLine("Get方法执行");
        string str = await GetString();
        Console.WriteLine("Get方法结束");
        return str.Length;
    }

    static Task<string> GetString()
    {
        return Task<string>.Run(() =>
        {
            Thread.Sleep(2000);
            return "GetString的返回值";
        });
    }

    static void Main(string[] args)
    {
        Console.WriteLine("主线程");
        Task<int> task = GetStrLengthAsync();
        Console.WriteLine("主线程继续执行");
        Console.WriteLine("Task返回的值" + task.Result);
        Console.WriteLine("主线程结束");
    }

}
```

---

### 进程与线程

- 进程（process）：
  当一个程序开始运行时，它就是一个进程，进程包括运行中的程序和程序所使用到的内存和系统资源。
  而一个进程又是由多个线程所组成的。
- 线程（thread）：
  线程是程序中的一个执行流，每个线程都有自己的专有寄存器(栈指针、程序计数器等)，但代码区是共享的。
  多线程是指程序中包含多个执行流，即在一个程序中可以同时运行多个不同的线程来执行不同的任务，也就是说允许单个程序创建多个并行执行的线程来完成各自的任务。

### 同步与异步

- 同步（sync）
  发出一个功能调用时，在没有得到结果之前，该调用就不返回。
- 异步（async）
  与同步相对，调用在发出之后，这个调用就直接返回了，所以没有返回结果。当这个调用完成后，一般通过状态、通知和回调来通知调用者。对于异步调用，调用的返回并不受调用者控制。
  - 通知调用者的三种方式：
    状态：即监听被调用者的状态（轮询），调用者需要每隔一定时间检查一次，效率会很低。
    通知：当被调用者执行完成后，发出通知告知调用者，无需消耗太多性能。
    回调：与通知类似，当被调用者执行完成后，会调用调用者提供的回调函数。

### 阻塞与非阻塞

- 阻塞（block）
  阻塞调用是指调用结果返回（或者收到通知）之前，当前线程会被挂起，即不继续执行后续操作。
  简单来说，等前一件做完了才能做下一件事。
- 非阻塞（non-block）
  阻塞调用指在不能立刻得到结果之前，该调用不会阻塞当前线程。

### 线程池

全称为托管线程池

应用场景：任务并行库 (TPL)操作、异步 I/O 完成、计时器回调、注册的等待操作、使用委托的异步方法调用和套接字连接。

把创建的线程存起来，形成一个线程池(里面有多个线程)，当要处理任务时，若线程池中有空闲线程(前一个任务执行完成后，线程不会被回收，会被设置为空闲状态)，则直接调用线程池中的线程执行(例 asp.net 处理机制中的 Application 对象)，

通过 <kbd>System.Threading.ThreadPool</kbd> 类，我们可以使用线程池。ThreadPool 类是静态类，它提供一个线程池，可用于执行任务、发送工作项、处理异步 I/O、代表其他线程等待以及处理计时器。

要点：

1. 不要将长时间运行的操作放进线程池中；
2. 不应该阻塞线程池中的线程；
3. 线程池中的线程都是后台线程(又称工作者线程)；

ThreadPool 有一个 <kbd>QueueUserWorkItem()</kbd> 方法，该方法接受一个代表用户异步操作的委托(名为 WaitCallback )，调用此方法传入委托后，就会进入线程池内部队列中。

WaitCallback 委托的定义如下：

```javascript
public delegate void WaitCallback(object state);
```
