# Infosys Java + Spring Boot Interview Questions & Answers
### For 4 Years of Experience

This guide compiles the most commonly asked questions in Infosys technical interviews for Java/Spring Boot developers with around 4 years of experience, based on recent candidate interview experiences and standard topic coverage for this experience band. Questions are grouped by topic: **Core Java & Java 8**, **Collections & Multithreading**, **Spring Core**, **Spring Boot**, **Spring MVC/REST**, **Spring Data JPA/Hibernate**, **Microservices**, and **SQL/Project Experience**.

---

## Section 1: Core Java Fundamentals

### 1. What is Java and why is it platform-independent?
Java code is compiled into bytecode (`.class` files) rather than native machine code. The JVM (Java Virtual Machine) on any operating system can interpret/execute this bytecode, so the same compiled file runs unchanged on Windows, Linux, or macOS — "write once, run anywhere."

### 2. Difference between JDK, JRE, and JVM?
- **JVM**: The engine that executes bytecode; provides memory management, garbage collection.
- **JRE**: JVM + core libraries needed to *run* Java applications.
- **JDK**: JRE + development tools (compiler `javac`, debugger, etc.) needed to *build* Java applications.

### 3. Explain the four pillars of OOP with examples.
- **Encapsulation**: Wrapping data and methods together, restricting direct access via private fields and public getters/setters.
- **Inheritance**: A class acquiring properties/behavior of another using `extends`.
- **Polymorphism**: One interface, many implementations — achieved via method overloading (compile-time) and overriding (runtime).
- **Abstraction**: Hiding implementation details and exposing only essential features, via abstract classes or interfaces.

### 4. Difference between method overloading and overriding?
| Overloading | Overriding |
|---|---|
| Same method name, different parameter list | Same method signature in subclass |
| Resolved at compile time | Resolved at runtime |
| Can be in the same class | Requires inheritance |

### 5. Difference between abstract class and interface?
An abstract class can have both abstract and concrete methods, constructors, and instance variables — a class can extend only one. An interface (pre-Java 8) had only abstract methods; since Java 8, it can also have `default` and `static` methods, and a class can implement multiple interfaces, enabling a form of multiple inheritance of behavior.

### 6. Why doesn't Java support multiple inheritance for classes?
To avoid the **diamond problem** — ambiguity when two parent classes have a method with the same signature. Java sidesteps this by allowing multiple interface implementation (with default methods resolved explicitly if conflicts occur) but only single class inheritance.

### 7. What is the difference between `==` and `.equals()`?
`==` compares references (memory addresses) for objects, or actual values for primitives. `.equals()` is a method that can be overridden to compare logical/content equality (e.g., `String.equals()` compares character sequences, not memory location).

### 8. What is the `final` keyword used for?
- `final` variable: value cannot be reassigned once initialized.
- `final` method: cannot be overridden by subclasses.
- `final` class: cannot be extended/subclassed (e.g., `String` class).

### 9. What are static and instance variables/methods?
Static members belong to the class itself and are shared across all instances; instance members belong to individual objects. Static methods can't access instance variables directly and can't be overridden (only hidden).

### 10. What is the difference between `String`, `StringBuilder`, and `StringBuffer`?
`String` is immutable — every modification creates a new object. `StringBuilder` is mutable and not thread-safe, making it faster for single-threaded string manipulation. `StringBuffer` is mutable and thread-safe (synchronized methods), useful in multithreaded contexts but slightly slower than `StringBuilder`.

---

## Section 2: Java 8 Features (Heavily Emphasized at Infosys)

### 11. What are the major features introduced in Java 8?
Lambda expressions, the Stream API, functional interfaces, default/static methods in interfaces, the `Optional` class, method references, and the new `java.time` Date/Time API.

### 12. What is a lambda expression?
A concise way to represent an anonymous function — an implementation of a functional interface — without boilerplate. Example: `(a, b) -> a + b` instead of writing a full anonymous class implementing an interface with one abstract method.

