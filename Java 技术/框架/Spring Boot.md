官网：https://spring.io/projects/spring-boot

> Spring Boot 可以轻松创建独立的、基于 Spring 的生产级应用程序，它可以让你“运行即可”。大多数 Spring Boot 应用程序只需要少量的 Spring 配置。

SpringBoot 功能：

- 创建独立的 Spring 应用程序
- 直接嵌入 Tomcat、Jetty 或 Undertow（无需部署 WAR 包，打包成 Jar 本身就是一个可以运行的应用程序）
- 提供一站式的 `starter` 依赖项，以简化 Maven 配置（需要整合什么框架，直接导对应框架的 starter 依赖）
- 尽可能自动配置 Spring 和第三方库（除非特殊情况，否则几乎不需要你进行什么配置）
- 提供生产就绪功能，如指标、运行状况检查和外部化配置
- 没有代码生成，也没有 XML 配置的要求

SpringBoot 是现在最主流的开发框架，它提供了一站式的开发体验，大幅度提高了我们的开发效率。

## 走进 SpringBoot

在 SSM 阶段，当需要搭建一个基于 Spring 全家桶的 Web 应用程序时，不得不做大量的依赖导入和框架整合相关的 Bean 定义，光是整合框架就花费了大量的时间，但是实际上发现，整合框架其实基本都是一些固定流程，每创建一个新的 Web 应用程序，基本都会使用同样的方式去整合框架，实际完全可以将一些重复的配置作为约定，只要框架遵守这个约定，提供默认的配置就好，这样就不用再去配置了，约定优于配置！<br>
而 SpringBoot 正是将这些过程大幅度进行了简化，它可以自动进行配置，开发者只需要导入对应的启动器（starter）依赖即可。

## 初始项目文件结构

在创建 SpringBoot 项目之后，首先会自动生成一个主类，而主类中的 `main` 方法中调用了 `SpringApplication` 类的静态方法来启动整个 SpringBoot 项目，并且主类的上方有一个 `@SpringBootApplication` 注解：

```java
@SpringBootApplication
public class SpringBootTestApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootTestApplication.class, args);
    }

}
```

- 测试类，测试类的上方仅添加了一个 `@SpringBootTest` 注解：

```java
@SpringBootTest
class SpringBootTestApplicationTests {
    @Test
    void contextLoads() {}
}
```

- pom.xml 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!-- 父工程 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>SpringBootStudy</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>SpringBootStudy</name>
    <description>SpringBootStudy</description>
    <properties>
        <java.version>8</java.version>
    </properties>
    <dependencies>
        <!-- spring-boot-starter：SpringBoot 核心启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- spring-boot-starter-test：SpringBoot 测试模块启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <!-- SpringBoot Maven插件 -->
        <plugins>
            <!-- 打包Jar -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

- application.properties：SpringBoot 的配置文件，所有依赖的配置都在这里编写，但是一般情况下只需要配置必要项即可。

## 整合 Web 相关框架

先导入依赖

```xml
<!-- spring-boot-starter-test：SpringBoot web 模块启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- Lombok 是一个 Java 库，能自动插入编辑器并构建工具，简化 Java 开发。通过添加注解的方式，不需要为类编写 getter() 或 equals() 方法，同时可以自动化日志变量。 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

完成后，直接运行项目主类，就可以直接访问：http://localhost:8080/<br>
由于没有编写任何的请求映射，所以没有数据。

![初次运行 Spring Boot 项目](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring%20Boot/%E5%88%9D%E6%AC%A1%E8%BF%90%E8%A1%8C%20Spring%20Boot%20%E9%A1%B9%E7%9B%AE.png)

Spring Boot 日志中除了最基本的 SpringBoot 启动日志以外，还新增了内嵌 Web 服务器（Tomcat）的启动日志，并且显示了当前 Web 服务器所开放的端口，并且自动帮助初始化了DispatcherServlet，但是只创建了项目，导入了 web 相关的 starter 依赖，没有进行任何的配置，实际上它使用的是 starter 提供的默认配置进行初始化的。

由于SpringBoot是自动扫描的，因此直接创建一个Controller即可被加载：

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class MainController {
    //直接访问 http://localhost:8080/ 即可，不用加 web 应用程序名称了
    @RequestMapping("/")
    @ResponseBody
    public String index() {
        return "Hello World";
    }
}
```

它还可以自动识别类型，如果返回的是一个对象类型的数据，那么它会自动转换为JSON数据格式，无需配置：

```java
import lombok.Data;

@Data
public class Student {
    int studentID;
    String name;
    String sex;
}
```

```java
@RequestMapping("/student")
@ResponseBody
public Student student(){
    Student student = new Student();
    student.setName("小明");
    student.setSex("男");
    student.setSid(10);
    return student;
}
```

最后浏览器能够直接得到 `application/json` 的响应数据。

![JSON 响应对象数据](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring%20Boot/JSON%20%E5%93%8D%E5%BA%94%E5%AF%B9%E8%B1%A1%E6%95%B0%E6%8D%AE.png)

### 修改 Web 相关配置

如果需要修改 Web 服务器的端口或是一些其他的内容，可以直接在 `application.properties` 中进行修改，它是整个 SpringBoot 的配置文件：

```properties
# 修改端口为80
server.port=80
```

还可以编写自定义的配置项，并在项目中通过`@Value`直接注入：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class MainController {
    @Value("${test.data}")
    int data;

    @RequestMapping("/")
    @ResponseBody
    public String index() {
        return "Hello World" + data;
    }
}
```

```properties
test.data=100
```

![自定义配置项](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring%20Boot/%E8%87%AA%E5%AE%9A%E4%B9%89%E9%85%8D%E7%BD%AE%E9%A1%B9.png)

通过这种方式，就可以更好地将一些需要频繁修改的配置项写在配置文件中，并通过注解方式去获取值。

配置文件除了使用 `properties` 格式以外，还有一种叫做 `yaml` 格式，它的语法如下：

```yaml
一级目录:
	二级目录:
	  三级目录1: 值
	  三级目录2: 值
	  三级目录List: 
	  - 元素1
	  - 元素2
	  - 元素3
```

每一级目录都是通过缩进（不能使用Tab，只能使用空格）区分，并且键和值之间需要添加 `冒号 + 空格` 来表示。

SpringBoot 也支持这种格式的配置文件，可以将 `application.properties` 修改为`application.yml` 或是 `application.yaml` 来使用 YAML 语法编写配置：

```yaml
server:
  port: 80
```

## 整合 Spring Security

导入 Spring Boot 项目 Spring Security 的 starter 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

导入依赖后，直接启动 SpringBoot 应用程序，SpringSecurity 会自动生成一个默认用户`user`，它的密码会出现在日志中：

```
2023-03-12 17:58:13.762  INFO 11536 --- [           main] .s.s.UserDetailsServiceAutoConfiguration : 

Using generated security password: 81638855-cf00-4c0a-b653-c1b288fc6bf1
```

其中`81638855-cf00-4c0a-b653-c1b288fc6bf1`就是随机生成的一个密码，我们可以使用用户名：`user` 和此密码在 security 自动生成的登录页面完成登录。

![security 生成的登录页面](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring%20Boot/Security%20%E7%94%9F%E6%88%90%E7%9A%84%E7%99%BB%E5%BD%95%E9%A1%B5%E9%9D%A2.png)

也可以在配置文件中直接配置：

```yaml
spring:
  security:
    user:
      name: test   # 用户名
      password: 123456  # 密码
      roles:   # 角色
      - user
      - admin
```

实际上这样的配置方式就是一个 `inMemoryAuthentication`，只是可以直接配置而已。

当然，页面的控制和数据库验证还是需要提供 `WebSecurityConfigurerAdapter` 的实现类去完成：

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/login").permitAll()
            .anyRequest().hasRole("user")
            .and()
            .formLogin();
    }
}
```

注意这里不需要再添加 `@EnableWebSecurity` 了，因为 starter 依赖已经帮我们添加了。

## 整合 MyBatis

同样的，只需要导入对应的 starter 依赖即可：

```xml
<!-- SpringBoot 整合 mybatis 模块 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.0</version>
</dependency>
<!-- 连接 MySQL 的驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

导入依赖后，直接启动会报错，是因为有必要的配置我们没有去编写，需要指定数据源的相关信息：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/数据库名
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

再次启动，成功。

Spring Boot 整合 MyBatis 后，不需要再去设置 MyBatis 的相关 Bean 了，也不需要添加任何 `@MapperSacn` 注解，因为 starter 已经做好了，它会自动扫描项目中添加了 `@Mapper` 注解的接口，直接将其注册为 Bean，不需要进行任何配置。

```java
import lombok.Data;

@Data
public class UserData {
    Integer id;
    String username;
    String password;
    String role;
}
```

```java
@Mapper
public interface MainMapper {
    @Select("select * from users where username = #{username}")
    UserData findUserByName(String username);
}
```

当然也可以直接 `@MapperScan` 使用包扫描。

```java
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("xxx.xxx.mapper")
public class SpringBootStudyApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootStudyApplication.class, args);
    }
}
```

添加Mapper之后，使用方法和 SSM 阶段是一样的，可以将其与 Spring Security 结合使用：

```java
import com.example.springbootstudy.enity.UserData;
import com.example.springbootstudy.mapper.MainMapper;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
public class UserAuthService implements UserDetailsService {
    @Resource
    MainMapper mapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserData data = mapper.findUserByName(username);
        if(data == null) throw new UsernameNotFoundException("用户 "+username+" 登录失败，用户名不存在！");
        return User
                .withUsername(data.getUsername())
                .password(data.getPassword())
                .roles(data.getRole())
                .build();
    }
}
```

最后在 SecurityConfiguration.java 配置一下自定义验证即可。

> <font color="green">提示</font>
>
> 自定义验证以后，那么之前在 application.yml 配置文件里面配置的 Spring Security 用户就失效了。

```java
import com.example.springbootstudy.service.UserAuthService;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

import javax.annotation.Resource;

@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Resource
    UserAuthService userAuthService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/login").permitAll()
                // .anyRequest().hasRole("user")    只有 user 角色可以通过验证
                .anyRequest().hasanyRole("admin", "user")   //amdin 和 user 角色都可以通过验证
                .and()
                .formLogin();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .userDetailsService(userAuthService)
                .passwordEncoder(new BCryptPasswordEncoder());
    }
}
```

> 日志系统如若出现此系统 `Encoded password does not look like BCrypt`，说明数据库中的密码未加密，需要数据库中的密码是加密后的结果。
>
> 可用此方法获取字段加密后的结果，然后手动录入数据库。
>
> ```java
> @Test
> public void test(){
> BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
> System.out.println(encoder.encode("user")); //输入想要加密的 password
> }
> ```

在首次使用时，会发现日志中输出以以下语句：

```
2023-03-12 21:01:26.361  INFO 16396 --- [nio-8080-exec-4] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2023-03-12 21:01:26.871  INFO 16396 --- [nio-8080-exec-4] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
```

实际上，SpringBoot 会自动为 Mybatis 配置数据源，默认使用的就是 `HikariCP` 数据源。

## 整合 Thymeleaf

整合 Thymeleaf 也只需导入对应的 starter 即可：

```xml
<!-- spring boot 整合 thymeleaf -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

接着直接使用即可：

```java
@RequestMapping("/login")
public String login() {
    return "login";
}
```

> <font color="blue">注意</font>
>
> 这样只能正常解析 HTML 页面，但是 js、css 等静态资源需要进行路径指定，不然无法访问。

在 application.yml 配置文件中配置静态资源的访问前缀：

```yaml
spring:
	mvc:
  	static-path-pattern: /static/**
```

接着把登陆页面实现一下吧。

记得在 html 页面前缀加上 Thymeleaf 空间

```html
<html lang="en" xmlns:th=http://www.thymeleaf.org
xmlns:sec=http://www.thymeleaf.org/extras/spring-security>
```

SecurityConfiguration.java

```java
import com.example.springbootstudy.service.UserAuthService;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl;

import javax.annotation.Resource;
import javax.sql.DataSource;

@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Resource
    UserAuthService userAuthService;
    @Resource
    DataSource dataSource;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/static/**").permitAll()
                .anyRequest().hasAnyRole("user", "admin")
                .and()
                .formLogin()
                .loginPage("/login")
                .loginProcessingUrl("/doLogin")
                .permitAll()
                .defaultSuccessUrl("/index",true)
                .and()
                .rememberMe()
                .tokenRepository(new JdbcTokenRepositoryImpl(){{setDataSource(dataSource);}});
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .userDetailsService(userAuthService)
                .passwordEncoder(new BCryptPasswordEncoder());
    }
}
```

## 日志系统

### 日志门面和日志实现

日志实现：如 JUL（Java util logging，Java 原生日志框架）。可以直接使用JUL为日志框架来规范化打印日志

日志门面（Facade）：如 Slf4j（simple logging facade for java），是把不同的日志系统的实现进行了具体的抽象化，只提供了统一的日志使用接口，使用时只需要按照其提供的接口方法进行调用即可。由于它只是一个接口，并不是一个具体的可以直接单独使用的日志框架，所以最终日志的格式、记录级别、输出方式等都要通过接口绑定的具体的日志系统来实现。这些具体的日志系统就有 log4j、logback、java.util.logging 等，它们才实现了具体的日志系统的功能

日志门面和日志实现就像JDBC和数据库驱动一样，一个是画大饼的，一个是真的去做饼的。

![SLF4J](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring%20Boot/SLF4J.png)

不同的框架可能使用了不同的日志框架，希望所有的框架一律使用日志门面（Slf4j）进行日志打印。<br>可以采取类似于偷梁换柱的做法，只保留不同日志框架的接口和类定义等关键信息，而将实现全部定向为 Slf4j 调用。相当于有着和原有日志框架一样的外壳，对于其他框架来说依然可以使用对应的类进行操作，而具体如何执行，真正的内心已经是Slf4j的了。

![SLF4J 调用其他日志实现](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring%20Boot/SLF4J%20%E8%B0%83%E7%94%A8%E5%85%B6%E4%BB%96%E6%97%A5%E5%BF%97%E5%AE%9E%E7%8E%B0.png)

SpringBoot 为了统一日志框架的使用，做了这些事情：

* 直接将其他依赖以前的日志框架剔除
* 导入对应日志框架的Slf4j中间包
* 导入自己官方指定的日志实现，并作为 Slf4j 的日志实现层

### 在SpringBoot中打印日志信息

