---
title: Some notes on "Operation System"
categories:
  - Note
tags:
  - Operation System
  - Baoyan
show_date: true
toc: true
toc_label: On this page
toc_icon: "file-alt"
---

# Notes

## Process & thread & IPC

### Process

- 可执行程序在一个数据集合上的一次活动，因此是动态的（进程包括：创建、调度、消亡）
- type：
  - foreground process：用户创建
  - background process(daemon)：不受终端控制
- 进程创建的时机：
  - reboot
  - 一次任务的启动
  - fork
- 进程调度
- 进程消亡
  - 正常退出
  - 例如因编译错误的主动错误退出
  - 被动错误退出
  - killed
- implementation：
  - 包括三大部分：PCB（状态信息、处理机状态、调度信息、控制信息）、Stack、Virtual Address Space（program and data，由虚表控制）
  - PCB table：存放了所有能操控的进程的PCB，PCB的大小代表了OS的并发度
  - process context
    - 保存以前的PCB
    - 加载进程调度所需要执行的PCB
    - cache
    - 虚拟地址映射转换（开销很大）



### Thread

- why：提高程序的并发度，减少调度开销
- how：一个进程内设置多个线程，实现进程内部的并发，以提高程序的并发度；线程的context小，能够快速切换，调度开销小
- how do：
  - 将进程的resource unit和scheduling unit分开
  - 具体来说：将进程的PCB拆成两部分，一部分为资源和地址空间，另一部分为调度、状态等信息，后一部分成为线程的TCB，因此进程内的线程共享同一资源、共享同一虚拟地址空间，只拥有必要的数据结构（TLB、stack、register、counter），因此可以实现更小的调度开销
- type：
  - user-level thread：
    - 线程不被kernel所知道，一个进程内的某个线程死了，整个进程就死了
    - 用户有权限通过thread lib操控线程，包括创建、调度、消亡等
  - kernel-level thread
    - 用户没有权限操控线程



### process v.s. thread

- 进程是资源分配的最小单位，一个进程内的所有线程共享同一资源

- 进程是虚拟地址空间（虚表）的最小单位，一个进程内的所有线程共享同一虚拟地址空间
- 进程的切换（process context switch）需要保存和加载资源、虚拟地址空间映射转换，开销很大，而线程的切换（thread context switch）开销很小，因为共享了资源和空间不需要切换
- 进程都是由OS控制，而线程可以由OS控制，也可以由user控制
- 进程和线程都具有生命周期，能够创建子线程、子进程



### process v.s. program

- 进程是动态的（是程序的一次活动），程序是静态的
- 进程是temporary，程序是permanent（由指令组成）
- 作业由程序、数据、操作说明书三部分构成，则进程和作业/程序都是一一对应的



### Parallel（并行）和Concurrent（并发）

- concurrent：在一个时间段内有多个任务在执行，但某个时间片上只有一个任务在执行，宏观上并行，微观上串行，只依赖于单核CPU，涉及多道程序（multiprogramming）设计
- parallel：多个任务一起执行，需要依赖多核CPU
- parallel是concurrent的子集



### mutual exclusion

- disabling interrupts
- lock variables
- strict alternation
- Peterson's solution
- TSL
- sleep and wakeup
- semaphore



### IPC(inter-processes communication)

- three types:
  - shared memory(`OpenMP`, `Pthread`)（共享变量不是全局变量）
  - shared file mode
  - message passing(`MPI`)（pipe、消息队列）
  - 另一个理解：共享内存、信息传递、管道、信号、信号量、套接字
- three classical problems:
  - producer-consumer problem
  - dining-philosophers problem
  - readers and writers problem



### priority-inversion problem（优先级反转问题）

- 低优先级任务持有高优先级任务所需要的临界资源，高优先级任务因此阻塞，一直等待低优先级任务释放临界资源；而由于低优先级任务获得的CPU时间片少，若出现介于两者优先级之间的中优先级任务，此任务并不需要那个临界资源，那么该任务将会获得更多的CPU时间片；若高优先级任务忙等，那么低优先级任务无法与高优先级任务争夺CPU时间片，导致无法执行，无法释放资源，进而反过来阻塞了高优先级任务，使得高优先级任务无法获得临界资源。
- 若使用了不恰当的scheduling，这是busy-waiting所可能会导致的问题



### semaphore

- 包含counter（资源数）和queue（block队列）
- num_blocked_processes = -counter if counter < 0 else num_available_resources  = counter

- p操作：申请资源，若申请失败则block（从ready队列移至block队列）
- v操作：释放资源，若释放后block队列仍有进程，则唤醒
- used for
  - mutual exclusion(mutex) - appearing in the same process
  - process synchronization(for `pthread` lib, it is related to **barrier**) - not appearing in the same process
    - three methods to realize barrier in `pthread` lib: mutex, semaphore and busy-waiting
  - **attention**: synchronization must be in front of mutex
