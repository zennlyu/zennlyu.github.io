---
title: CSAPP-Ch02_Bits & Bytes & Integer & Floating Point
categories: [csapp]
tags: [csapp]
---

3种重要的数字表示：

- unsigned
- two's complement
- floating point

阅读技巧：首先给出以数学形式表示的属性，作为原理。然后，用例子和非形式化的讨论来解释这个原理。我们建议你反复阅读原理描述和它的示例与讨论，直到你对该属性的说明内容及其重要性有了牢固的直觉。对于更加复杂的属性，还会提供推导，其结构看上去将会像一个数学证明。虽然最终你应该尝试理解这些推导，但在第一次阅读时你可以跳过它们。

## 1 信息存储

大多数计算机使用 8 位的块，或者字节（byte），作为最小的可寻址的内存单位，而不是访问内存中单独的位。

### 虚拟内存

- 机器级程序将内存视为一个非常大的字节数组，称为**虚拟内存（virtual memory）**。
- 内存的每个字节都由一个唯一的数字来标识，称为它的**地址（address）**
- 所有可能地址的集合就称为**虚拟地址空间（virtual address space）**。顾名思义，这个虚拟地址空间只是一个展现给机器级程序的概念性映像。
- 实际的实现（见第 9 章）是将动态随机访问存储器（DRAM）、闪存、磁盘存储器、特殊硬件和操作系统软件结合起来，为程序提供一个看上去统一的字节数组。
- 本章重点是编译器和运行时系统是如何将存储器空间划分为更可管理的单元，来存放不同的程序对象（program object），即程序数据、指令和控制信息。

可以用各种机制来分配和管理程序不同部分的存储。这种管理完全是在虚拟地址空间里完成的。

- 例如，C 语言中一个指针的值（无论它指向一个整数、一个结构或是某个其他程序对象）都是某个存储块的第一个字节的虚拟地址。
- C 编译器还把每个指针和类型信息联系起来，这样就可以根据指针值的类型，生成不同的机器级代码来访问存储在指针所指向位置处的值。
- 尽管 C 编译器维护着这个类型信息，但是它生成的实际机器级程序并不包含关于数据类型的信息。

每个程序对象可以简单地视为一个字节块，而程序本身就是一个字节序列。

### 十六进制表示法

### 字数据大小

![2-3](../statics/csapp/2-3.png)

每台计算机都有一个字长（word size），指明指针数据的标称大小（nominal size）。因为虚拟地址是以这样的一个字来编码的，所以字长决定的最重要的系统参数就是虚拟地址空间的最大大小。也就是说，对于一个字长为 w 位的机器而言，虚拟地址的范围为 0-2<sup>w</sup>-1，程序最多访问 2<sup>w</sup>个字节。

```sh
linux> gcc -m32 prog.c
```

###### C 语言数据类型分配字节数（C标准保证的字节数和典型的字节数之间的关系）

为了避免由于依赖“典型”大小和不同编译器设置带来的奇怪行为，`ISO C99` 引人了一类数据类型，其数据大小是固定的，不随编译器和机器设置而变化。其中就有数据类型 `int32_t` ,`int64_t`, 分别为 4 和 8 个字节

大部分数据类型都编码为有符号数值，除非有前缀关键字 `unsigned` 或对确定大小的数据类型使用了特定的无符号声明。

> 数据类型 char 是一个例外。尽管大多数编译器和机器将它们视为有符号数，但 C 标准不保证这一点。
>
> 相反，正如方括号指示的那样，程序员应该用有符号字符的声明来保证其为一个字节的有符号数值。
>
> 不过，在很多情况下，程序行为对数据类型 char 是有符号的还是无符号的并不敏感。

大多数机器支持两种不同的浮点数格式：单精度 float & 双精度 double

以下声明都一样

```c
unsigned long
unsigned long int
long unsigned
long unsigned int
```

###### 声明指针

```c
T *p;		// 对于任何数据类型 T，p 是一个指针变量，指向一个类型为 T 的对象
char *p	// 将一个指针声明为指向一个 char 类型的对象
```

可移植性的一个方面就是使程序对不同数据类型的确切大小不敏感。**C 语言标准对不同数据类型的数字范围设置了下界（这点在后面还将讲到），但是却没有上界。**

