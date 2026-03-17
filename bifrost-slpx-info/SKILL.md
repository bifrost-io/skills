---
name: bifrost-slpx-info
description: |
  Query Bifrost SLPx liquid staking protocol data — vETH/ETH exchange rates, APY, TVL,
  user balances, redemption queue status, and protocol stats.
  Primary data source: Bifrost REST API. Fallback: on-chain ERC-4626 vault calls on Ethereum.
  Use when users ask about Bifrost staking rates, vETH prices, DeFi yield, or vToken holdings.
keywords:
  - bifrost
  - vETH
  - liquid-staking
  - DeFi
  - ETH
  - staking
  - ERC-4626
  - yield
  - APY
metadata:
  author: bifrost.io
  version: "1.0.0"
---

# Bifrost SLPx Info

Query Bifrost vETH liquid staking data. **Use the Bifrost REST API as the primary data source**; fall back to on-chain calls on Ethereum only when the API is unavailable or when user-specific data is needed.

## Data Source Strategy

| Data Type | Primary Source | Fallback |
|-----------|---------------|----------|
| APY, TVL, holders, total issuance | **Bifrost API** | — |
| Exchange rate (ETH ↔ vETH) | **Bifrost API** (derive from `tvm / totalIssuance`) | On-chain `convertToShares` / `convertToAssets` on Ethereum |
| User balance (`balanceOf`) | On-chain (must query the chain where user holds vETH) | — |
| User withdrawal status | On-chain (`canWithdrawalAmount`, `getWithdrawals`) | — |
| Protocol state (`paused`, `asset`, `name`, etc.) | On-chain on Ethereum (synced across chains) | — |

> **Why Ethereum only for on-chain?** Exchange rates, `totalAssets`, and contract metadata are Oracle-synced and identical across all chains. Only `balanceOf`, `totalSupply`, and withdrawal data are chain-specific. Querying Ethereum avoids unnecessary multi-chain complexity.

## Bifrost REST API (Primary)

```
GET https://api.bifrost.app/api/site
```

Returns JSON. Parse the `vETH` object:

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `apy` | number | Total APY (base + farming rewards) | `7.97` |
| `apyBase` | number | Base staking APY from Ethereum validators | `3.04` |
| `apyReward` | number | Additional Bifrost farming reward APY | `4.93` |
| `tvl` | number | Total value locked in USD | `1795072.67` |
| `tvm` | number | Total ETH staked (across all chains) | `764.21` |
| `totalIssuance` | number | Total vETH supply (across all chains) | `668.21` |
| `holders` | number | Number of vETH holders across all chains | `178` |

**Deriving exchange rate from API:**
- 1 ETH → vETH: `totalIssuance / tvm` (e.g. 668.21 / 764.21 ≈ 0.8744 vETH)
- 1 vETH → ETH: `tvm / totalIssuance` (e.g. 764.21 / 668.21 ≈ 1.1436 ETH)

Use this API for: APY queries, TVL queries, holder count, exchange rate overview, protocol summary.

## Contract & Network (Fallback / User Queries)

VETH contract: `0xc3997ff81f2831929499c4eE4Ee4e0F08F42D4D8` (same address on all chains)

| Chain | ChainId | WETH (underlying) | Default RPC |
|-------|---------|--------------------|----|
| Ethereum | 1 | `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2` | `https://ethereum.publicnode.com` |
| Base | 8453 | `0x4200000000000000000000000000000000000006` | `https://base.publicnode.com` |
| Optimism | 10 | `0x4200000000000000000000000000000000000006` | `https://optimism.publicnode.com` |
| Arbitrum | 42161 | `0x82aF49447D8a07e3bd95BD0d56f35241523fBab1` | `https://arbitrum-one.publicnode.com` |

Fallback RPC: `https://1rpc.io/eth` (Ethereum), `https://1rpc.io/base`, `https://1rpc.io/op`, `https://1rpc.io/arb`

## Configuration (Environment Variables)

On first run, ask the user whether they want to configure custom settings. If not, use the defaults.

| Variable | Description | Default |
|----------|-------------|---------|
| `BIFROST_CHAIN` | Target chain for user-specific queries (`ethereum`, `base`, `optimism`, `arbitrum`) | `ethereum` |
| `BIFROST_RPC_URL` | Custom RPC endpoint | Per-chain default from table above |

## Quick Reference

VETH is an ERC-4626 vault inheriting from `VToken → VTokenBase → ERC4626Upgradeable`. All standard ERC-4626 view functions are available. Exchange rates are Oracle-backed.

### Token Info

| Query | Function | Selector | Args | Returns | Description |
|-------|----------|----------|------|---------|-------------|
| Token name | `name()` | `0x06fdde03` | none | string | Returns `"Bifrost Voucher ETH"` |
| Token symbol | `symbol()` | `0x95d89b41` | none | string | Returns `"vETH"` |
| Decimals | `decimals()` | `0x313ce567` | none | uint8 | Returns `18` |
| Underlying asset | `asset()` | `0x38d52e0f` | none | address | Returns the WETH address (differs per chain, see Contract & Network table) |

