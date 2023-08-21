## 委托（Delegate）

C# 中的委托（Delegate）类似于 C 或 C++ 中函数的指针。

委托（Delegate） 是存有对某个方法的引用的一种引用类型变量。引用可在运行时被改变。

委托（Delegate）特别用于实现事件和回调方法。所有的委托（Delegate）都派生自 System.Delegate 类。

### 声明委托（Delegate）

例如，假设有一个委托：

```javascript
public delegate int MyDelegate (string s);
```

上面的委托可被用于引用任何一个带有一个单一的 string 参数的方法，并返回一个 int 类型变量。

声明委托的语法如下：

```javascript
delegate <return type> <delegate-name> <parameter list>
```

### 实例化委托（Delegate）

一旦声明了委托类型，委托对象必须使用 new 关键字来创建，且与一个特定的方法有关。当创建委托时，传递到 new 语句的参数就像方法调用一样书写，但是不带有参数。

```javascript
public delegate void printString(string s);
...
printString ps1 = new printString(WriteToScreen);
printString ps2 = new printString(WriteToFile);
```

演示了委托的声明、实例化和使用，该委托可用于引用带有一个整型参数的方法，并返回一个整型值。

```javascript
using System;

// 委托的声明
delegate int NumberChanger(int n);
namespace DelegateAppl
{
  class TestDelegate
  {
    static int num = 10;
    public static int AddNum(int p)
    {
      num += p;
      return num;
    }

    public static int MultNum(int q)
    {
      num *=q;
      return num;
    }

    public static int getNum()
    {
      return num;
    }

    static void Main(string[] args)
    {
      NumberChanger nc1 = new NumberChanger(AddNum);
      NumberChanger nc2 = new NumberChanger(MultNum);

      nc1(25);
      Console.WriteLine("Value of Num: {0}", getNum());
      nc2(5);
      Console.WriteLine("Value of Num: {0}", getNum());
      Console.ReadKey();
    }
  }

}
```

### 委托的多播（Multicasting of a Delegate）

委托对象可使用 "+" 运算符进行合并。一个合并委托调用它所合并的两个委托。**只有相同类型的委托** 可被合并。

"-" 运算符可用于从合并的委托中移除组件委托。

使用委托的这个有用的特点，您可以创建一个委托被调用时要调用的方法的调用列表。这被称为委托的 **多播（multicasting)** ，也叫组播。

委托的多播示例:

```javascript
using System;

delegate int NumberChanger(int n);
namespace DelegateAppl
{
  class TestDelegate
  {
  static int num = 10;
  public static int AddNum(int p)
  {
    num += p;
    return num;
  }

  public static int MultNum(int q)
  {
    num *= q;
    return num;
  }

  public static int getNum()
  {
    return num;
  }

  static void Main(string[] args)
  {
    // 创建委托实例
    NumberChanger nc; //只是声明了一个变量，并没有实际创建实例。后续的代码中将其指向一个实际的实例。
    NumberChanger nc1 = new NumberChanger(AddNum);
    NumberChanger nc2 = new NumberChanger(MultNum);
    nc = nc1;
    nc += nc2;
    // 调用多播
    nc(5);
    Console.WriteLine("Value of Num: {0}",getNum());
    Console.ReadKey();
  }
  }
}
```

### 委托（Delegate）的用途

委托 printString 可用于引用带有一个字符串作为输入的方法，并不返回任何东西。

```javascript
示例：
我们使用这个委托来调用两个方法，第一个把字符串打印到控制台，第二个把字符串打印到文件：


using System;
using System.IO; // 该命名空间包含了与输入输出操作相关的类型和类库

namespace DelegateAppl
{
  class PrintString
  {
    static FileStream fs;
    static StreamWriter sw;
    // 委托声明
    public delegate void printString(string s);

    // 该方法打印到控制台
    public static void WriteToScreen(string str)
    {
      Console.WriteLine("The String is: {0}", str);
    }

    // 该方法打印到文件
    public static void WriteToFile(string s)
    {
      fs = new FileStream("C:\\message.txt", FileMode.Append, FileAccess.Write);
      sw = new StreamWriter(fs);
      sw.WriteLine(s);
      sw.Flush(); //被调用来确保在关闭文件流之前，将所有数据都写入文件。
      sw.Close(); // 被调用以关闭文件流和相关资源
      fs.Close();
    }

    // 该方法把委托作为参数，并使用它调用方法
    public static void sendString(printString ps)
    {
      ps("Hello World");
    }

    static void Main(string[] args)
    {
      printString ps1 = new printString(WriteToScreen);
      printString ps2 = new printString(WriteToFile);
      sendString(ps1);
      sendString(ps2);
      Console.ReadKey();
    }
  }
}
```
