# Coding Standards — Java Readability Conventions

Portable, project-agnostic readability conventions that layer on top of Alibaba P3C. Where these
rules conflict with P3C, these win. The `java-code-style` skill (co-located in this directory)
applies these automatically when generating or reviewing Java.

This file is the **canonical source** shared across all consuming repos via the
`jcng-dev-standards` submodule mounted at `.claude/skills/`. A project may still keep a thin
`docs/coding-standards.md` pointer, but the authoritative spec with examples lives here.

---

## 1. Method body breathing room

**Blank lines around control-flow blocks.**
Every `if`, `else`, `for`, `while`, or `switch` block must be preceded by a blank line and followed
by a blank line, unless it is the very first or last statement in the method body, or it is directly
nested inside another control-flow block.

```java
// BAD — control flow cramped against variable declarations
List<Long> ownerIds = owners.items();
Map<Long, List<Long>> clientsByOwner = clientService.findClientIdsByAssignedStaffIds(ownerIds);
for (Long staffId : ownerIds) {
    if (owner == null) {
        continue;
    }
    rosters.add(...);
}
return PageResult.of(rosters, owners.total(), page, size);

// GOOD — blank lines let the eye parse structure
List<Long> ownerIds = owners.items();
Map<Long, List<Long>> clientsByOwner = clientService.findClientIdsByAssignedStaffIds(ownerIds);

for (Long staffId : ownerIds) {
    if (owner == null) {
        continue;
    }

    rosters.add(...);
}

return PageResult.of(rosters, owners.total(), page, size);
```

---

## 2. Named boolean variables for complex conditions

Extract compound conditions into a named `boolean` before the `if`. The name documents intent;
the `if` line stays readable. (Reinforces P3C flow rule 4.)

```java
// BAD
if (client.asignId() != null && client.asignId() != 0L) { ... }

// GOOD
boolean hasAssignedOwner = client.asignId() != null && client.asignId() != 0L;
if (hasAssignedOwner) { ... }
```

---

## 3. Cross-service call comments

Any line that calls into an injected service, repository, or port must be preceded by a `//`
comment on the line above it explaining the *business step* being performed — not restating the
method name, but saying what it resolves or why it is needed.

This applies in orchestration services, repository impls, and anywhere multiple external calls are
composed in sequence.

```java
// BAD — reader must infer purpose from the method name
Map<Long, List<Long>> clientsByOwner = clientService.findClientIdsByAssignedStaffIds(ownerIds);
Map<Long, List<Long>> loginsByClient = tradingAccountService.findLoginsByClientIds(allClientIds);
Map<Long, StaffDTO> ownersById = staffService.getByIds(ownerIds)...

// GOOD — each call is self-documenting
// Fetch all clients grouped by their assigned staff (owner) for this page
Map<Long, List<Long>> clientsByOwner = clientService.findClientIdsByAssignedStaffIds(ownerIds);

// Resolve the trading logins for every client in one batch (avoids per-client N+1)
Map<Long, List<Long>> loginsByClient = tradingAccountService.findLoginsByClientIds(allClientIds);

// Resolve owner name + department id in one batch
Map<Long, StaffDTO> ownersById = staffService.getByIds(ownerIds)...
```

The comment should be:
- One line (`//` style, above the call — P3C comment rule 4)
- Focused on *why* or *what business concept* this resolves, not a restatement of the method name
- Kept in sync when the call changes (P3C comment rule 7)

---

## 4. Extract multi-concern private methods in orchestration services

When a private helper resolves more than one distinct concern (e.g. owner name, team name, and
recommender in the same body), break it into one private method per concern. The calling method
then reads as a named sequence of steps.

