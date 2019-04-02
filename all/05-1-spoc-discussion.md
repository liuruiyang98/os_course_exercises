#lec11: 进程／线程概念spoc练习

## 视频相关思考题

### 11.1 进程的概念

1. 什么是程序？什么是进程？

   > 答：程序是：对实现预期目的一系列动作的执行过程的描述，由一系列操作指令的序列组成。
   >
   > 　　进程是：一个具有一定独立功能的程序上一个数据集合上的一次动态执行的过程


2. 进程有哪些组成部分？

   > 答：一个进程包含了正在运行的一个程序的所以状态信息。包括代码、数据、状态寄存器、通用寄存器、进程占用的系统资源


3. 请举例说明进程的独立性和制约性的含义。

   > 答：独立性：不同进程的工作不相互影响
   >
   > 　　制约性：因访问共享数据/资源或进程间的同步而产生制约


4. 程序和进程联系和区别是什么？

   > 答：程序和进程的联系是：进程是操作系统处于执行状态程序的抽象，同一个程序的多次执行过程对应不同的进程
   >
   > 　　程序和进程的区别：进程是动态的，程序是静态的；进程是暂时的，程序是永久的；进程与程序的组成不同，进程的组成包括程序、数据和进程控制块。


### 11.2 进程控制块

1. 进程控制块的功能是什么？

   > 答：功能是 管理和控制进程运行


2. 进程控制块中包括什么信息？

   > 答：包括进程标识信息，处理机现场状态，进程控制信息（调度和状态信息，进程间通信信息，存储管理信息，进程所用资源，有关数据结构连接信息）


3. ucore的进展控制块数据结构定义中哪些字段？有什么作用？

   > struct proc_struct {
   >
   > ​    enum proc_state state;                      // Process state
   >
   > ​    int pid;                                                  // Process ID
   >
   > ​    int runs;                                                // the running times of Proces
   >
   > ​    uintptr_t kstack;                                  // Process kernel stack
   >
   > ​    volatile bool need_resched;              // bool value: need to be rescheduled to release CPU?
   >
   > ​    struct proc_struct *parent;               // the parent process
   >
   > ​    struct mm_struct *mm;                     // Process's memory management field
   >
   > ​    struct context context;                      // Switch here to run process
   >
   > ​    struct trapframe *tf;                          // Trap frame for current interrupt
   >
   > ​    uintptr_t cr3;                                       // CR3 register: the base addr of Page Directroy Table(PDT)
   >
   > ​    uint32_t flags;                                     // Process flag
   >
   > ​    char name[PROC_NAME_LEN + 1];  // Process name
   >
   > ​    list_entry_t list_link;                           // Process link list 
   >
   > ​    list_entry_t hash_link;                       // Process hash list
   >
   > };

   
### 11.3 进程状态

1. 进程生命周期中的相关事件有些什么？它们对应的进程状态变化是什么？

   > 答：进程生命周期：进程创建、进程执行、进程等待、进程抢占、进程唤醒、进程结束
   >
   > 　　进程状态：创建、就绪、运行、等待、退出
   >
   > 　　进程状态变迁事件：抢占、唤醒、结束等


### 11.4 三状态进程模型

1. 运行、就绪和等待三种状态的含义？7个状态转换事件的触发条件是什么？

   > 答：运行：程序正在处理机上执行
   >
   > 　　就绪：进程获取了除处理机之外的所有资源，得到处理机即可运行
   >
   > 　　等待：进程正在等待某一事件的出现或某一资源而暂停运行
   >
   > 　　触发条件是：启动、进入就绪队列、被调度、等待事件、时间片用完、事件发生、结束


### 11.5 挂起进程模型

1. 引入挂起状态的目的是什么？

   > 答：目的是：减少进程占用内存，把处在挂起状态的进程映像到磁盘上

2. 引入挂起状态后，状态转换事件和触发条件有什么变化？

   > 答：等待被分成了等待挂起和等待两个状态；就绪被分成了就绪挂起和就绪两个状态；多了三个变迁：激活和挂起、事件发生

3. 内存中的什么内容放到外存中，就算是挂起状态？

   > 答：进程内核栈被放到外存（ucore中没有挂起状态）

### 11.6 线程的概念

1. 引入线程的目的是什么？

   > 答：目的是：在进程之间增加一类实体，满足实体间可以并发执行，实体之间共享相同的地址空间。使得在同一地址空间内实现并发的函数执行

2. 什么是线程？

   > 答：线程是进程的一部分，描述指令流执行状态。它是进程中的指令执行流的最小单元 ，是CPU调度的基本单位。

