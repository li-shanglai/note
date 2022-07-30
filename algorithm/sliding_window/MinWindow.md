# 最小覆盖子串

给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。



**注意：**

- 对于 `t` 中重复字符，我们寻找的子字符串中该字符数量必须不少于 `t` 中该字符数量。
- 如果 `s` 中存在这样的子串，我们保证它是唯一的答案。



**示例 1：**

```
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
```

**示例 2：**

```
输入：s = "a", t = "a"
输出："a"
```

**示例 3:**

```
输入: s = "a", t = "aa"
输出: ""
解释: t 中两个字符 'a' 均应包含在 s 的子串中，
因此没有符合条件的子字符串，返回空字符串。
```



**提示：**

- `1 <= s.length, t.length <= 105`
- `s` 和 `t` 由英文字母组成



**进阶：**你能设计一个在 `o(n)` 时间内解决此问题的算法吗？

Related Topics

- 哈希表

- 字符串

- 滑动窗口

```java
public class MinWindow {

    public static String minWindow(String s, String t) {
        String minSubString = "";

        //保存t中字符出现频次
        HashMap<Character, Integer> map = new HashMap<>();

        //在t中遍历
        for (int i = 0; i < t.length(); i++) {
            char c = t.charAt(i);
            int count = map.getOrDefault(c, 0);
            map.put(c,count + 1);
        }

        //定义左右指针
        int start = 0, end = 1;
        HashMap<Character, Integer> sMap = new HashMap<>();

        int cCount = 0;

        while (end <= s.length()){
            char c = s.charAt(end - 1);
            if (map.containsKey(c)){
                sMap.put(c,sMap.getOrDefault(c,0) + 1);

                if (sMap.get(c) <= map.get(c)){
                    cCount++;
                }
            }

            while (cCount == t.length() && start < end){
                if (minSubString.equals("") || end - start < minSubString.length()){
                    minSubString = s.substring(start,end);
                }
                char nc = s.charAt(start);
                if (map.containsKey(nc)){
                    sMap.put(nc,sMap.getOrDefault(nc,0) - 1);
                    if (sMap.get(nc)  < map.get(nc)){
                        cCount--;
                    }
                }
                start++;
            }
            end++;
        }
        return minSubString;
    }


    public static String minWindow4(String s, String t) {
        String minSubString = "";

        //保存t中字符出现频次
        HashMap<Character, Integer> map = new HashMap<>();

        //在t中遍历
        for (int i = 0; i < t.length(); i++) {
            char c = t.charAt(i);
            int count = map.getOrDefault(c, 0);
            map.put(c,count + 1);
        }

        //定义左右指针
        int start = 0, end = 1;
        HashMap<Character, Integer> sMap = new HashMap<>();
        while (end <= s.length()){
            char c = s.charAt(end - 1);
            if (map.containsKey(c)){
                sMap.put(c,sMap.getOrDefault(c,0) + 1);
            }

            while (check(map,sMap) && start < end){
                if (minSubString.equals("") || end - start < minSubString.length()){
                    minSubString = s.substring(start,end);
                }
                char nc = s.charAt(start);
                if (map.containsKey(nc)){
                    sMap.put(nc,sMap.getOrDefault(nc,0) - 1);
                }
                start++;
            }
            end++;
        }

        return minSubString;
    }

    public static String minWindow3(String s, String t) {
        String minSubString = "";

        //保存t中字符出现频次
        HashMap<Character, Integer> map = new HashMap<>();

        //在t中遍历
        for (int i = 0; i < t.length(); i++) {
            char c = t.charAt(i);
            int count = map.getOrDefault(c, 0);
            map.put(c,count + 1);
        }

        //定义左右指针
        int start = 0, end = t.length();

        while (end <= s.length()){
            HashMap<Character, Integer> sMap = new HashMap<>();

            for (int k = start; k < end; k++) {
                char c = s.charAt(k);
                int count = sMap.getOrDefault(c, 0);
                sMap.put(c,count + 1);
            }

            if (check(map,sMap)){
                if (minSubString.equals("") || end - start < minSubString.length()){
                    minSubString = s.substring(start,end);
                }
                start ++;
            }else {
                end++;
            }
        }
        return minSubString;
    }

    //暴力法
    public static String minWindow2(String s, String t) {
        String minSubString = "";

        //保存t中字符出现频次
        HashMap<Character, Integer> map = new HashMap<>();

        //在t中遍历
        for (int i = 0; i < t.length(); i++) {
            char c = t.charAt(i);
            int count = map.getOrDefault(c, 0);
            map.put(c,count + 1);
        }

        //在s中遍历
        for (int i = 0; i < s.length(); i++) {
            for (int j = i + t.length(); j <= s.length(); j++) {
                HashMap<Character, Integer> sMap = new HashMap<>();

                for (int k = i; k < j; k++) {
                    char c = s.charAt(k);
                    int count = sMap.getOrDefault(c, 0);
                    sMap.put(c,count + 1);
                }

                if (check(map,sMap) && (minSubString.equals("") || j - i < minSubString.length())){
                    minSubString = s.substring(i,j);
                }
            }
        }

        return minSubString;
    }

    public static boolean check(HashMap<Character,Integer> tmap,HashMap<Character,Integer> smap){
        for (Character c : tmap.keySet()) {
            if (smap.getOrDefault(c,0) < tmap.get(c)){
                return false;
            }
        }
        return true;
    }


    public static void main(String[] args) {
        String s = "ADOBECODEBANC", t = "ABC";
        String res = minWindow(s, t);
        System.out.println(res);
    }
}
```







