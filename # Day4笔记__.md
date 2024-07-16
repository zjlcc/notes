# Day4笔记

## ASP.NET Core

### 启动流程

- `Program` 类的`Main`方法为入口，先添加配置文件、环境变量，再为要创建的主机设置一些默认的选项，例如应用程序的基本设置和默认的配置提供者，再通过`ConfigureWebHostDefaults(webBuilder => {...})`  委托完成` WebHostBuilder`的配置，并指定应用程序启动时要使用的`Startup`类。
- `ASP.NET Core`运行时会实例化指定的 `Startup`类，`ASP.NET Core` 就会调用其 `ConfigureServices` 方法。这个方法接受一个`IServiceCollection`对象作为参数，用于配置应用程序的服务依赖注入容器，再会调用`Configure`方法，配置应用程序的 HTTP 请求处理管道，可以添加和配置各种中间件，来处理请求和响应，以及设置路由、静态文件服务、异常处理、身份验证、授权等，通过 `ConfigureServices` 方法注册的服务和 `Configure` 方法中添加的中间件，可以集成第三方库和框架。
- 当 `Configure` 方法完成后，ASP.NET Core 应用程序就会根据 `Startup` 类中的配置开始运行。这意味着，所有的服务注册、中间件配置都已经完成，应用程序可以接受和处理来自客户端的请求。

### 配置文件

- 加载顺序为：appsettings.json，appsettings.{Environment}.json，User secrets，环境变量，命令行参数
- 可以通过注入IConfiguration来访问对应的变量值

### 依赖关系注入

- 注入方式：构造函数注入、属性注入、方法注入、通过IServiceProvider直接获取。最好采用构造函数注入。
- 注册接口与实现关系：使用services.AddTransient,services.AddScoped或services.AddSingleton注册，对应的生命周期分别是瞬时生命周期、作用域生命周期、单例生命周期。瞬时生命周期服务在它们每次请求时被创建。作用域生命周期服务在每次请求时被创建一次。单例生命周期服务在它们第一次被请求时创建，并且每个后续请求将使用相同的实例。

### 中间件

- 中间件可同时被访问，可处理请求和响应，可向下一个中间件传递，也可发生中断。
- 默认静态文件夹为wwwroot，要使用静态文件需要调用UseStaticFiles()启用对应中间件，使用默认文件则调用UseDefaultFiles()。