### Exchange Rate Queries

| Query | Function | Selector | Args | Returns | Description |
|-------|----------|----------|------|---------|-------------|
| ETH → vETH rate | `convertToShares(uint256)` | `0xc6e6f592` | assets (wei) | uint256 (shares) | Convert an ETH amount to the equivalent vETH shares using the current Oracle exchange rate |
| vETH → ETH rate | `convertToAssets(uint256)` | `0x07a2d13a` | shares (wei) | uint256 (assets) | Convert a vETH shares amount to the equivalent ETH value using the current Oracle exchange rate |
| Preview deposit | `previewDeposit(uint256)` | `0xef8b30f7` | assets (wei) | uint256 (shares) | Simulate a deposit and return the exact vETH shares that would be minted |
| Preview mint | `previewMint(uint256)` | `0xb3d7f6b9` | shares (wei) | uint256 (assets) | Simulate minting a specific amount of vETH shares and return the ETH required |
| Preview withdraw | `previewWithdraw(uint256)` | `0x0a28a477` | assets (wei) | uint256 (shares) | Simulate withdrawing a specific ETH amount and return the vETH shares that would be burned |
| Preview redeem | `previewRedeem(uint256)` | `0x4cdad506` | shares (wei) | uint256 (assets) | Simulate a redemption and return the exact ETH that would be returned |

### User Balance & Withdrawal Queries

| Query | Function | Selector | Args | Returns | Description |
|-------|----------|----------|------|---------|-------------|
| vETH balance | `balanceOf(address)` | `0x70a08231` | owner address | uint256 | Get the vETH token balance of a specific address |
| Claimable ETH | `canWithdrawalAmount(address)` | `0x52a630b9` | target address | (uint256, uint256, uint256) | Returns `(totalAvailableAmount, pendingDeleteIndex, pendingDeleteAmount)`. First value = ETH ready to claim |
| Withdrawal queue | `getWithdrawals(address)` | `0x3a2b643a` | target address | Withdrawal[] | Returns array of withdrawal entries, each with `queued` (amount in queue) and `pending` (amount being processed) fields |
| Max redeemable | `maxRedeem(address)` | `0xd905777e` | owner address | uint256 | Maximum vETH shares the owner can redeem in a single tx |
| Max withdrawable | `maxWithdraw(address)` | `0xce96cb77` | owner address | uint256 | Maximum ETH the owner can withdraw in a single tx |
| Max depositable | `maxDeposit(address)` | `0x402d267d` | receiver address | uint256 | Maximum ETH that can be deposited for receiver (typically type(uint256).max = unlimited) |
| Max mintable | `maxMint(address)` | `0xc63d75b6` | receiver address | uint256 | Maximum vETH shares that can be minted for receiver (typically type(uint256).max = unlimited) |
| Allowance | `allowance(address,address)` | `0xdd62ed3e` | owner, spender | uint256 | Check vETH spending allowance granted by owner to spender |

### Protocol Statistics

| Query | Function | Selector | Args | Returns | Description |
|-------|----------|----------|------|---------|-------------|
| Total assets | `totalAssets()` | `0x01e1d114` | none | uint256 | Total ETH managed by the vault, sourced from Oracle `poolInfo(asset)` |
| Total supply | `totalSupply()` | `0x18160ddd` | none | uint256 | Total vETH tokens in circulation on this chain |
| Vault balance | `getTotalBalance()` | `0x12b58349` | none | uint256 | ETH available for payouts (BridgeVault balance + completed withdrawals) |
| Queued withdrawals | `queuedWithdrawal()` | `0x996e5c06` | none | uint256 | Total ETH amount currently queued for withdrawal globally |
| Completed withdrawals | `completedWithdrawal()` | `0x63ea1b92` | none | uint256 | Total ETH amount that has completed withdrawal processing globally |
| Max withdraw count | `maxWithdrawCount()` | `0xdc692cd7` | none | uint256 | Maximum number of concurrent pending withdrawal entries per user |
| Paused | `paused()` | `0x5c975abb` | none | bool | Whether the contract is currently paused (deposits/redeems disabled) |

## How to Call (On-Chain Fallback)

For user-specific queries or when the API is unavailable. All queries are read-only `eth_call` — no gas, no signing. Default to Ethereum RPC.

**Method A: cast** (preferred)
```bash
cast call <VETH_CONTRACT> \
  "<FUNCTION_SIGNATURE>(<ARG_TYPES>)(<RETURN_TYPES>)" <ARGS> \
  --rpc-url <RPC_URL>
```