### 5.2.2 分析

所谓“子串”，指的是字符串中连续的一部分字符集合。这就要求我们考察的所有字符，应该在同一个“窗口”内，这样的问题非常适合用滑动窗口的思路来解决。

而所谓的“最小子串”，当然就是符合要求的、长度最小的子串了。

另外还有一个小细节：需要找出“包含T所有字符的最小子串”，那T中的字符会不会有重复呢？给出的示例“ABC”没有重复，但提交代码之后会发现，测试用例是包含有重复字符的T的，比如“ABBC”在这种情况下，重复出现的字符“B”在子串中也要重复出现才可以。

### 5.2.3 方法一：暴力法

最简单直接的方法，就是直接枚举出当前字符串所有的子串，然后一一进行比对，选出覆盖T中所有字符的最小的那个。进一步我们发现，其实只需要枚举长度大于等于T的子串就可以了。

 

这里的核心问题是，怎样判断一个子串中包含了T中的所有字符？

如果T中没有重复，那这个非常简单，只要再遍历一遍T，依次检查每个字符是否包含就可以了；但现在T中字符可能重复，如果一个字符“A”重复出现3次，那我们寻找的子串中也必须有3个“A”。

所以我们发现，只要统计出T每个字符出现的次数，然后在子串中进行比对就可以。这可以用一个HashMap来进行存储，当然也可以更简单的只用一个数组来存。

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220729165738397.png" alt="image-20220729165738397" style="zoom:50%;" />

子串S符合要求的条件是：统计T中每个字符出现的次数，全部小于等于在S中出现次数。

 

代码如下：

 

**public class** MinimumWindowSubstring {*
\*   **public** String minWindow(String s, String t) {*
\*     String minSubString = **""**;*
\*     HashMap<Character, Integer> tCharFrequency = **new** HashMap<>();
     *//* *统计**t**中的字符频次
\*     **for** (**int** i = 0; i < t.length(); i++){
       **char** c = t.charAt(i);
       **int** count = tCharFrequency.getOrDefault(c, 0);
       tCharFrequency.put(c, count + 1);
     }
     *//* *遍历每个字符
\*     **for** (**int** i = 0; i < s.length(); i++){*
\*       **for** (**int** j = i + t.length(); j <= s.length(); j++){*
\*         HashMap<Character, Integer> subStrCharFrequency = **new** HashMap<>();*
\*         **for** (**int** k = i; k < j; k++){
           **char** c = s.charAt(k);
           **int** count = subStrCharFrequency.getOrDefault(c, 0);
           subStrCharFrequency.put(c, count + 1);
         }*
\*         **if** (check(tCharFrequency, subStrCharFrequency) && (j - i < minSubString.length() || minSubString.equals(**""**) )){
           minSubString = s.substring(i, j); *
\*         }
       }
     }
     **return** minSubString;
   }
   *//* *定义一个方法，用来检查子串是否符合要求
\*   **public boolean** check( HashMap<Character, Integer> tFreq, HashMap<Character, Integer> subStrFreq ){*
\*     **for** (**char** c: tFreq.keySet()) {
       **if** (subStrFreq.getOrDefault(c, 0) < tFreq.get(c)){
         **return false**;
       }
     }
     **return true**;
   }
 }

 

**复杂度分析**

 

