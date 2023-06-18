### 面向对象编程

Alan Kay 总结了 Smalltalk 语言的**5大特征**，这些特征代表了**纯粹的面向对象编程的方式**。

* **万物皆对象**。可以把对象想象为一种神奇的变量，它可以存储数据，同时可以“发出请求”，让它执行一些操作。对于你想要解决的问题中的任何元素，都可以在程序中用对象来呈现；
* **一段程序实际上就是多个对象通过发送消息来通知彼此要干什么**。当你向一个对象 “发送消息” 时，实际情况是你发送了一个请求去调用该对象的某个方法；
* **从内存角度而言，每一个对象都是由其他更为基础的对象组成的**。换句话说，通过将现有的几个对象打包在一起，你就创建了一种新的对象。这种做法展现了对象的**简单性**，同时隐藏了程序的**复杂性**；
* **每一个对象都有类型**。具体而言，每一个对象都是通过某个类生成的实例，这里说的 “类” 就（几乎）等同于 “类型”。**一个类最显著的特性是 “你可以发送什么消息给它”**
* **同一类型的对象可以接收相同的消息**。举例来说，因为一个 “圆形” 对象同样也是一个 “形状” 对象，所以 “圆形” 也可以接收 “形状” 类型的消息。这就意味着，你为 “形状” 对象编写的代码自然可以适用于任何的 “形状” 子类对象。这种**可替换性**是面向对象编程的一个基石

### 访问修饰符

* **public**: 表示定义的内容可以被所有人访问
* **protect**: 对同一个包内的类和所有子类可见
* **private**: 在同一个类内可见
* **缺省**：同一个包内可见

### 基本类型

* 无须使用 new 来创建基本类型的变量，而是直接创建一个 “自动变量” ，注意**不是引用** 

* 即使是在不同的机器上，**这些类型所占用的空间也保持一致**。而这种一致性也是 Java 程序比其他语言程序移植性更好的原因之一

* 所占用的空间大小：

  | 基本类型 |     大小      | 最小值  | 最大值  |  包装类   |
  | :------: | :-----------: | :-----: | :-----: | :-------: |
  | boolean  |      ——       |   ——    |   ——    |  Boolean  |
  |   char   | 16位（2字节） | \u0000  | \uffff  | Character |
  |   byte   | 8位（1字节）  |  -128   |   127   |   Byte    |
  |  short   | 16位（2字节） | -32768  |  32767  |   Short   |
  |   int    | 32位（4字节） |  -2^31  | 2^31-1  |  Integer  |
  |   long   | 64位（8字节） |  -2^63  | 2^63-1  |   Long    |
  |  float   | 32位（4字节） | IEEE754 | IEEE754 |   Float   |
  |  double  | 64位（8字节） | IEEE754 | IEEE754 |  Double   |

* 所有数值类型都是**有符号的**

* boolean 类型的**空间大小没有明确标出**，其对象只能被赋值为 true 或 false

* 包装类：

  ~~~java
  // 通过包装类可以将基本类型呈现为位于堆上的非原始对象
  char c = 'x';
  Character ch = new Character(c);
  // 或者
  Character ch = new Character('x');
  
  // 自动装箱 机制能够将基本类型对象自动转换为包装类对象
  Character ch = 'x';
  // 也可以再转换回来
  char c = ch;
  ~~~

* 高精度数字：Java 提供了两个**支持高精度计算的类**，分别是 **BigInteger** 和 **BigDecimal**。由于涉及更多的计算量，导致的结果就是相关操作的效率有所降低。

  * BigInteger 可以支持任意精度的**整数**。也就是说，在运算时你可以表示任意精度的整数值，而不用担心丢失精度
  * BigDecimal 可用于任意精度的**定点数**，可用于**货币计算**

### 基本类型的默认值

* 当变量作为**类成员**存在时，Java 才会将其初始化为以下的**默认值**：

  | 基本类型 |    默认值     |
  | :------: | :-----------: |
  | boolean  |     false     |
  |   char   | \u0000 (null) |
  |   byte   |   (byte) 0    |
  |  short   |   (short) 0   |
  |   int    |       0       |
  |   long   |      0L       |
  |  float   |     0.0f      |
  |  double  |     0.0d      |

* 这种机制并**不会应用于局部变量**，因为局部变量并不是类的字段。

  ~~~java
  int x;
  // 变量 x 可能是一个任意值，而不会自动被初始化为 0
  // 因此，在使用变量 x 之前，必须为其赋值以确保正确性
  ~~~

### 数组

