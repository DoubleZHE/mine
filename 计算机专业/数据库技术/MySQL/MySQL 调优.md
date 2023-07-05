## 索引

![索引图](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E7%B4%A2%E5%BC%95%E5%9B%BE.png)

什么是索引？

在关系数据库中，索引是一种数据结构。它将数据提前按照一定的规则进行排序和组织，能够帮助快速定位到数据记录的数据，加快数据库表中数据的查找和访问速度。

**能实现快速定位数据的一种存储结构，其设计思想是以空间换时间。**

> 像书籍的目录、文件夹、标签、房号 ... 都可以帮助快速定位，都可以视为索引。

#### 索引的种类

在 MySQL 中索引是在存储引擎层实现的，而不是在服务器层实现的，所以不同存储引擎具有不同的索引类型和实现。
常见的索引分类如下：

- 按数据结构分类：**B++ Tree 索引**、Hash 索引、Full-text 索引
- 按物理存储分类：聚集索引、非聚集索引
- 按字段特性分类：主键索引（PRIMARY KEY）、唯一索引（UNIQUE）、普通索引（INDEX）、全文索引（FULLTEXT）
- 按字段个数分类：单列索引、联合索引（也叫复合索引、组合索引）

#### 常见索引数据结构和区别

常见的索引数据结构：二叉树、红黑树、B 树、B+ 树

区别：树的高度影响获取数据的性能（每个树节点都是一次磁盘 I /O)

#### 二叉树

特点：每个节点最多有两个子节点，小在左，大在右，数据随机性情况下树杈越明显。

![二叉树示例1](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E4%BA%8C%E5%8F%89%E6%A0%91%E7%A4%BA%E4%BE%8B1.png)

如果数据是按顺序依次进入：
树的高度则会很高（就是一个链表结构），此时元素的查找效率就等于链表查询 $O_{(n)}$，数据检索效率极为低下。

极端情况下，就是一个链表结构（如下图），此时元素的查找效率就等于连表查询 $O_{(n)}$。

![二叉树示例2](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E4%BA%8C%E5%8F%89%E6%A0%91%E7%A4%BA%E4%BE%8B2.png)

#### 红黑树（平衡二叉树）

虽通过自旋平衡，子节点会自动分叉为 2 个分叉，从而减少树的高度，当数据有序插入时比二叉树数据检索性能更佳。
但是如果数据量过大，节点个数就越多，树高度也会增高（也就是树的深度越深），增加磁盘 I/O 次数，影响查询效率。

![平衡二叉树的旋转](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E5%B9%B3%E8%A1%A1%E4%BA%8C%E5%8F%89%E6%A0%91%E7%9A%84%E6%97%8B%E8%BD%AC.gif)

#### B- 树（平衡多路查找树）

B 树的出现可以解决树高度的问题。
之所以是 B 树，而并不是名称中“XXX二叉树”，也就是它不再限制一个父节点中只能有两个子节点，而是允许 M 个子节点（M > 2）。不仅如此，B 树的一个节点可以存储多个元素，相比较于前面的那些二叉树数据结构又将整体的树高度降低了。

> MySQL 中是 16 阶平衡多路查找树，每个节点最多可以存储 15 个元素，最多有 16 个分支。

![B 树（多路平衡二叉树）](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/B%20%E6%A0%91%EF%BC%88%E5%A4%9A%E8%B7%AF%E5%B9%B3%E8%A1%A1%E4%BA%8C%E5%8F%89%E6%A0%91%EF%BC%89.png)

B 树的节点可以包含有多个子节点，所以 **B 树是一棵多叉树**，它的每一个节点包含的最多子节点数量的称为 B 树的阶。
如下图是一棵三阶的 B 树的插入过程：

![B- Tree 插入过程](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/B-%20Tree%20%E6%8F%92%E5%85%A5%E8%BF%87%E7%A8%8B.gif)

当一棵 3 阶的 B 树查找 7 这个元素是的流程：
先从根节点出发，判断 7 在 4 和 8 之间，根据 P2 存储指针 6 的节点，判断 7 大于 6，最后指针找到叶子节点，也就找到了匹配 7 的键值。

> 可以发现一棵 3 阶的 B 树在查找叶子节点时，由于树高度只有 3，所以查找过程最多只需要 3 次的磁盘 I/O 操作。
> 数据量不大时可能不太真切，但当数据量大时，节点也会随着增多，此时如果还是自平衡二叉树的场景下，由于二叉树只能最多 2 个叶子节点的约束，也只能纵向扩展子节点，树的高度会很高，意味着需要更多的操作磁盘 I/O 次数。
> B- Tree 则可以通过横向扩展节点从而降低树的高度，所以效率自然要比二叉树效率更高。
>
> B 树其实已经减少了磁盘 I/O 的操作，同时支持按区间查找，但并不高效。
>
> B- Tree 范围查找效率低下。

#### B+ 树

B+ Tree 是在 B- Tree 基础上的一种优化，其更适合作存储索引结构。
在 B+ Tree 中，非叶子节点仅存储键值，不存储数据，而所有数据记录均存储在叶子节点上，并且数据是按照顺序排列的。
在 B+ Tree 中，各个数据页之间是通过双向链表连接的，叶子节点中的数据是通过单向链表连接的。

B+ Tree 结构图如下：

![B+ Tree 结构图](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/B+%20Tree%20%E7%BB%93%E6%9E%84.png)

