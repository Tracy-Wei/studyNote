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

WriteLine()：属于在控制台输出

ReadLine()：用于接收来自用户的输入，只接受字符串格式的数据。

### 引用类型：例如object、dynamic 和 string

引用类型可以指向一个内存位置，如果多个变量引用相同的数据（即它们都持有相同的内存地址），么当一个变量改变了该数据的值时，其他引用相同数据的变量也会自动反映这种值的变化。

### 对象类型

对象（Object）类型 是 C# 通用类型系统（Common Type System - CTS）中所有数据类型的终极基类。

所以对象（Object）类型可以被分配任何其他类型（值类型、引用类型、预定义类型或用户自定义类型）的值。（在分配值之前，需要先进行类型转换）

装箱：当一个值类型转换为对象类型时
例如：
![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/fa4bad78-e973-448b-9e5d-7b80c384495e)

拆箱：当一个对象类型转换为值类型时

### 动态类型

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/b3c09111-200d-4006-ac4b-471248e83783)

### 字符串（String）类型
1."runoob.com"

2.@"runoob.com"

3. @"C:\Windows"
   
4.以任意换行，换行符及缩进空格都计算在字符串长度之内

### 指针类型（Pointer types）

写法：type* identifier;

## 类型转换

强制转换：double 为 int=》i = (int)d

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/97955d68-ab5d-466d-a123-8ae80e5d483b)

## 变量

```javascript
int num;
num = Convert.ToInt32(Console.ReadLine());
```
Convert.ToInt32()：把用户输入的数据转换为 int 数据类型。

### C# 中的 Lvalues 和 Rvalues

lvalue：lvalue 表达式可以出现在赋值语句的左边或右边。

rvalue：rvalue 表达式可以出现在赋值语句的右边，不能出现在赋值语句的左边。

## 常量与变量

常量写法：const <data_type> <constant_name> = value;

变量写法：<data_type> <variable_list>;比如：int I,j,k

## C#支持的访问修饰符

public：所有对象都可以访问。

private：对象本身在对象内部可以访问。

protected：只有该类对象及其子类对象可以访问。

internal：同一个程序集的对象可以访问。

protected internal：访问限于当前程序集或派生自包含类的类型。

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/356d15ad-fa92-4984-b29a-74e0f8aa4b70)

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/030fb240-1642-49a9-bc8c-8d2eb3d02ca9)

##方法

定义方法的语法：
```javascript
<Access Specifier> <Return Type> <Method Name>(Parameter List)
{
   Method Body
}
```
![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/4a5278e2-afc4-4e19-8f22-0bf7af763e83)

## 可空类型

? 单问号用于对 int、double、bool 等无法直接赋值为 null 的数据类型进行 null 的赋值，意思是这个数据类型是 Nullable（可空） 类型的。

?? 双问号用于判断一个变量在为 null 的时候返回一个指定的值。

声明一个 nullable 类型（可空类型）的语法如下：
```javascript
< data_type> ? <variable_name> = null;
```

??合并运算符：
```javascript
num3 = num1 ?? 5.34;      // num1 如果为空值则返回 5.34
```

## 数组

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/17ba9eeb-cec8-4653-9c3c-58bc4dd3e844)

### forearch循环

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/9c8d9d57-adf7-4a8b-9009-4c81721d11ce)

## 结构体

C# 中，结构体是值类型数据结构。它使得一个单一变量可以存储各种数据类型的相关数据。struct 关键字用于创建结构体。结构体是用来代表一个记录。

为了定义一个结构体，您必须使用 struct 语句。struct 语句为程序定义了一个带有多个成员的新的数据类型。

```javascript
struct Books{
    public string title
}
```

类和结构有以下几个基本的不同点：
1.类是引用类型，结构是值类型。

2.结构不支持继承。

3.结构不能声明默认的构造函数。

结构体中声明的字段无法赋予初值，类可以:

（❌）
```javascript
struct test001
{
    private int aa = 1;
}
```

