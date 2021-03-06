---
title: S081 RISC-V Book en Ch
categories: [mit_opencourseware]
tags: [S081, Operating System]
---

# 第2章 操作系统的组织形式

操作系统的一个关键要求是要一次支持多个活动。比如，使用系统调用fork，一个进程可以启动新进程。

操作系统必须在这些进程间**分时共享**计算机资源。比如，即使系统中的进程数比硬件CPU多，则操作系统也必须保证所有进程都有执行的机会。

操作系统必须安排进程之间的**隔离**。即， 如果一个进程里有错误，并出现故障，则不应影响不依赖该进程的那些进程。但是，彻底地隔离又太强了，因为进程间可能存在交互，比如管道。

因此，操作系统必须满足三个要求：**多路复用**、**隔离**、**交互**。

本章概述了如何组织操作系统来实现前述3个要求。

虽然已证明有很多方式来实现这3个要求，但是本文重点关注有关整体内核的主流设计，比如许多Unix操作系统使用的就是这种设计。

本章也概述了xv6进程，其中进程是xv6系统的隔离单元。
本章也概述了当xv6启动时，第一个进程是如何创建的。

xv6在一台多核的RISC-V的微处理器上运行，许多xv6的底层功能都是特定于RISC-V的。RISC-V是一个64位的CPU，xv6是用`LP64`C编写的，表示在C语言里的long和pointer是64位的，而不是32位。

本书假定读者了解在某种体系架构上的一些机器级编程，并会介绍RISC-V特定的思想。有关RISC-V的有用的参考是《The RISC-V Reader: An Open Architecture Atlas》。

在一台完整的电脑上的CPU是被支持硬件包围着的，其中的大部分都是I/O接口。通过qemu的`-machine virt`选项，可为支持硬件编写xv6。这里的支持硬件包括RAM、一个包含启动代码的ROM、用户键盘/屏幕的串行连接、一个存储用的磁盘。

## 2.1 对物理资源进行抽象

为什么会有操作系统？

如果没有操作系统，则可以通过类库的形式来实现系统调用，应用通过链接到类库的方式来使用类库。
使用类库的方式实现系统调用，每个应用甚应用至可以根据自身需要量身定制自己的类库。
使用类库的方式来实现系统调用，应用能直接跟硬件资源交互，以对应用最优的方式来使用硬件资源。
一些嵌入式设备或者实时系统采用这种组织形式。
类库方法的缺点是：如果有多个应用在运行，则这些应用必须有良好的行为，比如每个应用必须周期性地放弃CPU，以便其他应用可以运行。如果所有的应用都彼此信任，且没有错误，则这种协作式分时共享策略是可行的。由于对于应用来说，更常见的是彼此不信任，或者有错误。所以我们想要的隔离度要比协作式策略更强。

换句话说，协作式策略需要的隔离是弱隔离。

禁止应用直接访问敏感的硬件资源，转而将资源抽象为服务，对实现强隔离是有帮助的。
比如，UNIX应用只能通过文件系统的`open`、`read`、`write`、`close`等系统调用来跟存储交互，而不是直接读写磁盘。UNIX的文件系统为应用提供了路径名，且允许操作系统来管理磁盘。

即使不关心隔离特性，那些存在交互或者只希望彼此保持隔离的程序也可能会发现：文件系统是一个比直接使用磁盘更方便的抽象。

类似地，通过按需保存和恢复寄存器状态，UNIX透明地在多个进程间切换硬件CPU，使得应用自身意识不到分时共享。这种透明性允许：即使有应用陷入死循环，则操作系统仍可以共享CPU。

比如，UNIX进程使用`exec`来构建它的内存映像，而不是直接跟物理内存交互。这样就允许：操作系统决定进程在内存中的放置位置。如果内存不够用的话，则操作系统甚至会将进程的数据保存在磁盘上。除此之外，`exec`为用户提供了方便的文件系统来存储可执行程序映像。

