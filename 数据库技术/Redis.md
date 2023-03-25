MySQL 数据库，它是一种传统的关系型数据库，开发者可以使用 MySQL 来更好地管理和组织项目数据，虽然在小型 Web 应用下，只需要一个 MySQL + MyBatis 自带的缓存系统就可以胜任大部分的数据存储工作。<br>但是 MySQL 的缺点也很明显，它的数据始终是存储在硬盘上的，对于用户信息这种不需要经常发生修改的内容，使用 MySQL 存储确实可以，但是如果是快速更新或是频繁使用的数据，比如微博热搜、双十一秒杀，这些数据不仅要求服务器需要提供更高的响应速度，而且还需要面对短时间内上百万甚至上千万次访问，而 MySQL 的磁盘 IO 读写性能完全不能满足上面的需求，能够满足上述需求的只有内存，因为速度远高于磁盘 IO。

Redis 提供了更好的解决方案，来存储上述这类特殊数据，弥补 MySQL 的不足，以应对大数据时代的重重考验。

# Redis 数据库

Redis 不属于传统的数据库，属于 NoSQL。

## NoSQL 概论

NoSQL 全称是 Not Only SQL（不仅仅是 SQL）它是一种非关系型数据库，相比传统 SQL 关系型数据库，NoSQL 相较之的区别：

* 不保证关系数据的 ACID 特性

> ACID，是指数据库管理系统（DBMS）在写入或更新资料的过程中，为保证事务（transaction）是正确可靠的，所必须具备的四个特性：原子性（atomicity，或称不可分割性）、一致性（consistency）、隔离性（isolation，又称独立性）、持久性（durability）。

* 并不遵循 SQL 标准
* 消除数据之间关联性

NoSQL 的优势：

* 远超传统关系型数据库的性能
* 非常易于扩展
* 数据模型更加灵活
* 高可用

NoSQL 的优势正是高并发海量数据的解决方案！

NoSQL 数据库分为以下几种：

* **键值存储数据库：**所有的数据都是以键值方式存储的，类似于 HashMap，使用起来非常简单方便，性能也非常高。
* **列存储数据库：**这部分数据库通常是用来应对分布式存储的海量数据。键仍然存在，但是它们的特点是指向了多个列。
* **文档型数据库：**它是以一种特定的文档格式存储数据，比如 JSON 格式，在处理网页等复杂数据时，文档型数据库比传统键值数据库的查询效率更高。
* **图形数据库：**利用类似于图的数据结构存储数据，结合图相关算法实现高速访问。

Redis 数据库，就是一个开源的**键值存储数据库**，所有的数据全部存放在内存中，它的性能大大高于磁盘 IO，并且它也可以支持数据持久化，还支持横向扩展、主从复制等。

实际生产中，一般会配合使用 Redis 和 MySQL 以发挥它们各自的优势，取长补短。

## 基本操作

在使用 MySQL 时，需要先在数据库中创建一张表，并定义好表的每个字段内容，最后再通过 `insert` 语句向表中添加数据，而 Redis 并不具有 MySQL 那样的严格的表结构，Redis 是一个键值数据库，因此，可以像 Map 一样的操作方式，通过键值对向 Redis 数据库中添加数据（操作起来类似于向一个 HashMap 中存放数据）

在 Redis 下，数据库是由一个整数索引标识，而不是由一个数据库名称。 <br>默认情况下，连接 Redis 数据库之后，会使用 0 号数据库，开发者可以通过 Redis 配置文件中的参数来修改数据库总数，默认为 16 个。

```sql
select 序号;	#通过 select 语句进行切换数据库：
```

### 数据操作

- 向 Redis 数据库中添加数据：

    ```sql
    set <key> <value>
    mset [<key> <value>]...	#一次性添加多条数据
    ```

    所有存入的数据默认会以**字符串**的形式保存，键值具有一定的命名规范，以方便快速定位数据属于哪一个部分。<br>比如用户的数据：

    ```sql
    set user:info:用户ID:name lbw	#使用冒号来进行板块分割，表示用户 XXX 的信息中的 name 属性，值为lbw
    ```

- 通过键值获取存入的值：

    ```sql
    get <key>
    ```

