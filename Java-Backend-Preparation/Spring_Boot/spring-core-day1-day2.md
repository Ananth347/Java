# Spring Core — Day 1 & Day 2 Study Guide

---

# DAY 1 — Spring Fundamentals

## 1. Spring vs Spring Boot

### Level 1 — Definition
Spring is a large, general-purpose framework for building Java applications. Its core job is to manage objects for you (Dependency Injection) and provide modules for web apps, data access, security, and more. Spring Boot is built **on top of** Spring — it's not a replacement, it's a convenience layer that sets up Spring for you automatically.

### Level 2 — Purpose
Plain Spring requires a lot of manual setup: you have to configure beans, servers, dependencies, and XML/Java config by hand. This is powerful but slow to start with. Spring Boot exists to remove that setup pain — you get a working, production-ready app in minutes using "opinionated defaults" (sensible choices made for you) and auto-configuration.

### Level 3 — Internal Working
Spring Boot scans your classpath (the libraries you've added) and automatically configures beans based on what it finds. For example, if it sees a database driver on the classpath, it auto-configures a DataSource bean for you — you didn't have to write that code. This happens through a mechanism called **auto-configuration**, driven by conditional checks (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, etc.). Spring Boot also embeds a server (like Tomcat) directly inside your app, so you don't need to deploy a separate WAR file to an external server.

### Level 4 — Code
Plain Spring (manual, verbose):
```java
AnnotationConfigApplicationContext context =
    new AnnotationConfigApplicationContext(AppConfig.class);
MyService service = context.getBean(MyService.class);
```
Spring Boot (automatic):
```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```
That one annotation `@SpringBootApplication` bundles `@Configuration`, `@ComponentScan`, and `@EnableAutoConfiguration` together.

### Level 5 — Problem Solving
- **Symptom:** "It works in plain Spring but Spring Boot auto-configures something I didn't want."
  **Fix:** Exclude it explicitly: `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)`.
- **Symptom:** "My app is heavier / slower to start than expected."
  **Cause:** Boot loads many auto-configurations even if unused. Trim unused starter dependencies.
- **Symptom:** "Confusing which config actually took effect."
  **Fix:** Run with `--debug` flag to print the auto-configuration report.

### Level 6 — Interview Answer
"Spring is the core framework providing IoC, DI, and various modules like Spring MVC and Spring Data. Spring Boot sits on top of it and eliminates boilerplate configuration through auto-configuration, starter dependencies, and an embedded server — so you can go from zero to a running application very quickly, without losing any of Spring's underlying power."

---

## 2. IoC and DI

### Level 1 — Definition
**IoC (Inversion of Control)** is a design principle: instead of your code creating and controlling its own dependencies, control is handed over to a container/framework. **DI (Dependency Injection)** is the specific technique Spring uses to implement IoC — the framework "injects" (supplies) the objects a class needs, rather than the class creating them itself.

### Level 2 — Purpose
Without IoC, if `OrderService` needs a `PaymentGateway`, you'd write `new PaymentGateway()` inside `OrderService`. This tightly couples the two classes, making testing and swapping implementations hard. IoC/DI decouples them — `OrderService` just declares "I need a `PaymentGateway`," and Spring decides which implementation to hand it. This makes code more modular, testable (easy to inject mocks), and maintainable.

### Level 3 — Internal Working
Spring maintains a container (the `ApplicationContext`) that holds instances of your classes (called **beans**). At startup, Spring scans for classes marked as components, creates instances of them, resolves their dependencies (by matching types/names), and wires them together — all before your code ever calls `new`. This wiring graph is built once, typically at application startup.

### Level 4 — Code
```java
// Without DI (tight coupling)
class OrderService {
    private PaymentGateway gateway = new StripeGateway(); // hardcoded
}

// With DI (Spring manages it)
@Service
class OrderService {
    private final PaymentGateway gateway;

    @Autowired
    public OrderService(PaymentGateway gateway) {
        this.gateway = gateway; // Spring supplies this
    }
}
```

### Level 5 — Problem Solving
- **Symptom:** `NoSuchBeanDefinitionException` — Spring can't find a bean to inject.
  **Cause:** Class isn't annotated (`@Component`/`@Service`) or isn't in a scanned package.
- **Symptom:** `NoUniqueBeanDefinitionException` — multiple beans match the same type.
  **Fix:** Use `@Qualifier` or `@Primary` to disambiguate.
- **Symptom:** Circular dependency error (Bean A needs B, B needs A).
  **Fix:** Redesign to break the cycle, or use setter injection instead of constructor injection as a workaround.

### Level 6 — Interview Answer
"IoC means the framework, not my code, controls object creation and wiring. Dependency Injection is how Spring implements that — it constructs my objects' dependencies and injects them, usually via the constructor. This decouples classes from their concrete dependencies, which makes the code far easier to test and change."

---

## 3. ApplicationContext

### Level 1 — Definition
The `ApplicationContext` is Spring's IoC container — the central object that creates, configures, and manages all your beans for the lifetime of the application.

### Level 2 — Purpose
Someone has to actually hold and manage all these beans, resolve their dependencies, and hand them out when needed. That "someone" is the `ApplicationContext`. It's also the gateway to extra features beyond basic DI: event publishing, internationalization (i18n), resource loading, and environment/property access.

### Level 3 — Internal Working
When the application starts, the `ApplicationContext`:
1. Reads configuration (Java config classes, XML, or annotations).
2. Scans for bean definitions (metadata describing each bean — its class, scope, dependencies).
3. Instantiates beans (respecting dependency order).
4. Injects dependencies into each bean.
5. Runs any post-processing and lifecycle callbacks (like `@PostConstruct`).
6. Keeps the fully-wired beans ready to be retrieved via `getBean()` or injected elsewhere.

`BeanFactory` is the more basic, lower-level container interface; `ApplicationContext` extends it with enterprise features and is what's used in practice.

### Level 4 — Code
```java
ApplicationContext context =
    new AnnotationConfigApplicationContext(AppConfig.class);

MyService service = context.getBean(MyService.class);
service.doWork();
```
In Spring Boot this is handled for you: `SpringApplication.run()` creates and returns the `ApplicationContext` internally.

### Level 5 — Problem Solving
- **Symptom:** `getBean()` throws "no such bean" even though the class has `@Component`.
  **Cause:** The class's package isn't included in component scanning — check `@ComponentScan` base packages.
- **Symptom:** Beans seem to be created twice.
  **Cause:** Accidentally creating a second `ApplicationContext` (e.g., in tests) instead of reusing one.

### Level 6 — Interview Answer
"The ApplicationContext is Spring's IoC container — it's responsible for creating beans, injecting their dependencies, managing their lifecycle, and giving the application access to features like events and externalized configuration. It's the object that makes IoC actually happen at runtime."

---

## 4. Bean Lifecycle

### Level 1 — Definition
The bean lifecycle is the sequence of steps Spring follows to create, initialize, use, and eventually destroy a bean.

### Level 2 — Purpose
Beans often need setup logic (e.g., opening a connection) after their dependencies are injected, and cleanup logic (e.g., closing that connection) before the application shuts down. Understanding the lifecycle tells you exactly when your dependencies are guaranteed to be ready, and where to hook in custom logic safely.

### Level 3 — Internal Working
Simplified order of events for each bean:
1. **Instantiation** — Spring calls the constructor.
2. **Populate properties** — dependencies are injected (fields/setters).
3. **Aware interfaces** (if implemented) — e.g., `BeanNameAware` gets told its bean name.
4. **BeanPostProcessor "before" hooks** — run for all beans.
5. **`@PostConstruct` method** (or `InitializingBean.afterPropertiesSet()`) — your custom init logic runs here.
6. **BeanPostProcessor "after" hooks**.
7. Bean is now fully ready and in use.
8. **`@PreDestroy` method** (or `DisposableBean.destroy()`) — runs when the context shuts down.

### Level 4 — Code
```java
@Component
public class DatabaseConnector {

    @PostConstruct
    public void init() {
        System.out.println("Opening connection...");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Closing connection...");
    }
}
```

### Level 5 — Problem Solving
- **Symptom:** `@PreDestroy` never runs.
  **Cause:** The context wasn't closed properly (e.g., in a test, or a non-web app that didn't call `context.close()`). Spring Boot web apps handle this automatically on shutdown.