SpringBoot 使用的是 Slf4j 作为日志门面，Logback【[Logback](http://logback.qos.ch/) 是 log4j 框架的作者开发的新一代日志框架，它效率更高、能够适应诸多的运行环境，同时天然支持 SLF4J】作为日志实现，对应的依赖为：

```xml
<!-- spring boot 日志系统 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

导入依赖后需要打印日志时，可以像这样：

```java
@RequestMapping("/login")
public String login(){
    Logger logger = LoggerFactory.getLogger(MainController.class);
    logger.info("用户访问了一次登陆界面");
    return "login";
}
```

同时因为之前已经在整合 Web 时添加了 Lombok 依赖，所以也可以直接一个注解搞定：

```java
@Slf4j
@Controller
public class MainController {
    @RequestMapping("/login")
    public String login(){
        log.info("用户访问了一次登陆界面");
        return "login";
    }
}
```

日志级别从低到高分为 TRACE（轨迹） < DEBUG（调试排错） < INFO（信息） < WARN（警告） < ERROR（错误） < FATAL（致命的），SpringBoot 默认只会打印 INFO 以上级别的信息。

### 配置 Logback 日志

[Logback官网](https://logback.qos.ch)

和 JUL 一样，Logback 也能实现定制化。开发者可以编写对应的配置文件，SpringBoot 推荐将配置文件名称命名为 `logback-spring.xml` 表示这是 SpringBoot 下 Logback 专用的配置，可以使用SpringBoot 的高级 Proﬁle 功能，它的内容类似于这样：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 配置 -->
</configuration>
```

最外层由 `configuration` 标签包裹，一旦编写，那么就会替换默认的配置，所以如果内部什么都不写的话，那么会导致 SpringBoot 项目没有配置任何日志输出方式，控制台也不会打印日志。

配置一个控制台日志打印时可以直接导入并使用 SpringBoot 预设好的日志格式。在 `本地仓库\org\springframework\boot\spring-boot\{SpringBoot 版本号}\spring-boot-{SpringBoot 版本号}.jar!\org\springframework\boot\logging\logback\defaults.xml` 中已经把日志的输出格式定义好了，开发者只需要设置对应的 `appender` 即可：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
为导入提供默认的 logback 配置
-->

<included>
	<conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
	<conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
	<conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />

	<property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
	<property name="CONSOLE_LOG_CHARSET" value="${CONSOLE_LOG_CHARSET:-${file.encoding:-UTF-8}}"/>
	<property name="FILE_LOG_PATTERN" value="${FILE_LOG_PATTERN:-%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
	<property name="FILE_LOG_CHARSET" value="${FILE_LOG_CHARSET:-${file.encoding:-UTF-8}}"/>

	<logger name="org.apache.catalina.startup.DigesterFactory" level="ERROR"/>
	<logger name="org.apache.catalina.util.LifecycleBase" level="ERROR"/>
	<logger name="org.apache.coyote.http11.Http11NioProtocol" level="WARN"/>
	<logger name="org.apache.sshd.common.util.SecurityUtils" level="WARN"/>
	<logger name="org.apache.tomcat.util.net.NioSelectorPool" level="WARN"/>
	<logger name="org.eclipse.jetty.util.component.AbstractLifeCycle" level="ERROR"/>
	<logger name="org.hibernate.validator.internal.util.Version" level="WARN"/>
	<logger name="org.springframework.boot.actuate.endpoint.jmx" level="WARN"/>
</included>
```

#### 创建控制台日志打印

导入后，利用预设的日志格式创建一个控制台日志打印：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!--  导入其他配置文件，作为预设  -->
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />

    <!--  Appender 作为日志打印器配置，这里命名随意  -->
    <!--  ch.qos.logback.core.ConsoleAppender 是专用于控制台的 Appender  -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>${CONSOLE_LOG_CHARSET}</charset>
        </encoder>
    </appender>

    <!--  指定日志输出级别，以及启用的 Appender，这里就使用了上面的 ConsoleAppender  -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

配置完成后，控制台已经可以正常打印日志信息了。

#### 开启文件打印

只需要配置一个对应的 Appender 即可：

```xml
<!--  ch.qos.logback.core.rolling.RollingFileAppender 用于文件日志记录，它支持滚动  -->
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <encoder>
        <pattern>${FILE_LOG_PATTERN}</pattern>
        <charset>${FILE_LOG_CHARSET}</charset>
    </encoder>
    <!--  自定义滚动策略，防止日志文件无限变大，也就是日志文件写到什么时候为止，重新创建一个新的日志文件开始写  -->
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <!--  文件保存位置以及文件命名规则，这里用到了 %d{yyyy-MM-dd} 表示当前日期，%i 表示这一天的第 N 个日志  -->
        <FileNamePattern>log/%d{yyyy-MM-dd}-spring-%i.log</FileNamePattern>
        <!--  到期自动清理日志文件  -->
        <cleanHistoryOnStart>true</cleanHistoryOnStart>
        <!--  最大日志保留时间，此处设定为 7 天自动清理  -->
        <maxHistory>7</maxHistory>
        <!--  最大单个日志文件大小  -->
        <maxFileSize>10MB</maxFileSize>
    </rollingPolicy>
</appender>

<!--  指定日志输出级别，以及启用的 Appender，这里就使用了上面的 RollingFileAppender  -->
<root level="INFO">
    <appender-ref ref="FILE"/>
</root>
```

配置完成后，日志文件也能自动生成了。

#### 自定义官方提供的日志格式

官方文档：https://logback.qos.ch/manual/layouts.html

这里需要提及的是 MDC 机制，Logback 内置的日志字段还是比较少，如果需要打印有关业务的更多的内容，包括自定义的一些数据，需要借助 logback MDC 机制，MDC 为 `Mapped Diagnostic Context（映射诊断上下文）`，即将一些运行时的上下文数据通过 logback 打印出来，此时需要借助 org.sl4j.MDC 类。

比如需要记录是哪个用户访问网站的日志，只要是此用户访问网站，都会在日志中携带该用户的ID，希望每条日志中都携带这样一段信息文本，而官方提供的字段无法实现此功能，这时就需要使用 MDC 机制：

```java
@Slf4j
@Controller
public class MainController {
    @RequestMapping("/login")
    public String login(){
      	//这里用 Session 代替 ID
        MDC.put("reqId", request.getSession().getId());
        log.info("用户访问了一次登陆界面");
        return "login";
    }
}
```

通过这种方式，就可以向日志中传入自定义参数了。在日志中添加这样一个占位符 `%X{键值}`，名字保持一致：

```xml
 %clr([%X{reqId}]){faint} 
```

这样当向 MDC 中添加信息后，只要是当前线程（本质是 ThreadLocal 实现）下输出的日志，都会自动替换占位符。

### 自定义 Banner

实际上 Banner 部分和日志部分是独立的，SpringBoot 启动后，会先打印 Banner 部分，这个Banner 部分也是可以自定义。

可以直接来配置文件所在目录下创建一个名为 `banner.txt` 的文本文档，内容随意：

```txt
//                          _ooOoo_                               //
//                         o8888888o                              //
//                         88" . "88                              //
//                         (| ^_^ |)                              //
//                         O\  =  /O                              //
//                      ____/`---'\____                           //
//                    .'  \\|     |//  `.                         //
//                   /  \\|||  :  |||//  \                        //
//                  /  _||||| -:- |||||-  \                       //
//                  |   | \\\  -  /// |   |                       //
//                  | \_|  ''\---/''  |   |                       //
//                  \  .-\__  `-`  ___/-. /                       //
//                ___`. .'  /--.--\  `. . ___                     //
//              ."" '<  `.___\_<|>_/___.'  >'"".                  //
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
//      ========`-.____`-.___\_____/___.-`____.-'========         //
//                           `=---='                              //
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
//             佛祖保佑          永无 BUG         永不修改           //
```

可以使用在线生成网站进行生成自己的个性 Banner：https://www.bootschool.net/ascii

甚至还可以使用颜色代码来为文本切换颜色：

```
${AnsiColor.BRIGHT_GREEN}  //绿色
```

也可以获取一些常用的变量信息：

```
${AnsiColor.YELLOW} 当前 Spring Boot 版本：${spring-boot.version}
```

## 多环境配置

在日常开发中，项目会有多个环境。例如开发环境（develop）也就是研发过程中疯狂敲代码修 BUG 阶段，生产环境（production ）项目开发得差不多了，才可以放在服务器上跑了。<br>不同的环境下，可能配置文件也存在不同，但是不可能切换环境的时候又去重新写一次配置文件，所以开发者可以将多个环境的配置文件提前写好，进行自由切换。

由于 SpringBoot 只会读取 `application.properties` 或是 `application.yml` 文件，那么怎么才能实现自由切换呢？<br>SpringBoot 提供了一种方式：可以通过配置文件指定：

1. 创建两个环境的配置文件，`application-dev.yml` 和 `application-prod.yml`分别表示开发环境和生产环境的配置文件，比如开发环境使用的服务器端口为 8080，而生产环境下可能就需要设置为 80 或是 443 端口，那么这个时候就需要不同环境下的配置文件进行区分：

   ```yaml
   server:
     port: 8080
   ```

   ```yaml
   server:
     port: 80
   ```

2. SpringBoot 自带的 Logback 日志系统也是支持多环境配置的，比如想在开发环境下输出日志到控制台，而生产环境下只需要输出到文件即可，这时就需要进行环境配置：

   ```xml
   <springProfile name="dev">
       <root level="INFO">
           <appender-ref ref="CONSOLE"/>
           <appender-ref ref="FILE"/>
       </root>
   </springProfile>
   
   <springProfile name="prod">
       <root level="INFO">
           <appender-ref ref="FILE"/>
       </root>
   </springProfile>
   ```

> <font color="brown">注意</font>
>
> `springProfile` 是区分大小写的！

3. Maven 设置多环境

   ```xml
   <!--分别设置开发、生产环境-->
   <profiles>
       <!-- 开发环境 -->
       <profile>
           <id>dev</id>
           <activation>
               <activeByDefault>true</activeByDefault>
           </activation>
           <properties>
               <environment>dev</environment>
           </properties>
       </profile>
       <!-- 生产环境 -->
       <profile>
           <id>prod</id>
           <activation>
               <activeByDefault>false</activeByDefault>
           </activation>
           <properties>
               <environment>prod</environment>
           </properties>
       </profile>
   </profiles>
   ```

4. 需要根据环境的不同，排除其他环境的配置文件：

   ```xml
   <resources>
   <!--排除配置文件-->
       <resource>
           <directory>src/main/resources</directory>
           <!--先排除所有的配置文件-->
           <excludes>
               <!--使用通配符，当然可以定义多个 exclude 标签进行排除-->
               <exclude>application*.yml</exclude>
           </excludes>
       </resource>
   
       <!--根据激活条件引入打包所需的配置和文件-->
       <resource>
           <directory>src/main/resources</directory>
           <!--引入所需环境的配置文件-->
           <filtering>true</filtering>
           <includes>
               <include>application.yml</include>
               <!--根据 maven 选择环境导入配置文件-->
               <include>application-${environment}.yml</include>
           </includes>
       </resource>
   </resources>
   ```

5. 直接将 Maven 中的 `environment` 属性，传递给 SpringBoot 的配置文件，在构建时替换为对应的值：

   ```yaml
   spring:
     profiles:
       active: '@environment@'  #注意 YAML 配置文件需要加单引号，否则会报错
   ```

这样，根据 Maven 环境的切换，SpringBoot 的配置文件也会进行对应的切换。

最后打开 Maven 栏目，就可以自由切换了，直接勾选即可，注意切换环境之后要重新加载一下 Maven 项目，不然不会生效！

![maven 多环境选择](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring%20Boot/maven%20%E5%A4%9A%E7%8E%AF%E5%A2%83%E9%80%89%E6%8B%A9.png)

## 打包运行

点击 Maven 生命周期中的 `package` 即可，它会自动将其打包为可直接运行的 Jar 包，第一次打包可能会花费一些时间下载部分依赖的源码一起打包进 Jar 文件。

在打包的过程中还会完整的将项目跑一遍进行测试，如果不想测试直接打包，可以在执行 Maven 目标中使用以下命令：

```shell
mvn package  -DskipTests
```

> <font color="green">提示</font>
>
> ![执行 Maven 目标](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring%20Boot/%E6%89%A7%E8%A1%8C%20Maven%20%E7%9B%AE%E6%A0%87.png)

打包后，会得到一个名为`springboot-study-0.0.1-SNAPSHOT.jar`的文件，这时在 CMD 窗口中输入命令：

```shell
java -jar springboot-study-0.0.1-SNAPSHOT.jar
```

输入后，就可以看到 Java 项目成功运行起来了，如果手动关闭窗口会导致整个项目终止运行。

## 扩充 Spring 框架

### 任务调度

为了执行某些任务，可能需要一些非常规的操作：比如希望使用多线程来处理结果或是执行一些定时任务，到达指定时间再去执行。

之前可以创建一个新的线程来处理，或是使用 TimerTask 来完成定时任务，但现在 Spring 框架为此提供了更加便捷的方式进行任务调度。

#### 异步任务

1. 需要使用 Spring 异步任务支持，需要在配置类上添加 `@EnableAsync` 或是在 SpringBoot 的启动类上添加也可以。

   ```java
   @EnableAsync
   @SpringBootApplication
   public class SpringBootWebTestApplication {
       public static void main(String[] args) {
           SpringApplication.run(SpringBootWebTestApplication.class, args);
       }
   }
   ```

2. 在需要异步执行的方法上，添加 `@Async` 注解即可将此方法标记为异步，当此方法被调用时，会异步执行，也就是新开一个线程执行，不是在当前线程执行。

   ```java
   import org.springframework.scheduling.annotation.Async;
   import org.springframework.stereotype.Service;
   
   @Service
   public class TestService {
       @Async
       public void test(){
           try {
               Thread.sleep(3000);
               System.out.println("我是异步任务！");
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
       }
   }
   ```

   ```java
   import com.example.springbootstudy.service.TestService;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   import javax.annotation.Resource;
   
   @Controller
   public class MainController {
       @Resource
       TestService testService;
   
       @RequestMapping("/login")
       public String login() {
           testService.test();
           System.out.println("我是同步任务");
           return "login";
       }
   }
   ```

实际上这也是得益于 AOP 机制，通过线程池实现，但是也要注意，正是因为它是 AOP 机制的产物，所以它只能是在 Bean 中才会生效！

使用 @Async 注释的方法可以返回 'void' 或 "Future" 类型。

> Future 类是一种异步任务监视器，可以让提交者能够监视任务的执行，同时可以取消任务的执行，也可以获取任务返回结果。
>
> 作用：<br>		比如在做一定的任务运算的时候，需要等待比较长时间，这个任务是比较耗时的，需要比较繁重的运算，比如加密压缩等等。如果一直在等程序执行完成是不明智的，这时可以将这个比较耗时的任务交给子线程执行，然后通过 Future 类监视线程执行，获取返回的结果。
>
> 这样一来就提高了工作效率，这是一种异步的思想。【并发编程】

#### 定时任务

定时任务其实就是指定在哪个时候再去执行，在 JavaSE 阶段使用过 TimerTask 来执行定时任务。

Spring 中的定时任务是全局性质的，当 Spring 程序启动后，那么定时任务也就跟着启动了。开发者可以在配置类上添加 `@EnableScheduling` 或是在 SpringBoot 的启动类上添加也可。

1. 在 SpringBoot 的启动类上添加 `@EnableScheduling`

   ```java
   @EnableScheduling
   @SpringBootApplication
   public class SpringBootWebTestApplication {
       public static void main(String[] args) {
   		SpringApplication.run(SpringBootWebTestApplication.class, args);
       }
   }
   ```

2. 创建一个定时任务配置类，在配置类里面编写定时任务：

   ```java
   @Configuration
   public class ScheduleConfiguration {
       @Scheduled(fixedRate = 2000)
       public void task(){
           System.out.println("我是定时任务！"+new Date());
       }
   }
   ```

`@Scheduled` 中有很多参数，开发者需要指定 `cron`, `fixedDelay(String)`, or `fixedRate(String)` 的其中一个，否则无法创建定时任务。<br>它们的区别如下：

- fixedDelay：在上一次定时任务执行完之后，间隔多久继续执行。
- fixedRate：无论上一次定时任务有没有执行完成，两次任务之间的时间间隔。
- [cron](/Library/cron 表达式.md) ：使用 cron 表达式来指定任务计划。

### 监听器

监听实际上就是等待某个事件的触发，当事件触发时，对应事件的监听器就会被通知。

```java
@Component
public class TestListener implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println(event.getApplicationContext());
    }
}
```

通过监听事件，就可以在对应的时机进行一些额外的处理，开发者可以通过断点调试来查看一个事件是如何发生，以及如何通知监听器的。

通过阅读源码可以得知，一个事件实际上就是通过 `publishEvent` 方法来进行发布的，也可以自定义项目中的事件，并注册对应的监听器进行处理。

1. 自定义 event 事件

```java
public class TestEvent extends ApplicationEvent {   //需要继承 ApplicationEvent
    public TestEvent(Object source) {
        super(source);
    }
}
```

2. 编写监听器代码

```java
@Component
public class TestListener implements ApplicationListener<TestEvent> {
    @Override
    public void onApplicationEvent(TestEvent event) {
        System.out.println("自定义事件发生了："+event.getSource());
    }
}
```

3. 在控制器中发布事件

```java
import com.example.springbootstudy.listener.TestEvent;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.annotation.Resource;

