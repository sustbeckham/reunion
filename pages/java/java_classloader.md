---
title: ClassLoader
last_updated: Feb 25, 2016
summary: "类加载器"
sidebar: mydoc_sidebar
permalink: java_classloader.html
folder: java
---

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

{% include links.html %}
