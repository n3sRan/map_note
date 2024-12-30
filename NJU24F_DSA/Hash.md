# Hashing

## Hash Function

哈希函数

### 取余法

通常取质数, 防止冲突.

### 平方取中法

![](../0_Attachment/Pasted%20image%2020241206211936.png)

### 乘法杂凑函数

![](../0_Attachment/Pasted%20image%2020241206212001.png)

### 字符串的哈希函数

#### 简单相加

#### 指数多项式

## Solve Collision

冲突解决

### Linear Probing 线性探测法

被占了往后放, 搜索时若不同往后搜, 不能直接删而是加删除标记 

### Quadratic probing 二次探测法

规则如上, 但放的下标为$d+1, d+2^2, d+3^2 ...$, 。$d$为由哈希函数算出的下标

### Double Hashing 双散列哈希

如果第一个哈希函数算出的下标d被占, 则使用第二个哈希函数算出哈希值c, 放$d+c, d+2c, d+3c ...$

占用率 > 70%或没地方放时, 进行扩容 (rehash), 新空间大小为**比原大小的两倍还大的质数**

### Separate Chaining 分离链接法

将哈希表每个表项设为线性表

## Code

代码实现

前三个主要是`findPos()`方法的不同.

### Linear Probing 线性探测法

```java
// 通用类与接口
public interface Hashable {
    int hash(int tableSize); // 计算哈希值的方法
}

// 哈希表中的条目类
class HashEntry {
    Hashable element; // 存储的元素
    boolean isActive; // 条目是否活跃

    // 构造一个活跃的条目
    public HashEntry(Hashable e) {
        this(e, true);
    }

    // 构造一个条目，指定元素和活跃状态
    public HashEntry(Hashable e, boolean i) {
        this.element = e;
        this.isActive = i;
    }
}

public class LinearProbingHashTable {
    private static final int DEFAULT_TABLE_SIZE = 11; // 默认的哈希表大小
    protected HashEntry[] array; // 哈希表数组
    private int currentSize; // 当前活跃条目的数量

    // 默认构造函数
    public LinearProbingHashTable() {
        this(DEFAULT_TABLE_SIZE);
    }

    // 指定大小的构造函数
    public LinearProbingHashTable(int size) {
        allocateArray(size); // 分配数组
        makeEmpty(); // 清空哈希表
    }

    // 分配哈希表数组
    private void allocateArray(int arraySize) {
        array = new HashEntry[arraySize];
    }

    // 清空哈希表
    public void makeEmpty() {
        currentSize = 0;
        for (int i = 0; i < array.length; i++) {
            array[i] = null;
        }
    }

    // 查找元素
    public Hashable find(Hashable x) {
        int currentPos = findPos(x); // 找到元素的位置
        return isActive(currentPos) ? array[currentPos].element : null; // 返回元素或null
    }

    // 找到元素的位置
    private int findPos(Hashable x) {
        int currentPos = x.hash(array.length); // 计算哈希值
        while (array[currentPos] != null && !array[currentPos].element.equals(x)) {
            currentPos++; // 线性探测法
            if (currentPos >= array.length) {
                currentPos -= array.length; // 环绕数组
            }
        }
        return currentPos;
    }

    // 判断位置是否活跃
    private boolean isActive(int currentPos) {
        return array[currentPos] != null && array[currentPos].isActive;
    }

    // 插入元素
    public void insert(Hashable x) {
        int currentPos = findPos(x); // 找到插入位置
        if (isActive(currentPos)) {
            return; // 如果位置活跃，则返回
        }
        array[currentPos] = new HashEntry(x, true); // 插入新条目
        if (++currentSize > array.length / 2) {
            rehash(); // 如果超过负载因子，则重新哈希
        }
    }

    // 移除元素
    public final void remove(Hashable x) {
        int currentPos = findPos(x); // 找到元素位置
        if (isActive(currentPos)) {
            array[currentPos].isActive = false; // 将条目设为非活跃
        }
    }

    // 重新哈希
    private void rehash() {
        HashEntry[] oldArray = array; // 保存旧数组
        allocateArray(nextPrime(2 * oldArray.length)); // 分配新数组
        currentSize = 0;
        for (HashEntry entry : oldArray) {
            if (entry != null && entry.isActive) {
                insert(entry.element); // 插入旧数组中的活跃条目
            }
        }
    }

    // 找到下一个素数
    private static int nextPrime(int n) {
        if (n % 2 == 0) {
            n++;
        }
        while (!isPrime(n)) {
            n += 2;
        }
        return n;
    }

    // 判断是否为素数
    private static boolean isPrime(int n) {
        if (n == 2 || n == 3) {
            return true;
        }
        if (n == 1 || n % 2 == 0) {
            return false;
        }
        for (int i = 3; i * i <= n; i += 2) {
            if (n % i == 0) {
                return false;
            }
        }
        return true;
    }

    // 计算字符串的哈希值
    public static int hash(String key, int tableSize) {
        int hashVal = 0;
        for (int i = 0; i < key.length(); i++) {
            hashVal = 37 * hashVal + key.charAt(i);
        }
        hashVal %= tableSize;
        if (hashVal < 0) {
            hashVal += tableSize;
        }
        return hashVal;
    }
}
```

