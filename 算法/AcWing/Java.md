# 输入输出处理

## 输入样例-字符串

```java
3
aba
5
ababa
```
高效流读入方法

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;

public class Main {

    public static void main(String[] args) throws NumberFormatException, IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(System.out));

        int n = Integer.parseInt(reader.readLine());

        // 模式串p
        String P = reader.readLine();
        char[] p = new char[n + 1];
        for (int i = 1; i <= n; i++)
            p[i] = P.charAt(i - 1);

        int m = Integer.parseInt(reader.readLine());
        String S = reader.readLine();
        // 总串s
        char[] s = new char[m + 1];
        for (int i = 1; i <= m; i++)
            s[i] = S.charAt(i - 1);
    }
}
```

Scanner读入

**注意读完数字后，要nextLine，然后才能到下一行**

```java
public class Main {

    public static void main(String[] args) {

        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        scanner.nextLine();
        String P = scanner.nextLine();
        // 模式串p
        char[] p = new char[n + 1];
        for (int i = 1; i <= n; i++)
            p[i] = P.charAt(i - 1);

        int m = scanner.nextInt();
        scanner.nextLine();
        String S = scanner.nextLine();
    }
}

```

