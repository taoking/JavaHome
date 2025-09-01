# Java 12 新特性

Java 12 于 2019 年 3 月发布。它引入了一些预览功能和 API 增强，其中最受关注的是 Switch 表达式的预览。

## 1. Switch 表达式 (Switch Expressions - 预览功能)

Java 12 首次引入了 Switch 表达式作为预览功能，极大地增强了 `switch` 语句的能力。

- **目的**：简化 `switch` 结构，使其可以作为表达式返回值，避免了繁琐的 `break` 和临时变量。
- **新语法**：使用 `case L -> ...` 标签，无需 `break`，并且可以保证穷尽所有可能的情况。

**传统 `switch` 语句：**
```java
String dayName;
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        dayName = "Work Day";
        break;
    case TUESDAY:
        dayName = "Rest Day";
        break;
    default:
        dayName = "Unknown";
}
```

**使用 Switch 表达式 (Java 12 预览版)：**
```java
String dayName = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> "Work Day";
    case TUESDAY                -> "Rest Day";
    default                     -> "Unknown";
};
```

## 2. Shenandoah: 低暂停时间的垃圾收集器 (实验性)

引入了一个新的垃圾收集器 Shenandoah，其主要目标是通过与正在运行的 Java 线程同时执行更多的 GC 工作来减少 GC 暂停时间。这对于需要低延迟和高响应性的应用程序特别有价值。

## 3. `String` 类的新方法

`String` 类增加了两个有用的方法：`indent()` 和 `transform()`。

- **`indent(int n)`**: 对字符串进行缩进。如果 `n` > 0，则在每行开头添加 `n` 个空格；如果 `n` < 0，则尝试从每行开头删除最多 `|n|` 个空格。
- **`transform(Function)`**: 将一个函数应用于字符串，可以方便地进行链式转换。

```java
// indent
String text = "Hello\nWorld";
System.out.println(text.indent(2));
// 输出:
//   Hello
//   World

// transform
String result = "12345".transform(s -> new StringBuilder(s).reverse().toString());
System.out.println(result); // "54321"
```

## 4. `Files.mismatch()` 方法

`java.nio.file.Files` 中添加了 `mismatch()` 方法，用于比较两个文件的内容是否相同。它会返回第一个不匹配字节的位置，如果文件内容完全相同，则返回 -1L。

```java
Path path1 = Files.createTempFile("file1", ".txt");
Path path2 = Files.createTempFile("file2", ".txt");
Files.writeString(path1, "Java 12");
Files.writeString(path2, "Java 12");

long mismatch = Files.mismatch(path1, path2);
System.out.println(mismatch); // -1 (文件内容相同)
```

## 5. Teeing Collector

Stream API 增加了一个新的收集器 `Collectors.teeing()`。它可以将一个流的元素同时传递给两个下游收集器，然后将它们的结果合并成一个最终结果。

**示例：计算平均值**
```java
// 计算流中数字的总和和计数，然后计算平均值
double average = Stream.of(1, 2, 3, 4, 5)
    .collect(Collectors.teeing(
        Collectors.summingDouble(i -> i),      // 第一个收集器：求和
        Collectors.counting(),                 // 第二个收集器：计数
        (sum, count) -> sum / count            // 合并函数
    ));

System.out.println(average); // 3.0
```
