---
title: SSIM和PSNR
date: 2021-11-18 17:32:13
categories: 
- [笔记, 图像补全]
mathjax: true
---
# SSIM
Structural Similarity（SSIM）结构相似性

<!--more-->

## Matlab版定义
MatLab版见[官方介绍](https://ww2.mathworks.cn/help/images/ref/ssim.html)。
基于三个项的计算，即亮度项、对比度项和结构项。
$$
\operatorname{SSIM}(x, y)=[l(x, y)]^{\alpha} \cdot[c(x, y)]^{\beta} \cdot[s(x, y)]^{\gamma}
$$
$$
\begin{aligned}
l(x, y) &=\frac{2 \mu_{x} \mu_{y}+C_{1}}{\mu_{x}^{2}+\mu_{y}^{2}+C_{1}} \\
c(x, y) &=\frac{2 \sigma_{x}{\sigma_{y}}+C_{2}}{\sigma_{x}^{2}+\sigma_{y}^{2}+C_{2}} \\
s(x, y) &=\frac{\sigma_{x y}+C_{3}}{\sigma_{x} \sigma_{y}+C_{3}}
\end{aligned}
$$

其中$\mu$和$\sigma$表示均值和方差。

如果 α = β = γ = 1（Exponents 的默认值），且 C3 = C2/2（C 3 的默认选择），则索引简化为：

$$\operatorname{SSIM}(x, y)=\frac{\left(2 \mu_{x} \mu_{y}+C_{1}\right)\left(2 \sigma_{x y}+C_{2}\right)}{\left(\mu_{x}^{2}+\mu_{y}^{2}+C_{1}\right)\left(\sigma_{x}^{2}+\sigma_{y}^{2}+C_{2}\right)}$$

### 性质：
1. Symmetry: $S(\mathbf{x}, \mathbf{y})=S(\mathbf{y}, \mathbf{x})$;
2. Boundedness: $S(\mathbf{x}, \mathbf{y}) \leq 1$;
3. Unique maximum: $S(\mathbf{x}, \mathbf{y})=1$ if and only if $\mathbf{x}=\mathbf{y}$ (in discrete representations, $x_{i}=y_{i}$ for all $i=1,2, \cdots, N$ );

### 使用方式
```Matlab
ssimval = ssim(A,ref)
```


## Python版实现
[官方解释](https://scikit-image.org/docs/dev/auto_examples/transform/plot_ssim.html) ：MSE不能很好的考虑到纹理，因此引入SSIM。

### 使用方式
```Python
from skimage.metrics import structural_similarity as ssim
import matplotlib.pyplot as plt
# 针对多通道图像，默认最后一维度为通道，每个通道进行单独计算，然后平均
a = plt.imread('a.jpg')
b = plt.imread('b.jpg')
result = ssim(a,b,multichannel=True) 
```

采用公式为
$$\operatorname{SSIM}(x, y)=\frac{\left(2 \mu_{x} \mu_{y}+C_{1}\right)\left(2 \sigma_{x y}+C_{2}\right)}{\left(\mu_{x}^{2}+\mu_{y}^{2}+C_{1}\right)\left(\sigma_{x}^{2}+\sigma_{y}^{2}+C_{2}\right)}$$
其中，
$C_{1}=\left(K_{1} L\right)^{2}$，$C_{2}=\left(K_{2} L\right)^{2}$，$K_{1}=0.01$, $K_{1}=0.03$。 $L$是动态范围：若像素值为[0,255]，则$L=255$。

# PSNR
峰值信噪比Peak signal-to-noise ratio，单位是dB

$$ M S E=\frac{1}{m n} \sum_{i=0}^{m-1} \sum_{j=0}^{n-1}[I(i, j)-K(i, j)]^{2} $$
$$ P S N R=10 \log _{10}\left(\frac{\left(2^{n}-1\right)^{2}}{M S E}\right) $$

其中，n表示图像位数，比如n=8。

多通道图像如RGB图像，在计算MSE时就进行了进行了平均，最终MSE是标量。

## Python的使用方式
```Python
from skimage.metrics import peak_signal_noise_ratio as psnr
a = plt.imread('a.jpg')
b = plt.imread('b.jpg')
psnr_ = psnr(a,b)
```
注意：数据类型不同会导致psnr值不同。

# 参考文献
1.  Image quality assessment: From error visibility to structural similarity. IEEE Transactions on Image Processing, https://ece.uwaterloo.ca/~z70wang/publications/ssim.pdf
2.  https://en.wikipedia.org/wiki/Peak_signal-to-noise_ratio


# 遗留问题
SSIM是怎么考虑纹理等信息的