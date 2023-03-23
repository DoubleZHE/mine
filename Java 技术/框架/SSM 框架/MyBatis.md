## 一、MyBatis 简介

### 1. 原始 Jdbc 的分析

原始 Jdbc 开发存在的问题如下：

1. 数据库连接创建、释放频繁造成系统资源浪费从而影响系统性能
2. sql 语句在代码中硬编码，造成代码不易维护，实际应用 sql 变化的可能较大，sql 变动需要改变 Java 代码
3. 查询操作是，需要手动将结果集中的数据封装到实体中。插入数据时，需要手动将实体的数据设置到 sql 语句的占位符位置

应对上述问题给出的结局方案：
1. 使用数据库连接池初始化连接资源
2. 将 sql 语句抽取到 xml 配置文件中
3. 使用反射、内省等底层技术，自动将实体与表进行属性与字段的自动映射

### 2. MyBatis 概述

- MyBatis 是一个开源、轻量级的数据持久化框架，是 JDBC 和 Hibernate 的替代方案。MyBatis 内部封装了 JDBC，简化了加载驱动、创建连接、创建 statement 等繁杂的过程，开发者只需要关注 SQL 语句本身，而不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁琐的过程
- MyBatis 通过 xml 或注解的方式将要执行的各种 statement 配置起来，并通过 Java 对象和 statement 中的 sql 的动态参数进行映射生成最终执行的 sql 语句
- 最后 MyBatis 框架执行 sql 并将结果映射为 Java 对象 并返回。采用 ORM 思想解决了实体和数据库映射的问题，对 jdbc 进行了封装，屏蔽了 jdbc API 底层访问细节，使我们不用与 jdbc API 打交道，就可以完成对数据库的持久化操作

---

## 二、MyBatis 快速入门

MyBatis 官网地址：https://mybatis.org/mybatis-3/

### MyBatis 开发步骤

1. 添加 MyBatis 的坐标

```xml
<dependencies>
    <!-- 数据库驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.25</version>
    </dependency>

    <!-- MyBatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.6</version>
    </dependency>

    <!-- 测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>

    <!-- 日志 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```

2. 创建 user 数据表

![](R:/ImageRepository/MyBatis/202111181427984.png)

3. 编写 User 实体类

```java
public class User {
    private Integer id;
    private String username;
    private String password;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

4. 编写映射文件 UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="userMapper">
    <select id="findAll" resultType="mybatis.domain.User">
        select * from user
    </select>
</mapper>
```

5. 编写核心文件 SqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 数据源环境 -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///test_mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="962464"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 加载映射文件 -->
    <mappers>
        <mapper resource="mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

6. 编写测试类

```java
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

@Test
public void test1() throws IOException {
    //获取核心配置文件
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    //获取 session 工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    //获得 session 会话对象
    SqlSession sqlSession = sqlSessionFactory.openSession();
    //执行操作  参数：namespace.id
    List<User> userList = sqlSession.selectList("userMapper.findAll");
    //打印数据
    System.out.println(userList);
    //释放资源
    sqlSession.close();
}
```

---

## 三、MyBatis 的映射文件概述

![](R:/ImageRepository/MyBatis/202111181436761.png)

### MyBatis 映射器的主要元素

|元素名称|描述|备注|
|---|---|---|
|mapper|映射文件的根节点，只有 namespace 一个属性|namespace 作用如下：<br> <ul> <li>用于区分不同的 mapper，全局唯一</li> <li>绑定 DAO 接口，即面向接口编程。当 namespace 绑定某一接口后，可以不用写该接口的实现类，MyBatis 会通过接口的完整限定名查找到对应的 mappper 配置来执行 SQL 语句。一次 namespace 的命名必须要跟接口同名</li> </ul>|
|select|查询语句，最常用、最复杂的元素之一|可以自定义参数，返回结果等|
|insert|插入语句|执行后返回一个整数，代表插入的条数|
|update|更新语句|执行后返回一个整数，代表更新的条数|
|delete|删除语句|执行后返回一个整数，代表删除的条数|
|parameterMap|定义参数映射关系|即将被删除的元素，不建议使用|
|sql|允许定义一部分的 SQL，然后在各个地方引用它|例如，一张表列名，我们可以一次定义，在多个 SQL 语句中使用|
|resultMap|用来描述数据库结果集与对象的对应关系，它是最复杂、最强大的元素|提供映射规则|
|cache|配置给定命名空间的缓存|-|
|cache-ref|其它命名空间缓存配置的引用|-|

---

## 四、MyBatis 的增删改查操作

### 1. 查询数据

#### a> 在 mapper 文件中添加 SQL 查询操作

```xml
<select id="findAll" resultType="mybatis.domain.User">
    select * from user
</select>
```

#### b> 测试代码

```java
@Test
public void select() throws IOException {
    //获取核心配置文件
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    //获取 session 工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    //获得 session 会话对象
    SqlSession sqlSession = sqlSessionFactory.openSession();
    //执行操作  参数：namespace.id
    List<User> userList = sqlSession.selectList("userMapper.findAll");
    //打印数据
    System.out.println(userList);
    //释放资源
    sqlSession.close();
}
```

### 2. 插入数据

#### a> 在 mapper 文件中添加 SQL 插入操作

```xml
<insert id="save" parameterType="mybatis.domain.User">
    insert into user values(#{id}, #{username}, #{password})
</insert>
```

#### b> 测试代码

```java
@Test
public void insert() throws IOException {
    //模拟 user 对象
    User user = new User();
    user.setUsername("tom");
    user.setPassword("abc");

    //获取核心配置文件
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    //获取 session 工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    //获得 session 会话对象
    SqlSession sqlSession = sqlSessionFactory.openSession();
    //执行操作  参数：namespace.id
    sqlSession.insert("userMapper.save", user);

    //MyBatis 执行更新操作，提交事务
    sqlSession.commit();

    //释放资源
    sqlSession.close();
}
```

>插入操作注意问题：
>
><ul> <li>插入语句使用 insert 标签</li> <li>在映射文件中使用 parameterType 属性指定要插入的数据类型</li> <li>SQL 语句中使用 #{实体属性名} 方式引用实体中的属性值</li> <li>插入操作使用的 API 是 sqlSession.insert("namespace.id", 实体对象)</li> <li>插入操作涉及数据库数据变化，所以要使用 SqlSession 对象显式的提交事务，即 sqlSession.commit()</li> </ul>