> 因为从 1980 年左右到 2010 年左右，32 位机器和 32 位程序是主流的组合，许多程序的编写都假设为图 2-3 中 32 位程序的字节分配。随着 64 位机器的日益普及，在将这些程序移植到新机器上时，许多隐藏的对字长的依赖性就会显现出来，成为错误。
>
> 比如，许多程序员假设一个声明为 int 类型的程序对象能被用来存储一个指针。这在大多数 32 位的机器上能正常工作，但是在一台 64 位的机器上却会导致问题。

### 寻址和字节顺序

对于跨越多字节的程序对象，我们必须建立两个规则：

- 这个对象的地址是什么
- 在内存中如何排列这些字节

在几乎所有的机器上，多字节对象都被存储为连续的字节序列，对象的地址为所使用字节中最小的地址。

> 例如，假设一个类型为 int 的变量×的地址为 0x100, 也就是说，地址表达式&×的值为 0x100。
>
> 那么，（假设数据类型 int 为 32 位表示）x 的 4 个字节将被存储在内存的 0x100、0x101、0x102 和 0x103 位置。

**排列表示一个对象的字节有两个通用的规则** ：考虑一个 w 位的整数，位表示为 [x<sub>w-1</sub>,x<sub>w-2</sub>,x<sub>w-3</sub>,...x<sub>1</sub>,x<sub>0</sub>]

- 其中 x<sub>w-1</sub> 是最高有效位，x<sub>0</sub> 是最低有效位
- 假设 w 是 8 的倍数，这些位就能被分组为字节，其中最高有效字节包含位 [x<sub>w-1</sub>,x<sub>w-2</sub>,x<sub>w-3</sub> x<sub>w-4</sub>,x<sub>w-5</sub>,x<sub>w-6</sub>,x<sub>w-7</sub>]，最低有效字节位包含[x<sub>7</sub>,x<sub>6</sub>,x<sub>5</sub>,x<sub>4</sub>,x<sub>3</sub>,x<sub>2</sub>,x<sub>1</sub>,x<sub>0</sub>], 其他字节包含中间的位。

##### 字节顺序

- 某些机器选择在内存中按照从最低有效字节到最高有效字节的顺序存储对象，而另一些机器则按照从最高有效字节到最低有效字节的顺序存储。

- 前一种规则一最低有效字节在最前面的方式，称为**小端法（little endian）**。后一种规则一最高有效字节在最前面的方式，称为**大端法（big endian)**

- 双端法

- 对于大多数应用程序员来说，其机器所使用的字节顺序是完全不可见的。无论为哪种类型的机器所编译的程序都会得到同样的结果。

- 不过有时候，字节顺序会成为问题。

  - 首先是在不同类型的机器之间通过网络传送二进制数据时，一个常见的问题是当小端法机器产生的数据被发送到大端法机器或者反过来时，接收程序会发现，字里的字节成了反序的。为了避免这类问题，网络应用程序的代码编写必须遵守已建立的关于字节顺序的规则，以确保发送方机器将它的内部表示转换成网络标准，而接收方机器则将网络标准转换为它的内部表示。

    我们将在第 11 章中看到这种转换的例子。

  - 第二种情况是，当阅读表示整数数据的字节序列时，字节顺序也很重要。这通常发生在检查机器级程序时。

  - 第三种情况是当编写规避正常的类型系统的程序时。C语言可以通过使用强制类型转换（cast） 或联合（union）来允许以一种数据类型引用一个对象，而这种数据类型与创建这个对象时定义的数据类型不同。大多数应用编程都强烈不推荐这种编码技巧，但是它们对系统级编程来说是非常有用，甚至是必需的。

    ```c
    /* 
    打印程序对象的字节表示。这段代码使用强制类型转换来规避类型系统。
    很容易定义针对其他数据类型的类似函数.
    分别输出类型为 int、f1oat 和 void*的 C 程序对象的字节表示。
    * 可以观察到它们仅仅传递给 show bytes 一个指向它们参数 x 的指针&x，且这个指针被强制类型转换为“unsigned char*”。
    * 这种强制类型转换告诉编译器，程序应该把这个指针看成指向一个字节序列，而不是指向一个原始数据类型的对象。
    * 然后，这个指针会被看成是对象使用的最低字节地址。
    */ 
    #include <stdio.h>
    
    typedef unsigned char *byte_pointer;
    
    void show_bytes(byte_pointer start, size_t len) {
    	size_t i;
    	for (i = 0; i < len; i++) {
    		printf("%.2x\n", start[i]);
    	}
    }
    
    void show_int(int x) {
    	show_bytes((byte_pointer) &x, sizeof(int));
    }
    
    void show_float(int x) {
    	show_bytes((byte_pointer) &x, sizeof(float));
    }
    
    void show_pointer(void *x) {
    	show_bytes((byte_pointer) &x, sizeof(void *));
    }
    ```
  
    参数 12345 的十六进制表示为 0x00003039。对于 int 类型的数据，除了字节顺序以外，我们在所有机器上都得到相同的结果。特别地，我们可以看到在 Linux32、Windows 和 Linux64 上，最低有效字节值 0x39 最先输出，这说明它们是小端法机器；而在 Sun 上最后输出，这说明 Sun 是大端法机器。同样地，float 数据的字节，除了字节顺序以外， 也都是相同的。另一方面，**指针值却是完全不同的**。不同的机器/操作系统配置使用不同的存储分配规则。一个值得注意的特性是 Linux32、Windows 和 Sun 的机器使用 4 字节地址，而 Linux64 使用 8 字节地址。

    ![2-6](../statics/csapp/2-6.png)
    
    可以观察到，尽管浮点型和整型数据都是对数值 12345 编码，但是它们有截然不同的字节模式：整型为 0x00003039, 而浮点数为 0x4640E400。一般而言，这两种格式使用不同的编码方法。如果我们将这些十六进制模式扩展为二进制形式，并且适当地将它们移位，就会发现一个有 13 个相匹配的位的序列，用一串星号标识出来：![float](../statics/csapp/float.png)