- 它还支持数据的过期时间设定（当数据到达指定时间时，会被自动删除）：

    ```sql
    set <key> <value> EX 秒
    set <key> <value> PX 毫秒
    ```

    也可以单独为其他的键值对设置过期时间：

    ```sql
    expire <key> 秒
    ```

    通过下面的命令来查询某个键值对的过期时间还剩多少：

    ```sql
    ttl <key>	#使用此命令查看过期时间还剩多少时，结果为 -1 表示为永久，结果为 -2 表示已过期
    pttl <key>	#毫秒显示
    persist <key>	#转换为永久
    ```

- 删除数据（删除命令可以同时拼接多个键值一起删除）：

    ```sql
    del <key>...
    ```

- 查看数据库中所有的键值时：

    ```sql
    keys *
    ```

    也可以查询某个键是否存在：

    ```sql
    exists <key>...
    ```

    还可以随机拿一个键：

    ```sql
    randomkey
    ```

- 将一个数据库中的内容移动到另一个数据库中：

    ```sql
    move <key> 数据库序号
    ```

- 修改一个键为另一个键：

    ```sql
    rename <key> <新的名称>
    renamex <key> <新的名称>	#renamex 会检查新的名称是否已经存在
    ```
    
- 存放的数据是一个数字，对其进行自增自减操作：

    ```sql
    incr <key>	#等价于 a = a + 1
    incrby <key> b	#等价于 a = a + b
    decr <key>	#等价于 a = a - 1
    ```
    
- 查看值的数据类型：

    ```sql
    type <key>	#默认 string 类型存储
    ```

Redis 数据库也支持多种数据类型，但是它更偏向于在 Java 中认识的那些数据类型。

## 数据类型介绍

一个键值对除了存储一个 String 类型的值以外，还支持多种常用的数据类型。

### Hash

这种类型本质上就是一个 HashMap，也就是嵌套了一个 HashMap 罢了，在 Java 中就像这样：

```java
Map<String, String> hash = new HashMap<>();	//Redis 默认存 String 类似于这样
Map<String, Map<String, String>> hash = new HashMap<>();	//Redis 存 Hash 类型的数据类似于这样
```

它比较适合存储类这样的数据，由于值本身又是一个 Map，因此可以在此 Map 中放入类的各种属性和值，以实现一个 Hash 数据类型存储一个类的数据。

- 添加一个 Hash 类型的数据：

    ```sql
    hset <key> [<字段> <值>]...
    ```

- 直接获取：

    ```sql
    hget <key> <字段>
    hgetall <key>	#获取所有的字段和值
    ```

- 判断某个字段是否存在：

    ```sql
    hexists <key> <字段>    #存在返回 1，不存在返回 0
    ```

- 删除 Hash 中的某个字段：

    ```sql
    hdel <key>
    ```

> <font color="green">提示</font>
> 
> 操作一个 Hash 时，实际上就是普通操作命令前面添加一个 `h`，这样就能以同样的方式去操作 Hash 里面存放的键值对了。

- 几个比较特殊的。

    - 查看 Hash 中一共存了多少个键值对：

        ```sql
        hlen <key>
        ```

    - 获取所有字段的值：

        ```sql
        hvals <key>
        ```

> <font color="brown">注意</font>
> 
> Hash 中只能存放字符串值，不允许出现嵌套的的情况。

### List

它就是一个列表，而列表中存放一系列的字符串，它支持随机访问，支持双端操作，就像使用 Java 中的 LinkedList 一样。

- 可以直接向一个已存在或是不存在的 List 中添加数据，如果不存在，会自动创建：

    ```sql   
    lpush <key> <element>...	#向列表头部添加元素    
    rpush <key> <element>...	#向列表尾部添加元素
    linsert <key> before/after <指定元素> <element>	#在指定元素前面/后面插入元素
    ```

- 获取元素：

    ```sql
    lindex <key> <下标> #根据下标获取元素
    lpop <key>  #获取并移除头部元素
    rpop <key>  #获取并移除尾部元素
    lrange <key> start stop #获取指定范围内的
    ```

- 下标可以使用负数来表示从后到前数的数字:

    ```sql
    lrange <key> 0 -1 #获取索引值为 0 至倒数第一个元素的所有元素。即获取列表中的全部元素
    lrange <key> 1 -2 #获取第二个元素至倒数第二个元素的所有元素
    ```

