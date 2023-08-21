## C# 集合（Collection）

- 集合（Collection）类是专门用于数据存储和检索的类。这些类提供了对栈（stack）、队列（queue）、列表（list）和哈希表（hash table）的支持。大多数集合类实现了相同的接口。

- 集合（Collection）类服务于不同的目的，如为元素动态分配内存，基于索引访问列表项等等。这些类创建 Object 类的对象的集合。在 C# 中，Object 类是所有数据类型的基类。

### List

List 是个强类型，很安全。其次看那个尖括号，它是 C#2.0 时加入的泛型，所以并不存在像 ArrayList。那样要拆/装箱以此造成性能浪费。

```javascript
var list=new List<int>();
```

List 通过索引分配，索引与数组一样，从 0 开始。它可以通过索引来读取值：

```javascript
var a=new List<int>();
a.Add(12);
a.Add(10);
Console.WriteLine(a[0]);
```

列表可以有相同的项，而且项是手动排序。

在改变项后，要注意项的索引会发生改变：

```javascript
var a=new List<int>();
a.Add(12);
a.Add(10);
Console.WriteLine(a[0]);
/* ------- */
a.Remove(12);
Console.WriteLine(a[0]);
```

常用的列表方法：

1. Add() 将东西加入到列表的最后。
2. Remove() 删掉项中第一个匹配你想删除的条件的项（删去第一个匹配此条件的项）。
3. Clear() 清空所有项。
4. Sort() 用系统默认的方式对项进行排序。
5. Contains() 查看某项是否存在于列表中。

示例：

```javascript
using System;
using static System.Console;
using System.Collections.Generic;
namespace HelloWorldApplication
{
  class HelloWorld
  {
    static void Main(string[] args)
    {
      var a = new List<int>();
      a.Add(2);
      a.Add(6);
      a.Add(2);
      a.Add(10);
      Console.WriteLine($"第一个数为{a[0]}");
      a.Remove(2);
      a.Sort();
      foreach (var a2 in a)
      {
        WriteLine(a2);
      }
      bool a3 = a.Contains(2);
      WriteLine(a3);
      Console.ReadKey();
    }
  }
}
```

### 字典

构造用法：

```javascript
var a=new Dictionary<TKey,TValue>();
```

首先，字典有一个键<TKey>和一个值<TValue>

- 其中键必须是唯一的，不能重复。
- 键不能是空引用

可以用键来索引，就不用索引值来索引了。

```javascript
WriteLine(a[TKey]);
```

字典常用的东西：

1. Add()：添加键和值

2. Clean()：清空字典中所有键和值

3. Count：获取字典中有多少对键和值

4. Remove() ：删掉一个键和值；

5. ContainsKey()/ContainsValue()：查看是否包含指定的键/值；

字典,堆栈，队列不能排序，如果想对字典排序就要用其它方法或集合，如 SortedDictionary<TKey,TValue>。

示例：

```javascript
using System;
using System.Collections.Generic;
namespace HelloWorldApplication
{
  class Program
  {
    static void Main(string[] args)
    {
      var a = new Dictionary<int,int>();
      a.Add(12,14);
      a.Add(0,1);
      Console.WriteLine("删去前的count: " + a.Count);
      a.Remove(0);
      Console.WriteLine(a[12]);
      Console.WriteLine(a.Count);
      Console.WriteLine(a.ContainsKey(0));
      Console.ReadKey();
    }
  }
}
```