---

B 树和 B+ 树的区别，MySQL 为什么要选择 B+ 树作为默认索引的数据结构？

B+ Tree 结构实现数据索引具有如下优点：

- 非叶子节点上可以存储更多的键值，相应的树的阶数（节点的子节点树）就会更大【树也就会变得更矮更胖】。这样查找数据进行磁盘 I/O 的次数就会大大减少，数据查询的效率也会更快；
- 所有数据记录都有序存储在叶子节点上，就会使得范围查找、排序查找、分组查找以及去重查找变得异常简单；
- 数据页之间、数据记录之间都是通过链表链接的，有了这个结构的支持就可以方便地在数据查询后进行升序或者降序操作。

如果一个表没有主键索引还会创建 B+ 树吗？

会的！
InnoDB 是 MySQL 中的一种存储引擎，它会为每个表创建一个主键索引。如果表没有明确的主键索引，InnoDB 会使用一个隐藏的、自动生成的主键来创建索引。这个隐藏的主键索引使用的就是 B+ 树结构。
因此，在 InnoDB 中，即使表没有明确的主键索引，也会创建一个 B+ 树索引。

### Hash 索引

Hash 索引其实用的不多，最主要是因为最常见的存储引擎 InnoDB 不支持显式的创建 Hash 索引，只支持自适应 Hash 索引。
虽然可以使用 SQL 语句在 InnoDB 显式声明 Hash 索引，但其实是不生效的，**通过 `show index from TableName` 就会发现实际还是 B+ 树**。

在存储索引中，Memory 引擎支持 Hash 索引。

![Hash 索引结构](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/Hash%20%E7%B4%A2%E5%BC%95%E7%BB%93%E6%9E%84.png)

#### Hash 索引优缺点

- hash 索引只能用于等值比较，所以查询效率非常高
- 不支持范围查询，也不支持排序，因为索引列的分布是无序的

### 聚簇索引与非聚簇索引

按物理存储分类：InnoDB 得存储方式是聚集索引，MyISAM 的存储方式是非聚簇索引。

![MyISAM 非聚簇索引结构](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/MyISAM%20%E9%9D%9E%E8%81%9A%E7%B0%87%E7%B4%A2%E5%BC%95%E7%BB%93%E6%9E%84.png)

![InnoDB 聚簇索引结构](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/InnoDB%20%E8%81%9A%E7%B0%87%E7%B4%A2%E5%BC%95%E7%BB%93%E6%9E%84.png)

#### 聚簇索引

1. 聚簇索引将数据存储在所引述的叶子节点上
2. 聚簇索引可以减少一次查询，因为查询索引树的同时就能获取到数据
3. 聚簇索引的缺点是对数据进行修改或删除时需要更新索引树，会增加系统的消耗
4. 聚簇索引通常用于数据库系统中，主要用于提高查询效率

#### 非聚簇索引（二级索引 / 辅助索引）

1. 非聚簇索引不将数据存储在索引树的叶子节点上，而是存储在数据页中
2. 非聚簇索引在查询数据时需要两次查询：一次查询索引树，获取数据页的地址；再通过数据页的地址查询数据**【但如果是索引覆盖的话实际上是不用回表的】**
3. 非聚簇索引的优点：对数据进行修改或删除操作时不需要更新索引树，减少了系统的消耗
4. 非聚簇索引通常用于数据库系统中，主要用于提高数据更新和删除操作的效率

### 二级索引

在 MySQL 中，创建一张表时会默认为主键创建聚簇索引，B+ 树将表中所有的数据组织起来，即数据就是索引主键，所以在 InnoDB 里，主键索引也被称为聚簇索引，索引的叶子节点存的是整行数据。
除了聚簇索引以外的所有索引都称之为二级索引，二级索引的叶子节点内容是主键的值。

```sql
-- 例如创建一张表
CREATE TABLE users (
	id INT NOT NULL,
    name VARCHAR(20) NOT NULL,
  	age INT NOT NULL,
  	PRIMARY KEY(id)
);

-- 新建一个以age字段的二级索引:
ALTER TABLE users ADD INDEX index_age(age);

-- MySQL会分别创建主键id的聚簇索引和age的二级索引
```

![ID 主键索引和 Age 二级索引](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/ID%20%E4%B8%BB%E9%94%AE%E7%B4%A2%E5%BC%95%E5%92%8C%20Age%20%E4%BA%8C%E7%BA%A7%E7%B4%A2%E5%BC%95.png)

在 MySQL 中主键索引的叶子节点存储整行数据，二级索引叶子节点存储主键的值。

### 回表

```sql
-- 假设对 name 字段创建了一个索引，再执行下面这条 sql 语句
SELECT * FROM users WHERE age = 35;
```

![二级索引回表查询过程](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E4%BA%8C%E7%BA%A7%E7%B4%A2%E5%BC%95%E5%9B%9E%E8%A1%A8%E6%9F%A5%E8%AF%A2%E8%BF%87%E7%A8%8B.png)

由于查询条件是 name，所以会走 name 索引。整个过程大致分为以下步骤：

1. 从根节点出发，21 < 35，定位右边指针
2. 在索引叶子节点找到 35 的第一条记录，也就是 id = 9 的那条
3. 由于是 select *，还要查其他字段，此时就会根据 id = 9 到聚簇索引（主键索引）中查找其他字段数据，这个查找过程就被称为**回表**。

