# Spring Boot Core — Top 50 Interview Questions & Answers (2026 Edition)

> Focused exclusively on Spring Boot core framework fundamentals.  
> Covers Spring Boot 3.x / Spring Framework 6.x / Jakarta EE migration.  
> Every answer includes code, interviewer traps, and edge cases.

---

## Table of Contents

1. [Spring Boot Fundamentals (Q1–Q10)](#1-spring-boot-fundamentals)
2. [Dependency Injection & IoC (Q11–Q20)](#2-dependency-injection--ioc)
3. [Configuration (Q21–Q28)](#3-configuration)
4. [Starters & Dependencies (Q29–Q33)](#4-starters--dependencies)
5. [Auto-Configuration Deep Dive (Q34–Q40)](#5-auto-configuration-deep-dive)
6. [Actuator — Core Endpoints (Q41–Q44)](#6-actuator--core-endpoints)
7. [Spring Boot Internals (Q45–Q50)](#7-spring-boot-internals)

---

## 1. Spring Boot Fundamentals

---

### Q1. What is `@SpringBootApplication` and what meta-annotations does it combine?

**Answer:**

`@SpringBootApplication` is a convenience **meta-annotation** that combines three annotations:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@SpringBootConfiguration   // ① → implies @Configuration
@EnableAutoConfiguration   // ② → triggers auto-configuration
@ComponentScan             // ③ → scans current package + sub-packages
public @interface SpringBootApplication {
    // exposes attributes from all three
    @AliasFor(annotation = EnableAutoConfiguration.class, attribute = "exclude")
    Class<?>[] exclude() default {};

    @AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
    String[] scanBasePackages() default {};
}
```

| Meta-Annotation            | Purpose                                           |
|----------------------------|---------------------------------------------------|
| `@SpringBootConfiguration` | Marks class as a configuration source (wraps `@Configuration` with `proxyBeanMethods=true` by default) |
| `@EnableAutoConfiguration` | Activates auto-configuration via `AutoConfigurationImportSelector` |
| `@ComponentScan`           | Recursively scans from the annotated class's package downward |

**Interview Trap:** "Can you use these three annotations separately instead of `@SpringBootApplication`?"  
→ Yes. This is common when you need **different base packages** for component scanning vs. the main class location, or when you want to **exclude specific auto-configurations** more granularly.

**Edge Case:** If your main class is in `com.example`, only `com.example.**` is scanned. Beans in `com.other` are invisible unless you explicitly add `scanBasePackages`.

```java
// Equivalent to @SpringBootApplication but with fine-grained control
@SpringBootConfiguration
@EnableAutoConfiguration(exclude = DataSourceAutoConfiguration.class)
@ComponentScan(basePackages = {"com.example.core", "com.example.api"})
public class MyApp { }
```

---

### Q2. How does Spring Boot auto-configuration work internally?

**Answer:**

Auto-configuration follows this pipeline:

1. `@EnableAutoConfiguration` imports `AutoConfigurationImportSelector`.
2. The selector reads **candidate configuration classes** from:
   - **Spring Boot 2.x:** `META-INF/spring.factories` under key `EnableAutoConfiguration`
   - **Spring Boot 3.x:** `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (new file-per-line format)
3. Candidates are **filtered** by `@Conditional*` annotations — only classes whose conditions pass are loaded.
4. Surviving configurations are **ordered** via `@AutoConfigureOrder`, `@AutoConfigureBefore`, `@AutoConfigureAfter`.
5. Beans defined in those configurations are registered into the `ApplicationContext`.

```
┌──────────────────────────────┐
│  @EnableAutoConfiguration    │
│  ↓                           │
│  AutoConfigurationImport     │
│  Selector                    │
│  ↓                           │
│  Load candidates from        │
│  .imports / spring.factories │
│  ↓                           │
│  Apply @Conditional filters  │
│  ↓                           │
│  Order remaining configs     │
│  ↓                           │
│  Register beans              │
└──────────────────────────────┘
```

**Spring Boot 3.x Change:**  
`spring.factories` for auto-configuration is **deprecated**. The new `.imports` file uses a simpler format — one fully qualified class name per line:

```text
# META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
com.example.MyCustomAutoConfiguration
com.example.AnotherAutoConfiguration
```

**Interview Trap:** "Does auto-configuration override your explicit beans?"  
→ No. Auto-configuration classes use `@ConditionalOnMissingBean` extensively, meaning your **explicit `@Bean` definitions always win**.

---

### Q3. Explain the key `@Conditional` annotations used in auto-configuration.

**Answer:**

| Annotation                       | Condition Passes When...                                              |
|----------------------------------|-----------------------------------------------------------------------|
| `@ConditionalOnClass`            | Specified class is on the classpath                                   |
| `@ConditionalOnMissingClass`     | Specified class is NOT on the classpath                               |
| `@ConditionalOnBean`             | A bean of specified type/name already exists in the context            |
| `@ConditionalOnMissingBean`      | No bean of specified type/name exists — **most common in auto-config** |
| `@ConditionalOnProperty`         | A configuration property has a specific value                         |
| `@ConditionalOnResource`         | A specified resource (file) exists on the classpath                   |
| `@ConditionalOnWebApplication`   | Running in a web application context (Servlet or Reactive)            |
| `@ConditionalOnNotWebApplication`| NOT running in a web context                                          |
| `@ConditionalOnExpression`       | A SpEL expression evaluates to `true`                                 |
| `@ConditionalOnJava`             | Running on a specific Java version range                              |

```java
@Configuration
@ConditionalOnClass(DataSource.class)           // only if JDBC is on classpath
@ConditionalOnProperty(
    prefix = "app.datasource",
    name = "enabled",
    havingValue = "true",
    matchIfMissing = true                       // defaults to enabled
)
public class MyDataSourceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean                   // user's bean takes priority
    public DataSource dataSource() {
        return new HikariDataSource();
    }
}
```

**Interview Trap:** "What's the difference between `@ConditionalOnBean` evaluated at class level vs. method level?"  
→ **Class-level:** evaluated early, before any `@Bean` methods in that class run. The entire configuration is skipped if the condition fails.  
→ **Method-level:** evaluated later, per-bean. Other beans in the same configuration may still be registered.

**Pitfall:** `@ConditionalOnBean` depends on **bean definition order**. If the bean you're checking for hasn't been defined yet (processed later), the condition will fail. Always use `@AutoConfigureAfter` to control ordering.

---

### Q4. How does component scanning work and what are its boundaries?

**Answer:**

`@ComponentScan` (included via `@SpringBootApplication`) recursively scans the **package of the annotated class** and all its sub-packages for classes annotated with **stereotype annotations**:

| Stereotype        | Purpose                                  |
|-------------------|------------------------------------------|
| `@Component`      | Generic Spring-managed component         |
| `@Service`        | Business logic layer (semantic)          |
| `@Repository`     | Data access layer + exception translation|
| `@Controller`     | Spring MVC controller                    |
| `@RestController` | `@Controller` + `@ResponseBody`          |
| `@Configuration`  | Java-based configuration class           |

```java
// Main class in com.example.app → scans com.example.app.**
@SpringBootApplication
public class MyApp { }

// ❌ This bean will NOT be found — it's outside the scan boundary
package com.other.service;
@Service
public class ExternalService { }
```

**Customizing scan scope:**

```java
@SpringBootApplication(scanBasePackages = {
    "com.example.app",
    "com.other.service"       // now included
})
public class MyApp { }

// OR use filters
@ComponentScan(
    basePackages = "com.example",
    includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyCustomAnnotation.class),
    excludeFilters = @Filter(type = FilterType.REGEX, pattern = "com\\.example\\.test\\..*")
)
```

**Interview Trap:** "Does `@ComponentScan` find `@Bean` methods inside `@Configuration` classes?"  
→ No directly — it finds the `@Configuration` **class** via its stereotype. The `@Bean` methods inside are then processed by Spring's `ConfigurationClassPostProcessor`, not by component scanning itself.

---

### Q5. Describe the Spring Boot application startup lifecycle.

**Answer:**

```
main() → SpringApplication.run()
  │
  ├── 1. Create SpringApplication instance
  │     • Detect web application type (NONE, SERVLET, REACTIVE)
  │     • Load ApplicationContextInitializers (from spring.factories)
  │     • Load ApplicationListeners (from spring.factories)
  │     • Deduce main application class
  │
  ├── 2. Run
  │     a. Create & start StopWatch
  │     b. Create BootstrapContext
  │     c. Publish ApplicationStartingEvent
  │     d. Prepare Environment → ApplicationEnvironmentPreparedEvent
  │     e. Print Banner
  │     f. Create ApplicationContext (based on web type)
  │     g. Prepare context (apply initializers, register sources)
  │     h. Refresh context → bean creation, auto-configuration
  │     i. Publish ApplicationStartedEvent
  │     j. Call Runners (ApplicationRunner, CommandLineRunner)
  │     k. Publish ApplicationReadyEvent
  │
  └── 3. Return ConfigurableApplicationContext
```

**Key Events in Order:**

| Event                                | When                                              |
|--------------------------------------|---------------------------------------------------|
| `ApplicationStartingEvent`           | After `run()` starts, before any processing       |
| `ApplicationEnvironmentPreparedEvent`| Environment ready, context not yet created         |
| `ApplicationContextInitializedEvent` | Context created, before bean definitions loaded    |
| `ApplicationPreparedEvent`           | Bean definitions loaded, before refresh            |
| `ApplicationStartedEvent`            | Context refreshed, before runners called           |
| `AvailabilityChangeEvent(CORRECT)`   | Liveness = CORRECT state                           |
| `ApplicationReadyEvent`              | After runners complete — app is fully ready        |
| `AvailabilityChangeEvent(ACCEPTING)` | Readiness = ACCEPTING_TRAFFIC                      |
| `ApplicationFailedEvent`             | If startup fails at any point                      |

```java
@Component
public class StartupListener {

    @EventListener(ApplicationReadyEvent.class)
    public void onReady(ApplicationReadyEvent event) {
        System.out.println("Application is ready to serve traffic!");
    }

    @EventListener(ApplicationStartedEvent.class)
    public void onStarted(ApplicationStartedEvent event) {
        System.out.println("Context refreshed, about to call runners...");
    }
}
```

**Interview Trap:** "What's the difference between `ApplicationStartedEvent` and `ApplicationReadyEvent`?"  
→ `Started` fires **before** runners; `Ready` fires **after** runners. If a `CommandLineRunner` fails, `ApplicationReadyEvent` is never published — `ApplicationFailedEvent` fires instead.

---

### Q6. What is the difference between `@SpringBootConfiguration` and `@Configuration`?

**Answer:**

`@SpringBootConfiguration` is a **specialization** of `@Configuration`:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Configuration
public @interface SpringBootConfiguration {
    @AliasFor(annotation = Configuration.class)
    boolean proxyBeanMethods() default true;
}
```

| Aspect                    | `@Configuration`                      | `@SpringBootConfiguration`              |
|---------------------------|----------------------------------------|-----------------------------------------|
| Bean proxying default     | `proxyBeanMethods = true`              | Same — `true`                           |
| Semantic meaning          | Generic config class                   | "This is the main config for a Boot app"|
| Used by test framework?   | No special treatment                   | `@SpringBootTest` searches up packages to find it |
| Restriction               | Multiple allowed per app               | **Should have only one per application** |

**Why it matters in tests:**

```java
@SpringBootTest  // searches for @SpringBootConfiguration to determine the root config
class MyIntegrationTest {
    // If you have two @SpringBootConfiguration classes, tests may pick the wrong one
}
```

**Interview Trap:** "What does `proxyBeanMethods = true` do?"  
→ It creates a **CGLIB proxy** of the configuration class so that inter-`@Bean` method calls return the **same singleton** instance rather than creating a new object each time. Setting it to `false` (lite mode) skips the proxy — useful for performance but breaks singleton guarantees on cross-method calls.

```java
@Configuration(proxyBeanMethods = true) // FULL mode (default)
public class AppConfig {

    @Bean
    public ServiceA serviceA() {
        return new ServiceA(commonDep()); // returns THE SAME singleton
    }

    @Bean
    public CommonDep commonDep() {
        return new CommonDep();
    }
}

@Configuration(proxyBeanMethods = false) // LITE mode
public class LiteConfig {

    @Bean
    public ServiceA serviceA() {
        return new ServiceA(commonDep()); // creates NEW instance!
    }

    @Bean
    public CommonDep commonDep() {
        return new CommonDep();
    }
}
```

---

### Q7. How do you exclude specific auto-configurations?

**Answer:**

Four methods, in order of preference:

```java
// Method 1: Via @SpringBootApplication attribute (most common)
@SpringBootApplication(exclude = {
    DataSourceAutoConfiguration.class,
    HibernateJpaAutoConfiguration.class
})
public class MyApp { }

// Method 2: Via @EnableAutoConfiguration attribute
@EnableAutoConfiguration(exclude = DataSourceAutoConfiguration.class)

// Method 3: Via property (useful for externalized config)
// application.properties
spring.autoconfigure.exclude=\
  org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
  org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration

// Method 4: Via excludeName (when class not on classpath)
@SpringBootApplication(excludeName = "org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration")
```

**When to use which:**

| Method              | Use When                                            |
|---------------------|-----------------------------------------------------|
| `exclude`           | Class is on classpath, permanent exclusion           |
| `excludeName`       | Class may not be on classpath, avoids compile error  |
| `spring.autoconfigure.exclude` | Environment-specific exclusion, test profiles |

**Interview Trap:** "What happens if you exclude a class that doesn't exist?"  
→ `exclude` with `Class<?>` → **compile error** if the class isn't on the classpath.  
→ `excludeName` with `String` → silently ignored if class doesn't exist.

---

### Q8. What are the different ways to run a Spring Boot application?

**Answer:**

```java
// 1. Traditional main method
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}

// 2. Using SpringApplicationBuilder (fluent API, useful for customization)
new SpringApplicationBuilder(MyApp.class)
    .bannerMode(Banner.Mode.OFF)
    .profiles("dev")
    .run(args);

// 3. Executable JAR (after mvn package / gradle bootJar)
// java -jar myapp-1.0.0.jar --server.port=9090

// 4. Maven plugin
// mvn spring-boot:run

// 5. Gradle plugin
// gradle bootRun

// 6. WAR deployment (extending SpringBootServletInitializer)
@SpringBootApplication
public class MyApp extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(MyApp.class);
    }
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

**Interview Trap:** "Why might you use `SpringApplicationBuilder` instead of `SpringApplication.run()`?"  
→ Fluent API supports: parent-child context hierarchies, programmatic profile activation, custom banner, property sources added before startup, listener registration, and building complex startup sequences.

---

### Q9. What changed in Spring Boot 3.x with the Jakarta EE migration?

**Answer:**

Spring Boot 3.0+ requires **Java 17 minimum** and migrates from Java EE (`javax.*`) to **Jakarta EE 9+ (`jakarta.*`)**.

| Aspect                   | Spring Boot 2.x            | Spring Boot 3.x                |
|--------------------------|----------------------------|---------------------------------|
| Java baseline            | Java 8 / 11               | **Java 17** minimum             |
| Namespace                | `javax.servlet.*`          | `jakarta.servlet.*`             |
| Persistence              | `javax.persistence.*`      | `jakarta.persistence.*`         |
| Validation               | `javax.validation.*`       | `jakarta.validation.*`          |
| Spring Framework         | 5.x                        | **6.x**                         |
| Auto-config registration | `spring.factories`         | `.imports` file (new format)    |
| Observability            | Micrometer + Sleuth        | Micrometer + **Micrometer Tracing** (Sleuth deprecated) |
| Native images            | Experimental               | **First-class GraalVM support** |
| HTTP interfaces          | Not available              | `@HttpExchange` declarative clients |
| Problem Details          | Manual                     | RFC 7807 built-in via `ProblemDetail` |

```java
// Spring Boot 2.x
import javax.servlet.http.HttpServletRequest;
import javax.persistence.Entity;

// Spring Boot 3.x → must change ALL imports
import jakarta.servlet.http.HttpServletRequest;
import jakarta.persistence.Entity;
```

**Interview Trap:** "Is `spring.factories` completely removed in Boot 3?"  
→ No. `spring.factories` still works for **other** extension points (like `ApplicationContextInitializer`, `EnvironmentPostProcessor`). Only the `EnableAutoConfiguration` key is deprecated — use the new `.imports` file for auto-configuration classes.

---

### Q10. What is the purpose of the `spring-boot-devtools` module?

**Answer:**

`devtools` enhances the development experience with:

| Feature                  | Description                                                  |
|--------------------------|--------------------------------------------------------------|
| **Automatic Restart**    | Restarts app on classpath changes using two classloaders (base + restart). Only the restart classloader reloads. |
| **LiveReload**           | Triggers browser refresh when resources change               |
| **Property Defaults**    | Disables template caching, enables DEBUG logging for web     |
| **Remote Debugging**     | Optional remote update + restart for deployed apps           |
| **H2 Console**           | Auto-enables `/h2-console` in development                    |

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>  <!-- not transitive to dependents -->
</dependency>
```

**Critical Detail — Two Classloader Architecture:**

```
Base ClassLoader → third-party JARs (rarely change) — NOT restarted
Restart ClassLoader → your application classes — discarded & recreated on change
```

**Interview Trap:** "Is `devtools` active in production?"  
→ **No.** It is **automatically disabled** when running as a packaged JAR (`java -jar`), when launched with a special classloader, or when the `spring.devtools.restart.enabled=false` property is set. The `optional=true` Maven scope ensures it's not included in transitive dependencies.

**Pitfall:** `devtools` changes default property values (e.g., disables Thymeleaf caching). A bug that only appears in production but not in dev may be caused by these overrides.

---

## 2. Dependency Injection & IoC

---

### Q11. Describe the complete Spring bean lifecycle.

**Answer:**

```
1.  Bean Definition Loading (from @Component, @Bean, XML)
2.  BeanFactoryPostProcessor executes (can modify definitions)
         ↓
3.  Bean Instantiation (constructor call)
4.  Dependency Injection (field, setter, constructor)
5.  BeanNameAware.setBeanName()
6.  BeanFactoryAware.setBeanFactory()
7.  ApplicationContextAware.setApplicationContext()
8.  BeanPostProcessor.postProcessBeforeInitialization()
9.  @PostConstruct method
10. InitializingBean.afterPropertiesSet()
11. Custom init-method (via @Bean(initMethod="..."))
12. BeanPostProcessor.postProcessAfterInitialization()
         ↓ (bean is now ready for use)
         ↓ ... application runs ...
         ↓
13. @PreDestroy method
14. DisposableBean.destroy()
15. Custom destroy-method (via @Bean(destroyMethod="..."))
```

```java
@Component
public class LifecycleDemo implements
        BeanNameAware, InitializingBean, DisposableBean {

    private final MyDependency dep;

    // Step 3: Instantiation + Step 4: Constructor injection
    public LifecycleDemo(MyDependency dep) {
        System.out.println("1. Constructor");
        this.dep = dep;
    }

    // Step 5
    @Override
    public void setBeanName(String name) {
        System.out.println("2. BeanNameAware: " + name);
    }

    // Step 9
    @PostConstruct
    public void postConstruct() {
        System.out.println("3. @PostConstruct");
    }

    // Step 10
    @Override
    public void afterPropertiesSet() {
        System.out.println("4. InitializingBean.afterPropertiesSet");
    }

    // Step 13
    @PreDestroy
    public void preDestroy() {
        System.out.println("5. @PreDestroy");
    }

    // Step 14
    @Override
    public void destroy() {
        System.out.println("6. DisposableBean.destroy");
    }
}
```

**Interview Trap:** "In what order do `@PostConstruct`, `InitializingBean`, and custom `initMethod` execute?"  
→ `@PostConstruct` → `InitializingBean.afterPropertiesSet()` → custom `initMethod`. Same order for destruction: `@PreDestroy` → `DisposableBean.destroy()` → custom `destroyMethod`.

**Edge Case:** In Spring Boot 3.x (Jakarta EE), `@PostConstruct` and `@PreDestroy` come from `jakarta.annotation` package, not `javax.annotation`.

---

### Q12. What are the different bean scopes in Spring?

**Answer:**

| Scope         | Instances | Available In        | Description                                       |
|---------------|-----------|---------------------|---------------------------------------------------|
| `singleton`   | 1         | All contexts        | **Default.** One instance per ApplicationContext   |
| `prototype`   | N         | All contexts        | New instance every time the bean is requested      |
| `request`     | 1/request | Web only            | One instance per HTTP request                      |
| `session`     | 1/session | Web only            | One instance per HTTP session                      |
| `application` | 1/app     | Web only            | One instance per `ServletContext`                  |
| `websocket`   | 1/ws      | WebSocket only      | One instance per WebSocket session                 |

```java
@Component
@Scope("prototype")
public class PrototypeBean {
    private final String id = UUID.randomUUID().toString();
}

@Component
public class SingletonBean {
    private final PrototypeBean proto;

    // ❌ TRAP: This injects ONE prototype instance into the singleton!
    // The same instance is reused for the singleton's entire lifecycle.
    public SingletonBean(PrototypeBean proto) {
        this.proto = proto;
    }
}
```

**The Singleton–Prototype Injection Problem:**

When a singleton depends on a prototype, the prototype is resolved **once** at singleton creation. Solutions:

```java
// Solution 1: ObjectFactory / ObjectProvider
@Component
public class SingletonBean {
    private final ObjectProvider<PrototypeBean> protoProvider;

    public SingletonBean(ObjectProvider<PrototypeBean> protoProvider) {
        this.protoProvider = protoProvider;
    }

    public void doWork() {
        PrototypeBean fresh = protoProvider.getObject(); // new instance each call
    }
}

// Solution 2: @Lookup method injection
@Component
public abstract class SingletonBean {
    @Lookup
    public abstract PrototypeBean getPrototype(); // Spring overrides this
}

// Solution 3: Scoped proxy
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class PrototypeBean { }
```

**Interview Trap:** "Are prototype beans destroyed by Spring?"  
→ **No.** Spring does not manage the full lifecycle of prototype beans. `@PreDestroy` and `DisposableBean.destroy()` are **never called** by the container for prototype-scoped beans. You must handle cleanup yourself.

---

### Q13. Compare `@Autowired`, `@Inject`, and constructor injection.

**Answer:**

| Feature                  | `@Autowired` (Spring)    | `@Inject` (Jakarta/JSR-330)   | Constructor Injection       |
|--------------------------|--------------------------|-------------------------------|-----------------------------|
| Source                   | `org.springframework`    | `jakarta.inject` (Boot 3.x)  | No annotation needed        |
| `required` attribute     | Yes (`required=false`)   | No — always required          | N/A (implicit if single constructor) |
| Qualifiers               | `@Qualifier`             | `@Named`                      | `@Qualifier` on param       |
| Field injection          | Yes                      | Yes                           | No                          |
| Setter injection         | Yes                      | Yes                           | Yes (with `@Autowired`)     |
| Recommended?             | Acceptable               | Acceptable                    | **Yes — best practice**     |

```java
// ✅ BEST PRACTICE: Constructor injection (no annotation needed with single constructor)
@Service
public class OrderService {
    private final PaymentGateway gateway;
    private final OrderRepository repo;

    // Spring auto-detects this as the injection point (since Spring 4.3)
    public OrderService(PaymentGateway gateway, OrderRepository repo) {
        this.gateway = gateway;
        this.repo = repo;
    }
}

// ⚠️ Field injection — harder to test, hides dependencies
@Service
public class OrderService {
    @Autowired
    private PaymentGateway gateway;   // can't be final
}

// Optional dependency with constructor injection
@Service
public class OrderService {
    private final Optional<CacheManager> cacheManager;

    public OrderService(Optional<CacheManager> cacheManager) {
        this.cacheManager = cacheManager;
    }
}
```

**Why constructor injection is preferred:**

1. Enables **immutable** fields (`final`).
2. Required dependencies are enforced at **compile time** — not runtime NPE.
3. Easy to **unit test** without Spring context — just call the constructor.
4. Makes **circular dependencies** immediately visible (fail-fast).

**Interview Trap:** "When is `@Autowired` still required on a constructor?"  
→ When the class has **multiple constructors**. Spring cannot auto-detect which one to use unless exactly one is annotated with `@Autowired`.

---

### Q14. How does Spring resolve circular dependencies, and what changed in Boot 3.x?

**Answer:**

**Circular dependency:** Bean A depends on Bean B, and Bean B depends on Bean A.

**Constructor injection:** → `BeanCurrentlyInCreationException` — **always fails.** No workaround exists in the container itself.

**Field/setter injection (singleton scope):** Spring historically resolved this using **three-level caches** (early reference exposure):

```
singletonObjects          → fully initialized beans (Level 1)
earlySingletonObjects     → early references (partially initialized) (Level 2)
singletonFactories        → ObjectFactory to create early references (Level 3)
```

Process: Spring creates A, finds it needs B, starts creating B, finds it needs A, gets A's **early reference** from the factory cache, completes B, then completes A.

**Spring Boot 2.6+ / 3.x change:**

```properties
# Circular dependencies are PROHIBITED BY DEFAULT since Boot 2.6
# This will throw an error at startup
spring.main.allow-circular-references=false   # DEFAULT

# To temporarily allow (NOT recommended)
spring.main.allow-circular-references=true
```

**Proper solutions:**

```java
// Solution 1: Redesign — extract shared logic
@Service
public class OrderService {
    private final SharedLogic shared;
    // ...
}

@Service
public class PaymentService {
    private final SharedLogic shared;
    // ...
}

// Solution 2: @Lazy on one dependency
@Service
public class OrderService {
    private final PaymentService paymentService;

    public OrderService(@Lazy PaymentService paymentService) {
        this.paymentService = paymentService; // proxy injected, resolved on first use
    }
}

// Solution 3: Use events to decouple
@Service
public class OrderService {
    private final ApplicationEventPublisher publisher;

    public void placeOrder(Order order) {
        // ...
        publisher.publishEvent(new OrderPlacedEvent(order));
    }
}

@Service
public class PaymentService {
    @EventListener
    public void onOrderPlaced(OrderPlacedEvent event) {
        // process payment
    }
}
```

**Interview Trap:** "Does `@Lazy` solve circular dependencies with constructor injection?"  
→ **Yes**, because `@Lazy` injects a **proxy**, not the actual bean. The proxy is created without needing the real bean, breaking the cycle. The real bean is resolved on first method call.

---

### Q15. Explain `@Lazy` initialization — bean-level vs. global.

**Answer:**

```java
// Bean-level lazy initialization
@Component
@Lazy
public class ExpensiveService {
    public ExpensiveService() {
        System.out.println("Initialized only when first requested");
    }
}

// Injection-point lazy
@Service
public class MyService {
    private final ExpensiveService expensive;

    public MyService(@Lazy ExpensiveService expensive) {
        this.expensive = expensive; // proxy, not real instance
    }
}
```

**Global lazy initialization (Spring Boot 2.2+):**

```properties
spring.main.lazy-initialization=true
```

| Aspect                | Eager (default)                    | Lazy                                |
|-----------------------|------------------------------------|-------------------------------------|
| Initialization        | At context startup                 | On first access                     |
| Startup time          | Slower                             | Faster                              |
| Fail-fast             | Yes — errors caught at startup     | No — errors deferred to runtime     |
| Memory at startup     | Higher                             | Lower initially                     |

**Interview Trap:** "What's the danger of global lazy initialization?"  
→ Configuration errors and missing beans are **not detected at startup** — they surface at runtime when the bean is first accessed, potentially in production. Also, the **first request** to the application may be slow because beans are initialized on-demand.

**Edge Case:** `@Lazy(false)` on a specific bean **overrides** the global lazy setting, forcing eager initialization for critical beans like health checks.

---

### Q16. How does bean definition overriding work?

**Answer:**

By default in Spring Boot 2.1+, bean overriding is **disabled**:

```properties
# Default since Boot 2.1
spring.main.allow-bean-definition-overriding=false

# Enable if needed
spring.main.allow-bean-definition-overriding=true
```

When enabled, if two beans share the same name, the **last one registered wins**:

```java
@Configuration
public class Config1 {
    @Bean
    public MyService myService() {
        return new MyServiceImpl1();  // registered first
    }
}

@Configuration
public class Config2 {
    @Bean
    public MyService myService() {
        return new MyServiceImpl2();  // overrides Config1's bean
    }
}
```

**Resolution order** (later wins):
1. Component-scanned beans
2. Java `@Configuration` beans (in source order)
3. Auto-configuration beans (processed last, designed to be overridden)

**Interview Trap:** "If overriding is disabled and two beans have the same name, what happens?"  
→ `BeanDefinitionOverrideException` at startup with a clear error message naming both conflicting definitions.

**Best practice:** Use `@ConditionalOnMissingBean` in your own configurations to avoid hard conflicts:

```java
@Configuration
public class DefaultConfig {
    @Bean
    @ConditionalOnMissingBean
    public MyService myService() {
        return new DefaultMyService(); // only if no other MyService exists
    }
}
```

---

### Q17. Explain `@Primary` and `@Qualifier` for resolving ambiguous dependencies.

**Answer:**

When multiple beans of the same type exist, Spring throws `NoUniqueBeanDefinitionException`. Resolution strategies:

```java
public interface NotificationSender {
    void send(String message);
}

@Component("emailSender")
public class EmailSender implements NotificationSender { }

@Component("smsSender")
@Primary  // ← this bean is chosen when no qualifier is specified
public class SmsSender implements NotificationSender { }

// Injection scenarios:
@Service
public class NotificationService {

    // Uses SmsSender because it's @Primary
    public NotificationService(NotificationSender sender) { }

    // Explicitly chooses EmailSender regardless of @Primary
    public NotificationService(@Qualifier("emailSender") NotificationSender sender) { }
}
```

| Mechanism         | Scope        | Use When                                          |
|-------------------|--------------|---------------------------------------------------|
| `@Primary`        | Global       | One bean should be the default for its type        |
| `@Qualifier`      | Per-injection | You need a specific bean at a specific injection point |
| Bean name match   | Per-injection | Parameter name matches the bean name               |

**Resolution priority:** `@Qualifier` > `@Primary` > bean name match > fail.

```java
// Bean name matching (implicit qualifier)
@Service
public class NotificationService {
    // Parameter name "emailSender" matches bean name → EmailSender injected
    public NotificationService(NotificationSender emailSender) { }
}
```

**Custom qualifier annotations (best practice for type safety):**

```java
@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Email { }

@Component
@Email
public class EmailSender implements NotificationSender { }

@Service
public class NotificationService {
    public NotificationService(@Email NotificationSender sender) { }
}
```

---

### Q18. What is `ObjectProvider` and when should you use it?

**Answer:**

`ObjectProvider<T>` is a Spring interface that provides **deferred and safe** bean resolution:

```java
@Service
public class ReportService {
    private final ObjectProvider<CacheManager> cacheProvider;

    public ReportService(ObjectProvider<CacheManager> cacheProvider) {
        this.cacheProvider = cacheProvider;
    }

    public void generateReport() {
        // Safe — returns null if bean doesn't exist
        CacheManager cache = cacheProvider.getIfAvailable();

        // With fallback
        CacheManager cache2 = cacheProvider.getIfAvailable(NoOpCacheManager::new);

        // Iterate over all beans of this type
        cacheProvider.orderedStream().forEach(cm -> cm.clearAll());

        // Get unique bean or throw if ambiguous
        CacheManager cache3 = cacheProvider.getIfUnique();
    }
}
```

| Method                    | Behavior                                             |
|---------------------------|------------------------------------------------------|
| `getObject()`             | Returns bean or throws `NoSuchBeanDefinitionException` |
| `getIfAvailable()`        | Returns bean or `null`                                |
| `getIfAvailable(Supplier)`| Returns bean or supplier's result                     |
| `getIfUnique()`           | Returns bean only if exactly one exists, else `null`  |
| `orderedStream()`         | Returns ordered stream of all matching beans          |
| `forEach(Consumer)`       | Iterates all beans of type                            |

**Use cases:**
- **Optional dependencies** — without `@Autowired(required=false)`
- **Multiple implementations** — iterate or pick specific ones
- **Prototype scope** — get new instance each time via `getObject()`
- **Avoiding eager initialization** — bean resolved on first call

**Interview Trap:** "How is `ObjectProvider` different from `@Autowired(required=false)`?"  
→ `ObjectProvider` is more powerful: supports fallbacks, streaming multiple beans, uniqueness checks, and works with constructor injection without making the constructor parameter nullable.

---

### Q19. How do `@Import` and `@ImportResource` work?

**Answer:**

```java
// @Import — directly registers configuration classes or individual beans
@Configuration
@Import({
    DatabaseConfig.class,           // another @Configuration class
    ExternalService.class,          // a plain @Component class
    MyImportSelector.class,         // an ImportSelector implementation
    MyImportBeanDefinitionRegistrar.class // programmatic registration
})
public class AppConfig { }

// ImportSelector — dynamically choose which configs to import
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata metadata) {
        // Can read annotation attributes from metadata
        return new String[] {
            "com.example.config.CacheConfig",
            "com.example.config.MetricsConfig"
        };
    }
}

// ImportBeanDefinitionRegistrar — full control over bean registration
public class MyRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata,
                                         BeanDefinitionRegistry registry) {
        GenericBeanDefinition def = new GenericBeanDefinition();
        def.setBeanClass(MyCustomBean.class);
        registry.registerBeanDefinition("myCustomBean", def);
    }
}
```

```java
// @ImportResource — loads XML bean definitions
@Configuration
@ImportResource("classpath:legacy-beans.xml")
public class LegacyConfig { }
```

| Mechanism                     | Use Case                                            |
|-------------------------------|-----------------------------------------------------|
| `@Import(Config.class)`       | Pull in configuration outside component scan range   |
| `@Import(ImportSelector)`     | Conditional or dynamic configuration selection       |
| `@Import(Registrar)`          | Programmatic bean definition (frameworks, proxies)   |
| `@ImportResource`             | Legacy XML configuration migration                   |

**Interview Trap:** "`AutoConfigurationImportSelector` — what is it?"  
→ It's the `ImportSelector` used by `@EnableAutoConfiguration` to load all auto-configuration candidates. This is the **core mechanism** powering Spring Boot's auto-configuration.

---

### Q20. What are `BeanPostProcessor` and `BeanFactoryPostProcessor`?

**Answer:**

| Aspect                   | `BeanFactoryPostProcessor`           | `BeanPostProcessor`                         |
|--------------------------|--------------------------------------|---------------------------------------------|
| **When invoked**         | After bean **definitions** loaded, before any beans instantiated | After each bean is **instantiated** |
| **Operates on**          | Bean **definitions** (metadata)      | Bean **instances**                          |
| **Use case**             | Modify property values, add definitions | Wrap beans in proxies, modify instances     |
| **Key implementation**   | `PropertySourcesPlaceholderConfigurer` | `AutowiredAnnotationBeanPostProcessor`     |

```java
// BeanFactoryPostProcessor — modifies definitions before instantiation
@Component
public class MyBeanFactoryPP implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory factory) {
        BeanDefinition def = factory.getBeanDefinition("myService");
        def.getPropertyValues().add("timeout", 5000);
    }
}

