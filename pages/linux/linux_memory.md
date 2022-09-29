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




## 一次Java进程OOM的排查分析(glibc)

```yaml
背景:
Linux的RES大于实际虚拟机申请内存
  
记录:  
1. 条件反射往堆外内存方向去考虑, 甚至无需引入NativeMemoryTracking
2. Arthas的dashboard可视化方式展现虚拟机内存分布, 还挺直观
3. 堆外内存直接先用pmap -x来确认, 这里发现了glibc的64M的arena经典问题, 4核32个, 8核64个
4. 网上传的MALLOC_ARENA_MAX=1这个设置看起来没用, 不作为优先选项
5. 使用jcmd pid GC.run可以强制触发一次GC
6. 该场景属于glibc内部机制的问题，glibc并未回收内存而是放到了自己内部的unsortbin中打算二次利用
 
解决方案 
1. 更换内存分配为jemalloc或tcmalloc。
2. 个人觉得生产环境还是慎重, 资源充足情况下可先尝试内存分配扩容
```
[原文链接](https://juejin.cn/post/6854573220733911048)




## 一次Java进程OOM的排查分析(hibernate)

```yaml
背景:
Linux的RES大于实际虚拟机申请内存;Hibernate老版本;
  
记录:  
1. 条件反射往堆外内存方向去考虑, 甚至无需引入NativeMemoryTracking 
2. 堆外内存直接先用pmap -x来确认, 这里发现了glibc的64M的arena经典问题, 4核32个, 8核64个
3. 可以利用gperftools来生成内存分配释放图(经验足可绕过该步骤)
4. dump内存找一找有无残留的不可达Inflater对象, 没有就是glibc这个二道贩子，否则要再看看
5. 原文用jstack, 我觉得重启vm然后arthas的stack命令也能做到这点
6. Hibernate的锅 

解决方案 
1. 新版本Hibernate修复了,升级
2. 别用Hibernate

```
[原文链接](https://heapdump.cn/article/3530243)




## SpringBoot引起的堆外内存泄漏

```yaml
背景:
Linux的RES大于实际虚拟机申请内存;
  
记录:  
1. pmap -x 29776 | sort -k 2 -n -r, 排序一下是个好习惯
2. gdb并attach上后, dump memory mem.bin startAddr endAddr可以将内存dump到本地
3. 对于dump下来的内存可以通过strings命令看下大概长啥样
4. 可以项目启动时使用strace看看申请内存明细, strace -f -e”brk,mmap,munmap, 免去LD_PRELOAD的麻烦
5. 然后用jstack速度去抓线程在干啥。这一步骤可能要快点，慢点就看不到现场了
6. 依然是glibc这个二道贩子把内存截胡了

解决方案:
1. 我有点怀疑美团是否真的弄清楚问题的本质了, 如果是glibc的问题, springboot升级有何用? 

```
[原文链接](https://tech.meituan.com/2019/01/03/spring-boot-native-memory-leak.html)
  
{% include links.html %}
