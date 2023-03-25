# Spring 5

## 概述

1. Spring是轻量级的开源的JavaEE框架

2. Spring框架可以解决企业应用开发的复杂性

3. Spring有两个核心部分：`IOC` 和` Aop`
   * IOC：控制反转，把创建对象过程交给Spring进行管理
   * Aop：面向切面，不修改源代码进行功能增强
4. spring又叫做容器，spring作为容器，装的是java对象。可以让spring创建Java对象，给属性赋值
5. Spring特点
   - 方便解耦，简化开发
   - Aop编程支持
   - 方便程序测试
   - 方便和其他框架进行整合
   - 方便进行事务操作
   - 降低 API 开发难度

**5、现在课程中，选取Spring版本 5.x**

## 入门案例

1. 下载Spring5。下载地址：https://repo.spring.io/release/org/springframework/spring/

   使用的5.2.6版本

2. 打开idea工具，创建java工程

3. 导入Spring5相关jar包

4. 创建类，在这个类中创建方法

   ```java
   public class User {
       public void add(){
           System.out.println(("add..."));
       }
   }
   ```

5. 创建Spring配置文件，在配置文件配置创建的对象

   1）Spring配置文件使用xml格式 

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
   <!--    配置User对象创建-->
       <bean id="user" class="Spring5.User"></bean>
   </beans>
   ```

6. 进行测试代码编写

   ```java
   public class TestSpring5 {
       @Test
       public void testAdd(){
           //1.加载spring配置文件
           ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
   
           //2.获取配置创建的对象
           User user = context.getBean("user", User.class);
   
           System.out.println(user);
           user.add();
       }
   }
   ```
   
   结果：
   
      ![image-20210915162344](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/20210915162344.png)

------



## IOC

^2158fb

### IOC（概念和原理）

^f92f57

1. 什么是 IOC ？

   1）控制反转，把对象创建和对象之间的调用过程，交给 Spring 进行管理

   2）使用 IOC 目的：为了耦合度降低

2. IOC 底层原理

   1）xml 解析、工厂模式、反射

3. 画图讲解 IOC 底层原理

   ![image-20210915172237](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/20210915172237.png)

   ![image-20210915172823](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/20210915172823.png)
   
   ![image-20210915173621733](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20210915173621733.png)

------



### IOC（接口）

1. IOC 思想基于 IOC 容器完成，IOC 容器底层就是对象工厂

2. Spring 提供 IOC 容器实现两种方式：（两个接口）

   1）BeanFactory：IOC 容器基本实现，是 Spring 内部使用的接口，不提供开发人员进行使用

   - 加载配置文件时候不会创建对象，在 获取（使用）对象才去创建对象

   2）ApplicationContext：BeanFactory 接口的子接口，提供更多更强大的功能，一般由开发人员进行使用

   - 加载配置文件时候就会把配置文件对象进行创建

3. ApplicationContext接口有实现类

   ![image-20210915185204751](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20210915185204751.png)

------



### IOC 操作 Bean 管理（概念）

1. 什么是 Bean 管理？

   1）Bean 管理指的是两个操作

   2）Spring 创建对象
   
   3）Spring 注入属性

2. Bean 管理操作有两种方式

   1）基于 xml 配置文件方式实现

   2）基于注解方式实现

------



#### IOC操作Bean管理（基于xml方式）

1. **基于 xml 方式创建对象**

   ```xml
   <!--    配置User对象创建-->
       <bean id="user" class="Spring5.User"></bean>
   ```

   1）在 spring 配置文件中，使用 bean 标签，标签里面添加对应属性，就可以实现对象创建

   2）在 bean 标签有很对属性，介绍常见的属性

   - id 属性：唯一标识
   - class 属性：类全路径（包类路径）

   3）创建对象时侯，默认也是执行无参数构造方法完成对象创建

2. **基于 xml 方式注入属性**

   1）DI：依赖注入，就是注入属性

   - 第一种注入方式：使用set方法进行注入

     ① 创建类，定义属性和对应的set

     ```java
     public class Book {
     
         //创建属性
         private String bname;
         private  String bauthor;
     
         //创建属性对应的 set 方法
         public void setBname(String bname) {
             this.bname = bname;
         }
     
         public void setBauthor(String bauthor) {
             this.bauthor = bauthor;
         }
     }
     ```
   
     ② 在 spring 配置文件配置对象创建，配置属性注入
   
     ```xml
         <!--2、set方法注入属性-->
         <bean id="book" class="Spring5.Book">
             <!-- 使用property完成属性注入
                     name：类里面属性名称
                     value：向属性注入的值
              -->
             <property name="bname" value="易筋经"></property>
             <property name="bauthor" value="达摩祖师"></property>
         </bean>
     ```
   
     
   
   - 第二种注入方式：使用有参构造进行注入
   
     ① 创建类，定义属性，创建属性对应有参构造方法
   
     ```java
     public class Orders {
     
         //属性
         private String name;
         private String address;
     
         //有参构造
         public Orders(String name, String address) {
             this.name = name;
             this.address = address;
         }
         
     }
     ```
   
     ② 在spring配置文件中进行配置
   
     ```xml
     	<!-- 3、有参构造注入属性 -->
         <bean id="orders" class="Spring5.Orders">
             <constructor-arg name="name" value="PC"></constructor-arg>
             <constructor-arg name="address" value="China"></constructor-arg>
         </bean>
     ```
   
   - p名称空间注入---(底层依然是使用set方法进行注入)
   
     - 使用 p名称空间注入，可以简化基于xml配置方式
   
       ① 第一步，添加 p名称空间在配置文件中
   
       ![image-20210915213308547](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20210915213308547.png)
   
       ② 第二步，进行属性注入，在 bean标签里面进行操作
   
       ```xml
       	<!-- p名称空间注入 -->
           <bean id="book" class="Spring5.Book" p:bname="九阳神功" p:bauthor="无名氏">
           </bean>
       ```

------



##### IOC操作Bean管理（xml注入其他类型属性）

1. 字面量

   - null值

     ```xml
     	<!-- null值 -->
         <property name="address">
             <null/>
         </property>
     ```

     

   - 属性值包含特殊符号

     ```xml
     	<!-- 属性值包含特殊符号
                     1、把<>进行转义
                     2、把带特殊符号内容写到CDATA
              -->
         <property name="address">
             <value><![CDATA[<<南京>>]]></value>
         </property>
     ```

2. **注入属性-外部bean**

   1）创建两个类service类和dao类

   2）在service调用dao里面的方法

3. **注入属性-内部bean**

   1）`一对多`关系：部门和员工。一个部门有多个员工，一个员工属于一个部门，部门是一，员工是多。

   2）`在实体类之间表示一对多关系`

   ​	部门类代码：

   ```java
   public class Dept {
   
       private String dname;
   
       public void setDname(String dname) {
           this.dname = dname;
       }
   }
   ```

   ​	员工类代码：

   ```java
   public class Emp {
   
       private String ename;
       private String gender;
       private Dept dept;  //员工属于某一个部门，使用对象形式表示
   
       public void setDept(Dept dept) {
           this.dept = dept;
       }
   
       public void setEname(String ename) {
           this.ename = ename;
       }
   
       public void setGender(String gender) {
           this.gender = gender;
       }
   }
   ```

   3）在spring配置文件中进行配置

   ```xml
   	<!-- 内部bean -->
       <bean id="emp" class="Spring5.bean.Emp">
           <!-- 设置两个普通属性 -->
           <property name="ename" value="lucy"></property>
           <property name="gender" value="女"></property>
           <!-- 对象类型属性 -->
           <property name="dept">
               <bean id="dept" class="Spring5.bean.Dept">
                   <property name="dname" value="安保部"></property>
               </bean>
           </property>
       </bean>
   ```

4. **注入属性-级联赋值**

   1）第一种写法
   
   ```xml
       <!-- 级联赋值 -->
       <bean id="employee" class="Spring5.bean.Employee">
           <!-- 设置两个普通属性 -->
           <property name="ename" value="lucy"></property>
           <property name="gender" value="女"></property>
           <!-- 级联赋值 -->
           <property name="dept" ref="dept"></property>
       </bean>
       <bean id="dept" class="Spring5.bean.Dept">
           <property name="dname" value="财务部"></property>
       </bean>
   ```
   
   2）第二种写法
   
   ![image-20210919181751371](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20210919181751371.png)
   
   ```xml
       <!-- 级联赋值 -->
       <bean id="employee" class="Spring5.bean.Employee">
           <!-- 设置两个普通属性 -->
           <property name="ename" value="lucy"></property>
           <property name="gender" value="女"></property>
           <!-- 级联赋值 -->
           <property name="dept" ref="dept"></property>
           <property name="dept.dname" value="技术部"></property>
       </bean>
       <bean id="dept" class="Spring5.bean.Dept">
           <!--<property name="dname" value="财务部"></property>-->
       </bean>
   ```

------



##### IOC操作Bean管理（xml注入集合属性）

1. **注入数组类型属性**

2. **注入List集合类型属性**

3. **注入Map集合类型属性**

   ① 创建类，定义数组、List、Map、Set类型属性，生成对应的Set()方法

   ```java
   public class Student {
   
       //1、数组类型属性
       private String[] courses;
       //2、List集合类型属性
       private List<String> list;
       //3、Map集合类型属性
       private Map<String, String> maps;
       //4、Set集合类型属性
       private Set<String> sets;
   
       public void setCourses(String[] courses) {
           this.courses = courses;
       }
   
       public void setList(List<String> list) {
           this.list = list;
       }
   
       public void setMaps(Map<String, String> maps) {
           this.maps = maps;
       }
   
       public void setSets(Set<String> sets) {
           this.sets = sets;
       }
   }
   ```

   ② 在spring配置文件进行配置

   ```xml
       <!-- 1、集合类型属性注入 -->
       <bean id="student" class="CollectionType.Student">
           <!-- 数组类型属性注入 -->
           <property name="courses">
               <array>
                   <value>java课程</value>
                   <value>数据库课程</value>
               </array>
           </property>
   
           <!-- List类型属性注入 -->
           <property name="list">
               <list>
                   <value>张三</value>
                   <value>李四</value>
               </list>
           </property>
   
           <!-- Map类型属性注入 -->
           <property name="maps">
               <map>
                   <entry key="JAVA" value="java"></entry>
                   <entry key="PHP" value="php"></entry>
               </map>
           </property>
   
           <!-- Set类型属性注入 -->
           <property name="sets">
               <set>
                   <value>MySQL</value>
                   <value>Redis</value>
               </set>
           </property>
       </bean>
   ```

4. **在集合里面设置对象类型值**

   Spring配置文件代码：

   ```xml
       <bean>
   		<!-- 注入List集合类型，值是对象 -->
           <property name="courseList">
               <list>
                   <ref bean="course1"></ref>
                   <ref bean="course2"></ref>
               </list>
           </property>
   	</bean>
   
   	<!-- 创建多个course对象 -->
       <bean id="course1" class="CollectionType.Course">
           <property name="cName" value="Spring5框架"></property>
       </bean>
       <bean id="course2" class="CollectionType.Course">
           <property name="cName" value="MyBatis框架"></property>
       </bean>
   ```

   

5. **把集合注入部分提取出来**

   1）在spring配置文件中引入名称空间util

   Spring配置文件代码：

   ```xml
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:p="http://www.springframework.org/schema/p"
          xmlns:util="http://www.springframework.org/schema/util"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
   
   </beans>
   ```

   2）使用util标签完成List集合注入提取

   spring配置文件代码：

   ```xml
       <!-- 1、提取List集合类型属性注入 -->
       <util:list id="book_list">
           <value></value>
           <value>易筋经</value>
           <value>九阴真经</value>
           <value>九阳神功</value>
       </util:list>
   
       <!-- 2、提取List集合类型属性注入使用 -->
       <bean id="book" class="CollectionType.Book">
           <property name="bookList" ref="book_list"></property>
       </bean>
   ```

------



##### IOC操作Bean管理（FactoryBean）

1. Spring有两种类型bean，一种普通bean，另一种工厂bean（FactoryBean）

   - 普通bean：在配置文件中定义bean类型就是返回类型

   - 工厂bean（FactoryBean）：在配置文件中定义bean类型可以和返回类型不一样

     ① 第一步：创建类，让这个类作为工厂bean，实现接口FactoryBean

     ② 第二步：实现接口里面的方法，在实现的方法中定义返回的bean类型

     class类代码：

     ```java
     public class MyBean implements FactoryBean<Course> {
     
         //定义返回bean
         @Override
         public Course getObject() throws Exception {
             Course course = new Course();
             course.setcName("abc");
             return course;
         }
     
         @Override
         public Class<?> getObjectType() {
             return null;
         }
     }
     ```

     spring配置文件代码：

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <beans xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
     
         <bean id="myBean" class="FactoryBean.MyBean">
     
         </bean>
     
     </beans>
     ```

     测试类代码：

     ```java
         @Test
         public void testCollection3() {
             ApplicationContext context = new ClassPathXmlApplicationContext("bean3.xml");
             Course course = context.getBean("myBean", Course.class);
             System.out.println("course = " + course);
         }
     ```

