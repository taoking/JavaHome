# Java 16 新特性

Java 16 于 2021 年 3 月发布。此版本将 Java 14 和 15 中广受欢迎的几个预览功能（如 Records 和模式匹配）正式确定下来。

## 1. Records (标准版)

经过两轮预览，Records 在 Java 16 中被正式确定下来。现在，你可以放心地在生产代码中使用 `record` 来定义不可变的数据载体类，从而极大地减少样板代码。

```java
// Records 已成为标准功能
public record Point(int x, int y) {}
```

## 2. instanceof 的模式匹配 (标准版)

`instanceof` 的模式匹配也在此版本中被正式确定下来。这使得类型检查和转换的代码更加简洁和安全。

```java
// 模式匹配已成为标准功能
Object obj = "Hello, Java 16";
if (obj instanceof String s) {
    // s可以直接使用，无需强制转换
    System.out.println(s.length());
}
```

## 3. 密封类 (第二次预览)

密封类（Sealed Classes）在 Java 16 中进入了第二次预览。根据社区的反馈进行了一些改进，例如放宽了对 `sealed` 类所在包的限制。

## 4. Vector API (孵化器)

引入了一个新的 Vector API 作为孵化功能。这个 API 旨在提供一种表达向量计算的方式，这些计算在运行时可以可靠地编译为 CPU 架构上的最佳向量指令（如 SIMD 指令），从而获得优于等效标量计算的性能。

```java
// 示例：使用 Vector API 对两个数组进行求和
// int[] a = { ... };
// int[] b = { ... };
// int[] c = new int[a.length];
// var SPECIES = IntVector.SPECIES_PREFERRED;
// 
// for (int i = 0; i < a.length; i += SPECIES.length()) {
//     var mask = SPECIES.indexInRange(i, a.length);
//     var va = IntVector.fromArray(SPECIES, a, i, mask);
//     var vb = IntVector.fromArray(SPECIES, b, i, mask);
//     var vc = va.add(vb);
//     vc.intoArray(c, i, mask);
// }
```

## 5. 默认启用 C++14 语言特性

在 JVM 的 C++ 源代码中，现在可以使用 C++14 版本的语言特性。这对 Java 开发者是透明的，但有助于 JVM 开发人员利用更现代的 C++ 特性来改进和维护 JVM。

## 6. 将 JDK 移植到 Alpine Linux

JDK 被成功移植到使用 musl 作为主要 C 库的 Alpine Linux 以及其他 Linux 发行版上。这对于构建轻量级容器镜像非常重要，因为 Alpine Linux 是容器化环境中的热门选择。
