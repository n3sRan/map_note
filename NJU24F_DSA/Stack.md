# Stack & Queue

## Stack

### 中缀表达式转换后缀表达式

```java
import java.util.Stack;

public class PostfixConverter {

    public void postfix(String expression) {
        Stack<Character> stack = new Stack<>();
        char y;
        stack.push('#');

        for (char ch : expression.toCharArray()) {
            if (Character.isDigit(ch)) {
                System.out.print(ch);
            } else if (ch == ')') {
                y = stack.pop();
                while (y != '(') {
                    System.out.print(y);
                    y = stack.pop();
                }
            } else {
                while (!stack.isEmpty() && isp(stack.peek()) > icp(ch)) {
                    y = stack.pop();
                    System.out.print(y);
                }
                stack.push(ch);
            }
        }

        while (!stack.isEmpty()) {
            y = stack.pop();
            System.out.print(y);
        }
    }

    // 返回操作符的优先级
    private int icp(char ch) {
        switch (ch) {
            case '+':
            case '-':
                return 1;
            case '*':
            case '/':
                return 2;
            case '^':
                return 3;
            default:
                return 0;
        }
    }

    // 返回操作符的优先级
    private int isp(char ch) {
        switch (ch) {
            case '+':
            case '-':
                return 1;
            case '*':
            case '/':
                return 2;
            case '^':
                return 3;
            case '(':
                return 0;
            default:
                return -1;
        }
    }

    public static void main(String[] args) {
        PostfixConverter converter = new PostfixConverter();
        String expression = "3+5*(2-8)"; // 示例前缀表达式
        converter.postfix(expression);
    }
}
```

## Queue

### 杨辉三角

```java
public class Yanghui {
    public static void main(String args[ ] ) {
        int n = 10;
        int mat[][] = new int [n][];//申请第一维的存储空间
        int i, j;
        for ( i = 0; i < n; i++) {
            mat[i] = new int [i+1];//申请第二维的存储空间 ，每次长度不同
            mat[i][0] = 1;mat[i][i] = 1;
            for ( j = 1;j < i; j++){
                mat[i][j] = mat[i-1][j-1] + mat[i-1][j];
            }
                
        }
        for ( i = 0; i< mat.length; i++) {
            for ( j = 0; j < n-i; j++) System.out.print("   ");
            for ( j = 0; j < mat[i].length; j++){
                System.out.print("   " + mat[i][j])
            }  
            System.out.println( );
        }
    }
}
```