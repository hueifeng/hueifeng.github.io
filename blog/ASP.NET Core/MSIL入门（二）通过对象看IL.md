# 前言

上一篇文章主要介绍了IL的概念以及基础的示例代码，在这一篇文章中我们将通过对象调用看IL。

# 创建对象与调用方法

```csharp
class Program
{
    static void Main(string[] args)
    {
        var obj = new MyClass();
        Console.WriteLine(obj.Say());
    }
}

class MyClass
{
    private const string Str = "Hello";

    public string Say()
    {
        return Str;
    }
}
```

实例字段每次创建类型实例的时候都会进行创建，它们属于这个类型的实例，而静态字段由类型的所有实例共享，并且它会在类型加载时创建。某些静态字段（文本字段和映射字段）从不分配。加载程序只需要记录要映射的字段的位置，并在字段寻址时寻址这些位置。高级别的编译器（IL assembler不执行此操作，将其会留给我们开发人员）把实例字段和三种静态字段中的两种静态字段（非文本静态字段）的类型，很容易从字段标记中识别出引用字段的类型。
当在IL中找到标记时，JIT编译器不必深入元数据检索并检查字段的标志。此时所有IL有两组用于字段加载和存储指令。实例字段的指令为ldfld，ldflda和stfld;静态字段的指令是ldsfld，ldsflda和stsfld。尝试将静态字段指令与stance字段一起使用会导致JIT编译失败。逆向组合是可行的，但是它需要将实例指针加载到堆栈上，这当然对于静态字段是完全多余的。对静态字段使用实例字段指令的好处是，它允许以相同的方式访问静态字段和实例字段。

**Callvirt** 对象调用后期绑定方法，并且将返回值推送到计算堆栈上。

```
.field private static literal string Str = "Hello" //静态公共字段的定义 

.method private hidebysig static void
    Main(
      string[] args
    ) cil managed
  {
    .entrypoint //主函数，程序的入口
    .maxstack 1 //栈的最大深度
    .locals init (
      [0] class ConsoleApp1.MyClass obj //本地变量的定义
    )

    // [8 9 - 8 10]
    IL_0000: nop //什么都不做

    // [9 13 - 9 37]
    IL_0001: newobj       instance void ConsoleApp1.MyClass::.ctor() //新建类型为MyClass的对象，并将对象放到堆栈
    IL_0006: stloc.0 //把计算堆栈顶部的值（obj）放到调用堆栈索引0处

    // [10 13 - 10 42]
    IL_0007: ldloc.0      //把调用堆栈索引为0处的值复制到计算堆栈
    IL_0008: callvirt     instance string ConsoleApp1.MyClass::Say() //调用MyClass.Say方法
    IL_000d: call         void  [System.Console]System.Console::WriteLine(string) //调用WriteLine
    IL_0012: nop //什么都不做

    // [11 9 - 11 10]
    IL_0013: ret //return 

  } // end of method Program::Main

  .method public hidebysig specialname rtspecialname instance void
    .ctor() cil managed
  {
    .maxstack 8 //栈的最大深度

    IL_0000: ldarg.0 //将索引为 0 的参数（this）加载到计算堆栈上。
    IL_0001: call         instance void [System.Runtime]System.Object::.ctor() //从堆栈中取出对象，并调用基类(Object)的构造函数
    IL_0006: nop //什么都不做
    IL_0007: ret //return 

  } // end of method Program::.ctor
```

```csharp
class Program
{
    static void Main(string[] args)
    {
        MyClass obj = new MyClass();
        obj.Name = "Hello";
        obj.Say();
    }
}

public class MyClass
{
    public string Name { get; set; }
        
    public void Say()
    {
        Console.WriteLine(Name);
    }
}
```