时间复杂度：O(|s|^3)，事实上，应该写作O(|s|^3+|t|)，这里|s|表示字符串s的长度，|t|表示t的长度。我们枚举s所有的子串，之后又要对每一个子串统计字符频次，所以是三重循环，耗费O(|s|^3)。另外还需要遍历t统计字符频次，耗费O(|t|)。t的长度本身要小于s，而且本题的应用场景一般情况是关键字的全文搜索，t相当于关键字，长度应该远小于s，所以可以忽略不计。

空间复杂度：O(C)，这里C表示字符集的大小。我们用到了HashMap来存储S和T的字符频次，而每张哈希表中存储的键值对不会超过字符集的大小。

### 5.2.4 方法二：滑动窗口

暴力法的缺点是显而易见的：时间复杂度过大，超出了运行时间限制。在哪些方面可以优化呢？

仔细观察可以发现，我们在暴力求解的时候，做了很多无用的比对：对于字符串“ADOBECODEBANC”，当找到一个符合条件的子串“ADOBEC”后，我们会继续仍以“A”作为起点扩展这个子串，得到一个符合条件的“ADOBECO”——它肯定符合条件，也肯定比之前的子串长，这其实是完全不必要的。

 

代码实现上，我们可以定义两个指针：指向子串“起始点”的左指针，和指向子串“结束点”的右指针。它们一个固定、另一个移动，彼此交替向右移动，就好像开了一个大小可变的窗口、在不停向右滑动一样，所以这就是非常经典的滑动窗口解决问题的应用场景。所以有时候，滑动窗口也可以归类到双指针法。

 

代码如下：

 

**public** String minWindow(String s, String t) {
   String minSubString = **""**;
   HashMap<Character, Integer> tCharFrequency = **new** HashMap<>();*
\*   **for** (**int** i = 0; i < t.length(); i++){
     **char** c = t.charAt(i);
     **int** count = tCharFrequency.getOrDefault(c, 0);
     tCharFrequency.put(c, count + 1);
   }
   *//* *设置左右指针
\*   **int** lp = 0, rp = t.length();
   **while** ( rp <= s.length() ){*
\*     HashMap<Character, Integer> subStrCharFrequency = **new** HashMap<>();*
\*     **for** (**int** k = lp; k < rp; k++){
       **char** c = s.charAt(k);
       **int** count = subStrCharFrequency.getOrDefault(c, 0);
       subStrCharFrequency.put(c, count + 1);
     }*
\*     **if** ( check(tCharFrequency, subStrCharFrequency) ) {*
\*       **if** ( minSubString.equals(**""**) || rp - lp < minSubString.length() ){
         minSubString = s.substring(lp, rp);
       }
       lp++;
     }
     **else
**       rp++;
   }
   **return** minSubString;
 }

 

**复杂度分析**

时间复杂度：O(|s|^2)，尽管运用双指针遍历字符串，可以做到线性时间O(|s|)，但内部仍需要循环遍历子串，总共消耗O(|s|)*O(|s|)=O(|s|^2)。

空间复杂度：O(C)，这里C表示字符集的大小。同样用到了HashMap来存储S和T的字符频次，而每张哈希表中存储的键值对不会超过字符集的大小。

### 5.2.5 方法三：滑动窗口优化

这里考虑进一步优化：

我们计算子串S的字符频次时，每次都要遍历当前子串，这做了很多重复工作。

其实，每次都只是左指针或右指针做了一次右移，只涉及到**一个字符的增减**。我们不需要重新遍历子串，只要找到移动指针之前的S中，这个字符的频次，然后再加一或者减一就可以了。

具体应该分左指针右移和右指针右移两种情况讨论。

l 左指针i右移（i –> i+1）。这时子串长度减小，减少的一个字符就是s[i]，对应频次应该减一。

l 右指针j右移（j -> j+1）。这时子串长度增加，增加的一个字符就是s[j]，对应频次加1。

 

代码如下：

 

