---
title: Java
date: 2023-07-19 23:48:07
mdate: 2023-07-19 23:52:09
---
# Java 概述

Java 是一门面向对象的编程语言，不仅吸收了 C++ 语言的各种优点，还摒弃了 C++ 里难以理解的多继承、指针等概念，因此 Java 语言具有功能强大和简单易用两个特征。Java 语言作为静态面向对象编程语言的优秀代表，极好地实现了面向对象理论，允许程序员以优雅的思维方式进行复杂的编程。

## Java 语言特点

![Java 语言特点](../../../ImageRepository/%E8%AE%A1%E7%AE%97%E6%9C%BA%E4%B8%93%E4%B8%9A/Java%20%E6%8A%80%E6%9C%AF/%E9%9D%A2%E8%AF%95%E9%A2%98/Java%20SE/Java%20%E8%AF%AD%E8%A8%80%E7%89%B9%E7%82%B9.png)

- 面向对象（封装，继承，多态）；
- 平台无关性，平台无关性的具体表现在于，Java 是“一次编写，到处运行（Write Once，Run any Where）”的语言，因此采用 Java 语言编写的程序具有很好的可移植性，而保证这一点的正是 Java 的虚拟机机制。在引入虚拟机之后，Java 语言在不同的平台上运行不需要重新编译。
- 支持多线程。C++ 语言没有内置的多线程机制，因此必须调用操作系统的多线程功能来进行多线程程序设计，而 Java 语言却提供了多线程支持；
- 编译与解释并存；

## JVM、JRE、JDK 之间的关系

   Java 虚拟机 JVM 保证 Java 一次编写到处运行，可移植性好；

   JRE 是运行环境，不能创建新程序。JVM 包含于其中；

   JDK 功能最齐全，包括编译器和各种工具，写代码就需要这个

   ![JVM、JRE、JDK 之间的关系](../../../ImageRepository/%E8%AE%A1%E7%AE%97%E6%9C%BA%E4%B8%93%E4%B8%9A/Java%20%E6%8A%80%E6%9C%AF/%E9%9D%A2%E8%AF%95%E9%A2%98/Java%20SE/JVM%E3%80%81JRE%E3%80%81JDK%20%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB.png)

## public、protected、default、private 的区别

   - default 是默认的，什么都不屑，在同一个包内是可见的，不使用任何修饰符。*可以理解为`包级别的 private`，即只有同一个包中的类能访问*

   - public 能用来修饰类，在一个 Java 源文件中智能有一个类被声明为 public，而且一旦有一个类为 public，那么这个 Java 源文件的文件名就必须要和这个被 public 所修饰的类的类名相同，否则编译不能通过。
     public 用来修饰类中成员（变量和方法），被 public 所修饰的成员可以在任何类中都能被访问到。

   - protected 是受保护的，收到该类所在的包所保护，被 protected 所修饰的成员会被位于同一 package 中的所有类访问到。并且，被 protected 所修饰的成员也能被该类的所有子类所继承。

     > - 基类的 protected 成员是包内可见的，并且对子类可见；
     > - 若子类与基类不在同一包中，那么在子类中，子类实例可以访问其从基类继承而来的 protected 方法，而不能访问基类实例的 protected 方法。

   - private 私有的，即只能在当前类中被访问到，它的作用域最小

| 修饰符    | 当前类 | 同一包内 | 子孙类（同一包） | 子孙类（不同包） | 其他包 |
| --------- | ------ | -------- | ---------------- | ---------------- | ------ |
| public    | Y      | Y        | Y                | Y                | Y      |
| protected | Y      | Y        | Y                | Y / N            | N      |
| default   | Y      | Y        | Y                | N                | N      |
| private   | Y      | N        | N                | N                | N      |

## final、finally、finalize 的区别

1. final：不可变，可以修饰变量、方法和类。修饰变量时，这个变量必须初始化，所以也称为常量。

2. finally：异常处理的一部分，只能用在 `try ... catch` 中，并且附带一个语句块。表示无论是否抛出异常这个 finally 中的语句一定会被执行。

