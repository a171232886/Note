---
title: Transformer中的位置编码
date: 2023-05-06 00:04:05
categories: 
- [公式推导, NLP]
mathjax: true
---

# 0. 前言
"Attention is all you need"中有一个位置编码方案，目的是消除绝对位置的影响。本文尝试从数学角度验证该编码方案可使两位置的编码相关性只受相对位置影响，而不受绝对位置影响。

<!--more-->

# 1. 位置编码
$$
\begin{aligned}
    PE(p,2i) &= \sin(10^{-\frac{\displaystyle 4 \cdot 2i}{\displaystyle d_m}}\cdot p) \\
    PE(p,2i+1) &= \cos(10^{-\frac{\displaystyle 4 \cdot 2i}{\displaystyle d_m}}\cdot p) \\
\end{aligned}
$$

探究此编码方案，两个位置的位置编码的相关性只受相对位置影响，而不受绝对位置影响

#  2. 分析
1. 假设条件：
    - $p$ 表示第p个单词
    - $d_m$ 为词嵌入向量的维数，假设为偶数，即 $d_m \mod 2 =0$
    - $n$ 表示维度，$n = 0, 1, ..., d_m-1$
    - $i = 0, 1,..., \frac{d_m}{2} -1$
2. 简化表示，记
    $$
        f(n) = 10^{-\frac{\displaystyle 4 \cdot n}{\displaystyle d_m}}
    $$

    得到，
    $$
    PE(p,n) =  \left\{
        \begin{array}{rcl}
            \sin\big(f(n) \cdot p \big)       &,      & n =2i （偶数） \\
            \cos\big(f(n) \cdot p \big)       &,      & n =2i+1 （奇数）\\
        \end{array} \right. 
    \tag{2.1}
    $$
    
    此时某一单词的位置编码为
    $$
    PE(p,:) = [\sin\big(f(0) \cdot p \big) , \   \cos\big(f(0) \cdot p \big), \  ..., \ \sin\big(f(n) \cdot p \big) , \   \cos\big(f(n) \cdot p \big), \  ..., \ \sin\big(f(d_m-1) \cdot p \big) , \   \cos\big(f(d_m-1) \cdot p \big)  ]
    $$

    发现 \sin 和 \cos 总是成对出现，使用
    $$
    g(p,i) = [\sin\big(f(i) \cdot p \big) , \   \cos\big(f(i) \cdot p \big)]
    $$
    简化表示，得到
    $$
    PE(p,:) = [g(p,0), \ ..., \ g(p,i), \ ..., \ g(p,\frac{d_m}{2} -1) ]
    \tag{2.2}
    $$
    $PE(p,:)$简记为$PE(p)$

    从(2.1)到(2.2)强烈建议，实际推算一下体会其巧妙，可假设$d_m=10$

3. 明确任务：位置编码的相关性只受相对位置影响
    假设存在一个矩阵$T(k)$, 满足
    $$
    T(k) \times PE(p) = PE(p+k) 
    $$
    此时，命题“位置编码的相关性只受相对位置影响”可表示为，
    $$
    PE(p+k) \cdot PE(p) = T(k) \times PE(p) \cdot PE(p) = N \cdot T(k)
    \tag{2.3}
    $$
    其中N为常数

    **因此，只要我们能构建出T(k)，并且$PE(p) \cdot PE(p) = N$，则该命题成立**

4. 分析$ PE(p) \cdot PE(p) $
    $$
    \begin{aligned}
        PE(p) \cdot PE(p) &= g(p,0) \cdot g(p,0) + ... + g(p,\frac{d_m}{2} -1) \cdot g(p,\frac{d_m}{2} -1) \\

                          &= \sum \limits_{i=0}^{\frac{d_m}{2} -1} g(p,i) \cdot g(p,i) \\

                          &= \sum \limits_{i=0}^{\frac{d_m}{2} -1} 1 \\

                          &= \frac{d_m}{2} \\
    \end{aligned}
    $$

    即 $N = \frac{d_m}{2}$

5. 构建$T(k)$，

    $$
    {T(k)}=\left[\begin{array}{cccc}
    {\Phi(k,0)} & \mathbf{0} & \cdots & \mathbf{0} \\
    \mathbf{0} & {\Phi(k,1)} & \cdots & \mathbf{0} \\
    \mathbf{0} & \mathbf{0} & \ddots & \mathbf{0} \\
    \mathbf{0} & \mathbf{0} & \cdots & {\Phi(k,\frac{d_m}{2} -1)}
    \end{array}\right]
    $$

    其中，$\Phi(k,i)$
    $$
    {\Phi(k,i)}=\left[\begin{array}{cc}
    \cos \left(r_i k\right) & -\sin \left(r_i k\right) \\
    \sin \left(r_i k\right) & \cos \left(r_i k\right)
    \end{array}\right]
    $$

    此时，$T(k) \cdot PE(p) = PE(p+k)$具体到第$i$项为$\Phi(k,i)\times g(p,i) = g(p+k,i)$，（该步推导强烈建议实际推算一下），即
    $$
    \left[\begin{array}{cc}
    \cos \left(r_i k\right) & \sin \left(r_i k\right) \\
    -\sin \left(r_i k\right) & \cos \left(r_i k\right)
    \end{array}\right]\left[\begin{array}{c}
    \sin \left(f(i) \cdot p\right) \\
    \cos \left(f(i) \cdot p\right)
    \end{array}\right]=\left[\begin{array}{c}
    \sin \left(f(i) \cdot (p+k)\right) \\
    \cos \left(f(i) \cdot (p+k)\right)
    \end{array}\right]
    \tag{2.4}
    $$

    如果我们找到$r_i$如何计算，则矩阵$T$构建完毕。计算 $\sin \left(f(i) \cdot (p+k)\right)$,
    $$
    \cos(r_i k) \sin (f(i) \cdot p) + \sin (r_i k) \cos (f(i) \cdot p) = \sin (f(i) \cdot (p+k)) = \sin (f(i) \cdot p) \cos (f(i) \cdot k) + \cos (f(i) \cdot p)\sin (f(i) \cdot k)
    $$

    对应项相等可以得到，$r_i = f(i)$，此时矩阵$T$构建完毕

5. 至此我们构建一个矩阵$T$，可以使下式成立，
    $$
        PE(p+k) \cdot PE(p) = T(k) \times PE(p) \cdot PE(p) = \frac{d_m}{2} \cdot T(k)
    $$

    因此可以证明：该位置编码方案中，两个位置编码的相关性只受相对位置影响。


# 参考
1. https://zhuanlan.zhihu.com/p/106644634