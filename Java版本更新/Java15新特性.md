# Java 15 新特性

Java 15 于 2020 年 9 月发布。它带来了新的预览功能，如密封类，并让一些重要的功能（如 ZGC）进入了生产就绪阶段。

## 1. 密封类 (Sealed Classes - 预览功能)

密封类和接口用于限制哪些其他类或接口可以扩展或实现它们。

- **目的**：为类库作者提供更精细的控制，允许对类层次结构进行建模，这在模式匹配中特别有用。
- **关键字**：`sealed`, `permits`, `non-sealed`。

**示例：**
```java
// Shape 接口只允许 Circle 和 Square 实现
public sealed interface Shape permits Circle, Square {
    double area();
}

// final class, 终止继承
public final class Circle implements Shape {
    public final double radius;
    public Circle(double radius) { this.radius = radius; }
    public double area() { return Math.PI * radius * radius; }
}

// non-sealed class, 允许任何类继承它
public non-sealed class Square implements Shape {
    public final double side;
    public Square(double side) { this.side = side; }
    public double area() { return side * side; }
}

// 由于 Square 是 non-sealed, 任何类都可以继承它
public class SpecialSquare extends Square {
    public SpecialSquare(double side) { super(side); }
}
```

## 2. Records (第二次预览) & instanceof 模式匹配 (第二次预览)

在 Java 14 中引入的 Records 和 `instanceof` 模式匹配在 Java 15 中进入了第二次预览。它们的功能基本保持不变，主要是根据社区反馈进行了一些微调，为最终定稿做准备。

## 3. 文本块 (标准版)

经过两轮预览，文本块在 Java 15 中被正式确定下来，成为标准功能。现在可以放心在生产代码中使用多行字符串了。

```java
String json = """
              {
                  "name": "John Doe",
                  "age": 30
              }
              """;
```

## 4. ZGC 和 Shenandoah 垃圾收集器 (生产就绪)

在之前版本中作为实验性或预览功能的 ZGC 和 Shenandoah 低延迟垃圾收集器，在 Java 15 中被标记为生产就绪（Production Ready）。这意味着它们已经足够稳定，可以在生产环境中使用。

## 5. 隐藏类 (Hidden Classes)

这是一个主要面向框架和工具开发者的功能。隐藏类是一种不能被其他类的字节码直接使用的类。它们只能通过反射被发现和使用。

- **目的**：允许框架在运行时生成和加载类，而不会污染应用程序的类命名空间，并且可以独立卸载，从而减少内存占用。

## 6. 移除 Nashorn JavaScript 引擎

在 Java 11 中被标记为废弃的 Nashorn JavaScript 引擎及其相关 API，在 Java 15 中被正式移除。如果需要，可以考虑使用 GraalVM 等替代方案。