3. finalize：`java.lang.Object` 中的方法，也就是每一个对象都有这个方法，一个对象的 finalize 方法指挥调用一次，调用了不一定会被回收，因为只有对象被回收的时候才会被回收，就会导致前面调用，后面回收的时候出现问题，不推荐使用。

   > Object finalize() 方法用于实例被垃圾回收器回收的时触发的操作。
   >
   > 当 GC (垃圾回收器) 确定不存在对该对象的有更多引用时，对象的垃圾回收器就会调用这个方法。

## static 关键字的作用

首先只有 new 对象之后，数据存储空间才会被分配，方法才能供外界调用。但是当没有创建对象的时候也想要调用方法或者就是想为特定分配存储空间的时候，就需要用 static。
有了 static，成员变量或者方法就可以在没有所属类的时候被访问了。

## 面向对象和面向过程

面向过程的性能比较高，因为没有实例化等操作，开销比较小。

面向对象因为有了封装继承多态的过程，可以设计出低耦合的系统，使得系统更灵活，更容易维护。

> 封装：指封装成抽象的类，并且对于可信的类或者对象，是可以操作的，对于不可信的进行隐藏。
>
> 继承：指可以使用现有类的所有功能，而且还可以在现有功能的基础上做拓展。
>
> 多态：基于继承，指父类中定义的属性和方法被子类继承之后，可以具有不同的数据类型或者表现出不同的行为，使得同一个属性在父类及其子类中具有不同的含义。
>
> &emsp;&emsp;&ensp;重载：多态的一个例子，是编译时的多态。多态是运行时多态，也就是说编译时不确定调用哪个具体方法，一直延迟到运行时才可以确定，所以多态又叫做延迟方法。

## 多态

Java 实现多态有 3 个必要条件：继承、重写和线上转型。
只有满足这 3 个条件，开发人员才能够在同一个继承结构中使用统一的逻辑实现代码处理不同的对象，从而执行不同的行为。

继承：在多态中必须存在有继承关系的子类和父类。

重写：子类对父类中某些方法进行重新定义，在调用这些方法是就会调用子类的方法。

向上转型：在多态中需要将子类的引用赋给父类对象，只有这样该引用才既能可以调用父类的方法，又能调用子类的方法。

## 重载和重写区别

重载和重写都是实现多态的方式。

区别在于**重载是编译时多态，重写是运行时多态**。

重载（overloading）是在一个类里面，方法名字相同，但参数不同。返回类型可以相同也可以不同。每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。最常见的地方就是构造器的重载。

重写发生在子类与父类之间，重写方法返回值和形参都不能改变，与方法返回值和访问修饰符无关，即重载的方法不能根据返回类型进行区分。**外壳不变，核心重写！**

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

      - 数组 copy: Java 无法自己分配空间，通过 `arraycopy()` 这个 **native 方法实现数组的复制**

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
> Map 中的 entry：无序的、不可重复的。使用 Set 存储所有的 entry。

## Collection

### List

#### ArrayList

##### ArrayList 底层就是一个 `Object[]` 数组

ArrayList 底层数组默认初始化容量为 10
<font size=6px>①</font> jdk 1.8 中 ArrayList 底层先创建一个长度为 0 的数组
<font size="6px">②</font> 当第一次添加元素（调用 `add()` 方法）时，会初始化为一个长度为 10 的数组

**当 ArrayList 中的容量使用完之后，则需要对容量进行扩容：**

- ArrayList 容量使用完后，会”自动“创建容量更大的数组，并将原数组中所有元素拷贝过去，这会导致效率降低
- 优化：可以使用构造方法 `ArrayList(int capacity)` 或 `ensureCapacity(int capacity)` 提供一个初始化容量，避免刚开始就一直扩容，造成效率较低

##### ArratList 构造方法

- `ArrayList()`：创建一个初始化容量为 10 的空列表
- `ArrayList(int initialCapacity)`：创建一个指定初始化容量为 initialCapacity 的空列表
- `ArrayList(Collection<? extends E> c)`：创建一个包含指定集合中所有元素的列表

##### ArrayList 特点

优点：

