### [LINQ 查询][1]

查询指定要从数据源中检索的信息。 查询还可以指定在返回这些信息之前如何对其进行排序、分组和结构化。 查询存储在查询变量中，并用查询表达式进行初始化。。

所有 LINQ 查询操作都由以下三个不同的操作组成：

1. 获取数据源。
2. 创建查询。
3. 执行查询。

下图演示完整的查询操作。 在 LINQ 中，查询的执行不同于查询本身。 换句话说，仅通过创建查询变量不会检索到任何数据。
![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/853ec963-c29a-4ba4-aece-69838e80c83c)

##### 一些遇到的查询语句：

1. AA.where()：筛选满足条件的元素
2. AA.TolistAsync()：转化为列表
3. AA.First()：用于筛选找到指定条件的第一个元素
4. AA.shouldBe()：断言方法，通常用于单元测试框架中，用于比较实际值和期望值是否相等，如果相等，测试通过，如果不相等，测试失败。XXX.shouldBe(4)：预期 XXX 中有 4 个项目，也可以说是项数是否为 4，如果不是这样，测试将失败。
5. [Fact] Execute() 方法：这是一个 XUnit 测试方法，用于执行整个测试类。它使用 BDDfy 框架来运行测试方法，将测试结果输出为 BDD 风格的报告。
6. AA.ShouldBeGreaterThan 是一个断言方法，通常在测试代码中使用。这个方法用于比较两个数值，确保第一个数值大于第二个数值。
7. AA.Select(x=>x.AA）：表示你正在从集合中的每个元素 x 中选择（或投影）一个名为 AA 的属性或值，并将这些选择的值组成一个新的集合。结果是一个包含了所有 x.AA 值的集合。
8. AA.Any() 是一个 LINQ 查询操作符，用于检查集合（通常是 IEnumerable 或 IQueryable）中是否存在任何元素。它返回一个布尔值，如果集合中至少包含一个元素，则返回 true，否则返回 false。
9. AA.Distinct() 是一个 LINQ 查询操作，它用于移除采购文档号码集合中的重复项，以确保每个采购文档号码只出现一次。
10. AA.IsNullOrWhiteSpace(value)：用于检查字符串是否为 null、空字符串或仅包含空白字符。
11. SingleOrDefault：用于从集合中选择唯一一个满足指定条件的元素，或者如果没有满足条件的元素，则返回默认值或引发异常。
12. [其他...][2]

##### 延迟执行

查询变量本身只存储查询命令。 查询的实际执行将推迟到在 foreach 语句中循环访问查询变量之后进行。

```javascript
//  Query execution.
foreach (int num in numQuery)
{
    Console.Write("{0,1} ", num);
}
```

foreach 语句也是检索查询结果的地方。 例如，在上一个查询中，迭代变量 num 保存了返回的序列中的每个值（一次保存一个值）。

由于查询变量本身从不保存查询结果，因此可以根据需要随意执行查询。 例如，可以通过一个单独的应用程序持续更新数据库。 在应用程序中，可以创建一个检索最新数据的查询，并可以按某一时间间隔反复执行该查询以便每次检索不同的结果。

##### 强制立即执行

- 对一系列源元素执行聚合函数的查询必须首先循环访问这些元素。 Count、Max、Average 和 First 就属于此类查询。
- 要强制立即执行任何查询并缓存其结果，可调用 ToList 或 ToArray 方法。

[1]: https://learn.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/introduction-to-linq-queries
[2]: https://learn.microsoft.com/zh-tw/dotnet/csharp/linq/query-a-collection-of-objects
