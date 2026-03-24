# Spring Boot REST API — Top 50 Interview Questions & Answers (2026)

> **Scope**: REST API development with Spring Boot 3.x / Spring 6.x  
> **Excludes**: Security, Testing, Microservices  
> **Format**: Concept → Code → Interviewer Focus → Pitfalls

---

## Section 1 — REST Controllers & Request Mapping

### Q1. What is the difference between `@RestController` and `@Controller`?

`@RestController` is a **composed annotation** that combines `@Controller` + `@ResponseBody`. Every handler method in a `@RestController` automatically serializes the return value into the HTTP response body.

```java
// Equivalent declarations
@RestController          // response body is implicit
public class OrderApi {
    @GetMapping("/orders")
    public List<Order> list() { return service.findAll(); }
}

@Controller              // requires explicit @ResponseBody
public class OrderApi {
    @GetMapping("/orders")
    @ResponseBody
    public List<Order> list() { return service.findAll(); }
}
```

| Aspect | `@Controller` | `@RestController` |
|---|---|---|
| Meta-annotations | `@Component` | `@Controller` + `@ResponseBody` |
| Default return semantics | View name resolution | Serialized response body |
| Content negotiation | Manual | Automatic via `HttpMessageConverter` |
| Use case | MVC with Thymeleaf/JSP | REST APIs |

**Interviewer focus**: Can you still return a view from `@RestController`? — *No, unless you explicitly return a `ModelAndView`, which defeats the purpose.*

**Pitfall**: Placing `@ResponseBody` on a class-level `@Controller` does NOT make it a `@RestController` for component scanning metadata purposes — Spring HATEOAS and OpenAPI generators check for `@RestController` specifically.

---

### Q2. Explain `@RequestMapping` and its specialized variants.

`@RequestMapping` is the general-purpose annotation; the shortcuts (`@GetMapping`, `@PostMapping`, etc.) appeared in Spring 4.3 and are preferred for readability.

```java
// General form
@RequestMapping(value = "/products", method = RequestMethod.GET,
                produces = "application/json")
public List<Product> list() { ... }

// Shortcut — preferred
@GetMapping(value = "/products", produces = MediaType.APPLICATION_JSON_VALUE)
public List<Product> list() { ... }
```

| Shortcut | HTTP Method | Typical Use |
|---|---|---|
| `@GetMapping` | GET | Read / Search |
| `@PostMapping` | POST | Create |
| `@PutMapping` | PUT | Full replace |
| `@PatchMapping` | PATCH | Partial update |
| `@DeleteMapping` | DELETE | Remove |

**Key attributes of `@RequestMapping`:**

- `value` / `path` — URI pattern
- `method` — HTTP method(s)
- `params` — Query parameter conditions (`params = "type=premium"`)
- `headers` — Header conditions (`headers = "X-API-Version=2"`)
- `consumes` — Request `Content-Type` filter
- `produces` — Response `Accept` filter

**Pitfall**: `@RequestMapping` at class level applies to ALL methods. If you set `produces = "application/json"` on the class, a method that needs to return CSV must explicitly override it.

---

### Q3. `@PathVariable` vs `@RequestParam` — When do you use each?

```java
// Path variable — identifies a resource
@GetMapping("/orders/{orderId}/items/{itemId}")
public OrderItem getItem(
        @PathVariable Long orderId,
        @PathVariable("itemId") Long itemId) { ... }

// Query parameter — filters, pagination, optional modifiers
@GetMapping("/orders")
public Page<Order> search(
        @RequestParam(defaultValue = "") String status,
        @RequestParam(required = false) LocalDate from,
        Pageable pageable) { ... }
```

| Aspect | `@PathVariable` | `@RequestParam` |
|---|---|---|
| Appears in | URI path `/orders/{id}` | Query string `?status=OPEN` |
| Required by default | Yes | Yes (configurable) |
| `defaultValue` support | No | Yes |
| Typical semantics | Resource identity | Filtering, sorting, paging |
| `Optional<T>` support | Yes (Spring 4.3+) | Yes |

**URI template with regex constraint:**

```java
@GetMapping("/users/{id:\\d+}")   // only digits
public User byId(@PathVariable Long id) { ... }

@GetMapping("/users/{slug:[a-z-]+}")  // slug pattern
public User bySlug(@PathVariable String slug) { ... }
```

**Pitfall**: If `@PathVariable` name differs from method parameter name you MUST specify the name attribute — `@PathVariable("orderId")`. With `-parameters` compiler flag this is optional but relying on it is fragile across builds.

**Interview trap**: What happens if `@RequestParam` is required but missing? — Spring throws `MissingServletRequestParameterException` which maps to **400 Bad Request**.

---

### Q4. How does `@RequestBody` work and what role does `HttpMessageConverter` play?

`@RequestBody` tells Spring to deserialize the HTTP request body into the annotated method parameter using a registered `HttpMessageConverter`.

```java
@PostMapping("/orders")
public ResponseEntity<Order> create(@Valid @RequestBody CreateOrderRequest req) {
    Order saved = service.create(req);
    URI location = URI.create("/orders/" + saved.getId());
    return ResponseEntity.created(location).body(saved);
}
```

**Flow**: Raw bytes → `HttpMessageConverter.read()` → Java object → handler method

Spring Boot auto-configures converters in this priority order:

1. `ByteArrayHttpMessageConverter`
2. `StringHttpMessageConverter`
3. `MappingJackson2HttpMessageConverter` (if Jackson on classpath)
4. `MappingJackson2XmlHttpMessageConverter` (if `jackson-dataformat-xml` on classpath)

**Pitfall**: `@RequestBody` is **required = true** by default. A request with an empty body throws `HttpMessageNotReadableException` (400). For optional bodies, use `@RequestBody(required = false)` and make the parameter `@Nullable`.

**Pitfall**: Forgetting `@RequestBody` entirely — Spring will try to resolve the parameter as a `@ModelAttribute` (form data), silently creating an empty object with default values.

---

### Q5. What are Matrix Variables and when are they useful?

Matrix variables are semicolon-delimited key-value pairs within a URI path segment, defined in RFC 3986.

```
GET /cars;color=red;year=2024/engines;type=hybrid
```

```java
@GetMapping("/cars/{carCriteria}/engines/{engineCriteria}")
public List<Car> filter(
        @MatrixVariable(pathVar = "carCriteria") Map<String, String> carFilters,
        @MatrixVariable(pathVar = "engineCriteria", name = "type") String engineType) {
    // carFilters = {color=red, year=2024}
    // engineType = "hybrid"
}
```

**Required setup** — matrix variables are **disabled by default**:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper helper = new UrlPathHelper();
        helper.setRemoveSemicolonContent(false);  // enable matrix variables
        configurer.setUrlPathHelper(helper);
    }
}
```

**Interview trap**: Interviewers ask this to test depth of REST knowledge. Knowing matrix variables exist AND that they are disabled by default demonstrates strong understanding.

---

### Q6. How does URI template pattern matching work in Spring MVC, and what changed in Spring 6?

Spring 6 (Boot 3) introduced `PathPatternParser` as the default, replacing the older `AntPathMatcher`.

| Feature | `AntPathMatcher` (legacy) | `PathPatternParser` (Spring 6 default) |
|---|---|---|
| Engine | String-based regex | Pre-parsed path pattern tree |
| Performance | Slower for complex patterns | Significantly faster |
| `{*path}` capture | Not supported | Supported (captures rest of path) |
| Regex in path vars | `{id:\\d+}` | `{id:\\d+}` |
| Trailing slash match | Enabled by default | **Disabled by default** |

```java
// Capture remaining path segments (Spring 6+)
@GetMapping("/files/{*resourcePath}")
public Resource serveFile(@PathVariable String resourcePath) {
    // /files/docs/2024/report.pdf → resourcePath = "/docs/2024/report.pdf"
}
```

**Critical breaking change in Spring Boot 3**: Trailing slash matching is **off** by default. `/users/` and `/users` are no longer equivalent. If your API relied on this, you must explicitly configure it:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer.setUseTrailingSlashMatch(true); // re-enable legacy behavior
    }
}
```

---

## Section 2 — Request/Response Handling

### Q7. Explain Content Negotiation in Spring Boot REST APIs.

Content negotiation determines the response format (JSON, XML, etc.) based on the client's `Accept` header.

```java
@GetMapping(value = "/reports/{id}",
            produces = { MediaType.APPLICATION_JSON_VALUE,
                         MediaType.APPLICATION_XML_VALUE })
public Report getReport(@PathVariable Long id) {
    return reportService.findById(id);
}
```

**Resolution strategy (default order):**

1. **`Accept` header** — `Accept: application/xml`
2. **Path extension** (disabled by default in Boot 3)
3. **Query parameter** — `/reports/1?format=xml` (must be enabled)

