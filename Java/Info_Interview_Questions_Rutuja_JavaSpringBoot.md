# Infosys Interview Questions & Answers — Java Spring Boot Developer
### Tailored to Rutuja Mungse's Resume (3.6 Years Experience — Cognizant)

This guide is customized around your actual experience: Java, Spring Boot, REST APIs, PostgreSQL, AWS (S3, CloudWatch), Azure Blob Storage, microservices, data migration/transformation utilities, JUnit/Mockito, SonarQube, and your full-stack pharmacy management project. It combines standard Infosys interview topics with questions an interviewer is likely to ask **directly off your resume**.

---

## Section 1: Questions Likely Asked Directly From Your Resume

### 1. Walk me through your role at Cognizant and the backend modules you've owned.
*(Use your own words — but structure it as:)* "I work as a Software Engineer at Cognizant, where I independently own design and development of backend modules and utilities in Java and Spring Boot, extending core platform functionality. I deliver high-priority features within Agile sprints, collaborating with internal and external global teams to translate business requirements into scalable REST API layers that integrate enterprise systems with real-time data synchronization."

### 2. You mention "zero-defect releases" — what practices helped you achieve that?
Rigorous unit testing with JUnit and Mockito before code review, static code analysis via SonarQube to catch code smells and vulnerabilities early, thorough peer code reviews, and validating edge cases in staging before deployment. The combination of automated testing and static analysis catches most issues before they reach production.

### 3. How did you use AWS S3 in your project, and why was it chosen for file storage?
S3 was used for orchestrating secure file transfers and storing large-scale digital asset records/metadata. S3 is preferred for this because it's highly durable, scalable object storage with fine-grained access control (IAM policies, bucket policies) and integrates well with Spring Boot via the AWS SDK for programmatic upload/download, without needing to manage file storage infrastructure ourselves.

### 4. How did you integrate AWS CloudWatch for monitoring, and what did you monitor?
CloudWatch was used to monitor system health and resilience of the cloud-ready infrastructure — typically tracking metrics like application logs, error rates, latency, and resource utilization, with alarms configured to alert the team proactively if thresholds (e.g., error spikes or high latency) are breached, rather than waiting for a client to report an issue.

### 5. What's the difference between AWS S3 and Azure Blob Storage, since you've used both?
Both are object storage services for unstructured data (files, binaries), but they belong to different cloud ecosystems. Conceptually: S3 "buckets" map to Blob Storage "containers." Functionally they're similar — durable, scalable storage with access control and lifecycle policies — but SDKs, authentication mechanisms (IAM vs. Azure AD/connection strings), and pricing models differ. Working with both shows adaptability across cloud providers, which is valuable in client environments with mixed infrastructure.

### 6. Tell me about the Java-based automation utilities you built for data migration. What problem did they solve?
*(Tailor to your real project, but structure as:)* "Manual data migration was time-consuming and error-prone across client migration engagements. I built Java automation utilities that handled end-to-end execution automatically — reading legacy data, applying transformation rules, and loading it into the target system — which eliminated manual effort and significantly reduced delivery timelines."

### 7. How did you ensure zero data loss while mapping legacy metadata to target schemas?
By designing transformation modules with strict validation at each step — verifying record counts before and after transformation, handling edge cases (nulls, format mismatches, encoding differences) explicitly rather than silently dropping bad records, and adding reconciliation checks/logging so any discrepancy between source and target could be caught and investigated immediately rather than going unnoticed.

### 8. What complex business transformation rules have you implemented, and how did you structure that code to stay maintainable?
*(Answer based on your actual work, but a good structure:)* "Rather than hardcoding all rules into one large class, I separated transformation logic into smaller, single-responsibility components/strategies per rule type, which made it easier to test each rule independently and to extend the system when new transformation rules came in, without touching unrelated logic."

