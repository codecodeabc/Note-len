https://juejin.cn/post/6878269091996663815



##### 1.下载配置好gradle

##### 2.build.gradle 添加镜像加速

```
repositories {
			maven{ url 'http://maven.aliyun.com/nexus/content/groups/public'}
			mavenCentral()
			maven { url "https://repo.spring.io/libs-spring-framework-build" }
```

##### 3.在项目目录下执行命令

```bash
# mac或Linux 系统
./gradlew :spring-oxm:compileTestJava
# Windows 系统
gradlew :spring-oxm:compileTestJava
```

##### 4.导入idea即可