```java
// BAD — three concerns in one private body
private ClientInfoDTO toClientInfo(AccountDetailDTO account) {
    Long clientId = account.userId();
    ClientDTO client = clientPayload(clientId);
    String owner = null;
    String salesTeam = null;
    String recommender = null;
    if (client != null) {
        if (client.asignId() != null && client.asignId() != 0L) {
            owner = staffService.getById(client.asignId()).map(s -> s.name()).orElse(null);
            salesTeam = departmentMembershipQueryService.departmentNameForStaff(client.asignId()).orElse(null);
        }
        if (client.parentId() != null && client.parentId() != 0L) {
            recommender = clientName(client.parentId());
        }
    }
    ...
}

// GOOD — one method per concern; caller reads like a sequence of named steps
private ClientInfoDTO toClientInfo(AccountDetailDTO account) {
    Long clientId = account.userId();
    ClientDTO client = clientPayload(clientId);
    String owner = resolveOwnerName(client);
    String salesTeam = resolveTeamName(client);
    String recommender = resolveRecommenderName(client);
    ...
}

private String resolveOwnerName(ClientDTO client) {
    boolean hasAssignedOwner = client != null && client.asignId() != null && client.asignId() != 0L;
    if (!hasAssignedOwner) {
        return null;
    }
    return staffService.getById(client.asignId()).map(StaffDTO::name).orElse(null);
}

private String resolveTeamName(ClientDTO client) {
    boolean hasAssignedOwner = client != null && client.asignId() != null && client.asignId() != 0L;
    if (!hasAssignedOwner) {
        return null;
    }
    return departmentMembershipQueryService.departmentNameForStaff(client.asignId()).orElse(null);
}

private String resolveRecommenderName(ClientDTO client) {
    boolean hasRecommender = client != null && client.parentId() != null && client.parentId() != 0L;
    if (!hasRecommender) {
        return null;
    }
    return clientName(client.parentId());
}
```

---

## 5. Group variable declarations, then leave a blank line before use

Declare all local variables for a logical step together at the top of the block, then leave one
blank line before the statement that consumes them. Do not interleave declarations and logic.

```java
// BAD — declaration and use interleaved with unrelated logic
String owner = null;
String salesTeam = null;
if (client != null) {
    String recommender = null;
    ...
}

// GOOD — all declarations up front, blank line before the block that fills them
String owner = null;
String salesTeam = null;
String recommender = null;

if (client != null) {
    ...
}
```

---

## 6. Return early — guard clauses over nested else

For any case where a method cannot proceed (missing entity, absent Optional, null input), return or
throw immediately at the top of the method. Do not wrap the happy path inside an `if` block.
P3C flow rule 3 permits this but only as [Recommended] with a basic example — this standard makes
it mandatory at service boundaries where `Optional` returns are common.

```java
// BAD — happy path buried inside a positive condition
public Optional<UplineDTO> upline(long userId) {
    if (partnerRepository.findPartner(userId).isPresent()) {
        return Optional.of(new UplineDTO(userId, partnerRepository.ancestorChain(userId)));
    } else {
        return Optional.empty();
    }
}

// GOOD — guard clause first, happy path is the straight line
public Optional<UplineDTO> upline(long userId) {
    if (partnerRepository.findPartner(userId).isEmpty()) {
        return Optional.empty();
    }

    return Optional.of(new UplineDTO(userId, partnerRepository.ancestorChain(userId)));
}
```

The guard clause pattern also applies to null checks and early-empty-list returns in repository
and service methods.

---

## 7. Express absence with `Optional`; never return `null` from a public service method

`null` is invisible in the return type — the caller cannot tell from the signature whether a null
check is required. Public service methods must follow these rules:

- If the result may be absent → return `Optional<T>`
- If the result is always a collection (possibly empty) → return `List<T>` / `Map<K,V>`, never `null`
- `null` is only acceptable as a field value inside a DTO where the wire contract permits it

P3C exception rule 9 only asks for a Javadoc note when null is possible — this rule goes further
and requires the absence to be expressed in the type itself.

```java
// BAD — caller has no signal that null is possible
public ClientDTO getClient(long id) { ... }

// GOOD — absence is part of the contract
public Optional<ClientDTO> getClient(long id) { ... }

// GOOD — always a list, never null
public List<ClientDTO> listClients(...) {
    if (noResults) {
        return List.of();
    }
    ...
}
```

---

## 8. Method ordering within a class

Declare methods in this order so a reader can scan top-to-bottom from contract to implementation:

1. `public` methods — the contract (most readers stop here)
2. `protected` methods — extension points
3. `private` / package-private helpers — the implementation detail
4. Static mapper / converter methods (e.g. `toDto`, `toEntity`, `toNodeDto`) — always last

Within each group, related methods stay adjacent (e.g. `resolveOwnerName` next to `resolveTeamName`).

```java
// GOOD ordering in a service impl
@Override
public PageResult<AccountDTO> listAccounts(...) { ... }   // 1. public

@Override
public Optional<AccountDetailDTO> getAccount(...) { ... } // 1. public

private ClientDTO clientPayload(Long clientId) { ... }    // 3. private helper

private static AccountDTO toAccountDto(...) { ... }       // 4. static mapper

private static AccountDetailDTO toDetailDto(...) { ... }  // 4. static mapper
```