------



##### IOC操作Bean管理（bean作用域）

1. 在Spring里面，设置创建bean实例是单实例还是多实例
2. 在Spring里面，默认情况下，bean是单实例对象

![image-20210921155742539](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20210921155742539.png)

3. 如何设置单实例还是多实例

1）在Spring配置文件bean标签里面有属性（scope）用于设置单实例还是多实例

2）scope属性值

> scope标签共有四个属性值：singleton、prototype、request、session
>
> singleton：默认值，表示是单实例对象。常用
>
> prototype：表示是多实例对象。常用
>
> request：将创建的实例对象放在请求中（仅了解，不常用）
>
> session：将创建的实例对象放在会话中（仅了解，不常用）

```xml
	<bean id="book" class="CollectionType.Book" scope="prototype">
        <property name="bookList" ref="book_list"></property>
    </bean>
```

![image-20210921160550203](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20210921160550203.png)

3)singleton和prototype区别

- singleton单实例，prototype多实例
- 设置scope值是singleton的时候，加载spring配置文件时候就会创建单实例对象；设置scope值是prototype时候，不是在加载spring配置文件时候创建对象，在调用getBean()方法时候创建多实例对象。

------



##### IOC操作Bean管理（bean生命周期）

> 生命周期：从对象创建到对象销毁的过程。

1. bean生命周期（步骤）

1）通过构造器创建bean实例（无参数构造）

2）为bean的属性设置值和对其他bean引用（调用set()方法）

3）调用bean的初始化的方法（需要进行配置初始化的方法）

4）bean可以使用了（对象获取到了）

5）当容器关闭时候，调用bean的销毁的方法（需要进行配置销毁的方法）

2. 演示bean生命周期

   类代码：

   ```java
   public class Orders {
       private String oName;
   
       //无参构造
       public Orders() {
           System.out.println("第一步：执行无参构造创建bean实例");
       }
   
       public void setoName(String oName) {
           this.oName = oName;
           System.out.println("第二步：调用set()设置属性值");
       }
   
       //创建执行的初始化的方法
       public void initMethod() {
           System.out.println("第三步：执行初始化的方法");
       }
   
       //创建执行的销毁方法
       public void destroyMethod(){
           System.out.println("第五步：执行销毁bean的方法");
       }
   
   }
   ```

   配置文件代码：

   ```xml
       <bean id="orders" class="bean.Orders" init-method="initMethod" destroy-method="destroyMethod">
           <property name="oName" value="手机"></property>
       </bean>
   ```

   测试代码：

   ```java
       @Test
       public void test4() {
           ApplicationContext context = new ClassPathXmlApplicationContext("bean4.xml");
           Orders order = context.getBean("orders", Orders.class);
           System.out.println("第四步：获取创建bean实例对象");
           System.out.println("order = " + order);
   
           //手动让bean实例销毁
           ((ClassPathXmlApplicationContext)context).close();
   
       }
   ```

   结果：

   ![image-20210922121713876](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20210922121713876.png)

3. bean的后置处理器。bean的生命周期共有七步。

   1）通过构造器创建bean实例（无参数构造）

   2）为bean的属性设置值和对其他bean引用（调用set()方法）

   **3）把bean实例传递bean后置处理器的方法**`postProcessBeforeInitialization`

   4）调用bean的初始化的方法（需要进行配置初始化的方法）

   **5）把bean实例传递bean后置处理器的方法**`postProcessAfterInitialization`

   6）bean可以使用了（对象获取到了）

   7）当容器关闭时候，调用bean的销毁的方法（需要进行配置销毁的方法）

4. 演示添加后置处理器效果

   1）创建类，实现接口BeanPostProcessor，创建后置处理器

   ```java
   public class MyBeanPost implements BeanPostProcessor {
   
       @Override
       public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
           System.out.println("在初始化之前执行的方法");
           return bean;
       }
   
       @Override
       public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
           System.out.println("在初始化之后执行的方法");
           return bean;
       }
   
   }
   ```

   2)在配置文件中配置后置处理器*（配置后在同一个配置文件中bean都会被添加后置处理器效果）*

   ```xml
   	<!-- 配置后置处理器 -->
       <bean id="myBeanPost" class="bean.MyBeanPost"></bean>
   ```

