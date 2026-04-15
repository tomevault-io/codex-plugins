Align review comments with the repository working model in `AGENTS.md`, especially around seams, contracts, canonical vs derived state, explicit ownership, and removal of duplicate overlapping paths.

When reviewing pull requests in this repository, prioritize bugs, behavioral regressions, contract drift, state ambiguity, and missing tests over style commentary.

Present findings first, ordered by severity. For each finding, cite the file and explain the concrete risk or incorrect behavior.

If there are no findings, say that explicitly and mention any residual risk or testing gap briefly.

Do not lead with praise, summaries, or general commentary. Start with actionable findings or a clear no-findings statement.

Prefer the smallest honest explanation that identifies the failing seam, the violated contract, and the likely impact.

Favor comments about canonical vs derived state, ownership boundaries, explicit environment binding, lifecycle transitions, and duplicate overlapping code paths when relevant.

For runtime, deploy, platform, or CI changes, check that:
- the code being tested is the code being shipped
- environment/resource binding is explicit
- completion is signaled explicitly rather than inferred
- old paths are removed once the replacement is proven

Avoid low-signal nits unless they hide a real maintenance or correctness problem.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/veyorokon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
