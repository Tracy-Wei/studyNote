## C# 反射（Reflection）

反射指程序可以访问、检测和修改它本身状态或行为的一种能力。

程序集包含模块，而模块包含类型，类型又包含成员。反射则提供了封装程序集、模块和类型的对象。

您可以使用反射动态地创建类型的实例，将类型绑定到现有对象，或从现有对象中获取类型。然后，可以调用类型的方法或访问其字段和属性。

用途:1.运行时查看特性（attribute）信息。2.审查集合中的各种类型，以及实例化这些类型。3.延迟绑定的方法和属性（property）。4.在运行时创建新类型，然后使用这些类型执行一些任务。

### 查看元数据
System.Reflection 类的 MemberInfo 对象需要被初始化，用于发现与类相关的特性（attribute）。

```javascript
System.Reflection.MemberInfo info = typeof(MyClass);
```

事例：
```javascript
using System;

[AttributeUsage(AttributeTargets.All)]
public class HelpAttribute : System.Attribute
{
  public readonly string Url;

  public string Topic
  {
    get
    {
      return topic;
    }
    set
    {
      topic = value;
    }
  }
  public HelpAttribute(string url)
  {
    this.Url = url;
  }

  private string topic;
}

[HelpAttribute("Information on the class MyClass")]
class MyClass
{
}

namespace AttributeAppl
{
  class Program
  {
    static void Main(string[] args)
    {
      System.Reflection.MemberInfo info = typeof(MyClass);
      object[] attributes = info.GetCustomAttributes(true);
      for (int i = 0; i<attributes.Length; i++)
      {
        System.Console.WriteLine(attributes[i]);
      }
      Console.ReadKey();
    }
  }
}
```
