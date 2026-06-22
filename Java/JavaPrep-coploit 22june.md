# Java Spring Boot Backend Developer — Interview Questions & Answers

**Excerpt from your resume:**  
Results-driven Java Developer with 3.6 years of experience designing and delivering scalable, production-ready backend systems using Java, Spring Boot, REST APIs, and SQL. **Technical skills include Java, SQL, Spring Boot, Spring Data JPA, Hibernate, PostgreSQL, MySQL, AWS S3, Azure Blob Storage, Docker, JUnit, Mockito.** 

---

## How to use this file
- Each question is numbered and followed by a concise, interview-ready answer.  
- Coding questions include runnable Java or SQL snippets.  
- Focus areas: **Core Java**, **Spring Boot**, **REST & APIs**, **Databases & SQL**, **Hibernate/JPA**, **Microservices & Cloud**, **DevOps & Docker**, **Testing**, **Design & Architecture**, and **Coding problems**.

---

# 1. Core Java (1–15)

1. **What are the main principles of OOP in Java?**  
   **Answer:** Encapsulation, Abstraction, Inheritance, Polymorphism. Encapsulation hides internal state via access modifiers; abstraction exposes essential features; inheritance enables reuse; polymorphism allows objects to be treated as instances of their parent types.

2. **Explain `final`, `finally`, and `finalize()`.**  
   **Answer:** `final` is a keyword for constants, methods, or classes; `finally` is a block executed after `try/catch`; `finalize()` is a deprecated method invoked by GC before object reclamation (avoid relying on it).

3. **Difference between `==` and `.equals()` for objects.**  
   **Answer:** `==` compares references (identity); `.equals()` compares logical equality (can be overridden).

4. **What is the Java memory model — stack vs heap?**  
   **Answer:** Stack stores method frames and primitives/local references; heap stores objects and shared data; GC manages heap; stack frames are thread-local and short-lived.

5. **Explain immutability and give an example.**  
   **Answer:** Immutable objects cannot change state after creation (e.g., `String`, `Integer`). Benefits: thread-safety, simpler reasoning, safe caching.

6. **What are checked vs unchecked exceptions?**  
   **Answer:** Checked exceptions must be declared/handled (compile-time), e.g., `IOException`. Unchecked exceptions extend `RuntimeException` and need not be declared, e.g., `NullPointerException`.

7. **What is the difference between `HashMap` and `ConcurrentHashMap`?**  
   **Answer:** `HashMap` is not thread-safe; `ConcurrentHashMap` supports concurrent access with internal locking/lock-free segments and does not allow `null` keys/values.

8. **Explain `synchronized` vs `volatile`.**  
   **Answer:** `synchronized` enforces mutual exclusion and memory visibility for a block/method. `volatile` ensures visibility of writes to variables across threads but does not provide atomicity for compound actions.

9. **What is the purpose of `transient` and `static` keywords?**  
   **Answer:** `transient` prevents a field from being serialized; `static` denotes class-level members shared across instances.

10. **Explain Java 8 Streams and functional interfaces.**  
    **Answer:** Streams provide a fluent API for processing collections (map, filter, reduce) with lazy evaluation. Functional interfaces (e.g., `Function`, `Predicate`) enable lambda expressions.

11. **How does garbage collection work in Java?**  
    **Answer:** GC reclaims unreachable objects. Common collectors: Serial, Parallel, CMS, G1. Generational GC divides heap into young/old generations to optimize for short-lived objects.

12. **What is method overloading vs overriding?**  
    **Answer:** Overloading: same method name, different parameter list in same class. Overriding: subclass provides specific implementation for superclass method (same signature).

13. **Explain `Optional` and when to use it.**  
    **Answer:** `Optional<T>` represents a value that may be present or absent, reducing `null` checks and `NullPointerException`. Use in return types to indicate optional results.

14. **What is a `ClassLoader`?**  
    **Answer:** Component that loads classes at runtime. Types: Bootstrap, Extension, System/Application. Custom class loaders can load classes from non-standard sources.

15. **Explain `Comparable` vs `Comparator`.**  
    **Answer:** `Comparable` defines natural ordering via `compareTo()` in the class. `Comparator` is a separate object for custom ordering (can be passed to sorting methods).

