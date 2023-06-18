# Java（2023.02.21）

## 数据结构

1. HashMap 底层

   1. HashMap 采用 **链地址法** 解决 哈希冲突

   2. HashMap 在 jdk1.7 中使用 **数组+链表** 实现，jdk1.8 使用 **数组+链表+红黑树** 实现

   3. HashMap 源码：

      ~~~java
      public class HashMap<K,V> extends AbstractMap<K,V>
          implements Map<K,V>, Cloneable, Serializable {
          
          // 默认初始化容量：16
          static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
          // 最大容量：2^30
          static final int MAXIMUM_CAPACITY = 1 << 30;
          // 默认负载因子：0.75f
          static final float DEFAULT_LOAD_FACTOR = 0.75f;
          // 当链表长度为 8 时，链表转换成红黑树
          static final int TREEIFY_THRESHOLD = 8;
          // 节点小于 6 时，红黑树转换回链表
          static final int UNTREEIFY_THRESHOLD = 6;
          // 最小树容量
          static final int MIN_TREEIFY_CAPACITY = 64;
          
          // HashMap 底层，table数组
          transient Node<K,V>[] table;
          // 大小，含有的key-value数量
          transient int size;
          // 用于快速失败，由于HashMap非线程安全，在对HashMap进行迭代时，如果期间其他线程的参与导致HashMap的结构发生变化了（比如put，remove等操作），需要抛出异常ConcurrentModificationException
          transient int modCount;
          // 阈值，当table == {}时，该值为初始容量（初始容量默认为16）；当table被填充了，也就是为table分配内存空间后，threshold一般为 capacity*loadFactory。HashMap在进行扩容时需要参考threshold
          int threshold;
          // 负载因子
          final float loadFactor;
          
          
          // HashMap 的节点类，也就是 key-value 键值对
          static class Node<K,V> implements Map.Entry<K,V>{
              final int hash;	// 只计算一次，避免重复计算
              final K key;
              V value;
              Node<K,V> next; // 指向下一个节点，单链表结构
              
              // hash 计算为 key的hash 异或 value的hash
              public final int hashCode() {
                  return Objects.hashCode(key) ^ Objects.hashCode(value);
              }
              // equals() 方法是 key相同并且value相同
              public final boolean equals(Object o) {......}
          }
          
          // HashMap 的构造器：（初始化容量，负载因子）、（初始化容量）、默认构造器（负载因子为默认负载因子）
          
          // HashMap 的 hash 算法
          static final int hash(Object key) {
              int h;
              return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
          }
          
          // 链表 转换成 树
          final void treeifyBin(Node<K,V>[] tab, int hash) {
              int n, index; Node<K,V> e;
              // 当 table 数组的大小 小于 最小转换成树的容量：64 时，不转换成树，而是扩容
              if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
                  resize();
              else if ((e = tab[index = (n - 1) & hash]) != null) {
                  TreeNode<K,V> hd = null, tl = null;
                  do {
                      TreeNode<K,V> p = replacementTreeNode(e, null);
                      if (tl == null)
                          hd = p;
                      else {
                          p.prev = tl;
                          tl.next = p;
                      }
                      tl = p;
                  } while ((e = e.next) != null);
                  if ((tab[index] = hd) != null)
                      hd.treeify(tab);
              }
          }
          
          // clear() 会清空 table 数组
          public void clear() {...}
          
          // 遍历
          public void forEach(BiConsumer<? super K, ? super V> action) {
              Node<K,V>[] tab;
              if (action == null)
                  throw new NullPointerException();
              if (size > 0 && (tab = table) != null) {
                  // 获取 modCount
                  int mc = modCount;
                  for (int i = 0; i < tab.length; ++i) {
                      for (Node<K,V> e = tab[i]; e != null; e = e.next)
                          action.accept(e.key, e.value);
                  }
                  // 如果遍历过程中，modCount 发生了改变，意味着HashMap被修改了，抛出异常
                  // 修改HashMap的方法都会让 modCount 自增，如put()、remove()等
                  if (modCount != mc)
                      throw new ConcurrentModificationException();
              }
          }
      }    
      ~~~

   4. **HashMap 插入值**：

      往数组中添加一个键值对，首先根据key，通过 hash() 方法计算出 key 的hash值；假设计算出来的 key 的hash值为 hashValue，那么再通过 **index = (length - 1) & hashValue** 这个公式计算出 index（下标）的值（**该公式** **保证了获取的index一定在数组范围内**），这里的 length  是数组的长度，最后将键值对掺入数组中下标为 index 的位置。

      通过 index = (length  - 1) & hashValue 可知，**计算出来的 index 是强依赖于数组长度**，那么当 **HashMap 扩容**之后，所有元素在数组中的下标需要**重新计算**

      如果计算出来的 index 位置已经有元素了，这个时候不会再另外计算一次 index，而是用链表的形式插入到 index 的位置（**jdk1.7 采用 头插法，jdk1.8 采用尾插法**，**使用尾插法的原因**：因为在多线程的的情况下可能会出现**环形链**的情况，因为HashMap在扩容之后是会重新计算每个元素的位置的，元素的位置改变说明各个元素之前的引用就会改变，所以在多线程下操作jdk1.7的HashMap的时候可能会出现死循环。而该用jdk8的尾插法，元素之前的引用顺序不会改变，也就不会引起死循环。）

   5. **HashMap 扩容**：

      HashMap根据两个属性进行扩容：capacity（默认容量为16，1<<<4），loadFactor（负载因子，默认为0.75）

      在默认情况下，16 * 0.75 = 12，当添加第13个元素时，就会进行扩容，**扩容之后的容量是原来容量的2倍**，并重新计算每个元素的index

      **为什么每次扩容都是 2 倍：**

      * hashMap的数组长度一定保持2的次幂，比如16的二进制表示为 10000，那么length-1就是15，二进制为01111，同理扩容后的数组长度为32，二进制表示为100000，length-1为31，二进制表示为011111。从下图可以我们也能看到这样会保证低位全为1，而扩容后只有一位差异，也就是多出了最左位的1，这样在通过 h&(length-1)的时候，只要h对应的最左边的那一个差异位为0，就能**保证得到的新的数组索引和老数组索引一致(大大减少了之前已经散列良好的老数组的数据位置重新调换)**
      * 还有，数组长度保持2的次幂，length-1的低位都为1，会使得**获得的数组索引index更加均匀**
      * 上面的&运算，**高位是不会对结果产生影响的**（hash函数采用各种位运算可能也是为了使得低位更加散列），我们只关注低位bit，如果低位全部为1，那么**对于h低位部分来说，任何一位的变化都会对结果产生影响**，也就是说，要得到index=21这个存储位置，h的低位只有这一种组合。这也是数组长度设计为必须为2的次幂的原因。
      * 如果不是2的次幂，也就是低位不是全为1此时，要使得index=21，h的低位部分不再具有唯一性了，哈希冲突的几率会变的更大，同时，index对应的这个bit位无论如何不会等于1了，而对应的那些数组位置也就被白白浪费了

   6. **为什么要使用红黑树**

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

   7. hash() 方法为什么要用 h>>>16:

      h >>> 16 相当于将高区的 16 位移动到低区的 16 位，再与原 hashCode 做异或运算，可以**将高低位二进制特征混合起来**

      由于计算 hash 值后，需要计算HashMap中数组的索引，计算公式：（length - 1）& hash，假设 length为默认值 16，计算时hash的高位的16位很有可能会被 length-1 屏蔽，**如果不做刚才的移位异或运算，那么在计算索引时将丢失高位特征**

      失去高位特征意味着hash碰撞的可能性会增大

      **使用异或运算的原因**：异或运算能更好的保留各部分的特征，如果采用 & 运算计算出来的值会向 0 靠拢，采用 | 运算计算出来的值会向 1 靠拢

      2倍扩容的原因也是类似：假设数组大小是 17，则索引公式就是 （17 - 1）& hash，16 的二进制是1 0000，参与 & 计算后，hash 会被更多的 0 屏蔽，计算结果只剩下 0 和 16 两种情况，哈希碰撞急剧上升

      综上，h>>>16 和 2倍扩容 的目的都是 为了让hash 后的结果更均匀的分布，减少哈希碰撞

   8. HashMap 采用了 **懒加载**，在创建 HashMap 时并没有初始化 Node<K,V>[] table，在第一次 put 时才初始化这个 table

   9. **HashMap 为什么在链表长度大于8时转换成红黑树**

      * 官方做过很多测试，**发现链表长度在大于8以后再出现hash 碰撞的可能性为 0.00000006（几乎为0）**，**也就是说哈希碰撞同一个位置8次的概率几乎为0**，虽然转换为红黑树后，查询效率比链表高，但是转换红黑树的过程是耗时的，而且在扩容时还要对红黑树重新左旋或右旋保持平衡。***\*阈值设置为8就是为了尽量减少hashmap中出现红黑树\****（hashmap中链表才是常态） 

      

2. 集合框架

   * Java 整个集合框架

     ![image-20230305221543965](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230305221543965.png)

   

   * List ：一串**有序**的集合

     1. ArrayList

        * 底层：transient Object elementData，elementData 是存放元素的容器，**ArrayList 基于数组实现**

        * ArrayList支持默认大小构造，和空构造，当空构造的时候存放数据的Object[] elementData是一个空数组 []。

        * ArrayList 有 modCount 属性，用于 快速失败机制

        * 当初始化的list是一个空ArrayList的时候，会直接扩容到DEFAULT_CAPACITY，该值大小是一个**默认值10**。而当添加进ArrayList中的元素超过了数组能存放的最大值就会进行扩容，扩容代码：

          ~~~java
          // 右移一位，相当于扩容为原先的 1.5 倍
          int newCapacity = oldCapacity + (oldCapacity >> 1);
          ~~~

        * 数组copy：Java 无法自己分配空间，通过 arraycopy 这个 **native 方法实现数组的复制**

     2. LinkedList

        * 底层：LinkedList由一个头节点和一个尾节点组成，分别指向链表的头部和尾部，是**双向链表**
        * 查询方法：先判断 index 更靠近头部还是尾部，靠近哪段从哪段遍历获取值
        * 修改：采用尾插法

     3. Vector

        * 底层：与 ArrayList 一样，数组实现
        * 线程安全，synchronized 修饰操作方法

     4. Stack

        * 继承自 Vector，因此线程安全
        * 后进先出

   * Queue：先进先出

     1. Deque 接口
        * Deque英文全称是Double ended queue，也就是俗称的双端队列。就是说对于这个队列容器，既可以从头部插入也可以从尾部插入，既可以从头部获取，也可以从尾部获取
        * Deque 继承自 Queue 接口，Queue 接口有 **从尾部插入，头部取出** 的操作方法，Deque 在此基础上，添加了从 头部插入，尾部取出的 操作方法
        * Deque在java.util中主要有**ArrayDeque**和**LinkedList**两个实现类，两者一个是基于数组的实现，一个是基于链表的实现
     2. PriorityQueue
        * Java中唯一一个Queue接口的直接实现，如其名字所示，优先队列，其内部支持按照一定的规则对内部元素进行排序。
        * 底层：存放元素的数组 和 Comparator 比较器（可以自定义比较规则，如果没有定义规则，按元素实现的Comparable 接口方法作为规则）
        * 当PriorityQueue不为空时插入一个新元素，会对其新元素进行**堆排序**处理
     3. ArrayDeque
        * 基于数组实现的双端队列
        * ArrayDeque是Deque接口的实现，和LinkedList不同的是，LinkedList既是List接口也是Deque接口的实现

   * Set：元素唯一

     1. HashSet
        * 底层：是一种Hash实现的集合，使用的底层结构是HashMap
   
     2. LinkedHashSet
        * 继承自 HashSet
   
     3. TreeSet
   
        * TreeSet集合底层实际上是一个TreeMap
   
        * TreeMap集合底层是一个二叉树。
   
        * 放到TreeSet集合中的元素，等同于放到TreeMap集合key部分了。
   
        * TreeSet集合中的元素：无序不可重复，但是可以按照元素的大小顺序自动排序。
   
   * Map：键值对集合
   
     ![image-20230305225523415](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230305225523415.png)
   
     1.  HashMap：见上文
     2.  HashTable
         * **HashTable和HashMap虽然都实现了Map接口，但是HashTable继承了DIctionary抽象类，而HashMap继承了AbstractMap抽象类**
         * Dictionary类中有这样一行注释，当key为null时会抛出空指针NullPointerException,这也说明了HashTabel是不允许Key为null
         * HashTable 底层存储是 Entry<?,?>[] table，HashMap 底层存储是 Node<K,V>[] table。HashTable 的 Entry 是单链表，而HashMap 不仅有 table 单链表，还有 TreeNode 树结构
         * HashTable中方法是用synchronized修饰的，所以其操作是**线程安全**的，但是效率会受影响。
     3.  
     
     
     
     3. 数组 List 互相转换
     
        * List.toArray()：集合转成 数组
     
        * Arrays.copyOf()：若数组元素为对象，则数组里面数据是对象引用。通过Arrays.copyOf() 方法产生的数组是一个**浅拷贝**
     
        * Arrays.asList()：数组转成 集合
     
          * Arrays.asList() 出现莫名其妙的问题，如下：
     
            ~~~java
            public static void main(String[] args) {
                int[] datas = new int[]{1,2,3,4,5};
                List list = Arrays.asList(datas);
                System.out.println(list.size());
            }
            // 预期结果是 5，而实际是 1
            ------------Output:
            1
            ~~~
     
            ~~~java
            // asList() 源码
            public static <T> List<T> asList(T... a) {
                return new ArrayList<T>(a);
            }
            ~~~
     
            注意这个参数:T…a，这个参数是一个**泛型的变长参数**，我们知道基本数据类型是不可能泛型化的，也是就说8个基本数据类型是不可作为泛型参数的，但是为什么编译器没有报错呢？这是因为在java中，**数组会当做一个对象来处理，它是可以泛型的**，所以我们的程序**是把一个int型的数组作为了T的类型，所以在转换之后List中就只会存在一个类型为int数组的元素了**。所以我们这样的程序System.out.println(datas.equals(list.get(0)));输出结果肯定是true。当然如果将int改为Integer，则长度就会变成5了。
     
            注意，**asList() 方法返回的 ArrayList 不是 java.util 中的 ArrayList，**而是Arrays 工具类的内部类，该内部类没有add() 方法，其父类的add() 方法也没有具体实现。**asList返回的是一个长度不可变的列表。数组是多长，转换成的列表是多长，我们是无法通过add、remove来增加或者减少其长度的。**
     
     
     
## Enum 枚举

1. 枚举的定义和使用

   ~~~java
   public class EnumDemo {
   
       public static void main(String[] args){
           //直接引用
           Day day =Day.MONDAY;
       }
   
   }
   //定义枚举类型
   enum Day {
       MONDAY, TUESDAY, WEDNESDAY,
       THURSDAY, FRIDAY, SATURDAY, SUNDAY
   }
   ~~~

2. 枚举的底层原理

   * 使用关键字enum创建枚举类型并编译后，编译器会为我们生成一个相关的类，这个类继承了Java API中的java.lang.Enum类，也就是说通过关键字enum创建枚举类型在编译后事实上也是一个类类型而且该类继承自java.lang.Enum类

   * 反编译后：

     ~~~java
     //反编译Day.class
     final class Day extends Enum
     {
         //编译器为我们添加的静态的values()方法
         public static Day[] values()
         {
             return (Day[])$VALUES.clone();
         }
         //编译器为我们添加的静态的valueOf()方法，注意间接调用了Enum类的valueOf方法
         public static Day valueOf(String s)
         {
             return (Day)Enum.valueOf(com/zejian/enumdemo/Day, s);
         }
         //私有构造函数
         private Day(String s, int i)
         {
             super(s, i);
         }
          //前面定义的7种枚举实例
         public static final Day MONDAY;
         public static final Day TUESDAY;
         public static final Day WEDNESDAY;
         public static final Day THURSDAY;
         public static final Day FRIDAY;
         public static final Day SATURDAY;
         public static final Day SUNDAY;
         private static final Day $VALUES[];
     
         static 
         {    
             //实例化枚举实例
             MONDAY = new Day("MONDAY", 0);
             TUESDAY = new Day("TUESDAY", 1);
             WEDNESDAY = new Day("WEDNESDAY", 2);
             THURSDAY = new Day("THURSDAY", 3);
             FRIDAY = new Day("FRIDAY", 4);
             SATURDAY = new Day("SATURDAY", 5);
             SUNDAY = new Day("SUNDAY", 6);
             $VALUES = (new Day[] {
                 MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
             });
         }
     }
     ~~~

   * 从上可知，编译器会帮助生成一个 **Day 类**（被 **final 修饰**，不可被继承），该类继承自 Enum 类。还帮助生成了 **7 个 Day 类型的实例对象**，这些实例对象都被 （**public static final 修饰**，常量的修饰），并为 Day 类创建了两个方法 values() 和 valueOf() 

   * 枚举类的常用方法

     1. compareTo(E o) ：比较此枚举与指定对象的顺序，返回值是比较对象的 ordinal 值的差值

     2. name() : 与 toString() 方法等同，返回此枚举常量的名称

     3. ordinal() : 返回枚举常量的序数

        ~~~java
        Day day = new Day[]{Day.MONDAY, Day.TUESDAY, Day.WEDNESDAY,
                        Day.THURSDAY, Day.FRIDAY, Day.SATURDAY, Day.SUNDAY};
        // ordinal()
        System.out.println("day["0"].ordinal():"+days[0].ordinal()); // 输出：0
        // 通过compareTo方法比较,实际上其内部是通过ordinal()值比较的
        System.out.println("days[0].compareTo(days[1]):"+days[0].compareTo(days[1])); // 输出：-1（0 - 1 = -1）
        // name()
        System.out.println("days[0].name():"+days[0].name()); // 输出：MONDAY
        ~~~



## 泛型

1. 解释：将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。

   泛型的**本质是为了参数化类型**（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。也就是说在泛型使用过程中，操作的数据类型被指定为一个参数

2. **泛型只在编译阶段有效**。在编译之后程序会采取**去泛型化**的措施。也就是说Java中的泛型，只在编译阶段有效。在编译过程中，**正确检验泛型结果后，会将泛型的相关信息擦出**，并且在对象进入和离开方法的边界处**添加类型检查和类型转换的方法**。也就是说，泛型信息不会进入到运行时阶段。

3. 泛型通配符：

4. 类型通配符一般是使用 **？**代替具体的类型实参，注意了，**此处’？’是类型实参，而不是类型形参** 。再直白点的意思就是，此处的 ？和Number、String、Integer一样都是一种实际的类型，可以把 ？看成所有类型的父类。是一种真实的类型。

   可以解决当具体类型不确定的时候，这个通配符就是 ?  ；当操作类型时，不需要使用类型的具体功能时，只使用Object类中的功能。那么可以用 ? 通配符来表未知类型。

5. 泛型方法必须使用 <T> 修饰，如：

   ~~~java
   public <T> T showKeyName();
   ~~~

6. **泛型信息对 Java 编译器可以见，对 Java 虚拟机不可见。**

7. 使用泛型的**好处**：
   在没有使用泛型的时期，假如我们定义List容器没有存储数据的类型，那我们就可以存储多种数据类型，比如我们原意是要存储整型，如果存储了部分字符串类型； 在获取list值value时，需要强制类型转换，有可能造成一些错误。如 ClassCastException 异常。

   好处

   1. 提供了编译期类型安全，确保只把正确类型的对象放入对象中
   2. 避免强制类型转换

8. **泛型擦除机制**



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



## 反射

1. **在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性**；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。
2. 获取Class对象的三种方式：
   * 对象实例.getClass()，如 **person.getClass()**
   * 任何数据类型（包括基本数据类型）都有一个“静态”的class属性，如 **Person.class**
   * 通过 Class类的静态方法：forName(String ClassName)，如 **Class.forName("Person")**。常用这种方式
3. **注意**：**在运行期间，一个类，只有一个Class对象产生**
4. **newInstance() 和 new() 的区别**：
   1. newInstance创建类是这个类必须**已经加载过且已经连接**，new创建类是则不需要这个类加载过
   2. **newInstance: 弱类型**(GC是回收对象的限制条件很低，容易被回收)、低效率、只能调用无参构造，**new 强类型**



## 类加载过程

### 一、加载

* 加载指的是**将类的class文件读入到内存**，并**将这些静态数据转换成方法区中的运行时数据结构**，并在堆中生成一个代表这个类的java.lang.Class对象，作为方法区类数据的访问入口，这个过程需要类加载器参与

