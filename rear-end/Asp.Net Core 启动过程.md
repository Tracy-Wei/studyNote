## ASP.NET Core 启动流程

ASP.NET Core 应用程序的启动主要包含三个步骤：

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/8ff8e38a-6497-401a-8b53-c0c2901e61b0)

1. CreateDefaultBuilder()：创建 IWebHostBuilder
2. Build()：IWebHostBuilder 负责创建 IWebHost
3. Run()：启动 IWebHost

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/6993b6a3-7e59-4bca-9094-8633be8fe81f)

其核心主要在于 WebHost 的创建，又可以划分为三个部分：

1. 构建依赖注入容器，初始通用服务的注册：BuildCommonService();
2. 实例化 WebHost：var host = new WebHost(...);
3. 初始化 WebHost，也就是构建由中间件组成的请求处理管道：host.Initialize();

### 宿主构造器：IWebHostBuilder

在启动 IWebHost 宿主之前，我们需要完成对 IWebHost 的创建和配置。而这一项工作需要借助**IWebHostBuilder 对象**来完成的，ASP.NET Core 中提供了默认实现 **WebHostBuilder**。而 WebHostBuilder 是由 WebHost 的同名工具类（Microsoft.AspNetCore 命名空间下）中的 **CreateDefaultBuilder** 方法创建的。=》总的来说就是用 Host 的 CreateDefaultBuilder（）来进行创建 IWebHostBuilder

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/d15d47e6-bd67-4389-bd26-a367e3b3ac54)

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/c1a77f83-f111-40a4-b12b-15db8fc356e9)

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/1190b42e-7b12-4802-a6dc-402220540f82)

创建完毕 WebHostBuilder 后，通过调用 UseStartup()来指定启动类，来为后续服务的注册及中间件的注册提供入口。

### 宿主：IWebHost

宿主的创建是通过调用 IWebHostBuilder 的 Build()方法来完成的。

### 启动 WebHost

WebHost 的启动主要分为两步：

1. 再次确认请求管道正确创建
2. 启动 Server 以监听请求
3. 启动 HostedService

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/bc49ec3c-e515-4990-be73-a22b221a319b)