- **Symptom:** `NullPointerException` inside `@PostConstruct`.
  **Cause:** Trying to use a dependency that hasn't been injected yet — double-check injection happened before this point (it should have, but check for circular dependency issues).

### Level 6 — Interview Answer
"A bean goes through instantiation, dependency injection, then any `@PostConstruct` initialization, then it's ready for use, and finally `@PreDestroy` cleanup when the container shuts down. This lets me safely put setup code in `@PostConstruct` knowing all dependencies are already injected, and cleanup code in `@PreDestroy`."

---

## 5. Bean Scopes

### Level 1 — Definition
Bean scope defines **how many instances** of a bean Spring creates and **how long** each instance lives. The two most common are `singleton` (one shared instance for the whole application) and `prototype` (a new instance every time it's requested).

### Level 2 — Purpose
Not every object should be shared. A stateless service (like a calculator) is safe and efficient as a singleton. But something that holds per-request or per-user state shouldn't be shared across everyone — that would cause data leaking between users. Scopes let you control this tradeoff.

### Level 3 — Internal Working
- **singleton** (default): Spring creates the bean once at startup (or on first use) and caches it; every injection point gets the same instance.
- **prototype**: Spring creates a brand-new instance every time the bean is requested/injected.
- **request** (web apps only): one instance per HTTP request.
- **session** (web apps only): one instance per HTTP session.

Under the hood, for singleton beans, Spring just returns the cached object from its internal bean registry. For prototype, it re-runs the whole creation process every time `getBean()` is called.

### Level 4 — Code
```java
@Component
@Scope("prototype")
public class ShoppingCart {
    // a new cart per request/injection
}

@Component
@Scope("singleton") // this is the default, so it's often omitted
public class TaxCalculator {
    // one shared instance
}
```

### Level 5 — Problem Solving
- **Symptom:** "My singleton bean seems to hold data from a different user/request" (a classic bug).
  **Cause:** Storing mutable per-user state in a singleton. **Fix:** Make the bean stateless, or change scope to `request`/`session`/`prototype`.
- **Symptom:** Injecting a `prototype` bean into a `singleton` bean gives you the SAME instance every time, not a new one.
  **Cause:** The singleton's dependency is only injected once, at singleton creation time. **Fix:** Use `ObjectProvider<T>` or a proxy (`proxyMode`) to fetch a fresh prototype on each use.

### Level 6 — Interview Answer
"Scope controls how many instances of a bean exist. Singleton — one instance shared app-wide — is the default and fine for stateless services. Prototype gives a new instance per request for the bean, useful for stateful, non-shared objects. Web-aware scopes like request and session tie a bean's lifetime to an HTTP request or session."

---

## 6. @Component / @Service / @Repository / @Controller

### Level 1 — Definition
These are all **stereotype annotations** that mark a class as a Spring-managed bean, so it gets picked up by component scanning. `@Service`, `@Repository`, and `@Controller` are all specialized versions of `@Component` — functionally similar, but semantically labeled for their role.

### Level 2 — Purpose
Spring needs some way to know which of your classes should become beans. Rather than manually registering every class, you annotate it and let component scanning find it automatically. Using the more specific annotations (`@Service` for business logic, `@Repository` for data access, `@Controller` for web endpoints) makes the codebase self-documenting and unlocks role-specific behavior.

### Level 3 — Internal Working
During component scanning, Spring looks for any class annotated with `@Component` **or any annotation that is itself annotated with `@Component`** (this is how `@Service`, `@Repository`, `@Controller` "inherit" component behavior — they're meta-annotated with `@Component`). It then registers a bean definition for each one found.

`@Repository` additionally enables automatic translation of database-specific exceptions into Spring's unified `DataAccessException` hierarchy, via a `BeanPostProcessor`.

`@Controller` (paired with `@RequestMapping` methods) is specially handled by Spring MVC's dispatcher to map HTTP requests to methods.

### Level 4 — Code
```java
@Repository
public class UserRepository {
    // data access logic
}

@Service
public class UserService {
    private final UserRepository repo;

    @Autowired
    public UserService(UserRepository repo) {
        this.repo = repo;
    }
}

@Controller
public class UserController {
    private final UserService service;

    @Autowired
    public UserController(UserService service) {
        this.service = service;
    }
}
```

### Level 5 — Problem Solving
- **Symptom:** Bean not found even though it has `@Component`.
  **Cause:** It's outside the package(s) scanned by `@ComponentScan` / `@SpringBootApplication`'s base package.
- **Symptom:** Raw SQL exceptions leaking out of the repository layer instead of clean Spring exceptions.
  **Cause:** Forgot `@Repository` annotation, so exception translation isn't applied.

### Level 6 — Interview Answer
"They're all stereotype annotations that register a class as a Spring bean. @Component is the generic form; @Service, @Repository, and @Controller are specializations that add semantic meaning and, in some cases, extra behavior — like @Repository's automatic exception translation. Using the specific one over @Component makes intent clear and gets you that role's extra features."

---

## 7. @Autowired vs Constructor Injection

### Level 1 — Definition
`@Autowired` is an annotation that tells Spring to inject a dependency automatically. It can be applied to a **constructor**, a **field**, or a **setter**. "Constructor injection" specifically means using `@Autowired` (or nothing at all, in modern Spring) on the constructor, so dependencies arrive as constructor parameters.

### Level 2 — Purpose
There are three ways to inject: field, setter, and constructor. Field injection (`@Autowired` directly on a field) is the shortest to write but has real downsides: it hides dependencies, makes the class hard to instantiate without Spring, and allows the object to exist in a half-wired state. Constructor injection is the recommended approach because it forces all required dependencies to be provided upfront, makes dependencies explicit and testable, and allows fields to be `final` (immutable).

### Level 3 — Internal Working
With constructor injection, Spring resolves and passes all constructor arguments **before** the object even exists — so the object is fully valid the instant it's constructed. With field injection, Spring creates the object first (using a no-arg constructor) and then reflectively sets the fields afterward — meaning there's a brief window where the object exists but isn't fully wired, and it also means fields can't be `final`.

Since Spring 4.3, if a class has only **one** constructor, `@Autowired` is optional — Spring infers it automatically.

### Level 4 — Code
```java
// Field injection (works, but discouraged)
@Service
public class OrderService {
    @Autowired
    private PaymentGateway gateway;
}

// Constructor injection (recommended)
@Service
public class OrderService {
    private final PaymentGateway gateway;

    // @Autowired is optional here since there's only one constructor
    public OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

### Level 5 — Problem Solving
- **Symptom:** Unit test needs `new OrderService()` but can't set the field-injected dependency.
  **Fix:** Switch to constructor injection — now you can just pass a mock directly: `new OrderService(mockGateway)`.
- **Symptom:** `NullPointerException` on a field-injected dependency used inside the constructor.
  **Cause:** Field injection happens **after** construction — the field is still null during the constructor body.
- **Symptom:** Circular dependency exception with constructor injection.
  **Fix:** This is actually a useful signal that your design has a cycle — refactor rather than switching to field injection just to "hide" the cycle.

### Level 6 — Interview Answer
"@Autowired can go on a field, setter, or constructor, but constructor injection is preferred: it makes dependencies explicit, allows immutable final fields, guarantees the object is fully initialized when created, and is far easier to unit test since you can just pass mocks into the constructor directly, without needing Spring at all."

---

# DAY 2 — Spring Configuration

## 1. Java Configuration

### Level 1 — Definition
Java configuration means defining Spring beans and application wiring using plain Java classes and annotations, instead of XML files.

### Level 2 — Purpose
Older Spring apps configured beans in verbose, error-prone XML. Java configuration lets you write configuration as regular, type-safe, refactorable Java code — you get compiler checking, IDE autocomplete, and the ability to use real logic (loops, conditionals) when defining beans.

### Level 3 — Internal Working
A class marked `@Configuration` is itself treated as a source of bean definitions. Spring processes it specially (using CGLIB proxying) so that calling one `@Bean` method from another within the same class returns the **same cached singleton instance** rather than creating a new one each time — preserving singleton semantics even in plain Java method calls.

### Level 4 — Code
```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentGateway paymentGateway() {
        return new StripeGateway();
    }

    @Bean
    public OrderService orderService() {
        // calling paymentGateway() here returns the SAME singleton instance
        return new OrderService(paymentGateway());
    }
}
```

### Level 5 — Problem Solving
- **Symptom:** Two different instances created when you expected one shared singleton.
  **Cause:** You called a plain method (not annotated `@Bean`) that constructs the object, instead of calling the `@Bean`-annotated method — bypass the proxy magic.
- **Symptom:** Class-level features like `final` classes break `@Configuration` proxying.
  **Fix:** `@Configuration` classes must not be `final` since Spring subclasses them via CGLIB.

### Level 6 — Interview Answer
"Java configuration replaces XML with type-safe Java classes annotated @Configuration, containing @Bean methods. Spring proxies these classes so that inter-bean method calls still respect singleton scope, giving you the flexibility of real code with the safety guarantees of the old XML approach."

---

## 2. @Configuration and @Bean

### Level 1 — Definition
`@Configuration` marks a class as a source of bean definitions. `@Bean` marks a method inside that class whose return value should be registered as a Spring-managed bean.

### Level 2 — Purpose
Sometimes you need to create a bean for a class you don't own (e.g., a third-party library class) — you can't add `@Component` to code you can't edit. `@Bean` solves this: you write a factory method that constructs and configures the object, and Spring manages it as a bean from then on.

### Level 3 — Internal Working
When the `ApplicationContext` processes a `@Configuration` class, it registers a bean definition for each `@Bean` method — using the method name as the default bean name (unless overridden) and the return type as the bean's type. The method is only actually invoked when Spring needs to instantiate that bean (respecting scope — singleton methods run once).

### Level 4 — Code
```java
@Configuration
public class ThirdPartyConfig {

    @Bean
    public RestTemplate restTemplate() {
        RestTemplate template = new RestTemplate();
        template.setErrorHandler(new CustomErrorHandler());
        return template;
    }
}
```
Anywhere else in the app:
```java
@Autowired
private RestTemplate restTemplate; // Spring hands you the configured bean
```

### Level 5 — Problem Solving
- **Symptom:** Bean name collision — two `@Bean` methods with the same name in different classes.
  **Fix:** Give the bean an explicit name: `@Bean("customRestTemplate")`.
- **Symptom:** `@Bean` method with complex constructor arguments — you want Spring to inject them.
  **Fix:** Just declare them as method parameters; Spring auto-injects matching beans:
  ```java
  @Bean
  public OrderService orderService(PaymentGateway gateway) {
      return new OrderService(gateway);
  }
  ```

### Level 6 — Interview Answer
"@Configuration marks a class as holding bean definitions, and @Bean marks a factory method whose return value Spring registers and manages as a bean. It's essential for wiring up third-party or manually-constructed objects that can't simply be annotated with @Component."

---

## 3. Component Scanning

### Level 1 — Definition
Component scanning is the process by which Spring automatically discovers classes annotated with stereotype annotations (`@Component`, `@Service`, etc.) in specified packages and registers them as beans, without you manually listing every class.

### Level 2 — Purpose
Manually registering every single bean would be tedious and error-prone as an app grows. Component scanning automates discovery — you just annotate the class, and as long as it's within a scanned package, Spring finds it.

### Level 3 — Internal Working
Spring walks the classpath under the configured base package(s), inspects each class's annotations, and if it matches an included filter (by default, any class carrying `@Component` or a meta-annotation of it), it creates a bean definition for it. In Spring Boot, `@SpringBootApplication` implicitly triggers `@ComponentScan` rooted at the package containing your main application class — so anything in that package or its sub-packages is scanned automatically.

### Level 4 — Code
```java
@SpringBootApplication // implies @ComponentScan on this package + subpackages
public class MyApp { }
```
Custom scan base packages:
```java
@Configuration
@ComponentScan(basePackages = {"com.example.services", "com.example.repositories"})
public class AppConfig { }
```

### Level 5 — Problem Solving
- **Symptom:** A `@Service` class isn't picked up as a bean.
  **Cause:** It lives in a package outside the scanned root — e.g., your main class is in `com.example.app` but the service is in `com.other.stuff`.
  **Fix:** Move the class into a sub-package of the scanned root, or explicitly widen `@ComponentScan`.

### Level 6 — Interview Answer
"Component scanning is Spring automatically finding and registering classes annotated with stereotype annotations, so I don't have to manually declare every bean. Spring Boot does this implicitly, rooted at your main application class's package — which is why package structure matters in Boot apps."

---

## 4. @Primary / @Qualifier

### Level 1 — Definition
`@Primary` and `@Qualifier` both resolve ambiguity when **multiple beans of the same type** exist and Spring doesn't know which one to inject. `@Primary` marks one bean as the "default choice." `@Qualifier` lets you specify, at the injection point, exactly which named bean you want.

### Level 2 — Purpose
If you have two implementations of `PaymentGateway` (say, `StripeGateway` and `PaypalGateway`), Spring can't guess which one a class wants when it asks for `PaymentGateway` by type alone. Without disambiguation, Spring throws an error. These annotations give you control over that choice.

### Level 3 — Internal Working
During dependency resolution, if Spring finds more than one bean matching the required type, it checks: is one of them marked `@Primary`? If yes, that one wins by default. If the injection point additionally specifies `@Qualifier("beanName")`, that takes precedence and Spring picks the bean matching that specific name/qualifier value, overriding `@Primary` if needed.

### Level 4 — Code
```java
@Component
@Primary
public class StripeGateway implements PaymentGateway { }

@Component
public class PaypalGateway implements PaymentGateway { }

@Service
public class OrderService {
    private final PaymentGateway gateway;

    // Without @Qualifier, StripeGateway wins because it's @Primary
    public OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}

@Service
public class RefundService {
    // Explicitly overrides to use PaypalGateway
    public RefundService(@Qualifier("paypalGateway") PaymentGateway gateway) { }
}
```

### Level 5 — Problem Solving
- **Symptom:** `NoUniqueBeanDefinitionException`.
  **Fix:** Add `@Primary` to your preferred default implementation, or use `@Qualifier` at every injection point.
- **Symptom:** `@Qualifier` value doesn't match anything.
  **Cause:** Default bean name is the class name with a lowercase first letter (`paypalGateway`, not `PaypalGateway`) unless explicitly renamed.

### Level 6 — Interview Answer
"When multiple beans share a type, Spring can't auto-resolve which to inject. @Primary designates a default winner among them. @Qualifier lets a specific injection point demand a particular named bean, overriding the default when needed. Together they resolve ambiguity cleanly."

---

## 5. Profiles

### Level 1 — Definition
Spring Profiles let you define **different sets of beans or configuration** for different environments (e.g., `dev`, `test`, `prod`) and activate only the relevant set at runtime.

### Level 2 — Purpose
A dev environment might use an in-memory database and mock services, while production uses a real database and real payment gateway. Without profiles, you'd need messy conditional logic scattered everywhere. Profiles let you cleanly tag beans by environment and switch between entire configurations with a single setting.

### Level 3 — Internal Working
Each bean (or `@Configuration` class) can be tagged with `@Profile("dev")`. At startup, Spring checks which profile(s) are **active** (set via `spring.profiles.active` property, an environment variable, or programmatically). Only beans whose profile matches an active profile — or beans with no profile annotation at all — get registered. Beans tagged for an inactive profile are simply skipped entirely; they never even get instantiated.

### Level 4 — Code
```java
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
        return new HikariDataSource(/* real prod DB config */);
    }
}
```
Activating a profile (`application.properties`):
```
spring.profiles.active=dev
```

### Level 5 — Problem Solving
- **Symptom:** Wrong `DataSource` is used, or none at all.
  **Cause:** Active profile doesn't match any `@Profile`-tagged bean — check `spring.profiles.active` is actually set (command-line args, environment variables, and properties files can conflict).
- **Symptom:** Multiple profiles seem to conflict.
  **Fix:** You can activate multiple profiles at once (`spring.profiles.active=dev,debug`) — beans from all active profiles are included.

### Level 6 — Interview Answer
"Profiles let me define environment-specific beans and configuration — like different data sources for dev versus production — and activate only one set at runtime via a property. Beans tagged for inactive profiles are never even created, keeping environments cleanly isolated."

---

## 6. application.properties / YAML

### Level 1 — Definition
`application.properties` and `application.yml` are the standard configuration files Spring Boot automatically loads at startup to externalize settings — like server port, database URL, and logging level — outside of your Java code.

### Level 2 — Purpose
Hardcoding values like database URLs or ports directly into Java code means you'd need to recompile the app to change them. Externalizing configuration into a properties/YAML file means the same compiled app can run differently in dev, test, and prod just by changing a config file — no code changes needed.

### Level 3 — Internal Working
On startup, Spring Boot loads `application.properties`/`application.yml` from a set of default locations (classpath root, `/config` subfolder, etc.) into its `Environment` abstraction — essentially a big unified key-value store. Later configuration sources (like environment variables or command-line args) can override earlier ones, following Spring Boot's defined precedence order. `@Value` and `@ConfigurationProperties` (covered next) then read values out of this `Environment`.

### Level 4 — Code
`application.properties`:
```
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
logging.level.org.springframework=DEBUG
```
Same thing in `application.yml` (YAML is hierarchical, less repetitive):
```yaml
server:
  port: 8081
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
logging:
  level:
    org.springframework: DEBUG
