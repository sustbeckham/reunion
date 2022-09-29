---
title: concurrent
last_updated: Feb 25, 2016
summary: "多线程"
sidebar: mydoc_sidebar
permalink: java_concurrent.html
folder: java
---

## 记一次故障引发的线程池使用的思考

```yaml
背景:
线程池;

原因:
例案中dubbo单机200的连接池，偶发超过200的qps进来但是请求的是同一个接口，这个接口又开了连接池调用三方接口，结果三方超时，新开的连接池
满了都在等。从而把上层的连接也给堵住了。

记录: 

解决方案: 
```

[原文链接](https://tech.youzan.com/ji-ci-gu-zhang-yin-fa-de-xian-cheng-chi-shi-yong-de-si-kao/)

{% include links.html %}