---

# 2. Spring Boot & Spring Ecosystem (16–30)

16. **What is Spring Boot and why use it?**  
    **Answer:** Spring Boot simplifies Spring app setup with auto-configuration, embedded servers, opinionated starters, and production-ready features (metrics, health checks).

17. **Difference between `@Component`, `@Service`, `@Repository`, and `@Controller`.**  
    **Answer:** All are stereotypes for Spring beans. `@Repository` adds persistence exception translation; `@Service` indicates business logic; `@Controller`/`@RestController` handle web requests; `@Component` is generic.

18. **What is `@RestController`?**  
    **Answer:** Combines `@Controller` and `@ResponseBody` to create REST endpoints returning JSON/XML directly.

19. **Explain dependency injection in Spring.**  
    **Answer:** IoC container manages object creation and wiring. Injection types: constructor, setter, field (constructor preferred for immutability and testability).

20. **What is Spring Data JPA?**  
    **Answer:** Abstraction over JPA/Hibernate providing repository interfaces, query derivation, pagination, and simplified CRUD operations.

21. **How to configure properties in Spring Boot?**  
    **Answer:** Use `application.properties` or `application.yml`. Profiles (`application-dev.yml`) allow environment-specific configs. Use `@Value` or `@ConfigurationProperties` to bind properties.

22. **Explain `@Transactional`.**  
    **Answer:** Declares transactional boundaries. Spring manages transactions (commit/rollback) based on exceptions. Use on service layer methods.

23. **What is actuator in Spring Boot?**  
    **Answer:** Provides production-ready endpoints (health, metrics, info) for monitoring and management.

24. **How to handle exceptions in Spring Boot REST APIs?**  
    **Answer:** Use `@ControllerAdvice` with `@ExceptionHandler` to centralize error handling and return consistent error responses.

25. **Explain `@RequestMapping` vs `@GetMapping`/`@PostMapping`.**  
    **Answer:** `@RequestMapping` is generic mapping; `@GetMapping`/`@PostMapping` are specialized shortcuts for HTTP methods.

26. **How to secure Spring Boot APIs?**  
    **Answer:** Use Spring Security for authentication/authorization. Configure filters, JWT/OAuth2, method-level security (`@PreAuthorize`), and CSRF protection as needed.

27. **What is Spring Boot starter?**  
    **Answer:** A curated dependency that brings a set of libraries and auto-configuration for a feature (e.g., `spring-boot-starter-web`).

28. **How to implement pagination with Spring Data JPA?**  
    **Answer:** Use `Pageable` and `Page<T>` in repository methods; pass `PageRequest.of(page, size, sort)`.

29. **Explain `Bean` scopes in Spring.**  
    **Answer:** Common scopes: `singleton` (one per container), `prototype` (new instance each request), `request`, `session` (web scopes).

30. **How to create custom health indicators?**  
    **Answer:** Implement `HealthIndicator` and register as a bean; actuator will include it in `/actuator/health`.

---

# 3. REST APIs & Web Services (31–40)

31. **What are REST principles?**  
    **Answer:** Statelessness, resource-based URIs, use of HTTP verbs (GET/POST/PUT/DELETE), representation via JSON/XML, HATEOAS (optional).

32. **Difference between PUT and PATCH.**  
    **Answer:** `PUT` replaces the entire resource; `PATCH` applies partial updates.

33. **How to version REST APIs?**  
    **Answer:** URI versioning (`/v1/resource`), header versioning, or content negotiation. Choose consistent approach and document.

34. **What is idempotency and why is it important?**  
    **Answer:** Idempotent operations produce same result when repeated (e.g., `PUT`, `GET`). Important for retries and fault tolerance.

35. **How to handle large file uploads in Spring Boot?**  
    **Answer:** Use streaming, multipart configuration, set `spring.servlet.multipart.max-file-size`, and offload to cloud storage (S3/Azure Blob) for scalability.

36. **Explain CORS and how to configure it.**  
    **Answer:** Cross-Origin Resource Sharing controls cross-domain requests. Configure via `@CrossOrigin`, `WebMvcConfigurer`, or gateway.

