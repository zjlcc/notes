# Day10 笔记

## AutoFac

### 生命周期

InstancePerLifetimeScope()方法用于在特定的生命周期范围内共享相同的实例。InstancePerMatchingLifetimeScope("lifetime-scope")方法设置匹配标签的生命周期，表示在指定标签的生命周期范围内共享同一个实例，可以确保在同一个生命周期范围内，所有请求都得到相同的实例，但在不同的生命周期范围内，实例是独立的。一个常见的例子是 Web 应用程序中的“每个请求”生命周期。

InstancePerDependency()方法设置瞬时生命周期，表示每个请求都会创建一个新的实例。
InstancePerRequest()在每个 HTTP 请求的生命周期内创建一个实例，并在同一个请求期间共享该实例。

SingleInstance()方法设置单例生命周期，表示整个应用程序生命周期共享同一个实例。
InstancePerOwned方法设置特定组件内的生命周期，表示通过特定组件内共享一个实例。

### 俘获依赖

*当一个本来应该存活很短时间*的组件被一个存活时间*很长的*组件所占据时，就会发生“捕获依赖”。我们可以使用一般规则来避免，消费组件的生命周期应该小于或等于被消费服务的生命周期。基本上，不要让单例承担每个请求一个实例的依赖关系，因为它会被保留太长时间。

### 处理

自动处理：需要组件实现IDisposable，当该组件生命周期结束时，Dispose()将调用组件的处理方法
指定处理：通过使用OnRelease生命周期事件，但是OnRelease方法将覆盖IDispable.dispose()处理。
禁用处理：注册时调用ExternallyOwned()，使用外部所有权注册，容器将不会调用Dispose()或DisposeAsync()

### Life event

OnPreparing在当需要组件新实例时，在调用前引发该事件，可以用于指定创建新实例考虑新的自定义参数。
OnActivating在使用前引发，在这里可以将实例切换为另一个实例或将其包装在代理中、进行属性注入或方法注入、执行其他初始化任务。
OnActivated组件完全构建后，将引发该事件。
OnRelease取代组件的标准清理行为，会覆盖Dispable.dispose()。



