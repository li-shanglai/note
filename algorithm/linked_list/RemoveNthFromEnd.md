# 删除链表的倒数第N个节点

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。



**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/03/remove_ex1.jpg)

```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

**示例 2：**

```
输入：head = [1], n = 1
输出：[]
```

**示例 3：**

```
输入：head = [1,2], n = 1
输出：[1]
```



**提示：**

- 链表中结点的数目为 `sz`
- `1 <= sz <= 30`
- `0 <= Node.val <= 100`
- `1 <= n <= sz`



**进阶：**你能尝试使用一趟扫描实现吗？

Related Topics

- 链表

- 双指针

```java
public class RemoveNthFromEnd {

    //双指针
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode sentinel = new ListNode(-1,head);
        //定义双指针
        ListNode first = sentinel,second = sentinel;

        //first先走n+1步
        for (int i = 0; i < n + 1; i++) {
            first = first.next;
        }

        while (first != null){
            first = first.next;
            second = second.next;
        }

        second.next = second.next.next;
        return sentinel.next;
    }


        //栈
    public ListNode removeNthFromEnd3(ListNode head, int n) {
        ListNode sentinel = new ListNode(-1,head);

        ListNode curr = sentinel;

        Stack<ListNode> stack = new Stack<>();

        while (curr != null){
            stack.push(sentinel);
            curr = curr.next;
        }

        for (int i = 0; i < n; i++) {
            stack.pop();
        }

        stack.peek().next = stack.peek().next.next;
        return sentinel.next;
    }


    public ListNode removeNthFromEnd2(ListNode head, int n) {
        int length = getLength(head);
        ListNode sentinel = new ListNode(-1,head);

        ListNode curr = sentinel;
        for (int i = 0; i < length - n; i++) {
            curr = curr.next;
        }

        curr.next = curr.next.next;
        return sentinel.next;
    }

    //遍历获取链表长度
    public static int getLength(ListNode head){
        int length = 0;
        while (head != null){
            length++;
            head = head.next;
        }
        return length;
    }
}
```