37. **What is HATEOAS?**  
    **Answer:** Hypermedia as the Engine of Application State — include links in responses to guide clients.

38. **How to document APIs in Spring Boot?**  
    **Answer:** Use Swagger/OpenAPI (springdoc-openapi or springfox) to generate interactive API docs.

39. **How to implement rate limiting?**  
    **Answer:** Use API gateway (e.g., Kong), Redis-based token bucket, or libraries like Bucket4j.

40. **How to return consistent error responses?**  
    **Answer:** Define an error DTO (code, message, timestamp, details) and use `@ControllerAdvice` to map exceptions to DTOs.

---

# 4. Databases & SQL (41–55)

41. **Explain normalization and its normal forms.**  
    **Answer:** Normalization reduces redundancy. 1NF (atomic values), 2NF (no partial dependency), 3NF (no transitive dependency), BCNF (stronger 3NF).

42. **What is an index and when to use it?**  
    **Answer:** Index speeds up reads by creating a lookup structure. Use on columns used in `WHERE`, `JOIN`, `ORDER BY`. Avoid over-indexing (write penalty).

43. **Difference between clustered and non-clustered index.**  
    **Answer:** Clustered index defines physical order of rows (one per table). Non-clustered index is separate structure pointing to rows.

44. **How to find the second highest salary in SQL?**  
    **Answer:**  
