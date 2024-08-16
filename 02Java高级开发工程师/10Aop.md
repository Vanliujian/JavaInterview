# Aop

1. 什么是AOP？
   - 面向切面编程，OOP的补充

2. 为什么要AOP？

   - 解决OOP不能解决的问题

   - 代码复用

   - 比如我有一个统计接口访问次数要求，如果不使用AOP，就需要将每个接口被调用的时候各自的count++。使用aop以后，只要接口被调用，就记录接口的信息，最后通过aop日志统计访问次数。

3. AOP的重要概念

   1. 关注点：记录日志、权限控制、事物控制 ......
   2. 切入点、连接点
   3. 通知，通知类型（环绕、前置、后置）
   4. 切面类（切入点+通知）

4. AOP的使用

   导入依赖：

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-aop</artifactId>
   </dependency>
   ```

   编写切面

   切入点要和通知联系起来

   ```java
   @Aspect
   @Component
   public class OrderAspect {
   
       //execution里的表达式表示，返回值任意，com.order.OrderController下的任意方法，参数任意
       @Pointcut(value = "execution(* com.order.OrderController.*(..))")
       public void pointCut1() {}
   
       @Around("pointCut1()")
       public Object around1(ProceedingJoinPoint joinPoint) {
           //执行方法前要做的事情
           System.out.println("前置任务");
           Object proceed = null;
           try {
               proceed = joinPoint.proceed();
               System.out.println(proceed);
           } catch (Throwable e) {
               throw new RuntimeException(e);
           }
           //执行方法后要做的事情
           System.out.println("后置任务");
           return proceed;
       }
   }
   ```

   上述会将OrderController类中的所有方法都织入通知

   如果想要某个方法不织入通知，可以再定义一个切入点，在通知上使用&&和!符号。

   ```java
   @Aspect
   @Component
   public class OrderAspect {
   
       @Pointcut(value = "execution(* com.order.OrderController.*(..))")
       public void pointCut1() {}
   
       //这个切入点具体到某个方法
       @Pointcut(value = "execution(* com.order.OrderController.showAll())")
       public void pointCut2() {}
   
     	//这样showAll方法就不会被切入通知了
       @Around("pointCut1() && !pointCut2")
       public Object around1(ProceedingJoinPoint joinPoint) {
           //执行方法前要做的事情
           System.out.println("前置任务");
           Object proceed = null;
           try {
               proceed = joinPoint.proceed();
               System.out.println(JSON.toJSONString(proceed));
           } catch (Throwable e) {
               throw new RuntimeException(e);
           }
           //执行方法后要做的事情
           System.out.println("后置任务");
           return proceed;
       }
   }
   ```

   > 上述&&和!符可能很麻烦，还可以自定义注解来实现这种功能。

   1. 自定义注解类

      ```java
      @Retention(RetentionPolicy.RUNTIME)
      @Target(ElementType.METHOD)
      public @interface MyAspectAnnotation {
          String value() default "";
      }
      ```

   2. 切面类设计好

      ```java
      @Aspect
      @Component
      public class OrderAspect {
      
          //定义注解的切入点
          @Pointcut(value = "@annotation(com.annotation.MyAspectAnnotation)")
          public void pointCut3() {}
      
      
          @Around("pointCut3()")
          public Object around1(ProceedingJoinPoint joinPoint) {
              //执行方法前要做的事情
              System.out.println("前置任务");
              Object proceed = null;
              try {
                  proceed = joinPoint.proceed();
                  System.out.println(JSON.toJSONString(proceed));
              } catch (Throwable e) {
                  throw new RuntimeException(e);
              }
              //执行方法后要做的事情
              System.out.println("后置任务");
              return proceed;
          }
      }
      ```

   3. 在需要切入的方法上添加自定义的注解

      下面只在add、edit、showAll方法上添加了注解，只有这三个方法被织入了。

      ```java
      @RestController
      @RequestMapping("/order")
      @Api("订单接口")
      public class OrderController {
      
          @Resource
          OrderService orderService;
      
          @MyAspectAnnotation("添加用户")
          @ApiImplicitParam(name = "user", value = "用户",dataType = "User",required = true,paramType = "body")
          @ApiResponse(code = 200,message = "success",response = Result.class)
          @RequestMapping(value = "/add", consumes = "application/x-protobuf",produces = "application/x-protobuf")
          public MessageProto.Result add(@RequestBody MessageProto.User user) {
              //......
          }
      
      
          @ApiImplicitParam(name = "id", value = "用户id",dataType = "int",required = true,paramType = "path")
          @DeleteMapping("/{id}")
          public Result delete(@PathVariable int id) {
      				//......
          }
      
          @MyAspectAnnotation("修改用户")
          @PutMapping("/{id}")
          public Result edit(@RequestBody User user) {
      				//......
          }
      
          @ApiOperation("根据id查询用户")
          @ApiImplicitParam(name = "id", value = "用户id",dataType = "int",required = true)
          @GetMapping("/{id}")
          public Result<User> show(@PathVariable int id) {
              //......
          }
      
          @MyAspectAnnotation("查询所有用户")
          @ApiOperation("查询所有用户")
          @GetMapping("/showAll")
          public Result<List<User>> showAll() {
              //......
          }
      }
      ```

## JoinPoint

包含和切入相关的信息，比如切入点的对象方法属性等。

## ProceedingJoinPoint

继承了JoinPoint，但是暴露了proceed方法。可以通过proceed方法执行目标方法。

```java
public interface ProceedingJoinPoint extends JoinPoint {
    void set$AroundClosure(AroundClosure var1);

