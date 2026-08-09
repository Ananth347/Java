# Spring Framework, Spring Boot & REST API — Q&A Notes

A question-and-answer study guide covering **Spring Framework Core**, **Spring Boot Fundamentals**, and **REST API Development**.

---

## 1. Spring Framework Core (Foundation)

### Q1. What is Spring, and why does it exist?
**A.** Before Spring, Java enterprise developers had to write huge amounts of boilerplate code — manually creating objects, wiring them together, and managing transactions by hand. None of this had anything to do with actual business logic.

Spring exists to eliminate that plumbing. It acts like a "manager" that creates your objects, connects them together, and manages their entire lifecycle, freeing developers to focus purely on business logic.

Spring is not one single library — it's an **umbrella project** made of many modules:
- **Spring Core** — the foundation (IoC container, Dependency Injection)
- **Spring MVC** — building web apps and REST APIs
- **Spring Data** — working with databases
- **Spring Security** — authentication and authorization
- **Spring Boot** — a layer that removes configuration pain

The single idea underlying everything: **Spring manages objects so you don't have to.**

---

### Q2. What is Inversion of Control (IoC)?
**A.** IoC is the most fundamental idea in Spring. Breaking down the term:
- **Control** = who creates objects and decides how they interact
- **Inversion** = flipping who is responsible for that

**Without Spring:** If class `A` needs class `B`, `A` creates `B` itself using `new B()`. Class `A` is in control.

**With Spring (IoC):** Class `A` doesn't create `B`. It simply declares "I need a `B`," and an external system — the **Spring Container** — hands it a ready-made instance. The responsibility for "who creates what" has been inverted from the class to the container.

**Why this matters:** `A` no longer cares *how* `B` is built, what `B`'s own dependencies are, or when `B` is destroyed. This results in:
- Easier testing (a fake/mock `B` can be substituted)
- Easier change (swap implementations without touching `A`)
- Looser coupling between classes

IoC is the general principle; **Dependency Injection** is the specific technique Spring uses to achieve it.

---

### Q3. What is Dependency Injection (DI), and what are the three ways to do it?
**A.** DI is how Spring implements IoC in practice — instead of a class creating its own dependencies, they are supplied ("injected") from outside.