##### IOC操作Bean管理（xml自动装配）

1. 什么是自动装配？

   根据指定装配规则（属性名称或者属性类型），Spring自动将匹配的属性值进行注入。
   
   > 实现自动装配:
   >     bean标签中属性autowire，配置自动装配。autowire属性常用两个值：
   >         byName：根据属性名称自动注入。注入值bean的id值和类属性名称一样。
   >         byType：根据属性类型自动注入。属性类型必须唯一。

2. 演示自动装配过程

   1）根据属性名称自动注入
   
   ```xml
       <bean id="employee" class="Autowire.Employee" autowire="byName"></bean>
       <bean id="dept" class="Autowire.Dept"></bean>
   ```
   
   2）根据属性类型自动注入
   
   ```xml
   	<bean id="employee" class="Autowire.Employee" autowire="byType"></bean>
   	<bean id="dept" class="Autowire.Dept"></bean>
   ```

##### IOC操作Bean管理（外部属性文件）

1. 直接配置数据库信息

   1）配置德鲁伊连接池

   2）引入德鲁伊连接池依赖jar包

   spring配置文件代码：

   ```xml
       <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
           <property name="driverClassName" value="com.mysql。cj.jdbc.Driver"></property>
           <property name="url" value="jdbc:mysql://localhost:3306/userDb"></property>
           <property name="username" value="root"></property>
           <property name="password" value="962464"></property>
       </bean>
   ```

2. 引入外部属性文件配置数据库连接池

   1）创建外部属性文件，properties格式文件，写数据库信息；

   ![image-20210927205805924](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20210927205805924.png)

   2）把外部properties属性文件引入到spring配置文案中

   ​	 ***① 引入context名称空间***

   ```xml
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:p="http://www.springframework.org/schema/p"
          xmlns:util="http://www.springframework.org/schema/util"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/util http://www.springframework.org/schema/beans/spring-util.xsd
                              http://www.springframework.org/schema/context http://www.springframework.org/schema/beans/spring-context.xsd">
   ```

   ​	② 在spring配置文件使用标签引入外部属性文件

   ```xml
       <!-- 引入外部属性文件 -->
       <context:property-placeholder location="jdbc.properties"/>
   
       <!-- 配置连接池 -->
       <!-- DruidDataSource dataSource = new DruidDataSource(); -->
       <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
           <!--
               dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
               set方法注入
            -->
           <!-- 获取properties文件内容，根据key获取，使用spring表达式获取 -->
           <property name="driverClassName" value="${jdbc.driverClass}"></property>
           <property name="url" value="${jdbc.url}"></property>
           <property name="username" value="${jdbc.username}"></property>
           <property name="password" value="${jdbc.password}"></property>
       </bean>
   ```

#### IOC操作Bean管理（基于注解方式）

1. 什么是注解

1）注解是代码特殊标记。格式：@注解名称（属性名称 = 属性值，属性名称 = 属性值...）

2）使用注解，注解作用在类上面，方法上面，属性上面；

3）使用注解目的：简化xml配置。

2. Spring针对Bean管理中创建对象提供注解

1）`@Component`

2）`@Service`

3）`@Controller`

4）`@Repository`

**上面四个注解功能是一样的都可以用来创建bean实例。**

3. 基于注解方式实现对象创建

   ① 第一步 引入依赖

   spring-aop-x.x.x.jar 

   [spring-aop]:https://mvnrepository.com/artifact/org.springframework/spring-aop

   ② 第二步 开启组件扫描

   ```xml
       <!-- 开启组件扫描
           1、如果扫描多个包，多个包使用逗号隔开
           2、扫描包的上层目录
        -->
       <context:component-scan base-package="java"></context:component-scan>
   ```

   

   ③ 第三步 创建类，在类上面添加创建对象注解

   ```java
   import org.springframework.stereotype.Component;
   
   //在注解里面value属性值可以省略不写，默认值是类名称，首字母小写
   @Component   //<bean id="userService" class="..">
   public class UserService {
       public void add() {
           System.out.println("service add...");
       }
   }
   ```

4. 开启组件扫描细节配置

```xml
    <!-- 示例一
        use-default-filters="false" 表示现在不适用默认filter，自己配置filter
        context:include-filter  设置扫描哪些内容

        只扫描service包中带有Controller注解的类
     -->
    <context:component-scan base-package="service" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- 示例二
        下面配置扫描包所有内容
        context:exclude-filter  设置哪些内容不进行扫描

        只不扫描service包中带有Controller注解的类
     -->
    <context:component-scan base-package="service">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
```

5. 基于注解方式实现属性注入

1）`@Autowired` 根据属性类型进行自动装配

第一步 把service和dao对象创建，在service和dao类添加创建对象注解

第二步 在service注入dao对象，在service类添加dao类型属性，在属性上面使用注解

```java
@Service
public class UserService {
    //定义dao类型属性
    //不需要添加set()方法
    //添加注入属性注解
    @Autowired
    private UserDao userDao;

    public void add() {
        System.out.println("service add...");
        userDao.add();
    }
}
```

2）`@Qualifier` 根据属性名称进行注入（这个` @Qualifier `注解的使用，和上面`@Autowired`一起使用）

```java
    //定义dao类型属性
    //不需要添加set()方法
    //添加注入属性注解
    @Autowired  //根据类型进行注入
    @Qualifier(value = "userDaoImpl1")  //根据名称进行注入
    private UserDao userDao;
```

3）`@Resource` 可以根据类型注入，也可以根据名称注入

```java
//    @Resource   //根据类型进行注入
    @Resource(name = "userDaoImpl1")    //根据名称进行注入
    private UserDao userDao;
```

4）` @Value ` 注入普通类型属性

```java
    @Value(value = "abc")
    private String name;
```

6. 完全注解开发

1）创建配置类，用来替代xml配置文件

```java
@Configuration  //作为配置类，替代xml配置文件
@ComponentScan(basePackages = {"service", "dao"})
public class SpringConfig {
}
```

2）编写测试类

```java
    @Test
    public void testService2() {
        //加载配置类
        ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
        UserService userService = context.getBean("userService", UserService.class);
        System.out.println(userService);
        userService.add();
    }
```

## AOP

### 概念

1. 什么是AOP

1）面向切面编程（方面）。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

2）通俗描述：不通过修改源代码方式，在主干功能里面添加新功能

3）使用登录例子说明AOP

![image-20211006211613790](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211006211613790.png)

### 底层原理

1. AOP底层使用动态代理

1）有两种情况动态代理

第一种 有接口情况，使用JDK动态代理

- 创建接口实现类代理对象，增强类的方法

![image-AimBG54cQrMeTN8](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/AimBG54cQrMeTN8.png)

第二种 无接口情况，使用CGLIB动态代理

- 创建子类的代理对象，增强类的方法

![image-20211006213942708](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211006213942708.png)

#### JDK动态代理

1. 使用JDK动态代理，使用Proxy类里面的方法创建代理对象

![image-20211007110527330](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211007110527330.png)

1）调用newProxyInstance()方法

![image-20211007110616591](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211007110616591.png)

> 方法有三个参数：
>
> 第一个参数，类加载器
>
> 第二个参数，增强方法所在的类，这个类实现的接口，支持多个接口
>
> 第三个参数，实现这个接口InvocationHandler，创建代理对象，写增强的方法

2. 编写JDK动态代理代码

1）创建接口，定义方法

```java
public interface UserDao {
    public int add(int a, int b);
    
    public void update(String id);
}
```

2）创建接口实现类，实现方法

```java
public class UserDaoImpl implements UserDao {
    @Override
    public int add(int a, int b) {
        return a + b;
    }

    @Override
    public String update(String id) {
        return id;
    }
}
```

3）使用Proxy类创建接口代理对象

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

public class JDKProxy {

    public static void main(String[] args) {
        //创建接口实现类代理对象
        Class[] interfaces = {UserDao.class};
/*        Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                return null;
            }
        });*/

        UserDaoImpl userDao = new UserDaoImpl();
        UserDao dao = (UserDao) Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces, new UserDaoProxy(userDao));
        int result = dao.add(1, 2);
        System.out.println("result = " + result);
    }

}

//创建代理对象代码
class UserDaoProxy implements InvocationHandler {

    //1、把创建的谁的代理对象，把那个谁传递过来
    //有参构造传递
    private Object object;
    public UserDaoProxy(Object obj) {
        this.object = obj;
    }

    //增强的逻辑
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        //方法之前处理
        System.out.println("方法之前执行" + method.getName() + "; 传递参数" + Arrays.toString(args));

        //被增强的方法执行
        Object res = method.invoke(object, args);

        //方法之后处理
        System.out.println("方法之后执行" + object);

