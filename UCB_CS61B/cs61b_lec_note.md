# UCB CS61B SP24

## Lecture Note

> 这里的笔记按照 (重要) 知识点划分而不是课程, 对不同部分的知识点作了一个简要的分块

### Java Basic

#### The Golden Rule of Equals (and Parameter Passing)

- 除了八种基本类型外, 赋值/传参都是复制的引用地址 (指针)

#### Nake Linked Lists vs. SLLists

- 使用SLLists隐藏递归的实现细节

#### Sentinel Node

- 哨兵节点: 一个一直存在的节点, 统一Lists內方法的实现 (不用为了特殊情况再写一份相近的代码, 从而实现代码的精简)
- DLList: 双向链表, 可以在首尾均放置一个sentinel, 也可以只使用一个使得链表形成环 (当迭代到sentinel时说明到达链表结尾)

#### Arrays vs. Classes

- Java允许在运行时评估数组索引
- 类成员变量名 (class member variable name) 在运行时不能被计算和使用 (computed and used), the only (easy) way to access a member of a class is with hard-coded dot notation
- 更多细节请参考操作系统课程 (or cs61c)

#### Interface vs. Implementation Inheritance

- Interface Inheritance (what): 接口继承, 列出需要实现的方法 (签名)
- Implementation Inheritance (how): 允许代码复用, 子类可使用父类/接口的方法(也可自行决定能否覆写) (Example: print() method in 61BList.java)

#### Constructor Behavior ls Slightly Weird

- Constructors are not inherited. However, the rules of Java say that all constructorsmust start with a call to one of the super class's constructors Link. 
- Java会在调用构造函数时自动**先**调用其 (即使没有写super()也会隐式调用**没有参数**的) 父类的构造函数

#### Compile-Time (Static) Types vs. Run-Time (Dynamic) Types

- **Compile-Time Type Checking**: 

  - Compiler allows method calls based on compile-time type of variable

  - Compiler also allows assignments based on complie-time types

  - ```java
    public static void main(String[] args){
    	VengefulSLList<Integer>vsl = new VengefulSLList<Integer>(9);
        SLList<Integer>sl=vsl;
        
        sl.addLast(50);
        sl.removeLast();
        
        sl.printLostItems(); //Compilation errors!
        VengefulSLList<Integer>vsl2 = sl; //Compilation errors!
    }
    ```

- **Expressions have compile-time types** 

  - ```java
    SLList<Integer>sl = new VengefulsLList<Integer>(9); //OK!
    VengefulsLList<Integer>vsl = new SLLlist<Integer>(9); //Compilation errors!
    ```

- **Casting**: 强制类型转换, 防止编译器报错 (有风险)

#### is-a vs. has-a

- Stack 栈
  - 只有pop和push操作
  - 如果is-a List, 则继承了List的所有方法如get, addfirst等不应该有的功能, ×
  - 如果has-a List, 则可以使用一个List(private)来存储数据, 只有pop (removeLast) 和push (addLast) 功能 √

#### Subtype Polymorphism

- **Polymorphism**: providing a single interface to entities (实体对象) of different types
- **Explicit Higher Order Functions vs. Subtype Polymorphism**:
  - 前者: 显式传递高阶函数 (e.g. Python) `def print_larger(x, y, compare, stringfy)`
  - 后者 "let the **object** do all the thinking for us"
- Example: 编写输出多个对象的max的函数, 多态的思想是提供一个接口Comparable让每个可比较对象继承compareTo方法, 再写一个统一的max函数, 而不是给每种对象编写一个比较其具体内容 (field) 的函数 (e.g. maxDog: dog.size, maxRuler: ruler.length)

#### Comparable Advantages

- Implementing Comparable allows library functions to compare custom types (e.g. finding max).

#### Comparator

- 比较器, 是一个类的从属功能 (静态嵌套类)
- a dog comparator is **not** a dog (也就说明可以把比较器写在类之外, 这在逻辑上没有区别)

#### For-each Iteration And ArraySets

- ArraySet **is** Iterable and **has a** Iterator

### Data Structure

#### Analysis of Algorithm Performance