```

### Level 5 — Problem Solving
- **Symptom:** Property value doesn't seem to apply.
  **Cause:** Wrong file location, typo in the key, or an environment variable is overriding it (env vars generally win over file properties).
- **Symptom:** YAML file "not parsing right."
  **Cause:** YAML is whitespace-sensitive — mixing tabs and spaces, or inconsistent indentation, breaks it silently.

### Level 6 — Interview Answer
"These files externalize configuration so the same compiled application can behave differently across environments without code changes. Spring Boot loads them into a unified Environment abstraction at startup, and values from higher-precedence sources — like environment variables — can override what's in the file."

---

## 7. Environment and External Configuration

### Level 1 — Definition
`Environment` is Spring's abstraction representing all configuration sources available to the application — properties files, environment variables, command-line arguments, and more — unified into one queryable interface.

### Level 2 — Purpose
Configuration can come from many places (files, OS environment variables, JVM system properties, command-line args). Rather than making you check each source manually, `Environment` merges them all into a single, consistent way to ask "what is the value of this property?" — with a well-defined precedence order for when sources conflict.

### Level 3 — Internal Working
`Environment` holds an ordered list of `PropertySource` objects. When you ask for a property, Spring checks each source **in priority order** and returns the first match. This is what allows, for example, a command-line argument (`--server.port=9090`) to override a value set in `application.properties`, without any manual coding.

### Level 4 — Code
```java
@Component
public class ConfigChecker {