// BeanPostProcessor — intercepts every bean after instantiation
@Component
public class MyBeanPP implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String name) {
        // Called BEFORE @PostConstruct / afterPropertiesSet
        if (bean instanceof MyService svc) {
            svc.setAuditEnabled(true);
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String name) {
        // Called AFTER initialization — common place to create proxies
        if (bean.getClass().isAnnotationPresent(Timed.class)) {
            return createTimingProxy(bean);
        }
        return bean;
    }
}
```

**How Spring uses these internally:**

| Processor                                      | What It Does                                    |
|------------------------------------------------|-------------------------------------------------|
| `ConfigurationClassPostProcessor` (BFPP)       | Processes `@Configuration`, `@Bean`, `@Import`  |
| `PropertySourcesPlaceholderConfigurer` (BFPP)  | Resolves `${...}` placeholders in definitions   |
| `AutowiredAnnotationBeanPostProcessor` (BPP)   | Processes `@Autowired`, `@Value` injection       |
| `CommonAnnotationBeanPostProcessor` (BPP)      | Processes `@PostConstruct`, `@PreDestroy`, `@Resource` |
| `AnnotationAwareAspectJAutoProxyCreator` (BPP) | Creates AOP proxies for `@Aspect` classes        |

**Interview Trap:** "If a `BeanPostProcessor` itself has `@Autowired` dependencies, are those dependencies also post-processed?"  
→ **No.** `BeanPostProcessor` beans are instantiated **very early**. Their dependencies may not benefit from other BPPs. Spring logs a warning: "Bean 'X' is not eligible for getting processed by all BeanPostProcessors."

---

## 3. Configuration

---

### Q21. Compare `application.properties` vs. `application.yml`. Which takes precedence?

**Answer:**

```properties
# application.properties — flat key-value
server.port=8080
spring.datasource.url=jdbc:mysql://localhost/db
spring.datasource.username=root
app.feature.flags[0]=flag-a
app.feature.flags[1]=flag-b
```

```yaml
# application.yml — hierarchical YAML
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://localhost/db
    username: root
