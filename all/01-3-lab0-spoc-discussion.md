# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

　　答：在硬件设计上至少需要  
　　　　至少需要提供特权指令类型及功能如下：  
　　　　　1. Exceptions：LIDT, LTR, IRET, STI, CLI（中断/异常/系统服务等管理指令）  
　　　　　2. Virtual Memory: MOV CRn, INVLPG, INVPCID（TLB/MMU等管理指令）  
　　　　　3. Privilege Modes: SYSRET, SYSEXIT, IRET（调整特权级管理指令）  
　　　　　4. Segmentation/Paging: LGDT, LLDT CRx: CR0,CR3（分段分页管理指令）....

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

　　答：实模式和保护模式的区别有：  
　　　　1. x86的实模式是CPU启动的时候的模式，这时候就相当于一个速度超快的8086；而保护模式是操作系统接管CPU后CPU进入的模式。  
　　　　2. 二者能够访问的物理内存空间大下不同。实模式下软件可访问的物理内存空间不能超过1M；而保护模式下可访问4G的物理内存空间(32条地址线)。  
　　　　3. 实模式下指针访问的是实际的物理地址，这样系统进程内存不受保护；而保护模式下物理内存不能直接被程序访问，其指针指向的虚拟地址都将由操作系统转换为实际物理地址再执行访问，实现了对进程内存的保护。  
　　　　4. 实模式下不能使用多线程，也不能实现权限分级；而保护模式下支持内存分页机制，提供对虚拟内存的良好支持，还支持优先级机制和良好的检查机制。  

　　　　物理地址：物理内存地址空间是处理器提交到总线上用于访问计算机系统中的内存和外设的最终地址，一个计算机系统只有一个物理地址空间。  
　　　　线性地址：线性地址空间是在操作系统的虚存管理之下每个运行的应用程序能访问的地址空间。每个运行的应用程序都认为自己独享整个西算计系统的地址空间，这样可以让多个运行的应用程序之间相互隔离。  
　　　　逻辑地址：逻辑地址空间是应用程序直接使用的地址空间。  

- 你理解的risc-v的特权模式有什么区别？不同模式在地址访问方面有何特征？

　　答：RISC-V架构定义了三种工作模式，又称特权模式（Privileged Mode）：  
　　　　1. Machine Mode：机器模式，简称M Mode。  
　　　　2. Supervisor Mode：监督模式，简称S Mode。  
　　　　3. User Mode：用户模式，简称U Mode。  
　　　　RISC-V架构定义M Mode为必选模式，另外两种为可选模式。通过不同的模式组合可以实现不同的系统。

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

　　答：这些数字表示该成员变量的位宽，即其所占的二进制位数(bit)。如 **gd_off_15_0** 表示段中偏移量低16位，其后面的 **16** 表示其占位宽为16。

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

　　答：执行上述指令后，intr的值为 0x10002。  

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