**1. Constructor Injection (recommended in almost all modern code):**
```java
@Service
public class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**2. Setter Injection:**
```java
@Service
public class OrderService {
    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**3. Field Injection (common in tutorials, discouraged in production):**
```java
@Service
public class OrderService {
    @Autowired
    private PaymentService paymentService;
}
```

**Why constructor injection is preferred:**
- Makes dependencies explicit and *mandatory* — the object cannot be constructed without them
- Works cleanly with `final` fields, so the dependency can never be reassigned later
- Makes unit testing trivial — just pass a mock into the constructor, no reflection tricks required

**Mental model:** DI is like a restaurant kitchen. The chef (your class) doesn't grow vegetables or raise chickens (create its own dependencies) — a supplier (the Spring container) delivers exactly the ingredients ordered, ready to use.

---

### Q4. What is a "bean" in Spring?
**A.** A bean is simply an object that the **Spring container** creates, configures, and manages — instead of your code manually managing it with `new`.

Any class can become a bean; what makes it a bean is not something special about the class, but the fact that Spring is responsible for that particular instance.

**Declaring a bean via component scanning:**
```java
@Component
public class NotificationSender {
    // Detected during component scanning and registered automatically
}
```

**Declaring a bean manually:**
```java
@Configuration
public class AppConfig {
    @Bean
    public NotificationSender notificationSender() {
        return new NotificationSender();
    }
}
```

Once registered, Spring tracks the bean in an internal registry and supplies it to any other bean that needs it via DI.

---

### Q5. What is the Bean Lifecycle?
**A.** Every bean goes through a well-defined lifecycle managed entirely by the container. Understanding it is essential for debugging startup issues.

1. **Instantiation** — Spring calls the constructor
2. **Populate properties** — dependencies are injected (DI happens here)
3. **Aware callbacks** — if the bean implements interfaces like `BeanNameAware` or `BeanFactoryAware`, Spring calls their callback methods
4. **`@PostConstruct`** — custom initialization logic runs right after dependency injection
5. **`InitializingBean.afterPropertiesSet()`** — older interface-based alternative to `@PostConstruct`
6. **Custom init-method** — defined via `@Bean(initMethod = "...")`
7. **Bean is ready and in use** — normal operating phase
8. **`@PreDestroy`** — called just before destruction (e.g., app shutdown)
9. **`DisposableBean.destroy()`** — interface-based alternative to `@PreDestroy`

```java
@Component
public class DatabaseConnector {

    @PostConstruct
    public void init() {
        System.out.println("Opening database connection pool...");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Closing database connection pool...");
    }
}
```

This is useful for opening/closing connections, warming caches, or validating configuration at startup.

---

### Q6. What is the Spring Container, and what are its two main types?
**A.** The Spring Container is the engine at the heart of the framework. It:
- Creates beans
- Wires them together (DI)
- Manages their full lifecycle
- Destroys them cleanly on shutdown

Think of it as a **factory floor**: you describe the parts (beans) and how they connect, and the container assembles the whole machine (your app) at startup.

Two flavors:
| Container | Description |
|---|---|
| `BeanFactory` | The most basic container; lazy-loads beans (created only when requested). Rarely used directly today. |
| `ApplicationContext` | A richer container built on top of `BeanFactory`. Used by virtually all real Spring applications. |

---

### Q7. What is `ApplicationContext`, and what extra features does it add?
**A.** `ApplicationContext` is the container implementation almost every Spring Boot app uses under the hood. It extends `BeanFactory` and adds enterprise-friendly features:
- **Eager bean initialization** by default — configuration errors surface immediately at startup, not later at runtime
- **Event publishing** via `ApplicationEventPublisher` — beans can publish/listen to events
- **Internationalization (i18n)** support
- **Easy resource access** (files, classpath resources, URLs)
- **Automatic AOP integration**

In Spring Boot, `SpringApplication.run()` constructs the context for you:
```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(MyApp.class, args);
        NotificationSender sender = context.getBean(NotificationSender.class);
    }
}
```

---

### Q8. What are Bean Scopes, and when should you use `prototype` instead of the default?
**A.** By default, every Spring bean is a **singleton** — one shared instance per container, reused everywhere.

| Scope | Meaning |
|---|---|
| `singleton` (default) | One shared instance for the whole application |
| `prototype` | A brand-new instance every time the bean is requested |
| `request` | One instance per HTTP request (web apps only) |
| `session` | One instance per HTTP session (web apps only) |
| `application` | One instance per `ServletContext` |

```java
@Component
@Scope("prototype")
public class ReportGenerator {
    // A new instance is created every time this bean is injected or fetched
}
```

**When to use `prototype`:** whenever the bean holds *mutable state specific to one usage* — e.g., if `ReportGenerator` accumulates data during generation and you don't want concurrent requests overwriting each other's data.

---

### Q9. What do the core Spring annotations do?
**A.** Modern Spring apps configure almost everything with annotations instead of XML.

| Annotation | Purpose |
|---|---|
| `@Component` | Generic stereotype marking a class as a Spring-managed bean |
| `@Service` | Semantically marks business-logic classes (functionally identical to `@Component`) |
| `@Repository` | Marks data-access classes; also auto-translates DB exceptions into Spring's unified exception hierarchy |
| `@Controller` / `@RestController` | Marks a web-layer class handling HTTP requests |
| `@Autowired` | Tells Spring to inject a dependency automatically |
| `@Configuration` | Marks a class as a source of bean definitions |
| `@Bean` | Marks a method inside `@Configuration` as producing a bean |
| `@Qualifier` | Used with `@Autowired` to pick a specific bean when multiple candidates of the same type exist |
| `@Value` | Injects a value from a properties file into a field |

```java
@Service
public class PricingService {

    @Value("${discount.percentage}")
    private double discountPercentage;

    private final TaxService taxService;

    public PricingService(@Qualifier("standardTaxService") TaxService taxService) {
        this.taxService = taxService;
    }
}
```

---

### Q10. What is Java Configuration, and how does it differ from XML config?
**A.** Before annotations, Spring apps were configured entirely through verbose XML files. Java Configuration replaced this with plain Java classes, giving you **type safety**, **refactoring support**, and the ability to write real logic in configuration.

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripePaymentGateway("api-key-here");
    }

    @Bean
    public OrderService orderService(PaymentGateway paymentGateway) {
        return new OrderService(paymentGateway); // Spring auto-passes the bean above
    }
}
```

Every method annotated `@Bean` inside a `@Configuration` class becomes a bean definition, registered under its return type (and, by default, its method name as the bean name).

---

### Q11. What is Component Scanning?
**A.** Rather than manually registering every `@Component`, `@Service`, `@Repository`, etc., Spring can automatically **scan your codebase** and find them.

```java
@SpringBootApplication  // already includes @ComponentScan internally
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