app:
  feature:
    flags:
      - flag-a
      - flag-b
```

| Aspect            | `.properties`                         | `.yml`                                   |
|-------------------|---------------------------------------|------------------------------------------|
| Syntax            | Flat key=value                        | Hierarchical indentation                 |
| Lists             | Indexed: `prop[0]=a`                  | YAML list: `- a`                         |
| Multi-document    | Not supported                         | Supported via `---` separator            |
| Profile-specific  | Separate files                        | Multi-document in one file or separate   |
| Precedence        | **Wins over .yml at same location**   | Loaded first, overridden by .properties  |

**Interview Trap:** "If both `application.properties` and `application.yml` exist, which wins?"  
→ **`.properties` overrides `.yml`** when both are in the same location. Properties files are loaded **after** YAML files, so they override matching keys.

**YAML multi-document profiles:**

```yaml
# Default properties (all profiles)
spring:
  application:
    name: my-app
---
spring:
  config:
    activate:
      on-profile: dev
server:
  port: 8081
---
spring:
  config:
    activate:
      on-profile: prod
server:
  port: 80
```

**Edge Case:** YAML is strict about indentation — tabs are **not allowed**. A misplaced tab silently corrupts values.

---

### Q22. Compare `@ConfigurationProperties` vs. `@Value`.

**Answer:**

```java
// @ConfigurationProperties — type-safe, structured binding
@Component
@ConfigurationProperties(prefix = "app.mail")
public class MailProperties {
    private String host;
    private int port = 25;              // default value
    private List<String> recipients;
    private Duration timeout;           // auto-converts "30s" → Duration

