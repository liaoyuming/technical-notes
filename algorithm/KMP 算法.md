看本文内容时，推荐先看下这篇文章[字符串匹配的KMP算法](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)，能帮助有个大致的理解。

### 定义
KMP（Knuth-Morris-Pratt）字符串匹配算法，常用于在一个文本串 S 内查找一个模式串 P 的出现位置，时间复杂度O(m+n)。

假设现在文本串 S 匹配到 i 位置，模式串 P 匹配到 j 位置
- 如果 j = -1 || S[i] == P[j]，则令 i++，j++，继续匹配下一个字符；
- 如果 j != -1 && S[i] != P[j]，则令 i 不变，j = next[j]。此举意味着失配时，模式串 P 相对于文本串 S 向右移动了 j - next [j] 位，即 P 应该跳到 next[j] 的位置进行匹配。

### 求 next 数组

KMP 关键在于求解模式串 P 的 next 数组。 而 next 数组各值的含义是，代表当前字符之前的字符串中，有多大长度的相同前缀后缀

#### 最长相同前缀后缀
如果给定的模式串是：“ABCDABD”，从左至右遍历整个模式串，其各个子串的前缀后缀分别如下表格所示：

![前缀后缀](http://wiki.jikexueyuan.com/project/kmp-algorithm/images/331.jpg)

也就是说，原模式串子串对应的各个前缀后缀的公共元素的最大长度表为

![最大长度表](http://wiki.jikexueyuan.com/project/kmp-algorithm/images/3311.jpg)

#### next 数组
失配时，模式串向右移动的位数为：已匹配字符数 - 失配字符的上一位字符所对应的最大长度值。因此，当匹配到一个字符失配时，其实没必要考虑当前失配的字符，只需查看失配字符的上一位字符对应的最大长度值。

故字符串 “ABCDABD” 的 next 数组如下： 

![next 数组](http://wiki.jikexueyuan.com/project/kmp-algorithm/images/3332.jpg)

不难发现，next 数组相当于找最大对称长度的前缀后缀，然后整体右移一位，初值赋为 -1。

#### next 数组优化

假设现在文本串 S 匹配到 i 位置，模式串 P 匹配到 j 位置。
- 当 P[j] != S[i] 时，原有逻辑是，接下来用 P[ next [j]] 跟 S[i] 匹配
- 如果 P[j] = P[ next[j] ]，必然导致下一步匹配失败，即 P[ next[j] ] != S[i]，所以不能允许 P[j] = P[next[j ]]。
- 因此，如果出现了 P[j] = P[ next[j] ]，则需要再次递归，即令 next[j] = next[ next[j] ]。

模式串 “abab” 求优化后的 next 数组，如下

![next 数组优化](http://wiki.jikexueyuan.com/project/kmp-algorithm/images/3383.jpg)

### 代码实现

```
// 给定一个 haystack 字符串和一个 needle 字符串
// 在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从 0 开始)。
// 如果不存在，则返回  -1。
func kmp(haystack string, needle string) int {
    if len(needle) < 1 {
        return 0
    }
    if len(haystack) < 1 {
        return -1
    }
    next := getNext(needle)
    i :=0
    j := 0
    for i<len(haystack) && j<len(needle) {
        if j == -1 || haystack[i] == needle[j] {
            i++
            j++
        } else {
            // 从 next 获取到右移的位置
            j = next[j]
        }
    }
    // 匹配成功
    if j == len(needle) {
        return i-j
    }
    return -1
}

func getNext(needle string) []int {
    next := make([]int, len(needle))
    next[0] = -1
    k := -1
    j := 0
    for j < len(needle)-1 {
        // needle[k] 前缀
        // needle[j] 后缀
        if k == -1 || needle[k] == needle[j] {
            // 如果 needle 中 [0, k] 的值 和 [j-k, j] 的值依次相同，则 next[j+1] = k+1 ，此为充分必要条件
            k++
            j++
            next[j] = k
            // 如果 needle[j] 和 needle[next[j]] 相同，则取 next[next[j]] 即 next[k]
            // 原因：如果 haystack[i] != needle[j]，若 needle[j] == needle[next[j]]， 则必定 haystack[i] != needle[next[j]]，因此需要避免这种重复匹配
            if needle[j] == needle[k] {
                next[j] = next[k]
            }
        } else {
            // 因为 k < j，所以存在 next[k] = k'，即 needle 中 [0, k'-1] 的值 和 [k-k', k-1] 的值依次相同
            // 因为 next[j] = k 即 [0, k-1] 的值 和 [j-k, j-1] 的值依次相同
            // 又 0 < k' < k
            // 因此，必然有 [0, k'-1] 与 [j-k', j-1] 的值依次相同
            // 如果 needle[k'] 与 needle[j] 相同， 则 [0, k'] 的值 和 [j-k', j] 的值依次相同，也就是 next[j+1] = k'+1
            // 所以，这里可以直接用 next[k] 赋值给 k
            k = next[k]
        }
    }
    return next
}
```

---
参考文献
- [KMP 算法](http://wiki.jikexueyuan.com/project/kmp-algorithm/define.html)