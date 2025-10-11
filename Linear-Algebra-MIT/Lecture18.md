# [Lecture 18: Properties of determinants](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/resources/lecture-18-properties-of-determinants/)

# 行列式及其性质

每个方阵都有行列式，行列式是跟该方阵有关的一个数字，行列式的值隐含了该方阵的许多特性。例如：

- 行列式为0的方阵一定不可逆
- 可逆的方阵行列式值一定不为0

行列式数学上写作 $|A|$ ，一般记为 $det(A)$。

## 行列式的三大基础特性

- 性质1：单位阵 $det(I)=1$。
- 性质2：交换任意两行后，行列式值的符号取反。
- 性质3：
    - a ： 行列式可以按行提取矩阵的系数，即:。
        
        $\begin{vmatrix}ta & tb \\c & d\end{vmatrix} = t\begin{vmatrix}a & b \\c & d\end{vmatrix}$
        
    - b : 行列式的行具有线性，即：
        
        $\begin{vmatrix}a+a' & b+b' \\c & d\end{vmatrix}=\begin{vmatrix}a & b \\c & d\end{vmatrix}+\begin{vmatrix}a' & b' \\c & d\end{vmatrix}$
        

> 注意3b里的线性是指行的线性，而非行列式具有线性。
> 

## 衍生推导特性

其他的特性都可以根据三大基础特性来进行推导，它们也十分重要，在我们计算行列式和推导其他特性的过程中常常用到。

- 性质4：如果存在相等的两行，则行列式为0。\
可以用性质2来证明，交换相等的两行，行列式的值会取反，但此时方阵并没有任何改变，因此只有行列式为0值才能满足性质2。
- 性质5：从i行中减去l行的k倍，行列式不发生改变。\
换言之，行列式不因矩阵消元而改变。这一性质对列变换也同样生效（相当于先转置，行变换后再转置回来，就是列变换）。根据性质3，可知：
    
    $$
    \left| \begin{array}{cc}a & b \\c-la & d-lb\end{array} \right| = \left| \begin{array}{cc}a & b \\c & d\end{array} \right| - l\left| \begin{array}{cc}a & b \\a & b\end{array} \right|
    $$
    
    再根据刚刚推导的性质4，等号右边第二项行列式为 0 ，性质5得证。
    
- 性质6：如果方阵的某一行全为 0，则行列式为 0。由性质3或性质5推理显然。
- 性质7：上三角方阵 $U$ 的行列式值等于对角线上元素的乘积：$\det(U) = d_1 d_2 \cdots d_n$。\
由性质5，从最后一行开始，将对角线元素上方的非零元素依次变为0，行列式值不变。再根据性质3a和性质1得证。
    
    $$
    \det(U) = \left| \begin{array}{}d_1 & * & \cdots & * \\0 & d_2&\cdots & *\\ \cdots&\cdots&\cdots&\cdots\\0&0&\cdots &d_n\end{array} \right| = \left| \begin{array}{}d_1 & 0 & \cdots & 0 \\0 & d_2&\cdots & 0\\ \cdots&\cdots&\cdots&\cdots\\0&0&\cdots &d_n\end{array} \right| = \\ 
    d_1 d_2 \cdots d_n \left| \begin{array}{} 1 & 0 & \cdots & 0 \\0 & 1 &\cdots & 0\\ \cdots&\cdots&\cdots&\cdots\\0&0&\cdots &1\end{array} \right| = d_1 d_2 \cdots d_n
    $$
    
- 性质8：当且仅当 $A$ 是奇异矩阵时（不可逆），$\det(A) = 0$。由性质7，若A可逆，消元得到的上三角矩阵 $U$ 对角线上各行都有主元，行列式必不为0。反之，消元后会出现全零行，由性质6可知行列式为0。
- 性质9：$\det(AB) = \det(A) \cdot \det(B)$。可通过行变换将A和B转化为对角阵，并利用性质5和性质7证明。\
由性质5，可以把 A 和 B 通过行变换最终转为对角阵 A‘，B’，此时 $\det(A')=\det(A), \det(B')=\det(B)$。而两个矩阵对角相乘的结果显而易见：
    
    $$
    \left. \begin{bmatrix}a_1&0&\ldots&0\\0&a_2&\ldots&0\\\ldots&\ldots&\ldots&\ldots\\0&0&\ldots&a_n\end{bmatrix} \right. \left. \begin{bmatrix}b_1&0&\ldots&0\\0&b_2&\ldots&0\\\ldots&\ldots&\ldots&\ldots\\0&0&\ldots&b_n\end{bmatrix} \right. = \left. \begin{bmatrix}a_1b_1&0&\ldots&0\\0&a_2b_2&\ldots&0\\\ldots&\ldots&\ldots&\ldots\\0&0&\ldots&a_nb_n\end{bmatrix} \right.
    $$
    

由性质7，可知 $\det(A^\prime B^\prime)=(a_1 \cdot a_2\cdot\ldots\cdot a_n)(b_1\cdot b_2\cdot\ldots\cdot b_n)$，故 $\det (A^{\prime}B^{\prime})=\det (A^{\prime}) \det (B^{\prime})$。

> 根据性质9可知：$\det(A^{-1}) = \frac{1}{\det(A)}$（当A可逆时），且$\det(A^2) = (\det(A))^2$。
> 
- 性质10：$\det(A^T) = \det(A)$。\
利用LU分解$A = LU$，则$A^T = U^T L^T$。\
结合性质9: $\det (A^T) = \det(U^T) \cdot \det(L^T)$，且三角阵转置后还是三角阵，故 $\det (U^T) = \det(U), \det (L^T) = \det(L)$，因此 $\det(A^T) = \det(A)$

## 二阶方阵的行列式

根据上述的性质，我们可以很容易推出二阶方阵的行列式：

$$
\begin{vmatrix}a & b\\c & d\end{vmatrix} = ad-bc
$$

熟练掌握这些行列式的特性，不仅可以让我们快速求解行列式，还可以看出矩阵的变换与行列式变化的联动关系。