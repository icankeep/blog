由于工作中经常用一些shell命令来解析日志和运维相关的工作，记录常用命令用于备忘

## grep

> 参考文档：
> [Linux命令：grep命令 | egrep命令](cnblogs.com/ysuwangqiang/p/11443785.html)

- 查找某关键词前后几行
```bash
#包含之后10行
grep -A 10 "demo" simple.log
#包含之前10行
grep -B 10 "demo" simple.log
#包含前后10行
grep -C 10 "demo" simple.log
```

- 递归查找目录下所有文件关键词

```bash
grep -rn '关键词' .
```

- 正则匹配某个关键词

```bash
# [I 2021-07-22 20:25:22.237 XX log:181] 302 GET /download/user/passer/test.txt?_xsrf=13sja -> https://demo.com/passer/1DF9E2B3S6E4X8A2S0T8L5.txt 
grep "download" simple.log | grep -Eo "[^/]+?xsrf=|[^/]+com"

 test.txt?_xsrf=
 demo.com
```

## awk
kill包括python关键字的进程
```bash
ps x | grep "python" | grep -v "grep" | awk '{print $1}' | xargs kill -9
```

## sed

匹配出日志中下载的文件名和对应用户名字
```bash
# [I 2021-07-22 20:25:22.237 XX log:181] 302 GET /download/user/passer/test.txt?_xsrf=13sja -> https://demo.com/passer/1DF9E2B3S6E4X8A2S0T8L5.txt 
grep "download" simple.log | sed -r "s/.*\/([^/]+)\?_xsrf=.*com\/([^/]+)\/.*/\1,\2/"
```

替换文件中某个字符串
```bash
sed -i 's/ARCHIVE.*=.*/test.zip/g' simple.log
```

在文件中的某一行插入
```bash
sed -i '2i/sed demo' simple.log
```


## cut
以t字符切割，取切割后第四列
```bash
cat simple.log | cut -dt -f4
```

## find
查找指定目录的文件
```bash
find . -name "*.md"
```

[查找指定目录指定大小的目录](https://blog.csdn.net/weixin_33713707/article/details/85933224)

```bash
# 一块，512字节
find . -type d -size 1
```

```bash
# 64字节
find . -type d -size 64c
```

```bash
# 64KB    MB, GB e.g. M G
find . -type d -size 64k
```

```bash
find . -type f -size 0 -exec ls -l {} \;
```

## df

查看磁盘占用情况
```bash
df -h
```

## du

查看当前目录各个子目录占用磁盘情况

```bash
du -sh *
```


