# 下一个排列

整数数组的一个 **排列** 就是将其所有成员以序列或线性顺序排列。

- 例如，`arr = [1,2,3]` ，以下这些都可以视作 `arr` 的排列：`[1,2,3]`、`[1,3,2]`、`[3,1,2]`、`[2,3,1]` 。

整数数组的 **下一个排列** 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 **下一个排列** 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。

- 例如，`arr = [1,2,3]` 的下一个排列是 `[1,3,2]` 。
- 类似地，`arr = [2,3,1]` 的下一个排列是 `[3,1,2]` 。
- 而 `arr = [3,2,1]` 的下一个排列是 `[1,2,3]` ，因为 `[3,2,1]` 不存在一个字典序更大的排列。

给你一个整数数组 `nums` ，找出 `nums` 的下一个排列。

必须**[ 原地 ](https://baike.baidu.com/item/原地算法)**修改，只允许使用额外常数空间。



**示例 1：**

```
输入：nums = [1,2,3]
输出：[1,3,2]
```

**示例 2：**

```
输入：nums = [3,2,1]
输出：[1,2,3]
```

**示例 3：**

```
输入：nums = [1,1,5]
输出：[1,5,1]
```



**提示：**

- `1 <= nums.length <= 100`
- `0 <= nums[i] <= 100`

Related Topics

- 数组

- 双指针

```java

public class NextPermutation {

    public static void nextPermutation(int[] nums) {
        int n = nums.length;
        //从右侧开始遍历
        int k = n - 2;
        //后一个元素小于前一个元素 则一直未下降
        //查找上升点
        while (k >= 0 && nums[k] >= nums[k + 1]){
            k--;
        }
        //当没有上升点时,则原始为最大排列 --> 直接排序反转即可
        if (k == -1){
          	//特殊情况 时间复杂度 O(nLogn)
            Arrays.sort(nums);
            return;
        }

        //查找子序列中比当前值刚好大的元素
        int i = k + 2;
        while (i < n && nums[i] > nums[k]){
            i++;
        }

        //交换
        int temp = nums[k];
        nums[k] = nums[i - 1];
        nums[i - 1] = temp;

        //将剩余元素进行位置对换
        int start = k + 1;
        int end = n - 1;
        while (start < end){
            int tmp = nums[start];
            nums[start] = nums[end];
            nums[end] = tmp;

            start++;
            end--;
        }

    }

    public static void main(String[] args) {
        int[] nums = {5,1,1};
        nextPermutation(nums);
        for (int num : nums) {
            System.out.println(num + "   ");
        }
    }
}
```



<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/%E6%88%AA%E5%B1%8F2022-07-26%2022.56.14.png" alt="截屏2022-07-26 22.56.14" style="zoom:67%;" />
