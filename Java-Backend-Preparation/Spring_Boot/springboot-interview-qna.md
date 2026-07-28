# Spring Boot — Interview Q&A (All Topics)

*Questions phrased the way interviewers actually ask them. Answers written in plain spoken English — the way you'd say it out loud in a room, not how a textbook would print it. Use these to practice saying answers, not just reading them.*

---

## Table of Contents

1. [Introduction & Core Concepts](#1-introduction--core-concepts)
2. [Project Setup & Structure](#2-project-setup--structure)
3. [Core Annotations](#3-core-annotations)
4. [Dependency Injection & Bean Lifecycle](#4-dependency-injection--bean-lifecycle)
5. [Auto-Configuration](#5-auto-configuration)
6. [Configuration, Properties & Profiles](#6-configuration-properties--profiles)
7. [REST APIs](#7-rest-apis)
8. [Validation](#8-validation)
9. [Exception Handling](#9-exception-handling)
10. [Spring Data JPA](#10-spring-data-jpa)
11. [Transactions](#11-transactions)
12. [Spring Security & JWT](#12-spring-security--jwt)
13. [Actuator & Observability](#13-actuator--observability)
14. [AOP](#14-aop)
15. [Caching](#15-caching)
16. [Scheduling & Async](#16-scheduling--async)
17. [Testing](#17-testing)
18. [Swagger/OpenAPI](#18-swaggeropenapi)
19. [File Upload & Download](#19-file-upload--download)
20. [Microservices](#20-microservices)
21. [Deployment & Production](#21-deployment--production)

---

## 1. Introduction & Core Concepts

**Q1. What is Spring Boot, and how is it different from the Spring Framework?**

So Spring Framework is the core — it gives you dependency injection, AOP, all of that. But setting up a plain Spring project used to mean a ton of manual configuration — you'd wire up the dispatcher servlet, configure a data source by hand, pick compatible versions of ten different libraries yourself, deploy to an external Tomcat. Spring Boot sits on top of Spring and takes all of that pain away. It gives you auto-configuration, so it looks at what's on your classpath and configures beans for you sensibly. It gives you starter dependencies so you're not chasing version compatibility. And it embeds Tomcat right inside the JAR, so you just run `java -jar` and you're live. So I'd say Spring Boot isn't a competing framework — it's Spring made fast to start with.

**Q2. Why do we need Dependency Injection at all? What problem does it solve?**

Without DI, your classes create their own dependencies — like `new RazorpayGateway()` right inside your service. That means your service is tightly coupled to that one specific implementation. If you want to swap it for a mock in tests, or swap providers later, you have to go change the code everywhere it's used. With DI, the container hands you the dependency — usually through the constructor — so your class doesn't care what the concrete implementation is, only that it satisfies an interface. That gives you loose coupling, and honestly the biggest day-to-day win is testability — I can just pass in a mock in a unit test without touching Spring at all.

**Q3. Can you walk me through what happens when a request hits a Spring Boot application?**

Sure — request comes in, hits the embedded Tomcat server first. From there it goes to the `DispatcherServlet`, which is Spring's front controller — basically the single entry point for every request. The `DispatcherServlet` uses handler mapping to figure out which controller method should handle this URL and HTTP method. Before it actually reaches the controller, it passes through any filters and interceptors — that's where things like authentication checks or logging usually happen. Then it hits your controller method, which typically delegates to a service, which talks to a repository, which talks to the database. The response comes back up through the same chain and gets serialized — usually to JSON by Jackson — before going back to the client.

**Q4. Is Spring Boot suitable for microservices, and why?**

Yeah, it's basically the default choice in the Java world for microservices. The big reason is the embedded server — each service is just a self-contained JAR you can run independently, no shared app server. Combine that with Spring Cloud for things like service discovery, config management, and circuit breakers, and you get a pretty complete toolkit. It also starts fast and has a small footprint compared to a full app-server deployment, which matters when you're running dozens of small services instead of one big one.

**Q5. What are the main advantages of using Spring Boot over plain Spring?**

I'd list four things: auto-configuration so you're not hand-wiring beans, starter POMs so dependency management isn't a headache, an embedded server so deployment is just running a JAR, and built-in production tooling through Actuator — health checks, metrics, all of that for free. On top of that, it pushes you toward Java-based configuration instead of XML, which is just nicer to work with and easier to refactor.

---

## 2. Project Setup & Structure

**Q1. How would you structure a Spring Boot project, and why organize it that way?**

I go with a layered package structure — controller, service, repository, entity, dto, config, exception. The idea is separation of concerns: the controller only deals with HTTP stuff, request in, response out. The service holds the actual business logic and is where transaction boundaries live. The repository is pure persistence, no business rules in there. And DTOs are what actually goes over the wire to the client — I never expose JPA entities directly, because entities carry things like lazy-loaded relationships and internal fields that shouldn't leak into an API response. This structure makes it easy for anyone new to the codebase to guess where something lives.

**Q2. What's inside `@SpringBootApplication`, and why does it matter where you place the main class?**

It's actually three annotations bundled into one — `@SpringBootConfiguration`, which just marks the class as a source of bean definitions, `@EnableAutoConfiguration`, which kicks off Spring Boot's whole auto-configuration mechanism, and `@ComponentScan`, which scans for `@Component`, `@Service`, `@Repository`, `@Controller` classes. The component scan defaults to scanning downward from the package the main class sits in. So if you put your main class in some sub-package instead of the root, anything in a sibling package just won't get picked up — and that's a classic "why isn't my bean being found" bug that's really just a package placement mistake.

**Q3. What's the difference between Maven and Gradle, and does it matter which one you use?**

Functionally they solve the same problem — dependency management and build lifecycle. Maven uses XML and a very rigid, convention-based lifecycle, which is why you see it a lot in enterprise and banking shops — it's predictable and stable. Gradle uses a Groovy or Kotlin DSL, and it's generally faster because of incremental builds and better caching, which is part of why a lot of product companies and Android projects lean toward it. Honestly for day-to-day development it doesn't change how you write Spring Boot code — it's really a build-tooling preference, not an architecture decision.

**Q4. What is the `spring-boot-starter-parent`, and what does it actually give you?**

It's a parent POM that Spring Boot provides, and its main job is dependency and plugin version management — it's basically a giant BOM, a Bill of Materials. So when I add `spring-boot-starter-web` without specifying a version, it's the parent POM that's pinning that to a version it knows is compatible with everything else in that Spring Boot release. It also sets sensible defaults like the Java compiler version and packaging. It saves you from the classic "dependency hell" where two libraries pull in conflicting versions of something.

**Q5. Why would you separate your project into DTOs instead of just using entities everywhere?**

A few real reasons. First, entities often have lazy-loaded associations — if you serialize an entity outside of a transaction, you can hit a `LazyInitializationException`. Second, entities can carry fields you never want exposed externally — internal notes, audit columns, whatever. Third, and probably the most important one long-term, is that if your API contract is literally your database schema, then every migration becomes a potential breaking change for your API consumers. A DTO decouples those two things, so I can refactor my database without breaking the contract clients depend on.

---

## 3. Core Annotations

**Q1. What's the actual difference between `@Component`, `@Service`, and `@Repository`?**

Functionally, `@Component` and `@Service` are identical — `@Service` is just a more specific name so the code reads better and communicates intent, this is business logic. `@Repository` is where there's an actual functional difference — it enables exception translation. So if Hibernate throws some low-level exception like a `ConstraintViolationException`, `@Repository` causes Spring to catch that and translate it into its own unified `DataAccessException` hierarchy. That means your service layer can catch a consistent Spring exception type regardless of whether you're using JPA, JDBC, or something else underneath.

**Q2. What's the difference between `@Controller` and `@RestController`?**

`@Controller` is meant for traditional MVC apps where a method's return value is treated as a view name — like the name of a Thymeleaf template to render. `@RestController` is `@Controller` combined with `@ResponseBody`, which means every method's return value gets written straight into the HTTP response body — as JSON typically — instead of being resolved to a view. So for building a REST API, you almost always want `@RestController`.

**Q3. When would you use `@Bean` instead of `@Component`?**

`@Component` is for classes you own — you put the annotation right on your own class and Spring picks it up during component scanning. `@Bean` is for when you don't own the class — like a third-party library class you can't annotate — or when constructing the object needs some custom logic that doesn't belong in a constructor. You put `@Bean` on a method inside a `@Configuration` class, and whatever that method returns becomes a Spring-managed bean.

**Q4. What does `@Qualifier` do, and when do you need it?**

You need it when there's more than one bean of the same type in the context and Spring can't figure out which one to inject just by type. Say I have two implementations of a `MessageSender` interface — one for SMS, one for email. If I just `@Autowired` a `MessageSender` field, Spring doesn't know which one I want. `@Qualifier("smsSender")` tells it exactly which bean by name. `@Primary` is the other way to solve this — it marks one bean as the default — but `@Qualifier` at the injection point always wins if both are present.

**Q5. Why should you avoid using `@Data` from Lombok on a JPA entity?**

`@Data` generates `equals()`, `hashCode()`, and `toString()` over every field, including relationships. On a JPA entity with bidirectional associations, that can cause infinite recursion — entity A's `toString()` calls entity B's `toString()`, which calls back into A, and so on. It also means your logs could accidentally print an entire object graph, including sensitive fields. The safer approach is to be explicit — use `@Getter` and `@Setter`, and if you need `equals`/`hashCode`, base it just on the ID field, not the whole object.

**Q6. What's the purpose of `@RequestMapping`, and how does it relate to `@GetMapping`, `@PostMapping`, and so on?**

`@RequestMapping` is the general-purpose mapping annotation — you can specify the HTTP method, path, headers, whatever you need. `@GetMapping`, `@PostMapping`, and the others are just shortcuts — `@GetMapping("/orders")` is exactly the same as `@RequestMapping(method = RequestMethod.GET, path = "/orders")`, just shorter and more readable. In practice you almost always use the shortcut versions.

---

## 4. Dependency Injection & Bean Lifecycle

**Q1. Why is constructor injection preferred over field injection?**

A few solid reasons. First, with constructor injection your dependencies can be declared `final`, which gives you immutability and guarantees the object is never in a half-initialized state. Second, it makes dependencies explicit — if your constructor suddenly needs eight parameters, that's an obvious signal your class is doing too much, whereas with field injection that same problem is hidden. Third, and this is the one I actually care about day to day — it makes testing so much easier. I can just call `new OrderService(mockGateway)` in a plain JUnit test without needing Spring involved at all. Field injection needs reflection tricks to set private fields in tests, which is just more friction.

**Q2. What's the danger of field injection specifically?**

The main danger is that a class can silently have way too many dependencies without you noticing, because there's no constructor signature forcing you to see them all at once. It's also harder to make the object immutable, and it makes unit testing without a Spring context awkward, since you typically need reflection or Mockito's `@InjectMocks` doing that reflection for you rather than just calling a constructor directly.

**Q3. What are the different bean scopes in Spring, and when would you use something other than singleton?**

Singleton is the default — one instance for the whole container, shared everywhere. Prototype gives you a brand new instance every time the bean is requested — useful for something stateful, like a report generator that holds per-request state. Then there's request and session scope, which only make sense in a web context — one instance per HTTP request or per HTTP session. Most of the time you're on singleton and don't think about it, but I've used prototype scope for things like a stateful builder object that shouldn't be shared across concurrent requests.

**Q4. What's the gotcha with injecting a prototype-scoped bean into a singleton bean?**

This one catches people off guard. If you inject a prototype bean directly into a singleton's field, that injection only happens once — when the singleton itself is created. So you end up with what looks like a prototype bean but behaves like a singleton, because you're holding onto that one instance forever. The fix is to inject an `ObjectProvider` of that type instead, and call `.getObject()` each time you actually need a fresh instance — that way you get a new prototype instance on every call, not just once at wiring time.

**Q5. What is `@PostConstruct` used for, and how is it different from putting that logic in the constructor?**

`@PostConstruct` runs after Spring has fully constructed the bean and injected all its dependencies. The key difference from a constructor is that inside `@PostConstruct`, you know for sure every dependency has already been wired in — so it's the right place for setup logic that actually needs those dependencies to be ready, like warming up a cache using an injected repository. A constructor technically has the dependencies too, but by convention we keep constructors minimal — just assignment — and put actual initialization logic in `@PostConstruct`.

**Q6. What happens with circular dependencies in Spring, and how should you actually fix them?**

If Bean A needs Bean B in its constructor, and Bean B needs Bean A in its constructor, Spring can't resolve that — with constructor injection it fails at startup with a `BeanCurrentlyInCreationException`. Honestly, I think that's a good thing, because it surfaces a real design problem immediately instead of letting it slide. The right fix is almost always to refactor — pull the shared logic both beans need into a third class they can both depend on. You can technically get around it with `@Lazy` on one of the injections, but that's a band-aid, not a fix — I'd only reach for it as a last resort under real time pressure.

---

## 5. Auto-Configuration

**Q1. Can you explain how Spring Boot's auto-configuration actually works under the hood?**

At startup, `@EnableAutoConfiguration` triggers Spring Boot to scan a file — in newer versions it's `AutoConfiguration.imports` under `META-INF/spring` — across every JAR on the classpath. That file lists a huge number of `@Configuration` classes, things like `DataSourceAutoConfiguration` or `WebMvcAutoConfiguration`. Each of those classes is guarded by conditional annotations — `@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty` — so Spring Boot only actually activates a given auto-configuration if the right classes are present and you haven't already defined your own conflicting bean. That's the whole "magic" — it's really just a big set of conditionally-activated configuration classes shipped with the framework.

**Q2. If you define your own `DataSource` bean, what happens to Spring Boot's auto-configured one?**

It just backs off. The auto-configured `DataSource` bean is annotated `@ConditionalOnMissingBean`, so the moment you define your own `DataSource` bean anywhere in your application context, that condition fails and Spring Boot's version never gets created. You don't need to explicitly disable or exclude anything — defining your own bean is enough.

**Q3. How would you explicitly exclude a specific auto-configuration?**

You'd use the `exclude` attribute on `@SpringBootApplication`, like `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)`. A real case where you'd need this — say you're building a service with no database at all, but you've still got some starter on the classpath that pulls in JPA transitively. Without the exclusion, Spring Boot tries to auto-configure a data source, can't find connection properties, and your app fails to start.

**Q4. What's the difference between `@ConditionalOnClass` and `@ConditionalOnBean`?**

`@ConditionalOnClass` checks whether a certain class exists on the classpath at all — so it's really a "is this dependency present" check, decided basically at compile/packaging time. `@ConditionalOnBean` checks whether a bean of a certain type already exists in the Spring context at the point that condition is evaluated — so it's a runtime check based on what's already been configured. You'll see both used together a lot — "only configure this if the class is on the classpath, and only if nobody's already defined this bean."

**Q5. Adding `spring-boot-starter-data-jpa` to your `pom.xml` seems to "magically" set up a data source and transaction manager with zero code. How does that actually happen?**

It's not really magic, it's the conditional configuration mechanism working as designed. The starter pulls in Hibernate, the JPA API, and a few other classes onto the classpath. That satisfies the `@ConditionalOnClass` checks in things like `DataSourceAutoConfiguration`, `HibernateJpaAutoConfiguration`, and `JpaRepositoriesAutoConfiguration`. Since you haven't defined your own conflicting beans, those conditions pass and Spring Boot builds a `DataSource`, an `EntityManagerFactory`, and a `PlatformTransactionManager` for you, using whatever properties you've set in `application.yml` for the connection details.

---

## 6. Configuration, Properties & Profiles

**Q1. What's the difference between `application.properties` and `application.yml`, and which do you prefer?**

They configure the exact same things, just different syntax. Properties files are flat key-value pairs, YAML lets you nest things hierarchically, which reads a lot cleaner once you've got deeply nested config like `spring.datasource.hikari.maximum-pool-size`. I generally prefer YAML for anything beyond a handful of properties, just because the nesting makes the structure of the config obvious at a glance. The one thing to be careful with in YAML is indentation — it's whitespace-sensitive, so a misplaced space can silently put a property under the wrong parent.

**Q2. What are Spring profiles, and how do you use them?**

Profiles let you swap out configuration depending on which environment you're running in — dev, test, prod — without touching code. You'd have `application-dev.yml`, `application-prod.yml`, alongside the base `application.yml`, and each one overrides or adds properties specific to that environment. You activate one by setting `spring.profiles.active`, either as a command-line argument or an environment variable. You can also mark entire beans with `@Profile("dev")` so, say, a mock payment gateway bean only gets created when you're running under the dev profile, and a real one gets created under prod.

**Q3. What's `@ConfigurationProperties`, and why would you use it instead of a bunch of `@Value` annotations?**

`@Value("${some.property}")` works fine for a single value, but once you've got a group of related properties — like a whole block of payment gateway config, URL, timeout, retry count — scattering `@Value` annotations everywhere gets messy and it's easy to typo a property key with no compile-time safety. `@ConfigurationProperties` lets you bind a whole prefix, like `app.payment`, to a strongly-typed class with getters and setters. It's type-safe, IDE-autocompletable, and if a property's missing or misnamed you'll usually catch it earlier than you would with scattered `@Value` calls.

**Q4. Why is `spring.jpa.hibernate.ddl-auto=update` considered dangerous in production?**

Because it lets Hibernate automatically alter your database schema to match your entity classes — and in production, that's not something you want happening automatically and silently. It can add columns, and in some cases even drop ones it thinks are no longer needed, based purely on what your Java entities look like at that moment. If someone deploys a slightly wrong entity, you could lose data or corrupt the schema with zero review. In production you want `ddl-auto` set to `validate` — it just checks the schema matches your entities and fails loudly if it doesn't — and actual schema changes should go through a proper migration tool like Flyway, where every change is a reviewed, versioned SQL script.

**Q5. How would you avoid hardcoding a database password in your config files?**

I'd never put it directly in `application.yml`, especially not in a file that's committed to git. Instead I'd reference an environment variable, something like `password: ${DB_PASSWORD}`, and that variable gets injected at deploy time — through Docker environment variables, a Kubernetes secret, or ideally pulled from an actual secrets manager like AWS Secrets Manager or Vault. The config file itself never contains the actual secret value.

**Q6. What's Flyway, and why would you use it instead of just letting Hibernate manage the schema?**

Flyway is a database migration tool — you write versioned SQL scripts, like `V1__create_users_table.sql`, `V2__add_email_index.sql`, and Flyway tracks which ones have already run against a given database in a history table, and applies any new ones automatically on startup. The big advantage over letting Hibernate auto-generate schema changes is that every change is explicit, reviewable, and reversible — which matters a lot in something like banking, where a schema change might need sign-off, and you absolutely want an audit trail of exactly what changed and when.

---

## 7. REST APIs

**Q1. Why use `ResponseEntity<T>` instead of just returning the object directly from a controller method?**

If you just return the object, Spring always sends back a 200 with that object as JSON — you don't get any control over the status code or headers. `ResponseEntity` lets you actually express the real HTTP semantics of what happened — return 201 with a `Location` header when something's created, 204 with an empty body on a successful delete, 404 when it's not found. For any API that's meant to be consumed by other teams or external clients, that level of control over status codes really matters, because a lot of client-side logic and monitoring depends on the status code being accurate, not just the body content.

**Q2. What's the difference between `@PathVariable` and `@RequestParam`?**

`@PathVariable` pulls a value out of the URL path itself — like the `42` in `/orders/42`. `@RequestParam` pulls a value from the query string — the `status` in `/orders?status=PAID`. As a rule of thumb, path variables are for identifying a specific resource, and request params are for filtering, sorting, or optional modifiers on a request.

**Q3. Why shouldn't you return JPA entities directly from a REST controller?**

Because entities and DTOs serve different purposes. An entity might have a lazy-loaded relationship — try to serialize that outside of an active transaction and you'll get a `LazyInitializationException`. Entities can also carry internal-only fields that were never meant to be exposed externally. And probably the biggest long-term issue — if your API response is literally your entity, then your database schema and your public API contract are the same thing, so any migration risks becoming a breaking change for whoever's consuming your API. Mapping to a dedicated DTO decouples those concerns.

**Q4. How do you handle API versioning in a Spring Boot REST API?**

The most common and simplest approach is putting the version right in the URL path, like `/api/v1/orders`. It's explicit, it's easy for consumers to understand, and it's what most public APIs — Stripe, Razorpay, and so on — actually do. There are alternatives, like putting the version in a custom `Accept` header, or as a query parameter, but those add complexity without much practical benefit for most APIs, so URI versioning is usually my default unless there's a specific reason not to.

**Q5. What status code would you return for a successful POST that creates a resource, and what should go along with it?**

201 Created, not 200. And along with it, you should set the `Location` header pointing to the URL of the newly created resource, so the client knows where to find it. In Spring you'd build that using `ServletUriComponentsBuilder` off the current request, then return `ResponseEntity.created(location).body(createdResource)`.

**Q6. How does Spring know to convert a Java object to JSON automatically?**

That's Jackson, which comes in transitively with `spring-boot-starter-web`. When a controller method is annotated `@RestController` — or `@ResponseBody` on `@Controller` — Spring uses an `HttpMessageConverter`, and for JSON that's backed by Jackson's `ObjectMapper`. It serializes your return object's fields into JSON automatically based on getters, no manual mapping needed unless you want to customize field names or formatting with annotations like `@JsonProperty`.

---

## 8. Validation

**Q1. How do you validate incoming request data in a Spring Boot controller?**

I use Bean Validation annotations on the DTO fields — things like `@NotNull`, `@NotBlank`, `@Size`, `@Email`, `@Pattern`. Then the important part — you have to annotate the controller parameter itself with `@Valid` for those constraints to actually get checked. If validation fails, Spring throws a `MethodArgumentNotValidException` before your controller method body even runs, and you'd handle that centrally in an exception handler to return a clean 400 response.

**Q2. What's the difference between `@NotNull`, `@NotEmpty`, and `@NotBlank`?**

`@NotNull` just checks the value isn't null — an empty string would still pass. `@NotEmpty` checks it's not null and not empty, so it works for strings and collections. `@NotBlank` is specifically for strings, and it goes a step further — it checks the string isn't null, isn't empty, and isn't just whitespace. So for something like a name or email field, `@NotBlank` is usually what you actually want, because `"   "` shouldn't count as a valid value.

**Q3. What actually triggers Bean Validation to run — does just putting `@NotNull` on a field do anything by itself?**

No, that's a common trap. Putting `@NotNull` or any other constraint annotation on a DTO field does nothing on its own — it's inert until something actually triggers validation against it. In a Spring MVC controller, that trigger is the `@Valid` annotation on the request parameter. Without `@Valid`, the constraints are just sitting there unused and invalid data sails right through.

**Q4. How would you write a custom validation annotation — say, to validate a PAN number format?**

You'd define a custom annotation, something like `@ValidPan`, meta-annotated with `@Constraint(validatedBy = PanNumberValidator.class)`. Then you implement `ConstraintValidator<ValidPan, String>`, and inside its `isValid` method you write the actual logic — in this case, matching against a regex for the PAN format. Once that's done, you just put `@ValidPan` on any field that needs it, same as you would with a built-in annotation like `@Email`. This is a really common pattern in Indian fintech codebases for things like PAN numbers, IFSC codes, or GSTIN.

**Q5. How do you return a clean, well-formatted error response when validation fails, instead of a raw stack trace?**

You handle `MethodArgumentNotValidException` in a `@RestControllerAdvice` class. Inside that handler, you can pull the individual field errors out of the exception's `BindingResult`, format them into something readable — like "amount: must be positive" — and wrap that into your own consistent error response object with a proper 400 status code. That way every validation failure across your entire API returns the exact same error shape, instead of leaking Spring's internal exception details to the client.

---

## 9. Exception Handling

**Q1. How do you handle exceptions globally in a Spring Boot application?**

I use `@RestControllerAdvice`, which is a class-level annotation that lets you define `@ExceptionHandler` methods that apply across every controller in the application. So instead of wrapping every controller method in try-catch blocks, I define one handler for `ResourceNotFoundException` that returns a 404, one for validation failures that returns a 400, and a catch-all handler for anything unexpected that returns a 500 — logging the real exception internally but never leaking stack trace details back to the client. It keeps error handling centralized and guarantees every error response across the API has the exact same shape.

**Q2. Why is it better to use a centralized exception handler instead of try-catch in each controller method?**

Two big reasons. First, it keeps controllers focused on the happy path — you're not repeating the same error-formatting logic in every single method. Second, and honestly more important, it guarantees consistency. If every developer writes their own try-catch, you'll end up with error responses that look completely different from endpoint to endpoint — different field names, different status code choices for the same kind of error. A centralized handler means the whole API speaks one consistent error language, which matters a lot for anyone consuming it, including monitoring tools.

**Q3. Should custom exceptions in a Spring Boot app be checked or unchecked?**

Unchecked — extending `RuntimeException`. This actually follows how Spring's own exceptions are designed — the whole `DataAccessException` hierarchy is unchecked. The reasoning is that checked exceptions force every method between where the exception is thrown and where it's caught to declare `throws`, which tightly couples every layer in between to that exception even if they don't care about it. Unchecked exceptions let you throw from deep inside a service and catch it cleanly at the top in your global handler, without every layer in between needing to know or care.

**Q4. What's the difference between using `@ResponseStatus` on an exception versus handling it in a `@RestControllerAdvice`?**

`@ResponseStatus` is the lightweight option — you just annotate your custom exception class with, say, `@ResponseStatus(HttpStatus.NOT_FOUND)`, and Spring automatically returns that status whenever the exception is thrown, no handler class needed. It's fine for very small apps. But it only controls the status code — it doesn't let you control the response body shape. For anything with a real API contract, I'd use a `@RestControllerAdvice` handler instead, because I also want control over exactly what the error body looks like — an error code, a message, a timestamp — not just the status.

**Q5. Why can't Spring Security authentication failures be handled through a normal `@RestControllerAdvice`?**

Because of where in the request pipeline they happen. Spring Security's filter chain runs before `DispatcherServlet` even routes the request to a controller — so an authentication failure there never reaches a point where `@RestControllerAdvice` could intercept it, since that mechanism only kicks in for exceptions thrown from inside controller method execution. Instead, Spring Security has its own hooks for this — an `AuthenticationEntryPoint` for handling 401s and an `AccessDeniedHandler` for 403s — and those are what you configure to control what an unauthenticated or unauthorized request actually gets back.

---

## 10. Spring Data JPA

**Q1. What is the N+1 problem, and how would you fix it?**

It happens when you load a list of entities — say a hundred orders — and then, for each order, you access a lazily-loaded relationship, like the order's user. Because that relationship is lazy, each access triggers its own separate query to the database. So instead of one query, you end up running one query for the orders plus a hundred more, one per order, for the users — that's the N+1. There are a few fixes. You can write a JPQL query using `JOIN FETCH` to pull the order and its user in a single query. You can use `@EntityGraph` to declare the same thing more declaratively on a repository method. Or, as a safety net, you can set Hibernate's default batch fetch size, which won't eliminate the extra queries entirely but will batch them together instead of firing one at a time.

**Q2. What's the difference between `FetchType.LAZY` and `FetchType.EAGER`, and what should you use by default?**

Lazy means the related entity isn't loaded from the database until you actually access it — the association is a proxy until then. Eager means it's loaded immediately, as part of the original query. The thing to be careful about is the defaults — `@ManyToOne` and `@OneToOne` default to eager, while `@OneToMany` and `@ManyToMany` default to lazy. My general recommendation is to override everything to be explicitly lazy, and only fetch what you actually need, on purpose, using a join fetch or entity graph — because eager associations silently balloon your queries as your object graph grows, and it's really easy not to notice until performance in production tells you.

**Q3. Why should you never use `EnumType.ORDINAL` for storing enums in the database?**

Because it stores the enum's position — 0, 1, 2 — instead of its actual name. That's fragile in a really dangerous way — if you ever reorder your enum values, or insert a new value somewhere in the middle instead of at the end, every existing row in the database silently gets reinterpreted as a different value. A row that used to mean `PENDING` might now read as `PAID` after a totally unrelated code change, with no error, no warning. `EnumType.STRING` stores the actual name, so it's completely safe against reordering — it costs a little more storage, but that's a trivial trade-off for not silently corrupting your data.

**Q4. What's the difference between optimistic and pessimistic locking, and when would you use each?**

Optimistic locking assumes conflicts are rare — it uses a version column, and every update includes a check that the version hasn't changed since you read the row. If two transactions try to update the same row based on stale data, the second one's update matches zero rows and Spring throws an `OptimisticLockException`, so you know to retry. No locks are held while you're just reading, so it's good for throughput. Pessimistic locking is the opposite approach — it actually locks the row at read time, using something like `SELECT ... FOR UPDATE`, so no other transaction can touch that row until the current one finishes. That's slower under load because things queue up, but it's the right tool when you absolutely cannot allow two transactions to interleave — the classic example being a bank account balance check-and-deduct, where you need to guarantee nobody else can read or modify that balance in between your check and your update.

**Q5. How would you prevent a double-spend or race condition on an account balance?**

I'd use pessimistic locking around the critical section — read the account with a `SELECT ... FOR UPDATE` so the row is locked, check the balance, deduct the amount, and commit, all within one transaction. That guarantees no other transaction can read or modify that same row while this one is in flight, so two concurrent transfer requests can't both pass the balance check based on the same starting balance and end up over-drafting the account. I'd combine this with a proper transaction boundary at the service layer, and for anything asynchronous or retried — like a message being redelivered — I'd also add an idempotency key so a retry doesn't process the same transfer twice.

**Q6. Why would you use an explicit join entity instead of `@ManyToMany` directly?**

Real many-to-many relationships almost always end up needing extra data eventually — like when a student enrolled in a course, or what role someone has in a relationship. A plain `@ManyToMany` mapping only gives you the join table with two foreign keys, nothing else, and adding a column to that join table later means an awkward migration to turn it into a proper entity anyway. So I usually just start with an explicit join entity — like an `Enrollment` class with `@ManyToOne` to both `Student` and `Course`, plus whatever extra fields I need. It's a bit more code upfront, but it avoids a painful refactor later and gives you a proper queryable object for the relationship itself.

**Q7. What does `@Transactional(readOnly = true)` actually do, and why bother adding it?**

It's a hint to Hibernate that this transaction won't do any writes, so Hibernate can skip the dirty-checking overhead it normally does to detect changes on managed entities. In some setups it can also let the connection get routed to a read replica. It's basically free to add — I put it on any service method that's purely reading data — and while the performance gain per call is small, it adds up, and it's also a nice signal in code review that this method genuinely doesn't mutate anything.

**Q8. What's the difference between `findById()` returning `Optional<T>` versus just returning `T` or throwing an exception directly?**

Returning `Optional<T>` forces the caller to explicitly deal with the "not found" case instead of accidentally getting a null and hitting a `NullPointerException` somewhere downstream. Spring Data JPA's `findById()` returns `Optional<T>` by default for exactly this reason. In practice you'd chain `.orElseThrow(() -> new ResourceNotFoundException(...))` right where you call it, so the not-found case turns into a proper domain exception immediately, rather than a null silently propagating through your code.

**Q9. How would you write a repository method that finds orders above a certain amount, placed after a certain date?**

If it's simple enough, Spring Data can derive it from the method name alone — something like `findByAmountGreaterThanAndCreatedAtAfter`. But once the query gets more complex than that, I'd switch to an explicit `@Query` with JPQL, which is clearer to read and easier to maintain than a long derived method name. For anything that needs database-specific features JPQL can't express, there's also the option of a native SQL query using `@Query(nativeQuery = true)` as an escape hatch.

**Q10. What's the purpose of pagination in a Spring Data JPA repository, and how does `Page<T>` help?**

Returning every row of a large table in one response is both slow and a bad experience for the client. `Pageable` lets you request a specific page and size, and Spring Data runs the query with the appropriate `LIMIT`/`OFFSET` under the hood. What's nice about `Page<T>` specifically, versus just a `List<T>`, is that Spring Data also runs a `COUNT(*)` query alongside it, so you get `getTotalElements()` and `getTotalPages()` for free — which the client needs to build proper pagination controls, without you having to write that counting query yourself.

---

## 11. Transactions

**Q1. Where should `@Transactional` be placed — controller, service, or repository — and why?**

On the service layer, not the controller and not the repository. The service layer is where a business operation is actually defined, and a single business operation might touch multiple repositories. If you put `@Transactional` on individual repository methods instead, then something like a fund transfer — which needs to debit one account and credit another — would run as two completely separate transactions. If the app crashed between those two steps, you'd end up with money debited from one account and never credited to the other, which is exactly the kind of thing a transaction boundary is supposed to prevent. So the service method that represents the whole business operation is the right transaction boundary.

**Q2. Why doesn't `@Transactional` work when a method calls another `@Transactional` method on `this`, within the same class?**

This comes down to how `@Transactional` is actually implemented — it works through a dynamically generated proxy that wraps your bean. When some external caller invokes a method on your bean, they're going through that proxy, and that's where the transaction logic actually kicks in. But if a method inside the class calls another method on `this` — so, a plain internal method call — that call never goes through the proxy at all, it's just a direct Java method call on the raw object. So the second method's `@Transactional` annotation is completely ignored in that case; it just runs inside whatever transaction context the first method already established, if any. The usual fix is to move that second method into a separate bean and inject it, so the call does go through a proxy.

**Q3. Does `@Transactional` roll back on every exception by default?**

No, and this trips people up a lot. By default, `@Transactional` only rolls back on unchecked exceptions — `RuntimeException` and `Error`. If your method throws a checked exception, the transaction actually commits anyway unless you've explicitly told it not to. So if you have a method that can throw something like an `IOException`, and a failure there should absolutely not be committed, you need to specify `@Transactional(rollbackFor = Exception.class)` explicitly — otherwise you can end up with a transaction that "succeeded" in the database despite the method having thrown an exception.

**Q4. What's the difference between transaction propagation types like `REQUIRED` and `REQUIRES_NEW`?**

`REQUIRED` is the default — if there's already a transaction in progress when this method is called, it just joins that same transaction. If there isn't one, it starts a new one. `REQUIRES_NEW` always suspends any existing transaction and starts a completely fresh, independent one. A real use case for `REQUIRES_NEW` is audit logging — say your main business transaction fails and rolls back, but you still want a record that the attempt happened and failed. If the audit log write used the same transaction, it would roll back too, along with everything else. Marking it `REQUIRES_NEW` means it commits independently, regardless of what happens to the transaction that called it.

**Q5. What are isolation levels, and do you usually need to change them from the default?**

Isolation levels control how much one transaction can see of another transaction's uncommitted or concurrent changes — things like dirty reads, non-repeatable reads, and phantom reads, in increasing order of strictness, up to `SERIALIZABLE`, which prevents all of them but costs the most in terms of concurrency. Honestly, in most applications I'd leave isolation at the database's default — which for MySQL InnoDB and Postgres is typically read committed — and instead reach for explicit locking, either optimistic with a version column or pessimistic with a `SELECT FOR UPDATE`, when I actually need to prevent a specific race condition. Cranking the isolation level up to serializable everywhere tends to hurt throughput more than it's usually worth.

**Q6. If a transactional method calls an external HTTP API in the middle of the transaction, what could go wrong?**

A few things. First, the database transaction stays open — and therefore any locks it's holding stay held — for the entire duration of that external call, which could be slow or even hang, and that ties up a database connection the whole time. If you're doing this under load, you can exhaust your connection pool pretty fast. Second, if the external call fails partway through, you need to think carefully about whether the transaction should roll back or not — and if the external call actually succeeded on the far end, but your local transaction then rolls back, you can end up in an inconsistent state where the external system thinks something happened that your database now says didn't. Generally I'd try to keep external calls outside the transaction boundary where possible, or use patterns like an outbox table to reliably decouple the two.

---

## 12. Spring Security & JWT

**Q1. Can you explain how the Spring Security filter chain works at a high level?**

Spring Security inserts a chain of servlet filters in front of the `DispatcherServlet`, so every request has to pass through that chain before it ever reaches a controller. Somewhere in that chain, your authentication mechanism runs — for a JWT setup, that's a custom filter that reads the token from the `Authorization` header, validates it, and if it's valid, sets the authenticated user into the security context. Later in the chain, there's an authorization check that decides whether this authenticated user is actually allowed to access the requested URL. If either step fails — no valid token, or valid token but not authorized — the request gets rejected right there in the filter chain, and it never even reaches your controller code.

**Q2. Why do you set `SessionCreationPolicy.STATELESS` for a JWT-based API?**

Because with JWT, the token itself carries everything needed to identify and authenticate the user on every single request — there's no need for the server to remember anything about who's logged in between requests. Setting the session policy to stateless tells Spring Security never to create or rely on an `HttpSession`. That matters a lot for scalability — if you're running multiple instances of your service behind a load balancer, a stateless design means any instance can validate any request independently, with no shared session state that needs to be synchronized or stuck to a specific server.

**Q3. Walk me through how you'd implement JWT authentication in Spring Boot.**

At a high level — a user logs in with email and password, which goes to an authentication endpoint. That endpoint uses Spring Security's `AuthenticationManager` to actually verify the credentials against what's stored, using a `PasswordEncoder` to compare hashes. If that succeeds, I generate a JWT containing the username and maybe their roles as claims, sign it with a secret key, and return it to the client. From then on, the client sends that token in the `Authorization` header on every request. On the server side, I've got a custom filter — extending `OncePerRequestFilter` — that runs early in the security filter chain, pulls the token out of the header, validates its signature and expiry, and if it's valid, builds an authentication object and sets it into the security context for that request.

**Q4. Why extend `OncePerRequestFilter` for a custom JWT filter instead of implementing `Filter` directly?**

`OncePerRequestFilter` guarantees the filter's logic only runs exactly one time per request, even in situations involving internal forwards or error dispatches, where a raw servlet filter chain might otherwise invoke your filter more than once for the same incoming request. Since you really don't want token validation logic running multiple times per request, it's the standard, safe base class to extend for exactly this kind of filter.

**Q5. How would you enforce that a user can only view their own account, not someone else's?**

I'd use method-level security with `@PreAuthorize`, something like `@PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")` on the service method. That way, either the caller is an admin, or the ID they're requesting matches their own authenticated identity — anything else gets denied before the method body even runs. I'd usually pair this with URL-level rules configured in the security filter chain too, as a bit of defense in depth, so the check isn't relying on just one layer.

**Q6. Why use `BCryptPasswordEncoder` instead of something like SHA-256 for hashing passwords?**

Because SHA-256 is designed to be fast, and fast is exactly the wrong property for password hashing — it makes brute-forcing a huge list of hashes cheap for an attacker. BCrypt is deliberately slow and adaptive — you can configure a work factor, and as hardware gets faster over time, you can just increase that factor to keep the hashing appropriately slow. It also automatically generates and stores a unique salt per password, so two users with the same password don't end up with the same hash. Basically, never roll your own password hashing — use a purpose-built algorithm like BCrypt, Argon2, or scrypt.

**Q7. What's the difference between authentication and authorization, and where does each happen in Spring Security?**

Authentication is proving who you are — that's the JWT filter validating the token and establishing who the request is coming from. Authorization is deciding what that authenticated user is actually allowed to do — that happens later in the filter chain, typically through the URL-based rules in `authorizeHttpRequests`, or at the method level with `@PreAuthorize`. It's worth keeping the two separate in your head because they fail differently too — a failed authentication is a 401, meaning "I don't know who you are," while a failed authorization is a 403, meaning "I know who you are, but you're not allowed to do this."

---

## 13. Actuator & Observability

**Q1. What is Spring Boot Actuator, and why is it useful in production?**

Actuator gives you a set of built-in production-monitoring endpoints basically for free, just by adding one dependency. Things like `/actuator/health` for a status check, `/actuator/metrics` for JVM and application metrics, and `/actuator/info` for build/version info. In production this is genuinely important — your load balancer or Kubernetes uses the health endpoint to decide whether to route traffic to an instance, and your metrics endpoint feeds into whatever monitoring system you're using, like Prometheus or Datadog. Without it you'd be building all of that observability tooling yourself from scratch.

**Q2. Which Actuator endpoints are exposed by default, and why is that important to know?**

Only `health` and `info` are exposed by default over HTTP — everything else needs to be explicitly opted in through the `management.endpoints.web.exposure.include` property. That's important because some endpoints are genuinely sensitive — `/actuator/env` can expose environment variables including things that look like secrets, `/actuator/heapdump` can leak whatever's currently sitting in memory. Exposing everything with a wildcard in production without any additional access control is a real security risk, so I'd only include exactly what I need, and put Actuator behind authentication or a separate internal-only management port.

**Q3. How would you add a custom health check — say, checking that a downstream payment gateway is reachable?**

I'd implement `HealthIndicator` in a Spring bean, and inside the `health()` method, try pinging the gateway. If it responds, I return `Health.up()`, and if it throws, I return `Health.down(exception)`. Spring Boot automatically discovers any bean implementing `HealthIndicator` and aggregates it into the overall `/actuator/health` response — so if my custom indicator reports down, the whole application's health check reports down too, which is exactly what a Kubernetes readiness probe needs to know to pull that instance out of rotation.

**Q4. What is Micrometer, and how does it relate to Actuator?**

Micrometer is the metrics facade underneath Actuator's metrics support — it's similar in spirit to how SLF4J is a facade over different logging implementations. You instrument your code once against Micrometer's API — counters, timers, gauges — and then depending on which registry dependency you add, those same metrics get shipped to Prometheus, Datadog, CloudWatch, whatever your monitoring stack actually is, with zero changes to your instrumentation code. Actuator's `/actuator/metrics` and `/actuator/prometheus` endpoints are really just Micrometer's data exposed over HTTP.

**Q5. Can you change a running application's log level without restarting it? How?**

Yes, and it's genuinely useful during an incident. The `/actuator/loggers` endpoint lets you view and change the configured log level for any package or class at runtime, over HTTP — you'd send a POST request with the new level, like DEBUG, for a specific package. It takes effect immediately, no restart or redeploy needed, which means if you're in the middle of debugging a production issue, you can turn up logging on just the relevant package, capture what you need, and turn it back down afterward without any downtime.

---

## 14. AOP

**Q1. What problem does AOP solve, and can you give a real example?**

AOP is for cross-cutting concerns — logic that applies across a lot of unrelated classes but isn't really part of any of their core business logic. Logging, performance timing, auditing, security checks are the classic examples. Without AOP you'd end up copy-pasting the same boilerplate — like a try-catch around timing logic — into every single service method. With AOP you write that logic once as an aspect, and declaratively apply it wherever it's needed using a pointcut expression, without touching the actual business logic classes at all.

**Q2. What's the difference between `@Before`, `@After`, `@AfterReturning`, `@AfterThrowing`, and `@Around`?**

`@Before` runs right before the target method executes. `@After` runs after it finishes, regardless of whether it succeeded or threw — basically like a finally block. `@AfterReturning` only runs on successful completion, and gives you access to the return value. `@AfterThrowing` only runs if the method threw an exception, and gives you access to that exception. `@Around` is the most powerful one — it wraps the entire method call, so you decide when, or even if, the actual method gets invoked, you can inspect and modify arguments before it runs, and modify or even swap out the return value after. Most logging and timing aspects use `@Around` because you need to measure from before the call to after it.

**Q3. How does Spring's AOP actually intercept method calls — what's the mechanism?**

It's proxy-based. When a bean has any aspect applied to it, Spring wraps it in a dynamically generated proxy — either a JDK dynamic proxy if the bean implements an interface, or a CGLIB subclass proxy if it doesn't. Calls coming from outside the bean go through that proxy first, and that's where the aspect's advice actually executes, before or around the real method call. This is exactly the same mechanism `@Transactional` and `@Cacheable` use under the hood — they're really just built-in aspects Spring ships with.

**Q4. Why does proxy-based AOP fail on self-invocation, and how is that related to the `@Transactional` gotcha?**

Because a proxy only intercepts calls coming in from outside the bean. If a method inside a class calls another method on `this` — a plain internal Java call — that call bypasses the proxy entirely, since you're not going through the wrapped object, you're calling directly on the raw instance. So any aspect, or any annotation like `@Transactional` or `@Async` that relies on this exact same proxy mechanism, silently doesn't apply to that internal call. It's the exact same root cause behind the classic `@Transactional` self-invocation gotcha — they're the same underlying limitation showing up in different annotations.

**Q5. Have you built a custom annotation-driven aspect before — how would that work, say for audit logging?**

Yeah, the pattern is — define a custom annotation, like `@AuditLog(action = "ORDER_CREATED")`, that you put on any method you want audited. Then you write an aspect with a pointcut like `@annotation(auditLog)`, using `@AfterReturning`, so it fires whenever a method carrying that annotation completes successfully. Inside the aspect, you'd save an audit entry to a repository, using the action string pulled off the annotation itself. It's a nice pattern because it decouples "this method needs to be audited" from "here's how auditing actually works" — the business logic method just wears the annotation and doesn't know or care how the logging actually happens.

---

## 15. Caching

**Q1. What's the difference between `@Cacheable`, `@CachePut`, and `@CacheEvict`?**

`@Cacheable` checks the cache first — if there's a hit, the method body doesn't even execute, it just returns the cached value directly. `@CachePut` always runs the method, and then updates the cache with whatever it returns — you'd use that for a write operation where you still want the cache refreshed with the latest value afterward. `@CacheEvict` removes an entry from the cache, or all entries if you set `allEntries = true` — you'd use that on a delete, or after something like a bulk import where the whole cache might be stale.

**Q2. Why is the default in-memory cache in Spring Boot not suitable for a multi-instance production deployment?**

Because the default cache is just a `ConcurrentHashMap` sitting in that specific JVM's memory. If you're running three instances of your service behind a load balancer, each one has its own completely separate cache — they're not synchronized in any way. That means one instance could serve stale cached data even after another instance has updated or evicted its own copy, and you're also just wasting memory by caching the same data three times over. For anything beyond a single-instance dev setup, you'd want a shared external cache like Redis, so every instance is reading and writing the same cache.

**Q3. What's a cache stampede, and how would you mitigate it?**

It happens when a popular cache entry expires, and a burst of concurrent requests all miss the cache at exactly the same moment, and all of them go hit the database simultaneously trying to repopulate that same entry — which can actually overload the database right when it's under the most pressure. A few common mitigations — stagger your TTLs with a bit of random jitter so entries don't all expire at the exact same instant, use a distributed lock so only one request actually goes to the database to repopulate the cache while the others wait or get a slightly stale value, or a stale-while-revalidate approach where you keep serving the old value while one background request refreshes it.

**Q4. How would you decide what TTL to set on a cached value?**

It really depends on how often the underlying data changes and how tolerant the use case is of slightly stale data. Something like product catalog data that rarely changes might get a long TTL, ten minutes or more. Something like an account balance that needs to be accurate would either not be cached at all, or cached for a very short window with explicit eviction on write. I'd also think about the cost of a cache miss — if hitting the database for this particular thing is expensive, that pushes toward a longer TTL, as long as the staleness is acceptable for that use case.

---

## 16. Scheduling & Async

**Q1. What's the difference between `fixedRate` and `fixedDelay` in `@Scheduled`?**

`fixedRate` schedules the next execution based on when the previous execution started, at a fixed interval — so if a task normally takes less time than the interval, it just fires regularly, but if a single run happens to take longer than the interval, the next run can actually start immediately after, or executions can start queuing up back to back. `fixedDelay` instead waits for the fixed interval to pass after the previous execution actually finishes, before starting the next one. If a task's duration is unpredictable and you don't want overlapping runs, `fixedDelay` is the safer default.

**Q2. By default, do all your `@Scheduled` tasks run in parallel?**

No, actually the opposite — by default Spring runs every `@Scheduled` method on a single shared thread. So if one scheduled task happens to take ten minutes, it blocks every other scheduled task in the entire application from running during that window. That's a real gotcha people hit in production. The fix is to configure a proper thread pool for the scheduler, using `SchedulingConfigurer`, so multiple scheduled tasks can actually run concurrently instead of queuing behind each other.

**Q3. How does `@Async` work, and what's a real use case for it?**

`@Async` lets a method run on a separate thread from a configured thread pool, instead of blocking the calling thread. A good real-world example is sending a confirmation email after an order is placed — the order itself doesn't need to wait for the email to actually send, so you mark the email-sending method `@Async`, and the calling code moves on immediately while the email goes out in the background. It's proxy-based, same as `@Transactional`, so it has the exact same self-invocation limitation — calling an `@Async` method on `this` from inside the same class won't actually run asynchronously.

**Q4. What's a common mistake people make with `@Async` methods that return `void`?**

Exceptions thrown inside an async void method get silently swallowed by default — there's no caller waiting around to catch anything, since the calling thread already moved on. That means a failure in a background task, like a failed email send, can just disappear without a trace unless you've explicitly configured an `AsyncUncaughtExceptionHandler` to catch and log it. A safer pattern is to have the async method return a `CompletableFuture` instead of void, so if the caller does eventually check or join that future, the exception actually surfaces there.

**Q5. Write a cron expression for running a job every day at 2 AM.**

`0 0 2 * * *`. Breaking that down, it's second, minute, hour, day-of-month, month, day-of-week — so zero seconds, zero minutes, two AM, every day of the month, every month, every day of the week. You'd put that as the value of the `cron` attribute on `@Scheduled`.

---

## 17. Testing

**Q1. How do you approach testing in a Spring Boot application — what mix of test types do you use?**

I think about it as a pyramid. At the base, the largest number of tests should be plain unit tests using JUnit and Mockito, with no Spring context involved at all — just constructing the service directly with mocked dependencies, which is fast. Above that, a smaller set of slice tests — things like `@WebMvcTest` for just the controller layer, or `@DataJpaTest` for just the repository layer — these load a partial Spring context, so they're a bit slower but still reasonably fast. At the top, a small number of full integration tests using `@SpringBootTest`, which boot the entire application context and are the slowest, but give you the most realistic end-to-end confidence. The key idea is the further up the pyramid you go, the fewer tests you want, because Spring context startup time adds up fast across a large test suite.

**Q2. What's the difference between `@Mock` and `@MockBean`?**

`@Mock` is plain Mockito — it creates a mock object with no Spring involved at all, which is what you'd use in a pure unit test alongside `@InjectMocks`. `@MockBean` is Spring Boot's testing annotation — it replaces an actual bean in the Spring application context with a mock, which only makes sense in tests that actually load some Spring context, like `@WebMvcTest` or `@SpringBootTest`. So the rule of thumb is, if there's no Spring context in the test at all, use `@Mock`; if there is a context and you want to swap out one specific bean within it, use `@MockBean`.

**Q3. What does `@WebMvcTest` actually load, and why would you use it instead of `@SpringBootTest`?**

`@WebMvcTest` loads only the web layer — controllers, `@ControllerAdvice` classes, filters, and the Jackson configuration for serialization — it deliberately does not load your service or repository beans, or connect to a real database. You'd typically mock out the service layer with `@MockBean` and use `MockMvc` to fire simulated HTTP requests at your controller. The advantage over `@SpringBootTest` is speed — since it's only loading a slice of the application context instead of the whole thing, it starts much faster, which matters a lot once you've got hundreds of these tests running in CI.

**Q4. What is Testcontainers, and why would you use it instead of an in-memory database like H2 for integration tests?**

Testcontainers lets you spin up a real database — like an actual MySQL or Postgres instance — inside a Docker container, just for the duration of a test run, and then tear it down afterward. The reason this matters is that H2, even in MySQL-compatibility mode, doesn't perfectly replicate every dialect-specific SQL behavior that MySQL or Postgres actually have — so a test passing against H2 doesn't fully guarantee it'll behave the same way against the real production database engine. Testcontainers closes that gap by testing against the real thing, and it's become the standard, current best practice for integration testing in the Spring ecosystem.

**Q5. How would you test a controller's validation logic without hitting a real database?**

I'd use `@WebMvcTest` scoped to that specific controller, with the service layer mocked out using `@MockBean`. Then I'd use `MockMvc` to send a request with intentionally invalid JSON — like a missing required field — and assert that the response status is 400. Since `@WebMvcTest` loads the actual validation and exception-handling infrastructure but nothing else, this verifies the validation and error-response behavior end-to-end at the HTTP layer, without needing a database or any of the business logic to actually run.

**Q6. Why is it important that your service classes use constructor injection when it comes to testing?**

Because it means you can construct the service directly in a plain unit test — `new OrderService(mockRepository, mockGateway)` — without needing Spring involved at all. If a class used field injection instead, you'd typically need Mockito's `@InjectMocks` to use reflection to force values into those private fields, which works, but it's a bit more indirect. Constructor injection just makes the dependency list explicit and the object trivially constructible in isolation, which is really the whole point of designing for testability in the first place.

---

## 18. Swagger/OpenAPI

**Q1. How do you document a REST API built with Spring Boot?**

I use springdoc-openapi — you add one dependency, `springdoc-openapi-starter-webmvc-ui`, and with basically zero configuration it reflects over your `@RestController` classes and generates an OpenAPI spec automatically, plus it exposes a Swagger UI page where you can browse and actually test every endpoint interactively. For a real API I'd also enrich it a bit further with annotations like `@Operation` and `@Schema` to add human-readable descriptions and examples on top of what's auto-generated, so the docs are actually useful to someone integrating against the API, not just a bare list of endpoints.

**Q2. What's the benefit of Swagger UI over just writing documentation in a wiki or README?**

The big one is that it's generated directly from your actual code, so it can't drift out of sync with reality the way a manually maintained doc can — if you add a field or change an endpoint, the docs update automatically the next time the app starts. It's also interactive — you can literally send test requests from the browser and see real responses, which is a lot faster for someone integrating against your API than just reading a static description and then writing test code themselves. For a JWT-secured API, you can also configure an "Authorize" button so a token can be pasted once and gets attached to every subsequent test call automatically.

**Q3. How would you add JWT authentication support to your Swagger UI so you can test protected endpoints?**

You configure a security scheme in an `OpenAPI` bean — defining a bearer-token, HTTP-type scheme with the format set to JWT — and reference it as a global security requirement. Once that's wired up, Swagger UI shows an "Authorize" button where you paste in a valid token, and from then on every test request you send through the UI automatically includes that token in the `Authorization` header, so you can actually exercise secured endpoints directly from the docs page.

---

## 19. File Upload & Download

**Q1. How do you handle file uploads in a Spring Boot REST API?**

The controller method takes a `MultipartFile` parameter, mapped from a multipart form data request. Before actually storing it, I'd validate a few things — check the content type against an allowlist, check the file size doesn't exceed some limit, and I'd never trust or reuse the original filename the client sent directly — instead I generate a random filename, usually a UUID, and just keep the original file extension. That avoids collisions, accidental overwrites, and any risk from a maliciously crafted filename.

**Q2. What security concerns come up specifically with file uploads?**

A few real ones. Path traversal — if you naively use the client-supplied filename to build a file path, someone could send a filename containing `..` sequences trying to write outside the intended upload directory, so you need to guard against that explicitly. Content-type spoofing — the client-declared content type can be faked, so for anything security-sensitive you'd want to verify the actual file content, not just trust the declared MIME type. And unrestricted file size — without an explicit limit configured, someone could upload something huge and exhaust disk space or memory, so you'd cap `max-file-size` and `max-request-size` in your configuration.

**Q3. Why would you avoid storing uploaded files on the local filesystem in a production system?**

Because local disk storage doesn't survive a container restart, and it doesn't work cleanly at all if you're running multiple instances behind a load balancer — a file uploaded to instance A wouldn't be visible from instance B. For any real production system, especially something with compliance or retention requirements like banking, you'd want to store files in object storage instead — something like AWS S3 — which is durable, shared across all instances, and typically has built-in versioning and lifecycle policies. The nice part is if you've abstracted file storage behind a service interface, swapping the local-disk implementation for an S3-backed one is a contained change that doesn't touch your controllers at all.

---

## 20. Microservices

**Q1. What's the difference between `RestTemplate`, `WebClient`, and Feign for calling another service?**

`RestTemplate` is the classic, synchronous, blocking HTTP client — it's been around the longest, and you'll still see it in a lot of existing codebases, but it's officially in maintenance mode now, Spring isn't actively adding new features to it. `WebClient` is the modern replacement — it's reactive and non-blocking by design, though you can still call `.block()` on it to use it synchronously if you don't need the reactive style. Feign is a bit different — it's a declarative client, you just write an interface with the endpoint mappings on it, and Spring generates the actual implementation for you at runtime, so calling another service ends up looking like calling a local method. In practice, for a greenfield project I'd lean toward WebClient or Feign over RestTemplate.

**Q2. What is a circuit breaker, and why do you actually need one in a microservices setup?**

Without one, if a downstream service — say an inventory service — starts responding slowly or timing out, every request into your service that depends on it also starts hanging, waiting on those slow calls. Under load, that can exhaust your own thread pool, and now your otherwise perfectly healthy service is failing too, purely because it's waiting on a struggling dependency — that's what people call a cascading failure. A circuit breaker watches the failure rate of calls to that dependency, and once it crosses a threshold, it "opens" — meaning it stops even attempting the call for a cool-down period and immediately returns a fallback instead. After that cool-down, it "half-opens" to test a few real calls and see if the downstream service has recovered, before fully closing again. It basically fails fast instead of failing slow, and that protects the rest of your system.

**Q3. How would you make an API endpoint idempotent, so retrying it doesn't create a duplicate side effect?**

The standard pattern is to require an idempotency key on the request — the client generates a unique key, usually a UUID, and sends it on every attempt, including retries of the exact same logical operation. On the server, before processing, you check whether you've already seen that key — if you have, you just return the same result you returned the first time, without reprocessing. If you haven't, you process it and store the key alongside the result. The important detail is that check-then-process logic on its own still has a race window under real concurrency, so you'd back it with a unique database constraint on the idempotency key column — that way even if two requests with the same key somehow arrive at nearly the same instant, the database itself guarantees only one of them actually succeeds in creating the record, and you catch that constraint violation to return the existing result for the other one.

**Q4. Why is idempotency especially important when you're using something like Kafka?**

Because message brokers generally guarantee at-least-once delivery, not exactly-once — meaning a consumer might receive and process the same message more than once, for example if the consumer crashed right after processing a message but before it could acknowledge that offset back to Kafka, causing the message to be redelivered on restart. If the consumer's processing logic isn't idempotent, that redelivery could mean double-charging a payment, or double-crediting an account. So the idempotency-key pattern isn't just for HTTP retries, it applies just as much to event consumers — you'd track which event IDs have already been processed and skip reprocessing ones you've already seen.

**Q5. What's the role of an API gateway in a microservices architecture?**

It's the single entry point that external clients actually talk to, and it's responsible for routing each incoming request to the correct downstream service. Beyond just routing, it's also the natural place to centralize cross-cutting concerns you don't want every individual service reimplementing — things like authentication, rate limiting, request logging, and sometimes response caching. Spring Cloud Gateway is the common choice in the Spring ecosystem for building this.

**Q6. What's the difference between service discovery and a config server in a microservices setup?**

Service discovery, using something like Eureka or Consul, solves the problem of services finding each other — instead of hardcoding another service's host and port, a service registers itself by name at startup, and other services look it up by that logical name, which matters a lot when instances are scaling up and down dynamically and their actual network addresses keep changing. A config server, like Spring Cloud Config, solves a different problem — centralizing configuration for every microservice in one place, typically backed by a Git repo, so instead of each service having its own scattered `application.yml`, they all pull their configuration from one central source, which makes managing config across dozens of services actually manageable.

---

## 21. Deployment & Production

**Q1. How would you containerize a Spring Boot application, and why use a multi-stage Docker build?**

I'd write a Dockerfile with two stages — a build stage that has the full JDK and Maven, which compiles and packages the app into a JAR, and a second, much leaner runtime stage that just has a JRE, and copies over only the final built JAR from the first stage. The point of doing it this way is that the final image doesn't carry around the entire build toolchain — Maven, the full JDK, source code — none of that needs to exist in the image that actually runs in production, so you end up with a smaller image and a smaller attack surface. It also plays nicely with Docker's layer caching, so dependency resolution only gets rerun when your `pom.xml` actually changes, not on every single code change, which speeds up builds a lot.

**Q2. What does "graceful shutdown" mean for a Spring Boot app, and why does it matter?**

Without it, when the process receives a termination signal — which is what happens routinely during a rolling deployment or when Kubernetes scales an instance down — the JVM can shut down more or less immediately, and any request that happened to be in-flight at that exact moment gets abruptly cut off, the client sees a connection reset. With graceful shutdown enabled, Spring Boot instead stops accepting new requests but keeps the server alive long enough to let in-flight requests actually finish, up to a configured timeout. It matters a lot in a system that deploys frequently, because otherwise every deployment causes a small but real number of failed requests for users who happened to be mid-request at the wrong moment.

**Q3. What's the difference between a liveness probe and a readiness probe in Kubernetes?**

A liveness probe answers "is this instance stuck or deadlocked and does it need to be killed and restarted" — if it fails, Kubernetes kills the pod and starts a fresh one. A readiness probe answers a different question — "is this instance currently able to serve traffic right now" — an instance could be perfectly alive but not ready yet, say it's still warming up a cache on startup, or its database connection pool is temporarily exhausted. If a readiness probe fails, Kubernetes just stops routing traffic to that pod without killing it, and starts routing to it again once it reports ready. Spring Boot Actuator can expose these as two separate endpoints, `/actuator/health/liveness` and `/actuator/health/readiness`, so you can wire each one to the correct Kubernetes probe.

**Q4. Why should secrets and environment-specific configuration never be baked into a Docker image?**

Because a Docker image is meant to be the same immutable artifact across every environment — dev, staging, production — and baking environment-specific values or secrets into it defeats that. If a database password ends up inside the image itself, then anyone with access to that image, which might be sitting in a shared container registry, effectively has that secret. Instead, you build one image and inject environment-specific configuration at runtime, through environment variables, a Kubernetes ConfigMap for non-sensitive values, and a Kubernetes Secret — or ideally an external secrets manager — for anything sensitive. That way the exact same image is genuinely portable across environments.

**Q5. What would you check before deploying a Spring Boot service to production for the first time?**

I'd go through a checklist — schema managed by a real migration tool like Flyway with `ddl-auto` set to validate, not update or create. Secrets pulled from environment variables or a secrets manager, never committed to the repo. Actuator endpoints restricted to only what's needed and protected behind auth. Connection pool sizes actually tuned for expected load rather than left at defaults. Structured logging with request or correlation IDs, so you can trace a request across logs. Timeouts and circuit breakers on any outbound calls to other services. Graceful shutdown enabled. And a global exception handler that's guaranteed to never leak stack traces or internal details back to a client. Most production incidents I've seen trace back to one of these being skipped, not to some exotic bug.

**Q6. Why is it recommended to set the JVM's minimum and maximum heap size to the same value when running in a container?**

Because if you leave them different, the JVM can dynamically resize the heap at runtime as memory pressure changes, which means your container's actual memory usage becomes somewhat unpredictable over time. In a container with a hard memory limit set by the orchestrator, an unexpected heap resize event pushing you over that limit gets the container killed by the OOM killer — often at the worst possible moment, under load. Setting `-Xms` and `-Xmx` to the same value gives you a fixed, predictable memory footprint from the moment the JVM starts, which makes capacity planning and container memory limits much more reliable.

---

*End of Q&A set. If you're prepping out loud, cover the answer, read only the question, and try to say the answer in your own words at roughly this same conversational pace before checking yourself against what's written here — reciting from memory under a bit of self-imposed pressure is much closer to what an actual interview feels like than silently reading.*