**Enabling parameter-based negotiation:**

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer
            .favorParameter(true)
            .parameterName("format")
            .defaultContentType(MediaType.APPLICATION_JSON)
            .mediaType("json", MediaType.APPLICATION_JSON)
            .mediaType("xml", MediaType.APPLICATION_XML);
    }
}
```

**Pitfall**: For XML support, you need `jackson-dataformat-xml` on the classpath. Without it, Spring returns `406 Not Acceptable` for `Accept: application/xml`.

---

### Q8. How do you customize Jackson's `ObjectMapper` in Spring Boot?

**Approach 1 — Properties (simple cases):**

```properties
spring.jackson.serialization.write-dates-as-timestamps=false
spring.jackson.serialization.indent-output=true
spring.jackson.deserialization.fail-on-unknown-properties=false
spring.jackson.default-property-inclusion=non_null
spring.jackson.date-format=yyyy-MM-dd'T'HH:mm:ss.SSSZ
```

**Approach 2 — `Jackson2ObjectMapperBuilderCustomizer` (fine-grained):**

```java
@Bean
public Jackson2ObjectMapperBuilderCustomizer jsonCustomizer() {
    return builder -> builder
        .serializationInclusion(JsonInclude.Include.NON_NULL)
        .featuresToEnable(SerializationFeature.INDENT_OUTPUT)
        .featuresToDisable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES)
        .modules(new JavaTimeModule())
        .dateFormat(new StdDateFormat().withColonInTimeZone(true));
}
```

**Approach 3 — Custom `ObjectMapper` bean (full control, use with caution):**

```java
@Bean
@Primary
public ObjectMapper objectMapper() {
    return JsonMapper.builder()
        .addModule(new JavaTimeModule())
        .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
        .build();
}
```

**Pitfall**: Declaring a custom `ObjectMapper` bean **replaces** the entire auto-configured one, losing Boot's defaults (e.g., `JavaTimeModule` registration). Prefer `Jackson2ObjectMapperBuilderCustomizer` for additive customization.

**Interview trap**: How do you handle `LocalDate` / `LocalDateTime`? — Register `JavaTimeModule` and disable `WRITE_DATES_AS_TIMESTAMPS`. This is auto-configured by Boot if `jackson-datatype-jsr310` is on classpath (it is, transitively).

---

### Q9. What is `ResponseEntity` and how does it differ from returning a plain object?

`ResponseEntity<T>` gives you full control over the HTTP response: status code, headers, and body.

```java
// Plain object — 200 OK with body, no header control
@GetMapping("/products/{id}")
public Product getProduct(@PathVariable Long id) {
    return service.findById(id);  // 200 OK always, null → empty body
}

// ResponseEntity — full control
@GetMapping("/products/{id}")
public ResponseEntity<Product> getProduct(@PathVariable Long id) {
    return service.findById(id)
        .map(product -> ResponseEntity.ok()
            .header("X-Product-Version", String.valueOf(product.getVersion()))
            .body(product))
        .orElse(ResponseEntity.notFound().build());
}
```

**Common `ResponseEntity` builder patterns:**

```java
ResponseEntity.ok(body)                              // 200 + body
ResponseEntity.ok().build()                           // 200 no body
ResponseEntity.created(uri).body(body)                // 201 + Location header
ResponseEntity.accepted().build()                     // 202
ResponseEntity.noContent().build()                    // 204
ResponseEntity.badRequest().body(errors)              // 400
ResponseEntity.notFound().build()                     // 404
ResponseEntity.status(HttpStatus.CONFLICT).body(msg)  // 409
ResponseEntity.unprocessableEntity().body(errors)     // 422
```

**Pitfall**: Returning `null` from a handler that returns a plain object results in a 200 OK with an empty body — NOT a 404. Always use `ResponseEntity` for proper status code control.

---

### Q10. How do you handle file uploads in a Spring Boot REST API?

```java
@PostMapping(value = "/documents", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public ResponseEntity<DocumentResponse> upload(
        @RequestPart("file") MultipartFile file,
        @RequestPart("metadata") DocumentMetadata metadata) {

    if (file.isEmpty()) {
        throw new BadRequestException("File is empty");
    }

    String stored = storageService.store(file.getOriginalFilename(),
                                          file.getInputStream());
    DocumentResponse resp = new DocumentResponse(stored, file.getSize());
    URI location = URI.create("/documents/" + resp.getId());
    return ResponseEntity.created(location).body(resp);
}
```

**Configuration in `application.yml`:**

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 50MB
      max-request-size: 50MB
      file-size-threshold: 2KB   # write to disk above this size
      location: /tmp/uploads
```

**`@RequestParam` vs `@RequestPart`:**

| Aspect | `@RequestParam` | `@RequestPart` |
|---|---|---|
| Resolution | Query param / form field | Multipart part with content type |
| JSON body part | No deserialization | Uses `HttpMessageConverter` |
| Recommended for | Simple file + string fields | File + complex JSON metadata |

**Pitfall**: Using `@RequestBody` instead of `@RequestPart` for multipart — `@RequestBody` tries to read the WHOLE body through a converter and fails with multipart.

**Multiple file upload:**

```java
@PostMapping("/batch-upload")
public ResponseEntity<List<String>> uploadMultiple(
        @RequestPart("files") List<MultipartFile> files) {
    List<String> ids = files.stream()
        .map(f -> storageService.store(f.getOriginalFilename(), f.getInputStream()))
        .toList();
    return ResponseEntity.ok(ids);
}
```

---

### Q11. Explain `@ModelAttribute` — how does it differ from `@RequestBody`?

| Aspect | `@ModelAttribute` | `@RequestBody` |
|---|---|---|
| Source | Query params + form fields | Request body (JSON/XML) |
| Content-Type | `application/x-www-form-urlencoded`, `multipart/form-data` | `application/json`, `application/xml` |
| Binding mechanism | `DataBinder` (setter-based) | `HttpMessageConverter` (Jackson) |
| Nested objects | Dot notation: `address.city=NY` | Standard JSON nesting |
| Default behavior | Implicit for non-simple types | Requires explicit annotation |

```java
// @ModelAttribute — form data binding
@PostMapping("/users")
public ResponseEntity<User> createFromForm(@ModelAttribute UserForm form) { ... }

// The @ModelAttribute is actually implicit for complex types in Spring MVC:
@PostMapping("/users")
public ResponseEntity<User> createFromForm(UserForm form) { ... }  // same behavior
```

**Interview trap**: What happens if you omit `@RequestBody` on a parameter? — Spring treats it as `@ModelAttribute` implicitly, which binds from query/form params. The JSON body is completely ignored, resulting in an object with all null fields.

---

### Q12. How does `HttpMessageConverter` selection work internally?

When a request arrives, Spring iterates through registered converters in order:

**Read (request deserialization):**
1. Check `canRead(targetType, contentType)` on each converter
2. First converter that returns `true` handles deserialization
3. If none matches → `HttpMediaTypeNotSupportedException` (415)

**Write (response serialization):**
1. Determine producible media types (from `produces`, `Accept` header)
2. Check `canWrite(returnType, mediaType)` on each converter
3. First match handles serialization
4. If none matches → `HttpMediaTypeNotAcceptableException` (406)

**Adding a custom converter:**

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        // WARNING: this REPLACES all defaults
        converters.add(new MappingJackson2HttpMessageConverter());
    }

    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        // SAFE: adds to existing defaults
        converters.add(new CsvHttpMessageConverter());
    }
}
```

**Pitfall**: Overriding `configureMessageConverters` instead of `extendMessageConverters` wipes out ALL defaults including Jackson. Always prefer `extendMessageConverters`.

---

## Section 3 — HTTP Methods & Status Codes

### Q13. What is idempotency and which HTTP methods are idempotent?

An operation is **idempotent** if calling it once has the same effect as calling it N times.

| Method | Idempotent | Safe | Typical Behavior |
|---|---|---|---|
| GET | Yes | Yes | Read resource; no side effects |
| HEAD | Yes | Yes | Like GET but no body |
| OPTIONS | Yes | Yes | Returns allowed methods |
| PUT | Yes | No | Full resource replacement |
| DELETE | Yes | No | Remove resource |
| POST | **No** | No | Create / trigger action |
| PATCH | **No** | No | Partial update (depends on implementation) |

**Why PUT is idempotent but POST is not:**

```
PUT /orders/42  {status: "SHIPPED"}  → always results in same state
PUT /orders/42  {status: "SHIPPED"}  → same result

POST /orders   {item: "Widget"}     → creates order #1
POST /orders   {item: "Widget"}     → creates order #2 (different resource)
```

**Pitfall on PATCH**: PATCH CAN be idempotent (JSON Merge Patch usually is) but is NOT guaranteed to be. JSON Patch (`application/json-patch+json`) operations like "increment counter" are non-idempotent.

**Interview trap**: "Is DELETE truly idempotent?" — Yes. First call deletes the resource (200/204), subsequent calls return 404 (resource already gone). The server-side **effect** is the same: resource doesn't exist.

---

### Q14. What are the correct HTTP status codes for CRUD operations?

```java
// CREATE — 201 Created + Location header
@PostMapping("/products")
public ResponseEntity<Product> create(@Valid @RequestBody CreateProductRequest req) {
    Product saved = service.create(req);
    URI location = ServletUriComponentsBuilder.fromCurrentRequest()
        .path("/{id}")
        .buildAndExpand(saved.getId())
        .toUri();
    return ResponseEntity.created(location).body(saved);
}

// READ — 200 OK or 404 Not Found
@GetMapping("/products/{id}")
public ResponseEntity<Product> findById(@PathVariable Long id) {
    return service.findById(id)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
}

// UPDATE (full) — 200 OK with body or 204 No Content
@PutMapping("/products/{id}")
public ResponseEntity<Product> replace(@PathVariable Long id,
                                        @Valid @RequestBody Product product) {
    Product updated = service.replace(id, product);
    return ResponseEntity.ok(updated);
}

