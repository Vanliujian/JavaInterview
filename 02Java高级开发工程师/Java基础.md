# Java基础

## 函数式接口

- Function：有入参有出参，专注于对内容的加工，并返回加工后的内容

  ```java
  // 对输入的字符串数字进行乘以10并强转为Integer返回。
  public static Integer a(Function<String, Integer> function, String temp) {
      return function.apply(temp);
  }
  
  public static void main(String[] args) {
      String input = "2";
      Integer ans = a(s -> {
          s = s + "0";
          return Integer.valueOf(s);
      }, input);
      System.out.println(ans);
  }
  ```

- Consumer：不需要返回值，消费指定泛型的数据。

- Supplier：不需要入参，有返回值。

- Predicate：确定传入的对象是否满足某种条件，返回类型是boolean。

  ```java
  //如果list符合predict的规范，就返回符合的结果集。
  public static List<Integer> a(Predicate<Integer> predicate, List<Integer> list) {
      ArrayList<Integer> ans = new ArrayList<>();
      list.forEach(value -> {
          if (predicate.test(value)) ans.add(value);
      });
      return ans;
  }
  
  public static void main(String[] args) {
      ArrayList<Integer> list = new ArrayList<>();
      list.add(10);
      list.add(12);
      list.add(3);
      List<Integer> ans = a(value -> value >= 10, list);
      System.out.println(ans);
  }
  ```

## 泛型

1. 只在编译阶段有效，了解一下泛型擦除。

   - 验证只在编译阶段有效

     ```java
     ArrayList<Integer> list1 = new ArrayList<>();
     ArrayList<String> list2 = new ArrayList<>();
     list1.add(1);
     list2.add("2");
     System.out.println(list2.getClass() == list1.getClass());
     ```

     上述结果输出true，证明在此情况下确实在运行阶段是无效的。

2. 使用方式有哪几种？

   - 泛型类

   - 泛型方法

     ```java
     //这样是可以的，但是需要注意静态方法。如果getAge加了static修饰，就错了。
     class Teacher<T> {
         public void getAge(T t) {}
     }
     
     //静态方法与其他不同的是，如果静态方法也要用泛型，需要将静态方法也定义为泛型方法
     class Teacher<T> {
         public static <T> void getAge(T t) {}
     }
     ```

     

   - 泛型接口

     ```java
     interface Person<T> {}
     
     class Student<T> implements Person<T> {}
     // 当一个类实现泛型接口的时候，必须显示声明泛型参数，否则编译会报错
     // 错误示范：class Student implements Person<T> {}
     	
     ```

     