### 覆盖索引

```sql
-- 执行如下 sql 语句
select id from users where age = 35;
```

这次查询字段从 select * 变成 select id，查询条件不变，所以也会走 age 索引。

![ID 主键索引和 Age 二级索引](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/ID%20%E4%B8%BB%E9%94%AE%E7%B4%A2%E5%BC%95%E5%92%8C%20Age%20%E4%BA%8C%E7%BA%A7%E7%B4%A2%E5%BC%95.png)

所以还是跟前面一样了，先从索引页中查出来 age = 35 对应的主键 id 之后发现，SQL 中需要查询字段的 id 值已经查到了。

而这种需要查询的字段都在索引列中的情况就被称为**覆盖索引**，索引列覆盖了查询字段的意思。

当使用覆盖索引时会减少回表的次数，这样查询速度更快，性能更高。

所以，在日常开发中，尽量不要 select * ，需要什么查什么，如果出现覆盖索引的情况，查询会快很多。

### 单列索引

```sql
ALTER TABLE users  ADD INDEX(name);
```

假设，对 name 字段加了一个普通非唯一索引，那么 name 就是索引列，同时 name 这个索引也就是单列索引。
此时如果往表中插入三条数据，那么 name 索引的叶子节点存的数据就如下图所示

![name 索引叶子节点数据](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/name%20%E7%B4%A2%E5%BC%95%E5%8F%B6%E5%AD%90%E8%8A%82%E7%82%B9%E6%95%B0%E6%8D%AE.png)

**MySQL 会根据 name 字段的值进行排序，这里假设张三排在李四前面，当索引列的值相同时，就会根据 id 排序，所以索引实际上已经根据索引列的值排好序了。**

MySQL 支持很多种排序规则，在建数据库或者是建表的时候等都可以指定排序规则。

![建库时指定排序规则](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E5%BB%BA%E5%BA%93%E6%97%B6%E6%8C%87%E5%AE%9A%E6%8E%92%E5%BA%8F%E8%A7%84%E5%88%99.png)

对于单个索引列数据查找也是跟聚簇索引一样，也会对数据分组，之后可以根据二分查找在单个索引列来查找数据。

当数据不断增多，一个索引页存储不下数据的时候，也会用多个索引页来存储，并且索引页直接也会形成双向链表。

![单列索引-多个索引页存储数据](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E5%8D%95%E5%88%97%E7%B4%A2%E5%BC%95-%E5%A4%9A%E4%B8%AA%E7%B4%A2%E5%BC%95%E9%A1%B5%E5%AD%98%E5%82%A8%E6%95%B0%E6%8D%AE.png)

当索引页不断增多是，为了方便在不同索引页中查找数据，也就会抽取一个索引页，除了存页中 id，同时也会存储这个 id 对应的索引列的值

![单列索引-索引页存储对应索引列](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E5%8D%95%E5%88%97%E7%B4%A2%E5%BC%95-%E7%B4%A2%E5%BC%95%E9%A1%B5%E5%AD%98%E5%82%A8%E5%AF%B9%E5%BA%94%E7%B4%A2%E5%BC%95%E5%88%97.png)

当数据越来越多越来越多，还会抽取，也会形成三层的一个B+树。

### 联合索引

```sql
ALTER TABLE users ADD INDEX(name, age, id);
```

>  除了单列索引，联合索引其实也是一样的，只不过索引页存的数据就多了一些索引列。

比如，在 name 和 age 上建立一个联合索引，此时单个索引页就如图所示

![联合索引-索引页](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E8%81%94%E5%90%88%E7%B4%A2%E5%BC%95-%E7%B4%A2%E5%BC%95%E9%A1%B5.png)

 

**先以 name 排序，name 相同时再以 age 排序，如果再有其它列，依次类推，最后再以 id 排序。**

相比于只有 name 一个字段的索引来说，索引页就多存了一个索引列。

最后形成的 B+ 树简化为如下图：

![联合索引-B+ Tree 结构](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E8%81%94%E5%90%88%E7%B4%A2%E5%BC%95-B+%20Tree%20%E7%BB%93%E6%9E%84.png)

#### 最左前缀原则

顾名思义是最左优先，以最左边的为起点任何连续的索引都能匹配上。

如果没有第一列的话，直接访问第二列，那第二列肯定是无序的，直接访问后面的列就用不到索引了

> 当创建 (a,b,c) 复合索引时，想要索引生效的话，只能使用 a 和 ab、ac 和 abc 三种组合！

#### 单列索引、联合索引分别的适用场景，联合索引的优势

1. 减少开销

   建一个联合索引 (a,b,c)，实际相当于建了 (a)、(a,b)、(a,b,c) 三个索引.每多一个索引,都会增加写操作的开销和磁盘空间的开销。对于大量数据的表,使用联合索引会大大的减少开销！

2. 覆盖索引

   对联合索引 (a,b,c)，如果有如下 SQL：

   ```sql
   select a,b,c from table where a='xxx' and b='xx';
   ```

   那么 MySQL 可以直接通过遍历索引取得数据,而无需回表，这减少了很多的随机 I/O 操作。
   减少 I/O 操作,特别是随机 I/O 其实是 DBA 主要的优化策略。所以，在真正的实际应用中，覆盖索引是主要的提升性能的优化手段之一。

