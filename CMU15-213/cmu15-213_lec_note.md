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

