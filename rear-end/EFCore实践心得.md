## EFCore 的构造思路步骤
 [参考][1]
1. 建 person 类

```javascript
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace PractiseForTracy.Core.Domain;

[Table("people")]
public class Person
{
    [Key] [Column("id")] public int Id { get; set; }

    [Column("first_name")]
    public string FirstName { get; set; }

    [Column("last_name")] public string LastName { get; set; }

    [Column("create_at")]
    public DateTime CreateAt { get; set; }

}
```

2. 创建数据库上下文

```javascript
using Microsoft.EntityFrameworkCore;
using PractiseForTracy.Core.Domain;

namespace PractiseForTracy.Core.Data;

public class MyDbContext:DbContext
{
    public DbSet<Person> People { get; set; }

    public MyDbContext(DbContextOptions<MyDbContext> options, DbSet<Person> people) : base(options)
    {
        People = people;
    }
}
```

3. 连接数据库
   表：

```javascript
CREATE TABLE `people`  (
  `Id` int NOT NULL AUTO_INCREMENT,
  `FirstName` varchar(50) NULL,
  `LastName` varchar(50) NULL,
  `CreatedAt` datetime NULL,
  PRIMARY KEY (`Id`)
);
```

appsetting.json:

```javascript
{
  "ConnectionStrings": {
    "Default": "server=localhost;database=people;user=root;password=123456"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

Startup.cs 添加 DbContext 的依赖注入:

```javascript
public Startup(IConfiguration configuration) // 允许configuration访问配置信息
{
    Configuration = configuration;
}

public IConfiguration Configuration { get; } //通过配置信息，存储注入的接口

public void ConfigureServices(IServiceCollection service)
{
    service.AddControllers();
    service.AddMvc();
    service.AddDbContext<MyDbContext>(option => option.UseMySql(Configuration.GetConnectionString("Default"), new MySqlServerVersion(new Version(5, 7, 0) )));
}
```

4. 创建 provider 数据访问层
   定义了三个方法：
   - Creat
   - GetPersonAll
   - GetPersonById

```javascript
using PractiseForTracy.Core.Data;
using PractiseForTracy.Core.Domain;

namespace PractiseForTracy.Core.IPersonService;

public interface IPersonDataProvider
{
    int Creat();

    List<Person> GetPersonAll();

    Person GetPersonById(int id);
}

public abstract class PersonDataProvider : IPersonDataProvider // 抽象类
{
    private readonly MyDbContext _dbContext;

    protected PersonDataProvider(MyDbContext dbContext) // 注入DbContext
    {
        _dbContext = dbContext;
    }

    public int Creat()
    {
        var person = new Person
        {
            FirstName = "Rector",
            LastName = "Liu",
            CreateAt = DateTime.Now,
        };
        _dbContext.People.Add(person);
        return _dbContext.SaveChanges();
    }

    public List<Person> GetPersonAll()
    {
        return _dbContext.People.ToList();
    }

    public Person GetPersonById(int id)
    {
        return _dbContext.People.Find(id) ?? new Person();
    }
}
```

5. 创建 service 业务层
   定义了三个方法：
   - AddPerson => 调用 provider 的 Creat
   - GetAllPersons => 调用 provider 的 GetPersonAll
   - GetPersonById => 调用 provider 的 GetPersonById

```javascript
using PractiseForTracy.Core.Domain;

namespace PractiseForTracy.Core.IPersonService;

public interface IPersonService
{
    string AddPerson();
    List<Person> GetAllPersons();
    Person GetPersonById(int id);
}

public class PersonService : IPersonService
{
    private readonly IPersonDataProvider _personDataProvider;

    public PersonService(IPersonDataProvider personDataProvider) // 注入Provider
    {
        _personDataProvider = personDataProvider;
    }

    public string AddPerson()
    {
        return _personDataProvider.Creat() > 0 ? "数据写入成功" : "数据写入失败";
    }

    public List<Person> GetAllPersons()
    {
        return _personDataProvider.GetPersonAll();
    }

    public Person GetPersonById(int id)
    {
        return _personDataProvider.GetPersonById(id);
    }

}
```

6. 创建控制器（Controllers）
   处理关于人员信息的 HTTP 请求

```javascript
using Microsoft.AspNetCore.Mvc;
using PractiseForTracy.Core.IPersonService;

namespace PractiseForTracy.Api.Controllers;

[ApiController]
[Route("api/[controller]/[action]")]
public class PeopleController : ControllerBase
{
    private readonly IPersonService _personService;

    public PeopleController(IPersonService personService) //注入service
    {
        _personService = personService;
    }

    [HttpGet]
    public IActionResult Create() //创建人员
    {
        return Ok(_personService.AddPerson()); // 调用service的三个方法
    }

    [HttpGet]
    public IActionResult GetById(int id) // 根据人员id获取人员信息
    {
        return Ok(_personService.GetPersonById(id));
    }

    [HttpPost]
    public IActionResult GetAll() // 获取所有人员的信息
    {
        return Ok(_personService.GetAllPersons());
    }
}
```

[1]: [https://github.com/DOGGIE4/desktop-tutorial/blob/main/EF%20Core%20Learn.md]
