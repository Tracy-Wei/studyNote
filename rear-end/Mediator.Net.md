## Mediator.Net 框架

是一个用于实现中介者（Mediator）模式的开源框架，它在 C# 应用程序中帮助实现解耦和更好的组织业务逻辑。框架中的中介者模式基于消息传递，允许不同的组件通过中介者进行通信，从而实现松耦合的架构。

中介者模式：用一个中介对象来封装一系列的对象交互。中介者使各个对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/4d351f5b-5e1c-4f1f-a8af-7653be7efa3e)

安装 nuget 包 Mediator.Net：

```javascript
Install-Package Mediator.Net
```

在 ASP.NET Core 应用程序中实现中介者模式，可以按照以下步骤进行：

1. 创建中介者类：创建一个中介者类，用于管理对象之间的交互。中介者类应该包含与其他对象通信的方法，并处理任何必要的逻辑。

2. 创建对象：创建需要相互交互的对象。这些对象可以包括服务、控制器、视图组件或任何其他需要相互交互的组件。

3. 注册对象：在中介者类中注册这些对象。这可以通过构造函数注入或手动注册等方式完成。

4. 调用中介者：在需要两个或多个对象之间交互的地方，调用中介者类中的相应方法。这将触发中介者类中的逻辑，并促进对象之间的通信。

Mediator：抽象中介者，定义了同事对象到中介对象的接口。

ConcreteMediator：具体中介者，实现了抽象类的方法。它需要知道所有具体的同事类，并从某个具体同事类接收消息，向某个具体同事类发送消息。

Colleague：抽象同事类。

ConcreteColleague：具体同事类，每个同事类都知道自己的行为，而不了解其它同事类的行为，但都认识中介对象。
示例：

```javascript
// 定义中介者接口
public interface IMediator
{
    void Send(string message, Colleague colleague);
}

// 定义同事类
public class Colleague
{
    private readonly IMediator _mediator;

    public Colleague(IMediator mediator)
    {
        _mediator = mediator;
    }

    public void Send(string message)
    {
        _mediator.Send(message, this);
    }

    public void Notify(string message)
    {
        Console.WriteLine($"Received message: {message}");
    }
}

// 定义中介者类
public class ConcreteMediator : IMediator
{
    private readonly Colleague _colleague1;
    private readonly Colleague _colleague2;

    public ConcreteMediator(Colleague colleague1, Colleague colleague2)
    {
        _colleague1 = colleague1;
        _colleague2 = colleague2;
    }

    public void Send(string message, Colleague colleague)
    {
        if (colleague == _colleague1)
        {
            _colleague2.Notify(message);
        }
        else
        {
            _colleague1.Notify(message);
        }
    }
}
```

在 Startup.cs 中注册中介者和同事类：

```javascript
services.AddScoped<IMediator, ConcreteMediator>();
services.AddScoped<Colleague>();

// 在Controller中使用中介者模式
public class HomeController : Controller
{
    private readonly Colleague _colleague;

    public HomeController(Colleague colleague)
    {
        _colleague = colleague;
    }

    public IActionResult Index()
    {
        _colleague.Send("Hello World!");
        return View();
    }
}
```

### 管道^[1]：

可以使用五种不同类型的管道

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/a64f96b0-aa38-4948-8dfd-f2ad0503a8f1)

全局接收管道
每当消息在到达下一个管道和处理程序之前发送、发布或请求时，都会触发此管道

命令接收管道
该管道将 ​​ 在到达其命令处理程序之后和之前触发 GlobalReceivePipeline，该管道仅用于 ICommand

事件接收管道
该管道将 ​​ 在到达其事件处理程序之后和之前触发 GlobalReceivePipeline，该管道仅用于 IEvent

请求接收管道
该管道将 ​​ 在到达其请求处理程序之后和之前触发 GlobalReceivePipeline，该管道仅用于 IRequest

发布管道
当在处理程序中发布 an 时，将触发此管道 IEvent，此管道仅用于 IEvent 且通常用作传出拦截器

上述管道最强大的功能是您可以添加任意数量的中间件。

设置中间件：

- 为您的中间件添加静态类
- 在刚刚添加的类中添加公共静态扩展方法，通常遵循 UseXxxx 命名约定
- 为您的中间件规范添加另一个类，请注意，这是您的中间件的实现

中间件可能需要一些依赖项，有两种方法可以实现

- 明确地传递它们
- 让 IoC 容器为您解决（如果您正在使用 IoC）

### 应用程序分为三个主要的层：

