# Java 20 新特性

Java 20 于 2023 年 3 月发布。此版本的主要工作是继续推进和完善在 Java 19 中引入的多个重要预览和孵化功能。

## 1. 虚拟线程 (Virtual Threads - 第二次预览)

虚拟线程在此版本中进入了第二次预览。API 方面没有重大变化，主要是根据社区反馈进行了一些小的改进和错误修复，使其更加稳定和可靠。

## 2. 结构化并发 (Structured Concurrency - 第二次孵化)

结构化并发在此版本中进入了第二次孵化。一个重要的变化是 `StructuredTaskScope` 进行了更新，可以支持在任务作用域内跨线程继承作用域值（Scoped Values），这对于在并发任务之间传递上下文信息非常有用。

## 3. 作用域值 (Scoped Values - 孵化功能)

这是一个新的孵化功能，旨在提供一种在线程内部以及线程之间共享不可变数据的现代化方法，尤其是在使用虚拟线程时。

- **目的**：替代 `ThreadLocal`，特别是在虚拟线程场景下。`ThreadLocal` 在虚拟线程环境下存在一些问题（如内存泄漏风险、数据继承成本高）。
- **特点**：数据是不可变的，并且只在特定的代码作用域内有效，可以安全地跨线程传递。

```java
// public final static ScopedValue<User> LOGGED_IN_USER = ScopedValue.newInstance();
// 
// // 在一个作用域内设置值
// ScopedValue.where(LOGGED_IN_USER, currentUser)
//            .run(() -> {
//                // 在这里，LOGGED_IN_USER.get() 会返回 currentUser
//                processRequest(); 
//            });
```

## 4. Record 模式 (Record Patterns - 第二次预览)

Record 模式也进入了第二次预览。此版本的一个主要改进是允许在 Record 模式中进行类型参数的推断，并支持与 `for` 循环等语句结合使用。

## 5. Switch 的模式匹配 (第四次预览)

Switch 的模式匹配在此版本中进入了第四次预览，API 已经非常稳定，主要进行了一些语法上的微调和对泛型模式匹配的增强，为在下一个 LTS 版本中最终定稿做好了准备。

## 6. 外部函数和内存 API (第二次预览)

这个旨在替代 JNI 的 API 也进入了第二次预览，API 更加精炼，并统一了 `MemorySegment` 和 `MemoryAddress` 的接口，使其更易于使用。