3. 效率高

   索引列多，通过联合索引筛选出的数据越少。比如有 1000w 条数据的表,有如下 SQL:

   ```sql
   select col1,col2,col3 from table where col1=1 and col2=2 and col3=3;
   ```

   假设：每个条件可以筛选出 10% 的数据

   A：如果只有单列索引，那么通过该索引能筛选出 `1000w * 10%=100w` 条数据，然后再回表从 100w 条数据中找到符合 `col2=2 and col3=3` 的数据，然后再排序，再分页，以此类推(递归)；

   B：如果是 (col1,col2,col3) 联合索引,通过三列索引筛选出 `1000w * 10% * 10% * 10%=1w`

### 索引下推

索引下推（INDEX CONDITION PUSHDOWN，简称 ICP）是在 MySQL 5.6 针对**扫描二级索引**的一项优化改进。 用来在范围查询时减少回表的次数 。ICP 适用于 MYISAM 和 INNODB。

```sql
ALTER TABLE users  ADD INDEX (name, age);
```

不使用索引下推实现：
![不使用索引下推实现模糊查询](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E4%B8%8D%E4%BD%BF%E7%94%A8%E7%B4%A2%E5%BC%95%E4%B8%8B%E6%8E%A8%E5%AE%9E%E7%8E%B0%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2.png)

```sql
-- 使用 EXPLAIN 关键字可以模拟优化器执行 SQL 查询语句，从而知道 MySQL 是如何处理 SQL 语句的。分析查询语句或是表结构的性能瓶颈
Explain SELECT * FROM user1 WHERE name LIKE 'A%' and age = 40;
```

![使用 EXPLAIN 分析不使用索引下推时的模糊查询](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E4%BD%BF%E7%94%A8%20EXPLAIN%20%E5%88%86%E6%9E%90%E4%B8%8D%E4%BD%BF%E7%94%A8%E7%B4%A2%E5%BC%95%E4%B8%8B%E6%8E%A8%E6%97%B6%E7%9A%84%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2.png)

使用索引下推实现
![使用索引下推实现模糊查询](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E4%BD%BF%E7%94%A8%E7%B4%A2%E5%BC%95%E4%B8%8B%E6%8E%A8%E5%AE%9E%E7%8E%B0%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2.png)

```sql
Explain SELECT * FROM user1 WHERE name LIKE 'A%' and age = 40;
```

![使用 EXPLAIN 分析使用索引下推时的模糊查询](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E4%BD%BF%E7%94%A8%20EXPLAIN%20%E5%88%86%E6%9E%90%E4%BD%BF%E7%94%A8%E7%B4%A2%E5%BC%95%E4%B8%8B%E6%8E%A8%E6%97%B6%E7%9A%84%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2.png)

![索引下推引入](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E7%B4%A2%E5%BC%95%E4%B8%8B%E6%8E%A8%E5%BC%95%E5%85%A5.png)

接下来要执行如下的 SQL

```sql
select * from users where name > '王五' and age > 22;
```

在 MySQL5.6 （不包括 5.6）之前，整个  SQL 大致执行步骤如下：

- 先根据二分查找，定位到 name > '王五' 的第一条数据，也就是 id=4 的那个赵六
- 之后就会根据 id=4 进行回表操作，到聚簇索引中查找 id=4 其它字段的数据，然后判断数据中的 age 是否大于 22，是的话就说明是我们需要查找的数据，否则就不是
- 之后顺着链表，继续遍历，然后找到一条记录就回一次表，然后判断 age，如此反复下去，直至结束

所以对于图上所示，整个搜索过程会经历 5 次回表操作，两个赵六，两个刘七，一个王九，最后符合条件的也就是 id=6 的赵六那条数据，其余 age 不符和。

虽然这么执行没什么问题，但是不知有没有发现其实没必要进行那么多次回表，因为光从上面的索引图示就可以看出，符合 `name > '王五' and age > 22` 的数据就 id=6 的赵六那条数据

所以在 MySQL5.6 之后，对上面的 age > 22 判断逻辑进行了优化

前面还是一样，定位查找到 id=4 的那个赵六，之后就**不回表**来判断 age 了，因为索引列有 age 的值了，那么直接根据索引中 age 判断是否大于 22，如果大于的话，再回表查询剩余的字段数据（因为是 `select *`），然后再顺序链表遍历，直至结束

所以这样优化之后，回表次数就成1了，相比于前面的 5 次，大大减少了回表的次数。

而这个优化，就被称为**索引下推**，就是为了减少回表的次数。

### 索引合并

索引合并（index merge）是从 MySQL5.1 开始引入的索引优化机制，在之前的 MySQL 版本中，一条 SQL多个查询条件只能使用一个索引，但是引入了索引合并机制之后，MySQL 在**某些特殊**的情况下会扫描多个索引，然后将扫描结果进行合并

结果合并会为下面三种情况：

- 取交集（intersect）
- 取并集（union）
- 排序后取并集（sort-union）

name 和 age 各自分别创建一个二级索引 `idx_name` 和 `idx_age`

#### 取交集（intersect）

当执行下面这条 SQL 就会出现取交集的情况

```sql
select * from users where name = '赵六' and age= 22;
```

查看执行计划

![执行计划-年龄等于](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E6%89%A7%E8%A1%8C%E8%AE%A1%E5%88%92-%E5%B9%B4%E9%BE%84%E7%AD%89%E4%BA%8E.png)

