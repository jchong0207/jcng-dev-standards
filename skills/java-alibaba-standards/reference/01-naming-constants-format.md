# Programming Specification — Naming, Constants, Formatting

Verbatim excerpts from https://alibaba.github.io/Alibaba-Java-Coding-Guidelines/

---

## Naming Conventions

**1. [Mandatory]** Names should not start or end with an underline or a dollar sign.

> Counter example: `_name / __name / $Object / name_ / name$ / Object$`

**2. [Mandatory]** Using Chinese, Pinyin, or Pinyin-English mixed spelling in naming is strictly prohibited.

> Positive example: `alibaba / taobao / youku / Hangzhou`. In these cases, Chinese proper names in Pinyin are acceptable.

**3. [Mandatory]** Class names should be nouns in UpperCamelCase except domain models: DO, BO, DTO, VO, etc.

> Positive example: `MarcoPolo / UserDO / HtmlDTO / XmlService / TcpUdpDeal / TaPromotion`
>
> Counter example: `marcoPolo / UserDo / HTMLDto / XMLService / TCPUDPDeal / TAPromotion`

**4. [Mandatory]** Method names, parameter names, member variable names, and local variable names should be written in lowerCamelCase.

> Positive example: `localValue / getHttpMessage() / inputUserId`

**5. [Mandatory]** Constant variable names should be written in upper characters separated by underscores.

> Positive example: `MAX_STOCK_COUNT`
>
> Counter example: `MAX_COUNT`

**6. [Mandatory]** Abstract class names must start with `Abstract` or `Base`. Exception class names must end with `Exception`. Test case names shall start with the class names to be tested and end with `Test`.

**7. [Mandatory]** Brackets are a part of an Array type. The definition could be: `String[] args;`

> Counter example: `String args[];`

**8. [Mandatory]** Do not add `is` as prefix while defining Boolean variable, since it may cause a serialization exception in some Java frameworks.

> Counter example: `boolean isSuccess;`

**9. [Mandatory]** A package should be named in lowercase characters. There should be only one English word after each dot.

> Positive example: `com.alibaba.open.util; MessageUtils`

**10. [Mandatory]** Uncommon abbreviations should be avoided for the sake of legibility.

> Counter example: `AbsClass (AbstractClass); condi (Condition)`

**11. [Recommended]** The pattern name is recommended to be included in the class name if any design pattern is used.

> Positive example: `public class OrderFactory; public class LoginProxy; public class ResourceObserver;`

**12. [Recommended]** Do not add any modifier, including `public`, to methods in interface classes for coding simplicity.

> Positive example: method definition in the interface: `void f();` constant definition: `String COMPANY = "alibaba";`

**13. [Mandatory]** All `Service` and `DAO` classes must be interfaces based on SOA principle. Implementation class names should end with `Impl`.

> Positive example: `CacheServiceImpl` to implement `CacheService`.

**13b. [Recommended]** If the interface name is to indicate the ability of the interface, then its name should be an adjective.

> Positive example: `AbstractTranslator` to implement `Translatable`.

**14. [For Reference]** An Enumeration class name should end with `Enum`. Its members should be spelled out in upper case words, separated by underlines.

> Positive example: Enumeration name: `DealStatusEnum`; Member name: `SUCCESS / UNKOWN_REASON`.

**15. [For Reference]** Naming conventions for different package layers.

> Service/DAO layer method prefixes:
> - `get` — get one object
> - `list` — get multiple objects
> - `count` — count
> - `insert` / `save` — insert
> - `delete` / `remove` — delete
> - `update` — modify
>
> Domain model naming: DO (Data Object), DTO (Data Transfer Object), VO (View Object), POJO (Plain Ordinary Java Object).

---

## Constant Conventions

**1. [Mandatory]** Magic values, except for predefined, are forbidden in coding.

> Counter example: `String key = "Id#taobao_" + tradeId;`

**2. [Mandatory]** `L` instead of `l` should be used for `long` or `Long` variable because `l` is easily to be regarded as number `1` in mistake.

> Counter example: `Long a = 2l;` — it is hard to tell whether it is number 21 or Long 2.

**3. [Recommended]** Constants should be placed in different constant classes based on their functions.

> Example: cache-related constants in `CacheConsts`; configuration-related constants in `ConfigConsts`.
>
> Note: It is difficult to find one constant in one big complete constant class.

**4. [Recommended]** Constants shared across layers should be placed at the appropriate level. Five layers exist (cross-application shared, application internal shared, sub-project shared, package internal shared, within-class shared) — pick the lowest layer that satisfies the sharing scope.

> Counter example: Classes A and B both define `public static final String YES` but with different values ("yes" vs "y") in different layers. When they get compared in production, the comparison silently fails.

**5. [Recommended]** Use an enumeration class if values lie in a fixed range or if the variable has attributes.

> Positive example:
> ```java
> public enum SeasonEnum {
>     SPRING(1), SUMMER(2), AUTUMN(3), WINTER(4);
>     // ...
> }
> ```

---

## Formatting Style

**1. [Mandatory]** Rules for braces:
- No line break before the opening brace.
- A line break after the opening brace.
- A line break before the closing brace.
- A line break after the closing brace **only if** that closing brace terminates a statement, the body of a method/constructor, or a named class. (No line break if it is followed by `else`, `catch`, `finally`, or a comma.)

**2. [Mandatory]** No space is used between the `(` character and its following character. Same for the `)` character and its preceding character.

**3. [Mandatory]** There must be one space between keywords, such as `if`/`for`/`while`/`switch`, and parentheses.

**4. [Mandatory]** There must be one space at both left and right side of operators, such as `=`, `&&`, `+`, `-`, the ternary operator, etc.

**5. [Mandatory]** Each time a new block or block-like construct is opened, the indent increases by four spaces. When the block ends, the indent returns to the previous level. Both tabs and spaces are forbidden; use 4 spaces only. (IDEA: set Tab size = 4, uncheck "Use tab character". Eclipse: same setting under tab policy.)

> Positive example:
> ```java
> public static void main(String[] args) {
>     String say = "hello";
>     int flag = 0;
>     if (flag == 0) {
>         System.out.println(say);
>     }
>     if (flag == 1) {
>         System.out.println("world");
>     } else {
>         System.out.println("ok");
>     }
> }
> ```

**6. [Mandatory]** Java code is limited to **120 characters per line**. When a line is over 120 characters, wrap it following these rules:
- The second line should be indented 4 spaces relative to the first.
- A line break should be after an operator, not before.
- A line break should be before a `.`, not after.
- A method with multiple arguments that requires wrapping should break **after** the comma.
- Do not insert spaces before `(`.

**7. [Mandatory]** There must be one space between a comma and the next parameter for methods with multiple parameters.

> Positive example: `f("a", "b", "c");`

**8. [Mandatory]** The charset encoding of text files should be **UTF-8**, and the line-break character should be in **Unix** format (`\n`), not Windows format (`\r\n`).

**9. [Recommended]** It is unnecessary to align variables by several spaces.

> Counter example (over-aligned):
> ```java
> int    a   = 3;
> long   b   = 4L;
> float  c   = 5F;
> ```
>
> Positive example:
> ```java
> int a = 3;
> long b = 4L;
> float c = 5F;
> ```

**10. [Recommended]** Use a single blank line to separate sections with the same logic or semantics. Do not insert multiple consecutive blank lines.