    // getters and setters required (or use records in Boot 3.x)
}

// @Value — individual property injection
@Component
public class MailService {
    @Value("${app.mail.host}")                    // exact key
    private String host;

    @Value("${app.mail.port:25}")                 // default if missing
    private int port;

    @Value("${app.mail.timeout:#{T(java.time.Duration).ofSeconds(30)}}")  // SpEL default
    private Duration timeout;

    @Value("#{${app.feature.map}}")               // SpEL: inline map
    private Map<String, String> features;
}
```

| Feature                         | `@ConfigurationProperties`          | `@Value`                          |
|---------------------------------|-------------------------------------|-----------------------------------|
| Binding style                   | Bulk — entire prefix tree           | Individual property               |
| Type safety                     | Full — validated at startup         | Limited — runtime errors          |
| Relaxed binding                 | Yes (`app-mail-host` = `app.mail.host`) | **No** — exact key match only  |
| Metadata / IDE support          | Via `spring-boot-configuration-processor` | None                        |
| Validation (`@Validated`)       | Supported (JSR-380)                 | Not supported                     |
| SpEL expressions                | **Not supported**                   | Supported                         |
| Complex types (List, Map, Duration) | Native support                 | Requires SpEL workarounds         |
| Immutable binding (records)     | `@ConstructorBinding` in Boot 3.x  | N/A                               |

```java
// Immutable config with records (Spring Boot 3.x)
@ConfigurationProperties(prefix = "app.mail")
public record MailProperties(
    String host,
    int port,
    List<String> recipients,
    Duration timeout
) { }

// Enable in main config
@SpringBootApplication
@EnableConfigurationProperties(MailProperties.class)
public class MyApp { }
```

**Interview Trap:** "Can you use SpEL in `@ConfigurationProperties`?"  
→ **No.** SpEL is only available with `@Value`. `@ConfigurationProperties` uses **relaxed binding** and type conversion, which is a different mechanism.

**Relaxed binding examples:**

```
app.mail.host           # standard
app.mail-host           # kebab-case
APP_MAIL_HOST           # environment variable
app.mailHost            # camelCase
```

All four resolve to the same `host` field in `@ConfigurationProperties`.

---

### Q23. Explain the Spring Boot externalized configuration precedence (from highest to lowest).

**Answer:**

Spring Boot loads properties in a **specific order** — higher precedence overrides lower:

```
1.  Command-line arguments (--server.port=9090)
2.  SPRING_APPLICATION_JSON (inline JSON in env var or system property)
3.  ServletConfig/ServletContext parameters
4.  JNDI attributes
5.  Java system properties (System.getProperties())
6.  OS environment variables
7.  Profile-specific application-{profile}.properties (outside JAR)
8.  Profile-specific application-{profile}.properties (inside JAR)
9.  application.properties (outside JAR)
10. application.properties (inside JAR)
11. @PropertySource annotations on @Configuration classes
12. Default properties (SpringApplication.setDefaultProperties())
```

**Simplified priority for interviews:**

```
Command-line args
  > OS environment variables
    > Profile-specific external config
      > Profile-specific internal config
        > External application.properties
          > Internal application.properties
            > @PropertySource
              > Defaults
```

```bash
# Command-line always wins
java -jar myapp.jar --server.port=9090

# Environment variable (relaxed binding: dots → underscores, uppercase)
export SERVER_PORT=9090

# External config file (next to JAR)
# ./config/application.properties — auto-detected
```

**Interview Trap:** "Where does `@PropertySource` fall in the precedence order?"  
→ Very **low priority** — near the bottom. This is a common mistake: developers assume `@PropertySource("classpath:custom.properties")` will override `application.properties`, but it's the opposite.

**Spring Boot 3.x Config Locations:**

```
file:./config/          ← highest (external, /config subdirectory)
file:./                 ← external, same directory as JAR
classpath:/config/      ← internal, /config package
classpath:/             ← internal, root (lowest)
```

---

### Q24. How do Spring profiles work?

**Answer:**

Profiles allow **environment-specific configuration** without code changes:

```java
// Activate via property
// application.properties
spring.profiles.active=dev,metrics

// Activate via command line
// java -jar myapp.jar --spring.profiles.active=prod

// Activate programmatically
SpringApplication app = new SpringApplication(MyApp.class);
app.setAdditionalProfiles("dev");
app.run(args);
```

```java
// Conditional beans based on profile
@Configuration
@Profile("dev")
public class DevConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder().setType(EmbeddedDatabaseType.H2).build();
    }
}

@Configuration
@Profile("prod")
public class ProdConfig {
    @Bean
    public DataSource dataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://prod-host:5432/db")
            .build();
    }
}

// Negation — active when profile is NOT active
@Component
@Profile("!prod")
public class MockPaymentGateway implements PaymentGateway { }
```

**Profile-specific files:**

```
application.properties          ← always loaded (defaults)
application-dev.properties      ← loaded when "dev" profile active
application-prod.properties     ← loaded when "prod" profile active
```

**Profile groups (Spring Boot 2.4+):**

```properties
# application.properties
spring.profiles.group.local=dev,h2,mock-services
spring.profiles.group.staging=qa,postgresql,real-services

# Activating "local" activates dev + h2 + mock-services
spring.profiles.active=local
```

**Interview Trap:** "What's the difference between `spring.profiles.active` and `spring.profiles.default`?"  
→ `active` **explicitly** sets profiles. `default` specifies what profile to use **only if no active profile is set**. If `active` is specified, `default` is completely ignored.

```properties
# Default profile (used when nothing else is set)
spring.profiles.default=dev
```

**Edge Case:** Profile-specific properties always override non-profile properties, **regardless** of file location. `application-dev.properties` inside the JAR overrides `application.properties` outside the JAR — for the same key.

---

### Q25. How do you create a custom `PropertySource`?

**Answer:**

```java
// Option 1: @PropertySource annotation (static resource)
@Configuration
@PropertySource("classpath:custom-config.properties")
@PropertySource("classpath:${app.config.path:default-config}.properties") // with placeholder
public class CustomPropertyConfig { }

