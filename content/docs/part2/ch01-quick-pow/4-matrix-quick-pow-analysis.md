---
title: 矩阵的递推关系分析
---

这篇文章可能会有一些难度，但是所有的预备基础都在前三篇文章中：

- 快速幂
- 快速幂取模
- 矩阵快速幂

## 引子

数字是我们在编程中最常接触的元数据。无论是在业务还是刷题，多半部分都是数字的运算，其次是字符串，再次是布尔。

虽然矩阵也是由数字构成的，但是矩阵往往是描述一个多元方程组的元数据。在业务中很少接触方程的运算，所以自然而然的《线性代数》这门重要但又不重要的学科在工作后说忘就忘。

所以，这篇文章可能对于你工作甚至是面试都不一定有直接的收益，如果考虑业务能力和面试 ROI，也就不需要再浪费时间阅读。

## 斐波那契的矩阵快速幂归纳

通过之前的文章，我们已经推出了斐波那契数列通过矩阵来表示的递推公式：

{{< katex display >}}
\begin{bmatrix}f(n)\\f(n-1)\end{bmatrix}=\begin{bmatrix}1&1\\ 1&0\end{bmatrix}\begin{bmatrix}f(n-1)\\f(n-2)\end{bmatrix}
{{< /katex >}}

再来分析一下我们将表达式 `f(n) = f(n - 1) + f(n - 2)` 转化成矩阵形式的递推公式到底目的是什么？为什么只要这么做，就可以带来优化算法时间复杂度的收益？

从这三个点来思考：

1. **关系变量减少**：原来是 {{< katex >}}f(n) = f(n - 1) + f(n - 2){{< /katex >}} ，通过矩阵表示后，降为 {{< katex >}}A(n - 1)·C = A(n){{< /katex >}} 。
2. 恰好通过矩阵形式表示后变成了一个**等比数列**的形式，这样就可以求出通向公式。而通向公式又是一个幂指数的运算，所以我们联想到了快速幂算法。
3. {{< katex >}}N · N{{< /katex >}} 方阵的矩阵乘法，遵循结合律。

## 归纳问题

思考点 3 是矩阵的规律，这个我们就不再细究。但是思考点 1 和 2，可以为我们延伸出以下的几种场景：

### 增加系数

重点在于如果我们遇到一个表达式 {{< katex >}}f(x){{< /katex >}} ，只要我们能得到它的递推公式，将其转换成  {{< katex >}}A(n - 1)·C = A(n){{< /katex >}} 的形式其实就可以沿用斐波那契数列矩阵快速幂的整体思路拉求解。既然这样，我们就再次从斐波那契数列入手，先对 {{< katex >}}f(n - 1){{< /katex >}} 和 {{< katex >}}f(n - 2){{< /katex >}} 加系数：

{{< katex display >}}
f(n)=a\ f(n-1)+b\ f(n-2)
{{< /katex >}}

使用与推导斐波那契矩阵表达式相同的方式，来对上式进行推导：

{{< katex display >}}
\begin{cases}\begin{aligned}& f(n)=&af(n-1)&+bf(n-2) \\ & f(n - 1)= &f(n-1)&\end{aligned} \end{cases}
{{< /katex >}}

{{< katex display >}}
\begin{bmatrix}f(n)\\f(n-1)\end{bmatrix}=\begin{bmatrix}a&b\\ 1&0\end{bmatrix}\begin{bmatrix}f(n-1)\\f(n-2)\end{bmatrix}
{{< /katex >}}