P3C OOP rule 16 mandates public → private → getter/setter; this adds static mappers as an
explicit last group, which P3C does not mention.

---

## 9. Method parameter alignment

When a method signature fits within 120 columns, keep all parameters on one line. When it
overflows, the Spotless/Eclipse formatter wraps the overflow onto the next line(s) at a 4-space
continuation indent — do not manually reformat signatures; let Spotless own this.

```java
// GOOD — fits on one line, stays on one line
public PageResult<AccountDTO> listAccounts(long userId, Integer managerId, int page, int size, String sort) {

// GOOD — overflows, Spotless wraps at 4-space continuation indent (accepted formatter output)
public PageResult<AccountTreeNodeDTO> accountTree(long userId, Long login, Integer managerId, int page, int size,
    String sort) {

// BAD — manually forcing each param to its own line when the signature could fit
public PageResult<AccountDTO> listAccounts(
    long userId,
    Integer managerId,
    int page,
    int size,
    String sort) {
```

---

## 10. Service method section labels

Every public service method body must be structured into labelled sections using `//` comments.
The labels make the method scannable at a glance without reading the detail.

**Required sections (in order):**
1. `// validate` — all `Assert.*` / guard checks on incoming parameters
2. `// init` — local variable declarations and any setup derived from the parameters (e.g. `Pagination.of(...)`)
3. Business logic steps — each cross-service call preceded by its own descriptive `//` comment (Rule 3)

Each section is separated by a blank line. Omit a section label only if the section is absent
(e.g. a method with no validation skips `// validate`).

```java
// BAD — no structure, validate and init cramped together with logic
public PageResult<AccountDTO> listAccounts(long userId, Integer managerId, int page, int size, String sort) {
    Assert.isTrue(userId > 0, "userId must be positive");
    Pagination pagination = Pagination.of(page, size, sort, ACCOUNT_SORTABLE);
    PageResult<BaseTradingAccount> result = tradingAccountRepository.listAccounts(userId, managerId, pagination);
    List<AccountDTO> items = result.items().stream().map(TradingAccountServiceImpl::toAccountDto).toList();
    return PageResult.of(items, result.total(), page, size);
}

// GOOD — three labelled sections, each visually distinct
public PageResult<AccountDTO> listAccounts(long userId, Integer managerId, int page, int size, String sort) {
    // validate
    Assert.isTrue(userId > 0, "userId must be positive");

    // init
    Pagination pagination = Pagination.of(page, size, sort, ACCOUNT_SORTABLE);

    // Page accounts for this user across MT4+MT5
    PageResult<BaseTradingAccount> result = tradingAccountRepository.listAccounts(userId, managerId, pagination);
    List<AccountDTO> items = result.items().stream().map(TradingAccountServiceImpl::toAccountDto).toList();

    return PageResult.of(items, result.total(), page, size);
}
```

---

## 11. Controller endpoint annotation formatting

Swagger/OpenAPI annotations on controller endpoints follow these layout rules. Spotless owns
wrapping — do not manually break lines that fit within 120 cols.

### `@Operation`
`summary` stays on the same line as `@Operation`. When `description` would push the line over 120
cols, Spotless wraps it to a 4-space continuation indent. Both attributes are always present.

```java
// GOOD — fits on one line
@Operation(summary = "Role list", description = "All roles (read-parity; no role-detail endpoint exists).")

// GOOD — description overflows, Spotless wraps it
@Operation(summary = "List accounts (paged)",
        description = "Unified MT4+MT5 account list for a user; managerId null = all platforms.")
```

### `@io.swagger...ApiResponse`
Same rule: `responseCode` on the same line; `description` wrapped to continuation when overflow.

```java
@io.swagger.v3.oas.annotations.responses.ApiResponse(responseCode = "200",
        description = "Envelope; success=false when the account is not found")
```

### `@Parameter` + binding annotation
`@Parameter` and its binding annotation (`@PathVariable`, `@RequestParam`) stay on the same line
as the parameter when they fit. When `@RequestParam(...)` overflows, Spotless wraps it to a
continuation indent — do not manually split `@Parameter` and `@RequestParam` onto separate lines.

```java
// GOOD — fits, stays on one line
@Parameter(description = "the MT login / account number") @PathVariable long login,

// GOOD — @RequestParam overflows, Spotless wraps it
@Parameter(description = "platform selector (1/5=MT4, 3=MT5), or null") @RequestParam(
        required = false) Integer managerId
```

---

## 12. `@Transactional` must be proxy-reachable; never on a self-invoked or private method