type是index_merge，并且possible_key和key都是idx_name和idx_age，说明使用了索引合并，并且Extra有Using intersect(idx_age, idx_name)，intersect 就是交集的意思。

整个过程大致是这样的，分别根据 idx_name 和 idx_age 取出对应的主键 id，之后将主键 id 取交集，那么这部分交集的 id 一定同时满足查询 `name = '赵六' and age= 22` 的查询条件（仔细想想），之后再根据交集的 id 回表

不过要想使用取交集的联合索引，需要满足各自索引查出来的主键 id 是排好序的，这是为了方便可以快速的取交集

比如下面这条 SQL 就无法使用联合索引：

```sql
select * from users where name = '赵六' and age > 22;
```

![执行计划-年龄大于](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E6%89%A7%E8%A1%8C%E8%AE%A1%E5%88%92-%E5%B9%B4%E9%BE%84%E5%A4%A7%E4%BA%8E.png)

只能用 name 这个索引，因为 age > 22 查出来的 id 是无序的。

#### 取并集（union）

取并集就是将前面例子中的 `and` 换成` or`

```sql
select * from users where name = '赵六' or age = 22;
```

前面执行的情况都一样，根据条件到各自的索引上去查，之后对查询的 id 取并集去重，之后再回表

同样地，取并集也要求各自索引查出来的主键 id 是排好序的，如果查询条件换成 age > 22 时就无法使用取并集的索引合并

```sql
select * from users where name = '赵六' or age > 22;
```

#### 排序后取并集（sort-union）

虽然取并集要求各自索引查出来的主键 id 是排好序的，但是如果遇到没排好序的情况，MySQL 会自动对这种情况进行优化，会先对主键 id 排序，然后再取并集，这种情况就叫排序后取并集（sort-union）。

比如上面提到的无法直接取并集的 SQL 就符合排序后取并集（sort-union）这种情况

```sql
select * from users where name = '赵六' or age > 22;
```

---

---

### 索引的优缺点

优点：

- 提高检索效率
- 降低排序成本，索引对应的字段会有自动排序功能，默认升序 ASC

缺点：

- 创建索引和维护索引耗费时间，且会随着数据量的增加而增加
- 索引需要占用物理空间，数据量越大，占用空间越大
- 降低表的增删改效率。因为每次增删改索引都需要进行动态维护

### 索引的使用场景

适用场景：比较频繁的作为查询条件的字段应该创建索引

非适用场景：

- 字段值的唯一性太差不适合单独作为索引
- 更新非常频繁的字段
- 不会出现在 where 条件语句中的字段

## 优化

### 优化方法

![优化方法](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/%E4%BC%98%E5%8C%96%E6%96%B9%E6%B3%95.png)

关于 SQL 优化方法，包括 5 点：

1. 创建索引减少扫描量；
2. 调整索引减少计算量；
3. 索引覆盖（减少不必访问的列，避免回表查询）； 
4. 干预执行计划；  
5. SQL改写； 

### 通过 Explain 干预执行计划

![Explain 详解思维导图](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/Explain%20%E8%AF%A6%E8%A7%A3%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE.jpeg)

#### Explain含义

Explain 是 SQL 分析工具中非常重要的一个功能，它可以模拟优化器执行查询语句，帮助我们理解查询是如何执行的；分析查询执行计划可以帮助我们发现查询瓶颈，优化查询性能。

#### Explain 作用

- 表的读取顺序
- SQL 执行时查询操作类型
- 可以使用哪些索引
- 实际使用哪些索引
- 每张表有多少行记录被扫描
- **SQL语句性能分析**

#### Explain 用法

```sql
drop table orders;
drop table products;
drop table users;
CREATE TABLE users (  
  id INT PRIMARY KEY AUTO_INCREMENT,  
  name VARCHAR(50) NOT NULL,  
  email VARCHAR(100) NOT NULL,  
  password VARCHAR(100) NOT NULL  
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE products (  
  id INT PRIMARY KEY AUTO_INCREMENT,  
  name VARCHAR(50) NOT NULL,  
  price FLOAT NOT NULL  
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE orders (  
  id INT PRIMARY KEY AUTO_INCREMENT,  
  user_id INT NOT NULL,  
  order_date DATETIME NOT NULL,  
  total_price FLOAT NOT NULL,  
  product_id INT NOT NULL,  
  FOREIGN KEY (user_id) REFERENCES users(id),  
  FOREIGN KEY (product_id) REFERENCES products(id)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

alter table users add index index_name_email (name,email);

INSERT INTO users (name, email, password)     
VALUES ('张三', 'zhangsan@example.com', 'password123'),     
('李四', 'lisi@example.com', 'password123'),     
('王五', 'wangwu@example.com', 'password123'),     
('赵六', 'zhaoli@example.com', 'password123'),     
('钱七', 'qianqi@example.com', 'password123');   

INSERT INTO products (name, price)     
VALUES ('产品 1', 10.00),     
('产品 2', 15.00),     
('产品 3', 20.00),     
('产品 4', 12.00),     
('产品 5', 18.00); 

INSERT INTO orders (user_id, order_date, total_price, product_id)     
VALUES (1, '2023-02-18 10:00:00', 100.00, 1),     
(2, '2023-02-18 11:00:00', 50.00, 2),     
(3, '2023-02-18 12:00:00', 20.00, 3),     
(4, '2023-02-18 13:00:00', 15.00, 4),     
(5, '2023-02-18 14:00:00', 25.00, 5); 
```

