# Sort

## 插入排序

### 直接插入

O($N^2$)

稳定

### 折半插入

O($N \log N$)

稳定

### 希尔排序

1. 取一增量(间隔gap < n), 按增量分组, 对每组使用**直接插入排序**或其他方法进行排序.
2. 减少增量 (分的组减少, 但每组记录增多). 直至增量为1, 即为一个组时.

不稳定

## 交换排序

### 冒泡排序

O($N^2$)

稳定

### 快速排序

见[CS61B-QuickSort](<../UCB_CS61B/cs61b_lec_note.md####Quicksort>)

不稳定

## 选择排序

### 直接排序

每次选择待排数组中的最小值, 与待排数组第一个值交换.

每次选择待排数组中的最大值, 与待排数组最后一个值交换.

O($N^2$)

稳定

### 锦标赛排序

![](../0_Attachment/Pasted%20image%2020241223180437.png)

### 堆排序

参考[PQ应用-堆排序](<./5_PQ.md###堆排序>)

不稳定

## 秩排序

```java
public static void Rank( int [] a, int n, int [] r) {
    // O(N^2)
    //Rank the n elements a[0:n-1]
    for(int i=0;i<n;i++){
        r[i]=0;
    }
    //首先将全部的数字重置为0
    for(int i=1;i<n;i++)
        for(int j=0;j<i;j++)
            if(a[j]<=a[i])
                r[i]++;
            else
                r[j]++;
            //胜者排名不需要向后增加，而败者需要增加次序。
}
public static void Rearrange( int [ ]a, int n, int[ ] r) {
    //In-place rearrangement into sorted order
    for(int i=0;i<n;i++)
        while(r[i]!=i) {
            //在数组[n]上的应该是第n+1大的数字
            int t=r[i];
            swap(a[i],a[t]);
            swap(r[i],r[t]);
        }
}
```

## 基数排序

CS61B-RadixSort

从最低位开始, 将数放进十个桶中 (队列), 再从小编号到大编号桶依次取出, 开始下一位的操作.

## 归并排序

```java
public static void mergeSort( Comparable [ ] a ) {
    Comparable [ ] tmpArray = new Comparable[a.length];
    mergeSort( a, tmpArray, 0, a.length – 1 );
}                            
private static void mergeSort( Comparable [ ] a, Comparable [] tmpArray, int left, int right ) {
    if( left < right ) {
        int center = ( left + right ) / 2;
        mergeSort(a, tmparray, left, center );
        mergeSort(a, tmpArray, center + 1, right );
        merge( a, tmpArray, left, center + 1, right );
    }
}
private static void merge( Comparable [ ] a, Comparable [] tmpArray, int leftPos, int rightPos, int rightEnd ) {
    int leftEnd = rightPos – 1;
    int tmpPos = leftPos;
    int numElements = rightEnd – leftPos + 1;
    while( leftPos <= leftEnd && rightPos <= rightEnd )
        if( a[ leftPos ].compareTo( a[ rightPos ] ) <= 0 )
            tmpArray[ tmpPos++ ] = a[ leftPos++ ];
        else
            tmpArray[ tmpPos++ ] = a[ rightPos++ ];
    while( leftPos <= leftEnd )
        tmpArray[ tmpPos++ ] = a[ leftPos++ ];
    while( rightpos <= rightEnd)
        tmpArray[ tmpPos++] = a[ rightpos++ ];
    for( int i = 0; i < numElements; i++, rightEnd-- )
        a[ rightEnd ] = tmpArray[ rightEnd ];
}     
```