### 13. What is a functional interface?
An interface with exactly one abstract method (it can have multiple default/static methods), marked optionally with `@FunctionalInterface`. Examples: `Runnable`, `Comparator`, `Function<T,R>`, `Predicate<T>`, `Supplier<T>`, `Consumer<T>`.

### 14. Explain the Stream API and its key operations.
Streams let you process collections in a functional, declarative style. Operations are split into:
- **Intermediate** (lazy, return a stream): `filter()`, `map()`, `sorted()`, `distinct()`
- **Terminal** (trigger execution): `collect()`, `forEach()`, `reduce()`, `count()`

Example: `list.stream().filter(x -> x > 10).map(x -> x * 2).collect(Collectors.toList())`

### 15. What is the difference between `map()` and `flatMap()`?
`map()` transforms each element into exactly one output element (one-to-one). `flatMap()` transforms each element into a stream of elements and then flattens all those streams into a single stream — useful when each input maps to zero or more outputs (e.g., flattening a `List<List<String>>` into a `List<String>`).

### 16. What is `Optional` and why was it introduced?
`Optional<T>` is a container object that may or may not hold a non-null value. It was introduced to avoid `NullPointerException` and force explicit handling of "value may be absent" cases instead of returning `null` silently. Common methods: `isPresent()`, `orElse()`, `orElseGet()`, `ifPresent()`, `map()`.

### 17. What is a method reference and how does it differ from a lambda?
A method reference (`ClassName::methodName`) is shorthand for a lambda that just calls an existing method. Example: `list.forEach(System.out::println)` is equivalent to `list.forEach(x -> System.out.println(x))` — it's more concise when the lambda body only delegates to an existing method.

### 18. What are default and static methods in interfaces, and why were they added?
Default methods (`default` keyword) let interfaces provide a method body, enabling new methods to be added to interfaces without breaking existing implementing classes. Static methods provide utility logic tied to the interface itself, not requiring an instance.

### 19. Explain `Predicate`, `Function`, `Supplier`, and `Consumer`.
- `Predicate<T>`: takes T, returns boolean (`test()`) — used for filtering.
- `Function<T,R>`: takes T, returns R (`apply()`) — used for transformation.
- `Supplier<T>`: takes nothing, returns T (`get()`) — used for lazy generation.
- `Consumer<T>`: takes T, returns nothing (`accept()`) — used for side-effect operations like printing.

### 20. How do you sort a list of objects using Streams and `Comparator`?
```java
list.stream()
    .sorted(Comparator.comparing(Employee::getSalary).reversed())
    .collect(Collectors.toList());
```
`Comparator.comparing()` extracts a sort key, and methods like `thenComparing()` and `reversed()` allow chaining for multi-level or descending sorts.

---

## Section 3: Collections & Multithreading

### 21. How does `HashMap` work internally?
A `HashMap` stores key-value pairs in an array of buckets. The key's `hashCode()` determines the bucket index. Within a bucket, entries are stored as a linked list (or, since Java 8, as a balanced tree if a bucket has 8+ collisions, for better worst-case lookup performance). `equals()` is used to confirm key matches within a bucket during collisions.

### 22. Difference between `HashMap`, `LinkedHashMap`, and `TreeMap`?
- `HashMap`: no ordering guarantee, O(1) average access.
- `LinkedHashMap`: maintains insertion order using a backing linked list.
- `TreeMap`: maintains keys in sorted (natural or comparator-defined) order, backed by a red-black tree, O(log n) access.

### 23. Difference between `ArrayList` and `LinkedList`?
`ArrayList` uses a dynamic array — fast random access (O(1)) but slower inserts/deletes in the middle (O(n) shifting). `LinkedList` uses a doubly linked list — fast inserts/deletes at the ends (O(1)) but slower random access (O(n) traversal).

