## volatile的内存语义实现

我们都知道，为了性能优化，JMM在不改变正确语义的前提下，会允许编译器和处理器对指令序列进行重排序，那如果想阻止重排序要怎么办了？答案是可以添加内存屏障。

> **内存屏障**

JMM内存屏障分为四类见下图，



![内存屏障分类表](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111061724203.webp)



java编译器会在生成指令系列时在适当的位置会插入内存屏障指令来禁止特定类型的处理器重排序。为了实现volatile的内存语义，JMM会限制特定类型的编译器和处理器重排序，JMM会针对编译器制定volatile重排序规则表：



![volatile重排序规则表](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111061724281.webp)



"NO"表示禁止重排序。为了实现volatile内存语义时，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的**处理器重排序**。对于编译器来说，发现一个最优布置来最小化插入屏障的总数几乎是不可能的，为此，JMM采取了保守策略：

1. 在每个volatile写操作的**前面**插入一个StoreStore屏障；
2. 在每个volatile写操作的**后面**插入一个StoreLoad屏障；
3. 在每个volatile读操作的**后面**插入一个LoadLoad屏障；
4. 在每个volatile读操作的**后面**插入一个LoadStore屏障。

需要注意的是：volatile写是在前面和后面**分别插入内存屏障**，而volatile读操作是在**后面插入两个内存屏障**

**StoreStore屏障**：禁止上面的普通写和下面的volatile写重排序；

**StoreLoad屏障**：防止上面的volatile写与下面可能有的volatile读/写重排序

**LoadLoad屏障**：禁止下面所有的普通读操作和上面的volatile读重排序

**LoadStore屏障**：禁止下面所有的普通写操作和上面的volatile读重排序

下面以两个示意图进行理解，图片摘自相当好的一本书《java并发编程的艺术》。



![volatile写插入内存屏障示意图](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111061724261.webp)





![volatile读插入内存屏障示意图](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111061724239.webp)



# 5. 一个示例

我们现在已经理解volatile的精华了，文章开头的那个问题我想现在我们都能给出答案了。更正后的代码为：

```
public class VolatileDemo {
    private static volatile boolean isOver = false;

    public static void main(String[] args) {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (!isOver) ;
            }
        });
        thread.start();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        isOver = true;
    }
}
复制代码
```

注意不同点，现在已经**将isOver设置成了volatile变量**，这样在main线程中将isOver改为了true后，thread的工作内存该变量值就会失效，从而需要再次从主内存中读取该值，现在能够读出isOver最新值为true从而能够结束在thread里的死循环，从而能够顺利停止掉thread线程。现在问题也解决了，知识也学到了：）。（如果觉得还不错，请点赞，是对我的一个鼓励。）



作者：你听___
链接：https://juejin.cn/post/6844903601064640525
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。