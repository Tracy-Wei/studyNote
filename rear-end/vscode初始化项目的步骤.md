========

---

# PractiseForTracy.net 构建指南

## 技术栈

| 类别        | 技术                            |
| ----------- | ------------------------------- |
| 语言 / 框架 | C# 10 / .NET 6.0 / ASP.NET Core |
| DI 容器     | Autofac 8.0                     |
| CQRS 中间件 | Mediator.Net 4.7                |
| ORM         | EF Core 6 + Pomelo MySQL        |
| 对象映射    | AutoMapper 12                   |
| API 文档    | Swashbuckle (Swagger)           |
| 数据库      | MySQL 5.7+                      |

---

## 项目分层职责

| 层     | 项目                       | 职责                                              |
| ------ | -------------------------- | ------------------------------------------------- |
| API 层 | `PractiseForTracy.Api`     | HTTP 入口、路由、Startup 配置                     |
| 业务层 | `PractiseForTracy.Core`    | Handler、Service、Repository、Middleware、DI 注册 |
| 合约层 | `PractiseForTracy.Message` | Command / Request / Event / DTO 定义（无逻辑）    |
| 测试层 | `*.Tests`                  | 单元测试 / 集成测试                               |

---

## Step 1：初始化解决方案结构

```bash
mkdir practisefortracy.net && cd practisefortracy.net
dotnet new sln -n PractiseForTracy

dotnet new webapi    -n PractiseForTracy.Api                -o src/PractiseForTracy.Api
dotnet new classlib  -n PractiseForTracy.Core               -o src/PractiseForTracy.Core
dotnet new classlib  -n PractiseForTracy.Message            -o src/PractiseForTracy.Message
dotnet new xunit     -n PractiseForTracy.UnitTests          -o src/PractiseForTracy.UnitTests
dotnet new xunit     -n PractiseForTracy.IntegrationTests   -o src/PractiseForTracy.IntegrationTests

# 注册到解决方案
dotnet sln add src/PractiseForTracy.Api/PractiseForTracy.Api.csproj
dotnet sln add src/PractiseForTracy.Core/PractiseForTracy.Core.csproj
dotnet sln add src/PractiseForTracy.Message/PractiseForTracy.Message.csproj

# 添加项目引用
dotnet add src/PractiseForTracy.Api/PractiseForTracy.Api.csproj \
    reference src/PractiseForTracy.Core/PractiseForTracy.Core.csproj \
              src/PractiseForTracy.Message/PractiseForTracy.Message.csproj

dotnet add src/PractiseForTracy.Core/PractiseForTracy.Core.csproj \
    reference src/PractiseForTracy.Message/PractiseForTracy.Message.csproj
```

---

## Step 2：安装 NuGet 包

```bash
# Api 层
cd src/PractiseForTracy.Api
dotnet add package Autofac.Extensions.DependencyInjection --version 8.0.0
dotnet add package Swashbuckle.AspNetCore --version 6.5.0

# Core 层
cd ../PractiseForTracy.Core
dotnet add package Autofac --version 8.0.0
dotnet add package Mediator.Net --version 4.7.0
dotnet add package Mediator.Net.Autofac --version 4.7.0
dotnet add package AutoMapper --version 12.0.1
dotnet add package AutoMapper.Contrib.Autofac.DependencyInjection --version 7.2.0-alpha.0
dotnet add package Pomelo.EntityFrameworkCore.MySql --version 7.0.0
dotnet add package EFCore.BulkExtensions --version 8.0.0-preview.7
dotnet add package Microsoft.EntityFrameworkCore --version 7.0.0
```

---

## Step 3：Message 层（合约定义）

```
PractiseForTracy.Message/
├── Commands/Product/AddProductCommand.cs
├── Requests/Product/GetProductByIdRequest.cs
├── Event/Product/ProductAddEvent.cs
├── DTO/AddProductDto.cs
└── Responses/PractiseForTracyResponse.cs
```

```csharp
// 统一响应体
public class PractiseForTracyResponse
{
    public int Code { get; set; }
    public string Msg { get; set; }
}

// Command（写操作）
public class AddProductCommand : ICommand
{
    public string Name { get; set; }
    public double Money { get; set; }
}

// Request（读操作）
public class GetProductByIdRequest : IRequest
{
    public int Id { get; set; }
}

// Domain Event
public class ProductAddEvent : IEvent
{
    public AddProductDto Product { get; set; }
}
```

---

## Step 4：Core 层

### 4.1 Domain 实体

```csharp
public interface IEntity { }

public class Product : IEntity
{
    public int Id { get; set; }
    public string Name { get; set; }
    public double Money { get; set; }
    public DateTime RecordCreatedOn { get; set; }
}
```