- disadv：信号量的控制分布在整个程序中，难以分析正确性



### mutex = binary semaphore

- semaphore mutex = 1
- for n parallel programming, the semaphore counter value is ranging from 1 to -(n-1)



### Monitor

- 为了克服semaphore的缺点，将信号量及其操作语句封装在一个对象内部，隐藏细节，使其更好控制
- 为了使得进程能够在monitor中等待，使用了**condition variable**，used for `wait` and `signal`



## Scheduling

### 主要内容

- two type:
  - non-preemptive scheduling
  - preemptive scheduling
- why：对于并发，宏观上并行，微观上串行，在一个时间片内只能运行一个进程，需要依靠OS进行调度，为进程分配CPU时间片
- when：
  - 创建子进程：是执行父进程还是子进程
  - 进程退出：选择下一个需要执行的进程
  - 进程阻塞：选择下一个需要执行的进程
  - IO interrupt：转IO中断处理进程
  - clock interrupt：选择下一个需要执行的进程
- goal：
  - 对于程序使用CPU的三种模式：IO密集型（IO bound）、计算密集型（CPU bound）和平衡型，对于IO密集型，**响应时间**很重要；对于计算密集型，**周转时间**很重要；对于平衡型，响应时间和周转时间的平衡很重要
  - CPU调度的目标就是最小化平均响应时间（**response time**）、最小化平均周转时间（**turnaround time**）、最大化吞吐率（**throughout**），能够适应不同的程序，保持系统各个功能部件均处于繁忙状态（**efficiency**）和提供某种公平的机制（**fairness**）（这也是**好的调度算法所需要的性质**）
  - for all systems，调度的目标是fairness, policy enforcement, resource balance
  - batch systems，调度的目标是**max throughout**, min turnaround time, max CPU utilization
  - interactive systems，调度的目标是**min response time**, best proportionality
  - real-time systems，调度的目标是**meeting deadlines**（在截止时间前完成所应该完成的任务）, predictability



### 单处理器进程调度算法

- batch systems
  - FCFS(First come first serve)
    - non-preemptive
    - disadv: **convoy effect**
      - ready队列中依次存在n-1个IO bound jobs和1个CPU bound job，n-1个IO bound jobs很快执行完并suspend等待IO，而紧接着根据队列顺序将会执行CPU bound job，导致**IO device idle**，最后执行完后，其他进程继续等待IO，导致**CPU idle**，最终导致**low CPU and IO device utilization**
  - shortest job first
    - non-preemptive or preemptive
    - disavd: **starvation**
- interactive systems(usually preemptive)
  - round robin
  - priority scheduling
    - FCFS with each priority level
  - multi queue
    - 进程永久性分配给队列，为不同队列分配优先级，以及分配对应的CPU时间片比例
    - background processes(**daemon**) use FCFS
    - system processes have hightest priority
  - multi-level feedback(multi-level queue with priority)
    - 为不同队列分配优先级，先执行完优先级高的队列，再执行优先级低的队列
    - 进程暂时性分配给队列，可以在队列间移动
    - 不同队列可以使用不同的调度策略，例如高优先级为round-robin，低优先级为FCFS，这样就使得同一进程能从高优先级队列不断向低优先级队列移动，直至进程完成
  - guaranteed sheduling
  - lottery scheduling
    - 根据可能性（tickets）调度进程，高优先级拥有更高的可能性（more tickets）
  - fair sharing scheduling
    - round robin对于进程是绝对公平的，但对于用户不是
    - 用户级别的平均调度，使得拥有任意数量的用户能够获得相同的CPU时间片数量



### 线程调度

- 用户级线程（kernel意识不到thread的存在）
  - kernel选择一个process，由process中的runtime system选择一个thread运行（round-robin）

- kernel级线程
  - 由kernel直接选择一个thread运行（round-robin）



## 其他

- 进程的状态：running、blocked（wait）、ready
- Spooling技术
- 时间中断（内中断）是使得进程进入ready状态，而不是blocked状态
- 通道是独立于CPU控制IO的设备
- memcpy，不仅要保证目标缓冲大于源缓冲，还要保证地址不重叠
- 进程间的通信：
  - 凡是与资源有关的操作，会影响其他进程的操作都需要系统调用实现
  - **原语**是操作或者指令集合，保证了其集合的原子性，进程和进程间的通信用的是通信原语，如信号、消息
  - 用户态切换到内核态
    - 系统调用
    - 异常
    - 外中断
