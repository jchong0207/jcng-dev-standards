# Programming Specification — Concurrency

Verbatim excerpts from https://alibaba.github.io/Alibaba-Java-Coding-Guidelines/#concurrency

This is the highest-risk section. Every Mandatory rule here protects against bugs that are hard to reproduce and harder to debug in production.

---

**1. [Mandatory]** Thread-safe should be ensured when initializing singleton instance, as well as all methods in it.

> Note: Resource driven class, utility class and singleton factory class are all included.

**2. [Mandatory]** A meaningful thread name is helpful to trace the error information, so assign a name when creating threads or thread pools.

> Positive example:
> ```java
> public class TimerTaskThread extends Thread {
>     public TimerTaskThread() {
>         super.setName("TimerTaskThread");
>         // ...
>     }
> }
> ```
>
> With a pool, use a `ThreadFactory` that assigns a name pattern:
> ```java
> ThreadFactory tf = new ThreadFactoryBuilder().setNameFormat("payment-worker-%d").build();
> ```

**3. [Mandatory]** Threads should be provided by thread pools. Explicitly creating threads is not allowed.

> Note: Using a thread pool can reduce the time of creating and destroying threads and save system resources. Without pools, lots of similar threads will be created leading to "running out of memory" or over-switching problems.

**4. [Mandatory]** A thread pool should be created by `ThreadPoolExecutor` rather than `Executors`. This makes the parameters of the thread pool understandable and reduces the risk of running out of system resources.

> Note — problems created by `Executors`:
> 1. `FixedThreadPool` and `SingleThreadPool`: maximum request queue size is `Integer.MAX_VALUE`. A large number of requests might cause OOM.
> 2. `CachedThreadPool` and `ScheduledThreadPool`: the number of threads allowed to be created is `Integer.MAX_VALUE`. Creating too many threads might lead to OOM.
>
> Positive example:
> ```java
> ExecutorService pool = new ThreadPoolExecutor(
>     8, 8, 60L, TimeUnit.SECONDS,
>     new ArrayBlockingQueue<>(1000),
>     namedFactory,
>     new ThreadPoolExecutor.CallerRunsPolicy()
> );
> ```

**5. [Mandatory]** `SimpleDateFormat` is unsafe; do not define it as a static variable. If have to, lock or `DateUtils` class must be used.

> Positive example — `ThreadLocal` wrapping (use this only if you cannot move to JDK 8 `DateTimeFormatter`):
> ```java
> private static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>() {
>     @Override
>     protected DateFormat initialValue() {
>         return new SimpleDateFormat("yyyy-MM-dd");
>     }
> };
> ```
>
> Note: In JDK 8, `Instant` can replace `Date`, `LocalDateTime` replaces `Calendar`, `DateTimeFormatter` replaces `SimpleDateFormat`. `DateTimeFormatter` is immutable and thread-safe — prefer it.

**6. [Mandatory]** `remove()` method must be implemented by `ThreadLocal` variables, especially when using thread pools in which threads are often reused. Otherwise, it may affect subsequent business logic and cause unexpected problems such as memory leak.

> Positive example:
> ```java
> objectThreadLocal.set(someObject);
> try {
>     // ...
> } finally {
>     objectThreadLocal.remove();
> }
> ```

**7. [Mandatory]** In highly concurrent scenarios, performance of `Lock` should be considered in synchronous calls. A block lock is better than a method lock. An object lock is better than a class lock.

**8. [Mandatory]** When adding locks to multiple resources, tables in the database, and objects at the same time, locking sequence should be kept consistent to avoid deadlock.

> Note: If thread 1 does update after adding locks to table A, B, C accordingly, the lock sequence of thread 2 should also be A, B, C. Otherwise deadlock might happen.
>
> A common pattern: acquire locks in numeric/alphabetical order — `min(id)` then `max(id)`.