        return res;
    }
}
```

运行结果：

![image-20211007114918187](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211007114918187.png)

### 常见术语

1. 连接点

   类里面哪些方法可以被增强，这些方法称为连接点

2. 切入点

   实际被真正增强的方法，称为切入点

3. 通知（增强）

   1）实际增强的逻辑部分称为通知（增强）

   2）通知有多种类型

   - 前置通知
   - 后置通知
   - 环绕通知
   - 异常通知
   - 最终通知 `finally`

4. 切面

   是一种动作，把通知应用到切入点的过程

### 操作

#### 准备

1. Spring框架一般基于`AspectJ`实现AOP操作

   1）什么是AspectJ？

   AspectJ不是Spring组成部分，独立AOP框架，一般把AspectJ和Spring框架一起使用，进行AOP操作

2. 基于AspectJ实现AOP操作

   1）基于xml配置文件实现

   2）基于注解方式实现（使用）

3. 在项目工程里面引入AOP相关依赖

```xml
	<dependencies>
        <dependency>
            <groupId>net.sourceforge.cglib</groupId>
            <artifactId>com.springsource.net.sf.cglib</artifactId>
            <version>2.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.aopalliance</groupId>
            <artifactId>com.springsource.org.aopalliance</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>com.springsource.org.aspectj.weaver</artifactId>
            <version>1.6.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>
    </dependencies>
```

![image-20211009192359889](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211009192359889.png)

4. 切入点表达式

   1）切入点表达式作用：知道对哪个类里面的哪个方法进行增强

   2）语法结构：

   `execution([权限修饰符][返回类型][类全路径][方法名称]([参数列表]))`

   > 举例一：对dao.BookDao类里面的add()进行增强
   >
   > `execution(* dao.BookDao.add(..))`

   > 举例二：对dao.BookDao类里面所有的方法进行增强
   >
   > `execution(* dao.BookDao.*(..))`

   > 举例三：对dao.BookDao包里面所有类，类里面所有方法进行增强
   >
   > `execution(* dao.*.*(..))`

#### AspectJ注解方式

1. 创建类，在类里面定义方法

```java
public class User {
    public void add() {
        System.out.println("add...");
    }
}
```

2. 创建增强类（编写增强逻辑）

   1）在增强类里面，创建方法，让不同方法代表不同通知类型

```java
/**
 * @ClassName UserProxy
 * @Description 增强的类
 * @Author DoubleZHE
 * @Create 2021/10/9 20:36
 */

public class UserProxy {

    //前置通知
    public void before() {
        System.out.println("before...");
    }

}
```

3. 进行通知的配置

   1）在spring配置文件中，开启注解扫描

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:aop="http://www.springframework.org/schema/aop"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                              http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                              http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
   
       <!-- 开启注解扫描 -->
       <context:component-scan base-package="AopAnnotation"></context:component-scan>
       
   </beans>
   ```

   2）使用注解User和UserProxy对象

   ![image-20211009215012960](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211009215012960.png)

   ![image-20211009215039979](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211009215039979.png)

   3）在增强类上面添加注解`@Aspect`

   ![image-20211009215341818](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211009215341818.png)

   4）在spring配置文件中开启生成代理对象

   ```xml
       <!-- 开启AspectJ生成代理对象 -->
       <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
   ```

4. 配置不同类型的通知

   1）在增强类的里面，在作为通知方法上面添加通知类型注解，使用切入点表达式配置

   ```java
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Before;
   import org.springframework.stereotype.Component;
   
   @Component
   @Aspect //生成代理对象
   public class UserProxy {
   
       //前置通知
       //@Before注解表示作为前置通知
       @Before(value = "execution(* AopAnnotation.User.add(..))")
       public void before() {
           System.out.println("before...");
       }
   
       //后置通知（返回通知）
       @AfterReturning(value = "execution(* AopAnnotation.User.add())")
       public void afterReturning() {
           System.out.println("afterReturning...");
       }
   
       //最终通知
       @After(value = "execution(* AopAnnotation.User.add())")
       public void after() {
           System.out.println("after...");
       }
   
       //异常通知
       @AfterThrowing(value = "execution(* AopAnnotation.User.add())")
       public void afterThrowing() {
           System.out.println("afterThrowing...");
       }
   
       //环绕通知
       @Around(value = "execution(* AopAnnotation.User.add())")
       public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
           System.out.println("环绕之前...");
   
           //被增强的方法执行
           proceedingJoinPoint.proceed();
   
           System.out.println("环绕之后...");
       }
   
   }
   ```

5. 相同的切入点抽取

```java
    //相同切入点抽取
    @Pointcut(value = "execution(* AopAnnotation.User.add(..))")
    public void point() {

    }

    //前置通知
    //@Before注解表示作为前置通知
    @Before(value = "point()")
    public void before() {
        System.out.println("before...");
    }
```

6. 有多个增强类多个同一方法进行增强，设置增强类优先级

   1）在增强类上面添加注解@Order(数字类型值)，数字类型值越小优先级越高

   ```java
   @Component
   @Aspect
   @Order(1)
   public class PersonProxy
   ```

7. 完全使用注解开发

创建配置类，不需要xml配置文件

```java
@Configuration
@ComponentScan(basePackages = {"java"})
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class ConfigAop {
}
```

#### AspectJ配置文件方式（了解）

1. 创建两个类，增强类和被增强类，创建方法

![image-20211011185558028](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211011185558028.png)

2. 在spring配置文件中创建两个类对象

```xml
    <!-- 1、创建对象 -->
    <bean id="book" class="Aop_xml.Book"></bean>
    <bean id="bookProxy" class="Aop_xml.BookProxy"></bean>
```

3. 在spring配置文件中配置切入点

```xml
    <!-- 配置aop增强 -->
    <aop:config>
        <!-- 切入点 -->
        <aop:pointcut id="p" expression="execution(* Aop_xml.Book.buy(..))"/>

        <!--配置切面-->
        <aop:aspect ref="bookProxy">
            <!-- 增强作用在具体的方法上 -->
            <aop:before method="before" pointcut-ref="p"></aop:before>
        </aop:aspect>
    </aop:config>
```

## JdbcTemplate

### 概念和准备

1. 什么是JdbcTemplate

Spring框架对JDBC进行封装，使用JdbcTemplate可以方便实现对数据库操作

2. 准备工作

1）引入相关依赖

```xml
    <dependencies>
        <dependency>
            <groupId>net.sourceforge.cglib</groupId>
            <artifactId>com.springsource.net.sf.cglib</artifactId>
            <version>2.2.0</version>
        </dependency>

        <dependency>
            <groupId>org.aopalliance</groupId>
            <artifactId>com.springsource.org.aopalliance</artifactId>
            <version>1.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>com.springsource.org.aspectj.weaver</artifactId>
            <version>1.6.8.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.1.1</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.9</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.25</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>
    </dependencies>
```

2）在spring配置文件中配置数据库连接池

```xml
    <!-- 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <property name="url" value="jdbc:mysql:///user_db"/>
        <property name="username" value="root"/>
        <property name="password" value="962464"/>
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    </bean>
```

3）配置JdbcTemplate对象，注入DataSource

```xml
	<!-- JdbcTemplate对象 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!-- 注入dataSource -->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
```

4）创建service类、dao类，在dao注入jdbcTemplate对象

- 配置文案

```xml
    <!-- 开启组件扫描 -->
    <context:component-scan base-package="com.Spring5"></context:component-scan>
```

- Service

```java
 @Service
 public class BookService {
 //注入dao
 @Autowired
 private BookDao bookDao;
 }
```

- Dao

```java
@Repository
public class BookDaoImpl implements BookDao {
    //注入JdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;
}
```

### 操作数据库

#### 添加操作

1. 对应数据库创建实体类

2. 编写service和dao
   1. 在dao进行数据库添加操作
   2. 调用JdbcTemplate对象里面update方法实现添加操作
   
   ```java
   int update(String var1, @Nullable Object... var2) throws DataAccessException;
   ```
   
   - ​	有两个参数
     - 第一个参数：sql语句
     - 第二个参数：可变参数，设置sql语句值
   
   ```java
   import com.Spring5.entity.Book;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.jdbc.core.JdbcTemplate;
   import org.springframework.stereotype.Repository;
   
   @Repository
   public class BookDaoImpl implements BookDao {
   
       //注入JdbcTemplate
       @Autowired
       private JdbcTemplate jdbcTemplate;
   
       @Override
       public void add(Book book) {
           //1、创建sql语句
           String sql = "insert into t_book values(?, ?, ?)";
   
           Object[] args = {book.getBookId(), book.getBookName(), book.getBookAuthor()};
   
           //2、调用方法实现
           int result = jdbcTemplate.update(sql, args);
           System.out.println(result);
       }
   }
   ```

