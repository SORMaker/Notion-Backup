# [Lecture 3：Multiplication and inverse matrices](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/resources/lecture-3-multiplication-and-inverse-matrices/)

## 1、Multiplication (矩阵乘法)

### 乘法定义视角

对于两个矩阵相乘$A_{m*\color{red}n}*B_{{\color{red}n}*p}=C_{m*p}$

尺寸方面必须满足**A的列数等于B的行数，C的规格就是A的行数*B的列数。**

对于矩阵C中第$i$行、第$j$列的元素可以记为$c_{ij}$，其中：

$C_{i j}=(\mathrm{A} \text { 中第 } \mathrm{i} \text { 行向量 })(\mathrm{B} \text { 中第 } \mathrm{j} \text { 列向量 })=\sum_{\mathrm{k}=1}^{\mathrm{n}} a_{i k} b_{k j} = a_{i1}b_{1j}+a_{i2}b_{2j}+a_{i3}b_{3j}+ \dots$

### 列向量线性组合的视角

根据之前矩阵与列向量的乘积，我们在计算矩阵与矩阵的乘积时，可以把后面的矩阵B看作列向量的组合，从而：

$\left[\begin{array}{llll}? & ? & ? & ? \\? & ? & ? & ? \\? & ? & ? & ?\end{array}\right]\left[\begin{array}{llll}a & b & c & d \\a & b & c & d \\a & b & c & d \\a & b & c & d\end{array}\right]=\left[\mathrm{A}\left[\begin{array}{l}a \\a \\a \\a\end{array}\right] \quad \mathrm{A}\left[\begin{array}{l}b \\b \\b \\b\end{array}\right] \quad \mathrm{A}\left[\begin{array}{l}c \\c \\c \\c\end{array}\right] \quad \mathrm{A}\left[\begin{array}{l}d \\d \\d \\d\end{array}\right]\right]=\mathrm{C}$

这样，B的各列，用来线性组合A矩阵的各列向量。

### 行向量线性组合的视角

同理，A的各行，用来线性组合B矩阵的各行向量。

$\begin{gathered}{
\left[\begin{array}{llll}a & a & a & a \\ b & b & b & b \\ c & c & c & c\end{array}\right]
\left[\begin{array}{llll}? & ? & ? & ? \\? & ? & ? & ? \\? & ? & ? & ? \\? & ? & ? & ?\end{array}\right]=
\left[\begin{array}{}{\left[\begin{array}{llll}a & a & a &a \end{array}\right] \mathrm{B}} \\ \\{\left[\begin{array}{llll}b & b & b & b\end{array}\right] \mathrm{B}} \\ \\{\left[\begin{array}{llll}c & c & c & c\end{array}\right] \mathrm{B}}\end{array}\right]=\mathrm{C}} \end{gathered}
$

### 列乘以行的视角

我们也可以用矩阵A的列向量乘上B的行向量得到各个矩阵，再将矩阵相加得到C。

比如下面这个例子：

$\left[\begin{array}{ll}2 & 7 \\3 & 8 \\4 & 9\end{array}\right]\left[\begin{array}{ll}1 & 6 \\0 & 0\end{array}\right]=\left[\begin{array}{l}2 \\3 \\4\end{array}\right]\left[\begin{array}{ll}1 & 6\end{array}\right]+\left[\begin{array}{l}7 \\8 \\9\end{array}\right]\left[\begin{array}{ll}0 & 0\end{array}\right]=\left[\begin{array}{ll}2 & 12 \\3 & 18 \\4 & 24\end{array}\right]$

**也就是说，矩阵相乘可以分解为多个秩1矩阵相加的形式（这部分内容后面会讲）。**

### 分块乘法

矩阵相乘不一定要按照顺序相乘，还可以进行分块相乘。比如：

$\left[\begin{array}{ll}A_1 & A_2 \\A_3 & A_4\end{array}\right]\left[\begin{array}{ll}B_1 & B_2 \\B_3 & B_4\end{array}\right]=\left[\begin{array}{ll}C_1 & C_2 \\C_3 & C_4\end{array}\right]$

其中$A_{1,2,3,4}和B_{1,2,3,4}都是划分之后的一块块矩阵$，如：$\left[\begin{array}{l|ll}? & ? & ? \\\hline ? & ? & ? \\? & ? & ?\end{array}\right]$。

那么$C_1=A_1B_1+A_2B_3$。

## 2、Inverse matrices (逆矩阵)

### 逆矩阵的定义及其存在条件

对于一个**方阵**$A$，如果存在一个矩阵$E$，使得$EA=I$，那么就称矩阵$A$存在一个逆矩阵，并记作$A^{-1}$，满足如下关系：

$AA^{-1} = A^{-1}A=I$

那么何时矩阵没有逆呢？

**如果存在一个非零向量x，使得$Ax=0$，那么矩阵A就是不可逆的，也就是没有逆矩阵。**

### 逆矩阵的求法

**Guass-Jordan方法：**

对于矩阵$A=\left[\begin{array}{ll}
1 & 3 \\
2 & 7
\end{array}\right]$，我们可以在其右边添加一个同大小的单位矩阵并进行行变换，当矩阵A变为单位阵时，右边添加的单位阵就是我们要找的逆矩阵$A^{-1}$：

$\left[\begin{array}{cc:cc}1 & 3&1 & 0 \\2 & 7&0 & 1\end{array}\right] \rightarrow\left[\begin{array}{cc:cc}1 & 3 & 1 & 0 \\0 & 1&-2 & 1\end{array}\right] \rightarrow\left[\begin{array}{cc:cc}1 & 0 & 7 & -3 \\0 & 1&-2 & 1\end{array}\right]$

---

### 逆矩阵的性质

- $(A^{-1})^{-1} = A$
- $(\lambda A)^{-1}=\frac{1}{\lambda}A^{-1}$
- $(AB)^{-1} = B^{-1}A^{-1}$
- $(A^T)^{-1}=(A^{-1})^T$

简单证明一下第三个和第四个性质。

性质三：

$$
\because ABB^{-1}A^{-1}=I,\\\therefore\ (AB)^{-1} = B^{-1}A^{-1}.
$$

性质四：

$$
\because AA^{-1}=I\\\therefore(AA^{-1})^T=(A^{-1})^TA^T=I\\ \therefore (A^T)^{-1}=(A^{-1})^T
$$