// UPDATE (partial) — 200 OK
@PatchMapping("/products/{id}")
public ResponseEntity<Product> patch(@PathVariable Long id,
                                      @RequestBody Map<String, Object> fields) {
    Product patched = service.patch(id, fields);
    return ResponseEntity.ok(patched);
}

// DELETE — 204 No Content
@DeleteMapping("/products/{id}")
public ResponseEntity<Void> delete(@PathVariable Long id) {
    service.delete(id);
    return ResponseEntity.noContent().build();
}
```

**Complete Status Code Reference for REST APIs:**

| Code | Meaning | When to Use |
|---|---|---|
| 200 | OK | Successful GET, PUT, PATCH with body |
| 201 | Created | Successful POST; include `Location` header |
| 202 | Accepted | Async processing started |
| 204 | No Content | Successful DELETE, PUT/PATCH with no body |
| 400 | Bad Request | Validation failure, malformed JSON |
| 404 | Not Found | Resource doesn't exist |
| 405 | Method Not Allowed | Wrong HTTP method on endpoint |
| 406 | Not Acceptable | Cannot produce requested `Accept` type |
| 409 | Conflict | Duplicate creation, optimistic lock failure |
| 415 | Unsupported Media Type | Wrong `Content-Type` |
| 422 | Unprocessable Entity | Semantic validation failure (valid JSON, bad business logic) |
| 429 | Too Many Requests | Rate limiting |
| 500 | Internal Server Error | Unhandled exception |

---

### Q15. How do you implement ETag-based conditional requests?

ETags enable cache validation. The server generates an ETag; the client sends it back in `If-None-Match` (GET) or `If-Match` (PUT/DELETE).

**Shallow ETag (response body hash) — automatic:**

```java
@Bean
public FilterRegistrationBean<ShallowEtagHeaderFilter> shallowEtagFilter() {
    FilterRegistrationBean<ShallowEtagHeaderFilter> registration = new FilterRegistrationBean<>();
    registration.setFilter(new ShallowEtagHeaderFilter());
    registration.addUrlPatterns("/api/*");
    return registration;
}
```

`ShallowEtagHeaderFilter` computes an MD5 of the response body. If the client sends `If-None-Match` with a matching ETag, Spring returns **304 Not Modified** with no body. Caveat: the controller still executes fully — no DB savings.

**Deep ETag (version-based) — manual, more efficient:**

```java
@GetMapping("/products/{id}")
public ResponseEntity<Product> findById(@PathVariable Long id) {
    Product product = service.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Product", id));

    String etag = "\"" + product.getVersion() + "\"";

    return ResponseEntity.ok()
        .eTag(etag)
        .cacheControl(CacheControl.maxAge(30, TimeUnit.SECONDS))
        .body(product);
}

// Conditional update — optimistic concurrency
@PutMapping("/products/{id}")
public ResponseEntity<Product> update(
        @PathVariable Long id,
        @RequestBody Product product,
        @RequestHeader(value = "If-Match", required = false) String ifMatch) {

    Product existing = service.findById(id).orElseThrow();
    if (ifMatch != null && !ifMatch.equals("\"" + existing.getVersion() + "\"")) {
        return ResponseEntity.status(HttpStatus.PRECONDITION_FAILED).build(); // 412
    }
    Product updated = service.update(id, product);
    return ResponseEntity.ok().eTag("\"" + updated.getVersion() + "\"").body(updated);
}
```

**Pitfall**: ETag values MUST be quoted strings per HTTP spec — `"v1"` not `v1`. Spring's `ResponseEntity.eTag()` adds quotes automatically if missing.

---

### Q16. PUT vs PATCH — What's the real difference and how do you implement PATCH?

| Aspect | PUT | PATCH |
|---|---|---|
| Semantics | Full replacement | Partial update |
| Missing fields | Set to null/default | Left unchanged |
| Idempotent | Yes | Not guaranteed |
| Request body | Complete resource representation | Only changed fields |

**PATCH with `Map<String, Object>` (simple approach):**

```java
@PatchMapping("/users/{id}")
public ResponseEntity<User> patch(@PathVariable Long id,
                                   @RequestBody Map<String, Object> updates) {
    User user = service.findById(id).orElseThrow();

    updates.forEach((key, value) -> {
        switch (key) {
            case "name"  -> user.setName((String) value);
            case "email" -> user.setEmail((String) value);
            // ignore unknown fields
        }
    });

    return ResponseEntity.ok(service.save(user));
}
```

**PATCH with JSON Merge Patch (RFC 7396) — better approach:**

```java
@PatchMapping(value = "/users/{id}",
              consumes = "application/merge-patch+json")
public ResponseEntity<User> mergePatch(
        @PathVariable Long id,
        @RequestBody JsonMergePatch patch) {

    User user = service.findById(id).orElseThrow();
    JsonValue patched = patch.apply(objectMapper.convertValue(user, JsonValue.class));
    User updatedUser = objectMapper.convertValue(patched, User.class);
    return ResponseEntity.ok(service.save(updatedUser));
}
```

**Pitfall**: With `Map<String, Object>`, you cannot distinguish between "field not sent" and "field explicitly set to null". JSON Merge Patch handles this: sending `{"name": null}` explicitly nulls the field, while omitting `name` leaves it unchanged.

---

### Q17. How do you return proper `Location` headers for created resources?

```java
@PostMapping("/orders")
public ResponseEntity<Order> create(@Valid @RequestBody CreateOrderRequest req) {
    Order saved = service.create(req);

    // Option 1: ServletUriComponentsBuilder (most common)
    URI location = ServletUriComponentsBuilder
        .fromCurrentRequest()      // base = current request URI
        .path("/{id}")
        .buildAndExpand(saved.getId())
        .toUri();
    // Produces: http://localhost:8080/orders/42

    // Option 2: UriComponentsBuilder (explicit control)
    URI location2 = UriComponentsBuilder
        .fromUriString("https://api.example.com/v2/orders/{id}")
        .buildAndExpand(saved.getId())
        .toUri();

    return ResponseEntity.created(location).body(saved);
}
```

**Behind a reverse proxy:**

```properties
# Trust forwarded headers (X-Forwarded-Host, X-Forwarded-Proto, etc.)
server.forward-headers-strategy=NATIVE
```

Without this, `ServletUriComponentsBuilder` generates `http://` URLs even when the client connects via `https://` through a load balancer.

---

## Section 4 — Exception Handling

### Q18. How does `@ExceptionHandler` work at the controller level?

`@ExceptionHandler` catches exceptions thrown by handler methods within the same controller.

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    @GetMapping("/{id}")
    public Order findById(@PathVariable Long id) {
        return service.findById(id)
            .orElseThrow(() -> new OrderNotFoundException(id));
    }

    @ExceptionHandler(OrderNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(OrderNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            Instant.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
}
```

**Resolution order**: Controller-level `@ExceptionHandler` takes priority over `@ControllerAdvice` for the same exception type.

**Pitfall**: `@ExceptionHandler` in a controller only handles exceptions from THAT controller's methods. For global handling, you need `@ControllerAdvice`.

---

### Q19. Explain `@ControllerAdvice` and `@RestControllerAdvice` for global exception handling.

```java
@RestControllerAdvice  // = @ControllerAdvice + @ResponseBody
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse(404, ex.getMessage(), Instant.now());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        List<FieldError> fieldErrors = ex.getBindingResult().getFieldErrors().stream()
            .map(fe -> new FieldError(fe.getField(), fe.getDefaultMessage()))
            .toList();
        ErrorResponse error = new ErrorResponse(400, "Validation failed", Instant.now(), fieldErrors);
        return ResponseEntity.badRequest().body(error);
    }

    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResponseEntity<ErrorResponse> handleMalformedJson(HttpMessageNotReadableException ex) {
        return ResponseEntity.badRequest()
            .body(new ErrorResponse(400, "Malformed JSON request", Instant.now()));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        log.error("Unhandled exception", ex);  // log the full stack trace
        return ResponseEntity.internalServerError()
            .body(new ErrorResponse(500, "Internal server error", Instant.now()));
    }
}
```

**Scoping `@ControllerAdvice`:**

```java
@RestControllerAdvice(basePackages = "com.app.api.v2")     // by package
@RestControllerAdvice(assignableTypes = {OrderController.class})  // by class
@RestControllerAdvice(annotations = PublicApi.class)        // by annotation
```

**Custom error response DTO:**

```java
public record ErrorResponse(
    int status,
    String message,
    Instant timestamp,
    List<FieldError> errors  // optional
) {
    public ErrorResponse(int status, String message, Instant timestamp) {
        this(status, message, timestamp, List.of());
    }
}

