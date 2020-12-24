# 第 5 章 基元类型、引用类型和值类型

本章内容：
* <a href="#5_1">编程语言的基元类型</a>
* <a href="#5_2">引用类型和值类型</a>
* <a href="#5_3">值类型的装箱和拆箱</a>
* <a href="#5_4">对象哈希码</a>
* <a href="#5_5">dynamic 基元类型</a>

本章将讨论 Microsoft .NET Framework 开发人员经常要接触的各种类型。所以开发人员都应熟悉这些类型的不同行为。我首次接触 .NET Framework 时没有完全理解基元类型、引用类型和值类型的区别，造成在代码中不知不觉引入 bug 和性能问题。通过解释类型之间的区别，希望开发人员能避免我所经历的麻烦，同时提高编码效率。

## <a name="5_1">编程语言的基元类型</a>

某些数据类型如此常用，以至于许多编译器允许代码以简化语法来操纵它们。例如，可用以下语法分配一个整数：  

```C#
System.Int32 a = new System.Int32();
```

但你肯定不愿意用这种语法声明并初始化整数，它实在是太繁琐了。幸好，包括 C# 在内的许多编译器都允许换用如下所示的语法：

```C#
int a = 0;
```

这种语法不仅增强了代码可读性，生成的 IL 代码还与使用 `System.Int32` 生成的 IL 代码完全一致。编译器直接支持的数据类型称为 **基元类型**(primitive type)。基元类型直接映射到 Framework 类库(FCL)中存在的类型。例如，C# 的 `int` 直接映射到 `System.Int32` 类型。因此，以下 4 行代码都能正确编译，并生成完全相同的 IL：

```C#
  int            a = 0;                   // 最方便的语法
  System.Int32   a = 0;                   // 方便的语法
  int            a = new int();           // 不方便的语法
  System.Int32   a = new System.Int32();  // 最不方便的语法
```

表 5-1 列出的 FCL 类型在 C# 中都有对应的基元类型。只要是符合公共语言规范(CLS)的类型，其他语言都提供了类似的基元类型。但是，不符合 CLS 的类型语言就不一定要支持了。

表 5-1 C# 基元类型与对应的 FCL 类型
|C#基元类型|FCL类型|符合CLS|说明|
|:---:|:---:|:----:|:---:|
|`sbyte`|`System.SByte`|否|有符合 8 位值|
|`byte`|`System.Byte`|是|无符号 8 位值|
|`short`|`System.Int16`|是|有符号 16 位值|
|`ushort`|`System.UInt16`|否|无符号 16 位值|
|`int`|`System.Int32`|是|有符号 32 位值|
|`uint`|`System.UInt32`|否|无符号 32 位值|
|`long`|`System.Int64`|是|有符号 64 位值|
|`ulong`|`System.UInt64`|否|无符号 64 位值|
|`char`|`System.Char`|是|16位 Unicode 字符(`char` 不像在非托管 C++ 中那样代表一个 8 位置)|
|`float`|`System.Single`|是|IEEE 32 位浮点值|
|`double`|`System.Double`|是|IEEE 64 位浮点值|
|`bool`|`System.Boolean`|是|`true`/`false` 值|
|`decimal`|`System.Decimal`|是|128 位高精度浮点值，常用于不容许舍入误差的金融计算。 128 位中， 1 位是符号，96位是值本身(*N*)，8位是比例引子(*k*)。 `decimal` 实际值是 ±*N*×10<sup>k</sup>，其中 -28<= *k* <=0。其余位没有使用|
|`string`|`System.String`|是|字符数组|
|`object`|`System.Object`|是|所有类型的基类型|
|`dynamic`|`System.Object`|是|对于 CLR， `dynamic` 和 `object` 完全一致。但 C# 编译器允许使用简单的语法让 `dynamic` 变量参与动态调度。详情参见本章最后的 5.5 节“`dynamic` 基元类型”|

从另一个角度，可认为 C# 编译器自动假定所有源代码文件都添加了以下 `using` 指令(参考第 4 章)：

```C#
using sbyte  = System.SByte;
using byte   = System.Byte;
using short  = System.Int16;
using ushort = System.UInt16;
using int    = System.Int32;
using uint   = System.UInt32;
···
```

C# 语言规范称：“从风格上说，最好是使用关键字，而不是使用完整的系统类型名称。”我不同意语言规范：我情愿使用 FCL 类型名称，完全不用基元类型名称。事实上，我希望编译器根本不提供基元类型名称，而是强迫开发人员使用 FCL 类型名称。理由如下。

1. 许多开发人员纠结于是用 `string` 还是 `String` 。由于 C# 的 `using`(一个关键字)直接映射到 `System.String`(一个 FCL 类型)，所以两者没有区别，都可使用。类似地，一些开发人员说应用程序在 32 位操作系统上运行， `int` 代表 32 位整数；在 64 位操作系统上运行， `int` 代表 64 位整数。这个说法完全错误。C# 的 `int` 始终映射到 `System.Int32`，所以不管在什么操作系统上运行，代表的都是 32 位整数。如果程序员习惯在代码中使用 `Int32`，像这样的误解就没有了。

2. C#的 `long` 映射到 `System.Int64` ，但在其他编程语言中，`long` 可能映射到 `Int16` 或 `Int32`。例如， C++/CLI 就将 `long` 视为 `Int32`。习惯于用一种语言写程序的人在看用另一种语言写程序的人在看用另一种语言写的源代码时，很容易错误理解代码意图。事实上，大多数语言甚至不将 `long` 当作关键字，根本不编译使用了它的代码。

3. FCL 的许多方法都将类型名作为方法名的一部分。例如， `BinaryReader` 类型的方法包括 `ReadBoolean`，`ReadInt32`，`ReadSingle` 等；而 `System.Convert` 类型的方法包括 `ToBoolean`，`ToInt32`，`ToSingle` 等。以下代码虽然语法没问题，但包含 `float` 的那一行显得很别扭，无法一下子判断该行的正确性：  

    ```C#
    BinaryReader br = new BinaryReaser(...);
    float val    = br.ReadSingle();    // 正确，但感觉别扭
    Single val   = br.ReadSingle();    // 正确，感觉自然
    ```

4. 平时只用 C# 的许多程序员逐渐忘了还可以用其他语言写面向 CLR 的代码，“C#主义”逐渐入侵类库代码。例如， Microsoft 的 FCL 几乎是完全用 C# 写的，FCL 团队向库中引入了像 `Array` 的 `GetLongLength` 这样的方法。该方法返回 `Int64` 值。这种值在 C# 中确实是 `long`，但在其他语言(比如 C++/CLI)中不是。另一个例子是 `System.Linq.Enumerable` 的 `LongCount` 方法。

考虑到所有这些原因，本书坚持使用 FCL 类型名称。

在许多编程语言中，以下代码都能正确编译并运行：

```C#
Int32 i = 5;   // 32 位值
Int64 l = i;   // 隐式转型为 64 位值
```

但根据第 4 章对类型转换的讨论，你或许认为上述代码无法编译。毕竟，`System.Int32` 和 `System.Int64` 是不同的类型，相互不存在派生关系。但事实上，你会欣喜地发现 C# 编译器正确编译了上述代码，运行起来也没有问题。这是为什么呢？原因是 C# 编译器非常熟悉基元类型，会在编译代码时应用自己的特殊规则。也就是说，编译器能识别常见的编程模式，并生成必要的 IL，使写好的代码能像预期的那样工作。具体地说，C# 编译器支持与类型转换、*字面值*以及操作符有关的模式。接着的几个例子将对它们进行演示。
> 即 literal ，也称为直接量或文字常量。本书将采用“字面值”这一译法。 —— 译注

首先，编译器能执行基元类型之间的隐式或显式转型，例如：

