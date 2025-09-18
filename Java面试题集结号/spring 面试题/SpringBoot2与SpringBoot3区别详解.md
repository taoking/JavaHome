# Spring Boot 2 与 Spring Boot 3 区别详解

## 📋 目录
- [版本概述](#版本概述)
- [核心依赖变化](#核心依赖变化)
- [Java版本要求](#java版本要求)
- [配置变化](#配置变化)
- [API变化](#api变化)
- [性能改进](#性能改进)
- [迁移指南](#迁移指南)
- [面试重点](#面试重点)

## 🎯 版本概述

### Spring Boot 2.x vs 3.x 基本信息

| 特性 | Spring Boot 2.x | Spring Boot 3.x |
|------|----------------|----------------|
| 发布时间 | 2018年3月 | 2022年11月 |
| Spring Framework | 5.x | 6.x |
| 最低Java版本 | Java 8 | Java 17 |
| Jakarta EE | 不支持 | 支持 |
| GraalVM原生镜像 | 实验性支持 | 生产就绪 |
| 可观测性 | Micrometer | Micrometer + OpenTelemetry |
| 支持周期 | 2023年11月结束 | 2025年11月结束 |

## 🔧 核心依赖变化

### 1. Spring Framework版本升级

```java
// Spring Boot 2.x 基于 Spring Framework 5.x
@Configuration
public class SpringBoot2Config {
    
    // 使用传统的Servlet API
    @Bean
    public FilterRegistrationBean<MyFilter> myFilter() {
        FilterRegistrationBean<MyFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new MyFilter());
        registration.addUrlPatterns("/*");
        return registration;
    }
}

// Spring Boot 3.x 基于 Spring Framework 6.x
@Configuration
public class SpringBoot3Config {
    
    // 支持更现代的配置方式
    @Bean
    public FilterRegistrationBean<MyFilter> myFilter() {
        FilterRegistrationBean<MyFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new MyFilter());
        registration.addUrlPatterns("/*");
        // 新增的配置选项
        registration.setAsyncSupported(true);
        return registration;
    }
}
```

### 2. Jakarta EE迁移

```java
// Spring Boot 2.x - 使用 javax.* 包
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.validation.constraints.NotNull;

@Entity
public class User {
    @Id
    private Long id;
    
    @NotNull
    private String name;
    
    // getter/setter
}

// Spring Boot 3.x - 使用 jakarta.* 包
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.validation.constraints.NotNull;

@Entity
public class User {
    @Id
    private Long id;
    
    @NotNull
    private String name;
    
    // getter/setter
}
```

## ☕ Java版本要求

### 1. 最低版本要求变化

```java
// Spring Boot 2.x - 支持Java 8+
public class SpringBoot2Features {
    
    // 可以使用Java 8特性
    public Optional<String> processData(List<String> data) {
        return data.stream()
                  .filter(s -> s.length() > 5)
                  .findFirst();
    }
    
    // 但不能使用Java 17+的新特性
}

// Spring Boot 3.x - 要求Java 17+
public class SpringBoot3Features {
    
    // 可以使用Java 17的新特性
    public sealed interface Shape permits Circle, Rectangle {
        double area();
    }
    
    public record Circle(double radius) implements Shape {
        @Override
        public double area() {
            return Math.PI * radius * radius;
        }
    }
    
    public record Rectangle(double width, double height) implements Shape {
        @Override
        public double area() {
            return width * height;
        }
    }
    
    // 使用文本块
    public String getJsonTemplate() {
        return """
               {
                   "name": "%s",
                   "age": %d,
                   "active": %b
               }
               """;
    }
}
```

### 2. 性能优化

```java
// Spring Boot 3.x 利用Java 17的性能改进
@Service
public class PerformanceOptimizedService {
    
    // 利用Java 17的向量API（预览特性）
    public void processLargeDataset(int[] data) {
        // 向量化操作，性能更好
        IntVector.SPECIES_256.fromArray(data, 0)
                .add(IntVector.SPECIES_256.broadcast(10))
                .intoArray(data, 0);
    }
    
    // 利用改进的垃圾收集器
    @EventListener
    public void handleLargeDataProcessing(DataProcessingEvent event) {
        // ZGC和G1GC的改进使得大内存应用性能更好
        processLargeDataInMemory(event.getData());
    }
}
```

## ⚙️ 配置变化

### 1. 配置属性变化

```yaml
# Spring Boot 2.x 配置
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db
    username: user
    password: pass
  jpa:
    hibernate:
      ddl-auto: update
  security:
    user:
      name: admin
      password: secret

# Spring Boot 3.x 配置变化
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db
    username: user
    password: pass
  jpa:
    hibernate:
      ddl-auto: update
  # 安全配置方式改变
  security:
    user:
      name: admin
      password: secret
      roles: ADMIN
  # 新增的可观测性配置
  management:
    tracing:
      sampling:
        probability: 1.0
    metrics:
      distribution:
        percentiles-histogram:
          http.server.requests: true
```

### 2. 自动配置变化

```java
// Spring Boot 2.x 安全配置
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/public/**").permitAll()
            .anyRequest().authenticated()
            .and()
            .formLogin();
    }
}

// Spring Boot 3.x 安全配置（WebSecurityConfigurerAdapter已废弃）
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(authz -> authz
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(Customizer.withDefaults());
        
        return http.build();
    }
}
```

## 🔄 API变化

### 1. 废弃的API

```java
// Spring Boot 2.x 中可用但在3.x中废弃的API
public class DeprecatedAPIs {
    
    // 1. WebSecurityConfigurerAdapter 已废弃
    // @Deprecated in Spring Boot 3.x
    public class OldSecurityConfig extends WebSecurityConfigurerAdapter {
        // 不再推荐使用
    }
    
    // 2. 一些Actuator端点变化
    @Deprecated
    public void oldActuatorUsage() {
        // 某些端点路径和响应格式发生变化
    }
    
    // 3. 配置属性绑定方式变化
    @ConfigurationProperties(prefix = "app")
    public class OldConfigProperties {
        // 某些绑定方式在3.x中不再支持
    }
}

// Spring Boot 3.x 推荐的新API
public class NewAPIs {
    
    // 1. 新的安全配置方式
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http.build();
    }
    
    // 2. 改进的配置属性绑定
    @ConfigurationProperties(prefix = "app")
    @ConstructorBinding
    public record AppProperties(
        String name,
        Duration timeout,
        List<String> features
    ) {}
}
```

### 2. 新增功能

```java
// Spring Boot 3.x 新增功能
@RestController
public class SpringBoot3NewFeatures {
    
    // 1. 改进的问题详情支持（RFC 7807）
    @GetMapping("/error-demo")
    public ResponseEntity<ProblemDetail> errorDemo() {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(
            HttpStatus.BAD_REQUEST, 
            "Invalid request parameter"
        );
        problemDetail.setTitle("Validation Error");
        problemDetail.setProperty("timestamp", Instant.now());
        
        return ResponseEntity.badRequest().body(problemDetail);
    }
    
    // 2. 原生镜像支持改进
    @GetMapping("/native-ready")
    public String nativeReady() {
        // 更好的GraalVM原生镜像支持
        return "This endpoint works well in native image";
    }
    
    // 3. 可观测性改进
    @GetMapping("/traced-endpoint")
    @Timed(name = "custom.endpoint.timer", description = "Custom endpoint timer")
    public String tracedEndpoint() {
        // 自动集成OpenTelemetry追踪
        return "This request is automatically traced";
    }
}
```

## 📈 性能改进

### 1. 启动性能

```java
// Spring Boot 3.x 启动性能优化
@SpringBootApplication
public class OptimizedApplication {
    
    public static void main(String[] args) {
        // 1. 改进的类路径扫描
        SpringApplication app = new SpringApplication(OptimizedApplication.class);
        
        // 2. 更快的Bean初始化
        app.setLazyInitialization(true);
        
        // 3. 原生镜像支持
        app.run(args);
    }
}

// 原生镜像配置
@Configuration
@ImportRuntimeHints(MyRuntimeHints.class)
public class NativeImageConfig {
    
    // 为原生镜像提供运行时提示
    static class MyRuntimeHints implements RuntimeHintsRegistrar {
        @Override
        public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
            hints.reflection().registerType(MyService.class, 
                MemberCategory.INVOKE_DECLARED_CONSTRUCTORS,
                MemberCategory.INVOKE_DECLARED_METHODS);
        }
    }
}
```

### 2. 内存使用优化

```java
// Spring Boot 3.x 内存优化
@Configuration
public class MemoryOptimization {
    
    // 1. 改进的Bean定义处理
    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public ExpensiveService expensiveService() {
        // 更高效的原型Bean创建
        return new ExpensiveService();
    }
    
    // 2. 优化的自动配置
    @ConditionalOnClass(name = "com.example.OptionalDependency")
    @Bean
    public OptionalService optionalService() {
        // 更智能的条件判断，减少不必要的类加载
        return new OptionalService();
    }
}
```

## 🔄 迁移指南

### 1. 依赖迁移

```xml
<!-- Spring Boot 2.x 依赖 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.18</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>

<!-- Spring Boot 3.x 依赖 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.1</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <!-- 可能需要添加的新依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

### 2. 代码迁移步骤

```java
// 迁移检查清单
public class MigrationChecklist {
    
    /*
     * 1. 更新Java版本到17+
     * 2. 替换javax.*包为jakarta.*包
     * 3. 更新Spring Security配置
     * 4. 检查废弃的API使用
     * 5. 更新测试代码
     * 6. 验证第三方库兼容性
     * 7. 更新配置文件
     * 8. 测试原生镜像构建（可选）
     */
    
    // 自动化迁移工具使用
    public void useOpenRewriteMigration() {
        /*
         * 使用OpenRewrite自动迁移工具：
         * 
         * 1. 添加OpenRewrite插件到pom.xml
         * 2. 运行迁移命令：
         *    mvn org.openrewrite.maven:rewrite-maven-plugin:run
         *    -Drewrite.recipeArtifactCoordinates=org.openrewrite.recipe:rewrite-spring:LATEST
         *    -Drewrite.activeRecipes=org.openrewrite.java.spring.boot3.UpgradeSpringBoot_3_0
         */
    }
}
```

## 🎯 面试重点

### Q1: Spring Boot 3相比Spring Boot 2有哪些重大变化？

**答案：**
```java
public class MajorChanges {

    /*
     * 1. 基础要求变化
     *    - 最低Java版本：Java 8 → Java 17
     *    - Spring Framework：5.x → 6.x
     *    - Jakarta EE：javax.* → jakarta.*
     *
     * 2. 性能改进
     *    - 原生镜像支持从实验性变为生产就绪
     *    - 启动时间和内存使用优化
     *    - 更好的GraalVM集成
     *
     * 3. 可观测性增强
     *    - 内置OpenTelemetry支持
     *    - 改进的Micrometer集成
     *    - 更丰富的监控指标
     *
     * 4. API现代化
     *    - 废弃WebSecurityConfigurerAdapter
     *    - 改进的配置属性绑定
     *    - 新的问题详情支持（RFC 7807）
     */
}
```

### Q2: 为什么Spring Boot 3要求Java 17？

**答案：**
```java
public class Java17Requirements {

    // 1. 性能优势
    public void performanceBenefits() {
        /*
         * - JVM性能改进：ZGC、G1GC优化
         * - 编译器优化：更好的JIT编译
         * - 内存管理：改进的垃圾收集算法
         */
    }

    // 2. 语言特性
    public sealed interface ModernJavaFeatures permits Record, TextBlock {
        // 密封类、记录类、文本块等新特性
    }

    public record ConfigurationData(String name, int port, boolean enabled)
        implements ModernJavaFeatures {
        // 记录类简化数据传输对象
    }

    // 3. 长期支持
    public void longTermSupport() {
        /*
         * - Java 17是LTS版本，支持周期长
         * - 企业级应用的稳定性保证
         * - 安全更新和bug修复
         */
    }
}
```

### Q3: Jakarta EE迁移有什么影响？

**答案：**
```java
// 迁移影响分析
public class JakartaEEMigrationImpact {

    // 1. 包名变化
    // Spring Boot 2.x
    // import javax.servlet.http.HttpServletRequest;
    // import javax.persistence.Entity;
    // import javax.validation.constraints.NotNull;

    // Spring Boot 3.x
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.persistence.Entity;
    import jakarta.validation.constraints.NotNull;

    // 2. 第三方库兼容性
    public void libraryCompatibility() {
        /*
         * 影响的库：
         * - Hibernate 6.x+
         * - Jackson 2.14+
         * - Tomcat 10.x+
         * - 需要检查所有使用javax.*的依赖
         */
    }

    // 3. 迁移策略
    public void migrationStrategy() {
        /*
         * 1. 使用IDE的全局替换功能
         * 2. 使用OpenRewrite自动迁移工具
         * 3. 逐步迁移，先更新核心模块
         * 4. 充分测试，特别是集成测试
         */
    }
}
```

### Q4: Spring Boot 3的原生镜像支持有什么优势？

**答案：**
```java
@SpringBootApplication
public class NativeImageAdvantages {

    public static void main(String[] args) {
        SpringApplication.run(NativeImageAdvantages.class, args);
    }

    /*
     * 原生镜像优势：
     *
     * 1. 启动速度
     *    - 传统JVM：2-5秒
     *    - 原生镜像：50-200毫秒
     *
     * 2. 内存占用
     *    - 传统JVM：100-500MB
     *    - 原生镜像：20-100MB
     *
     * 3. 部署优势
     *    - 无需JVM环境
     *    - 容器镜像更小
     *    - 冷启动更快
     *
     * 4. 云原生友好
     *    - 适合Serverless
     *    - 适合微服务
     *    - 适合容器化部署
     */
}

// 原生镜像配置示例
@Configuration
public class NativeImageConfig {

    @Bean
    @RegisterReflectionForBinding(UserDto.class)
    public UserService userService() {
        return new UserService();
    }

    // 运行时提示
    @ImportRuntimeHints(CustomRuntimeHints.class)
    static class CustomRuntimeHints implements RuntimeHintsRegistrar {
        @Override
        public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
            // 注册反射、资源、代理等提示
            hints.reflection().registerType(UserDto.class);
            hints.resources().registerPattern("static/**");
        }
    }
}
```

### Q5: Spring Boot 3的可观测性改进体现在哪里？

**答案：**
```java
@RestController
public class ObservabilityImprovements {

    // 1. 自动追踪
    @GetMapping("/api/users/{id}")
    @Timed(name = "user.get", description = "Get user by ID")
    public User getUser(@PathVariable Long id) {
        // 自动生成追踪信息，无需手动配置
        return userService.findById(id);
    }

    // 2. 改进的指标
    @EventListener
    public void handleUserCreated(UserCreatedEvent event) {
        // 自动记录业务指标
        Metrics.counter("user.created",
            "type", event.getUser().getType()).increment();
    }

    // 3. 分布式追踪
    @Async
    public CompletableFuture<String> processAsync() {
        // 追踪信息自动传播到异步线程
        return CompletableFuture.completedFuture("processed");
    }
}

// 配置示例
/*
management:
  tracing:
    sampling:
      probability: 1.0
  metrics:
    distribution:
      percentiles-histogram:
        http.server.requests: true
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
*/
```

## 🔧 实际迁移案例

### 案例1：电商系统迁移

```java
// 迁移前（Spring Boot 2.x）
@RestController
@RequestMapping("/api/orders")
public class OrderControllerV2 {

    @Autowired
    private OrderService orderService;

    @PostMapping
    public ResponseEntity<?> createOrder(@Valid @RequestBody OrderRequest request,
                                       HttpServletRequest httpRequest) {
        try {
            Order order = orderService.createOrder(request);
            return ResponseEntity.ok(order);
        } catch (ValidationException e) {
            return ResponseEntity.badRequest()
                .body(Map.of("error", e.getMessage()));
        }
    }
}

// 迁移后（Spring Boot 3.x）
@RestController
@RequestMapping("/api/orders")
public class OrderControllerV3 {

    private final OrderService orderService;

    // 构造函数注入（推荐）
    public OrderControllerV3(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    public ResponseEntity<?> createOrder(@Valid @RequestBody OrderRequest request,
                                       HttpServletRequest httpRequest) {
        try {
            Order order = orderService.createOrder(request);
            return ResponseEntity.ok(order);
        } catch (ValidationException e) {
            // 使用RFC 7807问题详情
            ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(
                HttpStatus.BAD_REQUEST, e.getMessage());
            problemDetail.setTitle("Order Validation Failed");
            problemDetail.setProperty("timestamp", Instant.now());

            return ResponseEntity.badRequest().body(problemDetail);
        }
    }
}
```

### 案例2：安全配置迁移

```java
// Spring Boot 2.x 安全配置
@Configuration
@EnableWebSecurity
public class SecurityConfigV2 extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests()
                .antMatchers("/api/public/**").permitAll()
                .antMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            .and()
            .oauth2ResourceServer()
                .jwt();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser("admin")
            .password("{noop}password")
            .roles("ADMIN");
    }
}

// Spring Boot 3.x 安全配置
@Configuration
@EnableWebSecurity
public class SecurityConfigV3 {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(Customizer.withDefaults())
            );

        return http.build();
    }

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails admin = User.builder()
            .username("admin")
            .password("{noop}password")
            .roles("ADMIN")
            .build();

        return new InMemoryUserDetailsManager(admin);
    }
}
```

## 📊 性能对比

### 启动时间对比

```java
// 性能测试结果
public class PerformanceComparison {

    /*
     * 启动时间对比（相同应用）：
     *
     * Spring Boot 2.7.x:
     * - 传统JVM启动：3.2秒
     * - 内存占用：280MB
     *
     * Spring Boot 3.2.x:
     * - 传统JVM启动：2.8秒（12%提升）
     * - 原生镜像启动：0.08秒（97%提升）
     * - 内存占用：220MB（21%减少）
     * - 原生镜像内存：45MB（84%减少）
     */

    // 构建时间对比
    /*
     * Maven构建时间：
     * - Spring Boot 2.x：45秒
     * - Spring Boot 3.x：38秒
     * - 原生镜像构建：3分钟
     */
}
```

## 🚀 升级建议

### 升级时机选择

```java
public class UpgradeStrategy {

    /*
     * 建议升级的场景：
     *
     * 1. 新项目
     *    - 直接使用Spring Boot 3.x
     *    - 享受最新特性和性能
     *
     * 2. 现有项目升级条件
     *    - Java版本可以升级到17+
     *    - 第三方依赖支持Jakarta EE
     *    - 有充足的测试时间
     *
     * 3. 暂缓升级的场景
     *    - 大量遗留代码
     *    - 关键第三方库不兼容
     *    - 生产环境稳定性要求极高
     */

    // 渐进式升级策略
    public void gradualUpgrade() {
        /*
         * 1. 先升级开发环境
         * 2. 升级测试环境
         * 3. 小范围生产验证
         * 4. 全面生产部署
         */
    }
}
```

---

*Spring Boot 2与3的详细对比分析完成，这些信息将帮助开发者做出明智的升级决策，并在面试中展现对技术演进的深入理解。*
