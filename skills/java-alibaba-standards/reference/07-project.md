# Project Specification

Verbatim excerpts from https://alibaba.github.io/Alibaba-Java-Coding-Guidelines/

---

## Application Layers

**1. [Recommended]** The upper layer depends on the lower layer by default. The seven-layer reference architecture is:

- **Open Interface Layer** — service encapsulated as an RPC interface, or exposed through the Web Layer as HTTP.
- **View Layer** — templates of each terminal render and execute (web/mobile/etc.).
- **Web Layer** — forward access control, basic parameter validation.
- **Service Layer** — concrete business logic implementation.
- **Manager Layer** — common business process layer: encapsulating third-party services, precipitation of general abilities, DAO interaction.
- **DAO Layer** — data access; interacts with MySQL, Oracle, HBase, etc.
- **External Interface / Third-Party Platform** — RPC clients to other services, REST clients to external APIs, message buses.

**2. [For Reference]** Exception handling varies by layer:
- **DAO Layer** — catch `Exception` and rethrow as `DAOException`. Do not log here (caught higher up).
- **Service Layer** — must record exception logs with parameter information.
- **Web Layer** — cannot throw exceptions; redirect to friendly error pages.
- **Open Interface Layer** — must use error codes and error messages.

**3. [For Reference]** Domain Model layers:
- **DO** (Data Object) — corresponds to the database table structure.
- **DTO** (Data Transfer Object) — objects transferred upward by the Service layer.
- **BO** (Business Object) — objects encapsulating business logic.
- **Query** — data query objects carrying query requests from upper layers.
- **VO** (View Object) — objects used in the Display layer.

---

## Library Specification

**1. [Mandatory]** GAV (GroupId / ArtifactId / Version) naming:
- **GroupId**: `com.{company/BU}.{business line}.{sub business line}` — maximum of 4 levels.
- **ArtifactId**: `<product name>-<module name>` (lowercase, hyphen-separated).
- Examples: `tc-client`, `uic-api`, `tair-tool`.

**2. [Mandatory]** Version naming: `<prime>.<secondary>.<revision>`
- **Prime** — incompatible API modification, or new features that can change product direction.
- **Secondary** — backward-compatible modification.
- **Revision** — bug fixes or feature additions that do not modify method signatures.

> The initial version must be `1.0.0`, not `0.0.1`.

**3. [Mandatory]** Online applications should not depend on `SNAPSHOT` versions (except security packages). Official releases must be verified to come from a central repository.

**4. [Mandatory]** When adding or upgrading libraries, keep the versions of dependent libraries unchanged unless necessary.

**5. [Mandatory]** Enumeration types may be defined or used for parameter types in libraries, but **cannot** be used for interface return types.

**6. [Mandatory]** When a group of libraries is used, define a uniform version variable to avoid inconsistent version numbers.

**7. [Mandatory]** For the same GroupId and ArtifactId, the Version must be the same across all sub-projects.

**8. [Recommended]** The declaration of dependencies in all POM files should be placed in a `<dependencies>` block. Versions of dependencies should be specified in `<dependencyManagement>` block.

**9. [Recommended]** Libraries should not include configuration — or at least should not add new configuration items.

**10. [For Reference]** Publishers should follow simplicity, stability, and traceability:
> Remove all unnecessary APIs and dependencies; include only Service APIs, necessary domain model objects, utility classes, constants, and enumerations.

---

## Server Specification

**1. [Recommended]** Reduce the `time_wait` value of the TCP protocol for high-concurrency servers.

> Example: `net.ipv4.tcp_fin_timeout = 30`

**2. [Recommended]** Increase the maximum number of file descriptors supported by the server.

> Prevents "open too many files" errors during high concurrent connections.

**3. [Recommended]** Set `-XX:+HeapDumpOnOutOfMemoryError` for the JVM, so JVM writes a heap dump when OOM occurs.

**4. [For Reference]** Use *forward* for internal redirection and URL-assembly utilities for external redirection.