3. 测试类

#### 修改和删除

```java
    @Override
    public void update(Book book) {
        String sql = "update t_book set book_name=?, book_status=? where book_id=?";
        Object[] args = {book.getBookName(), book.getBookStatus(), book.getBookId()};

        int result = jdbcTemplate.update(sql, args);
        System.out.println(result);
    }

    @Override
    public void delete(String id) {
        String sql = "delete from t_book where book_id=?";
        int result = jdbcTemplate.update(sql, id);
        System.out.println(result);
    }
```

#### 查询

##### 返回某个值

1. 查询表里面有多少条记录，返回是某个值
2. 使用JdbcTemplate实现查询返回某个值代码

![image-20211019184609687](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211019184609687.png)

两个参数

- 第一个参数：sql语句
- 第二个参数：返回类型Class

```java
    @Override
    public int selectCount() {
        String sql = "select count(*) from t_book";
        Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
        return count;
    }
```

##### 返回对象

1. 场景：查询图书详情
2. JdbcTemplate实现查询返回对象

![image-20211019184355637](C:\Users\Doubl\AppData\Roaming\Typora\typora-user-images\image-20211019184355637.png)

三个参数

- 第一个参数：sql语句
- 第二个参数：RowMapper，是一个接口，返回不同类型数据，使用这个接口里面实现类完成数据封装
- 第三个参数：sql语句值

```java
    @Override
    public Book selectObjectInfo(int id) {
        String sql = "select * from t_book where book_id=?";
        Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class), id);
        return book;
    }
```

##### 返回集合

1. 场景：查询图书列表分页
2. 调用JdbcTempalte方法实现查询返回集合

![image-20211019192011325](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211019192011325.png)

三个参数

- 第一个参数：sql语句
- 第二个参数：RowMapper，是一个接口，返回不同类型数据，使用这个接口里面实现类完成数据封装
- 第三个参数：sql语句值

```java
    @Override
    public List<Book> selectObjectAll() {
        String sql = "select * from t_book";
        List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));
        return bookList;
    }
```

#### 批量操作

操作表里面多条记录。

![image-20211019200340462](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211019200340462.png)

两个参数

- 第一个参数：sql语句
- 第二个参数：List集合，添加多条记录数据

---



##### 批量添加

```java
    @Override
    public void batchAddBook(List<Object[]> batchArgs) {
        String sql = "insert into t_book values(?,?,?)";
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        System.out.println(Arrays.toString(ints));
    }
```

##### 批量修改

```java
    @Override
    public void batchUpdateBook(List<Object[]> batchArgs) {
        String sql = "uodata t_book set book_name = ?, book_author = ? where book_id = ?";
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        System.out.println(ints);
    }
```

##### 批量删除

```java
    @Override
    public void batchDeleteBook(List<Object[]> batchArgs) {
        String sql = "delete from t_book where book_id = ?";
        int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
        System.out.println(Arrays.toString(ints));
    }
```

### 测试代码

```java
	//测试添加数据
    @Test
    public void testJdbcTemplate_add(){
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        Book book = new Book();
        book.setBookId(2);
        book.setBookName("挪威的森林");
        book.setBookAuthor("村上春树");
        bookService.addBook(book);
    }

    //测试修改数据
    @Test
    public void testJdbcTemplate_update() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        Book book = new Book();
        book.setBookId(001);
        book.setBookName("老人与海");
        book.setBookAuthor("海明威");
        bookService.updateBook(book);
    }

    @Test
    public void testJdbcTemplate_delete() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        bookService.deleteBook(001);
    }

    //测试查询返回某个值
    @Test
    public void testJdbcTemplate_SelectCount() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        System.out.println("count = " + bookService.findCount());
    }

    //测试查询返回某个对象
    @Test
    public  void testJdbcTemplate_SelectObject() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        System.out.println(bookService.findObject(2));
    }

    //测试查询返回对象集合
    @Test
    public  void testJdbcTemplate_SelectObjectAll() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        System.out.println(bookService.findObjectAll());
    }

    //测试批量添加对象数据
    @Test
    public  void testJdbcTemplate_BatchAdd() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        List<Object[]> batchArgs = new ArrayList<>();

        Object[] objectList1 = {3, "活着", "余华"};
        Object[] objectList2 = {4, "朝花夕拾", "鲁迅"};
        Object[] objectList3 = {5, "沁园春", "毛泽东"};

        batchArgs.add(objectList1);
        batchArgs.add(objectList2);
        batchArgs.add(objectList3);
        bookService.batchAdd(batchArgs);
    }

    //测试批量修改对象数据
    @Test
    public  void testJdbcTemplate_BatchUpdate() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        List<Object[]> batchArgs = new ArrayList<>();

        //注意sql语句值的顺序！！！
        Object[] objectList1 = {"活着", "余华", 3};
        Object[] objectList2 = {"狂人日记", "鲁迅", 4};

        batchArgs.add(objectList1);
        batchArgs.add(objectList2);
        bookService.batchUpdate(batchArgs);
    }

    //测试批量删除对象数据
    @Test
    public  void testJdbcTemplate_BatchDelete() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        BookService bookService = context.getBean("bookService", BookService.class);
        List<Object[]> batchArgs = new ArrayList<>();

        //注意sql语句值！！！
        Object[] objectList1 = {3};
        Object[] objectList2 = {4};

        batchArgs.add(objectList1);
        batchArgs.add(objectList2);
        bookService.batchDelete(batchArgs);
    }
```

## 事务操作

### 概念

1. 什么是事务？

   1）事务是数据库操作最基本单元，逻辑上一组操作，要么都成功，如果有一个失败则所有操作都失败

   2）典型场景：银行转账 

   ![image-20211020215057243](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211020215057243.png)

2. 事务四大特性（ACID）
   - 原子性
   - 一致性
   - 隔离性
   - 持久性

### 搭建事务操作环境

1. 创建数据库表，添加记录

![image-20211020195020493](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211020195020493.png)

2. 创建service，搭建dao，完成对象的创建和注入

   service注入dao，在dao注入JdbcTemplate，在JdbcTemplate注入DataSource

   ```java
   import com.Spring5.dao.UserDao;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Service;
   
   @Service
   public class UserService {
       //注入dao
       @Autowired
       private UserDao userDao;
   }
   ```

   ```java
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.jdbc.core.JdbcTemplate;
   import org.springframework.stereotype.Repository;
   
   @Repository
   public class UserDaoImpl implements UserDao {
       //注入JdbcTemplate
       @Autowired
       private JdbcTemplate jdbcTemplate;
   }
   ```

3. 在dao创建两个方法：多钱和烧钱的方法，在service中创建方法（转账的方法）

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class UserDaoImpl implements UserDao {
    //注入JdbcTemplate
    @Autowired
    private JdbcTemplate jdbcTemplate;

    //lucy转账100给mary
    /**
     * 多钱的方法
     */
    @Override
    public void addMoney() {
        String sql = "update t_account set money = money + ? where username = ?";
        System.out.println(jdbcTemplate.update(sql, 100, "mary"));
    }

    //lucy转账100给mary
    /**
     * 少钱的方法
     */
    @Override
    public void reduceMoney() {
        String sql = "update t_account set money = money - ? where username = ?";
        System.out.println(jdbcTemplate.update(sql, 100, "lucy"));
    }
}
```

```java
import com.Spring5.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {
    //注入dao
    @Autowired
    private UserDao userDao;

    //转账的方法
    public void accountMoney() {
        //lucy少100
        userDao.reduceMoney();

        //mary多100
        userDao.addMoney();
    }
}
```

测试代码

```java
    @Test
    public void testAccount() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
        UserService userService = context.getBean("userService", UserService.class);
        userService.accountMoney();
    }
```

4. 上面代码，如果正常执行没有问题的，但是如果代码执行过程中出现异常，有问题

   1）使用事务进行解决

   2）事务操作的基本过程

   ![image-20211021162337537](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211021162337537.png)

### Spring事务管理介绍

1.  事务添加到JavaEE三层结构Service层（业务逻辑层）

2. 在Spring进行事务管理操作

   有两种方式：编程式事务管理和声明式事务管理（一般使用声明式事务管理）

3.  声明式事务管理

    1）基于注解方式（使用）

    2）基于xml配置文件方式

4. **在Spring进行声明式事务管理，底层使用AOP原理**

5. Spring事务管理API：提供一个接口，代表事务管理器，这个接口针对不同的框架提供不同的实现类

   ![image-20211021205833287](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211021205833287.png)

### 声明式事务管理

#### 基于注解方式

1. 在Spring配置文件中配置事务管理器

```xml
	<!-- 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <property name="url" value="jdbc:mysql://localhost:3306/user_db"/>
        <property name="username" value="root"/>
        <property name="password" value="962464"/>
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    </bean>

    <!--创建事务管理器-->
    <bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入数据源 -->
        <property name="dataSource" ref="dataSource"></property>
    </bean>