```C#
Int32 i = 5;          // 从 Int32 隐式转型为 Int32
Int64 l = i;          // 从 Int32 隐式转型为 Int64
Single s = i;         // 从 Int32 隐式转型为 Single
Byte b = (Byte) i;    // 从 Int32 显式转型为 Byte
Int16 v = (Int16) s;  // 从 Single 显式转型为 Int16
```

只有在转换“安全”的时候，C#才允许隐式转型。所谓“安全”，是指不会发生数据丢失的情况，比如从 `Int32` 转换为 `Int64`。但如果可能不安全，C# 就要求显示转型。对于数值类型，“不安全”意味着转换后可能丢失精度或数量级。例如， `Int32` 转换为 `Byte` 要求显式转型，因为大的 `Int32` 数字可能丢失精度；`Single` 转换为 `Int16` 也要求显式转型，因为 `Signle` 能表示比 `Int16` 更大数量级的数字。

注意，不同编译器可能生成不同代码来处理这些转型。例如，将值为 6.8 的 `Single` 转型为 `Int32`，有的编译器可能生成代码对其进行截断(向下取整)，最终将 6 放到一个 `Int32` 中；其他编译器则可能将结果向上取整为 7 。顺便说一句，C# 总是对结果进行截断，而不进行向上取整。要了解 C# 对基元类型进行转型时的具体规则，请参加 C# 语言规范的 “转换” 一节。

除了转型，基本类型还能写成字面值(literal)。字面值可被看成是类型本身的实例，所以可像下面为实例(123 和 456)调用实例方法：

```C#
Console.WriteLine(123.ToString() + 456.ToString());     // "123456"
```

另外，如果表达式由字面值构成，编译器在编译时就能完成表达式求值，从而增强应用程序性能:

```C#
Boolean found = false;   // 生成的代码将 found 设为 0
Int32 x = 100 + 20 + 3； // 生成的代码将 x 设为 123
String s ="a " + "bc";   // 生成的代码将 s 设为 "a bc"
```

最后，编译器知道如何和以什么顺序解析代码中的操作符(比如 `+`，`-`，`*`，`/`，`%`，`&`，`^`，`|`，`==`，`!=`，`>`，`<`，`>=`，`<=`，`<<`，`>>`，`~`，`!`，`++`，`--` 等):

```C#
Int32 x = 100;                     // 赋值操作符
Int32 y = x + 123;                 // 加和赋值操作符
Boolean lessThanFifty = (y < 50);  // 小于和赋值操作符
```

### **checked 和 unchecked 基元类型操作符**

对基元类型执行的许多算术运算都可能造成溢出：

```C#
Byte b = 100;
b = (Byte) (b + 200);  // b 现在包含 44 (或者十六进制 2C)
```

> 重要提示 执行上述算术运算时，第一步要求所有操作数都扩大为 32 位值(或者 64 位值，如果任何操作数需要超过 32 位来表示的话)。所以 b 和 200(两个都不超过 32 位)首先转换成 32 位值，然后加到一起。结果是一个 32 位值(十进制 300，或十六进制 12C)。该值在寄回变量 b 前必须转型为 `Byte` 。C# 不隐式执行这个转型操作，这正是第二行代码需要强制转型 `Byte` 的原因。

溢出大多数时候是我们不希望的。如果没有检测到这种溢出，会导致应用程序行为失常。但极少数时候(比如计算哈希值或者校验和)，这种溢出不仅可以接受，还是我们希望的。

不同语言处理溢出的方式不同。C 和 C++ 不将溢出视为错误，允许值*回滚(wrap)*;应用程序将“若无其事”地运行。相反， Microsoft Visual Basic 总是将溢出视为错误，并在检测到到溢出时抛出异常。
> 所谓"回滚"，是指一个值超过了它的类型所允许的最大值，从而”回滚“到一个非常小的、负的或者未定义的值。 wrap 是 wrap-around 的简称。——译注

CLR 提供了一些特殊的 IL 指令，允许编译器选择它认为最恰当的行为。 CLR 有一个 `add` 指令，作用是将两个值相加，但不执行溢出检查。还有一个 `add.ovf` 指令，作用也是将两个相加，但会在发生溢出时抛出 `System.OverflowException` 异常。除了用于加法运算的 IL 指令，CLR 还为减、乘和数据转换提供了类似的 IL 指令，分别是 `sub/sub.ovf,mul/mul.ovf` 和 `conv/conv.ovf`。

C# 允许程序员自己决定如何处理溢出。溢出检查默认关闭。也就是说，编译器生成 IL 代码时，将自动使用加、减、乘以及转换指令的无溢出检查版本。结果是代码能更快地运行 —— 但开发人员必须保证不发生溢出，或者代码能预见到溢出。

让 C# 编译器控制溢出的一个办法是使用 `/checked+` 编译器开关。该开关指示编译器在生成代码时，使用加、减、乘和转换指令的溢出检查版本。这样生成的代码在执行时会稍慢一些，因为 CLR 会检查这些运算，判断是否发生溢出。如果发生溢出， CLR 抛出 `OverflowException` 异常。

除了全局性地打开或关闭溢出检查，程序员还可在代码的特定区域控制溢出检查。C# 通过 `checked` 和 `unchecked` 操作符来提供这种灵活性。下面是使用了 `unchecked` 操作符的例子：

```C#
UInt32 invalid = unchecked( (UInt32) (-1));   // OK
```

下例则使用了 `checked` 操作符：

```C#
Byte b = 100;
b = checked((Byte) (b + 200));    // 抛出 OverflowException 异常
```

在这个例子中， `b` 和 `200` 首先转换成 32 位值，然后加到一起，结果是 300。然后，因为显式转型的存在， `300` 被转换成一个 `Byte`，这造成 `OverflowException` 异常。`Byte` 在 `checked` 操作符外部转型则不会发生异常：  

```C#
b = (Byte) checked(b + 200);    // b 包含 44； 不会抛出 OverflowException 异常
```

除了 `checked` 和 `unchecked` 操作符，C# 还支持 `checked` 和 `unchecked` 语句，它们造成一个块中的所有表达式都进行或不进行溢出检查：

```C#
checked {                   // 开始 checked 块
  Byte b = 100;
  b = (Byte) (b + 200);     // 该表达式会进行溢出检查
}                           // 结束 checked 块
```

事实上，如果使用了 `checked` 语句块，就可将 += 操作符用于 `Byte`，稍微简化一下代码：  

```C#
checked {             // 开始 checked 块
  Byte b = 100;
  b += 200;           // 该表达式会进行溢出检查
}                     // 结束 checked 块
```

> 重要提示 由于 `checked` 操作符和 `checked` 语句唯一的作用就是决定生成哪个版本的加、减、乘和数据转换 IL 指令，所以在 `checked` 操作符或语句中调用方法，不会对该方法造成任何影响，如下例所示：

```C#
checked {
  // 假定 SomeMethod 试图把 400 加载到一个 Byte 中
  SomeMethod(400);
  // SomeMethod 可能会、也可能不会抛出 OverflowException 异常
  // 如果 SomeMethod 使用 checked 指令编程，就可能会抛出异常
  // 但这和当前的 checked 语句无关
}
```

根据我的经验，许多计算都会产生令人吃惊的结果。这一般是由于无效的用户输入，但也可能是由于系统的某个部分返回了程序员没有预料到的值。所以我对程序员有以下建议。

1. 尽量使用有符号数值类型(比如 `Int32` 和 `Int64`)而不是无符号数值类型(比如 `UInt32` 和 `UInt64`)。这允许编译器检测更多的上溢/下溢错误。除此之外，类库多个部分(比如 `Array` 和 `String` 的 `Length` 属性)被硬编码为返回有符号的值。这样在代码中四处移动这些值时，需要进行的强制类型转换就少了。较少的强制类型转换使代码更整洁，更容易维护。除此之外，无符号数值类型不符合 CLS。

