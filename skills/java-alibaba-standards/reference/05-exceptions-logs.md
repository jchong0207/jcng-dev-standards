# Exception & Logs

Verbatim excerpts from https://alibaba.github.io/Alibaba-Java-Coding-Guidelines/

---

## Exception

**1. [Mandatory]** Do not catch *Runtime* exceptions defined in JDK, such as `NullPointerException` and `IndexOutOfBoundsException`. Instead, pre-check is recommended whenever possible.

> Note: Use `try-catch` only if it is difficult to deal with pre-check, such as `NumberFormatException`.
>
> Positive example: `if (obj != null) { ... }`
>
> Counter example: `try { obj.method() } catch (NullPointerException e) { ... }`

**2. [Mandatory]** Never use exceptions for ordinary control flow. It is ineffective and unreadable.

**3. [Mandatory]** It is irresponsible to use a `try-catch` on a big chunk of code. Be clear about the stable and unstable code when using `try-catch`. The stable code means no exception will throw. For the unstable code, catch as specifically as possible.

**4. [Mandatory]** Do not suppress or ignore exceptions. If you do not want to handle it, then re-throw it. The top layer must handle the exception and translate it into what the user can understand.

**5. [Mandatory]** Make sure to invoke the rollback if a method throws an `Exception`.

> Spring note: rollback happens automatically only for unchecked exceptions by default. For checked exceptions, specify `@Transactional(rollbackFor = Exception.class)` or similar.

**6. [Mandatory]** Closeable resources (stream, connection, etc.) must be handled in a `finally` block. Never throw any exception from a `finally` block.

> Note: Use the *try-with-resources* statement to safely handle closeable resources (Java 7+).

**7. [Mandatory]** Never use `return` within a `finally` block. A `return` statement in a `finally` block will cause exceptions or result in a discarded return value in the `try-catch` block.

**8. [Mandatory]** The `Exception` type to be caught needs to be the same class or superclass of the type that has been thrown.

**9. [Recommended]** The return value of a method can be `null`. It is not mandatory to return an empty collection or object. Specify in Javadoc explicitly when the method might return `null`. The caller needs to make a `null` check to prevent `NullPointerException`.

> Note: It is the caller's responsibility to check the return value, as well as to consider the possibility that a remote call fails or another runtime exception occurs.

**10. [Recommended]** One of the most common errors is `NullPointerException`. Pay attention to the following situations:
1. If the return type is primitive, returning a value of wrapper class may cause `NullPointerException`.
   > Counter example: `public int f() { return Integer; }` — unboxing a `null` value will throw `NullPointerException`.
2. The return value of a database query might be `null`.
3. Elements in a collection may be `null`, even though `Collection.isEmpty()` returns `false`.
4. Return values from an RPC might be `null`.
5. Data stored in sessions might be `null`.
6. Method chaining, like `obj.getA().getB().getC()`, is likely to cause `NullPointerException`.

> Positive example: Use `Optional` to avoid null-check and NPE (Java 8+).

**11. [Recommended]** Choose between "throw exception" and "return error code". For HTTP or open-API providers, "error code" must be used. It is recommended to throw exceptions inside an application. For cross-application RPC calls, prefer encapsulating `isSuccess`, `errorCode` and a brief error message into a `Result`.

> Notes:
> 1. Using "throw exception" will cause a runtime error if the exception is not caught.
> 2. Without stack information, custom exceptions with simple error messages are not helpful in solving the problem. With stack information attached, data serialization and transmission cost are also problems when errors occur frequently.

**12. [Recommended]** Do not throw `RuntimeException`, `Exception`, or `Throwable` directly. Use well-defined custom exceptions such as `DAOException`, `ServiceException`, etc.

**13. [For Reference]** Avoid duplicate code (DRY — Do not Repeat Yourself).

> Note: Copy-paste inevitably leads to duplicated code. Keep logic in one place to make changes easier. Extract common code to methods, abstract classes, or shared modules.
>
> Positive example:
> ```java
> private boolean checkParam(DTO dto) {
>     // shared validation
> }
> ```

---

## Logs

**1. [Mandatory]** Do not use the APIs of log systems (Log4j, Logback) directly. Use SLF4J instead — it is a *facade* pattern and keeps log processing consistent.

> ```java
> import org.slf4j.Logger;
> import org.slf4j.LoggerFactory;
> private static final Logger logger = LoggerFactory.getLogger(Abc.class);
> ```

**2. [Mandatory]** Log files need to be kept for at least **15 days** because some kinds of exceptions happen weekly.

**3. [Mandatory]** Naming conventions of extended logs of an application (RBI, temporary monitoring, access log, etc.): `appName_logType_logName.log`
- `logType`: recommended classifications are `stats`, `desc`, `monitor`, `visit`, etc.
- `logName`: log description.

> Positive example: Monitoring the timezone conversion exception in the `mppserver` application:
> `mppserver_monitor_timeZoneConvert.log`
>
> Note: Classify logs. Error logs and business logs should be stored separately as far as possible — easier for developers to view and convenient for system monitoring.

**4. [Mandatory]** Logs at `TRACE` / `DEBUG` / `INFO` levels must use either conditional outputs or placeholders.

> Counter example — string concatenation is evaluated even when the log won't be printed:
> ```java
> logger.debug("Processing trade with id: " + id + " symbol: " + symbol);
> ```
> If the log level is `warn`, the above log won't be printed, but the concatenation still runs and `symbol.toString()` is called — wasting CPU.
>
> Positive example (conditional):
> ```java
> if (logger.isDebugEnabled()) {
>     logger.debug("Processing trade with id: " + id + " symbol: " + symbol);
> }
> ```
>
> Positive example (placeholders — preferred):
> ```java
> logger.debug("Processing trade with id: {} and symbol: {}", id, symbol);
> ```

**5. [Mandatory]** Ensure that the `additivity` attribute of a Log4j logger is set to `false`, to avoid redundancy and save disk space.

> Positive example:
> ```xml
> <logger name="com.taobao.ecrm.member.config" additivity="false">
> ```

**6. [Mandatory]** Exception information should contain two types of information: the context, and the exception stack. If you do not want to handle the exception, re-throw it.

> Positive example:
> ```java
> logger.error("various parameters or objects toString: " + ... + "_" + e.getMessage(), e);
> ```
> Pass the exception object `e` as the **last** argument so SLF4J attaches the stack trace.

**7. [Recommended]** Carefully record logs. Use `INFO` level selectively. Do not use `DEBUG` in production. If `WARN` is used to record business behavior, pay attention to log volume. Make sure server disks are not overfilled, and delete logs in time.

> Note: A large volume of invalid logs hurts performance and makes problems harder to find. Ask: do you really need this log? What will you do when you see it? Does it help troubleshooting?

**8. [Recommended]** Level `WARN` should be used to record invalid parameters, to track data when a problem occurs. Level `ERROR` should only record system logic errors, exceptions, and other important error messages.
