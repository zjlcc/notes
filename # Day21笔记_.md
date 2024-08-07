# Day21笔记

## HangFire

##### 后台处理

*Hangfire 服务器*部分负责后台作业处理。服务器不依赖于 ASP.NET，可以在任何地方启动，从控制台应用程序到 Microsoft Azure Worker Role。所有应用程序的单一 API 通过以下类公开BackgroundJobServer

```
varserver=newBackgroundJobServer();

server.Dispose();
```

`Dispose`尽可能调用该方法以使正常关闭功能正常运行。Hangfire 服务器由执行不同工作的不同组件组成：工作者监听队列并处理作业、循环调度程序将循环作业入队、调度轮询器将延迟作业入队、过期管理器删除过时的作业并尽可能保持存储清洁，等等。该`Dispose`方法是一个阻塞方法，它会等待所有组件都准备好关闭（例如，工作器会将中断的作业放回到其队列中）。因此，只有等待所有组件都准备好后，我们才能谈论优雅关闭。

严格来说，您不需要调用该Dispose方法。 Hangfire 甚至可以处理意外的进程终止，并会自动重试中断的作业。 但是，最好使用取消令牌来控制方法中的退出点。

##### 处理异常

当 Hangfire 在作业执行过程中遇到外部异常时，它会自动尝试将其状态更改为该`Failed`状态，并且始终可以在监视器 UI 中找到此作业（除非明确删除它，否则它不会过期）。

状态转换是作业过滤器可以拦截和更改初始管道的地方之一。而`AutomaticRetryAttribute`类就是其中之一，它安排失败的作业在增加延迟后自动重试。此过滤器全局应用于所有方法，默认情况下有 10 次重试机会。并且每次尝试失败时都会收到警告日志消息。如果重试次数超过其最大值，则作业将移至该`Failed`状态（带有错误日志消息），可以手动重试。
放置一个显式属性，可以修改重试次数：

```
[AutomaticRetry(Attempts = 0)]
publicvoidBackgroundMethod()
{
}
```

如果要更改默认全局值，可以添加新的全局过滤器：

```
GlobalJobFilters.Filters.Add(newAutomaticRetryAttribute{Attempts=5});
```

如果使用的是 ASP.NET Core，则可以使用 IServiceCollection 扩展方法 AddHangfire。AddHangfire 使用 GlobalJobFilter 实例，所以依赖项应该是 Transient 或 Singleton。

```
services.AddHangfire((provider,configuration)=>
{
configuration.UseFilter(provider.GetRequiredService<AutomaticRetryAttribute>());
}
```

默认情况下，Hangfire 使用以下的重试间隔策略：

* 首次失败后立即重试
* 第二次失败后等待 1 分钟
* 第三次失败后等待 2 分钟
* 第四次失败后等待 5 分钟
* 第五次失败后等待 15 分钟

也可以通过设置AutomaticRetry的DelaysInSeconds属性来自定义配置

##### 并行度

后台作业由 Hangfire Server 子系统内运行的专用工作线程池处理。启动后台作业服务器时，它会初始化线程池并启动固定数量的工作线程。
可以通过将值传递给方法来指定它们的数量`UseHangfireServer`。

```
varoptions=newBackgroundJobServerOptions{WorkerCount=Environment.ProcessorCount*5};
app.UseHangfireServer(options);
```

##### 任务队列

通过数据注解指定任务放到的队列

```
[Queue("alpha")]
publicvoidSomeMethod(){}

BackgroundJob.Enqueue(()=>SomeMethod());
```
给server配置多个队列

```
var options = new BackgroundJobServerOptions
{
    Queues = new[] { "alpha", "beta", "default" }
};

app.UseHangfireServer(options);
```

##### 实际使用考虑

**参数方面**：
方法调用（即作业）在后台作业创建过程中被序列化。使用TypeConverter类将参数转换为 JSON 字符串。如果参数有复杂的实体和/或大型对象（包括数组），最好将它们放入数据库中，然后仅将它们的标识传递给后台作业。
不要这样做：

```
public void Method(Entity entity){}
```

考虑这样做：

```
public void Method(int entityId){}
```

**任务设计方面**
可重入性是指方法可以在执行过程中被中断，然后可以安全地再次调用。中断可能由多种原因引起（例如异常、服务器关闭），Hangfire 将尝试多次重试处理。
不要这样做：

```
publicvoidMethod()
{
_emailService.Send("person@example.com","Hello!");
}
```

考虑这样做：

```
publicvoidMethod(int deliveryId)
{
       if(_emailService.IsNotDelivered(deliveryId))
       {
          _emailService.Send("person@example.com","Hello!");
          _emailService.SetDelivered(deliveryId);
       }
}
```

还有就是调用的作业方法都推荐public修饰，虽然 Hangfire 允许调用 `private` 或 `protected` 方法，但这通常需要通过一个 `public` 方法间接调用，所以还是得public。这种方法不太常见，因为它会增加额外的复杂性。

##### 作业过滤器

与 ASP.NET 过滤器一样，可以在方法、类和全局上应用过滤器

```
[LogEverything]
public class EmailService
{
    [LogEverything]
    public static void Send() { }
}
//全局过滤器
GlobalJobFilters.Filters.Add(new LogEverythingAttribute());
```

实现对应数据标注并实现各个阶段方法

```
public class LogEverythingAttribute : JobFilterAttribute,
IClientFilter, IServerFilter, IElectStateFilter, IApplyStateFilter
{
	//在作业创建之前调用
	public void OnCreating(CreatedContext context){}
	//在作业创建之后调用。
	public void OnCreated(CreatedContext context){}
	//在作业执行之前调用。
	public void OnPerforming(CreatedContext context){}
	//在作业执行之后调用。
	public void OnPerformed(CreatedContext context){}
	//在状态转换之前调用，允许你修改或替换即将应用的状态。
	public void OnStateElection(ElectStateContext context){}
	//在状态应用之后调用。
	public void OnStateApplied(ApplyStateContext context, IWriteOnlyTransaction transaction){}
	//在状态被替换之前调用。
	public void OnStateUnapplied(ApplyStateContext context, IWriteOnlyTransaction transaction){}
}
```




