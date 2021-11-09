https://juejin.cn/post/6994784895155322888

# 【Docker 系列】docker 学习八，Docker 网络

## 开始理解 docker

**一开始，咱们思考一下**，宿主机怎么和容器通信呢？

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809466.webp)

说容器之间是相互隔离的，那么他们是否可以通信？又是如何通信的呢？

## 开始探索

**我们先来看看咱环境中的镜像都有些啥，有 `xmtubuntu`**

```shell
# docker images
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
xmtubuntu             latest    c3e95388a66b   38 seconds ago   114MB
复制代码
```

**再来看看宿主机的网卡信息**

`ip addr `来查看咱们宿主机的网卡信息

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809879.webp)

我们发现有一个` docker0`，是因为我们的宿主机上面安装了docker 的服务，docker 会给我生成一个虚拟网卡，图中的这个 `docker0`就是虚拟网卡信息

**创建并启动一个docker 命名为 ubuntu1**

```
docker run -it --name ubuntu1 -P xmtubuntu
```

**查看一下宿主机网卡信息**

查看宿主机的网卡信息

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8cdc0df6fa3c45a39fb772c301cc7890~tplv-k3u1fbpfcp-watermark.awebp)

再查看 `ubuntu1` 的网卡信息，docker 也会默认给我们的容器分配`ip` 地址

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809847.webp)

可以发现宿主机的网卡信息 `docker0` 下面多了`117: veth838e165@if116:`，`ubuntu1`的网卡信息上也正好有`116: eth0@if117`

我们发现这些`veth`的编号是成对出现的，咱们的宿主机就可以和 `ubuntu1 `进行通信了

**使用宿主机（docker0）和`ubuntu1`互相 ping**

`docker0 `ping`ubuntu1` ok

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809898.webp)

`ubuntu1`ping`docker0 `，同样的 ok

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809862.webp)

咱们可以尝试再创建并启动一个docker 命名为 ubuntu2，方法和上述完全一致

```shell
# docker run -it -P --name ubuntu2 xmtubuntu
复制代码
```

进入容器，使用`ip a`查看到`ubuntu2`的网卡信息

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d561448d61f4996bd32bc888d7ea30a~tplv-k3u1fbpfcp-watermark.awebp)

宿主机上面查看网信息

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809752.webp) 宿主机上面又多了一个 veth ， `119: veth0b29558@if118`

`ubuntu2`上的网卡信息是`118: eth0@if119`，他们同样是成对出现的，小伙伴看到这里应该明白了吧

`ubuntu1` ping `ubuntu2` 呢？

ubuntu1 对应 172.18.0.2

ubuntu2 对应 172.18.0.3

```shell
# docker exec -it ubuntu1 ping 172.18.0.3
PING 172.18.0.3 (172.18.0.3) 56(84) bytes of data.
64 bytes from 172.18.0.3: icmp_seq=1 ttl=64 time=0.071 ms
64 bytes from 172.18.0.3: icmp_seq=2 ttl=64 time=0.070 ms
64 bytes from 172.18.0.3: icmp_seq=3 ttl=64 time=0.077 ms
复制代码
```

仍然是可以通信，非常 nice

## **原理是什么？**

上述的探索，我们发现宿主机创建的容器，都可以直接`ping`通宿主机，那么他们的原理是啥呢？

细心的 xdm 应该可以看出来，上述的例子中，`veth`是成对出现的，上述宿主机和容器能够进行网络通信，得益于这个技术`veth-pair`

**veth-pair**

**veth-pair **是一对虚拟设备接口，他们都是成对出现的，一段连着协议，一段彼此相连

正是因为这个特性，`veth-pair`在此处就是充当了一个桥梁，连接各种虚拟设备

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809917.webp)

通过上图我们可以得出如下结论：

- `ubuntu1` 和 `ubuntu2`他们是公用一个路由器，也就是 docker0，ubuntu1 能**ping**通`ubuntu2`是因为 docker0 帮助其转发的
- 所有的容器在不指定路由的情况下，都是以 docker0 作为路由，docker 也会给我们的容器分配一个可用的 ip
- docker0 是在宿主机上面安装 docker 服务就会存在的

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809474.webp)

那么通过上图我们就知道，容器和宿主机之前是通过桥接的方式来打通网络的。

Dcoker 中所有的网络接口都是虚拟的，因为虚拟的转发效率高呀，当我们删除某一个容器的时候，这个容器对应的网卡信息，也会被随之删除掉

**那么我们可以思考一下**，如果都是通过找`ip`地址来通信，如果 `ip`变化了，那么我们岂不是找不到正确的容器了吗？我们是否可以通过服务名来访问容器呢？

## –link

当然是可以的，当我们在创建和启动容器的时候加上`–link`就可以达到这个效果

我们再创建一个容器 `ubuntu3`，让他 link 到 `ubuntu2`

```
# docker run -it --name ubuntu3 -P --link ubuntu2 xmtubuntu
# docker exec -it ubuntu3 ping ubuntu2
PING ubuntu2 (172.18.0.3) 56(84) bytes of data.
64 bytes from ubuntu2 (172.18.0.3): icmp_seq=1 ttl=64 time=0.093 ms
64 bytes from ubuntu2 (172.18.0.3): icmp_seq=2 ttl=64 time=0.085 ms
64 bytes from ubuntu2 (172.18.0.3): icmp_seq=3 ttl=64 time=0.092 ms
64 bytes from ubuntu2 (172.18.0.3): icmp_seq=4 ttl=64 time=0.073 ms
复制代码
```

很明显，我们可以到看到 `ubuntu3 `可以通过服务名`ubuntu2 `直接和`ubuntu2`通信，但是反过来是否可以呢？