### 24. What is the difference between `HashSet` and `TreeSet`?
`HashSet` offers no ordering and O(1) average performance, backed internally by a `HashMap`. `TreeSet` maintains sorted order, backed by a `TreeMap`, with O(log n) operations.

### 25. What is `ConcurrentHashMap` and how is it different from a synchronized `HashMap`?
`ConcurrentHashMap` allows concurrent reads and segmented/striped locking for writes (rather than locking the entire map), giving much better throughput in multithreaded scenarios than wrapping a `HashMap` with `Collections.synchronizedMap()`, which locks the whole map for every operation.

### 26. What is the difference between `Thread` and `Runnable`?
Implementing `Runnable` and passing it to a `Thread` is generally preferred over extending `Thread` directly, because Java doesn't support multiple inheritance — if you extend `Thread`, you can't extend any other class, whereas implementing `Runnable` keeps that option open and separates the "task" from the "execution mechanism."

### 27. What is the `synchronized` keyword and how does it help with thread safety?
`synchronized` ensures only one thread can execute a block/method on a given monitor (object lock) at a time, preventing race conditions on shared mutable state. It can be applied to methods or specific code blocks for finer-grained control.

### 28. What is `volatile` and when would you use it?
`volatile` ensures that reads/writes to a variable go directly to main memory rather than a thread's local cache, guaranteeing visibility of updates across threads. It does NOT guarantee atomicity, so it's suitable for simple flags but not for compound operations like increment.

### 29. What is the Executor framework, and why use it over manually creating threads?
The `ExecutorService` framework (from `java.util.concurrent`) manages a pool of reusable threads, handling thread lifecycle, task queuing, and scheduling. This avoids the overhead and resource exhaustion risk of spawning a new thread for every task manually.

### 30. What is `CompletableFuture` used for?
It represents a future result of an asynchronous computation and allows chaining dependent async tasks (`thenApply`, `thenCompose`, `thenCombine`) without blocking, making it useful for building non-blocking pipelines, e.g., calling multiple downstream services in parallel and combining results.

---

## Section 4: Spring Core

### 31. What is Inversion of Control (IoC) and Dependency Injection (DI)?
IoC is a principle where the control of object creation and lifecycle is transferred from the application code to a container/framework. DI is the implementation mechanism for IoC — the Spring container "injects" required dependencies into a class rather than the class creating them itself, reducing tight coupling.

### 32. What is constructor injection vs. setter injection, and which is recommended?
Constructor injection passes dependencies via the constructor, making them mandatory and allowing immutability (fields can be `final`). Setter injection uses setter methods, making dependencies optional and mutable. **Constructor injection is generally recommended** because it ensures the object is always fully initialized, supports immutability, and makes circular dependencies fail fast at startup rather than at runtime.

### 33. What is the Spring Bean lifecycle?
Roughly: container instantiates the bean → dependencies are injected → `@PostConstruct` method (if any) runs → bean is ready for use → on shutdown, `@PreDestroy` method (if any) runs before the bean is destroyed. `BeanPostProcessor` implementations can hook into before/after initialization steps too.

### 34. What is the difference between `@Component`, `@Service`, `@Repository`, and `@Controller`?
All are specializations of `@Component` and get picked up by component scanning equally, but they carry semantic meaning: `@Service` marks business logic classes, `@Repository` marks data access classes (and additionally enables automatic exception translation for persistence exceptions), and `@Controller`/`@RestController` marks web layer classes. Using the right one improves readability and enables annotation-specific framework behavior.

### 35. What are Spring Bean scopes?
- **singleton** (default): one shared instance per Spring container.
- **prototype**: a new instance every time it's requested.
- **request/session/application**: web-aware scopes tied to HTTP request/session/application lifecycle (only in web-aware contexts).

### 36. What is `@Autowired` and how does Spring resolve which bean to inject when multiple candidates exist?
`@Autowired` tells Spring to automatically inject a matching bean by type. When multiple beans of the same type exist, Spring uses `@Qualifier("beanName")` to disambiguate, or you can mark one bean as `@Primary` to make it the default choice.

