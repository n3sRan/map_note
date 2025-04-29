# Lecture Note

## 5 Machine Level Programming: Basics

Goal: Look assembly programs instead write.

### History of intel processors and architectures

- Complex instruction set computer, CISC
  - IA32 IA64 (failed) x86-64
- Reduced instruction set computer, RISC
  - ARM Acorn RISC Machine

### C, Assembly, Machine code

#### Assembly Characteristics: Data Types

- Integer

- Floating points data

- Code

- *No aggregate types such as arrays or structures*

#### Assembly Characteristics: Operations

- arithmetic function

- transfer data

- transfer control


#### Object Code

从C到机器代码

- C 语言代码`a.c`经过编译器 (Compiler) 的处理`gcc -0g -S`成为汇编代码`a.s`
- 汇编代码`a.s`经过汇编器 (Assembler) 的处理成为对象程序`a.o`
- 对象程序`a.o`以及所需静态库`lib.a`经过链接器 (Linker) 的处理最终成为计算机可执行的程序`a.out`

Disassembling Object Code 反汇编

```bash
objdump -d a.out
```

```bash
(gdb) disassemble
```

### Assembly Basics

AT&T标准 (Linux标准, 课堂使用): `op src, dst`

Intel标准: `op dst, src`

#### Register

![](../0_Attachment/Pasted%20image%2020241210125127.png)

`%rsp(%esp)`: 栈指针

`%rbp(%ebp)`: 基指针

#### Operand Types

##### Imm

立即数 Immediate: \$10 \$0x400

##### Reg

寄存器值 Register

##### Mem

内存值 Memory: 8 consecutive bytes of memory at address given by register (%rax) (64位机器)

- Simple Memory Addressing Modes 简单寻址
  - 普通模式: (R) 
    - 相当于`Mem[Reg[R]]`
    - `(%rcx)`
  - 移位模式:  D(R)
    - 相当于`Mem[Reg[R]+D]`
    - `8(%rbp)`
- Complete Memory Addressing Modes 完整 (通用) 寻址
  - D(Rb, Ri, S)
  - 相当于`Mem[Reg[Rb] + S*Reg[Ri] + D]`
  - S: 系数, 为空则默认为1
  - Rb: 基址寄存器, Ri: 索引寄存器, D: 常数偏移量, 为空则默认为0
  - 非常方便的算术运算方式, 被编译器使用

### Arithmetic & Logical Operations

C compiler will figure out different instruction combinations to carry out computation.

#### Address Computation Instruction

lea: load effective address

`leaq Src, Dst`

