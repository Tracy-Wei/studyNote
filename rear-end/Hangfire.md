## Hangfire

可以在后台手动执行任务的任务调度框架。可以直观明了的查看作业调度情况，并且 Hangfire 不需要依赖于单独的应用程序执行，并且支持持久性存储。

安装：

```javascript
Install-Package Hangfire
```

配置安装：
[参考官网配置][1]

基于队列的任务处理(Fire-and-forget jobs):

- 基于队列的任务处理是 Hangfire 中最常用的，客户端使用 BackgroundJob 类的静态方法 Enqueue 来调用，传入指定的方法（或是匿名函数），Job Queue 等参数.(类似 MQ)

```javascript
var jobId = BackgroundJob.Enqueue(() => Console.WriteLine("Fire-and-forget!"));
```

延迟任务执行(Delayed jobs):

- 延迟（计划）任务跟队列任务相似，客户端调用时需要指定在一定时间间隔后调用

```javascript
var jobId = BackgroundJob.Schedule(
  () => Console.WriteLine("Delayed!"),
  TimeSpan.FromDays(7)
);
```

定时任务执行(Recurring jobs):

- 定时（循环）任务代表可以重复性执行多次，支持 CRON 表达式

```javascript
RecurringJob.AddOrUpdate(() => Console.WriteLine("Recurring!"), Cron.Daily);
```

延续性任务执行(Continuations)：

- 延续性任务类似于.NET 中的 Task,可以在第一个任务执行完之后紧接着再次执行另外的任务

```javascript
BackgroundJob.ContinueWith(jobId, () => Console.WriteLine("Continuation!"));
```

[1]: https://docs.hangfire.io/en/latest/getting-started/aspnet-core-applications.html
