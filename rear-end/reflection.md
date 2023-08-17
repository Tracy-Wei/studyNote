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

使用反射（Reflection）来读取 Rectangle 类中的元数据的事例：
```javascript
using System;
using System.Reflection;
namespace BugFixApplication
{
  [AttributeUsage(AttributeTargets.Class |
  AttributeTargets.Constructor |
  AttributeTargets.Field |
  AttributeTargets.Method |
  AttributeTargets.Property,
  AllowMultiple=true)]

  public class  DeBugInfo:System.Attribute
  {
    private int bugNo;
    private string developer;
    private string lastReview;
    public string message;

    public DeBugInfo(int bg, string dev, string d)
    {
      this.bugNo = bg;
      this.developer = dev;
      this.lastReview = d;
    }

    public int BugNo
    {
         get
         {
            return bugNo;
         }
    }
    public string Developer
    {
      get
      {
        return developer;
      }
    }

    public string LastReview
    {
      get
      {
        return lastReview;
      }
    }

    public string Message
    {
      get
      {
        return message;
      }
      set 
      { 
        message = value; 
      }
    }
  }

  [DeBugInfo(45, "Zara Ali", "12/8/2012", Message = "Return type mismatch")]
  [DeBugInfo(49, "Nuha Ali", "10/10/2012", Message = "Unused variable")]
  class Rectangle{
    protected double length;
    protected double width;

    public Rectangle(double l, double w)
    {
      length = l;
      width = w;
    }

    [DeBugInfo(55, "Zara Ali", "19/10/2012", Message = "Return type mismatch")]
    public double GetArea()
    {
      return length * width;
    }
    
    [DeBugInfo(56, "Zata Ali", "19/10/2012")]
    public void Display()
    {
      Console.WriteLine("Length: {0}", length);
      Console.WriteLine("Length: {0}", width);
      Console.WriteLine("Length: {0}", GetArea());
    }

    class ExecuteRectangle
    {
      static void Main(string[] args)
      {
        Rectangle r = new Rectangle(4.5,7.5);
        r.Display();
        Type type = typeof(Rectangle);
        
        foreach (Object attributes in type.GetCustomAttributes(false))
        {
          DeBugInfo dbi = (DeBugInfo)attributes;
          if (null != dbi)
          {
            Console.WriteLine("Bug no: {0}", dbi.BugNo);
            Console.WriteLine("Developer: {0}", dbi.Developer);
            Console.WriteLine("Last Reviewed: {0}",dbi.LastReview);
            Console.WriteLine("Remarks: {0}", dbi.Message);
          }
        }

        foreach (MethodInfo m in type.GetMethods())
        {
          foreach(Attribute a in m.GetCustomAttributes(true))
          {
            DeBugInfo dbi = (DeBugInfo)a;
            if(null != dbi)
            {
              Console.WriteLine("Bug no: {0}, for Method: {1}", dbi.BugNo, m.Name);
              Console.WriteLine("Developer: {0}", dbi.Developer);
              Console.WriteLine("Last Review: {0}", dbi.LastReview);
              Console.WriteLine("Remarks: {0}", dbi.Message);
            }
          }
        }

         Console.ReadLine();
      }
    }
  }
}
```
