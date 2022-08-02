# 栈

### 栈（Stack）

栈（Stack）又名堆栈，它是一种重要的数据结构。从数据结构角度看，栈也是线性表，其特殊性在于栈的基本操作是线性表操作的子集，它是操作受限的线性表，因此，可称为限定性的数据结构。

栈被限定仅在表尾进行插入或删除操作。表尾称为栈顶，相应地，表头称为栈底。所以栈具有“后进先出”（LIFO）的特点。

栈的基本操作除了在栈顶进行插入（入栈，push）和删除（出栈，pop）外，还有栈的初始化，判断是否为空以及取栈顶元素等。

![img](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image001.png)

# 队列

队列（Queue）是一种先进先出（FIFO，First-In-First-Out）的线性表。

在具体应用中通常用链表或者数组来实现。队列只允许在后端（称为 rear）进行插入操作，在前端（称为 front）进行删除操作。

队列的操作方式和堆栈类似，唯一的区别在于队列只允许新数据在后端进行添加。

 

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220802112810368.png" alt="image-20220802112810368" style="zoom:50%;" />

 

l 双端队列 (Deque:double ended queue)

双端队列，是限定插入和删除操作在表的两端进行的[线性表](https://baike.baidu.com/item/线性表/3228081)。

队列的每一端都能够插入[数据项](http://baike.baidu.com/view/178581.htm)和移除数据项。

相对于普通队列，双端队列的入队和出队操作在两端都可进行。所以，双端队列同时具有队列和栈的性质。

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220802112822002.png" alt="image-20220802112822002" style="zoom:50%;" />

l 优先队列

优先队列不再遵循先入先出的原则，而是分为两种情况：

**最大优先队列，无论入队顺序，当前最大的元素优先出队。**

**最小优先队列，无论入队顺序，当前最小的元素优先出队。**

比如有一个最大优先队列，它的最大元素是8，那么虽然元素8并不是队首元素，但出队的时候仍然让元素8首先出队：

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image003.png" alt="img" style="zoom:50%;" />

要满足以上需求，利用线性数据结构并非不能实现，但是时间复杂度较高，需要遍历所有元素，最坏时间复杂度O（n），并不是最理想的方式。

因此，一般是用**大顶堆**（Max Heap，有时也叫最大堆）来实现最大优先队列，每一次入队操作就是堆的插入操作，每一次出队操作就是删除堆顶节点。

**入队操作：**

1. 插入新节点5

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image004.png" alt="img" style="zoom:50%;" />

2. 新节点5上浮到合适位置。

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image005.png" alt="img" style="zoom:50%;" />

**出队操作：**

1. 把原堆顶节点10“出队”

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image006.png" alt="img" style="zoom:50%;" />

2. 最后一个节点1替换到堆顶位置

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image007.png" alt="img" style="zoom:50%;" />

3.节点1下沉，节点9成为新堆顶

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image008.png" alt="img" style="zoom:50%;" />

 

二叉堆节点上浮和下沉，操作次数不会超过数的深度，所以时间复杂度都是O(logn)。那么优先队列，入队和出队的时间复杂度，也是O(logn)。
