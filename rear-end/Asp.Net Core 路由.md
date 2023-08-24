## Asp.Net Core 路由

**官方解释**：负责匹配传入的 HTTP 请求，然后将这些请求发送到应用的可执行终结点。 终结点是应用的可执行请求处理代码单元。 终结点在应用中进行定义，并在应用启动时进行配置。 终结点匹配过程可以从请求的 URL 中提取值，并为请求处理提供这些值。 通过使用应用中的终结点信息，路由还能生成映射到终结点的 URL。（是一个允许你定义如何将传入的 HTTP 请求映射到应用程序中的特定处理代码，以便对请求进行处理并返回相应的结果。这样，你可以根据 URL 路径来确定应该执行哪些操作。）

可以使用以下内容配置路由：

- Controllers
- Razor Pages
- SignalR
- gRPC 服务
- 启用终结点的中间件，例如运行状况检查。
- 通过路由注册的委托和 Lambda。

#### 路由基础知识

常规路由的代码片段：

```javascript
routeBuilder.MapRoute("Default", "{controller=Home}/{action=Index}/{id?}");
```

示例：

```javascript
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

前面的示例包含使用 MapGet 方法的单个终结点：

- 当 HTTP GET 请求发送到根 URL / 时：
  - 将执行请求委托。
  - Hello World! 会写入 HTTP 响应。
- 如果请求方法不是 GET 或根 URL 不是 /，则无路由匹配，并返回 HTTP 404。

路由使用一对由 UseRouting 和 UseEndpoints 注册的中间件：

- **UseRouting** 向中间件管道添加路由匹配。 此中间件会查看应用中定义的终结点集，并根据请求选择最佳匹配。
- **UseEndpoints** 向中间件管道添加终结点执行。 它会运行与所选终结点关联的委托。

WebApplicationBuilder 配置中间件管道，该管道使用 UseRouting 和 UseEndpoints 包装在 Program.cs 中添加的中间件。 但是，应用可以通过显式调用这些方法来更改 UseRouting 和 UseEndpoints 的运行顺序。

显式调用 UseRouting：

```javascript
// 会注册一个在管道的开头运行的自定义中间件。
app.Use(async (context, next) => {
  // ...
  await next(context);
});

// 将路由匹配中间件配置为在自定义中间件之后运行。
app.UseRouting();

// 注册的终结点在管道末尾运行。
app.MapGet("/", () => "Hello World!");
```

#### 概念

路由系统通过添加功能强大的终结点概念，构建在中间件管道之上。 终结点代表应用的功能单元，在路由、授权和任意数量的 ASP.NET Core 系统方面彼此不同。

### 路由模板

<kbd>{controller}/{action}/{id?}</kbd>
路由参数必须具有名称，且可能指定了其他特性。

### 屬性路由(Attribute Routing)

- 套用 Attribute 在 Controller 類別：

  - [Route("api/[controller]")]

- 套用 Attribute 在 Action 方法：
  - [HttpGet("{id}")]
  - [HttpPost("")]
  - [HttpDelete("{id}")]
  - [HttpPut("{id}")]
  - [HttpPatch("{id}")]

其用法效果如：
![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/0a0b53c4-21c5-4bb1-83a4-7579aa126d1b)

**Action通常用于接收传过来的参数**

示例：
/customers/1/orders

```javascript
public class OrdersController : ControllerBase
{
    [HttpGet("customers/{customerId}/orders")]
    // [HttpGet("customers/{customerId:int}/orders")] =》customerId必須為int
    public IEnumerable<Order> FindOrdersByCustomer(int customerId) { ... }
}
```

屬性路由的參數還可以設定預設值或可選參數

```javascript
public class OrdersController : ControllerBase
{
    [HttpGet("{id:int=1}")]
    // [HttpGet("{id:int?}")] =》設置為可選參數，並讓預設值加在傳入的參數中
    public IEnumerable<Order> Get(int id) { ... }
}
```

### Controller 的前置詞

在 API 的 Controller 中，慣例都會加上 api/ 作為路由前置詞(Route Prefixes)後面再帶上 Controller 名稱
[Route("api/[controller]")]，不過如果今天有某個 Action 不想要沿用這樣子的前置詞，也可以為特定的 Action 做出自訂的路由，例如：

```javascript
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    [HttpGet("~/orders/")]
    public IEnumerable<Order> Get()
    {
        ...
    }
}
```
