##### 新生代GC（MinorGC/YoungGC）：
指发生在新生代的垃圾收集动作，因为 Java 对象大多都具备朝生夕灭的特性，所以 MinorGC 非常频繁，一般回收速度也比较快。

##### 老年代GC（MajorGC/FullGC）：
指发生在老年代的 GC，出现了 MajorGC，经常会伴随至少一次的 MinorGC（但非绝对的，在 Parallel Scavenge 收集器的收集策略里就有直接进行 MajorGC 的策略选择过程）。MajorGC 的速度一般会比 MinorGC 慢 10 倍以上。

##### 🦅 什么情况下会出现 Young GC？
对象优先在新生代 Eden 区中分配，如果 Eden 区没有足够的空间时，就会触发一次 Young GC 。

##### 🦅 什么情况下回出现 Full GC？

Full GC 的触发条件有多个，FULL GC 的时候会 STOP THE WORD 。

1、在执行 Young GC 之前，JVM 会进行空间分配担保——如果老年代的连续空间小于新生代对象的总大小（或历次晋升的平均大小），则触发一次 Full GC 。
2、大对象直接进入老年代，从年轻代晋升上来的老对象，尝试在老年代分配内存时，但是老年代内存空间不够。
3、显式调用 System#gc() 方法时。