- Asymptotic: 渐进分析, 当N足够大时, 时间增长速率与N增长速率呈什么关系
- Amortized: 摊销分析, 单个操作可能需要更长时间, 但如果进行多次操作, 可以拥有更好的平均性能
  - 可调整数组大小的ArrayList (之前的Lab实现), 只有在数组满了才翻倍resize, resize的时间与原数组大小呈线性, 则N次addLast操作消耗总时间为 (N (每次普通add操作的常数时间) + 1 + 2 + 4 + 8 ... +N (resize的总时间)), 即Θ(N), 摊销后addLast操作为Θ(1). 
  - WQU Disjoint sets with path compression
- Recursive: 递归分析 (e.g. 斐波那契数列)
- Remember: no magic shortcuts!

#### ADTs

- Abstract Data Type: 抽象数据类型, 可以理解为提供接口, 但可以有不同实现.
- 课上已实现的: List
- 需要去实现的: Set, Map (相关代码可参考Lab的实现)

#### BST

- Binary Search Tree 二叉搜索树
- 我们使用 Set/Map 时不需要考虑其中的顺序 (ADT规定), 但ADT接口的不同实现会影响性能/效率
- 类比二分查找: 搜索BST中数据时, 目标值等于当前值, 返回; 小于, 搜索左树; 大于, 搜索右树.
- 如果是随机生成的BST, 那么时间复杂度为$O(\log {n})$, 但BST在极端情况下时间复杂度会来到$O(n)$ (e.g.从1一直加到N)

#### B-Trees

- 对BST的优化
- 不允许新增叶, 但允许一个节点存放最多L个数据, 当数据多于L时, 节点向上以及旁边分裂, 根节点向上分裂时, 相当于全部节点向下移一层, 保证了叶的深度一致以及树的平衡, (相当于从下而上构建树), 保证了树的相关操作为$O(\log {n})$.
- Invariants 不变性: 
  - All leaves must be the same distance from the source.
  - A non-leaf node with 𝑘 items must have exactly 𝑘+1 children.
- 缺点: 在代码上比较难实现

#### Rotating Trees

- 前置: 树的旋转
- rotateLeft(G): Let x be the right child of G. Make G the new left child of x.
  rotateRight(G): Let x be the left child of G. Make G the new right child of x.
- 可以将旋转的过程视作先将两个结点合并, 再将原结点 "发射" 到另一侧的过程

#### RBT

- Red-Black Trees 红黑树
- 通过利用树的旋转, 使用 BST 的结构来模拟 B 树 (同构), 实现代码的简洁, 同时保证效率与性能
- 课堂上以及实验中使用的是 **LLRB** 左倾红黑树

#### Hashing

- 哈希表
- 用空间换时间, 将N个元素放进M个盒子里, M随N增长, 使得时间复杂度为$O(M/N)$, 即$O(1)$. (摊销分析)

#### PQ

- Priority Queue: 优先级队列, 一种抽象数据类型, 可以添加元素, 但只能访问/删除最小/最大的元素
- 可能的实现:
  - Ordered Array: `add`: Θ(𝑁) `getSmallest`: Θ(1) `removeSmallest`: Θ(𝑁)
  - Bushy BST: `add`: Θ(log𝑁) `getSmallest`: Θ(log𝑁) `removeSmalles`t: Θ(log𝑁), 但是不能有重复的元素
  - HashTable: `add`: Θ(1) `getSmallest`: Θ(𝑁) `removeSmallest`: Θ(𝑁)
- 更好的实现: **二叉堆**

#### Heap

- 堆: 一颗**完整的**二叉树, 每个结点的值**小于等于**其两个子节点的值, 所有结点尽量**左倾**
- 使用Heap实现PQ
  - `getSmallest`: Θ(1).
  - `add`: Add to the end of heap temporarily. Swim up the hierarchy to the proper place. 加到底, 往上浮 (交换). Θ(log𝑁).
  - `removeSmallest`: Swap the last item in the heap into the root. Sink down the hierarchy to the proper place. 将底部的替换顶部, 往下沉 (交换). Θ(log𝑁)
  - 补充: 相比BST, 支持重复项

####  Tree Traversal

- Level Order Traversal 层序: 逐层遍历
- Pre-order Traversal 前序: 先遍历自身, 再左树, 再右树
- In-order Traversal 中序: 先遍历左树, 再自身, 再右树
- Post-order Traserval 后序: 先遍历左树, 再右树, 再自身
- 补充: 这里的 "前, 中, 右" 是针对自身节点而言

#### Graph