- 移动列表元素

    ```sql
    rpoplpush 当前列表 目标列表 #将当前列表的最后一个元素取出并放入目标列表的头部，并返回元素
    ```

- 它还支持阻塞操作，类似于生产者和消费者

  比如想要等待列表中有了数据后再进行 pop 操作：
  
    ```sql
    blpop <key>... timeout  #如果列表中没有元素，那么就等待，如果指定时间（秒）内被添加了数据，那么就执行 pop 操作，如果超时就作废。支持同时等待多个列表，只要其中一个列表有元素了，那么就能执行
    ```

### Set 和 SortedSet

#### Set

Set 集合其实就像 Java 中的 HashSet 一样（在 JavaSE 中，HashSet 本质上就是利用了一个 HashMap，但是 Value 都是固定对象，仅仅是 Key 不同）它不允许出现重复元素，不支持随机访问，但是能够利用 Hash 表提供极高的查找效率。

- 向 Set 中添加一个或多个值：

    ```sql
    sadd <key> <value>...
    ```

- 查看 Set 集合中有多少个值：

    ```sql
    scard <key>
    ```

- 判断集合中是否包含：

    ```sql
    sismember <key> <value>	#是否包含指定值
    smembers <key>	#列出所有值
    ```

- 集合之间的运算：

    ```sql
    sdiff <key1> <key2>	#集合之间的差集
    sinter <key1> <key2>	#集合之间的交集
    sunion <key1> <key2>	#求并集
    sdiffstore 目标 <key1> <key2>	#将集合之间的差集存到目标集合中
    sinterstore 目标 <key1> <key2>	#同上
    sunionstore 目标 <key1> <key2>	#同上
    ```

- 移动指定值到另一个集合中：

    ```sql
    smove <key> 目标 value 
    ```

- 移除操作：

    ```sql
    spop <key>	#随机移除一个元素
    srem <key> <value>...	#移除指定
    ```

#### SortedSet

当要求 Set 集合中的数据按照指定的顺序进行排列时就可以使用 SortedSet，它支持为每个值设定一个分数，分数的大小决定了值的位置，所以它是有序的。

- 添加一个带分数的值：

    ```sql
    zadd <key> [<value> <score>]...
    ```

- 类似 Set 的查询、移除和获取：

    ```sql
    zcard <key>	#查询有多少个值
    zrem <key> <value>...	#移除
    zrange <key> start stop	#获取区间内的所有
    ```

- 由于所有的值都有一个分数，也可以根据分数段来获取：

    ``` sql
    zrangebyscore <key> start stop [withscores] [limit]	#通过分数段查看
    zcount <key>  start stop	#统计分数段内的数量、
    zrank <key> <value>	#根据分数获取指定值的排名
    ```

***

## 持久化

Redis 数据库中的数据都是存放在内存中，虽然很高效，但是这样存在一个非常严重的问题，如果突然停电，那数据不就全部丢失了吗？它不像硬盘上的数据，断电依然能够保存。

这个时候就需要持久化，开发者需要将数据备份到硬盘上，防止断电或是机器故障导致的数据丢失。

~~持久化的实现方式有两种方案：~~

持久化的实现方式有三种方案：

- RDB：直接保存当前**已经存储的数据**，相当于复制内存中的数据到硬盘上，需要恢复数据时直接读取即可；
- AOF：保存存放数据的**所有过程**，需要恢复数据时，只需要将整个过程完整地重演一遍就能保证与之前数据库中的内容一致；
- 混合持久化（Redis 4.0 引入）：混合持久化并不是一种全新的持久化方式，而是对已有方式的优化。混合持久化只发生于 AOF 重写过程。使用了混合持久化，重写后的新 AOF 文件前半段是 RDB 格式的全量数据，后半段是 AOF 格式的增量数据。

### RDB

RDB 就是第一种解决方案，那么如何将数据保存到本地呢？

可以使用命令：

```sql
save	#注意这个命令是直接保存，会占用一定的时间，也可以单独开一个子进程后台执行保存
bgsave
```

执行后，会在服务端目录下生成一个 dump.rdb 文件，而这个文件中就保存了内存中存放的数据，当服务器重启后，会自动加载里面的内容到对应数据库中。保存后我们可以关闭服务器：

