## C# 事件（Event）

事件（Event） 基本上说是一个用户操作，如按键、点击、鼠标移动等等，或者是一些提示信息，如系统生成的通知。应用程序需要在事件发生时响应事件。

C# 中使用事件机制实现线程间的通信。

### 通过事件使用委托

- 事件在类中声明且生成，且通过使用同一个类或其他类中的委托与事件处理程序关联。包含事件的类用于发布事件。这被称为 发布器（publisher） 类。

- 其他接受该事件的类被称为 订阅器（subscriber） 类。

- 事件使用 发布-订阅（publisher-subscriber） 模型。

- 订阅器（subscriber） 是一个接受事件并提供事件处理程序的对象。

- 在发布器（publisher）类中的委托调用订阅器（subscriber）类中的方法（事件处理程序）。

### 声明事件（Event）

```javascript
public delegate void BoilerLogHandler(string status);

// 基于上面的委托定义事件
public event BoilerLogHandler BoilerEventLog;
```

示例：

```javascript
using System;
namespace SimpleEvent
{
  /***********发布器类***********/
  public class EventTest
  {
    private int value;

    public delegate void NumManipulationHandler();

    public event NumManipulationHandler ChangeNum; // 定义事件
    protected virtual void OnNumChanged() // 事件模式中用于触发事件的标准方法。virtual可以在继承“EventTest”类的子类中被重写
    {
      if( ChangeNum != null )
      {
        ChangeNum(); // 事件被触发
      }
      else
      {
        Console.WriteLine("event not fire");
        Console.ReadKey(); // 回车继续
      }
    }

    public EventTest()
    {
      int n = 5;
      SetValue(n);
    }

    public void SetValue(int n)
    {
      if(value != n)
      {
        value = n;
        OnNumChanged();
      }
    }
  }

  /***********订阅器类***********/
  public class subscribEvent
  {
    public void printf()
    {
      Console.WriteLine("event fire");
      Console.ReadKey();
    }
  }

  public class MainClass
  {
    public static void Main()
    {
      EventTest e = new EventTest(); // 实例化对象
      subscribEvent v = new subscribEvent(); // 实例化对象
      e.ChangeNum += new EventTest.NumManipulationHandler(v.printf);  // printf订阅到ChangeNum事件中，当ChangeNum事件触发时printf方法会被调用，输出 "event fire"
      e.SetValue(7);
      e.SetValue(11);
    }
  }
}
```

示例 2：提供一个简单的用于热水锅炉系统故障排除的应用程序。当维修工程师检查锅炉时，锅炉的温度和压力会随着维修工程师的备注自动记录到日志文件中：

```javascript
using System;
using System.IO;

namespace BoilerEventAppl
{
  // boiler类
  class Boiler
  {
    private int temp;
    private int pressure;
    public Boiler(int t, int p)
    {
      temp = t;
      pressure = p;
    }

    public int getTemp()
    {
      return temp;
    }
    public int getPressure()
    {
      return pressure;
    }
  }
  // 事件发布器
  class DelegateBoilerEvent
  {
    // 定义委托类型 BoilerLogHandler，表示处理锅炉日志的方法
    public delegate void BoilerLogHandler(string status);

    // 基于上述委托定义事件 BoilerEventLog，用于记录锅炉事件日志
    public event BoilerLogHandler BoilerEventLog;

    // 记录锅炉信息的处理方法
    public void LogProcess()
    {
      string remarks = "O.K";
      Boiler b = new Boiler(100,12);
      int t = b.getTemp();
      int p = b.getPressure();
      if(t > 150 || t < 80 || p < 12 || p >15)
      {
        remarks = "Need Maintenance";
      }

      // 触发 BoilerEventLog 事件，将日志信息传递给订阅者
      OnBoilerEventLog("Logging Info:\n");
      OnBoilerEventLog("Temparature " + t + "\nPressure: " + p);
      OnBoilerEventLog("\nMessage: " + remarks);
    }

    // 触发事件的方法
    protected void OnBoilerEventLog(string message)
    {
      if(BoilerEventLog != null)
      {
        BoilerEventLog(message);
      }
    }
  }
  // 记录锅炉信息的类
  class BoilerInfoLogger
  {
    FileStream fs;
    StreamWriter sw;

    // 构造函数，初始化日志文件流
    public BoilerInfoLogger(string filename)
    {
      fs = new FileStream(filename,FileMode.Append,FileAccess.Write);
      sw = new StreamWriter(fs);
    }

    // 记录日志信息
    public void Logger(string info)
    {
      sw.WriteLine(info);
    }

    // 关闭日志文件流
    public void Close()
    {
      sw.Close();
      fs.Close();
    }
  }
  public class RecordBoilerInfo
  {
    static void Logger(string info)
    {
      Console.WriteLine(info);
    }

    static void Main(string[] args)
    {
      // 创建日志记录器对象并指定文件路径
      BoilerInfoLogger filelog = new BoilerInfoLogger("e:\\boiler.txt");

      // 创建事件发布器对象
      DelegateBoilerEvent boilerEvent = new DelegateBoilerEvent();

      // 订阅事件，将 Logger 方法和文件日志记录器绑定到 BoilerEventLog 事件
      boilerEvent.BoilerEventLog += new DelegateBoilerEvent.BoilerLogHandler(Logger);
      boilerEvent.BoilerEventLog += new DelegateBoilerEvent.BoilerLogHandler(filelog.Logger);

      // 触发事件并记录日志
      boilerEvent.LogProcess();

      Console.ReadLine();

      // 关闭文件日志记录器
      filelog.Close();
    }
  }
}
```
