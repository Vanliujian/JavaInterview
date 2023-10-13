# 自动配置原理

###### 元注解(Meta-Annotations)

1. 定义：用于注解其他注解的注解。用于定义和配置自定义注解的注解。

2. 功能：元注解提供了自定义注解的元信息，保留策略，目标元素类型，是否被继承等信息。

3. 常用的元注解：

   - @Retention：指定自定义注解的保留策略，注解在何时有效。

     提供了三种策略:

     1. RetentionPolicy.SOURCE，不包含在编译后的字节码文件，也不在运行环境中。
     2. RetentionPolicy.CLASS，注解被编译时会保留在字节码文件中，但不会在运行环境中。
     3. RetentionPolicy.RUNTIME，注解会被编译保存到字节码，也可以在运行时通过反射获取。可以在程序运行时影响类的行为。

   - @Target：指定自定义注解的使用范围。

     接收一个ElementType数组，每个元素都表示可用于哪里。

     常见的ElementType类型：

     - ElementType.TYPE：用于类、接口、枚举等。
     - ElementType.FIELD：用于字段(包括枚举常量)。
     - ElementType.METHOD：用于方法。
     - ElementType.PARAMETER：用于方法参数
     - ElementType.CONSTRUCTOR：用于构造方法
     - ElementType.LOCAL_VARIABLE：用于局部变量
     - ElementType.ANNOTATION_TYPE：用于注解类型
     - ElementType.PACKAGE：用于包声明

   - @Documented：指定注解是否应该被包含在生成的文档中。

   - @Inherited：指定注解是否需要被子类继承。

   - @Repeatable：指定注解是否可以重复定义在一个元素上。

     - 举例：

       ```java
       public class Test1 {
           public static void main(String[] args) throws NoSuchMethodException {
               Test1 test1 = new Test1();
               Method a = test1.getClass().getMethod("a");
               Annotation[] annotations = a.getAnnotations();
               for (Annotation annotation: annotations) {
                   System.out.println(annotation);
                   //输出结果：
                   //@com.van.common.Annotation1s(value=[@com.van.common.Annotation1(name=name1), @com.van.common.Annotation1(name=name2)])
                   //只输出了一个Annotation1s，说明自动将他们合并为一个了。
               }
           }
           @Annotation1(name = "name1")
           @Annotation1(name = "name2")
           public static void a() {}
       }
       
       @Repeatable(Annotation1s.class)
       @interface Annotation1 {
           String name();
       }
       
       @Retention(RetentionPolicy.RUNTIME)
       @interface Annotation1s {
           Annotation1[] value();
       }
       ```

###### @ComponentScan

- basePackages：指定要扫描的包路径，默认是注解所在类所在的包及其子包。

- excludeFilters：设置排除规则，让哪些类不注入spring中。

  ==TypeExcludeFilter：==

  SpringBootApplication注解中的ComponentScan是这样写的

  ```java
  @ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
  		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
  public @interface SpringBootApplication {}
  ```

  我们如果要自己设置规则，那么需要写一个过滤类继承TypeExcludeFilter，重写match方法。

  ```java
  public class MyTypeExcludeFilter extends TypeExcludeFilter {
  
      @Override
      public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
          //如果是Hello类，就排除掉。
          //TODO metadataReader.getClassMetadata可以获取到类的信息。但不能通过其getClass方法来判断是否是某个类的对象。这里的getClass方法存疑，所以就用getClassName来判断
          return metadataReader.getClassMetadata().getClassName().equals(Hello.class.getName());
      }
  }
  ```

  类准备好后，在启动类上写@ComponentScan注解的时候，添加上这个过滤规则。

  ```java
  @ComponentScan(basePackages = "com.van",excludeFilters = {@Filter(type = FilterType.CUSTOM, classes = MyTypeExcludeFilter.class)})
  public class Application {}
  ```

  //针对上述问题有一个疑问，@ComponentScan应该不存在覆盖吧，这个设置规则和另一个规则冲突了会怎么样？例如我typeexcludefilter设置Hello不注入，typeexcludeFilter2设置全部注入，试了一下答案是不会注入。

  //还有一个问题，在SpringBootApplication中有ComponentScan这个注解，它设置的规则和我在外面启动类上设置的会有什么冲突吗？

  我猜测底层可能是每一个被Component类似注解修饰的类过来都会走一遍过滤规则，一个类过来就所有规则都过一遍，如果都符合才注入，否则不会。这个类判断结束再判断下一个类

  ==AutoConfigurationExcludeFilter：==

  SpringBootApplication注解中的ComponentScan还有AutoConfigurationExcludeFilter，用来排除所有配置类`且是自动配置类`。

  
