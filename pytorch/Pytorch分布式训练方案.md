---
title: pytorch多卡训练
date: 2023-04-30 00:04:01
categories: 
- [笔记, pytorch多卡训练]
mathjax: true
---

# 1. pytorch多卡训练

在Ubuntu 20.04上，探索多块显卡的分布式训练

以下代码 pytorch 1.10 和 pytorch 2.0 均通过测试

<!--more-->

## 1.1 前置准备

### 1.1.1 禁用Ubuntu防火墙

==禁用Ubuntu防火墙！== 否则会导致`master_port`通信问题，以及后期NCCL基于`socket`通信问题

```bash
sudo ufw disable
```

### 1.1.2 ssh设置免密登录

1. 两台服务器均生成ssh-key

   ```
   ssh-keygen -t rsa
   ```

2. 若主机1的公钥名为`192_168_1_3_id_rsa.pub`

   在主机2上执行

   ```bash
   cd ~/.ssh
   scp WORK@192.168.1.3:/home/WORK/.ssh/id_rsa.pub 192_168_1_3_id_rsa.pub
   cat 192_168_1_3_id_rsa.pub >> authorized_keys
   ```

   此时主机1可以通过ssh免密访问主机2

   对主机1做同样操作

## 1.2 单机多卡测试

### 1.2.1 测试程序

参看：https://zhuanlan.zhihu.com/p/74792767

```python
# single_node.py
import torch
import torch.nn as nn
from torch.utils.data import Dataset, DataLoader
import os
from torch.utils.data.distributed import DistributedSampler

class RandomDataset(Dataset):
    def __init__(self, size, length):
        self.len = length
        self.data = torch.randn(length, size).to('cuda')

    def __getitem__(self, index):
        return self.data[index]

    def __len__(self):
        return self.len


class Model(nn.Module):
    def __init__(self, input_size, output_size):
        super(Model, self).__init__()
        self.fc = nn.Linear(input_size, output_size)

    def forward(self, input):
        output = self.fc(input)
        print("  In Model: input size", input.size(),
              "output size", output.size())
        return output

def main():
    # 1) 初始化
    torch.distributed.init_process_group(backend="nccl")

    input_size = 5
    output_size = 2
    batch_size = 30
    data_size = 90

    # 2） 配置每个进程的gpu
    local_rank = torch.distributed.get_rank()
    torch.cuda.set_device(local_rank)
    device = torch.device("cuda", local_rank)
    dataset = RandomDataset(input_size, data_size)

    # 3）使用DistributedSampler
    rand_loader = DataLoader(dataset=dataset,
                            batch_size=batch_size,
                            sampler=DistributedSampler(dataset))
    
    model = Model(input_size, output_size)

    # 4) 封装之前要把模型移到对应的gpu
    model.to(device)

    if torch.cuda.device_count() > 1:
        print("Let's use", torch.cuda.device_count(), "GPUs!")
        # 5) 封装
        model = torch.nn.parallel.DistributedDataParallel(model,
                                                        device_ids=[local_rank],
                                                        output_device=local_rank)

    for data in rand_loader:
        if torch.cuda.is_available():
            input_var = data
        else:
            input_var = data

        while (True):
            output = model(input_var)
            print("Outside: input size", input_var.size(), "output_size", output.size())


if __name__ == "__main__":
    # os.environ["CUDA_VISIBLE_DEVICES"] = "1"
    main()
```



### 1.2.2 运行脚本

建议脚本

```bash
python -m torch.distributed.launch --nproc_per_node=4  --master-port=29522 single_node.py
```

参数说明：

- nproc_per_node：该节点并行几个线程（有几块GPU）
- master-port：端口号
- master_addr：ip地址通常是127.0.0.1，指向本机。但默认就是指向本机，所以不用设置

最简脚本

```bash
python -m torch.distributed.launch --nproc_per_node=4  single_node.py
```

注意：也可以把`python -m torch.distributed.launch`替换为`torchrun`

```bash
torchrun --nproc_per_node=4  --master-port=29522 single_node.py
```





## 1.3 多机多卡测试

node0 和 node1 各有1张显卡

### 1.3.1 测试程序

参看：https://zhuanlan.zhihu.com/p/486130584

