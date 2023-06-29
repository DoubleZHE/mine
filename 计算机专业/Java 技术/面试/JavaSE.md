# 面向对象编程

Alan Kay 总结了 Smalltalk 语言的**5大特征**，这些特征代表了**纯粹的面向对象编程的方式**。

* **万物皆对象**。可以把对象想象为一种神奇的变量，它可以存储数据，同时可以“发出请求”，让它执行一些操作。对于你想要解决的问题中的任何元素，都可以在程序中用对象来呈现；
* **一段程序实际上就是多个对象通过发送消息来通知彼此要干什么**。当你向一个对象 “发送消息” 时，实际情况是你发送了一个请求去调用该对象的某个方法；
* **从内存角度而言，每一个对象都是由其他更为基础的对象组成的**。换句话说，通过将现有的几个对象打包在一起，你就创建了一种新的对象。这种做法展现了对象的**简单性**，同时隐藏了程序的**复杂性**；
* **每一个对象都有类型**。具体而言，每一个对象都是通过某个类生成的实例，这里说的 “类” 就（几乎）等同于 “类型”。**一个类最显著的特性是 “你可以发送什么消息给它”**
* **同一类型的对象可以接收相同的消息**。举例来说，因为一个 “圆形” 对象同样也是一个 “形状” 对象，所以 “圆形” 也可以接收 “形状” 类型的消息。这就意味着，你为 “形状” 对象编写的代码自然可以适用于任何的 “形状” 子类对象。这种**可替换性**是面向对象编程的一个基石

# 访问修饰符

* **public**: 表示定义的内容可以被所有人访问
* **protect**: 对同一个包内的类和所有子类可见
* **private**: 在同一个类内可见
* **缺省**：同一个包内可见

# 基本类型

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

## 基本类型的默认值

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

# 数组

* Java 数组一定会被初始化，并且无法访问数组边界之外的元素。这种边界检查的代价是需要消耗少许内存，以及运行时需要少量时间来验证索引的正确性。其背后的假设是，安全性以及生产力的改善完全可以抵消这些代价
* 当创建一个用于放置对象的数组时，实际上**数组里包含的是引用**，而这些引用会被自动初始化为一个**特殊值 ： null**。
* 创建一个放置基本类型的数组。编译器会**确保对该数据进行初始化**，并将数组元素的值设置为 **0**

# 方法签名

* **方法名**和**参数列表** 共同构成了方法的**"签名"** ，方法签名即该方法的**唯一标识符**

# static

* 第一种情况，有时我们需要一小块共享空间来保存某个特定的字段，而并不关心创建多少个对象，甚至有没有创建对象。
* 第二种情况，你需要使用一个类的某个方法，而该方法和具体的对象无关；换句话说，你希望即便没有生成任何该类的对象，依然可以调用其方法
* **不能在没有具体对象的情况下，使用static方法直接调用非static成员或方法**（因为非 static 成员和方法必须基于特定的对象才能运行）

# 对象赋值

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

# 运算符

Java 运算符大致分为逻辑运算符 `&&,||,!`、算数运算符 `+, -, *, / ,+=`、位运算符 `^,|,&`、其他运算符（三元运算符）。

## 逻辑运算符

### 逻辑与 &&

`&&` 逻辑与也称为短路逻辑与，先运算 `&&` 左边的表达式，一旦为假，后续不管多少表达式，均不再计算，一个为真，再计算右边的表达式，两个为真才为真。

### 逻辑或 ||

逻辑或 `||` 的运算规则是一个为真即为真，后续不再计算，一个为假再计算右边的表达式。

## 算数运算符

## 位运算符

### 按位与 &

`&` 按位与的运算规则是将两边的数转换为二进制位，然后运算最终值，运算规则即：对应位置的两个数据都为 1 才为 1。
$$
1 \& 1 = 1, 1 \& 0 = 0, 0 \& 1 = 0, 0 \& 0 = 0
$$

3 的二进制位是 0000 0011 ， 5 的二进制位是 0000 0101 ， 那么就是 $011 \& 101$，由按位与运算规则得知，$001 \& 101 = 0000 0001$，最终值为 1

7 的二进制位是0000 0111，那就是 $111 \& 101 = 101$，也就是 0000 0101，故值为 5

### 异或运算符 ^

`^` 异或运算符顾名思义，异就是不同，其运算规则为：对应位置，不同时取 1，相同时取 0，0 相同时取 0。（1 ^ 0 = 1 , 1 ^ 1 = 0 , 0 ^ 1 = 1 , 0 ^ 0 = 0）

5的二进制位是0000 0101 ， 9的二进制位是0000 1001，也就是0101 ^ 1001,结果为1100 , 00001100的十进制位是12

### 按位或 |

`|` 按位或和 `&` 按位与计算方式都是转换二进制再计算，不同的是运算规则：对应位置的一个数据为 1 即为 1。
$$
1 | 0 = 1 , 1 | 1 = 1 , 0 | 0 = 0 , 0 | 1 = 1
$$

6 的二进制位 0000 0110 , 2 的二进制位 0000 0010 , $110 | 010 = 110$，最终值 0000 0110，故 $6 | 2 = 6$

### 左移运算符 <<

`5 << 2` 的意思为 5 的二进制位往左挪两位，右边补 0，5 的二进制位是 0000 0101 ， 就是把有效值 101 往左挪两位就是 0001 0100 ，正数左边第一位补 0，负数补 1，等于乘于 2 的 n 次方，十进制位是 20

### 右移运算符 >>

**凡位运算符都是把值先转换成二进制再进行后续的处理**，5 的二进制位是 0000 0101，右移两位就是把 101 右移后为 0000 0001，正数左边第一位补 0，负数补 1，等于除于 2 的 n 次方，结果为 1

### 取反运算符 ~

取反就是将值转换为二进制位之后，1 取反为 0，0 取反为 1。5 的二进制位是 0000 0101，取反后为 1111 1010，值为 -6

### 无符号右移运算符

#### 正数无符号右移

