# Disjoint Set

不相交集, 并查集

*其实61B在讲渐进分析时讲的很明白了, 只是我没怎么做笔记*

## Equivalence Class

等价类

满足

- Reflexive 自反性
- Symmetric 对称性
- Transitive 传递性

## 并查集功能

- `Combine(a,b)`
- `Find(e)`

## 代码实现

```java
public class DisjSets {
    int[] s;
    
    //并查集的构造方法
    public DisjSets( int numElements ) {
    	s = new int [numElements];
    	for( int i = 0; i < s.length; i++ )
        	s[i] = -1; //一个根结点
	}
    
    //并查集的合并
    public void union( int root1, int root2 ) {
    	s[root2] = root1;
	}
    
    //并查集的查找，使用递归完成
    public int find( int x ) {
    	if( s[x] < 0 )
        	return x;
    	else
        	return find( s[x] );
	}
}
```

### 性能提升

#### 保持树的平衡

将矮的树挂到高的树下, 相当于结点数少的树挂到结点多的树下面.

根节点使用负数记录树高.

```java
public void union( int root1, int root2 ) { 
    if(s[root2] < s[root1])
        s[root1] = root2;
    else {
        if(s[root1] == s[root2] )
            s[root1]--;
        s[root2] = root1;
    }
}
```

#### 路径压缩

每进行一次Find, 将所有叶子挂到树根下以降低树的高度 (此时需要根节点用负数记录节点数)