---
title: pysot和vot-toolkit的结合
date: 2022-01-19 22:24:13
categories: 
- [笔记, 目标跟踪]
---

# 前言
能点进来看这个文章的，想必都是行内人士，那就直接讲重点。

其实人家pysot已经内置了vot数据集的测试和分析，其结果已经广泛用于各种paper上。比如SiamRPN++在VOT2016上的EAO是0.464。

但pysot只能运行VOT2016,2018,2019的数据集。因为pysot只提供了关于这几个的配置信息。于是我想把pysot和vot-toolkit结合起来。

关于vot-toolkit-python的使用，可以看我的上一篇博客[vot-toolkit-python的使用](https://blog.csdn.net/a171232886/article/details/122575852)。

<!--more-->

# 代码
这是pysot和vot-toolkit结合的python脚本，我将其命名为```test_vot.py```。
<font face="黑体"  color=#FF0000 size=4> 注意，这个文件放在```pysot-master```下面，而不是```pysot-master/tools```下面。</font>这样可回避pysot复杂的环境变量设置。

```python
# Copyright (c) SenseTime. All Rights Reserved.

from __future__ import absolute_import
from __future__ import division
from __future__ import print_function
from __future__ import unicode_literals

import argparse
import os
import sys

pysotpath = '/media/HardDisk_new/wh/others/pysot-master/'

import cv2
import torch
import numpy as np

from pysot.core.config import cfg
from pysot.models.model_builder import ModelBuilder
from pysot.tracker.tracker_builder import build_tracker
from pysot.utils.model_load import load_pretrain
from pysot.utils.bbox import get_axis_aligned_bbox
from vot_iter import vot

parser = argparse.ArgumentParser(description='siammask tracking')
parser.add_argument('--config', default='{}/experiments/siamrpn_r50_l234_dwxcorr/config.yaml'.format(pysotpath),
                    type=str, help='config file')
parser.add_argument('--snapshot', default='{}/experiments/siamrpn_r50_l234_dwxcorr/model.pth'.format(pysotpath),
                    type=str, help='snapshot of models to eval')
args = parser.parse_args()

torch.set_num_threads(1)
os.environ["CUDA_VISIBLE_DEVICES"] = "0"


def main():
    # load config
    cfg.merge_from_file(args.config)

    # create model
    model = ModelBuilder()

    # load model
    model = load_pretrain(model, args.snapshot).cuda().eval()

    # build tracker
    tracker = build_tracker(model)


    # vot-tool
    handle = vot.VOT("polygon")
    selection = handle.region()

    imagefile = handle.frame()
    if not imagefile:
        sys.exit(0)

    # BBox格式转化
    gt_bbox = [selection.points[0].x, selection.points[0].y,
               selection.points[1].x, selection.points[1].y,
               selection.points[2].x, selection.points[2].y,
               selection.points[3].x, selection.points[3].y]
    cx, cy, w, h = get_axis_aligned_bbox(np.array(gt_bbox))
    gt_bbox_ = [cx - (w - 1) / 2, cy - (h - 1) / 2, w, h]


    image = cv2.imread(imagefile, cv2.IMREAD_COLOR)
    tracker.init(image, gt_bbox_)
    while True:
        imagefile = handle.frame()
        if not imagefile:
            break
        image = cv2.imread(imagefile, cv2.IMREAD_COLOR)
        output = tracker.track(image)
        region = vot.Rectangle(output['bbox'][0], output['bbox'][1],
                               output['bbox'][2], output['bbox'][3])
        confidence = output['best_score']
        handle.report(region, confidence)


if __name__ == '__main__':
    main()
```

# 修改trackers.ini文件
```shell
[SiamRPNpp]
label = SiamRPNpp
protocol = traxpython  
command = test_vot 
paths = /media/HardDisk_new/wh/others/pysot-master/
env_PATH = /home/wh/anaconda3/envs/others/bin/python;${PATH}   
restart = true
```

# 实验
## 结果
1. pysot运行的SiamRPN++在VOT2016上的EAO=0.464；vot-toolkit的结果为0.430
2. 将vot-toolkit的运行结果用pysot的```eval.py```分析EAO=0.439；vot-toolkit分析pysot的运行结果为EAO=0.454。

## 分析
pysot和vot-toolkit的差距很有可能是，pysot在计算EAO时直接自定义了帧数考虑范围，代码如下（```./tookilt/evaluation/eao_benchmark.py```）
```python
        # NOTE we not use gmm to generate low, high, peak value
        if dataset.name == 'VOT2019':
            self.low = 46
            self.high = 291
            self.peak = 128
        elif dataset.name == 'VOT2018' or dataset.name == 'VOT2017':
            self.low = 100
            self.high = 356
            self.peak = 160
        elif dataset.name == 'VOT2016':
            self.low = 108
            self.high = 371
            self.peak = 168
```

vot-toolkit-python的高级python语法的大量使用，阻止了我继续分析。只要肯花时间，这是肯定能研究清楚的。但我见过的论文，都是以pysot的0.464为准。什么时候真正要用vot-toolkit-python什么时候再继续分析吧。