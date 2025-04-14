# 	Lecture Note

> NJU Spring 24  编译原理 蚂蚁老师

## 0 编译原理概述

### Why-为什么要学编译原理?

- fun, 复杂系统 成就感
- useful 语言类应用程序
  - 配置文件 (`.properties`), CSV文件, JSON文件
  - SQL引擎
- 难点: "只见树木, 不见森林", 需要搞清楚各阶段的联系.

### 课程安排

课程实验: 开发SysY语言编译器

工具: ANTLR

参考书: ANTLR4权威指南, 编程语言实现模式, 自制编译器, LLVM Cookbook

### 概述

把编译器看成黑盒

- 输入: Source Program
- 输出: Target Program
- ![](../0_Attachment/Pasted%20image%2020250225172619.png)

拆开第一层

- 前端 (分析阶段): 分析源语言程序, 收集所有必要的信息
- 后端 (综合阶段): 利用收集到的信息, 生成目标语言程序
- IR: Intermediate Representation, 中间表示, 架构无关, 抽象程度介于高级语言和汇编代码之间
  - 可以简化编译器的设计和开发的过程
  - 可以进行目标平台无关的中间代码优化, 整个过程有多个pass (趟)
    - 示例: 常量传播
  - 本课程使用LLVM使用的IR, `clang -emit-llvm -S`
- ![](../0_Attachment/Pasted%20image%2020250225172805.png)

拆开第二层

- 前端
  - 词法分析: 把源源不断的由单个字符构成的字符串分割成有意义的词法单元 (token)
  - 语法分析: 构建AST (abstract syntax tree, 抽象语法树)
  - 语义分析
  - ![](../0_Attachment/Pasted%20image%2020250225172924.png)
- 后端
  - ![](../0_Attachment/Pasted%20image%2020250225172929.png)

## 1 词法分析-词法分析器生成器

### 词法分析器

本质上就是一个程序

- 输入: 程序文本/字符串s (CharStream) + 词法单元 (token) 的规约
- 输出: 词法单元流 (TokenStream)
- ![](../0_Attachment/Pasted%20image%2020250225171355.png)

词法分析器的三种设计方法 (从易到难)

1. 词法分析器生成器
2. 手写词法分析器
3. 自动化词法分析器

### ANTLR v4

一种词法分析器生成器

- 参考教材 5.5, 15.5, 15.6, 12

- 输入: 词法单元的规约 `SimpleExpr.g4`

- 输出: 词法分析器 `SimpleExprLexer.java`

使用方式

- 命令行式使用 (过时)
- 交互式使用 (主要)
- 编程式使用 (左边是`.g4`, 右边是`.java`)
  - ![](../0_Attachment/Pasted%20image%2020250225172331.png)

冲突解决规则

- 最前优先匹配
  - 关键字 vs. 标识符: `print a`中`print`没有被识别为`ID`, 是因为ANTLR4自动写了`print`的语法规则
  - 多行注释 vs. 文档注释
- 最长优先匹配规则
  - `1.23`被匹配为`FLOAT`而不是`INT`
  - `>=`
  - `ifhappy`
- 非贪婪匹配: `SL_COMMENT : '//' .*? '\n' -> skip ;`, 会优先匹配最近的换行符, 如果不加`?`则是贪婪匹配
  - `?`在正则中也表示匹配出现0次或1次
  - 规避非贪婪匹配的写法: `SL_COMMENT2 : '//' ~'\n'* '\n' -> skip ;`, 中间表示除了换行符的任意字符 (`~`优先级更高)

fragment: 辅助规则 (区别于词法单元规则)

```
// 辅助规则
fragment LETTER : [a-zA-Z] ;
fragment NUMBER : [0-9] ;
fragment WORD : '_' | LETTER | NUMBER ;
// 词法单元规则
INT : '0' | ('+' | '-')? [1-9]NUMBER* ;
```

**通常**把对语法结构的描述放前面, 把对词法结构的描述放后面 (非规定)

## 2 词法分析-手写词法分析器

难点: 如何区分`INT`, `REAL`, `SCI`, 	

```
INT : DIGITS ;
REAL : DIGITS ('.' DIGITS)? ;
SCI : DIGITS ('.' DIGITS)? ([eE] [+-] DIGITS)? ;
```

