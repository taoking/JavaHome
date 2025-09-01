# Java 9 新特性

Java 9 在 2017 年 9 月发布，带来了许多重大变化，其中最引人注目的是 Java 平台模块化系统（JPMS）。

## 1. 模块化系统 (Project Jigsaw)

模块化是 Java 9 的核心功能。它将 JDK 划分为一组模块，并允许开发者将自己的代码也打包成模块。

- **目的**：解决 JAR Hell 问题，提高安全性（隐藏内部实现），提升性能。
- **核心**：`module-info.java` 文件。

一个简单的模块定义如下：

```java
// module-info.java
module com.example.mymodule {
    // 声明依赖于 java.sql 模块
    requires java.sql;
    
    // 声明将 com.example.mypackage 包导出，其他模块可以使用
    exports com.example.mypackage;
}
```

## 2. JShell (交互式命令行)

Java 9 引入了 `jshell`，这是一个交互式的 Read-Eval-Print Loop (REPL) 工具，可以快速测试代码片段，无需创建完整的类。

启动 `jshell`：
```bash
$ jshell
|  欢迎使用 JShell -- 版本 9
|  要大致了解该版本, 请键入: /help intro

jshell> int x = 10;
x ==> 10

jshell> System.out.println("Hello, JShell!");
Hello, JShell!
```

## 3. 接口的私有方法

Java 8 允许在接口中使用 `default` 和 `static` 方法。Java 9 更进一步，允许定义 `private` 方法。这有助于在接口内部重用代码，避免将实现细节暴露给外部。

```java
public interface MyInterface {

    void normalMethod();

    default void method1() {
        init();
        System.out.println("Executing default method 1");
    }

    default void method2() {
        init();
        System.out.println("Executing default method 2");
    }

    // 私有方法，用于在默认方法之间共享代码
    private void init() {
        System.out.println("Initializing...");
    }
}
```

## 4. Stream API 的改进

Stream API 增加了四个有用的新方法：`takeWhile`, `dropWhile`, `ofNullable`, `iterate`。

- **`takeWhile(Predicate)`**: 返回从流的开头开始，满足谓词条件的元素。一旦遇到不满足条件的元素，立即停止。
- **`dropWhile(Predicate)`**: `takeWhile` 的反操作。丢弃从流的开头开始，满足谓词条件的元素，然后返回剩余的流。

```java
Stream.of(1, 2, 3, 4, 5, 1, 2)
      .takeWhile(n -> n < 4)
      .forEach(System.out::print); // 输出: 123

Stream.of(1, 2, 3, 4, 5, 1, 2)
      .dropWhile(n -> n < 4)
      .forEach(System.out::print); // 输出: 4512
```

## 5. Try-With-Resources 语法的改进

Java 9 中，如果资源已经是 `final` 或等效于 `final` 的变量，则可以在 `try-with-resources` 语句中直接使用，而无需在 `try()` 中声明一个新变量。

```java
// Java 8
Reader reader = new StringReader("test");
try (Reader r = reader) {
    // ...
}

// Java 9
Reader reader = new StringReader("test");
try (reader) { // 更简洁
    // ...
}
```

## 6. 其他重要特性

- **`Optional` 类增强**：增加了 `ifPresentOrElse()`, `or()`, `stream()` 等方法。
- **多版本兼容 JAR 包**：允许一个 JAR 文件包含针对不同 Java 版本的类文件。
- **HTTP/2 客户端 API (孵化)**：引入了新的、现代的 HTTP 客户端 API，用于替代旧的 `HttpURLConnection`。
- **`ProcessHandle` API**：提供了管理和查询操作系统进程的能力。
