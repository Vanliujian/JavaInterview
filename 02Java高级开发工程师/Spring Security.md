# Spring Security

导入pom配置：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

1. 基于application.yml设置用户名和密码

   ```java
   //配置密码编码
   @Bean
   public PasswordEncoder passwordEncoder() {
       return NoOpPasswordEncoder.getInstance();
   }
   ```

   ```yaml
   spring:
     security:
       user:
         name: van
         password: liujian
   ```

2. 基于UserDetailsService设置用户名和密码

   ```java
   //配置密码编码
   @Bean
   public PasswordEncoder passwordEncoder() {
       return NoOpPasswordEncoder.getInstance();
   }
   ```

   ```java
   @Component
   public class MyUserDetailService implements UserDetailsService {
       @Resource
       PasswordEncoder passwordEncoder;
     
       @Override
       public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
           String encode = passwordEncoder.encode("van");
           //TODO 查询 mysql
           return new User("van",encode, AuthorityUtils.commaSeparatedStringToAuthorityList("admin,user"));
       }
   }
   ```

3. 继承WebSecurityConfigurerAdapter

   继承这个类实现configure方法，其中还可以根据UserDetailsService实现。也可以基于内存。

   ```java
   @Component
   public class MyUserDetailService implements UserDetailsService {
       @Resource
       PasswordEncoder passwordEncoder;
       @Override
       public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
           String encode = passwordEncoder.encode("van");
           //TODO 查询 mysql
           return new User("van",encode, AuthorityUtils.commaSeparatedStringToAuthorityList("admin,user"));
       }
   }
   ```

   ```java
   @Configuration
   public class SecurityConfig extends WebSecurityConfigurerAdapter {
   
       @Bean
       PasswordEncoder passwordEncoder() {
           return new BCryptPasswordEncoder();
       }
   
       @Resource
       UserDetailsService userDetailsService;
   
       //AuthenticationManager认证管理器
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           //这个可以不写，spring会自己拿到
           auth.userDetailsService(userDetailsService);
       }
     
       //基于内存
   //    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
   //      auth.inMemoryAuthentication()
   //        .withUser("van")
   //        .password(passwordEncoder().encode("van"))
   //        .authorities("admin");
   //    }
   }
   ```

   