MySQL 5.7 版本之前，使用 Explain Extended 在 Explain 的基础上额外多返回 filtered 列与 extra 列；

```sql
Explain Extended select * from users;
```

MySQL 5.7 版本之前，使用 Explain Partitions 在 Explain 的基础上额外多返回 partitions 列；

```sql
Explain Partitions select * from users;
```

MySQL 5.7 版本引入了这两个特性，直接使用 Explain 关键字可以将 partitions 列、filtered 列、extra 列直接查询出来。

```sql
Explain select * from users;
```

![Explain 关键字执行结果](R:/iCloudDrive/mine/ImageRepository/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF/MySQL/Explain%20%E5%85%B3%E9%94%AE%E5%AD%97%E6%89%A7%E8%A1%8C%E7%BB%93%E6%9E%9C.png)

Explain 语句返回列的各列含义：

| **列名**      | **含义**                                              |
| ------------- | ----------------------------------------------------- |
| id            | 每个 select 都有一个对应的id号，并且是从 1 开始自增的 |
| select_type   | 查询语句执行的查询操作类型                            |
| table         | 表名                                                  |
| partitions    | 表分区情况                                            |
| type          | 查询所用的访问类型                                    |
| possible_keys | 可能用到的索引                                        |
| key           | 实际查询用到的索引                                    |
| key_len       | 所使用到的索引长度                                    |
| ref           | 使用到索引时，与索引进行等值匹配的列或者常量          |
| rows          | 预计扫描的行数（索引行数或者表记录行数）              |
| filtered      | 表示符合查询条件的数据百分比                          |
| Extra         | SQL 执行的额外信息                                    |

#### Explain 返回列详解

##### 1. id 列

每个 select 都有一个对应的 id 号，并且是从 1 开始自增的。

- 如果 id 序号相同，从上往下执行。
- 如果 id 序号不同，序号大先执行。
- 如果两种都存在，先执行序号大，在同级从上往下执行。
- 如果显示 NULL，最后执行；表示结果集，并且不需要使用它来进行查询。

##### 2. select_type列

表示查询语句执行的查询操作类型。

###### 2.1 simple

简单 select，不包括 union 与子查询。

###### 2.2 primary

复杂查询中最外层查询，比如使用 union 或 union all 时，id 为 1 的记录 select_type 通常是 primary。

###### 2.3 subquery

指在 select 语句中出现的子查询语句,结果不依赖于外部查询（不在 from 语句中）。

###### 2.4 dependent subquery

指在 select 语句中出现的查询语句，结果依赖于外部查询。

###### 2.5 derived

派生表，在 FROM 子句的查询语句，表示从外部数据源中推导出来的，而不是从  SELECT 语句中的其他列中选择出来的。

###### 2.6 union

分 union 与 union all 两种，若第二个 select 出现在 union 之后，则被标记为 union；如果 union 被 from 子句的子查询包含，那么第一个 select 会被标记为 derived；union 会针对相同的结果集进行去重，union all 不会进行去重处理。

###### 2.7 dependent union

当 union 作为子查询时，其中第一个 union 为 dependent subquery，第二个 union 为 dependent union。

###### 2.8 union result

如果两个查询中有相同的列，则会对这些列进行重复删除，只保留一个表中的列。

##### 3. table列

查询所涉及的表名。如果有多个表，将显示多行记录

##### 4. partitions列

表分区情况。

查询语句所涉及的表的分区情况。具体来说，它会显示出查询语句在哪些分区上执行，以及是否使用了分区裁剪等信息。如果没有分区，该项为 NULL。

##### 5. type列

查询所使用的访问类型

效率从高到低分别为：**system > const > eq_ref > ref** > fulltext > ref_or_null **> range > index > ALL，**一般来说保证range级别，最好能达到ref级别。

###### 5.1 system

const 类型的一种特殊场景，查询的表只有一行记录的情况，并且该表使用的存储引擎的统计数据是精确的

InnoDB 存储引擎的统计数据不是精确的，虽然只有一条数据但是type类型为ALL；

```sql
DROP TABLE t;
CREATE TABLE t(i INT) ENGINE=InnoDb;
INSERT INTO t VALUES(1);
explain select * from t;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681365781705-5ee3b302-7e4c-40ed-94a8-72759b980e92.png)

Memory 存储引擎的统计数据是精确的，所以当只有一条记录的时候type类型为system。

```sql
DROP TABLE tt;
CREATE TABLE tt(i INT) ENGINE=memory;
INSERT INTO tt VALUES(1);
explain select * from tt;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681368181258-8eb20a57-c719-4756-afc6-1f0ec8a72ad2.png)

###### 5.2 const

基于主键或唯一索引查看一行，当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问转换成常量查询，效率高

```sql
explain
select * from orders where id = 1;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681366253574-0fa5ca6c-2d64-40f0-9a01-2454c736e53a.png)

###### 5.3 eq_ref

基于主键或唯一索引连接两个表，对于每个索引键值，只有一条匹配记录，被驱动表的类型为'eq_ref'

```sql
explain
select users.* from users inner join orders on users.id = orders.id;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681366272461-a8f66d17-3156-448c-97c5-d9421863ebb1.png)

###### 5.4 ref

