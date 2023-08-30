# Entity Framework Core

是一个用于 .NET 应用程序的对象关系映射（ORM）框架，它允许开发者使用面向对象的方式来操作数据库，而不需要直接编写 SQL 查询。

## Model Mapping（模型映射）

Model Mapping 是指将关系型数据库中的表结构映射到应用程序中的实体类（Model）的过程。这样做允许开发者使用面向对象的方式来操作数据库，而不必直接编写 SQL 查询语句。Model Mapping 在 Entity Framework Core 中是通过实体类、DbContext 类以及配置方式（Fluent API 或数据注解）来实现的。

在 Entity Framework Core 中，开发者需要定义一个继承自 DbContext 的类，这个类表示了数据库上下文，其中包含了用于与数据库交互的 DbSet 属性。每个 DbSet 属性都代表了数据库中的一个表，而实体类则代表了表中的行。

### DbContext 类：

DbContext 是一个重要的类，代表了数据库上下文。它包含了用于操作数据库的 DbSet 属性，每个 DbSet 属性对应一个数据库表。DbContext 类负责跟踪实体类的变化，并将这些变化同步到数据库中。

实例 DbContext 设计用于单个 工作单元。这意味着 DbContext 实例的生命周期通常很短。
典型工作单元涉及：

- 创建 DbContext 实例
- 通过上下文跟踪实体实例。实体被跟踪
  - 从查询返回
  - 添加或附加到上下文
- 根据需要对跟踪实体进行更改以实施业务规则
- 调用 SaveChanges 或 SaveChangesAsync 。EF Core 检测所做的更改并将其写入数据库。
- 实例 DbContext 已处置

依赖注入中的 DbContext：

```javascript
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    services.AddDbContext<ApplicationDbContext>(
        options => options.UseSqlServer("name=ConnectionStrings:DefaultConnection"));
}
```

配置 DbContextOptions
DbContext 必须具有的实例 DbContextOptions 才能执行任何工作。 DbContextOptions 实例传送配置信息，如：
数据库提供程序，若要使用，通常选择通过调用方法，如 UseSqlServer 或 UseSqlite

```javascript
optionsBuilder
  .UseSqlServer(connectionString, (providerOptions) =>
    providerOptions.CommandTimeout(60)
  )
  .UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
```

OnConfiguring

```javascript
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlite("Data Source=blog.db");
    }
}
```

### 实体类（Entity Class）：

实体类是用于表示数据库表中的数据行的类。每个实体类的属性通常映射到数据库表的列，而实体类的实例则对应于表中的数据记录。实体类定义了数据的结构和行为，同时也是开发者与数据库之间交互的主要接口。（一般统一放 Domain 里面）

示例：

```javascript
 public class Movie
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Director { get; set; }
        public int YearReleased { get; set; }

        // 构造函数
        public Movie(string title, string director, int yearReleased)
        {
            Title = title;
            Director = director;
            YearReleased = yearReleased;
        }
    }
```

### Fluent API 和数据注解：

Entity Framework Core 提供了两种方式来配置实体类和数据库表之间的映射关系：Fluent API 和数据注解。

- Fluent API：Fluent API 是一种通过代码来配置映射关系的方式。开发者可以在 DbContext 类的 OnModelCreating 方法中使用 Fluent API 方法来定义实体类的属性与数据库表的映射规则，例如设置主键、外键、索引等。
  示例：

  ```javascript
  using Microsoft.EntityFrameworkCore;

  namespace EFModeling.EntityProperties.FluentAPI.Required;

  internal class MyContext : DbContext
  {
      public DbSet<Blog> Blogs { get; set; }

      #region Required
      protected override void OnModelCreating(ModelBuilder modelBuilder)
      {
          modelBuilder.Entity<Blog>()
              .Property(b => b.Url)
              .IsRequired();
      }
      #endregion
  }

  public class Blog
  {
      public int BlogId { get; set; }
      public string Url { get; set; }
  }
  ```