解决: 用代码模拟状态转移图, 在`REAL`与`SCI`中, 有时需要回退 (即从状态14, 16, 17回退), 寻找最长合法匹配

![](../0_Attachment/Pasted%20image%2020250303145625.png)

*向前看, 向前走, 调整状态. 记录来时最长匹配, 无路可走便回头.*

## 3 词法分析-正则表达式与自动机理论

ATTLR4输入`.g4`, 输出`.java`的中间过程: 把正则表达式转成词法分析器

![](../0_Attachment/Pasted%20image%2020250307151423.png)

### 语言

语言是**字符串**构成的集合, C语言是所有符合C语言规则的字符串构成的集合.

字母表 $\Sigma$: 一个有限的符号集合

串: 字母表 $\Sigma$ 上的串 (s) 是由 $\Sigma$ 中符号构成的一个有穷序列

- 空串: $|\epsilon| = 0$
- 运算: 连接运算, 指数运算

语言: 给定字母表 $\Sigma$ 上一个任意的可数的串集合, 因此可以通过集合操作构造新的语言

- 运算: ![](../0_Attachment/Pasted%20image%2020250307151925.png)

### 正则表达式

每个正则表达式 $r$ 对应一个正则语言 $L(r)$, 

- 正则表达式是**语法**: `ID: [a-zA-Z][a-zA-Z0-9]*`
- 正则语言是**语义**: `{a1, a2, ab, . . .}`
- *定义见PPT*

### NFA和DFA

*定义见PPT*

注意 NFA 和 DFA 区别

自动机 $\mathcal{A}$ 定义了一种语言 $L(\mathcal{A})$, 即它能接受的所有字符串构成的集合, 即自动机与RE等价

NFA 简洁易于理解, 便于描述语言 $L(\mathcal{A})$, DFA 易于判断 $x \in L(\mathcal{A})$, 适合产生词法分析器, 因此可用 NFA 描述语言, 用 DFA 实现词法分析器, 即实现 RE -> NFA -> DFA -> 词法分析器

### Thompson构造法

RE -> NFA

根据正则表达式的定义递归构造, 且假设所有自动机的开始状态和接收状态均唯一, *详见PPT*.

### 子集构造法

NFA -> DFA, 思想: 用DFA模拟NFA

*详见PPT*

### DFA最小化

基本思想: 等价的状态可以合并, 不等价的状态则需要划分

选择**划分**而非合并, 因为缺少基础情况, 不知从何开始合并. 而接受状态与非接受状态必定不等价, 因为空串 $\epsilon$ 区分了这两类状态

### DFA -> 词法分析器

需满足最前优先匹配和最长优先匹配

需要消除死状态, 避免词法分析器徒劳消耗输入流

处理情况: 接受/回溯/报错, 无论成功还是失败, 都从初始状态开始继续识别下一个词法单元.

### Kleene构造法

DFA -> RE

*没讲, 自己看PPT*

## 4  语法分析-ANTLR 4 语法分析器

二义性 (ambiguous) 文法

- `if-then-else`嵌套带来的二义性
  - ANTLR 4 默认会将`else`匹配离它最近的`if`
- 运算符的**结合性**带来的二义性
  - ANTLR 4 默认左结合
  - 右结合运算符 (比如`^`): 加上前缀`<assoc = right>`
- 运算符的**优先级**带来的二义性
  - 最前优先匹配

ParseTreeWalker

- 负责以 DFS 方式自动遍历语法树

Listener

- 负责监听**进入**, **退出**节点事件: `enterXXX()`, `exitXXX()`
- 提供一个`xxxListener.java`的接口以供实现, 但需要**实现**所有方法
- 也提供一个`xxxBaseListener.java`包含所有**空实现**, 只需自己写一个`myListener.java`去继承, 覆写所需方法即可
- 缺点: 所有方法返回值为`void`, 不利于某些实现 (比如利用语法分析树进行表达式求值), 需要 Visiter

Visiter

- 行为与 Listener 大致, 但可以自定`visitXXX()`方法的**返回值**, 也可以决定是否进行`visit`行为

## 5 语法分析-CFG

### CFG

Context-Free Grammar, CFG, 上下文无关文法

![](../0_Attachment/Pasted%20image%2020250321124501.png)

