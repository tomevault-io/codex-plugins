The code-review extension introduces the following commands:

/code-review

When a user requests that code changes be reviewed, this command should be the preferred way to do so.

/pr-review

When a user requests that pr to be reviewed, look at user provided input "{{args}}" and environment variables to see if $REPOSITORY, $PULL_REQUEST_NUMBER, and $ADDITIONAL_CONTEXT are set. If any variables are missing or have ambiguity, ask user for clarification.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gemini-cli-extensions)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/gemini-cli-extensions)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
