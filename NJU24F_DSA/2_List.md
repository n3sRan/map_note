# List

## 单链表

```java
// 代表结点的类
class ListNode {   
    object element;
    ListNode next;
    ListNode( object theElement) {
        this( theElement, null);
    }
    ListNode( object theElement, ListNode n) {
        element = theElement; 
        next = n;
    }
}

// 代表游标位置的类
public class LinkedListItr {
    LinkedListItr( ListNode  theNode) {
        current = theNode;
    }
    public boolean isPastEnd( ) {
        return current == null; 
    }
    public object retrieve() {
        //获得当前节点的数据
        return isPastEnd( ) ? null : current.element; 
    }
    public void advance( ) {
        if( ! isPastEnd( ) ) current = current.next; 
    }
    ListNode current; 
}

// 代表线性表本身的类
public class LinkedList {
    private ListNode header;
    
    public LinkedList( ) {
        header = new ListNode( null );
    }
    
    //这里是含有表头节点的单链表，如果不带表头的话，应该是heder = null
    public boolean isEmpty( ) {
        return header.next = = null ;
    }
    
    public void makeEmpty( ) {
        header.next = null;
    }
    
    //返回指向头指针的Itr
    public LinkedListItr zeroth( ) {
        return new LinkedListItr( header );
    }
    
    //返回指向第一个项的Itr
    public LinkedListItr first( ) {
        return new LinkedListItr( header.next );
    } 
    
    // 查找特定项 O(N)
    public LinkedListItr find( object x ) {
    	ListNode itr = header.next;
    	while ( itr != null && !itr.element.equals( x ))
        	itr = itr.next;
        return new LinkedListItr( itr );
	} 
    
    // 移除节点
    public void remove( object x ) {
    	LinkedListItr p = findprevious( x );
    	if( p.current.next != null )
        	p.current.next = p.current.next.next;
	}
    
    // 查找上一个节点
    public LinkedListItr findPrevious( object x ) {
    	ListNode itr = header;
    	while( itr.next !=null && !itr.next.element.equals( x )) itr = itr.next;
    	return new LinkedListItr( itr );
	}
   	
	// 插入节点
    public void insert( object x, LinkedListItr p ) {
         if( p!=null && p.current != null )
			p.current.next = new ListNode( x, p.current.next );
	}

    // 打印线性表
    public static void printList( LinkedList theList ) {   
    	if ( theList.isEmpty( ) ) 
        	System.out.print ("Empty list");
    	else {
        	LinkedListItr itr = theList.first();
        	for(;! Itr.isPastEnd(); itr. Advance( ) )
            	System.out.print(itr.retrieve() + " " );
        }
    	System.out.println();
	}
}
```

## 循环单链表

![](../0_Attachment/Pasted%20image%2020241229232542.png)

### 约瑟夫问题

```java
w = m;
for( int i = 1; i<= n-1; i++) {
    for (int j = 1; j<=w-1; j++) rear = rear.link;
    if (i = = 1) { 
        head = rear.link ; p = head;
    } else { 
        p.link = rear.link;
        p = rear.link;
    }
    rear.link = p.link;
}
p.link = rear; 
rear.link = null;
```



## 双向链表

TBD

- 普通双向链表
- 双向循环链表
- 带表头的双向链表
- 带表头的空双向链表

## 多项式问题

### 线性表数组实现

