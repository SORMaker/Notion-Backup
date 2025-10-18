# [Lecture 23: Differential equations and exp(At)](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/resources/lecture-23-differential-equations-and-exp-at/)

# 微分方程

给定微分方程：

$$
\begin{cases} 
\frac{du_1}{dt} = -u_1 + 2u_2 \\
\frac{du_2}{dt} = u_1 - 2u_2
\end{cases}
$$

已知 $u(0) = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$，求 u(t)。

## 微分方程的矩阵解法

方程写作$\frac{du}{dt}=Au$,其中$A=\begin{bmatrix} -1 & 2 \\ 1 & -2 \end{bmatrix}$。很明显A是奇异矩阵，它的两个特征值分别是$\lambda_1=0$和$\lambda_2=-3$。

带入特征值求特征向量，易得$x_1=\begin{bmatrix} 2 \\ 1 \end{bmatrix}$,$x_2=\begin{bmatrix} 1 \\ -1 \end{bmatrix}$。

因此，方程组的通解为：$u(t)=c_1e^{\lambda_1t}x_1+c_2e^{\lambda_2t}x_2$。

> 这里是根据微分方程通解的形式，带入特征值和特征向量大胆猜测了二者之间的关系。我们可以带入原式来验证一下：取$u=e^{\lambda_1t}x_1$带入$\frac{du}{dt}=Au$，得到$\lambda_1e^{\lambda_1t}x_1=Ae^{\lambda_1t}x_1$，由于$e^{\lambda_1t}$是个常数，因此化简后就是$\lambda_1x_1=Ax_1$的形式。同理可以得到$\lambda_2x_2=Ax_2$，带入得到的就是A的特征方程，因此通解正是$u(t)=c_1e^{\lambda_1t}x_1+c_2e^{\lambda_2t}x_2$。
> 

根据初始态$u(0)$的值，我们带入可以求得系数c，分别得到:$c_1=\frac{1}{3},c_2=\frac{1}{3}$。

综上，该微分方程的通解为：$u(t)=\frac{1}{3}\begin{bmatrix} 2 \\ 1 \end{bmatrix}+\frac{1}{3}e^{-3t}\begin{bmatrix} 1 \\ -1 \end{bmatrix}$。

> 观察该通解，可以发现随着时间t的流逝，最终的$u(t)$会无线趋近于$\frac{1}{3}\begin{bmatrix} 2 \\ 1 \end{bmatrix}$。这一现象我们称之为趋于稳态。实际上，在做特征值分解时我们就可以观察到，由于A是奇异矩阵，它的特征值其一是0，此时$e^{\lambda_1t}$始终为1，而另一个特征值是小于0的，此时$e^{\lambda_2t}$随着t增大而无线趋近于0。
> 

> $u(0)$一开始，$u_1=1,u_2=0$，该微分方程的物理意义在于，随着时间的流逝，$u_1$逐渐减少直到趋近于$\frac{2}{3}$，$u_2$逐渐增加直到趋近于$\frac{1}{3}$，$u_1$的值在朝着$u_2$流去，此消彼涨。
> 

### 特征值与状态的关系

根据微分方程通解的形式，我们总结出以下结论：

- 当特征值的实部全都是负数时，$u(t)$将趋于$0$。这里值得注意的是，对于复数特征值的形式$a+bi$，只有实数部分会影响$u(t)$的走势，虚数部分并不会影响。
    - 比如$\lambda = -3+6i$，计算$|e^{(-3+6i)t}|$，其中$e^{6it}$部分是$|cos 6t+i sin 6t|=1$，这个虚部不管是什么值，他始终在复平面的单位圆上转圈。
- 如果有一个为$0$的特征值，且其他特征值的实部皆小于$0$，则$u(t)$最终会收敛到一个特定值。
- 如果存在某个实部大于$0$的特征值，那么$u(t)$最终是发散的。

## 解耦与$e^{At}$

