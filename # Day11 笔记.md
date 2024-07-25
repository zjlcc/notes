# Day11 笔记

## EF Core

### DbContext

DbContext类是实体框架的一个组成部分。的实例表示与数据库的会话，可用于查询实体实例并将其保存到数据库。 是工作单元和存储库模式的组合。

`DbContext`在 EF Core 中我们可以执行以下任务：

1. 管理数据库连接
2. 配置模型和关系
3. 查询数据库
4. 保存数据到数据库
5. 配置更改跟踪
6. 缓存
7. 交易管理

### 使用 DbContext

```
public class Student
{
    public int StudentId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }

    public int GradeId { get; set; }
    public Grade Grade { get; set; }
}
       
public class Grade
{
     public Grade()
     {
         Students = new List<Student>();
     }

    public int GradeId { get; set; }
    public string GradeName { get; set; }

    public IList<Student> Students { get; set; }
}
```

创建DbContext

```
public class SchoolContext : DbContext
{       
    //entities
    public DbSet<Student> Students { get; set; }
    public DbSet<Grade> Grades { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("Server=(localdb)\\mssqllocaldb;Database=SchoolDb;Trusted_Connection=True;");
    }
}
```

context 类包含两个`DbSet<TEntity>`属性，分别为`Student`和`Grade`，类型将映射到底层数据库中的`Students`和`Grades`表。在方法中`OnConfiguring()`，`OptionsBuilder`的实例用于指定数据库连接字符串。 

### 创建数据库架构

创建上下文和实体类后，就可以开始使用上下文类与底层数据库进行交互了。但是，在从数据库保存或检索数据之前，我们首先需要确保根据我们的实体和配置创建了数据库模式。

您可以使用两种方法创建数据库和模式：

1. 使用`EnsureCreated()`方法
2. 使用迁移 API

迁移 API 允许您根据实体创建初始数据库模式，然后当您添加/删除/修改实体和配置时，它将同步到数据库的相应模式更改，以便它与您的 EF Core 模型（实体和其他配置）保持兼容。





