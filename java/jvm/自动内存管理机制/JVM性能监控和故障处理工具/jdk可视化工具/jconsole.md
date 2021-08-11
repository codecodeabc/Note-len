#### jconsole - java监视与管理控制台

##### 1.启动jconsole进程

###### （1）本地连接

在JDK/bin 目录下 双击 jconsole.exe ，启动后会自动搜索本机运行的所有虚拟机进程，点击连接即可开始监控

![image-20210811152051398](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210811152051.png)

（2）远程连接

开启jmx允许远程调用，进程启动时配置以下参数，resin服务器则在<jvm-arg>里配置下列参数

```java
-Djava.rmi.server.hostname=10.160.13.111  #远程服务器ip，即本机ip
-Dcom.sun.management.jmxremote #允许JMX远程调用
-Dcom.sun.management.jmxremote.port=3214  #自定义jmx 端口号
-Dcom.sun.management.jmxremote.ssl=false  # 是否需要ssl 安全连接方式
-Dcom.sun.management.jmxremote.authenticate=false #是否需要秘钥
```

以下和启用ssl有关

```
# 以下和启用SSL有关
-Dcom.sun.management.jmxremote.ssl=true"
-Dcom.sun.management.jmxremote.registry.ssl=true"
-Dcom.sun.management.jmxremote.ssl.need.client.auth=true"
-Djavax.net.ssl.keyStore=<path to java-app.keystore>"
-Djavax.net.ssl.keyStorePassword=<java-app.keystore的密码>"
-Djavax.net.ssl.trustStore=<path to java-app.truststore>"
-Djavax.net.ssl.trustStorePassword=<java-app.truststore的密码>"
```

配置SSL安全连接示例

[SSL+JMX](https://blog.csdn.net/jhonhai/article/details/108436795?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-2.control)

[远程安全验证](https://docs.oracle.com/javase/6/docs/technotes/guides/management/agent.html#gdenv)

## 插件

Java代码  ![收藏代码](http://jiajun.iteye.com/images/icon_star.png)

```shell
jconsole -pluginpath C:\Java\jdk1.6.0_22\demo\management\JTop\JTop.jar 
```

![image-20210811155023360](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210811155023.png)

一看便知，是个什么东西。

## 推荐使用升级版 JConsole 即 jvisualvm 。

关于jvisualvm的使用，->http://jiajun.iteye.com/blog/1180230