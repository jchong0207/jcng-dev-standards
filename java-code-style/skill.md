---
name: java-code-style
description: Apply Java readability conventions when writing or reviewing Java code. Covers method body breathing room, named boolean conditions, cross-service call comments, private-method extraction, variable grouping, guard clauses, Optional returns, and method ordering. Triggers automatically alongside java-alibaba-standards for any Java work. Also invoke explicitly via /java-code-style.
---

# Java Code Style — Readability Conventions

Canonical source (for crm-java-user): `docs/coding-standards.md`. Read that file for the full
spec with before/after examples. The rules below are the enforcement checklist.

---

## When to invoke

Trigger automatically (alongside `java-alibaba-standards`) when any of these are true:

- Writing, editing, or reviewing any `.java` file under `src/main/java` or `src/test/java`
- Reviewing a diff that contains `.java` changes
- The user invokes `/java-code-style` explicitly

---

## Rules checklist

Read `docs/coding-standards.md` for full examples. Summary:

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
1. Apply all 8 rules proactively — do not wait to be asked.
2. If a method body exceeds ~15 lines without a blank line, that is a signal Rule 1 or Rule 4 is
   being violated.
3. Any `.service(...)`, `.repository(...)`, `.findXxx(...)`, `.getXxx(...)` call on an injected
   field is a cross-service call — Rule 3 applies.
4. Any public service method returning a single object must return `Optional<T>` if absence is
   possible — Rule 7. A `null` return from a public method is always a violation.
5. When writing a new class, lay out methods in Rule 8 order from the start; do not append to the
   bottom of the file without checking position.

## Review mode behaviour

Flag violations in this format:
```
[Style] ClassName.java:NN — Style rule <N>
  Issue: <what is wrong>
  Fix: <concrete change>
```