@Controller
public class MainController {
    @Resource
    ApplicationContext context;

    @RequestMapping("/login")
    public String login() {
        context.publishEvent(new TestEvent("有人访问了登录界面！"));
        return "login";
    }
}
```

### Aware 系列接口

Spring 源码中，经常会发现某些类的定义上，除了当时需要的继承关系以外，还实现了一些接口，这些接口的名称基本都是 `xxxxAware`。<br>比如 SpringSecurity 的源码中，AbstractAuthenticationProcessingFilter 类就是这样：

```java
public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean implements ApplicationEventPublisherAware, MessageSourceAware {
    ...
    ...
    ...
}
```

它除了继承自 `GenericFilterBean` 之外，还实现了 `ApplicationEventPublisherAware` 和 `MessageSourceAware` 接口。

Aware 的中文意思为**感知**。简单来说，它就是一个标识，实现此接口的类会获得某些感知能力，Spring 容器会在 Bean 被加载时，根据类实现的感知接口，会调用类中实现的对应感知方法。

比如 `AbstractAuthenticationProcessingFilter` 就实现了 `ApplicationEventPublisherAware` 接口，此接口的感知功能为事件发布器，在 Bean 加载时，会调用实现类中的 `setApplicationEventPublisher` 方法，而 `AbstractAuthenticationProcessingFilter` 类则利用此方法，在 Bean 加载阶段获得了容器的事件发布器，以便之后发布事件使用。

```java
public void setApplicationEventPublisher(ApplicationEventPublisher eventPublisher) {
    this.eventPublisher = eventPublisher;   //直接存到成员变量
}
```

```java
protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
    SecurityContext context = SecurityContextHolder.createEmptyContext();
    context.setAuthentication(authResult);
    SecurityContextHolder.setContext(context);
    if (this.logger.isDebugEnabled()) {
        this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
    }

    this.rememberMeServices.loginSuccess(request, response, authResult);
  	//在这里使用
    if (this.eventPublisher != null) {
        this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
    }

    this.successHandler.onAuthenticationSuccess(request, response, authResult);
}
```

同样的，除了 ApplicationEventPublisherAware 接口外，再来看一个接口。比如：

```java
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.stereotype.Service;

@Service
public class TestService implements BeanNameAware {
    @Override
    public void setBeanName(String s) {
        System.out.println(s);
    }
}
```

其中 `BeanNameAware` 就是感知 Bean 名称的一个接口，当 Bean 被加载时，会调用 `setBeanName` 方法并将 Bean 名称作为参数传递。

## Spring Boot 实现原理

### 启动原理

SpringBoot 项目启动之后，做了什么事情。

SpringBoot 项目启动之后 SpringApplication 中的静态 `run` 方法：

```java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    return run(new Class[]{primarySource}, args);
}
```

套娃如下：

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return (new SpringApplication(primarySources)).run(args);	//这里直接 new 了一个新的 SpringApplication 对象，传入主类作为构造方法参数，并调用了非 static 的 run() 方法
}
```

来看看构造方法里面做了什么事情：

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.sources = new LinkedHashSet();
    this.bannerMode = Mode.CONSOLE;
    this.logStartupInfo = true;
    this.addCommandLineProperties = true;
    this.addConversionService = true;
    this.headless = true;
    this.registerShutdownHook = true;
    this.additionalProfiles = Collections.emptySet();
    this.isCustomEnvironment = false;
    this.lazyInitialization = false;
    this.applicationContextFactory = ApplicationContextFactory.DEFAULT;
    this.applicationStartup = ApplicationStartup.DEFAULT;
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
  	//这里是关键，这里会判断当前 SpringBoot 应用程序是否为 Web 项目，并返回当前的项目类型
    this.webApplicationType = WebApplicationType.deduceFromClasspath();	//deduceFromClasspath 是根据类路径下判断是否包含 SpringBootWeb 依赖，如果不包含就是 NONE 类型，包含就是 SERVLET 类型
    this.bootstrapRegistryInitializers = new ArrayList(this.getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
    this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));	//创建所有 ApplicationContextInitializer 实现类的对象
    this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = this.deduceMainApplicationClass();
}
```

关键就在这里了，它是如何知道哪些类是 `ApplicationContextInitializer` 的实现类的呢？

这里就要提到 spring.factories 了，它是 Spring 仿造 Java SPI 实现的一种类加载机制。它在 META-INF/spring.factories 文件中配置接口的实现类名称，然后在程序中读取这些配置文件并实例化。这种自定义的 SPI 机制是 Spring Boot Starter 实现的基础。

SPI 的常见例子：

- 数据库驱动加载接口实现类的加载：JDBC 加载不同类型数据库的驱动
- 日志门面接口实现类加载：SLF4J 加载不同提供商的日志实现类

说白了就是人家定义接口，但是实现可能有很多种，但是核心只提供接口，需要我们按需选择对应的实现，这种方式是高度解耦的。

我们来看看 `getSpringFactoriesInstances` 方法做了什么：

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
  	//获取当前的类加载器
    ClassLoader classLoader = this.getClassLoader();
  	//获取所有依赖中 META-INF/spring.factories 中配置的对应接口类的实现类列表
    Set<String> names = new LinkedHashSet(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
  	//根据上方列表，依次创建实例对象  
  List<T> instances = this.createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
  	//根据对应类上的Order接口或是注解进行排序
    AnnotationAwareOrderComparator.sort(instances);
  	//返回实例
    return instances;
}
```

其中`SpringFactoriesLoader.loadFactoryNames`正是读取配置的核心部分，我们后面还会遇到。

接着我们来看run方法里面做了什么事情。

```java
public ConfigurableApplicationContext run(String... args) {
    long startTime = System.nanoTime();
    DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
    ConfigurableApplicationContext context = null;
    this.configureHeadlessProperty();
  	//获取所有的SpringApplicationRunListener，并通知启动事件，默认只有一个实现类EventPublishingRunListener
  	//EventPublishingRunListener会将初始化各个阶段的事件转发给所有监听器
    SpringApplicationRunListeners listeners = this.getRunListeners(args);
    listeners.starting(bootstrapContext, this.mainApplicationClass);

    try {
      	//环境配置
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        this.configureIgnoreBeanInfo(environment);
      	//打印Banner
        Banner printedBanner = this.printBanner(environment);
      	//创建ApplicationContext，注意这里会根据是否为Web容器使用不同的ApplicationContext实现类
        context = this.createApplicationContext();
        context.setApplicationStartup(this.applicationStartup);
      	//初始化ApplicationContext
        this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
      	//执行ApplicationContext的refresh方法
        this.refreshContext(context);
        this.afterRefresh(context, applicationArguments);
        Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
        if (this.logStartupInfo) {
            (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), timeTakenToStartup);
        }
        ....
}
```

我们发现，实际上SpringBoot就是Spring的一层壳罢了，离不开最关键的ApplicationContext，也就是说，在启动后会自动配置一个ApplicationContext，只不过是进行了大量的扩展。

我们来看ApplicationContext是怎么来的，打开`createApplicationContext`方法：

```java
protected ConfigurableApplicationContext createApplicationContext() {
    return this.applicationContextFactory.create(this.webApplicationType);
}
```

我们发现在构造方法中`applicationContextFactory`直接使用的是DEFAULT：

```java
this.applicationContextFactory = ApplicationContextFactory.DEFAULT;
```

```java
ApplicationContextFactory DEFAULT = (webApplicationType) -> {
    try {
        switch(webApplicationType) {
        case SERVLET:
            return new AnnotationConfigServletWebServerApplicationContext();
        case REACTIVE:
            return new AnnotationConfigReactiveWebServerApplicationContext();
        default:
            return new AnnotationConfigApplicationContext();
        }
    } catch (Exception var2) {
        throw new IllegalStateException("Unable create a default ApplicationContext instance, you may need a custom ApplicationContextFactory", var2);
    }
};

ConfigurableApplicationContext create(WebApplicationType webApplicationType);
```

DEFAULT是直接编写的一个匿名内部类，其实已经很明确了，正是根据`webApplicationType`类型进行判断，如果是SERVLET，那么久返回专用于Web环境的AnnotationConfigServletWebServerApplicationContext对象（SpringBoot中新增的），否则返回普通的AnnotationConfigApplicationContext对象，也就是到这里为止，Spring的容器就基本已经确定了。

注意AnnotationConfigApplicationContext是Spring框架提供的类，从这里开始相当于我们在讲Spring的底层源码了，我们继续深入，AnnotationConfigApplicationContext对象在创建过程中会创建`AnnotatedBeanDefinitionReader`，它是用于通过注解解析Bean定义的工具类：

```java
public AnnotationConfigApplicationContext() {
    StartupStep createAnnotatedBeanDefReader = this.getApplicationStartup().start("spring.context.annotated-bean-reader.create");
    this.reader = new AnnotatedBeanDefinitionReader(this);
    createAnnotatedBeanDefReader.end();
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

其构造方法：

```java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
    ...
    //这里会注册很多的后置处理器
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

```java
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(BeanDefinitionRegistry registry, @Nullable Object source) {
    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
    ....
    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet(8);
    RootBeanDefinition def;
    if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalConfigurationAnnotationProcessor")) {
      	//注册了ConfigurationClassPostProcessor用于处理@Configuration、@Import等注解
      	//注意这里是关键，之后Selector还要讲到它
      	//它是继承自BeanDefinitionRegistryPostProcessor，所以它的执行时间在Bean定义加载完成后，Bean初始化之前
        def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalConfigurationAnnotationProcessor"));
    }

    if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalAutowiredAnnotationProcessor")) {
      	//AutowiredAnnotationBeanPostProcessor用于处理@Value等注解自动注入
        def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalAutowiredAnnotationProcessor"));
    }
  
  	...
```

回到SpringBoot，我们最后来看，`prepareContext`方法中又做了什么事情：

```java
private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context, ConfigurableEnvironment environment, SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
  	//环境配置
    context.setEnvironment(environment);
    this.postProcessApplicationContext(context);
    this.applyInitializers(context);
    listeners.contextPrepared(context);
    bootstrapContext.close(context);
    if (this.logStartupInfo) {
        this.logStartupInfo(context.getParent() == null);
        this.logStartupProfileInfo(context);
    }

  	//将Banner注册为Bean
    ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
    beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
    if (printedBanner != null) {
        beanFactory.registerSingleton("springBootBanner", printedBanner);
    }

    if (beanFactory instanceof AbstractAutowireCapableBeanFactory) {
        ((AbstractAutowireCapableBeanFactory)beanFactory).setAllowCircularReferences(this.allowCircularReferences);
        if (beanFactory instanceof DefaultListableBeanFactory) {
            ((DefaultListableBeanFactory)beanFactory).setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
        }
    }

    if (this.lazyInitialization) {
        context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
    }

  	//这里会获取我们一开始传入的项目主类
    Set<Object> sources = this.getAllSources();
    Assert.notEmpty(sources, "Sources must not be empty");
  	//这里会将我们的主类直接注册为Bean，这样就可以通过注解加载了
    this.load(context, sources.toArray(new Object[0]));
    listeners.contextLoaded(context);
}
```

因此，在`prepareContext`执行完成之后，我们的主类成功完成Bean注册，接下来，就该类上注解大显身手了。

### 启动配置原理

既然主类已经在初始阶段注册为Bean，那么在加载时，就会根据注解定义，进行更多的额外操作。所以我们来看看主类上的`@SpringBootApplication`注解做了什么事情。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

我们发现，`@SpringBootApplication`上添加了`@ComponentScan`注解，此注解我们此前已经认识过了，但是这里并没有配置具体扫描的包，因此它会自动将声明此接口的类所在的包作为basePackage，因此当添加`@SpringBootApplication`之后也就等于直接开启了自动扫描，但是一定注意不能在主类之外的包进行Bean定义，否则无法扫描到，需要手动配置。

接着我们来看第二个注解`@EnableAutoConfiguration`，它就是自动配置的核心了，我们来看看它是如何定义的：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```

老套路了，直接一手`@Import`，通过这种方式来将一些外部的Bean加载到容器中。我们来看看AutoConfigurationImportSelector做了什么事情：

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
		...
}
```

我们看到它实现了很多接口，包括大量的Aware接口，实际上就是为了感知某些必要的对象，并将其存到当前类中。

其中最核心的是`DeferredImportSelector`接口，它是`ImportSelector`的子类，它定义了`selectImports`方法，用于返回需要加载的类名称，在Spring加载ImportSelector类型的Bean时，会调用此方法来获取更多需要加载的类，并将这些类一并注册为Bean：

```java
public interface ImportSelector {
    String[] selectImports(AnnotationMetadata importingClassMetadata);

    @Nullable
    default Predicate<String> getExclusionFilter() {
        return null;
    }
}
```

