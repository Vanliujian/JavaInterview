# 自动配置原理2

##### 注解学习

###### @Configuration(proxyBeanMethods = true)

proxyBeanMethods默认为true，即在这个配置类中的Bean如果调用其他Bean的方法，那么不会再次创建Bean，而是会去IOC容器中取。

```java
@Configuration(proxyBeanMethods = true)
public class MyConfig {
 		@Resource
  	Bean2 bean2;
    @Bean
    public Bean1 bean1() {
        Bean2 bean2 = bean2();
        System.out.println("this is bean1's bean2 " + bean2);
        return new Bean1();
    }

    @Bean
    public Bean2 bean2() {
        return new Bean2();
    }
}
```

上述Resource导入的Bean2和bean1方法中的bean2是同一个，因为设置了proxyBeanMethods=true。

如果设置为false，那么在@Bean注解的方法中调用其它@Bean的方法，都会重新执行那个@Bean的方法，每次在@Bean方法中获取的另一个Bean都是不同的。

###### @EnableConfigurationProperties

设置@ConfigurationProperties(prefix="")配置生效，如果使用@Component也可以使配置生效，EnableConfigurationProperties和Component两者选择其一就可以。

###### @Conditional

<a href="https://docs.spring.io/spring-boot/docs/2.7.2/reference/html/features.html#features.developing-auto-configuration.condition-annotations">Conditional Annotation官网</a>

- @ConditionalOnClass：如果某个`类存在`就为true

- @ConditionalOnMissingClass：如果某个`类不存在`就为true

- @ConditionalOnBean：如果某个`Bean存在`就为true

- @ConditionalOnMissingBean：xxx

- @ConditionalOnProperty：检查某个属性的值是否符合要求，符合即为true

  - value：同name，表示属性的名称
  - name：同value，表示属性的名称
  - prefix：前缀
  - matchIfMissing：default false。如果属性不存在返回false，不注入。
  - havingValue：期望值

  注意点：name和value都是数组，并不是说可以判定多个属性值为什么不同的值，而是多个属性值是否为某一个同一个值。

  ```java
  // 判断name1和name2是否都为true
  @ConditionalOnProperty(prefix = "user", name = {"name1","name2"}, havingValue = "true")
  ```

- @ConditionalOnResource：存在特定资源时为true

  ```java
  @ConditionalOnResource(resources = "my-config.properties")
  ```

- @ConditionalOnWebApplication：如果是Web应用程序就为true

- @ConditionalOnNotWebApplication：xxx

- @ConditionalOnExpression：根据SpEL表达式的结果判断是否为true

  ```java
  // 不使用=号，而是使用:号。
  @ConditionalOnExpression(value = "#{${user.name1:true} && ${user.name2:true}}")
  ```