* Java 数组一定会被初始化，并且无法访问数组边界之外的元素。这种边界检查的代价是需要消耗少许内存，以及运行时需要少量时间来验证索引的正确性。其背后的假设是，安全性以及生产力的改善完全可以抵消这些代价
* 当创建一个用于放置对象的数组时，实际上**数组里包含的是引用**，而这些引用会被自动初始化为一个**特殊值 ： null**。
* 创建一个放置基本类型的数组。编译器会**确保对该数据进行初始化**，并将数组元素的值设置为 **0**

### 方法签名

* **方法名**和**参数列表** 共同构成了方法的**"签名"** ，方法签名即该方法的**唯一标识符**

### static 

* 第一种情况，有时我们需要一小块共享空间来保存某个特定的字段，而并不关心创建多少个对象，甚至有没有创建对象。
* 第二种情况，你需要使用一个类的某个方法，而该方法和具体的对象无关；换句话说，你希望即便没有生成任何该类的对象，依然可以调用其方法
* **不能在没有具体对象的情况下，使用static方法直接调用非static成员或方法**（因为非 static 成员和方法必须基于特定的对象才能运行）

### 对象赋值

* 当操作一个对象的时候，我们**真正操作的是这个对象的引用**。所以当 “将一个对象赋值给另一个对象” 时，**其实是将这个引用从一个地方复制到另一个地方**

* **别名**

  ~~~java
  Tank t1 = new Tank();
  Tank t2 = new Tank();
  t1.level = 9;
  t2.level = 47;
  t1 = t2;		// 赋值操作的是引用，所以现在 t1 和 t2 共同指向一个对象，产生了 “别名”
  t1.level = 100;	
  System.out.println(t2.level) 	// 输出：100
  ~~~

* 方法调用中的别名：**将一个对象作为参数传递给方法时，也会产生别名**

  ~~~java
  class Letter {
      char c;
  }
  
  public class PassObject() {
      static void f(Letter y) {
          y.c = 'z';
      }
      
      public static void main(String[] args) {
          Letter x = new Letter();
          x.c = 'a';
          System.out.println("1. x.c: " + x.c)		// 输出：1. x.c: a
          f(x)	// 传递的是引用
          System.out.println("2. x.c: " + x.c)		// 输出：2. x.c: z
      }
  }
  ~~~

### 算数操作符

* 加减乘除：整数除法的结果会**舍弃小数位**，而不是四舍五入

* `Random`：`Random.nexInt()` 方法的参数设置了所生成随机数的上限，其下限是 0

* 一元操作符：一元减号 会 反转数据的符号。一元加号和一元减号相对应，它唯一的作用就是将较小类型的操作数提升为 int 类型

* 自动递增：++a : 先运算，后返回结果； a++ : 先返回结果，再运算

* 关系运算符：等于 和 不等于 适用于所有的基本数据类型，但其他比较操作符不适用于 boolean 类型

* 操作符 == 和 != 比较的是**对象的引用**。

* **Integer**：对于值范围 在 -128~127 的 Integer 类型来说，它生成的对象是相同的：**出于效率问题，Integer 会通过享元模式来缓存范围在 -128~127 内的对象，因此多次调用 `Integer.valueOf(127)` 生成的其实是同一个对象**。另外，**通过 `new Integer()` 生成的对象都是新创建的，无论其值处于什么范围。**

  > Java 9 及更新版本中已经弃用 `new Integer()` ，因为它的效率远远低于 `Integer.valueOf()`

* 科学记数法：e 表示 “10 的幂次”。编译器一般会将指数作为 double 类型处理，所以如果没有尾部的 f，我们会收到一条出错提示，告诉我们必须将double 类型转换成 float 类型

### String

* String 表示字符串类型，属于**引用数据类型**，不属于基本数据类型

* 双引号括起来的字符串，是 **“不可变”** 的，直接存储在**方法区**的**字符串常量池**中。原因是字符串在实际的开发中使用太频繁。为了执行效率，所以把字符串放到了方法区的字符串常量池中

  ~~~java
  public class StringTest02 {
      public static void main(String[] args) {
          String s1 = "hello";	// "hello"存储在方法区的字符串常量池当中
          String s2 = "hello";	//这个"hello"不会新建。（因为这个对象已经存在了！）
          
          // == 双等号比较的是变量中保存的内存地址
          System.out.println(s1 == s2); // true
  
          //x 和 y 两个对象变量的值各自存放在堆中
          String x = new String("xyz");
          String y = new String("xyz");
          
          // == 双等号比较的是变量中保存的内存地址，内存地址不同，所以 false
          System.out.println(x == y); //false
      }
  }
  ~~~

  ![image-20230228155010772](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230228155010772.png)

