# 去除重复字母

- 给你一个字符串 `s` ，请你去除字符串中重复的字母，使得每个字母只出现一次。需保证 **返回结果的字典序最小**（要求不能打乱其他字符的相对位置）。

  

  **示例 1：**

  ```
  输入：s = "bcabc"
  输出："abc"
  ```

  **示例 2：**

  ```
  输入：s = "cbacdcbc"
  输出："acdb"
  ```

  

  **提示：**

  - `1 <= s.length <= 104`
  - `s` 由小写英文字母组成

  

  **注意：**该题与 1081 https://leetcode-cn.com/problems/smallest-subsequence-of-distinct-characters 相同

  Related Topics

  - 栈

  - 贪心

  - 字符串

  - 单调栈

```java
public class RemoveDuplicateLetters {

    //贪心策略  O(n³)
    public static String removeDuplicateLetters2(String s) {
        if (s.length() == 0){
            return "";
        }

        int position = 0;
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) < s.charAt(position)){
                boolean isRepl = true;
                for (int j = position; j < i; j++) {
                    boolean isDup = false;
                    for (int k = i + 1; k < s.length(); k++) {
                        if (s.charAt(k) == s.charAt(j)){
                            isDup = true;
                            break;
                        }
                    }
                    if (!isDup){
                        isRepl = false;
                        break;
                    }
                }
                if (isRepl){
                    position = i;
                }
            }
        }
        return s.charAt(position) + removeDuplicateLetters(s.substring(position + 1).replaceAll(s.charAt(position) + "",""));
    }

    //时间复杂度  O(N)
    public static String removeDuplicateLetters3(String s) {
        if (s.length() == 0){
            return "";
        }

        int[] count = new int[26];
        int position = 0;

        for (int i = 0; i < s.length(); i++) {
            count[s.charAt(i) - 'a'] ++;
        }

        for (int i = 0; i < s.length(); i++) {

            if (s.charAt(i) < s.charAt(position)){
                position = i;
            }

            if (--count[s.charAt(i) - 'a'] == 0){
                break;
            }
        }
        //递归
        return s.charAt(position) + removeDuplicateLetters(s.substring(position + 1).replaceAll(s.charAt(position) + "",""));
    }

    //栈思想
    public static String removeDuplicateLetters(String s) {
        //字符栈 去重之后的结果
        Stack<Character> stack = new Stack<>();

        HashSet<Character> set = new HashSet<>();

        //保存元素在字符串中出现的最后位置
        HashMap<Character, Integer> map = new HashMap<>();

        for (int i = 0; i < s.length(); i++) {
            map.put(s.charAt(i),i);
        }

        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            if (!set.contains(c)){

                while (!stack.isEmpty() && c < stack.peek() && map.get(stack.peek()) > i){
                    set.remove(stack.pop());
                }

                stack.push(c);
                set.add(c);
            }
        }
        StringBuilder builder = new StringBuilder();
        for (Character character : stack) {
            builder.append(character.charValue());
        }

        return builder.toString();
    }

    public static void main(String[] args) {
        String s = "cbacdcbc";
        System.out.println(removeDuplicateLetters(s));
    }
}
```

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/%E6%88%AA%E5%B1%8F2022-07-28%2022.14.17.png" alt="截屏2022-07-28 22.14.17" style="zoom:67%;" />