### 3. 修改数据

#### a> 在 mapper 文件中添加 SQL 修改操作

```xml
<update id="update" parameterType="mybatis.domain.User">
    update user set username=#{username}, password=#{password} where id=#{id}
</update>
```

#### b> 测试代码

```java
@Test
public void update() throws IOException {
    //模拟 user 对象
    User user = new User();
    user.setId(7);
    user.setUsername("lucy");
    user.setPassword("123");

    //获取核心配置文件
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    //获取 session 工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    //获得 session 会话对象
    SqlSession sqlSession = sqlSessionFactory.openSession();
    //执行操作  参数：namespace.id
    sqlSession.update("userMapper.update", user);

    //MyBatis 执行更新操作，提交事务
    sqlSession.commit();

    //释放资源
    sqlSession.close();
}
```

>修改操作注意问题：
>
><ul> <li>修改语句使用 update 标签</li> <li>修改操作使用 API 是 sqlSession.update("namespace.id", 实体对象)</li> </ul>

### 4. 删除数据

#### a> 在 mapper 文件中添加 SQL 删除操作

```xml
<delete id="delete" parameterType="java.lang.Integer">    /* 根据 id 删除数据，所以 parameterType 的属性值为 java.lang.Integer */
    delete from user where id=#{id}
</delete>
```

#### b> 测试代码

```java
@Test
public void delete() throws IOException {
    //获取核心配置文件
    InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
    //获取 session 工厂对象
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    //获得 session 会话对象
    SqlSession sqlSession = sqlSessionFactory.openSession();
    //执行操作  参数：namespace.id
    sqlSession.delete("userMapper.delete", 7);

    //MyBatis 执行更新操作，提交事务
    sqlSession.commit();

    //释放资源
    sqlSession.close();
}
```

>删除操作注意问题：
>
><ul> <li>删除语句使用 delete 标签</li> <li>SQL 语句中使用 #{任意字符串} 方式引用传递的单个参数</li> <li>删除操作使用 API 是 sqlSession.delete("namespace.id", Object)</li> </ul>

---

## 五、MyBatis 核心配置文件概述

### 1. MyBatis 核心配置文件层级关系

- configuration 配置
    - properties 属性
    - settings 设置
    - typeAliases 类型别名
    - typeHandlers 类型处理器
    - objectFactory 对象工厂
    - plugins 插件
    - environments 环境配置
        - environment 环境变量
            - transactionManager 事务管理器
            - dataSource 数据源
    - databaseIdProvider 数据库厂商标识
    - mappers 映射器

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration><!-- 配置 -->
    <properties /><!-- 属性 -->
    <settings /><!-- 设置 -->
    <typeAliases /><!-- 类型命名 -->
    <typeHandlers /><!-- 类型处理器 -->
    <objectFactory /><!-- 对象工厂 -->
    <plugins /><!-- 插件 -->
    <environments><!-- 配置环境 -->
        <environment><!-- 环境变量 -->
            <transactionManager /><!-- 事务管理器 -->
            <dataSource /><!-- 数据源 -->
        </environment>
    </environments>
    <databaseIdProvider /><!-- 数据库厂商标识 -->
    <mappers /><!-- 映射器 -->
