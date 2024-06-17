---
title: EM算法公式推导
date: 2023-05-03 00:04:10
categories: 
- [公式推导, ML]
mathjax: true
---

# 0. 前言
以下针对《统计学习方法(第二版)》的中的内容

# 1. 背景介绍

# 2. EM公式推导

记可观测变量为$Y$，隐变量为$Z$，参数为$\theta$ 。根据对$Y$的$n$次观测，估计$\theta$

<!--more-->

1. 最大似然MLE：

   $$
   \begin{aligned}
      \hat{\theta} & = \mathop{\arg\max}\limits_{\theta} \ \prod \limits_{j=1}^{n} P(y_j |\theta) \\
                  & = \mathop{\arg\max}\limits_{\theta} \ \sum \limits_{j=1}^{n} \log P(y_j |\theta) \\
   \end{aligned}
   \tag{2.1}
   $$
   
2. 记第j个样本对应的似然函数 $L_j(\theta) = \log P(y_j |\theta)$

   $$
   \begin{aligned}
   L_j(\theta) - L_j(\theta^{(i)}) & = \log P(y_j |\theta)  - \log P(y_j |\theta^{(i)}) 	\\
   		  & = \log  \sum \limits_{z_j} P(y_j, z_j |\theta)  - \log P(y_j |\theta^{(i)}) 			\\
   		  & = \log \Big( \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \frac{ P(y_j, z_j |\theta)}{P(z_j | y_j, \theta^{(i)})} \Big)   - \log P(y_j |\theta^{(i)}) 			\\       
   \end{aligned}
   \tag{2.2}
   $$
   
3. 应用Jenson不等式，见书《统计学习方法》p179
   $$
   \log \sum \limits_i \lambda_i \ y_i \geqslant  \sum \limits_i \lambda_i \ \log(y_i)
   $$
   其中，$\sum \limits_i \lambda_i = 1$，$\lambda_i \geqslant 0$.  带入（2.2）得到

$$
\begin{aligned}
L_j(\theta) - L_j(\theta^{(i)}) & = \log  \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \frac{ P(y_j, z_j |\theta)}{P(z_j | y_j, \theta^{(i)})}  - \log P(y_j |\theta^{(i)})  \\ 

		  & \geqslant \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log \frac{ P(y_j, z_j |\theta)}{P(z_j | y_j, \theta^{(i)})}    - \log P(y_j |\theta^{(i)}) 	\\
		  
		  & = \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log \frac{ P(y_j, z_j |\theta)}{P(z_j | y_j, \theta^{(i)})}  - \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)})  \cdot \log P(y_j |\theta^{(i)}) 				\\
		  
		  & = \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log \frac{ P(y_j, z_j |\theta)}{P(z_j | y_j, \theta^{(i)})}  - \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log P(y_j |\theta^{(i)}) 				\\
		  
		  & = \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log \frac{ P(y_j, z_j |\theta)}{P(z_j | y_j, \theta^{(i)}) \cdot P(y_j |\theta^{(i)})}  \\
		  
		  & = \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log \frac{ P(y_j, z_j |\theta)}{P(y_j, z_j |\theta^{(i)})}  \\
		  
\end{aligned}
\tag{2.3}
$$

4. 记$L_j(\theta)$ 的下界为$B_j(\theta , \theta^{(i)})$
   $$
   \begin{aligned}
       B_j(\theta , \theta^{(i)}) & = L_j(\theta^{(i)}) + \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log \frac{ P(y_j, z_j |\theta)}{P(y_j, z_j |\theta^{(i)})}  \\  
       \end{aligned}
       \tag{2.4}
   $$

   通过提升下界为$B_j(\theta , \theta^{(i)})$的方式，提升$L(\theta)$，即
   $$
   \begin{aligned}   
    L_j(\theta) & = \mathop{\max} B_j(\theta , \theta^{(i)})  \\                
   \end{aligned}
   $$
   于是，
   $$
   \begin{aligned}   
    \theta^{i+1} & = \mathop{\arg\max}\limits_{\theta} B(\theta , \theta^{(i)})  \\   
    
                 & = \mathop{\arg\max}\limits_{\theta} \bigg(  L(\theta^{(i)}) + \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log \frac{ P(y_j, z_j |\theta)}{P(y_j, z_j |\theta^{(i)})}  \bigg) \\ 
                 
                 & = \mathop{\arg\max}\limits_{\theta} \bigg(  L(\theta^{(i)}) + \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log  P(y_j, z_j |\theta) - \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log P(y_j, z_j |\theta^{(i)})  \bigg) \\
   \end{aligned}
   $$
   
   其中，当$\theta^{(i)}$给定时，$L(\theta^{(i)})$，$P(z_j | y_j, \theta^{(i)})$， $P(y_j, z_j |\theta^{(i)})$均为定值
   
   所以，
   $$
   \begin{aligned} 
   \theta^{i+1} & = \mathop{\arg\max}\limits_{\theta} \sum \limits_{j=1}^{n} \bigg( \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log  P(y_j, z_j |\theta) \bigg) \\
                & = \mathop{\arg\max}\limits_{\theta} E_{Z|Y,\ \theta^{(i)}} \big[ \log  P(Y, Z |\theta) \big]
   \end{aligned}
   \tag{2.5}
   $$
   
