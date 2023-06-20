> 参考 https://zhuanlan.zhihu.com/p/308054212
> 
> 参考 https://segmentfault.com/a/1190000041879813?utm_source=sf-similar-article
> 
> [一篇很不错的文章，聊聊Linux I/O](https://www.0xffffff.org/2017/05/01/41-linux-io/?spm=a2c6h.12873639.article-detail.6.58554115pHDyjY)

> https://blog.51cto.com/u_11451275/4780026
## 传统IO

补充文件寻址的过程？

将硬盘里的文件发送的流程：
1. 应用程序调用read
2. 触发System call，进程从用户态切换到内核态
3. 首先通过DMA将硬盘中的文件拷贝到内核缓冲区
2. 由内核缓冲区通过cpu拷贝到用户态的缓冲区
3. 这时应用程序调用，write将用户态缓冲区的数据通过cpu拷贝到Socket缓冲区
4. 最后再Socket缓冲区通过DMA拷贝到网卡

![](imgs/Linux-IO-read&write.png)

## mmap
![](imgs/Linux-mmap.png)

## senfile
![](imgs/sendfile.png)
