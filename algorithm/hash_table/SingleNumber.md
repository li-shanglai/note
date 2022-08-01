# 只出现一次的数字

给定一个**非空**整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

**说明：**

你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

**示例 1:**

```
输入: [2,2,1]
输出: 1
```

**示例 2:**

```
输入: [4,1,2,1,2]
输出: 4
```

Related Topics

- 位运算

- 数组

```java
public class SingleNumber {

    //位运算
    public static int singleNumber(int[] nums) {
        int res = 0;
        for (int num : nums) {
            res ^= num;
        }
        return res;
    }


        //数学方法 set
    public static int singleNumber4(int[] nums) {
        int numsSum = 0;
        int setSum = 0;
        Set<Integer> set = new HashSet<>();
        for (int num : nums) {
            set.add(num);
            numsSum += num;
        }

        for (Integer i : set) {
            setSum += i;
        }

        return setSum * 2 - numsSum;
    }


        //hashmap
    public static int singleNumber3(int[] nums) {
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int num : nums) {
            if (map.containsKey(num)){
                map.remove(num);
            }else {
                map.put(num,1);
            }
        }
        return map.keySet().iterator().next();
    }


        //暴力法
    public static int singleNumber2(int[] nums) {
        List<Integer> list = new ArrayList<>();

        for (Integer num : nums) {
            if (list.contains(num)){
                list.remove(num);
            }else {
                list.add(num);
            }
        }
        return list.get(0);
    }

    public static void main(String[] args) {
        int[] nums = new int[]{4,1,2,1,2};
        int number = singleNumber(nums);
        System.out.println(number);
    }
}
```

数学上异或运算的概念：

 如果对 0 和二进制位做 XOR 运算，得到的仍然是这个二进制位

a⊕0=a

 如果对相同的二进制位做 XOR 运算，返回的结果是 0

a⊕a=0

 XOR 满足交换律和结合律

a⊕b⊕a=(a⊕a)⊕b=0⊕b=b

所以我们只需要将所有的数进行 XOR 操作，就能得到那个唯一的数字。



