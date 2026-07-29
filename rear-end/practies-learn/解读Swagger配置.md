# 解读 Swagger 配置（AddCustomSwagger）

解读 ASP.NET Core 中通过 `Swashbuckle` 配置 Swagger 文档、JWT Bearer 认证以及 XML 注释的完整流程。

---

## 一、代码解读

这段代码是一个 `IServiceCollection` 扩展方法，主要完成四件事：

```
AddCustomSwagger
    │
    ├── ① 注册 Swagger 文档（v1）
    ├── ② 配置 JWT Bearer 认证 → 解锁 Swagger UI 的 Authorize 按钮
    ├── ③ 引入 XML 注释 → 让接口注释显示在 Swagger 页面上
    └── ④ 注册自定义 Schema 过滤器 → 让枚举值的中文描述可见
```

```csharp
public static void AddCustomSwagger(this IServiceCollection services)
{
    services.AddSwaggerGen(c =>
    {
        // ====== 一、基础配置 ======
        c.EnableAnnotations();                                      // 启用注解（如 [SwaggerOperation] 等特性）
        c.SwaggerDoc("v1", new OpenApiInfo { Title = "SmartTalk.Api", Version = "v1" });

        // ====== 二、JWT Bearer 认证 ======
        c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme { ... });
        c.AddSecurityRequirement(new OpenApiSecurityRequirement { ... });

        // ====== 三、XML 注释 ======
        c.IncludeXmlComments(Path.Combine(basePath, xmlFilename), true);
        c.IncludeXmlComments(Path.Combine(basePath, SwaggerDocs.XmlName), true);

        // ====== 四、自定义过滤器 ======
        c.SchemaFilter<SwaggerShowEnumDescriptionFilter>();
    });
}
```

---

## 二、逐段详解

### 1. 基础 Swagger 文档注册

```csharp
c.EnableAnnotations();
c.SwaggerDoc("v1", new OpenApiInfo { Title = "SmartTalk.Api", Version = "v1" });
```

| 方法 | 作用 |
| ---- | ---- |
| `EnableAnnotations()` | 启用 `Swashbuckle.AspNetCore.Annotations`，使 `[SwaggerOperation]`、`[SwaggerParameter]` 等特性生效 |
| `SwaggerDoc("v1", ...)` | 注册一个名为 `"v1"` 的文档，Title 显示在 Swagger UI 顶部 |

> 一个项目可以注册多个文档（如 `v1`、`v2`），对应不同版本的 API。

---

### 2. JWT Bearer 认证配置（Authorize 按钮的核心）

这是 Swagger UI 右上角出现 **Authorize 按钮** 的关键代码，分为两步：

#### 2.1 定义安全方案（Security Definition）

```csharp
c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
{
    Scheme = "Bearer",
    Name = "Authorization",
    In = ParameterLocation.Header,
    Type = SecuritySchemeType.ApiKey,
    Description = @"JWT Authorization header using the Bearer scheme. ..."
});
```

| 属性 | 值 | 说明 |
| ---- | -- | ---- |
| `Scheme` | `"Bearer"` | 认证方案名 |
| `Name` | `"Authorization"` | 放在 HTTP 请求头的哪个字段里 |
| `In` | `ParameterLocation.Header` | 放在哪里：请求头 |
| `Type` | `SecuritySchemeType.ApiKey` | 认证类型：API Key 方式 |
| `Description` | ... | 显示在 Swagger UI 的说明文字 |

> 这段配置告诉 Swagger：**「本 API 通过请求头 `Authorization` 字段传递 Bearer token」**。

#### 2.2 声明安全要求（Security Requirement）

```csharp
c.AddSecurityRequirement(new OpenApiSecurityRequirement
{
    {
        new OpenApiSecurityScheme
        {
            Reference = new OpenApiReference
            {
                Type = ReferenceType.SecurityScheme,
                Id = "Bearer"
            },
            Scheme = "oauth2",
            Name = "Bearer",
            In = ParameterLocation.Header
        },
        new List<string>()
    }
});
```

| 属性 | 值 | 说明 |
| ---- | -- | ---- |
| `Reference.Type` | `ReferenceType.SecurityScheme` | 引用一个已定义的安全方案 |
| `Reference.Id` | `"Bearer"` | 引用上面 `AddSecurityDefinition` 中定义的 `"Bearer"` |
| `new List<string>()` | 空列表 | 表示不需要额外 scope（不是 OAuth2 的 scope 模式） |

