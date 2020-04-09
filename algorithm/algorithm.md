## 算法

掌握算法的唯一方式：多做题

### 排序算法

[推荐文章](https://www.cnblogs.com/onepixel/p/7674659.html)

#### 非线性时间 比较类排序

通过比较来决定元素间的相对次序，由于其时间复杂度不能突破O(nlogn)，因此称为非线性时间比较类排序。

##### 交换排序

- 冒泡排序

- 快速排序

##### 插入排序

- 简单插入排序

- 希尔排序

##### 选择排序

- 简单选择排序

- 堆排序

##### 归并排序

- 二路归并排序

- 多路归并排序

#### 线性时间 非比较类排序
不通过比较来决定元素间的相对次序，它可以突破基于比较排序的时间下界，以线性时间运行，因此称为线性时间非比较类排序。	

- 计数排序

- 基数排序

- 桶排序

#### 外排序 和 内排序

### 查找算法

-  顺序查找

-  二分查找

-  分块查找

- 静态树查找

  - 次优静态树查找

-  二叉查找树

  - 平衡二叉树 (AVL)

  - 红黑树

- 多路查找树

  - B树

  - B+树

-  哈希表查找

### 图

####  遍历

  -  深度优先搜索 DFS Depth-First-Search

  -  广度优先搜索 BFS Breadth-First-Search

####  拓扑排序
  [https://www.jianshu.com/p/b59db381561a	](https://www.jianshu.com/p/b59db381561a)

####  最短路径算法 (Shortest Path Algorithm)

####  最小生成树 (Minimum Spanning Trees MST)

  - 普里姆(Prim)算法

  - Kruskal 算法

### 常用算法

- 分治算法

  - 应用

    - 汉诺塔

    - 二分搜索

    - 快速排序

    - 归并排序

- 动态规划算法

  - 应用

    - 背包问题

    - 最长公共子串问题

- 贪心算法

  - 应用

    - 最小生成树

- 回溯算法

  - 应用

    - n皇后问题

  - 深度优先搜索 DFS Depth-First-Search

- 分支界限算法

  -  广度优先搜索 BFS Breadth-First-Search

- 双指针法

- 字符串匹配算法
  - [KMP 算法](./KMP.md)

- 蓄水池抽样算法

### 性能分析

- 时间复杂度

- 空间复杂度

### 算法题 

使用 △ 标记面试常考值，最大 5

#### 链表 △△△△△

[推荐文章 ](https://juejin.im/post/5c7c71c6f265da2dcb679e85)

常见题：

[链表反转](https://leetcode-cn.com/problems/reverse-linked-list/), [进阶版](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

[有序链表合并](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

[相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/)

[环形链表](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

[旋转链表](https://leetcode-cn.com/problems/rotate-list/)

[复制带随机指针的链表](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)

等等，

leetcode 里链表这一类题基本都要做 [跳转地址](https://leetcode-cn.com/problemset/all/?search=链表)

#### 树 △△△△

常见题：

[二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

[翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)

[二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

[从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

[二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/)

[二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

[最深叶节点的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-deepest-leaves/)

树这类题，偏简单，基本都能往递归的思路去实现，做个二三十道题就差不多了

#### 栈 △△△

  - 单调栈

常见题：

[有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)

[用栈实现队列](https://leetcode-cn.com/problems/implement-queue-using-stacks/)

[最小栈](https://leetcode-cn.com/problems/min-stack/)

[每日温度](https://leetcode-cn.com/problems/daily-temperatures/)

#### 队列 △

常见题：

[用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues/)

#### 堆 △

能手写个堆排序差不多了

#### 贪心 △

例题：

[分发糖果](https://leetcode-cn.com/problems/candy/)

#### 递归 △△△△△

常见题

[爬楼梯](https://leetcode-cn.com/problems/climbing-stairs/)

[斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)

另，多用于回溯算法，分支算法，二叉树、排序等相关题目，是非常基础的算法思想

#### 分治 △△△

分治，即”分而治之”，就是把一个复杂的问题分成两个或更多的相同或相似的子问题，直到最后子问题可以简单的直接求解，原问题的解即子问题的解的合并。

常见题：

归并排序

快速排序

[合并K个排序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

[数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

#### 回溯 △△△

回溯算法实际上一个类似枚举的搜索尝试过程，主要是在搜索尝试过程中寻找问题的解，当发现已不满足求解条件时，就 “回溯” 返回，尝试别的路径。

常见题：

[全排列](https://leetcode-cn.com/problems/permutations/)

[组合总和](https://leetcode-cn.com/problems/combination-sum/)

[子集](https://leetcode-cn.com/problems/subsets/)

[分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning/)

#### 图   

会拓扑排序、dfs、bfs 就行，一般考的比较少

#### 二分查找 △△△△

  - 二叉查找树

常见题：


#### 位运算 △△△

  - 异或

#### 哈希表

#### 字符串

#### 搜索 △△△

  - 深度优先搜索

  - 广度优先搜索

#### 动态规划 △△△

#### 双指针  △△△

#### 滑动窗口  △

#### 位图  △

  - 海量数据去重和排序

#### 外排序  △