By default, Spring Boot scans the package containing the main application class **and every sub-package beneath it** — which is why convention places the main class at the root package (e.g., `com.mycompany.myapp`).

To scan additional packages outside that structure:
```java
@SpringBootApplication
@ComponentScan(basePackages = {"com.mycompany.myapp", "com.othercompany.shared"})
public class MyApp { }
```

---

### Q12. What are Profiles, and why are they useful?
**A.** Real applications behave differently per environment — you don't want dev database credentials active in production. **Profiles** let you define environment-specific beans and configuration.

```java
@Service
@Profile("dev")
public class MockPaymentService implements PaymentService {
    // Fake payment service used only locally
}

@Service
@Profile("prod")
public class RealPaymentService implements PaymentService {
    // Actual payment gateway integration
}
```

Activate a profile via a property:
```
spring.profiles.active=dev
```
Or as a command-line argument:
```
java -jar myapp.jar --spring.profiles.active=prod
```

Only the bean matching the active profile is created — the other is completely ignored by the container.

---

### Q13. What is the `Environment` abstraction?
**A.** `Environment` is Spring's unified way of representing configuration from many sources — property files, environment variables, command-line arguments, JVM system properties — merged and exposed through one consistent interface.

```java
@Component
public class ConfigPrinter {

    private final Environment environment;

    public ConfigPrinter(Environment environment) {
        this.environment = environment;
    }

    @PostConstruct
    public void printConfig() {
        String dbUrl = environment.getProperty("spring.datasource.url");
        System.out.println("Connecting to: " + dbUrl);
    }
}
```

Spring resolves properties using a clear **precedence order** (highest to lowest): command-line arguments → JVM system properties → OS environment variables → `application.properties`/`application.yml` files. This lets you override config at deployment time without touching the packaged app.

---

### Q14. What is External Configuration, and what is the priority order Spring Boot checks?
**A.** External configuration means configuration values live **outside compiled code**, so behavior can change without recompiling or redeploying.

Spring Boot checks configuration sources in this priority order:
1. Command-line arguments
2. `SPRING_APPLICATION_JSON` environment variable
3. Servlet parameters
4. OS environment variables
5. `application.properties` / `application.yml` (and profile-specific variants like `application-prod.yml`)
6. Default values hardcoded in `@Value` annotations

This layered approach means the *same packaged JAR* can behave differently across dev, staging, and production, just by changing external config — never the code. This is a core principle of **twelve-factor app** design, which Spring Boot fully embraces.

---

## 2. Spring Boot Fundamentals

### Q15. What is Spring Boot, and what problem does it solve?
**A.** Plain Spring is powerful but historically painful to set up — manually configuring a web server, wiring dozens of beans, writing mountains of XML, and juggling compatible dependency versions.

Spring Boot is built **on top of** Spring (not a replacement) with the philosophy of **"convention over configuration."** It makes smart default decisions so you can go from zero to a running app in minutes.

Concretely, Spring Boot provides:
- **Auto-configuration** — guesses what you need and configures it automatically
- **Embedded web server** — no separate Tomcat installation required
- **Starter dependencies** — one dependency pulls in a whole compatible bundle
- **Production-ready features** out of the box (health checks, metrics)
- **A single executable JAR** runnable via `java -jar`

---

### Q16. What does the Spring Boot architecture / startup flow look like?
**A.** Spring Boot is built from a small number of core ideas stacked on top of each other:

```
Your Application Code
        |
Spring Boot Starters  (dependency bundles)
        |
Spring Boot Auto-Configuration  (smart defaults)
        |
Spring Framework Core  (IoC, DI, AOP, etc.)
        |
Embedded Server (Tomcat / Jetty / Undertow)
```

**On startup:**
1. `main()` calls `SpringApplication.run(...)`
2. Spring Boot creates the `ApplicationContext`
3. Auto-configuration classes inspect the classpath and configure beans accordingly (e.g., detecting a JPA driver → auto-configuring a `DataSource` and `EntityManagerFactory`)
4. Component scanning discovers your `@Component`/`@Service`/`@Controller` classes
5. The embedded server starts and begins listening for HTTP requests

---

### Q17. What is Spring Initializr?
**A.** Spring Initializr (at `start.spring.io`, and built into IntelliJ/VS Code/Eclipse) is a **project generator**. Instead of manually setting up folder structures, `pom.xml`/`build.gradle` files, and boilerplate, you:
1. Pick a build tool (Maven or Gradle)
2. Pick a language (Java, Kotlin, Groovy)
3. Pick a Spring Boot version
4. Pick dependencies (Web, JPA, Security, etc.) from a searchable list
5. Click "Generate" — a ready-to-run zipped project downloads

This saves setup time and guarantees compatible dependency versions from the start.

---

### Q18. What are Starter Dependencies?
**A.** Instead of adding many separate libraries individually (and hoping their versions are compatible), you add **one starter**, which pulls in everything related at tested, compatible versions.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

For example, `spring-boot-starter-web` silently brings in Spring MVC, Jackson (JSON), validation support, and an embedded Tomcat server — all in one line.

---

### Q19. How does Auto-Configuration actually work under the hood?
**A.** Auto-configuration is the "magic" that makes Spring Boot feel effortless. It scans what's on your classpath and automatically configures sensible beans — **without you writing any configuration.**

Example: if Spring Boot detects the H2 database driver on the classpath and no `DataSource` bean has been manually defined, it auto-configures an in-memory H2 `DataSource`.

Under the hood, this uses `@Configuration` classes combined with **conditional annotations**:

```java
@Configuration
@ConditionalOnClass(DataSource.class)
@ConditionalOnMissingBean(DataSource.class)
public class DataSourceAutoConfiguration {

    @Bean
    public DataSource dataSource() {
        // builds a sensible default DataSource
    }
}
```

- `@ConditionalOnClass` — "only apply this configuration if a specific class is present on the classpath"
- `@ConditionalOnMissingBean` — "only apply this if the developer hasn't already defined their own bean of this type" (so explicit config always wins over auto-configuration)

Auto-configuration is enabled via `@EnableAutoConfiguration`, though in practice this is bundled inside `@SpringBootApplication`, so you rarely write it directly.

---

### Q20. What are Embedded Servers, and why did Spring Boot move away from WAR deployment?
**A.** Traditionally, you'd build a WAR file and deploy it to a separately-installed application server (like Tomcat). Spring Boot flips this: the **server is embedded directly inside your JAR**.

This means:
- You run the app with a single command: `java -jar myapp.jar`
- No separate server installation or version-matching headaches
- Your application *is* the server — it starts the server itself, inside its own `main()`

By default, Spring Boot uses **Tomcat** with `spring-boot-starter-web`, but you can swap it for Jetty or Undertow:

```xml
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
```

---

### Q21. What is `application.properties`?
**A.** The classic, simplest way to externalize configuration — a flat key-value file at `src/main/resources/application.properties`, read automatically at startup.

```properties
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
spring.jpa.hibernate.ddl-auto=update
logging.level.org.springframework=INFO
```

Each property maps to internal configuration Spring Boot uses to configure beans — server port, database connection, JPA behavior, logging levels, etc.

---

### Q22. What is `application.yml`, and how does it differ from `.properties`?
**A.** YAML is an alternative format that lets you express **nested configuration hierarchically**, instead of repeating dotted prefixes.

The same config as `.properties`, in YAML:

```yaml
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: secret
  jpa:
    hibernate:
      ddl-auto: update

logging:
  level:
    org.springframework: INFO
```

Both formats achieve exactly the same result — it's a matter of preference and readability. You *can* use both, but best practice is to pick one per project to avoid confusion.

