---
title: Out-Of-Memory
last_updated: Feb 25, 2016
summary: "Java虚拟机内存耗尽。"
sidebar: mydoc_sidebar
permalink: jvm_oom.html
folder: jvm
---

## 记一次dubbo服务发现导致的OOM

```yaml
背景:
  DUBBO;MAT;OOM

记录:
  1. 很典型的OOM排查过程
  2. 一个细节是这种情况最好dump内存看, jmap -histo不一定能发现真实问题
  3. dominator_tree页面看到大对象问题的时候, 其实排查就已经结束了。
  4. 大对象RestProtocol的堆积是问题的本质

解决方案:
  1. 更新DUBBO版本
```
[原文链接](https://tech.youzan.com/ji-ci-dubbofu-wu-fa-xian-dao-zhi-de-oom/)

{% include links.html %}
