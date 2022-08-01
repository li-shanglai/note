# LRU缓存机制

LRU（Least recently used，最近最少使用）是一种常用的页面置换算法，选择最近最久未使用的页面予以淘汰。

所谓的“最近最久未使用”，就是根据数据的历史访问记录来判断的，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”。

LRU是最常见的缓存机制，在操作系统的虚拟内存管理中，有非常重要的应用，所以也是面试中的常客。

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220801182234692.png" alt="image-20220801182234692" style="zoom:50%;" />

具体实现上，既然保存的是键值对，而且要根据key来判断数据是否在缓存中，那么就可以用一个**HashMap**来作为缓存的存储数据结构。这样，我们的访问和插入，就都可以以常数时间进行了。

需要额外考虑的是，缓存空间有限，所以这个HashMap要有一个容量限制；而且当达到容量上限时，我们会运用LRU的策略删除最近最少使用的那个数据。

这就要求我们必须把数据，按照一定的线性结构排列起来，最新访问的数据放在后面，新数据的插入可以“顶掉”最前面的不常访问的数据。这种数据结构其实可以用**链表**来实现。

所以，我们最终可以使用一个哈希表+双向链表的数据结构，来实现LRU缓存机制。

![img](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image002.png)





请你设计并实现一个满足 [LRU (最近最少使用) 缓存](https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。



**示例：**

```
输入
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
输出
[null, null, null, 1, null, -1, null, -1, 3, 4]

解释
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
```



**提示：**

- `1 <= capacity <= 3000`
- `0 <= key <= 10000`
- `0 <= value <= 105`
- 最多调用 `2 * 105` 次 `get` 和 `put`

Related Topics

- 设计

- 哈希表

- 链表

- 双向链表

```java
```

