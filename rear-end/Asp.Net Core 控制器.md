## 控制器（Controller）

用于处理用户请求、执行业务逻辑并生成响应的核心组件之一。

负责处理来自用户的 HTTP 请求，根据请求的内容和路由规则，执行适当的操作，然后生成并返回 HTTP 响应。

#### 关键概念:

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

模型檢視控制器 (MVC) 架構模式可將一個應用程式劃分成三個主要元件：模型 (M)、檢視 (V) 和控制器 (C)。

Controllers：類別：
處理瀏覽器要求。
擷取模型資料。
呼叫傳迴響應的檢視範本。

<kbd>Controllers/HelloWorldController.cs </kbd>内容:

<!-- ```javascript

``` -->

UseDefaultFiles 在 UseStaticFiles 之前
UseStaticFiles：找到并返回它们
UseDefaultFiles：让网站跟目录 / 匹配到 /index.html 文件
