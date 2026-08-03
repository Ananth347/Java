# Spring Boot Production-Level Interview Questions (100+)

These are the "what would you actually do" questions — scenario-based, production debugging and troubleshooting style, the kind senior interviewers ask to see if you've really worked on live systems, not just built features. Answers are in plain English, describing the actual thought process and steps, not textbook theory.

---

## Section 1: "The App Is Down" — First Response

**1. Production is down. What's the very first thing you do?**
Stay calm and confirm the scope first — is it fully down, or partially degraded, and is it affecting all users or a specific region/feature? Check the health check endpoint and basic connectivity before assuming the worst. The goal in the first few minutes is triage, not root-causing — figure out if you need to roll back immediately to restore service, then investigate the actual cause afterward.

**2. How do you check if the application itself has crashed vs. just being slow?**
Check if the process is even running (`ps -ef | grep java` on the server, or the pod status in Kubernetes). If the process is alive but not responding, it's likely a hang, deadlock, or resource exhaustion rather than a crash — you'd then look at thread dumps and CPU/memory metrics instead of assuming it's simply down.

**3. What's the difference between checking application logs vs. system-level logs when debugging an outage?**
Application logs (from your Spring Boot app) tell you what your code was doing — exceptions, business logic errors. System-level logs (like `dmesg`, or the Kubernetes events) tell you about the environment — the OS killing a process due to memory pressure, disk running out of space, or the container being restarted by the orchestrator. You often need both to get the full picture.

**4. The application won't even start — what do you check first?**
Read the startup logs from the very top, not just the last error — Spring Boot's failure messages are usually quite descriptive about what failed (a missing bean, a bad property, a database connection failure). Common culprits: a missing or wrong environment variable/config value, a port already in use, or a database that isn't reachable yet at startup.

**5. Your app throws "Port already in use" on startup — what does that mean and how do you fix it?**
It means another process is already listening on the port your app is trying to bind to — often a previous instance of the same app that didn't shut down cleanly. You find the process using that port (e.g., `lsof -i :8080`) and either kill it or change your app's port, and longer term, make sure your deployment process properly stops the old instance before starting a new one.

**6. How do you quickly determine if an outage is caused by your application or by an upstream dependency (like a database or third-party API)?**
Check your application's own health endpoint components — most `/actuator/health` setups break down health by component (database, disk space, custom checks), immediately showing which dependency is failing. If the app's own health check passes but the app is still failing, the problem is more likely inside your own code or a dependency not covered by the health check.

**7. What is a rollback, and when should you choose to roll back instead of trying to fix forward?**
A rollback means reverting to the last known-good deployed version instead of trying to patch the current broken one live. You choose to roll back when the issue is clearly tied to a recent deployment and the fix isn't immediately obvious — restoring service quickly takes priority over root-causing under pressure; you investigate the actual bug afterward, calmly, with the pressure off.

**8. How would you confirm whether a recent deployment caused the outage?**
Check the deployment timeline against when the issue started — if the outage began right after a release, that's a strong signal. Compare the current version's behavior against the previous version's logs/metrics if you can, and check the diff of what actually changed in that release.

**9. What's the danger of restarting the application immediately without investigating first?**
You lose valuable diagnostic information — thread dumps, heap dumps, and the exact state that caused the failure disappear the moment the process restarts. If it's a recurring issue, restarting without understanding it just delays the same failure happening again, sometimes at an even worse time.

**10. What is a "runbook," and why do production teams maintain one?**
It's a documented, step-by-step guide for handling specific known types of incidents (like "database connection pool exhausted" or "disk space full") — having one means whoever's on call, even if they didn't build the feature, can follow clear, tested steps under pressure instead of improvising during a stressful outage.

---

## Section 2: High CPU & Performance Degradation

**11. The application's CPU usage suddenly spikes to 100%. How do you investigate?**
Take a thread dump while the CPU is high (using `jstack` or an APM tool), and correlate it with CPU usage per thread (using `top -H` on Linux to see per-thread CPU usage, then matching thread IDs to the dump) — this tells you exactly which thread(s) are burning CPU and what code they're executing at that moment.

**12. What is a thread dump, and what does it show you?**
It's a snapshot of every thread in the JVM at a specific moment — what each thread is currently doing, its state (running, waiting, blocked), and its full stack trace. It's invaluable for diagnosing hangs, deadlocks, and CPU spikes, since it shows you exactly where each thread is stuck or busy.

**13. How do you take a thread dump from a running Spring Boot application?**
Using `jstack <pid>` from the command line, or by sending a `SIGQUIT` signal to the process (which prints the dump to its standard output/logs), or through a monitoring tool/APM that captures it for you automatically on demand.

**14. A specific endpoint has suddenly become very slow, but the rest of the app is fine. How do you narrow it down?**
Check if this endpoint recently changed (new deployment, new query), look at its actual database queries with query logging or an APM tool to see if a query is slow or a new N+1 issue was introduced, and check if the traffic pattern to that specific endpoint changed (much higher volume than before).

