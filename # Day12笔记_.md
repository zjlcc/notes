# Day12笔记

## EF Core

### DbContext配置的优先级

通常情况下，如果 `DbContext` 是通过依赖注入（如 Autofac）创建的，那么依赖注入的配置会优先于 `OnConfiguring` 方法的配置。因为在依赖注入中，可以完全控制 `DbContext` 的创建和配置过程。

### 线程并发

- 让每个请求一个 `DbContext`。在 ASP.NET Core 中，通常将 `DbContext` 注册为作用域（Scoped）服务，这意味着每个 HTTP 请求将获得一个新的 `DbContext` 实例。
- 使用 `async` 和 `await` 关键字来处理数据库操作，这样可以避免阻塞线程，提升应用程序的性能。

### 数据库连接池

使用连接池时不能在注入和onconfigure方法中都对上下文配置进行修改

### 数据迁移

数据迁移（Migrations）是 Entity Framework Core 中用来管理数据库架构变更的机制。它允许你在数据库架构和代码模型之间保持同步。通过迁移，你可以轻松地将模型中的更改应用到数据库中，而无需手动编写 SQL 脚本。
以下是使用 EF Core 进行数据迁移的一般步骤：

1. **添加初始迁移** ：

	   dotnet ef migrations add InitialCreate

这将生成一个名为 `InitialCreate` 的迁移文件，里面包含了初始模型的数据库架构。

2. **更新数据库** ：

	```
	dotnet ef database update
	```

这将根据迁移文件中的指令创建或更新数据库架构。

### 反向工程

反向工程（Reverse Engineering）是从现有数据库生成实体模型和 DbContext 类的过程。它在你已有数据库时非常有用，可以通过 EF Core 工具自动生成对应的模型类和上下文类，避免手动创建模型。

```
dotnet ef dbcontext scaffold "Server=localhost;Database=mydb;User=root;Password=mypassword;" Pomelo.EntityFrameworkCore.MySql -o Models
```

使用的是 MySQL 数据库，并且指定连接字符串，-o将生成的的模型类放在Models目录下。

### 分组配置

实现IEntityTypeConfiguration接口，再通过model.applyconfigurationfromassembly完成批量添加实体配置，它代替了将实体配置直接写在 `DbContext` 的 `OnModelCreating` 方法中的做法，使配置代码更加清晰和易于管理。

### 实体跟踪

以下情况会被跟踪：

- 从数据库执行的查询返回通过add
- 通过Add、Attach、Update或类似方法附加到DbContext
- 连接到现有跟踪实体的新实体（其实都指向一个实例）

以下情况不再被跟踪:

- DbContext已释放
- 更改跟踪器（也就是使用非跟踪）
- 显示使用非跟踪查询

实体状态：

- Detached游离状态，未被DbContext跟踪的实体状态
- Added插入状态，调用插入方法，实体是新实体，并且尚未被插入到数据库中，意味着之后调用savechanges时插入
- Unchanged未修改状态，从数据库查询以来未进行修改的实体状态
- Modified已修改状态，对数据库进行修改的实体状态，调用更新方法，意味着之后调用savechanges时更新
- Deleted删除状态，实体存在数据库，调用删除方法，意味着之后调用savechanges时删除

**总结**：
调用savechanges会修改跟踪实体的状态，增加和修改会将实体跟踪状态变为Unchanged状态，而删除操作则		会将实体状态修改为Detached，因为删除了，数据在数据库中不存在了，Dbcontext就无法追踪了。

无跟踪查询对于只读，不更新数据的操作 非常有用，因为它减少了上下文的开销，并且可以提高性能。

若查询出的是对象的一部分实体，那么该实体也不会被追踪，但是若它内部包含的其他实体属性会被追踪。

