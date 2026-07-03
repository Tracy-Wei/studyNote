# DbUp 数据库迁移集成学习笔记

## 一、DbUp 是什么？

DbUp 是一个 .NET 数据库迁移（Database Migration）库。它的核心思想非常简单：**把数据库表结构的变更写成 SQL 脚本文件，按顺序执行，并自动记录哪些已经执行过，下次跳过。**

类比：Git 管理的是代码版本，DbUp 管理的是数据库表结构版本。

## 二、为什么需要 DbUp？

没有迁移工具时，团队协作会遇到这些痛点：

| 场景         | 没有 DbUp                                | 有了 DbUp                       |
| ------------ | ---------------------------------------- | ------------------------------- |
| 新同事搭环境 | "把这份 SQL 导一下" — 不知道哪份是最新的 | `dotnet run`，自动建表          |
| 上线加字段   | DBA 手动执行 ALTER TABLE，容易忘         | 把 SQL 放进项目，部署时自动执行 |
| 多环境同步   | 开发/测试/生产环境的表结构不一致         | 同一套脚本，保证一致            |
| 追溯变更     | "谁改的表结构？" — 没人知道              | Git 记录了一切                  |

## 三、DbUp 的工作原理

```
┌─────────────────────────────────────────────────────┐
│                    应用启动                          │
│                      │                              │
│                      ▼                              │
│       ┌──────────────────────┐                      │
│       │ 检查 SchemaVersions 表 │ ← DbUp 自动创建    │
│       └──────────────────────┘                      │
│                      │                              │
│                      ▼                              │
│       ┌──────────────────────┐                      │
│       │ 对比脚本 vs 已执行记录 │                     │
│       └──────────────────────┘                      │
│                      │                              │
│          ┌───────────┴───────────┐                  │
│          ▼                       ▼                  │
│   ┌──────────┐            ┌──────────┐              │
│   │  新脚本  │            │  已执行  │              │
│   │  → 执行  │            │  → 跳过  │              │
│   └──────────┘            └──────────┘              │
│          │                                          │
│          ▼                                          │
│   ┌──────────────────┐                              │
│   │ 记录到 SchemaVersions │                          │
│   └──────────────────┘                              │
└─────────────────────────────────────────────────────┘
```

核心原则：**脚本文件名一旦提交就不能再改**。要变更表结构就新增一个脚本。这和 Git 的 commit 不可变是同样的道理。

## 四、集成步骤（以 MySQL + .NET 9 为例）

### 4.1 安装 NuGet 包

```bash
dotnet add package dbup-mysql
```

`dbup-mysql` 是 MySQL 适配器，它会自动引入核心库 `dbup-core`。其他数据库选对应的包：

| 数据库     | 包名              |
| ---------- | ----------------- |
| MySQL      | `dbup-mysql`      |
| SQL Server | `dbup-sqlserver`  |
| PostgreSQL | `dbup-postgresql` |
| SQLite     | `dbup-sqlite`     |

### 4.2 创建 SQL 脚本目录

```
YourProject/
└── Dbup/
    ├── Script001_CreateProductTable.sql
    ├── Script002_CreateUserTable.sql
    └── Script003_AddEmailColumn.sql  ← 以后的变更
```

命名规范：`Script{序号}_{描述}.sql`，序号决定执行顺序。

脚本内容就是标准 SQL，建议加 `IF NOT EXISTS` 做防御：

```sql
-- Script001_CreateProductTable.sql
CREATE TABLE IF NOT EXISTS `Products` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(512) NOT NULL,
    `money` DOUBLE NOT NULL,
    `RecordCreatedOn` DATETIME NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 4.3 将 SQL 文件设为嵌入资源

在 `.csproj` 中添加：

```xml
<ItemGroup>
  <EmbeddedResource Include="Dbup\*.sql" />
</ItemGroup>
```

设为嵌入资源后，SQL 脚本会编译进 DLL，部署时不需要额外拷贝文件。

### 4.4 编写迁移执行器

```csharp
using System.Reflection;
using DbUp;
using Microsoft.Extensions.Logging;

public interface IDbUpRunner
{
    void RunMigrations();
}

public class DbUpRunner : IDbUpRunner
{
    private readonly string _connectionString;
    private readonly ILogger<DbUpRunner> _logger;

    public DbUpRunner(string connectionString, ILogger<DbUpRunner> logger)
    {
        _connectionString = connectionString;
        _logger = logger;
    }

