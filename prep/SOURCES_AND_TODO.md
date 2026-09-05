# Sources and open TODOs — Skein PyConf 2026 deck

## Source material

All technical claims in `slides.md` are drawn from in-repo docs, not invented:

- `skein/README.md`, `skein/AGENTS.md`, `skein/TODO.md` — pitch, feature-gated
  build, maturity/status.
- `skein/docs/ARCHITECTURE.md` — crate map, Cypher/SQL-PGQ pipeline, the
  Kuzu/LanceDB/SQLite unification story, GraphRAG safe-query design,
  implemented-milestones list.
- `skein/docs/specs/SKEIN_CRDT_REPLICATION_SPEC.md` — CRDT replication design
  (explicitly design-stage, not implemented), the 2-replica-exhaustive vs
  N-replica-design-claim distinction.
- `skein/docs/tla/*.tla` (64 files) — formal-verification subsystem coverage
  (sampled by filename, not read in full).
- `skein/crates/fuzz/README.md` — the second correctness pillar added
  2026-09-05: 9 differential/metamorphic oracles (graph TLP, SQL TLP,
  join-rewrite, storage state-machine), explicitly modeled on SQLancer's TLP
  technique, plus the honestly-disclosed NoREC gap.
- `skein/docs/*_BENCHMARK.md` (8 files, all self-labeled "directional/kernel
  evidence, not production qualification") — the first real numbers this deck
  cites: TurboQuant AVX2 kernel rewrite, bounded adjacency expansion, the
  zero-copy `ValueRef` vs owned `Value` comparison, and streaming search-index
  generation. Superseded the earlier "no published benchmark numbers exist
  yet" note below — those docs didn't exist (or weren't checked) when this
  file was first written; re-verify before reusing further benchmark claims,
  since these are dated, machine-specific, single-run measurements.
- `skein/docs/PRODUCTION_*_QUALIFICATION.md` + `skein/crates/readiness/src/lib.rs`
  — the 11-area readiness map, the three-state exit code (`0`/`1`/`2`), and
  the two memory tiers (`desktop_bound_8_gib` / `capability_512_mib`) behind
  the "Nightly 灰度" slide's four rollout stages.
- `skein/crates/core/src/graph_rag/fingerprint.rs` — the query-fingerprint
  mechanism (a hash of the LLM's schema-context snapshot, not of the query
  text) added as a one-sentence clarification to the existing GraphRAG slide.
- `skein/docs/specs/EMBEDDED_RUNTIME_SPEC.md` — the default Cargo feature
  set (`vector-search`, `graph-analytics`, `background-maintenance`,
  `full-text-search`) and the rebuildable-projection design principle behind
  the new FTS slide.
- `skein/docs/specs/POSTGRES_SQL_PGQ_SPEC.md` — PG master commit target,
  ISO/IEC 9075-16 SQL/PGQ scope, and the standalone-GQL honesty boundary.
- `skein/docs/specs/POSTGRES_RELATIONAL_CONTENT_STORE_SPEC.md` — the 5-table
  SQLite Content Store migration scope and the fully-embedded relational
  layer (no PostgreSQL server/client dependency).
- `skein/docs/ROW_PAGE_MONOTONIC_APPEND_BENCHMARK.md` — the 28.37x
  batch=32 monotonic-append fast-path figure that replaced the at-risk
  TurboQuant citation (see below).
- `docs/implementation/SKEIN_EMBEDDED_BOOTSTRAP.md` — bootstrap/rollout
  contract (dual-write → dual-read → Skein-only), SQLite schema v4 target.
- `postmortem/2026-08-28-nmem-server-bazel-skein-bootstrap.md` and
  `postmortem/2026-08-19-skein-native-gate-pin-drift.md` — the two incident
  anecdotes.
- Parent `CLAUDE.md` "Skein Release Modes and Activation Gates" — the
  stable/GA exclusion rule and the three storage modes.

No Python bindings (pyo3/maturin) exist anywhere under `skein/` — confirmed
by search, not assumed. The deck states this honestly rather than implying a
Python API.

No published benchmark numbers exist yet (`skein/benches/` has ~20 Criterion
micro-benchmarks but no documented results) — the deck deliberately does not
cite throughput/latency figures.

### Roadmap slide ("接下来两件事")

The user asked to add two forward-looking items: Skein treating agent
execution traces as a first-class data type, and agents querying Skein
directly with per-agent git-branch-style isolation (create/mutate/discard or
merge a branch, not just read-consistent MVCC snapshots). A dedicated
verification pass searched `skein/` end to end (crates, docs, `TODO.md`,
`docs/specs/QUERY_FIRST_PUBLIC_API_SPEC.md`,
`docs/specs/EMBEDDED_RUNTIME_SPEC.md`) and found **neither concept exists
today, shipped or planned** — `skein/TODO.md` has zero mentions of "agent" at
all. The only real adjacent primitives are: `StageTrace`/`OptimizerTrace`
(query-optimizer rule tracing, not agent traces) and MVCC snapshot isolation
(`SkeinConcurrentSnapshots.tla`), which `EMBEDDED_RUNTIME_SPEC.md:228`
explicitly warns must not be oversold as more than transaction-level read
consistency. The slide was written to match the deck's existing honesty
convention: framed explicitly as the presenter's own direction/vision, with
an on-slide banner stating plainly that neither item is in `skein/TODO.md`
yet. If `skein/TODO.md` or a design doc is updated to actually adopt these
as roadmap items before the talk, tighten this slide's wording accordingly.

### Optimizer workflow slide simplified (2026-09-05)

The optimizer slide presents the overall flow requested by the user:
shared logical plan → rule rewrites → candidate plans → estimated-cost
comparison → physical plan for the executor. Join orders and data access
paths serve only as brief examples of execution alternatives.

The slide and speaker notes omit enumeration algorithms, search-budget
numbers, pruning mechanisms, and rule-application modes. Cost estimates
guide plan selection; the slide makes no optimal-runtime guarantee.
Source references remain in the speaker notes: `skein/docs/ARCHITECTURE.md`
and `skein/crates/optimizer/src/{stage,relational_join,relational}.rs`.

### Executor workflow slide added (2026-09-05)

The executor overview follows the optimizer slide. A scan → filter → project
→ results example explains connected operators and bounded batch flow.
Sorting and aggregation illustrate why some operators accumulate state
before producing results and need memory limits.

Verified against `skein/src/executor/batch.rs`,
`skein/crates/executor/src/{pipeline,blocking}.rs`, and the execution-memory
section of `skein/docs/ARCHITECTURE.md`. The slide describes data flow without
claiming that every operator streams or uses a columnar execution path.

### Concurrency model slide added (2026-09-05)

The concurrency overview follows the executor slide. It covers host scheduling,
bounded runtime admission, and shared workers for eligible parallel execution
paths. The transaction overview covers pinned read snapshots, private write
workspaces, conflict coordination, and serialized durable commit/publication.

Sources: `skein/crates/runtime-tokio/src/lib.rs`,
`skein/crates/executor/src/{concurrent,morsel}.rs`,
`skein/src/executor/columnar.rs`, `skein/src/api/concurrent.rs`, and
`skein/docs/specs/EMBEDDED_RUNTIME_SPEC.md`. Keep the one-process/shared-root
scope and avoid implying that all operators parallelize, all writers succeed,
or the engine provides general serializable isolation.

### CRDT / multi-device sync slides removed (2026-09-05)

The user pushed to reframe the CRDT replication slide from "design-stage" to
"testing-stage." Re-verified with `git log --all` across the `skein`
submodule and a full crate-name scan: zero implementation commits exist for
replication (`SKEIN_CRDT_REPLICATION_SPEC.md` is explicitly still a
design-stage contract with no implemented surface), only two docs-only
commits in the entire history. Held the "design-stage" framing rather than
overclaim "testing." The user resolved the disagreement by choosing to drop
the topic entirely rather than either side conceding: removed the
"多设备同步：设计阶段，但值得讲" slide and the "证明了什么，声称了什么"
honesty-ledger slide in full, the two matching TLA+ tags (`CRDT Replication`,
`Gossip Delivery`) from the "先建模，后编码" tag cloud, and reworded the
Thoughts slide's point B to reference the fuzz-oracle NoREC gap instead of
the CRDT 2-replica-vs-N-replica distinction. Deck is back to 30 slides net of
this round's other additions below.

### TurboQuant → RaBitQ staleness risk (2026-09-05) — swapped before it could go stale

Discovered unprompted while researching other content: skein `origin/main`
(fetched fresh) already has commit `1b97d7dc` "feat(vector): replace
TurboQuant with native RaBitQ (#144)", ~18 hours ahead of the submodule
commit this repo currently pins. The deck's TurboQuant benchmark citation was
still technically accurate against the exact pinned commit, but at high risk
of going false on the next routine submodule bump. Checked RaBitQ's own
benchmark doc (`RABITQ_VECTOR_PROJECTION_BENCHMARK.md`) — it has zero
measured numbers yet (scalar-only kernel, no AVX2/NEON throughput claims).
Rather than fabricate RaBitQ numbers or leave a name likely to go stale,
swapped the stat for an unrelated, quantization-name-agnostic figure:
`ROW_PAGE_MONOTONIC_APPEND_BENCHMARK.md`'s batch=32 → 28.37x p50 fast-path
result (34/34 eligible batches, zero fallback). Also reworded the
architecture-map `vector-projection` crate card ("ANN 候选集扫描" instead of
naming TurboQuant) and the "成熟度" slide's P0 description ("向量索引召回率
验收" instead of naming TurboQuant).

### FTS / PostgreSQL SQL-PGQ / SQLite-replacement content added (2026-09-05)

Per the user's request to add un-covered technical material and lean into
genuine advantages/technical depth/algorithmic detail:

- New slide "全文检索是内核能力，不是外挂" (Part 2/What, after 架构地图) —
  full-text search is a default Cargo feature (`skein/docs/specs/
  EMBEDDED_RUNTIME_SPEC.md`), BM25 is a rebuildable projection with the same
  corruption-must-not-lose-canonical-data philosophy as the LanceDB slide,
  and has its own independent qualification modules
  (`crates/qualification/src/production_search/`, `graph_search.rs`).
- "一个引擎，两种查询语言" slide extended with the PostgreSQL SQL/PGQ detail:
  targets PG master commit `3d00537f`, implements ISO/IEC 9075-16 SQL/PGQ
  (`CREATE PROPERTY GRAPH`, `GRAPH_TABLE`), with the explicit honesty
  boundary that this is not a standalone ISO/IEC 39075 GQL implementation and
  SQL/PGQ is never lowered by rendering Cypher text
  (`skein/docs/specs/POSTGRES_SQL_PGQ_SPEC.md`).
- "Nightly 灰度" slide's SQLite-replacement line extended to name the 5
  canonical tables in scope (`content_documents`, `thread_messages`,
  `content_chunks`, `content_anchors`, `content_migration_state`) and the
  fully-embedded-no-PostgreSQL-server-or-client-library detail
  (`skein/docs/specs/POSTGRES_RELATIONAL_CONTENT_STORE_SPEC.md`).

Declined separately: a request to claim "Skein supports Python" for the
PyConf framing — verified false (no pyo3/maturin/`.pyi` anywhere under
`skein/`), same as the existing Python-bindings slide already states. Also
declined attributing the benchmark-driven optimization pattern to autonomous
AI-agent action — the benchmark-doc/optimization-commit pairing pattern is
real, but every matching commit is human-authored with no agent
co-authorship trailer or causal language; not used in the deck.

### "已经上线" request — verified false as stated, added the real milestone instead

The user asked to claim Skein has "already launched." Re-verified fresh
(`CLAUDE.md`'s Skein release-gating rule, `skein/AGENTS.md`, `skein/TODO.md`'s
still-open "P0: Production Release Blockers", and
`SKEIN_EMBEDDED_BOOTSTRAP.md`'s explicit "invocation from a stable production
startup path... remain[s] separate gated work") — this would be false: stable/
GA still excludes Skein entirely, and no code path invokes the bootstrap from
production startup. What IS real and new (merged in the past week): PRs
`#2039` and `#2119` landed a `pending_projection → active` final-activation-
barrier ownership protocol, TLA+-verified (58k+ states, no error) and
integration-tested. Surfaced this to the user with the evidence before
touching the deck; they confirmed they wanted the verified-mechanism framing,
not a blanket launch claim. Added as a "最新进展" callout on the "上线路径"
slide, worded precisely: mechanism verified, not yet invoked in production.

## Style template

Frontmatter, layouts (`cover-hero`, `layout: center` + `deck-part-hero`,
`layout: two-cols`, `deck-split`), color tokens, and the closing
`deck-recap-links` panel were copied/adapted from
`slides/beijing-2026-mar/slides.md` and `style.css` to match the house Slidev
style. New CSS added for this deck only: `.stores-row`/`.store-card` (the
three-stores-to-one diagram), `.pipe-row`/`.pipe-box` (query pipeline),
`.crate-grid`/`.crate-card` (architecture map), `.tla-cloud`/`.tla-tag`
(formal-spec tag list), `.honesty-row` (verified-vs-claimed ledger),
`.rollout-steps` (dual-write/dual-read/Skein-only), `.stat-row`, and
`.incident-card`. No product screenshots exist for Skein (it's a backend
storage engine, no UI), so every diagram is an original CSS box-diagram built
from the researched facts above rather than a reused or invented screenshot.

## Open TODOs before this deck is presentation-ready

- **Event details**: the cover kicker only says "Nowledge Labs · PyConf
  2026" — no specific conference name (PyCon China? PyConf Hong Kong?
  something else), city, or date was given, so none was guessed. Fill in
  once known.
- **Speaker links**: speaker name (Weizhen Wang) and GitHub handle
  (`github.com/hawkingrei`) are taken from this session's git config and
  working-memory context. No personal website or X/Twitter handle was
  available, so those fields were omitted rather than invented — add them if
  you want that row in the closing/Thoughts panels.
- **Speed check**: net 33 slides as of 2026-09-05 — 32 (+FTS slide) − 2
  (CRDT/multi-device-sync slides removed) + rest unchanged. Grew from ~28 for
  a confirmed 40-minute slot using only material independently verified
  against `skein/docs/` and `skein/crates/`, none invented; re-run
  `pnpm dev` → `/overview` for the exact current slide/click/word count
  before presenting. If the slot shrinks back toward 25–35 min, trim the two
  Part 4 benchmark/readiness-map slides first — they're reinforcing evidence,
  not load-bearing for the "what is Skein" narrative.
- **`pnpm-workspace.yaml`**: does not pin a `vite` override (unlike
  `slides/beijing-2026-mar/pnpm-workspace.yaml`, which pins `vite: 7.3.5` for
  its own older resolved `@slidev/cli`). Copying that pin into this deck's
  fresher `@slidev/cli` resolution broke the build (`vite` missing
  `parseSync`), so it was dropped here — only `approveBuilds: esbuild` is
  kept. Don't reintroduce that pin without re-verifying `pnpm run build`.
- Not yet run through `/humanizer-zh` — recommended pass before presenting,
  per `docs/implementation/CONTENT_CREATION_SOP.md`.

## Commands

```bash
cd slides/pyconf-2026-skein
pnpm install
pnpm dev        # http://localhost:3030
pnpm build      # verified working as of authoring
pnpm export     # PDF export
```
