# Java 18 新特性

Java 18 于 2022 年 3 月发布。此版本带来了一个非常实用的开箱即用工具，并继续推进了多个重要的孵化和预览功能。

## 1. 默认编码为 UTF-8

这是一个重要的全局性变更。从 Java 18 开始，所有标准的 Java API 都将默认使用 UTF-8 编码，而不再依赖于操作系统、用户语言环境等外部因素。这解决了长期以来困扰开发者的编码不一致问题，大大提高了跨平台开发的可靠性。

## 2. 简单的 Web 服务器

JDK 内置了一个开箱即用的、最小化的静态 HTTP 文件服务器。它不是为了在生产环境替代 Tomcat、Jetty 等服务器，而是为了在开发、测试和教学场景中快速启动一个服务来共享文件。

**启动一个服务器：**
```bash
# 在你的项目目录下运行
# 默认监听 8000 端口
$ jwebserver

# 也可以指定端口和目录
$ jwebserver -p 8080 -d /path/to/files
```

## 3. Java API 文档中的代码片段

这是一个针对文档工具 (Javadoc) 的改进。现在可以使用 `@snippet` 标签在 API 文档中包含更丰富、更健壮的示例代码。这使得示例代码更容易被验证、高亮和维护。

## 4. Switch 的模式匹配 (第二次预览)

Java 17 引入的 Switch 模式匹配在此版本中进入了第二次预览。主要根据社区反馈进行了一些改进，例如在处理 `null` 情况时更加完善。

```java
static String formatter(Object o) {
    return switch (o) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        // ...
        case null      -> "null"; // 可以直接处理 null case
        default        -> o.toString();
    };
}
```

## 5. 外部函数和内存 API (第二次孵化)

这个旨在替代 JNI 的 API 在此版本中进入了第二次孵化，API 设计上有了显著的改进，使其更加易用和强大。

## 6. Vector API (第三次孵化)

用于高性能向量计算的 Vector API 也进入了第三次孵化阶段，继续根据反馈进行完善。
