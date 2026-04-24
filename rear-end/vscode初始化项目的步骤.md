# PractiseForTracy 项目搭建指南

> Clean Architecture 分层架构 .NET 解决方案，包含 Api、Core、Message、IntegrationTests、UnitTests 五个项目。

---

## 目录

- [PractiseForTracy 项目搭建指南](#practisefortracy-项目搭建指南)
  - [目录](#目录)
  - [第一步：创建解决方案和目录结构](#第一步创建解决方案和目录结构)
  - [第二步：创建各个项目](#第二步创建各个项目)
  - [第三步：将项目加入解决方案](#第三步将项目加入解决方案)
  - [第四步：设置项目引用关系](#第四步设置项目引用关系)
  - [第五步：用 VS Code 打开](#第五步用-vs-code-打开)
  - [第六步：验证结构](#第六步验证结构)
  - [最终目录结构](#最终目录结构)
  - [补充](#补充)
    - [生成 .gitignore](#生成-gitignore)
    - [添加 EF Core（可选）](#添加-ef-core可选)
    - [项目引用关系说明](#项目引用关系说明)

---

## 第一步：创建解决方案和目录结构

```bash
mkdir PractiseForTracy && cd PractiseForTracy
dotnet new sln -n PractiseForTracy
mkdir src
```

---

## 第二步：创建各个项目

```bash
# Web API 项目
dotnet new webapi -n PractiseForTracy.Api -o src/PractiseForTracy.Api

# 核心业务层（类库）
dotnet new classlib -n PractiseForTracy.Core -o src/PractiseForTracy.Core

# 消息层（类库，用于 MQ/事件等）
dotnet new classlib -n PractiseForTracy.Message -o src/PractiseForTracy.Message

# 集成测试
dotnet new xunit -n PractiseForTracy.IntegrationTests -o src/PractiseForTracy.IntegrationTests

# 单元测试
dotnet new xunit -n PractiseForTracy.UnitTests -o src/PractiseForTracy.UnitTests
```

---

## 第三步：将项目加入解决方案

```bash
dotnet sln add src/PractiseForTracy.Api/PractiseForTracy.Api.csproj
dotnet sln add src/PractiseForTracy.Core/PractiseForTracy.Core.csproj
dotnet sln add src/PractiseForTracy.Message/PractiseForTracy.Message.csproj
dotnet sln add src/PractiseForTracy.IntegrationTests/PractiseForTracy.IntegrationTests.csproj
dotnet sln add src/PractiseForTracy.UnitTests/PractiseForTracy.UnitTests.csproj
```

---

## 第四步：设置项目引用关系

```bash
# Api 依赖 Core 和 Message
dotnet add src/PractiseForTracy.Api reference src/PractiseForTracy.Core
dotnet add src/PractiseForTracy.Api reference src/PractiseForTracy.Message

# 测试项目引用 Core（和 Api）
dotnet add src/PractiseForTracy.UnitTests reference src/PractiseForTracy.Core
dotnet add src/PractiseForTracy.IntegrationTests reference src/PractiseForTracy.Api
```

---

## 第五步：用 VS Code 打开

```bash
code .
```

---

## 第六步：验证结构

```bash
dotnet build
dotnet test
```

---

## 最终目录结构

```
PractiseForTracy/
├── src/
│   ├── PractiseForTracy.Api/
│   ├── PractiseForTracy.Core/
│   ├── PractiseForTracy.Message/
│   ├── PractiseForTracy.IntegrationTests/
│   └── PractiseForTracy.UnitTests/
├── .gitignore
├── PractiseForTracy.sln
└── README.md
```

---

## 补充

### 生成 .gitignore

```bash
dotnet new gitignore
```

### 添加 EF Core（可选）

如需使用 EF Core，在 `Core` 项目中安装相关包：

```bash
dotnet add src/PractiseForTracy.Core package Microsoft.EntityFrameworkCore
dotnet add src/PractiseForTracy.Core package Microsoft.EntityFrameworkCore.Design
# 按需选择数据库驱动，例如 SQL Server：
dotnet add src/PractiseForTracy.Core package Microsoft.EntityFrameworkCore.SqlServer
# 或 PostgreSQL：
dotnet add src/PractiseForTracy.Core package Npgsql.EntityFrameworkCore.PostgreSQL
```

### 项目引用关系说明

| 项目               | 引用              | 说明                   |
| ------------------ | ----------------- | ---------------------- |
| `Api`              | `Core`, `Message` | 入口层，依赖业务和消息 |
| `Core`             | 无                | 核心业务，不依赖其他层 |
| `Message`          | 无                | 消息/事件定义，独立    |
| `UnitTests`        | `Core`            | 对核心业务单元测试     |
| `IntegrationTests` | `Api`             | 对接口集成测试         |
