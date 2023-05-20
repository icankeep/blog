1. 创建从1到1000的目录
```bash
for i in {1..1000};
do
  echo 
```
1. 遍历某个目录下所有文件，修改权限
```bash
for i in `ls`;
do 
  echo "chmod 777 $i";
  chmod 777 $i;
done
```

