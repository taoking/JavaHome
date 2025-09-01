# Java 17 (LTS) 新特性

Java 17 于 2021 年 9 月发布，是继 Java 11 之后的又一个长期支持（LTS）版本。它将一些重要的预览功能（如密封类）转正，并带来了对未来 Java 编程风格有深远影响的新预览功能。

## 1. 密封类 (Sealed Classes - 标准版)

经过两轮预览，密封类在 Java 17 中被正式确定下来。它允许库作者精确控制哪些类可以继承或实现一个 `sealed` 的类或接口，这对于构建安全的、可预测的 API 非常重要。

```java
// Sealed Classes 已成为标准功能
public sealed interface Shape permits Circle, Rectangle {}

public final class Circle implements Shape {}
public final class Rectangle implements Shape {}
```

## 2. Switch 的模式匹配 (预览功能)

这是 Java 17 中最令人兴奋的预览功能。它极大地增强了 `switch` 语句，使其可以对类型进行匹配，并安全地执行类型转换。

- **目的**：简化复杂的、基于类型的条件判断，使代码更具可读性和安全性。

**传统方式：**
```java
static double getArea(Shape shape) {
    if (shape instanceof Circle c) {
        return Math.PI * c.radius() * c.radius();
    } else if (shape instanceof Rectangle r) {
        return r.length() * r.width();
    } else {
        throw new IllegalArgumentException("Unknown shape");
    }
}
```

**使用 Switch 模式匹配：**
```java
static double getArea(Shape shape) {
    return switch (shape) {
        // case 后面直接跟类型和变量名
        case Circle c -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.length() * r.width();
        // default 分支处理其他情况
        default -> throw new IllegalArgumentException("Unknown shape
");
    };
}
```

## 3. 外部函数和内存 API (孵化器)

引入了一个新的 API，旨在替代老旧且脆弱的 JNI (Java Native Interface)。这个 API 提供了一种更安全、更高效的方式来调用 Java 运行时之外的本地代码（如 C 库）和操作本地内存。

## 4. 强封装 JDK 内部 API

从 Java 9 开始，JDK 就试图限制对内部 API 的访问。在 Java 17 中，这一限制变得更加严格。默认情况下，无法再通过反射等“黑客”手段访问 JDK 的内部模块，除非通过特定的命令行参数显式授权。这有助于提高平台的安全性和可维护性。

## 5. 移除 RMI 激活机制

远程方法调用（RMI）的激活机制（Activation）是一个过时且很少使用的功能，已在此版本中被正式移除。

## 6. 废弃 Applet API

古老的 Applet API 在此版本中被正式标记为“废弃，并准备在未来移除”。这标志着 Applet 技术时代的终结。
