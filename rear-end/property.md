## C# 属性（Property）
属性（Property）:是类（class）、结构（structure）和接口（interface）的命名（named）成员。不会确定存储位置。它们具有可读写或计算它们值的访问器（accessors）。

域（Field）:类或结构中的成员变量或方法。

属性（Property）是域（Field）的扩展，且可使用相同的语法来访问。

它们使用 访问器（accessors） 让私有域的值可被读写或操作。

### 访问器（Accessors）
属性（Property）的访问器（accessor）包含有助于获取（读取或计算）或设置（写入）属性的可执行语句。访问器（accessor）声明可包含一个 get 访问器、一个 set 访问器，或者同时包含二者。

属性（Property）的用法：

```javascript
using System;
namespace runoob
{
  class Student
  {
    private string code = "N.A";
    private string name = "not known";
    private int age = 0;

    public string Code
    {
      get
      {
        return code;
      }
      set
      {
        code = value;
      }
    }

    public string Name
    {
      get
      {
        return name;
      }
      set
      {
        name = value;
      }
    }

    public int Age
    {
      get
      {
        return age;
      }
      set
      {
        age = value;
      }
    }

    // 覆盖了基类（一般是 object 类）中的 ToString() 方法。
    public override string ToString()
    {
      return $"Code: {Code}, Name: {Name}, Age: {Age}";
    }
  }
  class ExampleDemo
  {
    public static void Main(string[] args)
    {
      Student s = new Student();

      s.Code = "001";
      s.Name = "Zara";
      s.Age = 9;
      Console.WriteLine($"Student Info: {s}");
      s.Age++;
      Console.WriteLine($"Student Info: {s}");
      Console.ReadKey();
    }
  }
}
```

### 抽象属性（Abstract Properties）
抽象类可拥有抽象属性，这些属性应在派生类中被实现。

```javascript
using System;
namespace runoob
{
  // 抽象类
  public abstract class Person
  {
    // 抽象属性
    public abstract string Name
    {
      get; //访问器
      set;
    }

    // 抽象属性
    public abstract int Age
    {
      get;
      set;
    }
  }

  class Student: Person
  {
    private string code = "N.A";
    private string name = "not known";
    private int age = 0;

    public string Code
    {
      get
      {
        return code;
      }
      set
      {
        code = value;
      }
    }

    // 重写（覆盖）了基类中同名属性的派生类属性，因为基类 Person 中有一个抽象属性 Name
    public override string Name
    {
      get
      {
        return name;
      }
      set
      {
        name = value;
      }
    }

    public override int Age
    {
      get
      {
        return age;
      }
      set
      {
        age = value;
      }
    }

    // 覆盖了基类（一般是 object 类）中的 ToString() 方法。
    public override string ToString()
    {
      return $"Code: {Code}, Name: {Name}, Age: {Age}";
    }
  }
  class ExampleDemo
  {
    public static void Main(string[] args)
    {
      Student s = new Student();

      s.Code = "001";
      s.Name = "Zara";
      s.Age = 9;
      Console.WriteLine("Student Info: {0}", s);
      s.Age++;
      Console.WriteLine("Student Info: {0}", s);
      Console.ReadKey();
    }
  }
}
```
