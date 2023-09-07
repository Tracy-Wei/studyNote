## AutoMapper

是基于对象到对象约定的映射工具，它可以把复杂的对象模型转为 DTO。
简单来说就是：就是根据 A 的模型和 B 的模型中的定义，自动将 A 模型映射为一个全新的 B 模型。

### VO、DTO、DO、PO、DAO

- VO(View Object)：视图对象，用于展示层，它的作用是把某个指定页面(或组件)的所有数据封装起来。
- DTO(Data Transfer Object)：数据传输对象，泛指用于展示层与服务层之间的数据传输对象。
- DO(Domain Object)：领域对象，就是从现实世界中抽象出来的有形或无形的业务实体。
- PO(Persistent Object)：持久化对象，它跟持久层(通常是关系型数据库)的数据结构形成一一对应的映射关系，如果持久层是关系型数据库，那么，数据表中的每个字段(或若干个)就对应 PO 的一个(或若干个)属性。
- DAO(Data Access Object):数据访问对象，主要用来封装对数据库的操作。

例子：

```javascript
using AutoMapper;

namespace myC;

public class Program
{
    static void MapperInit()
    {
        var config = new MapperConfiguration(cfg =>
        {
            cfg.CreateMap<StudentPO, StudentDto>();
            cfg.CreateMap<StudentDto,StudentPO>();
        });
        config.AssertConfigurationIsValid();
    }

    static void Main(string[] args)
    {
        MapperInit();
        StudentPO po = new StudentPO() { ID = 1, FirstName = "Money" };
    }

    public class StudentPO // 持久化对象
    {
        public int ID { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public int Sex { get; set; }
        public DateTime Birth { get; set; }
        public string UserID { get; set; }
        public string Password { get; set; }
        public string Address { get; set; }
    }

    public class StudentDto // 数据传输对象
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public string UserID { get; set; }
    }
}
```

用法：

1. 配置映射关系：

```javascript
public class OrganizationProfile : Profile
{
	public OrganizationProfile()
	{
		CreateMap<Foo, FooDto>();
	}
}

```

2. ConfigureServices 里 Add,入参类型为 params

```javascript
services.AddAutoMapper(typeof OrganizationProfile);
```

3. 然后在 Controller 里使用即可：

```javascript
public XXXController(IMapper mapper)
{
	_mapper = mapper;
	var dest=mapper.Map<OrderDto>(order);
}
```

### 反向映射

PO->DTO 和 DTO->PO 转换的映射，我们也可以通过反向映射来代替：

```javascript
cfg.CreateMap<StudentPO, StudentDto>().ReverseMap();
```

ForMember()自定义映射关系：

```javascript
cfg.CreateMap<StudentPO, StudentDto>().ForMember(des => des.EnName,op => op.MapFrom(src => src.ZhName));
```

### Profile

是组织映射的另一种方式。新建一个类，继承 Profile，并在构造函数中配置映射。

```javascript
public class EmployeeProfile : Profile
{
    public EmployeeProfile()
    {
        CreateMap<Employee, EmployeeDto>();
    }
}

var config = new MapperConfiguration(cfg =>
{
    cfg.AddProfile<EmployeeProfile>();
});
```

AutoMapper 也可以在指定的程序集中扫描从 Profile 继承的类，并将其添加到配置中。

```javascript
var config = new MapperConfiguration((cfg) => {
  // 扫描当前程序集
  cfg.AddMaps(System.AppDomain.CurrentDomain.GetAssemblies());

  // 也可以传程序集名称（dll 名称）
  cfg.AddMaps("LibCoreTest");
});
```

### DTO 和实体类区别

- DTO（AddProductDto）：DTO 是数据传输对象的缩写，通常用于在不同的应用程序层之间传输数据。在这里，AddProductDto 用于在应用程序的不同部分传递有关产品的数据。DTO 类通常只包含与传输相关的数据，而不包含业务逻辑或数据访问逻辑。

- 实体类（Product）：实体类通常用于表示应用程序的领域模型，它们包含与业务逻辑和数据访问相关的信息。在这里，Product 可能是数据库中产品的模型，包含了与产品相关的数据库列以及与数据库交互的数据访问逻辑。

为什么要这样做的原因：

