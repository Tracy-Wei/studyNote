## 控制器（Controller）

用于处理用户请求、执行业务逻辑并生成响应的核心组件之一。

负责处理来自用户的 HTTP 请求，根据请求的内容和路由规则，执行适当的操作，然后生成并返回 HTTP 响应。

#### 关键概念:

1. 控制器类： 控制器是一个 C# 类，通常继承自 ASP.NET Core 框架提供的 ControllerBase 或 Controller 类。这些基类提供了处理请求的常见方法和属性。

2. 动作方法（Action Method）： 控制器类中的方法被称为动作方法。每个动作方法对应于一个特定的 HTTP 请求，例如 GET、POST 等。动作方法通常用于执行业务逻辑、获取数据，并生成响应。

3. 路由： ASP.NET Core 使用路由来将特定的 URL 映射到控制器和动作方法。在 Startup.cs 中配置路由规则，指定不同 URL 模式与控制器和动作方法之间的映射关系。

4. HTTP 请求和响应： 控制器的动作方法可以接收 HTTP 请求，并返回相应的 HTTP 响应。请求和响应中的数据可以通过参数和返回值来处理。

5. 参数绑定： 控制器的动作方法可以通过参数来获取 URL 中的参数、表单数据、查询字符串等。ASP.NET Core 使用模型绑定来将请求数据绑定到方法参数。

6. 视图生成： 控制器通常生成视图用于呈现 HTML 内容，然后返回该视图作为响应。视图是包含 HTML 和 Razor 代码的文件，它们可以使用模型数据来动态生成内容。

7. 过滤器： ASP.NET Core 控制器支持使用过滤器来在请求处理的不同阶段进行操作。过滤器可以用于实现身份验证、授权、日志记录等功能。

8. 依赖注入： 控制器可以通过依赖注入获取其他服务和组件，以处理业务逻辑。

9. RESTful API 支持： 控制器不仅用于生成 HTML 视图，还可以用于构建 RESTful API，返回 JSON 或其他格式的数据。

简单的 ASP.NET Core 控制器示例：

```javascript
using Microsoft.AspNetCore.Mvc;

public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View(); // 返回默认视图
    }

    public IActionResult Welcome(string name)
    {
        ViewData["Message"] = $"Hello, {name}!";
        return View();
    }
}
```