```sql
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

45. **Explain transactions and isolation levels.**  
    **Answer:** Transactions ensure ACID. Isolation levels: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE — trade-offs between consistency and concurrency.

46. **What is a deadlock and how to prevent it?**  
    **Answer:** Deadlock occurs when transactions wait on each other’s locks. Prevent by consistent lock ordering, short transactions, and using lower isolation levels where safe.

47. **How to optimize slow queries?**  
    **Answer:** Add indexes, avoid `SELECT *`, use proper joins, analyze query plan, denormalize when necessary, and cache results.

48. **Explain `JOIN` types: INNER, LEFT, RIGHT, FULL.**  
    **Answer:** `INNER` returns matching rows; `LEFT` returns all left rows + matches; `RIGHT` returns all right rows + matches; `FULL` returns all rows from both sides with matches where available.

49. **How to implement pagination in SQL?**  
    **Answer:** Use `LIMIT`/`OFFSET` (Postgres/MySQL) or keyset pagination for large datasets (use `WHERE id > last_id LIMIT n`).

50. **What is an execution plan and how to read it?**  
    **Answer:** Execution plan shows how DB executes a query (scans, joins, index usage). Use `EXPLAIN`/`EXPLAIN ANALYZE` to inspect and optimize.

51. **How to handle migrations in production?**  
    **Answer:** Use tools like Flyway or Liquibase, version-controlled migration scripts, and run migrations in CI/CD with rollback strategies.

52. **Explain ACID vs BASE.**  
    **Answer:** ACID (strong consistency) vs BASE (eventual consistency) used in distributed/noSQL systems.

53. **How to store large binary files in DB vs cloud storage?**  
    **Answer:** Prefer cloud/object storage (S3/Azure Blob) for large files; store metadata and references in DB to improve performance and scalability.

54. **What is sharding and when to use it?**  
    **Answer:** Sharding partitions data across multiple DB instances for horizontal scaling. Use when single DB cannot handle load or storage.

55. **How to design a schema for audit logs?**  
    **Answer:** Use append-only table with timestamp, user, action, entity id, old/new values; consider partitioning and TTL for retention.

---

# 5. Hibernate / JPA (56–65)

56. **Difference between `EntityManager` and `Session` (Hibernate).**  
    **Answer:** `EntityManager` is JPA API; `Session` is Hibernate native API. Both manage persistence context; `EntityManager` is preferred for portability.

57. **What is lazy vs eager fetching?**  
    **Answer:** Lazy fetches related entities on access; eager fetches immediately. Use lazy by default to avoid N+1 issues.

58. **How to avoid N+1 select problem?**  
    **Answer:** Use `JOIN FETCH`, batch fetching, `@EntityGraph`, or DTO projections to reduce queries.

59. **Difference between `save()` and `persist()` in Hibernate.**  
    **Answer:** `save()` returns generated id and may execute immediately; `persist()` follows JPA semantics and requires transaction.

60. **Explain first-level and second-level cache.**  
    **Answer:** First-level cache is session-scoped and always enabled. Second-level cache is session-factory scoped (e.g., Ehcache) and optional for cross-session caching.

61. **How to write custom queries in Spring Data JPA?**  
    **Answer:** Use `@Query` with JPQL/SQL or method name derivation (`findByStatusAndCreatedAtBetween`).

62. **What is optimistic vs pessimistic locking?**  
    **Answer:** Optimistic uses versioning and fails on conflict; pessimistic acquires DB locks to prevent concurrent updates.

63. **How to map inheritance in JPA?**  
    **Answer:** Strategies: `SINGLE_TABLE`, `JOINED`, `TABLE_PER_CLASS`. Choose based on query patterns and normalization needs.

64. **Explain DTO projection and why use it.**  
    **Answer:** DTOs fetch only required fields to reduce data transfer and avoid exposing entities; use constructor expressions or `@Query` projections.

65. **How to handle large batch inserts/updates in JPA?**  
    **Answer:** Use batching (`hibernate.jdbc.batch_size`), flush/clear periodically, and avoid cascading large object graphs.

---

# 6. Microservices & Cloud (66–78)

66. **What are benefits of microservices?**  
    **Answer:** Independent deployability, scalability, technology heterogeneity, fault isolation, and faster team autonomy.

67. **What is an API Gateway and why use it?**  
    **Answer:** Single entry point for clients; handles routing, authentication, rate limiting, and protocol translation.

68. **Explain service discovery.**  
    **Answer:** Mechanism for services to find each other (e.g., Eureka, Consul, DNS-based discovery).

69. **What is circuit breaker pattern?**  
    **Answer:** Prevents cascading failures by short-circuiting calls to failing services (e.g., Resilience4j, Hystrix).

70. **How to handle distributed tracing?**  
    **Answer:** Use tracing systems (Zipkin, Jaeger, OpenTelemetry) to propagate trace IDs and visualize request flows.

71. **Explain eventual consistency and how to design for it.**  
    **Answer:** Accept temporary inconsistency; use compensating transactions, idempotent operations, and event-driven patterns.

72. **What is message queue vs stream?**  
    **Answer:** Queue (RabbitMQ) for point-to-point tasks; stream (Kafka) for durable, ordered event logs and pub/sub.

73. **How to deploy Spring Boot apps to AWS?**  
    **Answer:** Options: ECS/EKS (containers), Elastic Beanstalk, EC2, or serverless (Lambda with Spring Cloud Function). Use S3 for assets and CloudWatch for logs/metrics.

74. **How to secure secrets in cloud deployments?**  
    **Answer:** Use AWS Secrets Manager, Parameter Store, Azure Key Vault, or environment variables injected securely via CI/CD.

75. **What is blue-green and canary deployment?**  
    **Answer:** Blue-green switches traffic between two identical environments; canary gradually shifts traffic to new version for validation.

76. **How to design idempotent APIs?**  
    **Answer:** Use idempotency keys, safe HTTP methods, and ensure repeated requests produce same result.

77. **Explain CAP theorem.**  
    **Answer:** In distributed systems, you can have at most two of Consistency, Availability, and Partition tolerance. Design choices depend on requirements.

78. **How to handle cross-service transactions?**  
    **Answer:** Use Sagas (choreography/orchestration) or two-phase commit (rarely used due to complexity).

---

# 7. DevOps, Docker & CI/CD (79–88)

79. **What is Docker and why use it?**  
    **Answer:** Containerization tool to package apps with dependencies for consistent runtime across environments.

80. **How to containerize a Spring Boot app?**  
    **Answer:** Create a `Dockerfile` using a JDK/JRE base image, copy jar, expose port, and run `java -jar app.jar`. Use multi-stage builds to reduce image size.

81. **What is the difference between image and container?**  
    **Answer:** Image is a read-only template; container is a running instance of an image.

82. **How to use Docker Compose?**  
    **Answer:** Define multi-container apps in `docker-compose.yml` with services, networks, and volumes for local development.

83. **What is CI/CD and typical pipeline stages?**  
    **Answer:** CI/CD automates build, test, and deployment. Stages: build, unit tests, integration tests, static analysis, package, deploy.

84. **How to implement health checks for containers?**  
    **Answer:** Use `HEALTHCHECK` in Dockerfile or Kubernetes liveness/readiness probes to detect unhealthy containers.

85. **What is infrastructure as code (IaC)?**  
    **Answer:** Declarative provisioning using tools like Terraform, CloudFormation, or ARM templates.

86. **How to monitor applications in production?**  
    **Answer:** Use metrics (Prometheus), logs (ELK/CloudWatch), tracing (Jaeger), and alerting (PagerDuty).

87. **What is immutable infrastructure?**  
    **Answer:** Replace servers/containers instead of modifying them in-place to ensure reproducibility and rollback simplicity.

88. **How to manage database migrations in CI/CD?**  
    **Answer:** Run Flyway/Liquibase migrations as part of deployment pipeline with versioned scripts and rollback plans.

---

# 8. Testing & Quality (89–97)

89. **What is unit testing vs integration testing?**  
    **Answer:** Unit tests validate individual components in isolation; integration tests validate interactions between components or with external systems.

90. **How to write unit tests for Spring components?**  
    **Answer:** Use JUnit for assertions and Mockito to mock dependencies. Use `@WebMvcTest` for controller slice tests and `@SpringBootTest` for full context.

91. **What is Mockito and how to use it?**  
    **Answer:** Mocking framework to create test doubles. Use `@Mock`, `@InjectMocks`, `when(...).thenReturn(...)`, and `verify()`.

92. **How to test JPA repositories?**  
    **Answer:** Use `@DataJpaTest` with in-memory DB (H2) or testcontainers for realistic DB testing.

93. **What is test coverage and is 100% always necessary?**  
    **Answer:** Coverage measures executed code during tests. 100% is not always practical; focus on meaningful tests for critical paths.

94. **How to do contract testing for microservices?**  
    **Answer:** Use Pact or Spring Cloud Contract to ensure provider/consumer compatibility.

95. **What is mutation testing?**  
    **Answer:** Introduce small changes (mutations) to code to ensure tests detect faults; helps evaluate test suite effectiveness.

96. **How to write performance tests?**  
    **Answer:** Use JMeter, Gatling, or k6 to simulate load and measure latency, throughput, and resource usage.

97. **How to integrate static code analysis?**  
    **Answer:** Use SonarQube in CI to detect bugs, vulnerabilities, and code smells; enforce quality gates.

---

# 9. Design, Architecture & Behavioral (98–104)

98. **How to design a scalable backend for file metadata and large assets?**  
    **Answer:** Store metadata in relational DB with indexes; store large binaries in object storage (S3/Azure Blob); use CDN for delivery; design async processing for heavy tasks.

99. **What is CQRS and when to use it?**  
    **Answer:** Command Query Responsibility Segregation separates read and write models for scalability and optimized queries. Use when read/write patterns differ significantly.

100. **How to approach debugging a production issue?**  
    **Answer:** Reproduce in staging, check logs/traces, analyze metrics, isolate root cause, apply fix with minimal risk, and perform postmortem.

101. **How to prioritize technical debt vs feature work?**  
    **Answer:** Evaluate business impact, risk, and cost. Schedule debt reduction in sprints and tie to feature work when possible.

102. **How to design APIs for backward compatibility?**  
    **Answer:** Use versioning, additive changes only, deprecation policy, and consumer-driven contract testing.

103. **How to handle authentication and authorization in microservices?**  
    **Answer:** Centralize auth via OAuth2/JWT tokens, validate tokens at gateway or service level, and use RBAC/ABAC for fine-grained access control.

104. **Behavioral: How do you ensure zero-defect releases?**  
    **Answer:** Emphasize automated tests, code reviews, static analysis, CI/CD gates, staging validation, and incremental rollouts (canary/blue-green).

---

# 10. Coding Problems (105–120)

105. **Reverse a string (Java).**  
```java
public String reverse(String s) {
    return new StringBuilder(s).reverse().toString();
}
```

106. **Check if a string is palindrome.**  
```java
public boolean isPalindrome(String s) {
    String cleaned = s.replaceAll("\\W", "").toLowerCase();
    return cleaned.equals(new StringBuilder(cleaned).reverse().toString());
}
```

107. **Find factorial (iterative).**  
```java
public long factorial(int n) {
    long result = 1;
    for (int i = 2; i <= n; i++) result *= i;
    return result;
}
```

108. **Fibonacci sequence (first N numbers).**  
```java
public List<Integer> fibonacci(int n) {
    List<Integer> res = new ArrayList<>();
    int a = 0, b = 1;
    for (int i = 0; i < n; i++) {
        res.add(a);
        int sum = a + b;
        a = b; b = sum;
    }
    return res;
}
```

109. **Check prime number.**  
```java
public boolean isPrime(int n) {
    if (n <= 1) return false;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) return false;
    }
    return true;
}
```

110. **Find duplicates in array.**  
```java
public Set<Integer> findDuplicates(int[] arr) {
    Set<Integer> seen = new HashSet<>();
    Set<Integer> dup = new HashSet<>();
    for (int v : arr) {
        if (!seen.add(v)) dup.add(v);
    }
    return dup;
}
```

111. **Merge two sorted arrays.**  
```java
public int[] merge(int[] a, int[] b) {
    int i=0,j=0,k=0; int[] res = new int[a.length + b.length];
    while (i<a.length && j<b.length) res[k++] = (a[i] <= b[j]) ? a[i++] : b[j++];
    while (i<a.length) res[k++] = a[i++];
    while (j<b.length) res[k++] = b[j++];
    return res;
}
```

112. **Implement Singleton (thread-safe).**  
```java
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) instance = new Singleton();
            }
        }
        return instance;
    }
}
```

113. **LRU Cache (conceptual):**  
**Answer:** Use `LinkedHashMap` with access-order or a doubly-linked list + hashmap for O(1) get/put.

114. **SQL: Employees with salary greater than average.**  
```sql
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