> **`AddSecurityDefinition` 只是「声明」有这个认证方式，`AddSecurityRequirement` 才是「要求」所有接口默认使用它。**

---

#### 二者关系

```
AddSecurityDefinition("Bearer")
    │
    │  定义了一个叫 "Bearer" 的安全方案
    │  告诉 Swagger：token 放在 Header → Authorization 里
    │
    ▼
AddSecurityRequirement(...)
    │
    │  引用上面定义的 "Bearer"
    │  告诉 Swagger：所有接口都要求这个认证
    │
    ▼
Swagger UI 会出现 🔒 Authorize 按钮
    │
    │  用户点击后输入 token
    │
    ▼
之后每次 "Try it out" 请求
都会自动带上 Authorization: Bearer <token>
```

---

### 3. XML 注释引入

```csharp
var basePath = AppContext.BaseDirectory;
var xmlFilename = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";

c.IncludeXmlComments(Path.Combine(basePath, xmlFilename), true);
c.IncludeXmlComments(Path.Combine(basePath, SwaggerDocs.XmlName), true);
```

| 代码 | 作用 |
| ---- | ---- |
| `AppContext.BaseDirectory` | 获取应用程序的基目录（bin 目录） |
| `Assembly.GetExecutingAssembly().GetName().Name` | 获取当前程序集名称，拼接成 `SmartTalk.Api.xml` |
| `IncludeXmlComments(path, true)` | 加载 XML 文件，`true` 表示包含控制器上的注释 |

**XML 文件从哪来？** 需要在 `.csproj` 中开启生成：

```xml
<PropertyGroup>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
</PropertyGroup>
```

开启后，编译时会把控制器/方法的 `<summary>` 注释输出到 XML 文件中，Swagger 再读取它显示在 UI 上。

> 这里引用了两个 XML：一个是主项目的，一个是 `SwaggerDocs.XmlName`（通常是存放 DTO 注释的独立项目），这样接口参数/返回值的注释也能显示。

---

### 4. 自定义 Schema 过滤器

```csharp
c.SchemaFilter<SwaggerShowEnumDescriptionFilter>();
```

`SchemaFilter` 用于在生成 Schema 时对模型进行自定义处理。这里 `SwaggerShowEnumDescriptionFilter` 典型作用是：

> **让枚举类型在 Swagger 中显示中文描述文本，而不是只显示数字或字段名。**

例如枚举：

```csharp
public enum OrderStatus
{
    [Description("待支付")] Pending,
    [Description("已支付")] Paid,
    [Description("已取消")] Cancelled
}
```

过滤后，Swagger UI 会展示 `0 = 待支付` 而不是 `0 = Pending`。

---

## 三、完整流程图

```
启动应用
    │
    ▼
Program.cs / Startup.cs 调用 services.AddCustomSwagger()
    │
    ├── 注册 Swagger 文档 "v1"
    ├── 配置 Bearer token 认证方案
    ├── 声明全局安全要求
    ├── 加载 XML 注释文件
    └── 注册枚举描述过滤器
    │
    ▼
Swagger UI（/swagger）
    │
    ├── 右上角 🔒 Authorize 按钮  ← 点开可输入 JWT token
    ├── 每个接口显示 Notes/Summary 注释
    ├── 参数/返回值有详细 Schema
    └── 枚举值显示中文描述
```

---

## 四、知识点速查

### SecuritySchemeType

| 类型 | 说明 |
| ---- | ---- |
| `ApiKey` | 通过 Header / Query / Cookie 传递一个 key |
| `Http` | HTTP 认证（如 Basic、Bearer） |
| `OAuth2` | OAuth2 流程 |
| `OpenIdConnect` | OpenID Connect |

### AddSecurityDefinition vs AddSecurityRequirement

| 方法 | 类比 | 作用 |
| ---- | ---- | ---- |
| `AddSecurityDefinition` | 定义一种锁的类型 | 「有 Bearer 这种认证方式」 |
| `AddSecurityRequirement` | 给门上锁 | 「所有接口默认要 Bearer 认证」 |

### 为什么用 `new List<string>()` 空列表？

Bearer + API Key 模式不需要 OAuth2 的 scope，传空列表即可。如果用的是完整的 OAuth2 流程，这个列表里就是 scope 名称，如 `new List<string> { "openid", "profile", "api" }`。

---

## 五、一句话总结

> `AddSecurityDefinition` 定义认证方式，`AddSecurityRequirement` 要求所有接口使用它 — 二者缺一不可，合在一起 Swagger UI 才会出现 **Authorize** 按钮。