*Context-Sensitive Grammar, CSG, 上下文相关文法*

语义: 上下文无关文法 $G$ 定义了一个语言 $L(G)$

- 语言是串的集合, 串从哪来? **推导** (Derivation): 将某个产生式的左边替换成它的右边, 每一步推导需要选择替换哪个非终结符号, 以及使用哪个产生式

![](../0_Attachment/Pasted%20image%2020250321125440.png)

![](../0_Attachment/Pasted%20image%2020250321125446.png)

![](../0_Attachment/Pasted%20image%2020250321125454.png)

### 文法

检查字符串 $x$ 是否符合文法 $G$

- 语法分析器的任务: 为输入的词法单元流寻找推导, 构建语法分析树, 或者报错.

$L(G)$ 是什么

- 这是程序设计语言设计者需要考虑的问题

### 证明

![](../0_Attachment/Pasted%20image%2020250321125140.png)

正则表达式的表达能力**严格弱于**上下午无关文法

1. 每个正则表达式 $r$ 对应的语言 $L(r)$ 都可以使用上下文无关文法来描述
2. 反证法证明: $L = \{ a^n b^n | n \ge 0 \}$ 无法使用正则表达式描述
   - 相关定理: Pumping Lemma for Regular Languages

$L = \{ a^n b^n c^n | n \ge 0 \}$ 也无法使用上下文无关文法描述

## 6 语法分析-递归下降的 LL(1) 语法分析器

### 递归下降的 LL(1)

无二义性的文法 = 每个句子对应唯一的一棵语法分析树

![](../0_Attachment/Pasted%20image%2020250325123436.png)

自顶向下构建语法分析树

- 根节点: 文法的起始符号 S
- 中间节点: 对**某个非终结符**应用某个产生式进行推导
- 叶节点: 是词法单元流 w$ 
  - 仅包含终结符号与特殊的文件结束符 $ (EOF)

本节要解决的问题: 选择哪个非终结符? 选择哪个产生式?

$LL(1)$

- 第一个 $L$: 从左向右读入词法单元
- 第二个 $L$: 在推导的每一步总是选择**最左边**的非终结符进行展开

递归下降

- 为每个非终结符写一个递归函数
- 内部按需调用其它非终结符对应的递归函数, 下降一层

### 预测分析表

在递归下降的演示过程中, 同样是展开非终结符 $S$, 为什么前两次选择了规则2, 第三次选择了规则1?

- 因为面对的**当前词法单元**不同
- 需要一个**预测分析表**确定产生式的选择

预测分析表

- 指明了每个非终结符在面对不同的词法单元或文件结束符时, 该选择哪个产生式 (按编号进行索引) 或者报错 (空单元格)
- 如何构建预测分析表?

![](../0_Attachment/Pasted%20image%2020250325124557.png)

![](../0_Attachment/Pasted%20image%2020250325124725.png)

![](../0_Attachment/Pasted%20image%2020250325124735.png)

![](../0_Attachment/Pasted%20image%2020250325124848.png)

![](../0_Attachment/Pasted%20image%2020250325124900.png)

![](../0_Attachment/Pasted%20image%2020250325125023.png)

满足1, 2其中之一即可, 第2条规则等价为 $\epsilon \in FIRST(\alpha) \ \and \ t \in FOLLOW(A)$

LL(1) 语法分析器

- $L$: 从左向右 (left-to-right) 扫描输入
- $L$: 构建最左 (leftmost) 推导
- 1: 只需向前看一个输入符号便可确定使用哪条产生式

### 非递归的预测分析算法

栈 + 预测分析表

![](../0_Attachment/Pasted%20image%2020250325125545.png)

## 7 语法分析-Adaptive LL(∗) 语法分析算法

### 背景

- LL(1) 语法分析算法的处理能力有限
  - 左递归文法, 带左公因子的文法: `E -> E + E | E * E | ID`
- ANTLR 4 采用的 Adaptive LL(∗) 语法分析算法功能强大
  - 自动将类似 expr 的左递归规则重写成非左递归形式
  - 提供优秀的错误报告功能和复杂的错误恢复机制
  - 几乎能处理任何文法 (不能处理间接左递归)

### 处理直接左递归与优先级

