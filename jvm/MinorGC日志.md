#### JVM参数
-verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:+UseSerialGC

#### MinorGC 日志 (清除 Eden、Survivor 区，就是 Minor GC )


```text
[GC (Allocation Failure) [DefNew: 7129K->922K(9216K), 0.0160834 secs] 7129K->5018K(19456K), 0.0223899 secs] [Times: user=0.00 sys=0.00, real=0.02 secs] 
Heap
 def new generation   total 9216K, used 7304K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  77% used [0x00000000fec00000, 0x00000000ff23ba10, 0x00000000ff400000)
  from space 1024K,  90% used [0x00000000ff500000, 0x00000000ff5e69a0, 0x00000000ff600000)
  to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
 tenured generation   total 10240K, used 4096K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  40% used [0x00000000ff600000, 0x00000000ffa00020, 0x00000000ffa00200, 0x0000000100000000)
 Metaspace       used 3274K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 356K, capacity 388K, committed 512K, reserved 1048576K
```

##### GC：

表明进行了一次垃圾回收，前面没有Full修饰，表明这是一次Minor GC ,注意它不表示只GC新生代，并且现有的不管是新生代还是老年代都会STW。

##### Allocation Failure

表明本次引起GC的原因是因为在年轻代中没有足够的空间能够存储新的数据了。

##### DefNew

表示GC发生的区域为Serial收集器的新生代中，DefNew是"Default New Generation"的缩写。Serial收集器的老年代和永久代分别表示为"Tenured"、"Perm"

##### 7129K->922K(9216K)

三个参数分别为：(新生代)GC前该内存区域(这里是年轻代)使用容量，GC后该内存区域使用容量，该内存区域总容量。

##### 0.0160834 secs

(新生代)GC消耗的时间

##### 7129K->5018K(19456K)

三个参数分别为：(堆)堆区垃圾回收前的大小，堆区垃圾回收后的大小，堆区总大小。

##### 0.0223899 secs

(堆)GC消耗的时间

##### [Times: user=0.00 sys=0.00, real=0.02 secs] 

分别表示用户态耗时，内核态耗时和总耗时
