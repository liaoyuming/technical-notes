## 海量数据

### 去重
> 10亿int整型数，以及一台可用内存为1GB的机器，时间复杂度要求O(n)，统计只出现一次的数？

1. 位图法
    - 数据结构：使用2个bit位，用 00、01、11 分别表示 0次、1次，大于1次
    - 内存空间：
        1. 一个int整形数占4byte，即32bit
        2. 2^32 bit * 2 = 2^30 byte = 1G
2. 分治法 + hash map
    - 哈希分桶：hash 映射到多个文件
    - 对每个文件构建 hash map， 找出只出现一次的数字

> 1000万字符串，其中有些是重复的，需要把重复的全部去掉，保留没有重复的字符串。请怎么设计和实现？

1. 分治法 + 前缀树
    - 先 hash 映射到多个文件，再分别通过构建 Trie（前缀树）去重

### 排序
> 给定一个文件，里面最多含有n个不重复的正整数，且其中每个数都小于等于n，n=10^7。排序后输出。
1. 位图法
    - 使用位图，之后顺序输出
    - 内存空间：10^7 bit = 1.25 MB
2. 分治法：
    - 多路归并排序
    
### top k 问题
> 提取出访问次数最多的IP
1. 分治法 + hash map

> 有1亿个浮点数，如果找出其中最大的10000个
1. 分治法 + 最大堆
    - 1亿数据分成 100 份，每份数据，通过最大堆获取最大的10000个数据，再从 100*10000 的数据中用最大堆获取最大的10000个数据
    - 内存空间：10^6 * 32 bit = 4 MB
2. 最小堆
    - 读取 10000 数构建大小为10000 的最大堆，遍历后续浮点数，依次比较，重新调整最大堆，知道遍历结束。
    - 时间复杂度：nmlogm
    - 内存空间：10000 * 32 bit = 40 KB

---

### 常用数据结构
1. bloom filter
2. hash map
3. bit map
4. 堆
5. trie 树