3. 进程与线程的联系和区别是什么？

   > 答：进程是资源分配单位：进程地址空间、进程占用资源
   >
   > 　　线程是CPU调度单位：CPU寄存器信息、堆栈信息
   >
   > 　　线程 = 进程 - 共享资源

### 11.7 用户线程

1. 什么是用户线程？

   > 答：用户线程是：一组以用户级的线程库函数完成线程的管理，包括线程的创建、终止、同步和调度等

2. 用户线程的线程控制块保存在用户地址空间还是在内核地址空间？

   > 答：保存在用户地址空间


### 11.8 内核线程

1. 用户线程与内核线程的区别是什么？

   > 答：用户线程是由函数库在用户态实现的线程机制；线程控制块保存在用户地址空间
   >
   > 　　内核线程是由内核通过系统调用调用实现的线程机制；线程控制块保存在内核地址空间

2. 同一进程内的不同线程可以共用一个相同的内核栈吗？

   > 答：用户线程是可以共用的

3. 同一进程内的不同线程可以共用一个相同的用户栈吗？

   > 答：不可以共用


## 选做题
1. 请尝试描述用户线程堆栈的可能维护方法。

## 小组思考题
(1) 熟悉和理解下面的简化进程管理系统中的进程状态变化情况。
 - [简化的三状态进程管理子系统使用帮助](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep7-process-run.md)
 - [简化的三状态进程管理子系统实现脚本](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep7-process-run.py)

(2) (spoc)设计一个简化的进程管理子系统，可以管理并调度如下简化进程。在理解[参考代码](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab4/process-concept-homework.py)的基础上，完成＂YOUR CODE"部分的内容。然后通过测试用例和比较自己的实现与往届同学的结果，评价自己的实现是否正确。可２个人一组。

### 进程的状态 

 - RUNNING - 进程正在使用CPU
 - READY   - 进程可使用CPU
 - DONE    - 进程结束

### 进程的行为
 - 使用CPU, 
 - 发出YIELD请求,放弃使用CPU


### 进程调度
 - 使用FIFO/FCFS：先来先服务,
   - 先查找位于proc_info队列的curr_proc元素(当前进程)之后的进程(curr_proc+1..end)是否处于READY态，
   - 再查找位于proc_info队列的curr_proc元素(当前进程)之前的进程(begin..curr_proc-1)是否处于READY态
   - 如都没有，继续执行curr_proc直到结束

### 关键模拟变量
 - 进程控制块
```
PROC_CODE = 'code_'
PROC_PC = 'pc_'
PROC_ID = 'pid_'
PROC_STATE = 'proc_state_'
```
 - 当前进程 curr_proc 
 - 进程列表：proc_info是就绪进程的队列（list），
 - 在命令行（如下所示）需要说明每进程的行为特征：（１）使用CPU ;(2)等待I/O
```
   -l PROCESS_LIST, --processlist= X1:Y1,X2:Y2,...
   X 是进程的执行指令数; 
   Ｙ是执行CPU的比例(0..100) ，如果是100，表示不会发出yield操作
```
 - 进程切换行为：系统决定何时(when)切换进程:进程结束或进程发出yield请求

### 进程执行
```
instruction_to_execute = self.proc_info[self.curr_proc][PROC_CODE].pop(0)
```

### 关键函数
 - 系统执行过程：run
 - 执行状态切换函数:　move_to_ready/running/done　
 - 调度函数：next_proc

### 执行实例

#### 例１
```
$./process-simulation.py -l 5:50
Process 0
  yld
  yld
  cpu
  cpu
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD
Time     PID: 0 
  1     RUN:yld 
  2     RUN:yld 
  3     RUN:cpu 
  4     RUN:cpu 
  5     RUN:yld 

```


#### 例２
```
$./process-simulation.py  -l 5:50,5:50
Produce a trace of what would happen when you run these processes:
Process 0
  yld
  yld
  cpu
  cpu
  yld

Process 1
  cpu
  yld
  cpu
  cpu
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD
Time     PID: 0     PID: 1 
  1     RUN:yld      READY 
  2       READY    RUN:cpu 
  3       READY    RUN:yld 
  4     RUN:yld      READY 
  5       READY    RUN:cpu 
  6       READY    RUN:cpu 
  7       READY    RUN:yld 
  8     RUN:cpu      READY 
  9     RUN:cpu      READY 
 10     RUN:yld      READY 
 11     RUNNING       DONE 
```
