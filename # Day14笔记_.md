# Day14笔记

## Mediator.NET

#### 1. **项目概述**

* **项目名称** : Mediator.Net
* **项目描述** : Mediator.Net 是一个用于 .NET 的库，旨在实现中介者模式（Mediator Pattern），使得系统中的对象能够通过中介者进行解耦的交互。
* **主要目标** : 简化对象之间的通信和依赖管理，提高系统的模块化和可维护性。

#### 2. **核心功能**

* **中介者（Mediator）** : 管理和协调消息的发送和处理。
* **消息（Messages）** : 包括命令（Command）、事件（Event）、查询（Query）等，用于定义系统中的操作和通知。
* **处理程序（Handlers）** : 处理特定类型消息的类，负责实际的业务逻辑。
* **异步处理** : 支持异步操作，以提高系统的性能和响应能力。
#### 3. **安装和配置**

* **NuGet 包** : 提供了一个 NuGet 包，可以通过 NuGet 进行安装和引用。
* **安装命令** :Install-Package Mediator.Net
* **配置示例** : 在 ASP.NET Core 项目中，可以通过以下方式配置 Mediator.NET：
  public class Startup
  {
  public void ConfigureServices(IServiceCollection services)
  {
  services.AddMediator(cfg =&gt;
  {
  // 注册处理程序
  cfg.RegisterHandlers(typeof(Startup).Assembly);
  });
  
  // 其他服务注册
  services.AddControllers();
  }
  
  }

#### 4. **用法示例**

* **发送命令** :
  public class CreateOrderCommand : ICommand
  {
  public int CustomerId { get; set; }
  pecimal OrderAmount { get; set; }
  }

public class CreateOrderCommandHandler : ICommandHandler&lt;CreateOrderCommand&gt;
{
public Task Handle(IReceiveContext&lt;CreateOrderCommand&gt; context)
{
// 处理 CreateOrderCommand 的逻辑
return Task.CompletedTask;
}
}

* **发布事件** :

public class OrderCreatedEvent : IEvent
{
public int OrderId { get; set; }
}

public class OrderCreatedEventHandler : IEventHandler&lt;OrderCreatedEvent&gt;
{
public Task Handle(IReceiveContext&lt;OrderCreatedEvent&gt; context)
{
// 处理 OrderCreatedEvent 的逻辑
return Task.CompletedTask;
}
}

* **查询数据** :
  public class GetOrderQuery : IQuery&lt;Order&gt;
  {
  public int OrderId { get; set; }
  }

public class GetOrderQueryHandler : IQueryHandler&lt;GetOrderQuery, Order&gt;
{
public Task&lt;Order&gt; Handle(IReceiveContext&lt;GetOrderQuery&gt; context)
{
// 处理 GetOrderQuery 的逻辑，返回 Order 对象
var order = new Order(); // 获取订单逻辑
return Task.FromResult(order);
}
}