### 37. What is AOP (Aspect-Oriented Programming) in Spring, and where have you used it?
AOP lets you modularize cross-cutting concerns (logging, security, transactions) separately from business logic using **aspects**, **advice** (`@Before`, `@After`, `@Around`, etc.), and **pointcuts** (expressions defining where advice applies). A common real-world use is a `@Around` advice that logs method execution time for all service-layer methods without modifying each method itself.

### 38. What is `ApplicationContext` vs `BeanFactory`?
`BeanFactory` is the basic IoC container providing DI. `ApplicationContext` is a superset that adds enterprise features: event publishing, internationalization, AOP integration, and easier bean configuration with annotations — it's what's used in virtually all real applications.

### 39. What is circular dependency in Spring and how do you resolve it?
It happens when Bean A depends on Bean B and Bean B depends on Bean A, causing Spring to fail at startup (particularly with constructor injection, since neither can be fully constructed first). Solutions: refactor the design to remove the cycle, use setter/field injection for one of the beans (allowing partial construction), or use `@Lazy` on one dependency.

### 40. What is `@Value` used for, and how is it different from `@ConfigurationProperties`?
`@Value("${some.property}")` injects a single property value directly into a field. `@ConfigurationProperties` binds a whole group of related properties into a strongly-typed POJO, which scales better, supports validation (`@Validated`), and is preferred for structured configuration over scattering multiple `@Value` annotations.

---

## Section 5: Spring Boot

### 41. What is Spring Boot, and how is it different from the Spring Framework?
Spring Boot is built on top of the Spring Framework to simplify application setup. It provides auto-configuration, embedded servers (Tomcat/Jetty by default), starter dependencies, and production-ready features (Actuator) so you can build a stand-alone, runnable application with minimal manual XML/Java configuration — whereas plain Spring requires more explicit setup.

### 42. How does Spring Boot auto-configuration work?
Spring Boot scans the classpath for dependencies and conditionally configures beans accordingly, driven by `@EnableAutoConfiguration` (bundled inside `@SpringBootApplication`). Conditional annotations like `@ConditionalOnClass` and `@ConditionalOnMissingBean` decide whether a given auto-configuration class applies, based on what's present on the classpath and what the developer has already defined — letting you override any default by simply defining your own bean.

### 43. What does `@SpringBootApplication` actually combine?
It's a meta-annotation combining `@Configuration` (marks the class as a source of bean definitions), `@EnableAutoConfiguration` (triggers auto-configuration), and `@ComponentScan` (scans the package and sub-packages for Spring-managed components).

### 44. What is Spring Boot Actuator, and which endpoints have you used?
Actuator exposes production-ready monitoring endpoints out of the box, such as `/actuator/health` (app health status), `/actuator/metrics` (JVM/app metrics), `/actuator/info`, and `/actuator/beans` (lists all registered beans) — useful for health checks and observability in production without writing custom code.

### 45. How do you manage different configurations for dev, QA, and prod environments?
Using **Spring Profiles** — separate `application-dev.properties`, `application-qa.properties`, `application-prod.properties` files, activated via `spring.profiles.active=dev` (set as an environment variable, JVM argument, or in the main properties file), letting the same JAR run with environment-specific settings.

### 46. How do you handle exceptions globally in a Spring Boot application?
Using `@ControllerAdvice` (or `@RestControllerAdvice` for REST APIs) combined with `@ExceptionHandler` methods that catch specific exception types and return a consistent error response structure across all controllers, instead of repeating try-catch blocks in every endpoint.

### 47. What is `@Scheduled` used for, and what's a real example?
`@Scheduled` (with `@EnableScheduling` on a configuration class) lets you run methods at fixed intervals or cron expressions — for example, a nightly job that clears stale cache entries or syncs data with another system, run via `@Scheduled(cron = "0 0 2 * * *")`.