</configuration>
```

>mybatis-config.xml 文件中的元素节点是有一定顺序的，节点位置必须按以上位置排序，否则会编译错误。

### 2. MyBatis 常用配置解析

#### environments 标签

![](R:/ImageRepository/MyBatis/202111181616112.png)

事务管理器（transactionManager）类型有两种：

- JDBC：这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域
- MANAGED：这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为

数据源（dataSource）类型有三种：

- UNPOOLED：这个数据源的实现只是每次被请求时打开和关闭连接
- POOLED：这种数据源的实现利用 “池” 的概念将 JDBC 连接对象组织起来
- JNDI：这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用

#### mapper 标签

该标签的作用是加载映射的，加载方式有如下几种：
- 使用相对于类路径的资源引用，例如：<mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
- 使用完全限定资源定位符（URL），例如：<mapper url="file:///var/mappers/AuthorMapper.xml"/>
- 使用映射器接口实现类的完全限定类名，例如：<mapper class="org.mybatis.builder.AuthorMapper"/>
- 将包内的映射器接口实现全部注册为映射器，例如：<package name="org.mybatis.builder"/>

#### properties 标签

实际开发中，习惯将数据源的配置信息单独抽取成一个 properties 文件，该标签可以加载额外配置的 properties 文件

![](R:/ImageRepository/MyBatis/202111181824226.png)

![](R:/ImageRepository/MyBatis/202111181836938.png)

#### typeAliases 标签

![](R:/ImageRepository/MyBatis/202111181835679.png)

MyBatis 框架已经为我们设置好了一些常用的类型的别名

![](R:/ImageRepository/MyBatis/202111181841864.png)

---

## 六、MyBatis 的相应 API

### 1. SqlSession 工厂构建器 SqlSessionFactoryBuilder

常用 API：SqlSessionFactory build(InputStream inputStream)

通过加载 MyBatis 的核心文件的输入流的形式构建一个 SqlSessionFactory 对象

```java
String resource = "org/mybatis/builder/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(inputStream);
```

其中，Resources 工具类，这个类在 org.apache.ibatis.io 包中。Resources 类可以帮助从类路径下、文件系统或一个 web URL 中加载资源文件

### 2. SqlSession 工厂对象 SqlSessionFactory

SqlSessionFactory 有多个方法创建 SqlSession 实例。常用的有如下两个：

|方法|解释|
|---|---|
|openSession()|会默认开启一个事务，但事务不会自动提交，也就意味着需要手动提交该事务，更新操作数据才会持久化到数据库中|
|openSession(boolean autoCommit)|参数为是否自动提交，如果设置为 true，那么不需要手动提交事务|

### 3. SqlSession 会话对象

SqlSession 实例在 MyBatis 中是非常强大的一个类。在这里你会看到所有执行语句、提交或回滚事务和获取映射器实例的方法。

执行语句的方法主要有：

```java
<T> T selectOne(String statement, Object parameter)
<E> List<E> selectList(String statement, Object parameter)
int insert(String statement, Object parameter)
int update(String statement, Object parameter)
int delete(String statement, Object parameter)
```

操作事务的方法主要有：

```java
void commit()
void rollback()
```

---

## 七、MyBatis 的 Dao 层实现

### 1. 传统开发方式

#### a> 编写 UserDao 接口

```java
public interface UserDao {
    List<User> findAll() throws IOException;
}
```

#### b> 编写 UserDaoImpl 实现

```java
import mybatis.dao.UserDao;
import mybatis.domain.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class UserDaoImpl implements UserDao {
    public List<User> findAll() throws IOException {
        InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        List<User> userList = sqlSession.selectList("userMapper.findAll");
        return userList;
    }
}
```

#### c> 测试传统方式

```java
@Test
public void testTraditionDao() throws IOException {
    UserDao userDao = new UserDaoImpl();
    List<User> all = userDao.findAll();
    System.out.println(all);
}
```

### 2. 代理开发方式

#### 2.1 代理开发方式介绍

采用 Mybatis 的代理开发方式实现 DAO 层的开发，这种方式是我们后面进入企业的主流。

Mapper 接口开发方法只需要程序员编写Mapper 接口（相当于Dao 接口），由Mybatis 框架根据接口定义创建接口的动态代理对象，代理对象的方法体同上边Dao接口实现类方法。

Mapper 接口开发需要遵循以下规范：
1. Mapper.xml文件中的namespace与mapper接口的全限定名相同
2. Mapper接口方法名和Mapper.xml中定义的每个statement的id相同
3. Mapper接口方法的输入参数类型和mapper.xml中定义的每个sql的parameterType的类型相同
4. Mapper接口方法的输出参数类型和mapper.xml中定义的每个sql的resultType的类型相同

#### 2.2 编写 UserMapper 接口

![](R:/ImageRepository/MyBatis/202111182146973.png)

#### 2.3 测试代理方式

```java
@Test
public void testProxyMapper() {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();
    
    //获得 MyBatis 框架生成的 UserMapper 接口的实现类
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    List<User> userList = mapper.findAll();
    System.out.println(userList);

    User user = mapper.findByID(1);
    System.out.println(user);
    
    //关闭会话对象
    sqlSession.close();
}
```

---

## 八、MyBatis 映射文件深入

### 1. 动态 SQL 语句

#### 1.1 动态 SQL 语句概述

动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其他类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。去利用动态 SQL，可以摆脱这种痛苦。

使用动态 SQL 并非易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语句，MyBatis 显著地提升了这一特性的易用性。

MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

- if
- choose(when, otherwise)
- trim(where, set)
- foreach

#### 1.2 动态 SQL 之 `<if>`

根据实体类的不同取值，使用不同的 SQL 语句来进行查询。

比如：在 id 如果不为空时可以根据 id 查询，如果 username 不为空时还要加入用户名作为条件。这种情况在多条件组合查询中经常会碰到。

```xml
<select id="findByCondition" parameterType="user" resultType="user">
    select * from user
    <where>
        <if test="id!=0">
            and id=#{id}
        </if>
        <if test="username!=null">
            and username=#{username}
        </if>
        <if test="password!=null">
            and password=#{password}
        </if>
    </where>
</select>
```

当查询条件 id、username 和 password 都存在时，控制台打印的 sql 语句如下：

```java
//获得 MyBatis 框架生成的 UserMapper 接口的实现类
UserMapper mapper = sqlSession.getMapper(UserMapper.class);

//模拟 user 对象
User condition = new User();
condition.setId(1);
condition.setUsername("张三");
condition.setPassword("123");
```

![](R:/ImageRepository/MyBatis/202111201922395.png)

当查询条件只有id存在时，控制台打印的sql语句如下：

```java
//获得 MyBatis 框架生成的 UserMapper 接口的实现类
UserMapper mapper = sqlSession.getMapper(UserMapper.class);

//模拟 user 对象
User condition = new User();
condition.setId(1);
```

![image-20211120192321819](R:/ImageRepository/MyBatis/202111201923882.png)

#### 1.3 动态 SQL 之 `<foreach>`

循环执行 SQL 的拼接操作。例如：`SELECT * FROM USER WHERE id IN (1,2,5)`

```xml
<select id="findByID" parameterType="list" resultType="user">
    select * from user
    <where>
        <foreach collection="list" open="id in(" close=")" item="id" separator=",">
            #{id}
        </foreach>
    </where>
</select>
```

测试代码如下：

```java
InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
SqlSession sqlSession = sqlSessionFactory.openSession();

//获得 MyBatis 框架生成的 UserMapper 接口的实现类
UserMapper mapper = sqlSession.getMapper(UserMapper.class);

//模拟 List<Integer> id 的数据
List<Integer> idList = new ArrayList<Integer>();
idList.add(1);
idList.add(3);

List<User> userList = mapper.findByID(idList);
System.out.println(userList);
```

![image-20211120194844054](R:/ImageRepository/MyBatis/202111201948227.png)

`<foreach>` 标签用于遍历集合。属性含义：

- `collection` 代表要遍历的集合元素，注意编写时不要写#{}
- `open` 代表语句的开始部分
- `close` 代表结束部分
- `item` 代表遍历集合的每个元素生成的变量名
- `sperator` 代表分隔符

### 2. SQL 片段抽取

Mapper.xml 文件中可将重复的 SQL 抽取出来，使用时用 `<include>` 引用即可，最终达到 SQL 重用的目的。

```xml
	<!-- SQL 语句抽取 -->
    <sql id="selectUser">select * from user</sql>

    <select id="findByCondition" parameterType="user" resultType="user">
        <include refid="selectUser"/>
        <where>
            <if test="id!=0">
                and id=#{id}
            </if>
            <if test="username!=null">
                and username=#{username}
            </if>
            <if test="password!=null">
                and password=#{password}
            </if>
        </where>
    </select>

    <select id="findByID" parameterType="list" resultType="user">
        <include refid="selectUser"/>
        <where>
            <foreach collection="list" open="id in(" close=")" item="id" separator=",">
                #{id}
            </foreach>
        </where>
    </select>
