## .NET 依赖项注入

ASP.NET Core 框架的依赖注入（Dependency Injection，简称 DI）是一种设计模式，用于管理组件之间的依赖关系，以及在运行时将所需的依赖项传递给组件。这种模式有助于实现松散耦合、可测试性和可维护性的代码。

.NET 支持依赖关系注入 (DI) 软件设计模式，这是一种在类及其依赖项之间实现控制反转 (IoC) 的技术。

.NET 中的依赖关系注入是框架的内置部分，与配置、日志记录和选项模式一样

#### 依赖关系注入通过以下方式解决了这些问题：

- 使用接口或基类将依赖关系实现抽象化。
- 在服务容器中注册依赖关系。 .NET 提供了一个内置的服务容器 IServiceProvider。 服务通常在应用启动时注册，并追加到 IServiceCollection。 添加所有服务后，可以使用 BuildServiceProvider 创建服务容器。
- 将服务注入到使用它的类的构造函数中。 框架负责创建依赖关系的实例，并在不再需要时将其释放。

### ASP.NET Core 中使用依赖注入的步骤如下：

1. 在 Startup.cs 文件的 ConfigureServices 方法中，使用 services 参数注册服务，指定服务接口和实现类之间的映射关系，以及服务的生命周期。

```javascript
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<IMyService, MyService>(); // 注册服务
}
```

2. 在需要使用服务的地方，通过构造函数注入服务。

```javascript
public class HomeController : Controller
{
    private readonly IMyService _myService;

    public HomeController(IMyService myService)
    {
        _myService = myService;
    }

    // 使用 _myService 进行操作
}
```

3. DI 容器会在运行时自动解析并提供所需的服务实例。

好处：

- 松散耦合： 组件之间的依赖关系减弱，使代码更易于维护和扩展。
- 可测试性： 依赖项可以被轻松地替换为模拟实现，以便更轻松地进行单元测试。
- 可配置性： 在 DI 容器中注册和配置服务，使应用程序更具灵活性，可以在运行时进行更改。
