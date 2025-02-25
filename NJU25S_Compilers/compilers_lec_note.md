# Lecture Note

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