```

2. 在Spring配置文件中开启事务注解

   1）在spring配置文件引入名称空间tx

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
```

​	2)开启事务注解

```xml
	<!--创建事务管理器-->
    <bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入数据源 -->
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!-- 开启事务注解 -->
    <tx:annotation-driven transaction-manager="dataSourceTransactionManager"/>
```

3. 在service类上面（获取service类里面方法上面）添加事务注解

   1）`@Transactional`这个注解添加到类上面，也可以添加到方法上面

   2）如果把这个注解添加类上面，这个类里面所有的方法都添加事务

   3）如果把这个注解添加方法上面，为这个方法添加事务

```java
@Service
@Transactional
public class UserService {}
```

##### 参数配置

在service类上面添加`@Transactional`，在这个注解里面可以配置事务相关参数

![image-20211022183114078](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211022183114078.png)



- `propagation`：事务传播行为

  多事务方法直接进行调用，这个过程中事务是如何进行管理的

  ![image-20211022192724258](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211022192724258.png)

  ![image-20211022192326550](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211022192326550.png)

  ```java
  @Transactional(propagation = Propagation.REQUIRED)
  ```

  

- `isolation`：事务隔离级别

  1）事务有特性称为隔离性，多事务操作之间不会产生影响，不考虑隔离性产生很多问题

  2）有三个读问题：脏读、不可重复读、幻读

  ​	脏读：一个未提交事务读取到另一个未提交事务的数据

  ​	![image-20211022193530318](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211022193530318.png)

  ​	不可重复读：一个未提交事务读取到另一提交事务修改数据

  ​		![image-20211022193845389](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211022193845389.png)

  ​	幻读：一个未提交事务读取到另一提交事务添加数据

  3）通过设置事务隔级别，解决读问题

  ​	![image-20211022195057348](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/image-20211022195057348.png)

  ```java
  @Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.REPEATABLE_READ)
  ```

  如若不设置，MySQL默认隔离级别为`REPEATABLE READ（可重复读）`



- `timeout`：超时时间

  1）事务需要在一定时间内进行提交，如果不提交进行回滚

  2）默认值为-1，设置时间以秒单位进行计算



- `readOnly`：是否只读

  1）读：查询操作

  ​	  写：添加修改删除操作

  2）readOnly默认值false，表示可以查询，可以添加修改删除操作

  3）设置readOnly值是true，设置称true之后，只能查询



- `rollbackFor`：回滚

   设置出现哪些异常进行事务回滚



- `noRollbackFor`：不回滚

  设置出现哪些异常不进行事务回滚



#### 基于xml配置文件方式
1. 在spring配置文件中进行配置

	第一步 配置事务管理器
	
	第二步 配置通知
	
	第三步 配置切入点和切面

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
                           http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 开启组件扫描 -->
    <context:component-scan base-package="com.Spring5"/>

    <!-- 数据库连接池 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <property name="url" value="jdbc:mysql://localhost:3306/user_db"/>
        <property name="username" value="root"/>
        <property name="password" value="962464"/>
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    </bean>

    <!-- JdbcTemplate对象 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!-- 注入dataSource -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 创建事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 注入数据源 -->
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!-- 配置通知 -->
    <tx:advice id="advice_tx">
        <!-- 配置事物参数 -->
        <tx:attributes>
            <!-- 指定哪种规则的方法上面添加事务
                    <tx:method name="account*"/>
             -->
            <tx:method name="accountMoney" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <!-- 配置切入点和切面 -->
    <aop:config>
        <!-- 配置切入点 -->
        <aop:pointcut id="pt" expression="execution(* com.Spring5.service.UserService.*(..))"/>

        <!-- 配置切面 -->
        <aop:advisor advice-ref="advice_tx" pointcut-ref="pt"/>
    </aop:config>

</beans>	
```

#### 完全注解开发

1. 创建配置类，使用配置类替代xml配置文案

```java
import com.alibaba.druid.pool.DruidDataSource;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.ComponentScan;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.jdbc.core.JdbcTemplate;  
import org.springframework.jdbc.datasource.DataSourceTransactionManager;  
import org.springframework.transaction.annotation.EnableTransactionManagement;  
  
import javax.sql.DataSource;  
  
@Configuration //配置类  
@ComponentScan(basePackages = "com.Spring5")    //组件扫描  
@EnableTransactionManagement //开启事务  
public class TxConfig {  
  
    /**
     * 创建数据库连接池
     * @return
     */
    @Bean
    public DruidDataSource getDruidDataSource() {
        DruidDataSource dataSource = new DruidDataSource();

        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/user_db");
        dataSource.setUsername("root");
        dataSource.setPassword("962464");

        return dataSource;
    }

    /**
     * 创建JdbcTemplate对象
     * @param dataSource
     * @return
     */
    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
        //到IOC容器中根据类型找到dataSource
        JdbcTemplate jdbcTemplate = new JdbcTemplate();

        //注入dataSource
        jdbcTemplate.setDataSource(dataSource);

        return jdbcTemplate;
    }

    /**
     * 创建事务管理器
     * @param dataSource
     * @return
     */
    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();

        transactionManager.setDataSource(dataSource);

        return transactionManager;
    }
}
```

## Spring5框架新功能

### 整个Spring5框架的代码基于Java 8，运行时兼容JDK 9，许多不建议使用的类和方法在代码库中删除

### 整合日志框架

Spring 5.0框架自带了通用的日志封装。

Spring 5已经移除了Log4jConfigListener，官方建议使用Log4j2。

**Spring 5框架整合Log4j2**

第一步 引入jar包

```xml
<!-- 使用Log4j2的相关依赖 -->  
<dependency>  
	<groupId>org.apache.logging.log4j</groupId>  
	<artifactId>log4j-api</artifactId>  
	<version>2.12.1</version>  
</dependency>  
<dependency>  
	<groupId>org.apache.logging.log4j</groupId>  
	<artifactId>log4j-core</artifactId>  
	<version>2.12.1</version>  
</dependency>  
<dependency>  
	<groupId>org.apache.logging.log4j</groupId>  
	<artifactId>log4j-slf4j-impl</artifactId>  
	<version>2.12.1</version>  
</dependency>  
<dependency>  
	<groupId>org.slf4j</groupId>  
	<artifactId>slf4j-api</artifactId>  
	<version>1.7.30</version>  
</dependency>
```

第二步 创建log4j2.xml配置文件`文件名固定`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!-- 日志级别以及优先级排序：OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!-- Configuration后面的status用于设置log4j2自身内部的信息输出，可以不设置。当设置成trace时，可以看到log4j2内部各种详细输出 -->
<configuration  status="INFO">

    <!-- 先定义所有的appender -->
    <appenders>
        <!-- 输出日志信息到控制台 -->
        <console name="Console" target="SYSTEM_OUT">
            <!-- 控制日志输出的格式 -->
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} {%t} %-5level %logger{36} - %msg%n"/>
        </console>
    </appenders>

    <!-- 然后定义logger，只有定义了logger并引入的appender，appender才会生效 -->
    <!-- root：用于指定项目的根日志，如果没有单独制定Logger，则会使用root作为默认的日志输出 -->
    <loggers>
        <root level="info">
            <appender-ref ref="Console"/>
        </root>
    </loggers>

</configuration>
```

### Spring 5框架核心容器支持@Nullable注解

`@Nullable`注解可以使用在方法、属性、参数上面。分别表示方法返回可以为空，属性值可以为空，参数可以为空。

1. 注解用在方法上面，方法返回值可以为空。

```java
@Nullable  
String getId();
```

2. 注解使用在属性上面，属性可以为空。

```java
@Nullable
private String name;
```

3. 注解使用在方法参数里面，方法参数可以为空。

```java
public <T> void registerBean(@Nullable String beanName, Class<T> beanClass, @Nullable Supplier<T> supplier, BeanDefinitionCustomizer... customizers) {  
    this.reader.registerBean(beanClass, beanName, supplier, customizers);  
}
```

### Spring 5核心容器支持函数式风格 GenericApplicationContext

