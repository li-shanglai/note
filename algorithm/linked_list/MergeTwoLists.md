# 合并两个有序链表

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/03/merge_ex1.jpg)

```
输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]
```

**示例 2：**

```
输入：l1 = [], l2 = []
输出：[]
```

**示例 3：**

```
输入：l1 = [], l2 = [0]
输出：[0]
```



**提示：**

- 两个链表的节点数目范围是 `[0, 50]`
- `-100 <= Node.val <= 100`
- `l1` 和 `l2` 均按 **非递减顺序** 排列

Related Topics

- 递归

- 链表

```java
public class MergeTwoLists {
  	
  	//递归
    public static ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        if (list1 == null){
            return list2;
        }

        if (list2 == null){
            return list1;
        }

        if (list1.val <= list2.val){
            list1.next = mergeTwoLists(list1.next,list2);
            return list1;
        }else {
            list2.next = mergeTwoLists(list1,list2.next);
            return list2;
        }
    }
  
  	//迭代
    public static ListNode mergeTwoLists2(ListNode list1, ListNode list2) {
        //定义哨兵节点
        ListNode sentinel = new ListNode(-1);
        //
        ListNode prev = sentinel;

        while (list1 != null && list2 != null){
            //比较头结点大小
            if (list1.val <= list2.val){
                prev.next = list1;
                prev = list1;
                list1 = list1.next;
            }else {
                prev.next = list2;
                prev = list2;
                list2 = list2.next;
            }
        }
        prev.next = (list1 == null) ? list2 : list1;
        return sentinel.next;
    }


    public static void main(String[] args) {

    }
}
```

