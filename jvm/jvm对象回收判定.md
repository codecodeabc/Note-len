#### 一.可回收对象的两种判断算法

##### 1.引用计数法（Java虚拟机没有采用引用计数法）

缺点：循环依赖对象无法被判定可回收

Object 1和Object 2其实都可以被回收，但是它们之间还有相互引用，所以它们各自的计数器为1，则还是不会被回收。

![image-20210810114541309](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210810114857.png?token=AIDRWUFF4COW5T7RPVVLZM3BCH3WU)

#### 2.可达性分析

通过一系列的“GC Roots”，也就是根对象作为起始节点集合，从根节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为引用链，如果某个对象到GC Roots间没有任何引用链相连

![image-20210810115238033](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210810115238.png?token=AIDRWUF6IOTCRWHQW42V2N3BCH4EK)



**问：哪些对象可以作为根对象GC ROOT**

 **a.** java虚拟机栈中的引用的对象。 

![image-20210810115619693](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210810115619.png?token=AIDRWUGENALH35RGUAUQCR3BCH4SG)

  **b**.方法区中的类静态属性引用的对象。 （一般指被static修饰的对象，加载类的时候就加载到内存中。）

![image-20210810115717192](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210810115717.png?token=AIDRWUE5FD6G7EPHPU43OK3BCH4VY)

  **c**.方法区中的常量引用的对象。 

![image-20210810115744221](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210810115744.png?token=AIDRWUDFRB7MYGXVSDRFII3BCH4X6)

  **d**.本地方法栈中的JNI（native方法）引用的对象



#### 二.判定对象失效后如何抢救

##### 1.重写finalize方法并建立引用

![image-20210810115941264](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210810115941.png?token=AIDRWUEAHV6W6H2K2DVTARTBCH464)



#### 三.方法区里的数据如何回收

![image-20210810120236889](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210810120236.png?token=AIDRWUBPAFTUCEQA6ZAFIVDBCH5JY)