基于非唯一索引连接两个表或通过二级索引列与常量进行等值匹配，可能会存在多条匹配记录

1. 关联查询，使用非唯一索引进行匹配。

```sql
explain
select users.* from users inner join orders on users.id = orders.user_id;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681366725970-346f1c6a-4c3c-4076-a06a-239a9f1c28f7.png)

2. 简单查询，使用二级索引列匹配。

```sql
explain
select * from orders where user_id = 1;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681366311397-1eb1ba55-cb00-464b-b3c8-2949bd18a718.png)

###### 5.5 range

使用非唯一索引扫描部分索引，比如使用索引获取某些范围区间的记录

```sql
explain
select * from orders where user_id > 3;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681366328856-b4f80e75-c7d3-40c7-a2c7-21c53c900bfb.png)

###### 5.6 index

扫描整个索引就能拿到结果，一般是二级索引，这种查询一般为使用覆盖索引（需优化，缩小数据范围）

```sql
explain
select user_id from orders;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681366836249-8d05dc8f-5859-467d-ac77-d5d95a7201ef.png)

###### 5.7 all

扫描整个表进行匹配，即扫描聚簇索引树（需优化，添加索引优化）

```sql
explain
select * from users;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681039103901-8b407ac2-cdb5-44c1-b3e4-3f35818907a7.png#averageHue=%23f9f8f7&clientId=u3c4b35c8-f43a-4&from=paste&height=90&id=u0ba8c3b4&name=image.png&originHeight=222&originWidth=2508&originalType=binary&ratio=2.4750001430511475&rotation=0&showTitle=false&size=91162&status=done&style=none&taskId=ua18177c9-8bb0-4230-b5e3-eeb307e3a25&title=&width=1013.3332747642473)

###### 5.8 NULL

MySQL 在优化过程中分解语句就已经可以获取到结果，执行时甚至不用访问表或索引。

```sql
explain 
select min(id) from users;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681039399639-75db39c7-cf2f-49c0-92fb-b00364a29c8d.png#averageHue=%23f9f8f7&clientId=u3c4b35c8-f43a-4&from=paste&height=87&id=u05b4bc7a&name=image.png&originHeight=216&originWidth=2621&originalType=binary&ratio=2.4750001430511475&rotation=0&showTitle=false&size=96676&status=done&style=none&taskId=uc8883d0d-ea15-424f-85ed-4f9afa42e25&title=&width=1058.9898377819347)

##### 6. possible_keys列

表示在查询中可能使用到某个索引或多个索引；如果没有选择索引，显示NULL

##### 7. key列

表示在查询中实际使用的索引，如果没有使用索引，显示NULL。

##### 8. key_len列

表示当优化器决定使用某个索引执行查询时，该索引记录的最大长度（主要使用在联合索引）

联合索引可以通过这个值算出具体使用了索引中的哪些列。

使用单例索引：

```sql
explain  
select * from users where id = 1;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681384753103-94604447-d2b6-49bb-bc4c-23c14b1a076b.png)



使用联合索引：

```sql
explain 
select * from users where name = '张三' and email = 'zhangsan@example.com';
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681043991351-f81c0cc8-6968-40b2-9e4b-f8163585d542.png#averageHue=%23f9f7f6&clientId=u3c4b35c8-f43a-4&from=paste&height=77&id=u98802bfd&name=image.png&originHeight=191&originWidth=2511&originalType=binary&ratio=2.4750001430511475&rotation=0&showTitle=false&size=99786&status=done&style=none&taskId=uc314c65d-cfe5-4c41-b9e6-e67085dcf97&title=&width=1014.5453959063099)
计算规则：

- 字符串：

char(n)：n个字节 

varchar(n)：如果是uft-8：3n+2字节，加的2个字节存储字符串长度。如果是utf8mb4：4n+2字节。

- 数值类型：

tinyint：1字节

smaillint：2字节

int：4字节

bigint：8字节

- 时间类型：

date：3字节

timestamp：4字节

datetime：8字节
字段如果为NULL，需要1个字节记录是否为NULL

![img](https://cdn.nlark.com/yuque/0/2023/png/22309163/1686491522804-98efd37f-1489-4202-8b36-51c2fdafa9c2.png)

##### 9. ref列

表示将哪个字段或常量和key列所使用的字段进行比较。

当使用索引列等值查询时，与索引列进行等值匹配的对象信息。

1.常量：

```sql
explain 
select * from users where name = '张三' and email = 'zhangsan@example.com';
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681447679631-4023e7e4-924b-45d9-97bc-a3015f5e2e89.png)



2.字段：

```sql
explain
select users.* from users inner join orders on users.id = orders.id;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681446642630-3813f19b-6e70-49d4-82da-12f4fdb00754.png)



3.函数

```sql
explain
select users.* from users inner join orders on users.id = trim(orders.id);
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681454629742-f28fd829-b9e5-4e5e-9488-9d29f3faba65.png)

##### 10. rows列

全表扫描时表示需要扫描表的行数估计值；索引扫描时表示扫描索引的行数估计值；值越小越好（不是结果集中的行数）

1.全表扫描

```sql
explain
select * from orders where user_id >= 3 and total_price = 25;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681449747200-d5473ed5-9cec-4ad0-b696-afc1c1427bc3.png)



2.索引扫描