- `Src` is address mode expression
- Set `Dst` (a register) to address denoted by expression
- 用法: 取地址/**简单计算**

#### Some Arithmetic Operations

- Tow Operand Instruction 需要两个操作数的指令
  - `Operation Src, Dest`
- One Operand 需要一个操作数的指令
  - `Operation Dest`

## 6 Machine Level Programming: Control

### Control: Condition Code

#### Processor State

- Temporary Data 临时数据: `%rax...`
- Location of runtime stack: `%rsp`
- Location of current code control point (pc): `%rip`
- **Status of recent tests: conditional codes**

#### Condition Codes

根据运算结果被置0/1

- CF: Carry Flag 进位 (for unsigned)

- SF: Signed Flag 符号位 (for signed) 

- ZF: Zero Flag 零位

- OF Overflow Flag 溢出位 (for signed)

- *Not set by `leaq`*

- Implicit setting (side effect) by arithmetic operation

- Explicit setting
  - `cmpq src2, src1` (computing src1-src2 without setting destination)
  - `test src2, src1` (computing src1&src2 without setting destination)

#### Reading Condition Codes

SetX Instructions 只改变最低字节 (8位)

- `movzbl` move with zero extension from byte to long

### Conditional Branches & Moves

#### Conditional Jump

类似于C语言的里的`goto`

#### Conditional Move

如果是简单的计算, 编译器会决定先把then和else的值计算出来, 最后才判断使用哪一个(move), 提高了效率 (符合现代处理器流水线设计, 更常见).

##### Bad Cases

- 复杂的计算
- 空指针访问
- 计算时会影响变量值

### Loops

- `-Og` jump-to-middle 
- `-O1` Do-While conversion
- `for`转成`while`: `-O1` 摒弃了初始测试

### Switch Statements

- `ja`: jump above, 根据无符号数比较结果跳转
  - ppt例子: 如果x>6或x<0都会跳到默认
- jump-table: 通过类似数组索引的方式提升效率 如果是一堆if-else则时间复杂度为O(n)
- case的值为负: 加一个偏移值以避免负索引
- case数量少但值比较大, 稀疏 (0和1000000): 转为if-else, 通过二分决策树 (jump-tree) 降低时间复杂度

### Summary

#### C Control

- if-then-else
- do-while
- while, for
- switch

#### Assembler Control

- Conditional jump
- Conditional move
- Indirect jump (via jump tables)
- Compiler generates code sequence to implement more complex control

#### Standard Techniques

- Loops converted to do-while or jump-to-middle form
- Large switch statements use jump tables
- Sparse switch statements may use decision trees (if-elseif-elseif-else)

## 7 Machine Level Programming: Procedures

### Procedures

**ABI**, application binary interface, 应用程序二进制接口 (机器程序级别的接口)

- *只要每个人都坚持使用这种通用接口标准, 甚至可以使用不同的编译器来编译代码, 并且让它们能在传参与返回数据方面相互协作*

Mechanisms

#### x86-64 Stack

- `pushq Src`
- `popq Dst`

#### Calling Conventions

##### Passing control

- `call label`: 将内存顺序上的下一条指令压入栈, 将pc (`%rip`) 指向`label`
- `ret`: 相当于`popq %rip`

##### Passing data

- 传参
  - 6个寄存器: `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, `%r9`
  - 参数大于6, 调用者 (caller) 实际上将使用**自己的栈帧**来存储这些参数 (反向顺序压栈)
- 返回值: `%rax`


##### Managing local data

###### Stack-Based Languages

- Languages that support recursion
- Stack discipline
- Stack allocated in **Frames**: state for
- 在任何给定的时间内只有一个函数在运行 (单线程)

###### Stack Frames

- 被`%rbp`和`%rsp`框住
- `%rbp`可选, 用的少

###### Register Saving Conventions

- caller saved  调用者保存

- callee saved 被调用者保存

- `%r10`, `%r11`: Caller-saved temporaries, 用于存储可以被任何函数修改的临时值
- `%rbx`, `%r12`, `%r13`, `%r14`: Callee-saved Temporaries 

#### Illustration of Recursion

Handled Without Special Consideration

Also works for mutual recursion

## 8 Machine Level Programming: Data

### Arrays

#### One-dimension

声明数组: 分配数组的空间 `int A1[3]`

声明指针: 分配指针的空间(8 bytes) `int (*A3) [3]`

`int (*A4[3])`, A4是3个元素的数组, 这些元素是指向整数的指针.

- `sizeof(A4) = 24, sizeof(*A4) = 8 sizeof(**A4) = 4`

#### Nested array vs. Multi-level array

Accesses looks similar in C (pgh\[index][digit], univ\[index][digit]), but address computations very different, 底层数据类型不同, 引用也不同.

- `Mem[pgh + 20 * index + 4 * digit]`
- `Mem[Mem[univ + 8*index] + 4 * digit]`

### Structures

Alignment 对齐

### Floating Point

Programming with SSE3

- `addss %xmm0, %xmm1`: add a single scalar 对单精度标量做加法运算
- `addps`: 同时做4个单精度浮点数加法运算 (SIMD, single instruction multiple data 单指令多数据运算)
- `addsd`: double precision
- 所有寄存器都是调用者保存 


## 9 Machine Level Programming: Advanced Topics

### Memory Layout

程序在内存中是如何组织的 (x86-64 Linux)

从高地址到低地址

- Stack 栈
  - 运行时栈最大为8MB
  - 从高地址往低地址生长
- Heap 堆
  - 从低地址往高地址生长
- Data 数据区
- Text 代码区

### Buffer Overflow

#### Vulnerability

- Unsafe String Library Code, 会导致缓冲区溢出.
  - `gets()`, `strcpy()`, `strcat()`, `scanf()`

#### Avoid overflow vulnerabilities in code

- 用更安全的方法
  - `fgets()`, `strncpy()`
- System-level protections
  - ASLR, address space layout randomization 每次程序运行时栈和堆的地址会变化
  - Nonexecutable code segments 给不同区域的内存打上权限标记 (比如给栈打上只读只写不可执行, 阻止代码注入式攻击)
- Stack Canaries认证机制 (矿工金丝雀), GCC默认打开的栈保护机制, 每次运行时在调用栈底设一个金丝雀值, 一旦该值被修改则终止运行

#### ROP

Return-Oriented Programming Attacks, 返回导向编程攻击

类比: 从一个人的说话录音中截取片段, 拼接成新的一段话来实施诈骗.

利用程序中原有的多个代码片段 (gadget), 一般是某个操作+`ret`进行拼接, 来达到修改寄存器等操作以攻击程序. (机器码层级, 做一次Attack Lab就懂了)

### Unions

union, 将同一块内存区域以不同形式看待(区别于强制类型转换)

- Little Endian 小端
  - IA-32
  - x86-64
  - ![](../0_Attachment/Pasted%20image%2020250220140617.png)
- Big Endian 大端
  - Sun
  - ![](../0_Attachment/Pasted%20image%2020250220140853.png)

## 10 Program Optimization

There's more to performance than asymptotic complexity.

### Generally Useful Optimizations

- Code motion/precomputation
  - 把相同结果的重复运算移到循环外
- Strength reduction
  - 减少开销较大的乘除法
- Sharing of common subexpressions
  - 重用部分表达式 (需要开销较大的计算) 的计算结果
- Removing unnecessary procedure calls
  - 不要把`strlen()`放在for循环内

### Optimization Blockers

- Procedure calls
- Memory aliasing
- 编译器对过程调用 (可能会产生副作用) 以及内存访问 (涉及对同一内存的更改) 的优化相当保守.

### Exploiting Instruction-Level Parallelism

充分利用现代处理器超标量流水线设计, 通过并发计算提高硬件利用率.

### Dealing with Conditionals

- 分支问题有时会影响性能.

- 解决: 分支预测.


## 11 The Memory Hierarchy 

详细版本: [存储器分层体系结构](../NJU24F_COA/3_存储器分层体系结构.md)

### Storage technologies and trends

- RAM
  - SRAM, DRAM
- Nonvolatile Memories
  - ROM, PROM, EPROM, EEPROM, Flash memory
- Disk
- SSD
- Bus

### Locality of reference

- Temporal locality 时间局部性
- Spatial locality 空间局部性

### Caching in the memory hierarchy

*这里的Cache指更广泛的缓存概念.*

![](../0_Attachment/Pasted%20image%2020241210180611.png)

Fundamental idea of a memory hierarchy

- For each k, the faster, smaller device at level k serves as a **cache** for the larger, slower device at level k+1.

General Cache Concepts

- Hit
- Miss
  - Cold (compulsory) miss
  - Conflict miss
  - Capacity miss

## 12 Cache Memories

### Cache memory organization and operation

详细版本: Cache

#### Organization

- Set, S: 每缓存组数
- Line, E: 每组行数 (块数)
- Byte, B: 每块数据区字节数
- Cache size = C * E * B data bytes

#### Performance Metrics

- Miss Rate
- Hit Time
- Miss Penalty

#### Writing Cache Friendly Code

- Our qualitative notion of locality is quantified through our understanding of cache memories, 我们对局部性的定性概念被**量化**了.
- Focus on the inner loops of the core functions. (Minimize the misses)
  - Repeated references to variables are good (temporal locality).
  - Stride-1 reference patterns are good (spatial locality).

### Performance impact of caches

#### The Memory Mountain

![](../0_Attachment/Pasted%20image%2020250220230301.png)

#### Rearranging loops to improve spatial locality

Matrix Multiplication

- 把i-j-k优化为k-i-j (保证最内部循环按行遍历矩阵)

#### Using blocking to improve temporal locality

Blocked Matrix Multiplication

- No blocking (i-j-k): $(9/8)* n^3$
- Blocking: $1/(4B)* n^3$, $3B^2 < C$

## 13 Linking

### Linking

Static Linking

![](../0_Attachment/Pasted%20image%2020250221180343.png)

Linker

- Modularity
- Efficiency

What Do Linkers Do?

1. Symbol Resolution
2. Relocation

Object Files

- Relocatable object file (`.o`)
- Executable object file (`a.out`)
- Shared object file (`.so`) (Called Dynamic Link Libraries, DLLs in Windows)

Executable and Linkable Format (ELF)

- ![](../0_Attachment/Pasted%20image%2020250221181200.png)

Linker Symbols

- Global symbols
- External symbols
- Local symbols

Static Libraries

- `.a` archive files
- ![](../0_Attachment/Pasted%20image%2020250223133022.png)

Shared Libraries

- Dynamic Linking at Load-time / Run-time
- ![](../0_Attachment/Pasted%20image%2020250223133206.png)

### Library Interpositioning

- occur at Compile time, Link time, Load / Run time
- Applications: security, debugging, monitoring and profiling (类似于61A里的函数wrapper/修饰器)

## 14 Exceptional Control Flow: Exceptions and Processes

## 15 Exceptional Control Flow: Signals and Nonlocal Jumps

## 16 System-Level I/O

### Unix I/O

#### Overview

- A Linux file is a sequence of m **bytes**: $B_0, B_1, ..., B_{m-1}$
- All **I/O devices** are represented as files
  - `/dev/sda2`
  - `/dev/tty2`
- Even the **kernel** is represented as a file
  - `/boot/vmlinuz-3.13.0-55-generic`: kernel image
  - `/proc`: kernel data structures
- Allows kernel to export **simple interface** called Unix I/O
  - `open()`, `close()`, `read()`, `write()`, `lseek()`

#### File Types

- Regular file: Contains arbitrary data
- Directory: Index for a related group of files
- Socket: For communicating with a process on another machine
- Others: Named pipes, Symbolic links, Character and block devices

#### Regular Files

- **Applications** often distinguish between text files and binary files. (**Kernel** doesn’t know the difference.)
  - Text files are regular files with only ASCII or Unicode characters
  - Binary files are everything else
- Text file is sequence of text lines. Text line is sequence of chars terminated by newline char.
- End of line (EOL)
  - Linux and Mac OS: `\n` (0xa)
  - Windows and Internet protocols: `\r\n` (0xd 0xa)

#### Directories

- Consists of an array of links, each link maps a filename to a file.
- Each directory contains at least two entries
  - `.`, a link to itself
  - `..`, 
  - a link to the parent directory in the directory hierarchy
- Directory Hierarchy: ![](../0_Attachment/Pasted%20image%2020250408161229.png)

#### On Short Counts

- can occur
  - Encountering (end-­‐of-­‐file) EOF on reads
  - Reading text lines from a terminal
  - Reading and writing network sockets
- never occur
  - Reading from disk files (except for EOF)
  - Writing to disk files
- Best practice is to always allow for short counts.

### RIO

robust I/O package

Buffered I/O

- File has associated buffer to hold bytes that have been read from file but not yet read by user code *(不用应用程序每次需要一个或少量字符时都进行一次系统调用, 而是先调用一次, 把若干字符缓存, 后续应用程序只需从缓冲区提取)*
- ![](../0_Attachment/Pasted%20image%2020250429104006.png)

### Metadata, sharing, and redirection

#### File Metadata

- Per-­‐file metadata maintained by kernel, accessed by users with the `stat()` and `fstat()`
- usage: `man stat`, `man 2 stat`
- ![](../0_Attachment/Pasted%20image%2020250408155934.png)

#### File Sharing

- Before `fork()`: ![](../0_Attachment/Pasted%20image%2020250408160137.png)
- After `fork()`: ![](../0_Attachment/Pasted%20image%2020250408160155.png)

#### I/O Redirection

- example: `ls > foo.txt`
- ![](../0_Attachment/Pasted%20image%2020250408160327.png)

### Standard I/O

- Contained by the C standard library
- Standard I/O models open files as **streams**, which are abstraction for a file descriptor and a buffer in memory.

### Closing remarks

![](../0_Attachment/Pasted%20image%2020250408160735.png)

*some pros and cons of them...*

#### Cons of Standard I/O

- not async-‐signal-­safe, and not appropriate for signal handlers
- not appropriate for input and output on network sockets

#### Functions you should never use on binary files

- Text-oriented I/O such as `fgets()`, `scanf()`, `rio_readlineb()`
- String functions

## 17 Virtual Memory Concepts

## 18 Virtual Memory Systems

## 19 Dynamic Memory Allocation: Basic

### Basic concepts

Dynamic Memory Allocation

- Programmers use dynamic memory allocators (such as `malloc()`) to acquire VM at run time.
- Allocator maintains heap as collection of variable sized blocks, which are either **allocated** or **free**
  - Explicit allocator: `malloc()` and `free()` in C
  - Implicit allocator: garbage collection in Java

Constraints

- Applications
  - Can issue arbitrary sequence of malloc and free requests
  - free request must be to a malloc’d block
- Allocators
  - Can’t control number or size of allocated blocks
  - Must respond immediately
  - Must allocate blocks from free memory
  - Must align blocks to satisfy all alignment requirements
  - Can manipulate and modify only **free** memory
  - Can’t move the allocated blocks once they are malloc’d

Performance Goal

- Throughput: Number of completed requests per unit time
- Peak Memory Utilization: aggregate payload / heap size

Fragmentation

- internal fragmentation: ![](../0_Attachment/Pasted%20image%2020250421150510.png)
- external fragmentation: ![](../0_Attachment/Pasted%20image%2020250421150525.png)

How Much to Free

- Use an extra word  (called header field or header) to keep the **length**  of the block

Keeping Track of Free Blocks

1. **Implicit list** using length—links all blocks: ![](../0_Attachment/Pasted%20image%2020250421151313.png)
2. **Explicit list** among the free blocks using pointers: ![](../0_Attachment/Pasted%20image%2020250421151329.png)
3. Segregated free list: Different free lists for different size classes
4. Blocks sorted by size: use a balanced tree to track

### Implicit free lists

Finding a Free Block (Placement policy)

- First-fit, next-fit, best-fit, ...

Allocating in Free Block (Splitting policy)

Freeing a Block (Coalescing policy: Immediate coalescing)

- Boundary tags: Replicate size/allocated word at bottom of free blocks
  - ![](../0_Attachment/Pasted%20image%2020250421152014.png)
- Optimization: Boundary tag needed only for free blocks
  - ![](../0_Attachment/Pasted%20image%2020250421152039.png)

## 20 Dynamic Memory Allocation: Advanced

### Explicit free lists

Maintain list(s) of free blocks, not all blocks.![](../0_Attachment/Pasted%20image%2020250422160703.png)

Freeing With Explicit Free Lists

- Unordered
  - LIFO (last-­‐in-­‐first-­‐out) policy: Insert freed block at the beginning of the free list
  - FIFO (first-­‐in-­‐first-­‐out) policy: Insert freed block at the end of the free list
  - Pro: simple and constant time
  - Con: studies suggest fragmentation is worse than address ordered
- Address-­‐ordered policy
  - Insert freed blocks so that free list blocks are always in address order: `addr(prev)` < `addr(curr)` < `addr(next)`
  - Con: requires search
  - Pro: studies suggest fragmentation is lower than LIFO/FIFO

### Segregated free lists

Each size class of blocks has its own free list: ![](../0_Attachment/Pasted%20image%2020250422161057.png)

Advantages

- Higher throughput: log time for power-­‐of-­‐two size classes
- Better memory utilization
  - First-­fit search of segregated free list approximates a best-­‐fit search of entire heap.
  - Extreme case: Giving each block its own size class is equivalent to best-­fit

### Garbage collection

Automatic reclamation of heap-­‐allocated storage, application never has to free.

Common in many dynamic languages like Python, Java...

Memory as a Graph

- Each block is a node in the graph
- Each pointer is an edge in the graph
- Locations not in the heap that contain pointers into the heap are called root nodes (e.g. registers, locations on the stack, global variables)
- ![](../0_Attachment/Pasted%20image%2020250422161442.png)

Mark and Sweep Collecting

- When out of space
  - Use extra mark bit in the head of each block
  - Mark: Start at roots and set mark bit on each reachable block
  - Sweep: Scan all blocks and free blocks that are not marked
  - ![](../0_Attachment/Pasted%20image%2020250422161650.png)

## 21 Network Programming: Part 1

> 基于 DS V3 整理, 删除了计算机网络相关内容

### Concept

客户端-服务器模型 (大多数网络应用基于客户端-服务器架构)

- 服务器: 管理资源 (如数据, 服务) , 处理客户端请求.
- 客户端: 发起请求并接收服务 (如浏览器, APP).
- 交互模式: 客户端通过请求激活服务器 (类似自动售货机).
- 进程与主机: 客户端和服务器是运行在不同或相同主机上的**进程**.

互联网协议 (IP) 

1. 核心功能: 
   - 命名机制: 统一IP地址格式 (如IPv4的32位地址).
   - 数据传输: 通过封装数据包 (含头部和负载) 跨网络传输.
   - 路由选择: 路由器根据目标地址转发数据包.
2. IPv4与IPv6: 
   - IPv4: 32位地址 (如`128.2.194.242`) , 当前主流.
   - IPv6: 128位地址, 兼容性仍在过渡中.
3. IP地址表示: 
   - 结构体: `struct in_addr`存储网络字节序 (大端序) 的32位地址.
   - 点分十进制: 如 `0x8002C2F2` 转为 `128.2.194.242`.

域名系统 (DNS) 

- 将域名 (如`www.cs.cmu.edu`) 映射到IP地址.
- 特点: 
  - 分布式数据库, 支持多对一, 一对多映射.
  - 命令行工具: `nslookup` 查询域名解析结果
  - 本地域名`localhost`固定映射到`127.0.0.1` (回环地址).

互联网连接

1. 连接特性
   - 点对点: 两端进程通信. 
   - 全双工: 双向同时传输. 
   - 可靠传输: 数据按序到达 (TCP协议).
2. 端口与套接字
   - 端口: 16位整数标识进程 (如 HTTP 服务使用端口 80).
   - 套接字 (Socket) : 通信端点, 由IP地址和端口唯一标识, 如`(cliaddr:cliport, servaddr:servport)`.

### Sockets Interface

概览 (CSAPP 库函数的封装方向): ![](../0_Attachment/Pasted%20image%2020250429153514.png)

#### Socket Address Structures

Generic socket address

- 用作 `connect()`, `bind()`, `accept()` 的地址参数

- 设计背景: 早期 C 语言缺乏 `void*` 通用指针

  ```c
  typedef struct sockaddr SA;
  struct sockaddr {
  	uint16_t sa_family; /* Protocol family */
  	char sa_data[14]; /* Address data. */
  };
  ```

- ![](../0_Attachment/Pasted%20image%2020250429152340.png)

Internet-­‐specific socket address

- 需强制转换为 `struct sockaddr*` 传递到相关函数

  ```c
  struct sockaddr_in {
      uint16_t      sin_family; /* Protocol family */
      uint16_t      sin_port;
      struct in_addr sin_addr;
      unsigned char  sin_zero[8];
  };
  ```

- ![](../0_Attachment/Pasted%20image%2020250429152359.png)

`getaddrinfo()`

- 网络编程中的关键函数, 其核心功能是**自动化处理地址和端口转换**, 使代码与协议 (如 IPv4/IPv6) 无关

  ```c
  int getaddrinfo(const char *host, /* Hostname or address */
  				const char *service, /* Port or service name */
  				const struct addrinfo *hints,/* Input parameters */
  				struct addrinfo **result); /* Output linked list */
  
  void freeaddrinfo(struct addrinfo *result); /* Free linked list */
  
  const char *gai_strerror(int errcode); /* Return error msg */
  ```

- `addrinfo`

  ```c
  struct addrinfo {
      int ai_flags; /* Hints argument flags */
      int ai_family; /* First arg to socket function */
      int ai_socktype; /* Second arg to socket function */
      int ai_protocol; /* Third arg to socket function */
      char *ai_canonname; /* Canonical host name */
      size_t ai_addrlen; /* Size of ai_addr struct */
      struct sockaddr *ai_addr; /* Ptr to socket address structure */
      struct addrinfo *ai_next; /* Ptr to next item in linked list */
  };
  ```

  ![](../0_Attachment/Pasted%20image%2020250429152838.png)
  
- 示例 *(DS V3 生成)*

  - 使用 `getaddrinfo()` (推荐)

    ```c
    struct addrinfo hints = {0};
    hints.ai_family = AF_UNSPEC; // 支持 IPv4/IPv6
    hints.ai_socktype = SOCK_STREAM;
    
    struct addrinfo *result;
    getaddrinfo("www.example.com", "80", &hints, &result);
    
    // 遍历 result 链表尝试连接
    int sockfd;
    for (struct addrinfo *p = result; p != NULL; p = p->ai_next) {
        sockfd = socket(p->ai_family, p->ai_socktype, p->ai_protocol);
        if (connect(sockfd, p->ai_addr, p->ai_addrlen) == 0) {
            break; // 连接成功
        }
        close(sockfd);
    }
    freeaddrinfo(result);
    ```

  - 不使用 `getaddrinfo()` (需要自行适配协议)

    ```c
    // 仅支持 IPv4，硬编码 DNS 查询和端口
    struct hostent *h = gethostbyname("www.example.com");
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(80); // 未处理服务名转换（如 "http"→80）
    memcpy(&addr.sin_addr, h->h_addr, h->h_length);
    
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);
    connect(sockfd, (struct sockaddr*)&addr, sizeof(addr));
    ```


`getnameinfo()`

- 反向转换 (地址到域名/服务名)

  ```c
  int getnameinfo(const SA *sa, socklen_t salen, /* In: socket addr */
                  char *host, size_t hostlen, /* Out: host */
                  char *serv, size_t servlen, /* Out: service */
                  int flags); /* optional flags */
  ```


#### Socket Interface Functions

`socket()`

- 创建套接字描述符
- `int socket(int domain, int type, int protocol);`
- 最佳实践: 通过 `getaddrinfo` 提供 `addr` 和 `addrlen`

`bind()`

- 将服务器地址与套接字描述符关联
- `int bind(int sockfd, SA *addr, socklen_t addrlen);`
- 最佳实践: 通过 `getaddrinfo` 提供 `addr` 和 `addrlen`

`listen()`

- `int listen(int sockfd, int backlog);`
- 将主动套接字转为监听套接字, `backlog` 指定待处理连接队列长度

`accept()`

- `int accept(int listenfd, SA *addr, socklen_t *addrlen);`
- 阻塞等待监听描述符上的连接请求, 客户端通过 `connect` 发起请求, 返回已连接描述符用于通信

`connect()`

- `int connect(int clientfd, SA *addr, socklen_t addrlen);`
- 向服务器发起连接请求

## 22 Network Programming: Part 2

### Socket Interface

#### Socket Helper Function

- 建立客户端连接: `int open_clientfd(char *hostname, char *port)`
- 创建监听套接字: `int open_listenfd(char *port)`

### Demo

#### Echo Client & Iterative Echo Server

- *直接读源码*

#### Tiny Web Server

- *也可以直接读源码*, 以下是一些概念
- Web Content: a sequence of bytes with an associated MIME type
- Static and Dynamic Content
  - Static content: content stored in files and retrieved in response to an HTTP request, like HTML files, images...
  - Dynamic content: content produced on-­‐the-­‐fly in response to an HTTP request, like content produced by a program executed by the server on behalf of the client
- URL
- HTTP Requests

#### Test Tool

- telnet

## 23 Concurrent Programming

> 基于 DS V3 整理

### Concurrent Programming is Hard

1. **难点核心**: 
   - 人脑习惯顺序思维, 时序概念容易误导
   - 穷举计算机系统中的事件序列几乎不可能
   - 主要问题类型: 
     - **竞态条件**: 结果依赖不可控的调度决策 (如航班最后座位分配)
     - **死锁**: 资源分配不当导致进程停滞 (如交通僵局)
     - **活锁/饥饿/公平性**: 外部事件或调度策略阻碍任务进展 (如排队被插队)
2. **传统迭代服务器的缺陷** (Tiny Web): 
   - 顺序处理请求 (处理完 Client1 才能处理 Client2)
   - 当服务器阻塞在某个客户端时, 其他客户端无法响应
   - 图示说明: 客户端请求在 TCP 监听队列中堆积 (TCP backlog)

### Concurrent Server Design

#### Process-based

- 原理: 为每个 client `fork()` 子进程, 父子进程共享监听描述符副本

  ```c
  while(1) {
    connfd = Accept(...);
    if (Fork() == 0) {
      Close(listenfd);
      echo(connfd);
      Close(connfd);
      exit(0);
    }
    Close(connfd);
  }
  ```

- 问题与要点

  - **僵尸进程处理**: 需注册 `SIGCHLD` 处理函数
  - **描述符引用计数**: `fork()`后 `connfd` 引用计数=2, 必须显式关闭
  - **TCP监听队列**: **内核**维护未完成连接队列 (SYN_RCVD)和已完成队列 (ESTABLISHED)

- 优缺点

  - ✅ 天然隔离, 无共享状态
  - ✅ 简单直接
  - ❌ 进程控制开销大
  - ❌ 进程间通信复杂 (需管道/共享内存)

#### Event-based

- 原理
  - **单进程**使用 I/O 多路复用 (`select`/`epoll`)
  - 维护活跃连接集合
  - 事件循环处理: 
    1. 监听描述符就绪 -> accept新连接
    2. 连接描述符就绪 -> 处理请求
- 关键特点
  - 完全控制调度策略
  - 需手动管理部分HTTP请求 (如不完整请求头)
- 优缺点
  - ✅ 单控制流, 易调试
  - ✅ 高性能 (nginx/Node.js采用)
  - ❌ 代码复杂度高
  - ❌ 无法利用多核 (如果想利用多核必须其多个进程)
  - ❌ 颗粒度并发困难

#### Thread-based

线程

- 线程 = 独立控制流 + 共享地址空间
- 线程上下文: 寄存器/栈指针/程序计数器
- 线程共享: 代码段/全局数据/打开文件

实现

- **线程分离**: 必须调用`pthread_detach`避免内存泄漏

- **资源管理**: `connfd` 需动态分配防止共享栈风险

- **线程安全**: 所有调用函数需线程安全

  ```c
  // main()
  while(1) {
    connfd_ptr = malloc(...); // 避免竞态
    *connfd_ptr = Accept(...);
    Pthread_create(..., thread, connfd_ptr);
  }
  
  void* thread(void* vargp) {
    int connfd = *((int*)vargp);
    Pthread_detach(pthread_self());
    free(vargp);
    echo(connfd);
    Close(connfd);
    return NULL;
  }
  ```

线程 vs 进程

- 相似性: 独立控制流/并发执行/上下文切换
- 核心差异
  - 线程共享地址空间
  - 线程创建开销小 (Linux: 10K cycles vs 20K cycles)
  - 资源自动回收 (进程需显式 `wait`)