### 48. What is `@Async` used for?
It marks a method to run asynchronously in a separate thread (backed by a configurable thread pool), so the calling thread isn't blocked — useful for tasks like sending notification emails after an order is placed, where the caller shouldn't wait for the email to actually send.

### 49. How do you handle database schema versioning in Spring Boot?
Using migration tools like **Flyway** or **Liquibase**, which track and apply versioned SQL/changelog scripts automatically on startup. This is preferred over `spring.jpa.hibernate.ddl-auto=update` in production, since auto-DDL updates can cause unintended schema changes or data loss.

### 50. What is the default embedded server in Spring Boot, and can you change it?
**Tomcat** is the default embedded server when using `spring-boot-starter-web`. You can switch to Jetty or Undertow by excluding the Tomcat starter dependency and adding the alternative starter (e.g., `spring-boot-starter-jetty`) in your build file.

---

## Section 6: Spring MVC / REST APIs

### 51. What is the difference between `@RestController` and `@Controller`?
`@Controller` is used for traditional MVC apps returning view names (e.g., to render an HTML page via a template engine). `@RestController` combines `@Controller` and `@ResponseBody`, meaning every method's return value is automatically serialized (typically to JSON) directly into the HTTP response body rather than resolved as a view.

### 52. Difference between `@PathVariable` and `@RequestParam`?
`@PathVariable` extracts values embedded in the URI path itself, e.g., `/users/{id}` → `@PathVariable Long id`. `@RequestParam` extracts values from query parameters, e.g., `/users?status=active` → `@RequestParam String status`.

### 53. How do you version a REST API in Spring Boot?
Common approaches: URI versioning (`/api/v1/users`), request parameter versioning (`?version=1`), header-based versioning (custom header like `X-API-Version`), or media type versioning (Accept header content negotiation). URI versioning is the simplest and most widely used in practice.

### 54. How would you validate request payloads in a REST API?
Using Bean Validation annotations (`@NotNull`, `@Size`, `@Email`, etc.) on the DTO fields, combined with `@Valid` on the controller method parameter. Validation failures throw a `MethodArgumentNotValidException`, which you can catch globally via `@ControllerAdvice` to return a clean, structured 400 error response.

### 55. What HTTP status codes would you return for: successful creation, validation failure, resource not found, and unauthorized access?
- Creation: **201 Created**
- Validation failure: **400 Bad Request**
- Resource not found: **404 Not Found**
- Unauthorized: **401 Unauthorized** (vs **403 Forbidden** when authenticated but not permitted)

### 56. How do you secure a REST API in Spring Boot?
Typically with **Spring Security**, using JWT (JSON Web Tokens) for stateless authentication: the client authenticates once and receives a token, which it sends in the `Authorization` header on subsequent requests; a security filter validates the token on each request rather than maintaining server-side session state.

---

## Section 7: Spring Data JPA / Hibernate

### 57. What is the difference between JPA and Hibernate?
JPA is a **specification** (a set of interfaces/annotations defining how Java objects map to database tables). Hibernate is the most popular **implementation** of that specification, providing the actual underlying ORM logic.

### 58. What is the difference between `EntityManager` and Hibernate `Session`?
`EntityManager` is the JPA-standard interface for managing entity persistence operations (portable across JPA providers). `Session` is Hibernate's native, provider-specific interface, offering some extra Hibernate-specific features (like batch operations) not exposed by the JPA spec. In most modern Spring Data JPA applications, you work with `EntityManager`/repositories rather than the raw Hibernate `Session` directly.

### 59. What is the N+1 select problem, and how do you fix it?
It occurs when fetching a list of entities triggers one query for the list, then an additional separate query *per entity* to lazily fetch each one's related association — resulting in N+1 total queries instead of one. Fixes include using `JOIN FETCH` in JPQL, `@EntityGraph`, or switching the fetch type to eager where appropriate (carefully, to avoid over-fetching).

