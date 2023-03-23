## SSM 框架整合

### 整合策略

SSM = Spring + Spring MVC + MyBatis = (Spring + MyBatis) + Spring MVC

先整合 Spring + MyBatis，然后再整合 Spring MVC



Spring MVC 负责请求的转发和视图管理，实现 MVC 实现模式

Spring 实现业务对象管理，管理 Spring MVC 和 MyBatis 相关对象的创建和注入

MyBatis 作为数据对象的持久化引擎



![image-20211126161251466]( https://doublezhe-repository-trial.oss-cn-beijing.aliyuncs.com/ImageRepository/Spring%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88/202111261612766.png)



**基于的需求：**查询 Account 表的全部数据显示到页面

### MyBatis 整合 Spring

- 整合目标
  - 数据库连接池以及事务管理都交给 Spring 容器来完成
  - SqlSessionFactory 对象应该放到 Spring 容器中作为单例对象管理
  - Mapper 动态代理对象交给 Spring 管理，从 Spring  容器中直接获得 Mapper 的代理对象
- 整合所需 Jar 分析
  - Junit 测试（4.12 版本）
  - MyBatis
  - Spring 相关（spring-context、spring-jdbc、spring-tx、aspectjweaver）
  - MyBatis / Spring 整合包
  - MySQL 数据库驱动
  - Druid 数据库连接池
- 整合后的 pom 坐标

```xml
  <dependencies>
    <!-- junit 测试 -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
    <!-- MyBatis -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.7</version>
    </dependency>
    <!-- Spring 相关 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.3.13</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.3.13</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.3.13</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>5.3.13</version>
    </dependency>
    <!--
      spring-context 中已包含 spring-aop，没必要再导入
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>5.3.13</version>
      </dependency>
    -->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.9.7</version>
    </dependency>
    <!-- MyBatis 与 Spring 的整合包 -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>2.0.6</version>
    </dependency>
    <!-- 数据库驱动：MySQL -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.25</version>
    </dependency>
    <!-- druid 连接池 -->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.10</version>
    </dependency>
  </dependencies>
```

#### Dao 层代码开发

1. 数据库建表

```mysql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for account
-- ----------------------------
DROP TABLE IF EXISTS `account`;
CREATE TABLE `account`  (
  `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '用户名',
  `money` int(0) NULL DEFAULT NULL COMMENT '账户余额',
  `cardID` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '银行卡号',
  PRIMARY KEY (`cardID`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of account
-- ----------------------------
INSERT INTO `account` VALUES ('张三', 8600, '6029621011000');
INSERT INTO `account` VALUES ('李四', 11500, '6029621011001');

SET FOREIGN_KEY_CHECKS = 1;
```

2. account 实体类

```java
	private String name;
    private int money;
    private String cardID;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getMoney() {
        return money;
    }

    public void setMoney(int money) {
        this.money = money;
    }

    public String getCardID() {
        return cardID;
    }

    public void setCardID(String cardID) {
        this.cardID = cardID;
    }

    @Override
    public String toString() {
        return "Account{" +
                "name='" + name + '\'' +
                ", money=" + money +
                ", cardID='" + cardID + '\'' +
                '}';
    }
```

3. Mapper 接口

```java
public interface AccountMapper {
    /**
     * 查询 account 表所有数据
     * @return
     * @throws Exception
     */
    @Select("select * from account")
    List<Account> queryAccountList() throws Exception;
}
```

4. 配置 dao 层的 Spring 核心配置文件

> applicationContext-dao.xml 需要配置的内容：
>
> 包扫描、数据库连接池、SqlSessionFactory 对象、Mapper 动态代理对象

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 包扫描 -->
    <context:component-scan base-package="integration.mapper"/>

    <!-- 数据库连接池以及事务管理都交给 Spring 容器来完成 -->
        <!-- 数据库连接池配置 -->
        <!-- 引入外部资源文件 -->
        <context:property-placeholder location="classpath*:jdbc.properties"/>

        <!-- 第三方 jar 中的 bean 定义在 xml 中 -->
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="${jdbc.driver}"/>
            <property name="url" value="${jdbc.url}"/>
            <property name="username" value="${jdbc.username}"/>
            <property name="password" value="${jdbc.password}"/>
        </bean>

    <!-- SqlSessionFactory 对象应该放到 Spring 容器中作为单例对象管理
            原来 MyBatis 中 sqlSessionFactory 的构建是需要素材的：mybatis-config.xml 中的内容
     -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 别名映射扫描 -->
        <property name="typeAliasesPackage" value="integration.pojo"/>
        <!-- 数据源 dataSource -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- Mapper 动态代理对象交给 Spring 管理，从 Spring  容器中直接获得 Mapper 的代理对象 -->
    <!-- 扫描 mapper 接口，生成代理对象存储在 IOC 容器中 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- mapper 接口包路径配置 -->
        <property name="basePackage" value="integration.mapper"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>

</beans>
```

5. 配置 JDBC 数据库连接池

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
#jdbc.driver=com.mysql.jdbc.Driver	# MySQL 8.0 以下使用
jdbc.url=jdbc:mysql:///数据库名
jdbc.username=root
jdbc.password=962464
```

#### Service 层代码开发

1. 创建 service 接口

```java
public interface AccountService {
    List<Account> queryAccountList() throws Exception;
}
```

2. 创建 service 接口实现类，并重写方法

```java
@Service
@Transactional
public class AccountServiceImpl implements AccountService {
    @Autowired
    private AccountMapper accountMapper;

    @Override
    public List<Account> queryAccountList() throws Exception {
        return accountMapper.queryAccountList();
    }
}
```

3. 配置 service 层的 Spring 核心配置文件

> applicationContext-service.xml 需要配置的内容：
>
> 包扫描、事务管理、事务管理注解驱动

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" 
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd 
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 包扫描 -->
    <context:component-scan base-package="integration.service"/>

    <!-- 事务管理 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    
    <!-- 事务管理注解驱动 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>

</beans>
```

#### 测试

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath*:application*.xml"})
public class MyBatisSpringTest {
    //希望测试 IOC 容器中的哪个对象注入即可
    @Autowired
    private AccountService accountService;

    @Test
    public void testMyBatisSpring() throws Exception {
        for (Account account : accountService.queryAccountList()) {
            System.out.println(account);
        }

    }
}
```

> 测试结果：
>
> ![image-20211128183305843]( https://doublezhe-repository-trial.oss-cn-beijing.aliyuncs.com/ImageRepository/Spring%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88/202111281833999.png)

### 整合 Spring MVC 

- 整合思路

在已有工程基础之上开发一个 Spring MVC 入门案例

- 引入 pom 坐标

```xml
	<!-- Spring MVC -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.3.13</version>
    </dependency>

    <!-- jsp-api & servlet-api -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.0</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
      <scope>provided</scope>
    </dependency>

    <!-- 页面使用 jstl 表达式 -->
    <dependency>
      <groupId>jstl</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
    <dependency>
      <groupId>taglibs</groupId>
      <artifactId>standard</artifactId>
      <version>1.1.2</version>
    </dependency>

    <!-- json 数据交互 -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.12.5</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.12.5</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>2.12.5</version>
    </dependency>
  </dependencies>
```

1. 将项目部署到 Tomcat 
2. 配置 web.xml 文件

> 需要配置的内容：
>
> web.xml启动spring容器、DispathcheServlet的声明、其余：session过期，字符串编码等

```xml
  <!-- 配置 Spring 配置文件路径 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:applicationContext*.xml</param-value>
  </context-param>
  <!-- Spring 框架启动，定义 Spring 监听器，加载 Spring -->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <!-- Spring MVC 启动 -->
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath*:springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <!--配置访问接口类型-->
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```

3. springmvc 核心配置文件

> 需要配置的内容：
>
> 扫描控制包、开启注解驱动

4. controller 控制层代码

```java
@Controller
@RequestMapping("/account")
public class AccountController {

    /**
     * Spring 容器和 SpringMVC 容器是有层次的（父子容器）
     * Spring 容器：service 对象 + dao 对象
     * Spring MVC 容器：controller 对象，可以引用到 Spring 容器中的对象
     */

    @Autowired
    private AccountService accountService;

    @RequestMapping("queryAll")
    @ResponseBody
    public List<Account> queryAll() throws Exception {
        return accountService.queryAccountList();
    }
}
```

#### 测试

1. 开启 Tomcat
2. 输入控制层链接

3. 浏览器结果

![image-20211128191813182]( https://doublezhe-repository-trial.oss-cn-beijing.aliyuncs.com/ImageRepository/Spring%E6%A1%86%E6%9E%B6%E6%95%B4%E5%90%88/202111281918266.png)

> json 数据交互 jar 包一定得有，不然会出现错误。需要 json 包将 Java 对象转换为 json 对象显示在浏览器中。

---

***Author DoubleZHE***

***2021.11.28***
