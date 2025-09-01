# Java 21 (LTS) 新特性

Java 21 于 2023 年 9 月发布，是继 Java 17 之后的最新一个长期支持（LTS）版本。它正式确定了虚拟线程、记录模式、Switch 模式匹配等一系列里程碑式的功能，标志着现代 Java 开发进入了一个新阶段。

## 1. 虚拟线程 (Virtual Threads - 标准版)

经过两轮预览，虚拟线程在 Java 21 中被正式确定下来。开发者现在可以在生产环境中大规模使用轻量级线程，用同步的方式写出高并发、高吞吐量的应用程序，而无需复杂的异步编程。

```java
// 虚拟线程已成为标准功能
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10_000; i++) {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));
            return i;
        });
    }
} // executor.close() 会隐式地等待所有任务完成
```

## 2. 记录模式 (Record Patterns - 标准版)

记录模式也在此版本中被正式确定下来。它允许在 `instanceof` 和 `switch` 中对 `record` 进行解构，使代码更加简洁和富有表现力。

```java
public record Point(int x, int y) {}

// 记录模式已成为标准功能
static void printSum(Object obj) {
    if (obj instanceof Point(int x, int y)) {
        System.out.println(x + y);
    }
}
```

## 3. Switch 的模式匹配 (标准版)

经过多轮预览，Switch 的模式匹配终于在 Java 21 中转正。它极大地增强了 `switch` 的能力，使其可以根据类型和 `record` 结构进行分支判断，并支持 `when` 子句进行更精细的控制。

```java
// Switch 模式匹配已成为标准功能
static String formatter(Object o) {
    return switch (o) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        case Point(int x, int y) -> String.format("Point at %d, %d", x, y);
        default        -> o.toString();
    };
}
```

## 4. 序列化集合 (Sequenced Collections)

引入了一套新的集合接口，用于表示具有确定出现顺序的集合。`List` 和 `Deque` 等都实现了新的 `SequencedCollection` 接口。

- **核心接口**：`SequencedCollection`, `SequencedSet`, `SequencedMap`。
- **新方法**：提供了如 `getFirst()`, `getLast()`, `addFirst()`, `addLast()`, `reversed()` 等统一的 API。

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");

// 新 API
System.out.println(list.getFirst()); // A
System.out.println(list.getLast());  // B

// 获取一个反向视图
List<String> reversedList = list.reversed();
System.out.println(reversedList); // [B, A]
```

## 5. 字符串模板 (String Templates - 预览功能)

这是一个新的预览功能，旨在简化包含运行时表达式的字符串的编写，比传统的字符串拼接或 `String.format()` 更安全、更强大。

```java
// 使用 STR 模板处理器
String name = "Joan";
String info = STR."My name is \{name}";
System.out.println(info); // My name is Joan
```

## 6. 未命名模式和变量 (Unnamed Patterns and Variables - 预览功能)

允许使用下划线 `_` 来表示未使用的模式变量或局部变量，以减少代码冗余。

```java
// 忽略 record 模式中的某个组件
// if (obj instanceof Point(int x, int _)) { ... }

// 忽略 switch case 中的变量
// case Point(int x, int _) -> ...

// 忽略 try-with-resources 中的资源变量
// try (var _ = ScopedValue.where(...)) { ... }
```

## 7. 结构化并发 (Structured Concurrency - 预览功能) & 作用域值 (Scoped Values - 预览功能)

这两个在之前版本中孵化的功能，在 Java 21 中都进入了预览阶段，API 更加成熟，离最终定稿更近一步。
