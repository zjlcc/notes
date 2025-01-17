# Day8笔记

## LINQ

### 查询变量的区别

IEnumerable和IQueryable序列之间的区别决定了查询在运行时如何执行。
对于`IEnumerable<T>`，返回的可枚举对象捕获传递给该方法的参数。枚举该对象时，将采用查询运算符的逻辑并返回查询结果。
对于`IQueryable<T>`，查询被转换为表达式树。当数据源可以优化查询时，表达式树可以转换为本机查询。诸如Entity Framework之类的库将 LINQ 查询转换为在数据库中执行的本机 SQL 查询。

### 查询操作组成

所有 LINQ 查询操作都由以下三个不同的操作组成：

1. 获取数据源。数据源必须实现 `IEnumerable<T>` 接口或者 `IQueryable<T>` 接口。
2. 创建查询。使用查询语法或方法链式调用
3. 执行查询。分为即时执行和延迟执行，返回标量结果的所有标准查询运算符都立即执行。 `Count`、`Max`、`Average` 和 `First` 就属于即时执行。结果被检索一次，然后存储以供将来使用。延迟执行指的是不在代码中声明查询的位置执行运算。 仅当对查询变量进行枚举时才执行运算，例如通过使用 `foreach` 语句执行。

### 查询运算符

* 使用关键字过滤数据`where`。
* 使用`orderby`和可选关键字对数据进行排序`descending`。
* 使用`group`和可选关键字对数据进行分组`into`。
* 使用关键字连接数据`join`。
* 使用关键字的项目数据`select`。

### 查询变量及查询语句

- 查询变量是存储查询而不是查询结果的任何变量。 更具体地说，查询变量始终是可枚举类型。查询变量显式类型以便显示查询变量与 select 子句之间的类型关系。 但是，还可以使用 var关键字指示编译器在编译时推断查询变量（或任何其他局部变量）的类型。
- 查询表达式必须以from子句开头，且必须以 select或group子句结尾。 在第一个from子句与最后一个`select`或  group子句之间，可以包含以下这些可选子句中的一个或多个：where、orderby、join、let，甚至是其他 from子句。 还可以使用 into关键字，使`join`或`group` 子句的结果可以充当相同查询表达式中的更多查询子句的源。

## AutoMapper

如何优化的查询效率：

- 投影查询
  AutoMapper 可以帮助将 ORM 返回的复杂对象（例如包含大量导航属性的实体对象）映射成简单的DTO（数据传输对象），只包含需要的字段。这样做可以减少数据库查询时返回的数据量，从而提高查询效率。
- 延迟加载
  ORM 框架通常支持延迟加载，但是如果不加以控制可能会导致 N+1 查询问题（即每次访问导航属性都会触发额外的数据库查询）。AutoMapper 可以显式地定义加载哪些导航属性，从而避免不必要的数据库访问。
- 缓存映射配置
  AutoMapper 可以配置静态缓存以加快映射速度，避免重复的映射配置解析过程，特别是在需要频繁进行映射操作时。







