## 基础概念

Microsoft中间语言（MSIL），也成为通用中间语言（CIL），是一组与平台无关的指令，由特定于语言的编译器从源代码生成。MSIL是独立于平台的，因此，他可以在任何公共语言基础架构支持特定的环境上执行。

通过JIT编译器将MSIL转换为特定计算机环境的特定机器代码。这是在执行MSIL之前完成的，同样，MSIL在需求的基础上转换为机器代码，既JIT编译器根据需要而不是整个MSIL进行编译。

公共语言运行时（CLR）中的执行过程：执行过程包括创建MSIL以及通过JIT编译器将MSIL转换为机器代码，如下所示：

![](https://imgkr.cn-bj.ufileos.com/7900bd9d-8afd-408e-ba5f-01f102df6493.png)

- 在CLR的编译期间，特定于语言的编译器会将源代码转换为MSIL。此外，与MSIL一起，在编译中还会生成元数据。元数据包含诸如代码中类型的定义和签名，运行时信息等信息。
- 通过组装MSIL，可以创建一个公共语言基础结构（CLI）组装。该程序集基本上是用于安全性，部署，版本控制等已编译的代码库，它具有两种类型，进程程序集（EXE）和程序集（DLL）。
- 然后JIT编译器将Microsoft中间语言（MSIL）转换为特定于JIT编译器运行所在的计算机环境的机器代码。MSIL在需求的基础上转换器为机器代码，即JIT编译器根据需要而不是整个MSIL进行编译。
- 然后，由JIT编译器获得的机器代码由计算机的处理器执行。


##  打印字符串

```csharp
static void Main(string[] args)
{
    Console.WriteLine("HelloWord!");
}
```

**.entrypoint** 定义程序的入口点，该函数在程序启动时由.NET运行库调用

**.maxstack** 定义函数代码所用堆栈的最大深度

**.ldstr** string把一个字符串常量装入堆栈

**call** function(parameters)调用静态函数，函数的参数必须在函数调用前装入堆栈

**pop** 取出栈顶的值，当我们不需要把值存入变量时使用

**ret** 从当前方法返回，并将返回值（如果存在）从调用方的计算堆栈推送到被调用方的计算堆栈上。

```csharp
  .method private hidebysig static void
    Main(
      string[] args
    ) cil managed
  {
    .entrypoint //主函数，程序的入口
    .maxstack 8 //栈的最大深度。

    // [8 9 - 8 10] 
    IL_0000: nop //什么都不做

    // [9 13 - 9 45]
    IL_0001: ldstr        "HelloWord!" //把字符串压入堆栈
    IL_0006: call         void [System.Console]System.Console::WriteLine(string) //调用WriteLine
    IL_000b: nop //什么都不做

    // [10 9 - 10 10]
    IL_000c: ret //return

  } // end of method Program::Main
```

## 数据运算

**add** 2个值相加。命令的参数必须在调用前装入堆栈，该函数从堆栈中移除参数并把运算后的结果压入堆栈。

**sub** 2个值相减

**mul** 2个值相乘

```csharp
static void Main(string[] args)
{
    int x = 1;
    Console.WriteLine(x * 3 + 1 - 1);
}
```


```
  .method private hidebysig static void
    Main(
      string[] args
    ) cil managed
  {
    .entrypoint  //主函数，程序的入口
    .maxstack 2  //栈的最大深度
    .locals init (
      [0] int32 x,
      [1] int32 y,
      [2] int32 z //本地变量定义，定义int类型的 x、y、z
    )

    // [8 9 - 8 10]
    IL_0000: nop //什么都不做

    // [9 13 - 9 23]
    IL_0001: ldc.i4.1  //把x的值放到计算堆栈上
    IL_0002: stloc.0  //把计算堆栈顶部的值（x）放到调用堆栈索引0处

    // [10 13 - 10 23]
    IL_0003: ldc.i4.3 //把z的值放到计算堆栈上
    IL_0004: stloc.1      //把计算堆栈顶部的值（y）放到调用堆栈索引1处

    // [11 13 - 11 23]
    IL_0005: ldc.i4.1 //把x的值放到计算堆栈上
    IL_0006: stloc.2      //把计算堆栈顶部的值（z）放到调用堆栈索引2处

    // [12 13 - 12 46]
    IL_0007: ldloc.0       //把调用堆栈索引为0处的值复制到计算堆栈
    IL_0008: ldloc.1      //把调用堆栈索引为1处的值复制到计算堆栈
    IL_0009: mul //相乘
    IL_000a: ldloc.2  //把调用堆栈索引为2处的值复制到计算堆栈
    IL_000b: add //相加
    IL_000c: ldloc.2      //把调用堆栈索引为2处的值复制到计算堆栈
    IL_000d: sub //相减
    IL_000e: call         void [System.Console]System.Console::WriteLine(int32) //调用WriteLine
    IL_0013: nop //什么都不做

    // [13 9 - 13 10]
    IL_0014: ret //return

  } // end of method Program::Main
```

## Reference

https://www.geeksforgeeks.org/cil-or-msil-microsoft-intermediate-language-or-common-intermediate-language/
