# System Design — Complete Beginner-to-Advanced Notes
### For: Ananth (Java / Spring Boot backend dev, Wipro banking domain)

> How to use this file: Read top to bottom. Each topic has — **What it is (in plain English)**, **Why it exists (the problem it solves)**, **How it works**, **Real-world analogy**, **Java/Spring Boot code example**, **Real systems that use it**, and **Common interview questions**. Don't skip the analogies — they're what make these concepts stick.

---

## Table of Contents

**BASIC**
1. [Load Balancer](#1-load-balancer)
2. [API Gateway](#2-api-gateway)
3. [Content Delivery Network (CDN)](#3-content-delivery-network-cdn)
4. [Database Fundamentals](#4-database-fundamentals)

**INTERMEDIATE**
5. [Caching](#5-caching)
6. [Rate Limiting](#6-rate-limiting)
7. [Database Sharding](#7-database-sharding)
8. [Replication](#8-replication)

**ADVANCED**
9. [CAP Theorem](#9-cap-theorem)
10. [Eventual Consistency](#10-eventual-consistency)
11. [Distributed Transactions](#11-distributed-transactions)
12. [Consensus Algorithms](#12-consensus-algorithms)
13. [Service Discovery](#13-service-discovery)
14. [Message Queues](#14-message-queues)
15. [System Observability](#15-system-observability)

---

# 1. Load Balancer

## What it is
A Load Balancer (LB) is a component that sits **in front of your servers** and distributes incoming traffic across multiple servers instead of letting one server take all the load.

## Why it exists
Imagine you have one server handling your banking app. It can handle, say, 1,000 requests/second. On salary day, you get 10,000 requests/second. That one server crashes. Now imagine 5 identical servers behind a load balancer — each only needs to handle ~2,000 req/sec. Nobody crashes.

It also solves a second problem: **if one server dies, traffic should not go to it anymore.** The load balancer detects dead servers (via health checks) and stops routing to them.

## How it works
1. Client sends request to a single address (e.g., `api.yourbank.com`).
2. That address actually points to the Load Balancer, not to any single app server.
3. The LB picks one of the healthy backend servers using an algorithm and forwards the request.
4. The backend server processes and responds — usually back through the LB.

### Common algorithms
| Algorithm | How it picks a server | When to use |
|---|---|---|
| Round Robin | Server 1, 2, 3, 1, 2, 3... in order | Servers are equally powerful |
| Least Connections | Server with fewest active connections | Requests vary a lot in duration |
| IP Hash | Same client IP always → same server | Need "sticky sessions" |
| Weighted Round Robin | More powerful servers get more requests | Servers have different capacities |

### Layer 4 vs Layer 7 Load Balancers
- **L4 (Transport layer)**: Balances based on IP + port only. Very fast, doesn't look at HTTP content. Example: AWS NLB.
- **L7 (Application layer)**: Understands HTTP — can route `/orders` to one service and `/users` to another. Example: NGINX, AWS ALB.

## Real-world analogy
Think of a restaurant with 5 waiters and 1 host at the door. The host (load balancer) looks at which waiter is least busy and sends the next customer there, instead of every customer walking randomly to a single overwhelmed waiter.

## Java / Spring Boot example
In real systems, you don't code the load balancer yourself (NGINX/AWS ELB do it) — but if you're doing **client-side load balancing** between microservices, Spring Cloud LoadBalancer is common:

```java
// build.gradle / pom.xml: add spring-cloud-starter-loadbalancer

@Configuration
public class LoadBalancerConfig {

    @Bean
    @LoadBalanced   // Tells RestTemplate to resolve "order-service" via service discovery + LB
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class OrderClient {

    private final RestTemplate restTemplate;

    public OrderClient(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    public Order getOrder(String orderId) {
        // "order-service" is a logical name; Spring Cloud LB picks
        // one healthy instance out of many registered instances
        return restTemplate.getForObject(
            "http://order-service/orders/" + orderId, Order.class);
    }
}
```

A minimal NGINX config doing L7 load balancing (this is what actually sits in front of your Spring Boot apps in production):

```nginx
upstream backend_servers {
    least_conn;                      # algorithm
    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
    server 10.0.0.3:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend_servers;
    }
}
```

## Real systems that use it
NGINX, HAProxy, AWS Elastic Load Balancer (ALB/NLB), Google Cloud Load Balancer, Envoy.

## Interview questions
- Difference between L4 and L7 load balancing?
- How does a load balancer detect an unhealthy server?
- What is a "sticky session" and when do you need it?
- Round robin vs least connections — when would round robin be a bad choice?

---

# 2. API Gateway

## What it is
An API Gateway is a **single entry point** that sits in front of all your backend microservices. Instead of clients calling 10 different microservices directly, they call the gateway, and the gateway routes the request to the right service.

## Why it exists
Say your banking app has: `auth-service`, `account-service`, `transaction-service`, `notification-service`. Without a gateway, your mobile app needs to know the address of all 4, handle auth for each separately, and handle rate limiting for each separately. That's a mess, and it exposes your internal architecture to the outside world (bad for security).

The API Gateway centralizes:
- **Routing** — `/accounts/**` → account-service, `/transactions/**` → transaction-service
- **Authentication/Authorization** — verify JWT once, at the edge
- **Rate limiting** — stop abuse before it even reaches your services
- **Request/response transformation** — e.g., combine 2 backend calls into 1 client response
- **Logging/monitoring** — one place to see all traffic

## How it works
```
Mobile App → API Gateway → auth-service (checks JWT)
                          → account-service (if authorized)
                          → transaction-service
```
The client only ever talks to the gateway. It has zero knowledge of how many microservices exist behind it.

## Real-world analogy
Think of a hotel reception desk. Guests don't wander into housekeeping, the kitchen, or maintenance directly — they go to reception, and reception directs the request to the right department. Reception also checks your ID (auth) before letting you do anything.

## Java / Spring Boot example
Spring Cloud Gateway is the standard tool for this in the Java ecosystem:

```java
@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRoutes(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("account-service-route", r -> r
                .path("/accounts/**")
                .filters(f -> f
                    .stripPrefix(0)
                    .addRequestHeader("X-Gateway-Source", "main-gateway"))
                .uri("lb://ACCOUNT-SERVICE"))   // lb:// = resolved via service discovery

            .route("transaction-service-route", r -> r
                .path("/transactions/**")
                .filters(f -> f.circuitBreaker(c -> c
                    .setName("txnCircuitBreaker")
                    .setFallbackUri("forward:/fallback/transactions")))
                .uri("lb://TRANSACTION-SERVICE"))
            .build();
    }
}
```

Adding a global JWT filter so every route is authenticated centrally:

```java
@Component
public class AuthenticationFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().getFirst("Authorization");

        if (token == null || !isValidJwt(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);   // pass on to the actual microservice
    }

    private boolean isValidJwt(String token) {
        // JWT signature + expiry validation logic here
        return true;
    }

    @Override
    public int getOrder() {
        return -1; // run before other filters
    }
}
```

## Real systems that use it
Netflix Zuul (older), Spring Cloud Gateway, Kong, AWS API Gateway, Apigee.

## Interview questions
- What problems does an API Gateway solve that a Load Balancer doesn't?
- What's the downside of an API Gateway? (Single point of failure, added latency, can become a bottleneck if not scaled)
- How would you implement rate limiting at the gateway level?
- Difference between API Gateway and reverse proxy?

---

# 3. Content Delivery Network (CDN)

## What it is
A CDN is a network of servers spread across the world that cache and serve **static content** (images, CSS, JS, videos) from a location physically close to the user.

## Why it exists
If your servers are in Mumbai and a user in New York requests your homepage image, that request has to travel ~13,000 km round trip. That's slow. A CDN caches a copy of that image on a server in New York (or nearby), so the user gets it in milliseconds instead of seconds.

This also **protects your origin server** — instead of every user hitting your actual backend for a logo image, 95% of them get served by the CDN edge server, and your backend only deals with actual dynamic requests.

## How it works
1. First user in a region requests `logo.png`. CDN edge server doesn't have it cached — it's a "cache miss." It fetches from your origin server, serves it to the user, AND stores a copy locally.
2. Next user in that same region requests `logo.png` — CDN edge server already has it ("cache hit") — serves instantly without touching your origin server at all.
3. Content usually has a **TTL (Time To Live)** — after which the CDN re-fetches from origin to keep it fresh.

## Real-world analogy
Think of a chain of bakeries. Instead of every customer in every city ordering bread directly from one central bakery in Mumbai and waiting for shipping, each city has its own local branch that keeps popular items pre-stocked. Only when something isn't in stock does the local branch place an order to the central bakery.

## Java / Spring Boot example
You don't build a CDN yourself, but you configure your app to work well with one — mainly via HTTP cache headers:

```java
@RestController
@RequestMapping("/static")
public class StaticContentController {

    @GetMapping("/logo.png")
    public ResponseEntity<Resource> getLogo() {
        Resource logo = new ClassPathResource("static/logo.png");

        return ResponseEntity.ok()
            .cacheControl(CacheControl.maxAge(30, TimeUnit.DAYS).cachePublic())
            .eTag("\"logo-v3\"")   // helps CDN/browser know if content changed
            .body(logo);
    }
}
```

Typical Spring Boot static resource caching config (so CDN/browser caches assets aggressively):

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/assets/**")
            .addResourceLocations("classpath:/static/assets/")
            .setCacheControl(CacheControl.maxAge(365, TimeUnit.DAYS).cachePublic());
    }
}
```

In practice, for a banking app, you'd put your React/Angular frontend's JS/CSS bundles and images behind a CDN like Cloudflare or AWS CloudFront, while API calls (dynamic, per-user data) go directly to your backend — **never cache sensitive per-user banking data on a CDN.**

## Real systems that use it
Cloudflare, Akamai, AWS CloudFront, Fastly.

## Interview questions
- What kind of content should NOT be served via CDN? (personal/sensitive dynamic data)
- What is cache invalidation and why is it hard? ("There are only two hard things in Computer Science: cache invalidation and naming things.")
- Push CDN vs Pull CDN — what's the difference?
- How does a CDN help during a DDoS attack?

---

# 4. Database Fundamentals

## What it is
The foundational layer that stores your application's structured or unstructured data — SQL (relational) or NoSQL (non-relational).

## SQL (Relational) Databases
Data is stored in **tables** with fixed schemas (rows and columns), and relationships between tables are enforced via foreign keys. Examples: MySQL, PostgreSQL, Oracle.

**Key properties: ACID**
- **Atomicity** — a transaction either fully happens or doesn't happen at all
- **Consistency** — data always moves from one valid state to another valid state
- **Isolation** — concurrent transactions don't interfere with each other
- **Durability** — once committed, data survives crashes

This is exactly why banking systems use SQL — you cannot have a transaction "half happen" (money debited from one account but not credited to another).

## NoSQL (Non-relational) Databases
No fixed schema, built for scale and flexibility. Categories:
- **Document stores** (MongoDB) — JSON-like documents
- **Key-Value stores** (Redis, DynamoDB) — simple key → value lookups, extremely fast
- **Column-family stores** (Cassandra) — optimized for huge write volumes
- **Graph databases** (Neo4j) — optimized for relationship-heavy data (social networks, fraud detection graphs)

## Real-world analogy
SQL is like a well-organized filing cabinet — every folder (table) has the exact same labeled sections (columns), and folders can reference each other ("see Folder B, Row 12"). NoSQL is like sticky notes in a box — flexible, fast to add, but you can't guarantee every note has the same fields.

## Java / Spring Boot example — SQL with JPA

```java
@Entity
@Table(name = "accounts")
public class Account {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String accountNumber;

    @Column(nullable = false)
    private BigDecimal balance;

    // getters/setters omitted
}

public interface AccountRepository extends JpaRepository<Account, Long> {
    Optional<Account> findByAccountNumber(String accountNumber);
}

@Service
public class TransferService {

    private final AccountRepository accountRepository;

    public TransferService(AccountRepository accountRepository) {
        this.accountRepository = accountRepository;
    }

    @Transactional  // ensures ACID: both updates succeed, or neither does
    public void transfer(String fromAcc, String toAcc, BigDecimal amount) {
        Account from = accountRepository.findByAccountNumber(fromAcc)
            .orElseThrow(() -> new AccountNotFoundException(fromAcc));
        Account to = accountRepository.findByAccountNumber(toAcc)
            .orElseThrow(() -> new AccountNotFoundException(toAcc));

        if (from.getBalance().compareTo(amount) < 0) {
            throw new InsufficientFundsException(fromAcc);
        }

        from.setBalance(from.getBalance().subtract(amount));
        to.setBalance(to.getBalance().add(amount));

        accountRepository.save(from);
        accountRepository.save(to);
        // If an exception is thrown anywhere above, @Transactional
        // rolls back BOTH updates — this is Atomicity in action.
    }
}
```

## Java / Spring Boot example — NoSQL with MongoDB

```java
@Document(collection = "notifications")
public class Notification {
    @Id
    private String id;
    private String userId;
    private String message;
    private Instant createdAt;
    // No fixed schema needed — you could add a "priority" field
    // to only SOME documents without altering every existing row
}

public interface NotificationRepository extends MongoRepository<Notification, String> {
    List<Notification> findByUserId(String userId);
}
```

## When to use which
| Use SQL when... | Use NoSQL when... |
|---|---|
| Data has clear relationships (accounts, transactions) | Data is flexible/evolving fast |
| You need strong consistency (banking, payments) | You need to scale writes horizontally (logs, events) |
| Complex queries/joins are common | Simple key-based lookups dominate |

## Interview questions
- Explain ACID properties with a banking example.
- When would you choose MongoDB over PostgreSQL?
- What is normalization and why does it matter?
- What's the difference between a primary key and a foreign key?

---

# 5. Caching

## What it is
Storing frequently accessed data **in memory** (fast) instead of fetching it from the database (slow) every single time.

## Why it exists
Database reads are relatively slow (disk I/O, network hop, query planning). If the same data — like a user's profile, or "today's exchange rate" — is requested 10,000 times a minute, hitting the DB every time is wasteful. Cache it once, and serve the next 9,999 requests from RAM.

## How it works
1. Request comes in for `getUser(123)`.
2. App checks cache first: "Do I have user 123 cached?"
3. **Cache hit** → return immediately from memory (microseconds).
4. **Cache miss** → fetch from DB, store in cache, then return.
5. Cached data has a **TTL (expiry)** so it doesn't go stale forever.

### Cache eviction policies (what to remove when cache is full)
- **LRU (Least Recently Used)** — remove the item not accessed in the longest time
- **LFU (Least Frequently Used)** — remove the item accessed the fewest times
- **FIFO** — remove the oldest item, regardless of usage

### Caching patterns
- **Cache-aside (lazy loading)** — app checks cache, loads from DB on miss, writes to cache (shown above — most common pattern)
- **Write-through** — every write goes to cache AND DB at the same time
- **Write-behind (write-back)** — write to cache immediately, DB is updated asynchronously later (faster writes, riskier — can lose data on crash)

## Real-world analogy
Think of your kitchen fridge vs the supermarket. Every time you need milk, you don't drive to the supermarket (database) — you check the fridge (cache) first. Only when the fridge is empty do you make the trip.

## Java / Spring Boot example (Redis)

```java
// application.yml
// spring:
//   redis:
//     host: localhost
//     port: 6379
//   cache:
//     type: redis

@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))     // TTL = 10 minutes
            .disableCachingNullValues();
    }
}

@Service
public class AccountService {

    private final AccountRepository accountRepository;

    public AccountService(AccountRepository accountRepository) {
        this.accountRepository = accountRepository;
    }

    @Cacheable(value = "accounts", key = "#accountNumber")
    public Account getAccount(String accountNumber) {
        System.out.println("Hitting DB for: " + accountNumber); // only prints on cache miss
        return accountRepository.findByAccountNumber(accountNumber)
            .orElseThrow(() -> new AccountNotFoundException(accountNumber));
    }

    @CachePut(value = "accounts", key = "#account.accountNumber")
    public Account updateAccount(Account account) {
        return accountRepository.save(account); // updates cache AND DB
    }

    @CacheEvict(value = "accounts", key = "#accountNumber")
    public void deleteAccount(String accountNumber) {
        accountRepository.deleteByAccountNumber(accountNumber); // removes stale cache entry
    }
}
```

Manual Redis usage without Spring's annotation abstraction (useful to understand what's really happening underneath):

```java
@Service
public class ExchangeRateService {

    private final StringRedisTemplate redisTemplate;
    private final ExchangeRateClient externalApiClient;

    public ExchangeRateService(StringRedisTemplate redisTemplate,
                                ExchangeRateClient externalApiClient) {
        this.redisTemplate = redisTemplate;
        this.externalApiClient = externalApiClient;
    }

    public BigDecimal getRate(String currencyPair) {
        String cacheKey = "rate:" + currencyPair;
        String cachedValue = redisTemplate.opsForValue().get(cacheKey);

        if (cachedValue != null) {
            return new BigDecimal(cachedValue); // cache hit
        }

        BigDecimal freshRate = externalApiClient.fetchRate(currencyPair); // cache miss
        redisTemplate.opsForValue().set(cacheKey, freshRate.toString(), Duration.ofMinutes(5));
        return freshRate;
    }
}
```

## Real systems that use it
Redis, Memcached, Caffeine (in-JVM cache), Ehcache.

## Interview questions
- Cache-aside vs write-through — trade-offs?
- How do you avoid a "cache stampede" (many requests missing cache at the exact same time)?
- What is cache invalidation, and how do you keep cache in sync with the DB?
- LRU vs LFU — give an example where LFU is clearly better.

---

# 6. Rate Limiting

## What it is
Restricting how many requests a client (user/IP/API key) can make in a given time window, to protect your system from abuse or overload.

## Why it exists
Without rate limiting, one buggy client (or malicious attacker) could send 100,000 requests/second and take your entire system down for everyone else — this is basically a self-inflicted DDoS. In banking, it also stops brute-force login attempts (someone trying 10,000 passwords per minute).

## How it works — common algorithms

### 1. Fixed Window
Count requests in a fixed time block (e.g., 100 requests per minute, resetting every minute). Simple, but has a burst problem at window boundaries.

### 2. Sliding Window Log
Keep a timestamp log of every request; only count requests within the last N seconds from "now." Accurate but memory-heavy.

### 3. Token Bucket (most common in practice)
A bucket holds tokens (e.g., max 10). Every request consumes 1 token. Tokens refill at a fixed rate (e.g., 1 token/second). If bucket is empty, request is rejected. Allows short bursts while enforcing a long-term average rate.

### 4. Leaky Bucket
Requests enter a queue (bucket) and are processed ("leak out") at a fixed rate — smooths out bursts completely, unlike token bucket.

## Real-world analogy
Token bucket = an amusement park ride that only lets in 1 person every 10 seconds, but if nobody's been in line for a while, a few people who arrive together can get on right away (because tokens accumulated) — until the "saved up" tokens run out.

## Java / Spring Boot example (Token Bucket with Bucket4j)

```java
// build.gradle: implementation 'com.bucket4j:bucket4j-core:8.7.0'

@Component
public class RateLimiterService {

    private final Map<String, Bucket> buckets = new ConcurrentHashMap<>();

    public Bucket resolveBucket(String apiKey) {
        return buckets.computeIfAbsent(apiKey, key -> createNewBucket());
    }

    private Bucket createNewBucket() {
        // Allow 10 requests, refilling 10 tokens every 1 minute
        Bandwidth limit = Bandwidth.classic(10,
            Refill.intervally(10, Duration.ofMinutes(1)));
        return Bucket.builder().addLimit(limit).build();
    }
}

@Component
public class RateLimitFilter extends OncePerRequestFilter {

    private final RateLimiterService rateLimiterService;

    public RateLimitFilter(RateLimiterService rateLimiterService) {
        this.rateLimiterService = rateLimiterService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain filterChain) throws IOException, ServletException {
        String apiKey = request.getHeader("X-API-KEY");
        Bucket bucket = rateLimiterService.resolveBucket(apiKey);

        if (bucket.tryConsume(1)) {
            filterChain.doFilter(request, response);  // token available, proceed
        } else {
            response.setStatus(429); // Too Many Requests
            response.getWriter().write("Rate limit exceeded. Try again later.");
        }
    }
}
```

For a distributed system (multiple app instances), you'd back this with Redis instead of an in-memory map, so all instances share the same rate-limit counters:

```java
@Service
public class RedisRateLimiter {

    private final StringRedisTemplate redisTemplate;

    public RedisRateLimiter(StringRedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public boolean isAllowed(String userId, int maxRequests, Duration window) {
        String key = "rate_limit:" + userId;
        Long currentCount = redisTemplate.opsForValue().increment(key);

        if (currentCount != null && currentCount == 1L) {
            redisTemplate.expire(key, window); // set TTL only on first request in window
        }
        return currentCount != null && currentCount <= maxRequests;
    }
}
```

## Real systems that use it
Kong, AWS API Gateway, NGINX (`limit_req` module), Stripe's API (famously strict rate limits), Bucket4j, Resilience4j.

## Interview questions
- Token bucket vs leaky bucket — main difference?
- How do you implement rate limiting across multiple servers (not just one JVM)?
- What HTTP status code should a rate-limited request return? (429)
- How would you rate-limit per-user vs per-IP, and why would you need both?

---

# 7. Database Sharding

## What it is
Splitting one large database into multiple smaller databases ("shards"), each holding a subset of the data, spread across different machines.

## Why it exists
A single database server has limits — CPU, RAM, disk I/O, and connection count. When your `transactions` table has 2 billion rows and a single server can't keep up with the write/read load, you can't just "add more RAM" forever (vertical scaling has a ceiling). Sharding lets you scale **horizontally** — add more machines, each handling a slice of the data.

## How it works
You pick a **shard key** (e.g., `user_id`), and use a strategy to decide which shard a given row goes to:

### 1. Range-based sharding
`user_id 1–1,000,000` → Shard A, `user_id 1,000,001–2,000,000` → Shard B. Simple, but can create "hot shards" if certain ranges get much more traffic.

### 2. Hash-based sharding
`shard = hash(user_id) % number_of_shards`. Spreads data evenly, but adding/removing shards means re-hashing almost everything (mitigated by **consistent hashing**, which minimizes reshuffling).

### 3. Geographic/directory-based sharding
Users in India → India shard, users in US → US shard. Good for data residency/compliance laws (very relevant for banking!).

## Real-world analogy
Imagine one giant library with every book in the world (single DB) vs. splitting it into "Fiction Library," "Science Library," "History Library" (shards) in different buildings, so no single building is overwhelmed by every visitor.

## Java / Spring Boot example — conceptual sharding router

```java
@Component
public class ShardRouter {

    private final Map<Integer, DataSource> shardDataSources; // shardId -> DataSource
    private final int totalShards;

    public ShardRouter(Map<Integer, DataSource> shardDataSources) {
        this.shardDataSources = shardDataSources;
        this.totalShards = shardDataSources.size();
    }

    public DataSource getShardFor(Long userId) {
        int shardId = (int) (userId % totalShards);  // simple hash-based routing
        return shardDataSources.get(shardId);
    }
}

@Repository
public class ShardedAccountRepository {

    private final ShardRouter shardRouter;

    public ShardedAccountRepository(ShardRouter shardRouter) {
        this.shardRouter = shardRouter;
    }

    public Account findAccountByUserId(Long userId) {
        DataSource shard = shardRouter.getShardFor(userId);
        // Use JdbcTemplate or EntityManager against the resolved shard's DataSource
        JdbcTemplate jdbcTemplate = new JdbcTemplate(shard);
        return jdbcTemplate.queryForObject(
            "SELECT * FROM accounts WHERE user_id = ?",
            new AccountRowMapper(), userId);
    }
}
```

In practice, most teams don't hand-roll this — tools like **Vitess** (used by YouTube, Slack), **Citus** (Postgres extension), or cloud-managed sharded databases (DynamoDB, Cosmos DB) handle the routing for you. But interviewers want you to understand the concept above.

## Sharding challenges (important for interviews!)
- **Cross-shard joins are hard** — if `orders` are in Shard A and `users` are in Shard B, joining them requires app-level logic, not a simple SQL JOIN.
- **Rebalancing** — adding a new shard means moving data around, which is expensive and risky.
- **Hot shards** — a celebrity user or popular key can overload one shard while others sit idle.

## Interview questions
- What is a shard key, and what makes a good one?
- Range-based vs hash-based sharding — trade-offs?
- How do you handle a query that needs data from 2 different shards?
- What is resharding, and why is it painful?

---

# 8. Replication

## What it is
Keeping copies of the same data on multiple database servers, so you have redundancy (for disaster recovery) and can spread out read traffic.

## Why it exists
1. **High Availability** — if your one and only DB server dies, your entire app goes down. With replication, a replica can take over.
2. **Read scaling** — reads (SELECTs) usually vastly outnumber writes. Route reads to multiple replicas instead of hammering one server.
3. **Disaster recovery** — a replica in a different data center/region protects you if an entire region goes down.

## How it works — Master-Slave (Primary-Replica) setup
- **Primary (Master)**: handles all **writes**.
- **Replica (Slave)**: receives a continuous stream of changes from the primary and stays in sync; handles **reads**.

```
                 WRITES
Client  ────────────────────►  Primary DB
                                    │
                          (replication stream)
                          ▼         ▼         ▼
                     Replica 1  Replica 2  Replica 3
                          ▲         ▲         ▲
Client  ────────────────────────────────────────
                 READS (load-balanced across replicas)
```

### Synchronous vs Asynchronous replication
- **Synchronous** — the primary waits for the replica to confirm the write before telling the client "success." Strong consistency, but slower (and if the replica is down, writes can stall).
- **Asynchronous** — the primary confirms the write immediately, replicas catch up in the background. Faster, but there's a small window where replicas have stale data ("replication lag").

### Master-Master (Multi-Primary) replication
Both nodes accept writes and sync with each other. Higher availability for writes, but introduces the hard problem of **conflict resolution** (what if both nodes get a conflicting write to the same row at the same time?).

## Real-world analogy
Think of a company with a head office (primary) that makes all the official decisions, and regional branch offices (replicas) that get copies of every decision so local customers can be served faster — but if a branch's copy is a few minutes behind the head office's latest update, that's "replication lag."

## Java / Spring Boot example — routing reads vs writes

```java
public enum DbType {
    WRITE, READ
}

public class RoutingContext {
    private static final ThreadLocal<DbType> CONTEXT = ThreadLocal.withInitial(() -> DbType.WRITE);

    public static void setReadOnly() { CONTEXT.set(DbType.READ); }
    public static void setWrite() { CONTEXT.set(DbType.WRITE); }
    public static DbType get() { return CONTEXT.get(); }
}

public class ReplicationRoutingDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return RoutingContext.get();  // returns WRITE or READ
    }
}

@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource routingDataSource(
            @Qualifier("primaryDataSource") DataSource primary,
            @Qualifier("replicaDataSource") DataSource replica) {

        ReplicationRoutingDataSource routingDataSource = new ReplicationRoutingDataSource();
        Map<Object, Object> targets = new HashMap<>();
        targets.put(DbType.WRITE, primary);
        targets.put(DbType.READ, replica);
        routingDataSource.setTargetDataSources(targets);
        routingDataSource.setDefaultTargetDataSource(primary);
        return routingDataSource;
    }
}

@Service
public class AccountReadService {

    @Transactional(readOnly = true)
    public Account getAccount(Long id) {
        RoutingContext.setReadOnly();   // this query goes to a replica
        try {
            // repository call happens here
            return accountRepository.findById(id).orElseThrow();
        } finally {
            RoutingContext.setWrite();  // reset for the next request
        }
    }
}
```

## Interview questions
- Synchronous vs asynchronous replication — trade-offs?
- What is replication lag, and how does it cause bugs? (e.g., user updates profile, immediately reloads, sees old data because the read hit a lagging replica)
- Master-Slave vs Master-Master — when would you choose each?
- How does replication relate to the CAP theorem? (see next section!)

---

# 9. CAP Theorem

## What it is
CAP Theorem states that a distributed data system can only guarantee **2 out of 3** of the following at the same time, when a network partition happens:

- **C — Consistency**: every read gets the most recent write (or an error). All nodes see the same data at the same time.
- **A — Availability**: every request gets a (non-error) response, even if it's not the most up-to-date data.
- **P — Partition Tolerance**: the system keeps working even if network communication between nodes breaks down.

## Why it matters
In real distributed systems, network partitions **will** happen — cables get cut, servers lose connectivity, data centers have outages. So **P is not optional** — you must design for it. That means the real choice is between **C and A** when a partition occurs.

- **CP system** (Consistency + Partition tolerance): During a partition, the system refuses to respond rather than return possibly-stale data. Example: A banking ledger — you'd rather show an error than show the wrong balance.
- **AP system** (Availability + Partition tolerance): During a partition, the system keeps responding even if the data might be slightly stale. Example: A social media "like" counter — showing a slightly outdated count is fine, refusing to load the page is not.

> Note: You already studied this in-depth with CP vs AP banking decision frameworks — this section is the refresher/interview-ready version.

## Real-world analogy
Two ATMs in different cities, network link between the bank's central servers goes down (partition):
- **CP choice**: Both ATMs refuse withdrawals until they can confirm with the central ledger that the balance is correct — annoying for the customer, but no risk of double-spending.
- **AP choice**: Both ATMs allow withdrawals based on their last known balance — customer is happy, but if they had ₹1000 and withdrew ₹800 from both ATMs "simultaneously," the bank just lost ₹600 to a consistency bug.

Real banking systems mostly pick **CP for the ledger itself**, but might pick **AP for less critical things** like "recent transaction history shown in the app."

## Java / Spring Boot example — illustrating a CP-style guard

```java
@Service
public class WithdrawalService {

    private final AccountRepository accountRepository;
    private final ReplicationHealthChecker healthChecker;

    public WithdrawalService(AccountRepository accountRepository,
                              ReplicationHealthChecker healthChecker) {
        this.accountRepository = accountRepository;
        this.healthChecker = healthChecker;
    }

    @Transactional
    public void withdraw(String accountNumber, BigDecimal amount) {
        // CP behavior: if we can't confirm we're talking to the
        // up-to-date primary (partition suspected), REFUSE the operation
        // rather than risk operating on stale data.
        if (!healthChecker.isPrimaryReachableAndInSync()) {
            throw new ServiceUnavailableException(
                "Cannot guarantee consistency right now. Please retry shortly.");
        }

        Account account = accountRepository.findByAccountNumberForUpdate(accountNumber)
            .orElseThrow(() -> new AccountNotFoundException(accountNumber));

        if (account.getBalance().compareTo(amount) < 0) {
            throw new InsufficientFundsException(accountNumber);
        }
        account.setBalance(account.getBalance().subtract(amount));
        accountRepository.save(account);
    }
}
```

Compare that to an AP-style approach for something non-critical, like a "recently viewed transactions" cache, where you'd just serve from whatever replica is reachable, even if slightly stale, instead of throwing an error.

## Real systems and their CAP choice
| System | Choice | Why |
|---|---|---|
| MongoDB (default config) | CP | Strong consistency for a single primary |
| Cassandra | AP (tunable) | Built for always-on, globally distributed writes |
| DynamoDB | AP (tunable) | Amazon prioritizes availability, offers "eventually consistent" or "strongly consistent" reads as a choice |
| Zookeeper | CP | Used for coordination — must be consistent |
| Traditional single-node RDBMS (MySQL, PostgreSQL) | CA (not really distributed) | CAP theorem technically doesn't apply until you replicate/partition it |

## Interview questions
- Give a real example of a CP system and an AP system.
- Why is "Partition Tolerance" not really optional in real distributed systems?
- How would you design a banking system around CAP theorem trade-offs?
- What is PACELC, and how does it extend CAP theorem? (Bonus: PACELC says even without a partition, you must choose between Latency and Consistency — worth mentioning in senior interviews)

---

# 10. Eventual Consistency

## What it is
A consistency model where, if no new writes happen, **all replicas will eventually converge to the same value** — but at any given moment, different nodes might have different (stale) values.

## Why it exists
It's the practical trade-off AP systems make. Instead of blocking every read until every replica agrees (which kills availability and speed), you accept that replicas sync up "eventually" — usually within milliseconds to seconds — in exchange for the system always being available and fast.

## How it works
1. Write happens on Node A.
2. Node A immediately returns success to the client (doesn't wait for other nodes).
3. In the background, Node A propagates the change to Node B, Node C, etc.
4. For a brief window, if you read from Node B before it received the update, you get the **old** value.
5. Eventually (hence the name), all nodes converge.

### Techniques used to make eventual consistency less painful
- **Read-your-own-writes consistency**: ensure a user always sees their own recent writes, even if other users might not yet.
- **Vector clocks / version vectors**: track the causal order of updates across nodes to resolve conflicts correctly.
- **Conflict-free Replicated Data Types (CRDTs)**: data structures specifically designed so concurrent updates can always be merged automatically without conflicts.

## Real-world analogy
Think of a WhatsApp group's "last seen" status syncing across your phone, laptop, and tablet. You mark a message as read on your phone. For a second or two, your laptop might still show it as unread. Eventually, it catches up. Nobody's upset about this because it's not mission-critical data.

## Java / Spring Boot example — DynamoDB-style eventually consistent read

```java
@Service
public class UserProfileService {

    private final DynamoDbClient dynamoDbClient;

    public UserProfileService(DynamoDbClient dynamoDbClient) {
        this.dynamoDbClient = dynamoDbClient;
    }

    // Eventually consistent read: cheaper, faster, might return slightly stale data
    public UserProfile getProfileEventuallyConsistent(String userId) {
        GetItemRequest request = GetItemRequest.builder()
            .tableName("UserProfiles")
            .key(Map.of("userId", AttributeValue.builder().s(userId).build()))
            .consistentRead(false)   // <-- eventually consistent (default, cheaper)
            .build();

        return mapToProfile(dynamoDbClient.getItem(request).item());
    }

    // Strongly consistent read: guarantees latest write, costs more, slightly slower
    public UserProfile getProfileStronglyConsistent(String userId) {
        GetItemRequest request = GetItemRequest.builder()
            .tableName("UserProfiles")
            .key(Map.of("userId", AttributeValue.builder().s(userId).build()))
            .consistentRead(true)    // <-- forces strong consistency
            .build();

        return mapToProfile(dynamoDbClient.getItem(request).item());
    }

    private UserProfile mapToProfile(Map<String, AttributeValue> item) {
        // mapping logic here
        return new UserProfile();
    }
}
```

A simple illustration of "read-your-own-writes" using a session-local cache to avoid showing a user their own stale data:

```java
@Service
public class ProfileUpdateService {

    private final UserProfileService userProfileService;
    private final Map<String, UserProfile> sessionCache = new ConcurrentHashMap<>();

    public ProfileUpdateService(UserProfileService userProfileService) {
        this.userProfileService = userProfileService;
    }

    public void updateProfile(String userId, UserProfile updated) {
        // write to the DB (eventually propagates to all replicas)
        userProfileService.save(updated);
        // but cache locally so THIS user immediately sees their own change,
        // even if a replica read would still return the old value
        sessionCache.put(userId, updated);
    }

    public UserProfile getProfile(String userId) {
        if (sessionCache.containsKey(userId)) {
            return sessionCache.get(userId); // read-your-own-writes guarantee
        }
        return userProfileService.getProfileEventuallyConsistent(userId);
    }
}
```

## Real systems that use it
DynamoDB, Cassandra, DNS (yes — DNS propagation is a classic eventual consistency example!), CouchDB.

## Interview questions
- Give a real-world feature where eventual consistency is perfectly acceptable.
- Give a real-world feature where it is NOT acceptable.
- What is "read-your-own-writes" consistency and why does it matter for UX?
- How would you resolve a conflict when two nodes get concurrent writes to the same key?

---

# 11. Distributed Transactions

## What it is
A mechanism to ensure that an operation spanning **multiple services or databases** either fully succeeds everywhere, or fully rolls back everywhere — extending the "Atomicity" guarantee (from ACID) across service/database boundaries.

## Why it exists
In a microservices world, a single business operation often touches multiple services. Example: "Transfer money" might need to:
1. Debit Account A (in `account-service`'s database)
2. Credit Account B (in `account-service`'s database, or even a different service)
3. Send a notification (in `notification-service`)
4. Log the transaction (in `audit-service`)

A normal `@Transactional` only protects **one** database. If step 1 succeeds but step 2 fails, you now have money that vanished. Distributed transactions solve this.

## Approach 1: Two-Phase Commit (2PC)
A coordinator asks all participants "can you commit?" (**Phase 1: Prepare**). If everyone says yes, coordinator tells everyone to actually commit (**Phase 2: Commit**). If anyone says no, coordinator tells everyone to rollback.

**Downside**: It's blocking — if the coordinator crashes between phases, participants can be stuck holding locks indefinitely. Rarely used in modern microservices because it doesn't scale well and creates tight coupling.

## Approach 2: Saga Pattern (much more common today)
Break the transaction into a sequence of local transactions, each with a corresponding **compensating action** to undo it if a later step fails.

### Choreography-based Saga
Each service listens for events and reacts — no central coordinator.
```
OrderService: creates order → publishes "OrderCreated"
PaymentService: listens, charges card → publishes "PaymentCompleted"
InventoryService: listens, reserves stock → publishes "StockReserved"

If PaymentService fails: publishes "PaymentFailed"
OrderService: listens, cancels the order (compensating action)
```

### Orchestration-based Saga
A central orchestrator explicitly tells each service what to do, step by step, and handles failures by calling compensating actions.

## Real-world analogy
Booking a flight + hotel + rental car as one trip package. If the hotel booking fails after the flight is already booked, you don't want to be stuck with a flight and no hotel — the system needs to **cancel the flight (compensating action)** to undo the partial success.

## Java / Spring Boot example — Orchestration-based Saga

```java
public enum SagaStep {
    ORDER_CREATED, PAYMENT_PROCESSED, INVENTORY_RESERVED, COMPLETED, FAILED
}

@Service
public class OrderSagaOrchestrator {

    private final OrderService orderService;
    private final PaymentServiceClient paymentClient;
    private final InventoryServiceClient inventoryClient;

    public OrderSagaOrchestrator(OrderService orderService,
                                  PaymentServiceClient paymentClient,
                                  InventoryServiceClient inventoryClient) {
        this.orderService = orderService;
        this.paymentClient = paymentClient;
        this.inventoryClient = inventoryClient;
    }

    public void executeOrderSaga(OrderRequest request) {
        Order order = orderService.createOrder(request);   // Step 1

        try {
            paymentClient.charge(order.getId(), order.getAmount());   // Step 2
        } catch (PaymentFailedException e) {
            orderService.cancelOrder(order.getId());  // compensate step 1
            throw new SagaFailedException("Payment failed, order cancelled", e);
        }

        try {
            inventoryClient.reserveStock(order.getItems());   // Step 3
        } catch (InsufficientStockException e) {
            paymentClient.refund(order.getId());        // compensate step 2
            orderService.cancelOrder(order.getId());     // compensate step 1
            throw new SagaFailedException("Stock reservation failed, rolled back", e);
        }

        orderService.markCompleted(order.getId());
    }
}
```

## Real systems that use it
Netflix (pioneered Saga pattern at scale), Uber (uses orchestration sagas for ride booking + payment), most large e-commerce platforms.

## Interview questions
- Why is 2PC rarely used in modern microservices?
- Choreography vs orchestration saga — trade-offs?
- What is a "compensating transaction"?
- How do you handle a saga step that fails AFTER a compensating action has already started (i.e., partial failure of rollback itself)?

---

# 12. Consensus Algorithms

## What it is
Algorithms that allow a group of distributed nodes to **agree on a single value or decision**, even if some nodes fail or messages get delayed/lost.

## Why it exists
In a distributed system with multiple nodes, you often need everyone to agree on something critical: "who is the leader?", "what is the correct order of these writes?", "did this transaction actually commit?" If nodes can't reach agreement reliably, you get split-brain scenarios (two nodes both think they're the leader) and data corruption.

## Paxos
The original, famously difficult-to-understand consensus algorithm (Leslie Lamport, 1989). Nodes propose values; a majority ("quorum") must accept a value before it's considered agreed upon. Extremely rigorous but notoriously hard to implement correctly — most engineers use libraries rather than writing it from scratch.

## Raft (much more commonly used/understood today)
Designed specifically to be **easier to understand** than Paxos while providing the same guarantees. Breaks the problem into 3 sub-problems:

1. **Leader Election** — nodes vote to elect one leader among themselves (using randomized timeouts to avoid ties).
2. **Log Replication** — the leader receives all writes, appends them to its log, and replicates that log to followers.
3. **Safety** — ensures that once a majority of nodes have committed an entry, it will never be lost, even if the leader crashes right after.

### How Raft leader election works (simplified)
1. All nodes start as "followers." Each has a random election timeout.
2. If a follower doesn't hear from a leader within its timeout, it becomes a "candidate" and requests votes from other nodes.
3. If it gets votes from a **majority**, it becomes the leader.
4. The leader sends periodic "heartbeats" to prevent followers from starting new elections.
5. If the leader crashes, followers stop getting heartbeats, timeout, and a new election starts.

## Real-world analogy
Imagine a company where the CEO suddenly goes missing. The board (nodes) can't just have everyone claim to be CEO — that's chaos (split-brain). Instead, they hold a vote; whoever gets a **majority** becomes the new CEO, and everyone agrees to follow that person's decisions until they, too, go missing.

## Java example — simplified Raft-style leader election logic (educational, not production-grade)

```java
public class RaftNode {

    enum State { FOLLOWER, CANDIDATE, LEADER }

    private State state = State.FOLLOWER;
    private int currentTerm = 0;
    private String votedFor = null;
    private final String nodeId;
    private final List<RaftNode> peers;
    private final Random random = new Random();

    public RaftNode(String nodeId, List<RaftNode> peers) {
        this.nodeId = nodeId;
        this.peers = peers;
    }

    // Called when election timeout fires (no heartbeat received from leader)
    public void startElection() {
        state = State.CANDIDATE;
        currentTerm++;
        votedFor = nodeId;

        int votesReceived = 1; // vote for self
        for (RaftNode peer : peers) {
            if (peer.requestVote(currentTerm, nodeId)) {
                votesReceived++;
            }
        }

        int majority = (peers.size() + 1) / 2 + 1;
        if (votesReceived >= majority) {
            becomeLeader();
        } else {
            state = State.FOLLOWER;  // election failed, revert and wait to retry
        }
    }

    public synchronized boolean requestVote(int term, String candidateId) {
        if (term > currentTerm && (votedFor == null || votedFor.equals(candidateId))) {
            currentTerm = term;
            votedFor = candidateId;
            return true;   // grants vote
        }
        return false;      // already voted this term, or stale term
    }

    private void becomeLeader() {
        state = State.LEADER;
        System.out.println(nodeId + " became leader for term " + currentTerm);
        // In a real system: start sending periodic heartbeats to followers here
    }
}
```

> In production, you'd never implement Raft yourself — you'd use **Zookeeper**, **etcd** (which uses Raft internally), or **Consul**. This code exists purely to make the concept concrete for interviews.

## Real systems that use it
- **etcd** (Raft) — powers Kubernetes' entire cluster state
- **Zookeeper** (a Paxos-like protocol called ZAB) — used by Kafka (older versions), Hadoop
- **Consul** (Raft) — service discovery + config

## Interview questions
- What problem does consensus solve that simple replication doesn't?
- Explain Raft leader election in your own words.
- What is "split-brain" and how does consensus prevent it?
- Why does Raft need a majority (quorum) rather than all nodes agreeing?

---

# 13. Service Discovery

## What it is
A mechanism that automatically tracks the network locations (IP + port) of service instances, so other services can find and call them **without hardcoding addresses** — critical because in cloud/container environments, instances get created, destroyed, and rescheduled constantly with new IPs.

## Why it exists
In a microservices architecture with auto-scaling, `order-service` might run on 3 instances today and 8 tomorrow, each with a different, dynamically-assigned IP. If `payment-service` hardcodes `order-service`'s IP, it breaks the moment that instance restarts or scales. Service discovery solves this by maintaining a live, self-updating registry.

## How it works

### Client-side discovery
1. Each service instance **registers itself** with a service registry on startup (e.g., "order-service is at 10.0.1.5:8080").
2. It sends periodic **heartbeats** to prove it's still alive.
3. When `payment-service` wants to call `order-service`, it asks the registry: "give me all healthy instances of order-service," then picks one (often combined with client-side load balancing).
4. If an instance stops sending heartbeats, the registry removes it.

### Server-side discovery
The client just calls a fixed load balancer/router address; the router itself queries the registry and forwards the request — client doesn't need discovery logic at all (e.g., AWS ECS with an ALB).

## Real-world analogy
Think of a company directory app. Employees move desks constantly (new IPs). Instead of memorizing "Ravi sits at desk 4B," you look him up in the directory (registry) every time, and it always shows his current desk. If Ravi leaves the building (instance dies) and doesn't check back in, the directory removes him.

## Java / Spring Boot example — Eureka

**Service registering itself (order-service):**
```java
// application.yml
// spring.application.name: order-service
// eureka.client.service-url.defaultZone: http://localhost:8761/eureka/

@SpringBootApplication
@EnableEurekaClient
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

**Eureka Server itself (the registry):**
```java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(DiscoveryServerApplication.class, args);
    }
}
```

**Another service discovering and calling order-service (payment-service):**
```java
@Service
public class PaymentService {

    private final DiscoveryClient discoveryClient;
    private final RestTemplate restTemplate;

    public PaymentService(DiscoveryClient discoveryClient, RestTemplate restTemplate) {
        this.discoveryClient = discoveryClient;
        this.restTemplate = restTemplate;
    }

    public Order fetchOrder(String orderId) {
        // Ask Eureka: "what instances of order-service are currently alive?"
        List<ServiceInstance> instances = discoveryClient.getInstances("order-service");

        if (instances.isEmpty()) {
            throw new ServiceUnavailableException("No instances of order-service available");
        }

        ServiceInstance instance = instances.get(0);  // (real code would load-balance across these)
        String url = instance.getUri() + "/orders/" + orderId;
        return restTemplate.getForObject(url, Order.class);
    }
}
```

Or, more idiomatically, combine it with `@LoadBalanced RestTemplate` (shown earlier in Load Balancer section) so Eureka lookup + load balancing happen automatically behind a logical service name like `http://order-service/orders/123`.

## Real systems that use it
Netflix Eureka, HashiCorp Consul, etcd, Kubernetes' built-in service discovery (DNS-based), Apache Zookeeper.

## Interview questions
- Client-side vs server-side service discovery — trade-offs?
- How does a service registry detect a dead instance?
- How does Kubernetes handle service discovery differently from Eureka?
- What happens if the service registry itself goes down? (This is why registries like Eureka are designed to be highly available/self-preserving — services can keep talking to their last-known-good list of instances)

---

# 14. Message Queues

## What it is
A system that lets services communicate **asynchronously** by sending messages through an intermediary queue, instead of calling each other directly and waiting for a response.

## Why it exists
Direct (synchronous) service-to-service calls create tight coupling and fragility: if `notification-service` is slow or down, and `order-service` calls it directly and waits, then `order-service` also becomes slow — even though sending a notification isn't actually urgent. Message queues **decouple** producers from consumers:
- The producer doesn't need to know who's listening, or wait for them to finish.
- If the consumer is temporarily down, messages just wait safely in the queue until it's back.
- You can add more consumers to handle load, without changing the producer at all.

## How it works
1. **Producer** publishes a message to a **queue** or **topic**.
2. Message sits in the broker (e.g., RabbitMQ, Kafka) until a consumer picks it up.
3. **Consumer** processes the message, often at its own pace.
4. Once processed, the message is acknowledged and removed (or, in Kafka's case, retained but marked as read up to that "offset").

### Queue vs Topic (Pub/Sub)
- **Queue (point-to-point)**: one message is consumed by exactly **one** consumer, even if multiple consumers are listening (used for work distribution — e.g., "process this payment").
- **Topic (publish/subscribe)**: one message is delivered to **every** subscriber (used for broadcasting events — e.g., "OrderCreated" event needs to be seen by inventory, notification, AND analytics services).

### RabbitMQ vs Kafka (a very common interview question)
| | RabbitMQ | Kafka |
|---|---|---|
| Model | Traditional message broker (push-based) | Distributed log (pull-based) |
| Best for | Complex routing, task queues, lower-latency individual messages | High-throughput event streaming, replay-ability |
| Message retention | Deleted after consumption (typically) | Retained for a configured period, even after being read |
| Ordering | Per-queue | Per-partition |

## Real-world analogy
Think of a restaurant kitchen order system. The waiter (producer) doesn't personally walk into the kitchen and wait for the chef to cook — they clip the order ticket to a rail (the queue). Chefs (consumers) pick up tickets as they become free. If a chef is busy, the ticket just waits on the rail — the waiter has already moved on to serve other tables.

## Java / Spring Boot example — RabbitMQ

```java
@Configuration
public class RabbitMQConfig {

    public static final String QUEUE_NAME = "notification.queue";

    @Bean
    public Queue notificationQueue() {
        return new Queue(QUEUE_NAME, true); // durable = survives broker restart
    }
}

// PRODUCER
@Service
public class OrderService {

    private final RabbitTemplate rabbitTemplate;

    public OrderService(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    public void placeOrder(Order order) {
        saveOrderToDb(order);

        // Don't wait for notification to be sent — fire and forget
        rabbitTemplate.convertAndSend(
            RabbitMQConfig.QUEUE_NAME,
            new NotificationMessage(order.getUserId(), "Your order has been placed!"));
    }

    private void saveOrderToDb(Order order) { /* ... */ }
}

// CONSUMER
@Component
public class NotificationConsumer {

    @RabbitListener(queues = RabbitMQConfig.QUEUE_NAME)
    public void handleNotification(NotificationMessage message) {
        System.out.println("Sending notification to user " + message.getUserId()
            + ": " + message.getText());
        // Even if this takes 3 seconds, the order-placement API already responded fast
    }
}
```

## Java / Spring Boot example — Kafka (event broadcasting to multiple services)

```java
// PRODUCER
@Service
public class OrderEventProducer {

    private final KafkaTemplate<String, OrderCreatedEvent> kafkaTemplate;

    public OrderEventProducer(KafkaTemplate<String, OrderCreatedEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void publishOrderCreated(Order order) {
        OrderCreatedEvent event = new OrderCreatedEvent(order.getId(), order.getUserId(), order.getTotal());
        kafkaTemplate.send("order-events", order.getId().toString(), event);
        // "order-id" as key ensures all events for the same order go to the same partition,
        // preserving order of events for that specific order
    }
}

// CONSUMER 1 — inventory-service listens to the same topic
@Component
public class InventoryEventConsumer {

    @KafkaListener(topics = "order-events", groupId = "inventory-service")
    public void onOrderCreated(OrderCreatedEvent event) {
        System.out.println("Reserving stock for order " + event.getOrderId());
    }
}

// CONSUMER 2 — analytics-service ALSO listens to the same topic independently
@Component
public class AnalyticsEventConsumer {

    @KafkaListener(topics = "order-events", groupId = "analytics-service")
    public void onOrderCreated(OrderCreatedEvent event) {
        System.out.println("Recording analytics for order " + event.getOrderId());
    }
}
```
Both consumers receive **every** event independently (different `groupId`s), which is the pub/sub behavior — this is the backbone of event-driven microservices architecture.

> This lines up with the Kafka order-management system you already built — this section is the conceptual "why" behind that project.

## Real systems that use it
Apache Kafka, RabbitMQ, AWS SQS/SNS, Google Pub/Sub, ActiveMQ.

## Interview questions
- Queue vs Topic — give a real example for each.
- Why is Kafka better for event streaming while RabbitMQ is better for task queues?
- How do you guarantee message ordering in Kafka? (Partitioning by key)
- What is "at-least-once" vs "exactly-once" vs "at-most-once" delivery, and which is hardest to achieve?
- How do you handle a "poison message" that keeps crashing your consumer? (Dead Letter Queue)

---

# 15. System Observability

## What it is
The ability to understand what's happening **inside** a running system from the outside, using three main pillars: **Logs, Metrics, and Traces**.

## Why it exists
In a monolith, if something breaks, you check one server's logs. In a distributed system with 50 microservices, a single user request might touch 10 different services. If something's slow or broken, "which of the 10 services is the problem?" becomes very hard to answer without observability tooling.

## The Three Pillars

### 1. Logging
Recording discrete events with context ("User 123 failed login at 10:32:05, reason: wrong password"). Good logging is **structured** (JSON, not free text) so it can be searched/filtered easily.

```java
@Slf4j
@Service
public class LoginService {

    public void login(String username, String password) {
        MDC.put("username", username);   // adds context to every log line in this thread
        try {
            if (!isValidPassword(username, password)) {
                log.warn("Login failed for user={} reason=invalid_password", username);
                throw new InvalidCredentialsException();
            }
            log.info("Login successful for user={}", username);
        } finally {
            MDC.clear();
        }
    }

    private boolean isValidPassword(String username, String password) {
        return true; // placeholder
    }
}
```

### 2. Metrics
Numeric measurements over time — request count, error rate, latency percentiles (p50, p95, p99), CPU/memory usage. Metrics answer "how is the system doing overall?" and power dashboards/alerts.

```java
@RestController
public class OrderController {

    private final MeterRegistry meterRegistry;
    private final OrderService orderService;

    public OrderController(MeterRegistry meterRegistry, OrderService orderService) {
        this.meterRegistry = meterRegistry;
        this.orderService = orderService;
    }

    @PostMapping("/orders")
    public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
        Timer.Sample sample = Timer.start(meterRegistry);
        try {
            Order order = orderService.create(request);
            meterRegistry.counter("orders.created", "status", "success").increment();
            return ResponseEntity.ok(order);
        } catch (Exception e) {
            meterRegistry.counter("orders.created", "status", "failure").increment();
            throw e;
        } finally {
            sample.stop(meterRegistry.timer("orders.create.duration"));
        }
    }
}
```
With `spring-boot-starter-actuator` + Micrometer, these metrics can be scraped by **Prometheus** and visualized in **Grafana** dashboards automatically.

### 3. Distributed Tracing
Tracks a **single request's journey** across multiple services, showing exactly where time was spent. Each request gets a unique **trace ID** that's passed along through every service call.

```java
// With Spring Cloud Sleuth / Micrometer Tracing, this happens mostly automatically —
// but conceptually, each service call propagates a trace ID + span ID:

@RestController
public class OrderController {

    private final Tracer tracer;
    private final PaymentServiceClient paymentClient;

    public OrderController(Tracer tracer, PaymentServiceClient paymentClient) {
        this.tracer = tracer;
        this.paymentClient = paymentClient;
    }

    @PostMapping("/orders")
    public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
        Span span = tracer.nextSpan().name("create-order").start();
        try (Tracer.SpanInScope ws = tracer.withSpanInScope(span)) {
            // trace context automatically propagates to paymentClient's HTTP call
            // via headers like "traceparent", so payment-service's logs/spans
            // link back to THIS specific request
            paymentClient.charge(request.getAmount());
            return ResponseEntity.ok(new Order());
        } finally {
            span.end();
        }
    }
}
```
When you look this trace ID up in **Jaeger** or **Zipkin**, you see a visual timeline: `API Gateway (5ms) → Order Service (20ms) → Payment Service (350ms!) → Inventory Service (10ms)` — instantly telling you Payment Service is the bottleneck.

## Real-world analogy
- **Logs** = a detailed diary entry for every event ("9:03am — customer called, upset about a late delivery")
- **Metrics** = the dashboard in a factory showing overall throughput, temperature, error rate — a bird's-eye view
- **Traces** = following one specific customer's order through every department (warehouse → packing → shipping → delivery) to see exactly which department caused the delay

## Real systems that use it
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana), Splunk
- **Metrics**: Prometheus + Grafana, Datadog
- **Tracing**: Jaeger, Zipkin, AWS X-Ray

## Interview questions
- What are the three pillars of observability, and what does each answer?
- How does distributed tracing work across service boundaries? (trace ID propagation via headers)
- What's the difference between monitoring and observability?
- How would you debug a slow API that touches 5 microservices, using these tools?

---

# Quick Recap Table (for revision before interviews)

| Topic | One-line summary | Real tool example |
|---|---|---|
| Load Balancer | Spreads traffic across servers | NGINX, AWS ELB |
| API Gateway | Single entry point for all microservices | Spring Cloud Gateway, Kong |
| CDN | Caches static content near the user | Cloudflare, CloudFront |
| Database Fundamentals | SQL (ACID, relational) vs NoSQL (flexible, scalable) | PostgreSQL, MongoDB |
| Caching | Store hot data in memory for speed | Redis, Memcached |
| Rate Limiting | Cap requests to prevent abuse | Bucket4j, Kong |
| Database Sharding | Split data across multiple DB servers | Vitess, Citus |
| Replication | Copy data across servers for HA + read scaling | MySQL replication |
| CAP Theorem | Pick 2 of Consistency/Availability/Partition tolerance | MongoDB (CP), Cassandra (AP) |
| Eventual Consistency | Replicas converge over time, not instantly | DynamoDB, Cassandra |
| Distributed Transactions | Multi-service atomicity via Saga pattern | Netflix-style Sagas |
| Consensus Algorithms | Nodes agree on a value despite failures | Raft (etcd), Paxos (Zookeeper/ZAB) |
| Service Discovery | Auto-tracks live service instances | Eureka, Consul, Kubernetes DNS |
| Message Queues | Async decoupled communication | Kafka, RabbitMQ |
| System Observability | Logs + Metrics + Traces to understand the system | ELK, Prometheus/Grafana, Jaeger |

---

# Suggested Study Order (if starting from zero)

1. Database Fundamentals → understand SQL/ACID first, everything else builds on this
2. Caching → easiest performance win, conceptually simple
3. Load Balancer + API Gateway → understand traffic entry points
4. CDN + Rate Limiting → round out the "basic" layer
5. Replication → before sharding (replication is simpler)
6. Database Sharding → now that replication makes sense
7. CAP Theorem → the conceptual foundation for everything advanced
8. Eventual Consistency → direct extension of CAP
9. Message Queues → very practical, ties into your existing Kafka project
10. Service Discovery → needed to understand microservices communication
11. Distributed Transactions (Saga) → builds on message queues + service discovery
12. Consensus Algorithms → hardest topic, save for last
13. System Observability → wrap-up topic, ties the whole system together operationally

---

*Notes generated from your system design roadmap image — covering Basic, Intermediate, and Advanced tiers with Java/Spring Boot examples tailored to your banking-domain background.*