##### C 语言技巧

###### 使用 typedef 命名数据类型

```c
typedef int *int_pointer; // 将类型 int_pointer 定义为一个指向 int 的指针，并且声明了一个这种类型的变量 ip
int_pointer ip;

int *ip
```

###### 使用 printf 格式化输出：对格式化细节的控制能力

> 指定确定大小数据类型的格式，如 it32t，要更复杂一些，相关内容参见 2.2.3 节的旁注。

###### 指针和数组

> 在 C 语言中，我们能够用数组表示法来引用指针，同时我们也能用指针表示法来引用数组元素。在这个例子中，引用 start [i】表示我们想要读取以 start 指向的位置为起始的第 i 个位置处的字节。



###### 指针的创建和间接引用

> C 的“取地址”运算符 & 创建一个指针。在这三行中，表达式 &x 创建了一个指向保存变量 x 的位置的指针。这个指针的类型取决于 x 的类型，因此这三个指针的类型分别为 int*、f1oat*和 void。（数据类型 void 是一种特殊类型的指针，没有相关联的类型信息。）
>
> 强制类型转换运算符可以将一种数据类型转换为另一种。因此，强制类型转换 (byte pointer)&x 表明无论指针&x 以前是什么类型，它现在就是一个指向数据类型为 unsigned char 的指针。这里给出的这些强制类型转换不会改变真实的指针，它们只是告诉编译器以新的数据类型来看待被指向的数据。
>

###### 生成一张 ASCII 表

```shell
man ascii
```



### 表示字符串

C 语言中字符串被编码为一个以 nuIl（其值为 0) 字符结尾的字符数组。每个字符都由某个标准编码来表示，最常见的是 ASCII 字符码。

因此，如果我们以参数“12345”和 6（包括终止符）来运行例程 show bytes，我们得到结果 31 32 33 34 35 00。

请注意，十进制数字×的 ASCII 码正好是 0x3x，而终止字节的十六进制表示为 0x00。在使用 ASCII 码作为字符码的任何系统上都将得到相同的结果，与字节顺序和字大小规则无关。因而，文本数据比二进制数据具有更强的平台独立性。



### 表示代码

计算机系统的一个基本概念：从机器的角度来看，程序仅仅只是字节序列。



### 布尔代数 & 位向量

###### 位向量的运算