    default void stack$AroundClosure(AroundClosure arc) {
        throw new UnsupportedOperationException();
    }

  	//执行连接点方法，返回结果
    Object proceed() throws Throwable;

    Object proceed(Object[] var1) throws Throwable;
}
```

其父类JoinPoint提供的方法（其中三个）：

- Object[] getArgs()：获取连接点方法的参数数组

- Signature getSignature()：获取连接点方法的签名对象

- Object getTarget()：获取连接点所在的目标对象

  

## 顺序

当一个方法被多个Around切入的时候，可以设置他们的顺序，使用@Order注解。

aop，是这样执行的，把多个对于同一个方法的advice，先调用前面的执行，执行到proceed的时候，将后面的advice都嵌入到前面的proceed中。

即便有多个aspect切面切入，被切入环绕通知的方法是只执行一次的

```java
@Around("@annotation(myAspectAnnotation)")
public Object around2(ProceedingJoinPoint joinPoint,MyAspectAnnotation myAspectAnnotation) {
  log.info("qian");
  Object proceed;
  try {
    proceed = joinPoint.proceed();
    System.out.println(JSON.toJSONString(proceed));
  } catch (Throwable throwable) {
    throw new RuntimeException(throwable);
  }
  log.info("hou"+myAspectAnnotation.value());
  return proceed;
}


@Around("@annotation(apiOperation)")
public Object around1(ProceedingJoinPoint joinPoint,MyAspectAnnotation apiOperation) {
  log.info("qian API");
  Object proceed;
  try {
    proceed = joinPoint.proceed();
    System.out.println(JSON.toJSONString(proceed));
  } catch (Throwable throwable) {
    throw new RuntimeException(throwable);
  }
  log.info("hou APIOPERATION"+apiOperation.value());
  return proceed;
}
输出结果：
2023-12-20 22:58:02.055  INFO 33518 --- [nio-8081-exec-1] com.aspect.OrderAspect                   : qian API
2023-12-20 22:58:02.055  INFO 33518 --- [nio-8081-exec-1] com.aspect.OrderAspect                   : qian
=======================================
{"code":200,"data":[{"age":21,"id":1,"name":"yi"},{"age":22,"id":2,"name":"er"},{"age":23,"id":3,"name":"san"},{"age":24,"id":4,"name":"si"},{"age":25,"id":5,"name":"wu"}],"message":"success"}
2023-12-20 22:58:02.162  INFO 33518 --- [nio-8081-exec-1] com.aspect.OrderAspect                   : hou查询所有用户
{"code":200,"data":[{"age":21,"id":1,"name":"yi"},{"age":22,"id":2,"name":"er"},{"age":23,"id":3,"name":"san"},{"age":24,"id":4,"name":"si"},{"age":25,"id":5,"name":"wu"}],"message":"success"}
2023-12-20 22:58:02.163  INFO 33518 --- [nio-8081-exec-1] com.aspect.OrderAspect                   : hou APIOPERATION查询所有用户
```



# AOP案例







control + O重写方法

control + i 实现方法

command E 查看最近使用的类

option command B 实现类