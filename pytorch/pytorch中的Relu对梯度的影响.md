---
title: Pytorch中的Relu对梯度的影响
date: 2021-12-29 21:39:11
categories: 
- [笔记, Python，Numpy和Pytorch]
---


# Relu层
卷积层和全连接层这种有可训练参数的，可以求梯度。Relu层怎么办？

**先说结论：Relu(x)，若x<=0，x的梯度为0，若x>0，梯度为x。**

<!--more-->

## 代码测试验证
测试一个简单的网络，pred=relu(WX)，loss = mean(pred - y)。不考虑Relu时，W的梯度应是X。

```python
import torch
import torch.nn as nn


def seed_torch(seed=0):
    # random.seed(seed)
    # os.environ['PYTHONHASHSEED'] = str(seed)
    # np.random.seed(seed)
    torch.manual_seed(seed)
    torch.cuda.manual_seed(seed)
    torch.backends.cudnn.benchmark = False
    torch.backends.cudnn.deterministic = True

class Model(nn.Module):
    def __init__(self):
        super(Model, self).__init__()
        self.relu = nn.ReLU()
        self.para = torch.nn.Parameter(torch.ones([3, 1]))

    def forward(self, x, y):
        pred = self.relu(torch.mm(x, self.para))
        loss = torch.mean(pred - y)
        return pred, loss

if __name__ == '__main__':
    seed_torch(12)
    x = torch.ones([4, 3])
    # x[:, 1] = torch.zeros([4])
    # x[:, 2] = -torch.ones([4])
    y = torch.rand([4, 1])

    model = Model()
    p, l = model(x, y)

    l.backward()
    print(x)
    # print(y)
    print(x.grad)
    print(model.para.grad)

```

 1. 第一种情况，WX>0
     当``x = torch.ones([4, 3])``，输出
	```bash
	tensor([[1., 1., 1.],
	        [1., 1., 1.],
	        [1., 1., 1.],
	        [1., 1., 1.]])
	None
	tensor([[1.],
	        [1.],
	        [1.]])
	```
  2. 第二种情况，WX=0
     当 
	   ```
	    x = torch.ones([4, 3])
	    x[:, 1] = torch.zeros([4])
	    x[:, 2] = -torch.ones([4])
	    ```
	    输出
	

		```bash
		tensor([[ 1.,  0., -1.],
		        [ 1.,  0., -1.],
		        [ 1.,  0., -1.],
		        [ 1.,  0., -1.]])
		None
		tensor([[0.],
		        [0.],
		        [0.]])
		```

  3. 第二种情况，WX<0
       当 
	   ```
	    x = torch.ones([4, 3]) *(-1)
	    ```
	    输出
	    
		```bash
		tensor([[-1., -1., -1.],
		        [-1., -1., -1.],
		        [-1., -1., -1.],
		        [-1., -1., -1.]])
		None
		tensor([[0.],
		        [0.],
		        [0.]])
		```