---
title: CSAPP-03 Machine Level Programming
categories: [csapp]
tags: [csapp]
---

## 1 历史观点

## 2 程序编码

```shell
linux> gcc -Og -o p p1.c p2.c
```

实际上 gcc 命令调用了一整套的程序，将源代码转化成可执行代码。

- 首先，C 预处理器扩展源代码，插入所有用#include 命令指定的文件，并扩展所有用#define 声明指定的宏。
- 其次，编译器产生两个源文件的汇编代码，名字分别为 P1. S 和 p2. S。
- 接下来，汇编器会将汇编代码转化成二进制目标代码文件 p1.o 和 p2.o。目标代码是机器代码的一种形式，它包含所有指令的二进制表示，但是还没有填入全局值的地址。
- 最后，链接器将两个目标代码文件与实现库函数（例如 printf）的代码合并，并产生最终的可执行代码文件 p（由命令行指示符 -o p 指定的）。可执行代码是我们要考虑的机器代码的第二种形式，也就是处理器执行的代码格式。

### 机器级代码

对于机器级编程来说，2 种抽象尤为重要

- 由指今集体系结构或指令集架构（Instruction Set Architecture, ISA） 来定义机器级程序的格式和行为，它定义了处理器状态、指令的格式，以及每条指令对状态的影响。
- 第二种抽象是，机器级程序使用的内存地址是虚拟地址，提供的内存模型看上去是一个非常大的字节数组。存储器系统的实际实现是将多个硬件存储器和操作系统软件组合起来。

x86-64 的机器代码和原始的 C 代码差别非常大。一些通常对 C 语言程序员隐藏的处理器状态都是可见的：

- ● 程序计数器（通常称为“PC”，在 x86-64 中用号 rip 表示）给出将要执行的下一条指令在内存中的地址。
- ● 整数寄存器文件包含 16 个命名的位置，分别存储 64 位的值。这些寄存器可以存储地址（对应于 C 语言的指针）或整数数据，有的寄存器被用来记录某些重要的程序状态，而其他的寄存器用来保存临时数据，例如过程的参数和局部变量，以及函数的返回值。
- ● 条件码寄存器保存着最近执行的算术或逻辑指令的状态信息。它们用来实现控制或数据流中的条件变化，比如说用来实现 if 和 while 语句。）
- ● 一组向量寄存器可以存放一个或多个整数或浮点数值。

机器代码只是简单地将内存看成一个很大的、按字节寻址的数组。

程序内存包含：程序的可执行机器代码，操作系统需要的一些信息，用来管理过程调用和返回的运行时栈，以及用户分配的内存块（比如说用 ma11oc 库函数分配的）。

程序内存用虚拟地址来寻址。在任意给定的时刻，只有有限的一部分虚拟地址被认为是合法的。

> 例如，x86-64 的虚拟地址是由 64 位的字来表示的。在目前的实现中，这些地址的高 16 位必须设置为 0，所以一个地址实际上能够指定的是 248 或 64TB 范围内的一个字节。较为典型的程序只会访问几兆字节或几千兆字节的数据。操作系统负责管理虚拟地址空间，将虚拟地址翻译成实际处理器内存中的物理地址。

### 代码示例

### 关于格式的注解

#### ATT 与 Intel 汇编代码格式

#### 内联汇编

## 3 数据格式

由于是从 16 位体系结构扩展成 32 位的，Intel 用术语 “字（word）”表示 16 位数据类型。因此，称 32 位数为“双字（double words）”，称 64 位数为“四字（quad words）”。图 3-1 给出了 C 语言基本数据类型对应的 x86-64 表示。标准 int 值存储为双字（32 位）。指针（在此用 char*表示）存储为 8 字节的四字，64 位机器本来就预期如此。x86-64 中，数据类型 1ong 实现为 64 位，允许表示的值范围较大。本章代码示例中的大部分都使用了指针和 1ong 数据类型，所以都是四字操作。x86-64 指令集同样包括完整的针对字节、字和双字的指令。![3-1](3-1.png)

浮点数主要有两种形式：

- 单精度（4 字节）值，对应于 C 语言数据类型 f1oat；
- 双精度（8 字节）值，对应于 C 语言数据类型 double。

x86 家族的微处理器历史上实现过对一种特殊的 80 位（10 字节）浮点格式进行全套的浮点运算（参见家庭作业 2.86）。可以在 C 程序中用声明 long double 来指定这种格式。不过我们不建议使用这种格式。它不能移植到其他类型的机器上，而且实现的硬件也不如单精度和双精度算术运算的高效。