（☑️）
```javascript
class test002
{
    private int aa = 1;
}
```

关于类与结构的选择：
类的对象是存储在堆空间中，结构存储在栈中。堆空间大，但访问速度较慢，栈空间小，访问速度相对更快。

结构：当我们描述一个轻量级对象的时候，结构可提高效率，成本更低。

类：在传值的时候希望传递的是对象的引用地址而不是对象的拷贝，就应该使用类了。

类的定义：类的对象由什么组成及在这个对象上可执行什么操作。对象是类的实例。构成类的方法和变量称为类的成员。

## 构造函数：

```javascript
using System;
namespace BoxApplication
{
    class Box
    {
       private double length;   // 长度
       private double breadth;  // 宽度
       private double height;   // 高度
       public void setLength( double len )
       {
            length = len;
       }

       public void setBreadth( double bre )
       {
            breadth = bre;
       }

       public void setHeight( double hei )
       {
            height = hei;
       }
       public double getVolume()
       {
           return length * breadth * height;
       }
    }
    class Boxtester
    {
        static void Main(string[] args)
        {
            Box Box1 = new Box();        // 声明 Box1，类型为 Box
            Box Box2 = new Box();        // 声明 Box2，类型为 Box
            double volume;               // 体积


            // Box1 详述
            Box1.setLength(6.0);
            Box1.setBreadth(7.0);
            Box1.setHeight(5.0);

            // Box2 详述
            Box2.setLength(12.0);
            Box2.setBreadth(13.0);
            Box2.setHeight(10.0);
       
            // Box1 的体积
            volume = Box1.getVolume();
            Console.WriteLine("Box1 的体积： {0}" ,volume);

            // Box2 的体积
            volume = Box2.getVolume();
            Console.WriteLine("Box2 的体积： {0}", volume);
           
            Console.ReadKey();
        }
    }
}
```

## 析构函数
当类的对象超出范围时执行。
类的名称前加上一个波浪形（~）作为前缀，它不返回值，也不带任何参数。
用于在结束程序（比如关闭文件、释放内存等）之前释放资源。析构函数不能继承或重载。

```javascript
     public Line()  // 构造函数
    {
         Console.WriteLine("对象已创建");
    }
     ~Line() //析构函数
```

## 静态成员：static，意味着类中只有一个该成员的实例。

静态变量用于定义常量，因为它们的值可以通过直接调用类而不需要创建类的实例来获取。

静态变量可在成员函数或类的定义外部进行初始化。你也可以在类的定义内部初始化静态变量。

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/ea70387e-cc26-44fc-9152-62f6836dd0e5)


## 继承
继承的思想实现了 属于（IS-A） 关系。

一个类可以派生自多个类或接口，这意味着它可以从多个基类或接口继承数据和函数。

C# 不支持多重继承。但是，您可以使用接口来实现多重继承。

动态多态性：C# 允许您使用关键字 abstract 创建抽象类，用于提供接口的部分类的实现。
当一个派生类继承自该抽象类时，实现即完成。

继承的示例：
![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/dfe1df4b-2f09-4a21-9349-477906ea94f9)

### 抽象类的一些规则：
1.不能创建一个抽象类的实例。

2.不能在一个抽象类外部声明一个抽象方法。

3.类定义前面放置关键字 sealed，可以将类声明为密封类。当一个类被声明为 sealed 时，它不能被继承。抽象类不能被声明为 sealed。

## 运算符重载

重载运算符是具有特殊名称的函数，是通过关键字 operator 后跟运算符的符号来定义的。与其他函数一样，重载运算符有返回类型和参数列表。

```javascript
public static Box operator+ (Box b, Box c)
{
   Box box = new Box();
   box.length = b.length + c.length;
   box.breadth = b.breadth + c.breadth;
   box.height = b.height + c.height;
   return box;
}
```
上面的函数为用户自定义的类 Box 实现了加法运算符（+）。它把两个 Box 对象的属性相加，并返回相加后的 Box 对象。

