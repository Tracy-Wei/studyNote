## C# 泛型（Generic）

- 泛型允许您编写一个可以与任何数据类型一起工作的类或方法。
- 可以通过数据类型的替代参数编写类或方法的规范。
- 当编译器遇到类的构造函数或方法的函数调用时，它会生成代码来处理指定的数据类型。

示例：

- System.Collections.Generic 包含了 C# 中的泛型集合类和接口，这些集合类在处理数据集合时提供了更好的类型安全性和性能。
  常见的泛型集合类和接口：

  1. List<T>：动态数组，可以根据需要调整大小的列表。
  2. Dictionary<TKey, TValue>：键值对集合，每个键关联一个值。
  3. Queue<T>：先进先出队列，用于实现队列数据结构。
  4. Stack<T>：后进先出堆栈，用于实现堆栈数据结构。
  5. LinkedList<T>：双向链表，允许在任何位置添加、删除元素。
  6. HashSet<T>：无序、唯一元素的集合。
  7. SortedSet<T>：有序、唯一元素的集合。
  8. Queue<T> 和 Stack<T>：分别实现队列和堆栈数据结构。

  > 可以在代码中直接使用这些集合类而不需要写完整的命名空间路径。例如，你可以这样使用 List<int> 而不必写成 System.Collections.Generic.List<int>。这种方式让代码更加简洁和可读性更高。

```javascript
using System;
using System.Collections.Generic;

namespace GenericApplication
{
  // 泛型类定义
  public class MyGenericArray<T>
  {
    private T[] array; // 声明一个私有数组
    public MyGenericArray(int size)
    {
      array = new T[size + 1];
    }
    public T getItem(int index) // 返回泛型类型
    {
      return array[index];
    }
    public void setItem(int index,T value)
    {
      array[index] = value;
    }
  }

  class Tester
  {
    static void Main(string[] args)
    {
      MyGenericArray<int> intArray = new MyGenericArray<int>(5); // 创建一个存储整数的泛型数组实例
      for (int i = 0; i < 5; i++)
      {
        intArray.setItem(i, i * 5);
      }
      for (int i = 0; i < 5; i++)
      {
        Console.Write(intArray.getItem(i) + " ");
      }
      Console.WriteLine();
      MyGenericArray<char> charArray = new MyGenericArray<char>(5);
      for (int i = 0; i < 5; i++)
      {
        charArray.setItem(i, (char)(i + 97));
      }
      for (int i = 0; i < 5; i++)
      {
        Console.Write(charArray.getItem(i) + " ");
      }
      Console.WriteLine();
      Console.ReadKey();
    }
  }
}
```

### 泛型（Generic）的特性

使用泛型是一种增强程序功能的技术，具体表现在以下几个方面：

- 它有助于您最大限度地重用代码、保护类型的安全以及提高性能。
- 您可以创建泛型集合类。.NET 框架类库在 System.Collections.Generic 命名空间中包含了一些新的泛型集合类。您- 可以使用这些泛型集合类来替代 System.Collections 中的集合类。
- 您可以创建自己的泛型接口、泛型类、泛型方法、泛型事件和泛型委托。
- 您可以对泛型类进行约束以访问特定数据类型的方法。
- 关于泛型数据类型中使用的类型的信息可在运行时通过使用反射获取。

### 泛型（Generic）方法

类型参数声明泛型方法，示例：

```javascript
using System;
using System.Collections.Generic;

namespace GenericMethodAppl
{
  class Program
  {
    static void Swap<T>(ref T lhs, ref T rhs)
    {
      T temp;
      temp = lhs;
      lhs = rhs;
      rhs = temp;
    }
    static void Main(string[] args)
    {
      int a, b;
      char c, d;
      a = 10;
      b = 20;
      c = 'I';
      d = 'V';

      Console.WriteLine("Int values before calling swap:");
      Console.WriteLine("a = {0}, b = {1}",a, b);
      Console.WriteLine("Char values before calling swap:");
      Console.WriteLine("c = {0}, d = {1}", c, d);

      Swap<int>(ref a, ref b);
      Swap<char>(ref c, ref d);

      Console.WriteLine("Int values before calling swap:");
      Console.WriteLine("a = {0}, b = {1}",a, b);
      Console.WriteLine("Char values before calling swap:");
      Console.WriteLine("c = {0}, d = {1}", c, d);
      Console.ReadKey();
    }
  }
}
```

### 泛型（Generic）委托

```javascript
delegate T NumberChanger<T>(T n);
```