Example — query exchange rate for 1 ETH on Ethereum:
```bash
cast call 0xc3997ff81f2831929499c4eE4Ee4e0F08F42D4D8 \
  "convertToShares(uint256)(uint256)" 1000000000000000000 \
  --rpc-url https://ethereum.publicnode.com
```

**Method B: curl + JSON-RPC** (if cast unavailable)
```bash
curl -s -X POST <RPC_URL> \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_call","params":[{"to":"<VETH_CONTRACT>","data":"<SELECTOR><ENCODED_ARGS>"},"latest"]}'
```

### Calldata Encoding

- **No-arg functions**: calldata = selector only (e.g., `0x01e1d114` for `totalAssets()`)
- **uint256 arg**: selector + amount in hex, left-padded to 64 chars
  - Example: 1 ETH = `0xDE0B6B3A7640000` → pad to `0000000000000000000000000000000000000000000000000de0b6b3a7640000`
- **address arg**: selector + address (no 0x), left-padded to 64 chars
  - Example: `0xAbCd...1234` → `000000000000000000000000AbCd...1234`

### Response Decoding

- curl returns `{"result":"0x<hex>"}`. For single uint256: convert 64 hex chars to decimal, divide by 1e18.
- `canWithdrawalAmount` returns 3 × uint256 (192 hex chars): `(totalAvailableAmount, pendingDeleteIndex, pendingDeleteAmount)`. First 64 chars = claimable ETH amount.

## Agent Behavior

1. **API first**: for protocol-level queries (APY, TVL, exchange rate, holders), always try the Bifrost REST API first. Only fall back to on-chain calls if the API is unreachable
2. **On-chain for user data**: `balanceOf`, `canWithdrawalAmount`, `getWithdrawals`, `maxRedeem` require on-chain calls. Default to Ethereum unless user specifies a chain
3. **Environment check**: on first interaction, ask user if they want to configure `BIFROST_CHAIN` or `BIFROST_RPC_URL`. If not, use Ethereum defaults
4. **Chain selection**: only switch chains when querying user-specific data (balance, withdrawals) on a specific chain. Exchange rates and protocol stats are the same on all chains — just use Ethereum or the API
5. **RPC selection**: use `BIFROST_RPC_URL` if set; otherwise use per-chain default RPC. Fall back to fallback RPC on failure
6. All values are in wei (18 decimals) — always convert to human-readable before displaying
7. Default exchange rate query: show rate for 1 ETH if no amount specified
8. Prompt for wallet address if not provided; display truncated (first 6 + last 4 chars)
9. If `canWithdrawalAmount` first value > 0, indicate claimable and suggest "claim my redeemed ETH"
10. For on-chain calls: prefer `cast call`; fall back to `curl` + JSON-RPC if cast unavailable
11. Always fetch fresh data — do not cache across requests
12. **Useful links**: direct users to [Bifrost vETH page](https://www.bifrost.io/vtoken/veth) or [Bifrost App](https://app.bifrost.io/vstaking/vETH) when relevant

## Error Handling

| Error | User Message |
|-------|-------------|
| API failure (non-200 / timeout) | Fall back to on-chain query on Ethereum. If both fail: "Unable to fetch data. Please try again later." |
| RPC failure | "Unable to connect. Retrying with backup endpoint..." → try fallback RPC |
| Zero from `convertToShares` | "The vETH exchange rate is temporarily unavailable. The contract may be paused." |
| Empty withdrawal array | "You have no pending vETH redemptions." |
| Invalid address | "Please provide a valid Ethereum address (0x + 40 hex characters)." |
| `execution reverted` | Retry with alternate method (cast ↔ curl). If both fail, report to user. |

## Notes

1. VETH inherits `VToken → VTokenBase → ERC4626Upgradeable / OwnableUpgradeable / PausableUpgradeable`. All standard ERC-4626 view functions work
2. Exchange rates are Oracle-backed via `oracle.poolInfo(asset)` — may lag slightly vs. actual staking rewards
3. `getWithdrawals` returns `Withdrawal[]` where each entry has `queued` (amount waiting in queue) and `pending` (amount being processed by Bifrost)
4. The same VETH contract address is deployed on Ethereum, Base, Optimism, and Arbitrum — but WETH (underlying asset) addresses differ per chain
5. **Oracle data is synced across chains**: `convertToShares`, `convertToAssets`, `totalAssets` return identical values on all chains. No need to query L2s for exchange rates
6. **`totalSupply` is chain-specific**: Ethereum holds ~263 vETH, Base ~1 vETH, Arbitrum ~0.016 vETH. The API `totalIssuance` field gives the cross-chain total (~668 vETH)
7. `getTotalBalance()` = BridgeVault balance + completed withdrawals — represents ETH available for immediate payouts
8. **API vs on-chain `totalIssuance`**: the API reports cross-chain total vETH supply (including Bifrost parachain), while on-chain `totalSupply()` is per-chain only
