---
title: Pytorch中的转置卷积
date: 2021-11-22 19:56:58
categories: 
- [笔记, Python，Numpy和Pytorch]
# mathjax: true
---

# 转置卷积(Transposed Convolution)
又称为转置卷积。

```torch.nn.ConvTranspose2d(in_channels, out_channels, kernel_size, stride=1, padding=0, output_padding=0, groups=1, bias=True, dilation=1, padding_mode='zeros')```


# 输出大小计算
先说结论
$$o' = (i' - 1)s + k - 2p+p'$$
**$p'$是```output_padding```，后面一直是在分析这个式子是怎么来的。**
注：当有空洞卷积时，k=dilation[0]×(kernel_size[0]−1)+1

<!--more-->

## 起点

首先回顾正常的卷积计算公式
$$o = \left\lfloor {\frac{(i - k + 2p)}{s}} \right\rfloor  + 1 $$
其中，$i$表示输入大小，$o$表示输出大小，$k$表示卷积核大小，$p$表示pading，$s$表示卷积步长。正常来说，$i>=o$

反卷积的输入$i'$输出$o'$，而此时$i'<=o'$。（毕竟我们的目的是放大图像。）**带入上式得到以下，我们需要根据下面式子，反推出$o'$。**

$$i' = \left\lfloor {\frac{o' - k + 2p}{s}} \right\rfloor  + 1$$

麻烦的是向下取整，所以分类讨论：

## 可以整除
即当$({o' - k + 2p})\%{s}=0$时，上式变为，
$$i' = \frac{o' - k + 2p}{s} + 1$$
此时反卷积的输入$i'$输出$o'$可以**一一对应**。
$$o' = (i' - 1)s + k - 2p$$

代码验证

```python
>>> x = torch.ones([1,10,5,5])
>>> a2 = torch.nn.ConvTranspose2d(in_channels=10,
                              out_channels=10,
                              kernel_size=3,
                              stride=2,
                              padding=0,
                              output_padding=0)
>>> y = a2(x)
>>> print(y.shape)

torch.Size([1, 10, 11, 11])
```

也就是当$i'=5，k=3，s=2，p=0$时，$o'=11$。

## 不可以整除
即当$({o' - k + 2p})\%{s} \ne0$时，向下取整不能去掉时。我们减去一部分，让它能取整。我们定义$p'=(o' - k + 2p)\% s$，也就是```output_padding```

$$
\begin{aligned}
i' &= \frac{o' - k + 2p - (o' - k + 2p)\% s}{s} + 1 \\
&= \frac{o' - k + 2p - p'}{s} + 1
\end{aligned}
$$

反推得到 $o' = (i' - 1)s + k - 2p+p'$

代码验证

```python
>>> x = torch.ones([1,10,4,4])
>>> a2 = torch.nn.ConvTranspose2d(in_channels=10,
                              out_channels=10,
                              kernel_size=3,
                              stride=2,
                              padding=0,
                              output_padding=1
                              )
>>> y = a2(x)
>>> print(y.shape)

torch.Size([1, 10, 10, 10])
```
也就是当$i'=4，k=3，s=2，p=0，p'=1$时，$o'=10$。

回过头来说，这样直接减去一项合理吗？看着不合理，其实很合理。毕竟标准卷积也是向下取整。而这里只是取下界：

# 综合起来
前面一直是反推反推，还得考虑是否整除。难道你还让计算机挨个去世吗？显然不是，计算机知道的只是下面这个式子，
$$o' = (i' - 1)s + k - 2p+p'$$
等式右边的参数全都可以控制输入，也就是控制了输出。前文只不过是对这个式子的理解，当$p'=0$是整除情况，当$p'\ne0$是不可整除情况。


# 参考文献
1. https://www.cnblogs.com/kk17/p/10111768.html
2. https://www.zhihu.com/question/48279880
3. https://pytorch.org/docs/1.4.0/nn.html?highlight=torch%20nn%20convtranspose2d#torch.nn.ConvTranspose2d
3. [A guide to convolution arithmetic for deep learning](https://arxiv.org/abs/1603.07285)