2. 写代码时，如果代码可能发生你不希望的溢出(可能是因为无效的输入，比如要求使用最终用户或客户机提供的数据)，就把这些代码放到 `checked` 块中。同时捕捉 `OverflowException`，得体地从错误中恢复。

3. 写代码时，将允许发生溢出的代码显式放到 `unchecked` 块中，比如在计算校验和时。

4. 对于没有使用 `checked` 或 `unchecked` 的任何代码，都假定你希望在发生溢出时抛出一个异常，比如在输入已知的前提下计算一些东西(比如质数)，此时的溢出应被计为 bug。

开发应用程序时，打开编译器的 `/checked+` 开关进行调试性生成。这样系统会对没有显式标记 `checked` 或 `unchecked` 的代码进行溢出检查，所以应用程序运行起来会慢一些。此时一旦发生异常，就可以轻松检测到，而且能及时修正代码中的 bug。但是，为了正式发布而生成应用程序时，应使用编译器的 `/checked-` 开关，确保代码能更快运行，不会产生溢出异常。要在 Microsoft Visual Studio 中更改 Checked 设置，请打开项目的属性页，点击”生成“标签，单击”高级“，再勾选”检查运算上溢/下溢“，如图 5-1 所示。  
![5_1](../resources/images/5_1.png)  
图 5-1 在 Visual Studio 的”高级生成设置“对话框中指定编译器是否检查溢出  

如果应用程序能容忍总是执行 `checked` 运算而带来的轻微性能损失，建议即使是为了发布而生成应用程序，也用 `/checked` 命令行开关进行编译，这样可防止应用程序在包含已损坏的数据(甚至可能是安全漏洞)的前提下继续运行。例如，通过乘法运算来计算数组索引时，相较于因为数学运算的 *”回滚“* 而访问到不正确的数组元素，抛出 `OverflowException` 异常才是更好的做法。
> 乘法运算可能产生一个较大的值，超出数组的索引范围。参见上一条关于 ”wrap“ 的注释。 —— 译注

