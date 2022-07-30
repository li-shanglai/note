# 滑动窗口最大值

给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

返回 *滑动窗口中的最大值* 。



**示例 1：**

```
输入：nums = [1,3,-1,-3,5,3,6,7], k = 3
输出：[3,3,5,5,6,7]
解释：
滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**示例 2：**

```
输入：nums = [1], k = 1
输出：[1]
```



**提示：**

- `1 <= nums.length <= 105`
- `-104 <= nums[i] <= 104`
- `1 <= k <= nums.length`

Related Topics

- 队列

- 数组

- 滑动窗口

- 单调队列

- 堆（优先队列）

```java
public class MaxSlidingWindow {

    //暴力    O(Nk)
    public static int[] maxSlidingWindow2(int[] nums, int k) {

        //总共有n-k+1个窗口
        int[] res = new int[nums.length - k + 1];

        for (int i = 0; i <= nums.length - k; i++) {
            int max = nums[i];
            for (int j = i + 1; j < i + k; j++) {
                if (nums[j] > max){
                    max = nums[j];
                }
            }
            res[i] = max;
        }
        return res;
    }


    //优先队列 -- 大顶堆 MaxHeap
    //O(NlogK)
    public static int[] maxSlidingWindow3(int[] nums, int k) {
        //总共有n-k+1个窗口
        int[] res = new int[nums.length - k + 1];

        //优先队列
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(k, new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o2 - o1;
            }
        });

        //构建
        for (int i = 0; i < k; i++) {
            maxHeap.add(nums[i]);
        }

        res[0] = maxHeap.peek();

        for (int i = 1; i < nums.length - k + 1; i++) {
            maxHeap.remove(nums[i - 1]);
            maxHeap.add(nums[i + k - 1]);
            res[i] = maxHeap.peek();
        }
        return res;
    }


    //双端队列
    public static int[] maxSlidingWindow4(int[] nums, int k) {
        int[] res = new int[nums.length - k + 1];

        //双向队列
        ArrayDeque<Integer> deque = new ArrayDeque<>();

        //初始化  第一个窗口
        for (int i = 0; i < k; i++) {
            while (!deque.isEmpty() && nums[i] > nums[deque.getLast()]){
                deque.removeLast();
            }
            deque.addLast(i);
        }

        res[0] = nums[deque.getFirst()];

        for (int i = k; i < nums.length; i++) {
            if (!deque.isEmpty() && deque.getFirst() == i - k){
                deque.removeFirst();
            }

            while (!deque.isEmpty() && nums[i] > nums[deque.getLast()]){
                deque.removeLast();
            }
            deque.addLast(i);

            res[i - k + 1] = nums[deque.getFirst()];
        }

        return res;
    }

    //左右扫描
    public static int[] maxSlidingWindow(int[] nums, int k) {
        int n = nums.length;
        int[] res = new int[nums.length - k + 1];
        int[] left = new int[n];
        int[] right = new int[n];

        for (int i = 0; i < n; i++) {
            //从左到右
            if (i % k == 0){
                left[i] = nums[i];
            }else {
                left[i] = Math.max(left[i - 1],nums[i] );
            }

            //从右到左
            int j = n - i - 1;
            if (j % k == k - 1 || j == n - 1){
                right[j] = nums[j];
            }else {
                right[j] = Math.max(right[j + 1],nums[j] );
            }
        }
        for (int i = 0; i < n - k + 1; i++) {
            res[i] = Math.max(right[i],left[i + k - 1] );
        }

        return res;
    }

    public static void main(String[] args) {
        int[] nums = {1,3,-1,-3,5,3,6,7};
        int k = 3;
        int[] res = maxSlidingWindow(nums, k);
        for (int re : res) {
            System.out.print(re + "  ");
        }
    }
}
```

### 左右扫描



算法的主要思想，是将输入数组分割成有 k 个元素的块，然后分别从左右两个方向进行扫描统计块内的最大值，最后进行合并。这里有一些借鉴了分治和动态规划的思想。

 

分块的时候，如果 n % k != 0，则最后一块的元素个数可能更少。

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220728235621914.png" alt="image-20220728235621914" style="zoom:50%;" />

 

开头元素为 `i` ，结尾元素为 `j` 的当前滑动窗口可能在一个块内，也可能在两个块中。

 

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220728235634306.png" alt="image-20220728235634306" style="zoom:50%;" />

 

 

情况 `1` 比较简单。 建立数组 `left`， 其中 `left[j]` 是从块的开始到下标 `j` 最大的元素，方向 `左``->``右`。

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220728235643775.png" alt="image-20220728235643775" style="zoom:50%;" />

 

为了处理更复杂的情况 2，我们需要数组 right，其中 right[j] 是从块的结尾到下标 j 最大的元素，方向 右->左。right 数组和 left 除了方向不同以外基本一致。

![img](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image008.jpg)

 

两数组合在一起，就可以提供相邻两个块内元素的全部信息。

现在我们考虑从下标 i 到下标 j的滑动窗口。 可以发现，这个窗口其实可以堪称两部分：以两块的边界（比如叫做m）为界，从i到m属于第一个块，这部分的最大值，应该从右往左，看right[i]；而从m到j属于第二个块，这部分的最大值应该从左往右，看left[j]。因此合并起来，整个滑动窗口中的最大元素为 max(right[i], left[j])。

![img](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image010.jpg)

 

同样，如果是第一种情形，都在一个块内，用上面的公式也是正确的（这时right[i] = left[j]，都是块内最大值）。

![img](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image012.jpg)

 

这个算法时间复杂度同样是O(N)，优点是不需要使用除数组之外的任何数据结构。
