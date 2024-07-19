# Day7笔记

## AutoMapper

AddAutoMapper(this IServiceCollection services, params Type[] profileAssemblyMarkerTypes)方法
调用

```
AddAutoMapperClasses(services, null, profileAssemblyMarkerTypes.Select(t => t.GetTypeInfo().Assembly))
传入服务对象和程序集的集合
```

再之后过滤

```
assembliesToScan = new HashSet<Assembly>(assembliesToScan.Where(a => !a.IsDynamic && a != typeof(Mapper).Assembly));
过滤动态加载或mapper属于的程序集
```

之后再把options => options.AddMaps(assembliesToScan)委托包装成ServiceDescriptor加入到ServiceCollection的ServiceDescriptor集合中

```
options => options.AddMaps(assembliesToScan)
主要把我们提供的程序集集合过滤不是Profile类型的，并且添加到MapperConfigurationExpression的Profile集合中
```

然后把我们的提供的符合条件的类加入到ServiceCollection的ServiceDescriptor集合中

再往后就是查看是否已注入`IMapper` 接口的服务，若没有就将注入生成MapperConfigurationExpression实例和Mapper实例的委托

```
services.AddSingleton<IConfigurationProvider>(sp =>
{
// A mapper configuration is required
var options = sp.GetRequiredService<IOptions<MapperConfigurationExpression>>();
return new MapperConfiguration(options.Value);
});
services.Add(new ServiceDescriptor(typeof(IMapper), sp => new Mapper(sp.GetRequiredService<IConfigurationProvider>(), sp.GetService), serviceLifetime));
```





