#### JVM参数说明

|  Option   |  Description  |
|  ----  | ----  |
| -verbose:gc	|显示GC的操作内容|
|  -Xms20M -Xmx20M	|设置堆大小为20M|
|  -Xmn10M	|设置新生代的内存空间大小为10M|
| -XX:+PrintGCDetails|打印GC中的变化|
|  -XX:SurvivorRatio=8|新生代中Eden区域与Survivor区域的大小比值|
| -XX:+UseSerialGC|	在新生代和老年代中使用串行收集器，由于-verbose:gc参数对Parallel Scavenge收集器不起作用， 无法显示显示GC的操作内容，因此采用串行收集器 |
| -XX:PretenureSizeThreshold=3145728|所占用内存大于该值的对象直接分配到老年代，3145728为3MB|
|-XX:MaxTenuringThreshold=1|对象晋升为老年代的年龄阀值为1|



