---
layout: post
title: Spring Boot
categories: Java
description: Java笔记
keywords: Java
---

# 引言
Spring Boot笔记，集成mybatis、使用spring-boot-devtools、打包、瘦身、Actuator、Profiles、Conditional、加载配置文件、禁用自动配置、Filter、生命周期、集成Open API、Redis、Artemis、RabbitMQ、Kafka。


# Spring Boot

Spring框架，它的主要功能包括IoC容器、AOP支持、事务支持、MVC开发以及强大的第三方集成功能等。而Spring Boot是一个基于Spring的套件，它帮我们预组装了Spring的一系列组件，以便以尽可能少的代码和配置来开发基于Spring的Java应用程序。

Spring Boot的目标就是提供一个开箱即用的应用程序架构，基于Spring Boot的预置结构继续开发，省时省力。

Spring Boot3.x版与Spring Boot 2.x版本，两者有以下不同：

|              | Spring Boot 2.x  | Spring Boot 3.x    |
| :----------- | :--------------- | :----------------- |
| Spring版本   | Spring 5.x       | Spring 6.x         |
| JDK版本      | >= 1.8           | >= 17              |
| Tomcat版本   | 9.x              | 10.x               |
| Annotation包 | javax.annotation | jakarta.annotation |
| Servlet包    | javax.servlet    | jakarta.servlet    |
| JMS包        | javax.jms        | jakarta.jms        |
| JavaMail包   | javax.mail       | jakarta.mail       |

如果使用Spring Boot的其他版本，则需要根据需要调整代码。