1. 类加载器
   * **启动类加载器**（Bootstrap ClassLoader）
     * 负责加载 JAVA_HOME\lib 目录中的，或通过-Xbootclasspath 参数指定路径中的，且被虚拟机认可（按文件名识别，如 rt.jar）的类。
   * **扩展类加载器**（Extension ClassLoader）
     * 负责加载 JAVA_HOME\lib*.jar 目录中的，或通过 java.ext.dirs 系统变量指定路径中的类库。
   * **应用程序类加载器**（Application ClassLoader）
   * 自定义类加载器（JVM并没有提供）
     * 应用程序根据自身需要自定义的ClassLoader，需要继承ClassLoader 类。
2. 双亲委派模型
   * **当一个类加载器收到了类加载请求，他首先不会尝试自己去加载这个类，而是把这个请求委派给父类去完成**。因此所有的加载请求都应该传送到启动类加载器中，只有当父类加载器反馈自己无法完成这个请求的时候（在它的加载路径下没有找到所需加载的Class），子类加载器才会尝试自己去加载。
   * 好处：不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同样一个 Object 对象



### 二、连接

* 当类被加载之后，系统为之生成一个对应的Class对象，接着将会进入连接阶段，**连接阶段负责把类的二进制数据合并到JRE中**（意思就是将java类的二进制代码合并到JVM的运行状态之中）。类连接又可分为如下3个阶段

1. 验证

   * 这一阶段的主要目的是**为了确保 Class 文件的字节流中包含的信息是否符合当前虚拟机的要求**，并且不会危害虚拟机自身的安全

2. 准备

   * 准备阶段是正式**为类变量分配内存并设置类变量的初始值**阶段，即在**方法区**中分配这些变量所使用的内存空间

   * **注意这里所说的初始值概念**：

     1. 如果一个类变量定义为：

        ~~~java
        public static int v = 8080;
        
        // 实际上变量 v 在准备阶段过后的初始值为 0, 而不是 8080。将 v 赋值为 8080 的 put static 指令是程序被编译后，存放于类构造器方法之中。
        ~~~

     2. 如果声明为：

        ~~~java
        public static final int v = 8080;
        
        // 在编译阶段会为 v 生成 ConstantValue 属性，在准备阶段虚拟机会根据 ConstantValue 属性将 v 赋值为 8080。
        ~~~

3. 解析

   * **解析阶段是指虚拟机将常量池中的符号引用替换为直接引用的过程**，
   * 符号引用就是 class 文件中的：
     1. CONSTANT_Class_info
     2. CONSTANT_Field_info
     3. CONSTANT_Method_info
        等类型的常量。
   * **符号引用**：**符号引用与虚拟机实现的布局无关**，引用的目标并不一定要已经加载到内存中。各种虚拟机实现的内存布局可以各不相同，但是它们能接受的**符号引用必须是一致的**，因为符号引用的字面量形式**明确定义在 Java 虚拟机规范的 Class 文件格式中**
   * **直接引用**：直接引用可以是指向目标的指针，相对偏移量或是一个能间接定位到目标的句柄。如果有了直接引用，那引用的目标必定已经在内存中存在



### 三、初始化

1. 初始化阶段是类加载最后一个阶段，到了初始阶段，才开始真正执行类中定义的 Java 程序代码。
2. JVM规范**并没有规定何时加载类**，但是严格规定了**什么时候必须初始化**：
   1. 类的主动引用（一定会发生类的初始化）
      * 调用类的静态成员（除了final常量），和静态方法
      * java.lang.reflect对类进行反射调用时
      * 初始化子类的时候，父类首先初始化
      * 虚拟机启动时，被执行的主类必须初始化
      * new一个类的对象
   2. 类的被动引用（不会发生类的初始化）
      * 当访问一个静态域时，只有真正声明这个域的类才会被初始化，如：当通过子类引用父类的静态变量时，不会导致子类初始化
      * 通过**数组定义类引用，不会触发此类的初始化**，如 int[] arr=new int[]
      * 引用常量不会发生此类的初始化，**常量在编译阶段就存入常量池了**
3. 初始化顺序
   * **静态变量/静态代码块 -> main方法 -> 非静态变量/代码块 -> 构造方法**
   * 父类–静态变量/父类–静态初始化块
     子类–静态变量/子类–静态初始化块
     父类–变量/父类–初始化块
     父类–构造器
     子类–变量/子类–初始化块
     子类–构造器
4. 卸载
   * 以下情况下，会触发卸载
     * java堆中不存在该类的任何实例。
     * 加载该类的ClassLoader已经被回收
     * 该类对应的java.lang.Class对象没有任何地方被引用，无法在任何地方通过反射访问该类的方法





## 设计模式-单例模式

* 饿汉式单例

  ~~~java
  // 饿汉式（基于classloder机制避免了多线程的同步问题）
  public class SingletonHungry {
  	// 私有静态，类加载时就创建实例
      private static SingletonHungry instance = new SingletonHungry();
  	// 构造器私有
      private SingletonHungry() {
      }
  
      public static SingletonHungry getInstance() {
          return instance;
      }
  }
  // 缺点：
  // 无法做到延迟创建对象，事实上如果该单例类涉及资源较多，创建比较耗时间时，我们更希望它可以尽可能地延迟加载，从而减小初始化的负载，因此有了 懒汉式单例
  ~~~

* 懒汉式单例

  ~~~java
  // 懒汉式单例模式（适合多线程安全）
  public class SingletonLazy {
  	// 私有静态 volatile
      private static volatile SingletonLazy instance;
  
      private SingletonLazy() {
      }
  
      // 加锁
      public static synchronized SingletonLazy getInstance() {
          if (instance == null) {
              instance = new SingletonLazy();
          }
          return instance;
      }
  }
  // 能够在多线程中很好的工作避免同步问题，同时也具备lazy loading机制，遗憾的是，由于synchronized的存在，效率很低，在单线程的情景下，完全可以去掉synchronized，为了兼顾效率与性能问题，改进后代码如下：
  ~~~

* 双重检查锁：

  ~~~java
  public class Singleton {
      private static volatile Singleton singleton = null;
  
      private Singleton(){}
  
      public static Singleton getSingleton(){
          if(singleton == null){
              synchronized (Singleton.class){
                  if(singleton == null){
                      singleton = new Singleton();
                  }
              }
          }
          return singleton;
      }    
  }
  // 主要在getSingleton()方法中，进行两次null检查。这样可以极大提升并发度，进而提升性能
  ~~~

* 静态内部类：

  ~~~java
  // 静态内部类
  public class SingletonInner {
      private static class Holder {
          private static SingletonInner singleton = new SingletonInner();
      }
  
      private SingletonInner(){}
  
      public static SingletonInner getSingleton(){
          return Holder.singleton;
      }
  }
  // 避免了静态实例在Singleton类的加载阶段就创建对象，线程安全
  // 从上述4种单例模式的写法中，似乎也解决了效率与懒加载的问题，但是它们都有两个共同的缺点：
  // 1. 序列化可能会破坏单例模式，比较每次反序列化一个序列化的对象实例时都会创建一个新的实例
  // 2. 使用反射强行调用私有构造器，解决方式可以修改构造器，让它在创建第二个实例的时候抛异常
  ~~~

* 枚举单例：

  ~~~java
  // 枚举单例，解决了上述两个缺点
  public enum  SingletonEnum {
      INSTANCE;
      private String name;
      public String getName(){
          return name;
      }
      public void setName(String name){
          this.name = name;
      }
  }
  ~~~



## 设计模式-工厂模式

* 简单工厂：在汉堡店里点餐，输入汉堡名，返回对应的汉堡。**汉堡店里点餐**

  ~~~java
  // 产品类
  abstract class BMW {
  	public BMW(){}
  }
   
  public class BMW320 extends BMW {
  	public BMW320() {
  		System.out.println("制造-->BMW320");
  	}
  }
  public class BMW523 extends BMW{
  	public BMW523(){
  		System.out.println("制造-->BMW523");
  	}
  }
  
  // 工厂
  public class Factory {
  	public BMW createBMW(int type) {
  		switch (type) {
  		case 320:
  			return new BMW320();
  		case 523:
  			return new BMW523();
  		default:
  			break;
  		}
  		return null;
  	}
  }
  
  public class Customer {
  	public static void main(String[] args) {
  		Factory factory = new Factory();
  		BMW bmw320 = factory.createBMW(320);
  		BMW bmw523 = factory.createBMW(523);
  	}
  }
  // 缺点在于不符合“开闭原则”，每次添加新产品就需要修改工厂类，并且工厂类集中了所有产品创建逻辑，一旦不能正常工作，整个系统都要受到影响
  ~~~

* 工厂方法：不同汉堡店生产不同的汉堡，如果想吃麦当劳的汉堡，就去麦当劳的店铺。**选择吃哪家的汉堡**

  ~~~java
  // 产品类
  abstract class BMW {
  	public BMW(){}
  }
  public class BMW320 extends BMW {
  	public BMW320() {
  		System.out.println("制造-->BMW320");
  	}
  }
  public class BMW523 extends BMW{
  	public BMW523(){
  		System.out.println("制造-->BMW523");
  	}
  }
  
  // 工厂类
  interface FactoryBMW {
  	BMW createBMW();
  }
   // 每个工厂只生产特定的产品
  public class FactoryBMW320 implements FactoryBMW{
  	@Override
  	public BMW320 createBMW() {
  		return new BMW320();
  	}
   
  }
  public class FactoryBMW523 implements FactoryBMW {
  	@Override
  	public BMW523 createBMW() {
  		return new BMW523();
  	}
  }
  
  public class Customer {
  	public static void main(String[] args) {
  		FactoryBMW320 factoryBMW320 = new FactoryBMW320();
  		BMW320 bmw320 = factoryBMW320.createBMW();
   
  		FactoryBMW523 factoryBMW523 = new FactoryBMW523();
  		BMW523 bmw523 = factoryBMW523.createBMW();
  	}
  }
  // 每个产品一条生产线，也就是如果再新增一个产品，需要同时新增一个生产该产品的工厂
  // 缺点在于，每增加一个产品都需要增加一个具体产品类和实现工厂类，使得系统中类的个数成倍增加，在一定程度上增加了系统的复杂度，同时也增加了系统具体类的依赖
  ~~~

* 抽象工厂：**汉堡店不能只生产汉堡，还要生产炸鸡**

  ~~~java
  //发动机以及型号  
  public interface Engine {}  
   
  public class EngineA implements Engine{  
      public EngineA(){  
          System.out.println("制造-->EngineA");  
      }  
  }  
  public class EngineB implements Engine{  
      public EngineB(){  
          System.out.println("制造-->EngineB");  
      }  
  }  
   
   
  //空调以及型号  
  public interface Aircondition {} 
   
  public class AirconditionA implements Aircondition{  
      public AirconditionA(){  
          System.out.println("制造-->AirconditionA");  
      }  
  }  
  public class AirconditionB implements Aircondition{  
      public AirconditionB(){  
          System.out.println("制造-->AirconditionB");  
      }  
  } 
  
  //创建工厂的接口  
  public interface AbstractFactory {  
      //制造发动机
      public Engine createEngine();
      //制造空调 
      public Aircondition createAircondition(); 
  }  
   
  //为宝马320系列生产配件  
  public class FactoryBMW320 implements AbstractFactory{     
      @Override  
      public Engine createEngine() {    
          return new EngineA();  
      }  
      @Override  
      public Aircondition createAircondition() {  
          return new AirconditionA();  
      }  
  }  
  //宝马523系列
  public class FactoryBMW523 implements AbstractFactory {  
       @Override  
      public Engine createEngine() {    
          return new EngineB();  
      }  
      @Override  
      public Aircondition createAircondition() {  
          return new AirconditionB();  
      }  
  } 
  
  public class Customer {  
      public static void main(String[] args){  
          //生产宝马320系列配件
          FactoryBMW320 factoryBMW320 = new FactoryBMW320();  
          factoryBMW320.createEngine();
          factoryBMW320.createAircondition();
            
          //生产宝马523系列配件  
          FactoryBMW523 factoryBMW523 = new FactoryBMW523();  
          factoryBMW523.createEngine();
          factoryBMW523.createAircondition();
      }  
  }
  
  // 缺点在于添加新的行为时比较麻烦，如果需要添加一个新产品族对象时，需要更改接口及其下所有子类
  ~~~



## 设计模式-适配器模式

* 类的适配器模式

  ~~~java
  // 目标接口，或称为标准接口
  interface Target {
  	public void request();
  }
  
  // 现有的类，不能修改它，不符合我们既有的标准接口的类
  class Adaptee {
  	public void specificRequest() {
  		System.out.println("被适配类具有 特殊功能...");
  	}
  }
  
  // 适配器类，继承了被适配类，同时实现标准接口
  class Adapter extends Adaptee implements Target{
  	public void request() {
  		super.specificRequest();
  	}
  }
  ~~~

* 对象的适配器模式：**委托**的方式

  ~~~java
  // 适配器类，直接关联被适配类，同时实现标准接口
  class Adapter implements Target{
  	// 直接关联被适配类
  	private Adaptee adaptee;
  	
  	// 可以通过构造函数传入具体需要适配的被适配类对象
  	public Adapter (Adaptee adaptee) {
  		this.adaptee = adaptee;
  	}
  	
  	public void request() {
  		// 这里是使用委托的方式完成特殊功能
  		this.adaptee.specificRequest();
  	}
  }
  // 使用对象适配器模式，可以使得 Adapter 类（适配类）根据传入的 Adaptee 对象达到适配多个不同被适配类的功能，当然，此时我们可以为多个被适配类提取出一个接口或抽象类
  ~~~

* 接口的适配器模式：有时我们写的一个接口中有多个抽象方法，**当我们写该接口的实现类时，必须实现该接口的所有方法**，这明显有时比较浪费，**因为并不是所有的方法都是我们需要的**，有时只需要某一些，此处为了解决这个问题，我们引入了接口的适配器模式，**借助于一个抽象类，该抽象类实现了该接口，实现了所有的方法，而我们不和原始的接口打交道，只和该抽象类取得联系**，所以我们写一个类，继承该抽象类，重写我们需要的方法就行。

  ~~~java
  // 接口
  public interface Sourceable {
  	public void method1();
  	public void method2();
  }
  // 抽象类实现两个接口方法
  public abstract class Wrapper2 implements Sourceable{
  	public void method1(){}
  	public void method2(){}
  }
  
  public class SourceSub1 extends Wrapper2 {
  	public void method1(){
  		System.out.println("the sourceable interface's first Sub1!");
  	}
  }
   
  public class SourceSub2 extends Wrapper2 {
  	public void method2(){
  		System.out.println("the sourceable interface's second Sub2!");
  	}
  }
  ~~~



## Spring AoP 动态代理

1. jdk 动态代理

   * 代码

     ~~~java
     // 被代理类的接口
     public interface IService {
         public void add();
         public void update();
     }
     
     // 被代理类
     public class ServiceImpl implements IService{
         @Override
         public void add() {
             System.out.println("IService.add()>>>");
         }
     
         @Override
         public void update() {
             System.out.println("IService.update()>>>");
         }
     }
     
     // 处理器，必须继承 InvocationHandler ，在这个类里面织入 切面
     public class MyInvocationHandler implements InvocationHandler {
         private Object target;
     
         public MyInvocationHandler() {
             super();
         }
     
         public MyInvocationHandler(Object target) {
             super();
             this.target = target;
         }
     
         // invoke() 方法中织入 切面，这里是定义了一个通用的织入了特定切面，target可以是任意类
         @Override
         public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
             System.out.println("Before------------------------");
             Object result = method.invoke(target, args);
             System.out.println("After------------------------");
             return result;
         }
     }
     
     @Test
     public void testJdk() {
         // 创建被代理类
         IService service = new ServiceImpl();
         // 根据被代理类 创建 handler
         MyInvocationHandler handler = new MyInvocationHandler(service);
         // 生产代理类，可以看出，该代理类也实现了被代理类的接口
         // 传参是 被代理类的类加载器、接口、处理类
         IService serviceProxy = (IService) Proxy.newProxyInstance(service.getClass().getClassLoader(), service.getClass().getInterfaces(), handler);
         serviceProxy.add();
         serviceProxy.update();
     }
     ~~~

2. Cglib 动态代理

   * 代码：

     ~~~java
     // 被代理类
     public class Base {
         public void add() {
             System.out.println("Base.add()***");
         }
     }
     
     // 切面，必须实现 MethodInterceptor
     public class MyAdvice implements MethodInterceptor {
         @Override
         public Object intercept(Object object, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
             System.out.println("Before--------------------------");
             methodProxy.invokeSuper(object, args);
             System.out.println("After--------------------------");
             return null;
         }
     }
     
     // 代理类
     public class MyProxy {
         public static Base getInstance(MyAdvice advice) {
             // 需要创建一个 Enhancer 实例
             Enhancer enhancer = new Enhancer();
             // 设置 enhancer 的父类是被代理类
             enhancer.setSuperclass(Base.class);
             // 给 enhancer 切面
             enhancer.setCallback(advice);
             // enhancer 根据被 代理类 和 切面，生成符合的 子类
             Base base = (Base) enhancer.create();
             return base;
         }
     }
     
     @Test
     public void testCglib() {
         MyAdvice advice = new MyAdvice();
         Base base = MyProxy.getInstance(advice);
         base.add();
     }
     ~~~

3. jdk 动态代理 和 Cglib 动态代理的 区别

   * Jdk动态代理：利用**拦截器**（必须实现InvocationHandler）加上**反射机制**生成一个代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理

     Cglib动态代理：利用**ASM框架**，对代理对象类生成的class文件加载进来，通过**修改其字节码生成子类**来处理
   * 什么时候用cglib什么时候用jdk动态代理？

     1. 目标对象生成了接口 默认用JDK动态代理

     2. 如果目标对象没有实现接口，必须采用cglib库，Spring会自动在JDK动态代理和cglib之间转换
   * JDK动态代理**只能对实现了接口的类生成代理**，而不能针对类；Cglib是**针对类实现代理**，主要是对指定的类生成一个子类，覆盖其中的方法，并覆盖其中方法的增强，但是因为采用的是继承，**所以该类或方法不能是被final修饰的**，对于final类或方法，是无法继承的
   * Spring如何选择是用JDK还是cglib？

     1. 当bean实现接口时，会用JDK代理模式

     2. 当bean没有实现接口，用cglib实现

     3. 可以**强制使用cglib**（在spring配置中加入<aop:aspectj-autoproxy proxyt-target-class=”true”/>）

4. AoP 相关术语

   * Joint point（连接点）：程序中任意条代码（如方法调用、对类成员的访问等）都可以是一个连接点
   * Pointcut （切点）：将要插入 Advice 的连接点
   * Advice （增强）：在 切点 周围要做的操作
   * Aspect （切面）：切点 + 增强

5. advice 的类型

   | 类型                  | 描述                                                         |
   | --------------------- | ------------------------------------------------------------ |
   | before advice         | 在 Pointcut 前执行。无法阻止 Pointcut 的执行，除非发生了异常 |
   | after return advice   | 在 Pointcut 正常返回后执行                                   |
   | after throwing advice | 在 Pointcut 抛出异常后执行                                   |
   | after (final) advice  | 无论 pointcut 是正常退出还是发生了异常，都会执行             |
   | around advice         | 在 pointcut 前后都执行，最常用                               |
   | introduction          | 可以为原有的对象增加新的属性和方法                           |

6. AoP 是什么？

   * AoP **面向切面编程**，将代码中重复的部分抽取出来，在需要执行的时候使用动态代理技术，在不修改源码的基础上对方法进行增强
   * 实现方式有 jdk 动态代理 和 cglib 动态代理
   * 常用场景包括 权限认证、错误处理、日志、事务等

7. AoP相关注解

   ![image-20230307214111525](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230307214111525.png)

8. AoP 应用

   1. 导入两个依赖包：

      ~~~xml
      <!-- AoP 执行主逻辑 -->
      <dependency>
          <groupId>org.aspectj</groupId>
          <artifactId>aspectjweaver</artifactId>
      </dependency>
      
      <!-- 提供相关注解 -->
      <dependency>
          <groupId>org.aspectj</groupId>
          <artifactId>aspectjrt</artifactId>
      </dependency>
      ~~~

   2. 开启对 AspectJ 的支持

      ~~~java
      @SpringBootApplication
      // 添加注解
      @EnableAspectJAutoProxy
      public class Demo07Application {
      ...
      }
      ~~~

   3. 编写切面

      ~~~java
      @Slf4j
      // 声明为 切面
      @Aspect
      // 注册组件
      @Component
      public class DemoLogger {
          // 增强类型为 around，execution 作为 pointcut 切入点
          @Around("execution(* com.example.demo07.controller.*.*(..))")
          public Object around(ProceedingJoinPoint joinPoint) throws Throwable {}
      }
      ~~~



## Spring IOC 控制反转

