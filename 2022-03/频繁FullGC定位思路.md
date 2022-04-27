怎么才算频繁？
一般来说FullGC几个小时或者几天一次

1. 首先查看FullGC前后老年代的内存占用情况，若GC前后，内存占用没有明显变化，则可能发生了
内存泄露，需要dump内存信息，查看占用内存的对象
2. 若发生FullGC时老年代并没有占满，有可能是metaspace空间满了导致的FullGC，需要检查代码中动态代理生成的代理类过多问题，或者调大metaspace再查看，[例子](https://isudox.com/2021/03/10/metaspace-threshold-fullgc-analyze/)
3. ygc比较频繁，一秒一次或几秒一次，导致一些持有时间略长的对象，通过年龄晋升到老年代，稍微加大升到老年代的年龄，或加大年轻代空间
4. 通过dump堆内存，看到堆里面对象基本都是合理的分布情况，那可能是老年代确实设的过小了，可以加大老年代空间

- [zk引发的](https://blog.csdn.net/qiubboy/article/details/117965200)
- [思路](https://www.jianshu.com/p/79fc0d77002e)
