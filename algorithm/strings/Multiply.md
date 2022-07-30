# 字符串相乘 #43

给定两个以字符串形式表示的非负整数 `num1` 和 `num2`，返回 `num1` 和 `num2` 的乘积，它们的乘积也表示为字符串形式。

**注意：**不能使用任何内置的 BigInteger 库或直接将输入转换为整数。



**示例 1:**

```
输入: num1 = "2", num2 = "3"
输出: "6"
```

**示例 2:**

```
输入: num1 = "123", num2 = "456"
输出: "56088"
```



**提示：**

- `1 <= num1.length, num2.length <= 200`
- `num1` 和 `num2` 只能由数字组成。
- `num1` 和 `num2` 都不包含任何前导零，除了数字0本身。

Related Topics

- 数学

- 字符串

- 模拟

```java
public class Multiply {

    public static String multiply(String num1, String num2) {

        String res = "0";

        //特殊情况
        if (num1.equals("0") || num2.equals("0")){
            return res;
        }

        //个位遍历每一个与num1相乘
        for (int i = num2.length() - 1; i >= 0 ; i--) {
            int n2 = num2.charAt(i) - '0';

            StringBuffer buffer = new StringBuffer();

            int carry = 0;

            for (int j = 0; j < num2.length() - 1 - i; j++) {
                buffer.append("0");
            }

            for (int j = num1.length() - 1; j >= 0; j--) {
                int n1 = num1.charAt(j) - '0';

                int p = n1 * n2 + carry;

                buffer.append(p % 10);
                carry = p / 10;
            }

            if (carry != 0){
                buffer.append(carry);
            }


            res = AddStrings.addStrings(res, buffer.reverse().toString());
        }


        return res;
    }
  
  
  //
  	public static String multiply(String num1, String num2) {
        //特殊情况
        if (num1.equals("0") || num2.equals("0")){
            return "0";
        }

        int[] arr = new int[num1.length() + num2.length()];

        for (int i = num1.length() - 1; i >= 0; i--) {
            int n1 = num1.charAt(i) - '0';
            for (int j = num2.length() - 1; j >= 0; j--) {
                int n2 = num2.charAt(j) - '0';
                int p = n1 * n2;

                int sum = p + arr[i + j + 1];
                arr[i + j + 1] = sum % 10;
                arr[i + j] += sum / 10;
            }
        }

        StringBuffer res = new StringBuffer();
        int start = arr[0] == 0 ? 1 : 0;
        for (int i = start; i < arr.length; i++) {
            res.append(arr[i]);
        }

        return res.toString();

    }

    public static void main(String[] args) {
        String num1 = "123", num2 = "456";

        System.out.println(multiply(num1, num2));
    }
}
```

