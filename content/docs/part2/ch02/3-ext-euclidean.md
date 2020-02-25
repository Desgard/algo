---
title: 扩展欧几里得算法
---

## 一道头条的笔试题

上个月在脉脉上看到一道头条校招的笔试题，看评论说是“地狱难度”的，我们通过这道题来延伸说一下。先来看下这题的题面：

{{< hint info >}}
有一台用电容组成的计算器，其中每个电容组件都有一个最大容量值（正整数）。

对于单个电容，有如下操作指令：

指令1：放电操作-把该电容当前电量值清零；

指令2：充电操作-把该电容当前电量补充到最大容量值；

指令3：转移操作-从电容 A 中尽可能多的将电量转移到电容 B ，转移不会有电量损失，如果能够充满 B 的最大容量，那剩余的电量仍然会留在 A 中。

现在已知有两个电容，其最大容量分别为 a 和 b，其初始状态都是电量值为 `0`，希望通过一些列的操作可以使其中某个电容（无所谓哪一个）中的电量值等于 `c` （`c`也是正整数），这一些列操作所用的最少指令条数记为 `M`，如果无论如何操作，都不可能完成，则定义此时 `M= 0`。

显然对于每一组确定的 `a`，`b`，`c`，一定会有一个 `M` 与之对应。
{{< /hint >}}

这里需要输入的是 `a`、`b`、`c` ，给出两个样例，例如 `a = 3, b = 4, c = 2` ，则最少需要 `4` 个指令完成。解释：设最大容量为 3 的是 A 号电容，另一个是 B 号电容，对应的操作是 `（充电 A）=> （转移 A -> B） => （充电 A）=> （转移 A -> B）` ，这样 A 就是目标的 `2` 电量。第二个样例 `a = 2, b = 3, c = 4`，由于 a 和 b 都无法到大目标电量 4，所以输出 0 代表无解。

这道题我们拿到以后，第一反应就是模拟三个指令，然后使用 BFS 广度优先搜索来搜出答案，只要任意情况到达目标的 c 值就停下来。但是题目中给出了数据量 {{< katex >}}0 \leq a, b, c \leq 10^9{{< /katex >}} ，这个数据量约束了我们无法使用暴力搜索来求解。

## 简要分析

首先从笔试的角度来分析，由于笔试时会有数据范围的测试，这道题给出的数据范围大概是这样：

```
0 < a, b, c < 10^5 (50%)
0 < a, b, c < 10^7 (30%)
0 < a, b, c < 10^9 (20%)
```

所以如果没有任何的思路和数论基础，我建议使用 BFS 直接写一版暴力，最少可以通过 {{< katex >}}> 50\%{{< /katex >}} 的数据，从而拿到一定的分数。（其实这就是 OI 得分赛制，没有思路先暴力抢分）。

下面我们来分情况讨论这个问题：

**情况一**

样例已经给出了一种边界情况，即当 {{< katex >}}c > max(a, b){{< /katex >}} ，这种情况是无法使得 A 和 B 的电量达到 c 的。直接输出 0。

**情况二**

还有一种我们可以直接想到的情况，当 `a = c` 或者 `b = c` 的时候，只进行一次充电操作就可以完成，直接输出 1。

**情况三**

接下来我们考虑一般情况，即需要满足以下前提条件：

{{< katex display >}}
c < max(a, b) \\ c \neq min(a, b)
{{< /katex >}}

我们将这个问题换一个思路转化一下假设给出的 `a` 、`b`、 `c` 一定有解，那么我们来设置对 A 做了 x 次的充（放）电，对 B 做了 y 次的充（放）电，并且做了 k 次的操作三。如果将 A、B 当做一个大电容来看这个电容只有充放电 a 单位、充放电 b 单位这 4 种操作。那么我们就可以列出一个关系式：

{{< katex display >}}
ax+by=c
{{< /katex >}}

由于 `a`、`b` 为非负整数，又因为前提条件 {{< katex >}}c < max(a, b){{< /katex >}} ，则 `x` 和 `y` 符号相反。

