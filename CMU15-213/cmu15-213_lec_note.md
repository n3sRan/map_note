## Lecture 5

Machine Level Programming I: Basics

look assembly programs instead write

### History of intel processors and architectures

一些历史

Complex instruction set computer CISC

Reduced instruction set computer RISC

IA32 IA64 (failed) x86-64

ARM Acorn RISC Machine

### C, Assembly, Machine code

三者的关系

Assembly Code View

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

汇编基础

AT&T标准 (Linux标准)

#### Register

寄存器

![](../0_Attachment/Pasted%20image%2020241210125127.png)

`%rsp(%esp)`: 栈指针

`%rbp(%ebp)`: 基指针

#### Operand Types

操作数类型

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

算术和逻辑操作

C compiler will figure out different instruction combinations to carry out computation.

#### Address Computation Instruction

lea: load effective adress

`leaq Src, Dst`

- `Src` is address mode expression
- Set `Dst` (a register) to address denoted by expression
- 用法: 取地址/简单计算

#### Some Arithmetic Operations

- Tow Operand Instruction 需要两个操作数的指令
  - `Operation Src, Dest`
- One Operand 需要一个操作数的指令
  - `Operation Dest`

## Lecture 6

### Control: Condition Code

状态码?

#### Processor State

- Temporary Data 临时数据: %rax...
- Location of runtime stack: $rsp
- Location of current code control point: %rip
- **Status of recent tests: conditional codes**

#### Condition Codes

有哪些, 如何被设置

- CF: Carry Flag 进位 (for unsigned)

- SF: Signed Flag 符号位 (for signed) 

- ZF: Zero Flag 零位

- OF Overflow Flag 溢出位 (for signed)

- *not set by `leaq`*

- Implicit setting (side effect) by arithmetic operation

- explicit setting
  - `cmpq src2, src1` (computing src1-src2 without setting destination)
  - `test src2, src1` (computing src1&src2 without setting destination)

#### Reading Condition Codes

如何读取

SetX Instructions 只改变最低字节 (8位)

`movzbl` move with zero extension from byte to long

### Conditional Branches & Moves

#### Conditional Jump

类似于C语言的里的`goto`

#### Conditional Move

如果是简单的计算, 编译器会决定先把then和else的值计算出来, 最后才判断使用哪一个(move), 提高了效率 (符合现代处理器流水线设计, 更常见).

##### Bad Cases

- 难的计算
- 空指针访问
- 计算时会改变变量值
- ...

### Loops

-Og 优化 jump-to-middle 

-O1 Do-While conversion

For 转成while -O1 摒弃了初始测试

### Switch Statements

`ja`: jump above 根据无符号数比较结果跳转 (ppt例子: 如果x>6或x<0都会跳到默认) 

jump-table: 通过类似数组索引的方式提升效率 如果是一堆if-else则时间复杂度为n

case的值为负: 加一个偏移值 避免负索引

case数量少但值比较大, 稀疏(0和1000000): 转为if-else, 通过二分决策树(jump-tree)降低时间复杂度

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

## Lecture 7

### Procedures

ABI application binary interface 应用程序二进制接口 (机器程序级别的接口)

*只要每个人都坚持使用这种通用接口标准, 甚至可以使用不同的编译器来编译代码, 并且让它们能在传参与返回数据方面相互协作*

Mechanisms

#### Stack Structure

x86-64 Stack

`pushq Src`

`popq Src`

#### Calling Conventions

##### Passing control

`call label` `ret` 完成了控制部分

##### Passing data

传参: 6个寄存器: %rdi %rsi %rdx %rcx %r8 %r9

返回值: %rax

##### Managing local data

###### Stack-Based Languages

- Languages that support recursion
- Stack discipline
- Stack allocated in **Frames**: state for
- 在任何给定的时间内只有一个函数在运行 (单线程)

###### Stack Frames

%rbp可选, 用的少

传递6个以上的参数时, 调用者实际上将使用自己的栈帧来存储这些参数 (反向压栈 )

###### Register Saving Conventions

caller saved  调用者保存

callee saved 被调用者保存

%r10 %r11 Caller-saved temporaries 用于存储可以被任何函数修改的临时值

%rbx %r12 %r13 %r14 Callee-saved Temporaries 

#### Illustration of Recursion

Handled Without Special Consideration

Also works for mutual recursion

## Lecture 8

### Arrays

#### One-dimension

声明数组: 分配数组的空间 `int A1[3]`

声明指针: 分配指针的空间(8 bytes) `int (*A3) [3]`

int (\*A4[3]) A4是3个元素的数组 这些元素是指向整数的指针 `sizeof(A4) = 24, sizeof(*A4) = 8 sizeof(**A4) = 4`

#### Nested array vs. Multi-level array

Accesses looks similar in C (pgh\[index][digit], univ\[index][digit]), but address computations very different 底层数据类型不同, 引用也不同

Mem[pgh + 20\*index + 4\*digit]

Mem[Mem[univ + 8\*index] + 4\  *digit]

### Structures

Alignment 对齐

### Floating Point

Programming with SSE3

addss %xmm0, %xmm1 add a single scalar 对单精度标量做加法运算

addps

同时做4个加法运算 (SIMD, single instruction multiple data 单指令多数据运算)

addsd double precision

所有寄存器都是调用者保存 

## Lecture 9

Machine-­‐Level Programming V: Advanced Topics

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

缓冲区溢出

#### Vulnerability

- Unsafe String Library Code
  - `gets()`, `strcpy()`, `strcat()`, `scanf()`

#### Avoid overflow vulnerabilities in code

- 用更安全的方法
  - `fgets()`, `strncpy()`
- system-level protections
  - ASLR, address space layout randomization 每次程序运行时栈和堆的地址会变化
  - Nonexecutable code segments 给不同区域的内存打上标记
- Stack Canaries 认证机制 (矿工金丝雀) gcc默认打开的栈保护机制

另外一种攻击方式: Return-Oriented Programming Attacks 返回导向编程

### Unions

union 将同一块内存区域以不同形式看待(区别于强制类型转换)

- Little Endian 小端
  - IA-32, x86-64
- Big Endian 大端
  - Sun

## Lecture 10

Program Optimization 程序优化

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

编译器对过程调用 (可能会产生副作用) 以及内存访问 (涉及对同一内存的更改) 的优化相当保守.

- Procedure calls
- Memory aliasing

### Exploiting Instruction-Level Parallelism

充分利用现代处理器超标量流水线设计, 通过并发计算提高硬件利用率.

### Dealing with Conditionals

分支问题有时会影响性能.

分支预测.

## Lecture 11

The Memory Hierarchy 存储器层级结构

详细版本: [存储器分层体系结构](../NJU24F_COA/存储器分层体系结构.md)

### Storage technologies and trends

- RAM
- Disk
- SSD
- Bus

### Locality of reference

- Temporal locality 时间局部性
- Spatial locality 空间局部性

### Caching in the memory hierarchy

![](../0_Attachment/Pasted%20image%2020241210180611.png)

Fundamental idea of a memory hierarchy

- For each k, the faster, smaller device at level k serves as a **cache** for the larger, slower device at level k+1.

General Cache Concepts

- Hit
- Miss
  - Cold (compulsory) miss
  - Conflict miss
  - Capacity miss