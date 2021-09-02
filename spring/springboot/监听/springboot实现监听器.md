##### 一.springboot 事件监听注解 @EventListener



##### 二.springboot事件类型

目前spring boot中支持的事件类型如下

1. **ApplicationFailedEvent：该事件为spring boot启动失败时的操作**
2. **ApplicationPreparedEvent：上下文context准备时触发**
3. **ApplicationReadyEvent：上下文已经准备完毕的时候触发**
4. **ApplicationStartedEvent：spring boot 启动监听类**
5. **SpringApplicationEvent：获取SpringApplication**
6. **ApplicationEnvironmentPreparedEvent：环境事先准备**



##### 三.注解使用

```java
@EventListener(ApplicationReadyEvent.class)
public void run(){
    log.info("容器启动完毕")
}
```



##### 四.代码实现监听器

第一：首先定义一个自己使用的监听器类并实现ApplicationListener接口。

```
@Componenpublic class MessageReceiver implements ApplicationListener<ApplicationReadyEvent> {
    private Logger logger = LoggerFactory.getLogger(MessageReceiver.class);
    
    private UserService userService = null;
    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        ConfigurableApplicationContext applicationContext = event.getApplicationContext();
　　　　　//解决userService一直为空
　　　　 userService = applicationContext.getBean(UserService.class); 　　　　 System.out.println("name"+userService.getName());
    }
}
```

第二：通过SpringApplication类中的addListeners方法将自定义的监听器注册进去

```
public class Application {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Application.class);
        application.addListeners(new MessageReceiver());
        application.run(args);
    
    }
}
```