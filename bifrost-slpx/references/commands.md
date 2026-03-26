# Command reference

Always pass `--json` when invoking from an agent unless you intentionally need human-readable text.

Prefix: `npx -y @bifrostio/slpx-cli`

## Query commands (all vTokens)

### `rate [amount]`

Exchange rate. Default: 1 unit of base asset.

```bash
npx -y @bifrostio/slpx-cli rate --json
npx -y @bifrostio/slpx-cli rate 10 --json
npx -y @bifrostio/slpx-cli rate --token vDOT --json
npx -y @bifrostio/slpx-cli rate 100 --token vKSM --json
```

Output fields: `inputAmount`, `outputAmount`, `inputToken`, `outputToken`, `rate` (`tokenToBase`: `inputToken` per 1 `outputToken`), `source` (on-chain fallback also: `chain`)

**`rate` semantics (agents):** snapshot only—how many units of `inputToken` one `outputToken` commands **at request time**. The JSON has **no** inception date, **no** history, and **no** “cumulative return” label. Do **not** equate `(rate − 1)` with “appreciation since inception” or any other period; that is inferential, not in the data.

### `apy`

Staking APY (base + reward). Add `--lp` for LP pool yields from DeFiLlama.

```bash
npx -y @bifrostio/slpx-cli apy --json
npx -y @bifrostio/slpx-cli apy --token vDOT --json
npx -y @bifrostio/slpx-cli apy --token vMANTA --json
npx -y @bifrostio/slpx-cli apy --token vDOT --lp --json
```

Output fields: `token`, `totalApy`, `baseApy`, `rewardApy`, `rewardApyIncentiveAsset`

- `baseApy` — native staking yield on the underlying chain (Bifrost API: `apyBase`)
- `rewardApy` — Bifrost farming incentive component (Bifrost API: `apyReward`)
- `rewardApyIncentiveAsset` — **vDOT** for **vETH** only; **BNC** for every other vToken
- `totalApy` — `baseApy` + `rewardApy`

With `--lp`: adds `lpPools` — each entry: `symbol`, `project`, `chain`, `lpApy`, `tvl`

### `info`

Protocol overview: rate, APY, TVL, holders. vETH also: contract, chains, paused.

```bash
npx -y @bifrostio/slpx-cli info --json
npx -y @bifrostio/slpx-cli info --token vDOT --json
npx -y @bifrostio/slpx-cli info --token vASTR --json
```

Output fields: `protocol`, `inputAmount`, `outputAmount`, `inputToken`, `outputToken`, `rate` (`tokenToBase`), `totalApy`, `baseApy`, `rewardApy`, `tvl`, `totalStaked` (amount in `inputToken`), `totalSupply` (amount in `outputToken`), `holders`  
`rate` here matches **`rate` command** semantics: **spot** `tokenToBase`, **no** time dimension in the JSON.  
vETH extra: `contract`, `chains`, `paused`

## On-chain commands (vETH only)

### `balance [address]`

vETH balance and ETH equivalent. **Omit `address`** to query the wallet from `BIFROST_SKILL_PRIVATEKEY`. Comma-separated addresses for batch.

```bash
npx -y @bifrostio/slpx-cli balance --json
npx -y @bifrostio/slpx-cli balance 0x742d...bD18 --json
npx -y @bifrostio/slpx-cli balance 0x742d...bD18 --chain base --json
npx -y @bifrostio/slpx-cli balance 0xAddr1,0xAddr2,0xAddr3 --json
```

Single: `address`, `vethBalance` (vETH, amount only), `ethValue` (ETH, amount only), `chain`  
Batch: `results` (`{address, vethBalance, ethValue}`), `chain`

### `status [address]`

Redemption queue status. Omit address to use `BIFROST_SKILL_PRIVATEKEY` (same as `balance`).

```bash
npx -y @bifrostio/slpx-cli status --json
npx -y @bifrostio/slpx-cli status 0x742d...bD18 --json
```

Output: `address`, `claimableEth` (ETH, amount only), `pendingEthAmount` (ETH, amount only), `chain`, `hint` (time estimate when pending; may include units in prose)

### `mint <amount>`

Stake ETH → vETH. `--weth` uses WETH instead of native ETH.

```bash
npx -y @bifrostio/slpx-cli mint 0.1 --json --dry-run --address 0x742d...
npx -y @bifrostio/slpx-cli mint 0.5 --chain base --json
npx -y @bifrostio/slpx-cli mint 0.1 --weth --json --dry-run --address 0x742d...
npx -y @bifrostio/slpx-cli mint 0.5 --weth --chain arbitrum --json
```

