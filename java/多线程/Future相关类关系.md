https://juejin.cn/post/6844904033673560077



# 彻底搞懂Future、Callable、FutureTask、Runnable

# 引言

- 看到标题一下子好几个名称，很可能你都知道、都听过、看过，但是你真是理清楚了它们的关系了吗？
- 在这个知识泛滥、技术焦虑的时刻，人人嘴里喷着高并发、大数据、分布式，很多估计对这个一头雾水，无论在开发还是面试过程中，一知半解还不如不知。

## Runnable/Thread

- 通常情况下的耗时操作会交给多线程来处理，Java中开启一个新线程很容易，继承自Thread或实现Runnable接口。下面是常规操作。



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/29/16f4faf1c4b7f008~tplv-t2oaga2asx-watermark.awebp)



- 开启多线程很多时候是为了利用CPU的多核能力。new Thread（）或实现Runnable很容易实现，那为何还需要Future、Callable呢？是JDK开发者嫌头发多了吗？
- 通常情况下我们只管一顿操作，开启线程扔出去，至于返回值我们开发中好像从来没管过。其实无论是new Thread（）还是实现Runnable在执行完了都是无法获取执行结果的，不是我们不想管而是管不了。至于线程执行成功还是失败，很多时候都是听天由命，因为大多情况下我们默认这个执行操作肯定会成功。出了问题也只能追日志了。
- 通过共享变量或者线程通信的方式倒是可以间接获取执行结果，但是相信我以你的水平，怕是要996解bug。

# Future机制

## Callable

- 那既然到这里了，相信你也能猜到Java 1.5中引入的Callable就是解决这个返回值的问题，



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/29/16f4fb4e6a614c15~tplv-t2oaga2asx-watermark.awebp)



- Callable是一个接口，一个函数式接口，也是个泛型接口。call（）有返回值，且返回值类型与泛型参数类型相同，且可以抛出异常。Callable可以看作是Runnable接口的补充。

## Future

- 也许Future的知名度更高，通常所说的Future机制而不是Callable机制。既然Callable可以解决无返回值的问题，那么Future又是什么呢？
- Future是为了配合Callable/Runnable而产生的，既然有返回值，那么返回什么？什么时候返回？这些问题其实都可以算在Future机制里。
- 简单来讲我认为Future是一个句柄，即Callable任务返回给调用方这么一个句柄，通过这个句柄我们可以跟这个异步任务联系起来，我们可以通过future来对任务查询、取消、执行结果的获取，是调用方与异步执行方之间沟通的桥梁。



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/29/16f4fb7dc0b3cde5~tplv-t2oaga2asx-watermark.awebp)



## FutureTask

- 到现在基本清晰了，Future机制就是为了解决多线程返回值的问题。但是Callable、Future、RunnableFuture都是接口，接口不干活啊。没关系，FutureTask来了。



![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111161535914.webp)



- FutureTask实现了RunnableFuture接口，同时具有Runnable、Future的能力，即既可以作为Future得到Callable的返回值，又可以作为一个Runnable。
- FutureTask是一个泛型类，下面是一个demo。



![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111161535834.webp)



# 小结

- Future机制并不是对Runnable的革名，只是对Runnable的扩展。Callable、Future、FutureTask的配合，解决Runnable无返回值的问题。
- Callable、Future、RunnableFuture都是接口，是FutureTask在背后默默的干活。
- 名词虽然多了点，但是其实并没有那么复杂，看完本文完全可以搞清楚他们的关系。在多线程开发中能够清晰地知道采取最好的方式。