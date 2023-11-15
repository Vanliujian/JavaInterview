# Swagger

作用：描述接口文档，前后端的桥梁

## 版本问题

springboot 2.6.0之前的版本spring.mvc.pathmatch.matching-strategy=ant-path-matcher

springboot 2.6.0之后的版本spring.mvc.pathmatch.matching-strategy=path_pattern_parser

swagger有swagger2和swagger3

1. 当swagger2和springboot 2.6.0之后的结合使用，如果不做任何处理会报错。
   - 原因：版本问题，可以在maven中查看各个版本依赖
   - 解决：使用合适的版本，在maven中找父依赖，一个一个找，最后使用合适的即可。

springboot 2.2.0.RELEASE 适配Swagger3

springboot 1.5.22 适配Swagger2

## Swagger2

pom.xml：

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```

配置类：

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .host("localhost:8081")//可不要
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com"))
                .paths(PathSelectors.any())
                .build();
    }
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("order-service")//微服务名称
                .description("订单swagger文档")
                .version("1.0")
                .contact(new Contact("van","www.vanliu.com","van@qq.com"))
                .build();
    }
}
```



## Swagger3

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

```java
@Configuration
@EnableOpenApi
public class SwaggerConfig3 {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.OAS_30)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com"))
                .build();
    }
}
```

