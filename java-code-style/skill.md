---
name: java-code-style
description: Apply Java readability conventions when writing or reviewing Java code. Covers method body breathing room, named boolean conditions, cross-service call comments, private-method extraction, variable grouping, guard clauses, Optional returns, method ordering, parameter alignment, service method section labels, controller endpoint annotation formatting, @Transactional proxy reachability and minimal transaction scope (self-invocation/private-method pitfalls, no slow or remote work inside the boundary), lock-compare-update documentation, and variant-dispatch internalization. Triggers automatically alongside java-alibaba-standards for any Java work. Also invoke explicitly via /java-code-style.
---

# Java Code Style — Readability Conventions

Canonical source: `coding-standards.md` (co-located in this skill directory — shared across all
consuming repos via the dev-standards submodule). Read that file for the full spec with
before/after examples. The rules below are the enforcement checklist.

---

## When to invoke

Trigger automatically (alongside `java-alibaba-standards`) when any of these are true:

- Writing, editing, or reviewing any `.java` file under `src/main/java` or `src/test/java`
- Reviewing a diff that contains `.java` changes
- The user invokes `/java-code-style` explicitly

---

## Rules checklist

Read `coding-standards.md` for full examples. Summary:

### Rule 1 — Blank lines around control-flow blocks
Every `if`/`else`/`for`/`while`/`switch` must be preceded AND followed by a blank line, unless:
- it is the very first or last statement in the method body, OR
- it is directly nested inside another control-flow block (no extra blank line between outer and inner)

### Rule 2 — Named boolean variables for complex conditions
Extract compound conditions (`a != null && a != 0L`, multi-clause `&&`/`||`) into a named
`boolean` local variable before the `if`. Name it after the business concept it tests.

### Rule 3 — Cross-service call comments
Every line calling an injected service, repository, or port must have a `//` comment on the line
immediately above it. The comment explains the *business step* — not a restatement of the method
name. Keep the comment in sync when the call changes.

### Rule 4 — Extract multi-concern private methods
A private helper that resolves more than one distinct business concern must be split into one
private method per concern. The public/calling method then reads as a sequence of named steps.

### Rule 5 — Group variable declarations, blank line before use
Declare all locals for a logical step together at the top of the block. Leave one blank line
between the declaration group and the first statement that fills or consumes them.

### Rule 6 — Return early; guard clauses over nested else
At service boundaries, bail out immediately for absent/null/empty cases. Do not wrap the happy
path inside a positive `if` block. Applies to `Optional.isEmpty()`, null checks, and empty-list
early returns. Stronger than P3C flow rule 3 (which is only [Recommended]).

### Rule 7 — Express absence with `Optional`; never return `null` from a public service method
- Absent single result → `Optional<T>`
- Always-a-collection result → `List<T>` / `Map<K,V>`, never `null`
- `null` is only permitted inside DTO fields where the wire contract requires it
Stronger than P3C exception rule 9 (which only asks for a Javadoc note).

### Rule 8 — Method ordering within a class
Declare in this order: `public` → `protected` → `private` helpers → `static` mappers/converters.
Related methods stay adjacent within each group. Static mappers (`toDto`, `toEntity`) always last.
Extends P3C OOP rule 16 (which omits the static-mapper group).

### Rule 9 — Method parameter alignment
Keep all params on one line when the signature fits within 120 cols. When it overflows, let
Spotless wrap — do not manually reformat. Never force each param onto its own line unless
Spotless produces that output itself.

### Rule 10 — Service method section labels
Every public service method body must be structured into labelled sections:
1. `// validate` — all `Assert.*` / guard checks on incoming parameters
2. `// init` — local variable declarations and setup derived from parameters
3. Business logic steps — each cross-service call preceded by its own descriptive `//` comment (Rule 3)

Each section separated by a blank line. Omit a label only if the section is absent.

### Rule 11 — Controller endpoint annotation formatting
`@Operation`: `summary` on the same line, `description` on continuation only when overflow.
`@io.swagger...ApiResponse`: same — `responseCode` on same line, `description` on continuation.
`@Parameter` + binding annotation (`@PathVariable`, `@RequestParam`): keep together on one line
when they fit; let Spotless wrap `@RequestParam(...)` to continuation when overflow. Never
manually split them onto separate lines.

