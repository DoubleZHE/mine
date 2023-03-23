---
title: Sping MVC
tags: 
 - 编程学习
 - Java
 - SSM 框架
categories: 
 - SSM 框架
keywords: 
date: 2022-01-06 19:11:37
description: 
top_img:
cover:https://qiniu.daxueyiwu.com/gxitsky/b4fac27466e511ec9ef0005056c000081640589731.jpg
---

## SpringMVC 简介

### 什么是 MVC

MVC 是一种软件架构的思想,将软件按照模型、视图、控制器来划分

M:Model,模型层,指工程中的 JavaBean,作用是处理数据

JavaBean 分为两类:

- 一类称为实体类 Bean:专门存储业务数据的,如 Student、User 等
- 一类称为业务处理 Bean:指 Service 或 Dao 对象,专门用于处理业务逻辑和数据访问。

V:View,视图层,指工程中的 html 或 jsp 等页面,作用是与用户进行交互,展示数据

C:Controller,控制层,指工程中的 servlet,作用是接收请求和响应浏览器

MVC的工作流程:
用户通过视图层发送请求到服务器,在服务器中请求被 Controller 接收,Controller 调用相应的 Model 层处理请求,处理完毕将结果返回到 Controller,Controller 再根据请求处理的结果找到相应的 View 视图,渲染数据后最终响应给浏览器 ^4ab29b

### 什么是 SpringMVC

SpringMVC 是 Spring 的一个后续产品,是 Spring 的一个子项目

SpringMVC 是 Spring 为表述层开发提供的一整套完备的解决方案。在表述层框架历经 Strust、WebWork、Strust2 等诸多产品的历代更迭之后,目前业界普遍选择了 SpringMVC 作为 Java EE 项目表述层开发的**首选方案**。

> 注:三层架构分为表述层(或表示层)、业务逻辑层、数据访问层,表述层表示前台页面和后台 servlet

### SpringMVC 的特点

- **Spring 家族原生产品**,与 IOC 容器等基础设施无缝对接
- **基于原生的 Servlet**,通过了功能强大的**前端控制器 DispatcherServlet**,对请求和响应进行统一处理
- 表述层各细分领域需要解决的问题**全方位覆盖**,提供**全面解决方案**
- **代码清新简洁**,大幅度提升开发效率
- 内部组件化程度高,可插拔式组件**即插即用**,想要什么功能配置相应组件即可
- **性能卓著**,尤其适合现代大型、超大型互联网项目要求

---

## HelloWorld

### 开发环境

IDE:IDEA 2021.2.3

构建工具:maven 3.8.1

服务器:Tomcat 9

Spring版本:5.3.9

### 创建maven工程

#### a> 添加web模块

