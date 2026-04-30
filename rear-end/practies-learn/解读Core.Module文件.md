# 解读 PractiseForTracyModule.cs

文件路径：`src/PractiseForTracy.Core/PractiseForTracyModule.cs`

这是整个 Core 层的"启动配置中心"，继承自 `Autofac.Module`，在应用启动时统一注册所有组件。

---

## 类定义

```csharp
public class PractiseForTracyModule : Module
```

继承自 Autofac 的 `Module`，这是 Autofac 的模块化注册方式，把所有依赖注入配置集中在这一个类里管理。

```csharp
private readonly Assembly[] _assemblies;
private readonly IConfiguration _configuration;
```

- `_assemblies`：保存需要扫描的程序集列表（用于自动注册 Handler 等）
- `_configuration`：保存配置对象（读取连接字符串等配置）

```csharp
public PractiseForTracyModule(IConfiguration configuration, params Assembly[] assemblies)
{
    _assemblies = assemblies;
    _configuration = configuration;
}
```

构造函数接收配置和一个或多个程序集，`params` 关键字允许传入任意数量的程序集参数。

---

## Load 方法 — 入口

重写 Autofac 的 `Load` 方法，Autofac 启动时会自动调用这个方法完成所有注册。将注册逻辑拆分为 5 个独立方法，职责清晰。

```csharp
protected override void Load(ContainerBuilder builder)
{
    RegisterMediator(builder);   // 注册消息总线
    RegisterDbContext(builder);  // 注册数据库上下文
    RegisterDependency(builder); // 注册业务服务
    RegisterSettings(builder);   // 注册配置项
    RegisterAutoMapper(builder); // 注册对象映射
}
```

---

## 五个注册方法解析

### ① RegisterMediator — 注册 Mediator + 全局管道中间件

```csharp
var mediatorBuilder = new MediatorBuilder();
```

创建 Mediator 的构建器实例。

```csharp
mediatorBuilder.RegisterHandlers(_assemblies);
```

扫描传入的程序集，自动注册所有 Command/Query/Event 的 Handler。

```csharp
mediatorBuilder.ConfigureGlobalReceivePipe(c =>
{
    c.UseUnitOfWork();    // 添加工作单元中间件（管理事务）
    c.UseUnifyResponse(); // 添加统一响应中间件（统一返回格式）
});
```

配置全局管道中间件，所有消息在到达 Handler 之前都会经过这两个中间件（类似 ASP.NET Core 的中间件管道）。

```csharp
builder.RegisterMediator(mediatorBuilder);
```

将配置好的 Mediator 注册到 Autofac 容器中。

---

### ② RegisterDbContext — 注册数据库上下文

```csharp
builder.RegisterType<PractiseForTracyDbContext>()
    .AsSelf()                  // 可以直接注入 PractiseForTracyDbContext 本身
    .As<DbContext>()           // 也可以通过 DbContext 基类注入
    .AsImplementedInterfaces() // 也可以通过它实现的接口注入
    .InstancePerLifetimeScope(); // 每个请求/作用域共享同一个实例（Scoped）
```

```csharp
builder.RegisterType<EfRepository>()
    .As<IRepository>()
    .InstancePerLifetimeScope();
```

注册 EF Core 的 Repository 实现，通过 `IRepository` 接口注入，Scoped 生命周期。

---

### ③ RegisterDependency — 自动扫描注册依赖

```csharp
foreach (var type in typeof(IDependency).Assembly.GetTypes()
    .Where(t => typeof(IDependency).IsAssignableFrom(t) && t.IsClass))
```

扫描 `IDependency` 所在程序集的所有类，筛选出实现了 `IDependency` 接口的具体类（排除接口和抽象类）。

```csharp
case bool _ when typeof(IScopedDependency).IsAssignableFrom(type):
    builder.RegisterType(type).AsImplementedInterfaces().InstancePerLifetimeScope();
    break;
```

如果该类实现了 `IScopedDependency` → 注册为 Scoped（每个请求一个实例）。

```csharp
case bool _ when typeof(ITransientDependency).IsAssignableFrom(type):
    builder.RegisterType(type).AsImplementedInterfaces().SingleInstance(); // ⚠️ Bug！
    break;
```

> **⚠️ 这里有一个 Bug！** 实现了 `ITransientDependency` 的类应该注册为 `.InstancePerDependency()`（瞬态），但代码错误地写成了 `.SingleInstance()`（单例）。

```csharp
default:
    builder.RegisterType(type).AsImplementedInterfaces();
    break;
```

其他情况默认注册，Autofac 默认生命周期为瞬态（`InstancePerDependency`）。

**修复方案** — 将 `.SingleInstance()` 改为 `.InstancePerDependency()`：

```csharp
case bool _ when typeof(ITransientDependency).IsAssignableFrom(type):
    builder.RegisterType(type).AsImplementedInterfaces().InstancePerDependency();
    break;
```

**三种生命周期对比：**

| Autofac 方法                  | 对应概念       | 含义                            |
| ----------------------------- | -------------- | ------------------------------- |
| `.SingleInstance()`           | 单例 Singleton | 整个应用只创建一次，全局共享    |
| `.InstancePerLifetimeScope()` | 作用域 Scoped  | 每个请求/作用域内共享同一个实例 |
| `.InstancePerDependency()`    | 瞬态 Transient | 每次注入都创建一个全新实例      |

`ITransientDependency` 顾名思义是瞬态，每次使用都应该拿到新实例，所以必须用 `.InstancePerDependency()`。原代码写成 `.SingleInstance()` 会导致本该是瞬态的服务变成单例，可能引发状态污染或线程安全问题。

---

### ④ RegisterSettings — 注册配置类

```csharp
var settingTypes = typeof(PractiseForTracyModule).Assembly.GetTypes()
    .Where(t => t.IsClass && typeof(IConfigurationSetting).IsAssignableFrom(t))
    .ToArray();
```

扫描当前程序集，找出所有实现了 `IConfigurationSetting` 接口的配置类（如 `DatabaseSetting`、`JwtSetting` 等）。

```csharp
builder.RegisterTypes(settingTypes).AsSelf().SingleInstance();
```

将这些配置类注册为单例，通过类型本身注入（`AsSelf()`），配置信息全局共享。

---

### ⑤ RegisterAutoMapper — 注册对象映射

```csharp
builder.RegisterAutoMapper(typeof(PractiseForTracyModule).Assembly);
```

扫描当前程序集，自动注册所有继承自 `Profile` 的 AutoMapper 映射配置类，并将 `IMapper` 注册到容器中。

---

## 总结

| 方法                 | 职责                            |
| -------------------- | ------------------------------- |
| `RegisterMediator`   | 注册命令/查询总线及中间件管道   |
| `RegisterDbContext`  | 注册 EF Core 数据库上下文和仓储 |
| `RegisterDependency` | 按接口标记自动扫描注册服务      |
| `RegisterSettings`   | 注册配置类为单例                |
| `RegisterAutoMapper` | 注册对象映射配置                |

整体是一个标准的 Autofac 模块化依赖注入配置，将所有基础设施的注册逻辑集中管理。