* 注意：字符串对象之间的比较不能使用“== ”,"=="不保险。应该调用[String类](https://so.csdn.net/so/search?q=String类&spm=1001.2101.3001.7020)的`equals()`方法

  ~~~java
  String k = new String("testString");
  //String k = null;
  // "testString"这个字符串可以后面加"."呢？
  // 因为"testString"是一个String字符串对象。只要是对象都能调用方法。
  System.out.println("testString".equals(k)); // 建议使用这种方式，因为这个可以避免空指针异常。
  System.out.println(k.equals("testString")); // 存在空指针异常的风险。不建议这样写。
  ~~~

* 为什么 **System.out.println(引用)** 会自动调用toString()方法？

  因为println()会调用String.**valueOf**()方法而String.valueOf()会调用toString()方法

* [String 全面讲解](https://blog.csdn.net/m0_58761900/article/details/125014074)

* String 源码分析

  ~~~java
  // 实现 Serializable（可序列化）, Comparable 接口
  public final class String
      implements java.io.Serializable, Comparable<String>, CharSequence {
      
      // final 修饰的 char[] ，说明字符串为 不可变
      private final char value[];
  
      // hash 默认值 0
      private int hash; 
  
      private static final long serialVersionUID = -6849794470754667710L;
      
      // equals()，比较的是字符串的内容
      public boolean equals(Object anObject) {
          // 1. 先判断是否是同一个对象
          if (this == anObject) {
              return true;
          }
          // 2. 判断是否是 String 类或子类
          if (anObject instanceof String) {
              // 是，可以转型
              String anotherString = (String)anObject;
              int n = value.length;
              // 3. 判断字符串长度，不相等肯定就是 false
              if (n == anotherString.value.length) {
                  char v1[] = value;
                  char v2[] = anotherString.value;
                  int i = 0;
                  // 4. 循环，比较各字符是否相等
                  while (n-- != 0) {
                      if (v1[i] != v2[i])
                          return false;
                      i++;
                  }
                  return true;
              }
          }
          return false;
      }
  
      // hashCode() 方法
      public int hashCode() {
          int h = hash;
          // 1. 看 hash值 是否已经计算过了，并判断 value 是否是空数组
          if (h == 0 && value.length > 0) {
              char val[] = value;
  
              // 循环
              // 31 作为乘积因子的原因：
           	// 		1. 31 是一个不大不小的质数，是作为hashCode乘积因子的优选质数之一
              // 		2. 31 可以被 JVM 优化：31 * i = （i << 5）- i，因为移位运算比乘法运行更快更省性能 
              for (int i = 0; i < value.length; i++) {
                  h = 31 * h + val[i];
              }
              // 将计算后的 hash 值赋值给 hash，这样就只需要计算一次 hash 值了
              hash = h;
          }
          return h;
      }
      
      // compareTo() 
      // 按字母顺序比较两个字符串，是基于字符串中每个字符的 Unicode 值。
      // 当两个字符串某个位置的字符不相同时，返回的是这一位置的字符 Unicode 值之差
      // 当两个字符串都相同时，返回两个字符串长度之差
      public int compareTo(String anotherString) {
          int len1 = value.length;
          int len2 = anotherString.value.length;
          int lim = Math.min(len1, len2);
          char v1[] = value;
          char v2[] = anotherString.value;
  
          int k = 0;
          while (k < lim) {
              char c1 = v1[k];
              char c2 = v2[k];
              if (c1 != c2) {
                  return c1 - c2;
              }
              k++;
          }
          return len1 - len2;
      }
      // compareToIgnoreCase() 方法在compareTo()方法的基础上忽略大小写，大写字母比小写字母的Unicode值小32，底层实现是先都转换成大写比较，然后都转换成小写比较
      
      // 当调用intern方法时，如果池中已经包含一个与该String确定的字符串相同equals(Object)的字符串，则返回该字符串。否则，将此String对象添加到池中，并返回此对象的引用。
  　　// 这句话什么意思呢？就是说调用一个String对象的intern()方法，如果常量池中有该对象了，直接返回该字符串的引用（存在堆中就返回堆中，存在池中就返回池中），如果没有，则将该对象添加到池中，并返回池中的引用。示例：
      // String str1 = "hello";//字面量 只会在常量池中创建对象
   	// String str2 = str1.intern();
      // System.out.println(str1==str2);//true
      public native String intern();
  }
  ~~~

* “ + ” 连接符 对字符串提供特殊支持

  ~~~java
  String s1 = "abc";
  String s2 = new String("java");
  String s3 = s1 + s2;
  
  // s3 的实现原理是，先 创建一个 StringBuilder() 对象，并调用append()方法将数据拼接，最后调用toString() 方法返回拼接好的字符串
  ~~~

* 字符串字面量拼接编译期优化：

  ![image-20230301111627563](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230301111627563.png)

* StringBuilder、StringBuffer 的区别

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