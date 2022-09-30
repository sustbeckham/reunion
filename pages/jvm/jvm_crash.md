---
title: Crash
last_updated: Feb 25, 2016
summary: "Java进程突然就消失了..."
sidebar: mydoc_sidebar
permalink: jvm_crash.html
folder: jvm
---

## 协助美团kafka团队定位Crash问题

```yaml
背景:
JDK7；G1垃圾回收; FGC; CRASH

原因:
Kafka的broker会对消息文件做mmap。每个文件占用内存还很大(1G), 逐渐到达物理上限。当最终物理内存不足却要索引下一个1G的文件时，由于映射失败虚拟机
尝试一次FGC, 而在FGC过程中发现Perm和MaxPerm不一致会对Perm扩容。扩容时候又调用了一次mmap(其实就是向操作系统申请分配内存)，而操作系统本身会
校验是否到达了映射上限值(/proc/sys/vm/max_map_count), 超出则会导致crash。 

记录:
1. Crash的时候看到err文件不要被开头的内存不足给骗了，留个心眼看下最后的实际内存使用。该case实际还有1G的内存没使用, 但开头提示192k都分配不了。
2. err文件的调用栈也要留意, 比如如果是GC引起的crash可以看到是哪种具体类型的FGC, 比如此处的VM_GenCollectFull。
3. 还是调用栈的问题, 源码如果读多了能反应过来在扩容(expand_by, grow_by)。
4. old区域扩容不太可能, 因为现在的开发条件反射都会配置xms和xmx相同
5. JDK7下, 如果未设置, 默认的PermSize和MaxPermSize并不相同, 可能会引起扩容, 扩容的本质就是mmap(申请内存)
6. 还是代码熟练度的问题, hotspot下全局搜索[->collect]能看到匹配VM_GenCollectFull的只有jvmti, gclocker, systemgc。其他场景可忽略。
7. GCLocker的本质是是别的任务在safepoint, GC的请求来了会被忽略, 当别的任务结束会补偿你一次GC
8. MaxDirectMemorySize的大小如果不设置, 基本和当前堆的大小一致(jvm.cpp#JVM_MaxMemory), 不同回收模型下有轻微差异。
9. err文件中的Internal exceptions部分也可以留意, 他反应了内部发生的一些异常信息。
10. 文中的OOM本质是在Java_sun_nio_ch_FileChannelImpl_map0处触发了max_map_count的上限, 被包装为了JNU_ThrowOutOfMemoryError
11. 但扩容的时候就没这么好运了(os::Linux::commit_memory_impl), 直接触发vm_exit_out_of_memory, 从而crash
12. 记得以后类似问题留意err文件的动态链接库内容, 比如cat hs_err_pid28608.log | grep 'Dynamic libraries' -A 75, 不断调整数字确认
13. RandomAccessFile.getChannel().map(FileChannel.MapMode.READ_WRITE, 0, 10)就可以动态映射, 但在MacOS环境下不能重现

解决方案: 
1. 调整/proc/sys/vm/max_map_count的大小。ElasticSearch在启动时就会建议调大该值。

基础学习:
1. 每个进程都有独立的虚拟地址空间，进程访问的虚拟地址并不是真正的物理地址
2. 虚拟地址可通过每个进程上的页表和物理地址进行映射，获得真正的物理地址
3. 若页表中找不到会产生缺页中断，分配真正的物理地址同时更新进程页表。如果此时物理内存耗尽，会基于内存替换算法淘汰部分页面到物理磁盘上
4. malloc的底层是brk(<128k)或者mmap(>=128k)
5. brk的本质是推_edata指针，brk分配的内存需要等高地址内存释放后才能释放，较为被动; mmap可通过free自由释放
6. mmap有2个维度的功能，1是向操作系统分配一块内存(fd=-1), 2是映射具体文件(具体fd值)
```

[原文链接](https://heapdump.cn/article/96265)

[Linux内存分配小结](https://www.cnblogs.com/sky-heaven/p/10005642.html)

[mmap介绍](https://www.cnblogs.com/arnoldlu/p/8330785.html)

[linux源码阅读](https://elixir.bootlin.com/linux/v5.5-rc2/source)

{% include links.html %}
