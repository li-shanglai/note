# 用栈实现队列

请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（`push`、`pop`、`peek`、`empty`）：

实现 `MyQueue` 类：

- `void push(int x)` 将元素 x 推到队列的末尾
- `int pop()` 从队列的开头移除并返回元素
- `int peek()` 返回队列开头的元素
- `boolean empty()` 如果队列为空，返回 `true` ；否则，返回 `false`

**说明：**

- 你 **只能** 使用标准的栈操作 —— 也就是只有 `push to top`, `peek/pop from top`, `size`, 和 `is empty` 操作是合法的。
- 你所使用的语言也许不支持栈。你可以使用 list 或者 deque（双端队列）来模拟一个栈，只要是标准的栈操作即可。



**示例 1：**

```
输入：
["MyQueue", "push", "push", "peek", "pop", "empty"]
[[], [1], [2], [], [], []]
输出：
[null, null, null, 1, 1, false]

解释：
MyQueue myQueue = new MyQueue();
myQueue.push(1); // queue is: [1]
myQueue.push(2); // queue is: [1, 2] (leftmost is front of the queue)
myQueue.peek(); // return 1
myQueue.pop(); // return 1, queue is [2]
myQueue.empty(); // return false
```





**提示：**

- `1 <= x <= 9`
- 最多调用 `100` 次 `push`、`pop`、`peek` 和 `empty`
- 假设所有操作都是有效的 （例如，一个空的队列不会调用 `pop` 或者 `peek` 操作）



**进阶：**

- 你能否实现每个操作均摊时间复杂度为 `O(1)` 的队列？换句话说，执行 `n` 个操作的总时间复杂度为 `O(n)` ，即使其中一个操作可能花费较长时间。

Related Topics

- 栈

- 设计

- 队列



一种直观的思路是，最终的栈里，按照“自顶向下”的顺序保持队列。也就是说，栈顶元素是最先入队的元素，而最新入队的元素要压入栈底。

我们可以用一个栈来存储元素的最终顺序（队列顺序），记作stack1；用另一个进行辅助反转，记作stack2。

最简单的实现，就是直接用stack2，来缓存原始压栈的元素。每次调用push，就把stack1中的元素先全部弹出并压入stack2，然后把新的元素也压入stack2；这样stack2就是完全按照原始顺序入栈的。最后再把stack2中的元素全部弹出并压入stack1，进行反转。

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220802122154312.png" alt="image-20220802122154312" style="zoom:50%;" />

 

```java
public class MyQueue {

    //入队时反转

    Stack<Integer> stack1;
    Stack<Integer> stack2;

    public MyQueue() {
        stack1 = new Stack<>();
        stack2 = new Stack<>();
    }

    public void push(int x) {
        while (!stack1.isEmpty()){
            stack2.push(stack1.pop());
        }
        stack2.push(x);
        while (!stack2.isEmpty()){
            stack1.push(stack2.pop());
        }
    }

    public int pop() {
        return stack1.pop();
    }

    public int peek() {
        return stack1.peek();
    }

    public boolean empty() {
        return stack1.isEmpty();
    }
}
```



可以不要在入队时反转，而是在出队时再做处理。

 

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220802122240519.png" alt="image-20220802122240519" style="zoom:50%;" />

 

执行出队操作时，我们想要弹出的是stack1的栈底元素。所以需要将stack1中所有元素弹出，并压入stack2，然后弹出stack2的栈顶元素。

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220802122249051.png" alt="image-20220802122249051" style="zoom:50%;" />

我们观察可以发现，stack2中的元素，其实就是保持着队列顺序的，所以完全没必要将它们再压回stack1，下次出队时，我们只要直接弹出stack2中的栈顶元素就可以了。

```java
public class MyQueue2 {

    //出队时反转

    Stack<Integer> stack1;
    Stack<Integer> stack2;

    public MyQueue2() {
        stack1 = new Stack<>();
        stack2 = new Stack<>();
    }

    public void push(int x) {
        stack1.push(x);
    }

    public int pop() {
        if (stack2.isEmpty()){
            while (!stack1.isEmpty()){
                stack2.push(stack1.pop());
            }
        }
        return stack2.pop();
    }

    public int peek() {
        if (stack2.isEmpty()){
            while (!stack1.isEmpty()){
                stack2.push(stack1.pop());
            }
        }
        return stack2.peek();
    }

    public boolean empty() {
        return stack1.isEmpty() && stack2.isEmpty();
    }
}
```





***摊还分析***（[Amortized Analysis](http://www.baidu.com/link?url=Nx6Z-cTzQXZd1HfCEt2GttvRXALZ6D-wC5mo3EuK_FupyDWoKTeHUyeF1NI60_GpyIft5pc_MT7xK_F3Qyhq4YT5XOpis9VJdOm_V2_vXaT5bpW611WZUVLkS7wGYeCa)，均摊法），用来评价某个数据结构的一系列操作的平均代价。

对于一连串操作而言，可能某种情况下某个操作的代价特别高，但总体上来看，也并非那么糟糕，可以形象的理解为把高代价的操作“分摊”到其他操作上去了，要求的就是均摊后的平均代价。

摊还分析的核心在于，最坏情况下的操作一旦发生了一次，那么在未来很长一段时间都不会再次发生，这样就会均摊每次操作的代价。

摊还分析与平均复杂度分析的区别在于，平均情况分析是平均所有的输入。而摊还分析是平均操作。在摊还分析中，不涉及概率，并且保证在最坏情况下每一个操作的平均性能。

所以摊还分析，往往会用在某一数据结构的操作分析上。
