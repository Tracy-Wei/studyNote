# .NET Handler 分类（CommandHandler / RequestHandler / EventHandler）

在采用 **CQRS（Command Query Responsibility Segregation）+ MediatR** 架构的 .NET 项目中，通常会将 Handler 按职责划分为三类：

- **CommandHandler**：处理命令（执行操作、修改数据）
- **RequestHandler**：处理查询（获取数据）
- **EventHandler**：处理事件（响应已发生的业务事件）

核心思想就是：**职责单一、业务解耦。**

---

# 一、整体流程

假设是一个考勤系统。

## 1. 用户执行签到

```text
用户点击签到
        │
        ▼
CheckInCommand
        │
        ▼
CommandHandler
        │
        ▼
数据库新增签到记录
        │
        ▼
发布 CheckInCompletedEvent
        │
        ▼
┌───────────────┬──────────────┬─────────────┬─────────────┐
│               │              │             │             │
▼               ▼              ▼             ▼             ▼
更新统计      写日志      推送通知      更新缓存      WebSocket推送
```

整个过程中：

- 用户真正调用的是 **Command**
- Command 修改数据库
- 修改成功后发布 Event
- Event 通知多个模块执行后续操作

---

## 2. 用户查询签到记录

```text
用户查看签到记录
        │
        ▼
GetAttendanceListRequest
        │
        ▼
RequestHandler
        │
        ▼
查询数据库
        │
        ▼
返回签到列表
```

整个过程不会修改任何数据。

---

# 二、CommandHandler（命令处理器）

## 定义

Command 表示：

> **我要执行一个动作。**

例如：

- CreateUserCommand
- UpdateUserCommand
- DeleteUserCommand
- LoginCommand
- CheckInCommand
- PauseWorkCommand
- ResumeWorkCommand

这些都属于业务操作。

---

## 职责

负责真正执行业务。

例如：

- 新增数据（INSERT）
- 修改数据（UPDATE）
- 删除数据（DELETE）
- 调用第三方接口
- 执行业务流程

例如：

```text
CheckInCommand
        │
        ▼
CheckInCommandHandler
        │
        ▼
验证业务
        │
        ▼
写数据库
        │
        ▼
返回成功
```

---

## 特点

✅ 修改数据

✅ 一次请求通常只有一个 Handler

✅ 可以返回执行结果

例如：

```csharp
Result
bool
Guid
UserDto
```

---

# 三、RequestHandler（查询处理器）

有些项目也叫 **QueryHandler**。

## 定义

Request（Query）表示：

> **我要获取数据。**

例如：

- GetUserRequest
- GetAttendanceListRequest
- SearchHistoryRequest
- GetOrderDetailRequest

---

## 职责

只负责查询。

例如：

```text
Request
      │
      ▼
查询数据库
      │
      ▼
返回 DTO
```

---

## 特点

✅ 不修改数据库

✅ 返回数据

✅ 一次请求通常只有一个 Handler

适用于：

- 查询列表
- 查询详情
- 搜索
- 分页
- 数据统计

---

# 四、EventHandler（事件处理器）

## 定义

Event 表示：

> **某件事情已经发生。**

注意：

不是

```text
CreateUser
```

而是

```text
UserCreated
```

说明：

用户已经创建完成。

例如：

- UserCreatedEvent
- OrderPaidEvent
- PasswordChangedEvent
- CheckInCompletedEvent

---

## Event 与 Command 的区别

Command：

```text
我要创建用户
```

Event：

```text
用户已经创建完成
```

Command 是"动作"。

Event 是"结果"。

---

## 职责

Event 不负责执行业务。

它负责通知：

> **这件事情已经完成了。**

例如：

```text
CheckInCompletedEvent
```

可以通知：

```text
CheckInCompletedEvent
        │
        ├── 写日志
        ├── 更新统计
        ├── 更新缓存
        ├── 推送消息
        ├── 发短信
        └── 发邮件
```

每一个都是独立的 EventHandler。

互相没有依赖。

---

## 特点

✅ 一个 Event 可以有多个 Handler

✅ Handler 之间互不影响

✅ 新增 Handler 不需要修改原来的代码

符合：

> 开闭原则（Open Closed Principle）

---

# 五、为什么需要 EventHandler？

假设没有 Event。

所有事情都写进 Command：

```text
CheckInCommandHandler

Handle()

├── 保存数据库
├── 发短信
├── 发邮件
├── 写日志
├── 更新缓存
├── 更新排行榜
├── 推送消息
├── 调用 ERP
└── 调用 OA
```