**15. What tools would you use to profile a Spring Boot application's performance in production?**
APM tools like New Relic, Datadog, or Dynatrace give you method-level timing breakdowns without needing to attach a heavy profiler directly. For lighter-weight, on-demand diagnostics, Java Flight Recorder (JFR) can be enabled with minimal overhead even in production.

**16. What is Java Flight Recorder (JFR), and why is it safe to use in production?**
It's a built-in JVM profiling tool designed with very low overhead specifically so it can be safely run continuously or on-demand in production, unlike older, heavier profilers — it captures detailed data on CPU usage, garbage collection, thread activity, and more, which you can analyze afterward.

**17. High CPU usage turns out to be caused by excessive garbage collection. What does that tell you, and what would you check next?**
It usually means the application is creating far too many objects, or the heap is undersized for the actual workload, forcing the GC to run constantly trying to reclaim space. You'd check heap usage graphs over time, GC logs to see collection frequency and duration, and look for recent code changes that might be creating unnecessary short-lived objects at scale.

**18. What is the difference between CPU-bound and I/O-bound performance issues, and why does the distinction matter?**
A CPU-bound issue means the CPU itself is the bottleneck — heavy computation, inefficient algorithms, excessive GC. An I/O-bound issue means threads are mostly waiting on something external — a slow database query, a slow downstream API call. The fixes are completely different: CPU-bound issues need code/algorithm optimization, while I/O-bound issues need faster queries, caching, connection pool tuning, or timeouts.

**19. How would you investigate whether a slow database query is causing overall application slowness?**
Enable slow query logging on the database (or use `spring.jpa.show-sql` combined with query timing in a non-production-mirrored environment), check the database's own slow query log, and correlate application response times with database query execution times using an APM tool's distributed tracing.

**20. What would you check if response times are fine most of the time but spike periodically at regular intervals?**
This pattern strongly suggests a scheduled job — check for `@Scheduled` tasks, cron jobs, or batch processes running around those times that might be competing for CPU, database connections, or locking rows/tables that live traffic also needs.

---

## Section 3: Memory Leaks & OutOfMemoryError

**21. The application crashes with `OutOfMemoryError: Java heap space`. What's your approach?**
First, capture a heap dump if one wasn't automatically generated (many production setups configure `-XX:+HeapDumpOnOutOfMemoryError` to do this automatically). Then analyze the heap dump with a tool like Eclipse MAT or VisualVM to see what's actually consuming the memory — usually it points straight to a specific class or collection holding far more objects than expected.

**22. What is a heap dump, and how do you generate one?**
It's a complete snapshot of every object currently in the JVM's heap memory at a given moment. You generate one manually with `jmap -dump:live,format=b,file=heap.hprof <pid>`, or automatically on OOM by adding the `-XX:+HeapDumpOnOutOfMemoryError` JVM flag, which is a standard practice to always have enabled in production.

**23. What are common causes of memory leaks in a Spring Boot application?**
Static collections (like a cache) that keep growing without any eviction, unclosed resources (database connections, file streams, HTTP clients not properly closed), listeners/callbacks that are registered but never unregistered, and ThreadLocal variables that aren't cleared, especially problematic with thread pools where threads are reused.

**24. How would you find what's causing a memory leak using a heap dump?**
Open the heap dump in a tool like Eclipse MAT, and look at the "dominator tree" or "leak suspects" report — it highlights which objects are retaining the most memory. From there, you trace back to see what's holding references to those objects (the "path to GC roots"), which usually reveals the actual leaking code.

**25. What is the difference between `OutOfMemoryError: Java heap space` and `OutOfMemoryError: Metaspace`?**
Heap space errors mean your actual application objects (business data, cached items) have exhausted the memory allocated for regular objects. Metaspace errors mean there's not enough memory for class metadata — often caused by dynamically loading too many classes at runtime, which is more common with frameworks doing heavy proxy/bytecode generation, or a classloader leak from repeatedly redeploying without proper cleanup.

**26. What would cause a memory leak specifically related to `ThreadLocal` usage in a Spring Boot app running with a thread pool?**
If a `ThreadLocal` value is set during a request but never explicitly cleared afterward, and the underlying thread is returned to a pool and reused for a later, unrelated request, that old data can leak into the next request's processing — and worse, since pooled threads live indefinitely, anything left in `ThreadLocal` accumulates and is never garbage collected as long as the thread stays alive.

**27. How do you monitor memory usage trends over time to catch a slow memory leak before it causes an outage?**
By tracking heap usage metrics (exposed via Actuator/Micrometer) in a dashboard over days/weeks — a genuine leak shows a steadily climbing baseline memory usage even after garbage collection runs, rather than memory that goes up and down normally with traffic and then returns to a stable baseline.

