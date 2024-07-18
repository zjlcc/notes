# Day6笔记

## AutoMapper

### 配置

MapperConfiguration通过构造函数创建实例并初始化配置：

```
varconfig=newMapperConfiguration(cfg=>{
    cfg.CreateMap<Foo,Bar>();
});
```

也可以通过创建继承自类Profile并将配置放入构造函数中初始化配置：

```
public class OrganizationProfile : Profile
{
	public OrganizationProfile()
	{
		CreateMap<Foo, FooDto>();
	}
}
```

也可以配置源和目标命名约定，字段过滤，可见性，使属性按指定规则相互映射

### 依赖注入

通过IServiceCollection拓展方法让 AutoMapper 知道这些配置文件在哪

```
services.AddAutoMapper(typeof(ProfileTypeFromAssembly1), typeof(ProfileTypeFromAssembly2) /*, ...*/);
```

### 投影

当源值投影到与源结构不完全匹配的目标时，必须指定自定义成员映射定义

```
var configuration = new MapperConfiguration(cfg =>
  cfg.CreateMap<CalendarEvent, CalendarEventForm>()
	.ForMember(dest => dest.EventDate, opt => opt.MapFrom(src => src.Date.Date))
	.ForMember(dest => dest.EventHour, opt => opt.MapFrom(src => src.Date.Hour))
	.ForMember(dest => dest.EventMinute, opt => opt.MapFrom(src => src.Date.Minute)));
```

通过Map方法完成投影

```
CalendarEventForm form = mapper.Map<CalendarEvent, CalendarEventForm>(calendarEvent);
```

### 嵌套映射

还可以使用另一个类型映射，其中源成员类型和目标成员类型也在映射配置中配置。这不仅使我们可以将源类型扁平化，还可以创建复杂的目标类型。

### 列表和数组

AutoMapper 仅需要配置元素类型，而不需要配置可能使用的任何数组或列表类型。

```
var configuration = new MapperConfiguration(cfg => cfg.CreateMap<Source, Destination>());

var sources = new[]
	{
		new Source { Value = 5 },
		new Source { Value = 6 },
		new Source { Value = 7 }
	};

IEnumerable<Destination> ienumerableDest = mapper.Map<Source[], IEnumerable<Destination>>(sources);
ICollection<Destination> icollectionDest = mapper.Map<Source[], ICollection<Destination>>(sources);
IList<Destination> ilistDest = mapper.Map<Source[], IList<Destination>>(sources);
List<Destination> listDest = mapper.Map<Source[], List<Destination>>(sources);
Destination[] arrayDest = mapper.Map<Source[], Destination[]>(sources);
```

还可以设置若源值为null，AutoMapper 将把目标字段映射到空集合

```
varconfiguration=newMapperConfiguration(cfg=>{
	cfg.AllowNullCollections=true;
	cfg.CreateMap<Source,Destination>();
});
```

### 压平

复杂的对象模型扁平化为更简单的模型，在 AutoMapper 中配置源/目标类型对时，配置器会尝试将源类型上的属性和方法与目标类型上的属性进行匹配。如果目标类型上的任何属性在源类型上不存在以“Get”为前缀的属性、方法或方法，AutoMapper 会将目标成员名称拆分为单个单词（按照 PascalCase 约定）。
还可以使用 IncludeMembers。当已经有从子类型到目标类型的映射时，可以将子对象的成员映射到目标对象。

### 反向映射和还原

通过调用ReverseMap，AutoMapper 创建包含取消展平的反向映射配置：

```
var configuration = new MapperConfiguration(cfg => {
  cfg.CreateMap<Order, OrderDto>()
     .ReverseMap();
});
```

```
var customer = new Customer {
  Name = "Bob"
};

var order = new Order {
  Customer = customer,
  Total = 15.8m
};

var orderDto = mapper.Map<Order, OrderDto>(order);

orderDto.CustomerName = "Joe";

mapper.Map(orderDto, order);

order.Customer.Name.ShouldEqual("Joe");
```

### 可查询拓展

当将 NHibernate 或 Entity Framework 等 ORM 与 AutoMapper 的标准`mapper.Map`功能一起使用时，当 AutoMapper 尝试将结果映射到目标类型时，ORM 将查询图中所有对象的所有字段。如果对应ORM 公开了IQueryable，您可以使用 AutoMapper 的 QueryableExtensions 辅助方法来缓和。

```
varconfiguration=newMapperConfiguration(cfg=>
	cfg.CreateProjection<OrderLine,OrderLineDTO>()
	.ForMember(dto=>dto.Item,conf=>conf.MapFrom(ol=>ol.Item.Name)));

publicList<OrderLineDTO>GetLinesForOrder(intorderId)
{
	using(varcontext=neworderEntities())
	{
		return context.OrderLines.Where(ol=>ol.OrderId==orderId)
		     .ProjectTo<OrderLineDTO>(configuration).ToList();
}
}
```




