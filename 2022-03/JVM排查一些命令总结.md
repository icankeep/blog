## 参考文档
https://www.hollischuang.com/archives/308

## 查看Java进程
```bash
jps
```

```bash
ps -ef | grep java
```

## 查看线程堆栈

```bash
jstack -l <java_pid>
```

## 查看堆配置详情
```bash
jmap -heap 57296
```

```bash
Attaching to process ID 57296, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.45-b02

using thread-local object allocation.
Garbage-First (G1) GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 4294967296 (4096.0MB)
   NewSize                  = 1363144 (1.2999954223632812MB)
   MaxNewSize               = 2575302656 (2456.0MB)
   OldSize                  = 5452592 (5.1999969482421875MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 268435456 (256.0MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 268435456 (256.0MB)
   G1HeapRegionSize         = 4194304 (4.0MB)

Heap Usage:
G1 Heap:
   regions  = 1024
   capacity = 4294967296 (4096.0MB)
   used     = 1466836160 (1398.8839721679688MB)
   free     = 2828131136 (2697.1160278320312MB)
   34.15244072675705% used
G1 Young Generation:
Eden Space:
   regions  = 327
   capacity = 2688548864 (2564.0MB)
   used     = 1371537408 (1308.0MB)
   free     = 1317011456 (1256.0MB)
   51.014040561622465% used
Survivor Space:
   regions  = 4
   capacity = 16777216 (16.0MB)
   used     = 16777216 (16.0MB)
   free     = 0 (0.0MB)
   100.0% used
G1 Old Generation:
   regions  = 20
   capacity = 1589641216 (1516.0MB)
   used     = 78521536 (74.88397216796875MB)
   free     = 1511119680 (1441.1160278320312MB)
   4.939576000525643% used

43667 interned Strings occupying 4590256 bytes.
```

## dump堆内存
```bash
jmap -dump:[live,]format=b,file=<filename> 57296
```

```bash
jmap -histo 57296
```

## gc情况
```bash
# jstat -gcutil <java_pid> <interval_time> <count>
jstat -gcutil 57296 1000 5
```

## 查看虚拟机一些参数配置
```bash
jinfo -flags <java_pid>
```
