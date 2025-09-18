# Spring Boot 2 完整学习指南 - 生命周期、加载方式、运行流程

## 📋 目录
- [Spring Boot 2 概述](#spring-boot-2-概述)
- [Spring Boot 生命周期](#spring-boot-生命周期)
- [自动配置原理](#自动配置原理)
- [启动流程详解](#启动流程详解)
- [Bean加载机制](#bean加载机制)
- [配置加载顺序](#配置加载顺序)
- [核心注解解析](#核心注解解析)
- [面试常见问题](#面试常见问题)

## 🎯 Spring Boot 2 概述

### 什么是Spring Boot？

Spring Boot是Spring团队提供的全新框架，设计目的是简化Spring应用的初始搭建以及开发过程。

**核心特性：**
- **约定优于配置**：提供默认配置，减少样板代码
- **自动配置**：根据classpath自动配置Spring应用
- **嵌入式服务器**：内置Tomcat、Jetty等服务器
- **生产就绪**：提供监控、健康检查等功能

### Spring Boot 2.x 主要改进

```java
// Spring Boot 2.x 主要变化
public class SpringBoot2Changes {
    
    // 1. 基于Spring Framework 5.x
    // 2. 最低JDK版本要求：Java 8
    // 3. 支持响应式编程（WebFlux）
    // 4. 配置属性绑定改进
    // 5. Actuator端点重构
    // 6. 安全配置简化
}
```

## 🔄 Spring Boot 生命周期

### 1. 应用生命周期概览

```java
@SpringBootApplication
public class Application {
    
    public static void main(String[] args) {
        // 1. 创建SpringApplication实例
        SpringApplication app = new SpringApplication(Application.class);
        
        // 2. 运行应用
        ConfigurableApplicationContext context = app.run(args);
        
        // 3. 应用运行中...
        
        // 4. 关闭应用
        context.close();
    }
}
```

### 2. 详细生命周期阶段

```java
@Component
public class ApplicationLifecycleListener {
    
    // 阶段1：应用启动前
    @EventListener
    public void handleApplicationStartingEvent(ApplicationStartingEvent event) {
        System.out.println("1. 应用开始启动");
    }
    
    // 阶段2：环境准备完成
    @EventListener
    public void handleApplicationEnvironmentPreparedEvent(ApplicationEnvironmentPreparedEvent event) {
        System.out.println("2. 环境准备完成");
    }
    
    // 阶段3：上下文准备完成
    @EventListener
    public void handleApplicationContextInitializedEvent(ApplicationContextInitializedEvent event) {
        System.out.println("3. 上下文初始化完成");
    }
    
    // 阶段4：上下文准备完成
    @EventListener
    public void handleApplicationPreparedEvent(ApplicationPreparedEvent event) {
        System.out.println("4. 上下文准备完成");
    }
    
    // 阶段5：应用启动完成
    @EventListener
    public void handleApplicationStartedEvent(ApplicationStartedEvent event) {
        System.out.println("5. 应用启动完成");
    }
    
    // 阶段6：应用就绪
    @EventListener
    public void handleApplicationReadyEvent(ApplicationReadyEvent event) {
        System.out.println("6. 应用就绪，可以接收请求");
    }
    
    // 阶段7：应用启动失败
    @EventListener
    public void handleApplicationFailedEvent(ApplicationFailedEvent event) {
        System.out.println("7. 应用启动失败");
    }
}
```

### 3. Bean生命周期

```java
@Component
public class BeanLifecycleDemo implements InitializingBean, DisposableBean {
    
    private String name;
    
    // 1. 构造函数
    public BeanLifecycleDemo() {
        System.out.println("1. 构造函数执行");
    }
    
    // 2. 属性注入
    @Autowired
    public void setName(@Value("${app.name:demo}") String name) {
        this.name = name;
        System.out.println("2. 属性注入完成: " + name);
    }
    
    // 3. BeanNameAware
    @Override
    public void setBeanName(String name) {
        System.out.println("3. BeanNameAware: " + name);
    }
    
    // 4. BeanFactoryAware
    @Override
    public void setBeanFactory(BeanFactory beanFactory) {
        System.out.println("4. BeanFactoryAware");
    }
    
    // 5. ApplicationContextAware
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        System.out.println("5. ApplicationContextAware");
    }
    
    // 6. @PostConstruct
    @PostConstruct
    public void postConstruct() {
        System.out.println("6. @PostConstruct执行");
    }
    
    // 7. InitializingBean
    @Override
    public void afterPropertiesSet() {
        System.out.println("7. InitializingBean.afterPropertiesSet()");
    }
    
    // 8. @PreDestroy
    @PreDestroy
    public void preDestroy() {
        System.out.println("8. @PreDestroy执行");
    }
    
    // 9. DisposableBean
    @Override
    public void destroy() {
        System.out.println("9. DisposableBean.destroy()");
    }
}
```

## ⚙️ 自动配置原理

### 1. @SpringBootApplication注解解析

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration  // 等同于@Configuration
@EnableAutoConfiguration  // 启用自动配置
@ComponentScan(excludeFilters = { // 组件扫描
    @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
    @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) 
})
public @interface SpringBootApplication {
    // ...
}
```

### 2. @EnableAutoConfiguration工作原理

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)  // 关键：导入自动配置选择器
public @interface EnableAutoConfiguration {
    // ...
}

// 自动配置导入选择器
public class AutoConfigurationImportSelector implements DeferredImportSelector {
    
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        // 1. 检查自动配置是否启用
        if (!isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        }
        
        // 2. 获取自动配置条目
        AutoConfigurationEntry autoConfigurationEntry = 
            getAutoConfigurationEntry(annotationMetadata);
        
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
    
    protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        // 3. 加载spring.factories中的配置类
        List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
        
        // 4. 去重
        configurations = removeDuplicates(configurations);
        
        // 5. 排除不需要的配置
        Set<String> exclusions = getExclusions(annotationMetadata, attributes);
        configurations.removeAll(exclusions);
        
        // 6. 过滤（根据条件注解）
        configurations = getConfigurationClassFilter().filter(configurations);
        
        return new AutoConfigurationEntry(configurations, exclusions);
    }
}
```

### 3. 条件注解机制

```java
// 示例：Redis自动配置
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(RedisOperations.class)  // 类路径存在RedisOperations
@EnableConfigurationProperties(RedisProperties.class)  // 启用配置属性
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")  // 不存在redisTemplate bean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)  // 存在唯一的连接工厂
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
    
    @Bean
    @ConditionalOnMissingBean  // 不存在StringRedisTemplate bean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        return new StringRedisTemplate(redisConnectionFactory);
    }
}
```

## 🚀 启动流程详解

### 1. SpringApplication.run()方法解析

```java
public class SpringApplication {
    
    public ConfigurableApplicationContext run(String... args) {
        // 1. 创建启动时间监控
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        
        // 2. 声明应用上下文和异常报告器
        DefaultBootstrapContext bootstrapContext = createBootstrapContext();
        ConfigurableApplicationContext context = null;
        Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
        
        // 3. 设置系统属性
        configureHeadlessProperty();
        
        // 4. 获取运行监听器
        SpringApplicationRunListeners listeners = getRunListeners(args);
        listeners.starting(bootstrapContext, this.mainApplicationClass);
        
        try {
            // 5. 准备应用参数
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
            
            // 6. 准备环境
            ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
            configureIgnoreBeanInfo(environment);
            
            // 7. 打印Banner
            Banner printedBanner = printBanner(environment);
            
            // 8. 创建应用上下文
            context = createApplicationContext();
            context.setApplicationStartup(this.applicationStartup);
            
            // 9. 准备上下文
            prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
            
            // 10. 刷新上下文（核心）
            refreshContext(context);
            
            // 11. 刷新后处理
            afterRefresh(context, applicationArguments);
            
            // 12. 停止计时
            stopWatch.stop();
            
            // 13. 发布启动完成事件
            listeners.started(context);
            
            // 14. 调用ApplicationRunner和CommandLineRunner
            callRunners(context, applicationArguments);
            
        } catch (Throwable ex) {
            handleRunFailure(context, ex, listeners);
            throw new IllegalStateException(ex);
        }
        
        try {
            // 15. 发布就绪事件
            listeners.running(context);
        } catch (Throwable ex) {
            handleRunFailure(context, ex, null);
            throw new IllegalStateException(ex);
        }
        
        return context;
    }
}
```

### 2. 上下文刷新过程

```java
// AbstractApplicationContext.refresh()方法
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        StartupStep contextRefresh = this.applicationStartup.start("spring.context.refresh");
        
        // 1. 准备刷新
        prepareRefresh();
        
        // 2. 获取BeanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        
        // 3. 准备BeanFactory
        prepareBeanFactory(beanFactory);
        
        try {
            // 4. 后置处理BeanFactory
            postProcessBeanFactory(beanFactory);
            
            StartupStep beanPostProcess = this.applicationStartup.start("spring.context.beans.post-process");
            
            // 5. 调用BeanFactoryPostProcessor
            invokeBeanFactoryPostProcessors(beanFactory);
            
            // 6. 注册BeanPostProcessor
            registerBeanPostProcessors(beanFactory);
            
            beanPostProcess.end();
            
            // 7. 初始化MessageSource
            initMessageSource();
            
            // 8. 初始化事件多播器
            initApplicationEventMulticaster();
            
            // 9. 刷新特定上下文（模板方法）
            onRefresh();
            
            // 10. 注册监听器
            registerListeners();
            
            // 11. 完成BeanFactory初始化
            finishBeanFactoryInitialization(beanFactory);
            
            // 12. 完成刷新
            finishRefresh();
        } catch (BeansException ex) {
            // 销毁已创建的Bean
            destroyBeans();
            cancelRefresh(ex);
            throw ex;
        } finally {
            resetCommonCaches();
            contextRefresh.end();
        }
    }
}
```

## 📦 Bean加载机制

### 1. Bean定义注册

```java
@Configuration
public class BeanLoadingDemo {
    
    // 方式1：@Bean注解
    @Bean
    @ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
    public MyService myService() {
        return new MyServiceImpl();
    }
    
    // 方式2：@Component扫描
    @Component
    public class ComponentService {
        // ...
    }
    
    // 方式3：@Import导入
    @Import({ImportedConfig.class, ImportSelector.class})
    public class ImportDemo {
        // ...
    }
    
    // 方式4：实现ImportSelector
    public class CustomImportSelector implements ImportSelector {
        @Override
        public String[] selectImports(AnnotationMetadata importingClassMetadata) {
            return new String[]{"com.example.ImportedService"};
        }
    }
}
```

### 2. Bean实例化过程

```java
// Bean实例化的详细过程
public class BeanInstantiationProcess {
    
    // 1. 实例化前处理
    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) {
        System.out.println("实例化前处理: " + beanName);
        return null; // 返回null继续正常实例化
    }
    
    // 2. 实例化后处理
    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) {
        System.out.println("实例化后处理: " + beanName);
        return true; // 返回true继续属性填充
    }
    
    // 3. 属性填充前处理
    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
        System.out.println("属性填充处理: " + beanName);
        return pvs;
    }
    
    // 4. 初始化前处理
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        System.out.println("初始化前处理: " + beanName);
        return bean;
    }
    
    // 5. 初始化后处理
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("初始化后处理: " + beanName);
        return bean;
    }
}
```

## 📋 配置加载顺序

### 1. 配置文件加载优先级

Spring Boot按以下顺序加载配置（优先级从高到低）：

```java
// 配置加载顺序示例
public class ConfigurationLoadingOrder {

    /*
     * 1. 命令行参数
     * java -jar app.jar --server.port=8081
     */

    /*
     * 2. SPRING_APPLICATION_JSON中的属性
     * SPRING_APPLICATION_JSON='{"server.port":8082}' java -jar app.jar
     */

    /*
     * 3. ServletConfig初始化参数
     * web.xml中的<init-param>
     */

    /*
     * 4. ServletContext初始化参数
     * web.xml中的<context-param>
     */

    /*
     * 5. JNDI属性
     * java:comp/env
     */

    /*
     * 6. Java系统属性
     * System.getProperties()
     */

    /*
     * 7. 操作系统环境变量
     * System.getenv()
     */

    /*
     * 8. RandomValuePropertySource
     * random.*属性
     */

    /*
     * 9. jar包外的application-{profile}.properties/yml
     * 10. jar包内的application-{profile}.properties/yml
     * 11. jar包外的application.properties/yml
     * 12. jar包内的application.properties/yml
     */

    /*
     * 13. @PropertySource注解指定的配置文件
     */

    /*
     * 14. 默认属性
     * SpringApplication.setDefaultProperties()
     */
}
```

### 2. 配置文件位置优先级

```java
// 配置文件搜索位置（优先级从高到低）
public class ConfigurationLocations {

    /*
     * 1. file:./config/          (项目根目录下的config文件夹)
     * 2. file:./                 (项目根目录)
     * 3. classpath:/config/      (classpath下的config文件夹)
     * 4. classpath:/             (classpath根目录)
     */

    // 自定义配置文件位置
    @SpringBootApplication
    public class Application {
        public static void main(String[] args) {
            System.setProperty("spring.config.location",
                "classpath:/custom/,file:./config/");
            SpringApplication.run(Application.class, args);
        }
    }
}
```

### 3. Profile配置

```java
// Profile配置示例
@Configuration
@Profile("dev")  // 开发环境配置
public class DevConfig {

    @Bean
    public DataSource dataSource() {
        return new HikariDataSource();
    }
}

@Configuration
@Profile("prod")  // 生产环境配置
public class ProdConfig {

    @Bean
    public DataSource dataSource() {
        return new DruidDataSource();
    }
}

// 激活Profile的方式
public class ProfileActivation {

    /*
     * 1. 配置文件中指定
     * spring.profiles.active=dev,test
     *
     * 2. 命令行参数
     * java -jar app.jar --spring.profiles.active=prod
     *
     * 3. 环境变量
     * export SPRING_PROFILES_ACTIVE=prod
     *
     * 4. 程序中指定
     * SpringApplication app = new SpringApplication(Application.class);
     * app.setAdditionalProfiles("dev");
     */
}
```

## 🏷️ 核心注解解析

### 1. 配置相关注解

```java
// @ConfigurationProperties - 配置属性绑定
@ConfigurationProperties(prefix = "app.user")
@Data
public class UserProperties {
    private String name;
    private Integer age;
    private List<String> hobbies;
    private Map<String, String> settings;

    // 嵌套配置
    private Database database = new Database();

    @Data
    public static class Database {
        private String url;
        private String username;
        private String password;
    }
}

// 使用配置属性
@RestController
@EnableConfigurationProperties(UserProperties.class)
public class UserController {

    private final UserProperties userProperties;

    public UserController(UserProperties userProperties) {
        this.userProperties = userProperties;
    }

    @GetMapping("/user/config")
    public UserProperties getUserConfig() {
        return userProperties;
    }
}
```

### 2. 条件注解详解

```java
// 条件注解示例
@Configuration
public class ConditionalConfig {

    // 基于类存在的条件
    @Bean
    @ConditionalOnClass(RedisTemplate.class)
    public RedisService redisService() {
        return new RedisService();
    }

    // 基于Bean存在的条件
    @Bean
    @ConditionalOnBean(DataSource.class)
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    // 基于Bean不存在的条件
    @Bean
    @ConditionalOnMissingBean(RestTemplate.class)
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    // 基于属性的条件
    @Bean
    @ConditionalOnProperty(
        name = "app.feature.enabled",
        havingValue = "true",
        matchIfMissing = false
    )
    public FeatureService featureService() {
        return new FeatureService();
    }

    // 基于表达式的条件
    @Bean
    @ConditionalOnExpression("${app.cache.enabled:false} and ${app.cache.type:redis} == 'redis'")
    public CacheManager cacheManager() {
        return new RedisCacheManager();
    }

    // 基于Web应用类型的条件
    @Bean
    @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
    public ServletWebServerFactory servletWebServerFactory() {
        return new TomcatServletWebServerFactory();
    }
}
```

### 3. 自定义条件注解

```java
// 自定义条件注解
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Conditional(OnDatabaseCondition.class)
public @interface ConditionalOnDatabase {
    DatabaseType value();
}

// 条件实现类
public class OnDatabaseCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Map<String, Object> attributes = metadata.getAnnotationAttributes(
            ConditionalOnDatabase.class.getName());

        DatabaseType required = (DatabaseType) attributes.get("value");
        String databaseUrl = context.getEnvironment().getProperty("spring.datasource.url");

        if (databaseUrl == null) {
            return false;
        }

        return switch (required) {
            case MYSQL -> databaseUrl.contains("mysql");
            case POSTGRESQL -> databaseUrl.contains("postgresql");
            case ORACLE -> databaseUrl.contains("oracle");
            default -> false;
        };
    }
}

// 使用自定义条件注解
@Configuration
public class DatabaseConfig {

    @Bean
    @ConditionalOnDatabase(DatabaseType.MYSQL)
    public MySQLDialect mysqlDialect() {
        return new MySQLDialect();
    }

    @Bean
    @ConditionalOnDatabase(DatabaseType.POSTGRESQL)
    public PostgreSQLDialect postgresqlDialect() {
        return new PostgreSQLDialect();
    }
}
```

## ❓ 面试常见问题

### Q1: Spring Boot的启动流程是什么？

**答案：**
1. **创建SpringApplication实例**：解析主配置类，确定应用类型
2. **准备环境**：加载配置文件，设置Profile
3. **创建ApplicationContext**：根据应用类型创建对应的上下文
4. **准备上下文**：设置环境，加载配置，注册Bean定义
5. **刷新上下文**：实例化Bean，处理依赖注入
6. **完成启动**：发布启动完成事件，调用Runner

### Q2: Spring Boot自动配置的原理是什么？

**答案：**
```java
// 自动配置原理
public class AutoConfigurationPrinciple {

    /*
     * 1. @EnableAutoConfiguration注解
     *    - 通过@Import导入AutoConfigurationImportSelector
     *
     * 2. AutoConfigurationImportSelector
     *    - 读取META-INF/spring.factories文件
     *    - 获取所有自动配置类的全限定名
     *
     * 3. 条件注解过滤
     *    - @ConditionalOnClass：类路径存在指定类
     *    - @ConditionalOnBean：容器中存在指定Bean
     *    - @ConditionalOnProperty：配置文件中存在指定属性
     *
     * 4. 配置类生效
     *    - 满足条件的配置类被加载
     *    - 创建相应的Bean实例
     */
}
```

### Q3: Spring Boot中Bean的加载顺序如何控制？

**答案：**
```java
// Bean加载顺序控制
@Configuration
public class BeanOrderConfig {

    // 1. 使用@Order注解
    @Bean
    @Order(1)
    public FirstService firstService() {
        return new FirstService();
    }

    @Bean
    @Order(2)
    public SecondService secondService() {
        return new SecondService();
    }

    // 2. 使用@DependsOn注解
    @Bean
    @DependsOn("firstService")
    public DependentService dependentService() {
        return new DependentService();
    }

    // 3. 通过构造函数依赖
    @Bean
    public ServiceWithDependency serviceWithDependency(FirstService firstService) {
        return new ServiceWithDependency(firstService);
    }
}
```

### Q4: Spring Boot如何实现零配置？

**答案：**
```java
// 零配置实现原理
public class ZeroConfigurationPrinciple {

    /*
     * 1. 约定优于配置
     *    - 默认配置文件位置：application.properties/yml
     *    - 默认静态资源位置：/static, /public, /resources, /META-INF/resources
     *    - 默认模板位置：/templates
     *
     * 2. 自动配置
     *    - 根据classpath中的jar包自动配置相应功能
     *    - 提供默认配置，用户可以覆盖
     *
     * 3. 嵌入式服务器
     *    - 内置Tomcat、Jetty、Undertow
     *    - 无需外部服务器部署
     *
     * 4. Starter依赖
     *    - 预定义依赖组合
     *    - 简化依赖管理
     */

    // 示例：Web应用零配置启动
    @SpringBootApplication
    public class ZeroConfigApp {
        public static void main(String[] args) {
            SpringApplication.run(ZeroConfigApp.class, args);
            // 仅需这一行代码即可启动Web应用
        }
    }
}
```

### Q5: Spring Boot Starter的工作原理？

**答案：**
```java
// 自定义Starter示例
// 1. 创建自动配置类
@Configuration
@ConditionalOnClass(MyService.class)
@EnableConfigurationProperties(MyServiceProperties.class)
public class MyServiceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyServiceProperties properties) {
        return new MyService(properties);
    }
}

// 2. 配置属性类
@ConfigurationProperties(prefix = "myservice")
@Data
public class MyServiceProperties {
    private String name = "default";
    private boolean enabled = true;
    private Duration timeout = Duration.ofSeconds(30);
}

// 3. 在META-INF/spring.factories中注册
/*
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.MyServiceAutoConfiguration
*/

// 4. 创建starter模块的pom.xml
/*
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
*/
```

### Q6: Spring Boot中如何处理循环依赖？

**答案：**
```java
// 循环依赖处理
@Service
public class ServiceA {

    // 1. 构造函数循环依赖（Spring无法解决）
    // private final ServiceB serviceB;
    // public ServiceA(ServiceB serviceB) { this.serviceB = serviceB; }

    // 2. 字段注入循环依赖（Spring可以解决）
    @Autowired
    private ServiceB serviceB;

    // 3. 使用@Lazy注解延迟初始化
    @Autowired
    @Lazy
    private ServiceB lazyServiceB;

    // 4. 使用ApplicationContext获取Bean
    @Autowired
    private ApplicationContext applicationContext;

    public void useServiceB() {
        ServiceB serviceB = applicationContext.getBean(ServiceB.class);
        serviceB.doSomething();
    }

    // 5. 使用@PostConstruct
    @PostConstruct
    public void init() {
        // 在初始化完成后处理依赖
    }
}

// 解决循环依赖的最佳实践
@Configuration
public class CircularDependencyConfig {

    // 重新设计，避免循环依赖
    @Bean
    public CommonService commonService() {
        return new CommonService();
    }

    @Bean
    public ServiceA serviceA(CommonService commonService) {
        return new ServiceA(commonService);
    }

    @Bean
    public ServiceB serviceB(CommonService commonService) {
        return new ServiceB(commonService);
    }
}
```

### Q7: Spring Boot的监控和健康检查如何实现？

**答案：**
```java
// Actuator监控配置
@Configuration
public class ActuatorConfig {

    // 1. 自定义健康检查
    @Component
    public class CustomHealthIndicator implements HealthIndicator {

        @Override
        public Health health() {
            // 检查外部服务状态
            boolean externalServiceUp = checkExternalService();

            if (externalServiceUp) {
                return Health.up()
                    .withDetail("external-service", "Available")
                    .withDetail("check-time", System.currentTimeMillis())
                    .build();
            } else {
                return Health.down()
                    .withDetail("external-service", "Unavailable")
                    .withDetail("error", "Connection timeout")
                    .build();
            }
        }

        private boolean checkExternalService() {
            // 实际的健康检查逻辑
            return true;
        }
    }

    // 2. 自定义指标
    @Component
    public class CustomMetrics {

        private final Counter requestCounter;
        private final Timer requestTimer;

        public CustomMetrics(MeterRegistry meterRegistry) {
            this.requestCounter = Counter.builder("custom.requests.total")
                .description("Total number of requests")
                .register(meterRegistry);

            this.requestTimer = Timer.builder("custom.requests.duration")
                .description("Request duration")
                .register(meterRegistry);
        }

        public void recordRequest() {
            requestCounter.increment();
        }

        public Timer.Sample startTimer() {
            return Timer.start();
        }
    }

    // 3. 自定义端点
    @Component
    @Endpoint(id = "custom")
    public class CustomEndpoint {

        @ReadOperation
        public Map<String, Object> customInfo() {
            Map<String, Object> info = new HashMap<>();
            info.put("status", "running");
            info.put("version", "1.0.0");
            info.put("uptime", ManagementFactory.getRuntimeMXBean().getUptime());
            return info;
        }

        @WriteOperation
        public void updateConfig(@Selector String key, String value) {
            // 更新配置的逻辑
            System.setProperty(key, value);
        }
    }
}

// application.yml配置
/*
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,custom
  endpoint:
    health:
      show-details: always
  metrics:
    export:
      prometheus:
        enabled: true
*/
```

## 🛠️ 实战应用场景

### 1. 多数据源配置

```java
@Configuration
public class MultiDataSourceConfig {

    @Primary
    @Bean(name = "primaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "secondaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean(name = "primaryJdbcTemplate")
    public JdbcTemplate primaryJdbcTemplate(@Qualifier("primaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean(name = "secondaryJdbcTemplate")
    public JdbcTemplate secondaryJdbcTemplate(@Qualifier("secondaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

### 2. 异步处理配置

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    @Bean(name = "taskExecutor")
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (ex, method, params) -> {
            System.err.println("异步方法执行异常: " + method.getName());
            ex.printStackTrace();
        };
    }
}

// 使用异步方法
@Service
public class AsyncService {

    @Async("taskExecutor")
    public CompletableFuture<String> processAsync(String data) {
        // 异步处理逻辑
        try {
            Thread.sleep(2000);
            return CompletableFuture.completedFuture("处理完成: " + data);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return CompletableFuture.failedFuture(e);
        }
    }
}
```

### 3. 缓存配置

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        RedisCacheManager.Builder builder = RedisCacheManager
            .RedisCacheManagerBuilder
            .fromConnectionFactory(redisConnectionFactory())
            .cacheDefaults(cacheConfiguration());

        return builder.build();
    }

    private RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(30))
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));
    }
}

// 使用缓存
@Service
public class UserService {

    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        // 查询数据库
        return userRepository.findById(id);
    }

    @CacheEvict(value = "users", key = "#user.id")
    public void updateUser(User user) {
        userRepository.save(user);
    }

    @CacheEvict(value = "users", allEntries = true)
    public void clearAllCache() {
        // 清除所有缓存
    }
}
```

## 📊 性能优化建议

### 1. 启动性能优化

```java
@SpringBootApplication
public class OptimizedApplication {

