# [Lecture 15: Projections onto Subspaces](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/resources/lecture-15-projections-onto-subspaces/)

## 二维空间投影

![2023-09-25-10-59-35.png](../.assert/Linear-Algebra-MIT/Lecture15/2023-09-25-10-59-35.png)

如图所示，在二维空间中，向量$\vec{p}$就是$\vec{b}$在$\vec{a}$上的投影，即 $\vec{p}=x\vec{a}$ ，显然他们的差值是$\vec{e}=\vec{b}−\vec{p}$，$\vec{e}$与$\vec{a}$垂直：

$$
\vec{a}^T\vec{e} = \vec{a}^T(\vec{b}−\vec{p}) = \vec{a}^T(\vec{b}−x\vec{a}) = 0\\
x = \frac{\vec{a}^T\vec{b}}{\vec{a}^T\vec{a}}
$$

代入 $\vec{p}=\vec{x}\vec{a}$ ，得到: $\vec{p} = \frac{\vec{a}^T\vec{b}}{\vec{a}^T\vec{a}}\vec{a}$

可以看到 $p$ 的形式中，分母是一个值，而分子中包含了 $b$ ，这也就表明投影得到的 $p$ 向量是通过前面的系数矩阵作用到 $b$ 上完成的，形式上写作：$\vec{p} = P\vec{b}$ ，其中这个系数矩阵  $P$ 我们称之为投影矩阵，即：$P = \frac{aa^T}{a^Ta}$ 

这个P矩阵有两个重要性质：

1. P是个对称矩阵，因此有 $P^T = P$ 。
2. $P^2 = P$
    1. 由 $P^2 = \frac{a(a^Ta)a^T}{(a^Ta)^2}$，而 $a^Ta$ 是内积，是一个数字，所以化简得到 $P$。
    2. 这一点很容易理解：$P^2$ 相当于对  $b$ 在 $a$ 上投影得到的 $p$ 继续再做一次投影，显然 $p$ 在 $a$ 上的投影是它本身。

<aside>
💡 投影的意义

为什么要做投影呢？回到上一节的 `Ax=b` 问题，我们知道 `Ax` 总是在 `A` 的列空间中，但是 `b` 不一定。

那么退而求其次，什么样的 `x` 可以最逼近 `b` 呢？换言之，就是让 `b` 的变化能够最小，而变化最小恰恰是通过投影来实现（所以 `e` 被视为误差）。

投影得到的 `p` 就是 `A` 列空间中最接近 `b` 的那一个。

</aside>

## 三维空间投影

![2023-09-25-11-42-06.png](../.assert/Linear-Algebra-MIT/Lecture15/2023-09-25-11-42-06.png)

延展到三维也并无二致，如图所示，$a_1$和$a_2$是构成平面的一组基，$p$ 是 $b$ 在平面上的投影，$p$ 可以表示为分别投影在这组基的每个向量上的投影和：$p = \hat{x}_1a_1 + \hat{x}_2a_2$，矩阵形式写作：$p = A\hat{x}$，其中$A = [a_1 \; a_2]$，$\hat{x} = \left[\begin{array}{}\hat{x}_1 \\ \hat{x}_2\end{array}\right]$

此平面即为矩阵 $A$ 的列空间，由于 $b$ 不在平面上，因此$Ax = b$无解，$A\hat{x} = p$中的$\hat{x}$就是最优解。

同样地，有 $e = b - p = b - A\hat{x}$ 与平面垂直($A$的列空间)，因此 
$$A^Te = 0：\left[ \begin{array}{}a_1^T \\ a_2^T\end{array}\right](b - A\hat{x}) = \left[\begin{array}{}0 \\ 0 \end{array}\right]$$

> 这与二维投影的$a^T(b - xa) = 0$并无差别，只不过二维空间的$A^T$只有一列罢了。

> 由$A^Te = 0$可知，由于列空间与左零空间正交，e一定在A的左零空间。

**化简上式，可得：$A^TA\hat{x} = A^Tb$，而这恰恰就是上一节我们谈及为了解决$Ax = b$无解时的工程惯用法式子。至此，我们总算理解了同时左乘$A^T$的物理意义。**