Native ETH (unsigned): `action`, `inputAmount`, `inputToken` (`ETH`), `expectedAmount`, `expectedToken` (`vETH`), `mode`, `from` (formatted signer/receiver when dry-run used key or `--address`), `unsigned.to`, `unsigned.value`, `unsigned.data`, `unsigned.chainId`  
WETH (unsigned): `action:mint-weth`, `inputAmount`, `inputToken` (`WETH`), `expectedAmount`, `expectedToken` (`vETH`), `mode`, `from`, `wethAddress`, `steps` (Approve, Deposit; deposit `receiver` matches `from` when address-only dry-run)  
Signed: same amount/token fields plus `from`, `txHash`, `explorer`

WETH contract addresses: see `tokens-and-chains.md`.

### `redeem <amount>`

Redeem vETH — **not instant** (queue, often 1–3 days).

> Before a real `redeem`, confirm with the user: amount, chain, and that redemption is queued. Use `--dry-run` first.

```bash
npx -y @bifrostio/slpx-cli redeem 0.1 --json --dry-run --address 0x742d...
```

### `claim`

Claim completed ETH after redemption.

```bash
npx -y @bifrostio/slpx-cli claim --json --dry-run --address 0x742d...
```

## Global options

| Option | Description | Default |
|--------|-------------|---------|
| `--token <name>` | `vETH`, `vDOT`, `vKSM`, `vBNC`, `vGLMR`, `vMOVR`, `vFIL`, `vASTR`, `vMANTA`, `vPHA` | `vETH` |
| `--chain <name>` | `ethereum`, `base`, `optimism`, `arbitrum` (vETH on-chain only) | `ethereum` |
| `--rpc <url>` | Custom RPC | auto per chain |
| `--json` | JSON output | false |

## Transaction-related options

| Option | Description |
|--------|-------------|
| `--dry-run` | Build unsigned tx, do not broadcast |
| `--weth` | Mint from WETH (not native ETH) |
| `--lp` | LP yields on `apy` only |
| `--address <addr>` | **`mint` / `redeem` / `claim`:** required for `--dry-run` when `BIFROST_SKILL_PRIVATEKEY` is unset; else optional if the env key is set. **`balance` / `status`:** use the positional `[address]` instead (comma-separated batch for `balance`); omit address only when the env key is set. |

## Workflow examples

### Compare APYs (with LP)

```bash
npx -y @bifrostio/slpx-cli apy --token vDOT --lp --json
npx -y @bifrostio/slpx-cli apy --token vETH --lp --json
npx -y @bifrostio/slpx-cli apy --token vKSM --json
```

### Batch balances

```bash
npx -y @bifrostio/slpx-cli balance 0xAddr1,0xAddr2,0xAddr3 --json
```

### Research → stake (vETH)

```bash
npx -y @bifrostio/slpx-cli info --json
npx -y @bifrostio/slpx-cli rate 1 --json
npx -y @bifrostio/slpx-cli apy --json
npx -y @bifrostio/slpx-cli mint 0.5 --json --dry-run --address 0xYour...
# With BIFROST_SKILL_PRIVATEKEY set, `--address` can be omitted on `--dry-run`.
# Remove `--dry-run` only after pre-tx checklist + explicit user approval.
```

### Redeem → claim (vETH)

```bash
npx -y @bifrostio/slpx-cli balance 0x... --json
npx -y @bifrostio/slpx-cli redeem 1.0 --json
npx -y @bifrostio/slpx-cli status --json
npx -y @bifrostio/slpx-cli claim --json
```

## Operational notes

1. Query commands use the Bifrost API for all 10 vTokens.
2. On-chain commands are vETH-only on the listed EVM chains.
3. Redemption goes through Bifrost’s cross-chain queue — not immediate settlement.
4. **`mint` / `redeem` / `claim`:** broadcasting without `--dry-run` requires a valid `BIFROST_SKILL_PRIVATEKEY`; `--dry-run` requires that env var **or** `--address`. See `references/errors.md` (`NO_PRIVATE_KEY`, `NO_PRIVATE_KEY_OR_ADDRESS`).
5. **`balance` / `status`:** omitting the address requires `BIFROST_SKILL_PRIVATEKEY`; else pass an address. See `NO_ADDRESS_OR_PRIVATE_KEY` in `references/errors.md`.
6. RPC: CLI may fall back to backup endpoints if primary RPC fails.
7. `--weth` unsigned flow emits Approve + Deposit steps.
8. `--lp` on `apy` pulls DeFiLlama pool data for the selected vToken.