![image-202110291048833](https://gitee.com/DoubleZHEz/mine/raw/master/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/SpringMVC/202110291048833.png)

#### b> 打包方式:war

#### c> 引入依赖

```xml
    <!-- 打包方式设置为 war -->
    <packaging>war</packaging>

    <dependencies>
        <!-- SpringMVC -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.9</version>
        </dependency>

        <!-- 日志 -->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>

        <!-- ServletAPI -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>

        <!-- Spring5 和 Thymeleaf 整合包 -->
        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring5</artifactId>
            <version>3.0.12.RELEASE</version>
        </dependency>
    </dependencies>
```

注:由于 Maven 的传递性,我们不必将所有需要的包全部配置依赖,而是配置最顶端的依赖,其他靠传递性导入。

### 配置 web.xml

注册 SpringMVC 的前端控制器 DispatcherServlet

#### a> 默认配置方式

此配置作用下,SpringMVC 的配置文件默认位于 WEB-INF 下,默认名称为 web.xml,例如,以下配置所对应 SpringMVC 的配置文件位于 WEB-INF 下,文件名为 web.xml

```xml
<!-- 配置 SpringMVC 的前端控制器,对浏览器发送的请求统一进行处理 -->
<servlet>
    <servlet-name>springMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>springMVC</servlet-name>
    <!--
        设置 springMVC 的核心控制器所能处理的请求的请求路径
        / 所匹配的请求可以是 /login或.html 或 .js 或 .css 方式的请求路径
        但是 / 不能匹配 .jsp 请求路径的请求
    -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

#### b> 扩展配置方式

可通过 init-param 标签设置 SpringMVC 配置文件的位置和名称,通过 load-on-startup 标签设置 SpringMVC 前端控制器 DispatcherServlet 的初始化时间

```xml
<!-- 配置 SpringMVC 的前端控制器,对浏览器发送的请求统一进行处理 -->
<servlet>
    <servlet-name>springMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 通过初始化参数指定 SpringMVC 配置文件的位置和名称 -->
    <init-param>
        <!-- contextConfigLocation 为固定值 -->
        <param-name>contextConfigLocation</param-name>
        <!-- 使用 classpath: 表示从类路径查找配置文件,例如 maven 工程中的 src/main/resources -->
        <param-value>classpath:springMVC.xml</param-value>
    </init-param>
    <!-- 
 		作为框架的核心组件,在启动过程中有大量的初始化操作要做
		而这些操作放在第一次请求时才执行会严重影响访问速度
		因此需要通过此标签将启动控制 DispatcherServlet 的初始化时间提前到服务器启动时
	-->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springMVC</servlet-name>
    <!--
        设置 springMVC 的核心控制器所能处理的请求的请求路径
        / 所匹配的请求可以是 /login 或 .html 或 .js 或 .css 方式的请求路径
        但是 / 不能匹配 .jsp 请求路径的请求
    -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

> 注:
>
> `<url-pattern>`标签中使用 / 和 /* 的区别:
>
> / 所匹配的请求可以是 /login 或 .html 或 .js 或 .css 方式的请求路径,但是 / 不能匹配 .jsp 请求路径的请求
>
> 因此就可以避免在访问 jsp 页面时,该请求被 DispatcherServlet 处理,从而找不到相应的页面
>
> /* 则能够匹配所有请求,例如在使用过滤器时,若需要对所有请求进行过滤,就需要使用 /\* 的写法

### 创建请求控制器

由于前端控制器对浏览器发送的请求进行了统一的处理,但是具体的请求有不同的处理过程,因此需要创建处理具体请求的类,即请求控制器

请求控制器中每一个处理请求的方法成为控制器方法

因为SpringMVC的控制器由一个 POJO(普通的 Java 类)担任,因此需要通过 @Controller 注解将其标识为一个控制层组件,交给 Spring 的 IoC 容器管理,此时 SpringMVC 才能够识别控制器的存在

```java
import org.springframework.stereotype.Controller;

@Controller
public class HelloController {
}
```

### 创建 springMVC 的配置文件

```xml
    <!-- 扫描组件 -->
    <context:component-scan base-package="Spring.MVC.controller"/>

    <!-- 配置 Thymeleaf 视图解析器 -->
    <bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
        <property name="order" value="1"/>
        <property name="characterEncoding" value="UTF-8"/>
        <property name="templateEngine">
            <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                <property name="templateResolver">
                    <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">

                        <!-- 视图前缀 -->
                        <property name="prefix" value="/WEB-INF/templates/"/>

                        <!-- 视图后缀 -->
                        <property name="suffix" value=".html"/>
                        <property name="templateMode" value="HTML5"/>
                        <property name="characterEncoding" value="UTF-8" />
                    </bean>
                </property>
            </bean>
        </property>
    </bean>
```

### 测试 HelloWorld

#### a> 实现对首页的访问

在 /WEB-INF/templates/ 路径下创建首页 index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>
    <h1>首页</h1>
</body>
</html>
```

在请求控制器中创建处理请求的方法

```java
// @RequestMapping 注解:处理请求和控制器方法之间的映射关系
// @RequestMapping 注解的 value 属性可以通过请求地址匹配请求,/ 表示的当前工程的上下文路径
// localhost:8080/springMVC/
@RequestMapping("/")
public String index() {
    //设置视图名称
    return "index";
}
```

#### b> 通过超链接跳转到指定页面

在主页index.html中设置超链接

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>
    <h1>首页</h1>
    <a th:href="@{/target}">访问目标页面 target.html</a>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    HelloWorld!!!
</body>
</html>
```

在请求控制器中创建处理请求的方法

```java
@RequestMapping("/target")
public String HelloWorld() {
    return "target";
}
```

### 总结

浏览器发送请求,若请求地址符合前端控制器的 url-pattern,该请求就会被前端控制器 DispatcherServlet 处理。前端控制器会读取 SpringMVC 的核心配置文件,通过扫描组件找到控制器,将请求地址和控制器中 @RequestMapping 注解的 value 属性值进行匹配,若匹配成功,该注解所标识的控制器方法就是处理请求的方法。处理请求的方法需要返回一个字符串类型的视图名称,该视图名称会被视图解析器解析,加上前缀和后缀组成视图的路径,通过 Thymeleaf 对视图进行渲染,最终转发到视图所对应页面。

---

## @RequestMapping 注解

### 注解功能

从注解名称上我们可以看到,@RequestMapping 注解的作用就是将请求和处理请求的控制器方法关联起来,建立映射关系。

SpringMVC 接收到指定的请求,就会来找到在映射关系中对应的控制器方法来处理这个请求。

### 注解位置

@RequestMapping 标识一个类:设置映射请求的请求路径的初始信息

@RequestMapping 标识一个方法:设置映射请求请求路径的具体信息

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/hello")
public class RequestMappingController {

	//此时请求映射所映射的请求的请求路径为:/hello/testRequestMapping
    @RequestMapping("/testRequestMapping")
    public String testRequestMapping(){
        return "success";
    }

}
```

### value 属性

@RequestMapping 注解的  value 属性通过请求的请求地址匹配请求映射

@RequestMapping 注解的 value 属性是一个字符串类型的数组,表示该请求映射能够匹配多个请求地址所对应的请求

@RequestMapping 注解的 value 属性必须设置,至少通过请求地址匹配请求映射

```html
    <a th:href="@{/testRequestMapping}">测试 @RequestMapping 注解的 value 属性 ---> /testRequestMapping</a><br>
    <a th:href="@{/test}">测试 @RequestMapping 注解的 value 属性 ---> /test</a><br>
```

```java
    @RequestMapping(
            value = {"/testRequestMapping", "/test"}
    )
    public String success() {
        return "success";
    }
```

### method 属性

@RequestMapping 注解的 method 属性通过请求的请求方式( get 或 post )匹配请求映射

@RequestMapping 注解的 method 属性是一个 RequestMethod 类型的数组,表示该请求映射能够匹配多种请求方式的请求

若当前请求的请求地址满足请求映射的 value 属性,但是请求方式不满足 method 属性,则浏览器报错 405:Request method 'POST' not supported

```html
    <a th:href="@{/test}">测试 @RequestMapping 注解的 method 属性 ---> GET</a><br>

    <form th:action="@{/test}" method="post">
        <input type="submit" value="测试 @RequestMapping 注解的 method 属性 ---> POST">
    </form>
```

```java
    @RequestMapping(
            value = {"/testRequestMapping", "/test"},
            method = {RequestMethod.GET, RequestMethod.POST}
    )
    public String testRequestMapping(){
        return "success";
    }
```

> 注:
>
> 1、对于处理指定请求方式的控制器方法,SpringMVC 中提供了 @RequestMapping 的派生注解
>
> 处理 get 请求的映射 ---> @GetMapping
>
> 处理 post 请求的映射 ---> @PostMapping
>
> 处理 put 请求的映射 ---> @PutMapping
>
> 处理 delete 请求的映射 ---> @DeleteMapping
>
> 2、常用的请求方式有 get,post,put,delete
>
> 但是目前浏览器只支持 get 和 post,若在 form 表单提交时,为 method 设置了其他请求方式的字符串(put 或 delete),则按照默认的请求方式 get 处理
>
> 若要发送 put 和 delete 请求,则需要通过 spring 提供的过滤器 HiddenHttpMethodFilter,在 RESTful 部分会讲到

### params 属性(了解)

@RequestMapping 注解的 params 属性通过请求的请求参数匹配请求映射

@RequestMapping 注解的 params 属性是一个字符串类型的数组,可以通过四种表达式设置请求参数和请求映射的匹配关系

"param":要求请求映射所匹配的请求必须携带 param 请求参数

"!param":要求请求映射所匹配的请求必须不能携带 param 请求参数

"param=value":要求请求映射所匹配的请求必须携带 param 请求参数且 param = value

"param!=value":要求请求映射所匹配的请求必须携带 param 请求参数但是 param != value

```html
<a th:href="@{/testParamsAndHeaders(username='admin', password=123456)}">测试 @RequestMapping 注解的 params 属性 ---> /testParamsAndHeaders</a><br>
```

```java
    @RequestMapping(
            value = "/testParamsAndHeaders",
            params = {"username", "password=!123456"}
    )
    public String testParamsAndHeaders() {
        return "success";
    }
```

> 注:
>
> 若当前请求满足 @RequestMapping 注解的 value 和 method 属性,但是不满足 params 属性,此时页面回报错 400:Parameter conditions "username, password!=123456" not met for actual request parameters: username={admin}, password={123456}

### headers 属性(了解)

@RequestMapping 注解的 headers 属性通过请求的请求头信息匹配请求映射

@RequestMapping 注解的 headers 属性是一个字符串类型的数组,可以通过四种表达式设置请求头信息和请求映射的匹配关系

"header":要求请求映射所匹配的请求必须携带 header 请求头信息

"!header":要求请求映射所匹配的请求必须不能携带 header 请求头信息

"header=value":要求请求映射所匹配的请求必须携带 header 请求头信息且 header = value

"header!=value":要求请求映射所匹配的请求必须携带 header 请求头信息且 header != value

若当前请求满足 @RequestMapping 注解的 value 和 method 属性,但是不满足 headers 属性,此时页面显示 404 错误,即资源未找到

```java
    @RequestMapping(
            value = "/testParamsAndHeaders",
            params = {"username", "password=123456"},
            headers = {"Host=localhost:8081"}   //Host=localhost:8080 这是正确的参数值。localhost 是 Tomcat 默认地址;8080 是 Tomcat 默认的端口号。
    )
    public String testParamsAndHeaders() {
        return "success";
    }
```

### SpringMVC 支持 ant 风格的路径

?:表示任意的单个字符。不支持 / , ? 之类的特殊字符

*:表示任意的 0 个或多个字符。不支持 / , ? 之类的特殊字符

**:表示任意的一层或多层目录

注意:在使用 `**` 时,只能使用 /**/xxx 的方式

```java
    @RequestMapping("/a?a/testAnt1")
    public String testAnt1() {
        return "success";
    }

    @RequestMapping("/a*a/testAnt2")
    public String testAnt2() {
        return "success";
    }

    @RequestMapping("/**/testAnt3") //@RequestMapping("/a**a/testAnt3") 这是错误的写法,**只可以单独使用,不可和其它字符拼接使用,不然就是被浏览器解析为单独的一个 *
    public String testAnt3() {
        return "success";
    }
```

```html
    <a th:href="@{/a1a/testAnt1}">测试 @RequestMapping 注解可以匹配 ant 风格的路径 ---> 使用 ?</a><br>
    <a th:href="@{/a123a/testAnt2}">测试 @RequestMapping 注解可以匹配 ant 风格的路径 ---> 使用 *</a><br>
    <a th:href="@{/a123a/testAnt3}">测试 @RequestMapping 注解可以匹配 ant 风格的路径 ---> 使用 **</a><br>
```

### SpringMVC 支持路径中的占位符(重点)

原始方式:/deleteUser?id=1

rest 方式:/deleteUser/1

SpringMVC 路径中的占位符常用于 RESTful 风格中,当请求路径中将某些数据通过路径的方式传输到服务器中,就可以在相应的 @RequestMapping 注解的 value 属性中通过占位符 {xxx} 表示传输的数据,在通过 @PathVariable 注解,将占位符所表示的数据赋值给控制器方法的形参

```html
    <a th:href="@{/testPath/1/admin}">测试 @RequestMapping 注解支持路径中的占位符 ---> /testPath</a><br>
```

```java
    @RequestMapping("/testPath/{id}/{username}")
    public String testPath(@PathVariable("id")Integer id, @PathVariable("username")String username) {
        System.out.println("id = " + id + ", username = " + username);
        return "success";
    }
//最终输出的内容为 ---> id = 1, username = admin
```

---

## SpringMVC 获取请求参数

### 通过 ServletAPI 获取

将 HttpServletRequest 作为控制器方法的形参,此时 HttpServletRequest 类型的参数表示封装了当前请求的请求报文的对象

```java
    @RequestMapping("/testServletAPI")
    //形参位置的 request 表示当前请求
    public String testServletAPI(HttpServletRequest request) {
        String username = request.getParameter("username");
        String password = request.getParameter("password");

        System.out.println("username = " + username + ", password = " + password);

        return "success";
    }
```

```html
    <a th:href="@{/testServletAPI(username='admin', password=123456)}">测试使用 servletAPI 获取请求参数</a><br>
```

### 通过控制器方法的形参获取请求参数

在控制器方法的形参位置,设置和请求参数同名的形参,当浏览器发送请求,匹配到请求映射时,在DispatcherServlet中就会将请求参数赋值给相应的形参

```html
    <a th:href="@{/testParam1(username='admin', password=123456)}">测试使用控制器的形参获取请求参数</a><br>

    <form th:action="@{/testParam2}" method="get">
        用户名:<input type="text" name="user_name"><br>
        密码:<input type="password" name="password"><br>
        兴趣爱好:<input type="checkbox" name="hobby" value="a">a
        <input type="checkbox" name="hobby" value="b">b
        <input type="checkbox" name="hobby" value="c">c<br>
        <input type="submit" value="测试使用控制器的形参获取请求参数">
    </form>
```

```java
    @RequestMapping("/testParam1")
    public String testParam1(String username, String password) {
        System.out.println("username = " + username + ", password = " + password);
        return "success";
    }

    @RequestMapping("/testParam2")
    public String testParam1(String username, String password, String[] hobby) {
        //若请求参数中出现多个同名的请求参数,可以在控制器方法的形参位置设置字符串类型或字符串数组接受此请求参数
        //若使用字符串类型的形参,最终结果为请求参数的每一个值之间使用逗号进行拼接
        System.out.println("username = " + username + ", password = " + password + ", hobby = " + Arrays.toString(hobby));
        return "success";
    }
```

> 注:
>
> 若请求所传输的请求参数中有多个同名的请求参数,此时可以在控制器方法的形参中设置字符串数组或者字符串类型的形参接收此请求参数
>
> 若使用字符串数组类型的形参,此参数的数组中包含了每一个数据
>
> 若使用字符串类型的形参,此参数的值为每个数据中间使用逗号拼接的结果

### @RequestParam

@RequestParam 是将请求参数和控制器方法的形参创建映射关系

@RequestParam 注解一共有三个属性:

value:指定为形参赋值的请求参数的参数名

required:设置是否必须传输此请求参数,默认值为 true

若设置为 true 时,则当前请求必须传输 value 所指定的请求参数,若没有传输该请求参数,且没有设置 defaultValue 属性,则页面报错 400:Required String parameter 'xxx' is not present;若设置为 false,则当前请求不是必须传输 value 所指定的请求参数,若没有传输,则注解所标识的形参的值为 null

defaultValue:不管 required 属性值为 true 或 false,当 value 所指定的请求参数没有传输或传输的值为 "" 时,则使用默认值为形参赋值

```java
    @RequestMapping("/testParam3")
    public String testParam2(
            @RequestParam(value = "user_name", required = false, defaultValue = "niubi") String username,
            String password,
            String[] hobby) {
        //若请求参数中出现多个同名的请求参数,可以在控制器方法的形参位置设置字符串类型或字符串数组接受此请求参数
        //若使用字符串类型的形参,最终结果为请求参数的每一个值之间使用逗号进行拼接
        System.out.println("username = " + username + ", password = " + password + ", hobby = " + Arrays.toString(hobby));
        return "success";
    }
```

```html
    <form th:action="@{/testParam2}" method="get">
        用户名:<input type="text" name="user_name"><br>
        密码:<input type="password" name="password"><br>
        兴趣爱好:<input type="checkbox" name="hobby" value="a">a
        <input type="checkbox" name="hobby" value="b">b
        <input type="checkbox" name="hobby" value="c">c<br>
        <input type="submit" value="测试使用控制器的形参获取请求参数">
    </form>
```

### @RequestHeader

@RequestHeader 是将请求头信息和控制器方法的形参创建映射关系

@RequestHeader 注解一共有三个属性:value、required、defaultValue,用法同 @RequestParam

```html
    <form th:action="@{/testHeader}" method="get">
        用户名:<input type="text" name="user_name"><br>
        密码:<input type="password" name="password"><br>
        兴趣爱好:<input type="checkbox" name="hobby" value="a">a
        <input type="checkbox" name="hobby" value="b">b
        <input type="checkbox" name="hobby" value="c">c<br>
        <input type="submit" value="测试使用控制器的形参获取请求参数">
    </form>
```

```java
    @RequestMapping("/testHeader")
    public String testHeader(
            @RequestParam(value = "user_name", required = false, defaultValue = "niubi") String username,
            String password,
            String[] hobby,
            @RequestHeader(value = "sayHaha", required = true, defaultValue = "haha") String host) {
        //若请求参数中出现多个同名的请求参数,可以在控制器方法的形参位置设置字符串类型或字符串数组接受此请求参数
        //若使用字符串类型的形参,最终结果为请求参数的每一个值之间使用逗号进行拼接
        System.out.println("username = " + username + ", password = " + password + ", hobby = " + Arrays.toString(hobby));
        System.out.println("host = " + host);
        return "success";
    }
```

>结果:
>
>username = admin, password = 123, hobby = [a, b]
>
>host = haha

### @CookieValue

@CookieValue 是将 cookie 数据和控制器方法的形参创建映射关系

@CookieValue 注解一共有三个属性:value、required、defaultValue,用法同 @RequestParam

```html
    <form th:action="@{/testCookie}" method="get">
        用户名:<input type="text" name="user_name"><br>
        密码:<input type="password" name="password"><br>
        兴趣爱好:<input type="checkbox" name="hobby" value="a">a
        <input type="checkbox" name="hobby" value="b">b
        <input type="checkbox" name="hobby" value="c">c<br>
        <input type="submit" value="测试使用控制器的形参获取请求参数">
    </form>
```

```java
    @RequestMapping("/testCookie")
    public String testCookie(
            @RequestParam(value = "user_name", required = false, defaultValue = "niubi") String username,
            String password,
            String[] hobby,
            @RequestHeader(value = "sayHaha", required = true, defaultValue = "haha") String host,
            @CookieValue("JSESSIONID") String JSESSIONID) {
        //若请求参数中出现多个同名的请求参数,可以在控制器方法的形参位置设置字符串类型或字符串数组接受此请求参数
        //若使用字符串类型的形参,最终结果为请求参数的每一个值之间使用逗号进行拼接
        System.out.println("username = " + username + ", password = " + password + ", hobby = " + Arrays.toString(hobby));
        System.out.println("host = " + host);
        System.out.println("JSESSIONID = " + JSESSIONID);
        return "success";
    }
```

>username = admin, password = 456, hobby = [a, b, c]
>
>host = haha
>
>JSESSIONID = 01261CB86816674AE6576B1C88C9D368

### 通过 POJO 获取请求参数

可以在控制器方法的形参位置设置一个实体类类型的形参,此时若浏览器传输的请求参数的参数名和实体类中的属性名一致,那么请求参数就会为此属性赋值

```html
    <form th:action="@{/testBean}" method="post">   <!-- post 有可能会乱码,但是 get 不会产生乱码 -->
        用户名:<input type="text" name="username"><br>
        密码:<input type="password" name="password"><br>
        性别:<input type="radio" name="sex" value="男">男<input type="radio" name="sex" value="女">女<br>
        年龄:<input type="text" name="age"><br>
        邮箱:<input type="text" name="email"><br>
        <input type="submit" value="使用实体类接受请求参数">
    </form>
```

```java
    @RequestMapping("/testBean")
    public String testBean(User user) {
        System.out.println(user);
        return "success";
    }
```

>结果:
>
>User{id=null, username='??????', password='962464', sex='??·', age=20, email='Double_ZHE@outlook.com'}
>
>post 有可能会乱码,但是 get 不会产生乱码

### 解决获取请求参数的乱码问题

解决获取请求参数的乱码问题,可以使用 SpringMVC 提供的编码过滤器 CharacterEncodingFilter,但是必须在 web.xml 中进行注册

```xml
    <!--配置 springMVC 的编码过滤器-->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>

        <init-param>
            <param-name>forceResponseEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

>结果:
>
>User{id=null, username='任喆喆', password='962464', sex='男', age=20, email='Double_ZHE@outlook.com'}

> 注:
>
> SpringMVC 中处理编码的过滤器一定要配置到其他过滤器之前,否则无效

>可以修改 Tomcat 的配置文件从根本上解决这个乱码问题
```xml
    <!-- 原来的配置 -->
    <!-- <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" /> -->

    <!-- 修改后的配置 -->
    <Connector port="8080" URLEncoding="UTF-8" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

---

## 域对象共享数据

### 使用 ServletAPI 向 request 域对象共享数据

```html
    <a th:href="@{/testRequestByServletAPI}">使用 ServletAPI 向 Request 域对象共享数据</a><br>
```

```html
    success<br>
    <p th:text="${testRequestScope}"></p>
```

```java
    @RequestMapping("/testRequestByServletAPI")
    public String testRequestByServletAPI(HttpServletRequest request) {
        request.setAttribute("testRequestScope", "hello, servletAPI");
        return "success";
    }
```

### 使用 ModelAndView 向 request 域对象共享数据

```html
    <a th:href="@{/testModelAndView}">使用 ModelAndView 向 Request 域对象共享数据</a><br>
```

```java
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
    /**
     * ModelAndView 有 Model 和 View 的功能
     * Model 主要用于向请求域共享数据
     * View 主要用于设置视图,实现页面跳转
     */
    ModelAndView mav = new ModelAndView();
    //向请求域共享数据
    mav.addObject("testRequestScope", "Hello, ModelAndView");
    //设置视图,实现页面跳转
    mav.setViewName("success");
    return mav;
}
```

### 使用 Model 向 request 域对象共享数据

```html
    <a th:href="@{/testModel}">使用 Model 向 Request 域对象共享数据</a><br>
```

```java
@RequestMapping("/testModel")
public String testModel(Model model){
    model.addAttribute("testRequestScope", "Hello, Model");
    return "success";
}
```

### 使用 map 向 request 域对象共享数据

```java
@RequestMapping("/testMap")
public String testMap(Map<String, Object> map){
    map.put("testRequestScope", "Hello, Map");
    return "success";
}
```

### 使用 ModelMap 向 request 域对象共享数据

```java
@RequestMapping("/testModelMap")
public String testModelMap(ModelMap modelMap){
    modelMap.addAttribute("testRequestScope", "Hello, ModelMap");
    return "success";
}
```

### Model、ModelMap、Map 的关系

Model、ModelMap、Map 类型的参数其实本质上都是 BindingAwareModelMap 类型的

```java
public interface Model{}
public class ModelMap extends LinkedHashMap<String, Object> {}
public class ExtendedModelMap extends ModelMap implements Model {}
public class BindingAwareModelMap extends ExtendedModelMap {}
```

### 向 session 域共享数据

```html
<p th:text="${session.testSessionScope}"></p>
```

```java
@RequestMapping("/testSession")
public String testSession(HttpSession session){
    session.setAttribute("testSessionScope", "Hello, session");
    return "success";
}
```

### 向 application 域共享数据

```html
    <a th:href="@{/testApplication}">使用 ServletAPI 向 application 域对象共享数据</a><br>
```

```html
    <p th:text="${application.testApplication}"></p>
```

```java
@RequestMapping("/testApplication")
public String testApplication(HttpSession session) {
    ServletContext application = session.getServletContext();
    application.setAttribute("testApplication", "Hello, Application");
    return "success";
}
```

![](https://gitee.com/DoubleZHEz/mine/raw/master/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/SpringMVC/202111052131426.png)

---

## SpringMVC 的视图

SpringMVC 中的视图是 View 接口,视图的作用渲染数据,将模型 Model 中的数据展示给用户

SpringMVC 视图的种类很多,默认有转发视图 InternalResourceView 和重定向视图 RedirectView

当工程引入 jstl 的依赖,转发视图会自动转换为 JstlView

若使用的视图技术为 Thymeleaf,在 SpringMVC 的配置文件中配置了 Thymeleaf 的视图解析器,由此视图解析器解析之后所得到的是 ThymeleafView

### ThymeleafView

当控制器方法中所设置的视图名称没有任何前缀时,此时的视图名称会被 SpringMVC 配置文件中所配置的视图解析器解析,视图名称拼接视图前缀和视图后缀所得到的最终路径,会通过转发的方式实现跳转

```html
<a th:href="@{/testThymeleafView}">测试 ThymeleafView</a><br>
```

```java
    @RequestMapping("/testThymeleafView")
    public String testThymeleafView() {
        return "success";
    }
```

![](https://gitee.com/DoubleZHEz/mine/raw/master/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/SpringMVC/202111071413874.png)

### 转发视图

SpringMVC 中默认的转发视图是 InternalResourceView

SpringMVC 中创建转发视图的情况:

当控制器方法中所设置的视图名称以 "forward:" 为前缀时,创建 InternalResourceView 视图,此时的视图名称不会被 SpringMVC 配置文件中所配置的视图解析器解析,而是会将前缀 "forward:" 去掉,剩余部分作为最终路径通过转发的方式实现跳转

例如 "forward:/","forward:/employee"

```html
<a th:href="@{/testForward}">测试 InternalResourceView</a><br>
```

```java
    @RequestMapping("/testForward")
    public String testForward() {
        return "forward:/testThymeleafView";
    }
```

![](https://gitee.com/DoubleZHEz/mine/raw/master/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/SpringMVC/202111071414792.png)

### 重定向视图

SpringMVC 中默认的重定向视图是 RedirectView

当控制器方法中所设置的视图名称以 "redirect:" 为前缀时,创建 RedirectView 视图,此时的视图名称不会被 SpringMVC 配置文件中所配置的视图解析器解析,而是会将前缀 "redirect:" 去掉,剩余部分作为最终路径通过重定向的方式实现跳转

例如 "redirect:/","redirect:/employee"

```html
<a th:href="@{/testRedirect}">测试 RedirectView</a><br>
```

```java
    @RequestMapping("/testRedirect")
    public String testRedirect() {
        return "redirect:/testThymeleafView";
    }
```

![](https://gitee.com/DoubleZHEz/mine/raw/master/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/SpringMVC/202111071414378.png)

> 注:
>
> 重定向视图在解析时,会先将 redirect: 前缀去掉,然后会判断剩余部分是否以 / 开头,若是则会自动拼接上下文路径

### 视图控制器 view-controller

当控制器方法中,仅仅用来实现页面跳转,即只需要设置视图名称时,可以将处理器方法使用 view-controller 标签进行表示

```xml
<!--
    Spring MVC 配置文件中
	path:设置处理的请求地址
	view-name:设置请求地址所对应的视图名称
-->
<mvc:view-controller path="/" view-name="index"/>   <!-- IDEA 2021.2.3 版本中 view-name属性值 index 报红,但是不影响解析运行。具体原因不明 -->
```

> 注:
>
> 当SpringMVC中设置任何一个 view-controller 时,其他控制器中的请求映射将全部失效,此时需要在 SpringMVC 的核心配置文件中设置开启 mvc 注解驱动的标签:
>
> <mvc:annotation-driven />

---

## RESTful

### RESTful简介

REST:**Re**presentational **S**tate **T**ransfer,表现层资源状态转移。

##### a> 资源

资源是一种看待服务器的方式,即,将服务器看作是由很多离散的资源组成。每个资源是服务器上一个可命名的抽象概念。因为资源是一个抽象的概念,所以它不仅仅能代表服务器文件系统中的一个文件、数据库中的一张表等等具体的东西,可以将资源设计的要多抽象有多抽象,只要想象力允许而且客户端应用开发者能够理解。与面向对象设计类似,资源是以名词为核心来组织的,首先关注的是名词。一个资源可以由一个或多个 URI 来标识。URI 既是资源的名称,也是资源在 Web 上的地址。对某个资源感兴趣的客户端应用,可以通过资源的 URI 与其进行交互。

##### b> 资源的表述

资源的表述是一段对于资源在某个特定时刻的状态的描述。可以在客户端-服务器端之间转移(交换)。资源的表述可以有多种格式,例如 HTML/XML/JSON/纯文本/图片/视频/音频 等等。资源的表述格式可以通过协商机制来确定。请求-响应方向的表述通常使用不同的格式。

##### c> 状态转移

状态转移说的是:在客户端和服务器端之间转移(transfer)代表资源状态的表述。通过转移和操作资源的表述,来间接实现操作资源的目的。

### RESTful的实现

具体说,就是 HTTP 协议里面,四个表示操作方式的动词:GET、POST、PUT、DELETE。

它们分别对应四种基本操作:GET 用来获取资源,POST 用来新建资源,PUT 用来更新资源,DELETE 用来删除资源。

REST 风格提倡 URL 地址使用统一的风格设计,从前到后各个单词使用斜杠分开,不使用问号键值对方式携带请求参数,而是将要发送给服务器的数据作为 URL 地址的一部分,以保证整体风格的一致性。

| 操作     | 传统方式         | REST风格                |
| -------- | ---------------- | ----------------------- |
| 查询操作 | getUserById?id=1 | user/1-->get请求方式    |
| 保存操作 | saveUser         | user-->post请求方式     |
| 删除操作 | deleteUser?id=1  | user/1-->delete请求方式 |
| 更新操作 | updateUser       | user-->put请求方式      |

```xml
    <!-- 在 springMVC.xml 文件中配置视图控制器 -->
    <mvc:view-controller path="/" view-name="index"/>   <!-- IDEA 2021.2.3 报红，但是不影响解析，具体原因不明 -->
    <mvc:view-controller path="/test_view" view-name="test_view"/>   <!-- IDEA 2021.2.3 报红，但是不影响解析，具体原因不明 -->
    <mvc:view-controller path="/test_rest" view-name="test_rest"/>   <!-- IDEA 2021.2.3 报红，但是不影响解析，具体原因不明 -->
```

```html
<a th:href="@{/users}">查询所有用户信息</a><br>
<a th:href="@{/user/1}">根据 ID 查询所有用户信息</a><br>
<form th:action="@{/user}" method="post">
    用户名:<input type="text" name="username"><br>
    密码:<input type="password" name="password"><br>
    <input type="submit" value="添加">
</form>
```

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

/**
 * @ClassName UserController
 * @Description 使用 RESTFul 模拟用户资源的增删改查
 *                  /users    GET     查询所有用户信息
 *                  /user/1    GET     根据用户 ID 查询用户信息
 *                  /user    POST     添加用户信息
 *                  /user/1    DELETE     删除用户信息
 *                  /user    PUT     更新用户信息
 * @Author DoubleZHE
 * @Create 2021/11/9 20:55
 */

@Controller
public class UserController {
    @RequestMapping(value = "/users", method = RequestMethod.GET)
    public String getAllUser() {
        System.out.println("查询所有用户信息");
        return "success";
    }

    @RequestMapping(value = "/user/{id}", method = RequestMethod.GET)
    public String getUserByID(@PathVariable String id) {
        System.out.println("根据用户 ID 查询用户信息");
        return "success";
    }

    @RequestMapping(value = "/user", method = RequestMethod.POST)
    public String insertUser(String username, String password) {
        System.out.println("添加用户信息:" + username + ", " + password);
        return "success";
    }
}
```

### HiddenHttpMethodFilter

由于浏览器只支持发送 get 和 post 方式的请求,那么该如何发送 put 和 delete 请求呢?

SpringMVC 提供了 **HiddenHttpMethodFilter** 帮助我们**将 POST 请求转换为 DELETE 或 PUT 请求**

**HiddenHttpMethodFilter** 处理 put 和 delete 请求的条件:

a> 当前请求的请求方式必须为 post

b> 当前请求必须传输请求参数 _method

满足以上条件,**HiddenHttpMethodFilter** 过滤器就会将当前请求的请求方式转换为请求参数 _method 的值,因此请求参数 \_method 的值才是最终的请求方式

在 web.xml 中注册 **HiddenHttpMethodFilter** 

```xml
    <!-- 配置 HiddenHttpMethodFilter -->
    <filter>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

```html
<form th:action="@{/user}" method="post">
    <input type="hidden" name="_method" value="PUT">
    用户名:<input type="text" name="username"><br>
    密码:<input type="password" name="password"><br>
    <input type="submit" value="修改">
</form>
```

> 注:
>
> 目前为止,SpringMVC 中提供了两个过滤器:CharacterEncodingFilter 和 HiddenHttpMethodFilter
>
> 在 web.xml 中注册时,必须先注册 CharacterEncodingFilter,再注册 HiddenHttpMethodFilter
>
> 原因:
>
> - 在 CharacterEncodingFilter 中通过 request.setCharacterEncoding(encoding) 方法设置字符集的
>
> - request.setCharacterEncoding(encoding) 方法要求前面不能有任何获取请求参数的操作
>
> - 而 HiddenHttpMethodFilter 恰恰有一个获取请求方式的操作:
>
> - ```
>   String paramValue = request.getParameter(this.methodParam);
>   ```



## RESTful案例

### 1、准备工作

和传统 CRUD 一样,实现对员工信息的增删改查。

- 搭建环境

    web.xml

    ```xml
    <!-- 配置编码过滤器 -->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceResponseEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- 配置处理请求方式 PUT 和 DELETE 的 HiddenHttpMethodFilter -->
    <filter>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- 配置 springMVC 的前端控制器 DispatcherServlet -->
    <servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springMVC.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup><!-- 将前端控制器的初始化时间提前到服务器启动时 -->
    </servlet>
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    ```

    SpringMVC 配置文件 springMVC.xml

    ```xml
    <!-- 开启组件扫描 -->
    <context:component-scan base-package="RESTFul.controller"/>

    <!-- 配置 Thymeleaf 视图解析器 -->
    <bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
        <property name="order" value="1"/>
        <property name="characterEncoding" value="UTF-8"/>
        <property name="templateEngine">
            <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
                <property name="templateResolver">
                    <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
                        <!-- 视图前缀 -->
                        <property name="prefix" value="/WEB-INF/templates/"/>

                        <!-- 视图后缀 -->
                        <property name="suffix" value=".html"/>
                        <property name="templateMode" value="HTML5"/>
                        <property name="characterEncoding" value="UTF-8"/>
                    </bean>
                </property>
            </bean>
        </property>
    </bean>
    ```

- 准备实体类

  ```java
  package RESTFul.bean;
  
  public class Employee {
  
     private Integer id;
     private String lastName;
  
     private String email;
     //1 male, 0 female
     private Integer gender;
     
     public Integer getId() {
        return id;
     }
  
     public void setId(Integer id) {
        this.id = id;
     }
  
     public String getLastName() {
        return lastName;
     }
  
     public void setLastName(String lastName) {
        this.lastName = lastName;
     }
  
     public String getEmail() {
        return email;
     }
  
     public void setEmail(String email) {
        this.email = email;
     }
  
     public Integer getGender() {
        return gender;
     }
  
     public void setGender(Integer gender) {
        this.gender = gender;
     }
  
     public Employee(Integer id, String lastName, String email, Integer gender) {
        super();
        this.id = id;
        this.lastName = lastName;
        this.email = email;
        this.gender = gender;
     }
  
     public Employee() {
     }
  }
  ```

- 准备 dao 模拟数据

  ```java
  package RESTFul.dao;

  import RESTFul.bean.Employee;
  import org.springframework.stereotype.Repository;

  import java.util.Collection;
  import java.util.HashMap;
  import java.util.Map;

  @Repository
  public class EmployeeDao {

      private static Map<Integer, Employee> employees = null;

      static{
          employees = new HashMap<Integer, Employee>();

          employees.put(1001, new Employee(1001, "E-AA", "aa@163.com", 1));
          employees.put(1002, new Employee(1002, "E-BB", "bb@163.com", 1));
          employees.put(1003, new Employee(1003, "E-CC", "cc@163.com", 0));
          employees.put(1004, new Employee(1004, "E-DD", "dd@163.com", 0));
          employees.put(1005, new Employee(1005, "E-EE", "ee@163.com", 1));
      }

      private static Integer initId = 1006;

      public void save(Employee employee){
          if(employee.getId() == null){
              employee.setId(initId++);
          }
          employees.put(employee.getId(), employee);
      }

      public Collection<Employee> getAll(){
          return employees.values();
      }

      public Employee get(Integer id){
          return employees.get(id);
      }

      public void delete(Integer id){
          employees.remove(id);
      }
  }
  ```

### 2、功能清单

| 功能                | URL 地址    | 请求方式 |
| ------------------- | ----------- | -------- |
| 访问首页√           | /           | GET      |
| 查询全部数据√       | /employee   | GET      |
| 删除√               | /employee/2 | DELETE   |
| 跳转到添加数据页面√ | /toAdd      | GET      |
| 执行保存√           | /employee   | POST     |
| 跳转到更新数据页面√ | /employee/2 | GET      |
| 执行更新√           | /employee   | PUT      |

### 3、具体功能:访问首页

##### a> 配置 view-controller

springMVC.xml

```xml
<!-- 配置视图控制器 -->
<mvc:view-controller path="/" view-name="index"/>   <!-- IDEA 2021.2.3 版本中 view-name 标签属性值报红，但是不影响解析，具体原因不明 -->

<!-- 开启 MVC 的注解驱动 -->
<mvc:annotation-driven/>
```

##### b> 创建页面

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8" >
    <title>Title</title>
</head>
<body>
<h1>首页</h1>
<a th:href="@{/employee}">访问员工信息</a>
</body>
</html>
```

### 4、具体功能:查询所有员工数据

##### a> 控制器方法

```java
import RESTFul.bean.Employee;
import RESTFul.dao.EmployeeDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Repository;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import java.util.Collection;

@Repository
@Controller
public class EmployeeController {
    @Autowired
    private EmployeeDao employeeDao;

    @RequestMapping(value = "/employee", method = RequestMethod.GET)
    public String getAllEmployee(Model model) {
        Collection<Employee> employeeCollection = employeeDao.getAll();
        model.addAttribute("employeeCollection", employeeCollection);
        return "employee_collection";
    }
}

```

##### b> 创建 employee_collection.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Employee Info</title>
</head>
<body>

<table id="dataTable" border="1" cellspacing="0" cellpadding="0" style="text-align: center;">
    <tr>
        <th colspan="5">Employee Info</th>  <!-- colspan="5" 规定单元格应该横跨的列数。注释：colspan="0" 告知浏览器使单元格横跨到列组 (colgroup) 的最后一列。 -->
    </tr>
    <tr>
        <th>id</th>
        <th>lastName</th>
        <th>email</th>
        <th>gender</th>
        <th>options(<a th:href="@{/toAdd}">add</a>)</th>
    </tr>
    <tr th:each="employee : ${employeeCollection}"> <!-- IDEA 2021.2.3 报红，但是不影响解析，具体原因不明 -->
        <td th:text="${employee.id}"></td>  <!-- IDEA 2021.2.3 报红，但是不影响解析，具体原因不明 -->
        <td th:text="${employee.lastName}"></td>    <!-- IDEA 2021.2.3 报红，但是不影响解析，具体原因不明 -->
        <td th:text="${employee.email}"></td>   <!-- IDEA 2021.2.3 报红，但是不影响解析，具体原因不明 -->
        <td th:text="${employee.gender}"></td>  <!-- IDEA 2021.2.3 报红，但是不影响解析，具体原因不明 -->
        <td>
            <a href="">delete</a>
            <a href="">update</a>
        </td>
    </tr>
</table>

</body>
</html>
```

### 5、具体功能:删除

##### a> 创建处理 delete 请求方式的表单

```html
<!-- 作用:通过超链接控制表单的提交,将 post 请求转换为 delete 请求 -->
<form id="delete_form" method="post">
    <!-- HiddenHttpMethodFilter 要求:必须传输 _method 请求参数,并且值为最终的请求方式 -->
    <input type="hidden" name="_method" value="delete"/>
</form>
```

##### b> 删除超链接绑定点击事件

引入 vue.js

```html
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script>  <!-- vue 下载地址：https://github.com/vuejs/vue/releases 只需要其中的 vue.js 文件 -->
```

删除超链接

```html
<a @click="deleteEmployee" th:href="@{'/employee/' + ${employee.id}}">delete</a>    <!-- IDEA 2021.2.3 报红，但是不影响解析，具体原因不明 -->
```

通过 vue 处理点击事件

```html
<script type="text/javascript">
    var vue = new Vue({
        el:"#dataTable",    /* el：提供一个在页面上已存在的 DOM 元素作为 Vue 实例的挂载目标。https://cn.vuejs.org/v2/api/#el */
        methods:{
            deleteEmployee:function (event) {
                var deleteForm = document.getElementById("deleteForm"); //根据 id 获取表单元素
                deleteForm.action = event.target.href;  //将触发点击事件的超链接的 href 属性赋值给表单的 action
                deleteForm.submit();    //提交表单
                event.preventDefault(); //取消超链接的默认行为
            }
        }
    });
</script>
```

>引入 vue.js 后需要重新打包,不然会报 404
>
>![](https://gitee.com/DoubleZHEz/mine/raw/master/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/SpringMVC/SpringMVC/202111122128207.png)
>
>还得在 springMVC.xml 文件中添加一行代码，不然也会报 404
>
>```
>   <!-- 开放对静态资源的访问
>          先是 SpringMVC 的 DispatcherServlet 来处理，如果找不到，则交给默认的 DefaultServlet 来处理。
>          如若都找不到则报 DefaultServletHttpRequestHandler 错误。（日志功能）
>   -->
>   <mvc:default-servlet-handler/>
>
>   <!-- 开启 MVC 的注解驱动 -->
>   <mvc:annotation-driven/>
>
>```

##### c> 控制器方法

```java
import org.springframework.web.bind.annotation.PathVariable;

@RequestMapping(value = "/employee/{id}", method = RequestMethod.DELETE)
public String deleteEmployee(@PathVariable("id") Integer id) {
    employeeDao.delete(id);
    return "redirect:/employee";
}
```

### 6、具体功能:跳转到添加数据页面

##### a> 配置 view-controller

```xml
<mvc:view-controller path="/toAdd" view-name="employee_add"/>   <!-- IDEA 2021.2.3 版本中 view-name 标签属性值报红，但是不影响解析，具体原因不明 -->
```

##### b> 创建 employee_add.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>add employee</title>
</head>
<body>

<form th:action="@{/employee}" method="post">
    lastName:<input type="text" name="lastName"><br>
    email:<input type="text" name="email"><br>
    gender:<input type="radio" name="gender" value="1">male
    <input type="radio" name="gender" value="0">female<br>
    <input type="submit" name="add"><br>
</form>

</body>
</html>
```

### 7、具体功能:执行保存

##### a> 控制器方法

```java
@RequestMapping(value = "/employee", method = RequestMethod.POST)
public String addEmployee(Employee employee) {
    employeeDao.save(employee);
    return "redirect:/employee";
}
```

### 8、具体功能:跳转到更新数据页面

##### a> 修改超链接

```html
<a th:href="@{'/employee/' + ${employee.id}}">update</a>    <!-- IDEA 2021.2.3 对 Thymeleaf 报红，但是不影响解析，具体原因不明 -->
```

##### b> 控制器方法

```java
@RequestMapping(value = "/employee/{id}", method = RequestMethod.GET)
public String getEmployeeByID(@PathVariable("id") Integer id, Model model) {
    Employee employee = employeeDao.get(id);
    model.addAttribute("employee", employee);
    return "employee_update";
}
```

##### c> 创建 employee_update.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>update employee</title>
</head>
<body>

<!-- IDEA 2021.2.3 对 Thymeleaf 报红，但是不影响解析，具体原因不明 -->
<form th:action="@{/employee}" method="post">
    <input type="hidden" name="_method" value="put">
    <input type="hidden" name="id" th:value="${employee.id}">
    lastName:<input type="text" name="lastName" th:value="${employee.lastName}"><br>
    email:<input type="text" name="email" th:value="${employee.email}"><br>
    <!--
        th:field="${employee.gender}" 可用于单选框或复选框的回显
        若单选框的 value 和 employee.gender 的值一致,则添加 checked="checked" 属性
    -->
    gender:<input type="radio" name="gender" value="1" th:field="${employee.gender}">male
    <input type="radio" name="gender" value="0" th:field="${employee.gender}">female<br>
    <input type="submit" name="update"><br>
</form>

</body>
</html>
```

### 9、具体功能:执行更新

##### a> 控制器方法

```java
@RequestMapping(value = "/employee", method = RequestMethod.PUT)
public String updateEmployee(Employee employee){
    employeeDao.save(employee);
    return "redirect:/employee";
}
```

---

## HttpMessageConverter

HttpMessageConverter，报文信息转换器，将请求报文转换为 Java 对象，或将 Java 对象转换为响应报文

HttpMessageConverter 提供了两个注解和两个类型：@RequestBody，@ResponseBody，RequestEntity，ResponseEntity

### @RequestBody

@RequestBody 可以获取请求体，需要在控制器方法设置一个形参，使用 @RequestBody 进行标识，当前请求的请求体就会为当前注解所标识的形参赋值

index.html

```html
<form th:action="@{/testRequestBody}" method="post">
    <input type="text" name="username"><br>
    <input type="password" name="password"><br>
    <input type="submit" value="测试 @RequestBody">
</form>
```

success.html

```html
<p><font size="10" color="red">success</font></p>
```

```java
@RequestMapping("/testRequestBody")
public String testRequestBody(@RequestBody String requestBody) {
    System.out.println("requestBody : " + requestBody);
    return "success";
}
```

>输出结果：
>
>requestBody : username=19891353163&password=962464

### RequestEntity

RequestEntity 封装请求报文的一种类型，需要在控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，可以通过 getHeaders() 获取请求头信息，通过 getBody() 获取请求体信息

```java
@RequestMapping("/testRequestEntity")
public String testRequestEntity(RequestEntity<String> requestEntity) {
    //当前 requestEntity 表示整个请求报文的信息
    System.out.println("请求头：" + requestEntity.getHeaders());
    System.out.println("请求体：" + requestEntity.getBody());
    return "success";
}
```

>输出结果：
>
>请求头：[host:"localhost:8080", connection:"keep-alive", content-length:"36", cache-control:"max-age=0", sec-ch-ua:""Microsoft Edge";v="95", "Chromium";v="95", ";Not A Brand";v="99"", sec-ch-ua-mobile:"?0", sec-ch-ua-platform:""Windows"", upgrade-insecure-requests:"1", origin:"http://localhost:8080", user-agent:"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36 Edg/95.0.1020.44", accept:"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9", sec-fetch-site:"same-origin", sec-fetch-mode:"navigate", sec-fetch-user:"?1", sec-fetch-dest:"document", referer:"http://localhost:8080/SpringMVC_demo4/", accept-encoding:"gzip, deflate, br", accept-language:"zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6", cookie:"Hm_lvt_b6cb5003addb8959e7e25b8fea13667e=1635685071,1635857075,1635941145,1636032532", Content-Type:"application/x-www-form-urlencoded;charset=UTF-8"]
>
>请求体：username=19891353163&password=962464

### @ResponseBody

@ResponseBody 用于标识一个控制器方法，可以将该方法的返回值直接作为响应报文的响应体响应到浏览器

```java
@RequestMapping("/testResponseBody")
@ResponseBody
public String testResponseBody(){
    return "success";
}
```

>结果：浏览器页面显示 success
>
>注：普通的文本类型 success，没有任何样式的 success。

### SpringMVC 处理 json

@ResponseBody 处理 json 的步骤：

a> 导入 jackson 的依赖

```xml
<!-- jackson -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.5</version>
</dependency>
```

b> 在 SpringMVC 的核心配置文件中开启 mvc 的注解驱动，此时在 HandlerAdaptor 中会自动装配一个消息转换器：MappingJackson2HttpMessageConverter，可以将响应到浏览器的 Java 对象转换为 Json 格式的字符串

```
<mvc:annotation-driven />
```

c> 在处理器方法上使用 @ResponseBody 注解进行标识

d> 将 Java 对象直接作为控制器方法的返回值返回，就会自动转换为 Json 格式的字符串

```java
@RequestMapping("/testResponseUser")
@ResponseBody
public User testResponseUser() {
    return new User(1001, "admin", "962464", 19, "男");
}
```

>浏览器的页面中展示的结果：
>
>{"id":1001,"username":"admin","password":"962464","age":19,"sex":"男"}

### SpringMVC 处理 ajax

a> 请求超链接：

```html
<div id="app">
    <a @click="testAxios" th:href="@{/testAxios}">SpringMVC 处理 ajax</a>
</div>
```

b> 通过 vue 和 axios 处理点击事件：

```html
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
<script type="text/javascript" th:src="@{/static/js/axios.min.js}"></script>
<script type="text/javascript">
    new Vue({
        el:"#app",
        methods: {
            testAxios: function (event) {
                axios({
                    method: "post",
                    url: event.target.href,
                    params: {
                        username: "DoubleZHE",
                        password: "962464"
                    }
                }).then(function (response) {
                    alert(response.data);
                });
                event.preventDefault();
            }
        }
    });
</script>
```

c> 控制器方法：

```java
@RequestMapping("/testAxios")
@ResponseBody
public String testAxios(String username, String password) {
    System.out.println(username + ", " + password);
    return "hello, axios";
}
```

>注：引入 .js 文件后 404 报错的问题在 `RESTful案例` 中有讲到解决办法

### @RestController 注解

@RestController 注解是 springMVC 提供的一个复合注解，标识在控制器的类上，就相当于为类添加了 @Controller 注解，并且为其中的每个方法添加了 @ResponseBody 注解

### ResponseEntity

ResponseEntity 用于控制器方法的返回值类型，该控制器方法的返回值就是响应到浏览器的响应报文。`ResponseEntity 可用于实现文件的上传和下载`

---

## 文件上传和下载

### 1、文件下载

使用 ResponseEntity 实现下载文件的功能

```java
@RequestMapping("/testDown")
public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException {
    //获取 ServletContext 对象
    ServletContext servletContext = session.getServletContext();
    //获取服务器中文件的真实路径
    String realPath = servletContext.getRealPath("/static/img/DO WHAT IS GREAT.jpg");
    //创建输入流
    InputStream inputStream = new FileInputStream(realPath);
    //创建字节数组
    byte[] bytes = new byte[inputStream.available()];
    //将流读到字节数组中
    inputStream.read(bytes);
    //创建 HttpHeaders 对象设置响应头信息
    MultiValueMap<String, String> headers = new HttpHeaders();
    //设置要下载方式以及下载文件的名字
    headers.add("Content-Disposition", "attachment;filename=DO WHAT IS GREAT.jpg"); //除了 filename 的属性值可以改，其他的都是固定的
    //设置响应状态码
    HttpStatus statusCode = HttpStatus.OK;
    //创建 ResponseEntity 对象
    ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes, headers, statusCode);
    //关闭输入流
    inputStream.close();

    return responseEntity;
}
```

### 2、文件上传

文件上传要求 form 表单的请求方式必须为 post，并且添加属性 enctype="multipart/form-data"

SpringMVC 中将上传的文件封装到 MultipartFile 对象中，通过此对象可以获取文件相关信息

上传步骤：

a> 添加依赖：

```xml
<!-- 文件上传 -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>
```

b> 在 SpringMVC 的配置文件中添加配置：

```xml
<!-- 配置文件上传解析器，将上传的的文件封装为 MultipartFile -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"/>
```

c> 控制器方法：

```java
@RequestMapping("/testUp")
public String testUp(MultipartFile photo, HttpSession session) throws IOException {
//获取上传文件的文件名
String fileName = photo.getOriginalFilename();

//处理文件重名问题
//获取上传文件的后缀名
String suffixName = fileName.substring(fileName.lastIndexOf("."));
//将 UUID 作为文件名
String uuid = UUID.randomUUID().toString();
//String uuid = UUID.randomUUID().toString().replaceAll("-", ""); //可将生成文件名中的 - 替换掉
//将 uuid 和 后缀名拼接后的结果作为最终的文件名
fileName = uuid + suffixName;

//通过 ServletContext 获取服务器中 photo 目录的路径
ServletContext servletContext = session.getServletContext();
String photoPath = servletContext.getRealPath("photo");

//确认文件上传到 photo 目录的路径
File file = new File(photoPath);
//判断 photoPath 所对应路径是否存在
if (!file.exists()) {
    //若不存在，则创建目录
    file.mkdir();
}
String finalPath = photoPath + File.separator + fileName;   //finalPath = xxx\photo\fileName.suffixName

//开始上传
photo.transferTo(new File(finalPath));

return "success";
}
```

---

## 拦截器

### 1、拦截器的配置

SpringMVC 中的拦截器用于拦截控制器方法的执行

SpringMVC 中的拦截器需要实现 HandlerInterceptor

SpringMVC 的拦截器必须在 SpringMVC 的配置文件中进行配置：

```xml
<bean class="com.atguigu.interceptor.FirstInterceptor"></bean>
<ref bean="firstInterceptor"></ref>
<!-- 以上两种配置方式都是对 DispatcherServlet 所处理的所有的请求进行拦截 -->
<!-- 不推荐以上两种方法 -->

<mvc:interceptor>
    <mvc:mapping path="/**"/>   <!-- /** 表示拦截所有。 /* 表示仅拦截一层目录 -->
    <mvc:exclude-mapping path="/"/> <!-- 将请求 / 排除在拦截器之外，允许请求路径为 /  -->
    <ref bean="firstInterceptor"></ref>
</mvc:interceptor>
<!-- 
	以上配置方式可以通过 ref 或 bean 标签设置拦截器，通过 mvc:mapping 设置需要拦截的请求，通过 mvc:exclude-mapping 设置需要排除的请求，即不需要拦截的请求
-->
```

### 2、拦截器的三个抽象方法

SpringMVC 中的拦截器有三个抽象方法：

preHandle：控制器方法执行之前执行 preHandle()，其 boolean 类型的返回值表示是否拦截或放行，返回 true 为放行，即调用控制器方法；返回 false 表示拦截，即不调用控制器方法

postHandle：控制器方法执行之后执行 postHandle()

afterComplation：处理完视图和模型数据，渲染视图完毕之后执行 afterComplation()

### 3、多个拦截器的执行顺序

a> 若每个拦截器的 preHandle() 都返回 true

此时多个拦截器的执行顺序和拦截器在 SpringMVC 的配置文件的配置顺序有关：

preHandle() 会按照配置的顺序执行，而 postHandle() 和 afterComplation() 会按照配置的反序执行

b> 若某个拦截器的 preHandle() 返回了 false

preHandle() 返回 false 和它之前的拦截器的 preHandle() 都会执行，postHandle() 都不执行，返回 false 的拦截器之前的拦截器的 afterComplation() 会执行

---

## 异常处理器

### 1、基于配置的异常处理

SpringMVC 提供了一个处理控制器方法执行过程中所出现的异常的接口：HandlerExceptionResolver

HandlerExceptionResolver 接口的实现类有：DefaultHandlerExceptionResolver 和 SimpleMappingExceptionResolver

SpringMVC 提供了自定义的异常处理器 SimpleMappingExceptionResolver，使用方式：

```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings">
        <props>
        	<!--
        		properties的键表示处理器方法执行过程中出现的异常
        		properties的值表示若出现指定异常时，设置一个新的视图名称，跳转到指定页面
        	-->
            <prop key="java.lang.ArithmeticException">error</prop>
        </props>
    </property>
    <!--
    	exceptionAttribute属性设置一个属性名，将出现的异常信息在请求域中进行共享
    -->
    <property name="exceptionAttribute" value="exception"></property>
</bean>
```

### 2、基于注解的异常处理

```java
//@ControllerAdvice将当前类标识为异常处理的组件
@ControllerAdvice
public class ExceptionController {

    //@ExceptionHandler用于设置所标识方法处理的异常
    @ExceptionHandler(ArithmeticException.class)
    //ex表示当前请求处理中出现的异常对象
    public String handleArithmeticException(Exception ex, Model model){
        model.addAttribute("ex", ex);
        return "error";
    }

}
```

---

## 注解配置 SpringMVC

使用配置类和注解代替 web.xml 和 SpringMVC 配置文件的功能

### 1、创建初始化类，代替 web.xml

在 Servlet3.0 环境中，容器会在类路径中查找实现 javax.servlet.ServletContainerInitializer 接口的类，如果找到的话就用它来配置 Servlet 容器。
Spring 提供了这个接口的实现，名为 SpringServletContainerInitializer，这个类反过来又会查找实现 WebApplicationInitializer 的类并将配置的任务交给它们来完成。Spring3.2 引入了一个便利的 WebApplicationInitializer 基础实现，名为 AbstractAnnotationConfigDispatcherServletInitializer，当我们的类扩展了 AbstractAnnotationConfigDispatcherServletInitializer 并将其部署到 Servlet3.0 容器的时候，容器会自动发现它，并用它来配置 Servlet 上下文。

```java
import org.springframework.web.filter.CharacterEncodingFilter;
import org.springframework.web.filter.HiddenHttpMethodFilter;
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

import javax.servlet.Filter;

public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {
    /**
     * 指定 Spring 的配置类
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    /**
     * 指定 SpringMVC 的配置类
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    /**
     * 指定 DispatchServlet 的映射规则，即 url-pattern
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    /**
     * 注册过滤器
     * @return
     */
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
        characterEncodingFilter.setEncoding("UTF-8");
        characterEncodingFilter.setForceResponseEncoding(true);
        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();

        return new Filter[]{characterEncodingFilter, hiddenHttpMethodFilter};
    }
}
```

### 2、创建 SpringConfig 配置类，代替 spring 的配置文件

```java
@Configuration
public class SpringConfig {
	//ssm 整合之后，spring 的配置信息写在此类中
}
```

### 3、创建 WebConfig 配置类，代替 SpringMVC 的配置文件

```java
import mvc.interceptor.testInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.context.ContextLoader;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.multipart.MultipartResolver;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.*;
import org.springframework.web.servlet.handler.SimpleMappingExceptionResolver;
import org.thymeleaf.spring5.SpringTemplateEngine;
import org.thymeleaf.spring5.view.ThymeleafViewResolver;
import org.thymeleaf.templatemode.TemplateMode;
import org.thymeleaf.templateresolver.ITemplateResolver;
import org.thymeleaf.templateresolver.ServletContextTemplateResolver;

import java.util.List;
import java.util.Properties;

/**
 * @ClassName WebConfig
 * @Description SpringMVC 的配置类。需要包含的内容如下：
 *                  1、扫描组件
 *                  2、视图解析器
 *                  3、view-controller
 *                  4、default-servlet-handler
 *                  5、mvc 注解驱动
 *                  6、文件上传解析器
 *                  7、异常处理器
 *                  8、拦截器
 * @Author DoubleZHE
 * @Create 2021/11/17 14:05
 */

@Configuration  //将当前类标识为一个配置类
@ComponentScan("mvc.controller")  //扫描组件
@EnableWebMvc   //mvc 注解驱动
public class WebConfig implements WebMvcConfigurer {
    //default-servlet-handler
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        testInterceptor testInterceptor = new testInterceptor();
        registry.addInterceptor(testInterceptor).addPathPatterns("/**");    //拦截所有请求
    }

    //view-controller
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/hello").setViewName("hello");
    }

    //文件上传解析器
    @Bean
    public MultipartResolver multipartResolver() {
        CommonsMultipartResolver commonsMultipartResolver = new CommonsMultipartResolver();
        return commonsMultipartResolver;
    }

    //异常处理器
    @Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
        Properties properties = new Properties();
        properties.setProperty("java.lang.ArithmeticException", "error")
        exceptionResolver.setExceptionMappings(properties);
        exceptionResolver.setExceptionAttribute("exception");
        resolvers.add(exceptionResolver);
    }

    //配置生成模板解析器
    @Bean
    public ITemplateResolver templateResolver() {
        WebApplicationContext webApplicationContext = ContextLoader.getCurrentWebApplicationContext();
        // ServletContextTemplateResolver 需要一个 ServletContext 作为构造参数，可通过 WebApplicationContext 的方法获得
        ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver(webApplicationContext.getServletContext());
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setCharacterEncoding("UTF-8");
        templateResolver.setTemplateMode(TemplateMode.HTML);
        return templateResolver;
    }

    //生成模板引擎并为模板引擎注入模板解析器
    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        return templateEngine;
    }

    //生成视图解析器并未解析器注入模板引擎
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setCharacterEncoding("UTF-8");
        viewResolver.setTemplateEngine(templateEngine);
        return viewResolver;
    }
}
```

### 4、测试功能

```java
@RequestMapping("/")
public String index(){
    return "index";
}
```

---

## SpringMVC 执行流程

### 1、SpringMVC常用组件

- DispatcherServlet：**前端控制器**，不需要工程师开发，由框架提供

作用：统一处理请求和响应，整个流程控制的中心，由它调用其它组件处理用户的请求

- HandlerMapping：**处理器映射器**，不需要工程师开发，由框架提供

作用：根据请求的 url、method 等信息查找 Handler，即控制器方法

- Handler：**处理器**，需要工程师开发

作用：在 DispatcherServlet 的控制下 Handler 对具体的用户请求进行处理

- HandlerAdapter：**处理器适配器**，不需要工程师开发，由框架提供

作用：通过 HandlerAdapter 对处理器（控制器方法）进行执行

- ViewResolver：**视图解析器**，不需要工程师开发，由框架提供

作用：进行视图解析，得到相应的视图，例如：ThymeleafView、InternalResourceView、RedirectView

- View：**视图**

作用：将模型数据通过页面展示给用户

### 2、DispatcherServlet 初始化过程

DispatcherServlet 本质上是一个 Servlet，所以天然的遵循 Servlet 的生命周期。所以宏观上是 Servlet 生命周期来进行调度。

![](https://gitee.com/DoubleZHEz/mine/raw/master/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/SpringMVC/202111171906612.png)

##### a> 初始化 WebApplicationContext

所在类：org.springframework.web.servlet.FrameworkServlet

```java
protected WebApplicationContext initWebApplicationContext() {
    WebApplicationContext rootContext =
        WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;

    if (this.webApplicationContext != null) {
        // A context instance was injected at construction time -> use it
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            if (!cwac.isActive()) {
                // The context has not yet been refreshed -> provide services such as
                // setting the parent context, setting the application context id, etc
                if (cwac.getParent() == null) {
                    // The context instance was injected without an explicit parent -> set
                    // the root application context (if any; may be null) as the parent
                    cwac.setParent(rootContext);
                }
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
        // No context instance was injected at construction time -> see if one
        // has been registered in the servlet context. If one exists, it is assumed
        // that the parent context (if any) has already been set and that the
        // user has performed any initialization such as setting the context id
        wac = findWebApplicationContext();
    }
    if (wac == null) {
        // No context instance is defined for this servlet -> create a local one
        // 创建WebApplicationContext
        wac = createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        // Either the context is not a ConfigurableApplicationContext with refresh
        // support or the context injected at construction time had already been
        // refreshed -> trigger initial onRefresh manually here.
        synchronized (this.onRefreshMonitor) {
            // 刷新WebApplicationContext
            onRefresh(wac);
        }
    }

    if (this.publishContext) {
        // Publish the context as a servlet context attribute.
        // 将IOC容器在应用域共享
        String attrName = getServletContextAttributeName();
        getServletContext().setAttribute(attrName, wac);
    }

    return wac;
}
```

##### b> 创建 WebApplicationContext

所在类：org.springframework.web.servlet.FrameworkServlet

```java
protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
    Class<?> contextClass = getContextClass();
    if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
        throw new ApplicationContextException(
            "Fatal initialization error in servlet with name '" + getServletName() +
            "': custom WebApplicationContext class [" + contextClass.getName() +
            "] is not of type ConfigurableWebApplicationContext");
    }
    // 通过反射创建 IOC 容器对象
    ConfigurableWebApplicationContext wac =
        (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);

    wac.setEnvironment(getEnvironment());
    // 设置父容器
    wac.setParent(parent);
    String configLocation = getContextConfigLocation();
    if (configLocation != null) {
        wac.setConfigLocation(configLocation);
    }
    configureAndRefreshWebApplicationContext(wac);

    return wac;
}
```

##### c> DispatcherServlet 初始化策略

FrameworkServlet 创建 WebApplicationContext 后，刷新容器，调用 onRefresh(wac)，此方法在 DispatcherServlet 中进行了重写，调用了 initStrategies(context) 方法，初始化策略，即初始化 DispatcherServlet 的各个组件

所在类：org.springframework.web.servlet.DispatcherServlet

```java
protected void initStrategies(ApplicationContext context) {
   initMultipartResolver(context);
   initLocaleResolver(context);
   initThemeResolver(context);
   initHandlerMappings(context);
   initHandlerAdapters(context);
   initHandlerExceptionResolvers(context);
   initRequestToViewNameTranslator(context);
   initViewResolvers(context);
   initFlashMapManager(context);
}
```

### 3、DispatcherServlet 调用组件处理请求

##### a> processRequest()

FrameworkServlet 重写 HttpServlet 中的 service() 和 doXxx()，这些方法中调用了 processRequest(request, response)

所在类：org.springframework.web.servlet.FrameworkServlet

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;

    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = buildLocaleContext(request);

    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    initContextHolders(request, localeContext, requestAttributes);

    try {
		// 执行服务，doService() 是一个抽象方法，在 DispatcherServlet 中进行了重写
        doService(request, response);
    }
    catch (ServletException | IOException ex) {
        failureCause = ex;
        throw ex;
    }
    catch (Throwable ex) {
        failureCause = ex;
        throw new NestedServletException("Request processing failed", ex);
    }

    finally {
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }
        logResult(request, response, failureCause, asyncManager);
        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```

##### b> doService()

所在类：org.springframework.web.servlet.DispatcherServlet

```java
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    logRequest(request);

    // Keep a snapshot of the request attributes in case of an include,
    // to be able to restore the original attributes after the include.
    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }

    // Make framework objects available to handlers and view objects.
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

    if (this.flashMapManager != null) {
        FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
        if (inputFlashMap != null) {
            request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
        }
        request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
        request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
    }

    RequestPath requestPath = null;
    if (this.parseRequestPath && !ServletRequestPathUtils.hasParsedRequestPath(request)) {
        requestPath = ServletRequestPathUtils.parseAndCache(request);
    }

    try {
        // 处理请求和响应
        doDispatch(request, response);
    }
    finally {
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // Restore the original attribute snapshot, in case of an include.
            if (attributesSnapshot != null) {
                restoreAttributesAfterInclude(request, attributesSnapshot);
            }
        }
        if (requestPath != null) {
            ServletRequestPathUtils.clearParsedRequestPath(request);
        }
    }
}
```

##### c> doDispatch()

所在类：org.springframework.web.servlet.DispatcherServlet

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // Determine handler for the current request.
            /*
            	mappedHandler：调用链。包含 handler、interceptorList、interceptorIndex
            	handler：浏览器发送的请求所匹配的控制器方法
            	interceptorList：处理控制器方法的所有拦截器集合
            	interceptorIndex：拦截器索引，控制拦截器afterCompletion()的执行
            */
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            // Determine handler adapter for the current request.
           	// 通过控制器方法创建相应的处理器适配器，调用所对应的控制器方法
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            // Process last-modified header, if supported by the handler.
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }
			
            // 调用拦截器的 preHandle()
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            // Actually invoke the handler.
            // 由处理器适配器调用具体的控制器方法，最终获得 ModelAndView 对象
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            applyDefaultViewName(processedRequest, mv);
            // 调用拦截器的 postHandle()
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            // As of 4.3, we're processing Errors thrown from handler methods as well,
            // making them available for @ExceptionHandler methods and other scenarios.
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
        // 后续处理：处理模型数据和渲染视图
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                               new NestedServletException("Handler processing failed", err));
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            // Instead of postHandle and afterCompletion
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            // Clean up any resources used by a multipart request.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
```

##### d> processDispatchResult()

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
                                   @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
                                   @Nullable Exception exception) throws Exception {

    boolean errorView = false;

    if (exception != null) {
        if (exception instanceof ModelAndViewDefiningException) {
            logger.debug("ModelAndViewDefiningException encountered", exception);
            mv = ((ModelAndViewDefiningException) exception).getModelAndView();
        }
        else {
            Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
            mv = processHandlerException(request, response, handler, exception);
            errorView = (mv != null);
        }
    }

    // Did the handler return a view to render?
    if (mv != null && !mv.wasCleared()) {
        // 处理模型数据和渲染视图
        render(mv, request, response);
        if (errorView) {
            WebUtils.clearErrorRequestAttributes(request);
        }
    }
    else {
        if (logger.isTraceEnabled()) {
            logger.trace("No view rendering, null ModelAndView returned.");
        }
    }

    if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
        // Concurrent handling started during a forward
        return;
    }

    if (mappedHandler != null) {
        // Exception (if any) is already handled..
        // 调用拦截器的 afterCompletion()
        mappedHandler.triggerAfterCompletion(request, response, null);
    }
}
```

### 4、SpringMVC的执行流程

1) 用户向服务器发送请求，请求被 SpringMVC 前端控制器 DispatcherServlet 捕获。

2) DispatcherServlet 对请求 URL 进行解析，得到请求资源标识符（URI），判断请求 URI 对应的映射：

a) 不存在

i. 再判断是否配置了 mvc:default-servlet-handler

ii. 如果没配置，则控制台报映射查找不到，客户端展示 404 错误

![](https://gitee.com/DoubleZHEz/mine/raw/master/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/SpringMVC/202111171913611.png)

![](https://gitee.com/DoubleZHEz/mine/raw/master/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/SpringMVC/202111171913283.png)

iii. 如果有配置，则访问目标资源（一般为静态资源，如：JS, CSS, HTML），找不到客户端也会展示 404 错误

![](https://gitee.com/DoubleZHEz/mine/raw/master/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/SpringMVC/202111171913646.png)

![](https://gitee.com/DoubleZHEz/mine/raw/master/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/SpringMVC/202111171914772.png)

b) 存在则执行下面的流程

3) 根据该URI，调用 HandlerMapping 获得该 Handler 配置的所有相关的对象（包括 Handler 对象以及 Handler 对象对应的拦截器），最后以 HandlerExecutionChain 执行链对象的形式返回。

4) DispatcherServlet 根据获得的 Handler，选择一个合适的 HandlerAdapter。

5) 如果成功获得 HandlerAdapter，此时将开始执行拦截器的 preHandler(…) 方法【正向】

6) 提取 Request 中的模型数据，填充 Handler 入参，开始执行 Handler（Controller) 方法，处理请求。在填充 Handler 的入参过程中，根据你的配置，Spring 将帮你做一些额外的工作：

a) HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息

b) 数据转换：对请求消息进行数据转换。如 String 转换成 Integer、Double 等

c) 数据格式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等

d) 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到 BindingResult 或 Error 中

7) Handler 执行完成后，向 DispatcherServlet 返回一个 ModelAndView 对象。

8) 此时将开始执行拦截器的 postHandle(...)方法【逆向】。

9) 根据返回的 ModelAndView（此时会判断是否存在异常：如果存在异常，则执行 HandlerExceptionResolver进 行异常处理）选择一个适合的 ViewResolver 进行视图解析，根据 Model 和 View，来渲染视图。

10) 渲染视图完毕执行拦截器的 afterCompletion(…) 方法【逆向】。

11) 将渲染结果返回给客户端。

---

***Author DoubleZHE***
***2021.11.17***