表示层（Presentation Layer）、业务逻辑层（Business Logic Layer）和数据访问层（Data Access Layer）。

### mediator 具体：

注册 Handler 和信息（Message）之间的绑定，当他收到特定信息时，会在它的注册表里找出对应的 Handler 从而调用。

#### mediator 具体是怎么实现寻找到对应的 handler 的呢？

1. 首先会在 controller 使用 SendAsync(command)/RequestAsync(request)
   （使用了 SendMessage 来判断传来的 IMessage 是属于 command/request/event 中的哪一种类型）
2. 随后选择到对应的管道，寻找到对应的 handler
3. 最后 handle 执行的任务。

#### Message Contract

用于在分布式系统中定义和传递消息的结构和内容。

在分布式系统中，不同的服务或组件可能需要相互通信，而消息通常是实现这种通信的一种方式。

Message Contract 确保不同服务之间的消息格式和内容是一致的，从而实现解耦和可维护性。

#### 应用步骤流程 ：

1. 首先会在controller引用mediator，用controller发对应信息（上述的Message Contract）
    ```javascript
    using Mediator.Net;
    using Microsoft.AspNetCore.Mvc;
    using PractiseForTracy.Message.Commands;

    namespace PractiseForTracy.Api.Controllers
    {
        [ApiController]
        [Route("api/[controller]")]
        public class PersonController : ControllerBase
        {
            public readonly IMediator _mediator;

            public PersonController(IMediator mediator)
            {
                _mediator = mediator;
            }
        
            [HttpPost]
            [Route("create")]
            public async Task<IActionResult> CreateAsync([FromBody] CreatePeopleCommand command)
            {
                var response = await _mediator
                    .SendAsync<CreatePeopleCommand, CratePeopleResponse>(command)
                    .ConfigureAwait(false);

                return Ok(response);
            }
        }
    }
    ```

    - SendAsync：是 Mediator 模式中的一个方法，用于发送一个请求或命令，并返回响应。在 Mediator 模式中，它将请求发送到适当的处理程序（也称为请求处理程序或命令处理程序），然后由处理程序执行相应的操作并返回结果。

        ```javascript
        var response = await _mediator.SendAsync<RequestType, ResponseType>(request);

        // <RequestType, ResponseType> : 指定请求的类型（RequestType）和期望的响应类型（ResponseType）。

        // request：要发送的请求实例，这个请求实例会传递给相应的处理程序来进行处理。
        ```

    - CreateAsync: 是一个控制器操作方法（Action Method），它用于处理 HTTP POST 请求，通常用于创建新资源

    - [FromBody]: 是一个属性，用于告诉 ASP.NET Core 框架从 HTTP 请求的主体中读取数据，然后将其绑定到方法的参数上。
  
    - CreatePeopleCommand: 是参数的类型，表示这个方法期望接收一个名为 command 的 HTTP 请求主体中的 JSON 数据，并将其反序列化为 CreatePeopleCommand 类型的对象。

    - Ok(response)指的是：这个方法会构造一个 HTTP 响应，将状态码设置为 200 OK，并将响应对象序列化为 JSON（默认情况下）并作为响应的内容返回给客户端。这样客户端就能够收到服务器处理后的数据。
      - Ok：表示成功的状态码，通常是 200 OK。
      - response：要返回给客户端的数据或对象。

    - ConfigureAwait(false)：是一种用于配置异步操作上下文的方法。用于指定在异步操作完成后是否要切换回原始的上下文（通常是调用线程的上下文）。

2. 根据sendAsynv方法去判断CreatPeopleCommand和CratePeopleResponse。
    ```javascript
    using Mediator.Net.Contracts;
    using PractiseForTracy.Message.Dto;

    namespace PractiseForTracy.Message.Commands;

    public class CreatePeopleCommand : ICommand
    {
        public CreateOrUpdatePeopleDto Person { get; set; }
    }

    public class CratePeopleResponse : IResponse
    {
        public string Result { get; set; }
    }
    ```
    ```javascript
    namespace PractiseForTracy.Message.Dto;

    public class CreateOrUpdatePeopleDto
    {
        public int Id { get; set; }

        public string FirstName { get; set; }
        
        public string LastName { get; set; }

        public DateTime CreateAt { get; set; }
    }
    ```

