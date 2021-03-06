---
title: 快速幂取模算法
---

![题头](https://raw.githubusercontent.com/Desgard/algo/img/img/part2/ch01/2-quick-pow-mod/title.png)

上一篇文章我们讲了如何将幂运算优化到 {{< katex >}}O(logN){{< /katex >}} 的方法。这一篇来研究一下，快速幂算法与取模运算是如何结合的。

## 取余和取模

首先我们要知道在编程语言中有 `%` 这么一个操作符，在各大编程书中称之为“取余运算”。在程序设计和抽象数学领域，我们管这个操作叫做：**取模运算**。

下面以整型 `a` 和 `b` 两个变量举例，在我们常说的取余运算中，其算法会分成以下的两个步骤：

1. 求整数商：`c = a / b`
2. 求余数：`r = a - c * b`

取模与取余不同在于第一步，求余在整除的时候是往 0 方向逼近，而取模是往负无穷方向逼急。这样就造成了在对负数进行取模运算的差异。所以在计算 `-7 % 4` 的时候，求余运算的结果是 `-3`，而取模运算是 `1`。

```
/// 第一步：求整数商
// 求余运算
-7 / 4 = -1 (逼近 0 )

// 求模运算
-7 / 4 = -2 (逼近负无穷)

/// 第二步：
// 求余运算
-7 - -1 * 4 = -7 + 4 = -3

// 求模运算
-7 - -2 * 4 = -7 + 8 = 1
```

当然，逼近方向其实不影响其他的运算规律，因为任意的 `a % b`，对于给定的 `a` 和 `b` ，计算结果一定是一个确定的值。所以我们只需了解这两种计算的区别就可以。

## 不同语言的 `%` 运算符表现

### **Python**

`%` 在不同的语言中，可能会表现出不同的运算。例如在 python 中，`%` 是取模运算，另外整除的表现也是像负无穷无限逼近，以及 `divmod` 也是求模的表现。（以下是使用 iPython 的调试结果）

```shell
In [1]: -7 % 4
Out[1]: 1

In [2]: divmod(-7, 4)
Out[2]: (-2, 1)

In [3]: -7 // 4
Out[3]: -2
```

### **C/C++/Java/Swift**

`%` 在 **C/C++/Java** 中均为**求余运算**的表现。

```cpp
// C++
int main() {
    cout << -7 / 4 << endl; // -1
    cout << -7 % 4 << endl; // -3
    return 0;
}
```

```java
// Java
public class TestMod {
    public static void main(String []args) {
        System.out.println(-7 / 4); // -1
        System.out.println(-7 % 4); // -3
    }
}
```

```swift
// Swift
print(-7 / 4) // -1
print(-7 % 4) // -3
```

只有掌握了在各个语言中的不同表现，我们才能在解题的时候知道具体的运算流程，否则当遇到求模处理时，由于语言的迁移使用而造成了想当然的情况。

## 求模运算的规律

这里我直接给出规律，如果想看证明请自行 Google。

```
(a + b) % p = (a % p + b % p) % p 
(a - b) % p = (a % p - b % p + p) % p 
(a * b) % p = (a % p * b % p) % p 
a ^ b % p = ((a % p)^b) % p 
```

## 结合快速幂算法

终于来到了最关键的地方，结合快速幂算法后会有什么影响呢？

其实在我们日常做题中，你会看到**输出结果对 xxxx 取模**。这种题目可能是有两种考察方向:

1. **在原算法的基础上，多一个取模运算来考察你对取模运算规律的掌握；**
2. **大数据时数据增长太快，64 位甚至 128 位的整形无法表示；**

对应的，我们快速幂的题目就是这样，假设让你求 a 的 b 次方，当 `a = 10` 且 `b = 20` 次方就已经超过了 64 位 `Int` 类型的范围（{{< katex >}}2^{64}{{< /katex >}} 次方约等于 `1.84 * 10^19`）。

所以，接下来我们要把求模运算也加入到快速幂运算中，我们先来观察上一篇文中所使用到的快速幂算法：

```cpp
int qpow(int x, int n) {
    int res = 1;
    while (n) {
        if (n & 1) res *= x; // 关注点 1
        x *= x; // 关注点 2
        n >>= 1;
    }
    return res;
}
```

我们来看关注点 1 和关注点 2 两个地方，分析得到这两个结论：

1. 我们的快速幂算法其实并没有真正的优化乘法效率，而是通过二进制拆分，从而优化了乘法运算的次数，具体的表现就是 `x *= x` 来扩大乘子的基数；
2. 在计算 `res` 的时候，`res *= x` 仍旧是一个累乘的过程，唯一的变化就是 `x` 在由于 `x *= x` 逐渐变化。这两个式子结合起来，其实就是 `res` 不断的去累乘多个 `x` 。

有了这两点分析，我们就可以套用求模运算规律了。

    (a * b) % p = (a % p * b % p) % p 

我们在所有乘法表达式的地方增加求模运算，其实反映出来的结果就是 `res` 不断累乘时候每一项都做一次求模运算。

有着以上思路我们来修改代码：

```cpp
int qpow(int x, int n, int m) {
    int res = 1;
    while (n) {
        if (n & 1) res = res * x % m;
        x = x * x % m;
        n >>= 1;
    }
    return res;
}

int main() {
    cout << qpow(10, 3, 997) << endl; // 3
    cout << qpow(10, 2, 997) << endl; // 100
    return 0;
}
```

好了，有了这两篇文章的知识点，你可以自己来尝试 [[LeetCode-372] 超级次方](https://leetcode-cn.com/classic/problems/super-pow/description/) 这道题目了。