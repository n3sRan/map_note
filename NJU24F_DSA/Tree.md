# Tree

## 二叉树

### 定义与性质

- $n$个结点的二叉树之间有 $n-1$ 条边
- 第 $i$ 层的节点数最多是 $2^i$ 个
- 高度为 $h$ (从0开始计) 的二叉树中结点最少 $h+1$ 个, 最多 $2^{h+1} - 1$ 个
- 如果一颗二叉树有$n_0$个树叶, 并且结点度数为2的节点有$n_2$个, 则$n_0 = n_2 + 1$
  - 设总节点数为 $n$, 度数为1的结点数为 $n_1$, 分支数为 $B$, 有 $n = B + 1$, $B = 1 * n _1 + 2 * n_2$
- 有$n$个结点的二叉树的高度最大为 $n-1$, 最小为$\log_2(n+1) - 1$ (向上取整)

#### 满二叉树

- 将二叉树排满

#### 完全二叉树

- 完全二叉树的最后一层可以不全满, 但是必须从左开始顺序无空缺
- 节点从0开始计数, 第$i$个节点的父节点是$(i - 1) / 2$, 左子节点为$2i + 1$, 右子节点为$2i + 2$

### 代码实现

#### 数组实现

- 很稀疏的二叉树会导致数组存储二叉树有大量的内存空间被浪费掉

#### 链表实现

```java
// 二叉树节点
class BinaryNode<T> {
    T element;
    BinaryNode<T> left, right;

    BinaryNode(T element, BinaryNode<T> left, BinaryNode<T> right) {
        this.element = element;
        this.left = left;
        this.right = right;
    }
}

// 二叉树
class BinaryTree<T> {
    BinaryNode<T> root;

    public void makeTree(T data, BinaryTree<T> leftch, BinaryTree<T> rightch) {
        root = new BinaryNode<>(data, leftch.root, rightch.root);
        leftch.root = null;
        rightch.root = null;
    }

    public void breakTree(T[] data, BinaryTree<T> leftch, BinaryTree<T> rightch) {
        if (root == null) throw new IllegalArgumentException("Tree is empty");
        data[0] = root.element;
        leftch.root = root.left;
        rightch.root = root.right;
        root = null;
    }
    
    // other method
}
```



### 二叉树遍历

#### 先序遍历

非递归实现先序遍历

```java
void preOrderIterative(TreeNode root) {
        if (root == null) return;  // 空树

        Stack<TreeNode> nodeStack = new Stack<>();  // 创建一个栈保存节点
        nodeStack.push(root);         // 根节点进栈

        while (!nodeStack.isEmpty()) {  // 栈非空时迭代处理
            TreeNode node = nodeStack.peek();  // 保存栈顶节点
            System.out.print(node.data + " ");  // 访问节点数据
            nodeStack.pop();                   // 栈顶节点出栈

            // 子节点入栈
            if (node.right != null) nodeStack.push(node.right);
            if (node.left != null) nodeStack.push(node.left);
        }
    }
```

#### 中序遍历

非递归实现中序遍历

```java
void inOrderIterative(TreeNode root) {
        if (root == null) return;  // 空树

        Stack<TreeNode> nodeStack = new Stack<>();   // 创建一个栈保存节点
        TreeNode currentNode = root;  // 维护一个当前节点指针

        // 当前节点非空, 或栈非空时迭代处理
        while (currentNode != null || !nodeStack.isEmpty()) {
            // 当前节点非空, 沿着左子树方向入栈
            while (currentNode != null) {
                nodeStack.push(currentNode);
                currentNode = currentNode.left;
            }
            // 此时栈顶节点没有左子树, 或已经访问完左子树
            currentNode = nodeStack.peek();     // 取栈顶节点
            System.out.print(currentNode.data + " ");  // 访问节点数据
            nodeStack.pop();                   // 出栈
            currentNode = currentNode.right;  // 将当前节点设为右子节点
        }
    }
```

#### 后序遍历

非递归实现后序遍历

```java
void postOrderIterative(TreeNode root) {
        if (root == null) return;  // 空树

        Stack<TreeNode> nodeStack = new Stack<>();   // 创建一个栈保存节点
        TreeNode currentNode = root;  // 维护一个当前节点指针
        TreeNode visitedNode = root;  // 保存上次一访问的节点, 初始化为root是利用二叉树是无环图

        // 当前节点非空, 或栈非空时迭代处理
        while (currentNode != null || !nodeStack.isEmpty()) {
            // 当前节点非空, 沿着左子树方向入栈
            while (currentNode != null) {
                nodeStack.push(currentNode);
                currentNode = currentNode.left;
            }
            currentNode = nodeStack.peek();  // 取栈顶元素
            // 如果栈顶元素有右子树, 且未被访问
            if (currentNode.right != null && currentNode.right != visitedNode) {
                currentNode = currentNode.right;
            } else {  // 子树为空或被访问过
                System.out.print(currentNode.data + " ");  // 访问节点数据
                visitedNode = currentNode;         // 记录当前访问的节点
                currentNode = null;                // 当前节点置为NULL, 防止重复访问左子树
                nodeStack.pop();                   // 出栈
            }
        }
    }
```

