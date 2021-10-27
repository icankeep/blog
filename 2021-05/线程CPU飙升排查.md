
## 迅速保存现场
```bash
top
jstack -l [pid]
```

```bash
#!/bin/bash

# 入参只有一个，即目标java的pid，如果没有，则默认找cpu最高的java进程
if [ -z "$1" ]; then
        ### 1.先找到消耗cpu最高的Java进程 ###
        pid=`ps -eo pid,%cpu,cmd --sort=-%cpu | grep java | grep -v grep | head -1 | awk 'END{print $1}' `
        if [ "$pid" =  ""  ]; then
                echo "无Java进程，退出。"
                exit
        fi
else
        pid=$1
fi

### 2.生成dump后的文件名 ###
curTime=$(date +%Y%m%dT%H:%M:%S)
# jstack后的文件会加上时间，便于对一个进程dump多次
dumpFilePath="/tmp/pid-$pid-$curTime.jstack"
echo -e "cpu最高的java进程: "`jps | grep $pid`"\n" > $dumpFilePath

### 3.取到该进程的所有线程及其cpu(只显示cpu大于0.0的线程) ###
echo -e "进程内线程cpu占比如下（不显示cpu占比为0的线程）:\n" >> $dumpFilePath
ps H -eo pid,tid,%cpu --sort=-%cpu | grep $pid | awk '$3 > 0.0 {totalCpu+=$3; printf("nid=0x%x, cpu=%s\n", $2, $3) >> "'$dumpFilePath'"} 
END{printf("cpu总占比:%s\n\n", totalCpu) >> "'$dumpFilePath'"}'

### 4.dump该进程 ###
echo -e "如下是原生jstack后的结果:\n" >> $dumpFilePath
jstack -l $pid >> $dumpFilePath

echo "dump成功，请前往查看(文件名包含时间，为了采集更准确，可以多执行几次该命令):" $dumpFilePath
exit
```


```bash
jmap -dump:live,format=b,file=<fileName> <pid>
jmap -histo:live <pid>
```