public record FieldError(String field, String message) {}
```

---

### Q20. What is `ProblemDetail` (RFC 7807) in Spring 6, and how do you use it?

Spring 6 introduced native support for RFC 7807 "Problem Details for HTTP APIs", providing a standard error response format.

**Default ProblemDetail JSON structure:**

```json
{
  "type": "https://api.example.com/errors/out-of-stock",
  "title": "Product Out of Stock",
  "status": 409,
  "detail": "Product 42 has 0 units available",
  "instance": "/orders/123"
}
```

**Usage in exception handler:**

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(OutOfStockException.class)
    public ProblemDetail handleOutOfStock(OutOfStockException ex) {
        ProblemDetail pd = ProblemDetail.forStatusAndDetail(
            HttpStatus.CONFLICT, ex.getMessage());
        pd.setType(URI.create("https://api.example.com/errors/out-of-stock"));
        pd.setTitle("Product Out of Stock");
        pd.setInstance(URI.create("/products/" + ex.getProductId()));

        // Custom extensions
        pd.setProperty("productId", ex.getProductId());
        pd.setProperty("availableStock", 0);
        return pd;
    }
}
```

**Enabling globally via properties:**

```properties
spring.mvc.problemdetails.enabled=true
```

This makes Spring's default exception handling (for validation errors, 404, etc.) also return `ProblemDetail` format instead of the legacy `DefaultErrorAttributes` JSON.

**Extending `ResponseEntityExceptionHandler`** gives you overrideable hooks for ALL Spring MVC exceptions:

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(
            MethodArgumentNotValidException ex,
            HttpHeaders headers,
            HttpStatusCode status,
            WebRequest request) {

        ProblemDetail pd = ProblemDetail.forStatus(status);
        pd.setTitle("Validation Failed");
        pd.setProperty("errors", ex.getBindingResult().getFieldErrors().stream()
            .map(fe -> Map.of("field", fe.getField(), "message", fe.getDefaultMessage()))
            .toList());
        return ResponseEntity.status(status).body(pd);
    }
}
```

**Interview trap**: What's the difference between defining your own `ErrorResponse` record vs using `ProblemDetail`? — `ProblemDetail` follows an industry standard (RFC 7807), making your API interoperable with standard client libraries. Custom structures require documentation.

---

### Q21. How does `@ResponseStatus` on a custom exception work?

```java
@ResponseStatus(HttpStatus.NOT_FOUND)  // 404
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String resource, Object id) {
        super(resource + " not found with id: " + id);
    }
}
```

When this exception escapes a handler method, Spring's `ResponseStatusExceptionResolver` reads the annotation and returns that status code. The response body uses Spring Boot's default error attributes.

**Limitations:**
- No control over the response body structure (uses DefaultErrorAttributes)
- Cannot add custom headers
- Status code is compile-time fixed

**Better alternative — `ResponseStatusException` (programmatic):**

```java
@GetMapping("/products/{id}")
public Product findById(@PathVariable Long id) {
    return service.findById(id)
        .orElseThrow(() -> new ResponseStatusException(
            HttpStatus.NOT_FOUND,
            "Product not found: " + id));
}
```

**Pitfall**: `@ResponseStatus` on an exception class is ignored if a `@ControllerAdvice` has an `@ExceptionHandler` for that exception type — the handler takes priority.

---

### Q22. What is the exception handling resolution order in Spring MVC?

When an exception is thrown from a handler method, Spring tries resolvers in this order:

1. **`@ExceptionHandler` in the same controller** — highest priority
2. **`@ExceptionHandler` in `@ControllerAdvice`** — global fallback (ordered by `@Order`)
3. **`ResponseStatusExceptionResolver`** — checks `@ResponseStatus` on exception class
4. **`DefaultHandlerExceptionResolver`** — handles standard Spring exceptions (e.g., `MethodArgumentNotValidException` → 400)
5. **Spring Boot's `/error` endpoint** — last resort (produces the default whitelabel error page)

**Ordering multiple `@ControllerAdvice`:**

```java
@RestControllerAdvice
@Order(1)  // higher priority (lower number = higher priority)
public class SpecificApiExceptionHandler { ... }

@RestControllerAdvice
@Order(2)  // fallback
public class GenericExceptionHandler { ... }
```

---

## Section 5 — Validation

### Q23. How do `@Valid` and `@Validated` differ?

| Feature | `@Valid` (Jakarta) | `@Validated` (Spring) |
|---|---|---|
| Package | `jakarta.validation` | `org.springframework.validation.annotation` |
| Validation groups | Not supported | Supported |
| Cascaded validation | Yes (nested objects) | Yes |
| Method-level validation | Requires Spring's `MethodValidationPostProcessor` | Enables method-level validation on class |
| Typical usage | `@RequestBody` parameter | Class-level for method param/return validation |

```java
// @Valid — standard usage
@PostMapping("/users")
public ResponseEntity<User> create(@Valid @RequestBody CreateUserRequest req) { ... }

// @Validated with groups
@PostMapping("/users")
public ResponseEntity<User> create(
    @Validated(OnCreate.class) @RequestBody CreateUserRequest req) { ... }