### 60. Difference between `FetchType.LAZY` and `FetchType.EAGER`?
`LAZY` loads the associated entity/collection only when it's actually accessed — better for performance when the association isn't always needed. `EAGER` loads it immediately along with the parent entity. `LAZY` is generally the safer default for collections to avoid unnecessary joins and large result sets.

### 61. What is the difference between `save()`, `saveAndFlush()`, and `flush()` in Spring Data JPA?
`save()` persists or updates an entity, but the change may stay in the persistence context until a transaction commits. `flush()` forces pending changes to be synchronized to the database immediately (without committing). `saveAndFlush()` does both — saves and immediately flushes — useful when you need the database state to be current within the same transaction (e.g., before a native query reads it).

### 62. What are the different states of a JPA entity?
- **Transient**: a new object not yet associated with a persistence context.
- **Persistent (Managed)**: attached to a session/EntityManager; changes are tracked and auto-synced.
- **Detached**: was persistent but the session is now closed; changes aren't tracked.
- **Removed**: marked for deletion, removed from the database on flush/commit.

### 63. What is optimistic locking, and how do you implement it in JPA?
Optimistic locking assumes conflicts are rare and checks for them only at commit time, typically using a `@Version` field on the entity. If two transactions read the same row and one updates it, the version number increments; when the second transaction tries to commit, the version mismatch causes an `OptimisticLockException`, preventing a silent overwrite.

### 64. How do you write a custom query in Spring Data JPA?
Either by using **derived query methods** (Spring parses the method name, e.g., `findByNameAndStatus`), or by writing explicit JPQL/native SQL using `@Query("SELECT u FROM User u WHERE u.status = :status")` on a repository method, with native SQL support via `@Query(value = "...", nativeQuery = true)`.

### 65. What is the difference between JPQL and native SQL queries?
JPQL is object-oriented — it queries entities and their fields, and is database-agnostic since Hibernate translates it to the underlying SQL dialect. Native SQL queries the actual database tables/columns directly, which is sometimes necessary for database-specific features or complex queries, but ties you to that specific database.

---

## Section 8: Microservices & System Design

### 66. What are the key characteristics of a microservices architecture?
Independently deployable services, each owning its own data store, organized around business capabilities, communicating over lightweight protocols (REST/messaging), with decentralized governance — allowing teams to develop, scale, and deploy services independently rather than as one large monolith.

### 67. How do microservices communicate with each other?
**Synchronously** via REST or gRPC calls (direct request-response), or **asynchronously** via message brokers like Kafka or RabbitMQ (publish-subscribe or queue-based), where the asynchronous approach decouples services and improves resilience to downstream failures.

### 68. What is service discovery, and why is it needed in microservices?
In a dynamic environment where service instances scale up/down and change IP addresses, service discovery (e.g., via Eureka or Consul) lets services register themselves and look up other services by logical name rather than hardcoded addresses.

### 69. What is an API Gateway, and what problems does it solve?
A single entry point that routes client requests to the appropriate backend microservice, while also handling cross-cutting concerns like authentication, rate limiting, logging, and request/response transformation — so individual services don't need to duplicate that logic.

### 70. What is the Circuit Breaker pattern, and where have you used it?
It prevents a failing downstream service from cascading failures across the system by "opening the circuit" (failing fast or returning a fallback) after a threshold of failures, rather than letting every caller keep waiting on a timeout. Commonly implemented with **Resilience4j** in Spring Boot applications, wrapping calls to external/downstream services.

### 71. How do you handle distributed transactions across microservices?
Since traditional ACID transactions don't span multiple service databases, the common approach is the **Saga pattern** — a sequence of local transactions, each triggering the next via events, with compensating transactions defined to undo prior steps if a later step fails.

