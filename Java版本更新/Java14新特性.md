# Java 14 新特性

Java 14 于 2020 年 3 月发布。此版本将之前的一些预览功能转正，并引入了像 Records 这样的重量级新预览功能。

## 1. Switch 表达式 (标准版)

在经过 Java 12 和 13 两轮预览后，Switch 表达式在 Java 14 中被正式确定下来。

- **最终语法**：使用 `case L -> ...` 的箭头语法，或者在代码块中使用 `yield` 关键字返回值。

```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
    default -> {
        System.out.println("Invalid day");
        yield 0;
    }
};
```

## 2. Records (预览功能)

Java 14 引入了 `record` 关键字作为预览功能，旨在创建不可变的数据聚合类（Data Carrier）。

- **目的**：极大地减少创建 POJO (Plain Old Java Object) 所需的样板代码。
- **自动生成**：编译器会自动为 `record` 生成构造函数、`equals()`, `hashCode()`, `toString()` 以及字段的 `getter` 方法。

**传统方式：**
```java
// 需要手动编写构造函数、getter、equals、hashCode、toString
class Point {
    private final int x;
    private final int y;
    // ... 大量样板代码
}
```

**使用 Record：**
```java
// 一行代码即可
public record Point(int x, int y) {}

// 使用
var point = new Point(1, 2);
System.out.println(point.x()); // 访问器方法，不是 getX()
System.out.println(point);     // Point[x=1, y=2]
```

## 3. 更实用的 NullPointerExceptions

JVM 改进了 `NullPointerException` 的提示信息，可以精确地指出哪个变量是 `null`，极大地提高了调试效率。

**示例代码：**
```java
a.b.c.doSomething();
```

**Java 14 之前的异常信息：**
```
Exception in thread "main" java.lang.NullPointerException
    at MyClass.main(MyClass.java:10)
```

**Java 14 之后的异常信息：**
```
Exception in thread "main" java.lang.NullPointerException: 
    Cannot read field "c" because "a.b" is null
    at MyClass.main(MyClass.java:10)
```

## 4. instanceof 的模式匹配 (预览功能)

简化了 `instanceof` 的使用，允许在检查类型的同时声明一个绑定变量，避免了显式的类型转换。

**传统方式：**
```java
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.toUpperCase());
}
```

**使用模式匹配：**
```java
if (obj instanceof String s) { // s 在 if 块内可用
    System.out.println(s.toUpperCase());
}
```

## 5. 文本块功能增强

文本块（在 Java 13 中预览）增加了两个新的转义序列：

- **`\` (反斜杠)**：用于在行尾取消换行符，可以将多行写成一行。
- **`\s`**: 表示一个单个的空格。

```java
String text = """
              line 1 \
              line 2 \
              line 3
              """;
// 实际内容是 "line 1 line 2 line 3"
```