```
  .class public auto ansi beforefieldinit
  ConsoleApp1.MyClass
    extends [System.Runtime]System.Object
{

  .field private string '<Name>k__BackingField' //自动生成的字段
    //自动生成获取属性值的方法，带有两个属性
    .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
      = (01 00 00 00 )
    .custom instance void [System.Diagnostics.Debug]System.Diagnostics.DebuggerBrowsableAttribute::.ctor(valuetype [System.Diagnostics.Debug]System.Diagnostics.DebuggerBrowsableState)
      = (01 00 00 00 00 00 00 00 ) // ........
      // int32(0) // 0x00000000 

  .method public hidebysig specialname instance string
    get_Name() cil managed
  {
    .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
      = (01 00 00 00 )
    .maxstack 8 //栈的最大深度

    // [17 30 - 17 34]
    IL_0000: ldarg.0      //将索引为 0 的参数（this）加载到计算堆栈上。
    IL_0001: ldfld        string ConsoleApp1.MyClass::'<Name>k__BackingField' //从堆栈取出对象，访问对象的指定字段，并把字段值存入堆栈
    IL_0006: ret //return 

  } // end of method MyClass::get_Name

  .method public hidebysig specialname instance void
    set_Name(
      string 'value'
    ) cil managed
  {
    .custom instance void [System.Runtime]System.Runtime.CompilerServices.CompilerGeneratedAttribute::.ctor()
      = (01 00 00 00 )
    .maxstack 8 //栈的最大深度

    // [17 35 - 17 39]
    IL_0000: ldarg.0      //将第一个参数（this）加载到计算堆栈上。
    IL_0001: ldarg.1      //将第二个参数加载到计算堆栈上。
    IL_0002: stfld        string ConsoleApp1.MyClass::'<Name>k__BackingField'
    IL_0007: ret //return 

  } // end of method MyClass::set_Name

  .method public hidebysig instance void
    Say() cil managed
  {
    .maxstack 8 //栈的最大深度

    // [20 9 - 20 10]
    IL_0000: nop //什么都不做

    // [21 13 - 21 37]
    IL_0001: ldarg.0      //将第一个参数（this）加载到计算堆栈上。
    IL_0002: call         instance string ConsoleApp1.MyClass::get_Name()
    IL_0007: call         void [System.Console]System.Console::WriteLine(string)
    IL_000c: nop //什么都不做

    // [22 9 - 22 10]
    IL_000d: ret

  } // end of method MyClass::Say

  .method public hidebysig specialname rtspecialname instance void
    .ctor() cil managed
  {
    .maxstack 8

    IL_0000: ldarg.0      // this
    IL_0001: call         instance void [System.Runtime]System.Object::.ctor() 
    IL_0006: nop //什么都不做
    IL_0007: ret //return 

  } // end of method MyClass::.ctor

  .property instance string Name()
  {
    .get instance string ConsoleApp1.MyClass::get_Name() //标识getter
    .set instance void ConsoleApp1.MyClass::set_Name(string) //标识setter
  } // end of property MyClass::Name
} // end of class ConsoleApp1.MyClass


.method private hidebysig static void
    Main(
      string[] args
    ) cil managed
  {
    .entrypoint //主函数，程序的入口
    .maxstack 2
    .locals init (
      [0] class ConsoleApp1.MyClass obj
    ) //本地变量定义

    // [8 9 - 8 10] 
    IL_0000: nop //什么都不做

    // [9 13 - 9 41]
    IL_0001: newobj       instance void ConsoleApp1.MyClass::.ctor()  //新建类型为MyClass的对象，并将对象放到堆栈
    IL_0006: stloc.0      //把计算堆栈顶部的值（obj）放到调用堆栈索引0处 

    // [10 13 - 10 32]
    IL_0007: ldloc.0      //把调用堆栈索引为0处的值复制到计算堆栈
    IL_0008: ldstr        "Hello" //将字符串"Hello"存入到堆栈
    IL_000d: callvirt     instance void ConsoleApp1.MyClass::set_Name(string) //从堆栈取出对象与参数，并调用对象的指定方法
    IL_0012: nop

    // [11 13 - 11 23]
    IL_0013: ldloc.0      //把调用堆栈索引为0处的值复制到计算堆栈
    IL_0014: callvirt     instance void ConsoleApp1.MyClass::Say() //调用MyClass.Say方法
    IL_0019: nop //什么都不做

    // [12 9 - 12 10]
    IL_001a: ret //return 

  } // end of method Program::Main

```