```

---

## 九、MyBatis 核心配置文件深入

### 1 typeHandlers 标签

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时，都会用类型处理器将获取的值以合适的方式转换成 Java 类型。下表描述了一些默认的类型处理器（截取部分）。

![image-20211120200322734](R:/ImageRepository/MyBatis/202111202003842.png)

你可以重写类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型。具体做法为：实现 org.apache.ibatis.type.TypeHandler 接口，或继承一个很便利的类 org.apache.ibatis.type.BaseTypeHandler，然后可以选择性地将它映射到一个 JDBC 类型。例如需求：一个 Java 中的 Date 数据类型，我想将之存到数据库的时候存成一个 1970 年至今的毫秒数，取出来时转换成 java 的 Date，即 java 的 Date 与数据库的 varchar 毫秒值之间转换。

开发步骤：
1. 定义转换类继承类 BaseTypeHandler<T>
2. 覆盖 4 个未实现的方法，其中 setNonNullParameter 为 java 程序设置数据到数据库的回调方法，getNullableResult 为查询时 mysql 的字符串类型转换成 java 的 Type 类型的方法
```java
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Date;

public class DateTypeHandler extends BaseTypeHandler<Date> {
    //将 java 类型转化成数据库需要的类型
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, Date date, JdbcType jdbcType) throws SQLException {
        long time = date.getTime();
        preparedStatement.setLong(i, time);
    }

    /**
     * 将数据库中类型转换为 java 类型
     * String 参数    要转换的字段名称
     * ResultSet 参数 查询出的结果集
     * @param resultSet
     * @param s
     * @return
     * @throws SQLException
     */
    public Date getNullableResult(ResultSet resultSet, String s) throws SQLException {
        //获取结果集中需要的数据 Long 转换成 Date 类型返回
        long aLong = resultSet.getLong(s);
        Date date = new Date(aLong);
        return date;
    }

    //将数据库中类型转换为 java 类型
    public Date getNullableResult(ResultSet resultSet, int i) throws SQLException {
        //获取结果集中需要的数据 Long 转换成 Date 类型返回
        long aLong = resultSet.getLong(i);
        Date date = new Date(aLong);
        return date;
    }

    //将数据库中类型转换为 java 类型
    public Date getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        long aLong = callableStatement.getLong(i);
        Date date = new Date(aLong);
        return date;
    }
}
```
3. 在 MyBatis 核心配置文件中进行注册
```xml
<!-- 注册类型处理器 -->
<typeHandlers>
    <typeHandler handler="mybatis.handler.DateTypeHandler"/>
</typeHandlers>
```
4. 测试转换是否正确
```java
@Test
public void testSave() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();

    //获得 MyBatis 框架生成的 UserMapper 接口的实现类
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    //创建 user 对象
    User user = new User();
    user.setUsername("测试");
    user.setPassword("zxc");
    user.setBirthday(new Date());

    //执行保存操作
    mapper.save(user);

    sqlSession.commit();
    sqlSession.close();
}
```
```java
@Test
public void testSelect() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();

    //获得 MyBatis 框架生成的 UserMapper 接口的实现类
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    User user = mapper.findByID(9);
    System.out.println("user.getBirthday() = " + user.getBirthday());

    sqlSession.commit();
    sqlSession.close();
}
```

>数据库数据：
>
>![](R:/ImageRepository/MyBatis/202111202120054.png)
>
>测试查询操作结果：
>
>user.getBirthday() = Sat Nov 20 21:08:17 CST 2021

### 2 plugins 标签

MyBatis 可以使用第三方的插件来对功能进行扩展。
分页助手 PageHelper 是将分页的负责操作进行封装，使用简单的方式即可以获得分页的相关数据。

开发步骤：

1. 导入通用 PageHelper 的坐标
```xml
<!-- 分页助手 -->
<dependencies>
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
        <version>3.7.5</version>
    </dependency>
    <dependency>
        <groupId>com.github.jsqlparser</groupId>
        <artifactId>jsqlparser</artifactId>
        <version>0.9.1</version>
    </dependency>
</dependencies>
```

2. 在 MyBatis 核心配置文件中配置 PageHelper 插件
```xml
<!-- 注意：分页助手的插件 配置在通用馆mapper之前 -->
<plugin interceptor="com.github.pagehelper.PageHelper">
    <!-- 指定方言 -->
    <property name="dialect" value="mysql"/>
