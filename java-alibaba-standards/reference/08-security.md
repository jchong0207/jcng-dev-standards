# Security Specification

Verbatim excerpts from https://alibaba.github.io/Alibaba-Java-Coding-Guidelines/

Every rule in this section is **[Mandatory]** unless tagged otherwise.

---

**1. [Mandatory]** User-owned pages or functions must be authorized.

> Note: Prevent access to and manipulation of other people's data without authorization checks, e.g. viewing or modifying other people's orders.
>
> Practical: never trust client-supplied IDs alone. Always check that the current authenticated principal owns / is permitted to act on the resource.

---

**2. [Mandatory]** Direct display of user sensitive data is not allowed. Displayed data must be desensitized.

> Note: Personal phone numbers should be displayed as `158****9119` — the middle 4 digits are hidden to prevent privacy leaks.
>
> Same idea applies to email (`j***@example.com`), national-ID number, bank-card number, etc. Apply at serialization time so it is hard to bypass.

---

**3. [Mandatory]** SQL parameters entered by users should be checked carefully or limited by METADATA, to prevent SQL injection. Database access by string-concatenated SQL is forbidden.

> ```java
> // Bad — SQL injection vector:
> "SELECT * FROM users WHERE id = " + userId
>
> // Good — parameterized:
> "SELECT * FROM users WHERE id = ?", userId
> ```
>
> Same rule for `ORDER BY` / `LIMIT` columns — these cannot be parameterized, so they must be **whitelisted** (compared against an allow-list of column names), never interpolated.

---

**4. [Mandatory]** Any parameter input by users must go through validation.

> Note: Ignoring parameter checks may cause:
> - Memory leak from excessive page sizes
> - Slow database queries from malicious `ORDER BY`
> - Arbitrary redirection
> - SQL injection
> - Deserialization injection
> - ReDoS (regex denial-of-service)
>
> Regular expressions used to validate user input can themselves be a vulnerability if an attacker constructs a special string causing catastrophic backtracking.

---

**5. [Mandatory]** It is forbidden to output user data to HTML pages without security filtering or proper escaping.

> Use the templating engine's auto-escaping output (`${var}` in most engines), not raw concatenation (`<%= var %>`).
> For user-supplied URLs, validate scheme (`http` / `https` only — reject `javascript:`, `data:`).

---

**6. [Mandatory]** Forms and AJAX submissions must be filtered by CSRF security check.

> Note: CSRF (Cross-Site Request Forgery) is a common programming flaw. For applications with CSRF holes, attackers can construct a URL in advance and modify user parameters as soon as the victim visits — without their awareness.
>
> Typical defenses: synchronizer token pattern (token in cookie + header, verified server-side), or `SameSite=Strict` cookies, or both.

---

**7. [Mandatory]** Use correct anti-replay restrictions — number limits, fatigue control, verification-code checking — to avoid abuse of platform resources such as SMS, email, telephone, orders, payments.

> Note: For example, if there is no limit on the rate at which SMS verification codes can be sent, users will be bothered and SMS-platform resources wasted.
>
> Apply per-IP, per-account, and per-device counters with sliding windows. Add captcha or fatigue-control on suspected abuse.

---

**8. [Recommended]** In scenarios where users generate content (posts, comments, instant messages), anti-spam keyword filtering and other risk-control strategies must be applied.

> Apply before persistence *and* before render.
