---
title: 欧几里得算法
---

什么是欧几里得算法？欧几里得算法是为了解决 GCD 问题，这里的 GCD 是指 Greatest Common Divisor 即最大公约数，而不是 iOS 中的 Grand Central Dispatch 🤣 。所以这篇分享是关于算法的。

## 欧几里得算法（GCD）

求 GCD 在数论中公认的最常用算法即为欧几里得算法，也就是我们在高中时学到的**辗转相除法**。

欧几里得算法的基本原理用一句话就可以说清楚：**两个整数的最大公约数等于其中较小的数和两数的差的最大公约数，即 {{< katex >}}gcd(a, b) = gcd(b, a\ mod\ b){{< /katex >}}**。

为什么可以这么求呢，这里可以简单证明一下：

假设 {{< katex >}}a, b (a > b){{< /katex >}} 两个数的一个公约数是 {{< katex >}}t{{< /katex >}} ，则有

{{< katex display >}}
a=n\times t \\ b=m\times t
{{< /katex >}}

因为 {{< katex >}}a > b{{< /katex >}} ，设 {{< katex >}}a = k × b + r{{< /katex >}} ，即 {{< katex >}}r = a\ mod\ b{{< /katex >}} ，将 {{< katex >}}a,b{{< /katex >}} 代入展开可得：

{{< katex display >}}
a = n \times t = k \times m \times t + r \\ \Rightarrow r = (n - k \times m) \times t
{{< /katex >}}

由于 {{< katex >}}(n-k \times m) \times t{{< /katex >}}一定是整数，所以 `a`、`b` 的公约数 `t` 也是 `r` 的约数。所以如果我们递归的求解 {{< katex >}}a\ mod\ b{{< /katex >}} 也就是 `a % b` ，就可以得到 `a`、`b` 的最大公约数 GCD 了。什么时候递归结束呢？当 `a % b == 0` 的时候，因为在这个过程中，如果 `a % b` 无法求得正整数 `r` 时，则无法继续按照上述规律继续拆分。

```python
# python
def gcd(a, b):
    return a if b == 0 else gcd(b, a % b)
```

```cpp
// C++
int gcd(int a, int b) {
    return b == 0 ? a : gcd(b, a % b);
}
```

这里另外提一句，`a`、`b` 两数的最大公倍数 `LCM(a, b) = a * b / GCD(a, b)` 。这里就不证明了，有兴趣的自己谷歌。