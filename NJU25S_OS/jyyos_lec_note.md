# Lecture Note

> NJU Spring 25 操作系统原理 蒋炎岩

## 1 操作系统概述

### Why-为什么学操作系统?

- 弄清楚: 什么是今天计算机世界万丈高楼工程奇迹的地基?
- 觉醒体内"编程力量"

### What-什么是操作系统?

操作系统 = Fun

- 操作系统是帮助我们更好地开发程序的, 知道操作系统"能做什么"很有必要
- 理解操作系统需要理解它发展的历史, 有三个重要的线索
  - 硬件 (计算机)
  - 软件 (程序, 本课程视角)
  - 操作系统 (管理硬件和软件的**软件**), 夹在上面两者的中间

理解操作系统

- 指硬件和软件的中间层, 对单机做出抽象以支撑多个程序执行

历史课

- 1940s: 没有操作系统
- 1950s-1960s
  - 库函数 + 管理程序**排队执行**的调度代码
  - 多用户轮流共享计算机, operator负责操作程序的切换
  - 出现了各类对象: 设备, 文件, 任务; 以及相关API: CTSS (Compatible Time-Sharing System)
- 1960s-1970s
  - 能载入多个程序到内存且调度它们的**管理程序**, 出现**进程**, **虚拟存储**等概念, 现代分时操作系统诞生
- 1970s+: UNIX 奠定了今天计算机世界的基础

### How-怎样学操作系统?

- 有梦想的CS人
  - 以"做出东西"为课程导向, 而不是讲很多细碎的知识点. 
  - 如果掌握了big picture, 可以根据自己的理解补充细节.
- 让"编码"成为利剑
  - 在大语言模型的帮助下自己动手解决问题.
  - 享受创造的乐趣, 不断提升自己.

## 2 应用视角的操作系统

### 操作系统上的程序

Everything is a state machine.

- 任何程序都在计算机上执行
- 计算机是状态机
- 程序的执行就是状态的变化
  - C语言解释器: PicoC

递归到非递归的转换, 可以把任意程序改写成**非递归**

- *在程序的执行过程中, 只有在正确的时间到达正确的位置, 程序才会做一件正确的事情*
- C代码总是可以改写成等价的SimpleC, 每条语句只包含一次运算
- C程序是一个状态机
  - 状态
    - [StackFrame, StackFrame, ...] + 全局变量
  - 初始状态
    - 仅有一个 StackFrame (main, argc, argv, **PC=0**)
    - 全局变量全部为初始值
  - 状态迁移
    - 执行 frames[-1].PC 处的简单语句

### 操作系统上的最小程序

处理器也是状态机: 参考NEMU

构造"最小"的程序: 写一个`_start()`, 从一开始就获得机器的控制权

退出程序

- 程序自己是不能"停下来"的: 指令集中没有一条关闭计算机的指令
- 需要借助**操作系统**
  - 把**系统调用**的参数放到寄存器中
  - 执行`syscall`, 操作系统接管程序, 其可以任意改变程序状态, 甚至终止程序

二进制程序是一个状态机

- 状态: gdb内可见的内存和寄存器
- 初始状态: 由ABI规定
- 状态迁移
  - 执行一条指令
  - `syscall`指令: 将状态机完全交给操作系统 (上帝)

### 操作系统上的应用程序

用户感受不到操作系统, 只能感受到操作系统上运行的程序 (进程)

任何程序 = `minimal.S` = 状态机, 可通过`strace`追踪

- 总是被操作系统加载开始, 通过另一个进程执行`execve`设置为初始状态
- 经历状态机执行 (计算 + syscalls)
- 最终调用`_exit`退出

应用程序 = 计算 + 操作系统API

操作系统的职责: 提供令应用程序舒适的抽象 (**对象 + API**)

## 3 硬件视角的操作系统

### 硬件视角的操作系统

硬件根本不知道有没有操作系统, 它只是一个执行指令的状态机, 见什么指令执行什么指令

计算机系统中的抽象: 下层不需要知道上面怎么用, 只管"无情地提供服务"

- 系统调用支撑应用程序
- 指令集支持高级语言程序

计算机系统的状态机模型

- 状态: 内存, 寄存器的数值
- 初始状态: 由系统设计规定 (CPU Reset)
- 状态迁移: 从PC取指令执行

操作系统就是一个普通的二进制**程序**

- 接管了中断, I/O, 而应用程序不能直接访问
- 把应用程序"放"到CPU上运行一会
  - 中断后, 操作系统又开始执行
  - 操作系统启动后, 操作系统就变成了一个中断处理程序

### 固件: 硬件和操作系统之间的桥梁

CPU是无情执行指令的机器

- 从CPU Reset开始执行, 从`Mem[PC]`取指令, 译码, 执行, 如此往复.
- 所以这里必须是合法的代码, 这段代码由系统厂商设置, 通过memory-map映射到一个特殊的存储器.

Firmware

- 固件: 与Software相对, 厂商固定在计算机系统的代码, 早期的固件存储在ROM中, 在今天可以直接升级固件了
- 功能
  - 运行程序前配置计算机系统: CPU电压, 内存时序, 接口开关...
  - **加载操作系统**
