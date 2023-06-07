1. https://juejin.cn/post/6844903622380093447
2. https://juejin.cn/post/6844903622384287751
3. https://juejin.cn/post/6844903622384287757
4. https://juejin.cn/post/6968641908314751012

## Server端
- TSimpleServer：采用最简单的阻塞IO，只有一个工作线程，循环监听新请求的到来并完成对请求的处理，一次只能接收和处理一个socket连接。
- TNonblockingServer： 单线程工作，采用NIO的方式，所有的socket都被注册到selector中，在一个线程中通过seletor循环监控所有的socket。 优点：IO多路复用，非阻塞IO 缺点：单线程，请求任务一个一个执行
- THsHaServer(半同步半异步)：TNonblockingServer类的子类，NIO方式， 主线程负责监听，线程池负责处理任务 缺点：新连接请求不能被及时接受
- TThreadPoolServer：阻塞socket方式工作，主线程负责阻塞式监听，业务处理交由一个线程池来处理 缺点： 受限于线程池中线程数量大小
- TThreadedSelectorServer：目前Thrift提供的最高级的模式，专门的线程AcceptThread用于处理新连接请求，及时响应大量并发连接请求，负载均衡器分散到多个SelectorThread线程中来完成

## 序列化层
传输协议上总体上划分为文本(text)和二进制(binary)传输协议

1、TBinaryProtocol – 二进制编码格式进行数据传输。

2、TCompactProtocol – 这种协议非常有效的，使用Variable-Length Quantity (VLQ) 编码对数据进行压缩。高效和压缩的二进制格式

3、TJSONProtocol – 使用JSON的数据编码协议进行数据传输。

4、TSimpleJSONProtocol – 这种节约只提供JSON只写的协议，适用于通过脚本语言解析

5、TDebugProtocol – 在开发的过程中帮助开发人员调试用的，以文本的形式展现方便阅读。


## 传输层
1、TSocket- 使用堵塞式I/O进行传输，也是最常见的模式。

2、TFramedTransport- 使用非阻塞方式，按块的大小，进行传输，类似于Java中的NIO。

3、TFileTransport- 顾名思义按照文件的方式进程传输，虽然这种方式不提供Java的实现，但是实现起来非常简单。

4、TMemoryTransport- 使用内存I/O，就好比Java中的ByteArrayOutputStream实现。

5、TZlibTransport- 使用执行zlib压缩，不提供Java的实现。