**9. [Mandatory]** When getting a lock by blocking methods (such as waiting in the blocking queue), `lock()` from `Lock` must be put **outside** the `try` block. Besides, make sure no method between `lock()` and the `try` block, in case the lock won't be released in the `finally` block.

> Note 1: If exceptions are thrown between `lock()` and the `try` block, the lock won't be released — other threads can't get it.
>
> Note 2: If `lock()` fails to run successfully and you've already entered the `try` block, `unlock()` may run on a lock you never held. AQS (`AbstractQueuedSynchronizer`) then throws `IllegalMonitorStateException`.
>
> Note 3: It is possible that the `lock()` implementation throws an unchecked exception, resulting in the same outcome as Note 2.
>
> Positive example:
> ```java
> lock.lock();             // OUTSIDE the try
> try {
>     // ...
> } finally {
>     lock.unlock();
> }
> ```

**10. [Mandatory]** A lock needs to be used to avoid update failure when modifying one record concurrently. Add lock either in application layer, in cache, or add optimistic lock in the database by using version as update stamp.

> Note: If access conflict probability is less than 20%, prefer optimistic lock; otherwise use pessimistic lock. Retry count for optimistic lock should be no less than 3.

**11. [Mandatory]** Run multiple `TimerTask` by using `ScheduledExecutorService` rather than `Timer`, because `Timer` will kill all running threads in case of failing to catch exceptions.

**12. [Recommended]** When using `CountDownLatch` to convert asynchronous operations to synchronous ones, each thread must call `countDown` before quitting. Make sure to catch any exception during thread running, to let `countDown` be executed. If the main thread cannot reach `await`, the program will hang until timeout.

> Note: Exceptions thrown by sub-threads cannot be caught by the main thread.
>
> Positive example:
> ```java
> try {
>     // work...
> } catch (Exception e) {
>     log.error("worker failed", e);
> } finally {
>     latch.countDown();
> }
> ```

**13. [Recommended]** Avoid using `Random` instance by multiple threads. Although it is safe to share this instance, competition on the same seed will damage performance.

> Note: `Random` instance includes instances of `java.util.Random` and `Math.random()`.
>
> Positive example: After JDK 7, `ThreadLocalRandom.current().nextInt()` can be used directly. Before JDK 7, create an instance per thread.

**14. [Recommended]** In concurrent scenarios, one easy solution to optimize the lazy initialization problem by using double-checked locking is to declare the object type as `volatile`.

> Counter example (missing `volatile` — another thread may see a partially-constructed `Helper`):
> ```java
> class Foo {
>     private Helper helper = null;
>     public Helper getHelper() {
>         if (helper == null) {
>             synchronized (this) {
>                 if (helper == null) {
>                     helper = new Helper();
>                 }
>             }
>         }
>         return helper;
>     }
> }
> ```
>
> Fix: declare `private volatile Helper helper = null;`

**15. [For Reference]** `volatile` is used to solve the problem of invisible memory in multiple threads. *Write-Once-Read-Many* can solve variable synchronization problems. But *Write-Many* cannot settle thread-safety. For `count++`, use:

> ```java
> AtomicInteger count = new AtomicInteger();
> count.addAndGet(1);
> ```
>
> Note: In JDK 8, `LongAdder` is recommended; it reduces retry times of optimistic locking and has better performance than `AtomicLong` under heavy contention.

**16. [For Reference]** Resizing `HashMap` when its capacity is not enough might cause a dead link and high CPU usage due to high concurrency. Avoid this risk in development.

> Notes: JDK 7 `HashMap` resize can form a cyclic linked list (100% CPU). JDK 8 fixes the cycle but can still drop entries under concurrent writes. Use `ConcurrentHashMap` for concurrent access, or size the `HashMap` up front to avoid resize.

**17. [For Reference]** `ThreadLocal` cannot solve update problems of a shared object. It is recommended to use a `static` `ThreadLocal` object that is shared by all operations in the same thread.