```sql
explain
select * from orders where user_id > 3;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681449133838-d210e8ea-6a40-4ccb-8ac3-c4d810032f1c.png)

##### 11. filtered列

表示符合查询条件的数据百分比。可以使用rows * filtered/100计算出与**explain**前一个表进行连接的行数。

前一个表指 explain 中的id值比当前表id值小的表，id相同的时候指后执行的表。

```sql
explain
select users.* from users inner join orders on users.id = orders.id;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681449894480-5b14d3bf-04ee-450b-bdf8-150cfa66ec57.png)

##### 12. Extra列

SQL执行查询的一些额外信息

###### 12.1 Using Index

使用非主键索引树就可以查询所需要的数据。一般是覆盖索引，即查询列都包含在辅助索引树叶子节点中，不需要回表查询。

```sql
explain
select user_id,id from orders where user_id = 1;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681044775275-6a7736f1-9b4a-4d35-ba85-dbedd53e6823.png#averageHue=%23f9f8f7&clientId=u3c4b35c8-f43a-4&from=paste&height=76&id=u7b383be1&name=image.png&originHeight=188&originWidth=2615&originalType=binary&ratio=2.4750001430511475&rotation=0&showTitle=false&size=94668&status=done&style=none&taskId=u6b2fb561-6ad5-47d3-94f0-ca68ee46dc6&title=&width=1056.5655954978097)

###### 12.2 Using where

不通过索引查询所需要的数据

```sql
explain
select * from orders where total_price = 100;

explain
select * from orders where user_id = 1 and total_price = 100;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681044854093-370ad425-71a6-4008-b7be-55467d1c9251.png#averageHue=%23f9f8f6&clientId=u3c4b35c8-f43a-4&from=paste&height=89&id=u580f800a&name=image.png&originHeight=220&originWidth=2326&originalType=binary&ratio=2.4750001430511475&rotation=0&showTitle=false&size=95387&status=done&style=none&taskId=u3ab5621d-ac5c-4cde-9102-073d8efd665&title=&width=939.7979254791226)

###### 12.3 Using index condition

表示查询列不被索引覆盖，where 条件中是一个索引范围查找，过滤完索引后回表找到所有符合条件的数据行。

```sql
explain
select * from orders where user_id > 3;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681045567332-67e9a87a-1b04-4693-b226-f427bc53fc45.png#averageHue=%23f8f6f5&clientId=u3c4b35c8-f43a-4&from=paste&height=91&id=uf0a1f432&name=image.png&originHeight=224&originWidth=2621&originalType=binary&ratio=2.4750001430511475&rotation=0&showTitle=false&size=107179&status=done&style=none&taskId=u7f4995fd-7f94-4714-8ad4-f2448f17c3b&title=&width=1058.9898377819347)

###### 12.4 Using temporary

表示需要使用临时表来处理查询；

1. total_price列无索引，需要创建一张临时表进行去重

```sql
explain
select distinct total_price from orders;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681045772656-8572b080-7051-4df2-8625-980f54c5771a.png#averageHue=%23f9f8f7&clientId=u3c4b35c8-f43a-4&from=paste&height=88&id=u2e823e00&name=image.png&originHeight=217&originWidth=2577&originalType=binary&ratio=2.4750001430511475&rotation=0&showTitle=false&size=98613&status=done&style=none&taskId=uf48f26c4-0390-45c5-a40a-b6688b717c4&title=&width=1041.212061031685)

2. name列有联合索引

```sql
explain
select distinct name from users;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681046236030-f805a8ac-1946-4600-a5b9-6bf583f5e39e.png#averageHue=%23f8f7f6&clientId=u3c4b35c8-f43a-4&from=paste&height=80&id=u139ce865&name=image.png&originHeight=199&originWidth=2663&originalType=binary&ratio=2.4750001430511475&rotation=0&showTitle=false&size=104632&status=done&style=none&taskId=u1da6ac88-d3b9-4bbf-bfb3-99387eff1d9&title=&width=1075.9595337708097)

###### 12.5 Using filesort

当查询中包含 order by 操作而且无法利用索引完成的排序操作，数据较少时从内存排序，如果数据较多需要在磁盘中排序。	需优化成索引排序。

1. total_price列无索引，无法通过索引进行排序。需要先保存total_price与对应的主键id，然后在排序total_price查找数据。

```sql
explain
select total_price from orders order by total_price;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681464127193-d83100a9-f5c3-4139-a8d0-7225ec490e8f.png)

2. name列有索引，因索引已经是排好序的所以直接读取就可以了。

```sql
explain
select name from users order by name;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681474747964-b0738d83-519e-4e6e-a649-d3ac167a7d5e.png)

###### 12.6 Select tables optimized away

使用某些聚合函数（min,max）来访问某个索引值。

```sql
explain 
select min(id) from users;

explain 
select min(password) from users;
```

![img](https://cdn.nlark.com/yuque/0/2023/png/35268836/1681039399639-75db39c7-cf2f-49c0-92fb-b00364a29c8d.png#averageHue=%23f9f8f7&clientId=u3c4b35c8-f43a-4&from=paste&height=87&id=YclHG&name=image.png&originHeight=216&originWidth=2621&originalType=binary&ratio=2.4750001430511475&rotation=0&showTitle=false&size=96676&status=done&style=none&taskId=uc8883d0d-ea15-424f-85ed-4f9afa42e25&title=&width=1058.9898377819347)