1. 解耦合：将 DTO 与实体类分开可以减少不同层之间的耦合度。例如，数据访问层通常需要实体类来与数据库交互，而控制器或服务层可能需要 DTO 来接收或传递数据。通过将它们分开，可以降低不同层之间的依赖性。

2. 数据传输：DTO 可以用于将实体类的数据传输到客户端或其他服务，而无需将实体类的所有属性暴露给外部。这可以增强安全性并减小传输的数据量。

3. 数据转换：DTO 还可以用于数据转换，将不同格式或结构的数据转换为其他格式，以满足客户端或服务的需求。在这里，CreateMap<Product, AddProductDto>() 和 CreateMap<AddProductDto, Product>() 是用于自动执行对象之间的映射。

综上所述，定义两个差不多相同的类用于映射的目的是为了更好地组织和管理数据，并在不同层之间传输和转换数据，同时减少耦合度和提高代码的可维护性。这是一种常见的软件设计实践，特别在使用 ORM 或需要数据传输的应用程序中。

### 命名约定

AutoMapper 基于相同的字段名映射，并且是 **不区分大小写** 的。

### 配置可见性

默认情况下，AutoMapper 仅映射 public 成员

如果映射到 private 属性的话，属性必须添加 private set，如：

```javascript
var config = new MapperConfiguration(cfg =>
{
    cfg.ShouldMapProperty = p => p.GetMethod.IsPublic || p.SetMethod.IsPrivate;
    cfg.CreateMap<Source, Destination>();
});
```

### 全局属性/字段过滤

```javascript
cfg.ShouldMapField = (fi) => false;
```

### 识别前缀和后缀

```javascript
cfg.RecognizePrefixes("My");
```

### 替换字符

```javascript
cfg.ReplaceMemberName("Ä", "A");
```

### 禁用构造函数映射：

禁用构造函数映射的话，目标类要有一个无参构造函数。

```javascript
var config = new MapperConfiguration((cfg) => cfg.DisableConstructorMapping());
```

### 如果需要在 依赖注入（DI）使用，Mapster 提供了 ​​IMapper​​and​​Mapper​​

```javascript
public void ConfigureServices(IServiceCollection services)
{
    var config = new TypeAdapterConfig();
    services.AddSingleton(config);//使用单例注册

    services.AddScoped<IMapper, ServiceMapper>();//注册服务
}

// Service进行依赖注入
private readonly IMapper _mapper;

public UserService(IMapper mapper) {
    _mapper = mapper;
}

public void Create(UserRequestDto request) {
    // 使用服务
    var user = _mapper.Map<User>(request);
}
```

### 手动控制某些成员的映射 ForMember+MapFrom

将 src 的 Date.Date 映射到 dest 的 EventDate 成员上：

```javascript
// Configure AutoMapper
var configuration = new MapperConfiguration(cfg =>
  cfg.CreateMap<CalendarEvent, CalendarEventForm>()
	.ForMember(dest => dest.EventDate, opt => opt.MapFrom(src => src.Date.Date))
	.ForMember(dest => dest.EventHour, opt => opt.MapFrom(src => src.Date.Hour))
	.ForMember(dest => dest.EventMinute, opt => opt.MapFrom(src => src.Date.Minute)));
```

### 嵌套（Nested）类和继承类映射

```javascript
var config = new MapperConfiguration(cfg => {
    cfg.CreateMap<OuterSource, OuterDest>();
    cfg.CreateMap<InnerSource, InnerDest>();
});

```

### 映射继承与多态 Include/IncludeBase

假如 ChildSource 继承 ParentSource，ChildDestination 继承 ParentDestination。并且有这么一个业务，ParentSource src=new ChildSource()需要把 src 转为 ParentDestination。

```javascript
var configuration = new MapperConfiguration(c=> {
    c.CreateMap<ParentSource, ParentDestination>()
	     .Include<ChildSource, ChildDestination>();
    c.CreateMap<ChildSource, ChildDestination>();
});
```

或者：

```javascript
CreateMap<ParentSource,ParentDestination>();
CreateMap<ChildSource,ChildDestination>().IncludeBase<ParentSource,ParentDestination>();
```

或者：

```javascript
CreateMap<ParentSource,ParentDestination>().IncludeAllDerived();
CreaetMap<ChildSource,ChildDestination>();
```

