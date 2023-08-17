## C# 特性（Attribute）
特性（Attribute）是用于在运行时传递程序中（比如类、方法、结构、枚举、组件等）元素的行为信息的声明性标签。您可以通过使用特性向程序添加声明性信息。一个声明性标签是通过放置在它所应用的元素前面的方括号（[ ]）来描述的。

特性（Attribute）用于添加元数据，如编译器指令和注释、描述、方法、类等其他信息。.Net 框架提供了两种类型的特性：预定义特性和自定义特性。

### 规定特性（Custom Attribute）
这是你自己定义的特性，用于为你的代码添加自定义的元数据。
 
```javascript
[attribute(positional_parameters, name_parameter = value, ...)]
element
```
### 预定义特性（Attribute）
这些特性是由 C# 和 .NET 框架预定义的，用于提供特定的行为或信息。

.Net 框架提供了三种预定义特性：AttributeUsage，Conditional，Obsolete

#### AttributeUsage
预定义特性 AttributeUsage 描述了如何使用一个自定义特性类。它规定了特性可应用到的项目的类型。

规定该特性的语法如下：
```javascript
[AttributeUsage(
   validon,   // 规定特性可被放置的语言元素
   AllowMultiple=allowmultiple,   // 为该特性的 AllowMultiple 属性（property）提供一个布尔值
   Inherited=inherited   // 为该特性的 Inherited 属性（property）提供一个布尔值。
)]
```

#### Conditional
这个预定义特性标记了一个条件方法，其执行依赖于指定的预处理标识符。

它会引起方法调用的条件编译，取决于指定的值，比如 Debug 或 Trace。例如，当调试代码时显示变量的值。

规定该特性的语法如下：
```javascript
[Conditional(
   conditionalSymbol
)]

//例如：[Conditional("DEBUG")]
```

#### Obsolete
这个预定义特性标记了不应被使用的程序实体。它可以让您通知编译器丢弃某个特定的目标元素。例如，当一个新方法被用在一个类中，但是您仍然想要保持类中的旧方法，您可以通过显示一个应该使用新方法，而不是旧方法的消息，来把它标记为 obsolete（过时的）。

规定该特性的语法如下：
```javascript
[Obsolete(
   message
)]
[Obsolete(
   message,   // 是一个字符串，描述项目为什么过时以及该替代使用什么。
   iserror   // 是一个布尔值。如果该值为 true，编译器应把该项目的使用当作一个错误。默认值是 false（编译器生成一个警告）。
)]

//例如：[Obsolete("Don't use OldMethod, use NewMethod instead", true)]
```

### 创建自定义特性（Attribute）
用于存储声明性的信息，且可在运行时被检索。

创建并使用自定义特性包含四个步骤：
1. 声明自定义特性
2. 构建自定义特性
3. 在目标程序元素上应用自定义特性
4. 通过反射访问特性

#### 声明自定义特性
一个新的自定义特性应派生自 System.Attribute 类。声明了一个名为 DeBugInfo 的自定义特性。
```javascript
// 一个自定义特性 BugFix 被赋给类及其成员
[AttributeUsage(AttributeTargets.Class |
AttributeTargets.Constructor |
AttributeTargets.Field |
AttributeTargets.Method |
AttributeTargets.Property,
AllowMultiple = true)] //这个属性表明该特性是否可以多次应用于同一个目标。

public class DeBugInfo : System.Attribute
```

#### 构建自定义特性
构建一个名为 DeBugInfo 的自定义特性，该特性将存储调试程序获得的信息。它存储下面的信息：
1. bug 的代码编号
2. 辨认该 bug 的开发人员名字
3. 最后一次审查该代码的日期
4. 一个存储了开发人员标记的字符串消息

 DeBugInfo 类将带有三个用于存储前三个信息的私有属性（private）和一个用于存储消息的公有属性（public）。所以 bug 编号、开发人员名字和审查日期将是 DeBugInfo 类的必需的定位（ positional）参数，消息将是一个可选的命名（named）参数。

每个特性必须至少有一个构造函数。必需的定位（ positional）参数应通过构造函数传递。
```javascript
// 一个自定义特性 BugFix 被赋给类及其成员
[AttributeUsage(AttributeTargets.Class |
AttributeTargets.Constructor |
AttributeTargets.Field |
AttributeTargets.Method |
AttributeTargets.Property,
AllowMultiple = true)]

// 定义一个名为 DeBugInfo 的特性，继承自 System.Attribute
public class DeBugInfo : System.Attribute
{
  // 私有字段，用于存储 Bug 号码、开发者和最后审核时间
  private int bugNo;
  private string developer;
  private string lastReview;

  // 公有字段，用于存储消息
  public string message;

  // 构造函数，用于初始化 Bug 号码、开发者和最后审核时间
  public DeBugInfo(int bg, string dev, string d)
  {
      this.bugNo = bg;
      this.developer = dev;
      this.lastReview = d;
  }

  // 公有属性 BugNo，用于获取 Bug 号码
  public int BugNo
  {
      get  // get和set，用于控制属性值的读取和设置行为。在这里，只定义了get访问器，所以该属性是只读的，不能在外部进行设置。
      {
          return bugNo;
      }
  }
  // 公有属性 Developer，用于获取开发者信息
  public string Developer
  {
      get
      {
          return developer;
      }
  }
  // 公有属性 LastReview，用于获取最后审核时间
  public string LastReview
  {
      get
      {
          return lastReview;
      }
  }
  // 公有属性 Message，用于获取或设置消息
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
```

#### 应用自定义特性
通过把特性放置在紧接着它的目标之前，来应用该特性：
```javascript
[DeBugInfo(45, "Zara Ali", "12/8/2012", Message = "Return type mismatch")]
[DeBugInfo(49, "Nuha Ali", "10/10/2012", Message = "Unused variable")]
class Rectangle
{
  // 成员变量
  protected double length;
  protected double width;

  // 构造函数
  public Rectangle(double l, double w)
  {
      length = l;
      width = w;
  }

  // 应用 DeBugInfo 特性到 GetArea 方法
  [DeBugInfo(55, "Zara Ali", "19/10/2012",
  Message = "Return type mismatch")]
  public double GetArea()
  {
      return length * width;
  }

  // 应用 DeBugInfo 特性到 Display 方法
  [DeBugInfo(56, "Zara Ali", "19/10/2012")]
  public void Display()
  {
      Console.WriteLine("Length: {0}", length);
      Console.WriteLine("Width: {0}", width);
      Console.WriteLine("Area: {0}", GetArea());
  }
}
```

结合后：
```javascript

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

  public static void Main(string[] args)
  {
      Rectangle rect = new Rectangle(5.0, 10.0);
      rect.Display();
  }
}
```
