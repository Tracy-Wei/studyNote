## 概述

ASP.NET Core 是一个跨平台的高性能开源框架，用于生成启用云且连接 Internet 的新式应用。

好处与用途：

1. 生成 Web 应用和服务、物联网 (IoT) 应用和移动后端。
2. 在 Windows、macOS 和 Linux 上使用喜爱的开发工具。
3. 部署到云或本地
4. 在 .NET Core 上运行。

创建项目：

```javascript
dotnet new webapp -o aspnetcoreapp
```

信任证书：

```javascript
dotnet dev-certs https --trust
```

## 生命周期

ASP.NET Core 的依赖注入（DI）容器提供了三种生命周期：瞬时（Transient）、作用域（Scoped）和单例（Singleton）。这些生命周期影响着服务实例的创建和销毁方式，进而影响着应用程序的性能和可靠性。ASP.NET Core 的依赖注入生命周期，包括每种生命周期的特点、如何选择合适的生命周期、如何实现自定义的生命周期等。

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/f7fd89b4-704c-45e1-9a47-100ebff3afe0)

### 執行流程
![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/9646dd57-71d3-42c4-9be9-0a359516d260)

1. Main
   - NET Core 把 Web 及 Console 專案都變成一樣的啟動方式，預設從 Program.cs 的 Main() 做為程式進入點。
2. CreateHostBuilder
   - 可以在 Host 產生之前設定一些前置準備動作，當 Host 建立完成時，就可以使用已準備好的物件等。
3. Host 載入設定檔
   - CreateDefaultBuilder：預設會將 appsettings.json 作為設定檔,載入到 Startup 的 configuration。如果需要依照不同的環境(開發,正式…)來讀取設定檔。
   - ConfigureWebHostDefaults：預設會使用 Kestrel 作為 Server。
   - 設定 WebHostBuilder 產生的 WebHost 時,所要執行的類別(Startup)。
4. Startup - ConfigureServices
   - 將相依的服務註冊到 DI 容器。
   - 設定適合的 DI 生命週期
     - Singleton（单例生命周期）：是最长的生命周期，整个应用程序只会创建一个服务实例。适用于那些需要在整个应用程序中共享状态的服务，例如配置（Configuration）类、缓存（Cache）类等。
       示例代码：
       ```javascript
       services.AddSingleton<IMySingletonService,MySingletonService>();
       ```
       在上面的代码中，IMySingletonService 接口被注册为单例生命周期，整个应用程序只会创建一个 MySingletonService 实例。
     - Scoped（作用域生命周期）：介于瞬时生命周期和单例生命周期之间的生命周期。每次请求都会创建一个新的服务实例，但同一请求内的所有服务实例都是相同的。适用于那些需要在请求范围内共享状态的服务，例如业务逻辑层（BLL）中的 Service、控制器（Controller）等。
       示例代码：
       ```javascript
       services.AddScoped<IMyScopedService, MyScopedService>();
       ```
       在上面的代码中，IMyScopedService 接口被注册为作用域生命周期，同一请求内的所有 MyScopedService 实例都是相同的。
     - Transient（瞬时生命周期）：是最短的生命周期，每次请求都会创建一个新的服务实例。适用于那些无状态的服务，每次 Request 時就建立一個新的，永不共用
       示例代码：
       ```javascript
       services.AddTransient<IMyService, MyService>();
       ```
       在上面的代码中，IMyService 接口被注册为瞬时生命周期，每次请求都会创建一个新的 MyService 实例。
5. Startup - Configure
   - 參數的實例都是從 WebHost 注入進來，可依需求增減需要的參數。
   - 設定中介軟體(Middlewares)的流程。
6. Build
   - 當前置準備都設定完成後，就可以呼叫此方法實例化 Host，同時也會實例化 WebHost。
7. Run
   - 啟動 Host。