[Spring Boot的官网](https://spring.io/projects/spring-boot)。

## 标准Spring Boot应用

新建一个`springboot-hello`的工程，创建标准的Maven目录结构如下：

```ascii
springboot-hello
├── pom.xml
├── src
│   └── main
│       ├── java
│       └── resources
│           ├── application.yml
│           ├── logback-spring.xml
│           ├── static
│           └── templates
└── target
```

其中，在`src/main/resources`目录下：

**application.yml**

是Spring Boot默认的配置文件，它采用[YAML](https://yaml.org/)格式而不是`.properties`格式，**文件名必须是`application.yml`而不是其他名称。**

YAML格式比`key=value`格式的`.properties`文件更易读。比较一下两者的写法：

使用`.properties`格式：

```properties
# application.properties

spring.application.name=${APP_NAME:unnamed}

spring.datasource.url=jdbc:hsqldb:file:testdb
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driver-class-name=org.hsqldb.jdbc.JDBCDriver

spring.datasource.hikari.auto-commit=false
spring.datasource.hikari.connection-timeout=3000
spring.datasource.hikari.validation-timeout=3000
spring.datasource.hikari.max-lifetime=60000
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=1
```

使用YAML格式：

```yaml
# application.yml

spring:
  application:
    name: ${APP_NAME:unnamed}
  datasource:
    url: jdbc:hsqldb:mem:testdb
    username: sa
    password:
    driver-class-name: org.hsqldb.jdbc.JDBCDriver
    hikari:
      auto-commit: false
      connection-timeout: 3000
      validation-timeout: 3000
      max-lifetime: 60000
      maximum-pool-size: 20
      minimum-idle: 1
```

可见，YAML是一种层级格式，它和`.properties`很容易互相转换，它的优点是去掉了大量重复的前缀，并且更加易读。

 **也可以使用`application.properties`作为配置文件**，但不如YAML格式简单。

**使用环境变量**

在配置文件中，经常使用如下的格式对某个key进行配置：

```yaml
app:
  db:
    host: ${DB_HOST:localhost}
    user: ${DB_USER:root}
    password: ${DB_PASSWORD:password}
```

这种`${DB_HOST:localhost}`意思是，首先从环境变量查找`DB_HOST`，如果环境变量定义了，那么使用环境变量的值，否则，使用默认值`localhost`。

这使得我们在开发和部署时更加方便，因为开发时无需设定任何环境变量，直接使用默认值即本地数据库，而实际线上运行的时候，只需要传入环境变量即可：

```bash
$ DB_HOST=10.0.1.123 DB_USER=prod DB_PASSWORD=xxxx java -jar xxx.jar
```

------

**logback-spring.xml**是Spring Boot的logback配置文件名称（也可以使用`logback.xml`），一个标准的写法如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <appender name="APP_LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
          <file>app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <maxIndex>1</maxIndex>
            <fileNamePattern>app.log.%i</fileNamePattern>
        </rollingPolicy>
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>1MB</MaxFileSize>
        </triggeringPolicy>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="APP_LOG" />
    </root>
</configuration>
```

它主要通过`<include resource="..." />`引入了Spring Boot的一个缺省配置，这样我们就可以引用类似`${CONSOLE_LOG_PATTERN}`这样的变量。上述配置定义了一个控制台输出和文件输出，可根据需要修改。

`static`是静态文件目录，`templates`是模板文件目录，它们不再存放在`src/main/webapp`下，而是直接放到`src/main/resources`这个classpath目录，因为在Spring Boot中已经不需要专门的webapp目录了。

以上就是Spring Boot的标准目录结构，它完全是一个基于Java应用的普通Maven项目。

源码的目录结构：

```ascii
src/main/java
└── com.aotmd
    ├── Application.java
    ├── entity
    │   └── User.java
    ├── service
    │   └── UserService.java
    └── web
        └── UserController.java
```

在存放源码的`src/main/java`目录中，Spring Boot对Java包的层级结构有一个要求。根package是`com.aotmd`，下面还有`entity`、`service`、`web`等子package。Spring Boot要求`main()`方法所在的启动类必须放到根package下，命名不做要求，这里以`Application.java`命名，它的内容如下：

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }
}
```

启动Spring Boot应用程序只需要一行代码加上一个注解`@SpringBootApplication`，该注解实际上又包含了：

- `@SpringBootConfiguration`
  - `@Configuration`
- `@EnableAutoConfiguration`
  - `@AutoConfigurationPackage`
- `@ComponentScan`

这样一个注解就相当于启动了自动配置和自动扫描。

`pom.xml`内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>spring-boot-hello</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <!-- 集成Pebble View -->
        <!-- https://mvnrepository.com/artifact/io.pebbletemplates/pebble-spring-boot-starter -->
        <dependency>
            <groupId>io.pebbletemplates</groupId>
            <artifactId>pebble-spring-boot-starter</artifactId>
            <version>3.1.3</version>
        </dependency>


        <!-- JDBC驱动 -->
        <dependency>
            <groupId>org.hsqldb</groupId>
            <artifactId>hsqldb</artifactId>
        </dependency>
    </dependencies>
</project>
```

使用Spring Boot时，强烈推荐从`spring-boot-starter-parent`继承，因为这样就可以引入Spring Boot的预置配置。

紧接着，引入了依赖`spring-boot-starter-web`和`spring-boot-starter-jdbc`，它们分别引入了Spring MVC相关依赖和Spring JDBC相关依赖，无需指定版本号，因为引入的`<parent>`内已经指定了，只有我们自己引入的某些第三方jar包需要指定版本号。这里引入`pebble-spring-boot-starter`作为View，以及`hsqldb`作为嵌入式数据库。`hsqldb`已在`spring-boot-starter-jdbc`中预置了版本号，因此此处无需指定版本号。

**第三方jar的版本兼容性可以查看[mvnrepository](https://mvnrepository.com/artifact/io.pebbletemplates/pebble-spring-boot-starter/3.1.3)下的Compile Dependencies查看兼容性。**

根据`pebble-spring-boot-starter`的[文档](https://pebbletemplates.io/wiki/guide/spring-boot-integration/)，加入如下配置到`application.yml`：

```yaml
pebble:
  # 默认为".peb"，改为"":
  suffix:
  # 开发阶段禁用模板缓存:
  cache: false
  prefix: /templates/
```

对`Application`稍作改动，添加`WebMvcConfigurer`这个Bean：

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    @Bean
    WebMvcConfigurer createWebMvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addResourceHandlers(ResourceHandlerRegistry registry) {
                // 映射路径`/static/`到classpath路径:
                registry.addResourceHandler("/static/**")
                        .addResourceLocations("classpath:/static/");
            }
        };
    }
}
```

现在就可以直接运行`Application`，Spring Boot自动启动了嵌入式Tomcat，当看到`Started Application in xxx seconds`时，Spring Boot应用启动成功。

添加测试：

```java
@Controller
class TestController{
    @GetMapping("/test1")
    public ModelAndView test(){
        Map<String, String> map = new HashMap<>();
        map.put("body","你好");
        map.put("title","测试");
        return new ModelAndView("test.html",map);
    }
}
```

`test.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{title}}</title>
</head>
<body>
{{body}}
</body>
</html>
```

访问：`http://localhost:8080/test1`：

显示：`你好`

------

之前我们定义的数据源、声明式事务、JdbcTemplate在哪创建的？怎么就可以直接注入到自己编写的`UserService`中呢？

这些自动创建的Bean就是Spring Boot的特色：AutoConfiguration。

当引入`spring-boot-starter-jdbc`时，启动时会自动扫描所有的`XxxAutoConfiguration`：

- `DataSourceAutoConfiguration`：自动创建一个`DataSource`，其中配置项从`application.yml`的`spring.datasource`读取；
- `DataSourceTransactionManagerAutoConfiguration`：自动创建了一个基于JDBC的事务管理器；
- `JdbcTemplateAutoConfiguration`：自动创建了一个`JdbcTemplate`。

因此，自动得到了一个`DataSource`、一个`DataSourceTransactionManager`和一个`JdbcTemplate`。

类似的，当引入`spring-boot-starter-web`时，自动创建了：

- `ServletWebServerFactoryAutoConfiguration`：自动创建一个嵌入式Web服务器，默认是Tomcat；
- `DispatcherServletAutoConfiguration`：自动创建一个`DispatcherServlet`；
- `HttpEncodingAutoConfiguration`：自动创建一个`CharacterEncodingFilter`；
- `WebMvcAutoConfiguration`：自动创建若干与MVC相关的Bean。
- ...

引入第三方`pebble-spring-boot-starter`时，自动创建了：

- `PebbleAutoConfiguration`：自动创建了一个`PebbleViewResolver`。

Spring Boot大量使用`XxxAutoConfiguration`来使得许多组件被自动化配置并创建，而这些创建过程又大量使用了Spring的Conditional功能。例如`JdbcTemplateAutoConfiguration`，它的代码如下：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
@ConditionalOnSingleCandidate(DataSource.class)
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
@EnableConfigurationProperties(JdbcProperties.class)
@Import({ JdbcTemplateConfiguration.class, NamedParameterJdbcTemplateConfiguration.class })
public class JdbcTemplateAutoConfiguration {
}
```

当满足条件：

- `@ConditionalOnClass`：在classpath中能找到`DataSource`和`JdbcTemplate`；
- `@ConditionalOnSingleCandidate(DataSource.class)`：在当前Bean的定义中能找到唯一的`DataSource`；

该`JdbcTemplateAutoConfiguration`就会起作用。实际创建由导入的`JdbcTemplateConfiguration`完成：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnMissingBean(JdbcOperations.class)
class JdbcTemplateConfiguration {
    @Bean
    @Primary
    JdbcTemplate jdbcTemplate(DataSource dataSource, JdbcProperties properties) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        JdbcProperties.Template template = properties.getTemplate();
        jdbcTemplate.setFetchSize(template.getFetchSize());
        jdbcTemplate.setMaxRows(template.getMaxRows());
        if (template.getQueryTimeout() != null) {
            jdbcTemplate.setQueryTimeout((int) template.getQueryTimeout().getSeconds());
        }
        return jdbcTemplate;
    }
}
```

创建`JdbcTemplate`之前，要满足`@ConditionalOnMissingBean(JdbcOperations.class)`，即不存在`JdbcOperations`的Bean。

如果自己创建了一个`JdbcTemplate`，例如，在`Application`中自己写个方法：

```java
@SpringBootApplication
public class Application {
    ...
    @Bean
    JdbcTemplate createJdbcTemplate(@Autowired DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

那么根据条件`@ConditionalOnMissingBean(JdbcOperations.class)`，Spring Boot就不会再创建一个重复的`JdbcTemplate`（因为`JdbcOperations`是`JdbcTemplate`的父类）。

可见，Spring Boot自动装配功能是通过自动扫描+条件装配实现的，这一套机制在默认情况下工作得很好，但是，如果要手动控制某个Bean的创建，就需要详细地了解Spring Boot自动创建的原理，很多时候还要跟踪`XxxAutoConfiguration`，以便设定条件使得某个Bean不会被自动创建。

------

## 集成mybatis

引入对应版本的依赖即可：

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.1</version>
</dependency>
```

yml文件添加映射文件位置：

```yml
mybatis:
  mapper-locations: classpath:mapper/*.xml

logging:
  level:
    root: INFO
    com.aotmd: DEBUG
    org.mybatis: DEBUG
```

然后添加之前Spring的非配置部分：

```java
class User{
    private long id;
    private String email;
    private String password;
    private String name;
    private Long createdAt;
    public User() {}
    public String getEmail() {
        return email;
    }
    public String getPassword() {
        return password;
    }
    public String getName() {
        return name;
    }
    public void setEmail(String email) {
        this.email = email;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    public void setName(String name) {
        this.name = name;
    }

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public Long getCreatedAt() {
        return createdAt;
    }

    public void setCreatedAt(Long createdAt) {
        this.createdAt = createdAt;
    }

    @Override public String toString() {return "User{id=" + id + ", email='" + email + '\'' + ", password='" + password + '\'' + ", name='" + name + '\'' + ", createdAt=" + createdAt + '}'; }
}

interface UserMapper {
    User getById(@Param("id") long id);
    List<User> getAll(@Param("offset") int offset, @Param("maxResults") int maxResults);
    int insert(@Param("user") User user);
    int insert2(@Param("user") User user);
    int updateName(@Param("user") User user);
    int deleteById(@Param("id") long id);
    User getByEmailAndByPassword(@Param("user") User user);
}

@Component
class init{
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @PostConstruct
    public void init() {
        //在Spring容器启动时自动创建一个users表
        jdbcTemplate.update("DROP TABLE IF EXISTS users");
        jdbcTemplate.update("CREATE TABLE IF NOT EXISTS users ("
                + "id BIGINT IDENTITY NOT NULL PRIMARY KEY, "
                + "email VARCHAR(100) NOT NULL, "
                + "password VARCHAR(100) NOT NULL, "
                + "name VARCHAR(100) NOT NULL, "
                + "createdAt BIGINT NOT NULL, "
                + "UNIQUE (email))");
    }

}

@Component
class UserService {
    // 注入UserMapper:
    @Autowired
    UserMapper userMapper;

    public User getById(long id) {
        // 调用Mapper方法:
        return userMapper.getById(id);
    }
    public List<User> getAll(int offset,int maxResults){
        List<User> all = userMapper.getAll(offset, maxResults);
        return all;
    }
    public User register(String email, String password, String name){
        // 创建一个User对象:
        User user = new User();
        // 设置好各个属性:
        user.setEmail(email);
        user.setPassword(password);
        user.setName(name);
        user.setCreatedAt(System.currentTimeMillis());
        // 不要设置id，因为使用了自增主键
        // 保存到数据库:
        userMapper.insert2(user);
        // 现在已经自动获得了id:
        System.out.println(user.getId());
        return user;
    }

    public int updateEmail(String name, long id){
        // 创建一个User对象:
        User user = new User();
        // 设置好各个属性:
        user.setName(name);
        user.setId(id);
        int i = userMapper.updateName(user);
        System.out.println(i);
        return i;
    }
    public int delete(long id){
        int i = userMapper.deleteById(id);
        return i;
    }

    public User login(String email, String password){
        // 创建一个User对象:
        User user = new User();
        // 设置好各个属性:
        user.setEmail(email);
        user.setPassword(password);
        user=userMapper.getByEmailAndByPassword(user);
        return user;
    }
}

@RestController
@RequestMapping("/api")
class ApiController {
    @Autowired
    UserService userService;

    @GetMapping("/users")
    public List<User> users() {
        return userService.getAll(0,5);
    }

    @GetMapping("/users/{id}")
    public User user(@PathVariable("id") long id) {
        return userService.getById(id);
    }

    @PostMapping("/signin")
    public Map<String, Object> signin(@RequestBody SignInRequest signinRequest) {
        try {
            User user = userService.register(signinRequest.email, signinRequest.password,signinRequest.name);
            return Map.of("user", user);
        } catch (Exception e) {
            return Map.of("error", "SIGNIN_FAILED", "message", e.getMessage());
        }
    }

    public static class SignInRequest {
        public String email;
        public String password;
        public String name;
    }
}
```

映射文件也与之前相同：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.aotmd.UserMapper"><!--命名空间为接口名称-->
    <!--id需与接口方法完全一致,parameterType形参,resultType返回值类型-->
    <select id="getById" parameterType="Long" resultType="com.aotmd.User">
        SELECT *
        FROM users
        WHERE id = #{id}
    </select>
    <select id="getAll"  resultType="com.aotmd.User">
        SELECT *
        FROM users LIMIT #{offset}, #{maxResults}
    </select>
    <select id="getByEmailAndByPassword" resultType="com.aotmd.User">
        SELECT *
        FROM users
        WHERE email = #{user.email} and
        password=#{user.password}
    </select>
    <insert id="insert" parameterType="com.aotmd.User">
        INSERT INTO users (email, password, name, createdAt)
        VALUES (#{user.email}, #{user.password}, #{user.name}, #{user.createdAt})
    </insert>
    <insert id="insert2" parameterType="com.aotmd.User" useGeneratedKeys="true" keyProperty="id" keyColumn="id">
        INSERT INTO users (email, password, name, createdAt)
        VALUES (#{user.email}, #{user.password}, #{user.name}, #{user.createdAt})
    </insert>

    <update id="updateName" parameterType="com.aotmd.User">
        UPDATE users
        SET name = #{user.name}
        WHERE id = #{user.id}
    </update>

    <delete id="deleteById" parameterType="long">
        DELETE
        FROM users
        WHERE id = #{id}
    </delete>
</mapper>
```

启动，然后post访问`http://localhost:8080/api/signin`，附带json：`{"email":"tom@example.com","password":"tomcat","name":"test"}`，返回body：`{"user":{"id":0,"email":"tom@example.com","password":"tomcat","name":"test","createdAt":***}}`

再get访问`http://localhost:8080/api/users`，得到：`[{"id":0,"email":"tom@example.com","password":"tomcat","name":"test","createdAt":***}]`

功能正常。

## spring-boot-devtools

在开发阶段，我们经常要修改代码，然后重启Spring Boot应用。经常手动停止再启动，比较麻烦。

Spring Boot提供了一个开发者工具，可以监控classpath路径上的文件。只要源码或配置文件发生修改，Spring Boot应用可以自动重启。在开发阶段，这个功能比较有用。

要使用这一开发者功能，只需添加如下依赖到`pom.xml`：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

然后，没有然后了。直接启动应用程序，然后试着修改源码，保存，观察日志输出，Spring Boot会自动重新加载。

默认配置下，针对`/static`、`/public`和`/templates`目录中的文件修改，不会自动重启，因为禁用缓存后，这些文件的修改可以实时更新。

------

如果没有效果，那需要修改以下设置：

1. 设置IDEA的编译器：
   - File->Settings…->Build,Execution,Deployment->Compiler，勾选"Build project automatically"
   - 文件->设置...->构建、执行、部署->编译器，勾选"自动构建项目"

2. 应用程序运行时允许编译器自动生成：
   - 在IntellijIDEA中：按Ctrl+Shift+a，然后键入“注册表”并点击它。然后启用选项“compiler.Automake.Allow.When.app.Running”。
   - 在新版本这个选项已经被移到了高级设置中，文件->设置...->高级设置->编译器栏->“即使开发的应用程序当前正在运行，也允许自动make启动”。

   

还有可能是项目名称的问题：

在决定类路径上的条目更改时是否应触发重启时，**DevTools会自动忽略名为：**`Spring-Boot`、`Spring-Boot-DevTools`、`Spring-Boot-Autoconfiguration`、`Spring-Boot-Actuator`和`Spring-Boot-starter`的项目。

使用Ctrl+F9构建项目**会自动触发重新启动**。如果您希望在保存类文件后立即自动触发，可以按照问题中提供的热插拔链接进行操作。

Spring Boot还具有在特定文件发生更改时触发重新启动的选项，可以使用以下属性在应用程序中配置该选项

> spring.devtools.restart.trigger-file=
>
> Spring.devtools.restart.rigger-file=

参见： [Spring Boot Developer Tools Auto restart doesn't work in IntelliJ](https://stackoverflow.com/questions/53569745/spring-boot-developer-tools-auto-restart-doesnt-work-in-intellij)

## 打包Spring Boot应用

在Spring Boot应用中，Spring Boot自带一个更简单的`spring-boot-maven-plugin`插件用来打包，只需要在`pom.xml`中加入以下配置：

```xml
<project ...>
    ...
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

无需任何配置，Spring Boot的这款插件会自动定位应用程序的入口Class，执行以下Maven命令即可打包：

```bash
mvn clean package
```

以`spring-boot-hello`项目为例，打包后在`target`目录下可以看到两个jar文件：

```text
spring-boot-hello-1.0-SNAPSHOT.jar
spring-boot-hello-1.0-SNAPSHOT.jar.original
```

其中，`spring-boot-hello-1.0-SNAPSHOT.jar.original`是Maven标准打包插件打的jar包，它只包含我们自己的Class，不包含依赖，而`spring-boot-hello-1.0-SNAPSHOT.jar`是Spring Boot打包插件创建的包含依赖的jar，可以直接运行：

```bash
java -jar springboot-exec-jar-1.0-SNAPSHOT.jar
```

这样，部署一个Spring Boot应用就非常简单，无需预装任何服务器，只需要上传jar包即可。

在打包的时候，因为打包后的Spring Boot应用不会被修改，因此，默认情况下，`spring-boot-devtools`这个依赖不会被打包进去。但是要注意，使用早期的Spring Boot版本时，需要配置一下才能排除`spring-boot-devtools`这个依赖：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <excludeDevtools>true</excludeDevtools>
    </configuration>
</plugin>
```

如果不喜欢默认的项目名+版本号作为文件名，可以加一个配置指定文件名：

```xml
<project ...>
    ...
    <build>
        <finalName>awesome-app</finalName>
        ...
    </build>
</project>
```

这样打包后的文件名就是`awesome-app.jar`。

------

在 IntelliJ IDEA 中运行打包好的 Spring Boot JAR 文件，可以按照以下步骤操作：

**方法 1：使用终端运行 JAR 文件**

1. 打开 IntelliJ IDEA 并导航到终端（View -> Tool Windows -> Terminal）。
2. 切换到包含 JAR 文件的目录，例如：
    ```bash
    cd target
    ```
3. 使用以下命令运行 JAR 文件：
    ```bash
    java -jar your-application.jar
    ```
    替换 `your-application.jar` 为实际的 JAR 文件名。

**方法 2：创建运行配置**

1. 打开 IntelliJ IDEA 并导航到 "Run/Debug Configurations"（Run -> Edit Configurations）。
2. 点击左上角的 `+` 号，选择 `Application`。
3. 配置运行配置：
    - **Name**: 运行配置的名称，例如 `Run Jar`.
    - **Main class**: 选择 `org.springframework.boot.loader.JarLauncher`。
    - **Program arguments**: 填写 JAR 文件的路径，例如 `path/to/your-application.jar`。
4. 点击 "OK" 保存运行配置。
5. 在 IntelliJ IDEA 的工具栏上选择新创建的运行配置，然后点击 "Run" 按钮。

**方法 3：使用 JAR 文件配置**

1. 打开 IntelliJ IDEA 并导航到 "Run/Debug Configurations"（Run -> Edit Configurations）。
2. 点击左上角的 `+` 号，选择 `JAR Application`。
3. 配置运行配置：
    - **Name**: 运行配置的名称，例如 `Run Jar`.
    - **Path to JAR**: 选择 JAR 文件的路径，例如 `target/your-application.jar`。
    - **Working Directory**: 设置为项目的根目录。
    - **VM options**: 根据需要填写，例如 `-Xmx1024m`。
4. 点击 "OK" 保存运行配置。
5. 在 IntelliJ IDEA 的工具栏上选择新创建的运行配置，然后点击 "Run" 按钮。



## 瘦身Spring Boot应用

使用Spring Boot提供的`spring-boot-maven-plugin`打包Spring Boot应用，可以直接获得一个完整的可运行的jar包，把它上传到服务器上再运行就极其方便。

但是这种方式也不是没有缺点。最大的缺点就是包太大了，动不动几十MB，在网速不给力的情况下，上传服务器非常耗时。引用到的Tomcat、Spring和其他第三方组件，只要版本号不变，这些jar就相当于每次都重复打进去，再重复上传了一遍。

真正经常改动的代码其实是自己编写的代码。如果只打包自己编写的代码，通常jar包也就几百KB。但是，运行的时候，classpath中没有依赖的jar包，肯定会报错。

如何只打自己编写的代码，同时又自动把依赖包下载到某处，并自动引入到classpath中。解决方案就是使用`spring-boot-thin-launcher`。

修改`<build>`-`<plugins>`-`<plugin>`，给原来的`spring-boot-maven-plugin`增加一个`<dependency>`如下：

```xml
<project ...>
    ...
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <dependencies>
                    <dependency>
                        <groupId>org.springframework.boot.experimental</groupId>
                        <artifactId>spring-boot-thin-layout</artifactId>
                        <version>1.0.31.RELEASE</version>
                    </dependency>
                </dependencies>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot.experimental</groupId>
                <artifactId>spring-boot-thin-maven-plugin</artifactId>
                <version>1.0.31.RELEASE</version>
                <executions>
                    <!--在构建时下载依赖项-->
                    <execution>
                        <id>resolve</id>
                        <goals>
                            <goal>resolve</goal>
                        </goals>
                        <inherited>false</inherited>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    <!-- 阿里云maven仓库 -->
    <repositories>
        <repository>
            <id>public</id>
            <name>aliyun nexus</name>
            <url>https://maven.aliyun.com/repository/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>public</id>
            <name>aliyun nexus</name>
            <url>https://maven.aliyun.com/repository/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
</project>
```

如果无法自动下载：

```xml
<dependency>
    <groupId>org.springframework.boot.experimental</groupId>
    <artifactId>spring-boot-thin-layout</artifactId>
    <version>1.0.31.RELEASE</version>
</dependency>
```

可以把它加入到

```xml
<project ...>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot.experimental</groupId>
			<artifactId>spring-boot-thin-layout</artifactId>
			<version>1.0.31.RELEASE</version>
		</dependency>
	</dependencies>
</project>
```

加载完毕后删除。

然后不需要任何其他改动了，直接按正常的流程打包，执行`mvn clean package`，`target`目录最终生成的可执行`spring-boot-hello-1.0-SNAPSHOT.jar`只有79KB左右。

直接运行`java -jar spring-boot-hello-1.0-SNAPSHOT.jar`，效果和上一节完全一样。显然，79KB的jar肯定无法放下Tomcat和Spring。那么，运行时这个`spring-boot-hello-1.0-SNAPSHOT.jar`又是怎么找到它自己依赖的jar包呢？

实际上`spring-boot-thin-launcher`这个插件改变了`spring-boot-maven-plugin`的默认行为。它输出的jar包只包含自己代码编译后的class，一个很小的`ThinJarWrapper`，以及解析`pom.xml`后得到的所有依赖jar的列表。

运行的时候，入口实际上是`ThinJarWrapper`，它会先在指定目录搜索看看依赖的jar包是否都存在，如果不存在，先从Maven中央仓库下载到本地，然后，再执行我们自己编写的`main()`入口方法。这种方式有点类似很多在线安装程序：用户下载后得到的是一个很小的exe安装程序，执行安装程序时，会首先在线下载所需的若干巨大的文件，再进行真正的安装。

这个`spring-boot-thin-launcher`在启动时搜索的默认目录是用户主目录的`.m2`，也可以指定下载目录，例如，将下载目录指定为当前目录：

```bash
java -Dthin.root=. -jar spring-boot-hello-1.0-SNAPSHOT.jar
```

上述命令通过环境变量`thin.root`传入当前目录，执行后发现当前目录下自动生成了一个`repository`目录，这和Maven的默认下载目录`~/.m2/repository`的结构是完全一样的，只是它仅包含`spring-boot-hello-1.0-SNAPSHOT.jar`所需的运行期依赖项。

 注意：只有首次运行时会自动下载依赖项，再次运行时由于无需下载，所以启动速度会大大加快。如果删除了repository目录，再次运行时就会再次触发下载。

**预热**

把79KB大小的`spring-boot-hello-1.0-SNAPSHOT.jar`直接扔到服务器执行，上传过程就非常快。但是，第一次在服务器上运行`spring-boot-hello-1.0-SNAPSHOT.jar`时，仍需要从Maven中央仓库下载大量的jar包，所以，`spring-boot-thin-launcher`还提供了一个`dryrun`选项，专门用来下载依赖项而不执行实际代码：

```bash
java -Dthin.dryrun=true -Dthin.root=. -jar spring-boot-hello-1.0-SNAPSHOT.jar
```

执行上述代码会在当前目录创建`repository`目录，并下载所有依赖项，但并不会运行我们编写的`main()`方法。此过程称之为“预热”（warm up）。

如果服务器由于安全限制不允许从外网下载文件，那么可以在本地预热，然后把`spring-boot-hello-1.0-SNAPSHOT.jar`和`repository`目录上传到服务器。只要依赖项没有变化，后续改动只需要上传`spring-boot-hello-1.0-SNAPSHOT.jar`即可。

如果在maven中使用相对路径引入了自己的jar,使用 `java -jar .\xxx.jar --thin.root=.` 会报错。如：`<systemPath>${project.basedir}/src/main/resources/lib/spring-file-storage-0.4.0.jar</systemPath>`

`thin.root`根目录默认用的是本地的m2目录：`${user.home}/.m2`

把自己的jar直接复制到开发环境和部署环境的m2目录下

```xml
<systemPath>
  ${user.home}/.m2/repository/com/***/abc.jar
</systemPath>
```

这样就不会提示找不到依赖了。

[Spring Boot Thin Launcher官网](https://github.com/spring-projects-experimental/spring-boot-thin-launcher)

## Actuator

如果需要对应用程序的状态进行监控，

使用JMX需要把一些监控信息以MBean的形式暴露给JMX Server，而Spring Boot已经内置了一个监控功能叫Actuator。

使用Actuator非常简单，只需添加如下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

然后正常启动应用程序，Actuator会把它能收集到的所有信息都暴露给JMX。此外，Actuator还可以通过URL`/actuator/`挂载一些监控点，例如，输入`http://localhost:8080/actuator/health`，可以查看应用程序当前状态：

```json
{
    "status": "UP"
}
```

Actuator默认把**所有访问点暴露给JMX**，但处于安全原因，只有`health`和`info`会暴露给Web。Actuator提供的所有访问点均在官方文档列出，要暴露更多的访问点给Web，需要在`application.yml`中加上配置：

```yml
management:
  endpoints:
    web:
      exposure:
        include: info, health, beans, env, metrics
```

要特别注意暴露的URL的安全性，例如，`/actuator/env`可以获取当前机器的所有环境变量，不可暴露给公网。

## Profiles

Profile本身是Spring提供的功能，Profile表示一个环境的概念，如开发、测试和生产这3个环境：

- native
- test
- production

或者按git分支定义master、dev这些环境：

- master
- dev

在启动一个Spring应用程序的时候，可以传入一个或多个环境，例如：

```bash
-Dspring.profiles.active=test,master
```

大多数情况下，使用一个环境就足够了。

Spring Boot对Profiles的支持在于，可以在`application.yml`中为每个环境进行配置。下面是一个示例配置：

```yml
spring:
  application:
    name: ${APP_NAME:unnamed}
  datasource:
    url: jdbc:hsqldb:file:testdb
    username: sa
    password:
    dirver-class-name: org.hsqldb.jdbc.JDBCDriver
    hikari:
      auto-commit: false
      connection-timeout: 3000
      validation-timeout: 3000
      max-lifetime: 60000
      maximum-pool-size: 20
      minimum-idle: 1

pebble:
  suffix:
  cache: false

server:
  port: ${APP_PORT:8080}

---

spring:
  config:
    activate:
      on-profile: test

server:
  port: 8000

---

spring:
  config:
    activate:
      on-profile: production

server:
  port: 80

pebble:
  cache: true
```

分隔符`---`，最前面的配置是默认配置，不需要指定Profile，后面的每段配置都必须以`spring.config.activate.on-profile: xxx`开头，表示一个Profile。上述配置默认使用8080端口，但是在`test`环境下，使用`8000`端口，在`production`环境下，使用`80`端口，并且启用Pebble的缓存。

如果不指定任何Profile，直接启动应用程序，那么Profile实际上就是`default`，可以从Spring Boot启动日志看出：

```text
... INFO 54252 --- [  restartedMain] com.aotmd.Application                    : No active profile set, falling back to default profiles: default
```

上述日志显示未设置Profile，使用默认的Profile为`default`。

要以`test`环境启动，可输入如下命令：

```text
java -Dspring.profiles.active=test -jar springboot-profiles-1.0-SNAPSHOT.jar
...
INFO 58848 --- [  restartedMain] com.aotmd.Application                    : The following profiles are active: test
...
INFO 13510 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8000 (http) with context path ''
...
```

从日志看到活动的Profile是`test`，Tomcat的监听端口是`8000`。

通过Profile可以实现一套代码在不同环境启用不同的配置和功能。

在启动一个Spring应用程序的时候，可以传入一个或多个环境。如`-Dspring.profiles.active=test,master`，那最终会以哪个为准呢？答案是：先合并配置，如果有冲突，后面的覆盖前面的。

也可以多文件配置，将单文件中用---分割的文档块，分离到单个文件，主配置文件`application.yml`，环境配置文件`application-{profile}.yml`，因为已经通过文件名称设置了`spring.config.activate.on-profile: xxx`，因此不再需要重复写了，则`application-test.yml`的文件内容为：

```yml
server:
  port: 8000
```

通过主配置文件中`spring.profiles.active: test`进行激活环境

或者使用环境参数激活：

- VM options参数：`-Dspring.profiles.active=test`
- Program argument参数：`--spring.profiles.active=test`

在新版本中`spring.profiles: test`更换成了`spring.config.activate.on-profile: test`

假设需要一个存储服务，在本地开发时，直接使用文件存储即可，但是，在测试和生产环境，需要存储到云端，如何通过Profile实现该功能？首先，要定义存储接口`StorageService`：

```java
public interface StorageService {

    // 根据URI打开InputStream:
    InputStream openInputStream(String uri) throws IOException;

    // 根据扩展名+InputStream保存并返回URI:
    String store(String extName, InputStream input) throws IOException;
}
```

本地存储可通过`LocalStorageService`实现：

```java
@Component
@Profile("default")
public class LocalStorageService implements StorageService {
    @Value("${storage.local:/var/static}")
    String localStorageRootDir;

    final Logger logger = LoggerFactory.getLogger(getClass());

    private File localStorageRoot;

    @PostConstruct
    public void init() {
        logger.info("Intializing local storage with root dir: {}", this.localStorageRootDir);
        this.localStorageRoot = new File(this.localStorageRootDir);
    }

    @Override
    public InputStream openInputStream(String uri) throws IOException {
        File targetFile = new File(this.localStorageRoot, uri);
        return new BufferedInputStream(new FileInputStream(targetFile));
    }

    @Override
    public String store(String extName, InputStream input) throws IOException {
        String fileName = UUID.randomUUID().toString() + "." + extName;
        File targetFile = new File(this.localStorageRoot, fileName);
        try (OutputStream output = new BufferedOutputStream(new FileOutputStream(targetFile))) {
            input.transferTo(output);
        }
        return fileName;
    }
}
```

而云端存储可通过`CloudStorageService`实现：

```java
@Component
@Profile("!default")
public class CloudStorageService implements StorageService {
    @Value("${storage.cloud.bucket:}")
    String bucket;

    @Value("${storage.cloud.access-key:}")
    String accessKey;

    @Value("${storage.cloud.access-secret:}")
    String accessSecret;

    final Logger logger = LoggerFactory.getLogger(getClass());

    @PostConstruct
    public void init() {
        // TODO:
        logger.info("Initializing cloud storage...");
    }

    @Override
    public InputStream openInputStream(String uri) throws IOException {
        // TODO:
        throw new IOException("File not found: " + uri);
    }

    @Override
    public String store(String extName, InputStream input) throws IOException {
        // TODO:
        throw new IOException("Unable to access cloud storage.");
    }
}
```

`LocalStorageService`使用了条件装配`@Profile("default")`，即默认启用`LocalStorageService`，而`CloudStorageService`使用了条件装配`@Profile("!default")`，即非`default`环境时，自动启用`CloudStorageService`。这样，一套代码，就实现了不同环境启用不同的配置。

## Conditional

使用Profile能根据不同的Profile进行条件装配，但是Profile控制比较糙，如果想要精细控制，用Profile就很难实现。

Spring本身提供了条件装配`@Conditional`，但是要自己编写比较复杂的`Condition`来做判断，比较麻烦。Spring Boot则准备好了几个非常有用的条件：

- `@ConditionalOnProperty`：如果有指定的配置，条件生效；
- `@ConditionalOnBean`：如果有指定的Bean，条件生效；
- `@ConditionalOnMissingBean`：如果没有指定的Bean，条件生效；
- `@ConditionalOnMissingClass`：如果没有指定的Class，条件生效；
- `@ConditionalOnWebApplication`：在Web环境中条件生效；
- `@ConditionalOnExpression`：根据表达式判断条件是否生效。

以最常用的`@ConditionalOnProperty`为例，把上一节的`StorageService`改写如下。首先，定义配置`storage.type=xxx`，用来判断条件，默认为`local`：

```yml
storage:
  type: ${STORAGE_TYPE:local}
```

设定为`local`时，启用`LocalStorageService`：

```java
@Component
@ConditionalOnProperty(value = "storage.type", havingValue = "local", matchIfMissing = true)
public class LocalStorageService implements StorageService {
    ...
}
```

`LocalStorageService`的注解，当指定配置为`local`，或者配置不存在，均启用`LocalStorageService`。

设定为`aws`时，启用`AwsStorageService`：

```java
@Component
@ConditionalOnProperty(value = "storage.type", havingValue = "aws")
public class AwsStorageService implements StorageService {
    ...
}
```

设定为`aliyun`时，启用`AliyunStorageService`：

```java
@Component
@ConditionalOnProperty(value = "storage.type", havingValue = "aliyun")
public class AliyunStorageService implements StorageService {
    ...
}
```

------

`@ConditionalOnProperty`：当指定的配置属性存在且符合预期时，条件生效。

**常用属性:**

- `name`: 要检查的属性名称。
- `havingValue`: 属性值必须与此值匹配才生效。
- `matchIfMissing`: 如果属性不存在，是否匹配。默认值是 `false`。

**示例:**
```java
@Configuration
@ConditionalOnProperty(name = "feature.enabled", havingValue = "true", matchIfMissing = true)
public class FeatureConfig {
    // 配置内容
}
```

`@ConditionalOnBean`：当指定的Bean存在时，条件生效。

**常用属性:**

- `value` 或 `type`: 要检查的Bean类型。
- `name`: 要检查的Bean名称。

**示例:**

```java
@Configuration
@ConditionalOnBean(name = "myBean")
public class BeanDependentConfig {
    // 配置内容
}
```

`@ConditionalOnMissingBean`：当指定的Bean不存在时，条件生效。

**常用属性:**
- `value` 或 `type`: 要检查的Bean类型。
- `name`: 要检查的Bean名称。

**示例:**

```java
@Configuration
@ConditionalOnMissingBean(name = "myBean")
public class MissingBeanConfig {
    // 配置内容
}
```

`@ConditionalOnMissingClass`：当指定的类不存在时，条件生效。

**常用属性:**
- `value`: 要检查的类名。

**示例:**

```java
@Configuration
@ConditionalOnMissingClass("com.example.SomeClass")
public class MissingClassConfig {
    // 配置内容
}
```

`@ConditionalOnWebApplication`：在Web应用环境中，条件生效。

**常用属性:**
- `type`: Web应用的类型，可以是 `SERVLET` 或 `REACTIVE`。

**示例:**
```java
@Configuration
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
public class WebAppConfig {
    // 配置内容
}
```

`@ConditionalOnExpression`：根据SpEL表达式判断条件是否生效。

**常用属性:**
- `value`: SpEL表达式。

**示例:**
```java
@Configuration
@ConditionalOnExpression("'${env}'.equals('dev')")
public class DevEnvConfig {
    // 配置内容
}
```



## 加载配置文件

加载配置文件可以直接使用注解`@Value`，例如，我们定义了一个最大允许上传的文件大小配置：

```yml
storage:
  local:
    max-size: 102400
```

在某个`FileUploader`里，需要获取该配置，可使用`@Value`注入：

```java
@Component
public class FileUploader {
    @Value("${storage.local.max-size:102400}")
    int maxSize;

    ...
}
```

在另一个`UploadFilter`中，因为要检查文件的MD5，同时也要检查输入流的大小，因此，也需要该配置：

```java
@Component
public class UploadFilter implements Filter {
    @Value("${storage.local.max-size:100000}")
    int maxSize;

    ...
}
```

多次引用同一个`@Value`不但麻烦，而且`@Value`使用字符串，缺少编译器检查，容易造成多处引用不一致（例如，`UploadFilter`把缺省值误写为`100000`）。

为了更好地管理配置，Spring Boot允许创建一个Bean，持有一组配置，并由Spring Boot自动注入。

假设在`application.yml`中添加了如下配置：

```yml
storage:
  local:
    # 文件存储根目录:
    root-dir: ${STORAGE_LOCAL_ROOT:/var/storage}
    # 最大文件大小，默认100K:
    max-size: ${STORAGE_LOCAL_MAX_SIZE:102400}
    # 是否允许空文件:
    allow-empty: false
    # 允许的文件类型:
    allow-types: jpg, png, gif
```

可以首先定义一个Java Bean，持有该组配置：

```java
public class StorageConfiguration {

    private String rootDir;//注意使用驼峰替代横线
    private int maxSize;
    private boolean allowEmpty;
    private List<String> allowTypes;

    // TODO: getters and setters
}
```

保证Java Bean的属性名称与配置一致即可。然后，添加两个注解：

```java
@Configuration
@ConfigurationProperties("storage.local")
public class StorageConfiguration {
    ...
}
```

`@ConfigurationProperties("storage.local")`表示将从配置项`storage.local`读取该项的所有子项配置，并且，`@Configuration`表示`StorageConfiguration`也是一个Spring管理的Bean，可直接注入到其他Bean中：

```java
@Component
public class StorageService {
    final Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    StorageConfiguration storageConfig;

    @PostConstruct
    public void init() {
        logger.info("Load configuration: root-dir = {}", storageConfig.getRootDir());
        logger.info("Load configuration: max-size = {}", storageConfig.getMaxSize());
        logger.info("Load configuration: allowed-types = {}", storageConfig.getAllowTypes());
    }
}
```

这样一来，引入`storage.local`的相关配置就很容易了，因为只需要注入`StorageConfiguration`这个Bean，这样可以由编译器检查类型，无需编写重复的`@Value`注解。

------

如果你的配置项有嵌套结构，可以在Java Bean中定义对应的嵌套类。下面是一个示例，其中包括嵌套配置项的处理。

首先，你的`application.yml`配置文件内容如下：

```yml
storage:
  local:
    root-dir: ${STORAGE_LOCAL_ROOT:/var/storage}
    max-size: ${STORAGE_LOCAL_MAX_SIZE:102400}
    allow-empty: false
    allow-types: jpg, png, gif
  remote:
    url: ${STORAGE_REMOTE_URL:http://example.com}
    timeout: ${STORAGE_REMOTE_TIMEOUT:5000}
```

接下来，你需要定义对应的Java Bean类来表示这个配置。可以将`local`和`remote`配置项分别封装到嵌套的类中。

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import java.util.List;

@Configuration
@ConfigurationProperties("storage")
public class StorageConfiguration {

    private Local local;
    private Remote remote;

    // Getters and setters for `local` and `remote`

    public static class Local {
        private String rootDir;
        private int maxSize;
        private boolean allowEmpty;
        private List<String> allowTypes;

        // Getters and setters for `rootDir`, `maxSize`, `allowEmpty`, and `allowTypes`
    }

    public static class Remote {
        private String url;
        private int timeout;

        // Getters and setters for `url` and `timeout`
    }

    // Getters and setters for `local` and `remote`
    public Local getLocal() {
        return local;
    }

    public void setLocal(Local local) {
        this.local = local;
    }

    public Remote getRemote() {
        return remote;
    }

    public void setRemote(Remote remote) {
        this.remote = remote;
    }
}
```

在上述代码中：
- `@Configuration`注解表示`StorageConfiguration`是一个Spring管理的Bean。
- `@ConfigurationProperties("storage")`注解表示从`storage`前缀开始读取配置项。
- `Local`和`Remote`是两个静态嵌套类，用于表示嵌套的配置项。

通过这种方式，Spring Boot会自动将配置文件中的值绑定到相应的Java Bean中。

你还需要为嵌套类定义getter和setter方法，例如：

```java
public static class Local {
    private String rootDir;
    private int maxSize;
    private boolean allowEmpty;
    private List<String> allowTypes;

    // Getters and setters
    public String getRootDir() {
        return rootDir;
    }

    public void setRootDir(String rootDir) {
        this.rootDir = rootDir;
    }

    public int getMaxSize() {
        return maxSize;
    }

    public void setMaxSize(int maxSize) {
        this.maxSize = maxSize;
    }

    public boolean isAllowEmpty() {
        return allowEmpty;
    }

    public void setAllowEmpty(boolean allowEmpty) {
        this.allowEmpty = allowEmpty;
    }

    public List<String> getAllowTypes() {
        return allowTypes;
    }

    public void setAllowTypes(List<String> allowTypes) {
        this.allowTypes = allowTypes;
    }
}

public static class Remote {
    private String url;
    private int timeout;

    // Getters and setters
    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public int getTimeout() {
        return timeout;
    }

    public void setTimeout(int timeout) {
        this.timeout = timeout;
    }
}
```

这样，配置文件中的值就会自动绑定到对应的Java Bean中，并且可以在你的应用程序中通过注入`StorageConfiguration` Bean来访问这些配置值。

## 禁用自动配置

Spring Boot大量使用自动配置和默认配置，极大地减少了代码，通常只需要加上几个注解，并按照默认规则设定一下必要的配置即可。例如，配置JDBC，默认情况下，只需要配置一个`spring.datasource`：

```yml
spring:
  datasource:
    url: jdbc:hsqldb:mem:testdb
    username: sa
    password:
    dirver-class-name: org.hsqldb.jdbc.JDBCDriver
```

Spring Boot就会自动创建出`DataSource`、`JdbcTemplate`、`DataSourceTransactionManager`，非常方便。

但是有时候又必须要禁用某些自动配置。例如，系统有主从两个数据库，而Spring Boot的自动配置只能配一个。

这个时候，针对`DataSource`相关的自动配置，就必须关掉。需要用`exclude`指定需要关掉的自动配置：

```java
@SpringBootApplication
// 启动自动配置，但排除指定的自动配置:
@EnableAutoConfiguration(exclude = DataSourceAutoConfiguration.class)
public class Application {
    ...
}
```

或者：

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class Application {
    ...
}
```

现在，Spring Boot不再自动创建`DataSource`、`JdbcTemplate`和`DataSourceTransactionManager`了。

还可以通过配置文件禁用：

```yaml
spring:
  autoconfigure:
    exclude: 
      - com.example.autoconfig.MyAutoConfiguration
```

## 使用自动配置

`spring.factories `这个文件的作用是配置 Spring Boot 的自动配置机制，通过指定自定义的配置类，使得 Spring Boot 能够在应用启动时自动加载和配置该类中的 beans 和相关设置。

具体作用包括：

1. **自定义配置**：允许开发者定义应用启动时需要的自定义 bean 和配置信息，确保应用具备所需的功能。

2. **简化配置**：通过自动配置，减少手动配置的复杂性，提供开箱即用的功能，提升开发效率。

3. **模块化**：可以将应用的不同功能模块分开，方便管理和维护。例如，将与事件传输相关的配置放在 `TransmitterConfig` 中。

4. **与 Spring Boot 生态兼容**：可以方便地与其他 Spring Boot 自动配置模块结合使用，提供一致的开发体验。

通常，在应用中使用自动配置能够使得设置更为简洁且易于理解。

------

**在自己的模块使用**

假设启动类为：`com.test.TestApplication`，现在，新建了一个类：`com.test2.Demo`:

```java
package com.test2;
public class Demo {}
```

想要将这个类注入IOC容器有以下方法：

- 在启动类上打上注释`@Import(Demo.class)` ，然后可以为`Demo`打上注释`@Configuration`，这是可选的。

- 为`Demo`打上注释`@Component`或其他同类注释，然后在主启动类上指定扫描路径`@ComponentScan(basePackages = {"com.test", "com.test2"})`。

- 使用XMl配置。`<beans><bean id="demo" class="com.test2.Demo"/></beans>`

- 在resources下面创建`META-INF/spring.factories`文件，内容为：

  ```text
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   com.test2.Demo
  ```

------

使用 Spring Boot 的自动配置机制，如果其他项目引入了你的包，它会根据 `spring.factories` 文件中的配置自动加载你定义的配置类。这是 Spring Boot 的一个重要特性，使得你的模块可以开箱即用。

当主启动类上使用了 `@EnableAutoConfiguration` 时，Spring Boot 会在启动过程中读取依赖包中的 `spring.factories` 文件，以确定需要自动装配哪些配置类。主启动类通常会使用 `@SpringBootApplication` 注解，其中包含了 `@EnableAutoConfiguration`。

- 在应用启动时，Spring Boot 会扫描类路径下的所有 JAR 包，查找 `META-INF/spring.factories` 文件。这个文件中列出了所有需要自动配置的类。
- Spring Boot 会根据 `spring.factories` 文件中的配置，尝试加载列出的自动配置类。这些类通常使用条件注解来判断是否应该被实例化和配置。
- 自动配置类中的条件注解（如 `@ConditionalOnClass`、`@ConditionalOnMissingBean` 等）会根据当前环境的条件进行评估，决定是否创建相应的 Beans。



**关系与优先级**

在 Spring Boot 中，依赖包内的 `application.yml` 和主模块的 `application.yml` 之间的关系主要体现在配置的优先级和覆盖规则上。

Spring Boot 会按照一定的顺序加载配置文件，通常包括：

- 主模块的 `src/main/resources/application.yml`
- 依赖包中的 `META-INF/application.yml`
- 依赖包中的 `application.yml`

**优先级**：

- 主模块中的 `application.yml` 通常具有更高的优先级，意味着如果同一个属性在主模块的配置文件和依赖包的配置文件中都有定义，主模块中的值将覆盖依赖包中的值。
- 这使得主模块能够定制或修改依赖包提供的默认配置。



**注解的使用**

可以在依赖包中使用 `@SpringBootApplication`。在创建自定义 Spring Boot Starter 时，可以在 Starter 内部使用 `@SpringBootApplication`，但通常建议将其放在主应用中，而不是依赖包中。

**注意事项**

- **包扫描**：`@SpringBootApplication` 注解默认会扫描该注解所在包及其子包。如果在依赖包中使用，确保该包结构与主应用的包结构相符，以便能够正确加载所有组件。
- **启动类冲突**：如果多个依赖包中都有 `@SpringBootApplication` 注解的启动类，可能会导致启动冲突。在这种情况下，通常建议只有一个启动类。
- **避免不必要的复杂性**：如果你的依赖包只是提供一些组件或服务，建议不在依赖包中使用 `@SpringBootApplication`，而是在主应用中使用，并通过配置类或其他方式来加载依赖包中的组件。

基本上你可以在依赖包中使用所有的 Spring Boot 注解，一般情况下，只会使用 `@ComponentScan`指定包扫描注解。



------

要创建一个自定义的自动配置类，可以按照以下步骤进行：

**创建配置类**：首先，定义一个配置类，并添加 `@Configuration` 注解。这将告知 Spring Boot 这是一个配置类。

```java
package com.example.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyCustomConfig {
    
    @Bean
    public MyService myService() {
        return new MyService();
    }

    // 可以添加更多的 bean
}
```

**定义自动配置类**：创建一个自动配置类，通常与其他配置文件放在同一包下，或放在 `spring.factories` 文件中指定的路径中。

```java
package com.example.autoconfig;

import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyAutoConfiguration {
    // 可选择定义 @Bean 方法
}
```

**创建 `spring.factories` 文件**：在 `src/main/resources/META-INF` 目录下创建 `spring.factories` 文件，指定自动配置类。

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.autoconfig.MyAutoConfiguration
```

------

如果你有多个配置类，并希望它们都能作为自动配置的一部分，可以按照以下方式组织和实现：

**创建多个配置类**：为每个配置功能创建独立的配置类，并使用 `@Configuration` 注解标记。

```java
package com.example.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyServiceConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}

@Configuration
public class AnotherServiceConfig {
    @Bean
    public AnotherService anotherService() {
        return new AnotherService();
    }
}
```

**自动配置类**：创建一个主的自动配置类，将所有其他配置类导入到该类中。

```java
package com.example.autoconfig;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@Import({MyServiceConfig.class, AnotherServiceConfig.class})
public class MyAutoConfiguration {
    // 可以选择定义更多的 bean
}
```

**`spring.factories` 文件**：确保在 `spring.factories` 文件中仅指向主自动配置类。

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.autoconfig.MyAutoConfiguration
```

**条件加载（可选）**：如果需要根据特定条件加载某些配置类，可以在配置类中使用条件注解，如 `@ConditionalOnProperty`、`@ConditionalOnClass` 等。

```java
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;

@Configuration
@ConditionalOnProperty(name = "my.feature.enabled", havingValue = "true")
public class MyFeatureConfig {
    @Bean
    public MyFeature myFeature() {
        return new MyFeature();
    }
}
```

------

**使用多个启动配置**

如果你想在 `spring.factories` 文件中定义多个自动配置类，可以在同一行中用逗号分隔多个类名，或者在新的一行中列出它们。以下是几种常见的写法：

方法一：用逗号分隔

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.example.autoconfig.FirstAutoConfiguration,\
  com.example.autoconfig.SecondAutoConfiguration
```

方法二：逐行列出

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.example.autoconfig.FirstAutoConfiguration
com.example.autoconfig.SecondAutoConfiguration
```

自动配置的顺序可能会影响行为，确保在定义时考虑到各个配置类之间的依赖关系。

## 主从数据库

要实现主从数据库支持，首先需要把主从数据库配置写到`application.yml`中，仍然按照Spring Boot默认的格式写，但`datasource`改为`datasource-master`和`datasource-slave`：

```yml
spring:
  datasource-master:
    url: jdbc:hsqldb:file:testdb
    username: sa
    password:
    dirver-class-name: org.hsqldb.jdbc.JDBCDriver
  datasource-slave:
    url: jdbc:hsqldb:file:testdb
    username: sa
    password:
    dirver-class-name: org.hsqldb.jdbc.JDBCDriver
```

两个数据库实际上是同一个库。如果使用MySQL，可以创建一个只读用户，作为`datasource-slave`的用户来模拟一个从库。

分别创建两个HikariCP的`DataSource`：

```java
public class MasterDataSourceConfiguration {
    @Bean("masterDataSourceProperties")
    @ConfigurationProperties("spring.datasource-master")
    DataSourceProperties dataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean("masterDataSource")
    DataSource dataSource(@Autowired @Qualifier("masterDataSourceProperties") DataSourceProperties props) {
        return props.initializeDataSourceBuilder().build();
    }
}

public class SlaveDataSourceConfiguration {
    @Bean("slaveDataSourceProperties")
    @ConfigurationProperties("spring.datasource-slave")
    DataSourceProperties dataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean("slaveDataSource")
    DataSource dataSource(@Autowired @Qualifier("slaveDataSourceProperties") DataSourceProperties props) {
        return props.initializeDataSourceBuilder().build();
    }
}
```

注意到上述class并未添加`@Configuration`和`@Component`，要使之生效，可以使用`@Import`导入：

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@Import({ MasterDataSourceConfiguration.class, SlaveDataSourceConfiguration.class})
public class Application {
    ...
}
```

上述两个`DataSource`的Bean名称分别为`masterDataSource`和`slaveDataSource`，我们还需要一个最终的`@Primary`标注的`DataSource`，它采用Spring提供的`AbstractRoutingDataSource`，代码实现如下：

```java
public class RoutingDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        // 从ThreadLocal中取出key:
        return RoutingDataSourceContext.getDataSourceRoutingKey();
    }
}
class RoutingDataSourceContext implements AutoCloseable {
    static final ThreadLocal<String> ctx = new ThreadLocal<>();
    public static void setDataSourceRoutingKey(String str) {
        ctx.set(str);
    }
    public static String getDataSourceRoutingKey() {
        return ctx.get();
    }
    @Override
    public void close() throws Exception {
        ctx.remove();
    }
}
```

`RoutingDataSource`本身并不是真正的`DataSource`，它通过Map关联一组`DataSource`，下面的代码创建了包含两个`DataSource`的`RoutingDataSource`，关联的key分别为`masterDataSource`和`slaveDataSource`：

```java
@Configuration
public class RoutingDataSourceConfiguration {
    @Primary
    @Bean
    DataSource dataSource(
            @Autowired @Qualifier("masterDataSource") DataSource masterDataSource,
            @Autowired @Qualifier("slaveDataSource") DataSource slaveDataSource) {
        var ds = new RoutingDataSource();
        // 关联两个DataSource:
        ds.setTargetDataSources(Map.of(
                "masterDataSource", masterDataSource,
                "slaveDataSource", slaveDataSource));
        // 默认使用masterDataSource:
        ds.setDefaultTargetDataSource(masterDataSource);
        return ds;
    }

    @Bean
    JdbcTemplate jdbcTemplate(@Autowired DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean
    DataSourceTransactionManager dataSourceTransactionManager(@Autowired DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

仍然需要自己创建`JdbcTemplate`和`PlatformTransactionManager`，注入的是标记为`@Primary`的`RoutingDataSource`。

这样，通过如下的代码就可以切换`RoutingDataSource`底层使用的真正的`DataSource`：

```java
RoutingDataSourceContext.setDataSourceRoutingKey("slaveDataSource");
jdbcTemplate.query(...);
```

只不过写代码切换DataSource即麻烦又容易出错，更好的方式是通过注解配合AOP实现自动切换，这样，客户端代码实现如下：

```java
@Controller
public class UserController {
	@RoutingWithSlave // <-- 指示在此方法中使用slave数据库
	@GetMapping("/profile")
	public ModelAndView profile(HttpSession session) {
        ...
    }
}
```

自定义的注解 `@RoutingWithSlave` 的作用只是调用 `RoutingDataSourceContext` 的构造方法向 `ThreadLocal` 储存了字符串 `slaveDataSource`。

```java
@Retention(RUNTIME)
@Target(METHOD)
@interface RoutingWithSlave{
}
@Aspect
@Component
public class RoutingAspect {
    @Around("@annotation(routingWithSlave)")
    public Object routingWithDataSource(ProceedingJoinPoint joinPoint, RoutingWithSlave routingWithSlave)
            throws Throwable {
        RoutingDataSourceContext.setDataSourceRoutingKey("slaveDataSource");
        return joinPoint.proceed();
    }
}
```

注意这里使用了形参名称，注意编译时添加`-parameters`参数，保留形参名称，参见[设计MVC框架](/2021/09/14/JavaWeb/#MVC)。`-parameters`的作用是在编译后的类文件中**保留方法参数的名称**。默认情况下，Java编译器在编译过程中会丢弃方法参数的名称，而只保留参数的顺序。

或者使用完全限定名：

```java
@Around("@annotation(com.aotmd.RoutingWithSlave)")
public Object routingWithDataSource(ProceedingJoinPoint joinPoint)
        throws Throwable {
    RoutingDataSourceContext.setDataSourceRoutingKey("slaveDataSource");
    return joinPoint.proceed();
}
```

如果想要确认是否真的切换了`DataSource`，可以覆写`determineTargetDataSource()`方法并打印出`DataSource`的名称：

```java
public class RoutingDataSource extends AbstractRoutingDataSource {
    final Logger logger = LoggerFactory.getLogger(getClass());
    @Override
    protected Object determineCurrentLookupKey() {
        // 从ThreadLocal中取出key:
        return RoutingDataSourceContext.getDataSourceRoutingKey();
    }
    @Override
    protected DataSource determineTargetDataSource() {
        DataSource ds = super.determineTargetDataSource();
        logger.info("determin target datasource: {}", ds);
        return ds;
    }
}
```

可以用一个图表示创建的DataSource以及相关Bean的关系：

```ascii
┌────────────────────┐       ┌──────────────────┐
│@Primary            │<──────│   JdbcTemplate   │
│RoutingDataSource   │       └──────────────────┘
│ ┌────────────────┐ │       ┌──────────────────┐
│ │MasterDataSource│ │<──────│DataSource        │
│ └────────────────┘ │       │TransactionManager│
│ ┌────────────────┐ │       └──────────────────┘
│ │SlaveDataSource │ │
│ └────────────────┘ │
└────────────────────┘
```

`DataSourceTransactionManager`和`JdbcTemplate`引用的都是`RoutingDataSource`，所以，这种设计的一个限制就是：在一个请求中，一旦切换了内部数据源，在同一个事务中，不能再切到另一个，否则，`DataSourceTransactionManager`和`JdbcTemplate`操作的就不是同一个数据库连接。

完整代码：

`RoutingDataSource.java`

```java
package com.aotmd;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
import org.springframework.stereotype.Component;
import javax.sql.DataSource;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import java.util.Map;

import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

class MasterDataSourceConfiguration {
    @Bean("masterDataSourceProperties")
    @ConfigurationProperties("spring.datasource-master")
    DataSourceProperties dataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean("masterDataSource")
    DataSource dataSource(@Autowired @Qualifier("masterDataSourceProperties") DataSourceProperties props) {
        return props.initializeDataSourceBuilder().build();
    }
}

class SlaveDataSourceConfiguration {
    @Bean("slaveDataSourceProperties")
    @ConfigurationProperties("spring.datasource-slave")
    DataSourceProperties dataSourceProperties() {
        return new DataSourceProperties();
    }

    @Bean("slaveDataSource")
    DataSource dataSource(@Autowired @Qualifier("slaveDataSourceProperties") DataSourceProperties props) {
        return props.initializeDataSourceBuilder().build();
    }
}

public class RoutingDataSource extends AbstractRoutingDataSource {
    final Logger logger = LoggerFactory.getLogger(getClass());
    @Override
    protected Object determineCurrentLookupKey() {
        // 从ThreadLocal中取出key:
        return RoutingDataSourceContext.getDataSourceRoutingKey();
    }
    @Override
    protected DataSource determineTargetDataSource() {
        DataSource ds = super.determineTargetDataSource();
        logger.info("determin target datasource: {}", ds);
        return ds;
    }
}
class RoutingDataSourceContext implements AutoCloseable {
    static final ThreadLocal<String> ctx = new ThreadLocal<>();
    public static void setDataSourceRoutingKey(String str) {
        ctx.set(str);
    }
    public static String getDataSourceRoutingKey() {
        return ctx.get();
    }
    @Override
    public void close() throws Exception {
        ctx.remove();
    }
}
@Configuration
class RoutingDataSourceConfiguration {
    @Primary
    @Bean
    DataSource dataSource(
            @Autowired @Qualifier("masterDataSource") DataSource masterDataSource,
            @Autowired @Qualifier("slaveDataSource") DataSource slaveDataSource) {
        var ds = new RoutingDataSource();
        // 关联两个DataSource:
        ds.setTargetDataSources(Map.of(
                "masterDataSource", masterDataSource,
                "slaveDataSource", slaveDataSource));
        // 默认使用masterDataSource:
        ds.setDefaultTargetDataSource(masterDataSource);
        return ds;
    }

    @Bean
    JdbcTemplate jdbcTemplate(@Autowired DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean
    DataSourceTransactionManager dataSourceTransactionManager(@Autowired DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
@Retention(RUNTIME)
@Target(METHOD)
@interface RoutingWithSlave{

}
@Aspect
@Component
class RoutingAspect {
    @Around("@annotation(routingWithSlave)")
    public Object routingWithDataSource(ProceedingJoinPoint joinPoint, RoutingWithSlave routingWithSlave)
            throws Throwable {
        RoutingDataSourceContext.setDataSourceRoutingKey("slaveDataSource");
        return joinPoint.proceed();
    }
}
```

`Application.java`:

```java
@MapperScan("com.aotmd")
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@Import({ MasterDataSourceConfiguration.class, SlaveDataSourceConfiguration.class})
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    @Bean
    WebMvcConfigurer createWebMvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addResourceHandlers(ResourceHandlerRegistry registry) {
                // 映射路径`/static/`到classpath路径:
                registry.addResourceHandler("/static/**")
                        .addResourceLocations("classpath:/static/");
            }
        };
    }
}
```

访问数据库可以用之前的。



## Filter

在Spring Boot中，添加一个`Filter`更简单了，可以做到零配置。

Spring Boot会自动扫描所有的`FilterRegistrationBean`类型的Bean，然后，将它们返回的`Filter`自动注册到Servlet容器中，无需任何配置。

以`AuthFilter`为例，首先编写一个`AuthFilterRegistrationBean`，它继承自`FilterRegistrationBean`：

```java
@Component
public class AuthFilterRegistrationBean extends FilterRegistrationBean<Filter> {
    @Autowired
    UserService userService;

    @Override
    public Filter getFilter() {
        setOrder(10);
        return new AuthFilter();
    }

    class AuthFilter implements Filter {
        ...
    }
}
```

`FilterRegistrationBean`本身不是`Filter`，它实际上是`Filter`的工厂。Spring Boot会调用`getFilter()`，把返回的`Filter`注册到Servlet容器中。因为可以在`FilterRegistrationBean`中注入需要的资源，然后，在返回的`AuthFilter`中，这个内部类可以引用外部类的所有字段，自然也包括注入的`UserService`，所以，整个过程完全基于Spring的IoC容器完成。

`AuthFilterRegistrationBean`使用了`setOrder(10)`，因为Spring Boot支持给多个`Filter`排序，**数字小的在前面**，所以，多个`Filter`的顺序是可以固定的。

再编写一个`ApiFilter`，专门过滤`/api/*`这样的URL。首先编写一个`ApiFilterRegistrationBean`：

```java
@Component
public class ApiFilterRegistrationBean extends FilterRegistrationBean<Filter> {
    @PostConstruct
    public void init() {
        setOrder(20);
        setFilter(new ApiFilter());
        setUrlPatterns(List.of("/api/*"));
    }

    class ApiFilter implements Filter {
        ...
    }
}
```

这个`ApiFilterRegistrationBean`和`AuthFilterRegistrationBean`又有所不同。因为要过滤URL，而不是针对所有URL生效，因此，在`@PostConstruct`方法中，通过`setFilter()`设置一个`Filter`实例后，再调用`setUrlPatterns()`传入要过滤的URL列表。

------

`FilterRegistrationBean` 是 Spring Framework 中的一个类，用于注册和配置 Servlet 过滤器。它提供了一些常用的方法来控制过滤器的注册和行为。以下是 `FilterRegistrationBean` 的一些常用方法及其简要说明：

1. **`setFilter(Filter filter)`**：设置过滤器实例。
```java
FilterRegistrationBean<Filter> registrationBean = new FilterRegistrationBean<>();
registrationBean.setFilter(new MyFilter());
```

2. **`setUrlPatterns(Collection<String> urlPatterns)`**：设置过滤器的 URL 模式。
```java
registrationBean.setUrlPatterns(Arrays.asList("/api/*"));
```

3. **`addUrlPatterns(String... urlPatterns)`**：添加过滤器的 URL 模式。
```java
registrationBean.addUrlPatterns("/api/*");
```

4. **`setServletNames(Collection<String> servletNames)`**：设置过滤器要应用的 Servlet 名称。
```java
registrationBean.setServletNames(Arrays.asList("myServlet"));
```

5. **`addServletNames(String... servletNames)`**：添加过滤器要应用的 Servlet 名称。
```java
registrationBean.addServletNames("myServlet");
```

6. **`setOrder(int order)`**：设置过滤器的执行顺序，值越小优先级越高。
```java
registrationBean.setOrder(1);
```

7. **`setDispatcherTypes(EnumSet<DispatcherType> dispatcherTypes)`**：设置过滤器的分发类型（如 `REQUEST`、`FORWARD`、`INCLUDE` 等）。
```java
registrationBean.setDispatcherTypes(EnumSet.of(DispatcherType.REQUEST, DispatcherType.FORWARD));
```

8. **`setAsyncSupported(boolean isAsyncSupported)`**：设置过滤器是否支持异步操作。
```java
registrationBean.setAsyncSupported(true);
```

9. **`setName(String name)`**：设置过滤器的名称。
```java
registrationBean.setName("myFilter");
```

10. **`setInitParameters(Map<String, String> initParameters)`**：设置过滤器的初始化参数。
```java
Map<String, String> initParams = new HashMap<>();
initParams.put("param1", "value1");
registrationBean.setInitParameters(initParams);
```


以下是一个完整的示例，展示如何使用 `FilterRegistrationBean` 来注册和配置一个过滤器：

```java
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Arrays;
import java.util.EnumSet;

@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<Filter> myFilter() {
        FilterRegistrationBean<Filter> registrationBean = new FilterRegistrationBean<>();
        
        Filter myFilter = new Filter() {
            @Override
            public void init(FilterConfig filterConfig) throws ServletException {
                // 初始化代码
            }

            @Override
            public void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
                    throws IOException, ServletException {
                // 过滤器逻辑
                chain.doFilter(request, response);
            }

            @Override
            public void destroy() {
                // 销毁代码
            }
        };
        
        registrationBean.setFilter(myFilter);
        registrationBean.setUrlPatterns(Arrays.asList("/api/*"));
        registrationBean.setOrder(1);
        registrationBean.setDispatcherTypes(EnumSet.of(DispatcherType.REQUEST));
        registrationBean.setAsyncSupported(true);
        registrationBean.setName("myFilter");
        registrationBean.setInitParameters(Map.of("param1", "value1"));
        
        return registrationBean;
    }
}
```

这个示例中，`FilterRegistrationBean` 被用来注册一个简单的过滤器，并设置了一些常见的配置选项。

## 在项目启动后运行一段代码

在Spring Boot项目启动后运行一段代码，你可以使用`CommandLineRunner`或`ApplicationRunner`接口。两者的用法类似，都可以在应用程序启动完成后执行特定代码。

**使用 `CommandLineRunner`**

`CommandLineRunner`接口提供了一个`run`方法，在Spring Boot启动后立即执行。你可以创建一个实现此接口的类。

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class MyCommandLineRunner implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        // 在应用程序启动后运行的代码
        System.out.println("Spring Boot 应用程序已启动，执行自定义代码...");
    }
}
```

**使用 `ApplicationRunner`**

`ApplicationRunner`接口与`CommandLineRunner`类似，但它接受一个`ApplicationArguments`对象，可以更方便地访问应用程序参数。

```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

@Component
public class MyApplicationRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // 在应用程序启动后运行的代码
        System.out.println("Spring Boot 应用程序已启动，执行自定义代码...");
    }
}
```

如果有多个`CommandLineRunner`或`ApplicationRunner`实现，并且希望它们按特定顺序执行，你可以实现`Ordered`接口或使用`@Order`注解。

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Order(1)
public class MyCommandLineRunner1 implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        System.out.println("第一个运行的 CommandLineRunner");
    }
}

@Component
@Order(2)
public class MyCommandLineRunner2 implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        System.out.println("第二个运行的 CommandLineRunner");
    }
}
```

这样，`MyCommandLineRunner1`将会在`MyCommandLineRunner2`之前执行。

在Spring Boot应用启动后运行代码的两种常用方法是实现`CommandLineRunner`或`ApplicationRunner`接口。选择其中之一，并将所需逻辑放入`run`方法中。通过实现`Ordered`接口或使用`@Order`注解，你可以控制多个运行器的执行顺序。

## 集成Open API

[Open API](https://www.openapis.org/)是一个标准，它的主要作用是**描述REST API**，既可以作为文档给开发者阅读，又可以让机器根据这个文档自动生成客户端代码等。

在Spring Boot应用中，假设编写了一堆REST API，只需要在`pom.xml`中加入以下依赖：

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.0.0</version>
</dependency>
```

直接启动应用，打开浏览器输入`http://localhost:8080/swagger-ui.html`

立刻可以看到自动生成的API文档，点击某个API还可以交互，即输入API参数，点“Try it out”按钮，获得运行结果。

因为引入`springdoc-openapi-ui`这个依赖后，它自动引入Swagger UI用来创建API文档。可以给API加入一些描述信息，例如：

```java
@RestController
@RequestMapping("/api")
public class ApiController {
    ...
    @Operation(summary = "Get specific user object by it's id.")
	@GetMapping("/users/{id}")
	public User user(@Parameter(description = "id of the user.") @PathVariable("id") long id) {
		return userService.getUserById(id);
	}
    ...
}
```

`@Operation`可以对API进行描述，`@Parameter`可以对参数进行描述，它们的目的是用于生成API文档的描述信息。

大多数情况下，不需要任何配置，就直接得到了一个运行时动态生成的可交互的API文档，该API文档总是和代码保持同步，大大简化了文档的编写工作。

要自定义文档的样式、控制某些API显示等，请参考[springdoc文档](https://springdoc.org/)。

**配置反向代理**

如果在服务器上，用户访问的域名是`https://example.com`，但内部是通过类似Nginx这样的反向代理访问实际的Spring Boot应用，比如`http://localhost:8080`，这个时候，在页面`https://example.com/swagger-ui.html`上，显示的URL仍然是`http://localhost:8080`，这样一来，就无法直接在页面执行API，非常不方便。

这是因为Spring Boot内置的Tomcat默认获取的服务器名称是`localhost`，端口是实际监听端口，而不是对外暴露的域名和`80`或`443`端口。要让Tomcat获取到对外暴露的域名等信息，必须在Nginx配置中传入必要的HTTP Header，常用的配置如下：

```ruby
# Nginx配置
server {
    ...
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    ...
}
```

然后，在Spring Boot的`application.yml`中，加入如下配置：

```yaml
server:
  # 实际监听端口:
  port: 8080
  # 从反向代理读取相关的HTTP Header:
  forward-headers-strategy: native
```

重启Spring Boot应用，即可在Swagger中显示正确的URL。

**使用knife4j**

在日常开发中，写接口文档是我们必不可少的，而Knife4j就是一个接口文档工具，可以看作是Swagger的升级版，但是界面比Swagger更好看，功能更丰富

早期，swagger-boostrap-ui是1.x版本，如今swagger-bootsrap-ui到2.x，同时也更改名字Knife4j，适用于单体和微服务项目。

[Knife4j官方网站](https://doc.xiaominfo.com/)。

导入依赖：

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>2.0.1</version>
</dependency>
```

创建配置类：

```java
package com.aotmd;

import io.swagger.annotations.ApiOperation;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.ApiKey;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

import java.util.List;

import static com.google.common.collect.Lists.newArrayList;



/**
 * Swagger2配置信息
 * 这里分了两组显示
 * 第一组是api，当作用户端接口
 * 第二组是admin，当作后台管理接口
 * 也可以根据实际情况来减少或者增加组
 */

@Configuration
@EnableSwagger2
public class Swagger2Config{

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                //加了ApiOperation注解的类，生成接口文档
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                //包下的类，生成接口文档
                //.apis(RequestHandlerSelectors.basePackage("com.aotmd"))
                .paths(PathSelectors.any())
                .build()
                .directModelSubstitute(java.util.Date.class, String.class)
                .securitySchemes(security());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("`文档`")
                .description("文档")
                .termsOfServiceUrl("*")
                .version("2.0.0")
                .build();
    }

    private List<ApiKey> security() {
        return newArrayList(
                new ApiKey("token", "token", "header")
        );
    }
}
```

实体类：

```java
@ApiModel("用户实体类")
@Data
@Builder
public class SwaggerUser {
    @ApiModelProperty("用户Id")
    private Long id;
    @ApiModelProperty("用户名称")
    private String name;
}
```

控制器：

```java
@Api(tags = "用户端控制器")
@RestController
@RequestMapping("/api")
public class ApiController {

    @ApiOperation(value = "获取数据")
    @GetMapping("/test")
    public List<SwaggerUser> test(@ApiParam(name = "id",value = "用户Id") Long id,
                                  @ApiParam(name = "name",value = "用户名称") String name){
        List<SwaggerUser> list = new ArrayList<>();
        list.add(SwaggerUser.builder().id(id).name(name).build());
        return list;
    }
}
```

运行后访问`http://127.0.0.1:8080/doc.html`即可。

## 集成Redis

在Spring Boot中，要访问Redis，可以直接引入`spring-boot-starter-data-redis`依赖，它实际上是Spring Data的一个子项目——Spring Data Redis，主要用到了这几个组件：

- Lettuce：一个基于Netty的高性能Redis客户端；
- RedisTemplate：一个类似于JdbcTemplate的接口，用于简化Redis的操作。

因为Spring Data Redis引入的依赖项很多，如果只是为了使用Redis，完全可以只引入Lettuce，剩下的操作都自己来完成。

如何把一个第三方组件引入到Spring Boot中：

首先，添加必要的几个依赖项：

```xml
<dependency>
    <groupId>io.lettuce</groupId>
    <artifactId>lettuce-core</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

在`spring-boot-starter-parent`中已经把常用组件的版本号确定下来了，因此不需要显式设置版本号。

第一步是在配置文件`application.yml`中添加Redis的相关配置：

```yaml
spring:
  redis:
    host: ${REDIS_HOST:localhost}
    port: ${REDIS_PORT:6379}
    password: ${REDIS_PASSWORD:}
    ssl: ${REDIS_SSL:false}
    database: ${REDIS_DATABASE:0}
```

然后，通过`RedisConfiguration`来加载它：

```java
@ConfigurationProperties("spring.redis")
public class RedisConfiguration {
	private String host;
	private int port;
	private String password;
	private int database;

    // getters and setters...
}
```

再编写一个`@Bean`方法来创建`RedisClient`，可以直接放在`RedisConfiguration`中：

```java
@ConfigurationProperties("spring.redis")
public class RedisConfiguration {
    ...

    @Bean
    RedisClient redisClient() {
        if ("".equals(this.password.trim())) {
     		this.password = null;
		}
        RedisURI uri = RedisURI.Builder.redis(this.host, this.port)
                .withPassword(this.password)
                .withDatabase(this.database)
                .build();
        return RedisClient.create(uri);
    }
}
```

在启动入口引入该配置：

```java
@SpringBootApplication
@Import(RedisConfiguration.class) // 加载Redis配置
public class Application {
    ...
}
```

如果在`RedisConfiguration`中标注`@Configuration`，则可通过Spring Boot的自动扫描机制自动加载，否则需要使用`@Import`手动加载。

用一个`RedisService`来封装所有的Redis操作。基础代码如下：

```java
@Component
public class RedisService {
    @Autowired
    RedisClient redisClient;

    GenericObjectPool<StatefulRedisConnection<String, String>> redisConnectionPool;

    @PostConstruct
    public void init() {
        GenericObjectPoolConfig<StatefulRedisConnection<String, String>> poolConfig = new GenericObjectPoolConfig<>();
        poolConfig.setMaxTotal(20);
        poolConfig.setMaxIdle(5);
        poolConfig.setTestOnReturn(true);
        poolConfig.setTestWhileIdle(true);
        this.redisConnectionPool = ConnectionPoolSupport.createGenericObjectPool(() -> redisClient.connect(), poolConfig);
    }

    @PreDestroy
    public void shutdown() {
        this.redisConnectionPool.close();
        this.redisClient.shutdown();
    }
}
```

上述代码引入了Commons Pool的一个对象池，用于缓存Redis连接。因为Lettuce本身是基于Netty的异步驱动，在异步访问时并不需要创建连接池，但基于Servlet模型的同步访问时，连接池是有必要的。连接池在`@PostConstruct`方法中初始化，在`@PreDestroy`方法中关闭。

下一步，是在`RedisService`中添加Redis访问方法。为了简化代码，仿照`JdbcTemplate.execute(ConnectionCallback)`方法，传入回调函数，可大幅减少样板代码。

首先定义回调函数接口`SyncCommandCallback`：

```java
@FunctionalInterface
public interface SyncCommandCallback<T> {
    // 在此操作Redis:
    T doInConnection(RedisCommands<String, String> commands);
}
```

编写`executeSync`方法，在该方法中，获取Redis连接，利用callback操作Redis，最后释放连接，并返回操作结果：

```java
public <T> T executeSync(SyncCommandCallback<T> callback) {
    try (StatefulRedisConnection<String, String> connection = redisConnectionPool.borrowObject()) {
        connection.setAutoFlushCommands(true);
        RedisCommands<String, String> commands = connection.sync();
        return callback.doInConnection(commands);
    } catch (Exception e) {
        logger.warn("executeSync redis failed.", e);
        throw new RuntimeException(e);
    }
}
```

可以针对常用操作把它封装一下，例如`set`和`get`命令：

```java
public String set(String key, String value) {
    return executeSync(commands -> commands.set(key, value));
}

public String get(String key) {
    return executeSync(commands -> commands.get(key));
}
```

类似的，`hget`和`hset`操作如下：

```java
public boolean hset(String key, String field, String value) {
    return executeSync(commands -> commands.hset(key, field, value));
}

public String hget(String key, String field) {
    return executeSync(commands -> commands.hget(key, field));
}

public Map<String, String> hgetall(String key) {
    return executeSync(commands -> commands.hgetall(key));
}
```

常用命令可以提供方法接口，如果要执行任意复杂的操作，就可以通过`executeSync(SyncCommandCallback<T>)`来完成。

完成了`RedisService`后，就可以使用Redis了。例如，在`UserController`中，在Session中只存放登录用户的ID，用户信息存放到Redis，提供两个方法用于读写：

```java
@Controller
public class UserController {
    public static final String KEY_USER_ID = "__userid__";
    public static final String KEY_USERS = "__users__";

    @Autowired ObjectMapper objectMapper;
    @Autowired RedisService redisService;

    // 把User写入Redis:
    private void putUserIntoRedis(User user) throws Exception {
        redisService.hset(KEY_USERS, user.getId().toString(), objectMapper.writeValueAsString(user));
    }

    // 从Redis读取User:
    private User getUserFromRedis(HttpSession session) throws Exception {
        Long id = (Long) session.getAttribute(KEY_USER_ID);
        if (id != null) {
            String s = redisService.hget(KEY_USERS, id.toString());
            if (s != null) {
                return objectMapper.readValue(s, User.class);
            }
        }
        return null;
    }
    ...
}
```

用户登录成功后，把ID放入Session，把`User`实例放入Redis：

```java
@PostMapping("/signin")
public ModelAndView doSignin(@RequestParam("email") String email, @RequestParam("password") String password, HttpSession session) throws Exception {
    try {
        User user = userService.signin(email, password);
        session.setAttribute(KEY_USER_ID, user.getId());
        putUserIntoRedis(user);
    } catch (RuntimeException e) {
        return new ModelAndView("signin.html", Map.of("email", email, "error", "Signin failed"));
    }
    return new ModelAndView("redirect:/profile");
}
```

需要获取`User`时，从Redis取出：

```java
@GetMapping("/profile")
public ModelAndView profile(HttpSession session) throws Exception {
    User user = getUserFromRedis(session);
    if (user == null) {
        return new ModelAndView("redirect:/signin");
    }
    return new ModelAndView("profile.html", Map.of("user", user));
}
```

从Redis读写Java对象时，序列化和反序列化是应用程序的工作，上述代码使用JSON作为序列化方案，简单可靠。也可将相关序列化操作封装到`RedisService`中，这样可以提供更加通用的方法：

```java
public <T> T get(String key, Class<T> clazz) {
    ...
}

public <T> T set(String key, T value) {
    ...
}
```

------

redis在各大操作系统中的安装使用都非常简单，默认配置就是监听`127.0.0.1:6379`，且无帐号密码。

[在windows通过虚拟机安装redis](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-on-windows/)，或通过doker镜像运行：

```bash
docker pull redis
docker run --name my-redis -p 6379:6379 -d redis
```

也可以使用[古早版本](https://github.com/microsoftarchive/redis)，输入：`redis-server redis.windows.conf` 即可启动redis。

如果要部署Redis为windows下的服务，可以输入：`redis-server --service-install redis.windows.conf`。

其他常用命令：

- 卸载服务：`redis-server --service-uninstall`
- 开启服务：`redis-server --service-start`
- 停止服务：`redis-server --service-stop`

测试：

可以通过set、get指令查看是否成功启动：

```cmd
C:\Redis>redis-cli
127.0.0.1:6379>set A 123
127.0.0.1:6379>get A
```

## 集成Artemis

ActiveMQ Artemis是一个JMS服务器，在[集成JMS](/2024/06/05/Spring/#集成jms)一节中介绍如何在Spring中集成Artemis，现在介绍在Spring Boot中集成Artemis。

创建一个`springboot-jms`工程，引入的依赖除了`spring-boot-starter-web`，`spring-boot-starter-jdbc`等以外，新增`spring-boot-starter-artemis`：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-artemis</artifactId>
</dependency>
```

同样无需指定版本号。

如何创建Artemis服务器已经在集成JMS一节中详细讲述了，此处不再重复。创建Artemis服务器后，在`application.yml`中加入相关配置：

```yaml
spring:
  artemis:
    # 指定连接外部Artemis服务器，而不是启动嵌入式服务:
    mode: native
    # 服务器地址和端口号:
    host: 127.0.0.1
    port: 61616
    # 连接用户名和口令由创建Artemis服务器时指定:
    user: admin
    password: password
```

和Spring版本的JMS代码相比，使用Spring Boot集成JMS时，只要引入了`spring-boot-starter-artemis`，Spring Boot会自动创建JMS相关的`ConnectionFactory`、`JmsListenerContainerFactory`、`JmsTemplate`等，无需再手动配置了。

发送消息时只需要引入`JmsTemplate`：

```java
@Component
public class MessagingService {
    @Autowired
    JmsTemplate jmsTemplate;

    public void sendMailMessage() throws Exception {
        String text = "...";
        jmsTemplate.send("jms/queue/mail", new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage(text);
            }
        });
    }
}
```

接收消息时只需要标注`@JmsListener`：

```java
@Component
public class MailMessageListener {
    final Logger logger = LoggerFactory.getLogger(getClass());

    @JmsListener(destination = "jms/queue/mail", concurrency = "10")
    public void onMailMessageReceived(Message message) throws Exception {
        logger.info("received message: " + message);
    }
}
```

应用程序收发消息的逻辑和Spring中使用JMS完全相同，只是通过Spring Boot，可以把工程简化到只需要设定Artemis相关配置。

## 集成RabbitMQ

JMS是JavaEE的消息服务标准接口，但是，如果Java程序要和另一种语言编写的程序通过消息服务器进行通信，那么JMS就不太适合了。

AMQP是一种使用广泛的独立于语言的消息协议，它的全称是Advanced Message Queuing Protocol，即高级消息队列协议，它定义了一种二进制格式的消息流，任何编程语言都可以实现该协议。实际上，Artemis也支持AMQP，但实际应用最广泛的AMQP服务器是使用[Erlang](https://www.erlang.org/)编写的[RabbitMQ](https://www.rabbitmq.com/)。

**安装RabbitMQ**

先从RabbitMQ的官网[下载](https://www.rabbitmq.com/download.html)并安装RabbitMQ，安装和启动RabbitMQ请参考官方文档。要验证启动是否成功，可以访问RabbitMQ的管理后台[http://localhost:15672](http://localhost:15672/)，RabbitMQ后台管理的默认用户名和口令均为`guest`。

**AMQP协议**

AMQP协议和JMS协议有所不同。在JMS中，有两种类型的消息通道：

1. 点对点的Queue，即Producer发送消息到指定的Queue，接收方从Queue收取消息；
2. 一对多的Topic，即Producer发送消息到指定的Topic，任意多个在线的接收方均可从Topic获得一份完整的消息副本。

而AMQP协议比JMS要复杂一点，它只有Queue，没有Topic，并且引入了Exchange的概念。当Producer想要发送消息的时候，它将消息发送给Exchange，由Exchange将消息根据各种规则投递到一个或多个Queue：

```ascii
                                    ┌───────┐
                                ┌──▶│Queue-1│
                  ┌──────────┐  │   └───────┘
              ┌──▶│Exchange-1│──┤
┌──────────┐  │   └──────────┘  │   ┌───────┐
│Producer-1│──┤                 ├──▶│Queue-2│
└──────────┘  │   ┌──────────┐  │   └───────┘
              └──▶│Exchange-2│──┤
                  └──────────┘  │   ┌───────┐
                                └──▶│Queue-3│
                                    └───────┘
```

如果某个Exchange总是把消息发送到固定的Queue，那么这个消息通道就相当于JMS的Queue。如果某个Exchange把消息发送到多个Queue，那么这个消息通道就相当于JMS的Topic。和JMS的Topic相比，Exchange的投递规则更灵活，比如一个“登录成功”的消息被投递到Queue-1和Queue-2，而“登录失败”的消息则被投递到Queue-3。这些路由规则称之为Binding，通常都在RabbitMQ的管理后台设置。

在RabbitMQ中，首先创建3个Queue，分别用于发送邮件、短信和App通知：q_app、q_mail、q_sms。

创建Queue时注意到可配置为持久化（Durable）和非持久化（Transient），当Consumer不在线时，持久化的Queue会暂存消息，非持久化的Queue会丢弃消息。

然后在Exchanges中创建一个Direct类型的Exchange，命名为`registration`，并添加q_mail、q_sms到Binding。

Binding的规则就是：凡是发送到`registration`这个Exchange的消息，均被发送到`q_mail`和`q_sms`这两个Queue。

再创建一个Direct类型的Exchange，命名为`login`，并添加q_app、q_mail、q_sms到Binding，且指定q_sms，Routing Key="login_failed"。

当发送消息给`login`这个Exchange时，如果消息没有指定Routing Key，则被投递到`q_app`和`q_mail`，如果消息指定了Routing Key="login_failed"，那么消息被投递到`q_sms`。

配置好RabbitMQ后，就可以基于Spring Boot开发AMQP程序。

**使用RabbitMQ**

首先创建Spring Boot工程`springboot-rabbitmq`，并添加如下依赖引入RabbitMQ：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

然后在`application.yml`中添加RabbitMQ相关配置：

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

并在`Application`中添加一个`MessageConverter`：

```java
import org.springframework.amqp.support.converter.MessageConverter;

@SpringBootApplication
public class Application {
    ...

    @Bean
    MessageConverter createMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

`MessageConverter`用于将Java对象转换为RabbitMQ的消息。默认情况下，Spring Boot使用`SimpleMessageConverter`，只能发送`String`和`byte[]`类型的消息，不太方便。使用`Jackson2JsonMessageConverter`，就可以发送JavaBean对象，由Spring Boot自动序列化为JSON并以文本消息传递。

因为引入了starter，所有RabbitMQ相关的Bean均自动装配。

可以直接注入`RabbitTemplate`：

```java
@Component
public class MessagingService {
    @Autowired
    RabbitTemplate rabbitTemplate;

    public void sendRegistrationMessage(RegistrationMessage msg) {
        rabbitTemplate.convertAndSend("registration", "", msg);
    }

    public void sendLoginMessage(LoginMessage msg) {
        String routingKey = msg.success ? "" : "login_failed";
        rabbitTemplate.convertAndSend("login", routingKey, msg);
    }
}
```

发送消息时，使用`convertAndSend(exchange, routingKey, message)`可以指定Exchange、Routing Key以及消息本身。这里传入JavaBean后会自动序列化为JSON文本。上述代码将`RegistrationMessage`发送到`registration`，将`LoginMessage`发送到`login`，并根据登录是否成功来指定Routing Key。

接收消息时，需要在消息处理的方法上标注`@RabbitListener`：

```java
@Component
public class QueueMessageListener {
    final Logger logger = LoggerFactory.getLogger(getClass());

    static final String QUEUE_MAIL = "q_mail";
    static final String QUEUE_SMS = "q_sms";
    static final String QUEUE_APP = "q_app";

    @RabbitListener(queues = QUEUE_MAIL)
    public void onRegistrationMessageFromMailQueue(RegistrationMessage message) throws Exception {
        logger.info("queue {} received registration message: {}", QUEUE_MAIL, message);
    }

    @RabbitListener(queues = QUEUE_SMS)
    public void onRegistrationMessageFromSmsQueue(RegistrationMessage message) throws Exception {
        logger.info("queue {} received registration message: {}", QUEUE_SMS, message);
    }

    @RabbitListener(queues = QUEUE_MAIL)
    public void onLoginMessageFromMailQueue(LoginMessage message) throws Exception {
        logger.info("queue {} received message: {}", QUEUE_MAIL, message);
    }

    @RabbitListener(queues = QUEUE_SMS)
    public void onLoginMessageFromSmsQueue(LoginMessage message) throws Exception {
        logger.info("queue {} received message: {}", QUEUE_SMS, message);
    }

    @RabbitListener(queues = QUEUE_APP)
    public void onLoginMessageFromAppQueue(LoginMessage message) throws Exception {
        logger.info("queue {} received message: {}", QUEUE_APP, message);
    }
}
```

上述代码一共定义了5个Consumer，监听3个Queue。

启动应用程序，注册一个新用户，然后发送一条`RegistrationMessage`消息。此时，根据`registration`这个Exchange的设定，会在两个Queue收到消息：

```text
try register by bob@example.com...
user registered: bob@example.com
queue q_mail received registration message: [RegistrationMessage: email=bob@example.com, name=Bob]
queue q_sms received registration message: [RegistrationMessage: email=bob@example.com, name=Bob]
```

当登录失败时，发送`LoginMessage`并设定Routing Key为`login_failed`，此时，只有`q_sms`会收到消息：

```text
try login by bob@example.com...
queue q_sms received message: [LoginMessage: email=bob@example.com, name=(unknown), success=false]
```

登录成功后，发送`LoginMessage`，此时，`q_mail`和`q_app`将收到消息：

```text
try login by bob@example.com...
queue q_mail received message: [LoginMessage: email=bob@example.com, name=Bob, success=true]
queue q_app received message: [LoginMessage: email=bob@example.com, name=Bob, success=true]
```

RabbitMQ还提供了使用Topic的Exchange（此Topic指消息的标签，并非JMS的Topic概念），可以使用`*`进行匹配并路由。可见，掌握RabbitMQ的核心是理解其消息的路由规则。

直接指定一个Queue并投递消息也是可以的，此时指定Routing Key为Queue的名称即可，因为RabbitMQ提供了一个`default exchange`用于根据Routing Key查找Queue并直接投递消息到指定的Queue。但是要实现一对多的投递就必须自己配置Exchange。

示例：

```java
@Component
@RabbitListener(queues = { QueueMessageListener.QUEUE_APP, QueueMessageListener.QUEUE_MAIL,
                          QueueMessageListener.QUEUE_SMS })
public class QueueMessageListener {

    final Logger logger = LoggerFactory.getLogger(getClass());

    static final String QUEUE_MAIL = "q_mail";
    static final String QUEUE_SMS = "q_sms";
    static final String QUEUE_APP = "q_app";

    @RabbitHandler
    public void onRegistrationMessage(Message messageOriginal, RegistrationMessage message) throws Exception {
        String queueName = messageOriginal.getMessageProperties().getConsumerQueue();
        logger.info("queue {} received registration message: {}", queueName, message);
    }

    @RabbitHandler
    public void onLoginMessage(Message messageOriginal, LoginMessage message) throws Exception {
        String queueName = messageOriginal.getMessageProperties().getConsumerQueue();
        logger.info("queue {} received login message: {}", queueName, message);
    }
}
```

解释：
- **`@RabbitListener(queues = {...})`**:  这个注解用于声明该类监听哪些队列的消息。在这个示例中，`QueueMessageListener` 类监听 `QUEUE_APP`、`QUEUE_MAIL` 和 `QUEUE_SMS` 三个队列的消息。当这些队列有新消息到达时，该类会被触发。
- **`@RabbitHandler`**:  该注解用于标记处理消息的方法。一个类可以有多个 `@RabbitHandler` 方法，它们会根据消息的类型来自动选择对应的方法进行处理。
- **`onRegistrationMessage(Message messageOriginal, RegistrationMessage message)`**:  是一个处理注册消息的方法。`@RabbitHandler` 注解表明这个方法会处理 `RegistrationMessage` 类型的消息。
  - **`Message messageOriginal`**:  这个参数是原始的 RabbitMQ 消息对象，包含消息的元数据（如消息属性、队列名等）。
  - **`RegistrationMessage message`**:  这是实际的消息体，Spring 会自动将消息反序列化为 `RegistrationMessage` 对象。
  - **`queueName`**:  使用 `messageOriginal.getMessageProperties().getConsumerQueue()` 获取当前消息来自的队列名称。这在日志中用于记录消息来自哪个队列。

当 RabbitMQ 中的 `q_mail`、`q_sms` 或 `q_app` 队列有新消息时，`QueueMessageListener` 会自动接收消息。Spring AMQP 框架根据消息的类型（例如 `RegistrationMessage` 或 `LoginMessage`）来选择合适的 `@RabbitHandler` 方法处理消息。

- 如果消息是 `RegistrationMessage` 类型的，则调用`onRegistrationMessage` 方法。
- 如果消息是 `LoginMessage` 类型的，则调用 `onLoginMessage` 方法。

该机制允许根据消息的不同类型，使用不同的方法进行处理，简化了消息处理逻辑的组织。

## 集成Kafka

JMS是JavaEE的标准消息接口，Artemis是一个JMS实现产品，AMQP是跨语言的一个标准消息接口，RabbitMQ是一个AMQP实现产品。

Kafka也是一个消息服务器，它的特点一是快，二是有巨大的吞吐量，Kafka没有实现任何标准的消息接口，它自己提供的API就是Kafka的接口。

Kafka本身是Scala编写的，运行在JVM之上。Producer和Consumer都通过Kafka的客户端使用网络来与之通信。从逻辑上讲，Kafka设计非常简单，它只有一种类似JMS的Topic的消息通道：

```ascii
                           ┌──────────┐
                       ┌──▶│Consumer-1│
                       │   └──────────┘
┌────────┐    ┌─────┐  │   ┌──────────┐
│Producer│───▶│Topic│──┼──▶│Consumer-2│
└────────┘    └─────┘  │   └──────────┘
                       │   ┌──────────┐
                       └──▶│Consumer-3│
                           └──────────┘
```

Kafka的一个Topic可以有一个至多个Partition，并且可以分布到多台机器上：

```ascii
            ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
             Topic
            │                   │
                ┌───────────┐        ┌──────────┐
            │┌─▶│Partition-1│──┐│┌──▶│Consumer-1│
             │  └───────────┘  │ │   └──────────┘
┌────────┐  ││  ┌───────────┐  │││   ┌──────────┐
│Producer│───┼─▶│Partition-2│──┼─┼──▶│Consumer-2│
└────────┘  ││  └───────────┘  │││   └──────────┘
             │  ┌───────────┐  │ │   ┌──────────┐
            │└─▶│Partition-3│──┘│└──▶│Consumer-3│
                └───────────┘        └──────────┘
            └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

Kafka只保证**在一个Partition内部，消息是有序的**，但是，存在多个Partition的情况下，Producer发送的3个消息会依次发送到Partition-1、Partition-2和Partition-3，Consumer从3个Partition接收的消息并不一定是Producer发送的顺序，因此，多个Partition只能保证接收消息大概率按发送时间有序，并不能保证完全按Producer发送的顺序。这一点在使用Kafka作为消息服务器时要特别注意，对发送顺序有严格要求的Topic只能有一个Partition。

Kafka的另一个特点是消息发送和接收都尽量使用批处理，一次处理几十甚至上百条消息，比一次一条效率要高很多。

最后要注意的是消息的持久性。Kafka总是将消息写入Partition对应的文件，消息保存多久取决于服务器的配置，可以按照时间删除（默认3天），也可以按照文件大小删除，因此，只要Consumer在离线期内的消息还没有被删除，再次上线仍然可以接收到完整的消息流。这一功能实际上是客户端自己实现的，客户端会存储它接收到的最后一个消息的`offsetId`，再次上线后按上次的`offsetId`查询。`offsetId`是Kafka标识某个Partion的每一条消息的递增整数，客户端通常将它存储在ZooKeeper中。

**安装Kafka**

首先从Kafka官网[下载](https://kafka.apache.org/downloads)最新版Kafaka，解压后在`bin`目录找到两个文件：

- `zookeeper-server-start.sh`：启动ZooKeeper（已内置在Kafka中）；
- `kafka-server-start.sh`：启动Kafka。

先启动ZooKeeper：

```bash
$ ./zookeeper-server-start.sh ../config/zookeeper.properties 
```

再启动Kafka：

```bash
./kafka-server-start.sh ../config/server.properties
```

看到如下输出表示启动成功：

```text
... INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```

如果要关闭Kafka和ZooKeeper，依次按Ctrl-C退出即可。

**使用Kafka**

在Spring Boot中使用Kafka，首先要引入依赖：

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

注意这个依赖是`spring-kafka`项目提供的。

然后，在`application.yml`中添加Kafka配置：

```yml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      auto-offset-reset: latest
      max-poll-records: 100
      max-partition-fetch-bytes: 1000000
```

除了`bootstrap-servers`必须指定外，`consumer`相关的配置项均为调优选项。例如，`max-poll-records`表示一次最多抓取100条消息。如果要查看配置名称，可以在IDE里定义一个`KafkaProperties.Consumer`的变量：

```java
KafkaProperties.Consumer c = null;
```

然后按住Ctrl查看源码即可。

**发送消息**

Spring Boot自动创建了一个`KafkaTemplate`用于发送消息。这是一个泛型类，而默认配置总是使用`String`作为Kafka消息的类型，所以注入`KafkaTemplate<String, String>`即可：

```java
@Component
public class MessagingService {
    @Autowired ObjectMapper objectMapper;

    @Autowired KafkaTemplate<String, String> kafkaTemplate;

    public void sendRegistrationMessage(RegistrationMessage msg) throws IOException {
        send("topic_registration", msg);
    }

    public void sendLoginMessage(LoginMessage msg) throws IOException {
        send("topic_login", msg);
    }

    private void send(String topic, Object msg) throws IOException {
        ProducerRecord<String, String> pr = new ProducerRecord<>(topic, objectMapper.writeValueAsString(msg));
        pr.headers().add("type", msg.getClass().getName().getBytes(StandardCharsets.UTF_8));
        kafkaTemplate.send(pr);
    }
}
```

发送消息时，需指定Topic名称，消息正文。为了发送一个JavaBean，这里没有使用`MessageConverter`来转换JavaBean，而是直接把消息类型作为Header添加到消息中，Header名称为`type`，值为Class全名。消息正文是序列化的JSON。

**接收消息**

接收消息可以使用`@KafkaListener`注解：

```java
@Component
public class TopicMessageListener {
    private final Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    ObjectMapper objectMapper;

    @KafkaListener(topics = "topic_registration", groupId = "group1")
    public void onRegistrationMessage(@Payload String message, @Header("type") String type) throws Exception {
        RegistrationMessage msg = objectMapper.readValue(message, getType(type));
        logger.info("received registration message: {}", msg);
    }

    @KafkaListener(topics = "topic_login", groupId = "group1")
    public void onLoginMessage(@Payload String message, @Header("type") String type) throws Exception {
        LoginMessage msg = objectMapper.readValue(message, getType(type));
        logger.info("received login message: {}", msg);
    }

    @KafkaListener(topics = "topic_login", groupId = "group2")
    public void processLoginMessage(@Payload String message, @Header("type") String type) throws Exception {
        LoginMessage msg = objectMapper.readValue(message, getType(type));
        logger.info("process login message: {}", msg);
    }

    @SuppressWarnings("unchecked")
    private static <T> Class<T> getType(String type) {
        // TODO: use cache:
        try {
            return (Class<T>) Class.forName(type);
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }
}
```

在接收消息的方法中，使用`@Payload`表示传入的是消息正文，使用`@Header`可传入消息的指定Header，这里传入`@Header("type")`，就是我们发送消息时指定的Class全名。接收消息时，我们需要根据Class全名来反序列化获得JavaBean。

上述代码一共定义了3个Listener，其中有两个方法监听的是同一个Topic，但它们的Group ID不同。假设Producer发送的消息流是A、B、C、D，Group ID不同表示这是**两个不同的Consumer**，它们将分别收取完整的消息流，即各自均收到A、B、C、D。Group ID相同的多个Consumer实际上被视作**一个**Consumer，即如果有两个Group ID相同的Consumer，那么它们各自收到的很可能是A、C和B、D。

运行应用程序，注册新用户后：

```text
try register by bob@example.com...
user registered: bob@example.com
 received registration message: [RegistrationMessage: email=bob@example.com, name=Bob]
```

用户登录后，观察日志输出：

```text
try login by bob@example.com...
received login message: [LoginMessage: email=bob@example.com, name=Bob, success=true]
process login message: [LoginMessage: email=bob@example.com, name=Bob, success=true]
```

因为Group ID不同，同一个消息被两个Consumer分别独立接收。如果把Group ID改为相同，那么同一个消息只会被两者之一接收。

在Kafka中是如何创建Topic的？又如何指定某个Topic的分区数量？

实际上开发使用的Kafka默认允许自动创建Topic，创建Topic时默认的分区数量是2，可以通过`server.properties`修改默认分区数量。

在生产环境中通常会关闭自动创建功能，Topic需要由运维人员先创建好。和RabbitMQ相比，Kafka并不提供网页版管理后台，管理Topic需要使用命令行，比较繁琐，只有云服务商通常会提供更友好的管理后台。

## 使用Undertow

Undertow 和 Tomcat 都是优秀的嵌入式 Web 容器，但它们的实现方式不同：

- **Tomcat** 传统上更稳定，功能更齐全，并且支持很多企业级应用的标准和协议。
- **Undertow** 更轻量、响应更快，并且具有更高的吞吐量，特别适合高并发场景。它是非阻塞的，因此对于处理大量短连接、高并发的场景有很好的性能。

Spring Boot 允许你自定义 Undertow 的配置。例如，如果你希望调整最大连接数、线程池等设置，可以在 `application.properties` 或 `application.yml` 中进行配置：

```yaml
server:
  port: 9520
  undertow:
    # 每块buffer的空间大小，越小空间被利用越充分
    # 不要设置太大，以免影响其他应用，合适即可
    buffer-size: ${SERVER_UNDERTOW_BUFFER_SIZE:1024}
    # 是否分配的直接内存(NIO直接分配的堆外内存)
    # 默认false
    direct-buffers: true
    # HTTP POST请求最大的大小
    # 默认0，无限制，可设置10M
    max-http-post-size: 1GB
    threads:
      # 设置IO线程数, 它主要执行非阻塞的任务,它们会负责多个连接
      # 默认值为8，建议设置每个CPU核心一个线程
      io: ${SERVER_UNDERTOW_IO_THREADS:8}
      # 阻塞任务线程池, 当执行类似servlet请求阻塞操作, undertow会从这个线程池中取得线程
      # 默认等于 io-threads*8，它的值设置取决于系统的负载，可适当调大该值6
      worker: ${SERVER_UNDERTOW_WORKER_THREADS:128}
    max-headers: 1024
```

添加 `spring-boot-starter-undertow` 依赖，并且 **删除** `spring-boot-starter-tomcat` 依赖（如果已经添加过）。这样做的目的是告诉 Spring Boot 使用 Undertow 作为嵌入式容器。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

**验证功能和性能**

在更换容器之后，应该进行一些基本的验证和性能测试：

- **功能验证**：确保你的应用在 Undertow 上正常运行，尤其是关键功能（如请求参数解析、RESTful API、会话管理等）。
- **性能测试**：如果你的应用有高并发的需求，测试 Undertow 在并发场景下的性能。Undertow 在这方面可能会有更好的表现。

**注意事项**
- **兼容性问题**：虽然 Spring Boot 通过自动配置来帮助适配 Undertow，但在少数情况下，某些功能可能存在不同的行为。例如，Tomcat 和 Undertow 在连接池和线程管理上有所不同，如果你有自定义的连接池或线程池管理，可能需要根据容器的特性进行适当调整。
- **会话管理**：Tomcat 和 Undertow 对 HTTP 会话的管理方式可能略有不同，确保在迁移时验证会话和 cookie 的管理。
- **日志输出**：确保你的日志配置没有依赖于 Tomcat 特定的行为。通常，Spring Boot 会自动适配，但值得进行一次检查。

总结

更换容器为 Undertow 后，Spring Boot 会自动进行适配，并替换原有的 Tomcat 设置。你只需要修改 `pom.xml`，添加 Undertow 依赖，并删除 Tomcat 依赖。同时，你可以根据需要调整 Undertow 特定的配置来优化性能。大部分情况下，Spring Boot 会做最好的适配工作，但确保经过充分的测试，特别是在高并发和功能验证方面。