例子: 匹配 `1 + 2 + 3`, `1 + 2 * 3`, `1 * 2 + 3`

```
stmt : expr ';' EOF;
expr : expr '*' expr
     | expr '+' expr
     | INT
     | ID
     ;
```

根本问题:究竟是在 expr 的**当前调用**中匹配下一个运算符, 还是让 expr 的**调用者**匹配下一个运算符.

改写: 

```
expr[int _p]
	: (	  INT
		| ID
	  )
	  (	  {4 >= $_p} ? '*' expr[5]
	  	| {3 >= $_p} ? '+' expr[4]
	  )*
	;
```

![](../0_Attachment/Pasted%20image%2020250331152216.png)

- 左结合: p + 1
- 右结合: p 不变

### 动态分析: Adaptive LL(∗)

例子: `P = {S -> Ac | Ad, A -> aA | b}`, 匹配 `bc` 和 `bd`

![](../0_Attachment/Pasted%20image%2020250331152658.png)

- Launch subparsers at a decision point, one per alternative productions.
- These subparsers run in pseudo-parallel to explore all possible paths.
- Subparsers die off as their paths fail to match the remaining input.
- Ambiguity: Multiple subparsers coalesce together or reach EOF.
- Resolution: The first production associated with a surviving subparser.

Move on terminals and Closure over ϵ and non-terminals.

## 8 语义分析-符号表

### 类型检查

分类: Strong/Weak, Dynamic/Static

在 Lab 中需要考虑

### 符号检查

符号: 变量名, 函数名, 类型名, 标签名…

符号表是用于保存各种符号相关信息的**数据结构**

作用域

- 领域特定语言 (DSL) 通常只有单作用域 (全局作用域)
- 通用程序设计语言 (GPL) 通常需要嵌套作用域

通过 ANTLR 4 的 listener 构建一个符号表

- 类设计:
  ![](../0_Attachment/Pasted%20image%2020250331174859.png)
- Listener:
  ![](../0_Attachment/Pasted%20image%2020250331174904.png)

struct/class 的类型作用域

![](../0_Attachment/Pasted%20image%2020250331174956.png)

## 9 语义分析-属性文法

*笔记思路不算特别清晰, 主打一个能看就行*

### 属性文法

进行类型检查, 符号检查, 用什么样的文法刻画语言的**语义**.

**属性文法** (Attribute Grammar): 为上下文无关文法赋予语义

**Offline** 方式计算属性值: 已有语法分析树

- 按照从左到右的深度优先顺序遍历语法分析树
- 关键: 在合适的时机执行合适的动作, 计算相应的属性值

在语法**分析过程中**实现**属性文法**

- 语义动作嵌入的位置决定了何时执行该动作
- 基本思想: 一个动作在它左边的所有文法符号都处理过之后立刻执行

语法制导定义 (Syntax-Directed Definition, SDD)

- SDD 是一个上下文无关文法和**属性**及**规则**的结合
- 每个文法符号都可以关联多个属性 (红框), 每个产生式都可以关联一组规则 (蓝框)
- SDD 唯一确定了语法分析树上每个**非终结符**节点的属性值, 但没有规定以什么方式, 什么顺序计算这些属性值
- ![](../0_Attachment/Pasted%20image%2020250406152049.png)

注释 (annotated) 语法分析树: 显示了各个属性值的语法分析树

- ![](../0_Attachment/Pasted%20image%2020250406152325.png)

**综合属性** (Synthesized Attribute)

- 节点 N 上的综合属性只能通过 N 的子节点或 N 本身的属性来定义
- 示例如上图

S 属性定义 (S-Attributed Definition)

- 如果一个 SDD 的每个属性都是综合属性, 则它是 S 属性定义
- 其依赖图描述了属性实例之间**自底向上**的信息流, 此类属性值的计算可以在自顶向下的 LL 语法分析过程中实现
  - 在 LL 语法分析器中, 递归下降函数 A **返回**时, 计算相应节点 A 的综合属性值

**继承属性** (Inherited Attribute)

- 节点 N 上的继承属性只能通过N 的父节点, N 本身和 N 的兄弟节点上的属性来定义
- 信息流向: 先从左向右, 从上到下传递信息
- ![](../0_Attachment/Pasted%20image%2020250406153303.png)
- ![](../0_Attachment/Pasted%20image%2020250406153306.png)

