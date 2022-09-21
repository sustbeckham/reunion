---
title: 内存
last_updated: Feb 25, 2016
summary: "操作系统维度的内存问题。"
sidebar: mydoc_sidebar
permalink: linux_memory.html
folder: linux
---

## 一次java内存top res高排查记录

```yaml
GOOD
1. RES高但dump下来实际很小, 可考虑怀疑堆外内存泄露
2. NMT开启: -XX:NativeMemoryTracking=[off | summary | detail], 默认关闭
3. pmap提供了进程的内存映射。能反映出进程的地址空间和内存状态信息。
4. pmap -x 28659 | sort -n -k3(第三列是RSS, 排序后看着更清楚)
5. 可以利用gcore pid导出core文件进行进一步分析
6. strings -10 core.pid > core.txt, 该命令可以把长度超过10的字符串拉出来

BAD
1. 实际上该文并未解决问题。排查过程最后的结论偏向于推测。 
```
[原文链接](https://heapdump.cn/article/4161842?from=pc)
  
{% include links.html %}
