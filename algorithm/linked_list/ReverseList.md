# 反转链表

给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/02/19/rev1ex1.jpg)

```
输入：head = [1,2,3,4,5]
输出：[5,4,3,2,1]
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/02/19/rev1ex2.jpg)

```
输入：head = [1,2]
输出：[2,1]
```

**示例 3：**

```
输入：head = []
输出：[]
```



**提示：**

- 链表中节点的数目范围是 `[0, 5000]`
- `-5000 <= Node.val <= 5000`



**进阶：**链表可以选用迭代或递归方式完成反转。你能否用两种方法解决这道题？

Related Topics

- 递归

- 链表



### 分析

链表的节点结构ListNode已经定义好，我们发现，反转链表的过程，其实跟val没有关系，只要把每个节点的next指向之前的节点就可以了。

从代码实现上看，可以有迭代和递归两种形式。

### 方法一：迭代

假设存在链表 1→2→3→null，我们想要把它改成null←1←2←3。

我们只需要依次迭代节点遍历链表，在迭代过程中，将当前节点的 next 指针改为指向前一个元素就可以了。

### 方法二：递归

递归的核心，在于当前只考虑一个节点。剩下部分可以递归调用，直接返回一个反转好的链表，然后只要把当前节点再接上去就可以了。

假设链表为（长度为m）：

n1 → n2 → …→nk−1 →nk →nk+1 →…→nm → null

若我们遍历到了nk，那么认为剩余节点nk+1到nm 已经被反转。

n1 → n2 → …→nk−1 →nk → nk+1 ←…← nm    

我们现在希望 nk+1 的下一个节点指向 nk，所以，应该有

nk+1.next = nk 

```java
public class ReverseList {

    //递归
    public static ListNode reverseList(ListNode head) {
        if (head == null || head.next == null){
            return head;
        }

        ListNode resHead = head.next;
        ListNode reverseList = reverseList(resHead);

        resHead.next = head;
        head.next = null;

        return reverseList;
    }

    //迭代
    public static ListNode reverseList2(ListNode head) {

        //定义两个指针
        ListNode curr = head;
        ListNode prev = null;

        while (curr != null){
            ListNode temp = curr.next;
            curr.next = prev;
            prev = curr;
            curr = temp;
        }

        return prev;

    }


    public static void main(String[] args) {

        ListNode l1 = new ListNode(1);
        ListNode l2 = new ListNode(2);
        ListNode l3 = new ListNode(3);
        ListNode l4 = new ListNode(4);
        ListNode l5 = new ListNode(5);

        l1.next = l2;
        l2.next = l3;
        l3.next = l4;
        l4.next = l5;
        l5.next = null;

        ListNode listNode = reverseList(l1);
        TestLinkedList.printListNode(listNode);

    }

  //  public class ListNode {
  //      int val;
  //      ListNode next;
  //      ListNode() {
  //
  //      }
  //      ListNode(int val) {
  //          this.val = val;
  //      }
  //      ListNode(int val, ListNode next) {
  //          this.val = val; this.next = next;
  //      }
  //}
}
```