#### 广度优先遍历

如果是数组实现的二叉树, 直接按顺序访问数组即可.

```java
// 链表实现的二叉树
public void bfs(TreeNode root) {
    if (root == null) return;
        
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root); // 将根节点加入队列

    while (!queue.isEmpty()) {
        TreeNode current = queue.poll(); // 获取并移除队列头部   
        System.out.print(current.val + " "); // 访问当前节点
        // 将左子节点加入队列
        if (current.left != null) {
            queue.offer(current.left);
        }
        // 将右子节点加入队列
        if (current.right != null) {
            queue.offer(current.right);
        }
    }
}
```

### 建立二叉树

#### 构造函数

[`makeTree()`](<####链表实现>)

#### 广义表

不考

#### 后缀表示

![](../0_Attachment/Pasted%20image%2020241229154019.png)

#### 已知先序遍历和中序遍历生成二叉树

```java
BinaryNode<Type> createBTFromPreIn(String pres, String ins) {
    if (pres.length() == 0) return null;

    BinaryNode<Type> temp = new BinaryNode<>(pres.charAt(0));
    int inpos = 0;

    // 从中序遍历中找到根节点的位置, 这样子根节点左侧的是左子树, 右侧的是右子树
    while (ins.charAt(inpos) != temp.element) 
        inpos++;

    // 小括号是重载的, 将先序遍历字符串的1到inpos取出来, 赋给中间变量
    String prestemp = pres.substring(1, inpos + 1);
    String instemp = ins.substring(0, inpos);
    temp.left = createBTFromPreIn(prestemp, instemp);

    prestemp = pres.substring(inpos + 1);
    instemp = ins.substring(inpos + 1);
    temp.right = createBTFromPreIn(prestemp, instemp);

    return temp; // 完成组装返回
}
```

#### 已知后序遍历和中序遍历生成二叉树

```java
BinaryNode<Type> createBTFromPostIn(String post, String in) {
        if (post.length() == 0) return null;

        // 根节点是后序遍历的最后一个元素
        BinaryNode<Type> temp = new BinaryNode<>(post.charAt(post.length() - 1));
        int inpos = 0;

        // 从中序遍历中找到根节点的位置, 这样子根节点左侧的是左子树, 右侧的是右子树
        while (in.charAt(inpos) != temp.element) 
            inpos++;

        // 小括号是重载的, 将后序遍历字符串的0到inpos-1取出来, 赋给中间变量
        String postLeft = post.substring(0, inpos);
        String inLeft = in.substring(0, inpos);
        temp.left = createBTFromPostIn(postLeft, inLeft);

        // 将后序遍历字符串的inpos到post.length()-2取出来, 赋给中间变量
        String postRight = post.substring(inpos, post.length() - 1);
        String inRight = in.substring(inpos + 1);
        temp.right = createBTFromPostIn(postRight, inRight);

        return temp; // 完成组装返回
    }
```

### 二叉树的应用

#### 树的表示方式: 左子女右兄弟

- 将树转成二叉树
- 将森林转成二叉树
  1. 每棵树转成二叉树
  2. 每棵树之间用右链相连

#### 树的遍历

- 深度优先遍历 (DFS)
  - 先序次序遍历: 先访问树的根
    - 同构成二叉树后对二叉树进行先序遍历
  - 后序次序遍历: 最后访问树的根
    - 同构成二叉树后对二叉树进行后序遍历
- 广度优先遍历

#### 森林的遍历

- 先根, 中根, 后根
  - 同构成兄弟树后进行先序, 中序, 后序遍历

## 线索二叉树

1. 目的: 让二叉树遍历的速度更快
2. 特点: 在树的节点中加入一个指针 (比如指向下一个节点)
3. n个结点的二叉树有2n个链域, 其中真正有用的是 n–1个, 其它 n+1 个都是空域 (null). 为了充分利用结点中的空域, 使得对某些运算更快, 如前驱或后继等运算.

以中序线索二叉树为例.

![](../0_Attachment/Pasted%20image%2020241227142752.png)

![](../0_Attachment/Pasted%20image%2020241227142759.png)

### 遍历中序线索二叉树

- 既不要递归, 也不要栈
- 算法:
  - 找到中序下的第一个结点 (First)
  - 不断找后继 (Next)

```java
// current 为全局变量
public ThreadNode<T> first() {
    while (!current.leftthread) {
        current = current.leftchild;
    }
    return current; // 找中序遍历的第一个节点
}

public ThreadNode<T> next() {
    ThreadNode<T> p = current.rightchild; // 可能是右子树的根节点, 也可能是右链
    if (!current.rightthread) {
        // 如果有右子树就要搜索到最左下部分
        while (!p.leftthread) {
            p = p.leftchild;
        }
    }
    current = p;
    return current;
}

public void inorder() {
    for (ThreadNode<T> p = first(); p != null; p = next()) {
        System.out.println(p.element);
    }
}
```

### 构建中序线索二叉树

![](../0_Attachment/Pasted%20image%2020241227143550.png)

```java
void inThread(ThreadNode<T> T) {
    Stack<ThreadNode<T>> s = new Stack<>();
    ThreadNode<T> p = T;
    ThreadNode<T> pre = null;

    for (;;) {
        // 查找到最左下部分的
        while (p != null) {
            s.push(p);
            p = p.leftchild;
        }
        // 开始弹出栈
        if (!s.isEmpty()) {
            p = s.pop();
            if (pre != null) {
                // 添加的代码, 在这时候处理pre
                if (pre.rightchild == null) {
                    pre.rightchild = p;
                    pre.rightthread = true;
                }
                // 处理p
                if (p.leftchild == null) {
                    p.leftchild = pre;
                    p.leftthread = true;
                }
            }
            pre = p;
            p = p.rightchild;
        } else {
            return;
        }
    }
}
```

## 哈夫曼树

![](../0_Attachment/Pasted%20image%2020241227153957.png)

- 增长树(使得原来的树的每一个度数都为2)
  - 对原二叉树中度为1的结点, 增加一个空树叶
  - 对原二叉树中的树叶, 增加两个空树叶
- 外通路长度 (外路径) E 根到每个外结点 (增长树的叶子) 的路径长度的总和(边数)
  - E = 3+3+2+3+4+4+3+3=25, 如上图例
- 内通路长度 (内路径) I: 根到每个内结点(非叶子)的路径长度的总和(边数). 原来的树上每一个节点到树根的长度的综合
  - I=2+1+0+3+2+2+1=11 如上图例
- 结点的带权路径长度: 一个结点的权值与结点的路径长度的乘积.
  - 每个结点的权重占有一定的值
- 带权的外路径长度: 各叶结点的带权路径长度之和.
- 带权的内路径长度: 各非叶结点的带权路径长度之和.
- 算法: 如何构造一棵哈夫曼树以及其带权路径长度 (WPL)
  - [视频演示](<https://www.bilibili.com/video/BV1kD4y1P76D>)
- 哈夫曼编码
  - 根据字符出现的频率编码, 使得电文总长度最短
  - 编码唯一, 即一个叶节点对应一种表示

## 广义表

不考

## 二叉搜索树

代码实现

```java
// 插入
private BinaryNode<T> insert(T x, BinaryNode<T> t) {
    if (t == null)
        return new BinaryNode<>(x);
    if (x.compareTo(t.element) < 0)
        t.left = insert(x, t.left);
    else if (x.compareTo(t.element) > 0)
        t.right = insert(x, t.right);
    return t;
}

// 搜索
private BinaryNode<T> find(T x, BinaryNode<T> t) {
    if (t == null)
        return null;
    if (x.compareTo(t.element) < 0)
        return find(x, t.left);
    else if (x.compareTo(t.element) > 0)
        return find(x, t.right);
    else
        return t;
}

// 寻找最大节点
private BinaryNode<T> findMax(BinaryNode<T> t) {
    if (t == null)
        return null;
    else if (t.right == null)
        return t;
    return findMax(t.right);
}

// 寻找最小节点
private BinaryNode<T> findMin(BinaryNode<T> t) {
    if (t == null)
        return null;
    else if (t.left == null)
        return t;
    return findMin(t.left);
}

// 删除指定节点
private BinaryNode<T> remove(T x, BinaryNode<T> t) {
    if (t == null)
        return t;
    if (x.compareTo(t.element) < 0)
        t.left = remove(x, t.left);
    else if (x.compareTo(t.element) > 0)
        t.right = remove(x, t.right);
    else if (t.left != null && t.right != null) {
        t.element = findMin(t.right).element; // 把右树最小的复制给t
        t.right = remove(t.element, t.right); // 递归的删除
    } else {
        t = (t.left != null) ? t.left : t.right; // 一颗子树的情况
    }
    return t;
}
```

### 索引二叉树

- `leftSize` = 左子树大小 + 1
- ![](../0_Attachment/Pasted%20image%2020241227155648.png)

## AVL树

对于每一个结点, 其左子树和右子树的高度之差不超过1.

平衡因子: 一个节点右子树树高减去左子树树高. (AVL树: 每个节点的平衡因子的绝对值大小不超过1)

### 插入

- [视频演示](<https://www.bilibili.com/video/BV1tZ421q72h>)
- 插入一个新结点后，需要从插入位置沿通向根的路径回溯，检查各结点左右子树的高度差，如果发现某点高度不平衡则停止回溯
- 定义: 右旋为顺时针旋转, 左旋为逆时针旋转
- 对失衡节点进行旋转:
  1. LL型: 失衡节点平衡因子为-2, 左子节点平衡因子为-1, 右旋**失衡节点**
  2. RR型: 失衡节点平衡因子为2, 右子节点平衡因子为1, 左旋**失衡节点**
  3. LR型: 失衡节点平衡因子为-2, 左子节点平衡因子为1, 先左旋**左子节点**, 再右旋**失衡节点**
  4. RL型: 失衡节点平衡因子为2, 左子节点平衡因子为-1, 先右旋**右子节点**, 再左旋**失衡节点**

```java
private AVLNode insert( Comparable x, AVLNode t ) {
    if (t == null)
        t = new AVLNode( x, null, null );
    else if ( x.compareTo(t.element) < 0 ){
        t.left = insert(x, t.left);//不仅x插入左子树，而其左子树已经调平衡了，也就会子树已经旋转过了
        if(height(t.left) – height(t.right) == 2 )
            if(x.compareTo(t.left.element)<0)
                //根据大小进行调整
                t = rotateWithLeftChild (t);//左子树的左子树，只要做一次右旋
            else t = doubleWithLeftChild(t);//左子树的右子树，需要先左旋再右旋
    //下面是对称的插入在右子树上
    }else if(x.compareTo(t.element)>0) { 
        t.right = insert(x, t.right );
        if( height(t.right)–height(t.left)== 2)
            if(x.compareTo(t.right.element)>0)
                t = rotateWithRightChild(t);
            else t = doubleWithRightChild(t)
    }else;
    t.height = max(height(t.left), height(t.right)) + 1;
    return t;
}

// 一次右旋
private static AVLNode rotateWithLeftChild( AVLNode k2 ) {
    AVLNode k1 = k2.left;//k1持有k2的左子树
    k2.left = k1.right;//k1的右子树挂到k2的左子树上
    k1.right = k2;//把k2自己挂到k1的右子树上
    k2.height = max(height(k2.left), height(k2.right)) + 1 ;
    k1.height = max(height(k1.left), k2.height) + 1;
    return k1;
}

// 一次左旋
private static AVLNode rotateWithRightChild(AVLNode k2){
    AVLNode k1 = k2.right;//k1持有k2的右子树
    k2.right = k1.left;//k2的右子树挂到k1的左子树上
    k1.left = k2;//把k2自己挂到k1的左子树上
    k2.height = max(height(k2.left),height(k2.right)) + 1;
    k1.height = max(k2.height,k1.right) + 1;
    return k1;
}

//先左旋再右旋
private static AVLNode doubleWithLeftChild( AVLNode k3 ) {
    k3.left = rotateWithRightChild(k3.left);
    return rotateWithLeftChild( k3 );
}

//先右旋再左旋
private static AVLNode doubleWithRightChild(AVLNode k3){
    k3.right = rotateWithLeftChild(k3.right);
    return rotateWithRightChild( k3 );
}
```

### 删除

- 和二叉树的删除一致
- 删除后从被删除节点开始向根部回溯, 检查节点是否失衡 (不平衡会传递, 所有可能需要多次旋转)

## B树

- 多叉平衡搜索树

- 参考[CS61B B-Trees](<../UCB_CS61B/cs61b_lec_note.md####B-Trees>)

- M叉搜索树中每个内部节点最多用M个子节点, 在M-1个元素之间. 除了根节点外最少拥有M/2 (向上取整) 个子节点,  M/2 - 1 (向上取整)个元素

- 插入: 不会向下新增叶, 只会让一个叶节点元素增多, 超过限制后将中间节点向上分裂

- 删除: 
  - [视频演示](<https://www.bilibili.com/video/BV1JU411d7iY>)
  - 可能有下溢出的风险
  - 删除非叶节点的元素最终转换为删除叶节点元素
  - 如果没有下溢出则无需挑战
  - 如果发生下溢出
    - 兄弟够借: 借
    - 兄弟不够: 合并 (可能导致父节点下溢出)

## B+树

- (不考)
- TBD

