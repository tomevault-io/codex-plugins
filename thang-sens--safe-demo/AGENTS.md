# Safe Multisig + CCIP Cross-Chain Transfer POC

## 🎯 Project Overview

Multi-company Safe (Gnosis Safe) multisig wallet management with **Chainlink CCIP** cross-chain token transfers.

**Key Architecture**: React (Vite) → Express API → MongoDB Atlas → Safe SDK → Ethereum/Multi-chain

Each company has a Safe Smart Account with multi-sig approval for both regular and **cross-chain CCIP** transactions.

## 🔧 Critical Version Requirements

**IMPORTANT - Ethers Version Split**:
- **Server**: `ethers v5` (required by Safe Protocol Kit v6 for deployment)
- **Client**: `ethers v6` (required by Web3Auth + CCIP SDK)
- **Safe SDK**: Protocol Kit v6, API Kit v4 (using `@safe-global/types-kit`, NOT deprecated `safe-core-sdk-types`)

**Never mix ethers versions** - server uses `ethers.providers.JsonRpcProvider`, client uses `BrowserProvider`.

## 🧠 Development Guidelines

### TypeScript & Structure
- All code in TypeScript (strict separation: `/server` vs `/client`)
- Use ESM imports (`import/export`, not `require`)
- **Never use `.js` extension in imports** - TypeScript handles this
- Server uses `type: "module"` in package.json

### Safe Transaction Flow (client/src/lib/safeFlow.ts)
**Critical pattern for CCIP transfers**:

1. **NEVER batch approval + CCIP send** - Safe's MultiSend uses DelegateCall which fails for CCIP
2. **Always propose separately**: 
   - First tx: `approve(routerAddress, amount)` 
   - Second tx: `ccipSend(...)` with high gas limit
3. **Check fresh on-chain allowance** before building CCIP tx - don't trust cache
4. **Use 5M gas limit** for CCIP execution (`executeTransaction` options)

```typescript
// Example from safeFlow.ts line 1156-1227
// IMPORTANT: We cannot batch approval + CCIP send
const transactions: MetaTransactionData[] = [];

// Step 1: Propose approval transaction
if (needsApproval) {
  const approvalTxHash = await proposeTransaction(/*...*/);
}

// Step 2: Propose CCIP send transaction (separate!)
const ccipTxHash = await proposeTransaction(/*...*/);
```

### Web3Auth Provider Pattern
**Critical workaround** for Safe SDK signing (client/src/lib/web3auth.ts):

- Safe's `signTransaction` requires `eth_signTypedData_v4`
- Use `getRawProvider()` to get Web3Auth's raw provider (not BrowserProvider wrapper)
- Pass raw provider to Safe.init as `provider` param

```typescript
const rawProvider = await getRawProvider();
const safe = await Safe.init({
  provider: rawProvider as unknown as string,
  signer: await signer.getAddress(),
  safeAddress,
});
```

### CCIP Configuration (client/src/lib/ccipConfig.ts)
- Network configs define `chainSelector`, `routerAddress`, supported tokens
- **4 testnets**: Ethereum Sepolia, Arbitrum Sepolia, Avalanche Fuji, Polygon Amoy
- CCIP message format uses **zeroHash for data** (not empty bytes) - matches SDK

## 🧪 Testing & Debugging

### Root-level Debug Scripts (node *.js)
**13 standalone debugging scripts** in project root - use these for troubleshooting:

- `test-ccip-safe.js` - Pre-flight checks (balances, approvals, gas estimates)
- `debug-ccip-tx.js <TX_HASH>` - Analyze failed CCIP transaction
- `compare-direct-vs-safe.js <TX_HASH>` - Compare working vs failing tx patterns
- `check-safe-status.js` - Verify Safe ownership, threshold, nonce
- `inspect-pending-tx.js` - List pending Safe transactions
- `reject-all-pending.js` - Mass-reject stuck transactions

**Pattern**: These scripts use plain ethers.js (no framework) for direct blockchain inspection.

### Development Workflow
```bash
# Terminal 1 - Backend (port 3000)
cd server && npm run dev

# Terminal 2 - Frontend (port 5173)  
cd client && npm run dev

# Debug CCIP issues
node test-ccip-safe.js
node debug-ccip-tx.js 0xTX_HASH
```

## 🔐 Environment Variables

**Server (.env)**: 
- `DEPLOYER_PRIVATE_KEY` - Must have Sepolia ETH (uses ethers v5)
- `INFURA_RPC_URL`, `MONGO_URI`, `SAFE_TX_SERVICE_URL`

**Client (.env)**:
- `VITE_WEB3AUTH_CLIENT_ID` - Web3Auth dashboard
- `VITE_CHAIN_ID` - **Must be hex** (e.g., `0xaa36a7` for Sepolia, NOT `11155111`)
- Multi-chain RPC URLs for CCIP (Arbitrum, Avalanche, Polygon)

## 📦 Key Files & Patterns

**Backend**:
- `server/services/safeService.ts` - Safe deployment using ethers v5
- `server/models/Company.ts` - Mongoose schema with Safe address mapping

**Frontend**:
- `client/src/lib/safeFlow.ts` (1378 lines) - **Core logic**: propose/confirm/execute, CCIP integration
- `client/src/lib/ccipConfig.ts` - Network configs, chain selectors, token addresses
- `client/src/lib/ccipSafeDebug.ts` - Diagnostic helpers (`checkApprovalStatus`, `analyzeSafeTransactionFailure`)
- `client/src/components/CCIPTransfer.tsx` - Multi-chain transfer UI with fee estimation

## ⚠️ Common Pitfalls

1. **GS013 Error**: Signatures not sorted by owner address (ascending). `safeFlow.ts:320` shows fix.
2. **CCIP Approval Timing**: Must execute approval tx BEFORE executing CCIP send (separate nonces).
3. **Gas Limit**: CCIP needs 5M gas limit on execution, NOT Safe's internal safeTxGas (3M).
4. **Chain ID Format**: Client env must use hex format (`0xaa36a7`), server uses decimal.
5. **Import Extensions**: Never add `.js` to TypeScript imports - causes resolution issues.

## 📚 Reference Docs

See `/docs` folder and root README.md, QUICKSTART.md for comprehensive guides.
Safe Flow: `SAFE_FLOW_DOCUMENTATION.md` | CCIP: `CCIP_INTEGRATION_GUIDE.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thang-sens)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/thang-sens)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