5. 最终简化得到的函数为$Q(\theta , \theta^{(i)})$

   $$
   \begin{aligned} 
   Q(\theta , \theta^{(i)}) & =  \sum \limits_{j=1}^{n} \bigg( \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log  P(y_j, z_j |\theta) \bigg) \\
                & = E_{Z|Y,\ \theta^{(i)}} \big[ \log  P(Y, Z |\theta) \big]
   \end{aligned}
   \tag{2.6}
   $$


# 3. 三硬币问题分析
1. 记$P(z_i = B) = \pi , \  P(y_i | z_i = B) = p, \  P(y_i | z_i = C) = q$
   $$
   \begin{aligned} 
   Q(\theta , \theta^{(i)}) & =  \sum \limits_{j=1}^{n} \bigg( \sum \limits_{z_j} P(z_j | y_j, \theta^{(i)}) \cdot \log  P(y_j, z_j |\theta) \bigg) \\
                & = \sum \limits_{j=1}^{n} \bigg( P(z_j = B | y_j, \theta^{(i)}) \cdot \log  P(y_j, z_j = B |\theta) + P(z_j = C | y_j, \theta^{(i)}) \cdot \log  P(y_j, z_j = C |\theta)  \bigg) \\
   \end{aligned}
   \tag{3.1}
   $$

2. 记 $\mu_j^{(i+1)} =  P(z_j = B | y_j, \theta^{(i)})$, （表示根据第i次得到的参数值，在第i+1次迭代中，在$y_j$条件下确定$z_j$的概率），则$1- \mu_j^{(i+1)} =  P(z_j = C | y_j, \theta^{(i)})$.
   $$
   \begin{aligned} 
   \mu_j^{(i+1)}  & = P(z_j = B | y_j, \theta^{(i)})   \\
                  & = \frac{P(y_j | z_j = B, \theta^{(i)}) \cdot P(z_j = B | \theta^{(i)})}{P(y_j | z_j = B, \theta^{(i)}) \cdot P(z_j = B | \theta^{(i)}) + P(y_j | z_j = C, \theta^{(i)}) \cdot P(z_j = C | \theta^{(i)})}
   \end{aligned}
   \tag{3.2}
   $$
   
   **注意，因为 $\theta^{(i)}$给定，所以$\mu_j^{(i+1)}$也是确定的**
   