// Option 2: EnvironmentPostProcessor (programmatic, loaded very early)
public class VaultPropertySourcePostProcessor implements EnvironmentPostProcessor {

    @Override
    public void postProcessEnvironment(ConfigurableEnvironment env,
                                        SpringApplication app) {
        Map<String, Object> vaultProperties = loadFromVault();
        env.getPropertySources().addFirst(  // addFirst = highest priority
            new MapPropertySource("vault", vaultProperties)
        );
    }

    private Map<String, Object> loadFromVault() {
        // Load secrets from Vault, AWS Secrets Manager, etc.
        return Map.of("db.password", "secret123");
    }
}
```

Register the `EnvironmentPostProcessor` in:

```text
# META-INF/spring.factories (Boot 2.x)
org.springframework.boot.env.EnvironmentPostProcessor=\
  com.example.VaultPropertySourcePostProcessor

# META-INF/spring/org.springframework.boot.env.EnvironmentPostProcessor.imports (Boot 3.x)
com.example.VaultPropertySourcePostProcessor
```

**Interview Trap:** "When does an `EnvironmentPostProcessor` run relative to bean creation?"  
→ **Before** the ApplicationContext is created and refreshed. This is one of the earliest extension points — you can inject properties before any `@ConfigurationProperties` or `@Value` binding occurs.

---

### Q26. Explain `@PropertySource` limitations and alternatives.

**Answer:**

| Limitation of `@PropertySource`                  | Alternative                                 |
|--------------------------------------------------|---------------------------------------------|
| Cannot load YAML files                           | Use `YamlPropertySourceLoader` or `EnvironmentPostProcessor` |
| Low priority (easily overridden)                 | Use `EnvironmentPostProcessor` with `addFirst()` |
| Static — no dynamic refresh                      | Use `@RefreshScope` (Spring Cloud) or custom `PropertySource` |
| Only works on `@Configuration` classes           | `EnvironmentPostProcessor` works globally   |
| No support for encrypted values                  | Use Jasypt or custom `PropertySource`       |

```java
// Loading YAML as a PropertySource
public class YamlPropertyLoader implements EnvironmentPostProcessor {
    @Override
    public void postProcessEnvironment(ConfigurableEnvironment env,
                                        SpringApplication app) {
        YamlPropertySourceLoader loader = new YamlPropertySourceLoader();
        try {
            List<PropertySource<?>> sources = loader.load("custom",
                new ClassPathResource("custom-config.yml"));
            sources.forEach(env.getPropertySources()::addLast);
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    }
}
```

---

### Q27. How does `@ConfigurationProperties` validation work?

**Answer:**

```java
@ConfigurationProperties(prefix = "app.database")
@Validated  // ← required to activate JSR-380 validation
public class DatabaseProperties {

    @NotBlank(message = "Database URL must be provided")
    private String url;

    @Min(1)
    @Max(200)
    private int maxPoolSize = 10;

    @DurationMin(seconds = 1)
    @DurationMax(minutes = 5)
    private Duration connectionTimeout = Duration.ofSeconds(30);

    @Valid  // ← required for nested object validation
    private Pool pool = new Pool();

    public static class Pool {
        @Min(1)
        private int minIdle = 2;

        @Min(1)
        private int maxActive = 10;

        // getters + setters
    }

    // getters + setters
}
```

```properties
# application.properties
app.database.url=jdbc:postgresql://localhost/mydb
app.database.max-pool-size=50
app.database.connection-timeout=30s
app.database.pool.min-idle=5
app.database.pool.max-active=20
```

**Interview Trap:** "What annotation is needed on the class for validation to work?"  
→ `@Validated` on the configuration class **and** `@Valid` on any **nested objects** that also have constraints. Without `@Valid` on nested fields, their constraints are silently ignored.

**Startup failure message example:**

```
***************************
APPLICATION FAILED TO START
***************************
Binding to target com.example.DatabaseProperties failed:

    Property: app.database.url
    Value: null
    Reason: Database URL must be provided
```

---

### Q28. How do you use config data with Spring Boot 3.x `spring.config.import`?

**Answer:**

`spring.config.import` (introduced in Boot 2.4) replaces the legacy `spring.config.location` and `spring.config.additional-location`:

```properties
# application.properties
spring.config.import=optional:file:./config/extra.properties,\
                     optional:classpath:overrides.properties,\
                     configtree:/etc/secrets/

# "optional:" prefix → don't fail if file doesn't exist
# "configtree:" → maps directory structure to properties
#   /etc/secrets/db/password → db.password=<file contents>
```

**Config tree (Kubernetes secrets):**

```
/etc/secrets/
  ├── db/
  │   ├── password     → contains "s3cret"
  │   └── username     → contains "admin"
  └── api/
      └── key          → contains "abc123"

# Maps to:
db.password=s3cret
db.username=admin
api.key=abc123
```

```java
@ConfigurationProperties(prefix = "db")
public class DbConfig {
    private String username;   // "admin"
    private String password;   // "s3cret"
}
```

**Interview Trap:** "What's the difference between `spring.config.import` and `spring.config.additional-location`?"  
→ `spring.config.import` is the **modern approach** (Boot 2.4+) and supports pluggable config data backends (Vault, Consul, K8s ConfigMaps). `additional-location` is legacy and doesn't support custom importers.

---

## 4. Starters & Dependencies

---

### Q29. How do Spring Boot starters work internally?

**Answer:**

A starter is simply a **Maven/Gradle dependency that bundles** a curated set of transitive dependencies + optionally auto-configuration.

Starter anatomy:

```
spring-boot-starter-web
  ├── spring-boot-starter (core)
  │     ├── spring-boot
  │     ├── spring-boot-autoconfigure
  │     ├── spring-core, spring-context
  │     ├── spring-boot-starter-logging (Logback)
  │     └── snakeyaml
  ├── spring-web
  ├── spring-webmvc
  ├── jackson-databind
  └── spring-boot-starter-tomcat (embedded server)
        ├── tomcat-embed-core
        ├── tomcat-embed-el
        └── tomcat-embed-websocket
```

**Key insight:** Starters typically have **no code** — only a `pom.xml` with dependency declarations. Auto-configuration classes live in `spring-boot-autoconfigure` and are activated when starter dependencies appear on the classpath.

```
User adds spring-boot-starter-data-jpa
  → Pulls in Hibernate, Spring Data JPA, HikariCP, JDBC
  → Auto-configuration detects these classes
  → @ConditionalOnClass(EntityManagerFactory.class) passes
  → DataSource, EntityManagerFactory, TransactionManager auto-configured
```

**Interview Trap:** "Does a starter contain auto-configuration code?"  
→ Generally **no**. Official starters are dependency aggregators only. Auto-configuration lives in `spring-boot-autoconfigure`. However, **custom/third-party starters** typically include both the dependencies and auto-configuration in one module (or a companion autoconfigure module).

---

### Q30. What is `spring-boot-starter-parent` and is it required?

**Answer:**

`spring-boot-starter-parent` is a parent POM that provides:

1. **Dependency management** — pre-defined versions for hundreds of libraries
2. **Plugin configuration** — maven-compiler, maven-surefire, spring-boot-maven-plugin
3. **Resource filtering** — replaces `@...@` tokens in properties files
4. **Default Java version** — Java 17 for Boot 3.x
5. **UTF-8 encoding** as default

```xml
<!-- Standard approach: inherit from starter-parent -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.0</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <!-- NO version needed — managed by parent -->
    </dependency>
</dependencies>
```

**Alternative without parent (for corporate POMs):**

```xml
<!-- Use dependency management BOM instead -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.3.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- Must configure plugins manually -->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>3.3.0</version>
        </plugin>
    </plugins>
</build>
```

| Feature                      | With `starter-parent`    | With BOM only (`spring-boot-dependencies`) |
|------------------------------|--------------------------|--------------------------------------------|
| Dependency versions managed  | Yes                      | Yes                                        |
| Plugin configuration         | Auto-configured          | Must configure manually                    |
| Resource filtering           | Enabled                  | Must configure manually                    |
| Parent POM slot              | Occupied                 | Free for corporate parent POM              |
| Override dependency version  | `<properties>` section   | `<dependencyManagement>` section           |

**Interview Trap:** "How do you override a managed dependency version?"

```xml
<!-- With starter-parent: just override the property -->
<properties>
    <jackson.version>2.17.0</jackson.version>
</properties>

<!-- With BOM: re-declare in your own dependencyManagement -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.17.0</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

### Q31. How do you create a custom Spring Boot starter?

**Answer:**

A custom starter follows a **two-module convention**:

```
my-spring-boot-starter/
├── my-spring-boot-starter/           ← starter module (dependencies only)
│   └── pom.xml
└── my-spring-boot-starter-autoconfigure/  ← auto-config + logic
    ├── src/main/java/
    │   └── com/example/
    │       ├── MyAutoConfiguration.java
    │       ├── MyService.java
    │       └── MyProperties.java
    └── src/main/resources/
        └── META-INF/spring/
            └── org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

**Step 1: Properties class**

```java
@ConfigurationProperties(prefix = "my.service")
public class MyProperties {
    private String apiKey;
    private Duration timeout = Duration.ofSeconds(30);
    private boolean enabled = true;
    // getters + setters
}
```

**Step 2: Service class**

```java
public class MyService {
    private final MyProperties properties;

    public MyService(MyProperties properties) {
        this.properties = properties;
    }

    public String callApi(String input) {
        // uses properties.getApiKey(), properties.getTimeout()
        return "result";
    }
}
```

**Step 3: Auto-configuration**

```java
@AutoConfiguration   // Spring Boot 3.x annotation
@ConditionalOnClass(MyService.class)
@ConditionalOnProperty(prefix = "my.service", name = "enabled", havingValue = "true",
                       matchIfMissing = true)
@EnableConfigurationProperties(MyProperties.class)
public class MyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyProperties properties) {
        return new MyService(properties);
    }
}
```

**Step 4: Register in `.imports`**

```text
# META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
com.example.MyAutoConfiguration
```

**Step 5: Starter module `pom.xml`**

```xml
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>my-spring-boot-starter-autoconfigure</artifactId>
        <version>${project.version}</version>
    </dependency>
    <!-- add any mandatory transitive dependencies -->
</dependencies>
```

**Naming convention:**
- Official: `spring-boot-starter-{name}` (e.g., `spring-boot-starter-web`)
- Custom: `{name}-spring-boot-starter` (e.g., `my-spring-boot-starter`)

**Interview Trap:** "Why separate autoconfigure and starter modules?"  
→ Separation allows users who only want auto-configuration (without all transitive dependencies) to import the autoconfigure module alone. The starter is a convenience wrapper. In practice, simple starters often combine both in one module.

---

### Q32. Explain the Spring Boot dependency management mechanism.

**Answer:**

Spring Boot manages 800+ dependency versions through a BOM (Bill of Materials) hierarchy:

```
spring-boot-starter-parent
  └── spring-boot-dependencies (BOM)
        ├── Spring Framework 6.x
        ├── Hibernate 6.x
        ├── Jackson 2.x
        ├── HikariCP
        ├── Logback
        ├── JUnit 5
        ├── Tomcat 10.x
        └── ... 800+ managed versions