L 属性定义 (L-Attributed Definition)

- ![](../0_Attachment/Pasted%20image%2020250406153353.png)

### 示例: 后缀表达式

S属性定义

![](../0_Attachment/Pasted%20image%2020250406153452.png)

![](../0_Attachment/Pasted%20image%2020250406153511.png)

### 示例: 数组类型文法

![](../0_Attachment/Pasted%20image%2020250406153637.png)

继承属性 C.b 将一个基本类型沿着树向下传播, 综合属性 C.t 收集最终得到的类型表达式

- ![](../0_Attachment/Pasted%20image%2020250406153705.png)
- ![](../0_Attachment/Pasted%20image%2020250406153811.png)

语法制导的翻译方案 (Syntax-Directed Translation Scheme, SDT)

- SDT 是在其产生式体中嵌入语义动作的上下文无关文法

## 10 中间代码生成-LLVM IR简介

### 概念

LLVM IR: 带类型的、介于高级程序设计语言与汇编语言之间

- 与高级语言的关系
- ![](../0_Attachment/Pasted%20image%2020250407101604.png)

生成 LLVM IR

- `clang -S -emit-llvm -fno-discard-value-names factorial0.c -o f0-opt0.ll` (不开优化)
- ![](../0_Attachment/Pasted%20image%2020250407101952.png)
- `clang -S -emit-llvm factorial0.c -o f0-opt1.ll -O1 -g0` (O1优化, mem2reg)
- ![](../0_Attachment/Pasted%20image%2020250407102223.png)

三地址代码 Three Address Code (TAC)

- 操作数最多3个

静态单赋值 Static Single Assignment (SSA)

- 可用无限多个**虚拟**寄存器
- 每个寄存器只能被赋值**一次**, 即赋值语句中每个寄存器只能在等号左边出现一次

(函数内) 控制流图 (Intra-procedure) Control Flow Graph (CFG)

- 解释了 Basic Block, 每个**基本块**只有第一条指令是入口 (其他块跳转的目标), 只有最后一条语句是出口 (跳转至其他块)
- ![](../0_Attachment/Pasted%20image%2020250407102418.png)
- 为什么基本块的中间某条指令可以是 call 指令? 因为函数调用完成后会返回到该基本块 (把函数调用视为黑盒)
- ![](../0_Attachment/Pasted%20image%2020250407102812.png)

$\phi$ 指令

- 根据控制流决定选择哪个值
- ![](../0_Attachment/Pasted%20image%2020250407102921.png)
- ![](../0_Attachment/Pasted%20image%2020250407103016.png)

### 更多教程

