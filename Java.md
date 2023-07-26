## Java 基础

1. JVM、JRE、JDK之间的关系

   * **JVM** ：**Java 虚拟机**，Java程序需要运行在JVM上，以此实现跨平台
   * **JRE** ：**Java运行环境**，不能创建Java程序，包括 JVM 和 **核心类库**等。
   * **JDK** ：**Java开发工具包**，用于开发Java程序，包括 JRE 和 **开发工具**，比如 **java.exe** (运行java程序)、**javac.exe**（编译工具，生成class文件）、**javaw.exe**(运行GUI程序)

2. **跨平台性**

   * 指 Java 程序一次编译后，可在多个系统平台上运行，原理是在不同系统平台上安装对应的 JVM，那该系统就能运行 Java 程序

3. **面向对象**和**面向过程**的区别

   1. **面向过程**：**性能比面向对象高**，因为类调用时需要实例化，开销比较大。缺点是：没有面向对象易维护、易复用、易扩展
   2. **面向对象**：**易维护、易复用、易扩展**，由于封装、继承、多态的特性，可以设计出低耦合的系统，系统更加灵活易维护
   3. **面向过程是具体化、流程化的，解决一个问题，需要一步一步的分析和实现**
   4. **面向过程是模型化的**，只需**抽象出一个类**，这是一个封闭的盒子，**里面有数据和解决问题的方法**，需要什么功能直接调用就行，不必一步一步实现，具体里面功能是如何实现的，不需要知道。

4. 面向对象**三大特性**

   1. **封装**
      * 把一个对象的属性私有化，提供一些可以被外界访问的属性和方法。
      * 隐藏对象的属性和实现细节，仅提供公共访问方法，将变化隔离，便于使用，提高安全性
   2. **继承**
      * 在已存在的类的定义的基础上建立新的类，新类可以增加新的属性和方法，也可以使用父类的功能，但不能选择性地继承父类
      * 方便复用以前的代码
   3. **多态**
      * **一个引用变量到底指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在程序运行期间才能决定**
      * 两种形式实现多态：继承（多个子类对同一个方法重写）、接口（实现接口并覆盖接口中同一方法）
      * 两种实现：
        * **方法重载**：编译时多态。在一个类里面，方法名相同，形参列表不同
        * **方法重写**：运行时多态。发生在父子类之间，**方法签名（方法名+形参列表）**相同，实现不同
      * 实现多态需要做两件事：
        1. **方法重写**：子类继承并重写父类的方法
        2. **对象造型**：用父类型引用子类型对象，这样同样的引用调用同样的方法就会根据子类对象的不同而表现不同的行为

5. 抽象类和接口的区别

   1. 接口
      * 接口中所有方法自动是 public
      * 接口中可以定义常量（因为接口中的字段默认是 public static final），没有实例字段
      * Jdk 8 以后，可以有实现方法
      * Jdk 9 中，接口中的可以有私有方法，这些方法只能用于接口中其他方法的辅助方法
      * **默认方法**：**为接口方法提供一个默认实现**，用 **default** 修饰，默认方法可以调用其他方法。主要是为了“**接口演化**”，保证源代码兼容，**默认方法可以在为以前的接口增加方法时使用，这样以前实现了这个接口的类不必去修改**，因为新增的方法有默认方法，自动有默认实现。
      * **如果一个接口中一个方法定义为默认方法，然后又在超类或另一个接口中定义了同样的方法**
        * **超类优先**：如果超类提供了具体方法，方法签名相同的默认方法会被忽略
        * **接口冲突**：如果两个接口的默认方法的方法签名相同，必须手动解决冲突
        * 如果一个类扩展了一个超类和一个接口，并从超类和接口中继承了相同的方法，**遵循，类优先，只会考虑类方法**
   2. 抽象类
      * 抽象类中可以有具体实现的方法
      * 如果子类不是抽象类，子类需要实现抽象父类中所有的抽象方法
      * 抽象类中可以有构造器
      * 子类只能继承一个父类，但是接口可以多继承

6. 父类的静态方法能否被子类重写

   * 不能，重写只适用于实例方法，如果子类中含有和父类相同方法签名的静态方法，一般称之为隐藏

7. 静态变量和实例变量的区别

   1. 静态变量：存储于方法区，属于类所有
   2. 实例变量：存储于堆中，其引用存在于当前线程栈