```shell
# docker exec -it ubuntu2 ping ubuntu3
ping: ubuntu3: Name or service not known
复制代码
```

不行？这是为什么呢？

我们来查看一下 `ubuntu3 `的本地 `/etc/hosts `文件就清楚了

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/018775e9b7584bf285d2c004244ad37a~tplv-k3u1fbpfcp-watermark.awebp)

看到这里，这就清楚了 link 的原理了吧，就是在自己的  `/etc/hosts `文件中，加入一个`host`而已，这个知识点我们可以都知悉一下，但是这个 link 还是好搓，不好，他需要在创建和启动容器的时候使用，用起来不方便

**那么我们有没有更好的办法的呢？**

## 自定义网络

可以使用 `docker network ls`查看宿主机 docker 的网络情况

```shell
:~# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
8317183dfc58   bridge    bridge    local
997107487c6b   host      host      local
ab130876cbe6   none      null      local
复制代码
```

**网络模式**

- bridge

桥接，docker0 默认使用 bridge 这个名字

- host

和宿主机共享网络

- none

不配置网络

- container

容器网络连通，这个模式用的非常少，因为局限性很大

现在咱们可以自定义个网络，来连通两个容器

### 自定义网络

自定义一个 **mynet** 网络

```shell
# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
9a597fc31f1964d434181907e21ff7010738f3f7dc35ba86bf7434f05a6afc4a
复制代码
```

- docker network create

创建一个网络

- --driver

指定驱动是 bridge

- --subnet

指定子网

- --gateway

指定网关

此处我们设置子网是` --subnet 192.168.0.0/16`，网关是`192.168.0.1`，那么我们剩下可以使用的`ip`就是 `192.168.0.2 – 192.168.255.254` ， `192.168.255.255`是广播地址

### 清空已有的容器

清空所有测试的容器，减去干扰

### 创建并启动2个容器，分别是ubuntu1 和 ubuntu2

```shell
# docker run -it -P --name ubuntu1 --net mynet xmtubuntu
# docker run -it -P --name ubuntu2 --net mynet xmtubuntu
复制代码
```

此时我们可以查看一下宿主机的网卡信息，并验证两个容器直接通过容器名字是否可以通信

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809924.webp)

**我们思考一下自定义网络的好处**

咱们自定义 docker 网络，已经帮我们维护好了对应关系，这样做的好处是容器之间可以做到网络隔离，

例如

一堆 redis 的容器，使用 192.168.0.0/16 网段，网关是 192.168.0.1

一堆 mongodb的容器，使用 192.167.0.0/16 网段，网关是 192.167.0.1

这样就可以做到子网很好的隔离开来，不同的集群使用不同的子网，互不影响

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809882.webp)

**那么子网间是否可以打通呢？**

## 网络连通

两个不同子网内的容器如何连通了呢？

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809077.webp)

我们绝对不可能让不同子网的容器不通过路由的转发而直接通信，这是不可能的，子网之间是相互隔离的

但是我们有办法让 ubuntu3 这个容器通过和 mynet 打通，进而转发到 ubuntu1 或者 ubuntu2 中就可以了

### 打通子网

我们查看 docker network 的帮助手册

```
 docker network -h
```

![image-20210807203841520](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26335f26eb98464ba7e228e0e9ecf323~tplv-k3u1fbpfcp-watermark.awebp)

可以使用 `docker network connect `命令实现，在查看一下帮助文档

```shell
# docker network connect -h
Flag shorthand -h has been deprecated, please use --help

Usage:  docker network connect [OPTIONS] NETWORK CONTAINER

Connect a container to a network
复制代码
```

### 开始打通

```
docker network connect mynet ubuntu3
```

这个时候我们可以查看一下 mynet 网络的详情

```shell
# docker network inspect mynet
复制代码
```

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809814.webp)

可以看到在 mynet 网络上，又增加了一个容器，ip 是 192.168.0.4

没错，docker 处理这种网络打通的事情就是这么简单粗暴，直接在 ubuntu3 容器上增加一个虚拟网卡，让 ubuntu3 能够和 mynet 网络打通

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/202111091809643.webp)

宿主机当然也相应的多了一个`veth`

![]([gitee.com/common_dev/…](https://link.juejin.cn?target=https%3A%2F%2Fgitee.com%2Fcommon_dev%2Fmypic%2Fraw%2F)	master/image-20210807204514806.png)

现在，要跨网络操作别人的容器，我们就可以使用 `docker network connect`的方式将网络打通，开始干活了

大家对网络还感兴趣吗，哈哈，关于 docker 的前几期文章链接如下，可以逐步学习，慢慢深入，多多回顾

[【Docker 系列】docker 学习 五，我们来看看容器数据卷到底是个啥](https://juejin.cn/post/6993495393338146852)

[【Docker 系列】docker 学习 四，一起学习镜像相关原理](https://juejin.cn/post/6993095288839733284)

[【Docker 系列】docker 学习 三，docker 初步实战和 docker 可视化管理工具试炼](https://juejin.cn/post/6992728823481499678)

[【Docker 系列】docker 学习 二，docker 常用命令，镜像命令，容器命令，其他命令](https://juejin.cn/post/6992353652753039374)

[【Docker 系列】docker 学习 一，Docker的安装使用及Docker的基本工作原理 | 8月更文挑战](https://juejin.cn/post/6991982486725066759)

参考资料：

[docker docs](https://link.juejin.cn?target=https%3A%2F%2Fdocs.docker.com%2Freference%2F)


作者：小魔童哪吒
链接：https://juejin.cn/post/6994784895155322888
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。