- 图: A graph consists of:
  - A set of nodes (or vertices).
  - A set of zero of more edges, each of which connects two nodes.
- Basic Problems
- DFS: Depth First Search 深度优先搜索.
- BFS: Breadth First Search 广度优先搜索.

#### Shortest Paths

- 最短路径问题
- 对于非加权图, BFS在搜索完成后即等同于解决了最短路径问题
- 对于加权图
  - 原始人操作: 在权重为k的边上添加k-1个虚拟节点, 然后使用BFS, 但这会影响效率
  - **Dijkstra**:
    1. 创建一个 PQ, 其内容即为各个顶点, 对应的值时当前时刻每个顶点到源顶点的最短距离
    2. 将初始点的值设为0, 其他的设置为正无穷
    3. 当 PQ 非空时, 弹出其顶点, 并**relax**其所有边, 直到 PQ 为空. Relax: 检查当前顶点x的所有边e, 如果x的距离加上该边的权重小于e的另一个顶点y的距离, 则更新y. (弹出顶点的过程可以将顶点标记, 这意味这relax过程中可以不访问被标记过的顶点, 因为这些顶点已经找到了最短距离.)
    4. 局限性: 边的权重必须非负
  - **A***: D算法类似于同心圆一般的搜索, 但如果我只想知道两点间的最短路径呢? 比如广州到南京的最短路径, 我在D算法时实际上也搜索了广州到海南的最短路径, 但这是我不需要的, 这意味这即使海南离广州更近, 但我不想访问 (从PQ中弹出) 这一个顶点, 这需要给PQ的排列加一点方向性, 即根据一个已知的表来给PQ中每个点距离施加一点"惩罚", 例如在探索 "广州到南京的最短路径" 的表中我给每个点做一个 "惩罚" 预估计, 给海南的距离添加∞的惩罚, 这保证了我在弹出南京前不会先弹出海南 (弹出南京说明搜索结束), 提升了效率. 但如果这张表不够好, 则算法的正确性会受到影响, 比如之前的最短路径一定会包含武汉, 但是如果我们给武汉赋予较大的惩罚, 则A*算法得出的最短路径可能不经过武汉, 与正确答案偏离了.

#### MST

- Minimum Spanning Trees 最小生成树: 包含了图中所有结点的且总权重和最短的一棵树
- Cut 切割: 给定任何切割, 最小权重的横跨边在最小生成树中
- **Prim’s Algorithm**: 类似 Dijkstra 算法, 只是将到源顶点的距离换成到当前树的距离
  1. Start from some arbitrary start node.
  2. Repeatedly add the shortest edge that has one node inside the MST under construction.
  3. Repeat until there are V-1 edges.
- **Kruskal’s Algorithm**:
  1. Sort all the edges from lightest to heaviest.
  2. Taking one edge at a time (in sorted order), add it to our MST under construction if doing so does not introduce a cycle.
  3. Repeat until there are V-1 edges.

#### Topological Sorts

- 拓扑排序: 对图的顶点进行排序, 使得对于每一条有向边 u→v, u排序在v之前
- 算法实现: 对于所有入度为0的节点
  1. Perform a DFS traversal from every vertex in the graph, **not** clearing markings in between traversals.
  2. Record DFS postorder along the way.
  3. Topological ordering is the reverse of the postorder.

#### DAGs

- Directed Acyclic Graphs 有向无环图
- 拓扑排序的适用范围

#### Shortest Path Algorithm for DAGs

- 当DAGs中存在负权边时, Dijkstra算法可能失败, 如何规避?
- 失败原因: 在一个顶点a上relax它的边时, 如果存在负权边指向已经被标记的顶点b, 且b因为距离变小需要被更新, 此时D算法并不会更新, 因此失效;
- 解决: 先拓扑排序, 再按排序结果逐个访问节点并relax其边, 即保证了relax边时不会访问到 (也不可能) 已经被标记的节点 (所有边都指向一个方向).

#### Longest Paths

- 如何求无环最长路径? 指数时间复杂度 (无法应用, 等于无解)
- 如果是DAG呢?
  - Preprocess: 将所有边取负
  - 求最短路径
  - Postprocess: 将路径上的边取负

#### Reductions

- (归化?) 用 SPT-DAG 的方法来解决 LPT-DAG 问题, 便是一种 reduction 的策略 (DAG-LPT reduces to DAG-SPT.)

#### Tries

