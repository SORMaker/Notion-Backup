# [Lecture 8：Solving Ax b Row Reduced Form R](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/resources/lecture-8-solving-ax-b-row-reduced-form-r/)

## 可解性

这节我们要介绍如何来解 Ax=b ，但是这个方程并不一定有解。

我们还是通过一个例子来说明这个问题：

$$\text{求方程:}\begin{bmatrix}1&2&2&2\\2&4&6&8\\3&6&8&10\end{bmatrix}\begin{bmatrix}x_1\\x_2\\x_3\\x_4\end{bmatrix}=\begin{bmatrix}b_1\\b_2\\b_3\end{bmatrix}\text{ 的可解条件}$$

列增广矩阵，并消元，得到矩阵 U：

$$
\left[\left.A|b\right.\right]=\begin{bmatrix}1&2&2&2&b_1\\2&4&6&8&b_2\\3&6&8&10&b_3\end{bmatrix}=\begin{bmatrix}
\color{red}1 & 2 & 2 & 2 & b_1\\
0 & 0 & \color{red}2 & 4 & b_2 - b_1\\
0 & 0 & 0 & 0& b_3 - b_2 - b_1
\end{bmatrix} = U
$$

观察最后一行，代入方程会得到：$0 = b_3 - b_2 - b_1$ 。这一行方程必须成立。于是我们就得到了，本方程的可解条件为：$0 = b_3 - b_2 - b_1$。

如果从**初等变换的角度**去理解，如果方程组系数矩阵A的行的线性组合可以生成 0 向量，那么相同的组合作用在b的分量上，也必须得到 0。

如果从列的角度去理解，我们可以把方程组看做如下表达：

$x_{1}\begin{bmatrix}1\\2\\3\end{bmatrix}+x_{2}\begin{bmatrix}2\\4\\6\end{bmatrix}+x_{3}\begin{bmatrix}2\\6\\8\end{bmatrix}+x_{4}\begin{bmatrix}2\\8\\10\end{bmatrix}=\begin{bmatrix}b_{1}\\b_{2}\\b_{3}\end{bmatrix}$

如果从列的角度去理解，向量b为系数矩阵A的列的线性组合。

总结一下：
- **向量 b 满足什么条件时，方程 Ax = b 总有解？**
    - 列空间角度：当且仅当 b 属于 A 的列空间 C(A)。
    - 线性组合角度： b 是 A 的列向量的线性组合。
    - 初等变换的角度（消元法的角度）：如果 A 的各行线性组合得到零行，那么对 b 中元素线性组合，也必将得到自然数 0.

> 注：以上这三种说法等价。

## 解的结构

假设 $b=\begin{bmatrix}1\\5\\6\end{bmatrix}$ 满足可解条件，接下来我们求解方程。\
方程的解结构有两部分组成。

通解：通解是满足这个方程的所有解。对于 Ax = b 这个方程，通解 = 矩阵零空间向量 + 特解。其中矩阵零空间为 Ax = 0 的解， 他不会影响等式，而是使我们求出的解更具有普遍意义。

我们已经求得矩阵的零空间向量为$x=c_1\begin{bmatrix}-2\\1\\0\\0\end{bmatrix}+c_2\begin{bmatrix}2\\0\\-2\\1\end{bmatrix}$。

这里我们只需要求得特解，特解为 Ax = b 行最简形式自由变量全部为0时的解。

我们让$x_2,x_4 = 0$，回代方程得到：

$$\left\{ \begin{array}{c}x_1+2x_3=1\\2x_3=3\end{array} \right. $$

得到特解为：$x_p = \begin{bmatrix}-2\\0\\3/2\\0\end{bmatrix}$

所以最后的结果为：特解 + 矩阵零空间向量

$x_p + x_n = \begin{bmatrix}-2\\0\\3/2\\0\end{bmatrix}+c_{1}\begin{bmatrix}-2\\1\\0\\0\end{bmatrix}+c_{2}\begin{bmatrix}2\\0\\-2\\1\end{bmatrix}$

集合解释为：$\mathbb{R}^4$上的一个二维平面，很显然这个解集无法构成一个向量空间，因为解集中连零向量都没有。也可以理解为解集在空间中表现为$\mathbb{R}^4$中的一个不过原点的平面。

总结一下：

- 方程 Ax = b 的通解 **$x = x_{特解} + x_{零空间向量}$**
    - 特解$x_p$：首先令所有的自由变量为0，然后求解主变量，主变量和自由变量组合得到了特解 $x_p$。
    - 零空间向量 $x_n$: 求解方程 Ax = 0 即可得到$x_n$。
    - $x = x_p + x_n$

> 证明：$Ax_p = b,\ Ax_n = 0 \rarr A(x_p + x_n) = 0$
对于 Ax=b 这个方程， **通解 = 矩阵零空间向量 + 矩阵特解** 。这很好理解，矩阵零空间向量代入方程最后结果等于 0 ，所以它不会影响等式，而是把方程的解向量扩展到一个类似子空间的空间上。

## 解与秩的关系

考虑秩为 $r$ 的 $m \times n$ 矩阵 $A$，显然 $r \leq m, r \leq n$。

- 当列满秩时 $(r=n)$，消元后矩阵 $R$ 为 $\left[\begin{array}{}I\\0\end{array}\right]$ 的形式，意味着每一列都有主元，那么也就没有自由变量，此时零空间里只有零向量，因此若满足可解性，则解必唯一。
- 当行满秩时 $(r=m)$，消元后矩阵 $R$ 为 $\left[\begin{array}{}I & F\end{array}\right]$ 的形式，意味着没有零行，此时对 $Ax=b$ 没有任何约束，那么必然有解，自由变量有 $n-m$ 个，此时解有无穷多个。
- 当 $m=n=r$ 时，行列皆满秩，消元后为单位阵，不存在自由列，也没有零行对 $b$ 进行约束，因此必有解且解唯一。
- 当不满秩，即 $r < n$ 且 $r < m$ 时，消元后矩阵 $R$ 为 $\left[\begin{array}{}I&F\\0&0\end{array}\right]$ 的形式，此时有两种情况：
    - 无解：零行约束了 $b$，但 $b$ 不满足约束条件。
    - 无穷多个解：$b$ 满足零行约束，解集为：特解 + 零空间任意向量

总结一下：

| $r=m=n$ | $r=n<m$ | $r=m<n$ | $r<m,r<n$ |
| --- | --- | --- | --- |
| $R=I$ | $R=\begin{bmatrix}I\\0\end{bmatrix}$ | $R=\begin{bmatrix}I&F\end{bmatrix}$ | $R=\begin{bmatrix}I&F\\0&0\end{bmatrix}$ |
| $1\ solution$ | $0\ or\ 1\ solution$ | $\infty\ solution$ | $0\ or\ \infty\  solution$ |