### 4.2 Repository & UnitOfWork

```csharp
public interface IRepository
{
    Task<T> GetAsync<T>(int id) where T : class, IEntity;
    Task InsertAsync<T>(T entity) where T : class, IEntity;
    Task UpdateAsync<T>(T entity) where T : class, IEntity;
    Task DeleteAsync<T>(T entity) where T : class, IEntity;
}

public interface IUnitOfWork
{
    bool ShouldSaveChanges { get; set; }
    Task SaveChangesAsync(CancellationToken token = default);
    DbSet<T> Set<T>() where T : class;
}
```

`PractiseForTracyDbContext` 继承 `DbContext` 并实现 `IUnitOfWork`，`OnModelCreating` 中自动扫描所有 `IEntity` 类型完成注册。

### 4.3 DI 标记接口

```csharp
public interface IDependency { }
public interface IScopedDependency : IDependency { }
public interface ITransientDependency : IDependency { }
```

### 4.4 Service & DataProvider

| 类                    | 职责                                 |
| --------------------- | ------------------------------------ |
| `ProductDataProvider` | 直接操作 `IRepository`，处理映射逻辑 |
| `ProductService`      | 封装 DataProvider，供 Handler 调用   |

### 4.5 CQRS Handlers

```csharp
// Command Handler
public class AddProductCommandHandler
    : ICommandHandler<AddProductCommand, AddProductResponse> { ... }

// Request Handler
public class ProductRequestHandler
    : IRequestHandler<GetProductByIdRequest, GetProductByIdResponse> { ... }

// Event Handler
public class ProductAddEventHandler : IEventHandler<ProductAddEvent> { ... }
```

### 4.6 Pipeline Middleware

| Middleware                | 时机         | 作用                                     |
| ------------------------- | ------------ | ---------------------------------------- |
| `UnitOfWorkMiddleWare`    | After Handle | 若 `ShouldSaveChanges = true` 则提交事务 |
| `UnifyResponseMiddleware` | After Handle | 注入 `Code=200, Msg="ok"`                |

### 4.7 Autofac Module

`PractiseForTracyModule` 统一注册以下内容：

- Mediator（含 Handler 扫描 + Pipeline 配置）
- DbContext / IUnitOfWork（`InstancePerLifetimeScope`）
- IRepository（`InstancePerLifetimeScope`）
- 所有 `IScopedDependency` 实现（程序集自动扫描）
- 所有 `IConfigurationSetting` 实现（读取 `appsettings.json` 绑定）
- AutoMapper Profiles

---

## Step 5：Api 层

### Program.cs

```csharp
var host = Host.CreateDefaultBuilder(args)
    .UseServiceProviderFactory(new AutofacServiceProviderFactory())
    .ConfigureWebHostDefaults(webBuilder => webBuilder.UseStartup<Startup>())
    .Build();

host.Run();
```

### Startup.cs 关键方法

```csharp
// Autofac 专用入口
public void ConfigureContainer(ContainerBuilder builder)
{
    builder.RegisterModule(new PractiseForTracyModule(_configuration));
}
```

### appsettings.json

```json
{
  "ConnectionStrings": {
    "Default": "server=localhost;database=study;user=root;password=123456;port=3306;"
  }
}
```

### API 端点

| 方法 | 路径                | 功能     |
| ---- | ------------------- | -------- |
| GET  | `/api/home/hello`   | 健康检查 |
| GET  | `/api/product/{id}` | 查询商品 |
| POST | `/api/product/add`  | 新增商品 |

---

## Step 6：数据库迁移

```bash
cd src/PractiseForTracy.Core

dotnet tool install --global dotnet-ef

dotnet ef migrations add InitialCreate \
    --startup-project ../PractiseForTracy.Api \
    --output-dir Data/Migrations

dotnet ef database update \
    --startup-project ../PractiseForTracy.Api
```

---

## 请求流转图

```
HTTP Request
    │
    ▼
ProductController
    │  IMediator.SendAsync / RequestAsync
    ▼
Mediator Pipeline
    ├─► [前置] UnitOfWorkMiddleware.ExecuteBeforeHandle
    │
    ├─► CommandHandler / RequestHandler
    │       ├─► ProductService → ProductDataProvider → IRepository → DbContext (MySQL)
    │       └─► context.PublishAsync(ProductAddEvent) → ProductAddEventHandler
    │
    ├─► [后置] UnitOfWorkMiddleware：SaveChangesAsync（若 ShouldSaveChanges）
    └─► [后置] UnifyResponseMiddleware：注入 Code=200, Msg="ok"
    │
    ▼
HTTP Response (JSON)
```
