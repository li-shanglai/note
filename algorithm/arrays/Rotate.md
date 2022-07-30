# 旋转图像

给定一个 *n* × *n* 的二维矩阵 `matrix` 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在**[ 原地](https://baike.baidu.com/item/原地算法)** 旋转图像，这意味着你需要直接修改输入的二维矩阵。**请不要** 使用另一个矩阵来旋转图像。



**示例 1：**

<img src="https://assets.leetcode.com/uploads/2020/08/28/mat1.jpg" alt="img" style="zoom:67%;" />

```java
输入：matrix = [ [1,2,3],
								[4,5,6],
								[7,8,9]]
输出：[[7,4,1],
			[8,5,2],
			[9,6,3]]
```

**示例 2：**

<img src="https://assets.leetcode.com/uploads/2020/08/28/mat2.jpg" alt="img" style="zoom:67%;" />

```java
输入：matrix = [ [5,1,9,11],
								[2,4,8,10],
								[13,3,6,7],
								[15,14,12,16]]
输出：[[15,13,2,5],
			[14,3,4,1],
    	[12,6,8,9],
    	[16,7,10,11]]
```



**提示：**

- `n == matrix.length == matrix[i].length`
- `1 <= n <= 20`
- `-1000 <= matrix[i][j] <= 1000`



Related Topics

- 数组

- 数学

- 矩阵

```java
public class Rotate {
    //时间复杂度 O(n²)
    public static void rotate(int[][] matrix) {
        int n = matrix.length;

        //转置矩阵      对角线对称位置元素互换
        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }
        //翻转每行
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n / 2; j++) {
                int tmp = matrix[i][j];
                matrix[i][j] = matrix[i][n - j - 1];
                matrix[i][n - j - 1] = tmp;
            }
        }
    }


    public static void main(String[] args) {
        int[][] matrix = {
                {1,2,3},
                {4,5,6},
                {7,8,9}
        };

        int[][] matrix2 = {
                {5,1,9,11},
                {2,4,8,10},
                {13,3,6,7},
                {15,14,12,16}
        };

        rotate(matrix2);

        for (int[] ints : matrix2) {
            for (int anInt : ints) {
                System.out.print(anInt + "   ");
            }
            System.out.println();
        }
    }
}
```

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/%E6%88%AA%E5%B1%8F2022-07-27%2011.20.44.png" alt="截屏2022-07-27 11.20.44" style="zoom:67%;" />

```java
//分治法
    public static void rotate2(int[][] matrix) {
        int n = matrix.length;

        //遍历四分之一矩阵
        for (int i = 0; i < (n + 1) / 2; i++) {
            for (int j = 0; j < n / 2; j++) {
                int tmp = matrix[i][j];
                matrix[i][j] = matrix[n - j - 1][i];
                matrix[n - j - 1][i] = matrix[n - i - 1][n - j - 1];
                matrix[n - i - 1][n - j - 1] = matrix[j][n - i - 1];
                matrix[j][n - i - 1] = tmp;
            }
        }
    }
```

