# Graph

简略版: [CS61B-Graph](<../UCB_CS61B/cs61b_lec_note.md####Graph>)

## 图实现

### 邻接矩阵

![](../0_Attachment/Pasted%20image%2020241223170709.png)

### 邻接表

![](../0_Attachment/Pasted%20image%2020241223170927.png)

### 邻接多重表

![](../0_Attachment/Pasted%20image%2020241223171113.png)

![](../0_Attachment/Pasted%20image%2020241223171127.png)

## 图遍历

### DFS

### BFS

## 最小生成树

### Prim算法

1. 使用两个数组`Lowcost[ ]`, `nearvex[ ]`
2. `Lowcost[]`: 存放生成树顶点集合内顶点到生成树外各顶点的边上的当前最小权值.
3. `nearvex[]`: 记录生成树顶点集合外各顶点, 距离集合内那个顶点最近.

```java
public void Prim(MinSpanTree T) {
    float[] lowcost = new float[NumVertices];
    int[] nearvex = new int[NumVertices];

    // O(N)
    for (int i = 1; i < NumVertices; i++) {
        lowcost[i] = Edge[0][i]; // 0到其他所有边的权值
        nearvex[i] = 0;
    }

    nearvex[0] = -1; // 将0的nearvex设为-1, 表示已经加进MST
    MSTEdgeNode e = new MSTEdgeNode();

    // O(N^2)
    for (int i = 1; i < NumVertices; i++) {
        float min = Float.MAX_VALUE;
        int v = 0;

        for (int j = 1; j < NumVertices; j++) {
            if (nearvex[j] != -1 && lowcost[j] < min) {
                v = j;
                min = lowcost[j]; // 选择最小的边
            }
        }

        if (v != 0) {
            e.tail = nearvex[v];
            e.head = v;
            e.cost = lowcost[v];
            T.Insert(e); // 添加边进入最小生成树中去
            nearvex[v] = -1;

            for (int j = 1; j < NumVertices; j++) {
                if (nearvex[j] != -1 && Edge[v][j] < lowcost[j]) {
                    lowcost[j] = Edge[v][j];
                    nearvex[j] = v;
                }
            }
        }
    }
}
```

### Kruskal算法

1. 建立E条边的最小堆
   - 检测邻接矩阵, O($N^2$). 邻接表, O($E$).
   - 建堆, O($E \log E$).
2. 构造最小生成树
   - E次出堆, O($E \log E$).
   - 2E次`find()`, O($E \log N$).
   - N-1次`union()`, O($N$)

```java
public void kruskal() {
    int edgesAccepted;
    DisjSet s;
    priorityQueue h;
    Vertex u, v;
    SetType uset, vset;
    Edge e;
    h = readGraphIntoHeapArray();
    h.buildHeap() ;
    s = new DisjSet(NUM_VERTICES);

    edgesAccepted = 0 ;
    while(edgesAccepted < NUM_VERTICES – 1){
        e = h.deleteMin() ; //Edge e = (u, v)
        uset = s. find(u);
        vset = s.find(v);
        if(uset != vset) {
            edgesAccepted++;
            s.union(uset, vset);
        }
    }
}
```

## 最短路径

### Dijkstra 算法

适用于查找非负权单源最短路径.

访问一个节点时, 该节点的最短路径已经确定: 不可能从一条长的路径绕一圈而绕出比短路径更短的路线.

暴力解法: O($N^2$)

```java
private static final float MAXNUM = Float.MAX_VALUE;
private float[][] Edge; // 邻接矩阵
private float[] dist; // 存储从起点到每个节点的最短距离
private int[] s; // 存储节点是否已被访问
private int[] path; // 存储最短路径

public void shortestPath(int n, int v) {
     for (int i = 0; i < n; i++) {
         dist[i] = Edge[v][i]; // 更新距离
         s[i] = 0; // 初始化访问状态
         if (i != v && dist[i] < MAXNUM) {
             path[i] = v; // 可达，记录路径
         } else {
             path[i] = -1; // 不可达，记录 -1
         }
     }

     s[v] = 1; // 访问当前节点
     dist[v] = 0; // 当前节点到自己的距离为 0

     for (int i = 0; i < n - 1; i++) {
         float min = MAXNUM;
         int u = v;
         for (int j = 0; j < n; j++) {
             if (s[j] == 0 && dist[j] < min) {
                 u = j; // 找到未访问中距离最小的节点
                 min = dist[j];
             }
         }
         s[u] = 1; // 标记节点为已访问

         for (int w = 0; w < n; w++) {
             if (s[w] == 0 && Edge[u][w] < MAXNUM && dist[u] + Edge[u][w] < dist[w]) {
                 dist[w] = dist[u] + Edge[u][w]; // 更新距离
                 path[w] = u; // 更新路径
             }
         }
     }
}
```

优先队列实现: O($m \log n$)

```java
private int[] dis = new int[MAXN];
private boolean[] vis = new boolean[MAXN];

public void dijkstra(int n, int s) {
    private PriorityQueue<Node> q = new PriorityQueue<>();
    for (int i = 0; i <= n; i++) {
        dis[i] = Integer.MAX_VALUE;
        vis[i] = false;
    }
    dis[s] = 0;
    q.offer(new Node(0, s));
    while (!q.isEmpty()) {
        int u = q.poll().u;
        if (vis[u]) continue;
        vis[u] = true;
        for (Edge ed : e[u]) {
            int v = ed.v, w = ed.w;
            if (dis[v] > dis[u] + w) {
                dis[v] = dis[u] + w;
                // 可以把同一节点的不同距离加进PQ
                q.offer(new Node(dis[v], v));
            }
        }
    }
}
```

### Bellman–Ford 算法

可以求出有负权的最短路

$dist^k[u]$表示从源点v (固定) 到终点u (遍历) 的**只经过k条边**的最短路径长度.

递推: $dist^k[u] = min \{dist^{k-1}[u], dist^{k-1}[j] + Edge[j][u] \}$, 即尝试用v-j-u这条路径去更新到u的最短路的长度, 如果更优则进行更新.

最多递推$N-1$轮, 每轮检查所有边, 邻接表时间复杂度 O($NE$), 邻接矩阵时间复杂度 O($N^3$)

```java
private static final int MAXNUM = Integer.MAX_VALUE;
private int[][] Edge;
private int[] dist;
private int[] path; //记录路径
private int n;

public void BellmanFord(int v) {
    // 动态规划
    for (int i = 0; i < n; i++) {
        // 初始化dist距离数组
        dist[i] = Edge[v][i];
        if (i != v && dist[i] < MAXNUM) {
            path[i] = v;
        } else {
            path[i] = -1;
        }
        // 初始化路径数组
    }

    for (int k = 2; k < n; k++) {
	    // 每一轮都尝试更新到达u的最短路径
        for (int u = 0; u < n; u++) {
            if (u != v) {
	            // 遍历所有指向u的边
                for (int i = 0; i < n; i++) {
                    // 一直算到n-1步
                    if (Edge[i][u] != 0 && Edge[i][u] < MAXNUM && dist[u] > dist[i] + Edge[i][u]) {
                        dist[u] = dist[i] + Edge[i][u];
                        path[u] = i;
                    }
                }
            }
        }
    }
}
```

### Floyed 算法

适用于求任意两个结点之间的最短路.

第k次循环: 只允许经过结点1到k, 求从结点i到结点j的最短路长度.

```java
public void Alllength() {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            a[i][j] = Edge[i][j];
            if (i != j && a[i][j] < MAXNUM) {
                path[i][j] = i;
            } else {
                path[i][j] = -1;
            }
        }
    }

    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (a[i][k] + a[k][j] < a[i][j]) {
                    a[i][j] = a[i][k] + a[k][j];
                    path[i][j] = path[k][j];
                }
            }
        }
    }
}
```

## 图应用