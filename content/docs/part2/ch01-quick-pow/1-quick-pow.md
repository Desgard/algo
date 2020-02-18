---
title: 加速幂运算
---

幂运算是我们平时写代码的时候最常用的运算之一。根据幂运算的定义我们可以知道，如果我们要求 `x` 的 `N` 次幂，那么想当然的就会写出一个 `N` 次的循环，然后累乘得到结果。所以我们要求幂运算的复杂度仍旧是 {{< katex >}}O_{(N)}{{< /katex >}}?

那么有没有一种更快的方法呢？

这里给出一种在计算机领域常用的快速幂算法，又叫**蒙哥马利幂（Montgomery reduction）算法**，将 {{< katex >}}O_{(N)}{{< /katex >}} 降为 {{< katex >}}O_{(logN)}{{< /katex >}}。

我通过例子来讲解这个优化过程：

假设我们要算 `x` 的 `n` 次幂，使用累乘来编写代码：

```python
res = 1
for i in range(n):
    res *= x
```

好的，我们已经完成了 {{< katex >}}O_{(N)}{{< /katex >}} 的解法。

## 二进制拆分

为了优化这个算法，我们接下来进行数学推导：

我们继续思考当 `N = 10` 这个具体场景，我们可以把 10 写成二进制来表示 `1010(BIN)`，然后我们模拟一次**二进制转十进制的过程（复习一下大学知识）**：

{{< katex display >}}
10 = 2^3 \times \underline{1} + 2^2 \times \underline{0} + 2^1 \times \underline{1} + 2^0 \times \underline{0}
{{< /katex >}}

我用下划线把二进制的 `1010` 标识出来，这样大家就可以发现二进制和十进制转换时的代数式规律。

继续回想刚才的场景，那么我们求 `x` 的 `10` 次幂，则式子我们可以写成这样：

{{< katex display >}}
x^{10} = x^{2^3 \times 1 + 2^2 \times 0 + 2^1 \times 1 + 2^0 \times 0}=x^{2^3 \times 1}\times x^{2^2 \times 0}\times x^{2^1 \times 1}\times x^{2^0 \times 0}
{{< /katex >}}

我们按照二进制低位到高位从左往右交换一下位置：

{{< katex display >}}
x^{10}= (x^{2^0 \times 0})(x^{2^1 \times 1})(x^{2^2 \times 0})(x^{2^3 \times 1})
{{< /katex >}}

我们关注相邻的两项，如果我们不考虑幂指数的 `*0` 和 `*1` ，我们只看前半部分，会发现有这么一个规律：

{{< katex display >}}
\frac{x^{2^k}}{x^{2^{k-1}}}=\frac{x^{2^{k-1}*2}}{x^{2^{k-1}}}=\frac{(x^{2^{k-1}})^2}{x^{2^{k-1}}}=x^{2^{k-1}}
{{< /katex >}}

也就是说，不考虑幂指数的 `*0` 和 `*1` 右式，**左式每次只要每次乘以自身，就是下一项的左式**。在我们的例子中其实就是。

这里我们单独看**第三项和第二项的关系**

{{< katex display >}}
x^{2^{2}}=(x^{2^1})^2 
{{< /katex >}}

用编程思维来考虑这个问题，只要我们从 `x` 开始维护这么一个左式，每一次迭代都执行 `x *= x`，然后每次遇到右边是 `*1` 的情况，就记录一下 `res *= x` 是不是就能模拟咱们二进制拆分的计算思路了呢？

## 编程实现一下 x 的 10 次方

我们用上面的思路，通过代码**来计算一下 2 的 10 次方，答案应该是 1024**。

```cpp
#include <iostream>
using namespace std;

int main() {
    int n = 10; // 幂指数，下面通过二进制拆分成 1010
    int x = 2; // 底数
    int res = 1; // 累乘的答案
    while (n) {
        // 去除二进制的最低位，也就是上面推导中的右式，如果 n & 1 == 1，说明是 *1
        if (n & 1) {
            // 如果是 *1，则根据我们观察出来的规律，对维护的结果做累乘
            res *= x;
        }
        // 转换到下一位
        x *= x;
        // 二进制右移一位，目的是取到下一个低位二进制
        n >>= 1;
    }
    cout << res << endl; // 1024
    return 0;
}
```

是不是发现非常的简单！我们至此已经实现了快速幂算法。我们将  `n`, `x` 做成参数，编写一个快速幂的方法：

```cpp
#include <iostream>
using namespace std;

int qpow(int x, int n) {
    int res = 1;
    while (n) {
        if (n & 1) res *= x;
        x *= x;
        n >>= 1;
    }
    return res;
}

int main() {
    cout << qpow(2, 10) << endl; // 1024
    cout << qpow(4, 2) << endl;  // 16
    cout << qpow(5, 3) << endl;  // 125
    cout << qpow(10, 6) << endl; // 1000000
    return 0;
}
```

## 复杂度

通过上面对幂指数的拆分，发现快速幂只需要循环拆分的项数就可以完成整个幂运算。

我们不妨设求 x 的 N 次方，**并且令 x 的所有二进制位都为 1**，就可以得到下面这个等式：

{{< katex display >}}2^0+2^1+...+2^k=N{{< /katex >}}

那么其实，k 就是计算机需要计算的次数，也就是时间复杂度。套入公比是 1 的等比数列前 k 项和来反推 k 的大小：

{{< katex display >}}
\frac{a_1(1-q^k)}{1-q}=2^k-1=N \\ k=log_2{N-1} \Rightarrow O(logN){{< /katex >}}

好了，这就是快速幂的全部内容了。你可以使用这道题的知识来求解 [LeetCode 372. Super Pow]([https://leetcode-cn.com/classic/problems/super-pow/description/](https://leetcode-cn.com/classic/problems/super-pow/description/)) 。下一篇文我们来手把手 AC 这题。