### 72. What is the difference between monolithic and microservices architecture?
A monolith is a single deployable unit with all functionality bundled together — simpler to develop initially but harder to scale specific parts independently. Microservices split functionality into independently deployable, independently scalable services, at the cost of added operational complexity (network calls, distributed data consistency, monitoring across services).

---

## Section 9: SQL, Exception Handling & Project Experience

### 73. What is the difference between checked and unchecked exceptions?
Checked exceptions (e.g., `IOException`) must be declared or caught at compile time — they represent recoverable conditions outside the program's control. Unchecked exceptions (e.g., `NullPointerException`, extending `RuntimeException`) aren't enforced at compile time and typically represent programming errors.

### 74. What is try-with-resources, and why use it?
A `try` block that automatically closes resources implementing `AutoCloseable` (like database connections or file streams) at the end of the block, even if an exception occurs — eliminating the need for manual `finally` blocks to close resources and reducing the risk of resource leaks.

### 75. Difference between `INNER JOIN`, `LEFT JOIN`, and `RIGHT JOIN`?
`INNER JOIN` returns only rows with matches in both tables. `LEFT JOIN` returns all rows from the left table plus matched rows from the right (nulls where no match exists). `RIGHT JOIN` is the mirror — all rows from the right table plus matches from the left.

### 76. What is the difference between `WHERE` and `HAVING` clauses?
`WHERE` filters individual rows *before* grouping/aggregation is applied. `HAVING` filters *after* `GROUP BY` has aggregated rows, so it's used to filter on aggregate results like `COUNT()` or `SUM()`.

### 77. What indexing strategies have you used to improve query performance?
Adding indexes on frequently filtered/joined columns (especially foreign keys and columns in `WHERE` clauses) to speed up lookups, while being mindful that indexes slow down writes (`INSERT`/`UPDATE`) and consume storage — so they're added selectively based on actual query patterns, not blindly on every column.

### 78. Walk me through a challenging production issue you debugged in your current project.
*(Answer based on your own experience — interviewers want to see your debugging process: how you identified the issue via logs/metrics, isolated the root cause, implemented a fix, and what you did to prevent recurrence, e.g., adding monitoring or tests.)*
**Sample structure**: "We noticed intermittent 504 timeouts under load. I checked application logs and Actuator metrics, found a connection pool exhaustion issue on calls to a downstream service with no timeout configured. I added explicit connect/read timeouts and a circuit breaker, then added an alert on pool usage to catch it earlier next time."

### 79. How do you ensure code quality in your team's Spring Boot projects?
Typical practices: unit testing with JUnit and Mockito, code reviews via pull requests, static analysis tools (SonarQube), maintaining good test coverage on the service layer, and CI/CD pipelines that run tests automatically before merge/deployment.

### 80. How would you design a caching strategy for a frequently-read, rarely-updated REST endpoint?
Use Spring's `@Cacheable` annotation backed by a cache provider (e.g., Redis for a distributed cache, or in-memory Caffeine for a single instance), set an appropriate TTL, and invalidate/evict the cache entry on updates via `@CacheEvict` to avoid serving stale data.

---

## Tips for the Infosys Interview Round

- **Round structure**: Typically 1–2 technical rounds (covering Java fundamentals, Java 8, Spring Boot, and sometimes basic SQL/AOP) followed by a managerial round focused on project experience, problem-solving approach, and communication, then an HR/CTC discussion.
- Be ready to explain **your own project's architecture** in depth — interviewers commonly probe deeper into whatever framework or concept you mention on your resume.
- Brush up on **Java 8 Streams/Lambdas** thoroughly — it's consistently one of the most emphasized areas in Infosys interviews for this experience level.
- Practice explaining concepts **out loud, concisely** — interviewers value clear articulation as much as correctness.

---

*This document is based on publicly shared Infosys interview experiences and standard Java/Spring Boot interview topics for the 4-years-experience band. Actual questions may vary by interviewer, role, and project requirements.*
