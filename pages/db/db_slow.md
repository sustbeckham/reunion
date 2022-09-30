---
title: SLOW-QUERY
last_updated: Feb 25, 2016
summary: "慢查询..."
sidebar: mydoc_sidebar
permalink: db_slow.html
folder: slow
---

## 编码导致的不走索引问题

```yaml
背景:
MYSQL;未走索引;
  
记录:  
1. optimizer_trace可以用来发现具体SQL执行的明细(但一般很难去DB上直接这么玩)。
2. 字段外层套一层函数是走不到索引的，如文中的CONVERT。
3. 文中的2个表编码不一致, 其中一个用了utf8mb4(emoji支持), 所以才造成了这个局面
 
解决方案: 
1. 更新字段编码
```
[原文链接](https://zhuanlan.zhihu.com/p/397271448)

{% include links.html %}
