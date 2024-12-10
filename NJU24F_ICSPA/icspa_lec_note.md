# Lecture Notes

## 10 系统编程和基础设施

vim: 通过map实现一键编译+测试

### 开发的基础设施

我们还在面向"浪费时间"编程

#### 批量过程的效率

本质: 用编程取代重复劳动, 优化基础设施

培养程序员思维: 如果发现一个事情要重复3遍以上, 本能地将要重复做的事情去优化得更好, 因为自己并不知道未来还会去重复做多少次.

##### 正则表达式

INSTPAT中的宏匹配怎么从问号`?`改成星号`*`?

##### 批量生成相似代码

如何批量写50条`#define XN 随机数`?

- 通过命令行工具的组合生成代码

  ```bash
  seq 1 50 | shuf | cat -n | awk '{print "\#define X" $1 " " $2}'
  ```

##### 记录回放
vim实现

- normal模式下按下`qa`, (这里的a, b, c是指寄存器名称, vim会把录制好的宏放在这个寄存器中), 开始宏录制, 完成操作后再在normal模式按`q`, 结束宏录制
- normal模式下按下`@a`, 即重复录制的宏操作 (也可以在前面输入要重复的次数)

#### 基础设施的本质

通过适当的配置/脚本减少思维中断的时间, 提高连贯性, 保持短时记忆活跃

基本原则: 如果认为有提高效率的可能性, 一定有人已经做了

- 如果一个make过程需要2分钟, 那么我们可能会选择刷会手机 (或干别的事) 而不是干等, 思维容易就此中断
- 开启多核编译, 加快编译过程
- 配置一键编译, 加快输命令的过程

#### 心态分析

为什么我们会更倾向与做一个重复的操作而不是想办法优化?

- 惰性
- STFW需要一定时间, 短期收益为负 (很急)
- 对于一个小白, 可能会花更长时间在"优化"这件事上, 从而惧怕去尝试, 导致能力没有一点提升

### 调试的基础设施

#### 为什么debug这么困难

- 因为bug的触发经历了漫长的过程
- 需求 -> 设计 -> 代码 -> fault (bug) -> error (程序状态错) -> failure
- 我们只能观测到failure
- Fault和Error是一一对应的, 但failure和error却不是
- 我们可以检查状态的正确性, 但非常耗时
- bug的引入不可避免
- 对于PA来说, failure还算显而易见

#### 防御性编程 assert

error暴露得越晚, 越难调试

追随导致assert failure的变量值通常可以快速定位到bug

how: 写好读, 容易验证的代码, 在代码中添加更多的断言 (也可以理解为预防式编程, 对你的期待添加一个assert)

进阶: 如果在每一次指针访问时都添加一个断言呢?

- 编译选项: -fsanitize=address, Address Sanitizer, 动态程序分析

### 系统编程: 建立基础设施

*如何抱大腿?*

#### Delta Debugging

假设程序$P$会fail, $P_d$会pass, 且假设两份程序所有函数行为相同

- 将$P_d$中的函数$f$替换成P中的$f$
  - 依然通过: $f$实现正确
  - 否则: $f$有bug

#### Differential Testing

如果没有代码, 只有二进制文件呢?

- 每一条指令执行完后, 分别将两个程序所有寄存器的值打印出来比较, 就能**指令级**定位到出错的位置

真正的大腿: QEMU, ICS-PA = 缩水版QEMU

qemu-diff实现:

1. 启动gdb连接QEMU
2. 用`gdb_si()`在QEMU中执行一条指令
3. 比较指令执行后寄存器是否有区别

PA代码`dut.c`中没有diff-test的实际代码, 只有一堆函数指针, 这些函数封装了参考实现的功能, 使得既可以和QEMU diff-test, 又可以和NEMU, Spike 进行diff-test (相当于主板上内存的插槽, 需要比较哪个就把哪个插上去)

#### 小结

当轮子不够用时就去造轮子

- 调试困难, 想参考别人的实现
  - 替换调试法: 将正确实现的代码文件逐个替换 -> 改进: 逐个函数替换 -> 两个程序同时跑, 每执行一条指令对比一次状态
  - 学长/大神的"大腿"不够"大" -> 引入QEMU, Spike
  - 这就是diff-test
- 难以贯通多门实验课
  - Abstract Machine 作为中间层

轮子还不够用呢?

## 11 中断与分时多任务

