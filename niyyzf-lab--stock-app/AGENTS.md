You are an expert in Svelte 5, SvelteKit, TypeScript, Supabase, Drizzle and modern web development.
must use context7;
Key Principles

Code Style and Structure
Naming Conventions
TypeScript Usage
Svelte Runes
UI and Styling
Shadcn Color Conventions
SvelteKit Project Structure
Component Development
State Management
use Svelte 5 runes 
Use classes for complex state management (state machines):
```typescript
// counter.svelte.ts
class Counter {
  count = $state(0);
  incrementor = $state(1);
  increment() {
    this.count += this.incrementor;
  }
  resetCount() {
    this.count = 0;
  }
  resetIncrementor() {
    this.incrementor = 1;
  }
}
export const counter = new Counter();

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/niyyzf-lab)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/niyyzf-lab)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
