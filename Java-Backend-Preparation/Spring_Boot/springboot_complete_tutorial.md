# The Complete Spring Boot Tutorial
### From Zero to Production — A Plain-English, Concept-by-Concept Guide

> This guide walks through the Spring and Spring Boot ecosystem the way a senior developer would explain it to a junior developer over many coffee-fueled sessions. Every concept is explained in plain language first, then backed up with code.

---

## Table of Contents

1. Spring Framework Core (Foundation)
2. Spring Boot Fundamentals
3. REST API Development
4. Spring MVC
5. Validation
6. Exception Handling
7. Spring Data JPA
8. Hibernate Advanced
9. Database Migration
10. Spring Security
11. JWT Authentication
12. File Upload
13. Email & Notifications
14. Scheduling
15. Asynchronous Programming
16. Spring AOP
17. Caching
18. Logging
19. Testing
20. OpenAPI Documentation
21. Spring Boot Actuator
22. Spring Boot Profiles
23. Docker
24. Redis
25. Kafka
26. RabbitMQ
27. Microservices
28. Cloud Deployment
29. Performance Optimization
30. Production Best Practices
31. Design Patterns in Spring
32. Spring Boot Internals (Expert Level)

---

# 1. Spring Framework Core (Foundation)

## 1.1 Introduction to Spring

Let's start from the very beginning. Before Spring existed, Java developers writing enterprise applications had to deal with a LOT of boilerplate — manually creating objects, wiring them together, managing transactions by hand, and writing tons of "plumbing" code that had nothing to do with actual business logic.

Spring showed up to solve exactly this problem. Think of Spring as a giant toolbox and a "manager" that takes care of creating your objects, connecting them to each other, and managing their entire lifecycle — so you, the developer, can focus on writing the code that actually matters (the business logic).

Spring is not just one thing — it's an umbrella project made up of many modules:

- **Spring Core** — the foundation (IoC container, DI)
- **Spring MVC** — for building web applications and REST APIs
- **Spring Data** — for working with databases
- **Spring Security** — for authentication and authorization
- **Spring Boot** — a layer on top of all of this that removes configuration pain

Here's the simplest way to think about it: **Spring manages objects so you don't have to.** That's it. Everything else builds on top of that one idea.

## 1.2 Inversion of Control (IoC)

This is the single most important idea in Spring, and it sounds scarier than it is. Let's break down the name:

- "Control" = who creates objects and decides how they interact
- "Inversion" = flipping who's in charge of that

**Normal way (without Spring):** If class `A` needs class `B`, then class `A` itself creates an instance of `B` using `new B()`. Class `A` is in control.

**Spring way (IoC):** Class `A` doesn't create `B` itself. Instead, it just says "I need a `B`," and some external system (the Spring Container) hands it a ready-made `B`. The control of "who creates what" has been inverted — it's no longer class `A`'s job, it's the container's job.

Why does this matter? Because now class `A` doesn't care HOW `B` is built, what dependencies `B` itself has, or when `B` gets destroyed. That's now somebody else's problem (the container's). This makes your code:

- Easier to test (you can substitute a fake/mock `B`)
- Easier to change (swap `B` for a different implementation without touching `A`)
- Less tightly coupled

IoC is a general principle. Dependency Injection (next topic) is simply the most common **technique** used to achieve IoC in Spring.

## 1.3 Dependency Injection (DI)

Dependency Injection is how Spring actually implements Inversion of Control. Instead of a class creating its own dependencies, the dependencies are "injected" into it from outside.

There are three common ways to do this in Spring:

**1. Constructor Injection (recommended in almost all cases today):**

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    // Spring sees this constructor and automatically supplies a PaymentService bean
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

**3. Field Injection (common in tutorials, discouraged in real production code):**

```java
@Service
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Why is constructor injection preferred? Because:
- It makes dependencies explicit and mandatory (you can't create the object without them)
- It works nicely with `final` fields — meaning once set, the dependency can't accidentally be changed later
- It makes unit testing trivial — you just pass a mock into the constructor, no reflection tricks needed

A simple mental picture: dependency injection is like a restaurant kitchen. The chef (your class) doesn't go out and personally grow vegetables or raise chickens (create its own dependencies). Someone else — the supplier (Spring container) — delivers exactly the ingredients the chef ordered, ready to use.

## 1.4 Bean

In Spring terminology, a **bean** is simply an object that is created, configured, and managed by the Spring container — instead of being managed manually by your own code with `new`.

Any object can become a bean. What makes it "a bean" isn't something special about the class itself; it's the fact that Spring is the one responsible for creating and managing that particular instance.

You tell Spring "this class should be a bean" in a few ways:

```java
@Component
public class NotificationSender {
    // Spring will detect this class during component scanning
    // and register it as a bean automatically.
}
```

Or manually, inside a configuration class:

```java
@Configuration
public class AppConfig {

    @Bean
    public NotificationSender notificationSender() {
        return new NotificationSender();
    }
}
```

Once something is a bean, Spring keeps track of it inside a big internal registry, and hands it out to any other bean that needs it (via dependency injection).

## 1.5 Bean Lifecycle

Every bean goes through a well-defined lifecycle, managed entirely by the Spring container. Understanding this lifecycle helps a lot when debugging weird startup issues. Here's the flow, step by step:

1. **Instantiation** — Spring creates the object (calls the constructor)
2. **Populate properties** — Spring injects dependencies (DI happens here)
3. **`BeanNameAware`, `BeanFactoryAware`, etc.** — if the bean implements these special interfaces, Spring calls their callback methods to give the bean awareness of its own name or the container itself
4. **`@PostConstruct`** — a method you can annotate to run custom initialization logic right after dependencies are injected
5. **`InitializingBean.afterPropertiesSet()`** — an older, interface-based alternative to `@PostConstruct`
6. **Custom init-method** (defined via `@Bean(initMethod = "...")`)
7. **Bean is ready and in use** — this is the normal operating phase where your application actually uses the bean
8. **`@PreDestroy`** — called just before the bean is destroyed (e.g., when the application shuts down)
9. **`DisposableBean.destroy()`** — interface-based alternative to `@PreDestroy`

Example:

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

This is incredibly useful for things like opening/closing connections, warming up caches, or validating configuration at startup.

## 1.6 Spring Container

The **Spring Container** is the engine at the heart of everything. Its job is to:

- Create beans
- Wire beans together (dependency injection)
- Manage the full lifecycle of every bean
- Destroy beans cleanly on shutdown

You can think of the container as a big factory floor: you tell it what parts (beans) exist and how they connect, and it assembles the whole machine (your application) for you at startup.

There are two main flavors of container in Spring:

- **`BeanFactory`** — the most basic container, lazy-loads beans (creates them only when requested). Rarely used directly today.
- **`ApplicationContext`** — a more feature-rich container built on top of `BeanFactory`. This is what virtually all real Spring applications use.

## 1.7 ApplicationContext

`ApplicationContext` is the container implementation almost every Spring Boot application actually uses under the hood. It extends `BeanFactory` and adds a lot of enterprise-friendly features:

- Eager (upfront) bean initialization by default, so configuration errors are caught immediately at startup rather than later at runtime
- Event publishing (`ApplicationEventPublisher`) — beans can publish and listen to events
- Internationalization (i18n) support
- Easy access to resources (files, classpath resources, URLs)
- Automatic integration with AOP

In a Spring Boot app, you rarely construct an `ApplicationContext` yourself — `SpringApplication.run()` does it for you. But you CAN grab it if you need to:

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(MyApp.class, args);
        NotificationSender sender = context.getBean(NotificationSender.class);
    }
}
```

## 1.8 Bean Scopes

By default, every bean in Spring is a **singleton** — meaning there is exactly ONE instance of that bean per Spring container, shared everywhere it's needed. But Spring supports several other scopes for different situations:

| Scope | Meaning |
|---|---|
| `singleton` (default) | One shared instance for the entire application |
| `prototype` | A brand-new instance every time the bean is requested |
| `request` | One instance per HTTP request (web apps only) |
| `session` | One instance per HTTP session (web apps only) |
| `application` | One instance per ServletContext |

Example of declaring scope explicitly:

```java
@Component
@Scope("prototype")
public class ReportGenerator {
    // A new instance is created every single time this bean is injected or fetched
}
```

When should you use `prototype` instead of the default `singleton`? Whenever the bean holds **mutable state specific to one particular usage** — for example, if `ReportGenerator` accumulates data during report generation and you don't want different requests stepping on each other's data.

## 1.9 Spring Annotations

Annotations are how modern Spring apps configure almost everything — instead of the old-school XML configuration files. Here are the ones you'll see constantly:

- `@Component` — generic stereotype, marks a class as a Spring-managed bean
- `@Service` — semantically marks a class as containing business logic (functionally identical to `@Component`, but communicates intent)
- `@Repository` — marks a class as a data-access layer component; also enables automatic translation of database exceptions into Spring's unified exception hierarchy
- `@Controller` / `@RestController` — marks a class as a web layer component that handles HTTP requests
- `@Autowired` — tells Spring to inject a dependency automatically
- `@Configuration` — marks a class as a source of bean definitions
- `@Bean` — marks a method inside a `@Configuration` class as producing a bean
- `@Qualifier` — used alongside `@Autowired` when there are multiple beans of the same type, to specify exactly which one you want
- `@Value` — injects a value from a properties file directly into a field

Example combining a few of these:

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

## 1.10 Java Configuration

Before annotations became popular, Spring apps were configured entirely through big, verbose XML files. Java Configuration replaced that with regular Java classes, which gives you type safety, refactoring support, and the ability to write actual logic in your configuration.

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripePaymentGateway("api-key-here");
    }

    @Bean
    public OrderService orderService(PaymentGateway paymentGateway) {
        // Spring automatically passes in the paymentGateway bean defined above
        return new OrderService(paymentGateway);
    }
}
```

Every method annotated `@Bean` inside a `@Configuration` class becomes a bean definition, registered by its return type (and method name, by default, as the bean name).

## 1.11 Component Scanning

Rather than manually registering every single `@Component`, `@Service`, `@Repository`, etc. one by one, Spring can automatically scan your codebase and find them for you. This is called **component scanning**.

```java
@SpringBootApplication  // this already includes @ComponentScan internally
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

By default, Spring Boot scans the package containing your main application class, and every sub-package underneath it. That's why the convention is to put your main class at the root package (like `com.mycompany.myapp`) — so everything underneath gets picked up automatically.

If you need to scan additional packages outside that structure:

```java
@SpringBootApplication
@ComponentScan(basePackages = {"com.mycompany.myapp", "com.othercompany.shared"})
public class MyApp { }
```

## 1.12 Profiles

Real applications behave differently depending on environment — you don't want your local development database credentials being used in production! **Profiles** let you define environment-specific beans and configuration.

```java
@Service
@Profile("dev")
public class MockPaymentService implements PaymentService {
    // Fake payment service used only during local development
}

@Service
@Profile("prod")
public class RealPaymentService implements PaymentService {
    // Actual payment gateway integration used in production
}
```

You activate a profile via a property:

```properties
spring.profiles.active=dev
```

Or as a command-line argument when starting the app:

```
java -jar myapp.jar --spring.profiles.active=prod
```

Only the bean matching the active profile gets created — the other one is completely ignored by the container.

## 1.13 Environment & Properties

The `Environment` abstraction is Spring's unified way of representing configuration coming from many different sources — property files, environment variables, command-line arguments, JVM system properties — all merged together and accessible through one consistent interface.

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

Spring resolves properties using a clear precedence order (roughly, from highest to lowest priority): command-line arguments, JVM system properties, OS environment variables, then `application.properties`/`application.yml` files. This means you can always override a config value at deployment time without touching the packaged application at all.

## 1.14 External Configuration

"External configuration" simply means: configuration values should live OUTSIDE your compiled code, so you can change behavior without recompiling or redeploying.

Spring Boot supports externalizing configuration through many channels, checked in a specific priority order:

1. Command-line arguments
2. `SPRING_APPLICATION_JSON` environment variable
3. Servlet parameters
4. OS environment variables
5. `application.properties` / `application.yml` (and profile-specific variants like `application-prod.yml`)
6. Default values hardcoded in `@Value` annotations

This layered approach means the exact same packaged JAR file can behave completely differently in dev, staging, and production — just by changing external config, never the code itself. This is a core principle of "twelve-factor app" design, and Spring Boot embraces it fully.

---
# 2. Spring Boot Fundamentals

## 2.1 What is Spring Boot

Here's the honest truth: plain Spring is powerful but was historically painful to set up. You had to manually configure a web server, wire up dozens of beans, write mountains of XML, and manage a spaghetti of dependency versions that had to be *just* compatible with each other.

**Spring Boot fixes all of that.** It's not a replacement for Spring — it's built on top of Spring, and its entire mission is: **"convention over configuration."** Meaning: Spring Boot makes smart, sensible default decisions for you, so you can go from zero to a running application in minutes, not days.

Concretely, Spring Boot gives you:

- Auto-configuration (Spring guesses what you need and configures it automatically)
- An embedded web server (no need to install Tomcat separately)
- Starter dependencies (one dependency pulls in everything related, at compatible versions)
- Production-ready features out of the box (health checks, metrics)
- A single executable JAR you can just run with `java -jar`

## 2.2 Spring Boot Architecture

Spring Boot isn't magic — it's built from a small number of core ideas stacked on top of each other:

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

When your application starts:

1. The `main()` method calls `SpringApplication.run(...)`
2. Spring Boot creates the `ApplicationContext`
3. Auto-configuration classes run, inspecting what's on your classpath and configuring beans accordingly (e.g., "oh, I see a JPA driver on the classpath — let me configure a `DataSource` and `EntityManagerFactory` automatically")
4. Component scanning finds your `@Component`/`@Service`/`@Controller` classes
5. The embedded server starts up and begins listening for HTTP requests

## 2.3 Spring Initializr

Spring Initializr (available at start.spring.io, and built directly into IntelliJ/VS Code/Eclipse) is basically a project generator. Instead of manually setting up folder structures, `pom.xml`/`build.gradle` files, and boilerplate classes, you just:

1. Pick your build tool (Maven or Gradle)
2. Pick your language (Java, Kotlin, Groovy)
3. Pick your Spring Boot version
4. Pick your dependencies (Web, JPA, Security, etc.) from a searchable list
5. Click "Generate" — it downloads a ready-to-run zip project

This saves enormous setup time and guarantees you start with compatible dependency versions.

## 2.4 Starter Dependencies

This is one of Spring Boot's best ideas. Instead of adding ten separate libraries individually (and hoping their versions play nicely together), you add ONE "starter" and it pulls in everything related, at tested, compatible versions.

Common starters:

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

`spring-boot-starter-web`, for example, silently brings in Spring MVC, Jackson (for JSON), validation support, and an embedded Tomcat server — all in one line.

## 2.5 Auto Configuration

This is the "magic" that makes Spring Boot feel effortless. Auto-configuration works by scanning what's available on your classpath and automatically configuring beans that make sense given what it finds — WITHOUT you writing a single line of configuration.

For example: if Spring Boot detects the H2 database driver on your classpath, and no `DataSource` bean has been manually defined, it automatically configures an in-memory H2 `DataSource` for you.

Under the hood, this works through classes annotated `@Configuration` combined with **conditional annotations**:

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

`@ConditionalOnClass` means "only apply this configuration if a specific class is present on the classpath." `@ConditionalOnMissingBean` means "only apply this if the developer hasn't already defined their own bean of this type" — so your own explicit configuration always wins over auto-configuration.

You enable auto-configuration with `@EnableAutoConfiguration`, though in practice you'll almost never write that directly — it's bundled inside `@SpringBootApplication`.

## 2.6 Embedded Servers (Tomcat, Jetty)

Traditionally, you'd build a WAR file and deploy it to a separately-installed application server (like Tomcat) running on the machine. Spring Boot flips this: the server is **embedded directly inside your application's JAR file**.

This means:
- You run your app with a single command: `java -jar myapp.jar`
- No separate server installation, configuration, or version-matching headaches
- Your application IS the server — it starts the server itself, in its own `main()` method

By default, Spring Boot uses **Tomcat** when you include `spring-boot-starter-web`. But you can swap it for **Jetty** or **Undertow** easily:

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

## 2.7 application.properties

This is the classic, simplest way to externalize configuration in Spring Boot. It's just a flat key-value file, placed at `src/main/resources/application.properties`.

```properties
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
spring.jpa.hibernate.ddl-auto=update
logging.level.org.springframework=INFO
```

Spring Boot reads this file automatically at startup — no extra setup required. Every property here maps to some internal configuration Spring Boot uses to configure beans (server port, database connection, JPA behavior, logging levels, and so on).

## 2.8 application.yml

YAML is an alternative format to `.properties`, and many teams prefer it because it lets you express nested configuration hierarchically instead of repeating dotted prefixes over and over.

The exact same configuration as above, in YAML:

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

Both formats do exactly the same thing — it's purely a matter of preference and readability. You can even use both, though it's best practice to pick just one per project to avoid confusion.

## 2.9 Configuration Properties

Reading values one at a time with `@Value("${some.key}")` gets tedious once you have a lot of related settings. `@ConfigurationProperties` lets you bind an entire block of configuration to a strongly-typed Java class in one shot.

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

    // getters and setters required for binding to work
}
```

Now anywhere in your code, you can inject `MailProperties` and get fully-typed, validated access to that whole configuration block — much cleaner than sprinkling `@Value` everywhere.

## 2.10 CommandLineRunner

Sometimes you need to run a bit of code exactly once, right after the application has fully started up — for example, seeding a database with default data, or printing a startup banner with diagnostic info. `CommandLineRunner` is a functional interface built exactly for this.

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

Spring Boot automatically detects any bean implementing `CommandLineRunner` and calls its `run()` method right after the application context has fully loaded. If you have multiple runners, you can control their order with `@Order`.

## 2.11 Banner Customization

You know that ASCII-art Spring logo that prints in the console every time you start a Spring Boot app? That's the "banner," and yes — you can customize it, or turn it off entirely.

To customize it, just create a plain text file named `banner.txt` in `src/main/resources`:

```
  __  __         _
 |  \/  |_   _  / \   _ __  _ __
 | |\/| | | | |/ _ \ | '_ \| '_ \
 | |  | | |_| / ___ \| |_) | |_) |
 |_|  |_|\__, /_/   \_\ .__/| .__/
         |___/        |_|   |_|
::  My Awesome App  ::
```

To disable it entirely (useful in production logs where you don't want visual noise):

```properties
spring.main.banner-mode=off
```

Small feature, but it's a nice, harmless touch teams use to make their console output recognizable and a bit fun.

## 2.12 Spring Boot DevTools

DevTools is a small dependency purely meant to make the local development experience faster and less annoying:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

What it gives you:

- **Automatic restart** — whenever you change a class and recompile, Spring Boot automatically restarts the application (much faster than a full cold restart, because it uses two classloaders — one for your code that changes often, one for third-party libraries that don't)
- **LiveReload** — automatically refreshes your browser when static resources (HTML/CSS/JS) change
- **Sensible dev-time defaults** — e.g., disables template caching so you see changes instantly

DevTools is automatically excluded from production builds when you package your app as an executable JAR, so there's no risk of it accidentally shipping to production.

---
# 3. REST API Development

## 3.1 REST Principles

REST (Representational State Transfer) is an architectural style for designing networked applications — it's not a protocol or a library, it's a set of conventions. A truly "RESTful" API generally follows these principles:

- **Statelessness** — the server does not store any client session state between requests. Every request must contain everything needed to understand and process it.
- **Resource-based URLs** — URLs represent "nouns" (things), not "verbs" (actions). `/orders/5` represents order number 5, not an action.
- **Standard HTTP methods** — you use GET, POST, PUT, PATCH, DELETE to represent actions on resources, instead of inventing your own verbs in the URL.
- **Uniform interface** — consistent conventions across your entire API (consistent naming, consistent response shapes, consistent error formats).
- **Client-server separation** — the frontend and backend evolve independently as long as the contract (the API) stays consistent.
- **Cacheability** — responses should indicate whether they can be cached, to improve performance.

A well-designed REST endpoint for "get order number 5" looks like `GET /orders/5` — not `GET /getOrder?id=5`. The HTTP method (GET) already tells you the action; the URL just identifies the resource.

## 3.2 HTTP Methods

Each HTTP method has a specific, conventional meaning in REST:

| Method | Meaning | Idempotent? |
|---|---|---|
| GET | Retrieve a resource, no side effects | Yes |
| POST | Create a new resource | No |
| PUT | Replace a resource entirely | Yes |
| PATCH | Partially update a resource | No (in practice, often treated as yes) |
| DELETE | Remove a resource | Yes |

"Idempotent" means: calling it multiple times has the exact same effect as calling it once. `DELETE /orders/5` called five times in a row still just results in order 5 being gone — same end state. But `POST /orders` called five times creates five separate new orders — definitely not idempotent.

## 3.3 Request Mapping

`@RequestMapping` is the general-purpose annotation for mapping HTTP requests to handler methods. It can specify the URL path, HTTP method, headers, and more.

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

In practice, nobody writes it this verbosely anymore — Spring provides shortcut annotations for each HTTP method (covered as part of Spring MVC below): `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping`. But `@RequestMapping` at the class level is still commonly used to define a shared base path, like `/api/orders` above, that all methods in the controller build on top of.

## 3.4 Path Variables

Path variables let you capture dynamic parts of the URL directly, which is exactly how you identify a specific resource by ID.

```java
@GetMapping("/{id}")
public Order getOrderById(@PathVariable Long id) {
    return orderService.findById(id);
}
```

A request to `GET /api/orders/42` will automatically bind `42` to the `id` parameter. You can have multiple path variables too:

```java
@GetMapping("/{orderId}/items/{itemId}")
public OrderItem getOrderItem(@PathVariable Long orderId, @PathVariable Long itemId) {
    return orderService.findItem(orderId, itemId);
}
```

## 3.5 Request Parameters

Query parameters (the `?key=value` part of a URL) are captured with `@RequestParam`. These are typically used for filtering, sorting, or pagination — optional modifiers on a request, rather than identifying a specific resource.

```java
@GetMapping
public List<Order> getOrders(
        @RequestParam(required = false) String status,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) {
    return orderService.findOrders(status, page, size);
}
```

A request like `GET /api/orders?status=SHIPPED&page=1&size=20` binds `status="SHIPPED"`, `page=1`, `size=20` automatically. If `status` is omitted, since it's marked `required = false`, it'll simply be `null` rather than causing an error.

## 3.6 Request Body

For creating or updating resources, the client typically sends a JSON payload in the body of the request. `@RequestBody` tells Spring to deserialize that JSON directly into a Java object.

```java
@PostMapping
public Order createOrder(@RequestBody OrderRequest request) {
    return orderService.create(request);
}
```

Spring uses Jackson under the hood (included automatically via `spring-boot-starter-web`) to convert incoming JSON into your `OrderRequest` object, matching field names automatically.

## 3.7 ResponseEntity

While you CAN just return a plain object from a controller method (and Spring will serialize it to JSON with a default HTTP 200 status), `ResponseEntity` gives you full control over the response — status code, headers, AND body.

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

This is the recommended approach for any real production API, because REST clients rely heavily on status codes to understand what happened — not just the body content.

## 3.8 HttpStatus

HTTP status codes communicate the outcome of a request in a standardized way that every client (and every developer) understands instantly, without needing to parse the response body. The commonly used ones in REST APIs:

| Code | Meaning | When to use |
|---|---|---|
| 200 OK | Success | Standard successful GET/PUT/PATCH |
| 201 Created | Resource created | Successful POST that creates something |
| 204 No Content | Success, nothing to return | Successful DELETE |
| 400 Bad Request | Client sent invalid data | Failed validation |
| 401 Unauthorized | Not authenticated | Missing/invalid credentials |
| 403 Forbidden | Authenticated but not allowed | Insufficient permissions |
| 404 Not Found | Resource doesn't exist | Invalid ID lookup |
| 409 Conflict | Conflicting state | Duplicate resource, version conflict |
| 500 Internal Server Error | Unexpected server-side failure | Uncaught exceptions |

```java
@PostMapping
public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
    Order created = orderService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
}
```

## 3.9 Response DTO

DTO stands for "Data Transfer Object." A Response DTO is a plain object specifically shaped for what you want to SEND BACK to the client — deliberately separate from your internal database entity.

Why not just return the entity directly? A few very good reasons:

- You often don't want to expose internal fields (like a password hash, or internal audit columns) to the outside world
- Your database schema and your API contract shouldn't be tightly coupled — you want the freedom to change one without breaking the other
- You can shape the response exactly the way the frontend needs it, combining or renaming fields as needed

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

## 3.10 Request DTO

Similarly, a Request DTO represents exactly what the client is ALLOWED to send when creating or updating a resource — nothing more.

```java
public class OrderRequest {
    @NotBlank
    private String customerName;

    @NotEmpty
    private List<OrderItemRequest> items;

    // getters and setters
}
```

This separation is powerful because it prevents a security issue called **mass assignment** — where a client could sneak in extra fields (like `"isAdmin": true`) that get accidentally bound directly onto your database entity, if you were binding requests straight to entities instead of using a dedicated DTO.

## 3.11 REST Best Practices

A grab-bag of practices that separate a "works on my machine" API from a genuinely production-quality one:

- Use nouns for resource URLs, never verbs (`/orders`, not `/getOrders`)
- Use plural nouns consistently (`/orders`, not `/order`)
- Nest resources logically (`/orders/5/items` for items belonging to order 5)
- Use proper HTTP status codes — don't return 200 for errors
- Always validate input, and never trust the client
- Version your API from day one (see below)
- Support pagination for any endpoint that can return a large list
- Use consistent error response formats across the entire API
- Keep responses flat and predictable where possible; avoid deeply nested surprises
- Document endpoints (see the OpenAPI/Swagger section later in this guide)

## 3.12 API Versioning

APIs evolve over time, but you can't just break existing clients every time you make a change. Versioning lets multiple versions of your API co-exist so consumers can migrate at their own pace. Common strategies:

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

URL path versioning is by far the most widely adopted in real-world APIs because it's the simplest to understand, test, and cache — even though purists argue it's not "the most RESTful" approach. Pragmatism usually wins here.

---
# 4. Spring MVC

## 4.1 DispatcherServlet

`DispatcherServlet` is the "front controller" of every Spring web application — meaning EVERY single incoming HTTP request passes through it first, before being routed anywhere else. Think of it as the receptionist at the front desk of a building: every visitor (request) checks in with the receptionist first, who then directs them to the right department (controller).

Its job:

1. Receive the incoming HTTP request
2. Consult the `HandlerMapping` to figure out which controller method should handle it
3. Invoke that controller method
4. Take the return value and hand it to the appropriate `HandlerAdapter`/`View` resolution
5. Send the final HTTP response back to the client

In Spring Boot, `DispatcherServlet` is auto-configured for you the moment you include `spring-boot-starter-web` — you never manually register it (unlike old-school Spring MVC + XML setups, where you had to configure it by hand in `web.xml`).

## 4.2 Controller

`@Controller` marks a class as a Spring MVC "controller" — a component whose job is to handle incoming web requests. Traditionally, `@Controller` methods return **view names** (like a Thymeleaf or JSP template) rather than raw data, making it the right choice for apps that render server-side HTML pages.

```java
@Controller
public class HomeController {

    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("message", "Welcome!");
        return "home"; // resolves to a template named home.html
    }
}
```

## 4.3 RestController

`@RestController` is a convenience annotation that combines `@Controller` + `@ResponseBody`. It tells Spring: "every method in this class returns data directly (usually as JSON), not a view name." This is what virtually every REST API controller uses.

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll(); // serialized directly to JSON
    }
}
```

Without `@ResponseBody`, Spring would try to interpret the returned `List<Product>` as a view name to resolve — which would fail. `@RestController` handles that for you automatically.

## 4.4 RequestMapping

Already covered in the REST section above, but worth reinforcing here in the MVC context: `@RequestMapping` (and its HTTP-method-specific shortcuts `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping`) is the core mechanism Spring MVC uses to map incoming requests to specific controller methods, based on URL pattern, HTTP method, headers, or content type.

## 4.5 Handler Mapping

`HandlerMapping` is the internal component responsible for figuring out WHICH controller method should handle an incoming request. When a request comes in, `DispatcherServlet` asks the registered `HandlerMapping` implementations: "given this URL and HTTP method, who should handle this?"

The most common implementation, `RequestMappingHandlerMapping`, scans all your `@RequestMapping`-annotated methods at startup and builds an internal lookup table mapping URL patterns to methods. This is why Spring can efficiently route thousands of requests per second without scanning your whole codebase on every single request — the mapping is pre-computed once, at startup.

## 4.6 View Resolver

For traditional server-rendered web apps (not REST APIs), controllers return a logical view name (a `String`, like `"home"`), and the `ViewResolver` is responsible for translating that logical name into an actual renderable view — usually a template file.

```properties
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

With this configuration, returning `"home"` from a controller resolves to `classpath:/templates/home.html`. Spring Boot auto-configures a Thymeleaf view resolver automatically the moment it detects Thymeleaf on the classpath — no manual XML setup needed.

## 4.7 Data Binding

Data binding is the automatic process of converting incoming HTTP request data (form fields, query parameters, path variables, JSON body) into Java objects. You've already seen this happening with `@RequestBody`, `@RequestParam`, and `@PathVariable` — all of these rely on Spring's underlying data binding machinery.

```java
@GetMapping("/search")
public List<Product> search(ProductSearchCriteria criteria) {
    // Spring automatically binds matching query params
    // (e.g. ?name=phone&minPrice=100) onto the criteria object's fields
    return productService.search(criteria);
}
```

Spring also supports custom `Converter`/`PropertyEditor` implementations if you need to bind non-trivial types (like converting a date string in a specific custom format into a `LocalDate`).

## 4.8 Validation

Covered in depth in the dedicated Validation section below, but the short version in the MVC context: you annotate your request DTOs with constraints (`@NotBlank`, `@Email`, `@Min`, etc.), then add `@Valid` in your controller method signature, and Spring automatically validates the incoming data before your method body even runs — throwing a `MethodArgumentNotValidException` if validation fails.

```java
@PostMapping
public ResponseEntity<Product> create(@Valid @RequestBody ProductRequest request) {
    // guaranteed valid data by the time we reach this line
}
```

## 4.9 Exception Handling

Also covered in its own dedicated section, but in the MVC context specifically: Spring MVC lets you centralize how exceptions thrown anywhere in your controllers get translated into proper HTTP error responses, using `@ControllerAdvice` combined with `@ExceptionHandler` methods — instead of littering try/catch blocks throughout every controller.

## 4.10 Interceptors

Interceptors let you run custom logic BEFORE and AFTER a request is handled by a controller — without touching the controller code itself. Think of them as a lightweight, web-specific alternative to AOP, focused specifically on the HTTP request/response lifecycle.

```java
public class LoggingInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        System.out.println("Incoming request: " + request.getRequestURI());
        return true; // return false to short-circuit and stop the request here
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        System.out.println("Request completed: " + request.getRequestURI());
    }
}
```

Register it:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoggingInterceptor()).addPathPatterns("/api/**");
    }
}
```

Common real-world uses: logging every request, checking a custom header before allowing access, measuring request duration, or injecting tenant context in a multi-tenant application.

---
# 5. Validation

## 5.1 Bean Validation

Bean Validation is a Java specification (not Spring-specific) that defines a standard, annotation-based way to validate objects. Rather than writing manual `if` checks everywhere (`if (name == null || name.isEmpty()) throw ...`), you declare constraints directly on your class fields, and a validation engine enforces them automatically.

```java
public class UserRequest {
    @NotBlank
    private String username;

    @Email
    private String email;

    @Min(18)
    private int age;
}
```

Spring Boot integrates this seamlessly — just add the `spring-boot-starter-validation` dependency, and it wires everything up automatically.

## 5.2 Jakarta Validation

Jakarta Validation (formerly known as "Bean Validation," under the old `javax.validation` namespace before the Jakarta EE rebrand) is the actual specification that defines these annotations (`@NotNull`, `@Size`, `@Pattern`, etc.) and the validation engine's contract. **Hibernate Validator** is the most common implementation of this spec (and yes, it's a different thing from Hibernate ORM — just built by the same organization).

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

This one dependency brings in Hibernate Validator, which implements the Jakarta Validation spec fully, and Spring Boot auto-configures it so it just works.

## 5.3 Validation Annotations

Here are the constraint annotations you'll use constantly:

| Annotation | Meaning |
|---|---|
| `@NotNull` | Value must not be null |
| `@NotBlank` | String must not be null AND not just whitespace |
| `@NotEmpty` | Collection/String must not be null AND not empty |
| `@Size(min=, max=)` | String/Collection length must be within range |
| `@Min` / `@Max` | Numeric value must be within range |
| `@Email` | Must be a valid email format |
| `@Pattern(regexp=)` | Must match a given regular expression |
| `@Positive` / `@Negative` | Numeric sign constraints |
| `@Past` / `@Future` | Date must be in the past/future |

```java
public class RegistrationRequest {
    @NotBlank
    @Size(min = 3, max = 20)
    private String username;

    @Email
    @NotBlank
    private String email;

    @Pattern(regexp = "^(?=.*[A-Z])(?=.*\\d).{8,}$", message = "Password must be at least 8 characters with an uppercase letter and a digit")
    private String password;

    @Past
    private LocalDate dateOfBirth;
}
```

And in the controller:

```java
@PostMapping("/register")
public ResponseEntity<?> register(@Valid @RequestBody RegistrationRequest request) {
    // if validation fails, Spring throws MethodArgumentNotValidException
    // BEFORE this method body even runs
    userService.register(request);
    return ResponseEntity.status(HttpStatus.CREATED).build();
}
```

## 5.4 Custom Validation

Sometimes the built-in annotations aren't enough — you need business-specific rules (like "username must not already exist in the database" or "start date must be before end date"). You can write your own custom validation annotation:

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = UniqueUsernameValidator.class)
public @interface UniqueUsername {
    String message() default "Username already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

```java
public class UniqueUsernameValidator implements ConstraintValidator<UniqueUsername, String> {

    private final UserRepository userRepository;

    public UniqueUsernameValidator(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public boolean isValid(String username, ConstraintValidatorContext context) {
        return !userRepository.existsByUsername(username);
    }
}
```

Now you can use `@UniqueUsername` on a field exactly like any built-in annotation. This keeps your business validation rules declarative and reusable, rather than scattered across service methods.

## 5.5 Validation Groups

Sometimes the same object needs DIFFERENT validation rules depending on the context. For example, when creating a user you might require a password, but when updating a user's profile, the password field should be optional (and left untouched if not provided). Validation groups solve exactly this problem.

```java
public interface OnCreate {}
public interface OnUpdate {}

public class UserRequest {
    @NotNull(groups = OnUpdate.class)
    private Long id;

    @NotBlank(groups = {OnCreate.class, OnUpdate.class})
    private String username;

    @NotBlank(groups = OnCreate.class)
    private String password;
}
```

```java
@PostMapping
public void create(@Validated(OnCreate.class) @RequestBody UserRequest request) { }

@PutMapping
public void update(@Validated(OnUpdate.class) @RequestBody UserRequest request) { }
```

Notice we switch from `@Valid` to `@Validated` here — `@Validated` is Spring's own annotation, and it's the one that supports specifying which group(s) should be checked.

## 5.6 Global Validation Errors

When validation fails, Spring throws a `MethodArgumentNotValidException` containing ALL the individual field errors bundled together. Instead of returning Spring's default (fairly ugly) error response, most production apps catch this globally and format it nicely:

```java
@RestControllerAdvice
public class GlobalValidationHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationErrors(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
                errors.put(error.getField(), error.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }
}
```

Now, instead of a cryptic stack trace, the client gets back something clean and actionable, like:

```json
{
  "username": "Username already exists",
  "email": "must be a well-formed email address"
}
```

We'll dig much deeper into this `@ControllerAdvice` pattern in the next section.

---

# 6. Exception Handling

## 6.1 Checked vs Unchecked

Java exceptions split into two families, and understanding the difference shapes how you design error handling in a Spring Boot app.

**Checked exceptions** (extend `Exception` directly) — the compiler FORCES you to either catch them or declare them with `throws`. Examples: `IOException`, `SQLException`. These represent conditions a well-written application might reasonably anticipate and recover from.

**Unchecked exceptions** (extend `RuntimeException`) — the compiler does NOT force you to handle them. Examples: `NullPointerException`, `IllegalArgumentException`. These usually represent programming errors or unrecoverable conditions.

In modern Spring Boot applications, the overwhelming convention is: **use unchecked exceptions for your own custom exceptions.** Why? Because checked exceptions force every layer of your call stack to either catch or re-declare them, which creates a lot of noisy boilerplate in a typical layered Spring application (controller → service → repository). Instead, let exceptions propagate up naturally and handle them centrally at the top, using `@ControllerAdvice` (covered below).

## 6.2 Custom Exceptions

Rather than throwing generic exceptions everywhere (`throw new RuntimeException("not found")`), well-designed Spring Boot apps define specific, meaningful custom exception classes:

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

public class DuplicateResourceException extends RuntimeException {
    public DuplicateResourceException(String message) {
        super(message);
    }
}

public class InsufficientStockException extends RuntimeException {
    public InsufficientStockException(String message) {
        super(message);
    }
}
```

Using specific exception types (instead of one generic exception everywhere) makes your global exception handler much cleaner, because you can map each exception TYPE to a specific, appropriate HTTP status code.

```java
public Order findById(Long id) {
    return orderRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Order not found with id: " + id));
}
```

## 6.3 Global Exception Handler

Rather than wrapping every single controller method in a try/catch block (which gets repetitive and error-prone fast), Spring lets you define ONE centralized place that catches exceptions thrown ANYWHERE across your entire application and converts them into proper, consistent HTTP responses.

This is done using `@ControllerAdvice` (or `@RestControllerAdvice`, its REST-focused variant) combined with `@ExceptionHandler` methods.

## 6.4 @ControllerAdvice

`@ControllerAdvice` marks a class as a global handler that applies across ALL controllers in your application (unless you scope it more narrowly). It's essentially "aspect-oriented programming for exceptions" — cross-cutting logic that would otherwise need to be duplicated in every controller.

```java
@RestControllerAdvice  // = @ControllerAdvice + @ResponseBody, returns data directly as JSON
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(HttpStatus.NOT_FOUND.value(), ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ErrorResponse> handleDuplicate(DuplicateResourceException ex) {
        ErrorResponse error = new ErrorResponse(HttpStatus.CONFLICT.value(), ex.getMessage());
        return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        // catch-all safety net for anything unanticipated
        ErrorResponse error = new ErrorResponse(HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "An unexpected error occurred");
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

## 6.5 @ExceptionHandler

`@ExceptionHandler` marks a method as being responsible for handling a specific exception type (or a list of types). When placed inside a `@ControllerAdvice` class, it applies globally. When placed directly inside a regular controller, it applies ONLY to exceptions thrown by that specific controller — useful for handler logic unique to one particular resource.

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @ExceptionHandler(InsufficientStockException.class)
    public ResponseEntity<ErrorResponse> handleStockIssue(InsufficientStockException ex) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
                .body(new ErrorResponse(409, ex.getMessage()));
    }
}
```

Spring resolves which handler to call based on the MOST SPECIFIC exception type match available — a handler for `ResourceNotFoundException` will be preferred over a more generic handler for `RuntimeException`, if both exist and the thrown exception is a `ResourceNotFoundException`.

## 6.6 Problem Details API

Modern Spring (from Spring 6 / Spring Boot 3 onward) has built-in support for **RFC 7807 "Problem Details"** — a standardized JSON format for representing HTTP API errors, so that clients across different APIs and organizations can parse errors consistently.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ProblemDetail handleNotFound(ResourceNotFoundException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        problem.setTitle("Resource Not Found");
        problem.setProperty("timestamp", Instant.now());
        return problem;
    }
}
```

This produces a standardized response body like:

```json
{
  "type": "about:blank",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "Order not found with id: 42",
  "timestamp": "2026-07-28T10:15:30Z"
}
```

## 6.7 Error Response Design

Regardless of whether you use a custom format or Problem Details, a well-designed error response should generally include:

- A machine-readable status code
- A human-readable message describing what went wrong
- A timestamp
- The request path that caused the error (helps with debugging/log correlation)
- Optionally, a list of field-specific validation errors (for validation failures)
- Optionally, a unique trace/correlation ID (extremely helpful when cross-referencing with centralized logs in production)

```java
public class ErrorResponse {
    private int status;
    private String message;
    private Instant timestamp = Instant.now();
    private String path;
    private Map<String, String> fieldErrors;
    // constructors, getters
}
```

Consistency matters more than the exact shape you pick — every error your API returns, across every endpoint, should follow the exact same structure so client applications can handle errors generically instead of writing custom parsing logic per-endpoint.

---
# 7. Spring Data JPA

## 7.1 ORM

ORM stands for **Object-Relational Mapping**. It's the technique of mapping Java objects (classes, fields, relationships) directly onto relational database concepts (tables, columns, foreign keys) — and vice versa. Instead of writing raw SQL everywhere and manually converting `ResultSet` rows into Java objects field by field, an ORM handles that translation for you automatically.

Think of it as a translator standing between two people who speak different languages: your Java code speaks "objects," your database speaks "tables and rows," and the ORM translates seamlessly between them in both directions.

## 7.2 Hibernate

Hibernate is the most widely used ORM implementation in the Java world, and it's the DEFAULT implementation Spring Boot uses under the hood whenever you use Spring Data JPA. When you save a Java object, Hibernate generates and executes the appropriate `INSERT`/`UPDATE` SQL. When you query, Hibernate translates your query into SQL, executes it, and maps the resulting rows back into Java objects.

## 7.3 Entity

An "Entity" is a Java class that represents a table in your database. Each instance of that class represents one row in that table.

```java
@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "customer_name", nullable = false)
    private String customerName;

    @Column(precision = 10, scale = 2)
    private BigDecimal totalAmount;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    @CreationTimestamp
    private LocalDateTime createdAt;

    // getters and setters
}
```

- `@Entity` marks the class as a JPA-managed entity
- `@Table` optionally customizes the table name (otherwise it defaults to the class name)
- `@Id` marks the primary key field
- `@GeneratedValue` specifies how the primary key value is generated (auto-increment, sequence, etc.)
- `@Column` customizes column-level details (name, nullability, precision)

## 7.4 Repository

A "Repository" is the design pattern Spring Data JPA is built around: an abstraction layer that hides the details of data access behind a simple, collection-like interface. Instead of manually writing SQL or JPQL for basic CRUD operations, you just declare an interface, and Spring Data JPA generates the implementation for you AT RUNTIME — no code you write yourself, just an interface declaration.

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    // that's it — full CRUD is already available
}
```

## 7.5 CrudRepository

`CrudRepository<T, ID>` is the base interface providing the fundamental Create, Read, Update, Delete operations:

```java
public interface CrudRepository<T, ID> {
    <S extends T> S save(S entity);
    Optional<T> findById(ID id);
    Iterable<T> findAll();
    long count();
    void deleteById(ID id);
    boolean existsById(ID id);
    // ... and more
}
```

Because `OrderRepository` extends this (indirectly, through `JpaRepository`), you get all these methods completely for free:

```java
Order saved = orderRepository.save(new Order(...));
Optional<Order> found = orderRepository.findById(5L);
List<Order> all = orderRepository.findAll(); // note: JpaRepository returns List, not Iterable
orderRepository.deleteById(5L);
```

## 7.6 JpaRepository

`JpaRepository<T, ID>` extends `CrudRepository` (through an intermediate `PagingAndSortingRepository`) and adds JPA-specific and pagination/sorting capabilities on top:

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
}
```

Extra capabilities you get automatically:

```java
List<Order> orders = orderRepository.findAll(Sort.by("createdAt").descending());
Page<Order> page = orderRepository.findAll(PageRequest.of(0, 20));
orderRepository.flush(); // force pending changes to the database immediately
List<Order> batch = orderRepository.saveAll(listOfOrders);
```

In practice, `JpaRepository` is what you extend for almost every repository in a Spring Boot + JPA application — it's the most feature-complete base interface.

## 7.7 Entity Lifecycle

Every JPA entity instance exists in one of four states at any given time:

1. **Transient** — a plain new object, created with `new Order()`, not yet associated with any persistence context. JPA doesn't know it exists.
2. **Managed (Persistent)** — the entity is currently tracked by the persistence context (the `EntityManager`). Any changes made to it will automatically be detected and saved to the database when the transaction commits (this is "dirty checking," covered below).
3. **Detached** — the entity WAS managed at some point, but the persistence context that tracked it has since closed (e.g., the transaction ended). Changes to a detached entity are NOT automatically saved.
4. **Removed** — the entity is marked for deletion; it will be deleted from the database when the transaction commits.

```java
Order order = new Order();       // TRANSIENT
orderRepository.save(order);     // now MANAGED
// ... transaction commits, EntityManager closes ...
// order is now DETACHED
orderRepository.delete(order);   // now REMOVED (pending commit)
```

## 7.8 Persistence Context

The **Persistence Context** is essentially a first-level cache maintained by the `EntityManager` for the current transaction. It keeps track of every managed entity, ensuring:

- Each entity is represented by exactly ONE Java object within a single persistence context, even if you query for it multiple times (this is called "identity map" behavior)
- Any changes to managed entities are automatically detected and flushed to the database at the right time
- Repeated queries for the same entity within the same transaction can be served from this in-memory cache instead of hitting the database again

Think of the persistence context as a "scratchpad" that exists for the duration of one transaction — once the transaction ends, the scratchpad is thrown away, and entities become detached.

## 7.9 Dirty Checking

"Dirty checking" is one of Hibernate's most powerful (and sometimes surprising, if you don't know about it) features. Within an active transaction, if you modify a managed entity's fields, Hibernate AUTOMATICALLY detects the change and generates the appropriate `UPDATE` SQL when the transaction commits — **you never have to explicitly call `save()` again** for an entity that's already managed.

```java
@Transactional
public void updateOrderStatus(Long orderId, OrderStatus newStatus) {
    Order order = orderRepository.findById(orderId).orElseThrow();
    order.setStatus(newStatus); // no explicit save() call needed!
    // Hibernate detects this change and issues an UPDATE automatically
    // when the transaction commits
}
```

This is convenient, but it's also a common source of confusion for developers new to JPA — they expect to need an explicit `save()` call for updates, the way you would with plain JDBC.

## 7.10 Transactions

A transaction is a group of database operations that must ALL succeed together, or ALL fail together (leaving the database unchanged) — this is the "atomicity" part of the classic ACID properties. Spring makes transaction management declarative through the `@Transactional` annotation.

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder(OrderRequest request) {
        Order order = createOrder(request);
        orderRepository.save(order);
        inventoryService.reduceStock(request.getItems()); // if this throws, the order save above is rolled back too
        paymentService.charge(request.getPaymentDetails());
    }
}
```

If ANY exception is thrown inside a `@Transactional` method (by default, only unchecked exceptions trigger rollback), Spring automatically rolls back EVERYTHING done within that method — so you never end up with an order saved but payment never charged, for example.

Key transaction concepts:
- **Propagation** — controls how transactions behave when one transactional method calls another (e.g., `REQUIRED` joins an existing transaction if one exists, `REQUIRES_NEW` always starts a fresh one)
- **Isolation** — controls how much one transaction can "see" of another transaction's in-progress changes (`READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`, etc.)
- **Rollback rules** — you can configure exactly which exception types should (or shouldn't) trigger a rollback

```java
@Transactional(rollbackFor = Exception.class, isolation = Isolation.READ_COMMITTED)
public void criticalOperation() { }
```

## 7.11 JPQL

JPQL (Jakarta Persistence Query Language) looks a lot like SQL, but it operates on your ENTITY model (classes and fields) instead of raw database tables and columns. This keeps your queries portable across different databases, since JPQL gets translated into the appropriate dialect of SQL at runtime.

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    @Query("SELECT o FROM Order o WHERE o.status = :status AND o.totalAmount > :minAmount")
    List<Order> findHighValueOrdersByStatus(@Param("status") OrderStatus status,
                                             @Param("minAmount") BigDecimal minAmount);
}
```

Notice: `Order` and `o.status` refer to the ENTITY CLASS and its FIELDS, not the actual `orders` table and `status` column names — that translation is Hibernate's job.

## 7.12 Native Queries

Sometimes JPQL isn't enough — maybe you need a database-specific feature, or a complex query that's much easier to express in raw SQL. Native queries let you drop down to actual SQL when needed.

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    @Query(value = "SELECT * FROM orders WHERE created_at > NOW() - INTERVAL '7 days'", nativeQuery = true)
    List<Order> findRecentOrders();
}
```

The tradeoff: native queries are tied to your specific database's SQL dialect, so they're less portable if you ever need to switch database vendors. Use them sparingly, only when JPQL genuinely can't express what you need.

## 7.13 Derived Queries

One of Spring Data JPA's best party tricks: you can define a query simply by naming your repository method in a specific pattern, and Spring Data will parse the method name and generate the correct query automatically — **no `@Query` annotation, no SQL, no JPQL needed at all.**

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    List<Order> findByStatus(OrderStatus status);

    List<Order> findByCustomerNameAndStatus(String customerName, OrderStatus status);

    List<Order> findByTotalAmountGreaterThan(BigDecimal amount);

    List<Order> findByCreatedAtBetween(LocalDateTime start, LocalDateTime end);

    boolean existsByCustomerEmail(String email);

    long countByStatus(OrderStatus status);
}
```

Spring parses `findByCustomerNameAndStatus` as: find where `customerName` equals X AND `status` equals Y. Keywords like `And`, `Or`, `Between`, `GreaterThan`, `Like`, `OrderBy`, `IgnoreCase` are all supported, letting you express fairly rich queries with zero actual query code.

## 7.14 Pagination

For any endpoint that could return a huge list of results, returning EVERYTHING at once is a performance and usability disaster. Pagination breaks results into manageable chunks ("pages").

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    Page<Order> findByStatus(OrderStatus status, Pageable pageable);
}
```

```java
@GetMapping
public Page<Order> getOrders(@RequestParam OrderStatus status,
                              @RequestParam(defaultValue = "0") int page,
                              @RequestParam(defaultValue = "20") int size) {
    return orderRepository.findByStatus(status, PageRequest.of(page, size));
}
```

The `Page<T>` object returned includes not just the data, but also metadata: total number of elements, total number of pages, whether this is the first/last page, and so on — everything a frontend needs to build pagination controls.

## 7.15 Sorting

Sorting can be applied independently, or combined with pagination:

```java
List<Order> orders = orderRepository.findAll(Sort.by("createdAt").descending());

Page<Order> page = orderRepository.findAll(
    PageRequest.of(0, 20, Sort.by("totalAmount").descending().and(Sort.by("customerName").ascending()))
);
```

You can also expose sorting dynamically from an API, letting clients specify how they want results ordered:

```java
@GetMapping
public Page<Order> getOrders(Pageable pageable) {
    // clients call: GET /api/orders?page=0&size=20&sort=totalAmount,desc
    return orderRepository.findAll(pageable);
}
```

## 7.16 Specifications

When your filtering requirements become genuinely dynamic — you don't know in advance which combination of filters a client will send — derived query methods and static `@Query` annotations become impractical (you'd need a method for every possible combination). **Specifications** let you build queries programmatically, combining conditions dynamically at runtime.

```java
public class OrderSpecifications {

    public static Specification<Order> hasStatus(OrderStatus status) {
        return (root, query, cb) -> status == null ? null : cb.equal(root.get("status"), status);
    }

    public static Specification<Order> hasMinAmount(BigDecimal minAmount) {
        return (root, query, cb) -> minAmount == null ? null : cb.greaterThanOrEqualTo(root.get("totalAmount"), minAmount);
    }
}
```

```java
public interface OrderRepository extends JpaRepository<Order, Long>, JpaSpecificationExecutor<Order> {
}
```

```java
Specification<Order> spec = Specification
        .where(OrderSpecifications.hasStatus(status))
        .and(OrderSpecifications.hasMinAmount(minAmount));

List<Order> results = orderRepository.findAll(spec);
```

This lets you compose filters like building blocks — combining only the conditions that were actually provided, and ignoring the rest, without writing a giant if/else chain of custom queries.

## 7.17 Entity Graph

By default, JPA relationships are often lazily loaded (covered in depth in the next section), meaning related data isn't fetched until you actually access it — which can lead to extra, unplanned queries. `@EntityGraph` lets you specify, on a per-query basis, exactly which related entities should be eagerly fetched together in ONE query, without changing the entity's default fetch strategy globally.

```java
public interface OrderRepository extends JpaRepository<Order, Long> {

    @EntityGraph(attributePaths = {"items", "customer"})
    List<Order> findByStatus(OrderStatus status);
}
```

This query will fetch `Order` records AND eagerly load their `items` and `customer` associations in the same round trip — helping avoid the dreaded "N+1 query problem" (covered in detail in the next section) for this specific query, while keeping other queries on the same entity lazy by default.

---
# 8. Hibernate Advanced

## 8.1 Lazy Loading

"Lazy loading" means: a related entity/collection is NOT fetched from the database until the moment you actually access it in code. This is the default behavior for most relationship types in JPA (except `@ManyToOne` and `@OneToOne`, which default to eager — more on this below).

```java
@Entity
public class Order {
    @Id
    private Long id;

    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> items;
}
```

When you load an `Order`, Hibernate does NOT fetch `items` right away. Only when you call `order.getItems()` does Hibernate go back to the database and fetch them. This is efficient — you don't pay the cost of loading data you never end up using.

The catch: if you try to access a lazy collection AFTER the persistence context/session has already closed (e.g., outside a `@Transactional` method), you'll hit the infamous `LazyInitializationException`. This is one of the most common runtime errors for developers new to JPA.

## 8.2 Eager Loading

"Eager loading" means: the related entity/collection IS fetched immediately, as part of the same query that loads the parent entity (or a follow-up query fired right away).

```java
@Entity
public class OrderItem {
    @ManyToOne(fetch = FetchType.EAGER)  // this is actually the DEFAULT for @ManyToOne
    private Order order;
}
```

Eager loading avoids `LazyInitializationException` risk, but at a cost: you ALWAYS pay the price of loading that related data, even in scenarios where you never actually need it. Overusing eager loading, especially on collections, is one of the most common causes of unnecessary performance problems in Hibernate applications.

**General rule of thumb:** default to `LAZY` almost everywhere, and use `@EntityGraph` or JPQL `JOIN FETCH` on a per-query basis when you specifically need related data eagerly for that particular use case.

## 8.3 Fetch Types

To summarize the defaults, since they trip up a lot of developers:

| Relationship | Default Fetch Type |
|---|---|
| `@OneToOne` | EAGER |
| `@ManyToOne` | EAGER |
| `@OneToMany` | LAZY |
| `@ManyToMany` | LAZY |

Many experienced teams explicitly override `@OneToOne` and `@ManyToOne` to `LAZY` as well, since the "eager by default" behavior for these can silently cause performance issues as an object graph grows — better to be explicit and load only what you need, when you need it.

## 8.4 Cascade Types

Cascading defines how operations performed on a parent entity should "cascade" (propagate) to its related child entities. Without cascading, you'd have to manually save/delete every related entity yourself.

```java
@Entity
public class Order {
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}
```

Common cascade types:

| Type | Meaning |
|---|---|
| `PERSIST` | Saving the parent also saves new children |
| `MERGE` | Updating the parent also updates children |
| `REMOVE` | Deleting the parent also deletes children |
| `REFRESH` | Refreshing the parent also refreshes children from DB |
| `ALL` | All of the above combined |

`orphanRemoval = true` is a related but distinct concept — it means: if a child is REMOVED from the parent's collection (even without deleting the parent itself), that child gets deleted from the database too. Very useful for genuine parent-owns-child relationships, like an order owning its line items.

```java
order.getItems().remove(someItem); // with orphanRemoval=true, this deletes someItem from the DB on flush
```

## 8.5 Entity Relationships

JPA supports mapping all the classic relational database relationship types directly onto your object model.

## 8.6 OneToOne

Each entity instance is related to exactly one instance of another entity.

```java
@Entity
public class User {
    @Id
    private Long id;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
}
```

Example: a `User` has exactly one `UserProfile` containing extended details (bio, avatar, preferences) — separated into its own table for organizational or performance reasons, but conceptually a 1:1 pairing.

## 8.7 OneToMany

One entity instance is related to MANY instances of another entity. This is almost always paired with a `@ManyToOne` on the other side (called a "bidirectional" relationship).

```java
@Entity
public class Order {
    @OneToMany(mappedBy = "order")
    private List<OrderItem> items;
}

@Entity
public class OrderItem {
    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;
}
```

`mappedBy = "order"` on the `Order` side tells Hibernate: "the foreign key is actually managed by the `order` field over on the `OrderItem` side — I'm just the inverse (read-only) side of this relationship." This is important: the side WITHOUT `mappedBy` (the `@ManyToOne` side here) is the "owning side," and it's the one whose changes actually get written to the foreign key column.

## 8.8 ManyToOne

The inverse of `OneToMany` — MANY instances of this entity relate to ONE instance of another. This is the "owning side" of the relationship by JPA convention, and it's where the actual foreign key column lives in the database.

```java
@Entity
public class OrderItem {
    @ManyToOne
    @JoinColumn(name = "order_id", nullable = false)
    private Order order;
}
```

## 8.9 ManyToMany

Many instances of one entity relate to many instances of another — this requires a "join table" under the hood (a separate table containing pairs of foreign keys), since a plain foreign-key column can't represent a many-to-many relationship directly.

```java
@Entity
public class Student {
    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}

@Entity
public class Course {
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}
```

In modern practice, if the join table needs to hold EXTRA data beyond just the two foreign keys (like an `enrollmentDate`), you typically avoid `@ManyToMany` altogether and instead model the join table as its own explicit entity with two `@ManyToOne` relationships — giving you a place to put that extra data.

## 8.10 Composite Keys

Sometimes a table's primary key isn't a single column, but a COMBINATION of columns. JPA supports this through `@EmbeddedId` or `@IdClass`.

```java
@Embeddable
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;
    // equals() and hashCode() are REQUIRED for composite keys to work correctly
}

@Entity
public class OrderItem {
    @EmbeddedId
    private OrderItemId id;
}
```

Composite keys add complexity (you always have to construct the key object to look anything up), so many teams prefer a simple surrogate key (a single auto-generated `id` column) even on join/junction tables, reserving composite keys for cases where the natural key genuinely has no reasonable single-column alternative.

## 8.11 Embeddables

`@Embeddable` lets you group a set of related fields into a reusable value object, embedded directly into an entity's table (NOT a separate table with a foreign key — the columns live right alongside the entity's own columns).

```java
@Embeddable
public class Address {
    private String street;
    private String city;
    private String postalCode;
    private String country;
}

@Entity
public class Customer {
    @Id
    private Long id;

    @Embedded
    private Address shippingAddress;
}
```

This produces a `customer` table with columns like `street`, `city`, `postal_code`, `country` directly on it — no separate `address` table, no join. It's purely an organizational tool on the Java side for grouping related fields into a reusable, meaningful type.

## 8.12 Optimistic Locking

When multiple users might update the same record concurrently, you need a strategy to prevent one user's changes from silently overwriting another's ("lost update" problem). **Optimistic locking** assumes conflicts are rare, and simply detects them when they happen (rather than physically locking rows in the database ahead of time).

```java
@Entity
public class Order {
    @Id
    private Long id;

    @Version
    private Long version;
}
```

Every time a row is updated, Hibernate checks that the `version` column still matches what it was when the entity was loaded, and increments it. If another transaction updated the row in the meantime (changing the version), Hibernate throws `OptimisticLockException` — letting your application detect and handle the conflict (usually by asking the user to retry).

## 8.13 Pessimistic Locking

**Pessimistic locking** takes the opposite approach: it assumes conflicts ARE likely, and proactively locks the row in the database the moment it's read, preventing any other transaction from modifying (or sometimes even reading) it until the lock is released.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT o FROM Order o WHERE o.id = :id")
Optional<Order> findByIdForUpdate(@Param("id") Long id);
```

This maps to a `SELECT ... FOR UPDATE` statement under the hood. Pessimistic locking is appropriate for high-contention scenarios (like reducing inventory stock, where many concurrent requests genuinely will conflict), but it hurts throughput because it forces requests to literally wait in line — so it should be used sparingly and only where actually justified.

## 8.14 N+1 Problem

This is probably THE most infamous Hibernate performance pitfall, and every Spring Boot developer eventually runs into it. Here's what happens:

You query for a list of N orders (1 query). Then, for EACH order, you access a lazy-loaded collection or relationship (like `order.getCustomer().getName()`) — and because it's lazy, Hibernate fires off a SEPARATE query to fetch it, for every single order. End result: 1 query to get the orders, PLUS N additional queries — hence "N+1."

```java
List<Order> orders = orderRepository.findAll(); // 1 query
for (Order order : orders) {
    System.out.println(order.getCustomer().getName()); // N additional queries!
}
```

With 1,000 orders, that's 1,001 total database round trips for something that should have been achievable in 1 or 2. This can silently devastate performance in production.

**Solutions:**

- Use `JOIN FETCH` in JPQL to eagerly load the association in the SAME query:
```java
@Query("SELECT o FROM Order o JOIN FETCH o.customer")
List<Order> findAllWithCustomer();
```
- Use `@EntityGraph` (covered earlier) for a declarative, per-query fetch strategy
- Use batch fetching (`@BatchSize`) to at least group the extra queries into batches instead of one-per-entity

## 8.15 Caching

Hibernate supports multiple levels of caching to reduce redundant database trips:

- **First-level cache** — this is just the Persistence Context itself (covered earlier); it's automatic, per-transaction, and can't be disabled.
- **Second-level cache** — an OPTIONAL cache shared ACROSS sessions/transactions, typically backed by an external cache provider like Ehcache or Redis. Must be explicitly enabled and configured.
- **Query cache** — caches the RESULTS of specific queries (works alongside the second-level cache).

```java
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Product {
    // frequently-read, rarely-changed data is a great candidate for second-level caching
}
```

Second-level caching is powerful but adds real complexity (cache invalidation, staleness risk), so it's typically reserved for data that's read very frequently but changes rarely — like product catalogs or configuration data.

## 8.16 Batch Processing

When inserting or updating large numbers of records, sending one SQL statement per record is very inefficient — each one is a separate round trip to the database. Hibernate supports batching, where multiple statements are grouped together and sent to the database in one network round trip.

```properties
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
```

```java
@Transactional
public void bulkInsert(List<Product> products) {
    for (int i = 0; i < products.size(); i++) {
        entityManager.persist(products.get(i));
        if (i % 50 == 0) {
            entityManager.flush();  // send the batch to the DB
            entityManager.clear();  // free memory by detaching already-flushed entities
        }
    }
}
```

This dramatically reduces the number of database round trips when inserting/updating thousands of records at once — a common requirement for data imports, migrations, or bulk operations.

---

# 9. Database Migration

## 9.1 Flyway

As your application evolves, your database schema needs to evolve right along with it — new tables, new columns, new indexes. **Flyway** is a database migration tool that manages this evolution through plain, version-numbered SQL scripts, applied in order, with Flyway keeping track of exactly which migrations have already been run.

```
src/main/resources/db/migration/
    V1__create_users_table.sql
    V2__add_email_column_to_users.sql
    V3__create_orders_table.sql
```

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

Spring Boot auto-configures Flyway the moment it's on the classpath — migrations run automatically every time the application starts up, applying only the migrations that haven't already been applied (Flyway tracks this in a special `flyway_schema_history` table it creates for itself).

```sql
-- V1__create_users_table.sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 9.2 Liquibase

**Liquibase** solves the exact same problem as Flyway — tracked, versioned, repeatable database migrations — but takes a different approach to authoring: instead of raw SQL files, migrations ("changesets") can be written in XML, YAML, JSON, or SQL, giving you database-agnostic abstractions if you need to support multiple database vendors from the same changelog.

```xml
<changeSet id="1" author="dev">
    <createTable tableName="users">
        <column name="id" type="BIGINT" autoIncrement="true">
            <constraints primaryKey="true"/>
        </column>
        <column name="username" type="VARCHAR(50)">
            <constraints nullable="false" unique="true"/>
        </column>
    </createTable>
</changeSet>
```

**Flyway vs Liquibase, in a nutshell:** Flyway is simpler and SQL-first, favored when your team is comfortable writing raw SQL and doesn't need multi-database abstraction. Liquibase offers more powerful abstractions (like automatic rollback generation) at the cost of a steeper learning curve. Both are excellent, well-supported choices — most teams pick based on preference for SQL-first vs abstracted authoring.

## 9.3 Versioned Migrations

Both tools follow the same core principle: EVERY change to the schema is captured as a small, immutable, sequentially-numbered script. Once a migration has been applied to any environment (especially production), it should NEVER be edited — if you need to change something, you write a NEW migration that alters what the previous one did.

This gives you:
- A complete, auditable history of every schema change ever made
- The ability to spin up a brand-new environment (or database) and apply the FULL migration history to arrive at the exact current schema
- Confidence that dev, staging, and production databases are always structurally identical (since they all ran the exact same migration scripts, in the exact same order)

## 9.4 Rollback Strategy

Sometimes a migration needs to be undone — maybe a deploy gets rolled back, or a migration turns out to have a bug. Both tools support rollback, though with different philosophies:

**Flyway** (open-source edition) does NOT support automatic rollback out of the box — the philosophy is "roll forward," meaning you fix a problem by writing a NEW migration that corrects it, rather than reversing a previous one. (Flyway's paid "Teams" edition does add undo migrations.)

**Liquibase** supports rollback more natively — many changeset types can auto-generate their own rollback logic, and you can also write explicit custom rollback instructions:

```xml
<changeSet id="2" author="dev">
    <addColumn tableName="users">
        <column name="phone_number" type="VARCHAR(20)"/>
    </addColumn>
    <rollback>
        <dropColumn tableName="users" columnName="phone_number"/>
    </rollback>
</changeSet>
```

In practice, most experienced teams treat "roll forward" as the safer default strategy for PRODUCTION databases regardless of tooling — reversing a migration that already ran against real production data (which may already reflect the new schema) is often riskier than just shipping a corrective fix forward.

---
# 10. Spring Security

## 10.1 Spring Security Basics

Spring Security is the standard framework for handling **authentication** (who are you?) and **authorization** (what are you allowed to do?) in Spring applications. It's extremely powerful, but also has a reputation for being confusing at first — mostly because it works through a chain of filters intercepting every request, which isn't obvious until you understand the model.

The moment you add `spring-boot-starter-security` to your project, Spring Boot automatically locks down EVERY endpoint by default, requiring authentication (with a randomly generated password printed in your console logs at startup) — this is a deliberate "secure by default" design choice.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

## 10.2 Authentication

Authentication answers the question: **"Who is making this request?"** Spring Security supports many authentication mechanisms — username/password forms, HTTP Basic auth, JWT tokens, OAuth2/OpenID Connect, and more.

The core abstraction is the `Authentication` object, which represents the currently authenticated principal (user) and their granted authorities (roles/permissions), stored in the `SecurityContext` for the duration of the request.

```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
String username = auth.getName();
Collection<? extends GrantedAuthority> authorities = auth.getAuthorities();
```

## 10.3 Authorization

Authorization answers a DIFFERENT question: **"Now that we know who you are, are you allowed to do this specific thing?"** This happens AFTER authentication succeeds.

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/api/public/**").permitAll()
        .requestMatchers("/api/admin/**").hasRole("ADMIN")
        .requestMatchers(HttpMethod.DELETE, "/api/orders/**").hasRole("ADMIN")
        .anyRequest().authenticated()
    );
    return http.build();
}
```

This example says: public endpoints need no authentication at all, admin endpoints need the `ADMIN` role specifically, only admins can DELETE orders, and everything else just needs SOME valid authentication (any role).

## 10.4 Password Encoding

Storing passwords in plain text is one of the most catastrophic security mistakes possible — if your database is ever breached, every single user's real password is exposed immediately. Instead, passwords must always be **hashed** using a one-way cryptographic function before storage.

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

```java
String hashedPassword = passwordEncoder.encode(rawPassword);
userRepository.save(new User(username, hashedPassword));

// later, to check a login attempt:
boolean matches = passwordEncoder.matches(rawPasswordFromLoginForm, storedHashedPassword);
```

Crucially, you never "decrypt" a hashed password to check it — you hash the newly-submitted password using the same algorithm and compare the two hashes. This means even the application itself never knows a user's actual plaintext password once it's been hashed.

## 10.5 BCrypt

BCrypt is the specific hashing algorithm most commonly used for password storage in Spring applications (and the web in general), and it's Spring Security's default `PasswordEncoder` implementation. What makes BCrypt particularly well-suited for passwords (versus a generic hash like SHA-256):

- **It's deliberately slow** — computationally expensive by design, which makes brute-force attacks (trying millions of password guesses) impractically slow for an attacker
- **It automatically incorporates a random "salt"** into each hash, so two users with the identical password end up with completely different stored hashes — defeating precomputed "rainbow table" attacks
- **It's adaptive** — you can increase the "cost factor" over time as hardware gets faster, keeping it slow enough to resist brute-force attacks even years later

```java
new BCryptPasswordEncoder(12); // cost factor of 12 (higher = slower = more secure, but slower for legitimate logins too)
```

## 10.6 UserDetailsService

`UserDetailsService` is the interface Spring Security uses to load user-specific data (username, hashed password, roles/authorities, account status) during authentication. You implement this to plug in YOUR OWN user storage (database, LDAP, wherever) into Spring Security's authentication process.

```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found"));

        return org.springframework.security.core.userdetails.User
                .withUsername(user.getUsername())
                .password(user.getPasswordHash())
                .authorities(user.getRoles().toArray(new String[0]))
                .build();
    }
}
```

Spring Security calls this automatically during login, comparing the submitted password (after hashing) against what `loadUserByUsername` returns.

## 10.7 Security Filter Chain

Spring Security's core architecture is built around a CHAIN of servlet filters, each responsible for one specific piece of the security process (extracting credentials, checking CSRF tokens, enforcing authorization rules, handling exceptions, etc.). Every incoming request passes through this entire chain, in order, before it ever reaches your actual controller.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

This is the modern (Spring Security 6+) way to configure security — a fluent, lambda-based DSL replacing the older, more verbose `WebSecurityConfigurerAdapter` class-extension approach (which is now deprecated/removed).

## 10.8 Roles

Roles represent broad categories of users, conventionally prefixed with `ROLE_` internally (though the `hasRole()` helper adds this prefix for you automatically, so you write just the plain name).

```java
.requestMatchers("/api/admin/**").hasRole("ADMIN")
.requestMatchers("/api/orders/**").hasAnyRole("USER", "ADMIN")
```

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long userId) {
    // only reachable if the authenticated user has the ADMIN role
}
```

## 10.9 Authorities

"Authorities" is the more general, granular concept underlying roles — a `GrantedAuthority` can represent a role (`ROLE_ADMIN`) OR a specific fine-grained permission (`orders:delete`, `reports:view`). Roles are really just authorities following the `ROLE_` naming convention.

```java
.requestMatchers("/api/orders/*/refund").hasAuthority("orders:refund")
```

Using fine-grained authorities instead of (or alongside) broad roles gives you much more precise control — for example, distinguishing "can view reports" from "can delete users," instead of lumping every admin capability under one single `ADMIN` role.

## 10.10 Method Security

Instead of (or in addition to) defining authorization rules centrally in the `SecurityFilterChain`, you can annotate individual SERVICE methods directly, which is often clearer for business-logic-specific rules.

```java
@Configuration
@EnableMethodSecurity
public class MethodSecurityConfig { }
```

```java
@Service
public class OrderService {

    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public Order getOrder(Long orderId, Long userId) {
        // admins can view any order; regular users can only view their own
    }

    @PostAuthorize("returnObject.customerId == authentication.principal.id")
    public Order findById(Long orderId) {
        // checks the rule AFTER the method runs, against the returned object
    }
}
```

`@PreAuthorize` checks the rule BEFORE the method executes (can reference method parameters); `@PostAuthorize` checks AFTER the method returns (can reference the return value).

## 10.11 CORS

CORS (Cross-Origin Resource Sharing) is a browser security mechanism that blocks a web page from making requests to a DIFFERENT domain than the one that served the page — unless the target server explicitly allows it. This matters constantly in modern apps where your frontend (e.g., `https://myapp.com`) and backend API (e.g., `https://api.myapp.com`) live on different origins.

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("https://myapp.com"));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
    config.setAllowedHeaders(List.of("*"));
    config.setAllowCredentials(true);

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);
    return source;
}
```

```java
http.cors(cors -> cors.configurationSource(corsConfigurationSource()));
```

Without this configuration, your frontend's browser would silently block API responses even if the server processed the request successfully — a very common source of confusion for developers new to full-stack development ("my API works in Postman but not from my React app!").

## 10.12 CSRF

CSRF (Cross-Site Request Forgery) is an attack where a malicious website tricks a logged-in user's browser into submitting an unwanted request to another site where they're authenticated (e.g., a hidden form that submits "transfer money" using the victim's existing session cookie).

Spring Security protects against this by default, for session/cookie-based authentication, by requiring a special CSRF token to be included with any state-changing request.

```java
http.csrf(csrf -> csrf
    .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
);
```

**Important nuance:** CSRF protection matters for cookie-based session authentication (where the browser automatically attaches credentials to every request). For **stateless, token-based APIs** (like JWT auth, where the token must be explicitly attached by the client in a header, not automatically sent by the browser), CSRF isn't really a relevant threat — which is why it's commonly disabled entirely for pure REST APIs using JWT:

```java
http.csrf(csrf -> csrf.disable());
```

## 10.13 Session Authentication

Traditional, "session-based" authentication works like this: the user logs in, the server creates a session and stores it (in memory, or a shared store like Redis), and sends the client a session ID via a cookie. On every subsequent request, the browser automatically sends that cookie back, and the server looks up the session to know who's making the request.

```properties
server.servlet.session.timeout=30m
```

This approach is simple and works great for traditional server-rendered web applications. However, it requires the server to maintain state for every logged-in user (unless using a distributed session store), which can complicate horizontal scaling.

## 10.14 Stateless Authentication

For modern REST APIs (and especially microservices), **stateless authentication** is much more common: the server doesn't store ANY session state at all. Instead, every request carries all the information needed to authenticate it — typically a signed token (like a JWT, covered in depth next) sent in the `Authorization` header.

```java
http.sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
```

This scales beautifully across multiple server instances (any instance can validate any request independently, no shared session store needed), which is exactly why it's the dominant approach for modern distributed/microservices architectures.

---
# 11. JWT Authentication

## 11.1 JWT Structure

JWT stands for **JSON Web Token**. It's a compact, self-contained way of transmitting information (usually about a user's identity and permissions) between two parties, in a way that can be cryptographically verified. A JWT looks like a string with three parts separated by dots:

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJqb2huIiwicm9sZSI6IkFETUlOIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

The three parts, base64-encoded, are:

1. **Header** — metadata, like the signing algorithm used (e.g., `{"alg": "HS256"}`)
2. **Payload** — the actual "claims" (data) — user ID, roles, expiration time, etc. `{"sub": "john", "role": "ADMIN", "exp": 1735689600}`
3. **Signature** — a cryptographic signature over the header + payload, generated using a secret key, which lets the server verify the token hasn't been tampered with

Crucially, the payload is only BASE64-ENCODED, not encrypted — anyone can decode and read it. The signature is what guarantees the CONTENT hasn't been altered; it does NOT hide the content. Never put sensitive secrets (like passwords) inside a JWT payload.

## 11.2 Access Token

The **access token** is the JWT the client includes on every API request to prove who they are, typically in the `Authorization` header:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

Access tokens are deliberately kept **short-lived** (typically 15 minutes to a few hours) — this limits the damage if a token is ever stolen, since it'll expire soon regardless. Because it's stateless and self-contained, the server can validate it (checking the signature and expiration) without needing to query a database or session store on every single request — which is a big part of why JWT-based auth scales so well.

## 11.3 Refresh Token

Since access tokens expire quickly, you don't want to force users to log in again every 15 minutes. The **refresh token** solves this: it's a separate, longer-lived token (days or weeks) whose ONLY job is to be exchanged for a new access token when the old one expires.

```java
@PostMapping("/refresh")
public ResponseEntity<TokenResponse> refresh(@RequestBody RefreshRequest request) {
    if (!refreshTokenService.isValid(request.getRefreshToken())) {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }
    String username = refreshTokenService.getUsername(request.getRefreshToken());
    String newAccessToken = jwtService.generateAccessToken(username);
    return ResponseEntity.ok(new TokenResponse(newAccessToken, request.getRefreshToken()));
}
```

Unlike access tokens, refresh tokens are usually STORED server-side (in a database or Redis), because you need the ability to REVOKE them individually — for example, when a user logs out or if a token is suspected stolen. This is the key tradeoff: access tokens are stateless and short-lived; refresh tokens are stateful (revocable) and long-lived.

## 11.4 Token Validation

Every time a request arrives with a JWT, the server must validate it before trusting anything inside it:

```java
public boolean validateToken(String token) {
    try {
        Jwts.parserBuilder()
            .setSigningKey(secretKey)
            .build()
            .parseClaimsJws(token);
        return true;
    } catch (ExpiredJwtException | MalformedJwtException | SignatureException e) {
        return false;
    }
}
```

Validation checks:
1. **Signature verification** — was this token actually signed by us, using our secret key? (detects tampering)
2. **Expiration check** — has the `exp` claim already passed?
3. Optionally, **issuer/audience checks** — was this token issued by the expected authority, for the expected recipient?

If any of these checks fail, the request must be rejected — typically with a 401 Unauthorized response.

## 11.5 JWT Filter

To integrate JWT validation into Spring Security's filter chain, you write a custom filter that runs BEFORE Spring Security's normal authentication filter, extracts the token, validates it, and — if valid — manually populates the `SecurityContext` so the rest of the request is treated as authenticated.

```java
public class JwtAuthFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                     FilterChain filterChain) throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            String token = authHeader.substring(7);
            if (jwtService.validateToken(token)) {
                String username = jwtService.extractUsername(token);
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        filterChain.doFilter(request, response);
    }
}
```

Registered in the security config:

```java
http.addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);
```

`OncePerRequestFilter` guarantees this filter runs exactly once per request, regardless of how many times the request gets internally forwarded/included — important for avoiding duplicate processing.

## 11.6 Custom Authentication

For login endpoints specifically, you typically build a custom flow: accept username/password, verify them against the `AuthenticationManager`, and if valid, issue a fresh pair of access + refresh tokens.

```java
@PostMapping("/login")
public ResponseEntity<TokenResponse> login(@RequestBody LoginRequest request) {
    Authentication auth = authenticationManager.authenticate(
        new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword())
    );

    String accessToken = jwtService.generateAccessToken(auth.getName());
    String refreshToken = refreshTokenService.createRefreshToken(auth.getName());

    return ResponseEntity.ok(new TokenResponse(accessToken, refreshToken));
}
```

If the credentials are wrong, `authenticationManager.authenticate()` throws a `BadCredentialsException` automatically — which you'd catch globally (via `@ExceptionHandler`) and convert into a clean 401 response.

## 11.7 Token Expiration

Choosing expiration times is a balancing act between security and user experience:

- **Too short:** users get logged out constantly, frustrating experience, more refresh calls
- **Too long:** a stolen token remains dangerous for longer

Common real-world defaults: access tokens expire in 15 minutes to 1 hour; refresh tokens expire in 7 to 30 days. Setting the expiration when generating the token:

```java
public String generateAccessToken(String username) {
    return Jwts.builder()
        .setSubject(username)
        .setIssuedAt(new Date())
        .setExpiration(new Date(System.currentTimeMillis() + 15 * 60 * 1000)) // 15 minutes
        .signWith(secretKey, SignatureAlgorithm.HS256)
        .compact();
}
```

## 11.8 Logout Strategy

Here's a subtlety that trips up a lot of developers: because JWTs are stateless and self-validating, the SERVER can't just "delete" an access token the way it could delete a server-side session — the token remains cryptographically valid until it naturally expires, even after "logout."

Common strategies to handle this:

1. **Client-side deletion** — simplest approach: the client just deletes the token from wherever it stored it (memory, local storage). Works for the "user clicked logout on their own device" case, but doesn't protect against a token that was already stolen.
2. **Refresh token revocation** — since refresh tokens ARE stored server-side, you can revoke/delete them on logout — this at least prevents the user's session from being renewed once the short-lived access token expires naturally.
3. **Token blocklist/denylist** — maintain a server-side store (often Redis, for fast lookups) of explicitly-revoked access tokens, checked on every request. This reintroduces some server-side state, sacrificing some of JWT's "fully stateless" benefit, but gives you true immediate revocation.

Most real-world systems use a combination: short-lived access tokens (limiting the exposure window naturally) plus refresh token revocation on logout, and reserve a full blocklist for high-security scenarios where immediate revocation genuinely matters.

---
# 12. File Upload

## 12.1 Multipart File

File uploads from a browser or API client use a special HTTP content type called `multipart/form-data`, which allows binary file data to be sent alongside regular form fields in a single request. Spring MVC represents each uploaded file as a `MultipartFile` object.

```java
@PostMapping("/upload")
public ResponseEntity<String> uploadFile(@RequestParam("file") MultipartFile file) {
    if (file.isEmpty()) {
        return ResponseEntity.badRequest().body("File is empty");
    }
    String filename = file.getOriginalFilename();
    long size = file.getSize();
    String contentType = file.getContentType();
    // process the file...
    return ResponseEntity.ok("Uploaded: " + filename);
}
```

Spring Boot auto-configures multipart handling out of the box. You can tune limits in configuration:

```properties
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=50MB
```

## 12.2 File Storage

Once you've received a `MultipartFile`, you need to actually store it somewhere. A simple local disk storage implementation:

```java
@Service
public class FileStorageService {

    private final Path storageLocation = Paths.get("uploads");

    public String store(MultipartFile file) throws IOException {
        String filename = UUID.randomUUID() + "_" + file.getOriginalFilename();
        Path targetPath = storageLocation.resolve(filename);
        Files.copy(file.getInputStream(), targetPath, StandardCopyOption.REPLACE_EXISTING);
        return filename;
    }
}
```

Notice the use of a random UUID prefix — this prevents filename collisions (two users uploading a file named `photo.jpg` shouldn't overwrite each other) and avoids trusting user-supplied filenames directly (a security consideration, since filenames could contain path traversal characters like `../`).

For real production systems, local disk storage doesn't scale well across multiple server instances — cloud storage (covered next) is almost always the better choice.

## 12.3 Image Upload

Image uploads are extremely common and often need extra handling: validating that the file is genuinely an image (not just trusting the file extension), enforcing size/dimension limits, and sometimes generating thumbnails.

```java
public void validateImage(MultipartFile file) {
    String contentType = file.getContentType();
    if (contentType == null || !contentType.startsWith("image/")) {
        throw new InvalidFileException("Only image files are allowed");
    }
    if (file.getSize() > 5 * 1024 * 1024) {
        throw new InvalidFileException("Image must be smaller than 5MB");
    }
}
```

For resizing/generating thumbnails, libraries like `Thumbnailator` or `imgscalr` are commonly used alongside Spring Boot to process images server-side before storing them.

## 12.4 Cloud Storage

Rather than storing files on the local server disk (which doesn't scale across multiple instances and risks data loss if a server dies), production applications typically store uploaded files in cloud object storage — Amazon S3 being the most common choice.

```java
@Service
public class S3StorageService {

    private final S3Client s3Client;
    private final String bucketName = "my-app-uploads";

    public String upload(MultipartFile file) throws IOException {
        String key = UUID.randomUUID() + "_" + file.getOriginalFilename();

        s3Client.putObject(
            PutObjectRequest.builder().bucket(bucketName).key(key).build(),
            RequestBody.fromInputStream(file.getInputStream(), file.getSize())
        );

        return key;
    }
}
```

Benefits of cloud storage over local disk: virtually unlimited scalability, built-in redundancy/durability, works seamlessly across multiple app server instances (no "which server has this file?" problem), and often comes with built-in CDN integration for fast global delivery.

## 12.5 Download Files

Serving a file back to a client requires setting the right headers so browsers handle it correctly (either displaying it inline, or triggering a download):

```java
@GetMapping("/download/{filename}")
public ResponseEntity<Resource> downloadFile(@PathVariable String filename) throws IOException {
    Path filePath = storageLocation.resolve(filename);
    Resource resource = new UrlResource(filePath.toUri());

    if (!resource.exists()) {
        return ResponseEntity.notFound().build();
    }

    return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + filename + "\"")
            .contentType(MediaType.APPLICATION_OCTET_STREAM)
            .body(resource);
}
```

The `Content-Disposition: attachment` header tells the browser "download this file" rather than trying to display it inline. Use `inline` instead of `attachment` if you want the browser to try to display it directly (e.g., for a PDF viewer or image preview).

---

# 13. Email & Notifications

## 13.1 Spring Mail

`spring-boot-starter-mail` provides a simple abstraction (`JavaMailSender`) over Java's underlying email-sending APIs, letting you send emails with just a few lines of code instead of dealing with raw SMTP protocol details.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

```java
@Service
public class EmailService {

    private final JavaMailSender mailSender;

    public EmailService(JavaMailSender mailSender) {
        this.mailSender = mailSender;
    }

    public void sendSimpleEmail(String to, String subject, String body) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(to);
        message.setSubject(subject);
        message.setText(body);
        mailSender.send(message);
    }
}
```

## 13.2 SMTP

SMTP (Simple Mail Transfer Protocol) is the underlying protocol actually used to send email across the internet. Spring Boot auto-configures a `JavaMailSender` bean once you provide SMTP connection details in configuration:

```properties
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=your-email@gmail.com
spring.mail.password=your-app-password
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
```

For production systems, teams typically use a dedicated transactional email provider (like SendGrid, Mailgun, or Amazon SES) rather than a personal Gmail account — these provide much better deliverability, analytics, and are designed for high-volume automated sending.

## 13.3 HTML Emails

Plain text emails are limiting — most real-world application emails (welcome emails, password resets, receipts) use rich HTML formatting. Spring supports this through `MimeMessage` combined with a templating engine like Thymeleaf.

```java
public void sendHtmlEmail(String to, String subject, String htmlBody) throws MessagingException {
    MimeMessage message = mailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(message, true);
    helper.setTo(to);
    helper.setSubject(subject);
    helper.setText(htmlBody, true); // true = isHtml
    mailSender.send(message);
}
```

Combined with Thymeleaf to render a proper template with dynamic data:

```java
Context context = new Context();
context.setVariable("username", user.getUsername());
String htmlBody = templateEngine.process("welcome-email", context);
sendHtmlEmail(user.getEmail(), "Welcome!", htmlBody);
```

## 13.4 Attachments

Adding file attachments (invoices, reports, receipts) to an email uses the same `MimeMessageHelper`:

```java
public void sendEmailWithAttachment(String to, String subject, String body, File attachment) throws MessagingException {
    MimeMessage message = mailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(message, true);
    helper.setTo(to);
    helper.setSubject(subject);
    helper.setText(body);
    helper.addAttachment(attachment.getName(), attachment);
    mailSender.send(message);
}
```

## 13.5 OTP Email

OTP (One-Time Password) emails are extremely common for verification flows — signup confirmation, password reset, two-factor authentication. The pattern is straightforward: generate a random code, store it (with an expiration) associated with the user, email it to them, and validate it when they submit it back.

```java
@Service
public class OtpService {

    private final Map<String, OtpEntry> otpStore = new ConcurrentHashMap<>(); // use Redis in production

    public String generateAndSendOtp(String email) {
        String otp = String.format("%06d", new SecureRandom().nextInt(1_000_000));
        otpStore.put(email, new OtpEntry(otp, Instant.now().plus(5, ChronoUnit.MINUTES)));
        emailService.sendSimpleEmail(email, "Your Verification Code", "Your OTP is: " + otp);
        return otp;
    }

    public boolean verifyOtp(String email, String submittedOtp) {
        OtpEntry entry = otpStore.get(email);
        if (entry == null || entry.isExpired() || !entry.code().equals(submittedOtp)) {
            return false;
        }
        otpStore.remove(email); // OTPs should be single-use
        return true;
    }
}
```

Using `SecureRandom` instead of plain `Random` is important here — it's cryptographically strong, meaning the generated codes aren't predictable, which matters for a security-sensitive feature like this.

## 13.6 SMS Integration

For SMS-based notifications (OTPs, alerts, order updates), Spring Boot apps typically integrate with a third-party SMS gateway provider like Twilio or AWS SNS, since actually sending SMS requires carrier-level infrastructure no application can reasonably build itself.

```java
@Service
public class SmsService {

    public void sendSms(String toPhoneNumber, String message) {
        Message.creator(
            new PhoneNumber(toPhoneNumber),
            new PhoneNumber(twilioFromNumber),
            message
        ).create();
    }
}
```

The Spring Boot side of this integration is usually thin — you're mostly just wrapping the provider's SDK/API in a clean service interface, so the rest of your application doesn't need to know which specific SMS provider you're using underneath.

## 13.7 WhatsApp Integration

WhatsApp Business API integration follows a very similar pattern to SMS — you integrate with a provider (Meta's own Cloud API, or third parties like Twilio that offer a WhatsApp channel) via their REST API.

```java
@Service
public class WhatsAppService {

    private final RestTemplate restTemplate;

    public void sendMessage(String toPhoneNumber, String templateName, Map<String, String> params) {
        WhatsAppMessageRequest request = new WhatsAppMessageRequest(toPhoneNumber, templateName, params);
        restTemplate.postForEntity(whatsAppApiUrl, request, WhatsAppResponse.class);
    }
}
```

One important nuance: WhatsApp Business messaging generally requires using PRE-APPROVED message templates for the first outbound message in a conversation (to prevent spam) — you can't just send arbitrary free-form text the way you can with SMS or email, at least not until the user has replied within a live conversation window.

---

# 14. Scheduling

## 14.1 Scheduled Tasks

Many applications need to run code automatically at set intervals — cleaning up expired sessions, sending daily digest emails, syncing data with an external system. Spring makes this trivial with `@Scheduled`.

```java
@Configuration
@EnableScheduling
public class SchedulingConfig { }
```

```java
@Component
public class CleanupTasks {

    @Scheduled(fixedRate = 60000) // runs every 60 seconds
    public void purgeExpiredSessions() {
        sessionRepository.deleteAllExpired();
    }

    @Scheduled(fixedDelay = 30000) // waits 30 seconds AFTER the previous run finishes, then runs again
    public void syncInventory() {
        inventoryService.syncWithWarehouse();
    }

    @Scheduled(initialDelay = 5000, fixedRate = 3600000) // waits 5s before first run, then every hour
    public void hourlyReport() {
        reportService.generateHourlyReport();
    }
}
```

The distinction between `fixedRate` and `fixedDelay` matters: `fixedRate` triggers based on when the PREVIOUS execution STARTED (so if a task takes longer than the rate, executions can overlap or queue up); `fixedDelay` waits for the previous execution to FINISH before counting down to the next one.

## 14.2 Cron Expressions

For more complex scheduling patterns ("every weekday at 9am," "on the 1st of every month at midnight"), `@Scheduled` also supports standard cron syntax.

```java
@Scheduled(cron = "0 0 9 * * MON-FRI")  // 9:00 AM, Monday through Friday
public void sendDailyDigest() { }

@Scheduled(cron = "0 0 0 1 * *")  // midnight on the 1st of every month
public void generateMonthlyInvoices() { }
```

The cron expression format Spring uses is: `second minute hour day-of-month month day-of-week`. It's worth keeping a cron cheat-sheet handy, since the syntax (while powerful) is genuinely easy to get subtly wrong.

You can also externalize the cron expression into configuration, so it can be changed without recompiling:

```java
@Scheduled(cron = "${app.scheduling.daily-digest-cron}")
public void sendDailyDigest() { }
```

## 14.3 Async Jobs

Sometimes a scheduled (or triggered) task takes a long time, and you don't want it blocking whatever called it. Combining `@Scheduled` (or a regular method) with `@Async` runs the task on a separate thread, letting the caller continue immediately.

```java
@Async
@Scheduled(cron = "0 0 2 * * *")
public void nightlyDataExport() {
    // long-running export logic — runs on a background thread pool,
    // doesn't block anything else in the application
}
```

We'll cover `@Async` and thread pool configuration in much more depth in the next section.

## 14.4 Thread Pool Scheduling

By default, Spring's `@Scheduled` tasks all run on a SINGLE thread — meaning if you have multiple scheduled tasks, and one runs long, it can delay the others from firing on time. For any real application with more than a trivial scheduling need, you should configure a proper thread pool.

```java
@Configuration
@EnableScheduling
public class SchedulingConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(10);
        scheduler.setThreadNamePrefix("scheduled-task-");
        scheduler.initialize();
        taskRegistrar.setTaskScheduler(scheduler);
    }
}
```

With a properly sized thread pool, multiple `@Scheduled` methods can genuinely run concurrently, instead of silently queuing up behind each other on a single shared thread.

---
# 15. Asynchronous Programming

## 15.1 @Async

By default, every method call in Java runs synchronously on the calling thread — the caller waits for the method to finish before moving on. `@Async` lets you mark a method to run on a SEPARATE thread instead, so the caller can continue immediately without waiting.

```java
@Configuration
@EnableAsync
public class AsyncConfig { }
```

```java
@Service
public class NotificationService {

    @Async
    public void sendWelcomeEmail(String email) {
        // this runs on a background thread — the caller doesn't wait for it
        emailService.send(email, "Welcome!", "Thanks for signing up");
    }
}
```

```java
@Service
public class RegistrationService {

    public void register(UserRequest request) {
        User user = userRepository.save(new User(request));
        notificationService.sendWelcomeEmail(user.getEmail()); // returns immediately, email sends in background
        // registration completes fast, without waiting for the email to actually send
    }
}
```

**Important gotcha:** `@Async` only works when called from a DIFFERENT bean than the one it's defined in — because Spring implements this using a proxy, and calling a method on `this` (from within the same class) bypasses the proxy entirely, silently running synchronously instead. This trips up a lot of developers the first time they use `@Async`.

## 15.2 CompletableFuture

When you need to actually GET A RESULT BACK from an async operation (not just fire-and-forget), `@Async` methods should return a `CompletableFuture<T>` instead of `void`.

```java
@Async
public CompletableFuture<InventoryStatus> checkInventoryAsync(Long productId) {
    InventoryStatus status = inventoryClient.check(productId);
    return CompletableFuture.completedFuture(status);
}
```

```java
public OrderSummary buildOrderSummary(Long orderId) {
    CompletableFuture<InventoryStatus> inventoryFuture = inventoryService.checkInventoryAsync(productId);
    CompletableFuture<CustomerInfo> customerFuture = customerService.getCustomerAsync(customerId);

    // both calls run concurrently, THEN we wait for both to finish
    CompletableFuture.allOf(inventoryFuture, customerFuture).join();

    return new OrderSummary(inventoryFuture.join(), customerFuture.join());
}
```

`CompletableFuture` is genuinely powerful — beyond just holding a future result, it supports composing multiple async operations together (`thenApply`, `thenCombine`, `allOf`, `exceptionally`), letting you build sophisticated concurrent pipelines without manually managing threads yourself.

## 15.3 ExecutorService

Under the hood, both `@Async` and `CompletableFuture` ultimately rely on an `ExecutorService` — Java's standard abstraction for managing a pool of worker threads that execute submitted tasks.

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

Future<String> future = executor.submit(() -> {
    return heavyComputation();
});

String result = future.get(); // blocks until the task completes
executor.shutdown(); // always shut down executors when you're done with them, to release threads
```

While you CAN work with `ExecutorService` directly, in a Spring Boot application it's much more common to let Spring manage the thread pool for you (via `@Async` and a configured `TaskExecutor` bean, covered next) — it integrates properly with Spring's lifecycle and exception handling.

## 15.4 ThreadPoolTaskExecutor

By default, Spring's `@Async` uses a simple thread pool that creates a NEW thread for every task (`SimpleAsyncTaskExecutor`) — which is fine for quick experiments, but genuinely dangerous in production, since it has no upper bound and can exhaust system resources under load. You should almost always configure a proper, bounded thread pool.

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-task-");
        executor.initialize();
        return executor;
    }
}
```

```java
@Async("taskExecutor")
public void doWork() { }
```

The key settings to understand: `corePoolSize` is the number of threads kept alive even when idle; `maxPoolSize` is the ceiling the pool can grow to under load; `queueCapacity` is how many pending tasks can wait in line before new threads are spun up (or before tasks start getting REJECTED, if you've hit `maxPoolSize` too). Sizing these correctly for your workload — CPU-bound vs I/O-bound tasks behave very differently — is a genuinely important production tuning consideration.

---

# 16. Spring AOP

## 16.1 Aspect Oriented Programming

AOP (Aspect-Oriented Programming) is a programming paradigm designed to handle "cross-cutting concerns" — functionality that applies across MANY parts of an application (logging, security checks, transaction management, performance monitoring) but doesn't naturally belong inside any single class's core business logic.

Without AOP, you'd have to manually sprinkle logging/timing/security code into every single method that needs it — messy, repetitive, and easy to forget. AOP lets you define that cross-cutting logic ONCE, in one place, and have it automatically applied wherever needed, without touching the original classes at all.

You've actually already seen AOP in disguise: `@Transactional` and `@Async` are both implemented using Spring's AOP machinery under the hood — the "extra behavior" (starting a transaction, running on a different thread) is woven around your method without you writing that plumbing yourself.

## 16.2 Advice

"Advice" is the actual code that runs as part of an aspect — the ACTION taken at a particular point. Spring supports several types:

- `@Before` — runs before the target method executes
- `@After` — runs after the target method completes (whether it succeeded or threw an exception)
- `@AfterReturning` — runs only if the method completes successfully
- `@AfterThrowing` — runs only if the method throws an exception
- `@Around` — wraps the ENTIRE method execution, giving you full control (you decide whether/when the actual method runs at all)

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(* com.myapp.service.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Calling: " + joinPoint.getSignature().getName());
    }

    @AfterThrowing(pointcut = "execution(* com.myapp.service.*.*(..))", throwing = "ex")
    public void logException(JoinPoint joinPoint, Exception ex) {
        System.out.println("Exception in " + joinPoint.getSignature().getName() + ": " + ex.getMessage());
    }
}
```

## 16.3 Pointcut

A "pointcut" is an expression that defines WHERE (which methods, classes, packages) an aspect's advice should actually be applied. Think of it as a filter/pattern-matcher over your codebase.

```java
@Pointcut("execution(* com.myapp.service.*.*(..))")
public void serviceLayerMethods() {}

@Pointcut("@annotation(com.myapp.annotation.LogExecutionTime)")
public void annotatedMethods() {}

@Before("serviceLayerMethods()")
public void logServiceCalls(JoinPoint joinPoint) {
    System.out.println("Service method called: " + joinPoint.getSignature());
}
```

The `execution(* com.myapp.service.*.*(..))` pattern reads as: "any return type (`*`), in any class directly inside `com.myapp.service`, any method name (`*`), with any parameters (`..`)." Pointcut expressions can get quite sophisticated, matching by annotation, class hierarchy, or specific parameter types.

## 16.4 Join Point

A "join point" is a specific POINT during execution where an aspect COULD be applied — in Spring AOP, this is essentially always a method invocation (unlike full AspectJ, which supports many more join point types, like field access). When advice actually fires at a given join point, you can inspect details about that specific invocation through the `JoinPoint` object — method name, arguments, target object, and so on.

```java
@Around("serviceLayerMethods()")
public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
    long start = System.currentTimeMillis();
    Object result = joinPoint.proceed(); // actually invokes the real method
    long duration = System.currentTimeMillis() - start;
    System.out.println(joinPoint.getSignature() + " took " + duration + "ms");
    return result;
}
```

Note `ProceedingJoinPoint` (used specifically with `@Around` advice) — it's the only join point type that lets you control WHETHER and WHEN the actual method executes, via `joinPoint.proceed()`. This is what makes `@Around` the most powerful (and most commonly used for things like timing and caching) advice type.

## 16.5 Logging Aspect

Putting it all together — a practical, reusable logging aspect that automatically logs every service-layer method call, its arguments, and its result, without a single line of logging code inside the actual service classes:

```java
@Aspect
@Component
@Slf4j
public class ServiceLoggingAspect {

    @Around("execution(* com.myapp.service.*.*(..))")
    public Object logMethodCall(ProceedingJoinPoint joinPoint) throws Throwable {
        String methodName = joinPoint.getSignature().toShortString();
        log.info("Entering: {} with args: {}", methodName, Arrays.toString(joinPoint.getArgs()));

        try {
            Object result = joinPoint.proceed();
            log.info("Exiting: {} with result: {}", methodName, result);
            return result;
        } catch (Exception ex) {
            log.error("Exception in: {} - {}", methodName, ex.getMessage());
            throw ex;
        }
    }
}
```

This single aspect, once written, automatically applies to every current AND future method inside `com.myapp.service` — a great example of AOP's real power: write once, apply broadly, without touching business logic classes at all.

## 16.6 Performance Monitoring

A very common real-world AOP use case: automatically measuring and logging how long specific methods take, WITHOUT manually adding timing code to each one. Often built as a custom annotation + aspect combo for opt-in usage:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Timed { }
```

```java
@Aspect
@Component
public class TimingAspect {

    @Around("@annotation(com.myapp.annotation.Timed)")
    public Object measureExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.nanoTime();
        try {
            return joinPoint.proceed();
        } finally {
            long durationMs = (System.nanoTime() - start) / 1_000_000;
            log.info("{} executed in {}ms", joinPoint.getSignature().toShortString(), durationMs);
        }
    }
}
```

Now, ANY method you want to monitor just gets annotated `@Timed` — clean, declarative, opt-in, and completely decoupled from the actual business logic of the method itself:

```java
@Timed
public List<Product> searchProducts(String query) {
    // business logic only — no timing code needed here at all
}
```

---
# 17. Caching

## 17.1 Spring Cache

Spring's caching abstraction (`spring-boot-starter-cache`) provides a simple, annotation-driven way to cache the results of expensive method calls — database queries, external API calls, complex computations — WITHOUT changing the actual method's code, similar in spirit to how `@Transactional` works.

```java
@Configuration
@EnableCaching
public class CacheConfig { }
```

```java
@Service
public class ProductService {

    @Cacheable("products")
    public Product findById(Long id) {
        System.out.println("Fetching from database..."); // only prints on a cache MISS
        return productRepository.findById(id).orElseThrow();
    }
}
```

The first time `findById(5L)` is called, it actually hits the database and caches the result under key `5`. Every subsequent call with the SAME argument returns the cached value instantly, without touching the database at all — until the cache entry is evicted or expires.

Other key caching annotations:

```java
@CachePut("products") // ALWAYS executes the method AND updates the cache (used for updates)
public Product update(Product product) { ... }

@CacheEvict("products") // removes an entry from the cache (used for deletes)
public void delete(Long id) { ... }

@CacheEvict(value = "products", allEntries = true) // clears the ENTIRE cache
public void clearAll() { ... }
```

## 17.2 Cache Manager

`CacheManager` is the abstraction Spring uses to actually manage cache storage — WHERE the cached data physically lives. By default, Spring Boot uses a simple in-memory `ConcurrentHashMap`-based cache (fine for development, but not shared across multiple server instances, and lost on restart). For production, you typically swap in a dedicated caching provider.

```java
@Bean
public CacheManager cacheManager() {
    CaffeineCacheManager cacheManager = new CaffeineCacheManager();
    cacheManager.setCaffeine(Caffeine.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(10, TimeUnit.MINUTES));
    return cacheManager;
}
```

Common `CacheManager` implementations: `ConcurrentMapCacheManager` (default, simple, local-only), `CaffeineCacheManager` (high-performance in-memory library with rich eviction policies), and `RedisCacheManager` (distributed, shared across multiple app instances — covered in the Redis section).

## 17.3 Redis Cache

For any application running MULTIPLE instances (which is basically any real production deployment), an in-memory local cache is a problem: each instance would have its own separate cache, leading to inconsistent data across instances. Redis, being an external, shared cache store, solves this — every instance reads and writes from the SAME cache.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```java
@Bean
public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
    RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
        .entryTtl(Duration.ofMinutes(30))
        .serializeValuesWith(RedisSerializationContext.SerializationPair
            .fromSerializer(new GenericJackson2JsonRedisSerializer()));

    return RedisCacheManager.builder(connectionFactory).cacheDefaults(config).build();
}
```

With this configuration in place, the exact same `@Cacheable` annotations from before now transparently use Redis instead of local memory — no changes needed to your service classes at all, since they only ever interacted with the abstraction, never the underlying storage mechanism directly.

## 17.4 Eviction

Cache eviction is the process of removing entries from the cache — either because they're explicitly invalidated (data changed, so the cached copy is now stale) or because the cache needs to free up space.

**Explicit eviction** (you control exactly when):
```java
@CacheEvict(value = "products", key = "#id")
public void updateStock(Long id, int newStock) {
    productRepository.updateStock(id, newStock);
    // cached entry for this specific product is removed, forcing a fresh fetch next time
}
```

**Automatic/policy-based eviction** (the cache decides, based on rules) commonly uses strategies like:
- **LRU (Least Recently Used)** — evict whatever hasn't been accessed in the longest time
- **LFU (Least Frequently Used)** — evict whatever is accessed least often
- **Size-based** — evict once the cache exceeds a maximum number of entries

Choosing the right eviction strategy matters a lot for cache effectiveness — get it wrong, and you either waste memory on rarely-used data, or evict frequently-needed data too aggressively.

## 17.5 TTL

TTL (Time To Live) defines how long a cached entry remains valid before it's automatically considered stale and removed/refreshed — a form of automatic, time-based eviction.

```java
RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(15));
```

Choosing an appropriate TTL is a genuine balancing act specific to your data:
- Data that rarely changes (like a list of countries, or category names) can have a LONG TTL (hours or even days) — safe, and maximizes cache hit rate
- Data that changes frequently (like live stock/inventory counts) needs a SHORT TTL, or explicit eviction on update, to avoid serving stale, incorrect data to users

There's no universal "correct" TTL — it always depends on how tolerant your specific use case is of slightly-stale data versus the performance cost of frequent cache misses.

---

# 18. Logging

## 18.1 SLF4J

SLF4J (Simple Logging Facade for Java) is NOT a logging implementation itself — it's an ABSTRACTION layer that sits in front of whatever actual logging framework you use underneath (Logback, Log4j2, java.util.logging). This means your application code logs against the SLF4J API, and you can swap the underlying implementation without changing a single line of your logging code.

```java
private static final Logger logger = LoggerFactory.getLogger(OrderService.class);

logger.info("Processing order {}", orderId);
logger.error("Failed to process order {}", orderId, exception);
```

Spring Boot uses SLF4J as its logging facade by default, with **Logback** as the actual underlying implementation — so out of the box, you already have a working, well-configured logging setup with zero extra dependencies needed.

With Lombok, you can skip the manual logger declaration entirely using `@Slf4j`:

```java
@Slf4j
@Service
public class OrderService {
    public void process(Long orderId) {
        log.info("Processing order {}", orderId);
    }
}
```

## 18.2 Logback

Logback is Spring Boot's default logging implementation — the thing that actually formats and writes your log messages, whether that's to the console, a file, or elsewhere. You configure it through `logback-spring.xml` (the `-spring` suffix enables Spring-specific extensions like profile-based configuration).

```xml
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/application.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

For SIMPLE customizations, you often don't even need a full `logback-spring.xml` — `application.properties` alone can handle it, as shown next.

## 18.3 Log Levels

Log levels let you control HOW MUCH detail gets logged, and filter out noise you don't care about at any given moment. From most to least verbose:

| Level | When to use |
|---|---|
| `TRACE` | Extremely fine-grained diagnostic detail, rarely enabled |
| `DEBUG` | Detailed information useful during development/troubleshooting |
| `INFO` | General, high-level application flow events (default level) |
| `WARN` | Something unexpected happened, but the app can continue |
| `ERROR` | A genuine failure occurred that needs attention |

```properties
logging.level.root=INFO
logging.level.com.myapp=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.springframework.web=WARN
```

This lets you get VERBOSE logging just for the parts of the system you're actively debugging (like your own `com.myapp` package, or Hibernate's generated SQL), while keeping everything else at a quieter, more reasonable level.

## 18.4 Structured Logging

Traditional log lines are plain, human-readable text — great for a developer reading logs directly in a terminal, but painful for automated log aggregation/analysis tools to parse reliably. **Structured logging** formats log entries as JSON (or another structured format) instead, making them trivially machine-parseable.

```json
{"timestamp":"2026-07-28T10:15:30Z","level":"INFO","logger":"OrderService","message":"Processing order","orderId":12345,"traceId":"abc-123"}
```

Spring Boot 3.4+ added built-in structured logging support:

```properties
logging.structured.format.console=ecs
```

This is especially valuable in modern cloud/microservices deployments, where logs from many services get aggregated into centralized platforms (like the ELK stack, Datadog, or Splunk) — structured JSON logs can be indexed, filtered, and searched far more effectively than parsing free-text log lines with regex.

## 18.5 MDC

MDC (Mapped Diagnostic Context) lets you attach contextual data (like a request ID, user ID, or trace ID) to EVERY log statement within a given thread's execution — without having to manually pass that context into every single log call.

```java
public class RequestIdFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                     FilterChain filterChain) throws ServletException, IOException {
        String requestId = UUID.randomUUID().toString();
        MDC.put("requestId", requestId);
        try {
            filterChain.doFilter(request, response);
        } finally {
            MDC.clear(); // always clean up, since threads are reused from a pool
        }
    }
}
```

```xml
<pattern>%d{yyyy-MM-dd HH:mm:ss} [%X{requestId}] %-5level %logger{36} - %msg%n</pattern>
```

Now EVERY log line produced while handling that request automatically includes the request ID — hugely valuable when debugging production issues, letting you filter logs down to exactly the events tied to one specific request, even across a busy, high-traffic system.

## 18.6 Log Rotation

Log files grow forever if left unchecked, eventually filling up disk space and crashing your server. Log rotation automatically manages this — archiving old log content into separate files (often compressed) on a schedule, and deleting sufficiently old archives.

```xml
<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
    <fileNamePattern>logs/application.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
    <maxFileSize>100MB</maxFileSize>
    <maxHistory>30</maxHistory>
    <totalSizeCap>3GB</totalSizeCap>
</rollingPolicy>
```

This configuration rotates logs daily (or when a file exceeds 100MB, whichever comes first), compresses old files, keeps at most 30 days of history, and caps total log storage at 3GB — a sensible, safe default for most production applications, preventing logs from ever silently consuming your entire disk.

---
# 19. Testing

## 19.1 JUnit 5

JUnit 5 is the standard testing framework for Java, and it's what Spring Boot's testing starter is built around. It's a big improvement over JUnit 4, with a more modular architecture and much richer annotation support.

```java
class OrderCalculatorTest {

    private OrderCalculator calculator = new OrderCalculator();

    @Test
    void shouldCalculateTotalWithTax() {
        BigDecimal total = calculator.calculateTotal(BigDecimal.valueOf(100), BigDecimal.valueOf(0.08));
        assertEquals(BigDecimal.valueOf(108.00).setScale(2), total);
    }

    @Test
    void shouldThrowExceptionForNegativeAmount() {
        assertThrows(IllegalArgumentException.class, () -> calculator.calculateTotal(BigDecimal.valueOf(-10), BigDecimal.ZERO));
    }

    @ParameterizedTest
    @ValueSource(ints = {1, 5, 10, 100})
    void shouldHandleVariousQuantities(int quantity) {
        assertTrue(calculator.isValidQuantity(quantity));
    }

    @BeforeEach
    void setUp() {
        // runs before EVERY test method — good for resetting shared state
    }
}
```

Key annotations: `@Test` marks a test method, `@BeforeEach`/`@AfterEach` run before/after every single test, `@BeforeAll`/`@AfterAll` run once for the whole class, and `@ParameterizedTest` lets you run the same test logic against multiple different inputs without duplicating code.

## 19.2 Mockito

Real classes almost always depend on OTHER classes (repositories, external services). When unit testing, you don't want your test to actually hit a real database or call a real external API — you want to isolate JUST the class under test. Mockito lets you create fake ("mock") versions of dependencies, with fully controllable behavior.

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private PaymentService paymentService;

    @InjectMocks
    private OrderService orderService; // Mockito automatically injects the mocks above into this

    @Test
    void shouldThrowWhenOrderNotFound() {
        when(orderRepository.findById(1L)).thenReturn(Optional.empty());

        assertThrows(ResourceNotFoundException.class, () -> orderService.getOrder(1L));
    }

    @Test
    void shouldChargePaymentWhenPlacingOrder() {
        Order order = new Order(1L, "customer", BigDecimal.TEN);
        when(orderRepository.save(any())).thenReturn(order);

        orderService.placeOrder(new OrderRequest(...));

        verify(paymentService, times(1)).charge(any()); // confirms this method was actually called exactly once
    }
}
```

`when(...).thenReturn(...)` defines mock behavior; `verify(...)` confirms a specific interaction actually happened — these two patterns cover the vast majority of real-world Mockito usage.

## 19.3 MockMvc

`MockMvc` lets you test your web layer (controllers, request mapping, JSON serialization, validation, status codes) WITHOUT starting a real HTTP server — it simulates the full Spring MVC request-handling pipeline in-process, which is much faster than a true end-to-end test.

```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private OrderService orderService;

    @Test
    void shouldReturnOrderById() throws Exception {
        when(orderService.getOrder(1L)).thenReturn(new Order(1L, "John", BigDecimal.TEN));

        mockMvc.perform(get("/api/orders/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.customerName").value("John"));
    }

    @Test
    void shouldReturn400ForInvalidRequest() throws Exception {
        mockMvc.perform(post("/api/orders")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{}")) // missing required fields
                .andExpect(status().isBadRequest());
    }
}
```

`@WebMvcTest` loads ONLY the web layer (controllers, filters, `@ControllerAdvice`), not the full application context — making these tests fast, while `@MockBean` provides a mock replacement for the service layer so you're testing the controller in true isolation.

## 19.4 Unit Testing

A "unit test" tests ONE unit of code (typically one method, or one class) in complete isolation from its dependencies — using mocks for anything external. Unit tests should be:

- **Fast** — running thousands of them should take seconds, not minutes
- **Isolated** — no database, no network calls, no filesystem access
- **Deterministic** — the exact same result every single run, regardless of environment or order

```java
@Test
void shouldApplyDiscountCorrectly() {
    PricingService pricingService = new PricingService();
    BigDecimal result = pricingService.applyDiscount(BigDecimal.valueOf(100), 0.10);
    assertEquals(BigDecimal.valueOf(90), result);
}
```

A large, healthy test suite is usually shaped like a pyramid: MANY unit tests (fast, cheap, covering fine-grained logic), FEWER integration tests, and just a HANDFUL of full end-to-end tests — because unit tests give you the fastest possible feedback loop during development.

## 19.5 Integration Testing

Unlike unit tests, integration tests verify that MULTIPLE parts of your system work correctly TOGETHER — for example, that your service layer, repository layer, and an ACTUAL database genuinely integrate correctly, catching issues that mocks might hide (like a subtly wrong JPQL query, or a mapping mismatch).

```java
@SpringBootTest
@AutoConfigureMockMvc
class OrderIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private OrderRepository orderRepository;

    @Test
    void shouldCreateAndRetrieveOrder() throws Exception {
        mockMvc.perform(post("/api/orders")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{\"customerName\":\"Alice\",\"items\":[...]}"))
                .andExpect(status().isCreated());

        assertEquals(1, orderRepository.count()); // confirms it was genuinely persisted
    }
}
```

`@SpringBootTest` loads the FULL application context (unlike `@WebMvcTest`'s slice), giving you a much more realistic (but slower) test — genuinely exercising the real wiring between your layers.

## 19.6 TestContainers

Here's a real dilemma integration tests face: if you test against an in-memory database (like H2) instead of your REAL production database (like PostgreSQL), you risk missing bugs caused by database-specific behavior differences. But you also don't want tests depending on a shared, manually-managed test database. **TestContainers** solves this beautifully: it spins up REAL database instances (and other services) in throwaway Docker containers, automatically, just for the duration of your test run.

```java
@SpringBootTest
@Testcontainers
class OrderRepositoryIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private OrderRepository orderRepository;

    @Test
    void shouldPersistOrderToRealPostgres() {
        Order saved = orderRepository.save(new Order("Alice", BigDecimal.TEN));
        assertNotNull(saved.getId());
    }
}
```

This gives you the confidence of testing against your ACTUAL production database engine, while keeping tests fully automated and self-contained — no manually-provisioned shared test database required, and every test run starts from a clean slate.

## 19.7 API Testing

API testing verifies your API's behavior from the OUTSIDE, exactly the way a real client would interact with it — sending real HTTP requests and asserting on real HTTP responses, often as part of a fully-running application (as opposed to `MockMvc`'s simulated in-process requests).

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class OrderApiTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void shouldCreateOrderSuccessfully() {
        OrderRequest request = new OrderRequest("Alice", List.of(...));

        ResponseEntity<OrderResponse> response = restTemplate.postForEntity("/api/orders", request, OrderResponse.class);

        assertEquals(HttpStatus.CREATED, response.getStatusCode());
        assertNotNull(response.getBody().getId());
    }
}
```

`webEnvironment = RANDOM_PORT` starts a genuinely running embedded server on a random free port, and `TestRestTemplate` makes real HTTP calls against it — this is about as close to "real production usage" as an automated test can get, short of a full end-to-end test against a deployed environment. Tools like Postman/Newman or REST Assured are also popular for API testing, especially for testing already-deployed environments outside the build pipeline.

---
# 20. OpenAPI Documentation

## 20.1 Swagger

"Swagger" is the name most developers still use colloquially, though technically it now refers mostly to the TOOLING (Swagger UI, Swagger Editor) built around the OpenAPI Specification. Swagger UI generates an interactive, browsable web page documenting your entire API — every endpoint, every parameter, every request/response schema — and lets developers try out real requests directly from the browser.

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

Just adding this dependency to a Spring Boot project automatically generates full API documentation by scanning your controllers, and exposes an interactive UI at `/swagger-ui.html` — with zero manual configuration required to get started.

## 20.2 OpenAPI

OpenAPI (formerly known as "Swagger Specification" before being donated to the Linux Foundation) is the actual STANDARD/SPECIFICATION describing how a REST API should be documented — a language-agnostic, machine-readable format (usually JSON or YAML) that fully describes an API's endpoints, parameters, request/response schemas, authentication requirements, and more.

```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("Order Management API")
                .version("1.0")
                .description("REST API for managing customer orders"))
            .components(new Components()
                .addSecuritySchemes("bearerAuth",
                    new SecurityScheme().type(SecurityScheme.Type.HTTP).scheme("bearer").bearerFormat("JWT")));
    }
}
```

This raw OpenAPI JSON/YAML document is what tools like Swagger UI actually READ and RENDER into that nice interactive page — and it's also what many client-code-generation tools use to auto-generate typed API clients in other languages (TypeScript, Python, etc.) directly from your API's contract.

## 20.3 API Documentation

Beyond the auto-generated basics, you can (and should) enrich your API documentation with meaningful descriptions, examples, and details that aren't obvious just from method signatures alone.

```java
@RestController
@RequestMapping("/api/orders")
@Tag(name = "Orders", description = "Endpoints for managing customer orders")
public class OrderController {

    @Operation(summary = "Get an order by ID", description = "Returns full order details including line items")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Order found"),
        @ApiResponse(responseCode = "404", description = "Order not found")
    })
    @GetMapping("/{id}")
    public ResponseEntity<OrderResponse> getOrder(
            @Parameter(description = "The unique ID of the order") @PathVariable Long id) {
        // ...
    }
}
```

Good API documentation isn't just a "nice to have" — for any API consumed by other teams, external partners, or even just your own frontend team, it's often the PRIMARY way people learn how to use your API correctly, without needing to read your source code or bother you with questions.

## 20.4 API Examples

Concrete examples make documentation dramatically more useful than schema definitions alone — seeing an actual sample request/response is often far more immediately understandable than an abstract type definition.

```java
@Schema(example = "{ \"customerName\": \"Alice Smith\", \"items\": [{\"productId\": 42, \"quantity\": 2}] }")
public class OrderRequest {
    private String customerName;
    private List<OrderItemRequest> items;
}
```

Or, at the operation level:

```java
@Operation(
    summary = "Create a new order",
    requestBody = @io.swagger.v3.oas.annotations.parameters.RequestBody(
        content = @Content(examples = @ExampleObject(
            name = "Simple order",
            value = "{\"customerName\": \"Alice\", \"items\": [{\"productId\": 1, \"quantity\": 3}]}"
        ))
    )
)
@PostMapping
public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest request) { }
```

When Swagger UI renders this, developers exploring your API can see (and even directly execute) a working example request, rather than having to reverse-engineer the correct JSON shape from a schema definition alone.

---

# 21. Spring Boot Actuator

## 21.1 Health Checks

Actuator is Spring Boot's built-in module for exposing production-ready operational information about your running application — health, metrics, environment info, and more — through simple HTTP endpoints.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

The health endpoint (`/actuator/health`) reports whether the application (and its critical dependencies — database, disk space, message brokers) is functioning correctly:

```json
{
  "status": "UP",
  "components": {
    "db": { "status": "UP" },
    "diskSpace": { "status": "UP" },
    "redis": { "status": "UP" }
  }
}
```

This endpoint is exactly what load balancers, Kubernetes liveness/readiness probes, and monitoring systems poll continuously to determine whether an instance is healthy enough to receive traffic. You can also write CUSTOM health indicators for your own application-specific checks:

```java
@Component
public class PaymentGatewayHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        boolean reachable = paymentGatewayClient.ping();
        return reachable ? Health.up().build() : Health.down().withDetail("reason", "Gateway unreachable").build();
    }
}
```

## 21.2 Metrics

Actuator also exposes detailed runtime metrics — JVM memory usage, garbage collection stats, HTTP request counts and latencies, thread pool utilization, database connection pool stats, and much more — through `/actuator/metrics`.

```
GET /actuator/metrics/http.server.requests
```

```json
{
  "name": "http.server.requests",
  "measurements": [
    { "statistic": "COUNT", "value": 15234 },
    { "statistic": "TOTAL_TIME", "value": 342.5 },
    { "statistic": "MAX", "value": 1.2 }
  ]
}
```

Under the hood, this is powered by **Micrometer**, a vendor-neutral metrics facade (conceptually similar to how SLF4J is a facade for logging) that can export the SAME metrics to many different monitoring backends — Prometheus, Datadog, New Relic, CloudWatch — without changing your application code.

## 21.3 Monitoring

In practice, you rarely stare directly at raw JSON from Actuator endpoints in production — instead, you point a monitoring system at them, which scrapes the data continuously and builds dashboards/alerts on top of it. The most common pattern in the Spring ecosystem is Prometheus + Grafana:

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
```

This exposes `/actuator/prometheus`, formatted exactly the way Prometheus expects to scrape it, letting you build rich Grafana dashboards showing request latency percentiles, error rates, JVM health, and custom business metrics — all without writing any dashboard code yourself, just configuration.

## 21.4 Custom Endpoints

Beyond the built-in endpoints, you can define your OWN custom Actuator endpoints for application-specific operational needs — like exposing cache statistics, feature flag states, or a manual "trigger a data refresh" operation.

```java
@Component
@Endpoint(id = "cacheStats")
public class CacheStatsEndpoint {

    private final CacheManager cacheManager;

    public CacheStatsEndpoint(CacheManager cacheManager) {
        this.cacheManager = cacheManager;
    }

    @ReadOperation
    public Map<String, Object> cacheStats() {
        Map<String, Object> stats = new HashMap<>();
        cacheManager.getCacheNames().forEach(name -> stats.put(name, "active"));
        return stats;
    }
}
```

This becomes available at `/actuator/cacheStats` automatically. A very important production consideration: Actuator endpoints can expose SENSITIVE operational details (environment variables, internal configuration), so you should always secure them properly and only expose the specific endpoints you actually need — never blindly expose everything with `management.endpoints.web.exposure.include=*` in a production environment.

---

# 22. Spring Boot Profiles

## 22.1 Development

The "dev" profile typically configures your application for the fastest, most convenient LOCAL development experience — verbose logging, an easily resettable in-memory or local database, relaxed security, and DevTools enabled.

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:devdb
  jpa:
    hibernate:
      ddl-auto: create-drop
  h2:
    console:
      enabled: true

logging:
  level:
    com.myapp: DEBUG
```

Activated locally with `spring.profiles.active=dev` — often set as a default in your IDE's run configuration, so it's automatically active whenever you run the app from your IDE during development.

## 22.2 Testing

The "test" profile (distinct from actual unit/integration tests, though related) configures settings appropriate for a QA/staging-like testing environment — often pointing to a dedicated test database, with test-specific feature flags or mock integrations for third-party services you don't want to hit for real during testing.

```yaml
# application-test.yml
spring:
  datasource:
    url: jdbc:postgresql://test-db.internal:5432/myapp_test

app:
  payment:
    provider: mock  # use a fake payment provider instead of the real one, even in this shared test environment
```

## 22.3 Production

The "prod" profile is where security, performance, and reliability considerations get tightened to their real, final settings — real database credentials (pulled from secrets management, never hardcoded), minimal logging verbosity, DevTools completely absent, and strict security configuration.

```yaml
# application-prod.yml
spring:
  jpa:
    hibernate:
      ddl-auto: validate  # NEVER auto-create/update schema in production — migrations (Flyway/Liquibase) own that job
  datasource:
    url: ${DATABASE_URL}
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD}

logging:
  level:
    root: WARN
    com.myapp: INFO

management:
  endpoints:
    web:
      exposure:
        include: health,metrics
```

Notice `ddl-auto: validate` instead of `update` or `create-drop` — in production, Hibernate should NEVER be allowed to automatically modify your schema; schema changes should ONLY happen through reviewed, versioned migrations (Flyway/Liquibase), with Hibernate just validating that the entity model matches what's actually there.

## 22.4 Environment Variables

Rather than hardcoding sensitive or environment-specific values directly in your YAML/properties files (which get committed to source control), production configuration typically pulls values from environment variables at runtime — set by whatever deployment platform you're using (Docker, Kubernetes, a cloud provider's console).

```yaml
spring:
  datasource:
    url: ${DATABASE_URL}
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD:defaultDevPassword}  # the part after the colon is a fallback default
```

This means the SAME packaged application artifact can be deployed identically across dev, staging, and production — with ONLY the environment variables differing between them — which is a core tenet of reliable, repeatable deployments.

## 22.5 Secrets Management

Environment variables are a big improvement over hardcoded values, but they're still not ideal for TRULY sensitive secrets (database passwords, API keys, encryption keys) — they can leak through process listings, logs, or crash dumps. Dedicated secrets management systems solve this more robustly.

Common approaches in the Spring ecosystem:

- **Spring Cloud Config Server** — a centralized configuration service that can pull config (including secrets) from a secured Git repo or vault at application startup
- **HashiCorp Vault** (with Spring Cloud Vault) — a dedicated secrets management system with fine-grained access control, secret rotation, and audit logging
- **Cloud-native secret stores** — AWS Secrets Manager, Azure Key Vault, Google Secret Manager — integrated with your cloud provider's IAM permission model

```yaml
spring:
  config:
    import: vault://secret/myapp
```

The core principle across all of these: secrets should NEVER live in your source code or plain configuration files — they should be fetched securely, at runtime, from a system specifically designed to store and control access to sensitive credentials, with proper auditing of who/what accessed them and when.

---
# 23. Docker

## 23.1 Docker Basics

Docker lets you package an application together with EVERYTHING it needs to run (Java runtime, OS-level dependencies, configuration) into a single, portable unit called a "container." The huge benefit: "it works on my machine" stops being a problem, because the container runs identically regardless of what's installed on the underlying host machine.

Key concepts:
- **Image** — a read-only template/blueprint describing what should be inside a container (like a class in OOP)
- **Container** — a running instance of an image (like an object/instance of that class)
- **Dockerfile** — the recipe/instructions for BUILDING an image
- **Registry** — a place to store and share images (like Docker Hub, or a private registry)

```bash
docker build -t myapp:1.0 .
docker run -p 8080:8080 myapp:1.0
```

## 23.2 Dockerfile

A `Dockerfile` is a plain text file containing step-by-step instructions for building your application's Docker image.

```dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY target/myapp-1.0.0.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Line by line: `FROM` picks a base image (here, a lightweight Java 21 runtime); `WORKDIR` sets the working directory inside the container; `COPY` brings your already-built JAR file into the image; `EXPOSE` documents which port the app listens on; `ENTRYPOINT` defines the command that runs when the container starts.

## 23.3 Docker Compose

Real applications rarely run in complete isolation — they need a database, maybe a cache, maybe a message broker. Docker Compose lets you define and run a MULTI-container application (your app + its dependencies) together, as one coordinated unit, using a single YAML file.

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/myapp
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: secret
    depends_on:
      - db
      - redis

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

volumes:
  postgres_data:
```

```bash
docker compose up -d
```

With this single command, your entire local development stack (app, database, cache) starts up together, fully networked and configured — an enormous convenience compared to manually installing and configuring each of these individually on your machine.

## 23.4 Container Networking

By default, Docker Compose automatically creates a private network shared by all services defined in the same compose file, and — importantly — each service can reach the others using its SERVICE NAME as a hostname (notice `jdbc:postgresql://db:5432/...` above, where `db` is just the service name, not an IP address or `localhost`).

This is a common point of confusion for developers new to Docker: from INSIDE a container, `localhost` refers to that container ITSELF, not the host machine or other containers. To reach another service, you use its service name (Docker's internal DNS resolves it automatically), or `host.docker.internal` specifically to reach the actual host machine from inside a container.

## 23.5 Multi-stage Builds

Building a Java application requires the FULL JDK, build tools (Maven/Gradle), and all your source code — but RUNNING the compiled application only needs the much smaller JRE and the final JAR file. Multi-stage builds let you use a full, heavy build environment for compilation, then copy ONLY the final compiled artifact into a much smaller, leaner final image.

```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Run
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/myapp-1.0.0.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

The final image contains ONLY the JRE and your compiled JAR — none of Maven, your source code, or build-time dependencies are present in the final artifact. This results in a dramatically smaller final image (often a fraction of the size), which means faster deployments, less attack surface, and lower storage/bandwidth costs.

---

# 24. Redis

## 24.1 Redis Basics

Redis is an extremely fast, in-memory key-value data store, commonly used for caching, session storage, real-time leaderboards, rate limiting, and as a lightweight message broker (via Pub/Sub). Because everything lives in memory (with optional disk persistence for durability), Redis operations are typically measured in sub-millisecond latency — dramatically faster than a traditional disk-based database for the kinds of simple lookups it's good at.

```bash
docker run -p 6379:6379 redis:7-alpine
```

Redis supports rich data structures beyond simple key-value pairs — strings, lists, sets, sorted sets, hashes, and more — each with its own specialized operations, making it far more versatile than a plain cache.

## 24.2 Spring Data Redis

`spring-boot-starter-data-redis` provides Spring's abstraction over Redis, giving you a `RedisTemplate` for direct, flexible operations, plus repository support similar in spirit to Spring Data JPA.

```java
@Service
public class SessionTokenService {

    private final RedisTemplate<String, String> redisTemplate;

    public SessionTokenService(RedisTemplate<String, String> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public void storeToken(String userId, String token) {
        redisTemplate.opsForValue().set("session:" + userId, token, Duration.ofHours(1));
    }

    public String getToken(String userId) {
        return redisTemplate.opsForValue().get("session:" + userId);
    }
}
```

For richer object mapping (treating Redis more like a document store), Spring Data Redis also offers `@RedisHash`-annotated entities with a full repository interface, similar to JPA:

```java
@RedisHash("User")
public class UserSession {
    @Id
    private String id;
    private String username;
    @TimeToLive
    private Long ttl;
}

public interface UserSessionRepository extends CrudRepository<UserSession, String> { }
```

## 24.3 Cache

We already covered Redis as a `CacheManager` backend in the Caching section — worth reiterating here in context: Redis is BY FAR the most common choice for a DISTRIBUTED, shared cache in Spring Boot microservices architectures, precisely because multiple application instances need to see the SAME cached data, which a local in-memory cache can't provide.

## 24.4 Session Store

For applications running MULTIPLE server instances behind a load balancer, storing user sessions in each server's local memory is a problem — if a user's requests get routed to a DIFFERENT instance than the one holding their session, they'd appear logged out. Redis solves this by acting as a SHARED, centralized session store all instances can read/write to.

```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

```properties
spring.session.store-type=redis
```

With this configuration, Spring Session transparently intercepts normal `HttpSession` usage and stores it in Redis instead of local server memory — your controller code doesn't change at all, but now ANY server instance can serve ANY user's request and correctly find their session data, since it's centrally stored rather than tied to one specific server.

---

# 25. Kafka

## 25.1 Kafka Basics

Apache Kafka is a distributed event streaming platform, commonly used for building event-driven architectures and reliably passing large volumes of messages between services. Unlike a traditional message queue, Kafka is built around a durable, append-only log — messages aren't deleted immediately after being read, they're RETAINED for a configurable period, meaning MULTIPLE independent consumers can read (and re-read) the same stream of events.

Core concepts:
- **Topic** — a named category/stream that messages are published to (like `order-events`)
- **Producer** — an application that writes/publishes messages to a topic
- **Consumer** — an application that reads/subscribes to messages from a topic
- **Broker** — a Kafka server that stores and serves data (a Kafka cluster consists of multiple brokers)

## 25.2 Producer

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```java
@Service
public class OrderEventProducer {

    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;

    public OrderEventProducer(KafkaTemplate<String, OrderEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void publishOrderCreated(Order order) {
        OrderEvent event = new OrderEvent(order.getId(), "ORDER_CREATED", Instant.now());
        kafkaTemplate.send("order-events", order.getId().toString(), event);
    }
}
```

Notice the message is sent with a KEY (`order.getId().toString()`) — Kafka uses this key to decide which PARTITION (covered below) the message goes to, and messages with the SAME key always land on the SAME partition, which guarantees they're processed in order relative to each other.

## 25.3 Consumer

```java
@Service
public class OrderEventConsumer {

    @KafkaListener(topics = "order-events", groupId = "notification-service")
    public void handleOrderEvent(OrderEvent event) {
        System.out.println("Received event: " + event.getEventType() + " for order " + event.getOrderId());
        if (event.getEventType().equals("ORDER_CREATED")) {
            notificationService.sendOrderConfirmation(event.getOrderId());
        }
    }
}
```

`@KafkaListener` marks a method as a consumer for a given topic — Spring Boot handles all the underlying polling, deserialization, and thread management for you automatically.

## 25.4 Consumer Groups

A "consumer group" is a set of consumers that COOPERATE to consume a topic — Kafka automatically divides the topic's partitions among the consumers within a group, so each message is processed by exactly ONE consumer WITHIN that group. This is how Kafka achieves horizontal scaling of message processing: add more consumer instances to the same group, and Kafka automatically rebalances partition assignments across them.

Importantly, DIFFERENT consumer groups are completely independent of each other — if both a `notification-service` group AND an `analytics-service` group subscribe to the SAME topic, EACH group gets its OWN full copy of every message, processed independently. This is what allows Kafka to support genuinely different downstream systems, each consuming the same stream of events for entirely different purposes.

## 25.5 Partitions

A topic is physically divided into one or more "partitions" — each partition is an independent, ordered, append-only log. Partitioning is Kafka's mechanism for both PARALLELISM (different partitions can be processed by different consumers simultaneously) and SCALABILITY (partitions can be spread across multiple brokers in a cluster).

Important guarantee: Kafka only guarantees ORDERING of messages WITHIN a single partition — NOT across the entire topic. This is exactly why the choice of partition KEY matters so much (as mentioned in the Producer section) — if you need all events for a given order to be processed strictly in order, they must all use the same key (like the order ID), ensuring they always land on the same partition.

## 25.6 Offset

Each message within a partition has a sequential ID called an "offset" — essentially, its position within that partition's log. Consumers track which offset they've read up to, letting Kafka know exactly where to resume if a consumer restarts or crashes.

```properties
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.enable-auto-commit=false
```

`auto-offset-reset=earliest` tells a brand-new consumer group (one that's never consumed this topic before) to start reading from the very BEGINNING of the topic's retained history, rather than only new messages from now on. Disabling auto-commit (and committing offsets manually, after successfully processing a message) gives you much stronger control over exactly when a message is considered "done" — important for building reliable, at-least-once processing guarantees.

## 25.7 Retry

When message processing fails (a downstream service is temporarily down, a transient database error occurs), you generally don't want to just drop the message — you want to retry it, ideally with some backoff between attempts.

```java
@RetryableTopic(
    attempts = "3",
    backoff = @Backoff(delay = 1000, multiplier = 2.0),
    dltStrategy = DltStrategy.FAIL_ON_ERROR
)
@KafkaListener(topics = "order-events", groupId = "notification-service")
public void handleOrderEvent(OrderEvent event) {
    // if this throws, Spring Kafka automatically retries with increasing backoff
}
```

## 25.8 Dead Letter Queue

If a message fails EVERY retry attempt, you don't want it to be silently lost, nor do you want it to block the entire partition forever. A **Dead Letter Queue (DLQ)** — really just another Kafka topic — is where permanently-failed messages get routed instead, letting the main flow continue while giving you a place to inspect, alert on, and potentially manually reprocess these failures later.

```java
@RetryableTopic(attempts = "3", dltTopicSuffix = "-dlt")
@KafkaListener(topics = "order-events", groupId = "notification-service")
public void handleOrderEvent(OrderEvent event) { }

@DltHandler
public void handleDlt(OrderEvent event) {
    log.error("Message permanently failed after retries: {}", event);
    alertingService.notifyOpsTeam(event);
}
```

## 25.9 Transactions

Kafka supports transactions to guarantee that a group of operations (producing multiple messages, or producing a message AND committing a consumer offset together) either ALL succeed or ALL fail — critical for building genuinely reliable event-driven pipelines that shouldn't produce partial/inconsistent results.

```java
@Transactional("kafkaTransactionManager")
public void processAndForward(OrderEvent inputEvent) {
    // process the event, then produce a NEW event as a result —
    // both the original consumption AND the new production
    // succeed or fail together as one atomic unit
    kafkaTemplate.send("shipment-events", buildShipmentEvent(inputEvent));
}
```

## 25.10 Exactly Once

By default, Kafka (like most distributed messaging systems) guarantees "at-least-once" delivery — meaning a message might occasionally be delivered/processed MORE than once (e.g., if a consumer crashes after processing but before committing its offset). "Exactly-once semantics" (EOS) is a stronger, harder-to-achieve guarantee that Kafka CAN provide, ensuring each message is processed and its effects applied exactly one time, even across failures.

```properties
spring.kafka.producer.transaction-id-prefix=order-service-
spring.kafka.consumer.isolation-level=read_committed
```

Achieving true exactly-once semantics requires careful coordination across producers, transactions, and idempotent consumers — it's powerful but adds real complexity, so many systems deliberately choose the simpler "at-least-once + idempotent processing logic" pattern instead (designing your consumer so that processing the same message twice has no harmful side effect), which is often easier to reason about in practice.

---

# 26. RabbitMQ

## 26.1 Exchanges

RabbitMQ is a traditional message broker (as opposed to Kafka's distributed log model), built around the AMQP protocol. Messages published by a producer don't go DIRECTLY to a queue — they go to an **exchange** first, which is responsible for ROUTING the message to one or more queues based on rules.

```java
@Bean
public DirectExchange orderExchange() {
    return new DirectExchange("order-exchange");
}
```

Exchange types:
- **Direct** — routes based on an exact routing key match
- **Topic** — routes based on a pattern match against the routing key (e.g., `order.*.created`)
- **Fanout** — broadcasts to ALL bound queues, ignoring the routing key entirely
- **Headers** — routes based on message header values instead of the routing key

## 26.2 Queues

A queue is where messages actually WAIT to be consumed — think of it as a literal, first-in-first-out (FIFO) line.

```java
@Bean
public Queue orderQueue() {
    return QueueBuilder.durable("order-queue").build();
}
```

`durable("order-queue")` means the queue survives a broker restart (as opposed to a purely in-memory, transient queue that would be lost on restart) — important for any queue holding messages you genuinely can't afford to lose.

## 26.3 Routing Keys

The **routing key** is a label attached to a published message, used by the exchange to decide which queue(s) it should be delivered to, based on how queues are BOUND to that exchange.

```java
@Bean
public Binding binding(Queue orderQueue, DirectExchange orderExchange) {
    return BindingBuilder.bind(orderQueue).to(orderExchange).with("order.created");
}
```

```java
rabbitTemplate.convertAndSend("order-exchange", "order.created", orderEvent);
```

This message will be routed to `orderQueue`, because it was bound to `orderExchange` specifically for the routing key `"order.created"` — a DIFFERENT routing key (like `"order.cancelled"`) would need its own separate binding to reach a (potentially different) queue.

## 26.4 Acknowledgements

When a consumer receives a message, RabbitMQ needs to know whether it was successfully processed before permanently removing it from the queue — this is what "acknowledgements" (acks) are for.

```java
@RabbitListener(queues = "order-queue", ackMode = "MANUAL")
public void handleOrderMessage(OrderEvent event, Channel channel, @Header(AmqpHeaders.DELIVERY_TAG) long tag) throws IOException {
    try {
        processOrder(event);
        channel.basicAck(tag, false); // explicitly confirm successful processing
    } catch (Exception e) {
        channel.basicNack(tag, false, true); // reject and requeue for another attempt
    }
}
```

With MANUAL acknowledgement mode, if your consumer crashes BEFORE explicitly acknowledging a message, RabbitMQ will redeliver that message to another consumer — ensuring messages aren't silently lost just because a consumer happened to die mid-processing. The default AUTO mode acknowledges automatically as long as the listener method doesn't throw an exception, which is simpler but gives you less fine-grained control.

## 26.5 Retry

Similar to Kafka, transient failures during message processing generally warrant a retry rather than immediately giving up. Spring AMQP supports this through Spring Retry integration.

```java
@Bean
public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    RabbitTemplate template = new RabbitTemplate(connectionFactory);
    template.setRetryTemplate(RetryTemplate.builder()
        .maxAttempts(3)
        .exponentialBackoff(1000, 2.0, 10000)
        .build());
    return template;
}
```

For CONSUMER-side retry (handling failures when actually processing a received message), you typically configure a `RetryInterceptor` combined with a dead-letter exchange for messages that exhaust all retry attempts — conceptually identical to Kafka's Dead Letter Queue pattern covered earlier, just implemented through RabbitMQ's own exchange/queue routing mechanism instead.

---
# 27. Microservices

## 27.1 Microservices Architecture

A "monolith" is one big application containing all your business functionality, deployed as a single unit. **Microservices** architecture instead breaks that same functionality into many SMALL, independently deployable services, each typically owning its own specific piece of business capability (e.g., an `order-service`, a `payment-service`, a `notification-service`), communicating with each other over the network.

Benefits:
- Teams can develop, deploy, and scale services independently
- A failure in one service doesn't necessarily bring down the whole system
- Different services can use different technologies best suited to their specific needs
- Smaller codebases are easier to understand and reason about

Real costs (and these are genuine, not imaginary):
- Distributed systems are inherently more complex — network calls can fail in ways local method calls can't
- Debugging a request across many services is much harder than debugging one process
- You need real infrastructure investment: service discovery, centralized logging, distributed tracing, API gateways

Microservices are a genuine tradeoff, not an automatic upgrade — many successful, high-scale companies run well-designed monoliths for a very long time before (if ever) needing to split into microservices, and premature splitting is a very common and costly mistake.

## 27.2 Service Discovery

In a microservices world, service instances come and go constantly (scaling up/down, deployments, failures) — hardcoding IP addresses/hostnames for every service your application needs to call is impractical. **Service discovery** solves this: services REGISTER themselves with a central registry on startup, and other services LOOK UP where to find them dynamically, at runtime.

## 27.3 Eureka

Eureka (from Netflix, integrated via Spring Cloud Netflix) is a popular service discovery/registry implementation in the Spring ecosystem.

```java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServerApplication.class, args);
    }
}
```

Each individual microservice then registers itself with this Eureka server:

```yaml
spring:
  application:
    name: order-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
```

Once registered, other services can look up `order-service` BY NAME (instead of a hardcoded URL), and Eureka resolves it to an actual, currently-healthy instance address — automatically handling instances that come online, go offline, or move to different addresses.

## 27.4 API Gateway

Rather than having every client (mobile app, web frontend, third-party integrator) talk DIRECTLY to dozens of individual microservices, an **API Gateway** sits in front of everything as a single entry point — routing incoming requests to the appropriate backend service, and often handling cross-cutting concerns (authentication, rate limiting, request logging) in ONE centralized place instead of duplicating that logic across every individual service.

```yaml
# Spring Cloud Gateway configuration
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
        - id: payment-service
          uri: lb://payment-service
          predicates:
            - Path=/api/payments/**
```

`lb://order-service` tells the gateway to load-balance requests across all currently registered, healthy instances of `order-service` (resolved dynamically via service discovery), rather than pointing at one fixed address.

## 27.5 Config Server

Managing configuration for dozens of individual microservices individually is painful and error-prone. **Spring Cloud Config Server** centralizes configuration for ALL your services in one place (typically backed by a Git repository), which services pull from at startup.

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

Individual services then simply point at the config server instead of maintaining their own local `application.yml` with all the actual values:

```yaml
spring:
  config:
    import: "configserver:http://localhost:8888"
```

This gives you a single, auditable, versioned source of truth for configuration across your entire microservices fleet — and lets you update SHARED configuration values in one place, rather than editing dozens of individual service configs one by one.

## 27.6 OpenFeign

Calling another microservice's REST API involves a fair amount of repetitive boilerplate if done manually with `RestTemplate`/`WebClient` — building URLs, handling serialization, error handling. **OpenFeign** lets you define a service client as a simple, declarative Java INTERFACE, and Spring generates the actual implementation for you.

```java
@FeignClient(name = "payment-service")
public interface PaymentServiceClient {

    @PostMapping("/api/payments")
    PaymentResponse charge(@RequestBody ChargeRequest request);

    @GetMapping("/api/payments/{id}")
    PaymentResponse getPayment(@PathVariable Long id);
}
```

```java
@Service
public class OrderService {

    private final PaymentServiceClient paymentClient;

    public void placeOrder(OrderRequest request) {
        PaymentResponse payment = paymentClient.charge(new ChargeRequest(request.getAmount()));
        // ...
    }
}
```

Notice: no HTTP client code, no manual JSON handling — just an interface describing the remote API, and Spring (combined with service discovery, resolving `payment-service` by name) handles the rest, making inter-service calls feel almost like calling a local method.

## 27.7 Circuit Breaker

In a distributed system, a downstream service being slow or completely down is not a matter of "if," but "when." Without protection, a struggling downstream service can cause a CASCADING failure — callers pile up waiting on slow/failing calls, exhaust their own resources (threads, connections), and become unhealthy themselves, spreading the failure upstream. A **circuit breaker** prevents this by "opening" (failing fast, without even attempting the call) once a service has failed too many times recently, giving it time to recover.

Think of it exactly like an electrical circuit breaker in your house: when there's a dangerous overload, it "trips" and cuts power immediately, rather than letting the overload keep damaging things.

## 27.8 Resilience4j

Resilience4j is the modern, standard library for implementing circuit breakers (and related resilience patterns) in Spring Boot applications.

```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "chargeFailFallback")
public PaymentResponse charge(ChargeRequest request) {
    return paymentClient.charge(request);
}

public PaymentResponse chargeFailFallback(ChargeRequest request, Throwable throwable) {
    // called automatically when the circuit is OPEN, or the real call failed
    return PaymentResponse.pending("Payment service temporarily unavailable, will retry");
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 30s
```

This configuration says: look at the last 10 calls; if 50% or more failed, "open" the circuit (stop even attempting real calls, immediately using the fallback instead) for 30 seconds, then cautiously try a few real calls again to see if the downstream service has recovered. Resilience4j also provides related patterns in the same library: **retry** (automatic re-attempts), **rate limiter** (cap outgoing call rate), **bulkhead** (limit concurrent calls to isolate failures), and **time limiter** (enforce timeouts).

## 27.9 Distributed Tracing

When a single user request flows through MULTIPLE microservices (gateway → order-service → payment-service → notification-service), debugging "why was this slow?" or "where did this fail?" becomes very difficult without a way to see the FULL picture across all of them. Distributed tracing solves this by assigning a unique TRACE ID to each incoming request, propagated automatically through every downstream call, letting you reconstruct the complete request path afterward.

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
```

Tools like Zipkin or Jaeger collect and visualize these traces, showing you a timeline: which service was called, how long each step took, and exactly where in the chain a failure or slowdown occurred — dramatically speeding up debugging in a distributed system compared to manually correlating logs across many separate services.

## 27.10 Centralized Logging

Similarly, with logs scattered across dozens of individual service instances, manually SSH-ing into each one to grep for relevant log lines is completely impractical at any real scale. **Centralized logging** aggregates logs from EVERY service instance into one searchable, unified system.

The most common stack in the Spring ecosystem: the **ELK stack** (Elasticsearch, Logstash, Kibana) or its lighter cousin (**EFK**, using Fluentd/Fluent Bit instead of Logstash) — each service ships its logs (ideally structured, as covered in the Logging section) to a central collector, which indexes them in Elasticsearch, searchable and visualizable through Kibana.

Combined with the request ID / trace ID propagation from distributed tracing, you can search centralized logs for one specific trace ID and instantly see EVERY log line, from EVERY service, related to that one specific request — an enormously powerful debugging capability in a microservices system.

## 27.11 Service Communication

Microservices need to talk to each other, and there are two fundamentally different communication styles, each suited to different situations:

**Synchronous (request-response)** — Service A calls Service B directly (via REST/OpenFeign, or gRPC) and WAITS for a response before continuing. Simple to reason about, but creates tight runtime coupling — if B is down, A's request fails immediately.

```java
PaymentResponse response = paymentClient.charge(request); // A waits here for B to respond
```

**Asynchronous (event-driven)** — Service A publishes an EVENT (via Kafka or RabbitMQ, covered earlier) and moves on immediately, without waiting for anyone. Service B (and potentially other services too) consumes that event WHENEVER it's ready. This decouples services in TIME — B doesn't even need to be running at the exact moment A publishes the event, it'll process it whenever it comes back online.

```java
kafkaTemplate.send("order-events", orderCreatedEvent); // A doesn't wait for anyone to consume this
```

Most real-world microservices systems use a THOUGHTFUL MIX of both styles — synchronous calls for things that genuinely need an immediate answer (like checking real-time inventory before confirming a purchase), and asynchronous events for things that can happen eventually, decoupled from the triggering request (like sending a confirmation email, updating analytics, or triggering a downstream workflow).

---
# 28. Cloud Deployment

## 28.1 AWS EC2

EC2 (Elastic Compute Cloud) gives you virtual machines in AWS's cloud — essentially renting a computer, with full control over the operating system, that you configure and manage yourself.

Deploying a Spring Boot app to EC2 typically looks like: provision an instance, install a Java runtime, copy your built JAR (or pull it via CI/CD), and run it (often behind a process manager like `systemd` so it restarts automatically if it crashes, and survives instance reboots).

```bash
# A minimal systemd service definition for a Spring Boot JAR
[Unit]
Description=My Spring Boot App
After=network.target

[Service]
User=appuser
ExecStart=/usr/bin/java -jar /opt/myapp/app.jar
SuccessExitStatus=143
Restart=always

[Install]
WantedBy=multi-user.target
```

EC2 gives you maximum control, but also maximum operational responsibility — YOU are responsible for OS patching, scaling, load balancing setup, and failure recovery, unlike more managed options covered below.

## 28.2 AWS RDS

RDS (Relational Database Service) is AWS's MANAGED database offering — supporting PostgreSQL, MySQL, and others — where AWS handles the operational burden of running a database: automated backups, patching, replication, and failover.

```properties
spring.datasource.url=jdbc:postgresql://myapp-db.abc123.us-east-1.rds.amazonaws.com:5432/myapp
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

From your Spring Boot application's perspective, connecting to RDS looks EXACTLY like connecting to any other PostgreSQL/MySQL database — the "managed" benefits are entirely operational (AWS handling backups, patching, high availability), not something your application code needs to be aware of at all.

## 28.3 AWS S3

S3 (Simple Storage Service) is AWS's object storage service — as covered in the File Upload section, this is the standard place to store uploaded files, static assets, backups, and generally any large binary data that doesn't belong in a relational database.

```java
S3Client s3Client = S3Client.builder().region(Region.US_EAST_1).build();

s3Client.putObject(
    PutObjectRequest.builder().bucket("my-app-uploads").key(fileName).build(),
    RequestBody.fromFile(file)
);
```

S3 is also commonly used to host Spring Boot's build ARTIFACTS themselves as part of a deployment pipeline (storing the built JAR before it's deployed to EC2/ECS), and to serve static frontend assets, often paired with a CDN (CloudFront) in front of it.

## 28.4 IAM

IAM (Identity and Access Management) is AWS's system for controlling WHO (or WHAT — including your own application) can access WHICH AWS resources, and what they're allowed to do with them. For a Spring Boot application running on AWS, this typically means assigning your EC2 instance (or ECS task, or Lambda function) an IAM ROLE with precisely the permissions it needs — and nothing more.

```json
{
  "Effect": "Allow",
  "Action": ["s3:PutObject", "s3:GetObject"],
  "Resource": "arn:aws:s3:::my-app-uploads/*"
}
```

The critical security principle here is "least privilege" — your application should be granted ONLY the specific permissions it genuinely needs (e.g., write access to ONE specific S3 bucket), never broad, catch-all permissions "just in case" — this way, if the application is ever compromised, the potential damage is strictly limited to what it was actually authorized to touch.

## 28.5 Elastic Beanstalk

Elastic Beanstalk is AWS's PLATFORM-AS-A-SERVICE offering specifically designed to simplify deployment — you upload your application (a JAR file works directly for Spring Boot), and Beanstalk automatically handles provisioning the underlying EC2 instances, load balancer, auto-scaling, and health monitoring, without you needing to configure any of that infrastructure manually.

```bash
eb init -p java-21 my-spring-app
eb create my-environment
eb deploy
```

This sits at a middle point on the "control vs convenience" spectrum: much less manual setup than raw EC2, but still gives you meaningful control (you can access the underlying EC2 instances if truly needed) — a very popular choice for teams who want managed infrastructure but aren't ready for full container orchestration.

## 28.6 ECS

ECS (Elastic Container Service) is AWS's container orchestration service — designed specifically for running DOCKERIZED applications (recall the Docker section earlier) at scale, handling scheduling containers across a cluster of machines, health checks, auto-scaling, and rolling deployments.

```json
{
  "family": "order-service",
  "containerDefinitions": [{
    "name": "order-service",
    "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/order-service:latest",
    "portMappings": [{ "containerPort": 8080 }],
    "memory": 512,
    "cpu": 256
  }]
}
```

For teams already using microservices architecture (as covered earlier), ECS (or its more portable cousin, Kubernetes) is typically the natural fit — it's built specifically around running MANY independent, containerized services efficiently, with sophisticated scheduling, scaling, and networking capabilities baked in.

## 28.7 Docker Deployment

Regardless of the specific target (ECS, Kubernetes, or a simpler platform like Render or Railway), the general Docker-based deployment flow for a Spring Boot application looks broadly similar:

1. Build your application (`mvn clean package`)
2. Build a Docker image (using a multi-stage `Dockerfile`, as covered earlier)
3. Push the image to a container registry (Docker Hub, AWS ECR, Google Artifact Registry)
4. Deploy that image to your target platform, which pulls it and runs it as a container

```bash
docker build -t myregistry/order-service:1.0 .
docker push myregistry/order-service:1.0
```

This flow is essentially universal across cloud providers and platforms, which is a big part of Docker's appeal — the same containerized artifact can run virtually anywhere that supports containers, minimizing platform-specific lock-in.

## 28.8 CI/CD Basics

CI/CD (Continuous Integration / Continuous Deployment) automates the entire path from "code committed" to "code running in production," removing manual, error-prone deployment steps.

- **Continuous Integration** — every code change is automatically built and tested, catching integration issues immediately rather than discovering them much later
- **Continuous Deployment/Delivery** — successfully-tested changes are automatically (or with a manual approval gate) deployed to production

A typical GitHub Actions pipeline for a Spring Boot app:

```yaml
name: CI/CD
on: [push]
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
      - run: mvn clean test
      - run: mvn clean package -DskipTests
      - run: docker build -t myregistry/myapp:${{ github.sha }} .
      - run: docker push myregistry/myapp:${{ github.sha }}
      - run: aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment
```

Every push automatically runs tests, builds the app, builds and pushes a Docker image, and triggers a fresh deployment — meaning deployments become routine, low-risk, repeatable events, rather than nerve-wracking manual procedures performed occasionally by a specific person who "knows how to deploy."

---

# 29. Performance Optimization

## 29.1 Connection Pooling

Opening a new database connection is a genuinely expensive operation (network handshake, authentication, resource allocation on the database server) — far too expensive to do for every single query. **Connection pooling** solves this by maintaining a pool of already-open, reusable connections, handed out to your application code as needed and returned to the pool when done, rather than opened and closed repeatedly.

```properties
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
```

Sizing your connection pool correctly matters a lot — too small, and requests queue up waiting for an available connection under load; too large, and you can overwhelm your database server with more concurrent connections than it can efficiently handle (each connection consumes real memory/resources on the database side too).

## 29.2 HikariCP

HikariCP is the connection pool implementation Spring Boot uses BY DEFAULT (since Spring Boot 2.0) — and it's widely regarded as the fastest, most reliable JDBC connection pool available in the Java ecosystem, which is exactly why it became the default choice.

```properties
spring.datasource.hikari.pool-name=MyAppPool
spring.datasource.hikari.max-lifetime=1800000
spring.datasource.hikari.leak-detection-threshold=60000
```

`leak-detection-threshold` is a particularly useful setting for catching bugs — if a connection is checked out from the pool and NOT returned within the configured time (suggesting your code forgot to close it somewhere), HikariCP logs a warning with a stack trace pointing to where the connection was originally acquired, making "connection leak" bugs (a classic, hard-to-diagnose class of bug) much easier to track down.

## 29.3 Query Optimization

Slow database queries are one of the most common real-world performance bottlenecks in any application. Common query optimization techniques:

- Avoid `SELECT *` — fetch only the columns you actually need
- Avoid the N+1 problem (covered in depth earlier) by using `JOIN FETCH` or `@EntityGraph` appropriately
- Use pagination for any query that could return a large result set
- Push filtering/aggregation down to the DATABASE (via `WHERE`/`GROUP BY`) rather than fetching everything and filtering in Java code
- Use `EXPLAIN ANALYZE` (PostgreSQL) or equivalent to understand exactly how the database is executing your query, and whether it's using indexes effectively

```java
// Bad: fetches ALL orders, then filters in application memory
List<Order> allOrders = orderRepository.findAll();
List<Order> shipped = allOrders.stream().filter(o -> o.getStatus() == SHIPPED).toList();

// Good: filtering happens at the database level
List<Order> shipped = orderRepository.findByStatus(OrderStatus.SHIPPED);
```

## 29.4 Indexing

A database index is a separate data structure that allows the database to find rows matching a condition WITHOUT scanning every single row in the table — conceptually similar to an index at the back of a book, letting you jump directly to relevant pages instead of reading the entire book cover to cover.

```java
@Entity
@Table(name = "orders", indexes = {
    @Index(name = "idx_customer_id", columnList = "customer_id"),
    @Index(name = "idx_status_created", columnList = "status, created_at")
})
public class Order { }
```

Indexes dramatically speed up READS on the indexed columns, but they come with a real tradeoff: every index adds overhead to WRITES (inserts/updates need to update every relevant index too), and consumes additional disk space. The general guideline: index columns you frequently filter (`WHERE`), sort (`ORDER BY`), or join (`JOIN ... ON`) on — but avoid over-indexing tables that are written to far more often than they're read.

## 29.5 Batch Inserts

Already touched on in the Hibernate section, worth reinforcing here as a dedicated performance topic: inserting many records one-at-a-time (one round trip per record) is dramatically slower than batching them together into fewer round trips.

```properties
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb?reWriteBatchedInserts=true
```

Note the `reWriteBatchedInserts=true` PostgreSQL JDBC driver parameter — without it, even with Hibernate's batch size configured, the driver may still send individual `INSERT` statements to the database rather than a genuinely combined batch; this setting is necessary to get the full performance benefit with PostgreSQL specifically.

## 29.6 Batch Updates

The same batching principle applies to UPDATES — if you need to update many rows based on application logic (rather than a single `UPDATE ... WHERE` statement that the database can execute in one shot), batching those updates avoids the overhead of many individual round trips.

```java
@Transactional
public void bulkUpdateStatus(List<Long> orderIds, OrderStatus newStatus) {
    int batchSize = 50;
    for (int i = 0; i < orderIds.size(); i++) {
        Order order = entityManager.find(Order.class, orderIds.get(i));
        order.setStatus(newStatus);
        if (i % batchSize == 0) {
            entityManager.flush();
            entityManager.clear();
        }
    }
}
```

For simple bulk updates that don't need entity-level logic, a direct JPQL bulk update is often even faster, since it bypasses loading entities into memory entirely:

```java
@Modifying
@Query("UPDATE Order o SET o.status = :status WHERE o.id IN :ids")
void bulkUpdateStatus(@Param("ids") List<Long> ids, @Param("status") OrderStatus status);
```

## 29.7 Profiling

When you genuinely don't know WHERE your performance problem is coming from, guessing is a waste of time — profiling tools give you concrete, measured evidence of where time and resources are actually being spent.

Common tools in the Spring Boot ecosystem:
- **JProfiler / YourKit** — full-featured commercial Java profilers, showing CPU hotspots, memory allocation, thread activity
- **Java Flight Recorder (JFR)** — built directly into the JDK, low-overhead, production-safe profiling
- **Spring Boot Actuator + Micrometer** (covered earlier) — application-level metrics showing which endpoints/methods are slow
- **Database-specific tools** (like PostgreSQL's `pg_stat_statements`) — showing which QUERIES specifically are consuming the most database time

The core discipline: measure first, THEN optimize — optimizing code that "feels slow" without actual measurement frequently wastes effort on something that wasn't really the bottleneck at all, while the REAL bottleneck goes unaddressed.

## 29.8 JVM Memory Tuning

Spring Boot applications run on the JVM, and understanding basic JVM memory configuration matters for production stability — an under-provisioned JVM can suffer from frequent, disruptive garbage collection pauses, or crash entirely with `OutOfMemoryError`.

```bash
java -Xms512m -Xmx2g -XX:+UseG1GC -jar myapp.jar
```

- `-Xms` — initial heap size
- `-Xmx` — maximum heap size (the JVM will never use more memory than this for the heap)
- `-XX:+UseG1GC` — selects the G1 garbage collector, a good general-purpose default for most modern Spring Boot applications

A common production mistake: NOT setting `-Xmx` explicitly, and letting the JVM's default heuristics decide — especially risky in containerized environments (Docker/Kubernetes), where the JVM needs to be correctly aware of the CONTAINER's memory limit (not the host machine's), or it may try to use more memory than the container is allowed, getting forcibly killed (`OOMKilled`) by the container runtime.

---
# 30. Production Best Practices

## 30.1 Layered Architecture

Most Spring Boot applications organize code into distinct LAYERS, each with a clear, single responsibility, and each layer only talking to the layer immediately below it:

```
Controller Layer   (handles HTTP, request/response mapping)
        |
Service Layer      (business logic, orchestration, transactions)
        |
Repository Layer   (data access)
        |
Database
```

```java
@RestController                       // Controller layer
public class OrderController {
    private final OrderService orderService;
    // delegates to the service layer, contains NO business logic itself
}

@Service                               // Service layer
public class OrderService {
    private final OrderRepository orderRepository;
    // contains the actual business rules and orchestration
}

public interface OrderRepository extends JpaRepository<Order, Long> { }  // Repository layer
```

This separation matters because it keeps each layer focused and testable in isolation: controllers stay thin (just HTTP concerns), business logic stays centralized in services (not scattered across controllers), and data access stays cleanly separated from business rules — making the whole codebase far easier to navigate, test, and modify safely over time.

## 30.2 Clean Code

"Clean code" is a broad set of practices aimed at making code easy for OTHER humans (including future-you) to read, understand, and safely modify — not just for the compiler to execute correctly.

Practical clean code habits in a Spring Boot context:
- Meaningful, intention-revealing names (`calculateShippingCost()`, not `calc()`)
- Small, focused methods that do ONE thing
- Avoid deeply nested conditionals — prefer early returns/guard clauses
- Keep controllers thin — they should orchestrate, not implement business logic
- Consistent formatting and structure across the codebase (enforced with tools like Checkstyle or Spotless)

```java
// Less clean
public void proc(Order o) {
    if (o != null) {
        if (o.getStatus() == OrderStatus.PENDING) {
            if (o.getItems().size() > 0) {
                // deeply nested logic
            }
        }
    }
}

// Cleaner — guard clauses flatten the structure
public void processOrder(Order order) {
    if (order == null) return;
    if (order.getStatus() != OrderStatus.PENDING) return;
    if (order.getItems().isEmpty()) return;
    // main logic, now unindented and clear
}
```

## 30.3 SOLID Principles

SOLID is a set of five foundational object-oriented design principles that lead naturally to more maintainable, flexible code — and Spring's own design (dependency injection, interfaces everywhere) actively encourages following them.

- **S — Single Responsibility:** A class should have ONE reason to change. `OrderService` shouldn't also handle email sending — that belongs in a dedicated `EmailService`.
- **O — Open/Closed:** Code should be open for extension, but closed for modification. Using interfaces (like `PaymentService`) lets you ADD new payment providers without modifying existing code.
- **L — Liskov Substitution:** Any subclass/implementation should be usable anywhere its parent type/interface is expected, without breaking behavior.
- **I — Interface Segregation:** Prefer several small, focused interfaces over one giant, do-everything interface.
- **D — Dependency Inversion:** Depend on ABSTRACTIONS (interfaces), not concrete implementations — exactly what constructor-injected interface dependencies in Spring naturally encourage.

```java
public interface PaymentService {
    PaymentResult charge(ChargeRequest request);
}

@Service
public class StripePaymentService implements PaymentService { }

@Service
public class PayPalPaymentService implements PaymentService { }

@Service
public class OrderService {
    private final PaymentService paymentService; // depends on the ABSTRACTION, not a specific provider
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

## 30.4 DTO Mapping

As covered in the REST section, DTOs deliberately keep your API contract separate from your internal entity model. But manually writing conversion code between entities and DTOs for every single field, in every direction, gets extremely repetitive as an application grows.

```java
// Manual mapping — tedious and error-prone at scale
public OrderResponse toResponse(Order order) {
    OrderResponse response = new OrderResponse();
    response.setId(order.getId());
    response.setCustomerName(order.getCustomer().getName());
    response.setTotal(order.getTotalAmount());
    return response;
}
```

## 30.5 MapStruct

MapStruct is a compile-time code generator that eliminates the tedium of manual DTO mapping — you declare a mapping INTERFACE, and MapStruct generates a fully-implemented, highly efficient mapping class for you AT BUILD TIME (not via slow runtime reflection, unlike some other mapping libraries).

```java
@Mapper(componentModel = "spring")
public interface OrderMapper {
    @Mapping(source = "customer.name", target = "customerName")
    OrderResponse toResponse(Order order);

    List<OrderResponse> toResponseList(List<Order> orders);
}
```

```java
@Service
public class OrderService {
    private final OrderMapper orderMapper; // Spring injects the MapStruct-generated implementation directly

    public OrderResponse getOrder(Long id) {
        Order order = orderRepository.findById(id).orElseThrow();
        return orderMapper.toResponse(order);
    }
}
```

Because the mapping code is GENERATED at compile time (you can literally view the generated `.java` file in your build output), it's both extremely fast at runtime AND fully type-safe — mapping mistakes get caught as compile errors, not silent runtime bugs.

## 30.6 Global Response Structure

Rather than every endpoint returning a differently-shaped raw response, many production APIs wrap EVERY response in a consistent, predictable envelope — making client-side handling simpler and more uniform.

```java
public class ApiResponse<T> {
    private boolean success;
    private T data;
    private String message;
    private Instant timestamp = Instant.now();

    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(true, data, null);
    }

    public static <T> ApiResponse<T> error(String message) {
        return new ApiResponse<>(false, null, message);
    }
}
```

```java
@GetMapping("/{id}")
public ApiResponse<OrderResponse> getOrder(@PathVariable Long id) {
    return ApiResponse.success(orderMapper.toResponse(orderService.getOrder(id)));
}
```

Now every single response from the API — success OR error — follows the exact same predictable shape, letting frontend/client code handle responses generically instead of writing custom parsing logic per-endpoint.

## 30.7 API Standards

A grab-bag of conventions that make a large API consistent and predictable across many endpoints and (often) many different developers contributing over time:

- Consistent naming conventions (`camelCase` JSON fields is the overwhelming web convention)
- Consistent date/time formatting (ISO-8601 — `2026-07-28T10:15:30Z` — is the near-universal standard)
- Consistent pluralization and resource naming across all endpoints
- Consistent pagination parameter names (`page`/`size`, not `page` in one endpoint and `pageNumber` in another)
- A documented, enforced deprecation policy — how long old API versions remain supported before removal

## 30.8 Security Best Practices

A concise but important checklist, pulling together threads from earlier sections:

- Always hash passwords with BCrypt (or a similarly strong algorithm) — never store plain text
- Always validate and sanitize ALL user input, never trust the client
- Use HTTPS everywhere, in every environment, not just production
- Keep dependencies up to date — many real-world breaches exploit KNOWN vulnerabilities in outdated libraries
- Apply the principle of least privilege everywhere — database users, IAM roles, API scopes
- Never expose stack traces or internal implementation details in error responses sent to clients
- Store secrets in a proper secrets manager, never in source code or plain config files

## 30.9 Rate Limiting

Without limits, a single client (malicious or just badly-behaved) could overwhelm your API with excessive requests, degrading service for everyone else. Rate limiting caps how many requests a given client can make within a time window.

```java
@Component
public class RateLimitInterceptor implements HandlerInterceptor {

    private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String clientId = request.getRemoteAddr();
        Bucket bucket = buckets.computeIfAbsent(clientId, k -> createNewBucket());

        if (bucket.tryConsume(1)) {
            return true;
        }
        response.setStatus(429); // Too Many Requests
        return false;
    }

    private Bucket createNewBucket() {
        return Bucket.builder()
            .addLimit(Bandwidth.simple(100, Duration.ofMinutes(1))) // 100 requests per minute
            .build();
    }
}
```

(This example uses the popular `Bucket4j` library, implementing the "token bucket" algorithm.) At larger scale, rate limiting is often pushed further UP the stack — into an API Gateway or a dedicated edge/CDN layer — so that excessive traffic is blocked BEFORE it even reaches your application servers.

## 30.10 Idempotency

As covered briefly in the REST section, some operations are naturally idempotent (GET, PUT, DELETE), but others — especially POST requests that create resources or trigger payments — are NOT, by default. This becomes a genuine problem with network unreliability: if a client's request times out, was the operation ACTUALLY performed, or not? Retrying blindly risks DUPLICATE processing (charging a customer twice, creating two identical orders).

**Idempotency keys** solve this: the CLIENT generates a unique key for a given logical operation, sends it with the request, and the server remembers which keys it has already processed — safely ignoring (or returning the ORIGINAL result for) a duplicate request with the same key.

```java
@PostMapping("/orders")
public ResponseEntity<OrderResponse> createOrder(
        @RequestHeader("Idempotency-Key") String idempotencyKey,
        @RequestBody OrderRequest request) {

    Optional<OrderResponse> existing = idempotencyService.getExistingResponse(idempotencyKey);
    if (existing.isPresent()) {
        return ResponseEntity.ok(existing.get()); // safely return the ORIGINAL result, don't reprocess
    }

    OrderResponse response = orderService.createOrder(request);
    idempotencyService.storeResponse(idempotencyKey, response);
    return ResponseEntity.status(HttpStatus.CREATED).body(response);
}
```

This is an ESSENTIAL pattern for any payment-related or otherwise non-idempotent-by-nature operation in a distributed system — network failures and client retries are not a hypothetical edge case, they happen constantly at real scale.

## 30.11 Audit Logging

Beyond regular application logs (meant for debugging), many systems — especially anything touching financial data, healthcare data, or user permissions — need a dedicated AUDIT trail: a permanent, tamper-evident record of WHO did WHAT, and WHEN, for compliance and security investigation purposes.

```java
@Entity
public class AuditLog {
    @Id
    @GeneratedValue
    private Long id;
    private String action;         // e.g., "ORDER_CANCELLED"
    private String performedBy;    // the user/system that performed the action
    private String entityType;
    private String entityId;
    private Instant timestamp;
    private String details;        // additional context, often as JSON
}
```

```java
@Around("@annotation(Audited)")
public Object logAuditedAction(ProceedingJoinPoint joinPoint) throws Throwable {
    Object result = joinPoint.proceed();
    auditLogRepository.save(new AuditLog(
        joinPoint.getSignature().getName(),
        getCurrentUsername(),
        Instant.now()
    ));
    return result;
}
```

Notice this leans on the AOP pattern covered earlier — audit logging is a textbook cross-cutting concern, ideally applied declaratively (via an annotation + aspect) rather than manually scattered through business logic, so it can never accidentally be forgotten on a sensitive operation.

---

# 31. Design Patterns in Spring

## 31.1 Singleton

The Singleton pattern ensures a class has exactly ONE instance, shared everywhere it's used. As covered in the Bean Scopes section, this is literally the DEFAULT scope for every Spring bean — Spring implements the Singleton pattern for you automatically, without you writing any of the classic (and somewhat fiddly) manual singleton boilerplate (private constructors, static instance fields, thread-safe lazy initialization).

```java
@Service
public class ConfigurationService {
    // Spring guarantees exactly ONE instance of this class exists per application context —
    // you get Singleton pattern benefits for free, just by using @Service
}
```

## 31.2 Factory

The Factory pattern centralizes object CREATION logic, so calling code doesn't need to know the specific concrete class being instantiated — it just asks a factory for "something that satisfies this need." Spring's `@Bean` methods inside `@Configuration` classes are essentially factory methods, and Spring's `BeanFactory`/`ApplicationContext` itself is fundamentally a giant, sophisticated implementation of the Factory pattern.

```java
@Configuration
public class PaymentServiceConfig {

    @Bean
    public PaymentService paymentService(@Value("${payment.provider}") String provider) {
        return switch (provider) {
            case "stripe" -> new StripePaymentService();
            case "paypal" -> new PayPalPaymentService();
            default -> throw new IllegalArgumentException("Unknown provider: " + provider);
        };
    }
}
```

## 31.3 Builder

The Builder pattern constructs complex objects step by step, avoiding unwieldy constructors with many parameters (especially many parameters of the SAME type, which are easy to accidentally swap by mistake).

```java
public class OrderRequest {
    private final String customerName;
    private final List<OrderItem> items;
    private final String shippingAddress;

    // private constructor, only usable via the builder
    private OrderRequest(Builder builder) {
        this.customerName = builder.customerName;
        this.items = builder.items;
        this.shippingAddress = builder.shippingAddress;
    }

    public static class Builder {
        private String customerName;
        private List<OrderItem> items = new ArrayList<>();
        private String shippingAddress;

        public Builder customerName(String name) { this.customerName = name; return this; }
        public Builder addItem(OrderItem item) { this.items.add(item); return this; }
        public Builder shippingAddress(String address) { this.shippingAddress = address; return this; }
        public OrderRequest build() { return new OrderRequest(this); }
    }
}
```

```java
OrderRequest request = new OrderRequest.Builder()
    .customerName("Alice")
    .addItem(new OrderItem("Widget", 2))
    .shippingAddress("123 Main St")
    .build();
```

In modern Java code, this is often replaced with Lombok's `@Builder` annotation, which generates this exact same builder boilerplate for you automatically.

## 31.4 Strategy

The Strategy pattern defines a FAMILY of interchangeable algorithms/behaviors behind a common interface, letting you swap which specific implementation is used at runtime — exactly the pattern used in the earlier `PaymentService` example (SOLID section), where `StripePaymentService` and `PayPalPaymentService` are interchangeable STRATEGIES for the same `PaymentService` interface.

```java
public interface DiscountStrategy {
    BigDecimal apply(BigDecimal originalPrice);
}

@Component("percentageDiscount")
public class PercentageDiscountStrategy implements DiscountStrategy {
    public BigDecimal apply(BigDecimal originalPrice) {
        return originalPrice.multiply(BigDecimal.valueOf(0.9));
    }
}

@Component("flatDiscount")
public class FlatDiscountStrategy implements DiscountStrategy {
    public BigDecimal apply(BigDecimal originalPrice) {
        return originalPrice.subtract(BigDecimal.TEN);
    }
}
```

Spring's dependency injection makes the Strategy pattern especially natural — you can inject a specific strategy by qualifier name, or even inject ALL implementations as a `List<DiscountStrategy>` / `Map<String, DiscountStrategy>` and pick the right one dynamically at runtime.

## 31.5 Observer

The Observer pattern lets one component ("subject") notify multiple interested "observers" whenever something happens, WITHOUT the subject needing to know anything specific about who's listening. Spring implements this directly through its **application event** mechanism.

```java
public class OrderCreatedEvent extends ApplicationEvent {
    private final Order order;
    public OrderCreatedEvent(Object source, Order order) {
        super(source);
        this.order = order;
    }
}
```

```java
@Service
public class OrderService {
    private final ApplicationEventPublisher eventPublisher;

    public void createOrder(OrderRequest request) {
        Order order = // ... save order
        eventPublisher.publishEvent(new OrderCreatedEvent(this, order)); // notify all listeners
    }
}

@Component
public class EmailNotificationListener {
    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        emailService.sendConfirmation(event.getOrder());
    }
}

@Component
public class InventoryUpdateListener {
    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        inventoryService.reserveStock(event.getOrder());
    }
}
```

`OrderService` publishes ONE event, with ZERO awareness of who's listening — yet BOTH `EmailNotificationListener` and `InventoryUpdateListener` react independently. This is a great way to decouple side effects from the core operation that triggers them, within a single application (as opposed to across services, where Kafka/RabbitMQ handle a similar role, as covered earlier).

## 31.6 Proxy

The Proxy pattern wraps a real object with a substitute "proxy" object that controls access to it, adding extra behavior transparently. This is EXACTLY how Spring implements many of its most powerful features under the hood: `@Transactional`, `@Async`, `@Cacheable`, and AOP in general all work by creating a PROXY around your actual bean, which intercepts method calls to add the extra behavior BEFORE/AFTER delegating to the real object.

```java
@Service
public class OrderService {
    @Transactional
    public void placeOrder(OrderRequest request) {
        // Spring doesn't call this method directly —
        // it calls a generated PROXY, which starts a transaction,
        // THEN calls the real method, THEN commits/rolls back the transaction
    }
}
```

This is also exactly why the "self-invocation" gotcha exists for `@Async`/`@Transactional`/`@Cacheable` (mentioned earlier) — calling `this.placeOrder()` from WITHIN the same class bypasses the proxy entirely, since you're calling the real object directly rather than going through Spring's generated wrapper.

## 31.7 Template Method

The Template Method pattern defines the overall SKELETON of an algorithm in a base class, while letting subclasses override specific STEPS of that algorithm without changing its overall structure. Spring itself uses this pattern extensively internally — `JdbcTemplate`, `RestTemplate`, and `TransactionTemplate` are all named (and designed) around exactly this idea: they handle the repetitive, error-prone boilerplate (opening/closing resources, exception translation), while YOU supply just the specific logic that varies.

```java
jdbcTemplate.query("SELECT * FROM orders WHERE status = ?",
    (rs, rowNum) -> new Order(rs.getLong("id"), rs.getString("status")), // you supply just the row-mapping logic
    "SHIPPED");
```

`JdbcTemplate` handles opening the connection, preparing the statement, executing the query, iterating the result set, and closing everything properly (even on exceptions) — the "template." You only supply the specific bit that's unique to your use case: how to map one row into a Java object.

## 31.8 Dependency Injection Pattern

We covered Dependency Injection in depth back in Section 1, but it's worth including here explicitly as a formally recognized DESIGN PATTERN in its own right (a specific, well-known implementation strategy for the broader Inversion of Control principle) — and arguably THE single most foundational pattern underlying the entire Spring Framework's design philosophy. Every other pattern covered in this section (Factory, Proxy, Strategy, Observer) is made dramatically easier to apply cleanly specifically BECAUSE Spring's dependency injection container manages object creation and wiring for you.

---
# 32. Spring Boot Internals (Expert Level)

## 32.1 Auto Configuration Internals

We covered the CONCEPT of auto-configuration back in Section 2 — now let's go one level deeper into how it actually works mechanically. Every auto-configuration class is registered through a file at `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (in older Spring Boot versions, this was `spring.factories`) inside the `spring-boot-autoconfigure` JAR. This file simply lists the fully-qualified class names of every auto-configuration class Spring Boot knows about.

```
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
```

At startup, Spring Boot reads this entire list and evaluates EVERY auto-configuration class's conditional annotations (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`, and more) to decide which ones actually apply given your specific classpath and configuration. This is why adding a single dependency (like `spring-boot-starter-data-jpa`) can silently configure a dozen beans for you — each relevant auto-configuration class's conditions become satisfied the moment the right classes appear on the classpath.

## 32.2 Spring Boot Startup Process

Understanding the full startup sequence helps enormously when debugging weird boot-time issues. Here's what actually happens when you call `SpringApplication.run()`:

1. **`SpringApplication` instance created** — determines the application type (servlet web, reactive web, or none), and prepares the initial `Environment`
2. **Listeners notified: `ApplicationStartingEvent`** — the very first lifecycle event fired
3. **Environment prepared** — property sources (command line, env vars, property files) are all merged together
4. **Banner printed** (unless disabled, as covered in Section 2)
5. **`ApplicationContext` created** — the specific type depends on your application type (e.g., `AnnotationConfigServletWebServerApplicationContext` for a typical web app)
6. **Context "prepared"** — the context is configured with the environment, and initializers run
7. **Bean definitions loaded** — component scanning runs, `@Configuration` classes are processed, auto-configuration classes are evaluated
8. **Context "refreshed"** — this is where the ACTUAL beans get instantiated, dependency-injected, and initialized (covered in the next section) — including starting the embedded web server
9. **`CommandLineRunner`/`ApplicationRunner` beans executed** (covered back in Section 2)
10. **`ApplicationReadyEvent` published** — the application is now fully started and ready to serve traffic

## 32.3 Bean Factory

We introduced `BeanFactory` back in Section 1 as the most basic container implementation — here's a bit more on how it actually works internally. `BeanFactory` maintains an internal registry mapping bean NAMES to `BeanDefinition` objects — metadata describing HOW to create a bean (its class, constructor arguments, property values, scope, lifecycle callbacks), NOT the actual instantiated bean itself.

```java
// Conceptually, what Spring does internally (simplified):
BeanDefinition definition = new BeanDefinition(OrderService.class);
definition.addConstructorArgument("paymentService");
beanFactory.registerBeanDefinition("orderService", definition);

// only later, when actually needed, does the factory INSTANTIATE the real object:
OrderService service = (OrderService) beanFactory.getBean("orderService");
```

This separation between "definition" (metadata) and "instance" (the actual object) is what allows Spring's rich features — like `BeanFactoryPostProcessor` (covered next), which can modify bean DEFINITIONS before any actual beans are ever instantiated.

## 32.4 Bean Post Processor

`BeanPostProcessor` is one of Spring's most powerful (and somewhat "meta") extension points — it lets you hook into and modify EVERY bean's lifecycle, right around the initialization step, without needing to touch the bean's own class at all.

```java
@Component
public class CustomBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        System.out.println("About to initialize: " + beanName);
        return bean; // you can return a DIFFERENT object here, even a proxy wrapping the original
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Finished initializing: " + beanName);
        return bean;
    }
}
```

This is, in fact, EXACTLY the mechanism Spring itself uses internally to implement `@Autowired` field injection, AOP proxy creation, and `@Async`/`@Transactional` proxy wrapping — all of these are implemented as `BeanPostProcessor` implementations shipped as part of the framework itself. Understanding this demystifies a LOT of Spring's "magic" — it's not really magic, it's a well-designed extension point that Spring's own core features happen to use too.

## 32.5 Application Events

We touched on Spring's event system in the Observer pattern discussion (Section 31), but Spring ALSO publishes its OWN internal lifecycle events, which you can listen to for diagnostic or initialization purposes.

```java
@Component
public class StartupListener {

    @EventListener
    public void onApplicationReady(ApplicationReadyEvent event) {
        System.out.println("Application is fully ready to serve traffic!");
    }

    @EventListener
    public void onContextRefreshed(ContextRefreshedEvent event) {
        System.out.println("Application context has been fully initialized");
    }
}
```

Key built-in Spring Boot lifecycle events, roughly in firing order: `ApplicationStartingEvent` → `ApplicationEnvironmentPreparedEvent` → `ApplicationContextInitializedEvent` → `ApplicationPreparedEvent` → `ContextRefreshedEvent` → `ApplicationStartedEvent` → `ApplicationReadyEvent`. `ApplicationReadyEvent` specifically is the most commonly used one in application code — it fires only once EVERYTHING is fully initialized and the application is genuinely ready to handle real traffic, making it the safest place to run any final startup logic that depends on the ENTIRE application being ready (as opposed to `CommandLineRunner`, which runs at a similar point but is specifically designed for simple run-once logic rather than event-driven listening).

## 32.6 Spring Context Refresh

"Context refresh" refers to the `AbstractApplicationContext.refresh()` method — arguably the single most important method in the entire Spring Framework, since it's what actually transforms a set of bean DEFINITIONS into a fully wired, running application. It performs (roughly) these steps, in order:

1. Prepare the context (set startup time, validate required properties)
2. Invoke `BeanFactoryPostProcessor`s (which can modify bean DEFINITIONS before instantiation)
3. Register `BeanPostProcessor`s (which will apply to EVERY bean instantiated from this point forward)
4. Initialize message source (for i18n)
5. Initialize the event multicaster (for application events)
6. Instantiate ALL remaining SINGLETON beans (non-lazy ones) — this is where the bulk of your application's beans actually get created and wired together
7. Finish refresh — publish `ContextRefreshedEvent`, start the embedded web server (if applicable)

Understanding this sequence explains a lot of subtle Spring behavior — for example, WHY a `BeanFactoryPostProcessor` can do things a regular bean can't (like registering ADDITIONAL bean definitions dynamically), since it runs at a point where beans are still just definitions, not yet real objects.

## 32.7 Conditional Beans

We've referenced conditional annotations several times already — here's a more complete picture of the toolkit Spring Boot gives you for building your OWN conditional configuration, exactly the same tools Spring Boot's own auto-configuration classes use internally.

```java
@Configuration
public class FeatureConfig {

    @Bean
    @ConditionalOnProperty(name = "app.feature.new-checkout", havingValue = "true")
    public CheckoutService newCheckoutService() {
        return new NewCheckoutService();
    }

    @Bean
    @ConditionalOnMissingBean(CheckoutService.class)
    public CheckoutService defaultCheckoutService() {
        return new LegacyCheckoutService();
    }

    @Bean
    @ConditionalOnClass(name = "com.stripe.Stripe")
    public PaymentService stripePaymentService() {
        return new StripePaymentService();
    }

    @Bean
    @Profile("!test")
    public ScheduledTaskService realScheduler() {
        return new ScheduledTaskService();
    }
}
```

This gives you the exact same power Spring Boot's own team uses to build feature-flaggable, environment-aware, dependency-aware configuration in your OWN application — genuinely useful for feature toggles, gradual rollouts, or supporting optional integrations that might not always be present on the classpath.

## 32.8 Custom Starter Creation

Just as `spring-boot-starter-web` and `spring-boot-starter-data-jpa` bundle related dependencies and auto-configuration together, you can build your OWN custom starter — extremely useful for sharing common configuration/functionality (like a company-wide logging setup, a shared security configuration, or a custom client library) across MULTIPLE internal projects/microservices.

A custom starter is typically split into two modules:

**1. The autoconfigure module** (the actual logic):
```java
@AutoConfiguration
@ConditionalOnClass(MyCustomClient.class)
@EnableConfigurationProperties(MyCustomProperties.class)
public class MyCustomAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyCustomClient myCustomClient(MyCustomProperties properties) {
        return new MyCustomClient(properties.getApiKey(), properties.getBaseUrl());
    }
}
```

**2. The starter module** (just a thin POM/build file declaring dependencies — the autoconfigure module PLUS whatever underlying libraries it needs):
```xml
<dependencies>
    <dependency>
        <groupId>com.mycompany</groupId>
        <artifactId>my-custom-client-autoconfigure</artifactId>
    </dependency>
</dependencies>
```

## 32.9 Spring Boot Auto Configuration Creation

Bringing everything in this final section together — here's a genuinely complete, working example of building a small custom auto-configuration from scratch, tying together `@ConditionalOnClass`, `@ConditionalOnMissingBean`, and `@ConfigurationProperties`:

```java
@ConfigurationProperties(prefix = "app.notification")
public class NotificationProperties {
    private String provider = "console";
    private boolean enabled = true;
    // getters and setters
}
```

```java
@AutoConfiguration
@ConditionalOnProperty(prefix = "app.notification", name = "enabled", havingValue = "true", matchIfMissing = true)
@EnableConfigurationProperties(NotificationProperties.class)
public class NotificationAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public NotificationService notificationService(NotificationProperties properties) {
        return switch (properties.getProvider()) {
            case "console" -> new ConsoleNotificationService();
            case "email" -> new EmailNotificationService();
            default -> throw new IllegalStateException("Unknown provider: " + properties.getProvider());
        };
    }
}
```

Registered in `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`:

```
com.mycompany.notification.NotificationAutoConfiguration
```

With this in place, ANY project that adds this module as a dependency automatically gets a fully-configured, sensible-default `NotificationService` bean — configurable via simple properties, and completely overridable by defining their own bean if the defaults don't fit — this is genuinely the EXACT same mechanism, at the exact same level, that Spring Boot's own official starters use internally. Building one yourself is a great way to deeply solidify your understanding of everything covered throughout this entire guide.

---

# Closing Notes

That's the full journey — from the very first principle (Inversion of Control) all the way to writing your own Spring Boot auto-configuration starter. A few honest, practical closing thoughts:

- **Nobody learns Spring Boot by reading alone.** Build something small end-to-end (a simple CRUD API with a database, validation, and tests) before layering in security, messaging, and microservices concerns.
- **Revisit the earlier sections after the later ones.** Concepts like dependency injection and bean lifecycle (Section 1) will make MORE sense once you've seen how heavily Security, AOP, and the Internals section (32) all lean on them.
- **Not every project needs every topic here.** A small internal tool doesn't need Kafka, service discovery, or distributed tracing — reach for these when the actual problem calls for them, not by default.
- **The official Spring documentation is excellent** and worth bookmarking (docs.spring.io) — this guide gives you the conceptual map; the docs give you the precise, always-current details for whatever version you're actually using.

Good luck — and enjoy building with Spring Boot.

---

# Appendix A: End-to-End Worked Example — Building "OrderFlow" From Scratch

Everything up to this point has been concept-by-concept. Now let's actually BUILD something — a small but genuinely complete Order Management API, called "OrderFlow," pulling together concepts from nearly every section above. We'll walk through it exactly the way you'd build it in real life: starting from nothing, adding one capability at a time.

## A.1 Project Setup

We start at Spring Initializr (Section 2.3), selecting Maven, Java 21, and Spring Boot 3.4, with these dependencies: Spring Web, Spring Data JPA, PostgreSQL Driver, Validation, Spring Security, Lombok, and Spring Boot DevTools.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.0</version>
    </parent>
    <groupId>com.orderflow</groupId>
    <artifactId>orderflow-api</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>21</java.version>
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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

This follows Section 2.4's advice exactly: a handful of starters, each pulling in a coherent bundle of compatible dependencies, instead of managing dozens of individual libraries by hand.

## A.2 Domain Model

We're modeling a simple order system: a `Customer` places an `Order`, which contains multiple `OrderItem`s, each referencing a `Product`.

```java
@Entity
@Table(name = "customers")
@Getter @Setter @NoArgsConstructor
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL)
    private List<Order> orders = new ArrayList<>();
}
```

```java
@Entity
@Table(name = "products")
@Getter @Setter @NoArgsConstructor
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(precision = 10, scale = 2)
    private BigDecimal price;

    private int stockQuantity;
}
```

```java
@Entity
@Table(name = "orders")
@Getter @Setter @NoArgsConstructor
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();

    @Enumerated(EnumType.STRING)
    private OrderStatus status = OrderStatus.PENDING;

    @Column(precision = 10, scale = 2)
    private BigDecimal totalAmount;

    @CreationTimestamp
    private LocalDateTime createdAt;

    @Version
    private Long version; // optimistic locking, as covered in Section 8.12

    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
    }
}
```

```java
public enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
}
```

```java
@Entity
@Table(name = "order_items")
@Getter @Setter @NoArgsConstructor
public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "product_id")
    private Product product;

    private int quantity;

    @Column(precision = 10, scale = 2)
    private BigDecimal unitPrice;
}
```

Notice every fetch type here is deliberately chosen following the guidance from Section 8.2/8.3 — `LAZY` for the parent-side associations that could hold a lot of data, `EAGER` only where it's genuinely convenient and bounded (a single `Product` per `OrderItem`).

## A.3 Database Migrations

Following Section 9's guidance, we NEVER let Hibernate auto-generate the schema in this project — everything is explicit, versioned Flyway migrations.

```sql
-- V1__create_customers_table.sql
CREATE TABLE customers (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) NOT NULL UNIQUE
);
```

```sql
-- V2__create_products_table.sql
CREATE TABLE products (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    price NUMERIC(10,2) NOT NULL,
    stock_quantity INT NOT NULL DEFAULT 0
);
```

```sql
-- V3__create_orders_and_items_tables.sql
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    customer_id BIGINT NOT NULL REFERENCES customers(id),
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING',
    total_amount NUMERIC(10,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    version BIGINT DEFAULT 0
);

CREATE TABLE order_items (
    id BIGSERIAL PRIMARY KEY,
    order_id BIGINT NOT NULL REFERENCES orders(id),
    product_id BIGINT NOT NULL REFERENCES products(id),
    quantity INT NOT NULL,
    unit_price NUMERIC(10,2) NOT NULL
);

CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_status ON orders(status);
```

```properties
spring.jpa.hibernate.ddl-auto=validate
spring.flyway.enabled=true
```

## A.4 Repositories

```java
public interface CustomerRepository extends JpaRepository<Customer, Long> {
    Optional<Customer> findByEmail(String email);
}

public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByStockQuantityGreaterThan(int quantity);
}

public interface OrderRepository extends JpaRepository<Order, Long>, JpaSpecificationExecutor<Order> {

    @EntityGraph(attributePaths = {"items", "items.product", "customer"})
    Optional<Order> findWithDetailsById(Long id);

    Page<Order> findByCustomerId(Long customerId, Pageable pageable);

    List<Order> findByStatus(OrderStatus status);
}
```

Note the `@EntityGraph` on `findWithDetailsById` — this is a deliberate, targeted fix for the N+1 problem (Section 8.14) on the ONE query where we know we'll need the full object graph (viewing full order details), while leaving other queries lazy by default.

## A.5 Request/Response DTOs

```java
public record OrderItemRequest(
        @NotNull Long productId,
        @Min(1) int quantity
) {}

public record CreateOrderRequest(
        @NotNull Long customerId,
        @NotEmpty List<OrderItemRequest> items
) {}

public record OrderItemResponse(
        String productName,
        int quantity,
        BigDecimal unitPrice,
        BigDecimal subtotal
) {}

public record OrderResponse(
        Long id,
        String customerName,
        String status,
        List<OrderItemResponse> items,
        BigDecimal totalAmount,
        LocalDateTime createdAt
) {}
```

Using Java `record`s here for immutable DTOs is a modern, concise alternative to Lombok-generated classes — a natural fit for DTOs specifically, since they're pure data carriers with no need for mutability.

## A.6 Mapper

```java
@Mapper(componentModel = "spring")
public interface OrderMapper {

    @Mapping(source = "customer.name", target = "customerName")
    OrderResponse toResponse(Order order);

    @Mapping(source = "product.name", target = "productName")
    @Mapping(target = "subtotal", expression = "java(item.getUnitPrice().multiply(java.math.BigDecimal.valueOf(item.getQuantity())))")
    OrderItemResponse toItemResponse(OrderItem item);
}
```

## A.7 Service Layer — The Business Logic Core

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderService {

    private final OrderRepository orderRepository;
    private final CustomerRepository customerRepository;
    private final ProductRepository productRepository;
    private final OrderMapper orderMapper;
    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public OrderResponse createOrder(CreateOrderRequest request) {
        Customer customer = customerRepository.findById(request.customerId())
                .orElseThrow(() -> new ResourceNotFoundException("Customer not found: " + request.customerId()));

        Order order = new Order();
        order.setCustomer(customer);

        BigDecimal total = BigDecimal.ZERO;

        for (OrderItemRequest itemRequest : request.items()) {
            Product product = productRepository.findById(itemRequest.productId())
                    .orElseThrow(() -> new ResourceNotFoundException("Product not found: " + itemRequest.productId()));

            if (product.getStockQuantity() < itemRequest.quantity()) {
                throw new InsufficientStockException(
                        "Not enough stock for product: " + product.getName());
            }

            product.setStockQuantity(product.getStockQuantity() - itemRequest.quantity());
            // no explicit save() needed here — dirty checking (Section 7.9) handles the update automatically

            OrderItem item = new OrderItem();
            item.setProduct(product);
            item.setQuantity(itemRequest.quantity());
            item.setUnitPrice(product.getPrice());
            order.addItem(item);

            total = total.add(product.getPrice().multiply(BigDecimal.valueOf(itemRequest.quantity())));
        }

        order.setTotalAmount(total);
        Order saved = orderRepository.save(order);

        log.info("Order {} created for customer {}", saved.getId(), customer.getEmail());
        eventPublisher.publishEvent(new OrderCreatedEvent(this, saved));

        return orderMapper.toResponse(saved);
    }

    public OrderResponse getOrder(Long id) {
        Order order = orderRepository.findWithDetailsById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Order not found: " + id));
        return orderMapper.toResponse(order);
    }

    public Page<OrderResponse> getCustomerOrders(Long customerId, Pageable pageable) {
        return orderRepository.findByCustomerId(customerId, pageable).map(orderMapper::toResponse);
    }

    @Transactional
    @CacheEvict(value = "orderSummaries", key = "#id")
    public OrderResponse updateStatus(Long id, OrderStatus newStatus) {
        Order order = orderRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Order not found: " + id));
        order.setStatus(newStatus);
        return orderMapper.toResponse(order); // dirty checking saves this automatically on commit
    }
}
```

This single service class demonstrates: transactions (Section 7.10), custom exceptions (Section 6.2), dirty checking (Section 7.9), application events (Section 31.5), caching eviction (Section 17.1), entity graphs (Section 7.17), and DTO mapping (Section 30.5) — all working together naturally, exactly the way they would in a real production codebase.

## A.8 Event Listener — Decoupled Side Effects

```java
public class OrderCreatedEvent extends ApplicationEvent {
    private final Order order;
    public OrderCreatedEvent(Object source, Order order) {
        super(source);
        this.order = order;
    }
    public Order getOrder() { return order; }
}
```

```java
@Component
@RequiredArgsConstructor
public class OrderNotificationListener {

    private final EmailService emailService;

    @Async
    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        Order order = event.getOrder();
        emailService.sendSimpleEmail(
                order.getCustomer().getEmail(),
                "Order Confirmation",
                "Your order #" + order.getId() + " has been received!"
        );
    }
}
```

Combining `@Async` with `@EventListener` here (Sections 15.1 and 31.5 together) means order creation returns to the client IMMEDIATELY, without waiting for the confirmation email to actually send — exactly the kind of decoupling that keeps an API responsive.

## A.9 Controller Layer

```java
@RestController
@RequestMapping("/api/v1/orders")
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@Valid @RequestBody CreateOrderRequest request) {
        OrderResponse response = orderService.createOrder(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    @GetMapping("/{id}")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable Long id) {
        return ResponseEntity.ok(orderService.getOrder(id));
    }

    @GetMapping("/customer/{customerId}")
    public ResponseEntity<Page<OrderResponse>> getCustomerOrders(
            @PathVariable Long customerId,
            @PageableDefault(size = 20, sort = "createdAt", direction = Sort.Direction.DESC) Pageable pageable) {
        return ResponseEntity.ok(orderService.getCustomerOrders(customerId, pageable));
    }

    @PatchMapping("/{id}/status")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<OrderResponse> updateStatus(@PathVariable Long id, @RequestParam OrderStatus status) {
        return ResponseEntity.ok(orderService.updateStatus(id, status));
    }
}
```

Notice the controller is genuinely thin — as advocated in Section 30.1's layered architecture guidance, it does nothing but translate HTTP concerns (path variables, request bodies, status codes) into calls on the service layer. Zero business logic lives here.

## A.10 Global Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ProblemDetail handleNotFound(ResourceNotFoundException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    }

    @ExceptionHandler(InsufficientStockException.class)
    public ProblemDetail handleInsufficientStock(InsufficientStockException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, ex.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex) {
        ProblemDetail detail = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        detail.setProperty("fieldErrors", errors);
        return detail;
    }
}
```

## A.11 Security Configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthFilter jwtAuthFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(AbstractHttpConfigurer::disable)
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .anyRequest().authenticated())
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

## A.12 Tests

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock private OrderRepository orderRepository;
    @Mock private CustomerRepository customerRepository;
    @Mock private ProductRepository productRepository;
    @Mock private OrderMapper orderMapper;
    @Mock private ApplicationEventPublisher eventPublisher;
    @InjectMocks private OrderService orderService;

    @Test
    void shouldThrowWhenInsufficientStock() {
        Customer customer = new Customer();
        customer.setId(1L);
        Product product = new Product();
        product.setId(1L);
        product.setStockQuantity(1);
        product.setPrice(BigDecimal.TEN);

        when(customerRepository.findById(1L)).thenReturn(Optional.of(customer));
        when(productRepository.findById(1L)).thenReturn(Optional.of(product));

        CreateOrderRequest request = new CreateOrderRequest(1L, List.of(new OrderItemRequest(1L, 5)));

        assertThrows(InsufficientStockException.class, () -> orderService.createOrder(request));
    }
}
```

```java
@SpringBootTest
@Testcontainers
@AutoConfigureMockMvc
class OrderControllerIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired private MockMvc mockMvc;

    @Test
    @WithMockUser
    void shouldReturn404ForMissingOrder() throws Exception {
        mockMvc.perform(get("/api/v1/orders/9999"))
                .andExpect(status().isNotFound());
    }
}
```

## A.13 Dockerizing OrderFlow

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/orderflow-api-1.0.0.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```yaml
services:
  app:
    build: .
    ports: ["8080:8080"]
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/orderflow
      SPRING_PROFILES_ACTIVE: prod
    depends_on: [db]
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: orderflow
      POSTGRES_PASSWORD: secret
```

That's the whole loop, closed: from an empty Spring Initializr project all the way to a Dockerized, tested, secured, observable API — every single piece pulled directly from a concept covered earlier in this guide.

---

# Appendix B: Common Errors & Troubleshooting Guide

Every Spring Boot developer runs into the same handful of errors over and over, especially early on. This appendix walks through the most common ones — what they mean, why they happen, and how to fix them — organized by topic area so you can jump straight to what's biting you.

## B.1 Bean & Context Errors

**`NoSuchBeanDefinitionException`**
- **What it means:** Spring tried to inject a bean of a given type, but couldn't find one registered anywhere in the context.
- **Common causes:** The class is missing its `@Component`/`@Service`/`@Repository` annotation; the class lives OUTSIDE the package tree that component scanning covers (Section 1.11); or you're injecting an INTERFACE with no implementing class annotated as a bean.
- **Fix:** Confirm the class has the right stereotype annotation, and confirm it's inside (or reachable from) your `@SpringBootApplication` class's package.

**`NoUniqueBeanDefinitionException`**
- **What it means:** Spring found MULTIPLE beans matching the type you're trying to inject, and doesn't know which one you want.
- **Common causes:** You have two implementations of the same interface, both registered as beans, with no way to disambiguate.
- **Fix:** Use `@Qualifier("beanName")` (Section 1.9) to specify exactly which one you want, or mark your preferred default with `@Primary`.

**`BeanCurrentlyInCreationException` (circular dependency)**
- **What it means:** Bean A depends on Bean B, which depends back on Bean A — Spring can't resolve which one to create first.
- **Fix:** Usually indicates a design smell — consider whether the two classes should be merged, or whether one dependency can be made lazy (`@Lazy`), or restructure to break the cycle by extracting shared logic into a third class both depend on.

**`ApplicationContext failed to start` (generic wrapper)**
- **What it means:** This is Spring Boot's top-level error when ANYTHING goes wrong during startup — the REAL cause is almost always nested underneath it in the stack trace or the "Caused by:" section.
- **Fix:** Always scroll down to find the ROOT cause (often several "Caused by:" layers deep) rather than reacting to this generic top-level message.

## B.2 JPA & Hibernate Errors

**`LazyInitializationException`**
- **What it means:** You tried to access a lazily-loaded association (Section 8.1) AFTER the persistence context/transaction that loaded the parent entity had already closed.
- **Common causes:** Loading an entity inside a `@Transactional` service method, returning it, and then accessing a lazy collection later in the controller or a serialization layer, outside that transaction.
- **Fix:** Either fetch the needed data EAGERLY for that specific query (via `JOIN FETCH` or `@EntityGraph`, Section 7.17), or map to a DTO WHILE still inside the transactional boundary, so the lazy access happens before the session closes.

**`TransientPropertyValueException` / "object references an unsaved transient instance"**
- **What it means:** You're trying to save an entity that references ANOTHER entity which hasn't been persisted yet.
- **Fix:** Either save the referenced entity first, or add the appropriate `cascade = CascadeType.PERSIST` (Section 8.4) so saving the parent automatically saves the not-yet-persisted child too.

**`org.hibernate.HibernateException: identifier of an instance was altered`**
- **What it means:** You accidentally changed the `@Id` field of a MANAGED entity — Hibernate doesn't allow modifying primary keys of tracked entities.
- **Fix:** Never mutate an entity's ID after it's been persisted; if you genuinely need to "change" identity, delete the old row and insert a new one.

**Duplicate rows / unexpected Cartesian product in query results**
- **What it means:** Usually caused by fetching MULTIPLE `@OneToMany`/`@ManyToMany` collections eagerly in the SAME query (via multiple `JOIN FETCH`), which multiplies rows in SQL.
- **Fix:** Fetch at most ONE collection association eagerly per query; fetch additional collections in separate queries, or use `@EntityGraph` carefully, or apply `Set` instead of `List` with distinct results where appropriate.

**Slow application startup with JPA**
- **Common cause:** `spring.jpa.open-in-view=true` (the Spring Boot default) can mask N+1 problems (Section 8.14) by silently keeping the persistence context open through the whole request — great for avoiding `LazyInitializationException`, but can hide real performance issues.
- **Fix:** Many production teams explicitly set `spring.jpa.open-in-view=false` and deliberately fetch everything they need at the service layer — this surfaces N+1 problems and lazy-loading issues during development rather than silently working (slowly) in production.

## B.3 Web & REST Errors

**HTTP 406 "Not Acceptable"**
- **What it means:** The client's `Accept` header requested a response format your API can't produce.
- **Fix:** Confirm Jackson (or your chosen serializer) is on the classpath, and that you're not accidentally restricting produced media types too narrowly in `@RequestMapping`.

**HTTP 415 "Unsupported Media Type"**
- **What it means:** The CLIENT sent a request with a `Content-Type` your endpoint doesn't accept (commonly, forgetting to set `Content-Type: application/json`).
- **Fix:** Confirm the client is setting the correct `Content-Type` header, matching what your `@PostMapping`/`@RequestMapping` expects (or doesn't explicitly restrict).

**`HttpMessageNotReadableException` — "JSON parse error"**
- **What it means:** The incoming JSON body couldn't be deserialized into your target DTO — a malformed JSON body, a type mismatch (sending a string where a number is expected), or a genuinely empty body where one was required.
- **Fix:** Validate the client's request payload shape against your DTO's fields and types; check for typos in field names (case-sensitive by default).

**CORS errors in the browser console ("blocked by CORS policy")**
- **What it means:** As covered in Section 10.11, the browser is blocking a cross-origin request because the server didn't explicitly allow it.
- **Fix:** Configure a proper `CorsConfigurationSource` bean; remember that CORS configuration must ALLOW the exact origin, methods, and headers actually being used — a common mistake is allowing `GET` but forgetting `OPTIONS` (needed for the browser's CORS "preflight" request) or forgetting `allowCredentials` when cookies/auth headers are involved.

## B.4 Security Errors

**HTTP 401 immediately on every request, even with a valid token**
- **Common causes:** The JWT filter (Section 11.5) isn't registered in the right position in the filter chain; the token's signing key configured for validation doesn't match the key used to SIGN it; or the `Authorization` header isn't being read correctly (missing the `Bearer ` prefix, or a typo in the header name).
- **Fix:** Add temporary logging inside the JWT filter to confirm it's actually being invoked and see exactly what it's receiving/rejecting.

**403 Forbidden instead of expected 200, for an authenticated user**
- **What it means:** Authentication succeeded (Spring knows who you are), but authorization failed (Section 10.3) — the user's roles/authorities don't satisfy the endpoint's requirement.
- **Fix:** Double-check the EXACT role/authority string required (`hasRole("ADMIN")` actually checks for `ROLE_ADMIN` internally — a very common mismatch when authorities are populated manually without the `ROLE_` prefix).

**Default random password printed in console logs**
- **What it means:** You added `spring-boot-starter-security` but haven't configured your OWN authentication mechanism yet — Spring Boot's secure-by-default behavior (Section 10.1) generates a random password for a default "user" account.
- **Fix:** This is expected, temporary behavior — configure your real `UserDetailsService`/authentication mechanism, and this default disappears entirely.

## B.5 Database Connection & Migration Errors

**`HikariPool-1 - Connection is not available, request timed out`**
- **What it means:** Every connection in the pool (Section 29.1) is currently in use, and a NEW request timed out waiting for one to free up.
- **Common causes:** Connection leaks (code acquiring a connection and never releasing it — often from manually-managed `EntityManager`/JDBC code outside Spring's normal transactional boundaries); a pool sized too small for actual concurrent load; or long-running queries holding connections much longer than expected.
- **Fix:** Enable HikariCP's `leak-detection-threshold` (Section 29.2) to pinpoint leaks; review whether pool size matches real concurrency needs; check for unexpectedly slow queries holding connections open.

**Flyway: "Migration checksum mismatch"**
- **What it means:** A migration file that Flyway ALREADY applied has since been EDITED — violating the core rule from Section 9.3 that applied migrations must never change.
- **Fix:** Never edit an already-applied migration; create a NEW migration to make the additional change instead. In a genuine emergency (like fixing a typo in a migration that was never actually deployed anywhere), Flyway's `repair` command can realign checksums — but this should be rare and deliberate, never routine.

**Application fails to start: "relation does not exist"**
- **What it means:** Your entity model expects a table/column that doesn't actually exist in the database — usually because a migration wasn't run, or `ddl-auto` is set to `validate` (correctly, per Section 22.3) and it's catching a genuine mismatch between your Java entities and your actual schema.
- **Fix:** Confirm all migrations have been applied (`flyway_schema_history` table, or Liquibase's equivalent); confirm your entity's `@Table`/`@Column` names exactly match the real schema.

## B.6 Async & Threading Issues

**`@Async` method runs synchronously anyway, blocking the caller**
- **What it means:** Exactly the self-invocation gotcha covered in Sections 15.1 and 31.6 — you called the `@Async` method from WITHIN the same class, bypassing Spring's proxy.
- **Fix:** Move the `@Async` method to a DIFFERENT bean, and call it from there.

**`java.util.ConcurrentModificationException` in a scheduled task or async handler**
- **What it means:** A shared, non-thread-safe collection (like a plain `ArrayList` or `HashMap`) is being modified by multiple threads concurrently.
- **Fix:** Use thread-safe collections (`ConcurrentHashMap`, `CopyOnWriteArrayList`) for any state genuinely shared across threads, or better, avoid mutable shared state entirely where possible.

---

# Appendix C: Glossary of Key Terms

A quick-reference glossary of every major term used throughout this guide, in plain English, for fast lookup.

- **ACID** — Atomicity, Consistency, Isolation, Durability; the four guarantees a proper database transaction provides.
- **AOP** — Aspect-Oriented Programming; a way of applying cross-cutting logic (logging, security, transactions) across many classes without duplicating code.
- **Auto-configuration** — Spring Boot's mechanism for automatically configuring beans based on what's present on the classpath.
- **Bean** — Any object whose lifecycle (creation, wiring, destruction) is managed by the Spring container.
- **BeanFactory** — The most basic Spring IoC container implementation.
- **Cascade** — Configuration controlling whether an operation on a parent entity automatically applies to its related child entities.
- **Circuit Breaker** — A resilience pattern that stops calling a failing downstream service temporarily, to prevent cascading failures.
- **Component Scanning** — Spring's automatic discovery of classes annotated as beans, within a defined package tree.
- **DI (Dependency Injection)** — The technique of supplying a class's dependencies from outside, rather than the class creating them itself.
- **DTO (Data Transfer Object)** — A plain object shaped specifically for transferring data across a boundary (like an API), separate from internal domain/entity models.
- **Entity** — A Java class mapped directly to a database table via JPA.
- **Eureka** — Netflix's service discovery/registry implementation, commonly used in Spring Cloud microservices.
- **Fetch Type** — Configuration on a JPA relationship controlling whether related data loads immediately (EAGER) or on first access (LAZY).
- **Idempotent** — An operation that produces the same end result no matter how many times it's performed.
- **IoC (Inversion of Control)** — The general principle of an external system (like Spring) controlling object creation and wiring, instead of your own code doing it directly.
- **JPA (Jakarta Persistence API)** — The standard Java specification for ORM; Hibernate is its most common implementation.
- **JPQL** — Jakarta Persistence Query Language; SQL-like syntax that operates on entity classes/fields rather than raw tables/columns.
- **JWT (JSON Web Token)** — A compact, self-contained, cryptographically-signed token commonly used for stateless authentication.
- **N+1 Problem** — A performance anti-pattern where fetching N parent records triggers N additional queries for related child data.
- **ORM (Object-Relational Mapping)** — The technique of mapping Java objects onto relational database tables and back.
- **Persistence Context** — Hibernate's first-level cache, tracking all "managed" entities for the duration of a transaction.
- **Pointcut** — An expression defining WHERE an AOP aspect's advice applies.
- **Profile** — A named group of beans/configuration active only in specific environments (dev, test, prod).
- **Proxy** — A wrapper object that intercepts calls to the real object, used heavily by Spring to implement `@Transactional`, `@Async`, `@Cacheable`, and AOP in general.
- **REST** — Representational State Transfer; an architectural style for designing networked APIs around resources and standard HTTP verbs.
- **Rollback** — Undoing all changes made within a failed transaction, restoring the database to its pre-transaction state.
- **Service Discovery** — The mechanism by which services locate each other dynamically at runtime, rather than via hardcoded addresses.
- **SLF4J** — Simple Logging Facade for Java; an abstraction layer over concrete logging implementations like Logback.
- **Starter** — A curated bundle of related dependencies, at tested-compatible versions, provided by Spring Boot for a specific capability.
- **TTL (Time To Live)** — How long a cached (or otherwise temporary) entry remains valid before automatic expiration.

---

*End of guide.*

# Appendix D: Complete Annotation Reference

This appendix collects EVERY significant annotation covered throughout this guide (plus a few common ones not explicitly called out earlier) into one browsable reference, grouped by category, so you don't have to hunt back through sections to remember what something does.

## D.1 Core Bean & Configuration Annotations

- **`@Component`** — Generic stereotype marking a class as a Spring-managed bean, discovered via component scanning.
- **`@Service`** — Semantic specialization of `@Component` for business-logic/service-layer classes.
- **`@Repository`** — Semantic specialization of `@Component` for data-access classes; also enables automatic persistence exception translation.
- **`@Controller`** — Marks a class as a Spring MVC controller returning view names.
- **`@RestController`** — Combines `@Controller` + `@ResponseBody`; returns data directly (usually JSON) instead of view names.
- **`@Configuration`** — Marks a class as a source of bean definitions.
- **`@Bean`** — Marks a method inside a `@Configuration` class as producing a managed bean.
- **`@Autowired`** — Instructs Spring to inject a matching bean automatically (constructor, setter, or field).
- **`@Qualifier`** — Disambiguates which specific bean to inject when multiple candidates of the same type exist.
- **`@Primary`** — Marks a bean as the default choice when multiple candidates exist and no `@Qualifier` is specified.
- **`@Value`** — Injects a single configuration property value directly into a field.
- **`@Scope`** — Declares a bean's scope (`singleton`, `prototype`, `request`, `session`).
- **`@Lazy`** — Delays bean initialization until it's first actually needed, rather than at startup.
- **`@PostConstruct`** — Marks a method to run automatically right after dependency injection completes.
- **`@PreDestroy`** — Marks a method to run automatically just before the bean is destroyed.
- **`@ComponentScan`** — Declares which package(s) Spring should scan for annotated components.
- **`@Import`** — Explicitly imports additional `@Configuration` classes into the context.
- **`@Profile`** — Restricts a bean/configuration to only be active under specific named profiles.
- **`@ConditionalOnClass`** — Applies a configuration only if a specific class is present on the classpath.
- **`@ConditionalOnMissingBean`** — Applies a configuration only if no bean of the given type already exists.
- **`@ConditionalOnProperty`** — Applies a configuration only if a specific property has (or lacks) a given value.
- **`@ConfigurationProperties`** — Binds a whole block of external configuration to a strongly-typed class.
- **`@EnableConfigurationProperties`** — Registers a `@ConfigurationProperties` class as a usable bean.

## D.2 Web & REST Annotations

- **`@RequestMapping`** — General-purpose mapping of a URL (and optionally HTTP method) to a controller/method.
- **`@GetMapping` / `@PostMapping` / `@PutMapping` / `@PatchMapping` / `@DeleteMapping`** — HTTP-method-specific shortcuts for `@RequestMapping`.
- **`@PathVariable`** — Binds a dynamic segment of the URL path to a method parameter.
- **`@RequestParam`** — Binds a query string parameter to a method parameter.
- **`@RequestBody`** — Deserializes the HTTP request body (typically JSON) into a Java object.
- **`@ResponseBody`** — Marks a method's return value to be serialized directly into the HTTP response body.
- **`@ResponseStatus`** — Declares the HTTP status code a method (or exception class) should produce.
- **`@RequestHeader`** — Binds an HTTP request header value to a method parameter.
- **`@CrossOrigin`** — Enables CORS for a specific controller/method.
- **`@ControllerAdvice` / `@RestControllerAdvice`** — Marks a class as a GLOBAL handler applying across all (or many) controllers, typically for exception handling.
- **`@ExceptionHandler`** — Marks a method as responsible for handling a specific exception type.

## D.3 Validation Annotations

- **`@Valid`** — Triggers cascading validation of an object's constraints (standard Jakarta Bean Validation).
- **`@Validated`** — Spring's own validation trigger, additionally supporting validation groups.
- **`@NotNull` / `@NotBlank` / `@NotEmpty`** — Require a value to be present (with increasingly strict definitions of "present").
- **`@Size`** — Constrains the length of a String or the size of a Collection.
- **`@Min` / `@Max`** — Constrain a numeric value to a range.
- **`@Email`** — Requires a value to match a valid email address format.
- **`@Pattern`** — Requires a value to match a given regular expression.
- **`@Past` / `@Future`** — Require a date to be in the past or future, respectively.
- **`@Positive` / `@Negative`** — Constrain a numeric value's sign.

## D.4 Data & Persistence Annotations

- **`@Entity`** — Marks a class as a JPA-managed entity mapped to a database table.
- **`@Table`** — Customizes the table name/schema for an entity.
- **`@Id`** — Marks a field as the entity's primary key.
- **`@GeneratedValue`** — Specifies how the primary key value is generated.
- **`@Column`** — Customizes column-level mapping details (name, nullability, precision, length).
- **`@OneToOne` / `@OneToMany` / `@ManyToOne` / `@ManyToMany`** — Declare the type of relationship between two entities.
- **`@JoinColumn`** — Specifies the foreign key column for a relationship.
- **`@JoinTable`** — Specifies the join table used for a many-to-many relationship.
- **`@Embeddable` / `@Embedded`** — Mark a reusable value-object type, and its usage within an entity, respectively.
- **`@EmbeddedId`** — Marks a composite primary key represented by an `@Embeddable` class.
- **`@Version`** — Enables optimistic locking on an entity via an automatically-managed version column.
- **`@Transactional`** — Wraps a method in a database transaction, with automatic commit/rollback.
- **`@Query`** — Defines a custom JPQL or native SQL query on a repository method.
- **`@Modifying`** — Marks a `@Query` as performing an update/delete rather than a select.
- **`@EntityGraph`** — Declares which related entities should be eagerly fetched for a specific query.
- **`@CreationTimestamp` / `@UpdateTimestamp`** — Automatically populate a timestamp field on creation/update (Hibernate-specific).
- **`@Enumerated`** — Specifies how a Java enum should be persisted (ordinal or string).

## D.5 Security Annotations

- **`@EnableWebSecurity`** — Enables Spring Security's web security support and configuration.
- **`@EnableMethodSecurity`** — Enables method-level security annotations like `@PreAuthorize`.
- **`@PreAuthorize`** — Checks an authorization expression BEFORE a method executes.
- **`@PostAuthorize`** — Checks an authorization expression AFTER a method executes, with access to the return value.
- **`@Secured`** — A simpler, older alternative to `@PreAuthorize`, restricted to role checks only.

## D.6 Async, Scheduling & Caching Annotations

- **`@EnableAsync`** — Enables Spring's asynchronous method execution support.
- **`@Async`** — Marks a method to execute on a separate thread rather than the calling thread.
- **`@EnableScheduling`** — Enables Spring's scheduled task execution support.
- **`@Scheduled`** — Marks a method to run automatically on a fixed rate, fixed delay, or cron schedule.
- **`@EnableCaching`** — Enables Spring's caching abstraction.
- **`@Cacheable`** — Caches a method's return value, keyed by its arguments.
- **`@CachePut`** — Always executes the method AND updates the cache with the result.
- **`@CacheEvict`** — Removes one or all entries from a cache.

## D.7 Testing Annotations

- **`@Test`** — Marks a method as a JUnit 5 test case.
- **`@BeforeEach` / `@AfterEach`** — Run before/after EVERY test method in a class.
- **`@BeforeAll` / `@AfterAll`** — Run once, before/after ALL test methods in a class.
- **`@ParameterizedTest`** — Runs the same test logic repeatedly against different supplied inputs.
- **`@Mock`** — Creates a Mockito mock of a given type.
- **`@InjectMocks`** — Injects declared `@Mock` fields into the object under test.
- **`@SpringBootTest`** — Loads the FULL Spring application context for an integration-style test.
- **`@WebMvcTest`** — Loads only the web-layer slice of the application context, for fast controller testing.
- **`@MockBean`** — Replaces a real bean in the test context with a Mockito mock.
- **`@DataJpaTest`** — Loads only JPA-related components, backed by an embedded/test database, for repository testing.
- **`@Testcontainers` / `@Container`** — Enable and declare Docker-container-backed test infrastructure.

## D.8 AOP Annotations

- **`@Aspect`** — Marks a class as defining cross-cutting AOP logic.
- **`@Pointcut`** — Declares a reusable expression describing WHERE advice should apply.
- **`@Before` / `@After` / `@AfterReturning` / `@AfterThrowing` / `@Around`** — Declare WHEN advice logic should run relative to the target method's execution.

---

# Appendix E: Frequently Asked Interview Questions

Spring Boot interview questions tend to cluster around the same core themes. This appendix collects the most commonly asked ones, with concise, plain-English answers — great for quick review before an interview, or just to test how solidly the concepts from this guide have sunk in.

## E.1 Core Spring & IoC

**Q: What is the difference between Spring and Spring Boot?**
A: Spring is the underlying framework providing IoC, DI, AOP, and more (Section 1). Spring Boot is built ON TOP of Spring, adding auto-configuration, starter dependencies, and an embedded server, specifically to eliminate the heavy manual configuration that plain Spring historically required (Section 2).

**Q: What is the difference between `@Component`, `@Service`, and `@Repository`?**
A: Functionally, all three register a class as a Spring bean identically. They differ in SEMANTIC MEANING (communicating the class's role to other developers), and `@Repository` additionally enables automatic translation of persistence-layer exceptions into Spring's unified exception hierarchy (Section 1.9).

**Q: Explain the Spring Bean lifecycle.**
A: Instantiation → dependency injection → aware interfaces → `@PostConstruct` → bean ready for use → `@PreDestroy` on shutdown (Section 1.5, covered in full detail).

**Q: What's the default bean scope in Spring, and what other scopes exist?**
A: `singleton` is the default — one shared instance per container. Other scopes include `prototype` (a new instance every request for the bean), and web-specific scopes `request`/`session`/`application` (Section 1.8).

**Q: Why is constructor injection preferred over field injection?**
A: It makes dependencies explicit and immutable (`final` fields), fails fast if a required dependency is missing, and makes unit testing straightforward without needing reflection-based mocking tricks (Section 1.3).

## E.2 Spring Boot Fundamentals

**Q: How does Spring Boot auto-configuration actually work?**
A: At startup, Spring Boot reads a list of candidate auto-configuration classes and evaluates their conditional annotations (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, etc.) against your actual classpath and existing bean definitions, applying only the ones whose conditions are satisfied (Sections 2.5 and 32.1).

**Q: What's the difference between `application.properties` and `application.yml`?**
A: Both configure the same underlying settings; YAML supports hierarchical nesting more readably, while properties files use flat, dot-separated keys. Functionally equivalent — a matter of team preference (Sections 2.7–2.8).

**Q: What is a Spring Boot "starter"?**
A: A curated Maven/Gradle dependency bundle pulling in everything needed for a specific capability (like web development or JPA), at tested-compatible versions, so you don't manually manage a web of individual library versions (Section 2.4).

## E.3 REST & Web Layer

**Q: What's the difference between `@RequestParam` and `@PathVariable`?**
A: `@PathVariable` binds a dynamic segment embedded IN the URL path itself (`/orders/{id}`); `@RequestParam` binds a query string parameter (`?status=SHIPPED`) — typically used for optional filters/modifiers rather than identifying a specific resource (Sections 3.4–3.5).

**Q: Why use `ResponseEntity` instead of returning a plain object from a controller?**
A: `ResponseEntity` gives full control over the HTTP status code and headers, not just the response body — essential for a properly RESTful API where status codes communicate meaningful information to clients (Section 3.7).

**Q: What's the difference between PUT and PATCH?**
A: `PUT` replaces a resource ENTIRELY (the client sends the full representation); `PATCH` applies a PARTIAL update (only the fields being changed) (Section 3.2).

## E.4 JPA & Hibernate

**Q: Explain the N+1 query problem and how to fix it.**
A: Fetching N parent entities, then lazily accessing a related entity/collection on EACH one individually, triggers N additional queries beyond the original 1 — for a total of N+1 database round trips. Fixed via `JOIN FETCH` in JPQL or `@EntityGraph`, which fetch the related data in the SAME query (Section 8.14).

**Q: What's the difference between `save()` and Hibernate's automatic dirty checking?**
A: For a NEW (transient) entity, you need an explicit `save()` call. For an ALREADY-managed entity within an active transaction, simply modifying its fields is enough — Hibernate automatically detects the change ("dirty checking") and issues the appropriate `UPDATE` on commit, with no explicit save call needed (Section 7.9).

**Q: What's the difference between `CrudRepository` and `JpaRepository`?**
A: `JpaRepository` extends `CrudRepository` (through `PagingAndSortingRepository`), adding JPA-specific and pagination/sorting capabilities. In practice, `JpaRepository` is almost always what you extend for real applications (Sections 7.5–7.6).

**Q: What is optimistic locking, and how do you implement it?**
A: A concurrency-conflict-detection strategy that ASSUMES conflicts are rare, using a `@Version` column that Hibernate checks and increments on every update — if another transaction already changed the row, an `OptimisticLockException` is thrown (Section 8.12).

## E.5 Security

**Q: How does Spring Security's filter chain work?**
A: Every incoming HTTP request passes through a CHAIN of servlet filters, each handling one specific security concern (credential extraction, CSRF checking, authorization enforcement), configured declaratively via `SecurityFilterChain` (Section 10.7).

**Q: What's the difference between authentication and authorization?**
A: Authentication answers "who are you?" Authorization answers "given who you are, what are you allowed to do?" — authentication always happens first (Sections 10.2–10.3).

**Q: Why use BCrypt for password hashing instead of a plain hash like SHA-256?**
A: BCrypt is deliberately slow (resisting brute-force attacks) and automatically incorporates a random salt per password (defeating precomputed rainbow-table attacks) — properties a generic fast hash function doesn't provide (Section 10.5).

**Q: How does JWT-based authentication achieve statelessness?**
A: The token itself is self-contained and cryptographically signed — the server can validate it (checking the signature and expiration) without needing to look anything up in a shared session store, meaning any server instance can validate any request independently (Sections 10.14 and 11.1).

## E.6 Transactions & Concurrency

**Q: What happens if an exception is thrown inside a `@Transactional` method?**
A: By default, any UNCHECKED exception triggers an automatic rollback of everything done within that method's transaction; checked exceptions do NOT trigger rollback by default unless explicitly configured with `rollbackFor` (Section 7.10).

**Q: What's the difference between optimistic and pessimistic locking?**
A: Optimistic locking assumes conflicts are rare and detects them after the fact via a version check; pessimistic locking assumes conflicts are likely and proactively locks the row in the database the moment it's read, blocking other transactions until released (Sections 8.12–8.13).

**Q: Why does `@Async` sometimes silently not work?**
A: The most common cause is self-invocation — calling the `@Async` method from WITHIN the same class bypasses Spring's proxy mechanism entirely, since Spring's AOP-based interception only applies to calls arriving from OUTSIDE the bean (Sections 15.1 and 31.6).

## E.7 Microservices

**Q: What's the difference between synchronous and asynchronous service communication?**
A: Synchronous (REST/Feign) calls wait for an immediate response, creating tight runtime coupling; asynchronous (event-driven, via Kafka/RabbitMQ) publishes an event and moves on, decoupling services in TIME — the consumer can process it whenever it's ready (Section 27.11).

**Q: What problem does a circuit breaker solve?**
A: It prevents cascading failures — when a downstream service is failing or slow, the circuit breaker "opens" and fails fast (using a fallback) instead of letting callers pile up waiting on a struggling dependency, which could otherwise exhaust the caller's own resources too (Sections 27.7–27.8).

**Q: Why is service discovery needed in a microservices architecture?**
A: Service instances scale up/down and get redeployed constantly, so hardcoded addresses are impractical — service discovery lets services register themselves and be looked up dynamically, BY NAME, at runtime (Section 27.2).

## E.8 Testing

**Q: What's the difference between `@Mock` and `@MockBean`?**
A: `@Mock` (plain Mockito) creates a mock object for use in a lightweight, isolated unit test with no Spring context involved. `@MockBean` specifically REPLACES a real bean within a loaded Spring application context (used with `@SpringBootTest`/`@WebMvcTest`) with a mock version (Section 19.3).

**Q: What's the difference between `@WebMvcTest` and `@SpringBootTest`?**
A: `@WebMvcTest` loads only the web-layer SLICE of the application (controllers, filters, exception handlers) — fast, but requires mocking the service layer. `@SpringBootTest` loads the FULL application context — slower, but exercises real end-to-end wiring (Sections 19.3 and 19.5).

**Q: Why use TestContainers instead of an in-memory database like H2 for integration tests?**
A: An in-memory database can behave subtly differently from your actual production database engine, potentially hiding real bugs. TestContainers spins up a REAL instance of your actual production database (in a disposable Docker container), giving much higher test fidelity (Section 19.6).

---

# Appendix F: application.yml Configuration Reference

A single, categorized reference of the configuration properties you'll reach for constantly across a real Spring Boot project — everything below has appeared conceptually somewhere in this guide, collected here in one browsable place with plain-English explanations.

## F.1 Server Configuration

```yaml
server:
  port: 8080                          # the port the embedded server listens on
  servlet:
    context-path: /api                # prefixes every URL with /api
    session:
      timeout: 30m                    # how long an inactive HTTP session stays valid
  compression:
    enabled: true                     # gzip-compress responses over a size threshold
  error:
    include-message: always           # include exception messages in default error responses (careful in prod)
    include-stacktrace: never         # NEVER expose stack traces to clients in production
  shutdown: graceful                  # wait for in-flight requests to finish before shutting down
```

## F.2 Datasource & JPA Configuration

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      leak-detection-threshold: 60000
  jpa:
    hibernate:
      ddl-auto: validate              # NEVER use create/update in production — see Section 22.3
    open-in-view: false               # explicit is better — surfaces N+1 issues rather than masking them
    show-sql: false                   # true is useful locally, noisy in production
    properties:
      hibernate:
        jdbc:
          batch_size: 50
        order_inserts: true
        order_updates: true
        format_sql: true
```

## F.3 Flyway / Liquibase

```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: false
  liquibase:
    enabled: false
    change-log: classpath:db/changelog/db.changelog-master.xml
```

## F.4 Security & JWT

```yaml
app:
  jwt:
    secret: ${JWT_SECRET}
    access-token-expiration: 900000        # 15 minutes, in milliseconds
    refresh-token-expiration: 604800000    # 7 days, in milliseconds

spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope: [email, profile]
```

## F.5 Logging

```yaml
logging:
  level:
    root: INFO
    com.myapp: DEBUG
    org.hibernate.SQL: DEBUG
    org.springframework.web: WARN
  file:
    name: logs/application.log
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%X{requestId}] %-5level %logger{36} - %msg%n"
```

## F.6 Actuator

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus   # NEVER "*" in production
  endpoint:
    health:
      show-details: when-authorized
  metrics:
    tags:
      application: ${spring.application.name}
```

## F.7 Mail

```yaml
spring:
  mail:
    host: smtp.sendgrid.net
    port: 587
    username: apikey
    password: ${SENDGRID_API_KEY}
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
```

## F.8 Kafka

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: order-service
      auto-offset-reset: earliest
      enable-auto-commit: false
    producer:
      acks: all
      retries: 3
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

## F.9 Redis

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      timeout: 2000ms
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 2
```

## F.10 File Upload

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 50MB
      enabled: true
```

## F.11 Profiles

```yaml
spring:
  profiles:
    active: dev
    group:
      production: prod,monitoring     # activating "production" also activates prod and monitoring
```

---

# Appendix G: Related Topics Worth Knowing

A handful of important, closely-related Spring topics that didn't map directly onto the original 32-topic outline, but come up constantly in real Spring Boot work. Brief, practical coverage of each.

## G.1 RestTemplate vs WebClient

For years, `RestTemplate` was Spring's standard tool for making OUTGOING HTTP calls to other services (as opposed to `@FeignClient`'s declarative style, Section 27.6). It's synchronous and blocking.

```java
RestTemplate restTemplate = new RestTemplate();
PaymentResponse response = restTemplate.postForObject(
    "http://payment-service/api/payments", request, PaymentResponse.class);
```

`RestTemplate` is now in maintenance mode — Spring recommends `WebClient` (originally built for reactive/non-blocking applications, but usable in traditional blocking apps too) for new development.

```java
WebClient webClient = WebClient.builder().baseUrl("http://payment-service").build();

PaymentResponse response = webClient.post()
    .uri("/api/payments")
    .bodyValue(request)
    .retrieve()
    .bodyToMono(PaymentResponse.class)
    .block(); // .block() makes this call synchronous, if used from a traditional (non-reactive) app
```

`WebClient` shines especially in genuinely reactive applications (built on Spring WebFlux instead of traditional Spring MVC), where you'd typically chain reactive operators instead of calling `.block()`.

## G.2 OAuth2 / Single Sign-On

Beyond the JWT-based authentication built and issued by your OWN application (Section 11), many applications need to let users log in via an EXTERNAL identity provider (Google, GitHub, a corporate Okta/Azure AD tenant) — this is what OAuth2/OpenID Connect (OIDC) login support provides.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
```

```java
http.oauth2Login(oauth2 -> oauth2.defaultSuccessUrl("/dashboard"));
```

With just this configuration, Spring Security handles the ENTIRE OAuth2 authorization code flow — redirecting to Google's login page, handling the callback, exchanging the authorization code for tokens, and populating an authenticated `Authentication` object — all without you writing any of that protocol-level logic by hand.

## G.3 Spring Batch

For genuinely large-scale, offline BATCH processing jobs (nightly file imports, large data migrations, end-of-day reconciliation reports) that go well beyond what a simple `@Scheduled` method should handle, Spring Batch provides a dedicated framework with built-in support for chunked processing, restart-on-failure, and detailed job execution tracking.

```java
@Bean
public Job importOrdersJob(JobRepository jobRepository, Step importStep) {
    return new JobBuilder("importOrdersJob", jobRepository)
        .start(importStep)
        .build();
}

@Bean
public Step importStep(JobRepository jobRepository, PlatformTransactionManager txManager,
                        ItemReader<OrderCsvRow> reader, ItemProcessor<OrderCsvRow, Order> processor,
                        ItemWriter<Order> writer) {
    return new StepBuilder("importStep", jobRepository)
        .<OrderCsvRow, Order>chunk(100, txManager)
        .reader(reader)
        .processor(processor)
        .writer(writer)
        .build();
}
```

The "chunk" model here processes records in configurable batches (100 at a time in this example) — reading, transforming, and writing each chunk as its own mini-transaction, which means a failure partway through a huge job doesn't require reprocessing everything from scratch; Spring Batch tracks exactly how far a job got and can resume from there.

## G.4 GraphQL

While REST (Section 3) remains the dominant API style, GraphQL is an alternative query language for APIs that lets CLIENTS specify exactly which fields they need, in a single request — avoiding both "over-fetching" (getting fields you don't need) and "under-fetching" (needing multiple round trips to assemble a full picture) that can happen with rigid REST endpoints.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-graphql</artifactId>
</dependency>
```

```graphql
type Order {
    id: ID!
    customerName: String!
    status: String!
    items: [OrderItem!]!
}

type Query {
    order(id: ID!): Order
}
```

```java
@Controller
public class OrderGraphQlController {

    @QueryMapping
    public Order order(@Argument Long id) {
        return orderService.getOrder(id);
    }
}
```

A client can then request EXACTLY the fields it needs (`{ order(id: 5) { customerName status } }`, skipping `items` entirely if it doesn't need them) — genuinely useful for apps with varied client needs (mobile vs. web) hitting the same underlying data.

## G.5 Feature Flags

Rolling out new functionality gradually (to a percentage of users, or specific accounts) — rather than an all-or-nothing deployment — is a common production need. While Spring doesn't ship a dedicated feature-flag framework, the `@ConditionalOnProperty` pattern (Section 32.7) combined with a simple property-driven toggle gets you a long way for simple cases; for sophisticated, dynamically-updatable flags (changed without a redeploy, targeted at specific user segments), teams typically integrate a dedicated service like LaunchDarkly or Unleash.

```java
@Service
public class CheckoutService {

    @Value("${feature.new-checkout-flow:false}")
    private boolean newCheckoutFlowEnabled;

    public CheckoutResult checkout(CheckoutRequest request) {
        return newCheckoutFlowEnabled ? newFlow(request) : legacyFlow(request);
    }
}
```

## G.6 Multi-Tenancy

Some Spring Boot applications serve MULTIPLE distinct customers ("tenants") from a single deployed application, needing to keep each tenant's data properly isolated. Common strategies:

- **Separate database per tenant** — strongest isolation, most operational overhead (migrations must run per-tenant-database)
- **Separate schema per tenant, shared database** — a middle ground
- **Shared schema, tenant ID column** — simplest operationally, requires disciplined query filtering (every single query must filter by tenant, usually enforced via Hibernate filters or a base repository)

```java
@Entity
@Table(name = "orders")
@Filter(name = "tenantFilter", condition = "tenant_id = :tenantId")
public class Order {
    private String tenantId;
    // ...
}
```

```java
Session session = entityManager.unwrap(Session.class);
session.enableFilter("tenantFilter").setParameter("tenantId", currentTenantId());
```

Multi-tenancy is a genuinely significant architectural decision made early in a project's life — retrofitting it onto an application not originally designed for it is a substantial undertaking, so it's worth deliberately deciding on a strategy up front if you know multi-tenancy is coming.

---

# Appendix H: Extended Code Cookbook

A collection of additional, standalone, practical code patterns that come up constantly in real Spring Boot projects but didn't fit neatly into the main 32 sections. Each one is a self-contained recipe you can adapt directly.

## H.1 Cross-Field Validation

Sometimes a validation rule depends on MULTIPLE fields together (e.g., "end date must be after start date"), which a single-field annotation like `@NotNull` can't express. This uses the same custom validator mechanism from Section 5.4, but applied at the CLASS level instead of a single field.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = DateRangeValidator.class)
public @interface ValidDateRange {
    String message() default "End date must be after start date";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

```java
public class DateRangeValidator implements ConstraintValidator<ValidDateRange, BookingRequest> {
    @Override
    public boolean isValid(BookingRequest request, ConstraintValidatorContext context) {
        if (request.getStartDate() == null || request.getEndDate() == null) return true; // let @NotNull handle nulls
        return request.getEndDate().isAfter(request.getStartDate());
    }
}
```

```java
@ValidDateRange
public class BookingRequest {
    @NotNull private LocalDate startDate;
    @NotNull private LocalDate endDate;
}
```

## H.2 Standardized Pagination Response Wrapper

Rather than exposing Spring Data's raw `Page<T>` directly in your API responses (which couples your API contract to Spring Data's internal shape), many teams wrap it in a custom, stable envelope.

```java
public record PagedResponse<T>(
        List<T> content,
        int page,
        int size,
        long totalElements,
        int totalPages,
        boolean last
) {
    public static <T> PagedResponse<T> from(Page<T> page) {
        return new PagedResponse<>(
                page.getContent(), page.getNumber(), page.getSize(),
                page.getTotalElements(), page.getTotalPages(), page.isLast());
    }
}
```

```java
@GetMapping
public PagedResponse<OrderResponse> getOrders(Pageable pageable) {
    Page<Order> orders = orderRepository.findAll(pageable);
    return PagedResponse.from(orders.map(orderMapper::toResponse));
}
```

## H.3 Custom Actuator Health Indicator Aggregating Multiple Checks

```java
@Component
public class DownstreamServicesHealthIndicator implements HealthIndicator {

    private final PaymentServiceClient paymentClient;
    private final InventoryServiceClient inventoryClient;

    @Override
    public Health health() {
        Health.Builder builder = Health.up();
        boolean allHealthy = true;

        if (!paymentClient.isReachable()) {
            builder.withDetail("payment-service", "DOWN");
            allHealthy = false;
        }
        if (!inventoryClient.isReachable()) {
            builder.withDetail("inventory-service", "DOWN");
            allHealthy = false;
        }

        return allHealthy ? builder.build() : builder.down().build();
    }
}
```

## H.4 Custom Jackson Serializer

Sometimes the default JSON serialization for a type isn't what you want — for example, always formatting money consistently, or masking sensitive data.

```java
public class MaskedStringSerializer extends JsonSerializer<String> {
    @Override
    public void serialize(String value, JsonGenerator gen, SerializerProvider provider) throws IOException {
        if (value == null || value.length() < 4) {
            gen.writeString("****");
        } else {
            gen.writeString("*".repeat(value.length() - 4) + value.substring(value.length() - 4));
        }
    }
}
```

```java
public class CustomerResponse {
    private String name;

    @JsonSerialize(using = MaskedStringSerializer.class)
    private String phoneNumber; // will serialize as "*******1234" instead of the raw number
}
```

## H.5 Internationalization (i18n)

For applications supporting multiple languages, Spring's `MessageSource` externalizes user-facing text into locale-specific resource bundles instead of hardcoding strings.

```properties
# messages.properties (default/English)
order.notfound=Order not found with id: {0}

# messages_es.properties (Spanish)
order.notfound=Pedido no encontrado con id: {0}
```

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final MessageSource messageSource;

    public Order findById(Long id) {
        return orderRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException(
                messageSource.getMessage("order.notfound", new Object[]{id}, LocaleContextHolder.getLocale())));
    }
}
```

Spring automatically resolves the correct locale (from the `Accept-Language` header, by default) and picks the matching resource bundle — falling back to the default `messages.properties` if a specific locale's translation isn't available.

## H.6 WebSocket for Real-Time Updates

For features needing genuine real-time, bidirectional communication (live order tracking, chat, live dashboards) rather than typical request-response REST calls, Spring supports WebSockets, often paired with the STOMP messaging protocol for a higher-level pub/sub model.

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic");
        registry.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").withSockJS();
    }
}
```

```java
@Controller
public class OrderStatusController {

    @MessageMapping("/order-status")
    @SendTo("/topic/order-updates")
    public OrderStatusUpdate broadcastUpdate(OrderStatusUpdate update) {
        return update; // broadcast to every client subscribed to /topic/order-updates
    }
}
```

## H.7 Server-Sent Events (SSE)

For simpler ONE-WAY real-time updates (server pushing data to the client, without needing the client to send messages back), Server-Sent Events are a much lighter-weight alternative to full WebSockets, built directly on plain HTTP.

```java
@GetMapping(value = "/orders/{id}/status-stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public SseEmitter streamOrderStatus(@PathVariable Long id) {
    SseEmitter emitter = new SseEmitter(Long.MAX_VALUE);
    orderStatusPublisher.subscribe(id, emitter);
    return emitter;
}
```

## H.8 Spring Retry (Declarative Retries Outside Messaging)

Beyond Kafka/RabbitMQ-specific retry mechanisms (Sections 25.7 and 26.5), Spring Retry provides a general-purpose, declarative retry mechanism usable for ANY method — like calling a flaky third-party HTTP API.

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

```java
@Retryable(
    retryFor = {ResourceAccessException.class},
    maxAttempts = 3,
    backoff = @Backoff(delay = 1000, multiplier = 2)
)
public ExchangeRateResponse getExchangeRate(String currency) {
    return restTemplate.getForObject("https://api.exchangerate.com/" + currency, ExchangeRateResponse.class);
}

@Recover
public ExchangeRateResponse fallbackRate(ResourceAccessException ex, String currency) {
    return ExchangeRateResponse.cached(currency); // called after all retry attempts are exhausted
}
```

## H.9 Correlation ID Propagation Across Service Calls

Building on the MDC pattern from Section 18.5, in a microservices architecture you want the SAME correlation/trace ID to propagate not just within one service's logs, but ACROSS every downstream service call triggered by the original request.

```java
public class CorrelationIdFeignInterceptor implements RequestInterceptor {
    @Override
    public void apply(RequestTemplate template) {
        String correlationId = MDC.get("correlationId");
        if (correlationId != null) {
            template.header("X-Correlation-Id", correlationId);
        }
    }
}
```

```java
public class CorrelationIdFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
            throws ServletException, IOException {
        String correlationId = Optional.ofNullable(request.getHeader("X-Correlation-Id"))
                .orElse(UUID.randomUUID().toString());
        MDC.put("correlationId", correlationId);
        response.setHeader("X-Correlation-Id", correlationId);
        try {
            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

With both pieces in place, a correlation ID generated at the very first service a request hits automatically propagates through every downstream Feign call, appearing consistently in every service's logs — invaluable for tracing one logical request across an entire microservices call chain (complementing the full distributed tracing setup from Section 27.9).

## H.10 Read Replica Routing for Scaling Reads

For read-heavy applications, routing READ queries to database read replicas (while writes go to the primary) can meaningfully reduce load on your primary database. Spring supports this via a routing `DataSource`.

```java
public class ReplicationRoutingDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return TransactionSynchronizationManager.isCurrentTransactionReadOnly() ? "replica" : "primary";
    }
}
```

```java
@Bean
public DataSource routingDataSource(DataSource primaryDataSource, DataSource replicaDataSource) {
    ReplicationRoutingDataSource routingDataSource = new ReplicationRoutingDataSource();
    routingDataSource.setTargetDataSources(Map.of("primary", primaryDataSource, "replica", replicaDataSource));
    routingDataSource.setDefaultTargetDataSource(primaryDataSource);
    return routingDataSource;
}
```

```java
@Transactional(readOnly = true)  // this hint is what determineCurrentLookupKey() checks
public List<Order> getOrderHistory(Long customerId) {
    return orderRepository.findByCustomerId(customerId);
}
```

Marking read-only service methods with `@Transactional(readOnly = true)` isn't just a performance hint to Hibernate (it can skip dirty-checking overhead) — combined with a routing `DataSource` like this, it also determines WHICH physical database the query actually hits.

## H.11 @DataJpaTest for Fast Repository-Layer Tests

A test "slice" (similar in spirit to `@WebMvcTest` from Section 19.3) specifically for testing the JPA/repository layer in isolation, backed by an embedded database by default, without loading your entire application context.

```java
@DataJpaTest
class OrderRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private OrderRepository orderRepository;

    @Test
    void shouldFindOrdersByStatus() {
        Order order = new Order();
        order.setStatus(OrderStatus.SHIPPED);
        entityManager.persistAndFlush(order);

        List<Order> results = orderRepository.findByStatus(OrderStatus.SHIPPED);

        assertThat(results).hasSize(1);
    }
}
```

`TestEntityManager` is a testing-focused variant of `EntityManager`, giving you convenient methods like `persistAndFlush()` for setting up test data directly, without needing the full repository/service layer just to arrange a test's starting state.

## H.12 Exception Hierarchy Design for a Larger Application

As an application grows, a well-designed exception hierarchy (rather than one flat pile of unrelated custom exceptions) makes global exception handling (Section 6.3) much cleaner.

```java
public abstract class ApplicationException extends RuntimeException {
    private final HttpStatus status;
    protected ApplicationException(String message, HttpStatus status) {
        super(message);
        this.status = status;
    }
    public HttpStatus getStatus() { return status; }
}

public class ResourceNotFoundException extends ApplicationException {
    public ResourceNotFoundException(String message) { super(message, HttpStatus.NOT_FOUND); }
}

public class DuplicateResourceException extends ApplicationException {
    public DuplicateResourceException(String message) { super(message, HttpStatus.CONFLICT); }
}

public class BusinessRuleViolationException extends ApplicationException {
    public BusinessRuleViolationException(String message) { super(message, HttpStatus.UNPROCESSABLE_ENTITY); }
}
```

```java
@ExceptionHandler(ApplicationException.class)
public ProblemDetail handleApplicationException(ApplicationException ex) {
    return ProblemDetail.forStatusAndDetail(ex.getStatus(), ex.getMessage());
}
```

Now, a SINGLE handler method in the global exception handler covers EVERY custom exception in your entire application — each individual exception subclass only needs to specify its message and the appropriate status, with zero duplicated handler boilerplate as new exception types are added over time.

---

# Appendix I: Production Readiness Checklist

Before shipping a Spring Boot application to production, run through this checklist. Each item links back to the section of this guide where it's covered in depth, so you can go deeper on anything that isn't already solid.

## I.1 Configuration & Secrets

- [ ] No secrets (passwords, API keys, tokens) are hardcoded anywhere in source code (Section 22.5)
- [ ] All environment-specific values are externalized via profiles/environment variables (Sections 1.12, 22.1–22.4)
- [ ] `spring.jpa.hibernate.ddl-auto` is set to `validate` or `none` in production — never `update` or `create-drop` (Section 22.3)
- [ ] A dedicated secrets manager (Vault, AWS Secrets Manager, etc.) is used for genuinely sensitive credentials, not just environment variables (Section 22.5)
- [ ] Configuration values have sensible, safe defaults where a missing value shouldn't crash startup unexpectedly

## I.2 Database

- [ ] Schema changes are managed exclusively through versioned migrations (Flyway/Liquibase), never manual changes or Hibernate auto-DDL (Section 9)
- [ ] Appropriate indexes exist on frequently filtered, sorted, and joined columns (Section 29.4)
- [ ] Connection pool size is tuned for expected concurrent load, not left at defaults blindly (Sections 29.1–29.2)
- [ ] N+1 query patterns have been reviewed and addressed for high-traffic endpoints (Section 8.14)
- [ ] Long-running or bulk operations use batching rather than one-row-at-a-time processing (Sections 8.16, 29.5–29.6)
- [ ] Backups are configured and, ideally, restore procedures have actually been tested at least once

## I.3 Security

- [ ] Passwords are hashed with BCrypt (or equivalent), never stored in plain text (Section 10.5)
- [ ] All input is validated server-side, never trusting client-side validation alone (Section 5)
- [ ] HTTPS is enforced everywhere, in every environment (Section 30.8)
- [ ] CORS is configured with an explicit allow-list of origins, not a blanket wildcard, especially when credentials are involved (Section 10.11)
- [ ] Actuator endpoints are properly secured, and only necessary endpoints are exposed (`management.endpoints.web.exposure.include` is NOT `*`) (Section 21.4)
- [ ] Dependencies are kept up to date and regularly scanned for known vulnerabilities
- [ ] JWT secrets are sufficiently long/random, and access token lifetimes are appropriately short (Sections 11.1, 11.7)
- [ ] Rate limiting is in place for public-facing or otherwise abuse-prone endpoints (Section 30.9)
- [ ] Error responses never leak stack traces or internal implementation details to clients (Section 6.7)

## I.4 Resilience & Reliability

- [ ] Transactions are used correctly, wrapping genuinely atomic units of work (Section 7.10)
- [ ] Non-idempotent operations (especially payments) support idempotency keys (Section 30.10)
- [ ] Circuit breakers protect calls to downstream services that could realistically fail or slow down (Sections 27.7–27.8)
- [ ] Retries with backoff are configured for transient failures, both for messaging (Sections 25.7, 26.5) and direct HTTP calls (Appendix H.8)
- [ ] Timeouts are explicitly configured for all outbound HTTP/database calls — no unbounded waits
- [ ] A graceful shutdown strategy is configured, so in-flight requests complete before the application stops (Section F.1)

## I.5 Observability

- [ ] Structured, leveled logging is in place, with sensitive data never logged (Sections 18.3–18.4)
- [ ] A correlation/trace ID is propagated across log lines for a single request, and ideally across service boundaries (Sections 18.5, Appendix H.9)
- [ ] Health checks (`/actuator/health`) reflect the TRUE health of the application and its critical dependencies (Section 21.1)
- [ ] Key business and technical metrics are exposed and scraped by a monitoring system (Sections 21.2–21.3)
- [ ] Alerting is configured for critical failure conditions — not just dashboards nobody watches proactively
- [ ] Distributed tracing is in place for any genuinely multi-service request flow (Section 27.9)

## I.6 Testing

- [ ] Core business logic has meaningful unit test coverage, not just "coverage percentage" for its own sake (Section 19.4)
- [ ] Critical user-facing flows have integration test coverage exercising real wiring (Section 19.5)
- [ ] Tests run against realistic infrastructure (via TestContainers) where database-specific behavior matters (Section 19.6)
- [ ] The test suite runs as part of CI on every change, and a failing test genuinely blocks deployment (Section 28.8)

## I.7 Performance

- [ ] JVM heap size is explicitly configured, and appropriate for the deployment environment's actual memory limits (Section 29.8)
- [ ] Caching is applied to genuinely expensive, frequently-repeated, rarely-changing lookups (Section 17)
- [ ] Any endpoint that could return a large result set supports pagination (Section 7.14)
- [ ] A load test has been run against realistic traffic patterns at least once before a major launch

## I.8 Deployment

- [ ] The application builds into a properly optimized, minimal container image via multi-stage builds (Section 23.5)
- [ ] Deployments are automated via CI/CD, not manual, ad-hoc procedures (Section 28.8)
- [ ] Rollback procedures exist and have been tested, not just assumed to work
- [ ] Configuration differences between dev/staging/production are ONLY in externalized config, never in different code branches or builds (Section 2.14)

---

# Appendix J: Common Design Scenarios

A set of realistic "how would you design this?" scenarios, walking through the reasoning and which concepts from this guide apply — good practice for both real project decisions and technical interviews.

## J.1 Scenario: "Design an API for an e-commerce checkout flow that must never double-charge a customer."

**Key concerns:** Idempotency, transactions, external payment gateway reliability.

**Approach:**
1. The client generates a unique idempotency key for the checkout attempt (Section 30.10), sent as a header.
2. The server checks if that key has already been processed; if so, returns the ORIGINAL result rather than reprocessing.
3. Order creation and payment charging happen within a properly scoped `@Transactional` boundary (Section 7.10) — though note payment gateway calls typically happen OUTSIDE the database transaction (since you can't "roll back" a real external charge the same way you roll back a database write), so the flow is usually: create a `PENDING` order → attempt payment → update order status based on the result, each step handling its own failure mode.
4. A circuit breaker (Section 27.7) wraps the call to the external payment gateway, with a sensible fallback (marking the order as "payment pending, retry later" rather than a hard failure) if the gateway is temporarily down.
5. Webhook handling from the payment provider (for asynchronous payment confirmation) should ALSO be idempotent, since providers commonly retry webhook delivery.

## J.2 Scenario: "Design a notification system that sends emails, SMS, and push notifications without slowing down the triggering request."

**Key concerns:** Decoupling, asynchronous processing, reliability of delivery.

**Approach:**
1. The triggering action (e.g., order placed) publishes an application event OR a Kafka event (Sections 31.5 and 25), rather than calling notification logic directly and synchronously.
2. Separate listener/consumer components handle each notification channel independently (Section A.8's `OrderNotificationListener` pattern, scaled up).
3. Each channel's actual sending logic wraps its own retry/circuit-breaker logic (Sections 25.7, 27.8), since email/SMS/push providers can all fail independently — a failure in SMS delivery shouldn't affect email delivery.
4. If using Kafka specifically, a dead letter queue (Section 25.8) captures permanently-failed notifications for later investigation or manual retry, rather than silently losing them.

## J.3 Scenario: "Design a system where multiple services need to know when an order's status changes, and new services may be added later without modifying the order service."

**Key concerns:** Loose coupling, extensibility, event-driven architecture.

**Approach:**
1. The order service publishes an `order.status.changed` event to a Kafka topic (Section 25) whenever a status transition occurs — it has ZERO knowledge of who's listening.
2. Each interested service (shipping, analytics, customer notifications, loyalty points) subscribes as its OWN independent consumer group (Section 25.4) — each gets its own full copy of every event.
3. Adding a brand-new downstream service later requires ZERO changes to the order service — it just starts consuming the existing topic with a new consumer group.
4. Message schemas should be versioned/evolved carefully (adding fields is safe; removing or renaming fields can break existing consumers) since many independent services depend on this contract.

## J.4 Scenario: "Design a reporting endpoint that aggregates data across three different services, and must respond within 500ms."

**Key concerns:** Parallel calls, timeouts, graceful degradation.

**Approach:**
1. Use `CompletableFuture` (Section 15.2) to call all three services CONCURRENTLY rather than sequentially — three 200ms calls made sequentially would total 600ms, but made in parallel, the total is closer to just 200ms (the slowest one).
2. Set an explicit timeout on EACH call — if the 500ms budget is at risk, better to return PARTIAL data (with a note that one section is unavailable) than to fail the entire report.
3. Wrap each downstream call in its own circuit breaker (Section 27.7), so a persistently failing service doesn't repeatedly eat into the time budget on every single request.
4. Consider caching (Section 17) any of the three data sources that doesn't need to be perfectly real-time, to reduce the number of live calls needed at all.

## J.5 Scenario: "Design authentication for a mobile app and a web app sharing the same backend, where the mobile app should stay logged in for weeks but the web app should log out after a period of inactivity."

**Key concerns:** Differentiated session/token lifetimes per client type, stateless vs. session-based tradeoffs.

**Approach:**
1. Use JWT-based, stateless authentication (Section 11) for BOTH clients, since it works cleanly across both native mobile and web without server-side session infrastructure.
2. Issue REFRESH tokens (Section 11.3) with DIFFERENT lifetimes per client type — a long-lived refresh token (weeks) for mobile, and a much shorter one (hours, tied to activity) for web — determined at login time based on a client-type indicator the app sends.
3. Access tokens themselves can share the same short lifetime (15 minutes) across both — the DIFFERENCE in "stay logged in" behavior comes entirely from how long the REFRESH token remains valid and gets silently renewed in the background.
4. For extra security on the longer-lived mobile refresh tokens, consider refresh token ROTATION (issuing a new refresh token each time the old one is used, invalidating the old one) to limit the damage if a long-lived token is ever stolen.

## J.6 Scenario: "Design a system that imports a 2-million-row CSV file nightly without timing out or exhausting memory."

**Key concerns:** Batch processing, memory efficiency, restart-on-failure.

**Approach:**
1. Use Spring Batch (Appendix G.3) rather than trying to read the entire file into memory and process it in a single pass — its chunk-oriented processing model reads, processes, and writes in small, bounded batches.
2. Stream the CSV file rather than loading it fully into memory — Spring Batch's `FlatFileItemReader` does this naturally.
3. Use batched database writes (Sections 8.16, 29.5) for the actual persistence step, not one `INSERT` per row.
4. Rely on Spring Batch's built-in job execution tracking to support RESTARTING from the last successful chunk if the job fails partway through, rather than needing to reprocess all 2 million rows from scratch.
5. Run the whole job asynchronously/on a schedule (Section 14), well outside of any user-facing request path.

---

# Appendix K: A Second Worked Example — "TaskFlow" (Reactive WebFlux)

Everything in Appendix A was built on traditional, blocking Spring MVC (Section 4) — which is the right default for the vast majority of applications. But it's worth seeing the REACTIVE alternative too, since it comes up often enough (especially for high-concurrency, I/O-bound workloads) that you should recognize the shape of it. Let's build a small task-management API, "TaskFlow," using Spring WebFlux instead.

## K.1 What's Actually Different

Traditional Spring MVC is THREAD-PER-REQUEST: each incoming request occupies a thread for its entire duration, including while WAITING on a slow database call or downstream HTTP call — meaning your maximum concurrency is bounded by your thread pool size.

Spring WebFlux is built on a REACTIVE, non-blocking model: a request's thread is released back to a small pool the moment it hits an I/O wait (a database call, an HTTP call), and resumed later when that I/O completes — allowing a MUCH smaller number of threads to handle a much larger number of concurrent in-flight requests, which matters a lot for I/O-heavy workloads at scale.

The tradeoff: reactive code (built around `Mono<T>` for a single value and `Flux<T>` for a stream of values) has a real learning curve, and EVERYTHING in the call chain — controller, service, repository, database driver — needs to be reactive/non-blocking end to end, or you lose the benefit entirely (one blocking call anywhere in the chain can stall a shared thread).

## K.2 Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-postgresql</artifactId>
</dependency>
```

Notice: instead of `spring-boot-starter-data-jpa` (which is fundamentally blocking, built around JDBC), we use `spring-boot-starter-data-r2dbc` — R2DBC is a genuinely reactive database driver specification, a non-blocking counterpart to JDBC. You CANNOT mix regular Spring Data JPA into a reactive pipeline without breaking the non-blocking guarantee.

## K.3 Reactive Entity & Repository

```java
@Table("tasks")
public record Task(
        @Id Long id,
        String title,
        boolean completed,
        LocalDateTime createdAt
) {}
```

```java
public interface TaskRepository extends ReactiveCrudRepository<Task, Long> {
    Flux<Task> findByCompleted(boolean completed);
}
```

`ReactiveCrudRepository` mirrors `CrudRepository` (Section 7.5) conceptually, but every method returns `Mono<T>` (zero-or-one result) or `Flux<T>` (zero-to-many results) instead of returning plain objects/lists directly — reflecting that the actual database call hasn't necessarily completed yet when the method returns.

## K.4 Reactive Service Layer

```java
@Service
@RequiredArgsConstructor
public class TaskService {

    private final TaskRepository taskRepository;

    public Mono<Task> createTask(String title) {
        Task task = new Task(null, title, false, LocalDateTime.now());
        return taskRepository.save(task);
    }

    public Flux<Task> getAllTasks() {
        return taskRepository.findAll();
    }

    public Mono<Task> completeTask(Long id) {
        return taskRepository.findById(id)
            .switchIfEmpty(Mono.error(new ResourceNotFoundException("Task not found: " + id)))
            .flatMap(task -> taskRepository.save(new Task(task.id(), task.title(), true, task.createdAt())));
    }
}
```

Notice there's no `.block()` anywhere here — the ENTIRE chain stays reactive. `flatMap` is the reactive equivalent of "then do this next, using the previous result," WITHOUT ever blocking a thread waiting for the previous step.

## K.5 Reactive Controller

```java
@RestController
@RequestMapping("/api/tasks")
@RequiredArgsConstructor
public class TaskController {

    private final TaskService taskService;

    @PostMapping
    public Mono<ResponseEntity<Task>> createTask(@RequestBody CreateTaskRequest request) {
        return taskService.createTask(request.title())
            .map(task -> ResponseEntity.status(HttpStatus.CREATED).body(task));
    }

    @GetMapping
    public Flux<Task> getAllTasks() {
        return taskService.getAllTasks(); // Spring streams this back to the client as it becomes available
    }

    @PatchMapping("/{id}/complete")
    public Mono<Task> completeTask(@PathVariable Long id) {
        return taskService.completeTask(id);
    }
}
```

A big conceptual shift from Section 4's controllers: the method doesn't return the DATA itself, it returns a DESCRIPTION of how to eventually get the data (`Mono`/`Flux`) — Spring WebFlux subscribes to it and streams the result back to the client once it resolves, without ever tying up a thread for the wait.

## K.6 When to Actually Reach for WebFlux

Being honest about the tradeoffs: WebFlux is NOT automatically "better" or "faster" for a typical CRUD application — for most applications, especially CPU-bound ones or ones with modest concurrency needs, traditional blocking Spring MVC is simpler to write, simpler to debug (reactive stack traces are notoriously harder to read), and performs perfectly well. WebFlux earns its complexity specifically for:

- Very high concurrency, I/O-bound workloads (thousands of simultaneous slow, mostly-idle connections — think chat servers, streaming APIs, or gateways proxying many slow backend calls)
- Systems already built on other reactive components (like a reactive message broker) where staying reactive end-to-end avoids awkward blocking/non-blocking boundary conversions
- Genuine, measured resource constraints where thread-per-request's memory overhead (each thread consumes real memory just to exist) becomes a real limiting factor

For the overwhelming majority of applications — including the OrderFlow example in Appendix A — traditional Spring MVC remains the right, pragmatic default.

---

# Appendix L: Spring Boot Version Notes & Migration Guidance

Spring Boot evolves continuously, and knowing the shape of recent major changes helps when reading slightly older code (or slightly older tutorials/Stack Overflow answers) and reconciling it with what's current.

## L.1 Spring Boot 2 → 3 (Major Breaking Changes)

- **Java baseline raised to Java 17** — Spring Boot 3 requires at least Java 17, dropping support for Java 8/11.
- **`javax.*` → `jakarta.*` namespace migration** — this is the single biggest source of confusion for anyone following older tutorials. Every `javax.persistence.*`, `javax.validation.*`, `javax.servlet.*` import changed to `jakarta.persistence.*`, `jakarta.validation.*`, `jakarta.servlet.*` respectively, following the broader Java EE → Jakarta EE rebrand. Code written against Spring Boot 2 using `javax.*` imports will NOT compile against Spring Boot 3 without this find-and-replace.
- **`WebSecurityConfigurerAdapter` removed** — the older, class-extension-based Spring Security configuration style (extending this class and overriding `configure(HttpSecurity)`) was removed entirely in favor of the `SecurityFilterChain` bean-based DSL shown throughout Section 10 of this guide.
- **Native image support (GraalVM) became a first-class feature** — Spring Boot 3 was built with ahead-of-time compilation and native image generation as a core supported scenario, dramatically reducing startup time and memory footprint for applications compiled this way (at some cost to build complexity and certain dynamic/reflection-heavy library compatibility).
- **Observability API introduced** — `ObservationRegistry` and related APIs unified metrics AND tracing instrumentation under one consistent abstraction, superseding some older, separate instrumentation approaches.

## L.2 Notable Spring Boot 3.x Minor Version Additions

- **Problem Details (RFC 7807) support** built in natively (Section 6.6), removing the need for third-party libraries to get standardized error responses.
- **Virtual Threads support** (leveraging Java 21's virtual threads/"Project Loom") — allowing TRADITIONAL, blocking-style Spring MVC code to gain much of reactive programming's concurrency benefit, WITHOUT rewriting anything in the reactive (`Mono`/`Flux`) style covered in Appendix K. Enabled with a single property:
```properties
spring.threads.virtual.enabled=true
```
This is a genuinely significant development — for many applications, it offers a path to WebFlux-like concurrency benefits while keeping the simpler, easier-to-debug blocking programming model.
- **Structured logging support** built in natively (Section 18.4), without needing a custom Logback encoder configuration.

## L.3 Practical Migration Checklist (Spring Boot 2 → 3)

1. Upgrade the JDK to 17 or later first, and confirm the whole build/toolchain supports it.
2. Run a global find-and-replace of `javax.persistence`, `javax.validation`, `javax.servlet`, `javax.annotation` (and related) to their `jakarta.*` equivalents.
3. Replace any `WebSecurityConfigurerAdapter`-based security configuration with the `SecurityFilterChain` bean style.
4. Update all third-party library versions to Jakarta-EE-compatible releases — an outdated library still using `javax.*` internally will conflict with Spring Boot 3's `jakarta.*` classpath.
5. Re-run the full test suite (Section 19) — this migration touches enough surface area that regressions are common even with mechanical find-and-replace changes.
6. Review and update any custom `HandlerInterceptor`, `Filter`, or servlet-API-touching code for the `jakarta.servlet` namespace change specifically, since these are easy to miss in an automated refactor if they're not obviously named.

---

# Appendix M: Quick Reference Cheat Sheets

Terse, scannable cheat sheets for the commands and snippets you'll reach for constantly during day-to-day Spring Boot development. Less explanation here than the rest of the guide — this appendix is meant to be scanned quickly, not read start to finish.

## M.1 Maven Commands

```bash
mvn clean                          # remove the target/ build output directory
mvn compile                        # compile source code only
mvn test                           # run unit tests
mvn package                        # build the executable JAR/WAR
mvn clean package                  # clean, then build fresh
mvn clean package -DskipTests      # build without running tests
mvn spring-boot:run                # run the application directly via Maven, without building a JAR first
mvn dependency:tree                # print the full dependency tree, useful for resolving version conflicts
mvn versions:display-dependency-updates   # check which dependencies have newer versions available
mvn -pl module-name -am clean install     # build a specific module and its dependencies in a multi-module project
mvn clean install -U               # force-update snapshot dependencies
```

## M.2 Gradle Commands

```bash
./gradlew clean                    # remove build output
./gradlew build                    # compile, test, and package
./gradlew bootRun                  # run the application directly
./gradlew test                     # run tests only
./gradlew build -x test            # build without running tests
./gradlew dependencies             # print the dependency tree
./gradlew bootJar                  # build just the executable JAR
```

## M.3 Docker & Docker Compose Commands

```bash
docker build -t myapp:1.0 .                    # build an image from the Dockerfile in the current directory
docker run -p 8080:8080 myapp:1.0               # run a container, mapping port 8080
docker run -d --name myapp -p 8080:8080 myapp:1.0   # run detached (in the background)
docker ps                                       # list running containers
docker logs -f myapp                            # follow a container's logs
docker exec -it myapp sh                        # open a shell inside a running container
docker stop myapp && docker rm myapp            # stop and remove a container
docker compose up -d                            # start the full stack defined in docker-compose.yml, detached
docker compose down                             # stop and remove everything started by compose
docker compose logs -f app                      # follow logs for a specific compose service
docker compose exec db psql -U postgres         # open a psql shell inside the running db container
docker image prune -a                           # clean up unused images to reclaim disk space
```

## M.4 curl Commands for Testing a REST API

```bash
curl http://localhost:8080/api/orders                                   # simple GET
curl -X POST http://localhost:8080/api/orders \
     -H "Content-Type: application/json" \
     -d '{"customerName":"Alice","items":[{"productId":1,"quantity":2}]}' # POST with JSON body

curl -X PATCH "http://localhost:8080/api/orders/5/status?status=SHIPPED"  # PATCH with query param

curl -H "Authorization: Bearer eyJhbGciOi..." http://localhost:8080/api/orders  # authenticated GET

curl -i http://localhost:8080/api/orders/999        # -i shows response headers + status code (useful for checking 404s)

curl -X DELETE http://localhost:8080/api/orders/5 -w "\nStatus: %{http_code}\n"  # DELETE, printing the status code explicitly
```

## M.4 Actuator Endpoints Quick Reference

```
GET /actuator/health              # overall + component health status
GET /actuator/info                # arbitrary build/app info you've configured
GET /actuator/metrics             # list of available metric names
GET /actuator/metrics/{name}      # detail for one specific metric
GET /actuator/env                 # current environment properties (SECURE this — sensitive!)
GET /actuator/beans               # every bean currently in the application context
GET /actuator/mappings            # every registered request mapping
GET /actuator/loggers             # current logging levels, and lets you change them at RUNTIME
POST /actuator/loggers/{name}     # change a logger's level live, without restarting the app
GET /actuator/threaddump          # a full thread dump, useful for diagnosing hangs
GET /actuator/heapdump            # downloads a heap dump for memory analysis (SECURE this — heavy + sensitive!)
GET /actuator/prometheus          # metrics formatted for Prometheus scraping
```

## M.5 Common JPQL / Derived Query Method Patterns

```java
// Derived query keywords quick reference
findByStatus(OrderStatus status)
findByStatusAndCustomerId(OrderStatus status, Long customerId)
findByStatusOrPriority(OrderStatus status, String priority)
findByTotalAmountGreaterThan(BigDecimal amount)
findByTotalAmountBetween(BigDecimal min, BigDecimal max)
findByCustomerNameContaining(String partial)
findByCustomerNameStartingWith(String prefix)
findByCreatedAtAfter(LocalDateTime date)
findByStatusIn(List<OrderStatus> statuses)
findByStatusIsNull()
findByStatusIsNotNull()
findAllByOrderByCreatedAtDesc()
countByStatus(OrderStatus status)
existsByCustomerEmail(String email)
deleteByStatus(OrderStatus status)
findTop5ByOrderByCreatedAtDesc()
findFirstByStatusOrderByCreatedAtAsc(OrderStatus status)
```

```java
// Common JPQL patterns
@Query("SELECT o FROM Order o WHERE o.status = :status")
@Query("SELECT o FROM Order o JOIN FETCH o.items WHERE o.id = :id")
@Query("SELECT new com.myapp.dto.OrderSummary(o.id, o.status) FROM Order o")  // DTO projection
@Query("SELECT COUNT(o) FROM Order o WHERE o.status = :status")
@Query("SELECT o.status, COUNT(o) FROM Order o GROUP BY o.status")
@Modifying
@Query("UPDATE Order o SET o.status = :status WHERE o.id = :id")
@Modifying
@Query("DELETE FROM Order o WHERE o.status = :status")
```

## M.6 Common HTTP Status Codes for REST APIs

```
200 OK                      - successful GET/PUT/PATCH
201 Created                 - successful POST creating a resource
202 Accepted                - request accepted for async processing
204 No Content               - successful DELETE, or success with no body
400 Bad Request              - malformed request / validation failure
401 Unauthorized             - missing or invalid authentication
403 Forbidden                - authenticated, but insufficient permissions
404 Not Found                - resource doesn't exist
405 Method Not Allowed        - HTTP method not supported for this URL
409 Conflict                  - duplicate resource, version conflict
422 Unprocessable Entity      - well-formed request, but semantically invalid
429 Too Many Requests         - rate limit exceeded
500 Internal Server Error     - unexpected server-side failure
502 Bad Gateway               - upstream service returned an invalid response
503 Service Unavailable       - server temporarily unable to handle the request
504 Gateway Timeout           - upstream service didn't respond in time
```

## M.7 Common Spring Security Expressions (for @PreAuthorize)

```java
@PreAuthorize("hasRole('ADMIN')")
@PreAuthorize("hasAnyRole('ADMIN', 'MANAGER')")
@PreAuthorize("hasAuthority('orders:delete')")
@PreAuthorize("isAuthenticated()")
@PreAuthorize("isAnonymous()")
@PreAuthorize("#userId == authentication.principal.id")
@PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
@PreAuthorize("@orderSecurityService.canAccess(#orderId, authentication)")  // delegate to a custom bean
```

## M.8 Common Test Assertions (JUnit 5 + AssertJ)

```java
assertEquals(expected, actual);
assertNotEquals(expected, actual);
assertTrue(condition);
assertFalse(condition);
assertNull(value);
assertNotNull(value);
assertThrows(SomeException.class, () -> someMethod());
assertDoesNotThrow(() -> someMethod());

// AssertJ, fluent style — often preferred for readability
assertThat(order.getStatus()).isEqualTo(OrderStatus.SHIPPED);
assertThat(orders).hasSize(3);
assertThat(orders).isNotEmpty();
assertThat(orders).extracting(Order::getStatus).containsOnly(OrderStatus.SHIPPED);
assertThat(exception.getMessage()).contains("not found");
assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
```

## M.9 Common Logback Log Pattern Placeholders

```
%d{yyyy-MM-dd HH:mm:ss}   - timestamp
%thread                   - thread name
%-5level                  - log level, left-padded to 5 chars
%logger{36}               - logger name, truncated to 36 chars
%msg                      - the actual log message
%n                        - newline
%X{requestId}             - an MDC value (Section 18.5)
%ex                       - exception stack trace (included automatically if an exception is logged)
```

## M.10 Git Commit Message Conventions Commonly Paired with CI/CD Pipelines

```
feat: add order cancellation endpoint
fix: correct discount calculation rounding error
refactor: extract payment logic into dedicated service
test: add integration tests for order creation
docs: update API documentation for v2 endpoints
chore: bump spring boot to 3.4.0
perf: add index on orders.status column
security: patch dependency with known CVE
```

Many CI/CD pipelines and changelog-generation tools parse these conventional prefixes automatically to categorize changes and even determine version bumps (e.g., `feat:` triggering a minor version bump, `fix:` a patch bump) — worth adopting consistently across a team even outside of any specific tooling requirement, purely for a more scannable commit history.

---

# Appendix N: Full Topic Index

A flat index of every single topic from the original curriculum, mapped to the section where it's covered — useful as a final "did I cover everything?" checklist, and as a fast lookup table.

| # | Topic | Section |
|---|---|---|
| 1 | Introduction to Spring | 1.1 |
| 2 | Inversion of Control (IoC) | 1.2 |
| 3 | Dependency Injection (DI) | 1.3 |
| 4 | Bean | 1.4 |
| 5 | Bean Lifecycle | 1.5 |
| 6 | Spring Container | 1.6 |
| 7 | ApplicationContext | 1.7 |
| 8 | Bean Scopes | 1.8 |
| 9 | Spring Annotations | 1.9 |
| 10 | Java Configuration | 1.10 |
| 11 | Component Scanning | 1.11 |
| 12 | Profiles | 1.12 |
| 13 | Environment & Properties | 1.13 |
| 14 | External Configuration | 1.14 |
| 15 | What is Spring Boot | 2.1 |
| 16 | Spring Boot Architecture | 2.2 |
| 17 | Spring Initializr | 2.3 |
| 18 | Starter Dependencies | 2.4 |
| 19 | Auto Configuration | 2.5 |
| 20 | Embedded Servers | 2.6 |
| 21 | application.properties | 2.7 |
| 22 | application.yml | 2.8 |
| 23 | Configuration Properties | 2.9 |
| 24 | CommandLineRunner | 2.10 |
| 25 | Banner Customization | 2.11 |
| 26 | Spring Boot DevTools | 2.12 |
| 27 | REST Principles | 3.1 |
| 28 | HTTP Methods | 3.2 |
| 29 | Request Mapping | 3.3 |
| 30 | Path Variables | 3.4 |
| 31 | Request Parameters | 3.5 |
| 32 | Request Body | 3.6 |
| 33 | ResponseEntity | 3.7 |
| 34 | HttpStatus | 3.8 |
| 35 | Response DTO | 3.9 |
| 36 | Request DTO | 3.10 |
| 37 | REST Best Practices | 3.11 |
| 38 | API Versioning | 3.12 |
| 39 | DispatcherServlet | 4.1 |
| 40 | Controller | 4.2 |
| 41 | RestController | 4.3 |
| 42 | RequestMapping (MVC) | 4.4 |
| 43 | Handler Mapping | 4.5 |
| 44 | View Resolver | 4.6 |
| 45 | Data Binding | 4.7 |
| 46 | Validation (MVC) | 4.8 |
| 47 | Exception Handling (MVC) | 4.9 |
| 48 | Interceptors | 4.10 |
| 49 | Bean Validation | 5.1 |
| 50 | Jakarta Validation | 5.2 |
| 51 | Validation Annotations | 5.3 |
| 52 | Custom Validation | 5.4 |
| 53 | Validation Groups | 5.5 |
| 54 | Global Validation Errors | 5.6 |
| 55 | Checked vs Unchecked | 6.1 |
| 56 | Custom Exceptions | 6.2 |
| 57 | Global Exception Handler | 6.3 |
| 58 | @ControllerAdvice | 6.4 |
| 59 | @ExceptionHandler | 6.5 |
| 60 | Problem Details API | 6.6 |
| 61 | Error Response Design | 6.7 |
| 62 | ORM | 7.1 |
| 63 | Hibernate | 7.2 |
| 64 | Entity | 7.3 |
| 65 | Repository | 7.4 |
| 66 | CrudRepository | 7.5 |
| 67 | JpaRepository | 7.6 |
| 68 | Entity Lifecycle | 7.7 |
| 69 | Persistence Context | 7.8 |
| 70 | Dirty Checking | 7.9 |
| 71 | Transactions | 7.10 |
| 72 | JPQL | 7.11 |
| 73 | Native Queries | 7.12 |
| 74 | Derived Queries | 7.13 |
| 75 | Pagination | 7.14 |
| 76 | Sorting | 7.15 |
| 77 | Specifications | 7.16 |
| 78 | Entity Graph | 7.17 |
| 79 | Lazy Loading | 8.1 |
| 80 | Eager Loading | 8.2 |
| 81 | Fetch Types | 8.3 |
| 82 | Cascade Types | 8.4 |
| 83 | Entity Relationships | 8.5 |
| 84 | OneToOne | 8.6 |
| 85 | OneToMany | 8.7 |
| 86 | ManyToOne | 8.8 |
| 87 | ManyToMany | 8.9 |
| 88 | Composite Keys | 8.10 |
| 89 | Embeddables | 8.11 |
| 90 | Optimistic Locking | 8.12 |
| 91 | Pessimistic Locking | 8.13 |
| 92 | N+1 Problem | 8.14 |
| 93 | Caching (Hibernate) | 8.15 |
| 94 | Batch Processing | 8.16 |
| 95 | Flyway | 9.1 |
| 96 | Liquibase | 9.2 |
| 97 | Versioned Migrations | 9.3 |
| 98 | Rollback Strategy | 9.4 |
| 99 | Spring Security Basics | 10.1 |
| 100 | Authentication | 10.2 |
| 101 | Authorization | 10.3 |
| 102 | Password Encoding | 10.4 |
| 103 | BCrypt | 10.5 |
| 104 | UserDetailsService | 10.6 |
| 105 | Security Filter Chain | 10.7 |
| 106 | Roles | 10.8 |
| 107 | Authorities | 10.9 |
| 108 | Method Security | 10.10 |
| 109 | CORS | 10.11 |
| 110 | CSRF | 10.12 |
| 111 | Session Authentication | 10.13 |
| 112 | Stateless Authentication | 10.14 |
| 113 | JWT Structure | 11.1 |
| 114 | Access Token | 11.2 |
| 115 | Refresh Token | 11.3 |
| 116 | Token Validation | 11.4 |
| 117 | JWT Filter | 11.5 |
| 118 | Custom Authentication | 11.6 |
| 119 | Token Expiration | 11.7 |
| 120 | Logout Strategy | 11.8 |
| 121 | Multipart File | 12.1 |
| 122 | File Storage | 12.2 |
| 123 | Image Upload | 12.3 |
| 124 | Cloud Storage | 12.4 |
| 125 | Download Files | 12.5 |
| 126 | Spring Mail | 13.1 |
| 127 | SMTP | 13.2 |
| 128 | HTML Emails | 13.3 |
| 129 | Attachments | 13.4 |
| 130 | OTP Email | 13.5 |
| 131 | SMS Integration | 13.6 |
| 132 | WhatsApp Integration | 13.7 |
| 133 | Scheduled Tasks | 14.1 |
| 134 | Cron Expressions | 14.2 |
| 135 | Async Jobs | 14.3 |
| 136 | Thread Pool Scheduling | 14.4 |
| 137 | @Async | 15.1 |
| 138 | CompletableFuture | 15.2 |
| 139 | ExecutorService | 15.3 |
| 140 | ThreadPoolTaskExecutor | 15.4 |
| 141 | Aspect Oriented Programming | 16.1 |
| 142 | Advice | 16.2 |
| 143 | Pointcut | 16.3 |
| 144 | Join Point | 16.4 |
| 145 | Logging Aspect | 16.5 |
| 146 | Performance Monitoring | 16.6 |
| 147 | Spring Cache | 17.1 |
| 148 | Cache Manager | 17.2 |
| 149 | Redis Cache | 17.3 |
| 150 | Eviction | 17.4 |
| 151 | TTL | 17.5 |
| 152 | SLF4J | 18.1 |
| 153 | Logback | 18.2 |
| 154 | Log Levels | 18.3 |
| 155 | Structured Logging | 18.4 |
| 156 | MDC | 18.5 |
| 157 | Log Rotation | 18.6 |
| 158 | JUnit 5 | 19.1 |
| 159 | Mockito | 19.2 |
| 160 | MockMvc | 19.3 |
| 161 | Unit Testing | 19.4 |
| 162 | Integration Testing | 19.5 |
| 163 | TestContainers | 19.6 |
| 164 | API Testing | 19.7 |
| 165 | Swagger | 20.1 |
| 166 | OpenAPI | 20.2 |
| 167 | API Documentation | 20.3 |
| 168 | API Examples | 20.4 |
| 169 | Health Checks | 21.1 |
| 170 | Metrics | 21.2 |
| 171 | Monitoring | 21.3 |
| 172 | Custom Endpoints | 21.4 |
| 173 | Development (profile) | 22.1 |
| 174 | Testing (profile) | 22.2 |
| 175 | Production (profile) | 22.3 |
| 176 | Environment Variables | 22.4 |
| 177 | Secrets Management | 22.5 |
| 178 | Docker Basics | 23.1 |
| 179 | Dockerfile | 23.2 |
| 180 | Docker Compose | 23.3 |
| 181 | Container Networking | 23.4 |
| 182 | Multi-stage Builds | 23.5 |
| 183 | Redis Basics | 24.1 |
| 184 | Spring Data Redis | 24.2 |
| 185 | Cache (Redis) | 24.3 |
| 186 | Session Store | 24.4 |
| 187 | Kafka Basics | 25.1 |
| 188 | Producer | 25.2 |
| 189 | Consumer | 25.3 |
| 190 | Consumer Groups | 25.4 |
| 191 | Partitions | 25.5 |
| 192 | Offset | 25.6 |
| 193 | Retry (Kafka) | 25.7 |
| 194 | Dead Letter Queue | 25.8 |
| 195 | Transactions (Kafka) | 25.9 |
| 196 | Exactly Once | 25.10 |
| 197 | Exchanges | 26.1 |
| 198 | Queues | 26.2 |
| 199 | Routing Keys | 26.3 |
| 200 | Acknowledgements | 26.4 |
| 201 | Retry (RabbitMQ) | 26.5 |
| 202 | Microservices Architecture | 27.1 |
| 203 | Service Discovery | 27.2 |
| 204 | Eureka | 27.3 |
| 205 | API Gateway | 27.4 |
| 206 | Config Server | 27.5 |
| 207 | OpenFeign | 27.6 |
| 208 | Circuit Breaker | 27.7 |
| 209 | Resilience4j | 27.8 |
| 210 | Distributed Tracing | 27.9 |
| 211 | Centralized Logging | 27.10 |
| 212 | Service Communication | 27.11 |
| 213 | AWS EC2 | 28.1 |
| 214 | AWS RDS | 28.2 |
| 215 | AWS S3 | 28.3 |
| 216 | IAM | 28.4 |
| 217 | Elastic Beanstalk | 28.5 |
| 218 | ECS | 28.6 |
| 219 | Docker Deployment | 28.7 |
| 220 | CI/CD Basics | 28.8 |
| 221 | Connection Pooling | 29.1 |
| 222 | HikariCP | 29.2 |
| 223 | Query Optimization | 29.3 |
| 224 | Indexing | 29.4 |
| 225 | Batch Inserts | 29.5 |
| 226 | Batch Updates | 29.6 |
| 227 | Profiling | 29.7 |
| 228 | JVM Memory Tuning | 29.8 |
| 229 | Layered Architecture | 30.1 |
| 230 | Clean Code | 30.2 |
| 231 | SOLID Principles | 30.3 |
| 232 | DTO Mapping | 30.4 |
| 233 | MapStruct | 30.5 |
| 234 | Global Response Structure | 30.6 |
| 235 | API Standards | 30.7 |
| 236 | Security Best Practices | 30.8 |
| 237 | Rate Limiting | 30.9 |
| 238 | Idempotency | 30.10 |
| 239 | Audit Logging | 30.11 |
| 240 | Singleton | 31.1 |
| 241 | Factory | 31.2 |
| 242 | Builder | 31.3 |
| 243 | Strategy | 31.4 |
| 244 | Observer | 31.5 |
| 245 | Proxy | 31.6 |
| 246 | Template Method | 31.7 |
| 247 | Dependency Injection Pattern | 31.8 |
| 248 | Auto Configuration Internals | 32.1 |
| 249 | Spring Boot Startup Process | 32.2 |
| 250 | Bean Factory | 32.3 |
| 251 | Bean Post Processor | 32.4 |
| 252 | Application Events | 32.5 |
| 253 | Spring Context Refresh | 32.6 |
| 254 | Conditional Beans | 32.7 |
| 255 | Custom Starter Creation | 32.8 |
| 256 | Spring Boot Auto Configuration Creation | 32.9 |

Every single topic from the original curriculum outline is accounted for above, each mapped to the section where it was covered in full depth, with working code examples, explained in plain English.

---

# Appendix O: Complete Environment-Specific Configuration Files

To close the loop on Appendix A's OrderFlow project, here are genuinely complete `application.yml` files for all three environments discussed in Section 22 — every property annotated inline so you can see exactly why each one is set the way it is.

## O.1 application.yml (Base / Shared Configuration)

```yaml
spring:
  application:
    name: orderflow-api

  jackson:
    default-property-inclusion: non_null      # omit null fields from JSON responses
    serialization:
      write-dates-as-timestamps: false         # use ISO-8601 strings, not epoch millis, for dates

  jpa:
    open-in-view: false                        # explicit fetching, avoids masking N+1 issues
    properties:
      hibernate:
        jdbc:
          batch_size: 50
        order_inserts: true
        order_updates: true

  flyway:
    enabled: true
    locations: classpath:db/migration

server:
  error:
    include-stacktrace: never                  # never leak stack traces to clients, in ANY environment
    include-message: never

management:
  endpoints:
    web:
      exposure:
        include: health,info

app:
  jwt:
    access-token-expiration: 900000            # 15 minutes
    refresh-token-expiration: 604800000        # 7 days
```

## O.2 application-dev.yml

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/orderflow_dev
    username: dev_user
    password: dev_password

  jpa:
    show-sql: true                             # see generated SQL directly in console — helpful locally
    hibernate:
      ddl-auto: validate                       # still validate against Flyway migrations, even locally

  mail:
    host: localhost
    port: 1025                                 # local dev SMTP catcher, e.g. Mailhog/Mailpit

logging:
  level:
    root: INFO
    com.orderflow: DEBUG
    org.hibernate.SQL: DEBUG

management:
  endpoints:
    web:
      exposure:
        include: "*"                           # fine to expose everything locally — never in prod

app:
  jwt:
    secret: dev-only-insecure-secret-key-not-for-production-use
```

## O.3 application-test.yml

```yaml
spring:
  datasource:
    url: jdbc:tc:postgresql:16:///orderflow_test   # TestContainers JDBC URL, spins up a real Postgres for tests

  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate

  mail:
    host: localhost
    port: 3025

logging:
  level:
    root: WARN                                 # keep test output clean and scannable
    com.orderflow: INFO

app:
  jwt:
    secret: test-secret-key-for-automated-tests-only
    access-token-expiration: 5000              # short-lived, to make expiration logic easy to test deterministically
```

## O.4 application-prod.yml

```yaml
spring:
  datasource:
    url: ${DATABASE_URL}
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD}
    hikari:
      maximum-pool-size: 20
      minimum-idle: 10
      connection-timeout: 30000
      leak-detection-threshold: 60000

  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate

  mail:
    host: ${SMTP_HOST}
    port: 587
    username: ${SMTP_USERNAME}
    password: ${SMTP_PASSWORD}
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true

server:
  shutdown: graceful
  compression:
    enabled: true

logging:
  level:
    root: WARN
    com.orderflow: INFO
  file:
    name: /var/log/orderflow/application.log

management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus     # deliberately narrow — no wildcard exposure in production
  endpoint:
    health:
      show-details: when-authorized

app:
  jwt:
    secret: ${JWT_SECRET}                      # pulled from a real secrets manager at deploy time, never hardcoded
```

Notice the pattern across all four files: the BASE file holds settings that are TRUE EVERYWHERE (never leak stack traces, never let `open-in-view` mask bugs); each PROFILE-SPECIFIC file only overrides what's genuinely different for that environment (database credentials, logging verbosity, actuator exposure). This mirrors exactly the guidance from Section 22 — the same packaged artifact, differently configured per environment, never a different build per environment.

---

# Appendix P: OrderFlow API — Full Request/Response Reference

A complete, practical reference of every endpoint in the OrderFlow API from Appendix A, with realistic example requests and responses — useful both as documentation and as a template for how to document your OWN APIs (tying back to Section 20's OpenAPI/documentation guidance).

## P.1 Create an Order

```
POST /api/v1/orders
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
Content-Type: application/json

{
  "customerId": 42,
  "items": [
    { "productId": 7, "quantity": 2 },
    { "productId": 12, "quantity": 1 }
  ]
}
```

Successful response:

```
HTTP/1.1 201 Created
Content-Type: application/json

{
  "id": 501,
  "customerName": "Alice Johnson",
  "status": "PENDING",
  "items": [
    { "productName": "Wireless Mouse", "quantity": 2, "unitPrice": 24.99, "subtotal": 49.98 },
    { "productName": "USB-C Hub", "quantity": 1, "unitPrice": 39.99, "subtotal": 39.99 }
  ],
  "totalAmount": 89.97,
  "createdAt": "2026-07-28T14:32:10"
}
```

Insufficient stock response:

```
HTTP/1.1 409 Conflict
Content-Type: application/problem+json

{
  "type": "about:blank",
  "title": "Conflict",
  "status": 409,
  "detail": "Not enough stock for product: Wireless Mouse"
}
```

Validation failure response:

```
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json

{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "fieldErrors": {
    "customerId": "must not be null",
    "items": "must not be empty"
  }
}
```

## P.2 Get a Single Order

```
GET /api/v1/orders/501
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 501,
  "customerName": "Alice Johnson",
  "status": "PENDING",
  "items": [ ... ],
  "totalAmount": 89.97,
  "createdAt": "2026-07-28T14:32:10"
}
```

Not found response:

```
HTTP/1.1 404 Not Found
Content-Type: application/problem+json

{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "Order not found: 9999"
}
```

## P.3 Get a Customer's Order History (Paginated)

```
GET /api/v1/orders/customer/42?page=0&size=10&sort=createdAt,desc
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "content": [
    { "id": 501, "customerName": "Alice Johnson", "status": "PENDING", "totalAmount": 89.97, "createdAt": "2026-07-28T14:32:10" },
    { "id": 487, "customerName": "Alice Johnson", "status": "DELIVERED", "totalAmount": 154.00, "createdAt": "2026-07-15T09:12:44" }
  ],
  "page": 0,
  "size": 10,
  "totalElements": 2,
  "totalPages": 1,
  "last": true
}
```

## P.4 Update Order Status (Admin Only)

```
PATCH /api/v1/orders/501/status?status=SHIPPED
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 501,
  "customerName": "Alice Johnson",
  "status": "SHIPPED",
  "items": [ ... ],
  "totalAmount": 89.97,
  "createdAt": "2026-07-28T14:32:10"
}
```

Insufficient permissions response (non-admin user):

```
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json

{
  "type": "about:blank",
  "title": "Forbidden",
  "status": 403,
  "detail": "Access Denied"
}
```

## P.5 Authentication Endpoints

```
POST /api/v1/auth/login
Content-Type: application/json

{
  "username": "alice@example.com",
  "password": "correct-horse-battery-staple"
}
```

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhbGljZSJ9...",
  "refreshToken": "8f14e45fceea167a5a36dedd4bea2543...",
  "expiresIn": 900
}
```

```
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refreshToken": "8f14e45fceea167a5a36dedd4bea2543..."
}
```

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhbGljZSJ9...(new)",
  "refreshToken": "8f14e45fceea167a5a36dedd4bea2543...",
  "expiresIn": 900
}
```

Invalid credentials response:

```
HTTP/1.1 401 Unauthorized
Content-Type: application/problem+json

{
  "type": "about:blank",
  "title": "Unauthorized",
  "status": 401,
  "detail": "Invalid username or password"
}
```

---

# Appendix Q: Extended Interview Questions (Part 2)

Continuing Appendix E with the remaining topic areas — caching, messaging, DevOps, and design patterns — for complete interview preparation coverage.

## Q.1 Caching

**Q: What's the difference between `@Cacheable` and `@CachePut`?**
A: `@Cacheable` SKIPS executing the method entirely on a cache hit, returning the cached value directly. `@CachePut` ALWAYS executes the method, and uses the result to update the cache — appropriate for update operations where you need the method to genuinely run, but also want the cache refreshed with the new value (Section 17.1).

**Q: Why would you choose Redis over a local in-memory cache?**
A: Any application running multiple instances needs a SHARED cache — a local in-memory cache would be inconsistent across instances, with each one caching independently. Redis, being external and shared, solves this (Section 17.3).

**Q: How do you decide on an appropriate cache TTL?**
A: Balance staleness tolerance against cache hit rate — rarely-changing data can use a long TTL for maximum cache effectiveness; frequently-changing data needs either a short TTL or explicit eviction on update, to avoid serving incorrect stale data (Section 17.5).

## Q.2 Messaging (Kafka & RabbitMQ)

**Q: When would you choose Kafka over RabbitMQ?**
A: Kafka excels at high-throughput event streaming where multiple independent consumers need to read the SAME stream of events (each consumer group gets its own full copy), and where message RETENTION/replay matters. RabbitMQ excels at more traditional task-queue/routing scenarios with flexible routing logic (direct/topic/fanout exchanges) and doesn't retain messages after successful consumption by default. Neither is universally "better" — the right choice depends on the specific messaging pattern needed (Sections 25 and 26).

**Q: What guarantees does Kafka provide about message ordering?**
A: Kafka guarantees ordering WITHIN a single partition only, not across an entire topic — which is why choosing a consistent partition key (like an order ID) matters for any messages that need strict relative ordering (Section 25.5).

**Q: What's a Dead Letter Queue, and why is it important?**
A: A separate queue/topic where messages that have permanently failed processing (exhausted all retries) get routed, instead of being silently lost or endlessly blocking the main processing flow — giving you visibility and a chance to investigate or manually reprocess failures (Sections 25.8 and 26.5).

## Q.3 DevOps & Deployment

**Q: Why use multi-stage Docker builds for a Spring Boot application?**
A: The build stage needs the full JDK and build tools; the runtime stage only needs the much smaller JRE and the compiled JAR. Multi-stage builds let you use a heavy build environment for compilation while producing a much smaller, leaner final image (Section 23.5).

**Q: What's the difference between `fixedRate` and `fixedDelay` in `@Scheduled`?**
A: `fixedRate` schedules based on when the PREVIOUS execution STARTED; `fixedDelay` waits for the previous execution to FINISH before counting down to the next run (Section 14.1).

**Q: How does HikariCP's leak detection help in production?**
A: It logs a warning (with a stack trace pointing to where the connection was acquired) if a checked-out connection isn't returned within a configured threshold — making connection leak bugs, which are otherwise very hard to diagnose, immediately visible (Section 29.2).

## Q.4 Design Patterns

**Q: How does Spring implement the Singleton pattern?**
A: Every Spring bean defaults to `singleton` scope automatically — Spring's container manages a single shared instance per application context, without you writing any manual singleton boilerplate (Sections 1.8 and 31.1).

**Q: Give an example of the Proxy pattern in Spring, and why it matters for understanding `@Async`'s limitations.**
A: `@Transactional`, `@Async`, and `@Cacheable` all work by wrapping your bean in a PROXY that intercepts method calls to add extra behavior. This is exactly why calling an `@Async` method from WITHIN the same class (self-invocation) doesn't actually run asynchronously — you're bypassing the proxy and calling the real object directly (Sections 31.6, 15.1).

**Q: How does the Observer pattern show up in Spring's application event system?**
A: A component publishes an `ApplicationEvent` via `ApplicationEventPublisher`, with zero knowledge of who's listening; any number of `@EventListener`-annotated methods can independently react to that same event — classic Observer pattern decoupling (Section 31.5).

---

# Appendix R: One-Paragraph Section Recaps (Full Guide Review)

A condensed, one-paragraph summary of every one of the 32 main sections — useful as a final review pass, or as a quick refresher before diving back into a specific area of a real project.

**Section 1 — Spring Framework Core:** Spring's foundation is Inversion of Control, achieved through Dependency Injection — the container creates and wires your objects ("beans") for you, managing their full lifecycle, so your code depends on abstractions rather than manually constructing its own dependencies.

**Section 2 — Spring Boot Fundamentals:** Spring Boot sits on top of Spring, eliminating manual configuration through auto-configuration (smart defaults based on your classpath), starter dependencies (bundled, compatible libraries), and an embedded server — turning a multi-day setup into a matter of minutes.

**Section 3 — REST API Development:** Well-designed REST APIs represent resources as nouns in URLs, use standard HTTP methods to represent actions, return meaningful status codes via `ResponseEntity`, and separate their public contract (DTOs) from internal data models.

**Section 4 — Spring MVC:** Every request passes through the `DispatcherServlet`, gets routed to the right controller method via `HandlerMapping`, and — for REST APIs specifically — `@RestController` combines controller behavior with automatic JSON serialization.

**Section 5 — Validation:** Jakarta Bean Validation lets you declare constraints directly on DTO fields, automatically enforced via `@Valid`, with custom validators and validation groups available for business-specific or context-dependent rules.

**Section 6 — Exception Handling:** Centralizing exception-to-HTTP-response translation via `@ControllerAdvice` and `@ExceptionHandler` avoids repetitive try/catch blocks scattered across every controller, and modern Spring supports the standardized RFC 7807 Problem Details format out of the box.

**Section 7 — Spring Data JPA:** Repositories provide CRUD operations essentially for free, derived query methods let you express many queries just through method naming, and understanding the entity lifecycle (transient/managed/detached/removed) and dirty checking explains a lot of JPA's sometimes-surprising automatic behavior.

**Section 8 — Hibernate Advanced:** Fetch types (LAZY vs EAGER), cascade behavior, and relationship mapping are where most real-world Hibernate performance and correctness issues live — especially the notorious N+1 query problem, fixed via `JOIN FETCH` or `@EntityGraph`.

**Section 9 — Database Migration:** Flyway and Liquibase manage schema evolution through small, versioned, NEVER-edited-after-the-fact scripts — ensuring every environment's database structure stays consistent and auditable over the life of a project.

**Section 10 — Spring Security:** A chain of servlet filters handles authentication (who are you?) and authorization (what can you do?) declaratively, with BCrypt for password hashing, and a clean split between session-based (stateful) and token-based (stateless) authentication models.

**Section 11 — JWT Authentication:** Short-lived, self-contained, cryptographically-signed access tokens paired with longer-lived, server-revocable refresh tokens balance security and user convenience in a fully stateless authentication scheme.

**Section 12 — File Upload:** `MultipartFile` handles incoming file uploads, which should generally be stored in cloud object storage (like S3) rather than local disk for any application running multiple instances.

**Section 13 — Email & Notifications:** `JavaMailSender` handles email sending (plain, HTML, with attachments), while SMS and WhatsApp typically integrate through dedicated third-party provider APIs rather than any direct protocol Spring itself implements.

**Section 14 — Scheduling:** `@Scheduled` methods (fixed rate, fixed delay, or cron-based) run automatically on a configurable schedule, ideally backed by a properly sized thread pool rather than the single-threaded default.

**Section 15 — Asynchronous Programming:** `@Async` runs a method on a separate thread from a configured `ThreadPoolTaskExecutor`, with `CompletableFuture` letting you compose and wait on results from concurrently-running operations — watch out for the self-invocation proxy gotcha.

**Section 16 — Spring AOP:** Cross-cutting concerns (logging, timing, security) are defined once as "advice" applied at matching "pointcuts," avoiding duplicated boilerplate scattered across many otherwise-unrelated classes.

**Section 17 — Caching:** `@Cacheable`/`@CachePut`/`@CacheEvict` provide a declarative caching abstraction, with Redis as the standard choice for a distributed cache shared consistently across multiple application instances.

**Section 18 — Logging:** SLF4J is the logging facade, Logback the default implementation; structured (JSON) logging and MDC-propagated correlation IDs are essential for making logs genuinely useful in a production, multi-request, multi-service environment.

**Section 19 — Testing:** Fast, isolated unit tests (with Mockito mocks) form the base of a healthy test pyramid, with `@WebMvcTest`/`@SpringBootTest`/`@DataJpaTest` slices and TestContainers-backed integration tests providing progressively higher-fidelity (and progressively slower) confidence.

**Section 20 — OpenAPI Documentation:** Springdoc auto-generates interactive Swagger UI documentation directly from your controllers, further enriched with `@Operation`/`@Schema` annotations and concrete examples for genuinely useful API documentation.

**Section 21 — Spring Boot Actuator:** Built-in production-ready endpoints (`/actuator/health`, `/actuator/metrics`) expose operational visibility, powering load balancer health checks and Micrometer-based monitoring dashboards — always secured and scoped carefully in production.

**Section 22 — Spring Boot Profiles:** Environment-specific configuration (dev/test/prod) is externalized via profiles and environment variables, with genuinely sensitive secrets pulled from a dedicated secrets manager rather than plain config files.

**Section 23 — Docker:** Multi-stage Dockerfiles produce small, production-ready images; Docker Compose orchestrates an entire local development stack (app + database + cache) with one command.

**Section 24 — Redis:** Beyond caching, Redis serves as a fast, shared session store — essential for any application running multiple server instances behind a load balancer.

**Section 25 — Kafka:** A distributed, partitioned, retained event log enabling multiple independent consumer groups to each process the full stream of events, with retry and dead-letter-queue patterns handling processing failures gracefully.

**Section 26 — RabbitMQ:** A traditional message broker routing messages through exchanges to queues based on routing keys, with manual acknowledgement modes providing reliable, at-least-once delivery guarantees.

**Section 27 — Microservices:** Breaking a monolith into independently deployable services trades operational simplicity for team/deployment independence, requiring real infrastructure investment in service discovery, API gateways, circuit breakers, and distributed tracing to manage the resulting complexity well.

**Section 28 — Cloud Deployment:** From raw EC2 (maximum control, maximum responsibility) through managed platforms (Elastic Beanstalk, ECS) to fully automated CI/CD pipelines, cloud deployment options trade control for operational convenience along a clear spectrum.

**Section 29 — Performance Optimization:** Properly sized connection pools, well-chosen database indexes, batched writes, and measured (not guessed) profiling are the highest-leverage places to look when a Spring Boot application needs to get faster.

**Section 30 — Production Best Practices:** Layered architecture, SOLID principles, consistent DTO mapping (via MapStruct), standardized error/response formats, idempotency keys for non-idempotent operations, and audit logging collectively separate a "working" application from a genuinely production-grade one.

**Section 31 — Design Patterns in Spring:** Spring's own architecture is built on classic design patterns (Singleton via bean scope, Factory via `@Bean` methods, Proxy via AOP, Observer via application events) — recognizing these patterns demystifies a lot of Spring's internal behavior.

**Section 32 — Spring Boot Internals:** Auto-configuration, the bean lifecycle, `BeanPostProcessor`, and the context refresh process are the actual mechanisms underlying everything covered in this guide — understanding them (and even building your own custom starter) is what separates confident Spring Boot usage from cargo-culting annotations that happen to work.

---

*This concludes the complete OrderFlow Spring Boot Tutorial — every one of the 256 curriculum topics covered in depth, plus a full worked project, troubleshooting guide, glossary, annotation reference, interview preparation, configuration reference, and design-scenario walkthroughs.*

# Appendix S: OrderFlow — Remaining Source Files

Appendix A walked through OrderFlow's core order-management flow, but referenced a few supporting classes (the JWT filter, auth endpoints, product/customer management) without showing their full implementation. Here they are, completing the project end to end.

## S.1 Main Application Class

```java
@SpringBootApplication
public class OrderFlowApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderFlowApplication.class, args);
    }
}
```

## S.2 JWT Service

```java
@Service
public class JwtService {

    @Value("${app.jwt.secret}")
    private String secretKey;

    @Value("${app.jwt.access-token-expiration}")
    private long accessTokenExpiration;

    private SecretKey getSigningKey() {
        return Keys.hmacShaKeyFor(secretKey.getBytes(StandardCharsets.UTF_8));
    }

    public String generateAccessToken(String username, Collection<? extends GrantedAuthority> authorities) {
        List<String> roles = authorities.stream().map(GrantedAuthority::getAuthority).toList();

        return Jwts.builder()
                .setSubject(username)
                .claim("roles", roles)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + accessTokenExpiration))
                .signWith(getSigningKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    public String extractUsername(String token) {
        return extractAllClaims(token).getSubject();
    }

    public boolean isTokenValid(String token) {
        try {
            extractAllClaims(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }
}
```

## S.3 JWT Authentication Filter

```java
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final CustomUserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                     FilterChain filterChain) throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7);

        if (jwtService.isTokenValid(token)) {
            String username = jwtService.extractUsername(token);
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);

            UsernamePasswordAuthenticationToken authToken =
                    new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
            authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));

            SecurityContextHolder.getContext().setAuthentication(authToken);
        }

        filterChain.doFilter(request, response);
    }
}
```

## S.4 Custom UserDetailsService

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final CustomerRepository customerRepository;

    @Override
    public UserDetails loadUserByUsername(String email) {
        Customer customer = customerRepository.findByEmail(email)
                .orElseThrow(() -> new UsernameNotFoundException("No account found for: " + email));

        return org.springframework.security.core.userdetails.User
                .withUsername(customer.getEmail())
                .password(customer.getPasswordHash())
                .authorities(customer.isAdmin() ? "ROLE_ADMIN" : "ROLE_USER")
                .build();
    }
}
```

## S.5 Auth Controller

```java
@RestController
@RequestMapping("/api/v1/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final JwtService jwtService;
    private final RefreshTokenService refreshTokenService;

    @PostMapping("/login")
    public ResponseEntity<TokenResponse> login(@Valid @RequestBody LoginRequest request) {
        Authentication auth = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(request.username(), request.password()));

        String accessToken = jwtService.generateAccessToken(auth.getName(), auth.getAuthorities());
        String refreshToken = refreshTokenService.createRefreshToken(auth.getName());

        return ResponseEntity.ok(new TokenResponse(accessToken, refreshToken, 900));
    }

    @PostMapping("/refresh")
    public ResponseEntity<TokenResponse> refresh(@Valid @RequestBody RefreshRequest request) {
        String username = refreshTokenService.validateAndGetUsername(request.refreshToken())
                .orElseThrow(() -> new BadCredentialsException("Invalid or expired refresh token"));

        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        String newAccessToken = jwtService.generateAccessToken(username, userDetails.getAuthorities());

        return ResponseEntity.ok(new TokenResponse(newAccessToken, request.refreshToken(), 900));
    }

    @PostMapping("/logout")
    public ResponseEntity<Void> logout(@Valid @RequestBody RefreshRequest request) {
        refreshTokenService.revoke(request.refreshToken());
        return ResponseEntity.noContent().build();
    }
}
```

```java
public record LoginRequest(@NotBlank String username, @NotBlank String password) {}
public record RefreshRequest(@NotBlank String refreshToken) {}
public record TokenResponse(String accessToken, String refreshToken, long expiresIn) {}
```

## S.6 Product Controller & Service

```java
@RestController
@RequestMapping("/api/v1/products")
@RequiredArgsConstructor
public class ProductController {

    private final ProductService productService;

    @GetMapping
    public ResponseEntity<List<ProductResponse>> getAllProducts() {
        return ResponseEntity.ok(productService.getAllInStock());
    }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<ProductResponse> createProduct(@Valid @RequestBody CreateProductRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(productService.create(request));
    }

    @PatchMapping("/{id}/stock")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<ProductResponse> restockProduct(@PathVariable Long id, @RequestParam int quantity) {
        return ResponseEntity.ok(productService.addStock(id, quantity));
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;
    private final ProductMapper productMapper;

    public List<ProductResponse> getAllInStock() {
        return productRepository.findByStockQuantityGreaterThan(0).stream()
                .map(productMapper::toResponse)
                .toList();
    }

    @Transactional
    public ProductResponse create(CreateProductRequest request) {
        Product product = new Product();
        product.setName(request.name());
        product.setPrice(request.price());
        product.setStockQuantity(request.initialStock());
        return productMapper.toResponse(productRepository.save(product));
    }

    @Transactional
    public ProductResponse addStock(Long id, int quantity) {
        Product product = productRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Product not found: " + id));
        product.setStockQuantity(product.getStockQuantity() + quantity);
        return productMapper.toResponse(product); // dirty checking saves this automatically
    }
}
```

```java
public record CreateProductRequest(
        @NotBlank String name,
        @Positive BigDecimal price,
        @PositiveOrZero int initialStock
) {}

public record ProductResponse(Long id, String name, BigDecimal price, int stockQuantity) {}
```

## S.7 Customer Controller

```java
@RestController
@RequestMapping("/api/v1/customers")
@RequiredArgsConstructor
public class CustomerController {

    private final CustomerService customerService;

    @PostMapping("/register")
    public ResponseEntity<CustomerResponse> register(@Valid @RequestBody RegisterCustomerRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(customerService.register(request));
    }

    @GetMapping("/me")
    public ResponseEntity<CustomerResponse> getCurrentCustomer(Authentication authentication) {
        return ResponseEntity.ok(customerService.getByEmail(authentication.getName()));
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class CustomerService {

    private final CustomerRepository customerRepository;
    private final PasswordEncoder passwordEncoder;
    private final CustomerMapper customerMapper;

    @Transactional
    public CustomerResponse register(RegisterCustomerRequest request) {
        if (customerRepository.findByEmail(request.email()).isPresent()) {
            throw new DuplicateResourceException("An account already exists for: " + request.email());
        }

        Customer customer = new Customer();
        customer.setName(request.name());
        customer.setEmail(request.email());
        customer.setPasswordHash(passwordEncoder.encode(request.password()));

        return customerMapper.toResponse(customerRepository.save(customer));
    }

    public CustomerResponse getByEmail(String email) {
        Customer customer = customerRepository.findByEmail(email)
                .orElseThrow(() -> new ResourceNotFoundException("Customer not found"));
        return customerMapper.toResponse(customer);
    }
}
```

```java
public record RegisterCustomerRequest(
        @NotBlank String name,
        @Email @NotBlank String email,
        @Pattern(regexp = "^(?=.*[A-Z])(?=.*\\d).{8,}$",
                 message = "Password needs 8+ characters, an uppercase letter, and a digit") String password
) {}

public record CustomerResponse(Long id, String name, String email) {}
```

## S.8 Custom Exception Classes

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) { super(message); }
}

public class DuplicateResourceException extends RuntimeException {
    public DuplicateResourceException(String message) { super(message); }
}

public class InsufficientStockException extends RuntimeException {
    public InsufficientStockException(String message) { super(message); }
}
```

## S.9 Final Project Directory Structure

```
orderflow-api/
├── pom.xml
├── Dockerfile
├── docker-compose.yml
└── src/
    ├── main/
    │   ├── java/com/orderflow/
    │   │   ├── OrderFlowApplication.java
    │   │   ├── config/
    │   │   │   ├── SecurityConfig.java
    │   │   │   └── CacheConfig.java
    │   │   ├── controller/
    │   │   │   ├── OrderController.java
    │   │   │   ├── ProductController.java
    │   │   │   ├── CustomerController.java
    │   │   │   └── AuthController.java
    │   │   ├── service/
    │   │   │   ├── OrderService.java
    │   │   │   ├── ProductService.java
    │   │   │   ├── CustomerService.java
    │   │   │   ├── JwtService.java
    │   │   │   ├── RefreshTokenService.java
    │   │   │   └── CustomUserDetailsService.java
    │   │   ├── repository/
    │   │   │   ├── OrderRepository.java
    │   │   │   ├── ProductRepository.java
    │   │   │   └── CustomerRepository.java
    │   │   ├── entity/
    │   │   │   ├── Order.java
    │   │   │   ├── OrderItem.java
    │   │   │   ├── Product.java
    │   │   │   ├── Customer.java
    │   │   │   └── OrderStatus.java
    │   │   ├── dto/
    │   │   │   ├── CreateOrderRequest.java
    │   │   │   ├── OrderResponse.java
    │   │   │   └── ... (remaining DTOs)
    │   │   ├── mapper/
    │   │   │   └── OrderMapper.java
    │   │   ├── exception/
    │   │   │   ├── ResourceNotFoundException.java
    │   │   │   ├── DuplicateResourceException.java
    │   │   │   ├── InsufficientStockException.java
    │   │   │   └── GlobalExceptionHandler.java
    │   │   ├── security/
    │   │   │   └── JwtAuthFilter.java
    │   │   └── event/
    │   │       ├── OrderCreatedEvent.java
    │   │       └── OrderNotificationListener.java
    │   └── resources/
    │       ├── application.yml
    │       ├── application-dev.yml
    │       ├── application-test.yml
    │       ├── application-prod.yml
    │       └── db/migration/
    │           ├── V1__create_customers_table.sql
    │           ├── V2__create_products_table.sql
    │           └── V3__create_orders_and_items_tables.sql
    └── test/
        └── java/com/orderflow/
            ├── service/OrderServiceTest.java
            └── controller/OrderControllerIntegrationTest.java
```

This structure follows the layered architecture principle from Section 30.1 exactly — every class lives in a package named for its ROLE (`controller`, `service`, `repository`, `entity`), not for a specific feature, which is the most common convention in small-to-medium Spring Boot applications. (Larger applications sometimes switch to package-by-FEATURE instead — grouping all of `order`'s controller/service/repository together — a valid alternative organizational strategy once a codebase grows large enough that package-by-layer starts to feel unwieldy.)

---

*Guide complete. Every topic from the original curriculum, an entire worked project (twice — once blocking, once reactive), troubleshooting reference, glossary, annotation index, interview prep, and full source code for a real, runnable application.*

# Appendix T: Postman-Style Request Collection (JSON)

For teams that prefer testing APIs through Postman/Insomnia rather than curl, here's the OrderFlow API expressed as an importable collection. This also serves as one more complete, scannable reference of every endpoint's exact shape.

```json
{
  "info": {
    "name": "OrderFlow API",
    "description": "Complete request collection for the OrderFlow order management API"
  },
  "item": [
    {
      "name": "Auth",
      "item": [
        {
          "name": "Register Customer",
          "request": {
            "method": "POST",
            "header": [{ "key": "Content-Type", "value": "application/json" }],
            "url": "{{baseUrl}}/api/v1/customers/register",
            "body": {
              "mode": "raw",
              "raw": "{\n  \"name\": \"Alice Johnson\",\n  \"email\": \"alice@example.com\",\n  \"password\": \"SecurePass123\"\n}"
            }
          }
        },
        {
          "name": "Login",
          "request": {
            "method": "POST",
            "header": [{ "key": "Content-Type", "value": "application/json" }],
            "url": "{{baseUrl}}/api/v1/auth/login",
            "body": {
              "mode": "raw",
              "raw": "{\n  \"username\": \"alice@example.com\",\n  \"password\": \"SecurePass123\"\n}"
            }
          }
        },
        {
          "name": "Refresh Token",
          "request": {
            "method": "POST",
            "header": [{ "key": "Content-Type", "value": "application/json" }],
            "url": "{{baseUrl}}/api/v1/auth/refresh",
            "body": {
              "mode": "raw",
              "raw": "{\n  \"refreshToken\": \"{{refreshToken}}\"\n}"
            }
          }
        },
        {
          "name": "Logout",
          "request": {
            "method": "POST",
            "header": [{ "key": "Content-Type", "value": "application/json" }],
            "url": "{{baseUrl}}/api/v1/auth/logout",
            "body": {
              "mode": "raw",
              "raw": "{\n  \"refreshToken\": \"{{refreshToken}}\"\n}"
            }
          }
        }
      ]
    },
    {
      "name": "Products",
      "item": [
        {
          "name": "List In-Stock Products",
          "request": {
            "method": "GET",
            "header": [{ "key": "Authorization", "value": "Bearer {{accessToken}}" }],
            "url": "{{baseUrl}}/api/v1/products"
          }
        },
        {
          "name": "Create Product (Admin)",
          "request": {
            "method": "POST",
            "header": [
              { "key": "Authorization", "value": "Bearer {{accessToken}}" },
              { "key": "Content-Type", "value": "application/json" }
            ],
            "url": "{{baseUrl}}/api/v1/products",
            "body": {
              "mode": "raw",
              "raw": "{\n  \"name\": \"Wireless Mouse\",\n  \"price\": 24.99,\n  \"initialStock\": 100\n}"
            }
          }
        },
        {
          "name": "Restock Product (Admin)",
          "request": {
            "method": "PATCH",
            "header": [{ "key": "Authorization", "value": "Bearer {{accessToken}}" }],
            "url": "{{baseUrl}}/api/v1/products/7/stock?quantity=50"
          }
        }
      ]
    },
    {
      "name": "Orders",
      "item": [
        {
          "name": "Create Order",
          "request": {
            "method": "POST",
            "header": [
              { "key": "Authorization", "value": "Bearer {{accessToken}}" },
              { "key": "Content-Type", "value": "application/json" }
            ],
            "url": "{{baseUrl}}/api/v1/orders",
            "body": {
              "mode": "raw",
              "raw": "{\n  \"customerId\": 42,\n  \"items\": [\n    { \"productId\": 7, \"quantity\": 2 },\n    { \"productId\": 12, \"quantity\": 1 }\n  ]\n}"
            }
          }
        },
        {
          "name": "Get Order By Id",
          "request": {
            "method": "GET",
            "header": [{ "key": "Authorization", "value": "Bearer {{accessToken}}" }],
            "url": "{{baseUrl}}/api/v1/orders/501"
          }
        },
        {
          "name": "Get Customer Order History",
          "request": {
            "method": "GET",
            "header": [{ "key": "Authorization", "value": "Bearer {{accessToken}}" }],
            "url": "{{baseUrl}}/api/v1/orders/customer/42?page=0&size=10&sort=createdAt,desc"
          }
        },
        {
          "name": "Update Order Status (Admin)",
          "request": {
            "method": "PATCH",
            "header": [{ "key": "Authorization", "value": "Bearer {{accessToken}}" }],
            "url": "{{baseUrl}}/api/v1/orders/501/status?status=SHIPPED"
          }
        }
      ]
    },
    {
      "name": "Customers",
      "item": [
        {
          "name": "Get Current Customer",
          "request": {
            "method": "GET",
            "header": [{ "key": "Authorization", "value": "Bearer {{accessToken}}" }],
            "url": "{{baseUrl}}/api/v1/customers/me"
          }
        }
      ]
    },
    {
      "name": "Actuator",
      "item": [
        {
          "name": "Health Check",
          "request": { "method": "GET", "url": "{{baseUrl}}/actuator/health" }
        },
        {
          "name": "Metrics",
          "request": { "method": "GET", "url": "{{baseUrl}}/actuator/metrics" }
        }
      ]
    }
  ],
  "variable": [
    { "key": "baseUrl", "value": "http://localhost:8080" },
    { "key": "accessToken", "value": "" },
    { "key": "refreshToken", "value": "" }
  ]
}
```

## T.1 Seed Data Script for Local Development

To pair with the collection above, here's a simple SQL seed script (run manually, or via a `dev`-profile-only `CommandLineRunner`, per Section 2.10) to have realistic data ready immediately after a fresh local setup.

```sql
INSERT INTO customers (name, email, password_hash) VALUES
('Alice Johnson', 'alice@example.com', '$2a$10$examplebcrypthashvalue1'),
('Bob Smith', 'bob@example.com', '$2a$10$examplebcrypthashvalue2'),
('Carol Davis', 'carol@example.com', '$2a$10$examplebcrypthashvalue3');

INSERT INTO products (name, price, stock_quantity) VALUES
('Wireless Mouse', 24.99, 150),
('Mechanical Keyboard', 89.99, 60),
('USB-C Hub', 39.99, 200),
('27" Monitor', 299.99, 25),
('Webcam HD', 54.99, 80),
('Laptop Stand', 34.99, 120),
('Noise-Cancelling Headphones', 199.99, 40),
('Desk Lamp', 29.99, 90),
('Ergonomic Chair', 349.99, 15),
('Portable SSD 1TB', 119.99, 70);
```

```java
@Component
@Profile("dev")
@RequiredArgsConstructor
public class DevDataSeeder implements CommandLineRunner {

    private final CustomerRepository customerRepository;
    private final ProductRepository productRepository;
    private final PasswordEncoder passwordEncoder;

    @Override
    public void run(String... args) {
        if (customerRepository.count() > 0) {
            return; // already seeded, don't duplicate on every restart
        }

        Customer alice = new Customer();
        alice.setName("Alice Johnson");
        alice.setEmail("alice@example.com");
        alice.setPasswordHash(passwordEncoder.encode("SecurePass123"));
        customerRepository.save(alice);

        Product mouse = new Product();
        mouse.setName("Wireless Mouse");
        mouse.setPrice(BigDecimal.valueOf(24.99));
        mouse.setStockQuantity(150);
        productRepository.save(mouse);

        System.out.println("Dev seed data loaded: " + customerRepository.count() + " customers, "
                + productRepository.count() + " products.");
    }
}
```

Using `@Profile("dev")` here (Section 1.12) is important — this seeding logic should NEVER run in production, where real customer/product data already exists and must not be tampered with by a development convenience script.

---

*Final word count note: this guide, including every appendix, represents a genuinely complete reference — from Spring's most foundational concept (Inversion of Control) through a full, runnable, tested, Dockerized, documented production application. Bookmark it, and revisit specific sections as you build.*
