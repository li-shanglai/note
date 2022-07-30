# Algorithm

## 算法的特征

一个算法应该具有以下五个重要的特征：

l 有穷性（Finiteness）

算法的有穷性，是指算法必须能在执行有限个步骤之后终止。

l 确切性（Definiteness）

算法的每一步骤必须有确切的定义。

l 输入项（Input）

一个算法有0个或多个输入，以刻画运算对象的初始情况。所谓0个输入，是指算法本身定出了初始条件。

l 输出项（Output）

一个算法有一个或多个输出，以反映对输入数据加工后的结果。

l 可行性（Effectiveness）

算法中执行的任何计算步骤都是可以被分解为基本的可执行的操作步骤，即每个计算步骤都可以在有限时间内完成（也称之为有效性）。

 

## 1.3 算法复杂度

基于算法的有穷性，我们可以知道算法运行消耗的时间不能是无限的。而对于一个问题的处理，可能有多个不同的算法，它们消耗的时间一般是不同的；运行过程中占用的空间资源也是不同的。

这就涉及到对算法的性能考察。主要有两方面：时间和空间。在计算机算法理论中，用时间复杂度和空间复杂度来分别从这两方面衡量算法的性能。

### 1.3.1 时间复杂度（Time Complexity）

算法的时间复杂度，是指执行算法所需要的计算工作量。

一般来说，计算机算法是问题规模n 的函数f(n)，算法的时间复杂度也因此记做：T(n)=Ο(f(n))。

问题的规模n 越大，算法执行的时间的增长率与f(n) 的增长率正相关，称作渐进时间复杂度（Asymptotic Time Complexity）。

### 1.3.2 空间复杂度

算法的空间复杂度，是指算法需要消耗的内存空间。有时候做递归调用，还需要考虑调用栈所占用的空间。

其计算和表示方法与时间复杂度类似，一般都用复杂度的渐近性来表示。同时间复杂度相比，空间复杂度的分析要简单得多。

所以，我们一般对程序复杂度的分析，重点都会放在时间复杂度上。

### 1.3.3 时间复杂度的计算

要想衡量代码的“工作量”，我们需要将每一行代码，拆解成计算机能执行一条条“基本指令”。这样代码的执行时间，就可以用“基本指令”的数量来表示了。

真实的计算机系统里，基本指令包括：

算术指令（加减乘除、取余、向上向下取整）、数据移动指令（装载、存储、赋值）、控制指令（条件或无条件跳转，子程序调用和返回）。

 

我们来看一些具体的代码，分析一下它们的时间复杂度：

l int a = 1； 

简单赋值操作，运行时间 1（1个单位）

l if (a > 1) {} 

简单判断操作、条件跳转，运行时间 1

l for (int i = 0; i < N; i++) {  System.out.println(i); }

有循环，运行时间 1（i赋初值）+ N+1（判断）+N（打印）+N（i自增）= 3N + 2

### 1.3.4 复杂度的大O表示法

比起代码具体运行的时间，我们更关心的是，当它的输入规模增长的时候，它的执行时间我们是否还能够接受。

不同的算法，运行时间随着输入规模 n 的增长速度是不同的。我们可以把代码执行时间，表示成输入规模n的函数T(n)。

<img src="https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/image-20220728152823621.png" alt="image-20220728152823621" style="zoom:67%;" />

 

在算法分析中，一般用大O符号来表示函数的渐进上界。对于给定的函数g(n)，我们用O(g(n))来表示以下函数的集合：

l O(g(n)) = { f(n): 存在正常量 c 和 n0，使对所有 n≥n0 ，有 0≤f(n) ≤ cg(n) }

 

这表示，当数据量达到一定程度时，g(n) 的增长速度不会超过 O(g(n))限定的范围。也就是说，大O表示了函数的“阶数”，阶数越高，增长趋势越大，后期增长越快。

下图画出了常见的算法复杂度：

![img](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image004.jpg)

## 1.4 算法的分类

可以用两种不同的原则，来对算法做一个分类整理：

l 按照应用的目的来划分

搜索算法、排序算法、字符串算法、图算法、最优化算法、数学（数论）算法

l 按照具体实现的策略划分

暴力法、增量法、分治法、贪心、动态规划、回溯、分支限界法

![img](https://lsl-image.oss-cn-beijing.aliyuncs.com/note/images/clip_image006.jpg)

## 1.5 经典算法

在实际应用中，有一些经典算法和策略，都可以作为解决问题的思路：

l 二分查找

l 快速排序、归并排序

l KMP算法

l 快慢指针（双指针法）

l 普利姆（Prim）和 克鲁斯卡尔（Kruskal）算法

l 迪克斯特拉（Dijkstra）算法

l 其它优化算法：模拟退火、蚁群、遗传算法