UNIX进程间的许多进程的交互方式都是通过文件描述符发生的。文件描述符不仅抽象了许多细节(比如数据是存储在管道里还是文件里等)，而且还是以简化交互的形式定义的。比如，如果在管道中的一个应用失败了，则内核会为管道中的下一个进程生成一个`end-of-file`的信号。

为了既为程序员提供方便，也提供强隔离，则在图1.3中的系统调用接口是经过仔细设计的。

虽然UNIX接口不是唯一地抽象资源方式，但已被证明是一个很好的设计。

## 2.2 用户模式、内核模式及系统调用

强隔离要求应用和操作系统之间有一个明确的边界。如果应用出错了，我们不想让操作系统也崩溃，或者让其他应用崩溃。

操作系统应该能清理出错的应用，继续运行其他应用。

为了达到强隔离，则操作系统必须安排好使得应用不能修改操作系统的数据结构和指令，不能访问其他进程的内存。

CPU为强隔离提供硬件上的支持，比如，RISC-V有3种CPU执行指令的模式：机器模式，超级用户模式，用户模式。
在机器模式下执行指令具有完整的特权。机器模式主要用于配置计算机。
xv6在机器模式下执行若干行指令，然后切换到超级用户模式。

在超级用户模式下，允许CPU执行特权指令，比如使中断生效和失效、读写持有页表地址的寄存器等。
如果处于用户模式的应用尝试去执行一个特权指令，则CPU不会执行该指令，而是切换到超级用户模式，使得超级用户代码能终止该应用，因为应用做了本不应该它做的事。
第一章中的图1.1说明了这种组织方式。

应用只能执行用户模式的指令(比如，将数字相加等)，称该软件在用户控件运行。

在超级用户模式下的软件也能执行特权指令，称该软件在内核空间运行。
在内核空间中运行的软件成为内核。

想调用内核功能(比如xv6中的`read`系统调用)的的应用必须要切换到内核。
CPU提供了一个特定指令(RISC-V提供了`ecall`指令)，该指令的功能是从用户模式切换到超级用户模式的特定指令，进入内核的指定位置。
一旦CPU切换到了超级用户模式，则内核就能验证该系统调用的参数，决定是否允许应用执行请求操作，拒绝执行或者执行。

重要的是内核控制着切换到超级用户模式的入口点。如果应用能决定内核的进入点，则一个恶意的应用就能在一个跳过验证参数的点进入内核。

## 2.3 内核的组织结构

一个很关键的设计问题：操作系统的哪些部分应该以超级用户模式运行？

思路1：宏内核monolithic kernel

整个操作系统都驻留在内核，使得所有的系统调用都是在超级用户模式下运行。

在整体内核中，整个操作系统运行时都具有完整的硬件特权。

优点：

这样的组织很方便，因为

- 操作系统的设计者不用判断操作系统的哪一部分不需要完整的特权。
- 操作系统的各个部分之间协作起来会更容易，比如操作系统可能有一个被文件系统和虚拟内存系统共用的缓冲区缓存。

缺点：

操作系统的不同部分之间的接口通常会很复杂，从而导致操作系统的开发者很容易就犯错。

在整体内核中，出一个错就是致命的，因为在超级用户模式下的一个错误通常会导致内核崩溃。如果内核崩溃了，则计算机将停止工作，进而所有应用都失败了。计算机必须重新启动了。

思路2：微内核microkernel

为了降低内核中出错的风险，操作系统的设计者要

- 最小化能以超级用户模式运行的操作系统代码的数量；
- 以用户模式执行操作系统剩余的大部分代码；

图2.1说明了这种微内核设计。
作为进程运行的OS服务称为服务器，文件系统作为一个用户级别的进程在运行。

为了允许应用跟文件服务器交互，内核提供了进程间通讯机制来从一个处于用户模式的进程向另一个发送消息。
比如，如果shell想读写一个文件，则shell进程就会给文件服务器发送一个消息，并等待响应。
在微内核中，内核接口由若干个底层功能构成，比如开始一个应用、发送消息、访问硬件设备等。
这种组织形式允许内核可以相对地小，大部分操作系统驻留作为用户级别的服务器。

