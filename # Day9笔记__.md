# Day9笔记

## AutoFac

### 反射组件

1.通过反射生成的组件通常按类型注册：

```
varbuilder=newContainerBuilder();
builder.RegisterType<ConsoleLogger>();
builder.RegisterType(typeof(ConfigReader));
```

当使用基于反射的组件时， **Autofac 会自动使用从容器中可获取的最多参数的类的构造函数** 。

2.通过指定构造属性注册：

**您可以手动选择要使用的特定构造函数**`<span class="pre">UsingConstructor</span>`，并通过向方法和表示构造函数中的参数类型的类型列表注册您的组件来覆盖自动选择：

```
builder.RegisterType<MyComponent>()
.UsingConstructor(typeof(ILogger),typeof(IConfigReader));
```

请注意，您仍需要在解析时提供必要的参数，否则在尝试解析对象时会出现错误。您可以[在注册时传递参数](https://autofac.readthedocs.io/en/latest/register/parameters.html)，也可以[在解析时传递参数](https://autofac.readthedocs.io/en/latest/resolve/parameters.html)。

3.通过必需属性注册：
从 Autofac 7.0 开始，在基于反射的组件中，所有[必需的属性](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/required)都会自动解析，方式与构造函数参数相同。

组件的所有必需属性*必须*是可解析的服务（或作为[参数](https://autofac.readthedocs.io/en/latest/resolve/parameters.html)提供），否则在尝试解析组件时将引发异常。

例如，考虑具有以下属性的类：

```
publicclassMyComponent
{
publicrequiredILoggerLogger{protectedget;init;}

publicrequiredIConfigReaderConfigReader{protectedget;init;}
}
```

你可以注册并使用这个类，就像它有一个构造函数一样：

```
varbuilder=newContainerBuilder();
builder.RegisterType<MyComponent>();
builder.RegisterType<ConsoleLogger>().As<ILogger>();
builder.RegisterType<ConfigReader>().As<IConfigReader>();
varcontainer=builder.Build();

using(varscope=container.BeginLifetimeScope())
{
// Logger and ConfigReader will be populated.
varcomponent=scope.Resolve<MyComponent>();
}
```

所有基类上的必需属性也会自动设置（如果存在）；这使得必需属性对于深层对象层次结构很有用，因为它允许您避免使用服务集调用基构造函数；Autofac 将为您设置基类属性。

### 实例组件

在某些情况下，您可能希望预先生成一个对象实例并将其添加到容器中以供已注册的组件使用。您可以使用该RegisterInstance方法执行此操作：

```
varoutput=newStringWriter();
builder.RegisterInstance(output).As<TextWriter>();
```

### lambda表达式组件

Autofac 可以接受委托或者 lambda 表达式作为组件创建器：

```
builder.Register(c=>newA(c.Resolve<B>()));
```