- 位向量就是固定长度为 w、由 0 和 1 组成的串。
- 位向量的运算可以定义成参数的每个对应元素之间的运算。
- 假设 α 和 b 别表示位向量[a<sub>w-1</sub>, a<sub>w-2</sub>, …, a<sub>0</sub>] 和 [b<sub>w-1</sub>, b<sub>w-2</sub>, …, b<sub>0</sub>]。我们将 a&b 也定义为一个长度为 w 的位向量，其中第 i 个元素等于 a<sub>i</sub> & b<sub>i</sub>:,0≤i<w。
- 可以用类似的方式将运算 |, ^, 和 ~ 扩展到位向量上。

###### 位向量-表示有限集合

- 我们可以用位向量 [a<sub>w-1</sub>, a<sub>w-2</sub>, …, a<sub>0</sub>] 编码任何子集 A ⊆ {0,1,…,w-1}，其中 a<sub>i</sub>=1 当且仅当 i∈A。

  > 例如（记住我们是把 a<sub>w-1</sub> 写在左边，而将 a<sub>0</sub> 写在右边），位向量 a ≐ [01101001] 表示集合 A={0,3,5,6｝，而 b≐[01010101] 表示集合 B={0,2,4,6}。

- 使用这种编码集合的方法，布尔运算 | 和 & 分别对应于集合的并和交，而 ~ 对应于于集合的补。还是用前面那个例子，运算 α&b 得到位向量【01000001]，而 A∩B={0,6}。

- 在大量实际应用中，我们都能看到用位向量来对集合编码。

  > 例如，在第 8 章，我们会看到有很多不同的信号会中断程序执行。我们能够通过指定一个位向量掩码，有选择地使能或是屏蔽一些信号，其中某一位位置上为 1 时，表明信号 i 是有效的（使能），而 0 表明该信号是被屏蔽的。因而，这个掩码表示的就是设置为有效信号的集合。

###### 关于布尔代数和布尔环的更多内容

![bool](../statics/csapp/bool.png)

###### 练习题

![boolExe](../statics/csapp/boolExe.png)

### C 语言中的位级运算

###### C 语言支持按位布尔运算

这些运算能运用到任何 “整型” 数据类型上

正如示例说明的那样，确定一个位级表达式的结果最好的方法，就是将十六进制的参数扩展成二进制表示并执行二进制运算，然后再转换回十六进制。![xor](../statics/csapp/xor.png)

###### 位级运算的一个常见用法就是实现掩码运算

这里掩码是一个位模式，表示从一个字中选出的位的集合。

> 例子：掩码 0xFF（最低的 8 位为 1) 表示一个字的低位字节。位级运算 x&0xFF 生成一个由 x。的最低有效字节组成的值，而其他的字节就被置为 0。
>
> 比如，对于 x=0x89ABCDEF，其表达式将得到 0x000000EF。表达式 ~0 将生成一个全1的掩码，不管机器的字大小是多少。
>
> 尽管对于一个 32 位机器来说，同样的掩码可以写成 0xFFFFFFFF，但是这样的代码不是可移植的。

### C 语言中的逻辑运算

C 语言还提供了一组逻辑运算符 ‖、&&和 !，分别对应于命题逻辑中的 OR、AND 和 OT 运算。

逻辑运算很容易和位级运算相混淆，但是它们的功能是完全不同的。

逻辑运算认为所有非零的参数都表示 TRUE，而参数 0 表示 FALSE。它们返回 1 或者 0，分别表示结果为 TRUE 或者为 FALSE。以下是一些表达式求值的示例。

逻辑运算符&&和‖与它们对应的位级运算&和|之间第二个重要的区别是，如果对第一个参数求值就能确定表达式的结果，那么逻辑运算符就不会对第二个参数求值。因此，例如，表达式a&&5/a将不会造成被零除，而表达式p&&*p++也不会导致间接引用空指针。

### C 语言中的移位运算

对于一个位表示为  [x<sub>w-1</sub>,x<sub>w-2</sub>...x<sub>0</sub>] 的操作数 x, C 表达式 x << k 会生成一个值，其位表示为 [x<sub>w-k-1</sub>,x<sub>w-k-2</sub>,...,x<sub>0</sub>,0,...,0]。也就是说，x 向左移动 k 位，丢弃最高的 k 位，并在右端补 k 个 0。移位量应该是一个 0~w-1 之间的值。移位运算是从左至右可结合的，所以 x<<j<<k 等价于（x<<j)<<k。

有一个相应的右移运算 x>>k，但是它的行为有点微妙。一般而言，机器支持两种形式的右移：

