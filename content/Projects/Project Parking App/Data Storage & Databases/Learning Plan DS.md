### Week 38
[[Week 38 DS]]
- Learn: Schemas, roles, privileges.
- Learn: SERIAL vs IDENTITY, sequences.
- Learn: Native types (arrays, enums, JSON/JSONB).
- Hands-on: create a schema `parking`, with a `scans` table that uses JSONB for flexible metadata.
#### Tasks
- [x] Screenshot of your Postgres schema + sample JSONB insert.
- [ ] Write-up: “Schemas vs Databases in Postgres – how they compare to MSSQL.”
- [x] Reflection: BYTEA vs file paths for images – pros & cons.

### Week 39
- Learn: Index types (B-Tree, Hash, GIN, BRIN).
- Hands-on: create a GIN index on your JSONB metadata and query with `@>` operator.
- Learn: Common Table Expressions (CTEs) & Window Functions.
- Hands-on: Write a query to find the “most frequently scanned car” using `ROW_NUMBER()` or `RANK()`.
#### Tasks
- [ ] Screenshot of GIN index creation + query using JSONB operator.
- [ ] Query result screenshot for “top scanned cars” with a window function.
- [ ] Short reflection: difference between B-Tree and GIN indexes in practice.

### Week 40
- Learn: Transactions & ACID properties.
- Hands-on: run multi-step queries with `BEGIN/COMMIT/ROLLBACK`.
- Practice: simulate a failed transaction and confirm rollback works.
- Learn: Backup & restore with `pg_dump` / `pg_restore`.
#### Tasks
- [ ] Screenshot of a rollback test where data isn’t committed.
- [ ] Demo of database backup & restore with `pg_dump`.
- [ ] Write-up: why ACID matters for your number plate app.

### Week 41
- Learn: Views & materialized views.
- Hands-on: create a view for “scans in last 24h.”
- Experiment: store flexible data with JSONB vs relational table, compare queries.
#### Tasks
- [ ] Screenshot of your 24h scans view.
- [ ] Comparison query: JSONB vs relational table — show timing/plan.
- [ ] Reflection: trade-offs between JSONB flexibility vs relational integrity.