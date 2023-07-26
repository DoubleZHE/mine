## Spring 中 @Component 和 @Bean 注解的区别

> `@Component` 注解：一个通用注解，可以用于任何类上，包括普通的 Java 类、业务逻辑组件、持久化对象等等。**通过 @Component 注解，Spring 会自动创建该类的实例注入到 Spring IOC 容器中。**
>
> `@Bean` 注解：适用于配置类中声明一个 Bean。通常用在配置类的方法上面，表示把这个方法的返回对象注册到 Spring IOC 容器中。**通过 @Bean 注解，可以自定义 Bean 的创建和初始化过程，包括指定 Bean 的名称、作用域、依赖关系等**

- 用途不同

  `@Component` 用于标识普通的类

  `Bean` 是在配置类中声明和配置 Bean 对象

- 使用方式不同

  `@Component` 是一个**类级别**的注解，Spring 通过 `@ComponentScan` 注解扫描并注册为 Bean

  `Bean` 通过**方法级别**的注解，在配置类中**手动**声明和配置 Bean

- 控制权不同

  `@Component` 注解修饰的类是由 Spring 框架来创建和初始化的

  `Bean` 注解允许开发人员手动控制 Bean 的创建和配置过程