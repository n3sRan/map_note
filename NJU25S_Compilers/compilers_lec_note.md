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
