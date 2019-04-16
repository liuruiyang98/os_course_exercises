# lec14: lab5 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。


## 视频相关思考题

### 14.1 总体介绍

1. 第一个用户进程创建有什么特殊的？

   > 答：需要进行用户态代码段的初始化

2. 系统调用的参数传递过程？

   > 答：参见：用户态函数syscall()中的汇编代码；
   >
   > 　　Ref: https://www.ibm.com/developerworks/library/l-ia/index.html

3. getpid的返回值放在什么地方了？

   > 答：参见：用户态函数syscall()中的汇编代码；

### 14.2 进程的内存布局

1. ucore的内存布局中，页表、用户栈、内核栈在逻辑地址空间中的位置？

   > 答：见 memlayout.h
   >
   > 　　\#define VPT                 0xFAC00000
   >
   > 　　#define KSTACKPAGE          2                           // # of pages in kernel stack
   >
   > 　　#define KSTACKSIZE          (KSTACKPAGE * PGSIZE)       // sizeof kernel stack
   >
   > 　　#define USERTOP             0xB0000000
   >
   > 　　#define USTACKTOP           USERTOP
   >
   > 　　#define USTACKPAGE          256                         // # of pages in user stack
   >
   > 　　#define USTACKSIZE          (USTACKPAGE * PGSIZE)       // sizeof user stack
   >
   > 　　#define USERBASE            0x00200000
   >
   > 　　#define UTEXT               0x00800000                  // where user programs generally begin
   >
   > 　　#define USTAB               USERBASE                    // the location of the user STABS data structure

2. (spoc)尝试在panic函数中获取并输出用户栈和内核栈的函数嵌套信息和函数调用参数信息，然后在你希望的地方人为触发panic函数，并输出上述信息。

3. (spoc)尝试在panic函数中获取和输出页表有效逻辑地址空间范围和在内存中的逻辑地址空间范围，然后在你希望的地方人为触发panic函数，并输出上述信息。

4. 尝试在进程运行过程中获取内核空间中各进程相同的页表项（代码段）和不同的页表项（内核堆栈）？

### 14.3 执行ELF格式的二进制代码-do_execve的实现

1. 在do_execve中的的当前进程如何清空地址空间内容的？在什么时候开始使用新加载进程的地址空间？

   > 答：清空进程地址空间是在initproc所在进程地址空间
   >
   > 　　CR3设置成新建好的页表地址后，开始使用新的地址空间

2. 新加载进程的第一级页表的建立代码在哪？

   > 答：boot_cr3？ lcr3(boot_cr3);-------do_execve
   >
   > 　　    current->cr3 = PADDR(mm->pgdir);
   >
   >    　　 lcr3(PADDR(mm->pgdir)); —————— load_icode

3. do_execve在处理中是如何正确区分出用户进程和线程的？并为此采取了哪些不同的处理？

   > 答：线程的mm为空，进程的mm不为空。对于进程需要设置其页表为内核态页表，如果该进程没有线程了，则需要进行回收。
   >
   > if (mm != NULL) {
   >
   > ​        lcr3(boot_cr3);
   >
   > ​        if (mm_count_dec(mm) == 0) {
   >
   > ​            exit_mmap(mm);
   >
   > ​            put_pgdir(mm);
   >
   > ​            mm_destroy(mm);
   >
   > ​        }
   >
   > ​        current->mm = NULL;
   >
   > ​    }

### 14.4 执行ELF格式的二进制代码-load_icode的实现

1. 第一个内核线程和第一个用户进程的创建有什么不同？

   > 答：相应线程的内核栈创建时，多了SS和ESP的设置；
   >
   > 　　用户进程需要创建用户地址空间，并把用户代码复制到用户地址空间；

2. 尝试跟踪分析新创建的用户进程的开始执行过程？

   > 答：见我的lab5 实验报告

### 14.5 进程复制

1. 为什么新进程的内核堆栈可以先于进程地址空间复制进行创建？

   > 答：内核栈在进程的内核地址空间，而各进程的内核地址空间是共享的；

2. 进程复制的代码在哪？复制了哪些内容？

   > 答：do_fork

3. 进程复制过程中有哪些修改？为什么要修改？

   > 答：内核栈、页表、trapframe、context、PCB字段修改

4. 分析第一个用户进程的创建流程，说明进程切换后执行的第一条是什么。

   > 答：forkret 函数

### 14.6 内存管理的copy-on-write机制

1. 什么是写时复制？

   > 答：写时拷贝是一种可以推迟甚至避免拷贝数据的技术。内核此时并不复制整个进程的地址空间，而是让父子进程共享同一个地址空间。只有在需要写入的时候才会复制地址空间，从而使各个进行拥有各自的地址空间。也就是说，资源的复制是在需要写入的时候才会进行，在此之前，只有以只读方式共享。这种技术使地址空间上的页的拷贝被推迟到实际发生写入的时候。

2. 写时复制的页表在什么时候进行复制？共享地址空间和写时复制有什么不同？

   > 答：只有在需要写入的时候才会复制地址空间。共享地址空间在写入时不需要复制。

3. 存在有多个（n>2）进程具有父子关系，且采用了COW机制的情况。这个情况与只有父子两个进程的情况相比，在设计COW时，需要注意的新问题是什么？有何解决方案？

   > 答：新问题是如果只有父子两个进程写入的时候会复制地址空间，同时两个页面都可以赋值为可读可写。存在有多个（n>2）进程具有父子关系，写入使只是把复制的页面设置为可写，其余的依然是可读。同时增加引用计数来标记是否还有共用改页面的进程。


## 小组练习与思考题

### (1)(spoc) 在真实机器的u盘上启动并运行ucore lab,

请准备一个空闲u盘，然后请参考如下网址完成练习

https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-boot-with-grub2-in-udisk.md

> 注意，grub_kernel的源码在ucore_lab的lab1_X的git branch上，位于 `ucore_lab/labcodes_answer/lab1_result`

(报告可课后完成)请理解grub multiboot spec的含义，并分析ucore_lab是如何实现符合grub multiboot spec的，并形成spoc练习报告。

### (2)(spoc) 理解用户进程的生命周期。

> 需写练习报告和简单编码，完成后放到网络学堂 OR git server 对应的git repo中

### 练习用的[lab5 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab5/lab5-spoc-discuss)


#### 掌握知识点
1. 用户进程的启动、运行、就绪、等待、退出
2. 用户进程的管理与简单调度
3. 用户进程的上下文切换过程
4. 用户进程的特权级切换过程
5. 用户进程的创建过程并完成资源占用
6. 用户进程的退出过程并完成资源回收

> 注意，请关注：内核如何创建用户进程的？用户进程是如何在用户态开始执行的？用户态的堆栈是保存在哪里的？

阅读代码，在现有基础上再增加一个用户进程A，并通过增加cprintf函数到ucore代码中，
能够把个人思考题和上述知识点中的内容展示出来：即在ucore运行过程中通过`cprintf`函数来完整地展现出来进程A相关的动态执行和内部数据/状态变化的细节。(约全面细致约好)

请完成如下练习，完成代码填写，并形成spoc练习报告
