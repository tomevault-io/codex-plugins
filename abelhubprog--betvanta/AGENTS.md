## BetVanta — AI agent guide

Concise, repo-specific notes so an AI can ship quickly and safely.

## Architecture (what’s here)
- Monorepo:
  - `contracts/` Solidity (core, pools, governance, interfaces, oracles) + tests in `contracts/test` (Hardhat style).
  - `frontend/` Next.js 14 (App Router) + Tailwind + shadcn-like UI + ethers for Web3.
  - `shared/` TS types/utils used by the frontend.
  - `backend/` scaffold only (folders exist; no framework/entrypoint yet).

## Core behavior (what must stay true)
- Bet lifecycle in `contracts/core/BetVantaCore.sol`:
  - Events: `BetRequested`, `BetAccepted`, `BetSettled`.
  - States: `PENDING → ACCEPTED → SETTLED` (enum); `settleBet` is `onlyOwner`.
  - Uses ERC-20 `vantaToken`; callers must approve before `requestBet`.

## Dev workflows (how to run)
- Frontend app lives in `frontend/`:
  - Install: pnpm -w install
  - Dev: pnpm --filter @betvanta/frontend dev
  - Build: pnpm --filter @betvanta/frontend build
- Contracts (Hardhat-style tests present):
  - Compile/Test e.g.: pnpm hardhat compile | pnpm hardhat test (consider moving hardhat deps under `contracts/`).
- Shared lib: pnpm --filter @betvanta/shared build

## Conventions
- Frontend code uses `@/*` path alias from the repo root (see root `tsconfig.json`).
- Web3 logic lives in hooks: `frontend/src/hooks/useWeb3.ts`, `useBetting.ts`.
- UI forms: zod + react-hook-form + toast (see `PlaceBetForm.tsx`).

## Integration notes & gotchas
- Ethers mismatch: code uses v5 patterns (`ethers.providers.Web3Provider`, `ethers.utils.parseEther`) but packages in `frontend/`/`shared/` are v6.
  - Pick one: upgrade hooks to v6 (`BrowserProvider`, `parseEther`) or pin ethers v5 everywhere.
- ABI import: `useBetting.ts` imports `@/abis/BetVantaCore.json`, but no `abis/` exists at repo root; add it or fix the path.
- Contract surface: `useBetting.ts` expects `getBets()`, but `BetVantaCore.sol` doesn’t expose it; either add a view or fetch via events/mapping.
- Solidity imports: `contracts/core/BetVantaCore.sol` uses `./interfaces/...` but interfaces live in `contracts/interfaces/` (relative path likely `../interfaces/...`).
- Root `pnpm run dev` may fail; run the Next app from `frontend/` instead.

## Good references
- Forms/UI: `frontend/src/components/betting/PlaceBetForm.tsx`
- Web3 patterns: `frontend/src/hooks/useWeb3.ts`, `frontend/src/hooks/useBetting.ts`
- Contract design/events: `contracts/core/BetVantaCore.sol`, tests in `contracts/test/BetVantaCore.test.js`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Abelhubprog)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Abelhubprog)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