8. **static**

   * 第一种情况，有时我们需要一小块共享空间来保存某个特定的字段，而并不关心创建多少个对象，甚至有没有创建对象。
   * 第二种情况，你需要使用一个类的某个方法，而该方法和具体的对象无关；换句话说，你希望即便没有生成任何该类的对象，依然可以调用其方法
   * **不能在没有具体对象的情况下，使用static方法直接调用非static成员或方法**（因为非 static 成员和方法必须基于特定的对象才能运行）

9. 访问修饰符

   * **public**: 表示定义的内容可以被所有人访问
   * **protect**: 对同一个包内的类和所有子类可见
   * **private**: 在同一个类内可见
   * **缺省**：同一个包内可见

10. 基本数据类型

    1. byte（1字节）、short（2字节）、int（4字节）、long（8字节）、float（4字节）、double（8字节）、char（1字节）、boolean
    2. 都是**有符号的**

11. **包装类**

    * 包装类是一组**用于将基本数据类型包装成对象的类**。这些包装类**使得基本数据类型能够以对象的形式在Java程序中操作**
    * 例如，当需要在**集合类中存储基本数据类型时**，就可以使用包装类作为集合的元素类型。
    * 提供了自动装箱和自动拆箱特性，使得基本数据类型和包装类之间的转换更加便捷

    * 提供了许多方法来操作和比较这些对象
      * 类型转换：**intValue**，将Integer转换成int

      * 比较操作：equals、compareTo

      * 字符串转换：toString将Integer转换成表示整数的字符串、**valueOf**

      * 常量和静态方法：**Integer.MAX_VALUE**和Integer.MIN_VALUE，**Integer.parseInt** 将字符串转换为整数

    * 自动装箱和自动拆箱的原理
      * 原理是**通过Java编译器在编译时期进行的操作，编译器会根据上下文情况判断需要进行自动装箱还是自动拆箱，并自动插入相关的代码。**

      * 自动装箱：编译器自动插入 **Integer obj = Integer.valueOf(num);**

      * 自动拆箱；自动插入：**int num = obj.intValue();**

12. **Integer 享元模式**

    * 对于值范围 在 -128~127 的 Integer 类型来说，它生成的对象是相同的：**出于效率问题，Integer 会通过享元模式来缓存范围在 -128~127 内的对象，因此多次调用 Integer.valueOf(127) 生成的其实是同一个对象**。另外，通**过 new Integer() 生成的对象都是新创建的，无论其值处于什么范围。** **Java 9 及更新版本中已经弃用 new Integer() ，因为它的效率远远低于 Integer.valueOf()。** 

13. final

    * 被 final 修饰的类不能被继承
    * 被 final 修饰的方法不能被重写
    * 被 final 修饰的变量不能被改变，如果是引用，引用指向的内容可以被修改
    * final 的用处，比如 **定义程序中的常量**，一般定义一个常量类，放在 contants 包下，常量类中的常量被 **public static final 修饰**，**变量名全大写**，用 “_” 分隔单词。因为开发规范要求，**代码中尽量不要出现魔法值**（如 if i > 10，这个 10 就是魔法值，他人阅读代码时不明白 10 的意思是什么）

14. this 和 super

    1. this

       * this 是自身的一个对象，代表对象本身，可以理解为：指向对象本身的一个指针

       * 用法：

         * 普通的直接引用

         * 形参和成员名字重名，用 this 区分

           ~~~java
           public Person(String name, int age) {
               this.name = name;
               this.age = age;
           }
           ~~~

         * 引用本类的构造器（需放在构造方法的第一行，且只能调用一个）

           ~~~java
           class Person {
               public Person(String name) {
                   this.name = name;
               }
               public Person(String name, int age) {
                   this(name);
                   this.age = age;
               }
           }
           ~~~

    2. super

       * 指向自己父类对象的一个指针，而这个父类是离自己最近的一个父类
       * 用法：
         * 普通的直接引用
         * 子类中的成员变量或方法和父类中的成员变量和方法同名时，用于区分
         * 调用父类构造器（需放在构造方法的第一行）

    3. this 和 super 不能同时出现在同一个构造器里，且均不能在 statis 环境中使用

15. **== 和 equals() 的区别**

    * == ：对于基本数据类型比较的值，**对于引用类型，比较的是对象的内存地址**，即，判断两个对象是不是同一个对象
    * equals()：判断两个对象是否相等。**如果没有重写 equals() 方法，equals() 等价于 ==** 。**重写后**，一般重写 equals() 来判断两个**对象的内容**是否相等。