3. 根据对应的管道找到对应的handler，这里通过Handle方法来调用了 _personService 的 AddPersonAsync 方法。实现后会生成一个事件，并通过PublishAsync通知到对应的 EventHandler去做后续的处理。
    ```javascript
    using Mediator.Net.Context;
    using Mediator.Net.Contracts;
    using PractiseForTracy.Core.Service;
    using PractiseForTracy.Message.Commands;

    namespace PractiseForTracy.Core.Handlers;

    public class CreatePeopleCommandHandler : ICommandHandler<CreatePeopleCommand,CratePeopleResponse>
    {
        public readonly IPersonService _personService;

        public CreatePeopleCommandHandler(IPersonService personService)
        {
            _personService = personService;
        }

        public async Task<CratePeopleResponse> Handle(IReceiveContext<CreatePeopleCommand> context, CancellationToken cancellationToken)
        {
            var @event = await _personService.AddPersonAsync(context.Message, cancellationToken).ConfigureAwait(false);
            
            await context.PublishAsync(@event, cancellationToken).ConfigureAwait(false);

            return new CratePeopleResponse
            {
                Result = @event.Result
            };
        }
    }
    ```
4. EventHandler
    ```javascript
    using Mediator.Net.Context;
    using Mediator.Net.Contracts;
    using PractiseForTracy.Message.Events.People;

    namespace PractiseForTracy.Core.Handlers.EventHandler;

    public class PeopleCreatedEvent:IEventHandler<PeopleCreateEvent>
    {
        public async Task Handle(IReceiveContext<PeopleCreateEvent> context, CancellationToken cancellationToken)
        {
            await Task.CompletedTask;
        }
    }
    ```
    ```javascript
    using Mediator.Net.Contracts;

    namespace PractiseForTracy.Message.Events.People;

    public class PeopleCreateEvent : IEvent
    {
        public string Result { get; set; }
    }
    ```
5. PersonService中调用PersonDataProvider的CreatAsync方法,返回一个PeopleCreatedEvent 对象。
    ```javascript
    using AutoMapper;
    using Mediator.Net;
    using PractiseForTracy.Core.Domain;
    using PractiseForTracy.Core.Handlers.EventHandler;
    using PractiseForTracy.Core.Service;
    using PractiseForTracy.Message.Commands;
    using PractiseForTracy.Message.Events.People;

    namespace PractiseForTracy.Core.People
    {
        public class PersonService : IPersonService
        {
            private readonly IMapper _mapper;
            private readonly IPersonDataProvider _personDataProvider;
            
            
            public async Task<PeopleCreatedEvent> AddPersonAsync(CreatePeopleCommand command, CancellationToken cancellationToken)
            {
                var response = await _personDataProvider.CreatAsync(_mapper.Map<Person>(command.Person), cancellationToken)
                    .ConfigureAwait(false) > 0
                    ? "数据写入成功"
                    : "数据写入失败";

                return new PeopleCreatedEvent
                {
                    Result = response
                };
            }

            <!-- 无关代码，后续再做更改 -->
            public Task DelayCreatePerson(DelayCreatePeopleCommand command)
            {
                throw new NotImplementedException();
            }

            Task<PeopleCreateEvent> IPersonService.AddPersonAsync(CreatePeopleCommand command, CancellationToken cancellationToken)
            {
                throw new NotImplementedException();
            }
        }
    }
    ```
    
    ```javascript
    using PractiseForTracy.Core.Domain;
    using PractiseForTracy.Core.Service;

    namespace PractiseForTracy.Core.People;

    public interface IPersonDataProvider : IService
    {
        Task<int> CreatAsync(Person person, CancellationToken cancellationToken);
    }
    ```
6. 然后PersonDataProvider中的CreatAsync去AddAsync。
    ```javascript
    using PractiseForTracy.Core.Data;
    using PractiseForTracy.Core.Data.PractiseForTracyDbContext;
    using PractiseForTracy.Core.Domain;

    namespace PractiseForTracy.Core.People;

    public class PersonDataProvider
    {
        private readonly PractiseForTracyDbContext _dbContext;

        public PersonDataProvider(PractiseForTracyDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        public async Task<int> CreatAsync(Person person, CancellationToken cancellationToken)
        {
            await _dbContext.People.AddAsync(person, cancellationToken).ConfigureAwait(false);

            return await _dbContext.SaveChangesAsync(cancellationToken).ConfigureAwait(false);
        }
    }
    ```

----

步骤参考查阅了:lizzie学习笔记 [link3]

[1]: https://github.com/mayuanyang/Mediator.Net
[2]: #
[link3]: https://github.com/DOGGIE4/desktop-tutorial/blob/a9eb7f41ea01b136ab329d5c543c8dc4761b269c/Mediator%20Learn.md
