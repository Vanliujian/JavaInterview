# Protobuf

1. Protobuf是什么？

   是一种通讯协议，结构化数据存储格式。

2. Protobuf优点？

   - 支持多种语言
   - 高度压缩的通讯协议
   - 语言无关，平台无关，可序列化扩展

### 使用

1. 可以下载可执行程序，也可以在maven中添加插件执行。
2. 使用idea添加插件并在maven中添加依赖执行

- 首先定义好proto文件

  ```protobuf
  syntax="proto3";
  option java_package="com.protobuf";
  option java_outer_classname="MessageProto";
  
  message User {
    int32 id = 1;
    string name = 2;
    int32 age = 3;
  }
  
  message Result {
    int32 code = 1;
    string message = 2;
    string data = 3;
  }
  ```

  正常不需要message，因为使用protobuf的原因就是为了速度快，压缩精简。
  而且一般都是服务端和服务端的交互使用protobuf，前后端没有那种速度要求，所以前后端就使用正常的json。服务端和服务端都会提前定义好某个码代表的意思。

- 定义完proto文件后可以使用插件或者官网下载程序执行，将其转换为Java代码。

- 传输的时候在controller层使用protobuf格式接收

  @RequestMapping中，consumes就是服务端接收到的是什么格式，produces是服务端返回的是什么格式

  MessageProto.User就是生成的。

  至于它内部是要继续使用protobuf的User，还是转自己的User，这个看具体业务情况。如果速度要求很快，可以不转。

  ```java
  @ApiOperation("添加用户")
  @ApiImplicitParam(name = "user", value = "用户",dataType = "User",required = true,paramType = "body")
  @ApiResponse(code = 200,message = "success",response = Result.class)
  @RequestMapping(value = "/add", consumes = "application/x-protobuf",produces = "application/x-protobuf")
  public MessageProto.Result add(@RequestBody MessageProto.User user) {
  
    User user1 = new User(user.getId(), user.getName(), user.getAge());
    List<User> users = orderService.add(user1);
    MessageProto.Result.Builder builder = MessageProto.Result.newBuilder();
    builder.setCode(200);
    builder.setData(users.toString());
    MessageProto.Result build = builder.build();
    return build;
  }
  ```

pom依赖：

```xml
<dependency>
  <groupId>com.google.protobuf</groupId>
  <artifactId>protobuf-java-util</artifactId>
  <version>3.18.1</version>
</dependency>
<dependency>
  <groupId>com.googlecode.protobuf-java-format</groupId>
  <artifactId>protobuf-java-format</artifactId>
  <version>1.2</version>
</dependency>
```

插件：idea还需要安装插件protobuf或者genprotobuf

```xml
<build>
  <extensions>
    <extension>
      <groupId>kr.motd.maven</groupId>
      <artifactId>os-maven-plugin</artifactId>
      <version>1.5.0.Final</version>
    </extension>
  </extensions>
  <plugins>
    <plugin>
      <groupId>org.xolstice.maven.plugins</groupId>
      <artifactId>protobuf-maven-plugin</artifactId>
      <version>0.5.0</version>
      <configuration>
        <protocArtifact>
          com.google.protobuf:protoc:3.18.1:exe:${os.detected.classifier}
        </protocArtifact>
        <pluginId>grpc-java</pluginId>
        <pluginArtifact>
          io.grpc:protoc-gen-grpc-java:1.11.0:exe:${os.detected.classifier}
        </pluginArtifact>
      </configuration>
      <executions>
        <execution>
          <goals>
            <goal>compile</goal>
            <goal>compile-custom</goal>
          </goals>
        </execution>
      </executions>
    </plugin>

  </plugins>
</build>
```





> 补充

- grpc：

  是rpc的框架

  - 使用http2作为网络的传输层

  - 使用protobuf作为序列化协议

  - 通过protoc grpc插件生成易用的SDK