无符号右移运算符和右移运算符的主要区别在于负数的计算，因为无符号右移是高位补0，移多少位补多少个0。

15 的二进制位是 0000 1111 ， 右移 2 位 0000 0011，结果为 3

#### 负数无符号右移

-6 的二进制是 6 的二进制取反再加 1，6 的二进制也就是 0000 0000 0000 0000 0000 0000 0000 0110，取反后加 1 为1111 1111 1111 1111 1111 1111 1111 1010，右移三位 0001 1111 1111 1111 1111 1111 1111 1111

# 算数操作符

* 加减乘除：整数除法的结果会**舍弃小数位**，而不是四舍五入

* `Random`：`Random.nexInt()` 方法的参数设置了所生成随机数的上限，其下限是 0

* 一元操作符：一元减号 会 反转数据的符号。一元加号和一元减号相对应，它唯一的作用就是将较小类型的操作数提升为 int 类型

* 自动递增：++a : 先运算，后返回结果； a++ : 先返回结果，再运算

* 关系运算符：等于 和 不等于 适用于所有的基本数据类型，但其他比较操作符不适用于 boolean 类型

* 操作符 == 和 != 比较的是**对象的引用**。

* **Integer**：对于值范围 在 -128~127 的 Integer 类型来说，它生成的对象是相同的：**出于效率问题，Integer 会通过享元模式来缓存范围在 -128~127 内的对象，因此多次调用 `Integer.valueOf(127)` 生成的其实是同一个对象**。另外，**通过 `new Integer()` 生成的对象都是新创建的，无论其值处于什么范围。**

  > Java 9 及更新版本中已经弃用 `new Integer()` ，因为它的效率远远低于 `Integer.valueOf()`

* 科学记数法：e 表示 “10 的幂次”。编译器一般会将指数作为 double 类型处理，所以如果没有尾部的 f，我们会收到一条出错提示，告诉我们必须将double 类型转换成 float 类型

# String 字符串

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

* 为什么 `System.out.println(引用)` 会自动调用` toString()` 方法？

  因为 `println()` 会调用 `String.valueOf()` 方法，而 `String.valueOf()` 会调用 `toString()` 方法

  ```java
  public void print(String s) {
      write(String.valueOf(s));
  }
  
  public static String valueOf(Object obj) {
      return (obj == null) ? "null" : obj.toString();
  }
  ```

  