>> 重要提示 `System.Decimal` 是非常特殊的类型。虽然许多编程语言(包括 C# 和 Visual Basic)将 `Decimal` 视为基元类型，但 CLR 不然。这意味着 CLR 没有知道如何处理 `Decimal` 值的 IL 指令。在文档中查看 `Decimal` 类型，可以看到它提供了一系列 `pulbic static` 方法，包括 `Add`，`Subtract`，`Multiply`，`Divide` 等。此外， `Decimal` 类型还为 `+`，`-`，`\*`，`/`等提供了操作重载方法。

>> 编译使用了 `Decimal` 值的程序时，编译器会生成代码来调用 `Decimal` 的成员，并通过这些成员来执行实际运算。这意味着 `Decimal` 值的处理速度慢于 CLR 基元类型的值。另外，由于没有相应的 IL 指令来处理 `Decimal` 值，所以 `checked` 和 `unchecked` 操作符、语句以及编译器开关都失去了作用。如果对 `Decimal` 值执行的运算是不安全的，肯定会抛出 `OverflowException` 异常。

>> 类似地，`System.Numerics.BigInteger` 类型也在内部使用 `UInt32` 数组来表示任意大的整数，它的值没有上限和下限。因此，对 `BigInteger` 执行的运算永远不会造成 `OverflowException` 异常。但如果值太大，没有足够多的内存来改变数组大小，对 `BigInteger` 的运算可能抛出 `OutOfMemoryException` 异常。

## <a name="5_2">5.2 引用类型和值类型</a>

CLR 支持两种类型：**引用类型**和**值类型**。虽然 FCL 的大多数类型都是引用类型，但程序员用得最多的还是值类型。引用类型总是从托管堆分配， C# 的 `new` 操作符返回对象内存地址 —— 即指向对象数据的内存地址。使用引用类型必须留意性能问题。首先要认清楚以下四个事实。

1. 内存必须从托管堆分配。
2. 堆上分配的每个对象都有一些额外成员，这些成员必须初始化。
3. 对象中的其他字节(为字段而设)总是设为零。
4. 从托管堆分配对象时，可能强制执行一次垃圾回收。

如果所有类型都是引用类型，应用程序的性能将明显下降。设想每次使用 `Int32` 值时都进行一次内存分配，性能会受到多么大的影响！为了提升简单和常用的类型的性能，CLR 提供了名为“值类型”的轻量级类型。值类型的实例一般在线程栈上分配(虽然也可作为字段嵌入引用类型的对象中)。在代表值类型实例的变量中不包含指向实例的指针。相反，变量中包含了实例本身的字段。由于变量已包含了实例的字段，所以操作实例中的字段不需要提领指针。值类型的实例不受垃圾回收器的控制。因此，值类型的使用缓解了托管堆的压力，并减少了应用程序生存期内的垃圾回收次数。

文档清楚指出哪些类型是引用类型，哪些是值类型。在文档中查看类型时，任何称为“类”的类型都是引用类型。例如，`System.Exception` 类、`System.IO.FileStream` 类以及 `System.Random` 类都是引用类型。相反，所有值类型都称为结构或枚举。例如，`System.Int32` 结构、`System.Boolean` 结构，`System.Decimal` 结构、 `System.TimeSpan`结构、`System.DayOfWeek`枚举、`System.IO.FileAttributes`枚举以及 `System.Drawing.FontStyle` 枚举都是值类型。

进一步研究文档，会发现所有结构都是抽象类型 `System.ValueType` 的直接派生类。 `System.ValueType` 本身又直接从 `System.Object` 派生。根据定义，所有值类型都必须从 `System.ValueType` 派生。所有枚举都从 `System.Enum` 抽象类型派生，后者又从 `System.ValueType` 派生。CLR 和所有编程语言都给予枚举*特殊待遇*。欲知枚举类型的详情，请参见第 15 章 “枚举类型和位标志”。
> 将视为“一等公民”，直接支持各种强大的操作。在非托管环境中，枚举就没这么“好命”了。 —— 译注

虽然不能在定义值类型时为它选择基类型，但如果愿意，值类型可实现一个或多个接口。除此之外，所有值类型都隐式密封，目的是防止将值类型用作其他引用类型或值类型的基类型。例如，无法将 `Boolean`，`Char`，`Int32`，`UInt64`，`Single`，`Double`，`Decimal`等作为基类型来定义任何新类型。
 > 重要提示 对于许多开发人员(比如非托管 C/C++ 开发人员)，最初接触引用类型和值类型时都觉得有些不解。在非托管 C/C++ 中声明类型后，使用该类型的代码会决定是在线程栈上还是在应用程序的堆中分配类型的实例。但在托管代码中，要由定义类型的开发人员决定在什么地方分配类型的实例，使用类型的人对此并无控制权。

 以下代码和图 5-2 演示了引用类型和值类型的区别：

 ```C#
 // 引用类型(因为 'class')
 class SomeRef { public Int32 x; }

 // 值类型(因为 'struct')
 struct SomeVal { public Int32 x; }

 static void ValueTypeDemo() {
   SomeRef r1 = new SomeRef();  // 在堆上分配
   SomeVal v1 = new SomeVal();  // 在栈上分配
   rl.x = 5;                    // 提领指针
   v1.x = 5;                    // 在栈上修改
   Console.WriteLine(r1.x);     // 显示 “5”
   Console.WriteLine(v1.x);     // 同样显示 “5”
   // 图 5-2 的左半部分反映了执行以上代码之后的情况

   SomeRef r2 = r1;             // 只复制引用(指针)
   SomeVal v2 = v1;             // 在栈上分配并复制成员
   rl.x = 8;                    // r1.x 和 r2.x 都会更改
   v1.x = 9;                    // v1.x 会更改， v2.x 不变
   Console.WriteLine(r1.x);     // 显示 "8"
   Console.WriteLine(r2.x);     // 显示 "8"
   Console.WriteLine(v1.x);     // 显示 "9"
   Console.WriteLine(v2.x);     // 显示 "5"
 }
 ```

![5_2](../resources/images/5_2.png)  
图 5-2 图解代码执行时的内存分配情况  

上述代码中， `SomeVal` 类型用 `struct` 声明，而不是用更常用的 `class`。在 C# 中，用 `struct` 声明的类型是值类型，用 `class` 声明的类型是引用类型。可以看出，引用类型和值类型的区别相当大。在代码中使用类型时，必须注意是引用类型还是值类型，因为这会极大地影响在代码中表达自己意图的方式。

上述代码中有这样一行：

```C#
SomeVal vl = new SomeVal();    // 在栈上分配
```

因为这行代码的写法，似乎是要在托管堆上分配一个 `SomeVal` 实例。但 C# 编译器知道 `SomeVal` 是值类型，所以会生成正确的 IL 代码，在线程栈上分配一个 `SomeVal` 实例。 C# 还会确保值类型中的所有字段都初始化为零。

上述代码还可以像下面这样写：

```C#
SomeVal v1;    // 在栈上分配 
```

这一行生成的 IL 代码也会在线程栈上分配实例，并将字段初始化为零。唯一的区别在于，如果使用 `new` 操作符，C# 会认为实例已初始化。以下代码更清楚地进行了说明：  

```C#
// 这两行代码能通过编译，因为 C# 认为
// v1 的字段已初始化为 0
SomeVal v1 = new SomeVal();
Int32 a = v1.x;

// 这两行代码不能通过编译，因为 C# 不认为
// v1 的字段已初始化为 0
SomeVal v1；
Int32 a = v1.x;   // error CS0170：使用了可能未赋值的字段 “x” 
```

设计自己的类型时，要仔细考虑类型是否应该定义成值类型而不是引用类型。值类型有时能提供更好的性能。具体地说，除非满足以下全部条件，否则不应该将类型声明为值类型。

* 类型具有基元类型的行为。也就是说，是十分简单的类型，没有成员会修改类型的任何实例字段。如果类型没有提供会更改其字段的成员，就说该类型是**不可变**(immutable)类型。事实上，对于许多值类型，我们都建议将全部字段标记为 **readonly**(详情参见第 7 章 “常量和字段”)。

* 类型不需要从其他任何类型继承。

* 类型也不派生出其他任何类型。

类型实例大小也应在考虑之列，因为实参默认以传值方式传递，造成对值类型实例中的字段进行复制，对性能造成损害。同样地，被定义为返回一个值类型的方法在返回时，实例中的字段会复制到调用者分配的内存中，对性能造成损害。所以，要将类型声明为值类型，除了要满足以上全部条件，还必须满足以下任意条件。

* 类型的实例较小(16 字节或更小)。

* 类型的实例较大(大于 16 字节)，但不作为方法实参传递，也不从方法返回。

值类型的主要优势是不作为对象在托管堆上分配。当然，与引用类型相比，值类型也存在自身的一些局限。下面列出了值类型和引用类型的一些区别。

* 值类型对象有两种表示形式： **未装箱** 和 **已装箱**，详情参见下一节。相反，引用类型总是处于已装箱形式。

* 值类型从 `System.ValueType` 派生。该类型提供了与 `System.Object` 相同的方法。但 `System.ValueType` 重写了 `Equals` 方法，能在两个对象的字段值完全匹配的前提下返回 `true`。此外， `System.ValueType` 重写了 `GetHashCode` 方法。生成哈希码时，这个重写方法所用的算法会将对象的实例字段中的值考虑在内。由于这个默认实现存在性能问题，所以定义自己的值类型时应重写 `Equals` 和 `GetHashCode` 方法，并提供它们的显式实现。 本章末尾会讨论 `Equals` 和 `GetHashCode` 方法。

* 由于不能将值类型作为基类型来定义新的值类型或者新的引用类型，所以不应在值类型中引入任何新的虚方法。所有方法都不能是是抽象的，所有方法都隐式密封(不可重写)。

* 引用类型的变量包含堆中对象的地址。引用类型的变量创建时默认初始化 `null` ，表明当前不指向有效对象。视图使用 `null` 引用类型变量会抛出 `NullReferenceException` 异常。相反，值类型的变量总是包含其基础类型的一个值，而且值类型的所有成员都初始化为 **0**。值类型变量不是指针，访问值类型不可能抛出 `NullReferenceException` 异常。CLR 确实允许为值类型添加”可空“(nullability)标识。可空类型将在第 19 章”可空值类型“详细讨论。

* 将值类型变量赋给另一个值类型变量，会执行逐字段的复制。将引用类型的变量赋给另一个引用类型的变量只复制内存地址。

* 基于上一条，两个或多个引用类型变量能引用堆中同一个对象，所以对一个变量执行的操作可能影响到另一个变量引用的对象。相反，值类型变量自成一体，对值类型变量执行的操作不可能影响另一个值类型变量。

* 由于未装箱的值类型不在堆上分配，一旦定义了该类型的一个实例的方法不再活动，为它们分配的存储就会被释放，而不是等着进行垃圾回收。

> CLR 如何控制类型中的字段布局

>>为了提高性能，CLR 能按照它所选择的任何方式排列类型的字段。例如，CLR 可以在内存中重新安排字段的顺序，将对象引用分为一组，同时正确排列和填充数据字段。但在定义类型时，针对类型的各个字段，你可以告诉 CLR 是严格按照自己制定的顺序排列，还是按照 CLR 自己认为合适的方式重新排列。

>>为了告诉 CLR 应该怎样做，要为自己定义的类或结构应用 `System.Runtime.InteropServices.StructLayoutAttribute` 特性。可向该特性的构造器传递 `LayoutKind.Auto`，让 CLR 自动排列字段；也可传递 `LayoutKind.Sequential` ，让 CLR 保持你的字段布局；也可传递 `LayoutKind.Explicit`，利用偏移量在内存中显式排列字段。如果不为自己定义的类型显式指定 `StructLayoutAttribute`，编译器会选择它自认为最好的布局。

>>注意，Microsoft C# 编译器默认为引用类型(类)选择 `LayoutKind.Auto`，为值类型(结构)选择 `LayoutKind.Sequential` 。显然，C# 编译器团队认为和非托管代码互操作时会经常用到结构。为此，字段必须保持程序员定义的顺序。然而，假如创建的值类型不与非托管代码互操作，就应该覆盖 C# 编译器的默认设定，下面是一个例子：

```C#
using System;
using System.Runtime.InteropServices;

// 让 CLR 自动排列字段以增强这个值类型的性能
[StructLayout(LayoutKind.Auto)]
internal struct SomeValType {
  private readonly Byte m_b;
  private readonly Int16 m_x;
  ...
}
```

>>`StructLayoutAttribute` 还允许显式指定每个字段的偏移量，这要求向其构造器传递 `LayoutKind.Explicit`。 然后向值类型中的每个字段都应用 `System.Runtime.InteropServices.FieldOffsetAttribute` 特性的实例，向该特性的构造器传递 `Int32` 值来指出字段第一个字节距离实例起始处的偏移量(以字节为单位)。显式布局常用于模拟非托管 C/C++ 中的 *`union`*，因为多个字段可起始于内存的相同偏移位置。下面是一个例子：

>>> union 是特殊的类，union 中的数据成员在内存中的存储相互重叠。每个数据成员都从相同内存地址开始。分配给 union 的存储区数量是包含它最大数据成员所需的内存数。同一时刻只有一个成员可以被赋值。 —— 译注

```C#
using System;
using System.Runtime.InteropServices;

// 开发人员显式排列这个值类型的字段
[StructLayout(LayoutKind.Explicit)]
internal struct SomeValType {
  [FiledOffset(0)]
  private readonly Byte m_b;     // m_b 和 m_x 字段在该类型的实例中相互重叠

  [FiledOffset(0)]
  private readonly Int16 m_x;    // m_b 和 m_x 字段在该类型的实例中相互重叠
  ...
}
```

>> 注意在类型中，一个引用类型和一个值类型相互重叠是不合法的。虽然允许多个引用类型在同一个起始偏移位置相互重叠，但这无法验证(unverifiable)。定义类型，在其中让多个值类型相互重叠则是合法的。但是，为了使这样的类型能够验证(verifiable)，所有重叠字节都必须能通过公共字段访问。

## <a name="5_3">5.3 值类型的装箱和拆箱</a>

值类型比引用类型“轻”，原因是它们不作为对象在托管堆中分配，不被垃圾回收，也不通过指针进行引用。但许多时候都需要获取对值类型实例的引用。例如，假定要创建 `ArrayList` (`System.Collections` 命名空间中定义的一个类型)对象来容纳一组 `Point` 结构，代码如下：

```C#
// 声明值类型
struct Point {
  public Int32 x, y;
}

public sealed class Program {
  public static void Main() {
    ArrayList a = new ArrayList();
    Point p;                          // 分配一个 Point (不在堆中分配)
    for (Int32 i = 0; i < 10; i++) {
      p.x = p.y = i;                  // 初始化值类型中成员
      a.Add(p);                       // 对值类型装箱，将引用添加到 ArrayList 中
    }
    ...
  }
}
```

每次循环迭代都初始化一个 `Point` 的值类型字段，并将该 `Point` 存储到 `ArrayList` 中。但思考一下 `ArrayList` 中究竟存储了什么？是 `Point` 结构， `Point` 结构的地址，还是其他完全不同的东西？要知道正确答案，必须研究 `ArrayList` 的 `Add` 方法，了解它的参数被定义成什么类型。本例的 `Add` 方法原型如下：

```C#
public virtual Int32 Add(Object value);
```

可以看出 `Add` 获取的是一个 `Object` `参数，也就是说，Add` 获取对托管堆上的一个对象的引用(或指针)来作为参数。但之前的代码传递的是 `p`，也就是一个 `Point`，是值类型。为了使代码正确工作， `Point` 值类型必须转换成真正的、在堆中托管的对象，而且必须获取对该对象的引用。

将值类型转换成引用类型要使用**装箱**机制。下面总结了对值类型的实例进行装箱时所发生的事情。

1. 在托管堆中分配内存。分配的内存量是值类型各字段所需的内存量，还要加上托管堆所有对象都有的两个额外成员(类型对象指针和同步块索引)所需的内存量。

2. 值类型的字段复制到新分配的堆内存。

3. 返回对象地址。现在该地址是对象引用；值类型成了引用类型。

C# 编译器自动生成对值类型实例进行装箱所需的 IL 代码，但仍需理解内部发生的事情，对代码长度和性能心中有数。  
C# 编译器自动生成对值类型实例进行装箱所需的 IL 代码，但你仍然需要理解内部的工作机制才能体会到代码的大小和性能问题。

C# 编译器检测到上述代码是向要求引用类型的方法传递值类型，所以自动生成代码对对象进行装箱。所以在运行时，当前存在于 `Point` 值类型实例 `p` 中的字段复制到新分配的 `Point` 对象中。已装箱 `Point` 对象(现在是引用类型)的地址返回并传给 `Add` 方法。 `Point` 对象一直存在于堆中，直至被垃圾回收。 `Point` 值类型变量 `p` 可被重用，因为 `ArrayList`  不知道关于它的任何事情。在这种情况下，已装箱值类型的生存期超过了未装箱值类型的生存期。
> 注意 FCL 现在包含一组新的泛型集合类，非泛型集合类已成为“昨日黄花”。例如，应该使用 `System.Collections.Generic.List<T>` 类而不是 `System.Collections.ArrayList` 类。泛型集合类对非泛型集合类进行了大量改进。例如， API 得到简化和增强，集合类的性能也得到显著提升。但最大的改进就是泛型集合类允许开发人员在操作值类型的集合时不需要对集合中的项进行装箱/拆箱。单这一项改进，就使性能提升了不少。这是因为托管堆中需要创建的对象减少了，进而减少了应用程序需要执行的垃圾回收的次数。另外，开发人员还获得了编译时的类型安全性，源代码也因为强制类型转换的次数减少而变得更清晰。所有这一切都将在第 12 章 “泛型” 详细解释。

知道装箱如何进行后，接着谈谈拆箱。假定要用以下代码获取 `ArrayList` 的第一个元素：

```C#
Point p = (Point) a[0];
```

它获取 `ArrayList` 的元素0包含的引用(或指针)，试图将其放到 `Point` 值类型的实例 `p` 中。为此，已装箱 `Point` 对象中的所有字段都必须复制到值类型变量 `p` 中，后者在线程栈上。 CLR 分两步完成复制。第一步获取已装箱 `Point` 对象中的各个 `Point` 字段的地址。这个过程称为**拆箱**(unboxing)。第二步将字段包含的值从堆复制到基于栈的值类型实例中。

拆箱不是直接将装箱过程倒过来。拆箱的代价比装箱低得多。拆箱其实就是获取指针的过程，该指针指向包含在一个对象中的原始值类型(数据字段)。其实，指针指向的是已装箱实例中的未装箱部分。所以和装箱不同，拆箱不要求在内存中复制任何字节。知道这个重要区别之后，还应知道一个重点是，往往紧接着土拆箱发生一次字段复制。

装箱和拆箱/复制显然会对应用程序的速度和内存消耗产生不利影响，所以应留意编译器在什么时候生成代码来自动进行这些操作。并尝试手动编写代码，尽量减少这种情况的发生。

已装箱值类型实例在拆箱时，内部发生下面这些事情。

1. 如果包含“对已装箱值类型实例的引用”的变量为 `null`，抛出 `NullReferenceException` 异常。
2. 如果引用的对象不是所需值类型的已装箱实例，抛出 `InvalidCastException` 异常。

> CLR 还允许将值类型拆箱为相同值类型的可空版本。详情将在第 19 章讨论。

第二条意味着以下代码的工作方式和你想的可能不一样：

```C#
public static void Main() {
  Int32 x = 5;
  Object o = x;         // 对 x 装箱， o 引用已装箱对象
  Int16 y = (Int16) o;  // 抛出 InvalidCastException 异常  :System.InvalidCastException: 'Specified cast is not valid.'
}
```

从逻辑上说，完全能获取 `o` 引用的已装箱 `Int32`，将其强制转型为 `Int16` 。但在对于对象进行拆箱时，只能转型为最初未装箱的值类型——本例是 `Int32` 。以下是上述代码的正确写法：

```C#
public static void Main() {
  Int32 x = 5;
  Object o = x;                // 对 x 装箱， o 引用已装箱对象
  Int16 y = (Int16)(Int32)o;   // 先拆箱为正确类型，再转型
}
```

前面说过，一次拆箱操作经常紧接着一次字段复制。以下 C# 代码演示了拆箱和复制：

```C#
public static void Main() {
  Point p;
  p.x = p.y = 1;
  Object o = p;         // 对 p 装箱：o 引用已装箱实例  

  p = (Point) o;        // 对 o 拆箱，将字段从已装箱实例复制到栈变量中
}
```

最后一行，C# 编译器生成一条 IL 指令对 `o` 拆箱(获取已装箱实例中的字段的地址)，并生成另一条 IL 指令将这些字段从堆复制到基于栈的变量 `p` 中。

再来看看以下代码：

```C#
public static void Main() {
    Point p;
    p.x = p.y = 1;
    Object o = p;   // 对 p 装箱：o 引用已装箱实例

    // 将 Point 的 x 字段变成 2
    p = (Point)o;   // 对 o 拆箱，将字段从已装箱的实例复制到栈变量中
    p.x = 2;        // 更改栈变量的状态
    o = p;          // 对 p 装箱：o 引用新的已装箱实例
}
```

最后三行代码唯一的目的就是将 `Point` 的 `x` 字段从 **1** 变成 **2**。为此，首先要执行一次拆箱，再执行一次字段复制，再更改字段(在栈上)，最后执行一次装箱(在托管堆上创建全新的已装箱实例)。想必你已体会到了装箱和拆箱/复制对应用程序性能的影响。

有的语言(比如 C++/CLI)允许在不复制字段的前提下对已装箱的值类型进行拆箱。拆箱返回已装箱对象中的未装箱部分的地址(忽略对象的“类型对象指针”和“同步块索引”这两个额外的成员)。接着可利用这个指针来操纵未装箱实例的字段(这些字段恰好在堆上的已装箱对象中)。例如，上述代码用 C++/CLI 来写，效率会高很多，因为可直接在已装箱 `Point` 实例中修改 `Point` 的 `x` 字段的值。这就避免了在堆上分配新对象和复制所有字段两次！

> 重要提示 如果关心应用程序的性能，就应清楚编译器何时生成代码执行这些操作。遗憾的是，许多编译器都隐式生成代码来装箱对象，所以有时并不知道自己的代码会造成装箱。如果关心特定算法的性能，可用 ILDasm.exe 这样的工具查看方法的 IL 代码，观察 IL 指令 box 都在哪些地方出现。

再来看几个装箱和拆箱的例子：

```C#
public static void Main() {
  Int32 v = 5;       // 创建未装箱值类型变量
  Object o = v;      // o 引用已装箱的、包含值 5 的 Int32
  v = 123;           // 将未装箱的值修改成 123

  Console.WriteLine(v + "," + (Int32)o);     // 显示 "123,5"
}
```

能从上述代码中看出发生了多少次装箱吗？如果说 3 次，会不会觉得意外？让我们仔细分析一下代码，理解具体发生的事情。为了帮助理解，下面列出为这个 `Main` 方法生成的 IL 代码。我为这些代码加上了注释，方便你看清楚发生的每个操作：

```C#
.method public hidebysig static void  Main() cil managed
{
  .entrypoint
  // 代码大小       45 (0x2d)
  .maxstack  3
  .locals init ([0] int32 v,
           [1] object o)
  // 将 5 加载到 v 中
  IL_0000:  ldc.i4.5
  IL_0001:  stloc.0

  // 将 v 装箱，将引用指针存储到 o 中
  IL_0002:  ldloc.0
  IL_0003:  box        [mscorlib]System.Int32
  IL_0008:  stloc.1

  // 将 123 加载到 v 中
  IL_0009:  ldc.i4.s   123
  IL_000b:  stloc.0

  // 对 v 装箱，将指针保留在栈上以进行 Concat (连接)操作
  IL_000c:  ldloca.s   v
  IL_000d:  box       [mscorlib]System.Int32

  // 将字符串加载到栈上以执行 Concat 操作
  IL_0012:  ldstr      ","

  // 对 o 拆箱：获取一个指针，它指向栈上的 Int32 字段
  IL_0017:  ldloc.1
  IL_0018:  unbox.any  [mscorlib]System.Int32

  // 对 Int32 装箱，将指针保留在栈上以进行 Concat 操作
  IL_001d:  box       [mscorlib]System.Int32

  // 调用 Concat
  IL_0022:  call       string [mscorlib]System.String::Concat(object,
                                                              object,
                                                              object)

  // 将从 Concat 返回的字符串传给 WriteLine
  IL_0027:  call       void [mscorlib]System.Console::WriteLine(string)
  
  // 从 Main 返回，终止应用程序
  IL_002c:  ret
} // end of method App::Main 
```

首先在栈上创建一个 `Int32` 未装箱值类型实例( `v` )，将其初始化为 `5` 。再创建 `Object` 类型的变量( `o` )并初始化，让它指向 v 。但由于引用类型的变量始终指向堆中的对象，所以 C# 生成正确的 IL 代码对 `v` 进行装箱，将 `v` 的已装箱拷贝的地址存储到 `o` 中。接着，值 `123` 被放到未装箱值类型实例 `v` 中，但这个操作不会影响已装箱的 `Int32` ，后者的值依然为 `5`。

接着调用 `WriteLine` 方法， `WriteLine` 要求获取一个 `String` 对象，但当前没有 `String` 对象。相反，现在有三个数据项：一个未装箱的 `Int32` 值类型实例( `v` )，一个 `String` (它是引用类型)，以及对已装箱 `Int32` 值类型实例的引用( `o` )，它要转型为未装箱的 `Int32` 。必须以某种方法合并这些数据项来创建一个 `String` 。

为了创建一个 `String` ，C# 编译器生成代码来调用 `String` 的静态方法 `Concat` 。 该方法有几个重载版本，所有版本执行的操作都一样，只是参数的数量不同。由于需要连接三个数据项来创建字符串，所以编译器选择 `Concat` 方法的以下版本：

```C#
public static String Concat(Object arg0, Object arg1, Object arg2);
```

为第一个参数 `arg0` 传递的是 `v`。但 `v` 是未装箱的值参数，而 `arg0` 是 `Object`，所以必须对 `v` 进行装箱，并将已装箱的 `v` 的地址传给 `arg0` 。 对于 `arg1` 参数，字符串`“,”`作为 `String` 对象引用传递。对于 `arg2` 参数， `o` (一个 `Object` 引用)会转型为 `Int32` 。这要求执行拆箱(但不紧接着执行复制)，从而获取包含在已装箱 `Int32` 中的未装箱 `Int32` 的地址。这个未装箱的 `Int32` 实例必须再次装箱，并将新的已装箱实例的内存地址传给 `Concat` 的 `arg2` 参数。

`Concat` 方法调用指定的每个对象的 `ToString` 方法，将每个对象的字符串形式连接起来。从 `Concat` 返回的 `String` 对象传给 `WriteLine` 方法以显示最终结果。

应该指出，如果像下面这样写 `WriteLine` 调用，生成的 IL 代码将具有更高的执行效率：

```c#
Console.WriteLine(v + "," + o);     // 显示 "123,5"
```

这和前面的版本几乎完全一致，只是移除了变量 `o` 之前的( `Int32` )强制转型。之所以效率更高，是因为 `o` 已经是指向一个 `Object` 的引用类型，它的地址可直接传给 `Concat` 方法。所以，移除强制转型避免了两次操作：一次拆箱和一次装箱。不妨重新生成应用程序，观察 IL 代码来体会避免的额外操作：

```C#
.method public hidebysig static void  Main() cil managed
{
  .entrypoint
  // 代码大小       35 (0x23)
  .maxstack  3
  .locals init ([0] int32 v,
           [1] object o)
  // 将 5 加载到 v 中
  IL_0000:  ldc.i4.5
  IL_0001:  stloc.0

  // 将 v 装箱，将引用指针存储到 o 中
  IL_0002:  ldloc.0
  IL_0003:  box        [mscorlib]System.Int32
  IL_0008:  stloc.1

  // 将 123 加载到 v 中
  IL_0009:  ldc.i4.s   123
  IL_000b:  stloc.0

  // 对 v 装箱，将指针保留在栈上以进行 Concat (连接)操作
  IL_000c:  ldloca.s   v
  IL_000d:  box       [mscorlib]System.Int32

  // 将字符串加载到栈上以执行 Concat 操作
  IL_0012:  ldstr      ","

  // 将已装箱 Int32 的地址加载到栈上以执行 Concat 操作
  IL_0017:  ldloc.1

  // 调用 Concat
  IL_0018:  call       string [mscorlib]System.String::Concat(object,
                                                              object,
                                                              object)

  // 将从 Concat 返回的字符串传给 WriteLine
  IL_001d:  call       void [mscorlib]System.Console::WriteLine(string)
  
  // 从 Main 返回，终止应用程序
  IL_0022:  ret
} // end of method App::Main 
```

简单对比一下两个版本的 `Main` 方法的 IL 代码，会发现没有( `Int32` )转型的版本比有转型的版本小了 10 字节。第一个版本额外的拆箱/装箱步骤显然会生成更多的代码。更大的问题是，额外的装箱步骤会从托管堆中分配一个额外的对象，将来必须对其进行垃圾回收。这两个版本的结果一样，速度上的差别也并不明显。但是，假如在循环中发生额外的、不必要的装箱操作，就会严重影响应用程序的性能和内存消耗。

甚至可以这样调用 `WriteLine` ，进一步提升上述代码的性能：

```C#
 Console.WriteLine(v.ToString() + "," + (Int32)o);     // 显示 "123,5"
```

这会为未装箱的值类型实例 `v` 调用 `ToString` 方法，它返回一个 `String` 。 `String` 对象已经是引用类型，所以能直接传给 `Concat` 方法，不需要任何装箱操作。

下面是演示装箱和拆箱的另一个例子：

```C#
public static void Main() {
    Int32 v = 5;              // 创建未装箱的值类型变量
    Object o = v;             // o 引用 v 的已装箱版本

    v = 123;                  // 将未装箱的值类型修改成 123
    Console.WriteLine(v);     // 显示 “123”
    v = (Int32)o;             // 拆箱并将 o 复制到 v
    Console.WriteLine(v);     // 显示 “5”
}
```

上述代码发生了多少次装箱？答案是一次。之所以只发生一次装箱，是因为 `System.Console` 类已定义了获取单个 `Int32` 参数的 `WriteLine` 方法：

```C#
public static void WriteLine(Int32 value);
```

在前面对 `WriteLine` 的两次调用中，变量 `v` ( `Int32` 未装箱值类型实例)以传值方式传给方法。虽然 `WriteLine` 方法也许会在它自己内部对 `Int32` 装箱，但这已经不在我们的控制范围之内了。最重要的是，我们已尽可能地在*自己*的代码中减少了装箱。

仔细研究一下 FCL，会发现许多方法都针对不同的值类型参数进行了重载。例如， `System.Console` 类型提供了 `WriteLine` 方法的几个重载版本：

```C#
public static void WriteLine(Boolean);
public static void WriteLine(Char);
public static void WriteLine(Char[]);
public static void WriteLine(Int32);
public static void WriteLine(UInt32);
public static void WriteLine(Int64);
public static void WriteLine(UInt64);
public static void WriteLine(Single);
public static void WriteLine(Double);
public static void WriteLine(Decimal);
public static void WriteLine(Object);
public static void WriteLine(String);
```

以下几个方法也有一组类似的重载版本：`System.Console` 的 `Write` 方法， `System.IO.BinaryWriter` 的 `Write` 方法， `System.IO.TextWriter` 的 `Write` 和 `WriteLine` 方法， `System.Runtime.Serialization.SerializationInfo` 的 `AddValue` 方法， `System.Text.StringBuilder` 的 `Append` 和 `Insert` 方法。大多数方法进行重载唯一的目的就是减少常用类型的装箱次数。

但这些 FCL 类的方法不可能接受你自己定义的值类型。另外，即使是 FCL 中定义好的值类型，这些方法也可能没有提供对应的重载版本。调用方法并传递值类型时，如果不存在与值类型对应的重载版本，那么调用的肯定是获取一个 `Object` 参数的重载版本。将值类型实例作为 `Object` 传递会造成装箱，从而对性能造成不利影响。定义自己的类时，可将类中的方法定义为泛型(通过类型约束将类型参数限制为值类型)。这样方法就可获取任何值类型而不必装箱。泛型主题将在第 12 章讨论。

关于装箱最后注意一点：如果知道自己的代码会造成编译器反复对一个值类型装箱，请改成用手动方式对值类型进行装箱。这样代码会变得更小、更快。下面是一个例子：

```C#
using System;

public sealed class Program {
  public static void Main() {
    Int32 v = 5;      // 创建未装箱的值类型变量

  #if INEFFICIENT
    // 编译下面这一行， v 被装箱 3 次，浪费时间和内存
    Console.WriteLine("{0}, {1}, {2}", v, v, v);
  #else
    // 下面的代码结果一样，但无论执行速度，还是内存利用，都比前面的代码更胜一筹
    Object o = v;     // 对 v 进行手动装箱(仅 1 次)

    // 编译下面这一行不发生装箱
    Console.WriteLine("{0}, {1}, {2}", o, o, o);
  #endif
  }
}
```

在定义了 `INEFFICIENT` 符号的前提下编译，编译器会生成代码对 `v` 装箱 3 次，造成在堆上分配 3 个对象！这太浪费了，因为每个对象都是完全相同的内容：**5**。在没有定义 `INEFFICIENT` 符号的前提下编译， `v` 只装箱一次，所以只在堆上分配一个对象。随后，在对 `Console.WriteLine` 方法的调用中，对同一个已装箱对象的引用被传递 3 次。第二个版本执行起来快得多，在堆上分配的内存也要少得多。

通过这些例子，很容易判断在什么时候一个值类型的实例需要装箱。简单地说，要获取对值类型实例的引用，实例就必须装箱。将值类型实例传给需要获取引用类型的方法，就会发生这种情况。但这并不是要对值类型实例装箱的唯一情况。

前面说过，未装箱值类型比引用类型更“轻”。这要归结于以下两个原因。

* 不在托管堆上分配。
* 没有堆上的每个对象都有的额外成员：“类型对象指针” 和 “同步块索引”。

由于未装箱值类型没有同步块索引，所以不能使用 `System.Threading.Monitor` 类型的方法(或者 C# `lock` 语句)让多个线程同步对实例的访问。

虽然未装箱值类型没有类型对象指针，但仍可调用由类型继承或重写的虚方法(比如 `Equals`，`GetHashCode` 或者 `ToString`)。如果值类型重写了其中任何虚方法，那么 CLR 可以非虚地调用该方法，因为值类型隐式密封，不可能有类型从它们派生，而且调用虚方法的值类型实例没有装箱。然而，如果重写的虚方法要调用方法在基类中的实现，那么在调用基类的实现时，值类型实例会装箱，以便能够通过 `this` 指针将对一个堆对象的引用传给基方法。

但在调用非虚的、继承的方法时(比如 `GetType` 或 `MemberwiseClone`)，无论如何都要对值类型进行装箱。因为这些方法由 `System.Object` 定义，要求 `this` 实参是指向堆对象的指针。

此外，将值类型的未装箱实例转型为类型的某个接口时要对实例进行装箱。这是因为接口变量必须包含对堆对象的引用(接口主题将在第 13 章“接口”中讨论)。以下代码对此进行了演示：

```C#
using System;

internal struct Point : IComparable {
    private Int32 m_x, m_y;
    
    // 构造器负责初始化字段
    public Point(Int32 x, Int32 y)
    {
        m_x = x;
        m_y = y;
    }
    
    // 重写从 System.ValueType 继承的 ToString 方法
    public override string ToString()
    {
        // 将 point 作为字符创返回。注意：调用 ToString 以避免装箱
        return String.Format("({0}, {1})", m_x.ToString(), m_y.ToString());
    }
     
    // 实现类型安全的 CompareTo 方法
    public Int32 CompareTo(Point other)
    {
        // 利用勾股定理计算哪个 point 距离原点 (0, 0) 更远
        return Math.Sign(Math.Sqrt(m_x * m_x + m_y * m_y) - Math.Sqrt(other.m_x * other.m_x + other.m_y * other.m_y));
    }

    // 实现 IComparable 的 CompareTo 方法
    public Int32 CompareTo(Object o) {
        if(GetType() != o.GetType()) {
            throw new ArgumentException("o is not a Point");
        }
        // 调用类型安全的 CompareTo 方法
        return CompareTo((Point)o);
    }
}

public static class Program
{
    public static void Main()
    {
        // 在栈上闯将两个 Point 实例
        Point p1 = new Point(10, 10);
        Point p2 = new Point(20, 20);

        // 调用 ToString (虚方法)不装箱 p1
        Console.WriteLine(p1.ToString());      // 显示 "(10, 10)"

        // 调用 GetType (非虚方法)时，要对 p1 进行装箱
        Console.WriteLine(p1.GetType());       // 显示 "Point"

        // 调用 CompareTo 不装箱 p1 
        // 由于调用的是 CompareTo(Point) ，所以 p2 不装箱
        Console.WriteLine(p1.CompareTo(p2));   // 显示 "-1"

        // p1 要装箱，引用放到 c 中
        IComparable c = p1;
        Console.WriteLine(c.GetType());        // 显示 "Point"

        // 调用 CompareTo 不装箱 p1
        // 由于向 CompareTo 传递的不是 Point 变量，
        // 所以调用的是 CompareTo(Object) ，它要求获取对已装箱 Point 的引用
        // c 不装箱是因为它本来就引用已装箱 Point
        Console.WriteLine(p1.CompareTo(c));    // 显示 "0"

        // c 不装箱，因为它本来就引用已装箱 Point
        // p2 要装箱，因为调用的是 CompareTo(Object)
        Console.WriteLine(c.CompareTo(p2));    // 显示 "-1"

        // 对 c 拆箱，字段复制到 p2 中
        p2 = (Point) c;
        
        // 证明字段已复制到 p2 中
        Console.WriteLine(p2.ToString());      // 显示 "(10, 10)"
    }
}

```

上述代码演示了涉及装箱和拆箱的几种情形。

1. 调用 `ToString`
  调用 `ToString` 时 `p1` 不必装箱。表面看 `p1` 似乎必须装箱，因为 `ToString` 是从基类型 `System.ValueType` 继承的虚方法。通常，为了调用虚方法，CLR 需要判断对象的类型来定位类型的方法表。由于 `p1` 是未装箱的值类型，所以不存在“类型对象指针”。但 JIT 编译器发现 `Point` 重写了 `ToString` 方法，所以会生成代码来直接(非虚地)调用 `ToString` 方法，而不必进行任何装箱操作。编译器知道这里不存在多态性问题，因为 `Point` 是值类型，没有类型能从它派生以提供虚方法的另一个实现。但假如 `Point` 的 `ToString` 方法在内部调用 `base.ToString()` ，那么在调用 `System.ValueType` 的 `ToString` 方法时，值类型的实例会被装箱。

2. 调用`GetType`
  调用非虚方法 `GetType` 时 `p1` 必须装箱。`Point` 的 `GetType` 方法是从 `System.Object` 继承的。所以，为了调用 `GetType` ，CLR 必须使用指向类型对象的指针，而这个指针只能通过装箱 `p1` 来获得。

3. 调用 `CompareTo` (第一次)
   第一次调用 `CompareTo` 时 `p1` 不必装箱，因为 `Point` 实现了 `CompareTo` 方法，编译器能直接调用它。注意向 `CompareTo` 传递的是一个 `Point` 变量(`p2`)，所以编译器调用的是获取一个 `Point` 参数的 `CompareTo` 重载版本。这意味着 `p2` 以传值方式传给 `CompareTo`，无需装箱。

4. 转型为 `IComparable`
   `p1`转型为接口类型的变量 `c` 时必须装箱，因为接口被定义为引用类型。装箱 `p1` 后，指向已装箱对象的指针存储到变量 `c` 中。后面对 `GetType` 的调用证明 `c` 确实引用堆上的已装箱 `Point`。

5. 调用 `CompareTo` (第二次)
   第二次调用 `CompareTo` 时 `p1` 不必装箱，因为 `Point` 实现了 `CompareTo` 方法，编译器能直接调用。注意向 `CompareTo` 传递的是 `IComparable` 类型的变量 `c` ，所以编译器调用的是获取一个 `Object` 参数的 `CompareTo` 重载版本。这意味着传递的实参必须是指针，必须引用堆上一个对象。幸好， `c` 确实引用一个已装箱 `Point`， 所以 `c` 中的内存地址直接传给 `CompareTo` ，无需额外装箱。

6. 调用 `CompareTo` (第三次)
   第三次调用 `CompareTo` 时， `c` 本来就引用堆上的已装箱 `Point` 对象，所以不装箱。由于 `c` 是 `IComparable` 接口类型，所以只能调用接口的获取一个 `Object` 参数的 `CompareTo` 方法。这意味着传递的实参必须是引用了堆上对象的指针。所以 `p2` 要装箱，指向这个已装箱对象的指针将传给 `CompareTo`。

7. 转型为`Point`
   将 `c` 转型为 `Point` 时， `c` 引用的堆上对象被拆箱，其字段从堆复制到 `p2`。`p2` 是栈上的 `Point` 类型实例。

我知道，对于引用类型、值类型和装箱的所有这些讨论很容易让人产生挫败感。但是，任何 .NET Framework 开发人员只有在切实了解了这些概念之后，才能保障自己的长期成功。相信我，只有深刻理解了之后，才能更快、更轻松地构建高效率的应用程序。

### 5.3.1 使用接口更改已装箱值类型中的字段(以及为什么不应该这样做)

下面通过一些例子来验证自己对值类型、装箱和拆箱的理解程度。请研究以下代码，判断它会在控制台上显示什么：

```C#
using System;

// Point 是值类型
internal struct Point {
    private Int32 m_x, m_y;

    public Point(Int32 x, Int32 y) {
        m_x = x;
        m_y = y;
    }
    public void Change(Int32 x, Int32 y) {
        m_x = x; m_y = y;
    }

    public override string ToString() {
        return String.Format("({0}, {1})", m_x.ToString(), m_y.ToString());
    }
}

public sealed class Program {
    public static void Main() {
        Point p = new Point(1, 1);

        Console.WriteLine(p);

        p.Change(2, 2);
        Console.WriteLine(p);

        Object o = p;
        Console.WriteLine(o);

        ((Point)o).Change(3, 3);
        Console.WriteLine(o);
    }
}
```

程序其实很简单。 `Main` 在栈上创建 `Point` 值类型的实例(`p`)，将它的 `m_x` 和 `m_y` 字段设为 `1` 。然后，第一次调用 `WriteLine` 之前 `p` 要装箱。 `WriteLine` 在已装箱 `Point` 上调用 `ToString` ，并像预期的那样显示 `(1, 1)`。然后用 `p` 调用 `Change` 方法，该方法将 `p` 在栈上的 `m_x` 和 `m_y` 字段值都更改为 `2` 。第二次调用 `WriteLine` 时，再次对 `p` 进行装箱，像预料之中的那样显示 `(2, 2)`。