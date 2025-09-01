# Java 13 新特性

Java 13 于 2019 年 9 月发布。此版本对 Java 12 引入的一些预览功能进行了改进，并重新实现了一些底层 API。

## 1. 文本块 (Text Blocks - 预览功能)

Java 13 引入了文本块作为一项预览功能，旨在简化多行字符串的编写。

- **目的**：避免在字符串中大量使用转义字符（如 `\n`），使 SQL、JSON、HTML 等嵌入代码更易于读写。
- **语法**：使用三个双引号 `"""` 作为开始和结束分隔符。

**传统方式：**
```java
String html = "<html>\n" +
              "    <body>\n" +
              "        <p>Hello, World</p>\n" +
              "    </body>\n" +
              "</html>\n";
```

**使用文本块：**
```java
String html = """
              <html>
                  <body>
                      <p>Hello, World</p>
                  </body>
              </html>
              """;
```

## 2. Switch 表达式增强 (预览功能)

Java 12 的 Switch 表达式在 Java 13 中得到了改进。最主要的变化是引入了 `yield` 关键字，用于从 `switch` 表达式中返回一个值，以替代之前的 `break` 用法。

```java
String text = switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        yield "Work Day";
    case TUESDAY:
        yield "Rest Day";
    default:
        yield "Unknown";
};
```

**注意**：在 Java 14 中，`yield` 被最终确定下来，而 `break` 的返回值方式被移除。

## 3. 重新实现旧版 Socket API

`java.net.Socket` 和 `java.net.ServerSocket` 的底层实现被替换为一个更现代化、更易于维护和调试的实现。这个变化对开发者是透明的，旧的代码无需修改即可运行。

- **目的**：消除历史遗留的技术债，为未来的网络编程（如 Project Loom 的虚拟线程）提供更好的基础。

## 4. ZGC: 取消提交未使用的内存

ZGC 垃圾收集器得到了增强，现在可以将未使用的堆内存返还给操作系统。这对于需要关心内存占用的云原生和容器化环境非常有用。

## 5. 动态 CDS 归档 (Dynamic CDS Archives)

在 Java 10 引入的应用类数据共享（AppCDS）基础上进行了扩展。现在，可以在应用程序执行结束时动态生成共享归档，简化了 AppCDS 的使用流程。
