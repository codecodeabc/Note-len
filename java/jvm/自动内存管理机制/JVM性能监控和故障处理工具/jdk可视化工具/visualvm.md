#### visualvm - 多合一故障处理工具

##### 1.功能

![image-20210811155227684](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210811155227.png)

![image-20210811155328635](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210811155328.png)

![image-20210811155238783](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210811155238.png)



##### 2.启用工具

在JDK/bin 目录下 双击 jvisualvm.exe 程序

###### （1）添加主机

![image-20210811161643255](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210811161643.png)



（2）添加jmx连接

![image-20210811161701941](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210811161701.png)

注：开启jmx允许远程调用，进程启动时配置以下参数，resin服务器则在<jvm-arg>里配置下列参数

```java
-Djava.rmi.server.hostname=10.160.13.111  #远程服务器ip，即本机ip
-Dcom.sun.management.jmxremote #允许JMX远程调用
-Dcom.sun.management.jmxremote.port=3214  #自定义jmx 端口号
-Dcom.sun.management.jmxremote.ssl=false  # 是否需要ssl 安全连接方式
-Dcom.sun.management.jmxremote.authenticate=false #是否需要秘钥
```



###### （3）监控查看

![image-20210811161747044](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210811161747.png)



##### 3.安装插件

![image-20210811162050027](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210811162050.png)

![image-20210811162111952](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210811162112.png)



https://mp.weixin.qq.com/s/SsJeaWz4EvZvQkYjc6J6jg

插件离线安装

https://blog.csdn.net/q258523454/article/details/103475425

插件地址：https://visualvm.github.io/uc/8u131/updates.html



配置jstatd 查看远程gc  ,jstatd 端口需要映射出来

https://blog.csdn.net/zhang355/article/details/103499502

配置远程jstatd

http://tonfay.cn/articles/2019/08/26/1566791618191.html