**28. What is the difference between a memory leak and simply needing more heap memory?**
A memory leak means memory usage keeps growing indefinitely regardless of how much heap you allocate — eventually it'll run out no matter what, because objects are never being released. Needing more heap simply means the application's genuine, stable working set of data is larger than what's currently allocated — increasing heap size actually fixes that, but wouldn't fix a real leak, just delay the eventual crash.

**29. What steps would you take to prevent a similar OOM incident from happening again after root-causing one?**
Fix the actual leaking code (add proper cleanup, bounded caches with eviction, close resources correctly), add memory usage alerts that trigger well before hitting the actual limit, and consider adding automated heap dump generation on OOM if it wasn't already configured, so future incidents are easier to diagnose quickly.

**30. Would you recommend simply restarting the application periodically (like nightly) as a "fix" for a memory leak?**
It can be a reasonable short-term mitigation to keep the system stable while you investigate the real cause, but it should never be treated as an actual fix — it just delays the crash and masks the underlying problem, which will likely resurface in a worse way eventually, especially if traffic grows.

---

## Section 4: Thread Dumps, Deadlocks & Hangs

**31. The application seems "stuck" — requests come in but nothing responds, and CPU usage is low. What do you suspect?**
This pattern strongly suggests a deadlock, thread starvation, or all available threads being blocked waiting on something (like a hung external call with no timeout) — since CPU usage is low, it's not busy computing anything, it's just waiting. A thread dump is the first thing to pull in this situation.

**32. How do you identify a deadlock from a thread dump?**
Modern JVM thread dumps (via `jstack`) actually detect and explicitly report deadlocks at the bottom of the output, showing exactly which threads are involved and which locks they're each waiting on that the other holds — you don't have to manually trace it yourself in most cases.

**33. What is thread starvation, and how is it different from a deadlock?**
A deadlock is threads permanently blocked waiting on each other in a cycle. Thread starvation is when threads are technically able to run, but there simply aren't enough available threads in a pool to handle the current load, so requests queue up and time out — often caused by a pool size that's too small, or other threads holding onto resources for far longer than they should (like a hung downstream call with no timeout).

**34. All your application threads are stuck in "WAITING" state on a database call. What would you check?**
Check the database itself — is it under heavy load, has a long-running query locked a table other queries need, or is the connection pool exhausted so requests are waiting for a connection that never frees up? Also check if there's a timeout configured on those database calls at all — if there isn't, a single slow query can hang threads indefinitely instead of failing fast.

**35. Why is it dangerous to not set timeouts on external calls (HTTP calls to other services, database queries)?**
Without a timeout, a single slow or hung downstream dependency can cause your own threads to block indefinitely waiting for a response — and since threads are a limited resource, enough of these hanging calls can exhaust your entire thread pool, making your whole application unresponsive even though the actual root cause is external.

**36. What is the difference between a thread being in `BLOCKED` state vs. `WAITING` state in a thread dump?**
`BLOCKED` means the thread is waiting to acquire a lock (like a `synchronized` block) that another thread currently holds. `WAITING` (or `TIMED_WAITING`) generally means the thread is waiting for some condition or event to occur — like waiting on a `Future.get()`, a database response, or a `Thread.sleep()` — not necessarily contending over a lock with another thread.

**37. How would you take multiple thread dumps to diagnose an intermittent hang, and why take more than one?**
Take several thread dumps a few seconds apart during the hang. If the exact same threads are stuck at the exact same point in every dump, that's a strong signal of a genuine deadlock or permanent block. If the "stuck" threads are actually slowly progressing between dumps, it might just be a very slow operation rather than a true hang.

**38. What is a livelock, and how is it different from a deadlock?**
In a deadlock, threads are stuck and doing nothing at all, permanently blocked. In a livelock, threads are actively running and responding to each other, but they keep changing state in a way that never actually makes real progress — like two people repeatedly stepping aside for each other in a hallway and never getting past one another.

**39. What would you check if you suspect a synchronized block is causing a bottleneck under high load?**
Look at thread dumps during high load to see how many threads are `BLOCKED` waiting to enter that same synchronized section — if a large number are piled up there, that lock is your bottleneck. The fix usually involves reducing what's inside the critical section, using a more granular/different locking strategy, or replacing it with a non-blocking concurrent data structure if applicable.

**40. Your application hangs specifically under load testing but works fine with normal traffic. What would you suspect first?**
Suspect resource exhaustion under concurrency — thread pool size too small for the load, database connection pool too small, or a lock/synchronized section that becomes a serious bottleneck only when many threads are hitting it simultaneously, which wouldn't show up at all under light, normal traffic.

---

## Section 5: Database & Connection Pool Issues

