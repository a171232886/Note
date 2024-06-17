---
title: md文件图片引用格式转换及下载
date: 2022-01-11 10:24:25
categories: 
- [笔记, Hexo]
---

# 前言
CSDN上的图片引用格式为```![]()```，对图片调整为居中显示并按比例缩放，可以很简单:在图片链接尾添加```#pic_center =80%x80%```

```markdown
![在这里插入图片描述](https://img-blog.csdnimg.cn/e0a68bdbd10f41f081bb1c60d659bda4.png#pic_center =80%x80%)
```

但问题在于这种方式不能使用于其他博客比如Hexo，所以需要转化成更通用的方式，即html的方式

```html
<div align="center"> 
<img src="https://img-blog.csdnimg.cn/e0a68bdbd10f41f081bb1c60d659bda4.png" width="80%"> 
</div> 
```

<!--more-->

图片过多，考虑使用代码转化。

另外，博客上的图片应该自己保留一份，考虑使用代码自动完成。

# 代码
于是，就有了负责转化功能的```convert```和负责下载图片的```download_img```。将以下程序置放于存储md文件的文件夹下即可。

```python
import os
import requests

def convert(md_path, origin_save_path='./origin'):
    '''
    将markdown文件中的！![]()图片格式变为div格式
    :param md_path: 源文件路径
    :return:
    '''

    if md_path.find('\\'):
        save_name = md_path.split('\\')[-1]
    elif md_path.find('/'):
        save_name = md_path.split('/')[-1]
    else:
        save_name = md_path


    with open(md_path, 'r', encoding='utf-8') as f:
        lines = f.readlines()

    if not os.path.exists(origin_save_path):
        os.makedirs(origin_save_path)

    with open(os.path.join(origin_save_path, save_name), 'w', encoding='utf-8') as f:
        for line in lines:
            f.write(line)

    for idx, line in enumerate(lines):
        if '![' in line:

            start = line.find('(')
            end = line.find(')')
            img_url = line[start+1:end]

            lines.insert(idx, '<div align="center"> \n')
            lines[idx+1] = '<img src="{img_url}" width="80%"> \n'.format(img_url=img_url)
            lines.insert(idx+2, '</div> \n')


    with open(save_name, 'w', encoding='utf-8') as f:
        for line in lines:
            f.write(line)

    return

def download_img(md_path, img_save_path='./img'):
    '''
    将markdown文件中的！![]()图片格式变为div格式
    :param md_path: 源文件路径
    :return:
    '''

    if md_path.find('\\'):
        save_name = md_path.split('\\')[-1]
    elif md_path.find('/'):
        save_name = md_path.split('/')[-1]
    else:
        save_name = md_path

    # 读取md文件
    with open(md_path, 'r', encoding='utf-8') as f:
        lines = f.readlines()

    img = None
    img_idx = 0  # 用于图片计数
    for idx, line in enumerate(lines):

        if '![' in line:
            start = line.find('(')
            end = line.find(')')
            img_url = line[start+1:end]

            # 下载
            img = requests.get(img_url)

        elif '<img src=' in line:
            img_url = line.split('"')[1]
            # 下载
            img = requests.get(img_url)

        # 保存
        if img is not None:
            img_save_path_ = os.path.join(img_save_path, save_name)
            if not os.path.exists(img_save_path_):
                os.makedirs(img_save_path_)

            if '.gif' in img_url:
                with open(os.path.join(img_save_path_, '{}.gif'.format(img_idx)),
                          'wb') as f:
                    f.write(img.content)

                    img_idx += 1
                    img = None
            else:
                with open(os.path.join(img_save_path_, '{}.png'.format(img_idx)),
                          'wb') as f:
                    f.write(img.content)

                    img_idx += 1
                    img = None
    return

if __name__ == '__main__':
    # md_path = 'Pytorch中的Conv1d和Conv3d.md'

    # 指定文件路径
    # os.chdir('./new')
    file_path = os.getcwd()
    md_paths = []
    # 得到Markdown文件列表
    for i in os.listdir(file_path):
        if '.md' in i:
            md_paths.append(os.path.join(file_path, i))

    for md_path in md_paths:
        # convert(md_path)
        download_img(md_path)
```