```java
//函数式风格创建对象，交给Spring进行管理  
@Test  
public void testGenericApplicationContext1() {  
    //1. 创建GenericApplicationContext对象  
    GenericApplicationContext context = new GenericApplicationContext();  
  
    //2. 调用context的方法对象注册  
    context.refresh();  
    context.registerBean(User.class, () -> new User());  
  
    //3. 获取在Spring注册的对象  
    User user = (User) context.getBean("com.Spring5.New.User");  
    System.out.println(user);  
}  
  
@Test  
public void testGenericApplicationContext2() {  
    //1. 创建GenericApplicationContext对象  
    GenericApplicationContext context = new GenericApplicationContext();  
  
    //2. 调用context的方法对象注册  
    context.refresh();  
    context.registerBean("user", User.class, () -> new User());  
  
    //3. 获取在Spring注册的对象  
    User user = (User) context.getBean("user");  
    System.out.println(user);  
}
```

### Webflux

#### SpringWebflux介绍
1. Webflux是 Spring5 添加新的模块，用于 web 开发的，功能和 SpringMVC 类似的，Webflux 使用当前一种比较流行的响应式编程出现的框架。

![image-202110261932898](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/202110261932898.png)

2. 使用传统 web 框架，比如 SpringMVC，这些基于 Servlet 容器，Webflux 是一种异步非阻塞的框架，异步非阻塞的框架在 Servlet3.1 以后才支持，核心是基于 Reactor 的相关 API 实现的。

3. 解释什么是异步非阻塞

- 异步和同步

    上面都是针对对象不一样。

    异步和同步针对调用者，调用者发送请求，如果等着对方回应之后才去做其他事情就是同步，如果发送请求之后不等着对方回应就去做其他事情就是异步。

- 非阻塞和阻塞

    阻塞和非阻塞针对被调用者，被调用者收到请求之后，做完请求任务之后才给出反馈就是阻塞，收到请求之后马上给出反馈然后再去做事情就是非阻塞。

4. Webflux 特点：

    非阻塞式：在有限资源下，提高系统吞吐量和伸缩性，以 Reactor 为基础实现响应式编程。

    函数式编程：Spring5 框架基于 java8，Webflux 使用 Java8 函数式编程方式实现路由请求。

5. 比较 SpringMVC

![image-202110262127693](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/202110262127693.png)

1）两个框架都可以使用注解方式，都运行在 Tomcat 等容器中

2）SpringMVC 采用命令式编程，Webflux 采用异步响应式编程

#### 响应式编程

1. 什么是响应式编程
响应式编程是一种面向数据流和变化传播的编程范式。这意味着可以在编程语言中很方便地表达静态或动态的数据流，而相关的计算模型会自动将变化的值通过数据流进行传播。
电子表格程序就是响应式编程的一个例子。单元格可以办函字面值或类似"=B1+C1"的公式，而包含公式的单元格的值会依据其他单元格的值的变化而变化。

2. Java 8及其之前版本
提供的**观察者**模式两个类：`Observer` 和 `Observable`。*已被淘汰*
```java
import java.util.Observable;

public class ObserverDemo extends Observable {

    public static void main(String[] args) {
        ObserverDemo observer = new ObserverDemo();

        //添加观察者
        observer.addObserver((o, arg) -> {
            System.out.println("发生变化");
        });
        observer.addObserver((o, arg) -> {
            System.out.println("收到被观察者通知，准备改变");
        });

        observer.setChanged();  //数据变化

        observer.notifyObservers(); //通知

    }

}
```

##### Reactor实现

1. 响应式编程操作中，Reactor 是满足 Reactive规范的框架。

2. Reactor 有两个核心类，`Mono` 和 `Flux`，这两个类实现接口 `Publisher`，提供丰富操作符。`Flux` 对象实现发布者，返回 N 个元素；`Mono` 实现发布者，返回 0 或者 1 个元素。

3. Flux 和 Mono 都是数据流的发布之，使用 Flux 和 Mono 都可以发出三种数据信号：元素值、错误信号、完成信号。
   错误信号和完成信号都代表终止信号。终止信号用于告诉订阅者数据流结束了，错误信号终止数据流同时把错误信息传递给订阅者。

   ![image-202110272123654](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/202110272123654.png)

4. 代码演示 Flux 和 Mono

第一步 引入依赖
```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.1.5.RELEASE</version>
</dependency>
```

第二步 编写代码
```java
    public static void main(String[] args) {
        //just 方法直接声明
        Flux.just(1, 2, 3, 4);
        Mono.just(1);

        //其他方法
        Integer[] array = {1, 2, 3, 4};
        Flux.fromArray(array);

        List<Integer> list = Arrays.asList(array);
        Flux.fromIterable(list);

        Stream<Integer> stream = list.stream();
        Flux.fromStream(stream);

    }
```

5. 三种信号特点

- 错误信号和完成信号都是终止信号，但是不能共存；
- 如果没有发送任何元素值，而是直接发送错误或者完成信号，表示是空数据流；
- 如果没有错误信号，并且没有完成信号，表示是无线数据流。

6. 调用 `just()` 或者其他方法只是声明数据流，数据流并没有发出，只有进行订阅之后才会触发数据流，不订阅什么都不会发生的。

```java
Flux.just(1, 2, 3, 4).subscribe(System.out::print);
Mono.just(1).subscribe(System.out::print);
```

7. 操作符*（两个常见的操作符）*

对数据流进行一道道操作，称为操作符。比如工厂流水线。

- map：元素映射为新元素

  ![image-202110281006723](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/202110281006723.png)

- flatMap：元素映射为流
  
  把每个元素转换流，把转换之后多个流合并为大的流。

  ![image-202110281010103](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/202110281010103.png)




#### Webflux执行流程和核心API

SpringWebflux基于Reactor，默认使用容器是 `Netty`，Netty 是高性能，NIO 框架，异步非阻塞的框架。

1. Netty

- BIO，异步阻塞

![image-202110281031473](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/202110281031473.png)

- NIO，异步非阻塞

![image-202110281032375](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/202110281032375.png)

2. SpringWebflux 执行过程和 SpringMVC 相似的

SpringWebflux 核心控制器 DispatchHandler，实现接口 WebHandler。

接口WebHandler有一个方法
```java
public interface WebHandler {
    Mono<Void> handle(ServerWebExchange exchange);
}
```

```java
    public Mono<Void> handle(ServerWebExchange exchange) {  //放http请求响应信息
        if (this.handlerMappings == null) {
            return this.createNotFoundError();
        } else {
            return CorsUtils.isPreFlightRequest(exchange.getRequest()) ? this.handlePreFlight(exchange) : Flux.fromIterable(this.handlerMappings).concatMap((mapping) -> {
                return mapping.getHandler(exchange);    //根据请求地址获取对应mapping
            }).next().switchIfEmpty(this.createNotFoundError()).flatMap((handler) -> {
                return this.invokeHandler(exchange, handler);   //调用具体的业务方法
            }).flatMap((result) -> {
                return this.handleResult(exchange, result);
            });
        }
    }
```

3. SpringWebflux里面DispatcherHandler,负责请求的处理

- `HandlerMapping`：请求查询到处理的方法
- `HandlerAdapter`：真正负责请求处理
- `HandlerResultHandler`：响应结果处理

4. SpringWebflux实现函数式编程，需要两个接口：`RouterFunction`（路由处理）和`HandlerFunction`（处理函数）

#### 基于注解编程模型

SpringWebflux实现方式有两种：注解编程模型和函数式编程模型。

使用注解编程模型方式，和之前SpringMVC使用相似，只需要把相关依赖配置到项目中，SpringBoot自动配置相关运行容器，默认情况下使用 Netty 服务器。

第一步 创建 SpringBoot 工程，引入webflux 依赖

![image-202110281411939](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/202110281411939.png)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

第二步 配置启动的端口号

![image-202110281414772](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E6%A1%86%E6%9E%B6/Spring5/202110281414772.png)

第三步 创建包和相关类

- 实体类

```java
public class User {

    private String name;
    private String gender;
    private Integer age;

    public User(String name, String gender, Integer age) {
        this.name = name;
        this.gender = gender;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

- 创建用户操作接口

```java
import doublezhe_student.webflux.entity.User;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

public interface UserService {

    /**
     * 根据id查询用户
     * @param id
     * @return Mono<User>
     */
    Mono<User> getUserByID(int id);

    /**
     * 查询所有用户
     * @return Flux<User>
     */
    Flux<User> getAllUser();

    /**
     * 添加用户
     * @param userMono
     * @return Mono<Void>
     */
    Mono<Void> saveUserInfo(Mono<User> userMono);

}
```

- 用户接口实现类

```java
import doublezhe_student.webflux.entity.User;
import doublezhe_student.webflux.service.UserService;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.util.HashMap;
import java.util.Map;