到目前为止，我们了解了两种使用`@Import`有特殊机制的接口：ImportSelector（这里用到的）和ImportBeanDefinitionRegistrar（之前Mybatis-spring源码有讲）当然还有普通的`@Configuration`配置类。

我们可以来阅读一下`ConfigurationClassPostProcessor`的源码，看看它到底是如何处理`@Import`的：

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    List<BeanDefinitionHolder> configCandidates = new ArrayList();
  	//注意这个阶段仅仅是已经完成扫描了所有的Bean，得到了所有的BeanDefinition，但是还没有进行任何区分
  	//candidate是候选者的意思，一会会将标记了@Configuration的类作为ConfigurationClass加入到configCandidates中
    String[] candidateNames = registry.getBeanDefinitionNames();
    String[] var4 = candidateNames;
    int var5 = candidateNames.length;

    for(int var6 = 0; var6 < var5; ++var6) {
        String beanName = var4[var6];
        BeanDefinition beanDef = registry.getBeanDefinition(beanName);
        if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
            }
        } else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {   //判断是否添加了@Configuration注解
            configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
        }
    }

    if (!configCandidates.isEmpty()) {
        //...省略

      	//这里创建了一个ConfigurationClassParser用于解析配置类
        ConfigurationClassParser parser = new ConfigurationClassParser(this.metadataReaderFactory, this.problemReporter, this.environment, this.resourceLoader, this.componentScanBeanNameGenerator, registry);
      	//所有配置类的BeanDefinitionHolder列表
        Set<BeanDefinitionHolder> candidates = new LinkedHashSet(configCandidates);
      	//已经解析完成的类
        HashSet alreadyParsed = new HashSet(configCandidates.size());

        do {
            //这里省略，直到所有的配置类全部解析完成
          	//注意在循环过程中可能会由于@Import新增更多的待解析配置类，一律丢进candidates集合中
        } while(!candidates.isEmpty());

        ...

    }
}
```

我们接着来看，`ConfigurationClassParser`是如何进行解析的：

```java
protected void processConfigurationClass(ConfigurationClass configClass, Predicate<String> filter) throws IOException {
  	//@Conditional相关注解处理
  	//后面会讲
    if (!this.conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.PARSE_CONFIGURATION)) {
        ...
        }

        ConfigurationClassParser.SourceClass sourceClass = this.asSourceClass(configClass, filter);

        do {
          	//核心
            sourceClass = this.doProcessConfigurationClass(configClass, sourceClass, filter);
        } while(sourceClass != null);

        this.configurationClasses.put(configClass, configClass);
    }
}
```

最后我们再来看最核心的`doProcessConfigurationClass`方法：

```java
protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)
    ...

    processImports(configClass, sourceClass, getImports(sourceClass), true);    // 处理Import注解

		...

    return null;
}
```

```java
private void processImports(ConfigurationClass configClass, ConfigurationClassParser.SourceClass currentSourceClass, Collection<ConfigurationClassParser.SourceClass> importCandidates, Predicate<String> exclusionFilter, boolean checkForCircularImports) {
    if (!importCandidates.isEmpty()) {
        if (checkForCircularImports && this.isChainedImportOnStack(configClass)) {
            this.problemReporter.error(new ConfigurationClassParser.CircularImportProblem(configClass, this.importStack));
        } else {
            this.importStack.push(configClass);

            try {
                Iterator var6 = importCandidates.iterator();

                while(var6.hasNext()) {
                    ConfigurationClassParser.SourceClass candidate = (ConfigurationClassParser.SourceClass)var6.next();
                    Class candidateClass;
                  	//如果是ImportSelector类型，继续进行运行
                    if (candidate.isAssignable(ImportSelector.class)) {
                        candidateClass = candidate.loadClass();
                        ImportSelector selector = (ImportSelector)ParserStrategyUtils.instantiateClass(candidateClass, ImportSelector.class, this.environment, this.resourceLoader, this.registry);
                        Predicate<String> selectorFilter = selector.getExclusionFilter();
                        if (selectorFilter != null) {
                            exclusionFilter = exclusionFilter.or(selectorFilter);
                        }
									//如果是DeferredImportSelector的实现类，那么会走deferredImportSelectorHandler的handle方法
                        if (selector instanceof DeferredImportSelector) {
                            this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector)selector);
                          //否则就按照正常的ImportSelector类型进行加载
                        } else {
                          	//调用selectImports方法获取所有需要加载的类
                            String[] importClassNames = selector.selectImports(currentSourceClass.getMetadata());
                            Collection<ConfigurationClassParser.SourceClass> importSourceClasses = this.asSourceClasses(importClassNames, exclusionFilter);
                          	//递归处理，直到没有
                            this.processImports(configClass, currentSourceClass, importSourceClasses, exclusionFilter, false);
                        }
                      //判断是否为ImportBeanDefinitionRegistrar类型
                    } else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {
                        candidateClass = candidate.loadClass();
                        ImportBeanDefinitionRegistrar registrar = (ImportBeanDefinitionRegistrar)ParserStrategyUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class, this.environment, this.resourceLoader, this.registry);
                      	//往configClass丢ImportBeanDefinitionRegistrar信息进去，之后再处理
                        configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
                      //否则按普通的配置类进行处理
                    } else {
                        this.importStack.registerImport(currentSourceClass.getMetadata(), candidate.getMetadata().getClassName());
                        this.processConfigurationClass(candidate.asConfigClass(configClass), exclusionFilter);
                    }
                }
            } catch (BeanDefinitionStoreException var17) {
                throw var17;
            } catch (Throwable var18) {
                throw new BeanDefinitionStoreException("Failed to process import candidates for configuration class [" + configClass.getMetadata().getClassName() + "]", var18);
            } finally {
                this.importStack.pop();
            }
        }

    }
}
```

不难注意到，虽然这里额外处理了`ImportSelector`对象，但是还针对`ImportSelector`的子接口`DeferredImportSelector`进行了额外处理，Deferred是延迟的意思，它是一个延迟执行的`ImportSelector`，并不会立即进处理，而是丢进DeferredImportSelectorHandler，并且在`parse`方法的最后进行处理：

```java
public void parse(Set<BeanDefinitionHolder> configCandidates) {
     ...

    this.deferredImportSelectorHandler.process();
}
```

我们接着来看`DeferredImportSelector`正好就有一个`process`方法：

```java
public interface DeferredImportSelector extends ImportSelector {
    @Nullable
    default Class<? extends DeferredImportSelector.Group> getImportGroup() {
        return null;
    }

    public interface Group {
        void process(AnnotationMetadata metadata, DeferredImportSelector selector);

        Iterable<DeferredImportSelector.Group.Entry> selectImports();

        public static class Entry {
          ...
```

最后经过ConfigurationClassParser处理完成后，通过`parser.getConfigurationClasses()`就能得到通过配置类导入了哪些额外的配置类。最后将这些配置类全部注册BeanDefinition，然后就可以交给接下来的Bean初始化过程去处理了。

```java
this.reader.loadBeanDefinitions(configClasses);
```

最后我们再去看`loadBeanDefinitions`是如何运行的：

```java
public void loadBeanDefinitions(Set<ConfigurationClass> configurationModel) {
    ConfigurationClassBeanDefinitionReader.TrackedConditionEvaluator trackedConditionEvaluator = new ConfigurationClassBeanDefinitionReader.TrackedConditionEvaluator();
    Iterator var3 = configurationModel.iterator();

    while(var3.hasNext()) {
        ConfigurationClass configClass = (ConfigurationClass)var3.next();
        this.loadBeanDefinitionsForConfigurationClass(configClass, trackedConditionEvaluator);
    }

}

private void loadBeanDefinitionsForConfigurationClass(ConfigurationClass configClass, ConfigurationClassBeanDefinitionReader.TrackedConditionEvaluator trackedConditionEvaluator) {
    if (trackedConditionEvaluator.shouldSkip(configClass)) {
        String beanName = configClass.getBeanName();
        if (StringUtils.hasLength(beanName) && this.registry.containsBeanDefinition(beanName)) {
            this.registry.removeBeanDefinition(beanName);
        }

        this.importRegistry.removeImportingClass(configClass.getMetadata().getClassName());
    } else {
        if (configClass.isImported()) {
            this.registerBeanDefinitionForImportedConfigurationClass(configClass);  //注册配置类自己
        }

        Iterator var3 = configClass.getBeanMethods().iterator();

        while(var3.hasNext()) {
            BeanMethod beanMethod = (BeanMethod)var3.next();
            this.loadBeanDefinitionsForBeanMethod(beanMethod); //注册@Bean注解标识的方法
        }

      	//注册`@ImportResource`引入的XML配置文件中读取的bean定义
        this.loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());
     	 //注册configClass中经过解析后保存的所有ImportBeanDefinitionRegistrar，注册对应的BeanDefinition
        this.loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());
    }
}
```

这样，整个`@Configuration`配置类的底层配置流程我们就大致了解了。接着我们来看AutoConfigurationImportSelector是如何实现自动配置的，可以看到内部类AutoConfigurationGroup的process方法，它是父接口的实现，因为父接口是`DeferredImportSelector`，那么很容易得知，实际上最后会调用`process`方法获取所有的自动配置类：

```java
public void process(AnnotationMetadata annotationMetadata, DeferredImportSelector deferredImportSelector) {
    Assert.state(deferredImportSelector instanceof AutoConfigurationImportSelector, () -> {
        return String.format("Only %s implementations are supported, got %s", AutoConfigurationImportSelector.class.getSimpleName(), deferredImportSelector.getClass().getName());
    });
  	//获取所有的Entry，其实就是，读取spring.factories来查看有哪些自动配置类
    AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = ((AutoConfigurationImportSelector)deferredImportSelector).getAutoConfigurationEntry(annotationMetadata);
    this.autoConfigurationEntries.add(autoConfigurationEntry);
    Iterator var4 = autoConfigurationEntry.getConfigurations().iterator();

    while(var4.hasNext()) {
        String importClassName = (String)var4.next();
        this.entries.putIfAbsent(importClassName, annotationMetadata);
    }

}
```

我们接着来看`getAutoConfigurationEntry`方法：

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
  	//判断是否开启了自动配置，是的，自动配置可以关
    if (!this.isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    } else {
      	//根据注解定义获取一些属性
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
      	//得到spring.factories文件中所有需要自动配置的类
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        ... 这里先看前半部分
    }
}
```

注意这里并不是spring.factories文件中所有的自动配置类都会被加载，它会根据@Condition注解的条件进行加载。这样就能实现我们需要什么模块添加对应依赖就可以实现自动配置了。

所有的源码看不懂，都源自于你的心中没有形成一个完整的闭环！一旦一条线推到头，闭环形成，所有疑惑迎刃而解。

### 自定义Starter

我们仿照Mybatis来编写一个自己的starter，Mybatis的starter包含两个部分：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot</artifactId>
    <version>2.2.0</version>
  </parent>
  <!--  starter本身只做依赖集中管理，不编写任何代码  -->
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <name>mybatis-spring-boot-starter</name>
  <properties>
    <module.name>org.mybatis.spring.boot.starter</module.name>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <!--  编写的专用配置模块   -->
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
    </dependency>
  </dependencies>
</project>
```

因此我们也将我们自己的starter这样设计：

我们设计三个模块：

* spring-boot-hello：基础业务功能模块
* spring-boot-starter-hello：启动器
* spring-boot-autoconifgurer-hello：自动配置依赖

首先是基础业务功能模块，这里我们随便创建一个类就可以了：

```java
public class HelloWorldService {
	
}
```

启动器主要做依赖管理，这里就不写任何代码，只写pom文件：

```xml
<dependencies>
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>spring-boot-autoconfigurer-hello</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

导入autoconfigurer模块作为依赖即可，接着我们去编写autoconfigurer模块，首先导入依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
        <version>2.6.2</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <version>2.6.2</version>
        <optional>true</optional>
    </dependency>
    
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>spring-boot-hello</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

接着创建一个HelloWorldAutoConfiguration作为自动配置类：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication
@ConditionalOnClass(HelloWorldService.class)
@EnableConfigurationProperties(HelloWorldProperties.class)
public class HelloWorldAutoConfiguration {

    Logger logger = Logger.getLogger(this.getClass().getName());

    @Resource
    HelloWorldProperties properties;

    @Bean
    public HelloWorldService helloWorldService(){
        logger.info("自定义starter项目已启动！");
        logger.info("读取到自定义配置："+properties.getValue());
        return new HelloWorldService();
    }
}
```

对应的配置读取类：

```java
@ConfigurationProperties("hello.world")
public class HelloWorldProperties {

    private String value;