</plugin>
```

3. 测试分页数据获取
```java
@Test
public void testSelectAll() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();

    //设置分页相关参数  当前页 + 每页显示条数
    PageHelper.startPage(1, 3);

    //获得 MyBatis 框架生成的 UserMapper 接口的实现类
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    for (Order user : Mapper.findAll()) {
        System.out.println(user);
    }

    sqlSession.commit();
    sqlSession.close();
}
```

获得分页相关的其他参数：
```java
@Test
public void testSelectAll() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();

    //设置分页相关参数  当前页 + 每页显示条数
    PageHelper.startPage(1, 3);

    //获得 MyBatis 框架生成的 UserMapper 接口的实现类
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    for (Order user : Mapper.findAll()) {
        System.out.println(user);
    }


    //获取与分页相关参数
    PageInfo<User> pageInfo = new PageInfo<User>(userList);
    System.out.println("当前页：" + pageInfo.getPageNum());
    System.out.println("每页显示页数：" + pageInfo.getPageSize());
    System.out.println("总条数：" + pageInfo.getTotal());
    System.out.println("总页数：" + pageInfo.getPages());
    System.out.println("上一页：" + pageInfo.getPrePage());
    System.out.println("下一页：" + pageInfo.getNextPage());
    System.out.println("是否是第一页：" + pageInfo.isIsFirstPage());
    System.out.println("是否是最后一页：" + pageInfo.isIsLastPage());

    sqlSession.commit();
    sqlSession.close();
}
```

3. 知识小结

- properties 标签：该标签可以加载外部的 properties 文件
- typeAliases 标签：设置类别名
- environments 标签：数据源环境配置标签
- typeHandlers 标签：配置自定义类型处理器
- plugin 标签：配置 MyBatis 的插件

---

## 十、多表操作

### 1. 一对一查询

#### 1.1 一对一查询的模型

用户表和订单表的关系为，一个用户有多个订单，一个订单只从属于一个用户
一对一查询的的需求：查询一个订单，与此同时查询出该订单所属的用户

![](R:/ImageRepository/MyBatis/202111211020360.png)

#### 1.2 订单环境搭建

a> 数据库 user 用户表和 order 订单表

```sql
/*
 Navicat Premium Data Transfer

 Source Server         : mysql 8.0.25
 Source Server Type    : MySQL
 Source Server Version : 80025
 Source Host           : localhost:3306
 Source Schema         : test_mybatis

 Target Server Type    : MySQL
 Target Server Version : 80025
 File Encoding         : 65001

 Date: 21/11/2021 12:55:40
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for order
-- ----------------------------
DROP TABLE IF EXISTS `order`;
CREATE TABLE `order`  (
  `id` int(0) UNSIGNED NOT NULL AUTO_INCREMENT,
  `ordertime` varbinary(255) NOT NULL,
  `total` double(50, 0) NOT NULL,
  `uid` int(0) UNSIGNED NOT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  INDEX `order_ibfk_1`(`uid`) USING BTREE,
  CONSTRAINT `order_ibfk_1` FOREIGN KEY (`uid`) REFERENCES `user` (`id`) ON DELETE RESTRICT ON UPDATE RESTRICT
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of order
-- ----------------------------
INSERT INTO `order` VALUES (1, 0x323032312D31312D31352031303A32383A3437, 3000, 1);
INSERT INTO `order` VALUES (2, 0x323032312D30362D31372031373A32393A3133, 5800, 1);
INSERT INTO `order` VALUES (3, 0x323031372D30372D31322030373A32393A3333, 323, 2);
INSERT INTO `order` VALUES (4, 0x323031372D30362D32332030343A32393A3532, 2345, 1);
INSERT INTO `order` VALUES (5, 0x323032312D30312D32322031393A33303A3130, 100, 2);
INSERT INTO `order` VALUES (6, 0x323032302D30322D30372031303A33303A3331, 2009, 3);

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
  `id` int(0) UNSIGNED NOT NULL AUTO_INCREMENT,
  `username` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `password` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `birthday` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 9 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES (1, '张三', '123', NULL);
INSERT INTO `user` VALUES (2, '李四', '123', NULL);
INSERT INTO `user` VALUES (3, '王五', '123', NULL);
INSERT INTO `user` VALUES (4, '赵六', '123', NULL);
INSERT INTO `user` VALUES (5, '田七', '123', NULL);
INSERT INTO `user` VALUES (9, '测试', 'zxc', NULL);

SET FOREIGN_KEY_CHECKS = 1;
```

>一对一查询 SQL 语句如下：
>`select order.*, user.username, user.password from order,user where order.uid = user.id;`
>
>查询结果：
>![](R:/ImageRepository/MyBatis/202111211216084.png)

b> 创建 user、order 实体类
User.java
```java
public class User {
    private Integer id;
    private String username;
    private String password;
    private Date birthday;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", birthday=" + birthday +
                '}';
    }
}
```
Order.java
```java
public class Order {
    private Integer id;
    private Date orderTime;
    private double total;

    private User user;  //uid

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Date getOrderTime() {
        return orderTime;
    }

    public void setOrderTime(Date orderTime) {
        this.orderTime = orderTime;
    }

    public double getTotal() {
        return total;
    }

    public void setTotal(double total) {
        this.total = total;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    @Override
    public String toString() {
        return "Order{" +
                "id=" + id +
                ", orderTime=" + orderTime +
                ", total=" + total +
                ", user=" + user +
                '}';
    }
}
```

c> 创建 Mapper 接口
```java
public interface OrderMapper {
    List<Order> findAll();
}
```

d> 配置 Mapper.xml 映射器
```xml
<mapper namespace="mybatis.mapper.OrderMapper">
    <resultMap id="orderMap" type="order">
        <!-- 手动指定字段与实体属性的映射关系
                column：数据表的字段名称
                property：实体的属性名称
         -->
        <id column="id" property="id"/>
        <result column="orderTime" property="orderTime"/>
        <result column="total" property="total"/>
        <!--<result column="uid" property="user.id"/>-->
        <!--<result column="username" property="user.username"/>-->
        <!--<result column="password" property="user.password"/>-->

        <!--
            property：当前实体（order）中的属性名称（private User user）
            javaType：当前实体（order）中的属性的类型（user）
         -->
        <association property="user" javaType="user">
            <id column="uid" property="id"/>
            <result column="username" property="username"/>
            <result column="password" property="password"/>
        </association>
    </resultMap>
    <select id="findAll" resultMap="orderMap">
        select `order`.*, `user`.username, `user`.`password` from `order`,`user` where `order`.uid = `user`.id;
    </select>
</mapper>
```

e> 在 MyBatis 核心配置文件中注册映射器
```xml
<!-- mapper 映射器 -->
<mappers>
    <mapper resource="mapper/OrderMapper.xml"/>
</mappers>
```

f> 测试代码
```java
private OrderMapper orderMapper;

@Before
public void before() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession(true);
    orderMapper = sqlSession.getMapper(OrderMapper.class);
}

@Test
public void testOrderMap() throws IOException {
    for (Order order : orderMapper.findAll()) {
        System.out.println(order);
    }
}
```

>结果：
>
>![](R:/ImageRepository/MyBatis/202111211249209.png)

### 2. 一对多查询

#### 2.1 一对多查询模型

用户表和订单表的关系为，一个用户有多个订单，一个订单只从属于一个用户
一对多查询的需求：查询一个用户，与此同时查询出该用户具有的订单

![](R:/ImageRepository/MyBatis/202111211308926.png)

>一对多查询 SQL 语句：
>`SELECT user.*, order.id AS oid, order.ordertime, order.total FROM user, order WHERE user.id = order.uid`
>
>查询的结果如下：
>![](R:/ImageRepository/MyBatis/202111211349660.png)

#### 2.2 实现步骤

a> 修改 user 实体
```java
public class User {
    private Integer id;
    private String username;
    private String password;
    private Date birthday;
    private List<Order> orderList;  //当前用户存在哪些订单

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public List<Order> getOrderList() {
        return orderList;
    }

    public void setOrderList(List<Order> orderList) {
        this.orderList = orderList;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", birthday=" + birthday;
    }
}
```

b> 创建 UserMapper 接口
```java
public interface UserMapper {
    List<User> findAll();
}
```

c> 配置 UserMapper.xml
```xml
<mapper namespace="mybatis.mapper.UserMapper">
    <resultMap id="userMap" type="user">
        <id column="id" property="id"/>
        <result column="username" property="username"/>
        <result column="password" property="password"/>
        <result column="birthday" property="birthday"/>
        <!-- 配置集合信息
                property：集合名词
                ofType：当前集合中的数据类型
         -->
        <collection property="orderList" ofType="order">
            <!-- 封装 order 数据 -->
            <id column="oid" property="id"/>
            <result column="orderTime" property="orderTime"/>
            <result column="total" property="total"/>
        </collection>
    </resultMap>

    <select id="findAll" resultMap="userMap">
        SELECT `user`.*, `order`.id AS oid, `order`.ordertime, `order`.total FROM `user`, `order` WHERE `user`.id = `order`.uid
    </select>
</mapper>
```

d> 测试代码
```java
private UserMapper userMapper;

@Before
public void before() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession(true);
    userMapper = sqlSession.getMapper(UserMapper.class);
}

@Test
public void testUserMap() {
    for (User user : userMapper.findAll()) {
        System.out.println(user);
        for (Order order : user.getOrderList()) {
            System.out.println(order);
        }
        System.out.println("-------------------------------------------");
    }
}
```

>测试结果：
>![](R:/ImageRepository/MyBatis/202111211412592.png)

### 3. 多对多查询

#### 3.1 多对多查询模型

用户表和角色表的关系为，一个用户有多个角色，一个角色被多个用户使用
多对多查询的需求：查询用户同时查询出该用户的所有角色
![](R:/ImageRepository/MyBatis/202111211414462.png)

>多对多查询语句：
>`SELECT user.*, role.rolename, role.roleDesc FROM user, user_role, role WHERE user.id = user_role.user_id AND role.id = user_role.role_id;`
>查询得到的结果：
>![](R:/ImageRepository/MyBatis/202111211449424.png)

#### 3.2 实现步骤

a> 设计数据库 role 表、user_role 表
```sql
/*
 Navicat Premium Data Transfer

 Source Server         : mysql 8.0.25
 Source Server Type    : MySQL
 Source Server Version : 80025
 Source Host           : localhost:3306
 Source Schema         : test_mybatis

 Target Server Type    : MySQL
 Target Server Version : 80025
 File Encoding         : 65001

 Date: 21/11/2021 14:50:37
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for role
-- ----------------------------
DROP TABLE IF EXISTS `role`;
CREATE TABLE `role`  (
  `id` int(0) UNSIGNED NOT NULL AUTO_INCREMENT,
  `roleName` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `roleDesc` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of role
-- ----------------------------
INSERT INTO `role` VALUES (1, '院长', '负责全面工作');
INSERT INTO `role` VALUES (2, '研究员', '课程研发工作');
INSERT INTO `role` VALUES (3, '讲师', '授课工作');
INSERT INTO `role` VALUES (4, '助教', '协助解决学生的问题');
INSERT INTO `role` VALUES (5, '班主任', '负责学生的生活');
INSERT INTO `role` VALUES (6, '就业指导', '负责学生的就业工作');

-- ----------------------------
-- Table structure for user_role
-- ----------------------------
DROP TABLE IF EXISTS `user_role`;
CREATE TABLE `user_role`  (
  `user_id` int(0) UNSIGNED NOT NULL,
  `role_id` int(0) UNSIGNED NOT NULL,
  PRIMARY KEY (`user_id`, `role_id`) USING BTREE,
  INDEX `role_id`(`role_id`) USING BTREE,
  CONSTRAINT `user_role_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `user_role_ibfk_2` FOREIGN KEY (`role_id`) REFERENCES `role` (`id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of user_role
-- ----------------------------
INSERT INTO `user_role` VALUES (1, 1);
INSERT INTO `user_role` VALUES (1, 2);
INSERT INTO `user_role` VALUES (2, 2);
INSERT INTO `user_role` VALUES (2, 3);

SET FOREIGN_KEY_CHECKS = 1;
```

b> 创建 Role 实体，修改 User 实体
User.java
```java
public class User {
    private Integer id;
    private String username;
    private String password;
    private Date birthday;
    private List<Order> orderList;  //用户存在的订单
    private List<Role> roleList;    //用户具备的角色

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public List<Order> getOrderList() {
        return orderList;
    }

    public void setOrderList(List<Order> orderList) {
        this.orderList = orderList;
    }

    public List<Role> getRoleList() {
        return roleList;
    }

    public void setRoleList(List<Role> roleList) {
        this.roleList = roleList;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", birthday=" + birthday +
                ", roleList=" + roleList +
                '}';
    }
}
```

Role.java
```java
public class Role {
    private Integer id;
    private String roleName;
    private String roleDesc;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getRoleName() {
        return roleName;
    }

    public void setRoleName(String roleName) {
        this.roleName = roleName;
    }

    public String getRoleDesc() {
        return roleDesc;
    }

    public void setRoleDesc(String roleDesc) {
        this.roleDesc = roleDesc;
    }

    @Override
    public String toString() {
        return "Role{" +
                "id=" + id +
                ", roleName='" + roleName + '\'' +
                ", roleDesc='" + roleDesc + '\'' +
                '}';
    }
}
```

c> 添加 UserMappper 接口方法
```java
List<User> findUserRoleAll();
```

d> 配置 UserMapper.xml
```xml
<mapper namespace="mybatis.mapper.UserMapper">
    <resultMap id="userRoleMap" type="user">
        <!-- 封装 user 信息 -->
        <id column="user_id" property="id"/>
        <result column="username" property="username"/>
        <result column="password" property="password"/>
        <result column="birthday" property="birthday"/>

        <!-- 封装 user 内部的 roleList 信息 -->
        <collection property="roleList" ofType="role">
            <!-- 封装 role 数据 -->
            <id column="role_id" property="id"/>
            <result column="roleName" property="roleName"/>
            <result column="roleDesc" property="roleDesc"/>
        </collection>
    </resultMap>

    <select id="findUserRoleAll" resultMap="userRoleMap">
        select * from `user`, `user_role`, `role` where `user`.id = `user_role`.user_id and `user_role`.role_id = `role`.id
    </select>
</mapper>
```

e> 测试
```java
private UserMapper userMapper;

@Before
public void before() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession(true);
    userMapper = sqlSession.getMapper(UserMapper.class);
    orderMapper = sqlSession.getMapper(OrderMapper.class);
}

@Test
public void testRole() {
    for (User user : userMapper.findUserRoleAll()) {
        System.out.println(user);
        for (Role role : user.getRoleList()) {
            System.out.println(role);
        }
        System.out.println("-------------------------------------------");
    }
}
```

>测试结果：
>![](R:/ImageRepository/MyBatis/202111211603352.png)


### 4. 知识小结

MyBatis 多表查询配置方式：

- 一对一配置：使用 `<resultMap>`
- 一对多配置：使用 `<resultMap>` + `<collection>`
- 多对多配置：使用 `<resultMap>` + `<collection>`

---

## 十一、注解开发

### 1. 常用注解

- `@Insert` 实现新增
- `@Update` 实现更新
- `@Delete` 实现删除
- `@Select` 实现查询
- `@Result` 实现结果封装
- `@Results` 可以与 `@Result` 一起使用，封装多个结果集
- `@One` 实现一对一结果集封装
- `@Mary` 实现一对多结果集封装

### 2. 增删改查

#### 2.1 注解开发时的 Mapper 接口

```java
@Insert("insert into user values (#{id}, #{username}, #{password}, #{birthday})")
void save(User user);

@Update("update user set username=#{username}, password=#{password} where id=#{id}")
void update(User user);

@Delete("delete from user where id=#{id}")
void delete(Integer id);

@Select("select * from user where id=#{id}")
User findByID(Integer id);

@Select("select * from user")
List<User> findAll();
```

#### 2.2 注解开发核心配置文件中的映射器

加载使用了注解的 Mapper 接口

```xml
<mappers>
    <!-- 扫描使用注解的类 -->
    <mapper class="mapper.UserMapper"></mapper>
</mappers>
```

或者指定扫描包含映射关系的接口所在的包也可以

```xml
<mappers>
    <!-- 扫描使用注解的类所在的包 -->
    <mapper class="mapper"></mapper>
</mappers>
```

#### 2.3 测试代码

```java
private UserMapper mapper;

@Before
public void before() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession(true);
    mapper = sqlSession.getMapper(UserMapper.class);
}

@Test
public void testSave() {
    User user = new User();
    user.setUsername("tom");
    user.setPassword("abc");
    mapper.save(user);
}

@Test
public void testUpdate() {
    User user = new User();
    user.setId(11);
    user.setUsername("jerry");
    user.setPassword("123");
    mapper.update(user);
}

@Test
public void testDelete() {
    mapper.delete(11);
}

@Test
public void testFindByID() {
    System.out.println(mapper.findByID(2));
}

@Test
public void testFindAll() {
    for (User user : mapper.findAll()) {
        System.out.println(user);
    }
}
```

### 3. 注解实现复杂映射开发

实现复杂关系映射之前我们可以在 xml 映射文件中通过配置 `<resultMap>` 来实现。使用注解开发后，我们可以使用 `@Results` 注解、`@Result` 注解、`@One` 注解、`@Many` 注解组合完成复杂关系的配置

|注解|说明|
|---|---|
|@Results|代替的是标签 `<resultMap>` 该注解中可以使用单个 @Result 注解，也可以使用 @Result 集合。<br> 使用格式：<br> `@Results({@Result(), @Result()})` <br> 或 <br> `@Results(@Result)`|
|@Result|代替了 `<id>` 标签和 `<result>` 标签。<br> @Result 中属性介绍：<br> column：数据库的列名 <br> property：需要装配的属性名 <br> one：需要使用的 @One 注解 (@Result(one=@One)()) <br> many：需要使用的 @Many 注解 (@Result(many=@Many)())|
|@One|代替了 `<assocation>` 标签，是多表查询的关键，在注解中用来指定子查询返回单一对象。<br> @One 注解属性介绍：<br> select：指定用来多表查询的 sqlmapper <br> 使用格式：<br> `@Result(column="", property="", one=@One(select=""))`|
|@Many|代替了 `<collection>` 标签，是多表查询的关键，在注解中用来指定子查询返回对象集合。<br> 使用格式：<br> `@Result(property="", column="", many=@Many(select=""))`|

#### 3.1 一对一查询

##### 一对一查询的模型

用户表和订单表的关系为，一个用户有多个订单，一个订单只从属于一个用户
一对一查询的的需求：查询一个订单，与此同时查询出该订单所属的用户

![](R:/ImageRepository/MyBatis/202111211020360.png)

> 一对一查询 SQL 语句如下：
> `select order.*, user.username, user.password from order,user where order.uid = user.id;`
>
> 查询结果：
> ![](R:/ImageRepository/MyBatis/202111211216084.png)

##### a> 创建 User 和 Order 实体

```java
public class User {
    private Integer id;
    private String username;
    private String password;
    private Date birthday;

    private List<Order> orderList;  //用户订单

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public List<Order> getOrderList() {
        return orderList;
    }

    public void setOrderList(List<Order> orderList) {
        this.orderList = orderList;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", orderList=" + orderList +
                '}';
    }
}
```

```java
public class Order {
    private Integer id;
    private Date orderTime;
    private double total;

    private User user;  //uid

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public Date getOrderTime() {
        return orderTime;
    }

    public void setOrderTime(Date orderTime) {
        this.orderTime = orderTime;
    }

    public double getTotal() {
        return total;
    }

    public void setTotal(double total) {
        this.total = total;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    @Override
    public String toString() {
        return "Order{" +
                "id=" + id +
                ", orderTime=" + orderTime +
                ", total=" + total +
                ", user=" + user +
                '}';
    }
}
```

##### b> 创建 OrderMapper 接口

```java
public interface OrderMapper {
	List<Order> findAll();
}
```

##### c> 使用注解配置 Mapper

```java
public interface UserMapper {
    @Select("select * from `user` where id=#{id}")
    User findByID(Integer id);
}
```

```java
public interface OrderMapper {
    @Select("select * from `order`")
    @Results({
            @Result(column = "id", property = "id"),
            @Result(column = "orderTime", property = "orderTime"),
            @Result(column = "total", property = "total"),
            @Result(
                    property = "user",  //要封装的属性名称
                    column = "uid", //根据哪个字段去查询 property 属性值指定的表中数据
                    javaType = User.class,  //要封装的实体类型
                    one = @One(select = "mapper.UserMapper.findByID")   // select 属性：代表通过查询哪个接口的方法获取数据
            )

    })
	List<Order> findAll();
}
```

##### d> 测试

```java
private OrderMapper orderMapper;

@Before
public void before() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession(true);
    orderMapper = sqlSession.getMapper(OrderMapper.class);
}

@Test
public void testFindAll_Order() {
    for (Order order: orderMapper.findAll()) {
        System.out.println(order);
    }
}
```

> 测试结果：
>
> ![](R:/ImageRepository/MyBatis/202111232127593.png)

#### 3.2 一对多查询

##### 一对多查询的模型

用户表和订单表的关系为，一个用户有多个订单，一个订单只从属于一个用户
一对多查询的需求：查询一个用户，与此同时查询出该用户具有的订单

![](R:/ImageRepository/MyBatis/202111211308926.png)

>一对多查询 SQL 语句：
>`select * from user;`
>
>`select * from order where uid=查询出的用户id`
>
>查询的结果如下：
>![](R:/ImageRepository/MyBatis/202111211349660.png)

##### a> 修改 User 实体

```java
public class User {
    private Integer id;
    private String username;
    private String password;
    private Date birthday;

    private List<Role> roleList;    //用户角色

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public List<Order> getOrderList() {
        return orderList;
    }

    public void setOrderList(List<Order> orderList) {
        this.orderList = orderList;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", orderList=" + orderList +
                '}';
    }
}
```

##### b> 创建 UserMapper 接口方法

```java
List<User> findOrderAll();
```

##### c> 使用注解配置 UserMapper

```java
public interface OrderMapper {
    @Select("select * from `order` where uid=#{uid}")
	List<Order> findByUID(Integer uid);
}
```

```java
public interface UserMapper {
    @Select("select * from `user`")
    @Results({
            @Result(id = true, column = "id", property = "id"),
            @Result(column = "username", property = "username"),
            @Result(column = "password", property = "password"),
            @Result(
                    column = "id",
                    property = "orderList",
                    javaType = List.class,
                    many = @Many(select = "mapper.OrderMapper.findByUID")
            )
    })
    List<User> findOrderAll();
}
```

##### d> 测试

```java
private UserMapper userMapper;

@Before
public void before() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession(true);
    userMapper = sqlSession.getMapper(UserMapper.class);
}

@Test
public void testFindOrderAll_User() {
    for (User user : userMapper.findOrderAll()) {
        System.out.println(user);
    }
}
```

> 测试结果：
>
> ![image-20211123215950681](R:/ImageRepository/MyBatis/202111232159802.png)

#### 3.3 多对多查询

##### 多对多查询的模型

用户表和角色表的关系为，一个用户有多个角色，一个角色被多个用户使用
多对多查询的需求：查询用户同时查询出该用户的所有角色
![](R:/ImageRepository/MyBatis/202111211414462.png)

>多对多查询语句：
>`select * from user;`
>
>`select * from role, user_role where role.id=user_role.role_id and user_role.user_id=用户的id`
>
>查询得到的结果：
>![](R:/ImageRepository/MyBatis/202111211449424.png)

##### a> 创建 Role 实体，修改 User 实体

```java
public class Role {
    private Integer id;
    private String roleName;
    private String roleDesc;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getRoleName() {
        return roleName;
    }

    public void setRoleName(String roleName) {
        this.roleName = roleName;
    }

    public String getRoleDesc() {
        return roleDesc;
    }

    public void setRoleDesc(String roleDesc) {
        this.roleDesc = roleDesc;
    }

    @Override
    public String toString() {
        return "Role{" +
                "id=" + id +
                ", roleName='" + roleName + '\'' +
                ", roleDesc='" + roleDesc + '\'' +
                '}';
    }
}
```

```java
public class User {
    private Integer id;
    private String username;
    private String password;
    private Date birthday;

    private List<Order> orderList;  //用户订单
    private List<Role> roleList;    //用户角色

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public List<Role> getRoleList() {
        return roleList;
    }

    public void setRoleList(List<Role> roleList) {
        this.roleList = roleList;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", roleList=" + roleList +
                '}';
    }
}
```

##### b> 添加 UserMapper 接口方法

```java
List<User> findRoleAll();
```

##### c> 使用注解配置 Mapper

```java
public interface RoleMapper {
        @Select("select * from user_role, role where user_role.role_id=role.id and user_role.user_id=#{id}")
    List<Role> findByUserID(Integer id);
}
```

```java
public interface UserMapper {
    @Select("select * from `user`")
    @Results({
            @Result(id = true, column = "id", property = "id"),
            @Result(column = "username", property = "username"),
            @Result(column = "password", property = "password"),
            @Result(
                    property = "roleList",
                    column = "id",
                    javaType = List.class,
                    many = @Many(select = "mapper.RoleMapper.findByUserID")
            )
    })
    List<User> findRoleAll();
}
```

##### d> 测试

```java
private UserMapper userMapper;

@Before
public void before() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();
    userMapper = sqlSession.getMapper(UserMapper.class);
}

@Test
public void test() {
    for (User user : userMapper.findRoleAll()) {
        System.out.println(user);
    }

}
```

> 测试结果：
>
> ![image-20211124212223839](R:/ImageRepository/MyBatis/202111242122978.png)

---

***Author DoubleZHE***
***2021.11.24***