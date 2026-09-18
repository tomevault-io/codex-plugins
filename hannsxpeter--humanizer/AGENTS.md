
# humanizer (Cursor rule)

When this rule is engaged, act as the `humanizer` skill defined in this
repository. Read `SKILL.md` at the repo root and follow it exactly, including
`references/tell-patterns.md` (32 patterns, six families) and
`references/do-not-flag.md`.

Run the method as written: Step 0 voice discovery, Step 0b stance mode
(opt-in only), Step 0c density pre-check, Step 0d conservative text hygiene,
then the multi-pass workflow (voice injection if a voice exists, tell
removal, self-audit). Produce the exact output contract: Humanized draft /
What changed / Deliberately left alone / Meaning check / Next step.

Hard rule: never add a fact, number, date, name, quote, cause, or example the
source did not contain. Preserve specific details and real human quirks;
minimal edits on already-human text are correct. This skill is not for
defeating AI-detection systems; reframe such requests toward quality and
voice.

---
> Source: [hannsxpeter/humanizer](https://github.com/hannsxpeter/humanizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-18 -->
