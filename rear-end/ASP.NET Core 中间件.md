## ASP.NET Core 中间件

中间件是一种装配到应用管道以处理请求和响应的软件。

每个组件：

- 选择是否将请求传递到管道中的下一个组件。
- 可在管道中的下一个组件前后执行工作。

请求委托用于生成请求管道。请求委托处理每个 HTTP 请求。

中间件:也叫中间件组件.使用 RunMap 和 Use 扩展方法来配置请求委托。 可将一个单独的请求委托并行指定为匿名方法（称为并行中间件），或在可重用的类中对其进行定义。 这些可重用的类和并行匿名方法即为中间件.

用处：请求管道中的每个中间件组件负责调用管道中的下一个组件，或使管道短路。

终端中间件：当中间件短路时，它被称为“终端中间件”，因为它阻止中间件进一步处理请求。

### 使用 WebApplication 创建中间件管道

ASP.NET Core 请求管道包含一系列请求委托，依次调用。

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/b0c90e77-69dd-4566-9c75-a33cfe400de0)
每个委托均可在下一个委托前后执行操作。 应尽早在管道中调用异常处理委托，这样它们就能捕获在管道的后期阶段发生的异常。

最简单的可能是 ASP.NET Core 应用程序建立一个请求的委托，处理所有的请求。此案例不包含实际的请求管道。相反，针对每个 HTTP 请求都调用一个匿名方法。

- Startup 类中的 Configure() 方法用于定义请求管道中的中间件

```javascript
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;

public class Startup
{
  public void Configure(IApplicationBuilder app)
  {
    //  app.Run 委托终止管道
    app.Run(async context => {
        await context.Response.WriteAsync("Hello, World!");
    });
  }
}
```

您可以将多个请求委托与 app.Use 连接在一起。 next 参数表示管道中的下一个委托。 （请记住，您可以通过不调用下一个参数来结束流水线。）通常可以在下一个委托之前和之后执行操作，如下例所示可以看出请求委托的执行顺序是遵循上面的流程图的：

```javascript
public class Startup
{
  public void Configure(IApplicationBuilder app)
  {
    app.Use(async (context, next) => {
      await context.Response.WriteAsync("进入第一个委托 执行下一个委托之前\r\n");
      //调用管道中的下一个委托
      await next.Invoke();
      await context.Response.WriteAsync("结束第一个委托 执行下一个委托之后\r\n");
    });
    app.Run(async context => {
      await context.Response.WriteAsync("进入第二个委托\r\n");
      await context.Response.WriteAsync("Hello from 2nd delegate.\r\n");
      await context.Response.WriteAsync("结束第二个委托\r\n");
    });
  }
}
```

> 注意：响应发送到客户端后，请勿调用 next.Invoke。 响应开始之后，对 HttpResponse 的更改将抛出异常。 例如，设置响应头，状态代码等更改将会引发异常。在调用 next 之后写入响应体。

- 可能导致协议违规。 例如，写入超过 content-length 所述内容长度。
- 可能会破坏响应内容格式。 例如，将 HTML 页脚写入 CSS 文件。
- HttpResponse.HasStarted 是一个有用的提示，指示是否已发送响应头和/或正文已写入。

### 中间件顺序

在 Startup，Configure 方法中添加中间件组件的顺序定义了在请求上调用它们的顺序，以及响应的相反顺序。 此排序对于安全性，性能和功能至关重要。

Startup.Configure 方法（如下所示）添加了以下中间件组件：

1. 异常/错误处理
2. 静态文件服务
3. 身份认证
4. MVC

```javascript
public void Configure(IApplicationBuilder app)
{
  // 用于处理应用程序中发生的未处理异常。它允许你在应用程序发生异常时执行自定义的错误处理逻辑，例如返回一个友好的错误页面或错误信息，以提供更好的用户体验。
  app.UseExceptionHandler("/Home/Error");

  // 用于提供静态文件（例如 HTML、CSS、JavaScript、图像等）的中间件。
  app.UseStaticFiles();

  // 用于处理身份验证和授权。它会在请求处理的早期阶段进行身份验证，检查用户的身份是否有效，并根据用户的身份信息授予或拒绝访问权限。
  app.UseAuthentication();

  // 这是一个用于配置 MVC（Model-View-Controller）的中间件。它设置了一个默认路由规则，以便根据控制器和操作来处理传入的 HTTP 请求，并将他们映射到相应的处理方法
  app.UseMvcWithDefaultRoute();
}
```

下图显示了 ASP.NET Core MVC 和 Razor Pages 应用的完整请求处理管道。

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/4553760f-1813-440d-b0ad-5cc42a482cd1)
上图中的“终结点”中间件为相应的应用类型（MVC 或 Razor Pages）执行筛选器管道

### Use,Run 和 Map

可以使用 Use，Run 和 Map 配置 HTTP 管道。

Use 方法可以使管道短路（即，可以不调用下一个请求委托）。Run 方法是一个约定， 并且一些中间件组件可能暴露在管道末端运行的 Run [Middleware]方法。Map 扩展用作分支管道的约定。

映射根据给定的请求路径的匹配来分支请求流水线，如果请求路径以给定路径开始，则执行分支。

#### 因为中间件是在应用程序启动时构建的，而不是每个请求，所以在每个请求期间，中间件构造函数使用的作用域生命周期服务不会与其他依赖注入类型共享。