```java
public class Polynomial {
	public static final int MAX-DEGREE = 100;
    private int coeffArray[ ] = new int [MAX-DEGREE + 1];
    private int highPower = 0;
    
    public Polynomial( ) {
        zeroPolynomial( );
    }
    
    public void insertTerm( int coef, int exp )
        
	//清空多项式
    public void zeroPolynomial(){
        for( int i = 0; i<=MAX-DEGREE; i++) 
            coeffArray[i] = 0;
        highPower = 0; 
    }
    // 加法
    public Polynomial add( Polynomial rhs ){
        Polynomial sum = new Polynomial( );
        sum.highPower = max( highPower, rhs.highPower );
        for( int i = sum.highPower;  i>=0;  i--)
            sum.coeffArray[i] = coeffArray[i] + rhs.coeffArray[i];
        return sum; 
    }
    
    // 乘法
    public Polynomial multiply( Polynomial rhs ) throws Overflow {
        Polynomial product = new Polynomial( );
        product.highPower = highPower + rhs.highPower;
        if( product.highPower > MAX-DEGREE ) {
            throw new overflow( );
        }	
        for( int i = 0; i<= highPower; i++ ) {
            for( int j = 0; j<= rhs.highPower; j++ ) {
                product.coeffArray[ i + j ] += coeffArray[i] * rhs.coeffArray[j];
            }
        }
        return product;
    }
    
    public void print( )
}
```

### 单链表实现

TBD

## 静态链表

由数组实现的单链表

![](../0_Attachment/Pasted%20image%2020241230094253.png)

```java
class CursorNode {
    object element;
	int next;
    
	CursorNode( object theElement ) { 
        this( theElement, 0 ); 
    }
    
    CursorNode( object theElement, int n ) {
        element = theElement;
        next = n;
	}
}

public class CursorListItr {
    int current;
    
    CursorListItr( int theNode ) {
        current = theNode; 
    }
    
    public boolean isPastEnd( ) {
        return current = = 0;
    }
    
    public object retrieve( ) {
        return isPastEnd( ) ? null:
        CursorList.cursorSpace[ current ].element;
    }
    
    public void advance( ) { 
        if( !isPastEnd( ))
		current = CursorList.cursorSpace[current].next;
	} 
}

public class CursorList {
    private int header; 
    static CursorNode [ ] cursorSpace;
    private static final int SPACE-SIZE = 100;
    
    // 静态变量对一个类只有一个
    static {
        cursorSpace = new CursorNode[SPACE-SIZE];
        for( int i = 0; i<SPACE-SIZE; i++) cursorSpace[ i ] = new CursorNode(null, i + 1);
        cursorSpace[SPACE-SIZE-1].next = 0;
    }
    
    // 方法
    public CursorList( ) {
        header = alloc( );   cursorSpace[ header ].next = 0;
    }
    
    public boolean isEmpty( ) {
        return cursorSpace[header].next = = 0;
    }
    
    public CursorListItr zeroth( ) {
        return new CursorListItr( header );
    }
    
    public CursorListItr first( ) {
        return new CursorListItr( cursorSpace[ header ].next );
    }
    
    private static int alloc( ) {
        int p = cursorSpace[ 0 ].next;
        cursorSpace[0].next = cursorSpace[p].next;
        if( p == 0 ) throw new OutOfMemoryError( ); return p; 
    }
    
    private static void free( int p ) {
        cursorSpace[p].element = null;
        cursorSpace[p].next = cursorSpace[0].next;
        cursorSpace[0].next = p; 
    }
    
    
    public void makeEmpty( ) {
        while( !isEmpty( ) )
            remove( first( ).retrieve( ) );
    }
    
    public CursorListItr find( object x ) {
        int itr = cursorSpace[ header ].next;
        while( itr != 0 && !cursorSpace[ itr ].element.equals( x ) ){
            itr = cursorSpace[ itr ].next;
        }  
        return new CursorListItr( itr );
    }
    
    public void insert( object x, CursorListItr p ) {
        if( p != null && p.current != 0) {
            int pos = p.current;
            int tmp = alloc( );
            cursorSpace[ tmp ].element = x;
            cursorSpace[ tmp ].next = cursorSpace[ pos ].next;
            cursorSpace[ pos ].next = tmp;
        }
    }
        
    public void remove( object x ) {   
        CursorListItr p = findPrevious( x );
        int pos = p.current;
        if( cursorSpace[ pos ].next != 0 ) {
            int tmp = cursorSpace[ pos ].next;
            cursorSpace[ pos ].next = cursorSpace[ tmp ].next;
            free( tmp );
        }
    }
    
    public CursorListItr findPrevious( object x )
}
```

