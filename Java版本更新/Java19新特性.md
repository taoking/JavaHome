# Java 19 新特性

Java 19 于 2022 年 9 月发布。此版本带来了 Java 平台一个里程碑式的预览功能：虚拟线程（Virtual Threads），它有望彻底改变 Java 的并发编程模型。

## 1. 虚拟线程 (Virtual Threads - 预览功能)

虚拟线程是 Project Loom 的核心成果。它是一种轻量级的线程，由 JVM 管理，而不是直接映射到操作系统线程。

- **目的**：极大地提高高吞吐量并发应用程序的性能和可伸缩性。开发者可以用传统的“一个请求一个线程”的同步阻塞式代码，来达到异步非阻塞代码的性能。
- **特点**：创建成本极低，可以轻松创建数百万个虚拟线程。

**创建和使用虚拟线程：**
```java
// 直接创建并启动
Thread.startVirtualThread(() -> {
    System.out.println("Hello from a virtual thread!");
});

// 使用 Executors 创建
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    Future<?> future = executor.submit(() -> System.out.println("Running in a virtual thread"));
    future.get();
}
```

## 2. 结构化并发 (Structured Concurrency - 孵化功能)

这是一个新的并发编程模型，旨在简化多线程错误处理和任务协调。它将并发任务的生命周期视为一个单元，如果一个子任务失败，整个任务组可以被可靠地取消。

```java
// 示例：使用 StructuredTaskScope
// try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
//     Future<String> user = scope.fork(() -> findUser());
//     Future<Integer> order = scope.fork(() -> fetchOrder());
// 
//     scope.join();           // 等待所有子任务完成
//     scope.throwIfFailed();  // 如果有任何子任务失败，则抛出异常
// 
//     // 两个任务都成功后，处理结果
//     String userInfo = user.resultNow();
//     int orderInfo = order.resultNow();
//     // ...
// }
```

## 3. Record 模式 (Record Patterns - 预览功能)

扩展了模式匹配，使其可以用于解构 `record` 类型。这在 `instanceof` 和 `switch` 语句中非常有用。

**使用 Record 模式：**
```java
public record Point(int x, int y) {}

static void printSum(Object obj) {
    // 检查 obj 是否是 Point，如果是，则将其 x 和 y 解构到变量 ix 和 iy
    if (obj instanceof Point(int ix, int iy)) {
        System.out.println(ix + iy);
    }
}

// 也可以与 switch 模式匹配结合使用
// switch (obj) {
//     case Point(int x, int y) -> System.out.printf("Point at %d, %d%n", x, y);
//     ...
// }
```

## 4. Switch 的模式匹配 (第三次预览)

Switch 模式匹配在此版本中进入了第三次预览，继续根据反馈进行改进，例如增加了对 `when` 子句的支持，用于更复杂的条件判断。

## 5. 外部函数和内存 API (第二次预览)

此 API 在 Java 19 中进入了第二次预览阶段（注意：Java 18 是第二次孵化，Java 19 是第二次预览），API 设计更加精炼，朝着最终定稿又迈进了一步。