    public void setValue(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}
```

最后再编写`spring.factories`文件，并将我们的自动配置类添加即可：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.hello.autoconfigurer.HelloWorldAutoConfiguration
```

最后再Maven根项目执行`install`安装到本地仓库，完成。接着就可以在其他项目中使用我们编写的自定义starter了。

### Runner接口

在项目中，可能会遇到这样一个问题：我们需要在项目启动完成之后，紧接着执行一段代码。

我们可以编写自定义的ApplicationRunner来解决，它会在项目启动完成后执行：

```java
@Component
public class TestRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("我是自定义执行！");
    }
}
```

当然也可以使用CommandLineRunner，它也支持使用@Order或是实现Ordered接口来支持优先级执行。

实际上它就是run方法的最后：

```java
public ConfigurableApplicationContext run(String... args) {
    ....

        listeners.started(context, timeTakenToStartup);
  			//这里已经完成整个SpringBoot项目启动，所以执行所有的Runner
        this.callRunners(context, applicationArguments);
    } catch (Throwable var12) {
        this.handleRunFailure(context, var12, listeners);
        throw new IllegalStateException(var12);
    }

    try {
        Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
        listeners.ready(context, timeTakenToReady);
        return context;
    } catch (Throwable var11) {
        this.handleRunFailure(context, var11, (SpringApplicationRunListeners)null);
        throw new IllegalStateException(var11);
    }
}
```

官网：https://spring.io/projects/spring-boot

> Spring Boot 可以轻松创建独立的、基于 Spring 的生产级应用程序，它可以让你“运行即可”。大多数 Spring Boot 应用程序只需要少量的 Spring 配置。

SpringBoot 功能：

- 创建独立的 Spring 应用程序
- 直接嵌入 Tomcat、Jetty 或 Undertow（无需部署 WAR 包，打包成 Jar 本身就是一个可以运行的应用程序）
- 提供一站式的 `starter` 依赖项，以简化 Maven 配置（需要整合什么框架，直接导对应框架的 starter 依赖）
- 尽可能自动配置 Spring 和第三方库（除非特殊情况，否则几乎不需要你进行什么配置）
- 提供生产就绪功能，如指标、运行状况检查和外部化配置
- 没有代码生成，也没有 XML 配置的要求

SpringBoot 是现在最主流的开发框架，它提供了一站式的开发体验，大幅度提高了我们的开发效率。

## 走进 SpringBoot

在 SSM 阶段，当需要搭建一个基于 Spring 全家桶的 Web 应用程序时，不得不做大量的依赖导入和框架整合相关的 Bean 定义，光是整合框架就花费了大量的时间，但是实际上发现，整合框架其实基本都是一些固定流程，每创建一个新的 Web 应用程序，基本都会使用同样的方式去整合框架，实际完全可以将一些重复的配置作为约定，只要框架遵守这个约定，提供默认的配置就好，这样就不用再去配置了，约定优于配置！<br>
而 SpringBoot 正是将这些过程大幅度进行了简化，它可以自动进行配置，开发者只需要导入对应的启动器（starter）依赖即可。

## 初始项目文件结构

在创建 SpringBoot 项目之后，首先会自动生成一个主类，而主类中的 `main` 方法中调用了 `SpringApplication` 类的静态方法来启动整个 SpringBoot 项目，并且主类的上方有一个 `@SpringBootApplication` 注解：

```java
@SpringBootApplication
public class SpringBootTestApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootTestApplication.class, args);
    }

}
```

- 测试类，测试类的上方仅添加了一个 `@SpringBootTest` 注解：

```java
@SpringBootTest
class SpringBootTestApplicationTests {
    @Test
    void contextLoads() {}
}
```

- pom.xml 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!-- 父工程 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>SpringBootStudy</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>SpringBootStudy</name>
    <description>SpringBootStudy</description>
    <properties>
        <java.version>8</java.version>
    </properties>
    <dependencies>
        <!-- spring-boot-starter：SpringBoot 核心启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!-- spring-boot-starter-test：SpringBoot 测试模块启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <!-- SpringBoot Maven插件 -->
        <plugins>
            <!-- 打包Jar -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

- application.properties：SpringBoot 的配置文件，所有依赖的配置都在这里编写，但是一般情况下只需要配置必要项即可。

## 整合 Web 相关框架

先导入依赖

```xml
<!-- spring-boot-starter-test：SpringBoot web 模块启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- Lombok 是一个 Java 库，能自动插入编辑器并构建工具，简化 Java 开发。通过添加注解的方式，不需要为类编写 getter() 或 equals() 方法，同时可以自动化日志变量。 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

完成后，直接运行项目主类，就可以直接访问：http://localhost:8080/<br>
由于没有编写任何的请求映射，所以没有数据。

![初次运行 Spring Boot 项目](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%2520%25E6%258A%2580%25E6%259C%25AF/%25E6%25A1%2586%25E6%259E%25B6/Spring%2520Boot/%25E5%2588%259D%25E6%25AC%25A1%25E8%25BF%2590%25E8%25A1%258C%2520Spring%2520Boot%2520%25E9%25A1%25B9%25E7%259B%25AE.png)

Spring Boot 日志中除了最基本的 SpringBoot 启动日志以外，还新增了内嵌 Web 服务器（Tomcat）的启动日志，并且显示了当前 Web 服务器所开放的端口，并且自动帮助初始化了DispatcherServlet，但是只创建了项目，导入了 web 相关的 starter 依赖，没有进行任何的配置，实际上它使用的是 starter 提供的默认配置进行初始化的。

由于SpringBoot是自动扫描的，因此直接创建一个Controller即可被加载：

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class MainController {
    //直接访问 http://localhost:8080/ 即可，不用加 web 应用程序名称了
    @RequestMapping("/")
    @ResponseBody
    public String index() {
        return "Hello World";
    }
}
```

它还可以自动识别类型，如果返回的是一个对象类型的数据，那么它会自动转换为JSON数据格式，无需配置：

```java
import lombok.Data;

@Data
public class Student {
    int studentID;
    String name;
    String sex;
}
```

```java
@RequestMapping("/student")
@ResponseBody
public Student student(){
    Student student = new Student();
    student.setName("小明");
    student.setSex("男");
    student.setSid(10);
    return student;
}
```

最后浏览器能够直接得到 `application/json` 的响应数据。

![JSON 响应对象数据](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%2520%25E6%258A%2580%25E6%259C%25AF/%25E6%25A1%2586%25E6%259E%25B6/Spring%2520Boot/JSON%2520%25E5%2593%258D%25E5%25BA%2594%25E5%25AF%25B9%25E8%25B1%25A1%25E6%2595%25B0%25E6%258D%25AE.png)

### 修改 Web 相关配置

如果需要修改 Web 服务器的端口或是一些其他的内容，可以直接在 `application.properties` 中进行修改，它是整个 SpringBoot 的配置文件：

```properties
# 修改端口为80
server.port=80
```

还可以编写自定义的配置项，并在项目中通过`@Value`直接注入：

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class MainController {
    @Value("${test.data}")
    int data;

    @RequestMapping("/")
    @ResponseBody
    public String index() {
        return "Hello World" + data;
    }
}
```

```properties
test.data=100
```

![自定义配置项](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%2520%25E6%258A%2580%25E6%259C%25AF/%25E6%25A1%2586%25E6%259E%25B6/Spring%2520Boot/%25E8%2587%25AA%25E5%25AE%259A%25E4%25B9%2589%25E9%2585%258D%25E7%25BD%25AE%25E9%25A1%25B9.png)

通过这种方式，就可以更好地将一些需要频繁修改的配置项写在配置文件中，并通过注解方式去获取值。

配置文件除了使用 `properties` 格式以外，还有一种叫做 `yaml` 格式，它的语法如下：

```yaml
一级目录:
	二级目录:
	  三级目录1: 值
	  三级目录2: 值
	  三级目录List: 
	  - 元素1
	  - 元素2
	  - 元素3
```

每一级目录都是通过缩进（不能使用Tab，只能使用空格）区分，并且键和值之间需要添加 `冒号 + 空格` 来表示。

SpringBoot 也支持这种格式的配置文件，可以将 `application.properties` 修改为`application.yml` 或是 `application.yaml` 来使用 YAML 语法编写配置：

```yaml
server:
  port: 80
```

## 整合 Spring Security

导入 Spring Boot 项目 Spring Security 的 starter 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

导入依赖后，直接启动 SpringBoot 应用程序，SpringSecurity 会自动生成一个默认用户`user`，它的密码会出现在日志中：

```
2023-03-12 17:58:13.762  INFO 11536 --- [           main] .s.s.UserDetailsServiceAutoConfiguration : 

Using generated security password: 81638855-cf00-4c0a-b653-c1b288fc6bf1
```

其中`81638855-cf00-4c0a-b653-c1b288fc6bf1`就是随机生成的一个密码，我们可以使用用户名：`user` 和此密码在 security 自动生成的登录页面完成登录。

![security 生成的登录页面](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%2520%25E6%258A%2580%25E6%259C%25AF/%25E6%25A1%2586%25E6%259E%25B6/Spring%2520Boot/Security%2520%25E7%2594%259F%25E6%2588%2590%25E7%259A%2584%25E7%2599%25BB%25E5%25BD%2595%25E9%25A1%25B5%25E9%259D%25A2.png)

也可以在配置文件中直接配置：

```yaml
spring:
  security:
    user:
      name: test   # 用户名
      password: 123456  # 密码
      roles:   # 角色
      - user
      - admin
```

实际上这样的配置方式就是一个 `inMemoryAuthentication`，只是可以直接配置而已。

当然，页面的控制和数据库验证还是需要提供 `WebSecurityConfigurerAdapter` 的实现类去完成：

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/login").permitAll()
            .anyRequest().hasRole("user")
            .and()
            .formLogin();
    }
}
```

注意这里不需要再添加 `@EnableWebSecurity` 了，因为 starter 依赖已经帮我们添加了。

## 整合 MyBatis

同样的，只需要导入对应的 starter 依赖即可：

```xml
<!-- SpringBoot 整合 mybatis 模块 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.0</version>
</dependency>
<!-- 连接 MySQL 的驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

导入依赖后，直接启动会报错，是因为有必要的配置我们没有去编写，需要指定数据源的相关信息：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/数据库名
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

再次启动，成功。

Spring Boot 整合 MyBatis 后，不需要再去设置 MyBatis 的相关 Bean 了，也不需要添加任何 `@MapperSacn` 注解，因为 starter 已经做好了，它会自动扫描项目中添加了 `@Mapper` 注解的接口，直接将其注册为 Bean，不需要进行任何配置。

```java
import lombok.Data;

@Data
public class UserData {
    Integer id;
    String username;
    String password;
    String role;
}
```

```java
@Mapper
public interface MainMapper {
    @Select("select * from users where username = #{username}")
    UserData findUserByName(String username);
}
```

当然也可以直接 `@MapperScan` 使用包扫描。

```java
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("xxx.xxx.mapper")
public class SpringBootStudyApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootStudyApplication.class, args);
    }
}
```

添加Mapper之后，使用方法和 SSM 阶段是一样的，可以将其与 Spring Security 结合使用：

```java
import com.example.springbootstudy.enity.UserData;
import com.example.springbootstudy.mapper.MainMapper;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
public class UserAuthService implements UserDetailsService {
    @Resource
    MainMapper mapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserData data = mapper.findUserByName(username);
        if(data == null) throw new UsernameNotFoundException("用户 "+username+" 登录失败，用户名不存在！");
        return User
                .withUsername(data.getUsername())
                .password(data.getPassword())
                .roles(data.getRole())
                .build();
    }
}
```

最后在 SecurityConfiguration.java 配置一下自定义验证即可。

> <font color="green">提示</font>
>
> 自定义验证以后，那么之前在 application.yml 配置文件里面配置的 Spring Security 用户就失效了。

```java
import com.example.springbootstudy.service.UserAuthService;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

import javax.annotation.Resource;

@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Resource
    UserAuthService userAuthService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/login").permitAll()
                // .anyRequest().hasRole("user")    只有 user 角色可以通过验证
                .anyRequest().hasanyRole("admin", "user")   //amdin 和 user 角色都可以通过验证
                .and()
                .formLogin();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .userDetailsService(userAuthService)
                .passwordEncoder(new BCryptPasswordEncoder());
    }
}
```

> 日志系统如若出现此系统 `Encoded password does not look like BCrypt`，说明数据库中的密码未加密，需要数据库中的密码是加密后的结果。
>
> 可用此方法获取字段加密后的结果，然后手动录入数据库。
>
> ```java
> @Test
> public void test(){
> BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
> System.out.println(encoder.encode("user")); //输入想要加密的 password
> }
> ```

在首次使用时，会发现日志中输出以以下语句：

```
2023-03-12 21:01:26.361  INFO 16396 --- [nio-8080-exec-4] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2023-03-12 21:01:26.871  INFO 16396 --- [nio-8080-exec-4] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
```

实际上，SpringBoot 会自动为 Mybatis 配置数据源，默认使用的就是 `HikariCP` 数据源。

## 整合 Thymeleaf

整合 Thymeleaf 也只需导入对应的 starter 即可：

```xml
<!-- spring boot 整合 thymeleaf -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

接着直接使用即可：

```java
@RequestMapping("/login")
public String login() {
    return "login";
}
```

> <font color="blue">注意</font>
>
> 这样只能正常解析 HTML 页面，但是 js、css 等静态资源需要进行路径指定，不然无法访问。

在 application.yml 配置文件中配置静态资源的访问前缀：

```yaml
spring:
	mvc:
  	static-path-pattern: /static/**
```

接着把登陆页面实现一下吧。

记得在 html 页面前缀加上 Thymeleaf 空间

```html
<html lang="en" xmlns:th=http://www.thymeleaf.org
xmlns:sec=http://www.thymeleaf.org/extras/spring-security>
```

SecurityConfiguration.java

```java
import com.example.springbootstudy.service.UserAuthService;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl;

import javax.annotation.Resource;
import javax.sql.DataSource;

@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Resource
    UserAuthService userAuthService;
    @Resource
    DataSource dataSource;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/static/**").permitAll()
                .anyRequest().hasAnyRole("user", "admin")
                .and()
                .formLogin()
                .loginPage("/login")
                .loginProcessingUrl("/doLogin")
                .permitAll()
                .defaultSuccessUrl("/index",true)
                .and()
                .rememberMe()
                .tokenRepository(new JdbcTokenRepositoryImpl(){{setDataSource(dataSource);}});
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .userDetailsService(userAuthService)
                .passwordEncoder(new BCryptPasswordEncoder());
    }
}
```

## 日志系统

### 日志门面和日志实现

日志实现：如 JUL（Java util logging，Java 原生日志框架）。可以直接使用JUL为日志框架来规范化打印日志

日志门面（Facade）：如 Slf4j（simple logging facade for java），是把不同的日志系统的实现进行了具体的抽象化，只提供了统一的日志使用接口，使用时只需要按照其提供的接口方法进行调用即可。由于它只是一个接口，并不是一个具体的可以直接单独使用的日志框架，所以最终日志的格式、记录级别、输出方式等都要通过接口绑定的具体的日志系统来实现。这些具体的日志系统就有 log4j、logback、java.util.logging 等，它们才实现了具体的日志系统的功能

日志门面和日志实现就像JDBC和数据库驱动一样，一个是画大饼的，一个是真的去做饼的。

![SLF4J](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%2520%25E6%258A%2580%25E6%259C%25AF/%25E6%25A1%2586%25E6%259E%25B6/Spring%2520Boot/SLF4J.png)

不同的框架可能使用了不同的日志框架，希望所有的框架一律使用日志门面（Slf4j）进行日志打印。<br>可以采取类似于偷梁换柱的做法，只保留不同日志框架的接口和类定义等关键信息，而将实现全部定向为 Slf4j 调用。相当于有着和原有日志框架一样的外壳，对于其他框架来说依然可以使用对应的类进行操作，而具体如何执行，真正的内心已经是Slf4j的了。

![SLF4J 调用其他日志实现](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%2520%25E6%258A%2580%25E6%259C%25AF/%25E6%25A1%2586%25E6%259E%25B6/Spring%2520Boot/SLF4J%2520%25E8%25B0%2583%25E7%2594%25A8%25E5%2585%25B6%25E4%25BB%2596%25E6%2597%25A5%25E5%25BF%2597%25E5%25AE%259E%25E7%258E%25B0.png)

SpringBoot 为了统一日志框架的使用，做了这些事情：

* 直接将其他依赖以前的日志框架剔除
* 导入对应日志框架的Slf4j中间包
* 导入自己官方指定的日志实现，并作为 Slf4j 的日志实现层

### 在SpringBoot中打印日志信息