**41. Your logs show "Connection is not available, request timed out" from HikariCP. What does this mean and how do you fix it?**
It means all connections in your database connection pool are currently in use, and a new request timed out waiting for one to free up. You'd check if the pool size is genuinely too small for your traffic, or — more often — if something is holding connections too long (slow queries, a transaction that's not being closed properly, or a connection leak from code not releasing connections back to the pool).

**42. How would you diagnose a database connection leak?**
Monitor the connection pool's active vs. idle connection counts over time — a genuine leak shows the active count steadily climbing and never coming back down, even during low traffic periods. HikariCP also has leak detection you can enable (`leak-detection-threshold`), which logs a warning with a stack trace showing exactly where a connection was checked out but never returned within a configured time.

**43. What is connection pool sizing, and how would you decide on the right pool size?**
It's deciding how many database connections your application keeps open and ready to use at once. Too small causes requests to queue/time out waiting for a free connection under load; too large can overwhelm the database itself, since it also has its own connection limits, and unnecessarily held connections waste database resources. Sizing is generally based on actual load testing rather than guesswork, and the database's own max connection limit needs to be considered too, especially if multiple app instances share it.

**44. A specific query has become slow only in production, but runs fine in your local/dev environment. What would you investigate?**
Production data volume is almost always the difference — a query that's fast on a small dev dataset can be very slow on millions of production rows, especially if it's missing an index. You'd check the query's execution plan on production data specifically, and look for missing indexes on the columns being filtered/joined.

**45. How do you check if a slow query is missing an index?**
By running an `EXPLAIN` (or `EXPLAIN ANALYZE`) on the query in the actual database, which shows whether it's doing a full table scan (bad, especially on large tables) versus using an index efficiently — a full scan on a large table is usually the clearest sign a helpful index is missing.

**46. The database itself seems fine, but your application is still timing out on database calls. What else could be the cause?**
Network latency or packet loss between the application and the database, the connection pool being exhausted on the application side even though the database itself has capacity, or a firewall/security group misconfiguration intermittently dropping connections — the database's own health doesn't rule out issues in the path between it and your app.

**47. How would you handle a situation where a batch job is locking rows that live user traffic also needs?**
Investigate whether the batch job can process in smaller chunks/batches with brief pauses instead of one giant transaction holding locks for a long time, schedule it during genuinely low-traffic hours if possible, and consider whether it truly needs the isolation level it's currently using, or if a less strict one would reduce lock contention.

**48. What would you check if you suspect the database itself (not your application) is the bottleneck under high load?**
Check the database server's own CPU, memory, and disk I/O metrics, look at its slow query log for queries taking unusually long, and check for lock contention or blocking queries directly on the database side using its own diagnostic tools (like `pg_stat_activity` in PostgreSQL, or the equivalent for your specific database).

**49. How would you safely add an index to a large production table without causing downtime?**
Use the database's online/concurrent index creation feature if available (like `CREATE INDEX CONCURRENTLY` in PostgreSQL), which builds the index without locking the table for reads/writes, rather than a plain `CREATE INDEX` which can lock the table and block traffic for the duration of index creation on a large table.

**50. What would cause a sudden spike in database connections right after a deployment?**
A common cause is the new instance(s) starting up and initializing their connection pools at the same time the old instances are still running during a rolling deployment, briefly doubling the total connections in use — or a misconfiguration in the new version that increased the configured pool size unintentionally.

---

## Section 6: Slow APIs & Latency Debugging

**51. A specific API endpoint is slow, but you're not sure if it's your code, the database, or a downstream service. How do you isolate it?**
Use distributed tracing (via an APM tool or Micrometer Tracing) to break down the total request time into its individual segments — how long was spent in your own code versus waiting on the database versus waiting on an external API call — this tells you exactly where the time is actually going instead of guessing.

**52. What is p99 latency, and why do teams care about it more than average latency?**
p99 latency means 99% of requests complete faster than this value — it captures the worst-case experience for a meaningful chunk of your users. Average latency can look perfectly fine even while a significant number of real users are having a genuinely bad, slow experience, since a few very fast requests can hide a longer tail of slow ones in the average.

**53. Your average response time looks fine, but users are complaining about slowness. What might explain this gap?**
The average is likely being pulled down by a large number of very fast requests, masking a real subset of slow ones (a long tail) — you should look at percentile metrics (p95, p99) rather than the average, since that's what would actually reveal the experience of the affected users complaining.

**54. How would you investigate a slow API that's calling multiple downstream services sequentially?**
Check whether those calls genuinely need to happen one after another, or whether they're actually independent and could run in parallel (using something like `CompletableFuture` to call them concurrently) — sequential calls to independent services is a very common, easily fixed cause of unnecessarily slow response times.

**55. What is a timeout, and why should every external call in your code have one explicitly configured?**
A timeout defines the maximum time you're willing to wait for a response before giving up and failing that specific call. Without one, a single slow or hung dependency can hold your thread indefinitely, which can cascade into exhausting your whole thread pool and taking down your entire application because of just one slow dependency.

