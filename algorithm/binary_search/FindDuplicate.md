# 寻找重复数

给定一个包含 `n + 1` 个整数的数组 `nums` ，其数字都在 `[1, n]` 范围内（包括 `1` 和 `n`），可知至少存在一个重复的整数。

假设 `nums` 只有 **一个重复的整数** ，返回 **这个重复的数** 。

你设计的解决方案必须 **不修改** 数组 `nums` 且只用常量级 `O(1)` 的额外空间。



**示例 1：**

```
输入：nums = [1,3,4,2,2]
输出：2
```

**示例 2：**

```
输入：nums = [3,1,3,4,2]
输出：3
```



**提示：**

- `1 <= n <= 105`
- `nums.length == n + 1`
- `1 <= nums[i] <= n`
- `nums` 中 **只有一个整数** 出现 **两次或多次** ，其余整数均只出现 **一次**



**进阶：**

- 如何证明 `nums` 中至少存在一个重复的数字?
- 你可以设计一个线性级时间复杂度 `O(n)` 的解决方案吗？

Related Topics

- 位运算

- 数组

- 双指针

- 二分查找



把nums看成是顺序存储的链表，nums中每个元素的值是下一个链表节点的地址。

那么如果nums有重复值，说明链表**存在环**，本问题就转化为了找链表中环的入口节点，因此可以用**快慢指针**解决。

比如数组

[3，6，1，4，6，6，2]

保存为：

![img](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image002.jpg)

整体思路如下：

l 第一阶段，寻找环中的节点

a)   初始时，都指向链表第一个节点nums[0]；

b)   慢指针每次走一步，快指针走两步；

c)    如果有环，那么快指针一定会再次追上慢指针；相遇时，相遇节点必在环中

l 第二阶段，寻找环的入口节点（重复的地址值）

d)   重新定义两个指针，让before，after分别指向链表开始节点，相遇节点

e)   before与after相遇时，相遇点就是环的入口节点

第二次相遇时，应该有：

慢指针总路程 = 环外0到入口 + 环内入口到相遇点 (可能还有 + 环内m圈)

快指针总路程 = 环外0到入口 + 环内入口到相遇点 + 环内n圈

并且，快指针总路程是慢指针的2倍。所以：

环内n-m圈 = 环外0到入口 + 环内入口到相遇点。

把环内项移到同一边，就有：

**环内相遇点到入口 + 环内n-m-1 圈 = 环外0到入口**

 

这就很清楚了：从环外0开始，和从相遇点开始，走同样多的步数之后，一定可以在入口处相遇。所以第二阶段的相遇点，就是环的入口，也就是重复的元素。

```java
public class FindDuplicate {

    //快慢指针 (双指针)
    public static int findDuplicate(int[] nums) {
        //定义快慢指针
        int slow = 0,fast = 0;
        //环内相遇点
        do {
            slow = nums[slow];
            fast = nums[nums[fast]];
        }while (fast != slow);

        //环入口点
        int before = 0;
        int after = slow;

        //入口点即为相同元素
        while (before != after){
            before = nums[before];
            after = nums[after];
        }
        return before;
    }

    //HashMap
    //时间 O(n)  空间 O(n)
    public static int findDuplicate5(int[] nums) {

        Map<Integer,Integer> map = new HashMap<>();
        for (int num : nums) {
            if (map.containsKey(num)){
                return num;
            }else {
                map.put(num,1);
            }
        }
        return -1;
    }

    //HashSet
    public static int findDuplicate2(int[] nums) {

        HashSet<Integer> set = new HashSet<>();
        for (int num : nums) {
            if (set.contains(num)){
                return num;
            }else {
                set.add(num);
            }
        }
        return -1;
    }

    //排序
    //nLogn    O(1)
    public static int findDuplicate3(int[] nums) {

        Arrays.sort(nums);

        for (int i = 1; i < nums.length; i++) {
            if (nums[i] == nums[i - 1]){
                return nums[i];
            }
        }
        return -1;
    }


    //二分查找
    //O(nLogn)  O(1)
    public static int findDuplicate4(int[] nums) {

        //左指针
        int left = 1;
        //右指针
        int right = nums.length - 1;

        while (left <= right){
            int mid = (left + right) / 2;
            int count = 0;
            for (int i = 0; i < nums.length; i++) {
                if (nums[i] <= mid){
                    count++;
                }
            }
            if (count <= mid){
                left = mid + 1;
            }else {
                right = mid;
            }
            if (left == right){
                return left;
            }
        }
        return -1;
    }

    public static void main(String[] args) {
        int[] nums = {1,3,4,2,2};
        int duplicate = findDuplicate(nums);
        System.out.println(duplicate);
    }
}
```

