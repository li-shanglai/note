# 二分查找

二分查找也称折半查找（Binary Search），它是一种效率较高的查找方法，前提是数据结构必须先**排好序**，可以在**对数**时间复杂度内完成查找。

 

二分查找事实上采用的就是一种分治策略，它充分利用了元素间的次序关系，可在最坏的情况下用O(log n)完成搜索任务。

它的基本思想是：假设数组元素呈升序排列，将n个元素分成个数大致相同的两半，取a[n/2]与欲查找的x作比较，如果x=a[n/2]则找到x，算法终止；如 果x=a[n/2]，则我们只要在[数组](https://link.zhihu.com/?target=https%3A//baike.baidu.com/item/%E6%95%B0%E7%BB%84)a的左半部继续搜索x；如果x>a[n/2]，则我们只要在数组a的右 半部继续搜索x。

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/%E6%88%AA%E5%B1%8F2022-07-27%2017.00.07.png" alt="截屏2022-07-27 17.00.07" style="zoom:67%;" />

```java
public class BinarySearch {
    public static void main(String[] args) {

        int[] arr = {1,2,3,4,5,6,7,8,9};
        int key = 7;
        int search = binarySearch(arr, key);
        System.out.println(search);

    }

  	双指针实现
    public static int binarySearch(int[] a,int key){
        //定义初始位置 双指针
        int low = 0;
        int high = a.length - 1;

        //排除特殊情况
        if (key < a[low] || key > a[high]){
            return -1;
        }

        while (low <= high){
            int mid = (low + high) / 2;
            if (a[mid] < key){
                low = mid + 1;
            }else if (a[mid] > key){
                high = mid - 1;
            }else {
                return mid;
            }
        }

        return -1;
    }
  
  
  	//递归实现
    public static int binarySearch2(int[] a,int key,int fromIndex,int toIndex){

        if (key < a[fromIndex] || key > a[toIndex] || fromIndex > toIndex){
            return -1;
        }

        int mid = (fromIndex + toIndex) / 2;

        if (a[mid] < key){
            return binarySearch2(a,key,mid + 1,toIndex);
        } else if (key < a[mid]) {
            return binarySearch2(a,key,fromIndex,mid - 1);
        }else {
            return mid;
        }
    }
}
```

