---
title: 快速素数筛法
---


本文的内容实用而且简单！素数问题是从来都是数学家热衷探索的领域，也是程序设计竞赛和 LC 中，解决数论相关问题的基础，下面本文介绍如何更科学地筛素数和一些相关的小知识。

首先从定义来说， **素数，指整数在一个大于 1 的自然数中，除了1和此整数自身外，没法被其他自然数整除的数。**那么首先我们可以根据定义来写出我们的最暴力求解素数的程序

## 暴力统计素数

假设有 `n` 个数，我们的方法很简单，判断每个数是否有其他因子，如果有则不是素数，时间复杂度为 **`O(nlogn)`**。

我们以的题目 [《LeetCode-204 计数质数》](https://leetcode-cn.com/problems/count-primes/) 为例，题目描述：

{{< hint info >}}
统计所有小于非负整数 n 的质数的数量。

输入: 10

输出: 4

解释: 小于 10 的质数一共有 4 个, 它们是 2, 3, 5, 7 。
{{< /hint >}}

我们可以写出如下程序：

```python
class Solution:
    def countPrimes(self, n: int) -> int:
        ans = 0
        for i in range (2, n):
            flag = True
            # 只需要检查小于等于sqrt(n)的因数就可以了,因为大于的那部分一定对应着一个小于sqrt(n)的因数
            for j in range(2, int(math.sqrt(i)) + 1):
                if 0 == i % j:
                    flag = False
                    break
            if flag:
                ans += 1
        return ans
```        

功能上来说，我们的已经完全实现对素数进行计数，但是提交结果超出时间限制。所以接下来我们要改进一下算法。

## Eratosthenes 筛法

*Eratosthenes* 筛法进行的是打表，也就是平时说的离线操作，当查询量比较大的时候，我们往往采用这种方法进行离线操作处理；该算法的内容是：首先假设 `n` 个数全部都是素数，然后从 `2` 开始，把**每一个数**的**倍数**都**剔除**并标记成合数（因为合数肯定是有素因子的），这样列表中保存着的都是没有素因子的数，就是我们想要的质数了。

![Eratosthenes 筛法流程](https://raw.githubusercontent.com/Desgard/algo/img/img/part2/ch02/1-eratos-sive/etato-flow.gif)

下面我们对之前的代码进行优化

```python
class Solution:
    def countPrimes(self, n: int) -> int:
        n = max(2, n)   #处理输入数字为0的情况
        is_prime = n * [1] 
        is_prime[0] = is_prime[1] = 0
        
        for i in range(2, n):
            if is_prime[i]:
                for j in range(2, n):
                    if i * j >= n: 
                        break
                    is_prime[i * j] = 0
        return sum(is_prime)
```

显然这道题目比较简单，但是我们还能不能继续对算法进行优化呢？

很明显，很多合数有不止一个素因子，这样上述算法进行了一些重复性的计算，比如对数字 `6` 来说，素因子 `2` 和 `3` 在筛选过程中都对他进行了剔除标记，也就是说，所有 `6` 的倍数，至少都被 `2` 和 `3` 进行了重复的剔除。

## 欧拉筛法 - 线性筛

回忆一下，在我们的暴力算法中，为了简化计算，我们只对小于等于 `sqrt(n)` 的数进行取余检查；这里可以采取类似但是更简洁的办法，只要保证每个合数**只**会被他的**最小素因子**筛掉就可以了，所以我们优化算法的核心：

1. 寻找并保存当前的素数；
2. 对每个数的从小到大的**素数次倍数**进行标记，当发现这个数的素因子后停止（这也就保证每个数都是被**最小素因子**筛掉的）；

我们以 `i = 21` 为例，此时素数表为：`2, 3, 5, 7, 11, 13, 17, 19`

第一轮，标记 `2` 的倍数，`42`；

第二轮，标记 `3` 的倍数，`63`，这时候我们发现 `21` 的最小素因子是 `3`

{{< katex display >}}
21=3^1\times 7^1
{{< /katex >}}

也就是说 `63` **必定**是被 **`21` 的最小素因数 `3`** 来标记的，后边的所有素数次倍数也都**至少可以被 `3` 标记**，就是我们刚才说的重复操作，所以可以选择停止后面的操作节省时间。

这里额外需要一个列表保存已经筛选的素数，下面是我们优化后的代码，时间复杂度为 `O(n)`。

```python
class Solution:
    def countPrimes(self, n: int) -> int:
        n = max(2, n)
        primes = n * [0]
        cnt = 0
        is_prime = n * [1]
        is_prime[0] = is_prime[1] = 0
        
        for i in range(2, n):
            if is_prime[i]: 
                # 保存已经筛出的素数
                primes[cnt] = i 
                cnt += 1
            for j in range(cnt):
                # 如越界则停止
                if primes[j] * i >= n: 
                    break 
                # 标记 i 的素数次倍数
                is_prime[primes[j] * i] = 0
                # 如遇到 i 的素因数，则停止  
                if i % primes[j] == 0: 
                    break 
        return sum(is_prime)
```

## 一个要点

### 这段代码怎么解释？

```python
if i % primes[j] == 0:
    break
```

这句代码保证了每个数最多被筛一次，将时间复杂度降到了线性。证明如下：

因为 `primes[]` 数组中的素数是递增的，当 `i` 能整除 `prime[j]` 的时候，则 `i * prime[j + 1]` 这个合数可能能被 `prime[j]`  乘以某个数筛掉。

因为 `i` 中含有 `prime[j]` 且 `prime[j]` 比 `prime[j + 1]` 小，即 `i = k * prime[j]`  ，那么 `i * prime[j + 1] = (k * prime[j]) * prime[j + 1] = k' * prime[j]`，后面的素数同理。所以不用继续筛下去了。

隐藏，在满足 `i % prime[j] == 0` 这个条件之前以及第一次满足该条件时，`prime[j]` 一定是 `prime[j] * i` 的最小因子。

## 复杂度对比

Eratosthenes 筛法的时间复杂度理论值是 {{< katex >}}O(Nlog(logN)){{< /katex >}} ，而线性筛的理论复杂度是 {{< katex >}}O(N){{< /katex >}} 。可是我通过实际的时间统计，发现 Eratosthenes 筛法更快且更稳定（一脸黑人问号）🌚？如果有读者知道这是怎么一回事，欢迎下方留言。

画图代码：

```python
from matplotlib import pyplot as plt

x = []
y1, y2 = [], []
for i in range(0, 1000000, 50000):
    x.append(i)
    y1.append(eratosthenes_time(i)[1])
    y2.append(euler_time(i)[1])
    
plt.title('Time Complexity Analysis')
plt.plot(x, y1, color='green', label='eratosthenes_sieve')
plt.plot(x, y2, color='red', label='euler_sieve')

plt.xlabel("Parameter Range")
plt.ylabel("Time Complexity(s)")

plt.legend() # 显示图例
```

![时间开销比较](https://raw.githubusercontent.com/Desgard/algo/img/img/part2/ch02/1-eratos-sive/compare-graph.png)

## 总结

所以，我们在求解筛素数表的题目，只要使用 Eratosthenes 筛法就可以解决我们 80% 的问题。这种方法即容易理解，又比传统的暴力筛效率提升很多。