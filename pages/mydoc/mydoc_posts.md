---
title: LoadBalance(负载均衡) 
last_updated: Feb 25, 2016
summary: "TCP路由，网关协议转发等内容收录在此。"
sidebar: mydoc_sidebar
permalink: mydoc_posts.html
folder: network
---

## 2021.07.13 我们是这样崩的

```yaml
研发配置错误权重，七层SLB获取权重路由时有代码bug，lua脚本判断时陷入死循环, SLB机器CPU打满，下游服务无法接受响应
```
[原文链接](https://mp.weixin.qq.com/s?__biz=Mzg3Njc0NTgwMg==&mid=2247487272&idx=1&sn=038a30ce61706c97e3397eee982b1486&scene=21#wechat_redirect )

[拓展阅读:负载均衡的前世今生](https://cloud.tencent.com/developer/article/1458765)

{% include links.html %}