```sql
shutdown
```

重启后可以看到数据依然存在。

![RDB](../ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/Redis/RDB.PNG)

这种方式方便，但由于会完整复制所有的数据，如果数据库中的数据量比较大，那么复制一次可能就需要花费大量的时间，所以可以每隔一段时间自动进行保存。<br>再或者，如果基本上都是在进行读操作，而没有进行写操作，实际上只需要偶尔保存一次即可，因为数据几乎没有怎么变化，可能两次保存的都是一样的数据。

开发者可以在配置文件中设置自动保存，并设定在一段时间内写入多少数据时，执行一次保存操作：

```
save 300 10 # 300秒（5分钟）内有 10 个写入
save 60 10000 # 60秒（1分钟）内有 10000 个写入
```

> 配置的 save 使用的都是 bgsave 后台执行。

虽然 RDB 能够很好地解决数据持久化问题，但是它的缺点也很明显：每次都需要去完整地保存整个数据库中的数据，同时后台保存过程中也会产生额外的内存开销，最严重的是它并不是实时保存的，如果在自动保存触发之前服务器崩溃，那么依然会导致少量数据的丢失。

### AOF

AOF 会以日志的形式将开发者每次执行的命令都进行保存，服务器重启时会将所有命令依次执行，通过这种重演的方式将数据恢复，这样就能很好解决实时性存储问题。

![AOF](../ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/Redis/AOF.png)

配置日志保存策略，有三种策略：

* always：每次执行写操作都会保存一次
* everysec：每秒保存一次（默认配置），这样就算丢失数据也只会丢一秒以内的数据
* no：看系统心情保存

可以在配置文件中配置：

```sql
appendonly yes# 注意得改成 yes
appendfsync everysec	# 可配置为 appendfsync [always 或 everysec 或 no]
```

重启服务器后，可以看到服务器目录下多了一个 `appendonly.aof` 文件，存储的就是之前执行的命令。

 AOF 的缺点也很明显，每次服务器启动都需要进行过程重演，相比 RDB 更加耗费时间，并且随着操作变多，不断累计，可能到最后 `appendonly.aof` 文件会变得无比巨大。

Redis 有一个 AOF 重写机制进行优化。<br>比如执行了这样的语句：

```
lpush test 666
lpush test 777
lpush test 888
```

实际上用一条语句也可以实现：

```
lpush test 666 777 888
```

正是如此，只要开发者能够保证最终的重演结果和原有语句的结果一致，无论语句如何修改都可以，所以可以通过这种方式将多条语句进行压缩。

也可以输入命令来手动执行重写操作：

```sql
bgrewriteaof
```

或是在配置文件中配置自动重写：

```
auto-aof-rewrite-percentage 100	# 百分比计算
auto-aof-rewrite-min-size 64mb	# 当达到这个大小时，触发自动重写
```

### 混合持久化

### 三种持久化各自优缺点

* AOF：
  * 优点：存储速度快、消耗资源少、支持实时存储
  * 缺点：加载速度慢、数据体积大
* RDB：
  * 优点：加载速度快、数据体积小
  * 缺点：存储速度慢大量消耗资源、会发生数据丢失
* 混合持久化：
  * 优点：RDB 和 AOF 持久化的优点，开头为 RDB 的格式，使得 Redis 可以更快的启动，同时结合 AOF 的优点，有减低了大量数据丢失的风险。
  * 缺点：
    * AOF 文件中添加了 RDB 格式的内容，使得 AOF 文件的可读性变得很差；
    * 兼容性差，如果开启混合持久化，那么此混合持久化 AOF 文件，就不能用在 Redis 4.0 之前版本了。


***

## 事务和锁机制

和 MySQL 一样，在 Redis 中也有事务机制，当开发者需要保证多条命令一次性完整执行而中途不受到其他命令干扰时，就可以使用事务机制。

- 使用命令来直接开启事务：

```sql
multi
```

- 当输入完所有要执行的命令时，可以使用命令来立即执行事务：

```sql
exec
```

- 中途取消事务：

```sql
discard
```