- 向 ArrayList 末尾添加元素 `add()` 时，效率较高
- 查询效率高

缺点：

- 扩容会造成效率较低（可以通过指定初始化容量在一定程度上对其进行改善）
- 数组无法存储大数据量（因为很难找到一块很大的连续的内存空间）
- 向 ArrayList 中间添加元素 `add(int index)`，需要添加元素，效率较低（如果增/删操作较多时，可考虑改用链表）

##### ArrayList 如何线程安全

调用 Collections 工具类中的 `static List synchronizedList(List list)` 方法

#### LinkedList

##### LinkedList 特点

数据结构：LinkedList 底层是一个双向链表

优点：增/删效率高

缺点：查询效率较低

LinkedList 也有下标，但是内存不一定是连续的（类似 C++ 重载 [] 符号，将循位置访问模拟为循序访问）

LinkedList 可以调用 `get(int index)` 方法，返回链表中第 index 个元素。**每次查找都要从头结点开始遍历**

##### LinkedList 部分源码解读

###### add() 方法

```java
public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, java.io.Serializable {
    /**
     * LinkedList 底层是一个双向链表
     */
	private static class Node<E> {	//LinkedList 的节点静态内部类 Node
        E item;
        Node<E> next;	//后继
        Node<E> prev;	//前驱

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    
    /**
     * 链表长度
     */
    transient int size = 0;

    /**
     * 指向链表第一个节点
     */
    transient Node<E> first;

    /**
     * 指向链表最后一个节点
     */
    transient Node<E> last;
    
    /**
     * 链表末尾添加元素 e
     */
    void linkLast(E e) {
        final Node<E> l = last;	//暂时保存最后一个元素的指针
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;	//newNode 作为最后一个节点
        if (l == null)	//当前链表为空
            first = newNode;	//第一个添加的节点，即为 first
        else	//链表不为空
            l.next = newNode;	//当前链表的最后一个节点 next 指向 newNode
        size++;
        modCount++;
    }
    
    /**
     * 将指定的元素追加到此列表的末尾
     */
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
}
```

#### Vector

Vector 底层是数组

初始化容量为 10

Vector 是线程安全的（里面方法都带有 synchronized 关键字），效率较低，现在使用较少

> synchronized 关键字声明的方法同一时间只能被一个线程访问。synchronized 修饰符可以应用于四个访问修饰符。

##### 扩容

原容量使用完后，会进行扩容。新容量扩大为原始容量的 2 倍

#### 如何区分 List 中 remove(int index) 和 remove(Object obj)？

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

#### ArrayList 和 LinkedList 的区别

- ArrayList、LinkedList 都是 List 接口实现而来的，存储数据的特点是相同的，都是存储有序的、可重复的数据。
- 一个是 Array（动态数组）的数据结构，相当于动态数组；另一个是Link（链表）的数据结构，底层使用双向链表结构，也可当作堆栈、队列、双端队列。
- ArrayList 作为 List 接口的主要实现类。当随机访问 List 时（get 和 set 操作），ArrayList 比 LinkedList 的效率更高，因为 LinkedList 是线性的数据存储方式，所以需要移动指针从前往后依次查找。但 ArrayList 线程不安全。
- LinkedList 对于频繁的插入、删除操作，使用效率比 ArrayList 高

#### 为什么说 ArrayList 线程不安全？

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

#### 数组 List 互相转换

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

#### 如何解决 ArrayList 线程不安全

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

   CopyOnWriteArrayList 是一个线程安全的 ArrayList,其实现原理是读写分离，其对写操作使用 ReentrantLock 来上锁，对读操作则不加锁；CopyOnWriteArrayList 在写操作的时候，会将 list 中的数组拷贝一份副本，然后对其副本进行操作（如果此时其他线程需要读的事，那么其他线程读取的是原先的没有修改的数组，如果其他写操作的线程要进行写操作，需要等待正在写的线程操作完成，释放 ReentrantLock 后，去获取锁才能进行写操作），写操作完成后，会将 list 中数组的地址引用指向修改后的新数组地址。

   如果使用场景的写操作十分频繁的话，建议还是不要实现 CopyOnWriteArrayList，因为其添加的时候会造成数组的不断扩容和复制，十分消耗性能，会消耗内存，如果原数组的数据比较多的情况下，可能会导致 Young GC 或者 Full GC；并且其不能使用在实时读的场景，在写操作过程中是要花费时间的，读取的时候可能还是旧数据；
   CopyOnWriteArrayList 合适读多写少的场景， 如果我们在使用的时候没法保证 CopyOnWriteArrayList 到底要放多少数据的话，我们还是要谨慎使用，如果数据稍微有点多，每次写操作都重新拷贝数组，其代价实在太高昂了。