跟大部分Unix操作系统一样，xv6采用宏内核的方式来实现。因此，xv6的内核接口对应的是操作系统接口，内核实现了完整的操作系统。
由于xv6没有提供许多服务，因此跟一些微内核相比，xv6的内核相对较小，但是从概念上讲，xv6是宏内核。

## 2.4 xv6的代码组织结构

xv6的内核源代码是在`kernel`子目录下。遵循粗略的模块化概念，它被分成许多文件，如图2.2所示。

模块间接口被定义在`defs.h`中。

## 2.5 进程简介

xv6里的一个隔离单元就是一个进程。

进程抽象可以防止一个进程破坏或者监视另一个进程的内存、CPU、文件描述符等。
进程抽象还可以防止一个进程破坏内核，以便一个进程不能颠覆内核的隔离机制。

内核必须仔细地实现隔离机制，因为一个出错的或者恶意的应用会诱使操作系统做一些不好的事情，比如绕过隔离等。

内核用来实现进程的机制包括用户/超级用户模式标记、地址空间、线程的时间分片等。

为了有助于加强隔离度，进程抽象给程序提供了一种拥有自己专属机器的错觉。

进程不仅给程序提供了不能被其他进程读写的私有内存空间，即地址空间。
进程而且给程序提供了看起来似乎是自己的CPU来执行程序指令。

xv6使用页表(通过硬件来实现)来给予每个进程自己的地址空间。
RISC-V页表将虚拟地址(RISC-V指令能操作的地址)映射到物理地址(CPU芯片发送给主内存的地址)。

xv6为每个进程维护一张单独的页表，该页表定义了进程的地址空间。
如图2.3所示，一个地址空间包括进程的用户内存(从0开始)：

- 指令
- 全局变量
- 栈
- 堆
- trampoline
- trapframe

有若干个限制进程地址空间大小的因素：在RISC-V上的指针是64位的；硬件仅使用低39位去页表中查询虚拟地址；xv6仅使用这39位中的38位。