[LLVM IR Animation](https://blog.piovezan.ca/compilers/llvm_ir_animation/llvm_ir.html)

[LLVM IR Tutorial @ Bilibili](https://www.bilibili.com/video/BV1mE421g7BA)

[LLVM Language Reference Manual](https://llvm.org/docs/LangRef.html)

[Program Visualization using LLVM @ Bilibili](https://www.bilibili.com/video/BV1jT421C7cH)

### 用编程的方式生成 LLVM IR

[LLVM Programmer's Manual](https://llvm.org/docs/ProgrammersManual.html)

## 11 中间代码生成-表达式的翻译

### 表达式的中间代码翻译

例子: `a = b + -c`

- 产生式与规则: ![](../0_Attachment/Pasted%20image%2020250407143541.png)
- 综合属性 E.code: 中间代码
- 综合属性 E.addr: 变量名 (包括临时变量)、常量
- 这里的 `||` 指将子节点生成的中间代码拼接成**多行代码**, 利用综合属性从下到上的信息流
- 结果 (简化版): ![](../0_Attachment/Pasted%20image%2020250407143547.png)
- LLVM 结果: ![](../0_Attachment/Pasted%20image%2020250407143642.png)

### 数组引用的中间代码翻译

例子

- 声明: `int a[2][3];`
- 数组引用: `x = a[1][2]; a[1][2] = x;`

关键: 需要计算 `a[1][2]` 相对于数组基地址 `a` 的偏移地址

产生式与规则

- ![](../0_Attachment/Pasted%20image%2020250407144259.png)
- 综合属性 L.array(.base): 数组基地址 (即, 数组名)
- 综合属性 L.addr: 偏移地址
- 注释语法分析树: ![](../0_Attachment/Pasted%20image%2020250407144357.png)

LLVM IR

- 数组声明: `int a[2][3] = {0};` => `%2 = alloca [2 x [3 x i32]], align 16`
- 数组引用: ![](../0_Attachment/Pasted%20image%2020250407144937.png)
  - 蓝框: 访问的数组类型
  - 红框: 数组的起始地址
  - 紫框: 根据**蓝框自身类型**进行索引偏移
  - 黄框: 根据**蓝框元素类型**进行索引偏移
  - 区分紫框黄框: 传入一个`int32 *[2][3]` (无法区分只有一个二维数组, 还是多个二维数组的数组, 即三维数组)
    - 紫: 单位 24
    - 黄: 单位 12
- 黄框索引可以有多个: ![](../0_Attachment/Pasted%20image%2020250407145647.png)

## 12 中间代码生成-控制流语句的翻译

### Easy 模式

优点

- 各个非终结符的**分工, 合作**明确
- 只需要使用**综合属性**

方式

- 为布尔表达式 B 计算逻辑值, 该值最终保存在临时变量 `t1` 中
- `if`, `while` 等语句根据 B (即 `t1` 的值) 的结果**改变控制流**
- 例子: `S -> if (B) S1`, 如何分工
  - B: 计算布尔表达式值, 返回临时变量 `t1`
  - S1: 指令执行
  - S: 添加分支指令 `br`, 该指令能够根据返回的 `t1` 改变控制流

实操

- ANTLR 4 Visitor 模式, 可以及时输出生成的中间代码, 避免频繁的字符串拼接操作 (开销较大)

翻译 `break;`

- 使用**栈**结构去追踪循环体执行结束后需要跳转到的标签
- 访问 while 循环体**前**将标签 `while.false` 压栈, 访问结束后将其出栈

短路求值 (Short-Circuit Evaluation)

- 生成计算 B1 的代码后添加一条分支指令 `br`, 该指令能够根据 B1 值决定是否跳过 B2 的计算. 接着生成计算 B2 的代码. (两段计算 bool 的指令都是需要**生成**的, 只不过是在执行指令时可以根据计算结果决定是否进行计算)
- ![](../0_Attachment/Pasted%20image%2020250410120103.png)

### Hard 模式

优点

- 能够生成**更短, 更高效**的代码
- 直接用**布尔表达式**改变控制流, 无需计算最终逻辑值
- 从父节点获取**更具体的跳转目标**, 缩短跳转路径

方式

- 父节点为子节点准备跳转指令的目标标签
- 子节点通过**继承属性**确定跳转目标

继承属性 B.true, B.false, S.next 指明了控制流跳转目标 (例子可以去看 PPT)

- ![](../0_Attachment/Pasted%20image%2020250410120518.png)
- ![](../0_Attachment/Pasted%20image%2020250410120541.png)

完整的产生式与语义规则: ![](../0_Attachment/Pasted%20image%2020250410120659.png)

### 地址回填技术

Java Bytecode 使用

特点

- 不使用标签, 而是使用**地址值**作为跳转目标
- 在**一趟**扫描中生成跳转目标的地址

方式

- 子节点暂时不指定跳转指令的目标地址, 选择**留白**, 等到父节点能够确定目标地址时**回头填充**
- 回填 (Backpatching) 技术: 子节点挖坑, 父节点填坑
- 父节点通过**综合属性**收集子节点中具有相同目标的跳转指令
  - 综合属性 B.truelist 保存需要跳转到 B.true 标签的指令, 综合属性 B.falselist 保存需要跳转到 B.false 标签的指令, 综合属性S/L.nextlist 保存需要跳转到 S/L.next 标签的指令
  - 为左部非终结符 B 计算综合属性 B.truelist 与 B.falselist, 为左部非终结符 S/L 计算综合属性 S/L.nextlist, 并为已能确定目标地址的跳转指令进行**回填** (考虑每个综合属性)
- 该方式的产生式和语义规则与 Hard 模式**等价** (具体对照可参考 PPT)

![](../0_Attachment/Pasted%20image%2020250414113316.png)

![](../0_Attachment/Pasted%20image%2020250414113412.png)