实际上整个事务是创建了一个命令队列，它不像 MySQL 那种在事务中也能单独得到结果，而是提前将所有的命令装在队列中，但是并不会执行，而是等提交事务的时候再统一执行。

### 锁

实际上在 Redis 中也会出现多个命令同时竞争同一个数据的情况。<br>比如现在有两条命令同时执行，它们都要去修改 a 的值，那么这个时候就只能动用锁机制来保证同一时间只能有一个命令操作。

虽然 Redis 中也有锁机制，但是它是一种乐观锁，不同于 MySQL 中认识的锁是悲观锁。

* 悲观锁：时刻认为别人会来抢占资源，禁止一切外来访问，直到释放锁，具有强烈的排他性质。
* 乐观锁：并不认为会有人来抢占资源，所以会直接对数据进行操作，在操作时再去验证是否有其他人抢占资源。

Redis 中可以使用 watch 来监视一个目标，如果执行事务之前被监视目标发生了修改，则取消本次事务：

```sql
watch <key>...	#监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。
```

取消监视可以使用：

```sql
unwatch	#取消 watch 命令对所有 key 的监视。
```

## 使用 Java 与 Redis 交互

使用 Java 与 Redis 交互时需要使用到 Jedis 框架，它能够实现 Java 与 Redis 数据库的交互依赖：

```xml
<dependencies>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>4.0.0</version>
    </dependency>
    <!-- 需要添加 slf4j-jdk14 依赖，单独使用 jedis 时会爆红 -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-jdk14</artifactId>
        <version>1.7.36</version>
    </dependency>
</dependencies>
```

### 基本操作

连接Redis数据库

```java
public static void main(String[] args) {
    Jedis jedis = new Jedis("address", port);	//创建 Jedis 对象    
  	jedis.auth("password");	//若之前设置过 Redis 密码，则需要先输入密码，否则 ERROR: NOAUTH Authentication required.
  	jedis.close();	//使用之后关闭连接
}
```

通过 Jedis 对象就可以直接调用命令的同名方法来执行 Redis 命令了，比如：

```java
public static void main(String[] args) {
    //直接使用 try-with-resouse，编译后生成的 .class 文件中会自己生成 jedis.close()
    try(Jedis jedis = new Jedis("address", port)){
        jedis.set("test", "lbwnb");   //等同于 set test lbwnb 命令
        System.out.println(jedis.get("test"));  //等同于 get test 命令
    }
}
```

Hash 类型的数据：

```java
public static void main(String[] args) {
    try(Jedis jedis = new Jedis("address", port)){
        jedis.hset("hhh", "name", "sxc");   //等同于 hset hhh name sxc
        jedis.hset("hhh", "sex", "19");    //等同于 hset hhh age 19
        jedis.hgetAll("hhh").forEach((k, v) -> System.out.println(k+": "+v));
    }
}
```

列表操作：

```java
public static void main(String[] args) {
    try(Jedis jedis = new Jedis("address", port)){
        jedis.lpush("mylist", "111", "222", "333");  //等同于 lpush mylist 111 222 333 命令
        jedis.lrange("mylist", 0, -1)
                .forEach(System.out::println);    //等同于 lrange mylist 0 -1
    }
}
```

### Spring Boot 整合 Redis

![Spring Boot 整合 Redis 新建项目](../ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/Redis/Spring%20Boot%20%E6%95%B4%E5%90%88%20Redis%20%E6%96%B0%E5%BB%BA%E9%A1%B9%E7%9B%AE.png)

在 Spring Boot 项目中整合Redis操作框架，只需要一个 starter 即可，但是它底层没有用 Jedis，而是 Lettuce：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

starter 提供的默认配置会去连接本地的 Redis 服务器，并使用 0 号数据库，当然也可以手动配置 `application.yml`：

```yaml
spring:
  data:
    redis:
      host: IP Address，本机使用会 127.0.0.1 或 localhost
      port: 默认端口为 6379
      password: Redis 服务器密码，若在 Redis 中未设置密码，则无需配置此项
      database: 默认数据库为 0，Redis 默认共有 16 个数据库，0-15
```

starter 已经提供了两个默认的模板类：

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    )
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        return new StringRedisTemplate(redisConnectionFactory);
    }
}
```

可以直接注入`StringRedisTemplate`来使用使用这两个模板类：

```java
@SpringBootTest
class SpringBootTestApplicationTests {
    @Autowired
    StringRedisTemplate template;
    