16. 为什么重写 equals() 时必须重写 hashCode() 

    * 因为规定，如果两个对象相等（equals() 方法返回 true），则 hashCode 一定也相同
    * 而两个对象的 hashCode 相同，两个对象也不一定相等
    * 因此，equals() 方法被重写，hashCode 方法也必须重写
    * hashCode() 默认是对堆上的对象产生一个独特值，如果没重写 hashCode()，则两个对象无论如何都不会相等，即使这两个对象指向相同的数据

## String

1. String 表示字符串类型，属于 **引用数据类型**，不属于基本数据类型

2. 双引号括起来的字符串，是 “**不可变**” 的，直接存储在 **方法区** 的 **字符串常量池** 中（**后面将字符串常量池移到了堆中**）。原因是字符串在实际的开发中使用太频繁。为了执行效率，所以把字符串放到了方法区的字符串常量池中

   ~~~java
   public class StringTest02 {
       public static void main(String[] args) {
           String s1 = "hello";
           // "hello"是存储在方法区的字符串常量池当中
           // 所以这个"hello"不会新建。（因为这个对象已经存在了！）
           String s2 = "hello";
           
           // == 双等号比较的是变量中保存的内存地址
           System.out.println(s1 == s2); // true
   
           // 第一句创建了两个对象，一个是字符串常量池中的 “xyz”
           // 另一个是堆上的实例对象
           String x = new String("xyz");
           String y = new String("xyz");
           
           // == 双等号比较的是变量中保存的内存地址
           System.out.println(x == y); //false
       }
   }
   ~~~

   ![image-20230228155010772](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230228155010772.png)