SpringBoot 使用的是 Slf4j 作为日志门面，Logback【[Logback](http://logback.qos.ch/) 是 log4j 框架的作者开发的新一代日志框架，它效率更高、能够适应诸多的运行环境，同时天然支持 SLF4J】作为日志实现，对应的依赖为：

```xml
<!-- spring boot 日志系统 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

导入依赖后需要打印日志时，可以像这样：

```java
@RequestMapping("/login")
public String login(){
    Logger logger = LoggerFactory.getLogger(MainController.class);
    logger.info("用户访问了一次登陆界面");
    return "login";
}
```

同时因为之前已经在整合 Web 时添加了 Lombok 依赖，所以也可以直接一个注解搞定：

```java
@Slf4j
@Controller
public class MainController {
    @RequestMapping("/login")
    public String login(){
        log.info("用户访问了一次登陆界面");
        return "login";
    }
}
```

日志级别从低到高分为 TRACE（轨迹） < DEBUG（调试排错） < INFO（信息） < WARN（警告） < ERROR（错误） < FATAL（致命的），SpringBoot 默认只会打印 INFO 以上级别的信息。

### 配置 Logback 日志

[Logback官网](https://logback.qos.ch)

和 JUL 一样，Logback 也能实现定制化。开发者可以编写对应的配置文件，SpringBoot 推荐将配置文件名称命名为 `logback-spring.xml` 表示这是 SpringBoot 下 Logback 专用的配置，可以使用SpringBoot 的高级 Proﬁle 功能，它的内容类似于这样：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 配置 -->
</configuration>
```

最外层由 `configuration` 标签包裹，一旦编写，那么就会替换默认的配置，所以如果内部什么都不写的话，那么会导致 SpringBoot 项目没有配置任何日志输出方式，控制台也不会打印日志。

配置一个控制台日志打印时可以直接导入并使用 SpringBoot 预设好的日志格式。在 `本地仓库\org\springframework\boot\spring-boot\{SpringBoot 版本号}\spring-boot-{SpringBoot 版本号}.jar!\org\springframework\boot\logging\logback\defaults.xml` 中已经把日志的输出格式定义好了，开发者只需要设置对应的 `appender` 即可：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
为导入提供默认的 logback 配置
-->

<included>
	<conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
	<conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
	<conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />

	<property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
	<property name="CONSOLE_LOG_CHARSET" value="${CONSOLE_LOG_CHARSET:-${file.encoding:-UTF-8}}"/>
	<property name="FILE_LOG_PATTERN" value="${FILE_LOG_PATTERN:-%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
	<property name="FILE_LOG_CHARSET" value="${FILE_LOG_CHARSET:-${file.encoding:-UTF-8}}"/>

	<logger name="org.apache.catalina.startup.DigesterFactory" level="ERROR"/>
	<logger name="org.apache.catalina.util.LifecycleBase" level="ERROR"/>
	<logger name="org.apache.coyote.http11.Http11NioProtocol" level="WARN"/>
	<logger name="org.apache.sshd.common.util.SecurityUtils" level="WARN"/>
	<logger name="org.apache.tomcat.util.net.NioSelectorPool" level="WARN"/>
	<logger name="org.eclipse.jetty.util.component.AbstractLifeCycle" level="ERROR"/>
	<logger name="org.hibernate.validator.internal.util.Version" level="WARN"/>
	<logger name="org.springframework.boot.actuate.endpoint.jmx" level="WARN"/>
</included>
```

#### 创建控制台日志打印

导入后，利用预设的日志格式创建一个控制台日志打印：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!--  导入其他配置文件，作为预设  -->
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />

    <!--  Appender 作为日志打印器配置，这里命名随意  -->
    <!--  ch.qos.logback.core.ConsoleAppender 是专用于控制台的 Appender  -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>${CONSOLE_LOG_CHARSET}</charset>
        </encoder>
    </appender>

    <!--  指定日志输出级别，以及启用的 Appender，这里就使用了上面的 ConsoleAppender  -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

配置完成后，控制台已经可以正常打印日志信息了。

#### 开启文件打印

只需要配置一个对应的 Appender 即可：

```xml
<!--  ch.qos.logback.core.rolling.RollingFileAppender 用于文件日志记录，它支持滚动  -->
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <encoder>
        <pattern>${FILE_LOG_PATTERN}</pattern>
        <charset>${FILE_LOG_CHARSET}</charset>
    </encoder>
    <!--  自定义滚动策略，防止日志文件无限变大，也就是日志文件写到什么时候为止，重新创建一个新的日志文件开始写  -->
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <!--  文件保存位置以及文件命名规则，这里用到了 %d{yyyy-MM-dd} 表示当前日期，%i 表示这一天的第 N 个日志  -->
        <FileNamePattern>log/%d{yyyy-MM-dd}-spring-%i.log</FileNamePattern>
        <!--  到期自动清理日志文件  -->
        <cleanHistoryOnStart>true</cleanHistoryOnStart>
        <!--  最大日志保留时间，此处设定为 7 天自动清理  -->
        <maxHistory>7</maxHistory>
        <!--  最大单个日志文件大小  -->
        <maxFileSize>10MB</maxFileSize>
    </rollingPolicy>
</appender>

<!--  指定日志输出级别，以及启用的 Appender，这里就使用了上面的 RollingFileAppender  -->
<root level="INFO">
    <appender-ref ref="FILE"/>
</root>
```

配置完成后，日志文件也能自动生成了。

#### 自定义官方提供的日志格式

官方文档：https://logback.qos.ch/manual/layouts.html

这里需要提及的是 MDC 机制，Logback 内置的日志字段还是比较少，如果需要打印有关业务的更多的内容，包括自定义的一些数据，需要借助 logback MDC 机制，MDC 为 `Mapped Diagnostic Context（映射诊断上下文）`，即将一些运行时的上下文数据通过 logback 打印出来，此时需要借助 org.sl4j.MDC 类。

比如需要记录是哪个用户访问网站的日志，只要是此用户访问网站，都会在日志中携带该用户的ID，希望每条日志中都携带这样一段信息文本，而官方提供的字段无法实现此功能，这时就需要使用 MDC 机制：

```java
@Slf4j
@Controller
public class MainController {
    @RequestMapping("/login")
    public String login(){
      	//这里用 Session 代替 ID
        MDC.put("reqId", request.getSession().getId());
        log.info("用户访问了一次登陆界面");
        return "login";
    }
}
```

通过这种方式，就可以向日志中传入自定义参数了。在日志中添加这样一个占位符 `%X{键值}`，名字保持一致：

```xml
 %clr([%X{reqId}]){faint} 
```

这样当向 MDC 中添加信息后，只要是当前线程（本质是 ThreadLocal 实现）下输出的日志，都会自动替换占位符。

### 自定义 Banner

实际上 Banner 部分和日志部分是独立的，SpringBoot 启动后，会先打印 Banner 部分，这个Banner 部分也是可以自定义。

可以直接来配置文件所在目录下创建一个名为 `banner.txt` 的文本文档，内容随意：

```txt
//                          _ooOoo_                               //
//                         o8888888o                              //
//                         88" . "88                              //
//                         (| ^_^ |)                              //
//                         O\  =  /O                              //
//                      ____/`---'\____                           //
//                    .'  \\|     |//  `.                         //
//                   /  \\|||  :  |||//  \                        //
//                  /  _||||| -:- |||||-  \                       //
//                  |   | \\\  -  /// |   |                       //
//                  | \_|  ''\---/''  |   |                       //
//                  \  .-\__  `-`  ___/-. /                       //
//                ___`. .'  /--.--\  `. . ___                     //
//              ."" '<  `.___\_<|>_/___.'  >'"".                  //
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
//      ========`-.____`-.___\_____/___.-`____.-'========         //
//                           `=---='                              //
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
//             佛祖保佑          永无 BUG         永不修改           //
```

可以使用在线生成网站进行生成自己的个性 Banner：https://www.bootschool.net/ascii

甚至还可以使用颜色代码来为文本切换颜色：

```
${AnsiColor.BRIGHT_GREEN}  //绿色
```

也可以获取一些常用的变量信息：

```
${AnsiColor.YELLOW} 当前 Spring Boot 版本：${spring-boot.version}
```

## 多环境配置

在日常开发中，项目会有多个环境。例如开发环境（develop）也就是研发过程中疯狂敲代码修 BUG 阶段，生产环境（production ）项目开发得差不多了，才可以放在服务器上跑了。<br>不同的环境下，可能配置文件也存在不同，但是不可能切换环境的时候又去重新写一次配置文件，所以开发者可以将多个环境的配置文件提前写好，进行自由切换。

由于 SpringBoot 只会读取 `application.properties` 或是 `application.yml` 文件，那么怎么才能实现自由切换呢？<br>SpringBoot 提供了一种方式：可以通过配置文件指定：

1. 创建两个环境的配置文件，`application-dev.yml` 和 `application-prod.yml`分别表示开发环境和生产环境的配置文件，比如开发环境使用的服务器端口为 8080，而生产环境下可能就需要设置为 80 或是 443 端口，那么这个时候就需要不同环境下的配置文件进行区分：

   ```yaml
   server:
     port: 8080
   ```

   ```yaml
   server:
     port: 80
   ```

2. SpringBoot 自带的 Logback 日志系统也是支持多环境配置的，比如想在开发环境下输出日志到控制台，而生产环境下只需要输出到文件即可，这时就需要进行环境配置：

   ```xml
   <springProfile name="dev">
       <root level="INFO">
           <appender-ref ref="CONSOLE"/>
           <appender-ref ref="FILE"/>
       </root>
   </springProfile>
   
   <springProfile name="prod">
       <root level="INFO">
           <appender-ref ref="FILE"/>
       </root>
   </springProfile>
   ```

> <font color="brown">注意</font>
>
> `springProfile` 是区分大小写的！

3. Maven 设置多环境

   ```xml
   <!--分别设置开发、生产环境-->
   <profiles>
       <!-- 开发环境 -->
       <profile>
           <id>dev</id>
           <activation>
               <activeByDefault>true</activeByDefault>
           </activation>
           <properties>
               <environment>dev</environment>
           </properties>
       </profile>
       <!-- 生产环境 -->
       <profile>
           <id>prod</id>
           <activation>
               <activeByDefault>false</activeByDefault>
           </activation>
           <properties>
               <environment>prod</environment>
           </properties>
       </profile>
   </profiles>
   ```

4. 需要根据环境的不同，排除其他环境的配置文件：

   ```xml
   <resources>
   <!--排除配置文件-->
       <resource>
           <directory>src/main/resources</directory>
           <!--先排除所有的配置文件-->
           <excludes>
               <!--使用通配符，当然可以定义多个 exclude 标签进行排除-->
               <exclude>application*.yml</exclude>
           </excludes>
       </resource>
   
       <!--根据激活条件引入打包所需的配置和文件-->
       <resource>
           <directory>src/main/resources</directory>
           <!--引入所需环境的配置文件-->
           <filtering>true</filtering>
           <includes>
               <include>application.yml</include>
               <!--根据 maven 选择环境导入配置文件-->
               <include>application-${environment}.yml</include>
           </includes>
       </resource>
   </resources>
   ```

5. 直接将 Maven 中的 `environment` 属性，传递给 SpringBoot 的配置文件，在构建时替换为对应的值：

   ```yaml
   spring:
     profiles:
       active: '@environment@'  #注意 YAML 配置文件需要加单引号，否则会报错
   ```

这样，根据 Maven 环境的切换，SpringBoot 的配置文件也会进行对应的切换。

最后打开 Maven 栏目，就可以自由切换了，直接勾选即可，注意切换环境之后要重新加载一下 Maven 项目，不然不会生效！

![maven 多环境选择](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%2520%25E6%258A%2580%25E6%259C%25AF/%25E6%25A1%2586%25E6%259E%25B6/Spring%2520Boot/maven%2520%25E5%25A4%259A%25E7%258E%25AF%25E5%25A2%2583%25E9%2580%2589%25E6%258B%25A9.png)

## 打包运行

点击 Maven 生命周期中的 `package` 即可，它会自动将其打包为可直接运行的 Jar 包，第一次打包可能会花费一些时间下载部分依赖的源码一起打包进 Jar 文件。

在打包的过程中还会完整的将项目跑一遍进行测试，如果不想测试直接打包，可以在执行 Maven 目标中使用以下命令：

```shell
mvn package  -DskipTests
```

> <font color="green">提示</font>
>
> ![执行 Maven 目标](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%2520%25E6%258A%2580%25E6%259C%25AF/%25E6%25A1%2586%25E6%259E%25B6/Spring%2520Boot/%25E6%2589%25A7%25E8%25A1%258C%2520Maven%2520%25E7%259B%25AE%25E6%25A0%2587.png)

打包后，会得到一个名为`springboot-study-0.0.1-SNAPSHOT.jar`的文件，这时在 CMD 窗口中输入命令：

```shell
java -jar springboot-study-0.0.1-SNAPSHOT.jar
```

输入后，就可以看到 Java 项目成功运行起来了，如果手动关闭窗口会导致整个项目终止运行。

## 扩充 Spring 框架

### 任务调度

为了执行某些任务，可能需要一些非常规的操作：比如希望使用多线程来处理结果或是执行一些定时任务，到达指定时间再去执行。

之前可以创建一个新的线程来处理，或是使用 TimerTask 来完成定时任务，但现在 Spring 框架为此提供了更加便捷的方式进行任务调度。

#### 异步任务

1. 需要使用 Spring 异步任务支持，需要在配置类上添加 `@EnableAsync` 或是在 SpringBoot 的启动类上添加也可以。

   ```java
   @EnableAsync
   @SpringBootApplication
   public class SpringBootWebTestApplication {
       public static void main(String[] args) {
           SpringApplication.run(SpringBootWebTestApplication.class, args);
       }
   }
   ```

2. 在需要异步执行的方法上，添加 `@Async` 注解即可将此方法标记为异步，当此方法被调用时，会异步执行，也就是新开一个线程执行，不是在当前线程执行。

   ```java
   import org.springframework.scheduling.annotation.Async;
   import org.springframework.stereotype.Service;
   
   @Service
   public class TestService {
       @Async
       public void test(){
           try {
               Thread.sleep(3000);
               System.out.println("我是异步任务！");
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
       }
   }
   ```

   ```java
   import com.example.springbootstudy.service.TestService;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;
   
   import javax.annotation.Resource;
   
   @Controller
   public class MainController {
       @Resource
       TestService testService;
   
       @RequestMapping("/login")
       public String login() {
           testService.test();
           System.out.println("我是同步任务");
           return "login";
       }
   }
   ```

实际上这也是得益于 AOP 机制，通过线程池实现，但是也要注意，正是因为它是 AOP 机制的产物，所以它只能是在 Bean 中才会生效！

使用 @Async 注释的方法可以返回 'void' 或 "Future" 类型。

> Future 类是一种异步任务监视器，可以让提交者能够监视任务的执行，同时可以取消任务的执行，也可以获取任务返回结果。
>
> 作用：<br>		比如在做一定的任务运算的时候，需要等待比较长时间，这个任务是比较耗时的，需要比较繁重的运算，比如加密压缩等等。如果一直在等程序执行完成是不明智的，这时可以将这个比较耗时的任务交给子线程执行，然后通过 Future 类监视线程执行，获取返回的结果。
>
> 这样一来就提高了工作效率，这是一种异步的思想。【并发编程】

#### 定时任务

定时任务其实就是指定在哪个时候再去执行，在 JavaSE 阶段使用过 TimerTask 来执行定时任务。

Spring 中的定时任务是全局性质的，当 Spring 程序启动后，那么定时任务也就跟着启动了。开发者可以在配置类上添加 `@EnableScheduling` 或是在 SpringBoot 的启动类上添加也可。

1. 在 SpringBoot 的启动类上添加 `@EnableScheduling`

   ```java
   @EnableScheduling
   @SpringBootApplication
   public class SpringBootWebTestApplication {
       public static void main(String[] args) {
   		SpringApplication.run(SpringBootWebTestApplication.class, args);
       }
   }
   ```

2. 创建一个定时任务配置类，在配置类里面编写定时任务：

   ```java
   @Configuration
   public class ScheduleConfiguration {
       @Scheduled(fixedRate = 2000)
       public void task(){
           System.out.println("我是定时任务！"+new Date());
       }
   }
   ```

`@Scheduled` 中有很多参数，开发者需要指定 `cron`, `fixedDelay(String)`, or `fixedRate(String)` 的其中一个，否则无法创建定时任务。<br>它们的区别如下：

- fixedDelay：在上一次定时任务执行完之后，间隔多久继续执行。
- fixedRate：无论上一次定时任务有没有执行完成，两次任务之间的时间间隔。
- [cron](/Library/cron 表达式.md) ：使用 cron 表达式来指定任务计划。

### 监听器

监听实际上就是等待某个事件的触发，当事件触发时，对应事件的监听器就会被通知。

```java
@Component
public class TestListener implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println(event.getApplicationContext());
    }
}
```

通过监听事件，就可以在对应的时机进行一些额外的处理，开发者可以通过断点调试来查看一个事件是如何发生，以及如何通知监听器的。

通过阅读源码可以得知，一个事件实际上就是通过 `publishEvent` 方法来进行发布的，也可以自定义项目中的事件，并注册对应的监听器进行处理。

1. 自定义 event 事件

```java
public class TestEvent extends ApplicationEvent {   //需要继承 ApplicationEvent
    public TestEvent(Object source) {
        super(source);
    }
}
```

2. 编写监听器代码

```java
@Component
public class TestListener implements ApplicationListener<TestEvent> {
    @Override
    public void onApplicationEvent(TestEvent event) {
        System.out.println("自定义事件发生了："+event.getSource());
    }
}
```

3. 在控制器中发布事件

```java
import com.example.springbootstudy.listener.TestEvent;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.annotation.Resource;

