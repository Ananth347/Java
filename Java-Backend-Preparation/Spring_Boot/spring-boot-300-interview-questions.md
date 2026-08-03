# Spring Boot Interview Questions & Answers (300+)

Plain-English answers — written the way you'd actually explain it out loud in an interview, not textbook definitions. Organized by topic so you can prep section by section.

---

## Section 1: Spring Boot Basics & Introduction

**1. What is Spring Boot?**
It's a framework built on top of the Spring Framework that removes most of the manual setup and configuration Spring traditionally needed. It gives you sensible defaults and auto-configuration so you can get a working application running quickly, without wiring everything by hand.

**2. Why was Spring Boot introduced when Spring already existed?**
Spring was powerful but required a lot of XML or Java-based configuration for even simple things — data sources, view resolvers, servlet setup. Spring Boot was built to cut that boilerplate down drastically using auto-configuration and starter dependencies.

**3. What are the main features of Spring Boot?**
Auto-configuration, embedded servers (no need to install Tomcat separately), starter dependencies (pre-bundled sets of libraries), production-ready features via Actuator, and minimal XML configuration.

**4. Is Spring Boot a replacement for Spring?**
No — Spring Boot is built on top of Spring, not a replacement. It uses all of Spring's core features (IoC, DI, AOP) internally; it just removes the setup pain around using them.

**5. What is the difference between Spring and Spring Boot?**
Spring requires you to manually configure almost everything — beans, data sources, servlets. Spring Boot auto-configures most of that for you based on what's on your classpath, and bundles an embedded server so you don't need external deployment infrastructure to get started.

**6. What is a Spring Boot Starter?**
A starter is a curated set of dependencies bundled together for a specific purpose — for example, `spring-boot-starter-web` brings in everything needed to build a web app (Spring MVC, embedded Tomcat, Jackson for JSON), so you don't have to figure out and add each dependency individually.

**7. Name a few commonly used Spring Boot starters.**
`spring-boot-starter-web` for REST/web apps, `spring-boot-starter-data-jpa` for database access, `spring-boot-starter-security` for authentication/authorization, `spring-boot-starter-test` for testing, and `spring-boot-starter-actuator` for monitoring.

**8. What is Spring Initializr?**
It's a web tool (start.spring.io) that lets you generate a ready-to-go Spring Boot project skeleton — you pick your build tool, Java version, and dependencies, and it produces a downloadable project structure with everything wired up.

**9. Can Spring Boot applications be deployed as WAR files?**
Yes, although it's less common now — you can package it as a WAR and deploy to an external servlet container if needed, by extending `SpringBootServletInitializer`. Most modern deployments prefer the default executable JAR with an embedded server instead.

**10. What is the entry point of a Spring Boot application?**
A class with a `main()` method that calls `SpringApplication.run(YourClass.class, args)`. This bootstraps the whole Spring context, registers all your beans, and gets the application running.

**11. What does `SpringApplication.run()` actually do?**
It creates and configures the Spring `ApplicationContext`, registers all your beans, triggers auto-configuration, starts the embedded server if it's a web application, and fires off various lifecycle events along the way.

**12. What build tools does Spring Boot commonly use?**
Maven and Gradle are the two standard choices — both have Spring Boot plugins that let you build an executable "fat" JAR containing your code plus all dependencies.

**13. What is a "fat JAR" / "executable JAR"?**
It's a single JAR file that contains not just your compiled code, but also all its dependencies and even the embedded server, bundled together — so you can run the whole application with a simple `java -jar app.jar` command, no separate installation needed.

**14. What embedded servers does Spring Boot support?**
Tomcat by default, but you can switch to Jetty or Undertow by excluding the default starter and adding the alternative one instead.

**15. What is the advantage of an embedded server?**
You don't need to install and configure a separate application server — the server ships inside your application itself, making deployment simpler and much easier to containerize with Docker, since the whole app is self-contained.

---

## Section 2: Spring Boot Annotations

**16. What does `@SpringBootApplication` do?**
It's a convenience annotation that bundles three others: `@Configuration` (marks the class as a source of bean definitions), `@EnableAutoConfiguration` (turns on Spring Boot's automatic setup), and `@ComponentScan` (scans the package and sub-packages for components to register as beans).

**17. What is `@Configuration` used for?**
It marks a class as a source of bean definitions — methods inside it, annotated with `@Bean`, are used by Spring to create and manage objects in the application context.

**18. What is `@ComponentScan`?**
It tells Spring which packages to scan for classes annotated with `@Component`, `@Service`, `@Repository`, or `@Controller`, so it can automatically register them as beans without you declaring each one manually.

**19. What is `@EnableAutoConfiguration`?**
It tells Spring Boot to try and automatically configure your application based on the dependencies present on the classpath — for example, auto-configuring a `DataSource` if it sees a database driver.

**20. Difference between `@Component`, `@Service`, and `@Repository`?**
All three register a class as a Spring bean — functionally almost identical. `@Component` is the general-purpose stereotype, `@Service` signals business logic, and `@Repository` signals data access and additionally enables automatic translation of database-specific exceptions into Spring's consistent exception hierarchy.

**21. What is `@Autowired` used for?**
It tells Spring to automatically inject a required dependency into a field, constructor, or setter, rather than you creating that object manually with `new`.

**22. What is `@Qualifier` used for?**
When multiple beans of the same type exist, Spring can't decide which one to inject and throws an error. `@Qualifier("beanName")` tells Spring exactly which specific bean you want at that injection point.

**23. What is `@Primary`?**
When multiple beans of the same type exist, marking one with `@Primary` makes it the default choice for injection whenever no `@Qualifier` is specified.

**24. What is `@Value` used for?**
It injects a value from your application's configuration (like `application.properties`) directly into a field — for example, `@Value("${server.port}")` injects the configured server port into a variable.

**25. What is `@Controller`?**
It marks a class as a Spring MVC controller, typically used in traditional web apps where methods return view names to be rendered (like Thymeleaf templates), rather than data directly.

**26. What is `@RestController`?**
It's `@Controller` combined with `@ResponseBody` on every method — return values are serialized directly into the HTTP response body (usually JSON), exactly what's needed for building REST APIs.

**27. What is `@RequestMapping`?**
It maps HTTP requests to handler methods or classes, letting you specify the URL path, HTTP method, headers, and more. It's the general-purpose mapping annotation that the more specific ones are shortcuts for.

**28. Difference between `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`?**
They're shortcuts for `@RequestMapping(method = ...)`, one per HTTP verb — `@GetMapping` for fetching, `@PostMapping` for creating, `@PutMapping` for fully replacing, `@DeleteMapping` for removing, and `@PatchMapping` for partially updating a resource.

**29. What is `@PathVariable`?**
It extracts a value directly from the URL path — like the `5` in `/orders/5` — and binds it to a method parameter.

**30. What is `@RequestParam`?**
It extracts a value from the query string of a URL — like `status` in `/orders?status=PENDING` — and binds it to a method parameter, optionally with a default value.

**31. What is `@RequestBody`?**
It tells Spring to take the incoming HTTP request body (usually JSON) and automatically deserialize it into a Java object, so you can work with it as a typed parameter in your controller method.

**32. What is `@ResponseBody`?**
It tells Spring to serialize the return value of a method directly into the HTTP response body, instead of treating it as a view name to render.

**33. What is `@ResponseStatus`?**
It lets you specify a custom HTTP status code returned on success, or when a particular exception is thrown, instead of always defaulting to 200 OK.

**34. What is `@RequestHeader`?**
It extracts the value of a specific HTTP header from the incoming request and binds it to a method parameter — commonly used for reading an `Authorization` header.

**35. What is `@CrossOrigin`?**
It enables Cross-Origin Resource Sharing (CORS) for a controller or method, allowing browsers on a different domain to call your API — otherwise browsers block such requests by default.

**36. What is `@ControllerAdvice` / `@RestControllerAdvice`?**
They let you define centralized, global exception handling and cross-cutting logic across all your controllers, instead of repeating try-catch blocks everywhere. `@RestControllerAdvice` additionally serializes responses as JSON.

**37. What is `@ExceptionHandler`?**
Placed on a method (usually inside a `@ControllerAdvice` class) to designate it as the handler for a specific exception type — whenever that exception is thrown anywhere in a controller, Spring routes it to this method.

**38. What is `@Valid` used for?**
It triggers Bean Validation on an incoming object (like a `@RequestBody` DTO) based on annotations like `@NotNull` or `@Size` on its fields — if validation fails, Spring throws an exception you can handle globally.

**39. What is `@Bean`?**
Placed on a method inside a `@Configuration` class, telling Spring that the object returned by that method should be registered as a bean in the application context.