### 泛型映射

注：CreateMap 不需要传具体的 T

```javascript
public class Source<T> {}
public class Destination<T> {}
var configuration = new MapperConfiguration(cfg => cfg.CreateMap(typeof(Source<>), typeof(Destination<>)));
```

## Autofac 框架

AutoFac 是一个.NET 平台上的依赖注入（DI）容器和服务定位器，也是.net 下比较流行的实现依赖注入的工具之一。它的主要作用是管理应用程序中的组件依赖关系，确保组件（类、接口等）能够以一种松耦合的方式协同工作。

Autofac IOC 依赖注入方式和生命周期以及 Autofac 配置文件配置 IOC 属性注入

Autofac IOC 依赖注入方式：构造函数(默认)，属性，方法，属性(1、接口实现类的属性注入，2、Controller 控制器中的属性注入)

Autofac IOC 生命周期以  ：瞬态(InstancePerDependency)，单例(SingleInstance)，作用域(InstancePerLifetimeScope)，作用域(标志)(InstancePerMatchingLifetimeScope("标志")), 请求(PerRequest)等

应用的基本流程如下:

1. 按照 控制反转 (IoC) 的思想构建你的应用.
2. 添加 Autofac 引用.
3. 在应用的 startup 处
4. 创建 ContainerBuilder.
5. 注册组件.
6. 创建容器,将其保存以备后续使用.
7. 应用执行阶段
8. 从容器中创建一个生命周期.
9. 在此生命周期作用域内解析组件实例.

Autofac 的好处就在于它能**批量注入**和**属性注入**，当然在这里体现的就是 autofac 属性注入的优势，省去了构造函数注入的麻烦，

1. 改用 Autofac 来实现依赖注入：

```javascript
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        // 就是这句
        .UseServiceProviderFactory(new AutofacServiceProviderFactory())
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });

```

2. 添加我们自定义的 Autofac 注册类，并注册我们需要的服务（默认构造函数注入，支持属性注入）：

```javascript
public class AutofacModuleRegister : Autofac.Module
{
    //重写Autofac管道Load方法，在这里注册注入
    protected override void Load(ContainerBuilder builder)
    {
        builder.RegisterType<UserService>().As<IUserService>();
　　 }
 }
```

3. 在 Startup 类中添加方法：ConfigureContainer：

```javascript
public void ConfigureContainer(ContainerBuilder builder)
{
    // 直接用Autofac注册我们自定义的
    builder.RegisterModule(new AutofacModuleRegister());
}

```

4. 控制器内的方法不用去改：

```javascript
public class DefaultController : ControllerBase
{
    private readonly IUserService userService;
    public DefaultController(IUserService _userService)
    {
        this.userService = _userService;
    }

    [HttpGet]
  public string Autofac()
  {
      return userService.GetName("Autofac");
  }
}
```

### 在项目中实操：

1. appsetting.json 添加本地数据库

```javascript
  "ConnectionStrings": {
    "Default": "server=localhost;userid=root;password=123456;database=smart_faq;Allow User Variables=True;"
  },
```

2. 建立一个 ConnectionString 的类，通过这个类去寻找 appsetting 里面的数据库配置

```javascript
using Microsoft.Extensions.Configuration;

namespace PractiseForTracy.Core.Setting.System;

public class ConnectionString : IConfigurationSetting<string>
{
    public ConnectionString(IConfiguration configuration)
    {
        Value = configuration.GetConnectionString("Default");
    }

    public string Value { get; set; }
}
```

3. 在 DbContext 这里引用 ConnectionString，如果 MySqlServerVersion 报错，注意要安装 Pomelo.EntityFrameworkCore.MySql 依赖

```javascript
using System.Reflection;
using Microsoft.EntityFrameworkCore;
using PractiseForTracy.Core.Domain;
using PractiseForTracy.Core.Setting.System;

namespace PractiseForTracy.Core.Data;

public class PractiseForTracyDbContext : DbContext, IUnitOfWork
{
    private readonly ConnectionString _connectionString;

    public PractiseForTracyDbContext(ConnectionString connectionString)
    {
        _connectionString = connectionString;
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseMySql(_connectionString.Value, new MySqlServerVersion(new Version(5, 7, 0)));
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        typeof(PractiseForTracyDbContext).GetTypeInfo().Assembly.GetTypes()
            .Where(t => typeof(IEntity).IsAssignableFrom(t) && t.IsClass).ToList()
            .ForEach(x =>
            {
                if (modelBuilder.Model.FindEntityType(x) == null)
                    modelBuilder.Model.AddEntityType(x);
            });
    }

    public bool ShouldSaveChanges { get; set; }
}

```

