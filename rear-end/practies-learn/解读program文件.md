# 解读 Program 文件

解读 `ConfigurationBuilder` 和 `Host` 的使用。（⚠️注：自用留存底部引用的笔记作为副本来学习）

---

## 一. 代码解读

这段代码主要用 `ConfigurationBuilder` 构建一个实例对象 `configuration`：

```csharp
var configuration = new ConfigurationBuilder()
    .AddJsonFile("appsettings.json")
    .AddEnvironmentVariables()
    .Build();
```

- `new ConfigurationBuilder()` — 创建该类型实例对象
- `AddJsonFile("appsettings.json")` — 给 `configuration` 配置一个名为 `appsettings.json` 的配置文件
- `AddEnvironmentVariables()` — 向这个实例对象添加环境变量作为配置源
- `Build()` — 执行以上配置构建操作

---

这段代码的主要作用是创建应用程序的主机环境，配置 Web 主机的默认设置，包括加载通用配置、配置日志记录器、配置依赖注入容器，以及指定 `Startup` 类作为启动类：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

- 该方法最终返回一个 `IHostBuilder` 实例，即应用程序的主机环境构建器
- `Host` 类 — 用统一的方式启动、停止和管理应用程序的生命周期
- `CreateDefaultBuilder(args)` — 自动加载通用基本配置和依赖注入，返回一个默认实例对象
- `ConfigureWebHostDefaults` — 对 Web 主机的默认配置进行自定义，改用 `Startup` 作为启动类，并在启动时自动运行其配置方法
- `UseStartup<Startup>()` — 指定启动类，并在启动应用程序时自动运行启动类的配置方法

---

## 二. 知识点解读

### ConfigurationBuilder

`ConfigurationBuilder` 主要用于构建和配置不同类型的对象、服务或组件的实例。它提供了一种流畅的 API，可以通过链式调用方法来设置各种属性和选项，主要用于：

- 构建对象实例
- 配置选项
- 依赖注入

### AddJsonFile

`AddJsonFile("appsettings.json")` — 向 `ConfigurationBuilder` 添加一个 JSON 配置文件作为配置源。

### appsettings.json

`appsettings.json` 通常包含应用程序的配置信息，例如数据库连接字符串、日志级别等。

### AddEnvironmentVariables

`AddEnvironmentVariables()` 是 `ConfigurationBuilder` 类中的一个方法，用于将环境变量添加为配置源。

### Build()

执行配置构建操作，将所有添加的配置源整合成一个，最终返回一个对应数据类型的配置对象实例。

### Host

`Host` 类用于创建和管理应用程序的主机环境，提供了一种统一的方式来启动、停止和管理应用程序的生命周期，主要负责：

- 应用程序的启动和初始化
- 生命周期管理
- 应用程序配置
- 依赖注入容器
- 日志记录和诊断

| 方法                       | 说明                                 |
| -------------------------- | ------------------------------------ |
| `CreateDefaultBuilder`     | 创建基本配置（通用配置、依赖注入等） |
| `ConfigureWebHostDefaults` | 对 Web 主机的基本配置进行自定义      |

---

> 笔记来源：[CodeInterpreProgram.md](https://github.com/1sanqian/Learn-Project-Structure/blob/ride-doc-structuer-note/main/CodeInterpreProgram.md)