继续求解，当且仅当$A^TA$可逆时，我们可以求出最优解：$\hat{x} = (A^TA)^{-1}A^Tb$。而在上一讲的最后，我们推理出如果A的各列线性无关，那么$A^TA$可逆。

> 相比于求解$Ax = b$，只有当A可逆时，x才有解。而在对b进行了在A列空间的投影后，求解$\hat{x}$的条件得到了适当的放宽，此时只要求A的列向量线性无关即可。在实际工程项目中，我们测量得到的$m \times n$矩阵常常m很大，n很小，因此容易保证。

因此，投影向量$p = A\hat{x} = A(A^TA)^{-1}A^Tb$，投影矩阵为$P=A(A^TA)^{-1}A^T$（二维空间得$aa^T/a^Ta$）。

> 若A本身可逆，投影矩阵$A(A^TA)^{-1}A^T$就可进一步化简，此时$(A^TA)^{-1}$就可以拆成$A^{-1}(A^T)^{-1}$，此时$A(A^TA)^{-1}A^T = AA^{-1}(A^T)^{-1}A^T = I$，即得到单位阵。这一结论显然，因为A若本身可逆，那么$Ax = b$就有解，b本身就在A的列空间，投影b到A的列空间当然得到其本身，此时投影矩阵可不就是单位阵。

## 最小二乘法

前面也已经提到，投影的方式让我们得到了最优近似解，而 $e$ 向量就是其中的误差。实际上这一思想与我们做线性回归时，使用最小二乘法拟合直线的理论基础不谋而合。

![2023-09-25-14-56-45.png](../.assert/Linear-Algebra-MIT/Lecture15/2023-09-25-14-56-45.png)

对上述直线，我们想通过三个点来拟合一条直线 y = Cx + D。我们将已知的三个点 (1,1), (2,2), (3,2) 带入方程：

$$
\begin{cases}
C + D = 1 \\
2C + D = 2 \\
3C + D = 2
\end{cases}
$$

写作矩阵形式为：

$$
Ax = \begin{bmatrix}
1 & 1 \\
2 & 1 \\
3 & 1
\end{bmatrix}
\begin{bmatrix}
C \\ D
\end{bmatrix} =
\begin{bmatrix}
1 \\ 2 \\ 2
\end{bmatrix}
$$

显然x无解，三点根本不共线，于是利用投影找最优近似解。由于A中各列线性无关，所以 $A^TA\hat{x} = b$ 有解，展开写作：

$$
\begin{bmatrix}
1 & 2 & 3 \\
1 & 1 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 1 \\
2 & 1 \\
3 & 1
\end{bmatrix}
\begin{bmatrix}
C \\ D
\end{bmatrix} =
\begin{bmatrix}
1 & 2 & 3 \\
1 & 1 & 1
\end{bmatrix}
\begin{bmatrix}
1 \\ 2 \\ 2
\end{bmatrix}
$$

矩阵乘法运算得：

$$
\begin{bmatrix}
14 & 6 \\
6 & 3
\end{bmatrix}
\begin{bmatrix}
C \\ D
\end{bmatrix} =
\begin{bmatrix}
11 \\ 5
\end{bmatrix}
$$

亦即：

$$
\begin{cases}
14C + 6D = 11 \\
6C + 3D = 5
\end{cases}
$$

求得：$C = \frac{1}{2}, D = \frac{2}{3}$。

## 微积分解法与投影矩阵的联系

如果用传统微积分来解最小二乘，就需要对误差方程：

$$
E=(C+D−1)^2+(2C+D−2)^2+(3C+D−2)^2
$$

求最小值。方程有两个未知数，分别对C和D计算偏导的零值：

$$
\begin{cases}
2C+2(D−1)+8C+4(D−2)+18C+6(D−2)=0 \\
2D+2(C−1)+2D+2(2C−2)+2D+2(3C−2)=0
\end{cases}
$$

化简得到：

$$
\begin{cases}
14C+6D=11 \\
6C+3D=5
\end{cases}
$$

可以看出这里的两个式子其实和投影矩阵法得到的两个方程一模一样。