4. 在 program 加入 autofac 的引用，因为后续用到 autofac 配置依赖

```javascript
using Autofac.Extensions.DependencyInjection;

namespace PractiseForTracy.Api;

public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) => Host.CreateDefaultBuilder(args)
        .UseServiceProviderFactory(new AutofacServiceProviderFactory())
        .ConfigureWebHostDefaults(webBuilder => webBuilder.UseStartup<Startup>());
}
```

5. 在 startup 文件中引用 PractiseForTracyModule 使用依赖配置

```javascript
public void ConfigureContainer(ContainerBuilder builder)
    {
        builder.RegisterModule(new PractiseForTracyModule(Configuration, typeof(PractiseForTracyModule).Assembly));
    }
```

6. 上面的在 startup 文件中用到 PractiseForTracyModule，所以要建立了 module，利用 autofac 来配置

```javascript
using System.Reflection;
using Autofac;
using Mediator.Net;
using AutoMapper.Contrib.Autofac.DependencyInjection;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using PractiseForTracy.Core.Data;
using PractiseForTracy.Core.MiddleWares.UnifyResponse;
using PractiseForTracy.Core.MiddleWares.UnitOfWork;
using PractiseForTracy.Core.Service;
using PractiseForTracy.Core.Setting;
using Module = Autofac.Module;

namespace PractiseForTracy.Core;

public class PractiseForTracyModule : Module
{
    private readonly Assembly[] _assemblies;
    private readonly IConfiguration _configuration;

    public PractiseForTracyModule(IConfiguration configuration, params Assembly[] assemblies)
    {
        _assemblies = assemblies;
        _configuration = configuration;
    }

    protected override void Load(ContainerBuilder builder)
    {
        RegisterMediator(builder);

        RegisterDbContext(builder);

        RegisterDependency(builder);

        RegisterSettings(builder);

        RegisterAutoMapper(builder);
    }

    private void RegisterMediator(ContainerBuilder builder)
    {
        var mediatorBuilder = new MediatorBuilder();

        mediatorBuilder.RegisterHandlers(_assemblies);
        mediatorBuilder.ConfigureGlobalReceivePipe(c =>
        {
            c.UseUnitOfWork();
            c.UseUnifyResponse();
        });
    }

    private void RegisterDbContext(ContainerBuilder builder)
    {
        builder.RegisterType<PractiseForTracyDbContext>().AsSelf().As<DbContext>().AsImplementedInterfaces()
            .InstancePerLifetimeScope();
    }

    private void RegisterDependency(ContainerBuilder builder)
    {
        foreach (var type in typeof(IService).Assembly.GetTypes().Where(t => typeof(IService).IsAssignableFrom(t) && t.IsClass))
        {
            switch (true)
            {
                case bool _ when typeof(IInstancePerLifetimeScopeService).IsAssignableFrom(type):
                    builder.RegisterType(type).AsImplementedInterfaces().InstancePerLifetimeScope();
                    break;
                case bool _ when typeof(ISingletonService).IsAssignableFrom(type):
                    builder.RegisterType(type).AsImplementedInterfaces().SingleInstance();
                    break;
                default:
                    builder.RegisterType(type).AsImplementedInterfaces();
                    break;
            }
        }
    }

    private void RegisterSettings(ContainerBuilder builder)
    {
        var settingTypes = typeof(PractiseForTracyModule).Assembly.GetTypes()
            .Where(t => t.IsClass && typeof(IConfigurationSetting).IsAssignableFrom(t))
            .ToArray();

        builder.RegisterTypes(settingTypes).AsSelf().SingleInstance();
    }

    private void RegisterAutoMapper(ContainerBuilder builder)
    {
        builder.RegisterAutoMapper(typeof(PractiseForTracyModule).Assembly);
    }
}
```