public class UserServiceImpl implements UserService {
    //创建 map 集合存储数据（未操作数据库，所以使用 Map 存储）
    private final Map<Integer, User> userMap = new Ha```java
    import doublezhe_student.webflux_function.entity.User;
import doublezhe_student.webflux_function.service.UserService;
import org.springframework.http.MediaType;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import static org.springframework.web.reactive.function.BodyInserters.fromObject;

public class UserHandler {
    private final UserService userService;

    public UserHandler(UserService userService) {
        this.userService = userService;
    }

    /**
     * 根据 id 查询
     * @param request
     * @return
     */
    public Mono<ServerResponse> getUserById(ServerRequest request) {
        //获取 id 值
        int userId = Integer.valueOf(request.pathVariable("id"));

        //空值处理
        Mono<ServerResponse> notFound = ServerResponse.notFound().build();

        //调用 service 方法得到数据
        Mono<User> userMono = this.userService.getBeanByID(userId);

        //把 userMono 进行转换返回
        //使用 Reactor 操作符 flatMap
        return
                userMono
                        .flatMap(person -> ServerResponse.ok().contentType(MediaType.APPLICATION_JSON)
                                .body(fromObject(person)))
                        .switchIfEmpty(notFound);
    }

    /**
     * 查询所有用户
     * @return
     */
    public Mono<ServerResponse> getAllUser() {
        //调用 service 得到结果
        Flux<User> users = this.userService.getAllBean();

        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(users, User.class);
    }

    /**
     * 添加用户
     * @param request
     * @return
     */
    public Mono<ServerResponse> saveUser(ServerRequest request) {
        //得到 User 对象
        Mono<User> userMono = request.bodyToMono(User.class);

        return ServerResponse.ok().build(this.userService.saveBeanInfo(userMono));
    }
}
​```shMap<>();

    public UserServiceImpl() {
        this.userMap.put(1, new User("lucy", "男", 20));
        this.userMap.put(2, new User("mary", "女", 30));
        this.userMap.put(3, new User("jack", "女", 50));
    }

    @Override
    public Mono<User> getUserByID(int id) {
        return Mono.justOrEmpty(this.userMap.get(id));
    }

    @Override
    public Flux<User> getAllUser() {
        return Flux.fromIterable(this.userMap.values());
    }

    @Override
    public Mono<Void> saveUserInfo(Mono<User> userMono) {
        return userMono.doOnNext(person -> {
            //向 map 集合里面放数据
            int id = userMap.size() + 1;
            userMap.put(id, person);
        }).thenEmpty(Mono.empty());
    }
}
```

- 创建 controller

```java
import doublezhe_student.webflux.entity.User;
import doublezhe_student.webflux.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
public class UserController {
    //注入service
    @Autowired
    private UserService userService;

    /**
     * 根据id查询
     * @param id
     * @return 
     */
    @GetMapping("/user/{id}")
    public Mono<User> getUserById(@PathVariable int id) {
        return userService.getBeanByID(id);
    }

    /**
     * 查询所有用户
     * @return
     */
    @GetMapping("/user")
    public Flux<User> getAllUser() {
        return userService.getAllBean();
    }

    /**
     * 添加用户
     * @param user
     * @return
     */
    @GetMapping("/saveUser")
    public Mono<Void> saveUser(@RequestBody User user) {
        Mono<User> userMono = Mono.just(user);
        return userService.saveBeanInfo(userMono);
    }
}
```

***说明*** 

SpringMVC 方式实现：同步阻塞的方式，基于 SpringMVC + Serblet + Tomcat
SpringWebflux 方式实现：异步非阻塞方式，基于 SpringWebflux + Reactor + Netty

#### 基于函数式编程模型

1. 在使用函数式编程模型操作时候，需要自己初始化服务器

2. 基于函数式编程模型时候，有两个核心接口：`RouterFunction`（实现路由功能，请求转发给对应的 handler）和 `HandlerFunction`（处理请求，生成响应的函数）。核心任务：定义两个函数式接口的实现并且启动需要的服务器。

3. SpringWebflux 请求和响应不再是 ServletRequest 和 ServletResponse，而是 ServerRequest 和 ServerResponse

***步骤***

第一步 把注解编程模型工程复制一份

第二步 创建 Handler（具体实现方法）

```java
import doublezhe_student.webflux_function.entity.User;
import doublezhe_student.webflux_function.service.UserService;
import org.springframework.http.MediaType;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import static org.springframework.web.reactive.function.BodyInserters.fromObject;

public class UserHandler {
    private final UserService userService;

    public UserHandler(UserService userService) {
        this.userService = userService;
    }

    /**
     * 根据 id 查询
     * @param request
     * @return
     */
    public Mono<ServerResponse> getUserById(ServerRequest request) {
        //获取 id 值
        int userId = Integer.valueOf(request.pathVariable("id"));

        //空值处理
        Mono<ServerResponse> notFound = ServerResponse.notFound().build();

        //调用 service 方法得到数据
        Mono<User> userMono = this.userService.getBeanByID(userId);

        //把 userMono 进行转换返回
        //使用 Reactor 操作符 flatMap
        return
                userMono
                        .flatMap(person -> ServerResponse.ok().contentType(MediaType.APPLICATION_JSON)
                                .body(fromObject(person)))
                        .switchIfEmpty(notFound);
    }

    /**
     * 查询所有用户
     * @return
     */
    public Mono<ServerResponse> getAllUser(ServerRequest request) {
        //调用 service 得到结果
        Flux<User> users = this.userService.getAllBean();

        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(users, User.class);
    }

    /**
     * 添加用户
     * @param request
     * @return
     */
    public Mono<ServerResponse> saveUser(ServerRequest request) {
        //得到 User 对象
        Mono<User> userMono = request.bodyToMono(User.class);

        return ServerResponse.ok().build(this.userService.saveBeanInfo(userMono));
    }
}
```

第三步 初始化服务器，编写 Router

- 创建路由的方法

```java
    //创建 Router 路由
    public RouterFunction<ServerResponse> responseRouterFunction() {
        //创建 handler 对象
        UserService userService = new UserServiceImpl();
        UserHandler handler = new UserHandler(userService);

        //设置路由
        return RouterFunctions.route(
                GET("/users/{id}").and(accept(MediaType.APPLICATION_JSON)), handler::getUserById)
                .andRoute(GET("/users").and(accept(MediaType.APPLICATION_JSON)), handler::getAllUser);
    }
```

- 创建服务器，完成适配

```java
    //创建服务器，完成适配
    public void createReactorServer() {
        //路由和 handler适配
        RouterFunction<ServerResponse> router = responseRouterFunction();
        HttpHandler httpHandler = toHttpHandler(router);
        ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(httpHandler);

        //创建服务器
        HttpServer httpServer = HttpServer.create();
        httpServer.handle(adapter).bindNow();
    }
```

- 最终调用

```java
    public static void main(String[] args) throws Exception {
        Server server = new Server();
        server.createReactorServer();

        System.out.println("enter to exit");
        System.in.read();
    }
```

4. 使用`WebClnent`调用

```java
    public static void main(String[] args) {
        //调用服务器地址
        WebClient webClient = WebClient.create("http://127.0.0.1:13950");

        //根据 id 查询
        String id = "1";
        User userResult = webClient.get().uri("/user/{id}", id).accept(MediaType.APPLICATION_JSON).retrieve().bodyToMono(User.class).block();
        System.out.println(userResult.getName());

        //查询所有
        Flux<User> result = webClient.get().uri("/users").accept(MediaType.APPLICATION_JSON).retrieve().bodyToFlux(User.class);
        result.map(stu -> stu.getName()).buffer().doOnNext(System.out::println).blockFirst();
    }
```

## 课程总结

1. Spring框架概述

- 轻量级开源 JavaEE 框架，为了解决企业复杂性。两个核心组成：`IOC`和`AOP`
- Spring 5.2.6版本

2. **IOC容器**

- **IOC底层原理（工厂、反射等）**

- IOC接口（BeanFactory）

- **IOC操作 Bean 管理（基于xml）**

- **IOC操作 Bean 管理（基于注解）**

3. **Aop**

- **AOP 底层原理：动态代理。有接口（JDK 动态代理），无接口（CGLIB动态代理）**
- **术语：切入点、增强（通知）、切面**
- **基于 AspectJ 实现 AOP 操作**

4. **JdbcTemplate**

- **使用 JdbcTemplate 实现数据库 curd 操作**
- **使用 JdbcTemplate 实现数据库批量操作**

5. **事务管理**

- **事务概念**
- **重要概念：传播行为和隔离级别**
- **基于注解实现声明式事务管理**
- **完全注解方式实现声明式事务管理**

6. **Spring 5 新特性**

- **整合日志框架**
- **@Nullable 注解**
- **函数式注册对象**
- **整合 JUnit5 单元测试框架**
- **SpringWebflux 使用**

---

***Author DoubleZHE***
***2021.10.28***