### 9. How did you manage secure large-scale binary file transfers via Azure Blob Storage while meeting compliance requirements?
By using secure connection strings/SAS (Shared Access Signature) tokens with limited scope and expiry rather than exposing full storage account keys, encrypting data in transit (HTTPS) and at rest (Azure's built-in encryption), and ensuring access followed the principle of least privilege — only the specific containers/operations needed were authorized, in line with client security and compliance standards.

### 10. How did you optimize PostgreSQL performance for high-volume metadata?
Key techniques: adding indexes on frequently queried/filtered columns, avoiding `SELECT *` in favor of selecting only needed columns, using pagination for large result sets instead of loading everything into memory, analyzing slow queries with `EXPLAIN ANALYZE` to find bottlenecks, and batching inserts/updates rather than executing them one row at a time when processing large volumes of records.

### 11. Describe the pharmacy management system you built during your internship. What was your role end-to-end?
*(Tailor to your real experience, structure as:)* "It was a full-stack application using React on the frontend and Spring Boot with SQL on the backend. I implemented secure authentication workflows, a real-time inventory tracking module, and designed REST APIs to connect the frontend and backend. I also handled deployment on AWS and incorporated feedback from senior code reviews throughout."

### 12. How did you implement secure authentication in the pharmacy management system?
*(Typical answer if using Spring Security with JWT):* "Using Spring Security with JWT-based authentication — on login, the server validates credentials and issues a signed token; the client sends this token in the Authorization header on subsequent requests, and a security filter validates it on each request, avoiding the need for server-side session storage."

### 13. How did you design the real-time inventory tracking module?
*(General strong answer):* "The inventory table tracked stock levels, and any transaction (sale, restock) updated quantities within a transactional boundary to prevent race conditions when multiple operations happened concurrently. For real-time visibility on the frontend, the React app would re-fetch or receive updates reflecting the latest stock state after each transaction."

### 14. You're "Department Topper" in B.E. Computer Science — what foundational CS concepts do you rely on most in day-to-day backend work?
Data structures and algorithms for choosing the right collection/approach for a problem (e.g., HashMap for O(1) lookups vs. a sorted structure when order matters), database fundamentals (normalization, indexing, transactions) for schema design, and OOP principles for writing maintainable, extensible code — these fundamentals consistently show up even in "simple" CRUD or transformation logic.

---

## Section 2: Core Java & Java 8

### 15. What are the major Java 8 features you use regularly?
Streams API, lambda expressions, functional interfaces (`Predicate`, `Function`, `Consumer`, `Supplier`), `Optional` to avoid null checks, method references, and the `java.time` package for date/time handling instead of the legacy `Date`/`Calendar` classes.

### 16. How would you use Streams to process a large list of transformation records (relevant to your migration work)?
```java
List<Record> validRecords = records.stream()
    .filter(r -> r.getStatus() == Status.VALID)
    .map(this::applyTransformationRule)
    .collect(Collectors.toList());
```
This filters out invalid records and applies a transformation function in a clean, declarative pipeline instead of nested loops and conditionals.

### 17. What is `Optional` and how have you used it to avoid `NullPointerException` in metadata mapping?
`Optional<T>` wraps a value that might be absent, forcing explicit handling instead of letting `null` propagate silently. For example: `Optional.ofNullable(legacyRecord.getField()).map(this::transform).orElse(defaultValue)` — this avoids the bug-prone pattern of deep null checks scattered through transformation logic.

### 18. Difference between `HashMap` and `ConcurrentHashMap` — when would you use each in a migration utility?
`HashMap` is fine for single-threaded processing. If your migration utility processes batches concurrently (e.g., parallel streams or thread pools to speed up large-volume transformation), `ConcurrentHashMap` should be used for any shared map accessed by multiple threads, since plain `HashMap` isn't thread-safe and can corrupt internal state under concurrent modification.

### 19. What is the difference between checked and unchecked exceptions, and how do you handle exceptions in a data transformation pipeline?
Checked exceptions (like `IOException`) must be handled or declared; unchecked ones (like `IllegalArgumentException`) aren't enforced at compile time. In a transformation pipeline, I'd typically catch specific exceptions per record, log the failure with enough context to investigate, and continue processing remaining records (rather than letting one bad record crash the entire batch) — then report failed records separately for reconciliation.

### 20. What is the difference between `String`, `StringBuilder`, and `StringBuffer`, and which would you use when building large SQL strings dynamically?
`String` is immutable — repeated concatenation creates many intermediate objects, wasteful for building large strings. `StringBuilder` is mutable and efficient for single-threaded string building (like dynamically constructing query fragments), while `StringBuffer` adds synchronization overhead unnecessary unless multiple threads modify the same builder.

---

## Section 3: Spring Core & Spring Boot

### 21. What is Dependency Injection, and how does it make your Spring Boot modules more testable?
DI means the Spring container supplies a class's dependencies rather than the class creating them itself. This makes testing easier because in unit tests you can inject mock dependencies (via Mockito) instead of real ones — e.g., injecting a mocked repository into a service to test business logic in isolation without hitting an actual database.

### 22. Why do you prefer constructor injection over field injection (`@Autowired` on fields)?
Constructor injection makes dependencies explicit and allows fields to be `final` (immutable), ensures the object can't be constructed in an incomplete state, and makes unit testing straightforward since you can pass mocks directly via the constructor without needing reflection-based field injection in tests.

### 23. What is Spring Boot auto-configuration, and how does it simplify your day-to-day development?
It automatically configures beans based on what's on the classpath — e.g., adding `spring-boot-starter-data-jpa` automatically configures a `DataSource`, `EntityManager`, and transaction manager based on your `application.properties`, without manual XML/Java config. This lets you focus on business logic rather than wiring boilerplate.

### 24. How do you manage configuration differences between dev, QA, and production environments in your projects?
Using Spring Profiles — separate `application-{profile}.properties` files for each environment, activated via `spring.profiles.active`, set typically through environment variables or deployment pipeline parameters so the same build artifact can run correctly across environments without code changes.

### 25. How do you handle exceptions globally across your REST APIs?
Using `@RestControllerAdvice` combined with `@ExceptionHandler` methods for specific exception types, returning a consistent error response structure (status code, message, timestamp) — this avoids duplicating try-catch blocks in every controller and gives API consumers predictable error responses.

### 26. What is Spring Boot Actuator, and which endpoints would be useful for the CloudWatch monitoring you've worked with?
Actuator exposes endpoints like `/actuator/health` (overall app health, including custom health indicators for dependencies like the database), `/actuator/metrics` (JVM and custom application metrics), and `/actuator/info`. These can be scraped or pushed to CloudWatch to build dashboards and alarms around application health rather than relying purely on infrastructure-level metrics.

### 27. What is `@ConfigurationProperties`, and why might you prefer it over multiple `@Value` annotations for AWS/Azure credentials configuration?
`@ConfigurationProperties` binds a whole block of related properties (e.g., all AWS S3 config: bucket name, region, access settings) into one strongly-typed POJO, which is cleaner, supports validation, and scales better than scattering many individual `@Value("${...}")` annotations across classes.

### 28. How do you schedule recurring jobs in Spring Boot — for example, a nightly reconciliation job for migrated data?
Using `@Scheduled` with a cron expression on a method, combined with `@EnableScheduling` on a configuration class — e.g., `@Scheduled(cron = "0 0 1 * * *")` to run a reconciliation check every night at 1 AM, comparing source and target record counts and flagging mismatches.

### 29. What is `@Async`, and where might it help in a file transfer or migration utility?
`@Async` runs a method on a separate thread so the caller isn't blocked. For example, after uploading a file to S3/Blob Storage, sending a completion notification or triggering a downstream process could be done asynchronously so the main upload flow doesn't wait on that secondary action.

### 30. How would you implement retry logic for a flaky downstream API call or cloud storage operation?
Using Spring Retry (`@Retryable` with backoff configuration) to automatically retry transient failures (e.g., network blips during an S3 upload) a configured number of times before giving up, optionally combined with a fallback (`@Recover`) method if all retries fail — this is more robust than letting a single transient failure break the whole operation.

---

## Section 4: REST APIs & Microservices

### 31. How did you design REST APIs to enable real-time data synchronization across distributed systems, as mentioned in your resume?
*(Tailor, but structure as:)* "I designed RESTful endpoints following clear resource-based naming conventions, used appropriate HTTP methods and status codes, and ensured idempotency where needed for retries. For real-time sync, downstream systems would call these APIs immediately on data changes rather than relying on batch jobs, keeping data consistent across systems with minimal lag."

### 32. What is the difference between `@PathVariable` and `@RequestParam`, and how have you used them in your API layers?
`@PathVariable` extracts values from the URI path itself (e.g., `/assets/{assetId}`), used for identifying a specific resource. `@RequestParam` extracts query parameters (e.g., `/assets?status=active`), used for filtering, sorting, or optional parameters.

### 33. How do you handle API versioning when business requirements evolve?
Most commonly via URI versioning (`/api/v1/...`, `/api/v2/...`), which is simple and explicit for API consumers. Other approaches include header-based versioning, but URI versioning tends to be the most straightforward for cross-team/enterprise integration scenarios like the ones you describe.

### 34. What is the difference between synchronous and asynchronous communication between microservices, and when would you choose each?
Synchronous (REST/HTTP) calls block the caller until a response is received — simpler to reason about but creates tighter coupling and can cascade failures. Asynchronous communication (via message queues/Kafka) decouples services — the caller doesn't wait, improving resilience and scalability, but adds complexity around eventual consistency and message ordering.

### 35. What is the Circuit Breaker pattern, and how would it help when integrating with external enterprise systems?
It prevents repeatedly calling a failing downstream system by "opening" after a failure threshold and failing fast (or returning a fallback) for a cooldown period, instead of letting every request pile up on a timeout. This is especially relevant when integrating with external/legacy enterprise systems that may occasionally be slow or unavailable — it protects your service from cascading failures.

### 36. How is authentication typically handled in a microservices architecture?
Commonly via JWT — a central auth service issues a signed token after login, and each downstream microservice independently validates the token's signature (without querying a database each time), enabling stateless authentication across distributed services.

### 37. What is the Saga pattern, and why is it needed in microservices that don't share a single database?
Since each microservice typically owns its own database, a traditional ACID transaction can't span multiple services. The Saga pattern breaks a business transaction into a sequence of local transactions across services, each triggering the next, with **compensating transactions** defined to undo previous steps if a later step fails — maintaining eventual consistency instead of atomicity.

### 38. Difference between choreography and orchestration in the Saga pattern?
**Choreography**: each service listens for events and reacts independently — no central coordinator, more decoupled but harder to track the overall flow. **Orchestration**: a central orchestrator explicitly directs each step and handles failures/compensation — easier to monitor and reason about, but introduces a central point of coordination.

### 39. How would Kafka fit into a system like the one you described for real-time data synchronization?
Kafka could decouple producers and consumers of data-change events — e.g., when one enterprise system updates a record, it publishes an event to a Kafka topic, and any interested downstream system consumes that event asynchronously and updates its own state, rather than the producer directly calling every consumer synchronously. This improves scalability and fault tolerance.

### 40. How do you handle a failure when consuming a Kafka message — for example, a malformed record?
Typically by configuring a **dead-letter topic (DLT)** — after a configured number of retry attempts, the failing message is routed to a separate topic for investigation rather than blocking the main consumer or silently dropping the message, allowing the team to inspect and reprocess it later without affecting the main pipeline's throughput.

---

## Section 5: Spring Data JPA, Hibernate & SQL/PostgreSQL

### 41. What is the N+1 select problem, and how would it show up in a metadata-heavy application like yours?
If you fetch a list of parent entities (e.g., digital assets) and then lazily access a related collection (e.g., metadata tags) for each one individually, you trigger one query for the list plus N additional queries — one per asset. This is fixed using `JOIN FETCH` in a JPQL query or `@EntityGraph` to fetch the needed associations in a single query upfront.

### 42. Difference between `findById()` and a custom query — when would you write a custom repository method?
`findById()` (returns an `Optional<T>`) works for simple primary-key lookups. For anything beyond that — filtering by multiple fields, joins, or aggregations — you'd write either a derived query method (e.g., `findByStatusAndCreatedDateAfter`) or an explicit `@Query` with JPQL/native SQL for more complex logic that method-name derivation can't express cleanly.

### 43. How would you handle pagination for large metadata datasets in PostgreSQL via Spring Data JPA?
Using `Pageable` and returning a `Page<T>` from the repository method — e.g., `repository.findAll(PageRequest.of(page, size, sort))`. This translates to a `LIMIT`/`OFFSET` query under the hood, avoiding loading the entire large-volume dataset into memory at once.

### 44. What is the difference between `WHERE` and `HAVING` in SQL, relevant to reporting/reconciliation queries on migrated data?
`WHERE` filters individual rows before any grouping happens. `HAVING` filters after `GROUP BY` aggregation — useful for something like "show me all batches where the failed-record count exceeds 10," which requires filtering on an aggregated value (`COUNT()`), not a raw column.

### 45. How would you use `EXPLAIN ANALYZE` in PostgreSQL to diagnose a slow query on a large metadata table?
`EXPLAIN ANALYZE` shows the actual query execution plan along with real timing — revealing whether the database is doing a full table scan (slow) versus using an index (fast), how many rows are being processed at each step, and where the bulk of execution time is spent, guiding you toward adding the right index or rewriting the query.

### 46. What is optimistic locking, and would it be relevant when multiple processes update the same migration record concurrently?
Optimistic locking uses a `@Version` field on the entity — if two processes read the same record and both try to update it, the second commit detects the version mismatch and throws an `OptimisticLockException` instead of silently overwriting the first update. This is relevant for any concurrent batch processing scenario where the same record could theoretically be touched by more than one job.

### 47. Why is `ddl-auto=update` discouraged in production, and what would you use instead for schema changes in a client-facing system?
`ddl-auto=update` can make uncontrolled, automatic schema changes based on entity definitions, risking unintended data loss or inconsistent schema state across environments. Instead, controlled migration tools like **Flyway** or **Liquibase** should be used — versioned scripts that are reviewed, tested, and applied predictably across dev, QA, and production.

---

## Section 6: Testing, Code Quality & DevOps

### 48. How do you structure unit tests using JUnit and Mockito for a service that depends on a repository and an external API call?
Mock the repository and the external API client using `@Mock`/`@InjectMocks` (or `Mockito.mock()`), define expected behavior with `when(...).thenReturn(...)`, and assert that the service's business logic behaves correctly under different mocked scenarios (success, empty result, exception) — this tests the service's logic in complete isolation from real infrastructure.

### 49. What kinds of issues does SonarQube typically catch that you've relied on for zero-defect releases?
Code smells (overly complex methods, duplicated code), potential bugs (null pointer risks, resource leaks), security vulnerabilities (e.g., hardcoded credentials, SQL injection risks), and test coverage gaps — catching these before code review/merge reduces the chance of defects reaching QA or production.

### 50. How does your team's CI/CD pipeline typically work for a Spring Boot microservice, and where do JUnit/SonarQube fit in?
Typically: a commit triggers a build (Maven), runs unit/integration tests (JUnit/Mockito), then a SonarQube quality gate check — if tests fail or quality gate criteria aren't met (coverage, vulnerabilities), the pipeline fails and blocks merge/deployment. Only after passing all these stages does the artifact get deployed to the target environment, ensuring quality issues are caught automatically rather than relying purely on manual review.

---

## Interview Tips Based on Your Profile

- **Lead with your project ownership** — you've "taken full ownership of designing and developing backend modules," so be ready to go deep on architecture decisions you made, not just what you were told to build.
- **Be ready to discuss cloud trade-offs** — having both AWS and Azure experience is a strong differentiator; expect a question comparing them or asking why a particular service was chosen.
- **Quantify your impact where possible** — e.g., how much migration effort/time was actually saved by your automation utilities, since you've already framed this as a resume highlight.
- **Expect a managerial round** focused on your problem-solving approach, team collaboration across "internal and external global teams," and how you handle conflicting priorities in Agile sprints.
- **Brush up on Kafka and Saga pattern basics** even if you haven't used them directly — recent Infosys interviews for this experience band have included these topics, likely because most enterprise integration work eventually touches event-driven systems.

---

*This document is tailored using details from your resume combined with current Infosys interview patterns reported by recent candidates (2025–2026) for Java/Spring Boot roles in the 3–4 years experience band. Practice articulating your actual project specifics — interviewers consistently probe deeper into whatever you mention.*