- 数据注解：数据注解是通过在实体类的属性上添加特定的属性来定义映射关系的方式。通过在属性上添加 [Key]、[Required]、[Column] 等数据注解，可以指定属性在数据库中的映射方式，例如指定主键、设置列名等。
  示例：

  ```javascript
  using System.ComponentModel.DataAnnotations;

  public class Product
  {
      [Key] // 定义主键
      public int ProductId { get; set; }

      [Required] // 定义非空约束
      [StringLength(100)] // 定义最大长度
      public string Name { get; set; }

      [Column("first_name")] // 数据注解，用于将该属性映射到数据库表中的一个特定列
      public string FirstName { get; set; }
  }
  ```

**modelBuilder.Entity<T>()** 是用于配置实体类映射的方法。这个方法允许你在 OnModelCreating 方法中使用 Fluent API 来定义实体类与数据库表之间的映射关系、主键、外键以及其他高级配置。

配置主键：

```javascript
modelBuilder.Entity<ClassA>().HasKey(t => t.ID);    //配置ClassA的ID属性为主键
```

设置数据非数据库生成：

```javascript
modelBuilder.Entity<ClassA>().Property(t => t.Id).HasDatabaseGeneratedOption(DatabaseGeneratedOption.None);    //ClassA的Id属性不用数据库控制生成
```

设置字段为必需：

```javascript
modelBuilder.Entity<ClassA>().Property(t =>t.Id).IsRequired();   //设置ClassA类的Id属性为必需
```

属性不映射到数据库：

```javascript
modelBuilder.Entity<ClassA>().Ignore(t => t.A);    //调过ClassA类的A属性，让之不映射到数据库中
```

将属性映射到数据库中特定列名：

```javascript
modelBuilder.Entity<ClassA>()
    .Property(t => t.A)
    .HasColumnName("A_a");   //将类ClassA的属性A映射到数据库中对应列名A_a
```

实体类映射到的数据库表名：

```javascript
modelBuilder.Entity<ClassA>()
    .ToTable("TableA");
```

配置一对多关系，将一个实体类与多个目标实体关联：

```javascript
modelBuilder.Entity<ClassA>()
    .HasOne(a => a.OneToOneItem)
    .WithOne(o => o.ClassA);
```

配置索引，提高某些查询的性能：

```javascript
modelBuilder.Entity<ClassA>()
    .HasIndex(a => a.Name);
```

配置复合对象（Owned Entity），其中 TOwnedEntity 是复合对象的类型：

```javascript
modelBuilder.Entity<ClassA>()
    .OwnsOne(a => a.Address);
```

配置属性的默认值：

```javascript
modelBuilder.Entity<ClassA>()
    .Property(a => a.Status)
    .HasDefaultValue(Status.Active);
```

## Repository（仓储）

Repository 是一种设计模式，它将数据访问操作封装到一个单独的类中，从而提供了一种更抽象和可维护的方式来管理数据访问逻辑。在 Entity Framework Core 中，Repository 模式通常用于封装对数据库的操作，以及将数据操作从业务逻辑中解耦。

Repository 可以被实现为一组方法，用于执行常见的数据操作，如添加、查询、更新和删除数据（CRUD 操作）。这有助于保持代码的结构清晰，并且可以方便地在未来更改数据访问逻辑。

下面是一个简单的示例，展示如何实现一个基本的 UserRepository 仓储：

```javascript
public interface IUserRepository
{
    User GetById(int id);
    void Add(User user);
    void Update(User user);
    void Delete(int id);
}

public class UserRepository : IUserRepository
{
    private readonly MyDbContext _dbContext;

    public UserRepository(MyDbContext dbContext)
    {
      // 通过在 PersonDataProvider 类中注入 MyDbContext 类的实例，我们可以在该类中使用 _dbContext 来访问数据库
        _dbContext = dbContext;
    }

    public User GetById(int id)
    {
        return _dbContext.Users.Find(id);
    }

    public void Add(User user)
    {
        _dbContext.Users.Add(user);
        _dbContext.SaveChanges();
    }

    public void Update(User user)
    {
        _dbContext.Users.Update(user);
        _dbContext.SaveChanges();
    }

    public void Delete(int id)
    {
        var user = _dbContext.Users.Find(id);
        if (user != null)
        {
            _dbContext.Users.Remove(user);
            _dbContext.SaveChanges();
        }
    }
}
```

DbContext 的全部方法可查看：[link1]

[link1]: https://learn.microsoft.com/zh-tw/dotnet/api/microsoft.entityframeworkcore.dbcontext.add?view=efcore-7.0
