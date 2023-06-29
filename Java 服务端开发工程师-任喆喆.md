1. `String`、`StringBuffer` 和 `StringBuilder` 的 使用场景

   使用 `String` 类的场景：在字符串不经常变化的场景中可以使用 `String` 类，例如常量的声明、少量的变量运算；

   使用 `StringBuffer` 类的场景：在频繁进行字符串运算（如拼接、替换、删除等），并且运行在多线程环境中，则可以考虑使用 `StringBuffer`，例如 XML 解析、HTTP 参数解析和封装；

   使用 `StringBuilder` 类的场景：在频繁进行字符串运算（如拼接、替换、和删除等），并且运行在单线程的环境中，则可以考虑使用 `StringBuilder`，如 SQL 语句的拼装、JSON 封装等。

2. 编写 java 代码遍历一个 `List<Integer>` 中的元素，每遍历一个控制台输出后立即删除

   ```java
   import java.util.*;
   
   public class Main {
   	public static void main(String[] args) {
   		ArrayList<Integer> list = new ArrayList<>();
   		list.add(1);
   		list.add(2);
   		list.add(3);
   		list.add(4);
   
   		Iterator iterator = list.iterator();
   		while (iterator.hasNext()) {
   			System.out.println(iterator.next());
   			iterator.remove();
   		}
   
   		if (list.isEmpty()) {
   			System.out.println("list 已空");
   		}
   	}
   }
   ```

3. 访问修饰符 `public`, `private`, `protected` 以及不写（默认 `default`）时的区别？

   `public`：Java 语言中最没有访问限制的修饰符，一般称之为“公共的”。被其修饰的类、属性以及方法不仅可以跨类访问，而且允许跨 package 访问；

   `private`：Java 语言中对访问权限限制最严谨的修饰符，一般称之为“私有的”。被其修饰的类、属性以及方法只能被该类的对象访问，其子类不能访问，更不允许被跨 package 访问；

   `protected`：介于 public 和 private 之间的一种访问修饰符，一般称之为“被保护”。被其修饰的类、属性以及方法只能被类本身的方法及子类访问，即使子类在不同的包中也可以访问；

   `default`：即不加任何访问修饰符，只允许在同一个 package 中进行访问。

4. `Integer` 和 `int` 的区别和联系。

   区别：

   - 数据类型不同：int 是基础数据类型，而 Integer 是包装数据类型；
   - 默认值不同：int 的默认值是 0，而 Integer 的默认值为 null；
   - 内存中存储的方式不同：int 在内存中直接存储的是数据值，而 Integer 实际存储的是对象引用，当 new 一个 Integer 时实际上生成一个指针指向此对象；
   - 实例化方式不同：Integer 必须实例化才可以使用，而 int 不需要；
   - 变量的比较方式不同：int 可以使用 `==` 来对比两个变量是否相等，而 Integer 一定要使用 `equals` 来比较两个变量是否相等。

   联系：

   - Integer 类在对象中包装了一个基本类型 int 的值；
   - Integer 类型的对象包含一个 int 类型的字段；
   - Integer 类提供了多个方法，能在 int 类型和 String 类型之间互相转换，还提供了处理 int 类型时非常有用的其他一些常量和方法。

5. String str = “a”  str==”a” 返回什么 ，两个对象什么情况下用==返回true。

   `str == "a"` 返回 true。

   当两个对象指向同一引用地址时用 == 返回 true。

6. 列举出你知道的当服务端接收到一条客户端（或者WEB）用户发送的请求的时候，验证用户请求合法性的方式

   - 验证请求头 `Request Headers`：通常请求头中包含一些用于验证身份的参数，例如 API Key、Token 等。可以在服务端接受到请求后，验证这些参数是否合法；
   - 验证请求参数 `Request Parameters`：有些 API 需要客户端发送一些参数或者数据，服务端可以验证这些参数是否合法；
   - 验证请求来源 `Request Origin`：有些 API 只允许特定的域名与 IP 地址访问，可以在服务端接收到请求后，验证请求来源是否合法；
   - 验证请求签名 `Request Signature`：有些 API 需要客户端发送数字签名，服务端可以通过验证数字签名的方式来验证请求合法性；
   - 验证请求时间戳 `Request Timestamp`：有些 API 对请求的响应时间有严格要求，服务端可以验证请求的时间戳来确保请求在有效期内。

7. 如下玩家获得道具的记录表 t_item_log

   | 自增主键 | 玩家 ID  | 道具 id | 道具改变数量（小于 0 为消耗） | 记录时间 | 玩家等级 |
   | -------- | -------- | ------- | ----------------------------- | -------- | -------- |
   | id       | playerid | itemid  | num                           | Time     | level    |
   | int      |          | int     | int                           | int64    | int      |

   （1）查询 2018 年 8 月 5 日每个玩家消耗道具的数量

   ```sql
   select playerid, itemid, sum(0 - num)
   from t_item_log
   where `time` = '20180805' and num < 0
   group by playerid, itemid;
   ```

   （2）查询玩家每一次消耗道具 ID 为 1 时的玩家等级

   ```sql
   select playerid, itemid, `level`
   from t_item_log
   where itemid = 1
   ```

8. 考试包含 M 个科目，有学生 A，B，C 参加，在每一科考试中，第一,第二,第三名分别得 X，Y，Z 分，其中 X,Y,Z 为正整数且 X>Y>Z。最后 A 得 22 分，B 与 C均得 9 分，B 在语文考试取得第一。求 M 的值，并问在数学比赛谁得第二名。

   M = 5

   C 在数学比赛得第二名

9. 张三、李四、王五他们知道 箱子里有16张牌：红桃 Q、4、A、黑桃 2、7、3、J、8、4 草花 K、Q、5、4、6 方块A、5。约翰教授从这16张牌中挑出一张牌来，并把这张牌的点数告诉了李四，把这张牌的花色告诉了王五。这时，约翰教授问李四和王五：你们能从已知的点数或花色中推知这张牌是什么牌吗？ 于是，张三听到如下的对话：

   李四：我不知道这张牌。
   王五：我知道你不知道这张牌。
   李四：现在我知道这张牌了。 
   王五：我也知道了。
   听完以上的对话，张三就是什么牌了。 
   请问：这张牌是什么牌？

   方块 5