暂且，我们先不管做了几次操作三，先只考虑充放电问题，那其实就是已知 `a`、`b`、`c`，我们在给定范围内求解 `x` 和 `y` 的解就可以了。那么这个问题我们要如何求解呢？这就是**扩展欧几里得算法所要解决的问题**。

## 扩展欧几里得算法（*Extended Euclidean*）

在推导上述问题的求解算法之前，我们需要先了解以下几个概念知识。

### 丢番图方程（Diophantine Equation）

丢番图方程指的是：未知数个数多于方程个数，且未知数只能是整数的整数系数方程或方程组。例如以下式中，{{< katex >}}a, b, c{{< /katex >}} 都为整数:

{{< katex display >}}
a_1x_1^{b_1}+a_2x_2^{b_2}+......+a_nx_n^{b_n}=c
{{< /katex >}}

> 关于代数学鼻祖**丢番图**（**Diophantus**）除了有《算数》这本开山巨作之外，还有一个好玩的数学题目墓志铭，有兴趣可以自己了解。

### 裴蜀定理（Bézout's identity）

在数论中，裴蜀定理是一个关于最大公约数的定理。这个定理说明了对于任意整数 a、b 和他们的最大公约数 `d`，关于未知数 `x` 和 `y` 的线性丢番图方程：

{{< katex display >}}
ax+by=m
{{< /katex >}}

有解，当且仅当 `m` 是 `d` 的倍数时。这个等式也被称为裴蜀等式。

裴蜀等式有解时必然有无穷多个整数解，每组解 `x` 、`y` 都称之为裴蜀数，可用辗转相除法求得。

### 辗转相除法实现扩展欧几里得算法

既然说可以用辗转相除法来解决这个问题，那么我们先来说明一下如何通过辗转相除法来求二元一次线性丢番图方程。

**辗转相除法过程**

以 `23x + 17y = 1` 为例，我们来求 `GCD(23, 17)`：

{{< katex display >}}
23 = 17 \times 1 + 6\\ 17 = 6 \times 2 + 5 \\ 6 = 5 \times 1 + 1 \\ 5= 1 \times 5 + 0 \\1 = 0 \times0+1
{{< /katex >}}

**改写成余数形式**

将等式右边的第一项移项：

{{< katex display >}}
23 \times1+17\times -1=6 \ \ \ \ \ \ (1) \\ 17 \times 1 + 6 \times -2=5 \ \ \ \ \ \ \ \ (2)\\ 6 \times 1 + 5 \times -1 = 1\ \ \ \ \ \ \ \ \ \ (3) 
{{< /katex >}}

**反向带入原式**

带下划线的 `6` 和 `5` 会使用 `(1)` 和 `(2)` 两个式子反向带入，形同换元：

{{< katex display >}}
\begin{aligned} 1 =\  & \underline{6} \times 1 + \underline{5} \times -1 \\ =\ & (1) \times 1 + (2) \times -1 \\ =\ & (23 \times 1 + 17 \times -1) + [17 \times 1 + (23 \times 1 + 17 \times -1)\times -2]\times-1 \\ =\  & 23 \times 3+17 \times -4 \\ =\ & 23x+17y \end{aligned}
{{< /katex >}}

所以反解得，`x = 3, y = -4` 是上述二元一次线性丢番图方程的一组解。

## 扩展欧几里得算法证明

来观察一下辗转相除法的最后两个式子，终止条件是：

$$5= 1 \times 5 + 0 \\ 5=0 \times 5+5$$

当且仅当第二个式子为 `0` 的时候停止这个递归运算。如何延伸到一般情况呢？我们将待求变量设为字母来尝试一下。假设此时，我们要求 `an` 和 `bn` 为系数的二元一次线性丢番图方程的系数，即待求方程：

$$a_nx+b_ny=gcd(a_n, b_n)$$

根据上述的改写余数形式，我们可以列出式一（`|` 是整除的意思）：

$$\begin{aligned}a_0 \times 1 - b_0 \times (a_0|b_0) =a_0\ mod\ b_0\end{aligned}$$

假设未到达最终的终止条件，则有：

{{< katex display >}}
\begin{aligned}a_1 \times 1 - b_1 \times (a_1|b_1) =a_1\ mod\ b_1\end{aligned}
{{< /katex >}}