---

### Q23. What does `@ConfigurationProperties` do, and how is it better than `@Value` for related settings?
**A.** Reading values one at a time with `@Value("${some.key}")` gets tedious for many related settings. `@ConfigurationProperties` binds an entire configuration block to a strongly-typed Java class in one shot.

```yaml
app:
  mail:
    host: smtp.mailtrap.io
    port: 587
    from-address: no-reply@myapp.com
```

```java
@Component
@ConfigurationProperties(prefix = "app.mail")
public class MailProperties {
    private String host;
    private int port;
    private String fromAddress;
    // getters and setters required for binding
}
```

Now `MailProperties` can be injected anywhere, giving fully-typed, validated access to the whole configuration block — much cleaner than scattering `@Value` everywhere.

---

### Q24. What is `CommandLineRunner` used for?
**A.** Sometimes you need code to run exactly once, immediately after the application has fully started — e.g., seeding a database with default data, or printing startup diagnostics. `CommandLineRunner` is a functional interface built for exactly this.

```java
@Component
public class DataSeeder implements CommandLineRunner {

    private final UserRepository userRepository;

    public DataSeeder(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public void run(String... args) {
        if (userRepository.count() == 0) {
            userRepository.save(new User("admin", "admin@example.com"));
            System.out.println("Default admin user created.");
        }
    }
}
```

Spring Boot automatically detects any bean implementing `CommandLineRunner` and calls `run()` right after the context has fully loaded. With multiple runners, ordering can be controlled with `@Order`.

---

### Q25. Can you customize or disable the startup banner?
**A.** Yes. The ASCII-art Spring logo printed at startup ("the banner") can be customized or turned off.

**To customize:** create `banner.txt` in `src/main/resources`:
```
  __  __         _
 |  \/  |_   _  / \   _ __  _ __
 | |\/| | | | |/ _ \ | '_ \| '_ \
 | |  | | |_| / ___ \| |_) | |_) |
 |_|  |_|\__, /_/   \_\ .__/| .__/
         |___/        |_|   |_|
::  My Awesome App  ::
```

**To disable it entirely** (useful in production logs to reduce visual noise):
```properties
spring.main.banner-mode=off
```

A small feature, but a nice, harmless touch teams use to make console output recognizable.

---

### Q26. What is Spring Boot DevTools, and what does it provide?
**A.** DevTools is a small dependency aimed purely at making local development faster and less annoying:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

**What it gives you:**
- **Automatic restart** — recompiling a changed class triggers an automatic restart, much faster than a cold restart, because it uses two classloaders (one for frequently-changing app code, one for stable third-party libraries)
- **LiveReload** — automatically refreshes the browser when static resources (HTML/CSS/JS) change
- **Sensible dev-time defaults** — e.g., disables template caching so changes appear instantly

DevTools is automatically excluded from production builds when packaged as an executable JAR, so there's no risk of it shipping to production.

---

## 3. REST API Development

### Q27. What are the core principles of REST?
**A.** REST (Representational State Transfer) is an **architectural style**, not a protocol or library. Key principles:

- **Statelessness** — the server stores no client session state between requests; each request carries everything needed to process it
- **Resource-based URLs** — URLs represent "nouns" (things), not "verbs" (actions): `/orders/5` is order #5, not an action
- **Standard HTTP methods** — `GET`, `POST`, `PUT`, `PATCH`, `DELETE` represent actions on resources instead of inventing custom verbs in the URL
- **Uniform interface** — consistent naming, response shapes, and error formats across the whole API
- **Client-server separation** — frontend and backend evolve independently as long as the API contract stays consistent
- **Cacheability** — responses should indicate whether they can be cached, for performance

A well-designed endpoint for "get order 5" is `GET /orders/5` — **not** `GET /getOrder?id=5`, since the HTTP method already communicates the action.

---

### Q28. What do the standard HTTP methods mean, and which are idempotent?
**A.**

| Method | Meaning | Idempotent? |
|---|---|---|
| `GET` | Retrieve a resource, no side effects | Yes |
| `POST` | Create a new resource | No |
| `PUT` | Replace a resource entirely | Yes |
| `PATCH` | Partially update a resource | No (often treated as yes in practice) |
| `DELETE` | Remove a resource | Yes |

