# MySQL Rules

Verbatim excerpts from https://alibaba.github.io/Alibaba-Java-Coding-Guidelines/

---

## Table Schema Rules

**1. [Mandatory]** Columns expressing the concept of True or False must be named as `is_xxx`, whose data type should be `unsigned tinyint` (1 = True, 0 = False).

> Note: All columns with non-negative values must be unsigned.

**2. [Mandatory]** Names of tables and columns must consist of lowercase letters, digits, or underscores. Names starting with digits, and names that contain only digits between two underscores, are not allowed.

> Positive example: `getter_admin`, `task_config`, `level3_name`
>
> Counter example: `GetterAdmin`, `taskConfig`, `level_3_name`

**3. [Mandatory]** Plural nouns are not allowed as table names.

**4. [Mandatory]** Reserved keywords (`desc`, `range`, `match`, `delayed`, etc.) should not be used. Reference the MySQL official documentation.

**5. [Mandatory]** Index naming:
- Primary-key index: `pk_<column_name>`
- Unique index: `uk_<column_name>`
- Normal index: `idx_<column_name>`

> `pk` = primary key, `uk` = unique key, `idx` = index.

**6. [Mandatory]** Decimals should be typed as `DECIMAL`. `FLOAT` and `DOUBLE` are not allowed.

> Note: `FLOAT` and `DOUBLE` may have precision loss when stored, leading to incorrect data comparison results.

**7. [Mandatory]** Use `CHAR` if the lengths of information stored in that column are almost the same.

**8. [Mandatory]** The length of `VARCHAR` should not exceed 5000; otherwise, define it as `TEXT` and store it in a separate table to avoid effects on indexing efficiency of other columns.

**9. [Mandatory]** A table must include three columns: `id`, `gmt_create`, `gmt_modified`.

> Note: `id` is the primary key, `UNSIGNED BIGINT`, auto-incrementing with step length 1. The type of `gmt_create` and `gmt_modified` should be `DATETIME`.

**10. [Recommended]** Define table name as `<table_business_name>_<table_purpose>`.

> Positive example: `tiger_task`, `tiger_reader`, `mpp_config`

**11. [Recommended]** Try to define the database name same as the application name.

**12. [Recommended]** Update column comments once column meaning changes or new possible status values are added.

**13. [Recommended]** Some appropriate columns may be stored in multiple tables redundantly to improve search performance, but consistency must be maintained. Redundant columns should **not** be:
1. Columns with frequent modification.
2. Columns typed as very long `VARCHAR` or `TEXT`.

> Positive example: Product category names are short, frequently used, and almost never change. They may be stored redundantly in relevant tables to avoid join queries.

**14. [Recommended]** Database sharding is only recommended when there are more than **5 million rows** in a single table, or table capacity exceeds **2 GB**.

> Note: Do not shard during table creation if the anticipated data quantity is not expected to reach this scale.

**15. [For Reference]** Appropriate column length not only saves storage but also improves query efficiency. Pick the narrowest integer type that fits the value range (e.g. `TINYINT` for ages 0–127, `SMALLINT` for years).

---

## Index Rules

**1. [Mandatory]** Unique index should be used if the business logic is applicable.

> Note: Negative impact on insert efficiency is negligible; query speed improves significantly.

**2. [Mandatory]** `JOIN` is not allowed if more than three tables are involved. Columns to be joined must have absolutely similar data types. Make sure joined columns are indexed.

> Note: Indexing and SQL performance should be considered even if only 2 tables are joined.

**3. [Mandatory]** Index length must be specified when adding an index on `VARCHAR` columns. The index length should be set according to the data distribution.

> Note: Normally for `CHAR` columns, an index length of 20 can distinguish more than 90% of the data, calculated by `count(distinct left(column_name, index_length)) / count(*)`.

**4. [Mandatory]** `LIKE '%…'` or `LIKE '%…%'` is not allowed when searching with pagination. Use a search engine if it is really needed.

> Note: Index files have B-Tree's leftmost-prefix matching property. The index cannot be applied if the leftmost prefix value is not determined.

**5. [Recommended]** Make use of index order when using `ORDER BY`. The last columns of the `ORDER BY` clause should be at the end of a composite index.

> Positive example: `WHERE a = ? AND b = ? ORDER BY c;` — index `a_b_c`.
>
> Counter example: The index order will not take effect if the query condition contains a range, e.g. `WHERE a > 10 ORDER BY b;` cannot use index `a_b`.

**6. [Recommended]** Make use of *covering index* for queries to avoid additional lookups after searching the index.

> Note: If you only need to check the title of Chapter 11 of a book, you don't need to turn to the page — the table of contents already has the title. That's the covering index.
>
> Positive example: When the `EXPLAIN` result shows `Using index` in the Extra column, you've got a covering-index plan.

**7. [Recommended]** Use late join or sub-query to optimize deep-pagination scenarios.

> Note: Instead of bypassing offset rows, MySQL retrieves `offset+N` rows total, then drops `offset` and returns `N`. Very inefficient when offset is huge.
>
> Positive example:
> ```sql
> SELECT a.*
> FROM table1 a
> INNER JOIN (
>     SELECT id FROM table1 WHERE <conditions> LIMIT 100000, 20
> ) b ON a.id = b.id;
> ```