- 字典树 (检索数): 字符串(即由 ASCII 码构成)的检索数据结构, 在获取单词、添加单词和一些特殊字符串操作方面具有出色的性能. 比如寻找所有以sa开头的单词, 速度会比二叉树/哈希表快得多
- 如何表示字符串的结尾 (即树中含有这个字符串)? 
  - 将每个字符串的最后一个字符标记 (涂成蓝色).
- 时间复杂度: Θ(1).

### Algorithm

#### Selection Sort

- 选择排序: 依次寻找未排序的数组中的最小值放在已排序数组的最后
- 时间复杂度: $Θ(N^2)$ 空间复杂度: $Θ(1)$.

#### Heapsort

- 堆排序: 构造一个堆并依次取出顶部元素
- **Naive Heapsort**: 朴素堆排序
  1. Insert all items into a max heap, and discard input array. Create output array.
  2. Repeat N times:
     - Delete largest item from the max heap.
     - Put largest item at the end of the unused part of the output array.
  3. 时间复杂度: $Θ(N \log N)$ 空间复杂度: $Θ(N)$.
- **In-Place Heapsort**: 原地堆排序
  1. Rather than inserting into a new array of length N + 1, use a process known as “bottom-up heapification” to convert the array into a heap.
  2. Repeat N times: Delete largest item from the max heap, swapping root with last item in the heap.
  3. 时间复杂度: $Θ(N \log N)$ 空间复杂度: $Θ(1)$**. (Bad cache (61C) performance)

#### Mergesort

- 归并排序: 将数组一分为二, 分别进行归并排序 (递归), 再合并. (数组长度为1时不需排序只需合并)
- 时间复杂度: $Θ(N \log N)$ 空间复杂度: $Θ(N)$. (比堆排序快)

#### Insertion Sort

- 插入排序: 依次将未排序数组中的每个元素插入到已排序的数组中.
- 时间复杂度: $Θ(N^2)$
- **Naive Insertion Sort** 朴素插入排序
  - Starting with an empty **output** sequence.
  - Add each item from **input**, inserting into **output** at right point.
  - 空间复杂度: $Θ(N)$
- **In-Place Insertion Sort**
  - Do everything in place using swapping.
  - 空间复杂度: $Θ(1)$.
- Best for small N or almost sorted. 对于一些只有较少乱序元素的数组/长度较小的数组, 插入排序会很快
- Java: 归并排序时分割到较小部分就使用插入排序

#### Quciksort

- 快速排序: 任选一个基准值, 将小于基准值的数放在左边, 大于等于基准值的数放在右边, **分区**后该基准值位于正确的位置, 然后在分别对两边进行快速排序.
- 时间复杂度: 
  - best case: $Θ(N \log N)$
  - worst case: $Θ(N^2)$
  - Randomly chosen array case: $Θ(N \log N)$
- 空间复杂度: $Θ(\log N)$ (调用栈)
- Hoare Partitioning: 将空间复杂度降为 $Θ(1)$
  1. Create L and G pointers at left and right ends.
  2. L pointer is a friend to small items, and hates large or equal items. G pointer is a friend to large items, and hates small or equal items.
  3. Walk pointers towards each other, stopping on a hated item. When both pointers have stopped, swap and move pointers by one.
  4. When pointers cross, you are done.
- Quick Select: 用于寻找中位数基准值, 时间复杂度: $Θ(N)$
- 还有一些避免worst case的内容这里就不写了, 作用不大.

#### Comparison Sorts

- 以上均是
- 有什么不是comparison sort的呢?
  - Radix Sort, Sleepsort, Gravity Sort, BOGOsort

#### Theoretical Bounds on Sorting

- 适用于comparison sort
- 前提: $Θ(N \log N) = Θ(N!)$
- 排序的结果有$N!$种, 对于比较法这一**决策树**而言需要有$\log (N!)$层, 这决定了比较排序法时间复杂度的下限为$Θ(N \log N)$

#### Sorting Stability

- A sort is said to be stable if order of equivalent items is preserved.
- Insertion sort: stable 遇到相等就停了
- Mergesort: stable
- Quicksort: Depends on your partitioning strategy. 频繁交换: unstable, 遍历法: stable.
- Heapsort: unstable

#### Radix Sort

