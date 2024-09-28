## 一、了解Verilog HDL 

Verilog HDL是硬件描述语言的一种，用于数字电子系统设计。硬件描述语言是具有特殊结构，能够对硬件逻辑电路的功能进行描述的一种高级编程语言。

## 二、Verilog HDL 的模块

Verilog HDL 的基本设计单元是 “模块'”。一个模块是由两部分组成，一部分描述接口，另一部分描述逻辑功能，即定义输入时如何影响输出的，程序模块如下：

```verilog
module block1(a,b,c,d);
    input a,b;
    output c,d;
    
    assign c=a|b;
    assign d=a&b;
endmodule
```

Verilog HDL 模块的结构由在 `module` 和 `endmodule` 关键词之间的4个主要部分组成，如下所示：

```verilog
module block1(a,b,c,d);  //端口定义
    
    input a,b,c;
    output d;            // I/O说明
    
    wire x;              //内部信号声明
    
    assign d=a|x;
    assign x=(b&~c);     //功能定义
endmodule
```

### 1. 端口的定义

模块的端口声明了模块的输入输出口。格式如下：

> module 模块名(输入/输出端口列表)

在端口定义的圆弧中，是设计电路模块与外界联系的全部输入输出端口信号或引脚，它是设计实体对外的一个通信界面，是外界可以看到的部分 (不包含电源和接地端)，多个端口名之间用 “,” 分隔。<br>例如：`module adder(sum,count,ina,inb,inc);`

### 2.模块的描述方式

模块的内容包括 I/O说明、内部信号声明、功能定义。模块最重要的部分是逻辑功能定义部分。<br>有3种方式可在模块中产生逻辑，这3种不同的描述方式就是**数据流描述方式、行为描述方式和结构化描述方式**。

#### 2.1 数据流描述方式

用数据流描述方式对一个设计建模的最基本的机制就是使用连续赋值语句。在连续赋值语句中，某个值被指派给**线网变量**。<br>连续赋值语句的语法为：`assign [delay]LHS net = RHS_expression;`<br>右边表达式使用的操作数无论何时发生变化，右边的表达式都重新计算，并且在确定的时延后变化的值被赋予左边表达式的**线网变量**。时延定义了右边表达式操作数变化与赋值给左边表达式之间的持续时间。如果没有定义时延值，缺省时延为0。

下图是2-4解码器电路的建模实例模型：

![a2b9613bfa9eedf69498a7bd1952565a](assets/a2b9613bfa9eedf69498a7bd1952565a.png)

对2-4解码器电路采用数据流描述方式的代码如下：<br>

```verilog
`timescale 1ns/1ns
module Decoder2x4(A,B,EN,Z) ;
    input A,B,EN;
    output [0:3] Z;
    wire Abar,Bbar;
    assign #1 Abar=~A;// 语句1
    assign #1 Bbar=~B;//语句2
    assign #2 Z[0]=~(Abar &Bbar&EN); //语句3
    assign #2 Z[1]=~(Abar & B&EN);//语句4
    assign #2 Z[2]=~(A&Bbar&EN);//语句5
    assign #2 Z[3]=~(A&B&EN); //语句6
endmodule
```

以反引号 “ ` ” (Esc 下面的按键) 开始的第一条语句是编译器指令，编译器指令“timescale”将模块中所有时延的单位设置为 1ns，时间精度为 1ns。例如，在连续赋值语句中时延值#1 和#2 分别对应时延1ns 和2ns。模块 Decoder2x4 有3个输人端口和1个4位输出端口。线网类型说明了2个连线型变量Abar 和 Bbar (连线类型是线网类型的一种)。此外，模块包含6个连续赋值语句。<br>==注意==:连续赋值语句是如何对电路的数据流行为建模的;这种建模方式是隐式而非显式的建模方式。此外，连续赋值语句是并发执行的，也就是说各语句的执行顺序与其在描述中出现的顺序无关。<br>连续赋值语句实例波形图如下：<br>![2aae9926b64599ae5e5e43a90f4fc1f1_720](assets/2aae9926b64599ae5e5e43a90f4fc1f1_720.png)

当 EN 在第 5ns 变化时，语句3、4、5 和6执行。这是因为EN是这些连续赋值语句中右边表达式的操作数。Z[0]在第7ns 时被赋予新值0。当A在第 15ns 变化时，语句1、5和6执行。执行语句5 和6不影响Z[0]和Z[1]的取值。执行语句5导致Z[2]的值在第17ns变为0。执行语句1 导致Abar在第 16s 时被重新赋值。由于Abar的改变，反过来又导致 Z[0]值在第 18ns 时变为1。