**56. What is the Circuit Breaker pattern, and how does it help with cascading slowness across microservices?**
It monitors calls to a downstream service, and if failures/timeouts cross a threshold, it "opens" and stops making the real call for a period, failing fast (or returning a fallback) instead — this prevents your service from continuing to pile up slow, doomed calls to an already-struggling downstream service, which would otherwise make both services worse.

**57. How would you use caching to fix a slow endpoint that reads mostly unchanging data?**
Identify what data is read frequently but changes rarely, and cache it (in-memory or via Redis for multi-instance consistency) so repeated requests hit the fast cache instead of the slower database/API every single time — with an appropriate expiration or invalidation strategy so the cache doesn't serve stale data indefinitely.

**58. What would you check if latency is fine for most requests but a small subset take extremely long (a "long tail")?**
Look for a pattern in what's different about those slow requests specifically — a particular user with unusually large data, a specific query parameter causing an inefficient database path, or contention on a specific resource (like a lock) that only certain requests happen to hit.

**59. How would you determine if slowness is caused by garbage collection pauses rather than your actual application logic?**
Check GC logs and correlate GC pause events with the timing of slow requests — if slow requests consistently line up with when a GC pause occurred, the actual application logic isn't the problem; you'd then look at reducing object allocation, tuning heap size, or trying a different garbage collector suited to your workload.

**60. What role does connection keep-alive and connection pooling play in API latency, specifically for calls to other services?**
Without connection reuse, every single outgoing HTTP call has to pay the cost of establishing a brand-new TCP (and TLS, if HTTPS) connection from scratch, which adds real, avoidable latency. Using a properly configured `WebClient`/`RestTemplate` with connection pooling reuses existing connections for subsequent calls, cutting that overhead significantly for services you call frequently.

---

## Section 7: Logging, Monitoring & Alerting

**61. What's the difference between monitoring and logging, and why do you need both?**
Monitoring gives you aggregate, real-time metrics (CPU, memory, request rate, error rate) that tell you something is wrong and roughly how bad it is. Logging gives you the detailed, specific narrative of what actually happened in a particular request or process — you typically notice a problem through monitoring/alerts, then dig into logs to actually understand and fix it.

**62. What would you look for first in the logs when investigating a production incident?**
Start by narrowing the time window to right around when the issue started, then look for the first unusual entry (not necessarily the most recent error) — often, the earliest sign of trouble (a slow query warning, a connection pool warning) precedes the actual visible failure by some time and points more directly at the root cause.

**63. Why is a correlation/trace ID important when debugging an issue across microservices?**
A single user request often flows through multiple services — without a shared correlation ID attached to every log line for that request across all services, you have no reliable way to piece together the full journey and figure out exactly where in that chain something went wrong.

**64. What would you do if the logs for the time of the incident are missing or incomplete?**
Check if the logging level was set too high (like only logging ERROR, missing useful WARN/INFO context) or if logs were rotated/lost before you could review them — going forward, this is a signal to improve logging retention, verbosity for key events, and ensure critical context is always captured even under normal operation.

**65. How would you set up an alert to catch a memory leak before it causes an outage, rather than finding out after the crash?**
Set an alert on heap usage trending upward over a sustained period (not just a single spike, which could be a normal traffic burst), or an alert when memory usage stays above a high threshold for an extended time even after garbage collection cycles, rather than only alerting on an actual OOM crash after the fact.

**66. What is the danger of alert fatigue, and how would you avoid it?**
If a team receives too many low-priority or frequently false alerts, they start ignoring or being slower to respond to all alerts, including genuinely critical ones. You avoid it by tuning alert thresholds carefully to reduce noise, only alerting on things that genuinely require action, and routing different severities to different channels/urgency.

**67. What metrics would you want visible on a dashboard for a Spring Boot production service?**
Request rate and error rate, response time percentiles (especially p95/p99), CPU and memory usage, database connection pool usage, JVM garbage collection stats, and the health of any critical downstream dependencies — giving a quick, at-a-glance picture of overall system health.

**68. How would you debug an issue that only happens in production and can't be reproduced locally?**
Rely heavily on production logs, metrics, and traces from the actual incident rather than trying to force a local reproduction — check for production-specific differences (data volume, concurrent load, specific configuration, external dependencies not present locally) that could explain why it doesn't show up in a smaller, simpler local environment.

**69. What is log correlation, and how would you achieve it across a request that touches multiple microservices?**
It's tying together all log lines related to one single logical request, even as it passes through multiple separate services — typically done by generating a unique trace ID at the very first entry point (like the API Gateway), and propagating that same ID through every subsequent service call via a header, with every service including it in its own logs (often automated via a tracing library or MDC).