这里给出一道对应的训练习题：[HDU 1757 - A Simple Math Problem]([http://acm.hdu.edu.cn/showproblem.php?pid=1757](http://acm.hdu.edu.cn/showproblem.php?pid=1757))。

### 增加常数

在增加系数的基础上，我们可以继续的增加常数，例如下式：

{{< katex display >}}
f(n)=a\ f(n-1)+b\ f(n-2) + c
{{< /katex >}}

根据上面的推导经验，由于我们的右边要构造成 {{< katex >}}A(n - 1)·C{{< /katex >}} 的结构，为了保证递推关系，在这种情况下可以进行**扩维操作。**

{{< katex display >}}
\begin{cases}\begin{aligned}& f(n)&=&af(n-1)&+&bf(n-2)&+&c \\ & f(n - 1)&=&\ f(n-1) \\ &c&=&&&&c\end{aligned} \end{cases}
{{< /katex >}}

{{< katex display >}}
\begin{bmatrix}f(n)\\f(n-1)\\c\end{bmatrix}=\begin{bmatrix}a&b&1\\ 1&0&0 \\ 0&0&1\end{bmatrix}\begin{bmatrix}f(n-1)\\f(n-2)\\c\end{bmatrix}
{{< /katex >}}

由此，对于增加常数或者增加齐次的项数这种情况，可以使用上述方法，通过扩维来扩展矩阵对多项式的表达。

### 指数变量的处理

其实很多情况下，并不是单纯的 {{< katex >}}f(n){{< /katex >}} 的单一函数，也有可能含有 {{< katex >}}g(n){{< /katex >}} 的形式存在于代数项式中。如果我们想用矩阵来表示递推关系式，**必须要满足 {{< katex >}}g(n){{< /katex >}} 在乘积的情况下，表现出自变量 `n` 自增的情况**。符合这种条件的就是**指数函数**。

例如下式：

{{< katex display >}}
f(n)=a\ c^{n}+b\ f(n-1) + d
{{< /katex >}}

我们将指数函数用 {{< katex >}}g(n){{< /katex >}} 来表示，并且可以发现其中有这么一个规律：

{{< katex display >}}
g(n)=c^n
{{< /katex >}}

$$g(n)=g(n-1) \times g(1) = c^{n-1}c$$

这里 {{< katex >}}g(n){{< /katex >}} 的规律，其实就是我上面所说的对函数的乘积表现为自变量的加和。

所以这里可以如此构造 {{< katex >}}f(n){{< /katex >}} 的矩阵递推式：

{{< katex display >}}
\begin{bmatrix}f(n)\\c^n\\d\end{bmatrix}=\begin{bmatrix}b&ac&1\\ 0&c&0 \\ 0&0&1\end{bmatrix}\begin{bmatrix}f(n-1)\\c^{n-1}\\d\end{bmatrix}
{{< /katex >}}

如此，含有指数函数 {{< katex >}}g(n){{< /katex >}} 为项式的情况我们也可以通过矩阵快速幂来求解。

### 嵌套矩阵

通过上面的总结，其实我们解决的核心问题就是将递推公式转化成矩阵形式即可。继续来发散性思维，如果递推公式中已经有矩阵，那么是不是也可以使用相同的关系式思路来转化问题呢？

答案肯定是可以的。这里给出一道例题 [POJ-3233 Matrix Power Series]([http://poj.org/problem?id=3233](http://poj.org/problem?id=3233))。

题目描述了如果存在下列等式：

{{< katex display >}}
S_k=A+A^1+A^2+...+A^k=\sum^{k}_{i=1}A^i
{{< /katex >}}

其实根据这种求和式子我们可以立马推出：

{{< katex display >}}
S_k=A·S_{k-1} + A
{{< /katex >}}

泛型这是一个 `k` 和 `k - 1` 的递推公式，另外有一个常数项。根据前文的一些推导经验，我们来构造多项式和矩阵表示：

{{< katex display>}}
\begin{cases}\begin{aligned}& S(n)=&A·S(n-1)&+&A \\ & A= &&&A&\end{aligned} \end{cases}
{{< /katex >}}

由于 `S(n)` 和 `A` 都是矩阵，所以前文在构造矩阵的时候，其中的单位 `1` 都要改成单位矩阵 `E`。

{{< katex display>}}
\begin{bmatrix}S(n)\\A\end{bmatrix}=\begin{bmatrix}A&E\\ 0&E\end{bmatrix}\begin{bmatrix}S(n-1)\\A\end{bmatrix}
{{< /katex >}}

如此，矩阵的嵌套问题也就解决了。

## 总结与延伸学习

我们专题性地研究了快速幂算法，直至现在应该将在计算机领域中所有场景的快速幂问题全部覆盖到了。但其中让你受益最大的，仍旧是在数字领域中的**快速幂取模算法**，所以我个人的建议是矩阵场景下无需更多的关注。

这里给大家带来两个延伸学习：

1. 矩阵快速幂中其实还有一个瓶颈你可以继续深入的去研究，那就是**矩阵乘法的效率优化**。在之前的实现中，所有的矩阵乘法都是通过 {{< katex >}}O(n^3){{< /katex >}} 的方式来实现的，这里给你抛出一个有意思的矩阵乘法算法 - [Strassen algorithm]([https://zh.wikipedia.org/wiki/施特拉森演算法](https://zh.wikipedia.org/wiki/%E6%96%BD%E7%89%B9%E6%8B%89%E6%A3%AE%E6%BC%94%E7%AE%97%E6%B3%95))，它可以将矩阵乘法的时间复杂度优化到 {{< katex >}}O(n^{log7}){{< /katex >}} ，**其核心思想是分治**。
2. 在某些场景下的二项式展开也可以利用矩阵来描述递推式，这里给你延伸一下[**帕斯卡恒等式**](**[https://zh.wikipedia.org/wiki/帕斯卡法則](https://zh.wikipedia.org/wiki/%E5%B8%95%E6%96%AF%E5%8D%A1%E6%B3%95%E5%89%87)**)，**在某些二项式展开的情况下，可以快速进行递推，减少运算次数。对应习题是 [HDU-2855](**[http://acm.hdu.edu.cn/showproblem.php?pid=2855](http://acm.hdu.edu.cn/showproblem.php?pid=2855)**)。**

以上就是矩阵快速幂的所有进阶内容了，多谢大家支持。