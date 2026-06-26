# Programming Specification — Flow Control, Comments, Other

Verbatim excerpts from https://alibaba.github.io/Alibaba-Java-Coding-Guidelines/

---

## Flow Control Statements

**1. [Mandatory]** In a `switch` block, each `case` should be finished by `break` or `return`. If not, a comment should be included to describe at which case it will stop. Within every `switch` block, a `default` statement must be present, even if it is empty.

**2. [Mandatory]** Braces are used with `if`, `else`, `for`, `do` and `while` statements, even if the body contains only a single statement.

> Avoid:
> ```java
> if (condition) statements;
> ```

**3. [Recommended]** Use `else` as little as possible. `if-else` statements can be replaced by guard clauses:

> ```java
> if (condition) {
>     // ...
>     return obj;
> }
> // other logic that was in the else block moves here
> ```
>
> Note: If statements like `if() ... else if() ... else ...` must be used to express the logic, **[Mandatory]** nested conditional levels should not be more than three.
>
> Positive example — replace deeply nested `if-else` with guard statements (or with State pattern):
> ```java
> public void today() {
>     if (isBusy()) {
>         System.out.println("Change time.");
>         return;
>     }
>
>     if (isFree()) {
>         System.out.println("Go to travel.");
>         return;
>     }
>
>     System.out.println("Stay at home to learn Alibaba Java Coding Guidelines.");
>     return;
> }
> ```

**4. [Recommended]** Do not use complicated statements in conditional statements (except for frequently used methods like `getXxx`/`isXxx`). Use a `boolean` variable to store the result of a complicated expression temporarily — it increases readability.

> Positive example:
> ```java
> boolean existed = (file.open(fileName, "w") != null) && (...) || (...);
> if (existed) {
>     // ...
> }
> ```
>
> Counter example:
> ```java
> if ((file.open(fileName, "w") != null) && (...) || (...)) {
>     // ...
> }
> ```

**5. [Recommended]** Performance should be considered when loop statements are used. The following operations are better processed **outside** the loop: object/variable declaration, database connection, `try-catch` statements.

**6. [Recommended]** The size of input parameters should be checked, especially for batch operations.

**7. [For Reference]** Input parameters should be checked in the following scenarios:
1. Low-frequency methods.
2. Long-running methods where the cost of validation is negligible relative to the cost of an illegal-parameter failure.
3. Methods that require extremely high stability or availability.
4. Open API methods, including RPC/API/HTTP.
5. Authority-related methods.

**8. [For Reference]** Cases that input parameters do not require validation:
1. Methods very likely to be implemented in loops — note that callers must validate externally.
2. Methods in bottom layers very frequently called (e.g., DAO layer if DAO and Service are co-deployed).
3. `private` methods that can only be implemented internally, if all parameters are checked or manageable.

---

## Code Comments

**1. [Mandatory]** Javadoc should be used for classes, class variables and methods. The format should be `/** comment */`, rather than `// xxx`.

> Note: In IDEs, Javadoc can be seen directly when hovering — a good way to improve efficiency.

**2. [Mandatory]** Abstract methods (including methods in interfaces) should be commented by Javadoc. Javadoc should include method instruction, description of parameters, return values and possible exceptions.

**3. [Mandatory]** Every class should include information of author(s) and date.

**4. [Mandatory]** Single-line comments in a method should be put above the code to be commented, using `//`; multiple lines using `/* */`. Alignment for comments should be noticed carefully.

**5. [Mandatory]** All enumeration type fields should be commented as Javadoc style.

**6. [Recommended]** Local language can be used in comments if English cannot describe the problem properly. Keywords and proper nouns should be kept in English.

> Counter example: To explain "TCP connection overtime" as "Transmission Control Protocol connection overtime" only makes it more difficult to understand.

**7. [Recommended]** When code logic changes, comments need to be updated at the same time, especially for changes of parameters, return values, exceptions and core logic.

**8. [For Reference]** Notes need to be added when commenting out code.

> Note: If the code is likely to be recovered later, a reasonable explanation needs to be added. If not, delete directly because code history will be recorded by SVN or git.

**9. [For Reference]** Requirements for comments:
1. Be able to represent design ideas and code logic accurately.
2. Be able to represent business logic and help other programmers understand quickly.

**10. [For Reference]** Proper naming and clear code structure are self-explanatory. Too many comments need to be avoided because it may cause too much work on updating if code logic changes.

> Counter example:
> ```java
> // put elephant into fridge
> put(elephant, fridge);
> ```

**11. [For Reference]** Tags in comments (e.g. `TODO`, `FIXME`) need to contain author and time. Tags need to be handled and cleared regularly by scanning.

---

## Other

**1. [Mandatory]** When using regex, precompile to increase matching performance.

> Note: Do not define `Pattern pattern = Pattern.compile(...);` within the method body. Hoist it to a `static final` field.

**2. [Mandatory]** When using attributes of POJO in Velocity, use attribute names directly. The Velocity engine will invoke `getXxx()` of POJO automatically. For boolean attributes, Velocity invokes `isXxx()` — so do not use `is` as a prefix when naming boolean attributes.

> Note: For wrapper class `Boolean`, the Velocity engine will invoke `getXxx()` first.

**3. [Mandatory]** Variables must add an exclamation mark when passed to the Velocity engine from the backend, like `$!{var}`.

> Note: If the attribute is `null` or does not exist, `${var}` will be shown literally on the web page.

**4. [Mandatory]** The return type of `Math.random()` is `double`, with range `0 <= x < 1` (0 is possible). If a random integer is required, do not multiply `x` by 10 and round; use `nextInt` or `nextLong` from `Random`.

**5. [Mandatory]** Use `System.currentTimeMillis()` to get the current millisecond. Do not use `new Date().getTime()`.

> Note: To get more accurate time, use `System.nanoTime()`. In JDK 8, use `Instant` for time statistics.

**6. [Recommended]** Better not to contain variable declarations, logical symbols or any complicated logic in Velocity template files.

**7. [Recommended]** Size needs to be specified when initializing any data structure if possible, in order to avoid memory issues caused by unlimited growth.

**8. [Recommended]** Code or configuration noticed to be obsolete should be resolutely removed from projects.

> Note: Remove obsolete code or configuration in time to avoid redundancy. For code which is temporarily removed and likely to be reused, use `///` to add a reasonable note.
>
> Positive example:
> ```java
> public static void hello() {
>     /// Business is stopped temporarily by the owner.
>     // Business business = new Business();
>     // business.active();
>     System.out.println("it's finished");
> }
> ```
