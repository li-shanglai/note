# 三数之和

给你一个包含 `n` 个整数的数组 `nums`，判断 `nums` 中是否存在三个元素 *a，b，c ，*使得 *a + b + c =* 0 ？请你找出所有和为 `0` 且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。



**示例 1：**

```
输入：nums = [-1,0,1,2,-1,-4]
输出：[[-1,-1,2],[-1,0,1]]
```

**示例 2：**

```
输入：nums = []
输出：[]
```

**示例 3：**

```
输入：nums = [0]
输出：[]
```



**提示：**

- `0 <= nums.length <= 3000`
- `-105 <= nums[i] <= 105`

Related Topics

- 数组

- 双指针

- 排序

```java
public class ThreeSum {

    //双指针    [-1,0,1,2,-1,-4]
    //时间复杂度  O(n²)
    public static List<List<Integer>> threeSum(int[] nums) {

        //先排序   [-4,-1,-1,0,1,2]
        Arrays.sort(nums);
        List<List<Integer>> list = new ArrayList<>();

        int n = nums.length;
        for (int i = 0; i < n; i++) {
            //如果当前元素 >0 直接终止循环
            if (nums[i] > 0){
                break;
            }
            //如果当前元素 = 之前元素 则重复 跳过
            if (i > 0 && nums[i] == nums[i - 1]){
                continue;
            }
            //左指针
            int lp = i + 1;
            //右指针
            int rp = n - 1;

            //左指针 < 右指针
            while (lp < rp){
                int sum = nums[i] + nums[lp] + nums[rp];

                //双指针移动
                if (sum == 0){
                    list.add(Arrays.asList(nums[i],nums[lp],nums[rp]));
                    lp++;
                    rp--;

                    //跳过重复元素
                    while (lp < rp && nums[lp] == nums[lp - 1]){
                        lp++;
                    }
                    while (lp < rp && nums[rp] == nums[rp + 1]){
                        rp--;
                    }

                //增大 则左指针右移
                }else if (sum < 0){
                    lp++;
                //减小 则右指针左移
                }else {
                    rp--;
                }
            }
        }
        return list;
    }

    public static void main(String[] args) {
        int[] nums = {-1,0,1,2,-1,-4};
        int[] nums2 = {-2,0,0,2,2};
        List<List<Integer>> list = threeSum(nums2);
        for (List<Integer> l : list) {
            System.out.println(l.toString());
        }
    }
}
```