Spring's `@Transactional` is implemented by a **proxy** that wraps the bean. The annotation only
takes effect when the method is (a) `public` and (b) called **from outside the bean** so the call
passes through the proxy. Two structures silently disable it — the annotation is ignored, no
transaction opens, and the code looks correct while running in autocommit:

- **`private` (or non-public) `@Transactional` method** — the proxy cannot intercept it.
- **Self-invocation** — one method calling another on the same bean via `this`; the call never
  re-enters the proxy.

When you need an atomic unit that must exclude some work (e.g. a slow BCrypt verify) from the
transaction, do **not** extract it into a private `@Transactional` helper called from the same bean.
Use one of:
1. **`TransactionTemplate`** — open the transaction programmatically, exactly where the writes are.
   Immune to the proxy pitfalls; the boundary is explicit and cannot be defeated by a later refactor.
2. **A separate `@Service` bean** holding the `public @Transactional` method, injected and called
   across the bean boundary (so the call is proxied).

Also: declare `@Transactional` on **any** method whose own body does a lock-then-write or multiple
writes that must commit together — atomicity is a property the method guarantees about itself, not a
hope about its callers. With the default `REQUIRED` propagation it joins a caller's transaction when
one exists and opens its own when one does not, so the declaration is free when nested and essential
when called directly. (Reinforces P3C Exception rule 5 — rollback happens by exception propagation
out of the proxied boundary.)

```java
// BAD — @Transactional on a private method called via this: the proxy is bypassed, NO transaction
// opens, and a failed placeHold leaves an orphaned order row in autocommit.
public OrderDTO placeOrder(long memberId, PlaceOrderCommand cmd) {
    passwordService.verify(memberId, cmd.password());   // slow BCrypt, kept outside the tx
    return placeOrderTransactional(memberId, cmd);       // self-invocation → annotation inert
}

@Transactional(rollbackFor = Exception.class)
private OrderDTO placeOrderTransactional(long memberId, PlaceOrderCommand cmd) {
    long id = orderRepository.insert(...);               // write 1
    accountingService.placeHold(memberId, cmd.amount(), id);  // write 2 — NOT atomic with write 1
    return OrderConverter.toDto(orderRepository.findById(id));
}

// GOOD — TransactionTemplate opens the transaction at the call site; BCrypt runs before the first
// SQL so it never holds a connection. The two writes commit or roll back together.
public OrderDTO placeOrder(long memberId, PlaceOrderCommand cmd) {
    // validate / init ...
    passwordService.verify(memberId, cmd.password());

    return transactionTemplate.execute(status -> {
        long id = orderRepository.insert(...);
        accountingService.placeHold(memberId, cmd.amount(), id);  // joins this tx (REQUIRED)
        return OrderConverter.toDto(orderRepository.findById(id));
    });
}
```

**Red flags — the annotation is probably inert:** `@Transactional` on a `private`/`protected`
method; a `this.someTransactionalMethod(...)` call within the same class; a `@Transactional` method
that "fixes" atomicity but the orphaned-row bug still reproduces.

### Keep the transaction scope minimal — never wrap slow or external work

A transaction holds a pooled DB connection (and any row locks) for its **entire** duration. The
longer it stays open, the longer locks block other writers and the longer the connection is
unavailable to the pool — under load this causes lock contention and connection-pool starvation.
So the transaction must wrap **only** the database writes that must commit atomically, and nothing
else. Keep these **outside** the boundary:

- Slow CPU work — BCrypt/hashing, serialization, large in-memory transforms.
- Any network or external I/O — HTTP/RPC calls, message publishing, S3/file uploads, sending email.
- Input validation, order-number generation, DTO mapping, and read-backs that don't need to be in
  the same atomic unit.

Compute everything you can **before** opening the transaction, do the writes, and map/return
**after** it commits. This is also *why* the patterns above (a narrow `TransactionTemplate.execute`
block, or a small focused `@Transactional` method) are preferred over annotating a large
orchestration method: a method-level `@Transactional` silently pulls every line of the method —
including any slow or remote call — inside the boundary.