第二个式子中我们可以发现：

{{< katex display >}}
\begin{aligned}a_1&=b_0 \\ b_1&=a_0 \ mod \ b_0=a_0 \times 1 - b_0 \times (a_0|b_0)\end{aligned}
{{< /katex >}}

同理，第 n 个式子中有：

{{< katex display >}}
\begin{aligned}a_n \times 1 - b_n \times (a_n|b_n) &=a_n\ mod\ b_n\\a_{n}&=b_{n-1} \\ b_{n}&=a_{n-1} \times 1 - b_{n - 1} \times (a_{n-1} | b_{n-1}) = a_{n-1}\ mod \ b_{n -1}\end{aligned}
{{< /katex >}}

根据辗转相除的规则，我们知道第 0 项中 {{< katex >}}b = 0, a = 1{{< /katex >}} ，而我们要求的是第 `n` 项中的 `a` 和 `b`，所以可以通过 `a` 和 `b` 的递推公式逐一推导而来。

**如此我们证明了 `an` 和 `bn` 的递推关系，下面我们来证明 `xn` 的递推关系。**

{{< katex display >}}
\begin{aligned} a_0x_0+b_0y_0&=gcd(a_0, b_0)\\ a_1x_1+b_1y_1&=gcd(a_1, b_1)\end{aligned}
{{< /katex >}}

由上文证得了：

{{< katex display >}}
\begin{aligned}\\a_{1}&=b_{0} \\ b_{1}&=a_{0} \times 1 - b_{0} \times (a_{0} | b_{0}) = a_{0}\ mod \ b_{0}\end{aligned}
{{< /katex >}}

我们将其带入到第一个式子中：

{{< katex display >}}
b_0x_1+(a_0\ mod\ b_0)y_1=gcd(a_1, b_1) 
{{< /katex >}}

{{< katex display >}}
b_0x_1+[a_0\times 1 - b_0\times(a_0|b_0)]y_1=gcd(a_1, b_1)
{{< /katex >}}

所以可以求得：

{{< katex display >}}
a_0y_1-b_0[x_1-y_1(a_0|b_0)]=gcd(a_1, b_1)
{{< /katex >}}

由于辗转相除的推论我们可得：

{{< katex display >}}
gcd(a_1, b_1)=gcd(a_0, b_0)
{{< /katex >}}

所以：

{{< katex display >}}
a_0y_1-b_0[x_1-y_1(a_0|b_0)]=a_0x_0+b_0y_0
{{< /katex >}}

即：

{{< katex display >}}
\begin{aligned}x_0&=y_1 \\ y_0&=x_1-y_1(a_0|b_0)\end{aligned}
{{< /katex >}}

## 代码实现扩展欧几里得算法

为了实现上述的反向带入原式的过程，我们通过递归递归到最深的一层，将每一层的解带入即可完成最终的求解：

```python
# python
def ex_gcd(a, b):
      if b == 0:
          return 1, 0, a
      else:
          x, y, r = ex_gcd(b, a % b) 
          x, y = y, (x - (a // b) * y)
          return x, y, r
```

但是我们注意到，由于裴蜀定理，我们求解的丢番图方程中，等号右边的常数必须是 {{< katex >}}k \times gcd(a, b){{< /katex >}}。所以我们的求解其实是：

{{< katex display >}}
ax+by=gcd(a, b)\times k = c
{{< /katex >}}

所以通过扩展 GCD 算法求得的 `x0` 和 `y0` 这组解，并不是我们要求的最终解。同样的，我们对其扩大 k 倍就是我们想要对结果：

{{< katex display>}}
x =k·x_0= x_0 \times \frac{c}{gcd(a, b)} \\y = k·y_0= y_0 \times \frac{c}{gcd(a, b)}
{{< /katex >}}

## 小结

有了这些知识，你对那道“地狱难度”的头条面试题有没有更多的想法呢？这里有一道 **[LeetCode-365] 水壶问题** 你可以尝试一下，做完之后想必会对扩展 GCD 算法有更深的理解。

至于头条面试题，我将在下一篇文继续讲述并代码实现此题的解法。