* [String 全面讲解](https://blog.csdn.net/m0_58761900/article/details/125014074)

## String 源码分析

  ~~~java
  // 实现 Serializable（可序列化）, Comparable 接口
  public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
      /** 该值用于字符存储。final 修饰的 char[] ，说明字符串为不可变 */
      private final char value[];
  
      /** 缓存字符串的哈希代码 */
      private int hash; // 默认为 0
  
      /** 使用 JDK1.0.2 中的 serialVersionUID 实现互用性 */
      private static final long serialVersionUID = -6849794470754667710L;
      
      /**
       * 将此字符串与指定的对象进行比较。
       * 当且仅当参数不为 null 并且是表示与此对象相同的字符序列的 String 对象时，结果为 true
       *
       * @param  anObject 将此字符串 String 与之进行比较的对象
       *
       * @return 如果给定对象表示与该字符串等效的字符串，则为 true，否则为 false
       */
      public boolean equals(Object anObject) {
          if (this == anObject) {// 1. 先判断是否为同一个对象
              return true;
          }
          if (anObject instanceof String) {// 2. 判断是否是 String 类或子类
              String anotherString = (String)anObject;    //转型
              int n = value.length;
              if (n == anotherString.value.length) {  //3. 判断字符串长度，不相等肯定就是 false
                  char v1[] = value;
                  char v2[] = anotherString.value;
                  int i = 0;
                  while (n-- != 0) {  // 4. 循环，比较各字符是否相等
                      if (v1[i] != v2[i])
                          return false;
                      i++;
                  }
                  return true;
              }
          }
          return false;
      }
  
      /**
       * 返回此字符串的哈希代码。
       * String 对象的哈希代码计算为 s[0]*31^(n-1) + s[1]*31^(n-2）+ ... + s[n-1]
       * 使用 int 算术，其中 s[i] 是字符串的第 i 个字符，n 是字符串的长度，^ 表示取幂。（空字符串的哈希值为零）
       * 
       * @return  此对象的哈希代码值
       */
      public int hashCode() {
          int h = hash;
          if (h == 0 && value.length > 0) {   //判断 hash 值 是否已经计算过了，并判断 value 是否是空数组
              char val[] = value;
  
              for (int i = 0; i < value.length; i++) {    // 循环
                  /*
                  31 作为乘积因子的原因：
                      1. 31 是一个不大不小的质数，是作为 hashCode() 乘积因子的优选质数之一
                      2. 31 可以被 JVM 优化：31 * i = （i << 5）- i，因为移位运算比乘法运行更快更省性能
                  */
                  h = 31 * h + val[i];
              }
              hash = h;   // 将计算后的值赋给 hash，这样就只需要计算一次 hash 值了
          }
          return h;
      }
      
      // compareTo()：按字母顺序比较两个字符串，是基于字符串中每个字符的 Unicode 值。当两个字符串某个位置的字符不相同时，返回的是这一位置的字符 Unicode 值之差。当两个字符串都相同时，返回两个字符串长度之差
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
      // compareToIgnoreCase()：在 compareTo() 的基础上忽略大小写，大写字母比小写字母的 Unicode 值小 32，底层实现是先都转换成大写比较，然后都转换成小写比较
      
      // 当调用 intern 方法时，如果池中已经包含一个与该 String 确定的字符串相同 equals(Object) 的字符串，则返回该字符串。否则，将此 String 对象添加到池中，并返回此对象的引用。
  　　// 这句话什么意思呢？就是说调用一个 String 对象的 intern() 方法，如果常量池中有该对象了，直接返回该字符串的引用（存在堆中就返回堆中，存在池中就返回池中），如果没有，则将该对象添加到池中，并返回池中的引用。示例：
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
  
  // s3 的实现原理是，先创建一个 StringBuilder() 对象，并调用append()方法将数据拼接，最后调用toString() 方法返回拼接好的字符串
  ~~~

* 字符串字面量拼接编译期优化：

  ![image-20230301111627563](https://gitee.com/swlfox/picture-bed/raw/master/Img/image-20230301111627563.png)

## StringBuilder、StringBuffer 的区别

  1. StringBuilder、StringBuffer 都继承自 AbstractStringBuilder 类，AbstractStringBuilder 类使用 char 类型的数组保存字符串，有一个 count 属性用来保存当前保存的字符串个数，有一系列的字符串操作方法。

  2. StringBuilder、StringBuffer 的字符串操作方法都是直接调用的父类的方法，StringBuffer 的方法都被 synchronized 修饰，适用于多线程场景。

  3. StringBuffer 有一个 toStringCache 属性，被 transient 修饰。toStringCache 字段是为了作**缓存**的，缓存最后一次 toString 的内容，当被修改时这个 cache 清空，也就是说，如果没被修改，那么这个 toStringCache 就是上一次 toString 的结果：

     ~~~java
     /**
      * toString() 返回的最后一个值的缓存。
      * 在修改 StringBuffer 时清除。
      */
     private transient char[] toStringCache;
     
     // StringBuffer 的 toString()
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

# 集合

## Java 集合可分为 Collection 和 Map 两种体系

- Collection 接口：单列数据，定义了存取一组对象的方法的集合，用来存储一个一个的对象。

  - List：存储有序、可重复的数据。

    - ArrayList：作为 List 接口的主要实现类

      - 当随机访问 List 时（get 和 set 操作），效率高，但线程不安全；

      - 底层使用 Object[] elementData 存储，基于数组实现。ArrayList 支持默认大小构造，和空构造，当空构造的时候存放数据的 Object[] elementData 是一个空数组 []；

      - ArrayList 有 modCount 属性，用于快速失败 fail-fast 机制；

      - 当初始化的 list 是一个空 ArrayList 的时候，会直接扩容到 `DEFAULT_CAPACITY` 默认值 10。而当添加进 ArrayList 中的元素超过了数组能存放的最大值就会进行扩容，扩容代码：

        ```java
        // 右移一位，相当于扩容为原先的 1.5 倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        ```

      - 数组 copy：Java 无法自己分配空间，通过 `arraycopy()` 这个 **native 方法实现数组的复制**

        ```java
        /**
         * 从指定的源数组复制一个数组，从指定位置开始，到目标数组的指定位置。
         * @param src 源数组
         * @param srcPos 源数组中的起始位置
         * @param dest 目标数组
         * @param destPos 目标数据中的起始位置
         * @param length 要复制的数组元素的数量
         */
        public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
        ```

    - LinkedList：

      - 底层使用双线链表存储；

      - 对于频繁的插入、删除操作，使用此类效率比 ArrayList 高；
      - 查询方法：先判断 index 更靠近头部还是尾部，靠近哪段从哪段遍历获取值；

    - Vector：作为 List 接口的古老实现类。

      - 线程安全，但效率低；
      - 底层使用 Object[] elementData 存储。基于数组实现；

  - Set：存储无序的、不可重复的数据。

    - HashSet：作为 set 接口的主要实现类。线程不安全。可以存储 NULL 值。底层采用数组 + 链表的结构。
      - LinkedHashSet：作为 HashSet 的子类。遍历器内部数据时，可以按照添加的顺序遍历。
    - TreeSet：底层实际是 TreeMap，而 TreeMap 的底层是二叉树，放到 TreeSet 集合中的元素等同于放到了 TreeMap 集合 key 部分。无序不可重复，但可以按照添加对象的指定属性进行排序。

> 无序性：不等于随机性，存储的数据在底层数组中，并非按照数组索引的顺序添加，而是根据数据的哈希值决定的。<br>
> 不可重复性：保证添加的元素按照 equals() 判断时，不能返回 true。即相同的元素只能添加一个。

- Map 接口：双列数据，保存具有 `key - value 对` 的集合。
  - HashMap：作为 Map 的主要实现类。线程不安全，效率高。能存储 NULL 的 key 和 value。
    - LinkedHashMap：保证在遍历 Map 元素时可以按照添加的顺序实现遍历。<br>
      原因：在原有的 HashMap 底层结构基础上，添加了一对指针，指向前一个和后一个元素。<br>
      对于频繁的遍历操作，此类执行效率高于 HashMap。
  - TreeMap：保证按照添加的 `key - value` 键值对进行排序，实现排序遍历。此时考虑 key 的自然排序或者定制排序。底层使用红黑树。
  - Hashtable：作为古老的实现类，用 synchronized 修饰的，所以其操作是**线程安全**的，但是效率会受影响。HashTable 继承了 Dictionary 抽象类，Dictionary 类中有这样一行注释，当 key 为 null 时会抛出空指针 NullPointerException，不能存储 NULL 的 key 和 value。
    - Properties：常用于处理配置文件。key 和 value 都是 String 类型。

> HashMap 的底层：<br>
> JDK 7 以及之前版本：数组 + 链表<br>
> JDK 8：数组 + 链表 + 红黑树

> Map 结构的理解：<br>
> Map 中的 key：无序的、不可重复的。使用 Set 存储所有的 key --> key 所在的类要重写 `equals()` 和 `hashCode()`（以 HashMap 为例）<br>
> Map 中的 value：无序的、可重复的。使用 Collection 存储所有的 value --> value 所在的类要重写 `equals()`<br>
> 一个键值对：key - value 构成了一个 entry 对象。<br>
> Map 中的 entry：无序的、不可重复的。使用 Set 存储所有的 entry。<br>

## Collection

### 如何区分 List 中 remove(int index) 和 remove(Object obj)？

```java
@Test
public void testListRemove() {
	List list = new ArrayList();
	list.add(4);
	list.add(3);
	list.add(2);
	list.add(1;
	updateList1(list);
	System.out.println(list);
	updateList2(list);
	System.out.println(list);
}

private void updateList1(List list) {
	list.remove(3);	// 索引值 index = 3，输出结果为 [4, 3, 1]
}

private void updateList2(List list) {
	list.remove(new Integer(3));	//对象 value = 2，输出结果为 [4, 1]
}
```

`List.remove(Object obj)`：以对象方式删除指定元素

`List.remove(int index)`：删除指定索引位置的元素

> 一般来说，采用索引方式删除效率会更高；
> 对象方式删除需要遍历列表，索引删除只需要检查索引合法。

### ArrayList 和 LinkedList 的区别

- ArrayList、LinkedList 都是 List 接口实现而来的，存储数据的特点是相同的，都是存储有序的、可重复的数据。
- 一个是 Array（动态数组）的数据结构，相当于动态数组；另一个是Link（链表）的数据结构，底层使用双向链表结构，也可当作堆栈、队列、双端队列。
- ArrayList 作为 List 接口的主要实现类。当随机访问 List 时（get 和 set 操作），ArrayList 比 LinkedList 的效率更高，因为 LinkedList 是线性的数据存储方式，所以需要移动指针从前往后依次查找。但 ArrayList 线程不安全。
- LinkedList 对于频繁的插入、删除操作，使用效率比 ArrayList 高

### 为什么说 ArrayList 线程不安全？

首先看 ArrayList 源码

```java
public class ArrayList<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
		
    //默认的数组容量
    private static final int DEFAULT_CAPACITY = 10;

    ...
	
    //存放数据的底层数组，使用 transient 关键字不序列化某个变量
    transient Object[] elementData;

    //记录当前 ArrayList 的大小（它包含的元素数）。
    private int size;
  
  	...

	/**
	 * 将指定的元素追加到此列表的末尾。
	 * 添加一个元素时，做了如下两个操作：
	 * 1. 判断列表的 capacity 容量是否足够，是否需要扩容
	 * 2. 将元素放在列表的元素数组里面
	 */
   	public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // 这里会判断容量是否充足，不充足需要扩容
        elementData[size++] = e;
        return true;
    }
}
```

ArrayList 底层使用 `Object[] elementData 数组` 实现，用来保存所有的元素，以及一个 size 变量用来保存当前数组中已经添加了多少元素。

- 在多个线程进行 add 操作时可能会导致 elementData 数组越界

  ArrayList 默认数组大小为 10。假设现在已经添加进去 9 个元素了，size = 9。

  1.  线程 A 执行完 add 函数中的`ensureCapacityInternal(size + 1)`挂起了。
  2.  线程 B 开始执行，校验数组容量发现不需要扩容。于是把 "b" 放在了下标为 9 的位置，且 size 自增 1。此时 size = 10。
  3.  线程 A 接着执行，尝试把 "a" 放在下标为 10 的位置，因为 size = 10。但因为数组还没有扩容，最大的下标才为 9，所以会抛出数组越界异常 `ArrayIndexOutOfBoundsException`

  主要看这段代码：

  ```java
  elementData[size++] = e;
  ```

  不是一个原子操作，是分两步执行的，可拆分为：

  ```java
  elementData[size] = e;
  size++;
  ```

  情况一（size 不达标）：

  > 代码中有两个线程，假设为 t1 和 t2，有 ArrayList size=5（即其中有5个元素）。elementData.length=10<br>
  > t1 进入 add() 方法，这时获取到 size 值为 5，调用 ensureCapacityInternal() 方法判断容量是否需要扩容<br>
  > t2 也进入 add() 方法，这时获取到 size 值也为 5，也调用 ensureCapacityInternal() 方法判断容量是否需要扩容<br>
  > t1 发现自己的需求为 size+1=6，容量足够，无需扩容<br>
  > t1 发现自己的需求为也 size+1=6，容量足够，无需扩容<br>
  > t1 开始设置元素操作，`elementData[size] = e`，成功，<br>
  > t2 也开始设置元素操作，`elementData[size] = e`，成功，注意此时 t1 的 size+1 还没执行<br>
  > t1 size = size + 1 = 6，暂未写入主存<br>
  > t2 size = size + 1 此时因为 t1 操作完 size 还未写入主存，所以 size 依然为5，+1 后仍为 6<br>
  > t1 将 size=6 写入主存<br>
  > t2 将 size=6 写入主存<br>
  > 这样，size=6 比预期结果小了。

  情况二（size 达标）：

  > 代码中有两个线程，假设为 t1 和 t2，有 ArrayList size=5（即其中有 5 个元素）。elementData.length=10<br>
  > t1 进入 add()方法，这时获取到 size 值为 5，调用 ensureCapacityInternal() 方法判断容量是否需要扩容<br>
  > t2 也进入 add()方法，这时获取到 size 值也为 5，也调用ensureCapacityInternal()方法判断容量是否需要扩容<br>
  > t1 发现自己的需求为 size+1=6，容量足够，无需扩容<br>
  > t1 发现自己的需求为也 size+1=6，容量足够，无需扩容<br>
  > t1 开始设置元素操作，`elementData[size] = e`，成功，<br>
  > t2 也开始设置元素操作，`elementData[size] = e`，成功，注意此时 t1 的 size+1 还没执行<br>
  > t1 size = size + 1 = 6，并写入主存<br>
  > t2 size = size + 1 = 7<br>
  > 这样，size 符合预期，但是 t2 设置的值被覆盖，而且索引为 6 的位置将永远为 null，因为 size 已经为7，下次 add() 也会从 7 开始。除非手动 set 值。

- 当多线程环境下执行时，可能就会发生一个线程的值覆盖另一个线程添加的值

  依然看这段代码：

  ```java
  elementData[size++] = e;
  ```

    单线程执行这段代码完全没问题，可是到多线程环境下可能就有问题了，可能一个线程会覆盖另一个线程的值，逻辑如下：
        1. 列表为空 size = 0。
            2. 线程 A 执行完 `elementData[size] = e;`之后挂起。A 把 "a" 放在了下标为 0 的位置。此时 size = 0。
            3. 线程 B 执行 `elementData[size] = e;` 因为此时 size = 0，所以 B 把 "b" 放在了下标为 0 的位置，于是刚好把 A 的数据给覆盖掉了。
            4. 线程 B 将 size 的值增加为 1。
            5. 线程 A 将 size 的值增加为 2。
    这样子，当线程 A 和线程 B 都执行完之后理想情况下应该是 "a" 在下标为 0 的位置，"b" 在标为 1 的位置。而实际情况确是下标为 0 的位置为 "b"，下标为 1 的位置啥也没有。

### 数组 List 互相转换

* `List.toArray()`：集合转成 数组

* `Arrays.copyOf()`：若数组元素为对象，则数组里面数据是对象引用。通过 `Arrays.copyOf()` 方法产生的数组是一个**浅拷贝**

* `Arrays.asList()`：数组转成集合

  * `Arrays.asList()` 出现莫名其妙的问题，如下：

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

    注意这个参数：`T... a`，这个参数是一个**泛型的变长参数**，我们知道基本数据类型是不可能泛型化的，也是就说 8 个基本数据类型是不可作为泛型参数的，但是为什么编译器没有报错呢？这是因为在 java 中，**数组会当做一个对象来处理，它是可以泛型的**，所以我们的程序**是把一个 int 型的数组作为了 T 的类型，所以在转换之后 List 中就只会存在一个类型为 int 数组的元素了**。所以我们这样的程序 `System.out.println(datas.equals(list.get(0)));` 输出结果肯定是 true。当然如果将 int 改为 Integer，则长度就会变成 5 了。

    注意，**asList() 方法返回的 ArrayList 不是 java.util 中的 ArrayList，**而是 Arrays 工具类的内部类，该内部类没有 add() 方法，其父类的 add() 方法也没有具体实现。**asList() 返回的是一个长度不可变的列表。数组是多长，转换成的列表是多长，我们是无法通过 add()、remove() 来增加或者减少其长度的。**

### 如何解决 ArrayList 线程不安全

1. 将 ArrayList 替换成 Vector

   ```java
   Vector<Integer> arrayList = new Vector<>();
   ```

   Vector 是线程安全的，我们可以看下 Vector 底层的方法是同步的（Synchronized修饰），从而可以解决 ArrayList 线程不安全的问题

   ```java
   /**
    * Appends the specified element to the end of this Vector.
    * 将指定的元素追加到此 Vector 的末尾。
    *
    * @param e element to be appended to this Vector
    * @return {@code true} (as specified by {@link Collection##add})
    * @since 1.2
    */
   public synchronized boolean add(E e) {
       modCount++;
       ensureCapacityHelper(elementCount + 1);
       elementData[elementCount++] = e;
       return true;
   }
   ```

2. 使用 `Collections.synchronizedList()`

   ```java
   List<Integer> arrayList = Collections.synchronizedList(new ArrayList<>());
   ```

   `Collections.synchronizedList()` 方法中会根据传入的 List 是否实现 `RandomAccess` 这个接口来返回的 `SynchronizedRandomAccessList` 还是 `SynchronizedList`。

   ```java
   public static <T> List<T> synchronizedList(List<T> list) {
   return (list instanceof RandomAccess ?
           new SynchronizedRandomAccessList<>(list) :
           new SynchronizedList<>(list));
   }
   ```

3. 使用 `CopyOnWriteArrayList`

   ```java
   List<Integer> arrayList = new CopyOnWriteArrayList<>();
   ```

   CopyOnWriteArrayList 是一个线程安全的 ArrayList,其实现原理是读写分离，其对写操作使用ReentrantLock 来上锁，对读操作则不加锁；CopyOnWriteArrayList 在写操作的时候，会将 list 中的数组拷贝一份副本，然后对其副本进行操作（如果此时其他线程需要读的事，那么其他线程读取的是原先的没有修改的数组，如果其他写操作的线程要进行写操作，需要等待正在写的线程操作完成，释放 ReentrantLock 后，去获取锁才能进行写操作），写操作完成后，会将 list 中数组的地址引用指向修改后的新数组地址。

   如果使用场景的写操作十分频繁的话，建议还是不要实现 CopyOnWriteArrayList，因为其添加的时候会造成数组的不断扩容和复制，十分消耗性能，会消耗内存，如果原数组的数据比较多的情况下，可能会导致 Young GC 或者 Full GC；并且其不能使用在实时读的场景，在写操作过程中是要花费时间的，读取的时候可能还是旧数据；
   CopyOnWriteArrayList 合适读多写少的场景， 如果我们在使用的时候没法保证 CopyOnWriteArrayList 到底要放多少数据的话，我们还是要谨慎使用，如果数据稍微有点多，每次写操作都重新拷贝数组，其代价实在太高昂了。

> 一和二两种方法都是将所有的方法都加锁，那会导致效率低下，只能一个线程操作完，下一个线程获取到锁才能操作。
> CopyOnWriteArrayList 由于写时进行复制，内存里面同时存在两个对象占用内存，如果对象过大容易发送 YoungGc 和 FullGc，如果使用场景的写操作十分频繁的话，建议还是不要实现CopyOnWriteArrayList。

### HashMap的底层实现原理（烂大街面试题）（以JDK 7为例说明）

`HashMap map = new HahsMap();`<br>
在实例化以后，底层创建了长度是 16 的一维数组 `Entry[] table`。<br>
…<br>
可能已经执行过多次 `put()`<br>
…<br>
`map.put(key1, value1);`<br>
首先调用所在类的 `hashCode()` 计算 key1 哈希值，此哈希值经过某种算法计算以后，得到在 Entry 数组中的存放位置。<br>
如果此位置上的数据为空，此时的 key1-value1（实际上是 Entry[0]）添加成功。 --> 情况 1<br>
如果此位置上的数据不为空，（意味着此位置上存在一个或多个数据【以链表形式存在】），比较 key1 和已经存在的一个或多个数据的哈希值：<br>
&emsp;如果 key1 的哈希值与已经存在的数据的哈希值都不相同，此时 key1-value1 添加成功。 --> 情况 2<br>
&emsp;如果 key1 的哈希值与已经存在的某一个数据（key2-value2）的哈希值相同，继续比较：调用 key1 所在类的 equals(key2) 方法，比较：<br>
&emsp;&emsp;如果 equals() 返回 false：此时 key1-value1 添加成功。  --> 情况 3<br>
&emsp;&emsp;如果 equals() 返回 true：使用 value1 替换 value2<br>

> 补充：关于情况 2 和情况 3：此时 key1-value1 和原来的数据以链表的方式存储。

在不断的添加过程中，会涉及到扩容问题，当超出临界值（且要存放的位置非空）时，默认的扩容方式：扩容为原来的2倍，并将原有的数据复制过来。

#### JDK 8 相较于 JDK 7 在底层实现方面的不同：

1. `new HashMap()` 时底层没有创建一个长度为 16 的数组
2. JDK 8 底层的数组是：`Node[]`，而非 `Entry[]`
3. 首次调用 `put()` 方法时，底层创建长度为 16 的数组
4. 原来 JDK 7 底层结构只有：数组 + 链表。JDK 8 底层结构：数组 + 链表 + 红黑树。<br>
   当数组的某一个索引位置上的元素以链表形式存在的数据个数 > 8 且当前数组的长度 > 64 时，此时此索引位置上的所有数据改为用红黑树存储

> ![HashMap 源码中的重要常量](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E9%9D%A2%E8%AF%95%E9%A2%98/HashMap%20%E6%BA%90%E7%A0%81%E4%B8%AD%E7%9A%84%E9%87%8D%E8%A6%81%E5%B8%B8%E9%87%8F.png)

### iterator 迭代器接口

> 设计模式的一种（对象行为型模式），主要用于遍历 Collection 集合中的元素。<br>
> 提供一种方法顺序访问一个聚合对象中的各个对元素，而又不需要暴露该对象的内部表示。

iterator 迭代器接口源码如下：

```java
public interface Iterator<E> {  
    /**  
     * 如果迭代有更多元素，则返回true。（换句话说，如果next将返回元素而不是抛出异常，则返回true。）
     */
    boolean hasNext();  
  
    /**  
     * 返回迭代中的下一个元素。
     */
	E next();  

	/**  
     * 从基础集合中删除此迭代器返回的最后一个元素（可选操作）。每次调用 next 时只能调用此方法一次。如果在迭代过程中以调用此方法以外的任何方式修改了基础集合，则迭代器的行为未指定。
     * 抛出:
		UnsupportedOperationException：如果此迭代器不支持删除操作
		IllegalStateException：如果尚未调用下一个方法，或者在上一次调用下一方法之后已经调用了 remove方法
     * 实现：默认实现抛出 UnsupportedOperationException 实例
     * 要求:不执行其他操作。
     */
    default void remove() {  
        throw new UnsupportedOperationException("remove");  
    }  

	/**  
     * 对每个剩余元素执行给定的操作，直到所有元素都已处理完毕或该操作引发异常。如果指定了迭代顺序，则按迭代顺序执行操作。由操作引发的异常被转发给调用者。
     * 形参：action–每个元素要执行的操作
     * 抛出：
		NullPointerException：如果指定的操作为空
     * 实现 
     * 要求：默认实现行为 while (hasNext()) action.accept(next());`
     */
    default void forEachRemaining(Consumer<? super E> action) {  
        Objects.requireNonNull(action);  
        while (hasNext())  
            action.accept(next());  
    }  
}
```

在调用 it.next() 方法之前必须要调用 it.hasNext() 进行检测。若不调用，且 下一条记录无效，直接调用 it.next() 会抛出 NoSuchElementException 异常。

![迭代器执行原理](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E9%9D%A2%E8%AF%95%E9%A2%98/%E8%BF%AD%E4%BB%A3%E5%99%A8%E6%89%A7%E8%A1%8C%E5%8E%9F%E7%90%86.png)

## Map

`HashMap map = new HahsMap();`<br>
在实例化以后，底层创建了长度是 16 的一维数组 `Entry[] table`。<br>
…<br>
可能已经执行过多次 `put()`<br>
…<br>
`map.put(key1, value1);`<br>
首先调用所在类的 `hashCode()` 计算 key1 哈希值，此哈希值经过某种算法计算以后，得到在 Entry 数组中的存放位置。<br>
如果此位置上的数据为空，此时的 key1-value1（实际上是 Entry[0]）添加成功。 --> 情况 1<br>
如果此位置上的数据不为空，（意味着此位置上存在一个或多个数据【以链表形式存在】），比较 key1 和已经存在的一个或多个数据的哈希值：<br>
&emsp;如果 key1 的哈希值与已经存在的数据的哈希值都不相同，此时 key1-value1 添加成功。 --> 情况 2<br>
&emsp;如果 key1 的哈希值与已经存在的某一个数据（key2-value2）的哈希值相同，继续比较：调用 key1 所在类的 equals(key2) 方法，比较：<br>
&emsp;&emsp;如果 equals() 返回 false：此时 key1-value1 添加成功。  --> 情况 3<br>
&emsp;&emsp;如果 equals() 返回 true：使用 value1 替换 value2<br>

> 补充：关于情况 2 和情况 3：此时 key1-value1 和原来的数据以链表的方式存储。

在不断的添加过程中，会涉及到扩容问题，当超出临界值（且要存放的位置非空）时，默认的扩容方式：扩容为原来的2倍，并将原有的数据复制过来。

> ![HashMap 源码中的重要常量](https://gitee.com/DoubleZHEz/mine/raw/main/ImageRepository/Java%20%E6%8A%80%E6%9C%AF/%E9%9D%A2%E8%AF%95%E9%A2%98/HashMap%20%E6%BA%90%E7%A0%81%E4%B8%AD%E7%9A%84%E9%87%8D%E8%A6%81%E5%B8%B8%E9%87%8F.png)

### HashMap 底层

HashMap 采用 **链地址法** 解决 哈希冲突

#### HashMap 源码

~~~java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;	// 默认初始化容量：16
    static final int MAXIMUM_CAPACITY = 1 << 30;	// 最大容量：2^30
    static final float DEFAULT_LOAD_FACTOR = 0.75f;	// 默认负载因子：0.75f
    static final int TREEIFY_THRESHOLD = 8;	// 当链表长度为 8 时，链表转换成红黑树
    static final int UNTREEIFY_THRESHOLD = 6;	// 节点小于 6 时，红黑树转换回链表
    static final int MIN_TREEIFY_CAPACITY = 64;	// 最小树容量
    
    transient Node<K,V>[] table;	// HashMap 底层，table 数组
    transient int size;	// 大小，含有的 key-value 数量
    transient int modCount;	// 用于 fail-fast 策略，由于 HashMap 非线程安全，在对 HashMap 进行迭代时，如果期间其他线程的参与导致 HashMap 的结构发生变化了（比如 put，remove 等操作），需要抛出异常 ConcurrentModificationException 并发修改异常
    int threshold;	// 阈值，当 table == {} 时，该值为初始容量（初始容量默认为 16）；当 table 被填充了，也就是为 table 分配内存空间后，threshold 一般为 capacity * loadFactory （容量 * 负载因子）。HashMap 在进行扩容时需要参考 threshold
    final float loadFactor;	// 负载因子
    
    static class Node<K,V> implements Map.Entry<K,V>{	 // HashMap 的节点类，也就是 key-value 键值对
        final int hash;	// 只计算一次，避免重复计算
        final K key;
        V value;
        Node<K,V> next; // 指向下一个节点，单链表结构
        
        public final int hashCode() {	// hash 计算为 key 的 hash 异或 value 的 hash
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }
        
        public final boolean equals(Object o) {......}// equals() 方法是 key 相同并且 value 相同
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
        
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)	// 当 table 数组的大小 < 64（最小转换成树的容量）时，不转换成树，而是扩容
            resize();	//调整大小
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
            
            int mc = modCount;	// 获取 modCount
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e.key, e.value);
            }
            
            if (modCount != mc)	// 如果遍历过程中，modCount 发生了改变，意味着 HashMap 被修改了，抛出异常。修改 HashMap 的方法都会让 modCount 自增，如 put()、remove() 等
                throw new ConcurrentModificationException();
        }
    }
}    
~~~

#### JDK 8 相较于 JDK 7 在底层实现方面的不同：

1. `new HashMap()` 时底层没有创建一个长度为 16 的数组
2. JDK 8 底层的数组是：`Node[]`，而非 `Entry[]`
3. 首次调用 `put()` 方法时，底层创建长度为 16 的数组
4. 原来 JDK 7 底层结构只有：数组 + 链表。JDK 8 底层结构：数组 + 链表 + 红黑树。<br>
   当数组的某一个索引位置上的元素以链表形式存在的数据个数 > 8 且当前数组的长度 > 64 时，此时此索引位置上的所有数据改为用红黑树存储

#### HashMap 插入值

往数组中添加一个键值对，首先根据 key，通过 hash() 方法计算出 key 的 hash 值；假设计算出来的 key 的 hash 值为 hashValue，那么再通过 **index = (length - 1) & hashValue** 这个公式计算出 index（下标）的值（**该公式保证了获取的 index 一定在数组范围内**），这里的 length  是数组的长度，最后将键值对插入数组中下标为 index 的位置。

通过 index = (length  - 1) & hashValue 可知，**计算出来的 index 是强依赖于数组长度**，那么当**HashMap 扩容**之后，所有元素在数组中的下标需要**重新计算**

如果计算出来的 index 位置已经有元素了，这个时候不会再另外计算一次 index，而是用链表的形式插入到 index 的位置（**jdk1.7 采用头插法，jdk1.8 采用尾插法**，**使用尾插法的原因**：因为在多线程的的情况下可能会出现**环形链**的情况，因为 HashMap 在扩容之后是会重新计算每个元素的位置的，元素的位置改变说明各个元素之前的引用就会改变，所以在多线程下操作jdk1.7的 HashMap 的时候可能会出现死循环。而改用 jdk8 的尾插法，元素之前的引用顺序不会改变，也就不会引起死循环。）

#### HashMap 扩容

HashMap根据两个属性进行扩容：capacity（默认容量为16，1<<<4），loadFactor（负载因子，默认为0.75）

在默认情况下，16 * 0.75 = 12，当添加第 13 个元素时，就会进行扩容，**扩容之后的容量是原来容量的2倍**，并重新计算每个元素的 index

##### 2 倍扩容

* HashMap 的数组长度一定保持 2 的次幂的目的：

  1. 尽可能的减少元素位置的移动；
  2. 使元素均匀的散布 HashMap 中，减少 hash 碰撞。

  关键代码：`tab[(n - 1) & hash]`

#### 为什么要使用红黑树

红黑树：Red-Black Tree，简称R-B Tree，是一种**不严格的平衡二叉查找树**，红黑树中的节点，有颜色标记，并满足以下要求：

* 根节点是黑色的
* 每个叶子节点都是黑色的空节点，也就是说，叶子节点不存储数据
* 任何相邻的节点都不能同时为红色，红色节点是被黑色节点隔开的
* 每个节点，从该节点到其可达叶子节点的所有路径，都包含相同数目的黑色节点

红黑树的优势：

* 红黑树是 **“近似平衡”** 的
* 红黑树相比 AVL 树（自平衡二叉查找树，高度平衡树），红黑树不像 AVL 树 一样追求绝对的平衡，它允许局部很少的不完全平衡，这样对效率影响不大，但省去了很多没必要的调平衡操作，AVL 树调平衡有时代价很大
* 红黑树的高度只比 AVL 树的高度（logn）大一倍，但性能上去好很多
* AVL 树是高度平衡的二叉树，所以查找速度非常快，但是，为了维持高度平衡，在插入删除时，调整代价也很大。所以，对于有频繁插入删除操作的数据集合，使用 AVL 树代价有点高

#### hash() 方法为什么要用 h>>>16

h >>> 16 相当于将高区的 16 位移动到低区的 16 位，再与原 hashCode 做异或运算，可以**将高低位二进制特征混合起来**

由于计算 hash 值后，需要计算HashMap中数组的索引，计算公式：（length - 1）& hash，假设 length为默认值 16，计算时hash的高位的16位很有可能会被 length-1 屏蔽，**如果不做刚才的移位异或运算，那么在计算索引时将丢失高位特征**

失去高位特征意味着hash碰撞的可能性会增大

**使用异或运算的原因**：异或运算能更好的保留各部分的特征，如果采用 & 运算计算出来的值会向 0 靠拢，采用 | 运算计算出来的值会向 1 靠拢

2倍扩容的原因也是类似：假设数组大小是 17，则索引公式就是 （17 - 1）& hash，16 的二进制是1 0000，参与 & 计算后，hash 会被更多的 0 屏蔽，计算结果只剩下 0 和 16 两种情况，哈希碰撞急剧上升

综上，h>>>16 和 2倍扩容 的目的都是 为了让hash 后的结果更均匀的分布，减少哈希碰撞

#### HashMap 懒加载

HashMap 采用了 **懒加载**，在创建 HashMap 时并没有初始化 Node<K,V>[] table，在第一次 put 时才初始化这个 table

#### HashMap 为什么在链表长度大于 8 时转换成红黑树

官方做过很多测试，**发现链表长度在大于 8 以后再出现 hash 碰撞的可能性为 0.00000006（几乎为0）**，**也就是说哈希碰撞同一个位置 8 次的概率几乎为 0**，虽然转换为红黑树后，查询效率比链表高，但是转换红黑树的过程是耗时的，而且在扩容时还要对红黑树重新左旋或右旋保持平衡。**阈值设置为 8 就是为了尽量减少 HashMap 中出现红黑树**（HashMap 中链表才是常态） 

#### 数组 List 互相转换

* `List.toArray()`：集合转成 数组

* `Arrays.copyOf()`：若数组元素为对象，则数组里面数据是对象引用。通过Arrays.copyOf() 方法产生的数组是一个**浅拷贝**

* `Arrays.asList()`：数组转成 集合

  * `Arrays.asList()` 出现莫名其妙的问题，如下：

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

    注意这个参数:T… a，这个参数是一个**泛型的变长参数**，我们知道基本数据类型是不可能泛型化的，也是就说8个基本数据类型是不可作为泛型参数的，但是为什么编译器没有报错呢？这是因为在java中，**数组会当做一个对象来处理，它是可以泛型的**，所以我们的程序**是把一个int型的数组作为了T的类型，所以在转换之后List中就只会存在一个类型为int数组的元素了**。所以我们这样的程序System.out.println(datas.equals(list.get(0)));输出结果肯定是true。当然如果将int改为Integer，则长度就会变成5了。

    注意，**asList() 方法返回的 ArrayList 不是 java.util 中的 ArrayList，**而是Arrays 工具类的内部类，该内部类没有add() 方法，其父类的add() 方法也没有具体实现。**asList返回的是一个长度不可变的列表。数组是多长，转换成的列表是多长，我们是无法通过add、remove来增加或者减少其长度的。**

# Enum 枚举

## 枚举的定义和使用

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

## 枚举的底层原理

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

## 枚举类的常用方法

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

# 多线程

## 锁

### 乐观锁和悲观锁

| 术语   | 描述                                                         | 示例                                           |
| ------ | ------------------------------------------------------------ | ---------------------------------------------- |
| 乐观锁 | 每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据 | 版本号或时间戳控制，适用于多读少写的场景       |
| 悲观锁 | 每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁 | DB的行锁、表锁等，适用于数据一致性比较高的场景 |

#### 使用场景

##### 什么时候使用乐观锁？

资源提交冲突，其他使用方需要重新读取资源，会增加读的次数，但是可以面对高并发场景，前提是如果出现提交失败，用户是可以接受的。因此一般乐观锁只用在高并发、多读少写的场景。
 其中：GIT,SVN,CVS等代码版本控制管理器，就是一个乐观锁使用很好的场景，例如：A、B程序员，同时从SVN服务器上下载了code.html文件，当A完成提交后，此时B再提交，那么会报版本冲突，此时需要B进行版本处理合并后，再提交到服务器。这其实就是乐观锁的实现全过程。如果此时使用的是悲观锁，那么意味者所有程序员都必须一个一个等待操作提交完，才能访问文件，这是难以接受的。

##### 什么时候使用悲观锁？

一旦通过悲观锁锁定一个资源，那么其他需要操作该资源的使用方，只能等待直到锁被释放，好处在于可以减少并发，但是当并发量非常大的时候，由于锁消耗资源，并且可能锁定时间过长，容易导致系统性能下降，资源消耗严重。因此一般我们可以在并发量不是很大，并且出现并发情况导致的异常用户和系统都很难以接受的情况下，会选择悲观锁进行。

#### 示例

##### 乐观锁-版本号控制

```java
1, start transaction  
2, first_version = get_cur_version() // 获取当前数据版本
3, update_data(version+=1)           // 更新操作版本号+1
4, cur_version = get_cur_version()   // 提交更新时，获取版本号
5, if first_version == cur_version   // 比较提交时的版本号与第一次获取的版本号，如果一致，那么认为资源是最新的，可以更新
     then commit 
  else rollback or raise exception  // 否则回滚或者抛出异常
```

##### 乐观锁-时间戳控制

```java
1, start transaction
2, first_timestamp = get_cur_timestamp()
3, update_data(timestamp=get_sys_cur_timestamp)
4, if first_timestamp = get_cur_timestamp()
      then commit 
   else rollback or raise Exception
```

##### 悲观锁

悲观锁的实现一般都是通过锁机制来实现的，锁可以简单理解为资源的访问的入口。如果要对一个具有锁属性的资源执行访问时，在更新操作时，需要持锁权才能进行操作，但是往往这种操作可以保证数据的一致性和完整性。例如数据库表的行锁。

# 反射