#### 2.2 行为描述方式

##### 2.2.1 两种过程语句结构

设计的行为功能使用下述过程语句结构描述。<br>`initial` 语句：此语句只执行一次。<br>`always` 语句：此语句总是循环执行，或者说此语句重复执行。<br>只有**寄存器类型**的数据能够在这两种语句中被赋值。**寄存器类型**的数据在被赋新值前保持原有值不变。所有的初始化语句和`always`语句在 0 时刻并发执行。

下例为 **always 语句**对**1位全加器电路**建模的示例:<br>![image-20240928164206688](assets/image-20240928164206688.png)

其代码如下：

```verilog
module FASeq(ain,bin,cin,sum,cout);
    input ain,bin,cin;
    output sum,cout;
    reg sum,cout;
    reg s1,s2,s3;
    always@(ain or bin or cin)
        begin
            sum = (ain^bin)^cin;
            s1 = ain&cin;
            s2 = bin&cin;
            s3 = ain&bin;
            cout = (s1|s2)|s3;
        end 
endmodule    
```

模块 FASeq 有 3 个输入和 2 个输出。由于 sum、cout、s1、s2 和 s3 在 aways 语句中被赋值，它们被说明为 **reg 类型(reg 是寄存器数据类型的一种)**。always 语句中有一个与**事件控制 (紧跟在字符@后面的表达式)** 相关联的**顺序过程(begin - end对)**。这意味着只要ain、bin或 cin 上发生事件，即ain、bin或 cin 之一的值发生变化，顺序过程就执行。在==顺序过程中的语句顺序执行==，并且在顺序过程执行结束后被挂起。顺序过程执行完成后，always 语句再次等待 ain、bin或 cin 上发生的事件。

下面是 **initial 语句**的示例，其代码如下:<br>

```verilog
`timescale 1ns/1ns
module Test(Pop,Pid);
    output Pop,Pid;
    reg Pop,Pid;
    initial
        begin
            Pop=0;//         语句1
            Pid=0;//         语句2
            Pop = #5 1;//    语句3
            Pid = #3 1;//    语句4
            Pop = #6 0;//    语句5
            Pid = #2 0;//    语句6
        end
endmodule         
```

这一模块产生如图所示的波形：<br>![屏幕截图 2024-09-28 174249](assets/屏幕截图 2024-09-28 174249.png)

![efe4036ae150adb9cdc4ec685431d44a](assets/efe4036ae150adb9cdc4ec685431d44a.png)

initial 语句包含一个顺序过程，这一顺序过程在 0ns 时开始执行，并且在顺序过程中所有语句全部执行完毕后，initial 语句永远挂起。这一顺序过程包含带有定义语句内时延的分组过程赋值的实例。语句1 和语句 2 在 0ns 时执行第 3 条语句也在    0 时刻执行，导致 Pop 在第 5ns 时被赋值。语句4 在第 5ns 时执行，并且 Pid在第 8ns 时被赋值。同样，Pop 在第 14ns 时被赋值为0，Pid 在第 16ns 时被赋值为0。第 6 条语句执行后，initial 语句永远被挂起。

##### 2.2.2 时延的两种类型

在顺序过程中出现的语句是过程赋值模块化的实例。模块化过程赋值在下一条语句执行前完成执行。<br>过程赋值可以有一个可选的时延，时延可以细分为两种类型：<br>**语句间时延**：这是时延语句执行的时延。<br>**语句内时延**：这是右边表达式数值计算与左边表达式赋值间的时延。

**语句间时延**的示例代码如下：<br>

```verilog
sum = (ain^bin)^cin;
#4 s1 = ain&cin;
```

在第二条语句中的时延规定赋值延迟 4 个时间单位执行。简单来说，在第一条语句执行完后等待 4 个时间单位，然后执行第二条语句。

**语句内时延**的示例代码如下：

```verilog
sum = #3(ain^bin)^cin;
```

这个赋值中的时延意味着首先计算右边表达式的值，等待 3 个时间单位，然后赋值给sum。如果在过程赋值中未定义时延，缺省值为 0 时延，也就是说，赋值立即发生。这种形式及在 always 语句中指定语句的其他形式在此不做详细讨论。

#### 2.3 结构化描述方式

在 Verilog HDL 种使用如下方式描述结构：

- 内置门原语（在门级）
- 开关级原语（在晶体管级）
- 用户定义的原语（在门级）
- 模块实例（创建层级结构）

通过使用线网来相互连接。下面是结构化描述方式使用内置门原语描述全加器电路的实例。该实例基于上面1为全加器的逻辑图，其代码如下：<br>

```verilog
module FAStr(A, B,Cin, Sum, Cout);
    input A,B,Cin;
    output Sum,Cout;
    wire S1,T1,T2,T3;
    xor
    X1 (S1,A,B);
    X2 (Sum,S1,Cin);
    and
    A1 (T3,A,B);
    A2 (T2,B,Cin);
    A3 (T1,A,Cin);
    or
    O1 ( Cout,T1,T2,T3);
