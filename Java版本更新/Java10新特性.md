# Java 10 新特性

Java 10 于 2018 年 3 月发布，是 Java 采用新的、更快的半年发布周期后的第一个版本。它带来了一些实用的新功能，其中最著名的是局部变量类型推断。

## 1. 局部变量类型推断 (Local-Variable Type Inference)

这是 Java 10 最核心的特性。通过使用 `var` 关键字，可以让编译器推断出局部变量的类型，从而简化代码。

- **目的**：减少样板代码，提高代码的可读性。
- **注意**：`var` 只能用于局部变量（方法内、循环中等），不能用于成员变量、方法参数或返回类型。

**使用前：**
```java
String message = "Hello, Java 10!";
ArrayList<String> list = new ArrayList<String>();
```

**使用后：**
```java
var message = "Hello, Java 10!"; // 推断为 String
var list = new ArrayList<String>(); // 推断为 ArrayList<String>

for (var item : list) {
    System.out.println(item); // item 推断为 String
}
```

## 2. 不可变集合的增强

`List`, `Set`, `Map` 接口新增了 `copyOf`静态方法，用于创建一个不可修改的集合副本。

```java
var originalList = new ArrayList<String>();
originalList.add("a");
originalList.add("b");

// 创建一个不可修改的副本
var copy = List.copyOf(originalList);

// 尝试修改副本会导致 UnsupportedOperationException
// copy.add("c"); 
```

## 3. `Optional` 的 `orElseThrow()` 方法

`Optional` 类新增了 `orElseThrow()` 方法。当 `Optional` 为空时，它会抛出 `NoSuchElementException`。这与 `get()` 方法的行为完全相同，但方法名更能体现其作用。

```java
// 如果 Optional 为空，则抛出 NoSuchElementException
var value = optional.orElseThrow();
```

## 4. 垃圾收集器 (GC) 改进

- **G1 垃圾收集器的并行 Full GC**：在之前的版本中，G1 的 Full GC 是单线程的，可能导致较长的停顿。Java 10 将其改为了多线程，从而减少了最坏情况下的停顿时间。
- **垃圾收集器接口 (GC Interface)**：引入了一个干净的 GC 接口，使得在 JVM 中集成新的垃圾收集器变得更加容易。

## 5. 应用类数据共享 (Application Class-Data Sharing)

扩展了现有的类数据共享（CDS）功能，允许将应用程序的类也包含在共享归档中。这可以减少多个 JVM 实例之间的内存占用，并缩短启动时间。