```

**Pitfall**: Using `@Valid` when you need validation groups — it silently applies the `Default` group only, and group-specific constraints are skipped.

---

### Q24. Show common JSR-380 validation annotations and their usage.

```java
public record CreateUserRequest(

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be 2-100 characters")
    String name,

    @NotNull(message = "Email is required")
    @Email(message = "Email must be valid")
    String email,

    @NotNull
    @Min(value = 18, message = "Must be at least 18")
    @Max(value = 150, message = "Invalid age")
    Integer age,

    @NotBlank
    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Invalid phone number")
    String phone,

    @NotNull
    @PastOrPresent(message = "Birth date cannot be in the future")
    LocalDate birthDate,

    @NotEmpty(message = "At least one role required")
    List<@NotBlank String> roles,

    @Valid  // cascade validation into nested object
    @NotNull
    AddressRequest address
) {}
```

| Annotation | Applies to | Null handling |
|---|---|---|
| `@NotNull` | Any type | Fails on null |
| `@NotEmpty` | String, Collection, Map, Array | Fails on null or empty |
| `@NotBlank` | String only | Fails on null, empty, or whitespace |
| `@Size(min,max)` | String, Collection, Map, Array | Passes on null |
| `@Email` | String | Passes on null |
| `@Past`, `@Future` | Temporal types | Passes on null |
| `@Positive`, `@Negative` | Numeric types | Passes on null |
| `@Pattern` | String | Passes on null |

**Critical pitfall**: Most annotations (except `@NotNull`, `@NotEmpty`, `@NotBlank`) **pass on null**. If a field is optional, `@Email` alone won't reject null. Combine: `@NotNull @Email`.

---

### Q25. How do you create a custom validator?

**Step 1 — Custom annotation:**

```java
@Documented
@Constraint(validatedBy = UniqueEmailValidator.class)
@Target({ ElementType.FIELD, ElementType.PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface UniqueEmail {
    String message() default "Email already registered";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

**Step 2 — Implement `ConstraintValidator`:**

```java
@Component
public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {

    private final UserRepository userRepository;

    public UniqueEmailValidator(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public void initialize(UniqueEmail constraintAnnotation) {
        // optional initialization
    }

    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        if (email == null) return true;  // let @NotNull handle nulls
        return !userRepository.existsByEmail(email);
    }
}
```

**Step 3 — Usage:**

```java
public record RegisterRequest(
    @NotBlank String name,
    @NotNull @Email @UniqueEmail String email
) {}
```

**Pitfall**: Custom validators should return `true` for null values and let `@NotNull` handle null checks. This follows the Bean Validation convention and avoids double error messages.

---

### Q26. How do validation groups work?

Validation groups let you apply different constraints in different contexts (e.g., create vs update).

```java
// Marker interfaces
public interface OnCreate {}
public interface OnUpdate {}

public class ProductRequest {

    @Null(groups = OnCreate.class, message = "ID must be null for creation")
    @NotNull(groups = OnUpdate.class, message = "ID required for update")
    private Long id;

    @NotBlank(groups = { OnCreate.class, OnUpdate.class })
    private String name;

    @NotNull(groups = OnCreate.class)
    private BigDecimal price;
}
```

```java
@PostMapping("/products")
public ResponseEntity<Product> create(
        @Validated(OnCreate.class) @RequestBody ProductRequest req) { ... }

@PutMapping("/products/{id}")
public ResponseEntity<Product> update(
        @PathVariable Long id,
        @Validated(OnUpdate.class) @RequestBody ProductRequest req) { ... }
```

**Pitfall**: Constraints without an explicit `groups` attribute belong to the `Default` group. When you use `@Validated(OnCreate.class)`, only `OnCreate` constraints run — NOT `Default` group constraints unless you explicitly include `Default.class`:

```java
@Validated({ OnCreate.class, Default.class })
```

---

### Q27. How does method-level validation work with `@Validated`?

```java
@RestController
@Validated  // enables method-level validation via AOP proxy
@RequestMapping("/search")
public class SearchController {

    @GetMapping
    public List<Product> search(
            @RequestParam @NotBlank @Size(max = 100) String query,
            @RequestParam @Min(1) @Max(100) int limit) {
        return searchService.search(query, limit);
    }

    // Return value validation
    @GetMapping("/{id}")
    @NotNull  // validates return is not null
    public Product findById(@PathVariable @Positive Long id) {
        return service.findById(id).orElse(null);
    }
}
```

**How it works**: `@Validated` on the class triggers Spring's `MethodValidationPostProcessor`, which creates an AOP proxy that validates method parameters and return values.

**Exception thrown**: `ConstraintViolationException` (not `MethodArgumentNotValidException`). You need a separate handler:

```java
@ExceptionHandler(ConstraintViolationException.class)
public ProblemDetail handleConstraintViolation(ConstraintViolationException ex) {
    ProblemDetail pd = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
    pd.setProperty("violations", ex.getConstraintViolations().stream()
        .map(v -> Map.of(
            "path", v.getPropertyPath().toString(),
            "message", v.getMessage()))
        .toList());
    return pd;
}
```

**Interview trap**: `@Valid @RequestBody` validation failures throw `MethodArgumentNotValidException`, while `@Validated` method-level param validation throws `ConstraintViolationException`. You need handlers for BOTH.

---

## Section 6 — Response Customization

### Q28. How do you add custom response headers?

**Option 1 — Via `ResponseEntity`:**

```java
@GetMapping("/reports/{id}")
public ResponseEntity<Report> getReport(@PathVariable Long id) {
    Report report = service.findById(id);
    return ResponseEntity.ok()
        .header("X-Report-Generated", Instant.now().toString())
        .header("X-Report-Version", String.valueOf(report.getVersion()))
        .headers(httpHeaders -> {
            httpHeaders.set("X-Custom-1", "value1");
            httpHeaders.set("X-Custom-2", "value2");
        })
        .cacheControl(CacheControl.maxAge(1, TimeUnit.HOURS))
        .body(report);
}
```

**Option 2 — Via `HttpServletResponse` injection:**

```java
@GetMapping("/download/{fileId}")
public Resource download(@PathVariable String fileId, HttpServletResponse response) {
    response.setHeader("Content-Disposition", "attachment; filename=\"report.pdf\"");
    return storageService.loadAsResource(fileId);
}
```

**Option 3 — Global headers via `Filter` or `HandlerInterceptor`:**

```java
@Component
public class ApiHeaderInterceptor implements HandlerInterceptor {
    @Override
    public void postHandle(HttpServletRequest req, HttpServletResponse res,
                           Object handler, ModelAndView mav) {
        res.setHeader("X-API-Version", "2.0");
        res.setHeader("X-Response-Time", String.valueOf(System.currentTimeMillis()));
    }
}
```

---

### Q29. Explain `@ResponseStatus` at the method level vs exception level.

**On a handler method** — sets the response status code:

```java
@PostMapping("/notifications")
@ResponseStatus(HttpStatus.ACCEPTED)  // 202 instead of default 200
public void sendNotification(@RequestBody NotificationRequest req) {
    notificationService.enqueue(req);
    // no return value needed; Spring returns 202 with empty body
}

@DeleteMapping("/cache")
@ResponseStatus(HttpStatus.NO_CONTENT)  // 204
public void clearCache() {
    cacheManager.clear();
}
```

**On an exception class** — Spring returns this status when the exception escapes unhandled:

```java
@ResponseStatus(value = HttpStatus.CONFLICT, reason = "Resource already exists")
public class DuplicateResourceException extends RuntimeException { ... }
```

**Pitfall**: If a method has `@ResponseStatus(HttpStatus.CREATED)` AND returns `ResponseEntity.ok(body)`, `ResponseEntity` wins — you get 200 OK, not 201 Created. `ResponseEntity` always overrides `@ResponseStatus`.

---

### Q30. How do you implement streaming responses?

**`StreamingResponseBody` — for large file downloads without buffering:**

```java
@GetMapping("/export")
public ResponseEntity<StreamingResponseBody> exportData() {
    StreamingResponseBody stream = outputStream -> {
        try (var cursor = dataService.openCursor()) {
            while (cursor.hasNext()) {
                byte[] chunk = serialize(cursor.next());
                outputStream.write(chunk);
                outputStream.flush();
            }
        }
    };

    return ResponseEntity.ok()
        .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=export.csv")
        .contentType(MediaType.APPLICATION_OCTET_STREAM)
        .body(stream);
}
```

**`ResponseBodyEmitter` — push multiple objects over time:**

```java
@GetMapping("/live-feed")
public ResponseBodyEmitter liveFeed() {
    ResponseBodyEmitter emitter = new ResponseBodyEmitter(60_000L); // 60s timeout

    executor.execute(() -> {
        try {
            for (int i = 0; i < 100; i++) {
                emitter.send(dataService.getUpdate(i), MediaType.APPLICATION_JSON);
                Thread.sleep(1000);
            }
            emitter.complete();
        } catch (Exception e) {
            emitter.completeWithError(e);
        }
    });

    return emitter;
}
```

**`SseEmitter` — Server-Sent Events (SSE):**

```java
@GetMapping(value = "/events", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public SseEmitter streamEvents() {
    SseEmitter emitter = new SseEmitter(0L); // no timeout

    eventBus.subscribe(event -> {
        try {
            emitter.send(SseEmitter.event()
                .id(event.getId())
                .name(event.getType())
                .data(event.getPayload())
                .reconnectTime(5000));
        } catch (IOException e) {
            emitter.completeWithError(e);
        }
    });

    return emitter;
}
```

| Mechanism | Use Case | Content Type |
|---|---|---|
| `StreamingResponseBody` | Large file downloads | `application/octet-stream` |
| `ResponseBodyEmitter` | Multiple objects pushed over time | Configurable |
| `SseEmitter` | Real-time events to browser | `text/event-stream` |

---

## Section 7 — Advanced REST Patterns

### Q31. What are the common API versioning strategies and their trade-offs?

**Strategy 1 — URI Path Versioning:**

```java
@RestController
@RequestMapping("/api/v1/products")
public class ProductControllerV1 { ... }

@RestController
@RequestMapping("/api/v2/products")
public class ProductControllerV2 { ... }
```

**Strategy 2 — Header Versioning (custom header):**

```java
@GetMapping(value = "/products", headers = "X-API-Version=1")
public List<ProductV1> getProductsV1() { ... }

@GetMapping(value = "/products", headers = "X-API-Version=2")
public List<ProductV2> getProductsV2() { ... }
```

**Strategy 3 — Content Negotiation Versioning (media type):**

```java
@GetMapping(value = "/products",
            produces = "application/vnd.myapp.v1+json")
public List<ProductV1> getProductsV1() { ... }

@GetMapping(value = "/products",
            produces = "application/vnd.myapp.v2+json")
public List<ProductV2> getProductsV2() { ... }
```

**Strategy 4 — Query Parameter Versioning:**

```java
@GetMapping(value = "/products", params = "version=1")
public List<ProductV1> getProductsV1() { ... }
```

| Strategy | Pros | Cons |
|---|---|---|
| URI path | Simple, visible, cacheable | URL pollution, violates URI-identifies-resource |
| Custom header | Clean URIs | Not discoverable, harder to test |
| Media type | RESTfully correct, fine-grained | Complex, harder to test in browser |
| Query param | Simple to implement | Not RESTful, caching issues |

**Interviewer focus**: There's no universally "correct" answer. URI versioning is the most widely adopted in practice (GitHub, Twitter). Media type versioning is the most RESTfully pure. What matters is consistency within your API.

---

### Q32. How do you implement pagination and sorting with Spring Data?

```java
@GetMapping("/products")
public Page<ProductResponse> listProducts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "createdAt,desc") String[] sort) {

    Pageable pageable = PageRequest.of(page, size, Sort.by(parseSortOrders(sort)));
    return productRepository.findAll(pageable).map(ProductResponse::from);
}
```

**Or use automatic `Pageable` resolution (Spring Data Web support):**

```java
// Spring auto-resolves Pageable from query params:
// GET /products?page=0&size=20&sort=name,asc&sort=price,desc
@GetMapping("/products")
public Page<ProductResponse> listProducts(Pageable pageable) {
    return productRepository.findAll(pageable).map(ProductResponse::from);
}
```

**Configure defaults:**

```properties
spring.data.web.pageable.default-page-size=20
spring.data.web.pageable.max-page-size=100
spring.data.web.pageable.one-indexed-parameters=true  # page 1 = first page
```

**Returned `Page` JSON structure:**

```json
{
  "content": [ ... ],
  "pageable": { "pageNumber": 0, "pageSize": 20 },
  "totalElements": 156,
  "totalPages": 8,
  "last": false,
  "first": true,
  "numberOfElements": 20
}
```

**Pitfall**: `Page<T>` executes a COUNT query on every request. For large datasets, use `Slice<T>` (no count) or cursor-based pagination for better performance.

**Cursor-based pagination (better for infinite scroll):**

```java
@GetMapping("/feed")
public Slice<Post> getFeed(
        @RequestParam(required = false) Long afterId,
        @RequestParam(defaultValue = "20") int limit) {

    Pageable pageable = PageRequest.of(0, limit);
    return afterId == null
        ? postRepository.findAllByOrderByIdDesc(pageable)
        : postRepository.findByIdLessThanOrderByIdDesc(afterId, pageable);
}
```

---

### Q33. Explain HATEOAS and Spring HATEOAS basics.

HATEOAS (Hypermedia as the Engine of Application State) means responses include links to related actions and resources, so clients can navigate the API dynamically.

```java
// Add spring-boot-starter-hateoas dependency

@RestController
@RequestMapping("/orders")
public class OrderController {

    @GetMapping("/{id}")
    public EntityModel<Order> findById(@PathVariable Long id) {
        Order order = service.findById(id);

        return EntityModel.of(order,
            linkTo(methodOn(OrderController.class).findById(id)).withSelfRel(),
            linkTo(methodOn(OrderController.class).getItems(id)).withRel("items"),
            linkTo(methodOn(OrderController.class).cancel(id)).withRel("cancel")
                .andAffordance(afford(methodOn(OrderController.class).cancel(id)))
        );
    }

    @GetMapping
    public CollectionModel<EntityModel<Order>> listAll() {
        List<EntityModel<Order>> orders = service.findAll().stream()
            .map(order -> EntityModel.of(order,
                linkTo(methodOn(OrderController.class).findById(order.getId())).withSelfRel()))
            .toList();

        return CollectionModel.of(orders,
            linkTo(methodOn(OrderController.class).listAll()).withSelfRel());
    }
}
```

**Response with links:**

```json
{
  "id": 42,
  "status": "PROCESSING",
  "_links": {
    "self": { "href": "http://localhost:8080/orders/42" },
    "items": { "href": "http://localhost:8080/orders/42/items" },
    "cancel": { "href": "http://localhost:8080/orders/42/cancel" }
  }
}
```

**Key Spring HATEOAS types:**

| Type | Purpose |
|---|---|
| `EntityModel<T>` | Wraps a single resource with links |
| `CollectionModel<T>` | Wraps a collection with links |
| `PagedModel<T>` | Wraps a `Page` with pagination links |
| `RepresentationModelAssembler` | Reusable entity-to-model converter |

**Pitfall**: HATEOAS adds complexity. Use it when clients genuinely benefit from dynamic link discovery (e.g., multi-step workflows). For simple CRUD APIs, it may be over-engineering.

---

### Q34. How do you implement filtering and search patterns?

**Approach 1 — Query parameters (simple):**

```java
@GetMapping("/products")
public Page<Product> search(
        @RequestParam(required = false) String name,
        @RequestParam(required = false) String category,
        @RequestParam(required = false) BigDecimal minPrice,
        @RequestParam(required = false) BigDecimal maxPrice,
        Pageable pageable) {

    Specification<Product> spec = Specification.where(null);
    if (name != null)      spec = spec.and(ProductSpec.nameLike(name));
    if (category != null)  spec = spec.and(ProductSpec.inCategory(category));
    if (minPrice != null)  spec = spec.and(ProductSpec.priceGreaterThan(minPrice));
    if (maxPrice != null)  spec = spec.and(ProductSpec.priceLessThan(maxPrice));

    return productRepository.findAll(spec, pageable);
}
```

**Approach 2 — Filter object (cleaner for many params):**

```java
public record ProductFilter(
    String name,
    String category,
    BigDecimal minPrice,
    BigDecimal maxPrice,
    ProductStatus status
) {}

@GetMapping("/products")
public Page<Product> search(@Valid ProductFilter filter, Pageable pageable) {
    return productRepository.findAll(filter.toSpecification(), pageable);
}
```

**Approach 3 — RSQL / FIQL syntax (advanced):**

```
GET /products?filter=category==electronics;price=gt=100;name==*phone*
```

Using a library like `rsql-parser`:

```java
@GetMapping("/products")
public Page<Product> search(@RequestParam String filter, Pageable pageable) {
    Node rootNode = new RSQLParser().parse(filter);
    Specification<Product> spec = rootNode.accept(new CustomRSQLVisitor<>());
    return productRepository.findAll(spec, pageable);
}
```

---

### Q35. How do you implement partial response / field selection?

```java
// GET /users/42?fields=id,name,email
@GetMapping("/users/{id}")
public MappingJacksonValue getUser(
        @PathVariable Long id,
        @RequestParam(required = false) String fields) {

    User user = service.findById(id);
    MappingJacksonValue wrapper = new MappingJacksonValue(user);

    if (fields != null) {
        SimpleFilterProvider filters = new SimpleFilterProvider()
            .addFilter("fieldFilter",
                SimpleBeanPropertyFilter.filterOutAllExcept(fields.split(",")));
        wrapper.setFilters(filters);
    }

    return wrapper;
}
```

**Entity with `@JsonFilter`:**

```java
@JsonFilter("fieldFilter")
public record User(Long id, String name, String email, String phone, Address address) {}
```

**Alternative — Jackson `@JsonView`:**

```java
public class Views {
    public interface Summary {}
    public interface Detail extends Summary {}
}

public class User {
    @JsonView(Views.Summary.class) private Long id;
    @JsonView(Views.Summary.class) private String name;
    @JsonView(Views.Detail.class)  private String email;
    @JsonView(Views.Detail.class)  private Address address;
}

@GetMapping("/users")
@JsonView(Views.Summary.class)
public List<User> listAll() { return service.findAll(); }

@GetMapping("/users/{id}")
@JsonView(Views.Detail.class)
public User findById(@PathVariable Long id) { return service.findById(id); }
```

---

## Section 8 — Performance & Async Patterns

### Q36. How do async REST endpoints work in Spring Boot?

**Option 1 — `CompletableFuture` (most common):**

```java
@GetMapping("/reports/{id}")
public CompletableFuture<Report> generateReport(@PathVariable Long id) {
    return CompletableFuture.supplyAsync(() -> reportService.generate(id));
}
```

**Option 2 — `DeferredResult` (long polling, external event triggers):**

```java
@GetMapping("/notifications/poll")
public DeferredResult<Notification> longPoll() {
    DeferredResult<Notification> result = new DeferredResult<>(30_000L); // 30s timeout
    result.onTimeout(() -> result.setResult(Notification.empty()));

    notificationBus.registerListener(result::setResult);
    return result;
}
```

**Option 3 — `Callable` (simple async):**

```java
@GetMapping("/slow")
public Callable<String> slowEndpoint() {
    return () -> {
        Thread.sleep(5000);
        return "done";
    };
}
```

| Approach | Thread Model | Use Case |
|---|---|---|
| `CompletableFuture` | Releases servlet thread, runs on ForkJoinPool | CPU-bound async work |
| `DeferredResult` | Releases servlet thread, result set externally | Long polling, event-driven |
| `Callable` | Releases servlet thread, runs on `AsyncTaskExecutor` | Simple async I/O |

**Configure async executor:**

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean("apiExecutor")
    public Executor apiTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("api-async-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}
```

**Pitfall**: `CompletableFuture.supplyAsync()` uses the shared `ForkJoinPool` by default. For I/O-bound operations, pass a custom executor: `CompletableFuture.supplyAsync(() -> ..., apiExecutor)`.

---

### Q37. How does request/response compression work in Spring Boot?

```properties
# Enable compression
server.compression.enabled=true
server.compression.min-response-size=1024             # bytes; default is 2048
server.compression.mime-types=application/json,application/xml,text/html,text/plain
```

The embedded server (Tomcat/Jetty/Undertow) handles compression transparently. Responses are compressed only if the client sends `Accept-Encoding: gzip` (or `deflate`).

**For request decompression (client sends compressed body):**

Spring Boot 3.x does NOT auto-decompress request bodies. You need a filter:

```java
@Bean
public FilterRegistrationBean<CommonsRequestLoggingFilter> requestDecompressionFilter() {
    // Use server-specific configuration, e.g., Tomcat:
    // server.tomcat.connector-customizer to enable request decompression
}
```

Or use Undertow which supports `server.undertow.allow-encoded-slash=true`.

**Interview trap**: Compression saves bandwidth but adds CPU overhead. For very small payloads (<1KB), compression can actually increase response size due to encoding overhead. The `min-response-size` threshold prevents this.

---

### Q38. How do you implement caching for REST endpoints?

**HTTP Cache headers (client-side caching):**

```java
@GetMapping("/products/{id}")
public ResponseEntity<Product> getProduct(@PathVariable Long id) {
    Product product = service.findById(id);
    return ResponseEntity.ok()
        .cacheControl(CacheControl.maxAge(30, TimeUnit.MINUTES)
            .cachePublic()
            .noTransform())
        .body(product);
}
```

**Spring Cache abstraction (server-side caching):**

```java
@EnableCaching  // on @Configuration class

@RestController
@RequestMapping("/products")
public class ProductController {

    @GetMapping("/{id}")
    @Cacheable(value = "products", key = "#id")
    public Product findById(@PathVariable Long id) {
        return service.findById(id);  // called only on cache miss
    }

    @PutMapping("/{id}")
    @CachePut(value = "products", key = "#id")
    public Product update(@PathVariable Long id, @RequestBody Product product) {
        return service.update(id, product);  // updates cache with return value
    }

    @DeleteMapping("/{id}")
    @CacheEvict(value = "products", key = "#id")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        service.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Pitfall**: `@Cacheable` on `void` or `ResponseEntity<T>` methods is tricky — the entire `ResponseEntity` is cached including headers like timestamps. Cache the domain object, not the response wrapper.

**Pitfall**: `@Cacheable` uses Spring AOP proxies — self-invocation within the same class bypasses the cache.

```java
// BUG: findById() call bypasses cache because it's a self-invocation
@GetMapping("/products/{id}/summary")
public Summary getSummary(@PathVariable Long id) {
    Product p = findById(id);  // does NOT use cache — same-class call
    return new Summary(p);
}
```

---

### Q39. Explain `RestTemplate` vs `WebClient` vs `RestClient` (Spring 6.1+).

| Feature | `RestTemplate` | `WebClient` | `RestClient` (Spring 6.1+) |
|---|---|---|---|
| Style | Synchronous, blocking | Reactive (non-blocking) | Synchronous, fluent API |
| Status | Maintenance mode | Recommended for reactive | Recommended for servlet |
| HTTP client | Configurable (Apache, OkHttp) | Reactor Netty, Jetty | Configurable |
| API style | Template methods | Fluent builder | Fluent builder |
| Error handling | `ResponseErrorHandler` | `onStatus` | `onStatus` |

**`RestClient` — the modern replacement for `RestTemplate`:**

```java
@Bean
public RestClient restClient(RestClient.Builder builder) {
    return builder
        .baseUrl("https://api.example.com")
        .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
        .requestInterceptor((request, body, execution) -> {
            request.getHeaders().set("X-API-Key", apiKey);
            return execution.execute(request, body);
        })
        .build();
}

// Usage
Product product = restClient.get()
    .uri("/products/{id}", productId)
    .retrieve()
    .body(Product.class);

// With error handling
Product product = restClient.get()
    .uri("/products/{id}", productId)
    .retrieve()
    .onStatus(HttpStatusCode::is4xxClientError, (request, response) -> {
        throw new ProductNotFoundException(productId);
    })
    .body(Product.class);

// POST with body
Product created = restClient.post()
    .uri("/products")
    .body(newProduct)
    .retrieve()
    .body(Product.class);
```

**Pitfall**: `RestTemplate` is NOT deprecated but is in "maintenance mode" — no new features. For new Spring Boot 3.x projects, use `RestClient` (servlet apps) or `WebClient` (reactive apps).

---

### Q40. How do you configure connection pooling for `RestClient` / `RestTemplate`?

The default `SimpleClientHttpRequestFactory` creates a **new connection per request**. For production, use Apache HttpClient 5 or Jetty.

```java
@Bean
public RestClient restClient() {
    var connectionManager = PoolingHttpClientConnectionManagerBuilder.create()
        .setMaxConnTotal(200)           // total connections across all routes
        .setMaxConnPerRoute(50)         // connections per host
        .setDefaultConnectionConfig(
            ConnectionConfig.custom()
                .setConnectTimeout(Timeout.ofSeconds(5))
                .setSocketTimeout(Timeout.ofSeconds(30))
                .build())
        .build();

    CloseableHttpClient httpClient = HttpClients.custom()
        .setConnectionManager(connectionManager)
        .setDefaultRequestConfig(
            RequestConfig.custom()
                .setResponseTimeout(Timeout.ofSeconds(30))
                .build())
        .evictIdleConnections(TimeValue.ofSeconds(30))
        .build();

    return RestClient.builder()
        .requestFactory(new HttpComponentsClientHttpRequestFactory(httpClient))
        .baseUrl("https://api.example.com")
        .build();
}
```

**Key tuning parameters:**

| Parameter | Default | Production Recommendation |
|---|---|---|
| `maxConnTotal` | 25 | 100–200 based on load |
| `maxConnPerRoute` | 5 | 20–50 per downstream host |
| `connectTimeout` | Infinite | 3–5 seconds |
| `socketTimeout` | Infinite | 10–30 seconds |
| `evictIdleConnections` | Never | 30–60 seconds |

**Pitfall**: Without connection pooling, each HTTP call opens a new TCP connection + TLS handshake. This adds 50–200ms per call and can exhaust ephemeral ports under load.

---

## Section 9 — Spring Boot 3.x / Spring 6.x REST-Specific Features

### Q41. What are the key REST-related changes in Spring Boot 3 / Spring 6?

| Change | Details |
|---|---|
| Jakarta EE 10 | `javax.*` → `jakarta.*` across all APIs |
| `PathPatternParser` default | Faster URL matching, `{*path}` syntax, trailing slash off by default |
| `ProblemDetail` (RFC 7807) | Native error response format |
| `RestClient` (6.1) | Modern synchronous HTTP client replacing `RestTemplate` |
| HTTP Interface Clients (6.0) | Declarative HTTP clients via interfaces |
| `@HttpExchange` | Annotation-based HTTP client definitions |
| AOT / GraalVM | Ahead-of-time compilation support |
| Observation API | Micrometer-based request observation replacing older metrics |
| Virtual Thread support | `spring.threads.virtual.enabled=true` (Boot 3.2+) |

**Migration pitfalls:**

```java
// Before (Spring 5)
import javax.validation.Valid;
import javax.servlet.http.HttpServletRequest;

// After (Spring 6)
import jakarta.validation.Valid;
import jakarta.servlet.http.HttpServletRequest;
```

---

### Q42. How do HTTP Interface Clients work in Spring 6?

Declarative HTTP clients — define interfaces, Spring generates the implementation:

```java
public interface ProductApi {

    @GetExchange("/products/{id}")
    Product findById(@PathVariable Long id);

    @GetExchange("/products")
    List<Product> search(@RequestParam String query);

    @PostExchange("/products")
    Product create(@RequestBody CreateProductRequest request);

    @DeleteExchange("/products/{id}")
    ResponseEntity<Void> delete(@PathVariable Long id);
}
```

**Configuration:**

```java
@Bean
public ProductApi productApi(RestClient.Builder builder) {
    RestClient restClient = builder.baseUrl("https://api.example.com").build();
    HttpServiceProxyFactory factory = HttpServiceProxyFactory
        .builderFor(RestClientAdapter.create(restClient))
        .build();
    return factory.createClient(ProductApi.class);
}
```

**Annotations available:** `@GetExchange`, `@PostExchange`, `@PutExchange`, `@PatchExchange`, `@DeleteExchange`, and the general `@HttpExchange`.

**Pitfall**: HTTP Interface Clients currently don't support streaming responses or multipart uploads — use `RestClient` or `WebClient` directly for those.

---

### Q43. How do Virtual Threads improve REST API performance in Spring Boot 3.2+?

Virtual threads (Project Loom, Java 21+) allow one virtual thread per request without the scalability limits of platform threads.

```properties
# Enable virtual threads for all request handling
spring.threads.virtual.enabled=true
```

**Before (platform threads):**
- Tomcat default: 200 threads → max 200 concurrent blocking requests
- Each blocked thread consumes ~1MB stack memory

**After (virtual threads):**
- Millions of concurrent virtual threads possible
- Blocked virtual thread costs almost no memory
- No code changes required — Spring uses virtual threads for request handling

**When virtual threads help most**: APIs doing blocking I/O (database calls, external HTTP calls, file I/O). Virtual threads do NOT help with CPU-bound work.

**Pitfall**: `synchronized` blocks pin virtual threads to carrier threads. Use `ReentrantLock` instead:

```java
// BAD — pins virtual thread
synchronized (lock) {
    database.query();
}

// GOOD — virtual thread friendly
lock.lock();
try {
    database.query();
} finally {
    lock.unlock();
}
```

**Interview trap**: Virtual threads don't replace reactive programming entirely. For truly non-blocking stream processing with backpressure, WebFlux/Reactor is still relevant. Virtual threads simplify the traditional servlet model.

---

## Section 10 — Design & Best Practices Interview Questions

### Q44. How should you design REST resource URIs?

**Best practices:**

```
# Resources are nouns, not verbs
GET    /users                    ✓
GET    /getUsers                 ✗

# Plural nouns for collections
GET    /users                    ✓
GET    /user                     ✗

# Hierarchical for relationships
GET    /users/42/orders          ✓
GET    /getUserOrders?userId=42  ✗

# Use hyphens for readability
GET    /order-items              ✓
GET    /order_items              ✗ (less common in URLs)
GET    /orderItems               ✗

# Query params for filtering/sorting/paging
GET    /products?category=electronics&sort=price,asc

# Actions as sub-resources when needed
POST   /orders/42/cancel         ✓ (action on resource)
POST   /cancelOrder/42           ✗
```

**Anti-patterns to avoid:**

```
/api/v1/getAllProducts              # verb in URI
/api/v1/products/delete/42         # HTTP method in URI
/api/v1/products/42/details/       # redundant (GET already returns details)
/api/v1/product/create             # use POST /products instead
```

---

### Q45. How do you handle bulk/batch operations in a REST API?

```java
// Batch create
@PostMapping("/products/batch")
public ResponseEntity<BatchResult> batchCreate(
        @Valid @RequestBody List<CreateProductRequest> requests) {

    BatchResult result = new BatchResult();
    for (CreateProductRequest req : requests) {
        try {
            Product created = service.create(req);
            result.addSuccess(created.getId());
        } catch (Exception e) {
            result.addFailure(req, e.getMessage());
        }
    }

    HttpStatus status = result.hasFailures()
        ? HttpStatus.MULTI_STATUS    // 207
        : HttpStatus.CREATED;        // 201
    return ResponseEntity.status(status).body(result);
}
```

**Batch result structure:**

```json
{
  "totalRequested": 5,
  "succeeded": 3,
  "failed": 2,
  "results": [
    { "index": 0, "status": 201, "id": 101 },
    { "index": 1, "status": 201, "id": 102 },
    { "index": 2, "status": 400, "error": "Duplicate SKU" },
    { "index": 3, "status": 201, "id": 103 },
    { "index": 4, "status": 422, "error": "Invalid category" }
  ]
}
```

---

### Q46. How do you handle API rate limiting in Spring Boot?

**Using Bucket4j (token bucket algorithm):**

```java
@Component
public class RateLimitInterceptor implements HandlerInterceptor {

    private final Map<String, Bucket> cache = new ConcurrentHashMap<>();

    private Bucket resolveBucket(String clientId) {
        return cache.computeIfAbsent(clientId, key ->
            Bucket.builder()
                .addLimit(Bandwidth.classic(100,
                    Refill.greedy(100, Duration.ofMinutes(1))))
                .build());
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                              Object handler) throws Exception {
        String clientId = request.getRemoteAddr();
        Bucket bucket = resolveBucket(clientId);
        ConsumptionProbe probe = bucket.tryConsumeAndReturnRemaining(1);

        response.addHeader("X-Rate-Limit-Remaining",
            String.valueOf(probe.getRemainingTokens()));

        if (!probe.isConsumed()) {
            response.addHeader("X-Rate-Limit-Retry-After-Seconds",
                String.valueOf(probe.getNanosToWaitForRefill() / 1_000_000_000));
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("{\"error\":\"Rate limit exceeded\"}");
            return false;
        }
        return true;
    }
}
```

**Standard response headers for rate limiting:**

| Header | Meaning |
|---|---|
| `X-Rate-Limit-Limit` | Max requests per window |
| `X-Rate-Limit-Remaining` | Remaining requests in current window |
| `X-Rate-Limit-Reset` | UTC epoch seconds when window resets |
| `Retry-After` | Seconds to wait before retrying (on 429) |

---

### Q47. What is the difference between `@RequestParam`, `@RequestHeader`, and `@CookieValue`?

```java
@GetMapping("/search")
public SearchResult search(
    // From query string: /search?q=spring&lang=en
    @RequestParam String q,
    @RequestParam(defaultValue = "en") String lang,

    // From HTTP headers
    @RequestHeader("Accept-Language") String acceptLanguage,
    @RequestHeader(value = "X-Request-Id", required = false) String requestId,

    // From cookies
    @CookieValue(value = "sessionId", required = false) String sessionId,
    @CookieValue(value = "preferences", defaultValue = "{}") String prefs
) {
    // ...
}
```

| Annotation | Source | Default `required` |
|---|---|---|
| `@RequestParam` | Query string / form body | `true` |
| `@RequestHeader` | HTTP headers | `true` |
| `@CookieValue` | HTTP cookies | `true` |

All three support `required`, `defaultValue`, and `Optional<T>`.

---

### Q48. How do you implement proper error handling for downstream service calls?

```java
@Service
public class PaymentGatewayClient {

    private final RestClient restClient;

    public PaymentResult charge(PaymentRequest request) {
        try {
            return restClient.post()
                .uri("/charges")
                .body(request)
                .retrieve()
                .onStatus(status -> status.value() == 409, (req, res) -> {
                    throw new DuplicatePaymentException(request.getIdempotencyKey());
                })
                .onStatus(HttpStatusCode::is4xxClientError, (req, res) -> {
                    String body = new String(res.getBody().readAllBytes());
                    throw new PaymentClientException("Payment rejected: " + body);
                })
                .onStatus(HttpStatusCode::is5xxServerError, (req, res) -> {
                    throw new PaymentServiceUnavailableException("Gateway error");
                })
                .body(PaymentResult.class);
        } catch (ResourceAccessException e) {
            // Connection timeout, read timeout, DNS failure
            throw new PaymentServiceUnavailableException("Cannot reach payment gateway", e);
        }
    }
}
```

**Map downstream errors to appropriate API responses:**

```java
@RestControllerAdvice
public class DownstreamExceptionHandler {

    @ExceptionHandler(PaymentServiceUnavailableException.class)
    public ProblemDetail handleGatewayDown(PaymentServiceUnavailableException ex) {
        ProblemDetail pd = ProblemDetail.forStatus(HttpStatus.SERVICE_UNAVAILABLE);
        pd.setTitle("Payment service temporarily unavailable");
        pd.setProperty("retryAfter", 30);
        return pd;
    }

    @ExceptionHandler(PaymentClientException.class)
    public ProblemDetail handlePaymentRejected(PaymentClientException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.UNPROCESSABLE_ENTITY, ex.getMessage());
    }
}
```

---

### Q49. How do you document a REST API in Spring Boot?

**Using SpringDoc OpenAPI (successor to Springfox):**

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.6.0</version>
</dependency>
```

```java
@RestController
@RequestMapping("/products")
@Tag(name = "Products", description = "Product management APIs")
public class ProductController {

    @Operation(
        summary = "Find product by ID",
        description = "Returns a single product by its unique identifier",
        responses = {
            @ApiResponse(responseCode = "200", description = "Product found",
                content = @Content(schema = @Schema(implementation = Product.class))),
            @ApiResponse(responseCode = "404", description = "Product not found",
                content = @Content(schema = @Schema(implementation = ProblemDetail.class)))
        }
    )
    @GetMapping("/{id}")
    public ResponseEntity<Product> findById(
            @Parameter(description = "Product ID", example = "42")
            @PathVariable Long id) {
        return service.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
}
```

```properties
# application.properties
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.operations-sorter=method
springdoc.default-produces-media-type=application/json
```

**Access**: Swagger UI at `/swagger-ui.html`, OpenAPI spec at `/api-docs`.

---

### Q50. What are common REST API anti-patterns interviewers ask about?

| Anti-Pattern | Problem | Correct Approach |
|---|---|---|
| Verbs in URIs (`/getUsers`) | Not resource-oriented | `GET /users` |
| `200 OK` for everything | Hides errors from clients | Use correct status codes |
| Returning `null` body for missing resources | Ambiguous — is it null or not found? | `404 Not Found` |
| Exposing JPA entities directly | Tight coupling, security risk, circular refs | Use DTOs |
| Ignoring `Accept` / `Content-Type` headers | Breaks content negotiation | Always handle properly |
| No pagination on collections | Memory explosion, slow responses | Default pagination |
| Swallowing exceptions silently | `200 OK` with empty body on error | Proper error responses |
| `POST` for everything | Loses idempotency guarantees | Use correct HTTP methods |
| No input validation | SQL injection, invalid state | `@Valid` on all inputs |
| Exposing stack traces in errors | Security vulnerability | Custom error responses |

**Entity-to-DTO anti-pattern in detail:**

```java
// BAD — exposing JPA entity
@GetMapping("/users/{id}")
public User findById(@PathVariable Long id) {
    return userRepository.findById(id).orElseThrow();
    // Exposes password hash, internal IDs, lazy-load proxies, circular refs
}

// GOOD — DTO projection
@GetMapping("/users/{id}")
public UserResponse findById(@PathVariable Long id) {
    User user = userRepository.findById(id).orElseThrow();
    return UserResponse.from(user);  // controlled field exposure
}

public record UserResponse(Long id, String name, String email, Instant createdAt) {
    public static UserResponse from(User user) {
        return new UserResponse(user.getId(), user.getName(),
                                user.getEmail(), user.getCreatedAt());
    }
}
```

**Pitfall**: Using `@JsonIgnore` on entity fields instead of DTOs is a fragile workaround — one missed annotation and sensitive data leaks. DTOs provide a clean contract boundary.

---

## Quick Reference Cheat Sheet

### HTTP Status Codes Decision Tree

```
Creating a resource?
  └─ Success → 201 Created + Location header
  └─ Async   → 202 Accepted

Reading a resource?
  └─ Found        → 200 OK + body
  └─ Not modified → 304 Not Modified (ETag match)
  └─ Not found    → 404 Not Found

Updating a resource?
  └─ With body    → 200 OK + updated resource
  └─ Without body → 204 No Content

Deleting a resource?
  └─ Success → 204 No Content

Client error?
  └─ Bad syntax       → 400 Bad Request
  └─ Validation fail  → 400 or 422 Unprocessable Entity
  └─ Not found        → 404 Not Found
  └─ Wrong method     → 405 Method Not Allowed
  └─ Conflict/dup     → 409 Conflict
  └─ Rate limited     → 429 Too Many Requests

Server error?
  └─ Unexpected       → 500 Internal Server Error
  └─ Downstream down  → 502 Bad Gateway / 503 Service Unavailable
```

### Annotation Quick Reference

| Annotation | Purpose | Scope |
|---|---|---|
| `@RestController` | Controller + @ResponseBody | Class |
| `@RequestMapping` | Map URI to handler | Class / Method |
| `@GetMapping` etc. | HTTP method shortcuts | Method |
| `@PathVariable` | URI path segment | Parameter |
| `@RequestParam` | Query string param | Parameter |
| `@RequestBody` | Deserialize request body | Parameter |
| `@RequestPart` | Multipart part | Parameter |
| `@RequestHeader` | HTTP header value | Parameter |
| `@CookieValue` | Cookie value | Parameter |
| `@Valid` | Trigger Bean Validation | Parameter |
| `@Validated` | Validation with groups / method-level | Class / Parameter |
| `@ResponseStatus` | Set HTTP status | Method / Exception class |
| `@ExceptionHandler` | Handle exceptions | Method in Controller/Advice |
| `@RestControllerAdvice` | Global exception handling | Class |
| `@JsonView` | Field filtering by view | Method / Field |
| `@Cacheable` | Cache method result | Method |
| `@CrossOrigin` | CORS configuration | Class / Method |

---

*Last updated: March 2026 — Covers Spring Boot 3.x, Spring 6.x, Java 21+*