    @Test
    void contextLoads() {
        ValueOperations<String, String> operations = template.opsForValue();
        operations.set("c", "xxxxx");   //设置值
        System.out.println(operations.get("c"));   //获取值
      	
        template.delete("c");    //删除键
        System.out.println(template.hasKey("c"));   //判断是否包含键
    }
}
```

实际上所有的值的操作都被封装到了 `ValueOperations` 对象中，而普通的键操作直接通过模板对象就可以使用了，大致使用方式其实和 Jedis 一致。

#### 事务操作

由于 Spring 没有专门的 Redis 事务管理器，所以只能借用 JDBC 提供的。

导入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

在  `application.yml` 添加 JDBC 配置：

```yml
spring:
  datasource:
    url: jdbc:mysql://address:port/数据库名
    username: root
    password: 962464
    driver-class-name: com.mysql.cj.jdbc.Driver
```

Student 实体

```java
import lombok.AllArgsConstructor;
import lombok.Data;

import java.io.Serializable;

@Data
@AllArgsConstructor
public class Student implements Serializable {	//使用 jdk 自带的序列化
    String name;
    int age;
}
```

编写事务 service 代码：

```java
import com.example.jedisspringboot.entiy.Student;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.PostConstruct;
import javax.annotation.Resource;

/**
 * @ClassName RedisService
 * @Description
 * @Author DoubleZHE
 * @Create 2023/3/24 15:40
 */

@Service
public class RedisService {
    @Resource
    RedisTemplate<Object, Object> template;

    @PostConstruct
    public void init(){
        template.setEnableTransactionSupport(true);   //需要开启事务
//        template.setValueSerializer((new Jackson2JsonRedisSerializer<>(Object.class)));	//jackson-databind 依赖的序列化方式
    }

    @Transactional    //需要添加此注解
    public void test(){
        template.multi();
        template.opsForValue().set("d", "xxxxxx");
        template.exec();
    }
}
```

测试：

```java
@SpringBootTest
class JedisSpringbootApplicationTests {
    @Resource
    RedisService redisService;

    @Test
    void contextLoads() {
        redisService.test();
    }
}
```

为 RedisTemplate 对象配置一个 Serializer 来实现对象的 JSON 存储：

```java
@Test
void contextLoad2() {
    //注意 Student 需要序列化对象才能存入Redis
    template.opsForValue().set("student", new Student());
    System.out.println(template.opsForValue().get("student"));
}
```

***

## 使用 Redis 做缓存

使用 Redis 来实现一些框架的缓存和其他存储。

### MyBatis 二级缓存

MyBatis 的二级缓存机制，它是Mapper级别的缓存，能够作用与所有会话。<br>由于MyBatis的默认二级缓存只能是单机的，如果存在多台服务器访问同一个数据库，实际上二级缓存只会在各自的服务器上生效，但是理想状态下是多台服务器都能使用同一个二级缓存，这样就不会造成过多的资源浪费。

![MyBatis 二级缓存](../ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/Redis/MyBatis%20%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98.png)

开发者可以将 Redis 作为 MyBatis 的二级缓存，这样就能实现多台服务器使用同一个二级缓存，因为它们只需要连接同一个 Redis 服务器即可，所有的缓存数据全部存储在 Redis 服务器上。

为此需要手动实现 MyBatis 提供的 Cache 接口的缓存类：

```java
//实现 MyBatis 的 Cache 接口
public class RedisMyBatisCache implements Cache {
    private final String id;
    private static RedisTemplate<Object, Object> template;

   	//注意构造方法必须带一个 String 类型的参数接收 id
    public RedisMyBatisCache(String id){
        this.id = id;
    }

  	//初始化时通过配置类将 RedisTemplate 传参过来
    public static void setTemplate(RedisTemplate<Object, Object> template) {
        RedisMyBatisCache.template = template;
    }

    @Override
    public String getId() {
        return id;
    }

    @Override
    public void putObject(Object o, Object o1) {
        template.opsForValue().set(o, o1, 60, TimeUnit.SECONDS);	//直接向 Redis 数据库中丢数据即可，o 就是 Key，o1 就是 Value，60 秒为过期时间
    }