对于上述特征方程，原方程组的两个函数$u1$和$u2$相互耦合，矩阵对角化分解的解法实际上就是对二者进行解耦，得到特征值和特征向量。我们将$u$表示为特征向量的线性组合，写作：$u=Sv$，代入原方程得到：$\frac{dv}{dt}=S^{-1}ASv=\Lambda v$，而Λ是对角化矩阵，此时方程可以展开成：
$$
\begin{cases} \frac{dv_1}{dt}=\lambda_1v_1 \\ \frac{dv_2}{dt}=\lambda_2v_2 \\ ... \\ \frac{dv_n}{dt}=\lambda_nv_n \end{cases}
$$

此时，新方程组的各个$v_i$之间就不再耦合，其一般解形式为：$v(t)=e^{\Lambda t}v(0)$。

因此，$u(t)=e^{At}u(0)=Se^{\Lambda t}S^{-1}u(0)$

### 指数矩阵$e^{At}$

这里$e^{At}$为什么等于$Se^{\Lambda t}S^{-1}$呢？我们可以按泰勒级数进行展开：

$$
e^{At}=I+At+\frac{(At)^2}{2}+\frac{(At)^3}{6}+\cdots+\frac{(At)^n}{n!}+\cdots
$$

而另一方面根据: $A=S\Lambda S^{-1}$
$$
e^{At}=SS^{-1}+S\Lambda S^{-1}t+\frac{S\Lambda^2S^{-1}t^2}{2}+\\
\frac{S\Lambda^3S^{-1}t^3}{6}+\cdots+\frac{S\Lambda^nS^{-1}t^n}{n!}+\cdots
$$

提出 $S$ 和 $S−1$ 得到：
$$S(I+\Lambda t+\frac{\Lambda^2t^2}{2}+\frac{\Lambda^3t^3}{3}+\cdots+\frac{\Lambda^nt^n}{n}+\cdots)S^{-1}e^{At}$$

故$e^{At}=Se^{\Lambda t}S^{-1}$。

最后，来看看$e^{\Lambda t}$展开的形状：
$$
e^{\Lambda t}=\begin{bmatrix} e^{\lambda_1t} & 0 & \cdots & 0 \\ 0 & e^{\lambda_2t} & \cdots & 0 \\ \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & \cdots & e^{\lambda_nt} \end{bmatrix}
$$

> 实际上看到这里，就会发现最终判断收敛与发散的就是这个矩阵里的特征值，只不过对于微分方程来说，对角线上的元素不是$\lambda_it$，而是$e^{\lambda_it}$。
> 

## 二阶微分方程

对于二阶微分方程来说，可以利用上一节应付斐波那契数列的构造法，将：$y^{\prime \prime} + by^\prime + ky = 0$处理成：$u^{\prime} = \begin{bmatrix} y^{\prime \prime} \\ y^\prime \end{bmatrix} = \begin{bmatrix} -b & -k \\ 1 & 0 \end{bmatrix}\begin{bmatrix} y^\prime \\ y \end{bmatrix}$，$u=\begin{bmatrix} y^\prime \\ y \end{bmatrix}$

按照此法可以也可以推广到高阶，比如对5阶就转成一个5×5的矩阵对角化分解特征值、特征向量的问题。$\begin{bmatrix} y^{\prime \prime \prime \prime \prime} \\ y^{\prime \prime \prime \prime} \\ y^{\prime \prime \prime} \\ y^{\prime \prime} \\ y^{\prime} \end{bmatrix}=\begin{bmatrix} -b & -c & -d & -e & -f \\ 1 & 0 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 & 0 \\ 0 & 0 & 1 & 0 & 0 \\ 0 & 0 & 0 & 1 & 0 \end{bmatrix}\begin{bmatrix} y^{\prime \prime \prime \prime} \\ y^{\prime \prime \prime} \\ y^{\prime \prime} \\ y^{\prime} \\ y \end{bmatrix}$