@Controller
public class MainController {
    @Resource
    ApplicationContext context;

    @RequestMapping("/login")
    public String login() {
        context.publishEvent(new TestEvent("有人访问了登录界面！"));
        return "login";
    }
}
```

### Aware 系列接口

Spring 源码中，经常会发现某些类的定义上，除了当时需要的继承关系以外，还实现了一些接口，这些接口的名称基本都是 `xxxxAware`。<br>比如 SpringSecurity 的源码中，AbstractAuthenticationProcessingFilter 类就是这样：

```java
public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean implements ApplicationEventPublisherAware, MessageSourceAware {
    ...
    ...
    ...
}
```

它除了继承自 `GenericFilterBean` 之外，还实现了 `ApplicationEventPublisherAware` 和 `MessageSourceAware` 接口。

Aware 的中文意思为**感知**。简单来说，它就是一个标识，实现此接口的类会获得某些感知能力，Spring 容器会在 Bean 被加载时，根据类实现的感知接口，会调用类中实现的对应感知方法。

比如 `AbstractAuthenticationProcessingFilter` 就实现了 `ApplicationEventPublisherAware` 接口，此接口的感知功能为事件发布器，在 Bean 加载时，会调用实现类中的 `setApplicationEventPublisher` 方法，而 `AbstractAuthenticationProcessingFilter` 类则利用此方法，在 Bean 加载阶段获得了容器的事件发布器，以便之后发布事件使用。

```java
public void setApplicationEventPublisher(ApplicationEventPublisher eventPublisher) {
    this.eventPublisher = eventPublisher;   //直接存到成员变量
}
```

```java
protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
    SecurityContext context = SecurityContextHolder.createEmptyContext();
    context.setAuthentication(authResult);
    SecurityContextHolder.setContext(context);
    if (this.logger.isDebugEnabled()) {
        this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
    }

    this.rememberMeServices.loginSuccess(request, response, authResult);
  	//在这里使用
    if (this.eventPublisher != null) {
        this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
    }

    this.successHandler.onAuthenticationSuccess(request, response, authResult);
}
```

同样的，除了 ApplicationEventPublisherAware 接口外，再来看一个接口。比如：

```java
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.stereotype.Service;

@Service
public class TestService implements BeanNameAware {
    @Override
    public void setBeanName(String s) {
        System.out.println(s);
    }
}
```

其中 `BeanNameAware` 就是感知 Bean 名称的一个接口，当 Bean 被加载时，会调用 `setBeanName` 方法并将 Bean 名称作为参数传递。

## Spring Boot 实现原理

### 启动原理

SpringBoot 项目启动之后，做了什么事情。

SpringBoot 项目启动之后 SpringApplication 中的静态 `run` 方法：

```java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    return run(new Class[]{primarySource}, args);
}
```

套娃如下：

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return (new SpringApplication(primarySources)).run(args);	//这里直接 new 了一个新的 SpringApplication 对象，传入主类作为构造方法参数，并调用了非 static 的 run() 方法
}
```

来看看构造方法里面做了什么事情：

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.sources = new LinkedHashSet();
    this.bannerMode = Mode.CONSOLE;
    this.logStartupInfo = true;
    this.addCommandLineProperties = true;
    this.addConversionService = true;
    this.headless = true;
    this.registerShutdownHook = true;
    this.additionalProfiles = Collections.emptySet();
    this.isCustomEnvironment = false;
    this.lazyInitialization = false;
    this.applicationContextFactory = ApplicationContextFactory.DEFAULT;
    this.applicationStartup = ApplicationStartup.DEFAULT;
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
  	//这里是关键，这里会判断当前 SpringBoot 应用程序是否为 Web 项目，并返回当前的项目类型
    this.webApplicationType = WebApplicationType.deduceFromClasspath();	//deduceFromClasspath 是根据类路径下判断是否包含 SpringBootWeb 依赖，如果不包含就是 NONE 类型，包含就是 SERVLET 类型
    this.bootstrapRegistryInitializers = new ArrayList(this.getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
    this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));	//创建所有 ApplicationContextInitializer 实现类的对象
    this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = this.deduceMainApplicationClass();
}
```

关键就在这里了，它是如何知道哪些类是 `ApplicationContextInitializer` 的实现类的呢？

这里就要提到 spring.factories 了，它是 Spring 仿造 Java SPI 实现的一种类加载机制。它在 META-INF/spring.factories 文件中配置接口的实现类名称，然后在程序中读取这些配置文件并实例化。这种自定义的 SPI 机制是 Spring Boot Starter 实现的基础。

SPI 的常见例子：

- 数据库驱动加载接口实现类的加载：JDBC 加载不同类型数据库的驱动
- 日志门面接口实现类加载：SLF4J 加载不同提供商的日志实现类

说白了就是人家定义接口，但是实现可能有很多种，但是核心只提供接口，需要我们按需选择对应的实现，这种方式是高度解耦的。

我们来看看 `getSpringFactoriesInstances` 方法做了什么：

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
  	//获取当前的类加载器
    ClassLoader classLoader = this.getClassLoader();
  	//获取所有依赖中 META-INF/spring.factories 中配置的对应接口类的实现类列表
    Set<String> names = new LinkedHashSet(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
  	//根据上方列表，依次创建实例对象  
  List<T> instances = this.createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
  	//根据对应类上的Order接口或是注解进行排序
    AnnotationAwareOrderComparator.sort(instances);
  	//返回实例
    return instances;
}
```

其中`SpringFactoriesLoader.loadFactoryNames`正是读取配置的核心部分，我们后面还会遇到。

接着我们来看run方法里面做了什么事情。

```java
public ConfigurableApplicationContext run(String... args) {
    long startTime = System.nanoTime();
    DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
    ConfigurableApplicationContext context = null;
    this.configureHeadlessProperty();
  	//获取所有的SpringApplicationRunListener，并通知启动事件，默认只有一个实现类EventPublishingRunListener
  	//EventPublishingRunListener会将初始化各个阶段的事件转发给所有监听器
    SpringApplicationRunListeners listeners = this.getRunListeners(args);
    listeners.starting(bootstrapContext, this.mainApplicationClass);

    try {
      	//环境配置
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        this.configureIgnoreBeanInfo(environment);
      	//打印Banner
        Banner printedBanner = this.printBanner(environment);
      	//创建ApplicationContext，注意这里会根据是否为Web容器使用不同的ApplicationContext实现类
        context = this.createApplicationContext();
        context.setApplicationStartup(this.applicationStartup);
      	//初始化ApplicationContext
        this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
      	//执行ApplicationContext的refresh方法
        this.refreshContext(context);
        this.afterRefresh(context, applicationArguments);
        Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
        if (this.logStartupInfo) {
            (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), timeTakenToStartup);
        }
        ....
}
```

我们发现，实际上SpringBoot就是Spring的一层壳罢了，离不开最关键的ApplicationContext，也就是说，在启动后会自动配置一个ApplicationContext，只不过是进行了大量的扩展。

我们来看ApplicationContext是怎么来的，打开`createApplicationContext`方法：

```java
protected ConfigurableApplicationContext createApplicationContext() {
    return this.applicationContextFactory.create(this.webApplicationType);
}
```

我们发现在构造方法中`applicationContextFactory`直接使用的是DEFAULT：

```java
this.applicationContextFactory = ApplicationContextFactory.DEFAULT;
```

```java
ApplicationContextFactory DEFAULT = (webApplicationType) -> {
    try {
        switch(webApplicationType) {
        case SERVLET:
            return new AnnotationConfigServletWebServerApplicationContext();
        case REACTIVE:
            return new AnnotationConfigReactiveWebServerApplicationContext();
        default:
            return new AnnotationConfigApplicationContext();
        }
    } catch (Exception var2) {
        throw new IllegalStateException("Unable create a default ApplicationContext instance, you may need a custom ApplicationContextFactory", var2);
    }
};

ConfigurableApplicationContext create(WebApplicationType webApplicationType);
```

DEFAULT是直接编写的一个匿名内部类，其实已经很明确了，正是根据`webApplicationType`类型进行判断，如果是SERVLET，那么久返回专用于Web环境的AnnotationConfigServletWebServerApplicationContext对象（SpringBoot中新增的），否则返回普通的AnnotationConfigApplicationContext对象，也就是到这里为止，Spring的容器就基本已经确定了。

注意AnnotationConfigApplicationContext是Spring框架提供的类，从这里开始相当于我们在讲Spring的底层源码了，我们继续深入，AnnotationConfigApplicationContext对象在创建过程中会创建`AnnotatedBeanDefinitionReader`，它是用于通过注解解析Bean定义的工具类：

```java
public AnnotationConfigApplicationContext() {
    StartupStep createAnnotatedBeanDefReader = this.getApplicationStartup().start("spring.context.annotated-bean-reader.create");
    this.reader = new AnnotatedBeanDefinitionReader(this);
    createAnnotatedBeanDefReader.end();
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

其构造方法：

```java
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
    ...
    //这里会注册很多的后置处理器
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```

```java
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(BeanDefinitionRegistry registry, @Nullable Object source) {
    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
    ....
    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet(8);
    RootBeanDefinition def;
    if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalConfigurationAnnotationProcessor")) {
      	//注册了ConfigurationClassPostProcessor用于处理@Configuration、@Import等注解
      	//注意这里是关键，之后Selector还要讲到它
      	//它是继承自BeanDefinitionRegistryPostProcessor，所以它的执行时间在Bean定义加载完成后，Bean初始化之前
        def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalConfigurationAnnotationProcessor"));
    }

    if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalAutowiredAnnotationProcessor")) {
      	//AutowiredAnnotationBeanPostProcessor用于处理@Value等注解自动注入
        def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalAutowiredAnnotationProcessor"));
    }
  
  	...
```

回到SpringBoot，我们最后来看，`prepareContext`方法中又做了什么事情：

```java
private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context, ConfigurableEnvironment environment, SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
  	//环境配置
    context.setEnvironment(environment);
    this.postProcessApplicationContext(context);
    this.applyInitializers(context);
    listeners.contextPrepared(context);
    bootstrapContext.close(context);
    if (this.logStartupInfo) {
        this.logStartupInfo(context.getParent() == null);
        this.logStartupProfileInfo(context);
    }

  	//将Banner注册为Bean
    ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
    beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
    if (printedBanner != null) {
        beanFactory.registerSingleton("springBootBanner", printedBanner);
    }

    if (beanFactory instanceof AbstractAutowireCapableBeanFactory) {
        ((AbstractAutowireCapableBeanFactory)beanFactory).setAllowCircularReferences(this.allowCircularReferences);
        if (beanFactory instanceof DefaultListableBeanFactory) {
            ((DefaultListableBeanFactory)beanFactory).setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
        }
    }

    if (this.lazyInitialization) {
        context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
    }

  	//这里会获取我们一开始传入的项目主类
    Set<Object> sources = this.getAllSources();
    Assert.notEmpty(sources, "Sources must not be empty");
  	//这里会将我们的主类直接注册为Bean，这样就可以通过注解加载了
    this.load(context, sources.toArray(new Object[0]));
    listeners.contextLoaded(context);
}
```

因此，在`prepareContext`执行完成之后，我们的主类成功完成Bean注册，接下来，就该类上注解大显身手了。

### 启动配置原理

既然主类已经在初始阶段注册为Bean，那么在加载时，就会根据注解定义，进行更多的额外操作。所以我们来看看主类上的`@SpringBootApplication`注解做了什么事情。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

我们发现，`@SpringBootApplication`上添加了`@ComponentScan`注解，此注解我们此前已经认识过了，但是这里并没有配置具体扫描的包，因此它会自动将声明此接口的类所在的包作为basePackage，因此当添加`@SpringBootApplication`之后也就等于直接开启了自动扫描，但是一定注意不能在主类之外的包进行Bean定义，否则无法扫描到，需要手动配置。

接着我们来看第二个注解`@EnableAutoConfiguration`，它就是自动配置的核心了，我们来看看它是如何定义的：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```

