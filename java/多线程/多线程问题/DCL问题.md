

> 说起单例模式大家基本上已经很熟了。最为经典的例子double-check-locking （DCL）想想大家都写了很多次了。然而随着越来越透传的对JMM的理解，我们渐渐发现传统的DCL在理论上或许存在问题。

> 引用：
>
> 1. [Initialization-on-demand_holder_idiom](https://links.jianshu.com/go?to=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FInitialization-on-demand_holder_idiom)
> 2. [单例模式、双检测锁定DCL、volatile（转）](https://links.jianshu.com/go?to=https%3A%2F%2Fcrud0906.iteye.com%2Fblog%2F576321)



## 1.传统的例子

非常经典的例子，基本上对java有了解的同学都可以写出来，我们的例子，可能存在一个BUG，这个BUG的原因是，JMM出于对效率的考虑，是在happens-before原则内（out-of-order）乱序执行。



```java
public class LazySingleton {
    private int id;
    private static LazySingleton instance;
    private LazySingleton() {
        this.id= new Random().nextInt(200)+1;                 // (1)
    }
    public static LazySingleton getInstance() {
        if (instance == null) {                               // (2)
            synchronized(LazySingleton.class) {               // (3)
                if (instance == null) {                       // (4)
                    instance = new LazySingleton();           // (5)
                }
            }
        }
        return instance;                                      // (6)
    }
    public int getId() {
        return id;                                            // (7)
    }
}
```

## 2. 简单的原理性介绍。

我们初始化一个类，会产生多条汇编指令，然而总结下来，是执行下面三个事情：

1.给LazySingleton 的实例分配内存。
 2.初始化LazySingleton 的构造器
 3.将instance对象指向分配的内存空间（注意到这步instance就非null了）

Java编译器允许处理器乱序执行（out-of-order），我们有可能是1->2->3也有可能是1->3->2。即我们有可能在先返回instance实例，然后执行构造方法。

即：double-check-locking可能存在线程拿到一个没有执行构造方法的对象。

## 3.一个简单可能出错的执行顺序。

线程A、B执行getInstance().getId()

在某一时刻，线程A执行到(5)，并且初始化顺序为：1->3->2，当执行完将instance对象指向分配空间时。此时线程B执行(1)，发现**instance!=null**，直接使用instance.getId(),继续执行，**此时构造函数还没初始化id的值**，最后调用getId()返回0。此时切换到线程B对构造方法初始化。

## 4. 解决方案

### 方案一：

利用类第一次使用才加载，加载时同步的特性。
 优点是：官方推荐，可以可以保证实现懒汉模式。代码少。
 缺点是：第一次加载比较慢，而且多了一个类多了一个文件，总觉得不爽。



```java
public class SingletonKerriganF {     
      
    private static class SingletonHolder {     
        static final SingletonKerriganF INSTANCE = new SingletonKerriganF();     
    }     
      
    public static SingletonKerriganF getInstance() {     
        return SingletonHolder.INSTANCE;     
    }     
}    
```

### 方案二：利用volatile关键字

volatile禁止了指令重排序，所以确保了**初始化顺序一定是1->2->3**，所以也就不存在拿到未初始化的对象引用的情况。
 优点：保持了DCL，比较简单
 确定：volatile这个关键字多少会带来一些性能影响吧。



```java
public class Singleton(){  
    private volatile static Singleton singleton;  
    private Sington(){};  
    public static Singleton getInstance(){  
        if(singleton == null){  
            synchronized (Singleton.class){  
                if(singleton == null){  
                     singleton = new Singleton();    
                }  
            }
        }           
        return singleton;  
    }  
}  
```

## 方案三：初始化完后赋值。

通过一个temp，来确定初始化结束后其他线程才能获得引用。
 同时注意，JIT可能对这一部分优化，我们必须阻止JTL这部分的"优化"。

缺点是有点难理解，优点是：可以不用volatile关键字，又可以用DLC，岂不妙哉。



```java
public class Singleton {    
    
    private static Singleton singleton; // 这类没有volatile关键字    
    
    private Singleton() {    
    }    
    
    public static Singleton getInstance() {    
        // 双重检查加锁    
        if (singleton == null) {    
            synchronized (Singleton.class) {    
                // 延迟实例化,需要时才创建    
                if (singleton == null) {    
                        
                    Singleton temp = null;  
                    try {  
                        temp = new Singleton();    
                    } catch (Exception e) {  
                    }  
                    if (temp != null)    //为什么要做这个看似无用的操作，因为这一步是为了让虚拟机执行到这一步的时会才对singleton赋值，虚拟机执行到这里的时候，必然已经完成类实例的初始化。所以这种写法的DCL是安全的。由于try的存在，虚拟机无法优化temp是否为null  
                        singleton = temp; 
                }    
            }    
        }    
        return singleton;    
    }  
}  
```

https://www.jianshu.com/p/ca19c22e02f4