如图所示，大多数 GCC 生成的汇编代码指令都有一个字符的后缀，表明操作数的大小。

> 例如，数据传送指令有四个变种：movb（传送字节）、movw（传送字）、movl（传送双字）和 movg（传送四字）。后缀‘l’用来表示双字，因为 32 位数被看成是“长字（long word）”。

注意，汇编代码也使用后缀‘l'来表示 4 字节整数和 8 字节双精度浮点数。这不会产生歧义，因为浮点数使用的是一组完全不同的指令和寄存器。

## 4 访问信息

一个 x86-64 的中央处理单元（CPU）包含一组 16 个存储 64 位值的通用目的寄存器。这些寄存器用来存储整数数据和指针。![3-2](/Users/mac/github/zennlyu.github.io/hexo/source/statics/csapp/3-2.png)

图 3-2 显示了这 16 个寄存器。它们的名字都以号 r 开头，不过后面还跟着一些不同的命名规则的名字，这是由于指令集历史演化造成的。

最初的 8086 中有 8 个 16 位的寄存器，即图 3-2 中的 %ax 到 %bp。每个寄存器都有特殊的用途，它们的名字就反映了这些不同的用途。扩展到 IA32 架构时，这些寄存器也扩展成 32 位寄存器，标号从 %eax 到 %ebp。扩展到 x86-64 后，原来的 8 个寄存器扩展成 64 位，标号从 %rax 到 %rbp。除此之外，还增加了 8 个新的寄存器，它们的标号是按照新的命名规则制定的：从 %r8 到 %r15。

如图 3-2 中嵌套的方框标明的，指令可以对这 16 个寄存器的低位字节中存放的不同大小的数据进行操作。字节级操作可以访问最低的字节，16 位操作可以访问最低的 2 个字节，32 位操作可以访问最低的 4 个字节，而 64 位操作可以访问整个寄存器。

在后面的章节中，我们会展现很多指令，复制和生成 1 字节、2 字节、4 字节和 8 字节值。当这些指令以寄存器作为目标时，对于生成小于 8 字节结果的指令，寄存器中剩下的字节会怎么样，对此有两条规则：生成 1 字节和 2 字节数字的指令会保持剩下的字节不变；生成 4 字节数字的指令会把高位 4 个字节置为 0。后面这条规则是作为从 IA32 到 x86-64 的扩展的一部分而采用的。

就像图 3-2 右边的解释说明的那样，在常见的程序里不同的寄存器扮演不同的角色。

其中最特别的是栈指针 %rsp，用来指明运行时栈的结束位置。有些程序会明确地读写这个寄存器。另外 15 个寄存器的用法更灵活。少量指令会使用某些特定的寄存器。更重要的是，有一组标准的编程规范控制着如何使用寄存器来管理栈、传递函数参数、从函数的返回值，以及存储局部和临时数据。我们会在描述过程的实现时（特别是在 3.7 节中），讲述这些惯例。

### 操作数指示符

### 数据传送指令

### 数据传送示例

### 压入和弹出栈数据

## 5 算数和逻辑操作

### 加载有效地址

### 一元和二元操作

### 移位操作

### 讨论

### 特殊的算术控制

## 6 控制

### 条件码

### 访问条件码

### 跳转指令

### 跳转指令的编码

### 用条件控制来实现条件分支

### 用条件传送来实现条件分支

### 循环

### do-while

### while

### for

### switch

## 7 过程

### 运行时栈

### 转移控制

### 数据传送

### 栈上的局部存储

### 寄存器中的局部存储空间

### 递归过程

## 8 数组分配和访问

### 基本原则

### 指针运算

### 嵌套的数组

### 定长数组

### 变长数组



## 9 异质的数据结构

### 结构

### 联合

### 数据对齐

## 10 在机器级程序中将控制与数据结合起来

### 理解指针

### 应用：使用 GDB 调试器

### 内存越界引用和缓冲区溢出

### 对抗缓冲区溢出攻击

#### 栈随机化

#### 栈破坏检测

#### 限制可执行代码区域

#### 支持变长栈帧

## 11 浮点代码

### 浮点传送和转换操作

### 过程中的浮点代码

### 浮点运算操作

### 定义和使用浮点常数

### 在浮点代码中使用位级操作

### 浮点比较操作

### 对浮点代码的观察结论