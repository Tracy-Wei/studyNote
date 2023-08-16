## c#简介，结构与语法
### c#程序包括：
1.命名空间声明（Namespace declaration）

2.一个class 

3.class属性 

4.一个main方法 

5.语句（statements）&表达式（Expressions）

6.注释

### c#程序后缀是.cs

### c#一个简单写法结构
```javascript
using System;
namespace HelloWorldApplication
{
    /* 类名为 HelloWorld */
    class HelloWorld
    {
        /* main函数 */
        static void Main(string[] args)
        {
            /* 我的第一个 C# 程序 */
            Console.WriteLine("Hello World!");
            Console.ReadKey();
        }
    }
}
```
using 关键字用于在程序中包含 System 命名空间。 一个程序一般有多个 using 语句。

namespace 声明。一个 namespace 里包含了一系列的类。

class 声明。类 HelloWorld 包含了程序使用的数据和方法声明。

定义了 Main 方法，是所有 C# 程序的 入口点。Main 方法说明当执行时 类将做什么动作。

Console.ReadKey(); 是针对 VS.NET 用户的。这使得程序会等待一个按键的动作，防止程序从 Visual Studio .NET 启动时屏幕会快速运行并关闭。


### 语法注意！
1.C# 是大小写敏感的。

2.所有的语句和表达式必须以分号（;）结尾。

3.程序的执行从 Main 方法开始 

4.与 Java 不同的是，文件名可以不同于类的名称。

### 标识符是用来识别类、变量、函数或任何其它用户定义的项目:
1.标识符必须以字母、下划线或 @ 开头，后面可以跟一系列的字母、数字（ 0 - 9 ）、下划线（ _ ）、@

2.标识符中的第一个字符不能是数字

3.标识符必须不包含任何嵌入的空格或符号，比如 ? - +! # % ^ & * ( ) [ ] { } . ; : " ' / \

4.标识符不能是 C# 关键字。除非它们有一个 @ 前缀。 例如，@if 是有效的标识符，但 if 不是，因为 if 是关键字

5.标识符必须区分大小写。大写字母和小写字母被认为是不同的字母

6.不能与C#的类库名称相同


## 数据结构
分为：1.值类型 2.引用类型 3.指针类型

### 值类型： 例如int、char、float

```javascript
using System;

namespace DataTypeApplication
{
   class Program
   {
      static void Main(string[] args)
      {
         Console.WriteLine("Size of int: {0}", sizeof(int));
         Console.ReadLine();
      }
   }
}
```

其中{0}指的是占位符，给后面的 sizeof(int)来替换值

WriteLine()属于在控制台输出

ReadLine()读取用户输入

### 引用类型：例如object、dynamic 和 string
