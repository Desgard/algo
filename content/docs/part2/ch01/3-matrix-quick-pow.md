---
title: 矩阵快速幂
---

![题头](https://raw.githubusercontent.com/Desgard/algo/img/img/part2/ch01/3-matrix-quick-pow/title.png)

## 回顾

在上一篇文章中，我们对快速幂算法进行了如下的分析：

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

1. 我们的快速幂算法其实并没有真正的优化乘法效率，而是通过二进制拆分，从而优化了乘法运算的次数，具体的表现就是 `x *= x` 来扩大乘子的基数；
2. 在计算 `res` 的时候，`res *= x` 仍旧是一个累乘的过程，唯一的变化就是 `x` 在由于 `x *= x` 逐渐变化。这两个式子结合起来，其实就是 `res` 不断的去累乘多个 `x` 。

其中的关键就是快速幂其实没有**真正优化乘法的效率，而是优化了乘法运算的次数**。

我们换一个角度来想，**如果有这么一种东西，它也支持乘法和幂运算，同样也拥有像数的乘法一样的规律，是不是也可以进行快速幂的优化**？

## 斐波那契，万物之源

看到这个小标题是不是一脸蒙圈？

为什么快速幂会与斐波那契有关？听我来慢慢道来。

我们都知道斐波那契的递推公式：

{{< katex display >}}
Fib(n)=Fib(n - 1)+Fib(n-2)
{{< /katex >}}

所以 `Fib(n)` 和 `Fib(n - 1)` 是存在一定关系的。我们通过构造一个多项式，来找出关系：

{{< katex display >}}
\begin{aligned}
& f(n)&=f(n-1)&+f(n-2) \\ 
& f(n - 1)&= f(n-1)
&\end{aligned}
{{< /katex >}}

我们讲 `Fib(n - 1)` 也写入多项式方程中，目的是为了凑足一个多项式，从而将右式中只含有 `f(n - 1)` 和 `f(n - 2)` 。

我们把右边的 `f(n - 1)` 及 `f(n - 2)` 看做 `x1` 和 `x2` ，左边的 `f(n)` 和 `f(n - 1)` 看做 `y1` 和 `y2` ，得到下式：

{{< katex display >}}
\begin{aligned}& y_1=&x_1&+x_2 \\ & y_2= &x_1&\end{aligned} 
{{< /katex >}}

看到这个你是否想起了有一门叫做《线性代数》的课程，当遇到一个齐次线性方程组时，我们可以通过 **`系数矩阵 * N 维向量`** 来表示，即：

{{< katex display >}}
A·X=B
{{< /katex >}}

这里我们将上式通过矩阵方程来表示：

{{< katex display >}}
\begin{bmatrix}y_1\\y_2\end{bmatrix}=\begin{bmatrix}1&1\\ 1&0\end{bmatrix}\begin{bmatrix}x_1\\x_2\end{bmatrix}
{{< /katex >}}

{{< katex display >}}
\begin{bmatrix}f(n)\\f(n-1)\end{bmatrix}=\begin{bmatrix}1&1\\ 1&0\end{bmatrix}\begin{bmatrix}f(n-1)\\f(n-2)\end{bmatrix}
{{< /katex >}}

设{{< katex >}}\begin{bmatrix}f(n)\\f(n-1)\end{bmatrix}{{< /katex >}}为{{< katex >}}F(n){{< /katex >}} ，则{{< katex >}} F(n)=\begin{bmatrix}1&1\\ 1&0\end{bmatrix}·F(n-1)
{{< /katex >}}

这里我们把矩阵可以当成一个常数来看，其实这就是一个“等比数列”的地推公式，其“公比”就是那个零一矩阵！

所以我们可以得到：

{{< katex display >}}
\begin{bmatrix}f(n)\\f(n-1)\end{bmatrix}=\begin{bmatrix}1&1\\ 1&0\end{bmatrix}^{n-1}\begin{bmatrix}f(1)\\f(0)\end{bmatrix}
{{< /katex >}}

所以最终，我们将其转换成了一个求解矩阵幂运算的通项公式。

## 矩阵乘法实现

以下是矩阵乘法的规律：

{{< katex display>}}
A = \begin{bmatrix}a_{1, 1} & a_{1, 2} & a_{1,3}\\a_{2, 1} & a_{2, 2} & a_{2,3}\end{bmatrix}
{{< /katex >}}

{{< katex display>}}
B = \begin{bmatrix}b_{1, 1} & b_{1, 2} \\b_{2, 1} & b_{2, 2} \\ b_{3, 1} & b_{3, 2}\end{bmatrix}
{{< /katex >}}

{{< katex >}}
C=A·B=\begin{bmatrix}a_{1,1}b_{1, 1}+a_{1, 2}b{_{2, 1}+a_{1,3}b_{3, 1}} & a_{1,1}b_{1, 2}+a_{1, 2}b{_{2, 2}+a_{1,3}b_{3, 2}} \\a_{2,1}b_{1, 1}+a_{2, 2}b{_{2, 1}+a_{2,3}b_{3, 1}} & a_{2,1}b_{1, 2}+a_{2, 2}b{_{2, 2}+a_{2,3}b_{3, 2}} \end{bmatrix}
{{< /katex >}}

根据 `n * m` 和 `m * n` 矩阵的规律，我们来写一个矩阵乘法的实现：

```cpp
#define N 2

struct matrix {
    int m[N][N];
    matrix() {
        memset(m, 0, sizeof(m));
    }
    void prt();
};

matrix operator * (const matrix a, const matrix b) {
    matrix ans;
    for (int i = 0; i < N; ++ i) {
        for (int j = 0; j < N; ++ j) {
            for(int k = 0; k < N; ++ k) {
                ans.m[i][j] += a.m[i][k] * b.m[k][j];
            }
        }
    }
    return ans;
}

// 打印测试代码
void matrix::prt() {
    for (int i = 0; i < N; ++ i) {
        for (int j = 0; j < N; ++ j) {
            cout << this -> m[i][j] << " ";
        }
        cout << endl;
    }
}
```

## 改写快速幂类型

既然我们已经对矩阵的 `matrix` 的结构体做了乘法符号重载，那么我们的快速幂算法实现直接对类型做修改即可：

```cpp
matrix qpow(matrix x, int n) {
    matrix res;
    for (int i = 0; i < N; ++ i) {
        res.m[i][i] = 1;
    }
    while (n) {
        if (n & 1) res = res * x;
        x = x * x;
        n >>= 1;
    }
    return res;
}
```

## 根据 Fib 数列封装

上文我们推导出了斐波那契的矩阵通项公式：

{{< katex display >}}
\begin{bmatrix}f(n)\\f(n-1)\end{bmatrix}=\begin{bmatrix}1&1\\ 1&0\end{bmatrix}^{n-1}\begin{bmatrix}f(1)\\f(0)\end{bmatrix}
{{< /katex >}}

然后我们将 `f(1) = 1` 和 `f(0) = 0` 带入，形成 `base` 矩阵。在对左边的零一矩阵做 `n - 1` 的幂运算，乘以 `base` 矩阵，返回结果矩阵的 `res[0][0]` 就是我们要求的 `Fib[n]`。

```cpp
int fib(int n) {
    matrix a;
    a.m[0][0] = a.m[1][0] = a.m[0][1] = 1;

    matrix base;
    base.m[0][0] = 1;

    matrix ans = qpow(a, n - 1);
    ans = ans * base;

    return ans.m[0][0];
}
```

## 简单的单元测试

我们对矩阵快速幂求解斐波那契数列来做一个简单的单元测试，来查看是否满足斐波那契数列的规律。

```cpp
#include <iostream>

using namespace std;

#define N 2

struct matrix {
    int m[N][N];
    matrix() {
        memset(m, 0, sizeof(m));
    }
    void prt();
};

void matrix::prt() {
    for (int i = 0; i < N; ++ i) {
        for (int j = 0; j < N; ++ j) {
            cout << this -> m[i][j] << " ";
        }
        cout << endl;
    }
}

matrix operator * (const matrix a, const matrix b) {
    matrix ans;
    for (int i = 0; i < N; ++ i) {
        for (int j = 0; j < N; ++ j) {
            for(int k = 0; k < N; ++ k) {
                ans.m[i][j] += a.m[i][k] * b.m[k][j];
            }
        }
    }
    return ans;
}

matrix qpow(matrix x, int n) {
    matrix res;
    for (int i = 0; i < N; ++ i) {
        res.m[i][i] = 1;
    }
    while (n) {
        if (n & 1) res = res * x;
        x = x * x;
        n >>= 1;
    }
    return res;
}

int fib(int n) {
    matrix a;
    a.m[0][0] = a.m[1][0] = a.m[0][1] = 1;

    matrix base;
    base.m[0][0] = 1;

    matrix ans = qpow(a, n - 1);
    ans = ans * base;

    return ans.m[0][0];
}

int main() {
    cout << fib(1) << endl; // 1
    cout << fib(2) << endl; // 1
    cout << fib(3) << endl; // 2
    cout << fib(4) << endl; // 3
    cout << fib(5) << endl; // 5
    cout << fib(6) << endl; // 8
    cout << fib(7) << endl; // 13
}
```

## 结尾

好了这篇文章你已经学会了矩阵快速幂。可能你会觉得很少有场景会使用到这个。这个我说一句实话是这样，只有在一些特殊的递推公式中才能通过矩阵相乘的方式找到通项公式。后面我会总结一下有哪些常见的递推公式可以使用矩阵快速幂来求得通项公式。