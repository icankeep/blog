## 参考博客
https://www.cnblogs.com/ajianbeyourself/p/15650518.html

## Redis的BLPOP阻塞原理
当客户端向Redis通过BLPOP元素时，

如果key对应的列表中已存在元素，则直接pop出元素，返回给客户端

当不存在元素时，这时客户端会阻塞，直到该列表有新的元素加入

那么Redis服务端是怎么处理这个阻塞请求的呢？

1. 首先客户端发起请求
2. 服务端发现对应key没有元素，就会向blocking_keys中添加当前客户端client，此时客户端会一直阻塞
3. 如果客户端发来一个rpush key value命令，先从blocking_keys中查找是否存在对应的key，
如果存在就往ready_keys这个链表中添加该key；同时将value插入到对应的list中，并响应客户端。