```

**How it works:** Maven's `<dependencyManagement>` section declares versions without adding dependencies. When you add a dependency **without** a version, Maven looks up the tree to find a managed version.

```xml
<!-- You write: (no version) -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>

<!-- Maven resolves to version from spring-boot-dependencies BOM -->
<!-- e.g., jackson-databind 2.17.0 -->
```

**Interview Trap:** "What if you want a different version of a managed dependency?"  
→ You can override by declaring the version explicitly or by overriding the version property. **Be careful:** changing one library's version may cause compatibility issues with other managed dependencies.

---

### Q33. How does Spring Boot handle embedded server dependencies?

**Answer:**

```xml
<!-- spring-boot-starter-web includes Tomcat by default -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Switch to Jetty -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>

<!-- Or Undertow -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

| Server    | Starter                       | Key Feature                        |
|-----------|-------------------------------|------------------------------------|
| Tomcat    | `starter-tomcat` (default)    | Most widely used, mature           |
| Jetty     | `starter-jetty`               | Lightweight, good for async        |
| Undertow  | `starter-undertow`            | High performance, non-blocking     |

**Auto-configuration mechanism:**

```java
// Spring Boot checks which server is on classpath
@ConditionalOnClass({ Servlet.class, Tomcat.class, UpgradeProtocol.class })
public class TomcatServletWebServerFactoryAutoConfiguration { }

@ConditionalOnClass({ Servlet.class, Server.class, Loader.class })
public class JettyServletWebServerFactoryAutoConfiguration { }
```

**Interview Trap:** "Can you run Spring Boot without an embedded server?"  
→ Yes. Use `spring.main.web-application-type=none` or don't include any web starter. The `ApplicationContext` will be `AnnotationConfigApplicationContext` instead of a web context.

---

## 5. Auto-Configuration Deep Dive

---

### Q34. How does `@EnableAutoConfiguration` trigger auto-configuration?

**Answer:**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(AutoConfigurationImportSelector.class)  // ← the key
public @interface EnableAutoConfiguration {
    Class<?>[] exclude() default {};
    String[] excludeName() default {};
}
```

`AutoConfigurationImportSelector` implements `DeferredImportSelector`:

```
1. selectImports() is called by Spring's configuration processing
2. Reads candidate class names from:
   - META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports (Boot 3.x)
   - META-INF/spring.factories (Boot 2.x, still supported for backward compat)
3. Applies exclusions (from annotation + property)
4. Filters candidates using AutoConfigurationImportFilter
   - OnBeanCondition
   - OnClassCondition       ← fastest filter, checked first
   - OnWebApplicationCondition
5. Sorts by @AutoConfigureOrder, @AutoConfigureBefore, @AutoConfigureAfter
6. Returns the list of configuration classes to import
```

**Why `DeferredImportSelector` instead of `ImportSelector`?**  
→ Deferred selectors run **after** all regular `@Configuration` classes are processed. This ensures user-defined beans exist first, so `@ConditionalOnMissingBean` checks work correctly.

---

### Q35. Explain the `spring.factories` mechanism and the new `.imports` file format.

**Answer:**

**Legacy (`spring.factories` — Boot 2.x):**

```properties
# META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.example.FooAutoConfiguration,\
  com.example.BarAutoConfiguration

# Also used for other extension points:
org.springframework.context.ApplicationContextInitializer=\
  com.example.MyContextInitializer
org.springframework.boot.env.EnvironmentPostProcessor=\
  com.example.MyEnvProcessor
```

**New format (Boot 3.x — for auto-configuration only):**

```text
# META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
com.example.FooAutoConfiguration
com.example.BarAutoConfiguration
# One class per line, supports # comments
```

| Aspect               | `spring.factories`                    | `.imports` file                         |
|----------------------|---------------------------------------|-----------------------------------------|
| Format               | Key=value, comma-separated classes    | One class per line                      |
| Scope                | All extension points                  | Auto-configuration only                 |
| Performance          | All entries parsed even if unused     | Only auto-config entries loaded         |
| Boot version         | 2.x (still loaded in 3.x)            | 3.x+ preferred                          |
| Comments             | Not supported                         | `#` line comments supported             |

**Interview Trap:** "Is `spring.factories` completely gone in Boot 3?"  
→ No. For auto-configuration, the `.imports` format is preferred and `spring.factories` is deprecated for that key. But `spring.factories` is still the mechanism for other SPIs like `ApplicationContextInitializer`, `FailureAnalyzer`, etc.

---

### Q36. How do you debug auto-configuration decisions?

**Answer:**

**Method 1: `--debug` flag** (most common interview answer)

```bash
java -jar myapp.jar --debug
# OR
java -jar myapp.jar -Ddebug

# OR in application.properties
debug=true
```

This outputs the **Conditions Evaluation Report** at startup:

```
============================
CONDITIONS EVALUATION REPORT
============================

Positive matches:
-----------------
  DataSourceAutoConfiguration matched:
    - @ConditionalOnClass found required classes 'javax.sql.DataSource',
      'org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType'
    - @ConditionalOnMissingBean did not find any beans of type
      'io.r2dbc.spi.ConnectionFactory'

Negative matches:
-----------------
  ActiveMQAutoConfiguration:
    Did not match:
      - @ConditionalOnClass did not find required class
        'jakarta.jms.ConnectionFactory'

Exclusions:
-----------
  org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration

Unconditional classes:
----------------------
  org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration
```

**Method 2: Actuator `/conditions` endpoint**

```properties
management.endpoints.web.exposure.include=conditions
```

```bash
curl http://localhost:8080/actuator/conditions | jq
```

**Method 3: Programmatic**

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        ConfigurableApplicationContext ctx = SpringApplication.run(MyApp.class, args);
        ConditionEvaluationReport report =
            ConditionEvaluationReport.get(ctx.getBeanFactory());
        report.getConditionAndOutcomesBySource().forEach((source, outcomes) -> {
            System.out.println(source + " → " + outcomes.isFullMatch());
        });
    }
}
```

**Interview Trap:** "What's the difference between `--debug` and `--trace`?"  
→ `--debug` shows the auto-configuration conditions report. `--trace` shows **everything** `--debug` shows plus full startup trace logging (very verbose).

---

### Q37. How do `@AutoConfigureBefore` and `@AutoConfigureAfter` work?

**Answer:**

These annotations control the **processing order** of auto-configuration classes:

```java
@AutoConfiguration(after = DataSourceAutoConfiguration.class)
@ConditionalOnBean(DataSource.class)
public class MyJdbcAutoConfiguration {
    // Guaranteed to be processed AFTER DataSourceAutoConfiguration
    // so @ConditionalOnBean(DataSource.class) works correctly
}

@AutoConfiguration(before = WebMvcAutoConfiguration.class)
public class MyWebAutoConfiguration {
    // Processed BEFORE WebMvcAutoConfiguration — can define beans
    // that WebMvcAutoConfiguration's @ConditionalOnMissingBean will see
}
```

**Spring Boot 3.x `@AutoConfiguration` annotation:**

```java
// Combines @Configuration + ordering in one annotation
@AutoConfiguration(
    after = { DataSourceAutoConfiguration.class },
    before = { FlywayAutoConfiguration.class }
)
public class MyAutoConfiguration { }

// Equivalent in Boot 2.x:
@Configuration
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
@AutoConfigureBefore(FlywayAutoConfiguration.class)
public class MyAutoConfiguration { }
```

**Interview Trap:** "Does ordering guarantee that one bean exists before another?"  
→ Ordering guarantees **processing order of configurations**, not bean instantiation order. A `@Bean` in an "after" configuration can still be created before a bean in a "before" configuration if the container optimizes instantiation. Use `@DependsOn` for strict bean creation ordering.

---

### Q38. How do you write a custom `@Conditional` annotation?

**Answer:**

```java
// Step 1: Create the annotation
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Conditional(OnLinuxCondition.class)
public @interface ConditionalOnLinux { }

// Step 2: Implement the Condition interface
public class OnLinuxCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String os = context.getEnvironment().getProperty("os.name", "");
        return os.toLowerCase().contains("linux");
    }
}

// Step 3: Use it
@Configuration
@ConditionalOnLinux
public class LinuxSpecificConfig {
    @Bean
    public FileWatcher linuxFileWatcher() {
        return new InotifyFileWatcher();
    }
}
```

**Advanced: Using `SpringBootCondition` (better error messages):**

```java
public class OnFeatureFlagCondition extends SpringBootCondition {

    @Override
    public ConditionOutcome getMatchOutcome(ConditionContext context,
                                             AnnotatedTypeMetadata metadata) {
        Map<String, Object> attrs = metadata.getAnnotationAttributes(
            ConditionalOnFeatureFlag.class.getName());
        String flag = (String) attrs.get("value");
        boolean enabled = Boolean.parseBoolean(
            context.getEnvironment().getProperty("features." + flag, "false"));

        if (enabled) {
            return ConditionOutcome.match("Feature flag '" + flag + "' is enabled");
        }
        return ConditionOutcome.noMatch("Feature flag '" + flag + "' is disabled");
    }
}
```

**Interview Trap:** "When are conditions evaluated — at compile time or runtime?"  
→ **Runtime**, during ApplicationContext refresh. Conditions have access to the `Environment`, `BeanFactory`, `ClassLoader`, and `ResourceLoader` through `ConditionContext`.

---

### Q39. What is `@ConditionalOnProperty` and its `matchIfMissing` pitfall?

**Answer:**

```java
@Bean
@ConditionalOnProperty(
    prefix = "app.cache",
    name = "enabled",
    havingValue = "true",
    matchIfMissing = false    // DEFAULT: false → bean NOT created if property is absent
)
public CacheManager cacheManager() { }

@Bean
@ConditionalOnProperty(
    prefix = "app.cache",
    name = "enabled",
    havingValue = "true",
    matchIfMissing = true     // bean IS created if property is absent (opt-out model)
)
public CacheManager cacheManagerOptOut() { }
```

| `matchIfMissing` | Property Absent | Property = "true" | Property = "false" |
|-------------------|-----------------|--------------------|--------------------|
| `false` (default) | ❌ No match     | ✅ Match           | ❌ No match        |
| `true`            | ✅ Match        | ✅ Match           | ❌ No match        |

**Opt-in vs. Opt-out models:**

```java
// Opt-in: Feature disabled by default, user must enable
@ConditionalOnProperty(name = "app.feature.x", havingValue = "true")
// matchIfMissing = false (default) → feature OFF unless explicitly enabled

// Opt-out: Feature enabled by default, user can disable
@ConditionalOnProperty(name = "app.feature.x", havingValue = "true",
                       matchIfMissing = true)
// → feature ON unless user sets app.feature.x=false
```

**Interview Trap:** "What if you set `havingValue` but not `matchIfMissing`, and the property doesn't exist?"  
→ The condition does **not match**. This is the most common pitfall — developers expect the feature to be "on by default" but forget to set `matchIfMissing = true`.

---

### Q40. How does auto-configuration ordering interact with user configuration?

**Answer:**

The critical principle: **User configuration is always processed before auto-configuration.**

```
Phase 1: User's @Configuration classes (from @ComponentScan)
  ↓ all user @Bean methods registered
Phase 2: Auto-configuration classes (via @EnableAutoConfiguration)
  ↓ @ConditionalOnMissingBean checks find user's beans → skips auto-config beans
