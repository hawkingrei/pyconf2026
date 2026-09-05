# Sources and presentation scope

## Current deck

The deck contains 25 slides in four parts: storage choices, engine design,
verification and delivery, and rollout. The main narrative is in Chinese.
The slide-by-slide review is recorded in [NARRATIVE_REVIEW.md](NARRATIVE_REVIEW.md).

## Technical sources

Paths beginning with `skein/` refer to the local engine checkout used during
preparation, not files vendored into this presentation repository.

| Subject | Source | Presentation boundary |
| --- | --- | --- |
| Storage roles and module boundaries | `skein/docs/ARCHITECTURE.md` | Graph facts and content are authoritative; search indexes are rebuildable projections. |
| Embedded and host-served integration | `skein/docs/ARCHITECTURE.md`, `skein/docs/specs/EMBEDDED_RUNTIME_SPEC.md` | A host may expose the embedded kernel through its service API; this does not assert a separate server product. |
| SQL/PGQ front end | `skein/docs/specs/POSTGRES_SQL_PGQ_SPEC.md` | Separate language front ends converge after binding; no standalone GQL claim. |
| Optimizer | `skein/crates/optimizer/src/{stage,relational_join,relational}.rs` | Workflow only; estimated cost does not guarantee fastest runtime. |
| Executor | `skein/src/executor/batch.rs`, `skein/crates/executor/src/{pipeline,blocking}.rs` | Bounded batches for streaming work; blocking operators retain state. |
| Parallel work | `skein/crates/runtime-tokio/src/lib.rs`, `skein/crates/executor/src/{concurrent,morsel}.rs`, `skein/src/executor/columnar.rs` | Runtime admission and shared workers apply to eligible execution paths. |
| Transactions | `skein/src/api/concurrent.rs`, `skein/docs/specs/EMBEDDED_RUNTIME_SPEC.md` | Pinned snapshots, private changes, conflict coordination, serialized durable commit/publication; one process shares one root per database path. |
| Retrieval | `skein/docs/ARCHITECTURE.md`, `skein/docs/specs/EMBEDDED_RUNTIME_SPEC.md` | Full-text and vector retrieval are complementary entries; no automatic hybrid-ranking claim. |
| AI-facing integration | Presenter update, 2026-09-05 | AI authors queries and invokes the database through MCP; branches support independent exploration. Specific branch write/merge semantics are not asserted. |
| Formal models | `skein/docs/tla/`, `skein/AGENTS.md` | Model checking concerns the modeled state space, not proof of all implementation behavior. |
| Differential and metamorphic tests | `skein/crates/fuzz/README.md` | Compare plans, equivalent expressions, and storage states; omit the detailed oracle inventory. |
| Delivery incidents | `postmortem/2026-08-28-nmem-server-bazel-skein-bootstrap.md`, `postmortem/2026-08-19-skein-native-gate-pin-drift.md` | Dependency admission and version consistency are distinct delivery lessons. |
| Rollout path | `docs/implementation/SKEIN_EMBEDDED_BOOTSTRAP.md`, `skein/docs/specs/POSTGRES_RELATIONAL_CONTENT_STORE_SPEC.md` | Show validation steps without marking the final authority switch as already completed. |

SQLite comparison references were checked against the official documentation:
[deployment uses](https://www.sqlite.org/whentouse.html),
[recursive queries](https://www.sqlite.org/lang_with.html),
[FTS5](https://www.sqlite.org/fts5.html), and
[loadable extensions](https://www.sqlite.org/loadext.html).
Transactions and recursive CTEs are native capabilities. FTS5 is an official
extension supplied with SQLite sources, enabled at build time or loaded
separately; availability depends on the distribution's build configuration.
Vector retrieval can use a third-party extension such as
[sqlite-vec](https://github.com/asg017/sqlite-vec), requiring separate integration.
The comparison describes Mem's integration needs rather than ranking database
products or claiming that graph traversal is impossible in SQLite.

## Benchmark evidence

All figures are local workload measurements, not production qualification or
end-to-end application speedups.

| Measure | Source | Interpretation |
| --- | --- | --- |
| 28.37x append P50 ratio | `skein/docs/ROW_PAGE_MONOTONIC_APPEND_BENCHMARK.md`, 2026-08-20, macOS arm64 | Batch size 32, enabled candidate versus disabled baseline. `SyncOnCheckpoint` excludes a per-commit fsync comparison. |
| 3.10x adjacency P50 ratio | `skein/docs/EXECUTOR_MORSEL_BENCHMARK.md`, 2026-08-17, Apple M5 Max | Previous local spot check 843.990 us versus 272.464 us at degree 100,000, with LIMIT 50. |
| About 65% lower peak RSS growth | `skein/docs/SEARCH_GENERATION_BENCHMARK.md`, 2026-08-06, Apple M5 Max | 251,789,312 B versus 88,162,304 B for 100,000 documents; one release run per mode. |

The adjacency report supports early cursor termination and bounded executor
state. Degree-32 and degree-100,000 latencies were 14.582 us and 272.464 us;
they are not approximately equal, and the result does not prove constant
latency or constant whole-database memory.

## Presenter-provided status and directions

On 2026-09-05, the presenter reported that Skein is integrated into Nowledge Mem
and gradual rollout has started. Slide 22 states this update. It is not an
independently inspected deployment result or a claim that full rollout is done.

The presenter described MCP query access and branch support for the AI-native
database narrative on slide 15. The slide uses independent task exploration
to explain the purpose of branches. This is presenter-provided capability
context, not independent implementation verification of an MCP server or
branch lifecycle APIs.

The presenter also specified four future directions on 2026-09-05: open
source, mobile support, more PostgreSQL features, and better resource control
with feature selection from phones to servers. Slide 24 presents these as
future work, without adding release dates or claiming completed support.

## Before the talk

- Confirm the event name, city, and date; the deck currently says PyConf 2026.
- Rehearse the 25-slide deck against the confirmed 40-minute slot.
- Refresh operational status with the presenter if the rollout changes.
- Recheck dated benchmarks before substituting new numbers.

## Local commands

```bash
pnpm install --frozen-lockfile
pnpm dev
pnpm build
pnpm export
```

Keep the existing dependency lock and Vite configuration. The current lock
resolves Slidev 52.19.1; do not copy older Vite overrides from another deck.