> 一和二两种方法都是将所有的方法都加锁，那会导致效率低下，只能一个线程操作完，下一个线程获取到锁才能操作。
> CopyOnWriteArrayList 由于写时进行复制，内存里面同时存在两个对象占用内存，如果对象过大容易发送 YoungGC 和 Full GC，如果使用场景的写操作十分频繁的话，建议还是不要实现 CopyOnWriteArrayList。

### Set

#### HashSet

特点：HashSet 无序（没有下标），不可重复

HashSet 为 HashMap 的 key 部分

#### TreeSet

TreeSet 无序（没有下表），不可重复，但是可以排序

TreeSet 为 TreeMap 的 key 部分

### iterator 迭代器

Java 迭代器（Iterator）是 Java 集合框架中的一种机制，是一种用于遍历集合（如列表、集合和映射等）的接口。

它提供了一种统一的方式来访问集合中的元素，而不需要了解底层集合的具体实现细节。

Java Iterator（迭代器）不是一个集合，它是一种用于访问集合的方法，可用于迭代 [ArrayList](#ArrayList) 和 [HashSet](#HashSet) 等集合。

Iterator 是 Java 迭代器最简单的实现，ListIterator 是 Collection API 中的接口， 它扩展了 Iterator 接口。

![img](https://www.runoob.com/wp-content/uploads/2020/07/ListIterator-Class-Diagram.jpg)

迭代器接口定义了几个方法，最常用的是以下三个：

- **next()** - 返回迭代器的下一个元素，并将迭代器的指针移到下一个位置。
- **hasNext()** - 用于判断集合中是否还有下一个元素可以访问。
- **remove()** - 从集合中删除迭代器最后访问的元素（可选操作）。

Iterator 类位于 java.util 包中，使用前需要引入它，语法格式如下：

```
import java.util.Iterator; // 引入 Iterator 类
```

通过使用迭代器，我们可以逐个访问集合中的元素，而不需要使用传统的 for 循环或索引。这种方式更加简洁和灵活，并且适用于各种类型的集合。

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

![迭代器执行原理](../../../ImageRepository/%E8%AE%A1%E7%AE%97%E6%9C%BA%E4%B8%93%E4%B8%9A/Java%20%E6%8A%80%E6%9C%AF/%E9%9D%A2%E8%AF%95%E9%A2%98/Java%20SE/%E8%BF%AD%E4%BB%A3%E5%99%A8%E6%89%A7%E8%A1%8C%E5%8E%9F%E7%90%86.png)

#### Listiterator 接口

`LinkedList.add()` 只能将数据添加到链表的末尾

##### 将对象添加到链表的中间位置

如果要将对象添加到链表的中间位置，则需要使用 ListIterator 接口的 add() 方法

ListIterator 是 Iterator 的一个子接口

| 返回类型 | 方法名和参数    | 描述                                                         |
| -------- | --------------- | ------------------------------------------------------------ |
| void     | add(E e)        | 将指定的元素插入列表（可选操作）                             |
| boolean  | hasNext()       | 正向遍历列表，如果返回 true，说明列表不为空                  |
| boolean  | hasPrevious()   | 反向遍历列表，如果返回 true，说明列表不为空                  |
| E        | next()          | 返回列表中的下一个元素并向前移动光标位置                     |
| int      | nextIndex()     | 返回元素的索引，该索引将由对 next() 的后续调用返回           |
| E        | previous()      | 返回列表的上一个元素并向后移动光标位置                       |
| int      | previousIndex() | 返回元素的索引，该索引将由对 previous() 的后续调用返回       |
| void     | remove()        | 从列表中删除 next() 或 previous() 返回的最后一个元素（可选操作） |
| void     | set(E e)        | 将 next() 或 previous() 返回的最后一个元素替换为指定的元素（可选操作） |

##### remove() 方法

- 调用 next() 方法之后，remove() 删除的是迭代器左侧的元素（类似键盘的 backspace）
- 调用 previous() 方法之后，remove() 删除的是迭代器右侧的元素

##### add() 方法

- 调用 next() 方法之后，add() 在迭代器左侧添加一个元素
- 调用 previous() 方法之后，add() 在迭代器右侧添加元素

## Map

- Map 和 Collection 没有继承关系
- Map 以 `(key, value)` 的形式存储数据：键值对
  key 和 value 存储的都是对象的内存地址（引用）

### Map 接口常用方法

| 返回类型             | 方法名和参数                             | 描述                                                         |
| -------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| Set<K>               | keySet()                                 | 返回 map 中关于 key 的一个 Set 视图                          |
| Collection<V>        | values()                                 | 返回 map 中关于 value 的一个 Collection 视图                 |
| Set<Map.Entry<K, V>> | entrySet()                               | 返回此 map 中对应 Map.Entry<K, V> 的 Set 视图                |
| V                    | put(K key, V value)                      | 将指定的值与此 Map 中的指定键相关联                          |
| V                    | get(Oject key)                           | 返回指定键 key 映射到的值 value，如果此 Map 不包含键的映射，则返回 null |
| default V            | getOrDefault(Object key, V defaultValue) | 返回指定键 key 映射到的值 value，如果此 Map 不包含该键的映射 value 值，则返回 defaultValue |
| V                    | remove(Object key)                       | 如果 key 存在，从此 Map 中删除键 key 的映射                  |
| void                 | clear()                                  | 从此映射中删除所有 <K, V> 键值对                             |
| boolean              | isEmpty()                                | 如果此 Map 不包含 <K, V> 键值对，则返回 true                 |
| boolean              | containsKey(Object key)                  | 如果此 Map 包含指定键 key 的映射，则返回 true                |
| boolean              | containsValue(Object value)              | 如果此 Map 将一个或多个键 key 映射到指定值 value，则返回 true |

> <font color="red">注意</font>
>
> `Map.Entry<K, V>` 是 Map 的一个接口
> 接口中的内部接口默认是 `public static`

### Map 的遍历方式

#### 第一类

调用 map.keySet() 获取  的 keySet，然后取出 key 对应的 value

特点：效率相对较低。（因为还要根据 key 从哈希表中查找对应的 value）

1. 通过 foreach 遍历 map.keySet()，取出对应的 value

   ```java
   public static void printMap(Map<Integer, String> map) {
   	if (map.size() == 0) {
           System.out.println("the map is empty ...");
           return;
       }
       System.out.println("the elements in the map are like below ...");
       for (Integer key : map.keySet()) {
           System.out.println(key + " : " + map.getOrDefault(key, ""));
       }
   }
   ```

2. 通过迭代器迭代 map.keySet()，来取出对应的 value

   ```java
   public static void printMap(Map<Integer, String> map) {
   	Set<Integer> keySet = map.keySet();
   	Iterator<Integer> it = keySet.iterator();
   	while (it.hasNext()) {
   		Integer cntKey = it.next();
   		System.out.println(cntKey + " : " + map.getOrDefault(cntKey, ""));
   	}
   }
   ```

#### 第二类

调用 map.entrySet() 方法，获取 entrySet，然后直接从 entrySet 中获取 key 和 value

特点：

- 效率较高（直接从 node 中获取 key，value）
- 适用于大数据量 map 遍历

3. 调用 map.entrySet()，然后使用 foreach 遍历 entrySet

   ```java
   public static void printMap(Map<Integer, String> map) {
           Set<Map.Entry<Integer, String>> entry = map.entrySet();
           for (Map.Entry<Integer, String> it : entry)
                   System.out.println(it.getKey() + " : " + it.getValue());
   }
   ```

4. 调用 map.entrySet()，然后使用迭代器遍历 entrySet

   ```java
   public static void printMap(Map<Integer, String> map) {
       Set<Map.Entry<Integer, String>> entry = map.entrySet();
       Iterator<Map.Entry<Integer, String>> it = entrySet.iterator();
       while (it.hasNext()) {
           Map.Entry<Integer, String> node = it.next();
           System.out.pringln(node.getKey() + " : " + node.getValue());
       }
   }
   ```

### HashMap

#### 概述

1. HashMap 底层是一个数组

2. 数组中每个元素是一个单向链表（采用拉链法解决哈希冲突）
   单链表的节点每个节点都是 `Node<K, V>` 类型

3. 同一个单链表中所有 Node 的 hash 值不一定一样，但是它们对应的数组下标一定一样
   数组下标利用哈希函数/哈希算法根据 hash 值计算得到的

4. HashMap 是数组和单链表的结合体

   > - 数组查询效率高，但是增删元素效率较低
   > - 单链表在随机增删元素方面效率较高，但是查询效率较低
   >
   > HashMap 将二者结合起来，充分它们各自的优点

5. HashMap 特点

   无序、不可重复

   无序：因为不一定挂在那个单链表上了
   不可重复：通过重写 equals()

#### HashMap 底层

HashMap 采用**链地址法**解决哈希冲突

> ![HashMap 源码中的重要常量](../../../ImageRepository/%E8%AE%A1%E7%AE%97%E6%9C%BA%E4%B8%93%E4%B8%9A/Java%20%E6%8A%80%E6%9C%AF/%E9%9D%A2%E8%AF%95%E9%A2%98/Java%20SE/HashMap%20%E6%BA%90%E7%A0%81%E4%B8%AD%E7%9A%84%E9%87%8D%E8%A6%81%E5%B8%B8%E9%87%8F.png)
>
> - HashMap 默认初始化容量：16
>
>   必须是 2 的次幂，jdk 官方推荐的
>   这是因为达到散列均匀，为了提高 HashMap 集合的存取效率所必须的
>
> - HashMap 默认加载因子：0.75
>
>   数组容量达到 $ \frac{3}{4} $ 时，开始扩容
>
> - jdk 8 之后，对 HashMap 底层数据结构（单链表）进行了改进
>
>   如果单链表元素超过 8 个，则将单链表转变为红黑树
>   如果红黑树节点数量小于 6 时，会将红黑树重新变为单链表
>
> 这种方式是为了提高检索效率，二叉树的检索会再次缩小扫描范围，提高效率

##### HashMap 源码

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

##### HashMap的底层实现原理（烂大街面试题）（以 JDK 7为例说明）

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

###### JDK 8 相较于 JDK 7 在底层实现方面的不同

1. `new HashMap()` 时底层没有创建一个长度为 16 的数组
2. JDK 8 底层的数组是：`Node[]`，而非 `Entry[]`
3. 首次调用 `put()` 方法时，底层创建长度为 16 的数组
4. 原来 JDK 7 底层结构只有：数组 + 链表。JDK 8 底层结构：数组 + 链表 + 红黑树。<br>
   当数组的某一个索引位置上的元素以链表形式存在的数据个数 > 8 且当前数组的长度 > 64 时，此时此索引位置上的所有数据改为用红黑树存储

##### put() 方法原理

1. 先将 key，value 封装到 Node 对象中
2. 底层会调用 key 的 hashCode() 方法得出 hash 值
3. 通过哈希函数/哈希算法将 hash 值转换为数组的下标
   1. 如果下标位置上没有任何元素，就把 Node 添加到这个位置上
   2. 如果下标位置上有单链表，此时会将当前 Node 中的 key 与链表上每一个节点中的 key 进行 equals() 比较
      1. 如果所有的 equals() 方法返回都是 false，那么这个新节点 Node 将被添加到链表的末尾
      2. 如果其中有一个 equals() 返回了 true，那么链表中对应的这个节点上的 value 将会被新节点 Node 的 value 覆盖（保证了不可重复）

> <font color="green">tip</font>
>
> - HashMap 中允许 key 和 value 为 null，但是只能有一个（不可重复）
> - HashTable 中 key 和 value 都不允许为 null

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

##### HashMap 插入值

往数组中添加一个键值对，首先根据 key，通过 hash() 方法计算出 key 的 hash 值；假设计算出来的 key 的 hash 值为 hashValue，那么再通过 **index = (length - 1) & hashValue** 这个公式计算出 index（下标）的值（**该公式保证了获取的 index 一定在数组范围内**），这里的 length  是数组的长度，最后将键值对插入数组中下标为 index 的位置。

通过 index = (length  - 1) & hashValue 可知，**计算出来的 index 是强依赖于数组长度**，那么当**HashMap 扩容**之后，所有元素在数组中的下标需要**重新计算**

如果计算出来的 index 位置已经有元素了，这个时候不会再另外计算一次 index，而是用链表的形式插入到 index 的位置（**jdk1.7 采用头插法，jdk1.8 采用尾插法**，**使用尾插法的原因**：因为在多线程的的情况下可能会出现**环形链**的情况，因为 HashMap 在扩容之后是会重新计算每个元素的位置的，元素的位置改变说明各个元素之前的引用就会改变，所以在多线程下操作jdk1.7的 HashMap 的时候可能会出现死循环。而改用 jdk8 的尾插法，元素之前的引用顺序不会改变，也就不会引起死循环。）

##### HashMap 扩容

HashMap根据两个属性进行扩容：capacity（默认容量为16，1<<<4），loadFactor（负载因子，默认为0.75）

在默认情况下，16 * 0.75 = 12，当添加第 13 个元素时，就会进行扩容，**扩容之后的容量是原来容量的2倍**，并重新计算每个元素的 index

###### 2 倍扩容

* HashMap 的数组长度一定保持 2 的次幂的目的：

  1. 尽可能的减少元素位置的移动；
  2. 使元素均匀的散布 HashMap 中，减少 hash 碰撞。

  关键代码：`tab[(n - 1) & hash]`

##### 为什么要使用红黑树

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

##### hash() 方法为什么要用 h>>>16

h >>> 16 相当于将高区的 16 位移动到低区的 16 位，再与原 hashCode 做异或运算，可以**将高低位二进制特征混合起来**

由于计算 hash 值后，需要计算HashMap中数组的索引，计算公式：（length - 1）& hash，假设 length为默认值 16，计算时hash的高位的16位很有可能会被 length-1 屏蔽，**如果不做刚才的移位异或运算，那么在计算索引时将丢失高位特征**

失去高位特征意味着hash碰撞的可能性会增大

**使用异或运算的原因**：异或运算能更好的保留各部分的特征，如果采用 & 运算计算出来的值会向 0 靠拢，采用 | 运算计算出来的值会向 1 靠拢

2倍扩容的原因也是类似：假设数组大小是 17，则索引公式就是 （17 - 1）& hash，16 的二进制是1 0000，参与 & 计算后，hash 会被更多的 0 屏蔽，计算结果只剩下 0 和 16 两种情况，哈希碰撞急剧上升

综上，h>>>16 和 2倍扩容 的目的都是 为了让hash 后的结果更均匀的分布，减少哈希碰撞

##### HashMap 懒加载

HashMap 采用了 **懒加载**，在创建 HashMap 时并没有初始化 Node<K,V>[] table，在第一次 put 时才初始化这个 table

##### HashMap 为什么在链表长度大于 8 时转换成红黑树

官方做过很多测试，**发现链表长度在大于 8 以后再出现 hash 碰撞的可能性为 0.00000006（几乎为0）**，**也就是说哈希碰撞同一个位置 8 次的概率几乎为 0**，虽然转换为红黑树后，查询效率比链表高，但是转换红黑树的过程是耗时的，而且在扩容时还要对红黑树重新左旋或右旋保持平衡。**阈值设置为 8 就是为了尽量减少 HashMap 中出现红黑树**（HashMap 中链表才是常态） 