- 基数排序: 课上从 Digit-by-digit Sorting 和 Counting Sort 过渡过来. 
- **Counting Sort**
  - Alphabet keys only
  - 先遍历一遍数组, 给每种相同的数分一个区 (指定起始下标), 再遍历原数组按照起始下标填入新数组
  - 时间复杂度: $Θ(N + R)$
- **LSD Radix Sort**
  - Strings of alphabetical keys only
  - 从低位到高位进行遍历分组 (10组), 由于Counting Sort是稳定的, 遍历完成后排序完成
  - 时间复杂度: $Θ(WN + WR)$ (保证了R很小)
- **MSD Radix Sort**
  - 遍历最高位进行分组, 再在各分组**组内**将剩余位进行MSD Radix Sort.
  - 时间复杂度: 
    - best: $Θ(N + R)$ 
    - worst: $Θ(WN + WR)$ 

### Extension

#### Compression

- 理论依据: Shannon Entropy 香农熵
- 介绍了几种压缩文本的编码方法
  - Prefix Free Codes
  - Shannon Fano Codes
  - Huffman Coding
    - Build one corpus per input type.
    - For every possible input file, create a unique code just for that file. Send the code along with the compressed file. (更常用)

#### Turing Machine

- In general, we can find a reduction from a function problem (ex. What should I steal) to a decision problem (ex. Can you steal at least this much?)
- Formally, any decision problem in theoretical CS is said to run on a **Turing Machine**, which represents a model of **universal computation**; anything that can be solved by a computer program can also be solved by a Turing Machine.
  - Incidentally, when we talk about the time and space complexity of a Java program, the formal definition actually uses the equivalent Turing machine for precise definitions of "one unit of time" and "one unit of memory".
  - One major difference between a Java/Python program and a Turing machine is that the Turing machine has infinite memory.
  - A programming language is said to be **Turing-Complete** (图灵完备) if any Turing machine can be simulated with a program written in that language.

#### Deterministic Turing Machines

- A Turing machine with no randomness involved, or a **DTM**.
- Further, we will define the set **P** as the set of all decision problems that can be solved with a deterministic Turing Machine in **polynomial time**.
- Examples:
  - Is this array of length N sorted?
  - Does there exist a spanning tree of this graph (with N nodes) smaller than X weight?

#### Nondeterministic Turing Machines

- A **Nondeterministic Turing Machine** or **NTM** is a Turing Machine with one additional operation that can be performed: guessing.
  - Informally, we allow an operation int guess(int min, int max) that returns a number between min and max.
  - If ANY pattern of guesses ends up causing us to return True, then the nondeterministic Turing Machine returns True.
  - If ALL guess patterns return False, then the nondeterministic Turing Machine returns False.
- The set **NP** is the set of problems that can be solved in **polynomial time** with a NTM.
- 即猜一个答案, 该答案可在多项式时间內被验证正确与否. NTM允许**同时**验证所有答案. (开足够多个多元宇宙, 验证是否有一个宇宙里的答案是正确的)

#### P=NP?

- We will call a problem **NP-Hard** if every problem in NP reduces to that problem (or in other words, that problem is at least as hard as every problem in NP)
- We call a problem that's both in NP and NP-Hard an **NP-Complete problem**. Any NP-Complete problem is the hardest problem in NP.
- If even one of these problems has a reduction to a problem in P (or a polynomial time solution was discovered for it), then **every other NP problem will also be solved in polynomial time**. (注意solve和verify的区别)
- This is the **P=NP problem**: Find a Turing reduction from an NP problem to P (P = NP), or prove that no reduction exists (P != NP).

## Summary

>  这一部分照搬Lec40的slide

![](../0_Attachment/[61B%20SP24]%20Lecture%2040%20-%20Summary%20(0).jpg)

![](../0_Attachment/[61B%20SP24]%20Lecture%2040%20-%20Summary%20(1).jpg)

![](../0_Attachment/[61B%20SP24]%20Lecture%2040%20-%20Summary%20(2).jpg)

![](../0_Attachment/[61B%20SP24]%20Lecture%2040%20-%20Summary%20(3).jpg)

![](../0_Attachment/[61B%20SP24]%20Lecture%2040%20-%20Summary%20(4).jpg)

![](../0_Attachment/[61B%20SP24]%20Lecture%2040%20-%20Summary%20(5).jpg)

![](../0_Attachment/[61B%20SP24]%20Lecture%2040%20-%20Summary%20(6).jpg)