为什么 `while(1);`死循环不会把电脑卡死?

CPU如何从这个"取指-译码-执行"的循环做一些"其他"的事情? 即如何响应中断?

### 进程与控制流

程序 (静态概念)

进程 (动态概念)

一个程序可能对应不同的进程 (运行多个hello, 每个hello加上不同的参数)

每个进程具有独立的私有地址空间 (虚拟地址空间), 即每个进程都认为自己拥有连续, 完整, 可寻址的内存空间.

程序正常执行的控制流: 取指-译码-执行

- 按顺序取下一条指令
- 通过CALL/RET等指令跳转到目标地址

**异常的控制流**

- 如何产生?
  - 来自内部的异常: 缺页, 越权, 溢出...
  - 其他原因打断正常执行: 外部中断 (打印机坏了, 程序没法往下跑); 进程切换
- pc跳转到哪里? 如何打断?
  - 跳到中断处理程序

### 中断机制

#### 为什么要有中断?

- 拥有机械部件的I/O设备相比于CPU来说实在太慢了 (在打印机慢悠悠地打印时CPU完全可以去做别的事情而不是干等)
- 需要引入中断来弥补I/O设备的速度缺陷

#### 如何响应中断?

- 给CPU加个引脚 (接一根中断信号线INT), 也相当于硬件驱动的函数调用, 即在每条语句后都插入

  ```c
  if (pending_io && int_enabled) {
  	interrupt_handler();
  	pending_io = 0;
  }
  ```
  
  ![[../0_Attachment/Pasted image 20241129141458.png]]

#### 中断的实现

比函数调用更复杂. 

由于是从进程A切换到进程B, 相当于从状态机A切换到状态机B, 为了保证切换回进程A时能正常运行 (即进程A感知不到中断发生), 需要保存与状态机A相关的所有信息, 称为"保存现场". 与之相对应的, 切换回进程A时也需要"恢复现场".

- 像"函数调用"一样保存PC到堆栈
- 自动保存RIP, CS, RFLAGS, RSP, SS
- 根据**中断/异常号**跳转到**处理程序 (handler)** 
  - (特权级切换会触发堆栈切换)
  - `int $0x80`指令可以产生128号异常
  - 时钟会产生32号中断, 键盘会产生33 号中断
- ![[../0_Attachment/Pasted image 20241129142936.png]]

中断是强制的, 普通的进程不能将其关闭.

`CLI` Clear Interrupt Flag, 将中断标记位置0;

普通程序用asm嵌入这条指令: 引发 Segmentation Fault. 因为使用这条指令的权限不匹配, 相当于越界访问了权限不匹配的内存空间引发错误.

也可以自行实现响应中断的函数 (库函数signal.h)

**中断处理程序S和正在执行的程序P是两个程序**

- S={<R, M>}
- P发生异常 -> 硬件保存 -> 跳转到中断处理程序 -> 软件保存

### 实现分时多任务

![[../0_Attachment/Pasted image 20241129144430.png]]

"保存现场"保存了什么? 通过调试QEMU中运行的"迷你"操作系统`thread-os.c`来演示.

![[../0_Attachment/Pasted image 20241129145100.png]]

中断 + 更大的内存 = 分时多线程

- 操作系统背负了"调度"的职责

分时多线程 + 虚拟存储 = 进程

- 操作系统需要完成进程, 存储, 文件的管理


还有呢?

- 面对有限的功耗, 难以改进的制程, 无法提升的频率, 应用需求
- 多处理器, big.LITTLE, 异构处理器 (GPU, NPU, …)
- 单指令多数据 (MMX, SSE, AVX, …), 虚拟化 (VT; EL0/1/2/3), … , 安全执行环节(TrustZone; SGX), …

### 操作系统

**中断驱动的上下文切换**

- 应用程序视角
  - 一组系统调用的API
    - 进程管理, 存储管理, 文件管理……
- 硬件视角
  - 操作系统是一个中断处理程序
    - 设备中断: 上下文切换, 调用设备驱动程序……
    - 系统调用: 操作系统代码执行 (API实现)
    - 异常: 发送信号 (类似系统调用)

## 12 程序的性能优化

### 算法

### 代码

### 指令

编译器优化的能力和局限性

预处理 c preprocessor

编译 c compile

汇编 assemble

链接 link

手写汇编不一定比高级语言更快

编译器不会修改程序的行为

#### 现代处理器

#### SIMD