```python
# multi_node.py
import argparse
import os
import sys
import tempfile
import datetime
import torch

import torch.distributed as dist
import torch.nn as nn
import torch.optim as optim

from time import sleep
from urllib.parse import urlparse
from torch.nn.parallel import DistributedDataParallel as DDP

class ToyModel(nn.Module):
    def __init__(self):
        super(ToyModel, self).__init__()
        self.net1 = nn.Linear(10, 10)
        self.relu = nn.ReLU()
        self.net2 = nn.Linear(10, 5)

    def forward(self, x):
        return self.net2(self.relu(self.net1(x)))


def train():
    print("!!! [train]: 0")
    local_rank = int(os.environ["LOCAL_RANK"])
    rank = int(os.environ["RANK"])
    while True:
        print(f"[{os.getpid()}] (rank = {rank}, local_rank = {local_rank}) training...")
        model = ToyModel().cuda(local_rank)
        print("!!! [train][while]: 0")
        ddp_model = DDP(model, [local_rank])
        print("!!! [train][while]: 1")
        loss_fn = nn.MSELoss()
        optimizer = optim.SGD(ddp_model.parameters(), lr=0.001)
        optimizer.zero_grad()
        print("!!! [train][while]: 2")
        outputs = ddp_model(torch.randn(20, 10).to(local_rank))
        print("!!! [train][while]: 3")
        labels = torch.randn(20, 5).to(local_rank)
        loss = loss_fn(outputs, labels)
        loss.backward()
        print(f"[{os.getpid()}] (rank = {rank}, local_rank = {local_rank}) loss = {loss.item()}\n")
        optimizer.step()
        sleep(1)

def run():
    print("!!! [run]: 0")
    env_dict = {
        key: os.environ[key]
        for key in ("MASTER_ADDR", "MASTER_PORT", "WORLD_SIZE", "LOCAL_WORLD_SIZE")
    }
    print(f"[{os.getpid()}] Initializing process group with: {env_dict}")
    dist.init_process_group(backend="nccl", timeout=datetime.timedelta(seconds=30))
    #dist.init_process_group(backend="nccl")
    print("!!! [run]: 1")
    train()
    print("!!! [run]: 2")
    dist.destroy_process_group()
    print("!!! [run]: 3")


if __name__ == "__main__":
    # os.environ["CUDA_VISIBLE_DEVICES"] = "1"
    print("!!! This is Node1")
    run()
```



### 1.3.2 运行脚本

node0 (主节点) 执行

```bash
python -m torch.distributed.launch --nproc_per_node 1 --nnodes 2 --node_rank 0 --master_addr='192.168.1.2' --master_port='29521' multi_node.py
```

node1 执行

```bash
python -m torch.distributed.launch --nproc_per_node 1 --nnodes 2 --node_rank 1 --master_addr='192.168.1.2' --master_port='29521' multi_node.py
```

参数说明

- `nproc_per_node`：每个节点启动进程数量
- `nnodes`：节点总数
- `node_rank`：当前节点编号
- `master_addr`：主节点IP地址
- `master_port`：主节点端口号

另外，`python -m torch.distributed.launch ...` 可理解为将`torch.distributed.launch`当做脚本运行，后面都是传入的参数



#### 1.3.2.1 使用新版本的torchrun

以node1为例，此时执行脚本应写为

```bash
torchrun --nproc_per_node 1 --nnodes 2 --node_rank 0 --master_addr='192.168.1.2' --master_port='29521' multi_node.py
```

但注意，此时python程序中获取`lock_rank`的方式必须是

```python
local_rank = int(os.environ["LOCAL_RANK"])
```

详见：https://pytorch.org/docs/stable/elastic/run.html

客观的评价，`torch.distributed.launch`输出信息更多，推荐`torch.distributed.launch`



### 1.3.3 指定显卡

假设node0上有四张显卡，现在想调用GPU2

在终端中（至少在VSCode集成终端中），输入`CUDA_VISIBLE_DEVICES=0`无效。

**有效的方法**是在`multi_node.py`中的`main`中设定`os.environ["CUDA_VISIBLE_DEVICES"] = "2"`

理论上使用os设置和在终端中直接设置效果相同，但实际上不同，原因未知。



### 1.3.4 释放端口

有时程序未运行结束，用户强行结束。此时所用端口仍未释放。

查找占用29521端口号的程序的pid

```bash
lsof -i:29521
```

然后执行

```bash
sudo kill -9 <pid>
```



### 1.3.5 NCCL相关

NCCL是Nvidia显卡间通信的底层软件或协议，已经集成到pytorch中，==无需额外安装==。

NCCL官网：https://developer.nvidia.com/nccl/getting_started

Pytorch对于NCCL集成解释：https://pytorch.org/docs/stable/distributed.html?highlight=nvidia



但有时分布式训练启动会失败，分析原因可能涉及到NCCL，此时需要NCCL打印详细信息。可以添加临时变量，再次运行训练脚本。

```bash
export NCCL_DEBUG=info
export NCCL_SOCKET_IFNAME=eth0  
export NCCL_IB_DISABLE=1
```



# 参考

1. https://blog.csdn.net/m0_37426155/article/details/108129952
1. https://zhuanlan.zhihu.com/p/486130584
3. https://blog.csdn.net/magic_ll/article/details/122359490