3. $P(y_j | z_j = B, \theta^{(i)})$ 的计算 ($y_j =1$表示硬币为上)
   $$
   \begin{aligned} 
   P(y_j | z_j = B, \theta^{(i)}) & =\left\{
      \begin{array}{rcl}
      p^{(i)}       & ,     & {y_j = 1}\\
      1-p^{(i)}     & ,     & {y_j = 0}\\
      \end{array} \right.        \\
      & = (p^{(i)})^{y_j}(1-p^{(i)})^{1-y_j}
   \end{aligned}
   $$

   同理，
   $$
   P(y_j | z_j = C, \theta^{(i)}) = (q^{(i)})^{y_j}(1-q^{(i)})^{1-y_j}
   $$

   带入(3.2)得到，
   $$
   \begin{aligned} 
   \mu_j^{(i+1)}  & = \frac{P(y_j | z_j = B, \theta^{(i)}) \cdot P(z_j = B | \theta^{(i)})}{P(y_j | z_j = B, \theta^{(i)}) \cdot P(z_j = B | \theta^{(i)}) + P(y_j | z_j = C, \theta^{(i)}) \cdot P(z_j = C | \theta^{(i)})} \\
                  
                  & = \frac{\pi^{(i)} \cdot (p^{(i)})^{y_j}(1-p^{(i)})^{1-y_j}}{\pi^{(i)} \cdot (p^{(i)})^{y_j}(1-p^{(i)})^{1-y_j} + (1 - \pi^{(i)}) \cdot (q^{(i)})^{y_j}(1-q^{(i)})^{1-y_j}}
   \end{aligned}
   $$

4. 计算 $P(y_j, z_j = B |\theta)$ 
   $$
   \begin{aligned} 
      P(y_j, z_j = B |\theta) & = P(y_j | z_j = B , \theta) \cdot P(z_j = B , \theta) \\
                              & = p^{y_j} \cdot (1-p)^{1-y_j} \cdot \pi   \\
   \end{aligned}
   $$

   同理，
   $$
   \begin{aligned} 
      P(y_j, z_j = C |\theta) & = P(y_j | z_j = C , \theta) \cdot P(z_j = C , \theta) \\
                              & = q^{y_j} \cdot (1-q)^{1-y_j} \cdot (1-\pi)   \\
   \end{aligned}
   $$

5. 将第2，3，4步结果带入(3.1)
   $$
   \begin{aligned} 
   Q(\theta , \theta^{(i)}) & = \sum \limits_{j=1}^{n} \bigg( P(z_j = B | y_j, \theta^{(i)}) \cdot \log  P(y_j, z_j = B |\theta) + P(z_j = C | y_j, \theta^{(i)}) \cdot \log  P(y_j, z_j = C |\theta)  \bigg) \\
   
            & =  \sum \limits_{j=1}^{n} \bigg( \mu_j^{(i+1)} \cdot \log \big( p^{y_j} \cdot (1-p)^{1-y_j} \cdot \pi \big) + (1 - \mu_j^{(i+1)}) \cdot \log \big( q^{y_j} \cdot (1-q)^{1-y_j} \cdot (1-\pi) \big)  \bigg) \\
   \end{aligned}
   $$

6. 求导
   1. $\frac{\partial Q}{\partial \pi} = 0$
      $$
      \begin{aligned} 
      \frac{\partial Q}{\partial \pi} = \sum \limits_{j=1}^{n} \bigg( \frac{\mu_j^{(i+1)}}{\pi} + (-1) \cdot \frac{1 - \mu_j^{(i+1)}}{1 - \pi} \bigg) = 0 \\
      \end{aligned}
      $$

      可得，
      $$
      \begin{aligned} 
         \pi^{(i+1)} = \frac{\sum \limits_{j=1}^{n} \mu_j^{(i+1)}}{n} \\
      \end{aligned}
      $$

   2. $\frac{\partial Q}{\partial p} = 0$
      $$
      \begin{aligned} 
         \frac{\partial Q}{\partial p} & = \frac{\partial \sum \limits_{j=1}^{n} \bigg( \mu_j^{(i+1)} \cdot \log \big( p^{y_j} \cdot (1-p)^{1-y_j} \cdot \pi \big) \bigg)}{\partial p} \\
         
         & = \frac{\partial \sum \limits_{j=1}^{n} \bigg( \mu_j^{(i+1)} \cdot \big( y_j \cdot \log  p + (1 - y_j) \cdot \log  (1 - p) + \log\pi \big) \bigg)}{\partial p} \\
      
         & = \sum \limits_{j=1}^{n} \bigg( \mu_j^{(i+1)} \cdot \big( \frac{y_j}{p} - \frac{1 - y_j}{1 - p}\big) \bigg) = 0 \\
      \end{aligned}
      $$

      可得，
      $$
      \begin{aligned} 
         p^{(i+1)} = \frac{\sum \limits_{j=1}^{n} \mu_j^{(i+1)} \cdot y_j}{\sum \limits_{j=1}^{n} \mu_j^{(i+1)}} \\
      \end{aligned}
      $$

   3. $\frac{\partial Q}{\partial q} = 0$

      同理，
      $$
      \begin{aligned} 
         \frac{\partial Q}{\partial p} & = \frac{\partial \sum \limits_{j=1}^{n} \bigg( (1 - \mu_j^{(i+1)}) \cdot \big( y_j \cdot \log  (1-q) + (1 - y_j) \cdot \log  (1 - q) + \log (1 - \pi) \big) \bigg)}{\partial q} \\
      
         & = \sum \limits_{j=1}^{n} \bigg( (1 - \mu_j^{(i+1)}) \cdot \big( \frac{y_j}{q} - \frac{1 - y_j}{1 - q}\big) \bigg) = 0 \\
      \end{aligned}
      $$

      可得，
      $$
      \begin{aligned} 
         q^{(i+1)} = \frac{\sum \limits_{j=1}^{n} (1 - \mu_j^{(i+1)}) \cdot y_j}{\sum \limits_{j=1}^{n} (1 - \mu_j^{(i+1)})} \\
      \end{aligned}
      $$

# 4. GMM高斯混合模型应用EM估计参数

1. 模型定义
   $$
      \begin{aligned} 
         P(y_j | \theta) = \sum \limits_{k=1}^{K} a_k \cdot \phi(y_j | \theta_k) 
      \end{aligned}
   $$

   $\phi(y_j | \theta_k)$表示由第$k$个高斯分布生成$y_i$的概率。为方便，小王简记为$\phi_{jk}$
   $$
      \begin{aligned} 
         \phi(y_j | \theta_k) = \frac{1}{\sqrt{2\pi\sigma^2}}e^{-\frac{(y_j - \mu_k)^2}{2\sigma^2}} 
      \end{aligned}
   $$

2. Q函数
   $$
      \begin{aligned} 
         Q(\theta, \theta^{(i)}) = \sum \limits_{j=1}^{N} \sum \limits_{k=1}^{K} P(r_j = r_{jk} | y_j, \theta^{(i)}) \cdot \log P(r_j = r_{jk} , y_j | \theta)
      \end{aligned}  
      \tag{4.1} 
   $$

3. 计算 $ P(r_j = r_{jk} | y_j, \theta^{(i)}) $, 记为 $ \hat r_{jk} $
   $$
      \begin{aligned} 
         \hat r_{jk} & = P(r_j = r_{jk} | y_j, \theta^{(i)}) \\

         & = \frac{ P(y_j | r_j = r_{jk}, \theta^{(i)}) \cdot P(r_j = r_{jk} | \theta^{(i)})}{\sum \limits_{k'=1}^{K} P(y_j | r_j = r_{jk'}, \theta^{(i)}) \cdot P(r_j = r_{jk'} | \theta^{(i)})} \\

         & = \frac{\phi_{jk}^{(i)}  \cdot a_{k}^{(i)}}{\sum \limits_{k'=1}^{K} \phi_{jk'}^{(i)}  \cdot a_{k'}^{(i)}}
      \end{aligned}  
      \tag{4.2} 
   $$

   分析$\hat r_{jk}$, $n_k$ 表示N个样本中有$n_k$个样本是由第$k$个分量生成的
   $$
      \begin{aligned} 
         n_k = \sum \limits_{j=1}^{N} r_{jk}
      \end{aligned}    
      \tag{4.3}     
   $$

   $$
      \begin{aligned} 
         N = \sum \limits_{k=1}^{K} n_k
      \end{aligned}  
      \tag{4.4}      
   $$

5. 计算$P(r_j = r_{jk} , y_j | \theta)$
   $$
      \begin{aligned} 
         P(r_j = r_{jk} , y_j | \theta)  & = P(y_j | r_j = r_{jk} , \theta) \cdot P(r_j = r_{jk} | \theta) \\

         & = \phi_{jk} \cdot a_k \\
      \end{aligned}  
      \tag{4.5}      
   $$

6. 将第3，4步带入(4.1)
   $$
      \begin{aligned} 
         Q(\theta, \theta^{(i)}) & = \sum \limits_{j=1}^{N} \sum \limits_{k=1}^{K} P(r_j = r_{jk} | y_j, \theta^{(i)}) \cdot \log P(r_j = r_{jk} , y_j | \theta)  \\

         & = \sum \limits_{j=1}^{N} \sum \limits_{k=1}^{K} \hat r_{jk} \cdot \log (\phi_{jk} \cdot a_k) \\

         & = \sum \limits_{k=1}^{K} \sum \limits_{j=1}^{N} (\hat r_{jk} \cdot \log a_k + \hat r_{jk} \cdot \log \phi_{jk}) \\

         & = \sum \limits_{k=1}^{K} \big( \sum \limits_{j=1}^{N} \hat r_{jk} \cdot \log a_k + \sum \limits_{j=1}^{N} \hat r_{jk} \cdot \log \phi_{jk} \big) \\

         & = \sum \limits_{k=1}^{K} \big( \hat n_k \cdot \log a_k + \sum \limits_{j=1}^{N} \hat r_{jk} \cdot \log \phi_{jk} \big) \\

         & = \sum \limits_{k=1}^{K} \Big( \hat n_k \cdot \log a_k + \sum \limits_{j=1}^{N} \hat r_{jk} \cdot \big( -\frac{1}{2} \log 2\pi - \frac{1}{2} \log \sigma_k^2 - \frac{(y_j - \mu_k)^2}{2\sigma_k^2}  \big) \Big) \\


      \end{aligned}  
      \tag{4.6} 
   $$

   **考虑约束条件 $\sum \limits_{k=1}^{K} a_k = 1$，使用拉格朗日算子**

   $$
   \begin{aligned} 
      Q'(\theta, \theta^{(i)}) & = \sum \limits_{k=1}^{K} \Big( \hat n_k \cdot \log a_k + \sum \limits_{j=1}^{N} \hat r_{jk} \cdot \big( -\frac{1}{2} \log 2\pi - \frac{1}{2} \log \sigma_k^2 - \frac{(y_j - \mu_k)^2}{2\sigma_k^2}  \big) \Big)  + \lambda (\sum \limits_{k=1}^{K} a_k - 1) \\


   \end{aligned}  
   \tag{4.7} 
   $$

7. 求导
   1. $\frac{\partial Q'}{\partial a_k} = 0$
      $$
      \begin{aligned} 
         \frac{\partial Q'}{\partial a_k} & = \sum \limits_{k=1}^{K} ( \frac{\hat n_k}{a_k} + \lambda) =0 \\
      \end{aligned}  
      $$       

      一个可行的解
      $$
      \begin{aligned} 
         \frac{\hat n_k}{a_k} + \lambda & = 0 \\
         \hat n_k & = - \lambda a_k \\ 
      \end{aligned}  
      \tag{4.8}
      $$
      
      对(4.8) 两边求和
      $$
      \begin{aligned} 
         N = \sum \limits_{k=1}^{K} \hat n_k & = - \lambda \sum \limits_{k=1}^{K} a_k = -\lambda\\ 
      \end{aligned} 
      $$

      因此$\hat \lambda = - N$, 带入(4.8)可得到
      $$\hat a_k = \frac{\hat n_k}{N} $$
   
   2. $\frac{\partial Q'}{\partial \mu_k} = 0$
      $$
      \begin{aligned} 
         \frac{\partial Q'}{\partial \mu_k} & = \sum \limits_{k=1}^{K} \sum \limits_{j=1}^{N} \hat r_{jk} \cdot \frac{(y_j - \mu_k)}{\sigma_k^2} = 0 \\

         & = \sum \limits_{k=1}^{K} \frac{1}{\sigma_k^2} \sum \limits_{j=1}^{N} ( \hat r_{jk} y_j - \hat r_{jk} \mu_k) \\

         & = \sum \limits_{k=1}^{K} \frac{1}{\sigma_k^2} ( \sum \limits_{j=1}^{N} \hat r_{jk} y_j - \mu_k \hat n_{k})
      \end{aligned} 
      $$

      一个可行的解,
      $$
         \hat \mu_k = \frac{\sum \limits_{j=1}^{N} \hat r_{jk} y_j}{\hat n_{k}}
      $$

   3. $\frac{\partial Q'}{\partial \sigma_k^2} = 0$
      $$
      \begin{aligned} 
         \frac{\partial Q'}{\partial \sigma_k^2} & = \sum \limits_{k=1}^{K} \sum \limits_{j=1}^{N} \hat r_{jk} \cdot (-\frac{1}{\sigma_k^2} + \frac{(y_j - \mu_k)^2}{2\sigma_k^4}) = 0 \\
      \end{aligned} 
      $$

      令$t = \sigma_k^2$, 则
      $$
      \begin{aligned} 
         \frac{\partial Q'}{\partial t} & = \sum \limits_{k=1}^{K} \sum \limits_{j=1}^{N} \hat r_{jk} \cdot (-\frac{1}{t} + \frac{(y_j - \mu_k)^2}{2t^2}) = 0 \\
         
         & = \sum \limits_{k=1}^{K} \frac{\sum \limits_{j=1}^{N} \hat r_{jk}(y_j - \mu_k)^2 - \sum \limits_{j=1}^{N} \hat r_{jk} t}{2t^2}
      \end{aligned} 
      $$

      一个可能的解(此时自动使用$\hat\mu_k$？)
      $$
         \hat t = \hat \sigma_k^2 = \frac{\sum \limits_{j=1}^{N} \hat r_{jk}(y_j - \hat\mu_k)^2}{\sum \limits_{j=1}^{N} \hat r_{jk}} 

         = \frac{\sum \limits_{j=1}^{N} \hat r_{jk}(y_j - \hat\mu_k)^2}{\hat n_{k}}
      $$


# 参考
1. https://www.bilibili.com/video/BV1me4y1j7Uz?p=1