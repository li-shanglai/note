# 找到字符串中所有字母异位词

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

**异位词** 指由相同字母重排列形成的字符串（包括相同的字符串）。



**示例 1:**

```
输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。
```

**示例 2:**

```
输入: s = "abab", p = "ab"
输出: [0,1,2]
解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的异位词。
```



**提示:**

- `1 <= s.length, p.length <= 3 * 104`
- `s` 和 `p` 仅包含小写字母

Related Topics

- 哈希表

- 字符串

- 滑动窗口

### 分析

“字母异位词”，指“字母相同，但排列不同的字符串”。注意这里所说的“排列不同”，是所有字母异位词彼此之间而言的，并不是说要和目标字符串p不同。

另外，我们同样应该考虑，p中可能有重复字母。

### 方法一：暴力法

最简单的想法，自然还是暴力法。就是直接遍历s中每一个字符，把它当作子串的起始，判断长度为p.length()的子串是否是p的字母异位词就可以了。

考虑到子串和p中都可能有重复字母，我们还是用一个额外的数据结构，来保存每个字母的出现频次。由于本题的字符串限定只包含小写字母，所以我们可以简单地用一个长度为26的int类型数组来表示，每个位置存放的分别是字母a~z的出现个数。

 

代码如下：

```java
//暴力法
    public static List<Integer> findAnagrams2(String s, String p) {

        int[] charCount = new int[26];
        for (int i = 0; i < p.length(); i++) {
            charCount[p.charAt(i) - 'a'] ++;
        }

        ArrayList<Integer> res = new ArrayList<>();

        //遍历s
        for (int i = 0; i <= s.length() - p.length(); i++) {
            boolean isMatch = true;
            int[] scharCount = new int[26];
            for (int j = i; j < i + p.length(); j++) {
                scharCount[s.charAt(j) - 'a'] ++;
                if (scharCount[s.charAt(j) - 'a'] > charCount[s.charAt(j) - 'a']){
                    isMatch = false;
                    break;
                }
            }
            if (isMatch){
                res.add(i);
            }
        }
        return res;
    }
```

 

**复杂度分析**

时间复杂度：O(|s| * |p|)，其中|s|表示s的长度，|p|表示p的长度。时间开销主要来自双层循环，循环的迭代次数分别是(s.length-p.length+1)和 p.length, 所以时间复杂度为O((|s|-|p|+1) * |p|), 去除低阶复杂度，最终的算法复杂度为 O(|s| * |p|)。

空间复杂度：O(1)。需要两个大小为 26 的计数数组，分别保存p和当前子串的字母个数。尽管循环迭代过程中在不断申请新的空间，但是上一次申请的数组空间应该可以得到复用，所以实际上一共花费了2个数组的空间，因为数组大小是常数，所以空间复杂度为O(1)。

### 方法二：滑动窗口（双指针）

暴力法的缺点是显而易见的：时间复杂度较大，运行耗时较长。

我们在暴力求解的时候，其实对于很多字母是做了多次统计的。子串可以看作字符串上开窗截取的结果，自然想到，可以定义左右指针向右移动，实现滑动窗口的作用。在指针移动的过程中，字符只会被遍历一次，时间复杂度就可以大大降低。

 

代码如下：

 ```java
 //滑动窗口  -- 双指针
     public static List<Integer> findAnagrams(String s, String p) {
         int[] charCount = new int[26];
         for (int i = 0; i < p.length(); i++) {
             charCount[p.charAt(i) - 'a'] ++;
         }
 
         ArrayList<Integer> res = new ArrayList<>();
 
         //定义双指针
         int lp = 0, rp = 1;
         int[] scharCount = new int[26];
         while (rp <= s.length()){
             char nc = s.charAt(rp - 1);
             scharCount[nc - 'a']  ++;
             while (scharCount[nc - 'a'] > charCount[nc - 'a'] && lp < rp){
                 char rc = s.charAt(lp);
                 scharCount[rc - 'a']--;
                 lp++;
             }
             if (rp - lp == p.length()){
                 res.add(lp);
             }
             rp++;
         }
         return res;
     }
 ```

 

**复杂度分析**

时间复杂度：O(|s|)。 窗口的左右指针最多都到达 s 串结尾，s 串每个字符最多被左右指针都经过一次，所以时间复杂度为O(|s|)。

空间复杂度：O(1)。只需要两个大小为 26 的计数数组，大小均是确定的常量，所以空间复杂度为O(1)。
