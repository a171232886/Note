---
title: Ubuntu下查看CPU、内存和硬盘详细信息的几个命令
date: 2021-11-19 10:09:16
categories: 
- [笔记, Ubuntu、Win10与服务器]
---

转载自https://www.cnblogs.com/shixiangwan/p/7066085.html

CPU：
型号：`grep "model name" /proc/cpuinfo |awk -F ':' '{print $NF}'`

内存数量：`sudo dmidecode -t memory |grep -A16 "Memory Device$" |grep 'Size:.*MB' |wc -l`

内存支持类型：`sudo dmidecode -t memory |grep -A16 "Memory Device$" |grep "Type:"`

每个内存频率：`sudo dmidecode -t memory |grep -A16 "Memory Device$" |grep "Speed:"`

每个内存大小：`sudo dmidecode -t memory |grep -A16 "Memory Device$" |grep "Size:"`

硬盘：
硬盘数量、大小：`sudo fdisk -l |grep "Disk /dev/sd"`

硬盘型号：`sudo hdparm -i /dev/sda |grep "Model"`