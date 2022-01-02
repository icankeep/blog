## 手写一个自定义Spring Boot Starter

- [官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.developing-auto-configuration)
- [完整代码GitHub](https://github.com/icankeep/hello-spring-boot-starter)

<hr/>

1. 创建一个maven项目 hello-spring-boot-starter
2. 添加依赖
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
    </dependency>
    ```
3. 创建 `HelloProperties`， `HelloService`, `HelloAutoConfiguration`

    ```java
    // HelloProperties.java
    // ConfigurationProperties只有在EnableConfigurationProperties显示开启时才生效
   
    @ConfigurationProperties(prefix = "hello")
    public class HelloProperties {
        private String name;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    }
    ```
   
   ```java
   // HelloService的实现类 HelloServiceImpl.java
   
   public class HelloServiceImpl implements HelloService {
   
       @Resource
       private HelloProperties helloProperties;
   
       public String sayHello() {
           System.out.println("hello, " + helloProperties.getName());
           return "";
       }
   } 
   ```
   
   ```java
   // HelloAutoConfiguration.java
   
   @Configuration
   @ConditionalOnProperty(prefix = "hello", value = "enable", matchIfMissing = true)
   @EnableConfigurationProperties(HelloProperties.class)
   @ConditionalOnClass(HelloService.class)
   public class HelloAutoConfiguration {
   
       @Bean
       @ConditionalOnMissingBean(HelloService.class)
       public HelloService helloService() {
           return new HelloServiceImpl();
       }
   } 
   ```
   
4. 在resources目录下添加 `spring.factories`

    ```properties
    org.springframework.boot.autoconfigure.EnableAutoConfiguration=org.example.HelloAutoConfiguration
    ```
   
5. 整体目录结构如下，已完成基本的开发工作
    ```bash
    ├── pom.xml
    ├── src
    │   ├── main
    │   │   ├── java
    │   │   │   └── org
    │   │   │       └── example
    │   │   │           ├── HelloAutoConfiguration.java
    │   │   │           ├── HelloProperties.java
    │   │   │           └── service
    │   │   │               ├── HelloService.java
    │   │   │               └── impl
    │   │   │                   └── HelloServiceImpl.java
    │   │   └── resources
    │   │       └── META-INF
    │   │           └── spring.factories
    ```
   
6. mvn install 安装到本地maven仓库进行测试

7. spring-boot-web 测试模块引入依赖
    ```xml
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>hello-spring-boot-starter</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
    ```
   

