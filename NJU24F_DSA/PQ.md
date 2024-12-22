# PQ

## ADT

逻辑上的**最大**优先级队列

- instance 实例
  - 有限的元素的结合, 每个元素都有一个优先级
- operation 方法
  - `Create()`: 创建一个空的优先级队列
  - `Size()`: 返回队列中元素的个数
  - `Max()`: 返回队列中拥有最高优先级的元素
  - `Insert(x)`: 插入`x`进入队列
  - `DeleteMax(x)`: 删除队列中最高优先级的元素, 并且通过`x`返回它 (C++做法, 在Java中可以不传参数, 直接返回对象)

## 无序线性表实现

- `Insert()`: 直接放链表最右边, 时间复杂度 Θ(1).
- `Max()` / `DeleteMax()`: 需要遍历线性表, 时间复杂度 Θ(N).

## 堆实现

[参考61B-Heap](../UCB_CS61B/cs61b_lec_note.md####Heap)

物理 (代码) 上实现: 用数组, 0号位置不用, 1号位置为根节点, 第n个节点的左子节点为2n, 右子节点为2n+1, 父节点为n/2.

`Insert()` 上浮

```java
public MaxHeap insert(int x) throws NoMemException {
        if (currentSize == maxSize) {
            throw new NoMemException();
        }
        int i = ++currentSize;
        while (i != 1 && x > heap[i / 2]) {
            heap[i] = heap[i / 2];
            //不必每次都进行完全交换
            i /= 2;
        }
    	// 最后再把x放进正确的位置
        heap[i] = x;
        return this;
    }
```

`Delete()` 下沉

```java
public MaxHeap deleteMax() throws OutOfBoundsException {
        if (currentSize == 0) {
            throw new OutOfBoundsException();
        }
        
        // 获取最大值（根节点）
        int maxValue = heap[1];
        
        // 将最后一个元素移到根节点
        int lastValue = heap[currentSize--];
        int i = 1; // i 指向树根
        int ci = 2; // ci 先指向左子树
        
    	// 假设lastValue已经在根, 开始下沉
        while (ci <= currentSize) {
            // 如果 ci 未越界, 并且左子树的值小于右子树的值, 转向右子树
            // 保证父节点大于两个子节点
            if (ci < currentSize && heap[ci] < heap[ci + 1]) {
                ci++;
            }
            // 如果最后一个节点大于当前节点, 退出循环
            // 沉到了正确的地方 不需要再下沉了
            if (lastValue >= heap[ci]) break;
            heap[i] = heap[ci]; // 移动当前节点
            i = ci; // lastValue的新位置
            ci *= 2; // 更新ci为当前节点的左子节点
        }
        
        // 将最后一个节点放置到合适的位置
        heap[i] = lastValue;
        
        // 这里可以选择返回最大值或返回当前对象maxvlaue
        return this;
    }
```

## 初始化PQ

### 已在内存中

这个数组已经在内存里了, 只是顺序不对.

从最后一个节点的父节点开始, 对所有**非叶节点**进行进行**下沉**操作 (沉完第n-1层就到第n-2层).

时间复杂度: Θ(N).

![](../0_Attachment/Pasted%20image%2020241222162229.png)

```java
public void initialize(int[] a, int size, int arraySize) {
        heap = new int[arraySize + 1];
        System.arraycopy(a, 0, heap, 1, size); // 复制数组
        
        currentSize = size;
        maxSize = arraySize;
        
        // 从最后一个父节点向上调整最大堆结构
        for (int i = currentSize / 2; i >= 1; i--) {
            int y = heap[i]; // 保存父节点的值
            int c = 2 * i;   // c 标识左子节点
            while (c <= currentSize) {
                // 如果右子节点存在且大于左子节点, 转向右子节点
                if (c < currentSize && heap[c] < heap[c + 1]) {
                    c++;
                }
                // 如果 y 大于等于当前节点, 退出循环
                if (y >= heap[c]) {
                    break;
                }
                // 将当前节点移动到父节点位置
                heap[c / 2] = heap[c];
                c *= 2; // c 变为当前节点的左子节点
            }
            heap[c / 2] = y; // 将 y 放回合适的位置
        }
    }
```

### 不在内存中

对逐个元素进行`Insert()`

时间复杂度: Θ(NlogN)

![](../0_Attachment/Pasted%20image%2020241222162618.png)

## PQ应用

### 堆排序

1. 初始化一个n个元素的最大堆, 时间复杂度 O(N)
2. 每次删除最大的元素, 调整堆的时间复杂度 O(logN)
3. 总时间复杂度 O(NlogN)

代码上: 将根与堆最后一个节点交换, 再对新的根节点下沉, 删除完整个堆后就得到排序好的数组.

### 查找问题

在N个元素中找出第K个最大元素.

#### 1A算法

读入N个元素放入数组, 并将其选择排序, 返回适当的元素. 时间复杂度 O($N^2$)

#### 1B算法

1. 将K个元素读入数组, 并对其排序(按递减次序). 最小者在第K个位置上.
2. 一个一个地处理其余元素：每读入一个元素与数组中第K个元素(在K个元素中为最小)比较, 如果大于,则删除第K个元素, 再将该元素放在合适的位置上(调整过程)。如果小于, 则舍弃。最后在数组K位置上的就是第K个最大元素。
3. 时间复杂度 O($K + (N - K) * \log K$) = O($\log K$), 当 K = (N/2) 时为 Θ(Nlog N )

#### 堆实现: 6A算法

假设求第K个最小元素

1. 将N个元素建最小堆, O(N)
2. 执行K次delete, O(KlogN), 总时间复杂度 O(N + KlogN)
   - 如果 K = (N/2), O(NlogN)
   - 如果 K = N, O(NlogN), 即堆排序

#### 堆实现: 6B算法

假设求第K个最大元素

1. 前K个元素建立最小堆, O(K)
2. 其余元素逐一读入: 每读入一个元素与堆中第K个最大元素比 (实际上是堆中最小元素), O(1)
   - 大于, 将小元素去掉(堆顶), 该元素进入, 进行一次调整, O(logK)
   - 小于, 舍弃
3. O($K + (N - K)\log K$) = O($N\log K$), K = (N/2) 时, Θ(Nlog N)