    public static void main(String[] args) {
        // 1. 设置系统属性优化启动
        System.setProperty("spring.backgroundpreinitializer.ignore", "true");
        System.setProperty("spring.jmx.enabled", "false");

        SpringApplication app = new SpringApplication(OptimizedApplication.class);

        // 2. 禁用不需要的自动配置
        app.setWebApplicationType(WebApplicationType.SERVLET);

        // 3. 设置懒加载
        app.setLazyInitialization(true);

        app.run(args);
    }
}

// application.yml优化配置
/*
spring:
  main:
    lazy-initialization: true
  jmx:
    enabled: false
  autoconfigure:
    exclude:
      - org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration
      - org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration
*/
```

### 2. 内存优化

```java
@Configuration
public class MemoryOptimizationConfig {

    // 1. 使用对象池
    @Bean
    public GenericObjectPool<ExpensiveObject> expensiveObjectPool() {
        GenericObjectPoolConfig<ExpensiveObject> config = new GenericObjectPoolConfig<>();
        config.setMaxTotal(10);
        config.setMaxIdle(5);
        config.setMinIdle(2);

        return new GenericObjectPool<>(new ExpensiveObjectFactory(), config);
    }

    // 2. 配置连接池
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.hikari")
    public HikariDataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setMaximumPoolSize(20);
        dataSource.setMinimumIdle(5);
        dataSource.setConnectionTimeout(30000);
        dataSource.setIdleTimeout(600000);
        dataSource.setMaxLifetime(1800000);
        return dataSource;
    }
}
```

---

*这份Spring Boot 2完整学习指南涵盖了从基础概念到高级应用的所有关键知识点，是学习框架和面试准备的完整资料。*
