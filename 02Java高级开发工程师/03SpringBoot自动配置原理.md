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

