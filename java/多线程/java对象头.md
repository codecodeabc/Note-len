在同步的时候是获取对象的monitor,即获取到对象的锁。那么对象的锁怎么理解？无非就是类似对对象的一个标志，那么这个标志就是存放在Java对象的对象头。Java对象头里的Mark Word里默认的存放的对象的Hashcode,分代年龄和锁标记位。32为JVM Mark Word默认存储结构为（注:java对象头以及下面的锁状态变化摘自《java并发编程的艺术》一书，该书我认为写的足够好，就没在自己组织语言班门弄斧了）：



![Mark Word存储结构](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111061711021.webp)



如图在Mark Word会默认存放hasdcode，年龄值以及锁标志位等信息。

Java SE 1.6中，锁一共有4种状态，级别从低到高依次是：**无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态**，这几个状态会随着竞争情况逐渐升级。**锁可以升级但不能降级**，意味着偏向锁升级成轻量级锁后不能降级成偏向锁。这种锁升级却不能降级的策略，目的是为了提高获得锁和释放锁的效率。对象的MarkWord变化为下图：



![Mark Word状态变化](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111061711084.webp)


作者：你听___
链接：https://juejin.cn/post/6844903600334831629
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。