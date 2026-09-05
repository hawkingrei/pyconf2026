# Narrative review

Reviewed every slide and its speaker notes from commit `a75ba3e`.
The final deck has 25 slides, reduced from 34, in four parts.
The maturity update remains slide 22.

## Narrative ownership

Each substantive slide introduces one decision, mechanism, or piece of
evidence. The cover and section dividers provide orientation. They do not
repeat the detailed explanations. A single closing slide holds the links.

1. Storage choices: existing roles, the SQLite tradeoff, coordination costs.
2. Engine design: responsibilities, deployment, query flow, optimization,
   execution, concurrency, retrieval, and AI interaction through MCP and branches.
3. Verification and delivery: state models, result comparisons, delivery
   incidents, and workload-specific measurements.
4. Rollout: current integration, the validation path, and future directions.

## Review of every original slide

| Original | Final | Purpose and decision |
| ---: | ---: | --- |
| 1 | 1 | Keep the cover and visual thesis; remove the repeated opening pitch from the notes. |
| 2 | 2 | Align the agenda with four parts and the actual order of evidence. |
| 3 | 3 | Keep the problem-section divider; shorten the transition. |
| 4 | 4 | Explain only the old storage roles and their application-level coordination. |
| 5 | 5 | Label native SQL, official FTS5, and third-party vector extensions separately, then explain Mem's remaining integration work. |
| 6 | 6 | Explain concrete write, update, read, and recovery costs; distinguish graph facts from derived search indexes. |
| 7 | 8, 9 | Merge deployment and engine requirements into the architecture and deployment pages. |
| 8 | 7 | Keep the engine-section divider. |
| 9 | 8 | Combine the engine definition with a grouped architecture map; remove the repeated three-stores-to-one graphic. |
| 10 | 9 | Limit this page to embedding versus host-served integration; leave parallelism and resource admission to concurrency. |
| 11 | 10 | Show language convergence; keep standard/version details out of the main flow. |
| 12 | 11 | Preserve the optimizer workflow; remove bullets that merely repeat the diagram. |
| 13 | 12 | Explain batch flow and blocking work; remove the repeated operator-connection definition. |
| 14 | 13 | Keep execution parallelism and transaction coordination distinct; remove the repeated snapshot callout. |
| 15 | 8 | Merge the module inventory into the grouped architecture map. |
| 16 | 14 | Explain keyword versus semantic retrieval, with one authoritative explanation of index rebuildability. |
| 17 | 15 | Follow the presenter's AI-native framing: AI authors and runs queries through MCP, with branches for independent task exploration. Remove the schema-validation implementation details. |
| 18 | 16 | Broaden the section title to cover verification and delivery, including performance measurement. |
| 19 | 17 | Replace model counts and the unrelated cross-store statistic with commit, snapshot, and reclamation scenarios. |
| 20 | 18 | Present plan, expression, and state comparisons; remove the oracle inventory and repeated gap discussion. |
| 21 | 21 | Use the status divider to preview topics, leaving the integration milestone to the next page. |
| 22 | 22 | Preserve the presenter's concise integration and gradual-rollout statement. |
| 23 | 20 | Move measurement evidence into verification; label latency and memory separately and correct the constant-latency implication. |
| 24 | 23 | Explain the gradual switch in data authority; remove implementation controls and the misleading completed-final-stage highlight. |
| 25 | 23 | Merge the general validation principle into rollout; omit the internal domain and exit-code inventory. |
| 26 | 19 | Move the two delivery lessons into verification and give each a distinct takeaway. |
| 27 | 24 | Replace the former exploration ideas with the presenter's four directions: open source, mobile support, more PostgreSQL features, and resource control with feature selection from phones to servers. |
| 28 | Removed | Remove the audience-specific section divider at the presenter's request. |
| 29 | Removed | Remove the language-binding disclaimer page. |
| 30 | Removed | Remove the language-migration story at the presenter's request. |
| 31 | Removed | Remove the language-ecosystem comparison at the presenter's request. |
| 32 | Removed | Remove the first repeated closing summary. |
| 33 | Removed | Remove the second repeated closing summary, unsupported absolute comparisons, and duplicate links. |
| 34 | 25 | Keep one thank-you and links page; stop repeating the rollout milestone. |

## Consistency corrections

- Graph facts and original content remain authoritative; search projections
  can be rebuilt. The previous text called both graph and vector data indexes.
- Model checking covers the modeled state space. Normal and intentionally
  broken configurations have different expected outcomes.
- The adjacency benchmark compares roughly 14.6 us with 272 us across degrees;
  those latencies are not nearly equal. The supported conclusion concerns
  bounded cursor work and executor state.
- The memory result is about peak RSS growth, not a throughput multiplier.
- The rollout diagram shows a validation path, without asserting the current
  mode or completed authority migration.
- Remove references to deleted slides, repeated disclaimers, and duplicate
  closing link panels. All audience-facing explanations remain in Chinese.

## Verification

- `pnpm build` completed successfully with the existing lock and configuration.
- Slidev parsing confirmed 25 slides, four sections, unique titles, the
  maturity update on slide 22, and the four future directions on slide 24.
- Repository text search found no remaining content from the removed
  language-specific section.
- Browser inspection covered all 25 slides at 980 by 552 with click reveals
  fully shown. No text extended beyond the slide bounds; edited diagrams and
  content layouts were also inspected visually.
- Source details and operational-status provenance are recorded in
  `SOURCES_AND_TODO.md`.