**Idempotent** means calling it multiple times has the same effect as calling it once. `DELETE /orders/5` called five times still just results in order 5 being gone (same end state). But `POST /orders` called five times creates five separate new orders — clearly not idempotent.

---

### Q29. What does `@RequestMapping` do, and how does it relate to the HTTP-method-specific shortcuts?
**A.** `@RequestMapping` is the general-purpose annotation for mapping HTTP requests to handler methods — it can specify URL path, HTTP method, headers, and more.

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @RequestMapping(method = RequestMethod.GET)
    public List<Order> getAllOrders() {
        // ...
    }
}
```

In practice, nobody writes it this verbosely today — Spring provides shortcut annotations per HTTP method: `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping`. But `@RequestMapping` at the **class level** is still commonly used to define a shared base path (e.g., `/api/orders`) that all methods in the controller build on.

---

### Q30. How do Path Variables work?
**A.** Path variables capture dynamic segments of the URL — exactly how you identify a specific resource by ID.

```java
@GetMapping("/{id}")
public Order getOrderById(@PathVariable Long id) {
    return orderService.findById(id);
}
```

A request to `GET /api/orders/42` automatically binds `42` to `id`. Multiple path variables are supported:

```java
@GetMapping("/{orderId}/items/{itemId}")
public OrderItem getOrderItem(@PathVariable Long orderId, @PathVariable Long itemId) {
    return orderService.findItem(orderId, itemId);
}
```

---

### Q31. How do Request Parameters (`@RequestParam`) differ from Path Variables?
**A.** Query parameters (`?key=value`) are captured with `@RequestParam`, typically used for **filtering, sorting, or pagination** — optional modifiers on a request, rather than identifying a specific resource (which is what path variables are for).

```java
@GetMapping
public List<Order> getOrders(
        @RequestParam(required = false) String status,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) {
    return orderService.findOrders(status, page, size);
}
```

A request like `GET /api/orders?status=SHIPPED&page=1&size=20` binds `status="SHIPPED"`, `page=1`, `size=20` automatically. If `status` is omitted (`required = false`), it's simply `null` rather than causing an error.

---

### Q32. What does `@RequestBody` do?
**A.** For creating or updating resources, the client sends a JSON payload in the request body. `@RequestBody` tells Spring to deserialize that JSON directly into a Java object.

```java
@PostMapping
public Order createOrder(@RequestBody OrderRequest request) {
    return orderService.create(request);
}
```

Spring uses **Jackson** under the hood (auto-included via `spring-boot-starter-web`) to convert incoming JSON into your object, matching field names automatically.

---

### Q33. What is `ResponseEntity`, and why is it preferred over returning a plain object?
**A.** Returning a plain object from a controller works and Spring will serialize it to JSON with a default `200 OK`. But `ResponseEntity` gives **full control** over the status code, headers, *and* body.

```java
@GetMapping("/{id}")
public ResponseEntity<Order> getOrder(@PathVariable Long id) {
    Order order = orderService.findById(id);
    if (order == null) {
        return ResponseEntity.notFound().build();
    }
    return ResponseEntity.ok(order);
}
```

This is the recommended approach for production APIs, because REST clients rely heavily on **status codes** to understand what happened — not just body content.

---

### Q34. What are the most important HTTP status codes for a REST API?
**A.**

| Code | Meaning | When to use |
|---|---|---|
| `200 OK` | Success | Standard successful GET/PUT/PATCH |
| `201 Created` | Resource created | Successful POST that creates something |
| `204 No Content` | Success, nothing to return | Successful DELETE |
| `400 Bad Request` | Client sent invalid data | Failed validation |
| `401 Unauthorized` | Not authenticated | Missing/invalid credentials |
| `403 Forbidden` | Authenticated but not allowed | Insufficient permissions |
| `404 Not Found` | Resource doesn't exist | Invalid ID lookup |
| `409 Conflict` | Conflicting state | Duplicate resource, version conflict |
| `500 Internal Server Error` | Unexpected server-side failure | Uncaught exceptions |

```java
@PostMapping
public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
    Order created = orderService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
}
```

---

### Q35. What is a Response DTO, and why not just return the database entity directly?
**A.** DTO = "Data Transfer Object." A **Response DTO** is a plain object shaped specifically for what's sent back to the client — deliberately separate from the internal database entity.

**Reasons to avoid returning entities directly:**
- Avoids exposing internal fields (password hashes, internal audit columns) to the outside world
- Decouples the database schema from the API contract — one can change without breaking the other
- Lets you shape the response exactly how the frontend needs it, combining or renaming fields as needed

```java
public class OrderResponse {
    private Long id;
    private String customerName;
    private BigDecimal totalAmount;
    private String status;
    // constructors, getters
}
```

```java
@GetMapping("/{id}")
public OrderResponse getOrder(@PathVariable Long id) {
    Order order = orderService.findById(id);
    return new OrderResponse(order.getId(), order.getCustomer().getName(),
            order.getTotal(), order.getStatus().name());
}
```

---

### Q36. What is a Request DTO, and what security problem does it prevent?
**A.** A **Request DTO** represents exactly what the client is *allowed* to send when creating or updating a resource — nothing more.

```java
public class OrderRequest {
    @NotBlank
    private String customerName;