**40. What is `@Scope`?**
It defines the lifecycle scope of a bean — like `singleton` (one shared instance, the default) or `prototype` (a new instance every time it's requested).

**41. What is `@PostConstruct`?**
It marks a method to be run automatically right after a bean's dependencies have been injected — useful for any initialization logic that needs to run once the bean is fully constructed.

**42. What is `@PreDestroy`?**
It marks a method to be run automatically just before a bean is destroyed (like on application shutdown) — useful for cleanup work like closing connections.

**43. What is `@Lazy`?**
It tells Spring to delay creating a bean until it's actually needed for the first time, instead of creating it eagerly at startup — useful for beans that are expensive to create but rarely used.

**44. What is `@ConditionalOnProperty`?**
It's used in auto-configuration to only register a bean if a specific property is set to a specific value in your configuration — letting you toggle certain features on or off through config alone.

**45. What is `@Profile`?**
It marks a bean or configuration class to only be active when a specific Spring profile (like "dev" or "prod") is active, letting you have environment-specific beans.

**46. What is `@ConfigurationProperties`?**
It binds a whole group of related external properties (from `application.properties`/`yml`) directly into a Java object's fields, instead of injecting each one individually with `@Value` — cleaner for related configuration groups.

**47. What is `@EnableScheduling`?**
It turns on Spring's support for scheduled task execution, which is required before any `@Scheduled` annotated method will actually run.

**48. What is `@Scheduled`?**
It marks a method to run automatically on a defined schedule — either a fixed rate, fixed delay, or a cron expression — commonly used for background jobs like nightly report generation.

**49. What is `@Async`?**
It marks a method to run in a separate thread, asynchronously, rather than blocking the caller — useful for tasks like sending an email that don't need to hold up the main request.

**50. What is `@EnableAsync`?**
It's required at the application/configuration level to actually activate Spring's support for `@Async` methods.

**51. What is `@Transactional`?**
It wraps a method's execution in a database transaction, so all database operations inside it succeed together or roll back together if something fails.

**52. What is `@EnableTransactionManagement`?**
It activates Spring's annotation-driven transaction management, enabling `@Transactional` to actually work — in Spring Boot, this is typically auto-configured for you already.

**53. What is `@Entity`?**
It marks a Java class as a JPA entity — meaning it maps directly to a database table, with each instance representing one row.

**54. What is `@Table`?**
It's used alongside `@Entity` to specify the actual database table name (and sometimes schema) the entity maps to, if it's different from the default (the class name).

**55. What is `@Id`?**
It marks a field in an entity as the primary key of the corresponding database table.

**56. What is `@GeneratedValue`?**
It's used along with `@Id` to specify how the primary key value should be automatically generated — for example, using the database's auto-increment feature.

**57. What is `@Column`?**
It customizes how a field maps to its database column — like specifying a different column name, whether it's nullable, or its length.

**58. What is `@OneToMany`, `@ManyToOne`, `@OneToOne`, and `@ManyToMany`?**
These describe the type of relationship between two entities — one order having many order items (`@OneToMany`), many order items belonging to one order (`@ManyToOne`), a user having exactly one profile (`@OneToOne`), and students enrolled in many courses while courses have many students (`@ManyToMany`).

**59. What is `@JoinColumn`?**
It specifies the actual foreign key column used to join two entities in a relationship.

**60. What is `@Repository`?**
It marks an interface/class as a data-access component, and enables Spring's automatic translation of low-level database exceptions into its own consistent exception hierarchy.

---

## Section 3: Auto-Configuration & Starters

**61. How does Spring Boot auto-configuration actually work under the hood?**
Spring Boot scans a special file listing possible auto-configuration classes, then each one uses conditional annotations (like `@ConditionalOnClass`) to check if it should actually apply — for example, only auto-configuring a `DataSource` if a JDBC driver is found on the classpath.

**62. What is `@ConditionalOnClass`?**
It tells Spring to only register a bean/configuration if a specific class is present on the classpath — used heavily inside auto-configuration to detect which libraries you've included.

**63. What is `@ConditionalOnMissingBean`?**
It tells Spring to only register a bean if one of that type hasn't already been defined — this is how Spring Boot lets you override its own auto-configured beans simply by defining your own.

**64. Can you override an auto-configured bean?**
Yes — if you define your own bean of the same type, Spring Boot's auto-configuration backs off (thanks to `@ConditionalOnMissingBean`), and your custom bean is used instead.

**65. How would you exclude a specific auto-configuration class?**
Using `@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })`, or by setting the `spring.autoconfigure.exclude` property.

**66. What is the `spring-boot-starter-parent`?**
It's a parent POM that provides default configuration for Maven projects — sensible dependency versions, plugin configuration, and Java version settings — so you don't have to manage them yourself.

**67. Can you create your own custom starter?**
Yes — you bundle a set of related dependencies together along with an auto-configuration class in a separate module, so other projects can just add your starter dependency and get everything wired up automatically.

**68. What is `spring.factories` (or the newer `AutoConfiguration.imports`)?**
It's a file that lists all the classes Spring Boot should consider for auto-configuration when the application starts — this is the mechanism auto-configuration is actually built on.

**69. How do you check exactly which auto-configurations were applied in your app?**
By enabling debug mode (`--debug` flag or `debug=true` property), which prints an auto-configuration report at startup, showing which configurations were applied and which were skipped, with reasons.

**70. What happens if two starters try to auto-configure the same bean?**
Spring Boot's conditional annotations (like `@ConditionalOnMissingBean`) are designed to avoid actual conflicts by backing off if a bean already exists — but if there's a genuine ambiguity, you'll typically need to explicitly define which bean should be used, often with `@Primary` or `@Qualifier`.

---

## Section 4: Configuration Properties & Profiles

**71. What is `application.properties` used for?**
It's the central file for externalizing configuration — database URLs, server port, logging levels, and custom app settings — so you're not hardcoding values into your Java code.

**72. Difference between `application.properties` and `application.yml`?**
They serve the exact same purpose, just different formats — `.properties` is flat key-value pairs, `.yml` uses indentation-based nesting, which can be more readable for deeply nested configuration, but is also more sensitive to indentation mistakes.

**73. What are Spring Profiles?**
They let you maintain different sets of configuration for different environments (dev, test, prod), and activate the right one at runtime without changing code — using files like `application-dev.properties`.

**74. How do you activate a specific profile?**
By setting `spring.profiles.active=dev` in your properties file, as a command-line argument, or as an environment variable — whichever is most convenient for your deployment pipeline.

**75. Can multiple profiles be active at the same time?**
Yes — you can specify a comma-separated list, like `spring.profiles.active=dev,debug`, and Spring Boot merges the configuration from all active profiles.

**76. What is the order of precedence for configuration properties in Spring Boot?**
Roughly: command-line arguments override environment variables, which override profile-specific properties files, which override the default `application.properties`, which override defaults hardcoded in your code — command-line args generally win over everything else.

**77. What is `@ConfigurationProperties` and how is it different from `@Value`?**
`@Value` injects a single property into a single field. `@ConfigurationProperties` binds a whole group of related properties (like everything under `app.mail.*`) into a structured object's fields all at once — much cleaner when you have several related settings.

**78. How do you validate configuration properties?**
By adding `@Validated` on a `@ConfigurationProperties` class along with Bean Validation annotations (`@NotNull`, `@Min`, etc.) on its fields — the application will fail to start if the bound values don't meet those constraints.

**79. How do you provide default values for a property?**
With `@Value("${some.property:defaultValue}")`, or by simply setting a default value on the field itself when using `@ConfigurationProperties`.

**80. How would you externalize sensitive configuration like database passwords?**
Avoid hardcoding them in `application.properties` committed to source control — use environment variables, a secrets manager (like AWS Secrets Manager or HashiCorp Vault), or an encrypted config server (like Spring Cloud Config with encryption) instead.

**81. What is Spring Cloud Config?**
It's a centralized configuration server that lets multiple microservices pull their configuration from one shared source (often backed by a Git repo), instead of managing separate config files scattered across every service.

**82. What is a relaxed binding in Spring Boot configuration?**
Spring Boot is flexible about the exact format of property names — `my-property`, `myProperty`, and `MY_PROPERTY` (environment variable style) can all bind to the same configuration field, making it easy to configure via different mechanisms (files vs. env vars) consistently.

**83. How do you change the default server port?**
By setting `server.port=8081` in your properties file, or as an environment variable/command-line argument.

**84. How do you configure a custom banner or disable the Spring Boot startup banner?**
By placing a `banner.txt` file on the classpath for a custom one, or setting `spring.main.banner-mode=off` to disable it entirely.

**85. What is `@PropertySource`?**
It lets you load properties from a custom, additional file beyond the default `application.properties`, useful when you want to keep certain configuration groups in a separate file.

---

## Section 5: Bean Lifecycle & Dependency Injection

**86. What is a Spring Bean?**
It's simply an object whose creation, configuration, and lifecycle is managed by the Spring IoC container, instead of your own code creating and managing it directly.

**87. What is the default scope of a Spring Bean?**
Singleton — Spring creates exactly one shared instance of the bean per application context, and every place that injects it gets the same instance.

**88. What are the different bean scopes available?**
Singleton (one shared instance), Prototype (a new instance every time it's requested), and web-specific scopes like Request (one instance per HTTP request) and Session (one instance per user session).

**89. Explain the Spring Bean lifecycle briefly.**
Spring instantiates the bean, injects its dependencies, calls any `@PostConstruct` method, and the bean is then ready to use. When the application shuts down, Spring calls any `@PreDestroy` method before discarding the bean.

**90. What are the three types of Dependency Injection in Spring?**
Constructor injection (dependencies passed through the constructor — recommended, since it makes dependencies explicit and lets fields be `final`), setter injection (through setter methods, good for optional dependencies), and field injection (`@Autowired` directly on a field — concise but discouraged, since it hides dependencies and complicates testing).

**91. Why is constructor injection generally preferred over field injection?**
It makes all required dependencies explicit and visible right in the constructor signature, allows fields to be immutable (`final`), fails fast at startup if a dependency is missing, and makes unit testing much easier since you can just pass mocks directly without needing Spring or reflection.

**92. What happens if Spring can't find a bean to satisfy an `@Autowired` dependency?**
It throws a `NoSuchBeanDefinitionException` at startup, telling you it couldn't find a matching bean — this is one of the benefits of Spring's fail-fast approach, catching wiring mistakes immediately rather than at runtime later.

**93. What happens if there are two beans of the same type and Spring can't decide which to inject?**
It throws a `NoUniqueBeanDefinitionException` — you resolve it using `@Qualifier` to specify which one, or `@Primary` to mark one as the default.

**94. What is circular dependency in Spring, and how does Spring handle it?**
It's when Bean A depends on Bean B, and Bean B depends back on Bean A, creating a loop. Spring can sometimes resolve this automatically for singleton beans using setter/field injection (with some internal tricks), but it typically fails with constructor injection — the real fix is usually to redesign the beans to remove the circular relationship, or extract shared logic into a third bean.

**95. What is `ApplicationContext`?**
It's the central interface representing the Spring IoC container — it holds all your beans, manages their lifecycle, and provides features like event publishing and internationalization on top of the basic bean factory.

**96. Difference between `BeanFactory` and `ApplicationContext`?**
`BeanFactory` is the more basic container, providing core DI functionality with lazy bean initialization. `ApplicationContext` builds on top of it with more enterprise features (event handling, AOP integration, internationalization) and eagerly initializes singleton beans by default — it's what you use in virtually all real applications.

**97. What is a Bean Definition?**
It's the metadata Spring holds about how to create a specific bean — its class, scope, dependencies, initialization/destruction methods — before the actual object is instantiated.

**98. What is the difference between `@Component` and manually defining a `@Bean` in a `@Configuration` class?**
`@Component` (with component scanning) is for classes you own and can annotate directly. `@Bean` inside `@Configuration` is used when you need more control over the creation logic, or when registering a class you don't own (like a third-party library class you can't annotate).

**99. Can you inject a `List` or `Map` of all beans of a certain type?**
Yes — Spring supports injecting a `List<SomeInterface>` containing every bean implementing that interface, or a `Map<String, SomeInterface>` keyed by bean name, which is very useful for strategy-pattern style designs.

**100. What is a `FactoryBean`?**
It's a special kind of bean whose job is to produce another bean — instead of Spring returning the `FactoryBean` instance itself when you inject it, Spring calls its `getObject()` method and gives you the object that method returns.

---

## Section 6: REST APIs in Spring Boot

**101. Walk through what happens when a REST request hits a Spring Boot app.**
The embedded Tomcat receives the request first, then it flows through Spring's `DispatcherServlet`, which uses `HandlerMapping` to find the right controller method. Any filters/interceptors run along the way, the controller method executes (usually delegating to a service and repository layer), and the returned object gets serialized (typically to JSON) before being sent back.

**102. What is `DispatcherServlet`?**
It's the central front-controller in Spring MVC — every incoming HTTP request passes through it first, and it's responsible for routing the request to the correct controller method and handling the overall request/response lifecycle.

**103. How does Spring Boot convert Java objects to JSON automatically?**
It uses the Jackson library by default, which is auto-configured as part of `spring-boot-starter-web` — when a `@RestController` method returns an object, Jackson serializes it into JSON before sending the response.

**104. How do you rename a field in the JSON output without renaming the Java field?**
Using Jackson's `@JsonProperty("customName")` annotation on the field or getter.

**105. How do you exclude a field from JSON serialization?**
Using `@JsonIgnore` on the field, or `@JsonIgnoreProperties` on the class to exclude multiple fields at once — useful for hiding sensitive data like passwords.

**106. What is content negotiation in Spring REST APIs?**
It's how the server decides what format (JSON, XML, etc.) to return a response in, based on the client's `Accept` header — Spring Boot handles this automatically, defaulting to JSON if nothing else is specified or available.

**107. What is the difference between `@PathVariable` and `@RequestParam` again, in terms of REST design?**
Path variables typically identify a specific resource (`/users/5`), while request params are used for filtering, sorting, or pagination on a collection (`/users?sort=name&page=2`).

**108. How would you implement pagination in a Spring Boot REST API?**
Using Spring Data's `Pageable` parameter directly in a repository method or controller — Spring automatically parses query params like `page` and `size`, and returns a `Page<T>` object containing the data plus metadata like total pages and total elements.

**109. What status code should a successful POST that creates a resource return?**
201 Created, ideally along with a `Location` header pointing to the newly created resource's URL.

**110. How do you return a custom HTTP status along with a response body?**
By wrapping the response in a `ResponseEntity`, like `ResponseEntity.status(HttpStatus.CREATED).body(savedEntity)`.

**111. What is `ResponseEntity` used for?**
It represents the entire HTTP response — status code, headers, and body — giving you full control, unlike simply returning a plain object where Spring assumes a default 200 OK status.

**112. What is HATEOAS, and does Spring Boot support it?**
HATEOAS (Hypermedia as the Engine of Application State) means API responses include links to related actions/resources, so clients can navigate the API dynamically rather than hardcoding URLs. Spring Boot supports it through the `spring-hateoas` library.

**113. How do you version a REST API in Spring Boot?**
Common approaches: URL versioning (`/api/v1/users`), request header versioning, or query parameter versioning — URL versioning is the simplest and most widely used in practice.

**114. What is idempotency and which HTTP methods are expected to be idempotent?**
An idempotent operation gives the same result no matter how many times it's called. GET, PUT, and DELETE are expected to be idempotent; POST typically is not, since calling it repeatedly usually creates multiple new resources.

**115. How would you handle file uploads in a Spring Boot REST API?**
Using `MultipartFile` as a controller method parameter along with `multipart/form-data` content type — Spring Boot auto-configures the multipart resolver needed to handle this.

**116. What is `@ModelAttribute` used for?**
It binds form data or query parameters directly into a Java object automatically, commonly used in traditional MVC form submissions, less so in pure REST/JSON APIs.

**117. How does Spring Boot handle exceptions thrown in a controller by default?**
Without custom handling, Spring returns a generic error response (via its default `/error` mapping) with a stack trace or a basic error message and an appropriate status code — but production apps almost always override this with a `@RestControllerAdvice` for cleaner, consistent error responses.

**118. What is the purpose of DTOs in a REST API, and why not return Entities directly?**
DTOs (Data Transfer Objects) shape exactly what data goes over the wire, independent of your database structure — returning entities directly can leak internal fields, cause lazy-loading serialization errors, and tightly couples your API contract to your database schema.

**119. How would you handle API rate limiting in Spring Boot?**
Typically with a library like Bucket4j or Resilience4j, often backed by Redis for shared counters across multiple instances — you define limits per client/IP/API key and reject or delay requests exceeding them.

**120. What is Swagger/OpenAPI and how is it used with Spring Boot?**
It's a specification and toolset (via `springdoc-openapi` in modern Spring Boot) for automatically generating interactive API documentation directly from your controller annotations, letting other developers or QA explore and test your endpoints without needing separate manually written docs.

---

## Section 7: Spring Data JPA & Hibernate

**121. What is Spring Data JPA?**
It's a Spring project that sits on top of JPA/Hibernate and drastically reduces boilerplate — you just define a repository interface, and Spring generates the actual query implementations for common operations automatically.

**122. What is `JpaRepository`, and what does it give you out of the box?**
It's an interface you extend for your entity, and it automatically gives you methods like `save()`, `findById()`, `findAll()`, `deleteById()`, plus pagination and sorting support — all without writing any implementation code yourself.

**123. How does Spring Data generate queries just from method names?**
By parsing the method name itself — a method named `findByEmailAndStatus(String email, String status)` is automatically translated into a query filtering by both fields, based on Spring Data's naming conventions.

**124. What is `@Query` used for?**
It lets you write a custom JPQL or native SQL query directly on a repository method, for cases where the method-name-based query generation isn't flexible enough or gets too complex/unreadable.

**125. Difference between JPQL and native SQL in `@Query`?**
JPQL operates on your entity objects and their fields (not actual table/column names), and is database-independent. Native SQL is the actual raw SQL specific to your database, used when you need database-specific features JPQL doesn't support.

**126. What is the difference between `save()` and `saveAndFlush()`?**
`save()` may defer the actual database write until the transaction commits or a flush naturally happens. `saveAndFlush()` forces an immediate write to the database right away — useful when you need the generated ID immediately for further logic within the same transaction.

**127. What is Lazy Loading vs Eager Loading?**
Lazy loading defers fetching related data until you actually access it in code. Eager loading fetches related data immediately, together with the main entity. Lazy is generally preferred for performance, but can throw a `LazyInitializationException` if accessed after the session/transaction has closed.

**128. What is the N+1 query problem, and how do you fix it?**
It's when fetching N parent records triggers N additional separate queries for each one's related child data, instead of one combined query. It's fixed using `JOIN FETCH` in a JPQL query, or `@EntityGraph` to specify what should be fetched together upfront.

**129. What is `@EntityGraph`?**
It lets you declare, at the repository method level, exactly which related entities should be eagerly fetched together in a single query — a cleaner alternative to writing manual `JOIN FETCH` queries every time.

**130. What is the first-level cache in Hibernate?**
It's enabled by default and scoped to a single persistence context/session — if you request the same entity twice within one transaction, Hibernate returns the already-loaded object instead of querying again.

**131. What is the second-level cache in Hibernate?**
It's an optional cache shared across sessions (and potentially the whole application), requiring explicit setup with a provider like Ehcache or Redis — useful for data read frequently but rarely changed, like lookup/reference tables.

**132. What is the difference between `persist()`, `merge()`, and `save()`?**
`persist()` (pure JPA) inserts a new entity and returns void. `merge()` takes a detached entity (one not currently tracked by the session) and copies its state onto a managed entity, effectively updating it. `save()` is Spring Data JPA's own convenience method that handles both insert and update, returning the saved entity.

**133. What does "managed" vs "detached" entity mean in JPA?**
A managed entity is currently being tracked by the persistence context (session) — any changes to it are automatically detected and saved when the transaction commits. A detached entity is no longer tracked — changes to it won't automatically be persisted unless it's re-attached via `merge()`.

**134. What is dirty checking in Hibernate?**
Hibernate automatically detects if a managed entity's fields have changed since it was loaded, and if so, generates the necessary UPDATE SQL automatically at flush/commit time — you don't need to explicitly call an update method for managed entities.

**135. What is `@Transactional(readOnly = true)` used for with JPA?**
It signals to Hibernate that the method only reads data and won't modify anything, allowing it to skip some internal overhead (like dirty-checking), which is a small performance optimization for read-heavy methods.

**136. What are cascade types in JPA, and give an example.**
Cascade types define which operations on a parent entity should automatically apply to its related child entities — for example, `CascadeType.ALL` on an `Order`'s items means deleting the order also deletes all its items automatically.

**137. What is `orphanRemoval`?**
When set to true on a relationship, if a child entity is removed from its parent's collection (even without an explicit delete call), Hibernate automatically deletes that child from the database too — useful for compositions where a child truly has no meaning without its parent.

**138. What is the difference between `FetchType.LAZY` and `FetchType.EAGER`?**
`LAZY` means the related data is only loaded from the database when you actually access it. `EAGER` means it's loaded immediately along with the parent entity. `@ManyToOne` and `@OneToOne` default to eager, while `@OneToMany` and `@ManyToMany` default to lazy — often changed depending on actual usage patterns.

**139. How do you handle database migrations in a Spring Boot project?**
Using a migration tool like Flyway or Liquibase — you write versioned SQL/XML migration scripts, and the tool automatically applies any new ones at application startup, keeping the database schema in sync across environments in a controlled, trackable way.

**140. What is `spring.jpa.hibernate.ddl-auto` and what are its options?**
It controls how Hibernate manages your database schema at startup — `none` (do nothing), `validate` (check the schema matches your entities but don't change anything), `update` (add missing tables/columns, but never drop/remove anything), and `create`/`create-drop` (recreate the schema from scratch every time — useful for tests, never for production).

---

## Section 8: Transactions

**141. What is `@Transactional`?**
It wraps a method's execution in a database transaction so all its database operations succeed together, or roll back together if anything fails partway through.

**142. How does `@Transactional` work internally?**
Spring wraps your bean in a proxy — calling an `@Transactional` method actually calls this proxy first, which starts a transaction, invokes your real method, and commits or rolls back depending on whether an exception was thrown.

**143. Why doesn't `@Transactional` work when calling a method from within the same class?**
Because the proxy that adds transactional behavior is bypassed entirely when you call `this.someMethod()` directly from inside the same class — you're calling the real object directly, skipping the proxy's transaction logic. The usual fix is moving that method to a separate bean.

**144. What is the default rollback behavior of `@Transactional`?**
By default, Spring only rolls back on unchecked exceptions (subclasses of `RuntimeException`) — checked exceptions do NOT trigger a rollback unless you explicitly configure `rollbackFor`.

**145. How do you make a transaction roll back on a checked exception?**
By specifying `@Transactional(rollbackFor = YourCheckedException.class)`.

**146. What are transaction propagation types, and name a few?**
Propagation defines how a transactional method behaves when called from within another transaction — `REQUIRED` (default: join the existing transaction, or create one if none exists), `REQUIRES_NEW` (always start a fresh, independent transaction, suspending any existing one), and `NESTED` (starts a nested transaction that can roll back independently of the outer one, using savepoints).

**147. What are transaction isolation levels?**
They control how visible one transaction's uncommitted changes are to another — from least to most strict: Read Uncommitted, Read Committed, Repeatable Read, and Serializable, trading off consistency guarantees against performance.

**148. What is a distributed transaction, and why are they hard in microservices?**
It's a transaction spanning multiple independent databases/services, which need to either all commit or all roll back together — hard in microservices because each service typically owns its own database, and there's no single, shared transaction manager across them. Patterns like Saga are used instead of traditional distributed transactions.

**149. What is the Saga pattern?**
It's an approach for managing data consistency across microservices without a distributed transaction — a sequence of local transactions, each triggering the next step via events, with compensating actions defined to undo earlier steps if a later one fails.

**150. Can `@Transactional` be applied at the class level?**
Yes — applying it at the class level makes every public method in that class transactional by default, though you can still override the behavior on individual methods if needed.

**151. What happens if you call a `@Transactional` method on a bean that isn't managed by Spring (e.g., created with `new`)?**
Nothing transactional happens at all — since Spring's transaction handling relies entirely on proxies wrapping Spring-managed beans, an object you create manually with `new` bypasses the whole Spring container and its proxy mechanism.

**152. Is `@Transactional` supported on private methods?**
No — Spring's proxy-based AOP mechanism can't intercept private methods, since they can't be overridden by a proxy subclass, so `@Transactional` silently has no effect if placed on one.

---

## Section 9: Exception Handling & Validation

**153. How do you handle exceptions globally in Spring Boot?**
By creating a class annotated with `@RestControllerAdvice`, containing multiple `@ExceptionHandler` methods, one per exception type you want to handle — Spring automatically routes matching exceptions from any controller to the right handler method.

**154. How do you validate an incoming request body?**
By adding Bean Validation annotations (`@NotNull`, `@NotBlank`, `@Size`, `@Email`, etc.) on your DTO's fields, and putting `@Valid` in front of the `@RequestBody` parameter in your controller — Spring automatically validates and throws an exception if anything fails.

**155. What exception is thrown when `@Valid` validation fails?**
`MethodArgumentNotValidException` — typically caught in a global exception handler to produce a clean, structured list of validation errors instead of a raw stack trace.

**156. How would you create a custom validation annotation?**
By creating your own annotation along with a class implementing `ConstraintValidator`, defining the actual validation logic inside its `isValid()` method — useful for business-specific rules that built-in annotations don't cover.

**157. What is the difference between `@NotNull`, `@NotEmpty`, and `@NotBlank`?**
`@NotNull` just checks the value isn't null. `@NotEmpty` additionally checks a collection/string isn't empty (has at least one element/character). `@NotBlank` is for strings specifically, checking they're not null, not empty, and not just whitespace.

**158. How do you return a custom error response format from your API?**
By defining a standard error DTO (with fields like timestamp, status, message, path), and returning it consistently from every exception handler method in your `@RestControllerAdvice` class.

**159. What is the purpose of a custom exception class, like `ResourceNotFoundException`?**
It makes error handling far more meaningful and specific to your business domain than throwing a generic `RuntimeException` everywhere — and lets you handle each specific kind of failure differently and precisely in your global exception handler.

**160. How do you handle a scenario where multiple validation errors occur at once on a single request?**
Spring collects all failed field validations into a single `BindingResult`/`MethodArgumentNotValidException` object — in your global handler, you iterate over all the field errors and build a combined response listing every validation problem at once, rather than only reporting the first one.

**161. What is `@ExceptionHandler` vs `@ResponseStatus` — when would you use one over the other?**
`@ResponseStatus` is simplest for straightforward cases — just declaring a fixed status code on a custom exception class. `@ExceptionHandler` is more flexible, letting you build a fully custom response body, log the error, or apply different logic depending on the exception's details.

**162. What is a `HandlerExceptionResolver`?**
It's the lower-level Spring MVC mechanism that `@ExceptionHandler`/`@ControllerAdvice` is actually built on top of — you rarely implement it directly, but it's good to know it's the underlying piece resolving exceptions into responses.

---

## Section 10: Spring Security & JWT

**163. What is Spring Security?**
It's Spring's framework for handling authentication (who you are) and authorization (what you're allowed to do), including protection against common security threats like CSRF and session fixation, integrated tightly with the rest of the Spring ecosystem.

**164. What is the Spring Security filter chain?**
It's a sequence of filters that every incoming request passes through before reaching your controller — each filter handles one concern (authentication check, JWT validation, CSRF protection), and a request failing any required check is rejected immediately.

**165. What is JWT and how does it work in a Spring Boot app?**
JWT is a compact, signed token representing user claims (ID, roles, expiry). After login, the server issues a JWT; the client sends it back in the `Authorization` header on every request; the server verifies its signature (without needing shared session storage) to confirm the request is authenticated.

**166. What is the structure of a JWT?**
Three parts separated by dots: a header (specifying the signing algorithm), a payload (the actual claims/data), and a signature (generated using a secret key, letting the server verify the token hasn't been tampered with).

**167. Difference between Authentication and Authorization?**
Authentication verifies who you are (logging in). Authorization determines what you're allowed to do once identified (whether your role permits a specific action).

**168. What is `UserDetailsService`?**
It's an interface you implement to tell Spring Security how to load a user's details (username, password, roles) — usually from your database — so Spring can use that information during the authentication process.

**169. What is `PasswordEncoder`, and why should you never store plain-text passwords?**
It's used to hash passwords before storing them, and to verify a login attempt by hashing the entered password and comparing it — plain text passwords are catastrophic if the database is ever breached, since attackers get immediate access to every account. `BCryptPasswordEncoder` is the most commonly used implementation.

**170. What is CSRF, and how does Spring Security protect against it?**
CSRF (Cross-Site Request Forgery) tricks a logged-in user's browser into unknowingly submitting a malicious request to your app. Spring Security protects against it (for traditional session-based apps) using CSRF tokens that must accompany state-changing requests — though it's typically disabled for stateless, JWT-based REST APIs since there's no session to hijack in the first place.

**171. Why is JWT-based authentication considered stateless?**
Because all the information needed to verify a request is contained within the token itself — the server doesn't need to store any session data to know who's making the request, unlike traditional session-based authentication.

**172. What are some security concerns with using JWT?**
Since a JWT can't be revoked once issued (unlike a server-side session you can just delete), a stolen token remains valid until expiry — mitigated with short expiry times plus refresh tokens. Also, JWT payloads are only encoded, not encrypted, so sensitive data shouldn't be stored inside them.

**173. What is a Refresh Token, and why do we need it alongside an Access Token?**
Access tokens are kept short-lived for security, but that means users would have to log in frequently. A refresh token, which lives longer and is stored more securely, is used to obtain a new access token without requiring the user to log in again, until the refresh token itself expires or is revoked.

**174. What is `@PreAuthorize` used for?**
It lets you specify a security expression directly on a method (like `@PreAuthorize("hasRole('ADMIN')")`), so Spring checks the current user's authorities before allowing that method to execute at all.

**175. What is the difference between `hasRole()` and `hasAuthority()` in Spring Security expressions?**
`hasRole('ADMIN')` internally checks for an authority prefixed with `ROLE_` (so it checks for `ROLE_ADMIN`). `hasAuthority('ADMIN')` checks for the exact authority string as-is, with no automatic prefix — useful when you're using fine-grained permissions rather than broad roles.

**176. What is `SecurityContextHolder`?**
It's where Spring Security stores details about the currently authenticated user for the duration of a request, letting you access the logged-in user's information anywhere in your code without passing it explicitly through every method call.

**177. How would you secure specific endpoints differently (like public vs. admin-only)?**
By configuring a `SecurityFilterChain` bean where you explicitly define URL patterns and their required access rules — for example, permitting `/api/public/**` to everyone, while requiring an `ADMIN` role for `/api/admin/**`.

**178. What is OAuth2, and how does it relate to Spring Security?**
OAuth2 is an authorization framework/protocol that lets a user grant a third-party application limited access to their resources without sharing their password directly — commonly seen in "Login with Google" flows. Spring Security has dedicated OAuth2 support (`spring-security-oauth2`) to act as either a client or resource server in that flow.

**179. What is method-level security, and how do you enable it?**
It lets you apply security rules directly on individual service/controller methods (using `@PreAuthorize`, `@PostAuthorize`, `@Secured`) rather than only at the URL level — enabled via `@EnableMethodSecurity` (or the older `@EnableGlobalMethodSecurity`) on a configuration class.

**180. How do you handle unauthorized/forbidden access attempts gracefully in Spring Security?**
By implementing and registering custom `AuthenticationEntryPoint` (for unauthorized/401 cases) and `AccessDeniedHandler` (for forbidden/403 cases), so your API returns clean, consistent JSON error responses instead of Spring Security's default HTML error pages.

**181. What is a Filter in Spring Security, and how would you write a custom JWT filter?**
A filter intercepts requests before they reach your controllers. A custom JWT filter typically extends `OncePerRequestFilter`, extracts the JWT from the `Authorization` header, validates its signature and expiry, and if valid, sets the authenticated user into the `SecurityContextHolder` before letting the request continue.

**182. Why is `OncePerRequestFilter` commonly used for custom security filters?**
It guarantees the filter's logic executes exactly once per request, even in complex dispatch scenarios (like forwards/includes) where a request might otherwise pass through the filter chain more than once, avoiding duplicate processing.

**183. What is the purpose of disabling CSRF for stateless REST APIs?**
CSRF protection exists to protect session-based, cookie-authenticated apps from malicious cross-site requests. Since stateless JWT APIs don't rely on cookies/sessions for authentication, there's no session to hijack, so CSRF protection is unnecessary overhead and is typically disabled for pure REST APIs.

**184. What is CORS, and how do you configure it in Spring Boot?**
CORS controls whether a browser allows a web page from one origin/domain to make requests to your API hosted on a different origin. You configure it either with `@CrossOrigin` on specific controllers, or globally via a `CorsConfigurationSource` bean, specifying allowed origins, methods, and headers.

**185. What is a common mistake developers make with password storage, and how does Spring Security help avoid it?**
Storing passwords in plain text or with a weak/reversible encoding (like simple Base64) is a common and dangerous mistake. Spring Security's `PasswordEncoder` abstraction (with `BCryptPasswordEncoder` as the standard choice) enforces proper one-way, salted hashing, making it much harder for an attacker to recover the original password even with database access.

---

## Section 11: Spring Boot Actuator & Monitoring

**186. What is Spring Boot Actuator?**
It's a module that adds production-ready monitoring and management features to your application — endpoints exposing health status, metrics, environment info, and more — without you having to build that infrastructure yourself.

**187. What is the `/actuator/health` endpoint used for?**
It reports whether the application (and its key dependencies, like the database) is up and running correctly — commonly used by load balancers and orchestration tools like Kubernetes to decide whether to route traffic to an instance.

**188. How do you enable specific Actuator endpoints?**
By setting `management.endpoints.web.exposure.include` in your properties to list which endpoints should be exposed over HTTP — by default, only a minimal set (usually just `/health`) is exposed for security reasons.

**189. What is the `/actuator/metrics` endpoint used for?**
It exposes detailed application metrics — memory usage, HTTP request counts and timings, thread counts, and more — often scraped by monitoring tools like Prometheus for dashboards and alerting.

**190. How would you secure Actuator endpoints in production?**
By restricting which endpoints are exposed, requiring authentication for sensitive ones (like `/actuator/env` or `/actuator/beans`, which can leak internal configuration), and often exposing them only on a separate internal management port not reachable from the public internet.

**191. What is a custom health indicator, and how do you create one?**
It's a custom class implementing `HealthIndicator`, letting you add your own application-specific health checks (like verifying a critical third-party API is reachable) that get folded into the overall `/actuator/health` response automatically.

**192. What is Micrometer, and how does it relate to Spring Boot Actuator?**
Micrometer is a metrics-collection facade that Actuator uses internally — it provides a vendor-neutral API for recording metrics, which can then be exported to various monitoring backends (Prometheus, Datadog, New Relic) without changing your application code.

**193. How would you monitor a Spring Boot application in production, at a high level?**
Combining Actuator's exposed metrics/health endpoints with a metrics collection system like Prometheus, visualized in a dashboard like Grafana, along with centralized logging (like the ELK stack) and distributed tracing for microservices — giving visibility into health, performance, and errors across the running system.

---

## Section 12: Testing in Spring Boot

**194. What is `@SpringBootTest`?**
It loads the full Spring application context for integration testing — useful when you want to test how multiple components (controller, service, repository) actually work together, though it's slower than a plain unit test since it boots up the whole context.

**195. What is `@WebMvcTest`?**
It loads only the web layer (controllers and related MVC infrastructure) for testing, without starting the full application context — much faster than `@SpringBootTest`, and typically used alongside mocked service dependencies to test controller behavior in isolation.

**196. What is `@DataJpaTest`?**
It loads only the JPA-related components (repositories, entity manager) for testing, using an in-memory database by default — good for testing your repository queries without needing the whole application context.

**197. What is `MockMvc` used for?**
It lets you simulate HTTP requests against your controllers in a test, without actually starting a real server — you can assert on the response status, body, and headers, making controller testing fast and self-contained.

**198. What is Mockito, and how is it used in Spring Boot testing?**
It's a mocking framework used to create fake versions of a class's dependencies, letting you test a class (like a service) in isolation, without needing real database connections or external API calls.

**199. Difference between `@Mock` and `@MockBean`?**
`@Mock` (plain Mockito) creates a mock object for use in a standalone unit test, with no Spring context involved at all. `@MockBean` is Spring Boot's own annotation that replaces a real bean in the Spring application context with a mock, useful in integration tests (`@SpringBootTest`) where you want to fake out just one specific dependency (like an external API client) while the rest of the context loads normally.

**200. What is the difference between Unit Testing and Integration Testing in a Spring Boot context?**
Unit tests check a single class in isolation with all dependencies mocked, running fast without any Spring context. Integration tests verify multiple components genuinely work together correctly — like a controller actually talking to a real (or in-memory) database — running slower but catching issues that only surface when pieces are wired together for real.

**201. What is an embedded/in-memory database, and why use one for tests?**
It's a lightweight database (like H2) that runs entirely in memory and is created fresh for each test run — used so tests don't depend on or pollute a real, shared database, and so tests run fast and are fully repeatable.

**202. How do you test a REST controller's response status and body together?**
Using `MockMvc`, chaining `.andExpect(status().isOk())` and `.andExpect(jsonPath("$.field").value(...))` to assert both the HTTP status and specific fields in the JSON response body in the same test.

**203. What is `TestRestTemplate` used for?**
It's used in full integration tests (with `@SpringBootTest(webEnvironment = RANDOM_PORT)`) to make actual HTTP calls against a genuinely running embedded server, testing the whole stack end-to-end rather than simulating requests like `MockMvc` does.

**204. How would you test a method annotated with `@Transactional` that should roll back on failure?**
By writing a test that triggers the failure condition and then asserting that no partial data was actually persisted afterward — often combined with `@Rollback` on the test itself so any test data changes are automatically undone after the test runs, keeping the test database clean.

**205. What is `@BeforeEach` and `@AfterEach` used for in tests?**
`@BeforeEach` runs setup logic before every single test method (like initializing test data or mocks). `@AfterEach` runs cleanup logic after every test method, ensuring tests don't leave behind state that could affect other tests.

**206. What is the purpose of test profiles (like `application-test.properties`)?**
They let you configure test-specific settings — like pointing to an in-memory database instead of a real one — completely separate from your actual dev/prod configuration, activated automatically when running tests.

**207. How do you verify that a mocked method was actually called during a test?**
Using Mockito's `verify(mockObject).methodName(arguments)`, which asserts the method was invoked (optionally with specific arguments, and optionally a specific number of times) during the test execution.

**208. What is `@Test(expected = ...)` or `assertThrows()` used for?**
They're used to verify that a specific piece of code actually throws an expected exception — `assertThrows()` (the modern JUnit 5 style) lets you capture and further inspect the thrown exception within the test itself.

**209. What is a Test Slice in Spring Boot testing?**
It's a testing annotation (like `@WebMvcTest` or `@DataJpaTest`) that loads only a specific "slice" of the full application context relevant to what you're testing, instead of the whole thing — making tests significantly faster while still testing real Spring wiring for that layer.

**210. How would you approach testing a Spring Boot microservice that depends on another microservice's API?**
By mocking that external API call entirely in unit/service-layer tests (so tests don't depend on network availability), and using a tool like WireMock to simulate the external service's responses in integration tests, giving realistic but controlled and repeatable test conditions.

**211. What is `@ActiveProfiles` used for in tests?**
It explicitly activates a specific Spring profile just for that test class, ensuring it uses test-specific configuration (like an in-memory database) rather than whatever the default active profile might be.

---

## Section 13: Spring AOP (Aspect-Oriented Programming)

**212. What is Aspect-Oriented Programming (AOP), and why does Spring use it?**
AOP lets you separate cross-cutting concerns (logging, security checks, transaction management) from your core business logic, applying that shared behavior across many methods/classes without repeating the same code everywhere. Spring uses AOP internally to implement things like `@Transactional` and method-level security.

**213. What is an Aspect?**
It's a module that encapsulates a cross-cutting concern — for example, a `LoggingAspect` class that automatically logs the execution time of certain methods, without those methods needing to contain any logging code themselves.

**214. What is a Pointcut?**
It's an expression that defines exactly which methods (or classes) an aspect's advice should apply to — for example, "all methods inside any class in the `com.example.service` package."

**215. What is Advice in AOP, and what types exist?**
Advice is the actual action taken at a matched pointcut. `@Before` runs before the method executes, `@After` runs after (regardless of outcome), `@AfterReturning` runs only after a successful return, `@AfterThrowing` runs only if an exception was thrown, and `@Around` wraps the whole method call, letting you control whether it even executes at all.

**216. How does Spring AOP actually intercept method calls internally?**
By creating a dynamic proxy around your bean at startup — when you call a method on that bean, you're actually calling the proxy first, which runs any applicable advice before/after/around delegating to your real method.

**217. What is the limitation of Spring AOP compared to full AspectJ?**
Spring AOP only works on Spring-managed beans and only intercepts method calls made through the proxy (external calls) — it can't apply to private methods, or to internal self-invocations within the same class, unlike full AspectJ which weaves aspects directly into compiled bytecode.

**218. Give a practical real-world use case for a custom Spring AOP aspect.**
A logging aspect that automatically records the execution time of every service method, or an auditing aspect that logs which user performed which action, without cluttering every single business method with repetitive logging/auditing code.

**219. What is `@Around` advice, and why is it more powerful than `@Before`/`@After`?**
`@Around` advice wraps the entire method invocation, giving you full control — you can inspect/modify arguments before the call, decide whether to actually call the real method at all, catch and handle exceptions, and modify the return value, all of which simple before/after advice can't do on their own.

**220. What does `ProceedingJoinPoint` represent in `@Around` advice?**
It represents the actual method call being intercepted — calling `.proceed()` on it is what actually executes the original method; without calling it, the real method never runs at all.

**221. How would you use AOP to enforce API rate limiting or auditing consistently across many controller methods?**
By writing a pointcut matching all relevant controller methods (or a custom annotation you apply selectively), and an `@Around` advice that checks the rate limit or writes an audit log before/after delegating to the real method — keeping that concern completely separate from your actual business logic.

---

## Section 14: Caching in Spring Boot

**222. What is caching, and why use it?**
Caching stores frequently accessed data in a fast-access layer (often in-memory) so you don't repeatedly hit a slower resource (database, external API) for the same data — improving performance and reducing load on backend systems.

**223. What is `@EnableCaching`?**
It's required at the configuration/application level to activate Spring's caching abstraction, which is what makes annotations like `@Cacheable` actually take effect.

**224. What is `@Cacheable`?**
It marks a method so that its result is cached the first time it's called with a given set of arguments — subsequent calls with the same arguments return the cached result instantly, skipping the method's actual execution entirely.

**225. What is `@CacheEvict`?**
It marks a method to remove one or more entries from the cache when it's called — typically used on update/delete operations, so stale cached data doesn't linger after the underlying data has changed.

**226. What is `@CachePut`?**
Unlike `@Cacheable`, it always executes the actual method and then updates the cache with the new result — useful for update operations where you want the cache refreshed with the latest value, not skipped.

**227. What cache providers can you use with Spring Boot?**
The simple in-memory `ConcurrentMapCache` (fine for basic single-instance use), or more robust external providers like Redis, Ehcache, or Caffeine — Redis is especially popular for microservices since it can be shared across multiple application instances.

**228. What is cache invalidation, and why is it considered a hard problem?**
It's the challenge of making sure cached data gets removed or refreshed exactly when the underlying data actually changes — get it wrong, and you either serve stale data (cache not invalidated) or lose most of the caching benefit (invalidating too aggressively).

**229. How would you cache the result of a method that takes multiple parameters?**
Spring generates a cache key automatically based on all the method's parameters by default — but you can customize the key generation explicitly using the `key` attribute on `@Cacheable`, like `@Cacheable(value = "users", key = "#userId")`.

**230. What is a distributed cache, and why would you need one in a microservices setup?**
A distributed cache (like Redis) is shared across multiple application instances, so all instances see the same cached data consistently — a local, in-memory cache would mean each instance has its own separate, inconsistent cache, defeating the purpose in a horizontally scaled deployment.

**231. What is cache eviction policy, and name a couple of common ones.**
It's the rule that decides which entries get removed when a cache reaches its size limit — LRU (Least Recently Used, evicting the entry that hasn't been accessed in the longest time) and LFU (Least Frequently Used, evicting the entry accessed the fewest times) are two of the most common strategies.

---

## Section 15: Scheduling & Async Processing

**232. How do you schedule a task to run every night at midnight in Spring Boot?**
Using `@Scheduled(cron = "0 0 0 * * *")` on a method, after enabling scheduling with `@EnableScheduling` on a configuration class.

**233. Difference between `fixedRate` and `fixedDelay` in `@Scheduled`?**
`fixedRate` triggers the task at a consistent interval regardless of how long the previous execution took (so tasks could overlap if one runs long). `fixedDelay` waits for the specified interval only after the previous execution has finished, guaranteeing no overlap between runs.

**234. What is a cron expression, and what does `0 0 12 * * MON-FRI` mean?**
It's a string format defining a schedule using seconds, minutes, hours, day-of-month, month, and day-of-week fields. That specific expression means "run at exactly 12:00 PM, every weekday (Monday through Friday)."

**235. Why would you use `@Async` instead of just letting a method run normally?**
To let a slow or non-critical operation (like sending an email or generating a report) run in the background on a separate thread, so it doesn't block or slow down the response to the original caller/user.

**236. What is a common mistake with `@Async` methods that a lot of developers make?**
Calling an `@Async` method from within the same class — just like `@Transactional`, this bypasses the Spring proxy responsible for making it actually run asynchronously, so it silently runs synchronously on the same thread instead.

**237. How do you get the result of an asynchronous method call?**
By having the `@Async` method return a `CompletableFuture<T>` (or the older `Future<T>`), which the caller can later call `.get()` on to retrieve the result once the background task completes.

**238. How would you configure a custom thread pool for `@Async` tasks instead of using the default one?**
By defining a `TaskExecutor` bean (like `ThreadPoolTaskExecutor`) with your desired pool size settings, and referencing it by name in `@Async("yourExecutorBeanName")`.

**239. What happens if an exception is thrown inside an `@Async` void method?**
Since there's no caller waiting for a return value to propagate the exception through, it's silently swallowed by default unless you configure a custom `AsyncUncaughtExceptionHandler` to catch and log/handle it properly.

**240. Why is it important to size your async thread pool carefully in production?**
Too small a pool causes tasks to queue up and delay processing under load; too large a pool can exhaust system resources (memory, CPU context-switching overhead) or overwhelm downstream systems (like a database) with too many simultaneous connections.

**241. Give a real-world example where `@Scheduled` would be used in a banking application.**
A nightly batch job that calculates and applies interest to all savings accounts, or a scheduled task that flags overdue loan payments and triggers reminder notifications, running automatically every day without manual intervention.

---

## Section 16: Messaging (Kafka / RabbitMQ) with Spring Boot

**242. Why would a Spring Boot microservice use Kafka or RabbitMQ instead of calling another service's REST API directly?**
Direct REST calls tightly couple services together — if the receiving service is down, the call fails immediately. Messaging decouples services: the sender publishes an event and moves on, and the receiver processes it whenever it's ready, making the overall system more resilient to individual service outages.

**243. What is a Topic in Kafka?**
It's a named category/channel that messages are published to and consumed from — producers write to a topic, and consumers subscribe to read from it, similar conceptually to a named mailing list.

**244. What is a Partition in Kafka, and why does it matter?**
A topic is split into partitions to allow parallel processing — messages within a single partition are strictly ordered, but across different partitions there's no ordering guarantee, which is important to know when message order matters for your use case.

**245. What is a Consumer Group in Kafka?**
It's a set of consumers working together to process messages from a topic, where each partition is consumed by only one consumer within that group at a time — this is how Kafka allows you to scale out message processing across multiple consumer instances.

**246. Difference between Kafka and RabbitMQ, at a high level?**
Kafka is built for high-throughput, ordered event streaming, and retains messages for a configurable period even after they're consumed, making it great for event sourcing/replay. RabbitMQ is a traditional message broker built around flexible routing and messages typically being removed once acknowledged/consumed, better suited for classic task-queue style workloads.

**247. How do you produce a message to Kafka from a Spring Boot application?**
Using Spring Kafka's `KafkaTemplate`, injecting it into your service and calling `kafkaTemplate.send(topicName, message)`.

**248. How do you consume messages from a Kafka topic in Spring Boot?**
By writing a method annotated with `@KafkaListener(topics = "yourTopic")`, and Spring automatically invokes that method whenever a new message arrives on the topic.

**249. What happens if message processing fails in a Kafka consumer?**
Depending on configuration, the message might be retried, sent to a dead-letter topic for later inspection, or simply logged and skipped — you need to explicitly design this error-handling behavior, since Kafka itself doesn't automatically retry failed business logic for you.

**250. What is a Dead Letter Queue (DLQ)?**
It's a separate queue/topic where messages that repeatedly fail to be processed successfully are routed, so they don't block the main processing pipeline and can be investigated or reprocessed manually later.

**251. What does "at-least-once delivery" mean in messaging systems, and what does it imply for your consumer code?**
It means a message might occasionally be delivered more than once (though never lost) — this implies your consumer logic should be idempotent, meaning processing the same message twice shouldn't cause incorrect results (like double-charging a customer).

**252. What is `@RabbitListener` used for?**
It's Spring AMQP's equivalent of `@KafkaListener` — annotating a method with it makes Spring automatically invoke that method whenever a new message arrives on the configured RabbitMQ queue.

**253. How would you ensure ordering of related events in Kafka (like all events for a specific order)?**
By using the same partition key (like the order ID) when producing those related messages — Kafka guarantees messages with the same key always land in the same partition, and within a single partition, order is strictly preserved.

---

## Section 17: Microservices with Spring Boot

**254. Why is Spring Boot commonly used to build microservices?**
Its embedded server and self-contained executable JAR make each service easy to build, run, and containerize independently. Combined with Spring Cloud, it also provides ready-made solutions for common microservices concerns like service discovery, config management, and resilience.

**255. What is Spring Cloud, and how does it relate to Spring Boot?**
Spring Cloud is a set of tools built on top of Spring Boot specifically for building distributed systems/microservices — providing solutions for service discovery, centralized configuration, load balancing, and circuit breaking, so you don't have to build that infrastructure from scratch.

**256. What is Eureka, and what problem does it solve?**
Eureka is a service discovery server — services register themselves with it on startup, and other services look each other up by name through Eureka rather than hardcoding IP addresses, which is essential in dynamic environments where instances are constantly being created and destroyed.

**257. What is an API Gateway, and why use one in a microservices architecture?**
It's a single entry point that sits in front of all your microservices, routing client requests to the right internal service, and centralizing cross-cutting concerns like authentication, rate limiting, and logging instead of duplicating that logic in every microservice.

**258. What is Spring Cloud Gateway?**
It's Spring's own reactive API Gateway implementation, letting you define routing rules, filters, and cross-cutting policies declaratively, integrated naturally with the rest of the Spring Cloud ecosystem.

**259. What is a Circuit Breaker, and how is it implemented in Spring Boot?**
It prevents a failing downstream service from being repeatedly called and potentially overwhelming both services further — after a threshold of failures, the "circuit opens" and calls fail fast (or return a fallback) for a period, giving the failing service time to recover. Resilience4j (integrated via Spring Cloud Circuit Breaker) is the modern standard implementation.

**260. What is a fallback method in the context of Circuit Breakers?**
It's an alternative piece of logic that runs instead of the actual call when the circuit is open or the real call fails — like returning cached or default data instead of an error, so the user experience degrades gracefully rather than failing completely.

**261. What is the difference between client-side and server-side load balancing?**
Server-side load balancing uses a separate component (like a dedicated load balancer or the API Gateway) to distribute requests across service instances. Client-side load balancing (like Spring Cloud LoadBalancer) lets each calling service itself pick which instance to call, based on a list obtained from service discovery, without needing a separate central load balancer.

**262. How would two microservices maintain data consistency without sharing a database?**
Typically through event-driven communication — when one service's data changes, it publishes an event, and other services that need that information consume it and update their own local copies accordingly, accepting eventual consistency instead of strict, immediate consistency across services.

**263. What is Distributed Tracing, and why is it needed in microservices?**
Since a single user request often flows through multiple microservices, a plain log file from just one service doesn't show the full picture. Distributed tracing (using tools like Zipkin or Spring Cloud Sleuth/Micrometer Tracing) tags each request with a unique trace ID that follows it across every service it touches, letting you reconstruct and debug the entire request's journey.

**264. What is the Strangler Fig pattern, relevant when migrating a monolith to microservices?**
It's a gradual migration strategy — instead of rewriting the whole monolith at once, you incrementally build new functionality as separate microservices and route relevant traffic to them, slowly "strangling" and replacing pieces of the old monolith over time until it can eventually be retired.

**265. What is the database-per-service pattern in microservices?**
Each microservice owns and manages its own private database, which no other service is allowed to access directly — services can only get each other's data through well-defined APIs or events, keeping services properly decoupled and independently deployable.

**266. How do you handle centralized logging across many microservices?**
By having every service write structured logs (often including a shared trace/correlation ID), which are then aggregated centrally using a tool like the ELK stack (Elasticsearch, Logstash, Kibana) or a cloud-native logging service, so you can search and correlate logs across all services in one place.

**267. What is a health check endpoint used for in a microservices deployment (like Kubernetes)?**
Kubernetes (or any orchestrator) periodically calls a service's health endpoint to determine if it's running correctly — if it's unhealthy, the orchestrator can automatically restart the instance or stop routing traffic to it until it recovers.

**268. What is contract testing, and why does it matter between microservices?**
It verifies that the actual API contract between a consumer service and a provider service (the exact expected request/response shape) hasn't been broken, without needing to spin up the full other service during testing — catching integration-breaking changes early, before they reach production.

**269. What is the Sidecar pattern?**
It's an architectural pattern (common in service mesh setups like Istio) where auxiliary functionality — logging, monitoring, security — runs in a separate, co-located container alongside the main application container, handling cross-cutting concerns without the main application needing to implement them itself.

**270. What challenges does a microservices architecture introduce that a monolith doesn't have?**
Network latency and failure handling between services, harder-to-maintain data consistency across separate databases, more complex debugging/tracing across service boundaries, and significantly more operational overhead (deployment, monitoring, versioning) compared to managing a single deployable unit.

**271. What is Blue-Green deployment, and how is it used with Spring Boot microservices?**
It's a deployment strategy where you run two identical production environments ("blue" is the current live version, "green" is the new version) — once the green environment is verified healthy, traffic is switched over to it entirely, allowing near-zero-downtime deployments and an easy rollback (just switch traffic back to blue) if something goes wrong.

---

## Section 18: Logging in Spring Boot

**272. What logging framework does Spring Boot use by default?**
Logback, accessed through the SLF4J logging facade — SLF4J lets your code depend on a generic logging API without being tightly coupled to a specific logging implementation underneath.

**273. How do you change the logging level for a specific package in Spring Boot?**
By setting a property like `logging.level.com.example.service=DEBUG` in your `application.properties`, which controls how verbose that specific package's logging output is, independent of the rest of the application.

**274. What is the difference between logging levels like DEBUG, INFO, WARN, and ERROR?**
DEBUG is detailed, low-level information mainly useful during development/troubleshooting. INFO records normal but noteworthy application events. WARN indicates something unexpected happened but the application can still continue. ERROR indicates a real failure that likely needs attention.

**275. How would you write logs to a file instead of just the console?**
By setting `logging.file.name=app.log` (or `logging.file.path` for a directory) in your properties — Spring Boot's default Logback configuration handles writing to that file automatically, alongside or instead of console output.

**276. Why is structured (JSON) logging often preferred in production microservices?**
Plain text logs are hard to search and correlate at scale across many services. Structured JSON logs can be easily parsed, indexed, and queried by centralized logging systems (like the ELK stack), making it much easier to filter, search, and build dashboards from log data.

**277. What is MDC (Mapped Diagnostic Context), and why is it useful?**
It lets you attach contextual information (like a request ID or user ID) to a thread, which then automatically appears in every log line written during that request's processing — extremely useful for tracing a single request's full journey through logs, especially in concurrent or multi-service systems.

**278. What's a common logging mistake developers make in production applications?**
Logging sensitive data (passwords, tokens, full card numbers) directly, which is both a security and compliance risk, and logging excessively at DEBUG/INFO level in production, which bloats storage costs and makes it harder to find genuinely important log entries.

**279. What is log rotation, and why is it important?**
It's the practice of automatically archiving/compressing older log files and starting a fresh one after a certain size or time period, preventing a single log file from growing indefinitely and eventually filling up disk space.

---

## Section 19: Deployment, Docker & DevOps

**280. How do you containerize a Spring Boot application with Docker?**
By writing a `Dockerfile` that starts from a base Java image, copies your built executable JAR into the image, and specifies the command to run it (`java -jar app.jar`) — this produces a portable image that runs identically anywhere Docker is supported.

**281. What is a multi-stage Docker build, and why use one for Spring Boot?**
It uses multiple `FROM` stages in a single Dockerfile — one stage builds/compiles your application (needing the full JDK and build tools), and a final, much smaller stage just copies the finished JAR into a lightweight runtime image — keeping your final production image size much smaller and more secure.

**282. What is the purpose of a `.dockerignore` file?**
It tells Docker which files/folders to exclude when building the image (like your `.git` folder or local IDE files), keeping the build context smaller, faster, and avoiding accidentally including unnecessary or sensitive files in the image.

**283. How do you pass environment-specific configuration into a Dockerized Spring Boot app?**
By passing environment variables at container runtime (using `docker run -e` or a Kubernetes deployment spec's `env` section), which Spring Boot automatically picks up thanks to its relaxed property binding, without needing to rebuild the image for different environments.

**284. What is a health check in Kubernetes, and what Spring Boot Actuator endpoint typically backs it?**
Kubernetes periodically probes a container to see if it's alive (liveness probe) and ready to receive traffic (readiness probe) — these are typically configured to hit Spring Boot Actuator's `/actuator/health` endpoint, letting Kubernetes automatically restart or stop routing to unhealthy instances.

**285. What is the difference between a Liveness Probe and a Readiness Probe in Kubernetes?**
A liveness probe checks whether the application is still running correctly at all — if it fails repeatedly, Kubernetes restarts the container. A readiness probe checks whether the application is currently ready to accept traffic — if it fails, Kubernetes stops routing new requests to it, without necessarily restarting it (useful during slow startup or temporary overload).

**286. What is a CI/CD pipeline, and how does it typically work for a Spring Boot project?**
Continuous Integration/Continuous Deployment automates the process of building, testing, and deploying your application whenever code is pushed — typically: code is pushed to a repository, a pipeline (like Jenkins, GitHub Actions, or GitLab CI) automatically runs tests and builds the JAR/Docker image, and if everything passes, it's automatically deployed to a staging or production environment.

**287. Why is it a good practice to keep configuration separate from your application code (the twelve-factor app principle)?**
It lets the exact same build artifact be deployed unchanged across every environment (dev, staging, prod), with only the external configuration differing — this avoids the risk of environment-specific bugs caused by rebuilding code differently per environment, and keeps secrets out of your source code.

**288. What is horizontal scaling, and how does it typically apply to a Spring Boot microservice?**
Instead of making a single instance more powerful (vertical scaling), you run multiple identical instances of the same service behind a load balancer, distributing traffic across them — Spring Boot's stateless, containerizable nature makes this straightforward, especially when combined with an orchestrator like Kubernetes that can automatically scale instance count up or down based on load.

---

## Section 20: Rapid Fire & Miscellaneous

**289. What is the difference between `spring-boot-starter-web` and `spring-boot-starter-webflux`?**
`spring-boot-starter-web` builds traditional, blocking, servlet-based web applications (Spring MVC, backed by Tomcat). `spring-boot-starter-webflux` builds reactive, non-blocking applications using Project Reactor, better suited for handling very high concurrency with fewer threads.

**290. What is Reactive Programming, in plain terms?**
It's a programming style built around asynchronous data streams that can emit values over time, rather than blocking a thread while waiting for a single result — you react to data as it arrives, which lets a small number of threads handle a very large number of concurrent requests efficiently.

**291. What is `WebClient`, and how is it different from `RestTemplate`?**
`WebClient` is Spring's modern, non-blocking, reactive HTTP client — it doesn't block a thread while waiting for a response. `RestTemplate` is the older, traditional, blocking HTTP client, and is now considered in maintenance mode, with `WebClient` recommended for new development.

**292. What is Spring Boot DevTools used for?**
It's a development-time convenience module that automatically restarts your application whenever it detects a code change, and provides other developer-friendly features like disabled template caching — never meant to be included in a production build.

**293. What is the difference between `CommandLineRunner` and `ApplicationRunner`?**
Both let you run some code right after the Spring application context has started up. `CommandLineRunner` gives you raw String array command-line arguments. `ApplicationRunner` gives you a more structured `ApplicationArguments` object, letting you access named options more conveniently.

**294. What is a `ShutdownHook` in the context of Spring Boot, and why might you need one?**
It's logic that runs automatically when the application is shutting down gracefully — useful for cleanup tasks like finishing in-flight requests, closing database connections properly, or flushing any buffered data before the process actually exits.

**295. What is Graceful Shutdown in Spring Boot, and why does it matter?**
It allows the application, on receiving a shutdown signal, to stop accepting new requests but finish processing any requests already in progress before actually terminating — important in production to avoid abruptly cutting off users mid-request during deployments or scaling events.

**296. What is `spring.main.web-application-type` used for?**
It explicitly tells Spring Boot what kind of application to bootstrap — `SERVLET` for a traditional web app, `REACTIVE` for a WebFlux app, or `NONE` for a non-web application (like a batch job or CLI tool) — usually auto-detected, but occasionally needs to be set explicitly.

**297. What is Spring Batch, and when would you use it over just writing a plain scheduled method?**
Spring Batch is a dedicated framework for processing large volumes of data in structured, resumable steps (read-process-write), with built-in support for things like chunk-based processing, retries, and restart-from-failure. You'd reach for it for genuinely large or complex batch jobs rather than simple lightweight scheduled tasks.

**298. What is the difference between a monolithic Spring Boot application and a modular monolith?**
A traditional monolith often has all code loosely organized with unclear boundaries between features. A modular monolith is still deployed as a single unit, but is deliberately organized into clearly separated, loosely coupled modules internally — giving some of the organizational benefits of microservices without the operational complexity of actually splitting into separate deployable services.

**299. What is the purpose of `spring-boot-maven-plugin` (or the Gradle equivalent)?**
It's what actually packages your application into an executable "fat" JAR (bundling dependencies and providing the necessary manifest so `java -jar` works), and also provides convenient commands like running the app directly through the build tool during development.

**300. What is Command Query Responsibility Segregation (CQRS), and how might it relate to a Spring Boot system?**
It's a pattern that separates the logic (and sometimes even the data models/databases) used for writing/updating data from the logic used for reading/querying it — useful in systems with very different read vs. write scaling needs. In Spring Boot, this might look like separate service classes, or even entirely separate microservices, for commands (writes) versus queries (reads).

**301. What is the difference between `@RequestMapping(method = RequestMethod.GET)` and `@GetMapping` — is there any real difference?**
No functional difference at all — `@GetMapping` is simply a more concise, readable shortcut annotation that Spring provides specifically for the GET case of `@RequestMapping`.

**302. How would you handle backward compatibility when changing a Spring Boot REST API that other teams depend on?**
By versioning the API (like introducing `/v2/` alongside the existing `/v1/`) rather than modifying the existing contract in place, giving consumers time to migrate, and only deprecating/removing the old version once you're confident nothing still depends on it.

**303. What is the significance of Spring Boot's "convention over configuration" philosophy?**
It means Spring Boot assumes sensible defaults for almost everything (like where to find templates, what port to run on, how to name database tables) so you only need to write configuration for the things where you actually want to deviate from those defaults — massively reducing the amount of boilerplate setup needed to get a working application.

**304. In one sentence, what's the single biggest reason companies choose Spring Boot for backend development?**
It combines the maturity and power of the broader Spring ecosystem with drastically reduced setup time and boilerplate, letting teams build production-ready, well-tested, easily deployable applications quickly, without sacrificing flexibility for more complex needs later.

---

## How to use this document
- Don't memorize word-for-word — say each answer out loud in your own phrasing once. Interviewers can tell the difference between "understood" and "recited."
- Group your prep by section rather than trying to cover everything in one sitting — 15–20 questions a day sticks far better than cramming 300 in a weekend.
- Wherever an answer describes a concept (like N+1 queries, or `@Transactional` self-invocation), try to also recall a specific moment from your own project at Wipro where you actually ran into it — a real example always lands better than a definition alone.