115. **SQL: Top N per group (e.g., top 3 salaries per department).**  
```sql
SELECT *
FROM (
  SELECT e.*, ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) rn
  FROM employees e
) t
WHERE rn <= 3;
```

116. **Detect cycle in linked list (Floyd’s algorithm).**  
```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

117. **Serialize/deserialize JSON in Spring Boot.**  
**Answer:** Use Jackson (`ObjectMapper`) or rely on Spring Boot auto-configured `MappingJackson2HttpMessageConverter`.

118. **Implement debounce/throttle for API calls (conceptual).**  
**Answer:** Use rate-limiting at gateway, or in-service use token bucket/Leaky bucket algorithms and Redis for distributed counters.

119. **Design a URL shortener (high level).**  
**Answer:** Use base62 encoding of sequence IDs or hash with collision handling, store mapping in DB/Redis, use caching and analytics pipeline for metrics.

120. **Write a REST endpoint to upload a file to S3 (Spring Boot snippet).**  
```java
@PostMapping("/upload")
public ResponseEntity<String> upload(@RequestParam("file") MultipartFile file) {
    String key = UUID.randomUUID().toString() + "-" + file.getOriginalFilename();
    s3Client.putObject(PutObjectRequest.builder().bucket(bucket).key(key).build(),
                       RequestBody.fromBytes(file.getBytes()));
    return ResponseEntity.ok(key);
}
```

---

## Final notes
- This file contains **120** focused questions and answers across the full backend stack, with **coding examples** and **SQL snippets** you can practice.  
- Tailor your answers in interviews by referencing **projects** from your resume (e.g., migration utilities, PostgreSQL optimizations, AWS S3 integrations, JUnit/Mockito testing) and describe **impact** (performance gains, zero-defect releases, automation benefits). The resume excerpt above highlights these strengths. 

---

If you want, I can:
- Convert this into a downloadable `.md` file and split it into **topic-specific practice sets** (e.g., 30 Java questions, 20 SQL problems, 20 Spring Boot scenarios), or  
- Generate **mock interview flashcards** or **timed practice tests** from these questions.