    @Override
    public Object getObject(Object o) {
        return template.opsForValue().get(o);	//根据 Key 直接从 Redis 数据库中获取值即可
    }

    @Override
    public Object removeObject(Object o) {
        return template.delete(o);	//根据 Key 删除
    }

    @Override
    public void clear() {
		template.execute((RedisCallback<Void>) connection -> {	//由于 template 中没封装清除操作，只能通过 connection 来执行
            connection.flushDb();	//通过 connection 对象执行清空操作
            return null;
        });
    }

    @Override
    public int getSize() {
        return template.execute(RedisServerCommands::dbSize).intValue();	//使用 connection 对象来获取当前的 Key 数量
    }
}
```

配置类：

```java
@Configuration
public class MainConfiguration {
    @Resource
    RedisTemplate<Object, Object> template;

    @PostConstruct
    public void init(){
        RedisMyBatisCache.setTemplate(template);	//将 RedisTemplate 给到 RedisMyBatisCache
    }
}
```

Mapper 上启用此缓存：

```java
//只需要修改缓存实现类 implementation 为 RedisMyBatisCache 即可
@CacheNamespace(implementation = RedisMyBatisCache.class)
@Mapper
public interface MainMapper {

    @Select("select name from student where sid = 1")
    String getSid();
}
```

提供一个测试用例来查看当前的二级缓存是否生效：

```java
@SpringBootTest
class SpringBootTestApplicationTests {
    @Resource
    MainMapper mapper;

    @Test
    void contextLoads() {
        System.out.println(mapper.getSid());
        System.out.println(mapper.getSid());
        System.out.println(mapper.getSid());
    }

}
```

手动使用客户端查看 Redis 数据库，可以看到已经有一条 MyBatis 生成的缓存数据了。

### Token持久化存储

我们之前使用SpringSecurity时，remember-me的Token是支持持久化存储的，而我们当时是存储在数据库中，那么Token信息能否存储在缓存中呢，当然也是可以的，我们可以手动实现一个：

```java
//实现PersistentTokenRepository接口
@Component
public class RedisTokenRepository implements PersistentTokenRepository {
  	//Key名称前缀，用于区分
    private final static String REMEMBER_ME_KEY = "spring:security:rememberMe:";
    @Resource
    RedisTemplate<Object, Object> template;

    @Override
    public void createNewToken(PersistentRememberMeToken token) {
      	//这里要放两个，一个存seriesId->Token，一个存username->seriesId，因为删除时是通过username删除
        template.opsForValue().set(REMEMBER_ME_KEY+"username:"+token.getUsername(), token.getSeries());
        template.expire(REMEMBER_ME_KEY+"username:"+token.getUsername(), 1, TimeUnit.DAYS);
        this.setToken(token);
    }

  	//先获取，然后修改创建一个新的，再放入
    @Override
    public void updateToken(String series, String tokenValue, Date lastUsed) {
        PersistentRememberMeToken token = this.getToken(series);
        if(token != null)
           this.setToken(new PersistentRememberMeToken(token.getUsername(), series, tokenValue, lastUsed));
    }

    @Override
    public PersistentRememberMeToken getTokenForSeries(String seriesId) {
        return this.getToken(seriesId);
    }

  	//通过username找seriesId直接删除这两个
    @Override
    public void removeUserTokens(String username) {
        String series = (String) template.opsForValue().get(REMEMBER_ME_KEY+"username:"+username);
        template.delete(REMEMBER_ME_KEY+series);
        template.delete(REMEMBER_ME_KEY+"username:"+username);
    }

  
  	//由于PersistentRememberMeToken没实现序列化接口，这里只能用Hash来存储了，所以单独编写一个set和get操作
    private PersistentRememberMeToken getToken(String series){
        Map<Object, Object> map = template.opsForHash().entries(REMEMBER_ME_KEY+series);
        if(map.isEmpty()) return null;
        return new PersistentRememberMeToken(
                (String) map.get("username"),
                (String) map.get("series"),
                (String) map.get("tokenValue"),
                new Date(Long.parseLong((String) map.get("date"))));
    }

