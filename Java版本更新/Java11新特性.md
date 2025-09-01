# Java 11 (LTS) 新特性

Java 11 于 2018 年 9 月发布，是一个长期支持（LTS）版本。它带来了许多实用的 API 增强和一些重要的平台更新。

## 1. 标准化的 HTTP Client API

在 Java 9 和 10 中作为孵化功能的 HTTP Client API，在 Java 11 中被正式标准化。它位于 `java.net.http` 包中，提供了对 HTTP/1.1 和 HTTP/2 的支持，并完全取代了旧的 `HttpURLConnection`。

- **核心类**：`HttpClient`, `HttpRequest`, `HttpResponse`。
- **特点**：支持同步和异步操作，使用 `CompletableFuture` 处理异步请求。

**示例：发送一个 GET 请求**
```java
var client = HttpClient.newHttpClient();
var request = HttpRequest.newBuilder()
        .uri(URI.create("https://api.github.com/"))
        .build();

// 异步方式
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
      .thenApply(HttpResponse::body)
      .thenAccept(System.out::println)
      .join();

// 同步方式
// HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
// System.out.println(response.body());
```

## 2. String 类的增强

`String` 类增加了几个非常方便的新方法：

- **`isBlank()`**: 判断字符串是否为空白（只包含空格、制表符等）。
- **`lines()`**: 将字符串按行分隔符转换为一个 `Stream<String>`。
- **`strip()`**, **`stripLeading()`**, **`stripTrailing()`**: 去除字符串首尾或单边的空白字符，功能比 `trim()` 更强大，能识别 Unicode 空白字符。
- **`repeat(int)`**: 将字符串重复指定的次数。

```java
// isBlank
System.out.println("  \t  ".isBlank()); // true

// lines
"Hello\nWorld".lines().forEach(System.out::println);

// strip
String s = "\u2005 Hello \u2005"; // \u2005 是一个 Unicode 空白符
System.out.println("'" + s.trim() + "'");      // '  Hello  '
System.out.println("'" + s.strip() + "'");     // 'Hello'

// repeat
System.out.println("Java ".repeat(3)); // "Java Java Java "
```

## 3. 直接运行单个 Java 源文件

从 Java 11 开始，你可以直接使用 `java` 命令来运行一个单个的 `.java` 源文件，无需先手动使用 `javac` 编译。

```bash
# 创建一个 HelloWorld.java 文件
# public class HelloWorld {
#     public static void main(String[] args) {
#         System.out.println("Hello, Java 11!");
#     }
# }

# 直接运行
$ java HelloWorld.java
Hello, Java 11!
```

## 4. Lambda 参数的局部变量类型推断

`var` 关键字现在可以用于 Lambda 表达式的参数声明中。这在需要给参数添加注解时非常有用。

```java
// Java 10 中不允许
// (var x, var y) -> x.process(y)

// Java 11 中允许
Predicate<String> predicate = (@Nonnull var s) -> s.length() > 0;
```

## 5. 文件读写 API 增强

`java.nio.file.Files` 类增加了 `writeString()` 和 `readString()` 方法，可以更方便地读写文件内容。

```java
Path path = Files.writeString(Files.createTempFile("test", ".txt"), "Hello, world!");
String content = Files.readString(path);
System.out.println(content); // Hello, world!
```

## 6. ZGC (Z Garbage Collector)

引入了 ZGC，这是一个可伸缩的、低延迟的垃圾收集器，作为一项实验性功能。它的设计目标是将 GC 停顿时间控制在 10ms 以内，即使是对于非常大的堆（TB 级别）。

## 7. 移除 Java EE 和 CORBA 模块

Java 11 从 JDK 中移除了过去被废弃的 Java EE 和 CORBA 模块（例如 `java.xml.ws`, `java.corba`）。如果你的应用需要这些模块，必须从外部库（如 Maven Central）中添加依赖。
