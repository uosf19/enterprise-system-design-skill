# Primary sources for production implementation guidance

This register accompanies derived guidance, not book excerpts or passing implementation tests. S01–S39 were consulted on 20 November 2026; WP-S01–WP-S10 on 21 November 2026. Applicability notes are synthesis. Verify changing platform details before implementation. Project-specific plans and local validation records are not distributed with this reusable skill.

## Primary source register

| ID | Primary source | Application and limitation |
| --- | --- | --- |
| S01 | [Vattani, Chierichetti, Lowenstein: Optimal Probabilistic Cache Stampede Prevention](https://cseweb.ucsd.edu/~avattani/papers/cache_stampede.pdf) | Exponential early refresh, negative logarithm, probabilistic rather than exclusive execution. |
| S02 | [PostgreSQL row security](https://www.postgresql.org/docs/18/ddl-rowsecurity.html) | Owner/bypass roles and integrity checks require separate review. |
| S03 | [CREATE POLICY](https://www.postgresql.org/docs/18/sql-createpolicy.html) | Read filtering and insert/update checks are distinct policy concerns. |
| S04 | [SET LOCAL](https://www.postgresql.org/docs/18/sql-set.html) | Transaction and savepoint lifetime of tenant context. |
| S05 | [PostgreSQL constraints](https://www.postgresql.org/docs/18/ddl-constraints.html) | Composite references and scoped uniqueness. |
| S06 | [Npgsql basic usage](https://www.npgsql.org/doc/basic-usage.html) | Data sources, connection pooling, transaction connection ownership. |
| S07 | [Npgsql performance](https://www.npgsql.org/doc/performance.html) | Connection reset and the risks of disabling it. |
| S08 | [EF Core connection resiliency](https://learn.microsoft.com/en-us/ef/core/miscellaneous/connection-resiliency) | Retried transaction unit and uncertain commit outcomes; examples may target another provider. |
| S09 | [EF Core transactions](https://learn.microsoft.com/en-us/ef/core/saving/transactions) | Explicit transactions, savepoints and shared connection requirements. |
| S10 | [PostgreSQL transaction isolation](https://www.postgresql.org/docs/18/transaction-iso.html) | Complete transaction retry and concurrency anomalies. |
| S11 | [PostgreSQL explicit locking](https://www.postgresql.org/docs/18/explicit-locking.html) | Row-lock lifetime and consistent lock ordering. |
| S12 | [SELECT / SKIP LOCKED](https://www.postgresql.org/docs/18/sql-select.html) | Useful for competing queue consumers, not a consistent business query. |
| S13 | [Sequence functions](https://www.postgresql.org/docs/18/functions-sequence.html) | Allocation is not rollback-safe/gapless. Commit-order counterexample below is our deduction. |
| S14 | [Logical decoding concepts](https://www.postgresql.org/docs/18/logicaldecoding-explanation.html) | Snapshot/stream boundary, redelivery and retained-WAL risks; CDC remains an alternative. |
| S15 | [PostgreSQL index-only scans](https://www.postgresql.org/docs/18/indexes-index-only-scans.html) | Heap storage and visibility requirements differ from InnoDB. |
| S16 | [PostgreSQL multicolumn index combinations](https://www.postgresql.org/docs/18/indexes-bitmap-scans.html) | Plan-dependent use including skip scans, not absolute prefix bans. |
| S17 | [PostgreSQL TOAST](https://www.postgresql.org/docs/18/storage-toast.html) | Large values may be kept out of line and need not be fetched by unrelated projections. |
| S18 | [CREATE INDEX](https://www.postgresql.org/docs/18/sql-createindex.html) | Concurrent build limitations and partial unique indexes. |
| S19 | [ALTER TABLE](https://www.postgresql.org/docs/18/sql-altertable.html) | DDL locks, validation and rewrite behavior depend on the operation. |
| S20 | [PostgreSQL continuous archiving/PITR](https://www.postgresql.org/docs/18/continuous-archiving.html) | Base backup plus required WAL chain; logical dumps are not that chain. |
| S21 | [SQLite synchronous](https://sqlite.org/pragma.html#pragma_synchronous) | WAL/FULL versus power-loss durability under NORMAL. |
| S22 | [SQLite WAL](https://sqlite.org/wal.html) | Live WAL state, checkpointing and published WAL-reset correction. |
| S23 | [SQLite backup API](https://sqlite.org/backup.html) | Consistent snapshot rather than copying a live main file alone. |
| S24 | [Drift encryption](https://drift.simonbinder.eu/platforms/encryption/) | Current native encrypted path and runtime cipher verification. |
| S25 | [Drift transactions](https://drift.simonbinder.eu/dart_api/transactions/) | Await all database work within the transaction. |
| S26 | [Drift isolates](https://drift.simonbinder.eu/isolates/) | Synchronous native DB I/O outside the UI isolate. |
| S27 | [ASP.NET Core rate limiting](https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit?view=aspnetcore-10.0) | Partitioned admission/concurrency policies; no unmeasured limits adopted. |
| S28 | [AWS: timeouts, retries and jitter](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/) | Bound retry amplification and dependency deadlines. |
| S29 | [Google SRE: alerting on SLOs](https://sre.google/workbook/alerting-on-slos/) | Error budgets and actionable burn-rate alerts. |
| S30 | [OWASP multi-tenant security](https://cheatsheetseries.owasp.org/cheatsheets/Multi_Tenant_Security_Cheat_Sheet.html) | Scope isolation beyond SQL, including caches/files/background work. |
| S31 | [RFC 8785](https://www.rfc-editor.org/rfc/rfc8785.html) | Canonical JSON constraints; not a business equivalence algorithm. |
| S32 | [OpenAPI 3.1.1](https://spec.openapis.org/oas/v3.1.1.html) | Existing contract version, not a claim to be the latest release. |
| S33 | [Redis eviction](https://redis.io/docs/latest/reference/eviction/) | Actual eviction policies; do not attribute Caffeine's W-TinyLFU design to stock Redis. |
| S34 | [Kafka design](https://kafka.apache.org/41/design/design/) | Page cache/zero-copy transfer versus disk persistence. |
| S35 | [Prometheus storage](https://prometheus.io/docs/prometheus/latest/storage/) | WAL and mmap head chunks are different structures. |
| S36 | [Spanner external consistency](https://docs.cloud.google.com/spanner/docs/true-time-external-consistency) | Transactional ordering is stronger than single-object linearizability. |
| S37 | [Spanner transactions](https://docs.cloud.google.com/spanner/docs/transactions) | Read-write concurrency can use locks; TrueTime does not remove all locking. |
| S38 | [RFC 9000](https://www.rfc-editor.org/rfc/rfc9000.html) | Variable-length CID and endpoint connection state. |
| S39 | [Vercel function limits](https://vercel.com/docs/functions/limitations) | Runtime/configuration-specific duration and streaming limits, not a universal 60s ceiling. |
| WP-S01 | [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) | Deny by default, verify each operation/resource and scope; preserve ERP's approved permission defaults and explicit restrictions. |
| WP-S02 | [OWASP CSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) | Cookie-authenticated browser mutation controls; SameSite/CORS alone are not a complete authorization/CSRF design. |
| WP-S03 | [ASP.NET Core 10 Data Protection configuration](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview?view=aspnetcore-10.0) | Persist/isolate/protect server key rings and test recovery/rotation. This is not the native database encryption or offline lease-signing key. |
| WP-S04 | [OWASP File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) | Hostile content, bounded parsing, generated names, storage outside public roots and authenticated retrieval; file type alone does not establish safety. |
| WP-S05 | [OWASP Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) | Session fixation/rotation, expiry and revocation lifecycle; choose exact values at 0B rather than inventing policy here. |
| WP-S06 | [OWASP Logging](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html) | Security/audit events with controlled access and redaction; exclude credentials, keys and unnecessary personal/payment data. |
| WP-S07 | [ASP.NET Core Identity API authorization](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-api-authorization?view=aspnetcore-10.0) | Platform authentication primitives do not by themselves implement device enrollment, offline authority, business grants or the full native credential contract. |
| WP-S08 | [OWASP SSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html) | If remote-source imports/provider adapters are enabled, constrain destinations/redirects/resolution and outbound access. No arbitrary URL fetch feature is introduced. |
| WP-S09 | [OWASP Software Supply Chain Security](https://cheatsheetseries.owasp.org/cheatsheets/Software_Supply_Chain_Security_Cheat_Sheet.html) | Dependency/artifact provenance and release integrity; actual signing/distribution/OS evidence remains platform work. |
| WP-S10 | [OWASP CSV Injection](https://community.owasp.org/attacks/CSV_Injection) | Treat spreadsheet formulas from imported/exported content as hostile; verify the target format/reader rather than assuming ordinary CSV quoting prevents execution. |