老套路了，直接一手`@Import`，通过这种方式来将一些外部的Bean加载到容器中。我们来看看AutoConfigurationImportSelector做了什么事情：

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
		...
}
```

我们看到它实现了很多接口，包括大量的Aware接口，实际上就是为了感知某些必要的对象，并将其存到当前类中。

其中最核心的是`DeferredImportSelector`接口，它是`ImportSelector`的子类，它定义了`selectImports`方法，用于返回需要加载的类名称，在Spring加载ImportSelector类型的Bean时，会调用此方法来获取更多需要加载的类，并将这些类一并注册为Bean：

```java
public interface ImportSelector {
    String[] selectImports(AnnotationMetadata importingClassMetadata);

    @Nullable
    default Predicate<String> getExclusionFilter() {
        return null;
    }
}
```

到目前为止，我们了解了两种使用`@Import`有特殊机制的接口：ImportSelector（这里用到的）和ImportBeanDefinitionRegistrar（之前Mybatis-spring源码有讲）当然还有普通的`@Configuration`配置类。

我们可以来阅读一下`ConfigurationClassPostProcessor`的源码，看看它到底是如何处理`@Import`的：

```java
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    List<BeanDefinitionHolder> configCandidates = new ArrayList();
  	//注意这个阶段仅仅是已经完成扫描了所有的Bean，得到了所有的BeanDefinition，但是还没有进行任何区分
  	//candidate是候选者的意思，一会会将标记了@Configuration的类作为ConfigurationClass加入到configCandidates中
    String[] candidateNames = registry.getBeanDefinitionNames();
    String[] var4 = candidateNames;
    int var5 = candidateNames.length;

    for(int var6 = 0; var6 < var5; ++var6) {
        String beanName = var4[var6];
        BeanDefinition beanDef = registry.getBeanDefinition(beanName);
        if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
            }
        } else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {   //判断是否添加了@Configuration注解
            configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
        }
    }

    if (!configCandidates.isEmpty()) {
        //...省略

      	//这里创建了一个ConfigurationClassParser用于解析配置类
        ConfigurationClassParser parser = new ConfigurationClassParser(this.metadataReaderFactory, this.problemReporter, this.environment, this.resourceLoader, this.componentScanBeanNameGenerator, registry);
      	//所有配置类的BeanDefinitionHolder列表
        Set<BeanDefinitionHolder> candidates = new LinkedHashSet(configCandidates);
      	//已经解析完成的类
        HashSet alreadyParsed = new HashSet(configCandidates.size());

        do {
            //这里省略，直到所有的配置类全部解析完成
          	//注意在循环过程中可能会由于@Import新增更多的待解析配置类，一律丢进candidates集合中
        } while(!candidates.isEmpty());

        ...

    }
}
```

我们接着来看，`ConfigurationClassParser`是如何进行解析的：

```java
protected void processConfigurationClass(ConfigurationClass configClass, Predicate<String> filter) throws IOException {
  	//@Conditional相关注解处理
  	//后面会讲
    if (!this.conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.PARSE_CONFIGURATION)) {
        ...
        }

        ConfigurationClassParser.SourceClass sourceClass = this.asSourceClass(configClass, filter);

        do {
          	//核心
            sourceClass = this.doProcessConfigurationClass(configClass, sourceClass, filter);
        } while(sourceClass != null);

        this.configurationClasses.put(configClass, configClass);
    }
}
```

最后我们再来看最核心的`doProcessConfigurationClass`方法：

```java
protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)
    ...

    processImports(configClass, sourceClass, getImports(sourceClass), true);    // 处理Import注解

		...

    return null;
}
```

```java
private void processImports(ConfigurationClass configClass, ConfigurationClassParser.SourceClass currentSourceClass, Collection<ConfigurationClassParser.SourceClass> importCandidates, Predicate<String> exclusionFilter, boolean checkForCircularImports) {
    if (!importCandidates.isEmpty()) {
        if (checkForCircularImports && this.isChainedImportOnStack(configClass)) {
            this.problemReporter.error(new ConfigurationClassParser.CircularImportProblem(configClass, this.importStack));
        } else {
            this.importStack.push(configClass);

            try {
                Iterator var6 = importCandidates.iterator();

                while(var6.hasNext()) {
                    ConfigurationClassParser.SourceClass candidate = (ConfigurationClassParser.SourceClass)var6.next();
                    Class candidateClass;
                  	//如果是ImportSelector类型，继续进行运行
                    if (candidate.isAssignable(ImportSelector.class)) {
                        candidateClass = candidate.loadClass();
                        ImportSelector selector = (ImportSelector)ParserStrategyUtils.instantiateClass(candidateClass, ImportSelector.class, this.environment, this.resourceLoader, this.registry);
                        Predicate<String> selectorFilter = selector.getExclusionFilter();
                        if (selectorFilter != null) {
                            exclusionFilter = exclusionFilter.or(selectorFilter);
                        }
									//如果是DeferredImportSelector的实现类，那么会走deferredImportSelectorHandler的handle方法
                        if (selector instanceof DeferredImportSelector) {
                            this.deferredImportSelectorHandler.handle(configClass, (DeferredImportSelector)selector);
                          //否则就按照正常的ImportSelector类型进行加载
                        } else {
                          	//调用selectImports方法获取所有需要加载的类
                            String[] importClassNames = selector.selectImports(currentSourceClass.getMetadata());
                            Collection<ConfigurationClassParser.SourceClass> importSourceClasses = this.asSourceClasses(importClassNames, exclusionFilter);
                          	//递归处理，直到没有
                            this.processImports(configClass, currentSourceClass, importSourceClasses, exclusionFilter, false);
                        }
                      //判断是否为ImportBeanDefinitionRegistrar类型
                    } else if (candidate.isAssignable(ImportBeanDefinitionRegistrar.class)) {
                        candidateClass = candidate.loadClass();
                        ImportBeanDefinitionRegistrar registrar = (ImportBeanDefinitionRegistrar)ParserStrategyUtils.instantiateClass(candidateClass, ImportBeanDefinitionRegistrar.class, this.environment, this.resourceLoader, this.registry);
                      	//往configClass丢ImportBeanDefinitionRegistrar信息进去，之后再处理
                        configClass.addImportBeanDefinitionRegistrar(registrar, currentSourceClass.getMetadata());
                      //否则按普通的配置类进行处理
                    } else {
                        this.importStack.registerImport(currentSourceClass.getMetadata(), candidate.getMetadata().getClassName());
                        this.processConfigurationClass(candidate.asConfigClass(configClass), exclusionFilter);
                    }
                }
            } catch (BeanDefinitionStoreException var17) {
                throw var17;
            } catch (Throwable var18) {
                throw new BeanDefinitionStoreException("Failed to process import candidates for configuration class [" + configClass.getMetadata().getClassName() + "]", var18);
            } finally {
                this.importStack.pop();
            }
        }

    }
}
```

不难注意到，虽然这里额外处理了`ImportSelector`对象，但是还针对`ImportSelector`的子接口`DeferredImportSelector`进行了额外处理，Deferred是延迟的意思，它是一个延迟执行的`ImportSelector`，并不会立即进处理，而是丢进DeferredImportSelectorHandler，并且在`parse`方法的最后进行处理：

```java
public void parse(Set<BeanDefinitionHolder> configCandidates) {
     ...

    this.deferredImportSelectorHandler.process();
}
```

我们接着来看`DeferredImportSelector`正好就有一个`process`方法：

```java
public interface DeferredImportSelector extends ImportSelector {
    @Nullable
    default Class<? extends DeferredImportSelector.Group> getImportGroup() {
        return null;
    }

    public interface Group {
        void process(AnnotationMetadata metadata, DeferredImportSelector selector);

        Iterable<DeferredImportSelector.Group.Entry> selectImports();

        public static class Entry {
          ...
```

最后经过ConfigurationClassParser处理完成后，通过`parser.getConfigurationClasses()`就能得到通过配置类导入了哪些额外的配置类。最后将这些配置类全部注册BeanDefinition，然后就可以交给接下来的Bean初始化过程去处理了。

```java
this.reader.loadBeanDefinitions(configClasses);
```

最后我们再去看`loadBeanDefinitions`是如何运行的：

```java
public void loadBeanDefinitions(Set<ConfigurationClass> configurationModel) {
    ConfigurationClassBeanDefinitionReader.TrackedConditionEvaluator trackedConditionEvaluator = new ConfigurationClassBeanDefinitionReader.TrackedConditionEvaluator();
    Iterator var3 = configurationModel.iterator();

    while(var3.hasNext()) {
        ConfigurationClass configClass = (ConfigurationClass)var3.next();
        this.loadBeanDefinitionsForConfigurationClass(configClass, trackedConditionEvaluator);
    }

}

private void loadBeanDefinitionsForConfigurationClass(ConfigurationClass configClass, ConfigurationClassBeanDefinitionReader.TrackedConditionEvaluator trackedConditionEvaluator) {
    if (trackedConditionEvaluator.shouldSkip(configClass)) {
        String beanName = configClass.getBeanName();
        if (StringUtils.hasLength(beanName) && this.registry.containsBeanDefinition(beanName)) {
            this.registry.removeBeanDefinition(beanName);
        }

        this.importRegistry.removeImportingClass(configClass.getMetadata().getClassName());
    } else {
        if (configClass.isImported()) {
            this.registerBeanDefinitionForImportedConfigurationClass(configClass);  //注册配置类自己
        }

        Iterator var3 = configClass.getBeanMethods().iterator();

        while(var3.hasNext()) {
            BeanMethod beanMethod = (BeanMethod)var3.next();
            this.loadBeanDefinitionsForBeanMethod(beanMethod); //注册@Bean注解标识的方法
        }

      	//注册`@ImportResource`引入的XML配置文件中读取的bean定义
        this.loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());
     	 //注册configClass中经过解析后保存的所有ImportBeanDefinitionRegistrar，注册对应的BeanDefinition
        this.loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());
    }
}
```

这样，整个`@Configuration`配置类的底层配置流程我们就大致了解了。接着我们来看AutoConfigurationImportSelector是如何实现自动配置的，可以看到内部类AutoConfigurationGroup的process方法，它是父接口的实现，因为父接口是`DeferredImportSelector`，那么很容易得知，实际上最后会调用`process`方法获取所有的自动配置类：

```java
public void process(AnnotationMetadata annotationMetadata, DeferredImportSelector deferredImportSelector) {
    Assert.state(deferredImportSelector instanceof AutoConfigurationImportSelector, () -> {
        return String.format("Only %s implementations are supported, got %s", AutoConfigurationImportSelector.class.getSimpleName(), deferredImportSelector.getClass().getName());
    });
  	//获取所有的Entry，其实就是，读取spring.factories来查看有哪些自动配置类
    AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = ((AutoConfigurationImportSelector)deferredImportSelector).getAutoConfigurationEntry(annotationMetadata);
    this.autoConfigurationEntries.add(autoConfigurationEntry);
    Iterator var4 = autoConfigurationEntry.getConfigurations().iterator();

    while(var4.hasNext()) {
        String importClassName = (String)var4.next();
        this.entries.putIfAbsent(importClassName, annotationMetadata);
    }

}
```

我们接着来看`getAutoConfigurationEntry`方法：

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
  	//判断是否开启了自动配置，是的，自动配置可以关
    if (!this.isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    } else {
      	//根据注解定义获取一些属性
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
      	//得到spring.factories文件中所有需要自动配置的类
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        ... 这里先看前半部分
    }
}
```

注意这里并不是spring.factories文件中所有的自动配置类都会被加载，它会根据@Condition注解的条件进行加载。这样就能实现我们需要什么模块添加对应依赖就可以实现自动配置了。

所有的源码看不懂，都源自于你的心中没有形成一个完整的闭环！一旦一条线推到头，闭环形成，所有疑惑迎刃而解。

### 自定义Starter

我们仿照Mybatis来编写一个自己的starter，Mybatis的starter包含两个部分：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot</artifactId>
    <version>2.2.0</version>
  </parent>
  <!--  starter本身只做依赖集中管理，不编写任何代码  -->
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <name>mybatis-spring-boot-starter</name>
  <properties>
    <module.name>org.mybatis.spring.boot.starter</module.name>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <!--  编写的专用配置模块   -->
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
    </dependency>
  </dependencies>
</project>
```

因此我们也将我们自己的starter这样设计：

我们设计三个模块：

* spring-boot-hello：基础业务功能模块
* spring-boot-starter-hello：启动器
* spring-boot-autoconifgurer-hello：自动配置依赖

首先是基础业务功能模块，这里我们随便创建一个类就可以了：

```java
public class HelloWorldService {
	
}
```

启动器主要做依赖管理，这里就不写任何代码，只写pom文件：

```xml
<dependencies>
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>spring-boot-autoconfigurer-hello</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

导入autoconfigurer模块作为依赖即可，接着我们去编写autoconfigurer模块，首先导入依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
        <version>2.6.2</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <version>2.6.2</version>
        <optional>true</optional>
    </dependency>
    
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>spring-boot-hello</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

接着创建一个HelloWorldAutoConfiguration作为自动配置类：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication
@ConditionalOnClass(HelloWorldService.class)
@EnableConfigurationProperties(HelloWorldProperties.class)
public class HelloWorldAutoConfiguration {

    Logger logger = Logger.getLogger(this.getClass().getName());

    @Resource
    HelloWorldProperties properties;

    @Bean
    public HelloWorldService helloWorldService(){
        logger.info("自定义starter项目已启动！");
        logger.info("读取到自定义配置："+properties.getValue());
        return new HelloWorldService();
    }
}
```

对应的配置读取类：

```java
@ConfigurationProperties("hello.world")
public class HelloWorldProperties {

    private String value;

    public void setValue(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}
```

最后再编写`spring.factories`文件，并将我们的自动配置类添加即可：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.hello.autoconfigurer.HelloWorldAutoConfiguration
```

最后再Maven根项目执行`install`安装到本地仓库，完成。接着就可以在其他项目中使用我们编写的自定义starter了。

### Runner接口

在项目中，可能会遇到这样一个问题：我们需要在项目启动完成之后，紧接着执行一段代码。

我们可以编写自定义的ApplicationRunner来解决，它会在项目启动完成后执行：

```java
@Component
public class TestRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("我是自定义执行！");
    }
}
```

当然也可以使用CommandLineRunner，它也支持使用@Order或是实现Ordered接口来支持优先级执行。

实际上它就是run方法的最后：

```java
public ConfigurableApplicationContext run(String... args) {
    ....

        listeners.started(context, timeTakenToStartup);
  			//这里已经完成整个SpringBoot项目启动，所以执行所有的Runner
        this.callRunners(context, applicationArguments);
    } catch (Throwable var12) {
        this.handleRunFailure(context, var12, listeners);
        throw new IllegalStateException(var12);
    }

    try {
        Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
        listeners.ready(context, timeTakenToReady);
        return context;
    } catch (Throwable var11) {
        this.handleRunFailure(context, var11, (SpringApplicationRunListeners)null);
        throw new IllegalStateException(var11);
    }
}
```

---

![项目目录](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring%20Boot/%E9%A1%B9%E7%9B%AE%E7%9B%AE%E5%BD%95.png)