### 添加代码，不改变引用的情况下改变字符串的值

```java
public static void main(String[] args) throws NoSuchFieldException, IllegalAccessExcption {
	String string = new String("abc");
    
    //添加 N 行代码，但必须保证 string 引用的只想不变，最终将输出 abcd
    Field value = string.getClass().getDeclaredField("value");
    value.setAccessible(true);
    value.set(string, "abcd".toCharArray());

    System.out.println("string = " + string);
}
```

