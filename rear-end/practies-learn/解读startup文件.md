# 解读 Startup 文件

解读 `IServiceCollection`、`IApplicationBuilder` 和 `IWebHostEnvironment` 三个接口的使用。

## 一. 代码解读

这段代码主要将 MVC 和 Web API 控制器相关的服务添加到服务容器中，使得应用程序可以使用 MVC 架构和 Web API 控制器来处理请求和生成响应。

MVC 是一种架构模式，把应用拆分成三层，让代码职责清晰：

```
用户请求
    │
    ▼
Controller（控制器）  ← 接收请求、处理逻辑、调用服务
    │
    ▼
Model（模型）        ← 数据、业务逻辑
    │
    ▼
View（视图）         ← 渲染 HTML 页面返回给用户
```

**三层各自干什么：**

| 层         | 职责                        | 举例                 |
| ---------- | --------------------------- | -------------------- |
| Model      | 数据结构 + 业务规则         | `User`、`Order` 类   |
| View       | 展示界面（HTML 模板）       | `.cshtml` Razor 页面 |
| Controller | 接请求 → 调 Model → 选 View | `UserController`     |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();              // 全功能：Controller + View + API
    // services.AddControllersWithViews(); // Controller + View（无 Razor Pages）
    // services.AddControllers();      // 纯 API，没有 View
    // services.AddRazorPages();       // 只有 Razor Pages
}
```

---

这段代码的主要作用是在开发环境下启用异常页面，并配置路由和终端点处理，使控制器能够处理请求和生成响应。

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseRouting();
    app.UseEndpoints(endpoints => { endpoints.MapControllers(); });
}
```

- `env.IsDevelopment()` — 若当前应用程序环境处于开发环境则返回 `true`
- `app.UseDeveloperExceptionPage()` — 在开发环境下显示详细的异常信息页面，能够捕获并显示应用程序中发生的异常
- `app.UseRouting()` — 用于启用路由功能，根据请求的 URL 路径来决定将请求发送给哪个中间件进行处理
- `app.UseEndpoints()` — 用于配置终端点路由
- `endpoints.MapControllers()` — 将 HTTP 请求映射到控制器的动作方法，用于处理请求并生成响应

---

## 二. 知识点解读

### ConfigureServices

`ConfigureServices` 方法是一个在启动时由框架自动调用的方法，用于配置应用程序的服务容器。`IServiceCollection` 是一个用于注册和管理应用程序服务的集合，主要用于：

- 构建对象实例
- 配置选项
- 依赖注入

### services.AddMvc()

`services.AddMvc()` 是将 MVC（Model-View-Controller）服务添加到服务容器中的方法。

- 它会注册必要的服务和中间件，以支持 MVC 架构模式
- 添加 MVC 服务后，应用程序可以使用控制器、视图和模型来处理请求和生成响应

### services.AddControllers()

`services.AddControllers()` 是将 Web API 控制器服务添加到服务容器中的方法。

- 它会注册 Web API 控制器所需的服务和中间件
- 添加 Web API 控制器服务后，应用程序可以使用控制器来处理 Web API 请求
- 注：AddMvc是全家桶，使用了AddMvc就可以不用AddControllers了，二选一即可。

### IServiceCollection

`IServiceCollection` 提供了一组方法，用于向服务容器中注册和配置应用程序的服务：

| 方法                                        | 生命周期          | 说明                           |
| ------------------------------------------- | ----------------- | ------------------------------ |
| `AddTransient<TService, TImplementation>()` | 瞬时（Transient） | 每次请求都创建一个新的实例     |
| `AddScoped<TService, TImplementation>()`    | 作用域（Scoped）  | 每个请求期间都会使用同一个实例 |
| `AddSingleton<TService, TImplementation>()` | 单例（Singleton） | 整个应用程序只会创建一个实例   |
| `AddTransient<T>()`                         | 瞬时              | 使用自身作为服务的实现类型     |
| `AddScoped<T>()`                            | 作用域            | 使用自身作为服务的实现类型     |
| `AddSingleton<T>()`                         | 单例              | 使用自身作为服务的实现类型     |

> `AddTransient()`、`AddScoped()` 和 `AddSingleton()` 方法还有其他重载，可以根据需要传递不同的参数来注册服务。

### IApplicationBuilder

`IApplicationBuilder` 提供了一组方法，用于向请求处理管道中添加中间件、配置中间件的顺序以及其他与请求处理相关的操作：

- `Use()` — 用于添加自定义的中间件到请求处理管道中
- `UseMiddleware<T>()` — 用于将指定的中间件类型添加到请求处理管道中
- `UseMiddleware<T>(params object[] args)` — 将指定的中间件类型添加到管道中，并传递参数给中间件的构造函数
- `UseRouting()` — 用于启用路由功能
- `UseEndpoints()` — 用于配置终端点路由
- `UseStaticFiles()` — 用于启用静态文件服务
- `UseAuthentication()` — 用于启用身份验证中间件
- `UseAuthorization()` — 用于启用授权中间件

### IWebHostEnvironment

`IWebHostEnvironment` 提供了访问和操作与 Web 主机环境相关的信息和属性的方法：

- `EnvironmentName` — 获取当前应用程序的环境名称，例如 `"Development"`、`"Staging"`、`"Production"` 等
- `ContentRootPath` — 获取应用程序的内容根目录的物理路径
- `WebRootPath` — 获取应用程序的 Web 根目录的物理路径
- `IsDevelopment()` — 判断当前应用程序是否处于开发环境
- `IsStaging()` — 判断当前应用程序是否处于暂存环境
- `IsProduction()` — 判断当前应用程序是否处于生产环境

---

> 笔记来源：[CodeInterpreStartup.md](https://github.com/1sanqian/Learn-Project-Structure/blob/ride-doc-structuer-note/main/CodeInterpreStartup.md)