    public void RunMigrations()
    {
        _logger.LogInformation("开始执行数据库迁移...");

        var upgrader = DeployChanges.To
            .MySqlDatabase(_connectionString)           // 1. 指定数据库类型
            .WithScriptsEmbeddedInAssembly(
                typeof(DbUpRunner).Assembly)             // 2. 从嵌入资源读取脚本
            .WithTransaction()                           // 3. 单个事务执行
            .LogTo(new DbUpLogger(_logger))              // 4. 接入日志
            .Build();

        var result = upgrader.PerformUpgrade();          // 5. 执行！

        if (result.Successful)
            _logger.LogInformation("数据库迁移完成。");
        else
            _logger.LogError(result.Error, "数据库迁移失败！");
    }
}
```

关键 API 解释：

| 方法                               | 作用                                           |
| ---------------------------------- | ---------------------------------------------- |
| `DeployChanges.To.MySqlDatabase()` | 指定目标数据库                                 |
| `WithScriptsEmbeddedInAssembly()`  | 从 DLL 嵌入资源加载 .sql 文件                  |
| `WithTransaction()`                | 所有脚本在一个事务中执行（全部成功或全部回滚） |
| `WithTransactionPerScript()`       | 每个脚本单独事务（推荐生产环境）               |
| `PerformUpgrade()`                 | 执行迁移，返回结果                             |

### 4.5 注册到 DI 容器（Autofac 示例）

```csharp
// 如果是 Autofac
builder.RegisterType<DbUpRunner>()
    .As<IDbUpRunner>()
    .SingleInstance();
```

### 4.6 在应用启动时调用

```csharp
// Program.cs
var app = builder.Build();

// ✅ 在 app.Run() 之前执行迁移
var dbUpRunner = app.Services.GetRequiredService<IDbUpRunner>();
dbUpRunner.RunMigrations();

app.Run();
```

> **为什么在 `app.Run()` 之前？** 确保数据库表结构就绪后，才开始接收 HTTP 请求。

## 五、日常使用流程

```
开发新功能需要加字段
    │
    ▼
创建 Script004_AddStatusColumn.sql
    │
    ▼
写 ALTER TABLE 语句
    │
    ▼
提交到 Git + 推代码
    │
    ▼
应用启动 → DbUp 自动执行新脚本
    │
    ▼
SchemaVersions 表新增一条记录
```

## 六、SchemaVersions 表长什么样？

DbUp 自动创建和维护，你不需要手动操作：

| ScriptName                                                | Applied             |
| --------------------------------------------------------- | ------------------- |
| `WorkTime.Core.Dbup.Script001_CreateProductTable.sql`     | 2026-07-03 10:30:00 |
| `WorkTime.Core.Dbup.Script002_CreateUserAccountTable.sql` | 2026-07-03 10:30:01 |
| `WorkTime.Core.Dbup.Script003_AddStatusColumn.sql`        | 2026-07-04 09:15:00 |

每次启动时 DbUp 查询这张表，只执行不在列表中的脚本。

## 七、最佳实践

1. **脚本不可变** — 已经提交并执行过的脚本永远不要修改。改表结构 = 新增脚本。
2. **脚本要幂等** — 加 `IF NOT EXISTS`、`IF EXISTS` 等判断，防止意外重复执行。
3. **序号留间隔** — 用 Script001、Script005、Script010 而非连续编号，方便中间插入。
4. **一个脚本做一件事** — 不要在一个脚本里同时建表和改字段，便于回滚定位。
5. **禁止手动改库** — 所有 DDL 变更必须通过 DbUp 脚本，否则环境和记录不一致。
6. **生产环境用 `WithTransactionPerScript()`** — 避免一个脚本失败影响后续独立脚本。

## 八、DbUp vs EF Core Migrations

|          | DbUp                             | EF Core Migrations                   |
| -------- | -------------------------------- | ------------------------------------ |
| 迁移形式 | 手写 SQL                         | 自动生成 C# 代码                     |
| 学习成本 | 低（就是写 SQL）                 | 中（要理解 EF 模型快照）             |
| 灵活性   | 高（存储过程、索引、视图随便写） | 中（受 EF 能力限制）                 |
| 适合场景 | 喜欢手写 SQL、需要精细控制       | 重度使用 EF Core、Code First         |
| 回滚     | 自己写回滚脚本                   | `dotnet ef database update` 可以回退 |

> 两者可以共存：DbUp 管理表结构和索引，EF Core 负责日常 CRUD。