- 逻辑右移算术右移逻辑右移在左端补 k 个 0，得到的结果是 [0,...,0,x<sub>w-1</sub>,x<sub>w-2</sub>,...,x<sub>k</sub>]
- 算术右移是在左端补及个最高有效位的值，得到的结果是[x<sub>w-1</sub>,...,x<sub>w-1</sub>,x<sub>w-1</sub>,x<sub>w-2</sub>...x<sub>k</sub>] 。这种做法看上去可能有点奇特，但是我们会发现它对有符号整数数据的运算非常有用。

让我们来看一个例子，下面的表给出了对一个 8 位参数×的两个不同的值做不同的移位操作得到的结果：

| Operation                | Value                     |
| ------------------------ | ------------------------- |
| Parameter x              | [01100011] [10010101]     |
| x << 4                   | [0011*0000*] [0101*0000*] |
| x >> 4(Logical shift)    | [*0000*0110] [*0000*1001] |
| x >> 4(arithmetic shift) | [*0000*0110] [*1111*1001] |

斜体的数字表示的是最右端（左移）或最左端（右移）填充的值。可以看到除了一个条目之外，其他的都包含填充 0。唯一的例外是算术右移【10010101] 的情况。因为操作数的最高位是 1，填充的值就是 1。

C 语言标准并没有明确定义对于有符号数应该使用哪种类型的右移一算术右移或者逻辑右移都可以。不幸地，这就意味着任何假设一种或者另一种右移形式的代码都可能会遇到可移 植性问题。然而，实际上，几乎所有的编译器/机器组合都对有符号数使用算术右移，且许多程序员也都假设机器会使用这种右移。另一方面，对于无符号数，右移必须是逻辑的。

与 C 相比，Jva 对于如何进行右移有明确的定义。表达是 x>>k 会将 x 算术右移 k 个位置，而 x>>k 会对 x 做逻辑右移。

###### 旁注移动 K 位，这里 k 很大

对于一个由位组成的数据类型，如果要移动 k≥w 位会得到什么结果呢？例如，计算下面的表达式会得到什么结果，假设数据类型 int 为 w=32：

```c
int 	 lval = 0xFEDCBA98  	<<32;
int 	 aval = 0xFEDCBA98  	>>36;
unsigned uval = OxFEDCBA98u >>40;
```

C 语言标准很小心地规避了说明在这种情况下该如何做。在许多机器上，当移动一个w位的值时，移位指令只考虑位移量的低 log<sub>2</sub>w 位，因此实际上位移量就是通过计算 k mod w 得到的。例如，当 ω=32 时，上面三个移位运算分别是移动 0、4 和 8 位，得到结果：

```c
lval 0xFEDCBA98
aval OxFFEDCBA9
uval OxOOFEDCBA
```

不过这种行为对于 C 程序来说是没有保证的，所以应该保持位移量小于待移位值的位数

另一方面，Java 特别要求位移数量应该按照我们前面所讲的求模的方法来计算。

## 2 整数表示

![2-8](../statics/csapp/2-8.png)

### 整型数据类型

C 语言支持多种整型数据类型——表示有限范围的整数。![2-9](../statics/csapp/2-9.png)![2-10](/Users/mac/github/zennlyu.github.io/hexo/source/statics/csapp/2-10.png)

每种类型都能用关键字来指定大小，这些关键字包括 char、short、long，同时还可以指示被表示的数字是非负数（声明为 unsigned），或者可能是负数（默认）。

如图 2-3 所示，为这些不同的大小分配的字节数根据程序编译为 32 位还是 64 位而有所不同。根据字节分配，不同的大小所能表示的值的范围是不同的。这里给出来的唯一一个与机器相关的取值范围是大小指示符 log 的。大多数 64 位机器使用 8 个字节的表示，比 32 位机器上使用的 4 个字节的表示的取值范围大很多。

图 2-9 和图 2-10 中一个很值得注意的特点是取值范围不是对称的一负数的范围比整数的范围大 1。当我们考虑如何表示负数的时候，会看到为什么会这样。![2-11](../statics/csapp/2-11.png)

C 语言标准定义了每种数据类型必须能够表示的最小的取值范围。特别地，除了固定大小的数据类型是例外，我们看到它们只要求正数和负数的取值范围是对称的。