```

This is why auto-configuration "backs off" when you define your own beans:

```java
// Your explicit configuration
@Configuration
public class MyConfig {
    @Bean
    public DataSource dataSource() {
        return new CustomDataSource(); // registered in Phase 1
    }
}

// Spring Boot's auto-configuration (Phase 2)
@AutoConfiguration
public class DataSourceAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean  // finds user's DataSource → SKIPS
    public DataSource dataSource() {
        return new HikariDataSource(); // never created
    }
}
```

**Interview Trap:** "What if you put your `@Configuration` class inside `@AutoConfiguration`?"  
→ If you register your class via `.imports` file, it becomes an auto-configuration and follows auto-configuration ordering rules. It will be processed **after** user configurations, which may cause your `@ConditionalOnMissingBean` checks to find user beans and skip.

---

## 6. Actuator — Core Endpoints

---

### Q41. What are the core Actuator endpoints and how do you configure them?

**Answer:**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

| Endpoint       | Path                 | Purpose                            | Exposed by Default (Web) |
|----------------|----------------------|------------------------------------|--------------------------|
| `health`       | `/actuator/health`   | Application health status          | Yes                      |
| `info`         | `/actuator/info`     | Application metadata               | Yes                      |
| `metrics`      | `/actuator/metrics`  | Application metrics (Micrometer)   | No                       |
| `env`          | `/actuator/env`      | Configuration properties           | No                       |
| `conditions`   | `/actuator/conditions`| Auto-config report                | No                       |
| `beans`        | `/actuator/beans`    | All beans in context               | No                       |
| `configprops`  | `/actuator/configprops` | `@ConfigurationProperties` dump | No                       |
| `mappings`     | `/actuator/mappings` | Request mappings                   | No                       |
| `loggers`      | `/actuator/loggers`  | Logger levels (read + write)       | No                       |
| `threaddump`   | `/actuator/threaddump` | JVM thread dump                  | No                       |
| `heapdump`     | `/actuator/heapdump` | Heap dump (hprof)                  | No                       |
| `shutdown`     | `/actuator/shutdown` | Graceful shutdown (POST)           | **No — disabled**        |

```properties
# Expose specific endpoints
management.endpoints.web.exposure.include=health,info,metrics,env

# Expose all
management.endpoints.web.exposure.include=*

# Exclude sensitive ones
management.endpoints.web.exposure.exclude=heapdump,threaddump

# Enable shutdown endpoint (disabled by default)
management.endpoint.shutdown.enabled=true

# Change base path
management.endpoints.web.base-path=/manage

# Run actuator on a different port
management.server.port=9090
```

**Interview Trap:** "What's the difference between `enabled` and `exposed`?"  
→ `enabled` controls whether the endpoint **exists** at all. `exposed` controls whether it's **accessible** via web/JMX. An endpoint must be both enabled AND exposed to be reachable.

```properties
# Endpoint exists but not web-accessible
management.endpoint.beans.enabled=true
management.endpoints.web.exposure.include=health   # beans not listed → not exposed

# Endpoint doesn't exist at all
management.endpoint.beans.enabled=false
```

---

### Q42. How do you create a custom health indicator?

**Answer:**

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {

    private final DataSource dataSource;

    public DatabaseHealthIndicator(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            if (conn.isValid(2)) {
                return Health.up()
                    .withDetail("database", "PostgreSQL")
                    .withDetail("validationQuery", "connection.isValid()")
                    .withDetail("maxPoolSize", 20)
                    .build();
            }
        } catch (SQLException e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .withException(e)
                .build();
        }
        return Health.unknown().build();
    }
}
```

**Response for `/actuator/health`:**

```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP",
      "details": {
        "database": "PostgreSQL",
        "validationQuery": "connection.isValid()",
        "maxPoolSize": 20
      }
    },
    "diskSpace": { "status": "UP" },
    "ping": { "status": "UP" }
  }
}
```

```properties
# Show full details (default shows only status)
management.endpoint.health.show-details=always    # always | when-authorized | never
management.endpoint.health.show-components=always

# Health indicator for groups (e.g., k8s liveness vs readiness)
management.endpoint.health.group.liveness.include=ping
management.endpoint.health.group.readiness.include=db,redis,diskSpace
```

**Health status aggregation:** The overall status is determined by the **worst** status using `StatusAggregator`:

```
UP < UNKNOWN < OUT_OF_SERVICE < DOWN
```

If any component is `DOWN`, the aggregate is `DOWN`.

---

### Q43. How do you create a custom Actuator endpoint?

**Answer:**

```java
@Component
@Endpoint(id = "features")
public class FeatureEndpoint {

    private final Map<String, Boolean> features = new ConcurrentHashMap<>(Map.of(
        "darkMode", true,
        "newCheckout", false
    ));

    // GET /actuator/features
    @ReadOperation
    public Map<String, Boolean> getAllFeatures() {
        return Collections.unmodifiableMap(features);
    }

    // GET /actuator/features/{name}
    @ReadOperation
    public Boolean getFeature(@Selector String name) {
        return features.get(name);
    }

    // POST /actuator/features/{name}
    @WriteOperation
    public void setFeature(@Selector String name, boolean enabled) {
        features.put(name, enabled);
    }

    // DELETE /actuator/features/{name}
    @DeleteOperation
    public void deleteFeature(@Selector String name) {
        features.remove(name);
    }
}
```

```properties
# Expose the custom endpoint
management.endpoints.web.exposure.include=health,info,features
```

| Annotation         | HTTP Method | Purpose              |
|--------------------|-------------|----------------------|
| `@ReadOperation`   | GET         | Read data            |
| `@WriteOperation`  | POST        | Create/update data   |
| `@DeleteOperation` | DELETE      | Remove data          |

**Interview Trap:** "`@Endpoint` vs `@WebEndpoint` vs `@ControllerEndpoint`?"  
→ `@Endpoint` exposes via **both** JMX and Web. `@WebEndpoint` exposes via **Web only**. `@ControllerEndpoint` (deprecated in Boot 3.2+) allowed full Spring MVC integration.

---

### Q44. How does the `/metrics` endpoint work with Micrometer?

**Answer:**

Spring Boot Actuator uses **Micrometer** as its metrics facade (like SLF4J for logging):

```bash
# List all available metrics
GET /actuator/metrics
{
  "names": ["jvm.memory.used", "http.server.requests", "process.cpu.usage", ...]
}

# Get specific metric with tags
GET /actuator/metrics/http.server.requests?tag=uri:/api/users&tag=status:200
```

**Custom metrics:**

```java
@Service
public class OrderService {
    private final Counter orderCounter;
    private final Timer orderProcessingTimer;
    private final AtomicInteger activeOrders;

    public OrderService(MeterRegistry registry) {
        // Counter — tracks cumulative count
        this.orderCounter = Counter.builder("orders.placed")
            .description("Total orders placed")
            .tag("region", "us-east")
            .register(registry);

        // Timer — tracks duration and count
        this.orderProcessingTimer = Timer.builder("orders.processing.time")
            .description("Order processing duration")
            .register(registry);

        // Gauge — tracks current value
        this.activeOrders = registry.gauge("orders.active",
            new AtomicInteger(0));
    }

    public void placeOrder(Order order) {
        orderCounter.increment();
        activeOrders.incrementAndGet();

        orderProcessingTimer.record(() -> {
            // process order
            processInternal(order);
        });

        activeOrders.decrementAndGet();
    }
}
```

| Meter Type      | Use Case                          | Example                          |
|-----------------|-----------------------------------|----------------------------------|
| `Counter`       | Monotonically increasing count    | Total requests, errors           |
| `Gauge`         | Current value (goes up/down)      | Active connections, queue size   |
| `Timer`         | Duration + count                  | Request latency, DB query time   |
| `DistributionSummary` | Distribution of values      | Payload sizes, batch sizes       |

**Interview Trap:** "What Micrometer registries does Spring Boot support?"  
→ Prometheus, Datadog, New Relic, InfluxDB, Graphite, Elastic, CloudWatch, and more. Just add the `micrometer-registry-{system}` dependency and auto-configuration handles the rest.

---

## 7. Spring Boot Internals

---

### Q45. How do you customize the `SpringApplication` class?

**Answer:**

```java
public class MyApp {
    public static void main(String[] args) {
        // Method 1: Direct customization
        SpringApplication app = new SpringApplication(MyApp.class);
        app.setBannerMode(Banner.Mode.OFF);
        app.setDefaultProperties(Map.of(
            "server.port", "9090",
            "spring.main.lazy-initialization", "true"
        ));
        app.setAdditionalProfiles("dev");
        app.setWebApplicationType(WebApplicationType.SERVLET);
        app.addListeners(new MyCustomListener());
        app.addInitializers(new MyContextInitializer());
        app.run(args);

        // Method 2: Fluent builder
        new SpringApplicationBuilder(MyApp.class)
            .bannerMode(Banner.Mode.OFF)
            .profiles("dev")
            .properties("server.port=9090")
            .listeners(new MyCustomListener())
            .initializers(new MyContextInitializer())
            .lazyInitialization(true)
            .logStartupInfo(false)
            .run(args);
    }
}
```

**`WebApplicationType` options:**

| Value      | ApplicationContext                              | Use Case                 |
|------------|-------------------------------------------------|--------------------------|
| `SERVLET`  | `AnnotationConfigServletWebServerApplicationContext` | Traditional web apps  |
| `REACTIVE` | `AnnotationConfigReactiveWebServerApplicationContext` | WebFlux apps         |
| `NONE`     | `AnnotationConfigApplicationContext`            | CLI tools, batch jobs    |

**Auto-detection logic:**
1. If Spring WebFlux is on classpath (and Servlet is not) → `REACTIVE`
2. If Servlet API is on classpath → `SERVLET`
3. Otherwise → `NONE`

```java
// Force non-web even with spring-boot-starter-web
app.setWebApplicationType(WebApplicationType.NONE);
```

---

### Q46. Compare `CommandLineRunner` vs. `ApplicationRunner`.

**Answer:**

Both run **after** the ApplicationContext is fully refreshed but **before** `ApplicationReadyEvent`:

```java
@Component
@Order(1)  // lower value = higher priority
public class DatabaseSeeder implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        // args = raw String array from command line
        // e.g., java -jar app.jar arg1 arg2 --name=value
        System.out.println("Raw args: " + Arrays.toString(args));
    }
}

@Component
@Order(2)
public class CacheWarmer implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // Parsed arguments with rich API
        List<String> nonOptionArgs = args.getNonOptionArgs();      // [arg1, arg2]
        boolean hasName = args.containsOption("name");              // true
        List<String> nameValues = args.getOptionValues("name");     // [value]
        String[] sourceArgs = args.getSourceArgs();                 // raw array
    }
}
```

| Feature                    | `CommandLineRunner`       | `ApplicationRunner`         |
|----------------------------|---------------------------|-----------------------------|
| Parameter type             | `String... args` (raw)    | `ApplicationArguments` (parsed) |
| Option parsing             | Manual                    | Built-in (`--key=value`)   |
| Non-option args            | Manual split              | `getNonOptionArgs()`       |
| Interface method           | `run(String...)`          | `run(ApplicationArguments)` |
| `@Order` / `Ordered`       | Supported                 | Supported                  |