- 内存管理：
  - **覆盖**技术的实现是把程序划分为若干个功能上相对独立的程序段，按照其自身的逻辑结构使那些不会同时运行的程序段共享同一块内存区域
  - 在分时系统中，用户的进程比内存能容纳的数量更多，系统将那些不再运行的进程或某一部分调出内存，暂时放在外存上的一个后备存储区，通常称为**交换**区，当需要运行这些进程时，再将它们装入内存
- reentrant code：不能自身修改的代码
- **通道**：通道是一种特殊的处理机，具有执行IO指令集的能力，通道并没有自己的存储器，因此通道程序是放在主存储器中的
- **管态**：特权态，系统态，是操作系统管理的程序执行时所处的状态
- **目态**：用户程序执行的状态
- **原语**和**广义指令**都可以被进程所调用，两者差别就是原语具有原子性，它是通过在执行过程中关闭中断实现的，一般由系统进程调用，而广义指令很多情况下可用目态下的系统进程完成，不一定要在管态执行，可以借助中断进入管态，交给对应的进程



## ext2 file system

inode号与文件(File/Directory)一一对应

- 打开文件，首先找到文件对应的inode号，其次找到对应的inode(index-node, 存放文件的基本信息和文件数据块位置(跟踪数据块方式：链式/索引等))，最后找到数据块(数据块中存放文件内容/目录项)

- 文件名不影响inode号

- 只要打开了文件，就只以inode号识别文件，因此可以在不关闭软件或重启的情况下更新，进而更新一个新的inode替换，等到下一次打开的时候，即通过新的inode找到新版文件



![](./img/ext2.png)

因此data bitmap和inode bitmap是不一样的，如果inode号已经用光了，无论是否有空闲块，都无法创建文件

根目录到子目录的过程：根目录inode号 --> 根目录inode --> 根目录的数据块(存放目录项，每一个目录项包含文件的基本信息和文件的inode号) --> 子目录的inode --> 子目录的数据块



inode具体位置：

- 如果存放文件所需的数据块小于10，直接查找
- 超过10，一级间接索引
- 若仍不满足，则出现二级、三级间接索引，那么inode所包含的内容为：
  - 10个指向数据块的指针
  - 1个指向索引块的指针
  - 1个指向二级索引块的指针
  - 1个指向三级索引块的指针



总结：不是所有的数据块都是存储真正的数据，但注意inode不是存放在数据区

数据块可能的内容：

- 真正的数据(File/Directory)
- 索引数据块(表)





## linux开机过程

1. 加载BIOS，读取硬件信息

   - BIOS包含了CPU信息、设备启动顺序信息、硬盘信息、内存信息、时钟信息、PnP特性等
   - BIOS将控制权交给MBR后，就由linux控制系统

2. 读取MBR

   - MBR包含了**引导程序**和**分区表**
   - 将MBR加载到Boot Loader中，用以将**活动分区的引导区**载入内存
   - MBR中引导程序发挥主导作用，优先于操作系统内核载入内存的指令，最后把控制权交给**活动分区的引导区**（因而主引导记录不依赖于任何操作系统，且硬盘的引导程序是可变的，因此可以实现多系统共存）

3. 启动Boot Loader

   - 严重依赖于硬件，是在操作系统内核载入前执行的汇编程序
   - 用以初始化硬件设备、建立内存空间映射图，为最后调用操作系统内核做准备

4. 加载内核，调用`start_kernal()`函数，进而调用一系列初始化函数并初始化各种设备，建立完整的linux内核环境

   - loader主要进行能让内核最低限度执行的初始化，而此处才是进行真正的初始化操作

   - 主要对进程管理（调度机制初始化、建立缓冲队列）、内存管理（初始化物理页面）、文件管理（初始化文件系统）、IO管理（初始化外中断）机制进行创建，同时初始化0号进程为idle进程，即系统空闲时占据CPU的进程
   - 最后通过`kernel_thread()`创建1号线程/进程，执行内核的`init()`函数

5. 执行**第一个运行程序**`/sbin/init`，根据读取的`/etc/inittab`文件确定运行等级

6. init进程执行**第一个用户层文件**`/etc/rc.d/rc.sysinit`脚本程序

   - 包括设定PATH、设定网络配置`/etc/sysconfig/network`、启动swap分区、设定`/proc`等等
   - 能够使得一般的用户程序可以正常地被执行，从而真正完成可供应用程序运行的系统环境

7. 依据`/etc/modules.conf`文件或`/etc/modules.d`目录下的文件来装载内核模块

8. 执行不同运行级别的脚本程序

   - 根据运行级别的不同，系统会运行rc0.d到rc6.d中的相应的脚本程序，来完成相应的初始化工作和启动相应的服务

9. 执行`/etc/rc.d/rc.local`

   - 用户自定义脚本

10. 执行`/bin/login`程序，进入登录状态