**70. Why should you avoid logging sensitive data (passwords, tokens, card numbers), and what would you do if you discovered it was accidentally happening?**
Logs are often stored, backed up, and viewed by more people/systems than the application itself, and are a common target/leak point for a security breach — plain-text sensitive data in logs can turn a minor bug into a serious compliance and security incident. If found, you'd immediately fix the logging code to mask/exclude that data, and separately assess whether existing logs containing it need to be purged or rotated out securely.

---

## Section 8: Deployment, Rollback & Configuration Issues

**71. A new deployment goes out and error rates spike immediately. What's your immediate action?**
Roll back to the previous stable version right away to restore service, rather than trying to debug and patch forward under pressure — investigate the actual root cause afterward, calmly, once the immediate impact on users is resolved.

**72. What is a canary deployment, and how does it help catch issues before they affect all users?**
It means rolling out a new version to a small subset of traffic/instances first, monitoring it closely for errors or performance issues, and only gradually rolling it out to everyone if it looks healthy — this limits the "blast radius" of a bad deployment to a small fraction of users instead of everyone at once.

**73. What is the danger of deploying a configuration change without going through the same review/testing process as a code change?**
Configuration changes (like a database URL, a feature flag, a connection pool size) can just as easily break production as a code bug can, but they're often treated more casually and pushed without the same scrutiny, testing, or rollback plan — leading to avoidable incidents that "weren't even a code change."

**74. After a deployment, the new version starts but immediately fails all health checks. What would you check?**
Check the exact error in the startup/health check logs first — often it's a missing or changed environment variable/config value between environments, a database migration that hasn't been applied yet that the new code expects, or a dependency/service the new version needs that isn't available yet.

**75. What is a blue-green deployment, and how does it make rollback fast and safe?**
You keep two full production environments — the currently live one ("blue") and the new one being deployed ("green") — and only switch traffic over to green once it's verified healthy. If something goes wrong, rollback is just switching traffic back to blue, which is nearly instant, rather than having to redeploy the old version from scratch.

**76. Your application works fine in staging but fails in production after deployment. What are common reasons for this?**
Differences in configuration (database credentials, external service URLs, feature flags) between environments, real production data triggering an edge case that staging's smaller/cleaner test data never hit, or differences in scale/load that staging simply doesn't replicate.

**77. What is a feature flag, and how does it help reduce risk during deployments?**
It's a toggle that lets you turn a new feature on or off (often per user segment, or gradually) without needing to redeploy code — so if a new feature causes issues, you can instantly disable it in production without a full rollback, and you can also gradually ramp up exposure instead of enabling it for everyone at once.

**78. How would you handle a situation where a required database migration wasn't applied before deploying new code that depends on it?**
The new code would likely fail immediately trying to query a column/table that doesn't exist yet — the fix is ensuring migrations are always applied as a required, automated step before the new application code starts (tools like Flyway/Liquibase can be configured to run automatically at startup, and deployment pipelines should sequence this correctly).