    private void setToken(PersistentRememberMeToken token){
        Map<String, String> map = new HashMap<>();
        map.put("username", token.getUsername());
        map.put("series", token.getSeries());
        map.put("tokenValue", token.getTokenValue());
        map.put("date", ""+token.getDate().getTime());
        template.opsForHash().putAll(REMEMBER_ME_KEY+token.getSeries(), map);
        template.expire(REMEMBER_ME_KEY+token.getSeries(), 1, TimeUnit.DAYS);
    }
}
```

接着把验证Service实现了：

```java
@Service
public class AuthService implements UserDetailsService {

    @Resource
    UserMapper mapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Account account = mapper.getAccountByUsername(username);
        if(account == null) throw new UsernameNotFoundException("");
        return User
                .withUsername(username)
                .password(account.getPassword())
                .roles(account.getRole())
                .build();
    }
}
```

Mapper也安排上：

```java
@Data
public class Account implements Serializable {
    int id;
    String username;
    String password;
    String role;
}
```

```java
@CacheNamespace(implementation = MyBatisRedisCache.class)
@Mapper
public interface UserMapper {

    @Select("select * from users where username = #{username}")
    Account getAccountByUsername(String username);
}
```

最后配置文件配一波：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
            .authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .and()
            .rememberMe()
            .tokenRepository(repository);
}

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
            .userDetailsService(service)
            .passwordEncoder(new BCryptPasswordEncoder());
}
```

OK，启动服务器验证一下吧。

***

## 三大缓存问题

**注意：**这部分内容作为选学内容。

虽然我们可以利用缓存来大幅度提升我们程序的数据获取效率，但是使用缓存也存在着一些潜在的问题。

### 缓存穿透

![缓存穿透](../ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/Redis/%E7%BC%93%E5%AD%98%E7%A9%BF%E9%80%8F.png)

当我们去查询一个一定不存在的数据，比如MyBatis在缓存是未命中的情况下需要从数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，造成缓存穿透。

这显然是很浪费资源的，我们希望的是，如果这个数据不存在，为什么缓存这一层不直接返回空呢，这时就不必再去查数据库了，但是也有一个问题，缓存不去查数据库怎么知道数据库里面到底有没有这个数据呢？

这时我们就可以使用布隆过滤器来进行判断。什么是布隆过滤器？（当然不是打辅助的那个布隆，只不过也挺像，辅助布隆也是挡子弹的）

![点击查看图片来源](https://s2.loli.net/2023/03/06/vWaLugBJMT3zIDC.jpg)

使用布隆过滤器，能够告诉你某样东西一定不存在或是某样东西可能存在。

布隆过滤器本质是一个存放二进制位的bit数组，如果我们要添加一个值到布隆过滤器中，我们需要使用N个不同的哈希函数来生成N个哈希值，并对每个生成的哈希值指向的bit位置1，如上图所示，一共添加了三个值abc。

接着我们给一个d，那么这时就可以进行判断，如果说d计算的N个哈希值的位置上都是1，那么就说明d可能存在；这时候又来了个e，计算后我们发现有一个位置上的值是0，这时就可以直接断定e一定不存在。

### 缓存击穿

![缓存击穿](../ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/Redis/%E7%BC%93%E5%AD%98%E5%87%BB%E7%A9%BF.png)

某个 Key 属于热点数据，访问非常频繁，同一时间很多人都在访问，在这个Key失效的瞬间，大量的请求到来，这时发现缓存中没有数据，就全都直接请求数据库，相当于击穿了缓存屏障，直接攻击整个系统核心。

这种情况下，最好的解决办法就是不让Key那么快过期，如果一个Key处于高频访问，那么可以适当地延长过期时间。

### 缓存雪崩

![缓存雪崩](../ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/Redis/%E7%BC%93%E5%AD%98%E9%9B%AA%E5%B4%A9.png)

当你的Redis服务器炸了或是大量的Key在同一时间过期，这时相当于缓存直接GG了，那么如果这时又有很多的请求来访问不同的数据，同一时间内缓存服务器就得向数据库大量发起请求来重新建立缓存，很容易把数据库也搞GG。

解决这种问题最好的办法就是设置高可用，也就是搭建Redis集群，当然也可以采取一些服务熔断降级机制，这些内容我们会在SpringCloud阶段再进行探讨。