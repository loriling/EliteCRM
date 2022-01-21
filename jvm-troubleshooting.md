# JVM 问题排查

**当tomcat（tomcat本身就是一个jvm的进程）卡死时候，第一时间收集相关日志。**

其中最关键的就是**线程堆栈dump日志(stack.log)**，cpu使用情况，内存使用情况的截图，如果出现OOM报错，则需要**内存使用信息日志(heap.log, histo-live.log)**，具体获取方式看下方详细说明。

[TOC]



## 1. jps 找jvm的pid

jdk自带命令：`jps -ml`

```shell
# 查看所有jvm进程，找到对应的pid
jps -ml


1393 com.aliyun.tianji.cloudmonitor.Application
1651 org.apache.catalina.startup.Bootstrap start
26115 sun.tools.jps.Jps -ml
18924 org.apache.catalina.startup.Bootstrap start
14383 -- process information unavailable
```

如果 jps信息里不能找到对应的pid，就用ps查看，根据关键词去找对应的进程信息

 `ps -ef|grep xxxx`



## 2. jinfo 查看当前jvm相关参数

`jinfo -flags pid` 用前面获取到的pid，获取jvm所有启动参数，查看相关启动配置是否正确，比如内存配置是否正确

```shell
jinfo -flags 1651

Attaching to process ID 1651, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.66-b17
Non-default VM flags: -XX:CICompilerCount=3 -XX:InitialHeapSize=262144000 -XX:MaxHeapSize=4164943872 -XX:MaxNewSize=1388314624 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=87031808 -XX:OldSize=175112192 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC 
Command line:  -Djava.util.logging.config.file=/apps/apache-tomcat-9.0.13-uat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Djava.security.egd=file:/dev/./urandom -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -Dcatalina.base=/apps/apache-tomcat-9.0.13-uat -Dcatalina.home=/apps/apache-tomcat-9.0.13-uat -Djava.io.tmpdir=/apps/apache-tomcat-9.0.13-uat/temp
```



## 3. jstack 收集线程堆栈信息

`jstack pid > 日志文件路径`

```shell
# 根据上面找到的pid，选择自己需要的那个，比如这里选了1651，收集堆栈dump，存到stack.log文件中
jstack 1651 > stack.log 
```



## 4. jmap 收集内存信息

1. 使用`jmap -heap pid`查看进程堆内存使用情况，包括使用的GC算法、堆配置参数和各代中堆内存使用情况

```shell
# 根据pid查看堆内存信息
jmap -heap 1651
# 或者写到日志文件
jmap -heap 1651 > heap.log


Attaching to process ID 1651, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.66-b17

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4164943872 (3972.0MB)
   NewSize                  = 87031808 (83.0MB)
   MaxNewSize               = 1388314624 (1324.0MB)
   OldSize                  = 175112192 (167.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 1384120320 (1320.0MB)
   used     = 385194968 (367.35054779052734MB)
   free     = 998925352 (952.6494522094727MB)
   27.829586953827828% used
From Space:
   capacity = 2097152 (2.0MB)
   used     = 852000 (0.812530517578125MB)
   free     = 1245152 (1.187469482421875MB)
   40.62652587890625% used
To Space:
   capacity = 2097152 (2.0MB)
   used     = 0 (0.0MB)
   free     = 2097152 (2.0MB)
   0.0% used
PS Old Generation
   capacity = 563085312 (537.0MB)
   used     = 299181944 (285.32213592529297MB)
   free     = 263903368 (251.67786407470703MB)
   53.13261376634878% used

65391 interned Strings occupying 7221632 bytes.
```

2. 使用`jmap -histo[:live] pid`查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象

```shell
# 直接查看存活的内存对象个数
jmap -histo:live 1651 | more
# 或者写到日志文件
jmap -histo:live 1651 > histo-live.log


num     #instances         #bytes  class name
----------------------------------------------
   1:        839279      130294192  [C
   2:        262144       20971520  org.apache.logging.log4j.core.async.RingBufferLogEvent
   3:          8287       20635104  [B
   4:        835886       20061264  java.lang.String
   5:        506920       16221440  java.util.HashMap$Node
   6:        420551       13457632  org.wltea.analyzer.dic.DictSegment
   7:        145925        7004400  java.util.HashMap
   8:         61617        6109480  [Ljava.util.HashMap$Node;
   9:        177042        5665344  [Lorg.wltea.analyzer.dic.DictSegment;
  10:         41106        5261568  com.elitekm.tree.Article
  11:        140493        4495776  java.util.LinkedList
  12:         47974        3981432  [Ljava.lang.Object;
  13:         11923        3854688  [I

```



## 5. top 指令 收集系统进程使用CPU信息和指定pid的进程的线程内存CPU使用情况

`top -Hp pid`  查看该进程下的线程信息

```shell
top -Hp 1651


top - 12:15:45 up 88 days, 13:13,  1 user,  load average: 0.42, 0.42, 0.51
Threads: 118 total,   0 running, 118 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.2 us,  2.2 sy,  0.0 ni, 92.2 id,  0.2 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem : 16266392 total,   408728 free,  9282448 used,  6575216 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  5038328 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                             
 1697 root      20   0 8088912   2.5g   7788 S  0.7 16.2 741:59.29 java                                                
 1670 root      20   0 8088912   2.5g   7788 S  0.3 16.2   8:48.65 java                                                
 1867 root      20   0 8088912   2.5g   7788 S  0.3 16.2  17:23.95 java                                                
 1890 root      20   0 8088912   2.5g   7788 S  0.3 16.2   2:41.21 java                                                
11981 root      20   0 8088912   2.5g   7788 S  0.3 16.2  11:20.70 java                                                
 1651 root      20   0 8088912   2.5g   7788 S  0.0 16.2   0:00.00 java                                                
 1652 root      20   0 8088912   2.5g   7788 S  0.0 16.2   0:30.19 java                                                
 1653 root      20   0 8088912   2.5g   7788 S  0.0 16.2  29:32.64 java                                                
 1654 root      20   0 8088912   2.5g   7788 S  0.0 16.2  29:31.78 java                                                
 1655 root      20   0 8088912   2.5g   7788 S  0.0 16.2  29:31.76 java
```

这里的PID就是线程号，如果某个占用内存或者CPU很高，就可以结合2中获取的线程堆栈信息查看具体这个线程是执行的什么方法