    @Autowired
    private Environment env;

    public void printPort() {
        String port = env.getProperty("server.port");
        System.out.println("Server running on port: " + port);
    }
}
```

### Level 5 — Problem Solving
- **Symptom:** A property value is different than what's in `application.properties`.
  **Cause:** Something higher in precedence (env variable, `-D` system property, command-line arg) is overriding it — this is expected behavior, not a bug, but worth knowing the order.
- **Symptom:** `env.getProperty()` returns `null`.
  **Fix:** Provide a default: `env.getProperty("some.key", "defaultValue")`, and double check the key spelling.

### Level 6 — Interview Answer
"Environment is Spring's unified abstraction over all configuration sources — files, env vars, system properties, command-line args — merged with a clear precedence order. It's what lets external configuration flexibly override internal defaults without any code changes."

---

## 8. @Value and @ConfigurationProperties

### Level 1 — Definition
`@Value` injects a **single** configuration property value directly into a field. `@ConfigurationProperties` binds a **whole group** of related properties onto a Java object's fields at once, based on a common prefix.

### Level 2 — Purpose
`@Value` is quick and simple for grabbing one or two individual values. But when you have many related settings (say, ten different `myapp.mail.*` properties), repeating `@Value` ten times is tedious and error-prone. `@ConfigurationProperties` solves that by binding an entire structured block of configuration to a strongly-typed Java class in one shot — with type conversion, validation, and IDE autocomplete support.

### Level 3 — Internal Working
`@Value("${key}")` triggers Spring to resolve that specific placeholder from the `Environment` at bean creation time and inject the resulting value into the field. `@ConfigurationProperties(prefix = "myapp.mail")` instead tells Spring to take every property starting with that prefix and match each remaining part of the key to a matching setter/field on the class (using relaxed binding rules — `myapp.mail.host` maps to a `host` field, etc.), converting types as needed (String to int, to List, to nested objects).

### Level 4 — Code
`@Value` — single property:
```java
@Component
public class MailSender {
    @Value("${myapp.mail.host}")
    private String host;
}
```
`@ConfigurationProperties` — grouped properties:
```java
@Component
@ConfigurationProperties(prefix = "myapp.mail")
public class MailProperties {
    private String host;
    private int port;
    private boolean sslEnabled;
    // getters and setters required
}
```
```properties
myapp.mail.host=smtp.example.com
myapp.mail.port=587
myapp.mail.ssl-enabled=true
```

### Level 5 — Problem Solving
- **Symptom:** `@Value` field stays null/empty.
  **Cause:** Typo in the property key, or missing `${...}` syntax, or property doesn't exist in any loaded source (no default provided).
  **Fix:** Provide a default: `@Value("${myapp.timeout:5000}")`.
- **Symptom:** `@ConfigurationProperties` fields aren't populated.
  **Cause:** Missing getters/setters, or the class isn't registered as a bean (`@Component`, or `@EnableConfigurationProperties` if defined separately) — also double-check the prefix matches exactly.
- **Symptom:** Want validation on the properties (e.g., port must be positive).
  **Fix:** Add `@Validated` plus JSR-303 annotations like `@Min` on the fields.

### Level 6 — Interview Answer
"@Value pulls a single property value into a field using a placeholder expression — good for one-offs. @ConfigurationProperties binds a whole prefixed group of related properties onto a strongly-typed class at once, which is cleaner and safer for larger configuration blocks, and supports validation and nested structures that @Value can't easily handle."

---

*End of Day 1 & Day 2 Core guide.*
