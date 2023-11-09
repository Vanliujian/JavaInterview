# RestTemplate

两个微服务之间如果要相互调用就可以使用RestTemplate，也可以使用spring cloud feign。

访问Restful服务

先写一个普通的增删改查服务，再用另一个服务RestTemplate调用

`order-service服务`端口号8081

```java
@RestController
@RequestMapping("/order")
public class OrderController {

    @Resource
    OrderService orderService;

 		//添加User
    @PostMapping("/add")
    public Result<List<User>> add(@RequestBody User user) {
        List<User> users = orderService.add(user);
        return new Result<>(200,"增加成功",users);
    }
		//根据id删除User
    @DeleteMapping("/{id}")
    public Result delete(@PathVariable int id) {
        orderService.delete(id);
        return Result.success();
    }
		//根据id修改User
    @PutMapping("/{id}")
    public Result edit(@RequestBody User user) {
        orderService.edit(user);
        return Result.success(orderService.showAll());
    }
		//根据id查询User
    @GetMapping("/{id}")
    public Result<User> show(@PathVariable int id) {
        User user = orderService.selectById(id);
        return Result.success(user);
    }
		//查询所有User
    @GetMapping("/showAll")
    public Result<List<User>> showAll() {
        return Result.success(orderService.showAll());
    }
}
```

使用RestTemplate调用服务

RestTemplate不会默认注入spring，需要我们导入，可以在使用template的类上的构造方法中传入RestTemplateBuilder来构造。

Autowired不仅可以使用在变量上，也可以使用在构造方法和set方法上。

```java

@RestController
@RequestMapping("/say")
public class HelloController {

    RestTemplate restTemplate;
    @Autowired
    public HelloController(RestTemplateBuilder restTemplateBuilder) {
        restTemplate = restTemplateBuilder.build();
    }
    @Resource
    HelloService helloService;

    @RequestMapping(method = RequestMethod.GET,path = "/add")
    public Result restAdd() {
        ResponseEntity<Result> responseEntity = restTemplate.postForEntity("http://localhost:8081/order/add", new User(7, "seven", 27), Result.class);
        System.out.println(responseEntity);
        return responseEntity.getBody();
    }

    @GetMapping("/rest/delete")
    public void restDelete() {
        restTemplate.delete("http://localhost:8081/order/{id}",1);
    }

    @GetMapping("/rest/edit")
    public void restEdit() {
        User user = new User(1, "zhangsan2", 13);
        //restTemplate的put和delete是没有返回值的
        restTemplate.put("http://localhost:8081/order/edit",user,Result.class);
    }

    //如果修改的时候使用put方法会没有返回值，可以使用exchange方法可以获得返回值。exchange和execute两个是万能的。
    @GetMapping("/rest/edit2")
    public String restEdit2() {
        User user = new User(1, "zhangsan2", 13);
        HttpEntity<Object> httpEntity = new HttpEntity<>(user);
        //其中exchange方法中的uriVariables参数对应url中的占位符
        ResponseEntity<Result> responseEntity = restTemplate.exchange("http://localhost:8081/order/{id}", HttpMethod.PUT, httpEntity, Result.class,1);
        return responseEntity.toString();
    }

    @GetMapping("/rest/selectById")
    public Result<User> restSelect() {
        //Result必须有无参构造函数，因为远程调用后会先调用无参构造再赋值到对象中。
        Result result = restTemplate.getForObject("http://localhost:8081/order/{id}", Result.class, 2);
        return result;
    }

    @GetMapping("/rest/showAll")
    public Result<List<User>> restSelectAll() {
        Result result = restTemplate.getForObject("http://localhost:8081/order/showAll", Result.class);
        return result;
    }
}
```

常用的方法：

- getForObject
- getForEntity
- postForObject
- postForEntity
- put
- delete
- exchange
- execute

在Test中也可以使用TestRestTemplate

TestRestTemplate也不会默认注入容器，可以通过new TestRestTemplate，或者在SpringBootTest中添加webEnvirement参数为RANDOM_PORT

写法一：注入，在注解上要添加webEnvirement

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class OrderTest {

    @Resource
    TestRestTemplate restTemplate;
    @Test
    public void orderAddTest() {
        ResponseEntity<Result> responseEntity = restTemplate.exchange("http://localhost:8081/order/{id}", HttpMethod.DELETE, null, Result.class, 1);
        System.out.println(responseEntity.getBody());
    }
}
```

写法二：new一个出来

```java
@SpringBootTest
public class OrderTest {
  
    TestRestTemplate restTemplate = new TestRestTemplate();
    @Test
    public void orderAddTest() {
        ResponseEntity<Result> responseEntity = restTemplate.exchange("http://localhost:8081/order/{id}", HttpMethod.DELETE, null, Result.class, 1);
        System.out.println(responseEntity.getBody());
    }
}
```



> 注意

- RestTemplate：阻塞的
- WebClient：无阻塞的，依赖webflux