**Interview Trap:** "What happens if a Runner throws an exception?"  
→ The application **fails to start**. `ApplicationReadyEvent` is **not published** — instead, `ApplicationFailedEvent` fires and the context is closed.

```java
// Using @Bean instead of @Component — allows multiple runners per class
@Configuration
public class RunnerConfig {

    @Bean
    @Order(1)
    public CommandLineRunner initDatabase(UserRepository repo) {
        return args -> {
            repo.save(new User("admin"));
        };
    }

    @Bean
    @Order(2)
    public ApplicationRunner warmCaches(CacheManager cacheManager) {
        return args -> {
            cacheManager.getCache("users").put("admin", loadAdmin());
        };
    }
}
```

---

### Q47. How do application events and listeners work in Spring Boot?

**Answer:**

**Publishing custom events:**

```java
// Define event
public class OrderCreatedEvent extends ApplicationEvent {
    private final Order order;

    public OrderCreatedEvent(Object source, Order order) {
        super(source);
        this.order = order;
    }

    public Order getOrder() { return order; }
}

// Publish event
@Service
public class OrderService {
    private final ApplicationEventPublisher publisher;

    public OrderService(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    @Transactional
    public Order createOrder(OrderRequest request) {
        Order order = orderRepo.save(new Order(request));
        publisher.publishEvent(new OrderCreatedEvent(this, order));
        return order;
    }
}

// Listen to event
@Component
public class NotificationListener {

    // Synchronous by default
    @EventListener
    public void handleOrderCreated(OrderCreatedEvent event) {
        sendEmail(event.getOrder());
    }

    // Async listener
    @Async
    @EventListener
    public void handleOrderCreatedAsync(OrderCreatedEvent event) {
        generateInvoicePdf(event.getOrder());
    }

    // Conditional listener
    @EventListener(condition = "#event.order.total > 1000")
    public void handleHighValueOrder(OrderCreatedEvent event) {
        alertManager(event.getOrder());
    }

    // Transaction-bound listener
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void afterOrderCommitted(OrderCreatedEvent event) {
        // Only fires if the transaction commits successfully
        publishToMessageBroker(event.getOrder());
    }
}
```

**Listening to startup events (before context exists):**

Events published before the `ApplicationContext` is created cannot use `@EventListener`. Register via `spring.factories` or `SpringApplication.addListeners()`:

```java
public class EarlyStartupListener implements ApplicationListener<ApplicationEnvironmentPreparedEvent> {
    @Override
    public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
        // Environment is ready but context doesn't exist yet
        ConfigurableEnvironment env = event.getEnvironment();
    }
}
```

```text
# META-INF/spring.factories
org.springframework.context.ApplicationListener=\
  com.example.EarlyStartupListener
```

**Interview Trap:** "Are `@EventListener` methods synchronous or asynchronous?"  
→ **Synchronous** by default — they run in the publisher's thread. Add `@Async` (and enable `@EnableAsync`) for asynchronous execution. Be careful: async listeners won't propagate exceptions to the publisher.

---

### Q48. How does `@ComponentScan` differ from `@Import` in practice?

**Answer:**

| Aspect                    | `@ComponentScan`                         | `@Import`                                 |
|---------------------------|------------------------------------------|-------------------------------------------|
| Discovery                 | Automatic — scans packages               | Explicit — specifies exact classes        |
| What it finds             | `@Component` stereotype classes          | Any class (config, component, selector)   |
| Use case                  | Application-level component discovery    | Framework/library configuration wiring    |
| Performance               | Slower (classpath scanning)              | Faster (no scanning)                      |
| Typical user              | Application developer                    | Library/framework author                  |

```java
// @ComponentScan — broad, automatic
@ComponentScan(basePackages = "com.example")
// Scans all sub-packages, finds everything annotated with stereotype annotations

// @Import — precise, explicit
@Import({
    SecurityConfig.class,          // specific config class
    RedisConfig.class,             // another config
    MyImportSelector.class         // dynamic selection
})
// Only these specific classes are imported — nothing else
```

**When to use which:**

- **Application code:** `@ComponentScan` (simpler, convention-based)
- **Library/starter code:** `@Import` (explicit, no assumptions about user's package structure)
- **Cross-module dependencies:** `@Import` (import specific configs from other modules)
- **Testing:** `@Import` (import only what you need for a focused test)

```java
// Test with minimal context — @Import is faster than full scan
@SpringBootTest
@Import(OrderService.class)
class OrderServiceTest { }
```

---

### Q49. How do you customize the Spring Boot banner?

**Answer:**

```
// Default: Spring text banner at startup
// Customization options:

// 1. Custom text banner: src/main/resources/banner.txt
//    Supports placeholders:
//    ${spring-boot.version}
//    ${application.title}
//    ${application.version}
//    ${AnsiColor.BRIGHT_GREEN}

// 2. Custom image banner: src/main/resources/banner.gif (or .jpg, .png)
//    Converted to ASCII art at startup

// 3. Programmatic banner
```

```java
SpringApplication app = new SpringApplication(MyApp.class);

// Disable banner
app.setBannerMode(Banner.Mode.OFF);

// Console only (not in log file)
app.setBannerMode(Banner.Mode.CONSOLE);

// Log only (not in console)
app.setBannerMode(Banner.Mode.LOG);

// Custom programmatic banner
app.setBanner((environment, sourceClass, out) -> {
    out.println("=================================");
    out.println("  My Application v" +
        environment.getProperty("app.version", "1.0.0"));
    out.println("  Active profiles: " +
        String.join(", ", environment.getActiveProfiles()));
    out.println("=================================");
});
```

```properties
# application.properties
spring.main.banner-mode=off

# Custom banner file location
spring.banner.location=classpath:custom-banner.txt
spring.banner.image.location=classpath:banner.png
```

**Interview Trap:** This is a low-weight question, but interviewers use it to test if you've actually worked with Spring Boot vs. just read about it. Knowing about `Banner.Mode.OFF` and the `banner.txt` file signals real hands-on experience.

---

### Q50. Explain graceful shutdown in Spring Boot.

**Answer:**

Graceful shutdown (introduced in Boot 2.3) allows in-flight requests to complete before the application shuts down:

```properties
# Enable graceful shutdown
server.shutdown=graceful

# Maximum wait time for active requests to complete
spring.lifecycle.timeout-per-shutdown-phase=30s
```

**Shutdown sequence:**

```
1. Shutdown signal received (SIGTERM / actuator /shutdown)
2. Server stops accepting NEW requests (returns 503)
3. Waits for in-flight requests to complete (up to timeout)
4. ApplicationContext close begins
   a. @PreDestroy methods called
   b. DisposableBean.destroy() called
   c. SmartLifecycle.stop() called (ordered by phase)
5. JVM exits
```

```java
// Custom shutdown behavior with SmartLifecycle
@Component
public class GracefulShutdownHandler implements SmartLifecycle {
    private volatile boolean running = false;

    @Override
    public void start() {
        running = true;
    }

    @Override
    public void stop(Runnable callback) {
        // Custom cleanup: flush queues, close connections, etc.
        System.out.println("Draining message queue...");
        drainQueue();
        running = false;
        callback.run(); // MUST call to signal completion
    }

    @Override
    public boolean isRunning() { return running; }

    @Override
    public int getPhase() {
        return SmartLifecycle.DEFAULT_PHASE; // higher phase = shut down first
    }
}
```

**Per embedded server behavior:**

| Server    | Behavior During Graceful Shutdown                         |
|-----------|-----------------------------------------------------------|
| Tomcat    | Stops accepting at network level, waits for active reqs   |
| Jetty     | Stops accepting, waits with timeout                       |
| Undertow  | Stops accepting, waits for existing exchanges             |

**Interview Trap:** "What's the default shutdown behavior without `server.shutdown=graceful`?"  
→ **Immediate** (`server.shutdown=immediate`). The server shuts down without waiting — in-flight requests may receive connection reset errors. This is the default because graceful shutdown has a performance cost (the timeout period delays JVM exit).

**Edge Case:** The shutdown timeout `spring.lifecycle.timeout-per-shutdown-phase` applies **per SmartLifecycle phase**, not total. If you have multiple phases, total shutdown time could be `timeout × number_of_phases`.

---

## Quick-Reference Cheat Sheet

### Annotation Cheat Sheet

| Annotation                        | Category         | Key Behavior                                               |
|-----------------------------------|------------------|------------------------------------------------------------|
| `@SpringBootApplication`          | Fundamentals     | `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` |
| `@ConditionalOnMissingBean`       | Auto-config      | Only create bean if none of this type exists                |
| `@ConditionalOnClass`             | Auto-config      | Only process if class is on classpath                       |
| `@ConditionalOnProperty`          | Auto-config      | Only process if property matches value                      |
| `@ConfigurationProperties`        | Config           | Type-safe bulk property binding                             |
| `@Profile`                        | Config           | Bean active only in specified profile                       |
| `@Lazy`                           | DI               | Defer bean creation until first access                      |
| `@Primary`                        | DI               | Default bean when multiple candidates                       |
| `@Qualifier`                      | DI               | Select specific bean by name                                |
| `@AutoConfiguration`              | Auto-config (3.x)| Marks auto-configuration class with ordering support        |
| `@Endpoint`                       | Actuator         | Custom actuator endpoint                                    |

### Key Properties Cheat Sheet

| Property                                           | Default    | Purpose                        |
|----------------------------------------------------|------------|--------------------------------|
| `spring.main.allow-circular-references`            | `false`    | Allow circular DI              |
| `spring.main.allow-bean-definition-overriding`     | `false`    | Allow bean name collision      |
| `spring.main.lazy-initialization`                  | `false`    | Global lazy init               |
| `spring.main.web-application-type`                 | auto       | SERVLET / REACTIVE / NONE      |
| `spring.main.banner-mode`                          | CONSOLE    | OFF / CONSOLE / LOG            |
| `spring.profiles.active`                           | (none)     | Active profiles                |
| `spring.autoconfigure.exclude`                     | (none)     | Exclude auto-configs           |
| `server.shutdown`                                  | immediate  | immediate / graceful           |
| `spring.lifecycle.timeout-per-shutdown-phase`      | 30s        | Graceful shutdown timeout      |
| `management.endpoints.web.exposure.include`        | health,info| Exposed actuator endpoints     |
| `debug`                                            | false      | Auto-config conditions report  |

### Boot 2.x → 3.x Migration Checklist

| Area                     | Boot 2.x                              | Boot 3.x                                |
|--------------------------|---------------------------------------|------------------------------------------|
| Java version             | 8 / 11                               | **17 minimum**                           |
| Namespace                | `javax.*`                             | `jakarta.*`                              |
| Auto-config registration | `spring.factories`                    | `.imports` file                          |
| Auto-config annotation   | `@Configuration` + `@AutoConfigureAfter` | `@AutoConfiguration(after=...)`      |
| Tracing                  | Spring Cloud Sleuth                   | **Micrometer Tracing**                   |
| HTTP clients             | `RestTemplate`                        | `RestClient` (new), `WebClient`          |
| Problem details          | Manual                                | RFC 7807 `ProblemDetail` built-in        |
| Native images            | Experimental                          | First-class GraalVM support              |
| `@ConstructorBinding`    | Required on type                      | Inferred (annotation optional on records)|

---

*End of Spring Boot Core Interview Questions — 2026 Edition*