### Quadratic probing 二次探测法

```java
 public class QuadraticProbingHashTable {
     
    // 其余代码同上
     
    // 找到元素的位置
    private int findPos(Hashable x) {
        int collisionNum = 0;
        int currentPos = x.hash(array.length); // 计算哈希值
        while (array[currentPos] != null && !array[currentPos].element.equals(x)) {
            currentPos += 2 * collisionNum - 1; // 二次探测法 n2 - (n-1)2= 2n-1
            if (currentPos >= array.length) {
                currentPos -= array.length; // 环绕数组
            }
        }
        return currentPos;
    }
     
     // 其余代码同上
}
```

### Double Hashing 双散列哈希

```java
public class DoubleHashingHashTable {
         
    // 其余代码同上
     
    // 找到元素的位置
    private int findPos(Hashable x) {
        int offset = hash2(x); // 计算第二个哈希值
        int currentPos = x.hash(array.length); // 计算第一个哈希值
        while (array[currentPos] != null && !array[currentPos].element.equals(x)) {
            currentPos += offset; // 使用第二个哈希值进行探测
            if (currentPos >= array.length) {
                currentPos -= array.length; // 环绕数组
            }
        }
        return currentPos;
    }

    // 计算第二个哈希值
    private int hash2(Hashable x) {
        return 7 - (x.hash(array.length) % 7); // 使用一个小于表大小的素数
    }
     
     // 其余代码同上
}
```



### Separate Chaining 分离链接法

- `Employee` 类实现了 `Hashable` 接口，这意味着 `Employee` 类必须实现 `Hashable` 接口中的 `hash` 方法。
- `SeparateChainingHashTable` 类的方法（如 `insert`, `remove`, `find`）接受 `Hashable` 类型的参数，因此可以处理任何实现了 `Hashable` 接口的对象，包括 `Employee` 对象

```java
import java.util.LinkedList;

public class SeparateChainingHashTable {
    // 默认的哈希表大小
    private static final int DEFAULT_TABLE_SIZE = 101;
    // 存储链表数组
    private LinkedList<Hashable>[] theLists;

    // 默认构造函数，使用默认大小
    public SeparateChainingHashTable() {
        this(DEFAULT_TABLE_SIZE);
    }

    // 带参数的构造函数，指定哈希表大小
    public SeparateChainingHashTable(int size) {
        theLists = new LinkedList[nextPrime(size)];
        for (int i = 0; i < theLists.length; i++) {
            theLists[i] = new LinkedList<>();
        }
    }

    // 插入元素
    public void insert(Hashable x) {
        LinkedList<Hashable> whichList = theLists[x.hash(theLists.length)];
        if (!whichList.contains(x)) {
            whichList.add(x);
        }
    }

    // 移除元素
    public void remove(Hashable x) {
        LinkedList<Hashable> whichList = theLists[x.hash(theLists.length)];
        whichList.remove(x);
    }

    // 查找元素
    public Hashable find(Hashable x) {
        LinkedList<Hashable> whichList = theLists[x.hash(theLists.length)];
        int index = whichList.indexOf(x);
        return index != -1 ? whichList.get(index) : null;
    }

    // 清空哈希表
    public void makeEmpty() {
        for (LinkedList<Hashable> list : theLists) {
            list.clear();
        }
    }

    // 哈希函数，将字符串转换为哈希值
    public static int hash(String key, int tableSize) {
        int hashVal = 0;
        for (int i = 0; i < key.length(); i++) {
            hashVal = 37 * hashVal + key.charAt(i);
        }
        hashVal %= tableSize;
        if (hashVal < 0) {
            hashVal += tableSize;
        }
        return hashVal;
    }

    // 获取下一个素数
    private static int nextPrime(int n) {
        if (n % 2 == 0) {
            n++;
        }
        while (!isPrime(n)) {
            n += 2;
        }
        return n;
    }

    // 判断一个数是否为素数
    private static boolean isPrime(int n) {
        if (n == 2 || n == 3) {
            return true;
        }
        if (n == 1 || n % 2 == 0) {
            return false;
        }
        for (int i = 3; i * i <= n; i += 2) {
            if (n % i == 0) {
                return false;
            }
        }
        return true;
    }
}

public interface Hashable {
    // 哈希函数接口，返回哈希值
    int hash(int tableSize);
}

public class Employee implements Hashable {
    private String name;
    private double salary;
    private int seniority;

    // 构造函数
    public Employee(String name, double salary, int seniority) {
        this.name = name;
        this.salary = salary;
        this.seniority = seniority;
    }

    // 实现 Hashable 接口的哈希函数
    @Override
    public int hash(int tableSize) {
        return SeparateChainingHashTable.hash(name, tableSize);
    }

    // 重写 equals 方法，比较两个 Employee 对象是否相等
    @Override
    public boolean equals(Object rhs) {
        if (this == rhs) {
            return true;
        }
        if (rhs == null || getClass() != rhs.getClass()) {
            return false;
        }
        Employee employee = (Employee) rhs;
        return name.equals(employee.name);
    }

    // 重写 hashCode 方法
    @Override
    public int hashCode() {
        return name.hashCode();
    }
}
```