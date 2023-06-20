## 参考博客
https://segmentfault.com/a/1190000024540660

## Tomcat线程模型
有BIO/NIO/AIO几种模式

NIO在Tomcat8之后是默认方式，线程模型和主从Reactor模式类似

### BIO模型

在BIO模型中，主要参与的角色有：Acceptor和Handler工作线程池。对应于前文中Api的请求过程，它们的分工如下：

- Acceptor：Accepter线程专门负责建立网络连接（accept）。新连接创建后，交给Handler工作线程池处理请求。
- Handlers：针对每个请求的连接，Handler工作线程池都会分配一个线程，执行后面的所有步骤（read、decode、process、encode、send）。

前文的知识点有铺垫，read和send是面向网络I/O的，在等待读写就绪过程中，其实是CPU阻塞的。因此Handler工作线程池中的每个线程，都会因为I/O阻塞而“空等待”，造成浪费。

### NIO模型
tomcat的NIO模型，相比较于BIO模型，多了个Poller角色：Acceptor、Poller和Handler工作线程池。这三个角色是不是很熟悉，如果将Poller换成Reactor，是不是就是Reactor模型。没错，tomcat的nio模型，的确就是基于主从Reactor模型，只不过将Reactor换了个名字而已。

- Acceptor：Accepter线程专门负责建立网络连接（accept）。新连接创建后，不是直接使用Worker线程处理请求，而是先将请求发送给Poller缓冲队列。
- Poller：在Poller中，维护了一个Selector对象，当Poller从缓冲队列中取出连接后，注册到该Selector中，阻塞等待读写就绪（read等待就绪、send等待就绪）。
-  Handlers：遍历Selector，找出其中就绪的IO操作，并交给Worker线程处理（read内存读、decode、process、encode、send内存写）。