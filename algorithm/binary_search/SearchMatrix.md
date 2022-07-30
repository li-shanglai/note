# 搜索二维矩阵

编写一个高效的算法来判断 `m x n` 矩阵中，是否存在一个目标值。该矩阵具有如下特性：

- 每行中的整数从左到右按升序排列。
- 每行的第一个整数大于前一行的最后一个整数。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/05/mat.jpg)

```
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 3
输出：true
```

**示例 2：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/11/25/mat2.jpg)

```
输入：matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]], target = 13
输出：false
```



**提示：**

- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= m, n <= 100`
- `-104 <= matrix[i][j], target <= 104`

Related Topics

- 数组

- 二分查找

- 矩阵



既然这是一个查找元素的问题，并且数组已经排好序，我们自然可以想到用二分查找是一个高效的查找方式。

输入的 m x n 矩阵可以视为长度为 m x n的有序数组：

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220727175517430.png" alt="image-20220727175517430" style="zoom:80%;" />

行列坐标为(row, col)的元素，展开之后索引下标为idx = row * n + col；反过来，对于一维下标为idx的元素，对应二维数组中的坐标就应该是：

row = idx / n;  col = idx % n;

```java
public class SearchMatrix {

    //二分查找
    //将行列转换为一行数组 再计算对应的行列号
    public static boolean searchMatrix(int[][] matrix, int target) {

        int m = matrix.length;
        if (m == 0){
            return false;
        }
        int n = matrix[0].length;

        //左右指针
        int left = 0;
        int right = m * n - 1;

        while (left <= right){
            //中间位置
            int mid = (left + right) / 2;
            //行列
            int midEle = matrix[mid / n][mid % n];

            if (midEle < target){
                left = mid + 1;
            }else if (midEle > target){
                right = mid - 1;
            }else {
                return true;
            }
        }
        return false;
    }

    public static void main(String[] args) {
        int[][] matrix = {
                {1,3,5,7},
                {10,11,16,20},
                {23,30,34,60}
        };

        int target = 13;

        boolean b = searchMatrix(matrix, target);
        System.out.println(b);
    }

}
```