![image](https://github.com/Tracy-Wei/studyNote/assets/109784975/0433d824-6de7-41a2-96ea-d5ad8f44dc8f)

通常情况下，类需要进行类似于加法、减法、乘法、除法等运算操作时才会使用运算符重载。

目的是使代码更加易读和语义清晰。如果类表示了某种数学或逻辑概念，并且对于这些概念的运算操作有明确的语义，那么运算符重载可能会更有意义。

```javascript
public static Box operator+ (Box b, Box c)
{
   Box box = new Box();
   box.length = b.length + c.length;
   box.breadth = b.breadth + c.breadth;
   box.height = b.height + c.height;
   return box;
}
```

## C# 接口（Interface）

接口定义了所有类继承接口时应遵循的语法合同。接口定义了语法合同 "是什么" 部分，派生类定义了语法合同 "怎么做" 部分。

### 定义接口: InterfaceImplementer.cs

接口使用 interface 关键字声明，它与类的声明类似。接口声明默认是 public 的。下面是一个接口声明的实例：
```javascript
using System;

interface IParentInterface
{
    void ParentInterfaceMethod();
}

interface IMyInterface : IParentInterface
{
    // 接口成员
    void MethodToImplement();
}

class InterfaceImplementer : IMyInterface
{
    static void Main()
    {
        InterfaceImplementer iImp = new InterfaceImplementer();
        iImp.MethodToImplement();
        iImp.ParentInterfaceMethod();
    }

    public void MethodToImplement()
    {
        Console.WriteLine("MethodToImplement() called.");
    }

    public void ParentInterfaceMethod()
    {
        Console.WriteLine("ParentInterfaceMethod() called.");
    }
}
```
输出结果：
```javascript
MethodToImplement() called.
ParentInterfaceMethod() called.
```

继承接口后，我们需要实现接口的方法 MethodToImplement() , 方法名必须与接口定义的方法名一致。

语法注意：口命令以 I 字母开头，这个接口只有一个方法 MethodToImplement()，可以按照需求设置参数和返回值。

## C# 预处理器指令

预处理器指令都是以 # 开始，它不是语句，所以不以分号（;）结束。

例如：
```javascript
#define PI 
using System;
namespace PreprocessorDAppl
{
   class Program
   {
      static void Main(string[] args)
      {
         #if (PI)
            Console.WriteLine("PI is defined"); //PI不存在，则这条语句不编译
         #else
            Console.WriteLine("PI is not defined"); //PI存在，则这条语句不编译
         #endif
         Console.ReadKey();
      }
   }
}
```

## C# 异常处理

try：一个 try 块标识了一个将被激活的特定的异常的代码块。后跟一个或多个 catch 块。

catch：程序通过异常处理程序捕获异常。catch 关键字表示异常的捕获。

finally：finally 块用于执行给定的语句，不管异常是否被抛出都会执行。例如，如果您打开一个文件，不管是否出现异常文件都要被关闭。

throw：当问题出现时，程序抛出一个异常。使用 throw 关键字来完成。

C# 中的异常类主要是直接或间接地派生于 System.Exception 类。System.ApplicationException 和 System.SystemException 类是派生于 System.Exception 类的异常类。

System.ApplicationException 类支持由应用程序生成的异常。所以程序员定义的异常都应派生自该类。

System.SystemException 类是所有预定义的系统异常的基类。

```javascript
using System;
namespace ErrorHandlingApplication
{
    class DivNumbers
    {
        int result;
        DivNumbers()
        {
            result = 0;
        }
        public void division(int num1, int num2)
        {
            try
            {
                result = num1 / num2;
            }
            catch (DivideByZeroException e)
            {
                Console.WriteLine("Exception caught: {0}", e);
            }
            finally
            {
                Console.WriteLine("Result: {0}", result);
            }

        }
        static void Main(string[] args)
        {
            DivNumbers d = new DivNumbers();
            d.division(25, 0);
            Console.ReadKey();
        }
    }
}
```