```java
// BAD — a slow HTTP call and BCrypt both run while the transaction (and its connection) is open.
@Transactional(rollbackFor = Exception.class)
public OrderDTO placeOrder(long memberId, PlaceOrderCommand cmd) {
    passwordService.verify(memberId, cmd.password());          // ~100ms BCrypt — holds a connection
    FxRate rate = fxClient.fetchRate(cmd.currency());          // network I/O — holds a connection
    long id = orderRepository.insert(toOrder(cmd, rate));
    accountingService.placeHold(memberId, cmd.amount(), id);
    return OrderConverter.toDto(orderRepository.findById(id));
}

// GOOD — slow + remote work happens first; the tx wraps only the two writes.
public OrderDTO placeOrder(long memberId, PlaceOrderCommand cmd) {
    passwordService.verify(memberId, cmd.password());          // outside the tx
    FxRate rate = fxClient.fetchRate(cmd.currency());          // outside the tx
    Order toInsert = toOrder(cmd, rate);                       // prepared outside the tx

    long id = transactionTemplate.execute(status -> {
        long orderId = orderRepository.insert(toInsert);
        accountingService.placeHold(memberId, cmd.amount(), orderId);
        return orderId;
    });

    return OrderConverter.toDto(orderRepository.findById(id)); // read-back after commit
}
```

If a remote call genuinely must be coordinated with the DB write (e.g. call a payment gateway then
record the result), do **not** hold a transaction across the call — use a saga / outbox / compensating
action instead. A transaction is not the tool for cross-system consistency.

---

## 13. Document the lock → compare → update pattern on balance/row mutations

Any method that does a pessimistic read-modify-write (`SELECT … FOR UPDATE`, check the locked
state, then write) must label the three steps in comments, so the concurrency contract is visible
without reverse-engineering it. The lock is held across all three steps inside one transaction; the
update releases it on commit.

```java
// GOOD — each phase is named; a reader sees the lock is held across check-and-write
@Transactional(rollbackFor = Exception.class)
public void debitWithHoldRelease(long memberId, String currency, BigDecimal amount, ...) {
    // 1. Lock: load the wallet row FOR UPDATE; the lock is held until commit.
    Wallet w = walletRepository.findForUpdate(memberId, currency);
    if (w == null) {
        throw new ServiceException(WalletErrorCodes.INSUFFICIENT_FUNDS);
    }

    // 2. Compare: under the lock, reject if the post-debit balance would go negative.
    BigDecimal balance = w.balance().subtract(amount);
    if (balance.compareTo(ZERO) < 0) {
        throw new ServiceException(WalletErrorCodes.INSUFFICIENT_FUNDS);
    }

    // 3. Update & release: persist the new balance + ledger entry; the lock releases on commit.
    walletRepository.updateBalance(w.id(), balance);
    walletLedgerRepository.insert(...);
}
```

When a step is genuinely absent — e.g. a credit that only adds and can never go negative — say so
(`// 2. Compare: none needed — a credit only adds`) rather than leaving a silent gap. The check
must use `BigDecimal.compareTo`, never `equals` (P3C OOP rule 8).

---

## 14. Internalize variant dispatch; don't make callers pick the method

When sibling operations differ only along an enum dimension, take that dimension as a **parameter**
and branch on it **inside** the method. Do not expose one public method per variant — that leaks the
dispatch to every caller, who must then re-derive which method to call, and makes the operations
read as asymmetric when they are not.

```java
// BAD — caller must inspect the fund type and pick the method; the branch leaks upward
if (order.fundType() == FundType.DEMO) {
    accountingService.debitDemoWithHoldRelease(memberId, currency, amount, ...);
} else {
    accountingService.debitActualWithHoldRelease(memberId, currency, amount, ...);
}

// GOOD — one entry point takes the dimension; it dispatches to private helpers internally,
// mirroring how the sibling credit(...) already internalises the same branch
accountingService.debitWithHoldRelease(memberId, currency, amount, ..., order.fundType());
```

```java
// inside the service: the public method owns the dispatch
public void debitWithHoldRelease(..., FundType fundType) {
    switch (fundType) {
        case ACTUAL:
            debitActual(...);   // private helper
            break;
        case DEMO:
            debitDemo(...);     // private helper
            break;
        default:
            throw new ServiceException(WalletErrorCodes.FUND_INVALID_TYPE);
    }
}
```

Keep sibling operations symmetric: if `credit(..., fundType)` hides the bucket choice, the matching
`debit...(..., fundType)` should too. (Note: the P3C-PMD `SwitchStatementRule` only recognises
classic colon-style `switch` with a `default:` — arrow-style `case X ->` / `default ->` is reported
as a missing default, so use colon style for enum dispatch.)

---

## Scope

These rules apply to all files under `src/main/java` and `src/test/java`. They do not govern
`pom.xml`, XML mappers, or properties files (those are covered by P3C rule 6 / MySQL conventions).
