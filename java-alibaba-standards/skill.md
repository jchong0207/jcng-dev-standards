---
name: java-alibaba-standards
description: Apply the Alibaba Java Coding Guidelines (P3C) when writing or reviewing Java code. Use this skill whenever generating, editing, or reviewing files under src/main/java or src/test/java, or any Java-language code. Covers concurrency, OOP/collections, exceptions and logging, MySQL/ORM, security, naming and formatting. Also invoke explicitly via /java-alibaba-standards.
---

# Java Alibaba Coding Standards (P3C)

Canonical source: https://alibaba.github.io/Alibaba-Java-Coding-Guidelines/

This skill is an entry point. The actual rules — quoted verbatim from the source, with original examples — live in `reference/`. Load only the files relevant to the work in front of you.

---

## When to invoke

Trigger automatically when any of these are true:

- About to write, edit, or refactor a `.java` file
- Reviewing a diff or PR that contains `.java` changes
- Designing a new Java class, service, controller, or DAO
- The user invokes `/java-alibaba-standards` explicitly

Skip for: build files (`pom.xml`), pure config (XML, properties), JSP templates, third-party `.java` files inside `vendor/`, `lib/`, `target/`, `node_modules/`.

---

## Workflow: load only what you need

Do **not** read every reference file up front — that wastes context. Decide which rule categories apply to the task, then read just those.

| Reference file | Read it when... |
|---|---|
| `reference/01-naming-constants-format.md` | Creating new identifiers, choosing class/method/field/package names, defining constants, reviewing brace/indent/whitespace style |
| `reference/02-oop-collections.md` | Editing POJOs, overriding `equals`/`hashCode`/`toString`, using generics, working with `List`/`Map`/`Set`/`Arrays.asList`/`subList`/`Comparator` |
| `reference/03-concurrency.md` | Anything with threads, locks, `ThreadLocal`, `volatile`, `synchronized`, executors, `Atomic*`, `SimpleDateFormat`, singletons, static mutable state |
| `reference/04-flow-comments-other.md` | Conditional logic, switch/if-else chains, loop optimization, regex precompile, time/date code, Velocity templates, deleting obsolete code |
| `reference/05-exceptions-logs.md` | try/catch blocks, finally blocks, custom exceptions, transaction rollback, SLF4J usage, log levels, log placeholders |
| `reference/06-mysql.md` | DAOs, MyBatis XML, JPA repositories, schema design, index design, raw SQL, ORM mapping |
| `reference/07-project.md` | Adding modules, layering decisions (Controller/Service/Manager/DAO), POM dependency declarations, library versioning, server tuning |
| `reference/08-security.md` | Auth checks, user input validation, SQL parameters, HTML output escaping, CSRF, rate limiting, PII display, anti-replay |

**For a broad "review this whole file" request**, read `03-concurrency.md`, `05-exceptions-logs.md`, `08-security.md` first — they catch the highest-severity bugs — then sweep the rest if the file warrants it.

---

## Two modes

### Generation mode
You are writing or editing Java code. Read the relevant reference files, then apply every **[Mandatory]** rule. Apply **[Recommended]** rules unless surrounding code already violates them in a load-bearing way (matching local style sometimes beats a one-line deviation). Ignore **[For Reference]** rules unless asked.

### Review mode
You are scanning existing code for violations. Output findings grouped by severity, in this format:

```
[Mandatory] PayService.java:142 — Concurrency rule 9
  Issue: lock.lock() inside try block; IllegalMonitorStateException risk if acquisition fails.
  Fix: move lock.lock() above the try block.

[Mandatory] OrderDao.java:88 — Security rule 3
  Issue: SQL built via string concatenation with userId.
  Fix: use ? placeholder via JdbcTemplate / MyBatis #{param}.
```

Lead with Mandatory findings. Cap Recommended at the top 10 most impactful (don't bury the user). Skip For-Reference findings unless asked.

---

## Severity legend

| P3C tag | Chinese | Meaning | Action |
|---|---|---|---|
| **[Mandatory]** | 强制 | Bug or strong correctness/safety risk | Must fix |
| **[Recommended]** | 推荐 | Strongly preferred pattern | Fix unless justified |
| **[For Reference]** | 参考 | Style / informational | Nice to have |

---

## Citing rules in output

Cite as `<Section> rule <N>`, e.g. "Concurrency rule 4" or "Security rule 3". This matches the structure of the canonical document and the reference files in this skill, and stays stable across edits.

---

## Integration notes

- **This is prompt-time guidance, not a scanner.** For automated CI enforcement, pair with the `p3c-pmd` Maven ruleset, or the "Alibaba Java Coding Guidelines" IntelliJ plugin (publisher: Alibaba).
- **Project rules win over P3C on conflict.** If a project layers its own coding-standards doc on top (a `CONTRIBUTING.md`, `docs/style.md`, or its own per-repo conventions), follow the project's version and call out the conflict in review output so the reader can decide.
- **Shared readability conventions:** These standards' readability layer lives in the co-located `java-code-style/coding-standards.md` (shared across all consuming repos via this submodule) and is enforced by the `java-code-style` skill. Always invoke `java-code-style` in parallel when working in any consuming repo — it covers blank-line structure, named boolean conditions, cross-service call comments, private-method extraction, guard clauses, Optional returns, method ordering, parameter alignment, service method section labels, and controller annotation formatting that P3C does not address.
- Reference files contain **verbatim excerpts** from the canonical source as of the skill's authoring time. The Alibaba document is occasionally revised — if a rule's wording feels surprising, consult the canonical URL above.
- The reference files preserve the original numbering and `[Mandatory]/[Recommended]/[For Reference]` tags so they can be cross-referenced with the official document or with PMD output.
