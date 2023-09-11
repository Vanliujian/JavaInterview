# springboot

> pom.xml中parent、dependencyManagement、dependencies的关系。

首先parent项目中dependencies中的依赖，会全部自动依赖到子项目中。parent项目中的dependencyManagement里的依赖不会自动导入到子项目中。但是可以在子项目中手动导入，手动导入的时候不需要写版本号。

举例：

下图1是parent项目中的依赖，从3可以看出2不在parent项目中。

![image-20230909004130606](图片/image-20230909004130606.png)

下图是son01的依赖，如果写了parent标签，那么父项目的dependencies中的所有依赖都会被自动导入。只有手动添加了2，才会有2的依赖。

![image-20230909004404878](图片/image-20230909004404878.png)

> springboot优点

- 快速构建一个spring应用程序
- 嵌入的Tomcat、Jetty、Undertow，无须部署war文件。
- 提供starter poms来简化Maven配置，减少版本冲突。
- 对spring和第三方库提供默认配置，也可以修改默认值。
- 提供生产就绪型功能，如指标、健康检查和外部配置。
- 无须配置xml，无代码生成，开箱即用。

> 如果要打成jar包，需要添加如下插件的依赖，否则会报没有主清单属性的错误。

注：有时间可以了解一下MANIFEST.MF文件

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

> Application启动类

@SpringBootApplication注解添加在启动类上

> @ComponentScan扫描包

如果没写basePackages属性，就会自动设置当前启动类所在的包。

> Maven Pom和Maven Project区别

Maven Pom创建的时候默认没有目录，只有pom.xml文件，用来项目管理的。

Maven Project创建的时候有src/main/java目录。

> Spring Application

官网：<a href="https://docs.spring.io/spring-boot/docs/2.7.15/reference/html/features.html#features.spring-application">Spring Application</a>

可以通过这个类增加监听器等功能，也可以关闭banner图标。

加载外部配置文件。

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(Application.class);
        //关闭Banner图标显示。
        springApplication.setBannerMode(Banner.Mode.OFF);
        //加载外部配置文件
        //springApplication.setDefaultProperties();
        springApplication.run(args);
    }
}
```

如果要使用Spring Application，需要new出这个对象，并且类字节文件要在new的时候传进去。



# 课后题目

1. springboot的作用？
2. springboot有哪些特性？

