# 递归

## 全排列问题

1. 数组长度为1, 自身即为全排列
2. 数组长度大于1, 对第i个元素 (i从0到n-1), 与第0个元素交换, 将剩下的n-1个元素作全排列, 拼接后输出, 再交换回来

```java
public static void perm(char[] list, int k, int m) {
    // 检查是否达到了排列的末尾
    if (k == m) {
    	System.out.println(Arrays.toString(list));
    } else {
    	for (int i = k; i <= m; i++) {
    		// 交换元素
    		swap(list, k, i);
    		// 递归生成下一个部分的排列
    		perm(list, k + 1, m);
    		// 恢复交换
    		swap(list, k, i);
    	}
    }
}

private static void swap(char[] list, int i, int j) {
    char temp = list[i];
    list[i] = list[j];
    list[j] = temp;
}

public static void main(String[] args) {
    char[] list = { '1', '2', '3' };
    perm(list, 0, list.length - 1);
}
```

## 汉诺塔问题

```java
public static void Hanoi(int n, char source, char target, char auxiliary) {
    // 基本情况：如果只有一个盘子，直接移动
    if (n == 1) {
        System.out.println("Move disk 1 from " + source + " to " + target);
        return;
    }

    // 将 n - 1 个盘子从源柱子移动到辅助柱子
    Hanoi(n - 1, source, auxiliary, target);

    // Move the nth disk from source to target
    System.out.println("Move disk " + n + " from " + source + " to " + target);

    // 将 n - 1 个盘子从辅助柱子移动到目标柱子
    Hanoi(n - 1, auxiliary, target, source);
}
```