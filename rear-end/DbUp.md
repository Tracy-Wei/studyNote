## DbUp

DbUp 可以帮我们创建数据库，通过脚本文件创建表、添加数据；并可通过新创建脚本文件升级现有数据库

每个脚本文件只会执行一次，如果需要修改表结构或添加数据，添加新 sql 脚本文件，而不要修改原文件。

配置 DbUp 后

```javascript
var upgradeEngine = DeployChanges.To.MySqlDatabase(connectionString)
  .WithScriptsEmbeddedInAssembly(typeof 对应类.Assembly)
  .WithTransaction() // 开启事务
  .LogToAutodetectedLog() // 自动跟踪日志
  .LogToConsole() // 日志打印到控制台
  .Build();
```

可以：

- 获取将要执行的脚本 ( GetScriptsToExecute())
- 获取已执行的脚本 ( GetExecutedScripts())
- 检查是否需要升级 ( IsUpgradeRequired())
- 为任何新的迁移脚本创建版本记录而不执行它们 ( MarkAsExecuted)
- 对于使开发环境与自动化环境同步很有用
- 尝试连接数据库 ( TryConnect())
- 执行数据库升级 ( PerformUpgrade())
- 日志脚本输出 ( LogScriptOutput())

一个成功的数据库管理，需要从数据库部署策略中获得一些内容：

1. 源代码控制
   您的数据库不在源代码控制中？你不值得拥有。去使用 Excel 吧。
2. 可测试性
   我希望能够编写一个集成测试来备份旧状态，执行升级到当前状态，并验证数据没有损坏。
3. 持续集成
   我希望每次签入时，这些测试都由我的构建服务器运行。我希望 CI 构建能够进行生产备份、恢复它，并每晚运行和测试任何升级。
4. 无共享数据库
   每个开发人员都应该能够在自己的计算机上拥有数据库的副本。部署该数据库（包含示例数据）应该只需单击一下即可。
5. Dogfooding 升级
   如果 Susan 对数据库进行了更改，Harry 应该能够在他自己的数据库上执行她的转换。如果他有与她不同的测试数据，他可能会发现她没有发现的错误。哈利不应该只是毁掉他的数据库并重新开始。

这样做的好处是巨大的。通过每天测试我的转换，在我自己的开发测试数据中、在我的集成测试中、在我的构建服务器上、针对生产备份，我将确信我的更改将在生产中发挥作用。

集成到数据库：
[啊熙的笔记参考][1]

1.引用相关的包

```javascript
Install-Package dbup-core
Install-Package dbup-mysql
Install-Package dbup-sqlserver
```

2.创建一个类：

```javascript
public void Run()
    {
        EnsureDatabase.For.MySqlDatabase(连接数据库的字符串);

        var upgradeEngine = DeployChanges.To.MySqlDatabase(连接数据库的字符串)
            .WithScriptsEmbeddedInAssembly(typeof(这个类).Assembly)
            .WithTransaction()  //开启事务
            .LogToAutodetectedLog()  //自动跟踪日志
            .LogToConsole()  //日志打印到控制台
            .Build();

        var result = upgradeEngine.PerformUpgrade(); //执行数据库升级

        if (!result.Successful)
            throw result.Error;
        {
            Console.ForegroundColor = ConsoleColor.Green;
            Console.WriteLine("Success!");
            Console.ResetColor();
        }
    }
```

3.把创建的初始数据库脚本放在项目中

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/83a33b1f-2f63-4691-a7ee-88a3339b4c6e)

```javascript
<ItemGroup>
  <EmbeddedResource Include="Dbup\Scripts_2023\Script0001_initial_tables.sql" />
</ItemGroup>
```

4.在 program 中调用创建的类的 run 方法

```javascript
new DbUpRunner(new ConnectionString(configuration).Value);
```

[1]: https://github.com/MonesyH/C-Sharp-learn/blob/main/DbUp.md