此外，数据类型 int 可以用 2 个字节的数字来实现，而这几乎回退到了 16 位机器的时代。

还可以看到，log 的大小可以用 4 个字节的数字来实现，对 32 位程序来说这是很典型的。固定大小的数据类型保证数值的范围与图 2-9 给出的典型数值一致，包括负数与正数的不对称性。

### 无符号数的编码

###### 无符号编码的定义

假设有一个整数数据类型有位。我们可以将位向量写成 $\vec{x}$，表示整个向量，或者写成 [x<sub>w-1</sub>,x<sub>w-2</sub>,...,x<sub>0</sub>]，表示向量中的每一位。把 $\vec{x}$ 看做一个二进制表示的数，就获得了 $\vec{x}$  的无符号表示。在这个编码中，每个位 x<sub>i</sub> 都取值为 0 或 1，后一种取值意味着数值 2<sup>i</sup> 应为数字值的一部分。我们用一个函数 B2U (Binary to Unsigned 的缩写，长度为 w）来表示：

 —— —— 对于向量 $\vec{x}$ = [x<sub>w-1</sub>,x<sub>w-2</sub>...x<sub>0</sub>]: $B2U_w(\vec{x})≐\sum\limits_{i=0}^{w-1}x_i2^i$

在这个等式中，符号“≐”表示左边被定义为等于右边。

函数 B2Uw 将一个长度为的 0、1 串映射到非负整数。

> 举一个示例，图 2-11 展示的是下面几种情况下 B2U 给出的从位向量到整数的映射：

让我们来考虑一下位所能表示的值的范围。

- 最小值是用位向量 [00… 0] 表示，也就是整数值 0，而最大值是用位向量 [11…1] 表示，也就是整数值。 $UMax_w≐\sum\limits_{i=0}^{w-1}2^i=2^w-1$
- 以4位情况为例，UMax<sub>4</sub>=B2U<sub>4</sub> ([1111]) =2<sup>4</sup>-1=15
- 因此，函数 B2U 能够被定义为一个映射 B2U<sub>w</sub>: {0,1}<sup>w</sup>→{0，…，2<sup>w</sup>-1}。

无符号数的二进制表示有一个很重要的属性，也就是每个介于 0 一 2”一 1 之间的数都有唯一一个位的值编码。例如，十进制值 11 作为无符号数，只有一个 4 位的表示，即【1011]。我们用数学原理来重点讲述它，先表述原理再解释。 

###### 原理：无符号数编码的难一性：函数 $B2U_w$ 是一个双射

数学术语双射是指一个函数 f 有两面：它将数值 x 映射为数值 y，即 y=f (x），但它也可以反向操作，因为对每一个 y 而言，都有唯一一个数值 x 使得 f (x) =y。这可以用反函数 f-1 来表示，在本例中，即 x=f-1 (y）。函数 B2U 将每一个长度为的位向量都映射为 0~2”一 1 之间的一个唯一值；反过来，我们称其为 U2B（即“无符号数到二进制”），在 0~2”一 1 之间的每一个整数都可以映射为一个唯一的长度为的位模式。



### 补码编码 

###### 补码编码的定义

对于向量 $\vec{x}$ = [x<sub>w-1</sub>,x<sub>w-2</sub>...x<sub>0</sub>] ： $B2T_w(\vec{x})≐-x_{w-1}2^{w-1}+\sum\limits_{i=0}^{w-2}x_i2^i$

最高有效位 $x_{w-1}$ 也称为符号位，它的“权重”为 $-2^{w-1}$，是无符号表示中权重的负数。

符号位被设置为 1 时，表示值为负，而当设置为 0 时，值为非负。这里来看一个示例，图 2-13 展示的是下面几种情况下 B2T 给出的从位向量到整数的映射。

![2-13](../statics/csapp/2-13.png)

### 有符号数和无符号数的转换

### C 语言中的有符号数和无符号数

### 扩展一个数字的位表示

### 截断数字

### 关于有符号数和无符号数的建议



## 3 整数运算

### 无符号加法

### 补码加法

### 补码的非

### 无符号乘法

### 补码乘法

### 乘以常数

### 关于整数运算的思考



## 4 浮点数

### 二进制小数

### IEEE 浮点表示

### 数字示例

### 舍入

### 浮点运算

### C 语言中的浮点数