**79. What would you check if a deployment succeeds but the application seems to be running the old code/behavior?**
Check if the deployment pipeline actually deployed the new artifact to the right instances (sometimes only some instances get updated during a rolling deployment, and you're hitting an old one), verify the build actually included your latest changes, and rule out caching (browser cache, CDN cache, or an application-level cache) serving stale responses that look like old behavior.

**80. What's the value of having a documented rollback plan before every deployment, rather than figuring it out during an incident?**
Under the stress of an active incident, people make mistakes and waste critical time figuring out the "how" of rolling back. Having it pre-planned and even automated (a single button/command to revert) means the actual rollback can happen in minutes rather than requiring careful, error-prone manual steps while users are actively affected.

---

## Section 9: Kafka / Messaging in Production

**81. Messages are piling up in a Kafka topic and consumers aren't keeping up. What would you check?**
Check if consumers are actually running and healthy, look at consumer lag metrics to see exactly how far behind they are, and check if a specific consumer instance is stuck/slow (perhaps due to a poison message or an external call it's waiting on) rather than all consumers being generally overwhelmed.

**82. What is consumer lag, and why is it an important metric to monitor?**
It's the difference between the latest message produced to a topic and the last message a consumer group has actually processed — growing lag means consumers can't keep up with the rate of incoming messages, and if left unchecked, it means increasingly stale/delayed processing, which can cascade into other problems downstream.

**83. What is a "poison message," and how would you handle one that's repeatedly crashing your consumer?**
It's a specific message that consistently fails processing no matter how many times it's retried — often due to malformed data or an edge case the code doesn't handle. You'd configure a dead-letter topic so that after a limited number of retries, the message is moved aside instead of blocking the entire consumer from processing everything after it, then investigate and fix that specific message/edge case separately.

**84. How would you scale out Kafka consumers to handle increased load?**
Increase the number of consumer instances within the same consumer group, up to the number of partitions the topic has (since each partition can only be actively consumed by one consumer in a group at a time) — if you already have as many consumers as partitions, you'd need to increase the topic's partition count to allow further parallelism.

**85. What would you do if a producer is failing to send messages to Kafka?**
Check connectivity to the Kafka brokers first, check if the specific topic exists and is healthy (not under-replicated or offline), and look at the actual exception being thrown by the producer — common causes include network issues, authentication/authorization misconfiguration, or the topic simply not existing yet.

**86. How do you ensure a Kafka consumer doesn't process the same message twice in a way that causes real business problems (like double-charging a customer)?**
Design consumer logic to be idempotent — for example, checking if a given transaction ID has already been processed before actually applying its effect, so that even if the same message is delivered more than once (which can happen with at-least-once delivery), reprocessing it has no harmful additional effect.

**87. What would you check if messages seem to be processed out of order when they shouldn't be?**
Check whether related messages (like all events for the same order) are actually being sent with the same partition key — Kafka only guarantees ordering within a single partition, so if related messages land in different partitions, there's no ordering guarantee between them at all.

**88. A Kafka consumer keeps rebalancing repeatedly, disrupting processing. What might be causing this?**
Common causes: the consumer taking too long to process a batch of messages (exceeding `max.poll.interval.ms`, making Kafka think it's dead and triggering a rebalance), network instability between the consumer and the broker, or consumer instances being restarted/scaled frequently — each rebalance briefly pauses processing across the whole group while partitions are reassigned.

---

## Section 10: Security Incidents in Production

**89. You detect unusual, high-volume traffic hitting your login endpoint. What do you suspect, and what would you do?**
This looks like a brute-force or credential-stuffing attack. You'd implement or verify rate limiting on that endpoint specifically, consider temporarily blocking the source IP(s) if identifiable, and check if any accounts show signs of actual successful unauthorized access as a result.

**90. How would you respond if you discovered an API key or database password was accidentally committed to a public source code repository?**
Immediately revoke/rotate the exposed credential — treat it as compromised the moment it's public, regardless of how quickly it's removed from the repo, since it may already have been scraped or cached elsewhere. Then investigate whether it was actually misused, and put a preventive measure in place (like secret-scanning in the CI pipeline) to catch this before it happens again.

**91. A JWT secret used to sign tokens may have been compromised. What's the impact, and what would you do?**
Anyone with the secret can forge valid tokens claiming to be any user, including an admin — this is a serious incident. You'd rotate the signing secret immediately (which invalidates all previously issued tokens, forcing everyone to log in again), and investigate logs for any signs the compromised secret was actually used to forge access before rotation.

**92. How would you detect and respond to an unusually high rate of 401/403 errors in your API logs?**
A spike in unauthorized/forbidden responses can indicate someone probing your API with invalid or stolen credentials, or scanning for accessible endpoints — you'd look at the source IPs and patterns involved, consider rate limiting or temporarily blocking clearly malicious sources, and verify no legitimate users are being incorrectly denied access due to a bug rather than an actual attack.

**93. What would you check if you suspect an SQL injection vulnerability might have been exploited in production?**
Check application logs and database query logs around the suspected time for unusual, malformed, or suspicious query patterns, verify the affected endpoint is actually using parameterized queries/prepared statements properly (rather than raw string concatenation, which is the actual root vulnerability), and assess what data might have been exposed if the exploit succeeded.

**94. How would you handle discovering that sensitive data was accidentally exposed in an API response (like returning full card numbers)?**
Immediately fix the code to stop exposing that data (patch and deploy as a priority), assess how long the exposure existed and estimate what data might have already been accessed by whom, and follow your organization's incident response/compliance process for reporting a data exposure, since this often has legal/regulatory obligations beyond just the technical fix.

---

## Section 11: Scaling & Load Issues

**95. Traffic to your application has doubled overnight and it's struggling to keep up. What do you check first?**
Check which resource is actually the bottleneck first — CPU, memory, database connections, or thread pool exhaustion — rather than blindly scaling everything, since scaling the wrong resource wastes time and money without fixing the real constraint.

**96. What is the difference between scaling up and scaling out, and when would you choose one over the other?**
Scaling up means making a single instance more powerful (more CPU/RAM). Scaling out means adding more instances of the same application behind a load balancer. Scaling out is generally preferred for stateless Spring Boot services since it also adds redundancy, but scaling up can be a faster short-term fix if a single specific resource (like memory) is clearly the immediate constraint.

**97. Your application scales out fine, but the database becomes the bottleneck under high load. What are your options?**
Add read replicas to offload read-heavy queries away from the primary database, introduce caching for frequently read data to reduce direct database load, optimize slow queries/add missing indexes, and if the write load itself is the true bottleneck, consider partitioning/sharding the data, though that's a bigger architectural change.

**98. What is autoscaling, and what metric would you typically use to trigger it for a Spring Boot service?**
It's automatically adjusting the number of running instances based on current demand, rather than a fixed, manually-set count. Common trigger metrics include CPU utilization, memory usage, or request queue length/latency — CPU-based autoscaling is the most common starting point, though it's not always the right signal depending on your actual bottleneck.

**99. What would you check if autoscaling adds new instances but the overall system still doesn't improve?**
Check if the actual bottleneck is somewhere the new instances can't help with at all — like the database or a downstream service that's shared across all instances — adding more application instances doesn't help if they're all still waiting on the same overloaded shared dependency.

**100. What is a thundering herd problem, and how might it show up during a scaling event or after a restart?**
It's when a large number of clients/instances all hit a shared resource (like a cache that just expired, or a database) at exactly the same moment, overwhelming it — for example, if many new instances start up simultaneously and all try to warm up their own local caches by hitting the database at once, potentially overwhelming it right when you need it healthy the most.

---

## Section 12: Real-World Scenario & Judgment Questions

**101. A critical bug is found in production, but the proper fix will take a day to develop and test properly. What do you do in the meantime?**
Assess the actual severity and blast radius first — if it's causing real user/business harm right now, consider a quick, safe mitigation (disabling the specific broken feature via a flag, rolling back to before it was introduced, or a narrow, well-tested hotfix) while the proper, complete fix is developed and tested proprely without cutting corners under pressure.

**102. How would you decide whether an incident needs to page someone at 2 AM versus waiting until morning?**
Base it on actual user/business impact and urgency — is real money being lost, are customers actively unable to use a critical function, is data at risk — versus something that's degraded but has a workaround, or affects a very small percentage of low-priority traffic that can reasonably wait a few hours without meaningful harm.

**103. After resolving a production incident, what should happen next?**
A blameless post-mortem/root cause analysis — documenting what happened, why, how it was detected and resolved, and concrete action items to prevent it (or catch it faster) next time — focused on improving the system and process, not on assigning individual blame.

**104. What is the difference between a symptom and a root cause, and why does that distinction matter during an incident?**
A symptom is what you directly observe (high CPU, slow responses, errors) — the root cause is the actual underlying reason those symptoms are happening (a specific bug, a missing index, a resource leak). Fixing only the symptom (like just restarting the app) without finding the root cause means the same issue is very likely to recur later.

**105. How would you balance restoring service quickly versus fully understanding the root cause during an active incident?**
Prioritize restoring service first if there's real, ongoing user impact — a rollback or quick mitigation buys time and stops the bleeding — then investigate the actual root cause calmly afterward, when there's no pressure of an ongoing outage, so you don't rush into a wrong conclusion or a worse fix.

**106. A teammate deployed a change that caused an incident. How would you handle that conversation afterward?**
Focus entirely on the system and process — what allowed this change to reach production without being caught (missing tests, no canary rollout, no review of that specific risk) — rather than blaming the individual, since anyone could have made a similar mistake, and a blame-focused culture just makes people afraid to admit issues or ask for help in the future.

**107. How would you communicate an ongoing production incident to non-technical stakeholders?**
Keep it clear, honest, and focused on user/business impact rather than deep technical jargon — what's affected, roughly how many users/what functionality, what you're doing about it, and a realistic estimate (or honest "we don't know yet") for resolution — with regular, proactive updates rather than making them ask for status repeatedly.

**108. What would you do differently in your development process after experiencing a production incident caused by insufficient testing?**
Identify specifically what kind of test would have actually caught this issue (a missing edge case, a load/performance test, an integration test between two specific services) and add that category of testing going forward — a blanket "write more tests" isn't as useful as identifying the specific gap that let this particular issue slip through.

**109. How do you decide what should be automated (like automatic rollback or automatic scaling) versus what should require human judgment during an incident?**
Automate well-understood, low-risk, clearly-defined responses to known failure patterns (like automatically restarting a genuinely crashed instance, or auto-scaling based on clear metrics) — but keep human judgment in the loop for ambiguous, high-stakes, or novel situations where an automated action could make a bad situation worse if the assumption behind it turns out to be wrong.

**110. What's a practical way to reduce how often "the app is down at 2 AM" actually happens, beyond just fixing bugs as they come up?**
Invest in better observability (so issues are caught and understood before they become full outages), stress/load test realistic production-like scenarios before they happen live, build in resilience patterns (timeouts, circuit breakers, retries with backoff) so individual dependency failures don't cascade into full outages, and maintain a genuine blameless post-mortem culture so lessons from past incidents actually get acted on.

---

## How to use this document
- These are judgment and process questions as much as technical ones — interviewers are listening for a calm, structured approach (triage → mitigate → investigate → fix → prevent), not just the right buzzwords.
- Whenever you can, replace the general answer with a real, specific story from your own experience at Wipro — "here's an actual production issue I debugged and how I approached it" is far more convincing in a senior-leaning interview than a textbook description of the process.
- Practice explaining your troubleshooting steps out loud in order — interviewers are often testing whether you have a repeatable, level-headed method, not just whether you know the right terms.