    @NotEmpty
    private List<OrderItemRequest> items;

    // getters and setters
}
```

This separation prevents **mass assignment** — a security issue where a client could sneak in extra fields (e.g., `"isAdmin": true`) that get accidentally bound directly onto a database entity, if requests were bound straight to entities instead of dedicated DTOs.

---

### Q37. What are some REST API best practices?
**A.**
- Use **nouns** for resource URLs, never verbs (`/orders`, not `/getOrders`)
- Use **plural nouns** consistently (`/orders`, not `/order`)
- **Nest resources logically** (`/orders/5/items` for items belonging to order 5)
- Use **proper HTTP status codes** — never return `200` for errors
- **Always validate input**, and never trust the client
- **Version your API** from day one
- **Support pagination** for any endpoint that can return a large list
- Use **consistent error response formats** across the entire API
- Keep responses **flat and predictable**; avoid deeply nested surprises
- **Document endpoints** (e.g., via OpenAPI/Swagger)

---

### Q38. What are the different API Versioning strategies?
**A.** APIs evolve, but existing clients can't be broken every time a change is made. Versioning lets multiple API versions co-exist so consumers migrate at their own pace.

**1. URL Path Versioning (most common, most explicit):**
```
GET /api/v1/orders
GET /api/v2/orders
```
```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderControllerV1 { }

@RestController
@RequestMapping("/api/v2/orders")
public class OrderControllerV2 { }
```

**2. Header Versioning:**
```
GET /api/orders
Headers: X-API-Version: 2
```

**3. Query Parameter Versioning:**
```
GET /api/orders?version=2
```

**4. Media Type (Accept header) Versioning:**
```
Accept: application/vnd.myapp.v2+json
```

**URL path versioning is the most widely adopted** in real-world APIs because it's simplest to understand, test, and cache — even though purists argue it's not "the most RESTful" approach. Pragmatism usually wins.

---

## Quick Recap Table

| Concept | One-line takeaway |
|---|---|
| IoC | Control of object creation moves from your code to the container |
| DI | The technique used to achieve IoC — dependencies are supplied, not self-created |
| Bean | Any object managed by the Spring container |
| Bean Lifecycle | Instantiate → inject → init callbacks → ready → destroy callbacks |
| ApplicationContext | The feature-rich container almost all Spring apps use |
| Bean Scopes | `singleton` (default, shared) vs `prototype` (new instance per request) and web-specific scopes |
| Profiles | Environment-specific beans/config, activated via `spring.profiles.active` |
| Spring Boot | Convention-over-configuration layer on top of Spring |
| Auto-Configuration | Classpath-driven, conditional bean configuration |
| Starters | One dependency = a whole compatible bundle |
| Embedded Server | The app *is* the server — no separate install needed |
| REST | Resource-based, stateless, HTTP-method-driven API design |
| DTOs | Separate shapes for what goes in (`Request`) and out (`Response`) of an API, decoupled from entities |
| API Versioning | URL path versioning is the pragmatic default choice |
