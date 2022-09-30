---
title: GC
last_updated: Feb 25, 2016
summary: "垃圾回收。"
sidebar: mydoc_sidebar
permalink: jvm_gc.html
folder: jvm
---

## 命令行生成MAT分析报告

```yaml
背景:
dump出来的堆太大, MAT无法打开;

记录:
1. MAT在MAC下的安装目录基本在/Applications/mat.app/Contents/Eclipse, 新建下方脚本
2. 调整MemoryAnalyzer.ini中的内存到适当位置

脚本:
./ParseHeapDump.sh \
/Users/zhangyingchao/Documents/dump/my.dump  \
org.eclipse.mat.api:suspects \
org.eclipse.mat.api:overview \
org.eclipse.mat.api:top_components
```

[原文链接](https://www.cnblogs.com/hellxz/p/use_mat_linux_command_line_generate_reports.html)




## 从一起GC血案谈到反射原理

```yaml
背景:
JDK7；G1垃圾回收; FGC;

原因:
G1在JDK7下只有FGC才会进行PermGen中的类卸载。该系统流量大, G1GC频繁, 同时系统设置了-XX:SoftRefLRUPolicyMSPerMB=0,
这意味着软引用会被快速回收掉。而该系统XFire协议下反序列化使用的是Method.invoke逻辑, 相关的Method就是软引用。默认15次
调用后会生成新的字节码类, 本来这个步骤虚拟机是为了加速, 但由于软引用会被快速回收, 又进入下一个15次调用轮回, 久而久之会在
PermGen驻留较多的DelegateClassloader, 以及加载好的MethodAccessor。同时, 因为NativeMethodAccessorImpl的处理是
非线程安全的, 这里会有多个线程同时生成MethodAccessor和DelegateClassloader的可能性, 加剧了问题的发生。

记录:
1. -XX:SoftRefLRUPolicyMSPerMB.  这个选项没找到好的中文解释，但是设置为0会导致软引用快速被回收, 也算是本文的核心诱因。
2. 每次调用getDeclaredMethod其实会新生成一份Method对象的copy。所以可以业务缓存起来
3. 默认反射调用制定方法15次后(native), 底层会基于asm生成高性能的字节码版本, 后续会基于这个版本运行。
4. 上述15这个值可以通过-Dsun.reflect.inflationThreshold=x或者-Dsun.reflect.noInflation=true来指定。

解决方案:
1. 升级到JDK8+
2. 避免上述场景的XFire协议
3. SoftRefLRUPolicyMSPerMB搞大点
```

[原文链接](https://heapdump.cn/article/54786?from=pc)



## HBase问题诊断–RegionServer宕机

```yaml
背景:
FGC;HBase;CMS;

原因:
不是个好的Case。着重说明了面对FGC联动引发的链接被动关闭问题是如何解决的(挂起线程, 导致和下游交互的线程也被挂起), 但是我没看到FGC
发生的本质是啥

记录:
1. fgc(concurrent mode failure)。意味着虚拟机还未执行完本次GC的情况下又来了大量数据导致内存不够, 虚拟机被迫挂起所有线程FGC
2. -XX:CMSInitiatingOccupancyFraction=60意味着CMGC会在老年代使用达到60%的时候触发一次CMSGC.

疑问:
1. 还是没说明流量为什么大的原因? 是我看漏了么?
```

[原文链接](http://hbasefly.com/2016/04/15/hbase-regionserver-crash/)

{% include links.html %}