### Rule 12 — `@Transactional` must be proxy-reachable, with minimal scope
`@Transactional` only takes effect on a `public` method called from **outside** the bean (through
the Spring proxy). On a `private`/`protected` method, or one reached by self-invocation
(`this.method()`), it is silently a no-op — no transaction, runs in autocommit. To exclude work
(e.g. BCrypt) from the tx, use a `TransactionTemplate` at the call site or a separate `@Service`
bean — never a private `@Transactional` helper called from the same bean. Declare `@Transactional`
on any method whose body does a lock-then-write or multiple atomic writes (`REQUIRED` joins a
caller's tx or opens its own). Reinforces P3C Exception rule 5.
**Minimize the scope:** a transaction holds a DB connection and any row locks for its whole
duration, so wrap **only** the atomic writes. Keep slow CPU work (BCrypt), network/external I/O
(HTTP/RPC, messaging, uploads, email), validation, and read-backs OUTSIDE the boundary — compute
before, write inside, map/return after commit. Never hold a transaction across a remote call (use a
saga/outbox instead). This is the main reason to prefer a narrow `TransactionTemplate.execute` block
or a small focused `@Transactional` method over annotating a large orchestration method.

### Rule 13 — Document the lock → compare → update pattern
Pessimistic read-modify-write methods (`SELECT … FOR UPDATE`, check, write) label the three steps
in comments: `// 1. Lock` (held until commit), `// 2. Compare` (under the lock; say so when no
compare is needed), `// 3. Update & release` (lock releases on commit). Use `BigDecimal.compareTo`,
never `equals`.

### Rule 14 — Internalize variant dispatch
When sibling operations differ only along an enum dimension, take it as a parameter and branch
**inside** the method (dispatching to private helpers) rather than exposing one public method per
variant that forces every caller to pick. Keep siblings symmetric (if `credit(..., fundType)` hides
the bucket, so should `debit(..., fundType)`). Use colon-style `switch` with `default:` — the
P3C-PMD `SwitchStatementRule` does not recognise arrow-syntax `default ->`.

---

## Priority

- These rules take precedence over P3C defaults where they conflict.
- P3C `java-alibaba-standards` still applies for all other matters (naming, concurrency, exceptions,
  ORM, security). Load that skill in parallel — do not skip it.
- Spotless/Eclipse formatter owns whitespace between tokens; these rules govern blank-line
  structure and comment placement, which Spotless does not enforce.

---

## Generation mode behaviour

When writing new code:
1. Apply all 14 rules proactively — do not wait to be asked.
2. If a method body exceeds ~15 lines without a blank line, that is a signal Rule 1 or Rule 4 is
   being violated.
3. Any `.service(...)`, `.repository(...)`, `.findXxx(...)`, `.getXxx(...)` call on an injected
   field is a cross-service call — Rule 3 applies.
4. Any public service method returning a single object must return `Optional<T>` if absence is
   possible — Rule 7. A `null` return from a public method is always a violation.
5. When writing a new class, lay out methods in Rule 8 order from the start; do not append to the
   bottom of the file without checking position.
6. Keep method params on one line when the signature fits in 120 cols — Rule 9. Let Spotless own
   the wrapping on overflow; never hand-split a signature that could fit.
7. Every public service method gets `// validate`, `// init`, and labelled business-logic sections
   from the start — Rule 10. Write the labels as you go, not as an afterthought.
8. On controller endpoints, lay out `@Operation` / `@ApiResponse` / `@Parameter` per Rule 11; let
   Spotless wrap on overflow, never hand-split annotations that fit.
9. Never put `@Transactional` on a `private` method or one reached by self-invocation — Rule 12.
   For a partial-method atomic unit, use a `TransactionTemplate` or a separate bean. Keep the
   transaction scope minimal: slow work (BCrypt), network/external I/O, validation, and read-backs
   stay OUTSIDE the boundary; wrap only the atomic writes — never hold a tx across a remote call.
10. Label the lock → compare → update steps on any pessimistic read-modify-write — Rule 13.
11. Take an enum dimension as a parameter and dispatch internally rather than exposing one public
    method per variant — Rule 14; use colon-style `switch` with `default:`.

## Review mode behaviour

Flag violations in this format:
```
[Style] ClassName.java:NN — Style rule <N>
  Issue: <what is wrong>
  Fix: <concrete change>
```
