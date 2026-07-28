# Spring Boot — Complete Reference Notes

*A from-fundamentals-to-production guide, written for a Java backend developer moving from service-based work into product/fintech-style engineering.*

---

## Table of Contents

1. [Introduction & Core Concepts](#1-introduction--core-concepts)
2. [Project Setup & Structure](#2-project-setup--structure)
3. [Core Spring Annotations](#3-core-spring-annotations)
4. [Dependency Injection & Bean Lifecycle](#4-dependency-injection--bean-lifecycle)
5. [Auto-Configuration Internals](#5-auto-configuration-internals)
6. [Configuration — Properties, YAML, Profiles](#6-configuration--properties-yaml-profiles)
7. [Building REST APIs](#7-building-rest-apis)
8. [Validation](#8-validation)
9. [Exception Handling](#9-exception-handling)
10. [Spring Data JPA](#10-spring-data-jpa)
11. [Transactions](#11-transactions)
12. [Spring Security & JWT](#12-spring-security--jwt)
13. [Actuator & Observability](#13-actuator--observability)
14. [Aspect-Oriented Programming (AOP)](#14-aspect-oriented-programming-aop)
15. [Caching](#15-caching)
16. [Scheduling & Async](#16-scheduling--async)
17. [Testing](#17-testing)
18. [API Documentation — OpenAPI/Swagger](#18-api-documentation--openapiswagger)
19. [File Upload & Download](#19-file-upload--download)
20. [Microservices Essentials](#20-microservices-essentials)
21. [Deployment & Production Best Practices](#21-deployment--production-best-practices)

---

## 1. Introduction & Core Concepts

### What Spring Boot actually is

Spring Boot is **not a replacement for the Spring Framework** — it's an opinionated layer on top of it that removes boilerplate. Plain Spring requires you to configure `DispatcherServlet`, view resolvers, data sources, transaction managers, etc., by hand (originally in XML). Spring Boot gives you:

- **Auto-configuration** — sensible defaults inferred from what's on your classpath.
- **Starter dependencies** — curated dependency bundles (`spring-boot-starter-web`, `spring-boot-starter-data-jpa`) so you don't hand-pick compatible versions.
- **Embedded servers** — Tomcat/Jetty/Undertow bundled in the JAR, so `java -jar app.jar` just runs — no external servlet container to deploy to.
- **Production-readiness** — Actuator gives you health checks, metrics, and info endpoints out of the box.
- **No XML required** — everything is Java config + annotations.

### Why this matters in interviews

A very common interview question is: *"What is Spring Boot and how is it different from Spring?"* The trap is saying "Spring Boot is faster" or "Spring Boot is a framework" — it's more precise to say: **Spring Boot is Spring Framework + auto-configuration + convention-over-configuration + embedded servers**, aimed at cutting time-to-first-request.

### The core problem Spring (not Boot) solves: Inversion of Control

Traditionally, your code creates its own dependencies:

```java
public class OrderService {
    private PaymentGateway paymentGateway = new RazorpayGateway(); // tight coupling
}
```

With IoC, the **container** creates and injects dependencies:

```java
@Service
public class OrderService {
    private final PaymentGateway paymentGateway;

    // Spring injects whichever PaymentGateway bean exists
    public OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
}
```

This is the foundation everything else in Spring builds on — testability (swap in a mock), loose coupling (swap `RazorpayGateway` for `StripeGateway` without touching `OrderService`), and centralized lifecycle management.

### The Spring Boot request lifecycle (mental model)

```
Client Request
     │
     ▼
Embedded Tomcat (port 8080)
     │
     ▼
DispatcherServlet (front controller)
     │
     ▼
HandlerMapping -> finds matching @RestController method
     │
     ▼
Filters / Interceptors (auth, logging)
     │
     ▼
Controller -> Service -> Repository -> Database
     │
     ▼
Response serialized (Jackson -> JSON) back through DispatcherServlet
```

Keep this diagram in your head — nearly every topic below (annotations, security filters, exception handling, AOP) is really about **where in this pipeline you're allowed to intercept.**

---

## 2. Project Setup & Structure

### Spring Initializr

Every real Spring Boot project starts at [start.spring.io](https://start.spring.io) or via IDE integration. Key choices:

| Choice | Recommendation |
|---|---|
| Build tool | Maven (more common in enterprise/banking shops) or Gradle (faster builds, common at product companies) |
| Language | Java |
| Spring Boot version | Latest stable (avoid SNAPSHOT/milestone builds in production) |
| Packaging | JAR (almost always, unless deploying to an external app server) |
| Java version | 17 or 21 (LTS versions) |

### Standard Maven project structure

```
my-app/
├── src/
│   ├── main/
│   │   ├── java/com/company/myapp/
│   │   │   ├── MyAppApplication.java     <- @SpringBootApplication entry point
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   ├── repository/
│   │   │   ├── entity/ (or model/)
│   │   │   ├── dto/
│   │   │   ├── config/
│   │   │   ├── exception/
│   │   │   └── security/
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       └── static/, templates/
│   └── test/
│       └── java/com/company/myapp/
├── pom.xml
└── README.md
```

**Layered architecture explanation** (this is what interviewers actually probe for — not just "where do files go"):

- **Controller** — HTTP concerns only: request mapping, status codes, calling the service layer. Never put business logic here.
- **Service** — business logic, transaction boundaries (`@Transactional` goes here, not on repositories).
- **Repository** — persistence only. No business logic.
- **DTO** — what crosses the wire. Never expose JPA entities directly in API responses (lazy-loading exceptions, over-exposure of internal fields, tight coupling of API contract to DB schema).
- **Entity** — maps to DB tables.

### Minimal pom.xml (Maven)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
    </parent>

    <groupId>com.company</groupId>
    <artifactId>myapp</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**Note the `spring-boot-starter-parent`** — this is what gives you dependency version management ("BOM" - Bill of Materials) so you never have to specify a version for `spring-boot-starter-*` artifacts yourself; the parent POM pins compatible versions for the entire Spring ecosystem.

### The entry point

```java
package com.company.myapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyAppApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyAppApplication.class, args);
    }
}
```

`@SpringBootApplication` is itself a meta-annotation bundling three things (asked constantly in interviews):

```java
@SpringBootConfiguration   // marks this as a source of bean definitions (a specialised @Configuration)
@EnableAutoConfiguration   // triggers Spring Boot's auto-configuration mechanism
@ComponentScan             // scans this package and sub-packages for @Component, @Service, @Repository, @Controller
public @interface SpringBootApplication { ... }
```

This is why your main class should sit at the **root package** — `@ComponentScan` defaults to scanning downward from wherever `@SpringBootApplication` is placed. Put it in the wrong package and beans in sibling packages silently never get registered.

---

## 3. Core Spring Annotations

Group these mentally — interviewers ask "what's the difference between X and Y" far more than "what does X do."

### Stereotype annotations (component scanning)

| Annotation | Purpose |
|---|---|
| `@Component` | Generic Spring-managed bean |
| `@Service` | Semantic specialization of `@Component` for business logic. No functional difference from `@Component` — purely for readability and future AOP targeting. |
| `@Repository` | Specialization for persistence layer. **Functional difference**: enables automatic translation of persistence-technology-specific exceptions (e.g., `SQLException`, Hibernate's `ConstraintViolationException`) into Spring's unified `DataAccessException` hierarchy via `PersistenceExceptionTranslationPostProcessor`. |
| `@Controller` | Specialization for MVC controllers returning **views** (template names). |
| `@RestController` | `@Controller` + `@ResponseBody` combined — every method's return value is written directly to the HTTP response body (as JSON, typically) instead of being resolved as a view name. |

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // A DataIntegrityViolationException (Spring's) is thrown here,
    // not a raw ConstraintViolationException (Hibernate's) — because @Repository
    // wires in exception translation.
}
```

### Configuration annotations

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new RazorpayGateway(apiKey());
    }

    @Bean
    @Profile("dev")
    public PaymentGateway mockPaymentGateway() {
        return new MockPaymentGateway();
    }
}
```

- `@Configuration` — marks a class as a source of bean definitions (Java-based config, replaces XML `<beans>`).
- `@Bean` — inside a `@Configuration` class, marks a method whose return value should be registered as a Spring bean. Use this for **third-party classes you don't own** (can't put `@Component` on a library class) or when construction needs custom logic.
- `@ComponentScan` — tells Spring which packages to scan for `@Component`-family annotations.
- `@Import` — explicitly pulls in another `@Configuration` class.

### Dependency injection annotations

```java
@Service
public class OrderService {

    private final PaymentGateway paymentGateway; // constructor injection - PREFERRED

    @Autowired // optional on a single constructor since Spring 4.3+, but explicit is fine
    public OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
}
```

- `@Autowired` — marks a constructor, field, or setter for dependency injection.
- `@Qualifier("beanName")` — disambiguates when multiple beans of the same type exist.
- `@Primary` — marks a default bean when multiple candidates exist (loses to an explicit `@Qualifier` at the injection point).
- `@Value("${some.property}")` — injects a value from `application.properties`/environment.

```java
@Service
public class NotificationService {

    @Autowired
    @Qualifier("smsSender")
    private MessageSender sender;

    @Value("${notification.retry.count:3}") // 3 is the default if property is absent
    private int retryCount;
}
```

### Web / MVC annotations

| Annotation | Purpose |
|---|---|
| `@RequestMapping` | Base mapping; can specify method, path, params, headers |
| `@GetMapping` / `@PostMapping` / `@PutMapping` / `@DeleteMapping` / `@PatchMapping` | Shortcuts for `@RequestMapping(method = ...)` |
| `@PathVariable` | Binds a URI template variable |
| `@RequestParam` | Binds a query parameter |
| `@RequestBody` | Deserializes the request body (JSON -> Java object) |
| `@ResponseBody` | Serializes the return value directly to the response body |
| `@RequestHeader` | Binds a specific HTTP header |
| `@ResponseStatus` | Sets the HTTP status code on a method or exception class |

Covered with full examples in [Section 7](#7-building-rest-apis).

### Lombok (not Spring, but nearly universal alongside it)

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String email;
}
```

Interview trap: **never use `@Data` on a JPA `@Entity`.** `@Data` generates `equals()`/`hashCode()` over all fields including relationships, which can trigger infinite recursion on bidirectional associations, and it generates a mutable, all-fields-in-`toString()` class — noisy and dangerous to log (PII, secrets). Prefer explicit `@Getter`/`@Setter` and a manually-scoped `@ToString`/`equals` (usually keyed off `id` only).

---

## 4. Dependency Injection & Bean Lifecycle

### Three injection styles — and why constructor injection wins

```java
// 1. Field injection — AVOID in production code
@Service
public class OrderService {
    @Autowired
    private PaymentGateway paymentGateway;
}

// 2. Setter injection — used for optional dependencies
@Service
public class OrderService {
    private PaymentGateway paymentGateway;

    @Autowired
    public void setPaymentGateway(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
}

// 3. Constructor injection — PREFERRED
@Service
public class OrderService {
    private final PaymentGateway paymentGateway;

    public OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }
}
```

**Why constructor injection is the standard answer in interviews:**

1. Fields can be `final` -> immutability, guaranteed initialization, thread-safety by construction.
2. Dependencies are explicit in the constructor signature -> a class with 8 constructor params is an obvious code smell (too many responsibilities); field injection hides this.
3. Testable without Spring — you can `new OrderService(mockGateway)` in a plain JUnit test, no reflection needed.
4. Fails fast — a missing bean is a compile-time-adjacent failure (app won't start) rather than a runtime `NullPointerException` discovered later.

With Lombok, this collapses to one line:

```java
@Service
@RequiredArgsConstructor // generates a constructor for all `final` fields
public class OrderService {
    private final PaymentGateway paymentGateway;
    private final OrderRepository orderRepository;
}
```

### Bean scopes

| Scope | Meaning |
|---|---|
| `singleton` (default) | One instance per Spring container |
| `prototype` | New instance every time the bean is requested |
| `request` | One instance per HTTP request (web-aware contexts only) |
| `session` | One instance per HTTP session |

```java
@Bean
@Scope("prototype")
public ReportGenerator reportGenerator() {
    return new ReportGenerator(); // fresh, stateful instance every injection
}
```

Interview trap: injecting a `prototype`-scoped bean into a `singleton`-scoped bean naively gives you the **same prototype instance forever**, because the singleton is only constructed once and the injection happens once at that time. Fix: use `ObjectProvider<T>` / `@Lookup` to fetch a fresh instance on each method call.

```java
@Service
public class ReportService {
    private final ObjectProvider<ReportGenerator> generatorProvider;

    public ReportService(ObjectProvider<ReportGenerator> generatorProvider) {
        this.generatorProvider = generatorProvider;
    }

    public void generate() {
        ReportGenerator generator = generatorProvider.getObject(); // fresh instance each call
        generator.run();
    }
}
```

### Bean lifecycle

```
Instantiate bean
     │
     ▼
Populate properties (dependency injection happens here)
     │
     ▼
BeanNameAware / BeanFactoryAware / ApplicationContextAware callbacks (if implemented)
     │
     ▼
@PostConstruct method (or InitializingBean.afterPropertiesSet())
     │
     ▼
     Bean is ready — lives in the container
     │
     ▼
@PreDestroy method (or DisposableBean.destroy()) on container shutdown
```

```java
@Component
public class CacheWarmer {

    @PostConstruct
    public void warmUp() {
        System.out.println("Pre-loading cache at startup...");
    }

    @PreDestroy
    public void cleanUp() {
        System.out.println("Flushing cache before shutdown...");
    }
}
```

`@PostConstruct` is the standard place to run logic that needs the bean's dependencies **already injected** (unlike a constructor, where you sometimes only have raw dependencies but haven't done any setup that depends on them being fully wired into the broader context).

### Circular dependency problem

```java
@Service
public class AService {
    public AService(BService bService) { ... }
}

@Service
public class BService {
    public BService(AService aService) { ... } // circular!
}
```

With constructor injection this **fails at startup** with `BeanCurrentlyInCreationException` — which is actually a good thing; it surfaces a design flaw immediately rather than working "by accident" via field injection (Spring can sometimes resolve circular field-injected dependencies using early bean references, which just delays the pain). The correct fix is almost always to **refactor** — extract shared logic into a third class both depend on — not to add `@Lazy` as a band-aid, though `@Lazy` is the quick technical escape hatch if you're stuck.

---

## 5. Auto-Configuration Internals

This is the section that separates "I use Spring Boot" from "I understand Spring Boot" — a strong differentiator at product companies.

### How `@EnableAutoConfiguration` actually works

1. Spring Boot scans `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (older versions used `spring.factories`) across all JARs on the classpath.
2. This file lists hundreds of `@Configuration` classes — e.g., `DataSourceAutoConfiguration`, `JpaRepositoriesAutoConfiguration`, `WebMvcAutoConfiguration`.
3. Each one is annotated with **conditional annotations** that decide whether it should actually activate.

```java
@Configuration
@ConditionalOnClass(DataSource.class)               // only if this class is on the classpath
@ConditionalOnMissingBean(DataSource.class)          // only if the user hasn't defined their own
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {

    @Bean
    @ConditionalOnProperty(prefix = "spring.datasource", name = "url")
    public DataSource dataSource(DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().build();
    }
}
```

**This is why adding `spring-boot-starter-data-jpa` to your `pom.xml` "magically" wires up a `DataSource`, `EntityManagerFactory`, and `TransactionManager` with zero code** — the classes exist on the classpath, the conditions pass, and Spring Boot creates the beans using properties you supply in `application.yml`.

### Key conditional annotations

| Annotation | Activates when... |
|---|---|
| `@ConditionalOnClass` | a given class is present on the classpath |
| `@ConditionalOnMissingClass` | a given class is absent |
| `@ConditionalOnBean` | a given bean already exists in the context |
| `@ConditionalOnMissingBean` | a given bean does **not** exist (lets you override defaults by just defining your own bean) |
| `@ConditionalOnProperty` | a property has a specific value (or is merely present) |
| `@ConditionalOnWebApplication` | the app is a web application |

### Overriding auto-configuration (very common real-world task)

```java
@Configuration
public class CustomDataSourceConfig {

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        config.setMaximumPoolSize(20);
        return new HikariDataSource(config);
    }
}
```

Because `DataSourceAutoConfiguration`'s bean is annotated `@ConditionalOnMissingBean`, defining your own `DataSource` bean **anywhere** in your app disables the auto-configured one — no need to exclude anything explicitly.

### Explicitly excluding an auto-configuration

```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
public class MyAppApplication { ... }
```

Common real reason: you want a web app with no database at all, and without the exclusion Spring Boot fails startup trying (and failing) to configure a `DataSource` it can't find connection properties for.

### Writing your own starter (occasionally asked at senior levels)

A "starter" is really just: a `pom.xml` artifact + an auto-configuration class + the registration file:

```
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

containing the fully-qualified class name of your `@Configuration` class. Anyone who adds your starter JAR to their classpath gets your auto-configuration for free — exactly how third-party libraries (e.g., a company-internal "audit-logging-starter") plug into the ecosystem.

---

## 6. Configuration — Properties, YAML, Profiles

### application.properties vs application.yml

Both are supported; YAML is generally preferred for readability with nested/hierarchical config.

```properties
# application.properties
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true
logging.level.com.company.myapp=DEBUG
```

```yaml
# application.yml — same config, nested
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: ${DB_PASSWORD}   # pulled from environment variable
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true

logging:
  level:
    com.company.myapp: DEBUG
```

**`${DB_PASSWORD}`** — never hardcode secrets in `application.yml` committed to git. Pull from environment variables, or better, a secrets manager (AWS Secrets Manager, HashiCorp Vault) in production.

### Type-safe configuration with `@ConfigurationProperties`

Instead of scattering `@Value("${...}")` everywhere, group related properties:

```yaml
app:
  payment:
    gateway-url: https://api.razorpay.com
    timeout-ms: 5000
    retry-attempts: 3
```

```java
@Component
@ConfigurationProperties(prefix = "app.payment")
@Getter
@Setter
public class PaymentProperties {
    private String gatewayUrl;
    private int timeoutMs;
    private int retryAttempts;
}
```

```java
@Service
@RequiredArgsConstructor
public class PaymentService {
    private final PaymentProperties paymentProperties;

    public void charge() {
        // paymentProperties.getTimeoutMs(), etc. — type-safe, IDE-autocompletable
    }
}
```

Don't forget `@EnableConfigurationProperties(PaymentProperties.class)` on a `@Configuration` class, or rely on `@Component` scanning to pick it up (both work; `@ConfigurationProperties` classes are commonly just annotated `@Component` directly, as shown above, in modern Spring Boot).

### Profiles

Profiles let you swap configuration per environment without code changes.

```
application.yml           <- shared/common config
application-dev.yml       <- dev overrides
application-prod.yml      <- prod overrides
application-test.yml      <- test overrides
```

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:devdb
  jpa:
    hibernate:
      ddl-auto: update

logging:
  level:
    root: DEBUG
```

```yaml
# application-prod.yml
spring:
  datasource:
    url: jdbc:mysql://prod-db:3306/mydb
  jpa:
    hibernate:
      ddl-auto: validate   # NEVER 'update' or 'create' in production

logging:
  level:
    root: WARN
```

Activate via:

```bash
java -jar app.jar --spring.profiles.active=prod
# or as an env var
export SPRING_PROFILES_ACTIVE=prod
```

Profile-specific beans:

```java
@Configuration
public class GatewayConfig {

    @Bean
    @Profile("prod")
    public PaymentGateway razorpayGateway() {
        return new RazorpayGateway();
    }

    @Bean
    @Profile("dev")
    public PaymentGateway mockGateway() {
        return new MockPaymentGateway();
    }
}
```

**Interview trap:** `ddl-auto: update` or `create-drop` in production is one of the most common "bad practice" answers interviewers fish for — it can silently alter/drop production schema. Production should always use `validate` (fail if entity/schema mismatch) with schema changes managed by a migration tool.

### Schema migrations — Flyway / Liquibase (the correct production pattern)

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

```
src/main/resources/db/migration/
├── V1__create_users_table.sql
├── V2__add_email_index.sql
└── V3__add_orders_table.sql
```

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Flyway runs pending migrations automatically on startup, tracks applied versions in a `flyway_schema_history` table, and gives you a reviewable, versioned audit trail of every schema change — essential in a banking/fintech context where schema changes need review and rollback capability.

---

## 7. Building REST APIs

### A complete, realistic controller

```java
@RestController
@RequestMapping("/api/v1/orders")
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;

    @GetMapping
    public ResponseEntity<List<OrderResponse>> getAllOrders(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        return ResponseEntity.ok(orderService.getOrders(page, size));
    }

    @GetMapping("/{orderId}")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable Long orderId) {
        return ResponseEntity.ok(orderService.getOrder(orderId));
    }

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@Valid @RequestBody CreateOrderRequest request) {
        OrderResponse created = orderService.createOrder(request);
        URI location = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(created.getId())
                .toUri();
        return ResponseEntity.created(location).body(created);
    }

    @PutMapping("/{orderId}")
    public ResponseEntity<OrderResponse> updateOrder(
            @PathVariable Long orderId,
            @Valid @RequestBody UpdateOrderRequest request) {
        return ResponseEntity.ok(orderService.updateOrder(orderId, request));
    }

    @DeleteMapping("/{orderId}")
    public ResponseEntity<Void> deleteOrder(@PathVariable Long orderId) {
        orderService.deleteOrder(orderId);
        return ResponseEntity.noContent().build();
    }
}
```

### Why `ResponseEntity<T>` instead of returning the object directly

```java
// Weaker — status is always 200, no control over headers
@GetMapping("/{id}")
public OrderResponse getOrder(@PathVariable Long id) {
    return orderService.getOrder(id);
}

// Correct — full control over status code, headers, body
@GetMapping("/{id}")
public ResponseEntity<OrderResponse> getOrder(@PathVariable Long id) {
    return ResponseEntity.ok(orderService.getOrder(id));
}
```

`ResponseEntity` lets you correctly express **201 Created** with a `Location` header on POST, **204 No Content** on DELETE, **404** vs **200**, custom headers (rate-limit counters, pagination totals) — all things a real API contract needs and a bare return type can't express.

### DTOs — never expose entities

```java
// Entity — internal, maps to DB
@Entity
@Table(name = "orders")
public class Order {
    @Id @GeneratedValue
    private Long id;
    private BigDecimal amount;
    private String internalNotes;   // should NEVER reach the client
    @ManyToOne
    private User user;              // lazy-loaded — serializing this directly can blow up
}

// DTO — the actual API contract
public record OrderResponse(
        Long id,
        BigDecimal amount,
        String status,
        Instant createdAt
) {}
```

```java
// mapping — manual, or via MapStruct in larger codebases
public OrderResponse toResponse(Order order) {
    return new OrderResponse(order.getId(), order.getAmount(), order.getStatus(), order.getCreatedAt());
}
```

Reasons this matters (a strong interview answer): entities carry lazy-loaded associations that throw `LazyInitializationException` if serialized outside a transaction; entities expose internal-only fields; and coupling your API contract directly to your DB schema means every migration risks becoming a breaking API change.

### Request/response binding annotations, side by side

```java
@GetMapping("/users/{userId}/orders")
public List<OrderResponse> getUserOrders(
        @PathVariable Long userId,                             // from the URI path: /users/42/orders
        @RequestParam(required = false) String status,         // from ?status=PAID
        @RequestHeader("X-Request-Id") String requestId) {      // from an HTTP header
    ...
}

@PostMapping("/users")
public UserResponse createUser(@RequestBody CreateUserRequest request) { // from the JSON body
    ...
}
```

### Content negotiation

Spring Boot defaults to JSON via Jackson (`spring-boot-starter-web` pulls it in transitively). You rarely need to configure this, but it's worth knowing `@RequestMapping(produces = "application/json")` / `consumes` exist for explicit contracts, and that returning XML requires `jackson-dataformat-xml` on the classpath plus the client sending an `Accept: application/xml` header.

### Global base path & versioning

```java
@RestController
@RequestMapping("/api/v1/orders")   // v1 baked into the path — simplest, most common versioning strategy
public class OrderController { ... }
```

Alternative approaches (worth knowing, rarely needed for CRUD APIs): header-based versioning (`Accept: application/vnd.company.v2+json`) or query-param versioning (`?version=2`) — URI versioning wins on simplicity and is what most public APIs (Stripe, Razorpay) use in practice.

---

## 8. Validation

### Bean Validation (JSR-380 / Jakarta Validation)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

```java
public record CreateOrderRequest(

        @NotNull(message = "userId is required")
        Long userId,

        @NotNull @DecimalMin(value = "0.01", message = "amount must be positive")
        BigDecimal amount,

        @NotBlank(message = "currency is required")
        @Pattern(regexp = "INR|USD|EUR", message = "unsupported currency")
        String currency,

        @Email(message = "invalid email format")
        String notifyEmail,

        @Size(min = 1, max = 500)
        String note
) {}
```

```java
@PostMapping
public ResponseEntity<OrderResponse> createOrder(@Valid @RequestBody CreateOrderRequest request) {
    // if validation fails, Spring throws MethodArgumentNotValidException
    // BEFORE this method body ever executes
    return ResponseEntity.ok(orderService.createOrder(request));
}
```

**`@Valid` is what triggers validation** — without it, the constraint annotations on the DTO are inert. This is a common gotcha: adding `@NotNull` to a field does nothing unless the controller parameter is also annotated `@Valid` (or `@Validated` for method-level / group validation).

### Common constraint annotations

| Annotation | Checks |
|---|---|
| `@NotNull` | value is not null |
| `@NotEmpty` | not null and not empty (collections, strings) |
| `@NotBlank` | not null, not empty, and not just whitespace (strings only) |
| `@Size(min, max)` | length/size bounds |
| `@Min` / `@Max` | numeric bounds |
| `@DecimalMin` / `@DecimalMax` | for `BigDecimal` bounds |
| `@Email` | basic email format |
| `@Pattern(regexp = ...)` | regex match |
| `@Past` / `@Future` | date constraints |
| `@Positive` / `@PositiveOrZero` | numeric sign constraints |

### Validating path variables and request params

```java
@RestController
@Validated // required at the CLASS level for method-parameter validation to work
public class OrderController {

    @GetMapping("/orders/{orderId}")
    public OrderResponse getOrder(@PathVariable @Positive Long orderId) {
        ...
    }
}
```

### Custom validators

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PanNumberValidator.class)
public @interface ValidPan {
    String message() default "invalid PAN number format";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class PanNumberValidator implements ConstraintValidator<ValidPan, String> {
    private static final Pattern PAN_PATTERN = Pattern.compile("[A-Z]{5}[0-9]{4}[A-Z]{1}");

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        return value != null && PAN_PATTERN.matcher(value).matches();
    }
}
```

```java
public record KycRequest(
        @ValidPan String panNumber
) {}
```

This pattern — a custom annotation + a `ConstraintValidator` implementation — is exactly how domain-specific validation (PAN numbers, IFSC codes, GSTIN) gets built in Indian fintech codebases, and it's a great thing to have ready as a talking point in interviews.

### Turning validation failures into a clean error response

Covered fully in [Section 9](#9-exception-handling) — but the key point here is: `MethodArgumentNotValidException` is what `@Valid` throws on failure, and it needs a `@ExceptionHandler` to turn into a well-formed 400 response instead of a generic Spring stack trace.

---

## 9. Exception Handling

### The centralized pattern: `@RestControllerAdvice`

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
                HttpStatus.NOT_FOUND.value(), "NOT_FOUND", ex.getMessage(), Instant.now());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(InsufficientBalanceException.class)
    public ResponseEntity<ErrorResponse> handleInsufficientBalance(InsufficientBalanceException ex) {
        ErrorResponse error = new ErrorResponse(
                HttpStatus.UNPROCESSABLE_ENTITY.value(), "INSUFFICIENT_BALANCE", ex.getMessage(), Instant.now());
        return ResponseEntity.status(HttpStatus.UNPROCESSABLE_ENTITY).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
                .map(fieldError -> fieldError.getField() + ": " + fieldError.getDefaultMessage())
                .collect(Collectors.joining(", "));
        ErrorResponse error = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(), "VALIDATION_ERROR", message, Instant.now());
        return ResponseEntity.badRequest().body(error);
    }

    @ExceptionHandler(Exception.class) // catch-all — last line of defense
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        ErrorResponse error = new ErrorResponse(
                HttpStatus.INTERNAL_SERVER_ERROR.value(), "INTERNAL_ERROR",
                "An unexpected error occurred", Instant.now());
        // log the FULL exception internally — never leak stack traces to the client
        log.error("Unhandled exception", ex);
        return ResponseEntity.internalServerError().body(error);
    }
}
```

```java
public record ErrorResponse(int status, String code, String message, Instant timestamp) {}
```

**Why `@RestControllerAdvice` instead of try/catch in every controller method:** centralizes error-formatting logic in one place, keeps controllers focused on the happy path, and guarantees a **consistent error response shape** across the entire API — which API consumers (and API gateway/monitoring tooling) depend on.

### Custom domain exceptions

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

public class InsufficientBalanceException extends RuntimeException {
    public InsufficientBalanceException(String message) {
        super(message);
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderResponse getOrder(Long id) {
        Order order = orderRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Order not found: " + id));
        return toResponse(order);
    }
}
```

### Checked vs. unchecked exceptions in Spring apps

Spring's own exception hierarchy (`DataAccessException` and friends) is entirely **unchecked** (`RuntimeException`), and this is a deliberate design choice worth quoting in interviews: checked exceptions force every layer between the throw site and the catch site to declare `throws`, which couples layers together and makes refactoring painful. Custom domain exceptions in Spring apps should almost always extend `RuntimeException`.

### `@ResponseStatus` — the lightweight alternative

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

This sets the status automatically without a `@RestControllerAdvice` handler — fine for very small apps, but for anything with a real API contract, prefer the centralized handler above so you also control the **response body shape**, not just the status code.

### HandlerExceptionResolver — the mechanism underneath

Worth knowing at a senior level: `@ExceptionHandler`/`@RestControllerAdvice` works via `ExceptionHandlerExceptionResolver`, one of several `HandlerExceptionResolver` implementations Spring MVC tries in order when a controller method throws. This is the same extension point Spring Security's filter chain plugs into for auth failures — which is why 401/403 responses sometimes need to be handled *differently* (via `AuthenticationEntryPoint`/`AccessDeniedHandler`, see [Section 12](#12-spring-security--jwt)) rather than through `@RestControllerAdvice` — security filter exceptions happen **before** `DispatcherServlet` hands off to a controller at all.

---

## 10. Spring Data JPA

### Entity mapping basics

```java
@Entity
@Table(name = "orders", indexes = @Index(name = "idx_user_id", columnList = "user_id"))
@Getter @Setter @NoArgsConstructor
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "amount", nullable = false, precision = 19, scale = 2)
    private BigDecimal amount;

    @Enumerated(EnumType.STRING)   // ALWAYS STRING, never ORDINAL — see trap below
    @Column(nullable = false)
    private OrderStatus status;

    @ManyToOne(fetch = FetchType.LAZY)   // ALWAYS LAZY by default for @ManyToOne/@OneToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @CreationTimestamp
    @Column(updatable = false)
    private Instant createdAt;

    @UpdateTimestamp
    private Instant updatedAt;

    @Version // optimistic locking — see below
    private Long version;
}
```

**Interview trap #1 — `EnumType.ORDINAL`:** storing an enum's ordinal (0, 1, 2...) instead of its name means **reordering the enum, or inserting a new value in the middle, silently corrupts existing data** (row that said `PENDING` now reads as `PAID` after a code change). Always use `EnumType.STRING`.

**Interview trap #2 — fetch types:** `@ManyToOne` and `@OneToOne` default to `FetchType.EAGER`, but `@OneToMany` and `@ManyToMany` default to `FetchType.LAZY`. The universally-recommended production default is to make **everything `LAZY`** explicitly and fetch what you need on purpose (via `JOIN FETCH` or an entity graph) — EAGER associations silently balloon queries as your object graph grows.

### Relationships

```java
// One-to-Many (Order has many OrderItems)
@Entity
public class Order {
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();

    // Helper methods to keep both sides of the relationship in sync
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
    }
}

@Entity
public class OrderItem {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;
}
```

```java
// Many-to-Many (Student <-> Course) via an explicit join entity — PREFERRED over @ManyToMany
@Entity
public class Enrollment {
    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    private Student student;

    @ManyToOne
    private Course course;

    private LocalDate enrolledOn;   // extra data on the relationship itself
}
```

**Why an explicit join entity beats `@ManyToMany` directly:** real-world many-to-many relationships almost always need extra columns eventually (`enrolledOn`, `role`, `status`) — modeling it as two `@OneToMany`/`@ManyToOne` pairs through a join entity from day one avoids a painful migration later, and gives you a queryable, first-class object for the relationship itself.

### Cascade types and `orphanRemoval`

| Cascade | Meaning |
|---|---|
| `PERSIST` | saving the parent also saves new children |
| `MERGE` | updating the parent also updates children |
| `REMOVE` | deleting the parent also deletes children |
| `ALL` | all of the above (+ `REFRESH`, `DETACH`) |

`orphanRemoval = true` additionally deletes a child when it's **removed from the collection**, even if the parent itself isn't deleted (e.g., `order.getItems().remove(item)` triggers a DELETE on that `OrderItem` row).

### Repositories

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    // Derived query — Spring Data generates the SQL from the method name
    List<Order> findByUserIdAndStatus(Long userId, OrderStatus status);

    Optional<Order> findByOrderReference(String reference);

    boolean existsByOrderReference(String reference);

    long countByStatus(OrderStatus status);

    // JPQL — for anything more complex than a derived query can express
    @Query("SELECT o FROM Order o WHERE o.amount > :minAmount AND o.createdAt >= :since")
    List<Order> findLargeRecentOrders(@Param("minAmount") BigDecimal minAmount, @Param("since") Instant since);

    // JOIN FETCH to avoid N+1 — fetches Order + User in a single query
    @Query("SELECT o FROM Order o JOIN FETCH o.user WHERE o.status = :status")
    List<Order> findByStatusWithUser(@Param("status") OrderStatus status);

    // Native SQL — escape hatch for DB-specific features
    @Query(value = "SELECT * FROM orders WHERE amount > :minAmount ORDER BY amount DESC LIMIT 10", nativeQuery = true)
    List<Order> findTop10ByAmount(@Param("minAmount") BigDecimal minAmount);

    // Modifying query — needs @Modifying + a transaction
    @Modifying
    @Query("UPDATE Order o SET o.status = :status WHERE o.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") OrderStatus status);
}
```

### The N+1 problem (asked in almost every mid-level Java interview)

```java
// N+1 in action:
List<Order> orders = orderRepository.findAll();       // 1 query
for (Order order : orders) {
    System.out.println(order.getUser().getName());    // N additional queries — one per order!
}
```

Because `user` is `LAZY`, each `.getUser()` call outside the initial fetch triggers a **separate** SQL query to load that specific user — for 1,000 orders, that's 1,001 queries. Three fixes, in order of preference:

```java
// Fix 1: JOIN FETCH (best for a known, bounded query)
@Query("SELECT o FROM Order o JOIN FETCH o.user")
List<Order> findAllWithUser();

// Fix 2: @EntityGraph (declarative, reusable across methods)
@EntityGraph(attributePaths = {"user"})
List<Order> findByStatus(OrderStatus status);

// Fix 3: batch fetching (good default safety net, doesn't eliminate N+1 but shrinks N to N/batchSize)
// application.yml:
// spring.jpa.properties.hibernate.default_batch_fetch_size: 20
```

### Pagination and sorting

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    Page<Order> findByUserId(Long userId, Pageable pageable);
}
```

```java
@GetMapping
public ResponseEntity<Page<OrderResponse>> getOrders(
        @RequestParam Long userId,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {

    Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
    Page<Order> orders = orderRepository.findByUserId(userId, pageable);
    Page<OrderResponse> response = orders.map(this::toResponse);
    return ResponseEntity.ok(response);
}
```

`Page<T>` gives you `getTotalElements()`, `getTotalPages()`, `hasNext()` for free — Spring Data runs a `COUNT(*)` query alongside the paged query automatically.

### Optimistic vs. pessimistic locking

```java
@Entity
public class Account {
    @Id
    private Long id;
    private BigDecimal balance;

    @Version   // optimistic locking: adds a WHERE version = ? to every UPDATE
    private Long version;
}
```

Optimistic locking (via `@Version`) throws `OptimisticLockException` if two transactions try to update the same row based on stale data — the second writer's `UPDATE ... WHERE id = ? AND version = ?` matches zero rows, and Spring/Hibernate detects this and fails that transaction. This is the default choice for most apps: no locks held, good throughput, contention handled by retry.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT a FROM Account a WHERE a.id = :id")
Optional<Account> findByIdForUpdate(@Param("id") Long id);
```

Pessimistic locking (`SELECT ... FOR UPDATE`) actually blocks other transactions from reading/writing that row until the current transaction commits. **This is the standard answer for "how would you prevent a double-spend / race condition on account balance"** in a fintech interview — pessimistic locking around the balance-check-and-deduct critical section guarantees no other transaction can interleave.

---

## 11. Transactions

### `@Transactional` — where it goes and why

```java
@Service
@RequiredArgsConstructor
public class TransferService {

    private final AccountRepository accountRepository;

    @Transactional
    public void transferFunds(Long fromId, Long toId, BigDecimal amount) {
        Account from = accountRepository.findByIdForUpdate(fromId)
                .orElseThrow(() -> new ResourceNotFoundException("Account not found: " + fromId));
        Account to = accountRepository.findByIdForUpdate(toId)
                .orElseThrow(() -> new ResourceNotFoundException("Account not found: " + toId));

        if (from.getBalance().compareTo(amount) < 0) {
            throw new InsufficientBalanceException("Insufficient balance in account " + fromId);
        }

        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));

        // no explicit save() needed inside a @Transactional method —
        // dirty checking flushes changes to managed entities automatically at commit
    }
}
```

**`@Transactional` belongs on the service layer, not the repository or controller.** The service layer is where a *business operation* (which may touch multiple repositories) is defined — that's the natural transaction boundary. Putting it on individual repository methods would mean `transferFunds` above runs as two separate transactions (one per account), destroying atomicity: a crash between the two updates leaves money debited from one account and never credited to the other.

### The AOP-proxy gotcha: self-invocation

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder(Order order) {
        saveOrder(order);       // calling a @Transactional method THROUGH `this` — the proxy is bypassed!
    }

    @Transactional
    public void saveOrder(Order order) { ... }
}
```

**This is one of the most-tested Spring gotchas in interviews.** `@Transactional` works via a dynamically-generated proxy wrapping your bean. When `placeOrder` calls `saveOrder` via `this.saveOrder(...)` (implicitly, as above), the call **never goes through the proxy** — it's a plain Java method call on the raw object — so `saveOrder`'s own `@Transactional` boundary is silently skipped; it just executes inside whatever transaction (if any) `placeOrder` already established. Fix: move `saveOrder` to a different bean and inject it, or use `AopContext.currentProxy()` (ugly, avoid).

### Propagation

| Propagation | Behavior |
|---|---|
| `REQUIRED` (default) | join the existing transaction if one exists, else start a new one |
| `REQUIRES_NEW` | always suspend any existing transaction and start a fresh one |
| `NESTED` | starts a nested transaction (savepoint) inside an existing one — can roll back independently |
| `MANDATORY` | must run within an existing transaction, throws if none exists |
| `NOT_SUPPORTED` | runs non-transactionally, suspending any existing transaction |

```java
@Service
@RequiredArgsConstructor
public class AuditLogService {

    // Audit logs should be written even if the enclosing business transaction rolls back
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logAction(String action, Long userId) {
        auditRepository.save(new AuditLog(action, userId, Instant.now()));
    }
}
```

`REQUIRES_NEW` here is deliberate: if `placeOrder()` fails and rolls back, you still want the audit trail entry ("order attempt failed") to persist — which requires it to commit in its own, independent transaction.

### Isolation levels

| Level | Prevents |
|---|---|
| `READ_UNCOMMITTED` | nothing — dirty reads possible |
| `READ_COMMITTED` (most DBs' default) | dirty reads |
| `REPEATABLE_READ` | dirty reads + non-repeatable reads |
| `SERIALIZABLE` | dirty reads + non-repeatable reads + phantom reads (strictest, slowest) |

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
public void criticalFinancialOperation() { ... }
```

In practice, most applications leave isolation at the database default (`READ_COMMITTED` for PostgreSQL/MySQL InnoDB) and reach for **explicit locking** (`@Version` or `PESSIMISTIC_WRITE`, see [Section 10](#10-spring-data-jpa)) rather than cranking isolation up to `SERIALIZABLE`, which tanks throughput under contention.

### Rollback rules

```java
@Transactional(rollbackFor = Exception.class) // rollback on ANY exception, including checked ones
public void riskyOperation() throws IOException {
    ...
}
```

**Default behavior:** `@Transactional` only rolls back on unchecked exceptions (`RuntimeException` and `Error`) — a checked exception, by default, **does not** trigger a rollback and the transaction commits anyway. This surprises people constantly; if your method can throw a checked exception and a failed operation should NOT be committed, you must specify `rollbackFor = Exception.class` explicitly.

### readOnly optimization

```java
@Transactional(readOnly = true)
public List<OrderResponse> getOrders() {
    return orderRepository.findAll().stream().map(this::toResponse).toList();
}
```

`readOnly = true` is a hint to Hibernate to skip dirty-checking overhead and can let the driver route to a read-replica in some setups — cheap to add correctly on every read-only service method, and worth mentioning proactively in interviews as a performance-awareness signal.

---

## 12. Spring Security & JWT

### The filter chain — mental model

Spring Security works by inserting a chain of **servlet filters** in front of `DispatcherServlet`. Every request passes through this chain before it ever reaches a controller:

```
Request
   │
   ▼
SecurityContextPersistenceFilter
   │
   ▼
UsernamePasswordAuthenticationFilter / your custom JWT filter
   │
   ▼
ExceptionTranslationFilter  (catches AuthenticationException / AccessDeniedException)
   │
   ▼
FilterSecurityInterceptor  (authorization decision: is this user allowed?)
   │
   ▼
DispatcherServlet -> Controller
```

This is why security errors (401/403) are handled differently from application errors — they're thrown and caught **inside the filter chain**, before a controller (and therefore `@RestControllerAdvice`) is ever involved.

### Basic security configuration (Spring Security 6+, no XML, lambda DSL)

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthFilter jwtAuthFilter;
    private final AuthenticationProvider authenticationProvider;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())   // stateless JWT APIs don't need CSRF protection
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**", "/api/v1/public/**").permitAll()
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.GET, "/api/v1/orders/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))  // no HttpSession — JWT is self-contained
            .authenticationProvider(authenticationProvider)
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint((request, response, authException) ->
                    response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Unauthorized"))
                .accessDeniedHandler((request, response, accessDeniedException) ->
                    response.sendError(HttpServletResponse.SC_FORBIDDEN, "Forbidden")));

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

**`SessionCreationPolicy.STATELESS`** is the key line for a JWT-based API: it tells Spring Security never to create or use an `HttpSession`, because the JWT itself carries all the identity information needed on every request — this is what makes the API horizontally scalable (any instance can validate any request without shared session state).

### UserDetailsService & AuthenticationProvider

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        User user = userRepository.findByEmail(email)
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + email));

        return org.springframework.security.core.userdetails.User.builder()
                .username(user.getEmail())
                .password(user.getPasswordHash())
                .authorities(user.getRoles().stream()
                        .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                        .toList())
                .build();
    }
}
```

```java
@Configuration
@RequiredArgsConstructor
public class AuthConfig {

    private final CustomUserDetailsService userDetailsService;

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(new BCryptPasswordEncoder());
        return provider;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

### JWT generation and validation

```java
@Component
public class JwtService {

    @Value("${jwt.secret}")
    private String secretKey;

    @Value("${jwt.expiration-ms}")
    private long expirationMs;

    public String generateToken(UserDetails userDetails) {
        return Jwts.builder()
                .subject(userDetails.getUsername())
                .claim("roles", userDetails.getAuthorities().stream()
                        .map(GrantedAuthority::getAuthority).toList())
                .issuedAt(new Date())
                .expiration(new Date(System.currentTimeMillis() + expirationMs))
                .signWith(getSigningKey())
                .compact();
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }

    public String extractUsername(String token) {
        return extractClaims(token).getSubject();
    }

    private boolean isTokenExpired(String token) {
        return extractClaims(token).getExpiration().before(new Date());
    }

    private Claims extractClaims(String token) {
        return Jwts.parser().verifyWith(getSigningKey()).build()
                .parseSignedClaims(token).getPayload();
    }

    private SecretKey getSigningKey() {
        return Keys.hmacShaKeyFor(Decoders.BASE64.decode(secretKey));
    }
}
```

### The JWT authentication filter

```java
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                     FilterChain filterChain) throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7);
        String username = jwtService.extractUsername(token);

        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);

            if (jwtService.isTokenValid(token, userDetails)) {
                UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

**`OncePerRequestFilter`** guarantees this filter runs exactly once per request even in forwarding/error-dispatch scenarios — the standard base class for any custom Spring Security filter.

### Login endpoint

```java
@RestController
@RequestMapping("/api/v1/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final JwtService jwtService;
    private final CustomUserDetailsService userDetailsService;

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@Valid @RequestBody LoginRequest request) {
        authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(request.email(), request.password()));

        UserDetails userDetails = userDetailsService.loadUserByUsername(request.email());
        String token = jwtService.generateToken(userDetails);

        return ResponseEntity.ok(new AuthResponse(token));
    }
}
```

`authenticationManager.authenticate(...)` throws `BadCredentialsException` automatically if the password doesn't match — you don't manually compare hashes; `DaoAuthenticationProvider` does that internally using the configured `PasswordEncoder`.

### Method-level security

```java
@Configuration
@EnableMethodSecurity   // enables @PreAuthorize / @PostAuthorize / @Secured
public class MethodSecurityConfig {}
```

```java
@Service
public class AccountService {

    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public Account getAccount(Long userId) {
        // only an ADMIN, or the user themselves, can call this
        ...
    }
}
```

This is the standard way to enforce "a user can only view their own resource" at the **service** layer — defense in depth alongside URL-based rules in `SecurityFilterChain`.

### Password hashing — never roll your own

`BCryptPasswordEncoder` is the standard choice: it's adaptive (configurable work factor to stay slow as hardware improves) and automatically salts each hash. Never use plain MD5/SHA-256 for passwords — they're fast by design, which is exactly wrong for password hashing (makes brute-forcing cheap).

---

## 13. Actuator & Observability

### Adding Actuator

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  endpoint:
    health:
      show-details: when-authorized   # never "always" in production — leaks internal details publicly
  metrics:
    tags:
      application: myapp
```

**By default only `/actuator/health` and `/actuator/info` are exposed** — you must explicitly opt other endpoints in via `management.endpoints.web.exposure.include`. Exposing everything (`include: "*"`) in production is a real security risk (`/actuator/env` can leak secrets, `/actuator/heapdump` can leak in-memory data) — expose only what you need, and put Actuator endpoints behind authentication/a separate management port.

### Key built-in endpoints

| Endpoint | Purpose |
|---|---|
| `/actuator/health` | UP/DOWN status, aggregates all `HealthIndicator` beans (DB, disk space, custom checks) |
| `/actuator/info` | Static app info (version, build) — populate via `info.*` properties or `build-info` goal |
| `/actuator/metrics` | JVM, HTTP, and custom metrics |
| `/actuator/prometheus` | Metrics in Prometheus scrape format (needs `micrometer-registry-prometheus`) |
| `/actuator/env` | Environment properties (sensitive — restrict access) |
| `/actuator/loggers` | View/change log levels **at runtime**, no redeploy |
| `/actuator/threaddump` | JVM thread dump — invaluable for debugging deadlocks/hangs |

### Custom health indicators

```java
@Component
public class PaymentGatewayHealthIndicator implements HealthIndicator {

    private final PaymentGatewayClient client;

    @Override
    public Health health() {
        try {
            client.ping();
            return Health.up().withDetail("gateway", "reachable").build();
        } catch (Exception e) {
            return Health.down(e).withDetail("gateway", "unreachable").build();
        }
    }
}
```

This gets automatically aggregated into the overall `/actuator/health` response — if this indicator reports `DOWN`, the whole app's health check reports `DOWN`, which is exactly what Kubernetes/load balancer readiness probes key off of to pull an unhealthy instance out of rotation.

### Custom metrics with Micrometer

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final MeterRegistry meterRegistry;

    public void createOrder(CreateOrderRequest request) {
        Timer.Sample sample = Timer.start(meterRegistry);
        try {
            // ... business logic
            meterRegistry.counter("orders.created", "currency", request.currency()).increment();
        } finally {
            sample.stop(meterRegistry.timer("orders.creation.time"));
        }
    }
}
```

Micrometer is a **facade** (like SLF4J is for logging) — the same instrumentation code ships metrics to Prometheus, Datadog, CloudWatch, or New Relic depending only on which registry dependency you add, with zero code changes.

### Runtime log-level changes (genuinely useful in production incidents)

```bash
curl -X POST localhost:8080/actuator/loggers/com.company.myapp.service.PaymentService \
  -H "Content-Type: application/json" \
  -d '{"configuredLevel": "DEBUG"}'
```

Flip a package to `DEBUG` mid-incident to get more detail, then flip it back — no restart, no redeploy.

---

## 14. Aspect-Oriented Programming (AOP)

### The problem AOP solves: cross-cutting concerns

Logging, auditing, performance timing, and security checks are needed across dozens of unrelated classes — sprinkling the same boilerplate into every method is repetitive and error-prone. AOP lets you define this logic **once** and declaratively apply it wherever needed.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### Core AOP vocabulary

| Term | Meaning |
|---|---|
| **Aspect** | A module encapsulating a cross-cutting concern (a class annotated `@Aspect`) |
| **Join point** | A point during execution — in Spring AOP, always a method call |
| **Advice** | The action taken at a join point (before/after/around) |
| **Pointcut** | An expression selecting which join points an advice applies to |
| **Weaving** | The process of linking aspects into the target objects (Spring does this at runtime via proxies) |

### A logging/timing aspect

```java
@Aspect
@Component
@Slf4j
public class LoggingAspect {

    @Pointcut("within(com.company.myapp.service..*)")
    public void serviceLayer() {}

    @Around("serviceLayer()")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        String methodName = joinPoint.getSignature().toShortString();

        try {
            Object result = joinPoint.proceed();   // actually invokes the real method
            long duration = System.currentTimeMillis() - start;
            log.info("{} executed in {} ms", methodName, duration);
            return result;
        } catch (Exception e) {
            log.error("{} threw {}: {}", methodName, e.getClass().getSimpleName(), e.getMessage());
            throw e;
        }
    }
}
```

### Advice types

```java
@Aspect
@Component
public class AuditAspect {

    @Before("execution(* com.company.myapp.service.PaymentService.*(..))")
    public void beforePayment(JoinPoint joinPoint) {
        System.out.println("About to call: " + joinPoint.getSignature().getName());
    }

    @AfterReturning(pointcut = "execution(* com.company.myapp.service.PaymentService.*(..))", returning = "result")
    public void afterPaymentSuccess(Object result) {
        System.out.println("Payment call succeeded, result: " + result);
    }

    @AfterThrowing(pointcut = "execution(* com.company.myapp.service.PaymentService.*(..))", throwing = "ex")
    public void afterPaymentFailure(Exception ex) {
        System.out.println("Payment call failed: " + ex.getMessage());
    }

    @After("execution(* com.company.myapp.service.PaymentService.*(..))")
    public void afterPayment() {
        System.out.println("Payment call finished (success or failure)");
    }
}
```

- `@Before` — runs before the method, cannot stop execution (short of throwing).
- `@AfterReturning` — runs after successful completion, has access to the return value.
- `@AfterThrowing` — runs only if the method threw an exception.
- `@After` — runs regardless of outcome (like `finally`).
- `@Around` — full control: can inspect/modify arguments, skip the call entirely, modify the return value, or suppress exceptions. Most powerful, most commonly used for timing/auditing.

### Custom annotation-driven aspects (a very practical, interview-impressive pattern)

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AuditLog {
    String action();
}
```

```java
@Aspect
@Component
@RequiredArgsConstructor
public class AuditLogAspect {

    private final AuditRepository auditRepository;

    @AfterReturning("@annotation(auditLog)")
    public void logAction(JoinPoint joinPoint, AuditLog auditLog) {
        auditRepository.save(new AuditEntry(auditLog.action(), Instant.now()));
    }
}
```

```java
@Service
public class OrderService {

    @AuditLog(action = "ORDER_CREATED")
    public Order createOrder(CreateOrderRequest request) {
        ...
    }
}
```

This is exactly how `@Transactional` and `@Cacheable` themselves work under the hood — Spring provides the annotation, a proxy intercepts calls to methods carrying it, and an aspect (built into Spring's core in those cases) runs the cross-cutting logic. Understanding this pattern means you understand roughly a third of "how does Spring even work" at a deep level.

### Important limitation: proxy-based AOP

Spring's default AOP is **proxy-based**, which means (same root cause as the `@Transactional` self-invocation gotcha in [Section 11](#11-transactions)):

1. Only **public** method calls made **from outside the bean** (through the proxy) are advised.
2. Self-invocation (a method calling another method on `this`) bypasses the proxy and the aspect never fires.
3. `final` classes/methods can't be proxied via CGLIB subclassing (relevant if not using interfaces).

---

## 15. Caching

### Enabling caching

```java
@Configuration
@EnableCaching
public class CacheConfig {}
```

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

### The core annotations

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;

    @Cacheable(value = "products", key = "#productId")
    public Product getProduct(Long productId) {
        System.out.println("Hitting the database..."); // only prints on a cache MISS
        return productRepository.findById(productId)
                .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }

    @CachePut(value = "products", key = "#product.id")   // always runs, refreshes the cache
    public Product updateProduct(Product product) {
        return productRepository.save(product);
    }

    @CacheEvict(value = "products", key = "#productId")
    public void deleteProduct(Long productId) {
        productRepository.deleteById(productId);
    }

    @CacheEvict(value = "products", allEntries = true)
    public void clearAllProductCache() {
        // useful after a bulk import/migration
    }
}
```

- `@Cacheable` — checks the cache first; on a hit, the method body **never executes**, the cached value is returned directly.
- `@CachePut` — always executes the method, then updates the cache with the result (use for writes that must also refresh what's cached).
- `@CacheEvict` — removes an entry (or all entries) from the cache.

### Backing cache providers

By default Spring Boot uses a simple in-memory `ConcurrentHashMap`-based cache (`spring-boot-starter-cache` alone) — fine for a single-instance dev setup, **wrong for production** with multiple app instances, since each instance would have its own independent cache (stale data risk, wasted memory). Production apps almost always plug in Redis:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```yaml
spring:
  cache:
    type: redis
  data:
    redis:
      host: localhost
      port: 6379

  cache:
    redis:
      time-to-live: 600000   # 10 minutes, in ms
```

With Redis as the backing store, **every app instance shares the same cache** — a write from instance A is immediately visible (and cache-invalidatable) from instance B, which is essential once you're running more than one replica behind a load balancer.

### Conditional caching

```java
@Cacheable(value = "products", key = "#productId", condition = "#productId > 0", unless = "#result.price > 10000")
public Product getProduct(Long productId) {
    ...
}
```

`condition` decides whether to even attempt caching (evaluated **before** the method call, based on arguments); `unless` decides whether to skip caching the *result* (evaluated **after**, based on the return value) — useful for not caching very large or sensitive objects.

### Cache stampede awareness (a good thing to bring up proactively)

A naive cache expiring under high load can cause many concurrent requests to all miss simultaneously and hammer the database at once ("cache stampede" / "thundering herd"). Mitigations worth knowing about: staggered TTLs (add jitter so entries don't all expire at the same instant), a distributed lock around cache-population, or a "stale-while-revalidate" pattern (serve the slightly-stale value while one request refreshes it in the background).

---

## 16. Scheduling & Async

### Scheduled tasks

```java
@Configuration
@EnableScheduling
public class SchedulingConfig {}
```

```java
@Component
public class ReconciliationJob {

    @Scheduled(cron = "0 0 2 * * *")   // every day at 2:00 AM
    public void runDailyReconciliation() {
        System.out.println("Running end-of-day reconciliation...");
    }

    @Scheduled(fixedRate = 60000)   // every 60 seconds, measured from START of previous execution
    public void pollExternalGateway() { ... }

    @Scheduled(fixedDelay = 60000)  // every 60 seconds, measured from END of previous execution
    public void syncInventory() { ... }

    @Scheduled(initialDelay = 5000, fixedRate = 30000) // wait 5s after startup, then every 30s
    public void warmCaches() { ... }
}
```

**`fixedRate` vs `fixedDelay`** is a classic interview question: `fixedRate` schedules the *next* run based on when the *previous run started* (so if a task takes longer than the interval, executions can queue up back-to-back); `fixedDelay` waits for the interval to elapse *after the previous run finishes* — safer default when a task's duration is variable and you don't want overlapping runs.

### Cron expression cheat sheet

```
 ┌───────────── second (0-59)
 │ ┌───────────── minute (0-59)
 │ │ ┌───────────── hour (0-23)
 │ │ │ ┌───────────── day of month (1-31)
 │ │ │ │ ┌───────────── month (1-12)
 │ │ │ │ │ ┌───────────── day of week (0-7, 0/7=Sunday)
 │ │ │ │ │ │
 0 0 2 * * *        every day at 2:00:00 AM
 0 */15 * * * *     every 15 minutes
 0 0 9 * * MON-FRI  9:00 AM, weekdays only
 0 0 0 1 * *        midnight on the 1st of every month
```

### The single-threaded scheduler trap

By default, `@Scheduled` methods all run on a **single shared thread** — if `runDailyReconciliation` takes 10 minutes, every other `@Scheduled` task in the app is blocked and delayed behind it. Fix:

```java
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(Executors.newScheduledThreadPool(10));
    }
}
```

### Async execution

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

```java
@Service
public class NotificationService {

    @Async("taskExecutor")
    public CompletableFuture<Void> sendEmailAsync(String to, String subject, String body) {
        // runs on a thread from the pool above, NOT the calling thread
        emailClient.send(to, subject, body);
        return CompletableFuture.completedFuture(null);
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final NotificationService notificationService;

    public void placeOrder(Order order) {
        saveOrder(order);
        notificationService.sendEmailAsync(order.getUserEmail(), "Order confirmed", "..."); // fire-and-forget
        // this line returns immediately — the caller doesn't wait for the email to send
    }
}
```

**Same self-invocation gotcha as `@Transactional`:** `@Async` is also proxy-based. Calling an `@Async` method on `this` from within the same class silently runs synchronously.

**`@Async` methods returning `void` swallow exceptions silently** unless you configure an `AsyncUncaughtExceptionHandler` — a very real production gotcha (a background email-send failure disappears without a trace). Prefer returning `CompletableFuture<T>` so exceptions surface when the future is joined/handled, or explicitly register a custom handler.

---

## 17. Testing

### The testing pyramid in a Spring Boot context

```
        ▲
       / \        Few — full end-to-end (real HTTP, real/near-real DB)
      /E2E\
     /-----\
    /  ITs  \      Some — @SpringBootTest / @WebMvcTest / @DataJpaTest
   /---------\
  / Unit Tests \   Many — plain JUnit + Mockito, no Spring context
 /-------------\
```

**Rule of thumb:** the more of your test suite is unit tests with no Spring context, the faster your build is. Spring context startup (`@SpringBootTest`) is expensive (seconds per test class) — reserve it for genuine integration coverage.

### Pure unit test — no Spring context at all

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private PaymentGateway paymentGateway;

    @InjectMocks
    private OrderService orderService;

    @Test
    void createOrder_shouldSaveAndReturnOrder_whenPaymentSucceeds() {
        // Arrange
        CreateOrderRequest request = new CreateOrderRequest(1L, BigDecimal.TEN, "INR");
        when(paymentGateway.charge(any())).thenReturn(true);
        when(orderRepository.save(any(Order.class))).thenAnswer(inv -> inv.getArgument(0));

        // Act
        OrderResponse response = orderService.createOrder(request);

        // Assert
        assertThat(response.amount()).isEqualByComparingTo(BigDecimal.TEN);
        verify(orderRepository, times(1)).save(any(Order.class));
        verify(paymentGateway).charge(request);
    }

    @Test
    void createOrder_shouldThrow_whenPaymentFails() {
        when(paymentGateway.charge(any())).thenReturn(false);
        CreateOrderRequest request = new CreateOrderRequest(1L, BigDecimal.TEN, "INR");

        assertThrows(PaymentFailedException.class, () -> orderService.createOrder(request));
        verify(orderRepository, never()).save(any());
    }
}
```

Constructor injection (from [Section 4](#4-dependency-injection--bean-lifecycle)) is exactly what makes this test possible without booting Spring at all — `@InjectMocks` just calls the constructor with the `@Mock`s.

### `@WebMvcTest` — controller layer only

```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean   // replaces the real OrderService bean in the (sliced) Spring context with a mock
    private OrderService orderService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void getOrder_shouldReturn200_whenOrderExists() throws Exception {
        OrderResponse response = new OrderResponse(1L, BigDecimal.TEN, "PAID", Instant.now());
        when(orderService.getOrder(1L)).thenReturn(response);

        mockMvc.perform(get("/api/v1/orders/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.status").value("PAID"));
    }

    @Test
    void createOrder_shouldReturn400_whenAmountMissing() throws Exception {
        String invalidJson = """
                {"userId": 1, "currency": "INR"}
                """;

        mockMvc.perform(post("/api/v1/orders")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(invalidJson))
                .andExpect(status().isBadRequest());
    }
}
```

`@WebMvcTest` loads **only** the web layer (controllers, `@ControllerAdvice`, filters, Jackson config) — not the full application context, so no real database, no real service beans. This makes it fast and lets you verify HTTP-level contract (status codes, JSON shape, validation) in isolation.

### `@DataJpaTest` — repository layer only

```java
@DataJpaTest   // configures an in-memory embedded DB (H2 by default) and JPA infrastructure only
class OrderRepositoryTest {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void findByUserIdAndStatus_shouldReturnMatchingOrders() {
        User user = new User("test@example.com");
        entityManager.persist(user);

        Order order = new Order(BigDecimal.TEN, OrderStatus.PAID, user);
        entityManager.persist(order);
        entityManager.flush();

        List<Order> results = orderRepository.findByUserIdAndStatus(user.getId(), OrderStatus.PAID);

        assertThat(results).hasSize(1);
        assertThat(results.get(0).getAmount()).isEqualByComparingTo(BigDecimal.TEN);
    }
}
```

### `@SpringBootTest` — full integration test

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@Testcontainers
class OrderIntegrationTest {

    @Container
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", mysql::getJdbcUrl);
        registry.add("spring.datasource.username", mysql::getUsername);
        registry.add("spring.datasource.password", mysql::getPassword);
    }

    @Autowired
    private MockMvc mockMvc;

    @Test
    void fullOrderFlow_shouldPersistAndReturnCreatedOrder() throws Exception {
        String requestBody = """
                {"userId": 1, "amount": 100.00, "currency": "INR"}
                """;

        mockMvc.perform(post("/api/v1/orders")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(requestBody))
                .andExpect(status().isCreated())
                .andExpect(header().exists("Location"));
    }
}
```

**Testcontainers** (used here for a real MySQL in a Docker container, spun up just for the test run) is the modern, production-quality alternative to testing against H2-in-memory for integration tests — H2 doesn't perfectly replicate MySQL/Postgres-specific SQL dialect quirks, which can hide real bugs. This is a strong thing to mention in interviews as current best practice.

### Mocking external HTTP calls with WireMock / MockRestServiceServer

```java
@RestClientTest(PaymentGatewayClient.class)
class PaymentGatewayClientTest {

    @Autowired
    private PaymentGatewayClient client;

    @Autowired
    private MockRestServiceServer server;

    @Test
    void charge_shouldReturnSuccess_whenGatewayReturns200() {
        server.expect(requestTo("https://api.gateway.com/charge"))
                .andRespond(withSuccess("{\"status\":\"SUCCESS\"}", MediaType.APPLICATION_JSON));

        boolean result = client.charge(new ChargeRequest(BigDecimal.TEN));

        assertThat(result).isTrue();
    }
}
```

---

## 18. API Documentation — OpenAPI/Swagger

### Adding springdoc-openapi

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

That single dependency, with **zero configuration**, exposes:

- `/v3/api-docs` — the raw OpenAPI 3.0 JSON spec
- `/swagger-ui.html` — an interactive UI to browse and test every endpoint

It works by reflecting over your `@RestController` classes and `@RequestMapping` annotations at startup — no manual spec-writing required, though you can (and should, for a real API) enrich it.

### Enriching the generated docs

```java
@RestController
@RequestMapping("/api/v1/orders")
@Tag(name = "Orders", description = "Order management endpoints")
@RequiredArgsConstructor
public class OrderController {

    @Operation(summary = "Get an order by ID", description = "Returns full order details including line items")
    @ApiResponses({
            @ApiResponse(responseCode = "200", description = "Order found",
                    content = @Content(schema = @Schema(implementation = OrderResponse.class))),
            @ApiResponse(responseCode = "404", description = "Order not found", content = @Content)
    })
    @GetMapping("/{orderId}")
    public ResponseEntity<OrderResponse> getOrder(
            @Parameter(description = "The order's unique ID", example = "1042")
            @PathVariable Long orderId) {
        ...
    }
}
```

```java
public record CreateOrderRequest(
        @Schema(description = "ID of the user placing the order", example = "1")
        Long userId,

        @Schema(description = "Order amount in the specified currency", example = "499.00")
        BigDecimal amount
) {}
```

### Global API metadata + JWT auth in Swagger UI

```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("Order Service API")
                        .version("1.0")
                        .description("Order management microservice"))
                .addSecurityItem(new SecurityRequirement().addList("bearerAuth"))
                .components(new Components().addSecuritySchemes("bearerAuth",
                        new SecurityScheme()
                                .type(SecurityScheme.Type.HTTP)
                                .scheme("bearer")
                                .bearerFormat("JWT")));
    }
}
```

This adds an "Authorize" button in Swagger UI so a JWT can be pasted once and automatically attached to every subsequent test request in the UI — genuinely useful day-to-day, not just for documentation.

---

## 19. File Upload & Download

### File upload

```java
@RestController
@RequestMapping("/api/v1/files")
@RequiredArgsConstructor
public class FileController {

    private final FileStorageService fileStorageService;

    @PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<FileUploadResponse> uploadFile(
            @RequestParam("file") MultipartFile file,
            @RequestParam(required = false) String description) {

        if (file.isEmpty()) {
            throw new IllegalArgumentException("Uploaded file is empty");
        }

        String storedFileName = fileStorageService.store(file);
        return ResponseEntity.ok(new FileUploadResponse(storedFileName, file.getSize(), file.getContentType()));
    }

    @PostMapping(value = "/upload-multiple", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ResponseEntity<List<String>> uploadMultiple(@RequestParam("files") List<MultipartFile> files) {
        List<String> storedNames = files.stream()
                .map(fileStorageService::store)
                .toList();
        return ResponseEntity.ok(storedNames);
    }
}
```

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB
```

### Validating uploads (important — never trust client-supplied content type alone)

```java
@Service
public class FileStorageService {

    private static final Set<String> ALLOWED_TYPES = Set.of("image/png", "image/jpeg", "application/pdf");
    private static final long MAX_SIZE = 5 * 1024 * 1024; // 5MB

    public String store(MultipartFile file) {
        if (!ALLOWED_TYPES.contains(file.getContentType())) {
            throw new InvalidFileException("Unsupported file type: " + file.getContentType());
        }
        if (file.getSize() > MAX_SIZE) {
            throw new InvalidFileException("File exceeds maximum size of 5MB");
        }

        String originalFilename = StringUtils.cleanPath(file.getOriginalFilename());
        if (originalFilename.contains("..")) {
            throw new InvalidFileException("Invalid file path"); // path traversal guard
        }

        String extension = originalFilename.substring(originalFilename.lastIndexOf('.'));
        String storedFileName = UUID.randomUUID() + extension; // never trust/reuse the client's filename directly

        try {
            Path targetPath = Paths.get(uploadDir).resolve(storedFileName);
            Files.copy(file.getInputStream(), targetPath, StandardCopyOption.REPLACE_EXISTING);
        } catch (IOException e) {
            throw new FileStorageException("Failed to store file", e);
        }

        return storedFileName;
    }
}
```

Three real security concerns baked into that method: **content-type allowlisting** (client-declared `Content-Type` can be spoofed, but it's still a first line of defense — production systems layer on magic-byte sniffing for anything security-sensitive), **path traversal prevention** (`..` in a filename), and **filename randomization** (never write a file to disk under a name the client controls — collisions, overwrites, and injection risk).

### File download

```java
@GetMapping("/download/{fileName}")
public ResponseEntity<Resource> downloadFile(@PathVariable String fileName) throws IOException {
    Path filePath = Paths.get(uploadDir).resolve(fileName).normalize();

    if (!filePath.startsWith(Paths.get(uploadDir))) {
        throw new InvalidFileException("Invalid file path"); // path traversal guard, again, on the read side
    }

    Resource resource = new UrlResource(filePath.toUri());
    if (!resource.exists()) {
        throw new ResourceNotFoundException("File not found: " + fileName);
    }

    String contentType = Files.probeContentType(filePath);

    return ResponseEntity.ok()
            .contentType(MediaType.parseMediaType(contentType != null ? contentType : "application/octet-stream"))
            .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + resource.getFilename() + "\"")
            .body(resource);
}
```

For real production systems (especially in a banking context with compliance requirements around document retention), file storage almost always moves off the local filesystem entirely and onto object storage (AWS S3 / Azure Blob) — the `FileStorageService` abstraction above is exactly what lets you swap a local-disk implementation for an S3-backed one without touching the controller.

---

## 20. Microservices Essentials

*(You've already built a hands-on Kafka-based order system covering event-driven communication — this section focuses on the synchronous/infrastructure side that complements it.)*

### Service-to-service HTTP calls: RestTemplate vs. WebClient vs. Feign

```java
// RestTemplate — the classic, blocking, synchronous client (legacy but still widely seen)
@Configuration
public class RestTemplateConfig {
    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder
                .setConnectTimeout(Duration.ofSeconds(3))
                .setReadTimeout(Duration.ofSeconds(5))
                .build();
    }
}
```

```java
// WebClient — the modern, reactive (but usable synchronously too) replacement for RestTemplate
@Service
public class InventoryClient {

    private final WebClient webClient;

    public InventoryClient(WebClient.Builder builder) {
        this.webClient = builder.baseUrl("http://inventory-service").build();
    }

    public InventoryResponse checkStock(Long productId) {
        return webClient.get()
                .uri("/api/v1/inventory/{productId}", productId)
                .retrieve()
                .onStatus(HttpStatusCode::is4xxClientError, response ->
                        Mono.error(new ResourceNotFoundException("Product not found")))
                .bodyToMono(InventoryResponse.class)
                .timeout(Duration.ofSeconds(3))
                .block(); // .block() makes it synchronous — or return the Mono directly for a reactive chain
    }
}
```

```java
// Feign — declarative, interface-based HTTP client (very common at product companies)
@FeignClient(name = "inventory-service", url = "${inventory.service.url}")
public interface InventoryFeignClient {

    @GetMapping("/api/v1/inventory/{productId}")
    InventoryResponse checkStock(@PathVariable Long productId);
}
```

```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final InventoryFeignClient inventoryClient; // just inject and call it like a local method
}
```

**Interview-ready comparison:** `RestTemplate` is in **maintenance mode** (not deprecated, but Spring's docs point new code toward `WebClient`); `WebClient` is non-blocking/reactive-capable and the current recommendation; `Feign` trades some flexibility for the least boilerplate — you write an interface, Spring generates the implementation, and it integrates cleanly with load-balancing (via Spring Cloud LoadBalancer) when service discovery is in play.

### Resilience: timeouts, retries, circuit breakers (Resilience4j)

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
```

```java
@Service
@RequiredArgsConstructor
public class InventoryClient {

    private final InventoryFeignClient feignClient;

    @CircuitBreaker(name = "inventoryService", fallbackMethod = "fallbackStock")
    @Retry(name = "inventoryService")
    @TimeLimiter(name = "inventoryService")
    public CompletableFuture<InventoryResponse> checkStock(Long productId) {
        return CompletableFuture.supplyAsync(() -> feignClient.checkStock(productId));
    }

    public CompletableFuture<InventoryResponse> fallbackStock(Long productId, Throwable t) {
        // called when the circuit is OPEN or all retries are exhausted
        return CompletableFuture.completedFuture(InventoryResponse.unknown());
    }
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      inventoryService:
        sliding-window-size: 10
        failure-rate-threshold: 50    # open the circuit if 50% of the last 10 calls failed
        wait-duration-in-open-state: 10s
  retry:
    instances:
      inventoryService:
        max-attempts: 3
        wait-duration: 500ms
```

**Why a circuit breaker matters (a genuinely important fintech/product-company concept):** without one, if `inventory-service` starts timing out, every request to `order-service` piles up waiting on those timeouts, exhausting `order-service`'s own thread pool — a slow downstream service takes down an otherwise-healthy upstream one ("cascading failure"). A circuit breaker detects the failure rate, "opens" (stops even attempting the call, fails fast with the fallback) for a cool-down period, then "half-opens" to test if the downstream has recovered.

### Service discovery & config (brief — usually infra-team owned, but worth recognizing)

- **Eureka / Consul** — service registry; instances register themselves, other services discover them by logical name instead of hardcoded host:port.
- **Spring Cloud Config Server** — centralizes `application.yml` for all microservices in one Git-backed repo, with per-service, per-profile overrides.
- **API Gateway (Spring Cloud Gateway)** — the single entry point that routes to the right downstream service, and is the natural place to centralize cross-cutting concerns like auth, rate limiting, and request logging.

### Idempotency in distributed systems

Given your ledger project already deals with this directly — the core interview-ready explanation: any endpoint that can be safely retried by a client (or a message broker redelivering a Kafka message) needs an **idempotency key** so a retry doesn't double-charge or double-create a resource.

```java
@PostMapping("/payments")
public ResponseEntity<PaymentResponse> createPayment(
        @RequestHeader("Idempotency-Key") String idempotencyKey,
        @RequestBody PaymentRequest request) {

    return paymentRepository.findByIdempotencyKey(idempotencyKey)
            .map(existing -> ResponseEntity.ok(toResponse(existing))) // already processed — return the same result
            .orElseGet(() -> {
                Payment payment = paymentService.process(request, idempotencyKey);
                return ResponseEntity.status(HttpStatus.CREATED).body(toResponse(payment));
            });
}
```

A unique constraint on `idempotency_key` at the database level is the actual guarantee here — the check-then-insert above still has a race window under true concurrency, so the DB constraint (and catching the resulting `DataIntegrityViolationException` to return the existing record) is what makes this airtight.

---

## 21. Deployment & Production Best Practices

### Packaging & running

```bash
# Build an executable JAR (Tomcat embedded inside it)
mvn clean package

# Run it
java -jar target/myapp-1.0.0.jar --spring.profiles.active=prod
```

### Dockerizing a Spring Boot app

```dockerfile
# Multi-stage build — keeps the final image small, no build tools shipped to production
FROM eclipse-temurin:17-jdk-alpine AS build
WORKDIR /app
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline
COPY src ./src
RUN ./mvnw clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
RUN addgroup -S spring && adduser -S spring -G spring   # never run as root in a container
USER spring
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```bash
docker build -t myapp:1.0.0 .
docker run -p 8080:8080 -e SPRING_PROFILES_ACTIVE=prod -e DB_PASSWORD=secret myapp:1.0.0
```

**Multi-stage builds** matter for two reasons: the final image doesn't carry the entire Maven/JDK build toolchain (smaller image, smaller attack surface), and Docker layer caching means `dependency:go-offline` is only re-run when `pom.xml` changes, not on every source code change — much faster iterative builds.

### Externalized configuration in containers (12-Factor App principle)

Never bake environment-specific values into the image. Pass them at runtime:

```yaml
# docker-compose.yml (local/dev)
services:
  app:
    image: myapp:1.0.0
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      SPRING_DATASOURCE_URL: jdbc:mysql://db:3306/mydb
      DB_PASSWORD: ${DB_PASSWORD}
    depends_on:
      - db
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_DATABASE: mydb
```

In real production (Kubernetes), the equivalent is a `ConfigMap` for non-sensitive config and a `Secret` (or an external secrets manager synced in) for credentials — never committed to git, never baked into the image.

### Graceful shutdown

```yaml
server:
  shutdown: graceful

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

Without this, a `SIGTERM` (which is what Kubernetes/Docker send on a rolling deploy or scale-down) kills the JVM immediately — any in-flight request gets an abrupt connection reset. With graceful shutdown enabled, Spring Boot stops accepting new requests but lets in-flight ones finish (up to the configured timeout) before the JVM actually exits.

### JVM tuning basics (worth being able to speak to)

```bash
java -Xms512m -Xmx512m \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -jar app.jar
```

- `-Xms`/`-Xmx` — set equal in containers to avoid the JVM resizing the heap at runtime (predictable memory footprint, matters for container memory limits/OOM-killer behavior).
- `G1GC` — the default collector on modern JDKs, a solid choice for most server workloads; only reach for `ZGC`/`Shenandoah` if you have measured, specific low-pause-time requirements.

### Health checks wired into orchestration

```yaml
management:
  endpoint:
    health:
      probes:
        enabled: true   # exposes /actuator/health/liveness and /actuator/health/readiness separately
```

```yaml
# Kubernetes deployment snippet
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 30
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
```

**Liveness vs. readiness, a real distinction:** liveness answers "is this instance stuck/deadlocked and needs to be killed and restarted?"; readiness answers "is this instance currently able to serve traffic?" (e.g., it might be alive but still warming up its cache, or its DB connection pool is exhausted) — Kubernetes stops routing traffic to a not-ready pod without killing it, but restarts a not-alive one.

### Production checklist (a genuinely useful thing to keep handy)

- [ ] `ddl-auto: validate`, schema managed by Flyway/Liquibase — never `update`/`create` in prod
- [ ] Secrets pulled from environment/secrets manager, never committed
- [ ] Actuator endpoints restricted (`exposure.include` limited, auth-protected)
- [ ] Connection pool sized appropriately (`spring.datasource.hikari.maximum-pool-size`) — not left at the default for expected load
- [ ] Structured logging (JSON) with correlation/request IDs for tracing across services
- [ ] `@Transactional(readOnly = true)` on read-only service methods
- [ ] Circuit breakers/timeouts on all outbound service calls
- [ ] Graceful shutdown enabled
- [ ] Rate limiting at the gateway or via a filter for public-facing endpoints
- [ ] Global exception handler never leaks stack traces or internal details to clients
- [ ] All entity associations explicitly `LAZY`, N+1s checked via query logging in staging

---

## Quick-Reference: Most-Asked Interview Questions Mapped to Sections

| Question | Section |
|---|---|
| Spring vs Spring Boot? | [§1](#1-introduction--core-concepts) |
| `@Component` vs `@Service` vs `@Repository`? | [§3](#3-core-spring-annotations) |
| Why constructor injection over field injection? | [§4](#4-dependency-injection--bean-lifecycle) |
| How does auto-configuration actually work? | [§5](#5-auto-configuration-internals) |
| What is the N+1 problem and how do you fix it? | [§10](#10-spring-data-jpa) |
| Optimistic vs pessimistic locking? | [§10](#10-spring-data-jpa) |
| Why does `@Transactional` sometimes silently not work? | [§11](#11-transactions) |
| How does Spring Security's filter chain work? | [§12](#12-spring-security--jwt) |
| How would you prevent a double-spend/race condition? | [§10](#10-spring-data-jpa), [§20](#20-microservices-essentials) |
| What is a circuit breaker and why do you need one? | [§20](#20-microservices-essentials) |
| How do you ensure idempotency on a payment API? | [§20](#20-microservices-essentials) |
| `fixedRate` vs `fixedDelay`? | [§16](#16-scheduling--async) |
| `@Cacheable` vs `@CachePut` vs `@CacheEvict`? | [§15](#15-caching) |
| How do you structure tests (unit vs slice vs integration)? | [§17](#17-testing) |
| What goes into a production checklist before go-live? | [§21](#21-deployment--production-best-practices) |

---

*End of notes. Suggested next step: pick one section a day, rebuild the code example from memory without looking, then try to extend it (e.g., after Section 12, add refresh-token support to the JWT flow) — active recall plus extension is what actually sticks before interviews.*
