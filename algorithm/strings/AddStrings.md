# 字符串相加

给定两个字符串形式的非负整数 `num1` 和`num2` ，计算它们的和并同样以字符串形式返回。

你不能使用任何內建的用于处理大整数的库（比如 `BigInteger`）， 也不能直接将输入的字符串转换为整数形式。



**示例 1：**

```
输入：num1 = "11", num2 = "123"
输出："134"
```

**示例 2：**

```
输入：num1 = "456", num2 = "77"
输出："533"
```

**示例 3：**

```
输入：num1 = "0", num2 = "0"
输出："0"
```





**提示：**

- `1 <= num1.length, num2.length <= 104`
- `num1` 和`num2` 都只包含数字 `0-9`
- `num1` 和`num2` 都不包含任何前导零

Related Topics

- 数学

- 字符串

- 模拟

```java
public class AddStrings {
    public static String addStrings(String num1, String num2) {

        //结果
        StringBuffer buffer = new StringBuffer();

        //遍历字符串的初始位置
        int i = num1.length() - 1;
        int j = num2.length() - 1;

        //进位
        int carry = 0;

        //从右侧开始遍历(个位)  空位补零
        while (i >= 0 || j >= 0 || carry != 0){
            int n1 = i >= 0 ? num1.charAt(i) - '0' : 0;
            int n2 = j >= 0 ? num2.charAt(j) - '0' : 0;

            //相同位置相加
            int sum = n1 + n2 + carry;

            buffer.append(sum % 10);
            carry = sum / 10;

            i--;
            j--;
        }

        return buffer.reverse().toString();
    }

    public static void main(String[] args) {
        String num1 = "11", num2 = "123";
        String s = addStrings(num1, num2);
        System.out.println(s);
    }
}
```