3. 注意：字符串对象之间的比较不能使用“== ”,"=="不保险。应该调用[String类](https://so.csdn.net/so/search?q=String类&spm=1001.2101.3001.7020)的**equals**方法。

   ~~~java
   String k = new String("testString");
   //String k = null;
   // "testString"这个字符串可以后面加"."呢？
   // 因为"testString"是一个String字符串对象。只要是对象都能调用方法。
   System.out.println("testString".equals(k)); // 建议使用这种方式，因为这个可以避免空指针异常。
   System.out.println(k.equals("testString")); // 存在空指针异常的风险。不建议这样写。
   ~~~

4. String 之所以是 不可变，是因为 String 是通过一个 char[] 数组存储字符，并且被 final 修饰

5. String 本身也被 final 修饰，不能被继承

6. **“ + ” 连接符 对字符串提供特殊支持**

   ~~~java
   String s1 = "abc";
   String s2 = new String("java");
   String s3 = s1 + s2;
   
   // s3 的实现原理是，先 创建一个 StringBuilder() 对象，并调用append()方法将数据拼接，最后调用toString() 方法返回拼接好的字符串
   ~~~

7. 字符串字面量拼接编译期优化：

   ![image-20230301111627563](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230301111627563.png)

8. String 类的常用方法

   * indexOf(): 返回指定字符的索引
   * charAt(): 返回指定索引处的字符
   * replace(): 字符串替换
   * trim(): 去除字符串两端的空白
   * split(): 分割字符串，返回一个分割后的字符串数组
   * getBytes(): 返回字符串的byte类型数组
   * length(): 返回字符串的长度
   * toLowerCase(): 将字符串转成小写字母
   * toUpperCase(): 将字符串转成大写字母
   * subString(): 截取字符串
   * equals(): 字符串比较

9. **StringBuilder、StringBuffer 的区别**

   1. StringBuilder、StringBuffer 都继承自 AbstractStringBuilder 类，AbstractStringBuilder类使用 char 类型的数组保存字符串，有一个 count 属性用来保存当前保存的字符串个数，有一系列的字符串操作方法。

   2. StringBuilder、StringBuffer 的字符串操作方法都是直接调用的父类的方法，StringBuffer 的方法都被 synchronized 修饰，适用于多线程场景。

   3. StringBuffer 有一个 toStringCache 属性，被 transient 修饰。toStringCache 字段是为了作**缓存**的，缓存最后一次 toString 的内容，当被修改时这个 cache 清空，也就是说，如果没被修改，那么这个 toStringCache 就是上一次toString 的结果：

      ~~~java
      // StringBuffer 的 toString() 方法
      @Override
      public synchronized String toString() {
          if (toStringCache == null) {
              toStringCache = Arrays.copyOfRange(value, 0, count);
          }
          return new String(toStringCache, true);
      }
      ~~~

      StringBuffer 中的 toStringCache 是字符数组 value 复制的一个副本，每当 value 发生改变时，toStringCache 都会被置为空。这就保证了每次只要 StringBuffer 对象发生改变，再调用  toString() 方法就必然产生一个新的 toStringCache 数组，从而保证了引用了旧的 toStringCache 的字符串对象不会发生改变。即使多个线程同时访问 StringBuffer 对象，某一时刻也只有一个线程能够进入修改 toStringCache 和 value 的代码块，这通过修饰 StringBuffer 方法的 synchronized 来保证。

   4. 有 append()、insert()、replace()、deleteCharAt()、indexOf()、reverse()、toString()等方法，可以实现字符串的增删改查等基本功能

## 集合

### ArrayList

* 底层：transient Object elementData，elementData 是存放元素的容器，**ArrayList 基于数组实现**

* ArrayList支持默认大小构造，和空构造，当空构造的时候存放数据的Object[] elementData是一个空数组 []。

* 当初始化的list是一个空ArrayList的时候，会直接扩容到DEFAULT_CAPACITY，该值大小是一个**默认值10**。而当添加进ArrayList中的元素超过了数组能存放的最大值就会进行扩容，扩容代码：

  ~~~java
  // 右移一位，相当于扩容为原先的 1.5 倍
  int newCapacity = oldCapacity + (oldCapacity >> 1);
  ~~~

* 数组copy：Java 无法自己分配空间，通过 arraycopy 这个 **native 方法实现数组的复制**

* 如何将 ArrayList 变成线程安全的

  * 调用 Collections.synchronizedList(List list) 

### LinkedList

* 底层：LinkedList由一个头节点和一个尾节点组成，分别指向链表的头部和尾部，是**双向链表**
* **LinkedList 不仅实现了 List 接口，还实现了 Deque 双端队列接口**，而ArrayList 只实现了 List 接口
* 查询方法：先判断 index 更靠近头部还是尾部，靠近哪段从哪段遍历获取值
* 修改：采用尾插法

### HashMap

* HashMap 在 jdk1.7 中使用 **数组+链表** 实现，jdk1.8 使用 **数组+链表+红黑树** 实现

* **key 的 哈希算法**：h ^ (h >>> 16)

  * h >>> 16 相当于将高区的 16 位移动到低区的 16 位，再与原 hashCode 做异或运算，可以**将高低位二进制特征混合起来**，减少 hash 碰撞的可能性
  * **使用异或运算的原因**：**异或运算能更好的保留各部分的特征**，如果**采用 & 运算计算出来的值会向 0 靠拢，采用 | 运算计算出来的值会向 1 靠拢**

* 根据 key 的哈希值**计算 索引位置**

  * **index = (length - 1) & hashValue**
  * 该公式保证了获取的 index 一定在数组范围内

* HashMap 扩容：**2倍扩容**

  * HashMap根据两个属性进行扩容：capacity（默认容量为16，1<<<4），loadFactor（负载因子，默认为0.75）

    在默认情况下，16 * 0.75 = 12，当添加第13个元素时，就会进行扩容，**扩容之后的容量是原来容量的2倍**，并重新计算每个元素的index

  * **为什么是 2倍扩容**：

    * 因为 底层数组的索引位置计算公式是 ：index = （length - 1）& hashValue，2倍扩容使得数组长度始终保持 2 的次幂，如最初的默认长度 16，那么 length - 1 的二进制就是 01111，2 倍扩容后是 32，length -1 的二进制是 011111。这样会保证 length - 1低位全是 1，扩容后也是在左边多了一位 1，这样通过 length - 1）& hashValue 计算的时候，只要 hashValue 的最左边一位差异位是 0，就能**保证得到的新的数组索引和老数组索引一致(大大减少了之前已经散列良好的老数组的数据位置重新调换)**
    * 还有，数组长度保持2的次幂，length-1的低位都为1，会使得**获得的数组索引index更加均匀**
    * 索引计算采用 & 运算，hashValue 的高位是不会对结果产生影响的，我们只关注低位，如果低位全是 1（length - 1 保证了低位全是 1），那么对于 hashValue 的低位部分来说，任何一位 的变化都会对结果产生影响，**也就是说，要得到index=21这个存储位置，h的低位只有这一种组合。这也是数组长度设计为必须为2的次幂的原因。**
    * 如果不是2的次幂，也就是低位不是全为 1 此时，要使得 index = 21，hashValue 的低位部分不再具有唯一性了，哈希冲突的几率会变的更大，同时，index 对应的这个 bit 位无论如何不会等于 1 了，而对应的那些数组位置也就被白白浪费了

* **插入值**

  1. 计算 key 的 哈希值

  2. 判断 HashMap 是否为空，为空则创建 Node<K,V> [] table

  3. 根据 key 的哈希值计算出要插入的数组索引 i

  4. 如果 table[ i ] 等于 null，直接插入

  5. 否则，如果 key 已经存在，直接更新 value

  6. 如果 key 不存在，判断 table[ i ] 节点的类型是红黑树节点还是链表节点，将 key-value 插入。

     * 链表插入时，jdk 1.7 采用头插法，jdk 1.8 采用尾插法。
     * 头插法在多线程情况下可能会出现 **环形链** 的情况

     * **判断链表长度是否大于 8 ，转换成红黑树**

  7. 判断是否要扩容，进行扩容

* **为什么要使用红黑树**

  红黑树：Red-Black Tree，简称R-B Tree，是一种**不严格的平衡二叉查找树**，红黑树中的节点，有颜色标记，并满足以下要求：

  * 根节点是黑色的
  * 每个叶子节点都是黑色的空节点，也就是说，叶子节点不存储数据
  * 任何相邻的节点都不能同时为红色，红色节点是被黑色节点隔开的
  * 每个节点，从该节点到其可达叶子节点的所有路径，都包含相同数目的黑色节点

  红黑树的优势：

  * 红黑树是 **“近似平衡”** 的
  * 红黑树相比 avl树（自平衡二叉树，高度平衡树），红黑树不像 avl树 一样追求绝对的平衡，它允许局部很少的不完全平衡，这样对效率影响不大，但省去了很多没必要的调平衡操作，avl 树调平衡有时代价很大
  * 红黑树的高度只比 avl树的高度（logn）大一倍，但性能上去好很多
  * avl 树是高度平衡的二叉树，所以查找速度非常快，但是，为了维持高度平衡，在插入删除时，调整代价也很大。所以，对于有频繁插入删除操作的数据集合，使用 avl 树代价有点高

* **HashMap 为什么在链表长度大于8时转换成红黑树**

  * 官方做过很多测试，**发现链表长度在大于8以后再出现hash 碰撞的可能性为 0.00000006（几乎为0）**，**也就是说哈希碰撞同一个位置8次的概率几乎为0**，虽然转换为红黑树后，查询效率比链表高，但是转换红黑树的过程是耗时的，而且在扩容时还要对红黑树重新左旋或右旋保持平衡。***\*阈值设置为8就是为了尽量减少hashmap中出现红黑树\****（hashmap中链表才是常态） 

* **HashMap 线程不安全**，多线程情况下建议使用 ConcurrentHashMap

* HashMap 添加和查询的顺序不一致，要解决这个问题，可以使用 LinkedHashMap



### HashTable

* HashTable 也**实现了 Map 接口**，但与HashMap 不同的是，HashTable **继承的是 Dictionary 类**，HashMap 继承的是 AbstractHashMap 类。HashTable 继承了Dictionary 的一个 **elements 方法，返回的是所有值的枚举。**
* HashTable **不允许 key 为 Null** ，会抛出空指针异常
* HashTable **计算key 的hash 值是用的Object 的 hashCode 方法**，在**计算索引**的时候，用的公式是 **（hash & 0x7fffffff）% tab.length** ，0x7fffffff 是 第一位是0，其余31位都是1 的二进制数，然后和 tab 的长度取模获取索引位置，**取模法比HashMap 的位运算法效率低**
* HashTable 的 **扩容机制是 2*N + 1**，使得数组的长度**尽量为奇数或素数，这样可以减少 Hash 碰撞。**
* HashTable 的方法都有被 Synchronized 修饰。HashTable 在扩容时，**采用的头插法迁移数据**，使用头插法的效率高，虽然 rehash() 方法没有被 Synchronized 修饰，但是所有调用 rehash 方法的方法均被 Synchronized修饰



### ConcurrentHashMap

* JDK1.7 中，采用 分段锁 **Segment + HashEntry**，HashMap 是只有一个数组来存储，而ConcurrentHashMap 是把 这样的数组拆分成多个，每一个分组配一把锁，当一个线程占用锁访问其中一段数据，其他段的数据也能被其他线程访问到。
* Segment 继承了 ReentrantLock（可重入锁），默认有16 个Segment ，也就是并发度默认是16 ；内部有个 HashEntry[] 数组，HashEntry 类相当于 HashMap 中的 Node 节点，不过**HashEntry 中的 value 和 next 都被 volatile 修饰，保证多线程下数据获取时的可见性。**
* 在进行 put 时，会先hash定位到 是哪个Segment，在定位元素所在的链表头部**（二次定位）**。首先会尝试获取锁，**如果获取失败，就通过 scanAndLockForPut() 方法自旋获取锁，如果重试次数达到 Max_Scan_Retries 改为阻塞锁获取，保证能获取成功**。
* get 时，也是二次定位，由于HashEntry 的共享变量被 volatile 修饰，所以每次获取到的都是最新值。**get 不需要加锁，因为 volatile 保证了可见性**
* JDK1.8 改用了 和jdk1.8中 HashMap相同的 **数组+链表+红黑树**的结构，采用 **CAS + synchronized 锁住链表的头结点**，实现了更小的锁粒度。JDK1.8 **之所以采用 synchronized 替换 可重入锁** ReentrantLock ，是因为**JDK1.6 对 synchronized 锁的实现进行了大量优化，并且 synchronized 有多种锁状态，会从无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁一步步转换**。Node 节点的 value 和 next 都用了 volatile 修饰，保证可见性
* put 时，根据 key 计算hash值，然后判断是否需要初始化 table，然后定位到 Node，拿到首节点，判断首节点：
  1. 如果为 null，通过 CAS 尝试添加
  2. 如果 **Node.hash = MOVED = -1**，说明其他线程在扩容，参与一起扩容
  3. 如果都不满足，synchronized 锁住Node 节点，判断是链表还是红黑树，遍历插入，链表大于8时扩容或者转换成红黑树
* get 无须加锁，因为volatile保证了可见性
* ConcurrentHashMap **不支持 key 或 value 为 null**。value不能为null 的原因是，ConcurrentHashMap 用于多线程，如果ConcurrentHashMap .get(key) 返回 null，不知道这个null 是没有映射的value 是 null，还是 没找到对应的key 而为null，**有二义性**。而 key 不能为 null 源码就是这么写的，设计之初就不允许key 为 null
* ConcurrentHashMap **迭代器是弱一致性**。迭代器创建后，就会按照哈希表结构遍历每个元素，但在遍历过程中，内部元素可能会发生变化，**如果变化发生在已遍历过的部分，迭代器就不会反映出来，而如果变化发生在未遍历过的部分，迭代器就会发现并反映出来**，这就是弱一致性



### 集合框架

* 整体框架

  ![image-20230305221543965](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230305221543965.png)

* List：一串**有序**的集合
  * ArrayList
  * LinkedList
  * Vector
    * 底层：与 ArrayList 一样，数组实现
    * 线程安全，synchronized 修饰操作方法
  * Stack
    * 继承自 Vector，因此线程安全
    * 后进先出
* Queue：先进先出
  * Deque 接口：双端队列
    * Deque 继承自 Queue 接口，Queue 接口有 **从尾部插入，头部取出** 的操作方法，Deque 在此基础上，添加了从 头部插入，尾部取出的 操作方法
    * Deque在java.util中主要有**ArrayDeque**和**LinkedList**两个实现类，两者一个是基于数组的实现，一个是基于链表的实现
  * PriorityQueue：优先队列
    * 存放元素的数组 和 Comparator 比较器（可以自定义比较规则，如果没有定义规则，按元素实现的Comparable 接口方法作为规则）
    * 当PriorityQueue不为空时插入一个新元素，会对其新元素进行**堆排序**处理
  * ArrayDeque
* Set：元素唯一
  * HashSet ：底层 HashMap
  * LikedHashSet：继承自 HashSet
  * TreeSet
    * TreeSet集合底层实际上是一个TreeMap
    * TreeMap集合底层是一个二叉树。

    * 放到TreeSet集合中的元素，等同于放到TreeMap集合key部分了。

    * TreeSet集合中的元素：无序不可重复，但是可以按照元素的大小顺序自动排序。
* Map：键值对集合
  * HashMap
  * HashTable
* **List 和 Set 的区别**
  * List ： 有序，按对象进入的顺序保存对象，可重复，允许多个 null 元素对象，可以使用 迭代器取出所有元素逐一遍历。还可以根据索引获取指定的元素
  * Set ： 无序，不可重复，最多只允许一个 Null 对象，取元素只能用迭代器取得所有元素逐一遍历
* **数组 List 互相转换**
  * List.toArray()：集合转成 数组

  * Arrays.copyOf()：若数组元素为对象，则数组里面数据是对象引用。通过Arrays.copyOf() 方法产生的数组是一个**浅拷贝**

  * Arrays.asList()：数组转成 集合



## Java8新特性

1. **Lambda表达式**
   * Lambda表达式是一个匿名函数，可以使用它写出简洁灵活的代码。
   * 函数式接口：只包含一个抽象方法的接口，并且可以使用Lambda表达式来创建该接口的对象，使用@FunctionInterface注解，可不加，主要用来检测是否是函数式接口的
   * 内置函数式接口：Function、Consumer（有参无返回值）、Supplier（无参有返回值）、Predicate（有参有返回值，返回值是boolean）
   * 方法引用：有三种情况可以使用方法引用：
     * 对象：：实例方法
     * 类：：实例方法
     * 类：：静态方法
   * 构造器引用：和方法引用类似，类：：new
2. **Stream**
   1. 帮助更好的对数据进行集合操作，提供了高效且易用的处理数据的方式
   2. 特点：
      1. 不会存储数据
      2. 不会改变源对象，会返回一个持有结果的新Stream对象
      3. 延迟执行，等到有结果时才会执行
      4. 和list不同，Stream代表的是任意Java对象的序列，且stream输出的元素可能并没有预先存储在内存中，而是实时计算出来的。可以“存储”有限个或无限个元素

   3. 操作
      1. 创建Stream：.stream()、Stream.of()
      2. 中间操作：map()、mapToInt()、filter()、limit、skip、sorted、distinct
      3. 终止操作：forEach、min、max、count、collect、iterator

3. **接口默认方法**（default修饰）和 静态方法（static修饰）
4. **Optional类**：是一个容器，代表一个值存在或不存在，原来使用null表示，现在用Optional，可以避免空指针异常



## 泛型

* 泛型是一种在**编译时期**实现**类型参数化**的机制，它允许在定义类、接口和方法时使用类型参数，以便在使用这些类、接口和方法时指定具体的类型。

* 好处

  1. 类型安全：使用泛型可以在编译器进行类型检查，避免了在运行时出现类型转换错误的可能
  2. 代码重用：可以编写通用的代码，可用于处理多种类型的数据。比如集合类，如果一个类型一个 LinkedList 类，那要写很多种类，而有了泛型，只需要定义一个 LinkedList 类，在具体使用时再定义存储数据的类型
  3. 简化代码：避免了显式的类型转换，进而减少了运行时的类型检查和类型转换，提高执行效率

* 泛型通配符

  * 类型通配符一般是使用 **？**代替具体的类型实参，注意了，**此处’？’是类型实参，而不是类型形参** 。再直白点的意思就是，此处的 ？和Number、String、Integer一样都是一种实际的类型，可以把 ？看成所有类型的父类。是一种真实的类型。

  * 泛型方法必须使用 <T> 修饰，如：

    ~~~java
    public <T> T showKeyName();
    ~~~



## 反射

* **在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性**；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。
* 获取Class对象的三种方式：
  * 对象实例.getClass()，如 **person.getClass()**
  * 任何数据类型（包括基本数据类型）都有一个“静态”的class属性，如 **Person.class**
  * 通过 Class类的静态方法：forName(String ClassName)，如 **Class.forName("Person")**。常用这种方式
* **注意**：**在运行期间，一个类，只有一个Class对象产生**
* **newInstance() 和 new() 的区别**：
  1. newInstance创建类是这个类必须**已经加载过且已经连接**，new创建类是则不需要这个类加载过
  2. **newInstance: 弱类型**(GC是回收对象的限制条件很低，容易被回收)、低效率、只能调用无参构造，**new 强类型**
* 优点：
  1. 动态性：反射允许在运行时状态动态加载类、创建对象、调用方法和访问字段，而不需要在编译时就确定类的具体信息。**使得代码更加灵活，适应不同的需求和场景**
  2. 灵活性：**通过反射，可以绕过访问权限限制**，访问和修改私有字段和方法。对于特殊需求和场景可能非常有用
  3. 框架和库的支持：Spring 等框架利用反射机制**实现了如依赖注入、动态代理、ORM（对象关系映射）等功能**
* 缺点：
  1. 性能开销：反射需要动态解析类的结构，通过反射API执行相应的操作，可能会导致性能降低
  2. 安全性限制：**反射可以绕过访问权限限制，可能导致安全风险**



## 注解

1. Java 自带注解：

   | 注解名称             | 功能描述                                   |
   | -------------------- | ------------------------------------------ |
   | @override            | 检查该方法是否是重写方法                   |
   | @Deprecated          | 标记过时方法，如果使用该方法，会报编译警告 |
   | @SuppressWarnings    | 指示编译器去忽略注解中声明的警告           |
   | @FunctionalInterface | Java8支持，标识一个匿名函数或函数式接口    |

2. 元注解：用于修饰注解的注解，通常用在注解的定义上

   | 注解名称    | 描述                                                         |
   | ----------- | ------------------------------------------------------------ |
   | @Retention  | 该注解在哪一个级别可用，SOURCE(只在源文件中，编译时丢弃)、CLASS(记录在类文件中，不加载到JVM，默认值)、RUNTIME(保留在源文件、类文件中，加载到JVM) |
   | @Documented | 生成文档信息的时候保留注解，对类作辅助说明                   |
   | @Target     | 用于描述注解的使用范围（即：被描述的注解可以用在什么地方），如类、构造函数、方法等 |
   | @Inherited  | 说明子类可以继承父类中的该注解                               |
   | @Repeatable | 表示注解可以重复使用                                         |

3. 自定义注解：**@interface**

   ~~~java
   // 自定义注解例子
   @Documented
   @Retention(RetentionPolicy.RUNTIME)
   @Target(ElementType.METHOD)
   @Inherited
   public @interface HelloAnnotation {
       String value();
   }
   ~~~

4. **注解的本质就是一个继承了 Annotation 接口的接口**



## 异常

1. 异常：指在程序运行过程中可能出现的错误或异常情况。是一种用于处理错误和异常情况的机制，它允许程序在出现异常时进行适当的处理，以保证程序的稳定西和可靠性

2. 异常架构

   1. 都是继承自 Throwable 类，分为 Error 和 Exception，Exception 下有受检异常和非受检异常
   2. Error：
      * 如果应用程序出现了 Error，将无法恢复，只能重新启动应用程序
      * 典型的Error异常是：OutOfMemoryError 内存不足异常
   3. 受检异常：
      * 编译器必须处理的异常，编译期检查
      * **除RuntimeException及其子类**，其他的 Exception 异常都属用受检异常。需要使用 try-catch 捕获，或者在方法签名中用throws 关键字抛出，否则编译不通过
   4. 非受检异常：
      * 又称为 运行时异常
      * 编译器不会进行检查并且不要求处理的异常，没有 try-catch 或throws 也能正常通过

3. Error和Exception的区别

   * Error：虚拟机相关错误，比如系统崩溃、内存不足、堆栈溢出等，编译器不会对这类错误进行检测，也不应该对这类错误进行捕获。一旦这类错误发生，通常应用程序会被终止
   * Exception： 可以在应用程序中进行捕获并处理，遇到这类错误，应进行处理，使应用程序可以继续正常运行

4. throw和throws的区别

   * throw：用在**方法内部**，**只能抛出一种异常**，受检异常和非受检异常都可以被抛出
   * throws：用在**方法声明上**，**可以抛出多个异常**，用来标识该方法可能抛出的异常列表。一个方法用 throws 标识了，调用该方法的方法中要包含可处理异常的代码，或者也用 throws 声明相应的异常

5. try-catch-finally 中哪部分可以省略

   * **catch 可以省略**：如果在 try 块中没有需要捕获的异常，或者将异常抛给上层调用处理，那么可以省略 catch 块。这种情况通常与 throws 语句一起使用，**将异常传递给上层调用进行处理**。

     ~~~java
     try {
         // 可能抛出异常的代码
     } throws SomeException {
         // 异常处理逻辑
     }
     ~~~

   * finally 块省略：finally块是可选的，可以省略不写

6. 常见的运行时异常（RuntimeException）：

   1. NullPointerException（空指针异常）
   2. ArrayIndexOutOfBoundsException（数组索引越界异常）
   3. ClassCastException（类转换异常）
   4. llegalArgumentException（非法参数异常）
   5. llegalStateException（非法状态异常）
   6. ArithmeticException（算术异常）
   7. NumberFormatException（数字格式化异常）
   8. UnsupportedOperationException（不支持的操作异常）







