# Flare Hackathon (ETH Oxford 2026) — Repo Context + Plan Mapping (Read Only)

## Purpose of this prompt
You are GitHub Copilot working inside a repo that currently contains an **empty / template Next.js React app**.  
The goal of this prompt is to give you full context of what we are trying to build for the **Flare Hackathon**, what the intended architecture is, what constraints we’ve agreed, and what the next steps are — so you can quickly map the plan to the repo and understand what needs adding.

**Important:** Do NOT implement anything yet.  
Do NOT propose fixes or write code in response.  
Just use this as context to understand the plan and the repo shape.

---

## Hackathon Context (Flare)
This project is for the Flare hackathon (ETH Oxford 2026).

### MAIN TRACK requirement
The dApp must use at least one Flare Enshrined Data Protocol:
- FTSO (Flare Time Series Oracle) price feeds
- FDC (Flare Data Connector)
- Secure Random Number
- FAssets (FXRP)

### Our intended approach
We are targeting MAIN TRACK using **FTSO** (price feeds) as the required Flare protocol.

### Judging criteria we are aligning to
- Must use at least one Flare data protocol (FTSO)
- Must solve a real-world problem / be impactful
- Must add “Feedback building on Flare” in README

---

## Product / Use Case (Insurtech-based)
We are coming from an insurtech background (no Web3 experience).

We want to build a dApp conceptually similar to:
- claim created
- payout is pending
- payout is executed when price conditions are met

### Core idea
Insurance companies (or insurers holding crypto) want to pay claims when the market price is optimal.

Example:
- Claim is created with a payout amount and a target price trigger.
- Contract uses Flare FTSO to validate the current price at execution time.
- When the price hits the insurer-defined “best time” (or stop loss / take profit rules), the claim can be paid.

---

## Web3 Concepts We Clarified
### Smart contract meaning
In Web3, “smart contract” = a deployed on-chain application + persistent state.
It is NOT “one contract per insurance claim”.

### One smart contract stores many claims
We will deploy **one** contract (e.g. `ClaimsPayoutManager`) and it will store many claim records inside its storage (mapping/array).

### Paying a claim does NOT destroy the contract
A claim can remain pending for an arbitrary time.  
When paid, the claim state is updated (e.g. `paid = true`), and an event is emitted.

### Smart contracts cannot run background jobs
Contracts do not “wake up”.
A transaction must be submitted to execute logic.

Therefore:
- Manual execution via UI is acceptable for MVP
- A keeper bot / worker can be added later to automate execution
- Anyone can trigger execution if the contract validates the rules itself

---

## Data Storage Approach (Hackathon-friendly)
We explicitly decided:
- No backend DB is required for hackathon
- Claims + payments should be stored on-chain in contract state
- Events should be emitted to allow listing history in the UI

### Listing after app restart
We want to support:
- Close app → reopen → load claims again

Mechanisms:
1) Read contract storage:
   - `nextClaimId`
   - `claims[id]`
2) Query contract events:
   - `ClaimCreated`
   - `ClaimPaid`

We also discussed a possible minimal DB index for IDs, but agreed it is not required for hackathon MVP.

---

## Tech Stack Intent
### Frontend
- Next.js (React)
- Ethers.js (or wagmi/viem later, but currently ethers is simplest)
- MetaMask wallet connect
- Client-side pages are fine (SSR not required)

### Blockchain
- Flare Coston2 testnet for development
  - chainId: 114
  - RPC: https://coston2-api.flare.network/ext/C/rpc
  - faucet: https://faucet.flare.network/coston2
- Eventually can be mainnet Flare chainId 14

### Contracts
- Hardhat (TypeScript) as the contract toolchain
- Separate `/contracts` folder at repo root (not under Next.js `src`)
- This repo currently does NOT contain a Hardhat project yet

### Deployment
- Next.js frontend can be deployed to Azure Static Web Apps (client-side)
- Optional: keeper worker could run as Azure Function / container later
- No API is required for MVP

---

## Current Repo State
- There is a Next.js template app running locally.
- It is NOT connected to any chain yet.
- It does NOT produce or deploy contracts yet.
- There is no Hardhat folder yet.
- We need to add a proper contract project structure.

---

## Planned Repo Structure
We discussed and agreed a clean long-term layout:

/contracts   -> Hardhat + Solidity + deploy scripts + artifacts
/web         -> Next.js frontend (optional; if Next.js is already at root, it can stay there)

Key point:
- Hardhat project must NOT live under Next.js `src/`.

---

## MVP Plan (Incremental Milestones)
### Milestone 1 — Deploy a Hello World contract to Flare Coston2
- Add Hardhat project under `/contracts`
- Configure Coston2 network (chainId 114)
- Use `.env` with RPC + private key (burner wallet)
- Create a basic `HelloFlare.sol` contract:
  - stores a message string
  - `setMessage`
  - `getMessage`
  - emits event on update
- Deploy via `npx hardhat run ... --network coston2`
- Verify contract address in Coston2 explorer

### Milestone 2 — Connect Next.js to MetaMask + contract
- Add `ethers` to Next.js
- Implement wallet connect using `window.ethereum`
- Ensure correct chainId (114)
- Load contract using:
  - ABI
  - deployed address
- Read + write message from UI

### Milestone 3 — Add claim model and payment events
- Replace hello world with a contract that stores claims:
  - claim struct
  - mapping of claimId -> claim
  - create claim
  - execute claim if eligible
  - emit events for created + paid

### Milestone 4 — Flare protocol usage (FTSO)
- Add Flare periphery package
- Read price feed inside contract
- Enforce payout eligibility based on FTSO price

### Milestone 5 — UI lists + payments timeline
- Load claims by reading contract state
- Load payment history by querying events

Optional:
- keeper bot / worker service to automate execution

---

## Important Hackathon Framing
We want the contract to be the source of truth and enforce rules:
- even if an Azure worker exists, contract must re-check FTSO at execution time

We want to clearly show in demo:
- claim created
- pending state
- current oracle price visible
- execute claim
- payment event emitted
- claim becomes paid

---

## What Copilot should do with this context
Use this to:
- Inspect the current repo layout
- Identify where to add `/contracts`
- Identify where Next.js currently lives (root or `/web`)
- Understand the architecture and incremental plan
- Understand the target chain, testnet, and Flare-specific constraints
- Understand how on-chain state + events will be used for listing without a DB

(Again: do not implement anything yet.)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SLAP-Solutions) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