**public** String minWindow(String s, String t) {
   String minSubString = **""**;*
\*   HashMap<Character, Integer> tCharFrequency = **new** HashMap<>();*
\*   **for** (**int** i = 0; i < t.length(); i++){
     **char** c = t.charAt(i);
     **int** count = tCharFrequency.getOrDefault(c, 0);
     tCharFrequency.put(c, count + 1);
   }*
\*   HashMap<Character, Integer> subStrCharFrequency = **new** HashMap<>();*
\*   **int** lp = 0, rp = 1;*
\*   **while** ( rp <= s.length() ){*
\*     **char** newChar = s.charAt(rp - 1);
     **if** ( tCharFrequency.containsKey(newChar) ){
       subStrCharFrequency.put(newChar, subStrCharFrequency.getOrDefault(newChar, 0) + 1);
     }*
\*     **while** ( check(tCharFrequency, subStrCharFrequency) && lp < rp ){*
\*       **if** ( minSubString.equals(**""**) || rp - lp < minSubString.length() ){
         minSubString = s.substring(lp, rp);
       }*
\*       **char** removedChar = s.charAt(lp);
       **if** ( tCharFrequency.containsKey(removedChar) ){
         subStrCharFrequency.put(removedChar, subStrCharFrequency.getOrDefault(removedChar, 0) - 1);
       }
       lp++;*
\*     }
     rp++; *
\*   }
   **return** minSubString;
 }

 

**复杂度分析**

时间复杂度：O(|s|)。尽管有双重循环，但我们可以发现，内外两重循环其实做的只是分别移动左右指针。最坏情况下左右指针对 s 的每个元素各遍历一遍，哈希表中对 s 中的每个元素各插入、删除一次，对 t 中的元素各插入一次。另外，每次调用check方法检查是否可行，会遍历整个 t 的哈希表。

哈希表的大小与字符集的大小有关，设字符集大小为 C，则渐进时间复杂度为 O(C⋅∣s∣+∣t∣)。如果认为t的长度远小于s，可以近似为O(|s|)。

这种解法实现了线性时间运行。

 

空间复杂度：O(C)，这里C表示字符集的大小。同样用到了HashMap来存储S和T的字符频次，而每张哈希表中存储的键值对不会超过字符集的大小。

### 5.2.6 方法四：滑动窗口进一步优化

我们判断S是否满足包含T中所有字符的时候，调用的方法check其实又是一个暴力法：遍历T中所有字符频次，一一比对。上面的复杂度分析也可以看出，遍历s只用了线性时间，但每次都要遍历一遍T的频次哈希表，这就耗费了大量时间。

我们已经知道，每次指针的移动，只涉及到**一个字符的增减**。所以我们其实不需要知道完整的频次HashMap，只要获取改变的这个字符的频次，然后再和T中的频次比较，就可以知道新子串是否符合要求了。

 

代码如下：

 

**public** String minWindow(String s, String t) {*
\*   String minSubString = **""**;*
\*   HashMap<Character, Integer> tCharFrequency = **new** HashMap<>();*
\*   **for** (**int** i = 0; i < t.length(); i++){
     **char** c = t.charAt(i);
     **int** count = tCharFrequency.getOrDefault(c, 0);
     tCharFrequency.put(c, count + 1);
   }*
\*   HashMap<Character, Integer> subStrCharFrequency = **new** HashMap<>();*
\*   **int** lp = 0, rp = 1;*

\*   **int** charCount = 0;
 *
\*   **while** ( rp <= s.length() ){*
\*     **char** newChar = s.charAt(rp - 1);
     **if** ( tCharFrequency.containsKey(newChar) ){
       subStrCharFrequency.put(newChar, subStrCharFrequency.getOrDefault(newChar, 0) + 1);*
\*       **if** ( subStrCharFrequency.get(newChar) <= tCharFrequency.get(newChar) )
         charCount ++;
     }*
\*     **while** ( charCount == t.length() && lp < rp ){*
\*       **if** ( minSubString.equals(**""**) || rp - lp < minSubString.length() ){
         minSubString = s.substring(lp, rp);
       }*
\*       **char** removedChar = s.charAt(lp);
       **if** ( tCharFrequency.containsKey(removedChar) ){
         subStrCharFrequency.put(removedChar, subStrCharFrequency.getOrDefault(removedChar, 0) - 1);*
\*         **if** ( subStrCharFrequency.get(removedChar) < tCharFrequency.get(removedChar) )
           charCount --;
       }
       lp++; *
\*     }

​     rp++; *
\*   }
   **return** minSubString;
 }

 

**复杂度分析**

时间复杂度：O(|s|)。同样，内外双重循环只是移动左右指针遍历了两遍s；而且由于引入了charCount，对子串是否符合条件的判断可以在常数时间内完成，所以整体时间复杂度为O(|s|+|t|)，近似为O(|s|)。

空间复杂度：O(C)，这里C表示字符集的大小。同样用到了HashMap来存储S和T的字符频次，而每张哈希表中存储的键值对不会超过字符集的大小。