因此，最大的地址是![2^{38}-1=0x3fffffffff](https://math.jianshu.com/math?formula=2%5E%7B38%7D-1%3D0x3fffffffff)，即MAXVA(定义可见`kernel/risv.h`)。

在地址空间的顶部，xv6预留了一页用于trampoline，一页用于映射进程的trapframe来切换到内核。

这里留了两个问题：

- 什么是trampoline？
- 什么是trapframe？

xv6内核为每个进程维护了许多状态片段，且将这些状态收集在了结构体`struct proc`(定义见`kernel/proc.h`)中。

进程最重要的内核状态片段就是页表、内核栈、运行状态。

每个进程有一个执行线程来执行该进程的指令。
一个线程能被挂起，然后恢复。

为了在进程间透明地切换，内核会挂起当前运行线程，恢复另一个进程的线程。

线程的许多状态是保存在线程栈上的，比如局部变量、函数调用的返回地址等。

每个进程有两个栈：用户栈和内核栈(`p->kstack`)。
当进程在执行用户指令时，只有用户栈在使用，内核栈是空的。
当进程进入内核，可能是系统调用或者中断，内核代码在内核栈上执行。注意，当一个进程处于内核中时，用户栈中仍保存有数据，只是没被激活使用而已。
一个进程的线程可交替使用用户栈和内核栈。
内核栈是跟用户代码隔离开的，以便及时用户栈被破坏了，内核仍可以执行。

一个进程通过执行RISC-v的`ecall`指令来进行系统调用。

- 该指令提升硬件特权等级，修改程序计数器到某个内核定义的入口点。内核入口点的代码切换到一个内核栈，执行实现了该系统调用的内核指令。

当系统调用完成时，内核通过调用`sret`指令来切换回用户栈，和返回到用户空间。该指令降低硬件特权等级，恢复执行紧邻系统调用指令的那条用户指令。

一个进程的线程可以在内核中因为等待I/O而阻塞，当I/O完成时，该线程就在挂起的地方开始执行。

`p->kstack`：进程的内核栈。

`p->state`表示进程的状态，包括已分配、待运行、运行中、等待I/O、退出等。

`p->pagetable`持有的是进程的页表，格式要满足RISC-V硬件的要求。
xv6使得：在用户空间中执行进程时，分页硬件使用进程的`p->pagetable`。
进程的页表也记录用于存储进程内存的物理页的地址。



# 第4章 陷阱和系统调用

有3类事件可导致CPU把普通的指令执行搁置在一边，强制把控制权转移到能处理事件的特定代码处。

- 系统调用
  用户程序执行`ecall`指令来请求内核为它做一些事；
- 异常
  一条指令(用户或者内核)做了非法的事，比如除以0、使用了一个非法的虚拟地址等；
- 设备中断
  设备发出了需要关注的信号，比如磁盘完成了读或者写操作等

本书中使用**陷阱trap**作为这3种情形的泛称。

当陷阱出现时，无论正在执行什么代码都需要恢复，不应该感知到发生了任何特殊的事件。

即，我们通常希望陷阱是透明的，这对于中断来说极其重要，因为被中断的代码不期望感知到有特殊事件发生了。

通常的步骤是：

- 陷阱强制将控制权转交给内核；
- 内核保存寄存器及其他状态，使得执行能被恢复；
- 内核执行合适的处理代码(比如，系统调用的实现或者设备驱动)；
- 内核恢复被保存的状态，从陷阱中返回；
- 从被打断处恢复原始代码的执行；

xv6内核处理所有类型的陷阱。

- 对系统调用来说，这是很自然的事。
- 对中断来说，也是很有意义的，因为隔离性要求：用户进程不能直接访问设备，只有内核有处理设备所需的状态。
- 对异常来说，也很有意义，因为xv6对来自用户空间的所有异常的作出的响应是杀掉相应的进程。

xv6的陷阱处理分为4个阶段：

- RISC-V的CPU采取的硬件操作；
- 1个为内核C代码准备好路径的汇编向量；
- 1个决定如何处理陷阱的C陷阱处理程序；
- 系统调用或者设备驱动服务例程；

虽然这3类陷阱之间的共性建议：内核可使用一条代码路径来处理所有类型的陷阱，但是事实证明，**针对用户空间陷阱、内核空间陷阱、计时器中断等3种不同的情形，有单独的汇编向量及C陷阱处理程序是很方便的**。



## 4.1 RISV的陷阱机制

每个RISC-V的CPU都有一套控制寄存器，内核可向其中写入信息来告知CPU如何处理陷阱，内核可从中读数据来查找有关已发生的陷阱信息。

在`riscv.h`中包含了xv6使用的定义。

| Register       | Function                                                     |
| -------------- | ------------------------------------------------------------ |
| stvec寄存器    | 内核向其中写入中断处理程序的地址；<br/>RISC-V将跳转到这里记录的地址处理陷阱； |
| sepc寄存器     | 当陷阱发生时，RISC-V将程序计数器的值保存在这里，因为随后pc的值将被stvec的值覆盖掉；<br/>sret指令拷贝sepc的值到pc中；<br/>内核可向spec中写入值来控制sret返回到哪里； |
| scause寄存器   | RISC-V在这里放入一个数，描述的是陷阱发生的原因；             |
| sscratch寄存器 | 内核在这里放置一个值，这个值会在处理程序开始时很有用；       |
| sstatus寄存器  | 在sstatus中的SIE位控制的是设备中断是否生效；<br/>如果内核清除了SIE，则RISC-V将延迟设备中断直到内核设置了SIE。<br/>在sstatus中的SPP位记录的是陷阱来自用户模式还是超级用户模式，及控制sret返回到哪种模式。 |

> 以上5个跟陷阱相关的寄存器都是在超级用户模式下处理的，在用户模式下不能读写这5个寄存器的值。

在机器模式下，有等价的一套控制寄存器来用于陷阱处理。

xv6仅在计时器中断的特殊情况下使用这些寄存器。

在多核处理器的每个CPU都有自己的一套类似这样的寄存器；在任意给定时刻，可能不止有一个CPU在处理陷阱。

当需要强制处理一个陷阱时，RISV 硬件对除了计时器中断外的所有陷阱类型做下面几件事：

1. 如果陷阱是一个设备中断，且sstatus中的SIE标记位被清除了，则什么都不做；
2. 清除sstatus中的SIE标记，使中断失效；
3. 拷贝pc到spec；
4. 保存当前模式到sstatus的SPP标记位；
5. 设置scause寄存器来反映陷阱的起因；
6. 设置模式为超级用户模式；
7. 拷贝stvec到pc
8. 跳转到新的pc处开始执行

**注意：**CPU没有切换到内核的页表，没有切换到内核栈中，没有保存除了pc之外的任何寄存器。这些是内核软件必须要做的任务。

**理由：**CPU在处理陷阱的过程中做少量的工作是为了给软件提供更大的灵活性。比如，一些操作系统在某些情况下不需要页表切换的，这可以提升性能。

能不能对CPU的陷阱处理步骤进行进一步的简化？

- 假设CPU不切换程序寄存器pc。则还在运行用户指令，陷阱就切换到超级用户模式了。这样，那些用户指令就能破坏用户/内核隔离机制了，比如通过修改satp寄存器来指向允许访问整个物理内存的页表了。
- 因此，CPU切换到由stvec寄存器指定的内核指令地址是非常重要的。



## 4.2 来自用户空间的陷阱

- 问题：一次完整的来自用户空间的陷阱处理流程是怎样的？
- 问题：`uservec`做了哪些事？
- 问题：`usertrap`做了哪些事？
- 问题：`usertrapret`做了哪些事？
- 问题：`userret`做了哪些事？

当CPU在用户空间执行时，如果用户程序做了一个系统调用，或者做了非法的事，或者某个设备中断了，则就可能会发生一个陷阱。

处理来自用户空间的陷阱的代码路径是先`uservec`，后`usertrap`。
返回时是先`usertrapret`，后`userret`。

来自用户空间的陷阱处理代码要比来自内核的更具有挑战性：因为`satp`指向的是一个没有映射到内核的用户页表，栈指针可能包含一个无效甚至是恶意的值。

因为RISC-V硬件在陷阱期间不切换页表，则用户页表必须包含`uservec`的映射，即`stvec`指向的陷阱向量指令。

`uservec`必须切换`satp`以指向内核页表；为了在切换后继续执行指令，必须将`uservec`映射到内核页表中跟用户页表中相同的地址。

xv6使用包含`uservec`的`trampoline页`来满足这些约束。xv6将`trampoline页`映射到内核页表和每个用户页表中的虚拟地址是相同的。这个虚拟地址就是`TRAMPOLINE`。

在`trampoline.S`中设置了`trampoline`的内容。

当执行用户代码时，设置`stevec`为`uservec`。

当uservec开始执行时，所有32个寄存器包含的都是被打断代码的值。
但是，为了设置satp及生成保存寄存器值的地址，uservec需要能修改一些寄存器。

RISC-V以sscratch寄存器的形式提供了一个帮手。
在uservec开头的csrrw指令交换了a0寄存器和sscratch寄存器的值。

现在用户代码的a0寄存器的值被保存了；
uservec有一个寄存器a0可以使用了；
ao包含了内核先前放在sscratch里的值。

uservec的下一个任务是保存用户寄存器的值。
在进入用户空间前，内核之前设置sscratch指向每个进程的trapframe，该trapframe有空间来保存所有的寄存器。
由于satp仍指向用户页表，则uservec需要将trapframe映射到用户地址空间。

当创建进程时，xv6会分配一个页给该进程的trapframe，将该页映射到用户虚拟地址TRAPFRAME。进程的p->trapframe指向的就是陷阱栈，不过是它的物理地址，以便内核能通过内核页表来使用它。

因此，在交换了a0和sscratch后，a0持有了一个指向当前进程陷阱栈的指针。现在，uservec可以将所有的用户寄存器保存到陷阱栈中，包括用户的a0。

陷阱栈包含了指向当前进程内核栈的指针、当前CPU的hartid、usertrap的地址、内核页表的地址。

uservec检索这些值，切换satp指向内核页表，调用usertrap。

usertrap的工作是判断陷阱的起因，处理陷阱，并返回。
首先，修改stvec，使得陷阱在内核中将被kernelvec处理。
然后，保存sepc的值，因为在usertrap中的一个进程切换可能会导致sepc的值被覆盖。
接着，如果一个陷阱是系统调用，则syscall会处理陷阱；
如果陷阱是一个设备中断，则devintr会处理陷阱；
如果陷阱是一个异常，则内核会杀掉出错的进程；

系统调用路径会将保存的用户pc加4，因为在系统调用情形下，RISC-V会让程序指针指向ecall指令。

退出过程中，usertrap会检查进程是否被杀死，或者应该让出CPU(如果这是一个计时器中断)。

如何返回到用户空间？

第一步：调用usertrapret。
该函数先设置RISC-V的控制寄存器为将来的来自用户空间的陷阱做好准备。包括：设置stvec指向uservec，准备好uservec依赖的trapframe字段，设置sepc为先前保存的用户pc。
然后，调用在trampoline页上的userret，该trampiline页在用户页表和内核页表中都有映射，理由是在userret中的汇编码将切换页表。

usertrapret对userret的调用传递了一个指向进程用户页表(在a0中)和TRAPFRAME(在a1中)的指针。

userret切换satp指向进程的用户页表。
回想一下，用户页表既映射了trampoline页，也映射了TRPFRAME，但没有映射来自内核的其他内容。
在用户页表和内核页表中，trampoline页被映射到相同的虚拟地址，这就允许：在修改satp之后，uservec继续执行。

userret拷贝trapframe上保存的用户a0到sscratch中，为以后的TRAPFRAME交换做好准备。

从现在起，userret能使用的数据只有寄存器的内容和trapframe的内容。
接下来，userret从trapframe中恢复被保存的寄存器，做最后一次a0和sscratch的交换来恢复a0，为接下来的陷阱而保存TRAPFRAME，使用sret返回到用户空间。



## 4.3 代码：系统调用

第2章以initcode.S结束，在initcode.S中触发了exec系统调用。让我们看看用户的调用是如何抵达内核中的exec系统调用实现的。

用户代码在寄存器a0和a1中放入了用于exec的参数，在寄存器a7中放入了系统调用编号。

系统调用编号是跟syscalls数组的条目相匹配的，其中syscalls是一个由多个函数指针组成的表。

ecall指令进入到内核，执行uservec、usertrap，然后是syscall。

syscall从在trapframe上被保存的寄存器a7的值中检索出系统调用号，使用它作为syscalls的索引。寄存器a7包含的值为SYS_exec，导致调用系统调用实现sys_exec。

当从系统调用实现函数中返回时，syscall在p->trapframe->a0中记录返回值。由于在RISC-V上的C调用约定在寄存器a0中放入返回值，则前述操作会导致初始用户空间对exec()调用返回syscall在trapframe->a0中放入的值。

系统调用约定返回负数表示错误，0或者整数表示成功。

如果系统调用号非法，则syscall会输出错误，并返回-1。



## 4.4 代码：系统调用参数

寄存器->陷阱帧trapframe

内核中的系统调用实现需要找到用户代码传递的参数。

因为用户代码调用的是经过包装的系统调用函数，所以参数最开始是按照RISC-V的C调用约定，保存在寄存器中的。

内核的陷阱trap代码将寄存器的值保存到当前进程的陷阱帧trap frame中，内核代码是从当前进程的trapframe上找到系统调用参数的。

函数argint、argaddr、argfd分别从陷阱帧上检索第n个参数作为整数、指针、或者文件描述符。这3个函数都是调用argraw来检索到合适的被保存的用户寄存器。

有些系统调用传递指针作为参数，内核必须使用这些指针来读写用户内存。比如，系统调用exec传递给内核一个指针数组，引用的是在用户空间的字符串参数。

这些指针带来两个挑战：

1. 用户程序可能会出错或者有恶意，可能会传递给内核一个无效的指针、或者旨在骗过内核来访问内核的内存代替访问用户内存。
2. xv6的内核页表映射跟用户页表映射不一样，所以内核不能使用普通的指令来加载或者保存来自用户提供的地址。

内核实现了向用户提供的地址以及从用户提供的地址安全转移数据的功能，比如`fetchstr`函数。

诸如`exec`等文件系统调用使用fetchstr函数来从用户空间检索字符串式的文件名参数。

fetchstr函数调用copyinstr来做实际的工作。

copyinstr从用户页表pagetale中的虚拟地址srcva处最多拷贝max个字节。它使用walkaddr来遍历软件形式的页表来确定srcva对应的物理地址pa0。因为内核映射所有的RAM地址到相同的内核虚拟地址，所以copyinstr可以直接从pa0拷贝字符串字节到dst。

walkaddr会检查用户提供的虚拟地址是否属于用户地址空间，所以程序不能骗过内核去读其他内存。

类似的函数还有copyout：将数据从内核拷贝到用户提供的地址。



## 4.5 内核空间中的陷阱

来自内核空间的陷阱处理步骤

- 保存寄存器
- 处理陷阱
- 从陷阱中返回

根据是在用户空间执行代码还是在内核空间执行代码，xv6配置CPU陷阱寄存器的方式是不一样的。

当内核在CPU上执行时，内核将stvec指向在kernelvec处的汇编代码。

由于xv6已经在内核里了，kernelvec能依赖stap来设置内核页表和栈指针来引用有效的内核栈。

kernelvec保存所有的寄存器，以便被打断的代码最终能无扰动地恢复执行。

kernelvec将寄存器保存在被打断的内核线程的栈上，这是有意义的，因为此时这些寄存器的值是属于该内核线程的。如果陷阱导致切换到一个不同的线程，这一点非常重要，陷阱将实际返回到新线程的栈上，同时安全地把被打断线程的寄存器保存在自己的栈上。

在保存完寄存器后，kernelvec就跳转到kerneltrap。

kerneltrap为两类陷阱做好准备：设备中断和异常。
它调用devintr来检查和处理设备中断。如果陷阱不是设备中断，则一定是异常；如果在xv6内核中发生了异常，则通常是一个致命错误；内核调用panic并停止执行。

如果由于计时器中断调用了kerneltrap，且一个进程的内核线程正在运行，则kerneltrap调用yield来给其他进程一个运行的机会。在将来的某个时间点，总有某个线程会yied，让我们的线程及它的kerneltrap恢复执行。

当kerneltrap执行完后，需要返回到被陷阱打断的代码处。kerneltrap检索那些控制寄存器的值，返回给kernelvec。

kernelvec从栈上弹出保存的寄存器值，并执行sret；sret拷贝sepc到pc，恢复执行被打断代码的值。

由于一个yield会干扰spec和sstatus，所以kernel在一开始就先保存spec和sstatus的值。

非常值得思考的一个问题是：如果kerneltrap由于计时器中断调用了yield，则陷阱返回是如何发生的？

当CPU从用户空间进入内核，xv6设置该CPU的stvec指向kernelvec。
当内核正在执行，但stvec指向uservec时，存在一个时间窗口。
在这个时间窗口内，使设备中断失效时非常关键的。

幸运的是，RISC-V在取一个陷阱的开始就让中断失效，直到xv6设置了stvec后，才让中断生效。