endmodule
// A-ain B-bin Cin-cin Sum-sum Cout-cout T1、T2、T3-s1、s2、s3
```

在这一实例中，模块包含门的实例语句，也就是说包含内置门 `xor`、`and` 和 `or` 的实例语句。门实例由线网类型变量 S1、T1、T2 和 T3 互连。由于没有指定的顺序，门实例语句可以以任何顺序出现;图中显示了纯结构; `xor`、`and` 和`or` 是内置门原语；X1、X2、A1 等是实例名称。紧跟在每个门后的信号列表是它的互连;列表中的第一个是门输出，余下的是输人。例如，S1 与`xor` 门实例X1的输出连接，而 A 和 B 与实例 X1 的输人连接。<br>4 位全加器可以使用4 个1位全加器模块描述，描述的逻辑图如下图所示：<br>![f9d54f877930c4ed05d4cc4c40959a80_720](assets/f9d54f877930c4ed05d4cc4c40959a80_720.png)

代码如下：<br>

```verilog
module FourBitFA (FA,FB,FCin,FSum,FCout) ;
    parameter SIZE =4;
    input [SIZE:1] FA,FB;
    output [SIZE:1] FS um
    input FCin;
    input FCout;
    wire [1:SIZE-1] FTemp;
    FAStr
    FA1(.A(FA[1]),.B(FB[1]),.Cin( FCin),.Sum( FSum[1]),.Cout( FTemp[1]));
    FA2(.A(FA[2]),.B(FB[2]),.Cin( FTemp[1]),.Sum(FSum[2]),.Cout(FTemp[2]));
    FA3(.A(FA[3]),.B(FB[3]),.Cin( FTemp[2]),.Sum(FSum[3]),.Cout(FTemp[3]));
    FA4(.A(FA[4]),.B(FB[4]),.Cin( FTemp[3]),.Sum(FSum[4]),.Cout(FCout)):
endmodule
```

在这一实例中，模块实例用于建模 4 位全加器。在模块实例语句中，端口可以与名称或位置关联。前两个实例 FA1 和 FA2 使用命名关联方式，也就是说，端口的名称和它连接的线网被显式描述 (每一个的形式都为“portmame (netame))”。最后两个实例语句，实例 FA3 和 FA4 使用位置关联方式将端口与线网关联。这里关联的顺序很重要，例如，在实例 FA4 中，第一个FA[4]与FAStr 的端口A连接，第二个 FB[4] 与 FAS 的端口B连接，余下的以此类推。

#### 2.4 混合设计描述方式

在模块中，结构的和行为的结构可以自由混合。也就是说，模块描述中可以包含实例化的门、模块实例化语句、连续赋值语句，以及 always 语句和 initial 语句的混合。它们之间可以相互包含。来自 always 语和 initial 语(切记只有存器类型数据可以在这两种语句中赋值)的值能够驱动门或开关，而来自于门或连续赋值语句 (只能驱动线网) 的值能够反过来用于触发 always 语句和 initial 语句。

混合设计方式的 1 位全加器实例代码如下：<br>

```verilog
module FAMix(A,B,Cin,Sum,Cout);
    input A,B,Cin;
    output Sum,Cout;
    reg Cout;
    reg T1,T2,T3;
    wire S1;
    xor 
    X1 (S1,A,B);      //门实例语句
    always@ (A or B or Cin)  //always语句
        begin
            T1 = A&Cin;
            T2 = B&Cin;
            T3 = A&B;
            Cout = (T1|T2)|T3;
        end
    assign Sum =S1^Cin;     //连续赋值语句
endmodule
```

只要 A或 B上有事件发生，门实例语句即被执行。只要 A、B或C上有事件发生，就执行 always 语句，并且只要 S1 或 Cin 上有事件发生，就执行连续赋值语句。