- Firmware就是一段代码
  - 一个小的"操作系统"
  - Legacy BIOS (Basic I/O System)
  - [UEFI (Unified Extensible Firmware Interface)](https://www.zhihu.com/question/21672895)

### 加载操作系统

IBM PC/PC-DOS 2.0 (1983)

- 如果磁盘的前512字节以`0x55, 0xAA`结尾, BIOS会加载这512字节到内存`0x7c00`

Firmware和系统程序员的第一个接口

- 可以写446字节的16-bit代码: 446 = 512 - 2 (`0x55, 0xAA`) - 64 (分区表)

编译OpenSBI和调试Makefile

- 让AI来帮忙

## 4 数学视角的操作系统

> 程序就是状态机; 状态机可以用程序表示. 因此: 我们可以用更"简单"的方式 (例如 Python) 描述状态机, 建模操作系统上的应用, 并且实现操作系统的可执行模型. 而一旦把操作系统, 应用程序当做"数学对象"处理, 那么我们图论, 数理逻辑中的工具就能被应用于处理程序, 甚至可以用图遍历的方法证明程序的正确性. 

### 数学视角的计算机程序

程序是数学严格的对象

- 程序 = 初始状态 + 迁移函数
- 在这个视角下, 程序和数学对象已经无限接近了
  - $f(s)=s′$

证明程序 $f$ 正确

- 暴力枚举 (测试即其中一些代表性样例)
- 写出证明 (像数学证明一样)

### 数学视角的操作系统

操作系统 = 状态机的管理者, 其自身也是一个**状态机**

- 可以同时容纳多个程序状态机
  - 程序内计算: 选一个程序执行一步
  - 系统调用: 创建新状态机, 退出状态机, 打印字符...

课堂示例: 用 Python 实现一个玩具操作系统 (要点: Python 中`yield`的用法)

- 操作系统中的对象
  - 状态机 (进程)
    - Python 代码
    - 初始时, 仅有一个状态机 (main)
    - 允许执行计算或 read, write, spawn 系统调用
  - 一个进程间共享的 buffer ("设备")
- 系统调用

UNIX 系统的基本模型

- 进程
- 系统调用
- 上下文切换
- 调度

并发

### 状态机模型与模型检查器

计算机系统中的不确定性

- 调度: 状态机的选择不确定
- I/O: 系统外的输入不确定
- 这样程序的执行就不是"一条线"了

程序定义了状态机 $G(V, E)$, 加上一个起点 $v_0$, 再加上 $F \in V$ 是坏的状态, 程序正确 = 不存在从 $v_0$ 到 $v \in F$ 的路径

- 在 $G$ 里找路径: DFS/BFS
- 在理论上, 软件是可以自动证明的

综合一下: 一个更复杂的玩具-模型检查器

- 模型, 检查器, 可视化
- 更多系统调用

## 5 程序和进程

> 操作系统通过进程抽象为应用程序提供了独立的执行环境. 进程是操作系统中最基本的资源分配单位, 它包含程序本身的状态和操作系统内部的状态. 操作系统提供了一系列系统调用 (如 Linux 中的 fork, exec, wait 和 exit, Windows 中的 CreateProcess 和TerminateProcess) 来创建, 管理和终止进程.

虚拟化

- 把物理计算机**抽象**成虚拟计算机: 程序好像独占计算机运行, 而不用考虑中断等问题

### 程序和进程

程序是状态机的静态描述

- 描述了所有可能的程序状态
- 程序 (动态) 运行起来, 就成了**进程** (进行中的程序)

进程: 程序的**运行时状态**随时间的演进

- 获取进程的信息: *问AI*

代码质量

- 机器永远是对的, 未测代码永远是错的
- **测试框架**

### 进程管理

#### 进程管理系统调用

操作系统 = 状态机的管理者 = 进程管理者

直观的想法 (例子: Windows)

- 创建状态机: `spawn(path, argv)`
- 销毁状态机: `_exit()`

UNIX

- **复制**状态机: `fork()`
- **复位**状态机: `execve()`

#### 创建状态机

`pid_t fork(void);`

立即复制状态机

- 所有状态完整拷贝: 寄存器, 内存, 以及进程在**操作系统**里的状态

如何区分?

- 新创建进程返回`0`
- 执行 fork 的进程返回子进程的进程号

进程树

- "托孤"机制

`fork-demo-2`里的`printf()`

- 管道符输出: 把字符串存在缓冲区, 直到最后一次性输出, 这导致了`fork`时缓冲区也被复制
- 命令行输出: 立即输出一行, `fork`时**缓冲区**不同

#### 复位状态机

`int execve(const char *filename, char * const argv[], char * const envp[]);`

唯一能够"执行程序"的系统调用

- 将当前进程**重置**成一个可执行文件描述状态机的初始状态
- **操作系统维护的状态不变**: 进程号, 目录, 打开的文件...
- 程序 (代码, 数据) 能被正确加载到内存, PC位于程序入口

#### 销毁状态机

`void _exit(int status);`

立即摧毁状态机, 允许有一个返回值, 该返回值可以被父进程获取

## 6 进程的地址空间

> 操作系统通过虚拟内存为每个进程提供独立的地址空间, 实现了进程间的隔离和保护. 操作系统通过 mmap/munmap 实现地址空间的管理, 并且还提供特定机制 (如 procfs, ptrace, process_vm_writev, 共享内存等) 访问彼此的地址空间.

### 进程的初始状态

进程 `execve` 后的初始状态: 寄存器, 内存 (暂时忽略操作系统管理的状态)

- *观察: 用**工具** (AI辅助生成) 看真实的操作系统*
- ABI 中规定的 initial state (只规定了部分寄存器和栈)
- Binary 中指定的 PT_LOAD 段
  - 内存是分成一段一段的, 每一段都有访问权限

statedump

- GPT: `vvar`, `vdso`是 Linux 内核中的两种特殊内存区域, 用于优化用户空间


### 进程的地址空间管理

#### Memery Map 系统调用

用于在状态机状态上增加/删除/修改一段可访问的内存 (即改变进程的地址空间)

```c
// 映射
void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);

// 取消映射
int munmap(void *addr, size_t length);

// 修改映射权限
int mprotect(void *addr, size_t length, int prot);
```

`pmap`: 查看进程的地址空间

#### 使用 `mmap`

- 申请大量内存空间, 瞬间完成
- 映射大文件, 只访问其中的一小部分

### 入侵进程的地址空间

允许一个进程对其他进程的地址空间有访问权, 意味着可以任意改变另一个程序的行为.

示例

- 调试器 (gdb), 可以任意观测和修改程序的状态
- **游戏外挂**

#### 卡带机时代

- 物理入侵: 金手指, 通过一个 Look-up Table (LUT), 当CPU 读地址 $a$ 时读到 $x$ 则替换为 $y$.

#### 现在

- 观察内存变化 (金钱, 经验), 查找 + Filter
- 金山游侠: 专为游戏设计的调试器
- DMA: 内存控制器
- 外挂的另一个思路: 采集视频信号用于解析

## 7 访问操作系统中的对象

> 操作系统必需提供机制供应用程序访问操作系统对象. 对于 UNIX 系统, 文件描述符是操作系统中表示打开文件 (操作系统对象) 的指针; 而 Windows 则更直接地提供了 handle 机制.

### 文件描述符

文件

- 字节流 (terminal, random)
- 字节序列 (普通文件, 如`a.txt`)

文件描述符

- 指向操作系统对象的**指针**
  - Everything is a file, 通过指针可以访问一切
- 对象的访问都需要指针 (系统调用)
  - `open()`, `close()`
  - 解引用: `read()`, `write()`
  - 指针运算: `lseek()`
  - 指针间赋值: `dup()`

文件描述符的分配

- 总是分配最小的未使用描述符
- 打开的文件数有限

文件描述符中的 offset

- *参考ICS-PA*
  - 文件描述符是"进程状态"的一部分, 保存在操作系统中, 程序只能通过整数编号访问
  - 文件描述符自带一个`offset`
- 对于 UNIX, `fork()`和`dup()`之后, 文件描述符的`offset`是共享的

Handle

- 句柄, Windows中的文件描述符, 默认handle是不继承的 (与 UNIX 相反)

### 访问操作系统中的对象

UNIX: 遵循 Filesystem Hierarchy Standard (FHS)

任何"可读写"的东西都可以是文件

- 真实设备
  - `/dev/sda`
  - `/dev/tty`
- 虚拟设备, 并没有实际的"文件", 而是操作系统为它们实现了特别的`read`和`write`操作
  - `/dev/random`: 随机数
  - `/dev/null`: 黑洞
  - `/sys/class/backlight`: 屏幕亮度

管道: 特殊的"文件"流

- 由读者和写者共享
- 读口支持`read`, 写口支持`write`

匿名管道

- `int pipe(int pipefd[2]);`
- 返回两个文件描述符
- 进程同时拥有读口和写口
- 搭配`fork()`使用

### 一切皆文件

文件描述符

- **适合**字节流的顺序读/写, 没有数据就等待, 比如管道
- **不适合**字节序列, 因为需要`lseek()` (`mmap`更香)

优点

- 一套 API 访问所有对象, 例如一切都可以`| grep`
- 优雅

缺点

- 和各种 API 紧密耦合
- 对高速设备不够友好

## 8 终端和 UNIX Shell

> 通过 freestanding 的 shell, 我们阐释了"可以在系统调用上创建整个操作系统应用世界"的真正含义: 操作系统的 API 和应用程序是互相成就, 螺旋生长的: 有了新的应用需求, 就有了新的操作系统功能. 而 UNIX 为我们提供了一个非常精简, 稳定的接口 (fork, execve, exit, pipe ,...), 纵然有沉重的历史负担, 它在今天依然工作得很好.

### 终端

打字机时代

- Shift: 使字锤或字模向上移动一段距离, 切换字符集
- CR: Carriage Return, `\r`, 回车, 将打印头移回行首
- LF: Line Feed, `\n`, 换行, 将纸张向上移动一行 (UNIX 的`\n`同时包含 CR 和 LF)

计算机终端原理

- 作为输出设备
  - 接受 UART 信号并显示, 通过转义字符 Escape Sequence 控制显示

- 作为输入设备
  - 把按键的 ASCII 码输出到 UART (所以有很多控制字符)


伪终端 Pseudo Terminal

- 一对"管道"提供双向通信通道
  - 主设备 (PTY Master): 连接到终端模拟器
  - 从设备 (PTY Slave): 连接到 shell 或其他程序, 例如 `/dev/pts/0`
- 伪终端经常被创建
  - ssh, tmux new-window, ctrl-alt-t, ...
  - `openpty()`: 通过 `/dev/ptmx` 申请一个新终端
    - 返回两个文件描述符 (master/slave)
- *详解一下过程 (by DeepSeek-R1)*
  - 打开`/dev/ptmx`, 获得`master`, 内核分配从设备`/dev/pts/0`, 文件描述符为`slave`.
  - 启动 Shell 进程, 通过`dup2()`将`stdin/stdout/stderr` 指向 `slave`.
  - 用户输入`ls`, 终端模拟器调用`write(master, "ls\n")`.
  - 内核将数据从主设备`master`转发到`/dev/pts/0`.
  - Shell 读取`/dev/pts/0`, 执行`ls`并将结果写入`/dev/pts/0`.
  - 内核将结果从`/dev/pts/0`返回到`master`, 终端模拟器读取并显示.

实现终端模拟器

- openpty + fork
  - 子进程: stdin/stdout/stderr 指向 slave
  - 父进程: 从 master 读取输出显示到屏幕; 将键盘输入写入 master

### 终端和操作系统

#### 背景

在命令行终端

- 在运行某个程序时可以通过 Ctrl-Z 停止程序运行, 把程序放到**后台**, 返回终端, 相当于 GUI 中的**最小化**按钮
- 在运行某个程序时可以通过 Ctrl-C 终止程序运行, 返回终端, 相当于 GUI 中的**关闭**按钮
- 通过命令`jobs`可以查看后台所有程序, 通过命令`fg`可以进入程序, 相当于把程序从后台切回**前台**, 在 GUI 中都有类似实现

那么问题来了: 我们有那么大一棵进程树, 都指向**同一个终端**, 有的在前台, 有的在后台, Ctrl-C 到底终止**哪个进程**?

- 终端直观传输字符, 而**操作系统**会接收这个字符并采取相应行动.
  - `fork()` 会产生树状结构 (还有托孤行为)
  - Ctrl-C 应该终止所有前台的"进程们", 但不能误伤后台的"进程们"

#### 会话 (Session) 和进程组 (Process Group)

给进程引入一个额外编号 (Session ID, 大分组)

- 子进程会继承父进程的 Session ID
  - 一个 Session 关联一个**控制终端** (Controlling Terminal)
    - Controlling Terminal 数据结构: Foreground PGID, Controlling SID
  - Leader 退出时, 全体进程收到 Hang Up (SIGHUP)

再引入另一个编号 (Process Group ID, 小分组)

- 只能有一个前台进程组
- 操作系统收到 Ctrl-C, 向前台进程组**所有进程**发送 SIGINT

进程数据结构: PID, PPID, PGID, SID

相关 API:

- setsid/getsid
- setpgid/getpgid
- tcsetpgrp/tcgetpgrp

#### POSIX Job Control

窗口和多任务: 终端可以有"一个前台进程组”

- 最小化 = Ctrl-Z (SIGTSTP)
  - SIGTSTP 默认行为暂停进程, 收到 SIGCONT 后恢复
- 切换 = fg/bg (tcsetpgrp)

为什么 Ctrl-C 能终止程序? *问 AI*

### UNIX Shell 编程语言

UNIX Shell 是一种**基于文本替换的极简编程语言**

- 只有一种类型: 字符串
- 不支持算术运算, 但可以 `expr 1 + 2`

语言机制

- 预处理
- 重定向
- 顺序结构
- 管道

例子: 实现重定向 (这也解释了为什么命令`sudo echo hello > /etc/a.txt`会没有权限)

```c
int fd_in  = open(..., O_RDONLY | O_CLOEXEC);
int fd_out = open(..., O_WRONLY | O_CLOEXEC);

int pid = fork();
if (pid == 0) {
    dup2(fd_in, 0);
    dup2(fd_out, 1);
    execve(...); // 这里才执行程序 "sudo echo", 之前的都是低权限
} else {
    close(fd_in);
    close(fd_out);
    waitpid(pid, &status, 0);
}
```

优点: 高效, 简介, 精确

缺点: 操作的优先级不明确...

## 9 C标准库和实现

> 在系统调用和语言机制的基础上, libc 为我们提供了开发跨平台应用程序的"第一级抽象". 在此基础上构建起了万千世界: C++ (扩充了 C 标准库), Java, 浏览器世界……今天, C 语言在应用开发方面有很多缺陷, 但仍然为"第一级抽象"提供了一个有趣的范本

理论上可行 != 工程上可行

### libc 简介

系统调用是地基, C语言是框架, 实现了系统调用的一层**浅封装**

The C Standard Library

- 语言机制上的运行库
  - 大部分可以用 C 语言本身实现
  - 少部分需要一些"底层支持", 比如体系结构相关的**内联汇编**
- 库也被标准化
  - ISO IEC 标准的一部分
  - POSIX C Library 的子集
    - 稳定, 可靠 (不用担心升级版本会破坏实现)
    - 极佳的移植性: 包括你自己的操作系统

libc是可以自己实现的: 课堂示例 minilibc

### 基础编程机制的抽象

configuration: 需要在长期实践中建立的概念

libc并不神秘, 但如何**学习**已有的 libc 的实现? 从 musl 开始

- 基本原则: 总有办法的, 毕竟可以问 AI

### 系统调用与环境的抽象

Standard I/O: `stdio.h`

`popen()`和`pclose()`

`perror()`

`environ`

### 动态内存管理

`malloc()`和`free()`

- 问题: 编程语言抽象不足, 可能会产生 use after free, double free, memory leak 的问题
- **最小完备性原则**和**机制策略分离**的反面教材

操作系统支持大段内存的**申请** (要多少有多少, 使用时再分配), 但不支持分配一小段内存

- 把这件事交给应用程序自己
- 自己在内存上实现一个**数据结构**

理论 v.s. 实践

- 脱离 workload 做优化就是耍流氓
- 在实际系统中, 我们通常不考虑 adversarial worst case, 因为现实中的应用是"正常"的, 不是"恶意"的.

`malloc()`的观察

- 大对象分配后应, 读写数量应当远大于它的大小
- 推论: 越小的对象创建/分配越频繁
- 在 oslabs 里, 我们几乎只要管好**小对象**就好了

## 10 可执行文件

### 可执行文件

定义

- 一个操作系统中的对象 (文件)
- 一个字节序列 (可以把它当字符串编辑)
- 一个描述了状态机初始状态的**数据结构**
  - 可执行文件只是一个数据结构, ELF 只是其中一种选择

工具

- binutils: 包含 readelf, objdump 等实用工具可以帮助了解 ELF 文件, 但有点**原始**
- **现代**工具: elfcat, 可以生成简洁易懂了html格式数据

### 和 ELF 搏斗的每一年

ELF 并不是一个人类友好的**状态机数据结构描述**, 它为了性能, 彻底违背了可读 "信息局部性" 原则

- 优点: 一个标准, 管好所有相关工具链
- 缺点: 支持的特性越多, 越不人类友好 (人类友好: 平坦的, 容易理解的信息, 所有需要的信息都立即可见)

选择**重开**: 设计一个 **FLE**, Funny (Fluffy) Linkable Executable

### Funny Little Executable

就像一种 DSL, Domain-specific Language

- 只要领域内的表达能力. 做了减法, 就可以更**专注**, 更简洁, 更自然.
- 例子: 正则表达式, Markdown

实现 FLE Binutils

- objdump/readfle/nm (显示)
- cc/as (编译)
- ld (链接)

### 操作系统和加载器

加载: (*重温 PA*) 把字节序列搬到内存, 然后设置正确的 PC, 开始运行

加载器: 内核实现的一部分, `execve()`, 由内核来解析 path, 完成加载

`#!`: UNXI 妙用, 优先使用 `#!` 指定的解释器

## 11 动态链接与加载

> 找到正确的思路, 我们就能在复杂的机制中找到主干: 在动态链接的例子里, 我们试着自己实现动态链接和加载. 在这个过程中, 我们"发明"了 ELF 中的重要概念, 例如 Global Offset Table, Procedure Linkage Table 等.

*参考 [CMU15-213: Linking](<../CMU15-213/cmu15-213_lec_note## 13 Linking>)*

### 动态链接: 机制

从**需求**出发: 实现运行库和应用代码分离

- 应用之间库的共享
  - 每个程序都需要 glibc, 但系统里只需要**一个副本**就可以了
  - 运行库和应用代码还可以分别**独立**升级
- 大型项目的分解
  - 改一行代码不用重新链接 2GB 的文件

### mmap 和虚拟内存

实验: `libbloat.so`

- 创建 1,000 个进程**动态链接** `libbloat.so` (一个 100M 的库) 的进程, 实际只占用了几百兆的内存

共享库的加载

- 对比实验: 从 gdb 的 `starti` 开始, 分别调试静态编译和动态链接的 `a.out`
- 对于动态链接
  - `a.out`的第一条指令不在程序里, 而在 `ld.so` 中, 这由 ELF 文件的 INTERP 段约定
  - `a.out`执行时, libc 还没有加载, 它是后来由 `mmap` 系统调用加载的
- 只读方式 mmap 同一个文件, **物理内存**中只有一个副本

虚拟内存管理

- 地址空间表面: 若干连续的内存段
  - 通过 mmap/munmap/mprotect 维护
- 实际: 分页机制维护的"幻象"

Virtual Memory

- 操作系统维护 **memory mappings** 的数据结构
- 延迟加载
- 写时复制 (Copy-on-Write)
  - `fork()`时, 父子进程先只读共享全部地址空间 (部分打上"可写"标记), Page fault 时, 写者复制一份


Memory Deduplication; Compression & Swapping

- 扫描内存
  - 有重复的 read-only pages, 合并
  - 发现 cold pages, 可以压缩/swap 到硬盘 (硬件提供了 Access/Dirty bit)
- 还能干嘛: *问 AI*

### 实现动态链接

*就像实现 FLE 一样, 但这里课上讲得神速, 没听懂*

应用程序的拆解

- 方案1: `libc.o`, 在加载时完成重定位
  - 加载 = 静态链接, 省了磁盘但没省内存
  - 缺点: 浪费时间 (链接需要解析很多不会用到的符号)
- 方案 2: `libc.so`, 让编译器生成**位置无关代码**
  - 加载 = mmap, 但函数调用时需要**额外一次**查表
  - 优点: 映射**同一个** `libc.so`, 内存中只需要一个副本

动态加载 (以 `dlbox` 为例)

- 编译时: 动态链接库调用 = 查表

  - `call  *TABLE[printf@symtab]`

- 链接时: 收集所有符号, "生成" 符号信息和相关代码

   ```c
    #define foo@symtab     1
    #define printf@symtab  2
    ...
    
    void *TABLE[N_SYMBOLS];
    
    void load(struct loader *ld) {
        TABLE[foo@symtab] = ld->resolve("foo");
        TABLE[foo@printf] = ld->resolve("printf");
        ...
    }
   ```

- GOT (Global Offset Table)

  - 对于每个需要动态解析的符号, GOT 中都有一个位置
  - (GOT 中 `printf()` 的地址才是真正的地址)

- PLT (Procedure Linkage Table)

  - 先编译, 后链接产生的问题: 对于某个函数的跳转 (**编译器**无法区分是自己编写的`foo()`还是动态链接库的`printf()`) 到底要不要查表

    - 如果选择全部查表跳转: 则性能会有极大损失
    - `ff 25 00 00 00 00  call *FOO_OFFSET(%rip)`

  - 为了性能, 不得不选择全部"直接"跳转

    - `e8 00 00 00 00  call <reloc>`
    - 如果函数来源**动态加载**, 则需借助 **PLT**
    - `e8 f0 fe ff ff  call 1060 <puts@plt>`

  - 怎么理解这个过程? *问 AI*

    ```markdown
    程序启动 → 内核加载动态链接器（ld-linux.so） → 加载依赖库（如libc.so）
           ↓
    首次调用printf@plt → GOT指向解析函数 → 动态链接器查找printf地址 → 更新GOT → 跳转执行
           ↓
    后续调用printf@plt → 直接跳转GOT中已存储的地址
    ```

- PLT 没能解决数据的问题

  - 对于 `extern int x`, 不能"间接跳转"
  - 不优雅的解决方法: `-fPIC` 默认会为所有 extern 数据增加一层间接访问


LD_PRELOAD

- 程序启动时可以优先动态链接指定的库 (符号解析和重定位遵循先到先得原则)
- `LD_PRELOAD=./mylib.so ./a.out`
- 例子: 给 `malloc()` 加个封装, 参照 [CMU15-213 Library Interpositioning](<../CMU15-213/cmu15-213_lec_note### Library Interpositioning>)

## 12 构建应用生态

### 从 UNIX 到 Linux

UNIX 最早版本: 甚至没有 `fork()`

- 从零开始很重要

历史

1. Minix1 (1987)
2. Minix2 (1997)
3. Minix3 (2006)

Linux (25 Aug 1991)

- 合适的人 + 合适的时间

Linux 的两面

- **Kernel**
  - 加载第一个进程
    - 相当于在操作系统中放置一个位于**初始状态的状态机**
    - Initramfs 模式
  - 包含一些进程可操纵的操作系统对象
  - *就这么多* (启动后 Kernel 就是一个 trap handler)
- Linux Kernel **系统调用**上的发行版和应用生态
  - 系统工具 coreutils, binutils, systemd
  - 桌面系统 Gnome, xfce, Android
  - 应用程序 file manager, vscode

### Linux 和应用程序的接口

**操作系统中的对象**应该也有一个初始状态

intiramfs

- 自己实现一个
  - 可以只有一个 `init` 文件
  - 系统启动后, Linux 还会增加 `/dev` 和 `/dev/console`
- 实际的
  - 基本命令行工具
  - 基础设备驱动程序
  - 文件系统工具

initramfs 并不是我们看到的 Linux 世界

- 启动的初级阶段
  - 加载剩余必要的驱动程序, 例如磁盘/网卡
  - 挂载必要的文件系统
    - Linux 内核有启动选项 (类似环境变量)
      - /proc/cmdline (man 7 bootparam)
    - 读取 root filesystem 的 /etc/fstab
  - 将根文件系统和控制权移交给另一个程序, 例如 systemd
- 启动的第二级阶段: `/sbin/init`
  - 加载必要的驱动
  - 释放 initramfs

切换根文件系统

```c
int pivot_root(const char *new_root, const char *put_old);
```

**操作系统 = 对象的集合**

- 初始状态
  - initramfs 中的对象 + `/dev/console` + 加载的 init
- 状态迁移
  - 选择一个进程 (对象) 执行一条指令
  - 系统调用
    - 进程管理: fork, execve, exit, waitpid, getpid, ...
    - 操作系统对象和访问: open, close, read, write, pipe, mount, mkfifo, mknod, stat, socket, ...
    - 地址空间管理: mmap, sbrk (mmap 的前身)
    - 以及一些其他的机制: pivot_root, chmod, chown, ...

### 构建应用程序的世界

操作系统仅有的机制: 初始状态 + 系统调用

操作系统完全"感知"不到应用程序

操作系统上的应用生态

- 应用商店模式
- 开源模式

## 虚拟化: 总结

至此, 我们终于完全展示了逐层抽象的计算机系统世界: 

- 硬件向上提供了指令集体系结构
- 基于指令集, 实现了操作系统 (对象, 系统调用 API 和 initramfs 定义的初始状态)
- 操作系统上面支撑了系统工具 (coreutils, shell, apt, gcc, ...)
- 系统工具上面才是各式各样的应用程序

实际上, 我们看到的计算机系统中的一切都是由应用程序"完成"的, 操作系统只是提供系统调用这个非常原始的服务接口. 正是系统调用 (包括操作系统中的对象) 这个稳定的, 向后兼容的接口随着历史演化和积累, 形成了难以逾越的技术屏障, 在颠覆性的技术革新到来之前, 另起炉灶都是非常困难的.

## 13 多处理器编程: 从入门到放弃

> 我们可以很容易地把状态机模型扩展为共享内存上的多线程模型——只是每次选择一个状态机执行一步, 通过提供 spawn 和 join 两个 API 来利用现有多处理器系统的共享内存能力.
>
> 然而, 由于编译优化的"无处不在"(处理器也是编译器), 共享内存并发的行为十分复杂. 与此同时, 人类又恰好是物理世界 (宏观时间) 中的"sequential creature”, 编程语言的直觉 (顺序/选择/循环结构) 也是围绕顺序程序设计, 因此共享内存上的并发编程是非常具有挑战性的"底层技术”. 在《操作系统》课中, 我们不建议大家"玩火”——我们之后会介绍多种并发控制技术, 使得我们可以在需要的时候避免并发的发生, 使并发程序退回到顺序程序, 从而使我们能够理解和控制并发.

### 入门: 共享内存线程模型与线程库

动机

- 单线程无法快速处理突发的大量的请求, 无法发挥多个 CPU 的性能
- 我们想要有**共享内存的进程**

解决: 增加一个操作系统 API:  `spawn()`

- 能增加一个"状态机", 有独立的栈, 但共享全局变量
- 状态迁移: 选择一个状态机执行一条语句 (指令)

并发 vs 并行

- 并发
  - 逻辑上的"同时执行"
    - 可以由操作系统/运行库模拟出的"轮流执行"
    - (也可以是真正同时执行)
- 并行
  - 真正意义上的"同时执行"
  - 有 (共享内存的) 多个处理器
    - 同时执行指令 (load/store 访问共享内存)

简化的线程 API (thread.h)

- `spawn(fn)`: 创建一个入口函数是 `fn` 的线程, 并立即开始执行
- `join()`: 等待所有运行线程的返回 (main 返回默认会 join 所有线程)

> **并发编程确实有诸多好处**
>
> 那么, 代价是什么呢? 代价就是我们需要重新理解"编程".

### 放弃: 状态迁移的确定性

确定性的丧失

- 虚拟化使进程认为**"世界上只有自己"**, 即除了系统调用, 程序的行为是 deterministic 的
- 但**并发打破了这一点**
  - 并发程序每次 non-deterministically 选一个线程执行, 这意味着 load 可能读到其他线程的 store (也可能不)
  - 这导致了程序理解起来相当困难

例子: 两个线程同时向银行取钱, 可能会导致余额变为负数 (即超大 unsigned long)

例子: 并发执行三个 T_sum, 求 sum 的最小值

```c
void T_sum() {
    for (int i = 0; i < 3; i++) {
        int t = load(sum);
        t += 1;
        store(sum, t);
    }
}
```

- 答案是 **2**, 可以构造出某个线程进行了 load(1) (1 这个值由其他线程或自己 store), 最终由该线程执行 store(2), 而其他线程的所有 load/store 发生在上述2条指令之间.
- 为什么**不是 1**, 想要最终 (即 i = 2时) store(1), 必须进行 load(0), 但 load(0) 只能发生在 i = 0;

### 放弃: 代码按顺序执行

编译优化

- 虚拟化和 Determinism: 除了系统调用, 没人能"干涉"程序的状态
- **编译器**会按照"系统调用"优化程序
  - 语句/指令不需要按程序声明的那样执行. 典型的优化: 死代码消除 (dead code elimination)
  - 只要保证优化前后的程序在系统调用层面上等价, 可以任意调换/删除语句. 但这和 non-determinism 是矛盾的, 比如 load 可能会读到来自其他线程写入的值.

例子: `while (!flag);`

- 编译器会基于 "flag 是不会被修改" 的角度进行优化 (但实际上 flag 会被其他线程修改)

控制编译器优化的行为 (**不是**本课程推荐的方法)

- 插入"不可优化"的代码块

  ```c
  while (!flag) {
      asm volatile ("" ::: "memory");
  }
  ```

- 标记变量 load/store 为不可优化

  ```c
  int volatile flag;
  while (!flag);
  ```

### 放弃: 全局的指令执行顺序

并发程序的状态机模型

- 状态迁移: 选择一个**线程**执行**一条指令**
- 内存读写是"立即"的
- 因此存在一个**全局的**指令执行顺序
- ![](../0_Attachment/Pasted%20image%2020250409205311.png)

以上是过渡简化的**幻觉**

- 即使多处理器系统很努力地维护这个幻觉, 但其依然靠不住
- 访问不同位置的内存的开销 (时间) 不同 (类比为多个星球上的相互通信)

真实的状态机模型: 宽松内存模型 (性能考虑)

- Store 写入 local memory (cache), 再慢慢同步给其他处理器
- 允许 load 读到 local memory (cache) 的旧值
- ![](../0_Attachment/Pasted%20image%2020250409205539.png)

"无序"的真实世界

- 共享的内存导致了"乱序": Store 对其他处理器可见的时间点可能不同
- 处理器内部也会"乱序"
  - 不同地址的 load/store 允许重排
  - "乱序执行" (out-of-order execution): 现代高性能处理器最值得骄傲的特性之一

## 14 并发控制: 互斥

> 并发编程"很难", 而类应对这种复杂性的方法就是退回到不并发. 我们可以在线程中使用 lock/unlock 实现互斥--所有被同一把锁保护的代码, 都失去了并发的机会 (虽然先后依然是不受控制的). 当然, 互斥的实现是相当有挑战性的, 现代系统中的互斥设计线程中的原子操作, 内核中的中断管理, 原子操作和自旋等机制. 值得注意的是, 而只要程序中"能并行"的部分足够多, 串行化一小部分也并不会对性能带来致命的影响. 

### 互斥: 阻止并发

从简单入手: `sum()`

- 希望有一个 API, 无论怎么执行, `sum()` 的求和都是正确的
- 互斥: **阻止同时的 sum++**

方案: Stop the World

- 让所有线程停止, 自己继续执行, 保证互斥
- 优化: 只要能声明"不能并发"的代码块就可以了, 其他 (和 `sum()` 无关的) 代码还是可以同时执行的

拟人视角

- 用 lock/unlock 标记一个代码块
- 所有标记的代码块之间 mutually exclusive

状态机视角

- 被标记的**整个代码块**的执行可以理解为**一次**状态迁移

实际生活中大部分计算都是可以并行的, 只有少部分需要 lock/unlock

### 使用共享内存实现互斥

用上厕所阐释 Peterson's Protocol

- 如果希望进入厕所, 按顺序执行以下操作
  1. 举起自己的旗子 (store 手)
  2. 把写有对方名字的字条贴在厕所门上 (store 门)
- 然后进入持续的观察模式:
  1. 观察对方是否举旗 (load 手)
  2. 观察厕所门上的名字 (load 门)
  3. **对方不举旗或名字是自己, 进入厕所, 否则继续观察**
- 出厕所后, 放下自己的旗子 (不用管门上的字条)

检验正确性: Model Checker

Peterson's Protocol 正确的**前提**

- Load/store 指令是瞬间完成且生效的
- 指令按照程序书写顺序执行
- 这些前提已被**放弃** (现在我们用的是高性能编译器和多核计算机)

实现正确的 Peterson 算法

- Compiler barrier 编译优化屏障
  - `asm volatile("": : :"memory");`
- Memory barrier (内存屏障)
  - `__sync_synchronize()` = Compiler Barrier +
    - x86: `mfence`
    - ARM: `dmb ish`
    - RISC-V: `fence rw, rw`
  - 能够实现单次 load/store 的原子性
- 该算法没有解决实际的问题, 我们需要 absolutely correct 的工程化方案

### 使用原子指令实现互斥

Peterson 算法的路线错误

- 试图在 load/store 上实现互斥
- 软件不好解决的, 硬件帮忙

关中断

- x86: `cli`
- RISC-V: `csrci mstatus, 8`
- 适用于单处理器系统, 操作系统内核代码

原子指令: 一个"不被打断"的 load + 计算 + store

- x86: Bus Lock (locked instruction)
  - `asm volatile("lock incq %0" : "+m"(sum)); `
- RISC-V: LR/SC & A 扩展
  - 来自 MIPS: Load-Linked/Store-Conditional (LL/SC)
- arm: ldxr/stxr, stadd (store add) 指令

自旋锁

```c
typedef struct {
  ...
} lock_t;
void spin_lock(lock_t *lk);
void spin_unlock(lock_t *lk);
```

### 使用系统调用实现互斥

Scalability: 系统随着需求或负载增加时, 依然能够保持性能和稳定性, 灵活扩展资源的能力

自旋锁的 Scalability 问题

- 除了获得锁的线程, 其他处理器上的线程都在**空转**
- **应用程序不能关中断**, 持有自旋锁的线程被切换, 导致 100% 的资源浪费

让系统调用帮忙

- `syscall(SYSCALL_acquire, &lk);`
  - 试图获得 lk, 但如果失败, 就切换到其他线程
- `syscall(SYSCALL_release, &lk);`
  - 释放 lk, 如果有等待锁的线程就唤醒
- 剩下的复杂工作都交给内核
  - 关中断 + 自旋
    - **自旋锁只用来保护操作系统中非常短的代码块**
  - 成功获得锁 -> 返回
  - 获得失败 -> 设置线程为"不可执行"并切换

pthread Mutex Lock (进行了性能优化)

```c
pthread_mutex_t lock;
pthread_mutex_init(&lock, NULL);

pthread_mutex_lock(&lock);
pthread_mutex_unlock(&lock);
```

## 15 并发控制: 同步 (1)

> 同步的本质是线程需要等待某件它所预期的事件发生, 而事件的发生总是可以用条件 (例如 depth 满足某个条件, 或是程序处于某个特定的状态) 来表达. 因此计算机系统的设计者实现了条件变量, 将条件检查和临界区"放在一起", 以实现线程间的同步.

互斥实现了**原子性** (A -> B 或 B -> A), 但没有给我们**确定性** (A -> B).

### 同步和条件变量

同步的定义: 两个或两个以上随时间变化的量在变化过程中保持一定的相对关系 (多个事件, 进程或系统在时间上协调一致, 确保按**预定顺序**或**同时**执行)

现实世界中的同步: 类比**事件** (代码的执行) 和**顺序**

- 交响乐团: 每个乐手都是一个"线程" (状态机)
  - 事件: 发出节拍, 演奏节拍
  - 顺序: 发出节拍 i -> 演奏节拍 i
- 约架
  - 事件 A: 第一个人到达, 事件 B: 第二个人到达
  - 不见不散: 有一个共同完成的事件 X
  - A -> X, B -> X, 因此一定有一个瞬间 A, B 完成, X 未开始
  - 同步点: 一个**确定**的状态, 用于实现并发控制
  - "控制": 将发散的并发程序状态"收束"

程序世界中的同步

- 同一线程内的事件天然存在 happens-before 关系
- 没有 happens-before 的代码就是**并发**的, 在多处理器系统中可以并行执行

实现同步: 通过互斥锁实现 happens-before, A -> B

- `A; can_proceed = true; (🚩)`
- `while (!can_proceed); B;`
- 缺点: 举旗前条件判断等于**空转**, 浪费 CPU 性能

实现同步: 借助操作系统的**条件变量机制**

- 条件不满足时等待 (让出 CPU), `wait`:  直接睡眠等待 (C++ 可以把条件放到 λ 表达式里)
- 条件满足时继续, `signal` / `broadcast`: 唤醒所有等待的线程

`orchestra.c` 和 `orchestra-cv.c` 的区别 (Claude 3.7 Thinking)

1. **CPU 资源利用率**
   - orchestra.c: 忙等待持续消耗 CPU 资源, 线程不断循环检查条件
   - orchestra-cv.c: 线程阻塞等待, 不消耗 CPU 资源, 更加高效
2. **唤醒机制**
   - orchestra.c: 没有显式唤醒机制, 依靠线程自己不断轮询检查条件
   - orchestra-cv.c: 使用 `cond_broadcast(&cv)` 主动通知所有等待线程
3. **系统负载**
   - orchestra.c: 随着等待线程数量增加, 系统负载线性增长
   - orchestra-cv.c: 无论等待线程数量多少, 系统负载基本稳定
4. **扩展性**
   - orchestra.c: 在大规模并发场景下性能急剧下降
   - orchestra-cv.c: 在高并发环境下表现更加稳定
5. **响应延迟**
   - orchestra.c: 可能有较低的响应延迟, 但代价是持续消耗资源
   - orchestra-cv.c: 有调度和唤醒延迟, 但总体效率更高

### 条件变量的正确打开方式

经典同步问题: 生产者-消费者问题

- Producer 和 Consumer 共享一个缓冲区
  - Producer (生产数据): 如果缓冲区有空位, 放入; 否则等待
  - Consumer (消费数据): 如果缓冲区有数据, 取走; 否则等待
    - **同步**: 同一个 object 的生产必须 happens-before 消费

简化版: 左右括号打印

- 生产 = 打印左括号 (push into buffer), 消费 = 打印右括号 (pop from buffer)
- 在 `printf()` 前后增加代码, 使得打印的括号序列满足
  - 不能输出错误的括号序列
  - 括号嵌套的深度不超过 n

关键: 想清楚**程序继续执行的条件**

奇怪的同步问题: **状态机**方式解决条件变量的判断

### 实现并发计算图

计算图模型: $G(V, E)$: 有向无环的 Dependency Graph

- 计算任务在节点上, 可以使用 shared memory
- 边 $(u,v)∈E(u,v)∈E$ 表示 v 的计算要用到 u 产生的值, $(u,v)$ 也是一个 happens-before 关系
- 这是一个非常基础的模型, 几乎总是可以用这个视角去理解并行计算, 如果节点独立计算时间足够长, 算法就是可高效并行的
- 例子: 最长公共子序列, 电路模拟, 深度神经网络

实现任意计算图: 为每个计算节点设置一个线程和条件变量

- 从生产者-消费者角度理解: 对于 u -> v, $T_u$ 完成后为 $T_v$ 生产一份, $T_v$ 消费 `num_pred` 份后继续

  ```c
  void T_u() {  // u -> v
      ... // u 的计算
      mutex_lock(v->lock);
      v->num_done++;
      cond_signal(v->cv);  // 这里是可以 signal 的
      mutex_unlock(v->lock);
  }
  
  void T_v() {
      mutex_lock(v->lock);
      while (!(v->num_done == v->num_predecessors)) {
          cond_wait(v->cv, v->lock);
      }
      mutex_unlock(v->lock);
      ... // v 的计算
  }
  ```

实现任意计算图: 实现一个任务的**调度器**

- 一个生产者 (scheduler), 许多消费者 (workers) 循环

- 双向生产/消费: $T_{worker}$ 生产 ready, 消费 job, $T_{scheduler}$ 消费 ready, 生产 job

  ```c
  mutex_lock(🔒);
  while (!(all_done || has_job(tid))) {
      cond_wait(&worker_cv[tid], 🔒);
  }
  mutex_unlock(🔒);
  
  if (all_done) {
      break;
  } else {
      process_job(tid);
  }
  
  signal(&sched_cv);
  ```

## 16 并发控制: 同步 (2)

> 信号量可以看做是互斥锁的一个"推广", 可以理解成游泳馆的手环, 停车场的车位, 餐厅的桌子和袋子里的球, 通过计数的方式实现同步-在符合这个抽象时, 使用信号量能够带来优雅的代码. 但信号量不是万能的——理解线程同步的条件才是真正至关重要的

**互斥**也实现了 happens-before

- release -> acquire, 即 释放锁 -> 获取锁

### 信号量

用互斥锁实现唤醒 (例子: 交响乐团)

- 初始时创建锁并将其锁上
- 指挥发出信号 = 释放锁 (**唤醒**)
- 乐手接收信号演奏 = 获取锁 + 执行演奏 (此时锁再次被锁上, 需要等待指挥的再次释放才能继续验演奏)

用互斥锁实现计算图

- 为每一条边 $e=(u,v)$ 分配一个互斥锁, 初始时, 全部处于锁定状态.
- 对于一个节点, 它需要获得所有入边的锁才能继续. 可以直接计算的节点立即开始计算.
- 计算完成后, 释放所有出边对应的锁 (唤醒下一个任务).

**以上为 undefined behavior** (不要滥用机制)

本质: Release as Synchronization

- acquire = 等待信号, release = 发出信号
- 信号可以理解为现实生活中的"资源许可" (停车位, 餐厅排队等号...)
- 适合处理**计数型**的同类资源
  - 停车 = "吃掉"车位; 离开 = "创造"车位 (允许凭空创造)

信号量

- 允许在不同的线程的 release/acquire

- "信号量是扩展的互斥锁"

  ```c
  void P(sem_t *sem) {
      // Prolaag - try + decrease/down/wait/acquire
      mutex_lock(&sem->lk);
      while (!(sem->count > 0)) {
          cond_wait(&sem->cv, &sem->lk);
      }
      sem->count--;  // 消耗一个信号
      mutex_unlock(&sem->lk);
  }
  
  void V(sem_t *sem) {
      // Verhoog - increase/up/post/signal/release
      mutex_lock(&sem->lk);
      sem->count++;  // 创建一个信号
      cond_broadcast(&sem->cv);
      mutex_unlock(&sem->lk);
  }
  
  // 把信号量当互斥锁用
  sem_t sem = SEM_INIT(1);
  
  void lock() {
      P(&sem);
  }
  
  void unlock() {
      V(&sem);
  }
  ```

### 用"互斥"实现同步

**信号量**的两种典型应用

- 实现一次临时的 happens-before: A -> B
  - A -> V(s) -> P(s) -> B, 这就是刚才的互斥锁实现同步
- 管理计数型资源
  - 停车场只有 n 个车位, 餐厅只有 n 桌

例子: join()

- worker: V (done)
- main: T个 P(done)

例子: **优雅**地实现生产者-消费者

```c
sem_t empty = SEM_INIT(depth);
sem_t fill = SEM_INIT(0);

void T_produce() {
    P(&empty);
    printf("(");
    V(&fill);
}

void T_consume() {
    P(&fill);
    printf(")");
    V(&empty);
}
```

### 信号量, 条件变量与同步

打印不同方向的鱼

- 不好用信号量表达 "二选一", 除非手动构建 happens-before

哲学家吃饭问题

- 条件变量: `avail[lhs] && avail[rhs]`
- 信号量
  - `P(&sem[lhs]) && P(&sem[rhs])` 隐藏的缺陷: 如果5个哲学家同时举起左手的叉子
  - 解决1: 从桌子上赶走一个人
  - 解决2: 给叉子编号, 总是先拿编号小的
  - 但这些都是非工程化的解决方法, 更理想的应该是设计一个 waiter, 负责分配叉子

用信号量实现条件变量

- 带来的问题: 唤醒丢失, 一个"早就 wait 但没有 P"的线程会抢走唤醒

  ```c
  void wait(cond_t *cv, mutex_t *mutex) {
      atomic_inc(&cv->nwait);
      mutex_unlock(mutex);
      P(&cv->sleep);
      mutex_lock(mutex);
  }
  
  void broadcast(cond_t *cv) {
      mutex_lock(&cv->lock);
      for (int i = 0; i < cv->nwait; i++)
          V(&cv->sleep);
      cv->nwait = 0;
      mutex_unlock(&cv->lock);
  }
  ```

- 并不合适

## 17 并发 Bugs

> 人类本质上是 sequential creature, 因此总是通过"块的顺序执行"这一简化模型去理解并发程序, 也因此带来了数据竞争, Atomicity violation (本应原子完成不被打断的代码被打断), Order violation (本应按某个顺序完成的未能被正确同步) 等问题. 数据竞争非常危险, 我们在编程时要尽力避免.

### 致命的并发 Bugs 和数据竞争

数据竞争 (Data Race)

- **不同的线程**同时访问**同一内存**, 且**至少有一个是写**. 
- Since C11: data race is undefined behavior
- 消灭了数据竞争 = 保证 serializability
  - 可能竞争的内存访问要么互斥, 要么同步 (回到顺序执行)

### 死锁

死锁

- AA-Deadlock: 锁住的执行流中再次申请同一把锁
- ABBA-Deadlock
  - 哲学家吃饭问题

在实际系统中避免死锁

- Lock Ordering
  - 任意时刻系统中的锁都是有限的
  - 可以给所有锁编号 (Lock Ordering), 严格按照从小到大的顺序获得锁
- Proof
  - 任意时刻, 总有一个线程获得"编号最大"的锁
  - 这个线程总是可以继续运行
- LockDep
  - 动态维护"锁依赖图"和环路检测

### 原子性/顺序违反

并发控制的机制: 后果自负

- 互斥锁 (lock/unlock) 实现原子性
  - 忘记上锁--原子性违反 (Atomicity Violation, AV)
- 条件变量/信号量 (wait/signal) 实现先后顺序同步
  - 忘记同步--顺序违反 (Order Violation, OV)

## 18 真实世界的并发编程 (1)

> 我们在"更好的编程"这条路上从未停止过努力. 针对不同应用的应用, 最终形成了"领域特定"的解决方案: Web 中的异步编程, 高性能计算中的 MPI 和 OpenMI, 数据中心中的 goroutines. 更有趣的是, 我们可以看到: 改变世界的技术, 往往只是一个小小的奇思妙想, 最终坚持到底得到的.

背景

- 并发编程核心抽象: 计算图
- 按照之前的方法 (条件变量 / 信号量) 来实现计算图会在代码中引入众多**干扰代码**
- 编程语言会增加一些功能受限的**语法**, 本质上都是描述计算图, 但能避免许多问题

### 高性能计算中的并发编程

来源: 密集型科学计算任务, 比如物理系统模拟, 挖矿, AI

特点: 物理世界具有"空间局部性", "模拟物理世界"的系统具有 embarrassingly parallel 的特性

计算图容易静态切分

- 机器级分解: [MPI](https://hpc-tutorials.llnl.gov/mpi/) - message passing libraries (机器间通信)

- 线程级分解: [OpenMP](https://www.openmp.org/) - multi-platform shared-memory parallel programming (C/C++ and Fortran)

  ```c
  // 对非 cs 专业友好的机制
  #pragma omp parallel num_threads(128)
  for (int i = 0; i < 1024; i++) {
  }
  ```

### 我们身边的并发编程

Web 1.0

- 只有 HTML, 没有 css, \<font\>, \<table\>, vbscript 和切图工程师一统天下

Web 1.0 -> 2.0

- Ajax: 运行网页实现"后台刷新", 即悄悄请求后端然后更新 DOMTree (让网页实现应用一般的功能)
- jQuery: A DOM Query Language (编程抽象)

Web 2.0 时代的并发编程: event-based concurrency (动态计算图)

-  **禁止任何计算节点并行** (和高性能计算完全不同的选择)
- 允许网络请求, sleep 在后台执行, 执行完后产生一个新的**事件** (计算节点)
- 基于假设: 网络访问占大部分时间, 浏览器内计算只是小部分
- 缺点: Callback hell (回调地狱), 不断嵌套的执行流

优化: 回归"描述计算图"

- 增加更加现代的前端"语法糖"

### 数据中心中的并发编程

需要高吞吐 (QPS) & 低延迟的事件处理

- 处理事件可能需要读写持久存储或请求网络上的服务
  - 延迟不确定
- 线程维护和上下文切换都会带来开销

版本答案: Serverless Computing

- Function as a Service (FaaS): 系统动态调整计算资源的分配
  - 程序员仅需编写大号的幂等的 Javascript 程序

背后系统的实现

- 数据保持一致 (Consistency), 服务时刻可用 (Availability), 容忍机器离线 (Partition tolerance) 不可兼得

协程: 操作系统"不感知"的上下文切换

- 进程内独立堆栈, 共享内存的**状态机**
- 但"一直执行", 直到 `yield()` 主动放弃处理器
- 其在同一个操作系统线程中执行, 可以由**程序**控制调度, 除了内存, 不会占用额外操作系统资源

Go 和 Goroutine: 兼有多处理器并行和轻量级并发

## 19 真实世界的并发编程 (2)

> 人类世界的需求一直是驱动技术革新的原动力. 回头看历史, 波澜壮阔的旅程又是显得那么理所应当--从 CPU 到领域加速器, 再变得"通用"一点, 就是 GPGPU. 也许有些出乎 Nvidia 意料的是, CUDA 没有在大家看好的科学计算领域掀起革命, 却引领了人工智能的时代. 回望历史, 展望未来, 同学们将在人类历史上找到自己的位置.

### 熟悉又陌生的 CPU

实际的 CPU

- 多个核心, 共享内存
- 并行编程
  - 每个 CPU 核心都是一个编译器, 动态 (运行时) 数据流分析和指令调度
  - 实现 CPI < 1
- 在**能效**和**性能**直接选择了后者
  - CPU 里的编译器会消耗巨量的能量, 这些能量使计算"尽快完成"
  - 但并不等同于"单位时间完成尽可能多的计算"
  - 功耗墙

面对功耗墙

1. 让一条指令能处理更多的数据
   - SIMD, 可以一次让一条指令对连续的操作数做相同运算
   - 没能解决的问题: SMID 指令依然是在 CPU 里调度的, 其参与到缓存和动态流水线中, 这意味着宽度不能做得太宽, 并行度有限
   - 提升性能只能横向拓展: 单 CPU -> 多 CPU -> 大小核 CPU
2. 用更多更简单的处理器
   - 多处理器系统, 异构多处理器: 同等面积, 处理器越简单, 数量越多

### GPU 和 GPGPU

领域专用加速器

- ISP: Image Signal Processing (手机相机)
- GPU: Graphics Processing Unit (图形渲染)
- DPU: Data Processing Unit (网络/数据处理)

NES Picture Processing Unit 绘图模型

- 以 8×8 的"贴块" (tile) 为绘制的基本单位: 背景画布 + 动态前景 (sprites)

- Sprite RAM: 256B, 64个 `Sprite = (x, y, tile, attr)`

- Massive Parallelism

  ```c
  // 逐行扫描
  for (int row = 0; row < lines; row++) {
  
      // 我们可以作弊
      #pragma omp parallel for
      for (int i = 0; i < 64; i++) {
          struct sprite *s = &SPRAM[i];
          if (s->y <= row && row < s->y + 8) {
              // 绘制 sprite
          }
      }
  
      display_row(row);
  }
  ```

- MMC1芯片: 通过 Memory Mapper 实现大幅动画

图形渲染管线

- CPU 不负责"绘制", 只负责通过某种语言"描述图形", (比如在 NES 只描述背景和 Sprite 及其位置), 剩下的处理由加速器负责
- Fixed-function pipeline
  - 每个部分都有可配置的参数, 例如, glLightfv, glMaterialfv 设置光源和材质
  - 3D 渲染管线: ![](../0_Attachment/Pasted%20image%2020250504153002.png)
- Rendering pipeline: 逐渐从固定功能到可编程
  - ![](../0_Attachment/Pasted%20image%2020250504153251.png)

- 还可以把各种计算问题转换为"图像"的计算 (物理模拟, 天气预报...)

  ```c
  void vertex_shader(  // HLSL
      float4 position : POSITION,
      float4 color : COLOR,
      out float4 oPosition : POSITION,
      out float4 oColor : COLOR
  ) {
      float r = rand(position.xyz, seed);
      float3 offset = (r - 0.5) * 0.1; // 随机扰动
      float4 newPos = position + float4(offset, 0.0);
      oPosition = mul(newPos, WorldViewProj);
      oColor = color;
  }
  ```



### AI 时代的并行编程

SIMT: Single Instruction, Multiple Threads

- 将 `shader()` 实现为线程

- 在 GPU 上启动 1902 \* 1080 个线程, 并行执行, 允许访问 shared memory (CUDA 就是这么干的)

- 并行执行时可以实现 Memory coalescing

  ```c
  void kernel(int row, int col) {
      t = ...;  // 任意 C/C++ 代码
      map[row * 1920 + col] = t;
  }
  ```

CUDA 擅长固定模式, 海量的计算

## 20 设备和驱动程序

> 输入/输出设备可以说是五花八门, 你也看到越来越多的设备上甚至 “自带电脑”. 但无论如何, 操作系统都把它们抽象成一个可以读写, 可以控制的, 实现了 struct file_operations 的文件 (操作系统对象)

### 输入/输出设备

从插入U盘到看到里面的文件, 发生了什么 (`/dev/` 下的对象不会凭空创建)

- **udev**: Linux 内核的设备管理器, 负责动态管理 `/dev` 目录下的设备节点
  - 当设备插入 (如U盘) 或移除时, udev 会根据预定义的规则 `/lib/udev/rules.d`, 自动创建/删除设备节点, 并执行相关操作, 如加载驱动, 设置权限等.
- **udisks2**: 当插入 USB 驱动器, SD 卡等存储设备时, `udisks2` 会通过 `udev` 检测到设备, 并触发后续操作 (真正负责 mount)

I/O 设备: 一个能与 CPU 交换数据的接口/控制器

-  几组约定好功能的线 (寄存器)
  - 通过握手信号从线上读出/写入数据
-  给寄存器"赋予" 一个内存地址 (Address Decoder)
  - CPU 可以直接使用**指令** (in/out/MMIO) 和设备交换数据
  - ![](../0_Attachment/Pasted%20image%2020250507164158.png)
-  GPIO (General Purpose Input/Output)
   -  仅需一根线和一条指令 (树莓派), 通过 Memory-mapped I/O 直接读取/写入电平信号
   -  从控制 LED 发光到发射核弹


### 输入/输出设备案例

串口

- COM1 (Communication 1)

  ```c
  #define COM1 0x3f8
  
  static int uart_init() {
    outb(COM1 + 2, 0);   // 控制器相关细节
    outb(COM1 + 3, 0x80);
    outb(COM1 + 0, 115200 / 9600);
    ...
  }
  
  static void uart_tx(AM_UART_TX_T *send) {
    outb(COM1, send->data);
  }
  
  static void uart_rx(AM_UART_RX_T *recv) {
    recv->data = (inb(COM1 + 5) & 0x1) ? inb(COM1) : -1;
  }
  ```

键盘控制器

磁盘控制器

- ATA (Advanced Technology Attachment)

  ```c
  void readsect(void *dst, int sect) {
    waitdisk();
    out_byte(0x1f2, 1);          // sector count (1)
    out_byte(0x1f3, sect);       // sector
    out_byte(0x1f4, sect >> 8);  // cylinder (low)
    out_byte(0x1f5, sect >> 16); // cylinder (high)
    out_byte(0x1f6, (sect >> 24) | 0xe0); // drive
    out_byte(0x1f7, 0x20);       // command (write)
    waitdisk();
    for (int i = 0; i < SECTSIZE / 4; i ++)
      ((uint32_t *)dst)[i] = in_long(0x1f0); // data
  }
  ```

打印机

- 根据指令 (字节流描述) 将文字/图片打印到纸张上
- PostScript: 一种描述页面布局的 DSL, PDF 是 PostScript 的 superset
- 实现打印机: 将汇编语言翻译成机械部件动作

总线

- 提供设备的**虚拟化**: 注册和转发, 从而实现硬件生态的**拓展性**
  - 把收到的地址 (总线地址) 和数据转发到相应的设备上
  - 例子: port I/O 的端口就是总线上的地址, IBM PC 的 CPU 其实只看到这一个 I/O 设备
- PICe 总线
  - 接口: 75W 供电, 需要 6-pin, 8-pin 的额外供电
  - 数据传输
    - PCIe 6.0 x16 带宽达到 128GB/s, 于是我们有了 800Gbps 的网卡
    - 总线自带 DMA (专门执行 memcpy 的处理器)
  - 中断管理
    - 将设备中断转发到操作系统 (Message-signaled Interrupts)

### 设备驱动程序

程序访问设备

- 设备是可以在程序之间共享的, 程序不能**直接**访问设备的寄存器
- 解决: 实现设备的**虚拟化** (就像 CPU 和内存一样)
  - Everything is a File: 像操作文件一样访问设备

设备驱动程序

- 一个 `struct file_operations` 的实现: 把系统调用**翻译**成与设备能听懂的数据 (就是一段普通的内核代码)
- 例子
  - `/dev/null`: 其读写函数就是 do nothing
  - `/proc/stat`: 实现读写操作即可 (并非真正读写了磁盘的一个文件, 而是访问操作系统所管理的对象)
  - `/dev/nuke0`

配置设备

- 设备除了传输数据之外还需要**配置** (打印机卡纸, 键盘跑马灯...)
- 实现方法
  1. 控制作为数据流的一部分 (ANSI Escape Code), 例如控制向终端输出的字体颜色
  2. 提供一个新的接口 (request-response)

ioctl

- "非数据"的设备功能几乎全部依赖 ioctl
- 设备的复杂性无法降低, 使其成了屎山

## 21 存储设备原理

### 1-Bit 的存储: 磁铁

原理: 电磁感应

磁带: 1928

- 存储特性: 价格低 (廉价材料, 几乎不涉及大规模集成电路), 容量高, 可靠性高 (适当封装)
- 读写性能: 顺序读写勉强 (需要等待定位), 随机读写几乎完全不行
- 应用场景: 冷数据的存档和备份

磁鼓: 1932

- 用旋转的二维平面存储数据 (无法内卷, 容量变小)
- 读写延迟不会超过旋转周期 (随机读写速度大幅提升)

磁盘: 1956

- ![](../0_Attachment/Pasted%20image%2020250510144955.png)
- 存储特性
  - 价格低: 高密度, 低成本
  - 容量高: 2.5D, 上万磁道
  - 可靠性高 (高速运转的机械部件是潜在的威胁)
- 读写性能
  - 顺序读写: 较高
  - 随机读写: 勉强 (需要等待定位)
- 应用场景: 计算机系统的主力数据存储 (便宜; 坏了还有可能修)
- 性能调优: 通过缓存/调度等缓解

### 1-Bit 的存储: 挖坑

光盘, Compact Disk (CD, 1980)

- 在反射平面 (1) 上挖上粗糙的坑 (0), **激光**扫过表面, 就能读出坑的信息来
- **容易复制**
- 存储特性
  - 价格极低
  - 容量高 (当然, 也没有那么高)
  - 可靠性高 (你划伤的是没有数据的那一面)
- 读写性能
  - 顺序读: 一般
  - 随机读: 低；很难写入 (CD/R & CD/RW)
- 应用场景: 一个时代的数字内容分发

### 1-Bit 的存储: 电荷

闪存, Flash Memory

- 原理: ![](../0_Attachment/Pasted%20image%2020250510145401.png)
- 存储特性
  - 价格低: 大规模集成电路
  - 容量高
  - 可靠性高: 集成电路封装, 不怕摔
- 读写性能: 极高, 而且有极高的扩展性 (电路是天然并行的), **容量越大, 速度越快**
- 应用场景: Tape is Dead, Disk is Tape, Flash is Disk, RAM Locality is King (Gim Gray, 2006)
- **FTL**: Flash Translation Layer
  - SSD, 优盘, 甚至是 TF 卡里都藏了**完整的计算机系统**
  - Wear Leveling: 用软件使写入变得"均匀" (类比虚拟内存)
  - 这也使得设备可以被"伪造"


### 存储设备在操作系统中的抽象

实现**寻址能力**的代价 (从 1-bit 到 1TB)

- 磁盘: 位置划分 + 扇区头
- 电路: 行 (字线) 和列 (位线) 选通信号
  - 这些都会消耗额外的资源 (面积)
- 解决方法: **按块访问**
  - "一块"可以共享 metadata
    - 物理分割, Erase 信号, 纠错码……
    - 磁盘是 `struct block disk[NUM_BLOCKS]`, Block 是读/写的最小单位

Linux Bio: request/response 接口

- 上层 (进程, 文件系统……) 可以任意提交请求
- 下层 (Bio + Driver) 负责调度
