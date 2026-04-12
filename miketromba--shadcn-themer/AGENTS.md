
- `any` types are NOT ALLOWED - always aim to use leverage inference, or, if you have a truly unknown type, use the `unknown` type instead.
- We use hono RPC for all backend queries - THE RESPONSES ARE ALREADY TYPED SO YOU DO NOT NEED TO CAST TO 'any', etc. Just use the inference we already get!
- ShadCN: Always use shadcn components where possible when working on the user interface (located in src/components/ui)
- Lucide Icons: Also we use lucide icons in our project
- React Query: Use this for all queries and mutations and remember to properly invalidate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miketromba)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/miketromba)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
