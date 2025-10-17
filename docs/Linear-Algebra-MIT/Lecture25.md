# [Lecture 25: Symmetric matrices and positive definiteness](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/resources/lecture-25-symmetric-matrices-and-positive-definiteness/)

# 对称矩阵与正定性

对称矩阵有很多特殊的性质，它们满足：

1. $A = A^T$
2. $v_i \cdot v_j = 0$ (能够找到彼此正交的特征向量)

根据此前掌握的特征分解知识，我们知道对于不同的特征值，分解得到的对应特征向量都是线性无关的。而对于特征值相同的情况，对称矩阵总是可以在零空间中找到彼此垂直的特征向量。

## 对称矩阵的对角化

根据上述两个性质，对称矩阵对角化便可拆解为：$A=Q \Lambda Q^T$，$Q$列向量也就是特征向量可以标准正交（模可标准化为1）。

> 实际上，A的对称性可以通过$Q \Lambda Q^T$看出，$(Q \Lambda Q^T)^T=(Q^T)^T \Lambda Q^T=Q \Lambda Q^T$。
> 

对称矩阵有一个非常重要的性质：**对称矩阵的特征值都是实数**。

可以用共轭来证明，由$Ax=\lambda x$，取共轭得到$A\bar{x}=\bar{\lambda}\bar{x}$（A是实数矩阵故不变），两边同时取转置：$\bar{x}^TA=\bar{x}^T\bar{\lambda}$，两边再同时乘上x，得到:$\bar{x}^TAx=\bar{x}^T\bar{\lambda}x$

将$Ax=\lambda x$带入左侧，得到：$\bar{x}^T\lambda x=\bar{x}^T\bar{\lambda}x$

因此，若$\bar{x}^Tx\neq 0$，则有$\lambda=\bar{\lambda}$，即特征值均为实数。

那么有没有可能$\bar{x}^Tx=0$呢？我们展开来看：$\bar{x}^Tx=[\bar{x}_1\bar{x}_2...\bar{x}_n]\begin{bmatrix}x_1\\x_2\\...\\x_n\end{bmatrix}=\bar{x}_1x_1+\bar{x}_2x_2+...+\bar{x}_nx_n$

其中每一项$\bar{x}_jx_j$都是$(a_j+b_ji)(a_j-b_ji)=a_j^2+b_j^2$，得到的都是大于0的实数，因此必有$\bar{x}^Tx\neq 0$。

> 同理，复数矩阵若满足$\bar{A}^T=A$，则特征值也都将是实数。
> 

## 特征值的性质

特征值已经证实都是实数了，那么是否还有什么其他特殊性质呢？

实际上也是有的，这里先给结论：

1. 对称矩阵的主元正负个数，与特征值的正负个数一致
2. 对称矩阵的主元乘积等于特征值的乘积，它们都等于$det(A)$

根据性质1，我们可以通过消元来快速确定特征值的正负值个数，无需解$n$次方程。

## 对称矩阵的另一个视角

对角化展开：$A=Q\Lambda Q^T=\begin{matrix}
    [q_1 & q_2 & \cdots & q_n]
\end{matrix}\begin{bmatrix}\lambda_1&0&...&0\\0&\lambda_2&...&0\\...&...&...&...\\0&0&...&\lambda_n\end{bmatrix}
\begin{matrix}
    [q_1^T & q_2^T & \cdots & q_n^T]
\end{matrix}$

由于$q_i$相互正交，中间的对角矩阵就相当于每一项的系数，最终得到的是：$A=\lambda_1q_1q_1^T+\lambda_2q_2q_2^T+...+\lambda_n q_n q_n^T$。

由于$q_iq_i^T$是单位向量，故$q_i^Tq_i=1$，此时就可以将$q_iq_i^T$改写成$\frac{q_iq_i^T}{q_j^Tq_j}$，显然这是投影矩阵的形式，可以理解为是$q_j$向$q_i$方向的投影。

换个角度来说：**每一个对称矩阵都是一些相互垂直的投影矩阵的线性组合**。

## 正定矩阵

正定矩阵是指满足以下条件的一类矩阵：

1. 所有特征值 $\lambda_i > 0$ 都是正数。
2. 所有主元 $d_i > 0$ 都为正。
3. 所有子行列式 $det(A_k) > 0$ 都为正。