1. BeanFactory 和 ApplicationContext

   * IOC 容器的实现主要是 BeanFactory接口和ApplicationContext接口：
     * BeanFactory：是Spring IOC的最底层设计，提供了先进的配置机制，使得任何类型的对象配置称为可能
     * ApplicationContext：继承自BeanFactory，实现了许多接口的扩展。
   * 区别：
     * BeanFactory的实现是**按需创建**，即第一次获取Bean才创建这个Bean
     * ApplicationContext会**一次性创建所有Bean**
     * ApplicationContext也提供了额外的功能，比如提供支持国际化、事件通知等
     * 通常总是使用ApplicationContext，很少使用BeanFactory。

2. bean 基于配置 实例化 的三种方式：

   1. 无参构造器：通过类的全限定名找到类的位置，通过无参构造函数创建对象

      ~~~xml
      <bean id="userInfo" class="com.tulun.Spring.IOC.ObjectUse.UserInfo"/>
      <!-- 如果不指定构造函数，会生成默认的无参构造函数
      	 如果指定有参构造，必须显性的指定一个无参构造函数，否则实例化对象就会抛出异常。->
      ~~~

   2. 静态工厂方法

      ~~~java
      // 首先，使用工厂的静态方法返回去一个对象
      public class ObjectFactory {
          //提供静态的方法来获取UserInfo对象
          public static UserInfo getBean() {
              return new UserInfo();
          }
      }
      ~~~

      ~~~xml
      <!--
      通过静态工厂获取对象
      class属性指向工厂类全限定名
      factory-method属性是工厂类中获取对象的静态方法
      -->
      <bean id="userInfo1" class="com.tulun.Spring.IOC.IOCDemo.ObjectFactory" factory-method="getBean"/>
      ~~~

   3. 普通工厂方法

      ~~~java
      // 通过工厂的非静态方法来得到一个对象
      public class ObjectFactory {
          //提供普通方法来获取UserInfo对象
          public UserInfo getUserInfo() {
              return new UserInfo();
          }
      }
      ~~~

      ~~~xml
      <!--
      通过普通工厂获取对象
      -->
      <!--创建工厂对象-->
      <bean id="factory" class="com.tulun.Spring.IOC.IOCDemo.ObjectFactory"/>
      <bean id="userInfo2" class="com.tulun.Spring.IOC.ObjectUse.UserInfo" factory-bean="factory" factory-method="getUserInfo"/>
      ~~~

3. 使用注解方式装配 bean

   * 使用 @Component 注解

   * 配置启动组件扫描注解

     ~~~xml
     <!--开启注解扫描，指定的包路径或者类名：扫描类、方法、属性上是否有注解-->
     <context:component-scan base-package="com.tulun.Spring.IOC.ObjectUse"/>
     ~~~

   * @Controller、@Service、@Repository 都是Component注解衍生出来，**功能是一样，可以互换**，为了区分被注解标注的类在不同的业务层，使逻辑更清晰。

4. 控制反转是一种通过描述（XML或注解）并通过第三方去生产或获取特定对象的方式。在Spring中实现控制反转的是IoC容器，其**实现方法是依赖注入（Dependency Injection,DI）**

5. 依赖注入 的实现方法

   1. **构造器注入**：检查被注入对象的构造方法，取得它所需要的依赖对象列表，进而为其注入相应的对象。

      优点：在对象构造完成后就处于**就绪状态**；创建时必须要指定构造方法中的全部参数，bean才能被创建，保证了对象创建出来之后，成员变量一定都有值

      缺点：当依赖对象较多时，构造方法的参数列表较长，构造方法**无法被继承，无法设置默认值**；必须要指定全部参数，否则无法创建，使用该方式改变了对象的创建过程

      ~~~java
      public class UserServiceImpl implements UserService {
          private UserDAO userDAO;
          
          //如果需要构造方法注入 则这里必须写上构造方法
          public UserServiceImpl(UserDAO userDAO) {
              super();
              this.userDAO = userDAO;
          }
       
          @Override
          public void add(User user) {
              // TODO Auto-generated method stub
              userDAO.save(user);
          }
          
      }
      ~~~

      

      ~~~xml
      <bean id="u" class="com.dao.UserDAOImpl">
      </bean>
      <bean id="userService" class="user.bean.UserServiceImpl">
          <!-- 构造注入 -->
          <constructor-arg>
              <ref bean="u"/>
          </constructor-arg>
      </bean>
      ~~~

      

   2. **setter 方法注入**：当前对象只需要为其依赖对象对应的属性添加setter方法，就可以通过setter方法将依赖对象注入到被依赖对象中

      优点：描述性上比构造器注入强，并且**可以被继承，允许设置默认值**

      缺点：**无法**在对象构造完成后马上进入就绪状态

      ~~~java
      public class UserServiceImpl implements UserService {
       
          private UserDAO userDAO;
          
          public UserDAO getUserDAO() {
              return userDAO;
          }
          
          //设值注入 为UserDAO的注入做准备
          public void setUserDAO(UserDAO userDAO) {
              this.userDAO = userDAO;
          }
          //实现负责业务逻辑的add方法
          @Override
          public void add(User user) {
              // TODO Auto-generated method stub
              userDAO.save(user);
          }
      }
      ~~~

      

      ~~~xml
      <!-- 配置数据访问类，实例名称为u -->
      <bean id="u" class="com.dao.UserDAOImpl">
      </bean>
      <bean id="user" class="com.bean.User">
      </bean>
      
      <!-- 设值注入 配置业务逻辑实现类，实例名称为userService-->
      <bean id="userService" class="com.bean.UserServiceImpl">
          <!-- 在这里实现注入，注入实例名称为u的实例到该实例的userDAO属性 -->
          <property name="userDAO">
              <ref bean="u" />
          </property>
      </bean>
      ~~~

      

   3. 接口注入：必须实现某个接口，接口提供方法来为其注入依赖对象

      缺点：使用少，因为它强制要求被注入对象实现不必要的接口，**侵入性强**。**Spring 不支持接口注入**



## Spring Bean

1. Bean 的作用域
   * **单例模式**（scope = "singleton"）：Spring 容器可以管理 singleton 作用域 Bean 的**全部生命周期**，在此作用域下，Spring 能够精确地知道该 **Bean 何时被创建，何时初始化完成，以及何时被销毁。**
   * **原型模式**（scope="prototype"）：**Spring 在创建好后交给使用者之后就不再管理后续的生命周期**。**只负责创建**。每次客户端请求 prototype 作用域的 Bean 时，**都会创建一个新的实例**
   * **request**：每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在**当前 HTTP request 内有效**
   * **session**：每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在**当前 HTTP session 内有效**
   * **global-session**：全局 session 作用域，仅仅在基于 Portlet 的 web 应用中才有意义，**Spring5 已经没有了**
   * **application**
   * 了解 Spring 生命周期的意义就在于，可以**利用 Bean 在其存活期间的指定时刻完成一些相关操作**。这种时刻可能有很多，但一般情况下，会在 Bean **被初始化后和被销毁前**执行一些相关操作
2. **Bean 的生命周期：实例化 -> 属性赋值 -> 初始化 -> 销毁**
   * **@PostConsturct**：初始化方法
   * **@PreDestory**：销毁方法
3. **Spring如何解决循环依赖问题**
   * 循环依赖问题在Spring中主要有三种情况：
     1. 通过**构造方法**进行**依赖注入时**产生的循环依赖问题
     2. 通过**setter方法**进行**依赖注入**且是在**多例（prototype）模式**下产生的循环依赖问题。
     3. 通过**setter方法**进行**依赖注入**且是在**单例模式下**产生的循环依赖问题
   * 在Spring中，只有第（3）种方式的循环依赖问题被解决了，其他两种方式在遇到循环依赖问题时都会产生异常：
     1. 第一种**构造方法注入的情况下**，在**new对象的时候就会堵塞住**了，其实也就是”先有鸡还是先有蛋“的历史难题。
     2. 第二种**setter方法（多例）的情况下**，**每一次getBean()时，都会产生一个新的Bean**，如此反复下去就会有无穷无尽的Bean产生了，最终就会导致OOM问题的出现。
   * Spring在单例模式下的setter方法依赖注入引起的循环依赖问题，主要是**通过二级缓存和三级缓存**来解决的，其中三级缓存是主要功臣。解决的核心原理就是：**在对象实例化之后，依赖注入之前，Spring提前暴露的Bean实例的引用在第三级缓存中进行存储**
4. Spring 自动装配：
   1. 基于xml配置中 5 种自动装配：
      * no ：**默认的方式**是**不进行自动装配的**，通过手工设置ref属性来进行装配bean。
      * byName：通过**bean的名称**进行自动装配，如果一个bean的 property 与另一bean 的name 相同，就进行自动装配
      * byType：通过参数的**数据类型**进行自动装配
      * constructor：利用**构造函数进行装配**，并且构造函数的**参数通过byType进行装配**
      * autodetect：**自动探测**，如果有构造方法，通过 construct的方式自动装配，否则使用 byType的方式自动装配
   2. 基于注解的自动装配：
      * @Autowired
        * 在使用@Autowired时，首先在容器中查询**对应类型的bean**：如果**查询结果刚好为一个**，就将该bean装配给@Autowired指定的数据；**如果查询的结果不止一个**，那么@Autowired会**根据名称**来查找；如果上述**查找的结果为空**，那么会抛出异常。解决方法时，使用**required=false**
        * @Autowired默认是**按照类型**装配注入的，默认情况下它**要求依赖对象必须存在**（可以设置它required属性为false）。
        * @Autowired如果需要**按照名称匹配**需要和**@Qualifier**一起使用
      * @Resource
        * 默认是**按照名称**来装配注入的，只有**当找不到与名称匹配的bean才会按照类型来装配注入。**



## Spring 事务

1. Spring事务的**本质其实就是数据库对事务的支持**，没有数据库的事务支持，spring是无法提供事务功能的。**Spring只提供统一事务管理接口，具体实现都是由各数据库自己实现**

2. **Spring事务的种类**：

   * 编程式事务管理：使用TransactionTemplate。

   * 声明式事务管理：建立在**AOP**之上的。其本质是通过AOP功能，**对方法前后进行拦截，将事务处理的功能编织到拦截的方法中**，也就是在目标方法开始之前启动一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

   * 声明式事务的优点在于：不需要在业务逻辑代码种掺杂事务管理代码，只需在**配置文件中做相关的事务规则声明**或通过**@Transactional**注解的方式，对业务代码污染小

     缺点在于：**最细粒度只能作用到方法级别**，无法做到像编程式事务那样可以作用到代码块级别。

3. 事务传播机制：

   * 事务传播机制实际上是**使用简单的ThreadLocal实现的**，所以，如果调用的方法是在新线程调用的，事务传播实际上是会失效的。
   * 类型：
     * required：（**默认传播行为**）如果当前没有事务，就创建一个新事务；如果当前存在事务，就加入该事务。（**有就加入，没有就自己建**）
     * required_new：无论当前存不存在事务，都创建新事务进行执行(**管你有没有，我都新建**，**new 一个**)
     * supports：如果当前存在事务，就加入该事务；如果当前不存在事务，就以非事务执行（**有就加入吧，没有就算了**）
     * Not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。（**才不用事务那种东西呢**）
     * nested：如果当前存在事务，则在**嵌套事务**内执行；如果当前没有事务，则**按REQUIRED属性执行**
     * mandatory：如果当前存在事务，就加入该事务；如果当前不存在事务，就抛出异常（**有就吃，没有就——领导，我没捞着饭呢**）
     * never：以非事务方式执行，如果当前存在事务，则抛出异常（**不能有事务，有我就骂街了**）
   * 隔离级别：
     * default：默认，使用数据库的默认事务隔离级别
     * uncommitted：读未提交
     * committed：读已提交
     * repeatable_read：可重复读
     * serializable：序列化，所有事务逐个依次执行

   

## Spring MVC

1. 常用注解：
   * @RequestMapping：用于映射web请求，包括访问路径和参数
   * @ResponseBody：支持将返回值放到response内，而不是一个页面，通常用户返回json数据
   * @RequestBody：允许request的参数在request体中，而不是在直接连接的地址后面（放在参数前）
   * @PathVariable：**用于接收路径参数**，比如@RequestMapping(“/hello/{name}”)声明的路径，**将注解放在参数前，即可获取该值**，通常作为Restful的接口实现方法
   * **@RestController**：相当于**@Controller和@ResponseBody**的组合，注解在类上，意味着，该Controller的所有方法都默认加上了@ResponseBody



## SpringBoot

1. @SpringBootApplication 注解：包含了以下3个注解：

   * @SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能
   * @EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项， 例
     如： java 如关闭数据源自动配置功能： @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class })
   * @ComponentScan：Spring组件扫描
   * **EnableAutoConfiguration：通过源码可以知道，该注解使用Import引入了AutoConfigurationImportSelector类，而AutoConfigurationImportSelector类通过SpringFactortisLoader加载了所有jar包的MATE-INF文件夹下面的spring.factories文件，spring.factories包含了所有需要装配的XXXConfiguration类的全限定名。XXXConfiguration类包含了实例化该类需要的信息，比如说如果这是个数据源Configuration类，那么就应该有数据库驱动、用户名、密码等等信息**
     SpringBoot 支持哪些日志框架

   * Java Util Logging, Log4j2, **Lockback（默认日志框架）**
   * 不管是那种日志框架他都支持将配置文件输出到控制台或者文件中。