**8. [Recommended]** The target of SQL performance optimization is that the `EXPLAIN` result type reaches `REF` level — `RANGE` at least, or `CONSTS` if possible.

> Counter example: Pay attention to the `index` type in `EXPLAIN`. A full scan of the index file is nearly as slow as a full table scan.

**9. [Recommended]** Put the most discriminative column to the leftmost position when adding a composite index.

> Positive example: For `WHERE a = ? AND b = ?`, if data in column `a` is nearly unique, index `idx_a` is enough.

**10. [For Reference]** Avoid these misunderstandings when adding indexes:
1. False: each query needs one index.
2. False: indexes consume too much storage and degrade update/insert significantly.
3. False: unique constraints should all be enforced in the application layer via "check then insert".

---

## SQL Rules

**1. [Mandatory]** Do not use `COUNT(column_name)` or `COUNT(constant)` in place of `COUNT(*)`. `COUNT(*)` is SQL-92 defined standard syntax to count rows.

> Note: `COUNT(*)` counts `NULL` rows; `COUNT(column_name)` does not.

**2. [Mandatory]** `COUNT(DISTINCT column)` counts distinct non-NULL values. `COUNT(DISTINCT col1, col2)` returns 0 if all values of one column are `NULL`, even if the other column has distinct non-NULL values.

**3. [Mandatory]** When all values of one column are `NULL`, `COUNT(column)` returns 0 while `SUM(column)` returns `NULL`. Watch out for `NullPointerException` when consuming `SUM()`.

> Positive example: `SELECT IF(ISNULL(SUM(g)), 0, SUM(g)) FROM table;`

**4. [Mandatory]** Use `ISNULL()` to check `NULL`. Result is `NULL` when comparing `NULL` with anything else.

> Notes:
> - `NULL <> NULL` returns `NULL`, not `false`.
> - `NULL = NULL` returns `NULL`, not `true`.
> - `NULL <> 1` returns `NULL`, not `true`.

**5. [Mandatory]** When coding DB pagination, return immediately once count is 0 — don't run the page query.

**6. [Mandatory]** Foreign keys and cascade updates are not allowed. All foreign-key logic should be handled in the application layer.

> Note: Foreign keys and cascading updates suit single-machine, low-concurrency systems — not distributed, high-concurrency clusters.

**7. [Mandatory]** Stored procedures are not allowed. They are difficult to debug, extend, and not portable.

**8. [Mandatory]** When correcting data — `DELETE` or `UPDATE` — `SELECT` first to ensure data correctness.

**9. [Recommended]** `IN` clauses should be avoided. If unavoidable, the size of the IN-list should be evaluated and kept under **1000**.

**10. [For Reference]** For globalization, characters should be represented and stored as UTF-8. Be cautious of character-vs-byte counting.

> Note: `SELECT LENGTH('轻松工作');` returns 12 (bytes). `SELECT CHARACTER_LENGTH('轻松工作');` returns 4 (characters). Use `UTF8MB4` to store emoji if needed.

**11. [For Reference]** `TRUNCATE` is not recommended even though it is faster than `DELETE` and uses less log/transaction resources. `TRUNCATE` is not transactional and does not fire triggers — problems may occur.

> Note: Functionally, `TRUNCATE TABLE` is similar to `DELETE` without `WHERE`.

---

## ORM Rules

**1. [Mandatory]** Specify column names in queries — do not use `*`.

> Notes:
> 1. `*` increases parsing cost.
> 2. It may introduce mismatch with `resultMap` when columns are added or removed.

**2. [Mandatory]** Boolean POJO properties cannot be prefixed with `is`, while DB column names *should* prefix with `is`. A mapping between properties and columns is required.

> Note: Refer to POJO and DB-column rules. Mapping is needed in `resultMap`. Code generated by MyBatis Generator may need adjustment.

**3. [Mandatory]** Do not use `resultClass` as the return parameter, even if all class property names match DB columns. A corresponding DO definition is needed.

> Note: Mapping configuration decouples DO definition from table columns, easing maintenance.

**4. [Mandatory]** Be cautious with parameters in XML configuration. Do not use `${}` in place of `#{}` / `#param#`. SQL injection may happen otherwise.

**5. [Mandatory]** iBatis built-in `queryForList(String statementName, int start, int size)` is not recommended.

> Note: It causes OOM because it retrieves *all* records for `statementName`, then applies `start, size` via `subList`.
>
> Positive example: Use `#start#`, `#size#` in `sqlmap.xml`:
> ```java
> Map<String, Object> map = new HashMap<>();
> map.put("start", start);
> map.put("size", size);
> ```

**6. [Mandatory]** Do not use `HashMap` or `Hashtable` as the DB query result type.

**7. [Mandatory]** `gmt_modified` should be updated with the current timestamp simultaneously with the DB record update.

**8. [Recommended]** Do not define a universal "update table" interface that accepts a POJO and always updates every column. Update only the intended columns.

**9. [For Reference]** Do not overuse `@Transactional`. Transactions affect QPS of the DB, and relevant rollback behavior must be considered.

**10. [For Reference]** `<isEqual>`'s `compareValue` is a constant (usually a number) compared against the property value. `<isNotEmpty>` runs when the property is not empty and not null. `<isNotNull>` runs when the property is not null.
