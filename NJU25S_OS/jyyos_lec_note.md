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

> 程序就是状态机; 状态机可以用程序表示. 因此：我们可以用更"简单"的方式 (例如 Python) 描述状态机, 建模操作系统上的应用, 并且实现操作系统的可执行模型. 而一旦把操作系统, 应用程序当做 “数学对象” 处理, 那么我们图论, 数理逻辑中的工具就能被应用于处理程序, 甚至可以用图遍历的方法证明程序的正确性. 

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

> 通过 freestanding 的 shell, 我们阐释了 “可以在系统调用上创建整个操作系统应用世界” 的真正含义: 操作系统的 API 和应用程序是互相成就, 螺旋生长的: 有了新的应用需求, 就有了新的操作系统功能. 而 UNIX 为我们提供了一个非常精简, 稳定的接口 (fork, execve, exit, pipe ,...), 纵然有沉重的历史负担, 它在今天依然工作得很好.

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

- 一对 “管道” 提供双向通信通道
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
  - 子进程：stdin/stdout/stderr 指向 slave
  - 父进程：从 master 读取输出显示到屏幕; 将键盘输入写入 master

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

窗口和多任务：终端可以有 “一个前台进程组”

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

实际上, 我们看到的计算机系统中的一切都是由应用程序 “完成” 的, 操作系统只是提供系统调用这个非常原始的服务接口. 正是系统调用 (包括操作系统中的对象) 这个稳定的, 向后兼容的接口随着历史演化和积累, 形成了难以逾越的技术屏障, 在颠覆性的技术革新到来之前, 另起炉灶都是非常困难的.
