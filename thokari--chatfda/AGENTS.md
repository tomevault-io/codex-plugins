# General

# Code
## Design
This project should demonstrate good code organisation and design.
Please try to keep the code simple, small, and concise. Don't include behaviour beyond what is being discussed. Álways produce minimal code.

## Formatting
We use 4 spaces indentation for typescript, but for configuration files, we keep them as they were generated. We do not use semicolons, but as with indentation, we keep them in configuration files.

## Comments
When there are comments in the code, you keep them as they are, unless you need to update them to reflect changes in the code. Do not add comments to describe what changed, or refer to deleted code. Use them judicously, and prefer factoring to smaller functions instead.

# Domain
## FDA Labels and Chunks
The idea of deduplicating chunks by label does not make a lot of sense, because chunks are segments of text from a single label. So two chunks from a label are probably different, and both relevant to a query, if this occurs.
Deduplication in the context of this data means that we remove or rank lower chunks that have similar text, but come from a different manufacturer. The challenge is getting more diversity into the chunks...

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thokari)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/thokari)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
