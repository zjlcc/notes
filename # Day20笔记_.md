# Day20笔记

## HangFire

#### 配置

ConfigureServices方法

```
// 配置 HangFire 服务
services.AddHangfire(config=> config.UseMemoryStorage());
// 添加 Hangfire 服务
services.AddHangfireServer();
```

Configure方法

```
// 配置 Hangfire 仪表板
app.UseHangfireDashboard();
```

#### Fire-and-Forget Jobs

执行专案或开启站台后会立刻执行该排程任务且只会执行一次，执行完会马上抛弃该任务。

```
BackgroundJob.Enqueue(() => TestFunction());
```

该`Enqueue`方法不会立即调用目标方法，而是运行以下步骤：

1. 序列化方法信息及其所有参数。
2. 根据序列化的信息创建一个新的后台作业。
3. 将后台作业保存到持久存储中。
4. 将后台作业排入其队列。

执行完这些步骤后，该`BackgroundJob.Enqueue`方法立即返回给调用者。另一个 Hangfire 组件，称为Hangfire Server，会检查持久存储中是否有排队的后台作业，并以可靠的方式执行它们。排队的作业由专用的工作线程池处理。每个工作线程都会调用以下过程：

1. 获取下一个作业并对其他工作人员隐藏。
2. 执行该作业及其所有扩展过滤器。
3. 从队列中移除该作业。

所以，只有处理成功后，该作业才会被移除。即使在执行过程中终止了某个进程，Hangfire 也会执行补偿逻辑来保证每个作业的处理。每个存储对于每个步骤都有自己的实现和补偿逻辑机制：

* **SQL Server**实现使用常规 SQL 事务，因此如果进程终止，后台作业 ID 会立即放回队列。
* **MSMQ**实现使用事务队列，因此无需定期检查。作业入队后几乎立即被提取。
* **Redis**实现使用阻塞`<span class="pre">BRPOPLPUSH</span>`命令，因此作业会立即获取，就像 MSMQ 一样。但如果进程终止，则只有在超时（默认为 30 分钟）后才会重新排队。

#### Delayed Jobs

该方法可以设定执行时间，但是也只会执行一次。

```
BackgroundJob.Schedule(() =>TestFunction(),TimeSpan.FromSeconds(3));
```

#### Recurring Jobs

经常性任务，重复执行以及定时执行，也是目前我最常使用在专案上的方法。

```
RecurringJob.AddOrUpdate(() => TestFunction() , “0 0 7 1 * ?”);
```



