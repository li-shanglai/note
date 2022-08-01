# 最长连续序列

给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 `O(n)` 的算法解决此问题。



**示例 1：**

```
输入：nums = [100,4,200,1,3,2]
输出：4
解释：最长数字连续序列是 [1, 2, 3, 4]。它的长度为 4。
```

**示例 2：**

```
输入：nums = [0,3,7,2,5,8,4,6,0,1]
输出：9
```



**提示：**

- `0 <= nums.length <= 105`
- `-109 <= nums[i] <= 109`

Related Topics

- 并查集

- 数组

- 哈希表

```java
public class LongestConsecutive {

    //hashset优化
    public static int longestConsecutive(int[] nums) {
        Set<Integer> set = new HashSet<>();
        for (int num : nums) {
            set.add(num);
        }
        int maxLen = 0;
        for (int i = 0; i < nums.length; i++) {
            int curr = nums[i];
            int currLen = 1;

            if (!set.contains(curr - 1)){
                while (set.contains(curr + 1)){
                    currLen++;
                    curr++;
                }

                if (currLen > maxLen){
                    maxLen = currLen;
                }
            }

        }

        return maxLen;

    }

    //hashset
    public static int longestConsecutive4(int[] nums) {
        Set<Integer> set = new HashSet<>();
        for (int num : nums) {
            set.add(num);
        }
        int maxLen = 0;
        for (int i = 0; i < nums.length; i++) {
            int curr = nums[i];
            int currLen = 1;
            while (set.contains(curr + 1)){
                currLen++;
                curr++;
            }

            if (currLen > maxLen){
                maxLen = currLen;
            }
        }

        return maxLen;

    }


    //暴力法 循环查找下一个数
    public static int longestConsecutive2(int[] nums) {
        int maxLen = 0;
        for (int i = 0; i < nums.length; i++) {
            int curr = nums[i];
            int currLen = 1;
            while (contains(nums,curr + 1)){
                currLen++;
                curr++;
            }

            if (currLen > maxLen){
                maxLen = currLen;
            }
        }

        return maxLen;
    }

    public static boolean contains(int[] nums,int target){
        for (int num : nums) {

            if (num == target){
                return true;
            }
        }
        return false;
    }


    public static void main(String[] args) {
        int[] nums = new int[]{0,3,7,2,5,8,4,6,0,1};
        int i = longestConsecutive(nums);
        System.out.println(i);
    }
}
```