随着业务越来越复杂：

```text
Handle()

3000+ 行代码
```

后期维护非常困难。

如果以后新增：

- 企业微信通知
- 钉钉通知
- AI 分析
- Kafka 消息

都需要修改原来的 Handler。

这违反：

> 开闭原则（对扩展开放，对修改关闭）。

---

采用 Event 后：

```text
CommandHandler
        │
        ▼
保存数据库
        │
        ▼
Publish(Event)
        │
        ▼
多个 EventHandler 独立处理
```

以后新增：

```text
SendEmailHandler

SendSmsHandler

StatisticsHandler

CacheHandler

KafkaHandler

AuditLogHandler
```

只需要新增 Handler。

原来的代码完全不用修改。

---

# 六、三者之间的关系

```text
                  用户请求
                      │
          ┌───────────┴────────────┐
          │                        │
      修改数据                  查询数据
          │                        │
          ▼                        ▼
   CommandHandler          RequestHandler
          │
          ▼
      修改数据库
          │
          ▼
   Publish(Event)
          │
          ▼
      EventHandler
          │
 ┌────────┼────────┬────────┬────────┐
 ▼        ▼        ▼        ▼        ▼
日志     缓存     通知     统计     推送
```

---

# 七、三者对比

| 类型           | 作用         | 是否修改数据       | 是否返回数据     | 一个请求对应几个 Handler |
| -------------- | ------------ | ------------------ | ---------------- | ------------------------ |
| CommandHandler | 执行业务命令 | ✅                 | 通常返回执行结果 | 1 个                     |
| RequestHandler | 查询数据     | ❌                 | ✅               | 1 个                     |
| EventHandler   | 响应业务事件 | 可选（视业务而定） | ❌               | 0 ~ N 个                 |

---

# 八、开发中的使用原则

## 使用 CommandHandler

当你的接口需要：

- 新增
- 修改
- 删除
- 登录
- 注册
- 签到
- 支付
- 上传文件

都应该使用 Command。

例如：

```
POST   /users
PUT    /users/{id}
DELETE /users/{id}
POST   /attendance/check-in
POST   /attendance/check-out
```

---

## 使用 RequestHandler

当你的接口只是查询：

- 获取用户信息
- 获取订单列表
- 获取签到记录
- 搜索数据
- 获取统计信息

都应该使用 Request。

例如：

```
GET /users
GET /users/{id}
GET /attendance
GET /orders
```

---

## 使用 EventHandler

当某个业务完成以后：

需要通知其它模块。

例如：

用户注册成功：

```
UserCreatedEvent
```

可能需要：

- 初始化用户配置
- 发欢迎邮件
- 发短信
- 写日志
- 推送消息
- 更新缓存

支付完成：

```
OrderPaidEvent
```

可能需要：

- 扣库存
- 发货
- 开发票
- 发短信
- 更新统计

签到完成：

```
CheckInCompletedEvent
```

可能需要：

- 更新考勤统计
- 推送通知
- 写日志
- WebSocket 实时刷新

---

# 九、一句话总结

## CommandHandler

> 我要做一件事（Do Something）

负责执行操作、修改数据。

---

## RequestHandler

> 我要获取数据（Get Something）

负责查询数据，不修改任何状态。

---

## EventHandler

> 某件事已经发生（Something Happened）

负责通知其他模块执行后续业务，实现业务解耦。

---

# 十、最佳实践

一个标准的业务流程推荐如下：

```text
Controller
      │
      ▼
Mediator.Send(Command)
      │
      ▼
CommandHandler
      │
      ├── 参数校验
      ├── 业务校验
      ├── 修改数据库
      ├── 保存事务
      └── Publish(Event)
                │
                ▼
         EventHandler（多个）
                ├── 写日志
                ├── 更新缓存
                ├── 更新统计
                ├── 推送消息
                ├── 发送通知
                └── 同步第三方系统
```

查询流程则保持简单：

```text
Controller
      │
      ▼
Mediator.Send(Request)
      │
      ▼
RequestHandler
      │
      ▼
查询数据库
      │
      ▼
返回 DTO
```

**核心原则：**

- **Command：负责改变状态（Write）**
- **Request：负责读取数据（Read）**
- **Event：负责通知和扩展（Notify）**

这样可以使代码职责清晰、易于维护，并且便于后续扩展复杂业务。