2. SpringBoot Starter 工作原理

   * Spring Boot 在启动的时候会干这几件事情：

     1.  Spring Boot 在启动时会去依赖的 Starter 包中寻找 resources/META-INF/**spring.factories** 文件，然后根据文件中配置的 Jar 包去扫描项目所依赖的 Jar 包。

     2.  根据 spring.factories 配置加载 AutoConfigure 类

     3.  根据 @Conditional 注解的条件，进行自动配置并将 Bean 注入 Spring Context

   * 自制Starter 

     1. 在资源目录`resources`下建立一个 **META-INF** 目录，并在该目录下建立一个 **spring.factories** 文件

     2. 在 **spring.factories** 文件中，指定你要被IOC容器管理的配置类的`全额限定名`就可以了

        ~~~properties
        org.springframework.boot.autoconfigure.EnableAutoConfiguration = \com.nrsc.starter.HelloAutoConfiguration
        ~~~

3. SpringBoot 事务的注解：@EnableTransactionManagement，然后在Service方法上添加注解Transactional便可

4. SpringBoot 配置加载顺序：

   1. properties文件
   2. YAML文件
   3. 系统环境变量
   4. 命令行参数

5. yaml 配置的优势

   * 配置有序
   * 简洁明了，支持数组，数组中的元素可以是基本数据类型也可以是对象
   * 相对于 properties 配置文件，yaml 缺点在于，不支持 @PropertySource 注解导入自定义的 yaml 配置

6.  Spring Boot 是否可以使用 XML 配置

   * Spring Boot 推荐使用 Java 配置而非 XML 配置，但是 Spring Boot 中也可以使用 XML 配置，通
     过 @ImportResource 注解可以引入一个 XML 配置 

7. SpringBoot 解决跨域问题

   1. 方案一：添加CORS 配置类

      ~~~java
      @Configuration
      public class CorsConfig {
      
          @Bean
          public WebMvcConfigurer corsConfigurer() {
              return new WebMvcConfigurerAdapter() {
                  @Override
                  public void addCorsMappings(CorsRegistry registry) {
                      registry.addMapping("/**")
                              .allowedOrigins("*")
                              .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                              .allowedHeaders("*")
                              .allowCredentials(true)
                              .maxAge(360);
                  }
              };
          }
      }
      ~~~

   2. 方案二：@CrossOrigin注解

      ~~~java
      @RestController
      @RequestMapping("/api")
      @CrossOrigin(origins = "*", methods = {RequestMethod.GET, RequestMethod.POST, RequestMethod.PUT, RequestMethod.DELETE, RequestMethod.OPTIONS})
      public class ApiController {
      
          @GetMapping("/users")
          public List<User> getUsers() {
              // ...
          }
      }
      ~~~

8. 



## Mysql

1. 去重

   * 方案一：使用distinct

     ~~~mysql
     -- 放在select 的后面，对后面所有字段的值统一进行去重
     -- distinct 通常效率较低，不适合用来展示去重后具体的值，一般与 count 配合用来计算条数
     select distinct task_id from task;
     ~~~

   * 方案二：group by

     ~~~mysql
     -- 列出task_id 的所有唯一值（去重后的记录，null也是值）
     select task_id from task group by task_id;
     ~~~

2. 数据类型

   * 数值类型
     * TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT，分别表示 1 字节、2 字节、3 字节、4 字节、8 字节的**整数类型**
     * 任何整数类型都可以加上 UNSIGNED 属性，表示无符号整数，0或正整数
     * 何整数类型都可以指定长度，但它**不会限制数据的合法长度，仅仅限制了显示长度**
   * 字符串类型
     *  VARCHAR、CHAR、TEXT、BLOB
     * **VARCHAR(n) 和 CHAR(n) 中的 n 并不代表字节个数，而是代表字符的个数**
   * 日期和时间类型
     * DATETIME、DATE 和 TIMESTAMP
     * 尽量使用 TIMESTAMP，空间效率高于 DATETIME

3. char 和 varchar 的区别

   1. char 定长，varchar 可以变长
   2. char 根据**声明的字符串长度**分配空间，并会使用空格对字符串右边进行**尾部填充**。所以在检索 CHAR 类型数据时尾部空格会被删除，如保存的是字符串 **`'char '`，但最后查询到的是 `'char'`**。又因为长度固定，所以**存储效率高于 VARCHAR 类型**。
   3. VARCHAR 在 MySQL 5.0 之后长度支持到 65535 字节，**但会在数据开头使用额外 1~2 个字节存储字符串长度（列长度小于 255 字节时使用 1 字节表示，否则 2 字节），在结尾使用 1 字节表示字符串结束。**
   4. 在存储方式上，CHAR 对英文字符（ASCII）占用 1 字节，对一个汉字使用用 2 字节。而 VARCHAR 对每个字符均使用 2 字节

4. char 和 varchar 如何选择？

   1. **对于经常变更的数据来说**，CHAR 比 VARCHAR更好，因为 **CHAR 不容易产生碎片**。
   2. 对于非常短的列或固定长度的数据（如 MD5），CHAR 比 VARCHAR 在存储空间上更有效率。
   3. 使用时要注意只分配需要的空间，更长的列排序时会消耗更多内存。
   4. 尽量避免使用 TEXT/BLOB 类型，查询时会使用临时表，导致严重的性能开销

5. char 、varchar、text 的区别

   * 长度方面：char 范围 0 ~ 255，varchar 最长64k，但 varchar 一般情况下存储都够用。大文本考虑 text ，最大4G
   * 效率方面：Char > Varchar > Text，但是如果使用的是 Innodb 引擎的话，推荐使用 Varchar 代替 Char。
   * 默认值：Char 和 Varchar 支持设置默认值，而 Text 不能指定默认值

6. 三大范式

   1. 第一范式：字段是不可分割的最小单元，也就是说表的每个列具有原子性
   2. 第二范式：在第一范式基础上，如果**表是单主键，那么主键以外的列必须完全依赖于主键**；如果表是**复合主键**，那么复合主键以外的列必须完全依赖于复合主键，不能仅依赖复合主键的一部分
   3. 第三范式：在第二范式的基础上，非主键列之间不能相关依赖。

7. 索引

   1. 索引种类
      * 物理结构上分：
        * **聚簇索引**（索引的键值的逻辑顺序与表中相应行的物理顺序一致，即每张表只能有一个聚簇索引，也就是我们常说的**主键索引**）
        * **非聚簇索引**（逻辑顺序则与数据行的物理顺序不一致）
      * 应用上分：
        * **普通索引**：基本索引类型，允许在定义索引的列中**插入重复值和空值**，**纯粹为了提高查询效率**。ALTER TABLE table_name ADD INDEX index_name (column)
        * **唯一索引**：索引列中的**值必须是唯一**的，但是**允许为空值**。通过 `ALTER TABLE table_name ADD UNIQUE index_name (column)` 创建
        * **主键索引**：特殊的唯一索引，也成聚簇索引，不允许有空值，并由数据库帮我们自动创建
        * **组合索引**：组合表中多个字段创建的索引，遵守**最左前缀匹配规则**
        * **全文索引**：只有在 **MyISAM 引擎**上才能使用，同时只支持 CHAR、VARCHAR、TEXT 类型字段上使用。
   
   2. 索引优缺点
      * 优点：
        * 通过创建唯一索引，可以保证数据库表中每一行数据的唯一性
        * 加快数据的检索速度
        * 加速表与表之间的连接
        * 在使用分组和排序子句进行数据检索时，同样可以显著减少查询中分组和排序的时间
      * 缺点：
        * **创建和维护索引需要耗费时间**，这种时间随着数据量的增加而增加，这样就降低了数据的维护速度
        * **索引需要占物理空间**，除了数据表占数据空间之外，每一个索引还要占一定的物理空间。如果要建立聚簇索引，那么需要的空间就会更大
   
   3. 索引设计原则
      * **选择唯一索引**：唯一性索引的值是唯一的，可以更快速的通过该索引来确定某条记录
      * **为常作为查询条件的字段建立索引**
      * **为经常需要排序、分组和联合操作的字段建立索引**：经常需要 ORDER BY、GROUP BY、DISTINCT 和 UNION 等操作的字段，排序操作会浪费很多时间。如果为其建立索引，可以有效地避免排序操作
      * **限制索引的数目**：每个索引都需要占⽤用磁盘空间，索引越多，需要的磁盘空间就越大，修改表时，对索引的重构和更新很麻烦
      * **小表不建议索引（如数量级在百万以内）**
      * **尽量使用数据量少的索引**：如果索引的值很长，那么查询的速度会受到影响。此时**尽量使用前缀索引**
      * **删除不再使用或者很少使用的索引**
   
   4. 索引的数据结构
      * 索引的数据结构和具体存储引擎的实现有关，MySQL 中常用的是 **Hash** 和 **B+ 树**索引
      * Hash 索引
        * 底层就是 **Hash 表**，进行**查询时调用 Hash 函数获取到相应的键值（对应地址）**，然后回表查询获得实际数据
        * Hash 进行**等值查询更快**，但**无法进行范围查询**。因为经过 Hash 函数建立索引之后，**索引的顺序与原顺序无法保持一致，故不能支持范围查询**。同理，也不支持使用索引进行排序
        * Hash **不支持模糊查询**以及多列索引的**最左前缀匹配**,因为 Hash 函数的值不可预测，如 AA 和 AB 的算出的值没有相关性
        * Hash 任何时候都避免不了回表查询数据
        * 虽然在等值上**查询效率高，但性能不稳定**，因为当某个键值存在大量重复时，产生 Hash 碰撞，此时查询效率反而可能降低
      * B+ 树
        * 底层实现原理是**多路平衡查找树**，对于**每一次的查询都是从根节点出发，查询到叶子节点方可以获得所查键值**，最后查询判断是否需要回表查询
        * 本质是一棵查找树，自然支持范围查询和排序
        * 在符合某些条件（聚簇索引、覆盖索引等）时候可以只通过索引完成查询，**不需要回表**
        * **查询效率比较稳定**，因为每次查询都是从根节点到叶子节点，且为树的高度
      * **回表**
        * 如果 select * from T where ID = 500，即主键查询方式，则只需要搜索 ID 这颗 B+ 树
        * 如果 select * from T where k = 5，即普通索引查询方式，则需要**先搜索k索引树，得到ID的值为500，再到ID索引树搜索一次**。这个过程称为回表。
   
   5. **为何使用 B+ 树而非二叉查找树做索引**
      * 二叉树的查找效率为 O(logn)，**当树过高时，查找效率会下降**。另外由于我们的索引文件并不小，所以是存储在磁盘上的。因而这种**树高会随数据量增多急剧增加**，每次更新数据**又需要通过左旋和右旋维护平衡的二叉树**，不太适合用于存储在磁盘上的索引文件
   
   6. **为何使用 B+ 树而非 B 树做索引**
      * **B+树查询效率更稳定**：B树的非叶子结点和叶子结点都存储数据，因此查询数据时，时间复杂度最好为 O(1)，最坏为 O(log n)。而 B+ 树只在**叶子结点存储数据**，**非叶子结点存储关键字**，且不同非叶子结点的关键字可能重复，因此查询数据时，**时间复杂度固定为 O(log n)**。
      * **B+ 树减少了 IO 次数**：B+ 树的非叶子结点只存关键字不存数据，因而**单个页可以存储更多的关键字**，即一次性读入内存的需要查找的关键字也就越多，磁盘的随机 I/O 读取次数相对就减少了
      * **B+ 树更加适合范围查找**：B+ 树叶子结点之间用链表有序连接，所以**扫描全部数据只需扫描一遍叶子结点**，利于扫库和范围查询；B 树由于非叶子结点也存数据，所以**只能通过中序遍历按序来扫**。也就是说，**对于范围查询和有序遍历而言**，B+ 树的效率更高
   
   7. 最左匹配原则
      * 最左匹配原则都是针对**联合索引**来说的。最左优先，以最左边为起点任何连续的索引都能匹配上。同时**遇到范围查询**（>、<、between、like）**就会停止匹配**
   
   8. **存储引擎**
      * InnoDB 和 MyISAM 的区别
        1. InnoDB 支持事务，而 MyISAM 不支持
        2. InnoDB 支持外键，而 MyISAM 不支持。因此将一个含有外键的 InnoDB 表 转为 MyISAM 表会失败
        3. InnoDB 和 MyISAM **均支持 B+ Tree 数据结构的索引**。但 **InnoDB 是聚集索引**，而 MyISAM 是非聚集索引
        4. **InnoDB 不保存表中数据行数**，执行 `select count(*) from table` 时需要全表扫描。而 MyISAM 用一个变量记录了整个表的行数，速度相当快（注意不能有 WHERE 子句）。**那为什么 InnoDB 没有使用这样的变量呢**？**因为InnoDB的事务特性**，在同一时刻表中的行数对于不同的事务而言是不一样的
        5. InnoDB **支持表、行（默认）级锁**，而 MyISAM 支持表级锁。InnoDB 的行锁是基于索引实现的，而不是物理行记录上。即访问如果没有命中索引，则也无法使用行锁，将要退化为表锁
        6. **InnoDB 必须有唯一索引（如主键**），如果没有指定，就会自动寻找或生产一个隐藏列 Row_id 来充当默认主键，而 Myisam 可以没有主键
      * **如何选择存储引擎**
        * 默认使用 InnoDB，MyISAM 适用以插入为主的程序，比如博客系统、新闻门户
      * **InnoDB 为何推荐使用自增主键**
        * **自增 ID 可以保证每次插入时 B+ 树索引是从右边扩展的**，因此相比自定义 ID （如 UUID）可以避免 B+ 树的频繁合并和分裂。如果使用字符串主键和随机主键，会使得数据随机插入，效率比较差
      * InnoDB 四大特性
        * 插入缓冲
        * 二次写
        * 自适应哈希索引
        * 预读
   
   9. 事务
      1. 事务的四大特性（ACID）
         * 原子性（Atomicity）：事务是最小的执行单位，要么全部完成，要么完全不起作用
         * 一致性（Consistency）：事务执行前后，数据保持一致，多个事务对同一个数据读取的结果是相同的
         * 隔离性（Isolation）：并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的
         * 持久性（durability）：一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响
      2. 事务的并发问题
         * **脏读**：**一个事务读取到另一个事务尚未提交的数据**。事务 A 读取事务 B 更新的数据，然后 B 回滚操作，那么 A 读取到的数据是脏数据。
         * **不可重复读**：**一个事务中两次读取的数据的内容不一致**。事务 A 多次读取同一数据，事务 B 在事务 A 多次读取的过程中，对数据作了更新并提交，导致事务 A 多次读取同一数据时，结果 不一致
         * **幻读：一个事务中两次读取的数据量不一致。** 系统管理员 A 将数据库中所有学生的成绩从具体分数改为 ABCDE 等级，但是系统管理员 B 就在这个时候插入了一条具体分数的记录，当系统管理员 A 改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读
      3. **不可重复读侧重于修改，幻读侧重于新增或删除**。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表。
      4. 事务的隔离级别
         * 读未提交：三个问题都解决不了
         * 读提交：解决脏读
         * 可重复读：解决脏读和不可重复读
         * 串行化：都能解决
      5. MySQL 默认 隔离级别：可重复读；Oracle 只支持 串行化 和 读已提交，默认是 读已提交
      6. start transaction标识事务开始，commit提交事务
   
   10. MySQL 服务器逻辑架构
       * MySQL服务器逻辑架构从上往下可以分为三层：
         1. 第一层：处理客户端连接、授权认证等
         
         2. 第二层：服务器层，负责查询语句的解析、优化、缓存以及内置函数的实现、存储过程等
         
         3. 第三层：存储引擎，负责MySQL中数据的存储和提取。**MySQL中服务器层不管理事务，事务是由存储引擎实现的**
         
            ![image-20230318105321591](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230318105321591.png)
   
   11. **ACID 实现原理**
   
       * 原子性（undo log 实现）
         * InnoDB存储引擎提供了两种事务日志：redo log(重做日志)和undo log(回滚日志)。其中**redo log用于保证事务持久性；undo log则是事务原子性和隔离性实现的基础**
         * 实现原子性的关键，是当事务回滚时能够撤销所有已经成功执行的sql语句。**InnoDB实现回滚，靠的是undo log：当事务对数据库进行修改时，InnoDB会生成对应的undo log；如果事务执行失败或调用了rollback，导致事务需要回滚，便可以利用undo log中的信息将数据回滚到修改之前的样子**
         * undo log 属于逻辑日志，它记录的是sql执行相关的信息。当发生回滚时，**InnoDB会根据undo log的内容做与之前相反的工作**：对于每个insert，回滚时会执行delete；对于每个delete，回滚时会执行insert；对于每个update，回滚时会执行一个相反的update，把数据改回去。
         
           以update操作为例：当事务执行update时，其**生成的undo log中会包含被修改行的主键(以便知道修改了哪些行)、修改了哪些列、这些列在修改前后的值等信息**，回滚时便可以使用这些信息将数据还原到update之前的状态
         
       * 持久性 （redo log 实现）
   
         * InnoDB作为MySQL的存储引擎，数据是存放在磁盘中的，但**如果每次读写数据都需要磁盘IO，效率会很低**。为此，**InnoDB提供了缓存(Buffer Pool)**，Buffer Pool中包含了磁盘中部分数据页的映射，作为访问数据库的缓冲：**当从数据库读取数据时，会首先从Buffer Pool中读取，如果Buffer Pool中没有，则从磁盘读取后放入Buffer Pool**；**当向数据库写入数据时，会首先写入Buffer Pool，Buffer Pool中修改的数据会定期刷新到磁盘中**（这一过程称为**刷脏**）。
   
           Buffer Pool的使用大大提高了读写数据的效率，**但是也带了新的问题**：**如果MySQL宕机，而此时Buffer Pool中修改的数据还没有刷新到磁盘，就会导致数据的丢失，事务的持久性无法保证**。
   
           于是，redo log被引入来解决这个问题：当数据修改时，除了修改Buffer Pool中的数据，还会在redo log记录这次操作；当事务提交时，会调用fsync接口对redo log进行**刷盘**。如果MySQL宕机，重启时可以读取redo log中的数据，对数据库进行恢复。redo log采用的是**WAL**（Write-ahead logging，预写式日志），**所有修改先写入日志，再更新到Buffer Pool，保证了数据不会因MySQL宕机而丢失，从而满足了持久性要求**。
   
           既然redo log也需要在事务提交时将日志写入磁盘，为什么它比直接将Buffer Pool中修改的数据写入磁盘(即刷脏)要快呢？主要有以下两方面的原因：
   
           1. 刷脏是随机IO，因为每次修改的数据位置随机，但写redo log是追加操作，属于顺序IO。
   
           2. 刷脏是以数据页（Page）为单位的，MySQL默认页大小是16KB，一个Page上一个小修改都要整页写入；而redo log中只包含真正需要写入的部分，无效IO大大减少
   
         * redo log 和 binlog 区别
   
           * 作用不同：**redo log 是用于 crash recovery（崩溃恢复**）的，保证MySQ**L宕机也不会影响持久性**；**binlog** 是用于 point-in-time recovery的，**保证服务器可以基于时间点恢复数据**，此外binlog还**用于主从复制**
           * 层次不同：redo log 是InnoDB存储引擎实现的，binlog是MySQL的服务器层实现的，同时支持InnoDB和其他存储引擎
           * 内容不同：redo log 是物理日志，内容基于磁盘的Page；binlog的内容是二进制的，根据binlog_format参数的不同，可能基于sql语句、基于数据本身或者二者的混合
           * 写入时机不同：binlog在事务提交时写入；redo log的写入时机相对多元。
             * 前面曾提到：当事务提交时会调用fsync对redo log进行刷盘；**这是默认情况下的策略**，**修改**innodb_flush_log_at_trx_commit参数可以改变该策略，但**事务的持久性将无法保证**。
             * 除了事务提交时，还有其他刷盘时机：如master thread每秒刷盘一次redo log等，这样的好处是不一定要等到commit时刷盘，commit速度大大加快
   
       * 隔离性
   
         * **严格的隔离性，对应了事务隔离级别中的Serializable** (可串行化)，但实际应用中出于性能方面的考虑很少会使用可串行化
         * 隔离性的探讨，分为两个方面:
           * (一个事务)写操作对(另一个事务)写操作的影响：**锁机制**保证隔离性
             * 锁机制的基本原理可以概括为：**事务在修改数据之前，需要先获得相应的锁**；获得锁之后，事务便可以修改数据；该事务操作期间，这部分数据是锁定的，**其他事务如果需要修改数据，需要等待当前事务提交或回滚后释放锁**。
             * **按照粒度，锁可以分为表锁、行锁以及其他位于二者之间的锁**。表锁在操作数据时会锁定整张表，并发性能较差；行锁则只锁定需要操作的数据，并发性能好。但是**由于加锁本身需要消耗资源**(获得锁、检查锁、释放锁等都需要消耗资源)，因此在锁定**数据较多情况下使用表锁可以节省大量资源**。MySQL中不同的存储引擎支持的锁是不一样的，例如MyIsam只支持表锁，而InnoDB同时支持表锁和行锁，且**出于性能考虑，绝大多数情况下使用的都是行锁**
           * (一个事务)写操作对(另一个事务)读操作的影响：**MVCC**保证隔离性
             * InnoDB默认隔离级别是 可重复读（Repeatable Read，RR），RR 无法避免幻读问题，但是InnoDB实现的RR可以避免幻读，使用MVCC（Multi-Version Concurrency Control，即**多版本的并发控制协议**）
             * MVCC最大的优点是读不加锁，因此 读写不冲突，并发性能好。InnoDB实现MVCC，多个版本的数据可以共存，主要基于以下技术及数据结构：
               1. 隐藏列：InnoDB中每行数据都有隐藏列，隐藏列中包含了本行数据的事务id、指向undo log的指针等
               2. 基于undo log的版本链：前面说到每行数据的隐藏列中包含了指向undo log的指针，而每条undo log也会指向更早版本的undo log，从而形成一条版本链
               3. ReadView：通过隐藏列和版本链，MySQL可以将数据恢复到指定版本；但是具体要恢复到哪个版本，则需要根据ReadView来确定。所谓ReadView，是指事务（记做事务A）在某一时刻给整个事务系统（trx_sys）打快照，之后再进行读操作时，会将读取到的数据中的事务id与trx_sys快照比较，从而判断数据对该ReadView是否可见，即对事务A是否可见
                  * trx_sys中的主要内容，以及判断可见性的方法如下：
                    - low_limit_id：表示生成ReadView时系统中应该分配给下一个事务的id。如果数据的事务id大于等于low_limit_id，则对该ReadView不可见。
                    - up_limit_id：表示生成ReadView时当前系统中活跃的读写事务中最小的事务id。如果数据的事务id小于up_limit_id，则对该ReadView可见。
                    - rw_trx_ids：表示生成ReadView时当前系统中活跃的读写事务的事务id列表。如果数据的事务id在low_limit_id和up_limit_id之间，则需要判断事务id是否在rw_trx_ids中：如果在，说明生成ReadView时事务仍在活跃中，因此数据对ReadView不可见；如果不在，说明生成ReadView时事务已经提交了，因此数据对ReadView可见
           * **RR虽然避免了幻读问题，但是毕竟不是Serializable，不能保证完全的隔离**
   
       * 一致性
   
         * 一致性是事务追求的最终目标：前面提到的原子性、持久性和隔离性，都是为了保证数据库状态的一致性。此外，除了数据库层面的保障，一致性的实现也需要应用层面进行保障
         * 实现一致性的措施包括：
           - **保证原子性、持久性和隔离性**，如果这些特性无法保证，事务的一致性也无法保证
           - **数据库本身提供保障**，例如不允许向整形列插入字符串值、字符串长度不能超过列的限制等
           - **应用层面进行保障**，例如如果转账操作只扣除转账者的余额，而没有增加接收者的余额，无论数据库实现的多么完美，也无法保证状态的一致
   
   12. MySQL 进阶
   
       1. 视图
   
          * 视图是一张虚拟表
   
          * 视图的创建：
   
            ~~~mysql
            create view 视图名（列名,...）as select 语句
            
            -- 示例
            -- create or replace view 意思是不存在就创建，存在就替换
            -- 可以把视图中的字段别名移到查询语句的后面
             CREATE OR REPLACE VIEW user_order_data
             AS
             SELECT
             u.username as username, 
             o.number as number , 
             tm.name as name , 
             tm.price as price , 
             od.items_num as items_num
             FROM
             (
             (orders as o INNER JOIN orderdetail as od ON o.id = od.orders_id ) 
             INNER JOIN items as tm ON od.items_id = tm.id 
             )
             INNER JOIN user as u ON o.user_id = u.id
             
             -- 删除视图
             drop view 视图名称
            ~~~
   
          * 创建视图没有个数限制，但是**如果一张视图嵌套或关联的表过多，会引发性能问题**
   
          * 如果创建视图的 sql 语句中存在 where 条件语句，使用视图时的语句中也有 where 语句，这两个 where 会自动组合
   
          * 如果视图中有 order by 语句，使用视图的语句中也有 order by，那么视图中的 order by 会被覆盖
   
          * 视图不能使用索引、触发器
   
          * 视图可以和普通表联结
   
          * 使用视图对数据进行更新（增删改），这些操作都直接作用到普通表中，但并非所有的视图都可以进行更新操作，如视图中存在分组、联结、聚合函数等
   
       2. 存储过程
   
          * 简单的理解**存储过程就是数据库中保存的一系列SQL命令的集合**，也就是说通过存储过程就可以编写流程语句，如循环操作语句等
   
          * 创建
   
            ~~~mysql
            -- 语法
            -- 参数的种类分3种，分别是IN、OUT、INOUT，其中IN为输入参数类型，OUT为输出参数类型，而INOUT既是输入类型又是输出类型
            CREATE PROCEDURE 存储过程名称(  参数的种类1 参数1 数据类型1
                                        [,参数的种类2 参数2 数据类型2])
                                        BEGIN
                                            处理内容
                                        END
                                        
            -- 示例
            -- 修改分隔符
            delimiter //
            -- 创建存储过程
            create procedure sp_item_price(out plow decimal(8,2), 
                                           out phigh decimal(8,2), 
                                           out pavg decimal(8,2))
                                           begin
                select min(price) into plow from items;
                select max(price) into phigh from items;
                select avg(price) into pavg from items;
                end;
                //
            delimiter ; -- 恢复分隔符
            
            -- 调用存储过程
            call sp_item_price(...)
            -- 删除
            drop procedure [if exists] sp_item_price;
            -- 查看状态
            show procedure status [like 'pattern']
            -- 查看创建语句
            show create procedure sp_item_price;
            
            -- if 条件语句
            if ... then ... else if ... then ... else ... end if;
            -- 多分支条件语句
            case ... when .. then ... else ... end case
            -- 循环控制语句
            repeat ... until ... end repeat;
            -- while
            while ... do ... end while;
            -- 定义变量
            declear 变量名，数据类型，[default value 默认值]
            ~~~
   
       3. 函数
   
          * 创建
   
            ~~~mysql
            CREATE FUNCTION 函数([参数名 数据类型[,….]]) RETURNS 返回类型 
            　　BEGIN
            　　　  SQL语句.....
            　　　　RETURN (返回的数据)
            　　END
            　　
            -- 示例
            create function fn_factorial(num int) returns int
            	begin
            	...
            	return result;
            	end
            	
            -- 调用，需要 select 关键字
            select fn_factorial(0)
            ~~~
   
          * 存储过程 和 函数 的区别
   
            1. 存储过程可以有多个 in，out，inout 参数，而函数只有输入参数类型，而且不能带 in
            2. 存储过程实现的功能复杂一些，函数的单一功能性（针对性）更强
            3. 存储过程可以返回多个值，函数只有一个返回值
            4. 存储过程一般独立执行，函数可以作为其他sql语句的组成部分出现
            5. 存储过程可以调用函数，函数不能调用存储过程
   
       4. 触发器
   
          * 可以简单理解一种特殊的存储过程。唯一不同的是我们只需要定义触发器，而不用手动调用触发器
   
          * 创建
   
            ~~~mysql
            CREATE TRIGGER trigger_name trigger_time
            trigger_event ON tbl_name
            FOR EACH ROW
            BEGIN
            trigger_stmt
            END
            
            -- trigger_name: 触发器名称
            -- trigger_time: 触发时机，取值为 before 或 after
            -- trigger_event: 触发事件，取值 insert、update、delete
            -- tbl_name: 在哪张表上建立触发器
            -- trigger_stmt: 触发器程序体
            -- FOR EACH ROW: 固定写法，指明触发器以行作为执行单位
            ~~~
   
   13. SQL
   
       * where 和 having 的区别
   
         * where 在分组前过滤数据，不能包含聚合函数
         * having 作用是筛选满足条件的分组，即分组之后过滤数据，条件中经常包含聚合函数，可以过滤特定的组，也可以使用多个分组标准进行分组
         * 条件的过滤顺序：on->join->where->group by->having
   
       * SQL 执行顺序
   
         ![image-20230318134021899](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230318134021899.png)
   
       * in 和 exist 的区别
   
         * in
   
           * in 在查询的时候，首先查询子查询的表，然后将内表与外表做一个笛卡尔积，然后按照条件进行筛选。**适合内表比外表数据少的情况**
   
           * 具体sql 示例
   
             ![image-20230318134816624](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230318134816624.png)
   
             ~~~mysql
             -- user 表 比 order 表大
             select * from user where user.id in (select order.user_id from order)
             ~~~
   
             1. 先执行子查询，得到结果：
   
                ![image-20230318134848897](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230318134848897.png)
   
             2. 将结果与原有的user 表做笛卡尔积，结果如下：
   
                ![image-20230318134926533](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230318134926533.png)
   
             3. 再根据 user.id IN user.user_id  的条件，将结果进行筛选（id 列 和 user_id 列值是否相等，把不相等的删除），最后等到结果：
   
                ![image-20230318135057385](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230318135057385.png)
   
         * exists
   
           * 指定一个子查询，检测行的存在。**遍历循环外表**，然后看外表中的记录有没有和内表的数据一样的。匹配上就将结果放入结果集中
   
           * 具体 sql 
   
             ~~~mysql
             -- 表与上文一样，执行结果一样
             select user.* from user where exists(select order.user_id from order where user.id = order.user_id)
             ~~~
   
             1. 使用exists 进行查询时，先查询主查询表，也就是说，先执行的 sql 是：select user.* from user
             2. 然后，根据表的每一条记录，执行 exists() 中的语句，依次去判断 where 后面的条件是否成立
             3. 将成立的结果返回
   
           * 也就是说，如果 主查询的表 A 有 1000 条数据，子查询表 B 有 10000 条数据，那么 exists() 会执行 1000 次。而 如果 A 有 10000 条数据，B 有100 条数据，exists() 还是会执行 10000 次。因此，**exists 适合 B 表 比 A 表数据大的情况**
   
           * **in() 是在内存里遍历比较，而exists() 需要查询数据库，查数据库消耗性能更高**
   
         * **not in 和 not exists** ：如果**查询语句使用了not in 那么内外表都进行全表扫描，没有用到索引；而not exists 的子查询依然能用到表上的索引。所以无论那个表大，用not exists都比not in要快**
   
       * Union 和 Union All 的区别
   
         * Union ：对两个结果集进行并集操作，**不包括重复行**，同时进行**默认规则的排序**
         * Union All ：对两个结果集进行并集操作，**包括重复行，不进行排序**
         * Union 因为要进行重复值扫描，所以效率低。**如果合并没有刻意要删除重复行，那么就使用Union All**
   
       * Drop、Delete 和 Truncate 的区别
   
         * 执行速度上：drop > truncate >> DELETE
         * delete：支持回滚，删除时 表结构还在，删除表的全部或部分数据。其实并不会真的把数据删除，**mysql 实际上只是给删除的数据打了个标记为已删除，因此 delete 删除表中的数据时，表文件在磁盘上所占空间不会变小，存储空间不会被释放，只是把删除的数据行设置为不可见**。虽然未释放磁盘空间，但是下次插入数据的时候，仍然可以重用这部分空间（重用 → 覆盖），**可通过 optimize table 被删除的表名 请立刻释放磁盘空间**
         * truncate：不支持回滚，表结构还在，删除表的所有数据。**执行之后立即生效，无法找回，立刻释放磁盘空间**。并且会**重置 auto_increment（自增序列）** 的值：对于MyISAM，truncate 会重置 auto_increment 的值为1。而delete 保持auto_increment。对于InnoDB，truncate 重置为 1，delete 保持，但是delete整个表后重启MySQL的话，重启后auto_increment 被置为1
         * drop：不支持回滚，从数据库中删除表的所有数据，包括索引和权限。drop **语句将删除表的结构被依赖的约束(constrain)、触发器(trigger)、索引(index); 依赖于该表的存储过程/函数将保留,但是变为 invalid 状态。**
         * 可以这么理解，一本书，delete是把目录撕了，truncate是把书的内容撕下来烧了，drop是把书烧了
   
   14. MySQL 优化：覆盖索引（延迟关联）
   
       * 当我们查询 **`select \* from T where k=5`** 其实会先到k那个索引树上查询k = 5，然后找到对应的id为500，**最后回表到主键索引的索引树找返回所需数据**。
         如果我们查询**`select id from T where k=5`** 则不需要回表就直接返回，因为 k 索引树上已经有 id 的信息了。
         也就是说，**基于非主键索引的查询需要多扫描一棵索引树**。因此，我们**在应用中应该尽量使用主键查询**，这也就是**覆盖索引**，由于覆盖索引**可以减少树的搜索次数，显著提升查询性能，所以使用覆盖索引是一个常用的性能优化手段**
   
       * 覆盖索引的一个具体实现就是 **延迟关联**
   
         * 比如MySQL 的分页，首先想到的就是 `offset、limit` 操作，但随着页数的增加，查询性能指数级增大。
   
         * 这是由**于 MySQL 并不是跳过 offset 的行数，而是取 offset + limit 行，然后丢弃前 offset 行，返回 limit 行**，当offset特别大的时候，效率就非常的低下
   
         * 通过 覆盖索引+延迟关联 来减少偏移量的定位进行优化：
   
           ~~~mysql
           -- 查询语句
           select id from product limit 10000000, 10
           -- 优化方式二
           -- 只查询 id 的分页，因为 id 是主键，即使子查询中有where 筛选，也不需要回表，因为其他的索引树上也有id的信息
           SELECT * FROM product a JOIN (select id from product limit 10000000, 10) b ON a.ID = b.id
           ~~~
   
   15. 数据表的约束：主键约束（primary key）、非空（not null）、默认值（default）、唯一（unique）、外键（foreign key）






## Redis

1. Redis，远程字典服务，C语言编写，Key-Value 数据库。与MySQL数据库不同的是，Redis的数据是存在内存中的。它的读写速度非常快，每秒可以处理超过10万次读写操作。因此redis被**广泛应用于缓存**。**6379端口**

2. Redis **基本数据类型**：

   * String（字符串）：C 语言的字符串是char[] 实现，但是Redis 使用 **SDS 封装**，选择SDS结构的原因之一是：**SDS中，O(1)时间复杂度，就可以获取字符串长度；而C 字符串，需要遍历整个字符串，时间复杂度为O(n)**

     ~~~c
     struct sdshdr{ 
         unsigned int len; // 标记buf的长度 
         unsigned int free; //标记buf中未使用的元素个数 
         char buf[]; // 存放元素的坑
     }

   * Hash（哈希）：在Redis中，**哈希类型是指v（值）本身又是一个键值对（k-v）结构**，字符串和哈希类对比如下图：

     ![image-20230310213654673](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230310213654673.png)

   * List （列表）：列表（list）类型是用来**存储多个有序的字符串**，一个列表最多可以存储2^32-1个元素

   * Set（集合）：也是用来保存多个的字符串元素，但是不允许重复元素

   * zset （有序集合）：已排序的字符串集合，同时元素不能重复

3. 单线程模型

   * Redis是**单线程模型的**，而单线程避免了CPU不必要的上下文切换和竞争锁的消耗。也正因为是单线程，**如果某个命令执行过长（如hgetall命令），会造成阻塞**
   * Redis 6.0 引入了**多线程提速**，它的执行命令操作内存的**仍然是个单线程。**

4. Redis 虚拟内存机制

   * 虚拟内存机制就是**暂时把不经常访问的数据**(冷数据)**从内存交换到磁盘中**，从而腾出宝贵的内存空间用于其它需要访问的数据(热数据)。通过VM功能可以实现**冷热数据分离**，使热数据仍在内存中、冷数据保存到磁盘。这样就可以避免因为内存不足而造成访问速度下降的问题。

5. **缓存穿透、缓存雪崩、缓存击穿（重点）**

   * 常见的缓存使用方式：**读请求来了，先查下缓存，缓存有值命中，就直接返回；缓存没命中，就去查数据库，然后把数据库的值更新到缓存，再返回**
   * 缓存穿透
     * 指查询一个**一定不存在的数据**，由于缓存是不命中时需要从数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，进而给数据库带来压力
     * 通俗点说，读请求访问时，缓存和数据库都没有某个值，这样就会导致每次对这个值的查询请求都会穿透到数据库，这就是缓存穿透
     * 产生缓存穿透的情况：
       1. **业务不合理设计**：比如大多数用户都没开守护，但是你的每个请求都去缓存，查询某个userid查询有没有守护
       2. **业务/运维/开发失误的操作**，比如缓存和数据库的数据都被误删除了
       3. **黑客非法请求攻击**，比如黑客故意捏造大量非法请求，以读取不存在的业务数据
     * 避免方法：
       1. 如果是非法请求，我们在API入口，**对参数进行校验**，过滤非法值
       2. 如果查询数据库为空，我们可以给缓存**设置个空值**，或者默认值。但是如有有写请求进来的话，需要更新缓存哈，以保证缓存一致性，同时，最后给缓存**设置适当的过期时间**。（业务上比较常用，简单有效）
       3. 使用**布隆过滤器**快速判断数据是否存在。即一个查询请求过来时，先通过布隆过滤器判断值是否存在，存在才继续往下查
   * 缓存雪崩
     * 指缓存中数据**大批量到过期时间**，而查询数据量巨大，请求都直接访问数据库，引起数据库压力过大甚至down机
     * 解决方案：
       1. 缓存雪奔一般是由于大量数据同时过期造成的，对于这个原因，可通过**均匀设置过期时间解决**，即让过期时间相对离散一点。如**采用一个较大固定值+一个较小的随机值**，5小时+0到1800秒这样子
       2. Redis 故障宕机也可能引起缓存雪奔。这就需要**构造Redis高可用集群**啦
   * 缓存击穿
     * 指**热点key在某个时间点过期**的时候，而恰好在这个时间点对这个Key有大量的并发请求过来，从而大量的请求打到数据库上
     * 缓存击穿和缓存雪崩看着有点像，其实它两区别是，**缓存雪奔是指数据库压力过大甚至down机**，**缓存击穿只是大量并发请求到了DB数据库层面**。可以认为**击穿是缓存雪崩的一个子集**吧。有些文章认为它俩区别，是区别在于**击穿针对某一热点key缓存，雪崩则是很多key**
     * 解决方案
       1. **使用互斥锁方案**。缓存失效时，不是立即去加载db数据，而是先使用某些带成功返回的原子操作命令，如(Redis的setnx）去操作，成功的时候，再去加载db数据库数据和设置缓存。否则就去重试获取缓存
       2.  **“永不过期”**，是指没有设置过期时间，但是热点数据快要过期时，异步线程去更新和设置过期时间

6. Redis 持久化

   * RDB

     * 将redis某一时刻的数据持久化到磁盘中，是一种**快照式**的持久化方法。

     * redis会**单独创建（fork）一个子进程来进行持久化**，而主进程是不会进行任何IO操作的，这样就确保了redis极高的性能。

     * 如果需要进行大规模数据的恢复，且**对于数据恢复的完整性不是非常敏感**，那RDB方式要比AOF方式更加的高效。

     * 如果你对数据的完整性非常敏感，那么RDB方式就不太适合你，因为即使你每5分钟都持久化一次，当redis故障时，仍然会有近**5分钟的数据丢失**

     * RDB持久化，是指在指定的时间间隔内，执行指定次数的写操作，将内存中的数据集快照写入磁盘中，它是Redis默认的持久化方式。执行完操作后，在指定目录下会生成一个 **dump.rdb** 文件，Redis 重启的时候，通过加载`dump.rdb`文件来恢复数据。RDB触发机制主要有以下几种

       ![image-20230311191711227](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230311191711227.png)

   * AOF（Append Only File，即只允许追加不允许改写的文件）

     * 采用**日志的形式**来记录**每个写操作**，追加到文件中，重启时再重新执行AOF文件中的命令来恢复数据。它主要解决数据持久化的实时性问题。**默认是不开启的**

     * 通过**配置redis.conf中的appendonly yes**就可以打开AOF功能。如果有写操作（如SET等），redis就会被追加到AOF文件的末尾

     * 默认的AOF持久化策略是每秒钟fsync一次（fsync是指把缓存中的写指令记录到磁盘中），因为在这种情况下，redis仍然可以保持很好的处理性能，即使redis故障，也只会丢失最近1秒钟的数据

     * 因为采用了追加方式，**如果不做任何处理的话，AOF文件会变得越来越大**，为此，redis提供了**AOF文件重写（rewrite）机制**，即当AOF文件的大小超过所设定的阈值时，redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。举个例子或许更形象，**假如我们调用了100次INCR指令，在AOF文件中就要存储100条指令，但这明显是很低效的，完全可以把这100条指令合并成一条SET指令**，这就是重写机制的原理

     * 优点：数据的一致性和完整性更高

       缺点：**AOF记录的内容越多，文件越大，数据恢复变慢**

     * 如果运气比较差，AOF文件出现了被写坏的情况，也不必过分担忧，r**edis并不会贸然加载这个有问题的AOF文件，而是报错退出**。这时可以通过以下步骤来**修复出错的文件**：

       1. 备份被写坏的AOF文件
       2. 运行redis-check-aof –fix进行修复
       3. 用diff -u来看下两个文件的差异，确认问题点
       4. 重启redis，加载修复后的AOF文件

     * AOF 重写 内部运行原理

       * 在重写即将开始之际，redis会**创建（fork）一个“重写子进程**”，这个子进程会**首先读取现有的AOF文件**，并将其包含的指令进行**分析压缩并写入到一个临时文件中**
       * 与此同时，**主工作进程**会将新接收到的**写指令一边累积到内存缓冲区中**，**一边继续写入到原有的AOF文件中**，这样做是保证原有的AOF文件的可用性，避免在重写过程中出现意外
       * 当“重写子进程”完成重写工作后，它会**给父进程发一个信号**，父进程收到信号后就会**将内存中缓存的写指令追加到新AOF文件中**
       * 当**追加结束后**，redis就会用**新AOF文件来代替旧AOF文件**，之后再有新的写指令，就都会追加到新的AOF文件中了

   * 对于我们应该选择RDB还是AOF，**官方的建议是两个同时使用**。这样可以提供更可靠的持久化方案

7. **Redis 高可用**（重点）

   1. 主从模式

      * 主从模式中，Redis部署了多台机器，有**主节点，负责读写**操作，有**从节点，只负责读操作**。从节点的数据来自主节点，实现原理就是**主从复制机制**

      * 主从复制包括**全量复制，增量复制**两种。一般当slave**第一次启动连接master，或者认为是第一次连接**，就采用**全量复制**，全量复制流程如下：

        ![image-20230311195100512](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230311195100512.png)

      * slave与master全量同步之后，master上的数据，如果再次发生更新，就会触发**增量复制**

        当master节点发生**数据增减**时，就会触发**replicationFeedSalves()**函数，接下来在 Master节点上调用的每一个命令会使用replicationFeedSlaves()来同步到Slave节点。**执行此函数之前呢，master节点会判断用户执行的命令是否有数据更新**，如果有数据更新的话，并且slave节点不为空，就会执行此函数。这个函数作用就是：**把用户执行的命令发送到所有的slave节点，让slave节点执行**。流程如下：

        ![image-20230311195459763](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230311195459763.png)

   2. 哨兵模式

      * **主从模式中，一旦主节点由于故障不能提供服务，需要人工将从节点晋升为主节点，同时还要通知应用方更新主节点地址**。显然，多数业务场景都不能接受这种故障处理方式。Redis从2.8开始正式提供了Redis Sentinel（哨兵）架构来解决这个问题
      * 哨兵模式，由一个或多个Sentinel实例组成的Sentinel系统，它可以**监视所有的Redis主节点和从节点，并在被监视的主节点进入下线状态时，自动将下线主服务器属下的某个从节点升级为新的主节点**。但是呢，一个哨兵进程对Redis节点进行监控，就可能会出现问题（单点问题），因此**，可以使用多个哨兵来进行监控Redis节点，并且各个哨兵之间还会进行监控**。
      * 简单来说，哨兵模式就三个作用：
        - 发送命令，等待Redis服务器（包括主服务器和从服务器）返回监控其运行状态；
        - 哨兵监测到主节点宕机，会自动将从节点切换成主节点，然后通过发布订阅模式通知其他的从节点，修改配置文件，让它们切换主机；
        - 哨兵之间还会相互监控，从而达到高可用
      * 哨兵的工作模式如下：

        1. 每个Sentinel以**每秒钟一次**的频率向它所知的Master，Slave以及其他Sentinel实例发送一个 **PING命令**。
        2. 如果一个实例（instance）距离最后一次有效回复 PING 命令的时间**超过 down-after-milliseconds 选项所指定的值**， 则这个实例会被 Sentinel标记为**主观下线**。
        3. 如果一个Master被标记为主观下线，则**正在监视这个Master的所有 Sentinel 要以每秒一次的频率确认Master的确进入了主观下线状态**。
        4. 当有足够数量的 Sentinel（大于等于配置文件指定的值）在指定的时间范围内确认Master的确进入了主观下线状态， 则Master会被标记为**客观下线**。
        5. 在一般情况下， 每个 Sentinel 会以每10秒一次的频率向它已知的所有Master，Slave发送 INFO 命令。
        6. 当Master被 Sentinel 标记为客观下线时，Sentinel 向下线的 Master 的所有 Slave 发送 INFO 命令的频率会从 10 秒一次改为每秒一次
        7. **若没有足够数量的 Sentinel同意Master已经下线， Master的客观下线状态就会被移除**；若Master 重新向 Sentinel 的 PING 命令返回有效回复， Master 的主观下线状态就会被移除。

   3. Cluster 集群模式

      * 哨兵模式基于主从模式，实现读写分离，它还可以自动切换，**系统可用性更高**。但是它**每个节点存储的数据是一样的，浪费内存，并且不好在线扩容**
      * 因此，Cluster集群应运而生，它在Redis3.0加入的，实现了Redis的**分布式存储**。对数据进行分片，也就是说**每台Redis节点上存储不同的内容**，来解决在线扩容的问题。并且，它也提供**复制和故障转移的功能**

8. 在生成 RDB期间，Redis 可以同时处理写请求么？

   * 可以的，Redis提供两个指令生成RDB，分别是save和bgsave。
     1. 如果是save指令，会阻塞，因为是主线程执行的。
     2. 如果是bgsave指令，是fork一个子进程来写入RDB文件的，快照持久化完全交给子进程来处理，父进程则可以继续处理客户端的请求。

9. 布隆过滤器

   * 布隆过滤器是一种**占用空间很小的数据结构**，它由一个**很长的二进制向量和一组Hash映射函数组成**，它**用于检索一个元素是否在一个集合中**，空间效率和查询时间都比一般的算法要好的多，缺点是**有一定的误识别率和删除困难**

   * 假设集合A有3个元素，分别为{**d1,d2,d3**}。有1个哈希函数，为**Hash1**。现在将A的每个元素映射到长度为16位数组B。

     1. 把d1映射过来，假设Hash1（d1）= 2，我们就把数组B中，下标为2的格子改成1

     2. 把**d2**也映射过来，假设Hash1（d2）= 5，我们把数组B中，下标为5的格子也改成1

     3. **d3**也映射过来，假设Hash1（d3）也等于 2，它也是把下标为2的格子标1

        ![image-20230311202552872](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230311202552872.png)

     4. 因此，我们要确认一个元素dn是否在集合A里，我们只要算出Hash1（dn）得到的索引下标，只要是0，那就表示这个元素不在集合A，如果索引下标是1呢？那该元素可能是A中的某一个元素。因为你看，d1和d3得到的下标值，都可能是1，还可能是其他别的数映射的，布隆过滤器是存在这个缺点的：**会存在hash碰撞导致的假阳性**，判断存在误差

   * 如何**减少这种误差**呢？

     - 搞**多几个哈希函数映射**，降低哈希碰撞的概率
     - 同时增加B数组的bit长度，可以增大hash函数生成的数据的范围，也可以降低哈希碰撞的概率

   * 即使存在误差，我们可以发现，布隆过滤器并**没有存放完整的数据**，它只是运用一系列哈希映射函数计算出位置，然后填充二进制向量。如果**数量很大的话**，布隆过滤器通过极少的错误率，换取了存储空间的极大节省，还是挺划算的

10. redis 集群搭建

    1. 单机版搭建：

       * 进入 redis/bin 目录，命令：./redis-server 启动redis （前端启动 redis）
       * 改为后端启动：找到 redis.conf 文件，修改 redis.conf 文件 -> daemonize：no 改为daemonize：yse；，在 /bin 目录下输入命令：./redis-server redis.conf （加载配置文件式启动），此时就是后台启动

    2. 集群搭建：

       * Redis集群**至少需要3个节点**，因为**投票容错机制要求超过半数节点认为某个节点挂了该节点才是挂了**，所以2个节点无法构成集群。 并且，要保证集群的高可用，需要**每个节点都有从节点**，也就是备份节点，**所以Redis集群至少需要6台服务器**。这里搭建 **伪分布式 集群**。步骤如下：

         1. **关闭防火墙**，在 /usr/local 目录下，**新建 redis-cluster 目录**，用于存放集群节点

         2. 把 **redis 目录下的 bin 目录下所有文件复制**到 /usr/local/redis-cluster/redis01 目录下，命令：cp -r redis/bin/ redis-cluster/redis01

         3. **删除 redis01 目录下的快照文件 dump.rdb，并且修改 redis.conf 文件**，具体修改两个地方：一个端口号修改为 7001，二是开启集群创建模式，打开注释即可（cluster-enabled yes）

         4. 将redis-cluster/redis01文件**复制5份**到redis-cluster目录下（redis02-redis06），创建6个redis实例，模拟Redis集群的6个节点。然后将其余5个文件下的redis.conf里面的端口号分别修改为7002-7006

         5. 要搭建集群的话，需要使用一个工具（脚本文件），这个工具在redis解压文件的源代码里。因为这个工具是一个ruby脚本文件，所以这个工具的运行需要ruby的运行环境，就相当于java语言的运行需要在jvm上。所以需要**安装ruby**。需要注意的是：**redis的版本和ruby包的版本最好保持一致**。

         6. 把这个ruby脚本工具复制到usr/local/redis-cluster目录下。那么这个**ruby脚本工具**在哪里呢？之前提到过，在redis解压文件的源代码里，**即redis/src目录下的redis-trib.rb文件**

         7. 然后使用该脚本文件搭建集群，指令如下：

            ```shell
            ./redis-trib.rb create --replicas 1 47.106.219.251:7001 47.106.219.251:7002 47.106.219.251:7003 47.106.219.251:7004 47.106.219.251:7005 47.106.219.251:7006
            1
            ```

         8. 最后连接集群节点，连接任意一个即可：

            ```shell
            redis01/redis-cli -p 7001 -c 
            ```

            注意：**一定要加上-c**，不然节点之间是无法自动跳转的

         9. 两条redis集群基本命令：

            1.  查看当前集群信息

            ```shell
            cluster info
            1
            ```

            2. 查看集群里有多少个节点

            ```shell
            cluster nodes
            1
            ```

11. 为什么要使用缓存

    1. **高性能**：比如一个操作请求过来，查询一个结果耗时600ms，并且这个结果可能后面几个小时都不会变。这就可以使用缓存，把这个结果存在缓存中，下次还有请求查这个数据时，不需要去数据库查询，直接从缓存中返回给他
    2. **高并发**：比如高峰期一秒钟来了1万个请求，一个mysql单机绝对会挂掉，这个时候使用缓存的话，把很多数据放到缓存中，缓存是走内存的，内存天然支持高并发。

12. redis 的过期策略

    1. **定期删除**：redis 默认是每隔100ms 就**随机抽取**一些设置了过期时间的key，**检查其是否过期**，如果过期就删除。不过，定期删除可能会导致很多过期key到了时间并没有被删除掉。这就用到了惰性删除
    2. **惰性删除**：当获取某个key 时，redis会检查一下，如果该key 过期了，就会把它删除，不会给你返回任何东西
    3. 定期删除+惰性删除存在的问题：如果某个key过期后，定期删除没删除成功，也没有请求去访问这个key，也就是惰性删除没生效，这个key就会一直在内存中
    4. 这时需要走redis 的**内存淘汰机制**，8种淘汰机制，推荐优先使用 **allkeys-LRU** 策略（在所有键值对中，移除最近最少使用的键值对），用到了 LRU 经典缓存算法

13. redis 支持事务

    1. multi 命令开始，exec 命令结束
    2. 所有指令在 **exec 之后才执行**，把命令放在一个队列中，收到exec 指令后，才开始执行该 事务队列，执行完毕后一次性返回所有指令的结果
    3. Redis 事务**不支持回滚**，以保持Redis 的简单、快速的特性
    4. **事务不会被打断**，因为Redis是单线程的，命令顺序执行，而不会被其他线程打断。如果有客户端发送命令来，也需要等待事务命令执行完毕才执行该请求

14. Redis 分布锁

    1. 先拿 setnx 来争抢锁，抢到之后，再用 expire 给锁加一个过期时间防止锁忘记释放
    2. 如果在 setnx 之后执行 expire 之前进程意外 crash 或者需要重启维护了，会怎么样
    3. 这时锁永远得不到释放。set 指令有非常复杂的参数，可以同时把 setnx 和 expire 合成一条指令来用

15. 找出固定已知前缀的key

    1. 用 **keys** 指令，可以扫出指定模式的 key 列表。如果redis 正在给线上提供业务服务，由于redis 是单线程的，**keys 会导致线程阻塞一段时间，线上服务停顿，直到指令执行完毕**
    2. 用 **scan** 指令，可以**无阻塞**的提取出指定模式的key列表，但**会有一定的重复概率**，可以在客户端去重一下。总体**所花费的时间会比直接用key 指令长**

16. 





## JVM 

1. JDK 包含 JRE，JRE 包括 JVM。

   JRE 大部分都是 C 和 C++ 语言编写的，他是我们在编译java时所需要的基础的类库

   Jdk还包括了一些Jre之外的东西 ，就是这些东西帮我们编译Java代码的， 还有就是监控Jvm的一些工具

2. **运行时数据区**

   ![image-20230313201838145](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230313201838145.png)

   1. 方法区
      * 用于存储已被Java虚拟机加载的**类信息、常量、静态变量**、即时编译器编译后的代码等数据。
      * 当方法区无法满足内存分配需求时，抛出OutOfMemoryError异常。
   2. 堆
      * java虚拟机所管理的内存中最大的一块，在虚拟机**启动时创建**。此内存区域的唯一目的就是**存放对象实例**。所有的对象实例以及**数组都要在堆上分配**
      * java堆是垃圾收集器管理的主要区域，因此也被成为“**GC堆**”
      * java堆可以处于物理上**不连续的内存空间**中。如果堆中没有内存可以完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常
   3. 程序计数器
      * 保存**当前线程**所正在执行的字节码指令的地址(行号)
      * 由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，一个处理器都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，**每条线程都有一个独立的程序计数器**，各个线程之间计数器互不影响，独立存储。称之为“线程私有”的内存。程序计数器内存区域是虚拟机中唯一没有规定OutOfMemoryError情况的区域。
   4. Java 虚拟机栈
      * 线程私有的，它的**生命周期和线程相同**
      * 每个方法在执行的同时都会创建一个栈帧（Stack Frame）用于**存储局部变量表、操作数栈、动态链接、方法出口**等信息
      * 解析栈帧：
        1. 局部变量表：是用来存储我们临时**8个基本数据类型、对象引用地址、returnAddress类型**。（**returnAddress中保存的是return后要执行的字节码的指令地址**。）
        2. 操作数栈：操作数栈就是用来操作的，例如代码中有个 i = 6*6，他在一开始的时候就会进行操作，读取我们的代码，进行计算后再放入局部变量表中去
        3. 动态链接：假如我方法中，有个 service.add()方法，**要链接到别的方法中去**，这就是动态链接，存储链接的地方。
        4. 出口：出口是什呢，**出口正常的话就是return 不正常的话就是抛出异常**
   5. 本地方法栈
      * 方法上带了 native 关键字的栈。本地方法栈中就是C和C++的代码

3. JVM 垃圾回收机制

   * 程序在运行过程中，会产生大量的内存垃圾（一些没有引用指向的内存对象都属于内存垃圾，因为这些对象已经无法访问，程序用不了它们了，对程序而言它们已经死亡），为了确保程序运行时的性能，java虚拟机在程序运行的过程中不断地进行自动的垃圾回收（GC）
   * finalize 方法
     1. finalize()方法是在每次执行GC操作之前时会调用的方法，可以用它做必要的清理工作
     2. 它是在Object类中定义的，因此所有的类都继承了它。子类覆盖finalize()方法以整理系统资源或者执行其他清理工作。finalize()方法是在垃圾收集器删除对象之前对这个对象调用的

4. 新生代、老年代、永久代（方法区）

   * 堆被划分成两个不同的区域：新生代 ( Young )、老年代 ( Old )
   * 老年代就一个区域。新生代 ( Young ) 又被划分为三个区域：**Eden、From Survivor、To Survivor**
   * **默认的**，新生代 ( Young ) 与老年代 ( Old ) 的比例的值为 **1:2** ( 该值可以通过参数 **–XX:NewRatio** 来指定 )，即：新生代 ( Young ) = 1/3 的堆空间大小。老年代 ( Old ) = 2/3 的堆空间大小
   * 默认的，Edem ： From Survivor ： To Survivor = **8 : 1 : 1** ( 可以通过参数 **–XX:SurvivorRatio** 来设定 )
   * JVM 每次只会使用 Eden 和其中的一块 Survivor 区域来为对象服务，所以无论什么时候，**总是有一块 Survivor 区域是空闲着的**。因此，新生代实际可用的内存空间为 9/10 ( 即90% )的新生代空间
   * **永久代就是JVM的方法区**。在这里都是放着一些被虚拟机加载的类信息，静态变量，常量等数据。这个区中的东西比老年代和新生代更不容易回收

5. 为什么要这样分代

   * 可以根据各个年代的特点进行对象分区存储，更便于回收，采用最适当的收集算法：
     1. 新生代中，每次垃圾收集时都发现大批对象死去，只有少量对象存活，便采用了复制算法，只需要付出少量存活对象的复制成本就可以完成收集
     2. 而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须采用“标记-清理”或者“标记-整理”算法
   * 数据会**首先分配到Eden区当中**（当然也有特殊情况，如果是**大对象那么会直接放入到老年代**（**大对象是指需要大量连续内存空间的java对象**）。**当Eden没有足够空间的时候**就会触发jvm发起一次**Minor GC**，。如果对象经过一次Minor-GC还存活，**并且又能被Survivor空间接受，那么将被移动到Survivor空间当中**。并将其年龄设为1，对象在Survivor**每熬过一次Minor GC，年龄就加1**，当年龄达到一定的程度（**默认为15**）时，就会被晋升到老年代中了，当然晋升老年代的年龄是可以设置的

6. Minor GC、Major GC、Full GC区别及触发条件

   * 区别
     1. Minor GC：新生代GC，发生在新生代的垃圾收集动作。由于java对象大都是朝生夕死的，所以**Minor GC非常频繁**，一般**回收速度也比较快**
     2. Major GC：老年代GC，通常执行**Major GC会连着Minor GC一起执行**。Major GC的速度要比Minor GC慢的多
     3. Full GC：是清理整个堆空间，包括年轻代和老年代
   * 触发条件
     1. Minor GC：**eden区满时**，触发MinorGC。即申请一个对象时，发现eden区不够用，则触发一次MinorGC。**新创建的对象大小 > Eden所剩空间**
     2. Major GC 和 Full GC：**Major GC通常是跟full GC是等价的**。
        * 每次**晋升到老年代的对象平均大小>老年代剩余空间**
        * **MinorGC后存活的对象超过了老年代剩余空间**
        * **永久代空间不足**
        * 执行System.gc()
        * CMS GC异常
        * 堆内存分配很大的对象

7. **判断对象是否存活**

   1. **引用计数器法**
      * 引用计数法就是**如果一个对象没有被任何引用指向**，则可视之为垃圾。这种方法的**缺点**就是它**很难解决对象之间相互循环引用的问题**（**循环引用**）
      * 什么是引用计数法：每个对象在创建的时候，就给这个对象绑定一个计数器。每当有一个引用指向该对象时，计数器加一；每当有一个指向它的引用被删除时，计数器减一。这样，当没有引用指向该对象时，计数器为0就代表该对象死亡
      * **主流 JVM 没有选用该方法**
   2. 可达性分析法
      * 从GC Roots开始向下搜索，搜索所走过的路径为引用链。当一个对象到GC Roots没用任何引用链时，则证明此对象是不可用的，表示可以回收
      * 可作为 GC Roots 的对象：
        1. 虚拟机栈（栈帧中的本地变量表）中引用的对象
        2. 方法区中类静态属性引用的对象
        3. 方法区中常量引用的对象
        4. 本地方法栈中JNI（即一般说的Native方法）引用的对象
      * 主流 JVM 采用该方法

8. **GC 算法**

   1. 标记-清除 算法
      * 为每个对象**存储一个标记位**，记录对象的状态（活着或是死亡）。
        分为两个阶段，一个是**标记阶段**，这个阶段内，为每个对象更新标记位，检查对象是否死亡；第二个阶段是**清除阶段**，该阶段对死亡的对象进行清除，执行 GC 操作
      * 优点：
        1. 可以解决**循环引用**
        2. 必要时才回收(内存不足时)
      * 缺点：
        1. 回收时，**应用需要挂起**，也就是stop the world
        2. 标记和清除的**效率不高**，尤其是要扫描的对象比较多的时候
        3. 会造成**内存碎片**(会导致明明有内存空间,但是由于不连续,申请稍微大一些的对象无法做到)
      * 应用场景：一般应用于**老年代**，因为老年代的对象生命周期比较长
   2. 标记-整理 算法
      * 标记-清除法的一个**改进版**。同样，在**标记阶段**，该算法也将所有对象标记为存活和死亡两种状态；不同的是，在第二个阶段，该算法**并没有直接对死亡的对象进行清理**，而是**将所有存活的对象整理一下，放到另一处空间**，然后把剩下的所有对象全部清除。这样就达到了标记-整理的目的。
      * 优点：
        1. 解决标记清除算法出现的内存碎片问题
      * 缺点：
        1. 压缩阶段，由于移动了可用对象，需要去更新引用
      * 应用场景：一般应用于**老年代**,因为老年代的对象生命周期比较长
   3. 复制 算法
      * 将**内存平均分成两部分**，然后每次只使用其中的一部分，**当这部分内存满的时候，将内存中所有存活的对象复制到另一个内存中**，然后将之前的内存清空，只使用这部分内存，循环下去
      * 与标记-整理算法的区别在于，该算法不是在同一个区域复制，而是将所有存活的对象复制到另一个区域内
      * 优点：
        1. 在存活对象不多的情况下，**性能高**，能**解决内存碎片**和java垃圾回收算法之-标记清除 中导致的**引用更新**问题
      * 缺点：
        1. 会**造成一部分的内存浪费**。不过可以根据实际情况，将内存块大小比例适当调整；如果存活对象的数量比较大，复制算法的性能会变得很差
      * 应用场景：
        1. 复制算法一般是使用在**新生代**中，因为新生代中的对象一般都是朝生夕死的，存活对象的数量并不多，这样使用复制算法进行拷贝时效率比较高
        2. 将新生代划分为Eden与2块Survivor Space(幸存者区) ，然后**在Eden –>Survivor Space 与To Survivor之间实行复制算法**
        3. jvm在应用复制算法时，**并不是把内存按照1:1来划分的**，这样太浪费内存空间了。一般的 jvm 都是 8:1。也即是说，Eden区 : From区 : To区域的比例是始终有**90%的空间是可以用来创建对象的,而剩下的10%用来存放回收后存活的对象**
   4. 分代 算法
      * **根据对象的存活周期**的不同将内存划分成几块，新生代和老年代，这样就可以**根据各个年代的特点采用最适当的收集算法**
      * 在**新生代**，每次垃圾收集器都发现有大批对象死去，只有少量存活，采用**复制算法**，只需要付出少量存活对象的复制成本就可以完成收集
      * **老年代**中因为对象存活率高、没有额外空间对它进行分配担保，就必须**标记清除**法或者标记整理算法进行回收

9. 垃圾回收器

   * 垃圾收集器是**垃圾回收算法**（引用计数法、标记清楚法、标记整理法、复制算法）**的具体实现**，不同垃圾收集器、不同版本的JVM所提供的垃圾收集器可能会有很在差别

     ![image-20230313212107597](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230313212107597.png)

   * 分类：

     1. 新生代收集器：Serial、ParNew、Parallel Scavenge
     2. 老年代收集器：CMS、Serial Old、Parallel Old
     3. 整堆收集器：G1

   * 详情

     1. Serial

        * 新生代、单线程、复制算法

        * 只会使用一个 CPU 或者线程去完成垃圾收集工作，而且在它进行垃圾收集时，**必须暂停其他所有的工作线程**，直到它收集结束

        * **简单高效**，适合单 CPU 环境。单线程没有线程交互的开销，因此**拥有最高的单线程收集效率**

        * 使用方式

          ~~~java
          "-XX:+UseSerialGC" 
          ~~~

     2. ParNew

        * 新生代、多线程、复制算法

        * Serial 多线程版本，除了多线程外，其他行为、特点与 Serial 一样

        * **只有它能与 CMS 配合使用**

        * 但在单个CPU环境中，不比Serail收集器好，多线程使用它比较好

          ~~~java
          设置垃圾收集器："-XX:+UseParNewGC"  --强制指定使用ParNew；    
          设置垃圾收集器： "-XX:+UseConcMarkSweepGC"  --指定使用CMS后，会默认使用ParNew作为新生代收集器；
          设置垃圾收集器参数："-XX:ParallelGCThreads"  --指定垃圾收集的线程数量，ParNew默认开启的收集线程与CPU的数量相同；
          ~~~

     3. Parallel Scavenge

        * 新生代、多线程、复制算法

        * 与ParNew 不同的是：**高吞吐量**为目标，（**减少垃圾收集时间**，让用户代码获得更长的运行时间）

          ~~~java
          设置垃圾收集器："-XX:+UseParallelGC"  --添加该参数来显式的使用改垃圾收集器；
          设置垃圾收集器参数："-XX:MaxGCPauseMillis"  --控制垃圾回收时最大的停顿时间(单位ms)
          设置垃圾收集器参数："-XX:GCTimeRatio"  --控制程序运行的吞吐量大小吞吐量大小=代码执行时间/(代码执行时间+gc回收的时间)
          设置垃圾收集器参数："-XX:UseAdaptiveSizePolicy"  --内存调优交给虚拟机管理
          ~~~

     4. Serial Old

        * 老年代、单线程、标记-整理 算法

        * Serial 老年代版本

          ~~~java
          在JDK1.5及之前，与Parallel Scavenge收集器搭配使用，
          在JDK1.6后有Parallel Old收集器可搭配。
          现在的作为CMS收集器的后备预案，在并发收集发生Concurrent Mode Failure时使用
          ~~~

     5. Parallel Old

        * 老年代、多线程、标记-整理 算法

        * 在单个CPU环境中，不比Serial Old收集器好，多线程使用它比较好

          ~~~java
          设置垃圾收集器："-XX:+UseParallelOldGC"：指定使用Parallel Old收集器；
          ~~~

     6. CMS

        * 老年代、**标记-清除 算法**

        * 一种以**获取最短回收停顿时间为目标**的收集器，适用于互联网站或者 **B/S 系统的服务端**上。

        * **并发收集、低停顿**。垃圾收集线程与用户线程（基本上）可以**同时工作**

        * 缺点：

          1. 对CPU资源非常敏感
          2. 无法处理浮动垃圾，可能出现"Concurrent Mode Failure"失败
          3. 产生大量内存碎片

        * 步骤：

          1. 初始标记：标记GC Roots 能直接关联到的对象
          2. 并发标记：进行GC Roots Tracing
          3. 重新标记：修正并发标记期间的变动部分
          4. 并发清除

          ~~~java
          设置垃圾收集器："-XX:+UseConcMarkSweepGC"：指定使用CMS收集器；
          ~~~

     7. G1

        * 整个 GC 堆（新生代和老年代）、采用不同方式处理不同时期的对象

        * **分代收集器**。当今收集器技术发展最前沿成果之一，是一款**面向服务端应用**的垃圾收集器。G1可以说是CMS的终极改进版，**解决了CMS内存碎片、更多的内存空间等问题**。虽然流程与CMS比较相似，但底层的原理已是完全不同

        * 特点：

          1. 充分利用多CPU、多核环境下的硬件优势
          2. 可以**并行**来缩短(Stop The World)停顿时间
          3. 并发让垃圾收集与用户程序同时进行
          4. 应用场景可以面向服务端应用，针对具有大内存、多处理器的机器
          5. 采用**标记-整理 + 复制算法**来回收垃圾

        * 运行步骤：

          1. 初始标记
          2. 并发标记
          3. 最终标记
          4. 筛选回收

          ~~~java
          设置垃圾收集器："-XX:+UseG1GC"：指定使用G1收集器；
          设置垃圾收集器参数："-XX:InitiatingHeapOccupancyPercent"：当整个Java堆的占用率达到参数值时，开始并发标记阶段；默认为45；
          设置垃圾收集器参数："-XX:MaxGCPauseMillis"：为G1设置暂停时间目标，默认值为200毫秒；
          设置垃圾收集器参数："-XX:G1HeapRegionSize"：设置每个Region大小，范围1MB到32MB；目标是在最小Java堆时可以拥有约2048个Region
          ~~~

   

10. 常用的 JVM 配置

    * 配置参数

      ~~~java
      -Xms：初始堆大小，JVM 启动的时候，给定堆空间大小
      -Xmx：最大堆大小，JVM 运行过程中，如果初始堆空间不足的时候，最大可以扩展到多少。 
      -Xmn：设置堆中年轻代大小。整个堆大小=年轻代大小+年老代大小+持久代大小。 
      -XX:NewSize=n 设置年轻代初始化大小大小 
      -XX:MaxNewSize=n 设置年轻代最大值
      -Xss：设置每个线程的堆栈大小。JDK5后每个线程 Java 栈大小为 1M，以前每个线程堆栈大小为 256K。
      -XX:MaxTenuringThreshold=n 设置年轻带垃圾对象最大年龄。如果设置为 0 的话，则年轻代对象不经过 Survivor 区，直接进入年老代。
      ~~~

    * 在哪里设置

      1. 单项目应用

         * Run -> Edit Configurations -> VM options (新版本的idea 中需要点击右上角的 Modify options -> add VM options)

      2. 全局配置

         1. 找到IDEA安装目录中的 bin 目录
         2. 找到 idea.exe.vmoptions 文件
         3. 打开该文件编辑并保存

      3. war （Tomcat）包

         * 修改 Tomcat 的 JVM 参数

         * Windows 下就是文件 /bin/**catalina.bat**，增加设置：JAVA_OPTIONS，如 set “JAVA_OPTIONS=-Xms512M -Xmx1024M”

         * Linux 下，tomcat 的 bin 目录下 catalina.sh 添加

           ~~~shell
           注意：位置要在cygwin=false前
           JAVA_OPTS="-Xms512M -Xmx1024M ...等等等 JVM参数"
           ~~~

      4. Jar 包，在 java -jar 后面插入 JVM 命令

11. JVM 调优

    1. 如果使用合理的JVM 参数配置，在大多数情况应该不需要调优。如：
       * XX:NewRatio=2，年轻代:老年代=1:2
       * -XX:SurvivorRatio=8，eden:survivor=8:1
       * 堆内存设置为物理内存的3/4左右

    2. 对于单台服务器来说，JVM 的一些 **核心指标** 的 **合理范围**：

       * jvm.gc.time：每分钟的GC耗时在1s以内，500ms以内尤佳

       * jvm.gc.meantime：每次YGC（新生代GC）耗时在100ms以内，50ms以内尤佳

       * jvm.fullgc.count：FGC（Full GC）最多几小时1次，1天不到1次尤佳

       * jvm.fullgc.time：每次FGC耗时在1s以内，500ms以内尤佳

    3. 优化步骤：

       1. 分析 JVM 的核心指标

          * CPU 指标

            * 查看占用 CPU 最多的进程
            * 查看占用 CPU 最多的线程
            * 查看线程堆栈快照信息

            ~~~shell
            // 显示系统各个进程的资源使用情况
            top
            // 查看某个进程中的线程占用情况
            top -Hp pid
            // 查看当前 Java 进程的线程堆栈信息
            jstack pid
            ~~~

            * 常用工具：JProfiler、Arthas 

          * JVM 指标

            * 查看当前 JVM 堆内存参数配置是否合理

            * 查看堆中对象的统计信息

            * 查看堆存储快照，分析内存的占用情况

              ~~~java
              // 查看当前的 JVM 参数配置
              ps -ef | grep java
              // 查看 Java 进程的配置信息，包括系统属性和JVM命令行标志
              jinfo pid
              // 输出 Java 进程当前的 gc 情况
              jstat -gc pid
              // 输出 Java 堆详细信息
              jmap -heap pid
              // 显示堆中对象的统计信息
              jmap -histo:live pid
              // 生成 Java 堆存储快照dump文件
              jmap -F -dump:format=b,file=dumpFile.phrof pid
              ~~~

            * 常用工具：Eclipse MAT、JConsole

          * JVM GC 指标

            * 查看每分钟GC时间是否正常

            * 查看每分钟YGC次数是否正常

            * 查看FGC次数是否正常

            * 查看单次FGC时间是否正常

            * 查看单次GC各阶段详细耗时，找到耗时严重的阶段

            * 查看对象的动态晋升年龄是否正常

            * 添加以下参数，来丰富我们的GC日志输出，方便我们定位问题：

              ~~~java
              // 打印GC的详细信息
              -XX:+PrintGCDetails
              // 打印GC的时间戳
              -XX:+PrintGCDateStamps
              // 在GC前后打印堆信息
              -XX:+PrintHeapAtGC
              // 打印Survivor区中各个年龄段的对象的分布信息
              -XX:+PrintTenuringDistribution
              // JVM启动时输出所有参数值，方便查看参数是否被覆盖
              -XX:+PrintFlagsFinal
              // 打印GC时应用程序的停止时间
              -XX:+PrintGCApplicationStoppedTime
              // 打印在GC期间处理引用对象的时间（仅在PrintGCDetails时启用）
              -XX:+PrintReferenceGC
              ~~~

       2. 确定优化目标

          * 定位出系统瓶颈后，在优化前先制定好优化的目标是什么:
            * 将FGC次数从每小时1次，降低到1天1次
            * 将每分钟的GC耗时从3s降低到500ms
            * 将每次FGC耗时从5s降低到1s以内

       3. 制定优化方案

          * 针对定位出的系统瓶颈制定相应的优化方案，常见的有：
            * 代码bug：升级修复bug。典型的有：死循环、使用无界队列。
            * 不合理的JVM参数配置：优化 JVM 参数配置。典型的有：年轻代内存配置过小、堆内存配置过小、元空间配置过小。

       4. 对比优化前后的指标，持续观察和跟踪优化效果

    4. 调优案例：metaspace (元空间) 导致频繁FGC问题（服务环境：ParNew + CMS + JDK8）

       1. 首先查看GC日志，发现出现FGC的原因是metaspace空间不够。对应的 GC 日志：

          ~~~java
          Full GC (Metadata GC Threshold)
          ~~~

       2. 进一步查看日志发现元空间存在内存碎片化现象。对应的 GC 日志：

          ~~~java
          Metaspace       used 35337K, capacity 56242K, committed 56320K, reserved 1099776K
          ~~~

          参数意义：

          * used ：已使用的空间大小
          * capacity：当前已经分配且未释放的空间容量大小
          * committed：当前已经分配的空间大小
          * reserved：预留的空间大小

          解析：元空间的分配以 **chunk** 为单位，当一个 ClassLoader 被垃圾回收时，所有属于它的空间（chunk）被释放，此时该 chunk 称为 Free Chunk，而 committed chunk 就是 capacity chunk 和 free chunk 之和

          ![image-20230313231141082](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230313231141082.png)

          之所以说**内存存在碎片化现象**就是**根据 used 和 capacity 的数据得来的**，上面说了元空间的分配以 chunk 为单位，即使一个 ClassLoader 只加载1个类，也会独占整个 chunk，所以**当出现 used 和 capacity 两者之差较大的时候，说明此时存在内存碎片化的情况**

          元空间主要适用于存放类的相关信息，而存在内存碎片化说明很可能创建了较多的类加载器，同时使用率较低。

          因此，**当元空间出现内存碎片化时，我们会着重关注是不是创建了大量的类加载器**

       3. 通过 dump 堆存储文件发现存在大量 DelegatingClassLoader：

          * 通过进一步分析，发现是由于反射导致创建大量 DelegatingClassLoader。其核心原理如下：
            * 在 JVM 上，最初是通过 JNI 调用来实现方法的反射调用，当 JVM 注意到通过反射经常访问某个方法时，它将生成字节码来执行相同的操作，称为膨胀（inflation）机制。如果使用字节码的方式，则会为该方法生成一个 DelegatingClassLoader，**如果存在大量方法经常反射调用，则会导致创建大量 DelegatingClassLoader**
       
       4. 反射调用频次达到多少才会从 JNI 转字节码？
       
          * 默认是**15次**，可通过参数 **-Dsun.reflect.inflationThreshold** 进行控制，在小于该次数时会使用 JNI 的方式对方法进行调用，如果调用次数超过该次数就会使用字节码的方式生成方法调用。
       
          * 分析结论：反射调用导致创建大量 DelegatingClassLoader，占用了较大的元空间内存，同时存在内存碎片化现象，导致元空间利用率不高，从而较快达到阈值，触发 FGC。
       
          * **优化策略**：
            1. 适当调大 metaspace 的空间大小。
            2. 优化不合理的反射调用。例如最常见的属性拷贝工具类 BeanUtils.copyProperties 可以使用 mapstruct 替换
       
          

12. JVM 可视化工具

    1. visual VM
       * 提供强大的分析能力，对 Java 应用程序做性能分析和调优。这些功能包括生成和分析海量数据、跟踪内存泄漏、监控垃圾回收器、执行内存和 CPU 分析
       * 位置：JDK 根目录的 bin 文件夹下的 visualvm.exe
    2. jconsole
       * JConsole 是一个内置 Java 性能分析器，可以从命令行或在 GUI shell 中运行。您可以轻松地使用 JConsole来监控 Java 应用程序性能和跟踪 Java 中的代码
       * jdk/bin 目录下面的 jconsole.exe

13. 永久代和元空间

    1. 永久代和元空间都是方法区的一种实现，JDK7之前是用永久代实现方法区，之后是用元空间实现
    2. 永久代的方法区和堆使用的物理内存是连续的，永久代的方法区存储了**类信息，运行时常量池，常量、静态变量等**，因为永久代的空间配置有限，很有可能出现OOM。
    3. 元空间实现的方法区，物理地址不再与堆连续，而是直接存在于本地内存中，理论上机器内存有多大，元空间就有多大。永久代在JDK8被移除，**将静态变量和字符串常量池移到了堆中**。（调整字符串常量池的原因：因为永久代的回收效率很低，在Full GC 时才会触发，而Full GC 在老年代空间不足才触发，这导致字符串常量池的回收效率不高，开发中会创建大量的字符串，回收效率低会导致永久代空间不足，放堆里能及时回收内存）
    4. 为什么使用元空间替代永久代？
       * 因为通常使用配置参数设置永久代的大小就决定了永久代的上限，但是不是总能知道设置多大合适，如果使用默认值很容易发生OOM。
       * 当使用元空间时，可以加载多少类的元数据就不再由参数控制，而由系统的实际可用空间来控制


14. JVM的组成（两个子系统和两个组件）

    * 两个子系统为Class loader(类装载)、Execution engine(执行引擎)；

      两个组件为Runtime data area(运行时数据区)、Native Interface(本地接口)。

      ![image-20230406171518690](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230406171518690.png)

      ![](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230406171611035.png)

    * JVM 8 的内存结构

      ![image-20230406171804347](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230406171804347.png)

    

15. 深拷贝和浅拷贝

    * 浅拷贝：只是增加了一个指针指向已经存在的内存地址
    * 深拷贝：增加一个指针，并申请一块新的内存，使这个增加的指针指向这个新的内存
    * 使用深拷贝时，释放内存的时候不会因为出现浅拷贝时释放同一个内存的错误
    * 浅复制：仅仅是指向被复制的内存地址，如果原地址发生改变，那浅复制出来的对象也会相应的改变
    * 深复制：在计算机中开辟一块**新的内存地址**用于存放复制的对象

16. **对象创建的过程**


    1. 类是否加载
       * 判断类是否已经被加载，没有先加载类，详见类的加载过程
    2. 内存是否规整
    
       1. 规整：采用**指针碰撞**
          * 如果Java堆内存是规整，即所有用过的内存放在一边，而空闲的的放在另一边。分配内存时将位于中间的指针指示器向空闲的内存移动一段与对象大小相等的距离，这样便完成分配内存工作
       2. 不规整：采用**空闲列表**
          * 如果Java堆的内存不是规整的，则需要由虚拟机维护一个列表来记录那些内存是可用的，这样在分配的时候可以从列表中查询到足够大的内存分配给对象，并在分配后更新列表记录
    
    3. 并发处理，解决并发问题
       * 对象的创建在虚拟机中是一个非常频繁的行为，哪怕只是修改一个指针所指向的位置，在并发情况下也是不安全的，可能出现正在给对象 A 分配内存，指针还没来得及修改，对象 B 又同时使用了原来的指针来分配内存的情况。解决这个问题有两种方
         * **CAS同步处理**
           * 采用 CAS + 失败重试来保障更新操作的原子性
         * **本地线程分配缓存（TLAB）**
           * 把内存分配的动作按照线程划分在不同的空间之中进行，即每个线程在 Java 堆中预先分配一小块内存，称为本地线程分配缓冲（Thread Local Allocation Buffffer, TLAB）。哪个线程要分配内存，就在哪个线程的 TLAB 上分配。只有 TLAB 用完并分配新的 TLAB 时，才需要同步锁。通过-XX:+/-UserTLAB参数来设定虚拟机是否使用TLAB
    4. 对象初始化

17. 对象访问定位


    1. 句柄访问：堆中划分一个句柄池，引用中存储对象的句柄地址，句柄中包含**对象实例数据**和**对象类型数据**的两个指针。优点是稳定的句柄地址，对象移动时只需改变实例数据指针，引用本身不用修改
    2. 直接指针访问：引用存储的直接是**对象的实例数据**指针，对象实例数据中**包含了对象类型数据指针**。优点是速度更快，节省了一次指针定位的时间。HotSpot采用该方法



## Mybatis

1. 概论

   1.  Mybatis 是一个**半ORM（对象关系映射）框架**，它内部封装了JDBC、加载驱动、创建连接、创建statement 等繁杂的过程，**开发者只需关注如何编写SQL语句**。
   2. **称之为 半ORM 是因为** **在查询关联对象或关联集合对象时，需要手动编写sql来完成**。不像Hibernate这种全自动ORM映射工具，Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取
   3. 通过xml 文件或注解的方式将要执行的各种 statement 配置起来，并通过java对象和 statement中sql的动态参数进行映射生成最终执行的sql语句，最后由mybatis框架执行sql并将结果映射为java对象并返回。（从执行sql到返回result的过程）

2. 优缺点

   * 优点：
     * 与JDBC 相比，减少了50%以上的代码量，消除了JDBC 大量冗余的代码，不需要手动开关连接
     * 基于SQL语句编程，SQL写在XML中，解除了sql和程序代码的耦合，便于统一管理；提供XML标签，支持编写动态SQL语句，并可重用。
     * 很好的与各种数据库兼容（因为Mybatis 使用JDBC来连接数据库，所以只要JDBC支持的数据库MyBatis都支持）
     * 能与Spring 很好的集成
     * 提供映射标签，支持对象与数据库的ORM字段关系映射；提供对象关系映射标签，支持对象关系组件维护
   * 缺点：
     * SQL 编写量较大
     * SQL 依赖于数据库，导致数据库移植性差

3. **#{} 和 ${} 的区别是什么**

   * ${} 是字符串替换，Mybatis 在处理 时把 它**直接替换成变量的值**
   * #{} 是预处理，**使用#{}可以有效的防止SQL注入，提高系统安全性**。Mybatis 在处理 #{} 时，会对 sql 语句进行预处理，将 sql 中的 #{} 替换成 ？，调用 PreparedStatement 的 set 方法来赋值

4. Mybatis 中 Dao 接口 和 XML 文件的 SQL 如何建立关联

   1. 解析XML

      * 首先，Mybatis 在初始化 SqlSessionFactoryBean 时，会找到 mapperLocations 配置的路径下中所有的XML文件并进行解析，这里我们重点关注两部分：

        1. 创建SqlSource

           *  Mybatis会**把每个SQL标签封装成SqlSource对象**，然后根据SQL语句的不同，又**分为动态SQL和静态SQL**。其中，静态SQL包含一段String类型的sql语句；而动态SQL则是由一个个SqlNode组成

        2. 创建 MappedStatement

           * 接下来，Mybatis会为XML中的每个SQL标签都生成一个MappedStatement对象，这里面有两个属性很重要：

             * **id**：全限定类名（XML 中的 namespace）+ 方法名（XML中的 sql 标签的 id）组成的ID
             * **sqlSource：**当前SQL标签对应的SqlSource对象

           * 创建完的 MappedStatement 对象会被添加到 Configuration 中，Configuration对象就是Mybatis中的大管家，基本所有的配置信息都维护在这里。当把所有的XML都解析完成之后，**Configuration就包含了所有的SQL信息**

             ![image-20230318172912097](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230318172912097.png)

               到目前为止，XML就解析完成了，当我们执行Mybatis方法的时候，就可以通过 “`全限定类名+方法名`” 找到 MappedStatement 对象，然后解析里面的SQL内容并进行执行即可

   2. Dao 接口代理

      * 使用 @MapperScan("com.xxx.dao") 注解，将包路径下的所有类注册到 Spring Bean 中，并将它们的beanClass设置为 **MapperFactoryBean**，MapperFactoryBean 实现了 FactoryBean 接口。当我们通过 @Autowired 注入这个Dao接口时，**返回的对象就是 MapperFactoryBean 这个工厂Bean中的 getObject() 方法对象**。这个方法就是通过J**DK动态代理**，返回了一个Dao接口的代理对象MapperProxy，调用Dao接口中的方法时，则会调用 MapperProxy 对象的 invoke() 方法。

   3. 执行

      * 通过statement（`全限定类型+方法名）`拿到MappedStatement 对象，然后通过执行器Executor去执行具体SQL并返回
      
        ![image-20230318173414895](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230318173414895.png)

5. **Mybatis是如何进行分页的？分页插件的原理是什么？**

   * Mybatis使用**RowBounds对象**进行分页，它是**针对ResultSet结果集执行的内存分页**，而非物理分页。**可以在sql内直接书写带有物理分页的参数来完成物理分页功能**，也可以使用分页插件来完成物理分页
   * 分页插件的基本原理是**使用Mybatis提供的插件接口**，实现自定义插件，在插件的拦截方法内**拦截待执行的sql，然后重写sql**，根据dialect方言，添加对应的物理分页语句和物理分页参数



## SpringMVC

1. 执行流程

   * 主要组件

     * 前端控制器（Dispatcher Servlet）：接收请求、响应结果，相当于转发器，有了 DispatcherServlet 就**减少了其它组件之间的耦合度**
     * 处理器映射器（HandlerMapping）：根据请求的URL来查找Handler
     * 处理器适配器（HandlerAdapter）：负责执行 Handler
     * 处理器 （Handler）：处理业务逻辑的 Java 类（也就是 Controller）
     * 视图解析器（ViewResolver）：进行视图解析，根据视图逻辑名将 ModelAndView 解析成真正的视图（View）
     * 视图（View）：View 是一个接口，它的实现类支持不同的视图类型，如 Jsp、freemarker、pdf等

   * 执行流程

     ![image-20230323195015744](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230323195015744.png)

   1. 用户发送请求到前端控制器（DispatcherServlet）。
   2. 前端控制器 （ DispatcherServlet ） 收到请求调用处理器映射器 （HandlerMapping），去查找处理器（Handler）。
   3. 处理器映射器（HandlerMapping）找到具体的处理器(可以根据 xml 配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给 DispatcherServlet。
   4. 前端控制器（DispatcherServlet）调用处理器适配器（HandlerAdapter）。
   5. 处理器适配器（HandlerAdapter）去调用自定义的处理器类（Controller）。
   6. 自定义的处理器类（Controller）将得到的参数进行处理并返回结果给处理器适配器（HandlerAdapter）。
   7. 处理器适配器 （ HandlerAdapter ）将得到的结果返回给前端控制器 （DispatcherServlet）。
   8. 前端控制器（DispatcherServlet ）将 ModelAndView 传给视图解析器 （ViewReslover）。
   9. 视图解析器（ViewReslover）将得到的参数从逻辑视图转换为物理视图并返回给前端控制器（DispatcherServlet）。
   10. 前端控制器（DispatcherServlet）调用物理视图进行渲染并返回。
   11. 前端控制器（DispatcherServlet）将渲染后的结果返回。



补充

1. 设计模式的六大原则

   * 开放封闭原则：对扩展开放，对修改关闭
   * 单一职责原则：一个类的功能应该保证单一，而不能囊括很多
   * 里氏替换原则：子类可以扩展父类的功能，但是不能改变父类原有的功能（设计继承抽象类，而不要继承普通类）。子类可以替换父类出现在父类能在的任何地方
   * 依赖倒置原则：抽象不依赖细节，细节依赖抽象。高层模块不应该依赖底层模块
   * 接口隔离原则：尽量使用多个专门的接口，而不是建立一个臃肿庞大的接口。
   * 迪米特原则：一个类应对其他对象尽可能少的了解

2. 设计模式的日常使用

   1. 策略模式：产品入库有几种不同的入库方式，如生产入库，退货入库，采购入库，其他入库。需要根据不同的入库方式。入库单实体，有个入库方式属性，定义一个统一的接口，然后不同的入库方式分别实现这个接口，在一个公用的处理类通过 ApplicationContext.getBean(入库方式属性)去获取不同的策略bean。

3. Redis 在项目中的使用

   1. 查看若依视频中验证码的实现

4. 前端浏览器兼容问题

   * 不同浏览器的标签默认的外边距和内边距不同
     * 解决方案：CSS 里 *{margin:0;padding:0}
   * 块属性标签 float 后，又有横行的 margin 情况下，IE6 显示 margin 比设置的大
     * 解决方案：在 float 标签样式控制中加入 display:inline; 将其转化为 行内属性
   * Vue 在 IE 浏览器上运行时会出现兼容问题：JS 语法报错、Css 样式错乱。原因是 IE11 浏览器下，部分JS 写法需要改变，部分样式IE不支持。
     * 解决方案：
       * 解决 JS 兼容性：**babel-polyfill**（npm install 后，在 main.js 最顶部 import，确保全面加载），**es6-promise**（npm install ，require('es6-promise').ployfill ） 以及 Babel-plugin-transform-es2015-modules-commonjs （上两个因为 require 和 import 混用，**打包出现问题，用这个插件解决**）

5. 域名解析

   1. 两种方式识别主机：**主机名、IP地址**。人们喜欢记忆主机名，路由器喜欢 IP 地址。**域名系统将 域名和IP地址 相互映射**

   2. DNS 服务器的部分层次结构，从上到下依次为**根域名服务器、顶级域名服务器和权威域名服务器**、**本地域名服务器**

   3. 解析过程：递归查询、迭代查询

      1. 递归查询：主机 -> 本地域名服务器递归查询 -> 本地域名 采用递归查询的方式向 根域名 查询 -> 根 收到委托后，用 递归查询 某个 顶级域名 -> 收到委托后，递归查询的方式 向某个 权限域名。
      2. 迭代查询：主机 向 本地域名 递归查询 -> 本地采用  迭代查询 问 根域名 -> 根 告诉 本地 下一次查询 哪个 顶级域名 -> 本地 迭代查询 顶级 -> 顶级 告诉 本地 下一次查询 哪个 权限域名 -> 本地 迭代查询 权限域名 -> 权限域名 告诉 本地 IP -> 本地告诉 主机 IP（**递归查询对服务器负担大，通常采用：从主机到本地用 递归，其余用迭代**）
      3. 区别：
         1. 递归：如果主机询问本地，本地不知道，本地就会代替主机去问根服务器
         2. 迭代：当根收到本地的迭代查询请求，要么给IP，要么告诉本地域名 下一步去哪里查询
      4. 高速缓存：为了**提高DNS的查询效率**，并**减轻根域名服务器的负荷**和减少因特网上的DNS查询报文数量，在域名服务器中广泛地使用了**高速缓存**。高速缓存**用来存放最近查询过的域名以及从何处获得域名映射信息的记录。**域名服务器**应为每项内容设置计时器并删除超过合理时间的项**（例如，每个项目只存放两天)

   4. 访问网址发生了什么

      1. DNS 解析：用户输入网址，浏览器获取到一个域名，实际需要一个 IP 地址。通过DNS 解析把域名 变成 地址
      2. TCP 连接：浏览器 通过 DNS 获取 Web 服务器地址后，向 服务器发起 TCP 连接请求
      3. 发送Http请求：通过三次握手建立连接后，浏览器将 Http 请求发送给服务器
      4. 处理请求并返回：服务器收到 Http 请求后，根据 Http 获取相应的文件，把文件发给 浏览器
      5. 浏览器渲染：浏览器根据响应开始显示页面，解析 HTML 文件构建 DOM 树，然后解析 CSS ，渲染树构建完成后，浏览器开始布局并绘制到屏幕上
      6. 断开连接：客户端和服务器端四次挥手断开连接

   5. Spring IOC

      1. 由 IOC 容器帮助对象找相应的依赖并注入，并不是由对象主动去找；资源集中管理，实现资源的可配置和易管理；降低了使用资源双方的依赖程度、松耦合

   6. 进程和线程

      1. 进程是操作系统进行资源分配的最小单元，线程是操作系统进行运算调度的最小单元，线程是进程中实际运行的单位，每个进程都有自己的内存和资源，进程中的线程会共享这些内存和资源。进程的创建、销毁和切换的开销远比线程大

      2. 进程之间的通信

         1. **管道**：**内核中的一串缓存**。管道传输的数据是单向的，如果需要相互通信，需要创建两个管道

         2. **消息队列**：A进程给B进程发送消息，A进程把数据放在对应的消息队列后就可以正常的返回，B进程需要的时候再进行读取数据。消息队列是保存在内存中的消息链表，消息的发送和接收方约定消息体的数据类型，每个消息体是固定大小的存储块。消息体被读取后，内核就会删除这个消息体（消息队列不适合比较大的数据传输，消息题有大小限制，队列也有长度限制。存在用户态和内核态之间的数据拷贝开销）

         3. **共享内存**：拿出一块虚拟化的地址空间，映射到相同的物理内存中。这个进程写入，另一个进程可以看到（会有多进程竞争共享资源）

         4. **信号量**：一个整型的计数器，用于实现进程间的互斥和同步，而不是用于缓存进程间通信的数据。**信号量表示资源的数量，控制信号量的方式有两种原子操作**：

            1. P操作：信号量 -1。< 0 表示资源被占用，进程需要阻塞等待，>= 0 资源可用，进程可继续执行
            2. V操作：信号量 +1。<= 0，表示当前有阻塞中的进程，将其唤醒运行；>0 ，表明当前没有阻塞中的进程
         5. Socket：不同主机之间的进程通信
      
      3. 线程之间通信
      
         1. 等待通知：两个线程对同一对象调用 等待wait 和通知 notify 方法来进行通讯
         2. join() ：在 join 线程完成后会调用 notifyAll() 方法，是在 JVM 实现中调用，所以这里看不出来
         3. volatile ：共享内存
         4. 管道通信
         5. 并发工具：CountDownLatch 并发工具
      
      4. 线程状态切换
      
         ![image-20230323221657161](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230323221657161.png)
      
      5. 创建线程三种方式
      
         1. 继承 Thread，重写 run
         2. 实现 Runable new Thread(new Runable { 重写 run})
         3. 实现 Callable：new FutureTask<>(() -> {重写run})，new Thread(FutureTask)
      
      6. 线程池工作原理
      
         1. 创建线程池，线程数为 0
         2. 当调用 execute 添加一个请求任务时，做判断：
            1. 当前线程数 < 核心线程数，创建线程完成任务
            2. 当前线程数 >= 核心线程数，任务放在队列中
            3. 队列满了，并且 正在运行的线程数 < 最大线程数，创建非核心线程运行队列中的任务
            4. 如果 队列满了，当前线程数 >= 最大线程数，采用 拒绝策略
         3. 一个线程完成任务后，从队列取一个任务执行
         4. 一个线程无事可做，超过 keepAliveTime 时，线程判断：当前线程数 > 核心线程数，该线程停掉
      
      7. 锁
      
         1. 分类
            * 乐观锁和悲观锁
              * 悲观锁：认为对同一个数据的并发操作，一定会发生修改，采取加锁的形式，认为不加锁数据一定会被修改
              * 乐观锁：读数据时认为不会被修改，更新数据时，通过判断现有数据和原数据是否一致来判断是否被其他线程操作，如果没污染就更新，污染了就不更新
            * 公平锁和非公平锁
              * 公平：排队，多个线程按申请锁的顺序来获取锁
              * 非公平：不排队，抢占式
              * ReentrantLock 提供公平和非公平锁实现
                * 公平锁：new ReentrantLock(true)
                * 非公平锁：new ReentrantLock(false)
                * 非公平锁：new ReetrantLock() ，默认构造器为非公平锁
            * 独占锁和共享锁
              * 独占锁：任何适合只有一个线程能执行资源操作
              * 共享锁：可以被多个线程读取，但只能一个线程修改
              * ReentrantReadWriteLock 共享锁
            * 可重入锁
              * 该线程获取该锁之后，可以无限次进入该锁
            * 自旋锁
              * 尝试获取锁的线程不会立即阻塞，而是采用循环的方式尝试获取锁。好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU
         2. CAS 和 ABA
            1. CAS：比较和交换。乐观锁实现，java.util.concurrent.AtomicInteger 就是借助 CAS 实现
            2. CAS 无法解决 ABA 问题：数据 A 原先是 200，线程一和线程二同时比对数据进行 -100 操作，线程一执行完成，线程三突然抢在线程二之前执行，把数据改回200，线程二这个时候认为数据没有被修改过，也执行 -100 操作
            3. 解决方案：添加 版本号
         3. 非公平锁吞吐量大于公平锁，为什么
            1. A 占用锁，B请求获取时阻塞，等待唤醒。这时有个 C线程，公平锁的话，C 排在 B 后面等待被唤醒。而非公平锁的话，C 可以抢先用，B 唤醒之前 C 可能已经使用完成了。从而节省了 C 等待和被唤醒的性能消耗
         4. volatile 的作用
            1. 保证变量对所有线程的可见性
            2. 禁止指令重排序优化
            3. Synchronized 既可以保证可见性，又能保证原子性。而volatile 只能保证可见性
         5. 锁升级（只升不降）
            1. **对象头**有记录持有锁的状态
            2. 无锁 -> 偏向锁：**锁第一次被线程获取，进入偏向锁**，记录该线程ID，此后**该线程每次进入该锁都不需要进行任何同步操作**，如果从头到尾只有这一个线程使用，**偏向锁几乎没有额外开销，性能很高**。当有其他线程竞争锁时，偏向状态失效。
            3. 偏向锁 -> 轻量锁：**第二个线程竞争时，升级为轻量级锁（自旋锁）**。**没抢到锁的线程自旋（忙等）**。如果多个线程使用独占锁，但是没有发生锁竞争，或者轻微的锁竞争，那 synchronized 是轻量锁，**允许短时间忙等，换取线程在用户态和内核态之间切换的开销**
            4. 轻量锁 -> 重量锁：**锁竞争严重，线程自旋次数达到最大，锁升级**。后续线程尝试获取锁时，发现被占用的锁是重量级锁，则**直接将自己挂起**，等待释放锁的线程去唤醒。
      
      8. Mybatis 分页实现：
      
         1. 数组：查询所有记录，保存到临时数组中，再通过 List 的 subList 方法，获取满足条件的所有记录
         2. 借助 Sql 语句进行分页：在mapper.xml 中的 sql 语句中，通过改变 limit 中 offset 和 pagesize 来实现
         3. 拦截器：最优方案，一劳永逸。@Intercepts 定义拦截器，继承 Interceptor，pageSize 属性 和 currentPage 属性，重写 intercept 方法，MappedStatement.matches(".+ByPage$")，拦截所有以 .ByPage 结尾的请求
         4. RowBounds 实现分页
            * 和 数组实现方式原理差不多，只是数组分页需要自己去实现分页逻辑，RowBounds 简化了而已。和数组方式一样，对内存消耗很大，甚至引发内存溢出。数据量大情况下，用拦截器
            * dao 层 加入 RowBounds 参数，在sevice 层 通过 offset 和 limit 两个参数构建 RowBounds。
      
      9. Spring，SpringMVC，SpringBoot 的区别
      
         1. Spring 是一个 一站式的轻量级Java 开发框架，核心是IOC 和 AOP
         2. SpringMVC 在 Spring 基础上加了 MVC 框架，处理web开发的路径映射和视图渲染
         3. SpringBoot 简化了配置流程，与前端解耦。
      
      10. SpringCloud 组件
      
         1. Eureka 服务发现（注册中心）：Eureka Client 服务注册和 Eureka Server 保存各服务所在的机器和端口号，负责发现
         2. Feign 负责和其他服务建立网络连接，动态代理
            * 首先在接口上定义FeignClient 注解，Feign 针对这个接口创建一个动态代理
            * 节奏调用这个接口时，本质就会调用 Feign 创建的动态代理
            * Feign 动态代理根据接口上的 @RequestMapping 等注解，来动态构造你要请求的服务地址
            * 最后，针对这个地址，发起请求，解析响应
         3. Ribbon 客服端负载均衡，采用 轮询算法
         4. Hystrix 隔离，熔断以及降级：每个服务一个线程池，其中一个服务A挂了，其他需要调用这个服务的服务B也就无法工作，但是B调用的C、D服务可用。这时请求时，B 正常调用 C、D，只是调用 A 时，每次都报错，可以将A **熔断**（比如5分钟内请求直接返回），那么可以 B 向正常用着，在数据库中记录这条消息，等A恢复后，根据记录手工执行那些消息，这就是**降级**。
         5. Zuul 服务网关，负责网络路由的
      
      11. 大文件查看
      
          1. 文件的行数：wc -l filename
          2. 文件开头：head -n filename （n 为行数）
          3. 文件结尾：tail -n filename
          4. 输出指定行的内容：sed -n ‘10，10000p’ big_logs_file
          5. 动态加载到内存：more，less，将文件部分内容加载到内存中
          6. 文件分割：split 
      
      12. get 和 post 请求的区别
      
          * get 把 参数包含在 URL 中，Post 通过 request body 传递参数
          * get 一般会被缓存，如常见的  CSS、JS、HTML 请求等都会被缓存；而 POST 请求默认是不进行缓存的。
          * GET 请求的参数是通过 URL 传递的，而 URL 的长度是有限制的，通常为 2k；POST 请求参数是存放在请求正文（request body）中的，所以没有大小限制。
          * GET 请求可以直接进行回退和刷新，不会对用户和程序产生任何影响；而 POST 请求如果直接回滚和刷新将会把数据再次提交
          * GET 请求的参数会保存在历史记录中，而 POST 请求的参数不会保留到历史记录中
          * GET 请求的地址可被收藏为书签，而 POST 请求的地址不能被收藏为书签
      
      13. http 和 https 的区别
      
          1. http 是超文本传输协议，信息是明文传输；https 具有安全性的 ssl/tls 加密传输协议。
          2. https 需要 CA 申请书，一般免费证书较少，因而需要一定的费用
          3. http 端口 80，https 端口 443
          4. http 连接简单，是无状态的



## 计算机网络

1. HTTP 超文本传输协议（应用层）

   * 端口：80
   * 采用TCP，是可靠的数据传输协议，**浏览器向服务器发收报文前，先建立TCP连接，HTTP使用TCP连接方式（HTTP自身无连接）**
   * **HTTP请求报文方式：**
     1. **GET**：请求指定的页面信息，并返回实体主体；（获取指定的服务端资源）
     2. **POST**：向指定资源提交数据进行处理请求；（提交数据到服务端）
     3. **DELETE**：请求服务器删除指定的页面；（删除指定的服务端资源）
     4. **HEAD**：请求读取URL标识的信息的首部，**只**返回报文头；
     5. **OPETION**：请求一些选项的信息；
     6. **PUT**：在指明的URL下存储一个文档
     7. **UPDATE**：更新指定的服务端资源

2. HTTPS  （S -> Secure，安全的HTTP协议）

   * 端口：443
   * 基于HTTP协议，通过SSL或TLS提供加密处理数据、验证对方身份以及数据完整性保护。

3. 应用层常用服务

   1. 应用层：为操作系统或网络应用程序提供访问网络服务的接口
   2. 数据传输基本单位是：报文
   3. **包含的主要协议：FTP（文件传送协议）、Telnet（远程登录协议）、DNS（域名解析协议）、SMTP（邮件传送协议），POP3协议（邮局协议），HTTP协议（Hyper Text Transfer Protocol）**

4. UDP

   * 用户数据报协议

   * 特点：

     * **无连接**：发送数据之前不用建立连接，发送之后不用断开连接
     * **不能保证可靠的交付数据**：主机不用维护复杂的连接状态表
     * **面向报文传输**：发送方的 UDP 对应用程序交下来的报文，在添加首部后就向下交付 IP 层。UDP 对应用层交下来的报文，既不合并，也不分拆，而是保留这些报文的边界。这就是说，应用层交给 UDP 多长的报文，UDP 就照样发送，即一次发送一个报文
     * **没有拥塞控制**：因此**网络出现的拥塞不会使源主机的发送速率降低**。这对某些实时应用是很重要的。**很多的实时应用（如：IP电话、实时视频会议等）要求源主机以恒定的速率发送数据**，并且允许在网络出现拥塞时丢失一部分数据，但却不允许数据有太大的时延。UDP 协议正好适合这种要求。
     * 支持一对一、一对多、多对多的交互通信
     * 首部开销只有 8 字节，TCP 要 20 字节首部

   * TCP

     * 传输控制协议

     * 特点：

       * 面向连接的
       * 面向字节流
       * 点对点通信
       * **可靠传输**：
         * 差错检测：利用编码实现数据包传输过程中的比特差错检测
         * **确认**：接收方向发送方反馈接收状态
         * **重传**：发送方重新发送接收方没有正确接收的数据
         * **序号**：确保数据按序提交
         * **计时器**：解决数据丢失问题
       * 全双工通道（每条TCP连接只能一对一）

     * **流量控制**（通过**滑动窗口**实现）

       ![image-20230403230527530](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230403230527530.png)

     * 拥塞控制

       * 拥塞控制与流量控制的区别：流量控制考虑**点对点的通信量的控制**，而拥塞控制考虑**整个网络**，是全局性的考虑

       * 拥塞控制的方法：**慢启动算法+拥塞避免算法**

       * **慢开始和拥塞避免：**

         1. 【**慢开始**】拥塞窗口从1指数增长；
         2. 到达阈值时进入【**拥塞避免】**，变成+1增长；
         3. **【超时】**，阈值变为当前cwnd的一半（不能<2）；
         4. 再从【**慢开始**】，拥塞窗口从1指数增长

         ![image-20230403230639523](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230403230639523.png)

       * **快重传和快恢复：**

         1. 发送方连续收到**3个冗余ACK**，执行【**快重传**】，不必等计时器超时；
         2. 执行【**快恢复**】，阈值变为当前cwnd的一半（不能<2），并从此**新的ssthresh**点进入【**拥塞避免】**。

         ![image-20230403230735070](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230403230735070.png)

         





分布式系统



微服务



软件开发流程



## Linux 命令 和 Shell 脚本

1. **[root@localhost ~]#** 的含义

   1. @之前的是当前登录的用户
   2. localhost是主机名字
   3. ~当前所在的位置（所在的目录）
   4. ~家目录
   5. /根目录
   6. \#的位置是用户标识
      - \#是超级用户
      - $普通用户

2. Linux命令写法：命令名 [选项]  [参数]

3. Shell 脚本

   1. 开头：#!/bin/bash
   2. 变量规则：不能以数字、特殊符号开头，可以使用下划线 _，不能使用破折号 -
   3. echo : 输出，System.out.println
   4. 系统变量：
      * $0 ：当前脚本的名称
      * $n ：当前脚本的第 n 个参数，n = 1，2， 3。。。
      * $* ：当前脚本的所有参数（不包括程序本身）
      * $# ：当前脚本的参数个数（不包括程序本身）
      * $? ：命令或程序执行完成后的状态，返回 0 表示执行成功
      * $$ ：程序本身的 PID 号
   5. 环境变量：PATH、HOME、USER、PWD、HOSTNAME

4. top 命令

   1. 类似于Windows的任务管理器

      ![image-20230404142025019](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230404142025019.png)

   2. uptime 命令，直接输入uptime

      ![image-20230404142103336](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230404142103336.png)

      系统时间，42min表示系统运行时间，1 user表示登录用户数，load average:0.50,0.64,0.46表示**最近1分钟，5分钟，15分钟**系统平均载荷。

5. chmod

   1. 读，写，执行 ：4，2，1
   2. 三个组，分别是 user（u，文件所属用户），group（g，所属组用户），other（o，其他用户）
   3.  chmod g + x 文件名 ：表示给 group 添加 x权限，如果用 g - x 表示删除 x 权限

6. 
